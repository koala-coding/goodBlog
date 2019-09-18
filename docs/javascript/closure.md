<!-- 闭包 -->
## 快速导航
- [什么是闭包](#什么是闭包)
- [闭包的常见案例分析](#闭包的常见案例分析)
  * [案例1-基本介绍](#案例1-基本介绍)
  * [案例2-前端实现点击事件](#案例2-前端实现点击事件)
  * [案例3-ajax请求](#案例3-ajax请求)
  * [案例4-for循环案例](#案例4-for循环案例)
- [闭包与内存泄漏](#闭包与内存泄漏)
- [闭包的应用](#闭包的应用)
- [面试题分析](#面试题分析)
- [总结](#总结)

## 什么是闭包
> 维基百科中的概念

-  在计算机科学中，闭包(也称词法闭包或函数闭包)是指一个函数或函数的引用，与一个引用环境绑定在一起，这个引用环境是一个存储该函数每个非局部变量(也叫自由变量)的表。
-  闭包，不同于一般的函数，它允许一个函数在立即词法作用域外调用时，仍可访问非本地变量


> 学术上

-  闭包是指在 JavaScript 中，内部函数总是可以访问**其所在的**外部函数中声明的参数和变量，即使在其外部函数被返回return掉（寿命终结）了之后。

> 个人理解

-  闭包是在函数里面定义一个函数，该函数可以是匿名函数，该子函数能够读写父函数的局部变量。


## 闭包的常见案例分析

**案例分析是从浅入深希望大家都看完!**

##### 案例1-基本介绍

```javascript
function A(){
    var localVal=10;
    return localVal;
}

A();//输出30


function A(){
    var localVal=10;
    return function(){
         console.log(localVal);
         return localVal;
    }
}
var func=A();
func();//输出10
```
两段代码，在第二段代码中，函数A内的匿名函数可以访问到函数A中的局部变量这就是闭包的基本使用。

##### 案例2-前端实现点击事件

```javascript
!function(){
    var localData="localData here";
    document.addEventListener('click',function(){
    console.log(localData);    
    });
}();
```
前端原始点击事件操作也用到了闭包来访问外部的局部变量。

##### 案例3-ajax请求

```javascript
!function(){
    var localData="localData here";
    var url="http://www.baidu.com";
    $.ajax({
      url:url,
      success:function(){
          //do sth...
          console.log(localData);
      }
    })
}();
```
在ajax请求的方法中也用到了闭包，访问外部的局部变量。

##### 案例4-for循环案例

```javascript
var arrays = [];

for (var i=0; i<3; i++) {
    arrays.push(function() {
        console.log('>>> ' + i); //all are 3
    });
}

```
上面的这段代码，刚看了代码一定会以为陆续打印出1，2，3，实际输出的是3，3，3，出现这种情况的原因是**匿名函数保存的是引用**，当for循环结束的时候，i已经变成3了，所以打印的时候变成3。出现这种情况的解决办法是利用闭包解决问题。

```javascript
for (var i=0; i<3; i++) {
    (function(n) {
        tasks.push(function() {
            console.log('>>> ' + n);
        });
    })(i);
}
```
**闭包里的匿名函数**，读取变量的顺序，先读取本地变量，再读取父函数的局部变量，如果找不到到全局里面搜索，**i作为局部变量存到闭包里面**，所以调整后的代码可以能正常打印1，2，3。


## 闭包与内存泄漏
- javascript回收后内存的方式:
  
javascript的主要通过计数器方式回收内存，假设有a，b，c三个对象，当a引用b的时候，那么b的引用计算器增加1(通俗的说用到那个对象哪个对象引用计算器增加1)，同时b引用c的时候，c引用计数器增加1，当a被释放的时候，b的引用计数器减少1，变成0的时候这个对象被释放，c计数器变成0，被释放,但是当遇到b和c之间互相引用的时候，无法通过计数器方式释放内存。

- 闭包可以导致上面所说b和c互相引用无法释放内存
第一个案例的代码拿过来分析：

```javascript
function A(){
    var localVal=10;
    return function(){
         console.log(localVal);
         return localVal;
    }
}
var func=A();
func();//输出10
```
当A函数结束的时候，想要释放，发现它的localVal变量被匿名函数引用，所有A函数无法释放，导致内存泄漏。但是也有好处，闭包正是可以做到这一点，因为它不会释放外部的引用，从而函数内部的值可以得以保留。

说明:闭包不代表一定会带来内存泄漏，良好的使用闭包是不会造成内存泄漏的。
## 闭包的应用
- 封装

```javascript
var person = function(){    
    //变量作用域为函数内部，外部无法访问    
    var name = "default";       
       
    return {    
       getName : function(){    
           return name;    
       },    
       setName : function(newName){    
           name = newName;    
       }    
    }    
}();    
     
print(person.name);//直接访问，结果为undefined    
print(person.getName());    
person.setName("kaola");    
print(person.getName());    
   
得到结果如下：  
   
undefined  
default  
kaola
```
- 实例中的for循环另一种形式

```javascript
doucument.body.innerHTML="<div id=div1>aaa</div>"+"<div id=div2>bbb</div>"+"<div id=div3>ccc</div>";
for(var i=1;i<4;i++){
    !function(i){
        document.getElementById('div'+i);
        addEventListener('click',function(){
           alert(i);//1,2,3 
        });
    }
}
```
- 结果缓存

```javascript
var CachedSearchBox = (function(){    
    var cache = {},    
       count = [];    
    return {    
       attachSearchBox : function(dsid){    
           if(dsid in cache){//如果结果在缓存中    
              return cache[dsid];//直接返回缓存中的对象    
           }    
           var fsb = new uikit.webctrl.SearchBox(dsid);//新建    
           cache[dsid] = fsb;//更新缓存    
           if(count.length > 100){//保正缓存的大小<=100    
              delete cache[count.shift()];    
           }    
           return fsb;          
       },    
     
       clearSearchBox : function(dsid){    
           if(dsid in cache){    
              cache[dsid].clearSelection();      
           }    
       }    
    };    
})();    
     
CachedSearchBox.attachSearchBox("input");
```
说明：开发中会碰到很多情况，设想我们有一个处理过程很耗时的函数对象，每次调用都会花费很长时间，那么我们就需要将计算出来的值存储起来，当调用这个函数的时候，首先在缓存中查找，如果找不到，则进行计算，然后更新缓存并返回值，如果找到了，直接返回查找到的值即可。闭包正是可以做到这一点，因为它不会释放外部的引用，从而函数内部的值可以得以保留。

## 面试题分析
闭包测试题: 求输出结果

```javascript
function fun(n,o){
    console.log(o);
    return {
        fun:function(m){//[2]
            return fun(m,n);//[1]
        }
    }
}

var a=fun(0);
a.fun(1);
a.fun(2);
a.fun(3);
var b=fun(0).fun(1).fun(2).fun(3);
var c=fun(0).fun(1);
c.fun(2);
c.fun(3);
```
由于分析内容比较多，大家可直接参考这篇文章
https://cnodejs.org/topic/567ed16eaacb6923221de48f

分析内容说明，在看这篇文章的时候，注意两点可能会看的更明白：
-  JS的词法作用域,JS变量作用域存在于函数体中即函数体，并且变量的作用域是在函数定义声明的时候就是确定的，而非在函数运行时。
-  在JS中调用函数的时候，如果用一个参数的方法调用两个参数的方法，这时候只是第二个参数未定义，代码不会报错停止运行，正常流程往下走，像面试题中仍然会返回一个对象。




## 总结
1. 闭包其实是在函数内部定义一个函数。
2. 闭包在使用的时候不会释放**外部的引用**，闭包函数内部的值会得到保留。
3. 闭包里面的匿名函数，读取变量的顺序，先读取本地变量，再读取父函数的局部变量。
4. 对于闭包外部无法引用它内部的变量，因此**在函数内部创建的变量**执行完后会立刻释放资源，不污染全局对象。
5. 闭包使用的时候要考虑到内存泄漏，因为不释放**外部引用**，但是合理的使用闭包是内存使用不是内存泄漏。
















