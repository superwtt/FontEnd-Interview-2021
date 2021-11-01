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

解析：

 ① `mergeOptions`：调用`resolveConstructorOptions(vm.constructor)`， 将传入new Vue的options和Vue构造函数上的options进行合并生成一个新的options。该函数用来解析`constructor`上的options属性。分为两种场景：

+ 第一种Ctor是基础Vue构造器的情况，比如是通过`new Vue`关键字新建Vue的实例

  ```javascript
  new Vue({
     el:"#app",
     data:{
         message:"Hello world"
     } 
  })
  ```

  这个时候，options就是Vue构造函数上的options

   ![](https://image-static.segmentfault.com/386/004/386004732-5ade8e89b3294_fix732)

+ 第二种是Ctor是通过Vue.extend方法扩展的情况，当Ctor是`Vue.extend`创建的子类，合并父类的options和自身的options

---

②  初始化生命周期、初始化事件中心、初始化渲染、初始化data/props/computed/watcher等等

+ `initState`，初始化数据，做了四件事：

  + `initProps`：
    + 遍历props中的每个属性，校验类型看看是否是指定的数据类型
    + 数据监测，给props赋值就报错
    + 给props添加响应式监听，proxy拦截设置get/set
  + `initMethods`：
    + 校验methods的名字是否有效，比如定义了函数名字但是没有函数体、跟props和vue实例是否同名，如果同名会发出警告
    + ![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/initMethods警告.png)
  + `initData`：遍历data，调用proxy给data数据添加响应式监听
  + `initComputed`：computed计算属性可以是一个函数也可以是一个对象。计算属性是基于它们的响应式依赖进行缓存的，只有相关响应式依赖发生改变时才会重新求值
    + ![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/computed缓存.png)
    + 如果computed属性是一个函数，则默认为getter函数；如果computed属性是一个对象，它有三个有效字段：set、get、cache。cache默认为true

  + `initWatch`：遍历watch中的属性，为每一个属性创建侦听器

---

③ 挂载：`vm.$mount(vm.$options.el)`，Vue原型对象上定义的$mount内部实际上是执行`mountComponent`。`mountComponent`做了四件事：

+ 判断是否有`template`属性，如果有`template`属性却没有使用`compiler`版本，那必须使用Vue加compiler的版本

+ 执行`beforeMount`生命周期钩子

+ 开始执行挂载，最核心的两个方法是`vm._render`和`vm._update`

  + `vm._render`：把实例渲染成一个虚拟VNode，通过执行`createElement`方法并返回VNode，它是虚拟node。该方法的作用就是创建一个树状的虚拟DOM树
    + `createElement`用来创建VNode以及各个节点的子节点
  + `vm._update`：首次渲染、数据更新DOM的时候调用，把VNode渲染成真实的DOM，`vm._update`的核心就是调用`vm._patch_`方法，实例化一个组件的时候，其整个过程就是遍历VNode Tree递归创建一个完整的DOM树并插入到Body上，内部实现的是真实的dom操作

+ 执行`mounted`生命周期钩子

  ```javascript
  Vue$3.prototype.$mount = function (
    el,
    hydrating
  ) {
    el = el && inBrowser ? query(el) : undefined;
    return mountComponent(this, el, hydrating)
  };
  
  // mountComponent
  function mountComponent (
    vm,
    el,
    hydrating
  ) {
    vm.$el = el;
    if (!vm.$options.render) {
      vm.$options.render = createEmptyVNode;
      if (process.env.NODE_ENV !== 'production') {
        /* istanbul ignore if */
        if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
          vm.$options.el || el) {
          warn(
            'You are using the runtime-only build of Vue where the template ' +
            'compiler is not available. Either pre-compile the templates into ' +
            'render functions, or use the compiler-included build.',
            vm
          );
        } else {
          warn(
            'Failed to mount component: template or render function not defined.',
            vm
          );
        }
      }
    }
    callHook(vm, 'beforeMount');
  
    var updateComponent;
    /* istanbul ignore if */
    if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
      updateComponent = function () {
        var name = vm._name;
        var id = vm._uid;
  
        var vnode = vm._render();
        measure(("vue " + name + " render"), startTag, endTag);
  
        vm._update(vnode, hydrating);
        measure(("vue " + name + " patch"), startTag, endTag);
      };
    } else {
      updateComponent = function () {
        vm._update(vm._render(), hydrating);
      };
    }
  
    vm._watcher = new Watcher(vm, updateComponent, noop);
    hydrating = false;
  
    // manually mounted instance, call mounted on self
    // mounted is called for render-created child components in its inserted hook
    if (vm.$vnode == null) {
      vm._isMounted = true;
      callHook(vm, 'mounted');
    }
    return vm
  }
  ```

---

④ 关于VNode

+ VNode就是对真实DOM的一种描述，它的核心定义无非就是几个关键属性、标签名、数据、子节点等，其他属性都是用来扩展VNode灵活性以及实现一些特殊功能的
+ 由于VNode是用来映射到真实DOM的渲染，不需要包含操作DOM的方法，因此它是非常轻量的

```javascript
// VNode结构
// 源码目录：src/core/vdom/vnode.js
export default class VNode {
  tag: string | void;
  data: VNodeData | void;
  children: ?Array<VNode>;
  text: string | void;
  elm: Node | void;
  ns: string | void;
  context: Component | void; // rendered in this component's scope
  key: string | number | void;
  componentOptions: VNodeComponentOptions | void;
  componentInstance: Component | void; // component instance
  parent: VNode | void; // component placeholder node

  // strictly internal
  raw: boolean; // contains raw HTML? (server only)
  isStatic: boolean; // hoisted static node
  isRootInsert: boolean; // necessary for enter transition check
  isComment: boolean; // empty comment placeholder?
  isCloned: boolean; // is a cloned node?
  isOnce: boolean; // is a v-once node?
  asyncFactory: Function | void; // async component factory function
  asyncMeta: Object | void
  isAsyncPlaceholder: boolean;
  ssrContext: Object | void;
  fnContext: Component | void; // real context vm for functional nodes
  fnOptions: ?ComponentOptions; // for SSR caching
  fnScopeId: ?string; // functional scope id support

  constructor (
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag
    this.data = data
    this.children = children
    this.text = text
    this.elm = elm
    this.ns = undefined
    this.context = context
    this.fnContext = undefined
    this.fnOptions = undefined
    this.fnScopeId = undefined
    this.key = data && data.key
    this.componentOptions = componentOptions
    this.componentInstance = undefined
    this.parent = undefined
    this.raw = false
    this.isStatic = false
    this.isRootInsert = true
    this.isComment = false
    this.isCloned = false
    this.isOnce = false
    this.asyncFactory = asyncFactory
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }

  // DEPRECATED: alias for componentInstance for backwards compat.
  /* istanbul ignore next */
  get child (): Component | void {
    return this.componentInstance
  }
}
```







#### 参考链接

[人人都能读懂的Vue源码系列](https://segmentfault.com/u/zhenren/articles)



[ustbhuangyi](https://ustbhuangyi.github.io/vue-analysis/v2/data-driven/new-vue.html)

















































































































