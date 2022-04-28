深入理解python特性
============================================
#### assert断言
1. 不要使用断言验证数据，因为命令行中使用-O 和-OO标识，或修改CPython中的PYTHONOPTIMIZE环境变量，都会全局禁用断言
	```
	#temp_test.py
	assert 1 == 2, '断言失败'
	```
	```
	>python36 temp_test.py
	>python36 -O temp_test.py
	```
1. 永不失败的断言
	```
	assert(1 == 2, 'This should fail')
	这是因为在Python中非空元组总为真值
	```
1. 注意事项
	- Python断言语句是一种测试某个条件的调试辅助功能，可作为程序的内部自检。 
	- 断言应该只用于帮助开发人员识别bug，它不是用于处理运行时错误的机制。 
	- 设置解释器可全局禁用断言
#### 巧妙地放置逗号
1. 字符串字面值拼接
	```
	name = [
		'lily',
		'tom'
		'tim'
	]
	print(name)
	```
	```
	tmp_str = (
		'abc'
		'def'
		)
	print(tmp_str)
	```
#### 上下文管理器和with语句
1. 简化一些通用资源管理模式，抽象出其中的功能，将其分解并重用
	```
	with open('hello.txt', 'w') as f: 
		f.write('hello, world!')
	```
	```
	threading.Lock类是Python标准库中另一个比较好的示例
	some_lock = threading.Lock() 
	
	# 有问题: 
	some_lock.acquire() 
	try:
		# 执行某些操作…… 
	finally: 
		some_lock.release() 
		
	# 改进版: 
	with some_lock: 
		#执行某些操作
	```
1. 在自定义对象中支持with
	- 只要实现所谓的上下文管理器 （context manager），就可以在自定义的类和函数中获得相同的功能
	- 上下文管理器是什么？这是一个简单的“协议”（或接口），自定义对象 需要遵循这个接口来支持with语句。总的来说，如果想将一个对象作为 上下文管理器，需要做的就是向其中添加__enter__和__exit__方 法
	- 基于类的上下文管理器
	```
	class ManagerFile:
    def __init__(self, name):
        self.name = name

    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()

	with ManagerFile('hello.txt') as f:
		f.write('hello word')
	```
	- 基于生成器的上下文管理器
	```
	from contextlib import contextmanager

	@contextmanager
	def manage_file(name):
		try:
			f = open(name, 'w')
			yield f
		finally:
			f.close()

	with manage_file('hello.txt') as f:
		f.write('hello word')
	```
#### 下划线、双下划线及其他
1. 前置单下划线：_var 
	- 前置单下划线只有约定含义，前置下划线的意思是提示其他程序员，以单下划线开头的变量或方法只在内部使用
	- 如果使用通配符导入从这个模块中导入所有名称，Python不会导 入带有前置单下划线的名称（除非模块中定义了__all__列表覆盖了这 个行为）
	```
	# my_module.py： 
	def external_func(): 
		return 23 
	def _internal_func(): 
		return 42
	 from my_module import * 
	 external_func() 23
	 _internal_func() NameError: "name '_internal_func' is not defined"
	```
	- 应避免使用通配符导入，因为这样就不清楚当前名称空间 中存在哪些名称了。为了清楚起见，最好坚持使用常规导入方法。与通配符导入不同，常规导入不受前置单下划线命名约定的影响
	```
	import my_module
	my_module.external_func() 23
	my_module._internal_func() 42
	```
2. 后置单下划线：var_ 
	- 某个变量最合适的名称已被Python语言中的关键字占用。因此，诸如class或def的名称不能用作Python中的变量名。在这种情况下，可以追加一个下划线来绕过命名冲突
3. 前置双下划线：__var 
	- 双下划线前缀会让Python解释器重写属性名称，以避免子类中的命名冲突
	```
	class TestClass:
		def __init__(self):
			self.__name = 'QQ'


	class ExtendTestClass(TestClass):
		def __init__(self):
			super(ExtendTestClass, self).__init__()
			self.__name = 'qq'


	tc = TestClass()
	print(dir(tc))

	etc = ExtendTestClass()
	print(dir(etc))
	print(etc.__name)
	名称改写也适用于方法名
	```
	```
	一个神奇的示例
	_ManageGlobal__val = 25

	class ManageGlobal:
		def test(self):
			return __val

	print(ManageGlobal().test())
	```
4. 前后双下划线：__var__ 
	- 双下划线方法通常被称为魔法方法
	- 双下划线方法是Python的核心功能，应根据需要使 用，其中并没有什么神奇或晦涩的内容
5. 单下划线：_
	- 表示变量是临时的或无关紧要的
	```
	我将元组解包为单独的变量，但其中只关注color字段的值。可是为了执行解包表达式就必须为元组中的所有值都分配变量，此时_用作占位符变量
	car = ('red', 'auto', 12, 3812.4)
	color, _, _, _ = car
	print(color)
	```
	- _在大多数Python REPL中是一个特殊变量，表 示由解释器计算的上一个表达式的结果
	```
	>>> 20 + 3 
	23
	>>> _ 
	23
	>>> print(_) 
	23
	```
#### 字符串格式化
1. “旧式”字符串格式化
	```
	 'Hello, %s' % name
	 ```
	 ```
	  'Hey %s, there is a % error!' % (name, errno)
	```
1. “新式”字符串格式化
	```
	 'Hello, {}'.format(name)
	```
	```
	'Hey {name}, there is a 0x{errno:x} error!'.format(name=name, errno=errno)
	```
1. 字符串字面值插值
	```
	f'Hello, {name}!'
	```
1. 模板字符串
	```
	from string import Template

	t = Template('this is a ua: $content')
	print(t.substitute(content='{Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Safari/537.36 Edg/100.0.1185.50}'))
	```
	- 最佳使用场景是用来处理程序用户生成的格式字符串，避免特殊输入出现安全问题
#### 函数
1. 函数是对象
	- 指向函数的变量和函数本身彼此独立
	```
	def func():
		print('call')

	f = func
	del func
	f()
	```
1. 函数可存储在数据结构中
	```
	list = [func,'test']
	```
1. 函数可传递给其他函数
	- 能接受其他函数作为参数的函数被称为高阶函数
	- Python中具有代表性的高阶函数是内置的map函数。map接受一个函数对 象和一个可迭代对象，然后在可迭代对象中的每个元素上调用该函数来生成结果
	```
	def func(num):
		return num * num

	print(list(map(func, [1, 2, 3])))
	```
1. 函数可以嵌套
	```
	def outer(text):
		def inner(input_value):
			return input_value.lower()
		return inner(text)

	print(outer('HELLO'))
	```
1. 函数可捕捉局部状态
	```
	def get_speak_func(text, volume):
		def whisper():
			return text.lower() + '...'

		def yell():
			return text.upper() + '!'

		if volume > 0.5:
			return yell
		else:
			return whisper
	仔细看看内部函数whisper和yell，注意其中并没有text参数。但不 知何故，内部函数仍然可以访问在父函数中定义的text参数。它们似乎 捕捉并“记住”了这个参数的值
	拥有这种行为的函数被称为词法闭包（lexical closure），简称闭包
	```
1. lambda表达式
	```
	add = lambda x, y: x + y
	print(add(1, 2))
	```
	```
	print((lambda x, y: x + y)(3, 4))
	```
	- 因此，lambda能方便灵活地快速定义Python函数。我一般在对可迭代对 象进行排序时，使用lambda表达式定义简短的key函数
	```
	tuples = [(1, 'd'), (2, 'b'), (4, 'a'), (3, 'c')]
	print(sorted(tuples, key=lambda x: x[0]))
	```
	```
	print(sorted(range(-5, 6), key=lambda x: x * x))
	```
	- 前面展示的两个示例在Python内部都有更简洁的实现，分别 是operator.itemgetter()和abs()函数
	- 如果想使用lambda表达式，那么请花几秒（或几分钟）思考，为了获得你所期望的结果，这种方式是否真的最简洁且最易维护
	- 将lambda和map()或filter()结合起来构建复杂的表达式也很难让人理解
	```
	# 有害： 
	>>> list(filter(lambda x: x % 2 == 0, range(16))) 
	[0, 2, 4, 6, 8, 10, 12, 14] 
	# 清晰： 
	>>> [x for x in range(16) if x % 2 == 0] 
	[0, 2, 4, 6, 8, 10, 12, 14]
	```
1. 装饰器
	- 对于理解装饰器来说，“头 等函数”中最重要的特性有： 
		- 函数是对象，可以分配给变量并传递给其他函数，以及从其他函数返回； 
		- 在函数内部也能定义函数，且子函数可以捕获父函数的局部状态（词法闭包）。
	- 最简单的装饰器
	```
	def null_decorator(func): 
		return func
	```
	- 使用Python的@语法能够更方便地修饰函数
	```
	@null_decorator 
	def greet(): 
		return 'Hello!'
	```
	- 装饰器可以修改行为
	```
	def uppercase(func): 
		def wrapper(): 
			original_result = func() 
			modified_result = original_result.upper() 
			return modified_result 
		return wrapper
	```
	- 将多个装饰器应用于一个函数
	```
	def strong(func): 
		def wrapper(): 
			return '<strong>' + func() + '</strong>' 
		return wrapper 
		
	def emphasis(func): 
		def wrapper(): 
			return '<em>' + func() + '</em>' 
		return wrapper
	
	@strong 
	@emphasis 
	def greet(): 
		return 'Hello!'
		
	'<strong><em>Hello!</em></strong>'
	从结果中能清楚地看出装饰器应用的顺序是从下向上
	```
	- 装饰接受参数的函数
	```
	def proxy(func): 
		def wrapper(*args, **kwargs): 
			return func(*args, **kwargs) 
		return wrapper
	```