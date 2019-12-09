## 为什么出现Promise
在javascript开发过程中，代码是单线程执行的，同步操作，彼此之间不会等待，这可以说是它的优势，但是也有它的弊端，如一些网络操作，浏览器事件，文件等操作等，都必须异步执行，针对这些情况，起初的操作都是使用回调函数实现。

实现方式如下（伪代码）：

```
function One(callback) {
    if (success) {
        callback(err, result);
    } else {
        callback(err, null);
    }
}

One(function (err, result) {
    //执行完One函数内的内容，成功的结果回调回来向下执行
})
```

上述代码只是一层级回调，如果代码复杂后，会出现多层级的回调，代码可读性也会很差，那有没有一种方式，不用考虑里面的内容，直接根据结果成功还是失败执行下面的代码呢？有的，Promise（承诺），在ES6中对Promise进行了统一的规范。

## Promise的含义
- 书上这么说：

  Promise 是异步编程的一种解决方案，比传统的解决方案–回调函数和事件－－更合理和更强大。它由社区最早提出和实现，ES6将其写进了语言标准，统一了语法，原生提供了Promise

  所谓Promise ，简单说就是一个容器，里面保存着某个未来才会结束的事件 (通常是一个异步操作）的结果。从语法上说，Promise是一个对象，从它可以获取异步操作的消息。
  Promise 对象的状态不受外界影响
  
- Promise/a+ 官方网站给出的定义
  A promise represents the eventual result of an asyncchronous operation

  翻译:表示一个异步操作的最终结果。

- 我的理解：

  Promise是**回调函数**可以**规范**的**链式**调用
  
## Promise原理与讲解
##### 原理
1.  Promise的三种状态
    - pending：进行中
    - fulfilled :执行成功
    - rejected ：执行失败

注意Promise在某一时刻只能处于一种状态

2. Promise的状态改变
    - pending------》fulfilled（resolved）
    - pending------》rejected

Promise的状态改变，状态只能由pending转换为fulfilled（resolved）或者rejected，一旦状态改变完成后将无法改变（不可逆性）

##### 用代码讲原理
1. 创建一个Promise

创建Promise需要用到Promise的构造函数来实现，代码如下：

```
var promise=new Promise(function(resolve,reject){
   // ...some async code 
   
   if(/* 一些异步操作成功*/)
   {
       resolve(value);
   }else
   {
       reject(error);
   }

})
```
代码分析：
- 在异步操作完成之后，会针对不同的返回结果调用resolve和reject。
- resolve和reject是两个函数，resolve是异步操作成功时候被调用，将异步操作的返回值作为参数传递到外部；reject是异步操作出异常时候被调用，将错误信息作为参数传递出去。
- <font color="red">Promise其实没有做任何实质的代码操作，它只是对异步操作回调函数的不同结果定义了不同状态。</font>
- resolve函数和reject函数只是把异步结果传递出去

2. 异步结果传递出去后，then来接
Promise对象将结果传递出来后，使用then方法来获取异步操作的值：
代码如下：

```
promise.then(function(value){
   //success
   
},function(error){

});
```
代码分析：
- then方法将两个匿名函数作为参数，接收resolve和reject这两个函数的值。
- value是执行成功的值，error是执行出错时的错误信息。
- 对于error错误异常结果出现的时候，可以不单独写匿名错误的函数，可以直接用catch抛出

```
promise.then(function (data){
    //success
})
.catch(function(error){
    //error  
})
```
- 注意then方法<font color="red">只是</font>用来获取异步操作的值。

3. then的返回值又是怎样呢？
先看一段调用两次then的代码：

```
//之前创建promise操作后
promise.then(function(value){
    conlose.log(value);  //有值
}).then(function(value)
{
   conlose.log(value);   //未定义
});
```
代码分析：
- 上面的第二个then方法中的值虽然是未定义，但是每一个then一定会<font color="red">返回一个新的promise对象</font>，但是默认是一个空对象。
- 对于这个空对象我们如果想继续做一些什么，需要进行处理，可以用非空Promise对这个空的进行赋值覆盖，然后继续then的链式调用。
- then 中的<font color="red">return</font>关键字很重要，联系着下一个then的调用。

##### 几个常用api

- Promise.resolve  
resolve方法用来将一个非Promise对象转化为Promise对象

转换的对象是一个常量或者不具备状态的语句，转换后的对象自动处于resolve状态。
转换的后的结果和原来一样

```
var promise =Promise.resolve("hello world");
promise.then(function(result){
  console.log(result);   //输出结果 hello world
})
```
转换的对象如果直接是一个异步方法，不可以这么使用。
- Promise.all(常用api)  
多个promise需要执行的时候，可以使用promise.all方法统一声明，该方法可以将多个Promise对象包装成一个Promise。

代码如下

```
promise.all(
//一系列promise操作
).then(function(results){
    
    
}).catch(function(error){
    
});
```
代码分析：
- promise.all对多个执行结果做一个包装传给了then
- promise.all中的执行顺序是怎么样的，Promise的执行顺序是从被创建开始的，也就是在调用all的时候，<font color="red">所有的promise都已经开始执行</font>了，all方法只是等到<font color="red">所有的对象都执行完成</font>，才会把结果<font color="red">传递给then</font>。
- all中的promise，如果有一个状态变成了reject那么转换后的Promise字节变成reject，错误信息传递给catch，不会传递给then。（但是并不是说all这里面刚开始执行成功的操作就不算数了）

## Promise在开发中的应用
项目开发中promise的应用代码：


```
Promise.all([
            self.count({phoneNumber: mobile, createdOn: {$gt: hour}}),
            self.count({ip: ip, createdOn: {$gt: hour}})
        ]).then(function (results) {
            if (results[0] >= 5) {
                return callback({code: -1, message: '短信发送频率过快，每手机号1小时内只能发送5次'});
            }
            if (results[1] >= 5) {
                return callback({code: -1, message: '短信发送频率过快，每IP1小时内只能发送5次'});
            }
            let code = {
                phoneNumber: mobile,
                code: tool.makeRandomStr(4, 1).toLowerCase(),
                createdOn: new Date(),
                expiredOn: new Date(new Date().getTime() + (20 * 60 * 1000)),			//20分钟失效
                ip: ip,
                isUsed: false
            };
            self.create(code, function (err, newCode) {
                if (newCode) {
                    sms.sendSMS(mobile, newCode.code, 'ali', function (err, body) {
                        console.log(body);
                        if (err)
                            console.log("短信验证码发送失败：", err);
                    });
                    callback({code: 0, message: "验证码已经发送"});
                } else {
                    callback({code: -1, message: "验证码发送失败，请重试"});
                }
            })
        })
```
项目开发过程中使用promise.all的代码，当时是为了实现短信验证码发送前的校验功能。
all中的两个promise，第一个是统计时间内该手机号发送验证码数量;第二个是统计时间内该ip发送验证码的数量。


## Promise使用过程中注意事项与误区
注意事项在上面原理讲解过程中，基本都提到过，只是重要的事情多说两遍。
- 状态不可逆性
- resolve函数和reject函数只是传递异步结果
- then进行层级调用的时候，每次的返回值都是一个空promise对象，如果想继续使用，赋值替换掉空promise对象，但是返回的时候return关键字很重要，不要忘了。
- promise.all中的执行顺序是并行的，但是会等全部完成的结果传递给then
- <font color="red">执行顺序</font>，promise是then方法调用之后才会执行吗？还是从创建那一刻就开始执行？
  promise从创建那一刻就开始执行，只是把结果传递给了then，then与promise的执行无关。


## Promise的反思
   Promise的讲解就到这里，但是大家在开发过程中，会发现有些时候多次操作异步会出现很多层级的调用，也就是
   
```
promise.then(...)

.then(...)

.then(...)
```
这种情况，代码虽然看起来会比callback的回调简介和规范了很多，但是还是感觉一些复杂，有没有更好的解决办法呢?请看下一篇博客

回调的终极使用--[async和await的讲解](/javascript/async-await.md)