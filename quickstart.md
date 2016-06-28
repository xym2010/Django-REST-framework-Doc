# 快速入门 

---

我们将创建一个简单的API去让管理员可以管理这个系统的用户和用户组。 

 * [Install](#install)
 * [Serizlizers](#serizlizers) 
 * [View](#view)
 * [URLs](#urls)
 * [Setting](#setting)
 * [Testing](#testing)


### Install
创建一个新的Django项目叫做tutorial，接着开启一个新的app叫做quickstart。

```python
# 创建项目目录
mkdir tutorial
cd tutorial

# 创建虚拟环境中隔离我们的依赖包
virtualenv env
source env/bin/activate  # On Windows use `env\Scripts\activate`

# 安装
pip install django
pip install djangorestframework

# 初始化项目和app
django-admin.py startproject tutorial .  # 注意"."是命令的一部分
cd tutorial
django-admin.py startapp quickstart
cd ..
```
> 这里虚拟环境可以用virtualenvwrapper来管理虚拟环境，安装方法看[这里][1]

接着，同步数据库

```python
python manage.py migrate
```

创建一个初始用户叫做admin，密码叫password123，我们稍后会用这个用户来验证我们的例子。

```python
python manage.py createsuperuser
```
> 以上安装的是Django和项目的初始化，记得还需要按照上一章的安装流程安装配置好REST framework。

### Setting
接着添加必要的配置，在tutorial/settings.py中添加以下配置。

```python
INSTALLED_APPS = (
     ...
     'rest_framework',
)

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': ('rest_framework.permissions.IsAdminUser',),
    'PAGE_SIZE': 10
}
```

### Serizlizers

我们先定义一下Serizlizers，我们先创建一个新的模块 tutorial/quickstart/serializers.py，用来表示我们的数据。

```python
from django.contrib.auth.models import User, Group
from rest_framework import serializers


class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ('url', 'username', 'email', 'groups')


class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ('url', 'name')

```
这里我们使用了HyperlinkedModelSerializer的超链接关系，我们也可以使用主键和其他不同的关系表示，但是超链接是一个很好的RESTful的设计。

> 什么是超链接的设计呢？就是一个url对应一个服务器的资源。有兴趣的可以去看看《RESTful Web APIs》

### View

接着添加view，编辑tutorial/quickstart/views.py加入以下代码

```python
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from tutorial.quickstart.serializers import UserSerializer, GroupSerializer


class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer


class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited.
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
```

这里把所有公共的方法（get, post, delete）都封装到了viewsets里面了，你可以很容易的把这些分离出来，全部写到view中，但是明显使用viewsets的代码更简洁明了。

### URLs
接着加入API URLs，编辑tutorial/urls.py文件如下：

```python
from django.conf.urls import url, include
from rest_framework import routers
from tutorial.quickstart import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# Wire up our API using automatic URL routing.
# Additionally, we include login URLs for the browsable API.
urlpatterns = [
    url(r'^', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

因为我们用了viewsets代替了views，所以只要简单的注册一下router对象就OK了。  
同样，如果你需要更好掌控API URLs，你可以很容易的分离出来单独处理。  
最后，为了正常的使用可视化的API界面，我们引入REST framework自带的login和logout的views（就是上面的api-auth），这些可视化操作有利于验证你的API。

### Testing
我们现在可以开始测试我们的API了，先启动项目。

```python
python manage.py runserver
```

** curl访问 **

```python
bash: curl -H 'Accept: application/json; indent=4' -u xuyiming:password123 http://127.0.0.1:8000/users/
[
    {
        "url": "http://127.0.0.1:8000/users/2/",
        "username": "admin",
        "email": "admin@163.com",
        "groups": []
    },
    {
        "url": "http://127.0.0.1:8000/users/1/",
        "username": "xuyiming",
        "email": "86021111@163.com",
        "groups": []
    }
]

```
> curl -H 是添加请求头参数，-u 是输入页面的验证账号和密码，注意这里要把xuyiming:password123 改成你之前创建的管理员的账号和密码。

** 使用httpie 工具访问 **

```python
bash: http -a admin:password123 http://127.0.0.1:8000/users/

HTTP/1.0 200 OK
Allow: GET, POST, HEAD, OPTIONS
Content-Type: application/json
Date: Tue, 28 Jun 2016 12:18:38 GMT
Server: WSGIServer/0.1 Python/2.7.10
Vary: Accept, Cookie
X-Frame-Options: SAMEORIGIN

[
    {
        "email": "admin@163.com",
        "groups": [],
        "url": "http://127.0.0.1:8000/users/2/",
        "username": "admin"
    },
    {
        "email": "86021111@163.com",
        "groups": [],
        "url": "http://127.0.0.1:8000/users/1/",
        "username": "xuyiming"
    }
]
```
** 使用浏览器直接访问 **

![testing][2]

如果你的结果和以上一样，那么恭喜你，如果你先想要深入学习，你可以看接下来的教程或者API指南。


  [1]: http://askubuntu.com/questions/244641/how-to-set-up-and-use-a-virtual-python-environment-in-ubuntu
  [2]: http://7xq2as.com1.z0.glb.clouddn.com/quickstart-test.png
