# Set 与 Map（Sets and Maps）


JavaScript 在绝大部分历史时期内只有一种集合类型，那就是数组（可能有人会质疑所有的非类数组对象都是键值对的集合，但其实它们的用途和数组有根本上的区别）。数组在 JavaScript 中的使用方式和其它语言很相似，但是其它集合类型的缺乏导致数组也经常被当作队列（queues）和栈（stacks）来使用。因为数组的索引只能是数字类型，当开发者觉得非数字类型的索引是必要的时候会使用非数组对象。这项用法促进了以非类数组对象为基础的 set 和 map 集合类型的实现。

set 是非重复值的集合。你一般不会像在数组中那样来访问 set 中的某个值；相反，它常被用来检查某个值是否存在。map 则是包含键和对应值的集合，所以 map 中的每个元素都有两块数据，当指定键的时候对应的值会被读取。map 常用作缓存以便之后需要的时候能快速提取数据。虽然 ECMAScript 5 没有正式支持 set 和 map，开发者使用非类数组对象对它们进行了模拟。

ECMAScript 6 正式为 JavaScript 添加了 set 和 map，本章介绍了这两种集合类型你所需要了解的的全部信息。

首先，我会讲解开发者在 ECMAScript 6 之前是使用了什么样的解决方案来实现 set 和 map，而且为什么这些方案是有问题的。在这些重要的背景浮出水面之后，我会介绍 ECMAScript 6 中 set 和 map 的工作机制。

<br />

### ECMAScript 5 中的 set 与 map（Sets and Maps in ECMAScript 5）

In ECMAScript 5, developers mimicked sets and maps by using object properties, like this:

```
let set = Object.create(null);

set.foo = true;

// 检查属性是否存在
if (set.foo) {

    // do something
}
```

The set variable in this example is an object with a null prototype, ensuring that there are no inherited properties on the object. Using object properties as unique values to be checked is a common approach in ECMAScript 5. When a property is added to the set object, it is set to true so conditional statements (such as the if statement in this example) can easily check whether the value is present.

The only real difference between an object used as a set and an object used as a map is the value being stored. For instance, this example uses an object as a map:

```
let map = Object.create(null);

map.foo = "bar";

// 提取属性值
let value = map.foo;

console.log(value);         // "bar"
```

This code stores a string value "bar" under the key foo. Unlike sets, maps are mostly used to retrieve information, rather than just checking for the key’s existence.

### 方案中的问题（Problems with Workarounds）

While using objects as sets and maps works okay in simple situations, the approach can get more complicated once you run into the limitations of object properties. For example, since all object properties must be strings, you must be certain no two keys evaluate to the same string. Consider the following:

```
let map = Object.create(null);

map[5] = "foo";

console.log(map["5"]);      // "foo"
```

This example assigns the string value "foo" to a numeric key of 5. Internally, that numeric value is converted to a string, so map["5"] and map[5] actually reference the same property. That internal conversion can cause problems when you want to use both numbers and strings as keys. Another problem arises when using objects as keys, like this:

```
let map = Object.create(null),
    key1 = {},
    key2 = {};

map[key1] = "foo";

console.log(map[key2]);     // "foo"
```

Here, map[key2] and map[key1] reference the same value. The objects key1 and key2 are converted to strings because object properties must be strings. Since "[object Object]" is the default string representation for objects, both key1 and key2 are converted to that string. This can cause errors that may not be obvious because it’s logical to assume that different object keys would, in fact, be different.

The conversion to the default string representation makes it difficult to use objects as keys. (The same problem exists when trying to use an object as a set.)

Maps with a key whose value is falsy present their own particular problem, too. A falsy value is automatically converted to false when used in situations where a boolean value is required, such as in the condition of an if statement. This conversion alone isn’t a problem–so long as you’re careful as to how you use values. For instance, look at this code:

```
let map = Object.create(null);

map.count = 1;

// 检查 "count" 是否存在或该值是否为零？
if (map.count) {
    // ...
}
```

This example has some ambiguity as to how map.count should be used. Is the if statement intended to check for the existence of map.count or that the value is nonzero? The code inside the if statement will execute because the value 1 is truthy. However, if map.count is 0, or if map.count doesn’t exist, the code inside the if statement would not be executed.

These are difficult problems to identify and debug when they occur in large applications, which is a prime reason that ECMAScript 6 adds both sets and maps to the language.

> JavaScript has the in operator that returns true if a property exists in an object without reading the value of the object. However, the in operator also searches the prototype of an object, which makes it only safe to use when an object has a null prototype. Even so, many developers still incorrectly use code as in the last example rather than using in.

<br />
