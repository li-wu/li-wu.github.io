---
title: 状态管理用到的一些工具和开源库
categories: JavaScript
date: 2018-10-10
---

[TOC]

最近项目中用到了Redux和一些开源的工具，稍微总结一下。

## Redux

阮一峰有三篇文章讲[Redux入门](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)， 当然最好的是去看[官方的文档](https://redux.js.org/)，也可以看看相关的视频（[前30集](https://egghead.io/courses/getting-started-with-redux)，[后30集](https://egghead.io/courses/building-react-applications-with-idiomatic-redux))。

### 设计思想

* Web应用是一个状态机，视图和状态一一对应；
* 所有的状态都保存在一个对象里面；

### 基本概念

#### Store

Store用来保存数据，整个应用只有一个Store。Redux用`createStore`来生成Store。

```javascript
import { createStore } from 'redux';
const store = createStore(fn);
```

`createStore`接受一个Reducer为参数，生成一个store对象。

#### State

Store对象包含所有的数据，如果要得到某个时点的数据，就要生成Store的快照，叫做State。

可以通过`store.getState()`拿到对应的State。

#### Action

Action表示View发出的通知，表示State要发生变化。Action 是一个对象。其中的`type`属性是必须的，表示 Action 的名称，其他属性可以自由设置。

```javascript
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```

#### Action Creator

Action Creator就是生成Action的函数，能避免每次都手写Action。

```javascript
const ADD_TODO = '添加 TODO';

function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}

const action = addTodo('Learn Redux');
```

#### Store.dispatch()

`store.dispatch()`是 View 发出 Action 的唯一方法。

```javascript
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});

// store.dispatch(addTodo('Learn Redux'));
```

#### Reducer

Store 收到 Action 以后，必须给出一个新的 State，这样 View 才会发生变化。这种 State 的计算过程就叫做 Reducer。Reducer 是一个函数，它接受 Action 和当前 State 作为参数，返回一个新的 State。

```javascript
const reducer = function (state, action) {
  // ...
  return new_state;
};
```

```javascript
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};
```

这个函数之所以成为Ruducer是因为它可以作为数组的`reduce`方法的参数。请看下面的例子，一系列 Action 对象按照顺序作为一个数组。

```javascript
const actions = [
  { type: 'ADD', payload: 0 },
  { type: 'ADD', payload: 1 },
  { type: 'ADD', payload: 2 }
];

const total = actions.reduce(reducer, 0); // 3
```

#### 纯函数

纯函数是指同样的输入必定得到同样的输出的函数，reducer就是纯函数。纯函数的一些约束：

* 不得改写参数
* 不能调用系统 I/O 的API
* 不能调用`Date.now()`或者`Math.random()`等不纯的方法，因为每次会得到不一样的结果

所以reducer必须这么写：

```javascript
// State 是一个对象
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });
  // 或者
  return { ...state, ...newState };
}

// State 是一个数组
function reducer(state, action) {
  return [...state, newItem];
}
```

#### Store.subscribe()

Store 允许使用`store.subscribe`方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。

```javascript
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

`store.subscribe`方法返回一个函数，调用这个函数就可以解除监听。

```javascript
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);

unsubscribe();
```

### Reducer的拆分

```javascript
const chatReducer = (state = defaultState, action = {}) => {
  const { type, payload } = action;
  switch (type) {
    case ADD_CHAT:
      return Object.assign({}, state, {
        chatLog: state.chatLog.concat(payload)
      });
    case CHANGE_STATUS:
      return Object.assign({}, state, {
        statusMessage: payload
      });
    case CHANGE_USERNAME:
      return Object.assign({}, state, {
        userName: payload
      });
    default: return state;
  }
};
```

上面例子里面的state的三个属性之间没有联系，可以reducer函数拆分开，最终合并成一个大的reducer。

```javascript
const chatReducer = (state = defaultState, action = {}) => {
  return {
    chatLog: chatLog(state.chatLog, action),
    statusMessage: statusMessage(state.statusMessage, action),
    userName: userName(state.userName, action)
  }
};
```

Redux 提供了一个`combineReducers`方法，用于 Reducer 的拆分。你只要定义各个子 Reducer 函数，然后用这个方法，将它们合成一个大的 Reducer。

```javascript
import { combineReducers } from 'redux';

const chatReducer = combineReducers({
  chatLog,
  statusMessage,
  userName
});
```

如果State属性名和Reducer不同名，就要采用下面的写法:

```javascript
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})

// 等同于
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```

### 工作流

![](/assets/images/frontend-tech/redux-flow.jpg)

### Example

```javascript
const Counter = ({ value, onIncrement, onDecrement }) => (
  <div>
  <h1>{value}</h1>
  <button onClick={onIncrement}>+</button>
  <button onClick={onDecrement}>-</button>
  </div>
);

const reducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT': return state + 1;
    case 'DECREMENT': return state - 1;
    default: return state;
  }
};

const store = createStore(reducer);

const render = () => {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({type: 'INCREMENT'})}
      onDecrement={() => store.dispatch({type: 'DECREMENT'})}
    />,
    document.getElementById('root')
  );
};

render();
store.subscribe(render);
```

##React-Redux

如果要使用React-Redux，需要掌握额外的 API，并且要遵守它的组件拆分规范。

###UI组件

在React-Redux中，所有组件分为UI组件和容器组件。UI 组件负责 UI 的呈现，容器组件负责管理数据和逻辑。

UI组件特征：

- 只负责 UI 的呈现，不带有任何业务逻辑
- 没有状态（即不使用`this.state`这个变量）
- 所有数据都由参数（`this.props`）提供
- 不使用任何 Redux 的 API

###容器组件

- 负责管理数据和业务逻辑，不负责 UI 的呈现
- 带有内部状态
- 使用 Redux 的 API

React-Redux 规定，所有的 UI 组件都由用户提供，容器组件则是由 React-Redux 自动生成。也就是说，用户负责视觉层，状态管理则是全部交给它。 

###connect()

`connect`用于从UI组件生成容器组件。

```javascript
import { connect } from 'react-redux'
const VisibleTodoList = connect()(TodoList);
```

`TodoList`是 UI 组件，`VisibleTodoList`就是由 React-Redux 通过`connect`方法自动生成的容器组件。

为了在容器组件中定义业务逻辑，需要给出如下两方面信息：

* 输入逻辑：外部的数据（即`state`对象）如何转换为 UI 组件的参数

* 输出逻辑：用户发出的动作如何变为 Action 对象，从 UI 组件传出去。

```javascript
import { connect } from 'react-redux'

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)
```

`mapStateToProps`负责输入逻辑，即将`state`映射到 UI 组件的参数（`props`），`mapDispatchToProps`负责输出逻辑，即将用户对 UI 组件的操作映射成 Action。

###mapStateToProps()

`mapStateToProps`是一个函数，作为函数，`mapStateToProps`执行后应该返回一个对象，里面的每一个键值对就是一个映射。

```javascript
const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```

返回的对象有一个`todos`属性，代表 UI 组件的同名参数，后面的`getVisibleTodos`也是一个函数，可以从`state`算出 `todos` 的值。

`mapStateToProps`的第一个参数总是`state`对象，还可以使用第二个参数，代表容器组件的`props`对象。

```javascript
// 容器组件的代码
//    <FilterLink filter="SHOW_ALL">
//      All
//    </FilterLink>

const mapStateToProps = (state, ownProps) => {
  return {
    active: ownProps.filter === state.visibilityFilter
  }
}
```

###mapDispatchToProps()

`mapDispatchToProps`是`connect`函数的第二个参数，用来建立 UI 组件的参数到`store.dispatch`方法的映射。也就是说，它定义了哪些用户的操作应该当作 Action，传给 Store。它可以是一个函数，也可以是一个对象。

```javascript
const mapDispatchToProps = (
  dispatch,
  ownProps
) => {
  return {
    onClick: () => {
      dispatch({
        type: 'SET_VISIBILITY_FILTER',
        filter: ownProps.filter
      });
    }
  };
}
```

```javascript
const mapDispatchToProps = {
  onClick: (filter) => {
    type: 'SET_VISIBILITY_FILTER',
    filter: filter
  };
}
```

###\<Provider\>组件

\<Provider\>组件目的是为了让组件容器拿到`state`。

```javascript
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp);

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

上面代码中，`Provider`在根组件外面包了一层，这样一来，`App`的所有子组件就默认都可以拿到`state`了。

###计数器示例

https://github.com/jackielii/simplest-redux-example/blob/master/index.js

```javascript
import React, { Component } from 'react'
import PropTypes from 'prop-types'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'
import { Provider, connect } from 'react-redux'

// React component
class Counter extends Component {
  render() {
    const { value, onIncreaseClick } = this.props
    return (
      <div>
        <span>{value}</span>
        <button onClick={onIncreaseClick}>Increase</button>
      </div>
    )
  }
}

Counter.propTypes = {
  value: PropTypes.number.isRequired,
  onIncreaseClick: PropTypes.func.isRequired
}

// Action
const increaseAction = { type: 'increase' }

// Reducer
function counter(state = { count: 0 }, action) {
  const count = state.count
  switch (action.type) {
    case 'increase':
      return { count: count + 1 }
    default:
      return state
  }
}

// Store
const store = createStore(counter)

// Map Redux state to component props
function mapStateToProps(state) {
  return {
    value: state.count
  }
}

// Map Redux actions to component props
function mapDispatchToProps(dispatch) {
  return {
    onIncreaseClick: () => dispatch(increaseAction)
  }
}

// Connected Component
const App = connect(
  mapStateToProps,
  mapDispatchToProps
)(Counter)

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```

##Immer

Github: https://github.com/mweststrate/immer

> *Create the next immutable state tree by simply modifying the current tree*

Immer帮助你更方便地使用Immutable State。

![](/assets/images/frontend-tech/immer.png)

###Sample

```javascript
import produce from "immer"

const baseState = [
    {
        todo: "Learn typescript",
        done: true
    },
    {
        todo: "Try immer",
        done: false
    }
]

const nextState = produce(baseState, draftState => {
    draftState.push({todo: "Tweet about it"})
    draftState[1].done = true
})
```

使用Immer之前：

```javascript
// redux-act reducer
const reducer = createReducer({
    [insert]: (state, payload) => {
        const { index, item } = payload;
        return {
            ...state,
            items: [
                ...state.items.slice(0, index),
                item,
                ...state.items.slice(index),
            ],
        };
    },
});
```

使用Immer之后:

```javascript
const reducer = createReducer({
    [insert]: (state, payload) => 
        produce(state, function() {
            const { index, item } = payload;
            this.items.splice(index, 0, item);
        }),
});
```

##Reselect

这个是Redux TodoList Example:https://redux.js.org/basics/usagewithreact

```javascript
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```

在上面的`mapStateToProps`中会调用`getVisibleTodos`去计算`todos`。这样有一个缺点是每当state更新的时候，`todos`都会被计算一次，当state tree很大，或者这个计算很耗时的时候，每次去计算就会带来性能问题。Reselect就是解决这个问题的。

Reselect提供`createSelector`来创建memorized selector。`createSelector`接受两个参数，一个是`input- selector`数组，另外一个是transform函数。如果Redux state状态改变导致其中的一个`input-selector` 的值改变，`selector`就会调用对应的transform函数。如果`input-selector`的值没有发生变化，那么就会直接返回之前计算的值。

```javascript
import { createSelector } from 'reselect'

const getVisibilityFilter = (state) => state.visibilityFilter
const getTodos = (state) => state.todos

export const getVisibleTodos = createSelector(
  [ getVisibilityFilter, getTodos ],
  (visibilityFilter, todos) => {
    switch (visibilityFilter) {
      case 'SHOW_ALL':
        return todos
      case 'SHOW_COMPLETED':
        return todos.filter(t => t.completed)
      case 'SHOW_ACTIVE':
        return todos.filter(t => !t.completed)
    }
  }
)
```

上面`getVisibilityFilter` and `getTodos` 就是input-selectors。



另外还有`rxjs`，下次单独分一篇来总结。

## References

* [http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)
* [https://redux.js.org/](https://redux.js.org/)
* [https://github.com/reduxjs/reselect](https://github.com/reduxjs/reselect)
* [https://redux.js.org/basics/usagewithreact](https://redux.js.org/basics/usagewithreact)
* [https://github.com/mweststrate/immer](https://github.com/mweststrate/immer)

-End-
