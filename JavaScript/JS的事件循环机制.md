#### JS的事件循环机制

1. 首先我们要知道

   + JavaScript是单线程语言
   + 事件循环机制EeentLoop是JavaScript的执行机制

2. JS的执行顺序

   + 同步和异步分别进入两个不同的执行场所，同步的进入主线程，异步的进入Event Table并注册回调函数

   + 同步和异步同时进行

   + 异步队列中，当指定的事情完成时，EventTable会把这个函数移入`EventQueue`

   + 在一次事件循环中，异步事件返回的结果，会被放入一个任务队列中，根据异步事件的类型，需要把事件放入到对应的微任务或宏任务队列

   + 当主线程空闲时，即执行栈为空，主线程会先查看微任务队列，执行清空之后再去查看宏任务队列，并执行清空

     ![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/js的事件循环机制.png)

3. 代码描述如下：

   ```javascript
   let data = [];
   $.ajax({
       url:'www.js.com',
       data:data,
       success:()=>{
          console.log("发送成功") 
       }
   })
   console.log("代码执行结束")
   ```

   解析：

   + js代码开始执行，ajax进入EventTable，并注册回调函数`success`
   + 执行`console.log("代码执行结束")`
   + ajax事件完成，回调函数`success`进入`EventQueue`
   + 主线程从`EventQueue`读取回调函数`success`并执行

---

#### 微任务和宏任务

1. 定义：微任务和宏任务都是异步任务，它们都属于一个队列，主要区别在于它们的执行顺序。**微任务先执行，等微任务全部执行完了才会继续执行宏任务**
2. 微任务、宏任务：
   + 宏任务有：script全局任务、`setTimeout`、`setInterval`、`setImmediate`、`I/O`、`UI rendering`
   + 微任务有：`Promise`、`process.nextTick`、`Object.observer`、`MutationObserver`等等

---

#### 关于setTimeout

1. `setTimeout`可以用来延时执行，我们经常这么实现延时3秒执行

   ```javascript
   setTimeout(()=>{
      console.log("延时3秒") 
   },3000)
   ```

2. `setTimeout`存在一个问题，有时候明明写的延时3秒，实际却是5、6秒才执行函数，这是怎么回事呢？

   ```javascript
   setTimeout(()=>{
      task() 
   },3000)
   sleep(10000000000000)
   ```

   控制台执行的结果发现**task()需要的时间远远超过3秒**  

   解析：

   + `sleep(10000000000000)`进入同步队列，`task()`进入异步队列。
   + 3秒过后，回调函数`task()`进入`EventQueue`，而主线程上的`sleep(10000000000000)`仍然在继续，所以`task()`只好等着
   +  `sleep(10000000000000)`执行完， `task()`终于从`EventQueue`进入主线程执行

   上述流程走完我们发现，`setTimeout`这个函数，经过指定的时间后，把要执行的任务（本例中为task()）加入到`EventQueue`中，又因为JS是单线程的，任务要一个一个去执行，<font style="color:red">如果前面的任务需要的时间太久，那么只能等着，导致真正的延时时间远远大于3秒</font>

3. `setTimeout(fn,0)`是什么意思？

   `setTimeout(fn,0)`的含义是，某个任务在主线程最早可得的空闲时间去执行，意思是不用再去等多少秒了，只要主线程执行栈内的同步任务全部执行完成，就立马执行

   ```javascript
   // 代码1
   console.log("先执行这里")
   setTimeout(() => {
       console.log('执行啦')
   },0);
   //先执行这里
   //执行啦
   
   //代码2
   console.log('先执行这里');
   setTimeout(() => {
       console.log('执行啦')
   },3000);  
   //先执行这里
   // ... 3s later
   // 执行啦
   ```

---

#### Promise与process.nextTick(callback)

process.nextTick是微任务



所有的宏任务先执行，执行完了之后再去任务队列里执行微任务，执行完所有的微任务之后，才会继续下一轮的事件循环



![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/%E5%AE%8F%E4%BB%BB%E5%8A%A1%E4%B8%8E%E5%BE%AE%E4%BB%BB%E5%8A%A1.png)



```javascript
 // 例题
 console.log('1');

setTimeout(function() {
    console.log('2');
    process.nextTick(function() {
        console.log('3');
    })
    new Promise(function(resolve) {
        console.log('4');
        resolve();
    }).then(function() {
        console.log('5')
    })
})
process.nextTick(function() {
    console.log('6');
})
new Promise(function(resolve) {
    console.log('7');
    resolve();
}).then(function() {
    console.log('8')
})

setTimeout(function() {
    console.log('9');
    process.nextTick(function() {
        console.log('10');
    })
    new Promise(function(resolve) {
        console.log('11');
        resolve();
    }).then(function() {
        console.log('12')
    })
})

作者：ssssyoki
链接：https://juejin.cn/post/6844903512845860872
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

输出结果：

1、7、6、8、2、4、3、5、9、11、10、12









































---

#### 例题

```javascript
console.log("script start")
let promise1 = new Promise(function (resolve){
  console.log("promise1")
  resolve()
  console.log("promise1 end")
}).then(function(){
  console.log("promise2")
})
setTimeout(function(){
   console.log("setTimeout")
})
console.log("script end")
// script start
// promise1
// promise1 end
// script end
// promise2
// settimeout
```

```javascript
async function async1(){
  console.log("async1 start");
  await async2();
  console.log("async1 end")  
}
async function async2(){
    console.log("async2")
}
console.log("script start")
setTimeout(function(){
  console.log("setTimeout")
},0)
async1();
new Promise(function(resolve){
  console.log("promise1");
  resolve();
}).then(function(){
  console.log("promise2")
})
console.log("script end");
// 执行结果
// script start
// async1 start
// async2
// promise1
// script end
// async1 end
// promise2
// setTimeout
```











