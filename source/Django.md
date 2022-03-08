#### 类视图与中间件
1. 类视图
- 引入
	- 以函数的方式定义的视图称为函数视图，函数视图便于理解；
	- 但是遇到一个视图对应的路径提供了多种不同HTTP请求方式的支持时，便需要在一个函数中编写不同的业务逻辑，代码可读性与复用性都不佳。
	```
	 def register(request):
	    """处理注册"""
	    # 获取请求方法，判断是GET/POST请求
	    if request.method == 'GET':
	        # 处理GET请求，返回注册页面
	        return render(request, 'register.html')
	    else:
	        # 处理POST请求，实现注册逻辑
	        return HttpResponse('这里实现注册逻辑')
	```
	- 使用类视图可以将视图对应的不同请求方式以类中的不同方法来区别定义
	```
	from django.views.generic import View
	class RegisterView(View):
	    """类视图：处理注册"""
	    def get(self, request):
	        """处理GET请求，返回注册页面"""
	        return render(request, 'register.html')
	    def post(self, request):
	        """处理POST请求，实现注册逻辑"""
	        return HttpResponse('这里实现注册逻辑')
	```
	- 好处：
		- 代码可读性好
		- 类视图相对于函数视图有更高的复用性， 如果其他地方需要用到某个类视图的某个特定逻辑，直接继承该类视图即可
- 使用
	- 定义类视图需要继承自Django提供的父类View；
	- 可使用from django.views.generic import View或者from django.views.generic.base import View 导入，如上所示；
	- 配置路由时，使用类视图的as_view()方法来添加。
	```
	urlpatterns = [
	    # 视图函数：注册
	    # url(r'^register/$', views.register, name='register'),
	    # 类视图：注册
	    url(r'^register/$', views.RegisterView.as_view(), name='register'),
	]
	```
- 原理
	```
	 @classonlymethod
	    def as_view(cls, **initkwargs):
	        """
	        Main entry point for a request-response process.
	        """
	        ...省略代码...
	        def view(request, *args, **kwargs):
	            self = cls(**initkwargs)
	            if hasattr(self, 'get') and not hasattr(self, 'head'):
	                self.head = self.get
	            self.request = request
	            self.args = args
	            self.kwargs = kwargs
	            # 调用dispatch方法，按照不同请求方式调用不同请求方法
	            return self.dispatch(request, *args, **kwargs)
	        ...省略代码...
	        # 返回真正的函数视图
	        return view
	    def dispatch(self, request, *args, **kwargs):
	        # Try to dispatch to the right method; if a method doesn't exist,
	        # defer to the error handler. Also defer to the error handler if the
	        # request method isn't on the approved list.
	        if request.method.lower() in self.http_method_names:
	            handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
	        else:
	            handler = self.http_method_not_allowed
	        return handler(request, *args, **kwargs)
	```
	- 调用流程 as_view-->view-->dispatch
	- getattr('对象','字符串')
- 使用装饰器
	- 为了理解方便，先来定义一个为函数视图准备的装饰器（在设计装饰器时基本都以函数视图作为考虑的被装饰对象），及一个要被装饰的类视图。
	```
	def my_decorator(func):
	    def wrapper(request, *args, **kwargs):
	        print('自定义装饰器被调用了')
	        print('请求路径%s' % request.path)
	        return func(request, *args, **kwargs)
	    return wrapper
	class DemoView(View):
	    def get(self, request):
	        print('get方法')
	        return HttpResponse('ok')
	    def post(self, request):
	        print('post方法')
	        return HttpResponse('ok')
	```
	- 在URL配置中装饰
	```
	urlpatterns = [
	    url(r'^demo/$', my_decorate(DemoView.as_view()))
	]
	```
	- 此种方式最简单，但因装饰行为被放置到了url配置中，单看视图的时候无法知道此视图还被添加了装饰器，不利于代码的完整性，不建议使用。
	- 此种方式会为类视图中的所有请求方法都加上装饰器行为（因为是在视图入口处，分发请求方式前）。
	- 在类视图中装饰
	- 在类视图中使用为函数视图准备的装饰器时，不能直接添加装饰器，需要使用method_decorator将其转换为适用于类视图方法的装饰器。
	- method_decorator装饰器使用name参数指明被装饰的方法
	```
	# 为全部请求方法添加装饰器
	@method_decorator(my_decorator, name='dispatch')
	class DemoView(View):
		def get(self, request):
			print('get方法')
			return HttpResponse('ok')
		def post(self, request):
			print('post方法')
			return HttpResponse('ok')
	# 为特定请求方法添加装饰器
	@method_decorator(my_decorator, name='get')
	class DemoView(View):
		def get(self, request):
			print('get方法')
			return HttpResponse('ok')
		def post(self, request):
			print('post方法')
			return HttpResponse('ok')
	```
	- 如果需要为类视图的多个方法添加装饰器，但又不是所有的方法（为所有方法添加装饰器参考上面例子），可以直接在需要添加装饰器的方法上使用method_decorator
	```
	from django.utils.decorators import method_decorator
	# 为特定请求方法添加装饰器
	class DemoView(View):
	    @method_decorator(my_decorator)  # 为get方法添加了装饰器
	    def get(self, request):
	        print('get方法')
	        return HttpResponse('ok')
	    @method_decorator(my_decorator)  # 为post方法添加了装饰器
	    def post(self, request):
	        print('post方法')
	        return HttpResponse('ok')
	    def put(self, request):  # 没有为put方法添加装饰器
	        print('put方法')
	        return HttpResponse('ok')
	```
- 类视图Mixin扩展类
	- 使用面向对象多继承的特性，可以通过定义父类（作为扩展类），在父类中定义想要向类视图补充的方法，类视图继承这些扩展父类，便可实现代码复用。
	- 定义的扩展父类名称通常以Mixin结尾
	```
	class MyDecoratorMixin(object):
	    @classmethod
	    def as_view(cls, *args, **kwargs):
	        view = super().as_view(*args, **kwargs)
	        view = my_decorator(view)
	        return view
	class DemoView(MyDecoratorMixin, View):
	    def get(self, request):
	        print('get方法')
	        return HttpResponse('ok')
	    def post(self, request):
	        print('post方法')
	        return HttpResponse('ok')
	```
	```
	class ListModelMixin(object):
	    """
	    list扩展类
	    """
	    def list(self, request, *args, **kwargs):
	        ...	
	class CreateModelMixin(object):
	    """
	    create扩展类
	    """
	    def create(self, request, *args, **kwargs):
	        ...
	class BooksView(CreateModelMixin, ListModelMixin, View):
	    """
	    同时继承两个扩展类，复用list和create方法
	    """
	    def get(self, request):
	        self.list(request)
	        ...
	    def post(self, request):
	        self.create(request)
	        ...
	```
1. 中间件
- Django中的中间件是一个轻量级、底层的插件系统，可以介入Django的请求和响应处理过程，修改Django的输入或输出
- 中间件的设计为开发者提供了一种无侵入式的开发方式，增强了Django框架的健壮性。
- 我们可以使用中间件，在Django处理视图的不同阶段对输入或输出进行干预。
- 中间件的定义方法
	- 定义一个中间件工厂函数，然后返回一个可以被调用的中间件。
	- 中间件工厂函数需要接收一个可以调用的get_response对象。
	- 返回的中间件也是一个可以被调用的对象，并且像视图一样需要接收一个request对象参数，返回一个response对象。
	```
	def simple_middleware(get_response):
	    # 此处编写的代码仅在Django第一次配置和初始化的时候执行一次。
	    def middleware(request):
	        # 此处编写的代码会在每个请求处理视图前被调用。
	        response = get_response(request)
	        # 此处编写的代码会在每个请求处理视图之后被调用。
	        return response
	    return middleware
	```
	- 例如，在users应用中新建一个middleware.py文件
	```
	def my_middleware(get_response):
	    print('init 被调用')
	    def middleware(request):
	        print('before request 被调用')
	        response = get_response(request)
	        print('after response 被调用')
	        return response
	    return middleware
	```
	- 定义好中间件后，需要在settings.py 文件中添加注册中间件
	```
	MIDDLEWARE = [
	    'django.middleware.security.SecurityMiddleware',
	    'django.contrib.sessions.middleware.SessionMiddleware',
	    'django.middleware.common.CommonMiddleware',
	    'django.middleware.csrf.CsrfViewMiddleware',
	    'django.contrib.auth.middleware.AuthenticationMiddleware',
	    'django.contrib.messages.middleware.MessageMiddleware',
	    'django.middleware.clickjacking.XFrameOptionsMiddleware',
	    'users.middleware.my_middleware',  # 添加中间件
	]
	```
	- 定义一个视图进行测试
	```
	def demo_view(request):
	    print('view 视图被调用')
	    return HttpResponse('OK')
	```
	- 备注：Django运行在调试模式下，中间件init部分有可能被调用两次
	- 多个中间件的执行顺序
		- 在请求视图被处理前，中间件由上至下依次执行
		- 在请求视图被处理后，中间件由下至上依次执行
	- 定义两个中间件
	```
	def my_middleware(get_response):
	    print('init 被调用')
	    def middleware(request):
	        print('before request 被调用')
	        response = get_response(request)
	        print('after response 被调用')
	        return response
	    return middleware
	
	def my_middleware2(get_response):
	    print('init2 被调用')
	    def middleware(request):
	        print('before request 2 被调用')
	        response = get_response(request)
	        print('after response 2 被调用')
	        return response
	    return middleware
	```
	- 注册添加两个中间件
	```
	MIDDLEWARE = [
	    'django.middleware.security.SecurityMiddleware',
	    'django.contrib.sessions.middleware.SessionMiddleware',
	    'django.middleware.common.CommonMiddleware',
	    'django.middleware.csrf.CsrfViewMiddleware',
	    'django.contrib.auth.middleware.AuthenticationMiddleware',
	    'django.contrib.messages.middleware.MessageMiddleware',
	    'django.middleware.clickjacking.XFrameOptionsMiddleware',
	    'users.middleware.my_middleware',  # 添加
	    'users.middleware.my_middleware2',  # 添加
	]
	```
	- 执行结果
	```
	init2 被调用
	init 被调用
	before request 被调用
	before request 2 被调用
	view 视图被调用
	after response 2 被调用
	after response 被调用
	```
#### 模板
1. Django自带模板
	- 配置
		- 在工程中创建模板目录templates
		- 在settings.py配置文件中修改TEMPLATES配置项的DIRS值
		```
		TEMPLATES = [
		    {
		        'BACKEND': 'django.template.backends.django.DjangoTemplates',
		        'DIRS': [os.path.join(BASE_DIR, 'templates')],  # 此处修改
		        'APP_DIRS': True,
		        'OPTIONS': {
		            'context_processors': [
		                'django.template.context_processors.debug',
		                'django.template.context_processors.request',
		                'django.contrib.auth.context_processors.auth',
		                'django.contrib.messages.context_processors.messages',
		            ],
		        },
		    },
		]
		```
	- 定义模板
		- 在templates目录中新建一个模板文件，如index.html
		```
		<!DOCTYPE html>
		<html lang="en">
		<head>
		    <meta charset="UTF-8">
		    <title>Title</title>
		</head>
		<body>
		    <h1>{{ city }}</h1>
		</body>
		</html>
		```
	- 模板渲染
		- Django提供了一个函数render实现模板渲染
		- render(request对象, 模板文件路径, 模板数据字典)
		```
		from django.shortcuts import render
		def index(request):
		    context={'city': '北京'}
		    return render(request,'index.html',context)
		```
	- 模板语法
		- 模板变量
			- 变量名必须由字母、数字、下划线（不能以下划线开头）和点组成。
			- 语法：{{变量}}
			- 模板变量可以使python的内建类型，也可以是对象
			```
			def index(request):
			    context = {
			        'city': '北京',
			        'adict': {
			            'name': '西游记',
			            'author': '吴承恩'
			        },
			        'alist': [1, 2, 3, 4, 5]
			    }
			    return render(request, 'index.html', context)
			```
			```
			<!DOCTYPE html>
			<html lang="en">
			<head>
			    <meta charset="UTF-8">
			    <title>Title</title>
			</head>
			<body>
			    <h1>{{ city }}</h1>
			    <h1>{{ adict }}</h1>
			    <h1>{{ adict.name }}</h1>  注意字典的取值方法
			    <h1>{{ alist }}</h1>  
			    <h1>{{ alist.0 }}</h1>  注意列表的取值方法
			</body>
			</html>
			```
		- 模板语句
			- for循环
			```
			{% for item in 列表 %}
			循环逻辑
			{{forloop.counter}}表示当前是第几次循环，从1开始
			{%empty%} 列表为空或不存在时执行此逻辑
			{% endfor %}
			```
			- if条件
			```
			{% if ... %}
			逻辑1
			{% elif ... %}
			逻辑2
			{% else %}
			逻辑3
			{% endif %}
			```
			- 比较运算符如：==、!=、<、>、<=、>=
			- 布尔运算符如：and、or、not
			- 运算符左右两侧不能紧挨变量或常量，必须有空格
			```
			{% if a == 1 %}  # 正确
			{% if a==1 %}  # 错误
			```
	- 过滤器
		- 使用管道符号|来应用过滤器，用于进行计算、转换操作，可以使用在变量、标签中。
		- 如果过滤器需要参数，则使用冒号:传递参数。
		```
		变量|过滤器:参数
		```
		- 列举几个自带过滤器
			- safe，禁用转义，告诉模板这个变量是安全的，可以解释执行
			- length，长度，返回字符串包含字符的个数，或列表、元组、字典的元素个数。
			- default，默认值，如果变量不存在时则返回默认值。
			```
			data|default:'默认值'
			```
			- date，日期，用于对日期类型的值进行字符串格式化，常用的格式化字符如下
				- Y表示年，格式为4位，y表示两位的年。
				- m表示月，格式为01,02,12等。
				- d表示日, 格式为01,02等。
				- j表示日，格式为1,2等。
				- H表示时，24进制，h表示12进制的时。
				- i表示分，为0-59。
				- s表示秒，为0-59。
				```
				value|date:"Y年m月j日  H时i分s秒"
				```
		- template提供的内置过滤器，不够用，不灵活，就可以自己定义一个过滤器
			- 在自己的app里建一个templatetags包，在包里创建一个后面要在HTML文件引用的py文件，
			- 在py文件中，先导入from django import template ，
			```
			  实例化对象register = template.Library()
			  创建一个template能认识的函数
			  对创建的每一个过滤器，都要用加上装饰器
			  
			  #导入包
			  from django import template
			  #实例化注册对象
			  register = template.Library()
			  #使用装饰器自定义过滤器
			  @register.filter
			  def func(x):
				return x*x
			```
			- 在HTML文件中引用
				- load templatetags
				- 使用过滤器
			- 注意点: templatetags文件夹 要在各自的应用内创建
	- 模板继承
		- 模板继承和类的继承含义是一样的，主要是为了提高代码重用，减轻开发人员的工作量
		- **父模板**
		- 如果发现在多个模板中某些内容相同，那就应该把这段内容定义到父模板中。
		- 标签block：用于在父模板中预留区域，留给子模板填充差异性的内容，名字不能相同。 
		- 为了更好的可读性，建议给endblock标签写上名字，这个名字与对应的block名字相同。
		- 父模板中也可以使用上下文中传递过来的数据。
		```
		{% block 名称 %}
		预留区域，可以编写默认内容，也可以没有默认内容
		{% endblock  名称 %}
		```
		- **子模板**
		- 标签extends：继承，写在子模板文件的第一行
		```
		{% extends "父模板路径"%}
		```
		- 子模版不用填充父模版中的所有预留区域，如果子模版没有填充，则使用父模版定义的默认值。
		- 填充父模板中指定名称的预留区域。
		```
		{% block 名称 %}
		实际填充内容
		{{ block.super }}用于获取父模板中block的内容
		{% endblock 名称 %}
		```
	- 注释
		- 单行注释语法
		```
		{#...#}
		```
		- 多行注释使用comment标签
		```
		{% comment %}
		...
		{% endcomment %}
		```
2. Jinja模板
- Django中使用jinja2模板
	- jinja2介绍
		- 是 Python 下一个被广泛应用的模板引擎，是由Python实现的模板语言
		- 他的设计思想来源于 Django 的模板引擎，并扩展了其语法和一系列强大的功能，尤其是Flask框架内置的模板语言
		- 由于django默认模板引擎功能不齐全,速度慢，所以我们也可以在Django中使用jinja2, jinja2宣称比django默认模板引擎快10-20倍。
	- 安装jinja2模块
	```
	pip install jinja2
	```
	- Django配置jinja2
		- 在项目文件中创建 jinja2_env.py 文件
		```
		from jinja2 import Environment
		def environment(**options):
		    env = Environment(**options)
		    return env
		```
		- 在settings.py文件
		```
		TEMPLATES = [
		    {
		        'BACKEND': 'django.template.backends.jinja2.Jinja2',#修改1
		        'DIRS': [os.path.join(BASE_DIR, 'templates')],
		        'APP_DIRS':True,
		        'OPTIONS':{
		            'environment': 'jinja2_env.environment',# 修改2
		            'context_processors':[
		                'django.template.context_processors.debug',
		                'django.template.context_processors.request',
		                'django.contrib.auth.context_processors.auth',
		                'django.contrib.messages.context_processors.messages',
		            ],
		        },
		    },
		]
		```
		- jinja2模板的使用绝大多数和Django自带模板一样
			- for循环有差异
			loop.index 当前循环迭代次数（从1开始）
			loop.index() 当前循环迭代次数（从0开始）
			loop.reindex 到循环结束需要迭代次数（从1开始）
			loop.reindex（） 到循环结束需要迭代次数（从0开始）
			loop.first 如果是第一次迭代，为True
			loop.last 如果是最后一次迭代，为True
			loop.length 序列中的项目数
			loop.cycle 在一串序列期间取值的辅助函数
	- jinja2自定义过滤器
	```
	from jinja2 import Environment
	def environment(**options):
	    env = Environment(**options)
	    # 2.将自定义的过滤器添加到 环境中
	    env.filters['do_listreverse'] = do_listreverse
	    return env
	# 1.自定义过滤器
	def do_listreverse(li):
	    if li == "B":
	        return "哈哈"
	```
1. CSRF攻击
	- CSRF全拼为Cross Site Request Forgery，译为跨站请求伪造。
	- CSRF指攻击者盗用了你的身份，以你的名义发送恶意请求。
	- 防止 CSRF 攻击
		- 在客户端向后端请求界面数据的时候，后端会往响应中的 cookie 中设置 csrf_token 的值
		- 在 Form 表单中添加一个隐藏的的字段，值也是 csrf_token
		- 在用户点击提交的时候，会带上这两个值向后台发起请求
		- 后端接受到请求，以会以下几件事件：
			- 从 cookie中取出 csrf_token
			- 从 表单数据中取出来隐藏的 csrf_token 的值
			- 进行对比
		- 如果比较之后两值一样，那么代表是正常的请求，如果没取到或者比较不一样，代表不是正常的请求，不执行下一步操作
#### 数据库
#### admin站点