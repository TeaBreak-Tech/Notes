<font face="黑体" size=4>
#Part1
##import
###package
A package contains \_\_init\_\_.py,this file is often an empty file. With this file, python will know this is a package.
###module
A module is a file containing Python definitions and statements. The file name is the module name with the suffix .py appended. 
***
eg.if we have a module called fibo.py
>from fibo import fib  
>import fibo as fib  
>from fibo import *
***
相对引用中，"."代表模板所在的当前包，".."代表模板的父包。  

##URL 调度器
当一个用户请求Django 站点的一个页面，下面是Django 系统决定执行哪个Python 代码使用的算法：

1. Django 确定使用根 URLconf 模块。通常，这是 ROOT\_URLCONF 设置的值，但如果传入 HttpRequest 对象（用户输入的网址）拥有 urlconf 属性，它的值将被用来代替 ROOT\_URLCONF 设置。

	>'articles/<int:year>/'匹配'articles/2003/'

2. Django 加载该 Python 模块(urls.py)并寻找可用的 urlpatterns 。它是 django.urls.path() .(要先from django.urls import path)  
Django 会按顺序遍历每个 URL 模式，然后会在所请求的URL匹配到第一个模式后停止，并与 path_info 匹配。  
一旦有 URL 匹配成功，Djagno 导入并调用相关的视图，这个视图是一个Python 函数。视图会获得如下参数：  
    (1) 一个 HttpRequest 实例。  
	(2) 关键字参数由路径表达式匹配的任何命名部分组成

	>URLconf： 'articles/<int:year>/<int:month>/<slug:slug>/'  
	>用户输入：/articles/2003/03/building-a-django-site/  
	>传入view模板中函数的关键字参数：request, year=2003, month=3, slug="building-a-django-site"

##include函数
函数 include() 允许引用其它 URLconfs。每当 Django 遇到 include() 时，它会截断与此项匹配的 URL 的部分，并将剩余的字符串发送到 URLconf 以供进一步处理。
***
host文件  将域名和IP地址一一对应

#Part2

##Database
The DATABASES setting must configure a “default” database; any number of additional databases may also be specified.

默认数据库：sqlite  
默认数据库中的一些键值：engine:可选用的后端数据库  
name：数据库的名称（包含绝对路径？）

##INSTALLED_APPS
包含项目中启用的所有Django应用。

django.contrib.admin -- 管理员站点。  
django.contrib.auth -- 认证授权系统。  
django.contrib.contenttypes -- 内容类型框架。  
django.contrib.sessions -- 会话框架。  
django.contrib.messages -- 消息框架。  
django.contrib.staticfiles -- 管理静态文件的框架。

这些应用中的部分应用运行可能需要数据表，所以:  
>python manage.py migrate  
****
migrate 命令检查 INSTALLED_APPS 设置，为其中的每个应用创建需要的数据表

##模型
###定义模型
在Django里写一个数据库驱动的 Web 应用的第一步是定义模型，模型是真实数据的简单明确的描述。它包含了储存的数据所必要的字段和行为。

每个模型被表示为 django.db.models.Model 类的子类。每个模型有许多类变量，它们都表示模型里的一个数据库字段。  
每个字段都是 Field 类的实例，字符字段被表示为 CharField ，日期时间字段被表示为 DateTimeField（？）
***
在我们创建的Choice和Question模型中，我们使用 ForeignKey 定义了一个关系。这将告诉 Django，每个 Choice 对象都关联到一个 Question 对象
###激活模型
创建模型的代码给了Django很多信息，Django可以创建与 Question 和 Choice 对象进行交互的 Python 数据库 API。  

>python manage.py makemigrations polls

makemigrations命令会让Django检测你对模型文件的修改，并且把修改的部分储存为一次迁移。
***
改变模型三部曲：  
1. 编辑models.py文件，改变模型  
2. 运行 python manage.py makemigrations 为模型的改变生成迁移文件。  
3. 运行 python manage.py migrate 来应用数据库迁移。
***
Notes:  
python类的实例方法、静态方法、类方法
实例方法只能被实例对象调用，静态方法（由@staticmethod装饰的方法）、类方法（由@classmethod装饰的方法），可以被类或类的实例对象调用。

实例方法，第一个参数必须要默认传实例对象，一般习惯用self。  
静态方法，参数没有要求。  
类方法，第一个参数必须要默认传类，一般习惯用cls。  
###API使用
为了用 Python 对象展示数据表对象，Django 使用了一套直观的系统：一个模型类代表一张数据表，一个模型类的实例代表数据库表中的一行记录，可以在python交互器中进行数据的插入，修改，删除等操作。  
***
save,get等方法  

#Part3
模板暂时不用管
##views
每个视图必须要做的只有两件事：返回一个包含被请求页面内容的 HttpResponse 对象，或者抛出一个异常
###抛出 404 错误
    from django.http import Http404
    from .models import Question
    def detail(request, question_id):
        try:
            question = Question.objects.get(pk=question_id)
        except Question.DoesNotExist:
            raise Http404("Question does not exist")
    return HttpResponse(...)
###get\_object\_or\_404()
The get_object_or_404() function takes **a Django model** as its first argument and **an arbitrary number of keyword arguments**, which it **passes to the get() function** of the model's manager. It raises Http404 if the **object doesn't exist**.
>question = get\_object\_or\_404(Question, pk=question_id)

#Part5
##自动化测试
创建在应用的tests.py中 
 
	import datetime	
	from django.test import TestCase
	from django.utils import timezone
	from .models import Question

	class QuestionModelTests(TestCase):

    	def test_was_published_recently_with_future_question(self):
            time = timezone.now() + datetime.timedelta(days=30)
        	future_question = Question(pub_date=time)
        	self.assertIs(future_question.was_published_recently(), False)
自动化测试的运行过程：  

- python manage.py test polls 将会寻找 polls 应用里的测试代码  
- 它找到了 django.test.TestCase 的一个子类  
- 它创建一个**特殊的数据库**供测试使用  
- 它在类中寻找测试方法
- 在 test\_was\_published\_recently\_with\_future\_question 方法中，它创建了一个 pub\_date 值为 30 天后的 Question 实例。  
- 接着使用 assertls() 方法，发现 was\_published\_recently() 返回了 True，而我们期望它返回 False