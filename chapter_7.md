# Set 与 Map（Sets and Maps）


JavaScript 在绝大部分历史时期内只有一种集合类型，那就是数组（可能有人会质疑所有的非类数组对象都是键值对的集合，但其实它们的用途和数组有根本上的区别）。数组在 JavaScript 中的使用方式和其它语言很相似，但是其它集合类型的缺乏导致数组也经常被当作队列（queues）和栈（stacks）来使用。因为数组的索引只能是数字类型，当开发者觉得非数字类型的索引是必要的时候会使用非数组对象。这项用法促进了以非类数组对象为基础的 set 和 map 集合类型的实现。

set 是非重复值的集合。你一般不会像在数组中那样来访问 set 中的某个值；相反，它常被用来检查某个值是否存在。map 则是包含键和对应值的集合，所以 map 中的每个元素都有两块数据，当指定键的时候对应的值会被读取。map 常用作缓存以便之后需要的时候能快速提取数据。虽然 ECMAScript 5 没有正式支持 set 和 map，开发者使用非类数组对象对它们进行了模拟。

ECMAScript 6 正式为 JavaScript 添加了 set 和 map，本章介绍了这两种集合类型你所需要了解的的全部信息。

首先，我会讲解开发者在 ECMAScript 6 之前是使用了什么样的解决方案来实现 set 和 map，而且为什么这些方案是有问题的。在这些重要的背景浮出水面之后，我会介绍 ECMAScript 6 中 set 和 map 的工作机制。

<br />

### ECMAScript 5 中的 set 与 map（Sets and Maps in ECMAScript 5）


在 ECMAScript 5 中，开发者使用对象属性来模拟 set 和 map，像这样：

```
let set = Object.create(null);

set.foo = true;

// 检查属性是否存在
if (set.foo) {

    // do something
}
```

本例中的 set 对象没有原型，确保它不会继承任何属性。在 ECMAScript 5 中使用对象属性来检查唯一值的用法十分普遍。当给 set 对象添加属性并设置值为 true 之后，条件判断语句（如本例中的 if 语句）可以轻松地检查某个值是否存在。

使用对象模拟的 set 和 map 之间唯一真正的区别是键值的类型。例如，下面的例子将对象做为 map 使用：

```
let map = Object.create(null);

map.foo = "bar";

// 提取属性值
let value = map.foo;

console.log(value);         // "bar"
```

该段代码将字符串 "bar" 赋值给了 foo 键。和 set 不同，map 大部分情况下被用来提取数据，而不是验证键是否存在

<br />

### 使用对象模拟的问题（Problems with Workarounds）


虽然在简单的情况下使用对象模拟的 set 和 map 没有太大的问题，不过当条件变得复杂时对象属性的限制很快就会暴露出来。例如，既然对象属性的类型必须为字符串，你必须保证键存储的值是唯一的。考虑如下的代码：

```
let map = Object.create(null);

map[5] = "foo";

console.log(map["5"]);      // "foo"
```

该例将 "foo" 值赋值给 5 这个数字类型键。在内部，数字类型的键会被转化为字符串，所以 map["5"] 和 map[5] 引用了相同的属性。当你想同时使用数字和字符串类型的键时，该内部实现是制造问题的根源。同样，当使用对象作为键的时候也会出现麻烦，例如：

```
let map = Object.create(null),
    key1 = {},
    key2 = {};

map[key1] = "foo";

console.log(map[key2]);     // "foo"
```

在这里，map[key2] 和 map[key1] 引用了相同的值。对象中的 key1 和 key2 被转化为字符串是因为对象的属性只能为该类型。既然对象的字符串表达形式是 "[object Object]"，那么 key1 和 key2 也不例外。开发者在一般的思维下都会自然认为不同的键名就代表不同的键，因此这里会造成难以察觉的错误。

默认的字符串转化使得对象很难被当作键来使用（该情况同样存在于 set）。

当键的值为假的情况下也会有一些问题。在条件判断中期待接收布尔值的位置，任何假值都会被自动转换为 false，比如 if 语句。只要你对要用的值稍加注意，一般这并不是个大的问题。例如，下面的代码：

```
let map = Object.create(null);

map.count = 1;

// 检查 "count" 是否存在或该值是否为假？
if (map.count) {
    // ...
}
```

该例中 map.count 的用法存在歧义。该语句的目的到底是检查 map.count 是否存在还是它的值是否为假。if 中的代码会执行是因为 1 被视为真值。然而 map.count 如果不存在或它的值为假则代码都不会被执行。

在大型应用的调试过程中这些都是棘手的问题，这也是 ECMAScript 6 添加 Set 和 Map 类型的主要原因之一。

<br />

> JavaScript 有 in 操作符可以在不读取值的情况下检查某个属性是否在对象中存在，如果是的话则返回 true。不过，该操作符还会检查对象的原型，这就使得该操作只有在对象不存在原型的条件下才是可靠的。即使这样，很多开发者都使用了上例中不当的方式而没有使用 in 。

<br />

### ECMAScript 6 中的 set（Sets in ECMAScript 6）


ECMAScript 6 中的 set 类型是一个博寒无重复元素的有序列表。Set 允许对内部某元素是否存在进行快速检查，使得元素的追踪操作效率更高。

<br />

#### 建立 Set 并添加项（Creating Sets and Adding Items）


set 由 new Set() 语句创建并通过调用 add() 方法来向 set 中添加项。你还可以查看 set 的 size 属性来获取项的数目：

```
let set = new Set();
set.add(5);
set.add("5");

console.log(set.size);    // 2
```

set 在比较值是否相等的时候不会做强制类型转换。这意味着在 set 可以同时包含数字 5 和 字符串 "5"（在 set 内部，该比较使用了第四章讨论过的 Object.is() 方法来决定两者的值是否相等）。你可以向 set 内部添加多个对象，它们的值会被认作是不相等的：

```
let set = new Set(),
    key1 = {},
    key2 = {};

set.add(key1);
set.add(key2);

console.log(set.size);    // 2
```

因为 key1 和 key2 不会转换为字符串，所以它们 set 认为两者都是唯一的（记住，如果它们被转换为字符串，那么值都是 "[object Object]"）。

如果 add() 方法由同一个参数调用了多次，那么首次之后的调用将会被忽略：

```
let set = new Set();
set.add(5);
set.add("5");
set.add(5);     // duplicate - this is ignored

console.log(set.size);    // 2
```

你可以使用数组来初始化一个 set，而且 Set 构造函数会确保使用数组中唯一存在的元素。例如：

```
let set = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
console.log(set.size);    // 5
```

在该例中，带有重复元素的数组被用来初始化 set 。虽然在数组中数字 5 出现了 4 次，但 set 中只将它存入一次。这项功能可以方便的转换 JSON 结构中已存在的数据。

<br />

> Set 构造函数实际上可以接收任何可迭代（iterable）对象作为属性。数组是默认的可迭代类型，所以它可以作为 Set 的参数。同样 set 和 map 也都是可迭代类型。Set 构造函数使用迭代器从参数中提取相应值。（可迭代类型与迭代器将在第八章讨论）

<br />

你可以使用 has() 方法来查看某个值是否在 set 中，像这样：


```
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true
console.log(set.has(6));    // false
```

在这里，因为 set 中不包含 6 ，所以 set.has(6) 会返回 false 。

<br />

#### 移除项（Removing Values）


将 set 中的值移除也是可以做到的。你可以使用 delete() 方法来销毁单个值，或者调用 clear() 方法来清空整个 set。下面的代码演示了这些操作：

```
let set = new Set();
set.add(5);
set.add("5");

console.log(set.has(5));    // true

set.delete(5);

console.log(set.has(5));    // false
console.log(set.size);      // 1

set.clear();

console.log(set.has("5"));  // false
console.log(set.size);      // 0
```

在调用 delete() 之后，只有 5 被移除；而 clear() 方法的执行使得 set 内部被清空。

这些方法提供了一个方便的机制用来追踪有序的唯一值。不过，给 set 添加项之后该怎样遍历它们呢？forEcah() 方法解决了这个问题。

<br />

#### Set 中的 forEach() 方法（The forEach() Method for Sets）


如果你曾处理过数组，那么 forEach() 方法你可能非常熟悉。ECMAScript 5 给数组添加了 forEach() 使得遍历操作每一项变得更加方便，不再需要循环。开发者在实践中享受到了好处并逐渐推广了它，于是 set 也顺势添加了该方法并且功能保持不变。

forEach() 方法接收一个含有三个参数的回调函数：

1. 下一位置的值
2. 和首个参数的值相同
3. 操作的 set 本身

比较奇怪的是 set 实现的 forEach() 方法和数组存在的差异在于接收的前两个参数是相同的。虽然这看来不大对劲，但却有意而为之的。

其它包含 forEach() 方法的对象（数组和 map）在该方法的回调函数中也会接收三个参数。前两个参数分别为下一位置的值和键（其中数组版本中的键为数字索引）。

但是 set 并不包含键。制定 ECMAScript 6 标准的相关人员本可以在 set 中设定 forEach() 的回调函数只接受两个参数。不过他们却另辟蹊径的找到了统一回调函数的办法：set 中的每一项既是键也是值。于是 set 为了和数组与 map 中的 forEach() 方法保持一致，将回调函数中的前两个参数设为相同。


除了参数个数的差异外，set 版本的 forEach() 和 数组基本相同。以下是一些代码来展示该方法是怎样工作的：

```
let set = new Set([1, 2]);

set.forEach(function(value, key, ownerSet) {
    console.log(key + " " + value);
    console.log(ownerSet === set);
});
```

该段代码将数组中的每一项添加到 set 中然后将值传递给 forEach() 的回调函数。回调函数每一次执行时，key 和 value 都是相同的，同时 ownerSet 则等同于 set。下面是输出结果： 

```
1 1
true
2 2
true
```

结果和使用数组是相同的，你可以给 forEach() 传入第二个参数 this 以便你在该方法中使用它。

```
let set = new Set([1, 2]);

let processor = {
    output(value) {
        console.log(value);
    },
    process(dataSet) {
        dataSet.forEach(function(value) {
            this.output(value);
        }, this);
    }
};

processor.process(set);
```

在上例中，processor.process() 方法在 set 上调用 forEach() 并将 this 值传递给回调函数。这对于 this.output() 会正确调用 processor.ouput() 是不可或缺的。forEach() 中的回调函数只需要首个参数， value，其它的都会忽略掉。你也可以使用箭头函数来替代作为第二个参数传递的 this，像这样：

```
let set = new Set([1, 2]);

let processor = {
    output(value) {
        console.log(value);
    },
    process(dataSet) {
        dataSet.forEach((value) => this.output(value));
    }
};

processor.process(set);
```

该例中的箭头函数会读取包含它的 process() 函数中的 this 值，所以 this.output() 会正确的调用 processor.output() 。

需要留心的是，虽然 set 可以很好的进行值的跟踪而且 forEach() 可以顺序处理其中的每一项，但是你无法使用下标来访问其中的值。如果你想这么做，最佳的方案还是将它转化为数组。

<br />

#### 将 set 转化为数组（Converting a Set to an Array）


将数组转化为 set 相当容易，你只需将数组传递给 Set 构造函数。使用扩展运算符将 set 转化为数组也并不复杂。第三章中介绍的扩展运算符（...）可以拆分数组中的项并传递给函数参数。你同样可以在可迭代对象上使用扩展运算符，例如 set，并将它转化为数组。例如：

```
let set = new Set([1, 2, 3, 3, 3, 4, 5]),
    array = [...set];

console.log(array);             // [1,2,3,4,5]
```

以上，set 在初始时读取了一个包含重复值的数组。set 清除掉了重复值之后，使用了扩展操作符将 set 中的项传入了一个新的数组当中。set 仍然包含创建时的项（1，2，3，4 和 5）。他们只是被复制到了新的数组中。

该实践在遇到清除已创建数组中的重复值的需求时特别好用。例如：

```
function eliminateDuplicates(items) {
    return [...new Set(items)];
}

let numbers = [1, 2, 3, 3, 3, 4, 5],
    noDuplicates = eliminateDuplicates(numbers);

console.log(noDuplicates);      // [1,2,3,4,5]
```

在 eliminateDuplicates() 函数中，set 只是被当作临时的代理以便在创建新的数组之前过滤掉重复项。

#### Weak Sets
The Set type could alternately be called a strong set, because of the way it stores object references. An object stored in an instance of Set is effectively the same as storing that object inside a variable. As long as a reference to that Set instance exists, the object cannot be garbage collected to free memory. For example:

```
let set = new Set(),
    key = {};

set.add(key);
console.log(set.size);      // 1

// eliminate original reference
key = null;

console.log(set.size);      // 1

// get the original reference back
key = [...set][0];
```

In this example, setting key to null clears one reference of the key object, but another remains inside set. You can still retrieve key by converting the set to an array with the spread operator and accessing the first item. That result is fine for most programs, but sometimes, it’s better for references in a set to disappear when all other references disappear. For instance, if your JavaScript code is running in a web page and wants to keep track of DOM elements that might be removed by another script, you don’t want your code holding onto the last reference to a DOM element. (That situation is called a memory leak.)

To alleviate such issues, ECMAScript 6 also includes weak sets, which only store weak object references and cannot store primitive values. A weak reference to an object does not prevent garbage collection if it is the only remaining reference.

<br />

#### Creating a Weak Set


Weak sets are created using the WeakSet constructor and have an add() method, a has() method, and a delete() method. Here’s an example that uses all three:

```
let set = new WeakSet(),
    key = {};

// add the object to the set
set.add(key);

console.log(set.has(key));      // true

set.delete(key);

console.log(set.has(key));      // false
```

Using a weak set is a lot like using a regular set. You can add, remove, and check for references in the weak set. You can also seed a weak set with values by passing an iterable to the constructor:

```
let key1 = {},
    key2 = {},
    set = new WeakSet([key1, key2]);

console.log(set.has(key1));     // true
console.log(set.has(key2));     // true
```

In this example, an array is passed to the WeakSet constructor. Since this array contains two objects, those objects are added into the weak set. Keep in mind that an error will be thrown if the array contains any non-object values, since WeakSet can’t accept primitive values.

<br />

#### Key Differences Between Set Types

The biggest difference between weak sets and regular sets is the weak reference held to the object value. Here’s an example that demonstrates that difference:

```
let set = new WeakSet(),
    key = {};

// add the object to the set
set.add(key);

console.log(set.has(key));      // true

// remove the last strong reference to key, also removes from weak set
key = null;
```

After this code executes, the reference to key in the weak set is no longer accessible. It is not possible to verify its removal because you would need one reference to that object to pass to the has() method. This can make testing weak sets a little confusing, but you can trust that the reference has been properly removed by the JavaScript engine.

These examples show that weak sets share some characteristics with regular sets, but there are some key differences. Those are:

1. In a WeakSet instance, the add() method, has() method, and delete() method all throw an error when passed a non-object.
2. Weak sets are not iterables and therefore cannot be used in a for-of loop.
3. Weak sets do not expose any iterators (such as the keys() and values() methods), so there is no way to programmatically determine the contents of a weak set.
4. Weak sets do not have a forEach() method.
5. Weak sets do not have a size property.


The seemingly limited functionality of weak sets is necessary in order to properly handle memory. In general, if you only need to track object references, then you should use a weak set instead of a regular set.

Sets give you a new way to handle lists of values, but they aren’t useful when you need to associate additional information with those values. That’s why ECMAScript 6 also adds maps.

<br />

