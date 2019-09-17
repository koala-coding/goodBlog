<!-- 高阶函数 -->
![](https://user-gold-cdn.xitu.io/2019/7/14/16bec3fc59d12ae5?w=940&h=400&f=jpeg&s=124779)
## 前言

一道经典面试题：

```javascript
//JS实现一个无限累加的add函数
add(1)  //1 
add(1)(2)  //3
add(1)(2)(3)  //6
```

当大家看到这个面试题的时候，能否在第一时间想到使用**高阶函数**实现？想到在实际项目开发过程中，用到哪些高级函数？有没有想过自己创造一个高阶函数呢？开始本篇文章的学习

## 高阶函数定义
`高阶函数`英文叫 `Higher-order function`。高阶函数是对其他函数进行操作的函数，操作可以是将它们作为参数，或者返回它们。简单总结为高阶函数是`一个接收函数作为参数`或者将`函数作为返回输出`的函数。


## 函数作为参数情况

  `Array.prototype.map`，`Array.prototype.filter`，`Array.prototype.reduce`和`Array.prototype.sort`是JavaScript中内置的高阶函数。它们接受一个函数作为参数，并应用这个函数到列表的每一个元素。下面是一些内置高阶函数的具体说明讲解，以及和不使用高阶函数情况下的对比

## Array.prototype.map

`map()`(映射)方法最后生成一个新数组，不改变原始数组的值。其结果是该数组中的每个元素都调用一个提供的函数后返回的结果。

```JavaScript
array.map(callback,[ thisObject]);
```

callback(回调函数)
```JavaScript
[].map(function(currentValue, index, array) {
    // ...
});
```

传递给 `map` 的回调函数（`callback`）接受三个参数，分别是`currentValue`——正在遍历的元素、index（可选）——元素索引、array（可选）——原数组本身，除了 `callback` 之外还可以接受 this 值（可选），用于执行 `callback` 函数时使用的this 值。

来个简单的例子方便理解，现在有一个数组`[1,2,3,4]`，我们想要生成一个新数组，其每个元素皆是之前数组的两倍，那么我们有下面两种使用高阶和不使用高阶函数的方式来实现。

### 不使用高阶函数

```javascript
// koala
const arr1 = [1, 2, 3, 4];
const arr2 = [];
for (let i = 0; i < arr1.length; i++) {
  arr2.push( arr1[i] * 2);
}

console.log( arr2 );
// [2, 4, 6, 8]
console.log( arr1 );
// [1, 2, 3, 4]
```

### 使用高阶函数

```javascript
// kaola
const arr1 = [1, 2, 3, 4];
const arr2 = arr1.map(item => item * 2);

console.log( arr2 );
// [2, 4, 6, 8]
console.log( arr1 );
// [1, 2, 3, 4]
```

###  map高阶函数注意点
callback需要有return值，否则会出现所有项映射为undefind；
```javascript
// kaola
const arr1 = [1, 2, 3, 4];
const arr2 = arr1.map(item => {});

console.log( arr2 );
// [ undefined, undefined, undefined, undefined ]
console.log( arr1 );
// [1, 2, 3, 4]
```
### map高阶函数对应的一道经典面试题

```javascript
//输出结果
["1", "2", "3"].map(parseInt);
```
看了这道题不知道会不会有大多数开发者认为输出结果是[1,2,3],**错误**

正确的输出结果为：

```JavaScript
[1,NaN,NaN]
```
##### 分析与讲解
因为`map`的`callback`函数有三个参数，正在遍历的元素, 元素索引(index), 原数组本身(array)。`parseInt`有两个参数，string和radix(进制)，注意第二个参数进制当为0或者没有参数的时候，parseInt()会根据string来判断数字的基数。当忽略参数 radix , JavaScript 默认数字的基数如下:

- 如果 string 以 "0x" 开头，parseInt() 会把 string 的其余部分解析为十六进制的整数。
- 如果 string 以 0 开头，那么 ECMAScript v3 允许 parseInt() 的一个实现把其后的字符解析为八进制或十六进制的数字。
- 如果 string 以 1 ~ 9 的数字开头，parseInt() 将把它解析为十进制的整数。

只传入parseInt的话，map callback会自动忽略第三个参数array。而index参数不会被忽略。而不被忽略的index(0,1,2)就会被parseInt当做第二个参数。

将其拆开看：

```javascript
parseInt("1",0);//上面说过第二个参数为进制，所以"1"，radix为0上面提到过，会忽略，根据string 以 1 ~ 9 的数字开头，parseInt() 将把它解析为十进制的整数1。
parseInt("2",1);//此时将2转为1进制数，由于超过进制数1，所以返回NaN。
parseInt("3",2);//此时将3转为2进制数，由于超过进制数1，所以返回NaN。
```

所以最终的结果为`[1,NaN,NaN]`。

那么如果想要得到`[1,2,3]`该怎么写。
```javascript
["1","2","3"].map((x)=>{
    return parseInt(x);
});
```
也可以简写为：
`["1","2","3"].map(x=>parseInt(x));`

这样写为什么就能返回想要的值呢？因为，传一个完整函数进去，有形参，有返回值。这样就不会造成因为参数传入错误而造成结果错误了，最后返回一个经纯函数处理过的新数组。


## Array.prototype.reduce

`reduce()` 方法对数组中的每个元素执行一个提供的 reducer 函数(升序执行)，将其结果汇总为单个返回值。传递给 reduce 的回调函数（`callback`）接受四个参数，分别是累加器 `accumulator`、`currentValue`——正在操作的元素、`currentIndex`（可选）——元素索引，但是它的开始会有特殊说明、array（可选）——原始数组本身，除了 `callback` 之外还可以接受初始值 `initialValue` 值（可选）。

- 如果没有提供 initialValue，那么第一次调用 callback 函数时，accumulator 使用原数组中的第一个元素，currentValue 即是数组中的第二个元素。 在没有初始值的空数组上调用 reduce 将报错。

- 如果提供了 initialValue，那么将作为第一次调用 callback 函数时的第一个参数的值，即 accumulator，currentValue 使用原数组中的第一个元素。

例子，现在有一个数组  [0, 1, 2, 3, 4]，需要计算数组元素的和，需求比较简单，来看下代码实现。

### 不使用高阶函数

```JavaScript
//koala
const arr = [0, 1, 2, 3, 4];
let sum = 0;
for (let i = 0; i < arr.length; i++) {
  sum += arr[i];
}

console.log( sum );
// 10
console.log( arr );
// [0, 1, 2, 3, 4]
```

### 使用高阶函数
#### 无 initialValue 值


```JavaScript
const arr = [0, 1, 2, 3, 4];
let sum = arr.reduce((accumulator, currentValue, currentIndex, array) => {
  return accumulator + currentValue;
});

console.log( sum );
// 10
console.log( arr );
// [0, 1, 2, 3, 4]
```

上面是没有 initialValue 的情况，代码的执行过程如下，callback 总共调用四次。

callback | accumulator | currentValue | currentIndex | array | return value 
---| --- |--- |--- |--- |---
first call  |	0 |	1 |	1 |	[0, 1, 2, 3, 4]	|1
second call	| 1	 |2	 |2	 |[0, 1, 2, 3, 4] |	3
third call	| 3	| 3	| 3	| [0, 1, 2, 3, 4]| 	6
fourth call| 	6 |	4 |	4	| [0, 1, 2, 3, 4] |	10



#### 有 initialValue 值
我们再来看下有 initialValue 的情况，假设 initialValue 值为 10，我们看下代码。

```JavaScript
//koala
const arr = [0, 1, 2, 3, 4];
let sum = arr.reduce((accumulator, currentValue, currentIndex, array) => {
  return accumulator + currentValue;
}, 10);

console.log( sum );
// 20
console.log( arr );
// [0, 1, 2, 3, 4]
```

代码的执行过程如下所示，callback 总共调用五次。

callback | accumulator | currentValue | currentIndex | array | return value 
---| --- |--- |--- |--- |---
first call|	10	|0	|0|	[0, 1, 2, 3, 4]|	10
second call	|10|	1|	1|	[0, 1, 2, 3, 4]|	11
third call|	11|	2|	2|	[0, 1, 2, 3, 4]	|13
fourth call	|13|	3|	3|	[0, 1, 2, 3, 4]	|16
fifth call|	16|	4|	4|	[0, 1, 2, 3, 4]	|20


## Array.prototype.filter

`filter`(过滤，筛选) 方法创建一个新数组,原始数组不发生改变。

```javascript
array.filter(callback,[ thisObject]);
```

其包含通过提供函数实现的测试的所有元素。接收的参数和 map 是一样的，filter的`callbac`k函数需要返回布尔值true或false. 如果为true则表示通过啦！如果为false则失败，其返回值是一个新数组，由通过测试为true的所有元素组成，如果没有任何数组元素通过测试，则返回空数组。

来个例子介绍下，现在有一个数组 `[1, 2, 1, 2, 3, 5, 4, 5, 3, 4, 4, 4, 4]`，我们想要生成一个新数组，这个数组要求没有重复的内容，即为去重。

### 不使用高阶函数

```JavaScript
const arr1 = [1, 2, 1, 2, 3, 5, 4, 5, 3, 4, 4, 4, 4];
const arr2 = [];
for (let i = 0; i < arr1.length; i++) {
  if (arr1.indexOf( arr1[i] ) === i) {
    arr2.push( arr1[i] );
  }
}
console.log( arr2 );
// [1, 2, 3, 5, 4]
console.log( arr1 );
// [1, 2, 1, 2, 3, 5, 4, 5, 3, 4, 4, 4, 4]
```

### 使用高阶函数

```JavaScript
const arr1 = [1, 2, 1, 2, 3, 5, 4, 5, 3, 4, 4, 4, 4];
const arr2 = arr1.filter( (element, index, self) => {
    return self.indexOf( element ) === index;
});

console.log( arr2 );
// [1, 2, 3, 5, 4]
console.log( arr1 );
// [1, 2, 1, 2, 3, 5, 4, 5, 3, 4, 4, 4, 4]
```
### filter注意点说明
`callback`在过滤测试的时候，一定要是Boolean值吗？
例子：

```JavaScript
var arr = [0, 1, 2, 3];
var arrayFilter = arr.filter(function(item) {
    return item;
});
console.log(arrayFilter); // [1, 2, 3]
```
通过例子可以看出:过滤测试的返回值只要是弱等于== true/false就可以了，而非非得返回 === true/false.



## Array.prototype.sort
`sort() `方法用原地算法对数组的元素进行排序，并返回数组，该排序方法会在原数组上直接进行排序，并不会生成一个排好序的新数组。排序算法现在是稳定的。默认排序顺序是根据字符串Unicode码点。


```JavaScript
// 语法
arr.sort([compareFunction])
```
`compareFunction`参数是可选的，用来指定按某种顺序进行排列的函数。注意该函数有两个参数：

**参数1:firstEl**

第一个用于比较的元素。

**参数2:secondEl**

第二个用于比较的元素。看下面的例子与说明：
```JavaScript
// 未指明compareFunction函数

['Google', 'Apple', 'Microsoft'].sort(); // ['Apple', 'Google', 'Microsoft'];

// apple排在了最后:
['Google', 'apple', 'Microsoft'].sort(); // ['Google', 'Microsoft", 'apple']

// 无法理解的结果:
[10, 20, 1, 2].sort(); // [1, 10, 2, 20]
//正确的结果
[6, 8, 1, 2].sort(); // [1, 2，6, 8]

// 指明compareFunction函数
'use strict';
var arr = [10, 20, 1, 2];
    arr.sort(function (x, y) {
        if (x < y) {
            return -1;
        }
        if (x > y) {
            return 1;
        }
        return 0;
    });
console.log(arr); // [1, 2, 10, 20]
```

如果没有指明 `compareFunction` ，那么元素会按照转换为的字符串的诸个字符的`Unicode`位点进行排序。例如 "Banana" 会被排列到 "cherry" 之前。当数字按由小到大排序时，10 出现在 2 之前，但因为（没有指明 `compareFunction`），比较的数字会先被转换为字符串，所以在`Unicode`顺序上 "10" 要比 "2" 要靠前。

如果指明了 `compareFunction` ，那么数组会按照调用该函数的返回值排序。即 a 和 b 是两个将要被比较的元素：

- 如果 compareFunction(a, b) 小于 0 ，那么 a 会被排列到 b 之前；

- 如果 compareFunction(a, b) 等于 0 ， a 和 b 的相对位置不变。备注： ECMAScript 标准并不保证这一行为，而且也不是所有浏览器都会遵守（例如 Mozilla 在 2003 年之前的版本）；

- 如果 compareFunction(a, b) 大于 0 ， b 会被排列到 a 之前。
compareFunction(a, b) 必须总是对相同的输入返回相同的比较结果，否则排序的结果将是不确定的。

### sort排序算法的底层实现
看了上面`sort`的排序介绍，我想小伙伴们肯定会对sort排序算法的内部实现感兴趣，我在sf上面搜了一下，发现有些争议。于是去查看了V8引擎的源码，发现在源码中的710行  

> 源码地址：https://github.com/v8/v8/blob/ad82a40509c5b5b4680d4299c8f08d6c6d31af3c/src/js/array.js


```JavaScript
// In-place QuickSort algorithm.
// For short (length <= 22) arrays, insertion sort is used for efficiency.
```

V8 引擎 sort 函数只给出了两种排序 `InsertionSort`和 `QuickSort`，**数量小于等于22**的数组使用 `InsertionSort`，比22大的数组则使用 `QuickSort`，有兴趣的可以看看具体算法实现。

> 注意：不同的浏览器引擎可能算法实现并不同，我这里只是查看了V8引擎的算法实现，有兴趣的小伙伴可以查看下其他开源浏览器具体sort的算法实现。

### 如何改进排序算法实现数字正确排序呢？

对于要比较数字而非字符串，比较函数可以简单的以 a 减 b，如下的函数将会将数组升序排列，降序排序则使用b-a。

```JavaScript
let compareNumbers= function (a, b) {
    return a - b;
}
let koala=[10, 20, 1, 2].sort(compareNumbers)

console.log(koala);
// [1 , 2 , 10 , 20]
```


## 函数作为返回值输出


返回一个函数，下面直接看两个例子来加深理解。

### isType 函数
我们知道在判断类型的时候可以通过`Object.prototype.toString.call` 来获取对应对象返回的字符串，比如：


```JavaScript
let isString = obj => Object.prototype.toString.call( obj ) === '[object String]';

let isArray = obj => Object.prototype.toString.call( obj ) === '[object Array]';

let isNumber = obj => Object.prototype.toString.call( obj ) === '[object Number]';
```

可以发现上面三行代码有很多重复代码，只需要把具体的类型抽离出来就可以封装成一个判断类型的方法了，代码如下。


```JavaScript
let isType = type => obj => {
  return Object.prototype.toString.call( obj ) === '[object ' + type + ']';
}

isType('String')('123');        // true
isType('Array')([1, 2, 3]);    // true
isType('Number')(123);            // true
```

这里就是一个高阶函数，因为 isType 函数将 `obj => { ... }` 这一函数作为返回值输出。

### add求和函数
前言中的面试题，用 JS 实现一个无限累加的函数 add，示例如下：
```JavaScript
add(1); // 1
add(1)(2);  // 3
add(1)(2)(3)； // 6
```


分析面试题的结构，都是将函数作为返回值输出，然后接收新的参数并进行计算。

我们知道打印函数时会自动调用 `toString()`方法（如果不知道的可以去看我的这篇文章），函数 add(a) 返回一个sum(b)函数，函数 sum() 中累加计算 a = a + b，只需要重写sum.toString()方法返回变量 a 就可以了。


```JavaScript
function add(a) {
    function sum(b) { // 使用闭包
        a = a + b; // 累加
        return sum;
     }
     sum.toString = function() { // 重写toString()方法
        return a;
    }
     return sum; // 返回一个函数
}

add(1); // 1
add(1)(2);  // 3
add(1)(2)(3)； // 6
```

## 如何自己创建高阶函数
前面讲了语言中内置的各种高阶函数。知道了到底啊什么是高阶函数，有哪些类型的高阶函数。那么让我们**自己创建一个高阶函数吧！**

假设 JavaScript 没有原生的` map `方法。 我们自己构建个类似map的高阶函数，从而创建我们自己的高阶函数。
假设我们有一个字符串数组，我们希望把它转换为整数数组，其中每个元素代表原始数组中字符串的长度。


```JavaScript
const strArray=['JavaScript','PHP','JAVA','C','Python'];
function mapForEach(arr,fn){
    const newArray = [];
    for(let i = 0; i<arr.length;i++){
        newArray.push({
            fn(arr[i])
        );
    }
    return newArray;
}
const lenArray = mapForEach(strArray,function(item){
    return item.length;
});

console.log(lenArray);//[10,3,4,1,6]
```
### 代码分析讲解：
我们创建了一个高阶函数 mapForEach ，它接受一个数组和一个回调函数 fn。 它循环遍历传入的数组，并在每次迭代时在 newArray.push 方法调用回调函数 fn 。

回调函数 fn 接收数组的当前元素并返回该元素的长度，该元素存储在 newArray 中。 for 循环完成后，newArray 被返回并赋值给 lenArray。



## 总结
我们已经了解了高阶函数和一些内置的高阶函数，还学习了如何创建自己的高阶函数。简而言之，高阶函数是一个可以接收函数作为参数，甚至返回一个函数的函数。 它就像常规函数一样，只是多了接收和返回其他函数的附加能力，即参数和输出。


### 公众号技术栈路线

大家好，我是koala，公众号「程序员成长指北」作者，这篇文章是【JS必知必会系列】的高阶函数讲解。目前在做一个node后端工程师进阶路线，加入我们一起学习吧！

![](http://img.xiaogangzai.cn/way.jpg)



### 加入我们

![](http://img.xiaogangzai.cn/code.jpg)

