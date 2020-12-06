# Cookie 的工作过程
用到两个字段：响应头字段 **Set-Cookie** 和请求头字段 **Cookie**。

![cookie](https://static001.geekbang.org/resource/image/9f/a4/9f6cca61802d65d063e24aa9ca7c38a4.png)

从这张图中我们也能够看到，Cookie 是由浏览器负责存储的，而不是操作系统。所以，它是“浏览器绑定”的，只能在本浏览器内生效。

# Cookie 的属性
## Cookie 的生存周期
Cookie 的有效期可以使用 Expires 和 Max-Age 两个属性来设置。

“**Expires**”俗称“过期时间”，用的是绝对时间点，可以理解为“截止日期”（deadline）。

“**Max-Age**”用的是相对时间，单位是秒，浏览器用收到报文的时间点再加上 Max-Age，就可以得到失效的绝对时间。

两者不一致时，浏览器以 Max-Age 为准。
 
## Cookie 的作用域
设置 Cookie 的作用域，让浏览器仅发送给特定的服务器和 URI，避免被其他网站盗用。作用域的设置比较简单，“**Domain**”和“**Path**”指定了 Cookie 所属的域名和路径，浏览器在发送 Cookie 前会从 URI 中提取出 host 和 path 部分，对比 Cookie 的属性。如果不满足条件，就不会在请求头里发送 Cookie。

## Cookie 的安全性
“**HttpOnly**”会告诉浏览器，此 Cookie 只能通过浏览器 HTTP 协议传输，禁止其他方式访问，浏览器的 JS 引擎就会禁用 document.cookie 等一切相关的 API，脚本攻击也就无从谈起了。

“**SameSite**”可以防范“跨站请求伪造”（XSRF）攻击，设置成“SameSite=Strict”可以严格限定 Cookie 不能随着跳转链接跨站发送，而“SameSite=Lax”则略宽松一点，允许 GET/HEAD 等安全方法，但禁止 POST 跨站发送。

“**Secure**”，表示这个 Cookie 仅能用 HTTPS 协议加密传输，明文的 HTTP 协议会禁止发送。但 Cookie 本身不是加密的，浏览器里还是以明文的形式存在。

# Cookie 的应用
Cookie 最基本的一个用途就是**身份识别**，保存用户的登录信息，实现会话事务。

Cookie 的另一个常见用途是**广告跟踪**。

# 总结