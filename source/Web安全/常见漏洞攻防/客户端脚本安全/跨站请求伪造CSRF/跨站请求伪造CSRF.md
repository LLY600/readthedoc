#### 原理
- 我们几乎每天都会使用到浏览器，我们的信息也会被浏览器“保存”。
- 浏览器如何保存你的身份信息：
	- 当我们访问一个web页面的时候，并不是我们自己去获取页面信息，而是浏览器去获取这些信息，并将它们进行了展示。
	- 这就说明，你允许浏览器代表你去和服务器进行交互。
	- 为了准确代表你的身份，浏览器通常会在Cookie中存储一些必要的身份信息。
	- 所以，我们访问一个网页，只需首次访问的时候登录就可以了。
- 从用户体验上来说，非常方便。但是，黑客正是利用这一点，编写恶意的js脚本，通过“钓鱼”的方式诱导你访问。
- 然后黑客通过js脚本获取你保存在网页的身份信息，执行黑客定义的操作。
- 而这一切，你是无感知的，这就是跨站请求伪造（CSRF）。

#### 攻击过程
- 假如黑客通过抓包找到了银行转账接口，如下：
```
接口地址：http://bank.com/transfer 
HTTP 方法：POST
接口参数：to（目标账户）、amount（金额）
```
- 在转账之前，你肯定进行了一次登录。假设这个接口参数中不再需要其他任何身份认证相关的信息。
- 如此，这个接口满足了 CSRF攻击的基本条件：
	- 使用Cookie进行认证；
	- 参数中不包含其他任何隐私信息。
- 那么黑客可以构造一个如下的漏洞利用空白网页，假设地址为http://hacker.com/exploit.html，生成短网址（如https://ock.cn/8q639），通过邮件、社交平台、即时消息等途经发给受害者（短网址生成：https://ock.cn/）
- exploit.html 利用代码如下：
```
<html>
  <body>
    <form name="nomoney" action="http://bank.com/transfer" method="POST">
      <input type="hidden" name="to" value="hacker" />
      <input type="hidden" name="amount" value="10000" />
    </form>
    <script>
      document.nomoney.submit();
    </script>
  </body>
</html>
```
- 在HTML中，script标签内的 JavaScript 脚本会在打开网页的时候自动执行。
- 因此，一旦用户访问了这个https://ock.cn/8q639页面，它就会自动提交 form 表单，向http://bank.com/transfer这个接口发起一个 POST 请求。
- 其中，to 和 amount 这两个参数，代表着用户向黑客的账号转账 10000 元。
- 只要这个用户之前登录过 bank.com，并且账户余额大于10000元，那么黑客就能够成功地收到这 10000 元的转账了。
- 在这个网页中，input的标签带有“hidden”属性，所以这整个过程对于用户来说都是不可见的。

#### CSRF分类
- 从漏洞利用角度来分类的话，CSRF 可以分为 CSRF 读与 CSRF 写。
	1. CSRF读
		- 通过伪造请求来获取返回的敏感信息，比如用户资料；常见的就是 JSON 劫持（详见下文），以及利用 Flash API 加载页面获取敏感信息。
	1. CSRF写
		- 通过伪造请求去修改网站数据，比如修改密码、发表文章、发送消息等操作。
	
#### CSRF攻击，黑客能做什么
- CSRF 也可以仿冒用户去进行一些功能操作的请求，比如修改密码、转账等等，相当于绕过身份认证，进行未授权的操作

#### CSRF防护
1. 验证 HTTP Referer 字段
	- 根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer（浏览器会自动加上这个字段），它记录了该 HTTP 请求的来源地址。
	- 要防御 CSRF 攻击，网站 A 只需要对于每一个请求验证其 Referer 值，如果是 A 网站的域名，则说明该请求是来自己的请求，是合法的。如果 Referer 是其他网站的话，则有可能是黑客的 CSRF 攻击，拒绝该请求。
	```
	#每个请求执行之前先执行before验证请求头中的referer
	@app.before_request
	def csrf_check_referer():
	    # 获取请求头中的referer，login页面不需要检查因为没有，只需要检查后续的跳转页面请求是否带referer字段
	    if request.path != "/login":
	        referer=request.referrer
	        print(referer)
	        if referer[:22] != "http://127.0.0.1:5000/":
	            return "page not found",404
	```
	- Referer 的值是由浏览器提供的，并不能保证浏览器自身没有安全漏洞。对于某些浏览器，目前已经有一些方法可以篡改 Referer 值，黑客完全可以修改 Referer 值，这样就可以通过验证，从而进行 CSRF 攻击

3. CSRFToken
	- 通过前面的例子，我们知道，CSRF 是通过自动提交表单的形式来发起攻击的；
	- 因此，我们只需要在这个接口中，加入一个黑客无法猜到的参数，就可以有效防止 CSRF了。
	- 因为 CSRF Token 是每次用户正常访问页面时，服务端随机生成返回给浏览器的。
	- 所以，每一次正常的转账接口调用，都会携带不同的 CSRF Token。黑客没有办法进行提前猜测，也就没有办法构造出正确的表单了。
	```
	import os
	from flask import Flask, request, render_template
	
	#初始化application
	from wtforms import ValidationError
	from registerform import MyForm
	
	app = Flask(__name__)
	#设置秘钥
	app.config["SECRET_KEY"] = os.urandom(10)
	
	#每个请求执行之前先执行before验证请求头中的referer
	@app.before_request
	def csrf_check_referer():
	    # 获取请求头中的referer，login页面不需要检查因为没有，只需要检查后续的跳转页面请求是否带referer字段
	    if request.path != "/login":
	        referer=request.referrer
	        print(referer)
	        if referer[:22] != "http://127.0.0.1:5000/":
	            return "page not found",404
	
	@app.route("/")
	def index():
	    return {"user":"lei"}
	
	@app.route('/login',methods=['GET','POST'])
	def login():
	#使用flask-wtf表单验证，会自动加上CSRF攻击的验证
	    form = MyForm()
	    if request.method == "GET":
	        return render_template('index.html', form=form)
	    if form.validate_on_submit():
	        return 'OK'
	    else:
	        raise ValidationError(message=form.errors)
	
	if __name__ == '__main__':
	    app.run(debug=True)
	
	form 表单验证模块：
	from flask_wtf import FlaskForm
	from wtforms import StringField
	from wtforms.validators import DataRequired,Length
	
	class MyForm(FlaskForm):
	验证前端传入的name不能为空，长度2-10；
	    name = StringField('name', validators=[DataRequired("姓名不能为空"),Length(2,10,"名字长度错误")])
	
	html 文件：
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <title>flask</title>
	</head>
	<body>
	<div style="color:red">欢迎~~~</div>
	<form method="POST" action="/login">
	#请求页面时会加上csrf_token，提交表单时会带上这个隐藏的token以和参数一起提交给服务器；
	{{ form.csrf_token }}
	<lable>姓名:</lable>
	    <input type="text" name="name" value="">
	<input type="submit" value="submit">
	</form>
	<a href="http://127.0.0.1:5000">点我</a>
	</body>
	</html>
	```
	- 在客户端向后端请求界面数据的时候，需要在 Form 表单中添加一个隐藏的的字段，值是 csrf_token。
	- 在用户点击提交的时候，会带上这个值向后台发起请求
	- 后端接受到请求，会比较值是否正确，如果没取到或者不正确，代表不是正常的请求，不执行下一步操作。
1. 通过二次验证（支付密码、验证码等）
	- 当你进行各类支付操作的时候，通常会要求你输入支付密码。你可能会觉得奇怪，明明自己已经登录了，为什么还需要输入一个独立的支付密码呢？
	- 这其实和 CSRF Token 的原理一样：这个独立的支付密码是需要用户输入的，只存在于用户的记忆中，因此，也是黑客无法获取到的参数。
	
#### CSRF测试思路
- 抓包记录正常的 HTTP 请求；分析 HTTP 请求参数是否可预测，以及相应的用途；去掉或更改 referer 为第三方站点，然后重放请求；判断是否达到与正常请求的同等效果，若是则可能存在 CSRF 漏洞，反之则不存在。

#### 工具推荐
- 目前没有特别好的自动化 CSRF 检测工具，大多是一些半自动的辅助类工具，比如 BurpSuite 上的 CSRF PoC 生成功能
- 也可以参考：CSRFTester，CSRF Request Builder
	
#### 靶场推荐
1. DVWA

	