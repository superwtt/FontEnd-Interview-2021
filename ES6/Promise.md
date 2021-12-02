1. 为什么能够使用.then
2. resolve的值为什么then回调能拿到
3. 错误的回调机制怎么做
4. new Promise是什么意思，构造函数内部的this表示什么意思
5. all race



#### Promise

异步编程的一种解决方案，比传统的解决方案（回调函数）更加合理和强大。

以往我们如果处理多层异步操作，往往会像下面那样编写代码：

```javascript
doSomething(function(result) {
  doSomethingElse(result, function(newResult) {
    doThirdThing(newResult, function(finalResult) {
      console.log('得到最终结果: ' + finalResult);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);
```

<font style="color:red">**这就是经典的回调地狱**</font>

通过`Promise`的改写上面的代码

```javascript
doSomething().then(function(result) {
  return doSomethingElse(result);
})
.then(function(newResult) {
  return doThirdThing(newResult);
})
.then(function(finalResult) {
  console.log('得到最终结果: ' + finalResult);
})
.catch(failureCallback);
```

瞬间感受到`Promise`解决异步操作的优点：

+ 链式操作降低了编码难度
+ 代码可读性明显增强

----

#### Promise的状态

Promise对象仅有三种状态：

+ `pending`：进行中
+ `fulfilled`：已成功
+ `rejected`：已失败

特点：

+ 对象的状态不受外界的影响，只有异步操作的结果，可以决定当前是哪一种状态
+ 一旦状态改变（从`pending`变为`fulfilled`和从`pending`变为`rejected`），就不会再变，任何时候都可以得到这个结果

---

#### Promise的用法

`Promise`对象是一个构造函数，用来生成`Promise`实例

`Promise`构造函数接收一个函数作为参数，该函数的两个参数分别为`resolve`和`reject`

+ `resolve`，将`Promise`对象的状态从未完成变成成功
+ `reject`，将`Promise`对象的状态从未完成变成失败

```javascript
const promise = new Promise(function(resolve,reject){});
```

---

#### 实例方法

`Promise`构建出来的实例存在以下方法：

+ `then()`
+ `catch()`
+ `finally()`

1. `then` ：实例状态发生改变时的回调函数，第一个参数是`resolved`状态的回调函数，第二个参数是`rejected`状态的回调函数

   <font style="color:red">**`then()`方法返回的是一个新的`Promise`实例，也就是`Promise`能链式调用的原因**</font>

   ```javascript
   getJSON("/posts.json").then(function(json) {
     return json.post;
   }).then(function(post) {
     // ...
   });
   ```

   

2. `catch`：`.then(null,rejection)`或`.then(undefined,rejection)`的别名，用于指定发生错误时的回调函数

   ```javascript
   getJSON('/posts.json').then(function(posts) {
     // ...
   }).catch(function(error) {
     // 处理 getJSON 和 前一个回调函数运行时发生的错误
     console.log('发生错误！', error);
   });
   ```

   

3. `finally`：用于指定不管`Promise`对象最后状态如何，都会执行的操作

   ```javascript
   promise
   .then(result => {···})
   .catch(error => {···})
   .finally(() => {···});
   ```

---

#### 构造函数方法

`Promise`构造函数存在以下方法：

+ `all()`：有一个失败就会进catch
+ `race()`
+ `resolve()`
+ `reject()`
+ `try()`
+ `allSettled()`：不管成功还是失败 都能执行回调函数



1. `all()`：`Promise.all()`方法用于将多个`Promise实例`，包装成一个新的`Promise实例`。如果其中一个失败了，整个promise链就会跳出

   ```javascript
   const p = Promise.all([p1,p2,p3])
   ```

   实例 `p` 的状态由`p1`、`p2`、`p3`决定，分为两种：

   + 只有 p1 p2 p3的状态都变成`fullfilled`，p的状态才会变成`fullfilled`，此时 p1 p2 p3的返回值组成一个数组，传递给 p 函数
   + 只要 p1 p2 p3之中有一个被`rejected`，p的状态就变成`rejected`，此时第一个被`reject`的实例的返回值会传递给 p 的回调函数

2. `race()`：同样是将多个`Promise`实例，包装成一个`Promise`实例

   ```javascript
   const p = Promise.race([p1,p2,p3])
   ```

   只要 p1 p2 p3之中有一个实例率先改变状态，p 的状态就跟着改变 

3. `resolve()`

4. `reject()`

5. `try()`

---

#### 手动实现

1. 基本构成：

   + Promise 存在三种状态：`Pending`、`Resolve`、`Reject`；
   + 成功时，不可转为其他状态，且必须有一个不可改变的值（value）
   + 失败时，不可转为其他状态，且必须有一个不可改变的原因（reason）
   + `new Promise((resolve,reject)=>{resolve(value)})` resolve为成功，接收参数value，状态改变为 fulfilled，不可再次改变

   + `new Promise((resolve,reject)=>{reject(reason)})` reject为失败，接收参数reason，状态改变为rejected，不可再次改变
   + <font style="color:red">**传递给`new Promise`的参数，是一个函数，表示需要确保该函数执行完了，再继续执行 `then` 的回调**</font>

   以上信息可以转化为下面的代码：

   ```javascript
   const PENDING = 'pending'
   const RESOLVED = 'resolved'
   const REJECT = 'rejected'
   
   // 不理解的话把 Promise 看成 AAA
   // class AAA
   // 我们的目的是，先把传入构造函数AAA的回调执行一下
   class Promise{
      constructor(executor){
        this.status = PENDING;// 初始化state为等待
        this.value = null; // 成功的值
        this.reason = null;  // 失败的原因
      
      // resolve用来改变AAA状态为成功，返回结果    
      let resolve = value=>{
        // resolve调用后 status状态就会改变为成功的状态  
        if(this.status === PENDING ){
           this.status = RESOLVED
           this.value = value // 储存成功的值  
        }
      }
      // reject用来改变AAA状态为失败，返回失败原因
      let reject = reason=>{
        // reject调用后 status状态就会变成失败  
        if(this.status === REJECT){
           this.status = REJECTED;
           this.reason = reason; // 失败的理由
        }
      } 
      
      // excutor用来执行回调，如果excutor执行出错，直接执行reject
      try{
         executor(resolve, reject); 
      }catch(err){
         reject(err) 
      }
     }
   }
   ```

   

2. <font style="color:red">`then`</font>方法有两个参数：`onFulfilled`和`onRejected`，成功有成功的值，失败有失败的原因

   + 当状态 status 为 fulfilled，则执行 onFulfilled，传入this.value。当状态为 rejected，则执行onRejected，传入 this.reason

   ```javascript
   class Promise {
     constructor(executor) {}
   
     then(onFulfilled, onRejected) {
       if (this.status === RESOLVED) {
         onFulfilled(this.value) // 状态为成功，执行onFulfilled，传入成功的值
       }
       if (this.status === REJECT) {
          onRejected(this.reason);// 状态为失败，执行onRejected，传入失败的原因
       }
     }
   }
   
   ```

   

3. 当我调用 then 函数时，Promise 的状态有可能还是pending状态，这时候需要将 then函数 的两个参数进行保存，等状态改变时再进行调用，then函数 有可能调用多次，所以我们将成功和失败存到各自的数组，一旦reject或者resolve，就调用他们

   ```javascript
   // 多个then的情况
   let p = new Promise();
   p.then()
   p.then()
   ```

   ```javascript
   class Promise{
      constructor(excutor){
        this.status = 'pending'
        this.value = null;
        this.reason = null;
          
        this.onResolvedCallbacks = []; // 成功存放的数组
        this.onRejectedCallbacks = []; // 失败存放的数组
        
        let resolve = value => {// 这个value从真实调用的时候获取
           if(this.status === 'pending'){
              this.status = 'fulfilled'; 
              this.value = value; 
              
               // 一旦resolve执行，调用成功数组的函数
             this.onResolvedCallbacks.forEach(fn=>fn()) 
           } 
        }   
        
        let reject = reason => {
           if(this.status === 'pending'){
              this.status = 'rejected';
              this.reason = reason;
              // 一旦reject执行，调用失败数组的函数
              this.onRejectedCallbacks.forEach(fn=>fn()); 
           } 
        }
        try{
           executor(resolve, reject); 
        }catch (err) {
         reject(err);
       }
      } 
       then(onFulfilled,onReject){
         if(this.status === 'fulfilled'){
            onFulfilled(this.value); 
         }
         if (this.state === 'rejected') {
            onRejected(this.reason);
         }; 
         // 当状态status为pending时
         if (this.status === 'pending') {
         // onFulfilled传入到成功数组
         this.onResolvedCallbacks.push(()=>{
           onFulfilled(this.value);
         })
         // onRejected传入到失败数组
         this.onRejectedCallbacks.push(()=>{
           onRejected(this.reason);
         })
       }
       }
   }
   ```

   

4. 到这里，一个最基本的 Promise就可以使用了

   ```javascript
   new Promise((resolve,reject)=>{
     setTimeout(()=>{
        resolve(1) 
     },500)  
   }).then(res=>{
      console.log(res) 
   })
   ```

    

5. 实现链式调用

   + 链式调用也是 Promise 的重点所在，因为有了链式调用，才能避免回调地狱的问题，`new Promise().then().then()`

   + 为了达成链式，我们默认在第一个<font style="color:red">`then`</font>里面返回一个promise，称为promise2：`promise2 = new Promise((resolve,reject)=>{})`；所以原理就是<font style="color:red">**`then`**函数返回一个新的 Promise </font>
     + 将这个 promise2 返回的值传递到下一个 then 中
     + 如果返回一个普通的值，则将普通的值传递给 下一个 then
   + 当我们在一个 then 中return了一个参数，参数未知，需判断，这个return出来新的promise就是onFulfilled()或者onRejected()的值
   + 规定onFulfilled()或者onRejected()的值即第一个then返回的值叫做x，判断x的函数叫做`resolvePromise`
     + 首先要看看 x 是不是promise
     + 如果是，则取它的结果作为新的promise2成功的结果
     + 如果是普通值，直接作为 promise2 成功的结果
     + 所以要比较 x 和 promise2
     + resolvePromise的参数有promise2（默认返回的promise）、x（我们自己`return`的对象）、resolve、reject
     + resolve和reject是promise2的

   ```javascript
   // then函数修改如下
   class Promise {
      then(onFulfilled,onRejected){
        let promise2 = new Promise((resolve, reject)=>{
         if (this.state === 'fulfilled') {
           let x = onFulfilled(this.value);
           // resolvePromise函数，处理自己return的promise和默认的promise2的关系
           resolvePromise(promise2, x, resolve, reject);
         };
         if (this.state === 'rejected') {
           let x = onRejected(this.reason);
           resolvePromise(promise2, x, resolve, reject);
         };
         if (this.state === 'pending') {
           this.onResolvedCallbacks.push(()=>{
             let x = onFulfilled(this.value);
             resolvePromise(promise2, x, resolve, reject);
           })
           this.onRejectedCallbacks.push(()=>{
             let x = onRejected(this.reason);
             resolvePromise(promise2, x, resolve, reject);
           })
         }
       });
        return promise2
      } 
   }
   ```

   

6. 实现 resolvePromise 函数：

   + x 不能是null
   + x是普通值，直接 resolve(x)
   + x是对象或者函数（包括promise），`let then = x.then`
   + 成功或者失败只能调用一个，所以设置一个called来防止多次调用

   ```javascript
   function resolvePromise(promise2, x, resolve, reject){
     // 循环引用报错
     if(x === promise2){
       // reject报错
       return reject(new TypeError('Chaining cycle detected for promise'));
     }
     // 防止多次调用
     let called;
     // x不是null 且x是对象或者函数
     if (x != null && (typeof x === 'object' || typeof x === 'function')) {
       try {
         // A+规定，声明then = x的then方法
         let then = x.then;
         // 如果then是函数，就默认是promise了
         if (typeof then === 'function') { 
           // 就让then执行 第一个参数是this   后面是成功的回调 和 失败的回调
           then.call(x, y => {
             // 成功和失败只能调用一个
             if (called) return;
             called = true;
             // resolve的结果依旧是promise 那就继续解析
             resolvePromise(promise2, y, resolve, reject);
           }, err => {
             // 成功和失败只能调用一个
             if (called) return;
             called = true;
             reject(err);// 失败了就失败了
           })
         } else {
           resolve(x); // 直接成功即可
         }
       } catch (e) {
         // 也属于失败
         if (called) return;
         called = true;
         // 取then出错了那就不要在继续执行了
         reject(e); 
       }
     } else {
       resolve(x);
     }
   }
   ```

   

7. 其他方法的实现

   ```javascript
   class Promise{
      static race(promises){
         return new Promise((resolve,reject)=>{
            promises.map(promise => {
               promise.then(resolve, reject);
             }); 
         }) 
      } 
       static all(promises) {
       let arr = [];
       let i = 0;
       return new Promise((resolve, reject) => {
         promises.map((promise, index) => {
           promise.then(data => {
             arr[index] = data;
             if (++i === promises.length) {
               resolve(arr);
             }
           }, reject);
         })
       })
     }
   }
   ```

8. 大功告成

   ```javascript
   class Promise{
     constructor(executor){
       this.state = 'pending';
       this.value = undefined;
       this.reason = undefined;
       this.onResolvedCallbacks = [];
       this.onRejectedCallbacks = [];
       let resolve = value => {
         if (this.state === 'pending') {
           this.state = 'fulfilled';
           this.value = value;
           this.onResolvedCallbacks.forEach(fn=>fn());
         }
       };
       let reject = reason => {
         if (this.state === 'pending') {
           this.state = 'rejected';
           this.reason = reason;
           this.onRejectedCallbacks.forEach(fn=>fn());
         }
       };
       try{
         executor(resolve, reject);
       } catch (err) {
         reject(err);
       }
     }
     then(onFulfilled,onRejected) {
       onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
       onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
       let promise2 = new Promise((resolve, reject) => {
         if (this.state === 'fulfilled') {
           setTimeout(() => {
             try {
               let x = onFulfilled(this.value);
               resolvePromise(promise2, x, resolve, reject);
             } catch (e) {
               reject(e);
             }
           }, 0);
         };
         if (this.state === 'rejected') {
           setTimeout(() => {
             try {
               let x = onRejected(this.reason);
               resolvePromise(promise2, x, resolve, reject);
             } catch (e) {
               reject(e);
             }
           }, 0);
         };
         if (this.state === 'pending') {
           this.onResolvedCallbacks.push(() => {
             setTimeout(() => {
               try {
                 let x = onFulfilled(this.value);
                 resolvePromise(promise2, x, resolve, reject);
               } catch (e) {
                 reject(e);
               }
             }, 0);
           });
           this.onRejectedCallbacks.push(() => {
             setTimeout(() => {
               try {
                 let x = onRejected(this.reason);
                 resolvePromise(promise2, x, resolve, reject);
               } catch (e) {
                 reject(e);
               }
             }, 0)
           });
         };
       });
       return promise2;
     }
     catch(fn){
       return this.then(null,fn);
     }
   }
   function resolvePromise(promise2, x, resolve, reject){
     if(x === promise2){
       return reject(new TypeError('Chaining cycle detected for promise'));
     }
     let called;
     if (x != null && (typeof x === 'object' || typeof x === 'function')) {
       try {
         let then = x.then;
         if (typeof then === 'function') { 
           then.call(x, y => {
             if(called)return;
             called = true;
             resolvePromise(promise2, y, resolve, reject);
           }, err => {
             if(called)return;
             called = true;
             reject(err);
           })
         } else {
           resolve(x);
         }
       } catch (e) {
         if(called)return;
         called = true;
         reject(e); 
       }
     } else {
       resolve(x);
     }
   }
   //resolve方法
   Promise.resolve = function(val){
     return new Promise((resolve,reject)=>{
       resolve(val)
     });
   }
   //reject方法
   Promise.reject = function(val){
     return new Promise((resolve,reject)=>{
       reject(val)
     });
   }
   //race方法 
   Promise.race = function(promises){
     return new Promise((resolve,reject)=>{
       for(let i=0;i<promises.length;i++){
         promises[i].then(resolve,reject)
       };
     })
   }
   //all方法(获取所有的promise，都执行then，把结果放到数组，一起返回)
   Promise.all = function(promises){
     let arr = [];
     let i = 0;
     function processData(index,data){
       arr[index] = data;
       i++;
       if(i == promises.length){
         resolve(arr);
       };
     };
     return new Promise((resolve,reject)=>{
       for(let i=0;i<promises.length;i++){
         promises[i].then(data=>{
           processData(i,data);
         },reject);
       };
     });
   }
   ```

---

#### 参考链接

[VeryGood](https://developer.aliyun.com/article/613412)

[Good](https://juejin.cn/post/6844904094516150285)

