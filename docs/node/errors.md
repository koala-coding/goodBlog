---
title: node.js十个常见误区
date: 2019-08-26
tags:
   - Node.js
---
- 译：koala
- 原文地址：http://www.toptal.com/nodejs/top-10-common-nodejs-developer-mistakes
- 原文作者：MAHMUD RIDWAN

**前言**

自 Node.js 面世以来，它获得了大量的赞美和批判。这种争论会一直持续，短时间内都不会结束。而在这些争论中，我们常常会忽略掉所有语言和平台都是基于一些核心问题来批判的，就是我们怎么去使用这些平台。无论使用 Node.js 编写可靠的代码有多难，而编写高并发代码又是多么的简单，这个平台终究是有那么一段时间了，而且被用来创建了大量的健壮而又复杂的 web 服务。这些 web 服务不仅拥有良好的扩展性，而且通过在互联网上持续的时间证明了它们的健壮性。

 
然而就像其它平台一样，Node.js 很容易令开发者犯错。这些错误有些会降低程序性能，有些则会导致 Node.js 不可用。在本文中，我们会看到 Node.js 新手常犯的`十种错误`，以及如何去避免它们。

 
 
 
## 错误1：阻塞事件循环
 
Node.js（正如浏览器）里的 JavaScript 提供了一种单线程环境。这意味着你的程序不会有两块东西同时在运行，取而代之的是异步处理 I/O 密集操作所带来的并发。比如说 Node.js 给数据库发起一个请求去获取一些数据时，Node.js 可以集中精力在程序的其他地方：

```javascript
// Trying to fetch an user object from the database. Node.js is free to run other parts of the code from the moment this function is invoked..
db.User.get(userId, function(err, user) {
	// .. until the moment the user object has been retrieved here
})
```

然而，在一个有上千个客户端连接的 Node.js 实例里，一小段 CPU 计算密集的代码会阻塞住事件循环，导致所有客户端都得等待。CPU 计算密集型代码包括了尝试排序一个巨大的数组、跑一个耗时很长的函数等等。例如：
```javascript
function sortUsersByAge(users) {
	users.sort(function(a, b) {
		return a.age &lt; b.age ? -1 : 1
	})
}
```
在一个小的“users” 数组上调用“sortUsersByAge” 方法是没有任何问题的，但如果是在一个大数组上，它会对整体性能造成巨大的影响。如果这种事情不得不做，而且你能确保事件循环上没有其他事件在等待（比如这只是一个 Node.js 命令行工具，而且它不在乎所有事情都是同步工作的）的话，那这没有问题。但是，在一个 Node.js 服务器试图给上千用户同时提供服务的情况下，它就会引发问题。

 
如果这个 users 数组是从数据库获取的，那么理想的解决方案是从数据库里拿出已排好序的数据。如果事件循环被一个计算金融交易数据历史总和的循环所阻塞，这个计算循环应该被推到事件循环外的队列中执行以免占用事件循环。

 
正如你所见，解决这类错误没有银弹，只有针对每种情况单独解决。基本理念是不要在处理客户端并发连接的 Node.js 实例上做 CPU 计算密集型工作。

 
 
## 错误2：多次调用一个回调函数
 
一直以来 JavaScript 都依赖于回调函数。在浏览器里，事件都是通过传递事件对象的引用给一个回调函数（通常都是匿名函数）来处理。在 Node.js 里，回调函数曾经是与其他代码异步通信的唯一方式，直到 promise 出现。回调函数现在仍在使用，而且很多开发者依然围绕着它来设置他们的 API。一个跟使用回调函数相关的常见错误是多次调用它们。通常，一个封装了一些异步处理的方法，它的最后一个参数会被设计为传递一个函数，这个函数会在异步处理完后被调用：
```javascript
module.exports.verifyPassword = function(user, password, done) {
	if(typeof password !== ‘string’) {
		done(new Error(‘password should be a string’))
		return
	}
 
	computeHash(password, user.passwordHashOpts, function(err, hash) {
		if(err) {
			done(err)
			return
		}
		
		done(null, hash === user.passwordHash)
	})
}
```
这里注意到除了最后一次，每次“done” 方法被调用之后都会有一个 return 语句。这是因为调用回调函数不会自动结束当前方法的执行。如果我们注释掉第一个 return 语句，然后传一个非字符串类型的 password 给这个函数，我们依然会以调用 computeHash 方法结束。根据 computeHash 在这种情况下的处理方式，“done” 函数会被调用多次。当传过去的回调函数被多次调用时，任何人都会被弄得措手不及。

 
避免这个问题只需要小心点即可。一些 Node.js 开发者因此养成了一个习惯，在所有调用回调函数的语句前加一个 return 关键词：
```javascript
if(err) {
	return done(err)
}
```
在很多异步函数里，这种 return 的返回值都是没有意义的，所以这种举动只是为了简单地避免这个错误而已。

 
 
## 错误3：深层嵌套的回调函数
 
深层嵌套的回调函数通常被誉为“ 回调地狱”，它本身并不是什么问题，但是它会导致代码很快变得失控：
```javascript
function handleLogin(..., done) {
	db.User.get(..., function(..., user) {
		if(!user) {
			return done(null, ‘failed to log in’)
		}
		utils.verifyPassword(..., function(..., okay) {
			if(okay) {
				return done(null, ‘failed to log in’)
			}
			session.login(..., function() {
				done(null, ‘logged in’)
			})
		})
	})
}
```
 
越复杂的任务，这个的坏处就越大。像这样嵌套回调函数，我们的程序很容易出错，而且代码难以阅读和维护。一个权宜之计是把这些任务声明为一个个的小函数，然后再将它们联系起来。不过，（有可能是）最简便的解决方法之一是使用一个 Node.js 公共组件来处理这种异步 js，比如 Async.js：

```javascript
function handleLogin(done) {
	async.waterfall([
		function(done) {
			db.User.get(..., done)
		},
		function(user, done) {
			if(!user) {
			return done(null, ‘failed to log in’)
			}
			utils.verifyPassword(..., function(..., okay) {
				done(null, user, okay)
			})
		},
		function(user, okay, done) {
			if(okay) {
				return done(null, ‘failed to log in’)
			}
			session.login(..., function() {
				done(null, ‘logged in’)
			})
		}
	], function() {
		// ...
	})
}
```
Async.js 还提供了很多类似“async.waterfall” 的方法去处理不同的异步场景。为了简便起见，这里我们演示了一个简单的示例，实际情况往往复杂得多。

 
（打个广告，隔壁的《ES6 Generator 介绍》提及的 Generator 也是可以解决回调地狱的哦，而且结合 Promise 使用更加自然，请期待隔壁楼主的下篇文章吧:D）

 
 
## 错误4：期待回调函数同步执行
 
使用回调函数的异步程序不只是 JavaScript 和 Node.js 有，只是它们让这种异步程序变得流行起来。在其他编程语言里，我们习惯了两个语句一个接一个执行，除非两个语句之间有特殊的跳转指令。即使那样，这些还受限于条件语句、循环语句以及函数调用。

 
然而在 JavaScript 里，一个带有回调函数的方法直到回调完成之前可能都无法完成任务。当前函数会一直执行到底：

```javascript
function testTimeout() {
	console.log(“Begin”)
	setTimeout(function() {
		console.log(“Done!”)
	}, duration * 1000)
	console.log(“Waiting..”)
}
```
你可能会注意到，调用“testTimeout” 函数会先输出“Begin”，然后输出“Waiting..”，紧接着几秒后输出“Done!”。

 
任何要在回调函数执行完后才执行的代码，都需要在回调函数里调用。

 
 
## 错误5：给“exports” 赋值，而不是“module.exports”
 
Node.js 认为每个文件都是一个独立的模块。如果你的包有两个文件，假设是“a.js” 和“b.js”，然后“b.js” 要使用“a.js” 的功能，“a.js” 必须要通过给 exports 对象增加属性来暴露这些功能：

```javascript
// a.js
exports.verifyPassword = function(user, password, done) { ... }
``` 
完成这步后，所有需要“a.js” 的都会获得一个带有“verifyPassword” 函数属性的对象：
```javascript
// b.js
require(‘a.js’) // { verifyPassword: function(user, password, done) { ... } } 
``` 
然而，如果我们想直接暴露这个函数，而不是让它作为某些对象的属性呢？我们可以覆写 exports 来达到目的，但是我们绝对不能把它当做一个全局变量：

```javascript
// a.js
module.exports = function(user, password, done) { ... }
``` 
注意到我们是把“exports” 当做 module 对象的一个属性。“module.exports” 和“exports” 这之间区别是很重要的，而且经常会使 Node.js 新手踩坑。

 
 
## 错误6：从回调里抛出错误
 
JavaScript 有异常的概念。在语法上，学绝大多数传统语言（如 Java、C++）对异常的处理那样，JavaScript 可以抛出异常以及在 try-catch 语句块中捕获异常：
```javascript
function slugifyUsername(username) {
	if(typeof username === ‘string’) {
		throw new TypeError(‘expected a string username, got '+(typeof username))
	}
	// ...
}
 
try {
	var usernameSlug = slugifyUsername(username)
} catch(e) {
	console.log(‘Oh no!’)
}
``` 
然而，在异步环境下，tary-catch 可能不会像你所想的那样。比如说，如果你想用一个大的 try-catch 去保护一大段含有许多异步处理的代码，它可能不会正常的工作：

```javascript
try {
	db.User.get(userId, function(err, user) {
		if(err) {
			throw err
		}
		// ...
		usernameSlug = slugifyUsername(user.username)
		// ...
	})
} catch(e) {
	console.log(‘Oh no!’)
}
``` 
如果“db.User.get” 的回调函数异步执行了，那么 try-catch 原来所在的作用域就很难捕获到回调函数里抛出的异常了。

 
这就是为什么在 Node.js 里通常使用不同的方式处理错误，而且这使得所有回调函数的参数都需要遵循 (err, ...) 这种形式，其中第一个参数是错误发生时的 error 对象。

 
 
## 错误7：认为 Number 是一种整型数据格式
 
在 JavaScript 里数字都是浮点型，没有整型的数据格式。你可能认为这不是什么问题，因为数字大到溢出浮点型限制的情况很少出现。可实际上，当这种情况发生时就会出错。因为浮点数在表达一个整型数时只能表示到一个最大上限值，在计算中超过这个最大值时就会出问题。也许看起来有些奇怪，但在 Node.js 中下面代码的值是 true：
```javascript
Math.pow(2, 53)+1 === Math.pow(2, 53)
``` 
很不幸的是，JavaScript 里有关数字的怪癖可还不止这些。尽管数字是浮点型的，但如下这种整数运算能正常工作：
```javascript
5 % 2 === 1 // true
5 >> 1 === 2 // true
```
然而和算术运算不同的是，位运算和移位运算只在小于 32 位最大值的数字上正常工作。例如，让“Math.pow(2, 53)” 位移 1 位总是得到 0，让其与 1 做位运算也总是得到 0：
```javascript
Math.pow(2, 53) / 2 === Math.pow(2, 52) // true
Math.pow(2, 53) >> 1 === 0 // true
Math.pow(2, 53) | 1 === 0 // true
``` 
你可能极少会去处理如此大的数字，但如果你需要的话，有很多实现了大型精密数字运算的大整数库可以帮到你，比如 node-bigint。

 
 
## 错误8：忽略了流式 API 的优势
 
现在我们想创建一个简单的类代理 web 服务器，它能通过拉取其他 web 服务器的内容来响应和发起请求。作为例子，我们创建一个小型 web 服务器为 Gravatar 的图像服务。

```javascript
var http = require('http')
var crypto = require('crypto')
 
http.createServer()
.on('request', function(req, res) {
	var email = req.url.substr(req.url.lastIndexOf('/')+1)
	if(!email) {
		res.writeHead(404)
		return res.end()
	}
 
	var buf = new Buffer(1024*1024)
	http.get('http://www.gravatar.com/avatar/'+crypto.createHash('md5').update(email).digest('hex'), function(resp) {
		var size = 0
		resp.on('data', function(chunk) {
			chunk.copy(buf, size)
			size += chunk.length
		})
		.on('end', function() {
			res.write(buf.slice(0, size))
			res.end()
		})
	})
})
.listen(8080)
``` 
在这个例子里，我们从 Gravatar 拉取图片，将它存进一个 Buffer 里，然后响应请求。如果 Gravatar 的图片都不是很大的话，这样做没问题。但想象下如果我们代理的内容大小有上千兆的话，更好的处理方式是下面这样：

```javascript
http.createServer()
.on('request', function(req, res) {
	var email = req.url.substr(req.url.lastIndexOf('/')+1)
	if(!email) {
		res.writeHead(404)
		return res.end()
	}
 
	http.get('http://www.gravatar.com/avatar/'+crypto.createHash('md5').update(email).digest('hex'), function(resp) {
		resp.pipe(res)
	})
})
.listen(8080)
``` 
这里我们只是拉取图片然后简单地以管道方式响应给客户端，而不需要在响应它之前读取完整的数据存入缓存。

 
 
## 错误9：出于 Debug 的目的使用 Console.log
 
在 Node.js 里，“console.log” 允许你打印任何东西到控制台上。比如传一个对象给它，它会以 JavaScript 对象的字符形式打印出来。它能接收任意多个的参数并将它们以空格作为分隔符打印出来。有很多的理由可以解释为什么开发者喜欢使用它来 debug 他的代码，然而我强烈建议你不要在实时代码里使用“console.log”。你应该要避免在所有代码里使用“console.log” 去 debug，而且应该在不需要它们的时候把它们注释掉。你可以使用一种专门做这种事的库代替，比如 debug。

 
这些库提供了便利的方式让你在启动程序的时候开启或关闭具体的 debug 模式，例如，使用 debug 的话，你能够阻止任何 debug 方法输出信息到终端上，只要不设置 DEBUG 环境变量即可。使用它十分简单：

```javascript
// app.js
var debug = require(‘debug’)(‘app’)
debug(’Hello, %s!’, ‘world’)
``` 
开启 debug 模式只需简单地运行下面的代码把环境变量 DEBUG 设置到“app” 或“*” 上：
```javascript
DEBUG=app node app.js
``` 
 
## 错误10：不使用监控程序
 
不管你的 Node.js 代码是跑在生产环境或是你的本地开发环境，一个能协调你程序的监控程序是十分值得拥有的。一条经常被开发者提及的，针对现代程序设计和开发的建议是你的代码应该有 `fail-fast` 机制。如果发生了一个意料之外的错误，不要尝试去处理它，而应该让你的程序崩溃然后让监控程序在几秒之内重启它。监控程序的好处不只是重启崩溃的程序，这些工具还能让你在程序文件发生改变的时候重启它，就像崩溃重启那样。这让开发 Node.js 程序变成了一个更加轻松愉快的体验。

 
Node.js 有太多的监控程序可以使用了，例如：

`pm2`

`forever`

`nodemon`

`supervisor`

 
所有这些工具都有它的优缺点。一些擅长于在一台机器上处理多个应用程序，而另一些擅长于日志管理。不管怎样，如果你想开始写一个程序，这些都是不错的选择。

 
## 总结
 
你可以看到，这其中的一些错误能给你的程序造成破坏性的影响，在你尝试使用 Node.js 实现一些很简单的功能时一些错误也可能会导致你受挫。即使 Node.js 已经使得新手上手十分简单，但它依然有些地方容易让人混乱。从其他语言过来的开发者可能已知道了这其中某些错误，但在 Node.js 新手里这些错误都是很常见的。幸运的是，它们都可以很容易地避免。我希望这个简短指南能帮助新手更好地编写 Node.js 代码，而且能够给我们大家开发出健壮高效的软件。


加入我们一起学习吧！
![](http://img.xiaogangzai.cn/leading.png)

node学习交流群

> 交流群满100人不能自动进群, 请添加群助手微信号:【coder_qi】备注node，自动拉你入群。