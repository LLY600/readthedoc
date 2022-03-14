1. Charles 工作原理
	- Charles 的工作原理类似于中间人代理，即任何客户端和服务端的通信都会经过 Charles，于是 Charles 便可以拦截来自客户端和服务端的任何请求，并进行修改
2. 搭建基础代理环境（Web端与App端）
	- Web 端
		- 下载安装：[https://www.charlesproxy.com]()
		- 首先去安装根证书。根证书的位置在 Help--> SSL Proxying --> Install Charles Root Certificate。
		- 证书安装后，点击 Charles 的菜单， 选择 Proxy-->macOS Proxy.
		- 如果想抓包 HTTPS 的请求，则需要通过 Charles 的菜单 Proxy-->SSL Proxying Settings 来进行设置。
		- 在弹出的对话框中，勾选Enable SSL Proxying，并点击 ADD 按钮，再编辑如下。
		- Host 填*，Port 填写 443，这样就可以抓取 https 的请求了。
	- App 端
		- 你需要在 Charles 所在的机器上进行如下配置，找到菜单 Proxy-->Proxy Settings 进行以下设置。
		- 填入代理端口 8888（也可以自定义填其他端口）；
		- 并且勾上Enable transparent HTTP proxying；
		- 勾选 Support HTTP/2。
		- 确定手机端代理地址，还是在 Charles 安装的那台机器上。
		- 点击 Charles 的菜单help–>SSL Proxying–> Install Charles Root Ceriticate On a Mobile Device or Remote Browser
		- 在 App 端，通过浏览器访问网页：chls.pro/ssl，并下载安装证书。
		- 需要注意的是，有些设备特别是 Android 设备无法直接安装下载的证书，需要进入到设置->安全->凭据存储 ->从存储设备安装证书。
3. 接口过滤、拦截和修改（过滤接口请求与动态修改请求数据）
	- 过滤接口请求
		- 因为所有的网络请求都会被抓取，那么信息太多也会造成干扰，所以可以过滤接口的请求。过滤接口的请求方式有 2 种。
		- **直接过滤**
			- 直接过滤很简单，直接在 Charles 的 Filter 这栏中填写你要过滤的关键字即可。
		- **通过菜单 Recording Settings 设置**
			- 首先，通过 Charles 菜单选择 Proxy-->Recording Settings。
			- 接着在弹出的对话框中，选择 Include， 然后点击 Add 按钮，并按照你的需求设置 Filter。注意：如果你什么都不填写，就表示全部符合条件。

	- 动态修改请求数据
		- 当发送给服务端的数据需要进行修改时，最简单的方式就是找到这个请求，然后鼠标右键，并选择 Breakpoints。
		- 这样，当你这请求再次被触发时，它就会被拦截。
		- 此时，点击 EditRequest，你将会看到这个请求的所有参数，同时你可以直接进行更改后点击 Execute 让它执行。
		- 点击 Execute 后，请求就会被发送。这时，你还可以对服务端返回给你的数据进行修改。
		- 点击 Edit Response，选择 JSON Text 标签，即可对请求的各项返回进行更改。

		- 打断点更改网络请求是 Charles 最常用的一个方法，在实践中，当测试版本升级，模拟接口返回错误时就会用到打断点。
		- 但一个接口、一个接口的打断点毕竟太浪费实践，除此之外，还可以自定义 BreakPoint Settings， 方法如下：菜单栏选择 Proxy-->BreakPoint Settings。
		- 然后在 Breaking Setting 中，勾选“Enable BreakPoints”，然后点击 Add 按钮
		- 你可以根据需求，自己定义要抓取的请求地址；并且可以通过勾选 Request、Reponse 的方式来决定是只更改发往服务端的请求数据（勾选 Request 即可）。
4. 远程映射
	- 通过 Breakpoint 的方式进行修改请求数据虽然有效，但是修改请求这个操作需要人工干预，需要花费时间。再者，在测试时有可能接口返回，你已经提前知道了，那么就不需要先请求再更改数据，可以准备好返回数据直接进行模拟。
	- 方式为点击 Tools --> Map Local，然后在弹出的对话框中，勾选“Enable Map Local”，接着设置你要覆盖的请求。
	- 注意：Map From 是你的原始请求。Map To 是你期望的结果，这个结果你可以放在本地文件中，以 Text 或者 Json 的格式保持都可以。当你使用 Map Local 后，当有目标请求发生时，Charles 直接将你提供的文件内容当做返回值返回。
	- 远程映射常常用于客户端或者服务端，对返回的时间有超时判断，或者需要更改太多服务端返回值时使用。 毕竟一个个更改接口返回值需要花费时间，而由于超时时间非常短，等你改完再去执行，接口就直接超时了，这个时候就需要用到远程映射来直接更改。
5. 慢网络模拟
	- 从菜单 Porxy--》Throttle Settings 进入，先通过 Add 添加你要进行网络速度限制的站点，然后勾选“Enable Throttling”，选择“Throttle preset”，这个时候你会看到不同的网络速度情况，可以根据需要选择，即可完成网络速度限制。
6. 微服务分支测试
	- 在采用微服务架构后，我们就需要对不同版本的微服务分支进行测试，那么这就有必要进行分支测试。在当下，分支测试通常都是根据 Header 进行区分。既然如此，我们就可以给指定的请求添加 Header，以 Charles 也可以进行微服务分支测试，方式如下：
	- 菜单 Tools -> Rewrite，然后勾选“Enable Rewrite”“Debug in Error Log”，点击 Add，并在右上的 Location 这个栏目下，点击 Add。
	- 在弹出的对话框中， Host 输入*，点击 OK。
	- 输入* 代表任何地址都将被 Rewrite。
	- 然后继续点击 Add 按钮。
	- 配置 Rewrite 的规则。
	- 其中，Type 的值是个下拉列表， 选择 add Header，where 勾选 Request。
	- 然后在下方 Replace 栏目，输入你想增加的 header 的 key 和 value，保存即可。
7. 简单接口并发测试
	- Charles 还可以做接口并发。针对某个接口，假设我们想测试其基本的性能，可以直接选择那个接口，然后右键选择 Repeat Advanced。
	- 此时，会弹出设置的框让你选择，其中：
	- Iterations：是并发轮次数，进行多少轮次的测试。
	- Concurrency：是并发线程数，每轮测试几个请求同时发。
	- Repeat delay：可设置轮次之间的间隔，以毫秒计算。
	- 点击确定。
	- 你会看到，所有的请求已经发出来了。通过这种方式，压力就产生了，我们可以观察服务端的响应时间来判断起基本的性能是否达标。
8. Charles 应用常见问题
	- 【Question 1 】
	- 问：配置好 Charles 后，浏览网页提示“您的连接不是私密连接”
	- 答：打开 SSL Proxying Setting，在弹出的对话框中，做如下操作。
	- 勾选 Enable SSL Proxying；
	- 点击 ADD 按钮，编辑如下。
	- Host 填*，Port 填写 443，这样就可以抓取 https 的请求了。
	- 【Question 2 】
	- 问：Charles 关闭，电脑端就无法上网了，为什么？
	- 答：查看电脑的网络设置->高级设置->代理，检查并取消选中 HTTP 和 HTTPS 代理选项。
	- 【Question 3 】
	- 问： iOS 上证书安装了，但是无法抓包，为什么？
	- 答： 两个检查。
	- 检查是否信任该证书。 在手机设置->通用->关于本机->证书信任设置中信任。
	- 通用->描述文件与设备管理，选中安装的配置文件，并验证。
	- 【Question 4 】
	- 问：App 端开启代理，为什么 Charles 没有出现“允许”提示？
	- 答： App 端和电脑端必须连接同一个网络。
	- 【Question 5 】
	- 问：某些安卓机型证书下载后无法直接安装。
	- 答：通常见于安卓机，例如小米手机。证书下载后，从“更多设置->系统安全->从存储设备安装”这个路径进行安装。

参考：[https://kaiwu.lagou.com/course/courseInfo.htm?courseId=1073#/detail/pc?id=8324]()