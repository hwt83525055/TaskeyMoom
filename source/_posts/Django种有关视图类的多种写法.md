---
title: Django中有关视图类的多种写法
date: 2017-6-28 23:35:51
tags: [Django, Python]
---
在后台开发的过程中发现有关视图其实是有多种实现方法的，那么今天就来统计一下到底有哪些不同的写法，逐步更新。

## 第一种 以方法为基础实现的视图
举个例子
```python
url(r'^welcome/$', views.welcome, name="welcome")
```
这是一行url视图函数，那么在views找一下这个welcome的方法
```python
def welcome(request):
    return render(request, 'welcome.html')
```
这里返回了一个render，启动服务之后可以看见返回的是welcome.html的页面
复杂一点，如何去实现某些功能或是请求呢？例如实现一个接口函数。看下面的代码

```python
url(r'^form/$', views.formmanage, name="formmanage"),
```
同样寻找一下他的views函数
```python
    activity = Activity_act()
    uid = request.session['login']
    form = activity.allActivity()
    return render(request, 'adminapp/biaodan-list.html',
                  {"activity": form,})
```
上图中的Activity_act是一个方法包中的某个方法，从返回值可以看出返回了form并将form传递在了render中，那么我们在调用网页的时候就可以通过template语句去调用form，具体的template的写法可以见其他文章，这就是一个以方法为基础实现的网页架构

下面我们再看看第二种写法
## 第二种 基于视图View本身的写法
[Django官方文档](https://docs.djangoproject.com/en/dev/ref/class-based-views/base/)

在文档中介绍了以下几种View

View

>View是一种基础视图，常见写法为
```python
class MyView(View):

    def get(self, request, *args, **kwargs):
        return HttpResponse('Hello, World!')
```
在url中的话这种实现通过View的as_view()函数
```python
url('^myview', Myview.as_view(), name='我的视图')
```

另外在View中提供了几个配置用的函数，在连接中有详解，这里就不细说了
当对myview进行get请求时就会返回Hello World的http形式的response
那么在django中他已经为我们封装好了其他许多好用的视图

Template_View

>Template_View顾名思义是一种模板类的视图，常见写法为
```python
from django.views.generic.base import TemplateView

from articles.models import Article

class HomePageView(TemplateView):

    template_name = "home.html"

    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['latest_articles'] = Article.objects.all()[:5]
        return context
```
依次看一下代码，首先template_name就是模板的名字，也就会在当前项目的statics下找到home.html进行直接渲染，不需要我们去返回某个render对网页进行渲染，接下来的get_context_data则是一个获取属性值的方法，同样只需要写个方法就可以在前端页面中自动获取到context这个属性值，是不是特别的方便呢？当然了，Template是继承于View的方法，所以所有View的配置他也可以使用

RedirectView

>RedirectView是一种重定向用的视图，暂时还没有用到，所以先不提，之后补充一下有关View的几种配置方法与特殊的写法


<font size=15>Thanks