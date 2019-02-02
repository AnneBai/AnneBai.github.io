---
layout: post
title: 再看闭包
---

## 闭包

> 闭包就是当一个函数即使是在它的词法作用域之外被调用时，也可以记住并访问它的词法作用域。

(这一篇相关的垃圾清理机制和函数作用域的理解已另外总结, 这里就不详细解释了)

我的理解: 如果在函数A内部创建一个函数B, 并通过赋值或返回的方式使外部作用域保留一个对内部函数B的引用, 那么函数A执行结束后,其所占用的内存/内部所有变量占用的内存都不会被回收, 直到函数B本身被销毁.

因为函数B拥有对函数A的作用域的引用, 垃圾回收到函数A这儿总会发现它"可能"还要被另一个随时可能执行的函数B用到, 只好完整保留它.

闭包的形式有多种,只要是实现可以在创建它的词法作用域之外的作用域中执行的函数, 都可以观察到闭包.
如:
```
function A() {
    var a = 1;
    function B() {
        console.log(a);
    }
    return B;
}
var b = A();
b(); // 1
```
或:
```
var b;
function A() {
    var a = 2;
    function B() {
        console.log(a);
    }
    b = B;
}
b(); // 2
```
函数内部创建函数返回给全局变量或执行中把内部函数赋值给全局变量, 全局变量持有内部函数B的引用, 内部函数不会被销毁, 同时它携带的外部函数A的整个变量对象也不会被销毁.

```
function A() {
    var a = 3;
    setTimeout(function(){
        console.log(a)
    }, 1000);
}
A(); // (1秒后打印) 3
```
使用计时器异步执行内部创建的函数, 也是闭包, 不过这个闭包只存在于A执行后1秒内.

计时器, 事件处理函数, 回调函数等都可能创建和使用闭包; 闭包本身与匿名函数/IIFE没有必然的关系, 它们只是闭包经常出现的场景.


## 循环中闭包的作用
```
for (var i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
// 5个6
```
使用闭包参数"固定"当前循环的变量值:
```
for (var i=1; i<=5; i++) {
	(function(i){
		setTimeout( function timer(){
			console.log( i );
		}, i*1000 );
	})(i)
}
// 1->5
```
这里闭包的作用与let/const相似, 把变量限制在当前循环的作用域而不是在外层共享.
```
for (let i=1; i<=5; i++) {
	setTimeout( function timer(){
		console.log( i );
	}, i*1000 );
}
// 1->5
```
## 模块中闭包的作用
封闭的作用域可以用来构建模块: 使用函数创建一个作用域, 在其内部声明一些变量和方法, 返回需要输出的变量和方法;

对外部作用域来说这些变量和方法是不存在的, 但是当执行模块函数并把返回值赋值给一个变量, 就得到了这个模块对象, 可以通过它访问到模块输出的变量和方法.

这些方法拥有访问模块函数内部所有变量的能力; 同时因为它们保留了对原模块函数的作用域的引用,所以它们也是闭包;
```
function moduleA() {
    var a = 1;
    var b = 2;
    var hi = "hello";
    function setB(x) {
        b = x;
    }
    function say() {
        console.log("a + b =", a + b, ",", hi);
    }
    return {
        setB,
        say,
    }
}
var module = moduleA();
module.say(); // a + b = 3 , hello
module.setB(3);
module.say(); // a + b = 4 , hello
```
得到的module可以看作是一个包含模块导出的属性的对象:
```
{
    say: ƒ say()
    setB: ƒ setB(x)
}
```
