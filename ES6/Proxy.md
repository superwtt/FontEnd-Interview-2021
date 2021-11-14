### Proxy

---

#### 定义

proxy在目标对象的外层搭建了一层拦截，外界对目标对象的操作，必须要通过这层拦截

---

#### 语法

```javascript
var newProxy = new Proxy(target,handler);
// new Proxy()表示生成一个Proxy实例，target表示你要拦截的目标对象，handler也是一个对象，用来定制拦截行为。监听整个对象
// Object.defineProperty(target, key, sharedPropertyDefinition)
```

1. `handler`解析：

   + `get(target,proKey,receiver)`：拦截对象属性的读取

   + `set(target,proKey,value,receiver)`：拦截对象属性的设置

   + `has(target,proKey)`：拦截`proKey in proxy`的操作，返回一个布尔值

   + `deleteProperty(target,proKey)`：拦截`delete proxy[propKey]`的操作，返回一个布尔值

   + ownKeys(target)：拦截`Object.keys(proxy)`、`for...in`等循环，返回一个数组

   + getOwnPropertyDescriptor(target, propKey)：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象

   + defineProperty(target, propKey, propDesc)：拦截`Object.defineProperty(proxy, propKey, propDesc）`，返回一个布尔值

   + preventExtensions(target)：拦截`Object.preventExtensions(proxy)`，返回一个布尔值

   + getPrototypeOf(target)：拦截`Object.getPrototypeOf(proxy)`，返回一个对象

   + isExtensible(target)：拦截`Object.isExtensible(proxy)`，返回一个布尔值

   + setPrototypeOf(target, proto)：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值

   + apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作

   + construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作

     ```javascript
     var person = {
       name: "张三"
     };
     
     var proxy = new Proxy(person, {
       get: function(target, propKey) {
         return Reflect.get(target,propKey)
       }
     });
     
     proxy.name // "张三"
     ```

---

#### Proxy和Object.defineProperty语法解析

1. `Object.defineProperty`

   ```javascript
   Object.defineProperty(data,'count',{
      get(){},
      set(){} 
   })
   ```

   <font style="color:red">必须要事先知道拦截的key是什么，这也就是为什么Vue2.0中无法监听到新增属性变化的原因</font>

2. `Proxy`

   ```javascript
   new Proxy(data,{
       get(key){},
       set(key,value){}
   })
   ```

   <font style="color:blue">可以看到，根本不需要关心具体的key，它拦截的是修改data上的任意key和读取data上的任意key。所以不管是已有的key还是新增的key都能监测到</font>

   Proxy是外界访问该数据的时候设置的一层代理，外界对数据的任何操作必须要通过这层拦截。

---

### Proxy VS Object.defineProperty

1. `Object.defineProperty`无法一次性监听对象所有属性，必须通过遍历或者递归来实现。而`Proxy`中，如果有多层属性嵌套的话，只有访问某个属性的时候才会递归处理下一级的属性

   ```javascript
   let girl = {
       name:"marry",
       age:22
   }
   // Proxy监听整个对象
   girl = new Proxy(girl,{
       get(){}
       set(){}
   })
   // Object.defineProperty
   Object.keys(girl).forEach(key=>{
       Object.defineProperty(girl,key,{
           if(Object.isObject(key)){
             Object.defineProperty(key,property,{
               set(){}
               get(){}
             }
           }
       })
   })
   ```

   ---

2. `Object.defineProperty`无法监听新增的属性

   + Proxy可以监听到新增加的属性，而Object.defineProperty不可以，需要自己手动再去添加一次监听，因此在Vue中想动态监听属性，一般使用Vue.set

     ```javascript
     let girl = {
         name:"marry",
         age:22
     }
     // Proxy监听整个对象
     girl = new Proxy(girl,{
         get(){}
         set(){}
     })
      // Object.defineProperty
        Object.keys(girl).forEach(key => {
          Object.defineProperty(girl, key, { // 必须要知道拦截的key是什么，所以无法监听到新增的属性
            set() {},
            get() {}
         })
      }); 
     // Proxy生效，Object.defineProperty不生效
     girl.hobby = "game"
     ```

   ---

3. `Object.defineProperty`无法响应数组的操作

   + Object.defineProperty可以监听数组的变化，但是对push、shift、pop、unshift进行响应，Vue中通过重写Array上的方法实现监听

     ```javascript
     const arrayProto = Array.prototype;
     const arrayMethods = Object.create(arrayProto)
     ['push','pop','shift','unshift','splice','sort','reverse'].forEach(method=>{
       const original = arrayProto[method];
       Object.defineProperty(arrayMethods,methods,{
         value: function mutaor(...args){
           return original.apply(this,args);
         },
         enumerable: false,
         writable: true,
         configurable: true
       })
     })
     ```

   + 对于新增的数组项，`Object.defineProperty`无法监听到

     ```javascript
  const arr = [1,2];
     // Proxy监听数组
     arr = new Proxy(arr,{
         get(){}
         set(){}
     })
     // Object.defineProperty
     arr.forEach((item,index)=>{
        Object.defineProperty(arr,`${index}`,{
            set(){}
            get(){}
        }) 
     })
     arr[0] = 10; // 都生效
     arr[3] = 10; // 只有Proxy生效
     arr.push(10); // 只有Proxy生效
     
     // Vue做法
     const methods = ['pop', 'shift', 'unshift', 'sort', 'reverse', 'splice', 'push'];
     Object.defineProperty(subArrProto, method, {
         set() {},
         get() {}
     })
     ```
   
   ---

4. Proxy拦截方法更多

   + Proxy中提供了13种拦截方法，包括get、set、apply、has、construct、deleteProperty、defineProperty等各种属性的操作，而Object.defineProperty只有get和set

5. Object.defineProperty兼容性更好

   + Proxy是新出的API兼容性不够好

