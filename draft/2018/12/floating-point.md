# JavaScript中的浮点数运算

### 背景
计算机硬件只能处理二进制数据, 所以数学上的小数, 在计算机里的处理比整数的处理复杂许多. 通常，计算机采用([binary](https://floating-point-gui.de/formats/binary/).[floating-point](https://floating-point-gui.de/formats/fp/))的形式来表示小数(这段太硬，暂时看不太懂，有兴趣的可以点链接自己翻译);
```js
0.1 + 0.2
// output 0.30000000000000004
```
出现类似的无限小数，换算成二进制之后，显然计算机不可能用无限的内存空间去存储无限小数，于是根据[IEEE754 STANDARD](https://en.wikipedia.org/wiki/IEEE_754), JS中的Number都是64位bit来存储(double), 一位符号位S, 11位指数位E, 52位尾数位M
```md
V = (-1)^S + 2^E + M
```
---
### 调试
根据标准, M可取值范围是52位二进制, 转换为十进制是**9.007199254740992**, 一共16位, 所以, 当四舍五入出现问题时, 可以将出问题的数字打印17位以上看看实际是多少, 例如
```js
0.3.toPrecision(17)
'0.29999999999999999'

3.005.toFixed(2)
'3.00' // not 3.01
3.005.toPrecision(17)
'3.0049999999999999'
```
原因一目了然

---
### 如何避免rounding error
- 如果你真的特别需要精确的计算(特别是处理钱相关的业务), 可以采用一些特别的[十进制数据类型](https://floating-point-gui.de/formats/exact/)
- 如果仅仅是简单的不想看到小数部分，展示的时候做一些格式化处理.
- 如果不想引入多余的所谓十进制库, 有个替代的办法就是全用整数处理，比如，处理钱的时候先乘以100，再去处理每一"分".

---
### 名词解释
- **significand** - 底数, 有时也写作**sign**
- **exponent** - 指数
- **mantissa** - 尾数, 也就是科学计数法的常数
- **integer** - 整数
- **rounding number** - 概数/约整数, 通俗的被认为是以一个或多个0结尾的整数, 可以认为是四舍五入的10的倍数.

---

### 参考资料
- [wiki-integer](https://en.wikipedia.org/wiki/Integer)
- [wiki-round_number](https://en.wikipedia.org/wiki/Round_number)
- [floating-point-guide](https://floating-point-gui.de/)
- [binary-convert](http://www.binaryconvert.com/convert_double.html)
- [JavaScript 浮点数陷阱及解法](https://github.com/camsong/blog/issues/9)
- [number-precision](https://github.com/nefe/number-precision)
