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

1. `Map`结构的属性名可以是任意的，比如`Object`和`Array`也都是可以的，而`Object`的属性名只能是整数、字符串、或者是Symbol类型

2. 同名碰撞

   + `Map`不会覆盖同名属性
   + `Object`会覆盖同名属性

3. 可迭代性

   + `Map`中的所有元素可以通过`for...of`方法或者使用内置的`forEach`遍历
   + `Object`使用`for...in`方法或者`Object.keys()`

4. 长度

   + `Map`可以直接拿到长度，而`Object`不行

     ```javascript
     let m = new Map()
     m.set({a:1}, 'hello,world')//dom对象作为键
     m.set(['username'],'jack')//数组作为键
     m.set(true,1)//boolean类型作为键
     
     console.log(m.size)//3
     ```

5. 有序性

   + 填入Map的元素会保持原有的顺序，而Object无法做到。

   + Object的key排序规则如下：

     + 如果key是整数（0,1,2,3-1，-2，-3）或者整数类型的字符串（如：”123“）,那么会按照<font color=red>**从小到大**</font>的顺序排序，比如：

       ```javascript
       var obj = {
         '-1': '全部',
         '0' : '正常',
         '1' : '失效'
       };
       // 0 1 -1
       ```

       

     + 如果key中除了整数或者整数类型的字符串以外，还有其他数据类型，则整数放在最前面，比如：

       ```javascript
       var obj = {
         'a': 111,
         '我' : 222,
         '1' : 333,
         '1.3': 444,
         '3': 555
       };
       // 1 3 a 我 1.3
       ```

       

     + 其他数据类型，都按照对象 key 的实际创建顺序排序

     + <font color=blue>总结排序规则：`Object.keys`在执行过程中，如果发现key是整数类型索引，那么它首先按照从小到大排序加入，然后再按照先来先到的创建顺序加入其它元素，最后加入`Symbol`类型的key</font>

6. 可展开

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

     

---

#### 如何选择使用

如何选择使用`Object`和`Map`取决于我们要使用的数据类型以及操作

1. 如果我们知道所有的key，它们都是整数、字符串或者Symbol类型，我们需要一个简单的结构去存储这些数据，`Object`是一个很好的选择

2. 如果需要考虑元素的顺序并且我们需要做一些简单的增删改查操作时选择Map更好。
3. Map在存储大量数据的场景下表现更好，尤其在key为未知的状态。