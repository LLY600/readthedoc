#### 数据库
1. ORM框架
	- O是object，也就类对象的意思，R是relation，翻译成中文是关系，也就是关系数据库中数据表的意思，M是mapping，是映射的意思。在ORM框架中，它帮我们把类和数据表进行了一个映射，可以让我们通过类和类对象就能操作它所对应的表格中的数据。ORM框架还有一个功能，它可以根据我们设计的类自动帮我们生成数据库中的表格，省去了我们自己建表的过程。
	- django中内嵌了ORM框架，不需要直接面向数据库编程，而是定义模型类，通过模型类和对象完成数据表的增删改查操作。
	- 使用django进行数据库开发的步骤如下：
		- 配置数据库连接信息
		- 在models.py中定义模型类
		- 迁移
		- 通过类和对象完成数据增删改查操作
1. 配置
	- 在settings.py中保存了数据库的连接配置信息，Django默认初始配置使用sqlite数据库
	- 使用MySQL数据库首先需要安装驱动程序
	```
	pip install PyMySQL
	```
	- 在Django的工程同名子目录的__init__.py文件中添加如下语句
	```
	from pymysql import install_as_MySQLdb
	install_as_MySQLdb()
	作用是让Django的ORM能以mysqldb的方式来调用PyMySQ
	```
	- 修改DATABASES配置信息
	```
	DATABASES = {
	    'default': {
	        'ENGINE': 'django.db.backends.mysql',
	        'HOST': '127.0.0.1',  # 数据库主机
	        'PORT': 3306,  # 数据库端口
	        'USER': 'root',  # 数据库用户名
	        'PASSWORD': 'mysql',  # 数据库用户密码
	        'NAME': 'django_demo'  # 数据库名字
	    }
	}
	```
	- 在MySQL中创建数据库
	```
	create database django_demo default charset=utf8;
	```
1. 定义模型类
	- 模型类被定义在"应用/models.py"文件中。
	- 模型类必须继承自Model类，位于包django.db.models中。
	- 定义
	```
	创建应用booktest，在models.py 文件中定义模型类
	
	from django.db import models
	#定义图书模型类BookInfo
	class BookInfo(models.Model):
	    btitle = models.CharField(max_length=20, verbose_name='名称')
	    bpub_date = models.DateField(verbose_name='发布日期')
	    bread = models.IntegerField(default=0, verbose_name='阅读量')
	    bcomment = models.IntegerField(default=0, verbose_name='评论量')
	    is_delete = models.BooleanField(default=False, verbose_name='逻辑删除')
	    class Meta:
	        db_table = 'tb_books'  # 指明数据库表名
	        verbose_name = '图书'  # 在admin站点中显示的名称
	        verbose_name_plural = verbose_name  # 显示的复数名称
	    def __str__(self):
	        """定义每个数据对象的显示信息"""
	        return self.btitle
	#定义英雄模型类HeroInfo
	class HeroInfo(models.Model):
	    GENDER_CHOICES = (
	        (0, 'female'),
	        (1, 'male')
	    )
	    hname = models.CharField(max_length=20, verbose_name='名称') 
	    hgender = models.SmallIntegerField(choices=GENDER_CHOICES, default=0, verbose_name='性别')  
	    hcomment = models.CharField(max_length=200, null=True, verbose_name='描述信息') 
	    hbook = models.ForeignKey(BookInfo, on_delete=models.CASCADE, verbose_name='图书')  # 外键
	    is_delete = models.BooleanField(default=False, verbose_name='逻辑删除')
	    class Meta:
	        db_table = 'tb_heros'
	        verbose_name = '英雄'
	        verbose_name_plural = verbose_name
	    def __str__(self):
	        return self.hname
	```
	- 数据库表名
		- 模型类如果未指明表名，Django默认以 小写app应用名_小写模型类名 为数据库表名。
		- 可通过db_table 指明数据库表名。
	- 关于主键
		- django会为表创建自动增长的主键列，每个模型只能有一个主键列，如果使用选项设置某属性为主键列后django不会再创建自动增长的主键列。
		- 默认创建的主键列属性为id，可以使用pk代替，pk全拼为primary key。
	- 属性命名限制
		- 不能是python的保留关键字。
		- 不允许使用连续的下划线，这是由django的查询方式决定的。
		- 定义属性时需要指定字段类型，通过字段类型的参数指定选项，语法如下：
		```
		属性=models.字段类型(选项)
		```
	- 字段类型
	```
	|AutoField	|自动增长的IntegerField，通常不用指定，不指定时Django会自动创建属性名为id的自动增长属性|
	|BooleanField	|布尔字段，值为True或False|
	|NullBooleanField	|支持Null、True、False三种值|
	|CharField	|字符串，参数max_length表示最大字符个数|
	|TextField	|大文本字段，一般超过4000个字符时使用|
	|IntegerField	|整数|
	|DecimalField	|十进制浮点数， 参数max_digits表示总位数， 参数decimal_places表示小数位数|
	|FloatField	|浮点数|
	|DateField	|日期， 参数auto_now表示每次保存对象时，自动设置该字段为当前时间，用于"最后一次修改"的时间戳，它总是使用当前日期，默认为False； 参数auto_now_add表示当对象第一次被创建时自动设置当前时间，用于创建的时间戳，它总是使用当前日期，默认为False; 参数auto_now_add和auto_now是相互排斥的，组合将会发生错误|
	|TimeField	|时间，参数同DateField|
	|DateTimeField	|日期时间，参数同DateField|
	|FileField	|上传文件字段|
	|ImageField	|继承于FileField，对上传的内容进行校验，确保是有效的图片|
	```
	- 选项
	```
	|null		|如果为True，表示允许为空，默认值是False											|
	|blank		|如果为True，则该字段允许为空白，默认值是False										|
	|db_column	|字段的名称，如果未指定，则使用属性的名称											|
	|db_index	|若值为True, 则在表中会为此字段创建索引，默认值是False								|
	|default	|默认																				|
	|primary_key|若为True，则该字段会成为模型的主键字段，默认值是False，一般作为AutoField的选项使用	|
	|unique		|如果为True, 这个字段在表中必须有唯一值，默认值是False								|
	null是数据库范畴的概念，blank是表单验证范畴的
	```
	- 外键
	- 在设置外键时，需要通过on_delete选项指明主表删除数据时，对于外键引用表数据如何处理，在django.db.models中包含了可选常量：
		- CASCADE 级联，删除主表数据时连通一起删除外键表中数据
		- PROTECT 保护，通过抛出ProtectedError异常，来阻止删除主表中被外键应用的数据
		- SET_NULL 设置为NULL，仅在该字段null=True允许为null时可用
		- SET_DEFAULT 设置为默认值，仅在该字段设置了默认值时可用
		- SET() 设置为特定值或者调用特定方法，如
		```
		from django.conf import settings
		from django.contrib.auth import get_user_model
		from django.db import models
		
		def get_sentinel_user():
		    return get_user_model().objects.get_or_create(username='deleted')[0]
		
		class MyModel(models.Model):
		    user = models.ForeignKey(
		        settings.AUTH_USER_MODEL,
		        on_delete=models.SET(get_sentinel_user),
		    )
		```
		- DO_NOTHING 不做任何操作，如果数据库前置指明级联性，此选项会抛出IntegrityError异常
1. 迁移
	- 将模型类同步到数据库中
	- 生成迁移文件
	```
	python manage.py makemigrations
	```
	- 同步到数据库中
	```
	python manage.py migrate
	```
1. 添加测试数据
	```
	insert into tb_books(btitle,bpub_date,bread,bcomment,is_delete) values
	('射雕英雄传','1980-5-1',12,34,0),
	('天龙八部','1986-7-24',36,40,0),
	('笑傲江湖','1995-12-24',20,80,0),
	('雪山飞狐','1987-11-11',58,24,0);
	```
	```
	insert into tb_heros(hname,hgender,hbook_id,hcomment,is_delete) values
	('郭靖',1,1,'降龙十八掌',0),
	('黄蓉',0,1,'打狗棍法',0),
	('黄药师',1,1,'弹指神通',0),
	('欧阳锋',1,1,'蛤蟆功',0),
	('梅超风',0,1,'九阴白骨爪',0),
	('乔峰',1,2,'降龙十八掌',0),
	('段誉',1,2,'六脉神剑',0),
	('虚竹',1,2,'天山六阳掌',0),
	('王语嫣',0,2,'神仙姐姐',0),
	('令狐冲',1,3,'独孤九剑',0),
	('任盈盈',0,3,'弹琴',0),
	('岳不群',1,3,'华山剑法',0),
	('东方不败',0,3,'葵花宝典',0),
	('胡斐',1,4,'胡家刀法',0),
	('苗若兰',0,4,'黄衣',0),
	('程灵素',0,4,'医术',0),
	('袁紫衣',0,4,'六合拳',0);
	```
1. 演示工具使用
	- Django的manage工具提供了shell命令，帮助我们配置好当前工程的运行环境（如连接好数据库等），以便可以直接在终端中执行测试python语句
	- 通过如下命令进入shell
	```
	python manage.py shell
	```
	- 导入两个模型类，以便后续使用
	```
	from booktest.models import BookInfo, HeroInfo
	```
1. 查看MySQL数据库日志
	- 查看mysql数据库日志可以查看对数据库的操作记录。 mysql日志文件默认没有产生，需要做如下配置
	```
	sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
	将general_log_file和general_log两处注释取消
	sudo service mysql restart
	tail -f /var/log/mysql/mysql.log
	```
1. 数据库操作
	- 增加
		- save
			```
			from datetime import date
			book = BookInfo(
				btitle='西游记',
				bpub_date=date(1988,1,1),
				bread=10,
				bcomment=10
			)
			book.save()
			hero = HeroInfo(
				hname='孙悟空',
				hgender=0,
				hbook=book
			)
			hero.save()
			hero2 = HeroInfo(
				hname='猪八戒',
				hgender=0,
				hbook_id=book.id
			)
			hero2.save()
			```
		- create
			```
			模型类.objects.create()
			
			HeroInfo.objects.create(
			    hname='沙悟净',
			    hgender=0,
			    hbook=book
			)
			```
	- 删除
		- 模型类对象delete
		```
		hero = HeroInfo.objects.get(id=13)
		hero.delete()
		```
		- 模型类.objects.filter().delete()
		```
		HeroInfo.objects.filter(id=14).delete()
		```
	- 修改
		- save
		```
		修改模型类对象的属性，然后执行save()方法
		
		hero = HeroInfo.objects.get(hname='猪八戒')
		hero.hname = '猪悟能'
		hero.save()
		```
		- update
		```
		使用模型类.objects.filter().update()，会返回受影响的行数
		
		HeroInfo.objects.filter(hname='沙悟净').update(hname='沙僧')
		```
	- 查询
		- 基本查询
			- get 查询单一结果，如果不存在会抛出模型类.DoesNotExist异常。
			- all 查询多个结果。
			- count 查询结果数量。
			```
			 BookInfo.objects.all()
			<QuerySet [<BookInfo: 射雕英雄传>, <BookInfo: 天龙八部>, <BookInfo: 笑傲江湖>, <BookInfo: 雪山飞狐>, <BookInfo: 西游记>]>
			book = BookInfo.objects.get(btitle='西游记')
			book.id
			BookInfo.objects.get(id=3)
			BookInfo.objects.get(pk=3)
			BookInfo.objects.get(id=100) #报错，没那么多数据		
			BookInfo.objects.count()
			```
		- 过滤查询
			- filter 过滤出多个结果
			- exclude 排除掉符合条件剩下的结果
			- get 过滤单一结果
			- 对于过滤条件的使用，上述三个方法相同，故仅以filter进行讲解
			```
			语法：
			属性名称__比较运算符=值
			# 属性名称和比较运算符间使用两个下划线，所以属性名不能包括多个下划线
			```
			- 相等
				- exact：表示判等
				```
				BookInfo.objects.filter(id__exact=1)
				可简写为：
				BookInfo.objects.filter(id=1)
				```
			- 模糊查询
				- contains：是否包含，如果要包含%无需转义，直接写即可
				```
				查询书名包含'传'的图书。
				
				BookInfo.objects.filter(btitle__contains='传')
				```
				- startswith、endswith：以指定值开头或结尾
				```
				查询书名以'部'结尾的图书
				
				BookInfo.objects.filter(btitle__endswith='部')
				```
				- 以上运算符都区分大小写，在这些运算符前加上i表示不区分大小写，如iexact、icontains、istartswith、iendswith
			- 空查询
				- isnull：是否为null
				```
				查询书名不为空的图书。
				
				BookInfo.objects.filter(btitle__isnull=False)
				```
			 - 范围查询
				- in：是否包含在范围内。
				```
				例：查询编号为1或3或5的图书
				BookInfo.objects.filter(id__in=[1, 3, 5])
				```
			- 比较查询
				- gt 大于 (greater then)
				- gte 大于等于 (greater then equal)
				- lt 小于 (less then)
				- lte 小于等于 (less then equal)
				```
				查询编号大于3的图书
				
				BookInfo.objects.filter(id__gt=3)
				```
				- 不等于的运算符，使用exclude()过滤器
				```
				查询编号不等于3的图书
				
				BookInfo.objects.exclude(id=3)
				```
			- 日期查询
				- year、month、day、week_day、hour、minute、second：对日期时间类型的属性进行运算
				```
				查询1980年发表的图书。
				BookInfo.objects.filter(bpub_date__year=1980)
				查询1980年1月1日后发表的图书。
				BookInfo.objects.filter(bpub_date__gt=date(1990, 1, 1))
				```
		- F对象
			- 之前的查询都是对象的属性与常量值比较，两个属性怎么比较呢？ 答：使用F对象，被定义在django.db.models中。
			```
			语法如下：
			F(属性名)
			```
			```
			查询阅读量大于等于评论量的图书。
			from django.db.models import F
			BookInfo.objects.filter(bread__gte=F('bcomment'))
			```
			```
			可以在F对象上使用算数运算。
			例：查询阅读量大于2倍评论量的图书。
			BookInfo.objects.filter(bread__gt=F('bcomment') * 2)
			```
		- Q对象
			- 多个过滤器逐个调用表示逻辑与关系，同sql语句中where部分的and关键字
			```
			查询阅读量大于20，并且编号小于3的图书。
			BookInfo.objects.filter(bread__gt=20,id__lt=3)
			或
			BookInfo.objects.filter(bread__gt=20).filter(id__lt=3)
			```
			- 如果需要实现逻辑或or的查询，需要使用Q()对象结合|运算符，Q对象被义在django.db.models
			```
			语法如下：
			Q(属性名__运算符=值)
			```
			```
			查询阅读量大于20的图书，改写为Q对象如下。
			from django.db.models import Q
			BookInfo.objects.filter(Q(bread__gt=20))
			```
			- Q对象可以使用&、|连接，&表示逻辑与，|表示逻辑或。
			```
			查询阅读量大于20，或编号小于3的图书，只能使用Q对象实现
			BookInfo.objects.filter(Q(bread__gt=20) | Q(pk__lt=3))
			```
			- Q对象前可以使用~操作符，表示非not。
			```
			查询编号不等于3的图书。
			BookInfo.objects.filter(~Q(pk=3))
			```
		- 聚合函数
			- 使用aggregate()过滤器调用聚合函数。聚合函数包括：Avg 平均，Count 数量，Max 最大，Min 最小，Sum 求和，被定义在django.db.models中
			```
			查询图书的总阅读量。
			from django.db.models import Sum
			BookInfo.objects.aggregate(Sum('bread'))
			注意aggregate的返回值是一个字典类型，格式如下：
			{'属性名__聚合类小写':值}
			如:{'bread__sum':3}
			
			使用count时一般不使用aggregate()过滤器
			查询图书总数。
			BookInfo.objects.count()
			注意count函数的返回值是一个数字。
			```
		- 排序
			```
			使用order_by对结果进行排序
			
			BookInfo.objects.all().order_by('bread')  # 升序
			BookInfo.objects.all().order_by('-bread')  # 降序
			```
		- 关联查询
			- 由一到多的访问语法
			```
			一对多的模型类对象.多对应的模型类名小写_set 例：
			b = BookInfo.objects.get(id=1)
			b.heroinfo_set.all()
			```
			- 由多到一的访问语法
			```
			多对一的模型类对象.多对应的模型类中的关系类属性名 例：
			h = HeroInfo.objects.get(id=1)
			h.hbook
			```
			- 访问一对应的模型类关联对象的id语法
			```
			多对应的模型类对象.关联类属性_id
			h = HeroInfo.objects.get(id=1)
			h.hbook_id
			```
		- 关联过滤查询
			- 由多模型类条件查询一模型类数据:
			```
			语法如下：
			关联模型类名小写__属性名__条件运算符=值
			```
			- 注意：如果没有"__运算符"部分，表示等于。
			```
			例：查询图书，要求图书英雄为"孙悟空"
			BookInfo.objects.filter(heroinfo__hname='孙悟空')
			查询图书，要求图书中英雄的描述包含"八"
			BookInfo.objects.filter(heroinfo__hcomment__contains='八')
			```
			- 由一模型类条件查询多模型类数据:
			```
			语法如下：
			一模型类关联属性名__一模型类属性名__条件运算符=值
			```
			- 注意：如果没有"__运算符"部分，表示等于。
			```
			例：查询书名为“天龙八部”的所有英雄。
			HeroInfo.objects.filter(hbook__btitle='天龙八部')
			查询图书阅读量大于30的所有英雄
			HeroInfo.objects.filter(hbook__bread__gt=30)
			```
1. 查询集 QuerySet
	- 概念
		- Django的ORM中存在查询集的概念。
		- 查询集，也称查询结果集、QuerySet，表示从数据库中获取的对象集合。
		- 当调用如下过滤器方法时，Django会返回查询集（而不是简单的列表）：
			- all()：返回所有数据。
			- filter()：返回满足条件的数据。
			- exclude()：返回满足条件之外的数据。
			- order_by()：对结果进行排序。
		- 对查询集可以再次调用过滤器进行过滤，如
		```
		BookInfo.objects.filter(bread__gt=30).order_by('bpub_date')
		```
		- 也就意味着查询集可以含有零个、一个或多个过滤器。过滤器基于所给的参数限制查询的结果。
		- 从SQL的角度讲，查询集与select语句等价，过滤器像where、limit、order by子句。
		- 判断某一个查询集中是否有数据：
		- exists()：判断查询集中是否有数据，如果有则返回True，没有则返回False。
	- 两大特性
		- 惰性执行
			- 创建查询集不会访问数据库，直到调用数据时，才会访问数据库，调用数据的情况包括迭代、序列化、与if合用
			```
			例如，当执行如下语句时，并未进行数据库查询，只是创建了一个查询集qs
			qs = BookInfo.objects.all()
			继续执行遍历迭代操作后，才真正的进行了数据库的查询
			for book in qs:
				print(book.btitle)
			```
		- 缓存
			- 使用同一个查询集，第一次使用时会发生数据库的查询，然后Django会把结果缓存下来，再次使用这个查询集时会使用缓存的数据，减少了数据库的查询次数。
			```
			情况一：如下是两个查询集，无法重用缓存，每次查询都会与数据库进行一次交互，增加了数据库的负载。
			from booktest.models import BookInfo
			[book.id for book in BookInfo.objects.all()]
			[book.id for book in BookInfo.objects.all()]
			```
			```
			情况二：经过存储后，可以重用查询集，第二次使用缓存中的数据。
			
			qs=BookInfo.objects.all()
			[book.id for book in qs]
			[book.id for book in qs]
			```
	- 限制查询集
		- 可以对查询集进行取下标或切片操作，等同于sql中的limit和offset子句。
		- 注意：不支持负数索引。
		- 对查询集进行切片后返回一个新的查询集，不会立即执行查询。
		- 如果获取一个对象，直接使用[0]，等同于[0:1].get()，但是如果没有数据，[0]引发IndexError异常，[0:1].get()如果没有数据引发DoesNotExist异常。
		```
		示例：获取第1、2项，运行查看。
		qs = BookInfo.objects.all()[0:2]
		```
2. 管理器Manager
	- 管理器是Django的模型进行数据库操作的接口，Django应用的每个模型类都拥有至少一个管理器。
	- 我们在通过模型类的objects属性提供的方法操作数据库时，即是在使用一个管理器对象objects。当没有为模型类定义管理器时，Django会为每一个模型类生成一个名为objects的管理器，它是models.Manager类的对象。
	- 我们可以自定义管理器，并应用到我们的模型类上。
	- 注意：一旦为模型类指明自定义的过滤器后，Django不再生成默认管理对象objects。
	- 自定义管理器类主要用于两种情况：
		- 修改原始查询集，重写all()方法。
			```
			打开booktest/models.py文件，定义类BookInfoManager
			
			#图书管理器
			class BookInfoManager(models.Manager):
				def all(self):
					#默认查询未删除的图书信息
					#调用父类的成员语法为：super().方法名
					return super().filter(is_delete=False)
			```
			```
			在模型类BookInfo中定义管理器
			
			class BookInfo(models.Model):
				...
				books = BookInfoManager()
			```
			```
			使用方法
			
			BookInfo.books.all()
			```
		- 在管理器类中补充定义新的方法
			```
			打开booktest/models.py文件，定义方法create。
			class BookInfoManager(models.Manager):
			    #创建模型类，接收参数为属性赋值
			    def create_book(self, title, pub_date):
			        #创建模型类对象self.model可以获得模型类
			        book = self.model()
			        book.btitle = title
			        book.bpub_date = pub_date
			        book.bread=0
			        book.bcommet=0
			        book.is_delete = False
			        # 将数据插入进数据表
			        book.save()
			        return book
			```
			```
			为模型类BookInfo定义管理器books语法如下
			class BookInfo(models.Model):
			      ...
			    books = BookInfoManager()
			```
			```
			调用语法如下：
			book=BookInfo.books.create_book("abc",date(1980,1,1))
			```
#### admin站点
1. 使用Admin站点
	- 假设我们要设计一个新闻网站，我们需要编写展示给用户的页面，网页上展示的新闻信息是从哪里来的呢？是从数据库中查找到新闻的信息，然后把它展示在页面上。但是我们的网站上的新闻每天都要更新，这就意味着对数据库的增、删、改、查操作，那么我们需要每天写sql语句操作数据库吗? 如果这样的话，是不是非常繁琐，所以我们可以设计一个页面，通过对这个页面的操作来实现对新闻数据库的增删改查操作。那么问题来了，老板说我们需要在建立一个新网站，是不是还要设计一个页面来实现对新网站数据库的增删改查操作，但是这样的页面具有一个很大的重复性，那有没有一种方法能够让我们很快的生成管理数据库表的页面呢？有，那就是我们接下来要给大家讲的Django的后台管理。Django能够根据定义的模型类自动地生成管理页面。
	- 管理界面本地化
		```
		在settings.py中设置语言和时区
		LANGUAGE_CODE = 'zh-hans' # 使用中国语言
		TIME_ZONE = 'Asia/Shanghai' # 使用中国上海时间
		```
	- 创建管理员
		```
		创建管理员的命令如下，按提示输入用户名、邮箱、密码。
		python manage.py createsuperuser
		打开浏览器，在地址栏中输入如下地址后回车。
		http://127.0.0.1:8000/admin/
		输入前面创建的用户名、密码完成登录。
		如果想要修改密码可以执行
		python manage.py changepassword 用户名
		```
	- App应用配置
		- 在每个应用目录中都包含了apps.py文件，用于保存该应用的相关信息。
		- 在创建应用时，Django会向apps.py文件中写入一个该应用的配置类，如
			```
			from django.apps import AppConfig
			class BooktestConfig(AppConfig):
			    name = 'booktest'
			```
		- 我们将此类添加到工程settings.py中的INSTALLED_APPS列表中，表明注册安装具备此配置属性的应用
			- AppConfig.name 属性表示这个配置类是加载到哪个应用的，每个配置类必须包含此属性，默认自动生成。
			- AppConfig.verbose_name 属性用于设置该应用的直观可读的名字，此名字在Django提供的Admin管理站点中会显示，如
			```
			from django.apps import AppConfig
			
			class BooktestConfig(AppConfig):
			    name = 'booktest'
			    verbose_name = '图书管理'
			```
	- 注册模型类
		- 登录后台管理后，默认没有我们创建的应用中定义的模型类，需要在自己应用中的admin.py文件中注册，才可以在后台管理中看到，并进行增删改查操作。
		```
		打开booktest/admin.py文件，编写如下代码：
		
		from django.contrib import admin
		from booktest.models import BookInfo,HeroInfo
		admin.site.register(BookInfo)
		admin.site.register(HeroInfo)
		```
	- 调整站点信息
		- admin.site.site_header 设置网站页头
		- admin.site.site_title 设置页面标题
		- admin.site.index_title 设置首页标语
		```
		在booktest/admin.py文件中添加一下信息
		from django.contrib import admin
		admin.site.site_header = '传智书城'
		admin.site.site_title = '传智书城MIS'
		admin.site.index_title = '欢迎使用传智书城MIS'
		```
1. 调整列表页展示
	- 调整列表页和编辑页必须先---定义与使用Admin管理类
	- Django提供的Admin站点的展示效果可以通过自定义ModelAdmin类来进行控制
	```
	定义管理类需要继承自admin.ModelAdmin类，如下
	
	from django.contrib import admin
	
	class BookInfoAdmin(admin.ModelAdmin):
	    pass
	```
	```
	使用管理类有两种方式：
	注册参数
	admin.site.register(BookInfo,BookInfoAdmin)
	装饰器
	@admin.register(BookInfo)
	class BookInfoAdmin(admin.ModelAdmin):
	    pass
	```
	
	- 列表中的列显示哪些字段
	```
	属性如下：
	list_display=[模型字段1,模型字段2,...]
	打开booktest/admin.py文件，修改BookInfoAdmin类如下：
	class BookInfoAdmin(admin.ModelAdmin):
	    ...
	    list_display = ['id','btitle']
	```
	
	- 页大小
	```
	每页中显示多少条数据，默认为每页显示100条数据，属性如下：
	list_per_page=100
	打开booktest/admin.py文件，修改AreaAdmin类如下：
	class BookInfoAdmin(admin.ModelAdmin):
	    list_per_page = 2
	```
	
	- "操作选项"的位置
	- 顶部显示的属性，设置为True在顶部显示，设置为False不在顶部显示，默认为True。
	- actions_on_top=True
	- 底部显示的属性，设置为True在底部显示，设置为False不在底部显示，默认为False。
	- actions_on_bottom=False
	```
	打开booktest/admin.py文件，修改BookInfoAdmin类如下：
	
	class BookInfoAdmin(admin.ModelAdmin):
	    ...
	    actions_on_top = True
	    actions_on_bottom = True
	```
	
	- 右侧栏过滤器
	- 属性如下，只能接收字段，会将对应字段的值列出来，用于快速过滤。一般用于有重复值的字段。
	```
	list_filter=[]
	打开booktest/admin.py文件，修改HeroInfoAdmin类如下：
	class HeroInfoAdmin(admin.ModelAdmin):
		...
		list_filter = ['hbook', 'hgender']
	```
	
	- 搜索框
	- 属性如下，用于对指定字段的值进行搜索，支持模糊查询。列表类型，表示在这些字段上进行搜索。
	- search_fields=[]
	```
	打开booktest/admin.py文件，修改HeroInfoAdmin类如下：
	class HeroInfoAdmin(admin.ModelAdmin):
	    ...
	    search_fields = ['hname']
	```
	
	- 将方法作为列
	- 列可以是模型字段，还可以是模型方法，要求方法有返回值。
	- 通过设置short_description属性，可以设置在admin站点中显示的列名
	```
	打开booktest/models.py文件，修改BookInfo类如下：
	
	class BookInfo(models.Model):
	    ...
	    def pub_date(self):
	        return self.bpub_date.strftime('%Y年%m月%d日')
	
	    pub_date.short_description = '发布日期'  # 设置方法字段在admin中显示的标题
	```
	```
	打开booktest/admin.py文件，修改BookInfoAdmin类如下：
	
	class BookInfoAdmin(admin.ModelAdmin):
	    ...
	    list_display = ['id','atitle','pub_date']
	```
	- 方法列是不能排序的，如果需要排序需要为方法指定排序依据。
	```
	admin_order_field=模型类字段
	打开booktest/models.py文件，修改BookInfo类如下：
	class BookInfo(models.Model):
	    ...
	    def pub_date(self):
	        return self.bpub_date.strftime('%Y年%m月%d日')
	    pub_date.short_description = '发布日期'
	    pub_date.admin_order_field = 'bpub_date'
	```
	
	- 关联对象
	- 无法直接访问关联对象的属性或方法，可以在模型类中封装方法，访问关联对象的成员
	```
	打开booktest/models.py文件，修改HeroInfo类如下：
	
	class HeroInfo(models.Model):
	    ...
	    def read(self):
	        return self.hbook.bread
	
	    read.short_description = '图书阅读量'
	```
	```
	打开booktest/admin.py文件，修改HeroInfoAdmin类如下：
	
	class HeroInfoAdmin(admin.ModelAdmin):
	    ...
	    list_display = ['id', 'hname', 'hbook', 'read']
	```
1. 调整编辑页展示
	- 显示字段
	```
	属性如下：
	fields=[]
	
	打开booktest/admin.py文件，修改BookInfoAdmin类如下：
	class BookInfoAdmin(admin.ModelAdmin):
	    ...
	    fields = ['btitle', 'bpub_date']
	```
	
	- 分组显示
	```
	属性如下：
	fieldsets=(
	    ('组1标题',{'fields':('字段1','字段2')}),
	    ('组2标题',{'fields':('字段3','字段4')}),
	)
	```
	```
	打开booktest/admin.py文件，修改BookInfoAdmin类如下：
	class BookInfoAdmin(admin.ModelAdmin):
	    ...
	    # fields = ['btitle', 'bpub_date']
	    fieldsets = (
	        ('基本', {'fields': ['btitle', 'bpub_date']}),
	        ('高级', {
	            'fields': ['bread', 'bcomment'],
	            'classes': ('collapse',)  # 是否折叠显示
	        })
	    )
	说明：fields与fieldsets两者选一使用。
	```
	
	- 关联对象
	- 在一对多的关系中，可以在一端的编辑页面中编辑多端的对象，嵌入多端对象的方式包括表格、块两种。
		- 类型InlineModelAdmin：表示在模型的编辑页面嵌入关联模型的编辑。
		- 子类TabularInline：以表格的形式嵌入。
		- 子类StackedInline：以块的形式嵌入。
	```
	打开booktest/admin.py文件，创建HeroInfoStackInline类。
	
	class HeroInfoStackInline(admin.StackedInline):
	    model = HeroInfo  # 要编辑的对象
	    extra = 1  # 附加编辑的数量
	```
	```
	打开booktest/admin.py文件，修改BookInfoAdmin类如下：
	
	class BookInfoAdmin(admin.ModelAdmin):
	    ...
	    inlines = [HeroInfoStackInline]
	```
	
	- 可以用表格的形式嵌入。
	```
	打开booktest/admin.py文件，创建HeroInfoTabularInline类。
	
	class HeroInfoTabularInline(admin.TabularInline):
	    model = HeroInfo
	    extra = 1
	```
	```
	打开booktest/admin.py文件，修改BookInfoAdmin类如下：
	
	class BookInfoAdmin(admin.ModelAdmin):
	    ...
	    inlines = [HeroInfoTabularInline]
	```
1. 上传图片
	- Django有提供文件系统支持，在Admin站点中可以轻松上传图片。
	- 使用Admin站点保存图片，需要安装Python的图片操作包
	```
	pip install Pillow
	```
	- 配置
		- 默认情况下，Django会将上传的图片保存在本地服务器上，需要配置保存的路径。
		- 我们可以将上传的文件保存在静态文件目录中，如我们之前设置的static_files目录中在settings.py 文件中添加如下上传保存目录信息
		```
		MEDIA_ROOT=os.path.join(BASE_DIR,"static_files/media")
		```
	- 为模型类添加ImageField字段
		```
		我们为之前的BookInfo模型类添加一个ImageFiled
		
		class BookInfo(models.Model):
		    ...
		    image = models.ImageField(upload_to='booktest', verbose_name='图片', null=True)
		```
		```
		upload_to 选项指明该字段的图片保存在MEDIA_ROOT目录中的哪个子目录
		进行数据库迁移操作
		
		python manage.py makemigrations
		python manage.py migrate
		```
	- 使用Admin站点上传图片
		- 进入Admin站点的图书管理页面，选择一个图书，能发现多出来一个上传图片的字段
	