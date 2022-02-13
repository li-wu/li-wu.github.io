---
title: React带参数的事件处理
date: 2018-04-18 00:00:00
categories: JavaScript
---

对于React Component中的事件的绑定，一种做法是采用`bind`来实现。看下面例子：

```javascript
import React, { Component } from 'react';

class Example extends Component {
    constructor(props) {
        super(props);
    }
    
    handleClick(e) {
        console.log('Clicked', e);
    }
    
    render() {
        <button onClick={this.handleClick.bind(this)}></button>
    }
}
```

如果要给事件处理函数传递额外的参数，可以在调用`bind`的时候添加。`bind`的作用就是使一个函数拥有预设的初始参数，这些参数（如果有的话）作为`bind()`的第二个参数跟在`this`（或其他对象）后面，之后它们会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在它们的后面。

```javascript
import React, { Component } from 'react';

class Example extends Component {
    constructor(props) {
        super(props);
    }
    
    handleClick(param, e) {
        console.log('Parameter', param);
        console.log('Clicked', e);
    }
    
    render() {
        <button onClick={this.handleClick.bind(this, 'Parameter')}></button>
    }
}
```

但是这样有一个问题就是组件每次在`render`的时候都会重新生成一个新的函数，这对性能有影响。那是不是可以把`bind`操作放到`constructor`里面？

```javascript
import React, { Component } from 'react';

class Example extends Component {
    constructor(props) {
        super(props);
        this.handleClick = this.handleClick.bind(this, 'Parameter');
    }
    
    handleClick(param, e) {
        console.log('Parameter', param);
        console.log('Clicked', e);
    }
    
    render() {
        <button onClick={this.handleClick}></button>
    }
}
```

不过这样就没法动态传递参数了。推荐的做法是使用属性初始化器（Property Initializer Syntax）和Curry来实现。

```javascript
import React, { Component } from 'react';

class Example extends Component {
    constructor(props) {
        super(props);
    }
    
    handleClick = (param) => (e) => {
        console.log('Parameter', param);
        console.log('Clicked', e);
    }
    
    render() {
        <button onClick={this.handleClick('Parameter')}></button>
    }
}
```

参考：

[https://medium.freecodecamp.org/reactjs-pass-parameters-to-event-handlers-ca1f5c422b9](https://medium.freecodecamp.org/reactjs-pass-parameters-to-event-handlers-ca1f5c422b9)

-End-

