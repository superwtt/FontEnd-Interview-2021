#### KeepAlive的用法

常常会问：

1. 怎么缓存当前组件？
2. 组件缓存后怎么更新？
3. 说说你对keep-alive的理解是什么？

---

#### KeepAlive是什么

`keep-alive`是Vue的内置组件，能在组件切换过程中将状态保留在内存中。用它包裹动态组件时，会缓存不活动的组件实例而不是销毁它们

1. 在动态组件中的应用

   ```vue
   <keep-alive :include="whiteList" :exclude="blackList" :max="amount">
     <component :is="currentComponent"><component />  
   </keep-alive>
   ```

2. 在`vue-router`中的应用

   ```javascript
   <keep-alive :include="whiteList" :exclude="blackList" >
      <router-view></router-view>
   </keep-alive>
   ```

3. `include`定义缓存的白名单，`keep-alive`会缓存命中的组件，`exclude`定义缓存黑名单，被命中的组件将不会被缓存

---

#### 使用场景

比如有两个tab标签页，第一个tab标签我们操作了界面，再切换到第二个标签页，如果不使用<keep-alive>缓存的话，再切换回第一个标签页时，第一个标签页会重新渲染，而不会保留我们刚刚操作的痕迹

---

#### 源码

源码目录：`src/core/components/keep-alive.js`

① 组件的定义部分

```javascript
export default {
  name: 'keep-alive',
  abstract: true, // 判断当前组件虚拟dom是否渲染成真实dom的关键

  props: {
    include: patternTypes, // 缓存白名单
    exclude: patternTypes, // 缓存黑名单
    max: [String, Number] // 缓存的组件实例数量上限
  },

  created () {
    this.cache = Object.create(null) // 缓存虚拟dom
    this.keys = [] // 缓存的虚拟dom的健集合
  },

  destroyed () {
    for (const key in this.cache) { // 删除所有的缓存
      pruneCacheEntry(this.cache, key, this.keys)
    }
  },

  mounted () {
    // 实时监听黑白名单的变动
    this.$watch('include', val => {
      pruneCache(this, name => matches(val, name))
    })
    this.$watch('exclude', val => {
      pruneCache(this, name => !matches(val, name))
    })
  },

  render () {
    // 先省略...
  }
}
```

解析：

1. 组件的定义：
   + `keep-alive`与我们定义组件的过程一样，先是设置组件的名字为keep-alive
2. 组件的生命周期：keep-alive在它的生命周期里定义了三个钩子函数
   + `created`：初始化，缓存VNode和键值
   + `destroyed`：删除`this.cache`中缓存的VNode实例
   + `mounted`：在`mounted`这个钩子中对`include`和`exclude`参数进行实时监听

---

#### 总结

1. `keep-alive`是用来缓存动态组件的，假设我们有两个标签页，我们在第一个标签页做了操作，比如上拉加载等操作，再切换到第二个标签页。如果不做`keep-alive`缓存，当我们返回第一个标签页时，第一个标签页将会重新渲染并不会保留我们的操作痕迹
2. 原理分为两个方面：
   + 在`keep-alive`组件定义的时候，会初始化两个变量`cache`和`key`，用来存储需要缓存的组件和对应的key 值  。**与我们平时做缓存一样，当cache这个store里有这个组件时直接返回，没有的话存储进cache仓库，下次访问时再返回**
   + 至于如何将这个缓存的组件渲染到页面，关键在patch阶段，因为我们已经有了vnode，所以直接在patch阶段渲染就好
     + 在`keep-alive`定义的时候有个`abstract`属性默认为true表示该组件是抽象组件
     + 在Vue初始化建立父子组件关系的过程中，如果是抽象组件就会选取该抽象组件的上一级作为父级而忽略这个抽象组件
     + 在patch阶段，`insert(parentElm, vnode.elm, refElm)`，将缓存的DOM（vnode.elm）插入父元素中

---

#### 参考链接

[掘金](https://juejin.cn/post/6844903837770203144)



































