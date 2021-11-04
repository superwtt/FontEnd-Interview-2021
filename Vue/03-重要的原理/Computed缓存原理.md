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

---

##### Dep.target变更为 渲染watcher

（`Dep.target`表示当前正在运行的watcher）

这里进入到`this.get()`，首先要明白一点，在模板中读取`{{sum}}`变量的时候，全局的`Dep.target`应该是`渲染Watcher`



①  全局的`Dep.target`状态是用一个栈`targetStack`来保存，便于前进和回退`Dep.target`，至于什么时候回退，下面的函数可以看到：

```javascript
// 此时的 Dep.target 是 渲染watcher，targetStack 是 [ 渲染watcher ] 
get () {
  pushTarget(this)
  let value
  const vm = this.vm
  try {
    value = this.getter.call(vm, vm)
  } finally {
    popTarget()
  }
  return value
}
```

解析：

+ 首先刚进去就是`pushTarget`，把`计算watcher`自身置为`Dep.target`，等待依赖收集。执行完`pushTarget(this)`之后，`Dep.target`变更为 `计算watcher`。此时的 Dep.target 是 计算watcher，targetStack 是 [ 渲染watcher，计算watcher ]

+ 然后会执行到 `value = this.getter.call(vm, vm)`，就是用户传入sum的函数

+ sum函数读取`this.count`，触发`count`的`get`劫持

  

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

+ 这里是调用`Dep.target.addDep(this)`去收集，绕回到`计算watcher`的`addDep`函数上去了

  ```javascript
  // watcher 的 addDep函数
  addDep (dep: Dep) {
    // 这里做了一系列的去重操作 简化掉 
    
    // 这里会把 count 的 dep 也存在自身的 deps 上
    this.deps.push(dep)
    // 又带着 watcher 自身作为参数
    // 回到 dep 的 addSub 函数了
    dep.addSub(this)
  }
  ```

+ 又回到`dep`上去了

  ```javascript
  class Dep {
    subs = []
  
    addSub (sub: Watcher) {
      this.subs.push(sub)
    }
  }
  ```

  这样就保存了`计算watcher`作为count的dep里的依赖了。

④ 经历了这样的收集流程后，此时的一些状态：

   1. sum的计算watcher

      ```javascript
      {
          deps: [ count的dep ],
          dirty: false, // 求值完了 所以是false
          value: 2, // 1 + 1 = 2
          getter: ƒ sum(),
          lazy: true
      }
      ```

      

   2. count的dep

      ```javascript
      {
          subs: [ sum的计算watcher ]
      }
      ```

此时求值结束，回到计算watcher的getter函数

```javascript
get () {
  pushTarget(this)
  let value
  const vm = this.vm
  try {
    value = this.getter.call(vm, vm)
  } finally {
    // 此时执行到这里了
    popTarget()
  }
  return value
}
```

执行到了`popTarget`，`计算watcher`出栈

---

##### Dep.target变更为渲染watcher

（此时的 Dep.target 是 渲染watcher，targetStack 是 [ 渲染watcher ] ）

上面的函数执行完毕，返回2这个value，此时对于sum属性的get访问还没结束

```javascript
Object.defineProperty(vm, 'sum', { 
    get() {
          // 此时函数执行到了这里
          if (Dep.target) {
            watcher.depend()
          }
          return watcher.value
        }
    }
})
```

此时的`Dep.target`当然是有值得，就是`渲染watcher`，所以进入了`watcher.depend()`逻辑，这一步相当关键

```javascript
// watcher.depend
depend () {
  let i = this.deps.length
  while (i--) {
    this.deps[i].depend()
  }
}
```

还记得刚刚的 `计算watcher` 的形态吗？它的deps里保存了count的dep。也就是说，又会调用 `count` 上的 `dep.depend()`

```javascript
class Dep {
  subs = []
  
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }
}
```

这次的 `Dep.target` 已经是 `渲染watcher` 了，所以这个 `count` 的 dep 又会把 `渲染watcher` 存放进自身的 `subs` 中。

`count的dep`:

```
{
    subs: [ sum的计算watcher，渲染watcher ]
}
```



那么来到了此题的重点，这时候 `count` 更新了，是如何去触发视图更新的呢？

再回到 `count` 的响应式劫持逻辑里去：

```javascript
// 在闭包中，会保留对于 count 这个 key 所定义的 dep
const dep = new Dep()

// 闭包中也会保留上一次 set 函数所设置的 val
let val

Object.defineProperty(vm, 'count', {
  set: function reactiveSetter (newVal) {
      val = newVal
      // 触发 count 的 dep 的 notify
      dep.notify()
    }
  })
})
```

好，这里触发了我们刚刚精心准备的 `count` 的 dep 的 `notify` 函数，感觉离成功越来越近了。

```javascript
class Dep {
  subs = []
  
  notify () {
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}
```

这里的逻辑就很简单了，把 `subs` 里保存的 watcher 依次去调用它们的 `update` 方法，也就是

1. 调用 `计算watcher` 的 update
2. 调用 `渲染watcher` 的 update

##### 计算watcher 的 update

```javascript
update () {
  if (this.lazy) {
    this.dirty = true
  }
}
```

wtf，就这么一句话…… 没错，就仅仅是把 `计算watcher` 的 `dirty` 属性置为 true，静静的等待下次读取即可。

##### 渲染watcher 的 update

这里其实就是调用 `vm._update(vm._render())` 这个函数，重新根据 `render` 函数生成的 `vnode` 去渲染视图了。

而在 `render` 的过程中，一定会访问到 `sum` 这个值，那么又回回到 `sum` 定义的 `get` 上：

```javascript
Object.defineProperty(vm, 'sum', { 
    get() {
        const watcher = this._computedWatchers && this._computedWatchers[key]
        if (watcher) {
          // ✨上一步中 dirty 已经置为 true, 所以会重新求值
          if (watcher.dirty) {
            watcher.evaluate()
          }
          if (Dep.target) {
            watcher.depend()
          }
          // 最后返回计算出来的值
          return watcher.value
        }
    }
})
```

由于上一步中的响应式属性更新，触发了 `计算 watcher` 的 `dirty` 更新为 true。 所以又会重新调用用户传入的 `sum` 函数计算出最新的值，页面上自然也就显示出了最新的值。

至此为止，整个计算属性更新的流程就结束了。



---

#### 总结

1. 初始化时：

   + 计算属性在初始化的时候，会实例一个计算watcher用来跟踪该计算属性，里面有一个字段dirty，这个字段表示是否需要重新求值，默认为true，因为第一次访问计算属性肯定是要求值的

   + 求值调用的是watcher.evaluate()函数，该函数会在求值完成之后，将dirty置为false，那么下次以同样的方式访问该属性时，就不需要重新求值 而是直接返回之前的值，这就是缓存的原理

2. 更新时：

   + 在读取计算属性的getter函数时，发现getter函数读取的是data里返回的响应式属性count，那么就会继续触发`count`的`get`劫持

     ```javascript
     sum(){
       return this.count + 1
     }
     ```

     

   + 在`count`的劫持里面，会收集所有count的依赖，根据响应式原理，在给count添加响应式劫持的时候，get用来收集所有的依赖，set用来通知依赖的更新，我们看看`notify()`函数做了什么

     ```javascript
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
       set: function reactiveSetter (newVal) {
           val = newVal
           // 触发 count 的 dep 的 notify
           dep.notify()
         }
       })
     })
     
     // notify()函数
     class Dep {
       subs = []
       
       notify () {
         for (let i = 0, l = subs.length; i < l; i++) {
           subs[i].update() // 把 subs 里保存的 watcher 依次去调用它们的 update 方法
         }
       }
     }
     
     ```

   + `notify()`函数主要就是遍历所有的subs依赖，依次去调用它们的`update`方法，对于计算属性来说，它的`update`方法就是将**<font style="color:red">`dirty`</font>**变成true

     ```javascript
     // 计算属性的update方法
     update () {
       if (this.lazy) {
         this.dirty = true
       }
     }
     ```

     **<font style="color:red">`dirty`变成true</font>**了之后，计算属性读取自身的getter函数，去执行watcher.evaluate()的时候，就会重新求值this.get()了，也就是实现了数据的更新

     ```javascript
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
     ```

---

#### 如何阻止缓存

计算属性有个cache字段，将cache变成false即可关闭Vue的计算属性缓存

```javascript
export function defineComputed (
  target: any,
  key: string,
  userDef: Object | Function
) {
  const shouldCache = !isServerRendering()
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = shouldCache
      ? createComputedGetter(key)
      : userDef
    sharedPropertyDefinition.set = noop
  } else {
      // 如果cache字段是false的话，直接执行get函数，就没有下面复杂的判断逻辑了
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
        `Computed property "${key}" was assigned to but it has no setter.`,
        this
      )
    }
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

---

#### 参考链接

https://cloud.tencent.com/developer/article/1614090