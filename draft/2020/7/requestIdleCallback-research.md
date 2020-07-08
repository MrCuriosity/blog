# Questions
- `requestIdleCallback`为什么最大的空闲时间是50ms?最后我在W3C规范的相应章节找到了答案 - [why50](https://www.w3.org/TR/2017/PR-requestidlecallback-20171010/#why50)
- `requestIdleCallback`总是与时间循环的帧数频率相同吗, 并不是:
```javascript
var now = +new Date();
var current = undefined;
var count = 0;
var frame = 0;
var rAF = function() {
  count++;
  requestAnimationFrame(() => {
    for (let i = 0; i < 1000000; i++) {
      var temp = `test - ${i}`;
    };
    current = +new Date();
    var duration = current - now;
    console.log(`duration in frame - ${frame}`, duration);
    now = +new Date();
    if (count < 10) {
      frame++;
      rAF();
    };
  });
  requestIdleCallback(deadline => {
    console.log('didTimeout', deadline.didTimeout);
    console.log('timeRemaining', deadline.timeRemaining());
  }, { timeout: 1000 });
}

rAF();

// 在console里运行几次, 可以看到`duration in frame - ${i}`和`timeRemaining x`不一定是成对出现, 也就是说有些帧并不够去执行一个callback, 则不执行, 等下一帧主线程的主要任务做完, 再来看idle的时间够不够.
```
- `requestIdleCallback`的回调如果是比较大的任务, 会让每一帧超时(> 50ms)吗？会:
```javascript
var previous = +new Date();
var current = undefined;
var frame = 0;
var someBigSyncTask = n => new Array(n).fill(true).map((v, i) => `test - ${i}`);
var test = function() {
  requestAnimationFrame(() => {
    current = +new Date();
    var duration = current - previous;
    console.log(`duration in frame - ${frame}`, duration);
    previous = +new Date();
    if (frame < 10) {
      frame++;
      test();
    };
  });
  requestIdleCallback(deadline => {
    if (frame === 3 || frame === 7) {
      var ret = someBigSyncTask(1000000);
      console.log(`ret - ${frame}`, ret);
    }
    console.log(`didTimeout - ${frame}`, deadline.didTimeout);
    console.log(`timeRemaining - ${frame}`, deadline.timeRemaining());
  });
}

test();
```
可以看到, 打印结果中的第**3**和第**7**次的duration打印都有点离谱, 因为我们执行了比较耗时的任务, 但后续的正常打印, 又回归正常。所以`requestIdleCallback`的剩余50ms的时间是浏览器规定的(*这个解释在[why50](https://www.w3.org/TR/2017/PR-requestidlecallback-20171010/#why50)里也能找相应的叙述*)。好了, 所以`requestIdleCallback`能拿来干啥？浏览器帮我们提供了可以大致知道是否空闲的API, 但很明显我们不能拿来干很耗时的阻塞任务, 它的callbacks应当是一些能预见的零碎的小任务, 其实react-fiber就是基于此实现的(每个虚拟fiber-node的工作量可预见的都不会太大, 这是fiber能这么做的前提).

- `setTimeout`和`requestAnimationFrame`各自产生的帧的剩余时间调度, 在对于`requestIdleCallback`时的处理一样吗？不一样
```javascript
function test(register) {
  var previous = +new Date();
  var current = undefined;
  var frame = 0;
  var func = () => {
    register(() => {
      current = +new Date();
      var duration = current - previous;
      console.log(`duration in frame - ${frame}`, duration);
      previous = +new Date();
      if (frame++ < 20) {
        console.log(`going to register idle callback in frame - `, frame - 1);
        requestIdleCallback(deadline => {
          console.log(`remaining millionseconds`, deadline.timeRemaining());
        });
        func();
      }
    });
  }
  func();
}

test(setTimeout);
// ---
// duration in frame - 0 2
// going to register idle callback in frame -  0
// duration in frame - 1 1
// going to register idle callback in frame -  1
// remaining millionseconds 49.68500000000001
// duration in frame - 2 2
// going to register idle callback in frame -  2
// duration in frame - 3 1
// going to register idle callback in frame -  3
// duration in frame - 4 5
// going to register idle callback in frame -  4
// duration in frame - 5 5
// going to register idle callback in frame -  5
// duration in frame - 6 5
// going to register idle callback in frame -  6
// duration in frame - 7 4
// going to register idle callback in frame -  7
// duration in frame - 8 5
// going to register idle callback in frame -  8
// duration in frame - 9 5
// going to register idle callback in frame -  9
// duration in frame - 10 5
// going to register idle callback in frame -  10
// duration in frame - 11 6
// going to register idle callback in frame -  11
// duration in frame - 12 5
// going to register idle callback in frame -  12
// remaining millionseconds 49.45
// remaining millionseconds 49.290000000000006
// remaining millionseconds 49.11500000000001
// remaining millionseconds 48.96
// remaining millionseconds 48.815000000000005
// ...

test(requestAnimationFrame);

// duration in frame - 0 1
// going to register idle callback in frame -  0
// remaining millionseconds 13.840000000000002
// duration in frame - 1 16
// going to register idle callback in frame -  1
// remaining millionseconds 14.220000000000002
// duration in frame - 2 16
// going to register idle callback in frame -  2
// remaining millionseconds 14.755
// duration in frame - 3 17
// going to register idle callback in frame -  3
// remaining millionseconds 14.245000000000001
// duration in frame - 4 16
// going to register idle callback in frame -  4
// remaining millionseconds 14.325000000000001
// duration in frame - 5 16
// going to register idle callback in frame -  5
// remaining millionseconds 15.250000000000002
// duration in frame - 6 18
// going to register idle callback in frame -  6
// remaining millionseconds 14.105
// duration in frame - 7 17
// going to register idle callback in frame -  7
// remaining millionseconds 13.485000000000001
// duration in frame - 8 15
// going to register idle callback in frame -  8
// remaining millionseconds 14.71
// duration in frame - 9 16
// going to register idle callback in frame -  9
// remaining millionseconds 14.885
// duration in frame - 10 16
// going to register idle callback in frame -  10
// remaining millionseconds 16.060000000000002
// duration in frame - 11 17
// going to register idle callback in frame -  11
// remaining millionseconds 15.240000000000002
// duration in frame - 12 17
// going to register idle callback in frame -  12
// remaining millionseconds 14.535000000000002
// duration in frame - 13 17
// going to register idle callback in frame -  13
// remaining millionseconds 14.535000000000002
// ...
```
可以看到, 测试`setTimeout`时, duration为其默认最短间隔左右(4ms), 但idle callback并不一定在其每一帧的末尾执行, 到了第12帧的时候一股脑执行了很多堆积的idle callbacks; 当测试`requestAnimationFrame`时, 打印非常均匀, 全部成对出现, 跟`requestIdleCallback`的[W3C定义](https://www.w3.org/TR/2017/PR-requestidlecallback-20171010/)一致.
