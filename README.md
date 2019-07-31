# RESTful API Design

## 什么是 REST？

REST（Representational State Transfer，表述性状态转换），是 Roy Fielding 博士在 2000 年他的博士论文中提出来的一种软件架构软件架构风格。它是一种针对网络应用的设计和开发方式，可以降低开发的复杂性，提高系统的可伸缩性。是一种定义了一组用于创建`Web`服务的软件架构风格。匹配或兼容 `REST` 架构风格的`Web`服务称为 RESTful Web 服务。RESTful Web 服务允许客户端发出`URI`（Uniform Resource Identifier，统一资源标识符）访问和操作`Web`资源的请求。

## 常见 HTTP 请求方法

- `GET`, `DELETE`, `HEAD` 方法，参数风格为标准的 `GET` 风格的参数，如 `url?a=1&b=2`
- `POST`, `PUT`, `PATCH`, `OPTIONS` 方法
  - 默认情况下请求实体会被视作标准 json 字符串进行处理，当然，依旧推荐设置头信息的 `Content-Type` 为 `application/json`
  - 在一些特殊接口中（会在文档中说明），可能允许 `Content-Type` 为 `application/x-www-form-urlencoded` 或者 `multipart/form-data` ，此时请求实体会被视作标准 `POST` 风格的参数进行处理

关于方法语义的说明：

- `OPTIONS` 用于获取资源支持的所有 HTTP 方法
- `HEAD` 用于只获取请求某个资源返回的头信息
- `GET` 用于从服务器获取某个资源的信息
  - 完成请求后返回状态码 `200 OK`
  - 完成请求后需要返回被请求的资源详细信息
- `POST` 用于创建新资源
  - 创建完成后返回状态码 `201 Created`
  - 完成请求后需要返回被创建的资源详细信息
- `PUT` 用于完整的替换资源或者创建指定身份的资源，比如创建 id 为 123 的某个资源
  - 如果是创建了资源，则返回 `201 Created`
  - 如果是替换了资源，则返回 `200 OK`
  - 完成请求后需要返回被修改的资源详细信息
- `PATCH` 用于局部更新资源
  - 完成请求后返回状态码 `200 OK`
  - 完成请求后需要返回被修改的资源详细信息
- `DELETE` 用于删除某个资源
  - 完成请求后返回状态码 `204 No Content`

在 REST 中，开发人员显式地使用 HTTP 方法，对系统资源进行创建、读取、更新和删除的操作。

***一个好的 RESTful API 只允许第三方调用者使用这五个 HTTP 动词进行数据交互，并且在 URL 段里面不出现任何其他的动词。***

## HTTP 请求方法幂等性

### 什么是幂等性？

在`HTTP/1.1`规范中，幂等性的定义是：

```Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request.```

从定义上看，HTTP 方法的幂等性是指一次和多次请求某一个资源应该具有同样的作用。

通俗点说，`幂等`就是一次或多次同样的请求，得到的结果也是同样的。不管多少次`GET`请求同一个资源，得到的结果是完全相同的，不会有任何副作用。`POST`每请求一次都会新增资源，所以是非幂等的。`PUT`方法同样的数据提交多少次，该资源都与第一次请求后的结果相同（`PUT`也可以认为是创建或更新资源，当指定 ID 的资源不存在时创建，存在时则更新）。`DELETE`每请求一次都会删除资源，所以是非幂等的。`PATCH`方法

### 常见方法的幂等性

| 方法名 | 幂等性 | 安全性 |
| ------ | ------ | ------|
| GET    | 是     ||
| POST   | 否     ||
| PUT    | 是     ||
| PATCH  | 否     ||
| DELETE | 是     ||

### 如何在分布式系统中实现幂等性

//TODO

## HTTP 状态码

### 请求成功

* 200 **OK** : 请求执行成功并返回相应数据，如 `GET` 成功
* 201 **Created** : 对象创建成功并返回相应资源数据，如 `POST` 成功；创建完成后响应头中应该携带头标 `Location` ，指向新建资源的地址
* 202 **Accepted** : 接受请求，但无法立即完成创建行为，比如其中涉及到一个需要花费若干小时才能完成的任务。返回的实体中应该包含当前状态的信息，以及指向处理状态监视器或状态预测的指针，以便客户端能够获取最新状态。
* 204 **No Content** : 请求执行成功，不返回相应资源数据，如 `PATCH` ， `DELETE` 成功

### 重定向

**重定向的新地址都需要在响应头 `Location` 中返回**

* 301 **Moved Permanently** : 被请求的资源已永久移动到新位置
* 302 **Found** : 请求的资源现在临时从不同的 URI 响应请求
* 303 **See Other** : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该使用 `GET` 方法进行请求。比如在创建已经被创建的资源时，可以返回 `303`
* 307 **Temporary Redirect** : 对应当前请求的响应可以在另一个 URI 上被找到，客户端应该保持原有的请求方法进行请求

### 条件请求

* 304 **Not Modified** : 资源自从上次请求后没有再次发生变化，主要使用场景在于实现数据缓存。
* 409 **Conflict** : 请求操作和资源的当前状态存在冲突。主要使用场景在于实现并发控制。
* 412 **Precondition Failed** : 服务器在验证在请求的头字段中给出先决条件时，没能满足其中的一个或多个。主要使用场景在于实现并发控制。

### 客户端错误

* 400 **Bad Request** : 请求体包含语法错误
* 401 **Unauthorized** : 需要验证用户身份，如果服务器就算是身份验证后也不允许客户访问资源，应该响应 `403 Forbidden` 。如果请求里有 `Authorization` 头，那么必须返回一个 [`WWW-Authenticate`](https://tools.ietf.org/html/rfc7235#section-4.1) 头
* 403 **Forbidden** : 服务器拒绝执行
* 404 **Not Found** : 找不到目标资源
* 405 **Method Not Allowed** : 不允许执行目标方法，响应中应该带有 `Allow` 头，内容为对该资源有效的 HTTP 方法
* 406 **Not Acceptable** : 服务器不支持客户端请求的内容格式，但响应里会包含服务端能够给出的格式的数据，并在 `Content-Type` 中声明格式名称
* 410 **Gone** : 被请求的资源已被删除，只有在确定了这种情况是永久性的时候才可以使用，否则建议使用 `404 Not Found`
* 413 **Payload Too Large** : `POST` 或者 `PUT` 请求的消息实体过大
* 415 **Unsupported Media Type** : 服务器不支持请求中提交的数据的格式
* 422 **Unprocessable Entity** : 请求格式正确，但是由于含有语义错误，无法响应
* 428 **Precondition Required** : 要求先决条件，如果想要请求能成功必须满足一些预设的条件

### 服务端错误

* 500 **Internal Server Error** : 服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。
* 501 **Not Implemented** : 服务器不支持当前请求所需要的某个功能。
* 502 **Bad Gateway** : 作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应。
* 503 **Service Unavailable** : 由于临时的服务器维护或者过载，服务器当前无法处理请求。这个状况是临时的，并且将在一段时间以后恢复。如果能够预计延迟时间，那么响应中可以包含一个 `Retry-After` 头用以标明这个延迟时间（内容可以为数字，单位为秒；或者是一个 [HTTP 协议指定的时间格式](http://tools.ietf.org/html/rfc2616#section-3.3)）。如果没有给出这个 `Retry-After` 信息，那么客户端应当以处理 500 响应的方式处理它。

`501` 与 `405` 的区别是：`405` 是表示服务端不允许客户端这么做，`501` 是表示客户端或许可以这么做，但服务端还没有实现这个功能

# 设计一个好的 RESTful API

## 协议

API 与用户的通信协议，总是使用`HTTPs`协议。

## 域名

应该尽量将API部署在专用域名之下。

``` 
https://api.example.com
```

如果确定 API 很简单，不会有进一步扩展，可以考虑放在主域名下。

``` 
https://example.org/api/
```

## 版本

常见的三种版本控制方式：

1. 在 URI 中放版本信息：`GET /v1/users/1`
2. Accept Header：`Accept: application/json+v1`
3. Accept Header：`Accept: vnd.example-com.foo+json; version=1.0`
4. 自定义 Header：`X-Api-Version:1`

第一种最明显，最方便，最常用。规范推荐将版本号放在 HTTP 头信息中。

## RESTful 设计

**写在最前面：最好不要在 URI 中出现动词！类似 Login、Logout、Auth等特定操作的除外。**

### 方法 + 宾语

RESTful 的核心思想就是，客户端发出的数据操作指令都是"方法 + 宾语"的结构。

#### 方法

方法指的就是`HTTP 请求方法`，常用的有`GET`, `DELETE`,`POST`, `PUT`, `PATCH` 。

#### 宾语

宾语就是`API`的`URI`，是`HTTP`动词作用的对象，是资源。它应该是名词，不能是动词。

### URI

URI 表示资源，资源一般对应服务器端领域模型中的实体类。

#### URI 规范

1. 不用大写
2. 用`-`不用`_`
3. 参数列表要 encode
4. URI 中的名词表示资源集合，使用复数形式。

#### 资源

URI 表示资源的两种方式：资源集合、单个资源。

**资源集合：** 

```
/categories // 所有类别
/categories/1/articles // 类别为 1 的所有文章
```

**单个资源**

```
/categories/1 // 类别1
/articles/14321 // ID 为 14321 的文章
```

##### 避免层级过深的 URI

`/`在url中表达层级，用于**按实体关联关系进行对象导航**，一般根据id导航。

过深的导航容易导致url膨胀，不易维护。

如 `GET /zoos/1/areas/3/animals/4`，尽量使用查询参数代替路径中的实体导航，如`GET /animals?zoo=1&area=3`；

如 `GET users/1/categories/1/articles/14321`，只需 `GET /articles/14321`就足够。

##### 资源不能是动词，但是可以是一种服务

如 ID 为 1 的账户给 ID 为 2 的账户转账 500

**错误写法**

```
POST /accounts/1/transfer/500/to/2
```

**正确写法**

```
POST /transanctions?from=1&to=2&amount=500.00 // 这里只是举个例子，其实应该写在 Body 里
```

##### HTTP 方法

**GET 查询**

```
GET /categories
GET /categories/1
GET /categories/1/articles
```

**POST 创建单个资源**

```
POST /users // 新增用户
POST /users/1/roles // 为 ID 为 1 的用户分配角色
```

**PUT 创建或更新资源（全量）**

```
PUT /users/10 // 新增 ID 为 10 的用户
PUT /users/10 // 修改(覆盖) ID 为 10 的用户的信息
```

**DELETE 删除资源**

```
DELETE /users/10
DELETE /users/10/roles
```

**PATCH 更新资源（局部）**

```
PATCH /users/10 // 局部更新 ID 为 10 的用户的信息
```

#### 复杂查询

查询可以带类似以下参数：

| .        | 示例                       | 备注                                         |
| :------- | :------------------------- | :------------------------------------------- |
| 过滤条件 | `?type=1&age=16`           | 允许一定的uri冗余，如`/zoos/1`与`/zoos?id=1` |
| 排序     | `?sort=age,desc`           |                                              |
| 投影     | `?whitelist=id,name,email` |                                              |
| 分页     | `?limit=10&offset=3`       |                                              |

#### Rsponse

在操作成功时，返回成功的 HTTP 状态码 2xx，以及具体数据。若非需要，**不建议**进行类似以下包装，直接返回数据 JSON 对象即可：

```json
{
  "success":true,
  "data":{"id":1,"name":"oktfolio"},
}
```

在操作失败时，返回失败的 HTTP 状态码如 400，以及具体错误信息（包括自定义错误码及描述等）。

```json
{
  "code": 40011,
  "message": "username cannot be blank",
  "status": "BAD_REQUEST",
  "timestamp": "2019-05-21T12:40:08Z"
}
```

若错误验证需要字段细分，则如下

```json

{
  "code" : 40010,
  "message" : "Validation Failed",
  "errors" : [
    {
      "code" : 40011,
      "field" : "username",
      "message" : "username cannot have fancy characters"
    },
    {
       "code" : 40012,
       "field" : "password",
       "message" : "password cannot be blank"
    }
  ]
}
```

### JSON 风格

#### 时间格式

遵循 [ISO 8601](https://www.iso.org/obp/ui/#iso:std:iso:8601:ed-3:v1:en)([Wikipedia](https://en.wikipedia.org/wiki/ISO_8601)) 建议的格式：

- 日期 `2014-07-09`
- 时间 `14:31:22+0800`
- 具体时间 `2007-11-06T16:34:41Z`
- 持续时间 `P1Y3M5DT6H7M30S` （表示在一年三个月五天六小时七分三十秒内）
- 时间区间 `2007-03-01T13:00:00Z/2008-05-11T15:30:00Z` 、 `2007-03-01T13:00:00Z/P1Y2M10DT2H30M` 、 `P1Y2M10DT2H30M/2008-05-11T15:30:00Z`
- 重复时间 `R3/2004-05-06T13:00:00+08/P0Y6M5DT3H0M0S` （表示从2004年5月6日北京时间下午1点起，在半年零5天3小时内，重复3次）

### 货币名称

货币名称可以参考 ISO 4217([Wikipedia](http://en.wikipedia.org/wiki/ISO_4217)) 中的约定，标准为货币名称规定了三个字母的货币代码，其中的前两个字母是 ISO 3166-1([Wikipedia](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)) 中定义的双字母国家代码，第三个字母通常是货币的首字母。在货币上使用这些代码消除了货币名称（比如 dollar ）或符号（比如 $ ）的歧义。

### 时区

客户端请求服务器时，如果对时间有特殊要求（如某段时间每天的统计信息），则可以参考 [IETF 相关草案](http://tools.ietf.org/html/draft-sharhalakis-httptz-05) 增加请求头 `Timezone` 。

```
Timezone: 2016-11-06 23:55:52+08:00;;Asia/Shanghai
```

具体格式说明：

```
Timezone: RFC3339 约定的时间格式;POSIX 1003.1 约定的时区字符串;tz datebase 里的时区名称
```

客户端最好提供所有字段，如果没有办法提供，则应该使用空字符串

如果客户端请求时没有指定相应的时区，则服务端默认使用最后一次已知时区或者 [UTC](http://zh.wikipedia.org/wiki/协调世界时) 时间返回相应数据。

PS 考虑到存在[夏时制](https://en.wikipedia.org/wiki/Daylight_saving_time)这种东西，所以不推荐客户端在请求时使用 Offset 。



