---
layout: post
title: React笔记
---

## JSX语法

+ 可以写任意表达式，用花括号包裹；
+ 声明React变量是必需的；
+ 组件名应该以大写字母开头；
+ 如果要根据表达式确定元素，可以赋值给一个大写字母开头的变量：

```javascript
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // 正确！JSX 标签名可以为大写开头的变量。
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

+ 属性可以通过{}传入表达式；
+ 可以通过数组的形式表示多个同级元素，需要每个元素都有一个独一无二的key；
+ `false`、`true`、`null` 或 `undefined`会被忽略(转换成字符串会完整显示)，但`0`依然会被渲染；


## 元素渲染

+ `ReactDOM.render`(元素对象/组件，挂载目标对象)
+ react对于重新创建或加载的元素，只更新改变的部分。

## 组件

+ react组件中，‘`class`’属性要写成'`className`'
+ 组件可以用函数定义（接收props并返回展示在页面的元素，无状态组件）；或用类来定义，可以使用局部状态、生命周期钩子；
+ 组件名称必须以大写字母开头；
+ 组件返回值只能有一个根元素，组合元素中在最外层用div包裹；
+ 组件不能修改它的props;
+ props应从组件自身的角度命名而不是上下文；

## props/state

+ props是实例对象的属性集合对象；
+ state是私有的，完全受控于当前组件；只适用于类；
+ state初始化只能在类的构造函数中；
+ state更新必须用`setState()`，传入更新后的state对象；
+ 使用`setState()`可以使组件进行必要的重新渲染；可以独立地更新state中的变量；也可以接收一个函数，根据之前的状态计算下一个状态；

## 生命周期

### 组件实例被创建和插入DOM中时：

+ constructor()
+ static getDerivedStateFromProps()
+ componentWillMount()(版本17后失效)
+ render()
+ componentDidMount()

### 属性或状态的改变会触发一次更新，重新渲染时：

+ componentWillReceiveProps()(版本17后失效)
+ static getDerivedStateFromProps()
+ shouldComponentUpdate()
+ componentWillUpdate()(版本17后失效)
+ render()
+ getSnapshotBeforeUpdate()
+ componentDidUpdate()

### 组件将被移除

+ componentWillUnmount()

### 渲染过程中发生错误

+ componentDidCatch()

### 组件装配和渲染过程

+ `render()`函数应为纯函数，不直接与浏览器交互，不直接改变组件状态；与浏览器交互的方法放在其他生命周期内；
+ `constructor(props)`中，应该在其他表达式之前首先调用`super(props)`,否则this的指向不对，无法通过`this.props`取得组件的props,可能使其他表达式出错;
+ `static getDerivedStatesFromProps(nextProps,prevProps)` 

  组件实例化和接收新属性时调用，返回一个对象更新组件的状态，或返回null不更新状态；一般是对props的变化才会响应；
+ `componentDidMount()` 组件被挂载后立即调用（在浏览器刷新屏幕之前发生）
+ `shouldComponentUpdate(nextProps,nextState)` 当接收到新属性或状态时，在重新渲染前调用，默认为true; 返回false时，`componentWillUpdate()`, `componentDidUpdate()`, `render()`不会调用；
+ `getSnapshotBeforeUpdate()`

在最新的渲染输出给DOM前调用，获取组件即将改变时当前的状态；其返回值作为参数传递给`componentDidUpdate()`
`componentDidUpdate(prevProps,prevState)` 更新发生后立即调用，此时可以操作DOM, 也可以发送请求；
+ `componentWillUnmount()` 在组件被卸载和销毁前调用，可以执行必要的清理工作，如解除定时任务、网络请求、事件绑定等。

## 事件处理

+ 事件绑定属性的命名用小驼峰方法；JSX中，如`onClick={handler}`,传入函数；
+ ES6 class语法不会自动绑定this，推荐使用构造函数中绑定或属性初始化器语法，将this对象绑定；
+ 向事件处理程序传递参数，事件对象event要排在其他参数的最后；

```javascript
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

## 条件渲染

+ 传统：if判断语句
+ jsx: &&短路操作符；可以根据条件判断显示或隐藏；

```javascript
return (<p>{false && "Please Login"}</p>）p标签中无内容；
```

+ 三目操作符；可以根据条件转换显示内容；

```javascript
return (
    <div>
    {isLogin ? (<UserGreeting />) :
         (<GuestGreeting />) }
    </div>
  )
```

设置`render`方法返回为`null`也可以隐藏组件，不影响生命周期函数的回调。

## 列表，keys

+ 列表的每一项都应该有一个独一无二的key；
+ key只和兄弟节点之间对比；需唯一；
+ key需要在数组的上下文中定义，不要在单项组件的内部定义；
+ key不会在props中传递

```javascript
function ListItem(props) {
  return <li>{props.value}</li>
}
function NumberList(props) {
  const numbers = props.numbers;
  const list = numbers.map((number) => <ListItem value={number.toString()} />);
  return (
    <ul>{list}</ul>
  )
};

```
## 表单

+ 一般使用受控组件；
+ 如果让表单数据由DOM处理，替代方案为使用非受控组件；
+ `<input type="text">`, `<textarea>`, `<select>`通过传入一个`value`属性来实现对组件的控制；
+ 通过设置不同的name，区分目标对象；

```javascript
  handleInputChange(e) {
    const target = e.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;
    
    this.setState({
      [name]:value
    });
  }
```

## 状态提升

+ 几个组件需要共享的状态，提升到它们最近的父组件中进行管理；
+ 在父组件中定义state和更新state的方法，作为props传入子组件中；子组件发生改变则触发父组件传入的事件处理函数，传入更新值，更新父组件的state；

## 组合，继承

+ 使用组合而非继承来复用组件之间的代码；
+ `props.children`将标签内任何内容通过props传入组件；
+ 也可以通过自定义属性传递组件；



