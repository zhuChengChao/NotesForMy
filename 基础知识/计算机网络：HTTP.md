# 计算机网络：HTTP

> 对《图解HTTP》做一个简单的阅读笔记。

## 先验知识

### HTTP和其他协议的关系

通过下图，了解IP协议，TCP协议，DNS服务在使用HTTP协议通信过程中各自发挥的作用：

![HTTP和其他协议的关系](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749548.PNG)

#### 服务器处理流程

接受客户端连接 ------> 接受请求报文 ------> 处理请求 ------> 访问Web资源 ------> 构造应答 ------> 发送应答

### URL和URI

#### URL

Uniform Resource Locator，统一资源定位符，访问web浏览器时需要输入的网页地址就是这个；

**表示了资源的地址，**根据URL是可以找到这个资源的；

例如：https://www.cnblogs.com/zhuchengchao/p/15557495.html

#### URI

Uniform Resource Identity，统一资源标识符，即用字符串标识了某一个资源；

例如：/zhuchengchao/p/15557495.html

#### 联系与区别

**URL为URI的子集**，举个例子：URI为——花，URL为——百合花；

URI用字符串标识某一互联网资源，而URL表示了资源的地址；

URI是抽象的，在详细定位资源时要使用URL；

## HTTP报文格式

### 请求报文格式

![请求报文](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749039.PNG)

### 请求报文的方法

#### GET：获取指定的服务端资源

* 请求参数直接跟在URL后面；
* 请求的URL长度有限制；
* 传输参数明文传输，不安全；
* 一般作为用户获取资源，即查询数据；

#### POST：提交数据到服务器端

* 请求参数在请求体中，即上图中的报文主体；
* 请求的URL长度没有限制；
* 相对安全；
* 一般用于修改数据；

#### 其他方法：

* PUT：传输文件；
* UPDATE：更新指定的服务端资源；
* DELETE：删除指定的服务端资源；
* HEAD：获取报文首部，和GET的区别为，不需要返回报文主体，只要首部即可；
* OPTIONS：查询支持的方法，是否支持GET，PUT，DELETE... ...

### 响应报文格式

![响应报文](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749068.PNG)

### 响应报文的状态码

![响应状态码](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749756.PNG)

#### 2XX：请求成功状态码

| 状态码 | 原因短语        | 说明                                             |
| ------ | --------------- | ------------------------------------------------ |
| 200    | OK              | 从客户端发出的请求在服务器端被正确处理了         |
| 204    | No Content      | 请求已经被成功处理，返回的相应报文中不含主体数据 |
| 206    | Partial Content | 服务器对客户端的范围请求相应成功                 |

#### 3XX：重定向状态码

| 状态码 | 原因短语           | 说明                                                         |
| ------ | ------------------ | ------------------------------------------------------------ |
| 301    | Moved Permanently  | 永久性重定向：为资源的请求分配了新的URI，以后都要使用这一个URI |
| 302    | No Content         | 临时重定向：为资源的请求分配了新的URI，仅限于本次            |
| 303    | See Other          | 与302相同，但明确表明需用GET方式获取资源                     |
| 304    | Not Modified       | 客户端发送附带的条件请求不满足，导致返回的响应内容不包含主体 |
| 307    | Temporary Redirect | 临时重定向：与302相同，但307遵守不将POST变为GET请求。        |

#### 4XX：客户端错误状态码

| 状态码 | 原因短语     | 说明                                 |
| ------ | ------------ | ------------------------------------ |
| 400    | Bad Request  | 请求报文存在语法错误，服务器无法理解 |
| 401    | Unauthorized | 发送的请求需要通过HTTP认证           |
| 403    | Forbidden    | 对资源的访问请求被服务器拒绝了       |
| 404    | Not Found    | 服务器上无法找到请求的资源           |

#### 5XX：服务器错误状态码

| 状态码 | 原因短语              | 说明                                             |
| ------ | --------------------- | ------------------------------------------------ |
| 500    | Internal Server Error | 服务器在执行请求时发生了错误，无法提供资源       |
| 503    | Service Unavailable   | 服务器暂时处于超负荷或正在停机维护，无法处理请求 |

## HTTP报文首部

HTTP协议的请求和响应报文中必定包含着HTTP的首部；

在上方HTTP的请求/响应报文的格式中可以看出，报文首部由如下部分组成：

* 请求报文：请求行，请求首部字段，通用首部字段，实体首部字段；
* 响应报文：状态行，响应首部字段，通用首部字段，实体首部字段。

格式：

* 首部字段名：字段值

### 通用首部字段

请求报文和响应报文都会使用的字段。

| 首部字段名        | 说明                                       | 示例                                                         |
| ----------------- | ------------------------------------------ | ------------------------------------------------------------ |
| Cache-Contorl     | 控制缓存行为                               | Cache-Contorl: private, max-age=0, no-chache                 |
| Connection        | 管理持久连接；控制不再转发给代理的首部字段 | Connection: Keep-Alive<br />Connection: 不再转发的首部字段名 |
| Data              | 创建报文的日期时间                         | Data: Thu, 26 Mar 2020 05:36:55 GMT                          |
| Pragma            | 报文指令                                   | Pragma: no-cache                                             |
| Trailer           | 报文末端的首部一览                         | Trailer: Expires<br />...(报文主体)...<br />Expires: Thu, 26 Mar 2020 05:36:55 GMT |
| Transfer-Encoding | 指定报文主体的传输编码方式                 | Transfer-Encoding: chunked                                   |
| Upgrade           | 升级为其他协议                             | Upgrade: TSL/1.0<br />Connection: Upgrade                    |
| Via               | 代理服务器的相关信息                       | Via: 1.0 gw.hacher.jp(Squid/3.1)<br />Via: 1.0 gw.hacher.jp(Squid/3.1), 1.1 al.example.com(Squid/2.7) |
| Waring            | 错误通知                                   | Waring: 警告码 警告的主机:端口号 “警告内容” (日期时间)<br />Waring: 113 gw.hacker.jp:8080 "Heuristic expiration" |

### 请求首部字段

客户端向服务器发送请求报文时使用的首部，补充了请求的附加内容，客户端信息，响应内容相关优先级等信息。

| 首部字段名          | 说明                                | 示例                                                         |
| ------------------- | ----------------------------------- | ------------------------------------------------------------ |
| Accept              | 用户代理可处理的媒体类型            | Accept：text/html,<br />application/xhtml+xml,<br />application/xml;q=0.9,<br />image/apng,*/*;q=0.8 |
| Accept-Charset      | 优先的字符集                        | Accept-Charset:<br />iso-8859-5.unicode-1-1;q=0.8            |
| Accept-Encoding     | 优先的内容编码                      | Accept-Encoding: gzip, deflate, br                           |
| Accept-Language     | 优先的语言(自然语言)                | Accept-Language: <br />zh-CN,zh;q=0.9,en;q=0.8               |
| Authorization       | Web认证信息                         | Authorization: Basic  avajoPdaD==                            |
| Expect              | 期待服务器的特定行为                | Expect: 100-continue                                         |
| From                | 用户的电子邮箱地址                  | From: xxx@xxx.xxx                                            |
| Host                | 请求资源所在的服务器                | Host: www.baidu.com                                          |
| if-Match            | 比较实体标记(ETag)                  | if-Match: "123456"                                           |
| if-Modified-Since   | 比较资源的更新时间                  | if-Modified-Since: 时间                                      |
| if-None-Match       | 与if-Match相反                      | if-None-Match: "123456"                                      |
| if-Range            | 资源未更新时发送实体Byte的范围请求  | if-Range: "123456"<br />Range: bytes=5001-10000              |
| if-Unmodified-Since | 与if-Modified-Since相反             | if-Unmodified-Since: 时间                                    |
| Max-Forwards        | 最大传输逐跳数，为0时，不再进行转发 | Max-Forwards：10                                             |
| Proxy-Authorization | 代理服务器要求客户端的认证信息      | Proxy-Authorization: <br />Basic  avajoPdaD==                |
| Range               | 实体的字节范围请求                  | Range: bytes=5001-10000                                      |
| Referer             | 对请求中URI的原始获取方             | Referer: https://www.baidu.com/                              |
| TE                  | 传输编码的优先级                    | TE: gzip, deflate; q=0.5                                     |
| User-Agent          | HTTP客户端程序的信息                | User-Agent: <br />Mozilla/5.0 (Windows NT 10.0; WOW64) <br />AppleWebKit/537.36 (KHTML, like Gecko) <br />Chrome/78.0.3904.108 Safari/537.36 |

### 响应首部字段

从服务器向客户端返回响应报文时使用的首部，补充了响应的附加内容，也会要求客户端附加额外的内容信息。

| 首部字段名         | 说明                         | 示例                                                    |
| ------------------ | ---------------------------- | ------------------------------------------------------- |
| Accept-Ranges      | 是否接受字节范围请求         | Accept-Ranges: bytes                                    |
| Age                | 推算资源创建经过时间         | Age: 53                                                 |
| ETag               | 资源的匹配信息               | ETag: "5e8d4b59-c5c"                                    |
| Location           | 令客户端重定向到指定URI      | Location: <br />http://.......                          |
| Proxy-Authenticate | 代理服务器对客户端的认证信息 | Proxy-Authenticate:<br />Basic realm="Usagidesign Auth" |
| Retry-After        | 对再次发起请求的时机要求     | Retry-After: 120                                        |
| Server             | HTTP服务器的安装信息         | server:  sffe                                           |
| Vary               | 代理服务器缓存的管理信息     | Vary : Accept-Encoding                                  |
| WWW-Authenticate   | 服务器对客户端的认证信息     | WWW-Authenticate: Basic<br />realm="Usagidesign Auth"   |

### 实体首部字段

针对请求报文和响应报文的实体部分使用的首部，补充了资源内容更新时间等于实体有关的信息。

| 首部字段名       | 说明                      | 示例                                              |
| ---------------- | ------------------------- | ------------------------------------------------- |
| Allow            | 资源可支持的HTTP方法      | Allow: GET, HEAD                                  |
| Content-Encoding | 实体主体适用的编码方式    | Content-Encoding: gizp                            |
| Content-Language | 实体主体的自然语言        | Content-Language: zh-CN                           |
| Content-Length   | 实体主体的大小(单位:Byte) | Content-Length: 1500                              |
| Content-Location | 替代对应资源的URI         | Content-Location:<br />http:// ../../..           |
| Content-MD5      | 实体主体的报文摘要        | Content-MD5: <br />JBK...ddkjfkl==                |
| Content-Range    | 实体主体的位置范围        | Content-Range:<br />bytes 5001-10000/10000        |
| Content-Type     | 实体主体的媒体类型        | Content-Type: image/jpeg                          |
| Expires          | 实体主体过去的日期时间    | Expires: <br />Sat, 23 May 2020 01:41:38 GMT      |
| Last-Modified    | 资源的最后修改日期时间    | Last-Modified:<br />Sat, 16 May 2020 01:41:38 GMT |

### 非HTTP1.1首部字段——Cookie

#### 概述

由于HTTP是**无状态协议**，即无法记住之前与其通讯的客户端；

Cookie技术通过在请求和响应报文中写入Cookie信息来控制客户端的状态；

![Cookie](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749355.PNG)

| 首部字段   | 说明                           | 首部类型     |
| ---------- | ------------------------------ | ------------ |
| Set-Cookie | 开始状态管理所使用的Cookie信息 | 响应首部字段 |
| Cookie     | 服务器接收到的Cookie信息       | 请求首部字段 |

#### **Set-Cookie**

```
Set-Cookie: status=enable; expire= 时间; path=/; domain=.***.com
```

字段说明：

| 属性        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| NAME=VALUE  | 赋予Cookie的名称和其值（必须）                               |
| expire=DATE | Cookie的有效期<br />（若无，则默认为浏览器关闭为止）         |
| path=PATH   | 将服务器上的文件目录作为Cookie的使用对象<br />（若无，则默认为文档所在的文件目录） |
| domain=域名 | 作为Cookie使用对象的域名<br />（若无，则默认为创建Cookie的服务器域名） |
| Secure      | 仅在HTTPS安全通信时才会发送Cookie                            |
| HttpOnly    | 加以限制，使Cookie不能被JavaScript脚本访问                   |

#### **Cookie**

```
Cookie: BAIDUID=***; BDORZ=***; COOKIE_SESSION=***; BD_HOME=1; H_PS_PSSID=***; sug=3; sugstore=0; ORIGIN=2; bdime=0Cookie： status=enable
```

告知服务器，当客户端想要获得HTTP状态管理支持时，就会在请求中包含从服务器接收到的Cookie

#### Session

服务器端会话技术，访问者从访问某个网站开始到离开为止；

在一次会话的多次请求间共享数据，将数据保存在服务器端的对象中；

由于HTTP是无协议的，因此会使用Cookie来管理Session，因此Session的实现是依赖于Cookie的；

Session与Cookie的区别：

* session存储数据在服务器端，Cookie在客户端；
* session没有数据大小限制，Cookie有；
* session数据安全，Cookie相对于不安全。

![Session](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749745.PNG)

## HTTPS

### 概述

HTTPS（Secure）是安全的HTTP协议，不作为新协议，只是在HTTP通信接口部分加上了SSL（Secure Socket Layer，安全套接层）和TSL（Transport Layer Security，安全层传输协议），HTTPS是身披SSL外壳的HTTP，如下：

![图示HTTPS](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749168.png)

即：HTTPS=HTTP+加密+完整性保护

**格式：**https://<主机>:<端口>/<路径>  

> 端口由原先的80端口，变为443端口

### HTTP存在的问题

**问题一：**HTTP是明文传输的，内容会被窃听；

**问题二：**不验证通信方的身份，可能遭遇伪装

**问题三：**无法证明报文的完整性，可能已遭篡改

### HTTPS解决问题

#### 采用加密方式

**通信加密：**

通过SSL或TSL的组合使用，加密HTTP的通信内容；

用SSL建立安全通信线路后，就可以在这条线路上进行HTTP通信；

![通信加密](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749171.PNG)

**内容加密：**

HTTP采用混合加密机制：综合使用了非对称和对称加密的方式，流程概括如下：

![加密方式](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749753.PNG)

根据了随机数1,2,3和相同的加密算法生成了共享秘钥；

双方使用对称秘钥进行加密通信。

> * 内容加密是指，将传输的报文主体进行加密，但是对于报文首部信息是不会加密的；
>
> * **共享加密=对称加密：**加密和解密都是通过同一个秘钥；
>
> * **公钥加密=非对称加密：**
>   * 公钥：可以随意发布在网上，可以解开有私钥加密的数据；
>   * 私钥：需要保密，不能被人知道，可以解开由公钥加密的数据。

#### 数字证书

通过数字证书来确定通信方，数字证书是可信任组织颁发给特定对象的认证；

通过证书，可以证明通信方就是意料之中的服务器。

![数字证书](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749489.PNG)



> 数字证书组成：证书格式、版本号；证书序列号；签名算法；有效期；对象名称；对象公开秘钥；...

### HTTPS通信步骤

![通信步骤](https://xycnotes.oss-cn-hangzhou.aliyuncs.com/img/202206251749324.PNG)

## 参考

《图解TCP/IP》 [日]竹下隆史等

《图解HTTP》[日]上野宣