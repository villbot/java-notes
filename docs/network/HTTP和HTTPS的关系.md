# HTTP与HTTPS的关系
## 什么是HTTP
> 超文本传输协议（Hypertext Transfer Protocol，HTTP）是一个简单的请求-响应协议，它通常运行在 TCP/IP 之上。它指定了客户端可能发送给服务器什么样的消息以及得到什么样的响应。请求和响应消息的头以ASCII形式给出；

* HTTP协议工作于客户端-服务端架构上。浏览器作为HTTP客户端通过URL向HTTP服务端即WEB服务器发送所有请求。
* HTTP协议定义了标准，简单快速可以描述请求类型（GET, POST、PUT 、HEAD、DELETE 、OPTIONS、PATCH、TRACE 和 CONNECT方法）
* HTTP是无连接的，无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间
* HTTP是无状态的，HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。
* HTTP是媒体独立的，这意味着，只要客户端和服务器知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型。

### HTTP请求结构

HTTP请求是由**请求行（request line）、请求头部（header）、空行和请求数据**四个部分组成，下图给出了请求报文的一般格式。
 ![2012072810301161](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_03_19/2012072810301161.png)

### HTTP响应结构

HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。
![httpmessage](https://elgchat-oss.oss-accelerate.aliyuncs.com/elgchat/2021_03_19/httpmessage.jpg)

### HTTP请求方法

HTTP1.0 定义了三种请求方法： GET, POST 和 HEAD方法。

HTTP1.1 新增了六种请求方法：OPTIONS、PUT、PATCH、DELETE、TRACE 和 CONNECT 方法。

| 方法    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| GET     | 请求指定的页面信息，并返回实体主体。                         |
| HEAD    | 类似于GET请求，只不过返回的相应中没有具体内容，用于获取报头  |
| POST    | 向指定资源提交数据进行处理请求，数据包含资源的建立/或已有的资源的修改 |
| PUT     | 从客户端向服务器传送的数据取代指定的文档内容                 |
| DELETE  | 请求服务器删除                                               |
| CONNECT | HTTP/1.1协议中预留给能将连接改为管道方式的代理服务器         |
| OPTIONS | 允许客户端查看服务器的性能                                   |
| TRACE   | 回显服务器收到的请求，主要用于测试和诊断                     |
| PATCH   | 是对PUT方法的补充，用来对已知资源进行局部更新                |

### HTTP响应信息
#### HTTP 响应头信息

| 应答头           | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| Allow            | 服务器支持哪些请求方法（如GET、POST等）。                    |
| Content-Encoding | 文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即request.getHeader("Accept-Encoding")）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面。 |
| Content-Length   | 表示内容长度。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入 ByteArrayOutputStream，完成后查看其大小，然后把该值放入Content-Length头，最后通过byteArrayStream.writeTo(response.getOutputStream()发送内容。 |
| Content-Type     | 表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentType。 |
| Date             | 当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦。 |
| Expires          | 应该在什么时候认为文档已经过期，从而不再缓存它？             |
| Last-Modified    | 文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置。 |
| Location         | 表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。 |
| Refresh          | 表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过setHeader("Refresh", "5; URL=http://host/path ")让浏览器读取指定的页面。  注意这种功能通常是通过设置HTML页面HEAD区的＜META HTTP-EQUIV="Refresh" CONTENT="5;URL=http://host/path "＞实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。   注意Refresh的意义是"N秒之后刷新本页面或访问指定页面"，而不是"每隔N秒刷新本页面或访问指定页面"。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是＜META HTTP-EQUIV="Refresh" ...＞。   注意Refresh头不属于HTTP 1.1正式规范的一部分，而是一个扩展，但Netscape和IE都支持它。 |
| Server           | 服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。 |
| Set-Cookie       | 设置和页面关联的Cookie。Servlet不应使用response.setHeader("Set-Cookie", ...)，而是应使用HttpServletResponse提供的专用方法addCookie。参见下文有关Cookie设置的讨论。 |
| WWW-Authenticate | 客户应该在Authorization头中提供什么类型的授权信息？在包含401（Unauthorized）状态行的应答中这个头是必需的。例如，response.setHeader("WWW-Authenticate", "BASIC realm=＼"executives＼"")。  注意Servlet一般不进行这方面的处理，而是让Web服务器的专门机制来控制受密码保护页面的访问（例如.htaccess）。 |

#### HTTP状态码
HTTP状态码由三个十进制数字组成，第一个十进制数字定义了状态码的类型，后两个数字没有分类的作用。HTTP状态码共分为5种类型：

| 分类 | 分类描述                                       |
| ---- | ---------------------------------------------- |
| 1**  | 信息，服务器收到请求，需要请求者继续执行操作   |
| 2**  | 成功，操作被成功接收并处理                     |
| 3**  | 重定向，需要进一步的操作以完成请求             |
| 4**  | 客户端错误，请求包含语法错误或无法完成请求     |
| 5**  | 服务器错误，服务器在处理请求的过程中发生了错误 |

常用的HTTP状态码：

* 200 - 请求成功
* 301 - 资源（网页等）被永久转移到其它URL
* 404 - 请求的资源（网页等）不存在
* 500 - 内部服务器错误



## 什么是HTTPS

> **HTTPS**（Hypertext Transfer Protocol Secure：超文本传输安全协议）是一种透过计算机网络进行安全通信的传输协议。HTTPS 经由 HTTP 进行通信，但利用 SSL/TLS 来加密数据包。HTTPS 开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。

那么，简单可以理解为HTTPS是在HTTP的基础上增加了SSL/TLS

## HTTPS工作原理

- 1、TCP 三次同步握手(具体参考 [TCP实现.md](TCP实现.md) )
- 2、浏览器将自己所支持的一套加密规则发送给网站
- 3、网站从中选择一组加密算法，将自己的身份信息以证书的形式返回给浏览器（证书中包含了网站地址、加密公钥以及证书的颁发机构等信息）
- 4、浏览器获得网站证书之后要验证证书合法性（证书签发机构是否合法，证书包含的网站地址是否与访问的地址一致）
- 5、如果证书受信，会在浏览器中显示小锁头，否则给我证书不受信提示
- 6、如果证书受信任或用户接受不信任证书，浏览器会生成一串随机数，并用证书中的提供的公钥加密
- 7、使用公钥加密后的随机数密数加密握手信息，发送给网站
- 8、网站接收发来的数据
  - 使用私密解密出密码
  - 使用密码解密握手信息
  - 使用密码再加密一段握手信息，发送给浏览器
- 9、浏览器解密并计算握手信息的Hash、如果遇服务器发来的信息Hash一致，握手结束，SSL 安全加密隧道协商完成
- 10、之后所有通信数据以加密的方式传输，用协商的对称加密算法和密钥加密，保证数据机密性；用协商的hash算法进行数据完整性保护，保证数据不被篡改。

## HTTP与HTTPS之间的区别

* HTTPS协议使用时需要到电子商务认证授权机构(CA)申请SSL证书
*  HTTP默认使用80端口，HTTPS默认使用443端口 
* HTTPS则是具有SSL加密的安全性传输协议，对数据的传输进行加密，效果上相当于HTTP的升级版，HTTPS比HTTP要更耗费服务器资源 
* HTTP的连接是无状态的，不安全的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比HTTP协议安全
* HTTP 页面响应速度比 HTTPS 快，主要是因为 HTTP 使用 TCP 三次握手建立连接，客户端和服务器需要交换 3 个包，而 HTTPS除了 TCP 的三个包，还要加上 ssl 握手需要的 9 个包，所以一共是 12 个包
