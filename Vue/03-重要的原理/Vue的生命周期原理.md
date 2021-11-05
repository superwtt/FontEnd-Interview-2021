#### Vue的生命周期

所谓生命周期，是指从编写模板开始到能在浏览器运行的整个过程中，我们需要在这些过程中完成自己的业务操作，比如初始化数据、请求服务端数据、挂载DOM、渲染、更新、卸载等等，Vue给我们提供的操作这些的能力

---

#### 各个生命周期的数据状态以及挂载状态

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/各个生命周期的数据状态.png)



![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/destory.png)

---

#### beforeCreate和created

1. `beforeCreate`：此时el尚未挂载、data尚未初始化、data里的数据也读取不到

2. `created`：此时el尚未挂载、data已经初始化、data里的数据能够读取

3. 源码目录：`src/core/instance/init.js/initMixin`

   ```javascript
   export function initMixin (Vue: Class<Component>) {
     // ...
     initLifecycle(vm)
     initEvents(vm)
     initRender(vm)
     callHook(vm, 'beforeCreate')
     initInjections(vm)
     initState(vm) // 初始化数据
     initProvide(vm) 
     callHook(vm, 'created')
     // ...
   
     if (vm.$options.el) {
       vm.$mount(vm.$options.el)
     }
   }
   ```

   解析：

   + 应用初始化的时候，在初始化数据`initState`前后，分别执行了`beforeCreate`和`created`生命周期钩子，所以在`created`生命周期里，能够访问到初始化后的data对象以及里面的数据

---

#### beforeMount和mounted

1. `beforeMount`：此时el尚未挂载`this.$el`是undefind、data已经初始化、data里的数据能够读取
2. `mounted`：此时el已经挂载,`this.$el`能够取到值、data已经初始化、data里的数据能够读取
3. 源码目录：`src/platforms/web/runtime/index.js`

① 初始化执行完`created`生命周期函数后，会进入挂载流程

```javascript
export function initMixin (Vue: Class<Component>) {
  // ...
  callHook(vm, 'beforeCreate')
  initState(vm) // 初始化数据
  callHook(vm, 'created')
  // ...

  if (vm.$options.el) {
    vm.$mount(vm.$options.el)
  }
}
```

---

② 进入到`Vue.prototype.$mount`，内部只是返回了`mountComponent`

```javascript
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
 
// beforeMount
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
  }
  callHook(vm, 'beforeMount')

  let updateComponent
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    // ...
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  vm._watcher = new Watcher(vm, updateComponent, noop)
  hydrating = false

  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

解析：

+ 由源码的`mountComponent`函数可知，在真正执行挂载之前先执行了生命周期`beforeMount`
+ 然后执行`vm._update`方法
+ 执行完`vm.update`之后，就会立即执行`mounted`生命周期

---

③ 我们看看`vm._update`干了什么，源码目录`src/platforms/web/runtime/index.js`

```javascript
Vue.prototype._update = function (vnode, hydrating) {
    const vm: Component = this
    if (vm._isMounted) {
      callHook(vm, 'beforeUpdate')
    }
    const prevEl = vm.$el
    const prevVnode = vm._vnode
    const prevActiveInstance = activeInstance
    activeInstance = vm
    vm._vnode = vnode
    if (!prevVnode) {
      // initial render
      // 初始化渲染 将vnode渲染为真实的dom节点 替换调用vm.$el根节点
      vm.$el = vm.__patch__(
        vm.$el, vnode, hydrating, false /* removeOnly */,
        vm.$options._parentElm,
        vm.$options._refElm
      )
      vm.$options._parentElm = vm.$options._refElm = null
    } else {
      // updates
      // 数据更新 更新时做diff操作
      vm.$el = vm.__patch__(prevVnode, vnode)
    }
    activeInstance = prevActiveInstance
    // update __vue__ reference
    if (prevEl) {
      prevEl.__vue__ = null
    }
    if (vm.$el) {
      vm.$el.__vue__ = vm
    }
    // if parent is an HOC, update its $el as well
    if (vm.$vnode && vm.$parent && vm.$vnode === vm.$parent._vnode) {
      vm.$parent.$el = vm.$el
    }
  }
```

解析：

+ `vm._update`方法，最终会把VNode渲染成真实的DOM
+ `vm._update`方法，被调用的时机有两个，一个是初始化渲染，一个是数据更新
+ `vm._update`的核心，就是调用`vm._patch_`，`vm._patch_`内部就是原生方法，生成真实的DOM了

---

#### beforeUpdate和updated

当进入`_update`函数的时候，会先执行`beforeUpdate`函数，`_update`执行时机有两个：一个是初始化的时候，一个是更新的时候



当我们触发一次更新，调用链如下：

 `class Watcher`-> `update` -> `queueWatcher` -> `nextTick(flushSchedulerQueue)` -> `callUpdatedHooks(updatedQueue)` -> `callHook(vm, 'updated')`

---

#### beforeDestroy和destroyed

1. `beforeDestory`：此时el已经挂载,`this.$el`是undefined、data已经初始化、data里的数据能够读取
2. `destory`：此时el已经挂载,`this.$el`是undefined、data已经初始化、data里的数据能够读取

---

#### 执行生命周期的方法callhook

````javascript
// 源码目录：src/core/instance/lifecycle.js/callHook
export function callHook (vm: Component, hook: string) {
  const handlers = vm.$options[hook]
  if (handlers) {
    for (let i = 0, j = handlers.length; i < j; i++) {
      try {
        handlers[i].call(vm)
      } catch (e) {
        handleError(e, vm, `${hook} hook`)
      }
    }
  }
  if (vm._hasHookEvent) {
    vm.$emit('hook:' + hook)
  }
}

````

解析：

- 多次调用的执行生命周期方法的`callHook`函数，不过就是循环注册在生命周期上的方法，然后依次调用它们

- 生命周期执行顺序：

  `beforeCreate` -> `created` -> `beforeMount` -> `mounted` -> `beforeUpdate` -> `updated` -> `beforeDestroy` -> `destroyed`

  源码在执行到`callHook(vm, 'beforeMount')`的时候，中间去注册了`updateComponent`方法，该方法中有`beforeUpdate`和`updated`生命周期钩子，为什么这两个生命周期钩子没有在`mounted`之前执行？

  因为只是将`updateComponent`注册成了方法，里面有一个`Watcher`实例去跟踪更新，只有当更新发生时才会去执行

---

#### 总结

1. Vue的生命周期，就是当我们从编写模板开始，到能够在浏览器运行这一过程中，提供给我们操作：什么时候初始化数据、请求服务端数据、挂载DOM、渲染、更新、卸载的能力
2. Vue的生命周期有：`beforeCreate`、`created`、`beforeMount`、`mounted`、`beforeUpdate`、`updated`、`beforeDestroy`、`destroyed`
3. `beforeCreate`、`created`在初始化的时候进行，前者什么都访问不到，后者能够访问到data上的变量和初始化数据
4. `beforeMount`、`mounted`在初始化数据完成之后进行，前者跟`created`状态一样，后者在执行完`mountComponent`到`patch `流程之后，挂载完成，能够访问到el数据







































































