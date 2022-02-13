---
title: JavaScript对象的浅拷贝和深拷贝
date: 2018-04-05 00:00:00
categories: JavaScript
---

### 浅拷贝

浅拷贝的概念很容易理解，即拷贝对象最外一层属性，更深层次的对象，则是通过引用指向同一块内存。

```javascript
function shadowClone(o) {
  const obj = {};
  for ( let i in o) {
    obj[i] = o[i];
  }
  return obj;
}

const oldObj = {
  a: 1,
  b: [ 'e', 'f', 'g' ],
  c: { h: { i: 2 } }
};

const newObj = shadowClone(oldObj);
console.log(newObj.c.h, oldObj.c.h); // { i: 2 } { i: 2 }
console.log(oldObj.c.h === newObj.c.h); // true
```

从上面的例子可以看出`oldObj.c.h`和`newObj.c.h`指向同一块内存，如果对`newObj.c.h`修改会影响`newObj.c.h`。这有时候并非你想要的，可能会带来潜在的问题。

实际开发中如果要使用浅拷贝可以使用ES6中的`Object.assign()`.

```javascript
const obj1 = {a: {b: 1}};
const obj2 = Object.assign({}, obj1);

obj1.a.b = 2;
console.log(obj2.a.b) // 2
```

### 深拷贝

深拷贝和浅拷贝的概念相对，即拷贝一个对象需要开辟一块新的内存地址，将原对象的各个属性逐个复制进去。对拷贝对象和源对象各自的操作互不影响。

在JavaScript中如果实现深拷贝？最简单的实现方法也许是用JSON对象的`parse`和`stringify`方法。即先将对象转化为JSON字符串，然后通过parse反序列化为JS对象。

```javascript
const newObj = JSON.parse(JSON.stringify(oldObj));
```

```javascript
const oldObj = {
  a: 1,
  b: [ 'e', 'f', 'g' ],
  c: { h: { i: 2 } }
};

const newObj = JSON.parse(JSON.stringify(oldObj));
console.log(newObj.c.h, oldObj.c.h); // { i: 2 } { i: 2 }
console.log(oldObj.c.h === newObj.c.h); // false
newObj.c.h.i = 'change';
console.log(newObj.c.h, oldObj.c.h); // { i: 'change' } { i: 2 }
```

这样使用可以满足简单的对象的深拷贝，能够处理JSON格式能表示的数据类型。但是存在很多问题：

1. 无法对正则表达式类型、函数类型等无法进行深拷贝；
2. 会抛弃对象的constructor,所有的构造函数会指向Object；
3. 对象有循环引用,会报错；

```javascript
// constructor
function person(pname) {
  this.name = pname;
}

const Messi = new person('Messi');

// Function
function say() {
  console.log('hi');
};

const oldObj = {
  a: say,
  b: new Array(1),
  c: new RegExp('ab+c', 'i'),
  d: Messi
};

const newObj = JSON.parse(JSON.stringify(oldObj));

// 无法复制函数
console.log(newObj.a, oldObj.a); // undefined [Function: say]
// 稀疏数组复制错误
console.log(newObj.b[0], oldObj.b[0]); // null undefined
// 无法复制正则对象
console.log(newObj.c, oldObj.c); // {} /ab+c/i
// 构造函数指向错误
console.log(newObj.d.constructor, oldObj.d.constructor); // [Function: Object] [Function: person]
```

从示例中可以看出使用JSON对象的`parse`和`stringify`存在的上述问题。如果存在循环引用也会出错：

```javascript
const oldObj = {};

oldObj.a = oldObj;

const newObj = JSON.parse(JSON.stringify(oldObj));
console.log(newObj.a, oldObj.a); // TypeError: Converting circular structure to JSON
```

实际开发中如果要实现深拷贝，建议使用lodash的[`cloneDeep`](https://lodash.com/docs/4.17.5#cloneDeep)。



如果需要自己实现深拷贝，就需要考虑需要如下问题：

* 对数组，正则对象，Date对象做特殊处理
* 处理对象原型
* 处理循环引用

当然最好的参考还是lodash中关于`cloneDeep`的[具体实现](https://github.com/lodash/lodash/blob/master/cloneDeep.js)。


