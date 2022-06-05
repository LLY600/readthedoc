# 为什么选择DRF
1. 提供了可视化的API调试界面，在线测试接口
2. 根据需求来选择常规视图功能或更高级的功能
3. 不用自己写大量的CRUD接口，简单配置即可
4. 支持ORM（对象映射关系）和**非ORM的数据序列化**
# Django和Drf请求的生命周期
## Django请求生命周期
![](uTools_1654323628804.png)
- 注解：每一个中间件对应一个类，类中可以定义方法：请求和响应时都要挨个的穿过这些类
1. 前端发送请求
2. wsgi, 就是socket服务端，用于接收用户请求并将请求进行初次封装，然后将请求交给web框架（Flask、Django）
3. 中间件处理请求，帮助我们对请求进行校验或在请求对象中添加其他相关数据，例如：csrf、request.session
4. 路由匹配，根据当前请求的URL找到视图函数，如果是FBV写法，通过判断method两类型，找到对应的视图函数；如果是CBV写法，匹配成功后会自动去找dispatch方法，然后Django会通过dispatch反射的方式找到类中对应的方法并执行
5. 视图函数，在视图函数中进行业务逻辑的处理，可能涉及到：orm、view视图将数据渲染到template模板
6. 视图函数执行完毕之后，会把客户端想要的数据返回给dispatch方法，由dispatch方法把数据返回经客户端
7. 中间件处理响应
8. wsgi，将响应的内容发送给浏览器
9. 浏览器渲染

## Drf请求生命周期
1. 前端发送请求
2. Django的wsgi
3. 中间件
4. 路由系统_执行CBV的as_view()，就是执行内部的dispath方法
5. 在执行dispath之前，有版本分析和渲染器
6. 在dispath内，对request封装
7. 版本
8. 认证
9. 权限
10. 限流
11. 通过反射执行视图函数
12. 如果视图用到缓存( request.data or request.query_params )就用到了解析器
13. 视图处理数据，用到了序列化(对数据进行序列化或验证)
14. 视图返回数据可以用到分页
```
# 定义案例视图分析过程
 
from rest framework.views import APIView
from rest framework.response import Response
class TestView(APIView):
	def get(self,request,*args,**kwargs):
		return Response("drf get ok")
	def post(self,request,*args,**kwargs):
		return Response("drf post ok") # 这里的Response必须是drf下的Response，不能是Django原生的HttpResponse或者是JsonResponse，否则会出错

urlpatterns = [
	path('test/',views.Testview.as_view(),name="Test"),
]
```
- 首先我们先从路由配置中看到views.TestView.as_view()，调用的是TestView类视图下的as_view方法，但是我们上面定义该方法的时候，没有重写as_view()方法，所以会调用父类APIView中的as_view方法
```
路径：rest_framework.views.APIView.as_view

@classmethod
def as_view(cls, **initkwargs):
	"""
	Store the original class on the view function.

	This allows us to discover information about the view when we do URL
	reverse lookups.  Used for breadcrumb generation.
	"""
	if isinstance(getattr(cls, 'queryset', None), models.query.QuerySet):
		def force_evaluation():
			raise RuntimeError(
				'Do not evaluate the `.queryset` attribute directly, '
				'as the result will be cached and reused between requests. '
				'Use `.all()` or call `.get_queryset()` instead.'
			)
		cls.queryset._fetch_all = force_evaluation

	**view = super().as_view(**initkwargs)**
	view.cls = cls
	view.initkwargs = initkwargs

	# Note: session based authentication is explicitly CSRF validated,
	# all other authentication is CSRF exempt.
	**return csrf_exempt(view)**
```
- 通过这行代码view = super().as_view(**initkwargs)，可以知道APIView的as_view方法也调用了父类（Django原生）View的as_view方法，还禁用了csrf认证，源码如下
```
路径：django.views.generic.base.View.as_view

@classonlymethod
def as_view(cls, **initkwargs):
	"""Main entry point for a request-response process."""
	for key in initkwargs:
		if key in cls.http_method_names:
			raise TypeError(
				'The method name %s is not accepted as a keyword argument '
				'to %s().' % (key, cls.__name__)
			)
		if not hasattr(cls, key):
			raise TypeError("%s() received an invalid keyword %r. as_view "
							"only accepts arguments that are already "
							"attributes of the class." % (cls.__name__, key))

	def view(request, *args, **kwargs):
		self = cls(**initkwargs)
		self.setup(request, *args, **kwargs)
		if not hasattr(self, 'request'):
			raise AttributeError(
				"%s instance has no 'request' attribute. Did you override "
				"setup() and forget to call super()?" % cls.__name__
			)
		**return self.dispatch(request, *args, **kwargs)**
	view.view_class = cls
	view.view_initkwargs = initkwargs

	# take name and docstring from class
	update_wrapper(view, cls, updated=())

	# and possible attributes set by decorators
	# like csrf_exempt from dispatch
	update_wrapper(view, cls.dispatch, assigned=())
	**return view**
```
- as_view方法返回的是view，view返回的是dispatch方法，dispatch方法也是调用的APIView下的dispatch方法，源码如下：
```
路径：rest_framework.views.APIView.dispatch

def dispatch(self, request, *args, **kwargs):
	"""
	`.dispatch()` is pretty much the same as Django's regular dispatch,
	but with extra hooks for startup, finalize, and exception handling.
	"""
	self.args = args
	self.kwargs = kwargs
	request = self.initialize_request(request, *args, **kwargs)
	self.request = request
	self.headers = self.default_response_headers  # deprecate?

	try:
		self.initial(request, *args, **kwargs)

		# Get the appropriate handler method
		if request.method.lower() in self.http_method_names:
			handler = getattr(self, request.method.lower(),
							  self.http_method_not_allowed)
		else:
			handler = self.http_method_not_allowed

		response = handler(request, *args, **kwargs)

	except Exception as exc:
		response = self.handle_exception(exc)

	self.response = self.finalize_response(request, response, *args, **kwargs)
	return self.response
```
- dispatch返回一个response响应对象，得到请求的响应结果，返回给前端
# RestFul规范
- REST: 表征性状态转移(Representational State Transfer)
- RESTful规范：web数据请求接口设计规范

1. 通常使用https请求
2. 域名：有api关键字出现
	- https://api.example.com  (存在跨域问题)
	- https://example.com/api
1. 版本：不同版本需要标注
	- https://example.com/api/v1 |  https://example.com/api/1
	- https://example.com/api/v2 |  https://example.com/api/2
1. 资源：请求的目标数据称之为资源，资源一般都有名词复数表示
	- https://example.com/api/v1/books
1. 操作方式：不从请求链接体现操作方式，从请求方式上决定操作方式
	- get：https://example.com/api/v1/books  获取所有
	- post：https://example.com/api/v1/books  新增一本
	- put：https://example.com/api/v1/book/1  更新id=1的一本
	- patch：https://example.com/api/v1/book/1  更新id=1的一本
	- delete：https://example.com/api/v1/book/1  删除id=1的一本
1. 资源过滤：通过接口传递参数来过滤资源
	- https://example.com/api/v1/books?limit=10  限制10条
1. 状态码：返回数据要标准状态码，通过在数据中 {"status": 200}
	- SUCCESS(0, "查询成功")
	- NODATA(1, "非正确，无数据，显示基本信息")
	- FEAILED(2, "查询失败")
1. 错误信息：请求失败需要标注错误信息  {"message": "请求参数不合法"}
1. 操作结果：请求操作成功的返回结果 {"results": []}
	- get：返回资源列表 | 返回单一资源
	- post：返回单一新增资源
	- put：返回更新的资源
	- patch：返回更新的资源
	- delete：返回空文档
1. 子资源返回资源接口：返回的资源如果有子资源，返回子资源的链接地址，如查找书，书的封面图片就可以url表示

# Web应用模式
1. 在开发Web应用中，有两种应用模式
	- 前后端不分离【客户端看到的内容和所有界面效果都是由服务端提供】
	- 前后端分离【把前端的界面效果html，css，js分离到另一个服务端，python服务端只需要返回数据即可】
# 环境安装与配置
1. 环境准备
	```
	依赖安装：python(3.5+)、django(2.2+)、djangorestframework、pymysql
	创建项目：命令行执行django-admin startproject drfdemo
	配置参数：举例runserver 127.0.0.1:8000
	点击运行
	```

1. 添加rest_framework应用
	- settings.py文件
	```
	INSTALLED_APPS = [
		...
		'rest_framework',
	]
	```
	- Drf实现接口开发步骤
		- 将请求的数据（如JSON格式）转换为模型类对象
		- 操作数据库
		- 将模型类对象转换为响应的数据（如JSON格式）
1. 体验Drf简写代码的过程
	```
	django-admin startapp original  # 用于写原生django实现代码
	django-admin startapp students  # 用于写Drf实现代码
	```
	```
	INSTALLED_APPS = [
	...
    'rest_framework',
    'original',
    'students',
	]
	```
	```
	class Student(models.Model):
    """学生信息"""
    name = models.CharField(max_length=255, verbose_name="姓名")
    sex = models.BooleanField(default=1, verbose_name="性别")
    age = models.IntegerField(verbose_name="年龄")
    classmate = models.CharField(max_length=5, verbose_name="班级编号")
    description = models.TextField(max_length=1000, verbose_name="个性签名")

    class Meta:
        db_table = "tb_student"
        verbose_name = "学生" 
        verbose_name_plural = verbose_name
	```
	```
	准备数据库环境，以mysql为例
	settings.py文件更改：
	DATABASES = {
    'default': {
        # 'ENGINE': 'django.db.backends.sqlite3',
        'ENGINE': 'django.db.backends.mysql',
        # 'NAME': BASE_DIR / 'db.sqlite3',
        'NAME': 'drf2022',
        'HOST': '127.0.0.1',
        'PORT': 3306,
        'USER': 'root',
        'PASSWORD': '123456',
		}
	}
	```
	```
	在主应用的__init__.py文件
	from pymysql import install_as_MySQLdb
	install_as_MySQLdb()
	原因：django不能直接操控pymysql,只能操控MySQLdb,因此需要做一定的转换
	```
	```
	执行迁移文件
	python manage.py makemigrations
	python manage.py migrate
	```
	- **原生模式**
		```
		import json
		from django.views import View
		from django.http.response import JsonResponse
		from .models import Student

		"""
		POST /students/ 添加一个学生
		GET /students/ 获取所有学生
		GET /students/<pk> 获取一个学生
		PUT /students/<pk> 更新一个学生
		DELETE /students/<pk> 删除一个学生

		一个路由一个类，所以分成2个类完成
		"""


		class StudentsView(View):
			def post(self, request):
				# 接受表单数据
				name = request.POST.get("name")
				sex = request.POST.get("sex")
				age = request.POST.get("age")
				classmate = request.POST.get("classmate")
				description = request.POST.get("description")

				# 校验数据（略）
				# 操作数据库，保存数据
				instance = Student.objects.create(
					name=name,
					sex=sex,
					age=age,
					classmate=classmate,
					description=description,
				)
				# 返回数据:
				return JsonResponse(data={
					"id": instance.pk,
					"name": instance.name,
					"sex": instance.sex,
					"age": instance.age,
					"classmate": instance.classmate,
					"description": instance.description,
				}, status=201)

			def get(self, request):
				# 读取数据库
				student_list = list(Student.objects.values())
				# 返回数据
				return JsonResponse(data=student_list, status=200, safe=False)


		class StudentInfoView(View):
			"""
			获取某条数据
			"""

			def get(self, request, pk):
				try:
					instance = Student.objects.get(pk=pk)
					return JsonResponse(data={
						"id": instance.pk,
						"name": instance.name,
						"sex": instance.sex,
						"age": instance.age,
						"classmate": instance.classmate,
						"description": instance.description,
					}, status=200)
				except Student.DoesNotExist:
					return JsonResponse(data={}, status=404)

			"""
			更新某条数据
			"""

			def put(self, request, pk):
				# 接受表单数据
				data = json.loads(request.body)
				name = data.get("name")
				sex = data.get("sex")
				age = data.get("age")
				classmate = data.get("classmate")
				description = data.get("description")
				# 校验数据（略）
				# 操作数据库，保存数据
				try:
					instance = Student.objects.get(pk=pk)
					instance.name = name
					instance.sex = sex
					instance.age = age
					instance.classmate = classmate
					instance.description = description
					instance.save()
				except Student.DoesNotExist:
					return JsonResponse(data={}, status=404)
				# 返回数据:
				return JsonResponse(data={
					"id": instance.pk,
					"name": instance.name,
					"sex": instance.sex,
					"age": instance.age,
					"classmate": instance.classmate,
					"description": instance.description,
				}, status=201)

			def delete(self, request, pk):
				"""删除一条数据"""
				try:
					Student.objects.filter(pk=pk).delete()
				except:
					pass
				return JsonResponse(data={}, status=204)
		```
		```
		子应用添加urls.py文件，内容如下：
		from . import views
		from django.urls import path, re_path

		urlpatterns = [
			path("students/", views.StudentsView.as_view()),
			re_path("^students/(?P<pk>\d+)$", views.StudentInfoView.as_view()),
		]
		- **说明**：
		- re_path和path的作用都是一样的。只不过re_path是在写url的时候可以用正则表达式，功能更加强大。
		- 写正则表达式都推荐使用原生字符串。也就是以r开头的字符串。
		- 在正则表达式中定义变量，需要使用圆括号括起来。这个参数是有名字的，那么需要使用（?P<参数的名字>）。然后在后面添加正则表达式的规则。
		- 如果不是特别要求。直接使用path就够了，省的把代码搞的很麻烦（因为正则表达式其实是非常晦涩的，特别是一些比较复杂的正则表达式，今天写的明天可能就不记得了）。除非是url中确实是需要使用正则表达式来解决才使用re_path。
		```
		```
		主应用urls.py文件，内容如下：
		from django.contrib import admin
		from django.urls import path, include

		urlpatterns = [
			path('admin/', admin.site.urls),
			path('api/', include("students_origin.urls")),
		]
		```
		```
		post请求需要在settings.py文件关闭csrf
		MIDDLEWARE = [
		'django.middleware.security.SecurityMiddleware',
		'django.contrib.sessions.middleware.SessionMiddleware',
		'django.middleware.common.CommonMiddleware',
		# 'django.middleware.csrf.CsrfViewMiddleware',
		'django.contrib.auth.middleware.AuthenticationMiddleware',
		'django.contrib.messages.middleware.MessageMiddleware',
		'django.middleware.clickjacking.XFrameOptionsMiddleware',
		]
		```
	- **Drf模式**
		- 在students子应用目录中新建serializers.py用于保存该应用的序列化器。
		- 创建一个StudentModelSerializer用于序列化与反序列化。
		```
		from rest_framework import serializers
		from students_origin.models import Student

		class StudentModelSerializers(serializers.ModelSerializer):
			class Meta:
				model = Student
				fields = "__all__"
		- model 指明该序列化器处理的数据字段从模型类Student参考生成
		- fields 指明该序列化器包含模型类中的哪些字段，'all‘指明包含所有字段
		```
		- 在views.py定义视图
		```
		from rest_framework.viewsets import ModelViewSet
		from students_origin.models import Student
		from .serializers import StudentModelSerializers

		class StudentModelViewSet(ModelViewSet):
		queryset = Student.objects.all()
		serializer_class = StudentModelSerializers
		- queryset 指明该视图集在查询数据时使用的查询集
		- serializer_class 指明该视图在进行序列化或反序列化时使用的序列化器
		```
		- 定义路由
		- 子应用定义urls.py
		```
		from . import views
		from rest_framework.routers import DefaultRouter

		router = DefaultRouter()
		router.register('stu', views.StudentModelViewSet, basename='stu')

		urlpatterns = [
					  ] + router.urls
		```
		- 总路由
		```
		from django.contrib import admin
		from django.urls import path, include

		urlpatterns = [
			path('admin/', admin.site.urls),
			path('api/', include("students_origin.urls")),
			path('api/', include("students_drf.urls")),
		]
		```
		- 页面访问(http://127.0.0.1:8000/api/)，可以看到DRF提供的API Web浏览页面
# APIView视图
- APIView是REST framework提供的所有视图的基类，继承自Django的View父类。
- APIView与View的不同之处在于：
	- 传入到视图方法中的是REST framework的Request对象，而不是Django的HttpRequeset对象；
	- 视图方法可以返回REST framework的Response对象，视图会为响应数据设置（render）符合前端期望要求的格式；
	- 任何APIException异常都会被捕获到，并且处理成合适格式的响应信息返回给客户端；
	- 重新声明了一个新的as_views方法并在dispatch()进行路由分发前，会对请求的客户端进行身份认证、权限检查、流量控制。
- APIView新增了类属性
	- authentication_classes 列表或元组，身份认证类
	- permissoin_classes 列表或元组，权限检查类
	- throttle_classes 列表或元祖，流量控制类
1. 请求
	- REST framework 传入视图的request对象不再是Django默认的HttpRequest对象，而是REST framework提供的扩展了HttpRequest类的Request类的对象。
	- REST framework 提供了Parser解析器，在接收到请求后会自动根据Content-Type指明的请求数据类型（如JSON、表单等）将请求数据进行parse解析，解析为类字典[QueryDict]对象保存到Request对象中。
	- Request对象的数据是自动根据前端发送数据的格式进行解析之后的结果。
	- 无论前端发送的哪种格式的数据，我们都可以以统一的方式读取数据
	- 常用属性
		- **.data**
		- request.data 返回解析之后的请求体数据。类似于Django中标准的request.POST和 request.FILES属性，但提供如下特性：
			- 包含了解析之后的文件和非文件数据
			- 包含了对POST、PUT、PATCH请求方式解析后的数据
			- 利用了REST framework的parsers解析器，不仅支持表单类型数据，也支持JSON数据
		- **.query_params**
		- request.query_params与Django标准的request.GET相同，只是更换了更正确的名称而已。
		- **request._request**
		- 获取django封装的Request对象
	- 基本使用
		- backlog
1. 响应
	```
	rest_framework.response.Response
	```
	- REST framework提供了一个响应类Response，使用该类构造响应对象时，响应的具体数据内容会被转换（render渲染器）成符合前端需求的类型。
	- REST framework提供了Renderer 渲染器，用来根据请求头中的Accept（接收数据类型声明）来自动转换响应数据到对应格式。如果前端请求中未进行Accept声明，则会采用Content-Type方式处理响应数据，我们可以通过配置来修改默认响应格式。
	- 可以在rest_framework.settings查找所有的drf默认配置项
	```
	REST_FRAMEWORK = {
	    'DEFAULT_RENDERER_CLASSES': (  # 默认响应渲染类
	        'rest_framework.renderers.JSONRenderer',  # json渲染器，返回json数据
	        'rest_framework.renderers.BrowsableAPIRenderer',  # 浏览器API渲染器，返回调试界面
	    )
	}
	```
	- 构造方式
		```
		Response(data, status=None, template_name=None, headers=None, content_type=None)
		```
		- drf的响应处理类和请求处理类不一样，Response就是django的HttpResponse响应处理类的子类。
		- data数据不要是render处理之后的数据，只需传递python的内建类型数据即可，REST framework会使用renderer渲染器处理data。
		- data不能是复杂结构的数据，如Django的模型类对象，对于这样的数据我们可以使用Serializer序列化器序列化处理后（转为了Python字典类型）再传递给data参数。
		- 参数说明：
			- data: 为响应准备的序列化处理后的数据；
			- status: 状态码，默认200；
			- template_name: 模板名称，如果使用HTMLRenderer 时需指明；
			- headers: 用于存放响应头信息的字典；
			- content_type: 响应数据的Content-Type，通常此参数无需传递，REST framework会根据前端所需类型数据来设置该参数
	- response对象的属性
		- .data：传给response对象的序列化后，但尚未render处理的数据
		- .status_code：状态码的数字
		- .content：经过render处理后的响应数据
	- 状态码
		```
		# 1）信息告知 - 1xx
		HTTP_100_CONTINUE
		HTTP_101_SWITCHING_PROTOCOLS

		# 2）成功 - 2xx
		HTTP_200_OK
		HTTP_201_CREATED
		HTTP_202_ACCEPTED
		HTTP_203_NON_AUTHORITATIVE_INFORMATION
		HTTP_204_NO_CONTENT
		HTTP_205_RESET_CONTENT
		HTTP_206_PARTIAL_CONTENT
		HTTP_207_MULTI_STATUS

		# 3）重定向 - 3xx
		HTTP_300_MULTIPLE_CHOICES
		HTTP_301_MOVED_PERMANENTLY
		HTTP_302_FOUND
		HTTP_303_SEE_OTHER
		HTTP_304_NOT_MODIFIED
		HTTP_305_USE_PROXY
		HTTP_306_RESERVED
		HTTP_307_TEMPORARY_REDIRECT

		# 4）客户端错误 - 4xx
		HTTP_400_BAD_REQUEST
		HTTP_401_UNAUTHORIZED
		HTTP_402_PAYMENT_REQUIRED
		HTTP_403_FORBIDDEN
		HTTP_404_NOT_FOUND
		HTTP_405_METHOD_NOT_ALLOWED
		HTTP_406_NOT_ACCEPTABLE
		HTTP_407_PROXY_AUTHENTICATION_REQUIRED
		HTTP_408_REQUEST_TIMEOUT
		HTTP_409_CONFLICT
		HTTP_410_GONE
		HTTP_411_LENGTH_REQUIRED
		HTTP_412_PRECONDITION_FAILED
		HTTP_413_REQUEST_ENTITY_TOO_LARGE
		HTTP_414_REQUEST_URI_TOO_LONG
		HTTP_415_UNSUPPORTED_MEDIA_TYPE
		HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE
		HTTP_417_EXPECTATION_FAILED
		HTTP_422_UNPROCESSABLE_ENTITY
		HTTP_423_LOCKED
		HTTP_424_FAILED_DEPENDENCY
		HTTP_428_PRECONDITION_REQUIRED
		HTTP_429_TOO_MANY_REQUESTS
		HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE
		HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS

		# 5）服务器错误 - 5xx
		HTTP_500_INTERNAL_SERVER_ERROR
		HTTP_501_NOT_IMPLEMENTED
		HTTP_502_BAD_GATEWAY
		HTTP_503_SERVICE_UNAVAILABLE
		HTTP_504_GATEWAY_TIMEOUT
		HTTP_505_HTTP_VERSION_NOT_SUPPORTED
		HTTP_507_INSUFFICIENT_STORAGE
		HTTP_511_NETWORK_AUTHENTICATION_REQUIRED
		```
# 序列化器-Serializer
1. 作用
	- 序列化,序列化器会把模型对象转换成字典,经过response以后变成json字符串
	- 反序列化,把客户端发送过来的数据,经过request以后变成字典,序列化器可以把字典转成模型
	- 反序列化,完成数据校验功能
1. 定义序列化器
	- serializers是drf提供给开发者调用的序列化器模块
	- 里面声明了所有的可用序列化器的基类：
	- Serializer：序列化器基类，drf中所有的序列化器类都必须继承于Serializer
	- ModelSerializer模型序列化器基类，是序列化器基类的子类，在工作中，除了Serializer基类以外，最常用的序列化器类基类
	```
	from rest_framework import serializers


	class StudentModelSerializers(serializers.ModelSerializer):
		# 转换的字段申明
		id = serializers.IntegerField()
		name = serializers.CharField()
		sex = serializers.BooleanField()
		age = serializers.IntegerField()
		description = serializers.CharField()
	```
	```


	```
	```
	from django.urls import path
	from . import views

	urlpatterns = [
		path('student', views.StudentView.as_view())
	]
	```
	```
	from django.contrib import admin
	from django.urls import path, include

	urlpatterns = [
		path('admin/', admin.site.urls),
		path('api/', include("students_origin.urls")),
		path('api/', include("students_drf.urls")),
		path('sers/', include("sers.urls")),
	]
	```

2. 创建Serializer对象
	- Serializer的构造方法为
	```
	Serializer(instance=None, data=empty, **kwarg)
	```
	- 用于序列化时，将模型类对象传入instance参数
	- 用于反序列化时，将要被反序列化的数据传入data参数
	- 除了instance和data参数外，在构造Serializer对象时，还可通过context参数额外添加数据，如
	```
	serializer = AccountSerializer(account, context={'request': request})
	```
	- 使用序列化器的时候一定要注意，序列化器声明了以后，不会自动执行，需要我们在视图中进行调用才可以。
	- 序列化器无法直接接收数据，需要我们在视图中创建序列化器对象时把使用的数据传递过来。
	- 序列化器的字段声明类似于我们前面使用过的表单系统。
	- 开发restful api时，序列化器会帮我们把模型数据转换成字典.
	- drf提供的视图会帮我们把字典转换成json,或者把客户端发送过来的数据转换字典.
3. 序列化器的使用
	- **序列化**
	```
	from django.views import View
	from django.http.response import JsonResponse
	from .serializers import StudentModelSerializers
	from students_origin.models import Student
	
	
	class StudentView(View):
	    """序列化一个对象"""
	
	    def get(self, request):
	        # 获取数据集
	        student_list = Student.objects.first()
	        # 实例化系列化器
	        serializer = StudentModelSerializers(instance=student_list)
	        # 获取转换后的数据
	        data = serializer.data
	        # 响应数据
	        return JsonResponse(data=data, status=200, safe=False, json_dumps_params={'ensure_ascii': False})
	
	    """序列化多个对象"""
	
	    def get_(self, request):
	        # 获取数据集
	        student_list = Student.objects.all()
	        # 实例化系列化器
	        serializer = StudentModelSerializers(instance=student_list, many=True)
	        # 获取转换后的数据
	        data = serializer.data
	        # 响应数据
	        return JsonResponse(data=data, status=200, safe=False, json_dumps_params={'ensure_ascii': False})
	```
	- **反序列化**
	```
	# -*- coding:utf-8 -*-
	"""
	文件: serializers.py
	"""

	from rest_framework import serializers
	from students_origin.models import Student


	def check_classmate(data):  # 第四种数据校验方式（较少用）
		"""外部验证函数"""
		if len(data) != 3:
			raise serializers.ValidationError(detail='班级编号格式不正确！', code='check_classmate')
		return data


	class StudentModelSerializers(serializers.Serializer):
		# 转换的字段申明
		id = serializers.IntegerField(read_only=True)
		name = serializers.CharField(required=True)
		sex = serializers.BooleanField(default=True)
		age = serializers.IntegerField(max_value=100, min_value=0, error_messages={
			'min_value': 'The Age Must Be >= 0!',  # 第一种数据校验方式
			'max_value': 'The Age Must Be <= 100!',
		})
		classmate = serializers.CharField(required=True, validators=[check_classmate])  # validators是外部验证函数选项，值是列表，成员是函数名
		description = serializers.CharField(allow_null=True, allow_blank=True)  # 允许客户端不填写数据，或者值为”“

		#class Meta:
			#model = Student
			#fields = "__all__"

		def validate_name(self, data):  # 第二种数据校验方式
			"""
			验证单个字段
			方法名的格式必须为：validate_字段，否则序列化器不识别
			validate开头的方法，会自动被is_valid调用
			"""
			if data in ['python', 'django']:
				# 在序列化器中，失败可以通过抛出异常的方式来告知is_valid
				raise serializers.ValidationError(detail='名称不能是python或者django！', code='valide_name')
			return data

		def validate(self, attrs):  # 第三种数据校验方式
			"""
			验证所有字段
			validate是固定的方法名
			参数attrs是序列化器实例化时的data选项数据
			"""
			if attrs.get('classmate') == '301' and attrs.get('sex'):
				raise serializers.ValidationError(detail='301班只能是小姐姐！', code='validate')
			return attrs

		def create(self, validated_data):
			student = Student.objects.create(**validated_data)
			return student

		def update(self, instance, validated_data):
			for key, value in validated_data.items():
				setattr(instance, key, value)
			instance.save()
			return instance
	```
	```
	# views.py
	
	import json
	from django.views import View
	from django.http.response import JsonResponse
	from .serializers import StudentModelSerializers
	from students_origin.models import Student


	class StudentView(View):
		"""序列化一个对象"""

		def get1(self, request):
			# 获取数据集
			student_list = Student.objects.first()
			# 实例化系列化器
			serializer = StudentModelSerializers(instance=student_list)
			# 获取转换后的数据
			data = serializer.data
			# 响应数据
			return JsonResponse(data=data, status=200, safe=False, json_dumps_params={'ensure_ascii': False})

		"""序列化多个对象"""

		def get2(self, request):
			# 获取数据集
			student_list = Student.objects.all()
			# 实例化系列化器
			serializer = StudentModelSerializers(instance=student_list, many=True)
			# 获取转换后的数据
			data = serializer.data
			# 响应数据
			return JsonResponse(data=data, status=200, safe=False, json_dumps_params={'ensure_ascii': False})

		def get3(self, request):
			"""反序列化，采用字段选项来验证数据"""
			# 接受客户端提交的数据
			# data = json.dumps(request.body)
			data = {
				'name': 'xiao',
				'age': 22,
				'sex': True,
				'classmate': '3011',
				'description': 'description',
			}
			# 实例化序列化器
			serializer = StudentModelSerializers(data=data)
			# 调用序列化器进行数据验证
			# ret = serializer.is_valid()  # 不抛出异常
			serializer.is_valid(raise_exception=True)  # 直接抛出异常，代码不往下执行，常用
			# 获取校验以后的结果
			# 返回数据
			return JsonResponse(dict(serializer.validated_data))

		def get4(self, request):
			"""反序列化，验证完成后数据入库"""
			# 接受客户端提交的数据
			# data = json.dumps(request.body)
			data = {
				'name': 'xiao',
				'age': 22,
				'sex': False,
				'classmate': '301',
				'description': 'description',
			}
			# 实例化序列化器
			serializer = StudentModelSerializers(data=data)
			# 调用序列化器进行数据验证
			serializer.is_valid(raise_exception=True)  # 直接抛出异常，代码不往下执行，常用
			# 获取校验以后的结果
			serializer.save()
			# 返回数据
			return JsonResponse(serializer.data, status=201)

		def get(self, request):
			"""反序列化，验证完成后更新数据入库"""
			# 根据客户访问的url地址，获取pk值
			try:
				student = Student.objects.get(pk=4)
			except Student.DoesNotExist:
				return JsonResponse({'error': '参数不存在'}, status=400)
			# 接受客户端提交的数据
			data = {
				'name': 'xiao~~~~~~',
				'age': 32,
				'sex': False,
				'classmate': '301',
				'description': 'description',
			}

			serializer = StudentModelSerializers(instance=student, data=data)
			# 验证数据
			serializer.is_valid(raise_exception=True)
			# 入库
			serializer.save()
			# 返回结果
			return JsonResponse(serializer.data, status=201)
	```
	- 序列化器常用字段类型
	```
	BooleanField	BooleanField()
	NullBooleanField	NullBooleanField()
	CharField	CharField(max_length=None, min_length=None, allow_blank=False, trim_whitespace=True)
	EmailField	EmailField(max_length=None, min_length=None, allow_blank=False)
	RegexField	RegexField(regex, max_length=None, min_length=None, allow_blank=False)
	SlugField	SlugField(maxlength=50, min_length=None, allow_blank=False) 正则字段，验证正则模式 [a-zA-Z0-9-]+
	URLField	URLField(max_length=200, min_length=None, allow_blank=False)
	UUIDField	UUIDField(format=‘hex_verbose’) format: 1) 'hex_verbose' 如"5ce0e9a5-5ffa-654b-cee0-1238041fb31a" 2） 'hex' 如 "5ce0e9a55ffa654bcee01238041fb31a" 3）'int' - 如: "123456789012312313134124512351145145114" 4）'urn' 如: "urn:uuid:5ce0e9a5-5ffa-654b-cee0-1238041fb31a"
	IPAddressField	IPAddressField(protocol=‘both’, unpack_ipv4=False, **options)
	IntegerField	IntegerField(max_value=None, min_value=None)
	FloatField	FloatField(max_value=None, min_value=None)
	DecimalField	DecimalField(max_digits, decimal_places, coerce_to_string=None, max_value=None, min_value=None) max_digits: 最多位数 decimal_palces: 小数点位置
	DateTimeField	DateTimeField(format=api_settings.DATETIME_FORMAT, input_formats=None)
	DateField	DateField(format=api_settings.DATE_FORMAT, input_formats=None)
	TimeField	TimeField(format=api_settings.TIME_FORMAT, input_formats=None)
	DurationField	DurationField()
	ChoiceField	ChoiceField(choices) choices与Django的用法相同
	MultipleChoiceField	MultipleChoiceField(choices)
	FileField	FileField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL)
	ImageField	ImageField(max_length=None, allow_empty_file=False, use_url=UPLOADED_FILES_USE_URL)
	ListField	ListField(child=, min_length=None, max_length=None)
	DictField	DictField(child=)
	```
	- 选项参数
	```
	max_length	最大长度
	min_lenght	最小长度
	allow_blank	是否允许为空
	trim_whitespace	是否截断空白字符
	max_value	最小值
	min_value	最大值
	```
	- 通用参数
	```
	read_only	表明该字段仅用于序列化输出，默认False
	write_only	表明该字段仅用于反序列化输入，默认False
	required	表明该字段在反序列化时必须输入，默认True
	default	反序列化时使用的默认值
	allow_null	表明该字段是否允许传入None，默认False
	validators	该字段使用的验证器
	error_messages	包含错误编号与错误信息的字典
	label	用于HTML展示API页面时，显示的字段名称
	help_text	用于HTML展示API页面时，显示的字段帮助提示信息
	```
	- **模型类序列化器**
	- ModelSerializer与常规的Serializer相同，但提供了：
		- 基于模型类自动生成一系列字段
		- 基于模型类自动为Serializer生成validators，比如unique_together
		- 包含默认的create()和update()的实现
	```
	文件: serializers.py

	from rest_framework import serializers
	from students_origin.models import Student

	class StudentModelSerializers(serializers.ModelSerializer):
		# 转换的字段声明
		nickname = serializers.CharField(read_only=True, default='L圈圈')

		# 如果当前继承的是ModelSerializer，需要模型类信息
		class Meta:
			model = Student  # 必填
			# fields = "__all__"  # 必填，可以是字符串("__all__")、元组、列表
			fields = ['id', 'name', 'sex', 'age', 'nickname']
			read_only_fields = []  # 选填，只读字段，表示设置这里的字段只会在序列化阶段采用
			extra_kwargs = {  # 选填，字段额外选项声明
				"age": {
					'max_value': 150,
					'min_value': 10,
					'error_messages': {
						'min_value': '年龄最小值必须大于10岁',
						'max_value': '年龄最大值必须小于150岁',
					}
				}
			}
	```
	```
	文件: views.py
	import json

	from django.views import View
	from django.http.response import JsonResponse
	from .serializers import StudentModelSerializers
	from students_origin.models import Student


	class StudentView1(View):
		"""序列化一个对象"""

		def get1(self, request):
			# 获取数据集
			student_list = Student.objects.first()
			# 实例化系列化器
			serializer = StudentModelSerializers(instance=student_list)
			# 获取转换后的数据
			data = serializer.data
			# 响应数据
			return JsonResponse(data=data, status=200, safe=False, json_dumps_params={'ensure_ascii': False})

		"""序列化多个对象"""

		def get2(self, request):
			# 获取数据集
			student_list = Student.objects.all()
			# 实例化系列化器
			serializer = StudentModelSerializers(instance=student_list, many=True)
			# 获取转换后的数据
			data = serializer.data
			# 响应数据
			return JsonResponse(data=data, status=200, safe=False, json_dumps_params={'ensure_ascii': False})

		def get3(self, request):
			"""反序列化，采用字段选项来验证数据"""
			# 接受客户端提交的数据
			# data = json.dumps(request.body)
			data = {
				'name': 'xiao',
				'age': 22,
				'sex': True,
				'classmate': '3011',
				'description': 'description',
			}
			# 实例化序列化器
			serializer = StudentModelSerializers(data=data)
			# 调用序列化器进行数据验证
			# ret = serializer.is_valid()  # 不抛出异常
			serializer.is_valid(raise_exception=True)  # 直接抛出异常，代码不往下执行，常用
			# 获取校验以后的结果
			# 返回数据
			return JsonResponse(dict(serializer.validated_data))

		def get4(self, request):
			"""反序列化，验证完成后数据入库"""
			# 接受客户端提交的数据
			# data = json.dumps(request.body)
			data = {
				'name': 'xiao',
				'age': 22,
				'sex': False,
				'classmate': '301',
				'description': 'description',
			}
			# 实例化序列化器
			serializer = StudentModelSerializers(data=data)
			# 调用序列化器进行数据验证
			serializer.is_valid(raise_exception=True)  # 直接抛出异常，代码不往下执行，常用
			# 获取校验以后的结果
			serializer.save()
			# 返回数据
			return JsonResponse(serializer.data, status=201)

		def get(self, request):
			"""反序列化，验证完成后更新数据入库"""
			# 根据客户访问的url地址，获取pk值
			try:
				student = Student.objects.get(pk=4)
			except Student.DoesNotExist:
				return JsonResponse({'error': '参数不存在'}, status=400)
			# 接受客户端提交的数据
			data = {
				'name': 'xiao~~~~~~',
				'age': 32,
				'sex': False,
				'classmate': '301',
				'description': 'description',
			}

			serializer = StudentModelSerializers(instance=student, data=data)
			# 验证数据
			serializer.is_valid(raise_exception=True)
			# 入库
			serializer.save()
			# 返回结果
			return JsonResponse(serializer.data, status=201)


	class StudentView(View):
		"""模型序列化器"""

		def get1(self, request):
			"""序列化1个对象"""
			# 获取数据集
			student = Student.objects.first()
			# 实例化系列化器
			serializer = StudentModelSerializers(instance=student)
			# 获取转换后的数据
			data = serializer.data
			# 响应数据
			return JsonResponse(data=data, status=200, safe=False, json_dumps_params={'ensure_ascii': False})

		def get2(self, request):
			"""序列化多个对象"""
			# 获取数据集
			student_list = Student.objects.all()
			# 实例化系列化器
			serializer = StudentModelSerializers(instance=student_list, many=True)
			# 获取转换后的数据
			data = serializer.data
			# 响应数据
			return JsonResponse(data=data, status=200, safe=False, json_dumps_params={'ensure_ascii': False})

		def get3(self, request):
			"""反序列化，采用字段选项来验证数据"""
			# 接受客户端提交的数据
			# data = json.dumps(request.body)
			data = {
				'name': 'xiao',
				'age': 226,
				'sex': True,
				'classmate': '3011',
				'description': 'description',
			}
			# 实例化序列化器
			serializer = StudentModelSerializers(data=data)
			# 调用序列化器进行数据验证
			# ret = serializer.is_valid()  # 不抛出异常
			serializer.is_valid(raise_exception=True)  # 直接抛出异常，代码不往下执行，常用
			# 获取校验以后的结果
			# 返回数据
			return JsonResponse(dict(serializer.validated_data))

		def get4(self, request):
			"""反序列化，验证完成后数据入库"""
			# 接受客户端提交的数据
			# data = json.dumps(request.body)
			data = {
				'name': 'xiao',
				'age': 22,
				'sex': False,
				'classmate': '301',
				'description': 'description',
			}
			# 实例化序列化器
			serializer = StudentModelSerializers(data=data)
			# 调用序列化器进行数据验证
			serializer.is_valid(raise_exception=True)  # 直接抛出异常，代码不往下执行，常用
			# 获取校验以后的结果
			serializer.save()
			# 返回数据
			return JsonResponse(serializer.data, status=201)

		def get(self, request):
			"""反序列化，验证完成后更新数据入库"""
			# 根据客户访问的url地址，获取pk值
			try:
				student = Student.objects.get(pk=4)
			except Student.DoesNotExist:
				return JsonResponse({'error': '参数不存在'}, status=400)
			# 接受客户端提交的数据
			data = {
				'name': 'xiao323',
				'age': 32,
				'sex': False,
				'classmate': '301',
				'description': 'description',
			}

			serializer = StudentModelSerializers(instance=student, data=data)
			# 验证数据
			serializer.is_valid(raise_exception=True)
			# 入库
			serializer.save()
			# 返回结果
			return JsonResponse(serializer.data, status=201)
	```
1. 序列化器嵌套
- 在序列化器使用过程中，一般个序列化器对应一个模型数据。往往因为模型之间会存在外键关联，所以一般在输出数据时不仅要获取当前模型的数据，甚至其他模型的数据也需要同时返回，这种情况下，我们可以通过序列化器嵌套调用的方式，除帮我们把当前模型数据进行转换以外，还可以同时转换外键对应的模型数据。
```
file:models.py

from datetime import datetime
from django.db import models

"""
db_constraint=False，表示关闭物理外键
"""


class Student(models.Model):
    name = models.CharField(max_length=50, verbose_name='姓名')
    age = models.SmallIntegerField(verbose_name='年龄')
    sex = models.BooleanField(default=False)

    class Meta:
        db_table = 'student'

    def __str__(self):
        return self.name


class Course(models.Model):
    name = models.CharField(max_length=50, verbose_name='课程名称')
    teacher = models.ForeignKey('Teacher', on_delete=models.DO_NOTHING, related_name='course', db_constraint=False)

    class Meta:
        db_table = 'course'

    def __str__(self):
        return self.name


class Teacher(models.Model):
    name = models.CharField(max_length=50, verbose_name='姓名')
    sex = models.BooleanField(default=False)

    class Meta:
        db_table = 'teacher'

    def __str__(self):
        return self.name


class Achievement(models.Model):
    score = models.DecimalField(default=0, max_digits=4, decimal_places=1, verbose_name='成绩')
    student = models.ForeignKey(Student, on_delete=models.DO_NOTHING, related_name='s_achievement', db_constraint=False)
    course = models.ForeignKey(Course, on_delete=models.DO_NOTHING, related_name='c_achievement', db_constraint=False)
    create_dtime = models.DateTimeField(auto_created=datetime.now)

    class Meta:
        db_table = 'achievement'

    def __str__(self):
        return self.score
```
```
准备测试数据

INSERT INTO teacher(id,name,sex)VALUES(1,'李老师',0);
INSERT INTO teacher(id,name,sex)VALUES(2,'曹老师',1);
INSERT INTO teacher(id,name,sex)VALUES(3,'许老师',1);

INSERT INTO student(id,name,age,sex)VALUES(1,'小明',18,1);
INSERT INTO student(id,name,age,sex)VALUES(2,'小黑',18,1);
INSERT INTO student(id,name,age,sex)VALUES(3,'小白',18,1);
INSERT INTO student(id,name,age,sex)VALUES(4,'小红',17,0);
INSERT INTO student(id,name,age,sex)VALUES(5,'小程',18,0);
INSERT INTO student(id,name,age,sex)VALUES(6,'小年',16,0);
INSERT INTO student(id,name,age,sex)VALUES(7,'小明2',18,1);
INSERT INTO student(id,name,age,sex)VALUES(8,'小黑2',22,1);
INSERT INTO student(id,name,age,sex)VALUES(9,'小白2',18,1);
INSERT INTO student(id,name,age,sex)VALUES(10,'小红2',17,0);
INSERT INTO student(id,name,age,sex)VALUES(11,'小程2',18,0);
INSERT INTO student(id,name,age,sex)VALUES(12,'小灰',16,0)

INSERT INTO course(id,name,teacher_id)VALUES(1,'python',1);
INSERT INTO course(id,name,teacher_id)VALUES(2,'python进阶',2);
INSERT INTO course(id,name,teacher_id)VALUES(3,'python高级',3);
INSERT INTO course(id,name,teacher_id)VALUES(4,'django',2);
INSERT INTO course(id,name,teacher_id)VALUES(5,'django进阶',3);
INSERT INTO course(id,name,teacher_id)VALUES(6,'django高级',3);

INSERT INTO achievement (create_dtime,id,score,course_id,student_id)VALUES ('2021-06-16 01:18:25.496880',1,100.0,1,1);
INSERT INTO achievement (create_dtime,id,score,course_id,student_id)VALUES ('2021-06-16 01:18:25.496880',2,109.8,2,2);
INSERT INTO achievement (create_dtime,id,score,course_id,student_id)VALUES ('2021-06-16 01:18:25.496880',3,108.8,3,3);
INSERT INTO achievement (create_dtime,id,score,course_id,student_id)VALUES ('2021-06-16 01:18:25.496880',4,78.0,4,4);
INSERT INTO achievement (create_dtime,id,score,course_id,student_id)VALUES ('2021-06-16 01:18:25.496880',5,78.8,5,5);
INSERT INTO achievement (create_dtime,id,score,course_id,student_id)VALUES ('2021-06-16 01:18:25.496880',6,78.0,6,6);
INSERT INTO achievement (create_dtime,id,score,course_id,student_id)VALUES ('2021-06-16 01:18:25.496880',7,78.0,1,7);
INSERT INTO achievement (create_dtime,id,score,course_id,student_id)VALUES ('2021-06-16 01:18:25.496880',8,88.0,2,8);
```
- 一对多或者多对多的序列化器嵌套返回多个数据情况
- 默认情况下，模型经过序列化器的数据转换，对于外键的信息，仅仅把数据库里面的外键ID
```
文件: serializers.py

from rest_framework import serializers
from school.models import Teacher, Student, Achievement, Course


class TeacherModelSerializers(serializers.ModelSerializer):
    class Meta:
        model = Teacher
        fields = "__all__"


class CourseModelSerializers(serializers.ModelSerializer):
    teacher = TeacherModelSerializers()

    class Meta:
        model = Course
        fields = "__all__"


class AchievementModelSerializers(serializers.ModelSerializer):
    course = CourseModelSerializers()

    class Meta:
        model = Achievement
        fields = "__all__"


class StudentModelSerializers(serializers.ModelSerializer):
    # 序列化器嵌套
    # 名称不能随便写，必须是外键名称
    # 如果多条数据，需要加many=True
    s_achievement = AchievementModelSerializers(many=True)

    class Meta:
        model = Student
        # fields = "__all__"
        fields = ['id', 'name', 's_achievement']
```
```
返回结果如下，嵌套很复杂

[
	{
	       "id": 1,
	       "name": "小明",
	       "s_achievement": [
	           {
	               "id": 1,
	               "course": {
	                   "id": 1,
	                   "teacher": {
	                       "id": 1,
	                       "name": "李老师",
	                       "sex": false
	                   },
	                   "name": "python"
	               },
	               "create_dtime": "2021-06-16T01:18:25.496880Z",
	               "score": "100.0",
	               "student": 1
	           },
	           {
	               "id": 9,
	               "course": {
	                   "id": 1,
	                   "teacher": {
	                       "id": 1,
	                       "name": "李老师",
	                       "sex": false
	                   },
	                   "name": "python"
	               },
	               "create_dtime": "2021-06-16T01:18:25.496880Z",
	               "score": "99.0",
	               "student": 1
	           }
	       ]
	   }
]
```
- source选项取代主键数值，在多对一或者一对一的序列化器嵌套中，通过sourc选项，直接通过外键指定返回的某个字段数据
```
文件: serializers.py

from rest_framework import serializers
from school.models import Teacher, Student, Achievement, Course


class TeacherModelSerializers(serializers.ModelSerializer):
    class Meta:
        model = Teacher
        fields = "__all__"


class CourseModelSerializers(serializers.ModelSerializer):
    teacher = TeacherModelSerializers()

    class Meta:
        model = Course
        fields = "__all__"


class AchievementModelSerializers(serializers.ModelSerializer):
    course_name = serializers.CharField(source='course.name', default='课程名称')
    teacher_name = serializers.CharField(source='course.teacher.name', default='老师名称')

    class Meta:
        model = Achievement
        fields = "__all__"


class StudentModelSerializers(serializers.ModelSerializer):
    # 序列化器嵌套
    # 名称不能随便写，必须是外键名称
    # 如果多条数据，需要加many=True
    s_achievement = AchievementModelSerializers(many=True)

    class Meta:
        model = Student
        # fields = "__all__"
        fields = ['id', 'name', 's_achievement']
```
```
使用depth指定嵌套深度
文件: serializers.py
from rest_framework import serializers
from school.models import Teacher, Student, Achievement, Course


class TeacherModelSerializers(serializers.ModelSerializer):
    class Meta:
        model = Teacher
        fields = "__all__"


class CourseModelSerializers(serializers.ModelSerializer):
    teacher = TeacherModelSerializers()

    class Meta:
        model = Course
        fields = "__all__"


class AchievementModelSerializers(serializers.ModelSerializer):
    course_name = serializers.CharField(source='course.name', default='课程名称')
    teacher_name = serializers.CharField(source='course.teacher.name', default='老师名称')

    class Meta:
        model = Achievement
        fields = "__all__"


class AchievementModelSerializers1(serializers.ModelSerializer):
    class Meta:
        model = Achievement
        fields = "__all__"
        # depth = 1  # 自动往下再嵌套一层
        depth = 2  # 自动往下再嵌套两层
        # 层级深度
        # 成绩--》课程 1
        # 成绩--》课程--》老师 2


class StudentModelSerializers(serializers.ModelSerializer):
    # 序列化器嵌套
    # 名称不能随便写，必须是外键名称
    # 如果多条数据，需要加many=True
    s_achievement = AchievementModelSerializers1(many=True)

    class Meta:
        model = Student
        # fields = "__all__"
        fields = ['id', 'name', 's_achievement']
```
```
# 自定义模型方法
models.py

from datetime import datetime

from django.db import models

"""
db_constraint=False，表示关闭物理外键
"""


class Student(models.Model):
    name = models.CharField(max_length=50, verbose_name='姓名')
    age = models.SmallIntegerField(verbose_name='年龄')
    sex = models.BooleanField(default=False)

    class Meta:
        db_table = 'student'

    def __str__(self):
        return self.name
 
    @property
    def achievement(self):
        # return self.s_achievement.values()
        # 如果只想要学生名称
        return self.s_achievement.values('student__name')


class Course(models.Model):
    name = models.CharField(max_length=50, verbose_name='课程名称')
    teacher = models.ForeignKey('Teacher', on_delete=models.DO_NOTHING, related_name='course', db_constraint=False)

    class Meta:
        db_table = 'course'

    def __str__(self):
        return self.name


class Teacher(models.Model):
    name = models.CharField(max_length=50, verbose_name='姓名')
    sex = models.BooleanField(default=False)

    class Meta:
        db_table = 'teacher'

    def __str__(self):
        return self.name


class Achievement(models.Model):
    score = models.DecimalField(default=0, max_digits=4, decimal_places=1, verbose_name='成绩')
    student = models.ForeignKey(Student, on_delete=models.DO_NOTHING, related_name='s_achievement', db_constraint=False)
    course = models.ForeignKey(Course, on_delete=models.DO_NOTHING, related_name='c_achievement', db_constraint=False)
    create_dtime = models.DateTimeField(auto_created=datetime.now)

    class Meta:
        db_table = 'achievement'

    def __str__(self):
        return f'{self.score}'
```
```
class StudentModelSerializers(serializers.ModelSerializer):
    class Meta:
        model = Student
        # fields = "__all__"
        fields = ['id', 'name', 'achievement']
```
- 
# 视图
## 参考[https://blog.csdn.net/LHQ626/article/details/118400296](https://blog.csdn.net/LHQ626/article/details/118400296)
![](uTools_1654421118321.png)
## 视图中调用Http请求和响应处理类
1. 什么时候声明的序列化器需要继承序列化器基类Serializer,什么时候继承模型序列化器类ModelSerializer?
	- 继承序列化器类Serializer
		- 字段声明
		- 验证
		- 添加/保存数据功能
	- 继承模型序列化器类Modelseria1izer
		- 字段声明[可近，看黹要]
		- Meta声明
		- 验证
		- 添加/保存数据功能[可选]
1. 看数据是否从mysql数据库中获取，如果是则使用ModelSerializer,不是则使用Serializer
### **http请求响应**
1. drf除了在数据序列化部分简写代码以外,还在视图中提供了简写操作,所以在django有的django.views.View基础上,d倒封装了多个视图子类出来提供给我们使用。
2. drf提供的视图的主要作用:
	- 控制序列化器的执行(检验、保存、转换数据)
	- 控制数据库查询的执行
	- 调用请求类和响应类（这两个类也是由drf帮我们再次扩展了一些功能类)
### 请求与响应
- 内容协商:drf在django原有的基础上,新增了—个request对象继承到了APIView视图类,并在django原有的HttpResponse响应类的基础上实现了一个子类rest_framework.response.Response响应类。这两个类,都是基于内容协商来完成数据的格式转换的。
- request->parser解析类->识别客户端请求头中的Content-Type完成数据转换成>类字典(QueryDict,字典的子类)
- response->renderer渲染类->识别客户端请求头的Accept来提取客户端期望返回的数据格式,->转换成客户端的期望格式数据。如果客户端没有申明Accept，则按照Content-Type格式进行转换
#### **Request**
1. REST framework传入视图的requests对象不再是Django默认的HttpRequest象,而是REST framework提供的扩展了HttpRequest类的Requests类的对象。
2. REST framework提供了Parser解析器,在接收到请求后会自动根据Content-Type指明的请求数据类型(如JSON.表单等)将请求数据进行parse解析,解析为类字典[QueryDict]对象保存到Request对象中。
3. Request对象的数据是自动根据前端发送数据的格式进行解析之后的结果,
4. 无论前端发送的哪种格式的数据,我们都可以以统一的方式读取数据
5. 基本属性
	- .data
		- request.data返回解析之后的请求体数据。类似于Django标准的 request.Post和request.FILES属性,但提供如下特性:
			- 包含了解析之后的文件和非文件数据
			- 包含了对POST、PUT、PATCH请求方式解析后的数据
			- 利用了REST framework的parsers解析器,不仅支持表单类型数据,也支持JSON据
	- .query_params（查询参数、查询字符串）
		- request.query_params与Django标准的request.Get相同,只是更换了更正确的名称而已,
	- request._request
		- 获取django封装的Request对象
#### **Response**
```
rest_framework.response.Response
```
- REST framework供了一个响应类 Response，使用该类构造响应对象时,响应的具体数据内容会被转换(render染器)成符合前端需求的类型。
- REST framework供了Renderer渲染器,用来根据请求头中的Accept (接收数据类型声明)来自动转换响应数据到对应格式。如果前端请求中未进行Accept明,则会采用Content-Type式处理响应数据,我们可以通过配置来修改默认响应格式。
- 可以在源码rest_framework.settings 找所有的drf默认配置项
```
REST FRAMEWORK={
'DEFAULT_RENDERER_CLASSES':( # 默认响应渲染类
	'rest_framework.renderers.JSONRenderer, # json渲染器,返回json数据
	'rest_framework.renderers.BrowsableAPIRenderer, # 浏览器API染器,返同调试界面，否则只能看到JSON字符串形式
	)
}
```
- 构造方式
	- Response(data,status=None,template_name=None,headers=None,content_type=None)
	- drf的响应处理类和请求处理类不一样，Response就是django的HttpResponse响应处理类的子类。
	- data数据不要是render处理之后的数据，只需传递oython的内建类型数据即可，REST framework会使用renderer渲染器处理data。
	- data不能是复杂结构的数据，如Django的模型类对象，对于这样的数据我们可以使用Serializer序列化器序列化处理后(转为了Python字典类型)再传递给data参数.
	- 参数说明：
		- data:为响应准备的序列化处理后的数据
		- status:状态码，默认200；
		- template_name:模板名称，如果使用HTMLRenderer时需指明：
		- headers:用于存放响应头信息的字典；
		- content_type:响应数据的Content-Type,通常此参数无需传递，REST framework:会根据前端所需类型数据来设置该参数。
- response对象的属性（工作很少用）
	- .data
	- 传给response对象的序列化后，但尚未render处理的数据
	- .status code
	- 状态码的数字
	- .content
	- 经过render处理后的响应数据
- 状态码
	- 为了方便设置状态码，REST framewrok在rest_framework.status模块中提供了常用http状态码的常量。
## 视图作用
	- 控制序列化器的执行（检验、保存、转换数据）
	- 控制数据库模型的操作
## 普通视图
### 两个视图基类
#### APIView基本视图类
```
rest_framework.views.APIView
```
- APIView是REST framework提供的所有视图的基类，继承自Diango的view父类。
- APIView与view的不同之处在于：
	- 传入到视图方法中的是REST framework的Request对象，而不是Django的HttpRequeset对象；
	- 视图方法可以返回REST framework的Response对象，视图会为响应数据设置(render)符合前端期望要求的格式：
	- 任何APIException异常都会被捕获到，并且处理成合适格式的响应信息返回给客户端，django的View中所有异常全部以HTML格式显示，drf的APIVlewi或者APIView的子类会自动根据客户端的Accepti进行错误信息的格式转换。
	- 重新声明了一个新的as_view方法并在dispatch()进行路由分发前，会对请求的客户端进行身份认证、权限检查、流量控制。
- APIView除了继承了View原有的属性方法以外，还新增了类属性：
	- authentication classes列表或元组，身份认证类
	- permissoin classes列表或元组，权限检查类
	- throttle classes列表或元祖，流量控制类
- 在APIView中仍以常规的类视图定义方法来实现get()、post()或者其他请求方式的方法。
- 适用于不需要数据库操作，如果操作数据库需要自己代码实现
```
文件: serializers.py

from rest_framework import serializers
from students_origin.models import Student

class StudentModelSerializers(serializers.ModelSerializer):
	class Meta:
		model = Student
		fields = "__all__"
```
```
文件: views.py

from rest_framework import status
from rest_framework.response import Response
from rest_framework.views import APIView

from students_origin.models import Student
from viewdemo.serializers import StudentModelSerializers


class StudentAPIView(APIView):
	def get(self, request):
		student_list = Student.objects.all()
		serializer = StudentModelSerializers(instance=student_list, many=True)
		return Response(serializer.data, status=status.HTTP_200_OK)

	def post(self, request):
		serializer = StudentModelSerializers(data=request.data)
		serializer.is_valid(raise_exception=True)
		serializer.save()
		return Response(serializer.data, status=status.HTTP_201_CREATED)


class StudentInfoAPIView(APIView):
	def get(self, request, pk):
		try:
			student = Student.objects.get(pk=pk)
		except Student.DoesNotExist:
			return Response(status=status.HTTP_404_NOT_FOUND)
		serializer = StudentModelSerializers(instance=student)
		return Response(serializer.data)

	def put(self, request, pk):
		try:
			student = Student.objects.get(pk=pk)
		except Student.DoesNotExist:
			return Response(status=status.HTTP_404_NOT_FOUND)
		serializer = StudentModelSerializers(instance=student, data=request.data)
		serializer.is_valid(raise_exception=True)
		serializer.save()
		return Response(serializer.data, status=status.HTTP_201_CREATED)

	def post(self, request):
		serializer = StudentModelSerializers(data=request.data)
		serializer.is_valid(raise_exception=True)
		serializer.save()
		return Response(serializer.data, status=status.HTTP_201_CREATED)

	def delete(self, request, pk):
		try:
			Student.objects.get(pk=pk).delete()
		except Student.DoesNotExist:
			pass
		return Response(status=status.HTTP_204_NO_CONTENT)
```
```
文件: urls.py

from django.urls import path, re_path
from . import views

urlpatterns = [
	path('students/', views.StudentAPIView.as_view()),
	re_path(r'^students/(?P<pk>\d+)/$', views.StudentInfoAPIView.as_view())
]
```
#### GenericAPIView通用视图类
- 通用视图类主要作用就是把视图中的独特的代码抽取出来，让视图方法中的代码更加通用，方便把通用代码进行简写。
```
rest_framework.generics.GenericAPIView
```
- 继承自APIView,主要增加了操作序列化器和数据库查询的方法，作用是为下面Mixi扩展类的执行提供方法支持。通常在使用时，可搭配一个或多个Mixin扩展类。
- 提供的关于序列化器使用的属性与方法
	- **属性：**
	- serializer_class指明视图使用的序列化器类
	- **方法：**
	- get_serializer_class(self)
	- 当出现一个视图类中调用多个序列化器时，那么可以通过条件判断在get serializer_class方法中通过返回不同的序列化器类名就可以让视图方法执行不同的序列化器对象了。
	- 返回序列化器类，默认返回serializer_class,可以重写
	- get_serializer(self,args,*kwargs)
	- 返回序列化器对像，主要用来提供给Mixi扩展类使用，如果我们在视图中想要获取序列化器对像，也可以直接调用此方法。
	- 注意，该方法在提供序列化器对象的时候，会向序列化器对象的context属性补充三个数据：request、format、view,这三个数据对象可以在定义序列化器时使用。
	- request当前视图的请求对象
	- view当前请求的类视图对象
	- format当前请求期望返回的数据格式
- 提供的关于数据库查询的属性与方法
	- **属性：**
	- queryset指明使用的数据查询集
	- **方法：**
	- get_queryset(self)
	- 返回视图使用的查询集，主要用来提供给Mix扩展类使用，是列表视图与详情视图获取数据的基础，默认返回queryset属性，可以重写
	- get_object(self)
	- 返回详情视图所需的模型类数据对象，主要用来提供给Mixi扩展类使用。
	- 在试图中可以调用该方法获取详情信息的模型类对象。
	- 若详情访问的模型类对象不存在，会返回404。
	- 该方法会默认使用APIView提供的check_object_permissions方法检查当前对象是否有权限被访问。
- 其他可以设置的属性
	- pagination_class指明分页控制类
	- filter_backends指明数据过滤控制后端
```
class StudentGenericAPIView(GenericAPIView):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers

	def get(self, request):
		queryset = self.get_queryset()  # GenericAPIView提供的方法
		serializer = self.get_serializer(instance=queryset, many=True)
		return Response(serializer.data)

	def post(self, request):
		serializer = self.get_serializer(data=request.data)
		serializer.is_valid(raise_exception=True)
		serializer.save()
		return Response(serializer.data, status=status.HTTP_201_CREATED)


class StudentInfoGenericAPIView(GenericAPIView):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers

	def get(self, request, pk):
		instance = self.get_object()
		serializer = self.get_serializer(instance=instance)
		return Response(serializer.data)

	def put(self, request, pk):
		instance = self.get_object()
		serializer = self.get_serializer(instance=instance, data=request.data)
		serializer.is_valid(raise_exception=True)
		serializer.save()
		return Response(serializer.data, status=status.HTTP_201_CREATED)

	def delete(self, request, pk):
		self.get_object().delete()
		return Response(status=status.HTTP_204_NO_CONTENT)
```
			
### 五个视图扩展类
```
file:views.py

from rest_framework.mixins import ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin

class StudentMixView(GenericAPIView, ListModelMixin, CreateModelMixin):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers

	def get(self, request):
		return self.list(request)

	def post(self, request):
		return self.create(request)


class StudentInfoMixView(GenericAPIView, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers

	def get(self, request, pk):
		return self.retrieve(request, pk=pk)

	def put(self, request, pk):
		return self.update(request, pk=pk)

	def delete(self, request, pk):
		return self.destroy(request, pk=pk)
```
### 九个视图子类
- 上面的接口代码还可以续更加的精简，drf在使用GenericAPIView和Mixins进行组合以后，还是供了视图子类。
- 视图子类是通用视图类和模型甘扩展类的子类，提供了各种的视图方法调用mixins操作
	- ListAPIView = GenericAPIView + ListModelMixin 获取多条数据的视图方法
	- CreateAPIView = GenericAPIView + CreateModelMixin 添加一条数据的视图方法
	- RetrieveAPIV1ew=GenericAPIView + RetrieveModeLMiXin 获取一条数据的视图方法
	- UpdateAPIView = GenericAPIView + UpdateModelMixin 更新一条数据的视图方法
	- DestroyAPIView  = GenericAPIView + DestroyModelMixin 删除一条数据的视图方法
- 组合视图子类
	- ListcreateAPIview = ListAPIView + CreateAPIView
	- RetrieveUpdateAPIView = RetrieveAPIView + UpdateAPIView
	- RetrieveDestroyAPIView = RetrieveAPIView + DestroyAPIView
	- RetrieveUpdateDestroyAPIView = RetrieveAPIView + UpdateAPIView + DestroyAPIView
```
file:views.py

from rest_framework.generics import ListAPIView, CreateAPIView, RetrieveAPIView, UpdateAPIView
from rest_framework.generics import ListCreateAPIView,RetrieveUpdateAPIView
# class StudentView(ListAPIView, CreateAPIView):
class StudentView(ListCreateAPIView):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers


# class StudentInfoView(RetrieveAPIView, UpdateAPIView):
class StudentInfoView(RetrieveUpdateAPIView):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers
```
### 视图集ViewSet
- 使用视图集ViewSet,可以将一系列视图相关的代码逻辑和相关的http请求动作封装到一个类中：
	- list()提供一组数据
	- retrieve()提供单个数据
	- create)创建数据
	- update()保存数据
	- destory()删除数据
- ViewSet视图集类不再限制视图方法名只允许get()、post()等这种情况了而是实现允许开发者根据自己的需要定义自定义方法名，例如list)、create()等，然后经过路由中使用htp和这些视图方法名进行绑定调用。
- 视图集只在使用as_view()方法的时候，才会将action动作与具体请求方式对应上。
- 基本视图集ViewSet解决了APIview中代码重复的问题
	```
	file:view.py
	from rest_framework.viewsets import ViewSet
	
	class StudentViewSet(ViewSet):
		"""
		可以把所有操作写在一个类里面
		"""

		def get_list(self, request):
			student_list = Student.objects.all()
			serializer = StudentModelSerializers(instance=student_list, many=True)
			return Response(serializer.data)

		def post(self, request):
			serializer = StudentModelSerializers(data=request.data)
			serializer.is_valid(raise_exception=True)
			serializer.save()
			return Response(serializer.data, status=status.HTTP_201_CREATED)

		def get_student_info(self, request, pk):
			try:
				student = Student.objects.get(pk=pk)
			except Student.DoesNotExist:
				return Response(status=status.HTTP_404_NOT_FOUND)
			serializer = StudentModelSerializers(instance=student)
			return Response(serializer.data)

		def update(self, request, pk):
			try:
				student = Student.objects.get(pk=pk)
			except Student.DoesNotExist:
				return Response(status=status.HTTP_404_NOT_FOUND)
			serializer = StudentModelSerializers(instance=student, data=request.data)
			serializer.is_valid(raise_exception=True)
			serializer.save()
			return Response(serializer.data, status=status.HTTP_201_CREATED)

		def delete(self, request, pk):
			pass
	```
	```
	file:urls.py
	
	urlpatterns = [
		path('students5/', views.StudentViewSet.as_view({
			'get': 'get_list',
			'post': 'post',
		})),L
		re_path(r'^students5/(?P<pk>\d+)/$', views.StudentViewSet.as_view({
			'get': 'get_student_info',
			'put': 'update',
		})),
	]
	```
- 通用视图集GenericViewSet解决了ViewSet中代码重复的问题
	```
	file:urls.py
	
	urlpatterns = [
		path('students6/', views.StudentGenericViewSet.as_view({
			'get': 'list',
			'post': 'create',
		})),

		re_path(r'^students6/(?P<pk>\d+)/$', views.StudentGenericViewSet.as_view({
			'get': 'retrieve',
			'put': 'update',
			'delete': 'destory',
		})),
	]
	```
	```
	file:views.py
	
	class StudentGenericViewSet(GenericViewSet):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers

	def list(self, request):
		queryset = self.get_queryset()
		serializer = self.get_serializer(instance=queryset, many=True)
		return Response(serializer.data)

	def create(self, request):
		serializer = self.get_serializer(data=request.data)
		serializer.is_valid(raise_exception=True)
		serializer.save()
		return Response(serializer.data, status=status.HTTP_201_CREATED)

	def retrieve(self, request, pk):
		instance = self.get_object()
		serializer = self.get_serializer(instance=instance)
		return Response(serializer.data)

	def update(self, request, pk):
		instance = self.get_object()
		serializer = self.get_serializer(instance=instance, data=request.data)
		serializer.is_valid(raise_exception=True)
		serializer.save()
		return Response(serializer.data, status=status.HTTP_201_CREATED)

	def destory(self, request, pk):
		self.get_object().delete()
		return Response(status=status.HTTP_204_NO_CONTENT)
	```
- GenericViewSet 通用视图集+混入类
	```
	file:views.py
	
	from rest_framework.mixins import ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin
	
	class StudentGenericViewSet(GenericViewSet, ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin,DestroyModelMixin):
		queryset = Student.objects.all()
		serializer_class = StudentModelSerializers
	```
- 上面继承的类太多，合并视图集
- ReadonlyModelViewset = mixins.RetrieveModelMixin + mixins.ListModelMixin + GenericViewset
	```
	class StudentReadOnlyModelViewSet(ReadOnlyModelViewSet, CreateModelMixin, UpdateModelMixin, DestroyModelMixin):
		queryset = Student.objects.all()
		serializer_class = StudentModelSerializers
	```
- ModelViewSet = ReadOnlyModelViewSet + CreateModelMixin + UpdateModelMixin + DestroyModelMixin
- 实现了5个API接口
	```
	class StudentModelViewSet(ModelViewSet):
		queryset = Student.objects.all()
		serializer_class = StudentModelSerializers
	```
# 路由Routers
- 对于视图集ViewSet,我们除了可以自己手动指明请求方式与动作action之间的对应关系外，还可以使用Routers来帮助我们快速实现路由信息。依赖于视图集，只有使用了视图集，才可以使用自动生成路由，如果是非视图集，不需要使用路由集routers
- REST framework提供了两个router类,使用方式一致的。结果多一个或少一个根目录url地址的问题而已，用任何一个都可以。
	- SimpleRouter
	- DefaultRouter
## 使用方法
```
file:urls.py

from rest_framework.routers import SimpleRouter, DefaultRouter

# 实例化路由类
router = DefaultRouter()  # 会含有根路由
# router = SimpleRouter()
# 路由注册视图集
# register(prefix, viewset, base_name)
	# prefix 该视图集的路由前缀	
	# viewset 视图集
	# base_name 路由别名的前缀，用来生成urls名字
router.register('studentR', views.StudentModelViewSet, basename='studentR')
print(f'router.urls：{router.urls}')
# 把生成的路由列表和urlpatterns进行拼接组合
urlpatterns += router.urls
```
```
HTTP 200 OK
Allow: GET, HEAD, OPTIONS
Content-Type: application/json
Vary: Accept

{
	"studentR": "http://127.0.0.1:8000/viewdemo/studentR/"
}
```
## 视图集中附加action的声明
```
action装饰器可以接收两个参数：
	methods: 声明该action对应的请求方式，列表传递
	detail: 声明该action的路径是否与单一资源对应，以及是否是：路由前缀/<pk>/action方法名/
		True 表示路径格式是xxx/<pk>/action方法名/
		False 表示路径格式是xxx/action方法名/
	url_path：声明该action的路由尾缀。

from rest_framework.decorators import action

class StudentModelViewSet(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentModelSerializers

    @action(methods=['get', 'post'], detail=True, url_path='user/login')
    # http://127.0.0.1:8000/viewdemo/studentR/2/user/login/
    def login1(self, request, pk):
        return Response({'msg': 'login success!'})

    @action(methods=['get'], detail=False)
    def login2(self, request):
        return Response({'msg': 'login success!'})
```
## 标记额外的路由行为
viewset只定义了list, update, retrieve, create, update, destory, partial_update方法,  如果你想自定义一些额外的操作，可以使用@detail_route 或者 @list_route来实现@detail_route 装饰器在其 URL 模式中包含 pk，用于支持需要获取单个实例的方法。@list_route 修饰器适用于在对象列表上操作的方法。
```
from django.contrib.auth.models import User
from rest_framework import status
from rest_framework import viewsets
from rest_framework.decorators import detail_route, list_route
from rest_framework.response import Response
from myapp.serializers import UserSerializer, PasswordSerializer
class UserViewSet(viewsets.ModelViewSet):
    """
    提供标准操作的视图集
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
    
    # 如果你不想使用默认的url，你可以通过设置url_path参数来改变url
    # 如果您不想使用生成的默认名称，则可以使用 url_name 参数对其进行自定义
    @detail_route(methods=['post'], url_path='change-password')
    def set_password(self, request, pk=None):
        user = self.get_object()
        serializer = PasswordSerializer(data=request.data)
        if serializer.is_valid():
            user.set_password(serializer.data['password'])
            user.save()
            return Response({'status': 'password set'})
        else:
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
        
    @list_route()
    def recent_users(self, request):
        recent_users = User.objects.all().order('-last_login')
        page = self.paginate_queryset(recent_users)
        if page is not None:
            serializer = self.get_serializer(page, many=True)
            return self.get_paginated_response(serializer.data)
        serializer = self.get_serializer(recent_users, many=True)
        return Response(serializer.data)
```
# 认证
## Django三种认证方式
### 系统：session认证
```
rest_framework.authentication.SessionAuthentication
```
ajax请求通过认证：cookie中要携带 sessionid、csrftoken，请求头中要携带 x-csrftoken

### 第三方：jwt认证 
```
rest_framework_jwt.authentication.JSONWebTokenAuthentication
```
ajax请求通过认证：请求头中要携带 authorization，值为 jwt空格token

### 自定义：基于jwt、其它（常用）
1. 自定义认证类，继承BaseAuthentication(或其子类)，重写authenticate
2. authenticate中完成
    - 拿到认证标识 auth
    - 反解析出用户 user
    - 前两步操作失败 返回None => 游客
    - 前两步操作成功 返回user，auth => 登录用户
    - 注：如果在某个分支抛出异常，直接定义失败 => 非法用户
## 流程
1. 当用户进行登录的时候，运行了登录类的as_view()方法，进入了APIView类的dispatch方法
2. 执行self.initialize_request这个方法，里面封装了request和认证对象列表等其他参数
3. 执行self.initial方法中的self.perform_authentication，里面运行了user方法
4. 再执行了user方法里面的self._authenticate()方法
（以上是drf源码内容，下面为自定义方法）
5. 然后执行了自己定义的类中的authenticate方法，自己定义的类继承了BaseAuthentication类，里面有  authenticate方法，如果自己定义的类中没有authenticate方法会报错
6. 把从authenticate方法得到的user和auth赋值给user和auth方法
7. 这两个方法把user和auth的值分别赋值给request.user：是登录用户的对象；request.auth：是认证的信息字典
```
封装request的源码
APIView的dispatch中有个self.initialize_request，它返回了一个Request类，它封装了django的request和认证对象列表等其他参数

class APIView(View):
    # 1.进入了APIView类的dispatch方法
	def dispatch(self, request, *args, **kwargs):

		self.args = args
		self.kwargs = kwargs
		# 2.执行了self.initialize_request这个方法，封装了request和认证对象列表等其他参数
		request = self.initialize_request(request, *args, **kwargs)
		self.request = request
        self.headers = self.default_response_headers  # deprecate?

        try:
            # 3.执行self.initial方法中的self.perform_authentication，里面运行了user方法
            self.initial(request, *args, **kwargs)

            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)
            
     # 2.执行了self.initialize_request这个方法，封装了request和认证对象列表等其他参数
     '''
     def initialize_request(self, request, *args, **kwargs):

         parser_context = self.get_parser_context(request)
 
         return Request(
             request,
             parsers=self.get_parsers(),
             authenticators=self.get_authenticators(), # [MyAuthentication(),]
             negotiator=self.get_content_negotiator(),
             parser_context=parser_context
         )
         '''
        
    # 3.执行self.initial方法中的self.perform_authentication，里面运行了user方法
    '''
    def initial(self, request, *args, **kwargs):

        self.format_kwarg = self.get_format_suffix(**kwargs)

        neg = self.perform_content_negotiation(request)
        request.accepted_renderer, request.accepted_media_type = neg
        
		# 版本方法在这里
        version, scheme = self.determine_version(request, *args, **kwargs)
        request.version, request.versioning_scheme = version, scheme

        # 认证 权限 节流三兄弟
        self.perform_authentication(request)
        self.check_permissions(request)
        self.check_throttles(request)
        '''
    
    # 3.执行self.initial方法中的self.perform_authentication，里面运行了user方法
	def perform_authentication(self, request):
        request.user
        
        
    # 4、再执行了user方法里面的self._authenticate()方法    
    @property
    def user(self):
        """
        Returns the user associated with the current request, as authenticated
        by the authentication classes provided to the request.
        """
        if not hasattr(self, '_user'):
            with wrap_attributeerrors():
                self._authenticate()
        return self._user
```
## 返回值
1.  没有携带认证信息，直接返回None => 游客
2.  有认证信息，校验成功，返回一个元组，第一个参数赋值给request.user，第二个赋值给request.auth
3.  有认证信息，校验失败，抛异常 => 非法用户

## 示例
- 因为认证组件中需要使用到登录功能，所以我们使用django内置admin站点并创建一个管理员。admin运营站点的访问地址：http://127.0.0.1:8000/admin
```
python manage.py createsuperuser
Username (leave blank to use 'administrator'): admin
Email address: admin@123.com
Password:
Password (again):
This password is too short. It must contain at least 8 characters.
This password is too common.
This password is entirely numeric.
Bypass password validation and create user anyway? [y/N]: y
Superuser created successfully.
```
```
页面改中文与时区
settings.py文件
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
```
- 可以在配置文件中配置全局默认的认证方案，常见的认证方式：cookie、session、token
- site-packages/rest_framework/settings.py 默认配置文件
```
DEFAULTS = {
	'DEFAULT_AUTHENTICATION_CLASSES': [
		'rest_framework.authentication.SessionAuthentication',
		'rest_framework.authentication.BasicAuthentication'
	],
}
```
- 也可以在具体的视图类中通过设置authentication_classess类属性来设置单独的不同的认证方式
```
from rest_framework.authentication import SessionAuthentication, BaseAuthentication
from rest_framework.views import APIView

class ExampleView(APIView):
	authentication_classes = [SessionAuthentication, BaseAuthentication]

	def get(self, request):
		pass
```
- 认证失败会有两种可能的返回值，这个需要我们配合权限组件来使用：
	- 401 Unauthorized 未认证
	- 403 Permission Denied 权限被禁止
- 自定义配置
```
文件: 主应用下创建authentication.py
from rest_framework.authentication import BaseAuthentication
from django.contrib.auth import get_user_model


class CustomAuthentication(BaseAuthentication):
	"""
	自定义认证方式
	"""

	def authenticate(self, request):
		"""
		认证方法
		request: 本次客户端发送过来的http请求对象
		"""
		user = request.query_params.get("user")
		pwd = request.query_params.get("pwd")
		if user != "root" or pwd != "pwd":
			return None
		# get_user_model获取当前系统中用户表对应的用户模型类
		user = get_user_model().objects.first()
		return (user, None)  # 按照固定的返回格式填写 （用户模型对象, None）
	# 这个方法一般用不到
	def authenticate_header(self, request):
		pass
```
```
文件:views.py

from drfdemo.authentication import CustomAuthentication

class CustomView(APIView):
	# 局部配置
	authentication_classes = [CustomAuthentication]

	def get(self, request):
		print(f'request.user:{request.user}')
		return Response({'msg': 'ok'})
```
```
全局配置
主应用settings.py

REST_FRAMEWORK = {
	# 配置认证方式的选项【drf的认证是内部循环遍历每一个注册的认证类，一旦认证通过识别到用户身份，则不会继续循环】
	'DEFAULT_AUTHENTICATION_CLASSES': (
		**'drfdemo.authentication.CustomAuthentication',**          # 自定义认证
		'rest_framework.authentication.SessionAuthentication',  # session认证
		'rest_framework.authentication.BasicAuthentication',    # 基本认证
	)
}
```
## JWT拓展
### 认证流程
![](uTools_1654330719067.png)
### 传统token方式和JWT认证方式差异
1. 传统token方式（空间换时间）
	- 用户登录成功后，服务端生成一个随机token给用户，并且在服务端(数据库或缓存)中保存一份token，以后用户再来访问时需携带token，服务端接收到token之后，去数据库或缓存中进行校验token的是否超时、是否合法。
1. JWT方式（时间换空间）
	- 用户登录成功后，服务端通过jwt生成一个随机token给用户（服务端无需保留token），以后用户再来访问时需携带token，服务端接收到token之后，通过jwt对token进行校验是否超时、是否合法。
### JWT优势
1. token只在前端保存，后端只负责校验。
2. 内部集成了超时时间，后端可以根据时间进行校验是否超时。
3. 由于内部存在hash256加密，所以用户不可以修改token，只要一修改就认证失败。
### token被盗怎么办
1. token的过期时间设置短一些。
2. token和用户设备指纹信息绑定，但是这样会影响到用户的一些体验，如果恶意APP装载到了你的手机里，此方法无用。
3. 使用https，这样很难被盗。
### JWT 和Session、Cookies的不同
1. 密码签名
	- JWT 具有加密签名，而 Session Cookies 则没有。
2. JWT是无状态的
	- JWT 是无状态的，因为声明被存储在客户端，而不是服务端内存中。
	- 身份验证可以在本地进行，而不是在请求必须通过服务器数据库或类似位置中进行。这意味着可以对用户进行多次身份验证，而无需与站点或应用程序的数据库进行通信，也无需在此过程中消耗大量资源。
3. 可扩展性
	- Session Cookies 是存储在服务器内存中，这就意味着如果网站或者应用很大的情况下会耗费大量的资源。由于 JWT 是无状态的，在许多情况下，它们可以节省服务器资源。因此 JWT 要比 Session Cookies 具有更强的可扩展性。
4. JWT支持跨域认证
	- Session Cookies 只能用在单个节点的域或者它的子域中有效。如果它们尝试通过第三个节点访问，就会被禁止。如果你希望自己的网站和其他站点建立安全连接时，这是一个问题。
	- 使用 JWT 可以解决这个问题，使用 JWT 能够通过多个节点进行用户认证，也就是我们常说的跨域认证
### JWT和Session、Cookies的选型
1. 对于只需要登录用户并访问存储在站点数据库中的一些信息的中小型网站来说，Session Cookies通常就能满足。
2. 如果你有企业级站点，应用程序或附近的站点，并且需要处理大量的请求，尤其是第三方或很多第三方（包括位于不同域的API），则 JWT 显然更适合。
### JWT实现过程
#### 加密
- 用户提交用户名和密码给服务端，如果登录成功，使用jwt创建一个token，并给用户返回，jwt的生成token格式如下，即：由 . 连接的三段字符串组成。
- 生成规则如下：
- 第一段HEADER部分，固定包含签名算法和token类型，对此json进行base64url加密，这就是token的第一段。
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
- 第二段PAYLOAD部分，包含一些数据，对此json进行base64Url加密，这就是token的第二段。
```
{
  "sub": "1234567890", 
  "name": "John Doe",
  "iat": 1516239022 # 时间戳
}
```
- 第三段SIGNATURE部分，把前两段的base密文通过.拼接起来，然后对其进行HS256加盐再加密，再然后对HS256密文进行base64url加密，最终得到token的第三段。
```
base64url(
    HMACSHA256(
      base64UrlEncode(header) + "." + base64UrlEncode(payload),
      your-256-bit-secret (秘钥加盐)
    )
)
```
- 最后将三段字符串通过 .拼接起来就生成了jwt的token。
- 注意：base64url加密是先做base64加密，然后再将 - 替代 + 及 _ 替代 /
#### 解密
- 一般在认证成功后，把jwt生成的token返回给用户，以后用户再次访问时候需要携带token，此时jwt需要对token进行超时及合法性校验。
- 第一步: 获取token，将token分割成 header_segment、payload_segment、crypto_segment 三部分
```
jwt_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
signing_input, crypto_segment = jwt_token.rsplit(b'.', 1)
header_segment, payload_segment = signing_input.split(b'.', 1)
```
- 第二步: 对第二段进行base64url解密，并获取payload信息，检测token是否已经超时
```
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022 # 超时时间
}
```
- 第三步: 把第1,2端拼接，再次执行HS256加密
```
第一步: 第1,2部分密文拼接起来
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwi
aWF0IjoxNTE2MjM5MDIyfQ

第二步:对前2部分密文进行HS256加密 + 加盐
密文 = base64解密(SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c)
如果相等,表示token未被修改过.(认证通过)
```
#### 基于Python的pyjwt模块创建jwt的token
```
pip3 install pyjwt
```
```
import jwt
import datetime
from jwt import exceptions
SALT = 'iv%x6xo7l7_u9bf_u!9#g#m*)*=ej@bek5)(@u3kh*72+unjv='

def create_token():
    # 构造header
    headers = {
        'typ': 'jwt',
        'alg': 'HS256'
    }
    
    # 构造payload
    payload = {
        'user_id': 1, # 自定义用户ID
        'username': 'liuly2', # 自定义用户名
        'exp': datetime.datetime.utcnow() + datetime.timedelta(minutes=5) # 超时时间
    }
    # 构造signature即token
    token = jwt.encode(payload=payload, key=SALT, algorithm="HS256", headers=headers).decode('utf-8')
    return token

if __name__ == '__main__':
    token = create_token()
    print(token)
```
```
import jwt
import datetime
from jwt import exceptions
def get_payload(token):
    """
    根据token获取payload
    :param token:
    :return:
    """
    try:
        # 从token中获取payload【不校验合法性】
        # unverified_payload = jwt.decode(token, None, False)
        # print(unverified_payload)
        # 从token中获取payload【校验合法性】
        verified_payload = jwt.decode(token, SALT, True)
        return verified_payload
    except exceptions.ExpiredSignatureError:
        print('token已失效')
    except jwt.DecodeError:
        print('token认证失败')
    except jwt.InvalidTokenError:
        print('非法的token')
if __name__ == '__main__':
    token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzM1NTU1NzksInVzZXJuYW1lIjoid3VwZWlxaSIsInVzZXJfaWQiOjF9.xj-7qSts6Yg5Ui55-aUOHJS4KSaeLq5weXMui2IIEJU"
    payload = get_payload(token)
```
#### drf实战应用
```
from api.extensions.auth import JwtQueryParamsAuthentication
from api.utils.jwt_auth import create_token
from rest_framework.views import APIView
from rest_framework.response import Response
from api import models

# 登录视图
class ProLoginView(APIView):
    # 登录界面不用做验证
    authentication_classes = []

    def post(self,request,*args,**kwargs):
        user = request.data.get('username')
        pwd = request.data.get('password')

        # 查库返回Queryset类型对象
        user_object = models.UserInfo.objects.filter(username=user,password=pwd).first()
        if not user_object:
            return Response({'code':1000,'error':'用户名或密码错误'})
        
		# 根据用户信息生成token
        token = create_token({'id':user_object.id,'name':user_object.username})

        return Response({'code': 1001, 'data': token})

    
# 订单视图
class ProOrderView(APIView):
    def get(self, request, *args, **kwargs):
        print(request.user)
        return Response('订单列表')
```
```
文件：auth.py

from rest_framework.authentication import BaseAuthentication
from rest_framework.exceptions import AuthenticationFailed
from django.conf import settings
import jwt
from jwt import exceptions


# 自定义基于BaseAuthentication的认证方法
class JwtQueryParamsAuthentication(BaseAuthentication):
    """
    用户需要在url中通过参数进行传输token，例如：
    http://www.pythonav.com?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzM1NTU1NzksInVzZXJuYW1lIjoid3VwZWlxaSIsInVzZXJfaWQiOjF9.xj-7qSts6Yg5Ui55-aUOHJS4KSaeLq5weXMui2IIEJU
    """
    def authenticate(self, request):
        # 获取token并判断token的合法性，query_params可以获取url中的参数，默认是Queryset对象
        token = request.query_params.get('token')
        salt = settings.SECRET_KEY

        # 1.切割
        # 2.解密第二段/判断过期
        # 3.验证第三段合法性

        try:
            # 解密token
            payload = jwt.decode(token, salt, True)
        except exceptions.ExpiredSignatureError:
            raise AuthenticationFailed({'code':1003,'error':"token已失效"})
        except jwt.DecodeError:
            raise AuthenticationFailed({'code':1004,'error':"token认证失败"})
        except jwt.InvalidTokenError:
            raise AuthenticationFailed({'code':1005,'error':"非法的token"})

        # 三种操作
        # 1.抛出异常，后续不再执行
        # 2.return一个元祖（1，2），认证通过；并将其分别赋值给request.user和request.auth
        # 3.None
        return(payload,token)
    
    
class JwtAuthorizationAuthentication(BaseAuthentication):
    """
    用户需要通过请求头的方式来进行传输token，例如：
    Authorization:jwt eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1NzM1NTU1NzksInVzZXJuYW1lIjoid3VwZWlxaSIsInVzZXJfaWQiOjF9.xj-7qSts6Yg5Ui55-aUOHJS4KSaeLq5weXMui2IIEJU
    """
    def authenticate(self, request):

        # 非登录页面需要校验token
        authorization = request.META.get('HTTP_AUTHORIZATION', '')
        auth = authorization.split()
        if not auth:
            raise exceptions.AuthenticationFailed({'error': '未获取到Authorization请求头', 'status': False})
        if auth[0].lower() != 'jwt':
            raise exceptions.AuthenticationFailed({'error': 'Authorization请求头中认证方式错误', 'status': False})

        if len(auth) == 1:
            raise exceptions.AuthenticationFailed({'error': "非法Authorization请求头", 'status': False})
        elif len(auth) > 2:
            raise exceptions.AuthenticationFailed({'error': "非法Authorization请求头", 'status': False})

        token = auth[1]
        result = parse_payload(token)
        if not result['status']:
            raise exceptions.AuthenticationFailed(result)

        # 如果想要request.user等于用户对象，此处可以根据payload去数据库中获取用户对象。
        return (result, token)
```
```
文件：jwt_auth.py

import jwt
import datetime
from django.conf import settings

# 创建生成的token方法
def create_token(payload, timeout):

    salt = settings.SECRET_KEY
    
    # 构造header
    headers = {
        'typ': 'jwt',
        'alg': 'HS256'
    }
    
    # 构造payload
    payload['exp'] = {datetime.datetime.utcnow() + datetime.timedelta(minutes=timeout)}
    
    # 构造signature即token
    token = jwt.encode(payload=payload, key=salt, algorithm="HS256", headers=headers).decode('utf-8')

    return token
```
```
文件：settings.py

# 全局使用
REST_FRAMEWORK = {
    "DEFAULT_AUTHENTICATION_CLASSES":['api.extensions.auth.JwtQueryParamsAuthentication',]
}
```

# 权限
## 两种实现
1. 系统：
	- AllowAny：允许所有用户，校验方法直接返回True
	- IsAuthenticated：只允许登录用户
		- 必须request.user和request.user.is_authenticated都通过
	- IsAuthenticatedOrReadOnly：游客只读，登录用户无限制
		- get、option、head 请求无限制
		- 前台请求必须校验 request.user和request.user.is_authenticated
	- IsAdminUser：是否是后台用户
		- 校验 request.user和request.user.is_staff    
		- is_staff(可以登录后台管理系统的用户)
1. 自定义：基于auth的Group与Permission表
	- 自定义权限类，继承BasePermission，重写has_permission
	- has_permission中完成
		- 拿到登录用户 user <= request.user
		- 校验user的分组或是权限
		- 前两步操作失败 返回False => 无权限
		- 前两步操作成功 返回True => 有权限

## 流程
1. 当用户执行一个业务的时候，运行了as_view方法，进入了APIView类的dispatch方法
2. （此处不需要封装request了，因为在认证过程中封装了）
3. 进入self.initial方法中的self.check_permissions(request)方法
4. 里面执行了for循环，把每个权限类实例化对象
5. 执行自己定义的权限类里面的has_permission方法，里面会判断request.user是否存在
6. 不存在就返回False，存在就返回True
7. 之后执行self.permission_denied报错方法，返回的是False就报错，可以自定义报错信息，在has_permission方法中写message = {"status": False, "error": "登录成功之后才能评论"}，就实现了自定义报错

## 返回值
1. True, 有权限，进行下一步认证(频率认证)
2. False, 无权限，将信息返回给前端

## 默认配置
- 可以在配置文件settings.py中全局设置默认的权限管理类
```
REST_FRAMEWORK = {
	'DEFAULT_PERMISSION_CLASSES': (
		'rest_framework.permissions.IsAuthenticated',
	)
}
```
- 如果未指明，则采用如下默认配置rest_framework/settings.py
```
DEFAULTS = {
	'DEFAULT_PERMISSION_CLASSES': (
	   'rest_framework.permissions.AllowAny',
	)
}
```
- 也可以在具体的视图中通过permission_classes属性来进行局部设
```
from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView

class ExampleView(APIView):
	permission_classes = (IsAuthenticated,)
```
- 提供的权限
	- AllowAny 允许所有用户，默认权限
	- IsAuthenticated 仅通过登录认证的用户
	- IsAdminUser 仅管理员用户
	- IsAuthenticatedOrReadOnly 已经登录认证的用户可以对数据进行增删改操作，没有登录认证的只能查看数据
```
from rest_framework.permissions import IsAuthenticated, IsAdminUser, IsAuthenticatedOrReadOnly

class HomeInfoAPIView(APIView):
	# permission_classes = [IsAuthenticated]
	# permission_classes = [IsAdminUser]
	permission_classes = [IsAuthenticatedOrReadOnly]

	def get(self, request):
		return Response({'msg': 'ok'})

	def post(self, request):
		return Response({'msg': 'ok'})
```
## 自定义权限
- 如需自定义权限，需继承rest_framework.permissions.BasePermission父类，并实现以下两个任何一个方法或全部
	- has_permission(self, request, view)：是否可以访问视图， view表示当前视图对象
	- has_object_permission(self, request, view, obj)：是否可以访问模型对象， view表示当前视图， obj为模型数据对象
```
文件: 主应用下创建permissions.py
from rest_framework.permissions import BasePermission

class DemoPermission(BasePermission):
	def has_permission(self, request, view):
		"""
		视图权限
		返回结果为True则表示允许访问视图类
		request: 本次客户端提交的请求对象
		view: 本次客户端访问的视图类
		"""
		user = request.query_params.get("user")
		return user == "admin"

	def has_object_permission(self, request, view, obj):
		"""
		模型权限
		返回结果为True则表示允许操作模型对象
		通常写了has_permission视图权限，就不需要再写这个了
		obj：本次权限判断的模型对象
		"""
		from school.models import Student
		if (isinstance(obj, Student)):
			user = request.query_params.get("user")
			return user == 'admin'
		return True
```
```
文件：views.py

局部使用
class StudentInfoAPIView(RetrieveAPIView):
queryset = Student
serializer_class = StudentModelSerializers
permission_classes = [DemoPermission]
```
```
全局配置
REST_FRAMEWORK = {
	# 权限全局配置
	# 'DEFAULT_PERMISSION_CLASSES': [
	#     # 设置所有视图只能被已经登录认证过的用户访问
	#     'drfdemo.permissions.DemoPermission',
	# ]
}
```
# 限流
- 匿名用户通过ip地址来控制访问频率
- 已登录用户通过token来控制
## 实现原理
- 把所有登录记录时间放在一个列表中，当用户请求网页的时候，用现在的时间减去约束的时间间隔，然后把小于这个时间记录排除，再计算出时间间隙的记录条数，如果其中的条数小于规定的条数则可以访问并且把当前时间添加进列表中，如果大于或等于则不让其访问。
## 流程
1. 当用户请求网页的时候，后台允许该界面的url中的as_views()，运行了as_view方法，进入了APIView类的dispatch方法
2. 进入self.initial方法中的self.check_throttles(request)方法
3. 循环运行节流类中的allow_request方法
4. 如果可以访问返回True，如果不能访问则返回False，之后返回check_throttles，如果是False则运行SimpleRateThrottle类中的wait方法得到需要等待的时间在页面上显示
## 返回值
1. True，允许访问
2. False，访问太频繁
3. wait()返回值：返回一个整数，表示下次还有多久可以访问
## 示例
### 方法一(重写原生方法BaseThrottle.allow_request)
```
文件：throttle.py

from rest_framework.throttling import BaseThrottle
import time

# 定义全局变量，用于存放访问记录
VISIT_RECORD = {}

# 需要继承BaseThrottle
class VisitThrottle(BaseThrottle):
    '''60s内只能访问3次'''

    def __init__(self): # 用于await计算剩余访问时间
        self.history = []

    # 在定义的类中复写allow_request方法，返回True或者False表示可以访问或者访问频率太高
    def allow_request(self, request, view):
        # 1.获取用户ip
        # 方法一：通过原生方法获取用户ip
        # remote_addr = request.META.get('REMOTE_ADDR')

        # 方法二：通过父方法获取用户唯一标识ip
        remote_addr = self.get_ident(request)
        ctime = time.time()
        # 这是用户第一次访问,将其进行记录，并且返回True，允许继续访问
        if remote_addr not in VISIT_RECORD:
            print("没有此IP")
            VISIT_RECORD[remote_addr] = [ctime,]
            return True
        # 如果不是第一次访问，获取所有的记录
        history = VISIT_RECORD.get(remote_addr)
        self.history = history

        # 判断最开始的时刻与现在的时刻的差值是否在规定的时间范围内，比如在60s内，如果不在，
        # 可以去除最开始的时刻记录
        while history and history[-1] < ctime - 60:
            # 删掉最后一个
            history.pop()
		# 此时列表中的时刻记录都是在规定的时间范围内，判断时刻的个数也就是访问的次数
        if len(history) < 3:
            history.insert(0, ctime)
            return True
        
	# 还需要等多少秒才能访问
    def wait(self):
        ctime = time.time()
        return 8-(ctime-self.history[-1])
```
```
全局
# settings.py
'DEFAULT_THROTTLE_CLASSES': ['drfdemo.throttle.VisitThrottle'],
```
```
局部使用
class AuthView(APIView):
    """
    用于用户登录认证
    """
    throttle_classes = [VisitThrottle,]
```
### 方法二（继承内置类）
```
from rest_framework.throttling import  SimpleRateThrottle

# 对游客的限制
class VisitThrottle(SimpleRateThrottle):
    scope = "Vistor"
    def get_cache_key(self, request, view):
        # 唯一表示是IP
        return self.get_ident(request)
    
# 对登陆用户的限制
class UserThrottle(SimpleRateThrottle):
    scope = "User"
    def get_cache_key(self, request, view):
        # 唯一表示是用户名
        return self.user.username
```
- scope从settings.py中寻找DEFAULT_THROTTLE_RATES字典的Key,就是访问频率限制，scope可以区分不同的函数的不同限制；get_cache_key(self, request, view)返回一个唯一标示用以区分不同的用户，对于匿名用户返回IP保存到缓存中限制访问，对于注册的用户取用户名（唯一）来区分就可以
```
全局应用

REST_FRAMEWORK = {
    ...
    'DEFAULT_THROTTLE_RATES': {
        # 对游客的限制每分钟3次
        'Vistor': '3/m',
        # 对登陆用户的限制每分钟10次
        'User': '10/m'
    }
}
```
```
主应用settings.py

REST_FRAMEWORK = {
	# 限流全局配置
	'DEFAULT_THROTTLE_CLASSES': [  # 限流配置类
		'rest_framework.throttling.AnonRateThrottle',  # 未认证用户[未登录用户]
		'rest_framework.throttling.UserRateThrottle',  # 已认证用户[已登录用户]
	],
	
	# 局部配置
	'DEFAULT_THROTTLE_RATES': {  # 频率配置
		'anon': '2/day',  # 针对游客的访问频率进行限制，实际上，drf只是识别首字母，但是为了提高代码的维护性，建议写完整单词
		'user': '5/day',  # 针对会员的访问频率进行限制，
	}
}

DEFAULT_THROTTLE_RATES 可以使用 second, minute, hour 或day来指明周期。
```
- 可选限流
	- AnonRateThrottle
		- 限制所有匿名未认证用户，使用IP区分用户。【很多公司这样的，IP结合设备信息来判断，当然比IP要靠谱一点点而已】
		- 使用DEFAULT_THROTTLE_RATES['anon'] 来设置频次
	- UserRateThrottle
		- 限制认证用户，使用User模型的 id主键 来区分。
		- 使用DEFAULT_THROTTLE_RATES['user'] 来设置频次
	- ScopedRateThrottle
		- 限制用户对于每个视图的访问频次，使用ip或user id。
```
from rest_framework.throttling import UserRateThrottle

class StudentInfoAPIView(RetrieveAPIView):
	queryset = Student
	serializer_class = StudentModelSerializers
	permission_classes = [DemoPermission]
	throttle_classes = [UserRateThrottle]
```
- 自定义限流
```
主应用settings.py配置

REST_FRAMEWORK = {
	# 限流全局配置
	'DEFAULT_THROTTLE_CLASSES': (  # 限流配置类
		'rest_framework.throttling.AnonRateThrottle',  # 未认证用户[未登录用户]
		'rest_framework.throttling.UserRateThrottle',  # 已认证用户[已登录用户]
		'rest_framework.throttling.ScopedRateThrottle',  # 基于自定义的命名空间限流

	),
	'DEFAULT_THROTTLE_RATES': {  # 频率配置
		'vip': '3/day',  # 针对会员的访问频率进行限制，
	}
}
```
```
views.py

class VipAPIView(APIView):
	# 配置自定义限流
	permission_classes = [IsAuthenticated]
	throttle_scope = "vip"

	def get(self, request):
		return Response({'msg': 'ok'})
```
# DRF版本控制
## 为什么需要版本控制
我们都知道每一个程序都是有版本的。而且版本也会升级从v1升级到v2、v3、v4·····，但是我们不可能因为新版本出现旧版本就不去维护，因为用户有权选择不更新版本。所以我们就需要对版本进行控制，这个DRF也给我们提供了一些封装好的方法
## 流程
1. 当前端来请求时，执行了as_views()方法，如果设置了全局版本或者进入了设置了版本的功能函数，则会先执行APIView类中的dispatch方法，之后再执行initial方法（可以看到将版本信息version和版本控制方案scheme分别赋值给了request.version 和 request.determine_version），然后进入了self.determine_version方法（self.determine_version这个方法是找我们自己定义的版本控制类，没有的话就返回（None,None））
2. 里面会先判断是否有versioning_class，如果没有就返回(None,None)，就代表没有版本，如果有就执行versioning_class(URLPathVersioning)类中的determine_version方法,它会返回版本
3. 里面会判断如果获取到的version为空则返回默认版本，并且还要判断版本是否存在允许出现的版本列表中，返回版本之后，再把版本号和版本类分别赋值给request.version和request.versioning_scheme
![](1431882-20181214203351645-1586263720.png)
DRF框架提供的版本控制方法（rest_framework.versioning）
## 用法
### 基于url的get传参方式(/users?version=v1)
```
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',            # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],   # 允许的版本
    'VERSION_PARAM': 'version'          # URL中获取值的key
}
```
```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view(),name='test'),
]
```
```
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import QueryParameterVersioning


class TestView(APIView):
    versioning_class = QueryParameterVersioning

    def get(self, request, *args, **kwargs):

        # 获取版本
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)

        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```
### 基于url的正则方式(/v1/users/)
```
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',            # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],   # 允许的版本
    'VERSION_PARAM': 'version'          # URL中获取值的key
}
```
```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^(?P<version>[v1|v2]+)/test/', TestView.as_view(), name='test'),
]
```
```
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import URLPathVersioning


class TestView(APIView):
    versioning_class = URLPathVersioning

    def get(self, request, *args, **kwargs):
        # 获取版本
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)

        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```
### 基于accept 请求头方式(Accept: application/json; version=1.0)
```
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',            # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],   # 允许的版本
    'VERSION_PARAM': 'version'          # URL中获取值的key
}
```
```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view(), name='test'),
]

```
```
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import AcceptHeaderVersioning


class TestView(APIView):
    versioning_class = AcceptHeaderVersioning

    def get(self, request, *args, **kwargs):
        # 获取版本 HTTP_ACCEPT头
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)
        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```
### 基于主机名方法(v1.example.com)
```
ALLOWED_HOSTS = ['*']
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',  # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],  # 允许的版本
    'VERSION_PARAM': 'version'  # URL中获取值的key
}
```
```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^test/', TestView.as_view(), name='test'),
]

```
```
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import HostNameVersioning

class TestView(APIView):
    versioning_class = HostNameVersioning

    def get(self, request, *args, **kwargs):
        # 获取版本
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)
        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```
### 基于django路由系统的namespace(example.com/v1/users/）
```
REST_FRAMEWORK = {
    'DEFAULT_VERSION': 'v1',  # 默认版本
    'ALLOWED_VERSIONS': ['v1', 'v2'],  # 允许的版本
    'VERSION_PARAM': 'version'  # URL中获取值的key
}
```
```
from django.conf.urls import url, include
from web.views import TestView

urlpatterns = [
    url(r'^v1/', ([
                      url(r'test/', TestView.as_view(), name='test'),
                  ], None, 'v1')),
    url(r'^v2/', ([
                      url(r'test/', TestView.as_view(), name='test'),
                  ], None, 'v2')),

]
```
```
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.versioning import NamespaceVersioning


class TestView(APIView):
    versioning_class = NamespaceVersioning

    def get(self, request, *args, **kwargs):
        # 获取版本
        print(request.version)
        # 获取版本管理的类
        print(request.versioning_scheme)
        # 反向生成URL
        reverse_url = request.versioning_scheme.reverse('test', request=request)
        print(reverse_url)

        return Response('GET请求，响应内容')

    def post(self, request, *args, **kwargs):
        return Response('POST请求，响应内容')

    def put(self, request, *args, **kwargs):
        return Response('PUT请求，响应内容')
```
###  全局使用
```
REST_FRAMEWORK = {
    'DEFAULT_VERSIONING_CLASS':"rest_framework.versioning.URLPathVersioning",
    'DEFAULT_VERSION': 'v1',
    'ALLOWED_VERSIONS': ['v1', 'v2'],
    'VERSION_PARAM': 'version' 
}
```

## 自定义（继承内置类determine_version）
```
自定义一个版本控制类

class MyVersion(object):
    def determine_version(self, request, *args, **kwargs):
        version = request.query_params.get("version")
        if not version:
            version = 'v1'
        return version
```
```
全局使用

REST_FRAMEWORK = {
    # 这个是默认使用的版本控制类
    "DEFAULT_VERSIONING_CLASS": "utils.version.MyVersion",     # 这个版本控制类的路径。
    # 默认使用的的版本 
    'DEFAULT_VERSION': 'v1',
    # 允许使用的版本
    'ALLOWED_VERSIONS': ['v1','v2'],
    # 版本使用的参数名称
    'VERSION_PARAM': 'version'
}

```
```
局部使用

from rest_framework.views import APIView
from rest_framework.versioning import QueryParameterVersioning, URLPathVersioning

class UsersView(APIView):
    versioning_class = ParamVersion
    # versioning_class = QueryParameterVersioning
    # versioning_class = URLPathVersioning
    
    def get(self, request, *args, **kwargs):
        # 获取用户veison
        print(version)
        return HttpResponse('用户列表')
```
```
在url中写路由

urlpatterns = [
    url(r'^version/', include('Version_Demo.urls'))    # 这里用了路由分发  
]
urlpatterns = [
    url(r'^demo/', Version_Demo.as_view()),
]
```
# DRF解析器
## 原理
根据content-type，选择对应的解析器去解析request.body中的数据格式，并将其赋值到request.data中
## 流程
1. 进入了APIView类的dispatch方法
2. 通过request.data获取用户请求头
3. 根据用户请求头和 parser_classes = [JSONParser, FormParser] 中支持的请求头进行比较
```
D:/python3.6/Lib/site-packages/rest_framework/settings.py:33

DEFAULTS = {
    'DEFAULT_PARSER_CLASSES': [
        'rest_framework.parsers.JSONParser',
        'rest_framework.parsers.FormParser',
        'rest_framework.parsers.MultiPartParser'
		]
	}
```
## 示例
```
局部

from rest_framework.parsers import JSONParser, FormParser

class ParserView(APIView):
    parser_classes = [JSONParser]

    def post(self, request, *args, **kwargs):
        """
        允许用户发送JSON格式的数据
            a. content-type: application/json
            b. {"name": 'alex', "age": 18}
        :param request:
        :param args:
        :param kwargs:
        :return:
        """
        """
        1. 获取用户请求
        2. 获取用户请求体
        3. 根据用户请求头和 parser_classes = [JSONParser, FormParser] 中支持的请求头进行比较
        4. JSONParser对象中去请求体
        5. request.data
        """
        print(request.data)
        return HttpResponse('JSON测试')
```
```
全局

REST_FRAMEWORK = {
    # JSONParser: 表示只能解析 application/json 请求头
    # FormParser: 表示只能解析 application/x-www-form-urlencoded 请求头
	"DEFAULT_PARSER_CLASSES":  ["rest_framework.JSONParser", "rest_framework.FormParser"]
}
```

# 过滤Filtering
	- 对于列表数据可能需要根据字段进行过滤，我们可以通过添加django-fitlter扩展来增强支持
	```
	pip install django-filter
	```
	- 主应用settings.py配置
		```
		INSTALLED_APPS = [
			'django_filters',  # 需要注册应用，
		]
		
		REST_FRAMEWORK = {
			'DEFAULT_FILTER_BACKENDS': ('django_filters.rest_framework.DjangoFilterBackend',)
		}
		```
		```
		views.py文件
		
		from rest_framework.generics import ListCreateAPIView
		from school.models import Student
		from school.serializers import StudentModelSerializers


		class FilterAPIView(ListCreateAPIView):
			queryset = Student.objects.all()
			serializer_class = StudentModelSerializers
			filter_fields = ['name', 'age'] # 表示根据name和age字段筛选
		```
# 排序Ordering
	- REST framework提供了OrderingFilter过滤器来帮助我们快速指明数据按照指定字段进行排序
	- 在类视图中设置filter_backends，使用rest_framework.filters.OrderingFilter过滤器，REST framework会在请求的查询字符串参数中检查是否包含了ordering参数，如果包含了ordering参数，则按照ordering参数指明的排序字段对数据集进行排序。
	- 前端可以传递的ordering参数的可选字段值需要在ordering_fields中指明。
	- 全局配置
		```
		配置文件，settings.py
		REST_FRAMEWORK ={
			'DEFAULT_FILTER_BACKENDS':[
				'rest_framework.filters.OrderingFilter',
			]
		}
		```
		```
		from rest_framework.generics import ListCreateAPIView
		from school.models import Student
		from school.serializers import StudentModelSerializers

		class FilterAPIView(ListCreateAPIView):
			queryset = Student.objects.all()
			serializer_class = StudentModelSerializers
			order_fields = ['id', 'age']
		```
		```
		访问：http://127.0.0.1:8000/opt/filter/?ordering=-age
		```
	- 单独配置
		- 上面提到，因为过滤和排序公用了一个配置项，所以排序和过滤要一起使用，则必须整个项目，要么一起全局过滤排序，要么一起局部过滤排序。决不能出现，一个全局，一个局部的这种情况，局部会覆盖全局的配置。
		```
		from rest_framework.generics import ListCreateAPIView
		from rest_framework.filters import OrderingFilter
		from django_filters.rest_framework import DjangoFilterBackend
		from school.models import Student
		from school.serializers import StudentModelSerializers

		class FilterAPIView(ListCreateAPIView):
			queryset = Student.objects.all()
			serializer_class = StudentModelSerializers
			# 局部排序
			filter_backends = [OrderingFilter, DjangoFilterBackend]
			order_fields = ['id', 'age']

			# 局部过滤
			filter_fields = ['id', 'age']
		```
# 分页Pagination
我们数据库有几千万条数据，这些数据需要展示，我们不可能直接从数据库把数据全部读取出来，这样会给内存造成特别大的压力，有可能还会内存溢出，所以希望一点一点的取，然后通过分页展示，但是在数据量特别大的时候，我们的分页会越往后读取速度越慢，怎么能让我的查询速度变快？drf给我们提供了三种分页方式
## 步骤
1. 数据库查询
2. 调用分页器
3. 构建序列化器, 进行序列化操作, 返回数据	

## 使用	
### 全局配置
```
主应用settings.py

REST_FRAMEWORK = {
	'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',  # 页码分页
	# 'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination', # 偏移量分页
	'PAGE_SIZE': 10  # 每页数目
}
```
```
from rest_framework.generics import ListAPIView
from school.models import Student
from school.serializers import StudentModelSerializers

class PageAPIView(ListAPIView):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers
```
```
from rest_framework.generics import ListAPIView
from school.models import Student
from school.serializers import StudentModelSerializers

class PageAPIView(ListAPIView):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers
	pagination_class = None  # 关闭全局分页效果
```
### 局部分页
```
主应用settings.py

REST_FRAMEWORK = {
	# 'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',  # 页码分页
	# 'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination', # 偏移量分页
	'PAGE_SIZE': 10  # 每页数目
}
```
```
from rest_framework.generics import ListAPIView
from school.models import Student
from school.serializers import StudentModelSerializers
from rest_framework.pagination import PageNumberPagination, LimitOffsetPagination

class PageAPIView(ListAPIView):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers
	# 局部分页
	pagination_class = PageNumberPagination
```
### 自定义分页器
```
from rest_framework.generics import ListAPIView
from school.models import Student
from school.serializers import StudentModelSerializers
from rest_framework.pagination import PageNumberPagination, LimitOffsetPagination

# 自定义分页器
class **PagePagination**(PageNumberPagination):
	page_query_param = 'page'  # 代表页码的变量名
	page_size_query_param = 'size'  # 每一页数据的变量名
	page_size = 4  # 每一页的数据量
	max_page_size = 5  # 允许调整的每一页最大数量

class PageAPIView(ListAPIView):
	queryset = Student.objects.all()
	serializer_class = StudentModelSerializers
	pagination_class = **PagePagination**
```
# 异常处理 Exceptions
	- REST framework本身在APIView提供了异常处理，但是仅针对drf内部现有的接口开发相关的异常进行格式处理，但是开发中我们还会使用到各种的数据或者进行各种网络请求，这些都有可能导致出现异常，这些异常在中是没有进行处理的，所以就会抛给django框架了，django框架会进行组织错误信息，作为htm页面返回给客户端，所在在前后端分离项目中，可能JS无法理解或者无法接收到这种数据，甚至导致JS出现措误的情况。因此为了避免出现这种情况，我们可以自定义一个属于自己的异常处理函数，对于无法处理的异常，我们自己编写异常处理的代码逻辑。
	- 针对于现有的的异常处理进行额外添加属于开发者自己的逻辑代码，一般我们编写的异常处理函数，会写一个公共的目录下或者主应用目录下。这里主应用下直接创建一个excepitions.py
	```
	文件: exceptions.py
	from rest_framework.response import Response
	from rest_framework.views import exception_handler as drf_exception_handler
	from django.db import DatabaseError

	def exception_handler(exc, context):
		"""
		exc：本次发生异常的对象，对象
		context：本次发生一次时的上下文环境信息，字典。所谓的执行上下文[context]，就是程序执行到当前一行代码时，能提供给开发者调用的环境信息异常发生时，代码所在的路径，时间，视图，客户端http请求等等...]
		"""
		response = drf_exception_handler(exc, context)
		if response is None:
			if isinstance(exc, ZeroDivisionError):
				response = Response({'detail': '除数不能为0！'})
			if isinstance(exc, DatabaseError):
				response = Response({'detail': '数据库处理异常！'})
		return response
	```
	```
	主应用settings.py
	
	REST_FRAMEWORK = {
		'EXCEPTION_HANDLER': 'drfdemo.exceptions.exception_handler'
	}
	```
	```
	views.py
	
	class PageAPIView(APIView):
    def get(self, request):
        1 / 0
        return Response({'msg': 'ok'})
	```
	- REST framework定义的异常
		- APIException 所有异常的父类
		- ParseError 解析错误
		- AuthenticationFailed 认证失败
		- NotAuthenticated 尚未认证
		- PermissionDenied 权限决绝
		- NotFound 未找到
		- MethodNotAllowed 请求方式不支持
		- NotAcceptable 要获取的数据格式不支持
		- Throttled 超过限流次数
		- ValidationError 校验失败
	- 很多的没有在上面列出来的异常，就需要我们在自定义异常中自己处理
1. 自动生成接口文档
	```
	pip install coreapi
	```	
	```
	总路由urls.py
	
	from rest_framework.documentation import include_docs_urls
	urlpatterns = [
		path('docs/', include_docs_urls(title='接口文档API'))
	]
	```
	```
	settings.py
	
	INSTALLED_APPS = [
		'coreapi',
	]
		
	REST_FRAMEWORK = {
	    'DEFAULT_SCHEMA_CLASS': 'rest_framework.schemas.AutoSchema',
	}
	```
	- 页面访问：http://127.0.0.1:8000/docs/
	- 接口描述信息
		- 单一方法的视图，可直接使用类视图的文档字符串
		```
		class BookListView(generics.ListAPIView):
			"""
			返回所有图书信息.
			"""
		```
		- 包含多个方法的视图，在类视图的文档字符串中，分开方法定义
		```
		class BookListCreateView(generics.ListCreateAPIView):
		    """
		    get:
		    返回所有图书信息.
		
		    post:
		    新建图书.
		    """
		```
		- 对于视图集ViewSet，仍在类视图的文档字符串中封开定义，但是应使用action名称区分
		```
		class BookInfoViewSet(mixins.ListModelMixin, mixins.RetrieveModelMixin, GenericViewSet):
		    """
		    list:
		    返回图书列表数据
		
		    retrieve:
		    返回图书详情数据
		
		    latest:
		    返回最新的图书数据
		
		    read:
		    修改图书的阅读量
		    """
		```
		- 两点说明：
			- 视图集ViewSet中的retrieve名称，在接口文档网站中叫做read
			- 参数的Description需要在模型类或序列化器类的字段中以help_text选项定义
			```
			class Student(models.Model):
			    ...
			    age = models.IntegerField(default=0, verbose_name='年龄', help_text='年龄')
			    ...
			```
			```
			class StudentSerializer(serializers.ModelSerializer):
			    class Meta:
			        model = Student
			        fields = "__all__"
			        extra_kwargs = {
			            'age': {
			                'required': True,
			                'help_text': '年龄'
			            }
			        }
			```
	- 通过swagger展示
		```
		pip install drf-yasg
		```
		```
		settings.py文件
		
		INSTALLED_APPS = [
			'drf_yasg',
		]
		```
		```
		urls.py
		
		from drf_yasg.views import get_schema_view
		from drf_yasg import openapi
		from rest_framework.permissions import IsAuthenticated

		schema_view = get_schema_view(
			openapi.Info(
				title="Swagger接口文档",
				default_version='v1.0',  # 必传
				description="描述信息",
				terms_of_service='',
				contact=openapi.Contact(email="liuly2@knownsec.com"),
				license=openapi.License(name="协议版木")
			),
			public=True,  # 允许所有人都能访问
			permission_classes=(IsAuthenticated,)
		)

		urlpatterns = [
			path('sw/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger')
		]
		```