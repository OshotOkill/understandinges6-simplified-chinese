# 附录A - 其它改进


除了本书讲述的主要变化之外，ECMAScript 6 还对 JavaScript 做了一些虽小但很有意义的改进，包括简化整型的使用，新的数学运算方法，Unicode 标识符的轻微调整以及规范化 \_\_ proto \_\_ 属性。本附录包含以上所有的内容。

<br />

### 本章小结
* [整型的使用](#Working-with-Integers)
* [新的算术方法](#New-Math-Methods)
* [Unicode 标识符](#Unicode-Identifiers)
* [\_\_proto\_\_ 属性的规范化](#Formalizing-the-proto-Property)

<br />

### <a id="Working-with-Integers"> 整型的使用（Working with Integers） </a>


JavaScript 使用 IEEE 754 编码系统来表示整型（integer）和浮点类型（float），不过多年来它们引发了很多问题。虽然该门语言在幕后忍痛做了很多工作来让开发者不需要关心数型（number）编码的细节，然而问题仍旧会层出不穷的出现。为了解决它们，ECMAScript 6 做了一些工作来让整型更容易辨识和使用。

<br />

#### 判断整型（Identifying Integers）


首先，ECMAScript 6 添加了 Number.isInterger() 方法用来判断某个值在 JavaScript 中是否为整型。虽然 JavaScript 使用 IEEE 754 来表示所有的数型，但是浮点类型和整型的存储方式是有差异的。Number.isInteger() 方法在调用时 JavaScript 引擎会根据值的存储形式来判断参数是否为整型。这意味着外观上看起来像浮点类型的数字也会被当作整型来存储，将它传给 Number.isInteger() 会返回 true 。例如：

```js
console.log(Number.isInteger(25));      // true
console.log(Number.isInteger(25.0));    // true
console.log(Number.isInteger(25.1));    // false
```

该段代码中，向 Number.isInteger() 传入 25 和 25.0 都会返回 true，即使后者看起来是浮点类型。在 JavaScript 中仅仅的给数字添加小数点并不会让它自动转化为浮点类型。既然 25.0 实际上等于 25，那么它会被当作整型存储。然而 25.1 由于小数位不为 0，所以将被视为浮点类型。

<br />

#### 安全的整型类型（Safe Integers）


IEEE 754 只能精确表示 -2<sup>53</sup> 到 2<sup>53</sup> 之间的整型数字，在该 “安全” 范围之外，数字的二进制表达就不再唯一（译者：参考 IEEE 754 的整型规范）。这意味着 JavaScript 只能在 IEEE 754 所能精确表示的范围内保证准确度。例如，考虑下面的例子：


```js
console.log(Math.pow(2, 53));      // 9007199254740992
console.log(Math.pow(2, 53) + 1);  // 9007199254740992
```

该例的输出并不是笔误，然而这两个不同的数值确实由相同的 JavaScript 整型表示。当数值愈发脱离安全范围，这个效果就越明显。

ECMAScript 6 引入了 Number.isSafeInteger() 方法来检查整型是否在该语言所能精确表达的范围内，同时还添加了 Number.MAX_SAFE_INTEGER 和 Number.MIN_SAFE_INTEGER 属性来分别表示安全范围的上下边界。Number.isSafeInteger() 方法判断一个值是否为整型并位于安全的整数范围内，如下所示：

```js
var inside = Number.MAX_SAFE_INTEGER,
    outside = inside + 1;

console.log(Number.isInteger(inside));          // true
console.log(Number.isSafeInteger(inside));      // true

console.log(Number.isInteger(outside));         // true
console.log(Number.isSafeInteger(outside));     // false
```

inside 代表最大的安全整数，所以 Number.isInteger() 和 Number.isSafeInteger() 都会返回 true 。outside 是首个存在问题的整型值，所以他虽然是整型但并不安全。

在绝大部分情况下，你只会想使用安全范围内的整型来做 JavaScript 的运算和比较，所以使用 Number.isSafeInteger() 做输入验证会是个好主意。

<br />

### <a id="New-Math-Methods"> 新的算术方法（New Math Methods） </a>


游戏与图形计算重要性的俱增使得 ECMAScript 6 在给 JavaScript 引入了类型数组（typed array）的同时也意识到 JavaScript 引擎应该更有效率的处理很多数学运算。但是类似于 asm.js （工作 JavaScript 的子集上以提升性能）的优化策略需要更多的信息才能最快处理计算。例如，知道一个数字究竟是 32位整型还是 64 位 浮点类型对基于硬件（hardware-based）的操作来讲非常重要，而且要比基于软件（software-based）的操作要快很多。

于是，ECMAScript 6 给 Math 对象添加了几个方法来提升常用的数学运算的性能，同时包含大量数学运算的应用性能也会提升，比如图形编程（graphics program）。下面列出了这些新的方法：

* `Math.acosh(x)`  返回 x 的反双曲余弦值（Returns the inverse hyperbolic cosine of x）。
* `Math.asinh(x)`  返回 x 的反双曲正弦值（Returns the inverse hyperbolic sine of x）。
* `Math.atanh(x)`  返回 x 的反双曲正切值（Returns the inverse hyperbolic tangent of x）。
* `Math.cbrt(x)`  返回 x 的立方根（Returns the cubed root of x）。
* `Math.clz32(x)`  返回 x 以 32 位整型数字的二进制表达形式开头为 0 的个数（Returns the number of leading zero bits in the 32-bit integer representation of x）。
* `Math.cosh(x)`  返回 x 的双曲余弦值（Returns the hyperbolic cosine of x）。
* `Math.expm1(x)`  返回 e<sup>x</sup> - 1 的值（Returns the result of subtracting 1 from the exponential function of x）。
* `Math.fround(x)`  返回最接近 x 的单精度浮点数（Returns the nearest single-precision float of x）。
* `Math.hypot(...values)`  返回参数平方和的平方根（Returns the square root of the sum of the squares of each argument）。
* `Math.imul(x, y)`  返回两个参数之间真正的 32 位乘法运算结果（Returns the result of performing true 32-bit multiplication of the two arguments）。
* `Math.log1p(x)`  返回以 自然对数为底的 x + 1 的对数（Returns the natural logarithm of 1 + x）。
* `Math.log10(x)`  返回以 10 为底 x 的对数Returns the base 10 logarithm of x.
* `Math.log2(x)`  返回以 2 为底 x 的对数（Returns the base 2 logarithm of x）。
* `Math.sign(x)`  如果 x 为负数返回 -1；+0 和 -0 返回 0；正数则返回 1（Returns -1 if the x is negative, 0 if x is +0 or -0, or 1 if x is positive）。
* `Math.sinh(x)`  返回 x 的双曲正弦值（Returns the hyperbolic sine of x）。
* `Math.tanh(x)`  返回 x 的双曲正切值（Returns the hyperbolic tangent of x）。
* `Math.trunc(x)`  移除浮点类型小数点后面的数字并返回一个整型数字（Removes fraction digits from a float and returns an integer）。

详细这些新的方法和实线细节超出了本书的描述范围。不过如果你的应用需要这些计算的话，那么确保在自己实现它之前先查看一下有没有现成的方法。

<br />

### <a id="Unicode-Identifiers"> Unicode 标识符（Unicode Identifiers） </a>


ECMAScript 6 提供了比之前版本的更好的 Unicode 支持度，同时也增添了标识符的种类。在 EMCAScript 5 中，你已经可以使用 Unicode 的转义字符串作为标识符。例如：

```js
// 在 ECMAScript 5 and 6 中有效
var \u0061 = "abc";

console.log(\u0061);     // "abc"

// 等效于:
console.log(a);          // "abc"
```

该例中在 var 声明之后你可以同时使用 \u0061 或 a 来访问这个变量。在 EMCAScript 6 中，你还能使用转义的 Unicode code point 作为标识符，像这样：

```js
// 在 ECMAScript 5 and 6 中有效
var \u{61} = "abc";

console.log(\u{61});      // "abc"

// 等效于:
 console.log(a);          // "abc"
```


本例只是使用了等效的 code point 替换了 \u0061 。除此之外它的行为和上例相同。

另外，ECMAScript 6 还正式根据 [Unicode Standard Annex #31: Unicode Identifier and Pattern Syntax](http://unicode.org/reports/tr31/) 规范了合法标识符，遵循以下规则：

1. 第一个字符必须是 $， _\_，或任何包含 由 ID_start 派生的核心属性（derived core properties）的 Unicode symbol 。*
2. 随后的字符必须为 $，_\_，\u200c（零宽的 non-joiner 字符），\u200d（零宽的 joiner 字符），或任何包含 由 ID_start 派生的核心属性的 Unicode symbol 。*

ID_Start 和 ID_Continue 的派生核心属性由 Unicode Identifier and Pattern Syntax 定义以便规定一种正确的方式来使用和命名（如变量和域名）标识符。该规范并不属于 JavaScript 。

<br />

### <a id="Formalizing-the-proto-Property"> \_\_proto\_\_ 属性的规范化（Formalizing the \_\_proto\_\_ Property） </a>


在 ECMAScript 5 规范完成之前，一些 JavaScript 引擎实现了一个被称为 \_\_proto\_\_ 的属性来对 [[Prototype]] 施行 get 和 set 操作。实际上，\_\_proto\_\_ 是 Object.getPrototypeOf() 和 Object.setPrototypeo() 方法的先驱。指望所有的 JavaScript 引擎移除这个属性是不现实的（有些流行的库还在使用 \_\_proto\_\_），于是 ECMAScript 6 将 \_\_proto\_\_ 标准化。不过附录 B 中的 ECMA-262 版本对该规范有如下警告：

<br />

> 这些特性并不是 ECMAScript 语言的核心部分。开发者在书写新的 ECMAScript 代码时不该使用或者干脆认为它们并不存在。除非这些特性是浏览器中的一部分或服务于与兼容性相关的 ECMAScript 代码，否则 ECMAScript 不鼓励实现它们。

<br />

ECMAScript 规范更推荐使用 Object.getPrototypeOf() 和 Object.setPrototypeOf() 是因为 \_\_proto\_\_ 有如下特征：

1. \_\_proto\_\_在对象字面量中只能出现一次，否则将会抛出错误。这是唯一受限制的对象字面量属性。
2. 动态属性形式的 ["\_\_proto\_\_"] 表现类似于一般的属性而且并不会返回或赋值给当前对象。它具有非动态属性的所有特征，这意味着它也是动态属性唯一的例外。

虽然你应该避免使用 \_\_proto\_\_ 属性，但是它的规范定义很有意思。在实现了 ECMAScript 6 的引擎中，Object.prototype.\_\_proto\_\_ 是一个访问器属性，它的 get 和 set 方法分别调用 Object.getPrototypeOf() 和 Object.setPrototypeOf() 。这意味着使用 \_\_proto\_\_ 和 Object.getPrototypeOf() 和 Object.setPrototypeOf() 没有本质上的区别，除了 \_\_proto\_\_ 可以直接在对象字面量中使用。以下是它的使用方式：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// 原型为 person
let friend = {
    __proto__: person
};
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true
console.log(friend.__proto__ === person);               // true

// 将原型设置为 dog
friend.__proto__ = dog;
console.log(friend.getGreeting());                      // "Woof"
console.log(friend.__proto__ === dog);                  // true
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

与调用 Object.create() 来创建 friend 对象相反，该例在标准的对象字面量中添加了 \_\_proto\_\_ 属性。不过，当使用前者这种方式时，你需要为所有额外添加的对象属性定义完整的描述符

<br />
