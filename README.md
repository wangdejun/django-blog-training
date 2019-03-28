
# 1.搭建开发环境

### 1.1 安装 Python


```python
pip install python
```

### 1.2 使用虚拟环境 Virtualenv

* 安装vitualenv
```
pip install virtualenv
```

* 为vitualenv指定python版本

* 激活vitualenv

```python
# macos系统下
source ../../bin/activate
## 在我的机器上是这个路径
source ~/Desktop/python-course/Course/python_code/python-course-env/bin/activate
```
windows系统下
```python
运行script脚本
```

### 1.3 安装 Django


```python
pip install django
```

### 1.4 建立 Django 工程

#### 1.4.1 windows先配置环境变量
* 首先激活虚拟环境


```python
(python-course-env) django-admin startproject project_name
```

### 1.5 Hello Django

blogs/urls.py
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

blog/urls.py¶
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

把 LANGUAGE_CODE 的值改为 zh-hans，TIME_ZONE 的值改为 Asia/Shanghai：


```python
blogproject/blogproject/settings.py

## 其它配置代码...

# 把英文改为中文
LANGUAGE_CODE = 'zh-hans'

# 把国际时区改为中国时区
TIME_ZONE = 'Asia/Shanghai'

## 其它配置代码...
```

# 2.建立博客应用


```python
python manage.py startapp blog
```


```python
blog\
    __init__.py
    admin.py
    apps.py
    migrations\
        __init__.py
    models.py
    tests.py
    views.py
```


```python
blogproject/blogproject/settings.py

## 其他配置项...

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog', # 注册 blog 应用
]

## 其他配置项...
```

# 3.创建Django博客的数据模型

设计博客的数据库表结构


```python
blog/models.py

from django.db import models
from django.contrib.auth.models import User


class Category(models.Model):
    """
    Django 要求模型必须继承 models.Model 类。
    Category 只需要一个简单的分类名 name 就可以了。
    CharField 指定了分类名 name 的数据类型，CharField 是字符型，
    CharField 的 max_length 参数指定其最大长度，超过这个长度的分类名就不能被存入数据库。
    当然 Django 还为我们提供了多种其它的数据类型，如日期时间类型 DateTimeField、整数类型 IntegerField 等等。
    Django 内置的全部类型可查看文档：
    https://docs.djangoproject.com/en/1.10/ref/models/fields/#field-types
    """
    name = models.CharField(max_length=100)


class Tag(models.Model):
    """
    标签 Tag 也比较简单，和 Category 一样。
    再次强调一定要继承 models.Model 类！
    """
    name = models.CharField(max_length=100)


class Post(models.Model):
    """
    文章的数据库表稍微复杂一点，主要是涉及的字段更多。
    """

    # 文章标题
    title = models.CharField(max_length=70)
    # 文章正文，我们使用了 TextField。
    # 存储比较短的字符串可以使用 CharField，但对于文章的正文来说可能会是一大段文本，因此使用 TextField 来存储大段文本。
    body = models.TextField()
    # 这两个列分别表示文章的创建时间和最后一次修改时间，存储时间的字段用 DateTimeField 类型。
    created_time = models.DateTimeField()
    modified_time = models.DateTimeField()
    # 文章摘要，可以没有文章摘要，但默认情况下 CharField 要求我们必须存入数据，否则就会报错。
    # 指定 CharField 的 blank=True 参数值后就可以允许空值了。
    excerpt = models.CharField(max_length=200, blank=True)
    # 这是分类与标签，分类与标签的模型我们已经定义在上面。
    # 我们在这里把文章对应的数据库表和分类、标签对应的数据库表关联了起来，但是关联形式稍微有点不同。
    # 我们规定一篇文章只能对应一个分类，但是一个分类下可以有多篇文章，所以我们使用的是 ForeignKey，即一对多的关联关系。
    # 而对于标签来说，一篇文章可以有多个标签，同一个标签下也可能有多篇文章，所以我们使用 ManyToManyField，表明这是多对多的关联关系。
    # 同时我们规定文章可以没有标签，因此为标签 tags 指定了 blank=True。
    # 如果你对 ForeignKey、ManyToManyField 不了解，请看教程中的解释，亦可参考官方文档：
    # https://docs.djangoproject.com/en/1.10/topics/db/models/#relationships
    category = models.ForeignKey(Category)
    tags = models.ManyToManyField(Tag, blank=True)
    # 文章作者，这里 User 是从 django.contrib.auth.models 导入的。
    # django.contrib.auth 是 Django 内置的应用，专门用于处理网站用户的注册、登录等流程，User 是 Django 为我们已经写好的用户模型。
    # 这里我们通过 ForeignKey 把文章和 User 关联了起来。
    # 因为我们规定一篇文章只能有一个作者，而一个作者可能会写多篇文章，因此这是一对多的关联关系，和 Category 类似。
    author = models.ForeignKey(User)
```

# 4.迁移数据库(让Django帮我们完成翻译)

### 4.1 迁移


```python
# 创建改变
python manage.py makemigrations
# 查看sql
python manage.py sqlmigrate blog 0001
# 真正迁移数据库
python manage.py migrate
```

### 4.2 play with django shell


```python
# 创建一个超级用户
python manage.py createsuperuser

# 开启动 shell
python manage.py shell

>>>from blog.models import Category, Tag, Post
>>>from django.utils import timezone
>>>from django.contrib.auth.models import User

# >>>Category.objects.all()
>>>c = Category.objects.get(name="blog")
>>>t = Tag.objects.get(name="Django")
>>>user = User.objects.get(pk=1)  # User.objects.get(pk=1)
>>>post = Post(title="第一篇博客", body="第一篇博客内容第一篇博客内容",created_time=timezone.now(), modified_time=timezone.now(), category=c, author=user)
>>>post.save()


```

post.save()之后, 查看数据库，发现blog_post表中多出来一条数据

### 4.3 运行结果

* 1.激活虚拟环境，
* 2.运行 python manage.py runserver 打开开发服务器，
* 3.在浏览器输入开发服务器的地址 http://127.0.0.1:8000/

### 4.4 使用Django模板系统

#建立模板文件夹
myprojects/template/blog/index.html
#目录如下
blogproject\
    manage.py
    blogproject\
        __init__.py
        settings.py
        ...
    blog\
        __init__.py
        models.py
        ...
    templates\
        blog\
            index.html

再一次强调 templates\ 目录位于项目根目录，而 index.html 位于 templates\blog 目录下，而不是 blog 应用下，如果弄错了你可能会得到一个TemplateDoesNotExist 异常。如果遇到这个异常，请回来检查一下模板目录结构是否正确。


```python
# templates\blog\index.html里面写入下面的内容
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
<h1>{{ welcome }}</h1>
</body>
</html>
```

这是一个标准的 HTML 文档，只是里面有两个比较奇怪的地方：{{ title }}，{{ welcome }}。这是 Django 规定的语法。用 {{ }} 包起来的变量叫做模板变量。Django 在渲染这个模板的时候会根据我们传递给模板的变量替换掉这些变量。最终在模板中显示的将会是我们传递的值。


```python
blogproject/settings.py

TEMPLATES = [
    {
        ...
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        ...
    },
]
```


```python
#视图函数可以改一下
blog/views.py

from django.http import HttpResponse
from django.shortcuts import render

def index(request):
    return render(request, 'blog/index.html', context={
                      'title': '我的博客首页', 
                      'welcome': '欢迎访问我的博客首页'
                  })
```

我们首先把 HTTP 请求传了进去，然后 render 根据第二个参数的值 blog/index.html 找到这个模板文件并读取模板中的内容。之后 render 根据我们传入的 context 参数的值把模板中的变量替换为我们传递的变量的值，{{ title }} 被替换成了 context 字典中 title 对应的值，同理 {{ welcome }} 也被替换成相应的值。


最终，我们的 HTML 模板中的内容字符串被传递给 HttpResponse 对象并返回给浏览器（Django 在 render 函数里隐式地帮我们完成了这个过程），这样用户的浏览器上便显示出了我们写的 HTML 模板的内容。

### 4.5 再次运行结果

* 1.激活虚拟环境（optional）
* 2.运行 python manage.py runserver 打开开发服务器，
* 3.在浏览器输入开发服务器的地址 http://127.0.0.1:8000/


```python
可以看到首页

欢迎访问我的博客首页!
```

### 4.6 注意事项
* 1.index.html 必须以 UTF-8 的编码格式保存，且小心不要往里面添加一些特殊字符，否则极有可能得到一个 UnicodeDecodeError 这样的错误。
* 2.templates\ 目录位于项目根目录，而 index.html 位于 templates\blog 目录下，而不是 blog 应用下，如果弄错了可能会得到一个TemplateDoesNotExist 异常。如果遇到这个异常，请回来检查一下<strong style="color:red">模板目录结构是否正确</strong>。
* 3.render函数与return的相通之处：=>最终，我们的 HTML 模板中的内容字符串被传递给 HttpResponse 对象并返回给浏览器<strong style="color:red">（Django 在 render 函数里隐式地帮我们完成了这个过程）</strong>，这样用户的浏览器上便显示出了我们写的 HTML 模板的内容。

# 5.真正Django blog的首页

### 5.1 首页视图函数


```python
blog/views.py

from django.shortcuts import render
from .models import Post

def index(request):
    post_list = Post.objects.all().order_by('-created_time')
    return render(request, 'blog/index.html', context={'post_list': post_list})
```

* 这里我们使用 all() 方法从数据库里获取了全部的文章，存在了 post_list 变量里。

* all 方法返回的是一个 QuerySet（可以理解成一个类似于列表的数据结构），由于通常来说博客文章列表是按文章发表时间倒序排列的，即最新的文章排在最前面，所以我们紧接着调用了 order_by方法对这个返回的queryset进行排序。

* 排序依据的字段是 created_time，即文章的创建时间。- 号表示逆序，如果不加 - 则是正序。 

* 接着如之前所做，我们渲染了 blog\index.html 模板文件，并且把包含文章列表数据的 post_list 变量传给了模板。

### 5.2 修改模板


```python
templates/blog/index.html
<article class="post post-1">
  ...
</article>

改成如下

...
{% for post in post_list %}
  <article class="post post-{{ post.pk }}">
    ...
  </article>
{% empty %}
  <div class="no-post">暂时还没有发布的文章！</div>
{% endfor %}
...
```

### 5.2处理静态文件

#### 5.2.1下载静态文件
我们的项目使用了从网上下载的一套博客模板https://github.com/wangdejun/django-blog-tutorial-templates， 最终我们的 blog 应用目录结构应该是这样的：


```python
blog\
    __init__.py
    static\
        blog\
            css\
                .css 文件...
            js\
                .js 文件...
    admin.py
    apps.py
    migrations\
        __init__.py
    models.py
    tests.py
    views.py
```

用下载的博客模板中的 index.html 文件替换掉之前我们自己写的 index.html 文件。如果你好奇，现在就可以运行开发服务器，看看首页是什么样子。

<img src="https://bkt.zmrenwu.com/未正确引入静态资源的博客首页.png"/>

#### 5.2.2 将静态文件放在blog app下面,目录结构如下


```python
├── __init__.py
├── admin.py
├── apps.py
├── models.py
├── static
│   ├── css
│   ├── img
│   └── js
├── tests.py
├── urls.py
└── views.py
```

#### 5.2.3 修改template/blog/index.html文件


```python
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <title>Black &amp; White</title>

    <!-- meta -->
    <meta charset="UTF-8">s
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <!-- css -->
    <link rel="stylesheet" href="{% static 'css/bootstrap.min.css' %}">
```

### 5.3 要点总结

* Make sure that django.contrib.staticfiles is included in your INSTALLED_APPS.

* In your settings file, define STATIC_URL, for example:
```python
STATIC_URL = '/static/'
```
* In your templates, use the static template tag to build the URL for the given relative path using the configured STATICFILES_STORAGE.
```html
{% load static %}
<img src="{% static "my_app/example.jpg" %}" alt="My image">
```

* Store your static files in a folder called static in your app. For 
```html
example my_app/static/my_app/example.jpg.
```


# 6-X. 在 Django Admin 后台发布文章

## 6.1 创建 Admin 后台管理员账户


```python
python manage.py createsuperuser

Username (leave blank to use 'zmrenwu@163.com'):  admin
Email address:  admin@example.com
Warning: Password input may be echoed.
Password:  ******
Warning: Password input may be echoed.
Password (again):  ******
Superuser created successfully.
```

## 6.2 在 Admin 后台注册模型


```python
blog/admin.py

from django.contrib import admin
from .models import Post, Category, Tag

admin.site.register(Post)
admin.site.register(Category)
admin.site.register(Tag)
```

## 6.3 定制Admin后台


```python
blog/admin.py

from django.contrib import admin
from .models import Post, Category, Tag

class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'created_time', 'modified_time', 'category', 'author']

# 把新增的 PostAdmin 也注册进来
admin.site.register(Post, PostAdmin)
admin.site.register(Category)
admin.site.register(Tag)
```

* 定制完毕

# 7 博客文章详情页

## 7.1 设计文章详情页的 URL


```python
blog/urls.py

from django.urls import path
from . import views

# 注意些上命名空间
app_name="blog"

urlpatterns = [
    path('', views.index, name="index"),
    path('post/<str:pk>/', views.detail, name='detail')
]

```


```python
blog/models.py

from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse
from django.utils.six import python_2_unicode_compatible

@python_2_unicode_compatible
class Post(models.Model):
    ...

    def __str__(self):
        return self.title

    # 自定义 get_absolute_url 方法
    # 记得从 django.urls 中导入 reverse 函数
    def get_absolute_url(self):
        return reverse('blog:detail', kwargs={'pk': self.pk})
```

## 7.2 编写 detail 视图函数


```python
blog/views.py

from django.shortcuts import render, get_object_or_404
from .models import Post

def index(request):
    # ...

def detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'blog/detail.html', context={'post': post})
```

## 7.3 编写详情页模板

### 7.3.1 将 single.html改为detail.html


```python
blogproject\
    manage.py
    blogproject\
        __init__.py
        settings.py
        ...
    blog/
        __init__.py
        models.py
        ,,,
    templates\
        blog\
            index.html
            detail.html
```


```python
templates/blog/index.html

<article class="post post-1">
  <header class="entry-header">
    <h1 class="entry-title">
      <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
    </h1>
    ...
  </header>
  <div class="entry-content clearfix">
    ...
    <div class="read-more cl-effect-14">
      <a href="{{ post.get_absolute_url }}" class="more-link">继续阅读 <span class="meta-nav">→</span></a>
    </div>
  </div>
</article>
{% empty %}
  <div class="no-post">暂时还没有发布的文章！</div>
{% endfor %}
```

# 这是一个标题
* list 
* list
* list

## 7.4 模板继承


```python
templates/base.html

...
<main class="col-md-8">
    {% block main %}
    {% endblock main %}
</main>
<aside class="col-md-4">
  {% block toc %}
  {% endblock toc %}
  ...
</aside>
...
```

### 7.4.1 分别在index.html里，和detail.html里继承模板


```python
{% extends "base.html" %}
```

# 8 使用markdown语法和代码高亮

为了让博客文章具有良好的排版，显示更加丰富的格式，我们使用 Markdown 语法来书写我们的博文。Markdown 是一种 HTML 文本标记语言，只要遵循它约定的语法格式，Markdown 的渲染器就能够把我们写的文章转换为标准的 HTML 文档，从而让我们的文章呈现更加丰富的格式，例如标题、列表、代码块等等 HTML 元素。由于 Markdown 语法简单直观，不用超过 5 分钟就可以掌握常用的标记语法，因此大家青睐使用 Markdown 书写 HTML 文档。下面让我们的博客也支持使用 Markdown 书写。

### 8.1 安装


```python
blog/views.py

import markdown
from django.shortcuts import render, get_object_or_404
from .models import Post

def detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    # 记得在顶部引入 markdown 模块
    post.body = markdown.markdown(post.body,
                                  extensions=[
                                     'markdown.extensions.extra',
                                     'markdown.extensions.codehilite',
                                     'markdown.extensions.toc',
                                  ])
    return render(request, 'blog/detail.html', context={'post': post})
```

### 8.2 safe filter

## 8.3 代码高亮


```python
8.3.1 安装
```


```python
templates/base.html

...
<link rel="stylesheet" href="{% static 'blog/css/pace.css' %}">
<link rel="stylesheet" href="{% static 'blog/css/custom.css' %}">
...
+ <link rel="stylesheet" href="{% static 'blog/css/highlights/github.css' %}">
```

页面侧边栏使用自定义标签

## 9.1 使用模板标签

### 9.1 模板标签目录结构


```python
建立新的模板标签包
```


```python
blog\
    __init__.py
    admin.py
    apps.py
    migrations\
        __init__.py
    models.py
    static\
    templatetags\
        __init__.py
        blog_tags.py
    tests.py
    views.py
```

### 9.2编写模板标签代码


```python
# 最新文章
from django import template
from ..models import Post

register = template.Library()

@register.simple_tag
def get_recent_posts(num=5):
    return Post.objects.all().order_by('-created_time')[:num]
```


```python
# 归档
blog/templatetags/blog_tags.py

@register.simple_tag
def archives():
    return Post.objects.dates('created_time', 'month', order='DESC')
```


```python
# 分类
blog/templatetags/blog_tags.py

from ..models import Post, Category

@register.simple_tag
def get_categories():
    # 别忘了在顶部引入 Category 类
    return Category.objects.all()
```

## 9.3 使用自定义的模板标签


```python
templates/base.html

<div class="widget widget-recent-posts">
  <h3 class="widget-title">最新文章</h3>
  {% get_recent_posts as recent_post_list %}
  <ul>
    {% for post in recent_post_list %}
    <li>
      <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
    </li>
    {% empty %}
    暂无文章！
    {% endfor %}
  </ul>
</div>

<div class="widget widget-archives">
  <h3 class="widget-title">归档</h3>
  {% archives as date_list %}
  <ul>
    {% for date in date_list %}
    <li>
      <a href="#">{{ date.year }} 年 {{ date.month }} 月</a>
    </li>
    {% empty %}
    暂无归档！
    {% endfor %}
  </ul>
</div>

<div class="widget widget-category">
  <h3 class="widget-title">分类</h3>
  {% get_categories as category_list %}
  <ul>
    {% for category in category_list %}
    <li>
      <a href="#">{{ category.name }} <span class="post-count">(13)</span></a>
    </li>
    {% empty %}
    暂无分类！
    {% endfor %}
  </ul>
</div>
```

# 10 分类与归档

### 10.1分类


```python
blog/views.py

import markdown

from django.shortcuts import render, get_object_or_404

# 引入 Category 类
from .models import Post, Category

def category(request, pk):
    # 记得在开始部分导入 Category 类
    cate = get_object_or_404(Category, pk=pk)
    post_list = Post.objects.filter(category=cate).order_by('-created_time')
    return render(request, 'blog/index.html', context={'post_list': post_list})
```


```python
blog/urls.py

from django.conf.urls import url

from . import views

app_name = 'blog'
urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^post/(?P<pk>[0-9]+)/$', views.detail, name='detail'),
    url(r'^archives/(?P<year>[0-9]{4})/(?P<month>[0-9]{1,2})/$', views.archives, name='archives'),
    + url(r'^category/(?P<pk>[0-9]+)/$', views.category, name='category'),
]
```


```python
templates/base.html

{% for category in category_list %}
<li>
  <a href="{% url 'blog:category' category.pk %}">{{ category.name }}</a>
</li>
{% endfor %}
```

### 归档


```python
blog/views.py

def index(request):
    post_list = Post.objects.all().order_by('-created_time')
    return render(request, 'blog/index.html', context={'post_list': post_list})
```


```python
blog/views.py

def archives(request, year, month):
    post_list = Post.objects.filter(created_time__year=year,
                                    created_time__month=month
                                    ).order_by('-created_time')
    return render(request, 'blog/index.html', context={'post_list': post_list})
```


```python
blog/urls.py

from django.conf.urls import url

from . import views

app_name = 'blog'
urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^post/(?P<pk>[0-9]+)/$', views.detail, name='detail'),
    + url(r'^archives/(?P<year>[0-9]{4})/(?P<month>[0-9]{1,2})/$', views.archives, name='archives'),
]
```
