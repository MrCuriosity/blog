# JavaScript中的浮点数运算

#### 背景
> 计算机硬件只能处理二进制数据, 所以数学上的小数, 在计算机里的处理比整数的处理复杂许多. 通常，计算机采用([binary](https://floating-point-gui.de/formats/binary/).[floating-point](https://floating-point-gui.de/formats/fp/))的形式来表示小数(这段太硬，暂时看不太懂，有兴趣的可以点链接自己翻译);
```js
0.1 + 0.2
// output 0.30000000000000004
```
---
#### 如何避免rounding error
> - 如果你真的特别需要精确的计算(特别是处理钱相关的业务), 可以采用一些特别的[十进制数据类型](https://floating-point-gui.de/formats/exact/)
> - 如果仅仅是简单的不想看到小数部分，展示的时候做一些格式化处理.
> - 如果不想引入多余的所谓十进制库, 有个替代的办法就是全用整数处理，比如，处理钱的时候先乘以100，再去处理每一"分".
---
> 名词解释
> - **integer** - 整数
> - **rounding number** - 概数/约整数, 通俗的被认为是以一个或多个0结尾的整数, 可以认为是四舍五入的10的倍数.
---
> 参考资料
> - [wiki-integer](https://en.wikipedia.org/wiki/Integer)
> - [wiki-round_number](https://en.wikipedia.org/wiki/Round_number)
> - [floating-point-guide](https://floating-point-gui.de/)
