## JS实现一个lodash接口的柯里化函数

```javascript
var abc = function(a, b, c) {
  return [a, b, c];
};

var curried = _.curry(abc);

curried(1)(2)(3);
// => [1, 2, 3]

curried(1, 2)(3);
// => [1, 2, 3]

curried(1, 2, 3);
// => [1, 2, 3]

// Curried with placeholders.
curried(1)(_, 3)(2);
// => [1, 2, 3]
```

> 摘自 [lodash文档v4.17.15](https://lodash.com/docs/4.17.15#curry)

---
现在开始来实现它，我的做法是从接口反推实现，我们先来细致拆分一下其功能:
  - 接收一个函数作为参数(_.curry(fn))
  - 返回值是一个函数
  - 调用其返回的函数，返回的是:
    - 一个函数(~~套娃~~递归警告), 此函数拥有部分已求得的值(预设的arguments，以及经过(多次)调用过后的剩余形参列表
    - 最终值

线索整理的差不多，开始coding:
```javascript
const curry = (fn) => { // fn as param
  return function(...rest) {
    if (/* still have some params to call */) {
      /* return a function to call recursively */
    } else {
      return fn.apply(null, rest); // final call
    }
  }
}
```

稍微分析，不难得到，if里需要的条件即:
  - 传入的参数个数，小于递归调用的fn所余下的参数个数
  - 递归调用自己，所以curry应该多一个参数，来表示剩下的参数个数
  - 上一条的边界条件: 首次调用，剩余个数 === fn.length

那么修改一下
```javascript
const curry = (fn, length) => { // fn as param
  const adaptedLen = length === undefined ? fn.length : length;
  return function(...rest) {
    if (rest.length < adaptedLen) {
      return curry.apply() // ???
    } else {
      return fn.apply(null, rest); // final call
    }
  }
}
```
问号注释这里又不对劲了，该return一个怎样的函数，参数和上下文该是怎样的？
  - 应该是递归调用curry
  - fn参数保留原来的fn没错，但是fn的形参应该有所改变:
    - 保留被执行过的结果(外部已经递归调用过的部分)
    - 保留剩余的参数列表

此时我想到一个ES6的接口很符合我们的要求, Function.prototype.bind
```javascript
function list() {
  return Array.prototype.slice.call(arguments);
}

function addArguments(arg1, arg2) {
    return arg1 + arg2
}

var list1 = list(1, 2, 3); // [1, 2, 3]

var result1 = addArguments(1, 2); // 3

// 创建一个函数，它拥有预设参数列表。
var leadingThirtysevenList = list.bind(null, 37);

// 创建一个函数，它拥有预设的第一个参数
var addThirtySeven = addArguments.bind(null, 37);

var list2 = leadingThirtysevenList();
// [37]

var list3 = leadingThirtysevenList(1, 2, 3);
// [37, 1, 2, 3]

var result2 = addThirtySeven(5);
// 37 + 5 = 42

var result3 = addThirtySeven(5, 10);
// 37 + 5 = 42 ，第二个参数被忽略
```
> 摘自[bind - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

将已经call的部分参数作为预设的参数列表，保留函数主体
```javascript
const curry = (fn, length) => { // fn as param
  const adaptedLen = length === undefined ? fn.length : length;
  return function(...rest) {
    if (rest.length < adaptedLen) {
      const partialCalledFn = fn.bind(null, ...rest);
      return curry.apply(null, [partialCalledFn, adaptedLen - rest.length]);
    } else {
      return fn.apply(null, rest); // final call
    }
  }
}
```

好嘛，现在来试一下
```javascript
const test = (a, b, c) => [a, b, c];
const curried = curry(test);
curried(2)(3)(3)
// [2, 3, 3]
curried(2)(3, 3)
// [2, 3, 3]
curried(233, 3)(666)
// [233, 3, 666]
```
