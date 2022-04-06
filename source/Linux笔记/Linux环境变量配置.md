Linux环境变量配置
================
#### 背景
- Linux是多用户的任务系统，对于每个用户都可以指定特定的用户变量。
- 针对不同程序在不同的环境下运行就需要修改环境变量进行定制。

#### 环境变量配置
- 按变量的周期划为永久变量和临时性变量2种
	- 永久变量：通过修改配置文件，配置之后变量永久生效。
	- 临时性变量：使用命令如export等命令设置，设置之后马上生效。当关闭shell的时候失效（这种主要用于测试比较多）。
- 按照影响范围分为用户变量和系统变量2种
	- 用户变量（局部变量）：修改的设置只对某个用户的路径或执行起作用；
	- 系统变量（全局变量）：影响范围是整个系统；
	- 下面所有例子的环境说明如下：
	```
	系统：Ubuntu 14.0
	用户名：uusama
	需要配置MySQL环境变量路径：/home/uusama/mysql/bin
	```

1. 读取环境变量

	- 读取环境变量的方法：
		- export命令显示当前系统定义的所有环境变量
		- echo $PATH命令输出当前的PATH环境变量的值
		- 这两个命令执行的效果如下
		```
		uusama@ubuntu:~$ export
		declare -x HOME="/home/uusama"
		declare -x LANG="en_US.UTF-8"
		declare -x LANGUAGE="en_US:"
		declare -x LESSCLOSE="/usr/bin/lesspipe %s %s"
		declare -x LESSOPEN="| /usr/bin/lesspipe %s"
		declare -x LOGNAME="uusama"
		declare -x MAIL="/var/mail/uusama"
		declare -x PATH="/home/uusama/bin:/home/uusama/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
		declare -x SSH_TTY="/dev/pts/0"
		declare -x TERM="xterm"
		declare -x USER="uusama"
		```
		
		```
		uusama@ubuntu:~$ echo $PATH
		/home/uusama/bin:/home/uusama/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
		```
	
		- 其中PATH变量定义了运行命令的查找路径，以冒号:分割不同的路径，使用export定义的时候可加双引号也可不加。


1. 环境变量配置方式
	- **export PATH**
		- 使用export命令直接修改PATH的值，配置MySQL进入环境变量的方法:
		```
		export PATH=/home/uusama/mysql/bin:$PATH
		#或者把PATH放在前面
		export PATH=$PATH:/home/uusama/mysql/bin
		```
		- 注意事项：
			- 生效时间：立即生效
			- 生效期限：当前终端有效，窗口关闭后无效
			- 生效范围：仅对当前用户有效
			- 配置的环境变量中不要忘了加上原来的配置，即$PATH部分，避免覆盖原来配置
	- **vim ~/.bashrc**
		- 通过修改用户目录下的~/.bashrc文件进行配置：
		```
		vim ~/.bashrc
		# 在最后一行加上
		export PATH=$PATH:/home/uusama/mysql/bin
		```
		- 注意事项：
			- 生效时间：使用相同的用户打开新的终端时生效，或者手动source ~/.bashrc生效
			- 生效期限：永久有效
			- 生效范围：仅对当前用户有效
			- 如果有后续的环境变量加载文件覆盖了PATH定义，则可能不生效
	- **vim ~/.bash_profile**
		```
		vim ~/.bash_profile
		# 在最后一行加上
		export PATH=$PATH:/home/uusama/mysql/bin
		```
		- 注意事项：
			- 生效时间：使用相同的用户打开新的终端时生效，或者手动source ~/.bash_profile生效
			- 生效期限：永久有效
			- 生效范围：仅对当前用户有效
			- 如果没有~/.bash_profile文件，则可以编辑~/.profile文件或者新建一个
	- **vim /etc/bashrc**
		- 该方法是修改系统配置，需要管理员权限（如root）或者对该文件的写入权限：
		```
		# 如果/etc/bashrc文件不可编辑，需要修改为可编辑
		chmod -v u+w /etc/bashrc
		vim /etc/bashrc
		# 在最后一行加上
		export PATH=$PATH:/home/uusama/mysql/bin
		```
		- 注意事项：
			- 生效时间：新开终端生效，或者手动source /etc/bashrc生效
			- 生效期限：永久有效
			- 生效范围：对所有用户有效
	- **vim /etc/profile**
		- 该方法修改系统配置，需要管理员权限或者对该文件的写入权限，和vim /etc/bashrc类似：
		```
		# 如果/etc/profile文件不可编辑，需要修改为可编辑
		chmod -v u+w /etc/profile
		vim /etc/profile
		# 在最后一行加上
		export PATH=$PATH:/home/uusama/mysql/bin
		```
		- 注意事项：
			- 生效时间：新开终端生效，或者手动source /etc/profile生效
			- 生效期限：永久有效
			- 生效范围：对所有用户有效
	- **vim /etc/environment**
		- 该方法是修改系统环境配置文件，需要管理员权限或者对该文件的写入权限：
		```
		# 如果/etc/bashrc文件不可编辑，需要修改为可编辑
		chmod -v u+w /etc/environment
		vim /etc/profile
		# 在最后一行加上
		export PATH=$PATH:/home/uusama/mysql/bin
		```
		- 注意事项：
			- 生效时间：新开终端生效，或者手动source /etc/environment生效
			- 生效期限：永久有效
			- 生效范围：对所有用户有效
  
#### 环境变量加载原理解析
- 上面列出了环境变量的各种配置方法，那么Linux是如何加载这些配置的呢？是以什么样的顺序加载的呢？
- 特定的加载顺序会导致相同名称的环境变量定义被覆盖或者不生效。
- 环境变量的分类
	- 环境变量可以简单的分成用户自定义的环境变量以及系统级别的环境变量。
	- 用户级别环境变量定义文件：~/.bashrc、~/.profile（部分系统为：~/.bash_profile）
	- 系统级别环境变量定义文件：/etc/bashrc、/etc/profile(部分系统为：/etc/bash_profile）、/etc/environment
	- 另外在用户环境变量中，系统会首先读取~/.bash_profile（或者~/.profile）文件，如果没有该文件则读取~/.bash_login，根据这些文件中内容再去读取~/.bashrc。

#### 测试环境变量加载顺序
- 为了测试各个不同文件的环境变量加载顺序，我们在每个环境变量定义文件中的第一行都定义相同的环境变量UU_ORDER，该变量的值为本身的值连接上当前文件名称。
- 需要修改的文件如下：
```
/etc/environment
/etc/profile
/etc/profile.d/test.sh，新建文件，没有文件夹可略过
/etc/bashrc，或者/etc/bash.bashrc
~/.bash_profile，或者~/.profile
~/.bashrc
```
- 在每个文件中的第一行都加上下面这句代码，并相应的把冒号后的内容修改为当前文件的绝对文件名。
```
export UU_ORDER="$UU_ORDER:~/.bash_profile"
```
- 修改完之后保存，新开一个窗口，然后echo $UU_ORDER观察变量的值：
```
uusama@ubuntu:~$ echo $UU_ORDER
$UU_ORDER:/etc/environment:/etc/profile:/etc/bash.bashrc:/etc/profile.d/test.sh:~/.profile:~/.bashrc
```
- 可以推测出Linux加载环境变量的顺序如下：
```
/etc/environment
/etc/profile
/etc/bash.bashrc
/etc/profile.d/test.sh
~/.profile
~/.bashrc
```

#### 环境变量文件加载详解
- 由上面的测试可容易得出Linux加载环境变量的顺序如下：
- 系统环境变量 -> 用户自定义环境变量 /etc/environment -> /etc/profile -> ~/.profile
- 打开/etc/profile文件你会发现，该文件的代码中会加载/etc/bash.bashrc文件，然后检查/etc/profile.d/目录下的.sh文件并加载。
```
# /etc/profile: system-wide .profile file for the Bourne shell (sh(1))
# and Bourne compatible shells (bash(1), ksh(1), ash(1), ...).

if [ "$PS1" ]; then
  if [ "$BASH" ] && [ "$BASH" != "/bin/sh" ]; then
	# The file bash.bashrc already sets the default PS1.
	# PS1='\h:\w\$ '
	if [ -f /etc/bash.bashrc ]; then
	  . /etc/bash.bashrc
	fi
  else
	if [ "`id -u`" -eq 0 ]; then
	  PS1='# '
	else
	  PS1='$ '
	fi
  fi
fi

if [ -d /etc/profile.d ]; then
  for i in /etc/profile.d/*.sh; do
	if [ -r $i ]; then
	  . $i
	fi
  done
  unset i
fi
```
- 其次再打开~/.profile文件，会发现该文件中加载了~/.bashrc文件。
```
# if running bash
if [ -n "$BASH_VERSION" ]; then
	# include .bashrc if it exists
	if [ -f "$HOME/.bashrc" ]; then
 . "$HOME/.bashrc"
	fi
fi

# set PATH so it includes user's private bin directories
PATH="$HOME/bin:$HOME/.local/bin:$PATH"
```
- 从~/.profile文件中代码不难发现，/.profile文件只在用户登录的时候读取一次，而/.bashrc会在每次运行Shell脚本的时候读取一次。

#### 一些小技巧
- 可以自定义一个环境变量文件，比如在某个项目下定义uusama.profile，在这个文件中使用export定义一系列变量，然后在~/.profile文件后面加上：sourc uusama.profile，这样你每次登陆都可以在Shell脚本中使用自己定义的一系列变量。
- 也可以使用alias命令定义一些命令的别名，比如alias rm="rm -i"（双引号必须），并把这个代码加入到~/.profile中，这样你每次使用rm命令的时候，都相当于使用rm -i命令，非常方便。

#### 参考链接
- [https://blog.csdn.net/AI_Green/article/details/121422245]()
