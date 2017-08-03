# domhandler3

基于[domhandler](https://www.npmjs.com/package/domhandler)的改造。

## 使用
```javascript
var handler = new DomHandler([ <func> callback(err, dom), ] [ <obj> options ]);
// var parser = new Parser(handler[, options]);
```

## 例子

```javascript
var htmlparser = require("htmlparser3");
var rawHtml = "Xyz <script language= javascript>var foo = '<<bar>>';< /  script><!--<!-- Waah! -- -->";
var handler = new htmlparser.DomHandler(function (error, dom) {
    if (error)
    	[...do something for errors...]
    else
    	[...parsing done, do something...]
        console.log(dom);
});

var parser = new htmlparser.Parser(handler);
parser.write(rawHtml);
parser.done();
```

Output:

```javascript
[{
    data: 'Xyz ',
    type: 'text'
}, {
    type: 'script',
    name: 'script',
    attribs: {
    	language: 'javascript'
    },
    children: [{
    	data: 'var foo = \'<bar>\';<',
    	type: 'text'
    }]
}, {
    data: '<!-- Waah! -- ',
    type: 'comment'
}]
```

## Option: normalizeWhitespace
指定文本节点的空白部分是否应该被标准化。默认值是："false"。

使用下面的html：

```html
<font>
	<br>this is the text
<font>
```

### 设置为: true

```javascript
[{
    type: 'tag',
    name: 'font',
    children: [{
    	data: ' ',
    	type: 'text'
    }, {
    	type: 'tag',
    	name: 'br'
    }, {
    	data: 'this is the text ',
    	type: 'text'
    }, {
    	type: 'tag',
    	name: 'font'
    }]
}]
```

### 设置为: false

```javascript
[{
    type: 'tag',
    name: 'font',
    children: [{
    	data: '\n\t',
    	type: 'text'
    }, {
    	type: 'tag',
    	name: 'br'
    }, {
    	data: 'this is the text\n',
    	type: 'text'
    }, {
    	type: 'tag',
    	name: 'font'
    }]
}]
```

## Option: withStartIndices
指定是否在每个节点都加入一个"startIndex"属性。当parser被使用在一个非流式的场景下时，`startIndex`是一个整数类型，指定节点在文档中的起始位置。默认值为：`false`。

## Option: attribTransforms

对象类型，允许开发者转换以及修改html标签。这个对象的属性必须要匹配正确的html标签。每一个属性都应该匹配一个函数，该函数接收一个"attribs"数组。你可以对attribs做任何你想做的事情，但是必须保证把他们返回回去，否则他们就会变成null！下面的例子把`img`和`a`标签的相对url路径转换成了绝对路径。  

或者是函数类型，会传递attribs给这个函数，函数只需要最后返回attribs即可。

```javascript
var htmlparser = require('htmlparser3');
var DomHandler = require('domhandler3');
var tohtml = require('htmlparser-to-html');
var url = require('url');

var html = '<a href="/NUlzFcY.gif"><img src="/NUlzFcY.gif"></a>';
var handler = function(domErr, dom) {
  if (domErr) {
    throw domErr;
  } else {
    html = tohtml(dom);
  }
};

var attribTransforms = {
  'img': function(attribs) {
    attribs.forEach(attrib => {
        if (attribs.src) {
          // this allows you to transform relative paths into absolute paths
          attrib.src = url.resolve('http://i.imgur.com', attrib.src);
        }
    });

    // remember to return the attribs or they will be null!
    return attribs;
  },
  'a': function(attribs) {
    attribs.forEach(attrib => {
        if (attrib.href) {
          attrib.href = url.resolve('http://i.imgur.com', attrib.href);
        }
    });

    return attribs;
  }
};

var domHandler = new DomHandler(handler, {attribTransforms: attribTransforms});

var parser = new htmlparser.Parser(domHandler);
parser.write(html);
parser.done();
console.log(html);
```
