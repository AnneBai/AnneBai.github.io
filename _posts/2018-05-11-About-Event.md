---
layout: post
title: js随笔--关于事件
---

### 事件流
>事件流描述的是从页面中接收事件的顺序；
>
>事件冒泡：事件开始时由最具体的元素（最内层）逐级向上传播到较不具体的节点（文档）；
>
>事件捕获：事件开始时由相对不具体的元素最快接收到事件，而最具体的元素最后接收到事件。

书中的解释比较绕，其实我比较粗暴的理解就是，冒泡就像水中的波纹传播，从触发点开始向外扩展；而捕获则像给皇帝传递消息，皇帝才是消息的目标，但消息要从城门到宫门到殿门一路传过来，最后才到皇帝手中。

### DOM事件流

>“DOM2级事件”规定的事件流包括三个阶段：*事件捕获阶段*、*处于目标阶段*和*事件冒泡*阶段。

### 事件处理程序

>事件处理程序：对用户或浏览器自身执行的某种动作（事件）作出响应的函数；名字以`on`开头，比如`onclick`, `onload`, `onmouseover`等；
>
为事件指定事件处理程序的方式包括：

1. 在HTML元素对象的同名属性中指定，可以是可执行的代码或调用已定义的函数。

    这种方式能够在执行时创建动态函数，可以直接访问事件对象及其属性。但缺点是易产生页面与程序的加载时差导致不可用、以及浏览器解析差异，且内容与行为不能分离；

2. 在脚本中将元素的事件处理程序对应属性设置为一个函数（DOM0级），常见比如`element.onclick = function`。

    这种方式中，事件处理程序在元素的作用域中执行，this引用当前指定元素；添加的事件处理函数可使用`element.onclick = null`移除；

3. 使用`addEventListener()`方法指定。

    可添加多个事件处理程序，且都在其依附的元素的作用域中执行；
    移除函数可使用`removeEventListener()`，必须使用与添加时相同的函数名称，添加的匿名函数无法移除；

4. IE8及更早版本中使用`attachEvent()`和`detachEvent()`方法添加和移除事件处理程序；
    使用`attachEvent()`方法添加的事件在**全局作用域**中执行，只支持事件冒泡。

跨浏览器解决方案：
```javascript
var EventUtil = {
    addHandler: function (element, type, handler) {
        if (element.addEventListener) {
            element.addEventListener(type, handler, false);
        } else if (element.attachEvent) {
            element.attachEvent("on"+type, handler);
        } else {
            element["on"+type] = handler;
        }
    },
    removeHandler: function (element, type, handler) {
        if (element.removeEventListener) {
            element.removeEventListener(type, handler, false);
        } else if (element.detachEvent) {
            element.detachEvent("on"+type, handler);
        } else {
            element["on"+type] = null;
        }
    } 
}
```
这里的添加和移除事件处理程序在所有浏览器中都可用，但是没有考虑IE事件处理程序的作用域问题，如果`handler`中涉及`this`则需要小心。

### 事件对象
当事件在DOM触发，事件处理程序执行，会产生event对象，包含了事件类型、元素等相关信息；一旦事件处理程序执行完成，event对象会被销毁。不同浏览器对event的支持方式不同。

因为事件处理程序的作用域与指定它的方式有关，在事件处理程序内部，`this`并不始终等于事件目标（只有在事件处理函数直接指定给该目标且依赖此目标作用域执行时，`this`和`target`(或`srcElement`)才是一致的）。

例如绑定在`body`的事件处理函数，点击其内部的子元素，点击事件冒泡到`body`触发该函数；此时点击的目标是这个子元素，即事件对象的`target`，但`body`的事件处理函数因为是绑定在`body`上，其内部的`this`指向`body`.

可以通过红皮书中的案例`EventUtil`对象实现跨浏览器增删事件处理程序及获取事件对象和目标对象。
```javascript
var EventUtil = {
    getEvent: function (event) {
        return event? event : window.event;
    },
    getTarget: function (event) {
        return event.target || event.srcElement;
    }
}
```
### 事件委托
通过使用跨浏览器方案获取当前事件对象和目标对象，也能方便地实现事件委托--不必依靠对每个目标分别指定事件处理程序，只要在最外层元素指定的事件处理程序中，指定对于不同的`target`需要执行什么动作就可以了，这样对于内存优化和性能优化都有益。

一个事件委托的例子：
```javascript
// HTML中，box2 为 body, 内含 box1, 及 box1 的子元素 box;
      document.body.onclick = function (event) {
        event = EventUtil.getEvent(event);
        var target = EventUtil.getTarget(event);
        switch (target.id) {
          case "box":
            target.style.backgroundColor = "lightgreen";
            alert("this is box");
            break;
          case "box1":
            target.style.backgroundColor = "yellow";
            alert("this is box1")
            break;
          case "box2":
            target.style.backgroundColor = "purple";
            break;
          //对box2本身也可以直接用default
          // default:
          //   target.style.backgroundColor = "purple";
        }
        document.body.onclick = null;
      };

```
>每当将事件处理程序指定给元素时，运行中的浏览器代码与支持页面交互的JavaScript代码之间就会建立一个连接。这种连接越多，页面执行起来就越慢;

为了优化内存与性能，在移除页面上的元素前，先手动删除其带有的事件处理程序，否则就算这个元素被移除，它的事件处理程序占有的内存可能无法被回收。

***
2018-05-11