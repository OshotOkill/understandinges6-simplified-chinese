# 扩展的对象功能（Expanded Object Functionality）


ECMAScript 6 着重专注于提升对象的实用性，因为 JavaScript 中几乎所有的类型都是对象。此外，在一般项目中对象的平均个数随着 JavaScript 应用的复杂度而增加，也就说项目中对象的数目会持续增长。对象越多就越需要有效地去使用它们。

ECMAScript 6 给对象的各个方面，从简单的语法扩展到操作与交互，都做了改进。

<br />

### 对象类别（Object Categories）


JavaScript 混合了多种术语来描述规范中定义的对象，而非针对浏览器或者 Node.js 这些执行环境。ECMAScript 6 规范明确定义了每种对象类别。理解该术语对于从整体上认识该门语言显得十分重要。对象类别包括：

* 普通对象（ordinary object）拥有 JavaScript 对象所有的默认行为。
* 特异对象（exotic object）的某些内部行为和默认的有所差异。
* 标准对象（standard object）是 ECMAScript 6 中定义的对象，例如 Array, Date 等，它们既可能是普通也可能是特异对象。
* 内置对象（built-in object）指 JavaScript 执行环境开始运行时已存在的对象。标准对象均为内置对象。

我会在整本书中使用这些术语来说明在 ECMAScript 6 中定义的各式对象。

<br />

### 对象字面量语法扩展（Object Literal Syntax Extensions）


对象字面量是 JavaScript 编程中流行的模式之一。JSON 就是由其衍生而来，而且对象几乎存在于因特网上每份JavaScript 文件中。对象字面量之所以流行是因为相比其它方式它能更简洁的创建对象。对于开发者幸运的是，ECMAScript 6 让对象字面量更为强大的同时还有多种方式使用起来更为简洁。

<br />

#### 简写的属性初始化（Property Initializer Shorthand）


在 ECMAScript 5 及之前的版本中，对象字面量是简单的键值对的集合。这意味着属性被初始化时可能有所重复，如下所示：

```
function createPerson(name, age) {
    return {
        name: name,
        age: age
    };
}
```
createPerson() 函数创建了一个属性名和参数名相同的对象。从结果上来看 name 和 age 有所重复，即使它们分别指的是对象的属性名和给提供值的变量。在返回的对象中，name 键的值被 name 变量赋给，age 键同理。

在 ECMAScript 6 中，你可以使用简写属性来消除对象属性名和本地变量名的重复。当对象属性名和本地变量名相同时，你可以省略冒号与值。如下，createPerson() 可以用 ECMAScript 6 重写：

```
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

#### 简化的函数（Concise Methods）


ECMAScript 6 同样改进了对象字面量中方法的赋值。在 ECMAScript 5 和更早的版本中，你必须指定一个名字并使用函数定义的完整形式来给对象添加方法，如下：

```
var person = {
    name: "Nicholas",
    sayName: function() {
        console.log(this.name);
    }
};
```

在 ECMAScript 6 中，该语法通过省略冒号和 function 关键字变得更简洁，也就是说你可以像下面这样重写上个例子：

```
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

```
var person = {},
    lastName = "last name";

person["first name"] = "Nicholas";
person[lastName] = "Zakas";

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

既然 lastName 已被赋值为 "last name"，而且该例中两个属性名都包含空格，所以使用点操作符来引用它们是不可能的。然而，方括号允许属性为任意字符串，所以 "first name" 和 "last name" 能被分别赋值为 "Nicholas"，"Zakas" 。

另外，你可以在对象字面量中直接使用字符串字面量做属性，像这样：

```
var person = {
    "first name": "Nicholas"
};

console.log(person["first name"]);      // "Nicholas"
```

使用这个模式的前提是要事先知道属性的名字，并且能由字符串字面量来表示。不过，如果 "first name" 属性名被包含在一个变量里（如之前的例子）或者需要计算才能得到，那么 ECMAScript 5 无法在对象字面量中使用该属性名。*

在 ECMAScript 6 中，动态计算属性名是对象字面量语法中的一部分，同样使用方括号来标识它在对象实例中的身份为计算得到的属性名。例如：

```
var lastName = "last name";

var person = {
    "first name": "Nicholas",
    [lastName]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

对象字面量内的方括号说明该属性名需要计算得到，得出的结果是以一个字符串。这意味着你可以如下在方括号内放入表达式：

```
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

### 新的方法（New Method）


ECMAScript 从第五版开始避免在 Object.prototype 上添加新的全局函数或方法，转而去考虑具体的对象类型如数组）应该有什么方法。当某些方法不适合这些具体类型时就将它们添加到全局 Object 上 *。ECMAScript 6 在全局 Object 上添加了几个新的方法来轻松地完成一些特定任务。 

<br />

#### Object.is()（The Object.is() Method）


在 JavaSciprt 中当你想比较两个值时，你极有可能使用比较操作符（==）或严格比较操作符（===）。许多开发者为了避免在比较的过程中发生强制类型转换，更倾向于后者。但即使是严格等于操作符，它也不是万能的。例如，它认为 +0 和 -0 是相等的，虽然它们在 JavaScript 引擎中表示的方式不同。同样 NaN === NaN 会返回 false，所以必须使用 isNaN() 函数才能判断 NaN 。

ECMAScript 6 引入了 Object.is() 方法来补偿严格等于操作符怪异行为的过失。该函数接受两个参数并在它们相等的返回 true 。只有两者在类型和值都相同的情况下才会判为相等。如下所示：

```
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

```
function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key) {
        receiver[key] = supplier[key];
    });

    return receiver;
}
```

mixin() 函数在迭代提供者（supplier）对象的过程中会将该对象的属性拷贝到接收者（receiver）对象的内部（浅拷贝，若属性值为对象则共享其引用）。这允许接收者对象不需要继承即可获得新的属性，如下面的代码所示：

```
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

```
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

```
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

```
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
>在该段代码中，提供者对象包含一个 name 访问器属性。在使用 Object.assign() 方法之后，receive.name 作为数据属性存在且值为 "file.js"，因为 Object.assign() 被调用时 supplier.name 返回 "file.js"。

<br />

### 重复的对象字面量属性（Duplicate Object Literal Properties）

ECMAScript 5 strict mode introduced a check for duplicate object literal properties that would throw an error if a duplicate was found. For example, this code was problematic:

```
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // syntax error in ES5 strict mode
};
```

When running in ECMAScript 5 strict mode, the second name property causes a syntax error. But in ECMAScript 6, the duplicate property check was removed. Both strict and nonstrict mode code no longer check for duplicate properties. Instead, the last property of the given name becomes the property’s actual value, as shown here:

```
"use strict";

var person = {
    name: "Nicholas",
    name: "Greg"        // no error in ES6 strict mode
};

console.log(person.name);       // "Greg"
```

In this example, the value of person.name is "Greg" because that’s the last value assigned to the property.

<br />

### 自身属性的枚举排序（Own Property Enumeration Order）

ECMAScript 5 didn’t define the enumeration order of object properties, as it left this up to the JavaScript engine vendors. However, ECMAScript 6 strictly defines the order in which own properties must be returned when they are enumerated. This affects how properties are returned using Object.getOwnPropertyNames() and Reflect.ownKeys (covered in Chapter 12). It also affects the order in which properties are processed by Object.assign().

The basic order for own property enumeration is:

1. All numeric keys in ascending order
2. All string keys in the order in which they were added to the object
3. All symbol keys (covered in Chapter 6) in the order in which they were added to the object

Here’s an example:

```
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

The Object.getOwnPropertyNames() method returns the properties in obj in the order 0, 1, 2, a, c, b, d. Note that the numeric keys are grouped together and sorted, even though they appear out of order in the object literal. The string keys come after the numeric keys and appear in the order that they were added to obj. The keys in the object literal itself come first, followed by any dynamic keys that were added later (in this case, d).

The for-in loop still has an unspecified enumeration order because not all JavaScript engines implement it the same way. The Object.keys() method and JSON.stringify() are both specified to use the same (unspecified) enumeration order as for-in.

While enumeration order is a subtle change to how JavaScript works, it’s not uncommon to find programs that rely on a specific enumeration order to work correctly. ECMAScript 6, by defining the enumeration order, ensures that JavaScript code relying on enumeration will work correctly regardless of where it is executed.

<br />

### 更多的原型属性（More Powerful Prototypes）

Prototypes are the foundation of inheritance in JavaScript, and ECMAScript 6 continues to make prototypes more powerful. Early versions of JavaScript severely limited what could be done with prototypes. However, as the language matured and developers became more familiar with how prototypes work, it became clear that developers wanted more control over prototypes and easier ways to work with them. As a result, ECMAScript 6 introduced some improvements to prototypes.

<br />

#### 改变对象的原型（Changing an Object’s Prototype）

Normally, the prototype of an object is specified when the object is created, via either a constructor or the Object.create() method. The idea that an object’s prototype remains unchanged after instantiation was one of the biggest assumptions in JavaScript programming through ECMAScript 5. ECMAScript 5 did add the Object.getPrototypeOf() method for retrieving the prototype of any given object, but it still lacked a standard way to change an object’s prototype after instantiation.

ECMAScript 6 changes that assumption by adding the Object.setPrototypeOf() method, which allows you to change the prototype of any given object. The Object.setPrototypeOf() method accepts two arguments: the object whose prototype should change and the object that should become the first argument’s prototype. For example:

```
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

// prototype is person
let friend = Object.create(person);
console.log(friend.getGreeting());                      // "Hello"
console.log(Object.getPrototypeOf(friend) === person);  // true

// set prototype to dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

This code defines two base objects: person and dog. Both objects have a getGreeting() method that returns a string. The object friend first inherits from the person object, meaning that getGreeting() outputs "Hello". When the prototype becomes the dog object, person.getGreeting() outputs "Woof" because the original relationship to person is broken.

The actual value of an object’s prototype is stored in an internal-only property called [[Prototype]]. The Object.getPrototypeOf() method returns the value stored in [[Prototype]] and Object.setPrototypeOf() changes the value stored in [[Prototype]]. However, these aren’t the only ways to work with the value of [[Prototype]].

<br />

#### 使用 super 引用来方便获取 prototype（Easy Prototype Access with Super References）

As previously mentioned, prototypes are very important for JavaScript and a lot of work went into making them easier to use in ECMAScript 6. Another improvement is the introduction of super references, which make accessing functionality on an object’s prototype easier. For example, to override a method on an object instance such that it also calls the prototype method of the same name, you’d do the following in ECMAScript 5:

```
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

// set prototype to person
Object.setPrototypeOf(friend, person);
console.log(friend.getGreeting());                      // "Hello, hi!"
console.log(Object.getPrototypeOf(friend) === person);  // true

// set prototype to dog
Object.setPrototypeOf(friend, dog);
console.log(friend.getGreeting());                      // "Woof, hi!"
console.log(Object.getPrototypeOf(friend) === dog);     // true
```

In this example, getGreeting() on friend calls the prototype method of the same name. The Object.getPrototypeOf() method ensures the correct prototype is called, and then an additional string is appended to the output. The additional .call(this) ensures that the this value inside the prototype method is set correctly.

Remembering to use Object.getPrototypeOf() and .call(this) to call a method on the prototype is a bit involved, so ECMAScript 6 introduced super. At its simplest, super is a pointer to the current object’s prototype, effectively the Object.getPrototypeOf(this) value. Knowing that, you can simplify the getGreeting() method as follows:

```
let friend = {
    getGreeting() {
        // in the previous example, this is the same as:
        // Object.getPrototypeOf(this).getGreeting.call(this)
        return super.getGreeting() + ", hi!";
    }
};
```

The call to super.getGreeting() is the same as Object.getPrototypeOf(this).getGreeting.call(this) in this context. Similarly, you can call any method on an object’s prototype by using a super reference, so long as it’s inside a concise method. Attempting to use super outside of concise methods results in a syntax error, as in this example:

```
let friend = {
    getGreeting: function() {
        // syntax error
        return super.getGreeting() + ", hi!";
    }
};
```

This example uses a named property with a function, and the call to super.getGreeting() results in a syntax error because super is invalid in this context.

The super reference is really powerful when you have multiple levels of inheritance, because in that case, Object.getPrototypeOf() no longer works in all circumstances. For example:

```
let person = {
    getGreeting() {
        return "Hello";
    }
};

// prototype is person
let friend = {
    getGreeting() {
        return Object.getPrototypeOf(this).getGreeting.call(this) + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);


// prototype is friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // error!
```

The call to Object.getPrototypeOf() results in an error when relative.getGreeting() is called. That’s because this is relative, and the prototype of relative is the friend object. When friend.getGreeting().call() is called with relative as this, the process starts over again and continues to call recursively until a stack overflow error occurs.

That problem is difficult to solve in ECMAScript 5, but with ECMAScript 6 and super, it’s easy:

```
let person = {
    getGreeting() {
        return "Hello";
    }
};

// prototype is person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);


// prototype is friend
let relative = Object.create(friend);

console.log(person.getGreeting());                  // "Hello"
console.log(friend.getGreeting());                  // "Hello, hi!"
console.log(relative.getGreeting());                // "Hello, hi!"
```

Because super references are not dynamic, they always refer to the correct object. In this case, super.getGreeting() always refers to person.getGreeting(), regardless of how many other objects inherit the method.

<br />

### “方法” 的正式定义（A Formal Method Definition）

Prior to ECMAScript 6, the concept of a “method” wasn’t formally defined. Methods were just object properties that contained functions instead of data. ECMAScript 6 formally defines a method as a function that has an internal [[HomeObject]] property containing the object to which the method belongs. Consider the following:

```
let person = {

    // method
    getGreeting() {
        return "Hello";
    }
};

// not a method
function shareGreeting() {
    return "Hi!";
}
```

This example defines person with a single method called getGreeting(). The [[HomeObject]] for getGreeting() is person by virtue of assigning the function directly to an object. The shareGreeting() function, on the other hand, has no [[HomeObject]] specified because it wasn’t assigned to an object when it was created. In most cases, this difference isn’t important, but it becomes very important when using super references.

Any reference to super uses the [[HomeObject]] to determine what to do. The first step is to call Object.getPrototypeOf() on the [[HomeObject]] to retrieve a reference to the prototype. Then, the prototype is searched for a function with the same name. Last, the this binding is set and the method is called. Here’s an example:

```
let person = {
    getGreeting() {
        return "Hello";
    }
};

// prototype is person
let friend = {
    getGreeting() {
        return super.getGreeting() + ", hi!";
    }
};
Object.setPrototypeOf(friend, person);

console.log(friend.getGreeting());  // "Hello, hi!"
```

Calling friend.getGreeting() returns a string, which combines the value from person.getGreeting() with ", hi!". The [[HomeObject]] of friend.getGreeting() is friend, and the prototype of friend is person, so super.getGreeting() is equivalent to person.getGreeting.call(this)`.

<br />

### 总结（Summary）

Objects are the center of programming in JavaScript, and ECMAScript 6 made some helpful changes to objects that both make them easier to deal with and more powerful.

ECMAScript 6 makes several changes to object literals. Shorthand property definitions make assigning properties with the same names as in-scope variables easier. Computed property names allow you to specify non-literal values as property names, which you’ve already been able to do in other areas of the language. Shorthand methods let you type a lot fewer characters in order to define methods on object literals, by completely omitting the colon and function keyword. ECMAScript 6 loosens the strict mode check for duplicate object literal property names as well, meaning you can have two properties with the same name in a single object literal without throwing an error.

The Object.assign() method makes it easier to change multiple properties on a single object at once. This can be very useful if you use the mixin pattern. The Object.is() method performs strict equality on any value, effectively becoming a safer version of === when dealing with special JavaScript values.

Enumeration order for own properties is now clearly defined in ECMAScript 6. When enumerating properties, numeric keys always come first in ascending order followed by string keys in insertion order and symbol keys in insertion order.

It’s now possible to modify an object’s prototype after it’s already created, thanks to ECMAScript 6’s Object.setPrototypeOf() method.

Finally, you can use the super keyword to call methods on an object’s prototype. The this binding inside a method invoked using super is set up to automatically work with the current value of this.


