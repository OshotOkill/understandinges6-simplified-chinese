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

ECMAScript 5 and earlier could compute property names on object instances when those properties were set with square brackets instead of dot notation. The square brackets allow you to specify property names using variables and string literals that may contain characters that would cause a syntax error if used in an identifier. Here’s an example:

```
var person = {},
    lastName = "last name";

person["first name"] = "Nicholas";
person[lastName] = "Zakas";

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

Since lastName is assigned a value of "last name", both property names in this example use a space, making it impossible to reference them using dot notation. However, bracket notation allows any string value to be used as a property name, so assigning "first name" to "Nicholas" and “last name" to "Zakas" works.

Additionally, you can use string literals directly as property names in object literals, like this:

```
var person = {
    "first name": "Nicholas"
};

console.log(person["first name"]);      // "Nicholas"
```

This pattern works for property names that are known ahead of time and can be represented with a string literal. If, however, the property name "first name" were contained in a variable (as in the previous example) or had to be calculated, then there would be no way to define that property using an object literal in ECMAScript 5.

In ECMAScript 6, computed property names are part of the object literal syntax, and they use the same square bracket notation that has been used to reference computed property names in object instances. For example:

```
var lastName = "last name";

var person = {
    "first name": "Nicholas",
    [lastName]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person[lastName]);          // "Zakas"
```

The square brackets inside the object literal indicate that the property name is computed, so its contents are evaluated as a string. That means you can also include expressions such as:

```
var suffix = " name";

var person = {
    ["first" + suffix]: "Nicholas",
    ["last" + suffix]: "Zakas"
};

console.log(person["first name"]);      // "Nicholas"
console.log(person["last name"]);       // "Zakas"
```

These properties evaluate to "first name" and "last name", and those strings can be used to reference the properties later. Anything you would put inside square brackets while using bracket notation on object instances will also work for computed property names inside object literals.

<br />

### 新方法（New Method）

One of the design goals of ECMAScript beginning with ECMAScript 5 was to avoid creating new global functions or methods on Object.prototype, and instead try to find objects on which new methods should be available. As a result, the Object global has received an increasing number of methods when no other objects are more appropriate. ECMAScript 6 introduces a couple new methods on the Object global that are designed to make certain tasks easier.

<br />

#### Object.is() 方法（The Object.is() Method）

When you want to compare two values in JavaScript, you’re probably used to using either the equals operator (==) or the identically equals operator (===). Many developers prefer the latter, to avoid type coercion during comparison. But even the identically equals operator isn’t entirely accurate. For example, the values +0 and -0 are considered equal by === even though they are represented differently in the JavaScript engine. Also NaN === NaN returns false, which necessitates using isNaN() to detect NaN properly.

ECMAScript 6 introduces the Object.is() method to make up for the remaining quirks of the identically equals operator. This method accepts two arguments and returns true if the values are equivalent. Two values are considered equivalent when they are of the same type and have the same value. Here are some examples:

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

In many cases, Object.is() works the same as the === operator. The only differences are that +0 and -0 are considered not equivalent and NaN is considered equivalent to NaN. But there’s no need to stop using equality operators altogether. Choose whether to use Object.is() instead of == or === based on how those special cases affect your code.

<br />

#### Object.assign() 方法（The Object.assign() Method）

Mixins are among the most popular patterns for object composition in JavaScript. In a mixin, one object receives properties and methods from another object. Many JavaScript libraries have a mixin method similar to this:

```
function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key) {
        receiver[key] = supplier[key];
    });

    return receiver;
}
```

The mixin() function iterates over the own properties of supplier and copies them onto receiver (a shallow copy, where object references are shared when property values are objects). This allows the receiver to gain new properties without inheritance, as in this code:

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

Here, myObject receives behavior from the EventTarget.prototype object. This gives myObject the ability to publish events and subscribe to them using the emit() and on() methods, respectively.

This pattern became popular enough that ECMAScript 6 added the Object.assign() method, which behaves the same way, accepting a receiver and any number of suppliers, and then returning the receiver. The name change from mixin() to assign() reflects the actual operation that occurs. Since the mixin() function uses the assignment operator (=), it cannot copy accessor properties to the receiver as accessor properties. The name Object.assign() was chosen to reflect this distinction.

Similar methods in various libraries may have other names for the same basic functionality; popular alternates include the extend() and mix() methods. There was also, briefly, an Object.mixin() method in ECMAScript 6 in addition to the Object.assign() method. The primary difference was that Object.mixin() also copied over accessor properties, but the method was removed due to concerns over the use of super (discussed in the “Easy Prototype Access with Super References” section of this chapter).

You can use Object.assign() anywhere the mixin() function would have been used. Here’s an example:

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

The Object.assign() method accepts any number of suppliers, and the receiver receives the properties in the order in which the suppliers are specified. That means the second supplier might overwrite a value from the first supplier on the receiver, which is what happens in this snippet:

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

The value of receiver.type is "css" because the second supplier overwrote the value of the first.

The Object.assign() method isn’t a big addition to ECMAScript 6, but it does formalize a common function found in many JavaScript libraries.

<br />

> #### Working with Accessor Properties
Keep in mind that Object.assign() doesn’t create accessor properties on the receiver when a supplier has accessor properties. Since Object.assign() uses the assignment operator, an accessor property on a supplier will become a data property on the receiver. For example:

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
>In this code, the supplier has an accessor property called name. After using the Object.assign() method, receiver.name exists as a data property with a value of "file.js" because supplier.name returned "file.js" when Object.assign() was called.


<br />


