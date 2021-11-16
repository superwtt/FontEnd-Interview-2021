#### Async/await

ES2017 标准引入了 async 函数，使得异步操作变得更加的方便

---

#### 特点

```javascript
const asyncReadFile = async function () {
  const f1 = await readFile('/etc/fstab');
  const f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};

// 执行
asyncReadFile();
```

解析：

1. 内置执行器：`Generator`函数的执行必须依靠执行器，也就是说，`async`函数的执行，与普通函数一模一样，只要一行；完全不像`Generator`函数，需要调用`next`方法才能真正执行得到结果

2. 更好的语义：<font style="color:red">`async `</font> 和 <font style="color:red">`await`</font>比起<font style="color:red">`*`</font> 和 <font style="color:red">`yield`</font>，语义更加清楚了。<font style="color:red">`async `</font> 表示函数里面有异步操作，<font style="color:red">`await`</font>表示紧跟在后面的表达式需要等待结果

3. 更广阔的适用性

4. <font style="color:red">**返回值是 Promise：async 函数的返回值是 Promise 对象**</font>，你可以用 then方法 指定下一步的操作，这比Generator函数的返回值是Ierator对象方便多了

   ```javascript
   // async函数返回的是Promise对象
   async function getStockPriceByName(name) {
     const symbol = await getStockSymbol(name);
     const stockPrice = await getStockPrice(symbol);
     return stockPrice;
   }
   getStockPriceByName('goog').then(function (result) {
     console.log(result);
   });
   ```

   

---

#### 语法

1. 返回 Promise 对象：

   + <font style="color:blue">**async函数返回一个 Promise对象** </font>，async函数 内部return 语句的返回值，会成为 then方法回调函数的参数

   ```javascript
   // async函数返回一个promise对象
   async function f(){
     return 'hello world'
   }
   f().then(res=>{
      console.log(res); // 'hello world' 
   })
   ```

   

2. Promise对象的状态变化：

   + async函数返回的 Promise对象，必须等到内部所有的`await`命里后面的Promise对象执行完，才会发生状态的改变
   + 也就是说，只有 `async`函数内部的异步操作执行完，才会执行 then方法指定的回调函数

   ```javascript
   async function getTitle(url) {
     let response = await fetch(url);
     let html = await response.text();
     return html.match(/<title>([\s\S]+)<\/title>/i)[1];
   }
   getTitle('https://tc39.github.io/ecma262/').then(console.log)
   // "ECMAScript 2017 Language Specification"
   ```

   解析：

   + 函数getTitle 内部有三个操作：抓取网页、取出文本、匹配页面标题，只有这三个操作全部完成，才会执行 then 方法里面的 console.log

3. await命令：

   + 正常情况下，await命令后面是一个Promise对象，返回该对象的结果，如果不是Promise对象，就直接返回对应的值

     ```javascript
     async function f() {
       // 等同于
       // return 123;
       return await 123;
     }
     
     f().then(v => console.log(v))
     // 123
     ```

     

4. 错误处理：如果 await后面的异步操作出错，那么等同于async函数返回的 Promise 对象被reject

   ```javascript
   async function f() {
     await new Promise(function (resolve, reject) {
       throw new Error('出错了');
     });
   }
   
   f()
   .then(v => console.log(v))
   .catch(e => console.log(e))
   // Error：出错了
   ```

   解析：

   + async函数 f 执行后，await 后面的 Promise对象会抛出一个错误对象，导致 catch方法 的回调函数被调用。

   + await命令后面的Promise对象，运行结果可能是rejected，所以最好把await命令放在`try...catch`代码块中

     ```javascript
     async function f() {
       try {
         await new Promise(function (resolve, reject) {
           throw new Error('出错了');
         });
       } catch(e) {
       }
       return await('hello world');
     }
     ```

