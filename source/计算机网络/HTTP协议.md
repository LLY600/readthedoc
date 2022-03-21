HTTP协议
===================================
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