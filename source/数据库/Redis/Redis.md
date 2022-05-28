Redis
=========================================
# 引言
## 背景
- 在Web应用发展的初期，那时关系型数据库受到了较为广泛的关注和应用，原因是因为那时候Web站点基本上访问和并发不高、交互也较少。而在后来，随着访问量的提升，使用关系型数据库的Web站点多多少少都开始在性能上出现了一些瓶颈，而瓶颈的源头一般是在磁盘的I/O上。而随着互联网技术的进一步发展，各种类型的应用层出不穷，这导致在当今云计算、大数据盛行的时代，对性能有了更多的需求，主要体现在以下四个方面：
	- 低延迟的读写速度：应用快速地反应能极大地提升用户的满意度
	- 支撑海量的数据和流量：对于搜索这样大型应用而言，需要利用PB级别的数据和能应对百万级的流量
	- 大规模集群的管理：系统管理员希望分布式应用能更简单的部署和管理
	- 庞大运营成本的考量：IT部门希望在硬件成本、软件成本和人力成本能够有大幅度地降低
- 为了克服这一问题，NoSQL应运而生，它同时具备了**高性能**、**可扩展性强**、**高可用**等优点，受到广泛开发人员和仓库管理人员的青睐。

## 关系型数据库（RMDBS）与非关系型数据库（NoSQL）的对比：
- 数据库中表与表的数据之间存在某种关联的内在关系，因为这种关系，所以我们称这种数据库为关系型数据库。典型：Mysql/MariaDB、postgreSQL、Oracle、SQLServer、DB2、Access、SQLlite3。特点：
	- 全部使用SQL（结构化查询语言）进行数据库操作。
	- 都存在主外键关系，表，等等关系特征。
	- 大部分都支持各种关系型的数据库的特性：事务、存储过程、触发器、视图、临时表、模式、函数
- NOSQL：not only sql，泛指非关系型数据库。泛指那些不使用SQL语句进行数据操作的数据库，所有数据库中只要不使用SQL语句的都是非关系型数据库。典型：Redis、MongoDB、hbase、 Hadoop、elasticsearch、图数据库(Neo4j、GraphDB、SequoiaDB)。。。。
	- 每一款都不一样。用途不一致，功能不一致，各有各的操作方式。
	- 基本不支持主外键关系，也没有事务的概念。（MongoDB号称最接近关系型数据库的，所以MongoDB有这些的。）
# redis介绍
## 定义
- Redis（Remote Dictionary Server ，远程字典服务） 是一个使用ANSI C编写的开源、支持网络、基于内存、可选持久性的键值对存储数据库，是NoSQL数据库。
- redis的出现主要是为了替代早期的Memcache缓存系统的。map内存型(数据存放在内存中)的非关系型(nosql)key-value(键值存储)数据库， 支持数据的持久化(基于RDB和AOF，注: 数据持久化时将数据存放到文件中，每次启动redis之后会先将文件中数据加载到内存，经常用来做缓存、数据共享、购物车、消息队列、计数器、限流等。(最基本的就是缓存一些经常用到的数据，提高读写速度)。
## redis特性：
- 速度快
- 持久化
- 多种数据结构
- 支持多种编程语言
- 功能丰富
- 简单：代码短小精悍
- 主从复制
- 高可用、分布式
## Redis的应用场景有哪些？
Redis 的应用场景包括：缓存系统（“热点”数据：高频读、低频写）、计数器、消息队列系统、排行榜、社交网络和实时系统。
## Redis的数据类型及主要特性
Redis提供的数据类型主要分为5种自有类型和一种自定义类型，这5种自有类型包括：String类型、哈希类型、列表类型、集合类型和顺序集合类型。
# redis环境安装
## 下载和安装
- 下载地址：https://github.com/tporadowski/redis/releases
- 使用以下命令启动redis服务端
```
redis-server C:/tool/redis/redis.windows.conf
```
- redis作为windows服务启动方式
```
redis-server --service-install redis.windows.conf
```
- 启动服务
```
redis-server –service-start
```
- 停止服务
```
redis-server –service-stop
```
- 如果连接操作redis，可以在终端下，使用以下命令：
```
redis-cli
```
ubuntu下安装：
```
安装命令：sudo apt-get install -y redis-server
卸载命令：sudo apt-get purge --auto-remove redis-server 
关闭命令：sudo service redis-server stop 
开启命令：sudo service redis-server start 
重启命令：sudo service redis-server restart
配置文件：/etc/redis/redis.conf
```
## redis的配置
```
cat /etc/redis/redis.conf
```
- redis的核心配置选项
	- 绑定ip：访问白名单，如果需要远程访问，可将此注释，或绑定1个真实ip
		```
		bind 127.0.0.1   xx.xx.xx.xx
		```
	- 端口，默认为6379

		```
		port 6379
		```
	- 是否以守护进程运行
		- 如果以守护进程运行，则不会在命令阻塞，类似于服务
		- 如果以守护进程运行，则当前终端被阻塞
		- 设置为yes表示守护进程，设置为no表示⾮守护进程
		- 推荐设置为yes
			```
			daemonize yes
			```
	- RDB持久化的备份策略（RDB备份是默认开启的）
		```
		 # save 时间 读写次数
		 save 900 1     # 当redis在900内至少有1次读写操作，则触发一次数据库的备份操作
		 save 300 10    # 当redis在300内至少有10次读写操作，则触发一次数据库的备份操作
		 save 60 10000  # 当redis在60内至少有10000次读写操作，则触发一次数据库的备份操作
		 ```
	- RDB持久化的备份文件

		```
		dbfilename dump.rdb
		```
	- RDB持久化数据库数据文件的所在目录
		```
		dir /var/lib/redis
		```
	- 日志文件所载目录
		```
		loglevel notice
		logfile /var/log/redis/redis-server.log
		```
	- 进程ID文件
		```
		pidfile /var/run/redis/redis-server.pid
		```
	- 数据库，默认有16个，数据名是不能自定义的，只能是0-15之间，当然这个15是数据库数量-1
		```
		database 16
		```
	- redis的登录密码，生产阶段打开，开发阶段避免麻烦，一般都是注释的。redis在6.0版本以后新增了ACL访问控制机制，新增了用户管理，这个版本以后才有账号和密码，再次之前只有没有密码没有账号
		```
		# requirepass foobared
		```
		- 注意：开启了以后，redis-cli终端下使用 auth 密码来认证登录。
	- AOF持久化的开启配置项(默认值是no，关闭状态)
		```
		appendonly no
		```
	- AOF持久化的备份文件（AOF的备份数据文件与RDB的备份数据文件保存在同一个目录下，由dir配置项指定）
		```
		appendfilename "appendonly.aof"
		```
	- AOF持久化备份策略[时间]
		```
		# appendfsync always
		appendfsync everysec    # 工作中最常用。每一秒备份一次
		# appendfsync no
		```
	- 哨兵集群：一主二从三哨兵(3台服务器)

## Redis的使用
- redis是一款基于CS架构的数据库，所以redis有客户端redis-cli，也有服务端redis-server。
- 其中，客户端可以使用go、java、python等编程语言，也可以终端下使用命令行工具管理redis数据库，甚至可以安装一些别人开发的界面工具，例如：RDM。
- redis-cli客户端连接服务器：
```
# redis-cli -h `redis服务器ip` -p `redis服务器port`
redis-cli -h 10.16.244.3 -p 6379
```
# redis数据类型
- redis可以理解成一个全局的大字典，key就是数据的唯一标识符。根据key对应的值不同，可以划分成5个基本数据类型。
## string类型:
- 字符串类型，是 Redis 中最为基础的数据存储类型，它在 Redis 中是二进制安全的，也就是byte类型。
- 单个数据的最大容量是512M。
```
key: 值
```
```
GET/MGET
SET/SETEX/MSET/MSETNX
INCR/DECR
GETSET
DEL
```
### 设置键值
- set 设置的数据没有额外操作时，是不会过期的。
```
set key value
```
- 设置键为name值为yuan的数据，一个变量可以设置多次
- 注意：redis中的所有数据操作，如果设置的键不存在则为添加，如果设置的键已经存在则修改。
- 设置一个键，当键不存在时才能设置成功，用于一个变量只能被设置一次的情况。
```
setnx  key  value
```
一般用于给数据加锁(分布式锁)
```
127.0.0.1:6379> setnx goods_1 101
(integer) 1
127.0.0.1:6379> setnx goods_1 102
(integer) 0  # 表示设置不成功

127.0.0.1:6379> del goods_1
(integer) 1
127.0.0.1:6379> setnx goods_1 102
(integer) 1
```
### 设置键值的过期时间
redis中可以对一切的数据进行设置有效期。以秒为单位
```
setex key seconds value
setex key1 10 value1
```
### 设置多个键值
```
mset key1 value1 key2 value2 ...
```
### 字符串拼接值
常见于大文件上传
```
127.0.0.1:6379> set title "my"
OK
127.0.0.1:6379> append title "book"
(integer) 6
127.0.0.1:6379> append title "is"
(integer) 8
127.0.0.1:6379> append title "python-redis"
(integer) 20
127.0.0.1:6379> get title
"mybookispython-redis"
```
### 根据键获取值
根据键获取值，如果不存在此键则返回nil
```
get key
```
根据多个键获取多个值
```
mget key1 key2 ...
```
getset：设置key的新值，返回旧值
```
127.0.0.1:6379> getset name haha
"liuly2"
127.0.0.1:6379> get name
"haha"
```
### 自增自减
web开发中的电商抢购、秒杀。游戏里面的投票、攻击计数。系统中计算当前在线人数、
```
127.0.0.1:6379> set id 10
OK
127.0.0.1:6379> incr id
(integer) 11
127.0.0.1:6379> get id
"11"
127.0.0.1:6379> decr id
(integer) 10
127.0.0.1:6379> get id
"10"
127.0.0.1:6379>
127.0.0.1:6379> incrby id 10  # 自增自减大于1的值时候用incrby
(integer) 20
127.0.0.1:6379> get id
"20"
```
### 获取字符串的长度
```
set name xiaoming
strlen name  # 8 
```
### 比特流操作
- 1字节=8比特 1kb = 1024字节 1mb = 1024kb 1gb = 1024mb
- 1个int8就是一个字节，一个中文：3个字节
- 签到记录
```
SETBIT     # 设置一个bit数据的值 
GETBIT     # 获取一个bit数据的值
BITCOUNT   # 统计字符串被设置为1的bit数.
BITPOS     # 返回字符串里面第一个被设置为1或者0的bit位。
```
    
## hash类型:
哈希类型，用于存储对象/字典，对象/字典的结构为键值对。key、域、值的类型都为string。域在同一个hash中是唯一的。
```
结构：
键key:{
    域field: 值value,
    域field: 值value,
    域field: 值value,
}
```
### 设置指定键的属性/域
设置指定键的单个属性
```
hset key field value
```
### 获取指定键的域/属性的值
获取指定键所有的域/属性
```
hkeys key
```
### 获取指定键的单个域/属性的值
```
hget key field
```
### 获取指定键的多个域/属性的值
```
hmget key field1 field2 ...
```
### 获取指定键的所有值
```
hvals key
```
### 获取hash的所有域值对
```
hgetall title
```
### 删除指定键的域/属性
```
hdel key field1 field2 ...
```
### 判断指定属性/域是否存在于当前键对应的hash中
```
hexists   key  field
```
### 属性值自增自减
```
hincrby key field number
```
## list类型:
- 队列，列表的子成员类型为string
```
key: [值1，值2, 值3.....]
```
### 添加子成员
```
# 在左侧(前)添加一条或多条数据
lpush key value1 value2 ...

# 在右侧(后)添加一条或多条数据
rpush key value1 value2 ...

# 在指定元素的左边(前)/右边（后）插入一个或多个数据
linsert key before 指定元素 value1 value2 ....
linsert key after 指定元素 value1 value2 ....
```
注意：当列表如果存在多个成员的情况下，默认只识别第一个。

### 获取列表成员
根据指定的索引(下标)获取成员的值，负数下标从右边-1开始，逐个递减
```
lindex key index
```
移除并获取列表的第一个成员或最后一个成员
```
lpop key  # 第一个成员出列
rpop key  # 最后一个成员出列
# 开发中往往使用rpush和lpop实现队列的数据结构->实现入列和出列
```
### 获取列表的切片
闭区间[包括stop]
```
lrange key start stop
```
### 获取列表的长度
```
llen key
```
### 按索引设置值
```
lset key index value
# redis的列表也有索引，从左往右，从0开始，逐一递增，第1个元素下标为0
# 索引可以是负数，表示尾部开始计数，如`-1`表示最后1个元素
```
### 删除指定成员
```
lrem key count value
# count表示删除的数量，value表示要删除的成员。该命令默认表示将列表从左侧前count个value的元素移除
# count==0，表示删除列表所有值为value的成员
# count >0，表示删除列表左侧开始的前count个value成员
# count <0，表示删除列表右侧开始的前count个value成员
```
## set类型:
无序集合，它的子成员类型为string类型，元素唯一不重复，没有修改操作，重点就是去重和无序。
```
key: {值1, 值4, 值3, ...., 值5}
```
### 添加元素
```
sadd key member1 member2 ...
```
### 获取集合的所有的成员
```
smembers key
```
### 获取集合的长度
```
scard keys 
```
### 随机抽取一个或多个元素
```
spop key [count=1]
# count为可选参数，不填则默认一个。被提取成员会从集合中被删除掉
```
### 删除指定元素
```
srem key value
```
### 交集、差集和并集
```
sinter  key1 key2 key3 ....    # 交集、比较多个集合中共同存在的成员
sdiff   key1 key2 key3 ....    # 差集、比较多个集合中不同的成员
sunion  key1 key2 key3 ....    # 并集、合并所有集合的成员，并去重
```
## zset类型(sortedSet):
有序集合，它的子成员值的类型为string类型，元素唯一不重复，没有修改操作。权重值(score,分数)从小到大排列。
```
key: {
	值1 权重值1(数字);
	值2 权重值2;
	值3 权重值3;
	值4 权重值4;
}
```
### 添加成员
```
zadd key score1 member1 score2 member2 score3 member3 ....
```
### 获取score在指定区间的所有成员
```
zrangebyscore key min max     # 按score进行从低往高排序获取指定score区间
zrevrangebyscore key min max  # 按score进行从高往低排序获取指定score区间
zrange key start stop         # 按scoer进行从低往高排序获取指定索引区间
zrevrange key start stop      # 按scoer进行从高往低排序获取指定索引区间
```
### 获取集合长度
```
zcard key
```
### 获取指定成员的权重值
```
zscore key member
```
### 获取指定成员在集合中的排名
排名从0开始计算
```
zrank key member      # score从小到大的排名
zrevrank key member   # score从大到小的排名
```
### 获取score在指定区间的所有成员数量
```
zcount key min max
```
### 给指定成员增加增加权重值
```
zincrby key score member
```
### 删除成员
```
zrem key member1 member2 member3 ....
```
### 删除指定数量的成员
```
# 删除指定数量的成员，从最低score开始删除
zpopmin key [count]
# 删除指定数量的成员，从最高score开始删除
zpopmax key [count]
```

## 使用场景
- 字符串string: 用于保存项目中普通数据，只要键值对都可以保存，例如，保存 session/jwt,定时记录状态，倒计时、验证码、防灌水答案 
- 哈希hash：用于保存项目中的一些结构体/map类型数据，但是不能保存多维结构，例如，商城的购物车，文章信息，json结构数据 
- 列表list：用于保存项目中的列表/切片数据，但是也不能保存多维结构，例如，消息队列，秒杀系统，排队， 
- 无序集合set: 用于保存项目中的一些不能重复的数据，可以用于过滤，例如，候选人名单, 作者名单， 
- 有序集合zset：用于保存项目中一些不能重复，但是需要进行排序的数据,例如：分数排行榜, 海选人排行榜，热搜排行，



## 任何数据类型都通用的命令
### 查找键
参数支持简单的正则表达式
```
keys pattern
```
查看所有键
```
keys *
```
```
# 查看名称中包含`a`的键
keys *a*
# 查看以a开头的键
keys a*
# 查看以a结尾的键
keys *a
```
### 判断键是否存在
如果存在返回1，不存在返回0
```
exists key1
```
### 查看键的的值的数据类型
```
type key
```
### 删除键以及键对应的值
```
del key1 key2 ...
```
### 查看键的有效期
```
ttl key
# 结果结果是秒作为单位的整数
# -1 表示永不过期
# -2 表示当前数据已经过期，查看一个不存在的数据的有效期就是-2
```
### 设置key的有效期
给已有的数据重新设置有效期，redis中所有的数据都可以通过expire来设置它的有效期。有效期到了，数据就被删除。
```
expire key seconds
```
### 清空所有key
慎用，一旦执行，则redis所有数据库0~15的全部key都会被清除
```
flushall
```
### key重命名
```
rename  oldkey newkey
```
### select切换数据库
redis的配置文件中，默认有0~15之间的16个数据库，默认操作的就是0号数据库
```
select <数据库ID>
```

# Python操作redis
## python对redis基本操作
### 连接redis
```
# 方式1
import redis

r = redis.Redis(host='127.0.0.1', port=6379)
r.set('foo', 'Bar')
print(r.get('foo'))

# 方式2
import redis

pool = redis.ConnectionPool(host='127.0.0.1', port=6379)
r = redis.Redis(connection_pool=pool)
r.set('bar', 'Foo')
print(r.get('bar'))
```
通常情况下, 当我们需要做redis操作时, 会创建一个连接, 并基于这个连接进行redis操作, 操作完成后, 释放连接,一般情况下, 这是没问题的, 但当并发量比较高的时候, 频繁的连接创建和释放对性能会有较高的影响。于是, 连接池就发挥作用了。连接池的原理是, 通过预先创建多个连接, 当进行redis操作时, 直接获取已经创建的连接进行操作, 而且操作完成后, 不会释放, 用于后续的其他redis操作。这样就达到了避免频繁的redis连接创建和释放的目的, 从而提高性能。

### 数据类型操作
```
import redis

pool = redis.ConnectionPool(host='127.0.0.1', port=6379, db=0, decode_responses=True)
r = redis.Redis(connection_pool=pool)

# （1）字符串操作：不允许对已经存在的键设置值
ret = r.setnx("name", "yuan")
print(ret)  # False
# （2）字符串操作：设置键有效期
r.setex("good_1001", 10, "2")
# （3）字符串操作：自增自减
r.set("age", 20)
r.incrby("age", 2)
print(r.get("age"))  # b'22'

# （4）hash操作：设置hash
r.hset("info", "name", "rain")
print(r.hget("info", "name"))  # b'rain'
r.hset("info", "gender", "male", {"age": 22})
print(r.hgetall("info"))  # {b'name': b'rain', b'gender': b'male', b'age': b'22'}

# （5）list操作：设置list
r.rpush("scores", "100", "90", "80")
r.rpush("scores", "70")
r.lpush("scores", "120")
print(r.lrange("scores", 0, -1))  # ['120', '100', '90', '80', '70']
r.linsert("scores", "AFTER", "100", 95)
print(r.lrange("scores", 0, -1))  # ['120', '100', '95', '90', '80', '70']
print(r.lpop("scores"))  # 120
print(r.rpop("scores"))  # 70
print(r.lindex("scores", 1)) # '95'

# （6）集合操作
# key对应的集合中添加元素
r.sadd("name_set", "zhangsan", "lisi", "wangwu")
# 获取key对应的集合的所有成员
print(r.smembers("name_set"))  # {'lisi', 'zhangsan', 'wangwu'}
# 从key对应的集合中随机获取 numbers 个元素
print(r.srandmember("name_set", 2))
r.srem("name_set", "lisi")
print(r.smembers("name_set"))  # {'wangwu', 'zhangsan'}

# （7）有序集合操作
# 在key对应的有序集合中添加元素
r.zadd("jifenbang", {"yuan": 78, "rain": 20, "alvin": 89, "eric": 45})
# 按照索引范围获取key对应的有序集合的元素
# zrange( name, start, end, desc=False, withscores=False, score_cast_func=float)
print(r.zrange("jifenbang", 0, -1))  # ['rain', 'eric', 'yuan', 'alvin']
print(r.zrange("jifenbang", 0, -1, withscores=True))  # ['rain', 'eric', 'yuan', 'alvin']
print(r.zrevrange("jifenbang", 0, -1, withscores=True))  # ['rain', 'eric', 'yuan', 'alvin']

print(r.zrangebyscore("jifenbang", 0, 100))
print(r.zrangebyscore("jifenbang", 0, 100, start=0, num=1))

# 删除key对应的有序集合中值是values的成员
print(r.zrem("jifenbang", "yuan"))  # 删除成功返回1
print(r.zrange("jifenbang", 0, -1))  # ['rain', 'eric', 'alvin']

# （8）键操作
r.delete("scores")
print(r.exists("scores"))
print(r.keys("*"))
r.expire("name",10)
```
## redis案例
### KV缓存
![](uTools_1653744188860.png)
第1个是最基础也是最常?的就是KV功能，我们可以用Redis来缓存用户信息、会话信息、商品信息等等。下面这段代码就是通过缓存读取逻辑。
```
import redis

pool = redis.ConnectionPool(host='127.0.0.1', port=6379, db=6, decode_responses=True)
r = redis.Redis(connection_pool=pool)

def get_user(user_id):
    user = r.get(user_id)
    if not user:
        user = UserInfo.objects.get(pk=user_id)
        r.setex(user_id, 3600, user)
    return user
```
### 分布式锁
- 什么是分布式锁
	- 分布式锁其实就是，控制分布式系统不同进程共同访问共享资源的一种锁的实现。如果不同的系统或同一个系统的不同主机之间共享了某个临界资源，往往需要互斥来防止彼此干扰，以保证一致性。
- 提到Redis的分布式锁，很多小伙伴马上就会想到setnx+ expire命令。即先用setnx来抢锁，如果抢到之后，再用expire给锁设置一个过期时间，防止锁忘记了释放。
	- SETNX 是SET IF NOT EXISTS的简写.日常命令格式是SETNX key value，如果 key不存在，则SETNX成功返回1，如果这个key已经存在了，则返回0。
- 假设某电商网站的某商品做秒杀活动，key可以设置为key_resource_id,value设置任意值，伪代码如下：
```
方案1
import redis

pool = redis.ConnectionPool(host='127.0.0.1')
r = redis.Redis(connection_pool=pool)
ret = r.setnx("key_resource_id", "ok")
if ret:
    r.expire("key_resource_id", 5)  # 设置过期时间
    print("抢购成功！")
    r.delete("key_resource_id")  # 释放资源
else:
    print("抢购失败！")
```
- 但是这个方案中，setnx和expire两个命令分开了，「不是原子操作」。如果执行完setnx加锁，正要执行expire设置过期时间时，进程crash或者要重启维护了，那么这个锁就“长生不老”了，「别的线程永远获取不到锁啦」。

```
方案2：SETNX + value值是(系统时间+过期时间)
为了解决方案一，「发生异常锁得不到释放的场景」，可以把过期时间放到setnx的value值里面。如果加锁失败，再拿出value值校验一下即可。加锁代码如下：
import time

def foo():
    expiresTime = time.time() + 10
    ret = r.setnx("key_resource_id", expiresTime)
    if ret:
        print("当前锁不存在，加锁成功")
        return True
    oldExpiresTime = r.get("key_resource_id")
    if float(oldExpiresTime) < time.time():  # 如果获取到的过期时间，小于系统当前时间，表示已经过期
        # 锁已过期，获取上一个锁的过期时间，并设置现在锁的过期时间
        newExpiresTime = r.getset("key_resource_id", expiresTime)
        if oldExpiresTime == newExpiresTime:
            #  考虑多线程并发的情况，只有一个线程的设置值和当前值相同，它才可以加锁
            return True  # 加锁成功
    return False  # 其余情况加锁皆失败

foo()
```
```
方案3
实际上，我们还可以使用Py的redis模块中的set函数来保证原子性（包含setnx和expire两条指令）代码如下：
r.set("key_resource_id", "1", nx=True, ex=10)
```
### 定时任务
利用 Redis 也能实现订单30分钟自动取消。
用户下单之后，在规定时间内如果不完成付款，订单自动取消，并且释放库存使用技术：Redis键空间通知（过期回调）用户下单之后将订单id作为key，任意值作为值存入redis中，给这条数据设置过期时间，也就是订单超时的时间启用键空间通知
```
from redis import StrictRedis

redis = StrictRedis(host='localhost', port=6379)

# 监听所有事件
# pubsub = redis.pubsub()
# pubsub.psubscribe('__keyspace@0__:*')
# print('Starting message loop')
# while True:
#     message = pubsub.get_message()
#     if message:
#         print(message)

# 监听过期key
def event_handler(msg):
    print("sss",msg)
    thread.stop()

pubsub = redis.pubsub()
pubsub.psubscribe(**{'__keyevent@0__:expired': event_handler})
thread = pubsub.run_in_thread(sleep_time=0.01)
```

### 延迟队列
延时队列可以通过Redis的zset(有序列表)来实现。我们将消息序列化为一个字符串作为zset的值。这个消息的到期时间处理时间作为score，然后用多个线程轮询zset获取到期的任务进行处理，多线程时为了保障可用性，万一挂了一个线程还有其他线程可以继续处理。因为有多个线程，所有需要考虑并发争抢任务，确保任务不能被多次执行。
```
import time
import uuid
import redis

pool = redis.ConnectionPool(host='127.0.0.1', port=6379, decode_responses=True)
r = redis.Redis(connection_pool=pool)

def delay_task(task_name, delay_time):
    # 保证value唯一
    task_id = task_name + str(uuid.uuid4())
    print()
    # 5s后重试
    retry_ts = time.time() + delay_time
    r.zadd("delay-queue", {task_id: retry_ts})

def loop():
    print("循环监听中...")
    while True:
        # 最多取1条
        task_list = r.zrangebyscore("delay-queue", 0, time.time(), start=0, num=1)
        if not task_list:
            # 延时队列空的，休息1s
            print("cost 1秒钟")
            time.sleep(1)
            continue
        task_id = task_list[0]
        success = r.zrem("delay-queue", task_id)
        if success:
            # 处理消息逻辑函数
            handle_msg(task_id)

def handle_msg(msg):
    """消息处理逻辑"""
    print(f"消息{msg}已经被处理完成！")

import threading
t = threading.Thread(target=loop)
t.start()

delay_task("任务1延迟5", 5)
delay_task("任务2延迟2", 2)
delay_task("任务3延迟3", 3)
delay_task("任务4延迟10", 10)
```
- redis的zrem方法是对多线程争抢任务的关键，它的返回值决定了当前实例有没有抢到任务，因为loop方法可能会被多个线程、多个进程调用， 同一个任务可能会被多个进程线程抢到，通过zrem来决定唯一的属主。
- 同时，一定要对handle_msg进行异常捕获， 避免因为个别任务处理问题导致的循环异常退出。

### 发布订阅
```
import threading
import redis

r = redis.Redis(host='127.0.0.1')

def recv_msg():
    pub = r.pubsub()
    pub.subscribe("fm104.5")
    pub.parse_response()
    while 1:
        msg = pub.parse_response()
        print(msg)

def send_msg():
    msg = input(">>>")
    r.publish("fm104.5", msg)

t = threading.Thread(target=send_msg)
t.start()
recv_msg()
```
# 参考
- [http://www.yuan316.com/post/redis/](http://www.yuan316.com/post/redis/)
- [Redis官方原版](https://redis.io/)
- [Redis中文官网](http://www.redis.cn)