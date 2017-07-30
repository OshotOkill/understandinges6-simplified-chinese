# Symbols and Symbol Properties


Symbol 是 ECMAScript 6 新引入的基本类型。其它基本类型包括：字符串类型（string），数字类型（number），布尔类型（boolean），null 和 undefined 。对象可以使用 symbol 来创建私有成员，这也是 JavaScript 开发者长久以来期待的一项特性。在 symbol 引入之前，若不论名称本身，任何字符串属性都可以很容易地被访问，而 “私有命名（private name）” 特性意味着开发者创建非字符串属性。因此，使用一般地方法无法访问它们。

私有命名的提案最终入驻 ECMAScript 6，并称其为 symbol，同时本章也会教导如何有效地使用它们。不过，symbol 仅保留了实现细节（即，引入了非字符串属性名）而放弃了隐秘性。相反，symbol 属性和其它对象属性不属于同一个类别。

<br />

### 本章小结
* [创建 Symbol](#Creating-Symbols)
* [使用 Symbol](#Using-Symbols)
* [共享 Symbol](#Sharing-Symbols)
* [Symbol 类型的转换](#Symbol-Coercion)
* [提取 Symbol 属性](#Retrieving-Symbol-Properties)
* [揭秘内置 Well-Known Symbols 的运作](#Exposing-Internal-Operations-with-Well-Known-Symbols)
* [总结](#Summary)

<br />

### <a id="Creating-Symbols"> 创建 Symbol（Creating Symbols） </a>


symbol 在 JavaScript 基本类型中比较特别，你可以用 true 和 42 分别代表布尔类型和数字类型，但是 symbol 类型却无法用字面量表示。你可以使用全局 Symbol 函数来创建一个 symbol，如下所示：

```js
let firstName = Symbol();
let person = {};

person[firstName] = "Nicholas";
console.log(person[firstName]);     // "Nicholas"
```

在这里，firstName 作为 symbol 类型被创建并赋值给 person 对象以作其属性。每次访问这个属性时必须使用该 symbol 。symbol 变量的良好命名是个不错的注意，你可以很容易地得知 symbol 代表的内容。

<br />

> **注意**: 因为 symbol 是基础类型，调用 new Symbol() 时会抛出错误。你也可以通过 new Object(yourSymbol) 来创建 Symbol 的一个实例，不过目前尚不清楚这样做有何意义。

<br />

Symbol 函数也会接收一个额外参数来作为自身的描述。该描述本身无法访问属性，不过它可以在调试中发挥作用。例如：

```js
let firstName = Symbol("first name");
let person = {};

person[firstName] = "Nicholas";

console.log("first name" in person);        // false
console.log(person[firstName]);             // "Nicholas"
console.log(firstName);                     // "Symbol(first name)"
```

symbol 的描述被存储在内部属性 [[Description]] 中。无论是显式还是隐式调用 symbol 的 toString() 方法，该属性都会被读取。firstName symbol 在本例中由 console.log() 隐式调用，并将其描述输出到控制台上。除此之外没有别的方法可以由代码访问 [[Description]] 内部属性。我推荐向每个 symbol 添加描述以便读取或调试 symbol。

<br />

> #### 识别 symbol（Identifying Symbols）

> 既然 symbol 是基础类型，你可以使用 typeof 操作符来判断变量是否为 symbol 。ECMAScript 6 拓展了 typeof 使其操作 symbol 时返回 "symbol"。例如：

```js
let symbol = Symbol("test symbol");
console.log(typeof symbol);         // "symbol"
```

> 虽然还有其它间接方式判断 symbol 变量，不过 typeof 操作符是最精准同时也是我推荐的方式。

<br />

#### <a id="Using-Symbols"> 使用 Symbol（Using Symbols） </a>


你可以使用 symbol 来替换动态属性名（computed property name）。在本章中你已经见过 symbol 和方括号的组合使用方式，你还可以在调用 Object.defineProperty() 和 Object.defineProperties() 的时候使用它们，例如；

```js
let firstName = Symbol("first name");

// 使用动态计算的属性名
let person = {
    [firstName]: "Nicholas"
};

// 修改为只读属性
Object.defineProperty(person, firstName, { writable: false });

let lastName = Symbol("last name");

Object.defineProperties(person, {
    [lastName]: {
        value: "Zakas",
        writable: false
    }
});

console.log(person[firstName]);     // "Nicholas"
console.log(person[lastName]);      // "Zakas"
```

本例首先使用动态属性的形式创建 firstName symbol 属性。与非 symbol 类型的动态属性不同，它是不可枚举的。接下来的几行代码将该属性设置为只读。之后，Object.defineProperties() 方法再次使用动态属性创建了一个新的 symbol 只读属性，不同的是它只是第二个参数的一部分。

虽然 symbol 可以用在任意动态属性名的位置，你仍然需要一种机制来共享 symbol 以便在不同的代码片段中有效地使用它们。

<br />

### <a id="Sharing-Symbols"> 共享 Symbol（Sharing Symbols） </a>


你或许想在不同部分的代码中使用相同的 symbol 。例如，在你的应用中有两种不同类型的对象要使用相同的 symbol 属性来表示唯一标识符。然而跨文件或代码库追踪这些 symbol 十分困难且容易出错。因此 ECMAScript 6 引入了全局 symbol 记录（registry）供你随时访问它们。

当你想创建并共享一个 symbol 时，要使用 Symbol.for() 方法而不是调用 Symbol() 。Symbol.for() 方法接收单个参数，即你想要创建的 symbol 字符串标识符。同时它也作为该 symbol 的描述。例如：

```js
let uid = Symbol.for("uid");
let object = {};

object[uid] = "12345";

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"
```

Symbol.for() 方法首先搜索全局 symbol 记录并查看是否有包含 "uid" 这个键的 symbol。如果结果为是，那么该方法返回这个已有的 symbol，否则它将关联此键并填入到全局 symbol 记录当中，之后再返回它。这意味着后续使用相同的键调用 Symbol.for() 会返回相同的 symbol，如下所示：

```js
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};

console.log(object[uid]);       // "12345"
console.log(uid);               // "Symbol(uid)"

let uid2 = Symbol.for("uid");

console.log(uid === uid2);      // true
console.log(object[uid2]);      // "12345"
console.log(uid2);              // "Symbol(uid)"
```

在本例中，uid 和 uid2 是相同的 symbol，因此它们可以互换。首次调用 Symbol.for() 创建了该 symbol，第二次调用会从全局 symbol 记录中提取它。

共享的 symbol 还有一个特别之处，你可以调用 Symbol.keyFor() 方法来提取全局 symbol 记录中某个 symbol 的关联键。例如：

```js
let uid = Symbol.for("uid");
console.log(Symbol.keyFor(uid));    // "uid"

let uid2 = Symbol.for("uid");
console.log(Symbol.keyFor(uid2));   // "uid"

let uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3));   // undefined
```

需要留意的是 uid 和 uid2 都返回 "uid" 。uid3 在全局 symbol 记录中不存在，Symbol.keyFor() 查不到它的关联键，因此返回 undefined 。

<br />

> **注意**: 全局 symbol 记录是个共享的环境，类似于全局作用域。这意味着你无法得知在该环境中究竟存在着哪些内容。例如，jQuery 代码可能会使用 "jquery." 作为所有键的前缀，例如 "jquery.element" 等。

<br />

### <a id="Symbol-Coercion"> Symbol 类型的强制转换（Symbol Coercion） </a>


Type coercion is a significant part of JavaScript, and there’s a lot of flexibility in the language’s ability to coerce one data type into another. Symbols, however, are quite inflexible when it comes to coercion because other types lack a logical equivalent to a symbol. Specifically, symbols cannot be coerced into strings or numbers so that they cannot accidentally be used as properties that would otherwise be expected to behave as symbols.

类型强制转换在 JavaScript 中意义重大，在该语言中它有着极高的灵活度。不过，symbol 类型的转换却十分不便，因为其它类型缺乏与 symbol 的等同逻辑。特别是 symbol 无法强制转换为字符串与数字，因此它们不可能无意间被用作。。。。。。。。。。

本章中的示例使用了 console.log() 来输出并显示 symbol，之所以能这么做是因为 console.log() 会对 symbol 调用 String() 以得到有用的输出。你可以直接使用 String() 来得到相同的结果。例如：

```js
let uid = Symbol.for("uid"),
    desc = String(uid);

console.log(desc);              // "Symbol(uid)"
```

Strings() 函数调用 uid.toString()。如果你直接将它与字符串拼接，那么一个错误会被抛出：

```js
let uid = Symbol.for("uid"),
    desc = uid + "";            // 错误!
```

uid 与空字符串拼接首先需要将 uid 转换为字符串。为了防止这种行为的发生，一旦检测到转换操作即抛出错误。

同理，你也无法将 symbol 强制转换为数字类型。所有作用于 symbol 的算术运算符都会抛出错误。例如：

```js
let uid = Symbol.for("uid"),
    sum = uid / 1;            // 错误!
```

该例试图让 symbol 除以 1，于是一个错误发生了。其它算术运算符也是如此（和其它非空值类似，所有的 symbol 被认同为 true，所以逻辑运算符不会有错误抛出）。

<br />

### <a id="Retrieving-Symbol-Properties"> 提取 Symbol 属性（Retrieving Symbol Properties） </a>


Object.keys() 和 Object.getOwnPropertyNames() 方法可以提取一个对象中的属性名。前者返回所有的可枚举（自有）属性名，后者则无视可枚举性而返回所有的（自有）属性名。不过ECMAScript 5 及更早的版本中没有能返回 symbol 属性的方法。于是，ECMAScript 6 引入了 Object.getOwnPropertySymbols() 方法来提取对象中的 symbol 属性。

Object.getOwnPropertySymbols() 方法一个包含自有 symbol 属性的数组。例如：

```js
let uid = Symbol.for("uid");
let object = {
    [uid]: "12345"
};

let symbols = Object.getOwnPropertySymbols(object);

console.log(symbols.length);        // 1
console.log(symbols[0]);            // "Symbol(uid)"
console.log(object[symbols[0]]);    // "12345"
```

该段代码中，对象包含 uid 这个 symbol 属性。Object.getOwnPropertySymbols() 返回的数组中只有这个属性。

对象在初始时不包含自有的 symbol 属性，不过它们可以继承原型中的 symbol 属性。ECMAScript 6 预先定义了一些 symbol，这些实现被称为 well-known symbols 。

<br />

### <a id="Exposing-Internal-Operations-with-Well-Known-Symbols"> 揭秘内置 Well-Known Symbols 的运作（Exposing Internal Operations with Well-Known Symbols） </a>


ECMAScript 5 的主题之一是暴露和定义 JavaScript 中开发者无法模拟出的一些 “魔法”。ECMAScript 6 延续了该传统并暴露出比以往更多更甚的 JavaScript 内部逻辑，

ECMAScript 6 预定义了一些 symbol 并命名为 well-known symbols，以代表以前由内部操作的 JavaScript 公共行为。每一项 well-known symbol 都代表 Symbol 对象的一个属性，例如 Symbol.create 。

well-known symbol 包含：

* Symbol.hasInstance - instanceof 使用此方法来判断对象的实例。
* Symbol.isConcatSpreadable - 布尔值，表明 Array.prototype.concat() 是否应该在参数为集合（collection）的情况下扁平化（flatten）其中的元素。
* Symbol.iterator - 返回迭代器的方法（迭代器将在第七章中介绍）。
* Symbol.match - String.prototype.match() 使用此方法比较字符串。
* Symbol.replace - String.prototype.replace() 使用此方法替换子字符串。
* Symbol.search - String.prototype.search() 使用此方法查找子字符串。
* Symbol.species - 创建派生对象的构造函数（派生对象将在第八章讨论）。
* Symbol.split - 使用此方法来分割字符串。
* Symbol.toPrimitive - 对象使用此方法返回以基本类型描述的自身形式。
* Symbol.toStringTag - Object.prototype.toString() 使用此字符串创建对象的描述。
* Symbol.unscopables - 对象，with 语句的作用域内不包含它的属性。

一些普遍使用的 well-known symbols 会在接下来的小节内讨论，其它 well-known symbols 将在本书合适的位置进行介绍。

<br />

> 重写定义好的 well-known symbol 方法会将一个常规（ordinary）对象转变为特异（exotic）对象，因为它改变了内部定义的默认行为。不过这对你的代码没有实际的影响，只是该对象自身的描述方式根据规范发生了变化。

<br />

#### Symbol.hasInstance 属性（The Symbol.hasInstance Property）


每个函数都含有 Symbol.hasInstance 方法来判断一个给定的对象是否为它的实例。该方法在 Function.prototype 上定义，因此所有的方法都继承了 instanceof 属性的默认行为。Symbol.hasInstance 属性本身被定义为只读（nonwritable），不可配置（nonconfigurable）和不可枚举（nonenumerable），以保证不会由于某些错误被重写。

Symbol.hasInstance 方法接收单个参数：要查看的值。如果传入的值为该函数的实例，那么它会返回 true 。为了理解 Symbol.hasInstance 的工作原理，考虑如下的代码：

```js
obj instanceof Array;
```

这段代码等价于：

```js
Array[Symbol.hasInstance](obj);
```

ECMAScript 6 对 instanceof 操作符做出了必要的调整，将它重新定义为该方法的简写形式。既然它调用了方法，那么实际上你可以更改 instanceof 的工作方式。


比如，你不想让一个函数被实例化为对象，你可以将 Symbol.hasInstance 的返回值硬编码为 false，像这样：

```js
function MyObject() {
    // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return false;
    }
});

let obj = new MyObject();

console.log(obj instanceof MyObject);       // false
```

你必须使用 Object.defineProperty() 来重写一个只读属性，该例正是如此并将 Symbol.hasInstance 重写为另一个新的函数。该函数总是返回 false，即便 obj 真的是 MyObject 类的实例，instanceof 操作符在 Object.defineProperty() 调用后仍然返回 false 。

当然，你可以检查传入的值并设置任意的条件来决定它是否应被视为实例。例如，你或许会将 1 至 100 以内的数字视为 special number 类型的实例。为了实现该行为，你可能会书写如下的代码：

```js
function SpecialNumber() {
    // 空函数
}

Object.defineProperty(SpecialNumber, Symbol.hasInstance, {
    value: function(v) {
        return (v instanceof Number) && (v >=1 && v <= 100);
    }
});

let two = new Number(2),
    zero = new Number(0);

console.log(two instanceof SpecialNumber);    // true
console.log(zero instanceof SpecialNumber);   // false
```

该段代码定义了一个 Symbol.hasInstance 方法，如果传入的值为数字类型的实例且在 1 至 100 以内，那么它会返回 true 。因此，即使 SpecialNumber 函数和变量 two 之间没有直接的联系，SpecialNumber 仍认为 two 是它的实例。注意 instanceof 的左操作数必须是对象才能触发 Symbol.hasInstance 的调用，非对象使用 instanceof 总是简单地返回 false 。

<br />

> **注意**: 你同样可以重写所有内置函数地默认 Symbol.hasInstance 属性，例如 Date 和 Error 。然而这种做法不值得推荐，它会让你的代码难以琢磨且运行时出乎意料。只在必要的时候重写自定义函数的 Symbo.hasInstance 是个不错的主意。

<br />

#### Symbol.isConcatSpreadable Symbol（The Symbol.isConcatSpreadable Symbol）


JavaScript 数组包含一个 concat() 方法以拼接两个数组。以下是使用该方法的演示：

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ]);

console.log(colors2.length);    // 4
console.log(colors2);           // ["red","green","blue","black"]
```

该段代码将一个新的数组拼接在 colors1 的尾部并创建了包含两个数组所有元素的 colors2 。然而，concat() 方法也可以接收非数组参数并在该情况下简单地将它们添加到数组地尾部。例如：

```js
let colors1 = [ "red", "green" ],
    colors2 = colors1.concat([ "blue", "black" ], "brown");

console.log(colors2.length);    // 5
console.log(colors2);           // ["red","green","blue","black","brown"]
```

在这里，额外的参数 "brown" 被传递给 concat() 并成为了 colors2 第五个元素。为什么数组和字符串在作为参数的时候被区别对待了呢？ JavaScript 规范指出数组中的元素会自动分离成为单个项，而其它类型不能这么做。在 ECMAScript 6 之前，没有任何办法能对该行为做出调整。

Symbol.isConcatSpreadable 属性是布尔值，表明一个对象含有 length 属性和数字键（numeric key），且数字键的值在 concat() 调用时应该单独添加到结果当中。与其它 well-known symbol 不同，默认情况下它不存在于任何标准的对象之中。相反，该 symbol 可以决定 concat() 如何作用于特定的类型对象的并短路（short-circuiting）掉它们的默认行为。你可以定义任意类型使其在 concat() 的调用中行为类似于数组，像这样：

```js
let collection = {
    0: "Hello",
    1: "world",
    length: 2,
    [Symbol.isConcatSpreadable]: true
};

let messages = [ "Hi" ].concat(collection);

console.log(messages.length);    // 3
console.log(messages);           // ["hi","Hello","world"]
```

本例中，collection 对象的创建方式很像数组：包含 length 属性和两个数字键。Symbol.isConcatSpreadable 属性被设置为 true 以表示属性值应该单独添加到数组中。当 collection 传入 concat() 方法时，结果数组中 "Hello" 和 "world" 分别作为独立的项并排在 "hi" 元素之后。

<br />

> 你也可以将数组的子类中的 Symbol.isConcatSpreadable 设置为 false 以防止包含的项被 concat() 调用分离。子类将在第八章讨论。

<br />

#### Symbol.match，Symbol.replace，Symbol.search，和 Symbol.split（The Symbol.match, Symbol.replace, Symbol.search, and Symbol.split Symbols）


字符串与正则表达式在 JavaScript 中总是息息相关。特别是字符串类型，它含有一些以正则表达式为参数的方法：

* match(regex) - 判断给定的字符串是否匹配一个正则表达式
* replace(regex, replacement) - 将与正则表达式匹配的字符串置换为指定的替代品
* search(regex) - 查询字符串中正则表达式匹配项的位置
* split(regex) - 根据正则表达式匹配项将字符串分割为数组中的项

在 ECMAScript 6 之前，这些方法与正则表达式交互的实现对开发者是隐藏的，开发者无法使用自定义的对象来模仿正则表达式对象的行为。ECMAScript 6 定义了四个 symbol 以对应于上述四个方法，它们可以有效地传授内置正则表达式对象的先天行为。

当调用 match()，replace()，search()，split() 方法并传入正则表达式（第一个参数）时，Symbol.match，Symbol.replace，Symbol.search，和 Symbol.split 分别代表正则表达式参数应该调用的方法。RegExp.prototype 定义了这四个 symbol 属性作为默认实现以供字符串方法使用。

知道了这些，你可以使用与正则表达式对象相似的方法来创建可以由字符串方法操作的对象。为了这么做，你可以代码中使用如下的 symbol 函数：

* Symbol.match - 函数，接收一个字符串为参数并返回含有匹配项的数组，如未有匹配项则返回 null 。
* Symbol.replace - 函数，接收一个字符串和一个预备替换字符串为参数，返回值也是字符串。
* Symbol.search - 函数，接收一个字符串为参数并返回匹配项的数字索引，如未有匹配项则返回 -1 。
* Symbol.split - 函数，接收一个字符串为参数并返回由匹配分割的单独项的数组。

对象具有定义这些属性的能力意味着你可以在不使用正则表达式的情况下创建实现了模式匹配（pattern matching）的对象并将它们投放到需要正则表达式参数的函数。下面的示例展演示了如何使用这些 symbol：

```js
// 实际等同于 /^.{10}$/
let hasLengthOf10 = {
    [Symbol.match]: function(value) {
        return value.length === 10 ? [value.substring(0, 10)] : null;
    },
    [Symbol.replace]: function(value, replacement) {
        return value.length === 10 ?
            replacement + value.substring(10) : value;
    },
    [Symbol.search]: function(value) {
        return value.length === 10 ? 0 : -1;
    },
    [Symbol.split]: function(value) {
        return value.length === 10 ? ["", ""] : [value];
    }
};

let message1 = "Hello world",   // 11 个字符
    message2 = "Hello John";    // 10 个字符


let match1 = message1.match(hasLengthOf10),
    match2 = message2.match(hasLengthOf10);

console.log(match1);            // null
console.log(match2);            // ["Hello John"]

let replace1 = message1.replace(hasLengthOf10),
    replace2 = message2.replace(hasLengthOf10);

console.log(replace1);          // "Hello world"
console.log(replace2);          // "Hello John"

let search1 = message1.search(hasLengthOf10),
    search2 = message2.search(hasLengthOf10);

console.log(search1);           // -1
console.log(search2);           // 0

let split1 = message1.split(hasLengthOf10),
    split2 = message2.split(hasLengthOf10);

console.log(split1);            // ["Hello world"]
console.log(split2);            // ["", ""]
```

hasLengthOf10 对象试图模仿正则表达式的行为并匹配长度恰好为 10 的字符串。hasLengthOf10 用对应的 symbol 实现了这四个方法，之后相应的方法将会在字符串上调用。。第一个字符串 message1 含有 11 个字符，所以它不符合匹配规则；第二个字符串 message 含有 10 个字符串，于是它成为了匹配的元素。尽管 hasLengthOf10 不是正则表达式，但是根据内部附加的方法，它仍然会被正确的使用。

虽然这个示例比较简单，它却可以实行比正则表达式力所能及到的还要复杂的匹配，这就给自定义模式匹配提供了不少可能性。

<br />

#### Symbol.toPrimitive symbol（The Symbol.toPrimitive Symbol）


JavaScript 经常会在某些特定操作发生时尝试将对象隐式的转换为基本类型值。例如，当你使用双等号（==）运算符来比较字符串和对象时，对象会在比较发生前转换为基本类型值。在以前对象被转换为何种基本类型的值是由内部操作决定的，但是 ECMAScript 6 将该值的决定权通过 Symbol.toPrimitive 方法暴露了出来。

Symbol.toPrimitive 方法在各个标准类型的原型上都有一席之地，并指示对象在转换为基本类型值的过程中究竟要做些什么。当需要向基本类型转换时，Symbol.toPrimitive 会被调用并传入单个参数，规范中该参数为 hint 。hint 参数为三个字符串中的一个。如果 hint 为 "number"，那么 Symbol.toPrimitive 应该返回一个数字。如果 hint 为 "string"，那么 Symbol.toPrimitive 要返回字符串。如果 hint 为 "default"，那么返回值的类型没有特殊要求。

对大多数标准对象来讲，数字模式（number mode）包含如下的行为，优先级从上到下：

1. 调用 valueOf() 方法，如果结果为基本类型值则返回它。
2. 否则，调用 toString() 方法，并在结果为基本类型值的情况下返回它。
3. 否则，抛出错误。

同样，大多数标准对象的字符串模式（string mode）拥有如下的行为，优先级从上到下：

1. 调用 toString() 方法，如果结果为基本类型值则返回它。
2. 否则，调用 valueOf() 方法，并在结果为基本类型值的情况下返回它。
3. 否则，抛出错误。

在很多情况下，标准对象将数字模式视为默认模式（default mode）（Date 除外，默认模式视为字符串）。通过定义 Symbol.toPrimitive 方法，你可以重写这些默认的强制类型转换行为。

<br />

> 默认模式只由 == 和 + 操作符，以及向 Date 构造函数传递参数时被使用。大多数操作使用字符串或数字模式。

<br />

要重写默认的转换行为，请使用 Symbol.toPrimitive 并将一个函数赋给它。例如：

```js
function Temperature(degrees) {
    this.degrees = degrees;
}

Temperature.prototype[Symbol.toPrimitive] = function(hint) {

    switch (hint) {
        case "string":
            return this.degrees + "\u00b0"; // degrees symbol

        case "number":
            return this.degrees;

        case "default":
            return this.degrees + " degrees";
    }
};

let freezing = new Temperature(32);

console.log(freezing + "!");            // "32 degrees!"
console.log(freezing / 2);              // 16
console.log(String(freezing));          // "32°"，原文有误（째）
```

该段代码定义了 Temperature 构造函数并重写了原型上的默认 Symbol.toPrimitive 方法。根据 hint 参数是否为字符串，数字或是默认模式（hint 参数由 JavaScript 引擎自动填充）来返回相应的值。在字符串模式下，Temperature() 函数返回温度（temperature）和 Unicode 表示的温度符号。在数字模式下，它只返回数值。在默认模式下，它会在数字的后面添加单词 "degrees" 并返回整个字符串。

每个 log 语句都会触发不同的 hint 参数值。+，/ 操作符和 String() 函数分别将 hint 设置为 "default"，"number" 和 "string" 已触发默认，数字和字符串模式。三种模式返回不同的值是可行的，不过更常见的做法是设置默认模式等同为字符串或数字模式。

<br />

#### Symbol.toStringTag symbol（The Symbol.toStringTag Symbol）


One of the most interesting problems in JavaScript has been the availability of multiple global execution environments. This occurs in web browsers when a page includes an iframe, as the page and the iframe each have their own execution environments. In most cases, this isn’t a problem, as data can be passed back and forth between the environments with little cause for concern. The problem arises when trying to identify what type of object you’re dealing with after the object has been passed between different objects.

JavaScript 最有趣的问题之一在于它的多个执行环境可以同时并存。它发生在如下情况：浏览器加载的页面中包含一个 iframe，而页面和 iframe 分别拥有各自的执行环境。在大部分场景下，这都不是问题，因为数据可以在不同的环境中反复传递而不需要特别的去关心它们。如果对象在不同的对象之间互相传递后你想要确认某个对象的具体类型，那么麻烦就会出现。

The canonical example of this issue is passing an array from an iframe into the containing page or vice-versa. In ECMAScript 6 terminology, the iframe and the containing page each represent a different realm which is an execution environment for JavaScript. Each realm has its own global scope with its own copy of global objects. In whichever realm the array is created, it is definitely an array. When it’s passed to a different realm, however, an instanceof Array call returns false because the array was created with a constructor from a different realm and Array represents the constructor in the current realm.

这个问题的经典案例是将 iframe 中的数组传递给包含它的页面，或反过来将数组传递给页面中的 iframe。在 ECMAScript 6 中的术语中，iframe 与包含它的页面代表不同的场景（realm）——即 JavaScript 的一种执行环境。每个场景拥有自己的全局作用域和对应全局对象的拷贝。不论数组在哪个场景中创建，它都是真正的数组。不过，当将它传递给不同的场景并对其调用 Array 对象的 instanceof 方法后，结果会返回 false，因为该数组由不同场景中的数组构造器创建，然而 Array 仅代表当前场景中的构造器。

<br />

##### 鉴定问题的解决方案（A Workaround for the Identification Problem）


Faced with this problem, developers soon found a good way to identify arrays. They discovered that by calling the standard toString() method on the object, a predictable string was always returned. Thus, many JavaScript libraries began including a function like this:

面对以上问题，开发者迅速发觉了一种好的办法来确认数组。他们了解到，向对象调用标准的 toString() 方法总会返回一个可预测的字符串。因此渐渐地，很多 JavaScript 库开始引入了如下的函数：

```js
function isArray(value) {
    return Object.prototype.toString.call(value) === "[object Array]";
}

console.log(isArray([]));   // true
```

This may look a bit roundabout, but it worked quite well for identifying arrays in all browsers. The toString() method on arrays isn’t useful for identifying an object because it returns a string representation of the items the object contains. But the toString() method on Object.prototype had a quirk: it included internally-defined name called [[Class]] in the returned result. Developers could use this method on an object to retrieve what the JavaScript environment thought the object’s data type was.

这看起来有些兜圈子，不过它确实很好的解决了在所有浏览器中如何确认数组的问题。这种解决方案用来确认对象则不是那么有效，因为它总会返回以字符串表示的对象包含的项。但是 Object.prototype 的 toString() 方法有一个怪异（quirk）之处：返回的结果包含 [[Class]] 内部定义命名。开发者可以使用这个方法来获取 JavaScript 环境推断的对象类型。

Developers quickly realized that since there was no way to change this behavior, it was possible to use the same approach to distinguish between native objects and those created by developers. The most important case of this was the ECMAScript 5 JSON object.

开发者迅速认识到既然没有任何办法可以更改这个行为，那么使用相同的方式来区分原生（native object）和开发者自定义的对象也是可能的。最重要的案例就是 ECMAScript 5 的 JSON 对象。

Prior to ECMAScript 5, many developers used Douglas Crockford’s json2.js, which creates a global JSON object. As browsers started to implement the JSON global object, figuring out whether the global JSON was provided by the JavaScript environment itself or through some other library became necessary. Using the same technique I showed with the isArray() function, many developers created functions like this:

在 ECMAScript 5 之前，很多开发者使用 Douglas Crockford 编写的 json2.js 来创建全局 JSON 对象。当浏览器开始实现 JSON 全局对象时，如何区分全局 JSON 是由 JavaScript 环境提供还是由开发者自定义变得很有必要。使用如上演示过的 isArray() 函数包含的技巧，很多开发者创建了如下这样的函数：

```js
function supportsNativeJSON() {
    return typeof JSON !== "undefined" &&
        Object.prototype.toString.call(JSON) === "[object JSON]";
}
```

The same characteristic of Object.prototype that allowed developers to identify arrays across iframe boundaries also provided a way to tell if JSON was the native JSON object or not. A non-native JSON object would return [object Object] while the native version returned [object JSON] instead. This approach became the de facto standard for identifying native objects.

Object.prototype 允许开发者跨越 iframe 的边界来确认数组，同样它也可以告知 JSON 是否为原生丢向。一个非原生 JSON 对象会返回 [object Object]，相反原生对象会返回 [object JSON]。该种方案成为了区分原生对象的事实标准。

<br />

##### ECMAScript 6 的答案（The ECMAScript 6 Answer）


ECMAScript 6 redefines this behavior through the Symbol.toStringTag symbol. This symbol represents a property on each object that defines what value should be produced when Object.prototype.toString.call() is called on it. For an array, the value that function returns is explained by storing "Array" in the Symbol.toStringTag property.

ECMAScript 6 通过 Symbol.toStringTag Symbol 重新定义了以上行为。该 Symbol 代表每个对象上都存在的一个属性，每次调用 Object.prototype.toString.call() 会返回这个属性值。对于数组来讲，该函数的返回值可以被解释为 "Array" 字符串作为值被存储到了 Symbol.toStringTag 属性中。

Likewise, you can define the Symbol.toStringTag value for your own objects:

类似的是，你可以给自己的对象定义 Symbol.toStringTag 的值：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

let me = new Person("Nicholas");

console.log(me.toString());                         // "[object Person]"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

In this example, a Symbol.toStringTag property is defined on Person.prototype to provide the default behavior for creating a string representation. Since Person.prototype inherits the Object.prototype.toString() method, the value returned from Symbol.toStringTag is also used when calling the me.toString() method. However, you can still define your own toString() method that provides a different behavior without affecting the use of the Object.prototype.toString.call() method. Here’s how that might look:

本例中，为了给对象创建一个字符串表达形式，便定义了 Symbol.toStringTag 属性来提供默认的行为。因为 Person.prototype 继承了 Object.prototype.toString() 方法，Symbol.toStringTag 的值会被 me.toString() 方法使用。不过，你依然可以定义自己的 toString() 方法，在不影响 Object.prototype.toString.call() 方法的前提下提供另一种不同的行为。

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";

Person.prototype.toString = function() {
    return this.name;
};

let me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Person]"
```

This code defines Person.prototype.toString() to return the value of the name property. Since Person instances no longer inherit the Object.prototype.toString() method, calling me.toString() exhibits a different behavior.

上段代码定义了 Person.prototype.toString() 并返回 name 属性。由于 Person 实例不再继承 Object.prototype.toString() 方法，调用 me.toString() 会展现不同的行为。

<br />

> All objects inherit Symbol.toStringTag from Object.prototype unless otherwise specified. The string "Object" is the default property value.

> 所有的对象都继承了 Object.prototype 上的 Symbol.toStringTag。除非特别设置，"Object" 字符串是默认的属性值。

<br />

There is no restriction on which values can be used for Symbol.toStringTag on developer-defined objects. For example, nothing prevents you from using "Array" as the value of the Symbol.toStringTag property, such as:

在开发者定义的对象上 Symbol.toStringTag 的值没有任何限制。例如，没有什么能阻止你将 "Array" 设置为 Symbol.toStringTag 的属性值，例如：

```js
function Person(name) {
    this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Array";

Person.prototype.toString = function() {
    return this.name;
};

let me = new Person("Nicholas");

console.log(me.toString());                         // "Nicholas"
console.log(Object.prototype.toString.call(me));    // "[object Array]"
```

The result of calling Object.prototype.toString() is "[object Array]" in this code, which is the same result you’d get from an actual array. This highlights the fact that Object.prototype.toString() is no longer a completely reliable way of identifying an object’s type.

本段代码中调用 Object.prototype.toString() 的结果是 "[object Array]"，它也是在真正的数组上调用该方法获得的结果。这突出说明了在确认对象类型时，Object.prototype.toString() 已经不能完全信赖。

Changing the string tag for native objects is also possible. Just assign to Symbol.toStringTag on the object’s prototype, like this:

更改原生对象的字符串标签也是可以的，只需要在对象原型上向 Symbol.toStringTag 赋值。如下所示：

```js
Array.prototype[Symbol.toStringTag] = "Magic";

let values = [];

console.log(Object.prototype.toString.call(values));    // "[object Magic]"
```

Even though Symbol.toStringTag is overwritten for arrays in this example, the call to Object.prototype.toString() results in "[object Magic]" instead. While I recommended not changing built-in objects in this way, there’s nothing in the language that forbids doing so.

即使上例只重写了数组的 Symbol.toStringTag，对其调用 Object.prototype.toString() 的结果仍然是 [object Magic]。虽然我不推荐像上例中更改内置对象，但是语言本身并没有任何办法禁止你这样做。

<br />

#### Symbol.unscopables Symbol（The Symbol.unscopables Symbol）


The with statement is one of the most controversial parts of JavaScript. Originally designed to avoid repetitive typing, the with statement later became roundly criticized for making code harder to understand and for negative performance implications as well as being error-prone.

with 语句是 JavaScript 中最具争议的部分之一。它起先的设计目的是用来避免重复书写代码，不过在那之后，with 语句因为晦涩难懂和对性能的消极影响被饱受批评，同时它也存在很多隐患。

As a result, the with statement is not allowed in strict mode; that restriction also affects classes and modules, which are strict mode by default and have no opt-out.

因此，严格模式下 with 语句被禁止使用，在类和模块中也不允许它的存在，因为两者默认以严格模式运行，而且没有办法妥协。

While future code will undoubtedly not use the with statement, ECMAScript 6 still supports with in nonstrict mode for backwards compatibility and, as such, had to find ways to allow code that does use with to continue to work properly.

虽然在未来的代码编写中 with 语句毫无疑问会被弃用，但 ECMAScript 6 为了向后兼容仍然支持在非严格模式下使用它。因此，必须找到一种方式使得 with 能继续正常工作。

To understand the complexity of this task, consider the following code:

为了了解这项任务的复杂程度，考虑如下的代码：

```js
let values = [1, 2, 3],
    colors = ["red", "green", "blue"],
    color = "black";

with(colors) {
    push(color);
    push(...values);
}

console.log(colors);    // ["red", "green", "blue", "black", 1, 2, 3]
```

In this example, the two calls to push() inside the with statement are equivalent to colors.push() because the with statement added push as a local binding. The color reference refers to the variable created outside the with statement, as does the values reference.

本例中，在 with 语句中两次调用的 push() 等同于两次调用 colors.push()，因为 with 语句将 push 添加到局部绑定中。color 引用指代 with 语句外创建的变量，values 同理。

But ECMAScript 6 added a values method to arrays. (The values method is discussed in detail in Chapter 7, “Iterators and Generators.”) That would mean in an ECMAScript 6 environment, the values reference inside the with statement should refer not to the local variable values, but to the array’s values method, which would break the code. This is why the Symbol.unscopables symbol exists.

但是 ECMAScript 6 向数组添加了 values 方法（values 方法将在第八章“迭代器与生成器”中详细介绍），这意味着在 ECMAScript 6 的执行环境中，with 语句中的 values 引用指代的并非局部变量 values，而是数组的 values 方法，于是代码无法正常运行。为了解决这个问题，Symbol.unscopables 应运而生。

The Symbol.unscopables symbol is used on Array.prototype to indicate which properties shouldn’t create bindings inside of a with statement. When present, Symbol.unscopables is an object whose keys are the identifiers to omit from with statement bindings and whose values are true to enforce the block. Here’s the default Symbol.unscopables property for arrays:

Array.prototype 根据 Symbol.unscopables symbol 来指示 with 语句中应该创建哪些绑定。Symbol.unscopables 以对象的形式存在，with 语句根据这个对象的属性标识符来确定代码块内应该存在哪些绑定。以下是数组默认的 Symbol.unscopables 值：

```js
// built into ECMAScript 6 by default
// ECMAScript 6 的默认配置
Array.prototype[Symbol.unscopables] = Object.assign(Object.create(null), {
    copyWithin: true,
    entries: true,
    fill: true,
    find: true,
    findIndex: true,
    keys: true,
    values: true
});
```

The Symbol.unscopables object has a null prototype, which is created by the Object.create(null) call, and contains all of the new array methods in ECMAScript 6. (These methods are covered in detail in Chapter 7, “Iterators and Generators,” and Chapter 9, “Arrays.”) Bindings for these methods are not created inside a with statement, allowing old code to continue working without any problem.

Symbol.unscopables 对象拥有一个空的原型（null prototype），它由 Object.create(null) 创建，并包含所有 ECMAScript 6 为数组添加的新方法（这些方法会在第八章“迭代器与生成器”和第十章“改进的数组功能”中详细说明）。with 语句中不会创建这些方法的绑定，以便让旧的代码正常运行。

In general, you shouldn’t need to define Symbol.unscopables for your objects unless you use the with statement and are making changes to an existing object in your code base.

总的来讲，除非你使用了 with 语句并对代码库中存在的对象进行变动，你不需要给对象定义 Symbol.unscopables。

<br />

### <a id="Summary> 总结（Summary） </a>


Symbols are a new type of primitive value in JavaScript and are used to create nonenumerable properties that can’t be accessed without referencing the symbol.

Symbol 是 JavaScript 中一种新的基本类型，它被用来创建不可枚举的属性而且只能通过引用 symbol 来访问。

While not truly private, these properties are harder to accidentally change or overwrite and are therefore suitable for functionality that needs a level of protection from developers.

虽然它们并非真正的私有属性，不过对开发者来讲，这些属性很难被意外修改和覆盖的特点使得它们很适合为某些设计功能做一定级别的防护。

You can provide descriptions for symbols that allow for easier identification of symbol values. There is a global symbol registry that allows you to use shared symbols in different parts of code by using the same description. In this way, the same symbol can be used for the same reason in multiple places.

为了让 symbol 值更易辨识，你可以为其提供一些描述。全局 symbol 记录的存在让你可以在代码的不同片段中通过相同的描述来使用共享的 symbol。同理，相同的 symbol 可以在多处使用。

Methods like Object.keys() or Object.getOwnPropertyNames() don’t return symbols, so a new method called Object.getOwnPropertySymbols() was added in ECMAScript 6 to allow retrieval of symbol properties. You can still make changes to symbol properties by calling the Object.defineProperty() and Object.defineProperties() methods.

类似于 Object.keys() 或 Object.getOwnPropertyNames() 这样的方法无法返回 symbol，于是 ECMAScript 6 引入了 Object.getOwnPropertySymbols() 这个新方法来提取 symbol 属性。你仍旧可以通过 Object.defineProperty() 或 Object.defineProperties() 来修改 symbol 属性。

Well-known symbols define previously internal-only functionality for standard objects and use globally-available symbol constants, such as the Symbol.hasInstance property. These symbols use the prefix Symbol. in the specification and allow developers to modify standard object behavior in a variety of ways.

well-known symbol 定义了标准对象中在以前只能由内部运作的功能。它们是全局可用的 symbol 常量且带有 Symbol. 前缀，例如 Symbol.hasInstance 属性，并允许开发者以各种各样的方式来修改标准对象的行为

<br />
