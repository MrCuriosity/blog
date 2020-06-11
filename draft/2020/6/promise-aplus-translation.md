> [原文](https://promisesaplus.com/)

**An open standard for sound, interoperable JavaScript promises—by implementers, for implementers.**
> 一个用以表示、使用JavaScript promises的开放标准——有开发者实施，供开发者使用.

A promise represents the eventual result of an asynchronous operation. The primary way of interacting with a promise is through its `then` method, which registers callbacks to receive either a promise’s eventual value or the reason why the promise cannot be fulfilled.
> 一个promise表示了一个异步操作的最终结果。promise的主要交互手段是其`then`方法，该方法注册了一系列回调用来接收promise的最终值，或是接收promise无法完成的原因。

This specification details the behavior of the `then` method, providing an interoperable base which all Promises/A+ conformant promise implementations can be depended on to provide. As such, the specification should be considered very stable. Although the Promises/A+ organization may occasionally revise this specification with minor backward-compatible changes to address newly-discovered corner cases, we will integrate large or backward-incompatible changes only after careful consideration, discussion, and testing.
> 这份规范详述了`then`方法的具体行为，提供了一份所有Promises/A+的实施方案可以参考的操作方法.正因为如此, 这份规范应当非常稳定。不过，Promises/A+组织可能会偶尔的做一些细微的向后兼容的修订，用以处理新发现的一些边角问题。我们只会在非常细致的考虑、讨论和测试之后才会集成一些大的或是向后兼容的改动。

Historically, Promises/A+ clarifies the behavioral clauses of the earlier [Promises/A](http://wiki.commonjs.org/wiki/Promises/A) proposal, extending it to cover de facto behaviors and omitting parts that are underspecified or problematic.
> 从历史角度来讲，Promises/A+规范明确了早期的[Promises/A](http://wiki.commonjs.org/wiki/Promises/A)提案具体行为条款.拓展并且使其覆盖了实际行为和先前不足的、有问题的部分。

Finally, the core Promises/A+ specification does not deal with how to create, fulfill, or reject promises, choosing instead to focus on providing an interoperable `then` method. Future work in companion specifications may touch on these subjects.
> 最终，Promises/A+规范的核心，并不涉及如何create/fulfill/reject promises, 取而代之的，我们聚焦在如何提供一个可互操作的`then`方法上。接下来，配套规范会涉及这些主题。
