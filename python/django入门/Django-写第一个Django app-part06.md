[Writing your first Django app, part 6](https://docs.djangoproject.com/en/1.11/intro/tutorial06/)

在Tutorial 5中，我们增加了一些测试代码，现在我们加入一些样式和图片。

除了服务器产生的HTML，Web应用通常需要提供一些附加的资源，如图片，脚本或CSS - 有这些内容才能呈现一个完整的web页面。在Django，我们把这些文件称为静态文件。

对于小的应用，这可能不会有大量工作，因为你可以把这些静态文件保存到你服务器可以找到的地方。无论如何，在一个大的工程中 - 尤其是这个工程包含有多个应用 - 处理这些由不现的应用提供的静态文件集合会有些麻烦。

这正是django.contrib.staticfiles所要提供的：它收集每个应用(或你指定的其它地方)的静态文件，并把这些文件放在一个统一的地方，使它可以很容易的生产环境中使用。

# 1. 定制你的App外观(look and feel)

首先在polls目录下创建一个static目录。Django从这儿读取静态文件，这与Django从polls/templates/中查找模板是一样的。

Django的STATICFILES\_FINDERS设置包含了一个查找器(finder)列表，它知道怎么在不同的数据源中发现静态文件。AppDirectoriesFinder是这些默认中的一个，他查找每个INSTALLED\_APPS中的static子目录，如我们刚在polls目录下创建的static。admin站点对于静态文件，使用相同的目录结构。

在你刚刚创建的static目录中，创建polls目录，并在polls目录中，创建一个style.css文件。

> Note:
> 
> 静态文件命名空间
> 
> 像模板一样，我可以放我们静态文件到polls/static，但是这不是一个好主意。Django在查找文件时，将会匹配第一个与他名字相同的文件。如果你不同应用间有相同的静态文件，Django将没有能力鉴别它们。我们需要告诉Django那一个是正确，最简单的方式就是使用"命名空间"。也就是，把这些静态文件放在一个命名为应用名称的目录中。

放以下代码到stylesheet中：

    # polls/static/polls/style.css

    li a {
        color: green;
    }

接下来，在polls/templates/polls/index.html顶部添加以下内容：

    # polls/templates/polls/index.html
    
    {% load static %}
    
    <link rel="stylesheet" type="text/css" href="{% static 'polls/style.css' %}" />

启动服务器：

    python3 manage.py runserver

{% static %}模板标签将生成静态文件的绝对路径。

那是所有你需要做的开发工作。刷新http://localhost:8000/polls/，你应该看到问题链接变成了绿色，那也意味着你的样式表正确的被加载。


# 2. 增加background-image

接下来，我们将创建一个子目录用来存储图片。在polls/static/polls目录下创建一个images子目录。在这个目录中，放一个background.gif图片。

然后添加以下内容到样式表(polls/static/polls/style.css)：

    # polls/static/polls/style.css
    
    body {
        background: white url("images/background.gif") no-repeat right bottom;
    }

刷新http://localhost:8000/polls/，你应该在屏幕的右下角，能看到背景已经被加载。


> Note:
> 
> 警告
> 
> 当然{% static %}模板标签在不是由Django创建的文件中是不可用的。如你的样式表文件。你应该总是使用相对路径在静态文件间进行引用，因为你可能改变STATIC_URL(使用static模板标签产生的URL)，而不需要修改静态文件中的路径。

这是一些基础。更多关于设置和框架中包含的其它部分，可以参见[the static files how to](https://docs.djangoproject.com/en/1.11/howto/static-files/)和[the staticfiles reference](https://docs.djangoproject.com/en/1.11/ref/contrib/staticfiles/)。[Deploying static files](https://docs.djangoproject.com/en/1.11/howto/static-files/deployment/)讨论了在真实的服务器环境中如何使用静态文件。
