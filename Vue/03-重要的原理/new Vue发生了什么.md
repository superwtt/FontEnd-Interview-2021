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

---

##### 2.0 关于Vue的定义

源码目录`src/core/instance/index.js`，Vue其实就是一个用ES5的Function实现的类，我们只能通过`new Vue`去实例化使用它

```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

①  `this._init`，把options作为参数传进去，options就是我们在实例化`new Vue`的时候传入的参数。

② `_init`方法在`initMixin`中，`_init`方法做了三件事：

+ `mergeOptions`：将传入`new Vue`的options和Vue构造函数上的options进行合并，构造函数上的options是Vue手动包装上去的
+ 初始化生命周期、初始化事件中心、初始化渲染、初始化data、props、computed、watcher等
+ $mount挂载

```javascript
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // ...
    // merge options
    // 有子组件时，_isComponent才会为true
    if (options && options._isComponent) {
      // 优化组件实例
      initInternalComponent(vm, options)
    } else {
      // 传入的options和vue自身的options进行合并  
      vm.$options = mergeOptions(

        // 解析当前实例构造函数上的options  
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
    // expose real self
    vm._self = vm
    initLifecycle(vm) // 初始化一些生命周期相关
    initEvents(vm) // 初始化事件相关属性，当有父组件的方法绑定在子组件的时候，供子组件调用
    initRender(vm) // 添加slot属性
    callHook(vm, 'beforeCreate') // 调用beforeCreated钩子 
    initInjections(vm) // 
    initState(vm) // 初始化数据，进行双向绑定
    initProvide(vm) // 
    callHook(vm, 'created') // 调用created钩子

    if (vm.$options.el) {
      vm.$mount(vm.$options.el) // 把模板转为render函数
    }
  }
}
```

③ `mergeOptions`：调用`resolveConstructorOptions(vm.constructor)`，





















































































































