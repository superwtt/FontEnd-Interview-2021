#### Reflect

ES6 提供了一个全新的API-Reflect，Reflect 和 Proxy是相对的，我们可以用 Reflect 操作对象

---

##### Reflect存在的意义

1. 将 Object 对象一些内部的方法，放到 Reflect 对象上，比如 `Object.defineProperty`

> 现阶段这些方法存在于 Object 和 Reflect 对象上，未来只存在于 Reflect 对象上



2. 操作对象时出现报错返回 false

   比如 `Object.defineProperty(obj,name,desc)`在无法定义属性时，会抛出一个错误，而`Reflect.defineProperty(obj,name,desc)`则会返回false，这样会更加合理一些。

   ```javascript
   // 旧写法
   try {
      Object.defineProperty(target, property, attributes);
   }.catch(err){
       // failure
   }
   
   
   // 新写法
   if(Reflect.defineProperty(target,property,attributes)){
       // success
   } else{
       // failure
   }
   ```

   

3. 让操作对象的编程变为函数式编程

   老写法有的是命令式编程，比如下面的例子

   ```javascript
   // 老写法
   "assign" in Object; // true
   
   // 新写法
   Reflect.has(Object,"assign"); // true
   ```

   

4. 保持和 Proxy 对象的方法一一对应

   ```javascript
   Proxy(target, {
     set: function (target, name, value, receiver) {
       var success = Reflect.set(target, name, value, receiver);
       if (success) {
         console.log("property" + name + "on" + target + "set do " + value);
       }
       return success;
     },
   });
   ```

   这就让 Proxy 对象可以方便地调用对应的 Reflect 方法，完成默认行为，作为修改行为的基础。也就是说，不管 Proxy 怎么修改默认行为，你总可以在 Reflect 上获取默认行为

---

##### 综上所述

1. 从Reflect对象上可以拿到语言内部的方法
2. 操作对象出现报错时返回false
3. 让操作对象都变成函数式编程
4. 保持和 proxy对象的方法一一对应

---

##### Reflect常用API

Reflect 一共有13个静态方法

1. `Reflect.defineProperty(target,name,desc)`与`Object.defineProperty(target,name,desc)`类似，当时对象无法定义时，`Object.defineProperty`会报错，而`Reflect.defineProperty`不会，它会返回 false，成功时返回 true，如果不是对象还是会报错
2. `Reflect.getPrototypeOf(target)` 与 `Object.getPrototypeOf` 一样，返回指定的对象的原型。
3. `Reflect.setProtoTypeOf(target,prototype)` 和 `Object.setPrototypeOf(target,prototype)` 一样，它将指定对象的原型设置为另一个对象
4. `Reflect.getOwnPropertyDescriptor(target,name)`和 `Object.getOwnPropertyDescriptor(target,name)` 一样，如果在对象中存在，则返回给定的属性的属性描述符
5. `Reflect.isExtensible(target)` 与 `Object.isExtensible(target)` 类似，判断一个对象是否可扩展(是否可以在它上面添加新的属性)，它们的不同点是，当参数不是对象时(原始值),Object 将它强制转换为一个对象，Reflect 直接报错
6. `Reflect.preventExtensions(target)` 与 `Object.preventExtensions(target)` 类似，阻止新属性添加到对象，不同点和上一条一样
7. `Reflect.apply(target,thisArg,args)`与 `Function.prototype.apply.call(fn,obj,args)` 一样
8. `Reflect.ownKeys(target)` 与 `Object.getOwnPropertyNames(target).concat(Object.getOwnPropertySymbols(target))` 一样，返回一个包含所有自身属性(不包含继承属性)的数组
9. `Reflect.has(target,name)` 与 `in` 操作符一样，让判断操作都变成函数行为
10. `Reflect.deleteProperty(target,name)` 与 `delete` 操作符一样，让删除操作变成函数行为，返回布尔值代表成功或者失败
11. `Reflect.construct(target,args[,newTarget]) `与 `new `操作符一样，target 构造函数，第二参数是构造函数的参数类数组，第三个是 `new.target` 的值
12. `Reflect.get(target,name[,receiver])` 与 `Obj[key]` 一样,第三个参数是当要取值的 key 部署了 getter 时，访问其函数的 this 绑定为 receiver 对象
13. `Reflect.set(target,name,value[,receiver])` 设置 target 对象的 key 属性等于 value，第三个参数和 set 一样。返回一个布尔值



---

#### Proxy

Proxy 就像在目标对象之间的一个代理，任何对目标的操作都要经过代理，代理就可以对外界的操作进行过滤和改写



语法：

Proxy是构造函数，它有两个参数 `target` 和 `handler`。

+ `target`：是Proxy包装的目标对象
+ `handler`：是一个对象，其属性是当执行一个操作时定义代理的行为函数

```javascript
var obj = new Proxy(
  {},
  {
    get: function (target, key, receiver) {
      console.log(`getting ${key}`);
      return Reflect.get(target, key, receiver);
    },
    set: function (target, key, value, receiver) {
      console.log(`setting ${key}`);
      return Reflect.set(target, key, value, receiver);
    },
  }
);
obj.count = 1; // setting count
++obj.count; // getting count setting count 2
```

