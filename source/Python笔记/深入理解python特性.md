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
	- 编写“可调试”的装饰器	
	- 在使用装饰器时，实际上是使用一个函数替换另一个函数。这个过程的 一个缺点是“隐藏”了（未装饰）原函数所附带的一些元数据，如包装闭包隐藏了原函数的名称、文档字符串和参数列表
	- 这增加了调试程序和使用Python解释器的难度。有一个方法能避免这个问题：使用Python标准库中的functools.wraps装饰器
	- 在自己的装饰器中使用functools.wraps能够将丢失的元数据从被装饰的函数复制到装饰器闭包中
	```
	import functools 
	def uppercase(func): 
		@functools.wraps(func) 
		def wrapper(): 
			return func().upper() 
		return wrapper
	```
	- 将functools.wraps应用到由装饰器返回的封装闭包中，会获得原函数的文档字符串和其他元数据
	- 建议最好在自己编写的所有装饰器中都使用functools.wraps！！！
1. 有趣的*args和**kwargs
	```
	def foo(required, *args, **kwargs): 
		print(required) 
		if args: 
			print(args) 
		if kwargs: 
			print(kwargs)
	```
	- 上述函数至少需要一个名为required的参数，但也可以接受额外的位置参数和关键字参数。 
	- 如果用额外的参数调用该函数，args将收集额外的位置参数组成元组， 因为这个参数名称带有*前缀。 
	- 同样，kwargs会收集额外的关键字参数来组成字典，因为参数名称带 有**前缀。 
	- 如果不传递额外的参数，那么args和kwargs都为空
	- 参数args和kwargs只是一个命名约定。哪怕将 其命名为*parms和**argv，前面的例子也能正常工作。
	- 实际起作用的 语法分别是星号（*）和双星号（**），即解包操作符
1. 函数参数解包
	- *和**操作符有一个非常棒但有点神秘的功能，那就是用来从序列和字 典中“解包”函数参数
	```
	tmp_tuple = (1, 2, 3)
	print(*tmp_tuple)
	```
	```
	# tmp_tuple = [x for x in range(3)]
	# 这种技术适用于任何可迭代对象，包括生成器表达式
	tmp_tuple = (x for x in range(3))

	def func(a, b, c):
		print(a * a, b * b, c * c)

	func(*tmp_tuple)
	```
	```
	def func(a, b, c):
		print(a * a, b * b, c * c)
	从字典中解包关键字参数的**操作符
	由于字典是无序的，8因此解包时会匹配字典键和函数参数：x参数接受 字典中与'x'键相关联的值
	tmp_dict = {'a': 1, 'b': 2, 'c': 3}
	func(**tmp_dict)
	```
	```
	如果使用单个星号（*）操作符来解包字典，则所有的键将以随机顺序 传递给函数
	def func(a, b, c):
		print(a, b, c)

	tmp_dict = {'x': 1, 'y': 2, 'z': 3}
	func(*tmp_dict)
	```
1. 返回空值
	- Python在所有函数的末尾添加了隐式的return None语句。因此，如果 函数没有指定返回值，默认情况下会返回None
	```
	def foo1(value): 
		if value: 
			return value 
		else:
			return None 
	def foo2(value): 
	"""纯return语句，相当于`return None`""" 
		if value: 
			return value 
		else:
			return 
	def foo3(value): 
	"""无return语句，也相当于`return None`""" 
		if value: 
			return value
	```
#### 类与面向对象
1. 对象比较：is与==
	- ==操作符比较的是相等性
	- 而is操作符比较的是相同性
	- 当两个变量指向同一个对象时，is表达式的结果为True； 当各变量指向的对象含有相同内容时，==表达式的结果为True
1. 字符串转换（每个类都需要__repr__）
	- 在Python中定义一个自定义类之后，如果尝试在控制台中输出其实例或 在解释器会话中查看，并不能得到十分令人满意的结果
	- 是向类中添加双下划线方法 __str__和__repr__。这两个方法以具有Python特色的方式在不同情况下将对象转换为字符串
	```

	class Car:
		def __init__(self, color, mileage):
			self.color = color
			self.mileage = mileage

		def __str__(self):
			return f'a {self.color} car'


	if __name__ == '__main__':
		c = Car('red', 8888)
		print(c)
	```
	- _str__和__repr它们各自在实际使用中的差异
	```
	import datetime

	today = datetime.date.today()
	print(today)
	print(str(today)) # 2022-04-30
	print(repr(today)) # datetime.date(2022, 4, 30)
	__str__函数的结果侧重于可读性，旨在为人们返回一 个简洁的文本表示
	__repr__侧重的则是得到无歧义的结果，生成的字符串更多的是帮助 开发人员调试程序
	```
	- 为什么每个类都需要__repr__
	- 如果不提供__str__方法，Python在查找__str__时会回退到__repr__ 的结果。因此建议总是为自定义类添加__repr__方法
	```
	class Car:
		def __init__(self, color, mileage):
			self.color = color
			self.mileage = mileage

		def __repr__(self):
			return f'{self.__class__.__name__}({self.color!r},{self.mileage!r})'

	if __name__ == '__main__':
		c = Car('red', 8888)
		print(c)
	```
1. 定义自己的异常类
	- 可以清楚地显示出潜在的错误，让函 数和模块更具可维护性。自定义错误类型还可用来提供额外的调试信息
	```
	class NameTooShortError(ValueError): 
		pass 
	def validate(name): 
		if len(name) < 10: 
			raise NameTooShortError(name)
	```
	- NameTooShortError异常类型，它扩展自内置的ValueError类。一般情况下自定义异常都是派生自Exception这个异常基类或其他内置的Python异常，如ValueError或TypeError，取决于哪个更合适
1. 克隆对象
	- Python中的赋值语句不会创建对象的副本，而只是将名称绑定到对象上。对于不可变对象也是如此。 
	- 但为了处理可变对象或可变对象集合，需要一种方法来创建这些对象 的“真实副本”或“克隆体”。 
	- 从本质上讲，你有时需要用到对象的副本，以便修改副本时不会改动本体
	- Python的内置可变容 器，如列表、字典和集合，调用对应的工厂函数就能完成复制
	- new_list = list(original_list) 
	- new_dict = dict(original_dict) 
	- new_set = set(original_set)
	```
	tmp_list = [1]
	copy_list = list(tmp_list)
	print(tmp_list == copy_list)
	print(tmp_list is copy_list)
	```
	- 但用这种方法无法复制自定义对象，且最重要的是这种方法只创建浅副本
	- 浅复制是指构建一个新的容器对象，然后填充原对象中子对象的引用。 本质上浅复制只执行一层，复制过程不会递归，因此不会创建子对象的副本。 
	- 深复制是递归复制，首先构造一个新的容器对象，然后递归地填充原始 对象中子对象的副本。这种方式会遍历整个对象树，以此来创建原对象 及其所有子项的完全独立的副本
	- 使用copy模块中定义的deepcopy()函 数创建深副本
	```
	import copy

	tmp_list = [[1, 2]]
	copy_list = copy.deepcopy(tmp_list)
	copy_list[0][0] = 2
	print(copy_list)
	print(tmp_list)
	```
	- 顺便说一句，还可以使用copy模块中的一个函数来创建浅副本。copy.copy()函数会创建对象的浅副本
1. 用抽象基类避免继承错误
	- 抽象基类（abstract base class，ABC）用来确保派生类实现了基类中的 特定方法
	```
	from abc import ABCMeta, abstractmethod

	class Base(metaclass=ABCMeta):
		@abstractmethod
		def foo(self):
			pass

		@abstractmethod
		def bar(self):
			pass

	class Concrete(Base):
		def foo(self):
			pass

	assert issubclass(Concrete, Base)
	c = Concrete()
	print(c)
	```
	- 不用abc模块的话，如果缺失某个方法，则只有在实际调用这个方法时 才会抛出NotImplementedError
1. namedtuple的优点
	- 简单元组有一个缺点，那就是存储在其中的数据只能通过整数索引来访 问。无法给存储在元组中的单个属性赋予名称，因而代码的可读性不 高。
	- 元组是一种具有单例性质的数据结构，很难保证两个元组存有相 同数量的字段和相同的属性，因此很容易因为不同元组之间的字段顺序 不同而引入难以意识到的bug
	- namedtuple就是具有名称的元组。存储在其中的每个对象都可以 通过唯一的（人类可读的）标识符来访问。因此不必记住整数索引，也 无须采用其他变通方法，如将整数常量定义为索引的助记符
	```
	from collections import namedtuple

	car = namedtuple('Car', 'color,mileage')
	my_car = car('red', 56798)
	```
	- 将字符串'Car'作为第一个参数传递给 namedtuple工厂函数。 这个参数在Python文档中被称为typename，在调用namedtuple函数时 作为新创建的类名称
	- 由于namedtuple并不知道创建的类最后会赋给哪个变量，因此需要明 确告诉它需要使用的类名
	- namedtuple会自动生成文档字符串和 __repr__，其中的实现中会用到类名
	- 将字段作为'color mileage'这样的字符串整体传递？ 答案是namedtuple的工厂函数会对字段名称字符串调用split()，将其解析为字段名称列表
	```
	'color mileage'.split() 
	['color', 'mileage'] 
	Car = namedtuple('Car', ['color', 'mileage'])
	```
	- 除了通过标识符来访问存储在namedtuple中的值之外，索引访问仍然可用
	```
	print(my_car.color)
	print(my_car[1])
	```
	- 元组解包和用于函数参数解包的*操作符也能正常工作
	```
	color, mileage = my_car
	print(color, mileage)
	print(*my_car)
	```
	- 子类化namedtuple
	```
	Car = namedtuple('Car', 'color mileage') 
	class MyCarWithMethods(Car): 
		def hexcolor(self): 
			if self.color == 'red': 
				return '#ff0000' 
			else:
				return '#000000'
	 c = MyCarWithMethods('red', 1234)
	 c.hexcolor()
	```
	- 由于namedtuple内部的结构比较特殊，因此很难添加新的不可变 字段。另外，创建namedtuple类层次的最简单方法是使用基类元组的 _fields属性
	```
	Car = namedtuple('Car', 'color mileage') 
	ElectricCar = namedtuple('ElectricCar', Car._fields + ('charge',))
	ElectricCar('red', 1234, 45.0)
	```
	- 内置的辅助方法
		- _asdict()辅助方法开始，该方法将namedtuple的内容以字典形式返回
		```
		print(my_car._asdict())
		print(json.dumps(my_car._asdict()))
		```
		- 另一个有用的辅助函数是_replace()。该方法用于创建一个元组的浅副本，并能够选择替换其中的一些字段
		```
		print(my_car._replace(color='blue'))
		```
		- 最后介绍的是_make()类方法，用来从序列或迭代对象中创建 namedtuple的新实例
		```
		print(my_car._make(['orange', '6666']))
		```
1. 类变量与实例变量的陷阱
	- 类变量在类定义内部声明（但位于实例方法之外），不受任何特定类实 例的束缚。类变量将其内容存储在类本身中，从特定类创建的所有对象 都可以访问同一组类变量。这意味着修改类变量会同时影响所有对象实 例。
	- 实例变量总是绑定到特定的对象实例。它的内容不存储在类上，而是存 储在每个由类创建的单个对象上。因此实例变量的内容与每个对象实例 相关，修改实例变量只会影响对应的对象实例
	```
	class Dog:
		legs = 4  # 类变量

		def __init__(self, name):
			self.name = name  # 实例变量

	xh = Dog('小灰')
	xb = Dog('小白')
	print(xh.name)
	print(xb.name)
	print(xh.legs)
	print(xb.legs)
	print(Dog.name)  # AttributeError: type object 'Dog' has no attribute 'name'
	print(Dog.legs)
	```
	- 虽然得到了想要的结果，但在实例中引入了一个legs实例变量。而新的legs实例变量“遮盖”了相同名称的类变量，在访问对象实例作用域时覆盖并隐藏类变量
	```
	xh.legs = 6
	print(xh.legs, xb.legs, Dog.legs)
	print(xh.__class__.legs)
	```
	- 一个示例
	```
	class CountedObject: 
		num_instances = 0 
		def __init__(self): 
			self.__class__.num_instances += 1
	```
	- 每次创建此类的新实例时，会运行__init__构造函数并将共享计数器 递增1
	- 下面是错误的案例：
	```
	class CountedObject: 
		num_instances = 0 
		def __init__(self): 
			self.num_instances += 1
	```