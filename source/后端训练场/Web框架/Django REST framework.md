Django REST framework
==========================
#### Web应用模式
1. 在开发Web应用中，有两种应用模式
	- 前后端不分离【客户端看到的内容和所有界面效果都是由服务端提供】
	- 前后端分离【把前端的界面效果html，css，js分离到另一个服务端，python服务端只需要返回数据即可】
#### 环境安装与配置
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
#### APIView视图
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
#### 序列化器-Serializer
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

#### 视图
1. 视图中调用Http请求和响应处理类
	- 什么时候声阴的序列化器需要继承序列化器基类Serializer,什么时候继承模型序列化器类ModelSerializer?
		- 继承序列化器类Serializer
			- 字段声明
			- 验证
			- 添加/保存数据功能
		- 继承模型序列化器类Modelseria1izer
			- 字段声明[可近，看黹要]
			- Meta声明
			- 验证
			- 添加/保存数据功能[可选]
	- 看数据是否从mysql数据库中获取，如果是则使用ModelSerializer,不是则使用Serializer
	- **http请求响应**
		- drf除了在数据序列化部分简写代码以外,还在视图中提供了简写操作,所以在django有的django.views.View基础上,d倒封装了多个视图子类出来提供给我们使用。
		- drf提供的视图的主要作用:
			- 控制序列化器的执行(检验、保存、转换数据)
			- 控制数据库查询的执行
			- 调用请求类和响应类（这两个类也是由drf帮我们再次扩展了一些功能类)
	- 请求与响应
		- 内容协商:drf在django原有的基础上,新增了—个request对象继承到了APIView视图类,并在django原有的HttpResponse响应类的基础上实现了一个子类rest_framework.response.Response响应类。这两个类,都是基于内容协商来完成数据的格式转换的。
		- request->parser解析类->识别客户端请求头中的Content-Type完成数据转换成>类字典(QueryDict,字典的子类)
		- response->renderer渲染类->识别客户端请求头的Accept来提取客户端期望返回的数据格式,->转换成客户端的期望格式数据。如果客户端没有申明Accept，则按照Content-Type格式进行转换
		- **Request**
		- REST framework传入视图的requests对象不再是Django默认的HttpRequest象,而是REST framework提供的扩展了HttpRequest类的Requests类的对象。
		- REST framework提供了Parser解析器,在接收到请求后会自动根据Content-Type指明的请求数据类型(如JSON.表单等)将请求数据进行parse解析,解析为类字典[QueryDict]对象保存到Request对象中。
		- Request对象的数据是自动根据前端发送数据的格式进行解析之后的结果,
		- 无论前端发送的哪种格式的数据,我们都可以以统一的方式读取数据
		- 基本属性
			- .data
				- request.data返回解析之后的请求体数据。类似于Django标准的 request.Post和request.FILES属性,但提供如下特性:
					- 包含了解析之后的文件和非文件数据
					- 包含了对POST、PUT、PATCH请求方式解析后的数据
					- 利用了REST framework的parsers解析器,不仅支持表单类型数据,也支持JSON据
			- .query_params（查询参数、查询字符串）
				- request.query_params与Django标准的request.Get相同,只是更换了更正确的名称而已,
			- request._request
				- 获取django封装的Request对象
		- **Response**
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
2. 视图作用
	- 控制序列化器的执行（检验、保存、转换数据）
	- 控制数据库模型的操作
1. 普通视图
	- 两个视图基类
		- APIView基本视图类
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
		GenericAPIView通用视图类
	- 五个视图扩展类
	- 九个视图子类
3. 视图集ViewSet
#### 路由Routers
1. REST framework提供了两个router
	- SimpleRouter
	- DefaultRouter
1. 使用方法
2. 视图集中附加action的声明
3. 路由router形成URL的方式