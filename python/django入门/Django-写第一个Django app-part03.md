[Writing your first Django app, part 3](https://docs.djangoproject.com/en/1.11/intro/tutorial03/)

# 1. 概述

View在Django应用中服务于某个特定功能，有一个特定模板。例如，在一个blog系统中，可能会有以下view：

* 博客主页(Blog homepage)
* 详情页面(Entry "detail" page)
* 年归档页面(Year-based archive page)
* 月归档页面(Month-based archive page)
* 日归档页面(Day-based archive page)
* 评论功能(Comment action)

在我们的Poll应用中，我们有四个view：

* Question "index" page - 显示最新的几条问题。
* Question "detail" page - 显示问题内容，不显示问题答案，但包含一个投票表单。
* Queation "results" page - 显示特定问题的结果。
* Vote action - 处理投票。

在Django中，Web页面和别的内容通过view来实现。每个view通过简单的Python函数或在基于类的view中的方法来实现。Django通过请求的url来选择对应的view。

URL pattern是一个对普通URL的简化。如，/newarchive/<year>/<month>/。

为了实现URL和view的映射，Django使用了"URLconfs"。URLconfs映射了URL patterns和view的关系。

[django.urls](https://docs.djangoproject.com/en/1.11/ref/urlresolvers/#module-django.urls)

# 2. 添加更多的view

打开polls/views.py，添加更多的view，这些view与之前的有点不同，因为他们需要传入一个参数：

    def detail(request, question_id):
        return HttpResponse("You're looking at question %s." % question_id)

    def results(request, question_id):
        response = "You're looking at the results of question %s."
        return HttpResponse(response % question_id)

    def vote(request, question_id):
        return HttpResponse("You're voting on question %s." % question_id)

添加新的view页面到polls.urls，vi polls/urls.py：

    from django.conf.urls import url

    from . import views

    urlpatterns = [
        # ex: /polls/
        url(r'^$', views.index, name='index'),
        # ex: /polls/5/
        url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
        # ex: /polls/5/results/
        url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
        #ex: /polls/5/vote/
        url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
    ]

访问以下链接：

[http://localhost:8000/polls/34/](http://localhost:8000/polls/34/)

[http://localhost:8000/polls/34/results/](http://localhost:8000/polls/34/results/)

[http://localhost:8000/polls/34/vote/](http://localhost:8000/polls/34/vote/)

当某人请求一个页面，如/polls/34/，Django将加载mysite.urls模块，这个可以通过ROOT_URLCONF来配置。他查找在urlpatterns中定义的变量，依次遍历正则表达式。在查找到匹配的项，如'^polls/'；剥离匹配的文本，如"polls/"，发送剩余的文本(如，34/)到polls.urls进行下一步处理。在这个地方，他匹配r'^(?P<question_id>[0-9]+)/$'，然后调用detail() view：

    detail(request=<HttpRequest object>, question_id='34')

'question_id=34'部分来自于(?P<question_id>[0-9]+)。通过圆括号括起来的模式，捕获模式匹配的文本，然后把他作为一个参数发送给view函数；?P<question_id>定义了将会被用来标识匹配模式的名字；[0-9]+是一个正则表达式，匹配一个数字序列。

由于URL模式是一个正则表达式，对于你想要使用他来做什么没有限制。不需要增加URL后缀，如.html；如果你想要达到这样的效果的话，通过以下方式可以实现：

    url(r'^polls/latest\.html$', views.index),

但是，不要这样做。这样做有点不明事理。

# 3. 写一些真实的view

每个view做以下两件事中的一件事：返回一个HttpResponse，包含请求页面的内容；或抛出一个Http 404。其它事情则取决于业务需求。

view可以来源于数据库或其它地方。可以使用Django的模板系统，或第三方的Python模板系统，也可以不使用模板系统。可以产生一个PDF文件，输出XML内容，在线生成一个ZIP文件，也可以任何你想要的内容形式，或通过Python库生成你想要的内容形式。

所有Django想要的是一个HttpResponse，或一个异常。

接下来我们就选择采用part2部门已经介绍过的Django内置的数据库API。接下来，我们在index() view引入这些API。在系统中根据publication date显示最新的5条poll问题，由逗号分隔。

    vi polls/views.py


    from django.http import HttpResponse

    from .models import Question

    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        output = ', '.join([q.question_text for q in latest_question_list])
        return HttpResponse(output)


> Note:
> 
> 使用sqlite3打开数据库
> 
>     sqlite3 db.sqlite3
>     
>     # 查看帮助
>     .help
>     
>     # 显示当前数据库名及挂载的数据库文件
>     .databases
>     
>     # 列出当前数据库中的表
>     .tables
>     
>     # 查询表数据
>     select * from polls_question;

进入admin管理后台，localhost:8000/admin/，添加问题数据。然后重新刷新polls页面。

使用Django模板系统，把Python代码与设计分离。

* 在polls下创建模板目录，templates。Django将会在这个目录中查找模板。
    
    工程的[TEMPLATES](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-TEMPLATES)中描述了Django如何加载和渲染模板。默认配置了DjangoTemplates，它的APP_DIRS设置为True。根据约定，DjangoTemplates查看所有INSTALLED_APPS下的templates子目录。
        
    在templates目录下再创建一个polls，在polls目录中创建index.html文件。最终的模板路径应该是polls/templates/polls/index.html。由于上面提到的app_directories工作机制，在Django中引用模板文件，可以简写成polls/index.html。

    > Note: Template namespacing
    > 
    > 现在我们把模板放在polls/templates(而不是polls/templates/polls)下可能将工作，但它不是一个好的解决办法。Django将通过名字选择第一个匹配的模板，如果你在其它应用中有一个同样名字的模板，Django将不能区分它们。我们需要告诉Django那个才是正确的选择，最容易的方式是通过namespacing它们。也就是放这些模板到应用它自己的模板目录下，如：polls/templates/polls。

* 创建模板，放以下内容到模板中：

        {% if latest_question_list %}
            <ul>
                {% for question in latest_question_list %}
                    <li>
                        <a href="/polls/{{ question.id }}/">
                        {{ question.question_text }}
                        </a>
                    </li>
                {% endfor %}
            </ul>
        {% else %}
            <p>No polls are available.</p>
        {% endif %}

* 更新index view，vi poll/views.py

        from django.http import HttpResponse
        
        from django.template import loader

        from .models import Question

        def index(request):
            latest_question_list = Qustion.objects.order_by('-pub_date')[:5]
            template = loader.get_template('polls/index.html')
            context = { 'latest_question_list': latest_question_list, }
            return HttpResponse(template.render(context, request))

以上代码加载了polls/index.html模板，同时传给模板一个context对象。context是一个映射了模板变量名和Python对象的字典。

## 3.1 简洁方式: render()

通常情况下，加载一个模板，填充context，返回一个HttpResponse对象，这个对象包含已渲染好的结果页面。Django提供了一种简洁方式，以下是重写后的index() view：

    vi polls/views.py


    from django.shortcuts import render
    from .models import Question

    def index(request):
        latest_question_list = Question.objects.order_by('-pub_date')[:5]
        context = {'latest_question_list': latest_question_list}
        return render(request, 'polls/index.html', context)

如果像上面这样修改所有view后，我们不需要再导入loader和HttpResponse。

render()函数第一个参数为request对象，第二个是模板名，可选的第三个参数。它返回一个HttpResponse对象。

# 4. 抛出404错误

现在让我们处理问题详情页面 - 页面显示某个poll的问题内容。

    vi polls/views.py

    from django.http import Http404
    from django.shortcuts import render

    from .models import Question

    # ...

    def detail(request, question_id):
        try:
            question = Question.objects.get(pk=question_id)
        except Question.DoesNotExist:
            raise Http404("Question does not exist")
        return render(request, 'polls/detail.html', {'question': question})

如果与requested ID相关联的问题不存在，view将抛出Http404。

接下来处理polls/detail.html模板，如果你想快速让上面的例子工作，可以只放以下内容到文件：

    {{ question }}

## 4.1 简洁方式：get_object_or_404()

如果对象不存在，通常的情况下是使用get()并且抛出Http404。Django提供了一种简洁方式，重写detail() view：

    vi polls/view.py

    from django.shortcuts import get_object_or_404, render

    from .models import Question

    def detail(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/detail.html', {'question': question})

get_object_or_404函数第一个参数是Django model，后面可以有多个条件参数，这些参数将被传递到model的get()函数中。如果对象不存在，他将抛出Http404。

> Note:
> 
> 为什么我们要使用get_object_or_404()替代在上层自动捕获ObjectDoesNotExist异常，或使用Http404来替代ObjectDoesNotExist？
> 
> 因为那样会耦合model层和view层。Django最重要的设计目标是维护松耦合。一些控制耦合的方法由[django.shortcuts](https://docs.djangoproject.com/en/1.11/topics/http/shortcuts/#module-django.shortcuts)模块提供。

# 5. 使用模板系统

回到我们poll应用中的detail() view。给定的context变量question，polls/detail.html可以实现如下：

    <h1>{{ question.question_text }}</h1>
    <ul>
        {% for choice in question.choice_set.all %}
            <li>{{ choice.choice_text }}</li>
        {% endfor %}
    </ul>

[更多信息，参见template guide](https://docs.djangoproject.com/en/1.11/topics/templates/)

# 6. 移出模板中的硬编码URL

在polls/index.html模板中，链接像这样:

    <li><a href="/polls/{{ question_id }}">{{ question.question_text }}</a></li>

硬编码的问题，在有大量模板的工程中去修改URL会变得很困难。因此，由于在polls.urls模块中通过url()函数定义了name参数，你可以从特定的URL中移出硬编码，而使用{% url %}模板tag来替换：

    <li><a href="{% url 'detail' question_id %}">{{ question.question_text }}</a></li>

这种方式是通过在polls.urls模块中查找URL定义。你可以看到detail的URL名定义如下：

    url(r^(?P<question_id>[0-9]+/$', views.detail, name='detail'),

如果你想修改polls detail view的URL，假设改为polls/specifics/12/，你不需要修改模板，只需要在polls/url.py中修改即可：

    url(r'^specifics/(?P<question_id>[0-9]+)/$', views.detail, name='detail'),

# 7. Namespacing URL names

tutorial工程只有一个app，polls。在真实的Django工程中，这儿可能有5, 10, 20 app，甚至更多。Django是如何区分他们的URL名呢？举例，polls app有一个detail view，在同一工程中另一个同名的view代表的是一个blog。在使用{% url %}时，如何让Django知道为那个app的view创建url呢？

答案是为你的URLconf添加命名空间。在polls/urls.py文件中，在头部添加一个app，设置应用的命名空间:

    from django.conf.urls import url

    from . import views

    app_name='polls'

    urlpatterns = [
        url(r'^$', views.index, name='index'),
        url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
        url(r'^(?P<question_id>[0-9]+)/result/$', views.results, name='results'),
        url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
    ]

查看模板polls/index.html：

    <li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</li>

将以上模板修改为：

    <li><a href="{% url 'polls.detail' question.id %}">{{ question.question_text }}</a></li>