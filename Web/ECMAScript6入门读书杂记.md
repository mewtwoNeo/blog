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

## 第10章 对象的扩展

### 属性的可枚举性和遍历

#### 可枚举性

对象的每个属性都有个描述对象(Object.getOwnPropertyDescriptor方法可以获取该属性的描述对象)。描述对象的enumerable(Boolean类型)属性表示这个属性是否可以被枚举。

目前有四个操作会忽略enumerable为false的属性：

- for...in 循环：只遍历对象自身的和继承的可枚举的属性。
- Object.keys()：返回对象自身的所有可枚举的属性的键名。
- JSON.stringify()：只串行化对象自身的可枚举的属性。
- Object.assing()：忽略enumerable为false的属性，只拷贝对象自身的可枚举属性。

只有for...in会返回继承的属性，其实enumerable最初也是为了防止for...in把对象所有内部属性和方法都遍历出来。

#### 遍历

ES6一共有5中方可以遍历对象属性。

1. for...in循环遍历对象自身和继承的可枚举属性(不含Symbol属性)。

2. Object.keys(obj)返回一个数组，包含对象自身（不含继承的）的所有可枚举的属性（不含Symbol属性）的键名。

3. Object.getOwnPropertyNames(obj)返回一个数组，包含对象自身（不含继承的）的所有属性（不含Symbol属性，但是包含不可枚举的属性）的键名。

4. Object.getOwnPropertySymbols(obj)返回一个数组，包含对象自身所有的Symbol属性的键名。

5. Reflect.ownKeys(obj)返回一个数组，包含对象自身的所有键名，不管是Symbol还是字符串，也不管是是否可枚举。

遍历顺序

先遍历数字属性再根据字符串属性加入时间遍历，最后根据Symbol属性加入时间遍历。

### super关键字

super关键字指向当前对象的原型对象。

super关键字表示原型对象时，只能在对象方法中调用，目前只有对象方法的简写可以正确识别。

### 对象的扩展运算符

#### 解构赋值

1. 由于解构赋值要求等号右边必须是一个对象，所以如果等号右边是undefined或null，就会报错。
2. 解构赋值必须是最后一个参数否者会报错。
3. 解构赋值的拷贝是浅拷贝。
4. 扩展运算符的解构赋值不能复制继承自原型对象的属性。

#### 扩展运算符

1. 由于数组是特殊的对象，所以对象的扩展运算符也可以用于数组

``` javascript
let foo = {...['a','b','c']};
foo
// {0:'a',1:'b',2:'c'}
```

2. 如果扩展运算符后面是一个空对象，则没有任何效果
3. 如果扩展运算符后面不是一个对象，则会转为对象

```javascript
{...'hello'}
// {0: "h", 1: "e", 2: "l", 3: "l", 4: "o"}
```
4. 对象的扩展运算符等同于Object.assign()方法
5. 扩展运算可以用于合并两个对象

```javascript
let ab = {...a, ...b};
// 等同于
let ab = Object.assign({}, a, b);
```

