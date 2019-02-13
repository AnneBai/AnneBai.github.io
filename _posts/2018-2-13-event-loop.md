## 单线程JavaScript
JavaScript本身是单线程的，也就是同一个时间只能做一件事，就像一个餐馆里只有一个店小二，每次只能服务一位或一桌客人。

在用户与浏览器交互的时候，脚本的执行只会是单个线程，这样不会出现复杂的同步问题。

但是如果在单线程中完全同步执行所有代码将会造成时间和资源的浪费, 因为很多耗时过程不得不等待结束; 假如餐馆里唯一的店小二要等一个客人吃完饭结账送客之后才继续服务下一个客人, 那这生意是没法儿做了...

幸运的是, 单线程的JavaScript支持异步操作, 也可以理解为只要需要, 线程会保持工作而不会阻塞在某个等待过程中.

## 事件循环和任务队列
事件循环在HTML的规范中定义，主要目的在于统一用户交互/网络/显示网页方面的程序行为。

事件循环与浏览上下文关联, 生命周期也与其浏览上下文直接相关; 一个浏览上下文总有一个事件循环来协调它的行为.

一个事件循环有一或多个任务队列(task queue)，作为待办任务的有序列表; 通常包括：

+ 事件处理函数
+ HTML 解析
+ 回调函数
+ 请求资源的响应回调
+ DOM操作

> 特定任务源产生的任务形成特定的事件循环, 每个事件循环会维护一个"当前执行任务"和"执行微任务队列检查标识"(布尔值, 初始为false); 浏览器对不同任务源的任务队列可能分配一定比例的时间, 执行任务时可以从任意任务队列获取任务, 但对每个队列来说一定是按"先进先出"的顺序;

执行任务的主线程是执行栈，同步任务在主线程中执行；异步任务会放到任务队列中等待；一旦执行结束，主线程栈已清空，系统会读取任务队列中排在队首的任务进栈执行。直到主线程再次清空，检查任务队列，如果有可执行的任务则执行，继续重复刚才的步骤。

这里的“任务队列”和现实世界的队列有相似的规则：“先进先出”；同样都在可执行的状态下，排在前面的任务会更早被处理；

## 同步和异步
JavaScript的任务可以分为同步和异步两种。

同步是指必须执行结束才能继续下一个任务，异步则是先“挂起”，放到一个“待办列表”（即任务队列）中，把当下手中紧急的事情忙完再拿过它来执行。

至于“挂起”到什么时候才可以执行, 这里涉及到任务队列中的宏任务和微任务。


## 宏任务/微任务
在浏览器上下文中，任务队列有宏任务(Macro Task)和微任务(Micro Task)的区分；来自不同任务源的事件/任务会被分配到不同的任务队列中，它们的处理顺序也有一定差异；

一般宏任务包括：
+ setTimeout/setInterval
+ DOM交互事件
+ Ajax回调函数

一般微任务包括：
+ promise的回调

在浏览器的事件循环模型中，比较关键的过程为：
1. 检查macrotask队列，运行最前面的一个任务，如果为空则进入下一步；
2. 检查microtask队列，按顺序执行任务将其清空；
3. 更新页面渲染；
4. 重复以上步骤；

简单的理解就是, 每执行完一个宏任务, 就要把在宏任务执行过程中产生的微任务队列清空, 然后再执行下一个宏任务. 如果在执行微任务的过程中产生新的微任务, 则会继续执行直到将其清空.

以上还有一点, 第3步"更新页面渲染"过程中包括执行`requestAnimationFrame`的回调函数列表. 个人目前的理解是: 在Event Loop中, Animation Frame的回调函数列表类似于宏任务队列, 是另一个任务队列(由任务源区分), 在 在更新页面渲染阶段会执行. `requestAnimationFrame`相比`setTimeout`函数在浏览器中得到更好的动画性能优化, 一般的更新频率是60Hz, 但浏览器会根据实际情况(例如资源限制, 是否是当前活动窗口等)进行合并和优化. 


#### 例1:

```
setTimeout(function fn1() {
    console.log("timeout")
    Promise.resolve().then(function fn2() {
        console.log("promise");
        setTimeout(function fn3() {
            console.log("promise-timeout")
        })
    })
});
Promise.resolve().then(function fn4() {
    console.log("promise1");
    setTimeout(function fn5() {
        console.log("promise1-timeout")
    });
    Promise.resolve().then(function fn6() {
        console.log("promise1-promise2");
        setTimeout(function fn7() {
            console.log("promise1-promise2-timeout")
        });
        Promise.resolve().then(function fn8() {
            console.log("promise1-promise2-promise3");
            setTimeout(function fn8() {
                console.log("promise1-promise2-promise3-timeout")
            })
        })
    });
});

/**
 * promise1, promise1-promise2, promise1-promise2-promise3
 * timeout, promise,
 * promise1-timeout,
 * promise1-promise2-timeout,
 * promise1-promise2-promise3-timeout,
 * promise-timeout,
 */
```

*一*

1. fn1 加入宏任务队列, fn4 加入微任务队列, 当前任务(主程序)执行完毕;
2. 检查微任务队列并清空: 执行fn4, 打印"promise1";
3. fn5 加入宏任务队列, fn6 加入微任务队列, 当前任务(fn4)执行完毕;
4. 此时微任务队列又有fn6, 继续执行fn6, 打印"promise1-promise2";
5. fn7 加入宏任务队列, fn8 加入微任务队列, 当前任务(fn6)执行完毕;
6. 此时微任务队列又有fn8, 继续执行fn8, 打印"promise1-promise2-promise3";
7. fn8 加入宏任务队列, 当前任务(fn8)执行完毕;
8. 微任务队列已清空, 进入下一个循环;

*二*

1. 当前宏任务队列为fn1, fn5, fn7, fn8, 取最早的任务fn1执行, 打印"timeout";
2. fn2 加入微任务队列; 当前任务(fn1)执行完毕;
3. 检查微任务队列并清空: 执行fn2, 打印"promise";
4. fn3 加入宏任务队列, 当前任务(fn2)执行完毕;
5. 微任务队列已清空, 进入下一个循环;

*三*

1. 当前宏任务队列为fn5, fn7, fn8, fn3, 取最早的任务fn5执行, 打印"promise1-timeout", 当前任务(fn5)执行完毕;
2. 微任务队列为空, 进入下一个循环;

*四*

1. 当前宏任务队列为fn7, fn8, fn3, 取最早的任务fn7执行, 打印"promise1-promise2-timeout", 当前任务(fn7)执行完毕;
2. 微任务队列为空, 进入下一个循环;

*五*

1. 当前宏任务队列为fn8, fn3, 取最早的任务fn8执行, 打印"promise1-promise2-promise3-timeout", 当前任务(fn5)执行完毕;
2. 微任务队列为空, 进入下一个循环;

*六*

1. 当前宏任务队列为fn3, 取最早的任务fn3执行, 打印"promise-timeout", 当前任务(fn3)执行完毕;
2. 微任务队列为空, 进入下一个循环;

#### 例2：

```
setTimeout(() => { // fn1
    console.log('fn1');
    Promise.resolve().then(res => { // fn2
      console.log('fn2');
    })
}, 100)
let i = 0;
function func() { // fn3
    console.log(i)
    Promise.resolve().then(res => console.log('fn4-' + i))  // fn4
    setTimeout(() => { // fn5
        console.log('fn5-' + i);
        Promise.resolve(3).then(res => { // fn6
            console.log('fn6-' + i);
        });
        if (++i < 2) {
            func();
        }
    })
}
setTimeout(() => { // fn7
    console.log("fn7")
    func();
}, 0);


// fn7 
// 0 fn4-0
// fn5-0 1 fn6-1 fn4-1
// fn5-1 fn6-2
// fn1 fn2
```
打印数字标识函数中的同步代码;
打印`t`标识同步代码中产生的宏任务执行, 打印`p`标识同步代码产生的微任务执行;
打印`tp`标识宏任务代码中的微任务执行;

*一*

1. 执行`setTimeout(fn1, 100)`, 设定timer于100ms后将fn1加入宏任务队列;
2. 执行`setTimeout(fn7, 0)`, 把fn7(函数func)加入宏任务队列, 主程序执行完毕;
3. 检查微任务队列(此时为空), 继续下一次循环;


*二*

1. 从宏任务队列中提取下一个任务执行, 此时为fn7, 执行fn7;
2. 打印"fn7";
3. 执行func函数, 当前i = 0, 打印i值为0;
4. 执行promise, 立即把fn4加入微任务队列;
5. 执行`setTimeout(fn5, 0)`, 把fn5加入宏任务队列, 当前任务执行完毕;
6. 检查微任务队列, 此时为fn4, 执行fn4, 打印"fn4-0";
7. 微任务队列清空, 进入下一个循环;

*三*

1. 从宏任务队列中取下一个任务执行, 此时为fn5, 执行fn5;
2. 打印"fn5-0";
3. 执行promise, 立即将fn6加入微任务队列;
4. 执行++i, 此时i = 1, i < 3为true, 执行func函数;
5. 打印i值为1;
6. 执行promise立即将fn4加入微任务队列;
7. 执行`setTimeout(fn5, 0)`, 把fn5加入宏任务队列, 当前任务执行完毕;
8. 检查微任务队列, 此时为fn6, fn4;
9. 执行fn6, 打印"fn6-1";
10. 执行fn4, 打印"fn4-1";

*四*

1. 从宏任务队列中取下一个任务执行, 此时为fn5, 执行fn5;
2. 打印"fn5-1";
3. 执行promise, 立即将fn6加入微任务队列;
4. 执行++i, 此时i = 2, i < false, 内部代码不执行, 当前任务执行完毕;
5. 检查微任务队列, 此时为fn6;
6. 执行fn6, 打印"fn6-2", 当前任务执行完毕, 微任务队列清空, 进入下一个循环;

*五*

从第一个循环开始执行的100ms之后才会把fn1添加入任务队列, 一旦进入任务队列, 由于此时执行栈处于空闲状态, 检测到有异步任务可执行则立即执行;
100ms后
1. fn1进入任务队列并被执行,打印"fn1";
2. 执行promise, 立即将fn2加入微任务队列, 当前任务执行完毕;
3. 检查微任务队列, 此时为fn2;
4. 执行fn2, 打印"fn2", 当前任务执行完毕, 微任务队列清空, 进入下一个循环;


参考:
+ [JavaScript中的事件循环 Event Loop](http://coderlt.coding.me/2017/12/13/event-loop/)
+ [HTML标准-Event loops](https://www.w3.org/TR/html5/webappapis.html#event-loop)
+ [MDN-Event loops](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)
