# 解构（Destructuring for Easier Data Access）


对象和数组字面量在 JavaScript 中是最常出现的两种表现形式。感谢 JSON 这种数据格式的流行，它们在该门语言中的地位变得举足轻重。事先定义好对象和数组，并在需要的时候系统性地提取出需要的部分，这在编程中十分常见。为了简化从数据结构中获取相关的子集的操作，ECMAScript 6 引入了解构（destructuring）。

<br />

### 解构的实用性在哪里（Why is Destructuring Useful?）


在 ECMAScript 5 或更早的版本中，从对象或数组中抓取特定的数据并赋值给本地变量需要书写很多并且相似的代码。例如：

```
let options = {
        repeat: true,
        save: false
   };

// 从对象中提取数据
let repeat = options.repeat,
    save = options.save;
```

这段代码反复地提取在 options 上存储地属性值并将它们传递同名的本地变量。虽然看起来比较简单，不过想象一下如果你有一大批变量由相同需求，你就只能一个一个地赋值。而且如果你需要从对象内部嵌套的结构来查找想要的数据，你极有可能为了一小块数据而走遍了整个结构。

这也是 ECMAScript 6 给对象和数组添加解构的原因。当你想要把数据结构分解为更小的部分时，从这些部分中提取数据会更容易些。很多语言都能使用精简的语法来实现解构操作。ECMAScript 6 实现解构的实际语法或许你已经非常熟悉：对象和数组字面量。

<br />

### 对象解构（Object Destructuring）


对象结构语法在赋值语句的左侧使用对象字面量，例如：

```
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

在该段代码中，node.type 的值由 type 这个本地变量存储，node.name 同理。该语法和第四章中介绍的简写的属性初始化是相同的。type 和 name 标识符具有本地声明变量和 options 对象属性的双重身份。

<br />

> ##### 不要忘记初始化（Don’t Forget the Initializer）
> 
> 当在解构中使用 var，let 或 const 来声明变量时，必须要由初始化操作。下面的代码会因为未初始化的存在而抛出错误：

```
// 语法错误！
var { type, name };

// 语法错误！
let { type, name };

// 语法错误！
const { type, name };
```
> when using destructuring.const 即使未使用解构也需要初始化，而 var 和 let 只有在解构的时候才会被强制要求初始化。

<br />

##### 解构赋值表达式（Destructuring Assignment）


目前为止的解构示例使用了变量声明。不过，在表达式中使用赋值也是可疑的。例如，你可以决定在变量声明之后改变它们的值，如下所示：

```
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;

// assign different values using destructuring
({ type, name } = node);

console.log(type);      // "Identifier"
console.log(name);      // "foo"
```

在本例中，node 对象中的 type 和 name 在声明处初始化，而另一个对同名变量在之后也被不同的值初始化。往下的一行使用了解构赋值表达式将两个变量的值更改为 node 对象对应属性的值。注意你必须在圆括号内才能使用解构表达式。这是因为暴露的花括号会被解析为块声明语句，而该语句不能存在于赋值操作符的左侧。圆括号的存在预示着之后的花括号不是块声明语句而应该被看作表达式，这样它才能正常工作。

解构赋值表达式会计算右侧的值（= 右侧）。也就是说你可以在任何期望传值的位置使用表达式。例如，将值传给函数：

```
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

outputInfo() 函数在调用时被传入解构赋值表达式。表达式计算的结果为 node 是因为它就是右侧的值。type 和 name 的赋值正常进行同时 node 被传给了 outputInfo()。 

<br />

> **注意**: 当解构赋值表达式的右侧（= 后面的表达式）计算结果为 null 或 undefined 的时候一个错误将被抛出。因为任何读取 null 或 undefined 的操作都会发生运行时错误（runtime error）

<br />

##### 默认值（Default Values）


当你使用解构赋值表达式语句时，如果你定义了一个变量而该变量名在对象中找不到对应的属性名，那么该本地变量的值为 undefined。例如：

```
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // undefined
```

该段代码额外添加了一个本地变量 value 并试图获取相应的值。然而，node 对象没有对应的属性，所以正如我们所想的那样 value 的值为 undefined。

你可以选择定义一个默认值以防对象中不存在对应的属性。想要这么做的方法是在变量后添加等于号（=）并写下默认值，像这样：

```
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type, name, value = true } = node;

console.log(type);      // "Identifier"
console.log(name);      // "foo"
console.log(value);     // true
```

在该例中，value 变量的默认值为 true。默认值只有在 node 内部没有 value 属性或 value 的属性值为 undefined 的时候才会被使用。既然现在没有 node.value 这个属性，那么 value 变量值就是默认值。这和第三章介绍的 “函数中的默认参数” 一节十分类似。

<br />

##### 赋值给不同的变量名（Assigning to Different Local Variable Names)


到目前为止，每个示例中的解构赋值都使用对象中的属性名做为本地变量的名称；例如，node.type 中的值由 type 变量来存储。在你有意这么做的时候并没有什么问题，但万一你不想呢？ECMAScript 6 对此添加了扩展语法允许你将值赋给不同名字的变量（别名），而且该语法看上去类似于使用非简写方式初始化对象字面量的属性。这里有个示例：

```
let node = {
        type: "Identifier",
        name: "foo"
    };

let { type: localType, name: localName } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "foo"
```

以上的代码使用了解构赋值声明了 localType 和 localName 变量，分别获取 node.type 和 node.name 的值。type: localType 语法的意思是寻找 type 属性并将其值赋给 localType 这个变量。这种语法完全不同于传统的对象字面量语法。后者是属性名在冒号左侧，值为右侧，而前者是属性名在右侧，左侧是要读取的属性值。

你也可以在别名后面添加默认值，方式依然是在其后添加等于符号和默认值，例如：

```
let node = {
        type: "Identifier"
    };

let { type: localType, name: localName = "bar" } = node;

console.log(localType);     // "Identifier"
console.log(localName);     // "bar"
```

在这里，localName 变量的默认值是 "bar"。该值会赋给 localName 是因为 node.name 属性是不存在的。

到现在你已经知道怎样使用解构来操作对象中名称为原始值（primitive value）的属性。对象解构同样可以在包含嵌套结构的对象中获取数据。

<br />

##### 嵌套的对象解构（Nested Object Destructuring）


使用类似于对象字面量的语法可以在对象嵌套的结构中提取你需要的数据。这里有个示例：

```
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

该例中使用的解构模式包含了花括号来指示在 node 属性 loc 的内部寻找 start 属性。上一节曾提到不论冒号在何处出现，左面的标识符是查找的参考对象而右侧的标识符是赋值的目标。当冒号的后面出现花括号时，代表寻找的目标在该对象更深的层级中。

你可以进一步对嵌套的属性使用别名来声明变量：

```
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

在这种写法的代码中，node.loc.start 的值由新命名的 localStart 变量存储。解构可以在任意的嵌套深度工作，而且每一层级的功能都不会被削减。

对象解构十分强大而且形式多样，但数组解构提供了另一些独特的功能用来提取数组中的数据。

<br />

> ##### 语法注意（Syntax Gotcha）
> 当在嵌套的数据结构中使用解构时要注意，你可能会无意间创建了无任何意义的语句。在对象解构时空的花括号是允许的，然而它们不会做任何事情。例如：

```
// 无任何变量声明！
let { loc: {} } = node;
```

> 在该声明语句中不存在任何绑定。因为冒号右侧的花括号中无内容，loc 是用来做参考对象而没有创建任何绑定。在该种情况下，该语句的意图应该是使用 = 来声明默认值而不是用 : 来定义参照对象。这种语法在将来可能会被判为违法，但是现在我们只能对它保持警惕。

<br />

### 数组解构（Array Destructuring)


数据解构的语法和对象解构看起来类似，只是将对象字面量替换成了数组字面量，而且解构操作的是数组内部的位置（索引）而不是对象中的命名属性，例如：

```
let colors = [ "red", "green", "blue" ];

let [ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

在这里，数组解构从 colors 数组中找到了 "red" 和 "green" 并将它们赋值给 fristColor 和 secondColor 变量。这些值的选择和它们在数组中的位置有关；实际的变量名称是任意的。任何没有在数组解构语句中显示声明的项都会被忽略掉。注意的是数组本身不会有任何影响：

你可以在解构语句忽略一些项而只给你想要的项提供命名。例如，你若只想获取数组中的第三个元素，那么你不必给前两项提供命名。以下示例演示了该情况：

```
let colors = [ "red", "green", "blue" ];

let [ , , thirdColor ] = colors;

console.log(thirdColor);        // "blue"
```

这代码中使用了解构赋值来获取 colors 中的第三个元素。thirdColor 之前的逗号是先前元素的占位符。使用该种方法你可以轻松的获取数组中任意位置的值而不需要给其它项提供命名。

和对象解构类似的是，你必须使用 var，let，const 对数组解构进行初始化。

<br />

##### 解构赋值表达式（Destructuring Assignment）


你可以想要赋值的情况下使用数组的解构赋值表达式，但是和对象解构不同，没必要将它们包含在圆括号中，例如：

```
let colors = [ "red", "green", "blue" ],
    firstColor = "black",
    secondColor = "purple";

[ firstColor, secondColor ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

该段代码的解构赋值和上例类似。唯一的区别是 firstColr 和 secondColor 在事先已定义。大部分情况下，上述代码使你足够了解如何去使用数组的解构赋值表达式，但其实它还有一个额外的比较实用的用法。

数组中的解构赋值表达式有一个独特的使用场景 —— 对两个变量的值进行交换。在排序算法中值交换操作是非常普遍的，在 ECMAScript 5 中则需要一个第三方变量，如下所示：

```
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

临时变量 tmp 对交换 a 和 b 的值来讲是必要的。不过在使用解构赋值表达式时，它就没有存在的理由了。这里演示了在 ECMAScript 6 中交换变量值的方式：

```
// 在 ECMAScript 6 中交换变量的值
let a = 1,
    b = 2;

[ a, b ] = [ b, a ];

console.log(a);     // 2
console.log(b);     // 1
```

该例中的解构赋值表达式仿佛在进行镜像操作。表达式的左侧（等于符号之前）和其它数组解构案例别无二致，右侧则是为了交换值而创建的临时数组字面量。解构发生在临时数组上，将 b 和 a 的值分别传给左侧数组第一及第二个元素，效果等同于交换变量值。

和对象的解构赋值表达式相同，若表达式右侧的计算值为 null 和 undefined，那么该解构赋值表达式会抛出错误。

<br />

##### 默认值（Default Values）


数组中的解构赋值表达式同样可以在任意位置指定默认值。当某个位置的项未被传值或传入的值为 undefined，那么它的默认值会被使用。例如：

```
let colors = [ "red" ];

let [ firstColor, secondColor = "green" ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

在这段代码中，colors 数组只有一个项，所以 secondColor 不会进行值得匹配。既然它被设置了默认值，那么 secondColor 的值即为 "green" 而不是 undefined 。

<br />

##### 嵌套的数组解构（Nested Destructuring）


你可以使用类似于解构嵌套对象的方式来解构嵌套的数组。在数组的内部添加一个数组即可在嵌套数组完成操作。像这样：

```
let colors = [ "red", [ "green", "lightgreen" ], "blue" ];

// 之后

let [ firstColor, [ secondColor ] ] = colors;

console.log(firstColor);        // "red"
console.log(secondColor);       // "green"
```

这在里，secondColor 变量获得是 colors 数组中的 "green"。该项被包含在另一个数组中，所以解构语句中额外的数组添加是必须的。和对象相同，获取任意嵌套深度的值是允许的。

<br />

##### 剩余项（Rest Items）


第三章介绍过剩余参数表达式在函数中的应用，而数组解构中有个类似的概念叫做剩余项。它使用 ... 语法来将剩余的项赋值给一个指定的变量，如下所示：

```
let colors = [ "red", "green", "blue" ];

let [ firstColor, ...restColors ] = colors;

console.log(firstColor);        // "red"
console.log(restColors.length); // 2
console.log(restColors[0]);     // "green"
console.log(restColors[1]);     // "blue"
```

colors 中的首项会被赋值给 firstColor，而其余项会被添加到 restColors 这个新数组中。该数组因此会有两个项："green 和 "blue"。剩余项在提取数组中特定的项并保持其它项可用的情况下十分好用，而且还有另一种实用的方法。

数组的复制在 JavaScript 中是一个容易忽视的问题。在 ECMAScript 5 中，开发者经常使用 concat() 方法来方便的克隆一个数组，例如：

```
// ECMAScript 5 中克隆数组的方法
var colors = [ "red", "green", "blue" ];
var clonedColors = colors.concat();

console.log(clonedColors);      //"[red,green,blue]"
```

虽然 concat() 方法的本意是用来合并两个数组，但是没有给它传递参数的时候他会返回一个该数组的克隆。在 ECMAScript 6 中，你可以使用剩余项来完成同样的任务。实现如下：

```
// ECMAScript 6 中克隆数组的方法
let colors = [ "red", "green", "blue" ];
let [ ...clonedColors ] = colors;

console.log(clonedColors);      //"[red,green,blue]"
```

在该例中，剩余项用来将 colors 数组中的项拷贝到 clonedColors 。虽然关于剩余项的复制意图看上去是否比 concat() 方法更为明确的争论是仁者见仁智者见智，但是它依然是个值得了解的使用方法。

剩余项必须是解构语句中的最后项并且不能在后面添加逗号，因为该行为会抛出语法错误。


