# 函数（Functions）


函数在任何编程语言中都是非常重要的一部分，而且在 ECMAScript 6 之前，函数自 JavaScript 诞生以来并未过有较大的变化。这导致很多问题的积压，同时一些细微的差异容易诱发错误，或使用冗余的代码来实现非常基本的功能。

ECMAScript 6 中的函数相较而言是个大的跃进，着手调查了 JavaScript 开发者多年的抱怨和需求。最终的结果是在 ECMAScript 5 函数的基础之上做了几项改进，增强 JavaScript 的编程体验并减少诱发错误的因素。

#### 带默认参数的函数（Functions with Default Parameter Values）


JavaScript 的函数比较特殊的是可以接受任意个数的参数，完全无视函数声明中的参数个数。这允许你通过自行给未传值的参数赋默认值来定义带有不同参数的函数。本章将介绍 ECMAScript 6 及之前的 ECMAScript 版本如何实现默认参数，同时引出的还有 arguments 对象，参数表达式（expressions as parameters）及 TDZ 的另一形式。

#### ECMAScript 5 默认参数模拟（Simulating Default Parameter Values in ECMAScript 5）

在 ECMAScript 5 或更早的版本中，你可能会使用下面的这种模式来给参数添加默认值

``` default-paramter-in-es5
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // 其它部分

}
```

在这个例子中，timeout 和 callback 都是可选参数因为未传入实参给它们的情况下会使用各自的默认值。或逻辑操作符（logical OR operator, ||）当左侧的值为假的情况下总会返回右侧的运算数。既然未被传入实参的形参的缺省值为 undefined，或逻辑操作符就经常被用来给遗漏的参数添加默认值。不过这个方案有个瑕疵，如果给 timeout 传入的值为 0，那么 timeout 的值会被替换为 2000，因为 0 被默认为假值。 


下面这个例子中会对参数实行类型检查：

```
function makeRequest(url, timeout, callback) {

    timeout = (typeof timeout !== "undefined") ? timeout : 2000;
    callback = (typeof callback !== "undefined") ? callback : function() {};

    // 其它部分

}
```

虽然这种写法更加健壮，但是为实现这个基本需求书写的代码还是太多了。该写法作为一种公共的模式已被各种流行的 JavaScript 库中所使用。

<br />

#### ECMAScript 6 中的默认参数（Default Parameter Values in ECMAScript 6）


ECMAScript 6 使参数获得默认值变得更加方便，当未传入实参给形参时形参的初始值会被使用，如下所示：

``` deafult-parameter-in-es6
function makeRequest(url, timeout = 2000, callback = function() {}) {

    // 其余代码

}
```

该函数只期待传入第一个参数。其余两个参数有各自的默认值。这使得函数体更加小巧因为你不再需要添加额外的代码来检查是否有遗漏的参数值。

如果使用三个参数来调用 makeRequest()，那么所有的默认值都不会被使用，例如： 

```
// timeout 和 callback 使用默认值
makeRequest("/foo");

// callback 使用默认值
makeRequest("/foo", 500);

// 没有使用任何默认值
makeRequest("/foo", 500, function(body) {
    doSomething(body);
});
```

ECMAScript 6 会认为 url 参数是必须传入的，这就是三次调用都传入了 "/foo" 的原因。其余两个参数被视作是可选的。

可以任意指定一个函数参数的默认值，如果之后的参数未设定默认值也是允许的。例如下面是合法的：

```
function makeRequest(url, timeout = 2000, callback) {

    // 其余代码

}
```

在该条件下，timeout 的默认值仅当未提供给实参或传入 undefined 的情况下才会被使用，如下所示：

```
// 使用 timeout 的默认值
makeRequest("/foo", undefined, function(body) {
    doSomething(body);
});

// 使用 timeout 的默认值
makeRequest("/foo");

// 不使用 timeout 的默认值
makeRequest("/foo", null, function(body) {
    doSomething(body);
});
```

在提供参数值的时候, null 是有效的，所以 makeRequest 的第三次调用中 timeout 的默认值不会被使用。

<br />

#### 默认参数对 arguments 对象的影响 （How Default Parameter Values Affect the arguments Object）

需要记住的是当使用默认参数的时候 arguments 对象的表现是不同的。在 ECMAScript 5 的非严格模式下，arguments 对象会反映出所有被命名的参数的变化。下面的代码演示了该工作机制：

```
function mixArgs(first, second) {
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d";
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", "b");
```
输出:

```
true
true
true
true
```

arguments 对象在非严格模式下总是实行更新反映出命名参数的变化。因此当 first 和 second 变量获得新值之后，arguments[0] 和 arguments[1] 也同步更新，使得 === 比较的值为 true 。

然而在 ECMAScript 5 的严格模式下，这个机制被取消了，arguments 对象不会反映任何命名参数。如下依旧是上例中的函数，但现在处在严格模式下： 

```
function mixArgs(first, second) {
    "use strict";

    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a", " b");
```
调用 mixArgs 输出：

```
true
true
false
false
```
本次 first 和 second 的更改不会映射给 arguments，所以输出如你所愿。

当使用 ECMAScript 6 的默认参数时，arguments 对象的表现和 ECMAScript 5 的严格模式一致，不管函数是否显式设定为严格模式。默认参数的存在会使 arguments 对象对该命名参数解绑。这是个细微但重要的细节，因为arguments 对象的使用方式发生了变化。考虑如下的代码：

```
// 非严格模式
function mixArgs(first, second = "b") {
    console.log(arguments.length);
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
    first = "c";
    second = "d"
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}

mixArgs("a");
```
输出：

```
1
true
false
false
false
```

在本例中，arguments.length 的值为 1 是因为你只给 mixArgs() 提供了一个参数。这说明 arguments[1] 的值如我们所期待的那样是 undefined，同时 first 也等同于 arguments[0] 。改变 first 和 second 的值不会对 arguments 造成任何效果，不论是在非严格模式还是严格模式下，所以你可以期待 arguments 总是反映出函数的首次调用状态。

<br />

#### 默认参数表达式（Default Parameter Expressions）


或许默认参数最有意思的特性就是传给它的不一定是原始值。例如你可以执行一个函数并把返回值提供给它：

```
function getValue() {
    return 5;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
```

在这里，如果提供实参给 add() 第二个参数，那么 getValue() 会被调用并提取值作为默认参数。需要注意的是 getValue() 只会在未提供实参给 second 的情况下才会被调用，而不是在解析阶段。这意味着 getValue() 内部的不同写法可能会提供不同的默认值，例如：

```
let value = 5;

function getValue() {
    return value++;
}

function add(first, second = getValue()) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 6
console.log(add(1));        // 7
```
在这个例子中，value 的初始值是 5 并随着每次 getValue() 的调用而递增。首次调用 add(1) 返回的值为 6，再次调用则返回 7，因为 value 的值已经增加了。由于 second 的默认值总是在当前 add 函数被调用的情况下才被计算，所以 value 的值可以随时被改变。

在使用函数调用作为默认值的时候要注意，如果你忘了在函数名后面添加括号，例如在上例中写成 second = getValue ，那么你传入的是一个函数引用而不是函数调用后返回的值。

该特性又引出了另一种有趣的使用方式。你可以把前面的参数作为后面参数的默认值，如下所示：

```
function add(first, second = first) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

在该段代码中，first 作为默认值提供给了 second 参数，意味着只传入一个参数时两个参数获得了相同的值。所以 add(1, 1) 和 add(1) 返回的值都是 2。进一步讲，你可以把 first 作为参数传给另一个函数以便计算返回值赋给 second ：

```
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7
```

这个例子将 getValue(first) 的返回值赋给 second，所以 add(1, 1) 返回 2，add(1) 返回 7（1 + 6）。

默认参数引用其它参数的场景只发生在引用之前的参数，即前面的参数不能访问后面的参数，例如：

```
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // 抛出错误
```

调用 add(undefined, 1) 发生错误是因为 second 在 first 之后定义，所以 first 无法访问 second 的值，要想知道缘由，就需要重温一重要概念：暂存性死区。

