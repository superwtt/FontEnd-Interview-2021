#### Generator函数

Generator函数是ES6提供的一种异步编程解决方案，语法行为与传统函数完全不一样

之前提到的 `Promise`已经是一种比较流行的解决异步的方案，那么为什么还出现`Generator`甚至`async/await`呢？

---

#### Generator函数

1. `Generator`函数是一个普通函数，但是有两个特征：

+ `function`关键字与函数名之间有一个星号

+ 函数体内部使用 `yield` 表达式，定义不同的内部状态

  ```javascript
  function* helloWorld(){
      yield 'hello'
      yield 'world'
      return 'ending'
  }
  ```

  

2. 通过 `yield` 关键字可以暂停`Generator`函数返回的遍历器对象的状态

   ```javascript
   function* helloWorldGenerator() {
     yield 'hello';
     yield 'world';
     return 'ending';
   }
   var hw = helloWorldGenerator();
   ```

   解析：上述代码存在三个状态：hello、world、ending，通过`next()`方法才会遍历到下一个内部状态，其运行逻辑如下：

   + 遇到 `yield` 表达式 就暂停执行后面的操作，并将紧跟在 `yield` 表达式后面的那个表达式的值，作为返回的对象的 value 属性值

   + 下次调用 next 方法时，再继续往下执行，直到遇到下一个 yield 表达式

   + 如果没有遇到新的 yield 表达式，就一直运行到函数结束，直到 return 语句为止，并将 return 语句后面的表达式的值，作为返回的对象的 value 的值

     ```javascript
     hw.next() // {value:'hello',done:false}
     
     hw.next() // {value:'world',done:false}
     
     hw.next() // {value:'ending',done:true}
     
     hw.next() // {value:undefind,done:true}
     ```

     解析：

     + done 用来判断是否存在下个状态，value 对应状态的值

   



---

#### 异步解决方案

回顾之前开展的异步解决方案：

+ 回调函数
+ Promise 对象
+ Generator 函数
+ async / await

这里通过文件读取案例，将几种解决异步的方案进行一个比较：

1. 回调函数

   ```javascript
   fs.readFile('/etc/fstab',function(err,data){
       if(err) throw err
       console.log(data)
       fs.readFile('/etc/shells',function(err,data){
          if(err) throw err
          console.log(err) 
       })
   })
   ```

   

2. Promise：就是为了解决回调地狱而产生的，将回调函数的嵌套，改成链式调用

   ```javascript
   const fs = require('fs');
   const readFile = function(fileName){
      return new Promise(function(resolve,reject){
          fs.readFile(fileName,function(err,data){
              if (error) return reject(error);
              resolve(data);
          })
      }) 
   }
   readFile('/etc/fstab').then(data=>{
       console.log(data)
       return readFile('/etc/shells')
   }).then(data=>{
       console.log(data)
   })
   ```

   这种链式操作形式，使异步任务的两端执行更清楚了，但是也存在了很明显的问题，代码变得冗余了，语义化并不是很强

   

3. Generator函数：`yield`表达式可以暂停函数执行，`next`方法用于恢复函数执行

   ```javascript
   const gen =function* (){
      const f1 = yield readFile('/etc/fstab');
      const f2 = yield readFile('/etc/shells');
      console.log(f1.toString());
      console.log(f2.toString());
   }
   ```

    

4. async / await：讲上面的`Generator`函数改成了`async/await`形式，更加简洁，语义化更强了

   ```javascript
   const asyncReadFile = async function(){
     const f1 = await readFile('/etc/fstab');
     const f2 = await readFile('/etc/shells');
     console.log(f1.toString());
     console.log(f2.toString()); 
   }
   ```

---

#### 区别

1. `Promise`编写代码相比`Generator`、`async`更加复杂化，且可读性稍差
2. `Generator`、`async`将异步的代码 同步化表达了
3. `async`是`Generator`的语法糖，相当于会自动执行`Generator`函数
4. `async`使用上更加简洁，将异步代码以同步的形式进行编写，是处理异步编程的最终方案