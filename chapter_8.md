# 迭代器与生成器（Iterators and Generators）


许多编程语言都做了这样的转变：迭代集合中的数据不再使用需要初始化变量并作为索引的 for 循环，转而使用迭代器（iterator）对象来程序化地返回集合中下一位置的项。迭代器使得集合的操作变得更容易，ECMAScript 6 也将其添加到了 JavaScript 当中。当迭代器和数组方法以及新添加的集合类型（如 set 和 map）结合之后，它就成为了高效处理数据的关键，而且该语言中很多部分都有迭代器的身影，例如新添加的 for-of 循环，扩展（...）运算符等。迭代器甚至还能简化异步编程。

本章涵盖了迭代器的许多实践，但首先，了解 JavaScript 添加迭代器的背景和缘由是很重要的。

<br />

### 循环问题（The Loop Problem）


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

### 什么是迭代器（What are Iterators?）


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

### 什么是生成器（What Are Generators?）


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

### 可迭代类型与 for-of（Iterables and for-of）


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

> 本章之后的 “生成器委托” 一节会描述如何使用其它对象中的迭代器。

<br />

现在你已经见识了数组默认迭代器的用法，然而 ECMAScript 6 还内置了许多迭代器使得操作集合中的数据更加轻松。

<br />

### 内置的迭代器（Built-in Iterators）


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

````
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

Fortunately, ECMAScript 6 aims to fully support Unicode (see Chapter 2), and the default string iterator is an attempt to solve the string iteration problem. As such, the default iterator for strings works on characters rather than code units. Changing this example to use the default string iterator with a for-of loop results in more appropriate output. Here’s the tweaked code:

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

### The Spread Operator and Non-Array Iterables

Recall from Chapter 7 that the spread operator (...) can be used to convert a set into an array. For example:

```
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5]
```

This code uses the spread operator inside an array literal to fill in that array with the values from set. The spread operator works on all iterables and uses the default iterator to determine which values to include. All values are read from the iterator and inserted into the array in the order in which values where returned from the iterator. This example works because sets are iterables, but it can work equally well on any iterable. Here’s another example:

```
let map = new Map([ ["name", "Nicholas"], ["age", 25]]),
    array = [...map];

console.log(array);         // [ ["name", "Nicholas"], ["age", 25]]
```

Here, the spread operator converts map into an array of arrays. Since the default iterator for maps returns key-value pairs, the resulting array looks like the array that was passed during the new Map() call.

You can use the spread operator in an array literal as many times as you want, and you can use it wherever you want to insert multiple items from an iterable. Those items will just appear in order in the new array at the location of the spread operator. For example:

```
let smallNumbers = [1, 2, 3],
    bigNumbers = [100, 101, 102],
    allNumbers = [0, ...smallNumbers, ...bigNumbers];

console.log(allNumbers.length);     // 7
console.log(allNumbers);    // [0, 1, 2, 3, 100, 101, 102]
```

The spread operator is used to create allNumbers from the values in smallNumbers and bigNumbers. The values are placed in allNumbers in the same order the arrays are added when allNumbers is created: 0 is first, followed by the values from smallNumbers, followed by the values from bigNumbers. The original arrays are unchanged, though, as their values have just been copied into allNumbers.

Since the spread operator can be used on any iterable, it’s the easiest way to convert an iterable into an array. You can convert strings into arrays of characters (not code units) and NodeList objects in the browser into arrays of nodes.

Now that you understand the basics of how iterators work, including for-of and the spread operator, it’s time to look at some more complex uses of iterators.

<br />

### Advanced Iterator Functionality

You can accomplish a lot with the basic functionality of iterators and the convenience of creating them using generators. However, iterators are much more powerful when used for tasks other than simply iterating over a collection of values. During the development of ECMAScript 6, a lot of unique ideas and patterns emerged that encouraged the creators to add more functionality. Some of those additions are subtle, but when used together, can accomplish some interesting interactions.

<br />

#### Passing Arguments to Iterators

Throughout this chapter, examples have shown iterators passing values out via the next() method or by using yield in a generator. But you can also pass arguments to the iterator through the next() method. When an argument is passed to the next() method, that argument becomes the value of the yield statement inside a generator. This capability is important for more advanced functionality such as asynchronous programming. Here’s a basic example:

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

The first call to next() is a special case where any argument passed to it is lost. Since arguments passed to next() become the values returned by yield, an argument from the first call to next() could only replace the first yield statement in the generator function if it could be accessed before that yield statement. That’s not possible, so there’s no reason to pass an argument the first time next() is called.

On the second call to next(), the value 4 is passed as the argument. The 4 ends up assigned to the variable first inside the generator function. In a yield statement including an assignment, the right side of the expression is evaluated on the first call to next() and the left side is evaluated on the second call to next() before the function continues executing. Since the second call to next() passes in 4, that value is assigned to first and then execution continues.

The second yield uses the result of the first yield and adds two, which means it returns a value of six. When next() is called a third time, the value 5 is passed as an argument. That value is assigned to the variable second and then used in the third yield statement to return 8.

It’s a bit easier to think about what’s happening by considering which code is executing each time execution continues inside the generator function. Figure 8-1 uses colors to show the code being executed before yielding.

<br />

![](https://leanpub.com/site_images/understandinges6/fg0601.png) 　　　　　　　　　　　Figure 8-1: Code execution inside a generator

<br />

The color yellow represents the first call to next() and all the code executed inside of the generator as a result. The color aqua represents the call to next(4) and the code that is executed with that call. The color purple represents the call to next(5) and the code that is executed as a result. The tricky part is how the code on the right side of each expression executes and stops before the left side is executed. This makes debugging complicated generators a bit more involved than debugging regular functions.

So far, you’ve seen that yield can act like return when a value is passed to the next() method. However, that’s not the only execution trick you can do inside a generator. You can also cause iterators throw an error.

<br />

#### Throwing Errors in Iterators

It’s possible to pass not just data into iterators but also error conditions. Iterators can choose to implement a throw() method that instructs the iterator to throw an error when it resumes. This is an important capability for asynchronous programming, but also for flexibility inside generators, where you want to be able to mimic both return values and thrown errors (the two ways of exiting a function). You can pass an error object to throw() that should be thrown when the iterator continues processing. For example:

```
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;       // yield 4 + 2, then throw
    yield second + 3;                   // never is executed
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // error thrown from generator
```

In this example, the first two yield expressions are evaluated as normal, but when throw() is called, an error is thrown before let second is evaluated. This effectively halts code execution similar to directly throwing an error. The only difference is the location in which the error is thrown. Figure 8-2 shows which code is executed at each step.

<br />

![](https://leanpub.com/site_images/understandinges6/fg0602.png)
　　　　　　　　　　　　Figure 8-2: Throwing an error inside a generator
            
<br />

In this figure, the color red represents the code executed when throw() is called, and the red star shows approximately when the error is thrown inside the generator. The first two yield statements are executed, and when throw() is called, an error is thrown before any other code executes.

Knowing this, you can catch such errors inside the generator using a try-catch block:

```
function *createIterator() {
    let first = yield 1;
    let second;

    try {
        second = yield first + 2;       // yield 4 + 2, then throw
    } catch (ex) {
        second = 6;                     // on error, assign a different value
    }
    yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next());                   // "{ value: 1, done: false }"
console.log(iterator.next(4));                  // "{ value: 6, done: false }"
console.log(iterator.throw(new Error("Boom"))); // "{ value: 9, done: false }"
console.log(iterator.next());                   // "{ value: undefined, done:\
 true }"
```

In this example, a try-catch block is wrapped around the second yield statement. While this yield executes without error, the error is thrown before any value can be assigned to second, so the catch block assigns it a value of six. Execution then flows to the next yield and returns nine.

Notice that something interesting happened: the throw() method returned a result object just like the next() method. Because the error was caught inside the generator, code execution continued on to the next yield and returned the next value, 9.

It helps to think of next() and throw() as both being instructions to the iterator. The next() method instructs the iterator to continue executing (possibly with a given value) and throw() instructs the iterator to continue executing by throwing an error. What happens after that point depends on the code inside the generator.

The next() and throw() methods control execution inside an iterator when using yield, but you can also use the return statement. But return works a bit differently than it does in regular functions, as you will see in the next section.

<br />

#### Generator Return Statements

Since generators are functions, you can use the return statement both to exit early and specify a return value for the last call to the next() method. In most examples in this chapter, the last call to next() on an iterator returns undefined, but you can specify an alternate value by using return as you would in any other function. In a generator, return indicates that all processing is done, so the done property is set to true and the value, if provided, becomes the value field. Here’s an example that simply exits early using return:

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

In this code, the generator has a yield statement followed by a return statement. The return indicates that there are no more values to come, and so the rest of the yield statements will not execute (they are unreachable).

You can also specify a return value that will end up in the value field of the returned object. For example:

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

Here, the value 42 is returned in the value field on the second call to the next() method (which is the first time that done is true). The third call to next() returns an object whose value property is once again undefined. Any value you specify with return is only available on the returned object one time before the value field is reset to undefined.

<br />

> The spread operator and for-of ignore any value specified by a return statement. As soon as they see done is true, they stop without reading the value. Iterator return values are helpful, however, when delegating generators.

<br />

#### Delegating Generators

In some cases, combining the values from two iterators into one is useful. Generators can delegate to other generators using a special form of yield with a star (*) character. As with generator definitions, where the star appears doesn’t matter, as long as the star falls between the yield keyword and the generator function name. Here’s an example:

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

In this example, the createCombinedIterator() generator delegates first to createNumberIterator() and then to createColorIterator(). The returned iterator appears, from the outside, to be one consistent iterator that has produced all of the values. Each call to next() is delegated to the appropriate iterator until the iterators created by createNumberIterator() and createColorIterator() are empty. Then the final yield is executed to return true.

Generator delegation also lets you make further use of generator return values. This is the easiest way to access such returned values and can be quite useful in performing complex tasks. For example:

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

Here, the createCombinedIterator() generator delegates to createNumberIterator() and assigns the return value to result. Since createNumberIterator() contains return 3, the returned value is 3. The result variable is then passed to createRepeatingIterator() as an argument indicating how many times to yield the same string (in this case, three times).

Notice that the value 3 was never output from any call to the next() method. Right now, it exists solely inside the createCombinedIterator() generator. But you can output that value as well by adding another yield statement, such as:

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

In this code, the extra yield statement explicitly outputs the returned value from the createNumberIterator() generator.

Generator delegation using the return value is a very powerful paradigm that allows for some very interesting possibilities, especially when used in conjunction with asynchronous operations.

> You can use yield * directly on strings (such as yield * "hello") and the string’s default iterator will be used.

<br />

### Asynchronous Task Running

A lot of the excitement around generators is directly related to asynchronous programming. Asynchronous programming in JavaScript is a double-edged sword: simple tasks are easy to do asynchronously, while complex tasks become an errand in code organization. Since generators allow you to effectively pause code in the middle of execution, they open up a lot of possibilities related to asynchronous processing.

The traditional way to perform asynchronous operations is to call a function that has a callback. For example, consider reading a file from the disk in Node.js:

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

The fs.readFile() method is called with the filename to read and a callback function. When the operation is finished, the callback function is called. The callback checks to see if there’s an error, and if not, processes the returned contents. This works well when you have a small, finite number of asynchronous tasks to complete, but gets complicated when you need to nest callbacks or otherwise sequence a series of asynchronous tasks. This is where generators and yield are helpful.

<br />

#### A Simple Task Runner

Because yield stops execution and waits for the next() method to be called before starting again, you can implement asynchronous calls without managing callbacks. To start, you need a function that can call a generator and start the iterator, such as this:

```
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            result = task.next();
            step();
        }
    }

    // start the process
    step();

}
```

The run() function accepts a task definition (a generator function) as an argument. It calls the generator to create an iterator and stores the iterator in task. The task variable is outside the function so it can be accessed by other functions; I will explain why later in this section. The first call to next() begins the iterator and the result is stored for later use. The step() function checks to see if result.done is false and, if so, calls next() before recursively calling itself. Each call to next() stores the return value in result, which is always overwritten to contain the latest information. The initial call to step() starts the process of looking at the result.done variable to see whether there’s more to do.

With this implementation of run(), you can run a generator containing multiple yield statements, such as:

```
run(function*() {
    console.log(1);
    yield;
    console.log(2);
    yield;
    console.log(3);
});
```

This example just outputs three numbers to the console, which simply shows that all calls to next() are being made. However, just yielding a couple of times isn’t very useful. The next step is to pass values into and out of the iterator.

<br />

#### Task Running With Data

The easiest way to pass data through the task runner is to pass the value specified by yield into the next call to the next() method. To do so, you need only pass result.value, as in this code:

```
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
        if (!result.done) {
            result = task.next(result.value);
            step();
        }
    }

    // start the process
    step();

}
```

Now that result.value is passed to next() as an argument, it’s possible to pass data between yield calls, like this:

```
run(function*() {
    let value = yield 1;
    console.log(value);         // 1

    value = yield value + 3;
    console.log(value);         // 4
});
```

This example outputs two values to the console: 1 and 4. The value 1 comes from yield 1, as the 1 is passed right back into the value variable. The 4 is calculated by adding 3 to value and passing that result back to value. Now that data is flowing between calls to yield, you just need one small change to allow asynchronous calls.

#### Asynchronous Task Runner

The previous example passed static data back and forth between yield calls, but waiting for an asynchronous process is slightly different. The task runner needs to know about callbacks and how to use them. And since yield expressions pass their values into the task runner, that means any function call must return a value that somehow indicates the call is an asynchronous operation that the task runner should wait for.

Here’s one way you might signal that a value is an asynchronous operation:

```
function fetchData() {
    return function(callback) {
        callback(null, "Hi!");
    };
}
```

For the purposes of this example, any function meant to be called by the task runner will return a function that executes a callback. The fetchData() function returns a function that accepts a callback function as an argument. When the returned function is called, it executes the callback function with a single piece of data (the "Hi!" string). The callback argument needs to come from the task runner to ensure executing the callback correctly interacts with the underlying iterator. While the fetchData() function is synchronous, you can easily extend it to be asynchronous by calling the callback with a slight delay, such as:

```
function fetchData() {
    return function(callback) {
        setTimeout(function() {
            callback(null, "Hi!");
        }, 50);
    };
}
```

This version of fetchData() introduces a 50ms delay before calling the callback, demonstrating that this pattern works equally well for synchronous and asynchronous code. You just have to make sure each function that wants to be called using yield follows the same pattern.

With a good understanding of how a function can signal that it’s an asynchronous process, you can modify the task runner to take that fact into account. Anytime result.value is a function, the task runner will execute it instead of just passing that value to the next() method. Here’s the updated code:

```
function run(taskDef) {

    // create the iterator, make available elsewhere
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to keep calling next()
    function step() {

        // if there's more to do
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

    // start the process
    step();

}
```

When result.value is a function (checked with the === operator), it is called with a callback function. That callback function follows the Node.js convention of passing any possible error as the first argument (err) and the result as the second argument. If err is present, then that means an error occurred and task.throw() is called with the error object instead of task.next() so an error is thrown at the correct location. If there is no error, then data is passed into task.next() and the result is stored. Then, step() is called to continue the process. When result.value is not a function, it is directly passed to the next() method.

This new version of the task runner is ready for all asynchronous tasks. To read data from a file in Node.js, you need to create a wrapper around fs.readFile() that returns a function similar to the fetchData() function from the beginning of this section. For example:

```
let fs = require("fs");

function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}
```

The readFile() method accepts a single argument, the filename, and returns a function that calls a callback. The callback is passed directly to the fs.readFile() method, which will execute the callback upon completion. You can then run this task using yield as follows:

```
run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

This example is performing the asynchronous readFile() operation without making any callbacks visible in the main code. Aside from yield, the code looks the same as synchronous code. As long as the functions performing asynchronous operations all conform to the same interface, you can write logic that reads like synchronous code.

Of course, there are downsides to the pattern used in these examples–namely that you can’t always be sure a function that returns a function is asynchronous. For now, though, it’s only important that you understand the theory behind the task running. Using promises offers more powerful ways of scheduling asynchronous tasks, and Chapter 11 covers this topic further.

<br />

### Summary

Iterators are an important part of ECMAScript 6 and are at the root of several key language elements. On the surface, iterators provide a simple way to return a sequence of values using a simple API. However, there are far more complex ways to use iterators in ECMAScript 6.

The Symbol.iterator symbol is used to define default iterators for objects. Both built-in objects and developer-defined objects can use this symbol to provide a method that returns an iterator. When Symbol.iterator is provided on an object, the object is considered an iterable.

The for-of loop uses iterables to return a series of values in a loop. Using for-of is easier than iterating with a traditional for loop because you no longer need to track values and control when the loop ends. The for-of loop automatically reads all values from the iterator until there are no more, and then it exits.

To make for-of easier to use, many values in ECMAScript 6 have default iterators. All the collection types–that is, arrays, maps, and sets–have iterators designed to make their contents easy to access. Strings also have a default iterator, which makes iterating over the characters of the string (rather than the code units) easy.

The spread operator works with any iterable and makes converting iterables into arrays easy, too. The conversion works by reading values from an iterator and inserting them individually into an array.

A generator is a special function that automatically creates an iterator when called. Generator definitions are indicated by a star (*) character and use of the yield keyword to indicate which value to return for each successive call to the next() method.

Generator delegation encourages good encapsulation of iterator behavior by letting you reuse existing generators in new generators. You can use an existing generator inside another generator by calling yield * instead of yield. This process allows you to create an iterator that returns values from multiple iterators.

Perhaps the most interesting and exciting aspect of generators and iterators is the possibility of creating cleaner-looking asynchronous code. Instead of needing to use callbacks everywhere, you can set up code that looks synchronous but in fact uses yield to wait for asynchronous operations to complete.

<br />