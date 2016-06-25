# 快速入门 

---

我们将创建一个简单的API去让管理员可以管理这个系统的用户和用户组。 

 * [项目安装](#项目安装)
 * [Serizlizers](#serizlizers) 
 * [View](#view)
 * [URLs](#urls)
 * [设置](#设置)
 * [测试](#测试)


### 项目安装 
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

这里把所有公共的方法(get, post, delete)都封装到了viewsets里面了，你可以很容易的把这些分离出来，全部写到view中，但是明显使用viewsets的代码更简洁明了。

### URLs
待编辑

### 设置
待编辑

### 测试
待编辑

  [1]: http://askubuntu.com/questions/244641/how-to-set-up-and-use-a-virtual-python-environment-in-ubuntu
