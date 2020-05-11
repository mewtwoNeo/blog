# 《ECMAScript 6 入门》读书杂记

## 第9章 数组的扩展

### 数组实例的fill()方法

fill方法使用给定值，填充一个数组。

``` javascript
['a', 'b', 'c'].fill(7)
// [7, 7, 7]

new Array(3).fill(7)
// [7, 7, 7]
```

如果填充的类型为对象，那么被赋值的是同一个内存地址的对象，而不是深拷贝对象。下面代码中 arr[0].name = "Ben" 处对arr数组的第一个对象赋值，因为是浅拷贝导致后面的对象也同时变化，arr[0].push(5) 同理。

``` javascript

let arr = new Array(3).fill({name: "Mike"});
arr[0].name = "Ben";
arr
// [{name: "Ben"}, {name: "Ben"}, {name: "Ben"}]

let arr = new Array(3).fill([]);
arr[0].push(5);
arr
// [[5], [5], [5]]

```

### 数组的fill()方法

没有该方法之前，我们通常使用数组的indexOf方法，检查是否包含某个值。

``` javascript
if (arr.indexOf(el) !== -1) {
  // ...
}
```

indexOf方法有两个缺点，一是不够语义化，它的含义是找到参数值的第一个出现位置，所以要去比较是否不等于-1，表达起来不够直观。二是，它内部使用严格相等运算符（===）进行判断，这会导致对NaN的误判。

``` javascript
[NaN].indexOf(NaN)
// -1
```

includes使用的是不一样的判断算法，就没有这个问题

``` javascript
[NaN].includes(NaN)
// true
```
