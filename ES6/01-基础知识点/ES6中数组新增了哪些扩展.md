#### ES6中数组新增了哪些扩展

1. 扩展运算符的应用：ES6通过扩展运算符`...`，好比是`rest`参数的逆运算，将一个数组转为用逗号分隔的参数序列

   ```javascript
   console.log(...[1,2,3])
   // 1 2 3
   
   console.log(1,...[2,3,4],5)
   // 1 2 3 4 5
   
   [...document.querySelectorAll('div')]
   // [div,div,div]
   ```

   + 可以将某些数据结构转为数组

     ```javascript
     [...document.querySelectorAll('div')]
     ```

   + 能够更加简单的实现数组的复制

     ```javascript
     const a1 = [1,2];
     const [...a2] = a1;
     // [1,2]
     ```

   + 数组的合并更加简洁

     ```javascript
     const arr1 = ['a','b']
     const arr2 = ['c']
     const arr3 = ['d','e']
     [...arr1,...arr2,...arr3] // ['a','b','c','d','e']
     ```

   + 与解构赋值结合起来，用于生成数组

     ```javascript
     const [first,...rest] = [1,2,3,4,5]
     first // 1
     rest  // [2,3,4,5]
     ```

   + 将字符串转为真正的数组

     ```javascript
     [...'hello']
     // ['h','e','l','l','o']
     ```

2. 构造函数新增的方法

   + `Array.from()`：将类数组结构转为数组

     ```javascript
     let arrayLike = {
        '0':'a',
        '1':'b',
        '2':'c',
        length:3
     }
     let arr2 = Array.from(arrayLike); // ['a','b','c']
     ```

   + `Array.of()`：将一组值转为数组

     ```javascript
     Array.of(3,8,11); // [3,8,11]
     ```

3. 实例对象新增的方法

   + `copyWithin(target,start,end)`：将指定位置的成员复制到其他位置，会覆盖原有成员，然后返回当前数组

   + `find()`、`findIndex()`

   + `fill()`：使用给定值，填充一个数组

     ```javascript
     new Array(3).fill(7)
     // [7,7,7]
     ```

   + `entries()`、`keys`、`values()`：对键值对的遍历、对键名的遍历、对键值的遍历

   + `includes()`：用于判断数组是否包含给定的值

     ```javascript
     [1,2,3].includes(2) // true
     ```

   + `flat()`、`flatMap()`：将数组扁平化处理，对原数据没有影响。`flat()`只会拉平一层，`flatMap()`会对数组的每个成员执行一个函数

4. 数组的空位：位置上没有值，在日常书写中需要避免空位

5. 排序稳定性：将`sort()`默认设置为稳定的排序算法