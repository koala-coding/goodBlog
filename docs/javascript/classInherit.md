![](http://img.xiaogangzai.cn/article_15.jpg)

## 前言

先上两段代码：
java中定义类:

```javascript
public class Person{
    private String name;
    private int age;
   
    public Person(String name,int age){
        this.name=name;
        this.age=age;
    }
    public void getInfo(){ 
        System.out.println(name+age);
    }
}
```


Es6中定义一个类：

```javascript
class Person{
    constructor(name,age){
        this.name=name;
        this.age=age;
    }
    getInfo(){
        return this.name+','+this.age;
    }
}
//调用
let person=new Person("koala","123");
```

通过上面两段代码引出我们今天要说的相关内容

## 类中的变量
- 二者异
在java中可以直接声明各种类型的私有变量，在ES6中的类不可以直接在类中声明私有变量，声明后会报错。
注意：但是随着v8的更新，在node12版本中，ES增加了一些新规范，其中就有**支持类的私有变量**这一条。
代码如下：

```javascript
class Greet {
  #name = 'World';
  get name() {
    return this.#name;
  }
  set name(name) {
    this.#name = name;
  }
  sayHello() {
    console.log(`Hello, ${this.#name}`);
  }
}
```
在类的外部获取#name变量会抛出异常

```javascript
const greet = new Greet()
greet.#name = 'NewName';
// -> SyntaxError
console.log(greet.#name)
// -> SyntaxError
```

## 类中的构造函数
- 二者同：

如果声明一个一个类的时候没有声明构造函数，那么会默认添加一个空的构造函数，构造函数在new实例化一个对象的时候会被调用
- 二者异：

在ES6中，可以在构造函数中直接定义类方法(类方法也可以是箭头函数)，代码如下

```javascript
constructor(name,age){
    this.name=name;
    this.age=age;
    this.getInfo()=()=>{
        console.log("name"+this.name+"sex"+this.sex);
        }         
 } 
```

## 类中的方法
- 二者同：

有参，无参函数，函数调用方式相同。静态方法，ES6中用static声明一个静态方法，方法只能用类名直接调用，不能通过类的实例调用
- 二者异：

ES6在类中声明函数，无需使用function关键字，java的类中必须使用关键字声明函数。

ES6方法内部访问类属性的时候需要this来访问，java不需要。

ES6的构造函数中可以定义函数，java不可。
## 类中的继承
- 二者同：

继承关键字都是extends，super方法的使用
- 二者异：

继承的调用:

ES6需要注意的是super只能调用父类方法，而不能调用父类的属性，方法定义再原型链中，属性定义在类的内部

java中，super关键字，可以通过super关键字来实现对父类成员的访问，用来引用当前对象的父类。

继承过程中的构造函数：

ES6中，子类中，super方法是必须调用的，因为子类本身没有自身的this对象，需要通过super方法拿到父类的this对象。在子类中，没有构造函数，那么在默认的构造方法内部自动调用super方法，继承父类的全部属性，子类的构造方法中，必须先调用super方法，然后才能调用this关键字声明其它属性。(子类的this就是在这里调用super之后，拿到父类的this，然后修改这个this来的）

```javascript
class Student extends Person{
    constructor(name,sex){
        console.log(this);//Error
        super(name,sex);
        this.sex=sex;
    }
}
```


java中，子类是不继承父类的构造器（构造方法或者构造函数）的，它只是调用（隐式或显式）。如果父类的构造器带有参数，则必须在子类的构造器中显式地通过 super 关键字调用父类的构造器并配以适当的参数列表。

如果父类构造器没有参数，则在子类的构造器中不需要使用 super **关键字调用父类构造器，系统会自动调用父类的无参构造器。 **

看一段面试问的比较多的代码实例：


```javascript
class SuperClass {
  private int n;
  SuperClass(){
    System.out.println("SuperClass()");
  }
  SuperClass(int n) {
    System.out.println("SuperClass(int n)");
    this.n = n;
  }
}
class SubClass extends SuperClass{
  private int n;
  
  SubClass(){
    super(300);
    System.out.println("SubClass");
  }  
  
  public SubClass(int n){
    System.out.println("SubClass(int n):"+n);
    this.n = n;
  }
}
public class TestSuperSub{
  public static void main (String args[]){
    SubClass sc = new SubClass();
    SubClass sc2 = new SubClass(200); 
  }
}
```
输出结果：

```java
SuperClass(int n)
SubClass
SuperClass()
SubClass(int n):200
```




## ES6中的类与原型链的关系
看一下文初定义的一个的javascript类。它和原型链对等的代码如下：

```javascript
//类
class Person{
    constructor(name,age){
        this.name=name;
        this.age=age;
    }
    getInfo(){
        return this.name+','+this.age;
    }
}

//原型链
function Person(name, age) {
    this.name = name;
    this.age = age;
}
  
Person.prototype.getInfo = function () {
    return this.name+','+this.age;
};
console.log(Person)
let person = new Person("koala", 123);
```
针对代码进行一下说明讲解：
- 声明的新对象都不具备原型链(类)的函数，但是却可以调用原型链的函数

```javascript
Object.getOwnPropertyNames(p)//[ 'name', 'age' ] 从输出结果可以看出只有这两个属性，不具有getInfo函数
console.log(p.getInfo());//输出结果 kaola,123
```
不管是在原型链还是类中，获取的结果都是['name','age']
- 直接打印class,发现类实际是个函数，就是对应的构造函数，this关键字代表实例对象,简单的说是class就是构造函数，prototype对象的constructor属性，直接指向“类”的本身。

```javascript
//直接打印类
console.log(Person);
console.log(Point===Point.prototype.constructor);//true
```

- 与ES5一样，实例的属性除非显式定义在其本身（即定义在this对象上），否则都是定义在原型上（即定义在class上）。

```javascript
 person.hasOwnProperty('name') // true
 person.hasOwnProperty('age') // true
 person.hasOwnProperty('getInfo') // false
 person.__proto__.hasOwnProperty('getInfo') // true getInfo是原型对象的属性
```

- 类的所有实例共享一个原型对象

```javascript
let person1 = new Person("koala1", 124);
console.log(person.__proto__===person1.__proto__);//true
```
- 类的实例  构造方法默认返回实例对象(this)  但是返回this可以修改

## ES6中类的出现有什么好处
- js中的类仍然是基于原型的。即使在创建类后修改类的构造函数上的原型对象仍然不会有任何问题。
例子代码如下：

```javascript
class Foo {
    constructor(name) {
        this.name = name;
    }

    test1() {
        console.log("test1: name = " + this.name);
    }
}
Foo.prototype.test2 = function() {
    console.log("test2: name = " + this.name);
};
```
- 类的语法简单，不易出错，尤其是在继承层次结构上简单很多。

- 类保护你免受无法使用构造函数使用新的常见错误(通过让构造函数抛出异常，如果这不是构造函数的有效对象)。

代码例子如下：
ES6中类出现后实现继承：
```javascript
// ES6
class Person {
    constructor(first, last) {
        this.first = first;
        this.last = last;
    }

    personMethod() {
        // ...
    }
}

class Employee extends Person {
    constructor(first, last, position) {
        super(first, last);
        this.position = position;
    }

    employeeMethod() {
        // ...
    }
}

class Manager extends Employee {
    constructor(first, last, position, department) {
        super(first, last, position);
        this.department = department;
    }

    managerMethod() {
        // ...
    }
}
```
ES6中类未出现前实现继承：

```javascript
// ES5
var Person = function(first, last) {
    if (!(this instanceof Person)) {
        throw new Error("Person is a constructor function, use new with it");
    }
    this.first = first;
    this.last = last;
};

Person.prototype.personMethod = function() {
    // ...
};

var Employee = function(first, last, position) {
    if (!(this instanceof Employee)) {
        throw new Error("Employee is a constructor function, use new with it");
    }
    Person.call(this, first, last);
    this.position = position;
};
Employee.prototype = Object.create(Person.prototype);
Employee.prototype.constructor = Employee;
Employee.prototype.employeeMethod = function() {
    // ...
};

var Manager = function(first, last, position, department) {
    if (!(this instanceof Manager)) {
        throw new Error("Manager is a constructor function, use new with it");
    }
    Employee.call(this, first, last, position);
    this.department = department;
};
Manager.prototype = Object.create(Employee.prototype);
Manager.prototype.constructor = Manager;
Manager.prototype.managerMethod = function() {
    // ...
};

```

附录：

java中的继承   http://www.runoob.com/java/java-inheritance.html


