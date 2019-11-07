
## 前言
写完上一篇文章[想学Node.js，stream先有必要搞清楚](https://juejin.im/post/5d25ce36f265da1ba84ab97a)
留下了悬念，`stream`对象数据流转的具体内容是什么？本篇文章将为大家进行深入讲解。


## Buffer探究 
看一段之前使用`stream`操作文件的例子：
```JavaScript
var fileName = path.resolve(__dirname, 'data.txt');
var stream=fs.createReadStream(fileName);
console.log('stream内容',stream);  
stream.on('data',function(chunk){
    console.log(chunk instanceof Buffer)
    console.log(chunk);
})
```
看一下打印结果，发现第一个stream是一个对象 ，截图部分内容。

![](https://user-gold-cdn.xitu.io/2019/7/17/16bfd60a4f3b2069?w=872&h=722&f=jpeg&s=101462)
第二个和第三个打印结果，

![](https://user-gold-cdn.xitu.io/2019/7/17/16bfd601607b160c?w=1372&h=80&f=jpeg&s=34184)
Buffer对象，类似数组，它的元素为16进制的两位数，即0到255的数值。可以看出stream中流动的数据是Buffer类型，二进制数据，接下来开始我们的Buffer探索之旅。

## 什么是二进制
二进制是计算机最底层的数据格式，字符串，数字，视频，音频，程序，网络包等，在最底层都是用二进制来进行存储。这些高级格式和二进制之间，都可以通过固定的编码格式进行相互转换。

例如，C语言中int32类型的十进制整数（无符号），就占用32bit即4byte，十进制的3对应的二进制就是`00000000 00000000 00000000 00000011`。字符串也是同理，可以根据ASCII编码规则或者unicode编码规则（如utf-8）等和二进制进行相互转换。总之，计算机底层存储的数据都是二进制格式，各种高级类型都有对应的编码规则和二进制进行相互转换。

## node中为什么会出现Buffer这个模块

在最初的`javascript`生态中，`javascript`还运行在浏览器端，对于处理Unicode编码的字符串数据很容易，但是对于处理二进制以及非`Unicode`编码的数据无能为力，但是对于`Server`端操作`TCP/HTTP`以及`文件I/O`的处理是必须的。我想就是因此在`Node.js`里面提供了`Buffer`类处理二进制的数据，可以处理各种类型的数据。

Buffer模块的一个说明。
> 在Node.js里面一些重要模块net、http、fs中的数据传输以及处理都有Buffer的身影，因为一些基础的核心模块都要依赖Buffer，所以在node启动的时候，就已经加载了Buffer，我们可以在全局下面直接使用Buffer，无需通过require()。且 Buffer 的大小在创建时确定，无法调整。


## Buffer创建

在 `NodeJS v6.0.0`版本之前，`Buffer`实例是通过 Buffer 构造函数创建的，即使用 new 关键字创建，它根据提供的参数返回不同的 Buffer，但在之后的版本中这种声明方式就被废弃了，替代 new 的创建方式主要有以下几种。

#### 1. Buffer.alloc 和 Buffer.allocUnsafe(创建固定大小的buffer)

用 `Buffer.alloc` 和 `Buffer.allocUnsafe` 创建 Buffer 的传参方式相同，参数为创建 Buffer 的长度，数值类型。

```JavaScript
// Buffer.alloc 和 Buffer.allocUnsafe 创建 Buffer
// Buffer.alloc 创建 Buffer,创建一个大小为6字节的空buffer，经过了初始化
let buf1 = Buffer.alloc(6);

// Buffer.allocUnsafe 创建 Buffer，创建一个大小为6字节的buffer，未经过初始化
let buf2 = Buffer.allocUnsafe(6);

console.log(buf1); // <Buffer 00 00 00 00 00 00>
console.log(buf2); // <Buffer 00 e7 8f a0 00 00>
```

通过代码可以看出，用 `Buffer.alloc` 和 `Buffer.allocUnsafe` 创建` Buffer` 是有区别的，`Buffer.alloc` 创建的 `Buffer` 是被初始化过的，即 `Buffer` 的每一项都用 00 填充，而 `Buffer.allocUnsafe` 创建的 Buffer 并没有经过初始化，在内存中只要有闲置的 Buffer 就直接 “抓过来” 使用。

`Buffer.allocUnsafe` 创建 `Buffer` 使得内存的分配非常快，但已分配的内存段可能包含潜在的敏感数据，有明显性能优势的同时又是不安全的，所以使用需格外 “小心”。

#### 2、Buffer.from(根据内容直接创建Buffer)

> Buffer.from(str,  ) 
支持三种传参方式：

- 第一个参数为字符串，第二个参数为字符编码，如 `ASCII`、`UTF-8`、`Base64` 等等。
- 传入一个数组，数组的每一项会以十六进制存储为 `Buffer` 的每一项。
- 传入一个` Buffer`，会将 `Buffer` 的每一项作为新返回 `Buffer` 的每一项。

**说明:`Buffer`目前支持的编码格式**

- ascii - 仅支持7位ASCII数据。
- utf8 - 多字节编码的Unicode字符
- utf16le - 2或4个字节，小端编码的Unicode字符
- base64 - Base64字符串编码
- binary - 二进制编码。
- hex - 将每个字节编码为两个十六进制字符。

##### 传入字符串和字符编码：
```JavaScript
// 传入字符串和字符编码
let buf = Buffer.from("hello", "utf8");

console.log(buf); // <Buffer 68 65 6c 6c 6f>
```
##### 传入数组：


```JavaScript
// 数组成员为十进制数
let buf = Buffer.from([1, 2, 3]);

console.log(buf); // <Buffer 01 02 03>
```


```JavaScript
// 数组成员为十六进制数
let buf = Buffer.from([0xe4, 0xbd, 0xa0, 0xe5, 0xa5, 0xbd]);

console.log(buf); // <Buffer e4 bd a0 e5 a5 bd>
console.log(buf.toString("utf8")); // 你好
```

在 `NodeJS` 中不支持 `GB2312` 编码，默认支持 `UTF-8`，在 `GB2312` 中，一个汉字占两个字节，而在 `UTF-8` 中，`一个汉字`占三个字节，所以上面 “你好” 的 `Buffer` 为 6 个十六进制数组成。


```JavaScript
// 数组成员为字符串类型的数字
let buf = Buffer.from(["1", "2", "3"]);
console.log(buf); // <Buffer 01 02 03>
```

传入的数组成员可以是任何进制的数值，当成员为字符串的时候，如果值是数字会被自动识别成数值类型，如果值不是数字或成员为是其他非数值类型的数据，该成员会被初始化为 00。

创建的 `Buffer` 可以通过 `toString` 方法直接指定编码进行转换，默认编码为 `UTF-8`。

##### 传入 Buffer：

```JavaScript
// 传入一个 Buffer
let buf1 = Buffer.from("hello", "utf8");

let buf2 = Buffer.from(buf1);

console.log(buf1); // <Buffer 68 65 6c 6c 6f>
console.log(buf2); // <Buffer 68 65 6c 6c 6f>
console.log(buf1 === buf2); // false
console.log(buf1[0] === buf2[0]); // true
buf1[1]=12;
console.log(buf1); // <Buffer 68 0c 6c 6c 6f>
console.log(buf2); // <Buffer 68 65 6c 6c 6f>
```

当传入的参数为一个 `Buffer` 的时候，会创建一个新的 `Buffer` 并复制上面的每一个成员。

`Buffer` 为引用类型，一个 `Buffer` 复制了另一个 Buffer 的成员，当其中一个 Buffer 复制的成员有更改，另一个 Buffer 对应的成员不会跟着改变，说明传入`buffer`创建新的`Buffer`的时候是一个深拷贝的过程。


## Buffer的内存分配机制
**buffer对应于 V8 堆内存之外的一块原始内存**

`Buffer`是一个典型的`javascript`与`C++`结合的模块，与性能有关的用C++来实现，`javascript` 负责衔接和提供接口。`Buffer`所占的内存不是`V8`堆内存，是独立于`V8`堆内存之外的内存，通过`C++`层面实现内存申请（可以说真正的内存是`C++`层面提供的）、`javascript` 分配内存（可以说`JavaScript`层面只是使用它）。`Buffer`在分配内存最终是使用`ArrayBuffer`对象作为载体。简单点而言， 就是`Buffer`模块使用`v8::ArrayBuffer`分配一片内存，通过`TypedArray`中的`v8::Uint8Array`来去写数据。


#### 内存分配的8K机制

- 分配小内存

说道Buffer的内存分配就不得不说`Buffer`的`8KB`的问题，对应`buffer.js`源码里面的处理如下：

```JavaScript
Buffer.poolSize = 8 * 1024;

function allocate(size)
{
    if(size <= 0 )
        return new FastBuffer();
    if(size < Buffer.poolSize >>> 1 )
        if(size > poolSize - poolOffset)
            createPool();
        var b = allocPool.slice(poolOffset,poolOffset + size);
        poolOffset += size;
        alignPool();
        return b
    } else {
        return createUnsafeBuffer(size);
    }
}
```
源码直接看来就是以8KB作为界限，如果写入的数据大于8KB一半的话直接则直接去分配内存，如果小于4KB的话则从当前分配池里面判断是否够空间放下当前存储的数据，如果不够则重新去申请8KB的内存空间，把数据存储到新申请的空间里面，如果足够写入则直接写入数据到内存空间里面，下图为其内存分配策略。

![Buffer内存分配策略图](https://user-gold-cdn.xitu.io/2019/7/16/16bfa9c8e4af644f?w=664&h=446&f=png&s=29461)
看内存分配策略图，如果当前存储了2KB的数据，后面要存储5KB大小数据的时候分配池判断所需内存空间大于4KB，则会去重新申请内存空间来存储5KB数据并且分配池的当前偏移指针也是指向新申请的内存空间，这时候就之前剩余的6KB(8KB-2KB)内存空间就会被搁置。至于为什么会用`8KB`作为`存储单元`分配，为什么大于`8KB`按照大内存分配策略，在下面`Buffer`内存分配机制优点有说明。


- 分配大内存

还是看上面那张内存分配图，如果需要超过`8KB`的`Buffer`对象，将会直接分配一个`SlowBuffer`对象作为基础单元，这个基础单元将会被这个大`Buffer`对象独占。

```JavaScript
// Big buffer,just alloc one
this.parent = new SlowBuffer(this.length);
this.offset = 0;
```
这里的`SlowBUffer`类实在`C++`中定义的，虽然引用buffer模块可以访问到它，但是不推荐直接操作它，而是用`Buffer`替代。这里内部`parent`属性指向的`SlowBuffer`对象来自`Node`自身`C++`中的定义，是`C++`层面的`Buffer`对象，所用内存不在`V8`的堆中

- 内存分配的限制

此外，`Buffer`单次的内存分配也有限制，而这个限制根据不同操作系统而不同，而这个限制可以看到`node_buffer.h`里面

```C
    static const unsigned int kMaxLength =
    sizeof(int32_t) == sizeof(intptr_t) ? 0x3fffffff : 0x7fffffff;
```
对于32位的操作系统单次可最大分配的内存为1G，对于64位或者更高的为2G。

#### buffer内存分配机制优点
`Buffer`真正的内存实在`Node`的`C++`层面提供的，`JavaScript`层面只是使用它。当进行小而频繁的`Buffer`操作时，采用的是`8KB`为一个单元的机制进行预先申请和事后分配，使得`Javascript`到操作系统之间不必有过多的内存申请方面的系统调用。对于大块的`Buffer`而言(大于`8KB`)，则直接使用`C++`层面提供的内存，则无需细腻的分配操作。

## Buffer与stream
#### stream的流动为什么要使用二进制Buffer
根据最初代码的打印结果，`stream`中流动的数据就是`Buffer`类型，也就是`二进制`。

**原因一：**

`node`官方使用二进制作为数据流动肯定是考虑过很多，比如在上一篇  [想学Node.js，stream先有必要搞清楚](https://juejin.im/post/5d25ce36f265da1ba84ab97a)文章已经说过，stream主要的设计目的——是为了优化`IO操作`（`文件IO`和`网络IO`），对应后端无论是`文件IO`还是`网络IO`，其中包含的数据格式都是未知的，有可能是字符串，音频，视频，网络包等等，即使就是字符串，它的编码格式也是未知的，可能`ASC编码`，也可能`utf-8`编码，对于这些未知的情况，还不如直接使用最通用的格式`二进制`.

**原因二：**

`Buffer`对于`http`请求也会带来性能提升。

举一个例子：

```JavaScript
const http = require('http');
const fs = require('fs');
const path = require('path');

const server = http.createServer(function (req, res) {
    const fileName = path.resolve(__dirname, 'buffer-test.txt');
    fs.readFile(fileName, function (err, data) {
        res.end(data)   // 测试1 ：直接返回二进制数据
        // res.end(data.toString())  // 测试2 ：返回字符串数据
    });
});
server.listen(8000);
```

将代码中的`buffer-test`文件大小增加到`50KB`左右，然后使用`ab`工具测试一下性能，你会发现无论是从`吞吐量`（Requests per second）还是连接时间上，返回二进制格式比返回字符串格式效率提高很多。为何字符串格式效率低？—— 因为网络请求的数据本来就是二进制格式传输，虽然代码中写的是 `response` 返回字符串，最终还得再转换为二进制进行传输，多了一步操作，效率当然低了。
#### Buffer在stream数据流转充当的角色

我们可以把整个`流(stream)`和`Buffer`的配合过程看作`公交站`。在一些公交站，`公交车`在没有装满乘客前是不会发车的，或者在特定的时刻才会发车。当然，乘客也可能在不同的时间，人流量大小也会有所不同，有人多的时候，有人少的时候，`乘客`或`公交车站`都无法控制人流量。

不论何时，早到的乘客都必须等待，直到`公交车`接到指令可以发车。当乘客到站，发现`公交车`已经装满，或者已经开走，他就必须等待下一班车次。

总之，这里总会有一个等待的地方，这个`等待的区域`就是`Node.js`中的`Buffer`，`Node.js`不能控制数据什么时候传输过来，传输速度，就好像公交车站无法控制人流量一样。他只能决定什么时候发送数据(公交车发车)。如果时间还不到，那么`Node.js`就会把数据放入`Buffer`等待区域中，一个在RAM中的地址，直到把他们发送出去进行处理。

**注意点：**

`Buffer`虽好也不要瞎用，`Buffer`与`String`两者都可以存储字符串类型的数据，但是，`String`与`Buffer`不同，在内存分配上面，`String`直接使用`v8堆存储`，不用经过`c++`堆外分配内存，并且`Google`也对`String`进行优化，在实际的拼接测速对比中，`String`比`Buffer`快。但是`Buffer`的出现是为了处理二进制以及其他非`Unicode`编码的数据，所以在处理`非utf8`数据的时候需要使用到`Buffer`来处理。


今天就分享这么多，如果对分享的内容感兴趣，可以关注公众号「程序员成长指北」，或者加入技术交流群，大家一起讨论。

Node系列原创文章:

[深入理解Node.js 中的进程与线程
](https://juejin.im/post/5d43017be51d4561f40adcf9)

[想学Node.js，stream先有必要搞清楚
](https://juejin.im/post/5d25ce36f265da1ba84ab97a)

[require时，exports和module.exports的区别你真的懂吗](https://juejin.im/post/5d5639c7e51d453b5c1218b4)

[源码解读一文彻底搞懂Events模块
](https://juejin.im/post/5d69eef7f265da03f12e70a5)

[Node.js 高级进阶之 fs 文件模块学习
](https://juejin.im/post/5d3f1664e51d4561a34618c1)
### 关注我
觉得不错点个Star，欢迎 加群 互相学习。
![](http://img.xiaogangzai.cn/leading.png)

