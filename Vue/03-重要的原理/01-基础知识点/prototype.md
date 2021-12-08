#### prototype指针、原型对象

1. 无论什么时候，只要我们创建一个新的函数，JS引擎就会根据一组特定的规则为该函数创建一个 prototype 属性（每个函数都有一个prototype属性），这个属性指向构造函数的原型对象

2. <font color=red>prototype 是一个指针，指向一个对象，该对象称为原型对象；原型对象存储着由该构造函数创建的实例所共享的属性和方法</font>

3. 原型是什么：每个JavaScript对象，除了null以外，在创建的时候都会与之关联另一个对象，这个对象就是我们所说的原型，每个对象都会从原型 “继承” 属性和方法

   ```javascript
   function Person(){}
   Person.prototype.name = "wtt"
   
   var p1 = new Person();
   var p2 = new Person();
   
   console.log(p1.name); // wtt
   console.log(p2.name); // wtt
   ```

   其中，name属性放在了 prototype 所指向的原型对象上面，所以无论Person生成多少个实例，他们都会拥有一个 name = "wtt"的属性

   我们用一个简单的示意图来表示构造函数，prototype和原型对象之间的关系：

   ![avatar](https://oscimg.oschina.net/oscnet/d2f167b48e64cf108ce9428c7fc92254441.jpg)

---

####  <font color=VioletRed >__ proto __</font>

1. 当我们调用构造函数，生成一个实例之后，该实例内部将包含一个__ proto __指针，指向它所在的构造函数的原型对象

2. __ proto __用来表示的是实例和原型对象的关系

3. 原型对象也是对象，是通过 Object 构造函数生成的，所以原型对象上也会有一个__ proto __指针，指向它的原型。

4. 一个函数既有可能有prototype属性，又可能有__ proto __属性，表示这个函数的原型对象是另一个构造函数的实例。常见的有：

   ```javascript
   Person.prototype._proto_ === Object.prototype
   ```

   我们更新一下示意图：

   ![avatar](https://oscimg.oschina.net/oscnet/9decec1b093aa16bd6746d21b94cfcabc62.jpg)

​    代码示例如下：

    ```javascript
function Person(){}
var p1 = new Person();
p1._proto_ === Person.prototype; // true
    ```

<font color=blue>通过以上分析我们知道，构造函数或者实例都有指针指向原型对象，那原型对象上有指针指向构造函数吗？</font>

---

####  <font color=MediumOrchid >constructor</font>

1. 在默认情况下，所有的原型对象都会自动获得一个`constructor`构造函数属性，这个属性指向prototype所在的构造函数，即从原型对象上也能有指针指向构造函数了

2. `Person.prototype.constructor === Person` ; // true

   我们更新一下示意图

   ![](https://oscimg.oschina.net/oscnet/9a5858d50e9b84670618ca28ac4b169f679.jpg)

从示意图我们可以得出：

```javascript
person._proto_ === Person.prototype
Person.prototype.constructor === Person
```

---

#### <font color=DarkMagenta>实例与原型之间的关系</font>

1. 当读取实例属性的时候，如果找不到，就会去实例对象关联的原型对象中查找是否有相关属性；如果有，则返回该属性的值；如果没有就会沿着原型链继续向上查找，直到找到最顶层为止。如果最后都没找到，返回null
2. 构造函数的原型对象，是由 Object 创造的，所以最后会去 Object 的原型对象上查找，如果还是没有查到，就会返回null，即`Object.prototype._proto_ === null`
3. `Person.prototype._proto_._proto_` => null，其中，`Person.prototype`是原型，原型上的__ proto __ 指向原型的原型，这里是`Object.prototype`，而`Object.prototype._proto_===null`

---

#### <font color=MediumOrchid3 >原型链</font>

1. 沿着原型对象查找的过程，依赖于原型链
2. 原型链示意图如下：红色部分就形成一条原型链

![](https://oscimg.oschina.net/oscnet/ed51385d4e238a3adacf38fe33f3e7fbd40.jpg)

---

#### <font color=green >判断方法</font>

1. `instance of`

2. `Object.prototype.isPrototypeOf()`

3. `Object.getPrototypeOf()`

   ```javascript
   function Person(){
      this.name = "wtt"
      this.age = 12
   }
   var p1 = new Person();
   console.log(p1 instanceof Person); // true
   console.log(Person.prototype.isPrototypeof(p1)); // true
   console.log(Object.getPrototypeOf(p1)); // {constructor:f()}
   ```

4. `p1.constrcutor===Person.prototype.constructor`：因为 p1 上没有 constructor 这个属性，当我们访问这个属性的时候，就会沿着它的原型链向上查找，到 Person.prototype 上发现有一个 constructor属性，于是输出 Person

5. 控制台输出如下：

   ![](https://oscimg.oschina.net/oscnet/d258ac1fa45ae77a1cb887c87940ee3871b.jpg)

   解析：

   + Person.prototype打印的是Person这个构造函数的原型对象
   + Person.prototype.constructor打印的是prototype所在的构造函数
   + Person.prototype.__ proto __打印的是构造函数Person所指向的原型对象
   + Person.prototype._ proto __.constructor打印的是Person这个构造函数所指向的原型对象的构造函数，这里是Object构造函数
   + Person.prototype.__ proto __ .__ proto __打印的是Object的原型对象，此时是null
   + JavaScript中，所有的对象都继承自Object，即那些对象都是Object对象的实例

   

---

#### 面试题常问

方法写在Object上，与写在Object.prototype上有什么区别（方法写在构造函数上与写在原型对象上有什么区别）？

+ <font color=Brown2 >实例方法、静态方法：直接定义在类/构造函数上的，是静态方法，通过类/构造函数直接能够访问，定义在prototype上的叫做实例方法，只能通过创建实例访问，并且每次创建实例的时候都会实例化一次</font>


+ <font color=PeachPuff3 >为什么有的写成实例方法，有的写成静态方法：有些方法是不需要实例化的，只要类就存在，比如Array.isArray()，如果我们写在原型上，只有数组的实例才会有，那么就没有判断的意义了。一般用作Helper函数，不需要操作类中属性的，写成静态方法。需要访问类内部属性的，写成实例方法</font>


+ <font color=Brown1 >同样是实例方法，写在类/构造函数内部和写在原型对象上有什么区别：定义在类/构造函数内部的方法，会在每一个实例上克隆（实例化）一遍；定义在原型对象上的方法，会让所有实例共享这个属性，不会在每个实例内部重新定义这个方法</font>

+ <font color=PaleVioletRed >如果我们需要创建很多个对象，对象上还有许多方法，那么为了节省内存，我们可以把这些方法都定义在prototype指向的原型对象上。区别就在于共享和每个实例都创建一份 </font>

+ <font color=RosyBrown1 >如果类/构造函数中有一些方法，我们既不想让所有实例都实例化一遍，又不想让实例共享，那么我们就只能把它写成静态的方法。</font>

+ ES5写法如下：

  ```JavaScript
  function Person(){
      this.getAge = function(){};  // 实例方法
      this.getName = function(){}; // 实例方法
  }
  Person.prototype.getSchool = function(){}; // 原型方法
  Person.getInfo = function(){}; // 静态方法
  ```

  ES6写法如下：

  ```JavaScript
  class Person(){
      static getName(){
          // 静态方法
      }
      getAge(){
          // 实例方法
      }
  }
  ```

---

#### <font color=red >注意点</font>

1. __ proto __ 与prototype之间的关系：__ proto __ 是指[[prototype]]，因为现在还没有标准的方法能够访问到[[prototype]]，但是主流浏览器都支持__ proto __ ，所以我们用__ proto __ 来确定实例和实例的原型之间的关系

   