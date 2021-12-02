#### scoped原理

---

##### 什么是scoped

在vue文件中的style标签上，有一个特殊的属性：scoped，当一个style标签拥有scoped属性时，它的css样式就只能作用于当前的组件，也就是说，该样式只能适用于当前组件的元素。

通过属性，可以使得组件之间的样式不互相污染，如果一个项目中的所有style标签全部加上了scoped，相当于实现了样式的模块化，使得我们在各组件中定义的css不产生污染

---

##### scoped工作原理

比如我们的一个button组件，我们定义了如下scoped样式，给按钮添加一个边框弧度：

```vue
<template>
<button class="button"></button>
</template>
<style scoped>
.button{
 border-radius:15px;         
}
</style>
```

这样最终渲染出来的页面代码是这样的：可以看到分别给HTML标签和CSS选择器都加上了`data-v-xxx`属性

```vue
<button class="button" data-v-123lsdfasi32>
  text
</button>
.button[data-v-123lsdfasi32]{
  border-radius: 1px;
}
```

---

##### scoped实现原理

<font color=red>**每个vue文件都将对应一个唯一的id，该id根据文件路径名和内容hash生成，通过组合形成scopeid。**</font>

编译 template 标签时，会为每个标签添加当前组件的scopedId，如：

```vue
<div class="demo">test</div>
// 会被编译成：
<div class="demo" data-v-12e4e11e>test</div>
```

编译 style 标签时，会根据当前组件的 scopedId 通过属性选择器和组合选择器输出样式，如：

```css
.demo{color:red}
.demo[data-v-12e4e11e]{color:red}
```

这样就相当于为我们配置的样式加上了一个唯一的标识



但是，有两个问题：

1. 渲染的 HTML 标签上的 data-v-xxx 属性是如何生成的？

   vnode是描述组件对应的HTML标签和结构，一个vnode包含了渲染DOM节点需要的基本属性，当然这个属性包含了scopeId。

   在模板编译器将模板生成vnode阶段，会对配置属性进行处理，也就是在这个过程中，scopeid被解析到vnode的配置属性。然后在render阶段执行时调用createElement，作为vnode的原始属性，渲染成DOM节点上。

2. CSS 代码中添加的属性选择器是如何实现的？

   同一个单页面组件中的style，与templateLoader中的scopeid保持一致

   通过 PostCSS 解析 style 标签内容，同时通过scopedPlugin 为每个选择器追加一个[scopid]的属性选择器

---

##### 使用注意点

1. 样式穿透



---

##### 总结

1. scoped的作用是隔离样式作用域，达到一个css效果不互相污染的效果
2. scopeid的生成原理是，根据vue文件的路径和内容生成的一串hash值，通过templateLoader添加到DOM上，通过styleLoader添加到css上

---

#### 参考链接

[Good](https://juejin.cn/post/6991354556349153293)

https://blog.csdn.net/qq_42049445/article/details/114965579





































