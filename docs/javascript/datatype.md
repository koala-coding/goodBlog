# 前言
![](https://user-gold-cdn.xitu.io/2019/6/22/16b7e5de56617bfb?w=940&h=400&f=png&s=367315)
之前面试了几个开发者，他们确实做过不少项目，能力也是不错的，但是发现js基础不扎实, 于是决定写一下这篇javascrip数据类型相关的基础文章，其实也不仅仅是因为面试了他们，之前自己在面试的时候，也曾经被虐过，面试官说过的最深刻的一句话我到现在都记得。
> 基础很重要，只有基础好才会很少出`bug`，大多数的`bug`都是基础不扎实造成的。


## 快速导航
- [```[问题引出]``` 常见面试题](#常见面试题)
- [js中的数据类型](#js中的数据类型)
    * [基本类型](#基本类型)
    * [对象类型](#对象类型)
- [js弱类型语言](#js弱类型语言)
- [js中的强制转换规则](#js中的强制转换规则)
    * [ToPrimitive(转换为原始值)](#ToPrimitive(转换为原始值))
    * [toString](#toString)
    * [valueOf](#valueOf)
    * [Number](#Number)
    * [String](#String)
    * [Boolean](#Boolean)
- [js转换规则不同场景应用](#js转换规则不同场景应用)
- [js中的数据类型判断](#js中的数据类型判断)
- [NaN相关总结](#NaN相关总结)
- [误区](#误区)



### 常见面试题
这里给出两道我们公司数据类型基础相关的面试题和答案，如果都能做对并且知道为什么（可以选择忽略本文章）：

```javascript
//类型转换相关问题
var bar=true;
console.log(bar+0);
console.log(bar+"xyz");
console.log(bar+true);
console.log(bar+false);
console.log('1'>bar);
console.log(1+'2'+false);
console.log('2' + ['koala',1]);

var obj1 = {
   a:1,
   b:2
}
console.log('2'+obj1)；

var obj2 = {
    toString:function(){
        return 'a'
    }
}
console.log('2'+obj2)

//输出结果  1 truexyz 2 1 false 12false 2koala,1 2[object Object] 2a
```

```javascript
//作用域和NaN 这里不具体讲作用域，意在说明NaN
var b=1;
function outer(){
    var b=2;
    function inner(){
        b++;
        console.log(b);
        var b=3;
    }
    inner();
}
outer();
//输出结果 NaN
```


本篇文章会以一个面试官问问题的角度来进行分析讲解

## js中的数据类型

> 面试官：说一说`javascript`中有哪些数据类型?

`JavaScript` 中共有七种内置数据类型，包括**基本类型**和**对象类型**。

### 基本类型
基本类型分为以下六种：

- string（字符串）
- boolean（布尔值）
- number（数字）
- symbol（符号）
- null（空值）
- undefined（未定义）

**注意**：
1. `string` 、`number` 、`boolean` 和 `null`  `undefined` 这五种类型统称为**原始类型**（Primitive），表示不能再细分下去的基本类型;

2. `symbol`是ES6中新增的数据类型，`symbol` 表示独一无二的值，通过 `Symbol` 函数调用生成，由于生成的 symbol 值为原始类型，所以 `Symbol` 函数不能使用` new` 调用；

3. `null` 和 `undefined` 通常被认为是特殊值，这两种类型的值唯一，就是其本身。

### 对象类型
对象类型也叫引用类型，`array`和`function`是对象的子类型。对象在逻辑上是属性的无序集合，是存放各种值的容器。对象值存储的是引用地址，所以和基本类型值不可变的特性不同，对象值是可变的。

# js弱类型语言
> 面试官：说说你对`javascript`是弱类型语言的理解?

`JavaScript` 是弱类型语言，而且`JavaScript` 声明变量的时候并没有预先确定的类型，变量的类型就是其值的类型，也就是说**变量当前的类型由其值所决定**,夸张点说上一秒种的`String`，下一秒可能就是个`Number`类型了，这个过程可能就进行了某些操作发生了强制类型转换。虽然弱类型的这种**不需要预先确定类型**的特性给我们带来了便利，同时也会给我们带来困扰，为了能充分利用该特性就必须掌握类型转换的原理。



# js中的强制转换规则

> 面试官：`javascript`中强制类型转换是一个非常易出现`bug`的点，知道强制转换时候的规则吗？


注：规则最好配合下面什么时候发生转换使用这些规则看效果更佳。

### `ToPrimitive`(转换为原始值)

`ToPrimitive`对原始类型不发生转换处理，只**针对引用类型（object）的**，其目的是将引用类型（object）转换为非对象类型，也就是原始类型。

`ToPrimitive` 运算符**接受一个值，和一个可选的期望类型作参数**。`ToPrimitive` 运算符将值转换为非对象类型，如果对象有能力被转换为不止一种原始类型，可以使用可选的 **期望类型** 来暗示那个类型。

转换后的结果**原始类型**是由期望类型决定的，期望类型其实就是我们传递的`type`。直接看下面比较清楚。
`ToPrimitive`方法大概长这么个样子具体如下。

```javascript
/**
* @obj 需要转换的对象
* @type 期望转换为的原始数据类型，可选
*/
ToPrimitive(obj,type)
```

##### type不同值的说明
- type为`string`:

1. 先调用`obj`的`toString`方法，如果为原始值，则`return`，否则进行第2步
2. 调用`obj`的`valueOf`方法，如果为原始值，则`return`，否则进行第3步
3. 抛出`TypeError` 异常

- type为`number`:

1. 先调用`obj`的`valueOf`方法，如果为原始值，则`return`，否则进行第2步
2. 调用`obj`的`toString`方法，如果为原始值，则`return`，否则第3步
3. 抛出`TypeError` 异常


- type参数为空
1. 该对象为`Date`，则type被设置为`String`
2. 否则，type被设置为`Number`

##### Date数据类型特殊说明：

对于`Date`数据类型，我们更多期望获得的是其转为时间后的字符串，而非毫秒值（时间戳），如果为`number`，则会取到对应的毫秒值，显然字符串使用更多。
其他类型对象按照取值的类型操作即可。

##### `ToPrimitive`总结
`ToPrimitive`转成何种原始类型，取决于type，type参数可选，若指定，则按照指定类型转换，若不指定，默认根据实用情况分两种情况，`Date`为`string`，其余对象为`number`。那么什么时候会指定type类型呢，那就要看下面两种转换方式了。

### toString

**Object.prototype.toString()**

`toString() `方法返回一个表示该对象的字符串。

每个对象都有一个 `toString()` 方法，当对象被表示为文本值时或者当以期望字符串的方式引用对象时，该方法被自动调用。

这里先记住，`valueOf()` 和 `toString() `在特定的场合下会自行调用。

### valueOf
`Object.prototype.valueOf()`方法返回指定对象的原始值。

`JavaScript` 调用 `valueOf()` 方法用来把对象转换成原始类型的值（数值、字符串和布尔值）。但是我们很少需要自己调用此函数，`valueOf` 方法一般都会被 `JavaScript` 自动调用。

不同内置对象的`valueOf`实现：

- String => 返回字符串值
- Number => 返回数字值
- Date => 返回一个数字，即时间值,字符串中内容是依赖于具体实现的
- Boolean => 返回Boolean的this值
- Object => 返回this

对照代码会更清晰一些：

```javascript
var str = new String('123');
console.log(str.valueOf());//123

var num = new Number(123);
console.log(num.valueOf());//123

var date = new Date();
console.log(date.valueOf()); //1526990889729

var bool = new Boolean('123');
console.log(bool.valueOf());//true

var obj = new Object({valueOf:()=>{
    return 1
}})
console.log(obj.valueOf());//1
```

### Number
`Number`运算符转换规则：

- `null` 转换为 0
- `undefined` 转换为 `NaN`
- `true` 转换为 1，`false` 转换为 0
- 字符串转换时遵循数字常量规则，转换失败返回` NaN`


> 注意：对象这里要先转换为原始值，调用`ToPrimitive`转换，type指定为`number`了，继续回到`ToPrimitive`进行转换。

### String
`String` 运算符转换规则

- `null` 转换为 `'null'`
- `undefined` 转换为 `undefined`
- `true` 转换为 `'true'`，`false` 转换为 `'false'`
- 数字转换遵循通用规则，极大极小的数字使用指数形式

> 注意：对象这里要先转换为原始值，调用`ToPrimitive`转换，`type`就指定为`string`了，继续回到`ToPrimitive`进行转换(上面有将到`ToPrimitive`的转换规则)。


```javascript
String(null)                 // 'null'
String(undefined)            // 'undefined'
String(true)                 // 'true'
String(1)                    // '1'
String(-1)                   // '-1'
String(0)                    // '0'
String(-0)                   // '0'
String(Math.pow(1000,10))    // '1e+30'
String(Infinity)             // 'Infinity'
String(-Infinity)            // '-Infinity'
String({})                   // '[object Object]'
String([1,[2,3]])            // '1,2,3'
String(['koala',1])          //koala,1
```

### Boolean
`ToBoolean` 运算符转换规则

除了下述 6 个值转换结果为 `false`，其他全部为` true`：

1. undefined
2. null
3. -0
4. 0或+0
5. NaN
6. ''（空字符串）

假值以外的值都是真值。其中包括所有对象（包括空对象）的转换结果都是`true`，甚至连`false`对应的布尔对象`new Boolean(false)`也是`true`


```javascript
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean('') // false

Boolean({}) // true
Boolean([]) // true
Boolean(new Boolean(false)) // true
```


# js转换规则不同场景应用
> 面试官问：知道了具体转换成什么的规则，但是都在什么情况下发生什么样的转换呢？


### 什么时候自动转换为string类型

- 在没有对象的前提下

    字符串的自动转换，主要发生在字符串的**加法运算**时。当一个值为字符串，另一个值为非字符串，则后者转为字符串。

```javascript
'2' + 1 // '21'
'2' + true // "2true"
'2' + false // "2false"
'2' + undefined // "2undefined"
'2' + null // "2null"
```

- 当有对象且与对象`+`时候

```javascript
//toString的对象
var obj2 = {
    toString:function(){
        return 'a'
    }
}
console.log('2'+obj2)
//输出结果2a

//常规对象
var obj1 = {
   a:1,
   b:2
}
console.log('2'+obj1)；
//输出结果 2[object Object]

//几种特殊对象
'2' + {} // "2[object Object]"
'2' + [] // "2"
'2' + function (){} // "2function (){}"
'2' + ['koala',1] // 2koala,1
```

对下面`'2'+obj2`详细举例说明如下：

1. 左边为`string`，`ToPrimitive`原始值转换后不发生变化
2. 右边转化时同样按照`ToPrimitive`进行原始值转换，由于指定的type是`number`，进行`ToPrimitive`转化调用`obj2.valueof()`,得到的不是原始值，进行第三步
3. 调用`toString()` ` return 'a'`
4. 符号两边存在`string`，而且是`+`号运算符则都采用`String`规则转换为`string`类型进行拼接 
5. 输出结果`2a`

对下面`'2'+obj1`详细举例说明如下：
1. 左边为`string`，`ToPrimitive`转换为原始值后不发生变化
2. 右边转化时同样按照`ToPrimitive`进行原始值转换，由于指定的type是`number`，进行`ToPrimitive`转化调用`obj2.valueof()`,得到`{ a: 1, b: 2 }`
3. 调用`toString()` ` return [object Object]`
4. 符号两边存在`string`，而且是+号运算符则都采用`String`规则转换为`string`类型进行拼接 
5. 输出结果`2[object Object]`

代码中几种特殊对象的转换规则基本相同，就不一一说明，大家可以想一下流程。

注意：不管是对象还不是对象，都有一个转换为原始值的过程，也就`是ToPrimitive`转换，只不过原始类型转换后不发生变化，对象类型才会发生具体转换。

#### string类型转换开发过程中可能出错的点：

```javascript
var obj = {
  width: '100'
};

obj.width + 20 // "10020"

```
预期输出结果120 实际输出结果10020

### 什么时候自动转换为Number类型

- 有加法运算符，但是无`String`类型的时候，都会优先转换为`Number`类型

    例子：

    ```javascript
    true + 0 // 1
    true + true // 2
    true + false //1
    ```
- **除了加法运算符，其他运算符都会把运算自动转成数值。**
例子：
    ```javascript
    '5' - '2' // 3
    '5' * '2' // 10
    true - 1  // 0
    false - 1 // -1
    '1' - 1   // 0
    '5' * []    // 0
    false / '5' // 0
    'abc' - 1   // NaN
    null + 1 // 1
    undefined + 1 // NaN
    
    //一元运算符（注意点）
    +'abc' // NaN
    -'abc' // NaN
    +true // 1
    -false // 0
    ```

>  注意：`null`转为数值时为0，而`undefined`转为数值时为`NaN`。

#### 判断等号也放在`Number`里面特殊说明

== 抽象相等比较与+运算符不同，不再是`String`优先，而是`Number`优先。
下面列举`x == y`的例子
1. 如果`x`,`y`均为`number`，直接比较
 没什么可解释的了

```javascript
1 == 2 //false
```


2. 如果存在对象，`ToPrimitive()`type为`number`进行转换，再进行后面比较

```javascript
var obj1 = {
    valueOf:function(){
        return '1'
    }
}
1 == obj1  //true
//obj1转为原始值，调用obj1.valueOf()
//返回原始值'1'
//'1'toNumber得到 1 然后比较 1 == 1
[] == ![] //true
//[]作为对象ToPrimitive得到 ''  
//![]作为boolean转换得到0 
//'' == 0 
//转换为 0==0 //true
```


3. 存在`boolean`，按照`ToNumber`将`boolean`转换为1或者0，再进行后面比较

```javascript
//boolean 先转成number，按照上面的规则得到1  
//3 == 1 false
//0 == 0 true
3 == true // false
'0' == false //true
```
 

4.如果`x`为`string`，`y`为`number`，`x`转成`number`进行比较

```javascript
//'0' toNumber()得到 0  
//0 == 0 true
'0' == 0 //true
```
 

### 什么时候进行布尔转换
 - 布尔比较时
 - `if(obj)` , `while(obj) `等判断时或者 三元运算符只能够包含布尔值
 
条件部分的每个值都相当于`false`，使用否定运算符后，就变成了`true`

```javascript
if ( !undefined
  && !null
  && !0
  && !NaN
  && !''
) {
  console.log('true');
} // true

//下面两种情况也会转成布尔类型
expression ? true : false
!! expression
```

# js中的数据类型判断

> 面试官问：如何判断数据类型？怎么判断一个值到底是数组类型还是对象?

三种方式，分别为 `typeof`、`instanceof` 和` Object.prototype.toString()`

### typeof
通过 `typeof`操作符来判断一个值属于哪种基本类型。

```javascript
typeof 'seymoe'    // 'string'
typeof true        // 'boolean'
typeof 10          // 'number'
typeof Symbol()    // 'symbol'
typeof null        // 'object' 无法判定是否为 null
typeof undefined   // 'undefined'

typeof {}           // 'object'
typeof []           // 'object'
typeof(() => {})    // 'function'
```

上面代码的输出结果可以看出，
1.  `null` 的判定有误差，得到的结果
如果使用 `typeof`，`null`得到的结果是`object`

2. 操作符对对象类型及其子类型，例如函数（可调用对象）、数组（有序索引对象）等进行判定，则除了函数都会得到 `object` 的结果。

综上可以看出`typeOf`对于判断类型还有一些不足，在对象的子类型和`null`情况下。

### instanceof
通过 `instanceof` 操作符也可以对对象类型进行判定，其原理就是测试构造函数的` prototype` 是否出现在被检测对象的原型链上。

```javascript
[] instanceof Array            // true
({}) instanceof Object         // true
(()=>{}) instanceof Function   // true
```

复制代码注意：`instanceof` 也不是万能的。
举个例子：

```javascript
let arr = []
let obj = {}
arr instanceof Array    // true
arr instanceof Object   // true
obj instanceof Object   // true
```

在这个例子中，`arr` 数组相当于 `new Array()` 出的一个实例，所以 `arr.__proto__ === Array.prototype`，又因为 `Array `属于 `Object` 子类型，即` Array.prototype.__proto__ === Object.prototype`，因此 `Object` 构造函数在 `arr` 的原型链上。所以 `instanceof` 仍然无法优雅的判断一个值到底属于数组还是普通对象。

还有一点需要说明下，有些开发者会说 `Object.prototype.__proto__ === null`，岂不是说 `arr instanceof null` 也应该为 `true`，这个语句其实会报错提示右侧参数应该为对象，这也印证 `typeof null` 的结果为 `object` 真的只是`javascript`中的一个` bug` 。

### Object.prototype.toString()
`Object.prototype.toString()` 可以说是判定 `JavaScript` 中数据类型的终极解决方法了，具体用法请看以下代码：

```javascript
Object.prototype.toString.call({})              // '[object Object]'
Object.prototype.toString.call([])              // '[object Array]'
Object.prototype.toString.call(() => {})        // '[object Function]'
Object.prototype.toString.call('seymoe')        // '[object String]'
Object.prototype.toString.call(1)               // '[object Number]'
Object.prototype.toString.call(true)            // '[object Boolean]'
Object.prototype.toString.call(Symbol())        // '[object Symbol]'
Object.prototype.toString.call(null)            // '[object Null]'
Object.prototype.toString.call(undefined)       // '[object Undefined]'

Object.prototype.toString.call(new Date())      // '[object Date]'
Object.prototype.toString.call(Math)            // '[object Math]'
Object.prototype.toString.call(new Set())       // '[object Set]'
Object.prototype.toString.call(new WeakSet())   // '[object WeakSet]'
Object.prototype.toString.call(new Map())       // '[object Map]'
Object.prototype.toString.call(new WeakMap())   // '[object WeakMap]'
```

我们可以发现该方法在传入任何类型的值都能返回对应准确的对象类型。用法虽简单明了，但其中有几个点需要理解清楚：

- 该方法本质就是依托`Object.prototype.toString() `方法得到对象内部属性 `[[Class]]`
- 传入原始类型却能够判定出结果是因为对值进行了包装
- `null` 和 `undefined` 能够输出结果是内部实现有做处理

# NaN相关总结

### `NaN`的概念
`NaN` 是一个全局对象的属性，`NaN` 是一个全局对象的属性，`NaN`是一种特殊的`Number`类型。

### 什么时候返回NaN （开篇第二道题也得到解决）

- 无穷大除以无穷大
- 给任意负数做开方运算
- 算数运算符与不是数字或无法转换为数字的操作数一起使用
- 字符串解析成数字

一些例子：

```javascript
Infinity / Infinity;   // 无穷大除以无穷大
Math.sqrt(-1);         // 给任意负数做开方运算
'a' - 1;               // 算数运算符与不是数字或无法转换为数字的操作数一起使用
'a' * 1;
'a' / 1;
parseInt('a');         // 字符串解析成数字
parseFloat('a');

Number('a');   //NaN
'abc' - 1   // NaN
undefined + 1 // NaN
//一元运算符（注意点）
+'abc' // NaN
-'abc' // NaN
```

# 误区
### `toString`和`String`的区别  
- `toString`
1. `toString()`可以将数据都转为字符串，但是`null`和`undefined`不可以转换。

    ```javascript
    console.log(null.toString())
    //报错 TypeError: Cannot read property 'toString' of null
    
    console.log(undefined.toString())
    //报错 TypeError: Cannot read property 'toString' of undefined
    ```
2. `toString()`括号中可以写数字，代表进制

    二进制：.toString(2); 
    
    八进制：.toString(8);
    
    十进制：.toString(10);
    
    十六进制：.toString(16);
    
- `String`

1.  `String()`可以将`null`和`undefined`转换为字符串，但是没法转进制字符串

    ```javascript
    console.log(String(null));
    // null
    console.log(String(undefined));
    // undefined
    ```