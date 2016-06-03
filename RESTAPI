## 康乐智慧数据接口规范 V1.0.0 ##

----

#### 声明 ####
本文档所描述的内容, 全部归康乐智慧(北京)科技有限公司所有.  
未经本公司许可, 其他个人或组织不得以任何形式复制文档内容.
若发现侵权行为, 本公司将依法追究其法律责任.

#### 目录 ####
- 数据传输标准

    **1. 主动传输数据: 设备采集到数据以后, 主动向康乐服务器推送数据.**
    
    **1.1. HTTP JSON API**  
    **1.2. UDP API**
    
    **2. 被动传输数据: 设备开放数据服务, 由康乐服务器定时发出数据请求**
    
    **3. 影像数据传输**

- 数据内容标准

    **1. 设备的识别与被检测人的识别**
    
    **2. 数据内容**

- 数据安全标准

    **1. 数据通道安全**
    
    **2. 数据加密**

----

## 数据传输标准 ##

**1. 主动传输数据: 设备采集到数据以后, 主动向康乐服务器推送数据.**

**1.1. HTTP JSON API**

- 使用TLS(HTTPS)访问API

    为了保证通信数据的保密性和完整性, 需要使用TLS(HTTPS)访问API.  
    非TLS的请求不会重定向到TLS的URL, 而是会返回403 Forbidden.  
    因为使用重定向, 会造成敏感数据在第一次请求时已经泄露, 而且客户端流量也会翻倍.
  
- 数据压缩采用gzip (推荐)

    为了减少网络传输的数据量, 传输的数据可以使用gzip进行压缩.  
    客户端需要设定Request header信息, 来标识客户端可以接受压缩的内容  
    `Accept-Encoding: gzip, compress`  
    响应的Body大小超过100byte时, 内容会被压缩, 头信息里会包含压缩的类型  
    `Content-Encoding: gzip`

- 授权Token

    服务器会对所有的API请求进行Token验证.  
    Token的获取方法如下:  
    a. 登陆平台开发者页面, 生成Token  
    b. 通过OAuthAPI, 使用账户信息换取Token  
    服务器使用Token来标识特定的客户端, 请妥善保管Token.

- IP地址验证 (推荐)

    为了更有效地保证传输数据的安全, 客户端可以选择限定IP地址. 服务器将只接受由这些IP发出的API调用请求.  
    限定IP地址, 可以通过如下方法设置:  
    a. 登陆平台开发者页面, 设置发起请求的IP地址  
    b. 通过IP地址绑定API, 设置发起请求的IP地址

- 速度限制策略

    为了防止恶意请求行为, 我们对客户端的请求频率有一定的限制.  
    当请求超过该限制, 服务器会返回429错误码, 具体限制情况可以查询Http Response Header信息.  
    `X-Rate-Limit-Limit`: 当前Token, 在一个时间段内可调用次数  
    `X-Rate-Limit-Remaining`: 当前Token, 在一个时间段内剩余的可用次数  
    `X-Rate-Limit-Reset`: 当前Token, 距离时间段重置剩余的秒数

- 缓存功能 (推荐)

    为了提高与服务器的交互效率, 客户端可以选择使用缓存功能.  
    a. 使用ETag和If-None-Match  
    对特定的API请求, 服务器会通过Response Header返回ETag值, 如:  
    `ETag: W/"7e3e-1534a39a332"`  
    对相同的API发送请求时, 客户端可以将接收到ETag值, 通过Header发送出去  
    `If-None-Match: W/"7e3e-1534a39a332"`  
    服务器会对If-None-Match进行比较, 如果发现资源没有发生变化, 则会返回304状态码.  
    b. 使用Last-Modified  
    对特定的API请求, 服务器会通过Response Header返回Last-Modified值, 如:  
    `Last-Modified: Sun, 06 Mar 2016 04:40:04 GMT`  
    对相同的API发送请求时, 客户端可以将接收到Last-Modified值, 通过Header发送出去  
    `If-Modified-Since: Sun, 06 Mar 2016 04:39:51 GMT`  
    服务器会对If-Modified-Since进行比较, 如果发现资源没有发生变化, 则会返回304状态码.  
    当ETag与Last-Modified同时被指定时, ETag会被优先使用
  
- JSON数据结构及参数名称 (参考[Google JSON Style](https://google.github.io/styleguide/jsoncstyleguide.xml))
  ```
  object {
    string apiVersion?;      # 使用的API版本信息
    string context?;
    string id?;              # 服务器提供的响应标识符, 每次请求都不会相同 (UUID)
    string method?;
    object {
      string id?
    }* params?;
    object {
      string kind?;
      string fields?;        # 限定要返回的字段内容, 只有在GET, PATCH方法中有效
      string etag?;
      string id?;
      string lang?;
      string updated?;       # ISO8601格式的UTC时间 (RFC 3339)
      boolean deleted?;
      integer currentItemCount?;
      integer itemsPerPage?;
      integer startIndex?;
      integer totalItems?;   # 所有数据的件数
      integer pageIndex?;
      integer totalPages?;
      string pageLinkTemplate /^https?:/ ?;
      object {}* next?;
      string nextLink?;
      object {}* previous?;
      string previousLink?;
      object {}* self?;
      string selfLink?;
      object {}* edit?;
      string editLink?;
      array [
        object {}*;
      ] items?;
    }* data?;
    object {
      integer code?;
      string message?;
      array [
        object {
          string domain?;
          string reason?;
          string message?;
          string location?;
          string locationType?;
          string extendedHelp?;
          string sendReport?;
        }*;
      ] errors?;
    }* error?;
  }*;
  ```
 
- 状态码, HTTP Header信息

    200 (OK) 成功  
    304 (Not modified) 缓存有效  
    401 (Unauthorized) 用户没有进行认证  
    403 (Forbidden) 用户没有访问特定资源的权限  
    404 (Not Found) 请求资源不存在  
    408 (Request timeout) 请求超时  
    415 (Unsupported Media Type) Content-Type需要制定为application/json  
    422 (Unprocessable Entity) 参数不合法  
    429 (Too Many Requests) 请求频率超配, 稍后再试  
    500 (Internal Server Error) 服务器处理请求出错, 需要联系维护人员
 
- API文档说明

    /api/pei 检查信息上传
    ----
    
    Description
    
    | 项目 | 描述 |
    |---|---|
    | Host | http://platform.k-caring.com |
    | Method | POST |
    | Data Type | JSON |
    | API Version | 1.0 |
    | Description | 用于设备信检查信息上传<br> PEI: Physical Examination Information |
    
    HTTP Header
    
    | 项目 | 类型 | 必须 | 描述 |
    |---|---|---|---|
    | token | String | Yes | 32位长的 AccessToken。 可以从管理界面获取<br> 如果在Parameter里指定了token, 这里可以省略 |
    
    HTTP Parameter
    
    | 项目 | 类型 | 必须 | 描述 |
    |---|---|---|---|
    | device id | String | Yes | 设备唯一识别号 |
    | user id | String | Yes | 用户唯一识别号 |
    | custom field | Object | No | 设备生成的检查项目 |
    
    Success
    
    | 项目 | 类型 | 例子 | 描述 |
    |---|---|---|---|
    | apiVersion | String | 1.0 | 版本号 |
    | data | Object | | 结果集 |
    | data.\_id | String | 57510ec89e457d0500ec06f6 | 数据标识 |
    
    ```
    {"apiVersion":"1.0","data":{"_id":"57510ec89e457d0500ec06f6"}}
    ```

    Error

    | 状态 | 描述 |
    |---|---|
    | 200 | OK 成功 |
    | 403 | Forbidden 用户没有访问特定资源的权限 |
    | 408 | Request timeout 请求超时 |
    | 500 | Server Error 服务器处理请求出错, 需要联系维护人员 |
    
    ```
    {"apiVersion":"1.0","error":{"code":403,"message":"token is empty"}}
    ```

    Request Example
    
    ```
    curl -F "type=0" -H "token:[AccessToken]" http://platform.k-caring.com/api/pei
    ```
 
- 其他

    a. 当客户端无法使用特定的HTTP方法时, 可以通过HTTP Request Header进行模拟  
    `X-HTTP-Method-Override=DELETE`  
    b. 支持的HTTP Method有  
    `GET 获取, POST 创建, PUT 更新, PATCH 更新, DELETE 删除`  
    c. 指定Content-Type  
    `Content-Type`需要被制定为`application/json`, 否则服务器会返回415错误
   
**1.2. UDP API**

    TODO:

**2. 被动传输数据: 设备开放数据服务, 由康乐服务器定时发出数据请求**
  
- 接口支持以下两种形式的服务

    HTTP SOAP (WebService)  
    HTTP JSON API

- 认证, 支持以下认证方式

    HTTP Basic认证  
    OAuth2认证  
    Access-Token认证

- 请求频度

    默认情况下, 服务器会每个5分钟发起一次请求

**3. 影像数据传输**

- 影像数据的传输统一使用 DICOM3.0接口标准

----

## 数据内容标准 ##

**1. 数据内容与数据取值范围**

- 数据内容(标准)

    数据内容参考 卫计委`卫生信息数据元目录 2001版` 
  
- 数据取值(标准)

    数据取值参考 卫计委`卫生信息数据元值域代码 2001版`

- 数据内容(扩展)

    当数据元目录内容无法描述上传数据内容是, 可采用扩展字段传输

**2. 设备的识别与被检测人的识别**

    除检测数据之外,  为了识别被检测人及数据的来源, 需要上传数据时, 必须附带以下信息
  
- deviceID 设备识别号

    用于识别数据的来源

- userID 用户识别号

    用于确定被检测对象

**3. 数据内容**

- 需要上传的数据内容
    - 原始数据: 原始数据部分, 采用卫计委`卫生信息数据元目录` 
    - 设备生成的分析报告: 属于扩展信息, 可有设备自行设定

----

## 数据安全标准 ##
**1. 数据通道安全**

    所有的数据传输都使用TLS, 来保证数据的保密性和完整性

**2. 数据加密** 

    数据加密, 采用 AES(Advanced Encryption Standard) 256位加密算法.

- AESKey 消息加解密Key的规定

    TODO

- AES加密模式, CBC模式

    秘钥长度为32个字节（256位）[RFC2315](http://tools.ietf.org/html/rfc2315)

- BASE64采用MIME格式, 规定如下

    TODO
