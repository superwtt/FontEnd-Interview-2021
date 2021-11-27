### 说说你对Vue的理解

最开始接触Vue的时候是2017年，那时候自己的项目还在用Jq+jsz，但是市场上已经很流行Vue了，所以自己就私下学习了一下，感觉特别惊艳：

1. 响应式编程：我无需再去频繁的操作DOM，直接操作数据就可以改变页面元素，相比于Jq频繁地addClass、removeClass简直是yyds
2. 组件化思想：我们的应用被拆分成不同的模块，每一个模块就是一个组件，每个模块又可以细分成不同的功能组件，相比于Jq一大长串的js代码比起来，降低了耦合度，代码维护起来也非常快
3. 指令系统：Vue中强大的指令系统，比如v-if、v-on、v-bind等，封装成指令直接调用就可以达到我们的目的，编程也更加轻松



现在Vue对前端而言可以说是一项必备的技能了，Vue4.0都出来了，对于自己而言，更加期待精细化的项目来提高自己对它的认知。

---

### 你总结的Vue的最佳实践是什么、Vue中的注意点、你开发过程中踩过的坑

1. 必要的：
   + data必须是一个函数
   + props尽量写的详细
   + v-for要设置键值
   + 避免v-for和v-if一起使用
   + 为组件的样式设置作用域
2. 强烈推荐的：
   + 每个组件单独成文件
   + 文件名字要统一，要么都是大写开头要么都是小写开头
   + template中的表达式要尽量简写
   + 指令要么都缩写要么都不缩写

---

### Vue项目中你遇到哪些坑？

1. 数据更新了，页面不渲染：
   + 如果是对象，把对象里每个属性都在data里声明一下，或者this.$set对象的属性，this.$forceUpdate强制更新
   + 如果是数组，监听不到下标的变化，可以使用this.$forceUpdate强制更新一下
   
2. 页面缓存，有个填写信息的页面，需要填写一部分信息，进入其他页面，再返回的时候，页面上填写的信息已经被销毁了，解决办法：

   + 使用Vue的 keep-alive 来完成页面的缓存

3. axios请求，post的方式数据一直传不到后端那边：

   + 原因：

     + 后端的注解使用了@RequestParam，意思是只能从请求的地址中获取参数，也就是只能从username=admin&password=admin这种字符串中解析出参数
     + post请求的两种传参方式data：{}，或者直接{}，在axios源码中，都会把Content-Type设置成application/json;charset=UTF-8，并且使用JSON.stringify序列化请求参数
     + 我们请求的Content-type被axios默认行为设置成了application/json;charset=UTF-8
     + 这就与我们服务器端要求的'Content-Type': 'application/x-www-form-urlencoded'以及@RequestParam不符合了

   + 解决：

     + URLSearchParams，主要用来处理URL上参数串的，主要处理形如AAA=BBB&CCC=DDD

       ```javascript
       // main.js里
       import axios from 'axios';
       axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
       Vue.prototype.$axios = axios;
       
       // 在组件vue里
       var params = new URLSearchParams();
       params.append('key1', 'value1');       //你要传给后台的参数值 key/value
       params.append('key2', 'value2');
       params.append('key3', 'value3');
       this.$axios({
           method: 'post',
           url:url,
           data:params
       }).then((res)=>{
           
       });
       ```

     + 使用qs包裹请求参数

       ```javascript
       let postData = this.$qs.stringify({
           key1:value1,
           key2:value2,
           key3:value3,
       });
       // var a = {name:'hehe',age:10};
       // qs.stringify(a); 'name=hehe&age=10'
       // JSON.stringify(a) '{"name":"hehe","age":10}'
       ```

     + 使用axios请求时，有data和params两种方式，默认都是使用data的方式，是直接把 json 放到请求体中提交到后端的，往往后端接收参数用的是 `@RequestParam`（通过字符串中解析出参数）需要后端修改注解成@RequestBody才行

       

4. 路由传参的坑：使用路由传参，当本页面刷新的时候，页面上是没有参数的，因为参数是从上个页面带过来的

   + 使用缓存或者vuex解决

---

### Request payload和form data的区别

FormData和Payload是浏览器传输给后端接口的两种内容格式，这两种方式浏览器是通过Content-Type来进行区分的：

+ 如果是 application/x-www-form-urlencoded的话，则为formdata方式。参数格式为：key=value&key=value&key=value

+ 如果是application/json或multipart/form-data的话，则为 request payload。参数格式为：

  {key1：value1，key2：value2}

---

### Vue3.0发布了，你升级了吗？出于哪方面考虑呢？

看下API，看看变化是否非常大。

+ 如果变化不大，可以根据工作需求
+ 如果变化很大，正好项目也有重构计划，就评估工作量，考虑重构

---

### 为什么Vue被称为渐进式框架，你是怎么理解的？

每个框架都有自己的特点，从而对我们开发者会有一定的要求，这些要求就是主张，主张有强有弱，它的强势程度会影响在业务开发中的使用方式，而Vue是主张最小的。



Vue的每一个功能，都可以抽离出来当成一个js库去使用，比如表单验证、用Vue管理页面的dom等，都可以抽象成一个小组件去在我们的项目中使用，或者加入vue-router路由组件开始做一个小型网站。我们可以在现有的项目上使用Vue实现一些小功能，也可以用Vue从0-1构建一个完整的工程



而不像React有自己的运行环境，你必须了解什么是函数式编程理念、什么是纯函数、如何隔离副作用等等一系列概念

---

### Vue项目部署

1. 如何部署：
   + 前后端分离的模式下，前后端是独立部署的，前端只需要将构建的文件夹上传至目标服务器指定的目录下就行。
2. 404问题：Vue项目在本地时运行正常，但部署到服务器上的时候刷新页面出现了404错误。404错误意味着链接指向的资源不存在，并且只有`history`模式下出现这个问题

---

### 你的项目中是怎么做性能优化的

主要体现在我上家公司的一个项目：打包很慢，71s，首屏时间也很长

1. 打包慢的问题：

   + 提升基础环境版本，把 npde/npm 的版本都看看，能提升的提升一下

   + 使用``webpack-bundle-analyzer`图形化界面，看了下到底哪里比较慢，发现项目中引入了jquery以及vue全家桶和elementui框架在图形化中占据的面积特别大

   + 将jquery、vue、vuerouter、moment这种不经常改动的库每次不用重新打包，下次打包直接引用即可。配置DLLPlugin和 DllReferencePlugin， 将引用的依赖提取。

     + 第一步，编写一个dll的配置文件

     ```javascript
     // 编写一个dll的配置文件，去打包dll文件
     const path = require('path')
     const DllPlugin = require('webpack/lib/DllPlugin')
     const { srcPath, distPath } = require('./paths')
     
     module.exports = {
       mode: 'development',
       // JS 执行入口文件
       entry: {
         // 以 React为例 模块的放到一个单独的动态链接库
         react: ['react', 'react-dom']
       },
       output: {
         // 输出的动态链接库的文件名称，[name] 代表当前动态链接库的名称，
         // 也就是 entry 中配置的 react 和 polyfill
         filename: '[name].dll.js',
         // 输出的文件都放到 dist 目录下
         path: distPath,
         // 存放动态链接库的全局变量名称，例如对应 react 来说就是 _dll_react
         // 之所以在前面加上 _dll_ 是为了防止全局变量冲突
         library: '_dll_[name]',
       },
       plugins: [
         // 接入 DllPlugin
         new DllPlugin({
           // 动态链接库的全局变量名称，需要和 output.library 中保持一致
           // 该字段的值也就是输出的 manifest.json 文件 中 name 字段的值
           // 例如 react.manifest.json 中就有 "name": "_dll_react"
           name: '_dll_[name]',
           // 描述动态链接库的 manifest.json 文件输出时的文件名称
           path: path.join(distPath, '[name].manifest.json'),
         }),
       ],
     }
     ```

     + 第二步，在webpack中配置映射关系，防止打包时再次引用npm包

       ```javascript
       // 第一，引入 DllReferencePlugin
       const DllReferencePlugin = require('webpack/lib/DllReferencePlugin');
        plugins: [
         //告诉 Webpack 使用了哪些动态链接库
               new DllReferencePlugin({
                   // 描述 react 动态链接库的文件内容
                   manifest: require(path.join(distPath, 'react.manifest.json')),
               }),
        ]
       ```

       

   + 没有改动的js文件，走缓存：使用cache-loader如果文件没有改动则使用缓存或者使用hash计算文件变动，如果文件被改变，则hash的值改变，在上线后，浏览器访问时没有改变的文件会命中缓存，从而达到性能优化的目的，使用方式如下：

     ```javascript
     output: {
        filename: 'bundle.[contentHash:8].js',  // 打包代码时，加上 hash 后缀
        path: distPath
     },
     ```

     

   + 一些公共的代码：在打包时，提取公共的代码，并实现代码分割，这样公共代码只要打包一次。对于我们项目来说，引用了一个多次计算活动积分的文件，那么这个就可以提取出来，主要还是webpack的配置，optimazation

     ```javascript
       optimization: {
           
             // 分割代码块
            splitChunks: {
             chunks: "async”,//默认作用于异步chunk，值为all/initial/async/function(chunk),值为function时第一个参数为遍历所有入口chunk时的chunk模块，chunk._modules为chunk所有依赖的模块，通过chunk的名字和所有依赖模块的resource可以自由配置,会抽取所有满足条件chunk的公有模块，以及模块的所有依赖模块，包括css
             minSize: 30000,  //表示在压缩前的最小模块大小,默认值是30kb
             minChunks: 1,  // 表示被引用次数，默认为1；
              maxAsyncRequests: 5,  //所有异步请求不得超过5个
             maxInitialRequests: 3,  //初始话并行请求不得超过3个
             automaticNameDelimiter:'~',//名称分隔符，默认是~
             name: true,  //打包后的名称，默认是chunk的名字通过分隔符（默认是～）分隔
             cacheGroups: { //设置缓存组用来抽取满足不同规则的chunk,下面以生成common为例
              common: {
              name: 'common',  //抽取的chunk的名字
              chunks(chunk) { //同外层的参数配置，覆盖外层的chunks，以chunk为维度进行抽取
              },
              test(module, chunks) {  //可以为字符串，正则表达式，函数，以module为维度进行抽取，只要是满足条件的module都会被抽取到该common的chunk中，为函数时第一个参数是遍历到的每一个模块，第二个参数是每一个引用到该模块的chunks数组。自己尝试过程中发现不能提取出css，待进一步验证。
              },
             priority: 10,  //优先级，一个chunk很可能满足多个缓存组，会被抽取到优先级高的缓存组中
            minChunks: 2,  //最少被几个chunk引用
            reuseExistingChunk: true，//  如果该chunk中引用了已经被抽取的chunk，直接引用该chunk，不会重复打包代码
            enforce: true  // 如果cacheGroup中没有设置minSize，则据此判断是否使用上层的minSize，true：则使用0，false：使用上层minSize
            }
         }
     }
             }
     ```

   + 大图片统一放在cdn上，小图压缩

   + 多线程打包Happypack

2. 首屏加载时间很长的问题：解决了项目打包比较慢的问题，其实已经轻量化了，首页加载的几个大的文件都小了很多，其次我还做了一些其他优化来提高首屏加载时间：

   + 做了路由懒加载
   + 图片cdn
   + UI框架按需引用
   + 代码层面的优化：
     + 合理使用v-if和v-show
     + 合理使用watch和computed
     + 使用v-for必须添加key, 最好为唯一id, 避免使用index, 且在同一个标签上，v-for不要和v-if同时使用
     + 定时器的销毁。可以在beforeDestroy()生命周期内执行销毁事件；也可以使用$once这个事件侦听器，在定义定时器事件的位置来清除定时器。详细见vue官网

---

##### 总结

1. 打包时间慢：
   + node/npm升级
   + webpack-bundle-anylzar
   + jquery/vue/router
   + 没有改动的js文件
   + jquery/vue-router那些库
   + 计算积分的公共文件
   + 图片
   + 多线程打包
2. 首页加载慢：
   + 基于打包时间慢，解决了一些大文件的体积，减小了入口文件的体积，另外开启了一个gzip的压缩
   + 路由懒加载，用到再去加载
   + UI框架按需引用
   + 代码层面的优化

---

#### 项目中你遇到哪些难题？解决方案是什么？

1. 上一个项目，因为要跟外国合作，他们没有采用前后端分离的形式，所以每次调试，都要打包发布，才能看到效果，再加上打包时间很长，导致效率特别低，所以就解决了打包时间慢的问题。并且不用手动发布，而是直接打包到指定文件夹，打包完成就可以看到效果，跟热更新一样
2. 企业微信开发，因为用户群体只是一些零售人员，所以其实生态不是特别全，在跟微信的开发人员沟通的时候，发现其实有好多不兼容的问题他们自己也不知道。这就考验自己的协调能力，怎么把一个原本答应下来的需求，转换方案去实现























