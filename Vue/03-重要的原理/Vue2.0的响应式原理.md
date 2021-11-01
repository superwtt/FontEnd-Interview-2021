#### Vue2.0的响应式原理

很多人把双向绑定等同于响应式，这是不对的，这两者是不一样的：

1. 双向绑定是model和view之间的绑定，是双向的
2. 响应式是一种基于订阅发布的技术，是单向的，可以用来实现前者
3. 响应式可以说是双向绑定的一半

---

##### Object.defineProperty

Vue的数据响应式原理是基于JS方法`Object.defineProperty()`来实现的，该方法不兼容IE8和FF22以下版本的浏览器，这也就是为什么Vue只能在这些版本之上的浏览器运行的原因。

---

##### 语法

`Object.defineProperty(obj,prop,description)`

其中，description是属性描述符，分为两种：

1. 数据描述：

   + value
   + writable
   + enumerable
   + configurable

2. 存取器描述：

   + get描述符：当访问属性时，会调用此函数
   + set描述符：当修改属性时，会调用此函数

3. 注意：在属性描述中，用了`get/set`属性来定义对应的方法，当使用了`get/set`之后，不允许使用`writable/value`

   ```javascript
   Object.defineProperty(obj,key,{
      get:function(){},
      set:function(){},
      configurable:true,
      enumerable:true
   })
   ```

   

一旦对象拥有了`getter`/`setter`方法，我们可以简单将该对象称为响应式对象。这两个操作符为我们提供了拦截数据进行操作的可能性

---

##### 简单响应式的实现

1. `Dep`：被观察类，用来生成被观察者
2. `Watcher`：观察者类，用来生成观察者
3. `Observer`：将被观察者中的普通数据转为响应式数据，从而实现响应式对象

---

##### 具体实现

① Dep被观察者类

```javascript
class Dep {
   constructor(){
      this.subs = [] 
   } 
   addSub(watcher){
     this.subs.push(watcher)  
   } 
   notify(data){
      this.subs.forEach(sub=>sub.update(data)) 
   } 
}
```

---

② Watcher

```javascript
class Watcher{
   constructor(cb){
      this.cb = cb 
   } 
   update(data){
      this.cb(data) 
   }
}
```

---

③ Observer

```javascript
class Observer{
   constructor(){
       
   }
   defineReactive(vm,obj){
      for(let key in obj){
         let value = obj[key];
         let dep = new Dep();
         Object.defineProperty(obj,key,{
            configurable:true,
            enumerable:true,
            get(){
              let watcher = new Watcher(v=>vm.innerText = v) 
              dep.addSub(watcher);
              return value  
            }
            set(newValue){
              value = newValue;
              dep.notify(newValue);
            } 
         }) 
      } 
   }
}
```

---

##### 参考链接

[Good](https://segmentfault.com/a/1190000038921922)

---

##### 总结

1. Vue的响应式基于一种发布-订阅的技术：

   + 使用`Obect.defineProperty`去拦截数据，添加get/set描述符，将数据变成响应式对象
   + 有一个被观察的`Dep`类，有一个观察者的`Watcher`类，还有一个将被观察者的数据变成响应式的类`Observer`类。当被观察者数据变化，能够通知到所有的观察者去更新自己的数据，即实现了响应式

   































