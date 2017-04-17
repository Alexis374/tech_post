#### logging
----
django用python的logging模块

logging分为四部分： logger handler filter formatter

##### logger
+logger是日志系统的入口。log level：DEBUG INFO WARNING ERROR
+ 一条日志记录本身有log level 和要描述事件的信息（如错误栈以及错误码）。当记录被传递到logger，需要比较两者的log level，如果消息的level等于或者超出logger的，这个消息就会被处理，否则会被忽略。
+被处理的消息会传递到handler

##### handler
+ handler决定消息怎样被处理。如打印到屏幕还是文件或者通过网络传输。
+ handler同样有级别。若记录的level小于handler的，会被忽略
+ 一个logger可以对应多个handler，每个handler可以有不同的级别。这样就可以根据消息不同的级别采取对应的措施。

##### filter
+ filter在消息从logger传递到handler时实施额外的控制。
+ 增加额外的标准，如只允许特定源头的ERROR信息被处理
+ 可以改变消息的优先级，如若满足特定条件，将ERROR降级为WARNING
+ filter可以在logger或handler上安装，多个filter可以组成filter链条来制定多个条件

##### formatter
+ 制定消息文本的格式，通常由python的格式化字符串组成。

----
##### 使用logging
+ 命名：使用模块名作为logger的名称，`logger = logging.getLogger(__name__)`，也可以输入点号分隔的字符串，`logger = logging.getLogger('project.interesting.staff')`，logger具有层次概念。`project.interesting`是`project.interesting.staff`的父模块，`project`是logger树的根。这样规定的原因是，logger可以向父模块传播。可以设置一个handler接受所有子模块的记录。当然也可以关闭传播。

+ logger方法：`debug()`,`info()`,`warning()`,`error()`,`critical()`，对应等级。`log()`人为发射一条有特定的等级的消息；`exception()`，创建一个ERROR消息，包装着当前的异常栈。

##### 在django里使用logging
+ django使用字典形式的配置。将字典赋值给`LOGGING`。
+ 默认情况下django.reqeust django.security不传播它们的log入口，下面的例子改变其行为。

```python
import os

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'DEBUG',
            'class': 'logging.FileHandler',
            'filename': '/path/to/django/debug.log',
        },
    },
    'loggers': {
        'django.request': {
            'handlers': ['file'],
            'level': 'DEBUG',
            'propagate': True,
        },
    },
}
```
+ 默认情况下，显示INFO级的信息，可以将 `DJANGO_LOG_LEVEL=DEBUG`，查看详细信息，包括数据库查询
+ 更详细的例子：
```python
import os
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '%(levelname)s %(asctime)s %(module)s %(process)d %(thread)d %(message)s'
        },
        'simple': {
            'format': '%(levelname)s %(message)s'
        },
    },
    'filters': {
        'special': {
            '()': 'project.logging.SpecialFilter',
            'foo': 'bar',
        }
    },
    'handlers': {
        'null': {
            'level': 'DEBUG',
            'class': 'logging.NullHandler',
        },
        'console': {
            'level': 'DEBUG',
            'class': 'logging.StreamHandler',
            'formatter': 'simple'
        },
        'mail_admins': {
            'level': 'ERROR',
            'class': 'django.utils.log.AdminEmailHandler',
            'filters': ['special']
        }
    },
    'loggers': {
        'django': {
            'handlers': ['null'],
            'propagate': True,
            'level': 'INFO',
        },
        'django.request': {
            'handlers': ['mail_admins'],
            'level': 'ERROR',
            'propagate': False,
        },
        'myproject.custom': {
            'handlers': ['console', 'mail_admins'],
            'level': 'INFO',
            'filters': ['special']
        }
    }
}
```
+ 上面定义了2个formatters，简单和详细，属性参考[python官方文档](https://docs.python.org/3/library/logging.html#logrecord-attributes)
+ filter名为special，是`project.logging.SpecialFilter`的实例，当需要传递特殊参数时，参数foo会被传递bar值，即`special =SpecialFilter(foo=bar)`
+ logger和handler较易理解。注意'myproject.custom'，有2个handler，console记录INFO及以上级别的，mail_admins，记录ERROR及以上级别的。

##### django内建logger

*django* 记录所有，但所有信息都不是直接被传递到这个 logger
*django.request* 记录所有request的信息，5xx被记录为ERROR，4XX被记录为WARNING，状态码和request对象同样被记录在此logger内
*django.db.backends*：每个app层级的被request对象执行的sql被记录为DEBUG信息。额外参数有duration，sql，params。只在调试模式下才启用。不包括框架级的sql，如set timezone，事务管理。要想得到这些 信息，只能打开数据库的记录了
*django.security.*,接受所有SuspiciousOperation，和其子类，级别多为warning，若操作到达了WSGI handler会变成ERROR，如客户端的HOST字段不再ALLOWED_HOST内，会返回400错误，ERROR信息会被传递给django.security.DisallowedHost。
+ 只有父级的django.security 默认被设置了。

##### handler
除了python logging模块的handler，django还增加了AdminEmailHandler,include_html设置为True，则可在邮件里看到类似调试模式下出错的页面。默认email_backend为EMAIL_BACKEND的设置。

```python
'handlers': {
    'mail_admins': {
        'level': 'ERROR',
        'class': 'django.utils.log.AdminEmailHandler',
        'include_html': True,
    }
},
```

##### filter
django额外提供了2个filter，
+ `class CallbackFilter(callback)`,接受一个callback函数，这个函数只接受一个参数，即要记录的信息。这个filter被设置后，有消息传入时调用callback函数，若callback返回False，handler不会处理。
如 过滤`UnreadablePostError`（当用户取消上传时触发），使之不向admin发邮件：
```python
from django.http import UnreadablePostError

def skip_unreadable_post(record):
    if record.exc_info:
        exc_type, exc_value = record.exc_info[:2]
        if isinstance(exc_value, UnreadablePostError):
            return False
    return True


'filters': {
    'skip_unreadable_posts': {
        '()': 'django.utils.log.CallbackFilter',
        'callback': skip_unreadable_post,
    }
},
'handlers': {
    'mail_admins': {
        'level': 'ERROR',
        'filters': ['skip_unreadable_posts'],
        'class': 'django.utils.log.AdminEmailHandler'
    }
},
```
+ RequireDebugFalse 只有当在非调试模式下才会记录  同理有RequireDebugTrue
```python
'filters': {
    'require_debug_false': {
        '()': 'django.utils.log.RequireDebugFalse',
    }
},
'handlers': {
    'mail_admins': {
        'level': 'ERROR',
        'filters': ['require_debug_false'],
        'class': 'django.utils.log.AdminEmailHandler'
    }
},
```
##### django默认logging配置
调试模式下：`django`跟踪所有INFO以上到屏幕，
非调试：django.request and django.security发送ERROR or CRITICAL信息给AdminEmailHandler，忽略所有WARNING及以下的信息。