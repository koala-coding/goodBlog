---
title: 消息队列助- Node.js实践篇
date: 2019-11-22
tags:
   - Node.js
---

## 为什么写这篇文章
- 现在的面试要求越来越高了，打开看了看几个BOSS招聘 Node.js 全栈开发的，其中都有一条“了解消息队列，并在项目中应用过”，呜呜呜
- 后端开发者应该都知道消息队列，但是一些前端开发者可能知道的并不多，但是你们可能好奇`抢票,商品秒杀`等功能是如何实现的，其实没有多么高大上，看了消息队列就知道了。

## 文章导图(你能学到)


![](http://img.xiaogangzai.cn/queue_01.jpg)

作者简介：koala，专注完整的 Node.js 技术栈分享，从 JavaScript 到 Node.js,再到后端数据库，祝您成为优秀的高级 Node.js 工程师。【程序员成长指北】作者，Github 博客开源项目 https://github.com/koala-coding/goodBlog


##  什么是消息队列

> “消息队列”是在消息的传输过程中保存消息的容器。

**个人理解**:我把它分成两个词`消息`和`队列`。当一大批客户端同时产生大量的网络请求（`消息`）时候，服务器的承受能力肯定是有一个限制的。这时候要是有个容器，先让这些消息排队就好了，还好有个叫`队列`的数据结构，通过`有队列属性的容器`排队(先进先出)，把消息再传到我们的服务器，压力减小了好多，这个很棒的`容器`就是消息队列

这段理解中还包含这个两个概念: 客户端->`生产者`    服务器->`消费者`
当有`消息队列`出现，`生产者`和`消费者`是必不可少的两个概念，上面的理解是`多个生产者`对应`一个消费者`，当然现实开发中还有`许多消费者`的情况哦。接下来的文章也会多次提到`生产-消费模型`。

### 消息队列优势
- **应用解耦**

    消息队列可以使消费者和生产者直接互不干涉，互不影响，只需要把消息发送到队列即可，而且可独立的扩展或修改两边的处理过程，只要能确保它们遵守同样的接口约定，可以生产者用Node.js实现，消费者用phython实现。
- **灵活性和峰值处理能力**

    当客户端访问量突然剧增，对服务器的访问已经超过服务所能处理的最大峰值，甚至导致服务器超时负载崩溃，使用消息队列可以解决这个问题，可以通过`控制消费者的处理速度`和`生产者可进入消息队列的数量`等来避免峰值问题

- **排序保证**

    消息队列可以控制数据处理的顺序，因为消息队列本身使用的是队列这个数据结构，`FIFO`(先进选出)，在一些场景数据处理的顺序很重要，比如商品下单顺序等。

- **异步通信**

    消息队列中的有些消息，并不需要立即处理，消息队列提供了异步处理机制，可以把消息放在队列中并不立即处理，需要的时候处理，或者异步慢慢处理，一些不重要的发送短信和邮箱功能可以使用。
- **可扩展性**

    前面提到了消息队列可以做到`解耦`，如果我们想增强消息入队和出队的处理频率，很简单，并不需要改变代码中任何内容，可以直接对消息队列修改一些配置即可，比如我们想限制每次发送给消费者的消息条数等。

> **有优势定有它现实的应用场景，文章后面会针对优势讲它们对应的应用场景。**

### 消息队列的类型介绍

#### 介绍几款目前市场上主流的消息队列(课外知识，可忽略)
- **Kafka**：是由 Apache 软件基金会开发的一个开源流处理平台，由 Scala 和 Java 编写，是一种高吞吐量的分布式发布订阅消息系统，支持单机每秒百万并发。另外，Kafka 的定位主要在日志等方面， 因为Kafka 设计的初衷就是`处理日志`的，可以看做是一个`日志（消息）系统`一个重要组件，针对性很强。0.8 版本开始支持复制，不支持事物，因此对消息的重复、丢失、错误没有严格的要求。
- **RocketMQ**：阿里开源的消息中间件，是一款低延迟、高可靠、可伸缩、易于使用的消息中间件，思路起源于 Kafka。最大的问题商业版收费，有些功能不开放。
- **RabbitMQ**：由 Erlang（有着和原生 Socket 一样低的延迟）语言开发基于 AMQP 协议的开源消息队列系统。能保证消息的可靠性、稳定性、安全性。`高并发`
的特性，毋庸置疑，RabbitMQ 最高，原因是它的实现语言是天生具备高并发高可用的erlang 语言，天生的`分布式`优势。

> **说明**：
> **本文主要以RabbitMQ讲解，较为常见。**
> **个人认为这几种消息队列中间件能实现的功能，通过 redis 也都能实现，思想。**

## 初识消息队列(消息队列在node.js中的简单应用)
### Rabbitmq基本安装
#### Mac版安装
直接通过 HomeBrew 安装，执行以下命令

```
brew install rabbitmq
```
#### 启动 rabbitmq

```
进入安装目录
$ /usr/local/Cellar/rabbitmq/3.7.8
启动
$ sbin/rabbitmq-server
```

浏览器输入 http://localhost:15672/#/ 默认用户名密码 guest

#### 安装后的基本示意图

![](http://img.xiaogangzai.cn/queue_02.jpg)

可视化界面可模块功能介绍:
*************************

> 其他系统安装请自行网上搜索

#### 几个端口区别说明
5672：通信默认端口号
15672：管理控制台默认端口号
25672：集群通信端口号
注意: 阿里云 ECS 服务器如果出现 RabbitMQ 安装成功，外网不能访问是因为安全组的问题没有开放端口 解决方案

#### Rabbitmq安装后的基本命令
以下列举一些在终端常用的操作命令

- whereis rabbitmq：查看 rabbitmq 安装位置
- rabbitmqctl start_app：启动应用
- whereis erlang：查看erlang安装位置
- rabbitmqctl start_app：启动应用
- rabbitmqctl stop_app：关闭应用
- rabbitmqctl status：节点状态
- rabbitmqctl add_user username password：添加用户
- rabbitmqctl list_users：列出所有用户
- rabbitmqctl delete_user username：删除用户
- rabbitmqctl add_vhost vhostpath：创建虚拟主机
- rabbitmqctl list_vhosts：列出所有虚拟主机
- rabbitmqctl list_queues：查看所有队列
- rabbitmqctl -p vhostpath purge_queue blue：清除队列里消息

> **注意**:以上终端所有命令,需要进入到rabbitmqctl的sbin目录下执行rabbitmqctl命令才有用，否则会报错：
![](http://img.xiaogangzai.cn/queue_03.jpg)

### Node.js实现一个简单的 HelloWorld 消息队列
画一张基本的图，HelloWorld 消息队列的图片，把下面几个概念都画进去。

*************
看这段代码前先说几个概念
- 生产者 ：生产消息的
- 消费者 ：接收消息的
- 通道 channel：建立连接后，会获取一个 channel 通道
- exchange ：交换机，消息需要先发送到 exchange 交换机，也可以说是第一步存储消息的地方(交换机会有很多类型，后面会详细说)。
- 消息队列 : 到达消费者前一刻存储消息的地方,exchange 交换机会把消息传递到此
- ack回执：收到消息后确认消息已经消费的应答
*************

#### amqplib模块
推荐一个 npm 模块`amqplib`。

Github: https://github.com/squaremo/amqp.node

```
$ npm install amqplib
```
#### 生产者代码 product.js

```javascript
const amqp =require('amqplib');

async function  product(params) {
    // 1.创建链接对象
    const connect =await amqp.connect('amqp://localhost:5672');
     // 1. 创建链接对象
     const connection = await amqp.connect('amqp://localhost:5672');

     // 2. 获取通道
     const channel = await connection.createChannel();
 
     // 3. 声明参数
     const routingKey = 'helloKoalaQueue';
     const msg = 'hello koala';
 
     for (let i=0; i<10000; i++) {
         // 4. 发送消息
         await channel.publish('', routingKey, Buffer.from(`${msg} 第${i}条消息`));
     }
 
     // 5. 关闭通道
     await channel.close();
     // 6. 关闭连接
     await connect.close();
}
product();
```
#### 生产者代码解释与运行结果
```javascript
执行 node product.js
```
代码注释中已经把基本的流程讲解了，但是我刚开始看的时候还有疑问，我想很多小伙伴也会有疑问，说明下:
- **疑问1**

    前面提到过交换机这个名词，`生产者`发消息的时候必须要指定一个 exchange，若不指定 exchange（为空）会默认指向 AMQP default 交换机，AMQP default 路由规则是根据 routingKey 和 mq 上有没有相同名字的队列进行匹配路由。上面这段代码就是默认指定的交换机。不同类型交换机详细讲解请往下看。
- **疑问2**

    生产者发送消息后，消息是发送到交换机exchange，但是这时候会创建队列吗？
    
    答案：代码中我们声明的是路由是routingKey，但是它并没有创建helloKoalaQueue 消息队列，消息只会发送到交exchange交换机。
    运行代码后看队列截图可以证明这一点:
    
- **说明1**

    生产者发送消息后，注意关闭通道和连接，只要消息发送成功后，连接就可以关闭了，消费者用任何语言去获取消息都可以，这也证明了消息队列优秀`解耦`的特性

- **说明2**

    可以多次执行`node product.js`生产者代码,消息会堆积到`交换机exchange`中，并不会覆盖，如果已执行过消费者并且确认了对应的`消息队列`，消息会从`exchange交换机`发送到`消息队列`，并存入到`消息队列`，等待`消费者`消费
    
    ![](http://img.xiaogangzai.cn/queue_04.jpg)
    
#### 消费者代码 consumer.js

```javascript
// 构建消费者
const amqp = require('amqplib');

async function consumer() {
    // 1. 创建链接对象
    const connection = await amqp.connect('amqp://localhost:5672');

    // 2. 获取通道
    const channel = await connection.createChannel();

    // 3. 声明参数
    const queueName = 'helloKoalaQueue';
  
    // 4. 声明队列，交换机默认为 AMQP default
    await channel.assertQueue(queueName);

    // 5. 消费
    await channel.consume(queueName, msg => {
        console.log('Consumer：', msg.content.toString());
        channel.ack(msg);
    });
}
consumer();
```
#### 生产者代码解释与运行结果
```javascript
执行 node consumer.js
```
- **运行后的执行结果**

    ![](http://img.xiaogangzai.cn/queue_05.jpg)
- **说明1**

    这时候我改变代码中的队列名称为`helloKoalaQueueHaHa`,这时候去看Rabbitmq可视化界面中，队列模块，创建了这个队列

    ![](http://img.xiaogangzai.cn/queue_06.jpg)
    
    看到这里再次证明了消息队列优秀的`解耦特性`，`消费者和生产者模型`之间没有任何联系，再次创建这个`helloKoalaQueueHaHa`路由名称的生产者，`消费者`也会正常消费，并且会打印消息，大家可以实际操作试一下。

- **说明2**

    这时候我改变代码中的队列名称为`helloKoalaQueueHaHa`,这时候去看Rabbitmq可视化界面中，队列模块，创建了这个队列

    ![](http://img.xiaogangzai.cn/queue_06.jpg)
    
    看到这里又再次证明了消息队列优秀的`解耦特性`，`消费者和生产者模型`之间没有任何联系，再次创建这个`helloKoalaQueueHaHa`路由名称的生产者，`消费者`也会正常消费，并且会打印消息，大家可以实际操作试一下。
### 如何释放掉消息队列
#### 可视化界面中直接删除掉消息队列
1. 访问http://{rabbitmq安装IP}:15672，登录。
2. 点击queues，这里可以看到你创建的所有的Queue，
3. 选中某一个Queue，然后会进入一个列表界面，下方有个Delete按钮，确认 Queue删除队列/Purge Message清除消息即可。

**弊端：**
这样只能一个队列一个队列的删除，如果队列中的消息过多就会特别慢。
#### 通过代码实现消息队列释放(删除)

## 消息队列交换机讲解
先记住一句话
> **生产者发消息的时候必须指定一个 exchange，否则消息无法直接到达消息队列，Exchange将消息路由到一个或多个Queue中（或者丢弃）**

然后开始本章节交换机的讲解

若不指定 exchange（为空）会默认指向 AMQP default 交换机，AMQP default 路由规则是根据 routingKey 和 mq 上有没有相同名字的队列进行匹配路由。

### 交换机的种类
常用的四种类型
- **fanout**

- **direct**

- **topic**

- **headers**



不管是哪一种类型的交换机，都有一个绑定binding的操作，只不过根据不同的交换机类型有不同的路由绑定策略。不同类型做的下图红色框框中的事。

![](http://img.xiaogangzai.cn/queue_07.jpg)
#### fanout（中文翻译 广播）
fanout类型的Exchange路由规则非常简单，它会把所有发送到该Exchange的消息路由到所有与它绑定的Queue中，不需要设置路由键。


![](http://img.xiaogangzai.cn/queue_08.jpg)
上图中，上图中，生产者（Producter）发送到Exchange（X）的所有消息都会路由到图中的两个Queue，并最终被两个消费者（consumer1与consumer2）消费。

> 说明:所有消息都会路由到两个Queue中，是两个消费者都可以收到全部的完全相同的消息吗? 答案是的，两个消费者收到的队列消息正常应该是完全相同的。这种类型常用于广播类型的需求，或者也可以消费者1记录日志 ，消费者2打印日志

**对应代码实现**：

生产者:
```javascript
const amqp = require('amqplib');

async function producer() {
    // 创建链接对象
    const connection = await amqp.connect('amqp://localhost:5672');

    // 获取通道
    const channel = await connection.createChannel();

    // 声明参数
    const exchangeName = 'fanout_koala_exchange';
    const routingKey = '';
    const msg = 'hello koala';

    // 交换机
    await channel.assertExchange(exchangeName, 'fanout', {
        durable: true,
    });

    // 发送消息
    await channel.publish(exchangeName, routingKey, Buffer.from(msg));

    // 关闭链接
    await channel.close();
    await connection.close();
}
producer();
```
消费者：
```javascript
const amqp = require('amqplib');

async function consumer() {
    // 创建链接对象
    const connection = await amqp.connect('amqp://localhost:5672');

    // 获取通道
    const channel = await connection.createChannel();

    // 声明参数
    const exchangeName = 'fanout_koala_exchange';
    const queueName = 'fanout_kaola_queue';
    const routingKey = '';

    // 声明一个交换机
    await channel.assertExchange(exchangeName, 'fanout', { durable: true });

    // 声明一个队列
    await channel.assertQueue(queueName);

    // 绑定关系（队列、交换机、路由键）
    await channel.bindQueue(queueName, exchangeName, routingKey);

    // 消费
    await channel.consume(queueName, msg => {
        console.log('Consumer：', msg.content.toString());
        channel.ack(msg);
    });

    console.log('消费端启动成功！');
}
consumer();
```
注意：其他类型代码已经放到 github，地址：**https://github.com/koala-coding/simple_rabbitmq**  欢迎 **star** 交流。

#### direct 

direct 把消息路由到那些 binding key与 routing key 完全匹配的 Queue中。

![](http://img.xiaogangzai.cn/queue_09.jpg)

以上图的配置为例，我们以 routingKey=”error” 发送消息到Exchange，则消息会路由到 amq1 和 amq2；如果我们以 routingKey=”info” 或 routingKey=”warning” 来发送消息，则消息只会路由到 Queue2。如果我们以其他 routingKey 发送消息，则消息不会路由到这两个 Queue 中。
#### topic
生产者指定 RoutingKey 消息根据消费端指定的队列通过模糊匹配的方式进行相应转发，两种通配符模式：
#：可匹配一个或多个关键字
*：只能匹配一个关键字

![](http://img.xiaogangzai.cn/queue_10.jpg)
#### headers
header exchange(头交换机)和主题交换机有点相似，但是不同于主题交换机的路由是基于路由键，头交换机的路由值基于消息的 header 数据。
主题交换机路由键只有是字符串,而头交换机可以是整型和哈希值
header Exchange 类型用的比较少，可以自行 google 了解。

## 消息队列的思考与深入探索
### 消息队列实现rpc
（本小段内容来源网上，参考文章说明）

![](http://img.xiaogangzai.cn/queue_11.jpg)
RPC 远程调用服务端的方法，使用 MQ 可以实现 RPC 的异步调用，基于 Direct 交换机实现

1. 客户端即是生产者又是消费者，向 RPC 请求队列发送 RPC 调用消息，同时监听 RPC 响应队列
2. 服务端监听RPC请求队列，收到消息后执行服务端的方法
3. 服务端将方法执行后的结果发送到RPC响应队列

（注意，这里只是提一下 RPC 这个知识，因为单单一个RPC一篇文章都不一定说说完，有兴趣的可以用队列尝试一下RPC）

### 是否有**消息持久化**的必要？
`消息队列`是存在内存中的，如果出现问题挂掉,消息队列中的消息会丢失。所以对于一些需求非常有持久化的必要！RabbitMQ 可以开启持久化。不同开发语言都可以设置持久化参数。

这里以Node.js为例子，其他语言可以自行搜索
```javascript
    await channel.assertExchange(exchangeName, 'direct', { durable: true });
    // 注意其中的{ durable: true }，这事对交换机持久化，还有其他的几种持久化方式
```
同时推荐一篇不错的写持久化的文章:
https://juejin.im/post/5d6f6b0ae51d45621512add0

### 消费者完成后是否有**消息应答**的必要？

消息应答简单的解释就是`消费者`完成了消费后，通知一下消息队列。

我觉得这个配置是有必要打开的，消费者完成消息队列中的任务，消费者可能中途失败或者挂掉，一旦 RabbitMQ 发送一个消息给消费者然后便迅速将该消息从`消息队列内存`中移除,这种情况下，消费者对应工作进程失败或者挂掉后，那该进程正在处理的消息也将丢失。而且，也将丢失所有发送给该进程的未被处理的消息。


为了确保消息永不丢失，RabbitMQ 支持消息应答机制。当消息被接受，处理之后一条应答便会从消费者回传至发送方，然后RabbitMQ将其删除。

如果某个消费者挂掉（信道、链接关闭或者 tcp 链接丢失）且没有发送 ack 应答，RabbitMQ 会认为该消息没有被处理完全然后会将其重新放置到队列中。通过这种方式你就可以确保消息永不丢失，甚至某个工作进程偶然挂掉的情况。

默认情况下消息应答是关闭的。是时候使用 false（auto-ack配置项）参数将其开启了

这里以 Node.js 为例子，其他语言可以自行搜索
```javascript
// 消费者消费时候的代码
await channel.consume(queueName, msg => {
    console.log('koala：', msg.content.toString());
    //... 这里可以放业务逻辑处理的代码，消费者完成后发送回执应答
    channel.ack(msg);// 消息应答
}, { noAck: false });
```

### 如何实现公平调度？

可以将`prefetch count`项的值配置为1，这将会指示 RabbitMQ 在同一时间不要发送超过一条消息给每个消费者。换句话说，直到消息被处理和应答之前都不会发送给该消费者任何消息。取而代之的是，它将会发送消息至下一个比较闲的消费者或工作进程。

这里以 Node.js 为例子，amqplib 库对于限流实现提供的接口方法 prefetch。

**prefetch 参数说明**：

- count：每次推送给消费端 N 条消息数目，如果这 N 条消息没有被ack，生产端将不会再次推送直到这 N 条消息被消费。
- global：在哪个级别上做限制，ture 为 channel 上做限制，false 为消费端上做限制，默认为 false。

```javascript
// 创建消费者的时候 限流参数设置
await channel.prefetch(1, false);
```

### 如何实现一个交换机给多个消费者依次发送消息，选择那种交换机？
如果一个生产者，两个消费者，发放消息，我想要的队列先给消费者1发，发完消费者1发消费者2，这样有顺序的交互发送，应该现在哪一种交换机呢？注意是交互，看完之后想一下？还有消费者完成后有没有手动回调消息队列完成的必要？消息持久化有必要没，持久化有什么好处？


(看完消息队列的消息传递，你会有疑问管道中的消息(生产者)是怎么被消费者消费的  放入队列，然后从队列被取出)


## 消息队列应用场景

1. **双十一商品秒杀/抢票功能实现**

    我们在双11的时候，当我们凌晨大量的秒杀和抢购商品，然后去结算的时候，就会发现，界面会提醒我们，让我们稍等，以及一些友好的图片文字提醒。而不是像前几年的时代，动不动就页面卡死，报错等来呈现给用户。
    
    **用一张图来解释消息队列在秒杀抢票等场景的使用：**
    （说明：往下看之前，如果你做过电商类秒杀，可以想想你是怎么实现的，我们可以一起讨论哦。这里只是想说下消息队列的作用，并不是最终优化的结果，比如用redis控制总缓存等）
    
    ![](http://img.xiaogangzai.cn/queue_12.jpg)
    这里在生成订单时候，不需要直接操作数据库 IO ，预扣库存。先扣除了库存，保证不超卖，然后异步生成用户订单，这里用到一次即时`消费队列`，这样响应给用户的速度就会快很多；而且还要保证不少卖，用户拿到了订单，不支付怎么办？我们都知道现在订单都有有效期，再使用一个`消息队列`，用于判断订单支付超时，比如说用户五分钟内不支付，订单就失效了，订单一旦失效，就会加入新的库存。这也是现在很多网上零售企业保证商品不少卖采用的方案。订单量比较少的情况下，生成订单非常快，用户几乎不用排队。
    
2. **积分兑换(积分可用于多平台)**

    积分兑换模块，有一个公司多个部门都要用到这个模块，这时候就可以通过消息队列解耦这个特性来实现。
    各部门系统做各部门的事，但是他们都可以用这个积分系统进行商品的兑换等。其他模块与积分模块完全解耦。

3. **发送邮件，用户大数据分析等 同步变异步功能实现**

    这个功能要说的比较多，从一个平台的用户注册开始。
    - 用户注册
    -  用户注册选择几个兴趣标签，这时候需要根据用户的属性，用户分析，计算出推荐内容
    -  注册后可能需要发送邮件给用户
    -  发送给用户一个包含操作指南的系统通知
    -  等等
    
    **正常情况注册，不出现高并发**：
    
    对于用户来说，他就是想注册用一下这个软件，只要服务端将他的账户信息存到数据库中他便可以登录上去做他想做的事情了。用户并不care这些事，服务端就可以把其他的操作放入对应的`消息队列`中然后马上返回用户结果，由消息队列`异步`的进行这些操作。
    
    **假如有大量的用户注册，发生了高并发**：
    
    邮件接口承受不住，或是分析信息时的大量计算使 cpu 满载，这将会出现虽然用户数据记录很快的添加到数据库中了，但是却卡在发邮件或分析信息时的情况，导致请求的响应时间大幅增长，甚至出现超时，这就有点不划算了。面对这种情况一般也是将这些操作放入消息队列（生产者消费者模型），消息队列慢慢的进行处理，同时可以很快的完成注册请求，不会影响用户使用其他功能。

4. **基于RabbitMQ的Node.js与Phython或其他语言实现通信**

    这里也是利用了 RabbitMQ 的解耦特性，不仅仅可以与 Phython，还可以与其他很多语言通信，就不具体说了。

## 总结
**亲，别只看，你试试呀**！直接开启服务，装个 RabbitMQ，挺有意思的，就算一个 HelloWorld 也能尝试出很多内容。而且本文说的很多内容都可以用 redis 来实现，也可以去看下我的 redis 文章。顺便说一句设计模式和数据结构是两个好东西，越来越能感觉到。

## 参考文章
https://www.cnblogs.com/baidawei/p/9172433.html

https://www.sojson.com/blog/48.html

https://www.zhihu.com/question/34243607/answer/58314162

https://bbs.csdn.net/topics/392169691?page=1

http://www.imooc.com/article/293742

https://mp.weixin.qq.com/s/wTkwJXlNr5CaI7uRntJ42A

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
### 关注我
- 欢迎加我微信(coder_qi)，拉你进技术群，长期交流学习...
- 欢迎关注「程序员成长指北」,一个用心帮助你成长的公众号...
![](http://img.xiaogangzai.cn/leading.png)