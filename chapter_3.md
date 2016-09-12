# 函数（Functions）


函数在任何编程语言中都是非常重要的一部分，而在 ECMAScript 6 之前，函数自 JavaScript 诞生以来并未过有较大的变化。这导致很多问题的积压，同时一些细微的差异容易诱发错误，或使用冗余的代码来实现非常基本的功能。

ECMAScript 6 中的函数相较而言是个大的跃进，着手调查了 JavaScript 开发者多年的抱怨和需求。最终的结果是在 ECMAScript 5 函数的基础之上做了几项改进，增强 JavaScript 的编程体验并减少诱发错误的因素。

<br />

### 本章小结
* [带默认参数的函数](#Functions-with-Default-Parameter-Values)
* [未命名参数](#Working-with-Unnamed-Parameters)
* [增强的 Function 构造函数](#Increased-Capabilities-of-the-Function-Constructor)
* [扩展运算符](#The-Spread-Operator)
* [ECMAScript 6 中的 name 属性](#ECMAScript-6-name-Property)
* [明确函数的双重用途](#Clarifying-the-Dual-Purpose-of-Functions)
* [块级函数](#Block-Level-Functions)
* [箭头函数](#Arrow-Functions)
* [尾调用优化](#Tail-Call-Optimization)
* [总结](#Summary)

<br />

### <a id="Functions-with-Default-Parameter-Values"> 带默认参数的函数（Functions with Default Parameter Values） </a>


JavaScript 的函数比较特殊的是可以接受任意个数的参数，完全无视函数声明中的参数个数。这允许你通过自行给未传值的参数赋默认值来定义带有不同参数的函数。本章将介绍 ECMAScript 6 及之前的 ECMAScript 版本如何实现默认参数，同时引出的还有 arguments 对象，参数表达式（expressions as parameters）及 TDZ 的另一形式。

<br />

#### ECMAScript 5 默认参数模拟（Simulating Default Parameter Values in ECMAScript 5）

在 ECMAScript 5 或更早的版本中，你可能会使用下面的这种模式来给参数添加默认值

```js
function makeRequest(url, timeout, callback) {

    timeout = timeout || 2000;
    callback = callback || function() {};

    // 其它部分

}
```

在这个例子中，timeout 和 callback 都是可选参数因为未传入实参给它们的情况下会使用各自的默认值。或逻辑操作符（logical OR operator, ||）当左侧的值为假的情况下总会返回右侧的运算数。既然未被传入实参的形参的缺省值为 undefined，或逻辑操作符就经常被用来给遗漏的参数添加默认值。不过这个方案有个瑕疵，如果给 timeout 传入的值为 0，那么 timeout 的值会被替换为 2000，因为 0 被默认为假值。


下面这个例子中会对参数实行类型检查：

```js
function makeRequest(url, timeout, callback) {

    timeout = (typeof timeout !== "undefined") ? timeout : 2000;
    callback = (typeof callback !== "undefined") ? callback : function() {};

    // 其它部分

}
```

虽然这种写法更加健壮，但是为实现这个基本需求书写的代码还是太多了。该写法作为一种公共的模式已被各种流行的 JavaScript 库中所使用。

<br />

#### ECMAScript 6 中的默认参数（Default Parameter Values in ECMAScript 6）


ECMAScript 6 中函数参数能更方便地获取默认值，当未传入实参给形参时形参的默认值会被使用，如下所示：

```js
function makeRequest(url, timeout = 2000, callback = function() {}) {

    // 其余代码

}
```

该函数只期待传入第一个参数。其余两个参数有各自的默认值。这使得函数体更加小巧因为你不再需要添加额外的代码来检查是否有遗漏的参数值。

如果使用三个参数来调用 makeRequest()，那么所有的默认值都不会被使用，例如：

```js
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

```js
function makeRequest(url, timeout = 2000, callback) {

    // 其余代码

}
```

在该条件下，timeout 的默认值仅当未提供给实参或传入 undefined 的情况下才会被使用，如下所示：

```js
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

```js
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

```js
true
true
true
true
```

arguments 对象在非严格模式下总是实行更新反映出命名参数的变化。因此当 first 和 second 变量获得新值之后，arguments[0] 和 arguments[1] 也同步更新，使得 === 比较的值为 true 。

然而在 ECMAScript 5 的严格模式下，这个机制被取消了，arguments 对象不会反映任何命名参数。如下依旧是上例中的函数，但现在处在严格模式下：

```js
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

```js
true
true
false
false
```

本次 first 和 second 的更改不会映射给 arguments，所以输出如你所愿。

当使用 ECMAScript 6 的默认参数时，arguments 对象的表现和 ECMAScript 5 的严格模式一致，不管函数是否显式设定为严格模式。默认参数的存在会使 arguments 对象对该命名参数解绑。这是个细微但重要的细节，因为arguments 对象的使用方式发生了变化。考虑如下的代码：

```js
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

```js
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

```js
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

```js
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

```js
function add(first, second = first) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

在该段代码中，first 作为默认值提供给了 second 参数，意味着只传入一个参数时两个参数获得了相同的值。所以 add(1, 1) 和 add(1) 返回的值都是 2。进一步讲，你可以把 first 作为参数传给另一个函数以便计算返回值赋给 second ：

```js
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

```js
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // 抛出错误
```

调用 add(undefined, 1) 发生错误是因为 second 在 first 之后定义，所以 first 无法访问 second 的值，要想知道缘由，就需要重温一重要概念：暂存性死区。

<br />

#### 默认参数的暂存性死区（Default Parameter Value Temporal Dead Zone）


第一章介绍了关于 let 和 const 的暂存性死区 （temporal dead zone, TDZ），其同样存在于默认参数中使得变量无法访问。 和 let 声明类似，每个参数都创建了一个新的绑定，但是在它们被初始化之前访问会抛出错误。初始化的方式是通过在函数被调用的时刻传递参数或者使用默认参数。

为了进一步说明默认参数中的 TDZ，考虑 “默认参数表达式” 中的例子：

```js
function getValue(value) {
    return value + 5;
}

function add(first, second = getValue(first)) {
    return first + second;
}

console.log(add(1, 1));     // 2
console.log(add(1));        // 7
```

调用 add(1, 1) 和 add(1) 事实上执行了以下语句以创建 first 和 second 参数和赋值：

```js
// 调用 add(1, 1) 的 JavaScript 描述
let first = 1;
let second = 1;

// 调用 add(1) 的 JavaScript 描述
let first = 1;
let second = getValue(first);
```

当函数 add() 执行时，first 和 second 的绑定被移入了特定的参数 TDZ（类似于 let）。之所以 second 可以被 first 初始化是因为 first 的初始化在前，反之则不能。现在考虑下面经过重写的 add() 函数：

```js
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // 抛出错误
```

在这个例子中调用 add(1, 1) 和 add(undefined, 1) 幕后做了如下工作：

```js
//调用 add(1, 1) 的 JavaScript 描述
let first = 1;
let second = 1;

// 调用 add(undefined, 1) 的 JavaScript 描述
let first = second;
let second = 1;
```

调用 add(undefind, 1）会抛出错误是因为 second 在 first 需要初始化的时候还未初始化。该时刻 second 仍在 TDZ 中所以访问它会出错。这些和第一章讨论的 let 绑定十分相似。

<br />

> 函数参数相比函数内部有着自己的作用域和 TDZ，意味着参数的默认值不能使用函数内部声明的任何变量。

<br />

### <a id="Working-with-Unnamed-Parameters"> 未命名参数（Working with Unnamed Parameters） </a>

目前为止本章只讨论了函数定义中的命名参数，然而 JavaScript 函数并不限制可传入实参的数量，你可以传递比命名参数个数或多或少数量的参数。默认的参数值针对的是传入参数少于命名参数的情况，同样 ECMAScript 6 也为另一种情况铺了条更好的路。

<br />

#### ECMAScript 5 中的未命名参数（Unnamed Parameters in ECMAScript 5）

早先，JavaScript 提供了 arguments 对象使得查看函数参数时，分别定义每个参数的名称显得不是很必要。虽然当大部分使用 arguments 对象的情况下都能正常工作，但有时使用它会显得十分繁琐。比如下例中使用 arguments 对象的方式：

```js
function pick(object) {
    let result = Object.create(null);

    // start at the second parameter
    for (let i = 1, len = arguments.length; i < len; i++) {
        result[arguments[i]] = object[arguments[i]];
    }

    return result;
}

let book = {
    title: "Understanding ECMAScript 6",
    author: "Nicholas C. Zakas",
    year: 2015
};

let bookData = pick(book, "author", "year");

console.log(bookData.author);   // "Nicholas C. Zakas"
console.log(bookData.year);     // 2015
```

这个函数模仿了 Underscore.js 库中的 pick() 方法，返回了原始对象属性的子集。本例中只定义了一个参数，希望接收一个对象以便拷贝它的属性。其它传入的参数为添加到 result 对象中的属性名。

pick() 函数有几个需要注意的地方。首先，该函数看起来不具备处理更多参数的能力，或许你会想定义额外的参数，但是你不能预测调用这个函数的时候到底会传入多少参数。其次，当你要复制属性的时候，arguments 对象的索引值要从 1 而不是 0 开始。记住索引的起始值不难，但毕竟多了一个不安要素。

ECMAScript 6 引入了剩余参数来解决这个问题。

<br />

#### 剩余参数（Rest Parameters）

剩余参数由三点（...）和一个命名参数（放在三点之后）指定。这个命名参数是一个包含其它传入参数的数组，“剩余” 这个名称也是由此而来。例如，pick() 可以使用剩余参数来重写：

```js
function pick(object, ...keys) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

在这个版本的函数中，keys 是一个囊括所有 object 之后传入参数的剩余参数（和 arguments 不同，它不包含第一个参数）。这意味着你可以放心大胆地迭代 keys 中所有的值。同时又一点好处是你可以根据函数的签名来判断它有处理任意参数的能力。

<br />

> 函数的 length 属性用来描述参数的个数，剩余参数对其并无影响。上例中 pick() 的 length 属性值仍为 1，因为它只包括 object 参数。

<br />

##### 剩余参数的限制（Rest Parameter Restrictions）

剩余参数有两点限制。其一是函数只能有一个剩余参数，且必须放在最后的位置。下面例子中的代码是不正确的：

```js
// 语法错误：剩余参数后不应有命名参数
function pick(object, ...keys, last) {
    let result = Object.create(null);

    for (let i = 0, len = keys.length; i < len; i++) {
        result[keys[i]] = object[keys[i]];
    }

    return result;
}
```

在这里，last 参数跟在剩余参数后面，这会导致一个语法错误。

第二个限制是剩余函数不能被用在对象字面量中的 setter 上，也就是说下面的代码会导致语法错误：

```js
let object = {

    // 语法错误：不能在 setter 上使用剩余参数
    set name(...value) {
        // do something
    }
};
```

这项限制的存在原因是对象字面量中的 setter 只被允许接受单个参数，而规范中的剩余参数可以接受无限个数的参数，所以它是不被允许的。

<br />

##### 剩余参数对 arguments 对象的影响（How Rest Parameters Affect the arguments Object）

设计剩余参数的目的是用来替代 ECMAScript 中的 arguments。原本在 ECMAScript 4 中就决定移除 arguments 并添加了剩余参数来允许传入无限个数的参数。虽然 ECMAScript 4 从未走上台面，但是这个主意被保留并在 ECMAScript 6 中重新引入，尽管 arguments 对象仍有一席之地。

arguments 对象通过反映传入的参数来和剩余参数共同协作，如下面所示：

```js
function checkArgs(...args) {
    console.log(args.length);
    console.log(arguments.length);
    console.log(args[0], arguments[0]);
    console.log(args[1], arguments[1]);
}

checkArgs("a", "b");
The call to checkArgs() outputs:

2
2
a a
b b
```

arguments 对象总是能正确的反映所有传入的参数而无视剩余参数的使用。

以上的内容对于你初用剩余参数来说已经足够了。

<br />

### <a id="Increased-Capabilities-of-the-Function-Constructor"> 增强的 Function 构造函数（Increased Capabilities of the Function Constructor） </a>

Function 构造函数用来动态创建一个新的函数，但是在 JavaScript 编程中甚少使用。传给该构造函数的参数全部为字符串，并被视为新创建函数的参数和函数主体，如下所示：

```js
var add = new Function("first", "second", "return first + second");

console.log(add(1, 1));     // 2
```

ECMAScript 6 通过允许设置默认参数和剩余参数来增强了 Function 构造函数的功能，现在你只需要添加等于号和默认值就可以设置默认参数，如下：

```js
var add = new Function("first", "second = first",
        "return first + second");

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

在这个例子中，当未传入参数给 second 时，first 的值会赋给它。此处的语法和其它未使用 Function 构造的函数一致。

至于剩余函数则只需要添加给参数前面添加 ... 即可，就像这样：

```js
var pickFirst = new Function("...args", "return args[0]");

console.log(pickFirst(1, 2));   // 1
```

这段代码使用了单个剩余参数并返回传入的第一个参数。

默认参数和剩余参数的添加保证 Function 拥有使用声明构造的函数的所有能力。

<br />

### <a id="The-Spread-Operator"> 扩展运算符（The Spread Operator） </a>

和剩余参数概念相近的是扩展运算符。相比剩余参数允许你把多个独立的参数整合到一个数组中，扩展运算符则允许你把一个数组中的元素分别作为参数传递给函数。考虑下 Math.max() 这个方法，它接受任意数量的参数并返回它们之中的最大值。下面这个例子使用了该方法：

```js
let value1 = 25,
    value2 = 50;

console.log(Math.max(value1, value2));      // 50
```

如果像例子这样只使用两个值，那么 Math.max() 是很容易使用的：传入两个值，并返回较大的那个。但如果你想提取一个数组中所有元素的最大值呢？ 在 ECMAScript 5 及 之前的版本中 Math.max() 是不允许你传入整个数组的，所以你只能自己筛选或者像下面这样使用 apply()：

```js
let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values));  // 100
```

以上的办法是可行的，但是这里使用 apply() 会令人困惑。事实上它使用了额外的语法模糊了代码的本意。

ECMAScript 6 的扩展运算符使得该需求很容易实现。你可以将数组添加 ... 前缀直接传给 Math.max() 而非调用apply()。JavaScript 引擎将数组分解并将其中的元素传给函数，像这样：

```js
let values = [25, 50, 75, 100]

// 等同于
// console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values));           // 100
```

现在调用 Math.max() 的方式看起来更熟悉而且避免了额外的 this 绑定（Math.max.apply()的首个参数）造成复杂度的增加，使其单纯作为一个数学运算操作。

你可以混用扩展运算符和其它参数。假设你想让 Math.max() 返回的最小值为 0（防止数组中包含负值）。你可以把 0 传入该函数并在其它位置使用扩展运算符，如下：

```js
let values = [-25, -50, -75, -100]

console.log(Math.max(...values, 0));        // 0
```

在本例中，传给 Math.max() 最后的参数为 0，其之前的参数由扩展运算符传入。

扩展运算符使得数组作为函数的参数变得更加容易，在大部分情况下你会发现扩展运算符可以很好的替代 apply() 方法。

到目前为止，除了你所见到 ECMAScript 6 中关于默认参数和剩余参数的用法之外，你还可以在 JavaScript 的 Function 构造函数中使用它们。

<br />

### <a id="ECMAScript-6-name-Property"> ECMAScript 6 中的 name 属性（ECMAScript 6’s name Property） </a>

JavaScript 中多种定义函数的方式使得函数的辨识成为了一种挑战。此外，匿名函数表达式的流行使得调试更加困难，堆栈在跟踪时难以阅读和解析。由于这个原因，ECMAScript 6 为所有的函数添加了 name 属性。

<br />

#### 选择正确的名称（Choosing Appropriate Names）

ECMAScript 6 中所有函数都有正确的 name 属性值。为了验证请看如下的代码，它使用了函数（声明）和函数表达式，并输出了各自的 name 属性值：

```js
function doSomething() {
    // ...
}

var doAnotherThing = function() {
    // ...
};

console.log(doSomething.name);          // "doSomething"
console.log(doAnotherThing.name);       // "doAnotherThing"
```

在这段代码中，doSomething() 的 name 属性值为 "doSomething"，因为它是由函数声明所定义的。匿名函数表达定义的 doAnotherThing() 的 name 属性值为 "doAnotherThing" ，因为这是它赋给变量的名称。

<br />

#### 特殊情况下的 name 属性（Special Cases of the name Property）

虽然函数声明和函数表达式定义的函数名寻找起来比较容易，但是 ECMAScript 6 更进一步确保了函数名称的正确性。为了说明请看如下的演示：

```js
var doSomething = function doSomethingElse() {
    // ...
};

var person = {
    get firstName() {
        return "Nicholas"
    },
    sayName: function() {
        console.log(this.name);
    }
}

console.log(doSomething.name);      // "doSomethingElse"
console.log(person.sayName.name);   // "sayName"
console.log(person.firstName.name); // "get firstName"
```

在这个例子中，doSomething.name 是 "doSomethingElse"，因为该函数表达式拥有自己的名称，优先级比赋给变量的名称更高。person.sayName() 的 name 属性值为 "sayName"，正如对象字面量定义的那样。类似的是 person.firstName 实际上是个 getter 方法，所以它的 name 是 "get firstName" 以便和其它情况区分。同样，setter 方法在 name 属性值里带有 set 前缀。

函数的名称还有其它特殊情况。使用过 bind() 的函数 name 属性值会添加 bound 前缀，而使用 Function 构造函数创建的函数 name 属性的值为 "anonymous"。

```js
var doSomething = function() {
    // ...
};

console.log(doSomething.bind().name);   // "bound doSomething"

console.log((new Function()).name);     // "anonymous"
```

被绑定过的函数名总是带有 "bound" 字符串前缀，所以绑定过的 doSomething() 函数名为 "bound doSomething"。

需要注意的是函数的 name 属性值并不等同于同名变量。name 属性的作用是为了在调试时获得有用的相关信息，所以 name 属性值是获取不到相关函数的引用的。

<br />

### <a id="Clarifying-the-Dual-Purpose-of-Functions"> 明确函数的双重用途（Clarifying the Dual Purpose of Functions） </a>

在 ECMAScript 5 和早期的版本中，函数的双重用途表现在是否使用 new 来调用它。当使用 new 时，函数中的 this 为一个新的对象并返回它，如下面的演示：

```js
function Person(name) {
    this.name = name;
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");

console.log(person);        // "[Object object]"
console.log(notAPerson);    // "undefined"
```

当创建 notAPerson 时，即未使用 new 来调用 Person() 会输出 undefined（同时在非严格模式下给全局对象添加了 name 属性）。Person 首字母的大写是唯一指示其应该被 new 调用的标识，这在 JavaScript 编程中十分普遍。函数双重角色的扮演在 ECMAScript 6 中发生了一些改变。

JavaScript 中的函数有两个不同的只有内部（internal-only）能使用的方法：[[call]] 与 [[Construct]]。当函数未被 new 调用时，[[call]] 方法会被执行，运行的是函数主体中的代码。当函数被 new 调用时，[[Construct]] 会被执行并创建了一个新的对象，称为 new target，之后会执行函数主体并把 this 绑定为该对象。带有 [[Construct]] 方法的函数被称为构造函数（constructor）。

<br />

> 不是每个函数内部都有 [[Construct]] 方法，所以并非所有的函数都能被 new 调用。在 “箭头函数” 小结中提到的箭头函数就没有该方法。

<br />

#### ECMAScript 5 中函数调用方式的判断（Determining How a Function was Called in ECMAScript 5）

在 ECMAScript 5 中判断函数是否被 new 调用过的方式是使用 instanceof，如下：

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // 使用 new
    } else {
        throw new Error("你必须使用 new 来调用 Person。")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");  // 抛出错误
```

在这里，this 的值会被用来判断是否为构造函数的实例，答案为是的话则正常执行，否则会抛出错误。因为 [[Construct]] 方法会创建 Person 的新实例并将它绑定到 this 上。遗憾的是，这个方案并不可靠，因为不使用 new 调用的函数，其 this 值也可能是 Person，如下所示：

```js
function Person(name) {
    if (this instanceof Person) {
        this.name = name;   // 使用 new
    } else {
        throw new Error("你必须使用 new 来调用 Person。")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // 正常运行！
```

调用 Person.call() 并将 person 变量作为第一个参数会将 Person 内部的 this 设置为 person。对于函数本身来讲，如何辨别它们显得无能为力。

<br />

#### 元属性 new.target(The new.target MetaProperty)


为了解决这个问题，ECMAScript 6 引入了 new.target 这个元属性。元属性指的是和目标（如 new）相关但并非被包含在一个对象内的属性。当函数内部的 [[Construct]] 方法被调用后，new 操作符调用的目标（target）将赋给 new.target。该目标通常为创建对象实例并将该实例赋值给 this 的构造函数。如果 [[call]] 被执行，那么 new.target 的值为 undefined 。

新引入的该元属性允许你通过检查 new.target 是否被定义来准确的判断出函数是否被 new 调用，如下所示：

```js
function Person(name) {
    if (typeof new.target !== "undefined") {
        this.name = name;   // 使用 new
    } else {
        throw new Error("你必须使用 new 来调用 Person。")
    }
}

var person = new Person("Nicholas");
var notAPerson = Person.call(person, "Michael");    // 错误！
```

使用 new.target 而非 instanceof Person 会正确的在未使用 new 调用构造函数的时候抛出错误。

你也可以通过检查 new.target 来判断特定的构造函数是否被调用。例子如下：

```js
function Person(name) {
    if (typeof new.target === Person) {
        this.name = name;   // 使用 new
    } else {
        throw new Error("你必须使用 new 来调用 Person。")
    }
}

function AnotherPerson(name) {
    Person.call(this, name);
}

var person = new Person("Nicholas");
var anotherPerson = new AnotherPerson("Nicholas");  // 错误！
```

在这段代码中，new.target 必须是 Person 才可以正常运行。当调用 new AnotherPerson("Nicholas") 时 Person.call(this, name)也随即运行，但由于 Person 构造函数的 new.target 为 undefined 所以会抛出错误。

<br />

> **警告**： 在函数之外使用 new.target 会抛出语法错误

<br />

ECMAScript 6 通过添加 new.target 消除了函数存在调用歧义的可能性。紧随该主题的是，ECMAScript 6 还解决另一个早先存在的含糊问题：在块内声明的函数。

<br />

### <a id="Block-Level-Functions"> 块级函数（Block-Level Functions） </a>

在 ECMAScript 3 或更早的版本中，在块中声明函数（块级函数）理论上会发生语法错误，但所有的浏览器却都支持这么做。遗憾的是，每个浏览器支持的方式都有些差异，所以最佳实践就是不要在块中声明函数（更好的选择是使用函数表达式）。

为了抑制这种分裂行为，ECMAScript 5 中的严格模式规定在块中声明函数会发生错误：

```js
"use strict";

if (true) {

    // 在 ES5 中抛出错误，ES6不会
    function doSomething() {
        // ...
    }
}
```

在 ECMAScript 5 中，这段代码会抛出语法错误。然而 ECMAScript 6 会将 doSomething() 函数视为块级声明并可以在块内的其它部分调用，例如：

```js
"use strict";

if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "undefined"
```

块级函数会被提升到块内的顶部，即使 typeof doSomething 语句在函数声明之前也会返回 "function"。一旦代码块执行完毕，doSomething() 也将不复存在。

<br />

#### 使用块级函数的时机（Deciding When to Use Block-Level Functions）

块级函数与 let 函数表达式的相似之处是它们都会在执行流跳出定义它们所在的块作用域之后被销毁。关键的区别是块级函数（声明）会被提升到顶部，而 let 函数表达式则不会，以下代码对其做了说明：

```js
"use strict";

if (true) {

    console.log(typeof doSomething);        // 抛出错误

    let doSomething = function () {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);
```

在这里，代码执行到 typeof doSomething 回戛然而止，因为 let 语句还未被执行，doSomething() 仍处在 TDZ 内部。了解改差异之后，你可以根据是否想让函数提升到顶部来选择块级函数（声明）或 let 函数表达式。

<br />

#### 非严格模式下的块级函数（Block-Level Functions in Nonstrict Mode）


ECMAScript 6 同样允许非严格模式下块级函数的存在，但是具体行为有些不同。函数的声明会被提升至函数作用域或全局作用域的顶部，而不是块内。例如：

```js
// ECMAScript 6 的行为
if (true) {

    console.log(typeof doSomething);        // "function"

    function doSomething() {
        // ...
    }

    doSomething();
}

console.log(typeof doSomething);            // "function"
```

在本例中，doSomething() 会被提升至全局作用域的顶部，所以在 if 代码块外它仍然存在。ECMAScript 6 标准化了该行为以消除不同浏览器之间存在的差异，于是在所有 ECMASCript 6 运行环境中该表现都是相同的。

允许在 JavaScript 中声明块级变量使得声明函数的能力得到了加强。然而 ECMAScript 6 还引入了另一种全新的声明函数的方法。

<br />

### <a id="Arrow-Functions"> 箭头函数（Arrow Functions） </a>


ECMAScript 6 最有意思的部分之一就是箭头函数。正如其名，箭头函数由 “箭头”（=>）这种新的语法来定义。但是箭头函数的表现在以下几个重要的方面不同于传统的 JavaScript 函数：

* 没有 this，super，arguments 和 new.target 绑定 - this，super，arguments 和 new.target 的值由最近的不包含箭头函数的作用域决定。（super 会在第四章讲解）

* 不能被 new 调用 - 箭头函数内部没有 [[Construct]] 方法，因此不能当作构造函数使用。使用 new 调用箭头函数会抛出错误。

* 没有 prototype - 既然你不能使用 new 调用箭头函数，那么 prototype 就没有存在的理由。箭头函数没有 prototype 属性。

* 不能更改 this - this 的值在函数内部不能被修改。在函数的整个生命周期内 this 的值是永恒不变的。

* 没有 arguments 对象 - 既然箭头函数没有 arguments 绑定，你必须依赖于命名或者剩余参数来访问该函数的参数。

* 不允许重复的命名参数 - 不论是在严格模式还是非严格模式下，箭头函数都不允许重复的命名参数存在，相比传统的函数，它们只有在严格模式下才禁止该种行为。

这些差异的存在是有理由的。首先也是最重要的是，在 JavaScript 编程 中 this 绑定是 中发生错误的根源之一。this 的值很容易丢失，使得程序以臆想之外的方式运行，而箭头函数解决了该问题。其次，箭头函数限制 this 为固定值的做法让 JavaScript 引擎可以对一些操作进行优化，相比普通的函数它们可能被视为构造函数或被其它因素修改。

其它方面差异的存在也是专注于减少潜在错误发生的可能性与歧义的消除，同时 JavaScript 引擎也能更好的优化箭头函数。

<br />

> **注意**： 箭头函数的 name 属性是存在的，并且和其它类别的函数遵循相同的规则

<br />

#### 箭头函数语法（Arrow Function Syntax）


根据你的需求，箭头函数的语法可以有多种形体。所有变体都是以参数为开头，箭头紧随其后，函数主体为结尾。参数和主体根据用途可以有不同的形式，例如西面的箭头函数接收单个参数并返回它：

```js
var reflect = value => value;

// 等同于：

var reflect = function(value) {
    return value;
};
```

当箭头函数只有一个参数时，该参数可以直接使用而不需要额外的语法。之后则是箭头和需要计算的表达式，即使没有显式书写 return 语句也会返回该参数。

如果你需要传入单个以上的参数，就需要用括号来包含它们，如下：

```js
var sum = (num1, num2) => num1 + num2;

// effectively equivalent to:

var sum = function(num1, num2) {
    return num1 + num2;
};
```

sum() 函数简单地接受两个参数并返回运算结果。它和 reflect() 函数唯一的区别在于参数是被括号包裹并由逗号分隔的（正如一般的函数那样）。

如果函数没有参数，那么在声明中就必须使用一对空括号，如下所示：

```js
var getName = () => "Nicholas";

// 等同于：

var getName = function() {
    return "Nicholas";
};
```

当你想使用传统的函数主体书写方式，尤其是主体内包含一条以上表达式的时候，你需要用花括号显式地包裹函数主体并使用 return 语句，如下面的 sum() 函数：

```js
var sum = (num1, num2) => {
    return num1 + num2;
};

// 等同于：

var sum = function(num1, num2) {
    return num1 + num2;
};
```

你可以大体上将传统函数花括号中的内容移植过去，需要注意的是 arguments 对象并不可用。

如果你想创建空函数，那么就必须使用花括号，像这样：

```js
var doNothing = () => {};

// 等同于：

var doNothing = function() {};
```js
花括号代表函数的主体，以上的示例中花括号的使用到目前为止一切正常。然而当不想使用传统的函数主体形式返回一个对象字面量的时候，必须将该对象放在括号中。例如：

```js
var getTempItem = id => ({ id: id, name: "Temp" });

// 等同于：

var getTempItem = function(id) {

    return {
        id: id,
        name: "Temp"
    };
};
```

将对象字面量放在括号内代表其并非为函数主体。

<br />

#### 创建即用函数表达式（Creating Immediately-Invoked Function Expressions）


在 JavaScript 中一种流行的函数使用方式是创建即用函数表达式（immediately-invoked function expressions, IIFEs）。IIFEs 允许你创建匿名函数并立即调用它，没有创建任何引用。当你想开拓一个不受项目中其它代码干扰的独立作用域时，这种方法特别好用，例如：

```js
let person = function(name) {

    return {
        getName: function() {
            return name;
        }
    };

}("Nicholas");

console.log(person.getName());      // "Nicholas"
```

在这段代码中，IIFE 创建了一个包含 getName() 方法的新对象。该方法返回传入的参数值，并显著地使 name 成为该对象的私有成员。

你可以用括号包裹箭头函数来达到同样的效果：

```js
let person = ((name) => {

    return {
        getName: function() {
            return name;
        }
    };

})("Nicholas");

console.log(person.getName());      // "Nicholas"
```

需要注意的是括号包裹的是箭头函数的定义，并不包括（"Nicholas"）。这和传统的函数不同，函数定义和传入的参数可以同时被括号包含。

<br />

#### 无 this 绑定（No this Binding）

JavaScript 编程中最常见的错误之一就是函数中 this 的绑定。由于函数内部的 this 可以在调用时被上下文替换，所以操作了意想不到的对象的几率很大。考虑如下的例子：

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", function(event) {
            this.doSomething(event.type);     // error
        }, false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

在这段代码中，PageHandler 对象用来处理页面上的交互。init() 方法注册事件并在回调函数里调用 this.doSomething()。但是该段代码没有按照我们想象的方式工作。

this.doSomething() 出现故障是因为 this 指代的是调用该函数的对象（在本例中是 document）而不是PageHandler。如果你试图运行这段代码那么错误将会被抛出，因为 this.doSomething() 在 document 对象上并不存在。

你可以显式地在函数上调用 bind() 来绑定 this 以便解决这个问题，像这样：

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click", (function(event) {
            this.doSomething(event.type);     // 无错误发生
        }).bind(this), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

现在代码如我们想象的那样运行，不过看起来有些奇怪。通过调用 bind(this)，你实际上创建了一个带有 this 绑定的新函数。为了避免额外函数的创建，更好的方式是使用箭头函数。

箭头函数没有 this 绑定，意味着 this 只能通过查找作用域链来确定。如果箭头函数被另一个不包含箭头函数的函数囊括，那么 this 的值和该函数中的 this 相等，否则 this 的值为 undefined。以下就是使用箭头函数重构的示例：

```js
var PageHandler = {

    id: "123456",

    init: function() {
        document.addEventListener("click",
                event => this.doSomething(event.type), false);
    },

    doSomething: function(type) {
        console.log("Handling " + type  + " for " + this.id);
    }
};
```

本例中 this.doSomething() 是在处理事件的箭头回调函数中调用。this 的值和 init() 方法相同，所以以上代码的作用和 bind(this) 类似。虽然 doSomething() 方法并不返回任何值，它仍然是函数主体中唯一需要运行的语句，所以花括号的使用没有必要。

箭头函数被定义为 “用完即仍” 的函数，所以不能被用来定义新类型；证据是箭头函数不存在一般函数中包含的 property 属性。使用 new 来调用箭头函数会发生错误，如下所示：

```js
var MyType = () => {},
    object = new MyType();  // 错误 - 你不能使用 new 调用箭头函数
```

在这段代码中，调用 new MyType() 失败的原因是箭头函数内部不存在 [[Construct]] 方法。箭头函数不能被 new 调用的特性使得 JavaScript 引擎能对它们做更深一步的优化。

同样，箭头函数中 this 的值由包含它的函数决定，所以你没有办法通过 call()，apply() 或 bind() 方法来改变 this 的值。

<br />

#### 箭头函数与数组（Arrow Functions and Arrays）


箭头函数写法的简洁特性也非常适合操作数组。例如你想自定义数组元素的排序方式，一般会使用下面的写法：

```js
var result = values.sort(function(a, b) {
    return a - b;
});
```

这些代码对如此简单的需求来讲显得冗余，相比如下使用箭头函数重构的版本更是如此：

```js
var result = values.sort((a, b) => a - b);
```

使用回调函数的数组方法，例如 sort()，map() 和 reduce() 都能享受箭头函数语法带来的好处 —— 把复杂的代码简单化。

<br />

#### 无 arguments 绑定（No arguments Binding）


虽然箭头函数没有自己的 arguments 对象，不过访问包含它们的函数中的 arguments 对象还是可行的。该对象不论箭头函数何时执行都能访问。例如：

```js
function createArrowFunctionReturningFirstArg() {
    return () => arguments[0];
}

var arrowFunction = createArrowFunctionReturningFirstArg(5);

console.log(arrowFunction());       // 5
```

在 createArrowFunctionReturningFirstArg() 内部，arguments[0] 元素被创建的箭头函数所引用。该元素为 传入 createArrowFunctionReturningFirstArg() 函数的首个参数。当箭头函数在随后执行时，返回的值为 5，这也正是传给示例中传给 createArrowFunctionReturningFirstArg() 的参数。虽然 createArrowFunctionReturningFirstArg() 被调用后箭头函数将不存在于该函数作用域内，但是作用域链中的 arguments 标识符依旧可以访问。

<br />

#### 查找箭头函数（Identifying Arrow Functions）


尽管语法不同，但箭头函数依旧属于函数，定义也是如此。考虑以下的代码：

```js
var comparator = (a, b) => a - b;

console.log(typeof comparator);                 // "function"
console.log(comparator instanceof Function);    // true
```

console.log() 输出的内容显示 typeof 和 instanceof 操作箭头函数的表现和其它函数完全一致。

同样，你也可以对箭头函数使用 call()，apply()，bind()，即使它们的 this 不受影响。如下所示：

```js
var sum = (num1, num2) => num1 + num2;

console.log(sum.call(null, 1, 2));      // 3
console.log(sum.apply(null, [1, 2]));   // 3

var boundSum = sum.bind(null, 1, 2);

console.log(boundSum());                // 3
```

sum() 函数被 call 和 apply() 调用并传递参数，类似于你使用其它函数的方式。bind() 方法创建了 boundSum() 函数，意味着 sum() 方法的参数已被绑定为 1 和 2，再次调用不需要传入参数。

你使用的任何匿名函数都能被箭头函数所替代，例如回调函数。下一节要讲述的是 ECMAScript 6 另一项主要的开发内容，不过它是由内部实现的，同时没有新的语法出现。

<br />

### <a id="Tail-Call-Optimization"> 尾调用优化（Tail Call Optimization） </a>

也许 ECMAScript 6 中关于函数的改进最有意思的是引擎针对尾部调用机制的优化。尾调用指的是一个函数在另一个函数的尾部被调用，像这样：

```js
function doSomething() {
    return doSomethingElse();   // 尾调用
}
```

ECMAScript 5 实现的尾调用和其它位置调用处理机制都是相同的：一个新的堆栈帧（stack frame）被创建并添加到堆栈上，以代表该函数被调用过。这意味着之前所有的堆栈帧在内存中持续存在，当调用栈过大时会产生一些问题。

<br />

#### 有何不同？（What’s Different?）


在严格模式下 ECMAScript 6 试图利用恰当的尾部函数调用来减少调用栈的大小（非严格模式下的尾调用未被考虑）。该优化使得尾部的函数调用不再增加，而是清除并利用已存在的堆栈帧（stack frame）。该优化需要如下条件：


1. 尾调用不能引用当前堆栈帧中的变量（即尾调用的函数不能是闭包）
2. 使用尾调用的函数在尾调用结束后不能做额外的操作
3. 尾调用函数值作为当前函数的返回值


下面的代码会被优化，因为以上三个条件全部符合：

```js
"use strict";

function doSomething() {
    // 优化
    return doSomethingElse();
}
```

该函数在尾部调用 doSomethingElse()之后立即返回该函数值，同时未引用当前作用域内的变量。不过一个小小的改动就会阻止优化的发生：

```js
"use strict";

function doSomething() {
    // 未优化 - 无返回值
    doSomethingElse();
}
```

类似的是，如果你的函数对尾调用函数值做了额外操作，那么该函数也不能被优化：

```js
"use strict";

function doSomething() {
    // 未优化 - 在函数执行并返回之前有额外的操作
    return 1 + doSomethingElse();
}
```

该例中函数在 doSomethingElse() 函数值返回之前对其做了加 1 操作，足以使优化关闭。

另一无意且常见的至使优化取消的使用方法是使用变量存储函数值，并返回这个变量：

```js
"use strict";

function doSomething() {
    // 未优化 - 函数调用未发生在尾部
    var result = doSomethingElse();
    return result;
}
```

本例中优化被取消的原因是 doSomethingElse() 的函数值没有被立即返回。

或许在尾调用优化需求中最难处理的是闭包。因为闭包需要访问包含该尾调用的函数中的变量，尾调用优化就会被取消。例如：

```js
"use strict";

function doSomething() {
    var num = 1,
        func = () => num;

    // 未优化 - 存在闭包
    return func();
}
```

该例中 func() 闭包需要访问本地变量 num。虽然 func() 调用后会马上返回该函数值，但是该引用的存在导致优化不会发生。

<br />

#### 如何使用尾调用优化（How to Harness Tail Call Optimization）

在实践中，尾调用优化发生在幕后，所以除非你有意的去优化某个函数否则不必想得太多。尾调用优化的主要使用场景是使用递归，而且该优化的效果及其显著。考虑如下计算阶乘（factorial）的函数：

```js
function factorial(n) {

    if (n <= 1) {
        return 1;
    } else {

        // 未优化 - 尾调用函数值有乘法运算
        return n * factorial(n - 1);
    }
}
```

该版本的函数不会被优化因为递归调用的 factorial() 的函数值总是要发生乘法运算，如果 n 是个非常大的数，那么调用栈会膨胀，而且有潜在的调用栈溢出的危险。

为了优化该函数，你必须保证函数调用之后不能有乘法运算。为了实现这一点，你可以使用默认参数来去除 return 语句中的乘法运算，之后把临时的结果传给下一次迭代。这些改进虽然功能和上例是相同的，但是会被 ECMAScript 6 的引擎优化。下面是重构后的函数：

```js
function factorial(n, p = 1) {

    if (n <= 1) {
        return 1 * p;
    } else {
        let result = n * p;

        // 优化
        return factorial(n - 1, result);
    }
}
```

在重写的 factorial() 函数中，添加了第二个参数 p 并带有默认值 1 。参数 p 负责保存上次乘法运算的结果，所以不需要调用其它的函数即可计算下一次的值。当 n 大于 1 的时候，先进行乘法运算并将值作为第二个参数传入 factorial()。这允许 ECMAScript 6 引擎去优化这些递归调用。

在书写递归函数的时候你应该考虑尾调用优化，因为它能提供显著的性能提升，尤其是那些带有昂贵计算（computationally-expensive）的函数。

<br />

### <a id="Summary"> 总结（Summary） </a>

ECMAScript 6 中的函数并未发生巨大的变化，相反，一系列小的改进使得函数更容易使用。

默认函数参数允许你方便地给那些未被传入实参的形参赋值。在 ECMAScript 6 之前，实现该需求需要在函数内部书写额外的代码来对参数的存在进行验证和赋值。

剩余参数允许你使用数组来包含所有的遗留参数。使用真正的数组并能自行挑选需要包含的参数使得剩余参数是比 arguments 更为灵活的解决方案。

扩展运算符是剩余参数的同伴，允许你将数组中的元素解构为参数并传给调用的函数。在 ECMAScript 6 之前，想要把数组的元素分别作为参数传给函数只有两种办法：手动将数组中的元素添加到参数的位置或者使用 apply()。在扩展运算符出现后，你可以很容易的把数组传递给函数，同时不需要担心 this 的绑定。

name 属性的添加使得在调试和评估中查找函数名变得极为方便。此外，ECMAScript 6 正式定义了块级函数，所以在严格模式下它们将不会抛出语法错误。

在 ECMAScript 6 中，函数的行为由普通调用时的内部函数 [[Call]]或被 new 调用时的内部函数[[Construct]]来决定。元属性 new.target 可以被用来判断函数是否被 new 调用。

ECMAScript 6 函数最大的变化就是箭头函数的引入。设计箭头函数的目的是为了替代匿名函数表达式。箭头函数有更简洁的语法，this 的词法绑定（lexical this binding）并移除了 arguments 对象。此外箭头函数无法更改 this 的绑定，所以它们不能被用作构造函数。

尾调用优化允许某些函数的调用被优化，以便减少调用栈的大小和内存占用，防止堆栈溢出。当符合相应条件时该优化会由引擎自动实现，然而你可以有目的地重写某些函数以便利用它。

<br />






