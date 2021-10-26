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

---

### Proxy VS Object.defineProperty

1. Object.defineProperty无法一次性监听对象所有属性，必须通过遍历或者递归来实现

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

   

2. Object.defineProperty无法监听新增的属性

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
          Object.defineProperty(girl, key, {
            set() {},
            get() {}
         })
      }); 
     // Proxy生效，Object.defineProperty不生效
     girl.hobby = "game"
     ```

3. Object.defineProperty无法响应数组的操作

   + Object.defineProperty可以监听数组的变化，但是对push、shift、pop、unshift进行响应，Vue中通过重写Array上的方法实现监听

   + 对于新增的数组项，Object.defineProperty无法监听到

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

4. Proxy拦截方法更多

   + Proxy中提供了13种拦截方法，包括get、set、apply、has、construct、deleteProperty、defineProperty等各种属性的操作，而Object.defineProperty只有get和set

5. Object.defineProperty兼容性更好

   + Proxy是新出的API兼容性不够好

