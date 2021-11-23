#### 前端是怎么防止 csrf 攻击的？

---

##### 什么是csfr攻击

csrf 是指跨站请求伪造，攻击者诱导受害者进入到第三方网站，在第三方网站中，向被攻击网站发送跨站请求，利用受害者在被攻击网站已经获取的注册凭着，达到冒充用户，对被攻击的网站执行某项操作的目的

---

##### 一个典型的 csrf 攻击有以下的流程

1. 受害者登录 a.com，并保留了登录凭证cookie
2. 攻击者引诱受害者访问 b.com
3. b.com 向 a.com发送了一个请求：a.com/action=xxx，浏览器会默认携带 a.com 的cookie
4. a.com 接收到请求后，对请求进行验证并确认是受害者的凭证，误认为是受害者自己发的请求
5. a.com以受害者的名义执行了a.com/action=xx
6. 攻击完成，攻击者在受害者不知情的情况下，冒充受害者，让a.com执行了自己的操作

![](https://raw.githubusercontent.com/superwtt/MyFileRepository/main/image/网络安全/csrf攻击例子.png)



---

##### 几种常见的攻击类型

1. get 类型的csrf：get 类型的csrf利用非常简单，通常只需要一个http请求，一般会这样利用：

   ```javascript
    ![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018b/ff0cdbee.example/withdraw?amount=10000&for=hacker)
   ```

   受害者访问含有这个img的图片之后，浏览器就会自动向 `http://bank.example/withdraw??amount=10000&for=hacker`发出一次http请求，bank.example就会收到包含受害者登录信息的一次跨域请求。

2. POST 类型的csrf：这种类型的csrf 利用起来通常使用的是一个自动提交的表单

   ```html
    <form action="http://bank.example/withdraw" method=POST>
       <input type="hidden" name="account" value="xiaoming" />
       <input type="hidden" name="amount" value="10000" />
       <input type="hidden" name="for" value="hacker" />
   </form>
   <script> document.forms[0].submit(); </script> 
   ```

   访问该页面后，表单会自动提交，相当于模拟用户完成了一次post请求

3. 链接类型的csrf：比起上面两种用户打开页面就中招的情况，这种需要用户点击链接才会触发。这种类型通常是在论坛中发布的图片中嵌入恶意链接，或者以广告的形式诱导用户中招，攻击者通常会以比较夸张的词语诱骗用户点击，例如：

   ```html
     <a href="http://test.com/csrf/withdraw.php?amount=1000&for=hacker" taget="_blank">
     重磅消息！！
     <a/>
   ```

   

---

##### csrf 攻击的特点

1. csrf 攻击一般发起在第三方网站，而不是被攻击的网站，即通常发生在第三方域名
2. csrf 攻击者不能获取到cookie等信息，只是利用

---

##### 如何防护

1. 阻止不明外域的访问，包括同源检测、Samesite Cookie：既然 csrf 来源于第三方网站，那么我们就直接禁止外域或者不受信任的域名对我们发起请求。如何判断请求来源是否来自外域呢？每个异步请求都会携带两个Header，用来标记来源域名：
   + origin
   + referrer
2. 提交时要求附加本域才能获取的信息
3. 双重cookie验证：利用 csrf攻击不能获取用户cookie，只能伪造请求的特点：
   + 用户访问网站页面时，向请求域名中注入一个cookie，内容为随机字符串，如csrfcookie=v8g94esdfhw
   +  在前端向后端发起请求时，取出cookie，并添加到url参数中
   + 后端校验cookie字段与参数中的cookie字段是否一致

---

##### 参考链接

[very good](https://tech.meituan.com/2018/10/11/fe-security-csrf.html)