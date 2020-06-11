## 手写一个Promise

> Promise是JS很重要的一部分（单线程异步编程）,利用其接口和规范来反推实现, 不失为加深理解的一个好方法.

先整理一个大体的骨架, 实现`then`和`resolve`的happy path, `reject`/`catch`/`race`/`all`等方法后续补完

  - 构造函数(`constructor`)接收一个回调(`register`)作为参数
    - 此回调又拥有两个参数(`resolve`和`reject`), 他们也是函数, 调用时机由用户决定(`register`内部, 同步或者异步)
    - 一个promise实例有三种状态: pending, fullfilled和rejected.
      - 启动后状态为pending, fullfilled和rejected各自对应resolve和reject调用后的状态
    - 一个promise实例有一个then方法
      - then方法也有两个参数, resolvedFn和rejectedFn
      - then可以被链式调用, resolvedFn的第一个参数是`register`里resolve的实参（首次被调用的then), 或者上一个then的resolvedFn的returnValue;所以链式调用then的实质是注册了一系列状态改变之后的回调函数, 到一个队列中, 而这个队列的执行是同步和异步混合的.
      - resolvedFn可以返回一个普通值, 也可以返回一个Promise实例（这里应该是难点, 没有这个特性, promise是实现会简单许多）

---

### 先写一个主体
```javascript
const PENDING = 'pending';
const FULLFILLED = 'fullfilled';
const REJECTED = 'rejected';

class MyPromise {
  constructor(register) {
    this.__state = PENDING; // 初始状态
    this.__successHandlersQueue = []; // then(resolvedFn)队列
    this.__value = undefined; // 当前值, 跟着__state变化
    const instance = this; // 拿一个变量固定一下this, 否则调用栈变化之后可能找不到实例

    const __resolve = function(value) {
      instance.__value = value;
      if (instance.__state === PENDING || instance.__state === FULLFILLED) {
        while (instance.__successHandlersQueue.length) {
          const param = instance.__value; // 上次的resolvedValue
          const handler = instance.__successHandlersQueue.shift(); // 后进先出, 依次调用
          const retOfThen = handler(param);
          instance.__value = retOfThen; // 把then的返回值传给下个then注册的回调使用
        }
      }
    };

    const __reject = function(reason) {};

    register(__resolve, __reject);
  }

  then(resolvedFn) {
    if (typeof resolvedFn === 'function') {
      this.__successHandlersQueue.push(resolvedFn);
    }

    return this; // 使其可以链式调用
  }
}
```
试一试
```javascript
const p = new MyPromise(resolve => {
  setTimeout(() => resolve('resolvedValue1'), 1000);
});
p.then(data => console.log('data in 1st then', data));
// MyPromise {__state: "pending", __successHandlersQueue: Array(1), __value: undefined}
// data in 1st then resolvedValue1
```
ok, 基本功能没问题, 再链式调用一下
```javascript
const p = new MyPromise(resolve => {
  setTimeout(() => resolve('resolvedValue1'), 1000);
});
p
  .then(data => {
    console.log('data in 1st then => ', data);
    return '1';
  })
  .then(data => console.log('data in 2nd then => ', data));
// data in 1st then resolvedValue1
// data in 2nd then 1
```
链式调用返回异步结果
```javascript
const p = new MyPromise(resolve => {
  setTimeout(() => resolve('resolvedValue1'), 1000);
});
p
  .then(data => {
    console.log('data in 1st then', data);
    return '1';
  })
  .then(data => {
    console.log('data in 2nd then', data);
    return new MyPromise(resolve => {
      setTimeout(() => {
        resolve('asyncResolvedValue1');
      }, 1000);
    });
  })
  .then(data => console.log('data in 3rd then', data))

// data in 1st then resolvedValue1
// data in 2nd then 1
// data in 3rd then MyPromise {__state: "pending", __successHandlersQueue: Array(0), __value: undefined}
```
3rd then 这里错了, 正确的打印应该是`asyncResolvedValue1`, 检查一下发现, 原因是
```javascript
const retOfThen = handler(param);
// handler(param)的返回值也可能是一个Promise, 而我们的意图是拿到其resolve的值
```
再处理一下__resolve函数
```javascript
const PENDING = 'pending';
const FULLFILLED = 'fullfilled';
const REJECTED = 'rejected';

class MyPromise {
  constructor(register) {
    this.__state = PENDING; // 初始状态
    this.__successHandlersQueue = []; // then(resolvedFn)队列
    this.__value = undefined; // 当前值, 跟着__state变化
    const instance = this; // 拿一个变量固定一下this, 否则调用栈变化之后可能找不到实例

    const __resolve = async function(value) {
      instance.__value = value;
      if (instance.__state === PENDING || instance.__state === FULLFILLED) {
        while (instance.__successHandlersQueue.length) {
          const param = instance.__value; // 上次的resolvedValue
          const handler = instance.__successHandlersQueue.shift();
          const retOfThen = handler(param);
          if (retOfThen instanceof MyPromise) {
            const asyncRetOfThen = await retOfThen;
            instance.__value = asyncRetOfThen;
          } else {
            instance.__value = retOfThen;
          }
        }
      }
    };

    const __reject = function(reason) {};

    register(__resolve, __reject);
  }

  then(resolvedFn) {
    if (typeof resolvedFn === 'function') {
      this.__successHandlersQueue.push(resolvedFn);
    }

    return this; // 使其可以链式调用
  }
}

var p = new MyPromise((resolve) => {
  setTimeout(() => {
    resolve(1);
  }, 1000);
});

p
.then(d1 => {
  console.log('d1 in then: ', d1);
  return new MyPromise(resolve => {
    setTimeout(() => resolve('thenPromiseValue 1'), 1000);
  });
})
.then(d2 => {
  console.log('d2 in then: ', d2);
  return 3;
})
.then(d3 => {
  console.log('d3 in then', d3);
  return new MyPromise(resolve => {
    setTimeout(() => resolve('thenPromiseValue 2'), 2000);
  })
})
.then(d4 => {
  console.log('d4 in then', d4);
})
// d1 in then:  1
// d2 in then:  thenPromiseValue 1
// d3 in then 3
// d4 in then thenPromiseValue 2
```
至此, 我们已经实现了
  - Promise的构造
  - then的链式调用(resolvedFn)
  - then的同步与异步的返回值传参(resolvedFn)

但这离一个符合 [Promise/A+规范](https://promisesaplus.com/) 的Promise类还差很多, 以及文章开头整理的规则也是仅仅凭借经验和直觉, 离规范所列的完整性和健壮性差很多
