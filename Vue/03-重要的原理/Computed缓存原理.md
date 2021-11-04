#### 计算属性的缓存原理

计算属性是基于它们的响应式依赖进行缓存的，只有相关响应式依赖发生改变时它们才会重新求值

---

##### 初始化过程

① `initComputed`的时候，会为每个计算属性实例化一个计算watcher，用来存放所有计算属性相关的watcher叫做计算watcher，计算watcher的核心是`get`求值和`update`更新。

watcher的初始状态中，有一个代表是否需要重新求值的属性`dirty`，`dirty`的默认值是true，因为首次在模板中访问计算属性`watcher.evaluat()`的时候肯定是要求值的。

```javascript
// watcher
{
   deps: [],
   dirty: true, // 缓存的关键
   getter: ƒ sum(),
   lazy: true, // 惰性计算，只有真正在模板中读取它的值后才会去计算
   value: undefined
   // ... 
}
```

---

② `watcher.evaluate()`先求值，然后把`dirty`置为false，下次没有特殊情况再次读取计算属性的时候，发现dirty为false，就会直接返回watcher.value这个值，**这就是计算属性缓存的概念**

```javascript
// 经过初始化流程之后的 计算属性sum get函数如下
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

// 这是evaluate函数
evaluate () {
  // 调用 get 函数求值
  this.value = this.get()
  // 把 dirty 标记为 false
  this.dirty = false
}
```

解析：

+ `dirty`代表脏数据，表示是否需要重新求值。第一次读取的时候肯定是为true表示需要求值
+ 求值调用`watcher.evaluate()`，这个函数是先求值，然后将`dirty`置为false，下次没有特殊情况再次读取sum的时候，发现`dirty`是false，就直接返回这个值`watcher.value`就可以了

---

##### 更新时

接下来我们需要看一下更新流程，看看下面例子中count的更新是怎么 触发sum在页面上的变更的

```vue
<div>{{sum}}<div>
<script>
  data(){
     return {
         count: 1
     } 
  },
  computed:{
     sum(){
        return this.count+1 
     } 
  }    
</script>    
```

解析：

①  在`evaluate()`函数里，也就是读取sum发现是脏数据时做的求值操作

```javascript
evaluate () {
  // 调用 get 函数求值
  this.value = this.get()
  // 把 dirty 标记为 false
  this.dirty = false
}
```













































② count是一个响应式属性，这里会触发count的get劫持

```javascript
// 在闭包中，会保留对于 count 这个 key 所定义的 dep
const dep = new Dep()

// 闭包中也会保留上一次 set 函数所设置的 val
let val

Object.defineProperty(vm, 'count', {
  get: function reactiveGetter () {
    const value = val
    // Dep.target 此时就是计算watcher
    if (Dep.target) {
      // 收集依赖
      dep.depend()
    }
    return value
  },
})
```

+ count会收集计算watcher作为依赖

③ 依赖是如何收集的呢？

```javascript
// dep.depend()
depend () {
  if (Dep.target) {
    Dep.target.addDep(this)
  }
}
```

解析：

+ 这里其实是调用`Dep.target.addDep(this)`去收集





#### 如何阻止缓存

