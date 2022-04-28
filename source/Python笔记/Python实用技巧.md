Python实用技巧
============================================
#### all() or any()
```
一般来说, 当我们有多个 and 条件时使用 all()，当我们有多个 or 条件时使用 any()
x = [True, True, False]
if any(x):
    print("至少有一个True")
if all(x):
    print("全是True")
if any(x) and not all(x):
    print("至少一个True和一个False")
```
#### 处理用户的多个输入
```
一般写法
n1 = input("enter a number : ")
n2 = input("enter a number : ")
n2 = input("enter a number : ")
print(n1, n2, n3)
```
```
更好的处理
n1, n2, n3 = input("enter a number : ").split()
print(n1, n2, n3)
```
#### 回文字符串判断
```
print("John Deo"[::-1])
```
```
v1 = "madam" 
v2 = "master" 
print(v1.find(v1[::-1]) == 0) # True
print(v1.find(v2[::-1]) == 0) # False
```
#### 尽量使用 Inline if statement
- 大多数情况下，我们在条件之后只有一个语句，因此使用Inline if statement 可以帮助我们编写更简洁的代码
```
一般写法
if name:
    print(name)
if name and age > 18:
    print("user is verified")
```
```
更好的处理
age > 18 and name and print("user is verified")
```
#### 找到list中重复最多的元素
- 在Python中可以使用 max( ) 函数并传递 list.count 作为key，即可找出列表list中重复次数最多的元素
```

lst = [1, 2, 3, 4, 3, 4, 4, 5, 6, 3, 1, 6, 7, 9, 4, 0]
most_repeated_item = max(lst, key=lst.count)
print(most_repeated_item)
```
#### bashplotlib
	- Bashplotlib 是一个 Python 库，他能够帮助我们在命令行(粗旷的环境)中绘制数据。
	```
	# 模块安装
	pip install bashplotlib
	# 绘制实例
	import numpy as np
	from bashplotlib.histpgram import plot_hist
	arr = np.ramdom.normal(size=1000, loc=0, scale=1)
	plot_hist(arr, bincount=50)
	```
#### collections
```
from collections import OrderedDict, Counter
# 记住键的添加顺序！
x = OrderedDict(a=1, b=2, c=3)
# 统计每个字符出现的频率
y = Counter("Hello World!")
```
```
import collections
print(collections.__all__)
```
```
namedtuple
我们知道tuple可以表示不变集合，例如，一个点的二维坐标就可以表示成p = (1, 2),但是，看到(1, 2)，很难看出这个tuple是用来表示一个坐标的,定义一个class又小题大做了
from collections import namedtuple
Point = namedtuple('point', ['x', 'y'])
p = Point(1, 2)
print(p.x)
namedtuple是一个函数，它用来创建一个自定义的tuple对象，并且规定了tuple元素的个数，并可以用属性而不是索引来引用tuple的某个元素
```
```
namedtuple使用list存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为list是线性存储，数据量大的时候，插入和删除效率很低
deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈
from collections import deque
q = deque(['b'])
q.appendleft('a')
q.append('c')
print(q)
deque除了实现list的append()和pop()外，还支持appendleft()和popleft()，这样就可以非常高效地往头部添加或删除元素
```
```
使用dict时，如果引用的Key不存在，就会抛出KeyError。如果希望key不存在时，返回一个默认值，就可以用defaultdict
from collections import defaultdict
d = defaultdict(lambda: 'default_value')
d['key'] = 'value'
print(d['key'])
print(d['not_exist'])
注意默认值是调用函数返回的，而函数在创建defaultdict对象时传入
```
```
python3.6之前使用dict时，Key是无序的。在对dict做迭代时，我们无法确定Key的顺序。如果要保持Key的顺序，可以用OrderedDict
from collections import OrderedDict
d = dict([('a', 1), ('b', 2), ('c', 3)])
od = OrderedDict([('a', 1), ('b', 2), ('c', 3)])
print(d)
print(od)
OrderedDict的Key会按照插入的顺序排列，不是Key本身排序
OrderedDict可以实现一个FIFO（先进先出）的dict，当容量超出限制时，先删除最早添加的Key
源码如下：
from collections import OrderedDict
class LastUpdatedOrderedDict(OrderedDict):

    def __init__(self, capacity):
        super(LastUpdatedOrderedDict, self).__init__()
        self._capacity = capacity

    def __setitem__(self, key, value):
        containsKey = 1 if key in self else 0
        if len(self) - containsKey >= self._capacity:
            last = self.popitem(last=False)
            print 'remove:', last
        if containsKey:
            del self[key]
            print 'set:', (key, value)
        else:
            print 'add:', (key, value)
        OrderedDict.__setitem__(self, key, value)
```
```
Counter是一个简单的计数器，例如，统计字符出现的个数
from collections import Counter
c = Counter()
tmp_str = 'fjdkasgkjqpejfkasfqjfkadsjgda'
for item in tmp_str:
    c[item] += 1
print(c)
```
```
合并2个字典
from collections import ChainMap
dict1 = {'name': 'jim', 'age': 21}
dict2 = {'high': 175, 'gender': '男'}
new_dict = ChainMap(dict1, dict2)
print(new_dict)
print(new_dict['name'])
print(new_dict['age'])
print(new_dict['high'])
```

#### dir
- 有没有想过如何查看 Python 对象内部并查看它具有哪些属性？在命令行中输入：
```
dir() 
dir("Hello World") 
dir(dir)
当以交互方式运行 Python 以及动态探索你正在使用的对象和模块时，这可能是一个非常有用的功能
```


#### emoji
- emoji[3] 是日本在无线通信中所使用的视觉情感符号，绘指图画，文字指的则是字符，可用来代表多种表情，如笑脸表示笑、蛋糕表示食物等。在中国大陆，emoji通常叫做“小黄脸”，或者直称emoji。
```
# 安装模块
pip install emoji
# 做个尝试
from emoji import emojize
print(emojize(":thumbs_up:"))
```


#### from  __future__  import
- Python 流行的结果之一，总是有新版本正在开发中。新版本意味着新功能 —— 除非你的版本已过时。
- 不过不要担心。使用该__future__模块[4]可以帮助你用Python的未来版本导入功能。从字面上看，这就像时间旅行、魔法或其他东西。
```
from __future__ import print_function
print("Hello World!")
```

#### geogy
- 地理，对大多数程序员来说是一个具有挑战性的领域。在获取地理信息或者绘制地图时，也会遇到不少问题。这个geopy 模块[5]让地理相关内容变得非常容易。
```
pip install geopy
它通过抽象一系列不同地理编码服务的 API 来工作。通过它，你能够获得一个地方的完整街道地址、纬度、经度甚至海拔高度。
还有一个有用的距离类。它以你偏好的测量单位计算两个位置之间的距离。
from geopy import GoogleV3
place = "221b Baker Street, London"
location = GoogleV3().geocode(place)
print(location.address)
print(location.location)
```
#### howdoi
- 当你使用terminal终端编程时，通过在遇到问题后会在StackOverflow上搜索答案，完后会回到终端继续编程，此时有时会不记得你之前查到的解决方案，此时需要重新查看StackOverflow，但又不想离开终端，那么此时你需要用到这个有用的命令行工具howdoi[6]。
```
pip install howdoi
无论你有什么问题，都可以问它，它会尽力回复。
howdoi vertical align css
howdoi for loop in java
howdoi undo commits in git
但请注意——它会从 StackOverflow 的最佳答案中抓取代码。它可能并不总是提供最有用的信息......
howdoi exit vim
```
#### inspect
- Python 的inspect模块[7]非常适合了解幕后发生的事情。你甚至可以调用它自己的方法！
- 下面的代码示例inspect.getsource() 用于打印自己的源代码。 inspect.getmodule() 还用于打印定义它的模块。
- 最后一行代码打印出它自己的行号。
```
import inspect
print(inspect.getsource(inspect.getsource))
print(inspect.getmodule(inspect.getmodule))
print(inspect.currentframe().f_lineno)
当然，除了这些微不足道的用途，inspect 模块可以证明对理解你的代码在做什么很有用。你还可以使用它来编写自文档化代码。
```

#### Jedi
- Jedi 库是一个自动完成和代码分析库。它使编写代码更快、更高效。
- 除非你正在开发自己的 IDE，否则你可能对使用Jedi [8]作为编辑器插件比较感兴趣。幸运的是，这已经有可用的负载！

#### **kwargs
- 在学习任何语言时，都会有许多里程碑。使用 Python 并理解神秘的**kwargs语法可能算作一个重要的里程碑。
- 字典对象前面的双星号**kwargs[9]允许你将该字典的内容作为命名参数传递给函数。
- 字典的键是参数名称，值是传递给函数的值。你甚至不需要调用它kwargs！
```
dictionary = {"a": 1, "b": 2}
def someFunction(a, b):
    print(a + b)
    return
# 这些做同样的事情:
someFunction(**dictionary)
someFunction(a=1, b=2)
当你想编写可以处理未预先定义的命名参数的函数时，这很有用。
```

#### 列表(list)推导式
- 关于 Python 编程，我最喜欢的事情之一是它的列表推导式[10]。
- 这些表达式可以很容易地编写非常顺畅的代码，几乎与自然语言一样。
```
numbers = [1,2,3,4,5,6,7]
evens = [x for x in numbers if x % 2 is 0]
odds = [y for y in numbers if y not in evens]
cities = ['London', 'Dublin', 'Oslo']

def visit(city):
    print("Welcome to "+city)
    
for city in cities:
    visit(city)
```
#### map
- Python 通过许多内置功能支持函数式编程。最有用的map()功能之一是函数——尤其是与lambda 函数[11]结合使用时。
```
x = [1, 2, 3] 
y = map(lambda x : x + 1, x)
# 打印出 [2,3,4]
print(list(y))
在上面的示例中，map()将一个简单的 lambda 函数应用于x. 它返回一个映射对象，该对象可以转换为一些可迭代对象，例如列表或元组。
```


#### newspaper3k
- 如果你还没有看过它，那么准备好被Python newspaper module [12]模块震撼到。它使你可以从一系列领先的国际出版物中检索新闻文章和相关的元数据。你可以检索图像、文本和作者姓名。它甚至有一些内置的 NLP 功能[13]。
- 因此，如果你正在考虑在下一个项目中使用 BeautifulSoup 或其他一些 DIY 网页抓取库，使用本模块可以为你自己节省不少时间和精力。
```
pip install newspaper3k
```
#### Operator overloading
- Python 提供对运算符重载的[14]支持，这是让你听起来像一个合法的计算机科学家的术语之一。
- 这实际上是一个简单的概念。有没有想过为什么 Python 允许你使用+运算符来添加数字以及连接字符串？这就是操作符重载的作用。
- 你可以定义以自己的特定方式使用 Python 的标准运算符符号的对象。并且你可以在与你正在使用的对象相关的上下文中使用它们。
```
class Thing:
    def __init__(self, value):
        self.__value = value
    def __gt__(self, other):
        return self.__value > other.__value
    def __lt__(self, other):
        return self.__value < other.__value
something = Thing(100)
nothing = Thing(0)
# True
something > nothing
# False
something < nothing
# Error
something + nothing
```
#### pprint
- Python 的默认print函数完成了它的工作。但是如果尝试使用print函数打印出任何大的嵌套对象，其结果相当难看。这个标准库的漂亮打印模块pprint[15]可以以易于阅读的格式打印出复杂的结构化对象。
- 这算是任何使用非平凡数据结构的 Python 开发人员的必备品。
```
import requests
import pprint
url = 'https://randomuser.me/api/?results=1'
users = requests.get(url).json()
pprint.pprint(users)
```
#### Queue
- Python 标准库的 Queue 模块实现支持多线程。这个模块让你实现队列数据结构。这些是允许你根据特定规则添加和检索条目的数据结构。
- “先进先出”（FIFO）队列让你可以按添加顺序检索对象。“后进先出”(LIFO) 队列让你可以首先访问最近添加的对象。
- 最后，优先队列让你可以根据对象的排序顺序检索对象。
- 这是一个如何在 Python 中使用队列Queue[16]进行多线程编程的示例。

#### __repr__
- 在 Python 中定义类或对象时，提供一种将该对象表示为字符串的“官方”方式很有用。例如：
```
file = open('file.txt', 'r') 
print(file) 
<open file 'file.txt', mode 'r' at 0x10d30aaf0>
这使得调试代码更加容易。将其添加到你的类定义中，如下所示：
class someClass: 
    def __repr__(self): 
        return "<some description here>"
someInstance = someClass()
# 打印 <some description here>
print(someInstance)
```
#### sh
- Python 是一种很棒的脚本语言。有时使用标准的 os 和 subprocess 库可能有点头疼。
- 该SH库[17]让你可以像调用普通函数一样调用任何程序——对于自动化工作流和任务非常有用。
```
import sh
sh.pwd()
sh.mkdir('new_folder')
sh.touch('new_file.txt')
sh.whoami()
sh.echo('This is great!')
```
#### Type hints
- Python 是一种动态类型语言。定义变量、函数、类等时不需要指定数据类型。这允许快速的开发时间。但是，没有什么比由简单的输入问题引起的运行时错误更烦人的了。
- 从 Python 3.5[18] 开始，你可以选择在定义函数时提供类型提示。
```
def addTwo(x : Int) -> Int:
    return x + 2
你还可以定义类型别名。

from typing import List
Vector = List[float]
Matrix = List[Vector]
def addMatrix(a : Matrix, b : Matrix) -> Matrix:
  result = []
  for i,row in enumerate(a):
    result_row =[]
    for j, col in enumerate(row):
      result_row += [a[i][j] + b[i][j]]
    result += [result_row]
  return result
x = [[1.0, 0.0], [0.0, 1.0]]
y = [[2.0, 1.0], [0.0, -2.0]]
z = addMatrix(x, y)
```
- 尽管不是强制性的，但类型注释可以使你的代码更易于理解。
- 它们还允许你使用类型检查工具，在运行前捕获那些杂散的 TypeError。如果你正在处理大型、复杂的项目，这是很有用的！

#### uuid
- 通过Python 标准库的 uuid 模块[19]生成通用唯一 ID（或“UUID”）的一种快速简便的方法。
```
import uuid
user_id = uuid.uuid4()
print(user_id)
```
- 这将创建一个随机的 128 位数字，该数字几乎肯定是唯一的。事实上，可以生成超过 2¹²² 种可能的 UUID。这超过了五个十进制 （或 5,000,000,000,000,000,000,000,000,000,000,000,000）。
- 在给定的集合中发现重复的概率极低。即使有一万亿个 UUID，重复存在的可能性也远低于十亿分之一。

#### Virtual environments
- 你可能同时在多个 Python 项目上工作。不幸的是，有时两个项目将依赖于相同依赖项的不同版本。你在你的系统上安装了什么?
- 幸运的是，Python支持对 虚拟环境[20] 的让你可以两全其美。从命令行：
```
python -m venv my-project 
source my-project/bin/activate 
pip install all-the-modules
```
- 现在，你可以在同一台机器上运行 Python 的独立版本和安装。

#### wikipedia
- 维基百科有一个很棒的 API，它允许用户以编程方式访问无与伦比的完全免费的知识和信息。在wikipedia模块[21]使访问该API非常方便。
```
import wikipedia
result = wikipedia.page('freeCodeCamp')
print(result.summary)
for link in result.links:
    print(link)
```
- 和真实站点一样，该模块提供了多语言支持、页面消歧、随机页面检索，甚至还有一个donate()方法。

#### xkcd
- 幽默是 Python 语言的一个关键特征，它是以英国喜剧小品剧Python飞行马戏团[22]命名的。Python 的许多官方文档都引用了该节目最著名的草图。不过，Python 的幽默并不仅限于文档。试试运行下面的行：
```
import antigravity
```
#### YAML
- YAML[23]指的是 “ 非标记语言” 。它是一种数据格式化语言，是 JSON 的超集。
- 与 JSON 不同，它可以存储更复杂的对象并引用它自己的元素。你还可以编写注释，使其特别适合编写配置文件。该PyYAML模块[24]可让你使用YAML使用Python。
- 安装并然后导入到你的项目中：
```
pip install pyyaml
import yaml
```
- PyYAML 允许你存储任何数据类型的 Python 对象，以及任何用户定义类的实例。
#### 将两个字典进行合并
```
d1 = {"v1": 22, "v2": 33}
d2 = {"v2": 44, "v3": 55}
d3 = {**d1, **d2}
print(d3)
```
#### zip
- 你曾经遇到过需要从两个列表中形成字典吗？
```
keys = ['a', 'b', 'c']
vals = [1, 2, 3]
zipped = dict(zip(keys, vals))
```
- 该zip()内置函数需要一系列可迭代的对象，并返回一个元组列表中。每个元组按位置索引对输入对象的元素进行分组。
- 你还可以通过调用对象来“解压缩”对象*zip()。
#### 字典按照value进行排序
```
d = {
    "v1": 80,
    "v2": 20,
    "v3": 40,
    "v4": 20,
    "v5": 10,
}
sorted_d = dict(sorted(d.items(), key=lambda item: item[1]))
print(sorted_d)
```
```
当然我们也可以使用itemgetter( )来替代上述 lambda函数,代码如下
from operator import itemgetter
sorted_d = dict(sorted(d.items(), key=itemgetter(1)))
```
#### 写在最后
- Python 是一种非常多样化且发展良好的语言，因此肯定会有许多我没有考虑的功能。如果你想了解更多的python模块，可以参考awesome-python[25]。

#### 参考资料
- [1] collections 模块: https://docs.python.org/3/library/collections.html
- [2] functions: https://docs.python.org/3/library/functions.html#dir
- [3] emoji: https://pypi.org/project/emoji/
- [4] __future__模块: https://docs.python.org/2/library/future.html
- [5] geopy 模块: https://geopy.readthedocs.io/en/latest/
- [6] howdoi: https://github.com/gleitz/howdoi
- [7] inspect模块: https://docs.python.org/3/library/inspect.html
- [8] Jedi : https://jedi.readthedocs.io/en/latest/docs/usage.html
- [9] **kwargs: https://docs.python.org/3/tutorial/controlflow.html#keyword-arguments
- [10] 列表推导式: https://docs.python.org/3/tutorial/datastructures.html#list-comprehensions
- [11] lambda 函数: https://docs.python.org/3/tutorial/controlflow.html#lambda-expressions
- [12] Python newspaper module : https://pypi.org/project/newspaper3k/
- [13] 内置的 NLP 功能: https://newspaper.readthedocs.io/en/latest/user_guide/quickstart.html#performing-nlp-on-an-article
- [14] 运算符重载的: https://docs.python.org/3/reference/datamodel.html#special-method-names
- [15] pprint: https://docs.python.org/3/library/pprint.html
- [16] Queue: https://www.tutorialspoint.com/python3/python_multithreading.htm
- [17] SH库: http://amoffat.github.io/sh/
- [18] Python 3.5: https://docs.python.org/3/library/typing.html
- [19] uuid 模块: https://docs.python.org/3/library/uuid.html
- [20] 虚拟环境: https://docs.python.org/3/tutorial/venv.html
- [21] wikipedia模块: https://wikipedia.readthedocs.io/en/latest/quickstart.html
- [22] Python飞行马戏团: https://en.wikipedia.org/wiki/Monty_Python's_Flying_Circus
- [23] YAML: http://yaml.org/
- [24] PyYAML模块: https://pyyaml.org/wiki/PyYAMLDocumentation
- [25] awesome-python: https://awesome-python.com/