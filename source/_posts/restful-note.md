title: Restful API浅析 之设计原则与案例修正
category: web
author: Mcl
date: 2015-11-04
tags: [restful]
---

> 之前写Restful的服务器端API，写的不好，痛定思痛，决心好好搞懂Restful。
> 在这一篇中我们首先搞懂Rest的含义，然后以我编写的失败的api为例，讲解如何编写出符合Rest风格的API。注意本篇指导原则一节大部分来自阮一峰老师的[RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

<!--more-->

---

## Restful浅析
REST是”REpresentational State Transfer”，一种翻译是”表现层状态转移”,首先看看wiki百科的介绍:

>REST - 表征状态转移是Roy Fielding博士在2000年他的博士论文中提出来的一种软件架构风格。 目前在三种主流的Web服务实现方案中，因为REST模式的Web服务与复杂的SOAP和XML-RPC对比来讲明显的更加简洁，越来越多的web服务开始采用REST风格设计和实现。例如，Amazon.com提供接近REST风格的Web服务进行图书查找；雅虎提供的Web服务也是REST风格的。 – wiki

从以上的介绍中我们知道REST是一种web软件架构风格，不过我还是不知道REST是个什么鬼，再查查看，有人这么解释REST:
```
- REST是一套用来创建Web Service的方法。
- REST式的Web Service的主旨是让事情尽量的简单化。
- REST式的Web Service使用HTTP里的方法：GET， POST， DELETE， PUT。你不需要使用URL或请求的内容来指定这个方法。
- REST式的Web Service使用URL来指明你将要操作什么对象。
- REST式的Web Service使用HTTP状态码作为返回值。
- REST式的Web Service调用产生的HTTP请求内容只是用于服务数据——不是用来指明调用方法，目标对象或返回值的。
```

简单来说，REST是所有web应用都应该遵守的架构设计指导原则，每一个URL代表一种资源，客户端通过四个HTTP动词，对服务器端资源进行操作，实现”表现层状态转化”。查看以下几篇文章深入地了解REST：
+ [理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html) - 阮一峰老师11年的文章,浅显易懂
+ [理解本真的REST架构风格](http://www.infoq.com/cn/articles/understanding-restful-style) -infoq上一篇翻译的文章，非常细致地解释了REST
+ [Richardson Maturity Model steps toward the glory of REST](http://martinfowler.com/articles/richardsonMaturityModel.html) - Martin fowler解释Richardson的REST3层成熟度模型
+ [从消费者的角度评估REST的价值](http://hippoom.github.io/blogs/value-of-hypermedia-from-client-perspective.html) -以举例的方式解释Richardson成熟度模型的第三个级别：Hypermedia，很有意思的一篇文章。

也许最快的学习方式是学习业界成熟的REST api设计,可以看看github是如何设计[github API](https://developer.github.com/v3/)的,另外也许你想看看提出REST的[那篇论文](http://www.ics.uci.edu/~fielding/pubs/webarch_icse2000.pdf)，TL;DR

## 设计原则

本来想写个REST API设计指南的，不过阮一峰老师在14年已经干了[这事](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)，这里我们转载该文章的原则，然后重新修改我们的api。另外一些好的指南:
+ [Some REST best practices](https://bourgeois.me/rest/)
+ [Principles of good RESTful API Design](http://codeplanet.io/principles-good-restful-api-design/)

## 指导原则

### 域名
应该尽量将API部署在专用域名之下,如:
```
https://api.example.com
```

如果确定API很简单，不会有进一步扩展，可以考虑放在主域名下:
```
https://example.org/api/
```

### 版本

应该将API的版本号放入URL。
```
https://api.example.com/v1/
```

### 路径

在RESTful架构中，每个网址代表一种资源（resource），所以网址中不能有动词，只能有名词，而且所用的名词往往与数据库的表格名对应。一般来说，数据库中的表都是同种记录的”集合”（collection），所以API中的名词也应该使用复数。
举例来说，有一个API提供动物园（zoo）的信息，还包括各种动物和雇员的信息，则它的路径应该设计成下面这样。
```
https://api.example.com/v1/zoos
https://api.example.com/v1/animals
https://api.example.com/v1/employees
```

### HTTP动词

对于资源的具体操作类型，由HTTP动词表示。
常用的HTTP动词有下面五个（括号里是对应的SQL命令）：
```
GET（SELECT）   ：从服务器取出资源（一项或多项）。
POST（CREATE）  ：在服务器新建一个资源。
PUT（UPDATE）   ：在服务器更新资源（客户端提供改变后的完整资源）。
PATCH（UPDATE） ：在服务器更新资源（客户端提供改变的属性）。
DELETE（DELETE）：从服务器删除资源。
```

一些例子：
```
GET    /zoos   			：列出所有动物园
POST   /zoos   			：新建一个动物园
GET    /zoos/ID			：获取某个指定动物园的信息
PUT    /zoos/ID			：更新某个指定动物园的信息（提供该动物园的全部信息）
PATCH  /zoos/ID			：更新某个指定动物园的信息（提供该动物园的部分信息）
DELETE /zoos/ID			：删除某个动物园
GET    /zoos/ID/animals		：列出某个指定动物园的所有动物
DELETE /zoos/ID/animals/ID	：删除某个指定动物园的指定动物
```

### 过滤信息（Filtering）

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。
下面是一些常见的参数:
```
?limit=10		：指定返回记录的数量
?offset=10		：指定返回记录的开始位置。
?page=2&per_page=100	：指定第几页，以及每页的记录数。
?sortby=name&order=asc	：指定返回结果按照哪个属性排序，以及排序顺序。
?animal_type_id=1	：指定筛选条件
```


参数的设计允许存在冗余，即允许API路径和URL参数偶尔有重复。比如，`GET /zoo/ID/animals` 与 `GET /animals?zoo_id=ID` 的含义是相同的。

### 状态码（Status Codes）

服务器向用户返回的状态码和提示信息，常见的有以下一些（方括号中是该状态码对应的HTTP动词）:
```
200 OK - [GET]				：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]		：用户新建或修改数据成功。
202 Accepted - [*]			：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]		：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]	：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]			：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 			：表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]			：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]		：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]				：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] ：当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]		：服务器发生错误，用户将无法判断发出的请求是否成功。
```

### 错误处理（Error handling）

如果状态码是`4xx`，就应该向用户返回出错信息。一般来说，返回的信息中将error作为键名，出错信息作为键值即可。
```
{
    error: "Invalid API key"
}
```

### 返回结果

针对不同操作，服务器向用户返回的结果应该符合以下规范。
```
GET    /collection 		：返回资源对象的列表（数组）
GET    /collection/resource 	：返回单个资源对象
POST   /collection 		：返回新生成的资源对象
PUT    /collection/resource 	：返回完整的资源对象
PATCH  /collection/resource 	：返回完整的资源对象
DELETE /collection/resource 	：返回一个空文档
```

### Hypermedia API

RESTful API最好做到Hypermedia，即返回结果中提供链接，连向其他API方法，使得用户不查文档，也知道下一步应该做什么。
比如，当用户向api.example.com的根目录发出请求，会得到这样一个文档。
```
{"link": {
  "rel":   "collection https://www.example.com/zoos",
  "href":  "https://api.example.com/zoos",
  "title": "List of zoos",
  "type":  "application/vnd.yourformat+json"
}}
```

上面代码表示，文档中有一个link属性，用户读取这个属性就知道下一步该调用什么API了。rel表示这个API与当前网址的关系（collection关系，并给出该collection的网址），href表示API的路径，title表示API的标题，type表示返回类型。
Hypermedia API的设计被称为HATEOAS。Github的API就是这种设计，访问api.github.com会得到一个所有可用API的网址列表。
```
{
  "current_user_url": "https://api.github.com/user",
  "authorizations_url": "https://api.github.com/authorizations",
  // ...
}
```

从上面可以看到，如果想获取当前用户的信息，应该去访问api.github.com/user，然后就得到了下面结果。
```
{
  "message": "Requires authentication",
  "documentation_url": "https://developer.github.com/v3"
}
```

上面代码表示，服务器给出了提示信息，以及文档的网址。

### 其他
+ API的身份认证应该使用OAuth 2.0框架。
+ 服务器返回的数据格式，应该尽量使用JSON，避免使用XML。

## 案例修正

下面我们使用上面的准则进行api的整理修改，注意我们目前没有做到Hypermedia

### 用户API : USERS

#### 修改前
+ `/api/v1/user/:user_id` 	GET  获取用户信息
+ `/api/v1/auth/register` 	POST 用户注册
+ `/api/v1/auth/login` 		POST 用户登录
+ `/api/v1/auth/confirmpw` 	POST 确认密码
+ `/api/v1/auth/changepw` 	POST 修改密码

#### 诊断
+ 资源没有使用复数形式，如`user`
+ `:user_id`和`users`冗余，使用`:id`即可
+ 存在动词如`register`,`login`等
+ `confirmpw`和`changepw`冗余,应该在改变密码的时候对密码进行确认。

#### 修改后

由于本项目的用户数据从后台导入，因此删除register功能，另外因为此项目为隐私项目进行授权后所有的请求需要附带token，这里省略access_token参数。
这里省略`/api/v1`的前缀:
+ `/authentication` GET 验证用户名密码，并重新生成token，返回基本的用户信息
+ `/users/changePassword` POST 重置用户密码，重新生成盐值和hash存储到数据库中，返回修改后的用户基本信息(这个暂时不知道怎么改,是使用users UPDATE方法，或者将改变密码变成一个名词的服务)
+ `/users/details` GET 获取用户详细信息，如最近的主题以及最近的回复

### 帖子API : POSTS
+ `/api/v1/recent` 					GET    获取最近帖子
+ `/api/v1/posts?limit=xx&page=xx` 	GET    获取帖子列表
+ `/api/v1/post/:post_id` 			GET    获取某个帖子内容
+ `/api/v1/post` 					POST   新建一个帖子
+ `/api/v1/post/:post_id/likes` 	POST   为某个帖子点赞
+ `/api/v1/post/:id` 				PUT    更新某个帖子
+ `/api/v1/post/:id` 				DELETE 删除某个帖子

#### 诊断
+ 资源没有使用复数形式
+ post_id冗余
+ recent的api和获取帖子列表冗余，设置limit为某值,page为1即可获得最近的帖子
+ 点赞功能并不在需求里，这里删除掉

#### 修改后
+ `/posts?limit=xx&page=xx` GET    获取帖子列表
+ `/posts/:id` 				GET    获取某个帖子的具体内容
+ `/posts`					POST   创建帖子，返回帖子基本内容。
+ `/posts/:id` 				DELETE 删除帖子，返回空文档。

### 回复API ： REPLYS
+ `/api/v1/reply` 			POST   创建一个回复
+ `/api/v1/reply/:reply_id` DELETE 删除某个回复

#### 诊断
+ 资源没有使用复数形式
+ `reply_id`冗余

#### 修改后
+ `replys` 		POST   创建一个回复，返回该回复(这里应该增加帖子的回复数目)
+ `replys/:id` 	DELETE 删除某个回复,返回空文档(这里应该减少帖子的回复数目)
