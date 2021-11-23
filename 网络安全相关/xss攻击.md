#### XSS攻击

跨域脚本攻击，是最常见的web攻击，是一种代码注入攻击。攻击者通过在目标网站上注入恶意脚本，使之在用户的浏览器上运行。



利用这些恶意脚本，攻击者可获取用户的敏感信息，如 cookie、sessionid等等，进而危害数据安全。



xss脚本通常能够窃取用户数据并发送到攻击者的网站，或者冒充用户，调用目标网站接口并执行攻击者指定的操作

---

##### XSS攻击的本质

1. 恶意代码未经过滤，与网站正常代码混在一起
2. 浏览器无法分辨哪些脚本是可信的，导致恶意脚本被执行

---

##### XSS有哪些注入方法：

1. 在html内嵌的文本中，恶意内容以script标签形式注入
2. 在内联的js中，拼接的数据形成了代码片段
3. 在标签属性中，如 a 标签包含js等可执行的代码
4. 在onload、onerror、onclick等事件中，注入不受控制的代码
5. 在style标签中，包含类型background:url(javascript:)的代码

---

##### XSS 攻击的分类

1. 存储型XSS
2. 反射型XSS
3. DOM型XSS：
   + 攻击者构造出特殊的url，其中包含恶意代码
   + 用户打开带有恶意代码的url
   + 用户浏览器接收到响应后解析执行，前端js取出url中的恶意代码并执行

---

##### 如何预防

1. 存储型XSS和反射型XSS属于服务端的安全漏洞
2. DOM型XSS：
   + 尽量不要使用innerHTML、outerHTML以及v-html和dangerouslaySetInnerHTML
   + DOM内联事件监听器，如onclick、onerror、onload、onmouseover等，<a>标签的href属性、JavaScript的eval()、setTimeout()、setInterval()等，都能把字符串作为代码运行，需要转义