# 解构（Destructuring for Easier Data Access）

JavaScript 中对象和数组字面量是最常见的两种用法，也感谢流行的 JSON 数据格式，它们已是这门语言特别重要的部分。定义对象和数组是很常见的，然后有条不紊地从它们的结构中提取相关信息。ECMAScript 6 通过添加 *destructuring* 简化了这个工作，它是把数据结构细化为一些更小的部分的过程。本章给你展示如何运用对象和数组解构。

<br />

### 本章小结
* [为什么解构实用](#Why-is-Destructuring-Useful)
* [对象解构](#Object-Destructuring)
* [数组解构](#Array-Destructuring)
* [混合解构](#Mixed-Destructuring)
* [解构参数](#Destructured-Parameters)
* [总结](#Summary)

<br />

### <a id="Why-is-Destructuring-Useful"> 为什么解构实用（Why is Destructuring Useful?） </a>


在 ECMAScript 5 以及更早的版本中，从对象或数组中获取特定的数据并赋值给本地变量可能会导致有很多看起来同样的代码。例如：

```js
let options = {
        repeat: true,
        save: false
   };

// 从对象中提取数据
let repeat = options.repeat,
    save = options.save;
```

这段代码从 `options` 对象提取 `repeat` 和 `save` 并保存到本地的同名变量。尽管这段代码看起来简单，但设想一下假如你有大量的变量要赋值；你将得一个一个地赋值。而且假如有一个嵌套的数据结构要遍历查找信息，你可能仅仅是为了找一小块数据就要访问整个数据结构。

这就是为什么 ECMAScript 6 给对象和数组添加解构。当你把数据结构细化成更小的部分，从中获取你需要的信息就变得非常容易。许多语言用极少的语法实现解构使其使用过程更简单。ECMAScript 6 的实现实际上利用你已经熟悉的语法：对象和数组字面量语法。

<br />

### <a id="Object-Destructuring"> 对象解构（Object Destructuring） </a>


对象解构的语法是在赋值操作的左边使用对象字面量，比如：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

这段代码中，`node.type` 的值被存储到一个叫 `type` 的变量而 `node.name` 的值被存储到一个叫 `name` 的变量。这个语法与第4章中介绍的对象字面量属性初始化简写（ object literal property initializer shorthand ）一样。标识符 `type` 和 `name` 都是本地变量声明并从 `node` 对象中读取属性。

<br />

> ##### 不要忘记初始化（Don’t Forget the Initializer）
> 
> 当声明变量使用解构时使用 `var`，`let`，或 `const`，你必须提供一个初始化（等号后面的值）。下面的几行代码由于缺少初始化都将报语法错误：
>
> ```js
> // 语法错误！
> var { type, name };
>
> // 语法错误！
> let { type, name };
> 
> // 语法错误！
> const { type, name };
>```
>
> 尽管 `const` 总是需要初始化，甚至当使用非解构（ nondestructured ）变量时也需要初始化，`var` 和 `let` 只在使用解构时需要初始化。

<br />

##### 解构赋值表达式（Destructuring Assignment）


目前为止对象解构的示例使用了变量声明。不过，也能在赋值中使用解构。例如，你可以决定在变量声明之后改变它们的值，如下所示：

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

// 用解构分配不同的值
({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

这个例子中，`type` 和 `name` 在声明的同时带值初始化，随后两个同名变量被初始化为不同的值。下一行使用解构赋值，通过从 `node` 对象读取来改变那些值。注意，你必须在解构赋值语句周围放圆括号。那是因为花括号被认为是块语句，而块语句不能出现在赋值操作的左侧。圆括号中的大括号表示它不是块语句而应该被理解为表达式，所以允许完成赋值操作。

解构赋值表达式计算其右值（ `=` 后面 ）。这意味着你可以在任何被认为是值的地方使用解构赋值表达式。例如，传一个值给函数：

```js
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

function outputInfo(value) {
    console.log(value === node);        // true
}

outputInfo({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

调用 `outputInfo()` 函数带一个解构赋值表达式。表达式计算结果为 `node` 是因为它是解构赋值表达式的右值。给 `type` 和 `name` 的赋值操作正常进行而 `node` 被传给 `outputInfo()`。

<br />

> 当解构赋值表达式的右值（ `=` 后面的表达式 ）为 `null` 或 `undefined` 时报错。这是因为任何试图读取 `null` 或 `undefined` 的属性会导致运行时错误（ runtime error ）。

<br />

##### 默认值（Default Values）


在你使用解构赋值表达式时，假如你指定了一个本地变量而它在对象中不存在对应的属性名，那么那个本地变量被赋值为 `undefined`。比如：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // undefined
```

这段代码定义了一个额外的本地变量叫做 `value` 并试图给它赋值。但是，`node` 对象中没有相应的 `value` 属性，所以和预计的一样该变量被赋值为 `undefined`。

当指定的属性不存在时你可以随意地定义一个默认值来使用。在属性名的后面插入一个等号 `=` 并指定默认值就行了，像这样：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value = true } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // true
```

这个例子中，变量 `value` 被给予 `true` 作为默认值。这个默认值只有属性不在 `node` 对象中或属性的值为 `undefined` 时被使用。由于这里没有 `node.value` 属性，所以变量 `value` 使用默认值。其工作方式类似于第3章中讨论过的函数的默认参数值。

<br />

##### 赋值给不同的变量名（Assigning to Different Local Variable Names)

至此，每个解构赋值的示例使用了对象属性名作为本地变量名：例如，`node.type` 的值存储于 `type` 变量。在你想使用同名的时候没什么问题，但如果你不想呢？ECMAScript 6 有一个扩展语法允许你给本地变量分配不同的名字，而那个语法看起来就像对象字面量的非简写属性初始化（ nonshorthand property initializer ）语法。这是一个示例：

```js
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type: localType, name: localName } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "foo"
```

这的代码使用解构赋值声明了变量 `localType` 和 `localName`，它们分别包含来自属性 `node.type` 和 `node.name` 的值。语法 `type: localType` 表示读取名为 `type` 的属性并存储它的值到 `localType` 变量。这个语法实际上是相反的传统对象字面量语法，传统的对象字面量语法是名称在冒号左侧而值在右侧。这里，名称在冒号右侧而值读取的位置在左侧。

在使用不同的变量名时你也可以添加默认值。等号和默认值依旧放在本地变量名后面。例如：

```js
let node = {
        type: "Identifier"
    };

let { type: localType, name: localName = "bar" } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "bar"
```

这里，变量 `localName` 有一个默认值 `"bar"`。该变量被赋值为它的默认值是因为没有 `node.name` 属性。

到目前为止，你已经明白如何处理对象属性是原始值的解构。对象解构也能被用于在嵌套对象结构中取值。

<br />

##### 嵌套的对象解构（Nested Object Destructuring）

通过使用类似对象字面量的语法，你能进入嵌套对象结构只获取你想要的信息。这是一个示例

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

let { loc: { start }} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
```

这个例子中的解构模式使用花括号表示它应该从 `node` 向下进入名为 `loc` 的属性并寻找 `start` 属性。记得上一节每当解构模式中有冒号，意味着冒号前的标识符正在给定位进行检查，并给冒号右边赋值。当冒号后面有花括号，表示目标值嵌套于对象中的另一个层级中。

你也可以进一步给本地变量名使用不同的名称：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        }
    };

// 提取 node.loc.start
let { loc: { start: localStart }} = node;

console.log(localStart.line);   // 1
console.log(localStart.column); // 1
```

这个版本的代码中，`node.loc.start` 被存储于一个新的本地变量叫做 `localStart`。解构模式能被嵌套任意深度，每个层级所有解构的功能都是可用的。

对象解构十分强大而且形式多样，但数组解构提供了另一些独特的功能用来提取数组中的数据。

对象解构非常强大而且有许多选择，不过数组解构提供一些特有的功能，这些功能允许你们从数组中提取信息。

<br />

> ##### 语法注意（Syntax Gotcha）
>
> 使用嵌套解构时要注意，因为你可能无意中创建了一个无效语句。对象解构中空的花括号是合法的，但它不做任何事情，比如：

```js
// 无任何变量声明！
let { loc: {} } = node;
```

> 这个语句中没有绑定被声明。由于花括号在右边，`loc` 被用作位置检查而不是创建一个绑定。这种情况可能是想用 `=` 定义默认值而不是 `:` 定义一个位置。这种语法在未来可能被定为非法的，不过现在，这是一个需要留意的问题。

<br />

### <a id="Array-Destructuring"> 数组解构（Array Destructuring) </a>

数组解构的语法非常想对象解构；它只是用数组字面量语法替换对象字面量语法。解构操作数组内部的位置而不是对象中可用的已命名的属性。例如：

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

这里，数组解构从数组 `colors` 中提取值 `"red"` 和 `"green"` 并把它们存到变量 `firstColor` 和 `secondColor`。那些值被选中是因为它们在数组中的位置；实际上变量命名可以是任何东西。任何在解构模式中没有提到的项会被忽略。记住在任何方式中数组本身不会改变。

你也可以在解构模式中省略元素而只提供变量名称给你感兴趣的元素。比如，如果你只想要数组的第三个值，你无需给第一项和第二项提供变量名。这里是它如何工作的代码示例：

```js
let colors = [ "red", "green", "blue" ];

let [ , , thirdColor ] = colors;

console.log(thirdColor);        // "blue"
```

这的代码使用解构赋值获取 `colors` 中的第三个元素。这个模式中 `thirdColor` 前面的逗号是数组中在它之前的元素的占位符。通过使用这个方法，你能很容易地从数组中间的任意位置取值而无需给它们提供变量名。

> 与对象解构一样，使用 `var`，`let` 或 `const` 声明解构时你必须总是要提供初始化。

<br />

##### 解构赋值（Destructuring Assignment）

你可以在上下文中使用数组解构赋值，但与对象解构不同，无需把表达式包裹在圆括号中。例如：

```js
let colors = [ "red", "green", "blue" ],
    firstColor = "black",
    secondColor = "purple";

[ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

这段代码中的解构赋值的工作方式与上一个数组解构的例子类似。唯一的区别是 `firstColor` 和 `secondColor` 已经被定义。大多数情况下，这可能是你知道关于数组解构表达式的所有所有知识，但是你可能会发现它还有一个实用之处。

数组解构有一个非常特殊的用法使两个变量交换值更加容易。值的交换在排序算法中很常见，在 ECMAScript 5 交换变量的方式涉及一个第三方临时变量，如下面这个例子中示范：

```js
// 在 ECMAScript 5 中交换变量的值
let a = 1,
    b = 2,
    tmp;

tmp = a;
a = b;
b = tmp;

console.log(a);     // 2
console.log(b);     // 1
```

为了交换 `a` 和 `b` 的值临时变量 `tmp` 是必要的。不过，使用解构赋值，就不需要那个额外的变量了。这里是 ECMAScript 6 中如何交换变量的示例：

```js
// 在 ECMAScript 6 中交换变量的值
let a = 1,
    b = 2;

[ a, b ] = [ b, a ];

console.log(a);     // 2
console.log(b);     // 1
```

这个例子中的数组解构赋值看起来像是一个镜像。左值（等号前面）就像那些其他数组解构示例中一样它也是解构模式。右值是一个数组字面量它被临时创建用于交换变量的值。解构操作发生在有 `b` 和 `a` 两个元素的临时数组上，复制其到解构的第一个和第二个值。得到的效果就是变量已经交换了值。

> 与对象解构赋值一样，数组解构赋值表达式的右值为 `null` 或 `undefined` 时会导致程序报错。

<br />

##### 默认值（Default Values）

数组解构赋值同样允许你在数组的任何位置指定默认值。解构元素在给定的位置不存在或值为 `undefined` 时使用默认值。例如：

```js
let colors = [ "red" ];

let [ firstColor, secondColor = "green" ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

这段代码中，数组 `colors` 只有一个元素，所以解构中的 `secondColor` 没有匹配的元素。由于设置了默认值，所以 `secondColor` 的值是 `green` 而不是 `undefined`。

<br />

##### 嵌套的数组解构（Nested Destructuring）

你可以用解构嵌套对象同样的方式解构嵌套数组。通过在总模式中插入另一个数组模式，解构将向下进入嵌套的数组，像这样：

```js
let colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// 之后

let [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

这里 `secondColor` 变量是指数组 `colors` 中的值 `"green"`。它是第二个数组中的元素，所以 `secondColor` 周围额外的方括号在解构模式中是必要的。和对象一样，你可以嵌套任意深度的数组。

<br />

##### 剩余项（Rest Items）

第3章介绍了函数的剩余参数，数组解构有一个同样的概念称作 *剩余项（rest items）*。剩余项使用 `...` 语法把数组中剩余的元素赋值到一个特殊变量。这里是一个示例：

```js
let colors = [ "red", "green", "blue" ];

let [ firstColor, ...restColors ] = colors;

console.log(firstColor);        // "red"
console.log(restColors.length); // 2
console.log(restColors[0]);     // "green"
console.log(restColors[1]);     // "blue"
```

`colors` 中的第一个元素被赋值给 `firstColor`，其余元素被赋值给一个新的数组 `restColors`。因此，数组 `restColors` 有两个元素 `"green"` 和 `"blue"`。剩余项对于从数组中提取特定元素并保持其余元素可用是很有用的，除此之外它还有另一个有用的用法。

JavaScript 数组中明显缺少的是轻松创建克隆的功能。ECMAScript 5 中，开发者们往往使用 `concat()` 方法作为克隆数组的简单方式。例如：

```js
// ECMAScript 5 中克隆数组的方法
var colors = [ "red", "green", "blue" ];
var clonedColors = colors.concat();

console.log(clonedColors);      //"[red,green,blue]"
```

虽然 `concat()` 方法的目的是把两个数组合并到一起，但是不带参数调用它返回的是源数组的一个克隆。ECMAScript 6 中，你可以使用剩余项这种方式来实现与通过预期的语法功能相同的事。它的工作原理是这样的：

```js
// ECMAScript 6 中克隆数组的方法
let colors = [ "red", "green", "blue" ];
let [ ...clonedColors ] = colors;

console.log(clonedColors);      //"[red,green,blue]"
```

这个示例中，剩余被项用于复制数组 `colors` 的值到数组 `clonedColors`。虽然关于这个技术是否使开发者们的意图比使用 `concat()` 方法更加清晰是一个感觉上问题，但这是个需要知道的实用方法。

> 剩余项必须是解构数组的最后一项并且它的后面不能跟逗号。剩余项后面包含逗号是语法错误。

<br />

### <a id="Mixed-Destructuring"> 混合解构（Mixed Destructuring） </a>

对象和数组解构可以一起使用来创建更加复杂的表达式。这样做，你可以从任何混合的对象和数组中提取只是你要的信息。例如：

```js
let node = {
        type: "Identifier",
        name: "foo",
        loc: {
            start: {
                line: 1,
                column: 1
            },
            end: {
                line: 1,
                column: 4
            }
        },
        range: [0, 3]
    };

let {
    loc: { start },
    range: [ startIndex ]
} = node;

console.log(start.line);        // 1
console.log(start.column);      // 1
console.log(startIndex);        // 0
```

这的代码分别提取 `node.loc.start` 和 `node.range[0]` 到 `start` 和 `startIndex`。注意 `loc:` 和 `range:` 在解构模式中仅仅是 `node` 对象中对应属性的位置。当你使用混合的对象和数组解构时没有 `node` 不能被提取的部分。这个方法对于不用经过整个数据结构就能提取出 JSON 配置结构的值来说特别实用。

<br />

### <a id="Destructured-Parameters"> 解构参数（Destructured Parameters） </a>

在传递函数参数时，解构还有一个特别实用的用法。当 JavaScript 函数接受大量可配置参数时，常见的模式是创建一个属性为可选参数的 `option` 对象，像这样：

```js
// options 中的属性表示额外的参数
function setCookie(name, value, options) {

    options = options || {};

    let secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;

    // 设置 cookie 的代码
}

// 第三个参数映射为配置
setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```

许多 JavaScript 类库都有看起来和这个例子一样的 `setCookie()` 函数。这个函数中，参数 `name` 和 `value` 是必填项，而 `secure`，`path`，`domain` 和 `expires` 不是。由于其数据没有优先级顺序，与列出形参相比，只有一个带配置属性的 `options` 对象是没问题的。但是，这种方式下只通过查看函数定义你不可能知道函数的内容；你需要阅读函数主体。

解构参数提供了一个选择使函数需要的参数更加明确。解构参数使用一个对象或数组解构模式代替形参。为了演示，看一下这个来自上一个实例的函数 `setCookie()` 的重写版：

```js
function setCookie(name, value, { secure, path, domain, expires }) {

    // 设置 cookie 的代码
}

setCookie("type", "js", {
    secure: true,
    expires: 60000
});
```

这个函数类似于上一个示例，不过现在，第三个参数使用解构提取必要的数据。解构参数外面的参数要求明确，同时，对于使用 `setCookie()` 的人来说根据扩展参数就能很清楚地知道什么配置项可用。当然，假如第三个参数是必填的，它应该包含的值是非常清楚的。解构参数也和普通参数一样如果不传则值为 `undefined`。

<br />

> 本章节中解构参数拥有到目前为止你已经学到的所有解构的功能。你可以使用默认值，混合对象和数组模式，以及使用不同于你正在读取的属性的变量名。

<br />

#### 解构参数是必填的（Destructured Parameters are Required）

解构参数有有一个奇怪的地方就是，默认情况下，在调用函数时不提供解构参数则报错。例如，这样调用上个例子中的 `setCookie()` 函数会报错：

```js
// 错误!
setCookie("type", "js");
```

第三个参数丢失了，所以它为 `undefined`。导致这个错误的原因是解构参数实际上只是解构声明的简写。当 `setCookie()` 函数被调用时，JavaScript 引擎实际上是这样做的：

```js
function setCookie(name, value, options) {

    let { secure, path, domain, expires } = options;

    // 设置 cookie 的代码
}
```

由于当右值为 `null` 或 `undefined` 时解构会报错，所以当第三个参数不传给 `setCookie()` 函数时也是同样的道理。

如果你希望解构参数是必填的，那么这不是什么问题。但是如果你希望解构参数是可选的，你可以通过给解构参数提供默认值来解决，像这样：

```js
function setCookie(name, value, { secure, path, domain, expires } = {}) {

    // ...
}
```

该例向第三个参数提供了一个对象作为默认值。这意味着如果 `setCookie()` 的未被传入第三个参数，那么 `secure`，`path`，`domain` 和 `expires` 的值均为 `undefined`，而且没有错误被抛出。

这个示例提供了一个新的对象作为默认值给第三个参数。提供一个默认值给解构参数意味着如果 `setCookie()` 没有被提供第三个参数， `secure`，`path`，`domain`，以及 `expires ` 都将是 `undefined`，而不会报错。

<br />

#### 解构参数的默认值（Default Values for Destructured Parameters）

正如你在解构赋值时那样，你可以指定解构默认值给解构参数。只要在参数后面加等号并指定默认值。例如：

```js
function setCookie(name, value,
    {
        secure = false,
        path = "/",
        domain = "example.com",
        expires = new Date(Date.now() + 360000000)
    } = {}
) {

    // ...
}
```

这的代码中，解构参数中的每个属性都有一个默认值，所以可以不用为了使用正确的值检查是否给定的属性已经被包含了。

<br />

### <a id="Summary"> 总结（Summary） </a>

解构使 JavaScript 中的对象和数组的工作更加简单。使用熟悉的对象字面量和数组字面量语法，你可以提取部分数据结构来获取只有你感兴趣的信息。对象模式允许你从对象提取数据而数组模式允许你从数组提取数据。

对象和数组解构都可以指定默认值给任何为 `undefined` 的属性或元素并且当右值为 `null` 或 `undefined` 时报错。你也可以用对象和数组解构来定位深度嵌套数据结构，并下降至任意深度。

解构声明使用 `var`，`let` 或 `const` 来创建变量必须总是要初始化。解构赋值被用来替代其它赋值并且允许你解构为对象属性和已存在的变量。

参数解构使用解构语法使得在函数参数中使用可选对象变得透明化。你实际感兴趣的数据可以使用命名参数详列。参数解构可以是对象形式，数组形式或混合形式，并同时拥有这些形式的全部功能。

作为函数参数时解构参数使用解构语法使 `option` 对象更透明。实际你感兴趣的数据可以随其他形参一起被列出。解构参数可以是数组模式，对象模式，或混合模式，并且你可以使用解构的全部功能。

<br />


