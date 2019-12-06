

## 为什么要学习this关键字
1. 面试会问啊！
总有一些面试官喜欢问你一段不可能这么写的代码。
看一道经典且古老的面试题(学完本文后，末尾会有一道更复杂的面试题等着你哦！)

    代码如下:
    ```javascript
    var a = 5;
    var obj = {
      a : 10,
      foo: function(){
        console.log(this.a)
      }
    }
    
    var bar = obj.foo
    obj.foo() 
    bar()
    ```
2.  我在读 Events 的 lib/events 源码的时候发现多次用到call关键字，看来有必要搞懂 this 与 call 相关的所有内容。

    其中几句代码是这样写的
    ```javascript
    // 场景1:
    function EventEmitter() {
      EventEmitter.init.call(this);
    }
    
    // 场景2:
    return this.listener.call(this.target);
    
    // 场景3:
    return listenerCount.call(emitter, type);
    ```
3. 箭头函数使用不当报错，在封装 Node.js 的一个 ORM 映射框架 Sequelize 时，封装表关联关系，由于使用箭头函数造成了读到的上下文发生变化，不是想要的 model 信息，而是指向了全局 。


4. call 关键字在写代码过程中还是比较常用的，有时候我们常常会使用 call 关键字来指定某个函数运行时的上下文，有时候还使用 call 关键字实现继承。

代码例子如下:
```javascript
var person = {
    "name": "koala"
};
function changeJob(company, work) {
    this.company = company;
    this.work    = work;
};

changeJob.call(person, '百度', '程序员');
console.log(person.work); // '程序员'
```

## 文章概览图


![文章概览图](https://user-gold-cdn.xitu.io/2019/9/11/16d1dc3fe9fe710a?w=880&h=966&f=jpeg&s=141780 ':size=600')

> 注:本文中如不特殊说明都是，输出结果都是指浏览器中输出结果。
 Github 博客开源项目 https://github.com/koala-coding/goodBlog

## 函数调用
JS（ES5）里面有三种函数调用形式：
```javascript
func(p1, p2) 
obj.child.method(p1, p2)
func.call(context, p1, p2) // 这里先不讲 apply
```
好多初学者都只用到过前两种情况，而且认为前两者优于第三者。直到几天前想系统复习一下this关键字，找this相关的各种资料，在知乎看到了一个关于this的讨论。
说第三种形式才是正常的调用形式。
```javascript
func.call(context,p1,p2)
```
其它两种都是语法糖，可以等价的变为`call`形式。
`func(p1, p2)等价于 func.call(undefined, p1, p2);`

`obj.child.method(p1, p2) 等价于 obj.child.method.call(obj.child, p1, p2);`
这么看我们的函数调用只有一种形式:
```javascript
func.call(context,p1,p2)
```
这时候是不是就知道this是什么了，就是上面的context。回到我开篇提到的面试题。
```javascript
var a = 5;
var obj = {
  a : 10,
  foo: function(){
    console.log(this.a)
  }
}

var bar = obj.foo
obj.foo() 
bar()
```
- obj.foo() 转化为call的形式就是obj.foo.call(obj) 

所以this指向了obj
- bar() 转化为call的形式就是bar.call()
由于没有传 context，所以 this 就是 undefined。
如果是在浏览器中，默认的 this 就是 window 对象；如果是在 Node.js 环境中默认的 this 就是 global 对象。
最终，在浏览器中运行结果为 5；在 Node.js 环境中为 undefined。

### Node.js 环境下指向全局的this关键字说明(你可能不知道)
为什么在浏览器或者前端环境可以直接正常输出值，而在 Node.js 环境中输出的却是`undefined`。
看一下这段代码你可能就懂了。

```javascript
(function(exports, require, module, __filename, __dirname) {
    {
    // 模块的代码
    // 所以那整个代码应该在这里吧
    var a = 10;
    function A(){
        a = 5;
        console.log(a);
        console.log(this.a);
    }
    // const haha = new A();
    A();
    }
});
```
先说一下 Node.js 环境下在运行某个 js 模块代码时候发生了什么，Node.js 在执行代码之前会使用一个代码封装器进行封装，例如下面所示：
```javascript
(function(exports, require, module, __filename, __dirname) {
    {
    // 模块的代码
    // 所以那整个代码应该在这里吧
    }
});
```
这段代码在 Node.js 环境下输出结果为`5，undefined`是不是就能理解了。
这里面的this是默认绑定指向全局，当输出this.a的时候，全局应该指向这个闭包的最外层。所以输出结果式是undefined。
### []语法中的this关键字
```javascript
function fn (){ console.log(this) }
var arr = [fn, fn2]
arr[0]() // 这里面的 this 又是什么呢？ 
```
我们可以把 arr[0]( ) 想象为arr.0( )，虽然后者的语法错了，但是形式与转换代码里的 obj.child.method(p1, p2) 对应上了，于是就可以愉快的转换了：
```
        arr[0]() 
假想为    arr.0()
然后转换为 arr.0.call(arr)
那么里面的 this 就是 arr 了
```
## this绑定原则
### 默认绑定
默认绑定是函数针对的独立调用的时候，不带任何修饰的函数引用进行调用，非严格模式下 this 指向全局对象(浏览器下指向 Window，Node.js 环境是 Global ），严格模式下，this 绑定到 undefined ,严格模式不允许this指向全局对象。
```javascript
var a = 'hello'

var obj = {
    a: 'koala',
    foo: function() {
        console.log(this.a)
    }
}

var bar = obj.foo

bar()              // 浏览器中输出: "hello"
```
这段代码，`bar()`就是默认绑定，函数调用的时候，前面没有任何修饰调用，也可以用之前的 `call`函数调用形式理解，所以输出结果是`hello`。

#### 默认绑定的另一种情况
在函数中以函数作为参数传递，例如`setTimeOut`和`setInterval`等，这些函数中传递的函数中的`this`指向，在非严格模式指向的是全局对象。

例子：
```javascript
var name = 'koala';
var person = {
    name: '程序员成长指北',
    sayHi: sayHi
}
function sayHi(){
    setTimeout(function(){
        console.log('Hello,', this.name);
    })
}
person.sayHi();
setTimeout(function(){
    person.sayHi();
},200);
// 输出结果 Hello,koala
// 输出结果 Hello,koala
```

### 隐式绑定
判断 this 隐式绑定的基本标准:函数调用的时候是否在上下文中调用，或者说是否某个对象调用函数。

例子:
```javascript
var a = 'koala'

var obj = {
    a: '程序员成长指北',
    foo: function() {
        console.log(this.a)
    }
}
obj.foo()       // 浏览器中输出: "程序员成长指北"
```
foo 方法是作为对象的属性调用的，那么此时 foo 方法执行时，this 指向 obj 对象。

**隐式绑定的另一种情况**

当有多层对象嵌套调用某个函数的时候，如 `对象.对象.函数`,this 指向的是最后一层对象。

例子:
```javascript
function sayHi(){
    console.log('Hello,', this.name);
}
var person2 = {
    name: '程序员成长指北',
    sayHi: sayHi
}
var person1 = {
    name: 'koala',
    friend: person2
}
person1.friend.sayHi();

// 输出结果为 Hello, 程序员成长指北
```
看完这个例子，是不是也就懂了隐式调用的这种情况。

### 显式绑定
显式绑定，通过函数call apply bind 可以修改函数this的指向。call 与 apply 方法都是挂载在 Function 原型下的方法，所有的函数都能使用。

#### call 和 apply 的区别
1. call和apply的第一个参数会绑定到函数体的this上，如果`不传参数`，例如`fun.call()`，非严格模式，this默认还是绑定到全局对象
2. call函数接收的是一个参数列表，apply函数接收的是一个参数数组。
```javascript
unc.call(thisArg, arg1, arg2, ...)        // call 用法
func.apply(thisArg, [arg1, arg2, ...])     // apply 用法
```
看代码例子:
```javascript
var person = {
    "name": "koala"
};
function changeJob(company, work) {
    this.company = company;
    this.work    = work;
};

changeJob.call(person, '百度', '程序员');
console.log(person.work); // '程序员'

changeJob.apply(person, ['百度', '测试']);
console.log(person.work); // '测试'
```
#### call和apply的注意点
这两个方法在调用的时候，如果我们传入数字或者字符串，这两个方法会把传入的参数转成对象类型。

例子:
```javascript
var number = 1, string = '程序员成长指北';
function getThisType () {
    var number = 3;
    console.log('this指向内容',this);
    console.log(typeof this);
}
getThisType.call(number);
getThisType.apply(string); 
// 输出结果
// this指向内容 [Number: 1]
// object
// this指向内容 [String: '程序员成长指北']
// object
```
#### bind函数
> bind 方法
会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。(定义内容来自于 MDN )

```javascript
func.bind(thisArg[, arg1[, arg2[, ...]]])    // bind 用法
```
例子:
```javascript
var publicAccounts = {
    name: '程序员成长指北',
    author: 'koala',
    subscribe: function(subscriber) {
        console.log(subscriber + this.name)
    }
}

publicAccounts.subscribe('小红')   // 输出结果: "小红 程序员成长指北"

var subscribe1 = publicAccounts.subscribe.bind({ name: 'Node成长指北', author: '考拉' }, '小明 ')
subscribe1()       // 输出结果: "小明 Node成长指北"
```

### new 绑定
使用new调用函数的时候，会执行怎样的流程：
1. 创建一个空对象
2. 将空对象的 _proto_ 指向原对象的 prototype
3. `执行构造函数中的代码`
4. 返回这个新对象

例子:
```javascript
function study(name){
    this.name = name;
	
}
var studyDay = new study('koala');
console.log(studyDay);
console.log('Hello,', studyDay.name);
// 输出结果
// study { name: 'koala' }
// hello，koala
```
在`new study('koala')`的时候，会改变this指向，将`this指向指定到了studyDay对象`。
注意:如果创建新的对象，构造函数不传值的话，新对象中的属性不会有值，但是新的对象中会有这个属性。

#### 手动实现一个new创建对象代码(多种实现方式哦)
```javascript
function New(func) {
    var res = {};
    if (func.prototype !== null) {
        res.__proto__ = func.prototype;
    }
    var ret = func.apply(res, Array.prototype.slice.call(arguments, 1));
    if ((typeof ret === "object" || typeof ret === "function") && ret !== null) {
        return ret;
    }
    return res;
}
var obj = New(A, 1, 2);
// equals to
var obj = new A(1, 2);
```

## this绑定优先级
上面介绍了 this 的四种绑定规则，但是一段代码有时候会同时应用多种规则，这时候 this 应该如何指向呢？其实它们也是有一个先后顺序的，具体规则如下:

**new绑定 > 显式绑定 > 隐式绑定 > 默认绑定**

## 箭头函数中的 this
### 箭头函数
在讲箭头函数中的 this 之前，先讲一下箭头函数。

**定义**
> MDN:箭头函数表达式的语法比函数表达式更短，并且不绑定自己的this，arguments，super或 new.target。这些函数表达式最适合用于非方法函数(non-method functions)，并且它们不能用作构造函数。

- 箭头函数中没有 arguments

常规函数可以直接拿到 arguments 属性，但是在箭头函数中如果使用 arguments 属性，拿到的是箭头函数外层函数的 arguments 属性。

例子:
```javascript
function constant() {
    return () => arguments[0]
}

let result = constant(1);
console.log(result()); // 1
```
**如果我们就是要访问箭头函数的参数呢？**

你可以通过 ES6 中 命名参数 或者 rest 参数的形式访问参数
```javascript
let nums = (...nums) => nums;
```
- 箭头函数没有构造函数

箭头函数与正常的函数不同，箭头函数没有构造函数 constructor，因为没有构造函数，所以也不能使用 new 来调用，如果我们直接使用 new 调用箭头函数，会报错。

例子:
```javascript
let fun = ()=>{}
let funNew = new fun(); 
// 报错内容 TypeError: fun is not a constructor
```
- 箭头函数没有原型

原型 prototype 是函数的一个属性，但是对于箭头函数没有它。

例子:
```javascript
let fun = ()=>{}
console.loh(fun.prototype); // undefined
```
- 箭头函数中没有 super

上面说了没有原型，连原型都没有，自然也不能通过 super 来访问原型的属性，所以箭头函数也是没有 super 的，不过跟 this、arguments、new.target 一样，这些值由外围最近一层非箭头函数决定。

- `箭头函数中没有自己的this`

箭头函数中没有自己的 this，箭头函数中的 this 不能用 call()、apply()、bind() 这些方法改变 this 的指向，箭头函数中的 this 直接指向的是`调用函数的 上一层运行时`。
```javascript
let a = 'kaola'

let obj = {
    a: '程序员成长指北',
    foo: () => {
        console.log(this.a)
    }
}

obj.foo()             // 输出结果: "koala"
```
看完输出结果，怕大家有疑问还是分析一下，前面我说的箭头函数中this直接指向的是`调用函数的上一层运行时`，这段代码`obj.foo`在调用的时候如果是不使用箭头函数this应该指向的是 obj ，但是使用了箭头函数，往上一层查找，指向的就是全局了，所以输出结果是`koala`。
### 自执行函数
什么是自执行函数？
自执行函数在我们在代码只能够定义后，无需调用，会自动执行。开发过程中有时间测试某一小段代码报错会使用。
代码例子如下：
```javascript
(function(){
    console.log('程序员成长指北')
})()
```
或者
```javascript
(function(){
    console.log('程序员成长指北')
}())
```
但是如果使用了箭头函数简化一下就只能使用第一种情况了。使用第二种情况简化会报错。
```javascript
(() => {
    console.log('程序员成长指北')
})()
```

## this应用场景
应用场景其实就是开篇说到的为什么写这篇文章，再重复一下。
1. 面试官他考！
2. 看源码总看见，有时候想确认一下当前的上下文指向。为什么源码中用的多，大家可以想想这个问题。
3. 我们写代码也会用，经常会出现用 call 指向某个对象的上下文，或者实现继承等等。

## 学后小练习
学到这里是不是发现开篇那道面试题有点简单，已经不能满足你目前对于 this 关键字的知识储备。好的，我们来一道复杂点的面试题。

代码如下：
```javascript
var length = 10;
function fn() {
    console.log(this.length);
}
 
var obj = {
  length: 5,
  method: function(fn) {
    fn();
    arguments[0]();
  }
};
 
obj.method(fn, 1);//输出是什么？
```

这段代码的输出结果是:`10,2`

认真读文章的应该都能正确的答出答案，每一个细节文章中都讲了，我在这就不具体分析，如果不懂可以再读文章，或者直接加我好友我们一起讨论，kaola 是一个乐于分享的人，期待与你共同进步。

声明:任何形式转载都请联系本人，如有问题也感谢您的指出和建议哦。

## 参考文章
- MDN中this关键字的讲解 https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this
- 知乎的一个关于this讨论 :https://www.zhihu.com/question/19636194
- 阮一峰老师的ES6书籍中箭头函数内容
- 书籍《你不知道的javascript》

