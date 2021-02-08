### Javascript 中的原型链总结

JS 中的对象 ('object'类型, 除了 `null`) 上的方法如果都通过实例方法来实现, 会占用海量的内存, 所以有一些方法是通过原型链来实现的, 比如

```javascript
var o = { name: "foo" };
o.hasOwnProperty("name"); // true
```

可以看到, 字面量申明 `o` 时并没有给它 `hasOwnProperty` 方法, 但其却可以调用这个方法, 这里就引出了 JS 中原型链最常见的一种应用: **如果对象没有实例方法, 可以通过原型链向上查找该方法, 直到原型链完结**

这里出现了几个问题

- 什么是原型链
- 原型链的终点是什么

### 1. 原型链

JS 中的原型链有一个基本规律：**实例对象的`.__proto__`键指向其构造函数的`.prototype`建, 都是引用类型 (一个对象, `.prototype` 其实就是原型对象), 当在其构造函数的原型对象上无法查找到该方法时, 继续向上查找(`.prototype.__proto__`)**

```javascript
var o = { name: "foo" };
o.__proto__ === Object.prototype; // true

/** 复杂一点的例子 **/
function Vehicle(brand) {
  this.brand = brand;
}

var gtr = new Vehicle("nissan");
gtr.hasOwnProperty("brand"); // true;
gtr.__proto__.hasOwnProperty("hasOwnProperty"); // false
gtr.__proto__.__proto__.hasOwnProperty("hasOwnProperty"); // true
// 这个时候往上再查找了一层, 因为第一层(gtr的构造函数Vehicle的原型对象并没有`hasOwnProperty`属性, 所以找到了Vehicle的构造函数Object, 也就是 gtr.__proto__.__proto__ === Object.prototype
```

其实, 原型链和构造函数链互为表里线, 引用一下 MDN 上 `instanceof` 操作符的定义:

> `instanceof` 运算符用来检测 `constructor.prototype` 是否存在于参数 object 的原型链上。

- 构造函数链的终点是 `Object`
- 原型链的终点是 `Object.prototype.__proto__`

### 2. 原型链的终点

这里就是 <del>梦开始</del> 一团浆糊的地方, 私以为是 Brendan Eich 设计的时候没想那么多, 导致在原型链终点的时候套娃.
通过上例我们其实可以看到原型链的终点是`null`, 这句话针对 plain object 很通顺, 但是
加入了函数以后, 事情变得复杂了起来, 所以我们将函数纳入讨论, 问题几乎变成了:

**Q: JS 中引用类型的起源？**

**A: 引用类型起源于两个对象, `Object` 与 `Function`, 并且 `Object` 与 `Function` 与其各自的构造函数和原型对象有如下特别的指向关系:**

```javascript
Function instanceof Object; // true
Object instanceof Function; // true
Function instanceof Function; // true
Object instanceof Object; // true

/** Function是二者的构造函数 **/
Function.constructor === Function; // true
Object.constructor === Function; // true

/** Object.prototype则是二者的原型对象 **/
Function.__proto__ === Function.prototype; // true
Object.__proto__ === Function.prototype; // true

Object.__proto__.__proto__ === Object.prototype &&
  Function.__proto__.__proto__ === Object.prototype; // true, 所以实例对象或者函数都继承了Object.prototype上的方法

/** 再往上 **/
Object.prototype.__proto__; // null
/** 原型链结束 **/
```

### 3. 总结

- 一个对象的原型链查找, 通过其构造函数的原型对象向上查找实现, 即：
  - `obj.constructor.prototype.constructor.prototype` ...
  - 或 `obj.__proto__.__proto__` ..., 二者等价
- `instanceof` 运算符用来检测 `constructor.prototype` 是否存在于参数 `object` 的原型链上
- `Function` 是 `Function` 和 `Object` 的构造函数
- `Object.__proto__.__proto__ === Object.prototype && Function.__proto__.__proto__ === Object.prototype`
- `Object.prototype === null`, 原型链结束
