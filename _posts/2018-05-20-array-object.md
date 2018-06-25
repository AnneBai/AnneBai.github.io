---
layout: post
title: ES6随笔--各数据类型的扩展（4）--数组和对象
---

## 数组的扩展
### 扩展运算符
形式为三个点`...数组`，用于把数组转为逗号分隔的参数序列；相当于`rest`参数的逆运算；
扩展运算符后面可以跟一个表达式；如果跟空数组则没有任何效果；

扩展运算符可以展开数组，可以替代`apply`把参数数组传给函数;
```
// 扩展运算符可以直接把数组拆为参数序列
      var a = [1,2,3],
          b = [4,5,6],
          c = [7,8,9];
      b.push(...a);
      console.log(b); //[4,5,6,1,2,3] 
      c.push(a);
      console.log(c); //[7,8,9,[1,2,3]]
```
扩展运算符可以用来复制数组而不是仅仅复制引用；
```
      const a1 = [1,2];
      let a2 = [...a1];
      let [...a3] = a1;
      console.log(a2); //[1,2]
      console.log(a3); //[1,2]
      console.log(a1 == a3); //false
      console.log(a2 === a3); //false
```
合并数组
```
// 合并数组
      var a = [1,2,3],
          b = [4,5,6],
          c = [7,8,9];
      console.log([...a, ...b, ...c]); //Array(9)
```
可以与解构赋值结合，用于生成数组；扩展运算符用于数组赋值时只能放在最后一位，否则会报错；  
```
const [...butLast, last] = [1, 2, 3, 4, 5];
// 报错
```
扩展运算符可以将字符串转为真正的数组，最大的优点是能够正确识别四个字节的Unicode字符；

任何 Iterator 接口的对象，都可以用扩展运算符转为真正的数组。

例如`Map`/`Set`结构、`Generator`函数返回的遍历器对象，都可以使用扩展运算符将其转变为数组；对于不能使用扩展运算符的类数组对象（具有`length`属性），可以使用`Array.from()`方法。
### Array.from()方法--转为数组，提供数组方法，绑定this对象
`Array.from`方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）。

Array.from还可以接受第二个参数，作用类似于数组的map方法，用来对每个元素进行处理，将处理后的值放入返回的数组。接收第三个参数，用于绑定`this`；
```
// Array.from可以接收第二个参数；类似于map
// 第二个参数方法中有this， 还可以在第三个参数中绑定this
      var a = {length:3, name:"obj1"}
      var b = Array.from(a);
      console.log(b); // [undefined, undefined, undefined]
      function fn() {
        return this.name;
      }
      var c = Array.from(a, fn, a);
      console.log(c); // ["obj1","obj1","obj1"]
```
```
// 返回各种数据的类型
function typesOf () {
  return Array.from(arguments, value => typeof value)
}
typesOf(null, [], NaN)
// ['object', 'object', 'number']
```
### Array.of方法，用于将一组数值转换为数组；
`Array.of()`方法可以用来代替`Array`或`new Array()`;总是返回参数值组成的数组，如果没有参数，就返回空数组；
### 数组实例的copyWithin()
```
Array.prototype.copyWithin(target, start = 0, end = this.length)
```
接收三个参数：

target（必需）：从该位置开始替换数据。如果为负值，表示倒数。

start（可选）：从该位置开始读取数据，默认为 0。如果为负值，表示倒数。

end（可选）：到该位置前停止读取数据，默认等于数组长度。如果为负值，表示倒数。

如果不是数值，会自动转为数值；

**该方法会修改原数组**；

### 数组实例的find()和findIndex();
数组实例的`find`方法，用于找出第一个符合条件的数组成员。它的参数是一个回调函数，所有数组成员依次执行该回调函数，直到找出第一个返回值为`true`的成员，然后返回该成员。如果没有符合条件的成员，则返回`undefined`。

数组实例的`findIndex`方法的用法与find方法非常类似，返回第一个符合条件的数组成员的位置，如果所有成员都不符合条件，则返回`-1`。

这两个方法都可以接受第二个参数，用来绑定回调函数的`this`对象。

### 数组实例的fill();
用于填充数组；
```
// fill方法
    // x的值在初始化时是随机的，但数组中的值都是一样的
      var x = Math.floor(Math.random()*100);
      var a = new Array(10).fill(x);
      console.log(a);
```
如果填充的类型是对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象；

### 数组实例的entries()/keys()/values()---用于遍历数组
`entries()`
    遍历键值对
`keys()`
    遍历键名
`values()`
    遍历键值
    
### 数组实例的includes();
该方法类似于字符串的同名方法；返回一个布尔值，表示是否包含给定值；

Map和Set数据结构有一个`has`方法，有差别需注意：

Map 结构的`has`方法，是用来查找键名的，比如`Map.prototype.has(key)`、`WeakMap.prototype.has(key)`、`Reflect.has(target, propertyKey)`。

Set 结构的`has`方法，是用来查找值的，比如`Set.prototype.has(value)`、`WeakSet.prototype.has(value)`。

### 数组的空位
ES6明确将空位转为`undefined`;
`Array.from()`和`...`扩展运算符将空位视为`undefined`;`fill()`会将空位视为正常的数组位置；`copyWithin()`会连空位一起复制；`for...of`循环也会遍历空位；

`entries()`/`keys()`/`values()`/`find()`/`findIndex()`会把空位视为`undefined`.
```
var a = [, , , , ,];
console.log(a.length); //5
console.log(a[1]); // undefinesd
```


## 对象的扩展

### 属性的简洁表示法
ES6允许直接写入变量和函数，作为对象的属性和方法；
```
    var name = "Anne",
        age = 25;
    const person1 = {
        name:name,
        age:age,
        sayMe () {
          alert("My name is "+ this.name + ", " + this.age+" years old.")
        }
      }
    person1.sayMe();
```

```
module.exports = { getItem, setItem, clear };
// 等同于
module.exports = {
  getItem: getItem,
  setItem: setItem,
  clear: clear
};
```
简写写法的属性名总是字符串；如果方法是一个Generator函数，前面需要添加星号；
### 属性名表达式
```
// 允许使用表达式作为属性名，需放在方括号内；
      let a = "Bt";
      let person1 = {
        [a+"name"]:"Betty",
        [a+"age"]:26
      }
      alert(person1.Btname); // Betty
      alert(person1["Btage"]); // 26
```

### 方法的name属性
正常对象的方法，返回方法名；

使用了getter和setter函数的方法，`name`属性是其属性的描述对象的`get`和`set`属性上；

`bind`函数返回的函数，`name`属性前面加`bound`；

`Function`构造函数创造的方法，`name`属性返回`anonymous`

`Symbol`值，返回其描述
### Object.is()
行为与`===`基本一致，不同之处在于：`+0`与`-0`不相等；`NaN`等于自身；
```
// Object.is()  
      console.log(Object.is(+0, -0))   // false
      console.log(Object.is(NaN, NaN)); // true
      console.log(Object.is("1", 1)); //false
```
### Object.assign()
将源对象的所有可枚举属性，复制给目标对象；第一个参数为目标对象，后面所有参数都是源对象；如有同名属性，后面属性会覆盖前面属性。

如果只有一个参数，会返回该对象，不是对象时转换成对象，`undefined`、`null`会报错；

多个参数时，首参数是字符串/`null`/`undefined`会报错，对象/数值/布尔值/数组不会报错；后面的参数中，如果是字符串或数组会以数组形式拷贝入对象；
```
// Object.assign()
      let a = Object.assign(2, "abc", true, [3, 4, 5]);
      console.log(a); // Number {2, 0:3, 1:4, 2:5}
      console.log(typeof a); // object
      console.log(a.length); // undefined
      console.log(a+3); // 5
      console.log(a instanceof Number); // true
```
```
// 字符串和数组都可以拷贝入对象；
let a = Object.assign(2, ["a", "b", "c", "d"], true,"efg");
      console.log(a); // Number {2,0: "e", 1: "f", 2: "g", 3: "d"}
```
`Object.assign()`只拷贝源对象的自身属性，不拷贝继承属性和不可枚举属性；对于对象执行的是浅拷贝，如果某个属性的值是对象，拷贝其引用而非副本；
```
// 浅拷贝
      var b = {};
      var c = {"name1":{first:"xiaoa"}}
      var a = Object.assign(b, c);
      console.log(b.name1); //{first:"xiaoa"}
      c.name1.first = "daa"; 
      console.log(b.name1); //{first:"daa"}
      console.log(a.name1); //{first:"daa"}
// 这种方式不会更改
      c.name1 ={first:"thaa"}; 
      console.log(b.name1); //{first:"daa"}
      console.log(a.name1); //{first:"daa"}
```
克隆对象，如果想要保留继承链，可以用以下方法：
```
function clone(origin) {
    let originProto = Object.getprototypeOf(origin);
    return Object.assign(Object.create(originProto), origin);
}
```
### 属性的可枚举性和遍历

`enumerable`是对象的*属性的属性*，存在于对象每个属性的描述对象中，可以通过`Object.getOwnPropertyDescriptor()`方法获取该描述对象；`enumerable`为`false`的属性表示不可枚举，在以下操作中会被忽略：
+ `for...in`  --遍历对象自身的和继承的可枚举属性
+ `Object.keys()`  --返回对象自身所有可枚举属性的键名
+ `JSON.stringify()`  --序列化对象自身的可枚举属性
+ `Object.assign()`  --只拷贝对象自身的可枚举属性

此外，ES6规定，所有Class的原型方法都是不可枚举的。

遍历对象属性的5种方法

1. `for...in`  --遍历；对象自身/继承\+可枚举\+非`Symbol`
2. `Object.keys(obj)` --返回键名数组；对象自身\+可枚举\+非`Symbol`
3. `Object.getOwnPropertyNames(obj)`  --返回键名数组；对象自身\+非`Symbol`
4. `Object.getOwnPropertySymbols(obj)` --返回键名数组；`Symbol`
5. `Reflect.ownKeys(obj)` --返回键名数组；对象自身

次序规则：

数值键(数值升序)--> 字符串键(加入时间升序)--> Symbol键(加入时间升序)

### Object.getOwnPropertyDescriptors()
返回指定对象所有自身属性（非继承属性的描述对象）；


### __proto__属性, Object.setPrototypeOf(), Object.getPrototypeOf()
`__proto__`属性指向对象的原型；看作是内部属性；

对原型的读写操作可以分别用`Object.getPropertyOf()`和`Object.setPropertyOf()`；

`Object.setPropertyOf(object, prototype)`相当于：
```
function (obj, proto) {
    obj.__proto__ = proto;
    return obj;
}
```
如果第一个参数不是对象就自动转为对象；如果第一个参数是`undefined`或`null`则会报错；
### super 关键字
指向当前对象的原型对象；只能用在对象的方法中，且需使用对象方法简写法；
```
      const proto = {
        name:"river",
        method: function () {
          alert(this.name);
        }
      }
      const obj = {
        name:"rock",
        method1 () {
          super.method();
        }
      }
      const obj1 = {
        name:"fish"
      }
      Object.setPrototypeOf(obj, proto);
      Object.setPrototypeOf(obj1, proto);
      obj.method1();  // "rock"
      obj1.method();  // "fish"
```
### Object.keys(), Object.values(), Object.entries()
可以用在`for...of`循环中，也可以直接获取键名/键值/键值对;`Object.entries()`还可以用来将对象转换为真正的Map结构。
```
      let {keys,values,entries} = Object;
      for (let key of keys(obj)) {
        console.log(key); 
      }
      // name, method1
      console.log(Object.values(obj));  // ["rock", ƒ]
      console.log(Object.entries(obj));  // [["name", "rock"], ["method1", ƒ]]
```
### 对象的扩展运算符
+ 可以用于解构赋值中最后一个参数，拷贝剩下的未遍历的属性；
+ 等号右侧不能是`null`或`undefined`；
+ 解构赋值是浅拷贝，也不会拷贝继承自原型对象的属性；
扩展运算符的应用：
+ 复制对象--取出参数对象所有可遍历属性，拷贝到当前对象中；
```
let aClone = { ...a };
// 等同于
let aClone = Object.assign({}, a);
```
+ 既拷贝对象也拷贝对象的原型属性
```
      const clone1 = Object.assign(
        Object.getPrototypeOf(obj),
        obj
      );
      const clone2 = Object.create(
        Object.getPrototypeOf(obj),
        Object.getOwnPropertyDescriptors(obj)
      );
```
+ 合并两个对象
```
let ab = {...a, ...b};
//等同于
let ab = Object.assign({}, a, b);
```
+ 复制一个对象并修改部分属性
```
      const a = {name:"a", number:1};
      const b = {...a, name:"b"};
      alert(b.name); // b
```

***
2018-5-20