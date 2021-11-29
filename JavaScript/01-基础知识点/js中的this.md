#### this对象

函数的调用方式决定了 this 的值。

this关键字是函数运行时自动生成的一个内部对象，只能在函数内使用，总是指向调用它的对象

举个例子：

```javascript
function baz(){
  // 当前调用栈是 baz
  // 因此当前调用位置是全局作用域
  console.log("baz")
  bar(); // <-- bar的调用位置  
}
function bar(){
   // 当前调用栈是：baz --> bar
   // 因此当前调用位置在baz中
   console.log("bar");
   foo(); // <-- foo的调用位置 
}
function foo(){
   // 当前调用栈是：baz --> bar --> foo
    // 因此，当前调用位置在bar中 
   console.log("foo") 
}
baz(); // <-- baz的调用位置
```

---

#### 绑定规则

根据不同的使用场合，this有不同的值，主要分为下面几种：

1. 默认绑定
2. 隐式绑定
3. new 绑定
4. 显示绑定

---

##### 默认绑定

全局环境中定义 person函数 内部使用 this关键字

```javascript
var name = "Jenny";
function person(){
   return this.name 
}
console.log(person()); // Jenny
```

解析：上述代码输出 Jenny，原因是调用函数的对象在浏览器中位于 window，因此 this指向 window，所以输出 Jenny

---

##### 隐式绑定

1. 函数还可以作为某个对象的方法调用，这时 this 就指这个上级对象

   ```javascript
   function test(){
      console.log(this.x)
   }
   var obj = {};
   obj.x = 1;
   obj.m = test;
   obj.m(); // 1
   ```

   

2. 函数包含多个对象，<font color=red>**尽管这个函数是被最外层的对象所调用，this 指向的也只是它的上一级对象**</font>

   ```javascript
   var o = {
     a:10,
     b:{
       fn:function(){
         console.log(this.a);
       }
     }
   }
   o.b.fn(); // undefined
   ```

   上述代码中，this 的上一级对象为 b ，b内部并没有 a 变量的定义，所以输出 undefined

   

---

##### 显示修改

`apply()`、`call()`、`bind()`是函数的一个方法，作用是改变函数的调用对象等等。它的第一个参数就表示改变后的调用这个函数的对象。因此，this指的是就是这第一个参数

---

#### 箭头函数

在定义时就确定了





























