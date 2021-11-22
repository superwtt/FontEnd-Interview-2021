#### Tree Shakig

`Tree-Shaking`是一种基于ES6 Module规范的Dead Code Elimaination 技术，它会在运行过程中静态分析模块之间的导入导出，去除无用的代码，减少打包体积

---

#### Dead code

Dead code 也叫死码，无用代码，主要包含了以下两点：

1. 不会被运行到的代码

   ```javascript
   // 不会被运行到的代码
   function foo(){
     return 'foo';
     var bar = "bar"
   }
   
   if(0){
      // 这个条件判断语句块内的代码永远不会被执行 
   }
   ```

   

2. 只会影响到无关程序运行结果的变量

   ```javascript
   function add(a, b) {
     let c = 1; // unused variable 在这里可以被看作死码
     return a + b;
   }
   ```

   

---

#### Tree Shaking的原理

借助ES6模块语法的静态结构，通过编译阶段的静态分析，找到没有引入的模块并打上标记，然后在压缩阶段利用像`uglify-js`这样的压缩工具删除这些没有用到的代码即可，即

1. 找到没有引入的模块打上标记：`unused harmony export xxx`

2. 删除这些没有用到的语句

   ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64044294f29e449e9c6016e724a93fdd~tplv-k3u1fbpfcp-watermark.awebp)

---

#### 参考链接

[掘金](https://juejin.cn/post/6955383260759195678)

[掘金](https://juejin.cn/user/1820446985555544)