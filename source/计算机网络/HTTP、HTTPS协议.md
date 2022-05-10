HTTP协议
===================================
#### 简介
- HTTP是Hypertext Transfer Protocal 的缩写，中文全称是超文本传输协议。
	- 超文本是指包含但不限于文本外的图片、音频、视频等多媒体资源。
	- 协议是通信双方约定好的数据传输格式以及通信规则。
- HTTP是TCP/IP协议簇的最高层--应用层协议。
- 浏览器和服务器在使用HTTP协议相互传递超文本数据时，将数据放入报文体内，同时填充首部(请求头或响应头)构成完整HTTP报文并交到下层传输层，之后每一层加上相应的首部（控制部分）便一层层的下发，最终由物理层将二进制数据以电信号的形式发送出去。HTTP报文结构如下
- backlog
#### HTTP发展历程
- ![](Snipaste_2022-05-09_11-09-54.png)
- 由HTTP的发展历程来看，最开始版本的HTTP(HTTP1.0)在每次建立TCP连接后只能发起一次HTTP请求，请求完毕就释放TCP连接。我们都知道TCP连接的建立需要经过三次握手的过程，而每次发送HTTP请求都需要重新建立TCP连接，毫无疑问是很低效的。所以HTTP1.1改善了这一点，使用长连接的机制，也就是“一次TCP连接，N次HTTP请求”。
- HTTP协议的长连接和短连接，实质上是 TCP 协议的长连接和短连接。
- 在使用长连接的情况下，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，客户端再次访问这个服务器时，会继续使用这一条已经建立的连接。Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。实现长连接需要客户端和服务端都支持长连接。
- （HTTP1.0若要开启长连接，需要加上Connection: keep-alive请求头）
#### HTTP安全性问题
1. 数据保密性问题
	- 因为HTTP无状态，而且又是明文传输，所有数据内容都在网络中裸奔，包用户括身份信息、支付账号与密码。这些敏感信息极易泄露造成安全隐患。
1. 数据完整性问题
	- HTTP数据包在到达目的主机前会经过很多转发设备，每一个设备节点都可能会篡改或调包信息，无法验证数据的完整性。
1. 身份校验问题
	- 有可能遭受中间人攻击，我们无法验证通信的另一方就是我们的目标对象。
- 因此，为了保证数据传输的安全性，必须要对HTTP数据进行加密。
#### 加密方式
1. 对称加密
	- 对称加密也称为密钥加密或单向加密，就是使用同一套密钥来进行加密和解密。密钥可以理解为加密算法。
	- 广泛使用的对称加密
	- ![](Snipaste_2022-05-09_11-21-50.png)
	- 优点：算法公开、简单，加密解密容易，加密速度快，效率高。
	- 缺点：相对来说不算特别安全，只有一把钥匙，密文如果被拦截，且密钥也被劫持，那么，信息很容易被破译。
	- 适用场景：加解密速度快、效率高，因此适用于大量数据的加密场景。由于如何传输密钥是较为头痛的问题，因此适用于无需进行密钥交换的场景，如内部系统，事先就可以直接确定密钥。
	- base64编码也属于对称加密哦
2. 非对称加密
	- 非对称加密使用一对密钥(公钥和私钥)进行加密和解密。非对称加密可以在不直接传递密钥的情况下，完成解密，具体步骤如下
		- 乙方生成两把密钥（公钥和私钥）。公钥是公开的，任何人都可以获得，私钥则是保密的。
		- 甲方获取乙方的公钥，然后用它对信息加密。
		- 乙方得到加密后的信息，用私钥解密。
	- 优点：强度高、安全性强于对称加密算法、无需传递私钥导致没有密钥泄露风险
	- 缺点：计算量大、速度慢
	- 适用场景：适用于需要密钥交换的场景，如互联网应用，无法事先约定密钥。可以与对称加密算法结合：
		- 利用非对称加密算法安全性较好的特点来传递对称加密算法的密钥。
		- 利用对称加密算法加解密速度快的特点，进行数据内容比较大的加密场景的加密。如HTTPS。
	- 如何选择
		- 选择对称加密：HTTP请求方使用对称算法加密数据，那么为了接收方能够解密，发送方还需要把密钥一同传递到接收方。在传递密钥的过程中还是可能遭到嗅探攻击，攻击者窃取密钥后依然可以解密从而得到发送的数据，所以这种方案不可行
		- 选择非对称加密：接收方保留私钥，把公钥传递给发送方。发送方用公钥来加密数据。接收方使用私钥解密数据。攻击者虽然不能直接获取这些数据（因为没有私钥），但是可以通过拦截传递的公钥，然后把自己的公钥传给发送方，再用自己的私钥对发送方发送数据进行解密。整个过程通信双方都不知道中间人的存在，但是中间人能够获得完整的数据信息
		- ![](Snipaste_2022-05-09_13-29-48.png)
3. 数字摘要
	- CA就是 Certificate Authority，颁发数字证书的机构。作为受信任的第三方，CA承担公钥体系中公钥的合法性检验的责任。证书就是源服务器向可信任的第三方机构申请的数据文件。这个证书除了表明这个域名是属于谁的，颁发日期等，还包括了第三方证书的私钥。服务器将公钥放在数字证书中，只要证书是可信的，公钥就是可信的

#### HTTP 协议优缺点
- **优点**
1. 灵活可扩展
	- 一个是语法上只规定了基本格式，空格分隔单词，换行分隔 字段等。另外一个就是传输形式上不仅可以传输文本，还可以传输图片，视频 等任意数据。
1. 请求-应答模式
	- 通常而言，就是一方发送消息，另外一方要接受消息，或 者是做出相应等。
1. 可靠传输
	- HTTP 是基于 TCP/IP，因此把这一特性继承了下来
1. 无状态
	- 这个分场景回答即可。 
- **缺点**
1. 无状态
	- 有时候，需要保存信息，比如像购物系统，需要保留下顾客信息 等等，另外一方面，有时候，无状态也会减少网络开销，比如类似直播行业这 样子等，这个还是分场景来说。 
1. 明文传输
	- 即协议里的报文(主要指的是头部)不使用二进制数据，而是文本 形式。这让 HTTP 的报文信息暴露给了外界，给攻击者带来了便利。 
1. 队头阻塞
	- 当 http 开启长连接时，共用一个 TCP 连接，当某个请求时间过 长时，其他的请求只能处于阻塞状态，这就是队头阻塞问题
#### HTTP请求头解释与示例
1. Accept 	
	- 指定客户端能够接收的内容类型 	
	- Accept: text/plain, text/html
2. Accept-Charset 	
	- 浏览器可以接受的字符编码集。 	
	- Accept-Charset: iso-8859-5
5. Accept-Encoding 	
	- 指定浏览器可以支持的web服务器返回内容压缩编码类型。 	
	- Accept-Encoding: compress, gzip
8. Accept-Language 	
	- 浏览器可接受的语言 	
	- Accept-Language:   en,zh
11. Accept-Ranges 	
	- 可以请求网页实体的一个或者多个子范围字段 	
	- Accept-Ranges: bytes
14. Authorization 	
	- HTTP授权的授权证书 	
	- Authorization:   Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
17. Cache-Control 	
	- 指定请求和响应遵循的缓存机制 	
	- Cache-Control: no-cache
20. Connection 	
	- 表示是否需要持久连接。（HTTP 1.1默认进行持久连接） 	
	- Connection:   close
23. Cookie 	
	- HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。 	
	- Cookie: $Version=1; Skin=new;
26. Content-Length 	
	- 请求的内容长度 	
	- Content-Length:   348
29. Content-Type 	
	- 请求的与实体对应的MIME信息 	
	- Content-Type:   application/x-www-form-urlencoded
32. Date 	
	- 请求发送的日期和时间 	
	- Date:   Tue, 15 Nov 2010 08:12:31 GMT
35. Expect 	
	- 请求的特定的服务器行为 	
	- Expect: 100-continue
38. From 	
	- 发出请求的用户的Email 	
	- From:   user@email.com
41. Host 	
	- 指定请求的服务器的域名和端口号 	
	- Host: www.nudgetech.com
44. If-Match 	
	- 只有请求内容与实体相匹配才有效 	
	- If-Match:   “737060cd8c284d8af7ad3082f209582d”
47. If-Modified-Since 	
	- 如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码 	
	- If-Modified-Since: Sat, 29 Oct   2010 19:43:31 GMT
50. If-None-Match 	
	- 如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变 	
	- If-None-Match:   “737060cd8c284d8af7ad3082f209582d”
53. If-Range 	
	- 如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag 	
	- If-Range: “737060cd8c284d8af7ad3082f209582d”
56. If-Unmodified-Since 	
	- 只在实体在指定时间之后未被修改才请求成功 	
	- If-Unmodified-Since:   Sat, 29 Oct 2010 19:43:31 GMT
59. Max-Forwards 	
	- 限制信息通过代理和网关传送的时间 	
	- Max-Forwards: 10
62. Pragma 	
	- 用来包含实现特定的指令 	
	- Pragma:   no-cache
65. Proxy-Authorization 	
	- 连接到代理的授权证书 	
	- Proxy-Authorization: Basic   QWxhZGRpbjpvcGVuIHNlc2FtZQ==
68. Range 	
	- 只请求实体的一部分，指定范围 	
	- Range:   bytes=500-999
71. Referer 	
	- 先前网页的地址，当前请求网页紧随其后,即来路 	
	- Referer:   http://www.nudgetech.com/index.html
74. TE 	
	- 客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息 	
	- TE:   trailers,deflate;q=0.5
77. Upgrade 	
	- 向服务器指定某种传输协议以便服务器进行转换（如果支持） 	
	- Upgrade: HTTP/2.0, SHTTP/1.3,   IRC/6.9, RTA/x11
80. User-Agent 	
	- User-Agent的内容包含发出请求的用户信息 	
	- User-Agent:   Mozilla/5.0 (Linux; X11)
83. Via 	
	- 通知中间网关或代理服务器地址，通信协议 	
	- Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)
86. Warning 	
	- 关于消息实体的警告信息 	
	- Warn:   199 Miscellaneous warning
#### HTTP响应头解释与示例
1. Accept-Ranges 	
	- 表明服务器是否支持指定范围请求及哪种类型的分段请求 	
	- Accept-Ranges: bytes
4. Age 	
	- 从原始服务器到代理缓存形成的估算时间（以秒计，非负） 	
	- Age:   12
7. Allow 	
	- 对某网络资源的有效的请求行为，不允许则返回405 	
	- Allow: GET, HEAD
10. Cache-Control 	
	- 告诉所有的缓存机制是否可以缓存及哪种类 	
	- Cache-Control:   no-cache
13. Content-Encoding 	
	- web服务器支持的返回内容压缩编码类型 	
	- Content-Encoding: gzip
16. Content-Language 	
	- 响应体的语言 	
	- Content-Language:   en,zh
19. Content-Length 	
	- 响应体的长度 	
	- Content-Length: 348
22. Content-Location 	
	- 请求资源可替代的备用的另一地址 
	- Content-Location:   /index.htm
25. Content-MD5 	
	- 返回资源的MD5校验值 	
	- Content-MD5:   Q2hlY2sgSW50ZWdyaXR5IQ==
28. Content-Range 	
	- 在整个返回体中本部分的字节位置 	
	- Content-Range:   bytes 21010-47021/47022
31. Content-Type 	
	- 返回内容的MIME类型 	
	- Content-Type: text/html;   charset=utf-8
34. Date 	
	- 原始服务器消息发出的时间 	
	- Date:   Tue, 15 Nov 2010 08:12:31 GMT
37. ETag 	
	- 请求变量的实体标签的当前值 	
	- ETag:   “737060cd8c284d8af7ad3082f209582d”
40. Expires 	
	- 响应过期的日期和时间 	
	- Expires:   Thu, 01 Dec 2010 16:00:00 GMT
43. Last-Modified 	
	- 请求资源的最后修改时间 	
	- Last-Modified: Tue, 15 Nov 2010   12:45:26 GMT
46. Location 	
	- 用来重定向接收方到非请求URL的位置来完成请求或标识新的资源 	
	- Location:   http://www.nugetech.com
49. Pragma 	
	- 包括实现特定的指令，它可应用到响应链上的任何接收方 	
	- Pragma: no-cache
52. Proxy-Authenticate 	
	- 它指出认证方案和可应用到代理的该URL上的参数 	
	- Proxy-Authenticate:   Basic
55. refresh 	
	- 应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持） 	
	- Refresh: 5;   url=http://www.nugetech.com
58. Retry-After 	
	- 如果实体暂时不可取，通知客户端在指定时间之后再次尝试 	
	- Retry-After:   120
61. Server 	
	- web服务器软件名称 	
	- Server: Apache/1.3.27 (Unix)   (Red-Hat/Linux)
64. Set-Cookie 	
	- 设置Http Cookie 	
	- Set-Cookie:   UserID=JohnDoe; Max-Age=3600; Version=1
67. Trailer 	
	- 指出头域在分块传输编码的尾部存在 	
	- Trailer: Max-Forwards
70. Transfer-Encoding 	
	- 文件传输编码 	
	- Transfer-Encoding:chunked
73. Vary 	
	- 告诉下游代理是使用缓存响应还是从原始服务器请求 	
	- Vary: *
75. Via 	
	- 告知代理客户端响应是通过哪里发送的 	
	- Via:   1.0 fred, 1.1 nowhere.com (Apache/1.1)
78. Warning 	
	- 警告实体可能存在的问题 	
	- Warning: 199 Miscellaneous   warning
81. WWW-Authenticate 	
	- 表明客户端请求实体应该使用的授权方案 	
	- WWW-Authenticate:   Basic
#### HTTP状态码解释与示例
1. 消息
	- 100 	Continue：继续。客户端应继续其请求
	- 101 	Switching   Protocols：切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议
	- 102 	Processing：由WebDAV（RFC 2518）扩展的状态码，代表处理将被继续执行
1. 成功
	- 200 	Ok：请求已成功，请求所希望的响应头或数据体将随此响应返回。出现此状态码是表示正常状态
	- 201 	Created：请求已经被实现，而且有一个新的资源已经依据请求的需要而建立，且其 URI 已经随Location 头信息返回。假如需要的资源无法及时建立的话，应当返回 '202 Accepted'
	- 202 	Accepted：已接受。已经接受请求，但未处理完成
	- 203 	Non-Authoritative   Information：非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本
	- 204 	No Content：无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档
	- 205 	Reset   Content：重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域
	- 206 	Partial Content：部分内容。服务器成功处理了部分GET请求
	- 207 	Multi-Status：由WebDAV(RFC   2518)扩展的状态码，代表之后的消息体将是一个XML消息，并且可能依照之前子请求数量的不同，包含一系列独立的响应代码
1. 重定向
	- 300 	Multiple Choices：多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择
	- 301 	Moved   Permanently：永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替
	- 302 	Move temporarily：临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI
	- 303 	See   Other：查看其它地址。与301类似。使用GET和POST请求查看
	- 304 	Not Modified：未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源
	- 305 	Use   Proxy：使用代理。所请求的资源必须通过代理访问
	- 306 	Switch Proxy：在最新版的规范中，306状态码已经不再被使用
	- 307 	Temporary   Redirect：临时重定向。与302类似。使用GET请求重定向
1. 请求错误
	- 400 	Bad Request：1.语义有误，当前请求无法被服务器理解。除非进行修改，否则客户端不应该重复提交这个请求2.请求参数有误
	- 401 	Unauthorized：请求要求用户的身份认证
	- 402 	Payment Required：该状态码是为了将来可能的需求而预留的
	- 403 	Forbidden：服务器理解请求客户端的请求，但是拒绝执行此请求
	- 404 	Not Found：服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面
	- 405 	Method   Not Allowed：客户端请求中的方法被禁止
	- 406 	Not Acceptable：服务器无法根据客户端请求的内容特性完成请求
	- 407 	Proxy   Authentication Required：请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权
	- 408 	Request Timeout：请求超时。客户端没有在服务器预备等待的时间内完成一个请求的发送。客户端可以随时再次提交这一请求而无需进行任何更改
	- 409 	Conflict：服务器完成客户端的PUT请求是可能返回此代码，服务器处理请求时发生了冲突
	- 410 	Gone：客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置
	- 411 	Length   Required：服务器无法处理客户端发送的不带Content-Length的请求信息
	- 412 	Precondition Failed：客户端请求信息的先决条件错误
	- 413 	Request   Entity Too Large：由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息
	- 414 	Request-URI Too Long：请求的URI过长（URI通常为网址），服务器无法处理
	- 415 	Unsupported   Media Type：对于当前请求的方法和所请求的资源，请求中提交的实体并不是服务器中所支持的格式，因此请求被拒绝
	- 416 	Requested Range Not Satisfiable：客户端请求的范围无效
	- 417 	Expectation   Failed：在请求头 Expect 中指定的预期内容无法被服务器满足，或者这个服务器是一个代理服务器，它有明显的证据证明在当前路由的下一个节点上，Expect 的内容无法被满足
1. 服务器错误
	- 500 	Internal Server Error：服务器遇到了一个未曾预料的状况，导致了它无法完成对请求的处理。一般来说，这个问题都会在服务器端的源代码出现错误时出现
	- 501 	Not   Implemented：服务器不支持当前请求所需要的某个功能。当服务器无法识别请求的方法，并且无法支持其对任何资源的请求
	- 502 	Bad Gateway：作为网关或者代理工作的服务器尝试执行请求时，从上游服务器接收到无效的响应
	- 503 	Service   Unavailable：由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中
	- 504 	Gateway Timeout：作为网关或者代理工作的服务器尝试执行请求时，未能及时从上游服务器（URI标识出的服务器，例如HTTP、FTP、LDAP）或者辅助服务器（例如DNS）收到响应
	- 505 	HTTP   Version Not Supported：服务器不支持，或者拒绝支持在请求中使用的 HTTP 版本。这暗示着服务器不能或不愿使用与客户端相同的版本。响应中应当包含一个描述了为何版本不被支持以及服务器支持哪些协议的实体
#### HTTPS
- HTTPS 并不是一项新的应用层协议，只是 HTTP 通信接口部分由 SSL 和 TLS 替代而已。HTTP 会先直接和 TCP 进行通信。而HTTPS 会演变为先和 SSL 进行通信，然后再由 SSL 和 TCP 进行通信。SSL 是一个独立的协议，不只有 HTTP 可以使用，其他应用层协议也可以使用，比如FTP、SMTP都可以使用SSL来加密
- ![](Snipaste_2022-05-09_13-39-21.png)
- HTTPS请求全流程：
	- 用户在浏览器发起HTTPS请求，默认使用服务端的443端口进行连接；
	- HTTPS需要使用一套CA 数字证书，证书内会附带一个服务器的公钥Pub，而与之对应的私钥Private保留在服务端不公开；
	- 服务端收到请求，返回配置好的包含公钥Pub的证书给客户端；
	- 客户端收到证书，校验合法性，主要包括是否在有效期内、证书的域名与请求的域名是否匹配，上一级证书是否有效（递归判断，直到判断到系统内置或浏览器配置好的根证书），如果不通过，则显示HTTPS警告信息，如果通过则继续；
	- 客户端生成一个用于对称加密的随机Key，并用证书内的公钥Pub进行加密，发送给服务端；
	- 服务端收到随机Key的密文，使用与公钥Pub配对的私钥Private进行解密，得到客户端真正想发送的随机Key；
	- 服务端使用客户端发送过来的随机Key对要传输的HTTP数据进行对称加密，将密文返回客户端；
	- 客户端使用随机Key对称解密密文，得到HTTP数据明文；
	- 后续HTTPS请求使用之前交换好的随机Key进行对称加解密。
- HTTPS确实解决了HTTP的三个安全性问题
	- 保密性：结合非对称加密和对称加密实现保密性。用非对称加密方式加密对称加密的秘钥，再用对称加密方式加密数据
	- 完整性：通过第三方CA的数字签名解决完整性问题
	- 身份校验：通过第三方CA的数字证书验证服务器的身份
- HTTPS的优缺点
- ![](Snipaste_2022-05-09_13-42-11.png)
#### HTTP 与 HTTPS 区别
1. HTTP 是明文传输协议，HTTPS 协议是由 SSL+HTTP 协议构建的可进行加密传 输、身份认证的网络协议，比 HTTP 协议安全。 
2. HTTPS 比HTTP 更加安全，对搜索引擎更友好，利于SEO,谷歌、百度优先索引HTTPS 网页。 
3. HTTPS 标准端口 443，HTTP 标准端口 80。 
4. HTTPS 需要用到 SSL 证书，而 HTTP 不用。 
- 记住下面2点HTTPS主要作用：
1. 对数据进行加密，并建立一个信息安全通道，来保证传输过程中的数据安全; 
2. 对网站服务器进行真实身份认证
#### GET 和 POST 的区别
1. 从缓存角度看，GET 请求后浏览器会主动缓存，POST 默认情况下不能。
2. 从参数角度来看，GET 请求一般放在URL中，因此不安全，POST请求放在请求体中，相对而言较为安全，但是在抓包的情况下都是一样的。 
3. 从编码角度看，GET 请求只能进行URL编码，只能接受ASCII码，而POST支持更多的编码类型且不对数据类型限制。
4. GET 请求幂等，POST 请求不幂等，幂等指发送 M 和 N 次请求（两者不相同且 都大于 1），服务器上资源的状态一致。 
5. GET 请求会一次性发送请求报文，POST 请求通常分为两个 TCP 数据包，首先发 header 部分，如果服务器响应 100(continue)， 然后发 body 部分。 
6. 从应用场景角度来看，Get多用于无副作用，幂等的场景，例如搜索关键字。Post多用于副作用，不幂等的场景，例如注册 。

#### 参考
- [https://mp.weixin.qq.com/s/7blDsYThToxkpsBJ_OF0ig](https://mp.weixin.qq.com/s/7blDsYThToxkpsBJ_OF0ig)