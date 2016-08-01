# 函数（Functions）


函数在任何编程语言中都是非常重要的一部分，而在 ECMAScript 6 之前，函数自 JavaScript 诞生以来并未过有较大的变化。这导致很多问题的积压，同时一些细微的差异容易诱发错误，或使用冗余的代码来实现非常基本的功能。

ECMAScript 6 中的函数相较而言是个大的跃进，着手调查了 JavaScript 开发者多年的抱怨和需求。最终的结果是在 ECMAScript 5 函数的基础之上做了几项改进，增强 JavaScript 的编程体验并减少诱发错误的因素。

<br />

### 带默认参数的函数（Functions with Default Parameter Values）


JavaScript 的函数比较特殊的是可以接受任意个数的参数，完全无视函数声明中的参数个数。这允许你通过自行给未传值的参数赋默认值来定义带有不同参数的函数。本章将介绍 ECMAScript 6 及之前的 ECMAScript 版本如何实现默认参数，同时引出的还有 arguments 对象，参数表达式（expressions as parameters）及 TDZ 的另一形式。

<br />

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

<br />

#### 默认参数的暂存性死区（Default Parameter Value Temporal Dead Zone）


第一章介绍了关于 let 和 const 的暂存性死区 （temporal dead zone, TDZ），其同样存在于默认参数中使得变量无法访问。 和 let 声明类似，每个参数都创建了一个新的绑定，但是在它们被初始化之前访问会抛出错误。初始化的方式是通过在函数被调用的时刻传递参数或者使用默认参数。

为了进一步说明默认参数中的 TDZ，考虑 “默认参数表达式” 中的例子：

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

调用 add(1, 1) 和 add(1) 事实上执行了以下语句以创建 first 和 second 参数和赋值：

```
// 调用 add(1, 1) 的 JavaScript 描述
let first = 1;
let second = 1;

// 调用 add(1) 的 JavaScript 描述
let first = 1;
let second = getValue(first);
```

当函数 add() 执行时，first 和 second 的绑定被移入了特定的参数 TDZ（类似于 let）。之所以 second 可以被 first 初始化是因为 first 的初始化在前，反之则不能。现在考虑下面经过重写的 add() 函数：

```
function add(first = second, second) {
    return first + second;
}

console.log(add(1, 1));         // 2
console.log(add(undefined, 1)); // 抛出错误
```

在这个例子中调用 add(1, 1) 和 add(undefined, 1) 幕后做了如下工作：

```
//调用 add(1, 1) 的 JavaScript 描述
let first = 1;
let second = 1;

// 调用 add(undefined, 1) 的 JavaScript 描述
let first = second;
let second = 1;
```

调用 add(undefind, 1）会抛出错误是因为 second 在 first 需要初始化的时候还未初始化。该时刻 second 仍在 TDZ 中所以访问它会出错。这些和第一章讨论的 let 绑定十分相似。

<br />

> **注意**: 函数参数相比函数内部有着自己的作用域和 TDZ，意味着参数的默认值不能使用函数内部声明的任何变量。

<br />

### 未命名参数（Working with Unnamed Parameters）

目前为止本章只讨论了函数定义中的命名参数，然而 JavaScript 函数并不限制可传入实参的数量，你可以传递比命名参数个数或多或少数量的参数。默认的参数值针对的是传入参数少于命名参数的情况，同样 ECMAScript 6 也为另一种情况铺了条更好的路。

#### ECMAScript 5 中的未命名参数（Unnamed Parameters in ECMAScript 5）

早先，JavaScript 提供了 arguments 对象使得查看函数参数时，分别定义每个参数的名称显得不是很必要。虽然当大部分使用 arguments 对象的情况下都能正常工作，但有时使用它会显得十分繁琐。比如下例中使用 arguments 对象的方式：

```
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

```
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

> **注意**： 函数的 length 属性用来描述参数的个数，剩余参数对其并无影响。上例中 pick() 的 length 属性值仍为 1，因为它只包括 object 参数。

<br />

##### 剩余参数的限制（Rest Parameter Restrictions）

剩余参数有两点限制。其一是函数只能有一个剩余参数，且必须放在最后的位置。下面例子中的代码是不正确的：

``` rest-parameter
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

```
let object = {

    // 语法错误：不能在 setter 上使用剩余参数
    set name(...value) {
        // do something
    }
};
```

这项限制的存在原因是对象字面量中的 setter 只被允许接受单个参数，而规范中的剩余参数可以接受无限个数的参数，所以它是不被允许的。

##### 剩余参数对 arguments 对象的影响（How Rest Parameters Affect the arguments Object）

设计剩余参数的目的是用来替代 ECMAScript 中的 arguments。原本在 ECMAScript 4 中就决定移除 arguments 并添加了剩余参数来允许传入无限个数的参数。虽然 ECMAScript 4 从未走上台面，但是这个主意被保留并在 ECMAScript 6 中重新引入，尽管 arguments 对象仍有一席之地。

arguments 对象通过反映传入的参数来和剩余参数共同协作，如下面所示：

```
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

### 增强的 Function 构造函数（Increased Capabilities of the Function Constructor）

Function 构造函数用来动态创建一个新的函数，但是在 JavaScript 编程中甚少使用。传给该构造函数的参数全部为字符串，并被视为新创建函数的参数和函数主体，如下所示：

```
var add = new Function("first", "second", "return first + second");

console.log(add(1, 1));     // 2
```

ECMAScript 6 通过允许设置默认参数和剩余参数来增强了 Function 构造函数的功能，现在你只需要添加等于号和默认值就可以设置默认参数，如下：

```
var add = new Function("first", "second = first",
        "return first + second");

console.log(add(1, 1));     // 2
console.log(add(1));        // 2
```

在这个例子中，当未传入参数给 second 时，first 的值会赋给它。此处的语法和其它未使用 Function 构造的函数一致。

至于剩余函数则只需要添加给参数前面添加 ... 即可，就像这样：

```
var pickFirst = new Function("...args", "return args[0]");

console.log(pickFirst(1, 2));   // 1
```

这段代码使用了单个剩余参数并返回传入的第一个参数。

默认参数和剩余参数的添加保证 Function 拥有使用声明构造的函数的所有能力。

<br />

### 扩展运算符（The Spread Operator）

和剩余参数概念相近的是扩展运算符。相比剩余参数允许你把多个独立的参数整合到一个数组中，扩展运算符则允许你把一个数组中的元素分别作为参数传递给函数。考虑下 Math.max() 这个方法，它接受任意数量的参数并返回它们之中的最大值。下面这个例子使用了该方法：

```
let value1 = 25,
    value2 = 50;

console.log(Math.max(value1, value2));      // 50
```

如果像例子这样只使用两个值，那么 Math.max() 是很容易使用的：传入两个值，并返回较大的那个。但如果你想提取一个数组中所有元素的最大值呢？ 在 ECMAScript 5 及 之前的版本中 Math.max() 是不允许你传入整个数组的，所以你只能自己筛选或者像下面这样使用 apply()：

```
let values = [25, 50, 75, 100]

console.log(Math.max.apply(Math, values));  // 100
```

以上的办法是可行的，但是这里使用 apply() 会令人困惑。事实上它使用了额外的语法模糊了代码的本意。

ECMAScript 6 的扩展运算符使得该需求很容易实现。你可以将数组添加 ... 前缀直接传给 Math.max() 而非调用apply()。JavaScript 引擎将数组分解并将其中的元素传给函数，像这样：

``` spread-operator
let values = [25, 50, 75, 100]

// 等同于
// console.log(Math.max(25, 50, 75, 100));
console.log(Math.max(...values));           // 100
```

现在调用 Math.max() 的方式看起来更熟悉而且避免了额外的 this 绑定（Math.max.apply()的首个参数）造成复杂度的增加，使其单纯作为一个数学运算操作。

你可以混用扩展运算符和其它参数。假设你想让 Math.max() 返回的最小值为 0（防止数组中包含负值）。你可以把 0 传入该函数并在其它位置使用扩展运算符，如下：

```
let values = [-25, -50, -75, -100]

console.log(Math.max(...values, 0));        // 0
```

在本例中，传给 Math.max() 最后的参数为 0，其之前的参数由扩展运算符传入。

扩展运算符使得数组作为函数的参数变得更加容易，在大部分情况下你会发现扩展运算符可以很好的替代 apply() 方法。

到目前为止，除了你所见到 ECMAScript 6 中关于默认参数和剩余参数的用法之外，你还可以在 JavaScript 的 Function 构造函数中使用它们。

<br />

### ECMAScript 6 中的 name 属性（ECMAScript 6’s name Property）

JavaScript 中多种定义函数的方式使得函数的辨识成为了一种挑战。此外，匿名函数表达式的流行使得调试更加困难，堆栈在跟踪时难以阅读和解析。由于这个原因，ECMAScript 6 为所有的函数添加了 name 属性。

<br />

#### 选择正确的名称（Choosing Appropriate Names）

ECMAScript 6 中所有函数都有正确的 name 属性值。为了验证请看如下的代码，它使用了函数（声明）和函数表达式，并输出了各自的 name 属性值：

```
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

```
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

```
var doSomething = function() {
    // ...
};

console.log(doSomething.bind().name);   // "bound doSomething"

console.log((new Function()).name);     // "anonymous"
```

被绑定过的函数名总是带有 "bound" 字符串前缀，所以绑定过的 doSomething() 函数名为 "bound doSomething"。

需要注意的是函数的 name 属性值并不等同于同名变量。name 属性的作用是为了在调试时获得有用的相关信息，所以 name 属性值是获取不到相关函数的引用的。

<br />

### 明确函数的双重用途（Clarifying the Dual Purpose of Functions）

在 ECMAScript 5 和早期的版本中，函数的双重用途表现在是否使用 new 来调用它。当使用 new 时，函数中的 this 为一个新的对象并返回它，如下面的演示：

```
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

> **注意**： 不是每个函数内部都有 [[Construct]] 方法，所以并非所有的函数都能被 new 调用。在后面的小结中提到的箭头函数就没有该方法。

<br />

#### ECMAScript 5 中函数调用方式的判断（Determining How a Function was Called in ECMAScript 5）

在 ECMAScript 5 中判断函数是否被 new 调用过的方式是使用 instanceof，如下：

```
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

```
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

```
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

```
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

ECMAScript 6 通过添加 new.target 解决了函数存在调用歧义的问题。紧随该主题的是，ECMAScript 6 还解决另一个早先存在的含糊问题：在块内声明函数。

<br />

### 块级函数（Block-Level Functions）



