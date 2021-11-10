#### this.$nextTick原理

Vue中数据的更新是异步的，有时候我们修改完数据之后，不能立马获取到更新后的DOM，这时候`this.$nextTick`就派上用场



我们可以简单理解为，当页面中的数据发生改变，就会把该任务放到一个异步队列中，只有在当前任务空闲时才会进行DOM渲染，当DOM渲染完成之后，该回调函数会自动执行

---

#### 原理

JS的事件循环和任务队列是理解 nextTick 的关键

---

#### JS的事件循环机制

1. 首先我们要知道

   + JavaScript是单线程语言
   + 事件循环机制EeentLoop是JavaScript的执行机制

2. JS的执行顺序

   + 同步和异步分别进入两个不同的执行场所，同步的进入主线程，异步的进入Event Table并注册回调函数

   + 同步和异步同时进行

   + 异步队列中，当指定的事情完成时，EventTable会把这个函数移入`EventQueue`

   + 在一次事件循环中，异步事件返回的结果，会被放入一个任务队列中，根据异步事件的类型，需要把事件放入到对应的微任务或宏任务队列

   + 当主线程空闲时，即执行栈为空，主线程会先查看微任务队列，执行清空，继续下一轮的事件循环

     ![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/js的事件循环机制.png)

---

#### Vue的异步更新策略

Vue实现响应式的DOM更新并不是数据发生变化之后DOM立即变化，而是按照一个策略进行的DOM更新，这就使得我们并不能在数据更新之后立马拿到更新后的DOM

至于为什么设计成异步的，其实很好理解：如果是同步的，当我们频繁地去改变状态值，会导致DOM不停地更新，这显然是不行的



当Data数据更新时，流程如下：

1. Data更新
2. 触发`Data.set`
3. 调用`dep.notify`
4. `Dep`会遍历所有相关的`Watcher`执行update方法
5. 将当前的watcher添加到异步队列`queueWatcher`
6. `queueWatcher`执行异步队列，并调用nextTick，将更新视图的方法当做参数传入

```javascript
// 当一个 Data 更新时，会依次执行以下代码
// 1. 触发 Data.set
// 2. 调用 dep.notify
// 3. Dep 会遍历所有相关的 Watcher 执行 update 方法
class Watcher {
  // 4. 执行更新操作
  update() {
    queueWatcher(this);
  }
}

const queue = [];

function queueWatcher(watcher: Watcher) {
  // 5. 将当前 Watcher 添加到异步队列
  queue.push(watcher);
  // 6. 执行异步队列，并传入更新视图的回调
  nextTick(flushSchedulerQueue);
}

// 更新视图的具体方法
function flushSchedulerQueue() {
  let watcher, id;
  // 排序，先渲染父节点，再渲染子节点
  // 这样可以避免不必要的子节点渲染，如：父节点中 v-if 为 false 的子节点，就不用渲染了
  queue.sort((a, b) => a.id - b.id);
  // 遍历所有 Watcher 进行批量更新。
  for (index = 0; index < queue.length; index++) {
    watcher = queue[index];
    // 更新 DOM
    watcher.run();
  }
}
```

---

#### nextTick的具体实现

nextTick函数的实现流程如下：

1. 第一步：将传给this.$nextTick的回调压入一个`callbacks`数组

2. 第二步：核心的异步延迟函数`timerFunc`，根据环境的不同，确认`timerFunc`是微任务还是宏任务，谷歌环境下肯定就是微任务了`timerFunc`有三种polyfill方式：
   + `Promise` 
   + `MutationObserver` 
   + `setImmediate` 
   + `setTimeout`

3. **`nextTick`方法最后是返回一个函数，将传给 `nextTick(flushSchedulerQueue)`的参数也就是视图更新先执行完，然后再调用`timerFunc()`函数，也就是传递给`this.$nextTick`获取最新DOM的操作.**

   `nextTick`涉及到两个回调：

   + 一个是queueWatcher的更新方法中，将更新视图的方法传递给nextTick，`nextTick(flushSchedulerQueue)`
   + 第二个是调用`this.$nextTick`方法传递的回调，会在第一步之后执行，所以总是能获取最新的DOM

```javascript
// 第一步：将this.$nextTick回调函数压入callbacks数组
export const nextTick = (function () {
  const callbacks = []
  let pending = false
  let timerFunc

  function nextTickHandler () {
    pending = false
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      copies[i]()
    }
  }
   
  // timerFunc的异步策略  
  if (typeof setImmediate !== 'undefined' && isNative(setImmediate)) {
    timerFunc = () => {
      setImmediate(nextTickHandler)
    }
  } else if (typeof MessageChannel !== 'undefined' && (
    isNative(MessageChannel) ||
    // PhantomJS
    MessageChannel.toString() === '[object MessageChannelConstructor]'
  )) {
    const channel = new MessageChannel()
    const port = channel.port2
    channel.port1.onmessage = nextTickHandler
    timerFunc = () => {
      port.postMessage(1)
    }
  } else
  /* istanbul ignore next */
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    // use microtask in non-DOM environments, e.g. Weex
    const p = Promise.resolve()
    timerFunc = () => {
      p.then(nextTickHandler)
    }
  } else {
    // fallback to setTimeout
    timerFunc = () => {
      setTimeout(nextTickHandler, 0)
    }
  }
  
  // 返回的函数，先执行视图更新，再执行this.$nextTick的回调
  return function queueNextTick (cb?: Function, ctx?: Object) {
    let _resolve
    callbacks.push(() => {
      if (cb) {
        try {
          cb.call(ctx)
        } catch (e) {
          handleError(e, ctx, 'nextTick')
        }
      } else if (_resolve) {
        _resolve(ctx)
      }
    })
    if (!pending) {
      pending = true
      timerFunc()
    }
    // $flow-disable-line
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise((resolve, reject) => {
        _resolve = resolve
      })
    }
  }
})()
```

---

#### 流程总结

当Vue中的数据发生变化时，为什么能在$nextTick里面获取到最新的DOM？

1. Vue中DOM的更新是异步的

2. nextTick的执行时机，涉及到JS的事件循环机制（在一个事件循环中发生的所有数据的改变都会在下一个事件循环的tick中来触发视图的更新）

   

**Vue更新之后的响应流程如下：**

Data更新 -> Data.set -> dep.notify -> watcher.update -> 将当前watcher添加到watcherQueue，有相同id的去除 -> 调用nextTick方法

**nextTick的执行时机**

由Vue的更新流程可知，Data触发的更新压入WatcherQueue，然后**`调用一次nextTick`**，把具体的更新方法 `flushSchedulerQueue `传给 `nextTick`进行调用。

`nextTick`内部会先将更新方法执行完，微任务队列先放置了一个DOM更新，再调用`timerFunc()`函数执行回调，微任务队列又放置了一个cb函数，此时微任务队列是**[DOM更新，cb回调]**

根据先来后到原则，Data的更新执行完生成了新的DOM，接下来执行`this.nextTick`的回调函数时，就能获取到更新后的DOM

---

#### 参考链接

[Good](https://segmentfault.com/a/1190000023649590)

[Good](https://segmentfault.com/a/1190000018328525)