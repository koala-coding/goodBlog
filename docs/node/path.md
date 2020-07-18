---
title: node核心模块 -文件路径path
date: 2019-07-30
tags:
   - Node.js
---

**前言**

之前在做`webpack`配置时候多次用到路径相关内容，最近在写项目的时候，有一个文件需要上传到阿里云oss的功能，同时本地服务器也需要保留一个文件备份。多次用到了文件路径相关内容以及Node核心API的`path`模块，所以系统的学习了一下，整理了这篇文章。

# node中的路径分类
node中的路径大致分5类，`dirname`,`filename`,`process.cwd()`,`./`,`../`,其中`dirname`,`filename`,`process.cwd()`绝对路径

通过代码对每个分类进行说明：

文件目录结构如下：

```
代码pra/
  - node核心API/
      - fs.js
      - path.js
```
path.js中的代码

```javascript
const path = require('path');
console.log(__dirname);
console.log(__filename);
console.log(process.cwd());
console.log(path.resolve('./'));
```
在代码pra目录下运行命令 `node node核心API/path.js`，我们可以看到结果如下：

```javascript
/koala/Desktop/程序员成长指北/代码pra/node核心API
/koala/Desktop/程序员成长指北/代码pra/node核心API/path.js
/koala/Desktop/程序员成长指北/代码pra
/koala/Desktop/程序员成长指北/代码pra
```
然后我们有可以在`node核心API目录下`运行这个文件，`node  path.js`,运行结果如下：

```javascript
/koala/Desktop/程序员成长指北/代码pra/node核心API
/koala/Desktop/程序员成长指北/代码pra/node核心API/path.js
/koala/Desktop/程序员成长指北/代码pra/node核心API
/koala/Desktop/程序员成长指北/代码pra/node核心API
```

对比输出结果，暂时得到的结论是
- __dirname: 总是返回被执行的 js 所在文件夹的绝对路径
- __filename: 总是返回被执行的 js 的绝对路径
- process.cwd(): 总是返回运行 node 命令时所在的文件夹的绝对路径
- ./: 跟 process.cwd() 一样，返回 node 命令时所在的文件夹的绝对路径

为什么说上面是暂时得到的结论，因为是有错误的，再看一段代码：
我们在path.js中加上这句代码


```javascript
exports.A = 1;
```
之前直接通过readFile读取文件路径报错，

```javascript
fs.readFile('./path.js',function(err,data){
   
});
```

现在在刚才报错的fs.js里面加这两句代码看看：


```javascript
const test = require('./path.js');
console.log(test)
```

在`代码pra/`目录下运行`node node核心API/fs.js`，最后查看结果，说明是可以访问到的：

```javascript
{ A: 1 }
```

那么关于 `./ `正确的结论是：

在 `require() `中使用是跟` __dirname` 的效果相同，不会因为启动脚本的目录不一样而改变，在其他情况下跟 `process.cwd()` 效果相同，是相对于启动脚本所在目录的路径。

#### 路径知识总结：
- `__dirname`： 获得当前执行文件所在目录的完整目录名
- `__filename`： 获得当前执行文件的带有完整绝对路径的文件名
- `process.cwd()`：获得当前执行node命令时候的文件夹目录名
- `./`： 不使用require时候，`./`与`process.cwd()`一样，使用`require`时候，与`__dirname`一样

只有在 require() 时才使用相对路径(./, ../) 的写法，其他地方一律使用绝对路径，如下：


```javascript
// 当前目录下
 path.dirname(__filename) + '/path.js'; 
// 相邻目录下
 path.resolve(__dirname, '../regx/regx.js');
```


## path
前面讲解了路径的相关比较，接下来单独聊聊path这个模块，这个模块在很多地方比较常用，所以，对于我们来说，掌握他，对我们以后的发展更有利，不用每次看webpack的配置文件还要去查询一下这个api是干什么用的，很影响我们的效率
> 这是api官网地址:https://nodejs.org/api/path.html

个人认为官网中的api没有必要都掌握，下面会对一些常用的api进行讲解，我经常用到的，或者作为一个前端开发工程师在webpack等工程配置的时候经常用到的。

### path.normalize
**举例说明**
```javascript
const path = require('path');

console.log(path.normalize('/koala/Desktop//程序员成长指北//代码pra/..'));
```

**规范后的结果**

```javascript
/koala/Desktop/程序员成长指北/代码pra
```
**作用总结**

规范化路径，把不规范的路径规范化。

### path.join

**举例说明**

```javascript
const path = require('path');
console.log(path.join('src', 'task.js'));

const path = require('path');
console.log(path.join(''));
```
**转化后的结果** 

```javascript
src/task.js
.
```

**作用总结**

> path.join([...paths])

1. 传入的参数是字符串的路径片段，可以是一个，也可以是多个
2. 返回的是一个拼接好的路径，但是根据平台的不同，他会对路径进行不同的规范化，举个例子，`Unix`系统是`/`，`Windows`系统是`\`，那么你在两个系统下看到的返回结果就不一样。
3. 如果返回的路径字符串长度为零，那么他会返回一个`.`，代表当前的文件夹。
4. 如果传入的参数中有不是字符串的，那就直接会报错


### path.parse
**举例说明**

```javascript
const path = require('path');
console.log(path.parse('/koala/Desktop/程序员成长指北/代码pra/node核心API'));
```

**运行结果**

```javascript
{ root: '/',
  dir: '/koala/Desktop/程序员成长指北/代码pra',
  base: 'node核心API',
  ext: '',
  name: 'node核心API' 
}
```
**作用总结** 

他返回的是一个对象，那么我们来把这么几个名词熟悉一下：
1. root：代表根目录
2. dir：代表文件所在的文件夹
3. base：代表整一个文件
4. name：代表文件名
5. ext: 代表文件的后缀名

### path.basename
**举例说明**

```javascript
const path = require('path');
console.log(path.basename('/koala/Desktop/程序员成长指北/代码pra/node核心API'));
console.log(path.basename('/koala/Desktop/程序员成长指北/代码pra/node核心API/path.js', '.js'));
```

**运行结果**

看了上面代码的例子，我想应该知道了basename结果，嘿嘿。

```javascript
node核心API
path
```
**作用总结** 

basename接收两个参数，第一个是`path`，第二个是`ext`(可选参数)，当输入第二个参数的时候，打印结果不出现后缀名

### path.dirname
**举例说明**

```javascript
const path = require('path');
console.log(path.dirname('/koala/Desktop/程序员成长指北/代码pra/node核心API'));
```
**运行结果** 

```javascript
/koala/Desktop/程序员成长指北/代码pra
```
**作用总结**

返回文件的目录完整地址

### path.extname
**举例说明**

```javascript
const path = require('path');
path.extname('index.html');
path.extname('index.coffee.md');
path.extname('index.');
path.extname('index');
path.extname('.index');
```
**运行结果**

```javascript
.html
.md
.
''
''
```
**作用总结**

返回的是后缀名，但是最后两种情况返回'',大家注意一下。

### path.resolve
**举例说明**

```javascript
const path = require('path');
console.log(path.resolve('/foo/bar', '/bar/faa', '..', 'a/../c'));
```
**输出结果**

```javascript
/bar/c
```
**作用总结**
> path.resolve([...paths])

path.resolve就相当于是shell下面的`cd`操作，从左到右运行一遍`cd path`命令，最终获取的绝对路径/文件名，这个接口所返回的结果了。但是`resolve`操作和`cd`操作还是有区别的，`resolve`的路径可以没有，而且最后进入的可以是文件。具体`cd`步骤如下

```javascript
cd /foo/bar/    //这是第一步, 现在的位置是/foo/bar/
cd /bar/faa     //这是第二步，这里和第一步有区别，他是从/进入的，也就时候根目录，现在的位置是/bar/faa
cd ..       //第三步，从faa退出来，现在的位置是 /bar
cd a/../c   //第四步，进入a，然后在推出，在进入c，最后位置是/bar/c
```

### path.relative
**举例说明**

```javascript
const path = require('path');

console.log(path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb'));

console.log(path.relative('/data/demo', '/data/demo'));

console.log(path.relative('/data/demo', ''));
```
**运行结果**

```javascript
../../impl/bbb
 ""
 ../../koala/Desktop/程序员成长指北/代码pra/node核心API
```
**作用总结**

path.relative(from, to)

描述：从from路径，到to路径的相对路径。

边界：

- 如果from、to指向同个路径，那么，返回空字符串。
- 如果from、to中任一者为空，那么，返回当前工作路径。
## 总结
本篇文章关于路径的知识就说到这里，基础很重要的，既能节约开发时间，又能减少报错。

今天就分享这么多，如果对分享的内容感兴趣，可以关注公众号「程序员成长指北」，或者加入技术交流群，大家一起讨论。

进阶技术路线

![](http://img.xiaogangzai.cn/way.jpg)
加入我们一起学习吧！
![](http://img.xiaogangzai.cn/code.jpg)