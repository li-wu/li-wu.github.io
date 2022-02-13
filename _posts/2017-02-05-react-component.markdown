---
date: 2017-02-05 19:50
title: React Component学习笔记
categories: react
---

## React组件知识点
1. 组件类首字母必须大写；
2. 所有的组件类必须有自己的`render`方法，用来输出组件；
3. 组件类只能包含一个顶层标签，否则会报错；
4. 组件的用法和原生的HTML标签完全一致，可以任意加入属性；
5. 属性可以在组件类的`this.props`对象上获取；
6. `class`属性要写成`className`，`for`属性要写成`htmlFor`，因`class`，`for`是JS保留字；
7. `this.props`和组件属性一一对应，除了`this.props.children`;
8. `this.props.children`表示组件的所有子节点；
9. `this.props.children`的值有三种可能：
    * 如果当前组件没有子节点，它就是`undeifned`；
    * 如果有一个子节点，数据类型是`object`；
    * 如果有多个节点，数据类型是`array`；
10. React提供一个工具方法`React.Children`来处理`this.pops.children`，而不必担心数据类型问题；
11. 组件类的`propTypes`属性用来验证组件实例的属性是否符合要求；
    ```
    propTypes: {
        title: React.PropTypes.string.isRequired,
    }
    ```
12. `getDefaultProps`方法可以用来设置组件属性的默认值；
    ```
    getDefaultProps: function() {
    	return {
    		title: 'hello world'
    	};
    }
    ```
13. 如果需要在组件中获取真实的DOM节点，就需要用到`ref`属性；
14. 由于`this.refs.[refName]`获取的是真实的DOM，所以必须等到虚拟DOM插入文档之后才能使用这个属性；
15. React将组件看成是一个状态机，`getInitialState`用来定义初始状态，也就是一个对象，这个对象可以通过`this.state`来读取；
16. `this.setState`方法用来修改状态值，每次修改之后会自动调用`this.render`方法重新渲染；
17. `this.props`表示那些一旦定义，就不再改变的特性，而`this.state`是会随着用户互动而产生变化的特性；
18. 对于表单，需要通过`event.target.value`来读取用户输入的值；
19. 组件的生命周期分为三个状态:
    * Mounting: 已插入真实DOM
    * Updating: 正在被重新渲染
    * Unmounting: 已移出真实DOM
20. React 为每个状态都提供了两种处理函数，will 函数在进入状态之前调用，did 函数在进入状态之后调用，三种状态共计五种处理函数：
    * `componentWillMount()`
    * `componentDidMount()`
    * `componentWillUpdate(object nextProps, object nextState)`
    * `componentDidUpdate(object prevProps, object prevState)`
    * `componentWillUnmount()`    
21. React还提供两种特殊的处理函数：
    * `componentWillReceiveProps(object nextProps)`：已加载组件收到新的参数时调用
    * `shouldComponentUpdate(object nextProps, object nextState)`：组件判断是否重新渲染时调用
