# 附录B - 领悟 ECMAScript 7（2016）


ECMAScript 6 的正式发布耗时四年，在那之后，TC-39 发觉这么长的发布周期很不合理。于是，他们决定将发布周期固定为一年以便让新的语言特性能尽早地投入到开发中。

发布周期的缩短意味着新的 ECMAScript 版本中的特性会少于 ECMAScript 6 。为了适应这种变化，新版本将不再以显著的版本号，而是发布的年份来命名。因此 ECMAScript 6 也被称为 ECMAScript 2015，同时 ECMAScript 7 的正式名称为 ECMAScript 2016 。TC-39 也预计未来的 ECMAScript 版本全部使用年份命名。

ECMAScript 2016 在 2016 年 3 月正式发布并向该语言添加了三项内容：一个新的数学运算符，数组方法和语法错误。本附录中就涵盖所有内容。

<br />

### 本章小结
* [求幂运算符](#The-Exponentiation-Operator)
* [Array.prototype.includes() 方法](#The-Array-prototype-includes-Method)
* [函数作用域中严格模式的变更](#Change-to-Function-Scoped-Strict-Mode)

<br />

### <a id="The-Exponentiation-Operator"> 求幂运算符（The Exponentiation Operator） </a>


ECMAScript 2016 对 JavaScript 语法的唯一改进就是引入了求幂运算符，用来改进对基数的指数运算。JavaScript 已经有 Math.pow() 方法可以求幂，不过 JavaScript 也是为数不多的不能使用运算符而只能通过调用方法来完成求幂操作的语言之一（同时一些开发者也指出使用操作符的可读性更强）。

求幂运算符是两颗星号（**），左侧为基数，右侧为指数。例如：

```js
let result = 5 ** 2;

console.log(result);                        // 25
console.log(result === Math.pow(5, 2));     // true
```

该例的计算 5 的 2 次方，结果 25 。你也可以使用 Math.pow() 并得出相同的结果。

<br />

#### 运算符的优先级（Order of Operations）


求幂运算符的优先级是在二元运算符中最高的（但是低于一元运算符）。这意味着在复合运算中它会被优先计算，如下所示：

```js
let result = 2 * 5 ** 2;
console.log(result);        // 50
```

5 与 2 的求幂运算先发生，并再与 2 相乘得出最终结果为 50 。

<br />

#### 操作数的限制（Operand Restriction）


相比其它运算符，求幂运算符有一个独特的限制。运算符左侧不能为除了 ++ 或 -- 之外的一元表达式。例如，下面的示例会抛出错误。

```js
// 语法错误
let result = -5 ** 2;
```

该例中，-5 会抛出语法错误，因为它包含歧义。- 作用的到底是 5 还是 5 ** 2 的结果呢？禁止左侧的一元表达式消除了这个歧义。为了阐明你的意图，你需要给 -5 或 5 ** 2 像下例这样添加圆括号：

```js
// 没问题
let result1 = -(5 ** 2);    // 等于 -25

// 同样也没问题
let result2 = (-5) ** 2;    // 等于 25
```

如果你给求幂表达式添加括号，那么 - 会作用于它的结果。当你给 -5 添加括号时，意味着你想将它作为乘数。

如果在求幂运算符左侧使用 ++ 或 -- ，圆括号就没有必要使用，因为它们作用于操作数的行为十分明确。++ 或 -- 作为前缀时会在任何其它的操作发生之前修改操作数的值，而作为后缀则会在整个表达式计算完成之前不会有任何操作。这两种在求幂运算符左侧的用例都很安全，如下所示：

```js
let num1 = 2,
    num2 = 2;

console.log(++num1 ** 2);       // 9
console.log(num1);              // 3

console.log(num2-- ** 2);       // 4
console.log(num2);              // 1
```

该例中，num1 在求幂操作之前值发生了增长，所以 num1 的值为 3 并在求幂后结果为 9 。对于 num2 来讲，在求幂操作完成前它的值始终为 2，之后它的值才降为 1 。

<br />

### <a id="The-Array-prototype-includes-Method"> Array.prototype.includes() 方法（The Array.prototype.includes() Method） </a>


你可能会想起 ECMAScript 6 添加的查看子字符串是否存在的 String.prototype.include() 方法。起初，ECMAScript 6 也想要引入 Array.prototype.includes() 方法来延续字符串与数组相似的传统。但是 Array.prototype.includes() 的相关规范没有在 ECMAScript 6 发布的期限内定稿，所以 Array.prototype.includes() 才会被推迟到 ECMAScript 2016 。

<br />

#### 如何使用 Array.prototype.includes() （How to Use Array.prototype.includes()）


Array.prototype.includes() 方法接收两个参数：要搜索的值和开始搜索的初始索引。当第二个参数被提供时，includes() 会从该索引处往后搜索（默认的初始索引值为 0）。如果想要搜索的值在数组中存在就会返回 true，否则为 false。例如：

```js
let values = [1, 2, 3];

console.log(values.includes(1));        // true
console.log(values.includes(0));        // false

// start the search from index 2
console.log(values.includes(1, 2));     // false
```

这里，调用 values.includes() 并传入 1 会返回 true，而传入 0 返回的是 false ，因为 0 在数组中不存在。当第二个参数被添加后，数组会从索引值为 2 的位置开始搜索（包括的值为 3）并返回 false，因为从索引的位置到数组末尾不存在 1 这个值。

<br />

#### 值的比较（Value Comparison）


includes() 方法中使用 === 操作符来做值的比较。不过有一点除外： NaN 被认为和另一个 NaN 相等，即使 Nan === Nan 的计算结果为 false 。这和 indexOf() 方法的行为不同，因为后者严格使用 === 判断。为了阐明这项差异，考虑如下的代码：

```js
let values = [1, NaN, 2];

console.log(values.indexOf(NaN));       // -1
console.log(values.includes(NaN));      // true
```

values.indexOf() 方法返回 -1，即使 NaN 存在于 values 数组中。另一方面，由于比较操作符开了小差，values.includes() 会返回 true 。

<br />

> **注意**: 如果你只想知道某个值在数组是否存在而不关心它的索引位置，我推荐使用 includes()，因为它对待 NaN 的方式和 indexOf() 方法不同。如果你想获取某个值在数组中的位置，那么必须使用 indexOf() 方法。

<br />

在实现中另一项怪异（quirk）之处是 +0 和 -0 被认为是相同的。在这个方面，indexOf() 和 includes() 的行为是一致的：

```js
let values = [1, +0, 2];

console.log(values.indexOf(-0));        // 1
console.log(values.includes(-0));       // true
```

在这里，当传入 -0 作为参数时，indexOf() 和 includes() 会匹配 +0，因为它们认为这两个值是相同的。注意这和 Object.is() 方法的行为是相反的，后者认为 +0 和 -0 是不同的值。

<br />

### <a id="Change-to-Function-Scoped-Strict-Mode"> 函数作用域中严格模式的变更（Change to Function-Scoped Strict Mode） </a>


ECMAScript 5 引入了严格模式，此时它相比 ECMAScript 6 的严格模式要简单些。先不管这些，ECMAScript 6 仍然允许你使用 "use strict" 在全局作用域（使所有的代码运行在严格模式下）或函数作用域（只有该函数内部为严格模式）内定义严格模式。后者在 ECMAScript 6 中由于参数定义的多样性会存在一些问题，特别是当你使用解构和默认参数的时候。为了理解这个问题，考虑如下的代码：

```js
function doSomething(first = this) {
    "use strict";

    return first;
}
```

在这里，该命名参数首先由默认参数值 this 赋值。那么你认为 first 的值是什么呢？ ECMAScript 6 规范指出 JavaScript 引擎在此时应该视参数运行在严格模式下，所以 first 应该为 undefind 。然而，在函数内部存在 "use strict" 的情况下，要将参数也一并运行在严格模式有一定的困难，因为参数默认值也可以是个函数。该问题的存在使得大部分 JavaScript 引擎都没有实现这个特性（所以 this 的值可能会是全局对象）。

咎于该实现的复杂性，ECMAScript 2016 规定如果函数使用了解构参数或默认参数，那么内部使用 "use strict" 声明将是违法的。只有当参数列表是简单的，并没有解构和默认值的情况下，函数主体才能有 "use strict" 出现。下面有一些示例：

```js
// 没有问题 - 使用了简单的参数列表
function okay(first, second) {
    "use strict";

    return first;
}

// 语法错误
function notOkay1(first, second=first) {
    "use strict";

    return first;
}

// 语法错误
function notOkay2({ first, second }) {
    "use strict";

    return first;
}
```

你依旧可以在函数只包含简单的参数列表的条件下使用 "use strict"，这也是 okay() 正常运行的原因（ECMAScript 5 同理）。notOkay1() 和 notOkay2() 函数都会抛出语法错误，因为它们分别使用了默认参数和解构参数。

总的来讲，这项改进同时解决了 JavaScript 开发者的困惑与 JavaScript 引擎的实现难题。

<br />
