---
layout: post
title: js随笔--JSON
---
 JSON，即 *Javascript Standard Object Notation* 或 *JavaScript Object Notation*

+ JSON是一种数据格式而非一种语言；使用javascript语言中的对象表示法；
+ JSON的语法可以表示**简单值**，**对象**，**数组**三种类型的值，不支持函数、变量或对象实例。
+ JSON表示数据的语法与JS相似，主要区别在于简单值不支持`undefined`，字符串和属性名必须使用双引号；表达对象和数组都没有变量声明也没有末尾分号；同一个对象内不能有两个同名属性；
+ JavaScript和JSON之间的互相转化可通过JSON对象的方法实现，`JSON.stringify()`和`JSON.parse()`除了第一个要转换的对象外还可以接收一个过滤器作为参数；过滤器可以是数组形式或函数形式；
+ 在过滤器中把某个属性值设置为`undefined`即为在输出结果中删除该对应属性。

```
sequenceDiagram
JavaScript->>JSON: (序列化) JSON.stringify()
JSON->>JavaScript: (解析) JSON.parse()
```
+ 序列化一个JavaScript对象时，`JSON.stringify()`还可以接收第三个参数设置缩进或空白符，用于结果的格式化；
+ `toJSON()`可以作为函数过滤器的补充；
+ 理解`JSON.stringify()`序列化一个javaScript对象的内部顺序：
    1. 如果有`toJSON()`且能返回有效值则调用该方法，否则返回对象本身；
    2. 如果有第二个参数，则应用此过滤器，传入第1步的返回值；
    3. 对第2步返回的对象每个值都进行相应序列化；
    4. 若有第三个参数，执行响应的格式化；

+ 对于动态数据，则只需要使用`for-in`循环，对对象中的属性遍历；需要检测属性值是否是数组（创建通用`isArray()`方法）；如果是数组也需要进行`for`循环遍历，以列表形式输出；
        
```
function isArray(arg) {
    if (typeof arg == "object") {
        var criteria = arg.constructor.toString().match(/array/i);
        return criteria != null;
    }
    return false;
}
```
+ 执行来自外部的信息需要先进行全面的检测确保安全；
+ 使用 `eval()` 可以让JS对传入其中的字符串脚本内容进行解析，但安全性很差，对于不能完全受控制的内容最好不要用`eval()`而是用`JSON.parse()`；
+ 输出可以使用新增DOM节点的方式；

