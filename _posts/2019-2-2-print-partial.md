---
layout: post
title: 随笔--浏览器打印部分内容
---

假设有需求如: 某个页面在使用浏览器默认打印功能导出/打印时只显示部分内容, 或隐藏某些元素;

## 1. CSS3媒体查询
打印时隐藏某些元素, 可以在CSS中写入以下代码, HTML中只需要对需要隐藏的元素设置`class="noPrint"`
```css
@media print {
    .noPrint{ display: none; } 
}
```

## 2. JavaScript操作DOM并打印
浏览器打印的方法是`window.print()`;

1. 网上找到的通过在body中置换html的方法并不好用, 还会使原先填写内容丢失和绑定事件的按钮失效;
```javascript
const print1 = function () {
    const doc = document;
    const body = doc.body;
    const oldHtml = body.innerHTML;
    body.innerHTML = doc.querySelector("#f1").outerHTML;
    window.print();
    body.innerHTML = oldHtml;
}
doc.querySelector("#print").addEventListener("click", print);
```
2. 先把body元素深克隆, 把body替换后打印, 最后还原, 也会造成原先绑定的事件丢失;另外`innerHTML`或`outerHTML`复制的html片段, 不会包含动态特性, 比如input未指定value而手动填入值, `innerHTML`/`outerHTML`中不会体现; 所以填写的值在打印的页面会丢失;
```javascript
const print2 = function () {
    const doc = document;
    const body = doc.body;
    const oldBody = body.cloneNode(true);
    body.innerHTML = doc.querySelector("#f1").outerHTML;
    window.print();
    body = oldBody;
}
```
3. 把body深克隆, 把要打印的元素保留在页面用于打印, 最后替换回原body, 也会造成事件丢失
```javascript
const print3 = function () {
    const body = doc.body;
    const oldBody = body.cloneNode(true);
    const f1 = document.querySelector("#f1");
    body.innerHTML = "";
    body.appendChild(f1);
    window.print();
    body = oldBody;
}
```

### 原因: 
节点的深克隆只能复制节点的结构和属性, 包括通过属性`onclick`绑定的事件, 不能复制动态绑定(在JavaScript代码中绑定)的事件;

`innerHTML`或`outerHTML`也不会复制绑定的事件, 只是复制HTML中的静态标签所包含的内容字符串(`outerHTML`会包括标签本身);

### 解决办法:
可以新建一个容器, 填入希望打印的元素(克隆)后覆盖在原网页之上用于打印, 然后再把该新元素从body移除, 这样不会对原来的页面造成影响, 也能正确反映输入框中修改的内容;
```javascript
// 不会丢失内容, 也不会丢失事件
const print = function () {
    const content = doc.createElement("div");
    content.style = "position:absolute;top:0;bottom:0;left:0;right:0;background:white";
    content.appendChild(doc.querySelector("#f1").cloneNode(true));
    doc.body.appendChild(content);
    window.print();
    doc.body.removeChild(content);
}
doc.querySelector("#print").addEventListener("click", print);
```


