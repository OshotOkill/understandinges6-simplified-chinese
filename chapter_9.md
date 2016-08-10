# 类（Introducing JavaScript Classes）

Class-Like Structures in ECMAScript 5
Class Declarations
Class Expressions
Classes as First-Class Citizens
Accessor Properties
Computed Member Names
Generator Methods
Static Members
Inheritance with Derived Classes
Using new.target in Class Constructors
Summary

Unlike most formal object-oriented programming languages, JavaScript didn’t support classes and classical inheritance as the primary way of defining similar and related objects when it was created. This left many developers confused, and from pre-ECMAScript 1 all the way through ECMAScript 5, many libraries created utilities to make JavaScript look like it supports classes. While some JavaScript developers do feel strongly that the language doesn’t need classes, the number of libraries created specifically for this purpose led to the inclusion of classes in ECMAScript 6.

While exploring ECMAScript 6 classes, it’s helpful to understand the underlying mechanisms that classes use, so this chapter starts by discussing how ECMAScript 5 developers achieved class-like behavior. As you will see after that, however, ECMAScript 6 classes aren’t exactly the same as classes in other languages. There’s a uniqueness about them that embraces the dynamic nature of JavaScript.

<br />

### Class-Like Structures in ECMAScript 5

In ECMAScript 5 and earlier, JavaScript had no classes. The closest equivalent to a class was creating a constructor and then assigning methods to the constructor’s prototype, an approach typically called creating a custom type. For example:

```
function PersonType(name) {
    this.name = name;
}

PersonType.prototype.sayName = function() {
    console.log(this.name);
};

let person = new PersonType("Nicholas");
person.sayName();   // outputs "Nicholas"

console.log(person instanceof PersonType);  // true
console.log(person instanceof Object);      // true
```

In this code, PersonType is a constructor function that creates a single property called name. The sayName() method is assigned to the prototype so the same function is shared by all instances of the PersonType object. Then, a new instance of PersonType is created via the new operator. The resulting person object is considered an instance of PersonType and of Object through prototypal inheritance.

This basic pattern underlies a lot of the class-mimicking JavaScript libraries, and that’s where ECMAScript 6 classes start.

<br />

### Class Declarations

The simplest class form in ECMAScript 6 is the class declaration, which looks similar to classes in other languages.

<br />

#### A Basic Class Declaration
Class declarations begin with the class keyword followed by the name of the class. The rest of the syntax looks similar to concise methods in object literals, without requiring commas between them. For example, here’s a simple class declaration:

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
}

let person = new PersonClass("Nicholas");
person.sayName();   // outputs "Nicholas"

console.log(person instanceof PersonClass);     // true
console.log(person instanceof Object);          // true

console.log(typeof PersonClass);                    // "function"
console.log(typeof PersonClass.prototype.sayName);  // "function"
```

The class declaration PersonClass behaves quite similarly to PersonType from the previous example. But instead of defining a function as the constructor, class declarations allow you to define the constructor directly inside the class with the special constructor method name. Since class methods use the concise syntax, there’s no need to use the function keyword. All other method names have no special meaning, so you can add as many methods as you want.

> Own properties, properties that occur on the instance rather than the prototype, can only be created inside a class constructor or method. In this example, name is an own property. I recommend creating all possible own properties inside the constructor function so a single place in the class is responsible for all of them.

Interestingly, class declarations are just syntactic sugar on top of the existing custom type declarations. The PersonClass declaration actually creates a function that has the behavior of the constructor method, which is why typeof PersonClass gives "function" as the result. The sayName() method also ends up as a method on PersonClass.prototype in this example, similar to the relationship between sayName() and PersonType.prototype in the previous example. These similarities allow you to mix custom types and classes without worrying too much about which you’re using.

<br />

#### Why to Use the Class Syntax

Despite the similarities between classes and custom types, there are some important differences to keep in mind:

1. Class declarations, unlike function declarations, are not hoisted. Class declarations act like let declarations and so exist in the temporal dead zone until execution reaches the declaration.
2. All code inside of class declarations runs in strict mode automatically. There’s no way to opt-out of strict mode inside of classes.
3. All methods are non-enumerable. This is a significant change from custom types, where you need to use Object.defineProperty() to make a method non-enumerable.
4. All methods lack an internal [[Construct]] method and will throw an error if you try to call them with new.
5. Calling the class constructor without new throws an error.
6. Attempting to overwrite the class name within a class method throws an error.

With all of this in mind, the PersonClass declaration from the previous example is directly equivalent to the following code, which doesn’t use the class syntax:

```
// direct equivalent of PersonClass
let PersonType2 = (function() {

    "use strict";

    const PersonType2 = function(name) {

        // make sure the function was called with new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.name = name;
    }

    Object.defineProperty(PersonType2.prototype, "sayName", {
        value: function() {

            // make sure the method wasn't called with new
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

First, notice that there are two PersonType2 declarations: a let declaration in the outer scope and a const declaration inside the IIFE. This is how class methods are forbidden from overwriting the class name while code outside the class is allowed to do so. The constructor function checks new.target to ensure that it’s being called with new; if not, an error is thrown. Next, the sayName() method is defined as nonenumerable, and the method checks new.target to ensure that it wasn’t called with new. The final step returns the constructor function.

This example shows that while it’s possible to do everything classes do without using the new syntax, the class syntax simplifies all of the functionality significantly.

<br />

> #### Constant Class Names
The name of a class is only specified as if using const inside of the class itself. That means you can overwrite the class name outside of the class but not inside a class method. For example:

```
class Foo {
   constructor() {
       Foo = "bar";    // throws an error when executed
   }
}

// but this is okay after the class declaration
Foo = "baz";
```

> In this code, the Foo inside the class constructor is a separate binding from the Foo outside the class. The internal Foo is defined as if it’s a const and cannot be overwritten. An error is thrown when the constructor attempts to overwrite Foo with any value. But since the external Foo is defined as if it’s a let declaration, you can overwrite its value at any time.

<br />

### Class Expressions

Classes and functions are similar in that they have two forms: declarations and expressions. Function and class declarations begin with an appropriate keyword (function or class, respectively) followed by an identifier. Functions have an expression form that doesn’t require an identifier after function, and similarly, classes have an expression form that doesn’t require an identifier after class. These class expressions are designed to be used in variable declarations or passed into functions as arguments.

<br />

#### A Basic Class Expression

Here’s the class expression equivalent of the previous PersonClass examples, followed by some code that uses it:

```
let PersonClass = class {

    // equivalent of the PersonType constructor
    constructor(name) {
        this.name = name;
    }

    // equivalent of PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};

let person = new PersonClass("Nicholas");
person.sayName();   // outputs "Nicholas"

console.log(person instanceof PersonClass);     // true
console.log(person instanceof Object);          // true

console.log(typeof PersonClass);                    // "function"
console.log(typeof PersonClass.prototype.sayName);  // "function"
```

As this example demonstrates, class expressions do not require identifiers after class. Aside from the syntax, class expressions are functionally equivalent to class declarations.

In anonymous class expressions, as in the previous example, PersonClass.name is an empty string. When using a class declaration, PersonClass.name would be "PersonClass".

<br />

> Whether you use class declarations or class expressions is mostly a matter of style. Unlike function declarations and function expressions, both class declarations and class expressions are not hoisted, and so the choice has little bearing on the runtime behavior of the code. The only significant difference is that anonymous class expressions have a name property that is an empty string while class declarations always have a name property equal to the class name (for instance, PersonClass.name is "PersonClass" when using a class declaration).

<br />

#### Named Class Expressions

The previous section used an anonymous class expression in the example, but just like function expressions, you can also name class expressions. To do so, include an identifier after the class keyword like this:

```
let PersonClass = class PersonClass2 {

    // equivalent of the PersonType constructor
    constructor(name) {
        this.name = name;
    }

    // equivalent of PersonType.prototype.sayName
    sayName() {
        console.log(this.name);
    }
};

console.log(typeof PersonClass);        // "function"
console.log(typeof PersonClass2);       // "undefined"
```

In this example, the class expression is named PersonClass2. The PersonClass2 identifier exists only within the class definition so that it can be used inside the class methods (such as the sayName() method in this example). Outside the class, typeof PersonClass2 is "undefined" because no PersonClass2 binding exists there. To understand why this is, look at an equivalent declaration that doesn’t use classes:

```
// direct equivalent of PersonClass named class expression
let PersonClass = (function() {

    "use strict";

    const PersonClass2 = function(name) {

        // make sure the function was called with new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }

        this.name = name;
    }

    Object.defineProperty(PersonClass2.prototype, "sayName", {
        value: function() {

            // make sure the method wasn't called with new
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

Creating a named class expression slightly changes what’s happening in the JavaScript engine. For class declarations, the outer binding (defined with let) has the same name as the inner binding (defined with const). A named class expression uses its name in the const definition, so PersonClass2 is defined for use only inside the class.

While named class expressions behave differently from named function expressions, there are still a lot of similarities between the two. Both can be used as values, and that opens up a lot of possibilities, which I’ll cover next.

<br />

### Classes as First-Class Citizens

In programming, something is said to be a first-class citizen when it can be used as a value, meaning it can be passed into a function, returned from a function, and assigned to a variable. JavaScript functions are first-class citizens (sometimes they’re just called first class functions), and that’s part of what makes JavaScript unique.

ECMAScript 6 continues this tradition by making classes first-class citizens as well. That allows classes to be used in a lot of different ways. For example, they can be passed into functions as arguments:

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

In this example, the createObject() function is called with an anonymous class expression as an argument, creates an instance of that class with new, and returns the instance. The variable obj then stores the returned instance.

Another interesting use of class expressions is creating singletons by immediately invoking the class constructor. To do so, you must use new with a class expression and include parentheses at the end. For example:

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

Here, an anonymous class expression is created and then executed immediately. This pattern allows you to use the class syntax for creating singletons without leaving a class reference available for inspection. (Remember that PersonClass only creates a binding inside of the class, not outside.) The parentheses at the end of the class expression are the indicator that you’re calling a function while also allowing you to pass in an argument.

The examples in this chapter so far have focused on classes with methods. But you can also create accessor properties on classes using a syntax similar to object literals.

