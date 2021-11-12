#### 说说var、let、const之间的区别

1. 变量提升：

   + var声明的变量存在变量提升，在声明之前调用值为undefined
   + let和const声明的变量不存在变量提升，在声明之前调用会报错

   ```javascript
   // var
   console.log(a); // undefined
   var a = 10;
   
   // let
   console.log(b) // cannot access 'b' before initialization
   let b = 10
   
   // const
   console.log(c);// cannot access 'b' before initialization
   const c = 10
   ```

2. 暂时性死区

   + var 不存在暂时性死区
   + let和const存在暂时性死区，只有等到变量的那一行代码出现，才可以获取和使用该变量

   ```javascript
   // var
   console.log(a)  // undefined
   var a = 10
   
   // let
   console.log(b)  // Cannot access 'b' before initialization
   let b = 10
   
   // const
   console.log(c)  // Cannot access 'c' before initialization
   const c = 10
   ```

3. 块级作用域

   + var声明的变量不存在块级作用域
   + let和const声明的变量存在块级作用域

   ```javascript
   // var
   {
       var a = 20;
   }
   console.log(a); // 20
   
   // let
   {
       let b = 20
   }
   console.log(b)  // Uncaught ReferenceError: b is not defined
   
   // const
   {
       const c = 20
   }
   console.log(c)  // Uncaught ReferenceError: c is not defined
   ```

4. 重复声明

   + var 允许重复声明变量，后面会覆盖前面的
   + let和const不允许重复声明变量

   ```javascript
   // var
   var a = 10
   var a = 20 // 20
   
   // let
   let b = 10
   let b = 20 // Identifier 'b' has already been declared
   
   // const
   const c = 10
   const c = 20 // Identifier 'c' has already been declared
   ```

5. 修改声明的变量

   + var和let可以修改声明的变量
   + const声明一个只读的常量，一旦声明，常量的值就不能改变

   ```javascript
   // var
   var a = 10;
   a = 20;
   console.log(a) // 20
   
   // let
   let b = 10
   b = 20
   console.log(b) // 20
   
   // const
   const c = 10
   c= 20
   console.log(c) // Uncaught TypeError: Assignment to constant variable
   ```

6. 使用：常量尽量使用const，其他情况大多数使用let，避免使用var

<font style="color:red">**总结：变量提升、块级作用域、暂时性死区、重复声明、修改声明的变量**</font>

---

