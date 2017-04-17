#### 用户鉴权
authentication：鉴定用户是谁
authorization：允许用户可以做什么
auth系统包括：
+ 用户
+ 权限
+ 组
+ 可配置的密码哈希系统
+ 登陆或限制的表单以及view工具
+ 可扩展的后台系统
不包括（由第三方提供）：
+ 密码长度检测
+ 限制登陆尝试
+ 第三方登陆，如OAuth

##### User对象
属性 username，password，email，first_name,last_name等
api：
```python
>>> from django.contrib.auth.models import User
>>> user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')

>>> user.last_name = 'Lennon'
>>> user.save()

改密码
>>> u = User.objects.get(username='john')
>>> u.set_password('new password')
>>> u.save()

认证用户
from django.contrib.auth import authenticate
user = authenticate(username='john', password='secret')
if user is not None:
    # the password verified for the user
    if user.is_active:
        print("User is valid, active and authenticated")
    else:
        print("The password is valid, but the account has been disabled!")
else:
    # the authentication system was unable to verify the username and password
    print("The username and password were incorrect.")
```

----
#### 权限和授权
+ admin权限有三类，add，change，delete，可以对model进行更改
+ User对象有`groups`和`user_permissions`两个多对多字段，可以进行`add`,`remove`,`clear`操作
+ 在运行 manage.py migrate时，会为每个安装的model增加默认的权限
+ 测试单个用户是否有 app_label为foo，model为 Bar的权限：
```python
user.has_perm(foo.add_bar)
user.has_perm(foo.change_bar)
user.has_perm(foo.delete_bar)
```

-----
#### 组
给用户分组可以更方便地管理权限，在组里的用户自动拥有组拥有的权限；分组还可以方便地扩展相应功能，如给Special user组里的人发邮件等

用代码创建权限：
```python
from myapp.models import BlogPost
from django.contrib.auth.models import Group, Permission
from django.contrib.contenttypes.models import ContentType

content_type = ContentType.objects.get_for_model(BlogPost)
permission = Permission.objects.create(codename='can_publish',
                                       name='Can Publish Posts',
                                       content_type=content_type)
```
所创建的权限，可以通过User.user_permissions 或Group的permissions 加给用户或组

##### 权限缓存
第一次对User进行权限检查（use.has_perm()）的时候，会对权限进行缓存，因为在一个请求响应周期通常不会对权限立即做检查。
如果非要这样做，可以重新获取user对象
