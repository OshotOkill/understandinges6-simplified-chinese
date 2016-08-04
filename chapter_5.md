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

A destructuring assignment expression evaluates to the right side of the expression (after the =). That means you can use a destructuring assignment expression anywhere a value is expected. For instance, passing a value to a function:

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

By using a syntax similar to object literals, you can navigate into a nested object structure to retrieve just the information you want. Here’s an example:

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

The destructuring pattern in this example uses curly braces to indicate that the pattern should descend into the property named loc on node and look for the start property. Remember from the last section that whenever there’s a colon in a destructuring pattern, it means the identifier before the colon is giving a location to inspect, and the right side assigns a value. When there’s a curly brace after the colon, that indicates that the destination is nested another level into the object.

You can go one step further and use a different name for the local variable as well:

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

// extract node.loc.start
let { loc: { start: localStart }} = node;

console.log(localStart.line);   // 1
console.log(localStart.column); // 1
```

In this version of the code, node.loc.start is stored in a new local variable called localStart. Destructuring patterns can be nested to an arbitrary level of depth, with all capabilities available at each level.

Object destructuring is very powerful and has a lot of options, but array destructuring offers some unique capabilities that allow you to extract information from arrays.

<br />

> ##### 语法陷阱（Syntax Gotcha）
Be careful when using nested destructuring because you can inadvertently create a statement that has no effect. Empty curly braces are legal in object destructuring, however, they don’t do anything. For example:

```
// no variables declared!
let { loc: {} } = node;
```

> There are no bindings declared in this statement. Due to the curly braces on the right, loc is used as a location to inspect rather than a binding to create. In such a case, it’s likely that the intent was to use = to define a default value rather than : to define a location. It’s possible that this syntax will be made illegal in the future, but for now, this is a gotcha to look out for.

<br />

### 数组解构（Array Destructuring)


