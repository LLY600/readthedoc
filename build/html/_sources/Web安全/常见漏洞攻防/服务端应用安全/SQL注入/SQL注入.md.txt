SQL注入
==========================================
#### 原理
- 是发生在应用程序和数据库层的安全漏洞
- 网站内部直接发送的SQL请求一般不会有危险，但实际情况是很多时候需要结合用户的输入数据动态的构造SQL语句，如果用户输入的数据被构造成恶意的SQL代码，Web应用又未对动态构造的SQL语句使用的参数进行审查，则会带来意想不到的危险
- 原理示意：
	- select id,name from table where id = ?
	- ?表示用户的输入，假如用户输入的是：1 or 1 = 1
	- 实际执行的SQL语句为select id,name from table where id = 1 or 1 = 1
	- 因为使用了or的运算符，这个where条件成立
	- 也就简化成了select id,name from table，获取到了所有的数据，进行了越权的查询

#### 分类
**从协议角度**
1. GET类型注入
	- 修改url中的一个字段
	
2. POST类型注入
	- 修改所发送的body中的字段
	- 使用一些代理工具，如burp suite等去修改回放请求

**从漏洞利用角度**
1. 布尔型注入
	- 做查询的时候联合一个布尔类型的判断
	- http://xxx.com/project?id=1 and substring(version(),1,1)=5
	- 如果返回id=1的数据，就表示数据库版本为5.x；反之。。。
	
4. 联合查询注入
	- 使用union的字段，能快速查找到我们想要查询的内容
	- http://xxx.com/project?id=1 union all select schema_name from information_schema.schemata
	
7. 时间延迟注入
	- 如果没有接受到服务器给的错误返回，类似union这种类型注入无法使用，考虑时间延迟注入
	- select * from table where id =1 and sleep(3)
	- 意思是让语句执行之后，等待3秒，才会返回查询的数据，就可以以时间为感知，了解代码是否被执行。主要利用场景，是页面没有给错误信息返回，无法使用union等注入类型

10. 报错型注入
	- DBMS数据库有一些内置函数，会有一些问题，利用这些问题，让服务器返回带有额外信息的报错，从侧面查询我们想要查询的数据
	- 利用group by的duplicate entry错误
	
13. 多语句查询注入
	- 联合多语句的一个查询，在执行一个普通查询的基础上穿插一个自己构造的独立的SQL语句
	- http://xxx.com/project?id=1;update table set name = 'xx' where id =1
	- 非常危险，意味着能对数据库做更新操作

