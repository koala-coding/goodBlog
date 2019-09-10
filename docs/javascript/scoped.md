## 快速导航
- [1.什么是作用域](#_1什么是作用域)
- [2.JavaScript中的作用域](#_2JavaScript中的作用域)
  * [2.1全局作用域](#_21全局作用域)
  * [2.2局部作用域](#_22局部作用域)
- [3.作用域链](#_3作用域链)
    * [JavaScript是如何执行的？](#_31JavaScript是如何执行的？)
    * [作用域链概念](#_32作用域链概念)
    * [`找` 过程LHS和RHS查询特殊说明](#_33找过程LHS和RHS查询特殊说明)
    * [作用域链总结](#_34作用域链总结)

## 1.什么是作用域
作用域是你的代码在运行时,某些特定部分中的变量,函数和对象的可访问性。换句话说，作用域决定了变量与函数的可访问范围，即`作用域控制着变量与函数的可见性和生命周期`。

## 2.JavaScript中的作用域
在 JavaScript 中有两种作用域
- 全局作用域
- 局部作用域

如果一个变量在函数外面或者大括号`{}`外声明,那么就定义了一个`全局作用域`,在ES6之前局部作用域只包含了函数作用域,**ES6为我们提供的`块级作用域`**,也属于局部作用域

### 2.1全局作用域
拥有全局作用域的对象可以在代码的任何地方访问到, 在js中一般有以下几种情形拥有全局作用域:
1. 最外层的函数以及最外层变量:

```JavaScript
var globleVariable= 'global';  // 最外层变量
function globalFunc(){         // 最外层函数
    var childVariable = 'global_child';  //函数内变量
    function childFunc(){        // 内层函数
        console.log(childVariable);
    }
    console.log(globleVariable)
}
console.log(globleVariable);  // global
globalFunc();                 // global
console.log(childVariable)   // childVariable is not defined
console.log(childFunc)       // childFunc is not defined

```
从上面代码中可以看到`globleVariable`和`globalFunc`在任何地方都可以访问到, 反之不具有全局作用域特性的变量只能在其作用域内使用。

2. 未定义直接赋值的变量(由于变量提升使之成为全局变量)

```javascript
function func1(){
    special = 'special_variable';
    var normal = 'normal_variable';
}
func1();
console.log(special);    //special_variable
console.log(normal)     // normal is not defined
```

虽然我们可以在全局作用域中声明函数以及变量, 使之成为全局变量, 但是不建议这么做,因为这可能会和其他的变量名冲突,一方面如果我们再使用`const`或者`let`声明变量, 当命名发生冲突时会报错。

```javascript
// 变量冲突
var globleVariable = "person";
let globleVariable = "animal"; // Error, thing has already been declared
```

另一方面如果你使用`var`申明变量，第二个申明的同样的变量将覆盖前面的,这样会使你的代码很难调试。

```javascript
var name = 'koala'
var name = 'xiaoxiao'
console.log(name);  // xiaoxiao
```

### 2.2局部作用域
和全局作用于相反，局部作用域一般只能在固定代码片段内可以访问到。最常见的就是**函数作用域**。

#### 2.2.1 函数作用域
> 定义在函数中的变量就在函数作用域中。并且函数在每次调用时都有一个不同的作用域。这意味着同名变量可以用在不同的函数中。因为这些变量绑定在不同的函数中，拥有不同作用域，彼此之间不能访问。

```javascript
//全局作用域
function test(){
    var num = 9;
    // 内部可以访问
    console.log("test中："+num);
}
//test外部不能访问
console.log("test外部:"+num);
```

注意点：

- 如果在函数中定义变量时,如果不添加var关键字,造成变量提升，这个变量成为一个全局变量。

```javascript
function doSomeThing(){
    // 在工作中一定避免这样写
    thing = 'writting';
    console.log('内部：'+thing);
}
console.log('外部:'+thing)
```
- 任何一对花括号`｛...｝`中的语句集都属于一个块, 在es6之前，在块语句中定义的变量将保留在它已经存在的作用域中：

```javascript
var name = '程序员成长指北';
for(var i=0; i<5; i++){
    console.log(i)
}
console.log('{}外部:'+i);
// 0 1 2 3 4  {}外部:5
```
我们可以看到变量`name`和变量`i`是同级作用域。



#### 2.2.2 在ES6块级作用域未讲解之前注意点

##### 变量提升
变量提升英文名字hoisting,MDN中对它的解释是变量申明是在任意代码执行前处理的，在代码区中任意地方申明变量和在最开始（最上面）的地方申明是一样的。也就是说，看起来一个变量可以在申明之前被使用！这种行为就是所谓的“hoisting”，也就是变量提升，看起来就像变量的申明被自动移动到了函数或全局代码的最顶上。
看一段代码：

```JavaScript
var tmp = new Date();
function f() {
    console.log(tmp);
    if(false) {
        var tmp='hello';
    }
}
```
这道题应该很多小伙伴在面试中遇到过，有人会认为输出的是当前日期。但是正确的结果是undefined。这就是由于变量提升造成的，在这里**申明提升了，定义的内容并不会提升**，提升后对应的代码如下：

```JavaScript
var tmp = new Date();
function f() {
    var tmp;
    console.log(tmp);
    if(false) {
        tmp='hello';
    }
}
f();
```
console在输出的时候，tmp变量仅仅申明了但未定义。所以输出undefined。虽然能够输出，但是并不推荐这种写法推荐的做法是在申明变量的时候，将所用的变量都写在作用域（全局作用域或函数作用域）的最顶上，这样代码看起来就会更清晰，更容易看出来哪个变量是来自函数作用域的，哪个又是来自作用域链

##### 重复声明
看一个例子：
```javascript
// var
var name = 'koloa';
console.log(name); // koala
if(true){
    var name = '程序员成长指北';
    console.log(name); // 程序员成长指北
}
console.log(name); // 程序员成长指北

```
 虽然看起来里面name申明了两次，但上面说了，js的var变量只有全局作用域和函数作用域两种，且申明会被提升，因此实际上name只会在最顶上开始的地方申明一次，`var name='`程序员成长指北'的申明会被忽略，仅用于赋值。也就是说上面的代码实际上跟下面是一致的。
```javascript
// var
var name = 'koloa';
    console.log(name); // koala
if(true){
    name = '程序员成长指北';
    console.log(name); // 程序员成长指北
}
console.log(name); // 程序员成长指北
```
##### 变量和函数同时出现的提升
如果有函数和变量同时声明了，会出现什么情况呢？看下面但代码

```JavaScript
console.log(foo);
var foo ='i am koala';
function foo(){}

```
输出结果是`function foo(){}`,也就是函数内容

如果是另外一种形式呢？

```JavaScript
console.log(foo);
var foo ='i am koala';
var foo=function (){}
```
输出结果是`undefined`

对两种结果进行分析说明：

第一种：函数申明。就是上面第一种，`function foo(){}`这种形式

另一种：函数表达式。就是上面第二种，`var foo=function(){}`这种形式

第二种形式其实就是var变量的声明定义，因此上面的第二种输出结果为undefined应该就能理解了。

而第一种函数申明的形式，在提升的时候，会被整个提升上去，包括函数定义的部分！因此第一种形式跟下面的这种方式是等价的！
```JavaScript
var foo=function (){}
console.log(foo);
var foo ='i am koala';
```

原因是：
1. 函数声明被提升到最顶上；
2. 申明只进行一次，因此后面`var foo='i am koala'`的申明会被忽略。
3. 函数申明的优先级优于变量申明，且函数声明会连带定义一起被提升（这里与变量不同）


接下来讲，在ES6中引入的块级作用域之后的事！

#### 2.2.2 块级作用域
>ES6新增了`let`和`const`命令，可以用来创建块级作用域变量，使用`let`命令声明的变量只在`let`命令所在`代码块`内有效。

let 声明的语法与 var 的语法一致。你基本上可以用 let 来代替 var 进行变量声明，但会将变量的作用域限制在当前代码块中。块级作用域有以下几个特点：

- 变量不会提升到代码块顶部且不允许从外部访问块级作用域内部变量
```javascript
console.log(bar);//抛出`ReferenceErro`异常: 某变量 `is not defined`
let bar=2;
for (let i =0; i<10;i++){
    console.log(i)
}
console.log(i);//抛出`ReferenceErro`异常: 某变量 `is not defined`
```
其实这个特点带来了许多好处，开发者需要检查代码时候，可以避免在作用域外意外但使用某些变量，而且保证了变量不会被混乱但复用，提升代码的可维护性。就像代码中的例子，一个只在for循环内部使用的变量i不会再去污染整个作用域。
- 不允许反复声明

ES6的`let`和`const`不允许反复声明，与`var`不同
```javascript
// var
function test(){
    var name = 'koloa';
    var name = '程序员成长指北';
    console.log(name); // 程序员成长指北
}

// let || const
function test2(){
    var name ='koloa';
    let name= '程序员成长指北'; 
    // Uncaught SyntaxError: Identifier 'count' has already been declared
}
```


看到这里是不是感觉到了块级作用域的出现还是很有必要的。

## 3.作用域链
在讲解作用域链之前先说一下，先了解一下 JavaScript是如何执行的？
### 3.1JavaScript是如何执行的？

![](https://user-gold-cdn.xitu.io/2019/6/27/16b94c342168e6da?w=2198&h=1138&f=png&s=308834)
JavaScript代码执行分为两个阶段：
#### 3.1.1 分析阶段
javascript编译器编译完成，生成代码后进行分析
- 分析函数参数
- 分析变量声明
- 分析函数声明

分析阶段的核心，在分析完成后（也就是接下来函数执行阶段的瞬间）会创建一个`AO(Active Object 活动对象)`

#### 3.1.2 执行阶段
分析阶段分析成功后，会把给`AO(Active Object 活动对象)`给执行阶段

- 引擎询问作用域，作用域中是否有这个叫X的变量
- 如果作用域有X变量，引擎会使用这个变量
- 如果作用域中没有，引擎会继续寻找（向上层作用域），如果到了最后都没有找到这个变量，引擎会抛出错误。

执行阶段的核心就是`找`,具体怎么`找`，后面会讲解`LHS查询`与`RHS查询`。

#### 3.1.3 JavaScript执行举例说明
看一段代码：

```javascript
function a(age) {
    console.log(age);
    var age = 20
    console.log(age);
    function age() {
    }
    console.log(age);
}
a(18);
```
###### 首先进入分析阶段
前面已经提到了，函数运行的瞬间，创建一个AO (Active Object 活动对象)

```javascript
AO = {}
```
**第一步：分析函数参数：**

```javascript
形式参数：AO.age = undefined
实参：AO.age = 18
```
**第二步，分析变量声明：**
```javascript
// 第3行代码有var age
// 但此前第一步中已有AO.age = 18, 有同名属性,不做任何事
即AO.age = 18
```
**第三步，分析函数声明：**

```javascript
// 第5行代码有函数age
// 则将function age(){}付给AO.age
AO.age = function age() {}
```
**函数声明注意点**：AO上如果有与函数名同名的属性,则会被此函数覆盖。但是一下面这种情况

```javascript
var age = function () {
            console.log('25');
        }
```
声明的函数并不会覆盖AO链中同名的属性

###### 进入执行阶段
分析阶段分析成功后，会把给`AO(Active Object 活动对象)`给执行阶段，引擎会询问作用域，`找`的过程。所以上面那段代码AO链中最初应该是

```javascript
AO.age = function age() {}
//之后
AO.age=20
//之后
AO.age=20
```
所以最后的输出结果是：

```javascript
function age(){
    
}
20
20
```

### 3.2作用域链概念
看了前面一个完整的javascript函数执行过程，让我们来说下作用域链的概念吧。`JavaScript上每一个函数执行时，会先在自己创建的AO上找对应属性值。若找不到则往父函数的AO上找，再找不到则再上一层的AO,直到找到大boss:window（全局作用域）。 而这一条形成的“AO链” 就是JavaScript中的作用域链。`

### 3.3`找`过程LHS和RHS查询特殊说明

LHS，RHS 这两个术语就是出现在引擎对变量进行查询的时候。在《你不知道的Javascript(上)》也有很清楚的描述。在这里，我想引用`freecodecamp `上面的回答来解释：

> LHS = 变量赋值或写入内存。想象为将文本文件保存到硬盘中。 RHS = 变量查找或从内存中读取。想象为从硬盘打开文本文件。 Learning Javascript, LHS RHS

#### 3.3.1 LHS和RHS特性

- 都会在所有作用域中查询
- 严格模式下，找不到所需的变量时，引擎都会抛出`ReferenceError`异常。
- 非严格模式下，`LHR`稍微比较特殊: 会自动创建一个全局变量
- 查询成功时，如果对变量的值进行不合理的操作，比如：对一个非函数类型的值进行函数调用，引擎会抛出`TypeError`异常

#### 3.3.2 LHS和RHS举例说明
例子来自于《你不知道的Javascript(上)》

```javascript
function foo(a) {
    var b = a;
    return a + b;
}
var c = foo( 2 );
```
直接看引擎在作用域`找`这个过程：
**LSH（写入内存）：**

```javascript
c=, a=2(隐式变量分配), b=
```
**RHS(读取内存)**：

```javascript
读foo(2), = a, a ,b
(return a + b 时需要查找a和b)
```


### 3.4作用域链总结
最后对作用域链做一个总结，引用《你不知道的Javascript(上)》中的一张图解释

![](https://user-gold-cdn.xitu.io/2019/6/27/16b94c2e994197b0?w=1420&h=1552&f=png&s=223558)
