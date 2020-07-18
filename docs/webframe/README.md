### 前端文章目录
#### css
* [`面试`10个css高频面试题](/webframe/css/1.md)
* [渲染树的形成原理你真的很懂吗?](/webframe/css/render.md)

#### javascript必知必会系列
- [`[数据类型]` 经常被面试官问道的JavaScript数据类型知识你真的懂吗？](/webframe/javascript/datatype.md)
- [`[作用域]`深入理解 JavaScript, 从作用域与作用域链开始](/webframe/javascript/scoped.md)
- [`[闭包]`理解javascript中的闭包](/webframe/javascript/closure.md)
- [`[高阶函数]`高阶函数详解与实战训练](/webframe/javascript/higherFunc.md)
- [`[赋值拷贝]`js中赋值•浅拷贝•深拷贝](/webframe/javascript/copy.md)
- [`[原型链]`原型链这么看好像并不难](/webframe/javascript/prototype.md)
- [`[this]`this关键字](/webframe/javascript/this.md)
- [exports和module.exports的区别](/webframe/javascript/exports.md)


#### es6 es7..
* [`ES6系列`ES6中的类(对比java学习)](/webframe/es6/classInherit.md)
* [`ES6系列`promise](/webframe/es6/promise.md)
* [`ES7系列`async和await讲解](/webframe/es6/async-await.md)

#### vue
* [`vue组件通信`vue中组件之间8中通信方式](/webframe/vue/messageWays.md)


## 交流群里讨论问题整理
- [vue中如何动态加载远程js文件]()

典型写法
```javascript
const script = document.createElement('script')
script.type = "text/javascript"
script.src = "js地址"
document.body.appendChild(script)
```
如果这个能满足你的需求，就不需要看下面的代码了。

很多时候，我们需要的是在js加载完成后，再执行一些逻辑。那其实也很简单，使用promise包裹一下就可以达到目的：

```javascript
function loadJS(src) {
    return new Promise((resolve, reject) => {
        let script = document.createElement('script')
        script.type = 'text/javascript'
        script.onload = () => {
            // 加载完成后
            resolve()
        }
        script.onerror = () => {
            reject()
        }

        script.src = src
        document.getElementsByTagName('body')[0].appendChild(script)
    })
},
    //调用loadJS
    loadJS('js地址')
```

#### 常用软件下载
- teamviewer破解版 远程控制软件(附带破解视频教学) 提取码：7aud[下载](https://pan.baidu.com/s/1O_9hBfqq1vBLkx9E51RrWA) 
- centOS mac版本[下载](https://pan.baidu.com/s/1geK2kF5)
- postman破解版 接口调试工具 提取码：t5e9 [下载](https://pan.baidu.com/s/1FB82YFv6r2eSvj-5O3nczA)
- git win_x64 提取码：v3f1 [下载](https://pan.baidu.com/s/112SCA8KeS2Up6mekDl1uGw) 
- git win_32 提取码：01fk [下载](https://pan.baidu.com/s/1tMG-7agcfELfcbzBIsC2hQ) 
- navicat for mysql10.0.11简体中文破解版 提取码：z59z [下载](https://pan.baidu.com/s/1udENOBe6P_KQ7d8fyMBR6A 
- axureRP 9 破解版 提取码：t7jh [下载](https://pan.baidu.com/s/164DU5VoB8hYxqoT-QQd8Wg)

