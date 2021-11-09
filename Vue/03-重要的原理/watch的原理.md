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

+ 当传入的键值是`obj.a.b`的形式时，`parsePath`用来解析它，每一项键值都会被触发依赖收集，也就是说，`obj.a.b`任何一项键值的值发生改变都会触发watch回调
+ 当实例化完成之后，watch对象上的每一个属性，都会被实例化成`Watcher`的实例，能够进行数据的监听和派发更新

---

⑤ 接下来`class Watcher`进入`get()`方法

```javascript
get() {
  pushTarget(this)  // 将当前user-watcher实例赋值给Dep.target，读取时收集依赖
  let value = this.getter.call(this.vm, this.vm)  // 将vm实例传给闭包，进行读取操作
    
  if (this.deep) {  // 如果有定义deep属性
    traverse(value)  // 进行深度监听
  }
  popTarget()
  return value  // 返回闭包读取到的值，参数immediate使用的就是这里的值
}
// ...
```

解析：

+ `pushTarget`是将当前用户的`Watcher`挂载到`Dep.target`上，在收集依赖的时候，找的就是`Dep.target`，表示当前的Watcher是谁
+ 执行数据劫持get()调用的就是getter函数，接着触发dep.depend收集依赖

---

⑥ 更新 在更新时首先触发的是“数据劫持set”，调用 dep.notify 通知每一个 watcher 的 update 方法。

```javascript
update () {
  if (this.lazy) { dirty置为true
    this.dirty = true
  } else if (this.sync) {
    this.run()
  } else {
    queueWatcher(this)
  }
}
```

---

 ⑦ 深度监听

收集依赖时，有一段逻辑是这样的，判断是否需要深度监听，调用`traverse`并将值传入

```javascript
if (this.deep) {
  traverse(value)
}

// 源码位置：/src/core/observer/traverse.js
```

解析：

- `traverse`函数的逻辑大概是递归获取每一项属性，触发他们的“数据劫持get”收集依赖
- 由于是递归进行监听，肯定会有一定的性能损耗。因为每一项属性都要走一遍依赖收集的流程，所以尽量在业务中避免这些操作

---

#### 总结

+ `watch`的多种调用方法，在初始化的时候，会去区分：字符串形式调用、对象形式调用、多重属性调用、是否传递了`immediate`、`deep`属性等

+ 为什么能够实现监听：在初始化时，会遍历watch上的每一个属性，为每一个属性添加`Watcher实例`，在`Watcher`中进行依赖收集和更新

+ `immediate`原理：给watch设置了`immediate`属性后，会将实例化后得到的值传入回调，并立即执行一次回调函数

+ `deep`的原理：递归调用获取每一项属性，触发他们的`数据劫持`

+ 如果watch属性，在data里是个对象，并在data里面声明了自己的属性值：

  ```javascript
  data(){
    return {
      obj:{
        a:'123'
      }
    }
  }
  ```

  那么在watch的初始化流程中，会递归obj的每一个属性，给他们加上监听，当obj自己的这些属性变化的时候，watch也能捕捉到

+ `initWatch` -> `createWatch` ->  `vm.$watch ` ->` class Watcher`让watch上被监听的属性具有数据劫持依赖收集和更新的能力

+ 计算属性和侦听属性一样，在初始化的时候都会实例化一个`Watcher`实例，用来跟踪数据的变化做出响应

---

#### 参考链接

[掘金1](https://juejin.cn/post/6844903926819454983)

[掘金2](https://juejin.cn/post/6844904201752068109)
