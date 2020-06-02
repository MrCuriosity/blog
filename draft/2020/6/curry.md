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
