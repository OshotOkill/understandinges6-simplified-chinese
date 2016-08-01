# 块级绑定 （Block Bindings）

变量声明的工作方式向来是 JavaScript 编程中难以理解的部分之一。在大部分C和类C（C-based）语言中，变量的声明与创建（或绑定）发生在同一位置，然而在 JavaScript 中情况就有所不同，变量的创建方式取决于你如何声明它，ECMAScript 6 提供了额外的选项方便你能自由控制变量的作用范围。本章会演示为什么传统的 var 声明令人费解，并引出 ECMAScript 6 中的块级绑定，罗列一些实践场景来使用它们。

<br />

#### var 声明与变量提升 （Var Declarations and Hoisting）

使用 var 关键字声明的变量，不论在何处都会被视作在函数级作用域内顶部的位置发生（如果不包含在函数内则为全局作用域内）。为了说明变量提升到底是什么，查看如下函数定义：

```
function getValue(condition) {

    if (condition) {
        var value = "blue";

        // 其它代码

        return value;
    } else {

        // value 可以被访问到，其值为 undefined

        return null;
    }

    // 这里也可以访问到 value，值仍为 undefined
}
```

如果你不太熟悉 JavaScript，或许会认为只有 condition 为 true 的变量 value 才会被创建。实际上，value 总是会被创建。JavaScript 引擎在幕后对 getValue 函数做了调整，可以视为：

```
function getValue(condition) {

    var value;

    if (condition) {
        value = "blue";

        // 其它代码

        return value;
    } else {

        return null;
    }
}
```

变量的声明被提升至顶部，但是初始化的位置并没有改变，这意味着在 else 从句内部也能访问 value 变量，但如果真的这么做的话，value 的值会是 undefined，因为它并没有被初始化或赋值。

刚开始接触 JavaScript 的开发者总是要花一段时间来习惯变量提升，对该独特概念的陌生也会造成 bug。因此 ECMAScript 6 引入了块级作用域的概念使得变量的生命周期变得更加可控。

<br />

#### 块级声明（Block-Level Declarations）

块级声明指的是该声明的变量无法被代码块外部访问。块作用域，又被称为词法作用域（lexical scopes），可以在如下的条件下创建：

* 函数内部
* 在代码块（即 { 和 } 内部）内部

块级作用域是很多类C语言的工作机制，ECMAScript 6 引入块级声明的目的是增强 JavaScript 的灵活性，同时又能与其它编程语言保持一致。

<br />

#### let 声明

let 声明的语法和 var 完全一致。你可以简单的将所有 var 关键字替换成 let，但是变量的作用域会限制在当前的代码块中（稍后讨论其它细微的差别）。既然 let 声明不会将变量提升至当前作用域的顶部，你或许要把它们手动放到代码块的开头，因为只有这样它们才能被代码块的其它部分访问。举个例子：

``` let
function getValue(condition) {

    if (condition) {
        let value = "blue";

        // 其它代码

        return value;
    } else {

        // value 并不存在（无法访问）

        return null;
    }

    // 这里 value 也不存在
}
```

本次 getValue 函数的写法的默认行为更贴近你脑海中C和其它类C语言的印像。既然变量的声明由 var 替换成了 let，那么它们就不会自动提升到当前函数作用域的顶部，除非执行流程到了 if 从句内部，其它情况时是没有办法对该变量进行访问的。如果 if 从句中 condition 的值为 false，那么 value 变量就不会被声明或者初始化。

<br />

#### 禁止重复声明


如果一个标识符在当前作用域里已经存在，那么再用 let 声明相同的标识符或抛出错误

``` duplicate
var count = 30;

// Syntax error
let count = 40;
```

在本例中，count 被声明了两次： 一次是被 var 另一次被 let 。因为 let 不会重新定义已经存在的标识符，所以会抛出一个错误。反过来讲，如果在当前包含的作用域内 let 声明了一个全新的变量，那么就不会有错误抛出。正如以下的代码演示：

```
var count = 30;

// 不会抛出错误
if (condition) {

    let count = 40;

    // 其它代码
}
```

本次 let 声明没有抛出错误的原因是 let 声明的变量 count 是在 if 从句的代码块中，并非和 var 声明的 count 处于同一级。在 if 代码块中，这个新声明的 count 会屏蔽掉全局变量 count ，避免在执行流程未跳出 if 之前访问到它。

<br />

#### const 声明（Constant Declarations）

在 ECMAScript 6 中也可使用常量（const）语法来声明变量。该种方式声明的变量会被视为常量，这意味着它们不能再次被赋值。由于这个原因，所有的 const 声明的变量都必须在声明处初始化。示例如下：

``` const
// 合法的声明
const maxItems = 30;

// Syntax error: missing initialization（语法错误：未初始化）
const name;
```

变量 maxItems 已经被初始化，所以这里不会出现任何问题。至于 name 变量，由于你未对其进行初始化赋值所以在运行时会报错。

<br />

##### const 声明 vs let 声明（Constants vs Let Declarations）


const 和 let 都是块级声明，意味着执行流跳出声明所在的代码块后就没有办法在访问它们，同样 const 变量也不会被提升，示例如下：

```
if (condition) {
    const maxItems = 5;

    // 其它代码
}

// maxItems 在这里无法访问
```

在这段代码中，maxItems 在 if 从句的代码块中作为常量被声明。一旦执行流跳出 if 代码块，外部就无法访问这个变量。

另一处和 let 相同的地方是，const 也不能对已存在的标识符重复定义，不论该标识符由 var（全局或函数级作用域）还是 let （块级作用域）定义。例如以下的代码：

```constVSlet
var message = "Hello!";
let age = 25;

// 下面每条语句都会抛出错误
const message = "Goodbye!";
const age = 30;

```

以上两条 const 声明如果单独存在即是合法的，很遗憾的是在本例中前面出现了 var 和 let 声明的相同标识符（变量）

尽管 const 和 let 使用起来很相似，但是必须要记住它们的根本性的差异：不管是在严格（strict）模式还是非严格模式（non-strict）模式下，const 变量都不允许被重复赋值。

```
const maxItems = 5;

maxItems = 6;      // 抛出错误
```

和其它编程语言类似，maxItems 不能被赋予新的值，然而和其它语言不同的是，const 变量的值如果是个对象，那么这个对象本身可以被修改。

##### 将对象赋值给 const 变量（Declaring Objects with Const）

const 声明只是阻止变量和值的再次绑定而不是值本身的修改。这意味着 const 不能限制对于值的类型为对象的变量的修改，示例如下：

```
const person = {
    name: "Nicholas"
};

// 正常
person.name = "Greg";

// 抛出错误
person = {
    name: "Greg"
};
```

在这里，person 变量一开始已经和包含一个属性的对象绑定。修改 person.name 是被允许的因为 person 的值（地址）未发生改变，但是尝试给 person 赋一个新值（代表重新绑定变量和值）的时候会报错。这个微妙之处会导致很多误解。只需记住：const 阻止的是绑定的修改，而不是绑定值的修改。

<br />

#### 暂存性死区（The Temporal Dead Zone）


let 或 const 声明的变量在声明之前不能被访问。如果执意这么做会出现错误，甚至是 typeof 这种安全调用（safe operations）也不被允许的：

```
if (condition) {
    console.log(typeof value);  // ReferenceError!
    let value = "blue";
}
```

在这里，变量 value 由 let 声明并被初始化，但是由于该语句之前抛出了错误导致其从未被执行。这种现象的原因是该语句存在于被 JavaScript 社区泛称为暂存性死区（Temproal Dead Zone, TDZ）之内。ECMAScript 规范中未曾对 TDZ 有过显式命名，不过在描述 let 和 const 声明的变量为何在声明前不可访问时，该术语经常被使用。本章会涵盖在 TDZ 中关于声明位置的一些微妙部分。此外示例虽然全部用的是 let ，但是实际用法和 const 别无二致。

当 JavaScript 引擎在作用域中寻找变量声明时，会将变量提升到函数/全局作用域的顶部（var）或是放入 TDZ（let 和const）内部。任何试图访问 TDZ 内部变量的行为都会以抛出运行时（runtime）错误而告终。当执行流达到变量声明的位置时，变量才会被移出 TDZ ，代表它们可以被安全使用。

由 let 或 const 声明的变量，如果你想在定义它们之前就使用，那么以上所述的准则都是铁打不变的。正如之前的示例所演示的那样，typeof 操作符都不能幸免。不过，你可以在代码块之外的地方对变量使用 typeof 操作符，但结果可能并不是你想要的。考虑如下的代码：

```
console.log(typeof value);     // "undefined"

if (condition) {
    let value = "blue";
}
```

当对 value 变量使用 typeof 操作符时它并没有处在 TDZ 内部，因为它的位置在变量声明位置所在的代码块之外。这意味着没有发生任何绑定，所以 typeof 仅返回 “undefined”

TDZ 只是发生在块级绑定中独特的特设定之一，另一个特殊设定发生在循环中。

#### 循环中的块级绑定（Block Binding in Loops）

或许开发者对块级作用域有强烈需求的场景之一就是循环，因为它们不想让循环外部访问到内部的索引计数器。举个例子，以下的代码在 JavaScript 编程中并不罕见：

```
for (var i = 0; i < 10; i++) {
    process(items[i]);
}

// 这里仍然可以访问到 i
console.log(i);                     // 10
```

块级作用域在其它语言内部是默认的，以上的代码的执行过程也并无差异，但区别在于变量 i 只能在循环代码块内部使用。然而在 JavaScript中，变量的提升导致块外的部分在循环结束后依然可以访问 i 。若将 var 替换为 let 则更符合预期：

``` block-bindings-in-loop
for (let i = 0; i < 10; i++) {
    process(items[i]);
}


// 在这里访问 i 会抛出错误
console.log(i);
```

在本例中变量 i 只存在于 for 循环代码块中，一旦循环完毕变量 i 将不复存在。

#### 循环中的函数（Functions in Loops）


长久以来 var 声明的特性使得在循环中创建函数问题多多，因为循环中声明的变量在块外也可以被访问，考虑如下的代码：

``` functions-in-loop
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push(function() { console.log(i); });
}

funcs.forEach(function(func) {
    func();     // 输出 "10" 共10次
});
```

你可能认为这段代码只是普通的输出 0 - 9 这十个数字，但事实上它会连续十次输出 “10”。这是因为每次迭代的过程中 i  是被共享的，意味着循环中创建的函数都保持着对相同变量的引用。当循环结束后 i 的值为 10，于是当 console.log(i)被调用后，该值会被输出。 


为了修正这个问题，开发者们在循环内部使用即时调用函数表达式（immediately-invoked function expressions, IIFEs）来迫使每次迭代时创建一份当前索引值的拷贝，示例如下：

``` IIFE-in-loop
var funcs = [];

for (var i = 0; i < 10; i++) {
    funcs.push((function(value) {
        return function() {
            console.log(value);
        }
    }(i)));
}

funcs.forEach(function(func) {
    func();     // 输出 0，1，2 ... 9
});
```

这种写法在循环内部使用了 IIFE，并将变量 i 的值传入 IIFE 以便拷贝索引值并存储起来，这里传入的索引值为同样被当前的迭代所使用，所以循环完毕后每次调用的输出值正如所期待的那样是 0 - 9 。幸运的是，ECMAScript 6 中 let 和 const 的块级绑定对循环代码进行简化。

#### 循环中的 let 声明（Let Declarations in Loops）


let 声明通过有效地模仿 上例中 IIFE 的使用方式来简化循环代码。在每次迭代中，一个新的同名变量会被创建并初始化。这意味着你可以抛弃 IIFE 的同时也能获得相同的结果。

``` let-in-loop
var funcs = [];

for (let i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}

funcs.forEach(function(func) {
    func();     // 输出 0，1，2 ... 9
})
```

这段循环代码的执行效果完全等同于使用 var 声明和 IIFE，但显然更加简洁。let 声明使得每次迭代都会创建一个变量 i，所以循环内部创建的函数会获得各自的变量 i 的拷贝。每份拷贝都会在每次迭代的开始被创建并被赋值。这同样适用于 for-in 和 for-of 循环，如下所示：

``` for-in
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

for (let key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // 输出  "A"，"B" 和 "C"
});

```

在本例中，for-in 循环的表现和 for 循环相同。每次循环的开始都会创建一个新的变量 key 的绑定，所以每个函数都会有各自 key 变量值的备份。结果就是每个函数都会输出不同的值。如果循环中 key 由 var 声明，那么所有的函数会输出 "C" 。

<br />

> **注意**： 不得不提的是，let 声明在上述循环内部中的表现是在规范中特别定义的，和非变量提升这一特性没有直接的关系。实际上，早期 let 的实现并不会表现中这种效果，它是在后来被添加到规范中的。

<br />

#### 循环中的 const 声明（Constant Declarations in Loops）


ECMAScript 6 规范中没有明确禁止在循环中使用 const 声明，不过其具体的表现要取决于你使用哪种循环方式。对于普通 的 for 循环你可以在初始化（initializer）语句里使用 const 声明，但当你想要修改该声明变量时循环会报错：

```const-in-loop
var funcs = [];

// throws an error after one iteration
for (const i = 0; i < 10; i++) {
    funcs.push(function() {
        console.log(i);
    });
}
```

在这段代码中，变量 i 被声明为常量。在循环开始迭代，即 i 为 0 的时候，代码是可以运行的，不过当步骤执行到 i++ 的那一刻会发生错误，因为你正在修改一个常量。由此，只有在循环过程中不更改在初始化语句中声明的变量的条件下，才能在里面使用 const 声明。


另外，当使用 for-in 或 for-of 循环时，const 声明的变量的表现和 let 完全一致。所以以下的代码不会出错：

```
var funcs = [],
    object = {
        a: true,
        b: true,
        c: true
    };

// 不会报错
for (const key in object) {
    funcs.push(function() {
        console.log(key);
    });
}

funcs.forEach(function(func) {
    func();     // 输出 "a"，"b" 和 "c"
});
```

这段代码的作用和 “循环中的 let 声明” 一节的第二个示例相同，唯一的差异是变量 key 的值不能被修改。for-in 和 for-of 循环能正常使用 const 是因为每次迭代都会创建一个新的变量绑定而不是去试图修改已存在的绑定（如上例）。

<br />

#### 全局块级绑定（Global Block Bindings）


let 与 const 另一处不同体现在全局作用域上。当在全局作用域内使用 var 声明时会创建一个全局变量，同时也是全局对象（浏览器环境下是 window）的一个属性。这意味着全局对象的属性可能会意外地被重写覆盖，例如：

```global-block-bindings
// 在浏览器中运行
var RegExp = "Hello!";
console.log(window.RegExp);     // "Hello!"

var ncz = "Hi!";
console.log(window.ncz);        // "Hi!"
```

虽然全局 RegExp 对象在 window 上已定义，但 var 声明很容易就能把它覆盖。这个例子就声明了一个新的 RegExp 变量并将原本的 RegExp 替换掉了。同样，ncz 也作为一个全局变量被声明并成为了 window 的属性。这就是 JavaScript 的工作机制。

如果你在全局作用域内使用 let 或 const，那么绑定就会发生在全局作用域内，但是不会向全局对象内部添加任何属性。这就意味着你不能使用 let 或 const 重写全局变量，而仅能屏蔽掉它们。如下所示：

```
// 在浏览器中运行
let RegExp = "Hello!";
console.log(RegExp);                    // "Hello!"
console.log(window.RegExp === RegExp);  // false

const ncz = "Hi!";
console.log(ncz);                       // "Hi!"
console.log("ncz" in window);           // false
```

在这里，let 声明创建了新的 RegExp 绑定并屏蔽掉了全局对象的 RegExp 属性。也就是说 window.RegExp 和 RegExp 并不等同，所以全局作用域并没有被污染。同样 const 声明创建了新的绑定的同时也没有在全局对象内部添加任何属性。这个设定使得在全局作用域内使用 let 或 const 声明要比 var 安全得多，特别是在你不想在全局对象内部添加属性的时候。

<br />

> **注意**： 如果你想让代码可以被全局对象访问，你仍然需要使用 var，特别是当你想要在多个 window 和 frame 之间共享代码的时候

<br />

#### 块级绑定的最佳实践（Emerging Best Practices for Block Bindings）


当 ECMAScript 6 还在酝酿中的时候，一个普遍的共识是使用 let 而不是 var 来作为默认的变量声明方式。对大多数 JavaScript 开发者来讲，let 才是 var 该有的表现形式，自然而然这种取代十分合理。在这个理念下，你应该使用 const 声明来保护一些变量不被修改。

然而，当越来越多的开发者迁移到 ECMAScript 6 之后，一个新的实践逐渐流行了起来：const 是声明变量的默认方式，仅当你明确哪些变量之后需要修改的情况下再用 let 声明那些变量。这个实践的缘由是大部分变量在初始化之后不应该被修改，因为这样做是造成 bug 的根源之一。这个理念有大批的受众而且在你接纳 ECMAScript 6 之后值得考虑。

<br />

#### 总结（Summary）

let 和 const 块级绑定给 JavaScript 引入了词法作用域的概念。这些声明不会被提升且仅存在于声明它们的代码块中。它们的行为和其它语言更为相似且减少了意外错误的发生，因为变量会在原处被声明。作为副作用之一，你不能在它们被声明之气就是用它们，即使是像 typeof 这种安全操作也不可以。暂存性死区（temproal dead zone, TDZ）中绑定的存在会导致声明之前的访问以失败告终。

在大多数情况下，let 和 const 的表现和 var 很相似，在循环中可不是这样。对于 let 和 const 来讲，每次迭代的开始都会创建新的绑定，意味着循环内部创建的函数可以在迭代时访问到当前的索引值，而不是在整个循环结束之后（即 var 的表现形式）。在 for 中使用 let 声明也同样适用，但 const 声明则会抛出错误。

目前关于块级绑定的最佳实践是使用 const 作为默认的声明方式，当变量需要更改时则用 let 声明，保证代码中最基本的不可变性能防止错误的发生。

<br />













