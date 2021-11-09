#### Vue的双向绑定原理

##### 定义

数据变化后能够更新视图，视图变化后能够更新数据

---

##### 基本使用

```vue
<template>
  <div>
    <p>
      {{name}}
      <button @click="changeName">修改名字</button>
    </p>
    <p>
      <input type="text" v-model="name">
    </p>
  </div>
</template>
<script>
  export default{
    data(){
      return {
        name:"这是初始时候的名字"
      }
    },
    methods:{
      changeName(){
         this.name = "这是修改过后的名字"
      }
    }
   }
<script>
```

解析：

+ 当我们点击按钮模拟用户操作界面时，data数据的改变能够同步到视图 -> view to model
+ 当我们通过input直接修改数据时，数据也能映射到视图 -> model to view

这就是双向绑定

---

##### 具体实现

在响应式原理的实现中，当我们编译静态文件时，会区分出是不是指令模式

```javascript
//model的更新
  modelUpdater({ vm, node, exp }) {
    node.value = vm.$data[exp];
    node.addEventListener("input", (e) => {
      this.vm.$data[exp] = e.target.value;
    });
  }
```

解析：

1. 监听input事件，将改变后的值赋值给data中的数据
2. data中的数据是响应式的，当它发生变化接收到新的值时，会去notify通知所有的依赖进行更新

---

##### 总结

1. Vue的双向绑定指的是数据的改变能够通知到视图，视图的改变也能更新到数据。
2. 其中的原理是监听输入框的input事件+Vue的响应式数据实现
