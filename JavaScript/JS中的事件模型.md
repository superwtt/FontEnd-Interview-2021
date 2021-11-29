#### 事件流/事件机制

事件捕获阶段

出于目标阶段

事件冒泡阶段

---

#### 事件模型

事件模型可以分为三种：

+ 原始事件模型：

  ```javascript
  // HTML中直接绑定
  <input type="button" onclick="fun()">
      
  // 通过JS代码绑定
  var btn = document.getElementById('.btn');
  btn.onclick = fun;    
  ```

+ 标准事件模型：即事件流

+ IE事件模型：事件处理、事件冒泡