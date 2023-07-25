---
title: React 笔记
date: 2023/7/25
updated: 
categories: 
  - React
tags: 
---

# 基础知识

## 关于 React

1. 什么是 React？
   
   **React** 是一个用于构建用户界面的 JavaScript 库。
   
   - 是一个将数据渲染为 HTML 视图的开源 JS 库。
   - 它遵循基于组件的方法，有助于构建可重用的 UI 组件。
   - 它用于开发复杂的交互式的 web 和移动 UI。

2. React 有什么特点？
   
   - 使用虚拟 DOM 而不是真正的 DOM。
   - 它可以用服务器渲染。
   - 它遵循单向数据流或数据绑定。
   - 高效。
   - 声明式编码，组件化编码。

3. React 的一些主要优点？
   
   - 它提高了应用的性能。
   
   - 可以方便在客户端和服务器端使用。
   
   - 由于使用 JSX，代码的可读性更好。
   
   - 使用React，编写 UI 测试用例变得非常容易。

## JSX 语法

1. 定义虚拟DOM，不能使用`“”`
2. 标签中混入JS表达式的时候使用`{}`：`id = {myId.toUpperCase()}`
3. 样式的类名指定不能使用class，使用`className`
4. 内敛样式要使用`{{}}`包裹：`style={{color:'skyblue',fontSize:'24px'}}`
5. 不能有多个根标签，只能有一个根标签。
6. 标签必须闭合，自闭合也行。
7. 如果小写字母开头，就将标签转化为 html 同名元素，如果 html 中无该标签对应的元素，就报错；如果是大写字母开头，react 就去渲染对应的组件，如果没有就报错。

# 面向组件编程

**渲染类组件标签的基本流程**：

1. React 内部会创建组件实例对象。
2. 调用`render()`得到虚拟 DOM ,并解析为真实 DOM。
3. 插入到指定的页面元素内部。

## 函数组件

```jsx
//1.先创建函数，函数可以有参数，也可以没有，但是必须要有返回值 返回一个虚拟DOM
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
//2.进行渲染
ReactDOM.Render(<Welcom name = "ljc" />,document.getElementById("div"));
```

上面的代码经历了以下几步：

1. 我们调用 `ReactDOM.render()` 函数，并传入 `<Welcome name="ljc" />` 作为参数。
2. React 调用 `Welcome` 组件，并将 `{name: 'ljc'}` 作为 props 传入。
3. `Welcome` 组件将 `Hello, ljc` 元素作为返回值。
4. React DOM 将 DOM 高效地更新为 `Hello,ljc`。

## 类组件

```jsx
class MyComponent extends React.Component {
    state = {isHot:false}
    render() {
        const {isHot} = this.state
        return <h1 onClick={this.changeWeather}>今天天气很{isHot?'炎热':'凉爽'}</h1>
    }
    changeWeather = ()=>{
        const isHot = this.state.isHot
        this.setState({isHot:!isHot})
    }
}
ReactDOM.render(<MyComponent/>,document.querySelector('.test'))
```

**在优化过程中遇到的问题：**

1. 组件中的 render 方法中的 this 为组件实例对象。

2. 组件自定义方法中由于开启了严格模式，this 指向 `undefined` 如何解决。
   
   - 通过 bind 改变 this 指向。
   
   - 推荐采用箭头函数，箭头函数的 `this` 指向。

3. state 数据不能直接修改或者更新。

## 组件实例的三大属性

### state

> React 把组件看成是一个状态机（State Machines）。通过与用户的交互，实现不同状态，然后渲染 UI，让用户界面和数据保持一致。
> 
> React 里，只需更新组件的 state，然后根据新的 state 重新渲染用户界面（不要操作 DOM）。

简单的说就是组件的状态，也就是该组件所存储的数据。

```jsx
class Weather extends React.Component {
  constructor(props) {
    super(props);
    // this.state = { weather: '凉爽' };
  }

  state = {
    weather: '炎热',
  }

  render() {
    return <h1>{ this.state.weather }</h1>
  }
}
```

state 即当前组件的实例属性。

在类组件中可以直接修改 state 值，但修改之后页面内容是不会变化的。

原因是 React 中不建议 `state`不允许直接修改，而是通过类的原型对象上的方法 `setState()`

> 页面的渲染靠的是 `render` 函数。

**setState()**

`this.setState(partialState, [callback]);`

- `partialState`：需要更新的状态的部分对象。
- `callback`：更新完状态后的回调函数。

两种写法：

1. 对象形式：`this.setState({ weather: '凉爽' })`;
2. 回调形式：`this.setState(state => ({ count: state.count+1 }));`

> `setState` 是一种合并操作，并不是替换操作。
> 
> 每一次调用 `setState` 将会触发 `render` 进行页面渲染，因此 `react` 内部会将本次更新所涉及到的所有 `setState` 进行收集合并，避免重复渲染。

- 在执行 `setState`操作后，React 会自动调用一次 `render()`
- `render()` 的执行次数是 1+n (1 为初始化时的自动调用，n 为状态更新的次数)

### props

与`state`不同，`state`是组件自身的状态，而`props`则是外部传入的数据。

**类组件中：**

```jsx
class Person extends React.Component {
  render() {
    return (
      <ul>
          <li> { this.props.name } </li>
          <li> { this.props.age } </li>
      </ul>
    )
  }
}
```

在使用的时候可以通过 `this.props` 来获取值 类式组件的 `props`:

1. 通过在组件标签上传递值，在组件中就可以获取到所传递的值
2. 在构造器里的 `props` 参数里可以获取到 `props`
3. 可以分别设置 `propTypes` 和 `defaultProps` 两个属性来分别操作 `props` 的规范和默认值，两者都是直接添加在类式组件的**原型对象**上的（所以需要添加 `static` ）
4. 同时可以通过 `...` 运算符来简化

```jsx
class Person extends React.Component {
  render() {
    return (
      <ul>
          <li> { this.props.name } </li>
          <li> { this.props.age } </li>
      </ul>
    )
  }

  static propTypes = {
    name: PropTypes.string.isRequired,
    age: propTypes.string,
  }

  static defaultPorps = {
    name: "Akira",
    age: 21
  }
}

/** 使用 */
const person = { name: "AkiraIce", age: 21 }
ReactDom.render(<Person { ...p } />, document.getElementById("div"));
```

**函数组件中：**

```jsx
function Person(props) {
  return (
    <ul>
        <li>{ props.name }</li>
        <li>{ props.age }</li>
    </ul>
  )
}
```

函数组件的 `props`定义:

1. 在组件标签中传递 `props`的值
2. 组件函数的参数为 `props`
3. 对 `props`的限制和默认值同样设置在原型对象上

### refs

如同 Vue 中的 ref，用来获取 DOM 节点的。

有三种操作`refs`的方法，分别为：

- 字符串形式

- 回调形式

- `createRef`形式
1. 字符串形式
   
   ```jsx
   class Component extends React.Component {
     inputHanddle = () => {
         const { input } = this.refs;
       console.log(input.value);
     }
     render() {
       return (
         <div>
           <input ref="input" />
         </div>
       )
     }
   }
   ```

2. 回调形式
   
   ref 的回调函数将自动接收当前 DOM 节点。
   
   ```jsx
   <input ref={ c => this.input1 = c } />
   ```

3. createRef 形式
   
   ```jsx
   class Component extends React.Component {
     MyRef = React.createRef();
     MyRef1 = React.createRef();
   
     //调用
     btnOnClick = () =>{
         //创建之后，将自身节点，传入current中
         console.log(this.MyRef.current.value);
     }
   
     render() {
       return (
         <input ref={this.MyRef} type="text" placeholder="点击弹出" />
         <input ref={this.MyRef1} type="text" placeholder="点击弹出" />
       )
     }
   }
   ```

4. 点击事件
   
   React 使用的是自定义事件，而不是原生的 DOM 事件
   
   React 的事件是通过事件委托方式处理的（为了更加的高效）
   
   可以通过事件的 `event.target`获取发生的 DOM 元素对象，可以尽量减少 `refs`的使用
   
   ```jsx
   class Component extends React.Component {
     clickHanddle = (event) => {
       console.log(event.target.value);
     }
     render() {
       return (
              <div onClick={ this.clickHanddle }>
           )
     }
   }
   ```

# 生命周期

React 生命周期主要包括三个节点：初始化阶段、更新阶段、销毁阶段。

## 初始化阶段

### constructor

`constructor` 在组建初始化的时候只会执行一次。

通常它用于做这两件事

1. 初始化函数内部 `state`
2. 绑定函数

```jsx
constructor(props) {
    console.log('进入构造器');
    super(props)
    this.state = { count: 0 }
}
```

### static getDerivedStateFormPorps

这是 React 新版本中新增的两个钩子之一，据说很少用。

`getDerivedStateFormProps` 在初始化和更新中都会被调用，它接收两个参数 `props` 和 `state`，且在 `render` 方法之前调用，它返回一个对象用来更新 `state`

```jsx
class Component extends React.Component {
  state = {
    count: 1,
  }

  static getDerivedFromProps(props) {
    return {count: props.count};
  }

  render() {
    return (
        <h1>{ this.count }</h1>
    )
  }
}

<Component count="111"> // 111
```

### componentWillMount（即将废除）

> 如果存在 `getDerivedStateFromProps` 和 `getSnapshotBeforeUpdate` 就不会执行生命周期`componentWillMount`

该方法只在挂在的时候调用一次，表示组件将要挂载，并且在 `render` 方法之前调用。

这个方法在 React 18版本中将要被废弃，官方解释是在 React 异步机制下，如果滥用这个钩子可能会有 Bug。

### render

`render` 方法是组件中必须实现的方法，用于渲染 DOM，但是它不会真正的操作 DOM，它的作用是吧需要的东西返回出去，实现渲染 DOM 操作的是 `ReactDOM.render()`

> 注意：避免在 `render` 中使用 `setState`，否则会死循环，因为每一次 setState 将会重新执行 render 一次。

### 初始化阶段总结

执行顺序 `constructor` -> `getDerivedStateFromProps` 或者 `componentWillMount` -> `render` -> `componentDidMount`

![image-20230625151832472](https://s2.loli.net/2023/06/25/VP1UJLka35hc9eH.png)

## 更新阶段

![image-20230625145804292](https://s2.loli.net/2023/06/25/IOpzrCiUkZDKAf6.png)

### getDeriveStateFormProps

执行生命周期`getDerivedStateFromProps`， 返回的值用于合并 `state`，生成新的`state`。

### shouldComponentUpdate

`shouldComponentUpdate()` 在组件更新之前调用，可以通过返回值来控制组件是否更新，允许更新返回 `true` ，反之不更新。

### render

在控制是否更新的函数中，如果返回 `true` 才会执行 `render` ,得到最新的 `React element`

### getSnapshotBeforeUpdate

在最近一次的渲染输出之前被提交之前调用，也就是即将挂载时调用。

相当于淘宝购物的快照，会保留下单前的商品内容，在 React 中就相当于是 即将更新前的状态。

> 它可以使组件在 DOM 真正更新之前捕获一些信息（例如滚动位置），此生命周期返回的任何值都会作为参数传递给 `componentDidUpdate()`。如不需要传递任何值，那么请返回 null。

### componentDidUpdate

组件在更新完毕后会立即被调用，首次渲染不会调用。

## 销毁阶段

### componentWillUnmount

在组件即将被卸载或销毁时进行调用。

## 总结

**初始化**

- constructor()
- static getDerivedStateFromProps()
- render()
- componentDidMount()

**更新**

- static getDerivedStateFromProps()
- shouldComponentUpdate()
- render()
- getSnapshotBeforeUpdate()
- componentDidUpdate()

**销毁**

- componentWillUnmount()
