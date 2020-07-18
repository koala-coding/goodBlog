---
title: Node.js 内存溢出时如何处理？
date: 2019-08-19
tags:
   - Node.js
---

Node.js 做密集型运算，或者所操作的数组、对象本身较大时，容易出现内存溢出的问题，这是由于 Node.js 的运行环境依赖 V8 引擎导致的。如果经常有较大数据量运算等操作，需要对 Node.js 运行环境限制有充分的了解。 
## 本文涵盖
1. 内存溢出问题
2. 为什么会内存溢出
    - 2.1 V8内存分配机制
    - 2.2 内存溢出的原因
3. 如何解决内存溢出问题

> 作者简介：koala，专注完整的 Node.js 技术栈分享，从 JavaScript 到 Node.js,再到后端数据库，祝您成为优秀的高级 Node.js 工程师。【程序员成长指北】作者，Github 博客开源项目 https://github.com/koala-coding/goodBlog

## 1.内存溢出问题
下面是我们在Node.js应用中经常遇到的两类内存溢出问题：

## 密集型运算

示例1：当我们需要批量处理一些数据（如：更新用户某项信息）时，我们可能需要一个较大的for或while循环来完成所有的数据的更新，如：
```javascript
for (var i = 0; i < 10000000; i++) {
    ((i) => {
        var site = {};
        site.name = 'koala';
        site.domain = '程序员成长指北';
        // 这里是一个保存或更新等操作

        setTimeout(()=>{
            console.log(i, site);
        }, 0)
    })(i)
}
```
## 操作的数据量较大
示例2：对象需要频繁的创建/销毁，或操作对象本身较大，如：

```javascript
var sites = [];
for (var x=0;x<5000;x++){
    var site=[];
    for (var y=0;y<5000;y++){
        site = [y, 'koala', '程序员成长指北'];
        sites.push(site);
    }
}
```


上面两类操作都会出现类似以下错误：
```javascript
<--- Last few GCs --->
……
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - process out of memory
Abort trap: 6
```

## 2. 为什么会内存溢出
### 2.1 V8内存分配机制
我们都知道，V8是 Google 在 Chrome 浏览器中使用的 JavaScript 引擎。而在浏览器环境中，运算一般不需要多大内存。
V8 对每个进程分配的运行内存，在32位系统中约为700MB，而在64位系统中约为1.4GB。


### 2.2 内存溢出的原因
Node.js 程序之所以会出内存溢出的情况，可以分为三方面的原因：
- 1. V8本身分配的内存较小
- 2. JavaScript语言本身限制
- 3. 程序员使用不当。

在示例1中，每次运算所需的内存量并不大，但由于for循环，造成V8内存不能及时释放。随着程序运行时候的增加，内存占用量会越来越大，并最终导致内存的溢出。

在示例2中，可能所创建对象本身并没有超过内存限制。但是除对象本身外：创建对象、对象引用、Node.js程序本身等都需要内存空间，这样就很容易导致内存的溢出。


## 3. 解决内存溢出问题
在Node.js应用开发过程中，了解V8内存分配和JavaScript语言限制是Node程序的基本素质。我们应该在应用中权衡利弊，综合考虑内存与程序的运行效率。以下几点防止内存溢出的建议：
### 3.1. 使用 async/await防止事件堆积,变为同步操作 

await将代码执行顺序变为了同步。这样可以使 V8 获得内存回收的机会，有效解决过多事件堆积造成的内存溢出。
我们可以使用await方法处理：

```javascript
async function dbFuc() {
for (let i = 0; i < 10000000; i++) {
    var site = {};
    site.name = 'koala';
    site.domain = '程序员成长指北';
    // 这里是一个保存或更新等操作

    await  console.log(i, site);

    }
}
dbFuc();
```

每次循环V8都会回收内存一次，因此内存不会再溢出。但这样做必然会造成运行效率的降低，而应该在速度在安全之间平衡，控制好循环的安全次数。
说明:实际开发中，上面这种虽然解决了内存溢出，但是仍然会造成进程阻塞，可以开启一个进程/线程来解决阻塞问题(具体可以看我的这篇文章[《深入理解Node.js 进程与线程(8000长文彻底搞懂)》](https://juejin.im/post/5d43017be51d4561f40adcf9))

### 3.2. 增加V8内存空间

Node.js提供了一个程序运行参数`--max-old-space-size`，可以通过该参数指定V8所占用的内存空间，这样可以在一定程度上避免程序内存的溢出。
如，我们可以在运行示例2程序时指定使用4G的内存：
`node --max-old-space-size=4096 app`


### 3.3. 使用非V8内存

Node.js程序所使用的内存分为两类：
- V8内存：数组、字符串等JavaScript内置对象，运行时使用“V8内存”
- 系统内存：Buffer 是 Node.js 的一个扩展对象，使用底层的系统内存，不占用V8内存空间。与 buffer 相关的文件系统 fs 和stream 流操作，都不会占用 V8 内存。
> (注: fs 和 stream 这两个模块我在 Node 进阶系列文章中已经详细介绍了, 这里就不赘述)

在程序允许的情况下，应该将数据保存在Buffer中，而不是转换成字符串等JS对象，这样可以避免V8内存的过多占用。（buffer可以看一下这篇文章[《Node进阶-探究不在V8堆内存中存储的Buffer对象》](https://juejin.im/post/5d2db6d9f265da1bcc1975d7)）

## Node系列原创文章

[深入理解Node.js 中的进程与线程
](https://juejin.im/post/5d43017be51d4561f40adcf9)

[想学Node.js，stream先有必要搞清楚
](https://juejin.im/post/5d25ce36f265da1ba84ab97a)

[require时，exports和module.exports的区别你真的懂吗](https://juejin.im/post/5d5639c7e51d453b5c1218b4)

[源码解读一文彻底搞懂Events模块
](https://juejin.im/post/5d69eef7f265da03f12e70a5)

[Node.js 高级进阶之 fs 文件模块学习
](https://juejin.im/post/5d3f1664e51d4561a34618c1)
## 关注我
- 欢迎加我微信(coder_qi)，拉你进技术群，长期交流学习...
- 欢迎关注「程序员成长指北」,一个用心帮助你成长的公众号...
![](http://img.xiaogangzai.cn/leading.png)