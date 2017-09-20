[Writing your first Django app, part 2](https://docs.djangoproject.com/en/1.11/intro/tutorial02/)

# 1. 配置数据库

    vi mysite/settings.py

默认数据库是SQLite。


    #默认数据库配置如下

    # Database
    # https://docs.djangoproject.com/en/1.11/ref/settings/#databases
    
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
        }
    }

* ENGINE - 如果希望切换到其它产品，可以调整ENGINE

    * django.db.backends.sqlite3
    * django.db.backends.postgresql
    * django.db.backends.mysql
    * django.db.backends.oracle

    [其它数据库对应的ENGINE](https://docs.djangoproject.com/en/1.11/ref/databases/#third-party-notes)

* NAME：数据库名字。如果你使用的是SQLite，数据库就是计算机上的一个文件。这种情况下，NAME应该是一个绝对的全路径，包含文件的文件名。默认值是os.path.join(BASE_DIR, 'db.sqlite3')，文件将被存储在工程目录下。


[请参见，数据库设置的详细描述](https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-DATABASES)

修改mysite/settings.py文件中的TIME_ZONE，设置成你所在地区的TIME ZONE。

留意文件头部的INSTALLED_APPS部分：

    # Application definition

    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',
    ]

该属性列出了所有已激活的Django application。 Apps可以被多个工程使用，你可以打包，然后分发给其它工程使用。

默认情况下，INSTALLED_APPS包含以下App：

* django.contrib.admin：admin站点
* django.contrib.auth：授权系统
* django.contrib.contenttypes：content type框架
* django.contrib.sessions：session框架
* django.contrib.messages：消息框架
* django.contrib.staticfiles：管理静态文件的框架

在使用以上应用前，我们需要在数据库中创建表。创建表可以通过以下命令来完成：

    python3 manage.py migrate

migrate命令通过查看INSTALLED_APPS的设置，根据mysite/settings.py中的数据库配置，创建所需要的数据表。

# 2. 创建model

在我们的Poll app中，我们将创建两个model：Question和Choice。Question包含一个问题和一个发布日期。Choice包含两个字段，选择文字和投票记录。每个Choice都会关联到某个Question。

    # 编辑polls/models.py

    from django.db import models


    class Question(models.Model):
        question_text = models.CharField(max_length=200)
        pub_date = models.DateTimeField('date published')


    class Choice(models.Model):
        question = models.ForeignKey(Question, on_delete=models.CASCADE)
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)

Django支持所有的数据库关系：many-to-one, many-to-many和one-to-one。

# 3. 激活model

上面的model代码给Django提供了许多信息，通过它，Django可以：

* 为App创建数据库中的schema(也就是数据表, table)
* 创建为Question和Choice对象创建Python的数据库访问API

但是，首先我们需要设置工程，让他知道polls app是被安装的。

为了把App包含到我们的工程中，我们需要在INSTALLED_APPS配置中，加入APP配置类的引用。PollsConfig类是在polls/apps.py文件中，因此它的路径是'polls.apps.PollsConfig'。编译mysite/settings.py文件，加入路径到INSTALLED_APPS设置，具体如下：

现在Django已知道有新的App被加入，运行另外一个命令：

    python3 manage.py makemigrations polls


    # 输出结果

    Migrations for 'polls':
      polls/migrations/0001_initial.py
        - Create model Choice
        - Create model Question
        - Add field question to choice

迁移产生的文件存储在polls/migrations目录下，打印迁移的SQL：

    python3 manage.py sqlmigrate polls 0001

再次运行migrate，为这些model创建表：

    python3 manage.py migrate


修改model的三步指南：

* 修改model，在models.py文件中
* 运行python manage.py makemigrations，为这些变化创建migrations
* 运行python manage.py migrate，应用这些变化到数据库中

[如何应用manage.py工具包](https://docs.djangoproject.com/en/1.11/ref/django-admin/)

# 4. 与API交互

进入python交互shell，与Django交互

    python manage.py shell

浏览数据库API：

    from polls.models import Question, Choice

    Question.objects.all()


    from django.utils import timezone
    
    q = Question(question_text="What's new?", pub_date=timezone.now())
    q.save()

    q.id
    q.question_text
    q.pub_date

    q.question_text = "What's up?"
    q.save()

    Question.objects.all()

    # 输出结果

    <QuerySet [<Question: Question object>]>

对于<Questsion: Question object>是没有用处，无法完全表达这个对象。让我们修复这个问题，为Question和Choice添加__str__()方法。


    # 为Question添加__str__()方法
    def __str__(self):
        return self.question_text


    # 为Choice添加__str__()方法
    def __str__(self):
        return self.choice_text

添加定制方法：

    import datetime

    from django.db import models
    from django.utils import timezone

    class Question(models.model):
        
        def was_published_recently(self):
        
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

再次运行python manage.py shell：

    python manage.py shell

    from polls.models import Question, Choice
    
    Question.objects.all()

    Question.objects.filter(id=1)

    Question.objects.filter(question_text__startswith='What')

    
    from django.utils import timezone
    current_year = timezone.now().year
    Question.objects.get(pub_date__year=current_year)

    Question.objects.get(id=2)

    Question.objects.get(pk=1)

    q = Question.objects.get(pk=1)

    q.was_published_recently()

    q = Question.objects.get(pk=1)

    q.choice_set.all()

    q.choice_set.create(choice_text='Not much', votes=0)

    q.choice_set.create(choice_text='The sky', votes=0)

    c = q.choice_set.create(choice_text='Just hacking again', votes=0)

    c.question
    
    q.choice_set.all()

    q.choice_set.count()

    Choice.objects.filter(question__pub_date__year=current_year)


    c = q.choice_set.filter(choice_text__startswith='Just hacking')
    c.delete()


[Accessing related objects](https://docs.djangoproject.com/en/1.11/ref/models/relations/)

[Field lookups](https://docs.djangoproject.com/en/1.11/topics/db/queries/#field-lookups-intro)

[数据库API](https://docs.djangoproject.com/en/1.11/topics/db/queries/)


# 5. 引入Django Admin


## 5.1 创建admin用户

    python3 manage.py createsuperuser

    # 输入用户名
        admin
    # 输入邮箱
        liuyf8688@126.com
    # 输入密码
        1qaz@WSX

## 5.2 启动开发服务器

    python3 manage.py runserver

[访问Admin站点](http://127.0.0.1:8000/admin/)

默认情况下，翻译功能是开启的，你可能会看到使用你本地的语言显示的登录界面。另外这个功能，也依赖于浏览器设置。

## 5.3 进入admin站点

首先看到是groups和users。该功能由django.contrib.auth验证框架提供。

## 5.4 修改，使poll app可以在admin中修改

告知admin，Question对象由admin界面。为了实现这个目的，修改polls.admin.py文件：

    from .models import Question

    admin.site.register(Question)

## 5.5 刷新admin

现在在Admin中将看到Question对象。

点击Questions，进入问题列表。

点击某个问题，可以对问题进行修改。



# X. 问题

## X.1 忘记admin密码

    python manage.py createsuperuser

    # 输入用户名

        liuyf

    # 输入邮箱
        
        liuyf8688@126.com

    # 输入密码

        1qaz@WSX

    # 输入确认密码

        1qaz@WSX
    
















