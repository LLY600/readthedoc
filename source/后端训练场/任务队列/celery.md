# 什么是celery
## celery是什么
Celery是一个简单、灵活且可靠的，处理大量消息的分布式系统，专注于实时处理的异步任务队列，同时也支持任务调度
Celery的架构由三部分组成，消息中间件（message broker），任务执行单元（worker）和任务执行结果存储（task result store）组成。
**消息中间件**
Celery本身不提供消息服务，但是可以方便的和第三方提供的消息中间件集成。包括，RabbitMQ, Redis等等
**任务执行单元**
Worker是Celery提供的任务执行的单元，worker并发的运行在分布式的系统节点中。
**任务结果存储**
Task result store用来存储Worker执行的任务的结果，Celery支持以不同方式存储任务的结果，包括AMQP, redis等
另外， Celery还支持不同的并发和序列化的手段
**并发**：Prefork, Eventlet, gevent, threads/single threaded
**序列化**：pickle, json, yaml, msgpack. zlib, bzip2 compression， Cryptographic message signing 等等
## 使用场景
celery是一个强大的 分布式任务队列的异步处理框架，它可以让任务的执行完全脱离主程序，甚至可以被分配到其他主机上运行。我们通常使用它来实现异步任务（async task）和定时任务（crontab)。
**异步任务**：将耗时操作任务提交给Celery去异步执行，比如发送短信/邮件、消息推送、音视频处理等等
**定时任务**：定时执行某件事情，比如每天数据统计
## celery优点
**Simple(简单)**
Celery 使用和维护都非常简单，并且不需要配置文件。
**Highly Available（高可用）**
woker和client会在网络连接丢失或者失败时，自动进行重试。并且有的brokers 也支持“双主”或者“主从”的方式实现高可用。
**Fast（快速）**
单个的Celery进程每分钟可以处理百万级的任务，并且只需要毫秒级的往返延迟（使用 RabbitMQ, librabbitmq, 和优化设置时）
**Flexible（灵活）**
Celery几乎每个部分都可以扩展使用，自定义池实现、序列化、压缩方案、日志记录、调度器、消费者、生产者、broker传输等等。
## celery安装
```
 pip install celery
```
# Celery执行异步任务
## 基本使用
创建异步任务执行文件celery_task.py
```
import celery
import time

backend = 'redis://127.0.0.1:6379/1'
broker = 'redis://127.0.0.1:6379/2'

cel = celery.Celery('celery_demo', backend=backend, broker=broker)


@cel.task
def send_email(name):
    print(f'向{name}发送邮件！')
    time.sleep(2)
    print(f'向{name}发送邮件完成！')
    return 'ok'


@cel.task
def send_msg(name):
    print(f'向{name}发送信息！')
    time.sleep(2)
    print(f'向{name}发送信息完成！')
    return 'ok'
```
创建执行任务文件produce_task.py
```
from celery_task import send_email, send_msg

result1 = send_email.delay('liu')
print(result1.id)

result2 = send_email.delay('lin')
print(result2.id)

result3 = send_msg.delay('yuan')
print(result3.id)

result4 = send_msg.delay('ha')
print(result4.id)
```
命令行启动
```
celery -A celery_task worker -l INFO
```
创建查看结果文件result.py
```
from celery.result import AsyncResult
from celery_task import cel

async_result = AsyncResult(id='88901fa8-c318-4ba2-9428-1fb60c003156', app=cel)
if async_result.successful():
    result = async_result.get()
    print(result)
elif async_result.failed():
    print('failed')
elif async_result.status == 'PENDING':
    print('任务等待中')
elif async_result.status == 'RETRY':
    print('任务异常后正在重试')
elif async_result.status == 'STARTED':
    print('任务已经开始被执行')
```
## celery多任务时项目结构
```
celeryPro
	celery_tasks
		__init__.py
		celery.py
		task_01.py
		task_02.py
	celery_result.py
	produce_task.py
```
```
文件: celery.py

from celery import Celery

cel = Celery('celery_demo',
             broker='redis://127.0.0.1:6379/1',
             backend='redis://127.0.0.1:6379/2',
             # 包含以下两个任务文件，去相应的py文件中找任务，对多个任务做分类
             include=['celery_tasks.task01',
                      'celery_tasks.task02'
                      ])
# 时区
cel.conf.timezone = 'Asia/Shanghai'
# 是否使用UTC
cel.conf.enable_utc = False
```
```
文件: task01.py
import time
from celery_tasks.celery import cel


@cel.task
def send_email(name):
    time.sleep(5)
    return f"完成向{name}发送邮件任务"
```
```
文件: task02.py

import time
from celery_tasks.celery import cel


@cel.task
def send_msg(name):
    time.sleep(5)
    return f"完成向{name}发送信息任务"
```
```
文件: produce_task.py

from celery_tasks.task01 import send_email
from celery_tasks.task02 import send_msg

# 立即告知celery去执行test_celery任务，并传入一个参数
result = send_email.delay('liu')
print(result.id)
result = send_msg.delay('lin')
print(result.id)
```
```
文件: celery_result.py

from celery.result import AsyncResult
from celery_tasks.celery import cel

async_result = AsyncResult(id="2c25c916-e57c-496e-95c5-fb1c787d11ec", app=cel)

if async_result.successful():
    result = async_result.get()
    print(result)
    # result.forget() # 将结果删除,执行完成，结果不会自动删除
    # async.revoke(terminate=True)  # 无论现在是什么时候，都要终止
    # async.revoke(terminate=False) # 如果任务还没有开始执行呢，那么就可以终止。
elif async_result.failed():
    print('执行失败')
elif async_result.status == 'PENDING':
    print('任务等待中被执行')
elif async_result.status == 'RETRY':
    print('任务异常后正在重试')
elif async_result.status == 'STARTED':
    print('任务已经开始被执行')
```
# Celery执行定时任务
## 普通模式
```
file: celery_task.py

import time
from celery_tasks.celery import cel

@cel.task
def send_email(name):
    time.sleep(5)
    return f"完成向{name}发送邮件任务"
```
```
文件: produce_task.py

from celery_tasks.task01 import send_email
from celery_tasks.task02 import send_msg
from datetime import datetime, timedelta

# 方式一
# v1 = datetime(2022, 5, 25, 21, 18, 0)
# print(v1)
# v2 = datetime.utcfromtimestamp(v1.timestamp())
# print(v2)
# result = send_email.apply_async(args=['lin'], eta=v2)

# 方式二
ctime = datetime.now()
# 默认使用utc时间
utc_time = datetime.utcfromtimestamp(ctime.timestamp())
time_delay = timedelta(seconds=5)
task_time = utc_time + time_delay
print(task_time)
result = send_email.apply_async(args=['lin'], eta=task_time)
print(result.id)
```
```
celery -A tmp_task worker -l INFO
```
运行produce_task.py

## 多任务结构
```
文件: celery.py

from celery import Celery
from celery.schedules import crontab
from datetime import timedelta

cel = Celery('celery_demo',
             broker='redis://127.0.0.1:6379/1',
             backend='redis://127.0.0.1:6379/2',
             # 包含以下两个任务文件，去相应的py文件中找任务，对多个任务做分类
             include=['celery_tasks.task01',
                      'celery_tasks.task02'
                      ])

# 时区
cel.conf.timezone = 'Asia/Shanghai'
# 是否使用UTC
cel.conf.enable_utc = False

cel.conf.beat_schedule = {
    'add-every-5-seconds': {
        'task': 'celery_tasks.task01.send_email',
        # 'schedule': 1.0,
        # 'schedule': crontab(minute="*/1"),
        'schedule': timedelta(seconds=5),
        'args': ('张三',)
    },
    'add-every-8-seconds': {
        'task': 'celery_tasks.task02.send_msg',
        'schedule': timedelta(seconds=8),
        'args': ('李四',)
    },
}
```
```
celery -A celery_tasks worker -l INFO
```
```
celery  -A celery_tasks beat
```
# celery在django中使用
结构如下：
```
├─drfdemo
│  │  asgi.py
│  │  authentication.py
│  │  exceptions.py
│  │  permissions.py
│  │  settings.py
│  │  urls.py
│  │  wsgi.py
│  │  __init__.py
│
├─mycelery
│  │  config.py
│  │  main.py
│  │  settings.py
│  │  __init__.py
│  │
│  ├─email
│  │  │  tasks.py
│  │  │  __init__.py
│  ├─sms
│  │  │  tasks.py
│  │  │  __init__.py
│
├─opt
│  │  admin.py
│  │  apps.py
│  │  models.py
│  │  tests.py
│  │  urls.py
│  │  views.py
│  │  __init__.py
│  │
│  ├─migrations
│  │  │  __init__.py
```
```
文件: email/tasks.py
# celery的任务必须写在tasks.py的文件中，别的文件名称不识别!!!
from mycelery.main import app
import time
import logging
log = logging.getLogger("django")

@app.task  # name表示设置任务的名称，如果不填写，则默认使用函数名做为任务名
def send_sms(mobile):
    """发送短信"""
    print("向手机号%s发送短信成功!" % mobile)
    time.sleep(5)
    return "send_sms OK"

@app.task  # name表示设置任务的名称，如果不填写，则默认使用函数名做为任务名
def send_sms2(mobile):
    print("向手机号%s发送短信成功!" % mobile)
    time.sleep(5)
    return "send_sms2 OK"
```
```
文件: config.py
broker_url = 'redis://127.0.0.1:6379/15'
result_backend = 'redis://127.0.0.1:6379/14'
```
```
文件: main.py

# 主程序
import os
from celery import Celery

# 创建celery实例对象
app = Celery("celery_demo")

# 把celery和django进行组合，识别和加载django的配置文件
os.environ.setdefault('DJANGO_SETTINGS_MODULE','mycelery.settings')

# 通过app对象加载配置
app.config_from_object("mycelery.config")

# 加载任务
# 参数必须必须是一个列表，里面的每一个任务都是任务的路径名称
# app.autodiscover_tasks(["任务1","任务2"])
app.autodiscover_tasks(["mycelery.sms", 'mycelery.email'])

# 启动Celery的命令
# 强烈建议切换目录到mycelery根目录下启动
# celery -A mycelery.main worker --loglevel=info
```
```
文件：views.py

from mycelery.email.tasks import send_sms, send_sms2
from datetime import timedelta, datetime

def celery_demo(request):
    send_sms.delay("110")
    send_sms2.delay("119")
    return HttpResponse('ok')

	ctime = datetime.now()
    # 默认用utc时间
    utc_ctime = datetime.utcfromtimestamp(ctime.timestamp())
    time_delay = timedelta(seconds=10)
    task_time = utc_ctime + time_delay
    result = send_sms.apply_async(["911", ], eta=task_time)
    print(result.id)
```
```
celery -A mycelery.main worker -l INFO
```