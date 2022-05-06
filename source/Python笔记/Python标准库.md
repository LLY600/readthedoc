Python标准库
============================================
#### 文本

1. string：通用字符串操作
2. re：正则表达式操作
3. difflib：差异计算工具
4. textwrap：文本填充
5. unicodedata：Unicode字符数据库
6. stringprep：互联网字符串准备工具
7. readline：GNU按行读取接口
8. rlcompleter：GNU按行读取的实现函数
#### 二进制数据

1. struct：将字节解析为打包的二进制数据
2. codecs：注册表与基类的编解码器
#### 数据类型

1. datetime：基于日期与时间工具
2. calendar：通用月份函数
3. collections：容器数据类型
4. collections.abc：容器虚基类
5. heapq：堆队列算法
6. bisect：数组二分算法
7. array：高效数值数组
8. weakref：弱引用
9. types：内置类型的动态创建与命名
10. copy：浅拷贝与深拷贝
11. pprint：格式化输出
12. reprlib：交替repr()的实现
#### 数学

1. numbers：数值的虚基类
2. math：数学函数
3. cmath：复数的数学函数
4. decimal：定点数与浮点数计算
5. fractions：有理数
6. random：生成伪随机数
#### 函数式编程

1. itertools：为高效循环生成迭代器
2. functools：可调用对象上的高阶函数与操作
3. operator：针对函数的标准操作
#### 文件与目录

1. os.path：通用路径名控制
2. fileinput：从多输入流中遍历行
3. stat：解释stat()的结果
4. filecmp：文件与目录的比较函数
5. tempfile：生成临时文件与目录
6. glob：Unix风格路径名格式的扩展
7. fnmatch：Unix风格路径名格式的比对
8. linecache：文本行的随机存储
9. shutil：高级文件操作
10. macpath：MacOS 9路径控制函数
#### 持久化

1. pickle：Python对象序列化
2. copyreg：注册机对pickle的支持函数
3. shelve：Python对象持久化
4. marshal：内部Python对象序列化
5. dbm：Unix“数据库”接口
6. sqlite3：针对SQLite数据库的API2.0
#### 压缩

1. zlib：兼容gzip的压缩
2. gzip：对gzip文件的支持
3. bz2：对bzip2压缩的支持
4. lzma：使用LZMA算法的压缩
5. zipfile：操作ZIP存档
6. tarfile：读写tar存档文件
#### 文件格式化

1. csv：读写CSV文件
2. configparser：配置文件解析器
3. netrc：netrc文件处理器
4. xdrlib：XDR数据编码与解码
5. plistlib：生成和解析Mac OS X.plist文件
#### 加密

1. hashlib：安全散列与消息摘要
2. hmac：针对消息认证的键散列
#### 操作系统工具

1. os：多方面的操作系统接口
2. io：流核心工具
3. time：时间的查询与转化
4. argparser：命令行选项、参数和子命令的解析器
5. optparser：命令行选项解析器
6. getopt：C风格的命令行选项解析器
7. logging：Python日志工具
8. logging.config：日志配置
9. logging.handlers：日志处理器
10. getpass：简易密码输入
11. curses：字符显示的终端处理
12. curses.textpad：curses程序的文本输入域
13. curses.ascii：ASCII字符集工具
14. curses.panel：curses的控件栈扩展
15. platform：访问底层平台认证数据
16. errno：标准错误记号
17. ctypes：Python外部函数库
#### 并发

1. threading：基于线程的并行
2. multiprocessing：基于进程的并行
3. concurrent：并发包
4. concurrent.futures：启动并行任务
5. subprocess：子进程管理
6. sched：事件调度
7. queue：同步队列
8. select：等待I / O完成
9. dummy_threading：threading模块的替代（当_thread不可用时）
10. _thread：底层的线程API（threading基于其上）
11. _dummy_thread：_thread模块的替代（当_thread不可用时）
#### 进程间通信

1. socket：底层网络接口
2. ssl：socket对象的TLS / SSL填充器
3. asyncore：异步套接字处理器
4. asynchat：异步套接字命令 / 响应处理器
5. signal：异步事务信号处理器
6. mmap：内存映射文件支持
#### 互联网

1. mail：邮件与MIME处理包
2. json：JSON编码与解码
3. mailcap：mailcap文件处理
4. mailbox：多种格式控制邮箱
5. mimetypes：文件名与MIME类型映射
6. base64：RFC
7. 3548：Base16、Base32、Base64编码
8. binhex：binhex4文件编码与解码
9. binascii：二进制码与ASCII码间的转化
10. quopri：MIME
11. quoted - printable数据的编码与解码
12. uu：uuencode文件的编码与解码
#### HTML与XML

1. html：HTML支持
2. html.parser：简单HTML与XHTML解析器
3. html.entities：HTML通用实体的定义
4. xml：XML处理模块
5. xml.etree.ElementTree：树形XML元素API
6. xml.dom：XML DOM API
7. xml.dom.minidom：XML DOM最小生成树
8. xml.dom.pulldom：构建部分DOM树的支持
9. xml.sax：SAX2解析的支持
10. xml.sax.handler：SAX处理器基类
11. xml.sax.saxutils：SAX工具
12. xml.sax.xmlreader：SAX解析器接口
13. xml.parsers.expat：运用Expat快速解析XML
#### 互联网协议与支持

1. webbrowser：简易Web浏览器控制器
2. cgi：CGI支持
3. cgitb：CGI脚本反向追踪管理器
4. wsgiref：WSGI工具与引用实现
5. urllib：URL处理模块
6. urllib.request：打开URL连接的扩展库
7. urllib.response：urllib模块的响应类
8. urllib.parse：将URL解析成组件
9. urllib.error：urllib.request引发的异常类
10. urllib.robotparser：robots.txt的解析器
11. http：HTTP模块
12. http.client：HTTP协议客户端
13. ftplib：FTP协议客户端
14. poplib：POP协议客户端
15. imaplib：IMAP4协议客户端
16. nntplib：NNTP协议客户端
17. smtplib：SMTP协议客户端
18. smtpd：SMTP服务器
19. telnetlib：Telnet客户端
20. uuid：RFC4122的UUID对象
21. socketserver：网络服务器框架
22. http.server：HTTP服务器
23. http.cookies：HTTPCookie状态管理器
24. http.cookiejar：HTTP客户端的Cookie处理
25. xmlrpc：XML - RPC服务器和客户端模块
26. xmlrpc.client：XML - RPC客户端访问
27. xmlrpc.server：XML - RPC服务器基础
28. ipaddress：IPv4 / IPv6控制库
#### 多媒体

1. audioop：处理原始音频数据
2. aifc：读写AIFF和AIFC文件
3. sunau：读写Sun AU文件
4. wave：读写WAV文件
5. chunk：读取IFF大文件
6. colorsys：颜色系统间转化
7. imghdr：指定图像类型
8. sndhdr：指定声音文件类型
9. ossaudiodev：访问兼容OSS的音频设备
#### 国际化

1. gettext：多语言的国际化服务
2. locale：国际化服务
#### 编程框架

1. turtle：Turtle图形库
2. cmd：基于行的命令解释器支持
3. shlex：简单词典分析
#### Tk图形用户接口

1. tkinter：Tcl / Tk接口
2. tkinter.ttk：Tk主题控件
3. tkinter.tix：Tk扩展控件
4. tkinter.scrolledtext：滚轴文本控件
#### 开发工具

1. pydoc：文档生成器和在线帮助系统
2. doctest：交互式Python示例
3. unittest：单元测试框架
4. unittest.mock：模拟对象库
5. test：Python回归测试包
6. test.support：Python测试工具套件
7. venv：虚拟环境搭建
#### 调试

1. bdb：调试框架
2. faulthandler：Python反向追踪库
3. pdb：Python调试器
4. timeit：小段代码执行时间测算
5. trace：Python执行状态追踪
#### 运行时

1. sys：系统相关的参数与函数
2. sysconfig：访问Python配置信息
3. builtins：内置对象
4. main：顶层脚本环境
5. warnings：警告控制
6. contextlib：with状态的上下文工具
7. abc：虚基类
8. atexit：出口处理器
9. traceback：打印或读取一条栈的反向追踪
10. future：未来状态定义
11. gc：垃圾回收接口
12. inspect：检查存活的对象
13. site：址相关的配置钩子（hook）
14. fpectl：浮点数异常控制
15. distutils：生成和安装Python模块
#### 解释器

1. code：基类解释器
2. codeop：编译Python代码
#### 导入模块

1. imp：访问import模块的内部
2. zipimport：从ZIP归档中导入模块
3. pkgutil：包扩展工具
4. modulefinder：通过脚本查找模块
5. runpy：定位并执行Python模块
6. importlib：import的一种实施
#### Python语言

1. parser：访问Python解析树
2. ast：抽象句法树
3. symtable：访问编译器符号表
4. symbol：Python解析树中的常量
5. token：Python解析树中的常量
6. keyword：Python关键字测试
7. tokenize：Python源文件分词
8. tabnany：模糊缩进检测
9. pyclbr：Python类浏览支持
10. py_compile：编译Python源文件
11. compileall：按字节编译Python库
12. dis：Python字节码的反汇编器
13. pickletools：序列化开发工具
#### 其他

1. formatter：通用格式化输出
#### Windows相关

1. msilib：读写Windows的Installer文件
2. msvcrt：MS VC + + Runtime的有用程序
3. winreg：Windows注册表访问
4. winsound：Windows声音播放接口
#### Unix相关

1. posix：最常用的POSIX调用
2. pwd：密码数据库
3. spwd：影子密码数据库
4. grp：组数据库
5. crypt：Unix密码验证
6. termios：POSIX风格的tty控制
7. tty：终端控制函数
8. pty：伪终端工具
9. fcntl：系统调用fcntl()和ioctl()
10. pipes：shell管道接口
11. resource：资源可用信息
12. nis：Sun的NIS的接口
13. syslog：Unix 日志服务
