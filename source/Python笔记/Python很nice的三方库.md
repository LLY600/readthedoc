Python很nice的三方库
============================================
#### Pyintervals
- Docs：[https://pyinterval.readthedocs.io/en/latest/index.html](https://pyinterval.readthedocs.io/en/latest/index.html)
- 解决数值区间问题

#### attrs、cattrs
- GitHub：[https://github.com/python-attrs/attrs、https://github.com/Tinche/cattrs](https://github.com/python-attrs/attrs、https://github.com/Tinche/cattrs)
- 简化类的定义、序列化反序列化等操作。

#### loguru
- GitHub：[https://github.com/Delgan/loguru](https://github.com/Delgan/loguru)
- 可简化日志记录写法。

#### autopep8
- GitHub：[https://github.com/hhatto/autopep8](https://github.com/hhatto/autopep8)
- 把 Python 代码转成符合 PEP8 规范的代码。

#### psutil
- GitHub：[https://github.com/giampaolo/psutil](https://github.com/giampaolo/psutil)
- Python 实现任务监控的库。

#### furl
- GitHub：[https://github.com/gruns/furl](https://github.com/gruns/furl)
- 对 url 的处理非常方便，比 urllib 等库好用多。

#### retrying、tenacity
- GitHub：[https://github.com/rholder/retrying、https://github.com/jd/tenacity](https://github.com/rholder/retrying、https://github.com/jd/tenacity)
- 异常重试库，如出错之后重试多少次，尤其在发起一些 HTTP 请求时非常有用，当然也能用于其他地方。

#### typing
- Docs：[https://docs.python.org/zh-cn/3/library/typing.html#module-typing](https://docs.python.org/zh-cn/3/library/typing.html#module-typing)
- 对 Python 类型的支持，支持多种类型、嵌套类型，也推荐多多使用 Python 的类型注解。

#### argparse
- Docs：[https://docs.python.org/zh-cn/3/library/argparse.html](https://docs.python.org/zh-cn/3/library/argparse.html)
- 个人曾经使用过几个命令行解析工具，如 docopt，但后来还是转回来了 argparse，功能齐全强大。

#### absl-py
- GitHub：[https://github.com/abseil/abseil-py](https://github.com/abseil/abseil-py)
- 个人感觉比 argparse 更易用的库，如 TensorFlow 就在使用这个，对于定义一些 Flag 非常方便。

#### pipenv
- GitHub：[https://github.com/pypa/pipenv](https://github.com/pypa/pipenv)
- 功能更全的包管理工具，集成虚拟环境、支持 Lock 机制锁定安装包版本和依赖信息。当然也有坑点，可自行搜索。

#### drf
- Docs：[https://www.django-rest-framework.org/](https://www.django-rest-framework.org/)
- 基于 Django 的 REST Framework，快速实现 REST API。

#### watchdog
- GitHub：[https://github.com/gorakhargosh/watchdog](https://github.com/gorakhargosh/watchdog)
- 方便监视文件系统改动。

#### glob
- Docs：[https://docs.python.org/3/library/glob.html](https://docs.python.org/3/library/glob.html)
- 对文件的操作非常方便。

#### 2to3
- Docs：[https://docs.python.org/2/library/2to3.html](https://docs.python.org/2/library/2to3.html)
- 把 Python2 代码转成 Python3 代码。

#### glom
- GitHub：[https://github.com/mahmoud/glom](https://github.com/mahmoud/glom)
- 对 JSON 嵌套的处理非常方便。

#### pathlib
- Docs：[https://docs.python.org/3/library/pathlib.html](https://docs.python.org/3/library/pathlib.html)
- 更为方便的 Python 路径操作库。

#### environs
- GitHub：[https://github.com/sloria/environs](https://github.com/sloria/environs)
- 对于环境变量的获取非常方便，支持多种类型，如 int、bool 等。

#### pysnooper
- GitHub：[https://github.com/cool-RR/PySnooper](https://github.com/cool-RR/PySnooper)
- 非常方便简单的 Python 调试器，可以追踪到代码每一处细节的执行状态。

#### tqdm
- GitHub：[https://github.com/tqdm/tqdm](https://github.com/tqdm/tqdm)
- 进度条控制显示非常方便。

#### sh
- GitHub：[https://github.com/amoffat/sh](https://github.com/amoffat/sh)
- 对 Linux 一些命令的封装，简单好用又高效。

#### faker
- GitHub：[https://github.com/joke2k/faker](https://github.com/joke2k/faker)
- 模拟数据的生成。

#### arrow、dateutil、dateparser、pendulum
- GitHub：
- [https://github.com/crsmithdev/arrow](https://github.com/crsmithdev/arrow)
- [https://github.com/dateutil/dateutil](https://github.com/dateutil/dateutil)
- [https://github.com/scrapinghub/dateparser](https://github.com/scrapinghub/dateparser)
- [https://github.com/sdispater/pendulum](https://github.com/sdispater/pendulum)
- 时间解析和处理库，非常方便。arrow 目前 Star 最多，好评最多。

#### yagmail
- GitHub：[https://github.com/kootenpv/yagmail](https://github.com/kootenpv/yagmail)
- 方便的发邮件库，替代自带的 smtplib。

#### chardet
- GitHub：[https://github.com/chardet/chardet](https://github.com/chardet/chardet)
- 字符串类型编码检测。

#### pypinyin
- GitHub：[https://github.com/mozillazg/python-pinyin](https://github.com/mozillazg/python-pinyin)
- 汉字转拼音，在一些中文转化处理上很有用。

#### sphinx
- Docs：[https://www.sphinx-doc.org/en/master/](https://www.sphinx-doc.org/en/master/)
- 编写文档使用，大多数 Python 库文档都是这个写的，如 Scrapy、requests。
- 个人 sphinx + markdown 的样例：https://github.com/Gerapy/Docs

#### jinja2
- GitHub：[https://github.com/pallets/jinja](https://github.com/pallets/jinja)
- 一个方便的模板引擎，呈现页面时很方便。

#### click
- GitHub：[https://github.com/pallets/click](https://github.com/pallets/click)
- 更方便灵活地实现命令行传递参数。

#### ray
- GitHub：[https://github.com/ray-project/ray](https://github.com/ray-project/ray)
- 分布式多进程管理。

#### supervisor
- GitHub：[https://github.com/Supervisor/supervisor](https://github.com/Supervisor/supervisor)
- 进程管理工具，如实现多任务后台运行，Docker 打包时会经常用到。

#### apscheduler
- GitHub：[https://github.com/agronholm/apscheduler](https://github.com/agronholm/apscheduler)
- Python 定时任务，不过 K8S 也可以实现，个人目前可能更倾向于 K8S。

#### intelpython
- Home：[https://software.intel.com/en-us/distribution-for-python](https://software.intel.com/en-us/distribution-for-python)
- 这不是 Python 库，是一个 Intel 开发的基于 Intel 处理器优化的 Python 解释器，对于大规模运算提升很大。
