# 扩展的对象功能（Expanded Object Functionality）


ECMAScript 6 着重专注于提升对象的实用性，因为 JavaScript 中几乎所有的类型都是对象。此外，在一般项目中对象的平均个数随着 JavaScript 应用的复杂度而增加，也就说项目中对象的数目会持续增长。对象越多就越需要有效地去使用它们。

ECMAScript 6 给对象的各个方面，从简单的语法扩展到操作与交互，都做了改进。

<br />

### 本章小结
* [对象类别](#Object-Categories)
* [对象字面量语法扩展](#Object-Literal-Syntax-Extensions)
* [新的方法](#New-Methods)
* [重复的对象字面量属性](#Duplicate-Object-Literal-Properties)
* [自身属性的枚举排序](#Own-Property-Enumeration-Order)
* [更多的原型属性](#More-Powerful-Prototypes)
* [何为 “方法”](#A-Formal-Method-Definition)
* [总结](#Summary)

<br />

### <a id="Object-Categories"> 对象类别（Object Categories） </a>


JavaScript 混合了多种术语来描述规范中定义的对象，而非针对浏览器或者 Node.js 这些执行环境。ECMAScript 6 规范明确定义了每种对象类别。理解该术语对于从整体上认识该门语言显得十分重要。对象类别包括：

* 普通对象（ordinary object）拥有 JavaScript 对象所有的默认行为。
* 特异对象（exotic object）的某些内部行为和默认的有所差异。
* 标准对象（standard object）是 ECMAScript 6 中定义的对象，例如 Array, Date 等，它们既可能是普通也可能是特异对象。
* 内置对象（built-in object）指 JavaScript 执行环境开始运行时已存在的对象。标准对象均为内置对象。

我会在整本书中使用这些术语来说明在 ECMAScript 6 中定义的各式对象。

<br />

### <a id="Object-Literal-Syntax-Extensions"> 对象字面量语法扩展（Object Literal Syntax Extensions） </a>


对象字面量是 JavaScript 编程中流行的模式之一。JSON 就是由其衍生而来，而且对象几乎存在于因特网上每份JavaScript 文件中。对象字面量之所以流行是因为相比其它方式它能更简洁的创建对象。对于开发者幸运的是，ECMAScript 6 让对象字面量更为强大的同时还有多种方式使用起来更为简洁。

<br />

#### 简写的属性初始化（Property Initializer Shorthand）


在 ECMAScript 5 及之前的版本中，对象字面量是简单的键值对的集合。这意味着属性被初始化时可能有所重复，如下所示：

```js
function createPerson(name, age) {
    return {
        name: name,
        age: age
    };
}
```

createPerson() 函数创建了一个属性名和参数名相同的对象。从结果上来看 name 和 age 有所重复，即使它们分别指的是对象的属性名和给提供值的变量。在返回的对象中，name 键的值被 name 变量赋给，age 键同理。

在 ECMAScript 6 中，你可以使用简写属性来消除对象属性名和本地变量名的重复。当对象属性名和本地变量名相同时，你可以省略冒号与值。如下，createPerson() 可以用 ECMAScript 6 重写：

```js
function createPerson(name, age) {
    return {
        name,
        age
    };
}
```

当对象字面量中的属性只有属性名的时候，JavaScript 引擎会在该作用域内寻找是否有和属性同名的变量。在本例中，本地变量 name 的值被赋给了对象字面量中的 name 属性。

该项扩展使得对象字面量的初始化变得简明的同时也消除了命名错误。对象属性被同名变量赋值在 JavaScript 中是一种普遍的编程模式，所以这项扩展的添加非常受欢迎。

<br />

#### 简写的方法（Concise Methods）


ECMAScript 6 同样改进了对象字面量中方法的赋值。在 ECMAScript 5 和更早的版本中，你必须指定一个名字并使用函数定义的完整形式来给对象添加方法，如下：

```js
var person = {
    name: "Nicholas",
    sayName: function() {
        console.log(this.name);
    }
};
```

在 ECMAScript 6 中，该语法通过省略冒号和 function 关键字变得更简洁，也就是说你可以像下面这样重写上个例子：

```js
var person = {
    name: "Nicholas",
    sayName() {
        console.log(this.name);
    }
};
```

这种简写的语法，也称为简写方法语法（concise method syntax）*，和上例同样在 person 对象内部创建了方法。sayName() 属性由匿名函数赋值并拥有 ECMAScript 5 sayName() 函数的全部特征。一项区别是简写方法可以使用 super（在 “使用 super 引用方便获取 prototype ” 一节中讨论），而非简写方法不能使用。

使用简写方式创建的方法，其 name 属性值为括号前的命名。在上个例子中，person.sayName() 的 name 属性为 "sayName"。

<br />

#### 动态计算的属性名（Computed Property Names）

在 ECMAScript 5 和早期的版本中，在对象实例名称后使用方括号包含而非点操作符操作的属性可以被动态计算。方括号允许你使用变量或包含字符串的字面量来指定属性名，虽然后者作为标识符和有语法错误 *，这里有个范例：

```js
var person = {},
    lastName = "last name";

person["first name"] = "Nicholas";
person[lastName] = "Zakas";

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

既然 lastName 已被赋值为 "last name"，而且该例中两个属性名都包含空格，所以使用点操作符来引用它们是不可能的。然而，方括号允许属性为任意字符串，所以 "first name" 和 "last name" 能被分别赋值为 "Nicholas"，"Zakas" 。

另外，你可以在对象字面量中直接使用字符串字面量做属性，像这样：

```js
var person = {
    "first name": "Nicholas"
};

console.log(person["first name"]);      // "Nicholas"
```

使用这个模式的前提是要事先知道属性的名字，并且能由字符串字面量来表示。不过，如果 "first name" 属性名被包含在一个变量里（如之前的例子）或者需要计算才能得到，那么 ECMAScript 5 无法在对象字面量中使用该属性名。*

在 ECMAScript 6 中，动态计算属性名是对象字面量语法中的一部分，同样使用方括号来标识它在对象实例中的身份为计算得到的属性名。例如：

```js
var lastName = "last name";

var person = {
    "first name": "Nicholas",
    [lastName]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

对象字面量内的方括号说明该属性名需要计算得到，得出的结果是以一个字符串。这意味着你可以如下在方括号内放入表达式：

```js
var suffix = " name";

var person = {
    ["first" + suffix]: "Nicholas",
    ["last" + suffix]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person["last name"]);       // "Zakas"
```

这些属性名的计算结果为 "first name" 和 "last name"，而且这些名称可以晚些时候用来引用属性。填入紧跟在对象实例后的方括号中的值同样能引用对象字面量中动态的计算属性名。

<br />

### <a id="New-Methods"> 新的方法（New Method） </a>


ECMAScript 从第五版开始避免在 Object.prototype 上添加新的全局函数或方法，转而去考虑具体的对象类型如数组）应该有什么方法。当某些方法不适合这些具体类型时就将它们添加到全局 Object 上 *。ECMAScript 6 在全局 Object 上添加了几个新的方法来轻松地完成一些特定任务。

<br />

#### Object.is() （The Object.is() Method）


在 JavaSciprt 中当你想比较两个值时，你极有可能使用比较操作符（==）或严格比较操作符（===）。许多开发者为了避免在比较的过程中发生强制类型转换，更倾向于后者。但即使是严格等于操作符，它也不是万能的。例如，它认为 +0 和 -0 是相等的，虽然它们在 JavaScript 引擎中表示的方式不同。同样 NaN === NaN 会返回 false，所以必须使用 isNaN() 函数才能判断 NaN 。

ECMAScript 6 引入了 Object.is() 方法来补偿严格等于操作符怪异行为的过失。该函数接受两个参数并在它们相等的返回 true 。只有两者在类型和值都相同的情况下才会判为相等。如下所示：

```js
console.log(+0 == -0);              // true
console.log(+0 === -0);             // true
console.log(Object.is(+0, -0));     // false

console.log(NaN == NaN);            // false
console.log(NaN === NaN);           // false
console.log(Object.is(NaN, NaN));   // true

console.log(5 == 5);                // true
console.log(5 == "5");              // true
console.log(5 === 5);               // true
console.log(5 === "5");             // false
console.log(Object.is(5, 5));       // true
console.log(Object.is(5, "5"));     // false
```

很多情况下 Object.is() 的表现和 === 是相同的。它们之间的区别是前者认为 +0 和 -0 不相等而 NaN 和 NaN 则是相同的。不过弃用后者是完全没有必要的。何时选择 Object.is() 与 == 或 === 取决于代码的实际情况。

<br />

#### Object.assign() （The Object.assign() Method）


混入（Mixin）是 JavaScript 中最流行的对象协作（object composition）模式。在一个混入中，一个对象接收另一个对象的属性及方法。很多 JavaScript 的库中都有类似于下例中的 mixin 方法：

```js
function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key) {
        receiver[key] = supplier[key];
    });

    return receiver;
}
```

mixin() 函数在迭代提供者（supplier）对象的过程中会将该对象的属性拷贝到接收者（receiver）对象的内部（浅拷贝，若属性值为对象则共享其引用）。这允许接收者对象不需要继承即可获得新的属性，如下面的代码所示：

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
};

var myObject = {};
mixin(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

在这里，myObject 接收 EventTarget.prototype 对象的方法。这就赋予了 myObject 通过 emit() 和 on() 方法单独发布和订阅事件的能力。

该模式在 ECMAScript 6 添加 Object.assign() 之前很流行。新的方法作用很相似，参数为一个接收者对象和任意数量的提供者对象。同时命名由 mixin() 变更为 assign() 更能反映出该方法的实质。既然 mixin() 函数使用了赋值操作符（=），那么接收者对象无法将其它对象的访问器属性（accessor properties）拷贝给自身。Object.assign() 的命名就是为了反映这个差异。

各式各样的库中都都有实现该基本需求但命名五花八门的方法；比较知名的有 extend() 和 mix() 。同样，有一短时期内 Object.mixin() 方法和 Object.assign() 并存于 ECMAScript 6 中。它们的主要差异是前者可以拷贝访问器属性到自身。不过，考虑到 super （在本章 “使用 super 引用来方便获取 prototype” 一节介绍）的使用方式又移除了它。

你可以在任何可以使用 mixin() 函数的场景来利用 Object.assign()。这里有个例子：

```js
function EventTarget() { /*...*/ }
EventTarget.prototype = {
    constructor: EventTarget,
    emit: function() { /*...*/ },
    on: function() { /*...*/ }
}

var myObject = {}
Object.assign(myObject, EventTarget.prototype);

myObject.emit("somethingChanged");
```

Object.assign() 方法接收任意数量的提供者对象，接收者对象根据提供者内部的属性定义顺序来接收它们。这意味
着后面的提供者对象可以重写前面的提供者对象的属性值。该情况在下面的例子出现：

```js
var receiver = {};

Object.assign(receiver,
    {
        type: "js",
        name: "file.js"
    },
    {
        type: "css"
    }
);

console.log(receiver.type);     // "css"
console.log(receiver.name);     // "file.js"
```

receiver.type 的值为 "css"，因为第二个提供者对象重写了第一个对象中的type属性。

Object.assign() 方法并未在 ECMAScript 6 中掀起多大波澜，不过它确实标准化了许多 JavaScript 库中都存在的一个公用方法。

<br />

> #### 操作访问器属性（Working with Accessor Properties）
>
需要注意的是 Object.assign() 在接收提供的访问器属性的时候不会创建自己的访问器属性。由于 Object.assign() 使用了赋值操作，所以访问器属性在接收者对象中作为数据属性（data property）存在。例如：

```js
var receiver = {},
    supplier = {
        get name() {
            return "file.js"
        }
    };

Object.assign(receiver, supplier);

var descriptor = Object.getOwnPropertyDescriptor(receiver, "name");

console.log(descriptor.value);      // "file.js"
console.log(descriptor.get);        // undefined
```

<br />

> 在该段代码中，提供者对象包含一个 name 访问器属性。在使用 Object.assign() 方法之后，receive.name 作为数据属性存在且值为 "file.js"，因为 Object.assign() 被调用时 supplier.name 返回 "file.js"。

<br />

### <a id="Duplicate-Object-Literal-Properties"> 重复的对象字面量属性（Duplicate Object Literal Properties） </a>


ECMAScript 5 在严格模式中检查对象字面量的属性，如若有重复存在便抛出错误。例如，下面的代码会有问题：

```js
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // ES5 严格模式下抛出错误
};
```

在 ECMAScript 5 的严格模式下运行时，第二个 name 属性会造成语法错误。但是在 ECMAScript 6 中，重复属性的检查被移除了，而且后面的同名属性值成为了该对象属性的最终值，如下所示：

```js
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // ES6 严格模式下正常运行
};

console.log(person.name);       // "Greg"
```

在本例中，person.name 的值时 "Greg" 因为该属性最后被赋的值是 "Greg"。

<br />

### <a id="Own-Property-Enumeration-Order"> 自身属性的枚举排序（Own Property Enumeration Order） </a>


ECMAScript 5 并没有定义枚举对象属性的顺序，并将其交给各 JavaScript 引擎自行决定。然而，ECMAScript 6 严格定义了枚举对象自身（own）属性时返回的属性名顺序。这对 Object.getOwnPropertyNames() 和 Reflect.ownKeys（在第十二章介绍）返回的属性名集合有一定影响，包括从 Object.assign() 中获得的属性。

枚举自身属性返回的属性名顺序的基本准则如下：

1. 类型为数字（numeric）键会升序。
2. 类型为字符的键按照被添加到对象时的顺序保持不变。
3. 类型为 Symbol（在第六章讲解）的键按照被添加到对象时的顺序保持不变。

这里有个示例：

```js
var obj = {
    a: 1,
    0: 1,
    c: 1,
    2: 1,
    b: 1,
    1: 1
};

obj.d = 1;

console.log(Object.getOwnPropertyNames(obj).join(""));     // "012acbd"
```

Object.getOwnPropertyNames() 方法返回的属性集合顺序依次为 0，1，2，a，c，b，d 。注意数字类型的键会被提升并自动排序，字符类型的键紧随其后并保持添加到对象时的顺序。对象字面量本身的键优先级比后动态添加到对象的高（在本例中，d）。

for-in 循环的枚举顺序仍不明确，因为各 JavaScript 引擎的实现不懂。同样 Object.keys() 和 JSON.stringify() 由于枚举顺序和 for-in 相同导致它们的具体结果也无确切定义。

虽然枚举顺序的变动对 JavaScript 的运作来讲细小甚微，但是依赖于枚举顺序而运行的项目并不罕见。ECMAScript 6 通过定义枚举的顺序使其与具体的执行环境无关，保证依赖于枚举的 JavaScript 代码能正常工作。

<br />

### <a id="More-Powerful-Prototypes"> 更多的原型属性（More Powerful Prototypes） </a>


原型（prototypes）是 JavaScript 中继承的基础，ECMAScript 6 仍继续努力让原型变得更为强大。早期的 JavaScript 对原型的功能限制极为严重。然而，当语言逐渐成熟而且开发者也愈加熟悉原型的工作机制之后，开发者希望获得原型更多控制权的同时又能方便使用它们。于是 ECMAScipt 6 对原型做了一些改进。

<br />

#### 改变对象的原型（Changing an Object’s Prototype）


一般情况下，原型在该对象由构造函数或 Object.create() 方法创建时出现。ECMAScript 5 下的 JavaScript 编程最重要的约定之一就是一个对象实例无法更改它的原型。即使 ECMAScript 5 添加了 Object.getPrototypeOf() 方法来提取给定对象的原型，对象实例依然缺乏修改其原型的标准方式。

ECMAScript 6 通过添加 Object.setPrototypeOf() 方法来对该约定做了变更。它允许你改变任何给定对象实例的原型。Object.setPrototypeof() 方法接收两个参数：需要改变原型的对象和你期望的原型对象。如下例所示：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};

// 原型为 person
let friend = Object.create(person);
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true

// 改变原型为 dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

该段代码定义了两个对象：person 与 dog ，它们拥有返回字符串的同名方法。friend 对象继承了 person 对象，意味从 friend 上调用 gretGreeting() 会输出 "Hello"。当原型变更为 dog 对象之后，friend.getGretting() 会输出 "Woof"，因为 friend 和 person 之间的联系已被切断。

The actual value of an object’s prototype is stored in an internal-only property called [[Prototype]]. The Object.getPrototypeOf() method returns the value stored in [[Prototype]] and Object.setPrototypeOf() changes the value stored in [[Prototype]]. However, these aren’t the only ways to work with the value of [[Prototype]].

对象原型的实际值由一个内部属性 [[Prototype]] 存储。Object.getPrototypeOf() 方法返回的就是 [[Prototype]] 的值，而 Object.setPrototypeOf() 则会更改它。不过，操作 [[Prototype]] 值的方法并不只有这些。

<br />


#### 使用 super 引用来方便获取 prototype（Easy Prototype Access with Super References）


在上文中提到，原型对于 JavaScript 来讲至关重要，ECMAScript 6 也增强了它的易用性，包括使用 super 引用来方便的获取对象原型中的功能。例如，当你重写原型中的方法时需要调用该原型方法，在 ECMAScript 5 中你可能会这么做：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

let dog = {
    getGreeting() {
        return "Woof";
    }
};


let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};

// 设定 friend 的原型为 person
Object.setPrototypeOf(friend, person);
console.log(friend.getGreeting());                      // "Hello, hi!"
console.log(Object.getPrototypeOf(friend) === person);  // true

// 设定 friend 的原型为 dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof, hi!"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

在本例中，friend 在调用 getGreeting() 后需要使用同名的原型方法来输出额外的字符。附加的 .call(this) 保证原型方法拥有正确的 this 值。

使用 Object.getPrototypeOf() 和 .call(this) 来调用原型方法显得有些笨重，所以 ECMAScript 6 引入了 super。简单的讲，super 是指向当前函数原型的指针，其值等同于 Object.getPrototypeof(this)。了解之后，你可以如下简化 getGreeting() 方法：

```js
let friend = {
    getGreeting() {
        // 相比上个例子，等同于：
        // Object.getPrototypeOf(this).getGreeting.call(this)
        return super.getGreeting() + ", hi!";
    }
};
```

上例中，调用 super.getGreeting() 等同于调用 Object.getPrototypeOf(this).getGreeing.call(this)。类似的是，你可以使用 super 引用来调用任何存在于原型中的方法。super 只能在简写方法中使用（concise methods），除此之外将发生语法错误，正如下例所示：

```js
let friend = {
    getGreeting: function() {
        // 语法错误
        return super.getGreeting() + ", hi!";
    }
};
```

该例在命名之后使用了 function 关键字，所以调用 super.getGreeting() 会出现错误，因为在该上下文（context）中 super 是不可用的。

当你使用了多重继承的时候，super 引用是相当强大的，因为该情况下 Object.getPrototypeOf() 并不适用于所有的场景，例如：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型为 person
let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);


// 原型为 friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // 错误！
```

调用 Object.getPrototypeOf() 发生了错误。这是因为 this 的值是 relative，而 relative 的原型是 friend 对象。当 this 为 relative 的情况下调用 friend.getGreeting().call() 时，进程反复运作并持续递归调用该方法，直到抛出栈溢出错误。

这个问题在 ECMAScript 5 中很难解决，但是 ECMAScript 6 引入了 super 使得该问题变得小菜一碟：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型为 person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);


// 原型为 friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // "Hello, hi!"
```

因为 super 引用并非是动态的，所以它们总是指向正确的对象。在本例中，super.getGreeting() 总是指代 person.getGretting()，不论有多少对象继承了该方法。

<br />

### <a id="A-Formal-Method-Definition"> 何为 “方法”（A Formal Method Definition） </a>


在 ECMAScript 6 之前，“方法” 这一概念并未有过正式定义。方法泛指那些对象中值为函数而非数据的属性。ECMAScript 6 正式将方法定义为带有 [[HomeObject]] 内部属性的函数，该属性指出方法的拥有者。考虑如下的例子：

```js
let person = {

    // 方法
    getGreeting() {
        return "Hello";
    }
};

// 不是方法
function shareGreeting() {
    return "Hi!";
}
```

该例中定义了 person 和名为 getGreeting() 的方法。由于该函数被直接分配给了 person 对象，所以 getGretting() 内部的 [[HomeObject]] 值为 person。另一方面，shareGretting() 由于在创建时没有分配给任何对象，所以不包含 [[HomeObject]]。大部分情况下该差异并不十分重要，不过要使用 super 引用的时候另当别论。

任何 super 引用都要由 [[HomeObject]] 来决定它们要做的工作。当使用时，首先做的是在 [[HomeObject]] 上调用 Object.getPrototypeOf() 来提取原型的引用，接下来在原型中寻找调用方法的命名。最后，绑定 this 值并调用该方法。下面有个例子：

```js
let person = {
    getGreeting() {
        return "Hello";
    }
};

// 原型为 person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);

console.log(friend.getGreeting());  // "Hello, hi!"
```

调用 friend.getGreeting()会返回 person.getGreeting() 与 ", hi!" 拼接后的字符串。friend.getGreeting() 的 [[HomeObject]] 值是 friend，该对象的原型是 person，所以 super.getGreeting() 等同于 person.getGreeting.call(this)。

<br />

### <a id="Summary"> 总结（Summary） </a>


对象是 JavaScript 编程的核心，ECMAScript 6 对它做了一些有益的改进令其变得更加易用和强大。

ECMAScrpit 6 为对象做了不少改进。简写属性定义让同作用域内的同名属性与变量之间的赋值更为简单。在其它位置应用的非字面值（non-literal value）可以被动态计算用做属性名。使用简写方法可以省略冒号与 function 关键字，让你在定义对象字面量方法的时候少敲了不少字母。ECMAScript 6 放宽了严格模式对于重复对象字面量属性的检查，意味着你可以在同一个对象字面量定义两个同名的属性而不抛出错误。

Object.assign() 方法简化了单个对象中多个属性的变动。当使用混入模式时非常有用。Object.is() 方法会针对传入的任何参数进行严格的比较，当处理 JavaScript 特殊值时结果比使用 === 更安全。

枚举属性的顺序在 ECMAScript 6 中变得明确。在枚举属性时，数字类型的键会升序并排在字符类型或 symbol 类型的键之前，后两者按照定义时的顺序保持不变。

现在对象实例可以去修改它的原型，多亏于 ECMAScript 6 的 Object.setPrototypeOf() 方法。

最后，你可以使用 super 关键字来调用在对象原型上的方法。该方法在调用的时 this 值就已经绑定完毕。

<br />
