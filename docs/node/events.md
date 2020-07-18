---
title: 源码解读,一文彻底搞懂Events模块
date: 2019-09-05
tags:
   - Node.js
   - Events
---


**前言**

为什么写这篇文章？
- 清楚的记得刚找node工作和面试官聊到了事件循环，然后面试官问事件是如何产生的？什么情况下产生事件。。。
- Events 在哪些场景应用到了？
- 之前封装了一个 RxJava 的开源网络请求框架，也是基于发布-订阅模式，语言都是相通的，挺有趣。表情符号
- Events 模块是我公众号 Node.js 进阶路线的一部分

作者简介：koala，专注完整的 Node.js 技术栈分享，从 JavaScript 到 Node.js,再到后端数据库，祝您成为优秀的高级 Node.js 工程师。【程序员成长指北】作者，Github 博客开源项目 https://github.com/koala-coding/goodBlog

## 面试会问
> - 说一下 Node.js 哪里应用到了发布/订阅模式
> - Events 模块在实际项目开发中有使用过吗？具体应用场景是？
> - Events 监听函数的执行顺序是异步还是同步的？
> - 说几个 Events 模块的常用函数吧？
> - 模拟实现 Node.js 的核心模块 Events

文章首发[Github](https://github.com/koala-coding/goodBlog) 博客开源项目 [https://github.com/koala-coding/goodBlog](https://github.com/koala-coding/goodBlog)
## 发布/订阅者模式
`发布/订阅者模式`应该是我在开发过程中遇到的最多的设计模式。`发布/订阅者模式`，也可以称之为消息机制，定义了一种依赖关系，这种依赖关系可以理解为 `1对N` (注意：不一定是1对多，有时候也会1对1哦)，观察者们同时监听某一个对象相应的状态变换，一旦变化则通知到所有观察者，从而触发观察者相应的事件，该设计模式解决了主体对象与观察者之间功能的`耦合`。
### 生活中的发布/订阅者模式
#### 警察抓小偷
在现实生活中，警察抓小偷是一个典型的观察者模式「这以一个惯犯在街道逛街然后被抓为例子」，这里小偷就是被观察者，各个干警就是观察者，干警时时观察着小偷，当小偷正在偷东西「就给干警发送出一条信号，实际上小偷不可能告诉干警我有偷东西」，干警收到信号，出击抓小偷。这就是一个观察者模式

#### 订阅了某个报社的报纸
生活中就像是去报社订报纸，你喜欢读什么报就去报社去交钱订阅，当发布了新报纸的时候，报社会向所有订阅了报纸的每一个人发送一份，订阅者就可以接收到。

#### 你订阅了我的公众号
我这个微信公号作者是发布者，您这些微信用户是订阅者「我发送一篇文章的时候，关注了【程序员成长指北】的订阅者们都可以收到文章。

### 实例的代码实现与分析
以大家`订阅公众号`为例子，看看`发布/订阅模式`如何实现的。(以订阅报纸作为例子的原因，可以增加一个`type`参数，用于区分订阅不同类型的公众号，如有的人订阅的是前端公众号，有的人订阅的是 Node.js 公众号，使用此属性来标记。这样和接下来要讲的 EventEmitter 源码更相符，另一个原因是这样你只要打开一个订阅号文章是不是就想到了发布-订阅者模式呢。)

代码如下:

```javascript
let officeAccounts ={
    // 初始化定义一个存储类型对象
    subscribes:{
        'any':[]
    },
    // 添加订阅号
    subscribe:function(type='any',fn){
        if(!this.subscribes[type]){
            this.subscribes[type] = [];
        }
        this.subscribes[type].push(fn);//将订阅方法存在数组中
    },
    // 退订
    unSubscribe:function(type='any',fn){
        this.subscribes[type] = 
        this.subscribes[type].filter((item)=>{
            return item!=fn;// 将退订的方法从数组中移除 
        });
    },
    // 发布订阅
    publish:function(type='any',...args){
        this.subscribes[type].forEach(item => {
            item(...args);// 根据不同的类型调用相应的方法
        });
    }

}

```
以上就是一个最简单的观察者模式的实现，可以看到代码非常的简单，核心原理就是将订阅的方法按分类存在一个数组中，当发布时取出执行即可

接下里看小明订阅【程序员成长指北】文章的代码：

```javascript
let xiaoming = {
    readArticle:function (info) {
        console.log('小明收到的',info);
    }
};

let xiaogang = {
    readArticle:function (info) {
        console.log('小刚收到的',info);
    }
};

officeAccounts.subscribe('程序员成长指北',xiaoming.readArticle);
officeAccounts.subscribe('程序员成长指北',xiaogang.readArticle);
officeAccounts.subscribe('某公众号',xiaoming.readArticle);

officeAccounts.unSubscribe('某公众号',xiaoming.readArticle);

officeAccounts.publish('程序员成长指北','程序员成长指北的Node文章');
officeAccounts.publish('某公众号','某公众号的文章');

```
运行结果:

```javascript
小明收到的 程序员成长指北的Node文章
小刚收到的 程序员成长指北的Node文章
```
- 结论

通过观察现实生活中的三个例子以及代码实例发现发布/订阅模式的确是1对N的关系。当发布者的状态发生改变时，所有订阅者都会得到通知。

![](http://img.xiaogangzai.cn/node_events_01.jpg)

- 发布/订阅模式的特点和结构
三要素： 

1. 发布者  
2. 订阅者   
3. 事件(订阅)

### 发布/订阅者模式的优缺点
- 优点

主体和观察者之间完全透明，所有的消息传递过程都通过消息调度中心完成，也就是说具体的业务逻辑代码将会是在消息调度中心内，而主体和观察者之间实现了完全的**松耦合**。对象直接的解耦，异步编程中，可以更松耦合的代码编写。

- 缺点

程序易读性显著降低；多个发布者和订阅者嵌套在一起的时候，程序难以跟踪，其实还是代码不易读，嘿嘿。

## EventEmitter 与 发布/订阅模式的关系
 Node.js 中的 EventEmitter
 模块就是用了发布/订阅这种设计模式，发布/订阅 模式在主体与观察者之间引入消息调度中心，主体和观察者之间完全透明，所  有的消息传递过程都通过消息调度中心完成，也就是说具体的业务逻辑代码将会是在消息调度中心内完成。

### 事件的基本组成要素


![](http://img.xiaogangzai.cn/node_events_02.jpg)

通过Api的对比，来看看Events模块

### EventEmitter 定义
Events是 Node.js 中一个使用率很高的模块，其它原生node.js模块都是基于它来完成的，比如流、HTTP等。它的核心思想就是 Events 模块的功能就是一个`事件绑定与触发`，所有继承自它的实例都具备事件处理的能力。
### EventEs 的一些常用官方API源码与发布/订阅模式对比学习

本模块的官方 Api 讲解不是直接带大家学习文档，而是
通过`对比`发布/订阅设计模式自己手写一个版本 Events 的核心代码来学习并记住Api
#### Events 模块
Events 模块只有一个 EventEmitter 类，首先定义类的基本结构
```
function EventEmitter() {
    //私有属性，保存订阅方法
    this._events = {};
}

//默认设置最大监听数
EventEmitter.defaultMaxListeners = 10;

module.exports = EventEmitter;
```


#### on 方法
on 方法，该方法用于订阅事件(这里 on 和 addListener 说明下)，Node.js 源码中这样把它们俩赋值了下，我也不太懂为什么？知道的小伙伴可以告诉我为什么要这样做哦。
```javascript
EventEmitter.prototype.addListener = function addListener(type, listener) {
  return _addListener(this, type, listener, false);
};

EventEmitter.prototype.on = EventEmitter.prototype.addListener;
```
接下来是我们对on方法的具体实践：
```javascript
EventEmitter.prototype.on =
    EventEmitter.prototype.addListener = function (type, listener, flag) {
		//保证存在实例属性
        if (!this._events) this._events = Object.create(null);

        if (this._events[type]) {
            if (flag) {//从头部插入
                this._events[type].unshift(listener);
            } else {
                this._events[type].push(listener);
            }

        } else {
            this._events[type] = [listener];
        }
		//绑定事件，触发newListener
        if (type !== 'newListener') {
            this.emit('newListener', type);
        }
    };
```
因为有其它子类需要继承自EventEmitter，因此要判断子类是否存在_event属性，这样做是为了保证子类必须存在此实例属性。而flag标记是一个订阅方法的插入标识，如果为'true'就视为插入在数组的头部。可以看到，这就是观察者模式的订阅方法实现。
#### emit方法
```javascript
EventEmitter.prototype.emit = function (type, ...args) {
    if (this._events[type]) {
        this._events[type].forEach(fn => fn.call(this, ...args));
    }
};
```
emit方法就是将订阅方法取出执行，使用call方法来修正this的指向，使其指向子类的实例。
#### once方法
```javascript
EventEmitter.prototype.once = function (type, listener) {
    let _this = this;

    //中间函数，在调用完之后立即删除订阅
    function only() {
        listener();
        _this.removeListener(type, only);
    }
    //origin保存原回调的引用，用于remove时的判断
    only.origin = listener;
    this.on(type, only);
};
```
once方法非常有趣，它的功能是将事件订阅“一次”，当这个事件触发过就不会再次触发了。其原理是将订阅的方法再包裹一层函数，在执行后将此函数移除即可。
#### off方法
```javascript
EventEmitter.prototype.off =
    EventEmitter.prototype.removeListener = function (type, listener) {

        if (this._events[type]) {
        //过滤掉退订的方法，从数组中移除
            this._events[type] =
                this._events[type].filter(fn => {
                    return fn !== listener && fn.origin !== listener
                });
        }
    };
```
off方法即为退订，原理同观察者模式一样，将订阅方法从数组中移除即可。
#### prependListener方法
```javascript
EventEmitter.prototype.prependListener = function (type, listener) {
    this.on(type, listener, true);
};
```
码此方法不必多说了，调用on方法将标记传为true（插入订阅方法在头部）即可。
以上，就将EventEmitter类的核心方法实现了。

#### 其他一些不太常用api
- `emitter.listenerCount(eventName)`可以获取事件注册的`listener`个数
- `emitter.listeners(eventName)`可以获取事件注册的`listener`数组副本。

#### Api学习后的小练习
```javascript
//event.js 文件
var events = require('events'); 
var emitter = new events.EventEmitter(); 
emitter.on('someEvent', function(arg1, arg2) { 
    console.log('listener1', arg1, arg2); 
}); 
emitter.on('someEvent', function(arg1, arg2) { 
    console.log('listener2', arg1, arg2); 
}); 
emitter.emit('someEvent', 'arg1 参数', 'arg2 参数'); 
```
执行以上代码，运行的结果如下：
```javascript
$ node event.js 
listener1 arg1 参数 arg2 参数
listener2 arg1 参数 arg2 参数
```
#### 手写代码后的说明
手写Events模块代码的时候注意以下几点：
- 使用订阅/发布模式
- 事件的核心组成有哪些
- 写源码时候考虑一些范围和极限判断

注意:我上面的手写代码并不是性能最好和最完善的，目的只是带大家先弄懂记住他。举个例子：
最初的定义EventEmitter类，源码中并不是直接定义 `this._events = {}`，请看：
```javascript

function EventEmitter() {
  EventEmitter.init.call(this);
}

EventEmitter.init = function() {

  if (this._events === undefined ||
      this._events === Object.getPrototypeOf(this)._events) {
    this._events = Object.create(null);
    this._eventsCount = 0;
  }

  this._maxListeners = this._maxListeners || undefined;
};
```
同样是实现一个类，但是源码中更注意性能，我们可能认为简单的一个 `this._events = {}`;就可以了，但是通过`jsperf`(一个小彩蛋，有需要的搜以下，查看性能工具) 比较两者的性能，源码中高了很多，我就不具体一一讲解了，附上源码地址，有兴趣的可以去学习
> lib/events源码地址  https://github.com/nodejs/node/blob/master/lib/events.js

源码篇幅过长，给了地址可以对比继续研究，毕竟是公众号文章，不想被说。但是一些疑问还是要讲的，嘿嘿。

![](http://img.xiaogangzai.cn/node_events_03.jpg)
## 阅读源码后一些疑问的解释
### 监听函数的执行顺序是同步 or 异步？
看一段代码：
```javascript
const EventEmitter = require('events');
class MyEmitter extends EventEmitter{};
const myEmitter = new MyEmitter();
myEmitter.on('event', function() {
  console.log('listener1');
});
myEmitter.on('event', async function() {
  console.log('listener2');
  setTimeout(() => {
    console.log('我是异步中的输出');
    resolve(1);
  }, 1000);
});
myEmitter.on('event', function() {
  console.log('listener3');
});
myEmitter.emit('event');
console.log('end');
```
输出结果如下:
```
// 输出结果
listener1
listener2
listener3
end
我是异步中的输出
```
EventEmitter触发事件的时候，各`监听函数的调用`是同步的（注意：监听函数的调用是同步的，'end'的输出在最后），但是并不是说监听函数里不能包含异步的代码，代码中listener2那个事件就加了一个异步的函数，它是最后输出的。

### 事件循环中的事件是什么情况下产生的？什么情况下触发的？
我为什么要把这个单独写成一个小标题来讲，因为发现网上好多文章都是错的，或者不明确，给大家造成了误导。

看这里，某API网站的一段话，具体网站名称在这里就不说了，不想招黑，这段内容没问题，但是对于刚接触事件机制的小伙伴容易混淆

![](http://img.xiaogangzai.cn/node_events_04.jpg)
以`fs.open`为例子，看一下到底什么时候产生了事件，什么时候触发，和EventEmitter有什么关系呢？

![](http://img.xiaogangzai.cn/node_events_05.jpg)


流程的一个说明：本图中详细绘制了从 异步调用开始--->异步调用请求封装--->请求对象传入I/O线程池完成I/O操作--->将完成的I/O结果交给I/O观察者--->从I/O观察者中取出回调函数和结果调用执行。
#### 事件产生
关于事件你看图中第三部分，事件循环那里。Node.js 所有的异步 I/O 操作(net.Server， fs.readStream 等)在`完成后`都会添加一个事件到事件循环的事件队列中。

#### 事件触发
事件的触发，我们只需要关注图中第三部分，事件循环会在事件队列中取出事件处理。`fs.open`产生事件的对象都是 events.EventEmitter 的实例，继承了EventEmitter，从事件循环取出事件的时候，触发这个事件和回调函数。


越写越多，越写越想，总是这样，需要控制一下。

![](http://img.xiaogangzai.cn/node_events_06.jpg)

### 事件类型为error的问题

当我们直接为EventEmitter定义一个error事件，它包含了错误的语义，我们在遇到 异常的时候通常会触发 error 事件。

当 error 被触发时，EventEmitter 规定如果没有响 应的监听器，Node.js 会把它当作异常，退出程序并输出错误信息。

```javascript
var events = require('events'); 
var emitter = new events.EventEmitter(); 
emitter.emit('error'); 
```
运行时会报错

```javascript
node.js:201 
throw e; // process.nextTick error, or 'error' event on first tick 
^ 
Error: Uncaught, unspecified 'error' event. 
at EventEmitter.emit (events.js:50:15) 
at Object.<anonymous> (/home/byvoid/error.js:5:9) 
at Module._compile (module.js:441:26) 
at Object..js (module.js:459:10) 
at Module.load (module.js:348:31) 
at Function._load (module.js:308:12) 
at Array.0 (module.js:479:10) 
at EventEmitter._tickCallback (node.js:192:40) 
```

我们一般要为会触发 error 事件的对象设置监听器，避免遇到错误后整个程序崩溃。
### 如何修改EventEmitter的最大监听数量？
默认情况下针对单一事件的最大listener数量是10，如果超过10个的话listener还是会执行，只是控制台会有警告信息，告警信息里面已经提示了操作建议，可以通过调用emitter.setMaxListeners()来调整最大listener的限制


```javascript
(node:9379) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 event listeners added. Use emitter.setMaxListeners() to increase limit

```
#### 一个打印warn详细内容的小技巧
上面的警告信息的粒度不够，并不能告诉我们是哪里的代码出了问题，可以通过process.on('warning')来获得更具体的信息（emitter、event、eventCount）

```javascript
process.on('warning', (e) => {
  console.log(e);
})


{ MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 event listeners added. Use emitter.setMaxListeners() to increase limit
    at _addListener (events.js:289:19)
    at MyEmitter.prependListener (events.js:313:14)
    at Object.<anonymous> (/Users/xiji/workspace/learn/event-emitter/b.js:34:11)
    at Module._compile (module.js:641:30)
    at Object.Module._extensions..js (module.js:652:10)
    at Module.load (module.js:560:32)
    at tryModuleLoad (module.js:503:12)
    at Function.Module._load (module.js:495:3)
    at Function.Module.runMain (module.js:682:10)
    at startup (bootstrap_node.js:191:16)
  name: 'MaxListenersExceededWarning',
  emitter:
   MyEmitter {
     domain: null,
     _events: { event: [Array] },
     _eventsCount: 1,
     _maxListeners: undefined },
  type: 'event',
  count: 11 }

```
## EventEmitter的应用场景
- 不能try/catch的错误异常抛出可以使用它
- 好多常用模块继承自EventEmitter
比如`fs`模块 `net`模块
- 面试题会考
- 前端开发中也经常用到发布/订阅模式(思想与Events模块相同)

## 发布/订阅模式与观察者模式的一点说明
观察者模式与发布-订阅者模式，在平时你可以认为他们是一个东西，但是在某些场合(比如面试)可能需要稍加注意，看一下二者的区别对比


借用网上的一张图

![](http://img.xiaogangzai.cn/node_events_07.jpg)
从图中可以看出，发布-订阅模式中间包含一个Event Channel
1. 观察者模式 中的观察者和被观察者之间还是存在耦合的，两者必须确切的知道对方的存在才能进行消息的传递。
2. 发布-订阅模式 中的发布者和订阅者不需要知道对方的存在，他们通过消息代理来进行通信，解耦更加彻底。

参考文章:
1. Node.js 官网
2. 朴灵老师的Node.js深入浅出
3. events在github中的源码地址 https://github.com/nodejs/node/blob/master/lib/events.js
4. JavaScript设计模式精讲-SHERlocked93

加入我们一起学习吧！

![](http://img.xiaogangzai.cn/leading.png)