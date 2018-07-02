# 不要把 JWT 用作 session 管理（下）：美好理想与现实困境

建议在阅读完上篇之后在阅读本篇，本篇中使用到了很多上一篇的概念和原理

在上篇中，我们全面了解了关于身份认证和用户授权的所有概念，讲解了 JWT (Json Web Token) 的工作原理和使用场景。并且在文章的最后引出了一个问题，JWT 可否替代传统的 session 机制？

我们再来回顾一下这个问题。

在传统的 session 机制下，用户的登陆状态是通过 session id 维系的：session id 同时存储在后端 Redis 和用户浏览器的 cookie 中，用户的每次请求都会以 cookie 的形式带上 session id，后端在收到请求之后就能够通过检测 session id 是否在 Redis 中来判断用户是否已经登陆:

![session flow](./images/token-not-session/session-flow.png)

理论上来说，JWT 机制可以取代 session 机制：用户不需要提前进行登陆，后端也不需要 Redis 记录用户的登陆信息。当用户需要调用接口时，必须上传一个合法的 JWT，每一次调用接口，后端都使用请求中附带的 JWT 做一次合法性的验证。这样也间接达到了认证用户的目的：

![jwt-flow](./images/token-not-session/jwt-flow.png)

注意上图中的 JWT 流程和上一篇中描述的有存在差异的地方：在上一篇中我们讲到客户端会保存一份 secret 用于生成 JWT，而上图中 JWT 完全是由客户端给予的。但这都不影响服务端对 JWT 的验证。而至于这两者适用于什么样的场景后面再说

session 机制和 JWT 机制本质上的区别的是前者是有状态的（stateful），因为它需要使用 Redis 管理每一个用户的状态，是否登陆，是否登出，是否过期，用户每一次的访问都和当前的状态有关；而后者是无状态的（stateless），因为你的每一次访问都是独立的，和之前是否访问过无关（但token 还是存在过期时间的，稍后谈如何处理这个问题）

更重要的是，这种“无状态”，恰恰也与 RESTful API 的特性不谋而合。我没法找到一个官方对于 RESTful API 无状态的定义， 但无状态确实是一种约定。比如在 [REST API Tutorial](https://restfulapi.net/statelessness/) 网站上关于无状态的定义如下

>As per the REST (REpresentational “State” Transfer) architecture, the server does not store any state about the client session on the server side. This restriction is called Statelessness. Each request from client to server must contain all of the information necessary to understand the request, and cannot take advantage of any stored context on the server. Session state is therefore kept entirely on the client. client is responsible for storing and handling all application state related information on client side.

接下来我们要对标这两种身份认证策略。同时我们还是以 Google API 作为 API 调用和实现的示例

我们先聊一些你会担心的，但是不成问题的问题

## 不成问题的问题

我们从安全出发，首先考虑两种常见的攻击：CSRF 和 XSS

- CSRF: CSRF 全称 Cross Site Request Forgery, 中文译为“跨站域请求伪造”: 
  - 假设你（作为受害者）刚刚访问了某个银行网站，因为 session id 同时种在了后端和浏览器 cookie 中并且没有过期，在相当常的一段时间内你能够不用登陆而继续在银行网站上执行操作。
  - 某个攻击者黑客已经发现银行网站通过接口`http://bank.example/transfer?from=A&to=B&amount=1000`用来将指定数量的金额从 A 转给 B。于是他伪造了一个网站，在网站上放置了一个图片，图片的 src 是`http://bank.example/transfer?from=YOU&to=HACKER&amount=10000`，也就是让把你转一万元给他的请求 URL。
  - 此时他会诱使你点击这个图片，一旦点击之后，就相当于向银行网站发出了一个转账请求，而又因为你刚刚登陆过网站 session 还没有过期的关系，银行不会再让你登陆，并且以为这是一个合法的转账请求。于是你在毫不知情的情况下使用了自己的账号向陌生人进行了转账，银行也认为这是合法的。

- XSS: XSS 全称 Cross-site scripting，中文译为“跨站点脚本攻击”。简单来说它允许攻击者在受害者的浏览器里执行恶意脚本：
  - 某个网站为文章提供留言功能，用户能够输入自己的留言，并且浏览其他人的留言
  - 某个黑客的留言不是正常的文本，而是一段 `<script/>` 恶意脚本。该脚本的内容是读取当前页面下的 cookie，并且发送到指定的网站。该脚本成功插入进了留言的数据库中
  - 这样一来每个用户在浏览该文章时，该段存有恶意脚本的留言都会和其他留言一起生成在页面上，还会额外的执行恶意脚本。

  当然目前已经有很完备的方案来应付这两类攻击，绝大部分站点都对这两类攻击免疫。接下来我们说的“容易”或者“难”受到攻击其实是在比较两个很小很小的概率而已。

很明显 CSRF 就是针对 session 机制而设计的。但是 JWT 机制就对于 CSRF 免疫了吗？不一定，这和 JWT 的存储方式。在浏览器中，JWT 的存储方式无非只有两种选择：(Local/Session) Storage 和 Cookie。绝大部分人会存储在 storage 中，需要使用时再从 storage 中取出然后附着再 HTTP Header 上。如但是如果存储在 cookie 里，你还会从 cookie 里取出并且附着在 HTTP Header 上吗？如果是那么就意味着 JWT 会被发送两份；如果不是，那么 JWT 就变成了一个类似于 session id 的角色，总是随着请求自动发出，又落入了 CSRF 的陷阱中。

至于 XSS 攻击，无论是 session id 和 JWT 风险都是一致的：既然攻击者都能在你的浏览器里执行脚本了，那么他既可以从 storage 和 cookie 里读取数据（cookie 可以启用 HttpOnly 选项来阻止 ），也可以主动向服务端发出伪造的请求。

JWT 和 cookie 非常相似，它们都会面临一些同样的问题，例如它们都会面容量上的限制（无论在存储方面还是在携带方面），它们都不允许在其中存储敏感信息（信用卡卡号）。最重要的，**它们还要解决过期（expire）和认证撤回（revoke）的问题**。这两点才是不要使用 JWT 作为 session 管理的重要原因

## 过期问题

JWT 和普通 token 一样，都会面临过期的问题（在这里我们不考虑不会过期的场景，不过期的 token 是毫无意义且对安全不负责任的）。而如果真的过期，我们不妨考虑有哪些方案处理这个问题

**Refresh Token**

在上一篇中我们聊到 refresh token 就是用来交换 JWT 的，和 JWT 相比它有更长的生命周期。同时因为直接和身份认证相关，所以它的安全也显得更重要。通常 refresh token 方案适用于服务端和移动应用或者设备应用上，因为在这两者的运行环节隔离并且私密，相反在浏览器中使用该方案就会显得非常的危险，所以这个方案并不适用于 web

**Sliding Session**

从名称上就可以看出，这本应该是针对 session 的解决方案，但也同时适用于 JWT。在 session 机制下，session id 也是存在过期时间的，过期时间同样很短。但是只要用户在尚未过期的期限内使用网站，那么就自动为 session id 续期。这样用户就能够在相当长的一段时间内不间断的使用网站。

同样在 JWT 的机制下，只要用户在还未过期的范围内使用网站，那么也自动为 JWT “续期”。只不过这里续期的方式是生成一个新的签名的 token。这里的续期方式还而可以划分为客户端方案和服务端方案：

  - 客户端方案：在 JWT 中存储过期时间，由客户端自行判断 JWT 是否即将过期，后端提供更换 JWT 的接口。如果客户端判断 JWT 即将过期了，那么自行通过接口更换新的 JWT
  - 服务端方案：在每个请求的返回响应中自带新的 JWT，这样每一次请求就意味着需要更新 JWT

但话说回来，用 session 的续期机制在 JWT 上进行实现，并且还实现的更加复杂，为什么不直接使用 session 呢？

**不一致的过期时间**

Web 站点和移动应用的过期时间并不一致。通常移动应用的过期时间会更长，例如数日之后你再打开应用仍然处于登陆状态。所以你可以在 JWT 中添加额外的字段来标记这是一个来自移动应用的 JWT，如果后端检测到了这个字段那么就自动忽视它的过期时间。当然移动应用上可以直接使用 refresh token 方案。但是如果用户修改了密码或者手机被偷了怎么办？这里我们就要考虑撤销的问题

## 撤回问题





## 参考资料

### TOKEN SESSION VS COOKIE SESSION

* https://ponyfoo.com/articles/json-web-tokens-vs-session-cookies
* https://auth0.com/blog/ten-things-you-should-know-about-tokens-and-cookies/

### TOKEN EXPIRED AND UPDATE

* https://stackoverflow.com/questions/39825953/handling-jwt-expiration-and-jwt-payload-update
* https://github.com/brahalla/Cerberus/issues/5
* https://softwareengineering.stackexchange.com/questions/338337/handling-token-renewal-session-expiration-in-a-restful-api
* https://www.zhihu.com/question/41248303
* https://news.ycombinator.com/item?id=11929267

### OTHER

* https://restfulapi.net/statelessness/
* https://stackoverflow.com/questions/34130036/how-to-understand-restful-api-is-stateless
* https://www.ibm.com/developerworks/cn/web/1102_niugang_csrf/