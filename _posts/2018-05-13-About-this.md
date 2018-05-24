---
layout: post
title: js随笔--关于this
---
——————2018-5-24 更新——————
ES6中的箭头函数是函数的另一种更简洁的表达形式；在箭头函数被创建时，this对象就已经被绑定给其创建时所在的对象而不是运行时再确定。可以认为箭头函数本就没有自己的this，如果它在另一个函数中被创建，那么它的this就指向包含函数的this, 这个对象在包含函数运行时确定；如果它被封装在一个对象中，那么它的this就指向这个对象。
——————原文——————

虽然这个问题大家都说不好写，但我还是尝试写出自己的理解，也是多一次整理思路的过程。初学者，有一些表述不当或理解不到位的地方，希望大家帮忙批评指正，感谢！

## 1) 基本概念
+ 执行环境和变量对象
> 执行环境定义了变量或函数有权访问的其他数据，决定了它们各自的行为；每个函数都有自己的执行环境，每个执行环境都有一个与之关联的变量对象，保存了环境中定义的所有变量和函数。
+ 作用域链
>当代码在一个环境中执行时，会创建变量对象的一个作用域链，保证对执行环境有权访问的所有变量和函数的有序访问；作用域链前端始终是当前执行代码所在环境的变量对象，其次是来自包含环境；全局执行环境的变量对象始终都是作用域链中最后一个对象。
+ this
>this引用的是函数执行的环境对象，在函数执行前this指向并不确定，只有在运行时基于函数的执行环境所绑定。

## 2) 常见情况
函数中的this根据被调用的情况/执行的环境不同其指向也不同，常见情况可能有以下几种：

1. 作为一个对象的方法被对象调用，或赋值给对象的一个属性后作为方法被调用---调用函数的对象；
2. 通过`call`, `apply`在传入对象的执行环境中调用，或用 `bind`绑定执行环境对象---绑定的对象；
3. 作为对象的事件处理程序函数，在对象发生该事件时触发执行---事件对象；
4. 作为构造器函数跟在new 后面被调用---返回的新对象；
5. 未通过以上方式绑定执行环境对象(而不是通过函数所在位置判断)---在全局作用域执行，this指向全局对象；

一个一个地说：

1. 常见的是作为对象的方法，可以直接作为对象的方法被定义，也可以作为公共方法在全局作用域定义然后赋值给对象的一个属性，通过对象来调用：
```
    var color = "blue";
    function test() {
      alert(this.color);
    }
    var obj = {
      color:"red",
      method:function () {
        alert(this.color); // 作为对象方法--this指向obj;
        test(); // 在对象方法里执行的全局方法--this指向全局；
      },
      method_1:test // 全局方法赋值给对象方法--this指向obj；
    };
    
    test(); // blue 
    obj.method(); // red  blue 
    obj.method_1(); // red 

```
2. 非对象方法的函数，调用时也可以通过call、apply绑定对象或在执行前用bind绑定对象；
>call, apply方法都是在特定的作用域中调用函数，实际上等于设置函数体内this对象的值，能够扩充函数赖以运行的作用域，并且作用域对象不需要与方法有任何耦合关系；

>bind()方法会创建一个函数的实例，其this值会被绑定到传给bind()函数的值。
```
    var color = "blue";
    function test() {
      alert(this.color);
    }
    var obj = {
      color:"red"
    };
    test(); //blue
    test.bind(obj)(); // red
    test.call(obj);// red
    test.apply(obj);// red
```
3. 作为事件处理程序，在事件发生时被调用，则指向发生此事件的对象：
```
  <p id="p1">这是一个段落</p>
  <script>
    function test() {
      alert(this);
    }
    var p1 = document.getElementById("p1");
    p1.onclick = function () {
      alert(this);        //[object HTMLParagraphElement]
      test();             //[object Window]
    }
  </script>

```
或通过把函数赋值给对象的事件属性：
```
    p1.onclick = test;     //[object HTMLParagraphElement]
```
即使是在事件处理函数内定义和执行的函数，没有绑定对象还是全局作用域：
```
//直接用表达式定义的函数，this指向全局
    p1.onclick = function () {
      var test1 = function () {
        alert(this);
      }
      test1(); // [object Window]
    }
// 如果是按对象方法定义的则会指向该对象
    p1.onclick = function () {
      p1.test2 = function () {
        alert(this);
      }
      p1.test2(); //[object HTMLParagraphElement]
    }
```

4. 作为构造器函数，跟在new后面被调用，this指向返回的新对象：
```
    var name = "Window",
        age = 100,
        say = function () {
          alert(name + ", " + age)
        };
    function Test() {
      this.name = "Anne";
      this.age = 18;
      this.say = function () {
        alert(this.name +", "+this.age);
      }
    }

    var a = new Test(); 
    a.say();         // Anne, 18
    window.say();    // window, 100
    Test.call(window);
    window.say();    // Anne, 18
```
5. 函数的作用域不能简单通过它执行时所在的位置，即使是嵌套在其他函数内部，嵌套在对象方法内部，只要没有在创建或调用函数时给它绑定对象，那么执行的作用域还是全局对象。

## 3) 特别注意
+ 数组也是一种对象，保存在数组中的函数在通过数组调用时实际上是在数组的作用域中执行，this会指向该数组；
```
      var arr = [test1,"second","third"];
      function test1() {
        alert(this.length);
      }
      
      arr[0]();  //3
```
+ 严格模式下未指定环境对象而调用函数，this值会是undefined而不会转型为window;
+ 指定事件处理函数时需要注意作用域问题，通过IE的`attachEvent`添加的事件处理程序函数是在全局作用域中执行，这时最好用跨浏览器解决方案利用`event`对象得到事件目标，而不要用`this`；

## 4) 总结
this在函数执行的过程中指向当前的环境对象；函数的执行环境不一定与定义时的环境相同，通过手动绑定对象作用域也能够对其执行环境进行控制。在使用事件处理程序函数时尤其要注意`this`的指向可能会发生变化。

