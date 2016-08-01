# 简介

JavaScript 语言的核心特性是由 ECMA-262 标准定义的，而这个标准定义的语言被称为 ECMAScript，你所熟悉的在浏览器或者是在 Node.js 中运行的 JavaScript 其实是 ECMAScript 的一个超集。浏览器及 Node.js 通过额外的对象和方法添加了更多的功能，但是核心部分和 ECMAScript 仍保持一致。 总的来讲 ECMA-262 的持续发展是 JavaScript 获得如此成功不可或缺的要素， 本书涵盖了到目前为止最近的一次针对该语言的主要更新内容： ECMAScript 6 。

<br />

#### ECMASciprt 6 的诞生之路

在2007年，JavaScript 已行至于交叉路口。AjSax 的流行宣告了动态 web 应用时代的到来，然而 JavaScript 自1999年 ECMA-262 发布了第三版（ES3）以后便从未发生变化，于是 TC-39 委员会便承担了发布下一版的任务，收集了大批草案并命名为 ECMAScript 4。ECMAScript 4 的变革范围十分广泛，语言的各个部分都有大大小小的变化。 添加的新特性中包括一些新语法，模块，类，传统的继承方式（classical inheritance），私有对象成员，可选类型注解（optional type annotations），以及其它等等。

ECMAScript 4 的变动之大造成了 TC-39 委员会内部的分歧，一部分成员认为这些更改有些过火了。于是一组来自于雅虎，谷歌和微软的成员便自行撰写了下一代 ECMAScript 的草案，称其为 ECMAScript 3.1 ，其中 “3.1” 代表在已有标准之上的小增集。

ECMAScript 3.1 的语法变动非常少，反而专注于属性特性（property attribute），支持原生 JSON 和在已有的对象之上添加更多方法。虽然早先曾有过 ECMAScript 3.1 与 EMCAScript 4 的融合尝试，不过两者之间巨大的差异和对语言发展方向认识的不同导致尝试失败了。

2008年，JavaScript的缔造者 Brendan Eich 认定 TC-39 应该专注于 ECMAScript 3.1 的标准化，ECMASciprt 4 中主要的语法变化和新特性应该搁置到下一代 ECMAScript 标准化之后。委员会的成员一起努力把 ECMAScript 3.1 和 ECMAScript 4 中的精华部分汇聚在一起，称其为 ECMAScript Harmony 。

ECMAScript 3.1 最终作为 ECMA-262 第五版标准被发布，别名为 ECMAScript 5 。委员会为了在命名上避免和已胎死腹中的 ECMAScript 4 混淆，并未发布该标准。在那之后，以 ECMAScript Harmony 为起始，“harmony” 为精神的下一版标准 ECMAScript 6 的发布工作正式启动。

ECMAScript 6 中所有选定草案完全被标准化的日期在2015年，因此正式被更名为 “ECMAScript 2015”（不过本书仍称其为 ECMAScript 6 ，因为开发者对这个名字更为熟悉）。该标准中新的特性作用范围十分广泛，包括全新的对象类型，模式以及给已有对象添加新的方法等等。 ECMAScript 6 的兴奋点在于所有的变动都是为了解决开发者在开发过程中实际存在的问题。

<br />

#### 关于本书

对 ECMAScript 6 特性的深入了解是所有 JavaScript 开发者提升自身水平的关键。在不久的将来，ECMAScript 6 中包含的新特性会是 JavaScript 应用开发的基础，这也是本书所要阐述的。我希望你们能够通过阅读本书来了解 ECMAScript 6 以便在需要使用的时候快速上手。

<br />

#### 浏览器及 Node.js 兼容性

许多 JavaScript 环境，如浏览器及 Node.js 都正在实现 ECMAScript 6。本书并不关心他们实现的差异性而仅关注在规范中定义的正确行为。因此在你的 JavaScript 环境中，一些行为可能与本书描述的不符。

<br />

#### 本书的适用人群

本书的目的是给那些已经熟悉 JavaScript 和 ECMAScript 5 的人提供教程，但是并不强求读者对该语言有深入的认识，仅仅是帮助了解 ECMAScript 5 和 ECMAScript 6 之间的差异。本书特别针对的是那些想了解这门语言最新特性并在浏览器或 Node.js 里实现的中级或高级开发者。

本书并不适合从未写过 JavaScript 的初学者，你需要一定的基础知识才能通读本书。

<br />

#### 总览

本书共13章，ECMAScript 6 中不同的部分由各自的章节分别阐释。许多章节都是以 ECMAScript 6 是怎样解决过去开发过程中存在的某处痛点开头，目的是为了让你对这些变更有个大体上的认识，此外所有的章节都包含实际的代码示例，以助你学习新的语法及概念。

<br />

第一章： **块级绑定（How Block Bindings Work）**

> 讨论了块级声明 let 和 const —— var 的替代者们。



第二章： **字符串及正则表达式（Strings and Regular Expressions ）**

> 涵盖了新增加的字符串操作和查看方法，以及字符串模板（template strings）等内容。



第三章： **ECMAScript 6 中的函数（Functions in ECMAScript 6）**

> 阐述了在 ECMAScript 6 中函数发生的变化，包括箭头函数，默认参数，剩余参数等。



第四章： **扩展的对象功能（Expanded Object Functionality）**

> 揭示了对象在创建，修改及使用过程中发生的变化，包括对象字面量以及新的反射方法（reflection methods）。


第五章： **解构（Destructuring for Easier Data Access）**

> 介绍了对象和数组的解构方法，允许你用更简洁的语法来分解（decompose）对象和数组。


第六章： **Symbols 与 Symbols属性（Symbols and Symbol Properties）**

> 解释了symbols的概念，这是一种定义属性的新方式。Symbols 是新添加的原始类型，可以用来模糊（并非隐藏）对象的属性和方法。


第七章： **Sets 与 Maps（Sets and Maps）**

> 展示了新的集合类型的细节，包括 Set，WeakSet，Map 和 WeakMap，这些类型在数组的基础之上添加了一组实用的扩展功能，包括添加语义（adding semantics），去重（de-duping）及针对JavaScript的内存管理（memory management）


第八章： **迭代器与生成器（Iterators and Generators）**

> 讨论了迭代器和生成器这两个新添加的特性，它们允许你使用另一种强有力的方式操作集合中的数据，而在 ECMAScript 6 之前的版本中这是绝对无法做到的。


第九章： **类 (Introducing JavaScript Classes)**

> 解释了在JavaScript中首次正式定义的类的概念。类的缺失是使其它语言开发者学习 JavaScript 感到困惑的原因之一，天之后使得 JavaScript 更易理解而且语法更为简洁


第十章： **改进的数组功能（Improved Array Capabilities）**

> 阐释了原生数组的一些变化及新的有趣的使用方式。


第十一章：**Promises 与异步编程（Promises and Asynchronous Programming）*** 

>  Promises 成为了语言的一部分，由底层实现并被广泛且流行的库所支持。ECMAScript 6 原生支持并标准化了 promises 。 
>  

第十二章： **代理与反射API（Proxies and the Reflection API）**

> 介绍了 JavaScript 中正式添加的反射API及新的代理对象（proxy object），使你可以拦截任何针对对象的操作。代理给予开发者操控对象空前的自由度和定义新交互方式的无限可能性。


第十三章： **模块（Encapsulating Code with Modules）**

> 官方正式定义了 JavaScript 中模块的格式，目的是取代这些年涌现的各式各样的模块加载方案。

<br />

附录 A： **其它改进（Smaller ECMAScript 6 Changes）**

> 集中介绍了 ECMAScript 6 中其它不太常见或者内容较少不大适合写为章节的内容。


附录 B： **领悟 ECMAScript 7（2016）（Understanding ECMAScript 7 (2016)）**

> 介绍了 ECMAScript 7（2016）新添加的两项内容，对 JavaScript 的改进相比 ECMAScript 6 甚微

<br />

#### 排版协定

本书会使用以下的排版协定：

* 斜体表示新术语
* 代码或文件名使用等宽字体

另外， 长代码会包含在使用等宽字体的代码块中，如：

```doSomething
function doSomething() {
    // empty
}
```

在代码块中， console.log() 右侧的注释表示代码执行后出现在浏览器或 Node.js 控制台上的输出内容，如

```comment
console.log("Hi");      // "Hi"
```

如果代码块中的某行代码抛出一个错误，右侧同样会有提示：

```error
doSomething();          // error!
```
<br />

#### 协助于勘误

你可以向英文原版提交 issue，建议或者PR： [https://github.com/nzakas/understandinges6](https://github.com/nzakas/understandinges6)

如果你在阅读的过程中抱有疑问，也可以发送邮件给：[http://groups.google.com/group/zakasbooks.](http://groups.google.com/group/zakasbooks.)

<br />

#### 鸣谢

英文原文的贡献者请查看 [原文](https://leanpub.com/understandinges6/read) Introduction 小结末尾

<br />
<br />
\* 代表翻译存在问题









