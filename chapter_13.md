# 模块（Encapsulating Code With Modules）


JavaScript 采用 “共享一切” 的加载代码方式是该语言最令人迷惑且容易出错的方面之一。其它语言使用包（package）的概念来定义代码的作用范围，然而在 ECMAScript 6 之前，每个 JavaScript 文件中定义的内容都由全局作用域共享。当 web 应用变得复杂并需要书写更多的 JavaScript 代码时，上述加载方式会出现命名冲突或安全方面的问题。ECMAScript 6 的目标之一就是解决作用域的问题并将 JavaScript 应用中的代码整理得更有条理，于是模块应运而生。

<br />

### 
* [什么是模块?](#What-are-Modules)
* [输出的基本概念](#Basic-Exporting)
* [引入的基本概念](#Basic-Importing)
* [export 和 import 的重命名](#Renaming-Exports-and-Imports)
* [模块中的默认值](#Default-Values-in-Modules)
* [绑定的再输出](#Re-exporting-a-Binding)
* [全局引入](#Importing-Without-Bindings)
* [模块加载](#Loading-Modules)
* [总结](#Summary)

<br />

### <a id="What-are-Modules"> 什么是模块？（What are Modules?） </a>


模块是指以不同方式加载的 JavaScript 文件（与 script 这种传统的加载模式相反）。这种方式很有必要，因为它和 script 使用不同的语义：

1. 模块中的代码自动运行在严格模式下，并无任何办法修改为非严格模式。
2. 模块中的顶级（top level）变量不会被添加到全局作用域中。它们只存在于各自的模块中的顶级作用域。
3. 模块顶级作用域中的 this 为 undefined 。
4. 模块不允许存在 HTML 式的注释（JavaScript 历史悠久的遗留特性）。
5. 模块必须输出可被模块外部代码使用的相关内容。
6. 模块可能会引入其它模块中的绑定。

这些差异刚开始看上去觉得并不是很大，不过它们体现了 JavaScript 关于加载和计算代码的显著变更，本章随后我会解释它们。模块真正的好处在于可以输出和引入需要的绑定，而不是文件中的所有内容。理解输出和引入是领悟模块与 script 之间差异的基础。

<br />

### <a id="Basic-Exporting"> 引入的基本概念（Basic Exporting） </a>


你可以使用 export 关键字来对外暴露模块中的部分代码。一般情况下，你可以在任何变量，函数或类声明之前添加这个关键字来输出它们，像这样：

```
// 输出变量
export var color = "red";
export let name = "Nicholas";
export const magicNumber = 7;

// 输出函数
export function sum(num1, num2) {
    return num1 + num1;
}

// 输出类
export class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }
}

// 该函数是模块私有的
function subtract(num1, num2) {
    return num1 - num2;
}

// 定义一个函数...
function multiply(num1, num2) {
    return num1 * num2;
}

// ...并在之后输出它
export { multiply };
```

该例中需要注意一些要点。首先，除 export 关键字之外，所有的声明和传统的形式完全一致。每个输出的函数或类都有一个名称；因为名称是必须的。除非你使用了 default 关键字（在 “模块中的默认值” 一节讨论），否则你不能使用该语法来输出匿名函数或类。

其次，multiply() 函数并未在定义的时候被输出。这是因为你不必每次都要输出一个声明：你可以输出一个引用。最后，该例中并未输出 subtract() 函数。该函数在外部是不可见的，因为任何未显式输出的变量，函数或类都是模块私有的。

<br />

### <a id="Basic-Importing"> 引入的基本概念（Basic Importing） </a>


一旦你有了包含输出内容的模块，你可以在另一个模块内使用 import 关键字来获取它的相关功能。import 语句包含两部分内容，分别为引入的标识符和输出这些标识符的模块。以下是该语句的基本形式：

```
import { identifier1, identifier2 } from "./example.js";
```

import 之后的花括号表示从模块中引入的绑定。from 关键字表示从哪个模块引入这些绑定。模块由一个包含模块路径的字符串表示（称为模块指示符，module sepcifier）。浏览器中的 `<script>` 元素也使用了这个路径形式，意味着它必须包含文件扩展名。另一方面，Node.js 使用自身定义的方式来区分本地文件和包。例如，`example` 会被认为是包而 `./example.js` 被认为是本地文件。

<br />

> import 的绑定序列看起来和解构对象相似，不过它们并无关系。

<br />

当从模块引入了一个绑定时，该绑定的行为类似于 const 。这意味着你不能再次定义一个同名的变量（包括再次引入同名绑定），或在 import 语句之前使用这个标识符，更改它的值也是不被允许的。

<br />

#### 引入单个绑定（Importing a Single Binding）

假设 “输出的基本概念” 一节中的首个示例中的代码包含在一个命名为 example.js 的模块中。你可以使用多种方式来引入并使用这个模块中的绑定。例如，只引入一个标识符：

```
// 只引入单个标识符
import { sum } from "./example.js";

console.log(sum(1, 2));     // 3

sum = 1;        // 错误
```

虽然 example.js 并非仅仅输出了这一个函数，但是该例只引入了它。如果你尝试给 sum 赋一个新值，由于它是不被允许的，所以会发生错误。

<br />

> **注意**：确保文件的路径开头包含 /，./ 或 ../ 以保证浏览器和 Node.js 之间的最佳兼容

<br />

#### 引入多个绑定（Importing Multiple Bindings）


如果你想从 example 模块引入多个绑定，你可以像下面这样显式的列出它们：

```
// 引入多个绑定
import { sum, multiply, magicNumber } from "./example.js";
console.log(sum(1, magicNumber));   // 8
console.log(multiply(1, 2));        // 2
```

该例引入了 example 模块中的三个绑定：sum，multiply 和 magicNumber。之后它们以类似于本地定义的方式使用。

<br />

#### 引入所有绑定（Importing All of a Module）


有一种特殊的情况允许你将整个模块视为单个对象引入。所有的输出可以以对象属性的方式访问。例如：

```
// 输出所有
import * as example from "./example.js";
console.log(example.sum(1, example.magicNumber));          // 8
console.log(example.multiply(1, 2));    // 2
```

在这段代码中，example.js 中的所有绑定被加载到 example 对象中。已命名的输出（sum() 函数，multiple() 函数和 magicNumber）可以以 example 属性的方式访问。这种形式被称为命名空间引入，因为 example 对象在 example.js 文件中并不存在。该对象作为命名空间包含 example.js 中的所有输出。

需要留意的是，不管你针对相同的模块使用了多少次 import 语句，该模块只会被执行一次。当 import 语句执行后，实例化的模块会驻留在内存中并随时可由另一个 import 语句引用。考虑如下的例子：

```
import { sum } from "./example.js";
import { multiply } from "./example.js";
import { magicNumber } from "./example.js";
```

虽然这里针对相同的模块使用了三个 import 语句，example.js 只会被执行一次。如果应用中的其它模块也从 example.js 中引入了绑定，那么它们会使用相同的本段代码中的模块实例。

<br />

> #### 模块语法的限制（Module Syntax Limitations）

> export 和 import 一个很重要的限制是它们必须在语句和函数的外部使用。例如，下面的代码会抛出语法错误：

```
if (flag) {
    export flag;    // 语法错误
}
```

> export 语句被 if 语句包含是不被允许的。export 不能带有条件或者动态地使用。这种限制的一个理由是 JavaScript 引擎可以静态决定输出的内容。因此，你只能在模块的顶级作用域下使用 export 。

> 相似的是，import 也只能在顶级作用域下而不是语句内部使用，意味着以下的代码也会抛出语法错误：

```
function tryImport() {
    import flag from "./example.js";    // 语法错误
}
```

> 基于和 export 相同的理由，你也不能动态地使用 import 绑定。export 和 import 关键字是静态的，所以文本编辑器可以很容易的获取模块中可用内容的信息。

<br />

#### import 绑定中微妙的怪异之处（A Subtle Quirk of Imported Bindings）


ECMAScript 6 中的 import 语句创建了只读的变量，函数和类而不仅仅只是普通的针对源绑定的引用。虽然从模块中引入的绑定不能对值进行更改，但是输出标识符的模块有这个权利。假如你想使用下面的模块：

```
export var name = "Nicholas";
export function setName(newName) {
    name = newName;
}
```

当你引入了这两个绑定后，setName() 函数可以改变 name 的值：

```
import { name, setName } from "./example.js";

console.log(name);       // "Nicholas"
setName("Greg");
console.log(name);       // "Greg"

name = "Nicholas";       // 错误
```

调用 setName("Greg") 会回溯到输出 setName() 的模块本身并在那里执行，将 name 更改为 "Greg"。注意这个变化会自动反射到引入的 name 绑定上，因为它是模块输出的 name 标识符的本地命名。以上代码中使用的 name 和模块输出的 name 并不相同。

<br />

### <a id="Renaming-Exports-and-Imports"> export 和 import 的重命名（Renaming Exports and Imports） </a>


有时候，你不想使用模块中输出的变量，函数和类的原始命名。幸运的是，不论是输出还是引入你都可以对命名进行更改。

首先是 export，假设你想给输出的函数起一个别名。你可以使用 as 关键字来指定它在模块外部可被使用的名称：

```
function sum(num1, num2) {
    return num1 + num2;
}

export { sum as add };
```

在这里，sum() 函数（sum 是本地命名）作为 add() 函数（add 是输出命名）输出。这意味着如果另一个模块想要引入这个函数，就必须使用 add：

```
import { add } from "./example.js";
```

如果引入该函数的模块也想使用不同的命名，可以这样：

```
import { add as sum } from "./example.js";
console.log(typeof add);            // "undefined"
console.log(sum(1, 2));             // 3
```

该段代码引入了 add() 函数并将它重命名为 sum()（本地变量）。这意味着 add 标识符没有添加到该模块内。 

<br />

### <a id="Default-Values-in-Modules"> 模块中的默认值（Default Values in Modules） </a>


当输出和引用模块的默认值时，模块语法得到了充分地利用，因为这种模式在其它模块系统，如 CommonJS（另一种在浏览器之外运行 JavaScrpit 的模块规范）。模块的默认值是由 default 关键字定义的单个变量，函数或类，而且你只能给模块设定一个默认值。多次使用 default 关键字会抛出语法错误。

<br />

#### 输出默认值（Exporting Default Values）


下面是个使用 default 关键字的简单例子：

```
export default function(num1, num2) {
    return num1 + num2;
}
```

这个模块将一个函数作为默认值并输出了它。default 关键字表明这里输出了默认值。该函数的命名并不是必须的，因为它就是这个模块化身。

你也可以定义一个标识符并将它放置在 export default 之后来作为该模块的默认值，例如：

```
function sum(num1, num2) {
    return num1 + num2;
}

export default sum;
```

在这里，sum() 函数的定义在前，稍后它作为模块输出的默认值。当输出的默认值需要计算的时候，你或许会使用这个方法。

第三种指定默认输出标识符的方法是使用重命名语法：

```
function sum(num1, num2) {
    return num1 + num2;
}

export { sum as default };
```

在重命名的输出中，default 标识符有着特殊的含义，它表明某个值应该由模块默认输出。由于在 JavaScript 中 default 是个关键字，所以它不能被用作变量，函数或类的名称（不过它可以用作属性名）。所以使用 default 作为重命名的输出是个特例，不过它与非默认输出语法保持了一致性。

<br />

#### 引入默认值（Importing Default Values）

你可以使用下面的语法引入模块的默认值：

```
// 引入默认值
import sum from "./example.js";

console.log(sum(1, 2));     // 3
```

该 import 语句引入了 example.js 模块的默认值。注意和引入非默认值不同，这里并没有使用花括号。本地命名 sum 代表模块默认输出的任意函数。这种语法是最简洁的，同时 ECMAScript 6 的缔造者们也期待它称为 web 上最常用的引入方式，因为它允许你使用已存在的对象。

对于同时使用了默认输出和非默认输出语法的模块，你可以在一个语句中同时引入它们。例如，如果你有以下的模块：

```
export let color = "red";

export default function(num1, num2) {
    return num1 + num2;
}
```

你可以使用下面的 import 语句同时引入 color 和 默认输出的函数：

```
import sum, { color } from "./example.js";

console.log(sum(1, 2));     // 3
console.log(color);         // "red"
```

逗号分割了引入的默认和非默认的本地命名，后者仍旧使用了花括号。需要牢记的在同一个 import 语句中，引入的默认值必须在非默认值之前。

和输出相似的是，你也可以在引入默认值的同时对其进行重命名：

```
// 和上例等效
import { default as sum, color } from "example";

console.log(sum(1, 2));     // 3
console.log(color);         // "red"
```

该段代码中，默认的输出（default）被重命名为 sum 并和 color 同时被引入。它和上个示例是等效的。

<br />

### <a id="Re-exporting-a-Binding"> 绑定的再输出（Re-exporting a Binding） </a>

有时你会想重新输出一些引入的模块（例如，你创建了包含一些小模块的库）。你可以使用本章中已经讨论过的方式来重新输出它们：

```
import { sum } from "./example.js";
export { sum }
```

这么做没有问题，不过单个语句也能完成相同的任务：

```
export { sum } from "./example.js";
```

该种形式会从指定的模块中查找 sum 声明并输出它。当然，你也可以对输出重新命名：

```
export { sum as add } from "./example.js";
```

在这里，sum 从 "./example.js" 引入并以 add 作为输出。

If you’d like to export everything from another module, you can use the * pattern:

如果你想输出另一个模块中的全部内容，那么你可以使用 * ：

```
export * from "./example.js";
```

输出的全部内容包括默认值（default）和已命名的输出，这会影响当前模块可输出的内容。如果 example.js 包含默认的输出值，你就不能再使用 export default 语法来定义当前模块的默认值。

<br />

### <a id="Importing-Without-Bindings"> 全局引入（Importing Without Bindings） </a>

一些模块可能并不输出任何内容，相反，他们只是修改全局作用域内的对象。虽然模块内部的顶级变量，函数和类并不会自动添加到全局作用域，但这不代表它们不能访问全局作用域。共享的内置对象如 Array 和 Object 在模块内部是可供访问的，而且对它们的修改还会影响到其它模块。

例如，如果你想给数组添加一个 pushAll() 方法，你可能会像下面这样定义一个模块：

```
// 没有输出和引入的模块
Array.prototype.pushAll = function(items) {

    // items 必须是数组
    if (!Array.isArray(items)) {
        throw new TypeError("Argument must be an array.");
    }

    // 使用内置的 push() 和扩展运算符
    return this.push(...items);
};
```

虽然没有输出和引入，该模块仍然是合法的。这段代码可以同时被当作模块和 scrpit 使用。既然没有任何输出内容，你可以使用简单的不需要任何绑定的引入语法来执行它们：

```
import "./example.js";

let colors = ["red", "green", "blue"];
let items = [];

items.pushAll(colors);
```

这段代码引入并执行了模块中包含的 pushAll() 方法，所以 pushAll() 被添加给数组的原型。现在这意味着 pushAll() 可以作用于当前模块内的所有数组。

<br />

> 全局引入一般被用来创建 polyfill 和 shim 。

<br />

### <a id="Loading-Modules"> 模块加载（Loading Modules） </a>

ECMAScript 6 只定义了模块的语法而未说明如何加载它们。后者如果纳入规范会比较复杂，它应由实现环境自行决定。ECMAScript 6 只对一个未详细说明并被称为 HostResolveImportedModule 的内部操作定义了加载机制的语法和相关抽象，并没有规范所有的 JavaScript 环境。具体实现由浏览器或 Node.js 根据各自环境的情况来自行决定。

<br />

#### 在浏览器中使用模块（Using Modules in Web Browsers）


在 ECMAScript 6 出现之前，浏览器中的 web 应用拥有多种加载 JavaScript 的方式。如下所示：

1. 通过使用 `<script>` 元素和 src 属性来决定需要加载代码的位置。
2. 不使用 src 属性，在 `<script>` 元素中使用内联 JavaScript 代码。
3. 加载 JavaScript 代码到 worker 环境中。

In order to fully support modules, web browsers had to update each of these mechanisms. These details are defined in the HTML specification, and I’ll summarize them in this section.

为了充分支持模块，浏览器需要改进以上机制。它们的相关细节由 HTML 规范定义，本章中我会对其作一些总结。

<br />

##### `<script>` 标签（Using Modules With `<script>`）


`script` 元素默认以 script 方式来加载 JavaScrpit 文件（不是模块）。同样在 type 属性缺失或值与 JavaScript content type 相关时（如 "text/javascript"）也是如此。`script` 元素可以执行内联或 src 定义文件中的代码。为了支持模块，type 的可选值中添加了 "module"。将 type 设定为 module 会通知浏览器将相关内联或文件中的代码视为模块而不是 script。这里有个简单示例：

```
<!-- 加载一个 JavaScript 模块文件 -->
<script type="module" src="module.js"></script>

<!-- 包含一个模块内联代码 -->
<script type="module">

import { sum } from "./example.js";

let result = sum(1, 2);

</script>
```

本例中的首个 `<script>` 元素使用 src 属性来加载一个外部模块文件。它与加载一个 script 的唯一区别是 type 的值为 "module" 。第二个 `<script>` 元素包含直接嵌入到网页中的模块。result 变量不会暴露给全局，它只存在于这个模块内部（即 `<script>`元素的内部）而不会被添加给 window 并作为其属性。

As you can see, including modules in web pages is fairly simple and similar to including scripts. However, there are some differences in how modules are loaded.

如你所见，在网页中使用模块相当简单而且和 script 十分相似。然而，它们的加载方式有一些区别。

<br />

> 你或许注意到了 "module" 在写法上与 "text/javascript" 这个 content type 不同。不过 JavaScript 模块文件的 content type 被视为与 script 文件一致。而且，当 `script` 元素中的 type 无法识别时，浏览器会忽略它，所以不支持模块的浏览器会自动无视 `<script type="module">`，以保证向下兼容。

<br />

##### 浏览器中的模块加载顺序（Module Loading Sequence in Web Browsers）


模块相比 script 的独特之处在于，它们可能会引入其它文件以保证内部代码正常执行。为了实现该功能，`script type="module"` 总是被视为使用了 defer 属性。

当加载 script 文件时，defer 属性是可选的，然而加载模块文件时 defer 会自动施行。模块文件会在 HTML 解析器遇到 `<script type="module">` 后立即下载，但是它们会在整个文档解析完毕之后执行。模块的执行顺序完全取决于它们在 HTML 文件中的位置。这意味着首个 `<script type="module">` 一定会最先执行，即使后面存在不使用 src 属性的内联模块。例如：

```
<!-- 最先执行 -->
<script type="module" src="module1.js"></script>

<!-- 第二个执行 -->
<script type="module">
import { sum } from "./example.js";

let result = sum(1, 2);
</script>

<!-- 第三个执行 -->
<script type="module" src="module2.js"></script>
```

这三个 `<script>` 元素会按照定义的顺序执行，所以 module1.js 保证会在内联模块之前运行，同理内联代码也会在 module2.js 之前运行。

每个模块都有可能引入了一个或多个其它模块，这使得情况变得更为复杂。因此模块首先会被完整的解析已确认所有 import 语句的存在。每条 import 语句又会触发一次 fetch（不论是通过网络还是在缓存中获得），并且在所有引入的资源被加载和执行之前，模块中内容不会运行。

不论是通过显式的 `<script type="module">` 还是隐式的 `import` 引入的模块，它们都会按照顺序执行。在上个示例中，加载的顺序是这样的：

1. 下载并解析 module1.js 。
2. 递归下载并解析 module1.js 中引入的资源。
3. Parse the inline module.解析内联模块。
4. 递归下载并解析内联模块中引入的资源。
5. 下载并解析 module2.js 。
6. 递归下载并解析 module2.js 中引入的资源。

模块一旦加载完毕，直到文档被完整解析之前不会有任何代码执行。在文档完全解析后，以下情况会发生：

1. 递归执行 module1.js 中引入的资源。
2. 执行 module1.js 。
3. 递归执行内联模块中引入的资源。
4. 执行内联模块。
5. 递归执行 module2.js 中引入的资源
6. 执行 module2.js

注意内联模块和其它两个模块相比，除了不必事先下载之外，引入资源的加载顺序和模块的执行顺序是完全相同的。

<br />

> 向 `<script type="module">` 添加 defer 属性会被忽略，因为它已经实行了该属性。

<br />

##### 浏览器中异步模块的加载顺序（Asynchronous Module Loading in Web Browsers）


或许你对 `script` 元素的 async 属性十分熟悉。当使用 script 时，async 会让 script 在下载和解析之后立即执行。文档中 async script 出现的顺序并不会会影响相互之间的执行顺序。它们在下载之后会立即执行而不需要等待文档解析完毕。

async 属性也可以用于模块。对 `<script type="module">` 添加 async 属性使得它们的行为与包含 async 属性的 script 类似。唯一的区别在于模块中引入的资源会在模块本身执行之前下载，以保证模块需要的函数会事先获得；但是模块的执行顺序是不能确定。考虑如下的代码：

```
<!-- 无法确定哪个模块会首先运行 -->
<script type="module" async src="module1.js"></script>
<script type="module" async src="module2.js"></script>
```

该例中异步加载了两个模块。仅仅观察模块中的代码无法确定模块的执行顺序。如果 module1.js 首先下载完毕（包括它引入的资源），那么它会首先执行。module2.js 也是同理。

<br />

##### 在 worker 中加载模块（Loading Modules as Workers）


worker，包括 web worker 和 service worker，会在网页之外的上下文中执行 JavaScript 代码。创建一个新的 worker 需要 worker 实例（或类）并传递 JavaScript 文件的位置。worker 默认的加载机制是将文件视为 script，像这样：

```
// 以 script 的方式加载 script.js
let worker = new Worker("script.js");
```

为了支持模块的加载，负责 HTML 标准的开发者们给构造函数添加了第二个参数。这个参数是包含 type 属性并且默认值为 script 的对象。你可以将 type 设定为 "module" 来加载模块。

```
// 以模块的方式加载 module.js
let worker = new Worker("module.js", { type: "module" });
```

This example loads module.js as a module instead of a script by passing a second argument with "module" as the type property’s value. (The type property is meant to mimic how the type attribute of `<script>` differentiates modules and scripts.) The second argument is supported for all worker types in the browser.

该例将第二个参数中的 type 属性值设定为 "module" 

Worker modules are generally the same as worker scripts, but there are a couple of exceptions. First, worker scripts are limited to being loaded from the same origin as the web page in which they are referenced, but worker modules aren’t quite as limited. Although worker modules have the same default restriction, they can also load files that have appropriate Cross-Origin Resource Sharing (CORS) headers to allow access. Second, while a worker script can use the self.importScripts() method to load additional scripts into the worker, self.importScripts() always fails on worker modules because you should use import instead.

<br />

##### Browser Module Specifier Resolution

All of the examples to this point in the chapter have used a relative module specifier path such as "./example.js". Browsers require module specifiers to be in one of the following formats:

* Begin with / to resolve from the root directory
* Begin with ./ to resolve from the current directory
* Begin with ../ to resolve from the parent directory
* URL format


For example, suppose you have a module file located at `https://www.example.com/modules/module.js` that contains the following code:

```
// imports from https://www.example.com/modules/example1.js
import { first } from "./example1.js";

// imports from https://www.example.com/example2.js
import { second } from "../example2.js";

// imports from https://www.example.com/example3.js
import { third } from "/example3.js";

// imports from https://www2.example.com/example4.js
import { fourth } from "https://www2.example.com/example4.js";
```

Each of the module specifiers in this example is valid for use in a browser, including the complete URL in the final line (you’d need to be sure ww2.example.com has properly configured its Cross-Origin Resource Sharing (CORS) headers to allow cross-domain loading). These are the only module specifier formats that browsers can resolve by default (though the not-yet-complete module loader specification will provide ways to resolve other formats). That means some normal looking module specifiers are actually invalid in browsers and will result in an error, such as:

```
// invalid - doesn't begin with /, ./, or ../
import { first } from "example.js";

// invalid - doesn't begin with /, ./, or ../
import { second } from "example/index.js";
```

Each of these module specifiers cannot be loaded by the browser. The two module specifiers are in an invalid format (missing the correct beginning characters) even though both will work when used as the value of src in a `<script>` tag. This is an intentional difference in behavior between `<script>` and import.

<br />

### Summary

ECMAScript 6 adds modules to the language as a way to package up and encapsulate functionality. Modules behave differently than scripts, as they don’t modify the global scope with their top-level variables, functions, and classes, and this is undefined. To achieve that behavior, modules are loaded using a different mode.

You must export any functionality you’d like to make available to consumers of a module. Variables, functions, and classes can all be exported, and there is also one default export allowed per module. After exporting, another module can import all or some of the exported names. These names act as if defined by let and operate as block bindings that can’t be redeclared in the same module.

Modules need not export anything if they are manipulating something in the global scope. You can actually import from such a module without introducing any bindings into the module scope.

Because modules must run in a different mode, browsers introduced `<script type="module">` to signal that the source file or inline code should be executed as a module. Module files loaded with `<script type="module">` are loaded as if the defer attribute is applied to them. Modules are also executed in the order in which they appear in the containing document once the document is fully parsed.

<br />