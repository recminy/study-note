# RESTful

REST Representational State Transfer的缩写,意思：表现层状态转化
RESTful是目前最流行一种互联网软件架构。它结构清晰、易于理解、扩展方便，对于RESTful架构：

- 每一个URI代表一种资源；

- 客户端和服务器之间，传递这种资源的某种表现层
- 客户端通过HTTP动词，对服务器端资源进行操作，实现"表现层状态转化"。

## 设计指南

### 专属域名及版本

应将API部署在专属域名下，且在URL中放入版本如：https://api.domain.com/v1，版本v1也可放入http头信息中，但没有URL方式直观。

### PATH(URI路径)

- https://api.domain.com/v1/zoos
- https://api.domain.com/v1/animals
- https://api.domain.com/v1/employees

### HTTP动作，动词

- GET=>(SELECT),从服务器取出一个或多个资源；
- POST=>(CREATE),在服务器新建一个资源；
- PUT=>(UPDATE),在服务器更新资源（客户端提供改变后的完整资源）;
- PATCH=>(UPDATE,同PUT不同的是，这里是局部更新;
- DELETE=>(DELETE),从服务器删除资源;
- HEAD:获取资源的元数据;
- OPTION:获取信息，关于资源的哪些属性是客户端可以改变的。

>GET /zoos：列出所有动物园
>
>POST /zoos：新建一个动物园
>
>GET /zoos/ID：获取某个指定动物园的信息
>
>PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
>
>PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
>
>DELETE /zoos/ID：删除某个动物园
>
>GET /zoos/ID/animals：列出某个指定动物园的所有动物
>
>DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物

### 过滤信息（Filtering）（Query）

如果记录数量很多，服务器不可能都将它们返回给用户。API应该提供参数，过滤返回结果。

```http
?page=10
?current_page=2
?per_page=20&page=1
?limit=10
?order_by=name&order=asc
?is_published=true
```

### HTTP状态码（Status Code）

- 2x系列表示成功
  - 200 OK - [GET]：服务器成功返回用户请求的数据；
  - 201CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功；
  - 202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）；
  - 204 NO CONTENT - [DELETE]：用户删除数据成功。

- 3x表示重定向系列
  - 301Moved Permanently（永久移动)；
  - 302 Found（发现）。
- 4x表示请求错误
  - 401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）；
  - 403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的；
  - 404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录；
  - 405 Method Not Allowed,请求的HTTP动作不被允许;
  - 406 Not Acceptable - [GET]：用户请求的格式不可得；
  - 409 HTTP Conflict - [POST/PUT/PACTH]: 和被请求的资源的当前状态之间存在冲突。
- 500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功

## URL设计

### 动词+宾语

RESTful 的核心思想就是，客户端发出的数据操作指令都是"动词 + 宾语"的结构。比如，`GET /articles`这个命令，`GET`是动词，`/articles`是宾语。

常用的动词语：GET/POST/PUT/PATCH/DELETE

### 动词覆盖

有些客户端只能使用`GET`和`POST`这两种方法。服务器必须接受`POST`模拟其他三个方法（`PUT`、`PATCH`、`DELETE`）。

这时，客户端发出的 HTTP 请求，要加上`X-HTTP-Method-Override`属性，告诉服务器应该使用哪一个动词，覆盖`POST`方法。

```http
POST /api/Person/4 HTTP/1.1  
X-HTTP-Method-Override: PUT
```

### 宾语必须是名词

宾语就是 API 的 URL，是 HTTP 动词作用的对象。它应该是名词，不能是动词。比如，`/articles`这个 URL 就是正确的，而下面的 URL 不是名词，所以都是错误的。

```http
/getAllContents
/createNewCar
/deleteAllRedCars
```

### 复数及避免多级URL

#### 复数

既然 URL 是名词，那么应该使用复数，还是单数？

这没有统一的规定，但是常见的操作是读取一个集合，比如`GET /articles`（读取所有文章），这里明显应该是复数。

为了统一起见，建议都使用复数 URL，比如`GET /articles/2`要好于`GET /article/2`。

#### 避免多级

常见的情况是，资源需要多级分类，因此很容易写出多级的 URL，比如获取某个作者的某一类文章。

```http
 GET /authors/12/categories/2
```

这种 URL 不利于扩展，语义也不明确，往往要想一会，才能明白含义。

更好的做法是，除了第一级，其他级别都用查询字符串表达。

```http
GET /authors/12?categories=2
# 查询已发布的文章
GET /articles?published=true
```

## Laravel中的RESTful

### 资源控制器操作处理

| Verb      | URI                    | Action  | Route Name     |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

## 附：URL构成

```http
scheme://host:port/path?query#fragment
```

- Scheme: 协议，如http(s),ftp,sftp,file等

- Host:主机名，如www.baidu.com

- Port: 端口：http默认80，https默认443

- Path:URI,主机上的目录或文件地址（虚拟）

- Query: 过滤信息

- Fragment：信息片段，字符串，用于指定网络资源中的某片断，html中的锚点

  example:

  - http://www.gushanxia.com/posts
  - http://www.gushanxia.com/posts/12
  - http://www.gushanxia.com/archives?keyword=RESTful&page=2#content
  - https://order.gushanxia.com/user/12
  - ftp://172.0.0.2:22/usr/local/mysql/

## 参考文档

- [阮一峰 - RESTful API 最佳实践](http://www.ruanyifeng.com/blog/2018/10/restful-api-best-practices.html)
- [阮一峰 - RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)
- [阮一峰 - 理解RESTful 架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)
- [资源控制器 - RESTful API在Laravel中的实践](https://learnku.com/docs/laravel/8.x/controllers/9368#resource-controllers)
- [GoWeb编程 - Web服务 - REST](https://learnku.com/docs/build-web-application-with-golang/083-rest/3205)

