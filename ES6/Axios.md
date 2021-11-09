#### Axios是什么

axios是一个<font style="color:red">**基于Promise的HTTP库**</font>，可以用在浏览器和node.js中。在Vue项目中，经常会选择Axios作为发送请求的http库

---

#### Axios基本使用

```javascript
import axios from "axios"

// 1.0 请求
axios(config) // 直接传入配置 与 axios.request()等价
axios(url[,config]) // 传入url和配置
axios[method](url[, option]) // 直接调用请求方法，传入url和配置
axios[method](url[, data[, option]]) // 直接调用请求方式方法，传入data、url和配置
axios.request(option) // 调用 request 方法
axios.all([axiosInstance1, axiosInstance2]).then()

// 2.0 自定义实例
axios.create(config) // 自定义配置，创建实例instance

// 3.0 请求拦截器
axios.interceptors.request.use(function(config){
  // 这里写发送请求前处理的代码
   return config
},function (error){
   // 这里写发送请求错误相关代码 
   return Promise.reject(error) 
})

// 4.0 响应拦截器
axios.interceptors.response.use(function(){
   // 这里写得到响应数据后处理的代码
   return response 
},function (error){
   // 这里写得到错误响应处理的代码
   return Promise.reject(error) 
})

// 5.0 取消请求
const CancelToken = axios.CancelToken;
const source = CancelToken.source();
axios.get('xxx',{
  cancelToken: source.token  
})
source.cancel('主动取消请求')
```

---

#### Axios特点

1. 基于`Promise`的异步ajax请求
2. 浏览器端和node端都可以使用
3. 支持请求/响应拦截器
4. 支持取消请求
5. 请求/响应数据转换
6. 批量发送多个请求

---

#### 为什么要封装Axios

axios的API很友好，我们完全可以在项目中直接使用



但是随着项目规模增大，如果每发起一次HTTP请求，就要把这些比如设置超时时间、设置请求头、根据项目环境判断使用哪个请求地址、错误处理等操作重新写一遍的话，会让代码冗余不堪，而且难以维护



这时候我们就需要对axios进行二次封装，让使用更加便利，代码简洁容易维护

---

#### 如何封装

开始封装前，需要跟后端沟通好一些约定：比如请求头、状态码、请求超时时间等

1. 设置请求前缀：根据开发、测试、生产环境的不同，区分前缀

   ```javascript
   if(process.env.NODE_ENV === 'dev'){
     baseUrl = 'http://dev.xxx.com' 
   } else if(process.env.NODE_ENV === 'prod'){
     baseUrl = 'http://prod.xxx.com'  
   }
   ```

   ```javascript
   // 在本地调试的时候，还得配置`devServer`实现代理转发，从而实现跨域
   devServer:{
     proxy:{
        '/proxyApi':{
          target:'http://dev.xxx.com',
          changeOrigin: true,
          pathRewrite:{
           '/proxyApi': ''  
          }
       }  
     }  
   }
   ```

2. 封装请求头：设置请求头与超时时间

   ```javascript
   const service = axios.create({
     //...
     timeout: 30000 , //请求30s超时
     headers:{
        get:{
          'Content-Type': 'application/x-www-form-urlencoded;charset=utf-8'
        },
        post:{
          'Content-Type': 'application/json;charset=utf-8'
        }
     } 
   })
   ```

3. 状态码：根据接口返回的不同状态码，响应不同的操作

4. 封装请求方法：根据get/post等方法需要进行再一次的封装

   ```javascript
   // get
   export function httpGet({url,params={}}){
      return new Promise((resolve,reject)=>{
        axios.get(url,{
           params
        }).then(res=>{
           resolve(res.data)
        }).catch(err=>{
           reject(err)
        })
      })
   }
   // post
   export function httpPost({
      url,data={},params={}
   }){
      return new Promise((resolve,reject)=>{
         //...
      })
   }
   ```

5. 请求拦截：请求发送之前做的操作

   ```javascript
   axios.interceptors.request.use(
      config=>{
        // 每次发送请求之前判断是否存在token
       // 如果存在，则统一在http请求的header都加上token，这样后台根据token判断你的登录情况，此处token一般是用户完成登录后储存到localstorage里的
       token && (config.headers.Authorization = token)
       return config
      },
      error =>{
         return Promise.error(error)
      }
   )
   ```

6. 响应拦截：请求响应之后做的操作，根据业务定制

   ```javascript
   axios.interceptors.response.use(response=>{
      if(response.status === 200){
        if(response.data.code === 511){
           // 未授权调取授权接口
        } else if(response.data.code === 510){
           // 未登录跳转登录页
        } else{
           return Promise.resolve(response)
        }
      } else{
         return Promise.reject(response)
      }
   },error=>{
      // 异常处理
   })
   ```

---

#### 手动实现一个Axios

1. 构建一个`Axios`构造函数，核心代码为`request`

   ```javascript
   class Axios{
      constructor(){}
      request(){
        return new Promise(resolve=>{
          const {url = '', method = 'get', data = {}} = config;
          // 发送ajax请求
          const xhr = new XMLHttpRequest();
          xhr.open(method, url, true);
          xhr.onload = function() {
          console.log(xhr.responseText)
             resolve(xhr.responseText);
          }
          xhr.send(data);  
        })  
      } 
   }
   ```

2. 导出Axios实例

   ```javascript
   function CreateAxiosFn(){
     let axios = new Axios();
     let req = axios.request.bind(axios);
     return req;  
   }
   // 得到最后的全局变量axios
   let axios = CreateAxiosFn();
   ```

   这边就已经能够实现`axios({})`这种请求，继续实现`axios.method()`这种请求

3. 实现`axios.method()`这种请求

   + 在原型上定义请求方法，去实现`axios.xxx`

     ```javascript
     const methodsArr = ['get', 'delete', 'head', 'options', 'put', 'patch', 'post'];
     methodsArr.forEach(met=>{
        Axios.prototype[met] = function(){
          console.log('执行'+met+'方法');  
          if(['get'],['delete'],['head'],['options']).includes(met)){
            return this.request({
              method: met,
              url: arguments[0],
              ...arguments[1] || {} 
            })  
          }  
          return this.request({
              method: met,
           url: arguments[0],
              data: arguments[1] || {},
           ...arguments[2] || {}
           })  
        } 
     })
     ```
   
   + 修改导出方法
   
     ```javascript
     let axios = new Axios();
     axios.get()
     ```
   

4. 实现请求拦截和响应拦截：

   + `axios.interceptors.request.use`
   + `axios.interceptors.response.use`

   ```javascript
   class InterceptorsManage {
      constructor(){
        this.handlers = [];  
      }  
      use(fullfield,rejected){
        this.handlers,push({
          fullfield,
          rejected  
        })  
      } 
   }
   class Axios(){
      constructor(){
        this.interceptors = {
            request: new InterceptorsManage,
            response: new InterceptorsManage   
        }  
      } 
      request(config) {
    	  // ...
      }
   }
   ```

---

#### 参考链接

[Good](https://github.com/febobo/web-interview/issues/26)

---

#### 总结

1. `Axios`是一个基于`Promise`的HTTP请求库，其实现的核心有两层：
   + 底层是让浏览器发送`XMLHttpRequest`:
     + `xhr.open(method,url)`
     + `xhr.onload`
     + `xhr.send(data)`
   + 其次再用`Promise`封装这层请求：
     + `return new Promise(xhr)`

2. 由于Axios强大的api和丰富的社区环境，我们经常会选用Axios作为日常项目发送Ajax请求的库







