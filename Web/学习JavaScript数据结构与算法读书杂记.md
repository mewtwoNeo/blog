# 《学习JavaScript数据结构与算法》读书杂记

## 第2章 数组

ES6 引入了一种新的原始数据类型Symbol，表示独一无二的值。它是 JavaScript 语言的第七种数据类型，前六种是：undefined、null、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）

NaN 属性是代表非数字值的特殊值。该属性用于指示某个值不是数字。可以把 Number 对象设置为该值，来指示其不是数字值。

``` JavaScript
 console.log(typeof Number.NaN === 'number'); // true
```

null的类型，使用typeof时返回的是object，这是由于历史原因造成的。1995年的 JavaScript 语言第一版，只设计了五种数据类型（对象、整数、浮点数、字符串和布尔值），没考虑null，只把它当作object的一种特殊值。
原理是这样的，不同的对象在底层都表示为二进制，在 JavaScript 中二进制前三位都为 0 的话会被判断为 object 类型，null 的二进制表示是全 0，自然前三位也是 0，所以执行 typeof 时会返回“object”。
后来null独立出来，作为一种单独的数据类型，为了兼容以前的代码，typeof null返回object就没法改变了。

``` JavaScript
console.log(typeof null);// object
```

### 迭代函数

- every方法会用传入的函数迭代数组中的每个元素，其中只要有一个值为false，就返回false。
- some方法会用传入的函数迭代数组的每个元素，其中只要有一个值为true，就返回true。
- map方法会迭代数组每个元素并执行一次传入的函数，然后将执行结果组成新的数组返回。
- filter法方会迭代数组每个元素并执行一次传入的函数，然后返回一个由传入函数执行结果为true的元素组成的新数组。
