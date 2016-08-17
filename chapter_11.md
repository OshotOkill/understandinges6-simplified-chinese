# Promise 与异步编程（Promises and Asynchronous Programming）


JavaScript 的一个强大特性就是它可以轻松地处理异步编程。作为面向互联网设计的语言，JavaScript 从一开始就需要响应一些诸如点击和按键这些用户交互的能力。Node.js 通过使用回调函数来替代事件进一步推广了 JavaScript 的异步编程。随着越来越多的项目开始使用异步编程，事件和回调函数已不能满足开发者的所有需求。因此 Promise 应运而生。

Promise 是异步编程的另一种选择，和其它语言一样，它延迟并在以后执行了某些东西。一个 promise 指定一些稍后执行的代码（正如事件和回调函数一样）并显式地表明该段代码是否执行成功。你可以根据代码执行的成功与否将 promise 串联（chain）起来以便让代码看起来更清晰和容易调试。

为了能更好的理解 promise 的工作原理，首先且重要的是明白它建立在哪些基本概念之上。

<br />

* [异步编程的背景](#Asynchronous-Programming-Background)
* [promise 的基础](#Promise-Basics)
* [promise 的全局 Rejection 处理](#Global-Promise-Rejection-Handling)
* [promises 链](#Chaining-Promises)
* [响应多个 promise](#Responding-to-Multiple-Promises)
* [promise 的继承](#Inheriting-from-Promises)
* [Summary](#Summary)

<br />

### <a id="Asynchronous-Programming-Background"> 异步编程的背景（Asynchronous Programming Background） </a>

JavaScript 的引擎建立在单线程事件轮询（single-threaded event loop）概念之上。单线程意味着一段时间内只能执行一段代码，与 Java 和 C++ 这些允许多段代码同时执行的多线程语言形成了鲜明对比。在基于多线程的软件中，维护并防止能被多段代码同时访问和修改的状态常常是个难题，也是经常制造 bug 的根源之一。

JavaScript 引擎在相同的时间内只能执行一段代码，所以引擎不需要追踪这些可能运行的代码，而是在它们准备好执行时将它们放置到任务队列（job queue）。当代码由 JavaScript 引擎执行完毕后，引擎通过 event loop 找到并执行队列中的下一个任务。event loop 是 JavaScript 引擎内部的线程用来监控代码的执行情况和管理任务队列。需要牢记的是既然它是个队列，那么任务就会由开始到最后的顺序依次执行。

<br />

#### 事件模型（The Event Model）


当一个用户点击了一个按钮或按下一个键盘上的某个按键时，一个事件如 onclick 会被触发。为了响应该交互，或许一个新的任务会被添加到任务队列中。这就是 JavaScript 异步编程中最基本的形式。关于处理事件的代码知道事件发生后才会执行，此时相应的上下文（context）会出现，例如：

```
let button = document.getElementById("my-btn");
button.onclick = function(event) {
    console.log("Clicked");
};
```

在该段代码中，console.log("Clicked"）在按钮被点击之前不会执行。而在点击之后，赋值给 onclick 的代码会被添加到任务队列并在先前的所有任务完成之后执行。

事件在简单的交互下能很好的工作，但串联多个独立的异步调用会很复杂，因为你必须追踪每个事件中的作用对象（在上例中为 button）。此外，你必须保证所有相应的处理程序在事件第一次发生之间注册完毕。例如，如果按钮在 onclick 事件注册之前就被点击，那么什么事情都不会发生。于是虽然事件可用于响应用户的交互和一些相似但不常见的目的，但它们面对更复杂的需求时显得不是那么灵活。

<br />

#### 回调模式（The Callback Pattern）


在 Node.js 诞生后，它通过在编程中广泛使用回调模式来进一步发展异步编程模型。回调模式和事件模型有些相似，因为异步的代码在之后的某一时刻才会执行。然而它们之间的差异在于前者要调用的函数是参数，如下所示：

```
readFile("example.txt", function(err, contents) {
    if (err) {
        throw err;
    }

    console.log(contents);
});
console.log("Hi!");
```

该例使用了 Node.js 中惯例的 error-first（错误在前）回调方式。readFile() 函数读取磁盘中的文件（由第一个参数指定）并在任务完成之后执行回调函数（第二个参数）。如果读取的过程中有错误发生，那么回调函数中的 err 参数是一个 error 对象；否则，contents 参数将以字符串的形式存储文件中的内容。

当使用回调模式时，readFile() 在一开始立即执行，在读取文件的时刻函数中断了运行。这意味着 console.log("Hi!") 虽然在 readFile() 之后，但是它会在 console.log(contents) 之前输出内容。当 readFile() 执行完毕后，回调函数和参数会被添加到任务队列中的末尾，并在队列先前的任务全部执行之后运行。

回调模式比事件更为灵活，因为回调函数更容易将多个调用串联在一起。例如：

```
readFile("example.txt", function(err, contents) {
    if (err) {
        throw err;
    }

    writeFile("example.txt", function(err) {
        if (err) {
            throw err;
        }

        console.log("File was written!");
    });
});
```

该段代码中，调用 readFile() 成功之后又出现了另一个异步调用 —— writeFile()，而且它们使用了相同的错误处理模式。当 readFile() 执行完毕后，回调函数被添加到任务队列并在执行后调用 writeFile() 方法（假设没有错误发生）。之后，writeFile() 执行完成并再次向任务队列添加任务（回调函数）。

该模式使用起来感觉相当不错，不过当回调函数嵌套过多时，你很快就会发现自己陷入了回调地狱（callback hell）。像这样：

```
method1(function(err, result) {

    if (err) {
        throw err;
    }

    method2(function(err, result) {

        if (err) {
            throw err;
        }

        method3(function(err, result) {

            if (err) {
                throw err;
            }

            method4(function(err, result) {

                if (err) {
                    throw err;
                }

                method5(result);
            });

        });

    });

});
```

嵌套过多的方法调用会形成错综复杂的代码，难以阅读和调试。回调方法在实现复杂的功能时同样易发生错误。如果你想让两个异步操作并行执行而且在全部完成之后发送通知呢？如果你想让两个异步操作同时执行但是只接受先完成任务的结果呢？

为了实现这些需求，就要追踪多个回调函数并做一些清理工作。promise 极大地降低了实现它们的困难程度。

<br />

### <a id="Promise-Basics"> promise 的基础（Promise Basics） </a>


promise 是异步操作结果的占位符。函数可以返回一个 promise，而不用订阅一个事件或向函数传递回调参数，像这样

```
// readFile 承诺在之后完成该操作
let promise = readFile("example.txt");
```

在这段代码中，readFile() 会稍后而非立即去读取文件。函数会返回一个 promise 对象来表示异步读取操作以便在之后你可以使用它。确切使用 promise 结果的时机完全取决于 promise 生命周期中的行为。

<br />

#### promise 的生命周期（The Promise Lifecycle）


每个 promise 的生命周期一开始都会处于短暂的挂起状态，表示异步操作仍未完成，即挂起的 promise 被认定是未定的（unsettled）。上例中的 promise 在 readFile() 返回结果之前就是处于挂起状态。一旦异步操作完成，promise 就被认为是已定（settled）的并处于以下的两种状态之一：

1. fulfilled: promise 的异步操作已完成。
2. rejected:  promise 的异步操作未完成，原因可能是发生了错误或其它理由。


内部属性 [[PromiseState]] 会根据 promise 的状态来决定自身的值，如 "pending"，"flfilled"，"rejected"。该属性并未向 promise 对象暴露，所以你无法获取并根据 promise 的状态来进行编程。不过你可以在 promise 所处状态改变之后使用 then() 方法来指定一些操作。

所有的 promise 都包含 then() 方法并接受两个参数。第一个参数是 promise 为 fulfilled 状态下调用的函数，任何于异步操作有关的额外数据都会传给该它。第二个参数是 promise 为 rejected 状态下调用的函数，它会被传入任何与操作未完成有关的数据。

<br />

> 以该种方式实现 then() 方法的对象被称为 thenable 。所有 promise 都是 thenable，但不是所有的的 thenable 都是 promise 。

<br />

then() 中的两个参数都是可选的，所以你可以选择同时对 fulfillment 和 rejection 或其中之一进行监听 。例如，考虑下面调用 then() 的例子：

```
let promise = readFile("example.txt");

promise.then(function(contents) {
    // fulfillment
    console.log(contents);
}, function(err) {
    // rejection
    console.error(err.message);
});

promise.then(function(contents) {
    // fulfillment
    console.log(contents);
});

promise.then(null, function(err) {
    // rejection
    console.error(err.message);
});
```

这三个 then() 调用了同一个 promise 。第一次的调用同时监听了 fulfillment 和 rejection。第二次调用只监听了 fulfillment；错误不会被报告。第三次调用仅监听了 rejection 而忽略了任务完成之后的报告。

promise 也包含一个 catch() 方法，它的行为等效于只传递 rejection 处理（handler）。例如，下面的 catch() 和 then() 调用在功能上是等效的。

```
promise.catch(function(err) {
    // rejection
    console.error(err.message);
});

// 等效于:

promise.then(null, function(err) {
    // rejection
    console.error(err.message);
});
```

then() 和 catch() 的目的是让你组合使用它们以用来正确的处理异步操作。该机制比事件和回调更优的原因是操作究竟是成功还是失败一目了然（事件在出现错误之后不会被触发，而回调函数则总是要查看 error 参数）。只需记住如果你不给 promise 添加 rejection 处理，那么所有的错误都会悄无声息的发生。就算 rejection 处理只负责打印错误，你也最好不要忽略它。

一个 fulfillment 或 rejection 的处理可以在 promise 已定之后被添加到任务队列中。这允许你随时添加 fulfillment 和 rejection 处理并保证它们会被调用。例如：

```
let promise = readFile("example.txt");

// 原始的 fulfillment 处理
promise.then(function(contents) {
    console.log(contents);

    // 新添加的处理
    promise.then(function(contents) {
        console.log(contents);
    });
});
```

这段代码中，fulfillment 又为相同的 promise 添加了另一个 fulfillment 处理，所以该处理会被添加到任务队列并在恰当的时机调用。rejection 也是同理。

<br />

> 会创建一个新的任务并在 promise 可用后执行。不过这些任务会被放置到一个单独的完全针对 promise 的任务队列中。只要你大体上了解任务队列的运行机制，那么这个单独的任务队列的细节对你学习如何使用 promise 来讲没有重要的影响。

<br />

#### 创建未定的 promise（Creating Unsettled Promises）


promise 由 Promise 构造函数创建。该构造函数接收一个参数：包含初始化 promise 代码的执行（executor）函数。该执行函数接收 resolve() 和 reject() 两个参数。resolve() 函数会在执行函数成功运行后发出信号表示该 promise 已经可用，而 reject() 函数代表改执行函数运行失败。

下面的例子以本章之前的示例为参考并使用 promise 实现了 Node.js 中的 readFile() 函数：

```
// Node.js 示例

let fs = require("fs");

function readFile(filename) {
    return new Promise(function(resolve, reject) {

        // 触发异步任务
        fs.readFile(filename, { encoding: "utf8" }, function(err, contents) {

            // 检查错误
            if (err) {
                reject(err);
                return;
            }

            // 读取操作成功
            resolve(contents);

        });
    });
}

let promise = readFile("example.txt");

// 同时监听 fulfillment 和 rejection
promise.then(function(contents) {
    // fulfillment
    console.log(contents);
}, function(err) {
    // rejection
    console.error(err.message);
});
```

该例中，Node.js 原生异步 fs.readFile() 函数的调用被 promise 包裹。执行函数分别向 reject() 和 resolve() 传递了 error 对象和 contents 。

需要注意的是执行函数在 readFile() 被调用后会立即执行。当 resolve() 和 reject() 在执行函数内部被调用后，为了处理这个 promise，一个任务会被放置到任务队列中。该种行为被称为任务调度（job scheduling），如果你曾经使用过 setTimeout() 或 setInterval() 函数，那么你已经对其有所了解。在任务调度中，你向任务队列添加了一个任务并声明：“现在不要执行它，以后再说。”例如，setTimeout() 函数允许你延迟将任务放入队列中的时间：

```
// 500ms 之后将这个函数添加到任务队列
setTimeout(function() {
    console.log("Timeout");
}, 500)

console.log("Hi!");
```

该段代码将任务添加到队列的时间延迟了 500ms 。两段 console.log() 调用会如下输出：

```
Hi!
Timeout
```

归功于这 500ms 延迟，setTimeout() 内部的输出在调用 console.log("Hi!") 之后。

Promise 的行为与上述相似。promise 中的执行函数会在执行流到达下方的源代码之前立即运行。例如：

```
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

console.log("Hi!");
The output for this code is:

Promise
Hi!
```

调用 resolve() 会触发一个异步操作。传递给 then() 或 catch() 的函数会被添加到任务队列并异步执行。如下所示：

```
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

promise.then(function() {
    console.log("Resolved.");
});

console.log("Hi!");
```

该例中的输出为：

```
Promise
Hi!
Resolved
```

注意虽然调用 then() 的位置在 console.log("Hi!") 之前，实际上它并不会立即执行（和执行函数不同）。这是因为 fulfillment 和 rejection 处理总会在执行函数运行完毕后被添加到任务队列的末尾。

<br />

#### 创建已定 promise（Creating Settled Promises）


Promise 构造函数由于其内部执行函数与生俱来的动态特性使得它是创建未定 promise 的最佳方式。如果你想创建一个 promise 来代表已知的单个值呢？若只是简单的将该值传递给 resolve() 函数来调度任务，这样做并没有意义。相反，有额外的两个方法专为这种传递特定值的已定 promise 而生。

<br />

##### 使用 Promise.resolve() （Using Promise.resolve()）


Promise.resolve() 方法接收单个参数并返回一个 fulfilled 状态的 promise。这意味着没有发生任务调度，而且你需要创建一个 fulfillment 处理来提取这个参数值。例如：

```
let promise = Promise.resolve(42);

promise.then(function(value) {
    console.log(value);         // 42
});
```

这段代码创建了一个状态为 fulfilled 的 promise，所以 fulfillment 处理接收的值为 42 。如果 rejection 处理被添加给这个 promise，那么它永远都不会被调用，因为 promise 永远不存在 rejected 状态。

<br />

##### 使用 Promise.reject() （Using Promise.reject()）


你也可以使用 Promise.reject() 方法来创建状态为 rejected 的 promise 。这和上述的 Promise.resolve() 的唯一区别就是创建的 promise 状态为 rejected，如下所示：

```
let promise = Promise.reject(42);

promise.catch(function(value) {
    console.log(value);         // 42
});
```

任何给这个 promise 添加的 rejection 处理都会被调用，而 fulfillment 处理不会做任何工作。

如果你向 Promise.resolve() 或 Promise.reject() 方法传递了一个 promise，那么该 promise 不会做任何修改而直接返回。

<br />

##### 非 promise 的 thenable 对象（Non-Promise Thenables）


Promise.resolve() 和 Promise.reject() 也可以接收非 promise 的 thenable 对象作为参数。在传递它之后，这些方法在调用 then() 之后创建一个新的 promise 。 

一个不属于 promise 的 thenable 指的是包含 then() 方法的对象。该方法接收 resolve 和 reject 作为参数，像这样：

```
let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
};
```

该例中的 thenable 对象除了 then() 方法之外没有任何可以和 promise 相关联的特征。你可以调用 Promise.resolve() 来将这个对象转化成状态为 fulfilled 的 promise：

```
let thenable = {
    then: function(resolve, reject) {
        resolve(42);
    }
};

let p1 = Promise.resolve(thenable);
p1.then(function(value) {
    console.log(value);     // 42
});
```

该例中，Promise.resolve() 调用了 thenable 对象中的 then() 方法，所以 promise 的状态可以被确定。该 thenable 对象中的状态为 fulfilled，因为 then() 方法内部调用了 resolve(42)。之后一个名称为 p1 并处于 fulfilled 状态的新 promise 被创建，同时 thenable 向其传值（42）。最后 p1 的fulfillment 处理会接受这个 42 作为参数。

上例 Promise.resolve() 和 thenable 的使用方式也可以创建一个 rejected 状态的 promise：

```
let thenable = {
    then: function(resolve, reject) {
        reject(42);
    }
};

let p1 = Promise.resolve(thenable);
p1.catch(function(value) {
    console.log(value);     // 42
});
```

该例和上个示例极为相似，唯一的区别是 thenable 对象处于 rejected 状态。当该对象执行 then() 方法之后，一个新的处于 rejected 状态的 promise 被创建并带有 42 这个值。之后该值会出入 p1 中的 rejection 处理。

Proimse.resolve() 和 Promise.reject() 以上述的方式工作以便使你方便的操作非 promise 的 thenable 对象。很多库都在 promise 被引入 ECMAScript 6 之前就使用了 thenable，所以将 thenable 转化为正式 promise 的向后兼容特性对于这些已存在的库来讲至关重要。如果你不确定一个对象是否是 promise，最好的办法就是将该对象传入 Promise.resolve() 或 Promise.reject()（取决于你期望的方式），因为 promise 在这些函数中不会发生任何改变。

<br />

#### 执行错误（Executor Errors）


当执行函数中抛出了错误，promise 的 rejection 处理会被调用。例如：

```
let promise = new Promise(function(resolve, reject) {
    throw new Error("Explosion!");
});

promise.catch(function(error) {
    console.log(error.message);     // "Explosion!"
});
```

该段代码中，执行函数内部抛出了错误。实际上每个执行函数内部都包含一个隐式的 try-catch，因此内部的错误会被捕捉并传入 rejection 处理。该例等效如下：

```
let promise = new Promise(function(resolve, reject) {
    try {
        throw new Error("Explosion!");
    } catch (ex) {
        reject(ex);
    }
});

promise.catch(function(error) {
    console.log(error.message);     // "Explosion!"
});
```

执行函数直接负责捕捉抛出的错误以简化该场景的实现，不过执行函数内部抛出的错误只会在 rejection 处理中显现。否则，该错误就会销声匿迹。这在开发者早期使用 promise 时是个麻烦，不过 JavaScript 环境通过提供 hook（挂钩）捕捉 rejected 状态的 promise 解决了该问题。

<br />

### <a id="Global-Promise-Rejection-Handling"> promise 的全局 Rejection 处理 </a>

promise 最有争议的部分在于如果未提供 rejection 处理，那么 promise 中的错误会悄无声息的发生。有些人认为这是该规范中最大的败笔，因为它是 JavaScript 语言中唯一不会让错误自动浮出水面的场景。

判断 promise 的 rejection 是否被处理不是很直观，这是 promise 的自身设定所导致的。例如，考虑下面的示例：

```
let rejected = Promise.reject(42);

// 在当前，rejected 未被处理

// 一段时间过后...
rejected.catch(function(value) {
    // 现在 rejected 已被处理
    console.log(value);
});
```

你可以随时调用 then() 或 catch() 并在无视 promise 是否已定的情况下正常工作，这导致获知处理 promise 的确切时机变得相当困难。在本例的情况下，promise 立即转变为 rejected 状态但并未马上处理。

虽然下个版本的 ECMAScript 很有可能会解决这个问题 ，但是浏览器和 Node.js 都施行了一些变化以解决开发者的痛点。这些变化并不是 ECMAScript 6 规范的一部分但都是处理 promise 的宝贵工具。 

<br />

#### Node.js 中的 rejection 处理（Node.js Rejection Handling）


在 Node.js 中，process 对象上有两个事件和 promise 的 rejection 处理有关：

* unhandledRejection:  在一次事件轮询中，当一个 promise 处于 rejected 状态却没有 rejection 处理它，该事件会被触发。
* rejectionHandled: 在一次事件轮询之后，如果存在 rejected 状态的 promise 并已被 rejection 处理过，该事件会被触发。


设计这些事件的目的是为了帮助辨识未处理的 rejected 状态的 promise 。

unhandledRejection 事件处理函数会被传入 rejection 的原因（通常是一个 error 对象）和 rejected 对象。以下代码做了演示：

```
let rejected;

process.on("unhandledRejection", function(reason, promise) {
    console.log(reason.message);            // "Explosion!"
    console.log(rejected === promise);      // true
});

rejected = Promise.reject(new Error("Explosion!"));
```

该例创建了一个 rejected 状态的 promise 并传入一个 error 对象以作为参数，同时还注册了一个 unhandledRejection 事件以作监听。事件处理函数接受了一个 error 对象和关联的 promise 作为第一，第二个参数。

rejectionHandled 事件处理函数只接受单个参数，即关联的曾处于 rejected 状态的 promise。如下所示：

```
let rejected;

process.on("rejectionHandled", function(promise) {
    console.log(rejected === promise);              // true
});

rejected = Promise.reject(new Error("Explosion!"));

// 等待 rejection 处理的添加
setTimeout(function() {
    rejected.catch(function(value) {
        console.log(value.message);     // "Explosion!"
    });
}, 1000);
```

在这里，rejectionHandled 事件会在 rejection 处理被调用时触发。如果 rejection 处理直接添加到新创建的处于 rejected 状态的 promise 之后，那么该事件不会被触发。因为 rejected 状态的 promise 的创建和相关 rejection 处理的调用会发生在事件轮询的相同周期内。

为了正确追踪潜在的未处理的 rejection，使用 rejectionHandled 和 unhandledRejection 事件能获取并保留它们的一个清单，并在一段时间之后对其检查。例如：

```
let possiblyUnhandledRejections = new Map();

// 当 rejection 未处理时，将其添加到 map 中
process.on("unhandledRejection", function(reason, promise) {
    possiblyUnhandledRejections.set(promise, reason);
});

process.on("rejectionHandled", function(promise) {
    possiblyUnhandledRejections.delete(promise);
});

setInterval(function() {

    possiblyUnhandledRejections.forEach(function(reason, promise) {
        console.log(reason.message ? reason.message : reason);

        // 处理这些 rejection
        handleRejection(promise, reason);
    });

    possiblyUnhandledRejections.clear();

}, 60000);
```

这只是个简单的未处理 rejection 的追踪器。它使用 map 来存储 promise 和相关的 rejection 理由。每个 promise 都作为键，理由作为其值。每次 rejectionHandled 触发后，处理过的 promise 从 map 中移除。因此，possiblyUnhandledRejections 随着事件的调用而增长或减少。调用 setInterval() 会周期性地检查清单中未处理地 rejection 并向控制台输出它们的相关信息（在实际场景中，你可能会做一些其它的操作来打印或处理这些 rejection）。该例中使用 map 而不是 weak map 的原因是你需要周期性地检查该 map 中 promise 的存在情况，而这是 weak map 不可能做到的。

这些仅是 Node.js 的特性，浏览器实现了相似的机制将未处理的 rejection 通知给开发者。

<br />

#### 浏览器中的 rejection 处理（Browser Rejection Handling）


浏览器同样设置了两个事件以便查找未处理的 rejection 。这些事件由 window 对象触发并等效于 Node.js 的相关实现。

* unhandledrejection: 在一次事件轮询中，当一个 promise 处于 rejected 状态却没有 rejection 处理它，该事件会被触发。
* rejectionhandled: 在一次事件轮询之后，如果存在 rejected 状态的 promise 并已被 rejection 处理过，该事件会被触发。

Node.js 的实现中，事件处理函数的参数是分别传入的，而浏览器中的事件处理函数参数接收一个包含以下属性的 event 对象：

* type: 事件的名称（"unhandledrejection" 或 "rejectionhandled"）。
* promise: 处于 rejected 状态的 promise 对象。
* reason: promise 中的 rejection 值。

浏览器实现的另一处差异是 rejection 的值（reason）两个事件都可以使用。例如：

```
let rejected;

window.onunhandledrejection = function(event) {
    console.log(event.type);                    // "unhandledrejection"
    console.log(event.reason.message);          // "Explosion!"
    console.log(rejected === event.promise);    // true
});

window.onrejectionhandled = function(event) {
    console.log(event.type);                    // "rejectionhandled"
    console.log(event.reason.message);          // "Explosion!"
    console.log(rejected === event.promise);    // true
});

rejected = Promise.reject(new Error("Explosion!"));
```

该段代码以 DOM 0级的写法同时向 onunhandledrejection 和 onrejectionhandled 赋值了事件处理函数（如果你喜欢的话也可以使用 addEventListener("unhandledrejection") 和 addEventListener("rejectionhandled")）。每个事件处理函数接收一个含有 rejected promise 相关信息的 event 对象，其中包括 type，promise 和 reason 属性。

在浏览器中书写追踪未处理 rejection 的代码和 Node.js 很相似：

```
let possiblyUnhandledRejections = new Map();

// 当 rejection 未处理时，将其添加到 map 中
window.onunhandledrejection = function(event) {
    possiblyUnhandledRejections.set(event.promise, event.reason);
};

window.onrejectionhandled = function(event) {
    possiblyUnhandledRejections.delete(event.promise);
};

setInterval(function() {

    possiblyUnhandledRejections.forEach(function(reason, promise) {
        console.log(reason.message ? reason.message : reason);

        // 处理这些 rejection
        handleRejection(promise, reason);
    });

    possiblyUnhandledRejections.clear();

}, 60000);
```

该实现几乎和 Node.js 无异。它们都使用 map 来存储 promise 和对应的 rejection 值并在留作以后检查。唯一的区别在于两者在事件处理函数中提取信息的位置。

处理 promise 的 rejection 可能有些棘手，不过你已经初步了解了 promise 的强大之处。现在是时候迈向下一步来串联使用一些 promise 了。

<br />

### <a id="Chaining-Promises"> promise 链（Chaining Promises） </a>

To this point, promises may seem like little more than an incremental improvement over using some combination of a callback and the setTimeout() function, but there is much more to promises than meets the eye. More specifically, there are a number of ways to chain promises together to accomplish more complex asynchronous behavior.

Each call to then() or catch() actually creates and returns another promise. This second promise is resolved only once the first has been fulfilled or rejected. Consider this example:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);
}).then(function() {
    console.log("Finished");
});
```

The code outputs:

```
42
Finished
```

The call to p1.then() returns a second promise on which then() is called. The second then() fulfillment handler is only called after the first promise has been resolved. If you unchain this example, it looks like this:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = p1.then(function(value) {
    console.log(value);
})

p2.then(function() {
    console.log("Finished");
});
```

In this unchained version of the code, the result of p1.then() is stored in p2, and then p2.then() is called to add the final fulfillment handler. As you might have guessed, the call to p2.then() also returns a promise. This example just doesn’t use that promise.

<br />

#### Catching Errors

Promise chaining allows you to catch errors that may occur in a fulfillment or rejection handler from a previous promise. For example:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    throw new Error("Boom!");
}).catch(function(error) {
    console.log(error.message);     // "Boom!"
});
```

In this code, the fulfillment handler for p1 throws an error. The chained call to the catch() method, which is on a second promise, is able to receive that error through its rejection handler. The same is true if a rejection handler throws an error:

```
let p1 = new Promise(function(resolve, reject) {
    throw new Error("Explosion!");
});

p1.catch(function(error) {
    console.log(error.message);     // "Explosion!"
    throw new Error("Boom!");
}).catch(function(error) {
    console.log(error.message);     // "Boom!"
});
```

Here, the executor throws an error then triggers the p1 promise’s rejection handler. That handler then throws another error that is caught by the second promise’s rejection handler. The chained promise calls are aware of errors in other promises in the chain.

<br />

> Always have a rejection handler at the end of a promise chain to ensure that you can properly handle any errors that may occur.

<br />

#### Returning Values in Promise Chains

Another important aspect of promise chains is the ability to pass data from one promise to the next. You’ve already seen that a value passed to the resolve() handler inside an executor is passed to the fulfillment handler for that promise. You can continue passing data along a chain by specifying a return value from the fulfillment handler. For example:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);         // "42"
    return value + 1;
}).then(function(value) {
    console.log(value);         // "43"
});
```

The fulfillment handler for p1 returns value + 1 when executed. Since value is 42 (from the executor), the fulfillment handler returns 43. That value is then passed to the fulfillment handler of the second promise, which outputs it to the console.

You could do the same thing with the rejection handler. When a rejection handler is called, it may return a value. If it does, that value is used to fulfill the next promise in the chain, like this:

```
let p1 = new Promise(function(resolve, reject) {
    reject(42);
});

p1.catch(function(value) {
    // first fulfillment handler
    console.log(value);         // "42"
    return value + 1;
}).then(function(value) {
    // second fulfillment handler
    console.log(value);         // "43"
});
```

Here, the executor calls reject() with 42. That value is passed into the rejection handler for the promise, where value + 1 is returned. Even though this return value is coming from a rejection handler, it is still used in the fulfillment handler of the next promise in the chain. The failure of one promise can allow recovery of the entire chain if necessary.

<br />

#### Returning Promises in Promise Chains

Returning primitive values from fulfillment and rejection handlers allows passing of data between promises, but what if you return an object? If the object is a promise, then there’s an extra step that’s taken to determine how to proceed. Consider the following example:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

p1.then(function(value) {
    // first fulfillment handler
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // second fulfillment handler
    console.log(value);     // 43
});
```

In this code, p1 schedules a job that resolves to 42. The fulfillment handler for p1 returns p2, a promise already in the resolved state. The second fulfillment handler is called because p2 has been fulfilled. If p2 were rejected, a rejection handler (if present) would be called instead of the second fulfillment handler.

The important thing to recognize about this pattern is that the second fulfillment handler is not added to p2, but rather to a third promise. The second fulfillment handler is therefore attached to that third promise, making the previous example equivalent to this:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = p1.then(function(value) {
    // first fulfillment handler
    console.log(value);     // 42
    return p2;
});

p3.then(function(value) {
    // second fulfillment handler
    console.log(value);     // 43
});
```

Here, it’s clear that the second fulfillment handler is attached to p3 rather than p2. This is a subtle but important distinction, as the second fulfillment handler will not be called if p2 is rejected. For instance:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

p1.then(function(value) {
    // first fulfillment handler
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // second fulfillment handler
    console.log(value);     // never called
});
```

In this example, the second fulfillment handler is never called because p2 is rejected. You could, however, attach a rejection handler instead:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

p1.then(function(value) {
    // first fulfillment handler
    console.log(value);     // 42
    return p2;
}).catch(function(value) {
    // rejection handler
    console.log(value);     // 43
});
```

Here, the rejection handler is called as a result of p2 being rejected. The rejected value 43 from p2 is passed into that rejection handler.

Returning thenables from fulfillment or rejection handlers doesn’t change when the promise executors are executed. The first defined promise will run its executor first, then the second promise executor will run, and so on. Returning thenables simply allows you to define additional responses to the promise results. You defer the execution of fulfillment handlers by creating a new promise within a fulfillment handler. For example:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

p1.then(function(value) {
    console.log(value);     // 42

    // create a new promise
    let p2 = new Promise(function(resolve, reject) {
        resolve(43);
    });

    return p2
}).then(function(value) {
    console.log(value);     // 43
});
```

In this example, a new promise is created within the fulfillment handler for p1. That means the second fulfillment handler won’t execute until after p2 is fulfilled. This pattern is useful when you want to wait until a previous promise has been settled before triggering another promise.

<br />

### <a id="Responding-to-Multiple-Promises"> Responding to Multiple Promises </a>

Up to this point, each example in this chapter has dealt with responding to one promise at a time. Sometimes, however, you’ll want to monitor the progress of multiple promises in order to determine the next action. ECMAScript 6 provides two methods that monitor multiple promises: Promise.all() and Promise.race().

<br />

#### The Promise.all() Method

The Promise.all() method accepts a single argument, which is an iterable (such as an array) of promises to monitor, and returns a promise that is resolved only when every promise in the iterable is resolved. The returned promise is fulfilled when every promise in the iterable is fulfilled, as in this example:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.then(function(value) {
    console.log(Array.isArray(value));  // true
    console.log(value[0]);              // 42
    console.log(value[1]);              // 43
    console.log(value[2]);              // 44
});
```

Each promise here resolves with a number. The call to Promise.all() creates promise p4, which is ultimately fulfilled when promises p1, p2, and p3 are fulfilled. The result passed to the fulfillment handler for p4 is an array containing each resolved value: 42, 43, and 44. The values are stored in the order the promises resolved, so you can match promise results to the promises that resolved to them.

If any promise passed to Promise.all() is rejected, the returned promise is immediately rejected without waiting for the other promises to complete:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.all([p1, p2, p3]);

p4.catch(function(value) {
    console.log(Array.isArray(value))   // false
    console.log(value);                 // 43
});
```

In this example, p2 is rejected with a value of 43. The rejection handler for p4 is called immediately without waiting for p1 or p3 to finish executing (They do still finish executing; p4 just doesn’t wait.)

The rejection handler always receives a single value rather than an array, and the value is the rejection value from the promise that was rejected. In this case, the rejection handler is passed 43 to reflect the rejection from p2.

<br />

#### The Promise.race() Method

The Promise.race() method provides a slightly different take on monitoring multiple promises. This method also accepts an iterable of promises to monitor and returns a promise, but the returned promise is settled as soon as the first promise is settled. Instead of waiting for all promises to be fulfilled like the Promise.all() method, the Promise.race() method returns an appropriate promise as soon as any promise in the array is fulfilled. For example:

```
let p1 = Promise.resolve(42);

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.race([p1, p2, p3]);

p4.then(function(value) {
    console.log(value);     // 42
});
```

In this code, p1 is created as a fulfilled promise while the others schedule jobs. The fulfillment handler for p4 is then called with the value of 42 and ignores the other promises. The promises passed to Promise.race() are truly in a race to see which is settled first. If the first promise to settle is fulfilled, then the returned promise is fulfilled; if the first promise to settle is rejected, then the returned promise is rejected. Here’s an example with a rejection:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = Promise.reject(43);

let p3 = new Promise(function(resolve, reject) {
    resolve(44);
});

let p4 = Promise.race([p1, p2, p3]);

p4.catch(function(value) {
    console.log(value);     // 43
});
```

Here, p4 is rejected because p2 is already in the rejected state when Promise.race() is called. Even though p1 and p3 are fulfilled, those results are ignored because they occur after p2 is rejected.

<br />

### <a id="Inheriting-from-Promises"> Inheriting from Promises </a>

Just like other built-in types, you can use a promise as the base for a derived class. This allows you to define your own variation of promises to extend what built-in promises can do. Suppose, for instance, you’d like to create a promise that can use methods named success() and failure() in addition to the usual then() and catch() methods. You could create that promise type as follows:

```
class MyPromise extends Promise {

    // use default constructor

    success(resolve, reject) {
        return this.then(resolve, reject);
    }

    failure(reject) {
        return this.catch(reject);
    }

}

let promise = new MyPromise(function(resolve, reject) {
    resolve(42);
});

promise.success(function(value) {
    console.log(value);             // 42
}).failure(function(value) {
    console.log(value);
});
```

In this example, MyPromise is derived from Promise and has two additional methods. The success() method mimics resolve() and failure() mimics the reject() method.

Each added method uses this to call the method it mimics. The derived promise functions the same as a built-in promise, except now you can call success() and failure() if you want.

Since static methods are inherited, the MyPromise.resolve() method, the MyPromise.reject() method, the MyPromise.race() method, and the MyPromise.all() method are also present on derived promises. The last two methods behave the same as the built-in methods, but the first two are slightly different.

Both MyPromise.resolve() and MyPromise.reject() will return an instance of MyPromise regardless of the value passed because those methods use the Symbol.species property (covered under in Chapter 9) to determine the type of promise to return. If a built-in promise is passed to either method, the promise will be resolved or rejected, and the method will return a new MyPromise so you can assign fulfillment and rejection handlers. For example:

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = MyPromise.resolve(p1);
p2.success(function(value) {
    console.log(value);         // 42
});

console.log(p2 instanceof MyPromise);   // true
```

Here, p1 is a built-in promise that is passed to the MyPromise.resolve() method. The result, p2, is an instance of MyPromise where the resolved value from p1 is passed into the fulfillment handler.

If an instance of MyPromise is passed to the MyPromise.resolve() or MyPromise.reject() methods, it will just be returned directly without being resolved. In all other ways these two methods behave the same as Promise.resolve() and Promise.reject().

<br />

#### Asynchronous Task Running

In Chapter 8, I introduced generators and showed you how you can use them for asynchronous task running, like this:

```
let fs = require("fs");

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

// Define a function to use with the task runner

function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}

// Run a task

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

There are some pain points to this implementation. First, wrapping every function in a function that returns a function is a bit confusing (even this sentence was confusing). Second, there is no way to distinguish between a function return value intended as a callback for the task runner and a return value that isn’t a callback.

With promises, you can greatly simplify and generalize this process by ensuring that each asynchronous operation returns a promise. That common interface means you can greatly simplify asynchronous code. Here’s one way you could simplify that task runner:

```
let fs = require("fs");

function run(taskDef) {

    // create the iterator
    let task = taskDef();

    // start the task
    let result = task.next();

    // recursive function to iterate through
    (function step() {

        // if there's more to do
        if (!result.done) {

            // resolve to a promise to make it easy
            let promise = Promise.resolve(result.value);
            promise.then(function(value) {
                result = task.next(value);
                step();
            }).catch(function(error) {
                result = task.throw(error);
                step();
            });
        }
    }());
}

// Define a function to use with the task runner

function readFile(filename) {
    return new Promise(function(resolve, reject) {
        fs.readFile(filename, function(err, contents) {
            if (err) {
                reject(err);
            } else {
                resolve(contents);
            }
        });
    });
}

// Run a task

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

In this version of the code, a generic run() function executes a generator to create an iterator. It calls task.next() to start the task and recursively calls step() until the iterator is complete.

Inside the step() function, if there’s more work to do, then result.done is false. At that point, result.value should be a promise, but Promise.resolve() is called just in case the function in question didn’t return a promise. (Remember, Promise.resolve() just passes through any promise passed in and wraps any non-promise in a promise.) Then, a fulfillment handler is added that retrieves the promise value and passes the value back to the iterator. After that, result is assigned to the next yield result before the step() function calls itself.

A rejection handler stores any rejection results in an error object. The task.throw() method passes that error object back into the iterator, and if an error is caught in the task, result is assigned to the next yield result. Finally, step() is called inside catch() to continue.

This run() function can run any generator that uses yield to achieve asynchronous code without exposing promises (or callbacks) to the developer. In fact, since the return value of the function call is always coverted into a promise, the function can even return something other than a promise. That means both synchronous and asynchronous methods work correctly when called using yield, and you never have to check that the return value is a promise.

The only concern is ensuring that asynchronous functions like readFile() return a promise that correctly identifies its state. For Node.js built-in methods, that means you’ll have to convert those methods to return promises instead of using callbacks.

<br />

> #### Future Asynchronous Task Running
> 
At the time of my writing, there is ongoing work around bringing a simpler syntax to asynchronous task running in JavaScript. Work is progressing on an await syntax that would closely mirror the promise-based example in the preceding section. The basic idea is to use a function marked with async instead of a generator and use await instead of yield when calling a function, such as:

```
(async function() {
    let contents = await readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

>The async keyword before function indicates that the function is meant to run in an asynchronous manner. The await keyword signals that the function call to readFile("config.json") should return a promise, and if it doesn’t, the response should be wrapped in a promise. Just as with the implementation of run() in the preceding section, await will throw an error if the promise is rejected and otherwise return the value from the promise. The end result is that you get to write asynchronous code as if it were synchronous without the overhead of managing an iterator-based state machine.

>The await syntax is expected to be finalized in ECMAScript 2017 (ECMAScript 8).

<br />

### <a id="Summary"> Summary </a>

Promises are designed to improve asynchronous programming in JavaScript by giving you more control and composability over asynchronous operations than events and callbacks can. Promises schedule jobs to be added to the JavaScript engine’s job queue for execution later, while a second job queue tracks promise fulfillment and rejection handlers to ensure proper execution.

Promises have three states: pending, fulfilled, and rejected. A promise starts in a pending state and becomes fulfilled on a successful execution or rejected on a failure. In either case, handlers can be added to indicate when a promise is settled. The then() method allows you to assign a fulfillment and rejection handler and the catch() method allows you to assign only a rejection handler.

You can chain promises together in a variety of ways and pass information between them. Each call to then() creates and returns a new promise that is resolved when the previous one is resolved. Such chains can be used to trigger responses to a series of asynchronous events. You can also use Promise.race() and Promise.all() to monitor the progress of multiple promises and respond accordingly.

Asynchronous task running is easier when you combine generators and promises, as promises give a common interface that asynchronous operations can return. You can then use generators and the yield operator to wait for asynchronous responses and respond appropriately.

Most new web APIs are being built on top of promises, and you can expect many more to follow suit in the future.

<br />

