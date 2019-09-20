## 前言
有些内容虽然不用，但是不代表面试不考。这里为大家送上5道常见的手写面试题，希望你们在面试中能遇到某一道也好，这样这篇文章就没有白写。

## 模仿实现new创建对象

```javascript
/**
 * 模仿 new
 * @return {} 
 */
function createNew() {
  let obj = {};
  let context = [].shift.call(arguments); // 获取构造函数
  obj.__proto__ = context.prototype;
  context.apply(obj, [...arguments]);
  return obj;
}

// @test
function Person(name) {
  this.name = name;
}
let p = createNew(Person, 'kaola');

```
## 模仿实现instanceof


```javascript
/**
 * 模仿实现 instanceof
 * @param   left  [左侧参数为一个实例对象]
 * @param   right [右侧为要判断的构造器函数]
 * @return  [true / false]
 */
function instanceOf (left, right) {
  let prototype = right.prototype; // 获取目标原型对象

  left = left.__proto__;

  while (true) {
    if(left == null) {
      return false;
    } else if (left == prototype) {
      return true;
    }
    left = left.__proto__
  }
}
```

## 模仿实现call

```javascript
/**
 * 模仿call
 * @param  context [要绑定的this对象]
 * @return         [返回函数执行结果]
 */
Function.prototype.call_new = function(context) {
  let context = context || window;
  context.fn = this;

  let args = [...arguments].clice(1);
  let result = context.fn(...args);
  delete context.fn;
  return result;
}


// @test

 foo.call_new(obj, 1,2,3)
```


## 模仿实现apply

```javascript
/**
 * 模仿apply
 * @param   context [要绑定的this对象]
 * @return  [执行结果]
 */
Function.prototype.apply_new = function(context) {
  let context = context || window;

  context.fn = this;

  let args = [...arguments][1];
  let result;
  if(args) {
    result = context.fn(...args);
  } else {
    result = context.fn()
  }

  delete context.fn;
  return result;

}
```


## 模仿实现bind


```javascript
/**
 * 模仿 bind
 * @param   context [要绑定的this对象]
 * @return  [执行结果]
 */
Function.ptototype.bind_new(context) {
  let self = this;
  let args = [...arguments].slice(1);
  return function() {
    let args1 = [...arguments].slice(1);
    return self.apply(context, args.concat(args1));
  }
}
```




