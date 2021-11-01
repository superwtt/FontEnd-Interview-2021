#### MVVM模式

 ![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/Vue/viewmodel.png)

---

##### MVVM的定义

MVVM是`Model-View-ViewModel`的缩写，是一种前端开发的架构模式，核心是提供View层和ViewModel层的双向数据绑定，这使得ViewModel状态的改变可以自动传递给View，View层的操作也能自动更新数据，即所谓的数据双向绑定。ViewModel负责连接View和Model，保证视图和数据的一致性



Vue.js是一个提供了`MVVM`风格的双向绑定的JavaScript库，这种轻量级的架构让前端开发更加高效、便捷

---

##### 优点

`MVVM`模式简化了界面与业务的依赖，解决了数据频繁更新带来的操作DOM的性能问题。开发者只需要专注对数据的维护即可，而不需要自己操作DOM

---

##### 总结

1. MVVM是一种前端开发模式，核心是View层和ViewModel层的数据双向绑定：View层也就是视图层操作数据能够更新到ViewModel层，ViewModel层数据的改变也能更新到视图层
2. Vue框架采用的就是这种模式，让我们只需要专注维护数据而不需要自己操作DOM