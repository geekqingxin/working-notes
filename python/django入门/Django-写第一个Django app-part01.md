[Writing your first Django app, part 1](https://docs.djangoproject.com/en/1.11/intro/tutorial01/)

# 1. 安装Django

## 1.1 安装Python 3(Ubuntu)

    sudo apt-get install python3
    
    # 验证是否安装成功
    python3

## 1.2 安装pip

    curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py

    sudo python3 get-pip.py

    # 验证是否安装成功
    pip list

    # 如在Ubuntu上同时安装了python2和python3，默认安装的pip是安装到python2中的

    sudo apt install python3-pip

    python3 -m pip install Django

    pip list
    

## 1.3 安装Django

    sudo pip3 install Django

    # 验证是否安装成功
    python3 -m django --version


# 2. 创建工程

    mkdir -P workspaces/python3_workspaces

    cd workspaces/python3_workspaces

    django-admin startproject mysite

> Note
> 
> python代码不应该放在服务器的/var/www上当中，而应该放在documentRoot外边，如/home/mycode



    sudo apt install tree

    # 查看代码结构

    liuyf@liuyf-VirtualBox:~/workspaces/python3_workspaces$ tree mysite/

    mysite/
    ├── manage.py
    └── mysite
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py
    
    1 directory, 5 files


文件说明：

* mysite项目名，Django不关心这个文件夹的名字
* manage.py
    
    一个实现了与Django工程交互的命令行工具
* 内部的mysite目录是项目中存储Python包的位置，他的名字是Python包名，在导入时需要(如, mysite.urls)
* mysite/__init__.py：是一个空文件，告诉Python这个目录是一个Python包
    
    [more about packages](https://docs.python.org/3/tutorial/modules.html#tut-packages)
* mysite/settings.py：当前Django工程的配置文件。
    
    [Django settings](https://docs.djangoproject.com/en/1.11/topics/settings/)
* mysite/urls.py：当前Django工程中声明的URL。

    [URL dispatcher](https://docs.djangoproject.com/en/1.11/topics/http/urls/)
* mysite/wsgi.py：支持WSGI兼容的服务器，这是入口地址。

    [How to deploy with WSGI](https://docs.djangoproject.com/en/1.11/howto/deployment/wsgi/)

# 3. 启动开发服务器

    python mysite/manage.py runserver

    # 日志输出
    Performing system checks...

    System check identified no issues (0 silenced).
    
    You have 13 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
    Run 'python manage.py migrate' to apply them.
    
    August 21, 2017 - 15:11:24
    Django version 1.11.4, using settings 'mysite.settings'
    Starting development server at http://127.0.0.1:8000/
    Quit the server with CONTROL-C.


> Note:
> 
> * 指定端口
> 
>     python3 mysite/manage.py runserver 8080
> 
> * 绑定全部端口
> 
>     python3 mysite/manage.py runserver 0:8080
> 
>     0表示：0.0.0.0
> 
> * 开发服务器将自动加载Python代码
> 
>     改代码不需要重启服务器，但如果添加文件，需要重启服务器

# 4. 创建Poll应用

> Note:
> 
> Project vs App
> 
> * app：是一个Web应用，如Weblog系统或简单投票系统
> * project：是一系列的配置加上特定网站的App
> * 一个工程可以包含多个App
> * 一个App也可以在多个工程中


创建应用：

    cd mysite
    python3 manage.py startapp polls

查看目录结构的变化：

    liuyf@liuyf-VirtualBox:~/workspaces/python3_workspaces/mysite$ tree
    .
    ├── manage.py
    ├── mysite
    │   ├── __init__.py
    │   ├── __pycache__
    │   │   ├── __init__.cpython-35.pyc
    │   │   └── settings.cpython-35.pyc
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    └── polls
        ├── admin.py
        ├── apps.py
        ├── __init__.py
        ├── migrations
        │   └── __init__.py
        ├── models.py
        ├── tests.py
        └── views.py

# 5. 创建第一个view

> Note:
> 
> 重新安装vi
> 
> sudo apt-get remove vim-common
> 
> sudo apt-get install vim


    vi polls/views.py

* 添加以下代码到文件中：

        from django.http import HttpResponse

        def index(request):
            return HttpResponse("Hello, world. You're at the polls index.")

* 映射URL：

        vi urls.py

* 添加以下代码到文件中：

        from django.conf.urls import url

        from . import views

        urlpatterns = [
            url(r'^$', views.index, name='index'),
        ]

* 修改urls.py如下

        from django.conf.urls import include, url
        from django.contrib import admin

        urlpatterns = [

            url(r'^polls/', include('polls.urls')),
            url(r'^admin/', amdin.site.urls),
        ]

* 启动开发服务器

        python manage.py runserver


# X. 问题

## X.1 No command 'django-amdin' found

错误提示：

liuyf@liuyf-VirtualBox:~/workspaces$ django-amdin startproject mysite
No command 'django-amdin' found, did you mean:
 Command 'django-admin' from package 'python-django-common' (main)
django-amdin: command not found

答案：

    sudo apt install python3-pip

    sudo pip3 install Django
