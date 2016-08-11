# 类（Introducing JavaScript Classes）


和大多数面向对象的语言（object-oriented programming language）不同，JavaScript 在诞生之初并不支持使用类和传统的类继承并作为主要的定义方式来创建相似或关联的对象。这很令开发者困惑，而且在早于 ECMAScript 1 到 ECMAScript 5 这段时期，很多库都创建了一些实用工具（utility）来让 JavaScript 从表层上支持类。尽管一些 JavaScript 开发者强烈主张该语言不需要类，但由于大量的库都对类做了实现，ECMAScript 6 也顺势将其引入。

在探索 ECMAScript 6 类的过程中，理解类的幕后机制是很有帮助的，所以本章会首先探讨开发者是怎样在 ECMAScript 5 中模拟类的实现的。而且在之后的小节内，你也会发现 ECMAScript 6 中的类和其他语言相比并不是完全等同的，目的是为了和 JavaScript 与生俱来的动态特性相配合。

<br />

* [Class-Like Structures in ECMAScript 5](#Class-Like-Structures-in-ECMAScript-5)
* [Class Declarations](#Class-Declarations)
* [Class Expressions](#Class-Expressions)
* [Classes as First-Class Citizens](#Classes-as-First-Class-Citizens)
* [Accessor Properties](#Accessor-Properties)
* [Computed Member Names](#Computed-Member-Names)
* [Generator Methods](#Generator-Methods)
* [Static Members](#Static-Members)
* [Inheritance with Derived Classes](#Inheritance-with-Derived-Classes)
* [Using new.target in Class Constructors](#Using-newtarget-in-Class-Constructors)
* [Summary](#Summary)

<br />

### <a name="Class-Like-Structures-in-ECMAScript-5"> ECMAScript 5 中的类结构（Class-Like Structures in ECMAScript 5） </a>


在 ECMAScript 5 或更早的版本中，JavaScript 没有类。和类这个概念及行为最接近的是创建一个构造函数并在构造函数的原型上添加方法，这种实现也被称为自定义的类型创建，例如：

```
function PersonType(name) {
    this.name = name;
}

PersonType.prototype.sayName = function() {
    console.log(this.name);
};

let person = new PersonType("Nicholas");
person.sayName();   // 输出 "Nicholas"

console.log(person instanceof PersonType);  // true
console.log(person instanceof Object);      // true
```

在本例中，PersonType 是带有单个属性 name 的构造函数。sayName() 方法会添加到 PersonType 的原型上以共享给所有的实例。之后 PersonType 的一个实例由 new 操作符创建。该 person 对象根据原型继承会被同时视为 PersonType 和 Object 的实例。

很多 JavaScript 库都是用这种基本的模式来对类进行模拟，同时这也是 ECMAScript 6 类的基础。

<br />

### <a name="Class-Declarations"> 类声明（Class Declarations） </a>


在 ECMAScript 6 中类存在的最简单的形式就是类声明，它看起来和其他语言中的类无异。

<br />

#### 一个基本的类声明（A Basic Class Declaration）


类声明包含 class 关键字和紧随其后的命名。剩下的语法看起来和对象字面量中的简写方法类似，不过在它们之间不需要逗号分隔。例如，下面就是一个简单的类声明：

```
class PersonClass {

    // 等效于 PersonType 构造函数
    constructor(name) {
        this.name = name;
    }

    // 等效于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
}

let person = new PersonClass("Nicholas");
person.sayName();   // 输出 "Nicholas"

console.log(person instanceof PersonClass);     // true
console.log(person instanceof Object);          // true

console.log(typeof PersonClass);                    // "function"
console.log(typeof PersonClass.prototype.sayName);  // "function"
```

PersonClass 类声明的行为和上个例子中的 PersonType 类似。作为构造函数的替代，类声明允许你直接在类的内部使用命名为 constructor 的方法来定义构造函数。因为类的方法使用简写语法，所以不需要 function 关键字。至于其它的方法并没有什么特殊的含义，你可以随意添加的需要的方法。

> 自有属性：属性只出现在实例而不是原型上，而且只能由构造函数和方法来创建。在本例中，name 就是自有属性。我建议尽可能的将所有自有属性创建在构造函数中，这样当查找属性时可以做到一目了然。

有意思的是，类声明只是上例中自定义类型的语法糖。PersonClass 声明实际上创建了一个行为和 constructor 方法相同的构造函数，这也是 typeof PersonClass 返回 "function" 的原因。sayName() 在本例中作为 PersonClass.prototype 的方法，和上个示例中 sayName() 和 PersonType.prototype 关系一致。这些相似度允许你混合使用自定义类型和类而不需要纠结使用方式。

<br />

#### 为何使用类声明（Why to Use the Class Syntax）


先不管类和自定义类型之间的相似程度，有以下重要的几点差异是必须熟记的：

1. 类声明和函数定义不同，它们是不会被提升的。类声明的行为和 let 比较相似，所以当执行流作用到类声明之前类会存在于暂存性死区（temporal dead zone）内。
2. 类声明中的代码自动运行在严格模式下，同时没有任何办法可以手动切换到非严格模式。
3. 所有的方法都是不可枚举的（non-enumerable），这和自定义类型相比是个显著的差异，因为后者需要使用 Object.defineProperty() 才能定义不可枚举的方法。
4. 所有的方法都不能使用 new 来调用，因为它们没有内部方法 [[Construct]]。
5. 不使用 new 来调用类构造函数会抛出错误。
6. 试图在方法内部重写类名的行为会抛出错误。

With all of this in mind, the PersonClass declaration from the previous example is directly equivalent to the following code, which doesn’t use the class syntax:

记住了以上几点后，PersonClass 声明等同如下未使用类语法的代码：

```
// 完全等效于 PersonClass
let PersonType2 = (function() {

    "use strict";

    const PersonType2 = function(name) {

        // 确保方法由 new 调用
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.name = name;
    }

    Object.defineProperty(PersonType2.prototype, "sayName", {
        value: function() {

            // 确保方法不被 new 调用
            if (typeof new.target !== "undefined") {
                throw new Error("Method cannot be called with new.");
            }

            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    return PersonType2;
}());
```

首先要注意这里有两个 PersonType2 的声明，分别由 let 和 const 在外部作用域与 IIFE 内部创建。这说明了为何类名只能在类外部而不能由内部方法来改写。构造函数会检查 new.target 以确保由 new 来调用；否则会抛出一个错误。接下来，sayName() 方法被设定为不可枚举，同时检查 new.target 属性确保不被 new 来调用。最后将该构造函数返回。

该例说明了虽然 class 能做到的不使用新语法也能完全做到，但是相比之下 class 的语法明显更为简洁。

<br />

> #### 恒定的类名（Constant Class Names）
> 
>   在类的内部，类名被视为由 const 声明的常量。这意味着你只有在类的外部才能对类名进行重写。例如：

```
class Foo {
   constructor() {
       Foo = "bar";    // 执行后抛出错误
   }
}

// but this is okay after the class declaration
Foo = "baz";
```

> 在这段代码中，类构造函数内部的 Foo 和类外部的 Foo 并不是同一个绑定。内部的 Foo 行为类似于 const 声明的变量，因此它不能被重写，否则会抛出错误。不过外部的 Foo 被视作由 let 声明的变量，所以你可以不限次数的重写它。

<br />

### <a name="Class-Expressions"> 类表达式（Class Expressions） </a>


类和函数的相似之处在于它们都有两种存在形式：声明和表达式。函数和类声明由关键字开始（分别为 function 和 class），之后为标识符。函数表达式不要求在 function 关键字后添加标识符，类表达式同理。设计类表达式的目的主要是为了将它赋值给变量或者传参给函数。

<br />

#### 一个基本的类表达式（A Basic Class Expression）


以下代码是等效于上例中类声明的类表达式：

```
let PersonClass = class {

    // 等效于 PersonType 构造函数
    constructor(name) {
        this.name = name;
    }

    // 等效于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};

let person = new PersonClass("Nicholas");
person.sayName();   // 输出 "Nicholas"

console.log(person instanceof PersonClass);     // true
console.log(person instanceof Object);          // true

console.log(typeof PersonClass);                    // "function"
console.log(typeof PersonClass.prototype.sayName);  // "function"
```

由该例所示，类表达式不需要在 class 后面使用标识符。除此之外，类表达式在功能上和类声明是相同的。

In anonymous class expressions, as in the previous example, PersonClass.name is an empty string. When using a class declaration, PersonClass.name would be "PersonClass".

在该匿名类表达式中，PersonClass.name 是个空的字符串。如果使用了类声明，那么 PersonClass.name 的值为 "PersonClass"。

* 注：译者在这里测试发现表达式和声明都会返回类名，原文有误？

<br />

> 不论你使用类声明还是类表达式都取决于你的习惯。与函数声明和函数表达式不同，类声明和表达式都不会被提升。所以定义类的方式对运行时（runtime）的代码不会有什么影响。唯一显著的区别是匿名函数表达式的 name 属性为空字符串而类声明的 name 属性始终为类名（例如，使用类声明时 PersonClass.name 为 "PersonClass"）。

<br />

#### 具名类表达式（Named Class Expressions）


上一节的示例中使用了匿名类表达式，不过和函数表达式一样，你也可以使用具名类表达式。要想这么做，只需像这样在 class 关键字后添加标识符:

```
let PersonClass = class PersonClass2 {

    // 等效于 PersonType 构造函数
    constructor(name) {
        this.name = name;
    }

    // 等效于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};

console.log(typeof PersonClass);        // "function"
console.log(typeof PersonClass2);       // "undefined"
```

在本例中，类表达式被命名为 PersonClass2 。该标识符只存在于定义的类内部，所以它只能由类的方法（例如本例中的 sayName() 方法）使用。在类的外部 typeof PersonClass2 为 "undefined" 是因为不存在任何 PersonClass2 的绑定。为了更好地理解它，查看下列未使用 class 的等效代码：

```
// 完全等效于 PersonClass
let PersonClass = (function() {

    "use strict";

    const PersonClass2 = function(name) {

        // 确保方法由 new 调用
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.name = name;
    }

    Object.defineProperty(PersonClass2.prototype, "sayName", {
        value: function() {

            // 确保方法不由 new 调用
            if (typeof new.target !== "undefined") {
                throw new Error("Method cannot be called with new.");
            }

            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
    });

    return PersonClass2;
}());
```

具名类表达式的创建稍稍改变了 JavaScript 引擎内部的工作方式。对于类声明，外部的绑定（由 let 定义）和内部的绑定（由 const 定义）使用了相同的命名。而具名类表达式内部使用 const 定义该命名，所以定义的 PersonClass2 只能在类的内部使用。

虽然具名类表达式的行为和具名函数表达式有些差异，不过它们之间的相似程度还是很高的。它们都可以被当作值使用，于是这就给它们提供了我接下来要讲到的很多的潜力。


<br />

### <a name="Classes-as-First-Class-Citizens"> 作为一等公民的类（Classes as First-Class Citizens） </a>


在编程中，如果某些东西能作为值使用，那么它就被称为一等公民。这意味着它可以传入函数，或作为函数的返回值，亦或能赋值给变量。JavaScript 中的函数就是一等公民（有时它们被称作一等函数），这也是 JavaScript 独特的部分之一。

ECMAScript 6 继续延续该传统并让类也成为了一等公民，允许类以各种不同的方式使用。例如，它们可以作为函数的参数：

```
function createObject(classDef) {
    return new classDef();
}

let obj = createObject(class {

    sayHi() {
        console.log("Hi!");
    }
});

obj.sayHi();        // "Hi!"
```

本例中，createObject() 函数被传入了一个匿名类表达式作为参数，并在 new 调用参数之后返回了这个实例。变量 obj 存储了该它。

类表达式另一个有趣的使用方式是通过立即调用（immediately invoking）类构造函数来创建单例（singleton）。想要这么做的话，你必须联合使用 new 及类表达式并在末尾包含一个圆括号。例如：

```
let person = new class {

    constructor(name) {
        this.name = name;
    }

    sayName() {
        console.log(this.name);
    }

}("Nicholas");

person.sayName();       // "Nicholas"
```

在这里，一个匿名类表达式被创建并迅速执行。该模式允许你使用 class 语法来创建单例并消除可供窥察的类引用的存在（还记得 PersonClass 并非在类外而是在内部创建绑定的例子吗）。类表达式末尾的圆括号表示有函数被调用，同时你也可以在括号内传递参数。

本章目前出现的示例仅专注于类和包含的方法。其实你可以使用类似于对象字面量的语法来给类创建访问器属性。

<br />

### <a name="Accessor-Properties"> 访问器属性（Accessor Properties） </a>

While own properties should be created inside class constructors, classes allow you to define accessor properties on the prototype. To create a getter, use the keyword get followed by a space, followed by an identifier; to create a setter, do the same using the keyword set. For example:

```
class CustomHTMLElement {

    constructor(element) {
        this.element = element;
    }

    get html() {
        return this.element.innerHTML;
    }

    set html(value) {
        this.element.innerHTML = value;
    }
}

var descriptor = Object.getOwnPropertyDescriptor(CustomHTMLElement.prototype,\
 "html");
console.log("get" in descriptor);   // true
console.log("set" in descriptor);   // true
console.log(descriptor.enumerable); // false
```

In this code, the CustomHTMLElement class is made as a wrapper around an existing DOM element. It has both a getter and setter for html that delegates to the innerHTML method on the element itself. This accessor property is created on the CustomHTMLElement.prototype and, just like any other method would be, is created as non-enumerable. The equivalent non-class representation is:

```
// direct equivalent to previous example
let CustomHTMLElement = (function() {

    "use strict";

    const CustomHTMLElement = function(element) {

        // make sure the function was called with new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.element = element;
    }

    Object.defineProperty(CustomHTMLElement.prototype, "html", {
        enumerable: false,
        configurable: true,
        get: function() {
            return this.element.innerHTML;
        },
        set: function(value) {
            this.element.innerHTML = value;
        }
    });

    return CustomHTMLElement;
}());
```

As with previous examples, this one shows just how much code you can save by using a class instead of the non-class equivalent. The html accessor property definition alone is almost the size of the equivalent class declaration.

<br />

### <a name="Computed-Member-Names"> Computed Member Names </a>

The similarities between object literals and classes aren’t quite over yet. Class methods and accessor properties can also have computed names. Instead of using an identifier, use square brackets around an expression, which is the same syntax used for object literal computed names. For example:

```
let methodName = "sayName";

class PersonClass {

    constructor(name) {
        this.name = name;
    }

    [methodName]() {
        console.log(this.name);
    }
}

let me = new PersonClass("Nicholas");
me.sayName();           // "Nicholas"
```

This version of PersonClass uses a variable to assign a name to a method inside its definition. The string "sayName" is assigned to the methodName variable, and then methodName is used to declare the method. The sayName() method is later accessed directly.

Accessor properties can use computed names in the same way, like this:

```
let propertyName = "html";

class CustomHTMLElement {

    constructor(element) {
        this.element = element;
    }

    get [propertyName]() {
        return this.element.innerHTML;
    }

    set [propertyName](value) {
        this.element.innerHTML = value;
    }
}
```

Here, the getter and setter for html are set using the propertyName variable. Accessing the property by using .html only affects the definition.

You’ve seen that there are a lot of similarities between classes and object literals, with methods, accessor properties, and computed names. There’s just one more similarity to cover: generators.

<br />

### <a name="Generator-Methods"> Generator Methods </a>

When Chapter 8 introduced generators, you learned how to define a generator on an object literal by prepending a star (*) to the method name. The same syntax works for classes as well, allowing any method to be a generator. Here’s an example:

```
class MyClass {

    *createIterator() {
        yield 1;
        yield 2;
        yield 3;
    }

}

let instance = new MyClass();
let iterator = instance.createIterator();
```

This code creates a class called MyClass with a createIterator() generator method. The method returns an iterator whose values are hardcoded into the generator. Generator methods are useful when you have an object that represents a collection of values and you’d like to iterate over those values easily. Arrays, sets, and maps all have multiple generator methods to account for the different ways developers need to interact with their items.

While generator methods are useful, defining a default iterator for your class is much more helpful if the class represents a collection of values. You can define the default iterator for a class by using Symbol.iterator to define a generator method, such as:

```
class Collection {

    constructor() {
        this.items = [];
    }

    *[Symbol.iterator]() {
        yield *this.items.values();
    }
}

var collection = new Collection();
collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}

// Output:
// 1
// 2
// 3
```

This example uses a computed name for a generator method that delegates to the values() iterator of the this.items array. Any class that manages a collection of values should include a default iterator because some collection-specific operations require collections they operate on to have an iterator. Now, any instance of Collection can be used directly in a for-of loop or with the spread operator.

Adding methods and accessor properties to a class prototype is useful when you want those to show up on object instances. If, on the other hand, you’d like methods or accessor properties on the class itself, then you’ll need to use static members.

<br />

### <a name="Static-Members"> Static Members </a>

Adding additional methods directly onto constructors to simulate static members is another common pattern in ECMAScript 5 and earlier. For example:

```
function PersonType(name) {
    this.name = name;
}

// static method
PersonType.create = function(name) {
    return new PersonType(name);
};

// instance method
PersonType.prototype.sayName = function() {
    console.log(this.name);
};

var person = PersonType.create("Nicholas");
```

In other programming languages, the factory method called PersonType.create() would be considered a static method, as it doesn’t depend on an instance of PersonType for its data. ECMAScript 6 classes simplify the creation of static members by using the formal static annotation before the method or accessor property name. For instance, here’s the class equivalent of the last example:

```
class PersonClass {

    // equivalent of the PersonType constructor
    constructor(name) {
        this.name = name;
    }

    // equivalent of PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }

    // equivalent of PersonType.create
    static create(name) {
        return new PersonClass(name);
    }
}

let person = PersonClass.create("Nicholas");
```

The PersonClass definition has a single static method called create(). The method syntax is the same used for sayName() except for the static keyword. You can use the static keyword on any method or accessor property definition within a class. The only restriction is that you can’t use static with the constructor method definition.

> Static members are not accessible from instances. You must always access static members from the class directly.

<br />

### <a name="Inheritance-with-Derived-Classes"> Inheritance with Derived Classes </a>

Prior to ECMAScript 6, implementing inheritance with custom types was an extensive process. Proper inheritance required multiple steps. For instance, consider this example:

```
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

function Square(length) {
    Rectangle.call(this, length, length);
}

Square.prototype = Object.create(Rectangle.prototype, {
    constructor: {
        value:Square,
        enumerable: true,
        writable: true,
        configurable: true
    }
});

var square = new Square(3);

console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true
```

Square inherits from Rectangle, and to do so, it must overwrite Square.prototype with a new object created from Rectangle.prototype as well as call the Rectangle.call() method. These steps often confused JavaScript newcomers and were a source of errors for experienced developers.

Classes make inheritance easier to implement by using the familiar extends keyword to specify the function from which the class should inherit. The prototypes are automatically adjusted, and you can access the base class constructor by calling the super() method. Here’s the ECMAScript 6 equivalent of the previous example:

```
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }
}

class Square extends Rectangle {
    constructor(length) {

        // same as Rectangle.call(this, length, length)
        super(length, length);
    }
}

var square = new Square(3);

console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true
```

This time, the Square class inherits from Rectangle using the extends keyword. The Square constructor uses super() to call the Rectangle constructor with the specified arguments. Note that unlike the ECMAScript 5 version of the code, the identifier Rectangle is only used within the class declaration (after extends).

Classes that inherit from other classes are referred to as derived classes. Derived classes require you to use super() if you specify a constructor; if you don’t, an error will occur. If you choose not to use a constructor, then super() is automatically called for you with all arguments upon creating a new instance of the class. For instance, the following two classes are identical:

```
class Square extends Rectangle {
    // no constructor
}

// Is equivalent to

class Square extends Rectangle {
    constructor(...args) {
        super(...args);
    }
}
```

The second class in this example shows the equivalent of the default constructor for all derived classes. All of the arguments are passed, in order, to the base class constructor. In this case, the functionality isn’t quite correct because the Square constructor needs only one argument, and so it’s best to manually define the constructor.

<br />

> **NOTE**: There are a few things to keep in mind when using super():

> 1. You can only use super() in a derived class. If you try to use it in a non-derived class (a class that doesn’t use extends) or a function, it will throw an error.
2. You must call super() before accessing this in the constructor. Since super() is responsible for initializing this, attempting to access this before calling super() results in an error.
3. The only way to avoid calling super() is to return an object from the class constructor.

<br />

#### Shadowing Class Methods

The methods on derived classes always shadow methods of the same name on the base class. For instance, you can add getArea() to Square to redefine that functionality:

```
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }

    // override and shadow Rectangle.prototype.getArea()
    getArea() {
        return this.length * this.length;
    }
}
```

Since getArea() is defined as part of Square, the Rectangle.prototype.getArea() method will no longer be called by any instances of Square. Of course, you can always decide to call the base class version of the method by using the super.getArea() method, like this:

```
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }

    // override, shadow, and call Rectangle.prototype.getArea()
    getArea() {
        return super.getArea();
    }
}
```

Using super in this way works the same as the the super references discussed in Chapter 4 (see “Easy Prototype Access With Super References”). The this value is automatically set correctly so you can make a simple method call.

<br />

#### Inherited Static Members

If a base class has static members, then those static members are also available on the derived class. Inheritance works like that in other languages, but this is a new concept for JavaScript. Here’s an example:

```
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }

    getArea() {
        return this.length * this.width;
    }

    static create(length, width) {
        return new Rectangle(length, width);
    }
}

class Square extends Rectangle {
    constructor(length) {

        // same as Rectangle.call(this, length, length)
        super(length, length);
    }
}

var rect = Square.create(3, 4);

console.log(rect instanceof Rectangle);     // true
console.log(rect.getArea());                // 12
console.log(rect instanceof Square);        // false
```

In this code, a new static create() method is added to the Rectangle class. Through inheritance, that method is available as Square.create() and behaves in the same manner as the Rectangle.create() method.

<br />

#### Derived Classes from Expressions

Perhaps the most powerful aspect of derived classes in ECMAScript 6 is the ability to derive a class from an expression. You can use extends with any expression as long as the expression resolves to a function with [[Construct]] and a prototype. For instance:

```
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x instanceof Rectangle);    // true
```

Rectangle is defined as an ECMAScript 5-style constructor while Square is a class. Since Rectangle has [[Construct]] and a prototype, the Square class can still inherit directly from it.

Accepting any type of expression after extends offers powerful possibilities, such as dynamically determining what to inherit from. For example:

```
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};

function getBase() {
    return Rectangle;
}

class Square extends getBase() {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x instanceof Rectangle);    // true
```

The getBase() function is called directly as part of the class declaration. It returns Rectangle, making this example is functionally equivalent to the previous one. And since you can determine the base class dynamically, it’s possible to create different inheritance approaches. For instance, you can effectively create mixins:

```
let SerializableMixin = {
    serialize() {
        return JSON.stringify(this);
    }
};

let AreaMixin = {
    getArea() {
        return this.length * this.width;
    }
};

function mixin(...mixins) {
    var base = function() {};
    Object.assign(base.prototype, ...mixins);
    return base;
}

class Square extends mixin(AreaMixin, SerializableMixin) {
    constructor(length) {
        super();
        this.length = length;
        this.width = length;
    }
}

var x = new Square(3);
console.log(x.getArea());               // 9
console.log(x.serialize());             // "{"length":3,"width":3}"
```

In this example, mixins are used instead of classical inheritance. The mixin() function takes any number of arguments that represent mixin objects. It creates a function called base and assigns the properties of each mixin object to the prototype. The function is then returned so Square can use extends. Keep in mind that since extends is still used, you are required to call super() in the constructor.

The instance of Square has both getArea() from AreaMixin and serialize from SerializableMixin. This is accomplished through prototypal inheritance. The mixin() function dynamically populates the prototype of a new function with all of the own properties of each mixin. (Keep in mind that if multiple mixins have the same property, only the last property added will remain.)

<br />

> **NOTE**: Any expression can be used after extends, but not all expressions result in a valid class. Specifically, the following expression types cause errors:

> * null
* generator functions (covered in Chapter 8)

> In these cases, attempting to create a new instance of the class will throw an error because there is no [[Construct]] to call.

<br />

#### Inheriting from Built-ins

For almost as long as JavaScript arrays have existed, developers have wanted to create their own special array types through inheritance. In ECMAScript 5 and earlier, this wasn’t possible. Attempting to use classical inheritance didn’t result in functioning code. For example:

```
// built-in array behavior
var colors = [];
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined

// trying to inherit from array in ES5

function MyArray() {
    Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
    constructor: {
        value: MyArray,
        writable: true,
        configurable: true,
        enumerable: true
    }
});

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 0

colors.length = 0;
console.log(colors[0]);             // "red"
```

The console.log() output at the end of this code shows how using the classical form of JavaScript inheritance on an array results in unexpected behavior. The length and numeric properties on an instance of MyArray don’t behave the same as they do for the built-in array because this functionality isn’t covered either by Array.apply() or by assigning the prototype.

One goal of ECMAScript 6 classes is to allow inheritance from all built-ins. In order to accomplish this, the inheritance model of classes is slightly different than the classical inheritance model found in ECMAScript 5 and earlier, in two big ways:

* In ECMAScript 5 classical inheritance, the value of this is first created by the derived type (for example, MyArray), and then the base type constructor (like the Array.apply() method) is called. That means this starts out as an instance of MyArray and then is decorated with additional properties from Array.

* In ECMAScript 6 class-based inheritance, the value of this is first created by the base (Array) and then modified by the derived class constructor (MyArray). The result is that this starts with all the built-in functionality of the base and correctly receives all functionality related to it.

The following example shows a class-based special array in action:

```
class MyArray extends Array {
    // empty
}

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined
```

MyArray inherits directly from Array and therefore works like Array. Interacting with numeric properties updates the length property, and manipulating the length property updates the numeric properties. That means you can both properly inherit from Array to create your own derived array classes and inherit from other built-ins as well. With all this added functionality, ECMAScript 6 and derived classes have effectively removed the last special case of inheriting from built-ins, but that case is still worth exploring.

<br />

#### The Symbol.species Property

An interesting aspect of inheriting from built-ins is that any method that returns an instance of the built-in will automatically return a derived class instance instead. So, if you have a derived class called MyArray that inherits from Array, methods such as slice() return an instance of MyArray. For example:

```
class MyArray extends Array {
    // empty
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3);

console.log(items instanceof MyArray);      // true
console.log(subitems instanceof MyArray);   // true
```

In this code, the slice() method returns a MyArray instance. The slice() method is inherited from Array and returns an instance of Array normally. Behind the scenes, it’s the Symbol.species property that is making this change.

The Symbol.species well-known symbol is used to define a static accessor property that returns a function. That function is a constructor to use whenever an instance of the class must be created inside of an instance method (instead of using the constructor). The following builtin types have Symbol.species defined:

* Array
* ArrayBuffer (discussed in Chapter 10)
* Map
* Promise
* RegExp
* Set
* Typed Arrays (discussed in Chapter 10)

Each of these types has a default Symbol.species property that returns this, meaning the property will always return the constructor function. If you were to implement that functionality on a custom class, the code would look like this:

```
// several builtin types use species similar to this
class MyClass {
    static get [Symbol.species]() {
        return this;
    }

    constructor(value) {
        this.value = value;
    }

    clone() {
        return new this.constructor[Symbol.species](this.value);
    }
}
```

In this example, the Symbol.species well-known symbol is used to assign a static accessor property to MyClass. Note that there’s only a getter without a setter, because changing the species of a class isn’t possible. Any call to this.constructor[Symbol.species] returns MyClass. The clone() method uses that definition to return a new instance rather than directly using MyClass, which allows derived classes to override that value. For example:

```
class MyClass {
    static get [Symbol.species]() {
        return this;
    }

    constructor(value) {
        this.value = value;
    }

    clone() {
        return new this.constructor[Symbol.species](this.value);
    }
}

class MyDerivedClass1 extends MyClass {
    // empty
}

class MyDerivedClass2 extends MyClass {
    static get [Symbol.species]() {
        return MyClass;
    }
}

let instance1 = new MyDerivedClass1("foo"),
    clone1 = instance1.clone(),
    instance2 = new MyDerivedClass2("bar"),
    clone2 = instance2.clone();

console.log(clone1 instanceof MyClass);             // true
console.log(clone1 instanceof MyDerivedClass1);     // true
console.log(clone2 instanceof MyClass);             // true
console.log(clone2 instanceof MyDerivedClass2);     // false
```

Here, MyDerivedClass1 inherits from MyClass and doesn’t change the Symbol.species property. When clone() is called, it returns an instance of MyDerivedClass1 because this.constructor[Symbol.species] returns MyDerivedClass1. The MyDerivedClass2 class inherits from MyClass and overrides Symbol.species to return MyClass. When clone() is called on an instance of MyDerivedClass2, the return value is an instance of MyClass. Using Symbol.species, any derived class can determine what type of value should be returned when a method returns an instance.

For instance, Array uses Symbol.species to specify the class to use for methods that return an array. In a class derived from Array, you can determine the type of object returned from the inherited methods, such as:

```
class MyArray extends Array {
    static get [Symbol.species]() {
        return Array;
    }
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3);

console.log(items instanceof MyArray);      // true
console.log(subitems instanceof Array);     // true
console.log(subitems instanceof MyArray);   // false
```

This code overrides Symbol.species on MyArray, which inherits from Array. All of the inherited methods that return arrays will now use an instance of Array instead of MyArray.

In general, you should use the Symbol.species property whenever you might want to use this.constructor in a class method. Doing so allows derived classes to override the return type easily. Additionally, if you are creating derived classes from a class that has Symbol.species defined, be sure to use that value instead of the constructor.

<br />

### <a name="Using-newtarget-in-Class-Constructors"> Using new.target in Class Constructors </a>

In Chapter 3, you learned about new.target and how its value changes depending on how a function is called. You can also use new.target in class constructors to determine how the class is being invoked. In the simple case, new.target is equal to the constructor function for the class, as in this example:

```
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle);
        this.length = length;
        this.width = width;
    }
}

// new.target is Rectangle
var obj = new Rectangle(3, 4);      // outputs true
```

This code shows that new.target is equivalent to Rectangle when new Rectangle(3, 4) is called. Class constructors can’t be called without new, so the new.target property is always defined inside of class constructors. But the value may not always be the same. Consider this code:

```
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle);
        this.length = length;
        this.width = width;
    }
}

class Square extends Rectangle {
    constructor(length) {
        super(length, length)
    }
}

// new.target is Square
var obj = new Square(3);      // outputs false
```

Square is calling the Rectangle constructor, so new.target is equal to Square when the Rectangle constructor is called. This is important because it gives each constructor the ability to alter its behavior based on how it’s being called. For instance, you can create an abstract base class (one that can’t be instantiated directly) by using new.target as follows:

```
// abstract base class
class Shape {
    constructor() {
        if (new.target === Shape) {
            throw new Error("This class cannot be instantiated directly.")
        }
    }
}

class Rectangle extends Shape {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

var x = new Shape();                // throws error

var y = new Rectangle(3, 4);        // no error
console.log(y instanceof Shape);    // true
```

In this example, the Shape class constructor throws an error whenever new.target is Shape, meaning that new Shape() always throws an error. However, you can still use Shape as a base class, which is what Rectangle does. The super() call executes the Shape constructor and new.target is equal to Rectangle so the constructor continues without error.

> Since classes can’t be called without new, the new.target property is never undefined inside of a class constructor.

<br />

### <a name="Summary"> Summary </a>

ECMAScript 6 classes make inheritance in JavaScript easier to use, so you don’t need to throw away any existing understanding of inheritance you might have from other languages. ECMAScript 6 classes start out as syntactic sugar for the classical inheritance model of ECMAScript 5, but add a lot of features to reduce mistakes.

ECMAScript 6 classes work with prototypal inheritance by defining non-static methods on the class prototype, while static methods end up on the constructor itself. All methods are non-enumerable, a feature that better matches the behavior of built-in objects for which methods are typically non-enumerable by default. Additionally, class constructors can’t be called without new, ensuring that you can’t accidentally call a class as a function.

Class-based inheritance allows you to derive a class from another class, function, or expression. This ability means you can call a function to determine the correct base to inherit from, allowing you to use mixins and other different composition patterns to create a new class. Inheritance works in such a way that inheriting from built-in objects like Array is now possible and works as expected.

You can use new.target in class constructors to behave differently depending on how the class is called. The most common use is to create an abstract base class that throws an error when instantiated directly but still allows inheritance via other classes.

Overall, classes are an important addition to JavaScript. They provide a more concise syntax and better functionality for defining custom object types in a safe, consistent manner.