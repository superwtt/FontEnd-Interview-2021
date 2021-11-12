#### ES6中对象新增了哪些扩展

1. 属性的简写：ES6中，当对象键名与对应值名相等的时候，可以进行简写

   ```javascript
   const baz = {foo:foo}
   // 等同于
   const baz = {foo}
   
   // 方法也能简写
   ```

2. 属性名表达式：ES6允许字面量定义对象时，将表达式放在花括号内

   ```javascript
   const a = {
      'first world':'hello' 
   }
   let obj = {
      ['h'+'ello'](){
        return 'hi'  
      } 
   }
   ```

3. super关键字：指向当前对象的原型对象

   ```javascript
   const proto = {
     foo: 'hello'
   };
   
   const obj = {
     foo: 'world',
     find() {
       return super.foo;
     }
   };
   
   Object.setPrototypeOf(obj, proto); // 为obj设置原型对象
   obj.find() // "hello"
   ```

4. 扩展运算符的应用：在解构赋值中，未被读取的可遍历的属性，分配到指定的对象上面。解构是浅拷贝

   ```javascript
   let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
   x // 1
   y // 2
   z // { a: 3, b: 4 }
   ```

5. 属性的遍历：

   + `for...in`
   + `Object.keys`
   + `Object.getOwnPropertyNames`
   + `Object.getOwnPropertySymbols`
   + `Reflect.ownKeys`

6. 对象新增的方法

   + `Object.is()`：与严格比较运算符===的行为基本一致

   + `Object.assign()`：用于对象的合并，将源对象的所有可枚举属性复制到目标对象上，该方法是浅拷贝

     ```javascript
     const target = {a:1,b:2};
     const source1 = {b:2,c:2};
     Object.assign(target,source1);
     target // {a:1,b:2,c:3}
     ```

   + `Object.getOwnPropertyDescriptors()`：获取属性描述符

   + `Object.setPrototypeOf()`、``Object.getPrototypeOf()`：设置一个对象的原型对象，读取一个对象的原型对象

   + `Object.keys()`，`Object.values()`，`Object.entries()`：遍历键名、键值、键值对

   + `Object.fromEntries()`：用于将键值对数组转为对象

     ```javascript
     Object.fromEntries([
       ['foo','bar'],
       ['baz',42]  
     ])
     // {foo:'bar',baz:42}
     ```