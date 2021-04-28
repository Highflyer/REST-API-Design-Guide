REST API 设计规范与最佳实践
========================

> ©2019, Ray Xue

**###目录**

  - [一、REST API 简介](#一rest-api-简介)
    - [1.1 术语](#11-术语)
    - [1.2 成熟度模型](#12-成熟度模型)
  - [二、URL 设计](#二url-设计)
    - [2.1 一般约定](#21-一般约定)
    - [2.2 动词 + 宾语](#22-动词--宾语)
    - [2.3 动词的覆盖](#23-动词的覆盖)
    - [2.4 宾语必须是名词](#24-宾语必须是名词)
    - [2.5 使用复数做资源名称](#25-使用复数做资源名称)
    - [2.6 避免多级 URL](#26-避免多级-url)
    - [2.7 搜索、排序、筛选和分页](#27-搜索排序筛选和分页)
  - [三、HTTP 状态码](#三http-状态码)
    - [3.1 始终使用精确的状态码](#31-始终使用精确的状态码)
    - [3.2 2xx 状态码](#32-2xx-状态码)
    - [3.3 3xx 状态码](#33-3xx-状态码)
    - [3.4 4xx 状态码](#34-4xx-状态码)
    - [3.5 5xx 状态码](#35-5xx-状态码)
  - [四、服务器回应](#四服务器回应)
    - [4.1 不要返回纯本文](#41-不要返回纯本文)
    - [4.2 发生错误时，不要返回 200 状态码](#42-发生错误时不要返回-200-状态码)
    - [4.3 提供链接](#43-提供链接)
  - [五、版本控制](#五版本控制)
  - [六、参考链接](#六参考链接)

---

## 一、REST API 简介

[RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) 是目前最流行的 API 设计规范，用于 Web 数据接口的设计。

在 2000 年，[Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding) 提议使用表现层状态转换 (英语：Representational State Transfer，缩写：REST) 作为设计 Web 服务的体系性方法。REST 是一种基于超媒体构建分布式系统的架构风格。REST 独立于任何基础协议，并且不一定绑定到 HTTP。但是，最常见的 REST 实现使用 HTTP 作为应用程序协议。

基于 HTTP 的 REST 的主要优势在于它使用开放标准，不会绑定 API 的实现，也不会将客户端应用程序绑定到任何具体实现。例如，可以使用 ASP.NET 或者 Node.js 编写 REST Web 服务，而客户端应用程序能够使用任何语言或工具来发起 HTTP 请求和分析 HTTP 响应。

它的大原则容易把握，但是细节不容易做对。我们必须进行较多的工作来实施 REST API 中的最佳实践。大多数情况下，懒惰或缺乏时间意味着我们不会付出努力，如此为我们的用户留下一个个古怪的、难用的却又脆弱的 API。

本文总结 RESTful 的设计细节，介绍如何设计出易于理解和使用的 API。

### 1.1 术语

下面是与 REST API 相关的非常重要的一些术语：

- **资源** - 资源是某个事物的对象或表示形式，它与某事物有一些关联的数据，和可以对其进行操作的方法集。例如，用户、商品和订单是资源，添加、查询、更新、删除是要对这些资源执行的操作。
- **集合** - 集合是指资源的集合，例如，products 是 product 资源的集合。
- **URI** - 统一资源标识符（Uniform Resource Identifier）是一个用于标识某一互联网资源名称的字符串，可以通过它定位资源，并对其执行某些操作。
- **端点** - 端点（Endpoint）是动词和 URI 的组合。例如：`GET /products`。
- **状态码** - 一个响应的状态由其状态代码（[HTTP Status Code](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)）指定。

### 1.2 成熟度模型

2008 年，Leonard Richardson 提议对 Web API 使用以下[成熟度模型](https://martinfowler.com/articles/richardsonMaturityModel.html)：

- 级别 0：定义一个 URI，所有操作是对此 URI 发出的 POST 请求。
- 级别 1：为各个资源单独创建 URI。
- 级别 2：使用 HTTP 方法来定义对资源执行的操作。
- 级别 3：使用超媒体（HATEOAS，参见 [HATEOAS - Wikipedia](https://en.wikipedia.org/wiki/HATEOAS)）。

根据 Roy Fielding 的定义，级别 3 对应于某个真正的 RESTful API。在实践中，许多发布的 Web API 大致处于级别 2。

## 二、URL 设计

### 2.1 一般约定

- URI 都用小写，路径中连字符使用破折号“-”；
- 使用 [JSON](https://en.wikipedia.org/wiki/JSON) 通信;
- API 带版本控制；
- 使用 [JWT](https://jwt.io) 令牌进行鉴权；

### 2.2 动词 + 宾语

RESTful 的核心思想就是，客户端发出的数据操作指令都是"动词 + 宾语"的结构。比如，`GET /products` 这个命令，`GET` 是动词，`/products` 是宾语。

动词通常就是如下五种 HTTP 方法，对应 CRUD 操作。

- `GET` - 检索位于指定 URI 处的资源的表示形式。响应消息的正文包含所请求资源的详细信息。
- `POST` - 在指定的 URI 处创建新资源。请求消息的正文将提供新资源的详细信息。请注意，POST 还用于触发不实际创建资源的操作，如登录、注销等。
- `PUT` - 在指定的 URI 处创建或替换资源。请求消息的正文指定要创建或更新的资源。
- `PATCH` - 对资源执行部分更新。请求正文包含要应用到资源的一组更改。
- `DELETE` - 删除位于指定 URI 处的资源。

*根据 HTTP 规范，动词一律大写。*

[](https://uploads.toptal.io/blog/image/123414/toptal-blog-image-1498567452740-6773f3fd56484fa794cbf1dbfb9ebb38.png)

特定请求的影响应取决于资源是集合还是单个项。下表汇总了使用电子商务示例的大多数 RESTful 实现所采用的常见约定。

| 资源                  | POST        | GET          | PUT                | DELETE       |
|---------------------|-------------|--------------|--------------------|--------------|
| /products          | 创建新商品       | 检索所有商品       | 批量更新商品             | 删除所有商品       |
| /products/1        | N/A     | 检索商品 1 的详细信息 | 如果商品 1 存在，则更新其详细信息 | 删除商品 1       |
| /products/1/orders | 创建商品 1 的新订单 | 检索商品 1 的所有订单 | 批量更新商品 1 的订单       | 删除商品 1 的所有订单 |

### 2.3 动词的覆盖

有些客户端只能使用 `GET` 和 `POST` 这两种方法。服务器必须接受 `POST` 模拟其它三个方法（`PUT`、`PATCH`、`DELETE`）。

这时，客户端发出的 HTTP 请求，要加上 `X-HTTP-Method-Override` 属性，告诉服务器应该使用哪一个动词，覆盖 `POST` 方法。

```
POST /api/products/4 HTTP/1.1  
X-HTTP-Method-Override: PUT
```

上面代码中，`X-HTTP-Method-Override` 指定本次请求的方法是 `PUT`，而不是 `POST`。

### 2.4 宾语必须是名词

宾语就是 API 的 URL，是 HTTP 动词作用的对象。它应该是名词，不能是动词。比如，`/products` 这个 URL 就是正确的，而下面的 URL 不是名词，所以都是错误的。

```
GET /getAllProducts
GET /getProductById/1
POST /createNewProduct
POST /deleteAllProducts
POST /deleteProductById/1
```

### 2.5 使用复数做资源名称

既然 URL 是名词，那么应该使用复数，还是单数？

这没有统一的规定，但是常见的操作是读取一个集合，比如 `GET /products`（获取所有商品），这里明显应该是复数。

为了统一起见，建议都使用复数 URL，比如 `GET /products/1` 要好于 `GET /product/1`。

### 2.6 避免多级 URL

常见的情况是，资源需要多级分类，因此很容易写出多级的 URL，比如获取某个分类的某个商品。

`GET /categories/12/products/2`

这种 URL 不利于扩展，语义也不明确，往往要想一会，才能明白含义。

更好的做法是，除了第一级，其它级别都用查询字符串表达。

`GET /products/2?category_id=12`

下面是另一个例子，查询已上架的商品。你可能会设计成下面的 URL。

`GET /products/discontinued`

其实使用下面的查询字符串的写法明显更好。

`GET /products?discontinued=false`

### 2.7 搜索、排序、筛选和分页

所有这些操作都只是对一个数据集的查询，我们无须用一系列新的 API 来处理这些操作，只需要使用 GET 方法的 API 附加查询参数。

举例说明：

**搜索**：在搜索商品列表时，API 端点应该是 `GET /products?search=Forrest%20Gump`

**排序**：如果客户想要获取商品的排序列表，端点 `GET /products` 应该在查询中接受排序参数，例如
`GET /products?sort=rank&order=desc` 会按照商品评级降序排列。【**建议**】如非必要，排序操作应该在客户端而不是在服务器端进行，这样可以避免服务器对结果集进行排序产生的额外压力。

**筛选**：若要筛选数据集，我们可以通过查询参数传递各种选项。例如 `GET /products?category=books&discontinued=false` 将过滤指定类别的已上架的商品列表数据。

**分页**：当数据集太大时，我们必须将数据集划分为更小的块，这有助于提高性能并更容易处理响应。例如 `GET /products?page=2&page_size=15`

## 三、HTTP 状态码

### 3.1 始终使用精确的状态码

客户端的每一次请求，服务器都必须给出回应。回应包括 HTTP 状态码和数据两部分。

HTTP 状态码是一个三位数，分五个类别。

- `1xx`：Informational - 相关信息
- `2xx`：Success - 成功
- `3xx`：Redirection - 重定向
- `4xx`：Client Error - 客户端错误
- `5xx`：Server Error - 服务器错误

这五大类总共包含上百个状态码，覆盖了绝大部分可能遇到的情况。每一种状态码都有标准的（或者约定的）解释，客户端只需查看状态码，就可以判断出发生了什么情况，所以服务器应该始终返回尽可能**精确**的状态码。

API 不需要 `1xx` 状态码，下面介绍其它四类状态码的精确含义。

### 3.2 2xx 状态码

`200` 状态码表示操作成功，但是不同的方法可以返回更精确的状态码。

```
GET: 200 OK
POST: 201 Created
PUT: 200 OK
PATCH: 200 OK
DELETE: 204 No Content
```

上面代码中，

`POST` 返回 `201` 状态码，表示生成了新的资源；

`DELETE` 返回 `204` 状态码，表示资源已经不存在。

此外，`202 Accepted` 状态码表示服务器已经接受请求，但尚未进行处理，会在未来再处理，这个状态码被设计用来将请求交由另外一个进程或者服务器来进行处理，或者是对请求进行批处理的情形。

下面是一个例子。

```json
HTTP/1.1 202 Accepted

{
  "task": {
    "href": "/api/jobs/12345",
    "id": "12345"
  }
}
```

有两种情况 `202 Accepted` 特别适合：

- 如果资源将作为将来要处理的结果而创建 —— 例如在一个任务完成之后。
- 如果资源已经以某种方式存在，但不应将其解释为错误。

### 3.3 3xx 状态码

API 用不到 `301` 状态码（永久重定向）和 `302` 状态码（暂时重定向，`307` 也是这个含义），因为它们可以由应用级别返回，浏览器会直接跳转，API 级别可以不考虑这两种情况。

API 用到的 `3xx` 状态码，主要是` 303 See Other`，表示参考另一个 URL。它与 `302` 和 `307` 的含义一样，也是"暂时重定向"，区别在于 `302` 和` 307` 用于 ` GET` 请求，而 `303` 用于 `POST`、`PUT` 和 `DELETE` 请求。收到 `303` 以后，浏览器不会自动跳转，而会让用户自己决定下一步怎么办。下面是一个例子。

```
HTTP/1.1 303 See Other
Location: /api/products/2
```

### 3.4 4xx 状态码

`4xx` 状态码表示客户端错误，适用于错误似乎是由客户端引起的情况。主要有下面几种。

- `400 Bad Request`：服务器无法理解客户端的请求，未做任何处理。当 `4xx` 没有适当的状态代码时，将使用此状态码。
- `401 Unauthorized`：用户未提供身份验证凭据，或者没有通过身份验证。
- `403 Forbidden`：用户通过了身份验证，但是不具有访问资源所需的权限。
- `404 Not Found`：所请求的资源不存在、不可用或不希望澄清其存在。
- `405 Method Not Allowed`：方法不允许，表示服务器明白请求中指定的的方法，但是目标资源不支持。
- `406 Not Acceptable`：用户请求的格式不可得（比如用户请求 JSON 格式，但是只有 XML 格式）。
- `410 Gone`：所请求的资源已从这个地址转移，不再可用。
- `415 Unsupported Media Type`：服务器无法处理请求消息的有效负载的 MIME 类型和内容编码，从而拒绝接受客户端的请求。
- `422 Unprocessable Entity`：请求的有效负载正文中包含的数据在语法上是正确的, 但内容是推测性错误的，无法处理。
- `429 Too Many Requests`：在一定的时间内客户端发送了太多的请求，即超出了“频次限制”。

### 3.5 5xx 状态码

`5xx` 状态码表示服务端错误，即服务器未能满足请求。一般来说，API 不会向用户透露服务器的详细信息，所以只要两个状态码就够了。

- `500 Internal Server Error`：客户端请求有效，服务器处理时发生了意外。
- `503 Service Unavailable`：服务器尚未处于可以接受请求的状态。

## 四、服务器回应

### 4.1 不要返回纯本文

API 返回的数据格式，不应该是纯文本，而应该是一个 JSON 对象，因为这样才能返回标准的结构化数据。所以，服务器回应的 HTTP 头的 `Content-Type` 属性要设为 `application/json`。

客户端请求时，也要明确告诉服务器，可以接受 JSON 格式，即请求的 HTTP 头的 `ACCEPT` 属性也要设成 `application/json`。下面是一个例子。

```
GET /orders/2 HTTP/1.1 
Accept: application/json
```

### 4.2 发生错误时，不要返回 200 状态码

有一种不恰当的做法是，即使发生错误，也返回 200 状态码，把错误信息放在数据体里面，就像下面这样。

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "failure",
  "data": {
    "error": "Expected at least two items in list."
  }
}
```

上面代码中，只有在解析数据体之后，才能得知操作失败。

这样的做法实际上取消了状态码，其糟糕的语义是完全不可取的。正确的做法是，状态码应反映发生的错误，具体的错误信息放在数据体里面返回。下面是一个例子。

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid payload.",
  "detail": {
     "surname": "This field is required."
  }
}
```

### 4.3 提供链接

API 的使用者未必知道 URL 是怎么设计的。一个解决方法就是，在回应中，给出相关链接，便于下一步操作。这样用户只要记住一个 URL，就可以发现其它的 URL。这种方法叫做 HATEOAS。

举例来说，GitHub 的 API 都在 api.github.com 这个域名。访问它，就可以得到其它 URL。

```json
{
  ...
  "feeds_url": "https://api.github.com/feeds",
  "followers_url": "https://api.github.com/user/followers",
  "following_url": "https://api.github.com/user/following{/target}",
  "gists_url": "https://api.github.com/gists{/gist_id}",
  "hub_url": "https://api.github.com/hub",
  ...
}
```

上面的回应中，挑一个 URL 访问，又可以得到别的 URL。对于用户来说，不需要记住 URL 设计，只要从 api.github.com 一步步查找就可以了。

HATEOAS 的格式没有统一规定，上面例子中，GitHub 将它们与其它属性放在一起。更好的做法应该是，将相关链接与其它属性分开。

```json
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "In progress",
   "links": {[
    { "rel":"cancel", "method": "delete", "href":"/api/status/1000" } ,
    { "rel":"edit", "method": "put", "href":"/api/status/1000" }
  ]}
}
```

## 五、版本控制

当我们的 API 被大量客户端消费时，通过一些重大更改来升级 API 将会导致使用我们的 API 破坏现有产品或服务。

`http://api.ourdomain.com/v1/products/1` 是一个很好的例子，它在路径中具有 API 的版本号。如果有任何重大更新，我们可以将一组新的 API 命名为 v2 或 v2.x.x。

根据[语义化版本控制规范](https://semver.org/lang/zh-CN/#%E6%91%98%E8%A6%81)，版本格式：主版号.次版号.修订号，版本号递增规则如下：

1. 主版号：当你做了不兼容的 API 修改；
2. 次版号：当你做了向下兼容的功能性新增；
2. 修订号：当你做了向下兼容的问题修正。

##

## 六、参考链接

- [API Design](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design), by MicroSoft Azure
- [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines)，by Microsoft
- [RESTful API Design: 13 Best Practices to Make Your Users Happy](https://blog.florimondmanca.com/restful-api-design-13-best-practices-to-make-your-users-happy), by Florimond Manca
- [RESTful API Designing Guidelines  —  The Best Practices](https://hackernoon.com/restful-api-designing-guidelines-the-best-practices-60e1d954e7c9), by Mahesh Haldar

---
**[⬆  返回目录](###目录)**
