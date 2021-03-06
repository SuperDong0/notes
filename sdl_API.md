### 设计与需求阶段

#### Authentication

身份认证 设计原则:使用成熟且安全的组件 不要造轮子

* 使用标准的认证协议(如 [JWT](https://jwt.io/), [OAuth](https://oauth.net/)). 不要使用Basic Auth
* 使用标准的 Authentication, token generating, password storing
* 在登录中使用 Max Retry 和自动封禁功能
* 加密所有的敏感数据

#### JWT

[JSON Web Token - jwt.io](https://jwt.io/introduction/)

* 使用随机复杂的密钥(`JWT Secret`) 以增加暴力枚举的难度
* 后端强制使用高强度算法(`HS256` or `RS256`). 不要从请求的payload中直接提取数据
* 设置尽可能短的token过期时间 token expiration (`TTL`, `RTTL`)
* 不要在JWT的payload中存储敏感数据 因为 [很容易被解码](https://jwt.io/#debugger-io)

#### OAuth

API访问授权标准 安全且开放

* 后端需要验证`redirect_uri` 且只允许白名单中的URL
* 始终尝试交换code而不是交换token (不允许 `response_type=token`)
* 使用 `state` 参数并填充随机的hash 以防御OAuth authentication过程中的CSRF(跨站请求伪造)
* 对不同的应用定义各自的 `default scope`参数(默认的作用区域) 和 `validate scope`参数(有效的作用区域)


#### Access

访问

* 限制requests(Throttling) 以避免DDoS和暴力枚举攻击
* 使用HTTPS 以避免MITM (Man in the Middle Attack)
* 使用`HSTS` 以避免SSL Strip攻击


#### Input

输入

* 根据操作 使用恰当的HTTP方法. 如果请求的方法不适用于请求的资源则返回 `405 Method Not Allowed`
  * HTTP方法(用途): `GET (read)`, `POST (create)`, `PUT/PATCH (replace/update)`,`DELETE (to delete a record)` 
* 验证`content-type` 只允许后端支持的格式 并在不满足条件的时候返回 `406 Not Acceptable`
  * content-type: `application/xml`, `application/json`等
* 验证POST请求中的`content-type`是否和后端所接受的格式相同
  * content-type: `application/x-www-form-urlencoded`, `multipart/form-data`, `application/json`等
* 验证用户输入 以避免常规web漏洞
  * 如`XSS`, `SQL-Injection`, `Remote Code Execution`等
* 不要在URL中使用任何敏感的数据 而是使用标准的Authorization header(认证请求头)
  * 敏感数据: `credentials`, `Passwords`, `security tokens`,`API keys`等 
* 验证用户输入 以避免常规web漏洞
  * 常规漏洞: 如`XSS`, `SQL-Injection`, `Remote Code Execution`等
* 使用API Gateway服务 来启用 缓存(caching) 和 访问速率限制策略(Rate Limit policies). 
  * API Gateway services: Quota, Spike Arrest, Concurrent Rate Limit

#### Processing

处理

* 检查是否所有的endpoints都被身份认证保护 以避免破坏认证过程(authentication process).
* 避免直接使用用户自有资源的资源id. 使用 `/me/orders` 替代 `/user/654321/orders`
* 使用`UUID` 避免使用自增id
* 如果需要解析 XML 文件, 确保实体解析(entity parsing)是关闭的以避免XXE(XML external entity attack)
* 如果需要解析 XML 文件, 确保实体扩展(entity expansion)是关闭的以避免通过指数实体扩展攻击(exponential entity expansion attack)实现的 `Billion Laughs/XML bomb`
* 文件上传功能 使用CDN完成
* 如果需要处理大量的数据, 后端使用Workers和Queues来快速返回响应.以避免HTTP阻塞
* 记得关闭DEBUG模式

#### Output

响应

* 响应中加上header
  * `X-Content-Type-Options: nosniff` 
  * `X-Frame-Options: deny`
  * `Content-Security-Policy: default-src 'none'`
* 删除指纹header
  * `X-Powered-By`, `Server`, `X-AspNet-Version` ...
* 响应中强制使用`content-type`header并且根据响应内容指定具体值 通常是`application/json`
* 不能返回敏感数据
  * 如 `credentials`, `Passwords`, `security tokens`...
* 操作结束时 返回恰当的状态码
  * 如 `200 OK`, `400 Bad Request`, `401 Unauthorized`, `405 Method Not Allowed` ...


#### CI & CD

持续集成&持续部署

* 用 单元/集成测试 覆盖范围 来审计设计和实现
* 使用代码审查(code review)流程 禁止自行批准
* 在推送到生产环境之前 确保服务的所有组件(包括第三方库和其他)都经过杀毒软件的静态扫描
* 为部署设计回滚解决方案


---

#### 参考

* [yosriady/api-development-tools](https://github.com/yosriady/api-development-tools) - A collection of useful resources for building RESTful HTTP+JSON APIs. 包括API规范 API框架 API文档 API监控 API网关 API测试 以及 各语言的API实现.
* [API-Security-Checklist](https://github.com/shieldfy/API-Security-Checklist/blob/master/README-zh.md)
