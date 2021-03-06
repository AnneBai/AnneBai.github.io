---
layout: post
title: JS--数组方法的总结
---

数组(Array)和对象(Object)几乎是很多程序语言中最常用的类型。在ECMAScript中，数组的长度可以动态变化，数组中的数据可以是不同类型，相比其他语言更加灵活。另外，ECMAScript数组原生支持很多实用的方法，给数据的保存和处理带来很大的方便。

由于数组是引用类型，需要注意方法的可变性，简单理解就是“是否会改变原数组”。这对于函数式编程尤其重要，因为可变方法可能会产生我们调用它的目的之外的副作用，导致一些不可预知的结果，更容易造成bug且给bug的定位增加了难度。

这里把数组的常用方法总结一下。由于是个人总结，如果有差错的地方还望大家及时指出。

tips:
1. 示例代码可以直接打开浏览器console进行运行和实验；
2. 为阅读方便，方法介绍时会用如`sort(?compareFn)`表示函数名和参数列表，参数前有`?`代表是可选参数。
3. 查看浏览器中实现的所有数组方法，可以直接在console中执行`console.dir(Array)`或`console.log("%O", Array)`, 能看到Array上的静态方法和其prototype上的方法列表；
## 静态方法
类似于ES6中在class类中定义的static的方法，不会被实例继承，只能通过类本身来访问：
### Array.isArray
这个方法用于检验传入值是否是数组，与`instanceof`相比，具有“跨iframe”的优点；因为一个浏览器中的多个window是不共享全局对象的，所以通过全局变量直接访问的Array也不一定指向同一个Array构造函数；而当执行`value instanceof Array`时，这里的Array不一定是创建value时所在全局对象下的Array, 可能会返回错误的信息。

```javascript
// 插入一个新iframe
var iframe = document.createElement('iframe');
document.body.appendChild(iframe);
var a = new window.frames[0].Array(2); // 在iframe中创建一个长度为2的数组

// 两个window的Array是不同的对象(构造函数)
a instanceof window.frames[0].Array // true
a instanceof window.Array // false

window.frames[0].Array.isArray(a); // true
window.Array.isArray(a) // true

// 因为不同window下的全局属性是不同的引用
window.frames[0].Array === window.Array // false
```
### Array.from, Array.of

这两个方法都可以用来创建新的数组实例
+ `Array.from(arrayLike, ?mapfn, ?thisArg)`接收3个参数，第一个是类数组或可迭代对象，后两个是可选的回调函数`mapFn`(`map`的回调函数)和`thisArg`(设置this)，相当于把可迭代对象转为数组后对其执行`map(mapFn, thisArg)`; 可迭代对象中如果有“空元素”(empty), 会处理为`undefined`返回；

```javascript
Array.from(new Array(3)); // [undefined, undefined, undefined]
Array.from("abcdefg", (s, i) => s + i); // ["a0", "b1", "c2", "d3", "e4", "f5", "g6"]
```

+ `Array.of(...arg)`接收1个或多个参数，会把参数列表按顺序作为新数组的元素，并返回该新数组；

```javascript
Array.of("a", 1, "b", 2); // ["a", 1, "b", 2]
```
## 不可变(immutable)方法

不可变方法不会影响原数组，对返回的数组本身进行的操作也和原数组无关；但需要区别的一种情况是，由于元素复制时都是浅复制，新数组中引用类型值的元素与原数组元素引用的是同一个对象，修改会相互影响，同时影响其他引用该对象的所有变量。

### map, filter, forEach
这三个方法属于ECMAScript定义的迭代方法，可以对数组中每个元素执行一定操作后返回一定的结果；它们都可以接收两个参数`(callbackfn, ?thisArg)`，第一个是要在每个元素上执行的回调函数，第二个参数是可选的，即运行该函数的作用域对象(指定this值)；需要注意的是如果使用箭头函数作为回调函数，this值是创建时绑定的，不会被第二个参数影响。
```javascript
// 第二个参数对this值的影响
[1, 2, 3].map(function (n) {
    return this.name;
}, {name: "Anne"})
// ["Anne", "Anne", "Anne"]

// 箭头函数回调, 创建时绑定了当前所在执行环境的this--> Window作为固定的this的值
[1, 2, 3].map(n => this, {name: "Anne"}) // [Window, Window, Window]
```
传入的回调函数会接收到三个参数`(item, index, array)`，即当前元素、当前索引和源数组；一般使用最多的是前两个。

+ `map`对每个元素执行该回调后，将回调函数返回值组成新的数组返回，用于对元素成员转换或取值；

```javascript
[1,2,3,4,5].map((n, i) => n + i); // [1, 3, 5, 7, 9]
```

+ `filter`是将执行回调函数后返回的是truthy值的元素保留组成新数组返回，通常用于数组的过滤；

```javascript
[1, 2, 3, 4, 5].filter((n, i) => n % 2 === 0); // [2, 4]
```

+ `forEach`则只是对每个元素执行回调函数的操作，没有返回值。

```javascript
let a = [1, 2, 3, 4, 5];
let b = [];
a.forEach((n, i) => b[i] = n + i) // (返回)undefined
a // (未改变)[1, 2, 3, 4, 5]
b // [1, 3, 5, 7, 9]
```
### some, every

这两个方法也属于迭代方法，同样接收一个回调函数参数和一个可选的作用域对象参数，回调函数会接收`(item, index, array)`作为参数并需要返回布尔类型值，作为每个元素是否符合条件的判断依据；

与上面方法不同的是它们返回的是布尔值；从字面可以看出，`some`代表的是“只要有符合条件的元素就返回true”而`every`则是“所有元素都符合条件才返回true”.所以:
```javascript
[1, 2, 3, 4, 5].some((n) => n % 2 === 0) // true
[1, 2, 3, 4, 5].every((n) => n % 2 === 0) // false
```
它们不一定会遍历所有的元素，当`some`遇到符合条件的元素或`every`遇到不符合条件的元素它们就会停止遍历直接返回结果，因为后面的遍历不再必要；
```javascript
// 输出true之前 执行了3次
[1, 2, 3, 4, 5].some(n => {
    console.count("some");
    return n === 3;
});

// 输出false之前 执行了1次
[1, 2, 3, 4, 5].every(n => {
    console.count("every");
    return n === 3;
});

```
### reduce, reduceRight
我觉得`reduce`函数*值得*是数组方法中被运用最多的方法之一(还有`map`和`filter`)。初学JavaScript时我对它认知较浅，只有在遇到类似书中所举的“数组求和”问题时才会想到它。但现在认识到它其实比我想象的能做更多事(本质还是一样的)，我写在另一篇总结里(还没写完)。

`reduce`和`reduceRight`是ES5中增加的数组归并方法。`reduce`会从第一项到最后一项遍历数组所有元素，构建一个最终返回的值(取决于回调函数)；`reduceRight`和`reduce`一样，只是遍历方向相反，从最后一项开始到第一项进行归并操作。

它们接收两个参数`(callbackfn, ?initialValue)`，第一个是在每一项上调用的回调函数，第二个是可选参数，用于设置初始值；例如书中的例子：
```javascript
[1, 2, 3, 4, 5].reduce((prev, cur) => prev + cur); // (数组元素的和)15
```
在每一项上调用的回调函数可以接收到四个参数，即`(accumulator, currentValue, currentIndex, sourceArray)`;

+ accumulator: 可理解为累积器，每次执行回调函数后的返回值，传入下一项中作为此参数；在`reduce`初始值(第二个参数)没有设定时，执行时会默认把数组中第一个元素作为这个参数直接在第二个元素上执行；如果传入了初始值，则先在第一个元素上执行，初始值作为回调的该参数。
+ currentValue: 当前元素
+ currentIndex: 当前索引
+ sourceArray: 源数组

可以验证，没有设定初始值时，执行回调函数的次数比元素个数少一个；而设定初始值时执行次数与元素个数相同。因为有初始值时遍历从第一个元素开始。
```javascript
[1, 2, 3, 4, 5].reduce((sum, cur) => {
    console.count("reduce-no-initail");
    return sum + cur;
});
// 输出结果 15 前，"reduce-no-initail"打印了4次

[1, 2, 3, 4, 5].reduce((sum, cur) => {
    console.count("reduce-initail");
    return sum + cur;
}, 0);
// 输出结果 15 前，"reduce-initail"打印了5次
```

### concat, slice
这两个方法不传入参数时都会简单浅复制已知数组并返回这个副本，所有也常用于复制数组或类数组/可迭代对象(通过`Array.prototype.concat.call(someObj)`或`[].concat.call(someObj)`的方式).

+ `concat(...items)`用于对数组副本的拼接和合并，接收0或多个参数，不传入参数时会返回将原数组浅复制后的新数组；传入1个或多个参数时，会在浅复制一份原数组的基础上，将每个参数(如果参数是数组则按序取出其中的元素，否则直接取该参数)作为元素按顺序拼接在其后；相当于直接将参数合并后执行了一次`flat()`再与原函数合并。

```javascript
[[0], 1].concat(2, [3, 4], [[5, 6], 7]); // [[0], 1, 2, 3, 4, [5, 6], 7]
```

+ `slice(?start, ?end)`用于返回数组的某一部分的副本，接收2个可选参数，代表起始索引和结束索引(左闭右开)， 不传参数的情况与`concat`相似，返回将原数组浅复制的新数组；传入一个参数则默认从该参数位置到数组末尾； 传入的负值参数会取绝对值后从后向前数，例如`-1`会被解释为倒数第一个元素的位置(其他数组方法对代表索引的负数参数的处理都与此相同)。

```javascript
[1,2,3,4].slice(2) // [3, 4]
[1,2,3,4].slice(2, -1) // [3]
```

### find, findIndex, indexOf, lastIndexOf, includes
这几个方法的相似之处都是执行对数组的查找操作；不同之处在于：
+ `find(predicate, ?thisArg)`和`findIndex(predicate, ?thisArg)`接收一个回调函数作为查找标准，该函数接收每个迭代元素的`(item, index, array)`参数，一旦执行后返回值为truthy则视为找到该元素，`find`将会返回该元素(或其引用)而`findIndex`返回该元素的索引，并停止查找；它们还可以接收第二个可选参数用于绑定`this`所指的对象；

```javascript
[
    {name: "a", val: 1},
    {name: "b", val: 2},
    {name: "c", val: 1}
].find(item => item.val === 1)
// {name: "a", val: 1}

function getVal2(o) {
    return o.val === this.val;
}

[
    {name: "a", val: 1},
    {name: "b", val: 2},
].find(getVal2, {name: "d", val: 2})
// {name: "b", val: 2}

```

+ `indexOf(searchElement, ?fromIndex)`和`lastIndexOf(searchElement, ?fromIndex)`接收的第一个参数是一个要查找的元素，并在迭代数组元素时使用`===`来判断是否是查找的元素，如果是则返回该元素的索引，如果最后都没有找到则返回`-1`; `lastIndexOf`和`indexOf`一样，只是是从数组末尾开始向前查找；它们可以接收第二个参数，用于设定从哪个位置开始查找；

```javascript
[NaN, +0, -0].indexOf(NaN) // -1
[NaN, -0].indexOf(0) // 1
```

+ `includes(searchElement, ?fromIndex)`的参数与`indexOf`相似，也是一个要查找的值，和一个可选的“起始位置”；但不同的是这个方法在对比元素时使用`sameValueZero`的判断方式，即`NaN`与`NaN`视为相等，其他与`===`判断相同。

```javascript
[NaN, -0].includes(NaN) // true
[NaN, -0].includes(0) // true

```

#### *关于`===`和'sameValueZero'相等性判断*

+ `===`不进行类型转换，直接对比值，如果是引用类型值则对比其是否指向同一个对象；`NaN`不等于自身，`0`与`+0`、`-0`互相相等；

+ 'sameValueZero'判断时除了对NaN处理为其与自身相等，其他均与`===`一样；

+ 另外ES6中新增的`Object.is()`，则在sameValueZero的基础上，增加了对`0`的符号的限制，用它来判断时`0`等于`+0`, 但它们不等于`-0`。

### flat, flatMap
`flat(?depth)`用于铺平数组，可以接收一个参数设定要铺平的深度(层数)， 默认为1，如果传入的深度值比数组本身的深度大，则与传入数组的最大深度效果相同， 数组被完全铺平成为一维数组。
```javascript
let a = [1, [2, [3, [4, 5], 6], 7], [8], 9];
a.flat() // [1, 2, [3, [4, 5], 6], 7, 8, 9]
a.flat(2) // [1, 2, 3, [4, 5], 6, 7, 8, 9]
a.flat(10) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
a.flat(Infinity) // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```
`flatMap(callback, ?thisArg)`则是`map`方法与`flat`的方法的结合体，接收一个回调函数和一个可选的this参数，作为第一步`map`的参数对每个元素执行并返回新的值，然后对构建的新数组进行展平1层。它与我们自己调用`map`方法后再调用`flat`方法效果相同，但效率可能更高一点点。

比较常见的情况如从对象数组中取出某些值为数组的属性值，然后希望变成一个一维数组方便执行其他操作，就可以用这个方法；
```javascript
let b = [
    {memberIds: [1,2,3]},
    {memberIds: [4,5,6]},
    {memberIds: [7,8,9,10]}
];
// 获取b中所有memberId组成一个一维数组
b.flatMap(obj => obj.memberIds) // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
// 和下面的方法结果一样
b.map(obj => obj.memberIds).flat() // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```
## 可变(mutable)方法

这里可变方法是指会改变数组本身的方法。并不是说可变方法不能用，它们有时候可能非常有用。如果开发人员自己清楚使用它们的目的和结果并只在需要的时候使用，有利于提高代码的可维护性和健壮性。

### fill, copyWithin, splice
这三个方法的相似性不是很高，但使用目的有一定的相似，即将数组中某些元素改变为我们希望的值，甚至插入/删除一些值。

+ `fill(value, ?start, ?end)`是对已有的数组填充，它的作用范围仅限于当前数组长度之内，不能改变数组长度。它接收三个参数，第一个是需要填充的值，第二个和第三个是可选的位置参数，设定填充的起始索引(含)和结束索引(不含)，如果不传则分别默认为0和数组的length。对负数位置参数的处理与上面提到的相同；如果超出了数组本身，则转为与它们最近的有效值(0 或 array.length)。执行结束返回改变后的数组。例如：

```javascript
[1,2,3,4].fill(5) // [5,5,5,5]
[1,2,3,4].fill(5, 2, 4) // [1,2,5,5]
[1,2,3,4].fill(5, -2, 4) // [1,2,5,5]
[1,2,3,4].fill(5, -3, 4) // [1,5,5,5]
[1,2,3,4].fill(5, -5, 4) // [5,5,5,5]
```

+ `copyWithin(target, start, ?end)`在数组内部复制一部分值到另一部分，也不能改变数组长度。它接收第一个参数是要放置复制元素的目标位置索引，第二个参数是要复制的部分的起始索引(含)，结束索引(不含)便是可选的第三个参数，如果没有传入则默认为函数长度，即从起始一直复制到末尾。复制的部分将从目标位置开始填充，覆盖对应位置原有的元素。执行结束后返回改变后的数组。

```javascript
[1,2,3,4].copyWithin(2, 0, 2) // [1,2,1,2]
[1,2,3,4].copyWithin(2, 0, 4) // [1,2,1,2]
[1,2,3,4].copyWithin(2, 4, 4) // [1,2,3,4] (没有复制到元素，也不会对原来的数组有影响)
```

+ `splice(start, ?deleteCount, ...items)`几乎可以算这些方法中最强大的方法了。它可以对数组任意位置执行插入、删除、替换操作，也可以改变数组长度。就像对数组做手术，而具体会做什么样的手术(执行什么操作)则完全由参数决定。它可以接收1个或多个参数，第一个设置起始位置，第二个为可选的“删除数量”，设置从起始位置(含)开始删除多少个元素，如果没有传入则默认将起始位置及其之后的元素全部删除。如果设置为0则不删除元素，并将其后的参数列表按顺序都从起始位置开始插入数组中。最后返回的是被删除的元素组成的数组。

```javascript
let a = [1,2,3,4,5,6];
a.splice(4) // [5, 6] 删除了5,6

a // [1,2,3,4] 数组a本身长度改变

a.splice(2, 0, 7, 8, 9, 0); // [] 没有删除元素
a // [1,2,7,8,9,0,3,4] // 7，8，9，0被作为插入元素从索引2开始插入，数组原来的元素被放到插入元素列表的后面
```

### push, shift, pop, unshift
这几个方法都用于对数组的头部或尾部进行插入和删除。`push`(末尾推入)和`pop`(末尾删除)操作在数组末尾，像使用栈一样使用数组，遵循“后进先出”的规则；`shift`(头部删除)和`unshift`(头部推入)作用于数组头部，结合使用`shift`和`push`或`unshift`和`pop`可以从正向和反向模拟队列行为，像使用队列一样使用数组，遵循“先进先出”的规则。

+ `push(...items)`方法可以接收任意多个参数，把它们按序依次添加到数组末尾，返回改变后的数组的长度；
+ `pop()`方法不接收参数，每次执行都会从数组中删除掉最后一项，并返回这个元素；
+ `unshift(...items)`方法与`push`的方向相反，把接收到的任意多个参数放在数组头部，返回改变后的数组长度；
+ `shift()`方法也不接收参数，每次执行都会从数组头部删除掉第一项并返回这个元素；

这几个方法也可以应用在类数组对象上，或者有`length`属性和数值字符串属性的对象，它们会根据`length`属性确定数组的末尾位置并访问对应位置，而与对象实际存在的元素个数或其他属性无关。

```javascript
var a = [];
a.push(1,2,3) // 3 (添加元素后的数组长度)
a // [1,2,3]
a.pop() // 3 (删除的尾部元素)
a // [1,2]
a.unshift(5,6,7,8); // 6 (添加元素后的数组长度)
a // [5,6,7,8,1,2]
a.shift() // 5 (删除的头部元素)
a // [6,7,8,1,2]
```

### reverse, sort

这两个是数组的重排序方法；
+ `reverse()`直接按元素的位置进行反序操作，并返回改变后的数组；

```javascript
[4,14,3,23].reverse() // [23,3,14,4]
[4,14,3,23].sort() // [14,23,3,4]
```

+ `sort(?compareFn)`方法则是默认根据对比元素的字符串表示的先后顺序升序排列--即使每个元素都是数值，也会先把它们转换为字符串，然后按照字符串的对比规则(对比它们的UTF-16字符编码值)确定排序关系。如果数组中有`undefined`，则它们不参与排序并被放置在最后。
 
一般情况下我们更多需要的是对一组数值或拥有数值类型属性的对象进行排序，直接调用`sort`是无法满足的，需要自己传入一个“比较函数”，接收两个值(a, b)作为参数并返回一个数值，如果返回负数则a排在b之前，如果返回正数则相反，如果返回是0, 则一般将这两个值保持原来的先后顺序一起与其他值按序排列。ECMAScript中没有保证对比时返回`0`的两个值一定保持先后顺序，所以并非所有的浏览器都能保证做到这一点([引用](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort))。

```javascript
[4,14,3,23].sort() // [14,23,3,4] 按数值转换为字符串后的字符编码排序而非数值本身
[4,14,3,23].sort((a, b) => a - b) // [3,4,14,23] 按数值的大小进行升序排列

let a = ["x", "u", "m", "a"];
[undefined, ...a, "undefined"].sort() // ["a", "m", "u", "undefined", "x", undefined]

```

## 可变方法的不可变替代

### 使用不可变方法复制数组
例如`slice`, `concat`, `map`, `filter`等，根据不同需要选择代替；
```javascript
// concat 代替 push / unshift
let a = [1,2,3];
a.push(4);
a // [1,2,3,4];

let b = a.concat(5);
b // [1,2,3,4,5]
a // [1,2,3,4]

// slice 代替 pop / shift
b.pop();
b // [1,2,3,4]
let c = b.slice(0, -1);
c // [1,2,3]
b // [1,2,3,4]
```

### 使用扩展操作符复制数组
扩展运算符可以方便地对数组进行复制或部分复制，不会改变原数组；
```javascript
let a = [1,2,3,4];
let b = [..a, 5];
b // [1,2,3,4,5]
a // [1,2,3,4]
```
但需要注意的是，不论是不可变方法还是扩展运算符，数组的复制都是浅复制，对于引用类型的元素复制的是其引用，而非整个对象。

## 注意：不可变方法隐蔽下的可变操作
还有一点值得注意，虽然不可变方法本身不会改变原数组，但是因为数组本身是引用类型的值，如果在回调函数中引用数组本身并对其元素进行改变操作或重新赋值，还是会“隐蔽地”修改原数组。这种做法应该尽量避免，因为在后续维护时可能会给别人带来不必要的困扰(不知在哪里莫名其妙值就被改变了)。例如：
```javascript
let c = [9, 8, 7, 6, 5];
c.map((n, i) => c[i] = i);
// 此时c已变成[0, 1, 2, 3, 4]
```
当回调函数非常长的时候这种问题更难定位，其他引用`c`值的变量很有可能也同时被影响。所以每个函数最好都目的明确只做一件事，把确实需要改变原引用值的操作放在一个专门的函数中操作，而不是散布在任何看起来不会发生这种改变的地方。

参考
+ [Array-JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)
