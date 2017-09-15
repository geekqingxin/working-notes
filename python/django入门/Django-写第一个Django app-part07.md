[Writing your first Django app, part 7](https://docs.djangoproject.com/en/1.11/intro/tutorial07/)

我们继续我们的Web-poll应用，我们将关注在定制由Django自动生成的admin站点。

# 1. 定制admin form

通过使用admin.site.register(Question)来注册Question对象，Django可以构造出一个默认的form表单。通常情况下，你想要定制admin form的外观和工作方式。你可以在注册对象时，同时告诉Django一些选项，来实现这个目的。

接下来，我们来实践一下，调整一下edit form中字段的顺序。使用以下内容替换admin.site.register(Question)：

    # polls/admin.py

    from django.contrib import admin
    from .models import Question

    class QuestionAdmin(admin.ModelAdmin):
    
        fields = ['pub_date', 'question_text']

    admin.site.register(Question, QuestionAdmin)

你可以参考这种模式 - 创建一个model admin类，然后把它作为admin.site.register()的第二个参数 - 如果你想要定制model的admin选项，你都可以这样实现。

刷新页面后，你就会发现变化，现在"Publication date"将会在"Question"前面被显示。

只有两个字段时，这没有什么值得称道的地方，但是如果admin form有很多字段时，选择一个合理的顺序是非常重要的。

说到有很多字段的form时，你可能想要把form划分成fieldset：

    # polls/admin.py

    from django.contrib import admin

    from .models import Question

    class QuestionAdmin(admin.ModelAdmin):

        fieldsets = [
            (None, {'fields' : ['question_text']}),
            ('Date information', {'fields' : ['pub_date']})
        ]

    admin.site.register(Question, QuestionAdmin)


在fieldsets中每个元组(tuple)的第一个元素是fieldset的标题。


# 2. 增加相关对象

好，我们完成了Question admin页面，但是每个问题有多个选项(Choice)，然后admin不显示这些选项。

有两种方式可以解决这个问题。第一种方案是像注册Question一样，我们也在admin中注册Choice。

这比较简单：

    # polls/admin.py
    
    from django.contrib import admin
    from .models import Choice, Question

    # ...
    admin.site.register(Choice)

现在Choices在Django admin中已经可以使用了。在这个form中，"Question"字段是一个下拉框，包含了数据库中每个问题。Django知道要将一个ForeignKey表示为admin中的一个<select\>框。当前情况中，只有存在一个问题。

同时也要注意"Question"后面的"Add Another"("+"号图标的链接)链接。所有有外链属性的字段，都会产生一个"Add Another"链接。当你点击"Add Another"，你会打开一个包含"Add question"表单的弹出窗口。如果你在这个窗口中添加一个问题，然后点击"保存"，Django将保存问题到数据库，并自动把他加载到"Add choice"表单的selected choice中。

移出Choice对象的register()调用。然后，Question注册代码：

    # polls/admin.py

    from django.contrib import admin
    from .models import Choice, Question

    class ChoiceInline(admin.StackedInline):
        model = Choice
        extra = 3

    class QuestionAdmin(admin.ModelAdmin):

        fieldsets = [
            (None, {'fields' : ['question_text']}),
            ('Date information', {'fields' : ['pub_date'], 'classes' : ['collapse']}),
        ]
        inlines = [ChoiceInline]

    admin.site.register(Question, QuestionAdmin)




