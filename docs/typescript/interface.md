## 接口带来了什么好处
### 好处One —— 过去我们写 JavaScript
JavaScript 中定义一个函数，用来获取一个用户的姓名和年龄的字符串：
```javascript
const getUserInfo = function(user) { 
    return name: ${user.name}, age: ${user.age} 
}
```
函数调用:
```javascript
getUserInfo({name: "koala", age: 18})
```
这对于我们之前在写 JavaScript 的时候，再正常不过了，但是如果这个 `getUserInfo` 在多人开发过程中，如果它是个`公共函数`，多个开发者都会调用，如果不是每个人点进来看函数对应`注释`，可能会出现以下问题：
```javascript
// 错误的调用
getUserInfo() // Uncaught TypeError: Cannot read property 'name' of undefined
console.log(getUserInfo({name: "kaola"})) // name: kaola, age: undefined
getUserInfo({name: "kaola", height: 1.66}) // name: koala, age: undefined
```
JavaScript 是**弱类型**的语言，所以并不会对我们传入的代码进行任何的检测，**有些错你自己都说不清楚**，但是就出了问题。
#### TypeScript 中的 interface 可以解决这个问题

```javascript
const getUserInfo = (user: {name: string, age: number}): string => {
  return `name: ${user.name} age: ${user.age}`;
};
```
正确的调用是如下的方式：
```javascript
getUserInfo({name: "kaola", age: 18});
```
如果调用者出现了错误的调用，那么 `TypeScript` 会直接给出错误的提示信息：
```
// 错误的调用
getUserInfo(); // 错误信息：An argument for 'user' was not provided.
getUserInfo({name: "coderwhy"}); // 错误信息：Property 'age' is missing in type '{ name: string; }'
getUserInfo({name: "coderwhy", height: 1.88}); // 错误信息：类型不匹配
```
这时候你会发现这段代码还是有点长，代码不便与阅读，这时候就体现了 `interface` 的必要性。

**使用 interface 对 user 的类型进行重构。**

我们先定义一个 `IUser` 接口：
```javascript
// 先定义一个接口
interface IUser {
  name: string;
  age: number;
}
```
接下来我们看一下`函数`如何来写：
```javascript
const getUserInfo = (user: IUser): string => {
  return `name: ${user.name}, age: ${user.age}`;
};

// 正确的调用
getUserInfo({name: "koala", age: 18});
```
// 错误的调用和之前一样，报错信息也相同不再说明。

**接口中函数的定义再次改造**

定义两个接口：
```typescript
type IUserInfoFunc = (user: IUser) => string;

interface IUser {
  name: string;
  age: number;
}
```
接着我们去定义函数和调用函数即可：
```typescript
const getUserInfo: IUserInfoFunc = (user) => {
  return `name: ${user.name}, age: ${user.age}`;
};
```
//  正确的调用
```typescript
getUserInfo({name: "koala", age: 18});
```
//  错误的调用
```typescript
getUserInfo();
```
### 好处TWO —— 过去我们用 Node.js 写后端接口

其实这个说明和上面类似，我再提一下，就是想证明 TypeScript 确实挺香的！
写一个后端接口，我要特意封装一个工具类，来检测前端给我传递过来的参数，比如下图中的`validate`专门用来检验参数的函数
![](https://user-gold-cdn.xitu.io/2019/11/18/16e7a1d724cdef1a?w=1634&h=474&f=png&s=106934)
但是有了 TypeScript 这个参数检验函数可以省略了，我们可以这样写：
```typescript
 const goodParams: IGoodsBody = this.ctx.body;
```
而`GoodsBody`就是对应参数定义的 `interface`，比如这个样子

```typescript
// -- 查询列表时候使用的接口
interface IQuery {
    page: number;
    rows: number;
    disabledPage?: boolean; // 是否禁用分页，true将会忽略`page`和`rows`参数
 }
// - 商品
export interface IGoodsQuery extends Query {
    isOnline?: string | number; // 是否出售中的商品
    goodsNo?: string; // 商品编号
    goodsName?: string; // 商品名称
 }
```
好的，说了他的几个好处之后，我们开始学习interface知识吧！还有很多优点哦！

作者简介：koala，专注完整的 Node.js 技术栈分享，从 JavaScript 到 Node.js,再到后端数据库，祝您成为优秀的高级 Node.js 工程师。【程序员成长指北】作者，Github 博客开源项目 https://github.com/koala-coding/goodBlog

## 接口的基础篇
### 接口的定义
和 java 语言相同，TypeScript 中定义接口也是使用 interface 关键字来定义：
```typescript
interface IQuery {
  page: number;
}
```
你会发现我都在接口的前面加了一个`I`，算是个人习惯吧，之前一直写 java 代码，另一方面`tslint`要求，否则会报一个警告，是否加看个人。


### 接口中定义方法
看上面的接口中，我们定义了 `page` 常规属性，定义接口时候不仅仅可以有 属性，也可以有方法，看下面的例子：
```javascript
interface IQuery {
  page: number;
  findOne(): void;
  findAll(): void;
}
```
如果我们有一个对象是该接口类型，那么必须包含对应的属性和方法(无可选属性情况)：
```javascript
const q: IQuery = {
  page: 1,
  findOne() {
    console.log("findOne");
  },
  findAll() {
    console.log("findAll");
  },
};
```
### 接口中定义属性
#### 普通属性
上面的 `page` 就是普通属性，如果有一个对象是该接口类型，那么必须包含对应的普通属性。就不具体说了。

#### 可选属性
默认情况下一个变量（对象）是对应的接口类型，那么这个变量（对象）必须实现接口中所有的属性和方法。

但是，开发中为了让接口更加的灵活，某些属性我们可能希望设计成可选的（想实现可以实现，不想实现也没有关系），这个时候就可以使用`可选属性`（后面详细讲解函数时，也会讲到函数中有可选参数）：
```javascript
interface IQuery {
  page: number;
  findOne(): void;
  findAll(): void;
  isOnline?: string | number; // 是否出售中的商品
  delete?(): void
}
```
上面的代码中，我们增加了`isOnline`属性和`delete`方法，这两个都是可选的：

> 注意:可选属性如果没有赋值，那么获取到的值是`undefined`；
 对于可选方法，必须先进行判断，再调用，否则会报错；
 
 ```javascript
const q: IQuery = {
  page: 1,
  findOne() {
    console.log("findOne");
  },
  findAll() {
    console.log("findAll");
  },
};

console.log(p.isOnline); // undefined
p.delete(); // 不能调用可能是“未定义”的对象。
```
正确的调用方式如下：
```javascript
if (p.delete) {
  p.delete();
}
```
大家可能会问既然是可选属性，可有可无的，那么为什么还要定义呢?对比起完全不定义，定义可选属性主要是：为了让`接口更加的灵活`，某些属性我们可能希望设计成可选,并且如果存在属性，能`约束类型`，而这也是十分关键的。

#### 只读属性
默认情况下，接口中定义的属性可读可写：
但是有一个关键字 `readonly`，定义的属性值，不可以进行修改，强制修改后报错。
```javascript
interface IQuery {
  readonly page: number;
  findOne(): void;
}
```
给` page `属性加了` readonly `关键字,再给它赋值会报错。
```javascript 
const q: IQuery = {
  page: 1,
  findOne() {
    console.log("findOne");
  },
};
q.page = 10;// Cannot assign to 'page' because it is a read-only property.
```
## 接口的高级篇
### 函数类型接口
Interface 还可以用来规范函数的形状。Interface 里面需要列出参数列表返回值类型的函数定义。写法如下：

- 定义了一个函数接口
- 接口接收三个参数并且不返回任何值
- 使用函数表达式来定义这种形状的函数

```javascript
interface Func {
    // ✔️ 定于这个函数接收两个必选参数都是 number 类型，以及一个可选的字符串参数 desc，这个函数不返回任何值
    (x: number, y: number, desc?: string): void
}

const sum: Func = function (x, y, desc = '') {
    // const sum: Func = function (x: number, y: number, desc: string): void {
    // ts类型系统默认推论可以不必书写上述类型定义
    console.log(desc, x + y)
}

sum(32, 22)
```
注意:不过上面的接口中只有一个函数，TypeScript 会给我们一个建议，可以使用 `type` 来定义一个函数的类型：
```javascript
type Func = (x: number, y: number, desc?: string) => void;
```
### 接口的实现
接口除了定义某种`类型规范`，也可以和其他编程语言一样，让一个`类去实现某个接口`，那么这个类就必须明确去拥有这个接口中的属性和实现其方法：

下面的代码中会有关于修饰符的警告，暂时忽略，后面详细讲解
// 定义一个实体接口
```javascript
interface Entity {
  title: string;
  log(): void;
}
```
// 实现这样一个接口
```javascript
class Post implements Entity {
  title: string;

  constructor(title: string) {
    this.title = title;
  }

  log(): void {
    console.log(this.title);
  }
}
```
**有些小伙伴的疑问？我定义了一个接口，但是我在继承这个接口的类中还要写接口的实现方法，那我不如直接就在这个类中写实现方法岂不是更便捷，还省去了定义接口？这是一个初学者经常会有疑惑的地方。**

解答这个疑惑之前，先记住两个字，**规范**!

这个规范可以达到你一看这名字，就知道他是用来干什么的，并且可拓展，可以维护。

- 在**代码设计**中，接口是一种规范；
接口通常用于来定义某种规范, 类似于你必须遵守的协议, 

- 站在**程序角度**上说接口只规定了类里必须提供的属性和方法，从而分离了规范和实现，增强了系统的可拓展性和可维护性；



### 接口的继承
和类一样，接口也能继承其他的接口。这相当于复制接口的所有成员。接口也是用关键字 `extends` 来继承。

```javascript
interface Shape {     //定义接口Shape
    color: string;
}

interface Square extends Shape {  //继承接口Shape
    sideLength: number;
}
```
一个 interface 可以同时继承多个 interface ，实现多个接口成员的合并。用**逗号**隔开要继承的接口。
``` javascript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}
```
　　需要注意的是，尽管支持继承多个接口，但是如果继承的接口中，定义的同名属性的类型不同的话，是**不能编译通过**的。如下代码：
```javascript
interface Shape {
    color: string;
    test: number;
}

interface PenStroke extends Shape{
    penWidth: number;
    test: string;
}
```
另外关于继承还有一点，如果现在有一个类实现了 Square 接口，那么不仅仅需要实现 Square 的方法，也需要实现 Square 继承自的接口中的方法，实现接口使用 `implements` 关键字 。

### 可索引类型接口
### interface和type的区别
#### type 可以而 interface 不行
- type 可以声明基本类型别名，联合类型，元组等类型
```
// 基本类型别名
type Name = string

// 联合类型
interface Dog {
    wong();
}
interface Cat {
    miao();
}

type Pet = Dog | Cat

// 具体定义数组每个位置的类型
type PetList = [Dog, Pet]
```

- type 语句中还可以使用 typeof 获取实例的 类型进行赋值
```javascript
// 当你想获取一个变量的类型时，使用 typeof

let div = document.createElement('div');
type B = typeof div
```

- type 其他骚操作
```javascript
type StringOrNumber = string | number;  
type Text = string | { text: string };  
type NameLookup = Dictionary<string, Person>;  
type Callback<T> = (data: T) => void;  
type Pair<T> = [T, T];  
type Coordinates = Pair<number>;  
type Tree<T> = T | { left: Tree<T>, right: Tree<T> };
```
#### interface 可以而 type 不行
interface 能够声明合并
```javascript
interface User {
  name: string
  age: number
}

interface User {
  sex: string
}

/*
User 接口为 {
  name: string
  age: number
  sex: string 
}
*/
```
另外关于type的更多内容，可以查看文档：[TypeScript官方文档](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md)

## 接口的应用场景总结
   在项目中究竟怎么用，开篇已经举了两个例子，在这里再简单写一点，最近尝试了一下egg+ts，学习下。在写查询参数检验的时候，或者返回固定数据的时候，都会用到接口，看一段简单代码，已经看完了上面的文章，自己体会下吧。
```javascript
import User from '../model/user';
import Good from '../model/good';

// 定义基本查询类型
// -- 查询列表时候使用的接口
interface Query {
    page: number;
    rows: number;
    disabledPage?: boolean; // 是否禁用分页，true将会忽略`page`和`rows`参数
  }

// 定义基本返回类型
type GoodResult<Entity> = {
    list: Entity[];
    total: number;
    [propName: string]: any;
};

// - 商品
export interface GoodsQuery extends Query {
    isOnline?: string | number; // 是否出售中的商品
    goodsNo?: string; // 商品编号
    goodsName?: string; // 商品名称
}
export type GoodResult = QueryResult<Good>;

```
## 总结
`TypeScript` 还是挺香的，预告一篇明天的发文吧，`TypeScript`强大的类型别名。今天就分享这么多，如果对分享的内容感兴趣，可以关注公众号「程序员成长指北」，加我微信(coder_qi)，拉你进技术群，长期交流学习。


## 参考文章
https://juejin.im/post/5c8fbf516fb9a070d8781b3c#heading-6

https://www.teakki.com/p/57dfb5a0d3a7507f975ea2cc

https://juejin.im/post/5c2723635188252d1d34dc7d

https://mp.weixin.qq.com/s/aj45tr7AZkWYbFXyA-Ku-Q

http://cw.hubwiz.com/card/c/55b724ab3ad79a1b05dcc26c/1/5/4/



### 关注我
- 欢迎加我微信(coder_qi)，拉你进技术群，长期交流学习...
- 欢迎关注「程序员成长指北」,一个用心帮助你成长的公众号...
![](https://user-gold-cdn.xitu.io/2019/10/29/16e166ee15647127?w=900&h=500&f=png&s=105652)