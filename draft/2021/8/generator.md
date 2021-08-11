## Generator原理及实现
星号函数出现以前, JS的异步编程写法基本有 **回调** 和 **Promise (本质上也是注册回调)  ** 两种方式, 并且写法不够优雅, 也不易读；generator的出现使得异步编程在 **语法** 上同步化, 非常易写易读, 除此之外, generator function所返回的迭代器提供了 **保留上下文并等待执行** 和 **注入** 的方式来控制迭代器对集合的迭代, 功能更强大。

### 一般用法
generator定义了一组有序状态, 并返回一个迭代器, 通过调用迭代器的`.next()`方法, 来逐步遍历这组状态。
- 首先, 像写平常的函数一样写星号函数, 只不过星号函数拥有一些`yield`关键字与`yield` **后面的表达式**, 表达式的值即集合中迭代到的值
```javascript
function* gen() {
  yield '1';
  yield '2';
}
let iterator = gen();
iterator.next(); // {value: "1", done: false}
iterator.next(); // {value: "2", done: false}
```
- `return`语句是这个有序集合结束的标志, 不显式的写`return`就是`undefined`, 迭代结束之后(迭代器的`done`属性为`true`), 继续调用`.next()`, 会一直得到`{value: undefined, done: true}`
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
- 作为`yield`表达式整体, 通常都是`undefined`
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
- 可以通过对`next()`传参，来影响上一个yield表达式的值
```javascript
function* gen() {
  const s = yield 'sentence 1';
  console.log('s => ', s);
}
let iterator = gen();
iterator.next(); // {value: "sentence 1", done: false}
iterator.next('an injected value');
// s =>  an injected value
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
- 可被一些接口迭代, 如`...`, `for of`, 解构赋值等
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
- `yield*`语法, 如果表达式也是一个迭代器, `yield*`会逐步迭代完这个迭代器
```javascript
function* genInner() {
  yield 'inner 1';
  yield 'inner 2';
}

function* genOuter1() {
  yield 'outer2 1';
  const iterator = genInner();
  yield iterator;
}
var iterator1 = genOuter1();
iterator1.next();
// {value: "outer 1", done: false}
iterator1.next();
// {value: genInner, done: false}done: falsevalue: genInner {<suspended>}[[Prototype]]: Object
iterator1.next();
// {value: undefined, done: true}

function* genOuter2() {
  yield 'outer2 1';
  const iterator = genInner();
  yield* iterator;
}
var iterator2 = genOuter2();
iterator2.next()
// {value: "outer2 1", done: false}
iterator2.next()
// {value: "inner 1", done: false}
iterator2.next()
// {value: "inner 2", done: false}
iterator2.next()
// {value: undefined, done: true}
```

### 迭代协议
generator函数的返回值是一个可迭代对象，这个对象遵循**迭代协议**，迭代协议分为**可迭代协议**和**迭代器协议**
> 可迭代协议允许 JavaScript 对象定义或定制它们的迭代行为，例如，在一个 for..of 结构中，哪些值可以被遍历到。一些内置类型同时是内置可迭代对象，并且有默认的迭代行为，比如 Array 或者 Map，而其他内置类型则不是（比如 Object)）。
> 要成为可迭代对象， 一个对象必须实现 `@@iterator` 方法。这意味着对象（或者它原型链上的某个对象）必须有一个键为 @@iterator 的属性，可通过常量 `Symbol.iterator` 访问该属性：
> 
> 当一个对象需要被迭代的时候（比如被置入一个 `for...of` 循环时），首先，会不带参数调用它的 `@@iterator` 方法，然后使用此方法返回的迭代器获得要迭代的值。
> 值得注意的是调用此零个参数函数时，它将作为对可迭代对象的方法进行调用。 因此，在函数内部，`this`关键字可用于访问可迭代对象的属性，以决定在迭代过程中提供什么。
> 此函数可以是普通函数，也可以是生成器函数，以便在调用时返回迭代器对象。 在此生成器函数的内部，可以使用`yield`提供每个条目。

总结一下得知：
  - **可迭代协议**规定了对象通过`[Symbol.iterator]`去访问一个无参方法（不一定像引用中说的一定是`@@iterator`方法）
  - 这个方法返回一个**迭代器对象**, 迭代器对象遵循**迭代器协议**
  > 迭代器协议定义了产生一系列值（无论是有限个还是无限个）的标准方式。当值为有限个时，所有的值都被迭代完毕后，则会返回一个默认返回值。只有实现了一个拥有以下语义（semantic）的 next() 方法，一个对象才能成为迭代器：
  > 一个无参数函数，返回一个应当拥有以下两个属性的对象：
  > 
  > **done（boolean）**
  > 如果迭代器可以产生序列中的下一个值，则为 false。（这等价于没有指定  done 这个属性。）
  > 如果迭代器已将序列迭代完毕，则为 true。这种情况下，value 是可选的，如果它依然存在，即为迭代结束之后的默认返回值。
  > 
  > **value**
  > 迭代器返回的任何 JavaScript 值。done 为 true 时可省略。
  > next() 方法必须返回一个对象，该对象应当有两个属性： done 和 value，如果返回了一个非对象值（比如 false 或 undefined），则会抛出一个 TypeError 异常（"iterator.next() returned a non-object value"）。
### 实现
归纳一下协议部分，一个对象要实现迭代协议，需要做到：
  - 可以通过`[Symbol.iterator]`访问到一个方法，这个方法返回一个可迭代对象
  - 可迭代对象提供了`next()`方法，其返回值是一个拥有`done`和`value`两个值的对象
    - `done（boolean）`，如果迭代器可以产生序列中的下一个值，则为`false`，反之为`true`（此时`value`是可选的，如果有`value`就是迭代结束之后的返回值)。
    - `value`，迭代器返回的任何JavaScript值，`done`为`true`时可忽略.

```javascript
function myGen() {
  
  const values = [];
  const iterator = {
    next: function() {
      
    },
    [Symbol.iterator]: function() { return this }
  }
}
```
### 应用场景 (tbd)  
  - await / async
  - redux-saga
---

参考
- [迭代协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols#%E5%86%85%E7%BD%AE%E5%8F%AF%E8%BF%AD%E4%BB%A3%E5%AF%B9%E8%B1%A1)
- [迭代器和生成器](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators)