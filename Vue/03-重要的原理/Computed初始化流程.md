#### Computed

Vue中的计算属性，不建议直接在`template`中写大量的计算代码，比如`{{this.firstName + ‘ ‘ + this.lastName}}`。因为在模板中放入太多声明逻辑会让模板本身过重，尤其当在页面中使用大量复杂的逻辑表达式处理数据时，会对页面可维护性造成很大的影响，而computed的设计初衷也正是解决此类问题

---

#### 基本使用

```javascript
computed:{
  // 作为方法使用
  message(){
    return "这是message经过计算后的值"  
  },
  // 作为对象使用
  assign:{
    set: function(){
       this.testData = "这是testData取值时候的值" 
    },
    get:function(){
       return "get的时候拿上这个值"+this.testData 
    }
  }    
}
```

解析：

1. 定义一个计算属性有两种写法，一种是直接跟一个函数，另一种是添加set和get方法的对象形式。计算属性默认只有`getter`，`getter`是不可以缺少的
   + 作为函数使用的时候，`getter`默认就是该函数
   + 作为对象使用的时候，`setter`和`getter`为显示指定的`get`和`set`
   + 如果想显示地定义`get`和`set`，那么最好还是使用对象的形式

---

#### 计算属性的初始化initComputed

① `new Vue`实例化的过程中，会调用`this._init`方法进行应用的初始化，包括了生命周期、事件、数据、属性、方法、执行生命周期等过程，其中数据的初始化方法`initState`包括了属性的初始化

```javascript
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

---

② 进入`initComputed`方法

```javascript
function initComputed (vm: Component, computed: Object) {
  const watchers = vm._computedWatchers = Object.create(null)
  const isSSR = isServerRendering()

  // 如果计算属性是方法，那么getter默认就是该方法
  // 如果计算属性是对象，那么getter就是显示指定的get值
  // 否则报错 getter是必须的
  for (const key in computed) {
    const userDef = computed[key]
    const getter = typeof userDef === 'function' ? userDef : userDef.get
    if (process.env.NODE_ENV !== 'production' && getter == null) {
      warn(
        `Getter is missing for computed property "${key}".`,
        vm
      )
    }

    if (!isSSR) {
      // 为计算属性添加监听
      watchers[key] = new Watcher(
        vm,
        getter || noop,
        noop,
        computedWatcherOptions
      )
    }

    // 判断computed属性是否有重名
    if (!(key in vm)) {
      // 没有重名
      defineComputed(vm, key, userDef)
    } else if (process.env.NODE_ENV !== 'production') {

      // 有重名  
      if (key in vm.$data) {
        warn(`The computed property "${key}" is already defined in data.`, vm)
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(`The computed property "${key}" is already defined as a prop.`, vm)
      }
    }
  }
}
```

解析：`initComputed`初始化做了三件事：

+ 确认属性的`getter`方法

+ 判断实例中是否已经存在重名的属性，因为不管是`props`还是`data`还是`methods`，最终都会挂载在实例下，通过`this.xxx`去访问，所以重名肯定不行

+ 依次为每个计算属性定义一个计算watcher动态监听，每个计算属性对应的计算`watcher`初始状态如下：

  ```javascript
  {
     deps:[],
     dirty: true, // 缓存的关键，第一次在模板中读取计算属性的时候，肯定是要求值的，所以这里默认为true
     getter: f sum(),
     lazy: true, // lazy是true，说明它的值是惰性计算的，只有到真正在模板里读取它的值才会计算
     value: undefined 
  }
  ```

---

③ 进入`defineComputed`，对计算属性添加属性修饰符。这个函数定义了当用户在调用计算属性时会发生的事情

```javascript
export function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  const shouldCache = !isServerRendering()

  // 如果是方法
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : userDef
    sharedPropertyDefinition.set = noop
  } else {
    // 如果是对象
    sharedPropertyDefinition.get = userDef.get
      ? shouldCache && userDef.cache !== false
        ? createComputedGetter(key)
        : userDef.get
      : noop
    sharedPropertyDefinition.set = userDef.set
      ? userDef.set
      : noop
  }
  if (process.env.NODE_ENV !== 'production' &&
      sharedPropertyDefinition.set === noop) {
    sharedPropertyDefinition.set = function () {
      warn(
        'Computed property "${key}" was assigned to but it has no setter.',
        this
      )
    }
  }
  // 将计算属性定义到Vue实例对象上  
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

解析：

+ `defineComputed`方法通过判断传进来的`computed`属性的值是方法还是对象，来重写`sharedPropertyDefinition`上的get属性描述符，然后将计算属性定义到Vue实例对象上

  + `sharedPropertyDefinition`的get函数也就是`createComputedGetter`的返回结果

  + `sharedPropertyDefinition`属性描述符是这样的：

    ```javascript
    const sharedPropertyDefinition = {
      enumerable: true,
      configurable: true,
      get: noop,
      set: noop
    }
    ```

---

④ `createComputedGetter`：用来判断是否需要重新求值，并返回最终的值

```javascript
function createComputedGetter (key) {
  return function computedGetter () {
    const watcher = this._computedWatchers && this._computedWatchers[key]
    if (watcher) {
      // 只有dirty了才会重新求值  
      if (watcher.dirty) {
        watcher.evaluate() // 计算求值，设置Dep.target
      }
      if (Dep.target) {
        watcher.depend() // 依赖收集
      }
      return watcher.value // 最后返回计算出来的值
    }
  }
}
```

---

⑤ 最终经过改写的`sharedPropertyDefinition`如下：

```javascript
sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: function computedGetter() {
    const watcher = this._computedWatchers && this._computedWatchers[key];
    if (watcher) {
      // 只有dirty了才会重新求值 watcher在初始化的时候默认是true需要求值
      if (watcher.dirty) {
        watcher.evaluate(); // 计算求值
      }
      if (Dep.target) {
        watcher.depend(); // 依赖收集
      }
      return watcher.value;
    }
  },
  set: userDef.set || noop,
};
```

解析：

- 当计算属性被调用时便会执行get函数，从而关联上观察者对象watcher然后执行`watcher.depend()`收集依赖和`wacth.evaluate()`计算求值
- dirty这个字段代表脏数据，说明这个数据需要重新调用用户传入的函数来求值。当第一次在模板中读取的时候一定是true，所以初始化一定会经历一次求值
- 到这里computed的初始化就做好了，等待模板中访问

---

#### 对比监听属性watch

1. 计算属性`computed`：
   + 支持缓存，`computed`属性值会默认走缓存，只有依赖发生改变，才会重新进行计算
   + 不支持异步，当`computed`内有异步操作时无效，无法监听数据的变化
   + 当多个属性影响一个属性的时候，建议使用`computed`
2. 监听属性`watch`:
   + 不支持缓存，数据变化，直接会触发相应的操作
   + 支持异步
   + 监听的属性必须是data声明过的属性或者父组件传递过来的props里的数据
   + 有两个参数可以额外调用：
     + `immediate`：组件加载立即触发回调执行
     + `deep`：深度监听
   + 当一个值的变化会引起一系列的操作，建议使用监听属性

---

#### 总结

1. 当组件开始初始化的时候，`computed`和`data`会建立自己的`响应系统`，通过`Observer`遍历`data/computed`中的每个属性`get/set`数据拦截
2. 初始化`computed`会调用`initComputed`函数，流程如下：
   + `initComputed` ：注册一个Watcher观察者实例，并在内部实例化一个Dep消息订阅器作后续收集依赖。内部继续调用`defineComputed` 
   + `defineComputed`：内部调用 `createComputedGetter`，添加属性描述符get/set
   +  `createComputedGetter` ：返回一个计算后的属性描述符：
     + `watcher.depend()`向自身的消息订阅器dep的subs中
     + 添加订阅者
     + `watcher.evaluate`执行getter函数求值
   + 完成属性描述符的定义 

   + 属性变化，触发set，然后调用dep.notify()方法，遍历dep中所有的订阅者，逐个调用watcher.update()方法，完成响应式的更新
3. <font style="color:blue">当某个属性发生变化，触发set拦截函数，然后调用自身消息订阅器dep的notify方法，遍历当前dep中保存着所有订阅者watcher的subs数组，逐个调用watcher的update方法，完成响应式更新</font>

---

#### 参考链接

[浅谈 Vue 中 computed 实现原理](https://mp.weixin.qq.com/s/igkif-J_BHd1q5mZ7TewCw)































































