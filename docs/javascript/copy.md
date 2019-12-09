# 搞不懂JS中赋值·浅拷贝·深拷贝的请看这里
## 前言
为什么写拷贝这篇文章？同事有一天提到了拷贝，他说赋值就是一种浅拷贝方式，另一个同事说赋值和浅拷贝并不相同。
我也有些疑惑，于是我去MDN搜一下拷贝相关内容，发现并没有关于拷贝的实质概念，没有办法只能通过实践了，同时去看一些前辈们的文章总结了这篇关于拷贝的内容,本文也属于公众号【程序员成长指北】学习路线中【JS必知必会】内容。

## 数据类型与堆栈的关系

### 基本类型与引用类型

- 基本类型：undefined,null,Boolean,String,Number,Symbol

- 引用类型：Object,Array,Date,Function,RegExp等

### 存储方式

- 基本类型：基本类型值在内存中占据固定大小，保存在`栈内存`中（不包含`闭包`中的变量）

![](https://user-gold-cdn.xitu.io/2019/7/8/16bd2282836bad2c?w=370&h=346&f=jpeg&s=17652)
- 引用类型：引用类型的值是对象，保存在`堆内存`中。而栈内存存储的是对象的变量标识符以及对象在堆内存中的存储地址(引用)，引用数据类型在栈中存储了指针，该指针指向堆中该实体的起始地址。当解释器寻找引用值时，会首先检索其在栈中的地址，取得地址后从堆中获得实体。

![](https://user-gold-cdn.xitu.io/2019/7/8/16bd228c2ad68a18?w=758&h=256&f=jpeg&s=24273)

**注意：**

 1. `闭包`中的变量并不保存在栈内存中，而是保存在堆内存中。这一点比较好想，如果`闭包`中的变量保存在了`栈内存`中，随着外层中的函数从调用栈中销毁，变量肯定也会被销毁，但是如果保存在了堆内存中，内存函数仍能访问外层已销毁函数中的变量。看一段对应代码理解下：

```javascript
function A() {
  let a = 'koala'
  function B() {
      console.log(a)
  }
  return B
}
```

2. 本篇所讲的浅拷贝和深拷贝都是对于引用类型的，对于基础类型不会有这种操作。

## 赋值操作
### 基本数据类型复制

看一段代码

```javascript
let a ='koala';
let b = a;
b='程序员成长指北'；
console.log(a); // koala
```
基本数据类型复制配图：

![](https://user-gold-cdn.xitu.io/2019/7/8/16bd22955385ae3e?w=698&h=289&f=png&s=10920)

**结论**：在栈内存中的数据发生数据变化的时候，系统会自动为新的变量分配一个新的之值在栈内存中，两个变量相互独立，互不影响的。

### 引用数据类型复制
看一段代码

```javascript
let a = {x:'kaola', y:'kaola1'}
let b = a;
b.x = '程序员成长指北';
console.log(a.x); // 程序员成长指北
```
引用数据类型复制配图：

![](https://user-gold-cdn.xitu.io/2019/7/8/16bd22a0d45de903?w=468&h=483&f=png&s=21450)
**结论**：引用类型的复制，同样为新的变量b分配一个新的值，保存在栈内存中，不同的是这个变量对应的具体值不在栈中，栈中只是一个地址指针。两个变量地址指针相同，指向堆内存中的对象，因此b.x发生改变的时候，a.x也发生了改变。


## 浅拷贝
### 浅拷贝定义：
不知道的api我一般比较喜欢看MDN，浅拷贝的概念MDN官方并没有给出明确定义，但是搜到了一个函数Array.prototype.slice，官方说它可以实现原数组的浅拷贝。
对于官方给的结论，我们通过两段代码验证一下，并总结出浅拷贝的定义。

- 第一段代码：
```javascript
var a = [ 1, 3, 5, { x: 1 } ];
var b = Array.prototype.slice.call(a);
b[0] = 2;
console.log(a); // [ 1, 3, 5, { x: 1 } ];
console.log(b); // [ 2, 3, 5, { x: 1 } ];
```
从输出结果可以看出，浅拷贝后，数组a[0]并不会随着b[0]改变而改变，说明a和b在栈内存中引用地址并不相同。

- 第二段代码

```javascript
var a = [ 1, 3, 5, { x: 1 } ];
var b = Array.prototype.slice.call(a);
b[3].x = 2;
console.log(a); // [ 1, 3, 5, { x: 2 } ];
console.log(b); // [ 1, 3, 5, { x: 2 } ];
```
从输出结果可以看出，浅拷贝后，数组中对象的属性会根据修改而改变，说明浅拷贝的时候拷贝的已存在对象的对象的属性引用。


- 浅拷贝定义

通过这个官方的`slice`浅拷贝函数分析`浅拷贝定义`：
> 新的对象复制已有对象中非对象属性的值和对象属性的引用。如果这种说法不理解换一种一个新的对象直接拷贝已存在的对象的对象属性的引用，即浅拷贝。

### 浅拷贝实例

#### Object.assign
- 语法：
> 语法：Object.assign(target, ...sources)

ES6中拷贝对象的方法，接受的第一个参数是`拷贝的目标target`，剩下的参数是拷贝的`源对象sources`（可以是多个）

- 举例说明：
```javascript
let target = {};
let source = {a:'koala',b:{name:'程序员成长指北'}};
Object.assign(target ,source);
console.log(target); // { a: 'koala', b: { name: '程序员成长指北' } }
source.a = 'smallKoala';
source.b.name = '程序员成长指北哦'
console.log(source); // { a: 'smallKoala', b: { name: '程序员成长指北哦' } }
console.log(target); // { a: 'koala', b: { name: '程序员成长指北哦' } }
```
从打印结果可以看出，`Object.assign`是一个浅拷贝,它只是在根属性(对象的第一层级)创建了一个新的对象，但是对于属性的值是对象的话只会拷贝一份相同的内存地址。

- Object.assign注意事项
1. 只拷贝源对象的自身属性（不拷贝继承属性）
2. 它不会拷贝对象不可枚举的属性
3. `undefined`和`null`无法转成对象，它们不能作为`Object.assign`参数，但是可以作为源对象

```javascript
Object.assign(undefined) // 报错
Object.assign(null) // 报错

let obj = {a: 1};
Object.assign(obj, undefined) === obj // true
Object.assign(obj, null) === obj // true
```

4. 属性名为` Symbol` 值的属性，可以被Object.assign拷贝。

#### Array.prototype.slice
这个函数在浅拷贝概念定义的时候已经进行了分析，看上文。

#### Array.prototype.concat
- 语法
> var new_array = old_array.concat(value1[, value2[, ...[, valueN]]])
参数：将数组和/或值连接成新数组

- 举例说明

```javascript
let array = [{a: 1}, {b: 2}];
let array1 = [{c: 3},{d: 4}];
let array2=array.concat(array1);
array1[0].c=123;
console.log(array2);// [ { a: 1 }, { b: 2 }, { c: 123 }, { d: 4 } ]
console.log(array1);// [ { c: 123 }, { d: 4 } ]
```
Array.prototype.concat也是一个浅拷贝，只是在根属性(对象的第一层级)创建了一个新的对象，但是对于属性的值是对象的话只会拷贝一份相同的内存地址。
#### ...扩展运算符
- 语法
> var cloneObj = { ...obj };

- 举例说明

```javascript
let obj = {a:1,b:{c:1}}
let obj2 = {...obj};
obj.a=2;
console.log(obj); //{a:2,b:{c:1}}
console.log(obj2); //{a:1,b:{c:1}}

obj.b.c = 2;
console.log(obj); //{a:2,b:{c:2}}
console.log(obj2); //{a:1,b:{c:2}}
```
扩展运算符也是浅拷贝，对于值是对象的属性无法完全拷贝成2个不同对象,但是如果属性都是基本类型的值的话,使用扩展运算符也是优势方便的地方。

补充说明：以上4中浅拷贝方式都不会改变原数组，只会返回一个浅拷贝了原数组中的元素的一个新数组。

### 自己实现一个浅拷贝
实现原理：新的对象复制已有对象中非对象属性的值和对象属性的`引用`,也就是说对象属性并不复制到内存。

- 实现代码:

```javascript
function cloneShallow(source) {
    var target = {};
    for (var key in source) {
        if (Object.prototype.hasOwnProperty.call(source, key)) {
            target[key] = source[key];
        }
    }
    return target;
}
```
- for in与hasOwnProperty函数说明，怕有些人小伙伴可能不清楚具体内容

**for in**

for...in语句以任意顺序遍历一个对象自有的、继承的、`可枚举的`、非Symbol的属性。对于每个不同的属性，语句都会被执行。

**hasOwnProperty**
> 语法：obj.hasOwnProperty(prop)      
prop是要检测的属性`字符串`名称或者`Symbol`

该函数返回值为布尔值，所有继承了 Object 的对象都会继承到 hasOwnProperty 方法，和 in 运算符不同，该函数会忽略掉那些从原型链上继承到的属性和自身属性。

## 深拷贝操作
说了赋值操作和浅拷贝操作，大家是不是已经能想到什么是深拷贝了，下面直接说深拷贝的定义。
### 深拷贝定义
> 深拷贝会另外拷贝一份一个一模一样的对象,从堆内存中开辟一个新的区域存放新对象,新对象跟原对象不共享内存，修改新对象不会改到原对象。

### 深拷贝实例
#### JSON.parse(JSON.stringify())

JSON.stringify()是前端开发过程中比较常用的深拷贝方式。原理是把一个对象序列化成为一个JSON字符串，将对象的内容转换成字符串的形式再保存在磁盘上，再用JSON.parse()反序列化将JSON字符串变成一个新的对象
- 举例说明：

```javascript
let arr = [1, 3, {
    username: ' koala'
}];
let arr4 = JSON.parse(JSON.stringify(arr));
arr4[2].username = 'smallKoala'; 
console.log(arr4);// [ 1, 3, { username: 'smallKoala' } ]
console.log(arr);// [ 1, 3, { username: ' koala' } ]
```
实现了深拷贝，当改变数组中对象的值时候，原数组中的内容并没有发生改变。JSON.stringify()虽然可以实现深拷贝，但是还有一些弊端比如不能处理函数等。

- JSON.stringify()实现深拷贝注意点

1. 拷贝的对象的值中如果有函数,undefined,symbol则经过JSON.stringify()序列化后的JSON字符串中这个键值对会消失
2. 无法拷贝不可枚举的属性，无法拷贝对象的原型链
3. 拷贝Date引用类型会变成字符串
4. 拷贝RegExp引用类型会变成空对象
5. 对象中含有NaN、Infinity和-Infinity，则序列化的结果会变成null
6. 无法拷贝对象的循环应用(即obj[key] = obj)


#### 自己实现一个简单深拷贝
深拷贝，主要用到的思想是递归，遍历对象、数组直到里边都是基本数据类型，然后再去复制，就是深度拷贝。
实现代码：

```javascript
    //定义检测数据类型的功能函数
    function isObject(obj) {
	    return typeof obj === 'object' && obj != null;
    }
   function cloneDeep(source) {

    if (!isObject(source)) return source; // 非对象返回自身
      
    var target = Array.isArray(source) ? [] : {};
    for(var key in source) {
        if (Object.prototype.hasOwnProperty.call(source, key)) {
            if (isObject(source[key])) {
                target[key] = cloneDeep(source[key]); // 注意这里
            } else {
                target[key] = source[key];
            }
        }
    }
    return target;
}


```
该简单深拷贝未考虑内容：
 遇到循环引用，会陷入一个循环的递归过程，从而导致爆栈

```javascript
// RangeError: Maximum call stack size exceeded
```
**小伙伴们有没有什么好办法呢，可以写下代码在评论区一起讨论哦！**

#### 第三方深拷贝库

该函数库也有提供_.cloneDeep用来做 Deep Copy（lodash是一个不错的第三方开源库，有好多不错的函数，也可以看具体的实现源码）

```javascript
var _ = require('lodash');
var obj1 = {
    a: 1,
    b: { f: { g: 1 } },
    c: [1, 2, 3]
};
var obj2 = _.cloneDeep(obj1);
console.log(obj1.b.f === obj2.b.f);
// false

```

## 拷贝内容总结
用一张图总结

![](http://img.xiaogangzai.cn/article_16.jpg)

今天就分享这么多，如果对分享的内容感兴趣，可以关注公众号「程序员成长指北」，或者加入技术交流群，大家一起讨论。

进阶技术路线

![](http://img.xiaogangzai.cn/way.jpg)
加入我们一起学习吧！
![](http://img.xiaogangzai.cn/code.jpg)
