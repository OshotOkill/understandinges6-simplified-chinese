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


每个 promise 的生命周期一开始都会处于短暂的挂起（pending）状态，表示异步操作仍未完成，即挂起的 promise 被认定是未定的（unsettled）。上例中的 promise 在 readFile() 返回结果之前就是处于挂起状态。一旦异步操作完成，promise 就被认为是已定（settled）的并处于以下的两种状态之一：

1. fulfilled: promise 的异步操作已完成。
2. rejected:  promise 的异步操作未完成，原因可能是发生了错误或其它理由。


内部属性 [[PromiseState]] 会根据 promise 的状态来决定自身的值，如 "pending"，"fulfilled" *，"rejected"。该属性并未向 promise 对象暴露，所以你无法获取并根据 promise 的状态来进行编程。不过你可以在 promise 所处状态改变之后使用 then() 方法来指定一些操作。

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

> 每次调用 then() 和 catch() 都会创建一个新的任务并在 promise 可用后执行。不过这些任务会被放置到一个单独的完全针对 promise 的任务队列中。只要你大体上了解任务队列的运行机制，那么这个单独的任务队列的细节对你学习如何使用 promise 来讲没有重要的影响。

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

这段代码创建了一个状态为 fulfilled 的 promise，所以 fulfillment 处理接收的值为 42 。如果 rejection 处理被添加给这个 promise，那么它永远都不会被调用，因为 promise 不存在 rejected 状态。

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


目前来看，promise 仅仅是回调和 setTimeout() 函数的混合和改进，实际上 proimse 还有很多能力未呈现出来。更确切地讲，有很多种方法通过串联 promise 来完成更复杂地异步操作。

实际上每一次调用 then() 和 catch() 都会返回另一个 promise 。它只会在之前的 promise 转化为 fulfilled 或 rejected 状态的那一刻后才会被处理 。考虑下面的示例：

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

该段代码输出:

```
42
Finished
```

调用 p1.then() 返回另一个 promise，而且该例又对新的 promise 调用了 then()。第二次调用 then() 后 fulfillment 处理函数只有在第一个 promise 完成之后被调用。如果你不使用链式调用，那么看起来像是这样：

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

在未使用链式代码的版本中，p1.then() 的结果存储到了 p2 中，p2.then() 被调用后最后的 fulfillment 处理才会被添加。正如你所想的那样，p2.then() 也会返回一个 promise，只是该例没有进一步使用它。

<br />

#### 捕获错误（Catching Errors）


promise 链允许你捕获上一个 promise 的 fulfillment 或 rejection 处理中的错误。例如：

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

在这段代码中，p1 的 fulfillment 处理抛出了错误。该链中的第二个 promise 调用了 catch() 方法，所以它通过 rejection 处理接收了这个错误。同样上一个 promise 如果在 rejection 处理中抛出了错误，这里同样也能接收：

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

这里，执行函数抛出了错误并触发了 p1 promise 的 rejection 处理。这个处理函数抛出了另一个错误又被下一个 promise 的 rejection 处理捕获。串联的 promise 调用能知晓链中其它 promise 的错误。

<br />

> 为了确保你能正确处理可能发生的错误，你总是需要在 promise 链的末尾添加一个 rejection 处理。

<br />

#### promise 链中的返回值（Returning Values in Promise Chains）


promise 链的另一个重要特征是链中的 promise 能够向下一个 promise 传递数据。你已经知道执行函数中的 resolve() 的参数会被传递给 promise 的 fulfillment 处理函数。你同样可以在 fulfillment 处理中通过返回某个指定值来在链中传递数据。例如：

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

p1 的 fulfillment 处理会返回 value + 1 的计算值。因为 value 的值是 42（由执行函数所得），所以 fulfillment 函数会返回 43 。该值会被传递给下一个 promise 的 fulfillment 处理并输出到控制台。

你可以用 rejection 处理做相同的事情。当 rejection 处理被调用后，也可以返回一个值。如果这么做，那么该值同样会传递给下一个 promise 的 fulfillment 处理，像这样：

```
let p1 = new Promise(function(resolve, reject) {
    reject(42);
});

p1.catch(function(value) {
    // 首个 fulfillment 处理
    console.log(value);         // "42"
    return value + 1;
}).then(function(value) {
    // 第二个 fulfillment 处理
    console.log(value);         // "43"
});
```

这在里，执行函数调用了 reject() 并传入 42 。它被传给 promise 的 rejection 处理，并返回 value + 1 的值。尽管这个返回值来自于 rejection 处理，他仍然会被链中的下一个 promise 的 fulfillment 处理使用。如果链中的某个 promise 失败，必要的话，可以通过上述方法来恢复整个 promise 链。

<br />

#### promise 链中的 promise 返回（Returning Promises in Promise Chains）


fulfillment 和 rejection 处理返回的原始值允许在 promise 中传递数据。如果你想返回一个对象呢？假设这个对象是 promise，那么为了决定下一步该做些什么，这里需要额外的步骤。考虑下面的示例：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

p1.then(function(value) {
    // 首个 fulfillment 处理
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // 第二个 fulfillment 处理
    console.log(value);     // 43
});
```

该段代码中，p1 安排了一个 resolve(42) 的任务。p1 的 fulfillment 处理返回了 p2 这个包含 resolve() 的 promise 。因为 p2 已经处于 fulfilled 状态，所以第二个 fulfillment 处理会被调用。相反，如果 p2 的状态是 rejected，那么 rejection 处理（如果存在）会被调用。

认识该模式重要的一点是第二个 fulfillment 处理并没有添加在 p2，而是第三个 promise 上。因此该 fulfillment 处理附着于第三个 promise，使得上例等效于：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

let p3 = p1.then(function(value) {
    // 首个 fulfillment 处理
    console.log(value);     // 42
    return p2;
});

p3.then(function(value) {
    // 第二个 fulfillment 处理
    console.log(value);     // 43
});
```

这里可以很明显的看到，第二个 fulfillment 处理添加给了 p3 而不是 p2 。这个区别虽不易察觉但至关重要，因为 p2 若处于 rejected 状态则第二个 fulfillment 处理不会被调用。例如：

```
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    reject(43);
});

p1.then(function(value) {
    // 首个 fulfillment 处理
    console.log(value);     // 42
    return p2;
}).then(function(value) {
    // 第二个 fulfillment 处理
    console.log(value);     // 不会被调用
});
```

该例中，因为 p2 的状态是 rejected，所以第二个 fulfillment 处理永远不会被调用。不过，你可以添加一个 rejection 处理：

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

在这里，p2 处于 rejected 状态并调用 rejection 处理。p2 中的 reject() 参数（43）会被传递给 rejection 处理。

在 fulfillment 或 rejeciton 处理中返回 thenable 对象并不会改变它们内部执行函数的行为。首个定义的 promise 会最先运行它的执行函数，接下来是第二个定义的 promise，以此类推。返回 thenable 对象仅允许你为这些 promise 定义额外的相应操作。你可以在 fulfillment 处理中创建一个新的 promise 来延迟 fulfillment 处理的执行。例如：

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

该例中，一个新的 promise 在 p1 的 fulfillment 处理函数中创建。这意味着第二个 fulfillment 处理直到 p2 处于 fulfilled 状态之前都不会执行。如果你想在前一个 promise 已定之前不想触发其它 promise，那么该模式十分有用。

<br />

### <a id="Responding-to-Multiple-Promises"> 响应多个 promise（Responding to Multiple Promises） </a>


到目前为止，本章中的每个示例在同一时间内都只响应了一个 promise。不过有时，你想要观察多个 promise 的进度来决定下一步的操作。ECMAScript 6 提供了两个方法负责此事：Promise.all() 和 Promise.race()

<br />

#### Promise.all() 方法（The Promise.all() Method）


Promise.all() 方法接收单个包含 promise 的可迭代对象参数（如数组），并在该对象包含的所有 promise 全部处理完毕之后返回一个已处理的 promise 。这个返回的 promise 会在所有 promise 处于 fulfilled 状态之后转变为该状态，如下所示：

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

这里的每个 promise 都带有一个数字。调用 Promise.all() 会创建 p4 promise，并在 p1，p2，p3 fulfilled 之后转变状态为 fulfilled 。传给 p4 的 fulfillment 处理的参数是一个包含所有已处理 promise 的值：42，43，44 的数组。这些值会在各个 promise 处理完成后依次存储到相应的变量中，所以你可以根据 promise 的处理结果来找出对应的 promise。

如果 Promise.all() 中的某个 promise 转变为 rejected 状态，那么会立即返回一个 rejected 状态的 promise 而不用等待其它 promise 完成执行。

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

在本例中，p2 处于 rejected 状态并返回 43 。p4 的 rejection 处理会被立即调用而无需等待 p1 或 p3 执行完毕（它们仍旧会完成执行，只是 p4 不等它们）。

rejection 处理总是会接收单个值，而不是数组，并且该值是 rejected 状态的 promise 所返回的。在本例的情况下，rejection 处理接收的参数为 p2 传递的 43 。

<br />

#### Promise.race() 方法（The Promise.race() Method）


Promise.race() 方法以另一种稍稍不同的方式来观察多个 promise 。该方法同样接收一个包含 promise 的可迭代类型并返回一个 promise，不过返回的时机是在单个 promise 执行完毕的那一刻，而不是像 Promise.all() 那样需要等待所有的 promise 都处于 fulfilled 状态。只要有 promise 转变为 fulfilled 状态，那么Promise.race() 就会返回它。例如：

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

在该段代码中，p1 是以 fulfilled promise 的身份所创建，而其它的 promise 需要做任务调度。p4 的 fulfillment 处理会被立即调用并传入值 42，其它的 promise 都被忽略。传给 Promise.race() 的 promise 真如竞赛一般看哪一个先处于已定状态，并返回胜出的 fulfilled promise：

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

这里，p2 在 Promise.race() 调用时已经处于 rejected 状态，所以 p4 的状态也是 rejected。虽然 p1 和 p3 处于 fulfilled 状态，但由于 p2 的原因它们被忽略了。

<br />

### <a id="Inheriting-from-Promises"> promise 继承（Promise Inheriting from Promises） </a>


和其它内置类型相似，你可以讲 promise 作为派生类的基类。这允许你以内置的 promise 为基础做一些改进。假如，你想创建一个包含 success() 和 failure() 方法的 promise 但又不想丢掉内置版本中的 then() 和 catch()，你可以如下创建该 promise 类型：

```
class MyPromise extends Promise {

    // 使用默认的构造函数

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

该例中，MyPromise 由 promise 派生并包含了两个额外方法。success() 和 failure() 方法分别模仿了 resolve() 与 reject() 。

success() 和 reject() 使用 this 来调用它们模仿的方法。派生的 promise 和内置的 promise 基本一致，除了前者可以调用 success() 和 failure() 。

既然静态方法也会被继承，那么 MyPromise.resolve()，MyPromise.reject()，MyPromise.race() 和 MyPromise.all() 方法同样在派生类内部存在。后两者和原版表现一直，而前两者有些不同。

MyPromise.resolve() 和 MyPromise.reject() 会返回 MyPromise 的示例而无视传给它们的参数，因为这些方法使用了 Symbol.species 属性（第九章已讨论）来决定 promise 返回的类型。如果一个属于内置类型的 promise 被传递给这些方法，该 promise 会被进行处理，并返回了一个新的 MyPromise 以供你赋值 fulfillment 和 rejection 处理。例如：

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

这里，p1 是内置的 promise 类型并被传递给 MyPromise.resolve() 方法。执行的结果是 p2 也是 MyPromise 类型并在 fulfillment 处理中接收 p1 的参数。

如果 MyPromise 的实例直接传递给 MyPromise.resolve() 或 MyPromise.reject() 方法，它们会直接返回而不需要被处理。在其它方面这两个方法与 Promise.resolve() 及 Promise.reject() 无异。

<br />

#### 运行异步任务（Asynchronous Task Running）


在第八章，我介绍了生成器并演示了如何使用它来运行异步任务，像这样“

```
let fs = require("fs");

function run(taskDef) {

    // 创建迭代器，使其在作用域其它部分可用。
    let task = taskDef();

    // 任务开始
    let result = task.next();

    // 递归函数并持续调用 next()
    function step() {

        // 如果还有工作要做
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

// 定义任务运行器需要的函数

function readFile(filename) {
    return function(callback) {
        fs.readFile(filename, callback);
    };
}

// 运行一个任务

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

该实现有一些弊端。首先，在一个函数中包裹所有相关的函数并返回一个函数很令人困惑（甚至这个句子本身读起来都让人困惑）。其次，没有任何办法来区分函数返回的值究竟是否接收回调函数作为参数。

在 promise 的帮助下，你可以通过判断这些异步操作是否为 promise 来大幅度简化和泛化任务运行器。通用的接口意味着你可以减少大段的异步代码。下面是一种简化任务运行器的方式：

```
let fs = require("fs");

function run(taskDef) {

    // 创建迭代器
    let task = taskDef();

    // 开始任务
    let result = task.next();

    // 使用函数递归进行迭代
    (function step() {

        // 如果还有工作要做
        if (!result.done) {

            // 使用 resolve() 来简化 promise 的处理
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

// 定义任务运行器需要的函数

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

// 运行一个任务

run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

在这个版本的代码中，一个通用的 run() 函数通过执行生成器来创建迭代器。它调用 task.next() 让任务开始进行并递归调用 step() 直到迭代器运行完毕。

在 step() 内部，如果工作还有剩余，result.done 为 false。此时，result.value 应该是个 promise，不过 调用 Promise.resolve() 的目的是预防未返回 promise 的函数。（记住，Promise.resolve() 只是让 promise 从内部通过而不做任何操作，而一个非 promise 类型会被包裹为 promise）。之后，一个 fulfillment 处理被添加并提取 promise 值来返回给迭代器。接着，在 step() 函数调用自身之前，result 会重新由下一个 yield 的返回结果赋值。

rejection 处理会存储任务失败而创建的 error 对象。task.throw() 方法将 error 对象返回给迭代器 *，同时在任务中如果捕获了某个错误，result 会被下一个 yield 的返回结果赋值。最后，在 catch 内部调用 step() 来继续执行任务。

run() 函数可以运行任何使用 yield 操作异步代码的生成器，同时也没有向开发者暴露 promise（或回调函数）。事实上，由于函数调用后的返回值总是被转换为 promise，该函数甚至可以返回其它类型。这意味着同步和异步的方法都可以由 yield 来调用，而且你永远不需要检查返回值是否为 promise 。

唯一需要注意的是确保异步函数如 readFile() 返回一个能正确标识状态的 promise。对于 Node.js 内置的方法来讲，意味着你必须将它们转化为返回 promise 而不是使用回调函数的方法。

<br />

> #### 未来的异步任务运行器（Future Asynchronous Task Running）

> 在我写这本书的时候，JavaScript 正准备引入一个新的简化语法来执行异步任务。该种实现是使用 await 语法并能完美作用于上述以 promise 为基础的示例。它的基本理念是使用 async 标记的函数和 await 而不是生成器与 yield 来调用函数，例如：

```
(async function() {
    let contents = await readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```

> 函数之前的 async 关键字标识它会执行一些异步操作。await 关键字指示 readFile("config.json") 应该返回一个 promise，如若不是，该返回值应该由 promise 包裹。和上述示例一样，如果 promise 是 rejected 状态，那么 await 会抛出错误，不然它将返回 promise 的值。这种写法的结果是你可以使用同步的语法来书写异步代码并不需要复杂的以迭代器为核心的状态机。

> await 语法有望在 ECMAScript 2017（ECMAScript 8）中正式采用。（译者：已经被纳入ES8）

<br />

### <a id="Summary"> 总结（Summary） </a>

JavaScript 引入并提供给 promise 更甚于事件和回调的控制性与组合性来提升异步编程的体验。JavaScript 引擎将 promise 添加给任务队列并通过任务调度来它们延期执行，同时另一个任务队列追踪 promise 的 fulfillment 和 rejection 以确保这些处理运行无误。

Promise 存在三种状态：挂起，fulfilled 和 rejected 。一个 promise 首先处于挂起状态，如果执行成功则状态转变为 fulfilled，否则为 rejected 。后两种情况下，处理函数会被添加以表示 promise 已处理。then() 方法允许添加 fulfillment 和 rejection 处理，而 catch() 方法接收 rejection 处理。

你可以用使用各种方式来串联 promise 并在链上传递信息。每次调用 then() 都会在先前的 promise 处理过后创建并返回一个新的状态为 resolved 的 promise 。promise 链可以触发一系列异步事件的响应。你也可以使用 Promise.race() 和 Promise.all() 来监察多个 promise 的进度并参照它们做出响应。

当混合生成器和 promise 时，运行异步任务更加方便，因为 promise 提供了异步操作可以返回的公共接口形式。于是你可以使用生成器和 yield 操作符来等待异步操作的完成并正确的响应它们。

很多新的 web API 建立在 promise 之上，你可以期待未来还会有络绎不绝的以 promise 为基础的 API 出现。

<br />

