# 迭代器与生成器（Iterators and Generators）


许多编程语言都做了这样的转变：迭代集合中的数据不再使用需要初始化变量并作为索引的 for 循环，转而使用迭代器（iterator）对象来程序化地返回集合中下一位置的项。迭代器使得集合的操作变得更容易，ECMAScript 6 也将其添加到了 JavaScript 当中。当迭代器和数组方法以及新添加的集合类型（如 set 和 map）结合之后，它就成为了高效处理数据的关键，而且该语言中很多部分都有迭代器的身影，例如新添加的 for-of 循环，扩展（...）运算符等。迭代器甚至还能简化异步编程。

本章涵盖了迭代器的许多实践，但首先，了解 JavaScript 添加迭代器的背景和缘由是很重要的。

<br />

### 本章小结
* [循环问题](#The-Loop-Problem)
* [什么是迭代器](#What-are-Iterators)
* [什么是生成器](#What-Are-Generators)
* [可迭代类型与 for-of](#Iterables-and-for-of)
* [内置的迭代器](#Built-in-Iterators)
* [扩展运算符与非数组可迭代类型](#The-Spread-Operator-and-Non-Array-Iterables)
* [迭代器高级用法](#Advanced-Iterator-Functionality)
* [运行异步任务](#Asynchronous-Task-Running)
* [总结](#Summary)

<br />

### <a id="The-Loop-Problem"> 循环问题（The Loop Problem） </a>


如果你曾使用过 JavaScript 来编程，那么你可能会见过下面的代码：

```
var colors = ["red", "green", "blue"];

for (var i = 0, len = colors.length; i < len; i++) {
    console.log(colors[i]);
}
```

这里使用了标准的 for 循环形式，使用变量 i 作为跟踪的索引。每次迭代 i 都会增加直到 i 大于数组的长度（存储在 len 中）

虽然该循环看上去确实简洁明了，但是当循环出现嵌套后复杂度会增加，同时还需要追踪多个变量。额外的复杂度易引出错误的发生，而且 for 循环天然的范例样式会书写在多个位置导致更多的错误出现。迭代器就是为了解决这个问题。

<br />

### <a id="What-are-Iterators"> 什么是迭代器（What are Iterators?） </a>


迭代器只是带有特殊接口的对象。所有迭代器对象都带有 next() 方法并返回一个包含两个属性的结果对象。这些属性分别是 value 和 done，前者代表下一个位置的值，后者在没有更多值可供迭代的时候为 true 。迭代器带有一个内部指针，来指向集合中某个值的位置。当 next() 方法调用后，指针下一位置的值会被返回。

若你在末尾的值被返回之后继续调用 next()，那么返回的 done 属性值为 true，value 的值则由迭代器设定。该值并不属于数据集，而是专门为数据关联的附加信息，如若该信息并未指定则返回 undefined 。迭代器返回的值和函数返回值有些类似，因为两者都是返回给调用者信息的最终手段。

了解上述的说明后，在 ECMAScript 5 中创建一个迭代器变得十分简单：

```
function createIterator(items) {

    var i = 0;

    return {
        next: function() {

            var done = (i >= items.length);
            var value = !done ? items[i++] : undefined;

            return {
                done: done,
                value: value
            };

        }
    };
}

var iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// for all further calls
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

createIterator() 函数返回一个带有 next() 方法的对象。每次调用该方法时，数组中下一位置的值会传给 value 并返回它。当 i 递增为 3 之后，返回的对象中 done 属性为 true，而 value 的值则是三元运算符的计算结果：undefind 。在数据集末尾之后的迭代，即末尾的数据返回之后再调用 next()，这两个属性值的结果也符合 ECMAScript 6 中的迭代器的规范。

从以上的示例来看，根据 ECMAScript 6 规范模拟实现的迭代器还是有些复杂。

幸运的是，ECMAScript 6 还提供了生成器，使得迭代器对象的创建容易了许多。

<br />

### <a id="What-Are-Generators"> 什么是生成器（What Are Generators?） </a>


生成器是返回迭代器的函数。生成器函数由 function 关键字和之后的星号（*）标识，同时还能使用新的 yield
关键字。星号的位置不能论是放在 function 关键字的后面还是在它们插入空格都是随意的，如下例所示：

```
// 生成器
function *createIterator() {
    yield 1;
    yield 2;
    yield 3;
}

// 调用生成器类似于调用函数，但是前者返回一个迭代器
let iterator = createIterator();

console.log(iterator.next().value);     // 1
console.log(iterator.next().value);     // 2
console.log(iterator.next().value);     // 3
```

createIterator() 前面的星号指示该函数是个生成器。ECMAScript 6 新引入的 yield 关键字指定迭代器调用 next() 时按顺序返回的值。本例中的生成的迭代器在 next() 方法调用后成功地返回了三个值：先是 1，接着是 2，最后是 3 。一个生成器可以被当做函数调用并创建迭代器。

或许生成器函数中最有意思的部分是，当执行流遇到 yield 语句时，该生成器就停止运转了。例如，当 yield 1 执行之后，该生成器函数就不会执行其它任何部分的代码直到迭代器再次调用 next() 。在那时，yield 2 会被执行。生成器函数在运行时能被中断执行的能力非常强大而且引出了很多有意思用法（在之后的 “迭代器高级用法” 小节介绍）。

yield 关键字可以和值或者是表达式一起使用，所以你可以通过生成器给迭代器添加一些项而并非手动地让迭代器一个接一个地返回它们。例如，下面演示了在 for 循环内部使用 yield：

```
function *createIterator(items) {
    for (let i = 0; i < items.length; i++) {
        yield items[i];
    }
}

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// 进一步调用
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

该例中 createIterator() 生成器函数被传入了一个数组。在函数内部，一个循环正在执行并把数组中的值返还给迭代器。每次遇到 yield 时，循环就会停止，而每次 next() 被调用时，循环又会继续运行直到再一次遇到 yield 语句。

生成器函数是 ECMAScript 6 引入的重要的特性之一。既然它是函数，那么它可以用在所有函数可用的位置上。本小节其余的部分则专注于其它且实用的方法来书写生成器。

<br />

> **注意**： yield 关键字只能用在生成器内部。在其它地方甚至是生成器内部的函数中使用都会抛出语法错误，例如：

```
function *createIterator(items) {

    items.forEach(function(item) {

        // 语法错误
        yield item + 1;
    });
}
```

> 尽管在严格意义上讲 yield 确实是在 createIterator() 内部，但 yield 是无法跨越函数边界的。某种程度上来说它和 return 比较类似，因为容器函数不能将内部函数的返回值直接作为自身的返回值。

<br />

#### 生成器函数表达式（Generator Function Expressions）


你可以使用函数表达式来创建生成器，只需在 function 关键字和圆括号之间添加星号（*）。例如：

```
let createIterator = function *(items) {
    for (let i = 0; i < items.length; i++) {
        yield items[i];
    }
};

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"

// 进一步调用
console.log(iterator.next());           // "{ value: undefined, done: true }"
```


该段代码中，createIterator() 是个生成器函数表达式而非函数声明。星号在 function 关键字和圆括号之间是因为函数表达式是匿名的。除此之外其它部分和上例是相同的，而且内部都含有 for 循环。

<br />

> 无法使用箭头函数来创建生成器。

<br />

#### 对象中的生成器方法（Generator Object Methods）


生成器本身只是函数，因此可以添加它们到对象中。例如，你可以使用 ECMAScript 5 风格的对象字面量来创建函数表达式：

```
var o = {

    createIterator: function *(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
};

let iterator = o.createIterator([1, 2, 3]);
```

你也可以使用 ECMAScript 6 的简写方法并将星号（*）放在方法名前面：

```
var o = {

    *createIterator(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
};

let iterator = o.createIterator([1, 2, 3]);
```

这些示例中的生成器和上一节 “生成器函数表达式” 中演示的功能相同；仅在语法方面有一些差异。在简写的生成器方法中，由于 createIterator() 方法没有 function 关键字，因此只能将星号放在方法名前面，当然你也可以在它们之间添加空白符。

<br />

### <a id="Iterables-and-for-of"> 可迭代类型与 for-of（Iterables and for-of） </a>


与迭代器紧密相关的是，可迭代类型是指那些包含 Symbol.iterator 属性的对象。该知名的 symbol 类型定义了返回迭代器的函数。在 ECMAScript 6 中，所有的集合对象（数组，set 和 map）与字符串都是可迭代类型，因此它们都有默认的迭代器。可迭代类型是为了 ECMAScript 新添加的 for-of 循环而设计的。

<br />

> 所有由生成器创建的迭代器都是可迭代类型，因为生成器在默认情况下会自赋值给 Symbol.iterator 属性。

<br />


在本章的开头我曾提到过在 for 循环中追踪索引的弊端。迭代器只是解决该问题的第一部分。for-of 循环则是第二部分：完全不需要在集合中追踪索引，让你更专注于集合内容的操作。

for-lof 循环会在可迭代类型每次迭代执行后调用 next() 并将结果对象存储在变量中。循环会持续进行直到结果对象的 done 属性为 true。如下所示：

```
let values = [1, 2, 3];

for (let num of values) {
    console.log(num);
}
```

This code outputs the following:

```
1
2
3
```

for-of 循环首先会调用 values 数组的 Symbol.iterator 方法来获取迭代器（Symbol.iterator 方法由幕后的 JavaScript 引擎调用）。之后再调用 iterator.next() 并将结果对象中的 value 属性值，即 1，2，3，依次赋给 num 变量。当检测到结果对象中的 done 为 true，循环会退出，所以 num 不会被赋值为 undefined 。

如果你只想简单的迭代数组或集合中的元素，那么 for-of 循环比 for 要更好。for-of 一般不容易出错，因为要追踪的条件更少。所以还是把 for 循环留给复杂控制条件的需求吧。

<br />

> **注意**：对非可迭代对象，null 和 undefind 使用 for-of 会抛出错误。

<br />

#### 访问默认迭代器（Accessing the Default Iterator）


你可以使用 Symbol.iterator 来访问对象默认的迭代器，像这样。

```
let values = [1, 2, 3];
let iterator = values[Symbol.iterator]();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

这段代码中获取了 values 默认的迭代器并迭代数组中的项。该过程和 for-of 循环幕后的操作流程是相同的。

既然 Symbol.iterator 定义了默认的迭代器，你可以如下使用它来确定一个对象是否可迭代：

```
function isIterable(object) {
    return typeof object[Symbol.iterator] === "function";
}

console.log(isIterable([1, 2, 3]));     // true
console.log(isIterable("Hello"));       // true
console.log(isIterable(new Map()));     // true
console.log(isIterable(new Set()));     // true
console.log(isIterable(new WeakMap())); // false
console.log(isIterable(new WeakSet())); // false
```

isIterable() 函数简单地查看对象是否有默认的并且类型为函数的迭代器。for-of 在执行前也会做相似的检查。

目前为止，本章的示例已说明如何使用可迭代对象内部的 Symbol.iterator，其实你还可以通过定义 Symbol.iterator 属性来创建自有的可迭代类型。

<br />

#### 创建可迭代类型（Creating Iterables）


开发者自定义的对象默认是不可迭代类型，但是你可以为它们创建 Symbol.iterator 属性并指定一个生成器来使这些对象可迭代。例如：

```
let collection = {
    items: [],
    *[Symbol.iterator]() {
        for (let item of this.items) {
            yield item;
        }
    }

};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}
```

该段代码输出:

```
1
2
3
```

首先，示例中创建了 collection 对象并包含一个默认的迭代器。该迭代器由 Symbol.iteartor 这个生成器方法来创建（注意星号仍然在方法名之前）。之后生成器使用 for-of 循环来迭代 this.items 中的元素并使用 yield 来返回它们。collection 对象依赖于默认的迭代器和 this.items 来工作，而非手动设定由默认迭代器返回的值。

<br />

> 本章之后的 “生成器代理” 一节会描述如何使用其它对象中的迭代器。

<br />

现在你已经见识了数组默认迭代器的用法，然而 ECMAScript 6 还内置了许多迭代器使得操作集合中的数据更加轻松。

<br />

### <a id="Built-in-Iterators"> 内置的迭代器（Built-in Iterators） </a>


迭代器是 ECMAScript 6 重要的一部分，你不需要为大部分内置类型创建自己的迭代器，因为 JavaScript 语言已经包含了它们。只有当这些内置的迭代器无法做出符合需求的行为时，你才需要考虑自行创建它们，尤其是在定义自己的对象和类的时候。否则，你完全可以用内置的迭代器去完成一些工作。或许使用迭代器最频繁是集合。

<br />

#### 集合迭代器（Collection Iterators）


ECMAScript 6 内置了三种类型的集合对象：数组，map 和 set 。它们都有如下内置的迭代器供你浏览数据。

* entries() - 返回一个数据集为集合中的键值对的迭代器
* values() - 返回一个数据集为集合中的值的迭代器
* keys() - 返回一个数据集为集合中的键的迭代器

你可以使用上述方法之一来提取集合中的迭代器。

<br />

##### entries() 迭代器（The entries() Iterator）


当每次 next() 被调用后，entries() 迭代器会返回包含两个项的数组。该数组中的项分别是集合中每一项的键和值。对于数组来讲，键是数字索引；对于 set ，每一项的键和值相同；而 map 则是正常返回每一项。

这里有一些该迭代器的演示：

```
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let entry of colors.entries()) {
    console.log(entry);
}

for (let entry of tracking.entries()) {
    console.log(entry);
}

for (let entry of data.entries()) {
    console.log(entry);
}
```

console.log() 会做如下输出：

```
[0, "red"]
[1, "green"]
[2, "blue"]
[1234, 1234]
[5678, 5678]
[9012, 9012]
["title", "Understanding ECMAScript 6"]
["format", "ebook"]
```

该段代码对每一个集合类型都使用了 entries() 方法以便获取对应的迭代器，之后使用 for-of 循环来迭代各自的项。控制台的输出清晰地显示了每个类型在每次迭代的返回结果。

<br />

##### values() 迭代器（The values() Iterator）


values() 迭代器简单地返回了集合中每一项的值，正如它们在集合中的表现的那样，例如：

```
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let value of colors.values()) {
    console.log(value);
}

for (let value of tracking.values()) {
    console.log(value);
}

for (let value of data.values()) {
    console.log(value);
}
```

这段代码输出如下：

```
"red"
"green"
"blue"
1234
5678
9012
"Understanding ECMAScript 6"
"ebook"
```

在本例中，调用 values() 迭代器返回了各自类型中对应的数据而并不需要获知数据在集合中的位置。

<br />

##### keys() 迭代器（the keys() Iterator）

The keys() iterator returns each key present in a collection. For arrays, it only returns numeric keys, never other own properties of the array. For sets, the keys are the same as the values, and so keys() and values() return the same iterator. For maps, the keys() iterator returns each unique key. Here’s an example that demonstrates all three:

keys() 迭代器返回集合中每一项的键。其中数组除了数字索引之外不会返回其它属性。由于 set 中的键和值相同，所以 keys() 和 values() 返回了相同的迭代器。对于 map 来讲，keys() 迭代器会返回每一项中的键。以下示例演示了这三种集合：

```
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let key of colors.keys()) {
    console.log(key);
}

for (let key of tracking.keys()) {
    console.log(key);
}

for (let key of data.keys()) {
    console.log(key);
}
```

该例做出如下输出：

```
0
1
2
1234
5678
9012
"title"
"format"
```

keys() 迭代器获取了 colors，tracking 和 data 各自所有的键，并将它们在 for-of 中打印输出。数组对象只会有索引数字输出，即使你尝试给数组添加命名属性也无济于事。这和 for-in 循环有些不同，因为 for-in 会迭代数组所有的属性而不仅仅是数字索引。

<br />

##### 集合类型的默认迭代器（Default Iterators for Collection Types）


每种集合类型都包含一个默认的迭代器以供 for-of 循环显式或隐式的使用。数组和 set 默认的迭代器是 values() 方法，而 map 则是 entries() 。这些默认设定能方便 for-of 循环迭代集合对象。例如，考虑如下的例子：

```
let colors = [ "red", "green", "blue" ];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "print");

// 等效于调用 colors.values()
for (let value of colors) {
    console.log(value);
}

// 等效于调用 tracking.values()
for (let num of tracking) {
    console.log(num);
}

// 等效于调用 using data.entries()
for (let entry of data) {
    console.log(entry);
}
```

由于集合未指定任何迭代器，所以默认的迭代器会被使用。默认迭代器是为了反映如何给数组，set 和 map做初始化而设计的，所以这段代码会输出：

```
"red"
"green"
"blue"
1234
5678
9012
["title", "Understanding ECMAScript 6"]
["format", "print"]
```

数组和 set 默认返回每一项的值，而 map 则返回每一项（可以再次直接传给 Map 构造函数）。另一方面，weak set 和 weak map 没有内置的迭代器。使用弱引用就意味着没有办法可以确切的获知集合中究竟有多少项，于是迭代它们也是不可能的。

<br />

> #### 解构与 for-of 循环（Destructuring and for-of Loops）

> Map 构造函数的默认行为有助于在 for-of 循环中使用解构。如下所示：

```
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

// 等效于调用 data.entries()
for (let [key, value] of data) {
    console.log(key + "=" + value);
}
```

> 该例中的 for-of 循环在每次迭代都使用了数组解构来获取键和值。在该形式下，你可以轻松地同时使用键和值，而不需操作一个含有两个元素的数组或重新返回 map 并手动获取键和值。对 map 使用解构让 for-of 能平等地对待所有集合类型。

<br />

#### 字符串迭代器（String Iterators）


在 ECMAScript 5 发布之后，字符串就慢慢的变的越来越像数组。例如 ECMAScript 5 正式对字符串启用了方括号语法来访问字符（例如，text[0] 可以获得该字符串中的首个字符，等等）。不过实际上，方括号语法访问的是编码单元（code unit）而非字符本身，所以当获取双字节字符时会有意想不到的结果，如下例所示：

```
var message = "A ð ®· B";

for (let i=0; i < message.length; i++) {
    console.log(message[i]);
}
```

这段代码使用方括号和 length 属性来迭代和打印一个包含 Unicode 字符的字符串，输出的结果有些出乎意料：

```
A
(blank)
(blank)
(blank)
(blank)
B
```

因为双字节字符被当作两个编码单元对待，所以输出的结果中 A 与 B 之间会有四个空行。

幸好，ECMAScript 6 的目标是完全支持 Unicode（查看第二章），所以字符串的默认迭代器就是为了解决字符的迭代问题而做的努力。于是，字符串默认迭代器作用的是字符本身而非编码单元。将上例中的循环重构为 for-of 会得出更合适的结果。下面是修改过的代码：

```
var message = "A ð ®· B";

for (let c of message) {
    console.log(c);
}
```

输出的内容如下：

```
A
(blank)
ð ®·
(blank)
B
```

这个字符串迭代的结果更符合你的预期：循环会正确的打印 Unicode 和其它字符。

<br />

#### NodeList 的迭代器（NodeList Iterators）


文档对象模型（Document Object Model, DOM）中包含了一个 NodeList 类型用来表示一些 DOM 元素的集合。对于那些面向浏览器编程的 JavaScript 开发者来讲，了解 NodeList 对象和 NodeList 数组之间的区别有些棘手。它们都含有代表项数目的 lengh 属性；都可以使用方括号来访问单独的项。然而在内部实现上，NodeList 数组的表现有些不同，导致一些困惑出现。

在 ECMAScript 6 添加了默认迭代器之后，关于 NodeList DOM 规范（实际上它是由 HTML 规范定义而非 ECMAScript 6）中也添加了默认迭代器，而且它和数组的默认迭代器行为是一致的。这意味着你可以使用 for-of 循环或在任何对象默认迭代器的内部来迭代 NodeList。例如：

```
var divs = document.getElementsByTagName("div");

for (let div of divs) {
    console.log(div.id);
}
```

该段代码调用 getElementsByTagName() 来获取一个包含 document 对象中所有 <div> 元素的 NodeList。之后 for-of 循环会像迭代一个标准数组一样获取每一个元素并输出元素的 ID。

<br />

### <a id="The-Spread-Operator-and-Non-Array-Iterables"> 扩展运算符与非数组可迭代类型（The Spread Operator and Non-Array Iterables） </a>


首先回顾一下在第七章中讨论过的使用扩展运算符将 set 转换为数组的示例：

```
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5]
```

这段代码对数组字面量使用扩展运算符以向其填充 set 中的元素。扩展运算符可以和任意的可迭代类型搭配并使用默认的迭代器来决定包含哪些值。迭代器会按顺序返回数据集中所有的项并依次插入到数组当中。该例中由于 set 是可迭代类型所以代码能正常运行，不过使用其它迭代类型也无可厚非，例如：

```
let map = new Map([ ["name", "Nicholas"], ["age", 25]]),
    array = [...map];

console.log(array);         // [ ["name", "Nicholas"], ["age", 25]]
```

在这里，扩展运算符将 map 转换为包含数组的数组。由于 map 默认的迭代器返回的是键值对，所以转换之后的数组从形式上看和传入 new Map() 中的参数相同。

你可以不限次数的在数组字面量内使用扩展运算符，而且你可以随时将多个项插入到可迭代对象中。项的位置会根据插入的顺序决定，例如：

```
let smallNumbers = [1, 2, 3],
    bigNumbers = [100, 101, 102],
    allNumbers = [0, ...smallNumbers, ...bigNumbers];

console.log(allNumbers.length);     // 7
console.log(allNumbers);    // [0, 1, 2, 3, 100, 101, 102]
```

在创建 allNumbers 的时候对 smallNumbers 和 bigNumbers 使用了扩展运算符。allNumbers 中的元素按照创建时插入数组的顺序排列：0 在开头，接着是 smallNumbers 中的项，紧随着的是 bigNumbers 的元素。原始数组并未发生改变，allNumbers 只是复制了它们的元素。

既然扩展运算符可以用在任意的可迭代类型上，那么它就成为了将可迭代类型转换为数组最简单的办法。你可以将字符串和浏览器中的 NodeList 对象分别转换为包含字符（不是编码单元）或 DOM 节点的数组。

目前你已经明白了迭代器以及 for-of 和扩展运算符的基本工作原理，现在是时候去了解一下关于迭代器更复杂的用法。

<br />

### <a id="Advanced-Iterator-Functionality"> 迭代器高级用法（Advanced Iterator Functionality） </a>


迭代器的基本用法和使用生成器创建它们的便利已经能完成很多的工作了。然而，迭代器仅用来迭代集合中的项就有些大材小用，实际上当它们被用作任务管理时更能体现出迭代器的强大之处。在 ECMAScript 6 的发展过程中，一些独特的想法和模式互相碰撞并激发创作者实现更多的功能。一些附加的用法可能微不足道，但将它们聚集在一起后能实现很多有意思的交互。

<br />

#### 向迭代器传入参数（Passing Arguments to Iterators）

在本章中，所有的示例都使用迭代器的 next() 方法或生成器的 yield 语句来获取相关值，其实你也可以通过迭代器的 next() 方法进行传值。当你向 next() 方法传入参数时，生成器使用该参数作为 yield 语句的值。该特性对于一些高级用法尤其是异步编程至关重要。下面是个基本的例子：

```
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;       // 4 + 2
    yield second + 3;                   // 5 + 3
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next(4));          // "{ value: 6, done: false }"
console.log(iterator.next(5));          // "{ value: 8, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

首次调用的 next() 有些特殊，传给它的任何参数都会被忽略。因为传递给 next() 的参数会作为已返回的 yield 语句的值，那么首次调用传给 next() 的参数必须要在返回首个 yield 语句之前可供访问。显然这是不可能的，所以没有理由给首次调用的 next() 方法传参。

当第二次调用 next() 时，4 作为参数被传入。在生成器函数内部该参数最终会赋值给 first 变量。因为该 yield 语句包含赋值操作，右侧的表达式会在首次调用 next() 时候计算，左侧的表达式会在第二次调用 next() 之后函数继续执行之前求值。因为第二次调用 next() 的时候传入了参数 4，该参数会被赋值给 first，之后函数继续执行。

第二个 yield 使用了首个 yield 语句的结果并进行加法操作，返回的值为 6 。接下来 next() 会被第三次调用，此时 5 被作为参数传入。该值会被赋给 second 变量并在第三个 yield 语句中使用，返回的结果为 8 。

如果考虑生成器函数内部在每次运行时都执行了哪些代码，思路可能会清晰一些。图 8-1 用颜色区分了每次 yield 之前代码的执行情况。

<br />

![](https://leanpub.com/site_images/understandinges6/fg0601.png) 　　　　　　　　　　　　　　　　　　　图 8-1: 生成器内部的代码执行

<br />


黄颜色代表第一次调用 next() 之后生成器内部代码的执行情况。湖蓝色代表的是调用 next(4) 运行的代码。紫色则是 next(5) 调用之后执行的代码。难点在于去理解每个表达式右侧的代码是如何在左侧的代码执行之前就中断的。这使得生成器的调试相比一般函数有些复杂。

目前，你已经见识了当传参给 next() 方法之后 yield 的表现和 return 十分神似。不过，这还不是生成器唯一的运行技巧。你还可以让迭代器抛出一个错误。

<br />

#### 在迭代器中抛出错误（Throwing Errors in Iterators）


不仅是数据，错误条件（error conditions）也能传入给迭代器。通过使用 throw() 方法，迭代器可以选择在某一次迭代抛出错误。这不仅对于异步编程来讲是相当重要的能力，而且生成器也变得更加灵活，因为你可以自行决定到底是返回值还是抛出错误（退出函数执行的两种方式）。你可以通过传递 Error 对象给 throw() 方法来让迭代器继续运行的时候抛出错误。例如：

```
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;       // yield 4 + 2, 之后抛出错误
    yield second + 3;                   // 不会执行
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // 由生成器抛出错误
```

在该例中，前两个 yield 表达式计算一切正常，但调用了 throw() 之后，let second 还未计算就会抛出错误。这种中断代码执行的行为类似于直接抛出错误。唯一的区别是中断的位置可能不同。图 8-2 展示了每一步代码的执行情况。

<br />

![](https://leanpub.com/site_images/understandinges6/fg0602.png)
　　　　　　　　　　　　　　　　　　　图 8-2: 在生成器中抛出错误

<br />

图中，红色的部分表示调用 throw()，红色的星号代表生成器抛出错误的大概位置。前两个 yield 声明被执行后，throw() 被调用，于是在代码执行之前一个错误会被抛出。

到了解了这些之后，你可以在生成器内部使用 try-catch 块来捕捉这些错误：

```
function *createIterator() {
    let first = yield 1;
    let second;

    try {
        second = yield first + 2;       // yield 4 + 2, then throw
    } catch (ex) {
        second = 6;                     // 有错误发生，给 second 赋另外的值
    }
    yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // "{ value: 9, done: false }"
console.log(iterator.next());                   // "{ value: undefined, done: true }"
```

本例中，一个 try-catch 块包裹了第二个 yield 语句。虽然该句执行时没有任何问题，但由于迭代器强制将错误传入块中，而且在抛出错误之前不会有任何代码会执行，于是执行流跳转到了 catch 块内将 second 赋值为 6 ，接着再执行到下一个 yield 语句并返回 9 。

要注意一件有趣的事情发生了：throw() 方法和 next() 方法返回相同形式的结果对象。这是因为生成器内部捕捉到了错误，并继续执行直到下个 yield 的位置并返回 9 。

更好的理解方式是把 next() 和 throw() 都当作迭代器的指令。next() 方法指示迭代器继续执行（或许还会传值），而 throw() 指示迭代器继续执行的同时并抛出错误。至于这些指令如何处理，这就由生成器内部的代码来决定。

next() 和 throw() 方法根据 yield 来控制迭代器内部的执行，其实 return 语句也是可以使用的。不过 return 的行为和在普通函数中不太一样，下一节你会看到它们的差异。

<br />

#### 包含 return 语句的生成器（Generator Return Statements）

既然生成器本质上是函数，你可以使用 return 语句来让它提前执行完毕并针对 next() 的调用来指定一个返回值。在本章的大部分实例中，最后一次在迭代器上调用 next() 会返回 undefined，不过你可以像在普通函数中那样使用 return 语句来指定另外的返回值。生成器会将 return 语句的出现判断为所有的任务已处理完毕，所以 done 属性会被赋值为 true，如果指定了返回值那么它会被赋给 value 属性。下面的示例演示了 return 是怎样让生成器提前执行完毕的：

```
function *createIterator() {
    yield 1;
    return;
    yield 2;
    yield 3;
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

该段代码中，生成器同时使用了 yield 和 return 语句。return 表明已经没有值可供迭代，所以剩余的 yield 语句不会被执行（它们是不可达的）。

你也可以指定一个返回值来赋给结果对象中的 value 属性，例如：

```
function *createIterator() {
    yield 1;
    return 42;
}

let iterator = createIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 42, done: true }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

在这里，第二次调用 next() 方法返回的 value 属性值为 42（同时 done 属性也是第一次为 true）。而第三次调用 next() 时返回的 value 属性又变回了 undefined 。任何指定的返回值只能被结果对象使用一次，之后再次调用 next() 的 value 属性值仍为 undefined 。

<br />

> 扩展运算符和 for-of 会忽略 return 语句的返回值。如果返回对象的 done 为 true，它们就会停止读取 value 属性。然而，当使用生成器代理时 return 会相当有用。

<br />

#### 生成器代理（Delegating Generators）

在某些情况下，将两个迭代器集中到一起会更实用。生成器可以使用 yield 和星号（*）这种特殊形式来代理其它生成器。根据生成器的规范，星号在哪里出现并无要求，只要它的位置在 yield 和生成器函数名的中间即可。如下所示：

```
function *createNumberIterator() {
    yield 1;
    yield 2;
}

function *createColorIterator() {
    yield "red";
    yield "green";
}

function *createCombinedIterator() {
    yield *createNumberIterator();
    yield *createColorIterator();
    yield true;
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: "red", done: false }"
console.log(iterator.next());           // "{ value: "green", done: false }"
console.log(iterator.next());           // "{ value: true, done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

本例中，createCombinedIterator() 生成器先后代理了 createNumberIterator() 和 createColorIteartor() 。从迭代器返回的值来看，它等价于只使用一个迭代器并返回了所有的值。每一次调用 next() 都会由恰当的代理迭代器处理直到 createNumberIterator() 和 createColorIterator() 没有值可供迭代。之后最终的 yield 执行并返回 true 。

生成器代理能让你以最简单的方式进一步使用生成器返回的值，同时在处理复杂的任务时也相当有用。例如：

```
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator(count) {
    for (let i=0; i < count; i++) {
        yield "repeat";
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield *createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

在这里，createCombinedIteartor() 生成器代理了 createNumberIterator() 并将它的返回值赋给 result 变量。createNumberIterator() 包含 return 3 语句，那么它的返回值就是 3 。之后 result 变量作为参数回传入 createRepeatingIterator() 以指示 yield 相同字符串的此书（在本例中是三次）。

注意 3 从未被 next() 方法输出。目前它只存在于 createCombinedIterator() 生成器的内部。不过你可以添加额外的 yield 语句来输出它们。例如：

```
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator(count) {
    for (let i=0; i < count; i++) {
        yield "repeat";
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield result;
    yield *createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next());           // "{ value: 1, done: false }"
console.log(iterator.next());           // "{ value: 2, done: false }"
console.log(iterator.next());           // "{ value: 3, done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: "repeat", done: false }"
console.log(iterator.next());           // "{ value: undefined, done: true }"
```

在这段代码中，显式添加了额外的 yield 语句来输出 createNumberIterator() 生成器的返回值。

实用返回值的生成器代理是一种非常强大的编程范式，允许一些有趣的想法变为现实，特别是与异步操作一同使用的时候。

<br />

> 你可以直接在字符串上使用 yield *（例如 yield * "hello"），字符串会使用默认的迭代器。

<br />

### <a id="Asynchronous-Task-Running"> 运行异步任务（Asynchronous Task Running） </a>


使用生成器进行异步编程有很多的兴奋点。在 JavaScript 中异步编程是把双刃剑：简单的任务很容易实现异步，当任务变得复杂的时候源代码的组织会是个大问题。因为生成器允许你在执行的过程中暂停，使用它们处理异步流程就增加了很多可能性。

异步操作的传统做法是在它结束之后调用回调函数。例如，考虑如下 Node.js 读取文件的代码：

```
let fs = require("fs");

fs.readFile("config.json", function(err, contents) {
    if (err) {
        throw err;
    }

    doSomethingWith(contents);
    console.log("Done");
});
```

fs.readFile() 方法包含 filename 参数和一个回调函数。当该操作完成后，回调函数开始执行。回调函数会检查是否有错误发生，如果没有问题则会处理返回的相应内容。在异步任务简单且有限的情况下，这种实现还算可以，一旦需要嵌套多个回调函数或者处理一大批异步任务时，代码会变得极其复杂。生成器和 yield 正好解决了这个问题。

<br />

#### 一个简单的任务运行器（A Simple Task Runner）


因为 yield 可以中断执行，并在继续运行之前等待 next() 方法的调用，你可以不使用回调函数来实现异步调用。首先，你需要一个函数来调用生成器以便让迭代器开始运行，例如这样：

```
function run(taskDef) {

    // 创建迭代器，使它们可以在别处使用
    let task = taskDef();

    // 任务开始执行
    let result = task.next();

    // 递归函数持续调用 next()
    function step() {

        // 如果任务未完成
        if (!result.done) {
            result = task.next();
            step();
        }
    }

    // 开始递归
    step();

}
```

run() 函数接收一个已定义的任务（生成器函数）作为参数。该函数内部调用生成器来创建迭代器并将其存储在 task 中。task 变量在（step）函数外部，所以它能被其它函数访问；本小节后面我会解释缘由。首次调用 next() 令迭代器开始运行并存储其结果以便之后使用。step() 函数检查 result.done 是否为 false，如果答案为是那么在递归自身之前先调用 next() 方法。每次调用 next() 都会将结果存储在 result 中，它总是会被最新的信息覆盖。首次调用 step() 会开始递归并查看 result.done 以判断是否还有更多的工作要做。

随着 run() 的实现，你可以运行一个带有多个 yield 语句的生成器。例如：

```
run(function*() {
    console.log(1);
    yield;
    console.log(2);
    yield;
    console.log(3);
});
```

该例只是简单地在控制台上输出了三个数字以演示所有 next() 的调用结果。然而，仅仅调用了几次 yield 并不是那么实用。下一步要实现的是给迭代器传值或提取迭代器返回的值。

<br />

#### 附加数据的任务运行器（Task Running With Data）


给任务运行器传入数据最简单的办法是将上一次 yield 返回的值传给下一次调用的 next() 方法。为此你只需传入result.value，如下所示：

```
function run(taskDef) {

    // 创建迭代器，使它们可以在别处使用
    let task = taskDef();

    // 任务开始执行
    let result = task.next();

    // 递归函数持续调用 next()
    function step() {

        // 如果任务未完成
        if (!result.done) {
            result = task.next(result.value);
            step();
        }
    }

    // 开始递归
    step();

}
```

既然 result.value 成为了 next() 的参数，那么在 yield 调用之间传递数据成为了可能，如下：

```
run(function*() {
    let value = yield 1;
    console.log(value);         // 1

    value = yield value + 3;
    console.log(value);         // 4
});
```

该例在控制台上输出了两个值：1 和 4 。1 由 yield 1 而来，因为它在返回之后又被传入并赋值给 value 。4 由变量 value 与 3 做加法运算而来，并将运算结果赋值给 value 。现在数据已经可以在 yield 调用之间流动，你只需一个小小的改变即可进行异步调用。

<br />

#### 异步任务运行器（Asynchronous Task Runner）


以上的例子中实现了在 yield 调用之间反复传递静态数据，但是等待异步处理的过程则有些不同。任务运行器需要明确回调函数如何使用这些数据。既然 yield 表达式会将值返回给任务运行器，就意味着任何函数的调用都必须返回一个值并以某种方式说明该调用是个异步操作，使得任务运行器处于待机状态。

Here’s one way you might signal that a value is an asynchronous operation:

下面是标识包含异步操作的一种方法：

```
function fetchData() {
    return function(callback) {
        callback(null, "Hi!");
    };
}
```

该例的目的是让任何由任务运行器调用的方函数返回另一个函数以供回调函数的执行。fetchData() 函数会返回一个参数为回调函数的函数。当返回的函数被调用后，回调函数和一块额外的数据（"Hi!" 字符串）一起执行。该作为参数的回调函数需要由任务运行器提供以确保回调函数能和当前的迭代器正确交互并执行。虽然 fetchData() 函数是同步的，你可以延迟回调函数的执行以将它改造为异步函数，例如：

```
function fetchData() {
    return function(callback) {
        setTimeout(function() {
            callback(null, "Hi!");
        }, 50);
    };
}
```

该版本的 fetchData() 在调用回调函数之前添加了 50ms 的延迟，目的是为了证实该模式同步和异步代码都可以使用。你只需保证遵循该模式的同时使用 yield 来返回每个要调用的函数即可。

当对函数如何标识自己包含异步操作有深入地了解之后，你可以将任务运行器以上述方式改造。当 result.value 为函数的时候，任务运行器会执行它而不是将它直接返回给 next() 方法。下面是重构的代码：

```
function run(taskDef) {

    // 创建迭代器，使它们可以在别处使用
    let task = taskDef();

    // 任务开始执行
    let result = task.next();

    // 递归函数持续调用 next()
    function step() {

        // 如果任务未完成
        if (!result.done) {
            if (typeof result.value === "function") {
                result.value(function(err, data) {
                    if (err) {
                        result = task.throw(err);
                        return;
                    }

                    result = task.next(data);
                    step();
                });
            } else {
                result = task.next(result.value);
                step();
            }

        }
    }

    // 开始递归
    step();

}
```

当 result.value 是个函数（使用 === 操作符检查），它会和回调函数一起被调用。该回调函数按照 Node.js 的惯例将潜在的错误作为第一个参数（err）并将结果作为第二个参数。如果 err 存在就意味着有错误发生并调用 task.throw() 和传入 error 对象而非调用 task.next() 以便让错误在正确的地方抛出。如果没有错误，那么 data 会传入 task.next() 并存储返回结果给 result。之后 step() 会再次被调用以继续接下来的任务。当 result.value 不是函数时，它会直接传给 next() 方法。

这个新版本的任务运行器已经做好了处理异步任务的准备。为了在 Node.js 中读取文件，你需要创建一个容器来包裹 fs.readFile() 以便让返回的函数和本节开始示例中 fetchData() 调用后的结果更为接近，例如：

```
let fs = require("fs");

function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}
```

readfile() 方法接收 filename 参数，并返回一个内部调用回调函数的函数。该回调函数会直接传给 fs.readFile() 方法并在异步方法结束后执行。你可以像下面这样使用 yield 来运行这个任务：

```
run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

该例在未给主要代码显式书写回调函数的同时实现了异步的 readFile() 操作。除了 yield 之外，该段代码看起来和同步无异。只要包含异步操作的函数和上述 fs.readFile() 接口一致，你就可以使用该例来书写从视觉上认为是同步的逻辑。

当然，这些实例中使用的模式也有不利的一面，因为你无法确定返回函数的函数是否是异步的。不过现在的重点是让你理解任务运行背后的原理。promise 提供了能更完善的办法来安排处理异步任务，而且第十一章会进一步探讨它。

<br />

### <a id="Summary"> 总结（Summary） </a>


迭代器是 ECMAScript 6 非常重要的一部分，同时也是 JavaScript 某些关键元素的核心。从表面上看，迭代器提供了一种简约的办法来使用简单的 API 返回一系列元素。然而 ECMAScript 6 还有更复杂的方式来运用迭代器。

对象使用 Symbol.iterator 这个 symbol 类型来定义默认的迭代器。不论是语言内置的还是开发者自定义的对象都能使用这个 symbol 来定义一个返回迭代器的方法。当对象含有 Symbol.iterator 时，它就是可迭代类型。

for-of 循环使用可迭代类型来循环返回一系列的元素。相比传统的 for 循环，for-of 在迭代上更为简单，因为你无需跟踪索引值和设定循环结束条件。for-of 循环会自动读取迭代器返回的值并在没有多余项可供迭代的情况下退出。

为了能让 for-of 更简单地使用，ECMAScript 6 很多类型都内置了迭代器，包括所有的集合类型 —— 数组，map，set ，它们各自的默认迭代器能方便地访问自身内容。同样字符串也含有默认的迭代器能轻松地返回每个字符（而非编码单元。

扩展运算符可以对可迭代类型使用并且能方便地将它们转换为数组，方式是通过读取迭代器的值来逐个将它们插入。

生成器是一种特殊的函数，能在被调用后自动创建一个迭代器。生成器由星号（*）定义并由 yield 关键字来决定每次调用 next() 方法的返回值。

生成器代理通过在新的生成器中重用已有的生成器来鼓励封装迭代器的工作。你可以使用在另一个生成器内部调用 yield * 而非 yield 来使用已存在的生成器。这些操作能创建可以返回多个迭代器的值的单个迭代器。

或许生成器和迭代器最有趣且令人兴奋的能力是书写整洁有序的异步代码。你可以在使用 yield 等待异步操作完成的同时书写看似同步的代码，而不再需要到处放置回调函数。

<br />
