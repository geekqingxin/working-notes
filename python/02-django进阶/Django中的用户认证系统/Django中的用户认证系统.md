[User authentication in Django](https://docs.djangoproject.com/en/1.11/topics/auth/)

Django包含一个用户认证系统。它处理用户的帐户、组、权限和基于cookie的用户session。文档的这部分介绍了默认实现是如何工作的，也介绍了如何根据你的需要来[定制和扩展](https://docs.djangoproject.com/en/1.11/topics/auth/customizing/)。

# 1. 总览

Django认证系统处理认证(authentication)和鉴权(authorization)。简单来说，认证验证用户是它所声称的人，鉴权指定了被认证的用户允许做什么事儿。术语认证通常就是指的这两部工作。

认证系统包含：

* 用户
* 权限：二进制标志(yes/no)来指示用户是否允许执行某个特定的任务。
* 组：将标签和权限应用到多个用户的通用方法。
* 一个可配置的密码Hash系统
* 提供用户登录的form表单和view，或一些限制内容(restricting content)
* 一个可插拔的后台系统

Django中认证系统的目标是非常通用的，不提供web认证系统常见的特性。一些公共问题的解决办法由第三方包来实现：
* 密码长度检测(Password strength checking)
* 登录次数限制(Throttling of login attempts)
* 第三方认证(如，OAuth)

# 2. 安装

认证支持是被捆绑作为一个Django contrib module的一部分：django.contrib.auth。默认情况下，要求的一些配置在通过django-admin startproject创建时，已经包含在settings.py中了，这由两部分组成：

* INSTALLED_APPS配置：
    * 'django.contrib.auth'：包含了认证框架的核心功能和一些默认的model。
    * 'django.contrib.contenttypes'：是一个Django [content type system](https://docs.djangoproject.com/en/1.11/ref/contrib/contenttypes/)，允许将权限和你创建的model进行关联。

* MIDDLEWARE配置：
    * SessionMiddleware管理session。
    * AuthenticationMiddleware使用session将用户的请求关联起来。

有了这些设置，运行manage.py migrate命令为定义在INSTALLED APP中的应用，创建认证和权限相关model的数据库表。

# 3. 用法

* [应用Django默认实现](01-应用Django默认实现.md)
* [默认实现的API参考文档]()
* [定制用户和认证]()
* [Django中的密码管理]()

