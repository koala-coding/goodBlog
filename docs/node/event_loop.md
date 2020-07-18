---
title: node事件循环
tags:
   - Node.js
---

**本文涵盖**
- 面试题的引入
- 笔者对事件循环面试题执行顺序的一些疑问
- 通过面试题对微任务、事件循环、定时器等对深入理解
- 结论总计
## 面试题
面试题如下，大家可以先试着写一下输出结果，然后再看我下面的详细讲解，看看会不会有什么出入，如果把整个顺序弄清楚node.js的执行顺序应该就没问题了。

```javascript
async function async1(){
    console.log('async1 start')
    await async2()
    console.log('async1 end')
  }
async function async2(){
    console.log('async2')
}
console.log('script start')
setTimeout(function(){
    console.log('setTimeout0') 
},0)  
setTimeout(function(){
    console.log('setTimeout3') 
},3)  
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('nextTick'));
async1();
new Promise(function(resolve){
    console.log('promise1')
    resolve();
    console.log('promise2')
}).then(function(){
    console.log('promise3')
})
console.log('script end')
```

面试题正确的输出结果

``` JavaScript
script start
async1 start
async2
promise1
promise2
script end
nextTick
async1 end
promise3
setTimeout0
setImmediate
setTimeout3
```
## 提出问题
在理解node.js的异步的时候有一些不懂的地方，使用node.js的开发者一定都知道它是单线程的，异步不阻塞且高并发的一门语言，但是**node.js在实现异步的时候，两个异步任务开启了，是就是谁快就谁先完成这么简单，还是说异步任务最后也会有一个先后执行顺序?对于一个单线程的的异步语言它是怎么实现高并发的呢?**
 
 好接下来我们就带着这两个问题来真正的理解node.js中的异步（微任务与事件循环）。
 

 Node 的异步语法比浏览器更复杂，因为它可以跟内核对话，不得不搞了一个专门的库 libuv 做这件事。这个库负责各种回调函数的执行时间，异步任务最后基于事件循环机制还是要回到主线程，一个个排队执行。


## 详细讲解
#### 1.本轮循环与次轮循环

异步任务可以分成两种。

1. 追加在本轮循环的异步任务
2. 追加在次轮循环的异步任务

所谓”循环”，指的是事件循环（event loop）。这是 JavaScript 引擎处理异步任务的方式，后文会详细解释。这里只要理解，本轮循环一定早于次轮循环执行即可。

Node 规定，process.nextTick和Promise的回调函数，追加在本轮循环，即**同步任务一旦执行完成，就开始执行它们**。而setTimeout、setInterval、setImmediate的回调函数，追加在次轮循环。

#### 2.process.nextTick()

1）**process.nextTick不要因为有next就被好多小伙伴当作次轮循环**。

2）Node 执行完所有同步任务，接下来就会执行process.nextTick的任务队列。

3）开发过程中如果想让异步任务尽可能快地执行，可以使用process.nextTick来完成。

#### 3.微任务（microtack）

根据语言规格，Promise对象的回调函数，会进入异步任务里面的”微任务”（microtask）队列。

微任务队列追加在process.nextTick队列的后面，也属于本轮循环。

根据语言规格，Promise对象的回调函数，会进入异步任务里面的”微任务”（microtask）队列。

微任务队列追加在process.nextTick队列的后面，也属于本轮循环。所以，下面的代码总是先输出3，再输出4。
```javascript
process.nextTick(() => console.log(3));
Promise.resolve().then(() => console.log(4));
```
//输出结果3，4

```javascript
process.nextTick(() => console.log(1));
Promise.resolve().then(() => console.log(2));
process.nextTick(() => console.log(3));
Promise.resolve().then(() => console.log(4));
```
//输出结果 1，3，2，4

注意，只有前一个队列全部清空以后，才会执行下一个队列。两个队列的概念 nextTickQueue 和微队列microTaskQueue，也就是说开启异步任务也分为几种，像promise对象这种，开启之后直接进入微队列中，微队列内的就是那个任务快就那个先执行完，但是针对于队列与队列之间不同的任务，还是会有先后顺序，这个先后顺序是由队列决定的。

#### 4.事件循环的阶段（idle, prepare忽略了这个阶段）

事件循环最阶段最详细的讲解（官网：https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#setimmediate-vs-settimeout）

1. timers阶段

   次阶段包括setTimeout()和setInterval()
2. IO callbacks

    大部分的回调事件，普通的caollback
3. poll阶段

   网络连接，数据获取，读取文件等操作
4. check阶段

   setImmediate()在这里调用回调

5. close阶段
    一些关闭回调，例如socket.on('close', ...)

-  事件循环注意点

1）Node 开始执行脚本时，会先进行事件循环的初始化，但是这时事件循环还没有开始，会先 完成下面的事情。

同步任务
发出异步请求
规划定时器生效的时间
执行process.nextTick()等等

最后，上面这些事情都干完了，事件循环就正式开始了。

2）事件循环同样运行在单线程环境下，高并发也是依靠事件循环，每产生一个事件，就会加入到该阶段对应的队列中，此时事件循环将该队列中的事件取出，准备执行之后的callback。

3）假设事件循环现在进入了某个阶段，即使这期间有其他队列中的事件就绪，也会先将当前队列的全部回调方法执行完毕后，再进入到下一个阶段。
 

#### 5.事件循环中的setTimeOut与setImmediate
由于setTimeout在 timers 阶段执行，而setImmediate在 check 阶段执行。所以，setTimeout会早于setImmediate完成。

```javascript
setTimeout(() => console.log(1));
setImmediate(() => console.log(2));
```

上面代码应该先输出1，再输出2，但是实际执行的时候，结果却是不确定，有时还会先输出2，再输出1。

这是因为setTimeout的第二个参数默认为0。但是实际上，Node 做不到0毫秒，最少也需要1毫秒，根据官方文档，第二个参数的取值范围在1毫秒到2147483647毫秒之间。也就是说，setTimeout(f, 0)等同于setTimeout(f, 1)。

实际执行的时候，进入事件循环以后，有可能到了1毫秒，也可能还没到1毫秒，取决于系统当时的状况。如果没到1毫秒，那么 timers 阶段就会跳过，进入 check 阶段，先执行setImmediate的回调函数。

但是，下面的代码一定是先输出2，再输出1。

```javascript
const fs = require('fs');
fs.readFile('test.js', () => {
 setTimeout(() => console.log(1));
 setImmediate(() => console.log(2));
});
```

上面代码会先进入 I/O callbacks 阶段，然后是 check 阶段，最后才是 timers 阶段。因此，setImmediate才会早于setTimeout执行。
#### 6.同步任务中async以及promise的一些误解

- 问题1:

在那道面试题中，在同步任务的过程中，不知道大家有没有疑问，为什么不是执行完async2输出后执行async1 end输出，而是接着执行promise1？

解答:引用阮一峰老师书中一句话：“ async 函数返回一个 Promise 对象，当函数执行的时候，一旦遇到 await 就会先返回，等到触发的异步操作完成，再接着执行函数体内后面的语句。”
简单的说，先去执行后面的同步任务代码，执行完成后，也就是表达式中的 Promise 解析完成后继续执行 async 函数并返回解决结果。（其实还是本轮循环promise的问题，最后的resolve属于异步，位于本轮循环的末尾。）

- 问题2:

console.log('promise2')为什么也是在resolve之前执行？

解答：注：此内容来源与阮一峰老师的ES6书籍，调用resolve或者reject并不会终结promise的参数函数的执行。因为立即resolved的Promise是**本轮循环**的末尾执行，同时总是**晚于本轮循环的同步任务**。正规的写法调用resolve或者reject以后，Promise的使命就完成了，后继操作应该放在then方法后面。所以最好在它的前面加上return语句，这样就不会出现意外

```javascript
new Promise((resolve,reject) => {
    return resolve(1);
    //后面的语句不会执行
    console.log(2);
}
```

- 问题3:

promise3和script end的执行顺序是否有疑问？

解答：因为立即resolved的Promise是**本轮循环**的末尾执行，同时总是**晚于本轮循环的同步任务**。 Promise 是一个立即执行函数，但是他的成功（或失败：reject）的回调函数 resolve 却是一个异步执行的回调。当执行到 resolve() 时，这个任务会被放入到回调队列中，等待调用栈有空闲时事件循环再来取走它。本轮循环中最后执行的。

## 整体结论
 
![](http://img.xiaogangzai.cn/16b12df18cc5e443.jpg)
 
 
 顺序的整体总结就是:
 同步任务-> 本轮循环->次轮循环
 
## 附件:参考资料
node.js官网：

- 事件循环：https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/#setimmediate-vs-settimeout
- Timers：https://nodejs.org/dist/latest-v10.x/docs/api/timers.html

觉得本文对你有帮助？请分享给更多人

::: warning 作者介绍
大家好，我是koala，在做一个一个Node.js高级进阶路线，今天就分享这么多，如果对分享的内容感兴趣，可以关注公众号`「程序员成长指北」`，或者加入技术交流群，大家一起讨论。
:::