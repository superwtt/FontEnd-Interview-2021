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

+ `all()`
+ `race()`
+ `resolve()`
+ `reject()`
+ `try()`



1. `all()`：`Promise.all()`方法用于将多个`Promise实例`，包装成一个新的`Promise实例`

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

   + 平时使用`Promise`我们知道，Promise 存在三种状态：`Pending`、`Resolve`、`Reject`；在`new Promise`时需要传入一个函数，参数为<font style="color:red">`resolve`</font>和<font style="color:red">`reject`</font>；

   + 最重要的还有个<font style="color:red">`then`</font>方法，<font style="color:red">`then`</font>函数可以传入两个函数作为参数，第一个函数用来获取异步操作的结果，第二个函数用来获取错误的原因
   + 除此之外还需要<font style="color:red">`value`</font>和<font style="color:red">`reason`</font>存放Promise的结果或者错误原因

   以上信息可以转化为下面的代码：

   ```javascript
   const PENDING = 'pending'
   const RESOLVED = 'resolved'
   const REJECT = 'rejected'
   
   
   
   
   ```

   

2. de

3. de

4. de

5. de

6. de















