## 对象着手
在谈原型链之前，先了解对象。
- 所有引用类型（函数，数组，对象）都拥有__proto__属性（隐式原型）
- 所有函数拥有prototype属性（显式原型）（仅限函数）
- 原型对象：拥有prototype属性的对象，在定义函数时就被创建

**prototype与__proto__两个概念**

- `prototype`：此属性只有构造函数才有，它指向的是当前构造函数的原型对象。
- `__proto__`：此属性是任何对象在创建时都会有的一个属性，它指向了产生当前对象的构造函数的原型对象，由于并非标准规定属性，不要随便去更改这个属性的值，以免破坏原型链，但是可以借助这个属性来学习，所谓的原型链就是由__proto__连接而成的链。

## 原型链详解
在js代码中 通过对象创建(下面一段简单的代码)详细分析原型链 一段简单代码:
```javascript
  function foo(){}
  foo.prototype.z=3;
  var obj=new foo();
  obj.y=2;
  obj.x=1;


  //调用
  obj.x;//1
  obj.y;//2


  obj.z;//3


  typeof obj.toString;//'function'
  'z' in obj;//true
  obj.hasOwnProperty('z');//false


  obj.z=5;
  obj.z;//5
  'z' in obj;//true
  obj.hasOwnProperty('z');//true
  foo.prototype.z;//3
```

**代码简单分析**

上面一段代码，声明第一个函数foo的时候，它就会带一个foo.prototype的属性，这个属性是一个对象属性，用new foo();构造器的方式构造一个新的对象obj。这时候这个obj的原型会指向foo的prototype属性。 对于这个foo函数的原型也会指向Object.prototype,这个Object.prototype也是有原型的，它的原型指向null。

**代码对象原型链图:**

![](https://camo.githubusercontent.com/bca984b0376f5458b52b30f49c09c3d57f513962/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f362f31392f313662366261333366326666656430323f773d34363626683d34383826663d706e6726733d3236303336) 


**对象访问属性顺序**

对象访问属性的顺序，是采用向上查找，如果当前对象没有，它会一直向上原型链中查找，一直找到null，如果还没有会返回undefind。

**对象中值修改说明**

代码中修改obj.z的值后，再次输出obj.z的时候是5,foo.prototype.z是3，说明我们在修改或添加对象的属性的时候，只是修改了对象本身obj.prototype.z中的值，而原型链中foo.prototype.z的值并不会修改。

**in，hasOwnProperty等方法的出现**

首先查看整个原型链，会想这两个方法是怎么来的，在foo的的proto指向上一级Object.prototype的时候，就可以访问Object中的一些函数和属性了，其中就包括这两个方法。

第一次调用
```javascript
'z' in obj;//true  
obj.hasOwnProperty('z');//false
```

表示的是z并不是obj这个对象上的，而是对象的原型链上的。

```javascript
 'z' in obj;//true
  obj.hasOwnProperty('z');//true
  foo.prototype.z;//3
```

第二次修改了obj.z的值，z就是obj这个对象上的了，但是也并没有修改原型链中的z的值。

**特殊说明**

_proto_是每一个对象都有的属性，它的指向会有一个特殊说明，大多数情况下 _proto_指向了**产生当前对象的构造函数的原型对象**，也就是那个 prototype。但是会有特殊的情况

- 特殊情况
```javascript
 var a={};
 var b=Object.create(a);
```

object.create是创建了一个空对象，空对象的原型指向a，a也是空对象，这其中不存在prototype;Object.create在继承中也常被使用，创建一个空对象指向()内的对象，这这样实现了b继承a，也不会篡改a中的内容，在这里就不具体说明了。

原理图分析
![](https://camo.githubusercontent.com/93d38d91dfb1c8c02c9dd7c76b79ba7f8d688a72/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031392f362f31392f313662366261333366326137383165623f773d34333026683d33363726663d706e6726733d3234343633)

## 总结

### 到底什么是原型链?
proto是任何对象都有的属性，在js中会形成一条proto连起来的链条，递归访问proto必须最终到头，并且值是null。

### 误区
写这篇总结的过程中，发现很多文章都写了“JS中万物皆对象”。难道真的是这样吗？ 错，JS世界很大，并不只有对象