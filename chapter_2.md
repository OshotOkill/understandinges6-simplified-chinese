# 字符串与正则表达式（Strings and Regular Expressions）


字符串可以说是程序设计中最为重要的数据类型之一。几乎每种高级编程语言都有它的一席之地，而且能有效的使用它也是开发者编写实用程序的基本准则。作为重要的扩展，正则表达式赋予开发者操作字符串的额外能力。ECMAScript 6 的缔造者们将这些事实牢记于心，改进了字符串和正则表达式，并添加了长久以来缺失的某些功能。本章会讲解它们的变化之处。

<br />
### 本章小结

* [更佳的 Unicode 支持](#Better-Unicode-Support)
* [字符串的其它改进](#Other-String-Changes)
* [正则表达式的其它改进](#Other-Regular-Expression-Changes)
* [模板字面量](#Template-Literals)
* [总结](#Summary)

<br />

> **注意**： gitbook 无法正常解析 \$\$ 字符，所以在模板字面量一节中 \$ \$ 实际上为 \$\$ 。

<br />

### <a id ="Better-Unicode-Support"> 更佳的 Unicode 支持（Better Unicode Support） </a>


ECMAScript 6 诞生之前，JavaScript 字符串（string）由 16 位编码的字符组成（UTF-16）。每个字符又由包含一个 16 位序列的代码单元（code unit）表示。所有的字符串属性和方法，例如 length 和 charAt()，都基于这些 16 位编码单元。曾经，16 位的容量对于任意字符的存放都是足够的，然而现在不是了，辛亏 Unicode 引入了扩展字符集（expanded character set）。

<br />

#### UTF-16 代码点（UTF-16 Code Points）


限制字符的长度在 16 位以内难以满足 Unicode 意图给世界上所有字符提供全局唯一标识符的雄心壮志。这些全局唯一标识符，也被称为代码点，仅是从 0 开始的数字。字符代码（character code）可能是你所想象的那样，由一个数字（代码点）来代表一个对应的字符。编码一个字符就必须要将代码点转换为对应一致的代码单元。对于 UTF-16 来讲，代码点可以由多个代码单元组成。

UTF-16 的前 2<sup>16</sup> 个代码点由单个 16 位代码单元表示。这个范围被称作基本多语言面（Basic Multilingual Plane，BMP）。任何超出该范围的部分都是增补的语言面（supplementary plane），代码点将不能被单一的 16 位代码单元表示。为了解决这个问题，UTF-16 引入了代理项对（surrogate pair）来让两个 16 位代码单元表示一个代码点。这意味着字符既可能是包含单个代码单元的 16位 BMP 字符，也可能是由两个代码单元组成的位于增补语言面的 32 位字符。

在 ECMAScript 5 中，所有的字符串操作针对的是 16 位代码单元，意味着如果编码的 UTF-16 字符串包含代理项对，那么可能会得到意想不到的结果，如下所示：

```js
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(text.charAt(0));        // ""
console.log(text.charAt(1));        // ""
console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
```

单个 Unicode 字符 "𠮷" 由代理项对表示，因此，本例中 JavaScript 在操作该字符串时会将它视为两个 16 位字符。这意味着：

* text 的长度为 2，即使它看起来是 1 。
* 试图匹配单个字符的正则表达式会以失败告终，因为表达式将其视作两个字符。
* charAt() 方法无法返回一个有效的字符串，因为包含的两个 16 位代码单元都无法被打印（printable）。

charCodeAt() 方法也不能正确识别字符。它返回的是对应代码单元的 16 位数字，但是在 ECMAScript 5 中这已经是所能获取到的最精确的文本值了。

另一方面，ECMAScript 6 要求以上 UTF-16 字符的编码问题必须得到解决。标准化基于新字符编码规范的字符串操作意味着 JavaScript 支持专门处理代理项对的功能。本章剩下的部分讨论了使用该功能的几个关键案例。

<br />

#### codePointAt() 方法（The codePointAt() Method）


为了全面支持 UTF-16，ECMAScript 6 新添加的方法之一就是 codePointAt()，它可以提取给定位置字符串的对应 Unicode 代码点。该方法接收代码单元而非字符的位置并返回一个整型值，正如下例中所展示的那样：

```js
var text = "𠮷a";

console.log(text.charCodeAt(0));    // 55362
console.log(text.charCodeAt(1));    // 57271
console.log(text.charCodeAt(2));    // 97

console.log(text.codePointAt(0));   // 134071
console.log(text.codePointAt(1));   // 57271
console.log(text.codePointAt(2));   // 97
```

除非操作的是非 BMP 字符，否则 codePointAt 方法的返回值与 charCodeAt() 相同。示例中的首个字符并没有位于 BMP 范围内，因此它包含两个代码单元，意味着 length 属性是 3 而不是 2 。charCodeAt() 方法返回的只是处于位置 0 的第一个代码单元，而 codePointAt() 返回的是完整的代码点，即使它分配给了多个代码单元。位置 1 （第一个字符的第二个代码单元）和位置 2 的值，两个方法返回的结果是相同的。

对一个字符调用 codePointAt() 方法是判断它所包含代码单元数量的最容易的方法。你可以书写下面的方法：

```js
function is32Bit(c) {
    return c.codePointAt(0) > 0xFFFF;
}

console.log(is32Bit("𠮷"));         // true
console.log(is32Bit("a"));          // false
```

16 位字符的上边界由十六进制 FFFF 表示，所以任何大于该数字的代码点必定由两个代码单元表示，总大小为 32 位。

<br />

#### String.fromCodePoint() 方法（The String.fromCodePoint() Method）


每当 ECMAScript 提供了完成某件事的方法，一般情况下它也会给出相反操作的解决方案。你可以使用 codePointAt() 来提取字符串中某个字符的代码点，也可以调用 String.fromCodePoint() 并使用给定的代码点来产生相应的单个字符。例如：

```js
console.log(String.fromCodePoint(134071));  // "𠮷"
```

可以将 String.fromCharCode() 视为 String.fromCharCode() 的完善版本。针对 BMP 字符两者会产生相同的结果，只有 BMP 之外的字符才会有差异。

<br />

#### normalize() 方法（The normalize() Method）


Unicode 另一个有趣的方面是，不同的字符在某些排序或比较操作中被认为是相同的。有两种方式可以确立两者之间的关系。第一，规范相等性（canonical equivalence）意味着两个代码点序列在各个方面都可以进行互换。例如，两个字符的组合根据规范相等性可以等同于一个字符。第二，是字符间的兼容性。两个兼容的代码点序列可能看上去不同，实际上在特定条件下可以互换（interchangeably）。

由于这些关系的存在，两个在根本（fundamentally）上相同的字符串可以包含不同的代码点序列。例如，字符 "æ" 和两个字符组成的 "ae" 字符串或许可以互换，但是除非以某种形式进行规范化，否则严格意义上讲它们并不相等。

ECMAScript 6 通过给字符串提供 normalize() 方法来支持 Unicode 规范格式。该方法接收一个字符串参数来获取以下 Unicode 规范格式中的一种并据此运行：

* Normalization Form Canonical Composition ("NFC"), 默认的规范格式
* Normalization Form Canonical Decomposition ("NFD")
* Normalization Form Compatibility Composition ("NFKC")
* Normalization Form Compatibility Decomposition ("NFKD")

解释这四种格式的差异超出了本书的范围。只需记住，当比较字符串时，它们必须被规范成同一种格式。例如：

```js
var normalized = values.map(function(text) {
    return text.normalize();
});

normalized.sort(function(first, second) {
    if (first < second) {
        return -1;
    } else if (first === second) {
        return 0;
    } else {
        return 1;
    }
});
```

该段代码讲 values 数组中的字符串转换为一种规范格式以便让元素可以被正确的排序。你也可以在比较函数中调用 normalize() 来对原始数组进行排序操作。如下所示：

```js
values.sort(function(first, second) {
    var firstNormalized = first.normalize(),
        secondNormalized = second.normalize();

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

该段代码再一次需要注意的是，first 和 second 都由同一种格式规范。这些示例都使用了默认的 NFC 格式，不过，使用其它格式也相当容易，像这样：

```js
values.sort(function(first, second) {
    var firstNormalized = first.normalize("NFD"),
        secondNormalized = second.normalize("NFD");

    if (firstNormalized < secondNormalized) {
        return -1;
    } else if (firstNormalized === secondNormalized) {
        return 0;
    } else {
        return 1;
    }
});
```

如果之前你从未担心过 Unicode 的规范化问题，那么暂时你可能用不到这个方法。倘若你曾经开发过一款国际化应用，你绝对会认为 normalize() 方法相当有用。

这些方法并不是 ECMAScript 6 针对 Unicode 字符串的全部改进。它还引入了两项实用的语法。

<br />

#### 正则表达式的 u 标志（The Regular Expression u Flag）


你可以使用正则表达式来完成很多字符串的基本操作。但是你需要牢记，正则表达式针对的是 16 位代码单元表示的单个字符。为了解决这个问题，ECMAScript 6 为正则表达式定义了代表 Unicode 字符的 u 标志。

<br />

##### u 标志的使用场景（The u Flag in Action）


当一个正则表达式启用了 u 标志符时，它将切换模式以作用于字符，而不是代码单元。这意味着正则表达式面对字符串中的代理项对时不再迷茫，并如预期的那样工作。例如，考虑以下的代码：

```js
var text = "𠮷";

console.log(text.length);           // 2
console.log(/^.$/.test(text));      // false
console.log(/^.$/u.test(text));     // true
```

正则表达式 /&.$/ 匹配单个字符的输入。当不使用 u 标志时，正则表达式匹配的是代码单元，所以它不能匹配日文字符（由两个代码单元表示）。启用 u 标志后，正则表达式比较字符而不是代码单元，所以日文字符会被匹配到。

<br />

##### 代码点的数量（Counting Code Points）


遗憾的是，ECMAScript 6 并没有添加能判断字符串所包含代码单元个数的方法，但是通过 u 标志，你可以使用正则表达式来进行计算它们，如下所示：

```js
function codePointLength(text) {
    var result = text.match(/[\s\S]/gu);
    return result ? result.length : 0;
}

console.log(codePointLength("abc"));    // 3
console.log(codePointLength("𠮷bc"));   // 3
```

该例调用了 match() 方法和 gu 标志来全局检查 text 中的空白和非空白字符（使用 [\s\S] 以确保匹配换行符），当至少有一个以上的匹配项时，返回包含它们的数组，所以数组的长度即为字符串中的代码单元数量。在 Unicode 中，字符串 "abc" 和 "𠮷bc" 同时含有三个字符，所以返回的数组长度为 3 。

<br />

> **注意**： 虽然该实现可以正常工作，但是它的运行速度并不快，尤其是操作一个长字符串。你也可以使用字符串迭代器来实现相同的需求（第八章讨论）。一般来讲，尽可能的减少需要操作的字符串中的代码点数量。

<br />

##### 判断是否支持 u 标志（Determining Support for the u Flag）


既然 u 标志是一项语法变更，在不兼容 ECMAScript 6 的 JavaScript 引擎中使用它会抛出一个语法错误。使用一个函数来判断是否支持 u 标志是最安全的方法，像这样：

```js
function hasRegExpU() {
    try {
        var pattern = new RegExp(".", "u");
        return true;
    } catch (ex) {
        return false;
    }
}
```

该函数使用 RegExp 构造函数并传递 u 标志作为参数。该语法即使在旧版本的 JavaScript 引擎中都是有效的，但是构造函数在不识别 u 标志的情况下会抛出一个错误。

如果你的代码仍然需要在老旧的 JavaScript 引擎中运行，那么最好在构造函数中使用 u 标志。这会预防语法错误的发生并允许你选择性地检测和使用 u 标志，而且代码执行不会被中断。

<br />

### 字符串的其它改进（Other String Changes）


JavaScript 的字符串特性总是落后于其它语言。比如，直到 ECMAScript 5 字符串才总算拥有了 trim() 方法，ECMAScript 6 则继续添加新功能以扩展 JavaScript 解析字符串的能力。

<br />

#### 确认子字符串的方法（Methods for Identifying Substrings）


自 JavaScript 引入了 indexOf() 方法后，开发者们使用它来确认字符串是否存在于其它字符串中。ECMAScript 6 包含以下三种方法来满足相同的需求：

* includes() 方法会在给定文本存在于字符串中的任意位置时返回 true，否则返回 false 。
* startsWith() 方法会在给定文本出现在字符串开头时返回 true，否则返回 false 。
* endsWith() 方法会在给定文本出现在字符串末尾时返回 true，否则返回 false 。

每个方法都接收两个参数：需要搜索的文本和可选的起始索引值。当提供第二个参数后，includes() 和 startsWith() 会以该索引为起始点进行匹配，而 endsWith() 将字符串的长度与参数值相减并将得到的值作为检索的起始点。若第二个参数未提供，includes() 和 startsWith() 会从字符串的起始中开始检索，endsWith() 则是从字符串的末尾。实际上，第二个参数减少了需要检索的字符串的总量。以下是使用这些方法的演示：

```js
var msg = "Hello world!";

console.log(msg.startsWith("Hello"));       // true
console.log(msg.endsWith("!"));             // true
console.log(msg.includes("o"));             // true

console.log(msg.startsWith("o"));           // false
console.log(msg.endsWith("world!"));        // true
console.log(msg.includes("x"));             // false

console.log(msg.startsWith("o", 4));        // true
console.log(msg.endsWith("o", 8));          // true
console.log(msg.includes("o", 8));          // false
```

前三次调用不包含第二个参数，所以它们在必要的情况下会检索整个字符串。最后三次调用只会检索字符串的一部分。调用 msg.startsWith("o", 4) 会从索引值为 4 ，即 "hello" 中的 o 处开始检索。调用 msg.endsWith("o", 8) 的起始检索位置和前者相同，因为参数 8 会由字符串的长度值（12）减去。调用 msg.includes("o", 8) 则会从索引值为 8，即 "world" 中的 "r" 处开始检索。

虽然这三个方法使得判断子字符串是否存在变得更容易，但是它们返回的只是一个布尔（boolean）值。如果你需要返回它们在另一个字符串中存在的确切位置，请使用 indexOf() 和 lastIndexOf() 。

<br />

> **注意**： 向 startsWith()，endsWith()，和 includes() 方法传入正则表达式会抛出错误，这和 indexOf() 与 lastIndexOf() 的表现相反，它们会将正则表达式转换为字符串并搜索它。

<br />

#### repeat() 方法（The repeat() Method）


ECMAScript 6 还向字符串添加了 repeat() 方法，它接受一个数字参数作为字符串的重复次数。该方法返回一个重复包含初始字符串的新字符串，重复次数等于参数。例如：

```js
console.log("x".repeat(3));         // "xxx"
console.log("hello".repeat(2));     // "hellohello"
console.log("abc".repeat(4));       // "abcabcabcabc"
```

该方法在同类中最为便利，在操作文本的场景中特别实用。当实现了格式化代码的实用程序（utility）需要添加缩进时，该方法非常适合，像这样：

```js
// 使用指定的数字添加空格缩进
var indent = " ".repeat(4),
    indentLevel = 0;

// 增加缩进
var newIndent = indent.repeat(++indentLevel);
```

首次调用 repeat() 创建包含 4 个空格的字符串，indentLevel 变量会持续追踪缩进的级别。之后，你可以仅通过增加 repeat() 的参数值来改变空格数量。

ECMAScript 6 同样为正则表达式添加了一些实用的功能来增加它们的适用场景。下一节会简要的介绍它们。

<br />

### 正则表达式的其它改进（Other Regular Expression Changes）


正则表达式是在 JavaScript 中操作字符串的重要方式之一，和很多其它语言相似，它在最近的几个版本中并未发生太大的变化。不过，为了和针对字符串的修改一起作伴，ECMAScript 6 也给正则表达式做了一些改进。

<br />

#### 正则表达式的 y 标志（The Regular Expression y Flag）


ECMAScript 6 将 Firefox 对正则表达式拓展的私有 y 标志纳入了标准。y 标志涉及正则表达式与检索相关的 sticky 属性，它表示从正则表达式设置的 lastIndex 属性值的位置开始检索字符串中的匹配字符。如果以该位置为起点，之后没有相应的匹配，那么正则表达式将停止检索。为了展示它是如何工作的，考虑如下的代码：

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

pattern.lastIndex = 1;
globalPattern.lastIndex = 1;
stickyPattern.lastIndex = 1;

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // Error! stickyResult is null
```

该例分别创建了未启用标志，启用 g 标志和启用 y 标志以使用粘滞模式（stickyPattern）的三个正则表达式。前三次 console.log() 调用后，它们都返回了末尾带有空格的 "hello " 字符串。

在那之后，三种匹配模式的 lastIndex 属性全部更改为 1，意味着它们的匹配起始位置为第二个字符。无标志位的正则表达式会完全忽略掉 lastIndex 并像往常那样返回 "hello1 " 匹配。带有 g 标志位的正则表达式返回的匹配是 "hello2 "，因为它的搜索起点为第二个字符（"e"）。启用粘滞模式的正则表达式在第二个字符之后没有获得任何匹配，所以 stickyResult 为 null 。

匹配操作一旦发生，带有粘滞标志位的匹配模式会保存上次匹配结果末尾的下一个字符的索引。如果一次匹配没有获得任何结果，那么 lastIndex 的值将重置为 0 。全局标志位的行为与其相同，如下所示：

```js
var text = "hello1 hello2 hello3",
    pattern = /hello\d\s?/,
    result = pattern.exec(text),
    globalPattern = /hello\d\s?/g,
    globalResult = globalPattern.exec(text),
    stickyPattern = /hello\d\s?/y,
    stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello1 "
console.log(stickyResult[0]);   // "hello1 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 7
console.log(stickyPattern.lastIndex);   // 7

result = pattern.exec(text);
globalResult = globalPattern.exec(text);
stickyResult = stickyPattern.exec(text);

console.log(result[0]);         // "hello1 "
console.log(globalResult[0]);   // "hello2 "
console.log(stickyResult[0]);   // "hello2 "

console.log(pattern.lastIndex);         // 0
console.log(globalPattern.lastIndex);   // 14
console.log(stickyPattern.lastIndex);   // 14
```

首次和第二次调用 exec() 之后，stickyPattern 和 globalPattern 的 lastIndex 属性值均分别为 7 和 14 。

关于粘滞标志位，有两处微妙的细节需要牢记：

1. 只有在正则表达式对象上（regular expression object）上调用方法 lastIndex 才会生效，例如 exec() 和 test()。如果将正则表达式传递给 match() 这样的字符串方法，粘滞行为则无法体现。
2. 当使用 ^ 字符来匹配字符串的首部时，粘滞正则表达式只会匹配整个字符串的首部位置（或多行模式下的行首）。如果 lastIndex 为 0，那么不论正则表达式是否带有粘滞标志位，^ 的表现均一致。

和其它正则表达式标志位相同，你可以根据属性来探测 y 是否存在。你可以如下检查 sticky 属性：

```js
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
```

如果粘滞标志存在，那么 sticky 属性为 true，否则为 false 。sticky 作为只读属性仅被用来检查 y 标志的存在而不能在代码中修改其值。

和 u 标志类似，y 标志同样是个语法变动，所以在旧的 JavaScript 引擎中它会造成语法错误。你可以使用下面的方式来检测：

```js
function hasRegExpY() {
    try {
        var pattern = new RegExp(".", "y");
        return true;
    } catch (ex) {
        return false;
    }
}
```

这段代码类似于 u 标志的检查，即如果不能使用 y 标志来创建正则表达式则返回 false 。同样，为了避免语法错误的发生，请确保在旧的 JavaScript 引擎中使用 RegExp 构造函数来定义使用 y 标志的正则表达式。

<br />

#### 正则表达式副本（Duplicating Regular Expressions）


在 ECMAScript 5 中，你可以将正则表达式传递给 RegExp 构造函数来创建它的副本，例如：

```js
var re1 = /ab/i,
    re2 = new RegExp(re1);
```

re2 变量只是 re1 的一个拷贝。但如果你为 RegExp 构造函数提供了第二个参数，即正则表达式的标志位，那么该段代码就失效了，正如下例所示：

```js
var re1 = /ab/i,

    // ES6 中尚可，但是在 ES5 中会抛出错误
    re2 = new RegExp(re1, "g");
```

如果你在 ECMAScript 5 中运行这段代码，那么你会收到一段错误说明，表示你不该在第一个参数已经为正则表达式的情况下提供第二个参数。ECMAScript 6 修改了上述行为，允许使用第二个参数且覆盖掉第一个参数中的标志位。例如：

```js
var re1 = /ab/i,

    // ES6 中尚可，但是在 ES5 中会抛出错误
    re2 = new RegExp(re1, "g");


console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"

console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true

console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false
```

该段代码中，re1 带有忽略匹配项大小写的 i 标志，re2 则含有 g 全局标志。RegExp 构造函数创建了 re1 匹配规则的副本并将 i 标志替换为 g 。如果第二个参数未提供，那么 re2 的标志和 re1 相同。

<br />

#### 标志位属性（The flags Property）


ECMAScript 6 在添加了新的标志位并改变了已有的一些标志位的行为的同时还为它们添加了一个关联属性。在 ECMAScript 5 中，你可以使用 source 属性来获得正则表达式中的文本，不过你若想获得字符串表示的标志位，则需要像下面这样解析 toString() 方法输出的内容：

```js
function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString() 输出 "/ab/g"
var re = /ab/g;

console.log(getFlags(re));          // "g"
```

正则表达式会被转换为字符串并返回最后一个 / 之后的字符，即标志位。

ECMAScript 6 添加了 flags 属性使其和 source 属性作伴，于是标志位字符串的获取容易了许多。两者均为原型上的 getter 访问器属性，因此它们是只读的。flags 属性令正则表达式调试与继承的分析工作变得更加轻松。

flags 属性返回正则表达式所有标志位的字符串形式。例如：

```js
var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g"
```

本例使用了比 toString() 方案少得多的代码并向控制台输出了 re 所有的标志位。同时使用 source 和 flags 允许你提取正则表达式的片段而不需要直接解析正则表达式本身。

目前为止本章介绍的字符串和正则表达式的改善的意义无疑是重大的，不过 ECMAScript 6 对字符串还有一项重大改进，它引入了一种新的字面量形式使得字符串的使用更加灵活。

<br />

### 模板字面量（Template Literals）


JavaScript 中的字符串相比其它语言有着太多的限制。例如，在 ECMAScript 6 之前本章介绍过的字符串的所有新方法都不能使用，而且字符串的拼接方式过于简陋。为了能让开发者解决更复杂的问题，ECMAScript 6 中的模板字面量提供了创建领域特定语言（domain-specific languages, DSLs）的语法使其相比 ECMAScript 5 或更早的版本能更安全的操作相应的内容（领域特定语言面向且专注于的是某单一特定目标的计算机程序设计语言，与通用目的语言如 JavaScript 相反）。ECMAScript wiki 提供了 [template literal strawman](http://wiki.ecmascript.org/doku.php?id=harmony:quasis) 的如下描述：

<br />

> 本方案通过语法糖扩展了 ECMAScript 的语法并允许库提供 DSLs 以便产生，查询并操作其它语言的相关内容且对 XSS，SQL 注入等攻击免疫或具有抗性。

<br />

不过实际上，模板字面量是 ECMAScript 6 针对 JavaScript 直到 ECMAScript 5 依然缺失的如下功能的回应：

* **多行字符串** 针对多行字符串的形式概念（formal concept）。
* **基本的字符串格式化** 将字符串中的变量置换为值的能力。
* **转义 HTML** 能将字符串进行转义并使其安全地插入到 HTML 的能力。

模板字面量以一种全新的表现形式解决了这些问题而不需要向 JavaScript 已有的字符串添加额外的功能。

<br />

#### 基本语法（Basic Syntax）


简言之，模板字面量由反引号（`）而非一般字符串使用的单或双引号囊括。考虑如下的例子：

```js
let message = `Hello world!`;

console.log(message);               // "Hello world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 12
```

该段代码证实 message 变量包含的是一个普通的 JavaScript 字符串。模板字面量语法创建了一个字符串并将其赋值给 message 变量。

如果你想在字符串中包含反引号，只需使用反斜杠（\）转义即可，如下所示：

```js
let message = `\`Hello\` world!`;

console.log(message);               // "`Hello` world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 14
```

模板字面量中无需转义单双引号。

<br />

#### 多行字符串（Multiline Strings）

JavaScript 开发者自从该语言诞生起就一直想要一种能创建多行字符串的方法。但是使用单或双引号时，整个字符串只能放在一行。

<br />

##### ECMAScript 6 之前的解决方案（Pre-ECMAScript 6 Workarounds）


感谢一个长久存在的 bug，JavaScript 的确有一种解决方案。你可以在新行之前放置反斜杠（\）来创建多行字符串。如下所示：

```js
var message = "Multiline \
string";

console.log(message);       // "Multiline string"
```

输出的 message 字符串不包含新行，因为反斜杠被视作续延（continuation）信号。为了在输出中包含新行，你需要手动添加它：

```js
var message = "Multiline \n\
string";

console.log(message);       // "Multiline
                            //  string"
```

在所有主流的 JavaScript 引擎中这应该会输出两行，但是该行为被定义为一个 bug 而且许多开发者都建议避免使用这种形式。

其它的解决方案一般依赖于数组或字符串拼接，例如：

```js
var message = [
    "Multiline ",
    "string"
].join("\n");

let message = "Multiline \n" +
    "string";
```

针对 JavaScript 缺乏的多行字符串特性，开发者给出的所有解决方案都存在某些瑕疵。

<br />

##### 多行字符串的简单使用方式（Multiline Strings the Easy Way)


ECMAScript 6 的模板字面量使多行字符串的创建更容易，因为它不需要特殊的语法。只需在想要的位置包含新行即可，而且输出结果也会包含它。例如：

```js
let message = `Multiline
string`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

反引号中的所有空白符都是字符串的一部分，使用缩进要小心。例如：

```js
let message = `Multiline
               string`;

console.log(message);           // "Multiline
                                //                 string"
console.log(message.length);    // 31
```

该段代码中，模板字面量第二行之前的空白符被视作字符串本身的一部分。如果将文本分行缩进对你来讲很重要，请考虑将多行模板字面量的第一行空置并在第二行开始缩进，如下所示：

```js
let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
```

该段代码在第一行开始创建模板字面量但是在第二行之前并没有包含任何字符。HTML 标签的缩进增强了可读性，之后 trim() 方法的调用移除了起始空白行。

<br />

> 如果你愿意的话，也可以在模板字面量中使用 \n 来指示新行的插入位置：

```js
let message = `Multiline\nstring`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

<br />

#### 字符串置换（Making Substitutions）


在这里，模板字面量看上去像是普通 JavaScript 字符串的升级版。两者之间的真正区别在于前者包含的置换操作。置换允许你将 JavaScript 表达式嵌入到模板字面量中并将其结果作为输出字符串中的一部分。

置换部分由 ${ 和 } 包含，其中可以放入任意 JavaScript 表达式。最简单的置换是将本地变量直接嵌入到结果字符串中，例如：

```js
let name = "Nicholas",
    message = `Hello, ${name}.`;

console.log(message);       // "Hello, Nicholas."
```

置换 ${name} 会访问本地变量 name 并将其值插入到 message 字符串中。message 变量会立即持有置换结果。

<br />

> 模板字面量可以访问作用域中定义的任何变量。若变量未定义，在严格和非严格模式下都会抛出错误。

<br />

既然置换的对象都是 JavaScript 表达式，那么可以置换的不仅仅是简单的变量名。你可以很容易地嵌入任意运算，函数调用，等等。例如：

```js
let count = 10,
    price = 0.25,
    message = `${count} items cost $ ${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

该段代码中模板字面量的一部分执行了一次计算。count 和 price 变量进行相乘并使用 .toFixed() 方法将结果的精度格式化为百分位。第二处置换位置之前的美元符号照常输出，因为没有花括号紧随其后。

模板字面量本身也是 JavaScript 表达式，意味着你可以将模板字面量放入到另一个模板字面量内部，如下所示：

```js
let name = "Nicholas",
    message = `Hello, ${
        `my name is ${ name }`
    }.`;

console.log(message);        // "Hello, my name is Nicholas."
```

该例将第二个模板字面量嵌入到第一个内。在首处 ${ 之后使用了另一个模板字面量。第二处 ${ 表示将要嵌入到内层模板字面量的表达式，即 name 变量。

<br />

#### 模板标签（Tagged Templates）


现在你已见识过模板字面量如何创建多行字符串，以及它不需要连接（concatenation）即可将值插入到字符串中。不过模板字面量真正的强大之处来源于模板标签。一个模板标签可以被转换为模板字面量并作为最终值返回。标签在模板的头部，即左 ` 字符之前指定，如下所示：

```js
let message = tag`Hello world`;
```

本例中，tag 即模板标签，并可被转换为 \`Hello world\` 模板字面量。

<br />

##### 定义标签（Defining Tags）


一个标签仅代表一个函数，它接收需要处理的模板字面量。标签分别接收模板字面量中的片段，且必须将它们组合以得出结果。函数的首个参数为包含普通 JavaScript 字符串的数组。余下的参数为每次置换的对应值。

标签函数一般使用剩余参数来定义，以便轻松地处理数据。如下：

```js
function tag(literals, ...substitutions) {
    // 返回一个字符串
}
```

为了更好地理解向标签传递的参数，考虑如下的例子：

```js
let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $ ${(count * price).toFixed(2)}.`;

    // 注意：gitbook 解析 markdown 语法时存在一个 bug，在这里 $ $ 实际为 $$
```

如果你拥有一个 passthru() 函数，那么它会接收三个参数。首当其冲的是一个 literals 数组，包含如下的元素：

* 在首次置换位置之前的空字符串（""）。
* 首次置换位置到第二次置换位置之前的字符串（" items cost $"）。
* 第二次置换位置之后的字符串（"."）。

下个参数为 10，它刚好为 count 变量的值，同时也是 substitutions 数组的首个元素。最后的参数为 "2.50"，即 (count * price).toFixed(2) 的计算结果，并作为 substitutions 数组的第二个元素。

注意 literals 的首个元素为空字符串，以保证 literals[0] 总是代表字符串的起始位置，正如 literals[literals.length - 1] 涵盖字符串的末尾。同时置换（substitution）元素数目也总是比字面量（literal）元素少 1，意味着表达式 substitutions.length === literals.length - 1 的值总是为 true 。

使用这种模式可以将 literals 与 substitutions 数组中的元素相互交织以创建一个结果字符串。literals 中的首个元素起头，substitutions 中的首个元素跟上，以此行动，直到结果字符串被创建完毕。你可以像下例这样交织使用两个数组中的值来模仿模板字面量的默认行为：

```js
function passthru(literals, ...substitutions) {
    let result = "";

    // 只根据 substitution 的数目来运行循环
    for (let i = 0; i < substitutions.length; i++) {
        result += literals[i];
        result += substitutions[i];
    }

    // 添加最后一个 literal
    result += literals[literals.length - 1];

    return result;
}

let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $ ${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

该例定义了 passthru 标签并具有模板字面量的默认行为。唯一的技巧是在循环中使用 substituions.length 而不是 literals.length 来避免 substituions 数组的越界。它的生效归功于 ECMAScript 6 对 literals 和 substituions 的良好定义。

<br />

> substituions 中包含的值不必是字符串。如上例所示，若表达式计算后的值为数字，那么该数值也会被传入。决定这些值的输出方式才是标签的本职工作

<br />

##### 在模板字面量中使用原始值（Using Raw Values in Template Literals）


模板标签也可以访问字符串的原始信息，主要是它可以在转义字符生效前访问它，而最简单的方式是使用内置的 String.raw() 标签。如下所示：

```js
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1);          // "Multiline
                                //  string"
console.log(message2);          // "Multiline\\nstring"
```

该段代码中，message1 中的 \n 被解释为新行，而 message2 返回了 \n 的原始形式 "\\n"（斜杠与字符 n）。类似于该种提取字符串原始信息的行为可以在必要时做更复杂的处理。

字符串的原始信息同样会被传递给模板标签。标签函数中的首个参数为包含额外属性 raw 的数组。raw 属性是含有每个字面量值的对应原始值的数组。例如，literals[0] 总是等同于它的原始值 literals.raw[0]。了解这些之后，你可以使用如下的代码来模仿默认的 String.raw()：

```js
function raw(literals, ...substitutions) {
    let result = "";

    // 只根据 substitution 的数目来运行循环
    for (let i = 0; i < substitutions.length; i++) {
        result += literals.raw[i];      // use raw values instead
        result += substitutions[i];
    }

    // 添加最后一个 literal
    result += literals.raw[literals.length - 1];

    return result;
}

let message = raw`Multiline\nstring`;

console.log(message);           // "Multiline\\nstring"
console.log(message.length);    // 17
```

这里并非使用 literals 而是 literals.raw 来输出结果字符串。这意味着包括 Unicode 代码点在内的任何转义字符都会以原始的形式返回。当你想在输出的字符串中包含转义字符时原始字符串非常好用（例如，如果你想要生成包含代码的文档，那么你期待的是输出实际代码而不是产生的效果）。

<br />

### <a id="Summary"> 总结（Summary） </a>


完整的 Unicode 支持允许 JavaScript 以合理的方式处理 UTF-16 字符。codePointAt() 和 String.fromCodePoint() 拥有的在代码点和字符之间的转换能力是字符串操作的一项重大进步。正则表达式新引入的 u 标志使得直接操作代码点而不是 16 位字符串变为可能，同时 normalize() 方法使得字符串之间的比较结果更为准确。

ECMAScript 6 也提供了操作字符串的新方法，允许你更容易地确认子字符串而无需获取它在整个字符串中的位置。正则表达式也引入了许多功能。

模板字面量是 ECMAScript 6 添加的一项重要内容，允许你创建领域特定语言（domain-specific languages, DSLs）以便让字符串的创建更加轻松。将变量直接嵌入到模板字面量中意味着开发者有一种比字符串拼接更为安全的方式来组合长字符串和变量。

相比传统字符串，模板字面量内置的多行字符串支持是一项实用的改进，这也是前者永远也无法做到的。尽管在模板字面量中允许多行的存在，你依旧可以使用 \n 或其它字符转义序列。

模板标签是创建 DSLs 最重要的部分。标签是接收模板字面量片段为参数的函数。你可以使用参数数据来返回恰当的字符串，其中包括字面量，原生字面量和置换值。标签根据它们来输出相应的结果。

<br />
