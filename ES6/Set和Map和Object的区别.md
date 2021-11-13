#### 怎么理解ES6新增的Set/Map两种数据结构

---

#### Set

`Set`是ES6新增的数据结构，类似于数组，但是成员的值都是唯一的，没有重复的值，<font style="color:red">**我们一般称为集合**</font>

1. `Set`的增删改查
   + `add()`
   + `delete()`
   + `has()`
   + `clear()`
2. `Set`的遍历
   + `keys()`
   + `values()`
   + `entries()`
   + `forEach()`

<font style="color:red">**总结：Set是一种没有重复数据的集合**</font>

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/ES6/set数据结构.png)

---

#### Map

`Map`类型是键值对的有序列表，键和值都可以是任意类型。`Map`本身是一个构造函数，用来生成`Map数据结构`

1. 增删改查

   + `size属性`：返回Map结构的成员总数

     ```javascript
     const map = new Map();
     map.set('foo',true);
     map.set('bar',false);
     
     map.size // 2
     ```

   + `set()`：设置键名key对应的键值为value，然后返回整个Map结构

   + `get()`读取key对应的键值

   + `has()`：表示某个键是否在当前Map对象中

   + `delete()`：删除某个键

   + `clear()`：清空Map结构

2. 遍历

   + `keys()`
   + `values()`
   + `entries()`
   + `forEach()`

<font style="color:red">**总结：Map类似于对象Object，有自己的键名和键值，只不过键名可以任意数据结构**</font>

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/ES6/map数据结构.png)

---

#### 与Object的区别

1. 同名碰撞

   + `Map`不会覆盖同名属性
   + `Object`会覆盖同名属性

2. 可迭代性

   + `Map`中的所有元素可以通过`for...of`方法或者使用内置的`forEach`遍历
   + `Object`使用`for...in`方法或者`Object.keys()`

3. 长度

   + `Map`可以直接拿到长度，而`Object`不行

     ```javascript
     let m = new Map()
     m.set({a:1}, 'hello,world')//dom对象作为键
     m.set(['username'],'jack')//数组作为键
     m.set(true,1)//boolean类型作为键
     
     console.log(m.size)//3
     ```

4. 有序性

   + 填入Map的元素会保持原有的顺序，而Object无法做到

5. 可展开

   + `Map`可以使用省略号语法展开，而`Object`不行

     ```javascript
     let m = new Map()
     m.set({a:1}, 'hello,world')//dom对象作为键
     m.set(['username'],'jack')//数组作为键
     m.set(true,1)//boolean类型作为键
     
     console.log([...m])//可以展开为二维数组
     
     let obj = new Object()
     obj['jack'] =  1
     obj[0] = 2
     obj[5] = 3
     obj['tom'] = 4
     console.log([...obj])//TypeError: obj is not iterable
     ```

     



