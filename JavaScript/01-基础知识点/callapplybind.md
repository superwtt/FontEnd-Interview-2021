#### call / apply / bind

1. `Object.prototype.toString.call`：判断变量类型
2. `Array.prototype.push.apply`：连接数组
3. `Array.prototype.slice.call`：将伪数组变为真正的数组

---

#### <font color=MediumOrchid>call</font>

call 有两个特点：

+ 改变 this 的指向
+ 执行函数

针对这些特点，我们来试着实现一个call方法，思路如下：

1. 改变this指向，并且执行函数

2. call 能接收多个参数的传递

3. call的第一个参数可以传null，此时表示window，并且执行的函数有可能有返回值

   ```javascript
   var foo = {
     value:1
   }
   function bar(){
     console.log(this.value)
   }
   bar.call(foo);
   ```

#####  <font color=MediumOrchid>第一步</font>

先看第一步，想要将bar函数内部的this指向foo的this，那么换而言之，将bar函数作为foo的属性，即

```
foo.bar()
```

这样就符合 this 指向的口头禅：谁调用函数，this就指向谁，这里是foo

然后运行一下 bar函数，再从foo的属性上删除它

我们看下第一版的代码

```javascript
// 样板代码 bar.call(foo)  等价于foo.call(bar)

Function.prototype.call2 = function(context){
   context.fn = this; // 谁调用this this就指向谁 这里是bar
   context.fn();
   delete context.fn(); // 删除foo上的属性 
}
var foo={
    value:1
}
function bar(){
    console.log(this.value)
}
bar.call2(foo); // 1
```

其中，context 代表 foo，fn 是 context(foo)上随意设置的一个属性

同时，bar函数 调用call方法，所以this表示bar

我们用伪代码描述一下上面的过程

```javascript
// 想要bar.call(foo) 等价于foo.call(bar)

foo.fn = bar
foo.fn()
delete foo.fn
```

---

##### <font color=MediumOrchid>第二步</font>

第二步的关键点在于，如何解决call函数的传参问题，我们看一下代码：

```javascript
Function.prototype.call2 = function(cntext){
   context.fn = this;
   var args = [];
   for(var i=1,len = arguments.length;i<len;i++){
       args.push("arguments["+i+"]")
   }
   eval("context.fn("+args+")")
   delete context.fn 
}
```

将传进 call 的参数序列，遍历放入数组中，用 eval 函数执行

---

##### <font color=MediumOrchid>第三步</font> 

第三步的关键在于，如果call的第一个参数是null，我们需要将this默认为window，并且如果我们执行的bar函数有返回值的话，需要将其返回出来

```javascript
Function.prototype.call2 = function(context){
  var context = context || window;
  context.fn = this  
  var args = [];
    
   for(var i = 1,len=arguments.length;i<len;i++){
         args.push("arguments"+i+"")
    }
    var result = eval("context.fn("+args+")")
    
    delete context.fn
    
    return result  
}
```

以上便是实现 call的完整代码，根据之前的思路分析，一步步实现其实并不难

---

#### <font color=Green>apply</font>

apply跟call类似，它也同样有两个特点：

+ 改变this的指向
+ 执行函数

但不同的是，apply传参的方式不是以参数序列的形式，而是数组的形式，我们看下代码：

```javascript
Function.prototype.apply2 = function(context,arr){
    var context = context || window;
    context.fn = this;
    
    var result;
    if(!arr){
        result = context.fn();
    } else{
        var args = [];
        
        for(var i = 1,len=arr.length;i<len;i++){
            args.push("arr["+i+"]")
        }
        
        result = eval("context.fn("+args+")");
        
        delete context.fn
        
        return result;
    }
}
```

apply的实现和call相比，除了传参方式不同导致的处理参数不同之外，其他大同小异，思路也可以参照 call 的实现

---

#### <font color=DarkOrange>bind</font>

bind与call、apply稍有不同，它的特点如下：

1. 不会执行函数，返回一个改变了上下文的新函数
2. 可以传入多个参数，bar函数内部可以传递参数，返回的函数也可以传递参数
3. 如果把返回出来的函数当成构造函数，那么bind绑定时指定的this值会失效

##### <font color=DarkOrange>第一步</font>

第一步的实现很简单，返回一个函数，我们看一下代码：
```JavaScript
Function.prototype.bind2 = function(context){
   var that = this;
   
   return function(){
       return that.apply(context); // return 防止函数有返回值
   }     
}
```

---

##### <font color=DarkOrange>第二步</font>

传入多个参数
```JavaScript
Function.prototype.bind2 = function(context){
   var that = this;
   
   var args = Array.prototype.slice.call(arguments,1); 
   // bind2后面可以传参数
   return function(){
       var bindArgs = Array.prototype.slice.call(argruments); 
       // 返回的新函数也可以传递参数
       return that.apply(context,args.concat(bindArgs))
   }    
}

var foo = {
    value:1
}
function bar(name,age,other){
    console.log(this.value); // 1
    console.log(name); // wtt
    console.log(age); // 18
    console.log(other); // xjw
}
var bindBar = bar.bind2(foo,'wtt',18);
bindBar("xjw");
```

我们可以看到，参数的处理有两处：
+ 从 bar.bind2(foo,args1,args2...)这边传递过去的参数
+ 从返回的新方法bindBar传递过去的参数

---

##### <font color=DarkOrange>第三步</font>

这是最复杂的一部分：当把bind返回的函数作为构造函数时，bind绑定时指定的this是无效的，我们看一下bind的效果
```JavaScript
var foo = {
    value:1
}
function bar(){
    this.habit = "music";
    console.log(this); // bar {habit: "music"}
}
var barBind = bar.bind(foo);
var bb = new barBind();
bb; // bar {habit: "music"}
```
我们可以看到，打印的this并不是foo，那我们应该怎么实现这样的效果呢
```JavaScript
Function.prototype.bind2 = function(context){
    var that = this;
    var args = Array.prototype.slice.call(arguments,1);
    var fBound = function(){
        var bindArgs = Array.prototype.slice.call(arguments);
        return that.apply(this instanceof fBound?this: context,args.concat(bindArgs))
    }    
    fBound.prototype = this.prototype; // 修改返回函数的 prototype 为绑定函数的 prototype，实例就可以继承绑定函数的原型中的值
    return fBound
}
```

当作为构造函数时，this指向实例，将绑定的函数this指向实例，可以让实例获得来自绑定函数的值

---

#### <font color=HotPink>常见应用</font>

1. 将伪数组转换为数组

   ```javascript
   Array.prototype.slice.call(arguments)
   // 或者
   Array.from(argumnts)
   ```

   

2. 数组连接

   ```javascript
   var arr1 = [1,2,3];
   var arr2 = [4,5,6];
   
   Array.prototype.push.apply(arr1,arr2)
   // 写法一样
   [].push.apply(arr1,arr2)
   ```

   

3. 判断变量类型

   ```javascript
   Object.prototype.toString.call(obj)
   
   // 写法一样
    Object.prototype.toString.call(arr)
    Object.prototype.toString.apply(arr)
    {}.toString.apply(arr)
   ```

   

4. 求数组最大值和最小值

   ```javascript
   var arr = [34,5,3,6,54,6,-67,5,7,6,-8,687];
   
   Math.min.apply(Math,arr);
   Math.max.apply(Math,arr);
   
   Math.min.call(Math,34,5,3,6,54,6,-67,5,7,6,-8,687);
   Math.max.call(Math,34,5,3,6,54,6,-67,5,7,6,-8,687);
   ```

   

---

#### <font color=LightCoral>总结</font>

call、apply、bind的异同点：

1. 相同点：

   + 都是用来改变this指向的，第一个参数都是将要改变成的this指向
   + 第一个参数不传，this默认都是window

2. 不同点：

   + 参数的传递方式不同

     ```javascript
     fn.call(obj,arg1,arg2,arg3...)
     fn.apply(obj,[arg1,arg2,arg3...])
     fn.bind(obj,arg1,arg2,arg3...)
     ```

     

   + call 和 apply在改变了this指向之后便执行了该函数，而bind只是返回了一个已经改变了的this值的函数，需要手动执行



























