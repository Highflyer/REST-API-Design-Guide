# REST API 设计规范与最佳实践

>作者：Ray

**目录**

  - [一、REST 简介](#一rest-简介)
    - [1.1 术语](#11-术语)
    - [1.2 成熟度模型](#12-成熟度模型)
  - [二、URL 设计](#二url-设计)
    - [2.1. 动词 + 宾语](#21-动词--宾语)
    - [2.2 动词的覆盖](#22-动词的覆盖)
    - [2.3 宾语必须是名词](#23-宾语必须是名词)
    - [2.4 使用复数做资源名称](#24-使用复数做资源名称)
    - [2.5 避免多级 URL](#25-避免多级-url)
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
  - [六、参考文献](#六参考文献)

---

## 一、REST 简介

RESTful 是目前最流行的 API 设计规范，用于 Web 数据接口的设计。

它的大原则容易把握，但是细节不容易做对。本文总结 RESTful 的设计细节，介绍如何设计出易于理解和使用的 API。

### 1.1 术语

如下是与 REST API 相关的非常重要的术语：

- **资源** - 资源是某个事物的对象或表示形式，它与某事物有一些关联的数据，并且可以对其进行操作的方法集。例如，用户、订单和文章是资源，删除、添加、更新是要对这些资源执行的操作。
- **集合** - 集合是资源集合，例如，articles 是 article 资源的集合。
- **URI** - 统一资源标识符（Uniform Resource Identifier）是一个用于标识某一互联网资源名称的字符串，可以通过它定位资源，并对其执行某些操作。
- **端点** - 端点（Endpoint）是动词和 URI 的组合。例如：`GET: /articles`。
- **状态码** - 一个响应的状态由其状态代码（Status Code）指定。

### 1.2 成熟度模型

2008 年，Leonard Richardson 提议对 Web API 使用以下成熟度模型：

- 级别 0：定义一个 URI，所有操作是对此 URI 发出的 POST 请求。
- 级别 1：为各个资源单独创建 URI。
- 级别 2：使用 HTTP 方法来定义对资源执行的操作。
- 级别 3：使用超媒体（HATEOAS，如下所述）。

根据 Fielding 的定义，级别 3 对应于某个真正的 RESTful API。 在实践中，许多发布的 Web API 大致处于级别 2。

## 二、URL 设计

### 2.1. 动词 + 宾语

RESTful 的核心思想就是，客户端发出的数据操作指令都是"动词 + 宾语"的结构。比如，`GET /articles` 这个命令，`GET` 是动词，`/articles` 是宾语。

动词通常就是五种 HTTP 方法，对应 CRUD 操作。

- `GET` 检索位于指定 URI 处的资源的表示形式。 响应消息的正文包含所请求资源的详细信息。
- `POST` 在指定的 URI 处创建新资源。 请求消息的正文将提供新资源的详细信息。 请注意，POST 还用于触发不实际创建资源的操作，如登录、注销。
- `PUT` 在指定的 URI 处创建或替换资源。 请求消息的正文指定要创建或更新的资源。
- `PATCH` 对资源执行部分更新。 请求正文包含要应用到资源的一组更改。
- `DELETE` 删除位于指定 URI 处的资源。

根据 HTTP 规范，动词一律大写。

特定请求的影响应取决于资源是集合还是单个子项。 下表汇总了使用电子商务示例的大多数 RESTful 实现所采用的常见约定。 请注意，并非所有这些请求都可以实现；这取决于特定方案。

| 资源                  | POST        | GET          | PUT                | DELETE       |
|---------------------|-------------|--------------|--------------------|--------------|
| /customers          | 创建新客户       | 检索所有客户       | 批量更新客户             | 删除所有客户       |
| /customers/1        | 错误          | 检索客户 1 的详细信息 | 如果客户 1 存在，则更新其详细信息 | 删除客户 1       |
| /customers/1/orders | 创建客户 1 的新订单 | 检索客户 1 的所有订单 | 批量更新客户 1 的订单       | 删除客户 1 的所有订单 |

### 2.2 动词的覆盖

有些客户端只能使用 `GET` 和 `POST` 这两种方法。服务器必须接受 `POST` 模拟其它三个方法（`PUT`、`PATCH`、`DELETE`）。

这时，客户端发出的 HTTP 请求，要加上 `X-HTTP-Method-Override` 属性，告诉服务器应该使用哪一个动词，覆盖 `POST` 方法。

```
POST /api/Person/4 HTTP/1.1  
X-HTTP-Method-Override: PUT
```

上面代码中，`X-HTTP-Method-Override` 指定本次请求的方法是 `PUT`，而不是 `POST`。

### 2.3 宾语必须是名词

宾语就是 API 的 URL，是 HTTP 动词作用的对象。它应该是名词，不能是动词。比如，`/articles` 这个 URL 就是正确的，而下面的 URL 不是名词，所以都是错误的。
```
**Don't**
GET: /getAllArticles
GET: /getArticlesById/1
POST: /createNewArticle
POST: /deleteAllArticles
POST: /deleteArticalById/1

**Do**
GET: /articles
GET: /articles/1
POST: /articles
DELETE: /articles
DELETE: /articles/1
```

### 2.4 使用复数做资源名称

既然 URL 是名词，那么应该使用复数，还是单数？

这没有统一的规定，但是常见的操作是读取一个集合，比如 `GET /articles`（读取所有文章），这里明显应该是复数。

为了统一起见，建议都使用复数 URL，比如 `GET /articles/2` 要好于 `GET /article/2`。

### 2.5 避免多级 URL

常见的情况是，资源需要多级分类，因此很容易写出多级的 URL，比如获取某个作者的某一类文章。

```
GET /authors/12/articles/2
```

这种 URL 不利于扩展，语义也不明确，往往要想一会，才能明白含义。

更好的做法是，除了第一级，其它级别都用查询字符串表达。

```
GET /articles/12?authorId=2
```

下面是另一个例子，查询已发布的文章。你可能会设计成下面的 URL。

```
GET /articles/published
```

查询字符串的写法明显更好。

```
GET /articles?published=true
```

### 2.6 搜索、排序、筛选和分页

所有这些操作都只是对一个数据集的查询，我们无须用一系列新的 API 来处理这些操作，只需要将查询参数追加到 GET 方法 API 中。

举例说明：

**搜索**：在搜索文章列表时，API 端点应该是 `GET /articles?search=Forrest Gump`

**排序**：如果客户想要获取文章的排序列表，端点 `GET /auticles` 应该在查询中接受多个排序参数，例如
`GET /articles?sort=rank&type=desc 会按照文章评级降序排列。**【建议】**如非必要，排序操作应在客户端而不是在服务器端进行，这样可以有效避免服务器进行排序产生的额外压力。

**筛选**：若要筛选数据集，我们可以通过查询参数传递各种选项。例如 `GET /artiles?category=gaming&location=india` 将用类别和地点是过滤文章列表数据。

**分页**：当数据集太大时，我们将数据集划分为更小的块，这有助于提高性能并更容易处理响应。 例如 `GET / articles?page=2&page_size=15` 表示获取每页 15 条记录第 2 页的文章列表。

## 三、HTTP 状态码

### 3.1 始终使用精确的状态码

客户端的每一次请求，服务器都必须给出回应。回应包括 HTTP 状态码和数据两部分。

HTTP 状态码就是一个三位数，分成五个类别。

- 1xx：Informational - 相关信息
- 2xx：Success - 成功
- 3xx：Redirection - 重定向
- 4xx：Client Error - 客户端错误
- 5xx：Server Error - 服务器错误

这五大类总共包含 100 多种状态码，覆盖了绝大部分可能遇到的情况。每一种状态码都有标准的（或者约定的）解释，客户端只需查看状态码，就可以判断出发生了什么情况，所以服务器应该始终返回尽可能精确的状态码。

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

上面代码中，`POST` 返回 `201` 状态码，表示生成了新的资源；`DELETE` 返回 `204`状态码，表示资源已经不存在。

此外，`202 Accepted` 状态码表示服务器已经收到请求，但还未进行处理，会在未来再处理，通常用于异步操作。下面是一个例子。

```json
HTTP/1.1 202 Accepted

{
  "task": {
    "href": "/api/company/job-management/jobs/2130040",
    "id": "2130040"
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
Location: /api/articles/1
```

### 3.4 4xx 状态码

`4xx` 状态码表示客户端错误，主要有下面几种。

- `400 Bad Request`：服务器不理解客户端的请求，未做任何处理。
- `401 Unauthorized`：用户未提供身份验证凭据，或者没有通过身份验证。
- `403 Forbidden`：用户通过了身份验证，但是不具有访问资源所需的权限。
- `404 Not Found`：所请求的资源不存在，或不可用。
- `405 Method Not Allowed`：用户已经通过身份验证，但是所用的 HTTP 方法不在他的权限之内。
- `410 Gone`：所请求的资源已从这个地址转移，不再可用。
- `415 Unsupported Media Type`：客户端要求的返回格式不支持。比如，API 只能返回 JSON 格式，但是客户端要求返回 XML 格式。
- `422 Unprocessable Entity`：客户端上传的附件无法处理，导致请求失败。
- `429 Too Many Requests`：客户端的请求次数超过限额。

### 3.5 5xx 状态码

`5xx` 状态码表示服务端错误。一般来说，API 不会向用户透露服务器的详细信息，所以只要两个状态码就够了。

- `500 Internal Server Error`：客户端请求有效，服务器处理时发生了意外。
- `503 Service Unavailable`：服务器无法处理请求，一般用于网站维护状态。

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

上面代码中，解析数据体以后，才能得知操作失败。

这张做法实际上取消了状态码，这是完全不可取的。正确的做法是，状态码反映发生的错误，具体的错误信息放在数据体里面返回。下面是一个例子。


HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "error": "Invalid payoad.",
  "detail": {
     "surname": "This field is required."
  }
}

### 4.3 提供链接

API 的使用者未必知道，URL 是怎么设计的。一个解决方法就是，在回应中，给出相关链接，便于下一步操作。这样的话，用户只要记住一个 URL，就可以发现其它的 URL。这种方法叫做 HATEOAS。

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

`http://api.ourservice.com/v1/articles/1` 是一个很好的例子，它在路径中具有 API 的版本号。如果有任何重大更新，我们可以将一组新的 API 命名为 v2 或 v2.x.x。

**附**：
版本格式：主版本号.次版本号.修订号，版本号递增规则如下：

1. 主版本号：当我们做了不兼容的 API 修改；
2. 次版本号：当我们做了向下兼容的功能性新增；
2. 修订号：当我们做了向下兼容的问题修正。

## 六、参考文献

- [API design](https://docs.microsoft.com/en-us/azure/architecture/best-practices/api-design), by MicroSoft Azure
- [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines)，by Microsoft
- [RESTful API Design: 13 Best Practices to Make Your Users Happy](https://blog.florimondmanca.com/restful-api-design-13-best-practices-to-make-your-users-happy), by Florimond Manca
- [RESTful API Designing guidelines — The best practices](https://hackernoon.com/restful-api-designing-guidelines-the-best-practices-60e1d954e7c9), by Mahesh Haldar
- [Semantic Versioning 2.0.0](https://semver.org/)

---