# somedecorators

Some very useful Python decorators, functions, classes, continuously updated.


一些非常实用的 Python 装饰器、函数、类，持续更新。

- 新增 robot_on_exception，报错通过企业微信机器人发送至企业微信群聊 :  from somedecorators import robot_on_exception
- 新增 wechat_on_exception，报错发送企业微信 :  from somedecorators import wechat_on_exception
- 新增 setup_logger，快速配置日志  : from somedecorators import setup_logger  
- 新增 ConfigManager，快速搞定配置文件 : from somedecorators import ConfigManager 

使用方法如下：

## 安装

```sh
pip install somedecorators
```

## 装饰器介绍：

#### 发送企业微信机器人

调用代码 
mentioned_list 参数可以不填写。
```python
@robot_on_exception(webhook_url= "你的企业威胁你机器人 webhook",mentioned_list=["@谁就填写谁的企业微信账号","user2"], extra_msg="运行报错")
def myfunc(args):
    if args == 1:
        raise Exception1
    elif args == 2:
        raise Exception2
    else:
        raise Exception3
```



#### timeit

耗时统计装饰器，单位是秒，保留 4 位小数


使用方法：

```python
from somedecorators import timeit
@timeit()
def test_timeit():
    time.sleep(1)

#test_timeit cost 1.0026 seconds

@timeit(logger = your_logger)
def test_timeit():
    time.sleep(1)
```


#### timeout

超时装饰器，单位是秒，函数运行超过指定的时间会抛出 TimeOutError 异常。

使用方法：

```python
import time
from somedecorators import timeout
@timeout(2)
def test_timeit():
    time.sleep(3)

#somedecorators.timeit.TimeoutError: Operation did not finish within 2 seconds
```



#### retry

重试装饰器
- 当被装饰的函数调用抛出指定的异常时，函数会被重新调用。
- 直到达到指定的最大调用次数才重新抛出指定的异常，可以指定时间间隔，默认 5 秒后重试。
- traced_exceptions 为监控的异常，可以为 None（默认）、异常类、或者一个异常类的列表或元组 tuple。
- traced_exceptions 如果为 None，则监控所有的异常；如果指定了异常类，则若函数调用抛出指定的异常时，重新调用函数，直至成功返回结果。
- 未出现监控的异常时，如果指定定了 reraised_exception 则抛出 reraised_exception，否则抛出原来的异常。


```python
from somedecorators import retry 

@retry(
    times=2,
    wait_seconds=1,
    traced_exceptions=myException,
    reraised_exception=CustomException,
)
def test_retry():
    # time.sleep(1)
    raise myException


test_retry()
```



#### email_on_exception

报错发邮件装饰器。当被装饰的函数调用抛出指定的异常时，函数发送邮件给指定的人员，使用独立的 [djangomail](https://github.com/somenzz/djangomail) 发邮件模块，非常好用。

- recipient_list: 一个字符串列表，每项都是一个邮箱地址。recipient_list 中的每个成员都可以在邮件的 "收件人:" 中看到其他的收件人。
- traced_exceptions 为监控的异常，可以为 None（默认）、异常类、或者一个异常类的元组。
traced_exceptions 如果为 None，则监控所有的异常；如果指定了异常类，则若函数调用抛出指定的异常时，发送邮件。

**使用方法**：

首先在项目目录新建 settings.py，配置邮件服务器或企业微信，内容如下：

```python
EMAIL_USE_LOCALTIME = True

#for unitest
#EMAIL_BACKEND = 'djangomail.backends.console.EmailBackend'
#EMAIL_BACKEND = 'djangomail.backends.smtp.EmailBackend'
EMAIL_USE_SSL = True
EMAIL_HOST = 'smtp.163.com' #可以换其他邮箱
EMAIL_PORT = 465
EMAIL_HOST_USER = 'your-username'
EMAIL_HOST_PASSWORD = '********'
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER
SERVER_EMAIL = EMAIL_HOST_USER


# 用于发送企业微信
CORPID="**********************"  # 企业 ID
APPID="*******"  # 企业应用 ID
CORPSECRET="************************" # 企业应用 Secret

```

如果你的文件名不是 settings.py，假如是 mysettings.py 则需要修改环境变量:

```python
os.environ.setdefault("SETTINGS_MODULE", "mysettings")
```
然后主程序中这样使用：

##### 监控所有的异常

```python
from somedecorators import email_on_exception 
#import os
#os.environ.setdefault("SETTINGS_MODULE", "settings") #默认配置，可以不写此行代码

@email_on_exception(['somenzz@163.com'])
def myfunc(arg):
    1/arg

myfunc(0)
```

你会收到如下的邮件信息，非常便于排查错误。

```sh
Subject: myfunc(arg=0) raise Exception
From: your-username
To: somenzz@163.com
Date: Fri, 11 Jun 2021 20:55:01 -0500
Message-ID: 
 <162346290112.13869.15957310483971819045@1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa>

myfunc(arg=0) raise Exception: division by zero 
 traceback:
 Traceback (most recent call last):
  File "/Users/aaron/github/somenzz/somedecorators/somedecorators/email.py", line 35, in wrapper
    return func(*args, **kwargs)
  File "/Users/aaron/github/somenzz/somedecorators/tests/tests.py", line 55, in myfunc
    return 1/arg
ZeroDivisionError: division by zero

extra_msg = 严重错误
```

##### 监控指定的异常

```python

from somedecorators import email_on_exception
import os
os.environ.setdefault("SETTINGS_MODULE", "settings")

class Exception1(Exception):
    pass

class Exception2(Exception):
    pass

class Exception3(Exception):
    pass

@email_on_exception(['somenzz@163.com'],traced_exceptions = Exception2)
def myfunc(args):
    if args == 1:
        raise Exception1
    elif args == 2:
        raise Exception2
    else:
        raise Exception3

myfunc(2)

```
上述代码只有在 raise Exception2 时才会发送邮件：

##### 不同的异常发给不同的人

```python
@email_on_exception(['somenzz@163.com'],traced_exceptions = Exception2)
@email_on_exception(['others@163.com'],traced_exceptions = (Exception1, Exception3))
def myfunc(args):
    if args == 1:
        raise Exception1
    elif args == 2:
        raise Exception2
    else:
        raise Exception3
```

是不是非常方便？

#### 发送企业微信

发送前需要在 settings.py 文件企业微信相关信息

settings.py 示例：

```python

CORPID="**********************"  # 企业 ID
APPID="*******"  # 企业应用 ID
CORPSECRET="************************" # 企业应用 Secret

```

调用代码 

```python
@wechat_on_exception(['企业微信接收者ID'],traced_exceptions = Exception2)
def myfunc(args):
    if args == 1:
        raise Exception1
    elif args == 2:
        raise Exception2
    else:
        raise Exception3
```


#### 快速配置日志 setup_logger

一个简单的使用日志的方法

```python
from somedecorators import setup_logger
logger = setup_logger("myapp")
logger.info("hello this is myapp log")
logger2 = setup_logger("myapp2", log_path="./log/app.log")
logger3 = setup_logger("myapp3",handlers=['console']) 
logger4 = setup_logger("myapp4",handlers=['console','file']) 
```

#### 快速搞定配置文件

```python
from somedecorators import ConfigManager
config_manager = ConfigManager("config.yml")
config_data = config_manager.get_all()
print(config_data)
```

## 参与项目

欢迎分享你最常用的装饰器、类、函数，加入到这里。

## 联系我 

微信：somenzz-enjoy
