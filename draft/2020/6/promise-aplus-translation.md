> [原文](https://promisesaplus.com/)

**An open standard for sound, interoperable JavaScript promises—by implementers, for implementers.**
> 一个用以表示、使用JavaScript promises的开放标准——由开发者实施，供开发者使用.

A promise represents the eventual result of an asynchronous operation. The primary way of interacting with a promise is through its `then` method, which registers callbacks to receive either a promise’s eventual value or the reason why the promise cannot be fulfilled.
> 一个promise表示了一个异步操作的最终结果。promise的主要交互手段是其`then`方法，该方法注册了一系列回调用来接收promise的最终值，或是接收promise无法完成的原因。

This specification details the behavior of the `then` method, providing an interoperable base which all Promises/A+ conformant promise implementations can be depended on to provide. As such, the specification should be considered very stable. Although the Promises/A+ organization may occasionally revise this specification with minor backward-compatible changes to address newly-discovered corner cases, we will integrate large or backward-incompatible changes only after careful consideration, discussion, and testing.
> 这份规范详述了`then`方法的具体行为，提供了一份所有Promises/A+的实施方案可以参考的操作方法.正因为如此, 这份规范应当非常稳定。不过，Promises/A+组织可能会偶尔的做一些细微的向后兼容的修订，用以处理新发现的一些边角问题。我们只会在非常细致的考虑、讨论和测试之后才会集成一些大的或是向后兼容的改动。

Historically, Promises/A+ clarifies the behavioral clauses of the earlier [Promises/A](http://wiki.commonjs.org/wiki/Promises/A) proposal, extending it to cover de facto behaviors and omitting parts that are underspecified or problematic.
> 从历史角度来讲，Promises/A+规范明确了早期的[Promises/A](http://wiki.commonjs.org/wiki/Promises/A)提案具体行为条款.拓展并且使其覆盖了实际行为和先前不足的、有问题的部分。

Finally, the core Promises/A+ specification does not deal with how to create, fulfill, or reject promises, choosing instead to focus on providing an interoperable `then` method. Future work in companion specifications may touch on these subjects.
> 最终，Promises/A+规范的核心，并不涉及如何create/fulfill/reject promises, 取而代之的，我们聚焦在如何提供一个可互操作的`then`方法上。接下来，配套的规范会涉及这些主题。

# 1.Terminology
  1.1 “promise” is an object or function with a then method whose behavior conforms to this specification.

  1.2 “thenable” is an object or function that defines a then method.

  1.3 “value” is any legal JavaScript value (including undefined, a thenable, or a promise).

  1.4 “exception” is a value that is thrown using the throw statement.

  1.5 “reason” is a value that indicates why a promise was rejected.

> # 1. 术语

  - 1.1 "promise"的类型是`object`or`function`, 并且有一个`then`方法, `then`的行为遵循规范.

  - 1.2 "thenable"是一个`object`or`function`，它有一个`then`方法

  - 1.3 "value"可以是一切合法的JS value(包括`undefined`, `thenable`或者`promise`)

  - 1.4 "exception"是一个由`throw`声明抛出的value[*不一定是`error`*]

  - 1.5 "reason"用来表明promise被`reject`的原因

# 2. Requirements
## 2.1 Promise States
A promise must be in one of three states: pending, fulfilled, or rejected.

    2.1.1 When pending, a promise:

      2.1.1.1 may transition to either the fulfilled or rejected state.

    2.1.2 When fulfilled, a promise:

      2.1.2.1 must not transition to any other state.

      2.1.2.2 must have a value, which must not change.

    2.1.3 When rejected, a promise:

      2.1.3.1 must not transition to any other state.

      2.1.3.2 must have a reason, which must not change.

Here, “must not change” means immutable identity (i.e. ===), but does not imply deep immutability.

> # 2. 需求
## 2.1 Promise States
promise的状态必须是以下三选一: `pending`, `fulfilled`, or `rejected`.

  - 2.1.1 当状态是`pending`时, promise:

    - 2.1.1.1 可以转换成`fulfilled` or `rejected`.

  - 2.1.2 当状态是`fulfilled`, promise:

    - 2.1.2.1 无法转换成其他任何状态.

    - 2.1.2.2 必须有一个value，并且不可改变.

  - 2.1.3 当状态是`rejected`时, promise:

    - 2.1.3.1 无法转换成其他任何状态.

    - 2.1.3.2 必须有一个reason，并且不可改变.

这里, “must not change”指的是id不可变(`===`), 但不意味着deepEqual(*可以改变引用对变量部的东西*)

# The then Method
A promise must provide a `then` method to access its current or eventual value or reason.

A promise’s `then` method accepts two arguments:

```javascript
promise.then(onFulfilled, onRejected)
```
- 2.2.1 Both onFulfilled and onRejected are optional arguments:

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
