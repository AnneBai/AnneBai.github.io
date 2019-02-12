---
layout: post
title: 内存管理和垃圾收集
---

# 内存管理和垃圾收集
## 基本类型和引用类型
在JavaScript中, 基本类型值是简单的数据段, 引用类型值是保存在内存中的对象. 

## 内存生命周期:
1. 分配所需内存
2. 使用分配到的内存读/写
3. 不需要时将内存释放/归还

JavaScript语言中,第一步和第三部都是隐含的. 一般在定义变量时就完成了内存分配, 有些函数的调用过程中或调用结果是分配对象内存/分配新变量或新对象. 


## 垃圾清理

高级语言的解释器嵌入了"垃圾回收器",它的主要工作是跟踪内存的分配和使用，以便当分配的内存不再使用时，自动释放它。原理比较简单: 找出不再继续使用的变量, 然后释放其占用的内存.(只能是一个近似的过程，因为要知道是否仍然需要某块内存是无法判定的, 即无法通过某种算法解决.) 垃圾收集器会按照固定的时间间隔周期性执行垃圾回收操作. 

垃圾回收的方式包括**引用计数**和**标记清除**;

**引用**的含义: 在内存管理的环境中, 一个对象如果有访问另一个对象的权限, 叫做一个对象引用另一个对象. 这里的对象包括数组或函数等所有引用类型值. 例如一个JavaScript对象具有对它原型的引用(隐式)和对它属性的引用(显式);

**引用计数**

引用计数是最初级的垃圾收集算法.

它的原理是跟踪记录每个变量被引用的次数, 没增加一个引用就给引用数加1, 反之减1; 当某个变量的引用次数是0, 就说明没有办法再获取到这个值, 那么垃圾清理时会将其回收. 但是这种方式有个缺点是无法处理循环引用, 导致有些内存无法回收, 例如:
```
function problem() {
    const objA = {
        a: 1,
    };
    const objB = {
        b: 2,
    };
    objA.dataB = objB;
    objB.dataA = objA;
}
```
`objA` 和`objB`通过各自的属性互相引用, 虽然`problem`函数执行完毕后这两个对象都离开了执行环境, 但它们的引用数都不是0, 而是2; 即使手动把 `objA`和`objB`重置为`null`(只是解除了这两个指针对其对象的引用), 这两个对象中也因为各子属性的互相引用, 引用数变为1而永远不会是0;这部分内存不会得到回收, 而如果该函数被多次调用, 每次都会新建这两个对象, 就会累积更多无法回收的内存.

目前最常用的"标记清除"方式则可以避免类似的问题, 因为在它的策略中, `objA`和`objB`所指的对象属于`problem`函数的局部变量, 函数执行时会创建它们, 一旦函数执行结束, 局部变量也会离开执行环境, 不会因循环引用而影响垃圾回收.

**标记清除**

标记清除的算法把"对象是否不再需要"简化为"对象是否可以访问";

这个算法假定设置一个叫做"root"的对象(JS中可以理解为全局对象). 从根开始引用的对象, 然后再找这些对象引用的对象...所有可以访问到的对象都会作为"将会继续使用的对象"保留, 而不能访问到的对象将会被当作"不会再使用的变量", 垃圾清理时便回收其内存.

> 当变量进入环境, 就将这个变量标记为"进入环境",当变量离开环境时, 则将其标记为"离开环境".  垃圾收集器在运行的时候会给存储在内存中的所有变量都打上标记, 然后去除环境中的变量和被环境中的变量引用的标记.在此之后依然有标记的则会被视为可以被删除了, 因为环境中的变量已经无法访问到这些变量了.最后垃圾收集器完成内存清除的工作, 销毁那些带标记的值并回收它们所占用的内存空间. (引自*JavaScript高级程序设计*p79)

2012年后所有现代浏览器都使用了标记清除垃圾回收算法.

**解除引用**

优化内存占用的最佳方式,就是为执行中的代码只保留必要的数据; 一旦数据不再有用, 可以将其值设置为`null`, 来使其脱离执行环境(以便下次清理垃圾时回收内存), 这适用于大多数全局变量和全局对象的属性, 局部变量会在它们离开执行环境时自动被解除引用.


主要参考资料:
+ *JavaScript高级程序设计(第三版)*
+ [MDN-内存管理](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)