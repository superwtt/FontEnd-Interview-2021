#### ES6构造函数语法糖：class类

为了解决ES6中原型链继承实现给我们造成的麻烦，ES6又给我们提供了一颗语法糖：Class

本文将通过以下几个关键字：Class、constructor、static、extends、super来具体了解一下Class

---

##### Class

顾名思义，就是“类”，对比传统构造函数和class的写法

```javascript
// 传统写法
function Animal(type,name){
   this.type=type;
   this.name=name; 
}
Animal.prototype.toString = function(){
    return '('+this.type+','+this.name')'
}
var m = new Animal('monkey','yuan')

// class写法
class Animal{
    constructor(type,name){
       this.type = type;
       this.name = name; 
    }
    toString(){
       return '('+this.type+','+this.name')'
    }
}
var m = new Animal('monkey','yuan')
m.toString(); // (monkey,yuan)
```

解析：

+ 通过 class 关键字可以定义类，提供了更接近传统语言的写法，引入了class类这个概念作为对象的模板

+ 类中所有的方法都是定义在类的 <font style="color:red">`prototype`</font>属性上

  ```javascript
  class Animal{
    constructor(){...}
    toString(){...}
    getName(){...}  
  }
  // 等价于
  Animal.prototype = {
    constructor(){...}
    toString(){...}
    getName(){...}   
  }             
  ```

+ 在类的实例上调用方法，其实就是调用原型上的方法

  ```javascript
  class A{}
  let a = new A();
  a.toString() === a.prototype.toString() // true
  ```

  

+ 类的方法（除了constructor之外），都定义在 prototype 对象上，所以类的新方法可以添加在 prototype 对象上，Object.assign() 方法可以很方便的一次性向类添加多个方法

  ```javascript
  class Animal{
    constructor(){...} 
  }
  Object.assign(Animal.prototype,{
    getName(){...}
  })    
  ```

  

+ 类的内部定义的所有方法都是不可枚举的

+ 类的调用必须要使用 new 命令，否则会报错

+ 类 不存在变量提升

+ this指向：类的方法内部如果含有this，它将默认指向类的实例，如果将该方法提出来单独使用，this指向的是该方法运行时所在的环境

+ 如何解决类方法中的 this 指向问题：

  + 在构造函数中绑定this
  + 使用箭头函数，<font style="color:blue">箭头函数的this，默认指向定义它时，所处上下文的对象的this指向</font>

---

##### constructor关键字

一个类必须要有`constructor`方法，如果没有显示地定义，一个空的constructor方法会被默认添加

```javascript
class Animal{}
// 等价于
class Animal{
    constructor(){}
}
```



实例的属性除非显示地定义在其本身，即this对象上，否则都是定义在原型对象上，类的所有实例共享一个原型对象

```javascript
class Person{
  // 自身属性
  constructor(name,age){
      this.name = name;
      this.age = age
  }
  // 原型对象上的属性
  say(){
    return 'My name is ' + this.name + ', I am ' + this.age + ' years old';    
   }  
}
```

---

##### static关键字

如果在一个方法前加上 static 关键字，就表示该方法不会被实例继承，而是直接通过类去调用，**称为静态方法**

```javascript
class Foo{
   static sayHello(){
       return 'hello'
   } 
}
Foo.sayHello();

var f = new Foo();
f.sayHello(); // 报错
```



父类的静态方法可以被子类继承

```javascript
class Foo{
    static sayHello(){
        console.log('hello')
    }
}
class Bar extends Foo{}
Bar.sayHello(); // hello
```



静态方法也可以从super对象上调用，super作为对象使用时，在普通方法中，指向父类的原型对象，在静态方法中，指向父类

```javascript
class Foo{
  static classMethod(){
    return 'hello'
  }  
}
class Bar extends Foo{
  static classMethod(){
    return super.classMethod() + ', too'; // 指向父类
  }  
}
```

---

##### extends关键字

在ES5中，处理原型的继承很麻烦

```javascript
function Monkey(type,name){
   Animal.apply(this, [type, name]); 
}
Monkey.prototype = Object.create(Animal.prototype,{
  toString:function(){}
})
Monkey.prototype.constrcutor = Monkey;
```



在ES6中，使用extends关键字实现原型继承

```javascript
class Monkey extends Animal{
    constructor(type,name){
        super(type,name) // super作为函数调用时，代表父类的构造函数
    }
    toString() {
      return  "monkey is" `${this.type}` ",name is "`{ this.name}`;
    }
}
```

---

##### super关键字

1. 当你想在子类中调用父类的函数时，super关键字就很有作用了：

   + 使用 super 关键字的时候，必须显示地指定是作为函数还是作为对象使用，否则会报错

   + <font style="color:red">`super作为函数使用时，表示父类的构造函数。ES6规定，子类的构造函数必须执行一次super函数</font>

   + super作为对象使用，在普通方法中，指向父类的原型对象；在静态方法中，指向父类

     ```javascript
     class Parent{
         static myMethod(msg){
             console.log("static")
         }
         myMethod(){
             console.log("instance")
         }
     }
     
     class Child extends Parent{
         static myMethod(){
             super.myMethod; // 静态方法中，指向父类
         }
          myMethod(){
            super.myMethod(); // 普通方法中，指向父类的原型对象
        }
     }
     
     Child.myMethod();// static
     var child = new Child();
     child.myMethod(); // instance
     ```

     

2. 通过super调用父类的方法时，super会绑定子类的this，React中类组件constructor的绑定

   ```javascript
   class A{
     constructor(){
       this.x = 1
     } 
      print(){
       console.log(this.x)  
     }
   }
   
   class B extends A{
     constructor(){
       super();
       this.x = 2;
     }  
     m(){
       console.log(this.x)     
     }
   }
   let b = new B();
   b.m() ;// 2
   ```

   

3. 由于绑定子类的this，因此通过super对某个属性赋值，这个super就是this，赋值的属性会变成子类实例的属性

   ```javascript
   class A{
     constructor(){
       this.x = 1
     }  
   }
   class B extends A{
     constructor(){
       super(); //
       this.x = 2;
      super.x = 3;
      console.log(super.x); // undefined 因为super代表的是父类的原型对象
      console.log(this.x); // 3 
     }  
   }
   ```