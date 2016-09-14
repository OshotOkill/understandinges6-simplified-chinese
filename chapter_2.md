# 字符串与正则表达式（Strings and Regular Expressions）


字符串可以说是程序设计中最为重要的数据类型之一。几乎每种高级编程语言都有它的一席之地，而且能有效的使用它也是开发者编写实用程序的基本准则。作为重要的扩展，正则表达式赋予开发者操作字符串的额外能力。ECMAScript 6 的缔造者们将这些事实牢记于心，改进了字符串和正则表达式，并添加了长久以来缺失的某些功能。本章会讲解它们的变化之处。

<br />
### 本章小结

* [更佳的 Unicode 支持](#Better-Unicode-Support)
* [字符串的其它改进](#Other-String-Changes)
* [正则表达式的其它改进](#Other-Regular-Expression-Changes)
* [模板字符串](#Template-Literals)
* [总结](#Summary)

<br />

### <a id ="Better-Unicode-Support"> 更佳的 Unicode 支持（Better Unicode Support） </a>


ECMAScript 6 诞生之前，JavaScript 字符串（string）由 16 位编码的字符组成（UTF-16）。每个字符又由包含一个 16 位序列的代码单元（code unit）表示。所有的字符串属性和方法，例如 length 和 charAt()，都基于这些 16 位编码单元。曾经，16 位的容量对于任意字符的存放都是足够的，然而 Unicode 引入了扩展字符集（expanded character set）使得

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

The single Unicode character "𠮷" is represented using surrogate pairs, and as such, the JavaScript string operations above treat the string as having two 16-bit characters. That means:

单个 Unicode 字符 "𠮷" 由代理项对表示，因此，本例中 JavaScript 在操作该字符串时会将它视为两个 16 位字符。这意味着：

* text 的长度为 2，即使它看起来是 1 。
* 试图匹配单个字符的正则表达式会以失败告终，因为表达式将其视作两个字符。
* charAt() 方法无法返回一个有效的字符串，因为包含的两个 16 位代码单元都无法被打印（printable）。

charCodeAt() 方法也不能正确识别字符。它返回的是对应代码单元的 16 位数字，但是在 ECMAScript 5 中这已经是所能获取到的最精确的文本值了。

另一方面，ECMAScript 6 要求以上 UTF-16 字符的编码问题必须得到解决。标准化基于新字符编码规范的字符串操作意味着 JavaScript 支持专门处理代理项对的功能。本章剩下的部分讨论了使用该功能的几个关键案例。

<br />

#### codePointAt() 方法（The codePointAt() Method）

One method ECMAScript 6 added to fully support UTF-16 is the codePointAt() method, which retrieves the Unicode code point that maps to a given position in a string. This method accepts the code unit position rather than the character position and returns an integer value, as these console.log() examples show:

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


Another interesting aspect of Unicode is that different characters may be considered equivalent for the purpose of sorting or other comparison-based operations. There are two ways to define these relationships. First, canonical equivalence means that two sequences of code points are considered interchangeable in all respects. For example, a combination of two characters can be canonically equivalent to one character. The second relationship is compatibility. Two compatible sequences of code points look different but can be used interchangeably in certain situations.

Unicode 另一个有趣的方面是，不同的字符在某些排序或比较操作中被认为是相同的。有两种方式可以确立两者之间的关系。第一，canonical equivalence 意味着两个代码点序列在各个方面都可以进行互换。例如，两个字符的组合根据 canonically equivalent 可以等同于一个字符。第二，是字符间的兼容性。两个兼容的代码点序列可能看上去不同，实际上在特定条件下可以互换（interchangeably）。

由于这些关系的存在，两个在根本（fundamentally）上相同的字符串可以包含不同的代码点序列。例如，字符 "æ" 和两个字符组成的 "ae" 字符串或许可以互换，但是除非以某种形式进行规范化，否则严格意义上讲它们并不相等。

ECMAScript 6 通过给字符串提供 normalize() 方法来支持 Unicode 规范格式。该方法接收一个字符串参数来获取以下 Unicode 规范格式中的一种并据此运行：

* Normalization Form Canonical Composition ("NFC"), 默认的规范格式
* Normalization Form Canonical Decomposition ("NFD")
* Normalization Form Compatibility Composition ("NFKC")
* Normalization Form Compatibility Decomposition ("NFKD")

解释这四种格式的区别超出了本书的范围。只需记住，当比较字符串时，它们必须被规范成同一种格式。例如：

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


You can accomplish many common string operations through regular expressions. But remember, regular expressions assume 16-bit code units, where each represents a single character. To address this problem, ECMAScript 6 defines a u flag for regular expressions, which stands for Unicode.

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

While these three methods make identifying the existence of substrings easier, each only returns a boolean value. If you need to find the actual position of one string within another, use the indexOf() or lastIndexOf() methods.

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

ECMAScript 6 standardized the y flag after it was implemented in Firefox as a proprietary extension to regular expressions. The y flag affects a regular expression search’s sticky property, and it tells the search to start matching characters in a string at the position specified by the regular expression’s lastIndex property. If there is no match at that location, then the regular expression stops matching. To see how this works, consider the following code:

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

This example has three regular expressions. The expression in pattern has no flags, the one in globalPattern uses the g flag, and the one in stickyPattern uses the y flag. In the first trio of console.log() calls, all three regular expressions should return "hello1 " with a space at the end.

After that, the lastIndex property is changed to 1 on all three patterns, meaning that the regular expression should start matching from the second character on all of them. The regular expression with no flags completely ignores the change to lastIndex and still matches "hello1 " without incident. The regular expression with the g flag goes on to match "hello2 " because it is searching forward from the second character of the string ("e"). The sticky regular expression doesn’t match anything beginning at the second character so stickyResult is null.

The sticky flag saves the index of the next character after the last match in lastIndex whenever an operation is performed. If an operation results in no match, then lastIndex is set back to 0. The global flag behaves the same way, as demonstrated here:

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

The value of lastIndex changes to 7 after the first call to exec() and to 14 after the second call, for both the stickyPattern and globalPattern variables.

There are two more subtle details about the sticky flag to keep in mind:

1. The lastIndex property is only honored when calling methods that exist on the regular expression object, like the exec() and test() methods. Passing the regular expression to a string method, such as match(), will not result in the sticky behavior.
2. When using the ^ character to match the start of a string, sticky regular expressions only match from the start of the string (or the start of the line in multiline mode). While lastIndex is 0, the ^ makes a sticky regular expression no different from a non-sticky one. If lastIndex doesn’t correspond to the beginning of the string in single-line mode or the beginning of a line in multiline mode, the sticky regular expression will never match.

As with other regular expression flags, you can detect the presence of y by using a property. In this case, you’d check the sticky property, as follows:

```js
var pattern = /hello\d/y;

console.log(pattern.sticky);    // true
```

The sticky property is set to true if the sticky flag is present, and the property is false if not. The sticky property is read-only based on the presence of the flag and cannot be changed in code.

Similar to the u flag, the y flag is a syntax change, so it will cause a syntax error in older JavaScript engines. You can use the following approach to detect support:

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

Just like the u check, this returns false if it’s unable to create a regular expression with the y flag. In one final similarity to u, if you need to use y in code that runs in older JavaScript engines, be sure to use the RegExp constructor when defining those regular expressions to avoid a syntax error.

<br />

#### Duplicating Regular Expressions

In ECMAScript 5, you can duplicate regular expressions by passing them into the RegExp constructor like this:

```js
var re1 = /ab/i,
    re2 = new RegExp(re1);
```

The re2 variable is just a copy of the re1 variable. But if you provide the second argument to the RegExp constructor, which specifies the flags for the regular expression, your code won’t work, as in this example:

```js
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");
```

If you execute this code in an ECMAScript 5 environment, you’ll get an error stating that the second argument cannot be used when the first argument is a regular expression. ECMAScript 6 changed this behavior such that the second argument is allowed and overrides any flags present on the first argument. For example:

```js
var re1 = /ab/i,

    // throws an error in ES5, okay in ES6
    re2 = new RegExp(re1, "g");


console.log(re1.toString());            // "/ab/i"
console.log(re2.toString());            // "/ab/g"

console.log(re1.test("ab"));            // true
console.log(re2.test("ab"));            // true

console.log(re1.test("AB"));            // true
console.log(re2.test("AB"));            // false
```

In this code, re1 has the case-insensitive i flag while re2 has only the global g flag. The RegExp constructor duplicated the pattern from re1 and substituted the g flag for the i flag. Without the second argument, re2 would have the same flags as re1.

<br />

#### The flags Property

Along with adding a new flag and changing how you can work with flags, ECMAScript 6 added a property associated with them. In ECMAScript 5, you could get the text of a regular expression by using the source property, but to get the flag string, you’d have to parse the output of the toString() method as shown below:

```js
function getFlags(re) {
    var text = re.toString();
    return text.substring(text.lastIndexOf("/") + 1, text.length);
}

// toString() is "/ab/g"
var re = /ab/g;

console.log(getFlags(re));          // "g"
```

This converts a regular expression into a string and then returns the characters found after the last /. Those characters are the flags.

ECMAScript 6 makes fetching flags easier by adding a flags property to go along with the source property. Both properties are prototype accessor properties with only a getter assigned, making them read-only. The flags property makes inspecting regular expressions easier for both debugging and inheritance purposes.

A late addition to ECMAScript 6, the flags property returns the string representation of any flags applied to a regular expression. For example:

```js
var re = /ab/g;

console.log(re.source);     // "ab"
console.log(re.flags);      // "g"
```

This fetches all flags on re and prints them to the console with far fewer lines of code than the toString() technique can. Using source and flags together allows you to extract the pieces of the regular expression that you need without parsing the regular expression string directly.

The changes to strings and regular expressions that this chapter has covered so far are definitely powerful, but ECMAScript 6 improves your power over strings in a much bigger way. It brings a type of literal to the table that makes strings more flexible.

<br />

### Template Literals

JavaScript’s strings have always had limited functionality compared to strings in other languages. For instance, until ECMAScript 6, strings lacked the methods covered so far in this chapter, and string concatenation is as simple as possible. To allow developers to solve more complex problems, ECMAScript 6’s template literals provide syntax for creating domain-specific languages (DSLs) for working with content in a safer way than the solutions available in ECMAScript 5 and earlier. (A DSL is a programming language designed for a specific, narrow purpose, as opposed to general-purpose languages like JavaScript.) The ECMAScript wiki offers the following description on the [template literal strawman](http://wiki.ecmascript.org/doku.php?id=harmony:quasis):

<br />

> This scheme extends ECMAScript syntax with syntactic sugar to allow libraries to provide DSLs that easily produce, query, and manipulate content from other languages that are immune or resistant to injection attacks such as XSS, SQL Injection, etc.

<br />

In reality, though, template literals are ECMAScript 6’s answer to the following features that JavaScript lacked all the way through ECMAScript 5:

* **Multiline strings** A formal concept of multiline strings.
* **Basic string formatting** The ability to substitute parts of the string for values contained in variables.
* **HTML escaping** The ability to transform a string such that it is safe to insert into HTML.

Rather than trying to add more functionality to JavaScript’s already-existing strings, template literals represent an entirely new approach to solving these problems.

<br />

#### Basic Syntax

At their simplest, template literals act like regular strings delimited by backticks (`) instead of double or single quotes. For example, consider the following:

```js
let message = `Hello world!`;

console.log(message);               // "Hello world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 12
```

This code demonstrates that the variable message contains a normal JavaScript string. The template literal syntax is used to create the string value, which is then assigned to the message variable.
If you want to use a backtick in your string, then just escape it with a backslash (\), as in this version of the message variable:

```js
let message = `\`Hello\` world!`;

console.log(message);               // "`Hello` world!"
console.log(typeof message);        // "string"
console.log(message.length);        // 14
```

There’s no need to escape either double or single quotes inside of template literals.

<br />

#### Multiline Strings

JavaScript developers have wanted a way to create multiline strings since the first version of the language. But when using double or single quotes, strings must be completely contained on a single line.

<br />

##### Pre-ECMAScript 6 Workarounds

Thanks to a long-standing syntax bug, JavaScript does have a workaround. You can create multiline strings if there’s a backslash (\) before a newline. Here’s an example:

```js
var message = "Multiline \
string";

console.log(message);       // "Multiline string"
```

The message string has no newlines present when printed to the console because the backslash is treated as a continuation rather than a newline. In order to show a newline in output, you’d need to manually include it:

```js
var message = "Multiline \n\
string";

console.log(message);       // "Multiline
                            //  string"
```

This should print Multiline String on two separate lines in all major JavaScript engines, but the behavior is defined as a bug and many developers recommend avoiding it.

Other pre-ECMAScript 6 attempts to create multiline strings usually relied on arrays or string concatenation, such as:

```js
var message = [
    "Multiline ",
    "string"
].join("\n");

let message = "Multiline \n" +
    "string";
```

All of the ways developers worked around JavaScript’s lack of multiline strings left something to be desired.

<br />

##### Multiline Strings the Easy Way

ECMAScript 6’s template literals make multiline strings easy because there’s no special syntax. Just include a newline where you want, and it shows up in the result. For example:
let message = `Multiline
string`;

```js
console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

All whitespace inside the backticks is part of the string, so be careful with indentation. For example:

```js
let message = `Multiline
               string`;

console.log(message);           // "Multiline
                                //                 string"
console.log(message.length);    // 31
```

In this code, all whitespace before the second line of the template literal is considered part of the string itself. If making the text line up with proper indentation is important to you, then consider leaving nothing on the first line of a multiline template literal and then indenting after that, as follows:

```js
let html = `
<div>
    <h1>Title</h1>
</div>`.trim();
```

This code begins the template literal on the first line but doesn’t have any text until the second. The HTML tags are indented to look correct and then the trim() method is called to remove the initial empty line.

<br />

> If you prefer, you can also use \n in a template literal to indicate where a newline should be inserted:

```js
let message = `Multiline\nstring`;

console.log(message);           // "Multiline
                                //  string"
console.log(message.length);    // 16
```

<br />

#### Making Substitutions

At this point, template literals may look like fancier versions of normal JavaScript strings. The real difference between the two lies in template literal substitutions. Substitutions allow you to embed any valid JavaScript expression inside a template literal and output the result as part of the string.

Substitutions are delimited by an opening ${ and a closing } that can have any JavaScript expression inside. The simplest substitutions let you embed local variables directly into a resulting string, like this:

```js
let name = "Nicholas",
    message = `Hello, ${name}.`;

console.log(message);       // "Hello, Nicholas."
```

The substitution ${name} accesses the local variable name to insert name into the message string. The message variable then holds the result of the substitution immediately.

<br />

> A template literal can access any variable accessible in the scope in which it is defined. Attempting to use an undeclared variable in a template literal throws an error in both strict and non-strict modes.

<br />

Since all substitutions are JavaScript expressions, you can substitute more than just simple variable names. You can easily embed calculations, function calls, and more. For example:

```js
let count = 10,
    price = 0.25,
    message = `${count} items cost $ ${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

This code performs a calculation as part of the template literal. The variables count and price are multiplied together to get a result, and then formatted to two decimal places using .toFixed(). The dollar sign before the second substitution is output as-is because it’s not followed by an opening curly brace.

Template literals are also JavaScript expressions, which means you can place a template literal inside of another template literal, as in this example:

```js
let name = "Nicholas",
    message = `Hello, ${
        `my name is ${ name }`
    }.`;

console.log(message);        // "Hello, my name is Nicholas."
```

This example nests a second template literal inside the first. After the first ${, another template literal begins. The second ${ indicates the beginning of an embedded expression inside the inner template literal. That expression is the variable name, which is inserted into the result.

<br />

#### Tagged Templates

Now you’ve seen how template literals can create multiline strings and insert values into strings without concatenation. But the real power of template literals comes from tagged templates. A template tag performs a transformation on the template literal and returns the final string value. This tag is specified at the start of the template, just before the first ` character, as shown here:

```js
let message = tag`Hello world`;
```

In this example, tag is the template tag to apply to the \`Hello world\` template literal.

<br />

##### Defining Tags

A tag is simply a function that is called with the processed template literal data. The tag receives data about the template literal as individual pieces and must combine the pieces to create the result. The first argument is an array containing the literal strings as interpreted by JavaScript. Each subsequent argument is the interpreted value of each substitution.

Tag functions are typically defined using rest arguments as follows, to make dealing with the data easier:

```js
function tag(literals, ...substitutions) {
    // return a string
}
```

To better understand what gets passed to tags, consider the following:

```js
let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;
```

If you had a function called passthru(), that function would receive three arguments. First, it would get a literals array, containing the following elements:

* The empty string before the first substitution ("")
* The string after the first substitution and before the second (" items cost $")
* The string after the second substitution (".")

The next argument would be 10, which is the interpreted value for the count variable. This becomes the first element in a substitutions array. The final argument would be "2.50", which is the interpreted value for (count * price).toFixed(2) and the second element in the substitutions array.

Note that the first item in literals is an empty string. This ensures that literals[0] is always the start of the string, just like literals[literals.length - 1] is always the end of the string. There is always one fewer substitution than literal, which means the expression substitutions.length === literals.length - 1 is always true.

Using this pattern, the literals and substitutions arrays can be interwoven to create a resulting string. The first item in literals comes first, the first item in substitutions is next, and so on, until the string is complete. As an example, you can mimic the default behavior of a template literal by alternating values from these two arrays:

```js
function passthru(literals, ...substitutions) {
    let result = "";

    // run the loop only for the substitution count
    for (let i = 0; i < substitutions.length; i++) {
        result += literals[i];
        result += substitutions[i];
    }

    // add the last literal
    result += literals[literals.length - 1];

    return result;
}

let count = 10,
    price = 0.25,
    message = passthru`${count} items cost $${(count * price).toFixed(2)}.`;

console.log(message);       // "10 items cost $2.50."
```

This example defines a passthru tag that performs the same transformation as the default template literal behavior. The only trick is to use substitutions.length for the loop rather than literals.length to avoid accidentally going past the end of the substitutions array. This works because the relationship between literals and substitutions is well-defined in ECMAScript 6.

<br />

> The values contained in substitutions are not necessarily strings. If an expression evaluates to a number, as in the previous example, then the numeric value is passed in. Determining how such values should output in the result is part of the tag’s job.

<br />

##### Using Raw Values in Template Literals

Template tags also have access to raw string information, which primarily means access to character escapes before they are transformed into their character equivalents. The simplest way to work with raw string values is to use the built-in String.raw() tag. For example:

```js
let message1 = `Multiline\nstring`,
    message2 = String.raw`Multiline\nstring`;

console.log(message1);          // "Multiline
                                //  string"
console.log(message2);          // "Multiline\\nstring"
```

In this code, the \n in message1 is interpreted as a newline while the \n in message2 is returned in its raw form of "\\n" (the slash and n characters). Retrieving the raw string information like this allows for more complex processing when necessary.

The raw string information is also passed into template tags. The first argument in a tag function is an array with an extra property called raw. The raw property is an array containing the raw equivalent of each literal value. For example, the value in literals[0] always has an equivalent literals.raw[0] that contains the raw string information. Knowing that, you can mimic String.raw() using the following code:

```js
function raw(literals, ...substitutions) {
    let result = "";

    // run the loop only for the substitution count
    for (let i = 0; i < substitutions.length; i++) {
        result += literals.raw[i];      // use raw values instead
        result += substitutions[i];
    }

    // add the last literal
    result += literals.raw[literals.length - 1];

    return result;
}

let message = raw`Multiline\nstring`;

console.log(message);           // "Multiline\\nstring"
console.log(message.length);    // 17
```

This uses literals.raw instead of literals to output the string result. That means any character escapes, including Unicode code point escapes, should be returned in their raw form. Raw strings are helpful when you want to output a string containing code in which you’ll need to include the character escaping (for instance, if you want to generate documentation about some code, you may want to output the actual code as it appears).

<br />

### <a id="Summary"> Summary </a>

Full Unicode support allows JavaScript to deal with UTF-16 characters in logical ways. The ability to transfer between code point and character via codePointAt() and String.fromCodePoint() is an important step for string manipulation. The addition of the regular expression u flag makes it possible to operate on code points instead of 16-bit characters, and the normalize() method allows for more appropriate string comparisons.

ECMAScript 6 also added new methods for working with strings, allowing you to more easily identify a substring regardless of its position in the parent string. More functionality was added to regular expressions, too.

Template literals are an important addition to ECMAScript 6 that allows you to create domain-specific languages (DSLs) to make creating strings easier. The ability to embed variables directly into template literals means that developers have a safer tool than string concatenation for composing long strings with variables.

Built-in support for multiline strings also makes template literals a useful upgrade over normal JavaScript strings, which have never had this ability. Despite allowing newlines directly inside the template literal, you can still use \n and other character escape sequences.

Template tags are the most important part of this feature for creating DSLs. Tags are functions that receive the pieces of the template literal as arguments. You can then use that data to return an appropriate string value. The data provided includes literals, their raw equivalents, and any substitution values. These pieces of information can then be used to determine the correct output for the tag.

