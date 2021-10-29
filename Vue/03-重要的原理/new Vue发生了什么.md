#### new Vue发生了什么

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/new-vue过程.png)

---

##### new Vue的几种写法

```javascript
// 1.0
new Vue({
    el:"#app",
    template:"<App />",
    components:{App}
})
// 2.0
new Vue({
    template:"<App />",
    components:{App}
}).amount("#app")
// 3.0
new Vue({
    render:h=>h(App),
}).amount("#app")
// 4.0
new Vue({
    render:function(h){
        return h(App)
    }
}).amount("#app")
```

---

##### 1.0 从入口文件开始

用vue-cli创建项目时，我们通常都会在页面的入口文件mainjs中new 一个Vue对象的实例

```javascript
// main.js中
import Vue from "vue";

new Vue({
    el:"#app",
    router,
    store,
    render:h=>h(App)
})
```



























































































































