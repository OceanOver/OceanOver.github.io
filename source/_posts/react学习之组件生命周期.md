title: react学习之组件生命周期
date: 2016-11-03 11:51:18
tags:
- React
---

### 概述
学习iOS时我们会去研究视图控制器的生命周期，理解控制器从创建到销毁。联想到React，其组件也有从创建到销毁的过程。在组件的生命周期中，随着组件props或state的改变，其DOM表现形式也会有所变化。在生命周期的不同阶段，React提供了不同的处理函数使我们能够实现对组件整个生命周期内的控制和处理。<!-- more -->

React组件的生命周期分为：创建期、存在期、销毁&清理期，对应组件生命周期里的三个状态：

* Mounting：已插入真实 DOM
* Updating：正在被重新渲染
* Unmounting：已移出真实 DOM


```
var App = React.createClass({
  componentWillMount: function(){
    // 初始化期，组件加载前调用
  },
  componentWillUpdate: function(){
    // 存在期，组件状态改变后重新渲染前调用
  },
  render: function () {
    // 初始化期或存在期调用
    return (
      <h1>itbilu.com</h1>
    )
  },
  componentWillUnmount: function (){
    // 销毁&清理期调用
  }
});
```

React 为每个状态都提供了两种处理函数，will 函数在进入状态之前调用，did 函数在进入状态之后调用，三种状态共计五种处理函数。

* componentWillMount()
* componentDidMount()
* componentWillUpdate(object nextProps, object nextState)
* componentDidUpdate(object prevProps, object prevState)
* componentWillUnmount()

此外，React 还提供两种特殊状态的处理函数。

* componentWillReceiveProps(object nextProps)：已加载组件收到新的参数时调用
* shouldComponentUpdate(object nextProps, object nextState)：组件判断是否重新渲染时调用（灵活地利用该处理函数可以优化性能）

### 组件创建
当组件被创建、首次渲染时，都会有一系列方法会被调用。

当首次创建一个组件时依次执行：

* getDefaultProps
* getInitialState
* componentWillMount
* render
* componentDidMount

#### getDefaultProps
该方法只在首次创建实例被调用一次，一般用于顶级组件（没有父组件的组件）进行处理，实例化组件时这个方法会对返回默认的props属性。
#### getInitialState
实例化组件时会被调用一次。通过这个方法，可以初始化每个实例的state。getInitialState在每次创建实例时都会被调用。
#### componentWillMount
首次渲染（首次挂载）前被调用，通过这个方法可以在组件被render前最后一次修改组件的state。
#### render
render方法会创建一个虚拟DOM，其返回值可以是：null、false或React组件，只能返回一个顶级组件。React会将它和真实DOM做比较，以判断是否需要对DOM进行修改。
#### componentDidMount
真实的DOM已经被渲染后该方法被调用，可以在componentDidMount方法内部通过this.getDOMNode()访问到它。当需要访问原始DOM属性，可以将这些操作挂载到这个方法上。

### 存在期
在这个时期，组件已经渲染好并且用户可以与它进行交互，用户的操作会改变组件的state，这时会有新的state流注入到组件树，这时会有一些方法会被触发。

state改变后，会有以下方法会被触发：

* componentWillReceiveProps
* shouldComponentUpdate
* componentWillUpdate
* render
* componentDidUpdate

#### componentWillReceiveProps
当组件的props属性被修改时，该方法会被调用，这时我们可以获得修改的props对象和更新的state。
#### shouldComponentUpdate
该方法接收到修改的props对象和更新的state，当我们确定组件不需要重新渲染新的props或state时，可以通过该方法返回false，这样React就会跳过调用render方法及之后的componentWillUpdate和componentDidUpdate方法。
#### componentWillUpdate
发生在组件在接收到新的props或state后，render方法进行渲染之前
#### componentDidUpdate
与componentDidMount方法类似，这个方法使我们可以获取到已经渲染好的DOM。

### 组件销毁
当使用完一个组件时，需要将其从DOM中卸载并销毁，这时只有componentWillUnmount一个函数会被调用，通过这个方法我们可以完成一些必要的销毁和清理工作，如：清除componentDidMount中添加的任务等。

