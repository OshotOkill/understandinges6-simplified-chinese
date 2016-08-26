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
* [Importing Without Bindings](#Importing-Without-Bindings)
* [Loading Modules](#Loading-Modules)
* [Summary](#Summary)

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

> An important limitation of both export and import is that they must be used outside other statements and functions. For instance, this code will give a syntax error: export 和 import 一个很重要的限制是它们必须在语句和函数的外部使用。例如，下面的代码会抛出语法错误：

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

There may be a time when you’d like to re-export something that your module has imported (for instance, if you’re creating a library out of several small modules). You can re-export an imported value with the patterns already discussed in this chapter as follows:

```
import { sum } from "./example.js";
export { sum }
```

That works, but a single statement can also do the same thing:

```
export { sum } from "./example.js";
```

This form of export looks into the specified module for the declaration of sum and then exports it. Of course, you can also choose to export a different name for the same value:

```
export { sum as add } from "./example.js";
```

Here, sum is imported from "./example.js" and then exported as add.

If you’d like to export everything from another module, you can use the * pattern:

```
export * from "./example.js";
```

By exporting everything, you’re including the default as well as any named exports, which may affect what you can export from your module. For instance, if example.js has a default export, you’d be unable to define a new default export when using this syntax.

<br />

### Importing Without Bindings

Some modules may not export anything, and instead, only make modifications to objects in the global scope. Even though top-level variables, functions, and classes inside modules don’t automatically end up in the global scope, that doesn’t mean modules cannot access the global scope. The shared definitions of built-in objects such as Array and Object are accessible inside a module and changes to those objects will be reflected in other modules.

For instance, if you want to add a pushAll() method to all arrays, you might define a module like this:

```
// module code without exports or imports
Array.prototype.pushAll = function(items) {

    // items must be an array
    if (!Array.isArray(items)) {
        throw new TypeError("Argument must be an array.");
    }

    // use built-in push() and spread operator
    return this.push(...items);
};
```

This is a valid module even though there are no exports or imports. This code can be used both as a module and a script. Since it doesn’t export anything, you can use a simplified import to execute the module code without importing any bindings:

```
import "./example.js";

let colors = ["red", "green", "blue"];
let items = [];

items.pushAll(colors);
```

This code imports and executes the module containing the pushAll() method, so pushAll() is added to the array prototype. That means pushAll() is now available for use on all arrays inside of this module.

> Imports without bindings are most likely to be used to create polyfills and shims.

<br />

### Loading Modules

While ECMAScript 6 defines the syntax for modules, it doesn’t define how to load them. This is part of the complexity of a specification that’s supposed to be agnostic to implementation environments. Rather than trying to create a single specification that would work for all JavaScript environments, ECMAScript 6 specifies only the syntax and abstracts out the loading mechanism to an undefined internal operation called HostResolveImportedModule. Web browsers and Node.js are left to decide how to implement HostResolveImportedModule in a way that makes sense for their respective environments.

<br />

#### Using Modules in Web Browsers

Even before ECMAScript 6, web browsers had multiple ways of including JavaScript in an web application. Those script loading options are:

1. Loading JavaScript code files using the `<script>` element with the src attribute specifying a location from which to load the code.
2. Embedding JavaScript code inline using the `<script>` element without the src attribute.
Loading JavaScript code files to execute as workers (such as a web worker or service worker).
3. In order to fully support modules, web browsers had to update each of these mechanisms. These details are defined in the HTML specification, and I’ll summarize them in this section.

<br />

##### Using Modules With `<script>`

The default behavior of the `<script>` element is to load JavaScript files as scripts (not modules). This happens when the type attribute is missing or when the type attribute contains a JavaScript content type (such as "text/javascript"). The `<script>` element can then execute inline code or load the file specified in src. To support modules, the "module" value was added as a type option. Setting type to "module" tells the browser to load any inline code or code contained in the file specified by src as a module instead of a script. Here’s a simple example:

```
<!-- load a module JavaScript file -->
<script type="module" src="module.js"></script>

<!-- include a module inline -->
<script type="module">

import { sum } from "./example.js";

let result = sum(1, 2);

</script>
```

The first `<script>` element in this example loads an external module file using the src attribute. The only difference from loading a script is that "module" is given as the type. The second `<script>` element contains a module that is embedded directly in the web page. The variable result is not exposed globally because it exists only within the module (as defined by the `<script>` element) and is therefore not added to window as a property.

As you can see, including modules in web pages is fairly simple and similar to including scripts. However, there are some differences in how modules are loaded.

> You may have noticed that "module" is not a content type like the "text/javascript" type. Module JavaScript files are served with the same content type as script JavaScript files, so it’s not possible to differentiate solely based on content type. Also, browsers ignore `<script>` elements when the type is unrecognized, so browsers that don’t support modules will automatically ignore the `<script type="module">` line, providing good backwards-compatibility.

<br />

##### Module Loading Sequence in Web Browsers

Modules are unique in that, unlike scripts, they may use import to specify that other files must be loaded to execute correctly. To support that functionality, `<script type="module">` always acts as if the defer attribute is applied.

The defer attribute is optional for loading script files but is always applied for loading module files. The module file begins downloading as soon as the HTML parser encounters `<script type="module">` with a src attribute but doesn’t execute until after the document has been completely parsed. Modules are also executed in the order in which they appear in the HTML file. That means the first `<script type="module">` is always guaranteed to execute before the second, even if one module contains inline code instead of specifying src. For example:

```
<!-- this will execute first -->
<script type="module" src="module1.js"></script>

<!-- this will execute second -->
<script type="module">
import { sum } from "./example.js";

let result = sum(1, 2);
</script>

<!-- this will execute third -->
<script type="module" src="module2.js"></script>
```

These three <`script>` elements execute in the order they are specified, so module1.js is guaranteed to execute before the inline module, and the inline module is guaranteed to execute before module2.js.

Each module may import from one or more other modules, which complicates matters. That’s why modules are parsed completely first to identify all import statements. Each import statement then triggers a fetch (either from the network or from the cache), and no module is executed until all import resources have first been loaded and executed.

All modules, both those explicitly included using `<script type="module">` and those implicitly included using import, are loaded and executed in order. In the preceding example, the complete loading sequence is:

1. Download and parse module1.js.
2. Recursively download and parse import resources in module1.js.
3. Parse the inline module.
4. Recursively download and parse import resources in the inline module.
5. Download and parse module2.js.
6. Recursively download and parse import resources in module2.js

Once loading is complete, nothing is executed until after the document has been completely parsed. After document parsing completes, the following actions happen:

1. Recursively execute import resources for module1.js.
2. Execute module1.js.
3. Recursively execute import resources for the inline module.
4. Execute the inline module.
5. Recursively execute import resources for module2.js.
6. Execute module2.js.

Notice that the inline module acts like the other two modules except that the code doesn’t have to be downloaded first. Otherwise, the sequence of loading import resources and executing modules is exactly the same.

> The defer attribute is ignored on `<script type="module">` because it already behaves as if defer is applied.

<br />

##### Asynchronous Module Loading in Web Browsers

You may already be familiar with the async attribute on the `<script>` element. When used with scripts, async causes the script file to be executed as soon as the file is completely downloaded and parsed. The order of async scripts in the document doesn’t affect the order in which the scripts are executed, though. The scripts are always executed as soon as they finish downloading without waiting for the containing document to finish parsing.

The async attribute can be applied to modules as well. Using async on `<script type="module">` causes the module to execute in a manner similar to a script. The only difference is that all import resources for the module are downloaded before the module itself is executed. That guarantees all resources the module needs to function will be downloaded before the module executes; you just can’t guarantee when the module will execute. Consider the following code:

```
<!-- no guarantee which one of these will execute first -->
<script type="module" async src="module1.js"></script>
<script type="module" async src="module2.js"></script>
```

In this example, there are two module files loaded asynchronously. It’s not possible to tell which module will execute first simply by looking at this code. If module1.js finishes downloading first (including all of its import resources), then it will execute first. If module2.js finishes downloading first, then that module will execute first instead.

<br />

##### Loading Modules as Workers

Workers, such as web workers and service workers, execute JavaScript code outside of the web page context. Creating a new worker involves creating a new instance Worker (or another class) and passing in the location of JavaScript file. The default loading mechanism is to load files as scripts, like this:

```
// load script.js as a script
let worker = new Worker("script.js");
```

To support loading modules, the developers of the HTML standard added a second argument to these constructors. The second argument is an object with a type property with a default value of "script". You can set type to "module" in order to load module files:

```
// load module.js as a module
let worker = new Worker("module.js", { type: "module" });
```

This example loads module.js as a module instead of a script by passing a second argument with "module" as the type property’s value. (The type property is meant to mimic how the type attribute of `<script>` differentiates modules and scripts.) The second argument is supported for all worker types in the browser.

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