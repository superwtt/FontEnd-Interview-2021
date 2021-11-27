#### Flex布局进阶

---

##### 父容器

1. 父容器设置换行方式：flex-wrap，决定子容器是否换行排列，不但可以顺序换行还支持逆序换行
   + `nowrap`：不换行
   + `wrap`：换行
   + `wrap-reverse `：逆序换行
2. 轴向与换行组合设置：flex-flow，相当于 flex-direction与flex-wrap的结合
   + `row`，`column`：设置主轴方向
   + `wrap`，`wrap-reverse`：设置换行模式
   + `row nowrap`、`column wrap`等，也可两者同时设置
3. 多行沿交叉轴对齐：align-content，当子容器多行排列时，设置行与行之间的对齐方式
   + `flex-start`：起始端对齐
   + `flex-end`：末尾端对齐
   + `center`：居中对齐
   + `space-around`：等边距均匀分布
   + `space-between`：等间距均匀分布
   + `stretch`：拉伸对齐

---

##### 子容器

1. 设置基准大小：flex-basis，表示在不伸缩的情况下，子容器的原始尺寸。主轴为横向时，代表宽度，主轴为纵向时，代表高度
2. 设置扩展比例：flex-grow
3. 设置伸缩比例：flex-shrink

---

##### 总结

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/flex/容器的概念.png)

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/flex/轴的概念.png)

---

##### 面试题常考

flex: 0 1 auto代表什么意思？

<font style="color:red">`flex：flex-grow|flex-shrink|flex-basis`</font>

代表，该子容器，不设置扩展比例，设置伸缩比例为1:1伸缩，基本大小为自动大小

---

##### 参考链接

[very good](https://juejin.im/post/58e3a5a0a0bb9f0069fc16bb#heading-4)