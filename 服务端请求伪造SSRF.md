#### 原理
- 我们知道，服务端有代理请求的功能：用户在浏览器中输入一个url(比如图片资源)，服务端会向这个url发起请求，通过访问其他服务器资源来完成正常的页面展示。这个时候，只要黑客在输入中提交一个内网url，就能让服务器发起一个黑客定义的内网请求，从而获取内网数据。
- 服务端作为内网设备，通常具有很高的权限，所以，这个伪造请求往往因为绕过大部分的认证和授权机制，而产生很严重的后果。

#### SSRF攻击，黑客能做什么
1. 内网探测
	- 内外网一般是隔离的，黑客在外网，是无法知道内网有哪些服务器，以及提供的服务。
	- 以百度搜图为例，通过拼接地址，如果是正常的图片地址，百度能请求到数据并返回。
	```
	https://image.baidu.com/search/detail?objurl=https://gimg2.baidu.com/image_search/src=http://p1.meituan.net/shaitu/3a50b3987880270eb76a8beb492d8d0f1577873.jpg
	```
	- 假设服务端逻辑是这样：在请求过程中，百度会判断拼接objurl返回数据的ContentType是否为image/jpeg，可能的返回结果有如下：
		- “是”，展示图片
		- “不是”，则返回格式错误
		- 无响应，则返回找不到图片
	- 构造恶意请求
	```
	https://image.baidu.com/search/detail?objurl=127.0.0.1:3306
	如果返回格式错误，则表示服务可用
	如果返回找不到图片，则表示服务不可用
	根据此返回信息，黑客就能知道服务端本地是否开启了Mysql服务
	重复此步骤，尝试不同的ip和端口号，一点一点探测内网结构
	```
	
2. 文件读取
	- 在uri中，开头的http://和https://表示需要什么协议去进行请求
	- 除此之外，还有很多协议，比如file://就是直接读取本地文件，如file:///etc/passwd，从而知道本地有哪些用户
	- 经过不断尝试，黑客就能把服务器中的文件都给获取出来，比如密钥、源码等敏感信息
#### SSRF防护
1. 白名单限制
	- 最简单、最高效的方式
	- 即对用户提交上来的url进行限制。比如，只允许是同一个域名下的url
	
2. 协议和资源类型限制
	- 对于使用协议，比如只允许http、https协议
	- 对于返回的内容，比如只允许图片格式的内容
	
3. 请求端限制
	- 为其他业务提供服务的接口尽量使用post（违背Restful？？，通常针对上行操作），避免get使用。因为，在ssrf中，发起一个post请求难度远大于get请求，毕竟发起post请求，需要在发起http请求的时候进行配置。
	- 为其他业务提供服务的接口，最好每次进行验证。

#### 工具推荐
1. Burp Collaborator
	- BurpSuite 默认提供 Collaborator Server 用于实现 DNS 解析，在一些无回显的安全测试中，会将解析日志返回给 BurpSuite
	- 以靶场题目为例
		- 打开 Burp Collaborator Client
		- 点击“Copy to clipboard”获取生成的域名
		- 访问后回到 Burp Collaborator client，点击“Poll now”就可以看到 DNS 请求记录，这说明存在SSRF漏洞。
		- 这个检测方法比较通用简单，且无须自己搭建 DNS 服务器，非常方便，强烈推荐！！！
2. SSRFmap
	- 官网地址：https://github.com/swisskyrepo/SSRFmap
	- 利用它可检测与利用 SSRF 漏洞
	- 它也整合了一些常用漏洞可以结合 SSRF 去利用，比如 fastjson、mysql、github 的一些历史漏洞
	- 还有端口扫描、读取文件等利用功能，都是实用的漏洞利用能力
	- 使用上会比 Burp Collaborator 便捷，但检测能力上其实没有 Burp Collaborator 那样准确，SSRFmap 有时会利用失败
	- 推荐使用 Burp Collaborator 用来检测 SSRF 是否存在，再使用 SSRFmap 做一些漏洞利用，比如读取敏感文件等操作
	
#### 靶场推荐
1. Pikachu
	- 皮卡丘漏洞练习平台
	