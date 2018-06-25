---
layout: post
title: ES6随笔--声明变量
---

## `let`命令
+ 所声明的变量只在其所处代码块内有效；
```
    {let a = 10};
    alert(a);  // ReferenceError: a is not defined
    
    {let a = 10;
     alert(a);} //10
```
+ 在`for`循环中，`for`关键字后面跟的设置循环变量语句和后面循环体内部语句是父作用域和子作用域的关系；
```
    var arr = [];
    for (let i = 0; i < 5; i++) {
      arr[i] = function () {
        let i = 10;
        console.log(i);
      };
    }
    arr[3](); // 10
    
    for (let i = 0; i < 5; i++) {
      arr[i] = function () {
        console.log(i);
      };
    }
    arr[3](); // 3
```
+ `let`声明的变量不会提升；
```
    for (let i = 0; i < 5; i++) {
      console.log(x + i);
      var x = 10; // NaN, 11, 12, 13, 14; 第一次循环变量未初始化
    }
    for (let i = 0; i < 5; i++) {
      console.log(x + i);
      let x = 10; // ReferenceError: x is not defined
    }
```
+ 在代码块内，`let`声明变量之前，该变量不可用，即使是在父作用域有同名变量也不行，形成“暂时性死区（TDZ）”
 ```
    var arr = [];
      for (let i = 0; i < 5; i++) {
        arr[i] = function () {
          console.log(i);
          let i = 10;
        };
      } 
    arr[3]();  //ReferenceError: i is not defined
    //换成var 就不会出错了：输出undefined.
    
 ```
 
+ `let`不允许在同一个作用域内，重复声明同一个变量；
```
    for (let i = 0; i < 5; i++) {
      let a = 3;
      let a = 2;      //Identifier 'a' has already been declared 
    } 
```

js中原来并没有块级作用域的概念，一方面使用`var`声明的变量存在提升现象，另一方面`var`在非函数作用域内声明的变量都是全局变量，在循环中可能造成混淆引用。

`let`命令一定程度上规避了这些问题，相当于在js中引入了块级作用域的概念。

## `const`声明一个只读的常量
+ 声明常量时必须立即初始化赋值；
+ 与`let`相似，`const`声明只在块级作用域内有效，不存在提升现象，也不允许在同一个作用域内重复声明。
+ `const`声明的基本类型值可以认为是常量不会改变，但如果声明一个引用类型值，其实际冻结的是这个引用（指针）而不是引用类型值本身。比如数组或对象，`const`声明指针后指针不能再改变，不能再给指针赋值，但可以在其指向的对象/数组本身加以操作。
+ 如果要冻结对象，使用`Object.freeze()`方法。
```
const a = Object.freeze({});
//下面操作不生效，严格模式会报错
a.prop = "foo";
```

在ES6中，声明变量由原来的`var`和`function`，增加到`var`/`function`/`let`/`const`/`import`/`class`六种；

## 变量的解构赋值
解构赋值可以以数组、对象和基本包装类型的对象等形式。
### 数组形式
+ 只要赋值语句的等号前后模式匹配，能够自动解构赋给对应变量对应的值，如果没有对应值则为`undefined`; 在数组中，`...foo`代表“剩下的值”组成的数组，必须放在最后；
```
    var [a, b, c, ...d] = [1, 2];
      console.log(a);  // 1
      console.log(b);  // 2
      console.log(c);  // undefined
      console.log(d);  // []
```

+ 如果等号左边是数组结构，只要等号后面的数据结构具有`iterator`接口，就能够按照数组解构赋值；否则会报错：
```
    let[a] = 1; //TypeError: 1 is not iterable
    let[b] = null; //TypeError: null is not iterable
    let[c] = undefined; //TypeError: undefined is not iterable
    let[d] = {}; //TypeError: {} is not iterable
```
+ 解构赋值也允许指定默认值，只有在赋值对应的位置严格等于`undefined`时默认值才会生效；如果默认值是表达式则只有在用到的时候才会求值；

### 对象形式
+ 解构赋值还可以用于对象：对象中的属性没有一定的顺序，所以在赋值时等号左、右要有完全一致的属性名才能取到值，否则就是`undefined`;
```
    let obj = {prop1:p1, prop2:p2} = {prop1:"first", prop2:"second"};
    // 属性名还是对象的属性名，变量被赋值；
      console.log(obj[prop1]);  // ReferenceError: prop1 is not defined
      console.log(obj["prop1"]); // "first"
      console.log(obj[p1]); // undefined
      console.log(p1); // "first"
    // 属性名同时也作为变量被赋值
    let obj = {prop1, prop2} = {prop1:"first", prop2:"second"};
      console.log(obj[prop1]);  // undefined
      console.log(prop1);  // "first"
      console.log(obj["prop1"]); // "first"
```
+ 可以转换为基本包装类型对象的基本类型的值，也可以用于解构赋值，包括数值、布尔值、字符串；（解构赋值的等号右边如果不是对象或数组，就先转换成对象，`null`和`undefined`不能转为对象，用于对象形式解构会报错）
+ 尽量不用圆括号，因为容易导致歧义；至少在赋值语句的模式部分不能使用圆括号；

## 变量解构赋值的用途
+ 变量值互换；
+ 函数返回的多个值可以一次取出赋给对应变量；
+ 函数参数定义和默认值；
+ 提取`JSON`数据；
+ 遍历`map`结构（`for...of`循环遍历配合解构赋值）；
+ 输入模块的指定方法；
+ 


***