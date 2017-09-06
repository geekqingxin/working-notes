[Writing your first Django app, part 5](https://docs.djangoproject.com/en/1.11/intro/tutorial05/)

# 1. 引入自动测试

## 1.1 什么是自动化测试？

自动化测试的不同是测试工作由系统代替你完成。一旦你创建了测试集，然后当你修改你的app时，你可以检测你的代码仍然像最初你的设计目的一样在工作，并且不需要执行耗时的手动测试。

## 1.2 为什么你需要创建测试？

为什么创建测试，为什么是现在呢？

### 1.2.1 测试将会节省你的时间

因为自动化测试可能在几秒钟内完成测试工作。如果某个地方出错了，测试也将辅助识别代码中引起异常行为的代码。

### 1.2.2 测试不仅会识别问题，也预防问题

只把测试认为是开发的对立面(negative aspect)是对测试的一种误解。

如果没有测试，应用程序的期望行为可能不透明的。甚至当他是你自己的代码，你有时也会浏览它，尝试找出什么是他明确要做的事情。

测试将会改变这样的认识，他们会从内部观察代码，当某处有问题时，他们将关注在有问题的部分 - 尽管你可能没有意识到它是有问题的。

### 1.2.3 测试使你的代码更有吸引力

Code without tests is broken by design.

### 1.2.4 测试将帮助团队一块工作

测试会保证同事不会故意破坏你的代码(在不知情的情况下，破坏代码)。如果你想要作为一个Django程序员，你必须擅长写测试。

# 2. 基本的测试策略

[test-driven development](https://en.wikipedia.org/wiki/Test-driven_development)，这种方法是在写应用代码前先写测试代码。这可能是有点反常规，但事实上他与大多数人常常做其它事是相类似的：他们描述一个问题，然后创建一些代码来实现它。测试驱动开发大大简化Python中测试用例的问题。

更常见的是，对于测试比较陌生的人，它将创建一些代码，之后再决定他应该有那些测试代码。也许先写测试是更好的，但不管如何，开始写测试永远都不晚。

有时它是非常困难的，从什么地方开始写测试。如何你写了数千行Python代码，选择某个地方开始测试可能不太容易。在这种情况，在你写新的功能前，开始写新的测试，或者当你开始feature或修改bug。

因此，让我们开始吧。

# 3. 开始写测试

## 3.1 找到一个BUG

幸运的是，在polls应用中，这儿有一个小的BUG，现在我们修复他：如果问题是被昨天(last day)发布，Question.was\_published\_recently()方法返回Ture(这是正确的)，但是如果Question's pub_date是在未来(它将不正确)。

为了检查是否BUG真的存在，使用Admin创建一个问题，发布日期选择未来某个时间，使用shell来验证方法。

[Django's admin shell](https://docs.djangoproject.com/en/1.11/ref/django-admin/#django-admin-shell)


    python manage.py shell


    # commands
    import datetime
    from django.utils import timezone
    from polls.models import Question

    future_question = Question(pub_date=timezone.now() + datetime.timedelta(days=30))

    future_question.was_published_recently()

由于在未来的内容不是最近，很显然这是错的。

## 3.2 创建测试，揭示(expose)BUG

刚刚在shell中完成的测试，这完全可以使用自动化测试完成，因此让我们转入自动化测试。

应用测试代码通常是在应用的tests.py文件中；测试系统将自动在所有以test开始的文件中查找测试代码。

将以下代码写入polls应用的tests.py文件中：

    import datetime

    from django.utils import timezone
    from django.test import TestCase
    from .models import Question

    class QuestionModelTests(TestCase):

        def test_was_published_recently_with_future_question(self):
            """
            was_published_recently() returns False for questions whose pub_date
            is in the future.
            """
            time = timezone.now() + datetime.timedelta(days=30)
            future_question = Question(pub_date=time)

            self.assertIs(future_question.was_published_recently(), False)


我们创建了django.test.TestCase的子类，它包含一个方法，这个方法使用pub\_date(未来某个时间)创建了一个Question实例。然后我们验证was\_published\_recently() - 是否返回False。


## 3.3 运行测试代码

    python manage.py test polls

输出结果：

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    F
    ======================================================================
    FAIL: test_was_published_recently_with_future_question (polls.tests.QuestionModelTests)
    ----------------------------------------------------------------------
    Traceback (most recent call last):
      File "/home/liuyf/workspaces/python3_workspaces/mysite/polls/tests.py", line 21, in test_was_published_recently_with_future_question
        self.assertIs(future_question.was_published_recently(), False)
    AssertionError: True is not False
    
    ----------------------------------------------------------------------
    Ran 1 test in 0.001s
    
    FAILED (failures=1)
    Destroying test database for alias 'default'...


这个测试做了哪些工作：

* python manage.py test polls查找polls应用中的测试用例
* 它查找到一个django.test.TestCase的子类
* 为了测试，专门创建了一个特殊的数据库
* 开始查找测试方法，所以以test开头的方法
* 在test\_was\_published\_recently\_with\_future\_question中，它创建了一个Question实例，它的pub_date是当天向后30天。
* 然后使用assertIs()方法，它发现它的was\_published\_recently()返回True，但是我们期望它返回False。

最后，测试通知我们测试失败，以及错误发生在那一行。

## 3.4 修复BUG

我们已经知道了问题是什么：如果pub\_date是在未来某个时间，Question.was\_published\_recently()应该返回False。修改model.py中的方法只是日期是在过去，它才返回True。

    # polls/models.py

    def was_published_recently(self):
        now = timezone.now();
        return now - datetime.timedelta(days=1) <= self.pub_date <= now

再次运行测试：

    python manage.py test polls

执行结果：

    Creating test database for alias 'default'...
    System check identified no issues (0 silenced).
    .
    ----------------------------------------------------------------------
    Ran 1 test in 0.035s

    OK
    Destroying test database for alias 'default'...

在找到一个BUG后，我们写了一个测试来揭示它并修改了代码中的BUG，最后我们测试代码正确的通过了测试。

在未来，我们应用中别的内容可能变错，但是我们确信我们不会意外再次引入这个BUG，因为只要运行测试，就会立刻警告我们。我们可以考虑应用的这一部分，现在被固定下来了，永远安全了。

## 3.5 更全面的测试

到目前为止，我们可以进一步约束was\_published\_recently()方法；事实上，在修复BUG时引入新的BUG，将会十分尴尬。

在这个类上加入两个新的测试方法，更全面的测试方法的行为：

    # polls/tests.py

    def test_was_published_recently_with_old_question(self):
        """
        was_published_recently() returns False for questions whose pub_date
        is older than 1 day.
        """
        time = timezone.now() - datetime.timedelta(days=1, seconds=1)
        old_question = Question(pub_date=time)
        self.assertIs(old_question.was_published_recently(), False)

    def test_was_published_recently_with_recent_question(self):
        """
        was_published_recently() returns True for question whose pub_date
        is within the last day.
        """
        time = timezone.now() - datetime.timedelta(hours=23, minutes=59, seconds=59)
        recent_question = Question(pub_date=time)
        self.assertIs(recent_question.was_published_recently(), True)

现在有三个测试，证实Question.was\_published\_recently()对于过去，最近和未来的问题，可以返回一个正确的值。

再次强调，polls是一个简单的应用，不管未来他变得多么复杂或他与别的代码交互，我们现在可以保证我们写过测试的方法，他的行为总是我们期望的结果。

# 4. 测试view

polls应用没有鉴别能力：它将发布任何问题，包括pub\_date是在未来某个时间的问题。我们应该改善这个。设置一个未来的时间的pub\_date，意味着问题会被立刻发布，但是在到达未来的时间点前，是不可见的。

## 4.1 对于view的测试

当我们修复上面的BUG，我们首先写了测试，然后修复代码。事实上这就是一个测试驱动开发的简单例子，但是跟我们做任务的顺序没有太大关系。

在我们的第一个测试中，我们密切关注在代码的内部行为。对于这个测试，我们想要检测他的行为，这个行为是用户通过浏览器可以体验到的行为。

在开始修复任何问题前，让我们看一下我们要使用的工具。

## 4.2 The Django test client

Django提供了一个测试的Client来代码来模拟用户在view的交互。我们可以在tests.py或shell中使用它。

我们再次打开shell，我们需要在这儿做一些，没有必要在tests.py中做的工作。首先在shell中设置测试环境：
    
启动shell：
    
    python manage.py shell

执行以下命令：
    
    from django.test.utils import setup_test_environment
    setup_test_environment()

setup\_test\_environment()安装一个模板渲染程序(template renderer)，允许我们检查response中的一些附加属性，如response.context。注意：这个方法不会设置测试数据库，因此接下来将依据存在的数据库来运行程序，根据数据库中所创建的问题不同，显示结果会有一些差别。如果settings.py中的TIME_ZONE设置不正确的话，你可能会得到不期望的结果。如果你不记得之前的设置，在继续前请检查。

接下来，我们需求导入test client类(后面在tests.py，我们将使用django.test.TestCase类，自带client，因此这个不是必须的)：

    from django.test import Client
    client = Client()

接下，我们要求client为我们做一些工作：

    response = client.get('/')

    # 输出
    Not Found: /

    # 我们应该期望从那个地址，得到一个404；
    # 如果你看到一个"Invalid HTTP_HOST header"错误和一个400响应，
    # 你可能忽略了之前描述的，需要先调用setup_test_environment()

    response.status_code

    # 输出
    404

    # 另外，我应该期望从'/polls/'获取一些内容
    # 我们将使用reverse()，而不是使用硬编码的URL

    from django.urls import reverse
    response = client.get(reverse('polls:index'))
    response.status_code

    # 输出
    200
    
    response.content

    # 输出
    '\n    <ul>\n        \n            <li>\n                <a href="/polls/4/">\n                Why do we need the tests?\n                </a>\n            </li>\n        \n            <li>\n                <a href="/polls/3/">\n                What is IPO?\n                </a>\n            </li>\n        \n            <li>\n                <a href="/polls/2/">\n                What is ICO?\n                </a>\n            </li>\n        \n            <li>\n                <a href="/polls/1/">\n                What&#39;s up?\n                </a>\n            </li>\n        \n    </ul>\n\n'

    response.context['latest_question_list']

    # 输出，最新的问题列表

    <QuerySet [<Question: Why do we need the tests?>, <Question: What is IPO?>, <Question: What is ICO?>, <Question: What's up?>]>

## 4.3 改善我们的view

polls列表显示了仍然没有被发布的polls。(i.e. 例如有一个在未来某个时间点的pub_date)。让我们修复它：

在Tutorial 4中，我们引入了基于类的view，基于ListView：

    # polls/views.py

    class IndexView(generic.ListView):
        template_name = 'polls/index.html'
        context_object_name = 'latest_question_list'

        def get_queryset(self):
            """Return the last five pubished questions."""

            return Question.objects.order_by('-pub_date')[:5]

我们需要修改get_queryset()方法，修改它，以便它也可以跟timezone.now()方法进行比较。首先，我们需要导入:

    # polls/views.py

    from django.utils import timezone

然后，我们必须修改get_queryset方法：

    # polls/views.py

    def get_queryset(self):

        """
        Return the last five published question (not including those set to be
        published in the future).
        """
        return Question.objects.filter(
            pub_date__lte=timezone.now()
        ).order_by('-pub_date')[:5]

Question.objects.filter(pub\_date\_\_lte=timezone.now())返回一个查询结果集，只包括pub_date小于或等于当前时间的问题列表。

## 4.4 测试我们的新view

现在开启服务器，在浏览器中打开网站，创建一个过去和未来的Question，来验证view的行为是否如期望的一样，检验只有这样的问题会被列出来。如果不想要每次修改会影响到这部分，让我们来创建个测试，基于我们上面的shell session。

加以下内容到polls/tests.py：

    from django.urls import reverse

我们创建简化函数来创建问题和新的测试类：

    # polls/tests.py

    def create_question(question_text, days):

        """
        Create a question with the given 'question_text' and published the
        given number of 'days' offset to now (negative for questions published
        in the past, positive for questions that have yet to be published).
        """

        time = timezone.now() + datetime.timedelta(days=days)
        return Question.objects.create(question_text=question_text, pub_date=time)

    class QuestionIndexViewTests(TestCase):
        
        def test_no_questions(self):

            """
            If no questions exist, an appropriate message is displayed.
            """
            response = self.client.get(reverse('polls:index'))
            self.assertEqual(response.status_code, 200)
            self.assertContains(response, "No polls are available.")
            self.assertQuerysetEqual(response.context['latest_question_list'], [])

        def test_past_question(self):

            """
            Questions with a pub_date in the past are displayed on the
            index page.
            """
            create_question(question_text="Past question.", days=-30)
            response = self.client.get(reverse('polls:index'))
            self.assertQuerysetEqual(
                response.context['latest_question_list'],
                    ['<Question: Past question.>']
            )

        def test_future_question(self):
            
            """
            Questions with a pub_date in the future aren't displayed on
            the index page.
            """
            create_question(question_text="Future question.", days=30)
            response = self.client.get(reverse('polls:index'))
            self.assertContains(response, "No polls are available.")
            self.assertQuerysetEqual(response.context['latest_question_list'], [])
        
        def test_future_question_and_past_question(self):
            
            """
            Even if both past and future questions exist, only past questions
            are displayed.
            """
            create_question(question_text="Past question.", days=-30)
            create_question(question_text="Future question.", days=30)
            response = self.client.get(reverse('polls:index'))
            self.assertQuerysetEqual(
                response.context['latest_question_list'],
                ['<Question: Past question.>']
            )

        def test_two_past_questions(self):
            
            """
            The questions index page may display multiple questions.
            """
            create_question(question_text="Past question 1.", days=-30)
            create_question(question_text="Past question 2.", days=-5)
            response = self.client.get(reverse('polls:index'))
            self.assertQuerysetEqual(
                response.context['latest_question_list'],
                ['<Question: Past question 2.>', '<Question: Past question 1.>']
            )

第一个是问题简洁函数，create_question，包含了重复的问题创建过程。

test\_no\_questions不创建任何问题，而是检查消息"No polls are avaiable."，并且验证latest\_question\_list是空的。注意：django.test.TestCase类提供了一些附加的assertion方法。在这个例子中，我们使用assertContains()和assertQuerysetEqual()。

在test\_future\_question，我们创建了一个pub_date是未来某个日期的一个问题。每个测试方法执行前，数据库都会被重置，因此第一个问题已在这儿了。与此同时，index也不会有任何问题列表。

以此类推，事实上，我们正在用测试讲一个admin录入和用户在页面体验的故事，每次系统中发生改变，都会检查每个问题的状态，只有期望的结果会被发布。

    python manage.py test polls

## 4.5 测试DetailView

我们的工作做的很好，无论如何，虽然未来的问题不会出现在首页，如果他们知道或猜测到正确的URL，用户仍然可以搜索到他们。因此，需要增加相类似的约束到DetailView：

    # polls/views.py

    class DetailView(generic.DetailView):

        ...

        def get_queryset(self):
            """
            Excludes any questions that aren't published yet.
            """
            return Question.objects.filter(pub_date__lte=timezone.now())

当然，我们也会增加一些测试，检查pub\_date是在过去某个时间的问题可以显示；pub\_date是在未来某个时间的问题不会显示。

    # polls/tests.py

    class QuestionDetailViewTests(TestCase):

        def test_future_question(self):
            
            """
            The detail view of a question with a pub_date in the future
            returns a 404 not found.
            """
            future_question = create_question(question_text='Future question.', days=5)
            url = reverse('polls:detail', args=(future_question.id,))
            response = self.client.get(url)
            self.assertEqual(response.status_code, 404)

        def test_past_question(self):
            
            """
            The detail view of a question with a pub_date in the past
            displays the question's text.
            """
            past_question = create_question(question_text='Past Question.', days=-5)
            url = reverse('polls:detail', args=(past_question.id,))
            response = self.client.get(url)
            self.assertContains(response, past_question.question_text)


测试：
    
    python manage.py test polls

## 4.6 更多的测试想法

我们应该给ResultsView增加一个相似的get\_queryset方法，并为他创建一个测试类。它与我们刚创建的很类似；事实上，他们有大量的重复代码。

我们也可以通过别的方式来改善我们的应用，并同时增加测试。举例，发布在站点上的问题，没有投票选项，这看起来有点不合适。因此，我们的view可以检查这个，排除这样的问题。我们的测试将创建一个没有投票选项的Question，然后测试它不能被发布，同样创建一个有投票选项的问题，测试它可以被发布。

也许以admin用户登入系统，应该被允许看见未发布的问题，但是普通的访问者则不能看到未发布的问题。再次强调，不管任何功能被加入到系统中，都应该同时增加一个测试，不管你是先写测试，然后再让写代码通过测试；还是先写业务逻辑代码，再写测试，证明他是正确的。

# 5. 测试，越多越好

似乎是我们测试的增长速度已经超过了我们的控制。按照这个速率，不久的将来测试的代码会超过应用本身的代码，重复的代码与其它代码的简洁相比，毫无美感之言。

没关系，让他们增长吧。大多数时候，你一次可能写一个测试，然后忘掉它。当你持续的开发你的程序，他将显示出他的好处。

有时测试需要被更新。假设我们修改了view，只有带投票选项的问题才会被发布。不这种情况下，我们已存在的大部分测试都会失败 - 他会明确的告诉我们那个测试需要被修改并更新，因此在某种情况度，测试会自己照顾自己。

最糟糙的情况是，随着你持续的开发，你可能发现你有一些测试是冗余的。尽管这不是一个问题；测试冗余是一种好事情。

只要你的测试是合理的安排的，他就不会变的难以管理。好的经验法则包括：

* 每个model或view应该是一个独立的TestClass
* 你想测试的每个条件集应该是一个独立的测试方法
* 测试方法的名字，可以描述他的功能

# 6. 更进一步的测试

这个tutorial仅仅引入了一些基本的测试。这儿有大量的工作，有许多的有用的工具可以达到这个目的。

举例，现在我们的测试只涵盖了model的一些内部逻辑和view发布信息的方式，你可以使用一个浏览器插件，如Selenium测试你的HTML在浏览器中渲染后的效果。这些工具不仅仅允许Django代码的行为，同时检查JavaScript代码的行为。它启动一个浏览器，开始与你的站点交互，好像一个人在操作它一样！Django包含了[LiveServerTestCase](https://docs.djangoproject.com/en/1.11/topics/testing/tools/#django.test.LiveServerTestCase)，可以与像Selenium这样的工具进行交互。

如果你有一个复杂的应用，你可能想要在每次提交后都运行测试，来实现持续集成，由程序自己控制代码质量 \- 至少部分自动化。

识别应用中未被测试代码的最好方式是检查代码覆盖。这也会帮助你识别脆弱(fragile)的代码和永远无法执行到的代码(dead code)。如果你不能测试某一部分代码，它通常意味着代码应该被重构或被移出。代码覆盖可以帮助你识别出dead code。

更多细节，请查看[Integration with coverage.py](https://docs.djangoproject.com/en/1.11/topics/testing/advanced/#topics-testing-code-coverage)

[Testing in Django](https://docs.djangoproject.com/en/1.11/topics/testing/)中有关于测试的更详细的信息

