# 简介

Javascript 语言的核心特性是由 ECMA-262 标准定义的，而这个标准定义的语言被称为 ECMAScript，你所熟悉的在浏览器或者是在 Node.js 中运行的 JavaScript 其实是 ECMAScript 的一个超集。浏览器及 Node.js 通过额外的对象和方法添加了更多的功能，但是核心部分和 ECMAScript 仍保持一致。 总的来讲 ECMA-262 的持续发展是 JavaScript 获得如此成功不可或缺的要素， 本书涵盖了到目前为止最近的一次针对该语言的主要更新内容： ECMAScript 6 。


#### ECMASciprt 6 的诞生之路

在2007年，JavaScript 已行至于交叉路口。Ajax 的流行宣告了动态 web 应用时代的到来，然而 JavaScript 自1999年 ECMA-262 发布了第三版（ES3）以后便从未发生变化，于是 TC-39 委员会便承担了发布下一版的任务，收集了大批草案并命名为 ECMAScript 4。ECMAScript 4 的变革范围十分广泛，语言的各个部分都有大大小小的变化。 添加的新特性中包括一些新语法，模块，类，传统的继承方式（classical inheritance），私有对象成员，可选类型注解（optional type annotations），以及其它等等。

ECMAScript 4 的变动之大造成了 TC-39 委员会内部的分歧，一部分成员认为这些更改有些过火了。于是一组来自于雅虎，谷歌和微软的成员便自行撰写了下一代 ECMAScript 的草案，称其为 ECMAScript 3.1 ，其中 “3.1” 代表在已有标准之上的小增集。

ECMAScript 3.1 的语法变动非常少，反而专注于属性特性（property attribute），支持原生 JSON 和在已有的对象之上添加更多方法。虽然早先曾有过 ECMAScript 3.1 与 EMCAScript 4 的融合尝试，不过两者之间巨大的差异和对语言发展方向认识的不同导致尝试失败了。

2008年，JavaScript的缔造者 Brendan Eich 认定 TC-39 应该专注于 ECMAScript 3.1 的标准化，ECMASciprt 4 中主要的语法变化和新特性应该搁置到下一代 ECMAScript 标准化之后。委员会的成员一起努力把 ECMAScript 3.1 和 ECMAScript 4 中的精华部分汇聚在一起，称其为 ECMAScript Harmony 。

ECMAScript 3.1 最终作为 ECMA-262 第五版标准被发布，别名为 ECMAScript 5 。委员会为了在命名上避免和已胎死腹中的 ECMAScript 4 混淆，并未发布该标准。在那之后，以 ECMAScript Harmony 为起始，“harmony” 为精神的下一版标准 ECMAScript 6 的发布工作正式启动。

ECMAScript 6 中所有选定草案完全被标准化的日期在2015年，因此正式被更名为 “ECMAScript 2015”（不过本书仍称其为 ECMAScript 6 ，因为开发者对这个名字更为熟悉）。该标准中新的特性作用范围十分广泛，包括全新的对象类型，模式以及给已有对象添加新的方法等等。 ECMAScript 6 的兴奋点在于所有的变动都是为了解决开发者在开发过程中实际存在的问题。

#### 关于本书

对 ECMAScript 6 特性的深入了解是所有 JavaScript 开发者提升自身水平的关键。在不久的将来，ECMAScript 6 中包含的新特性会是 JavaScript 应用开发的基础，这也是本书所要阐述的。我希望你们能够通过阅读本书来了解 ECMAScript 6 以便在需要使用的时候快速上手。

#### 浏览器及 Node.js 兼容性

许多 JavaScript 环境，如浏览器及 Node.js 都正在实现 ECMAScript 6。本书并不关心他们实现的差异性而仅关注注在规范中定义的正确行为。因此在你的 JavaScript 环境中，一些行为可能与本书描述的不符。

#### 本书的适用人群

本书的目的是给那些已经熟悉 JavaScript 和 ECMAScript 5 的人提供教程，但是并不强求读者对该语言有深入的认识，仅仅是帮助了解 ECMAScript 5 和 ECMAScript 6 之间的差异。本书特别针对的是那些想了解这门语言最新特性并在浏览器或 Node.js 里实现的中级或高级开发者。

本书并不适合从未写过 JavaScript 的初学者，你需要一定的基础知识才能通读本书。









