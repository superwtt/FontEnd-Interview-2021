#### ES6中 Decorator 

`Decorator`即装饰器，在不改变原类和使用继承的情况下，动态地扩展对象功能。本质上也不是什么高大上的结构，就是一个普通的函数用于扩展属性和类方法



定义一个士兵，这时候他什么装备也没有

```javascript
class soldier{}
```



定义一个得到AK装备的函数，即装饰器

```javascript
function strong(target){
   target.AK = true 
}
```



使用该装饰器对士兵进行增强

```javascript
@strong
class soldier{}
```



这时候士兵就有武器了

```javascript
solider.AK // true
```

---

#### 使用场景

```javascript
class MyReactComponent extends React.Component {}

export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);

// 等同于
@connect(mapStateToProps, mapDispatchToProps)
export default class MyReactComponent extends React.Component {}
```

---

#### 参考链接

https://github.com/febobo/web-interview/issues/44