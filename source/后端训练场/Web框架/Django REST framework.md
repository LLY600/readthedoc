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
		- GenericAPIView通用视图类
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
			pagination_class指明分页控制类
			filter_backends指明数据过滤控制后端
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
			
	- 五个视图扩展类
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
	- 九个视图子类
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
3. 视图集ViewSet
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
#### 路由Routers
- 对于视图集ViewSet,我们除了可以自己手动指明请求方式与动作action之间的对应关系外，还可以使用Routers来帮助我们快速实现路由信息。如果是非视图集，不需要使用路由集routers
- REST framework提供了两个router类,使用方式一致的。结果多一个或少一个根目录url地址的问题而已，用任何一个都可以。
	- SimpleRouter
	- DefaultRouter
1. 使用方法
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
	# base_name 路由别名的前缀
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
2. 视图集中附加action的声明
```
action装饰器可以接收两个参数：
	methods: 声明该action对应的请求方式，列表传递
	detail: 声明该action的路径是否与单一资源对应
路由前缀/<pk>/action方法名/
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
#### 相关功能组件
1. 认证
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
		"""
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
		```
		```
		file:views.py
		
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
				'drfdemo.authentication.CustomAuthentication',          # 自定义认证
				'rest_framework.authentication.SessionAuthentication',  # session认证
				'rest_framework.authentication.BasicAuthentication',    # 基本认证
			)
		}
		```
		
1. 权限
	- 权限控制可以限制用户对于视图的访问和对于具有模型对象的访问。
		- 在执行视图的as_view()方法的dispatch()方法前，会先进行视图访问权限的判断
		- 在通过get_object()获取具体模型对象时，会进行模型对象访问权限的判断
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
	- 自定义权限
	- 如需自定义权限，需继承rest_framework.permissions.BasePermission父类，并实现以下两个任何一个方法或全部
		- has_permission(self, request, view)
		- 是否可以访问视图， view表示当前视图对象
		- has_object_permission(self, request, view, obj)
		- 是否可以访问模型对象， view表示当前视图， obj为模型数据对象
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
		file：views.py
		
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
			#     'rest_framework.permissions.IsAuthenticated',
			# ]
		}
		```

1. 限流
	- 可以对接口访问的频次进行限制，以减轻服务器压力，或者实现特定的业务。
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
1. 过滤Filtering
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
1. 排序Ordering
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
1. 分页Pagination
	- django默认提供的分页器主要使用于前后端不分离的业务场景，所以REST framework也提供了分页的支持
	- 全局配置
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
	- 局部分页
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
	- 自定义分页器
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
1. 异常处理 Exceptions
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