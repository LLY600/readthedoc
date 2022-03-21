Readthedocs搭建
=========================================

本文介绍一种在线文档系统的搭建，需要借助Sphinx、github(或其他代码托管平台)和ReadtheDocs。

Sphinx是一个功能强大的文档生成器，具有许多用于编写技术文档的强大功能

	https://www.sphinx-doc.org/en/master/（官网）
	
	https://zh-sphinx-doc.readthedocs.io/en/latest/contents.html（使用手册）
	
gitee是一种版本管理系统，相比github，有着更快的访问速度

ReadtheDocs是一个在线文档托管服务，你可以从各种版本控制系统中导入文档

	https://readthedocs.org/（官网）

1、安装环境（git环境、python环境、python依赖包），ReadtheDocs和远程仓库账号已注册
	
	pip3 install sphinx
	
	pip3 install sphinx-autobuild
	
	pip3 install sphinx_rtd_theme
	
	pip3 install recommonmark
	
	pip3 install sphinx_markdown_tables

2、创建文件夹，进入文件夹，快速开始，命令行中执行：sphinx-quickstart，新建一个Sphinx的项目框架。会有如下的输出，根据提示输入项目名称、作者、版本号、语言等信息
	
	sphinx-quickstart
	
	Separate source and build directories (y/n) [n]: y
	
	Project name: SphinxDemo 
	
	Author name(s): xxpcb
	
	Project release []: v1.0
	
	Project language [en]: zh_CN

3、项目创建完成后，可以看到如下的目录结构：
	
	build：生成的文件的输出目录
	
	source: 存放文档源文件
	
	_static：静态文件目录，比如图片等
	
	_templates：模板目录
	
	conf.py：进行 Sphinx 的配置，如主题配置等
	
	index.rst：文档项目起始文件，用于配置文档的显示结构
	
	cmd.bat：这是自己加的脚本文件（里面的内容是‘cmd.exe’）,用于快捷的打开windows的命令行
	
	make.bat：Windows 命令行中编译用的脚本
	
	Makefile：编译脚本，make 命令编译时用
		
4、借助sphinx-autobuild工具启动HTTP服务
	
	sphinx-autobuild source build/html

5、更改样式主题到Sphinx官网

	https://sphinx-themes.org（官网选择主题）	
	pip install sphinx_rtd_theme（如安装主题：sphinx_rtd_theme）
	修改conf.py 文件，找到 html_theme 字段，如html_theme = 'sphinx_rtd_theme'

6、修改测试程序，Sphinx默认只支持reST格式的文件，reST的使用语法介绍见：

	https://zh-sphinx-doc.readthedocs.io/en/latest/rest.html


7、安装markdown支持、markdown表格支持的工具

	pip install  recommonmark
	pip install  sphinx_markdown_tables
	修改conf.py 文件，找到 extensions字段，修改为:extensions = ['recommonmark','sphinx_markdown_tables']
	注：支持markdown后，文档文件可以使用markdown格式，但文档的配置文件index.rst还要使用reST格式

8、修改文档结构，需要修改index.rst文件，首先来看一下这个文件中的内容：

	.. SphinxDemo documentation master file, created by
	   sphinx-quickstart on Sat Jun 26 17:56:51 2021.
	   You can adapt this file completely to your liking, but it should at least
	   contain the root `toctree` directive.	​
	Welcome to SphinxDemo's documentation!
	======================================		​
	.. toctree::
	   :maxdepth: 2
	   :caption: Contents:
	​	​
	Indices and tables
	==================
	​
	* :ref:`genindex`
	* :ref:`modindex`
	* :ref:`search`
	​
	两个点..+空格+后面的文本，代表注释（网页上不显示）
	等号线====+上一行的文本，代表一级标题
	.. toctree::声明的一个树状结构（Table of Content Tree）
	:maxdepth: 2 表示页面的级数最多显示两级
	:caption: Contents: 用于指定标题文本（可以不要）
	最下面的3行是索引和搜索链接（可以先不用管）

9、项目托管到gitee

	先到gitee上（https://gitee.com/）建立一个公开的仓库，然后将本地项目文件上传
	写一个.gitignore文件，用于指示编辑的文件（build目录）不上传到代码仓库
	git init
	git add -A
	git commit -m "first commit"
	git remote add origin https://gitee.com/xxpcb/sphinx-demo.git
	git push -u origin master
	
10、部署到ReadtheDocs展示

	借助ReadtheDocs平台：https://readthedocs.org/
	选择手动导入一个项目：
	将gitee仓库的HTTPS链接复制过来
	填写项目名称，填写gitee仓库的HTTPS链接
	点击Build version进行项目构建
