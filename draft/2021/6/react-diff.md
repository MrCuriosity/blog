### background

- 传统的两棵树 diff，时间复杂度在 O(n^3)，无疑这么高的复杂度在 DOM 更新中无法使用
- react 的 diff 有三个前提
  - DOM 操作中跨层级的移动少到忽略不计
    > tree diff, 新树比旧树在同一位置多了组件，连同其子树一起，直接新建，反之亦然；所以这个”前提“也是反模式，写 react 的时候应尽量避免
  - 不同的组件会产生不同的子树
    > 这是 component diff 的前提, 同样的，这个”前提“也是组件编写的反模式，如果有结构相似且会在业务中出现 diff 情况的组件，则应当抽象成一个，否则浪费开销
  - 同一层级的一组子节点，开发者可以通过 key 来进行区分
    > element diff，应该是 react diff 算法中最复杂的一步，一共有 3 种可能的操作
    >
    > - 插入: 新组件不在集合中，是全新的节点，插入节点
    > - 删除: 原来的集合有，更新的集合没有，删除节点
    > - 移动: [a, b, c, d] -> [a, d, b, c]无需在位置 1 的地方删除 b 再插入 d，这样后面的元素都会有操作，也是很大的性能浪费。由于有 key 的存在，react 判断集合中的元素本身没有变化，只是位置发生了改变

react diff 解决了复杂度的问题，但是带来了另外一个问题，diff 过程无法中断，会占用 JS 的主线程，使用户的交互和视图变得卡顿。于是 react 在 15.x 推出了 Fiber，fiber 使 diff 过程拆分成很小的任务穿插在高优先级的任务(交互、动画等)的间隙执行(requestIdleCallback)，fiber 可以看做 react diff 算法的威力加强版。

---

参考资料

- [组件更新流程一（调度任务）](https://github.com/KieSun/learn-react-essence/blob/master/%E7%BB%84%E4%BB%B6%E6%9B%B4%E6%96%B0%E6%B5%81%E7%A8%8B%E4%B8%80%EF%BC%88%E8%B0%83%E5%BA%A6%E4%BB%BB%E5%8A%A1%EF%BC%89.md)
- [完全理解 React Fiber](http://www.ayqy.net/blog/dive-into-react-fiber/)
