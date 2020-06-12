> [原文](https://promisesaplus.com/)\
> 文中斜体字是译者注

**An open standard for sound, interoperable JavaScript promises—by implementers, for implementers.**

一个用以表示、使用JavaScript promises的开放标准——由开发者实施, 供开发者使用.

A promise represents the eventual result of an asynchronous operation. The primary way of interacting with a promise is through its `then` method, which registers callbacks to receive either a promise’s eventual value or the reason why the promise cannot be fulfilled.

一个promise表示了一个异步操作的最终结果。promise的主要交互手段是其`then`方法, 该方法注册了一系列回调用来接收promise的最终值, 或是接收promise无法完成的原因。

This specification details the behavior of the `then` method, providing an interoperable base which all Promises/A+ conformant promise implementations can be depended on to provide. As such, the specification should be considered very stable. Although the Promises/A+ organization may occasionally revise this specification with minor backward-compatible changes to address newly-discovered corner cases, we will integrate large or backward-incompatible changes only after careful consideration, discussion, and testing.

这份规范详述了`then`方法的具体行为, 提供了一份所有Promises/A+的实施方案可以参考的操作方法.正因为如此, 这份规范应当非常稳定。不过, Promises/A+组织可能会偶尔的做一些细微的向后兼容的修订, 用以处理新发现的一些边角问题。我们只会在非常细致的考虑、讨论和测试之后才会集成一些大的或是向后兼容的改动。

Historically, Promises/A+ clarifies the behavioral clauses of the earlier [Promises/A](http://wiki.commonjs.org/wiki/Promises/A) proposal, extending it to cover de facto behaviors and omitting parts that are underspecified or problematic.

从历史角度来讲, Promises/A+规范明确了早期的[Promises/A](http://wiki.commonjs.org/wiki/Promises/A)提案具体行为条款.拓展并且使其覆盖了实际行为和先前不足的、有问题的部分。

Finally, the core Promises/A+ specification does not deal with how to create, fulfill, or reject promises, choosing instead to focus on providing an interoperable `then` method. Future work in companion specifications may touch on these subjects.

最终, Promises/A+规范的核心, 并不涉及如何create/fulfill/reject promises, 取而代之的, 我们聚焦在如何提供一个可互操作的`then`方法上。接下来, 配套的规范会涉及这些主题。

# 1.Terminology
  - 1.1 “promise” is an object or function with a then method whose behavior conforms to this specification.

  - 1.2 “thenable” is an object or function that defines a then method.

  - 1.3 “value” is any legal JavaScript value (including undefined, a thenable, or a promise).

  - 1.4 “exception” is a value that is thrown using the throw statement.

  - 1.5 “reason” is a value that indicates why a promise was rejected.

  # 1. 术语
  - 1.1 "promise"的类型是`object`or`function`, 并且有一个`then`方法, `then`的行为遵循规范.

  - 1.2 "thenable"是一个`object`or`function`, 它有一个`then`方法(*所以这个看起来像形容词的东西其实在这里是名词*)

  - 1.3 "value"可以是一切合法的JS value(包括`undefined`, `thenable`或者`promise`)

  - 1.4 "exception"是一个由`throw`声明抛出的value[*不一定是`error`*]

  - 1.5 "reason"用来表明promise被`reject`的原因

# 2. Requirements
## 2.1 Promise States
A promise must be in one of three states: pending, fulfilled, or rejected.

  - 2.1.1 When pending, a promise:

    - 2.1.1.1 may transition to either the fulfilled or rejected state.

  - 2.1.2 When fulfilled, a promise:

    - 2.1.2.1 must not transition to any other state.

    - 2.1.2.2 must have a value, which must not change.

  - 2.1.3 When rejected, a promise:

    - 2.1.3.1 must not transition to any other state.

    - 2.1.3.2 must have a reason, which must not change.

Here, “must not change” means immutable identity (i.e. ===), but does not imply deep immutability.

# 2. 需求
## 2.1 Promise 状态
promise的状态必须是以下三选一: `pending`, `fulfilled`, or `rejected`.

  - 2.1.1 当状态是`pending`时, promise:

    - 2.1.1.1 可以转换成`fulfilled` or `rejected`.

  - 2.1.2 当状态是`fulfilled`, promise:

    - 2.1.2.1 无法转换成其他任何状态.

    - 2.1.2.2 必须有一个value, 并且不可改变.

  - 2.1.3 当状态是`rejected`时, promise:

    - 2.1.3.1 无法转换成其他任何状态.

    - 2.1.3.2 必须有一个reason, 并且不可改变.

这里, “must not change”指的是id不可变(`===`), 但不意味着deepEqual(*可以改变引用对变量部的东西*)

# The `then` Method
A promise must provide a `then` method to access its current or eventual value or reason.

A promise’s `then` method accepts two arguments:

```javascript
promise.then(onFulfilled, onRejected)
```
- 2.2.1 Both `onFulfilled` and `onRejected` are optional arguments:

  - 2.2.1.1 If `onFulfilled` is not a function, it must be ignored.

  - 2.2.1.2 If `onRejected` is not a function, it must be ignored.

- 2.2.2 If `onFulfilled` is a function:

  - 2.2.2.1 it must be called after `promise` is fulfilled, with `promise`’s value as its first argument.

  - 2.2.2.2 it must not be called before `promise` is fulfilled.

  - 2.2.2.3 it must not be called more than once.

- 2.2.3 If `onRejected` is a function,

  - 2.2.3.1 it must be called after `promise` is rejected, with `promise`’s reason as its first argument.

  - 2.2.3.2 it must not be called before `promise` is rejected.

  - 2.2.3.3 it must not be called more than once.

- 2.2.4 `onFulfilled` or `onRejected` must not be called until the [execution context](https://es5.github.io/#x10.3) stack contains only platform code. [3.1].

- 2.2.5 `onFulfilled` and `onRejected` must be called as functions (i.e. with no `this` value). [3.2]

- 2.2.6 `then` may be called multiple times on the same promise.

  - 2.2.6.1 If/when `promise` is fulfilled, all respective `onFulfilled` callbacks must execute in the order of their originating calls to `then`.

  - 2.2.6.2 If/when `promise` is rejected, all respective `onRejected` callbacks must execute in the order of their originating calls to then.
- 2.2.7 then must return a promise [3.3].

```javascript
promise2 = promise1.then(onFulfilled, onRejected);
```
  - 2.2.7.1 If either `onFulfilled` or `onRejected` returns a value `x`, run the Promise Resolution Procedure `[[Resolve]](promise2, x)`.

  - 2.2.7.2 If either `onFulfilled` or `onRejected` throws an exception `e`, `promise2` must be rejected with `e` as the reason.

  - 2.2.7.3 If `onFulfilled` is not a function and `promise1` is fulfilled, `promise2` must be fulfilled with the same value as `promise1`.

  - 2.2.7.4 If `onRejected` is not a function and `promise1` is rejected, `promise2` must be rejected with the same reason as `promise1`.

# `then` 方法

promise必须提供一个`then`方法, 用来访问其当前value / 最终value / 或者reason.

promise的`then`方法接收两个参数:

```javascript
promise.then(onFulfilled, onRejected)
```

- 2.2.1 `onFulfilled`和`onRejected`都是可选参数:

  - 2.2.1.1 如果`onFulfilled`不是函数, 它会被忽略(*不传参*)

  - 2.2.1.2 如果`onRejected`不是函数, 他会被忽略(*不传参*).

- 2.2.2 如果`onFulfilled`是函数:

  - 2.2.2.1 它需要在`promise`被fulfilled之后被调用, `promise`的值是其第一个参数.

  - 2.2.2.2 它不能在`promise`被fulfilled之前被调用.

  - 2.2.2.3 它只能被调用一次.

- 2.2.3 如果`onRejected`是函数:
.
  - 2.2.3.1 他需要在`promise`被rejected之后调用, `promise`的reason是其第一个参数

  - 2.2.3.2 它不能再`promsie`被rejected之前被调用.

  - 2.2.3.3 它只能被调用一次.

- 2.2.4 `onFulfilled`或`onRejected`只能当[执行上下文](https://es5.github.io/#x10.3)只包含平台码时被调用[3.1](*我理解的是至少本次执行栈已完成, 保证其异步特性, 请看注释3.1*).

- 2.2.5 `onFulfilled`和`onRejected`必须以函数的形式调用(换言之, 没有`this`指针). [3.2]

- 2.2.6 `then`可以被同一个promise多次调用

  - 2.2.6.1 当`promise`被fulfilled时,所有`onFulfilled`回调必须以它们被`then`依次注册的顺序依次调用.

  - 2.2.6.2 If/when `promise` is rejected, all respective `onRejected` callbacks must execute in the order of their originating calls to then.

  - 2.2.6.2 当`promise`被rejected时,所有`onFulfilled`回调必须以它们被`then`依次注册的顺序依次调用.
- 2.2.7 `then`方法的返回值必须是一个promise[3.3].

```javascript
promise2 = promise1.then(onFulfilled, onRejected);
```

  - 2.2.7.1 如果以上代码中的`onFulfilled`或`onRejected` 返回一个`x`, 按2.3的顺序处理`[[Resolve]](promise2, x)`.

  - 2.2.7.2 如果以上代码中的`onFulfilled`或`onRejected` 抛出一个`e`, `promise2`必须以`e`作为其reason被rejected.

  - 2.2.7.3 如果`onFulfilled`不是函数并且`promise1`已经fulfilled, `promise2`必须和`promise1`同样的value被fulfilled.

  - 2.2.7.4 如果`onRejected`不是函数并且`promise1`已经rejected, `promise2`必须和`promise1`以同样的reason被rejected.

# 2.3 The Promise Resolution Procedure
The promise resolution procedure is an abstract operation taking as input a promise and a value, which we denote as `[[Resolve]](promise, x)`. If `x` is a thenable, it attempts to make `promise` adopt the state of `x`, under the assumption that `x` behaves at least somewhat like a promise. Otherwise, it fulfills `promise` with the value `x`.

This treatment of thenables allows promise implementations to interoperate, as long as they expose a Promises/A+-compliant `then` method. It also allows Promises/A+ implementations to “assimilate” nonconformant implementations with reasonable `then` methods.

# 2.3 Promise解析步骤
所谓**promise解析步骤**是一个抽象出来的操作, 表示为一个promose和一个value `[[Resolve]](promise, x)`.如果`x`是一个thenable, 在`x`工作起来至少像是一个`promise`的前提假设下(*thable, 鸭式辨形*), `promise`会采用`x`的state. 否则, 他会用`x`的value去fulfill这个promise.

这个基于thenables的探讨, 使得promise的(*各种*)实现之间得以交互——只要他们暴露有一个符合Promises/A+规范的`then`方法. 这也使得各种Promises/A+的实现能够兼容一些有着合规`then`方法但并不完善的其他promise实现(*比如我们练习时手写的并不那么完善的各种Promise*).

To run `[[Resolve]](promise, x)`, perform the following steps:

- 2.3.1 If `promise` and `x` refer to the same object, reject `promise` with a `TypeError` as the reason.
- 2.3.2 If `x` is a promise, adopt its state [3.4]:
  - 2.3.2.1 If `x` is pending, `promise` must remain pending until `x` is fulfilled or rejected.
  - 2.3.2.2 If/when `x` is fulfilled, fulfill `promise` with the same value.
  - 2.3.2.3 If/when `x` is rejected, reject `promise` with the same reason.
- 2.3.3 Otherwise, if `x` is an object or function,
  - 2.3.3.1 Let `then` be `x.then`. [3.5]
  - 2.3.3.2 If retrieving the property `x.then` results in a thrown exception `e`, reject `promise` with `e` as the reason.
  - 2.3.3.3 If `then` is a function, call it with `x` as `this`, first argument `resolvePromise`, and second argument `rejectPromise`, where:
    - 2.3.3.3.1 If/when `resolvePromise` is called with a value `y`, run `[[Resolve]](promise, y)`.
    - 2.3.3.3.2 If/when `rejectPromise` is called with a reason `r`, reject `promise` with `r`.
    - 2.3.3.3.3 If both `resolvePromise` and `rejectPromise` are called, or multiple calls to the same argument are made, the first call takes precedence, and any further calls are ignored.
    - 2.3.3.3.4 If calling `then` throws an exception `e`,
      - 2.3.3.3.4.1 If `resolvePromise` or `rejectPromise` have been called, ignore it.
      - 2.3.3.3.4.2 Otherwise, reject `promise` with `e` as the reason.
  - 2.3.3.4 If `then` is not a function, fulfill `promise` with `x`.
- 2.3.4 If `x` is not an object or function, fulfill `promise` with `x`.

`[[Resolve]](promise, x)`的运行依照如下步骤:

- 2.3.1 如果`promise`和`x`引用同一个对象, reject`promise` 并用`TypeError`作为reason.
- 2.3.2 如果`x`是promise, 采用它的state[3.4]:
  - 2.3.2.1 如果`x`正在pending, `promise`需要保持pending状态, 直到`x`被fulfilled或者rejected.
  - 2.3.2.2 如果/当`x`fulfilled, 用相同的value fulfill这个`promise`.
  - 2.3.2.3 如果/当`x`rejected, 用相同的reason reject这个`promise`.
- 2.3.3 相反的, 如果`x`是object或者function时,
  - 2.3.3.1 将`x.then`赋值给`then`. [3.5]
  - 2.3.3.2 If retrieving the property `x.then` results in a thrown exception `e`, reject `promise` with `e` as the reason.
  - 2.3.3.2 如果`x.then`的结果导致了一个异常`e`, 用`e`去reject这个`promise`.
  - 2.3.3.3 如果`then`是一个function, 调用这个function并且设置`x`作为上下文, `resolvePromise`和`rejectPromise`分别作为第一、第二个参数:
    - 2.3.3.3.1 如果/当`resolvePromise`被`y`调用, 那么运行`[[Resolve]](promise, y)`
    - 2.3.3.3.2 If/when `rejectPromise` is called with a reason `r`, reject `promise` with `r`.
    - 2.3.3.3.2 如果/当`rejectPromise`被一个reason`r`调用, 用`r`去reject这个`promise`.
    - 2.3.3.3.3 当`resolvePromise`和`rejectPromise`都调用, 或者其中一个被调用多次, 第一次的调用优先, 后续的调用忽略.
    - 2.3.3.3.4 当调用`then`抛出异常`e`时
      - 2.3.3.3.4.1 如果`resolvePromise`或`rejectPromise`已经调用过了, 忽略这个异常.
      - 2.3.3.3.4.2 否则, 用`e` reject 这个`promise`.
  - 2.3.3.4 If `then` is not a function, fulfill `promise` with `x`.
  - 2.3.3.4 如果`then`不是function, 用`x`去fulfill这个`promise`.
- 2.3.4 If `x` is not an object or function, fulfill `promise` with `x`.
- 2.3.4 如果`x`不是object或者function, 用`x`fulfill这个`promise`.

If a promise is resolved with a thenable that participates in a circular thenable chain, such that the recursive nature of `[[Resolve]](promise, thenable)` eventually causes `[[Resolve]](promise, thenable)` to be called again, following the above algorithm will lead to infinite recursion. Implementations are encouraged, but not required, to detect such recursion and reject `promise` with an informative `TypeError` as the reason. [3.6]

如果promise被一个thenable resolve, 这个thenable的循环调用链会变成`[[Resolve]](promise, thenable)`, 形成递归调用, 遵循以上的算法, 最终会形成无限递归. 所以, 在具体实现上我们鼓励(但不必须要求)检测这种无限递归的情况, 并且以一个具有提示信息的`TypeError`(*比如new TypeError('Infinite Loop Detected')*)作为reason去reject这个promise.[3.6]

# 3. Notes

- 3.1 Here “platform code” means engine, environment, and promise implementation code. In practice, this requirement ensures that `onFulfilled` and `onRejected` execute asynchronously, after the event loop turn in which `then` is called, and with a fresh stack. This can be implemented with either a “macro-task” mechanism such as `setTimeout` or `setImmediate`, or with a “micro-task” mechanism such as `MutationObserver` or `process.nextTick`. Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or “trampoline” in which the handlers are called.

- 3.2 That is, in strict mode `this` will be `undefined` inside of them; in sloppy mode, it will be the global object.

- 3.3 Implementations may allow `promise2 === promise1`, provided the implementation meets all requirements. Each implementation should document whether it can produce `promise2 === promise1` and under what conditions.

- 3.4 Generally, it will only be known that `x` is a true promise if it comes from the current implementation. This clause allows the use of implementation-specific means to adopt the state of known-conformant promises.

- 3.5 This procedure of first storing a reference to `x.then`, then testing that reference, and then calling that reference, avoids multiple accesses to the `x.then` property. Such precautions are important for ensuring consistency in the face of an accessor property, whose value could change between retrievals.

- 3.6 Implementations should not set arbitrary limits on the depth of thenable chains, and assume that beyond that arbitrary limit the recursion will be infinite. Only true cycles should lead to a `TypeError`; if an infinite chain of distinct thenables is encountered, recursing forever is the correct behavior.

# 3. 注释

- 3.1 Here “platform code” means engine, environment, and promise implementation code. In practice, this requirement ensures that `onFulfilled` and `onRejected` execute asynchronously, after the event loop turn in which `then` is called, and with a fresh stack. This can be implemented with either a “macro-task” mechanism such as `setTimeout` or `setImmediate`, or with a “micro-task” mechanism such as `MutationObserver` or `process.nextTick`. Since the promise implementation is considered platform code, it may itself contain a task-scheduling queue or “trampoline” in which the handlers are called.

- 3.1 这里, "平台码" 表示引擎代码(*e.g. V8)*/环境/promise本身实现的代码. 在实际情况中, 这保证了`onFulFilled`和`onRejected`一定是`then`被调用后, event loop交还主线程一个新的执行栈之后异步调用的. 这个特性可以通过`setTimeout`和`setImmediate`这种宏任务来实现, 也可以通过`MutationObserver`和`process.nextTick`这样的微任务来实现.(*这就是为啥一些考Promise和异步的面试题, 在浏览器和Node环境中打印顺序不同, 同一次执行栈中, 尽管微任务注册的可能比宏任务晚, 但最晚也在本次栈的末尾, 这里就不展开说这个了*). 由于promise的实现通常都考虑了平台码, 他自身就往往包含了任务调度队列或处理handlers的所谓“蹦床”一样的装置.
