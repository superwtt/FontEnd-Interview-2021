#### ES6中函数新增了哪些扩展

1. 参数：ES6允许为函数的参数设置默认值

   ```javascript
   function log(x, y = 'World') {
     console.log(x, y);
   }
   
   console.log('Hello') // Hello World
   console.log('Hello', 'China') // Hello China
   console.log('Hello', '') // Hello
   ```

2. 属性：函数的length属性，返回没有指定默认值的参数个数；name属性返回函数名

3. 作用域

4. 严格模式：只要函数参数使用了默认值、解构赋值、或者扩展运算符，那么函数内部就不能显式设定为严格模式，否则会报错

5. 箭头函数