## 快速导航

- [对于MVVM的理解](#对于MVVM的理解)
- [开发中常用的指令有哪些](#开发中常用的指令有哪些)
- [请详细说下你对vue生命周期的理解](#请详细说下你对vue生命周期的理解)
- [Vue的双向数据绑定原理是什么](#Vue的双向数据绑定原理是什么)
- [Proxy相比于defineProperty的优势](#Proxy相比于defineProperty的优势)
- [vue-router有哪几种导航守卫?](#vue-router有哪几种导航守卫)
    * [1.全局守卫](#_1全局守卫)
    * [2.路由独享守卫](#_2路由独享守卫)
    * [3.路由组件内的守卫](#_3路由组件内的守卫)
- [Vue的路由实现：hash模式 和 history模式](#Vue的路由实现hash模式和history模式)
    * [hash模式](#hash模式：)
    * [history模式：](#history模式：)
- [组件之间的传值通信](#组件之间的传值通信)
    * [1.父组件给子组件传值](#_1父组件给子组件传值)
    * [2.子组件向父组件通信](#_2子组件向父组件通信)
    * [3.非父子, 兄弟组件之间通信](#_3非父子,兄弟组件之间通信)
- [Vue与Angular以及React的区别？](#Vue与Angular以及React的区别？)
- [vuex是什么？怎么使用？哪种功能场景使用它？](#vuex是什么？怎么使用？哪种功能场景使用它？)


## 对于MVVM的理解
**MVVM** 是 Model-View-ViewModel 的缩写
**Model**: 代表数据模型，也可以在Model中定义数据修改和操作的业务逻辑。我们可以把Model称为数据层，因为它仅仅关注数据本身，不关心任何行为
**View**: 用户操作界面。当ViewModel对Model进行更新的时候，会通过数据绑定更新到View
**ViewModel**： 业务逻辑层，View需要什么数据，ViewModel要提供这个数据；View有某些操作，ViewModel就要响应这些操作，所以可以说它是Model for View.
总结： MVVM模式简化了界面与业务的依赖，解决了数据频繁更新。MVVM 在使用当中，利用双向绑定技术，使得 Model 变化时，ViewModel 会自动更新，而 ViewModel 变化时，View 也会自动变化。

## 开发中常用的指令有哪些
- `v-model` :一般用在表达输入，很轻松的实现表单控件和数据的双向绑定
- `v-html`: 更新元素的 innerHTML
- `v-show` 与 `v-if`: 条件渲染, 注意二者区别
>使用了v-if的时候，如果值为false，那么页面将不会有这个html标签生成。
v-show则是不管值为true还是false，html元素都会存在，只是CSS中的display显示或隐藏
- `v-on : click`: 可以简写为@click,@绑定一个事件。如果事件触发了，就可以指定事件的处理函数
- `v-for`:基于源数据多次渲染元素或模板块
- `v-bind`: 当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM
> 语法：`v-bind:title="msg"`
> 简写：`:title="msg"`


## 请详细说下你对vue生命周期的理解
vue生命周期总共分为8个阶段: 创建前/后，载入前/后，更新前/后， 销毁前/后。
- **beforeCreate** （创建前）vue实例的挂载元素`$el`和数据对象 `data`都是`undefined`, 还未初始化
- **created** (创建后) 完成了 `data`数据初始化, `el`还未初始化
- **beforeMount** (载入前) vue实例的`$el`和`data`都初始化了, 相关的render函数首次被调用。实例已完成以下的配置：编译模板，把data里面的数据和模板生成html。注意此时还没有挂载html到页面上。
- **mounted** (载入后) 在`el` 被新创建的 `vm.$el `替换，并挂载到实例上去之后调用。实例已完成以下的配置：用上面编译好的`html`内容替换`el`属性指向的`DOM`对象。完成模板中的html渲染到html页面中。此过程中进行ajax交互
- **beforeUpdate** (更新前) 在数据更新之前调用，发生在虚拟DOM重新渲染和打补丁之前。可以在该钩子中进一步地更改状态，不会触发附加的重渲染过程。
- **updated** （更新后） 在由于数据更改导致的虚拟DOM重新渲染和打补丁之后调用。调用时，组件DOM已经更新，所以可以执行依赖于DOM的操作。然而在大多数情况下，应该避免在此期间更改状态，因为这可能会导致更新无限循环。该钩子在服务器端渲染期间不被调用。
- **beforeDestroy**  (销毁前） 在实例销毁之前调用。实例仍然完全可用。
- **destroyed** (销毁后） 在实例销毁之后调用。调用后，所有的事件监听器会被移除，所有的子实例也会被销毁。该钩子在服务器端渲染期间不被调用。


## Vue的双向数据绑定原理是什么
vue.js 是采用数据劫持结合发布者-订阅者模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。
具体实现步骤，感兴趣的可以看看:
1. 当把一个普通 Javascript 对象传给 Vue 实例来作为它的 data 选项时，Vue 将遍历它的属性，用 Object.defineProperty 都加上 setter和getter 这样的话，给这个对象的某个值赋值，就会触发setter，那么就能监听到了数据变化
2. compile解析模板指令，将模板中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，更新视图
3. Watcher订阅者是Observer和Compile之间通信的桥梁，主要做的事情是: 
1、在自身实例化时往属性订阅器(dep)里面添加自己 
2、自身必须有一个update()方法 
3、待属性变动dep.notice()通知时，能调用自身的update()方法，并触发Compile中绑定的回调，则功成身退。
4. MVVM作为数据绑定的入口，整合Observer、Compile和Watcher三者，通过Observer来监听自己的model数据变化，通过Compile来解析编译模板指令，最终利用Watcher搭起Observer和Compile之间的通信桥梁，达到数据变化 -> 视图更新；视图交互变化(input) -> 数据model变更的双向绑定效果

```javascript
//vue实现数据双向绑定的原理就是用Object.defineproperty()重新定义（set方法）对象设置属性值和（get方法）获取属性值的操纵来实现的。
//Object.property()方法的解释：Object.property(参数1，参数2，参数3)  返回值为该对象obj
//其中参数1为该对象（obj），参数2为要定义或修改的对象的属性名，参数3为属性描述符，属性描述符是一个对象，主要有两种形式：数据描述符和存取描述符。这两种对象只能选择一种使用，不能混合使用。而get和set属于存取描述符对象的属性。
//这个方法会直接在一个对象上定义一个新属性或者修改对象上的现有属性，并返回该对象。

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
</head>
<body>
    <div id="myapp">
        <input v-model="message" /><br>
        <span v-bind="message"></span>
    </div>
    <script type="text/javascript">
        var model = {
            message: ""
        };
        var models = myapp.querySelectorAll("[v-model=message]");
        for (var i = 0; i < models.length; i++) {
            models[i].onkeyup = function() {
                model[this.getAttribute("v-model")] = this.value;
            }
        }
        // 观察者模式 / 钩子函数
        // defineProperty 来定义一个对象的某个属性
        Object.defineProperty(model, "message", {
            set: function(newValue) {
                var binds = myapp.querySelectorAll("[v-bind=message]");
                for (var i = 0; i < binds.length; i++) {
                    binds[i].innerHTML = newValue;
                };
                var models = myapp.querySelectorAll("[v-model=message]");
                for (var i = 0; i < models.length; i++) {
                    models[i].value = newValue;
                };
                this.value = newValue;
            },
            get: function() {
                return this.value;
            }
        })
</script>
</body>
</html>

```
##  Proxy相比于defineProperty的优势 
Object.defineProperty() 的问题主要有三个：
- 不能监听数组的变化
- 必须遍历对象的每个属性
- 必须深层遍历嵌套的对象

Proxy 在 ES2015 规范中被正式加入，它有以下几个特点：
- 针对对象：针对整个对象，而不是对象的某个属性，所以也就不需要对 keys 进行遍历。这解决了上述 Object.defineProperty() 第二个问题
- 支持数组：Proxy 不需要对数组的方法进行重载，省去了众多 hack，减少代码量等于减少了维护成本，而且标准的就是最好的。
- 
除了上述两点之外，Proxy 还拥有以下优势：
- Proxy 的第二个参数可以有 13 种拦截方法，这比起 Object.defineProperty() 要更加丰富
- Proxy 作为新标准受到浏览器厂商的重点关注和性能优化，相比之下 Object.defineProperty() 是一个已有的老方法。

## vue-router有哪几种导航守卫
- 全局守卫
- 路由独享守卫
- 路由组件内的守卫
### 1.全局守卫
vue-router全局有三个守卫：
1. router.beforeEach 全局前置守卫 进入路由之前
2. router.beforeResolve 全局解析守卫(2.5.0+) 在beforeRouteEnter调用之后调用
3. router.afterEach 全局后置钩子 进入路由之后

使用方法:
```javascript
 // main.js 入口文件
    import router from './router'; // 引入路由
    router.beforeEach((to, from, next) => { 
      next();
    });
    router.beforeResolve((to, from, next) => {
      next();
    });
    router.afterEach((to, from) => {
      console.log('afterEach 全局后置钩子');
    });
```


### 2.路由独享守卫
如果你不想全局配置守卫的话，你可以为某些路由单独配置守卫：

```javascript
const router = new VueRouter({
      routes: [
        {
          path: '/foo',
          component: Foo,
          beforeEnter: (to, from, next) => { 
            // 参数用法什么的都一样,调用顺序在全局前置守卫后面，所以不会被全局守卫覆盖
            // ...
          }
        }
      ]
    })
```


### 3.路由组件内的守卫
1. beforeRouteEnter 进入路由前, 在路由独享守卫后调用 **不能** 获取组件实例 `this`，组件实例还没被创建
2. beforeRouteUpdate (2.2) 路由复用同一个组件时, 在当前路由改变，但是该组件被复用时调用 可以访问组件实例 `this`
3. beforeRouteLeave 离开当前路由时, 导航离开该组件的对应路由时调用，可以访问组件实例 `this`


## Vue的路由实现:hash模式和history模式
### hash模式：
在浏览器中符号“#”，#以及#后面的字符称之为hash，用window.location.hash读取；
特点：hash虽然在URL中，但不被包括在HTTP请求中；用来指导浏览器动作，对服务端安全无用，hash不会重加载页面。
hash 模式下，仅 hash 符号之前的内容会被包含在请求中，如 `http://www.xiaogangzai.com`，因此对于后端来说，即使没有做到对路由的全覆盖，也不会返回 404 错误。

### history模式：
history采用HTML5的新特性；且提供了两个新方法：pushState（），replaceState（）可以对浏览器历史记录栈进行修改，以及popState事件的监听到状态变更。
history 模式下，前端的 URL 必须和实际向后端发起请求的 URL 一致，如 `http://www.xxx.com/items/id`。后端如果缺少对 /items/id 的路由处理，将返回 404 错误。Vue-Router 官网里如此描述：“不过这种模式要玩好，还需要后台配置支持……所以呢，你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。”


## 组件之间的传值通信
组件之间通讯分为三种: 父传子、子传父、兄弟组件之间的通讯

#### 1.父组件给子组件传值
使用`props`，父组件可以使用`props`向子组件传递数据。

父组件vue模板father.vue:
```html
<template>
    <child :msg="message"></child>
</template>

<script>
import child from './child.vue';
export default {
    components: {
        child
    },
    data () {
        return {
            message: 'father message';
        }
    }
}
</script>
```

子组件vue模板child.vue:

```javascript
<template>
    <div>{{msg}}</div>
</template>

<script>
export default {
    props: {
        msg: {
            type: String,
            required: true
        }
    }
}
</script>
```

#### 2.子组件向父组件通信
父组件向子组件传递事件方法，子组件通过$emit触发事件，回调给父组件。

父组件vue模板father.vue:

```html
<template>
    <child @msgFunc="func"></child>
</template>

<script>
import child from './child.vue';
export default {
    components: {
        child
    },
    methods: {
        func (msg) {
            console.log(msg);
        }
    }
}
</script>

```

子组件vue模板child.vue:

```html
<template>
    <button @click="handleClick">点我</button>
</template>

<script>
export default {
    props: {
        msg: {
            type: String,
            required: true
        }
    },
    methods () {
        handleClick () {
            //........
            this.$emit('msgFunc');
        }
    }
}
</script>

```


### 3.非父子,兄弟组件之间通信
vue2中废弃了$dispatch和$broadcast广播和分发事件的方法。父子组件中可以用props和$emit()。如何实现非父子组件间的通信，可以通过实例一个vue实例Bus作为媒介，要相互通信的兄弟组件之中，都引入Bus，然后通过分别调用Bus事件触发和监听来实现通信和参数传递。
Bus.js可以是这样:

```javascript
import Vue from 'vue'
export default new Vue()
```

在需要通信的组件都引入Bus.js:

```html
<template>
	<button @click="toBus">子组件传给兄弟组件</button>
</template>

<script>
import Bus from '../common/js/bus.js'
export default{
	methods: {
	    toBus () {
	        Bus.$emit('on', '来自兄弟组件')
	    }
	  }
}
</script>
```

另一个组件也import Bus.js 在钩子函数中监听on事件
```javascript
import Bus from '../common/js/bus.js'
export default {
    data() {
      return {
        message: ''
      }
    },
    mounted() {
       Bus.$on('on', (msg) => {
         this.message = msg
       })
     }
   }
```

## Vue与Angular以及React的区别？

> 版本在不断更新，以下的区别有可能不是很正确。而且工作中只用到vue，对angular和react不怎么熟


#### Vue与AngularJS的区别
- Angular采用TypeScript开发, 而Vue可以使用javascript也可以使用TypeScript
-  AngularJS依赖对数据做脏检查，所以Watcher越多越慢；Vue.js使用基于依赖追踪的观察并且使用异步队列更新，所有的数据都是独立触发的。
- AngularJS社区完善, Vue的学习成本较小

#### Vue与React的区别
- vue组件分为全局注册和局部注册，在react中都是通过import相应组件，然后模版中引用；
- props是可以动态变化的，子组件也实时更新，在react中官方建议props要像纯函数那样，输入输出一致对应，而且不太建议通过props来更改视图；
- 子组件一般要显示地调用props选项来声明它期待获得的数据。而在react中不必需，另两者都有props校验机制；
- 每个Vue实例都实现了事件接口，方便父子组件通信，小型项目中不需要引入状态管理机制，而react必需自己实现；
- 使用插槽分发内容，使得可以混合父组件的内容与子组件自己的模板；
- 多了指令系统，让模版可以实现更丰富的功能，而React只能使用JSX语法；
- Vue增加的语法糖computed和watch，而在React中需要自己写一套逻辑来实现；
- react的思路是all in js，通过js来生成html，所以设计了jsx，还有通过js来操作css，社区的styled-component、jss等；而 vue是把html，css，js组合到一起，用各自的处理方式，vue有单文件组件，可以把html、css、js写到一个文件中，html提供了模板引擎来处理。
- react做的事情很少，很多都交给社区去做，vue很多东西都是内置的，写起来确实方便一些， 比如 redux的combineReducer就对应vuex的modules， 比如reselect就对应vuex的getter和vue组件的computed， vuex的mutation是直接改变的原始数据，而redux的reducer是返回一个全新的state，所以redux结合immutable来优化性能，vue不需要。
- react是整体的思路的就是函数式，所以推崇纯组件，数据不可变，单向数据流，当然需要双向的地方也可以做到，比如结合redux-form，组件的横向拆分一般是通过高阶组件。而vue是数据可变的，双向绑定，声明式的写法，vue组件的横向拆分很多情况下用mixin。


## vuex是什么？怎么使用？哪种功能场景使用它？
1. vuex 就是一个仓库，仓库里放了很多对象。其中 state 就是数据源存放地，对应于一般 vue 对象里面的 data
2. state 里面存放的数据是响应式的，vue 组件从 store 读取数据，若是 store 中的数据发生改变，依赖这相数据的组件也会发生更新
3. 它通过 mapState 把全局的 state 和 getters 映射到当前组件的 computed 计算属性

vuex的使用借助官方提供的一张图来说明:
![在这里插入图片描述](http://img.xiaogangzai.cn/16b1b698a29094b9.jpg)

Vuex有5种属性: 分别是 state、getter、mutation、action、module;

#### state
Vuex 使用单一状态树,即每个应用将仅仅包含一个store 实例，但单一状态树和模块化并不冲突。存放的数据状态，不可以直接修改里面的数据。


#### mutations
mutations定义的方法动态修改Vuex 的 store 中的状态或数据。

#### getters
类似vue的计算属性，主要用来过滤一些数据。

#### action
actions可以理解为通过将mutations里面处里数据的方法变成可异步的处理数据的方法，简单的说就是异步操作数据。view 层通过 store.dispath 来分发 action。


vuex 一般用于中大型 web 单页应用中对应用的状态进行管理，对于一些组件间关系较为简单的小型应用，使用 vuex 的必要性不是很大，因为完全可以用组件 prop 属性或者事件来完成父子组件之间的通信，vuex 更多地用于解决跨组件通信以及作为数据中心集中式存储数据。
- 使用Vuex解决非父子组件之间通信问题
vuex 是通过将 state 作为数据中心、各个组件共享 state 实现跨组件通信的，此时的数据完全独立于组件，因此将组件间共享的数据置于 State 中能有效解决多层级组件嵌套的跨组件通信问题。 

- vuex 作为数据存储中心
vuex 的 State 在单页应用的开发中本身具有一个“数据库”的作用，可以将组件中用到的数据存储在 State 中，并在 Action 中封装数据读写的逻辑。这时候存在一个问题，一般什么样的数据会放在 State 中呢？ 目前主要有两种数据会使用 vuex 进行管理：
1、组件之间全局共享的数据
2、通过后端异步请求的数据
比如做加入购物车、登录状态等都可以使用Vuex来管理数据状态。

一般面试官问到这里vue基本知识就差不多了， 如果更深入的研究就是和你探讨关于vue的底层源码；或者是具体在项目中遇到的问题，下面列举几个项目中可能遇到的问题：
 1. 开发时，改变数组或者对象的数据，但是页面没有更新如何解决？

2. vue弹窗后如何禁止滚动条滚动？
3. 如何在 vue 项目里正确地引用 jquery 和 jquery-ui的插件


### 公众号技术栈路线

大家好，我是koala，公众号「程序员成长指北」作者，这篇文章是【JS必知必会系列】的高阶函数讲解。目前在做一个node后端工程师进阶路线，加入我们一起学习吧！

![](http://img.xiaogangzai.cn/way.jpg)


### 加入我们

![](http://img.xiaogangzai.cn/code.jpg)




