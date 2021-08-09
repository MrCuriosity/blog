## Generator原理及实现
星号函数出现以前，JS的异步编程写法基本有 **回调** 和 **Promise（本质上也是注册回调）** 两种方式，并且写法不够优雅，也不易读；generator的出现使得异步编程在 **语法** 上同步化，非常易写易读，除此之外，generator function所返回的迭代器提供了 **保留上下文并等待执行** 和 **注入** 的方式来控制迭代器对集合的迭代，功能更强大。

## 一般用法
generator定义了一组状态，并返回一个迭代器，通过调用迭代器的`.next()`方法，来逐步遍历这组状态。
- 首先，像写平常的函数一样写星号函数，只不过星号函数拥有一些`yield`关键字与`yield` **后面的表达式**，表达式的值即集合中迭代到的值
```javascript
function* gen() {
  yield '1';
  yield '2';
}
let iterator = gen();
iterator.next(); // {value: "1", done: false}
iterator.next(); // {value: "2", done: false}
```
- `return`语句是这个有序集合结束的标志，不显式的写`return`就是`undefined`，迭代结束之后(迭代器的`done`属性为`true`)，继续调用`.next()`，会一直得到`{value: undefined, done: true}`
```javascript
function* gen() {
  yield '1';
  yield '2';
}
let iterator = gen();
iterator.next(); // {value: "1", done: false}
iterator.next(); // {value: "2", done: false}
iterator.next(); // {value: undefined, done: true}

function* gen() {
  yield '1';
  yield '2';
  return 'Explicit return value';
}
let iterator = gen();
iterator.next(); // {value: "1", done: false}
iterator.next(); // {value: "2", done: false}
iterator.next(); // {value: "Explicit return value", done: true}
iterator.next(); // {value: undefined, done: true}
```
- 作为`yield`表达式整体，通常都是`undefined`
```javascript
function* gen() {
  const s = yield 'sentence 1';
  console.log('s => ', s);
}
let iterator = gen(); // {value: "sentence 1", done: false}
iterator.next();
// s =>  undefined
// {value: undefined, done: true}
```
- 天然支持异步
```javascript
function* gen() {
  yield Promise.resolve('async 1');
  console.log('sync 1');
}
let iterator = gen();
iterator.next().value; // Promise {<fulfilled>: "async 1"}
iterator.next()
// sync 1
// {value: undefined, done: true}
```
- 可被一些接口迭代，如`...`, `for of`, 解构赋值等
```javascript
function* gen() {
  yield '1';
  yield '2';
  yield '3';
  return '4';
}
let iterator1 = gen();
[...iterator1] // (3) ["1", "2", "3"]

let iterator2 = gen();
for (v of iterator2) {
  console.log('v => ', v);
}
// v =>  1
// v =>  2
// v =>  3
// undefined

let iterator3 = gen();
let v1, arr;
[v1, ...arr] = [...iterator3]; // (3) ["1", "2", "3"]
v1 // "1"
arr // (2) ["2", "3"]
```
---

参考
- [迭代协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#%E5%86%85%E7%BD%AE%E5%8F%AF%E8%BF%AD%E4%BB%A3%E5%AF%B9%E8%B1%A1)
- [迭代器和生成器](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators)