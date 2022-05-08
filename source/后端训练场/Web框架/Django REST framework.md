Django REST framework
==========================
#### 环境安装与配置
1. 环境准备
	```
	安装：Django、Drf、PyMysql
	创建项目：django-admin startproject drfdemo
	配置参数如：runserver 127.0.0.1:8000
	点击运行
	```

1. 添加Drf应用
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
	- pycharm一键选择、修改多个变量快捷键：Ctrl + Shift + Alt + j
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
	- Drf模式
		- 在students子应用目录中新建serializers.py用于保存该应用的序列化器。
		- 创建一个StudentModelSerializer用于序列化与反序列化。
		```
		from rest_framework import serializers
		from students_origin.models import Student

		class StudentModelSerializers(serializers.ModelSerializer):
			class Meta:
				model = Student
				fields = "__all__"
		```
		- 在views.py定义视图
		```
		from rest_framework.viewsets import ModelViewSet
		from students_origin.models import Student
		from .serializers import StudentModelSerializers

		class StudentModelViewSet(ModelViewSet):
		queryset = Student.objects.all()
		serializer_class = StudentModelSerializers
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
		页面访问(http://127.0.0.1:8000/api/)
		

