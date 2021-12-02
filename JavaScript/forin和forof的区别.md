#### for...in

for...in 循环只遍历可枚举属性，包括它原型链上的可枚举属性。循环将遍历对象：

+ 本身的所有可枚举属性
+ 对象从其构造函数原型上继承的属性
+ 新添加的属性

```javascript
var obj = {a:1,b:2,c:3}
for(let key in obj){
  console.log(key)  
}
// a b c
```



<font color=red>**对于for...in能把对象原型链上的属性也遍历出来的特性，我们可以使用hasOwnProperty去解决**</font>

```javascript
// 原型链上添加方法 contains 和 属性subTitle
Object.prototype.contains = function () {return null}
Object.prototype.subTitle = 'Flutter'

let obj = {
  a:1,
  b:2,
  c:3
}
obj.d = "DDD"
for (let i in obj) {
  console.log(i)
}
// a
// b
// c
// DDD
// contains
// subTitle

// 通过hasOwnProperty解决 指示对象自身属性中是否具有指定的属性
for (let i in obj) {
 if(obj.hasOwnProperty(i)){
    console.log(i); // a b c
 }
}
```





---

#### for...of

for...of 语句在可迭代对象，包括Array、Map、Set、String、Arguments、TypeArray等，上创建一个迭代循环，调用自定义迭代钩子，为每个不同的属性执行语句。它不会遍历到原型链上的属性，只是去获取数组的值

```javascript
const array1 = ['a','b','c']
for(const val of array1){
   console.log(val); 
}
// a b c
```

`for...of`不可以遍历普通对象，想要遍历对象的属性，可以用for...in或者内置的Object.keys()方法



---

#### for...in和for...of的区别

1. `for...in`以任意顺序迭代对象的可枚举属性

2. `for...of`语句遍历可迭代对象定义要迭代的数据

   ```javascript
   Object.prototype.objCustom = function() {}; 
   Array.prototype.arrCustom = function() {};
   
   let iterable = [3, 5, 7];
   iterable.foo = 'hello';
   
   for (let i in iterable) {
     console.log(i); // 0, 1, 2, "foo", "arrCustom", "objCustom"
   }
   
   for (let i in iterable) {
     if (iterable.hasOwnProperty(i)) {
       console.log(i); // 0, 1, 2, "foo"
     }
   }
   
   for (let i of iterable) {
     console.log(i); // logs 3, 5, 7
   }
   ```

   

---

#### 总结

for...in一般用来迭代对象的可枚举属性，包括原型链继承过来的都可以迭代到

for...of一般用来遍历数组、类数组对象、Set、Map等可迭代数据，遍历得到的是数组的元素