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





























