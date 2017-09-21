[Writing your first Django app, part 4](https://docs.djangoproject.com/en/1.11/intro/tutorial04/)

# 1. 在detail.html中增加form表单

    # poll/templates/polls/detail.html

    {% if error_message %}
        <p>
            <strong>
                {{ error_message }}
            </strong>
        </p>
    {% endif %}

    <form action="{% url 'polls:vote' question.id %}" method="post">
        {% csrf_token %}
        {% for choice in question.choice_set.all %}
            <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
            <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
        {% endfor %}
        <input type="submit" value="Vote" />
    </form>

创建一个投票表单，每个radio代表一种可选答案。

创建一个view来处理提交的数据。还记得在Tutorial 3中，我们创建了一个URLconf，如下行：

    # polls/urls.py

    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),

我们也创建了一个vote()函数的空实现。现在让我们创建一个真正的vote()函数。加以下内容到polls/views.py：

    # polls/views.py

    from django.shortcuts import get_object_or_404, render
    from django.http import HttpResponseRedirect, HttpResponse
    from django.urls import reverse
    from .models import Choice, Question

    def vote(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        try:
            selected_choice = question.choice_set.get(pk=request.POST['choice'])
        except (KeyError, Choice.DoesNotExist):
            return render(request, 'polls/detail.html', {
                'question': question,
                'error_message': "You didn't select a choice."
            })
        else:
            selected_choice.votes += 1
            selected_choice.save()
            return HttpResponseRedirect(reverse('polls:results', args=(question_id,)))

> Note:
> 
> 在成功处理后，总是返回一个HttpResponseRedirect。这可以阻止用户按返回键时，数据再次被提交。
> 
> 在HttpResponseRedirect的构造函数中我们传入了一个reverse函数。这个函数帮助避免在view中硬编码一个URL。在这儿，给出了对应的view名称，URL模式中的变量部分，指向了这个view。这样的话，在Tutorial 3中我们使用URLconf，这个reverse()调用将返回如下的字符串：
> 
>    '/polls/3/results/'
>
>这儿的3是一个question.id。这个Redirected URL然后调用results views显示完成最后的展现。

就像在Tutorial 3中提到的一样，request是一个HttpResponse对象。更多关于HttpRequest对象的内容，参见：[request and response documentation](https://docs.djangoproject.com/en/1.11/ref/request-response/)

在某人对问题进行投票后，vote() view将redirect到问题结果页。这个view代码如下：

    # polls/views.py
    
    from django.shortcuts import get_object_or_404, render
    
    def results(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        return render(request, 'polls/results.html', {'questio': question})

这与detail() view除了模板名不一样，其它几乎一样。稍候我们将剔除这些冗余代码。

现在，让我们创建一个polls/results.htm模板：

    # polls/templates/polls/results.html
    
    <h1>{{ question.question_text }}</h1>

    <ul>
        {% for choice in question.choice_set.all %}
            <li>{{ choice.choice_text }} -- {{ choice.votes }} vote {{ choice.votes | pluralize }}</li>
        {% endfor %}
    </ul>

    <a href="{% url 'polls:detail' question.id %}">Vote again?</a>

现在在浏览器中打开polls/1，对问题进行投票。你应该看到一个结果页，这个结果页在你每次投票后，都会更新。如果你没有选择选项，直接提交表单，你将会看到一个错误信息。

> Note:
> 
> vote() view中的代码，在多人同时提交时，会有并发问题。如现在投票数是42,两个同时投票，投票结果将显示43。这被称为临界条件(race condition)。如果你感兴趣，你可以读[Avoiding race conditions using F()](https://docs.djangoproject.com/en/1.11/ref/models/expressions/#avoiding-race-conditions-using-f)来解决这个问题。

# 2. 使用通用view(generic views)：Less code is better

detail()和results() view是非常简单的，就像上面说的一样，他们之间有一些冗余代码。index() view，只显示了一个polls列表，也与他们类似。

这些view代表了基本WEB开发的公共情况：根据URL中的参数，从数据库中提取数据，加载模板，返回填充后的模板页面。由于这是通用的，Django提供了一种快捷方式，被称为"generic views"系统。

generic view抽象了公共模式，目标是你甚至不需要写Python代码就可以写一个app。

接下来，我们将使用generic view system来改造poll app，因此会删除一些代码。我们要花几步来完成这个转换。

TODO：

* 转换到URLconf
* 删除老的无用的views。
* 基于Django's generic views引入新的view

读更多的细节。

> Note:
> 
> Why the code-shuffle?
> 
> 通常情况下，当你写一个Django app，你需要评估一下是否generic view对于解决你的问题是合适的；你从头开始使用它，胜过中途去重构你的代码。但是这个tutorial从现在开始，有意通过困难的方式来写view，并关注在核心概念。
> 
> 在使用计算器前，你应该首先了解一些基础的数学。(作者做了一个类比)

## 2.1 Amend URLconf

修改polls/urls.py URLconf，代码如下：

    from django.conf.urls import url

    from . import views
    
    app_name = 'polls'
    urlpatterns = [
        url(r'^$', views.IndexView.as_view(), name='index'),
        url(r'^(?P<pk>[0-9]+)/$', views.DetailView.as_view(), name='detail'),
        url(r'^(?P<pk>[0-9]+)/results/$', views.ResultsView.as_view(), name='results'),
        url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
    ]

注意：在第二个和第三个正则表达式的匹配模式中，<question_id>被改成了<pk>。

## 2.2 Amend views

接下来，我们打算移出老的index, detail和results view，然后使用Django's generic views替代。为了实现这个，我们打开polls/views.py，修改代码如下：

    from django.shortcuts import get_object_or_404, render
    from django.http import HttpResponseRedirect
    from django.urls import reverse
    from django.views import generic

    from .models import Choice, Question

    class IndexView(generic.ListView):
        template_name = 'polls/index.html'
        context_object_name = 'latest_question_list'

        def get_queryset(self):
            """Return the last five published question."""
            return Question.objects.order_by('-pub_date')[:5]

    class DetailView(generic.DetailView):
        model = Question
        template_name = 'polls/detail.html'

    class ResultsView(generic.DetailView):
        model = Question
        template_name = 'polls/results.html'

    def vote(request, question_id):
        question = get_object_or_404(Question, pk=question_id)
        try:
            selected_choice = question.choice_set.get(pk=request.POST['choice'])
        except (KeyError, Choice.DoesNotExist):
            return render(request, 'polls/detail.html', {
                'question': question,
                'error_message': "You didn't select a choice."
            })
        else:
            selected_choice.votes += 1
            selected_choice.save()
            return HttpResponseRedirect(reverse('polls:results', args=(question_id,)))

现在我们使用了两个generic view：ListView和DetailView。这两个view分别抽象了，显示"一个对象列表"和"特定对象的细节页面"。

* 每个generic view需要知道它要作用于那个model，这通过model属性来设置。
* DetailView generic view通过URL中的pk来获取主键值，因此我们在generic view中把question_id改成了pk。

默认情况下，DetailView generic view使用<app_name>/<model name>_detail.html的模板。在我们的示例中，它将使用"polls/question_detail.html"作为模板名。template_name是用来告诉Django使用一个特定的模板名来替代自动生成的默认模板名。我们也为results list view指定了template_name - 这样可以确保results view和detail view在展示时使用不同的外观，尽管他们后面都是使用的DetailView。

类似的，ListView generic view也会使用一个默认的<app_name>/<model name>_list.html模板名; 我们使用template_name告诉ListView使用我们存在的"polls/index.html"模板。

在tutorial中的前面的部分，提供给模板一个context包含两个question和latest\_question\_list context变量。对于DetailView中的question变量是自动被提供的 - 因为我们正在用Django model(Question)，Django可以为context变量选择一个恰当的名字。无论如何，对于ListView，自动产生的变量是question_list。如果要重载这个，我们可以提供一个context_object_name属性，指定我们想要使用的latest_question_list变量。做为一个替代方案，你可以修改你的模板，来匹配新的默认context变量，但是告诉Django使用那个变量，是更容易实现的方式。

[更多关于generic view的细节，请参考](https://docs.djangoproject.com/en/1.11/topics/class-based-views/)