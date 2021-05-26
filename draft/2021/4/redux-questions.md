- action 只是一个包含了`type`和可能有数据(`payload`)的简单对象，我们为什么需要 actionCreator 而不是直接写对象.
- reducer 为什么需要是纯函数.
- 纯函数之于 redux 的意义

  - 更易于追踪、测试、debug
  - 可预测(predictable)
  - time travel, development, test(react-devtool)

---

- state tree(store) is read-only.
- 每一次已发生的 action 对应一份独立的 state，不同时间的 state 之间不应该出现相同的引用(比如 previousState.a.list 与 currentState.a.list 都是同一份内存引用，这是不符合 redux 设计原则)
- 所有 action 都可以被有序排列，不会出现乱序的竞争情况，因为所有 action 都是同步的(异步 action 实际上是副作用回调+action).

基于以上 3 点，redux 实现 time-travel, replay 等功能很方便，易测试，易回溯(record).
