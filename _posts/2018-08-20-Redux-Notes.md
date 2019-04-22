---
layout: post
title: Redux笔记
---

Redux提供的功能可以理解为一个状态管理器，也就是viewModel的核心，能够把与UI动态界面相关的状态或数据统一管理。通过控制viewModle也就是store的状态变化，驱动UI界面自动更新，另一方面UI界面与用户相关的所有变更，也会实时被绑定的监听函数发起对store的更新。

## createStore的简单实现：
```javascript
const createStore = (reducer) => {
    let state;
    let listeners = [];
    const getState = () => state;
    const dispatch = (action) => {
        //执行state的更新
        state = reducer(state, action);
        //执行监听函数
        listeners.forEach((listener) => listener());
        };
        const subscribe = (listener) => {
        //添加监听函数
        listeners.push(listener);
        //返回可以解除监听的函数
        return () => {
        listeners = listeners.filter(l => l !== listener)
        }
    }
}
```
## combineReducers的简单实现
```javascript
const combineReducers = reducers => {
    //返回一个整体的reducer函数
        return (state = {}, action) => {
            //state中的属性名数组（子reducer函数对应的属性）
            return Object,keys(reducers).reduce(
            //reducers中的reducer对应的属性更新，归并结果得到新的state对象
            (nextState, key) => {
                nextState[key] = reducers[key](state[key], action);
                return nextState;
            },
            {}
        )
    }
}
```
## 中间件
中间件是一个函数，是`store.dispatch`方法的增强，在发出action和执行reducer之间添加了其他功能；
*使用*：
+ 需要： `applyMiddleware`函数和中间件函数；
+ 将中间件函数放在`applyMiddleware`方法中，传入`createStore`方法；
（中间件需按照一定次序传入`applyMiddleware`中，具体根据中间件规定；）
+ `applyMiddleware`方法接收中间件序列作为参数，将它们按顺序由内而外包装在`store.dispatch`方法之外；
*源码*：
```javascript
export default function applyMiddleware(...middlewares) {
    return (createStore) => (reducer, preloadedState, enhancer) => {
        var store = createStore(reducer, preloadedState, enhancer);
        var dispatch = store.dispatch;
        var chain = [];
        
        var middlewareAPI = {
            getState: store.getState,
            dispatch: (action) => dispatch(action)
        };
        chain = middlewares.map(middleware => middleware(middlewareAPI));
        dispatch = compose(...chain)(store.dispatch);
        
        return {...store, dispatch}
    }
}
```
其中的compose函数：
```javascript
export default function compose(...funcs) {
    if (funcs.length === 0) {
        return arg => arg;
    }
    if (funcs.length === 1) {
        return funcs[0];
    }
    return funcs.reduce((a, b) => (...args) => a(b(...args)));
}
/*例：*/  compose(a,b,c,d)(...args) = a(b(c(d(...args))));
```
+ 异步操作的action种类：发出时，成功时，失败时，通过标志位或type不同来区分；
+ state中也要有反映不同操作状态的属性；

1 操作--> 发出action-1-->state更新为“正在操作” --> rerender view;
2 操作结束--> 发出action-2-->state更新为“操作结束” --> rerender view;

## thunk中间件
```javascript
(import {createStore, applyMiddleware} from 'redux'; import thunk from 'redux-thunk')
const store = createStore(
    reducer,
    applyMiddleware(thunk)
);
//写出一个返回函数的Action Creator, 然后使用redux-thunk改造store.dispatch; 
Action Creator 例：
const fetchPosts = postTitle => (dispatch, getState) => {
    //先发出一个action, 操作开始
    dispatch(requestPosts(postTitle));
    return fetch(`/some/API/${postTitle}.json`)
         //收到结果转换为json格式
        .then(response => response.json())
         //再发出一个action，操作结束
        .then(json => dispatch(receivePosts(postTitle, json)));
};
fetchPosts 返回一个函数，执行后先发出一个action(即requestPosts(postTitle)), 然后进行异步操作.
```
## React-Redux 
UI组件只负责视图层，无状态，无redux的API，数据通过this.props提供；
容器组件只负责业务逻辑，有内部状态，有redux的API；由react-redux自动生成;

### connect() 
用于从UI组件生成容器组件；
*参数*： 	输入逻辑 `mapStateToProps` （从state获取哪些props）；
*输出逻辑* `mapDispatchToProps`  (将UI组件上的操作映射成action); 
*返回*：函数，接收UI组件为参数，返回容器组件；
`mapStateToProps(state, ownProps)`,  
+ 返回一个对象，包含要传递给UI组件的参数；每当state更新会自动执行重新计算参数，进而触发UI组件的重新渲染；
+ 第二个参数可选，ownProps代表容器组件的props对象；
`mapDispatchToProps(dispatch, ownProps)`, 
+ 作为函数：返回一个对象，每个键值对都是一组映射，定义UI组件参数怎样发出action;
+ 作为对象：键名是对应UI组件上的同名参数，键值为Action creator类似的函数，返回的action自动发出；

### Provider组件
+ react-redux提供Provider组件，用来让容器组件可以得到state;
+ 在UI组件最外层包裹Provider组件： `<Provider store={store}> ... </Provider>`

### bindActionCreators()
作用： 返回一个函数，调用这个函数时会自动dispatch对应的action
```javascript
function bindActionCreator(actionCreator, dispatch) {
    return function() {
        return dispatch(actionCreator.apply(this, arguments))
    }
}
或：
return function(...args) {return dispatch(actionCreator.apply(this, args))}
export default function bindActionCreators(actionCreators, dispatch) {
    if (typeof actionCreators ==="function") {
    return bindActionCreator(actionCreators, dispatch)
}
if (typeof actionCreators !== "object" || actionCreators === null) {
    throw new Error(
        `bindActionCreators expected an object or a function, 
        instead received ${actionCreators === null ? 'null' : typeof actionCreators}.` +
         `Did you write "import ActionCreators from" instead of 
        " import * as ActionCreators from" ?`
    )
}
//获取所有actionCreator函数的名字
const keys = Object.keys(actionCreators)
//对所有是函数的actionCreator函数进行绑定，并保存在boundActionCreators对象中
const boundActionCreators = {}
for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const actionCreator = actionCreator[key]
    if (typeof actionCreator === "function") {
        boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
}
//返回绑定之后的对象
return boundActionCreators
}
```








