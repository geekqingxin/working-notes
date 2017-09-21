[What to read next](https://docs.djangoproject.com/en/1.11/intro/whatsnext/)

# 1. 文档是如何被组织的？

Django主文档为了满足不同的需要，被划分成了多个块：

* [introductory material](https://docs.djangoproject.com/en/1.11/intro/)是为了Django新手或一般的Web开发者设计的。它没有解释深层次的内容，而是从高视角说明了如何进行Django开发。

* [topic guides](https://docs.djangoproject.com/en/1.11/topics/)，深度解读了Django的各个部分。这儿有Django [model system](https://docs.djangoproject.com/en/1.11/topics/db/)，[template engine](https://docs.djangoproject.com/en/1.11/topics/templates/)和[forms framework](https://docs.djangoproject.com/en/1.11/topics/forms/)的完事指南。

    这部分是需要花时间来认真阅读的，通过这部分的学习，将更了解Django内部的东西。

* Web开发通常非常宽泛，而不深入 - 问题通常横跨很多部分的内容。我们写了一系列[how-to guides](https://docs.djangoproject.com/en/1.11/howto/)来回答共同的"How do I ...?"问题。在这部分，你可以找到关于[generating PDFs with Django](https://docs.djangoproject.com/en/1.11/howto/outputting-pdf/)，[writing custom template tags](https://docs.djangoproject.com/en/1.11/howto/custom-template-tags/)等信息。
    
    通用问题的答案也可以查阅[FAQ](https://docs.djangoproject.com/en/1.11/faq/)。
* 指南和how-to没有涵盖Django中可用的类，函数和方法 - 如果你想要学习这部分，可以参考接下来的文档。这部分内容被放在了[reference](https://docs.djangoproject.com/en/1.11/ref/)中。
* 如果你想了解关于发布公用的项目，我们文档中有[几部分](https://docs.djangoproject.com/en/1.11/howto/deployment/)介绍不同的发布设置，同样也有一个[发布checklist](https://docs.djangoproject.com/en/1.11/howto/deployment/checklist/)。


# 2. How documentation is updated

正如Django代码库一样，每天都在开发和改进，我们的文档也在一直改善。我们修改文档基于以下几个原因：

* 内容上的修改，例如语法、排版调整
* 为了给已存在的部分增加内容或示例，而因发部分内容被扩充
* 增加对没有文档化的Django特性进行描述
* 当新Django新特性加入或Django APIs做了调整时，增加相关描述信息

Django文档也像代码一样被加入到了版本控制系统中。他位于git仓库的[docs](https://github.com/django/django/tree/master/docs)目录中。每个在线文档在仓库中是一个独立的文本文件。

# 3. Where to get it

你可以通过以下几种方式来获取Django文档：

## 3.1 On the web

最新的Django文档是位于[https://docs.djangoproject.com/en/dev/](https://docs.djangoproject.com/en/dev/)。这些HTML页面是根据版本控制系统中的文本文件自动生成的。这也意味着它是与Django相匹配的 - 他们包含了最新的修改及附加内容，他们讨论了最新的Django特性，这些特性只对采用了Django开发版本的用户是可用的。

我们鼓励你在[ticket system](https://code.djangoproject.com/)中提交修改的内容，修正和建议来帮助改善文档。Django会监控ticket系统，并使用你的反馈来发送文档。

注意：ticket应该是明确的关联到对应的文档，而不是问一个宽泛的技术支持问题。如果你想要获取Django特定设置的帮助，请使用[django-users](https://docs.djangoproject.com/en/1.11/internals/mailing-lists/#django-users-mailing-list)邮件列表或[#django IRC channel](irc://irc.freenode.net/django)。


## 3.2 In plain text

为了离线阅读，或只是为了方便，你可以直接阅读文本格式的Django文档。

如果你正在使用Django官方的release，注意代码压缩包中包含了docs目录，这个目录包含了该release的全部文档。

如果你正在使用Django的开发版本(aka "trunk")，docs目录中包含了全部文档。你可以通过git来更新文档。

使用文本文档的一个好处是可以使用Unix的grep来搜索文档中的关键字。例如，以下命令，将显示给你Django文档中所有出现"max_length"的位置：

    grep -r max_length /path/to/django/docs/


## 3.3 As HTML, locally

你可以获取一份HTML文档到本地，只需要要执行以下几步即可得到HTML：

* Django文档使用了一个Sphinx系统，把文档从文本转换成HTML。通过pip安装Sphinx：

    pip install Sphinx

* 然后使用Makefile转换文档到HTML：

    cd path/to/django/docs
    make html

    你需要使用[GNU make](https://www.gnu.org/software/make/)来完成这一步操作。

    如果你在Windows上，你可以使用批处理文件来替代：

        cd path\to\django\docs
        make.bat html

* 接下来，你将会在doc/_build/html中找到Django HTML文档。


# 4. Differences between versions

像之前说的那样，在git仓库中的文本文件包含了最新的内容。这些变化通常包含Django新加入到开发版中的特性文档 - the Git("trunk") version of Django。基于这个原因，我们要澄清一下框架不同版本文档的policy。

我们的policy：

* djangoproject.com是git中最新文档的HTML版本。这个版本与最新的官方release一致，并且加入了自上次release后增加和修改的特性。
* 由于我们在开发版中会增加特性，我们会尝试在相同的git提交事务中包含对应的文档更新。
* 在文档中特性的修改或附加内容，我们使用"New in version X.Y"，X.Y是下一个release版本(也就是正在开发的版本)。
* 文档的修改和改进可能会被移植到之前的release branch，这由委员会来决定，因此一旦一个Django的版本不再被支持，那么文档将不会再被更新。
* [主文档页面](https://docs.djangoproject.com/en/dev/)包含了所有之前版本的文档链接。请确保你在查看你所使用的Django版本的文档。





