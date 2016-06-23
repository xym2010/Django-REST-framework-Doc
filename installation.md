# 安装

---
## 安装依赖 ##
REST framework 需要以下的支持：

 - Python (2.7, 3.2, 3.3, 3.4, 3.5)
 - Django (1.7+, 1.8, 1.9)  
  
以下包是可选的：

 - [Markdown][1] (2.1.0+) - Markdown 为了更好的显示API结构。
 - [django-filter][2] (0.9.2+) - 过滤器支持。
 - [django-crispy-forms][3] - 提升过滤器在HTML页面中的显示。
 - [django-guardian][4] (1.1.1+) - 对象级的权限管理支持。

## 安装 ##

用pip安装  

```python
pip install djangorestframework  # 安装django rest framework
pip install markdown       # 安装markdown支持
pip install django-filter  # 安装过滤器支持
```
或者用git clone 安装

```
git clone git@github.com:tomchristie/django-rest-framework.git
```
  
在django 项目的settings.py 配置文件中添加**'rest_framework'**

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
)
```
如果你想要使用可以查看的API界面，需要添加REST framework的登陆和登出模块。只要把下面内容添加到你的根url.py的配置文件。

```python
urlpatterns = [
    ...
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

> **注意：这里的url路径可以是你自定义的，但是要保证include了rest_framework.urls 和使用rest_framework的namespace，如果你是Django1.9+，你可以省略namespace，因为REST framework会帮你设定。**

## 实例 ##
让我们看个使用REST framework搭建的实例。   
我们将创建一个可读写的API来获取访问这个项目的用户信息。  
任何REST framework通用的配置都是保存在一个叫REST_FRAMEWORK的字典中。那么把下面的代码加入到你的settings.py中。

```python
REST_FRAMEWORK = {
    # Use Django's standard `django.contrib.auth` permissions,
    # or allow read-only access for unauthenticated users.
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
    ]
}
```

不要忘记确保你已经在配置文件中的INSTALLED_APPS添加了rest_framework。  
我们现在准备创建我们的API了，下面是我们的根urls.py文件：

```python
from django.conf.urls import url, include
from django.contrib.auth.models import User
from rest_framework import routers, serializers, viewsets

# Serializers define the API representation.
class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ('url', 'username', 'email', 'is_staff')

# ViewSets define the view behavior.
class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer

# Routers provide an easy way of automatically determining the URL conf.
router = routers.DefaultRouter()
router.register(r'users', UserViewSet)

# Wire up our API using automatic URL routing.
# Additionally, we include login URLs for the browsable API.
urlpatterns = [
    url(r'^', include(router.urls)),
    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```
你可以打开你的浏览器通过 http://127.0.0.0:8000/ 访问你的API了，你可以看到一个叫'users'的API，如果你登陆了，你可以通过这个接口创建，修改和删除用户。  

大概就是下面这样。
![此处输入图片的描述][5]


  [1]: http://pypi.python.org/pypi/Markdown/
  [2]: http://pypi.python.org/pypi/django-filter
  [3]: https://github.com/maraujop/django-crispy-forms
  [4]: https://github.com/lukaszb/django-guardian
  [5]: http://7xq2as.com1.z0.glb.clouddn.com/install.png
