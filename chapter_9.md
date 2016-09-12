# 类（Introducing JavaScript Classes）


和大多数面向对象的语言（object-oriented programming language）不同，JavaScript 在诞生之初并不支持使用类和传统的类继承并作为主要的定义方式来创建相似或关联的对象。这很令开发者困惑，而且在早于 ECMAScript 1 到 ECMAScript 5 这段时期，很多库都创建了一些实用工具（utility）来让 JavaScript 从表层上支持类。尽管一些 JavaScript 开发者强烈主张该语言不需要类，但由于大量的库都对类做了实现，ECMAScript 6 也顺势将其引入。

在探索 ECMAScript 6 类的过程中，理解类的幕后机制是很有帮助的，所以本章会首先探讨开发者是怎样在 ECMAScript 5 中模拟类的实现的。而且在之后的小节内，你也会发现 ECMAScript 6 中的类和其他语言相比并不是完全等同的，目的是为了和 JavaScript 与生俱来的动态特性相配合。

<br />

### 本章小结
* [ECMAScript 5 中的类结构](#Class-Like-Structures-in-ECMAScript-5)
* [类声明](#Class-Declarations)
* [类表达式](#Class-Expressions)
* [作为一等公民的类](#Classes-as-First-Class-Citizens)
* [访问器属性](#Accessor-Properties)
* [动态计算的成员命名](#Computed-Member-Names)
* [生成器方法](#Generator-Methods)
* [静态成员](#Static-Members)
* [以派生类为继承方式](#Inheritance-with-Derived-Classes)
* [在类构造函数中使用 new.target](#Using-newtarget-in-Class-Constructors)
* [总结](#Summary)

<br />

### <a id="Class-Like-Structures-in-ECMAScript-5"> ECMAScript 5 中的类结构（Class-Like Structures in ECMAScript 5） </a>


在 ECMAScript 5 或更早的版本中，JavaScript 没有类。和类这个概念及行为最接近的是创建一个构造函数并在构造函数的原型上添加方法，这种实现也被称为自定义的类型创建，例如：

```js
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

### <a id="Class-Declarations"> 类声明（Class-Declarations） </a>


在 ECMAScript 6 中类存在的最简单的形式就是类声明，它看起来和其他语言中的类无异。

<br />

#### 一个基本的类声明（A Basic Class Declaration）


类声明包含 class 关键字和紧随其后的命名。剩下的语法看起来和对象字面量中的简写方法类似，不过在它们之间不需要逗号分隔。例如，下面就是一个简单的类声明：

```js
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

记住了以上几点后，PersonClass 声明等同如下未使用类语法的代码：

```js
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

```js
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

```js
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

* 注：译者在这里测试发现 Edge，Chrome 及 Opera 的匿名表达式都会返回类名，只有 FireFox 返回空字符串

<br />

> 不论你使用类声明还是类表达式都取决于你的习惯。与函数声明和函数表达式不同，类声明和表达式都不会被提升。所以定义类的方式对运行时（runtime）的代码不会有什么影响。唯一显著的区别是匿名函数表达式的 name 属性为空字符串而类声明的 name 属性始终为类名（例如，使用类声明时 PersonClass.name 为 "PersonClass"）。

<br />

#### 具名类表达式（Named Class Expressions）


上一节的示例中使用了匿名类表达式，不过和函数表达式一样，你也可以使用具名类表达式。要想这么做，只需像这样在 class 关键字后添加标识符:

```js
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

```js
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

### <a id="Classes-as-First-Class-Citizens"> 作为一等公民的类（Classes as First-Class Citizens） </a>


在编程中，如果某些东西能作为值使用，那么它就被称为一等公民。这意味着它可以传入函数，或作为函数的返回值，亦或能赋值给变量。JavaScript 中的函数就是一等公民（有时它们被称作一等函数），这也是 JavaScript 独特的部分之一。

ECMAScript 6 继续延续该传统并让类也成为了一等公民，允许类以各种不同的方式使用。例如，它们可以作为函数的参数：

```js
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

```js
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

### <a id="Accessor-Properties"> 访问器属性（Accessor Properties） </a>


虽然自有属性应该在类构造函数中创建，不过类也允许你在原型上创建访问器属性。为了创建一个 getter，需要使用 get 关键字，再加上一个空格和标识符；同理创建 setter 只需将上述步骤的关键字换成 set。例如：

```js
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

在该段代码中，CustomHTMLElement 是包含 DOM 元素的容器类。html 拥有 getter 和 setter 并分别代理 innerHTML 方法来操控元素。访问器属性在 CustomHTMLElement.prototype 上创建，和其它方法一样都是不可枚举的。未使用类的等效表示如下：

```js
// 完全等效于 PersonClass
let CustomHTMLElement = (function() {

    "use strict";

    const CustomHTMLElement = function(element) {

        // 确保方法由 new 调用
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

和之前的示例相同，本例只是展示了从非类语法切换到类语法能节省多少代码。前者定义 html 访问器属性的代码量几乎等同于使用类语法所需要的全部代码量。

<br />

### <a id="Computed-Member-Names"> 动态计算的成员命名（Computed Member Names） </a>


对象字面量和类之间的相似点还不仅仅只是这些。类方法和访问器属性同样可以使用动态计算的命名。你可以使用一对包含表达式的方括号来替代标识符，表示它们的命名是动态计算的。例如：

```js
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

该版本的 PersonClass 使用了一个变量赋值给内部的方法命名。"sayName" 会被赋值给 methodName 变量，于是 methodName 就用来声明方法。sayName() 方法在之后可以被直接使用。

访问器属性的命名也可以使用相同的方式来动态计算，像这样：

```js
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

在这里，getter 和 setter 绑定在以 propertyName 变量值命名的属性上。使用 .html 访问该属性只作用于定义本身。

你已经看到了类与对象字面量之间从方法，访问器属性和动态计算命名等多个方面的相似之处。除此之外还有一处值得一提：生成器。

<br />

### <a id="Generator-Methods"> 生成器方法（Generator Methods） </a>


在第八章介绍生成器之后，你懂得了在对象字面量内部的方法名之前使用星号（*）来创建生成器。同样类也允许在内部使用该语法将任何方法改造成生成器。如下所示：

```js
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

该段代码创建了 MyClass 类并带有 createIterator() 生成器方法。该方法返回了一个经过硬编码（hardcoded）的迭代器。当你想要一个具有集合性质的对象并能轻松迭代包含值的时候，生成器方法相当有用。数组，set 和 map 都拥有多个生成器方法以便给开发者提供多种选择来操作包含的项。

尽管生成器方法很有用，不过在集合性质的类中定义一个默认迭代器就再好不过了。你可以给类的 Symbol.iterator 定义一个生成器方法来设置类的默认迭代器，例如：

```js
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

// 输出:
// 1
// 2
// 3
```

该例使用了动态命名的生成器方法来代理 this.items 数组的 values() 迭代器。任何管理集合的类都应该包含一个默认迭代器，因为一些集合专属的操作要求作用的集合必须包含迭代器。现在，该集合的任何实例都能被 for-of 循环或扩展运算符直接使用。

为了对象实例能够使用它们，在类的原型上添加方法和访问器属性非常有用。不过令另一方面，当你想让方法和访问器属性只能由类自己使用时，你需要的是静态成员。

<br />

### <a id="Static-Members"> 静态成员（Static Members） </a>


给构造函数直接添加方法来模拟静态成员这在 ECMAScript 5 和更早的版本中是个常见的模式。例如：

```js
function PersonType(name) {
    this.name = name;
}

// 静态方法
PersonType.create = function(name) {
    return new PersonType(name);
};

// 实例方法
PersonType.prototype.sayName = function() {
    console.log(this.name);
};

var person = PersonType.create("Nicholas");
```

在其它编程语言中，PersonType.create() 这个工厂方法会被视为是静态的，因为他不依赖示例中的数据。ECMAScript 6 的类通过在方法和访问器属性之前使用正式的 static 注解简化了静态方法的创建。例如，下例中的类和上例相比是等效的：

```js
class PersonClass {

    // 等效于 PersonType 构造函数
    constructor(name) {
        this.name = name;
    }

    // 等效于 PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }

    // 等效于 PersonType.create
    static create(name) {
        return new PersonClass(name);
    }
}

let person = PersonClass.create("Nicholas");
```

在 PersonClass 的定义中包含一个 create() 静态方法。它和直接创建 sayName() 方法的差异在 static 这个关键字。你可以将 static 关键字添加给类中定义的除 constructor 之外的任何方法或访问器属性。

<br />

> 静态成员不能被实例访问。你必须通过类本身来使用它们。

<br />

### <a id="Inheritance-with-Derived-Classes"> 以派生类为继承方式（Inheritance with Derived Classes） </a>


ECMAScript 6 之前，实现自定义类型的继承是个昂贵的过程。正确的继承方式包含多个步骤。例如，考虑下面的例子：

```js
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

Square 继承了 Rectangle。方法是通过创建了以 Rectangle.prototype 为原型创建的新对象对 Square.prototype 进行重写，并在构造函数上调用 Rectangle.call() 方法。

类通过 extends 关键字并指定要继承的函数或类名简单地实现了继承。原型会自动调整，并可以通过 super() 方法来访问基类构造函数。这里是 ECMAScript 6 针对上例的等效写法：

```js
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

        // 等效于 Rectangle.call(this, length, length)
        super(length, length);
    }
}

var square = new Square(3);

console.log(square.getArea());              // 9
console.log(square instanceof Square);      // true
console.log(square instanceof Rectangle);   // true
```

本次 Square 类使用了 extends 关键字继承了 Rectangle 。Square 构造函数使用 super() 和指定的参数来调用 Rectangle 的构造函数。注意和 ECMAScript 5 中的写法不同，Rectangle 只在类声明中使用（extends 之后）。

类如果继承了其它类，那么它就是派生类。如果在派生类中定义 constructor 则必须使用 super()，否则会发生错误。如果你选择不使用 constructor，那么会自动调用 super() 和传入构造函数的参数以创建类的实例。例如，下面的两个类是等效的：

```js
class Square extends Rectangle {
    // no constructor
}

// 等效于

class Square extends Rectangle {
    constructor(...args) {
        super(...args);
    }
}
```

该例的第二个类展示了所有派生类默认构造函数的等效写法。所有的参数按顺序传递给基类的构造函数。在本例的情况下，square 其实只需要一个参数，所以默认构造函数的行为并不十分恰当，最好手动定义自己的构造函数。

<br />

> **注意**: 使用 super() 需要牢记以下几点：

> 1. 你只能在派生类中使用 super()，否则（没有使用 extends 的类或函数）一个错误会被抛出。
2. 你必须在构造函数的起始位置调用 super()，因为它会初始化 this。任何在 super() 之前访问 this 的行为都会造成错误。
3. 在类构造函数中，唯一能避免调用 super() 的办法是返回一个对象。

<br />

#### 隐藏类方法（Shadowing Class Methods）


派生类中的方法总是会屏蔽基类中的同名方法。例如，你可以在 Square 中重新定义 getArea()：

```js
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }

    // 重写并隐藏 Rectangle.prototype.getArea()
    getArea() {
        return this.length * this.length;
    }
}
```

既然 getArea() 已成为了 Square 自身的一份子，Rectangle.prototype.getArea() 方法就不再会被 Square 的实例调用。当然，你可以随时使用 super.getArea() 方法来调用基类中的同名方法，像这样：

```js
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

在这里 super 的行为和第四章中讨论过的 super 引用是相同的（查看 “使用 super 引用来方便获取 prototype ” 一节）。this 值会被自动且正确的绑定，所以你可以简单的调用方法。

<br />

#### 静态成员继承（Inherited Static Members）


如果基类中包含静态成员，那么派生类也可以直接使用它们。这里的继承机制和其它语言相同，不过对 JavaScript 而言它是个新的概念。如下所示：

```js
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

        // 等效于 Rectangle.call(this, length, length)
        super(length, length);
    }
}

var rect = Square.create(3, 4);

console.log(rect instanceof Rectangle);     // true
console.log(rect.getArea());                // 12
console.log(rect instanceof Square);        // false
```

该段代码中，一个新的 create() 静态方法添加给了 Rectangle 类。通过继承，该方法由 Square.create() 调用并且行为等同于 Rectangle.create() 。

<br />

#### 继承表达式的派生类（Derived Classes from Expressions）


或许 ECMAScript 6 派生类最强大的地方在于它们可以继承一个表达式。只要该表达式的计算结果包含 [[Construct]] 的函数和一个原型，那么就可以使用 extends 来继承它。例如：

```js
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

Rectangle 是 ECMAScript 5 风格的构造函数而 Square 是一个类。因为 Rectangle 包含 [[Construct]] 和一个原型，所以 Square 类依旧能直接继承它。

extends 可接受任意类型表达式的特性给类继承提供了无限的可能性，你可以动态决定要继承的内容。例如：

```js
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

getBase() 函数被直接调用的同时还作为类声明的一部分。该函数返回 Rectangle，使得它等效于上一个例子。而且，由于基类可以是动态决定，那么创建多种不同的继承方式也是有可能的。例如，你可以有效地创建 mixin：

```js
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

本例使用了 mixin 而不是传统的继承。mixin() 函数接收任意个数的混入对象参数。它创建了一个 base 函数并将想要混合的对象中的属性添加给了 base 的原型。该函数之后会被返回因此 Square 才可能使用 extends 来继承它。需要注意的是既然使用了 extends，那么构造函数必须要调用 super()。

Square 的实例中包含了从 AreaMixin 和 SerializableMixin 分别得到的 getArea() 和 serialize() 方法。这个是通过原型继承来实现的。mixin() 函数动态地将每一个 mixin 中的属性添加到一个新函数的原型上（注意的是如果多个 mixin 包含相同的属性，只有最后的会被采用）。

<br />

> **注意**: 虽然 extends 后可以添加任何表达式，但是不是所有的表达式最后都能产生一个有效的类。尤其是以下的几种会造成错误：

> * null
* 生成器函数 (第八章已讲述)

> 使用这些表达式创建类实例会发生错误的原因是它们不包含 [[Construct]] 。

<br />

#### 内置对象的继承（Inheriting from Built-ins）


自 JavaScript 数组存在的那天起，开发者就想通过使用继承的方式来定义特殊的数组类型。在 ECMAScript 5 和更早的版本中，这是不可能做到的。使用传统的继承方式无法获得想要的功能。例如：

```js
// 内置数组的行为
var colors = [];
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined

// 尝试使用 ES5 的继承方式

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

console.log() 在代码尾部的输出揭露了使用传统形式的 JavaScript 继承会得到意想不到的结果。MyArray 实例中的 length 和 numeric 属性的行为并不和内置的数组一致，因为这些特性仅仅通过调用 Array.apply() 或重新给原型赋值是做不到的。

ECMAScript 6 类的目标之一就是允许继承内置的对象。为了实现这个需求，类的继承模型和 ECMAScript 5 及更早版本的传统继承模型相比有些细微的不同，体现在以下方面：

* 在 ECMAScript 5 传统的继承中，this 首先由派生的类型（例如，MyArray）创建，之后才会调用基类的构造函数（类似于 Array.apply()）。这意味着 this 一开始就是 MyArray 的实例，Array 中的属性负责装饰（decorate）它。

* 在 ECMAScript 6 的类继承中，this 首先由基类创建并在之后由派生的类构造函数（MyArray）进行修改。于是 this 一开始便拥有内置对象的全部功能并在之后能接收其它功能扩展。


下面的示例展示了通过类继承的特殊数组：

```js
class MyArray extends Array {
    // empty
}

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length);         // 1

colors.length = 0;
console.log(colors[0]);             // undefined
```

Array 由 MyArray 直接继承因此后者的行为和数组一致，包括与数组索引的交互引起 length 属性的变化以及操作length 属性造成数字索引值的更改。这意味着你不仅可以正确的继承数组并定义自己的派生类，对其它内置对象的继承也是同理。该特性的添加使得 ECMAScript 6 和派生类消除了关于继承内置类型所有的特殊情况，不过这些情况依旧值得推敲。*

<br />

#### Symbol.species 属性（The Symbol.species Property）


关于继承内置对象的一个有趣现象是，任何返回内置对象实例的方法会自动返回派生类的实例。所以，如果你有一个继承数组的 MyArray 派生类，类似 slice() 的方法会返回 MyArray 的实例。例如：

```js
class MyArray extends Array {
    // empty
}

let items = new MyArray(1, 2, 3, 4),
    subitems = items.slice(1, 3);

console.log(items instanceof MyArray);      // true
console.log(subitems instanceof MyArray);   // true
```

该段代码中，slice() 方法返回一个 MyArray 实例。一般来讲，slice() 方法是继承自数组，应该返回数组的实例。之所以发生了这样的改变归咎于 Symbol.species 属性在幕后的操作。

知名的 symbol 类型 Symbol.species 被用来定义一个返回函数的静态访问器属性。该属性返回的是构造函数并可随时在实例方法中创建该类的实例（而不是使用构造函数）。以下的内置类型包含 Symbol.species 的定义：

* Array
* ArrayBuffer (第十章中讨论)
* Map
* Promise
* RegExp
* Set
* Typed Arrays (第十章中讨论)


以上的每个类型都包含返回 this 的 Symbol.species 属性，意味着该属性总是会返回正确的构造函数。如果你想在自定义的类中实现它，那么代码应该类似于下例：

```js
// 一些内置类型采取类似如下的方案
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

在本例中，Symbol.species 将一个静态访问器属性添加给 MyClass 。注意这里只有 getter 而未有 setter，因为更改类的 species 是不可能的。任何对 this.constructor[Symbol.species] 的调用都会返回 MyClass。clone() 方法没有直接使用 MyClass，而是调用了 Symbol.species 并返回一个新实例以便让派生类重写该属性。例如：

```js
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
    // 空代码块
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

在这里，MyDerivedClass1 继承了 MyClass 且并没有修改 Symbol.species 属性。当 clone() 调用后，它返回了 MyDerivedClass1 的实例，这正是由 this.constructor[Symbol.species] 所设定的。MyDerivedClass2 类继承了 MyClass 并将 Symbol.species 的返回值重写为 MyClass。即使 clone() 是由 MyDerivedClass2 调用的，它返回的依然是 MyClass 的实例。任何使用 Symbol.species 的派生类都能自行决定方法返回的实例类型。

例如，数组使用 Symbol.species 来指定返回值为数组的方法的返回类型。如果一个类是数组的派生，那么你可以决定继承的方法返回哪种类型，例如：*

```js
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

该段代码中 MyArray 继承了数组并重写了 Symbol.species 。现在所有继承的返回数组的方法它们的返回类型不再是 MyArray 而是 Array 。

一般来讲，如果你的类方法使用了 this.constructor，那么你应该给这个类设定 Symbol.species 属性。这样做能允许派生类方便地重写返回类型。此外，如果你想创建一个派生类来继承了一个已定义 Symbol.species 属性的基类，那么确保基类在返回该类实例的方法中使用该属性，而不是构造函数。

<br />

### <a id="Using-newtarget-in-Class-Constructors"> 在类构造函数中使用 new.target（Using new.target in Class Constructors） </a>


在第三章你已经了解了 new.target 是如何根据被调用的函数来决定自身的值。你也可以使用在类的构造函数中使用它来判断类是被如何调用的。简单情况下，new.target 的值等于类的构造函数，如下所示：

```js
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle);
        this.length = length;
        this.width = width;
    }
}

// new.target 为 Rectangle
var obj = new Rectangle(3, 4);      // 输出为 true
```

该段代码说明了在 new Rectangle(3, 4) 调用后， new.target 等价于 Rectangle 。类构造函数只能被 new 调用，所以 new.target 属性值总是由类内部的构造函数来决定。不过该值不总是相同的。考虑如下的代码：

```js
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

// new.target 为 Square
var obj = new Square(3);      // 输出为 false
```

Square 调用了 Rectangle 的构造函数，所以该时刻 new.target 等于 Square。这一点很重要，因为每个构造函数都能根据被调用的方式来修改自己的行为。例如，你可以使用 new.target 来创建一个不能被直接调用的抽象基类（abstract base class）。

```js
// 抽象基类
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

var x = new Shape();                // 抛出错误

var y = new Rectangle(3, 4);        // 没有错误发生
console.log(y instanceof Shape);    // true
```

在该例中，Shape 类的构造函数会在 new.target 为 Shape 时抛出错误，意味着 new Shape() 是不能使用的。然而，你可以将 Shape 作为基类，正如 Rectangle 那样。super() 调用会执行 Shape 的构造函数而且 new.target 等于 Rectangle，所以该构造函数不会发生错误并完整执行。

<br />

> 因为类必须由 new 来调用，所以 new.target 属性不会为 undefined，而是某个类的构造函数。

<br />

### <a id="Summary"> 总结（Summary） </a>


ECMAScript 6 的类使得 JavaScript 中的继承更容易实现，你再也不必扔掉其它语言中继承方面的相关知识。ECMAScript 6 中的类是 ECMAScript 5 传统继承模型的语法糖，但是也添加和消除了很多特性与错误。

ECMAScript 6 的类通过在类原型上定义非静态方法来延续原型继承的工作机制，同时静态方法会由类本身持有。所有的方法都不可枚举，这也是为了符合内置对象所有方法的默认行为。另外，类构造函数必须由 new 来调用，以防你不小心将类当作函数。

以类为基础的继承允许你将类派生自另一个类，函数或表达式。该特性使你能通过调用一个函数来决定继承的基类，同时还可以由 mixin 或其它协作模式来创建一个新类。此外，内置对象如数组的继承现在也能正常工作。

你可以在类构造函数内部使用 new.target 来根据具体的调用方式做出不同的行为。最常见的用法是创建一个直接调用会报错但是可以由其它类继承的抽象基类。

总之，类的添加对 JavaScript 至关重要。它提供了更简洁的语法和更好的实用性来定义一个安全且拥有一致表现形式的对象类型。

<br />