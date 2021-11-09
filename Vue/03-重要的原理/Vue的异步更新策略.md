#### Vue的异步更新策略

Vue在更新DOM时是异步执行的，只要侦听到数据变化，Vue将开启一个队列，去缓存同一个事件循环中发生的所有数据变更。如果同一个watcher被多次触发，只会被推入到队列中一次

---

#### 具体流程

① 当数据发生变化时

```javascript
// 当一个Data更新时，会依次执行以下代码
// 1.0 触发Data.set
// 2.0 调用dep.notify() 
// 3.0 Dep会遍历所有相关的watcher执行update方法
class Watcher {
  // 4. 执行更新操作
  update() {
     /* istanbul ignore else */
    if (this.lazy) {
      this.dirty = true
    } else if (this.sync) {
      /*同步则执行run直接渲染视图*/
      // 基本不会用到sync
      this.run()
    } else {
      /*异步推送到观察者队列中，由调度者调用。*/
      queueWatcher(this)
    }
  }
}

const queue = [];

function queueWatcher(watcher: Watcher) {
  /*获取watcher的id*/
  const id = watcher.id
  /*检验id是否存在，已经存在则直接跳过，不存在则标记哈希表has，用于下次检验*/
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
       // 5. 将当前 Watcher 添加到异步队列
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      let i = queue.length - 1
      while (i >= 0 && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(Math.max(i, index) + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      waiting = true
    // 6. 执行异步队列，并传入回调  
      nextTick(flushSchedulerQueue)
    }
  }  
}


```

解析：

+ `watcher`在`update`的时候，调用了一个`queueWatcher`方法，将`watcher`异步推送到观察者队列中；
+ 调用`queueWatcher`的时候，首先查看了`watcher`的id，保证相同的`watcher`不会被多次推入;
+ `waiting`是一个标识，如果没有在`waiting`状态，就异步执行`flushSchedulerQueue`方法

---

② `flushSchedulerQueue`

`flushScheduler`是下一个tick（tick是指宏任务）的时候的回调函数，主要就是去执行watcher中的run方法更新视图

```javascript
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

③ 流程图如下：

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/更新流程.png)

解析：

+ Vue在调用`Watcher`更新视图时，并不会直接进行更新，而是把需要更新的`Watcher`加入到`Queue`队列里，然后把具体更新的方法`flushSchedulerQueue`传给nextTick进行调用

---

#### 操作真实DOM

看一下Vue中文教程里的代码：

```vue
<div id="example">{{message}}</div>

var vm = new Vue({
  el:"#example",
  data:{
  message:"123"
 }
})
vm.message = 'new message'// 更改数据
vm.$el.textContent === 'new message' // false
Vue.nextTick(function () {
  vm.$el.textContent === 'new message' // true
})
```

解析：

+ 当设置`vm.mssage = 'new message'`的时候，该组件并不会立即进行渲染，而是在dep.notify()了watcher之后，被异步推到了调度者队列中，等`nextTick`的时候进行更新
+ `nextTick`则是去尝试原生的`Promise,then`和`MutationObserver`，如果都不支持，会采用`setTimeout()进行异步更新`

---

#### 总结

Vue中更新DOM是异步的：

1. 原因：如果DOM的更新是同步的，数据一变化就里面通知DOM更新，将会产生大量的视图渲染，应用的性能也会下降
2. 流程：当Vue中数据发生变化时
   + 触发Data.set()
   + 调用dep.notify()
   +  Dep 会遍历所有相关的 Watcher 执行 update 方法
   + update方法调用的是queueWatcher，将watcher推入更新队列









