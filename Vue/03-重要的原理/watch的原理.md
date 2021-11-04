#### watch侦听属性

watch的作用是用来监控一个值的变化，并拿到变化前后的值，做出一系列操作响应数据的变化

---

#### watch的初始化流程

① initWatch

```javascript
function initWatch (vm: Component, watch: Object) {
  for (const key in watch) {
    const handler = watch[key]
    if (Array.isArray(handler)) {
      for (let i = 0; i < handler.length; i++) {
        createWatcher(vm, key, handler[i])
      }
    } else {
      createWatcher(vm, key, handler)
    }
  }
}
```

解析：

+ `initWatch`的主要工作就是调用`createWatcher`创建监听实例
+ 数组声明的watch有多个回调，需要循环创建监听
+ 其他方式直接创建

---

② 进入创建方法`createWatcher`看看做了什么

```javascript
function createWatcher (
  vm: Component,
  keyOrFn: string | Function,
  handler: any,
  options?: Object
) {

  // 1
  if (isPlainObject(handler)) {
    options = handler
    handler = handler.handler
  }

  // 2
  if (typeof handler === 'string') {
    handler = vm[handler]
  }

  // 3 keyOrFn是watch的key值
  return vm.$watch(keyOrFn, handler, options)
}
```

解析：因为`watch`有多种调用方式

+ 根据`watch`的多种调用方式，获取`watch`上属性的回调
  + 对象声明的watch，从对象中取出回调
  + 字符串声明的watch，直接取实例上的方法
+ 返回Vue实例上的$watch方法

---

③ 继续看看Vue实例上的$watch干了什么，wm.$watch` 源码目录：`/src/core/instance/state.js

```javascript
Vue.prototype.$watch = function (
    expOrFn: string | Function,
    cb: any,
    options?: Object
  ): Function {
    const vm: Component = this
    // 如果cb是对象，当手动创建监听属性时
    if (isPlainObject(cb)) {
      return createWatcher(vm, expOrFn, cb, options)
    }
    // 1
    options = options || {}
    options.user = true // user-watcher标志位
    // 2
    const watcher = new Watcher(vm, expOrFn, cb, options)
    // 3
    if (options.immediate) { // 立即执行
      cb.call(vm, watcher.value) // 以当前值立即执行一次回调函数
  }
    // 4 返回一个函数，执行取消监听
  return function unwatchFn () {
      watcher.teardown()
  }
}  
```

解析：`vm.$watch`做了三件事

+ 设置标记位`options.user`为true，表明这是一个`user-watcher`，所有`用户Watcher`的options，都会带有`user`标识
+ 如果在组件中使用侦听属性时，设置了`watch`的立即执行属性`immediate`，会立即执行一次回调函数，这就是`immediate`的原理
+ 最后返回一个方法，执行后取消对该监听属性的监听

---

④ `vm.$watch`的流程走完，关键是去创建一个`Watcher`的监听实例，这里也是依赖收集和更新的触发点

```javascript
// 源码位置：/src/core/observer/watcher.js
class Watcher {
  constructor(vm, expOrFn, cb, options) {
    this.vm = vm
    vm._watchers.push(this)  // 添加到当前实例的watchers内
    
    if(options) {
      this.deep = !!options.deep  // 是否深度监听
      this.user = !!options.user  // 是否是user-wathcer
      this.sync = !!options.sync  // 是否同步更新
    }
    
    this.active = true  // // 派发更新的标志位
    this.cb = cb  // 回调函数
    
    if (typeof expOrFn === 'function') {  // 如果expOrFn是函数
      this.getter = expOrFn
    } else {
      this.getter = parsePath(expOrFn)  // 如果是字符串对象路径形式，返回闭包函数 obj.a.b.c，不仅仅是修改c的时候会触发回调，修改b、a以及obj同样会触发回调
    }
    ...
  }
}
```

解析：

+ 当是`usr-watcher`时，





---

⑤ 







































