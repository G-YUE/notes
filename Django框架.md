# Django框架

[toc]

## 基本配置

### 1.安装
	
	pip3 install django

### 2.创建django project

#### 通过终端创建
1.创建django程序

	django-admin startproject <proecjt名>
	例如：
		django-admin startproject mysite
	
2.创建project中的app文件目录
	
	python manage.py startapp <app名>
	例如：
		python manage.py startapp app01
	
3.进入程序目录

	cd mysite
	
4.启动socket服务端，等待用户发送请求(不写端口默认8000)

	python manage.py runserver 127.0.0.1:8080
	
5.其他命令

	python manage.py createsuperuser  # 用于django admin创建超级用户
	python manage.py makemigrations  # 记录对models.py文件的改动
	python manage.py migrate  # 将改动映射到数据库
	
#### 通过IDE创建
通过IDE（pycharm等）创建django project，以上命令均自动执行
### 3.project目录结构
目录结构介绍

	- mysite(~/PycharmProjects/mysite)
		|- app01/ 
			|- admin.py  # Django自带管理配置后台
			|- models.py  # 写类，根据类创建数据库表
			|- test.py  # 单元测试
			|- views.py  # 视图函数
			|- apps.py  # 配置文件 
		|- mysite/
			|- __init__.py
			|- setting.py
			|- urls.py
			|- wsgi.py 
		|- static/
		|- templates/
		|- utils/
		|- manage.py
### 4.setting.py配置文件
#### 指定template目录
在settings.py 文件中，指定template文件路径

```py
TEMPLATES = [
    {
    	...
        'DIRS': [os.path.join(BASE_DIR, 'templates')],
        ...
     },
```

#### 指定static目录
在settings.py 文件中，添加STATICFILES_DIRS，<font color='red'>注意要加逗号</font>

```py
STATIC_URL = '/static/'
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),  # 逗号
)
```
#### 配置数据库
* Django默认使用的数据库是sqllite3，我们要使用mysql就需要利用第三方工具连接数据库
* Django内部连接mysql的第三方工具是MySQLDB，但是python3不再支持MySQLDB
* 所以得修改django默认连接mysql方式为pymysql

1.创建mysql数据库
2.修改配置settings.py

```py	
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'dbname',
        'USER': 'root',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': 3306,
    }
}
```
3.修改项目同名文件下__init__.py文件
pymsql替换内部的myqldb，修改django默认连接mysql第三方工具

```py	
import pymysql
pymysql.install_as_MySQLdb()
```


-------


## 路由系统

### 路由系统简介
路由就是在urls.py文件中的url和函数的对应关系

```py
# urls.py文件
from django.conf.urls import url
from django.contrib import admin

# 对应关系
urlpatterns = [
    # url(r'^admin/', admin.site.urls),
    url(r'^login/', login),  # 对应关系
]
```
其中，
- 对应关系：／login/, login 
- 请求参数：request 
- 返回方式：HttpResponse\render\redirect 

案例

```py
from django.conf.urls import url
from django.contrib import admin

from django.shortcuts import HttpResponse,render,redirect
def login(request):
    print(request.GET)
    if request.method == "GET":
        return render(request,'login.html')
    else:
        # 用户POST提交的数据（请求体）
        u = request.POST.get('user')
        p = request.POST.get('pwd')
        if u == 'root' and p == '123':
            # 登录成功
            return redirect('/index/')
        else:
            # 登录失败
            return render(request,'login.html',{'msg': '用户名或密码错误'})
            
urlpatterns = [
# url(r'^admin/', admin.site.urls),
url(r'^login/', login),
url(r'^index/', index),
]
```
### 1.单一路由对应

	url(r'^index$', views.index),
	
### 2.基于正则的路由
基于正则的路由可以传递参数，根据传递方式可以分为，静态路由和动态路由两种
#### 静态路由
我们都知道url可以通过get方式传参数，这种方式被称作静态路由

	url(r'^index/?id=12', views.index),  
	
	# 对应views.py中的def index(request)，可以根据request.GET.get('id')获取参数值

#### 动态路由
路由中的url部分其实就是正则表达式，所以可以使用正则表达式替代get传参数，传递方式有两种 传递位置参数 和 传递命名参数

##### 位置参数
位置参数对参数顺序有要求

	url(r'^index/(\w+)/', views.index),  # (\w+)可以匹配任意字母或数字或下划线或汉字作为位置参数，对参数顺序有要求
	
	# 匹配的参数会被def index(request,a)中的参数a接收

##### 命名参数
参数通过参数名对应，所以对顺序没要求

	url(r'^index/(?P<a2>\d+)/(?P<a1>\d+)/', views.index),   # (?P<a1>\d+)可以指定匹配任意数字并且参数名为a1
	
	# 匹配的参数被def index(request,a1,a2)中的参数a1接收，对参数顺序没有要求

注意：位置传递和命名传递不可混用，所以使用要统一

#### 终止符
因为url就是为正则表达式，所以url前部匹配成功后就会调用函数，所以为了完美匹配可以在url结尾跟/或者$

	url(r'^index$', views.index),  # 只会匹配index
	url(r'^index', views.index),  # 会全部匹配index开头字符串
	
#### 伪静态
SEO（Search Engine Optimization，搜索引擎优化）静态网页的搜索权重高于动态网页，所以通过伪造动态网页为静态网页后缀达到权重增加的目的

	url(r'^index/(/w+).html$', views.index),  # 表示页面以.html结尾
			
所以，url正则表达式方式 + 伪静态 = 动态路由
不出现‘?id=..’get传参的写法，可以提高SEO的搜索权重
		
#### 默认页面
如果用户输入url不存在，可以制定到默认页面

	url(r'^', <指定函数名>),
	
### 3.添加额外的参数

	url(r'^manage/(?P<title>\w*)', views.manage,{'name':'kirs'}),
	
### 4.路由分发 
根据据app对路由规则进行分类

一个Django项目中可能会有众多业务线（众多app），如果一都使用同一个路由系统可能会导致url重名问题，所以可以根据每个app建立自己的urls.py文件，通过项目的路由系统来找到各自app的路由对应关系


1.在app01中新建urls.py,并import自己的views

```py
from django.conf.urls import url
from app01 import views
urlpatterns = [
    url(r'^index.html$', views.index),
]
```
2.在项目的urls.py中import包include,
在项目的urls.py中写入app01url和include内容

```py		
from django.conf.urls import url,include
urlpatterns = [
	url(r'^app01/', include('app01.urls')), 
]
```
这样，url来了，项目路由会先找对应app，找到后，去对应app中的urls中找对应关系
		
### 5.url关系命名
给url与函数关系命名

#### 视图函数应用
将来在视图函数中可以根据别名反生成url

	- 1.首先，urls.py中
		url(r'^index/', views.index, name='name1'),
		
	- 2.在views.py中import包reverse
		from django.urls import reverse
		url = reverse('name1')  # 获取到/index/
		
反生成带特定 位置参数的url
	
	- 1.urls.py中
		url(r'^index/(\w+)/', views.index, name='name1'),
	- 2.在views.py中
		from django.urls import reverse
		url = reverse('name1',args=(abc,))  # 获取到/index/abc/
		
反生成带特定 命名参数的url
	
	- 1.urls.py中
		url(r'^index/(?P<b1>\w+)/', views.index, name='name1'),
	- 2.在views.py中
		from django.urls import reverse
		url = reverse('name1',kwargs={'b1':abcd,})  # 获取到/index/abcd/
		
#### html应用
将来在html页面中可以根据别名取得url地址

	- 1.urls.py中
		url(r'^index/', views.index, name='name1'),
	
	- 2.a标签取url 
		- 原来写法
			<a href="/index/"></a>
		
		- 现在写法
			<a href="{% url "name1" %}"></a>  # 效果就是/index/
		
动态传递参数怎么办?

	- 1.urls.py中
		url(r'^index/(\w+)/(\w+)/', views.index, name='name1'),
	
	- 2.a标签取url
		- 原来写法: 
			<a href="/index/(\w+)/(\w+)/"></a>	
		- 现在写法:
			<a href="{% url "name1" 参数1 参数2 %}"></a>  # 效果就是/index/参数1/参数2
			# <a href="{% url "name1" x y %}"></a>  # 效果就是/index/peter/alex
	
PS：url与函数关系命名可以用于权限管理简化存储内容：数据库存储别名即可，就不用存储很长的url地址


-------

## 模版引擎

### 模版的执行
模版的创建过程，对于模版，其实就是读取模版（其中嵌套着模版标签），然后将 Model 中获取的数据插入到模版中，最后将信息返回给用户。

其中，返回方式：HttpResponse\render\redirect 

```py
#views.py
from django.shortcuts import HttpResponse, render, redirect

# 视图函数
def login(request): 
    # return HttpResponse('massage')  # 直接获取内容返回用户
    return render(request, 'login.html', {'key':'value'})  # 自动找到templates路径下的login.html文件，读取内容并返回给用户
    # return redirect('/login/')  # 重定向文件
```

#### render 返回参数

返回参数可以是

	字符串：'name': 'alex',
	列表：'users':['tom','kirs'],
	字典：'user_dict':{'k1': '123','k2':'234'},
	列表套字典：
	'user_list_dict': [
	                {'id':1, 'name': 'alex', 'email': 'alex3714@163.com'},
	                {'id':2, 'name': 'alex2', 'email': 'alex3714@1632.com'},
	                {'id':3, 'name': 'alex3', 'email': 'alex3713@1632.com'},
	            ]

html 中取值，直接通过.取值

	{{users.0}}  {{user.1}}
	{{user_dict.k1}}  {{user_dict.k2}}
案例

```py
def index(request):
    # return HttpResponse('Index')
    return render(
        request,
        'index.html',
        {
            'name': 'alex',
            'users':['李志','李杰'],
            'user_dict':{'k1': 'v1','k2':'v2'},
            'user_list_dict': [
                {'id':1, 'name': 'alex', 'email': 'alex3714@163.com'},
                {'id':2, 'name': 'alex2', 'email': 'alex3714@1632.com'},
                {'id':3, 'name': 'alex3', 'email': 'alex3713@1632.com'},
            ]
        }
    )
```

### 模版语言

#### html 渲染方式

	{{ }}
	
	{% for i in [列表|字典] %}
		{{i.列1}}
		{{i.列2}}
		...
	{% endfor%}
	
	{% if 条件%}
		...
	{% else %}
		...
	{% endif %}
	
#### 母版
之前说过的母版就是将公用内容存到html，其他子板extends即可

	{% extends "base.html" %}

#### include 
类似于python中的import
可以自定义html小组件，将小组件倒入其他页面应用，并且一个页面呢可以多次导入

	{% include 'pub.html' %}  # pub.html为组件
	
```html
#pub.html
<div>
<div class='...'>{{ name }}<>
<div>
```
PS：并且组件中可以接受后台参数
#### 模版引擎
在模版引擎中传入函数名，自动执行函数，不用加括号
通过后台传来的字典具有方法  
keys	

	- 遍历key
		{% for i in user_list.keys%}
			{{i}}
		{% endfor%}
values

	- 遍历value
		{% for i in user_list.values%}
			{{i}}
		{% endfor%}
items

	- 遍历key和value
		{% for k,v in user_list.items%}
			{{k}}
			{{v}}
		{% endfor%}
#### 内置方法
内置方法通过|方法名执行
	
	- 将字符串大写
		{{ name|upper }}

### 模版自定义函数

#### 创建模版自定义函数simple_filter
1.在app创建一个名为templatetags文件（名称不可改）
2.创建任意templatetags/xx.py文件,函数增加装饰器@register.filter

```py
#xx.py
from django import template
		
register = temlate.Library()
	
@register.filter
def my_upper(value):
	return value.upper()
```
3.导入文件：在html文件中导入之前创建的 xx.py 文件名

	{% load xxx %}
4.使用：

	{{name|my_upper}}
5.settings.py注册app
	
	INSTALLED_APPS = (
    ...
    'app',
	)

创建模版自定义函数simple_filter最多只能有两个参数，但是可以做条件判断
	
	- 最多两个参数,方式： {{第一个参数|函数名称:"第二个参数"}}
		def my_upper(value,arg):
			return value+arg
			
		{{name|my_upper:'abc'}}
	- 可以做条件判断

	
#### 创建模版自定义函数simple_tag	
在以上创建过程中，如果函数增加装饰器@register.simple_tag

```py
	@register.simple_tag
	def my_lower(value):
		return value.lower()
```
使用时参数个数无限制
	
	- 参数无限制：{% 函数名 参数 参数 ...%}
		def my_lower(value,arg1,arg2,...):
			return value + arg1 + arg2 +...
			
		{% my_lower name 'abc' 'bcd' ...%}
		

-------

## 视图函数

### request 请求信息

request.method  - 用户请求方法，GET或者POST
request.GET     - 获取GET方式提交的数据
request.POST    — 获取POST方式提交的数据

request.GET.get() 
request.POST.get()
request.POST.getlist() - 获取列表

其中，客户端POST方式提交，request同样可以获取GET数据，反之不可以

### 视图函数 CBV FBV

路由系统通过反射方式匹配method（post、get、put等）。

用户发来请求，url匹配到类，将method当作参数传入类，通过getattr找到对应方法。

#### FBV (Function-based views)
路由系统中，url对应函数就是FBV

urls.py
	
	- 直接跟方法名
		url(r'^login.html$',views.login),
		
views.py
	
	- 类名需要继承View，可以定义get、post等方法
		def login(request):
	    	return render(request, "login.html")

#### CBV (Class-based views)

同样，url也可以对应类，那么这就是CBV
	
urls.py
	
	- 类名后跟.as_view()
		url(r'^login.html$',views.Login.as_view()),
	
	
views.py
	
	- 类名需要继承View，可以定义get、post等方法
		from django.views import View
		
		class Login(View):
		    def get(self, request):
		        return render(request, "login.html")
		
		    def post(self, request):
		        print(request.POST.get("user"))
		        return HttpResponse("login.post")
		
from表单只有get、post方法，ajax不仅有post和get方法，还有很多其它方法，如put、delete等
PS：约定俗成：get查 post创建 put更新 delete删除

#### dispatch 方法 
在CBV的get、post方法执行之前View内部会先执行dispatch方法

```py
def dispatch(self, request, *args, **kwargs):
   if request.method.lower() in self.http_method_names:
       handler = getattr(self, request.method.lower(), self.http_method_not_allowed)
   else:
       handler = self.http_method_not_allowed
   return handler(request, *args, **kwargs)
```	     
所以我们可以重写dispatch方法，起到类似装饰器功能

装饰get、post等方法

```py	
def dispatch(self,request,*args,**kwargs):
	#执行get、post前代码
	func = super(Login,self).dispatch(request,*args,**kwargs)
	#执行get、post后代码
	return func
```

-------


## 中间件
### 原理介绍
django 中的中间件（middleware），在django中，中间件其实就是一个类，在请求到来和结束后，django会根据自己的规则在合适的时机执行中间件中相应的方法。

	请求-->
			中间件类1方法1-->
							中间件类2方法1-->
										   ...-->
										         路由系系统and视图函数 
										   ...<--
							中间件类2方法2<--
			中间件类1方法2<--				
	响应<--	
		
	其中,
	方法1都是 process_request()，
	方法二都是 process_response()
	
在django项目的settings模块中，有一个 MIDDLEWARE_CLASSES 变量，其中每一个元素就是一个中间件

```py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```
请求按照MIDDLEWARE中的中间件顺序执行，如果其中一个中间件在process_request()出现错误，就会直接执行这个中间件的process_response()，请求停止继续执行，返回错误信息

	请求-->
			中间件类1方法1-->
							中间件类2方法1-出错
										   |
							中间件类2方法2<--
			中间件类1方法2<--				
	响应<--	
	
### 自定义中间件

应用环境：于用户认证，替代auth装饰器，省去给全部视图函数添加装饰器

1.新建py文件，倒入入MiddlewareMixin
2.类中函数：process_request、process_response
3.注册中间件py文件

```py
# m1.py
from django.utils.deprecation import MiddlewareMixin
from django.shortcuts import HttpResponse
class M1(MiddlewareMixin):
    def process_request(self,request):
        print('m1.process_request')

    def process_view(self, request, callback, callback_args, callback_kwargs):
        print('m1.process_view')
        # response = callback(request,*callback_args,**callback_kwargs)
        # return response

    def process_response(self,request,response):
        print('m1.process_response')
        return response

    def process_exception(self, request, exception):
        print('m1.process_exception')

    def process_template_response(self,request,response):
        """
        视图函数的返回值中，如果有render方法，才被调用
        :param request:
        :param response:
        :return:
        """
        print('m1.process_template_response')
        return response

```
其中，
1.process_request 不能有return，如果return，中间件不再继续执行，直接返回
2.process_response 必须需要return 返回值
3.proccess_view 匹配到对应函数，并不会执行，
4.proccess_exception 异常处理，只有视图函数错误才会执行，中间件会从最后一个类中的proccess_exception开始向前执行，proccess_exception全部执行结束后，再从最后一个类中的process_response开始向前返回
5.proccess_template_view 视图函数有render，才执行,对所有请求，或一部分请求做批量处理,应用：记录日志功能

```py
# settings.py
MIDDLEWARE = [
    ...
    m1.Middle1,
]
```

Ajax请求返回需要后台通过HttpResponse返回json.dumps字典数据,这个功能可以封装成另一个类调用render，利用中间件中的proccess_template_view实现

```py
# views.py
from django.shortcuts import render,HttpResponse,redirect

class JSONResponse:
    def __init__(self,req,status,msg):
        self.req = req
        self.status = status
        self.msg = msg
    def render(self):
        import json
        ret = {
            'status': self.status,
            'msg':self.msg
        }
        return HttpResponse(json.dumps(ret))

def test(request):
    # print('test')
    # return HttpResponse('...')
    ret = {}
    return JSONResponse(request,True,"错误信息")
```
PS：在django1.10中，如果process_request中有异常或者return，中间件会从当前类的process_response开始返回

在django1.7或者1.8中，如果process_request中有异常或者return，中间件会从最后一个类中的process_response开始向前返回

-------

## admin


## Model

介绍Model之前先介绍一个概念：ORM

### ORM

#### 概述
ORM，即Object-Relational Mapping（对象关系映射），它的作用在关系型数据库和业务实体对象之间作一个映射，这样，我们在具体的操作业务对象的时候，就不需要再去和复杂的SQL语句打交道，只需简单的操作对象的属性和方法。

PS：ORM的优缺点
优点：没有复杂的sql操作，数据库结构简洁，迁移成本低（如从mysql->oracle）
缺点：性能较差、不适用于大型应用；复杂的SQL操作还需通过SQL语句实现

#### 功能
ORM可SQl语句一样可以操作表以及表数据，但是不能创建数据库

表：可以创建表、修改表、删除表（SQLAlchemy却无法修改表）
数据：增、删、改、查

### Model概念
我们说的Model就是Django自带的ORM模型，它包含了你存储数据的字段和行为，每个模型对应数据库中的一张表。

基础：

* 每个模型都需要继承django.db.models.Model
* 模型中的每个属性都对应数据库中的一个字段

### 简单示例
#### 连接mysql配置
基础：

* Django的ORM默认使用的数据库是sqllite3，我们要使用mysql就需要利用第三方工具连接数据库
* Django默认连接mysql的第三方工具是MySQLDB，但是python3不再支持MySQLDB
* 所以得修改django默认连接mysql方式为pymysql

1.创建mysql数据库
2.修改settings.py中数据库配置
	
	DATABASES = {
	    'default': {
	        'ENGINE': 'django.db.backends.mysql',
	        'NAME': 'dbname',
	        'USER': 'root',
	        'PASSWORD': '',
	        'HOST': 'localhost',
	        'PORT': 3306,
	    }
	}
3.向项目同名文件下的__init__.py文件添加
	
	- pymsql替换内部的myqldb，修改django默认连接mysql第三方工具
		import pymysql
		pymysql.install_as_MySQLdb()

#### 定义模型
定义UesrInfo类，有4个属性（app01/models.py文件）

	from diango.db import models
	
	class UserInfo(models.Model):
	    uid = models.BigAutoField(primary_key=True)  # int自增主键，默认不写django自动增加
	    username = models.CharField(max_length=32)   #char（32）
	    password = models.CharField(max_length=64)
	    age = models.IntegerField(default=1)         # int，默认值为1，也可以null=True
	    ug = models.ForeignKey("UserGroup", null=True)  # 创建外键，指定外键表名，可以为空，
    
	class UserGroup(Models.Model):
		title = models.CharField(max_length=32)
		

#### 使用模型
让Django使用定义的模型，添加应用名称app01到配置文件中的INSTALLED_APPS(setting.py文件)

	INSTALLED_APPS = [
	    ...
	    'app01',
	]

运行命令
	
	python3 manage.py makemigrations  # 应用生成迁移脚本
	python3 manage.py migrate  # 映射到数据库
	
PS：每次修改表结构和表结构后通过命令更新即可

### 字段

字段由models类属性指定

#### 字段类型

模型中的每个字段都是Field子类的某个实例。Django根据字段类的类型确定以下信息：

* 数据库当中的列类型（如：Integer、Varchar）
* 渲染表单时使用的html标签（如：`<input type='text'>、<select>`）
* 最基本的验证需求，它被用在django的后台管理站点和自动生成的表单中

Django内置了数十种字段类型，并且你可以自定义符合要求的字段类型

##### 内置字段类型

###### 字符串

	CharField(Field)
		- 字符类型
		- 必须提供max_length参数， max_length表示字符长度
###### 字符串（Django Admin中使用数据类型的并且带有对应类型验证）
	
	EmailField(CharField)
		- email
	IPAddressField(Field)
		- ip
	URLField(CharField)
		- url
	SlugField(CharField)
		- url并提供验证支持 字母、数字、下划线、连接符（减号）
	UUIDField(Field)
		- 提供对UUID格式的验证
	FilePathField(Field)
		- file并提供读取文件夹下文件的功能
		- 参数：
		      path,                      文件夹路径
		      match=None,                正则匹配
		      recursive=False,           递归下面的文件夹
		      allow_files=True,          允许文件
		      allow_folders=False,       允许文件夹
	FileField(Field)
		- 字符串，路径保存在数据库，文件上传到指定目录
		- 参数：
		  upload_to = ""      上传文件的保存路径
		  storage = None      存储组件，默认django.core.files.storage.FileSystemStorage
	ImageField(FileField)
		- 字符串，路径保存在数据库，文件上传到指定目录
		- 参数：
		  upload_to = ""      上传文件的保存路径
		  storage = None      存储组件，默认django.core.files.storage.FileSystemStorage
		  width_field=None,   上传图片的高度保存的数据库字段名（字符串）
		  height_field=None   上传图片的宽度保存的数据库字段名（字符串）
	CommaSeparatedIntegerField(CharField)
		- 字符串类型，格式必须为逗号分割的数字
###### 时间

	DateTimeField(DateField)
		- 日期+时间格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]
	DateField(DateTimeCheckMixin, Field)
		- 日期格式      YYYY-MM-DD

		- 参数auto_now_add=True
		 	字段在实例第一次保存的时候会保存当前时间，不管你在这里是否对其赋值
###### 数字

	IntegerField()
		- 整数列(有符号的) -2147483648 ～ 2147483647
	SmallIntegerField(IntegerField)
		- 小整数 -32768 ～ 32767
	BigIntegerField(IntegerField)
		- 长整型(有符号的) -9223372036854775808 ～ 9223372036854775807
	FloatField(Field)
    	- 浮点型
	DecimalField(Field)
	   - 10进制小数
	   - 参数：
	       max_digits，小数总长度
	       decimal_places，小数位长度
###### 布尔

	BooleanField(Field)
		- 布尔值类型
	NullBooleanField(Field):
		- 可以为空的布尔值

###### 枚举（django）
枚举没有自己的字段，是通过利用其它字段实现枚举功能

	color_list = (
		(1,'黑色'),
		(2,'白色'),
		(3,'蓝色')
	)
	color = models.IntegerField(choices=color_list)

#### 字段选项

全部字段类型可用、可选用

	null = True 
		- 为空
	default = '123' 
		- 默认值
	primary_key = True 
		- 主键
	max_length = 12
		- 最大长度
	unique_for_date时间创建索引
	
	db_index = True 
		- 单列索引 
	unique = True 
		- 单列唯一索引 
	
	class Meta:
		# 联合唯一索引
		unique_together = ( 
			('email','ctime'),
		)
		# 联合索引
		index_together = ( 
			('email','ctime'),
		)
		
	 null                数据库中字段是否可以为空
    db_column           数据库中字段的列名
    db_tablespace
    default             数据库中字段的默认值
    primary_key         数据库中字段是否为主键
    db_index            数据库中字段是否可以建立索引
    unique              数据库中字段是否可以建立唯一索引
    unique_for_date     数据库中字段【日期】部分是否可以建立唯一索引
    unique_for_month    数据库中字段【月】部分是否可以建立唯一索引
    unique_for_year     数据库中字段【年】部分是否可以建立唯一索引

    verbose_name        Admin中显示的字段名称
    blank               Admin中是否允许用户输入为空
    editable            Admin中是否可以编辑
    help_text           Admin中该字段的提示信息
    choices             Admin中显示选择框的内容，用不变动的数据放在内存中从而避免跨表操作
                        如：gf = models.IntegerField(choices=[(0, '何穗'),(1, '大表姐'),],default=1)

    error_messages      自定义错误信息（字典类型），从而定制想要显示的错误信息；
                        字典健：null, blank, invalid, invalid_choice, unique, and unique_for_date
                        如：{'null': "不能为空.", 'invalid': '格式错误'}

    validators          自定义错误验证（列表类型），从而定制想要的验证规则
                        from django.core.validators import RegexValidator
                        from django.core.validators import EmailValidator,URLValidator,DecimalValidator,\
                        MaxLengthValidator,MinLengthValidator,MaxValueValidator,MinValueValidator
                        如：
                            test = models.CharField(
                                max_length=32,
                                error_messages={
                                    'c1': '优先错信息1',
                                    'c2': '优先错信息2',
                                    'c3': '优先错信息3',
                                },
                                validators=[
                                    RegexValidator(regex='root_\d+', message='错误了', code='c1'),
                                    RegexValidator(regex='root_112233\d+', message='又错误了', code='c2'),
                                    EmailValidator(message='又错误了', code='c3'), ]
                            )

#### 关系字段

Django有一系列字段定义表与表的关系

* 一对多：models.ForeignKey(其他表)
* 多对多：models.ManyToManyField(其他表)
* 一对一：models.OneToOneField(其他表)

##### ForeignKey

多对一关系。

1.需要一个位置参数：与该模型相关的类。

```py
from django.db import models

class Car(models.Model):
    manufacturer = models.ForeignKey('Manufacturer')
    # ...

class Manufacturer(models.Model):
    # ...
    pass
```

2.对象与自己具有多对一关系：单表自关联

```py
# models.py文件
class U2U(models.Model): # 关系表
    b = models.ForeignKey('UserInfo', related_name='boys')
    g = models.ForeignKey('UserInfo', related_name='girls')
    # b = models.ForeignKey('UserInfo', related_query_name='boys')
    # g = models.ForeignKey('UserInfo', related_query_name='girls')
```
其中，要用到反向查找别名替换,如何让设置别名？这里涉及知识点

	related_query_name = 'b'
	related_name = 'a'
这样从关系表中查询UserInfo表中的信息时，使用别名即可

	obj.a_set.all()  # 通过related_query_name起名
	obj.a.all()  # 通过related_name起名
以上，通过ForeignKey + 关系表实现了 自关联

查询到与id为1的男生有关系的女生nickname

```py
    obj = models.UserInfo.objects.filter(id=1).first()  # 找到id为1的UserInfo对象obj
    result = obj.boys.all()  # 与对象obj有关的所有信息:Querset列表[U2Ud对象]
    for i in result:
        print(i.g.nickname)  # 从U2U对象正向查找g的nickname
```

##### ManyToManyField

多对多关联。

1.要求一个关键字参数：与该模型关联的类

```py
from django.db import models

class Car(models.Model):
    manufacturer = models.ManyToManyField('Manufacturer')
    # ...

class Manufacturer(models.Model):
    # ...
    pass
```

2.与自身进行关联的ManyToManyField

通过在Boy或Girl表中写 ManyToManyField(另一张表名)
	
	class Boy(models.Model):
	    name = models.CharField(max_length=32)
		 m = models.ManyToManyField('Girl')
		 
这样django会自动生成两表的关系表boy_m,但是boy_m表中只有三个字段：id、boy_id、girl_id，因为关系表boy_m是自动生成，没有对应类，所以不能直接通过获取关系表对象操作，那么要如何操作呢？

3.ManyToManyField 操作

对关系表boy_m的操作需要先获取一个Boy对象或者Girl对象

	- 获取对象 
		obj = models.Boy.objects.filter(name='alex').first()
	- 增加 
		obj.m.add(2)  # 给alex增加girl_id=2的数据
	- 批量增加
		obj.m.add(1,2,...)
	- 列表增加
		l = [1,2,...]
		obj.m.add(*l)
		
	- 删除
		obj.m.remove(1)  # 给alex删除girl_id=1这条数据
	- 批量删除，同添加
		obj.m.remove(1,2,...)
	- 列表删除，同添加
		l = [1,2,...]
		obj.m.remove(*l)
		
	- 重制 
		obj.m.set([1,2,...])  # 将原本的关系全部删除，插入心的关系
		
	- 查询 
		girl_list = obj.m.all()  # 查询到的是 QuerySet列，其中是Girl object
	- 条件查询
		girl_list = obj.m.filter(nick='alice')
		
	- 清空
		obj.m.clear()  # 清空与alex有关全部数据

那么对于Girl对象如何操作关系表呢？通过 小写表名_set实现

	- 获取对象 
		obj = models.Girl.objects.filter(nick='alice').first()
	- 查询
		boy_list = obj.boy_set.all()


##### OneToOneField

一对一关联关系。

最主要的用途是作为扩展自另外一个模型的主键；例如，多表继承就是通过对子模型添加一个隐式的一对一关联关系到父模型实现的。


#### 自定义字段类型

使用内部的class Meta 定义模型的元数据，例如：

	from django.db import models
	
	class Ox(models.Model):
	    horn_length = models.IntegerField()
	
	    class Meta:
	        ordering = ["horn_length"]
	        verbose_name_plural = "oxen"

模型元数据是“任何不是字段的数据”，比如排序选项（ordering），数据库表名（db_table）或者人类可读的单复数名称（verbose_name 和verbose_name_plural）。在模型中添加class Meta是完全可选的，所有选项都不是必须的。


#### 唯一索引

多对多以及如何建立联合唯一索引

关系表Love中写Class Meta 元选择

	Class Love(models.Model):
		b = models.ForeignKey('Boy')
		
		# b = models.ForeignKey(to='Boy',to_field='id') #关联哪个表哪个字段
		g = models.ForeignKey('Girl')
		
		Class Meta:  # 建立唯一索引
			unique_together = [
				('b','g'),
			]

#### ManyToManyField + ForeignKey
ManyToManyField中需要though和through_fields，但是方式二中的用法只有查询和清空有效

```pyclass Boy(models.Model):	id=models.AutoField(primary_key=True)	name=models.CharField(max_length=32)	m=models.ManyToManyField("Gril")	class Girl(models.Model):	id=models.AutoField(primary_key=True)	name=models.CharField(max_length=32)	class Love(models.Model):	b=models.ForeignKey("Boy")	g=models.ForeignKey("Gril")
	
	class Meta:
	unique_together = [
		('b','g'),
	]```这样会产生4个表 boy表、gril表、boy_m表即boy和gril的关系表  和 love表但是这时候就不能用m来进行add和remove来进行操作了，但是可以用clear这就需要建立m的时候这样写：

```pym = models.ManyToManyField('Girl',to="Boy",through="Love",through_fields=('b','g',))through：表示从哪个表中找through_fields：表示找哪个字段```这样在生成表的时候生成的就是3个表 boy表、gril表 和 love表		查询可以通过m快速查询，例如拿上述问题举例：```py1、	res=models.Boy.objects.filter(name="A").first()	g_list=res.love_set.all()	for i in g_list:		print(i.g.name)		2、 	obj = models.Boy.objects.filter(name='A').first()   g_list = obj.m.all()   拿的是gril的对象	for i in g_list:		print(i.name)
```

#### 递归关联（单表自关联）
数据库中一个表中可以根据id两两关联
例如，有UserInfo表，让其中id进行关联，如何实现自关联？

##### 通过ManyToManyField实现

```py
#models.py
class UserInfo(models.Model):
	...
    m = models.ManyToManyField('UserInfo')
```
创建后会生成关系表，其中字段：id、from_userinfo_id、to_userinfo_id,其中，from_userinfo_id存放男生id，to_userinfo_id存放女生id

查询与id=1的男生有关系的女生姓名

```py
    obj = models.UserInfo.objects.filter(id=1).first()
    result = obj.m.all()
    for i in result:
        print(i.nickname)
   
```
其中，obj.m是将obj作为from_userinfo_id，查询全部的to_userinfo_id

查询到与id为4的女生有关系的男生nickname

```py
    obj = models.UserInfo.objects.filter(id=4).first()
    result = obj.userinfo_set.all()
    for i in result:
        print(i.nickname)
```
其中，userinfo_set是将obj作为to_userinfo_id，查询全部的from_userinfo_id

##### FK自关联 （应用评论信息存储表）
应用于评论信息存储表

| id | news_id | conent | user | reply_id |
| --- | --- | --- | --- | --- |
| 1 | 1 | 什么鬼 | alex | null |
| 2 | 1 | 什么什么鬼 | egon | 1 |
| 3 | 1 | sb | eric | 2 |

```py
#models.py
class Comment(models.Model):
    news_id = models.IntegerField()
    conent = models.CharField(max_length=100)
    user = models.CharField(max_length=32)
    reply_id = models.ForeignKey('Comment', null=True, blank=True)
```

### 操作表（表查询）

一旦你建立好数据模型，Django 会自动为你生成一套数据库抽象的API，可以让你创建、检索、更新和删除对象。

#### 1.基本操作
views.py

```python
def sql(request):
    from app01 import models

    # 增加数据
    models.UserGroup.objects.create(title="销售部")
    models.UserInfo.objects.create(username='root', password='123', age="20", ug_id=1)

    # 查询
    user_list = models.UserInfo.objects.all()  # 拿到QuerySet列表，其中存储的对像（一行就是一个对象）
    #models.UserInfo.objects.all().first()  # 取到QuerySet中的第一个对象
    group_list = models.UserGroup.objects.filter(id=1, title="销售部")  # 条件查询
    group_list = models.UserGroup.objects.filter(id__gt=1)  # id>1（_lt小于）
    print(user_list, group_list)
    for i in user_list:  # 显示字段
        print(i.username, i.age)

    # 删除
    models.UserInfo.objects.filter(id=2).delete()  # 删除

    # 更新
    models.UserInfo.objects.filter(id=1).update(title="人事部")  # 更新

    return HttpResponse("...")
```	

#### 2.连表操作
##### 一对多
对象如何实现连表查询？
答：通过外键实现

通过操作表数据的介绍可知，一般查询方法有all(),filter(),如此这般查询到的结果是QuerySet列表，列表中存放的是对象（一行数据就是一个对象）

通过values()查询到的结果是QuerySet列表，列表中存放的是字典

通过values_list()查询到的结果是QuerySet列表，列表中存放的是元组


###### 对象

**正向操作**
本表中有外键的，直接通过 <font color='red'>外键列名.字段</font> ( ut.id ) 即可
	
    result = models.UserInfo.objects.all()               # 获取UserInfo表中全部QuerySet列表
    for i in result:
        print(i.id, i.name, i.age, i.ut_id, i.ut.title)  # 外键ut.title
	        
**反向操作** 
本表主键是其它表的外键，直接通过 <font color='red'>对象.小写被外键的表名_set.all()</font> ( obj.user_set.all() )即可 
	
    obj = models.UserType.objects.filter(title='物业部').first()  # 获取UserType表中名为物业部的QuerySet对象
    user_list = obj.userinfo_set.all()                           # 小写被外键的表名_set.all()
    
    for row in user_list:  
        print(row.id, row.name)

PS
表名_set可以理解为把外键表作为queryset对象提取

###### 字典

字典可以是直接通过values()得到，也可以是all().values()或者filter().values()得到

**正向操作**
获取字典 <font color='red'>外键列名.字段</font> ( ut__id )
		
	result = models.UserInfo.objects.all().values('id', 'name', 'ut__title')
	或者
	result = models.UserInfo.objects.values('id', 'name', 'ut__title')
	
	print(result)
    for obj in result:
        print(obj['id'], obj['name'], obj['ut__title'])
		        
其中，ut__title通过外键加双下划线列名，可以实现字典跨表查询
	
**反向操作**
想要通过字典反向跨表操作，通过 <font color='red'>小写表名__列名</font> 即可(只写小写表名为id)
	
	result = models.UserType.objects.all().values('userinfo__id','userinfo__username')
	等于
	result = models.UserType.objects.values('userinfo__id','userinfo__username')
	或者 可以+筛选
	result = models.UserType.objects.filter(title='物业部').values('userinfo__id','userinfo__username')
	
    for row in result:
        print(row['userinfo__id'], row['userinfo__username'])
	        
###### 元组

元组可以是直接通过values_list()得到，也可以是all().values_list()或者filter().values_list()得到
 
**正向操作**
获取元组 <font color='red'>外键列名__字段</font> （ut__title）
	
	obj = models.UserInfo.objects.all().values_list('id','name','ut__title') 
	或者
	obj = models.UserInfo.objects.values_list('id','name','ut__title') 
			
	for row in obj:
		print(row[0], row[1], row[3])
其中，ut__title通过外键加双下划线列名，可以实现元组的跨表查询
	
**反向操作**
想要通过元组反向跨表操作，通过 <font color='red'>小写表名__列名</font> 即可(只写小写表名为id)
	

    result = models.UserType.objects.values_list("id", "title", "userinfo", "userinfo__age", "userinfo__name")  
    
    for obj in result:
        print(obj[0], obj[1], obj[2], obj[3], obj[4])
	        
PS:

* filter().values()或者filter().values_list()可以增加筛选条件，这样将筛选结果再跨表操作
* 正向反向都是left join效果	

##### 多对多

已知：表Boy、表Girl、关系表Love
要求：查询到与alex有关系的girl

###### 一般查询
有四种查询方式

```py
# 方式一：从Boy表获取alex对象，反向操作Love表得到全部与alex有关系list，循环list正向取得girl名字，查询2n+2次
obj = models.Boy.objects.filter(name="alex").first() # 查询1次
    love_list = obj.love_set.all() # 查询1次
    for i in love_list: # 查询 2n次
        print(i.g.nick)
        
# 方式二：Love表字典正向操作取得有alex的list，循环list正向取得girl名字，查询2n+1次
    love_list = models.Love.objects.filter(b__name="alex") # 查询 1次
    for obj in love_list: # 查询 2n次
        print(obj.g.nick)

# 方式三：Love表left join Girl表正向操作取的有alex的字典list，循环list取得girl名字，查询n+1次
    love_list = models.Love.objects.filter(b__name='alex').values("g__nick") # 查询1次
    for item in love_list: # 查询n次
        print(item["g__nick"])
        
# 方式四：Love表inner join Girl表正向操作取的有alex的字典list，循环list取得girl名字，查询n+1次
    obj = models.Love.objects.filter(b__name='alex').select_related("g") # 查询 1次
    for i in obj: # 查询n次
        print(i.g.nick)
```
PS:推荐后两种，查询效率稍高


#### 3.进阶操作

##### 获取行数 .count()
	
	models.UserInfo.objects.all().count()
##### 比较 __gt
	
    models.UserInfo.objects.filter(id__gt=1)  # id>1
    models.UserInfo.objects.filter(id__gte=1)  # id>=1
    models.UserInfo.objects.filter(id__lt=2)  # id<2
    models.UserInfo.objects.filter(id__lte=2)  # id<=2
    models.UserInfo.objects.filter(id__gte=1, id__lt=2)  # id>=1,id<2
##### in 和 not in

    models.UserInfo.objects.filter(id__in=[1, 2, 4]) # 获取id在1，2，4中数据
    models.UserInfo.objects.exclude(id__in=[1, 2, 4]) # 获取除id在1，2，4中的数据
##### isnull 为空
查询字段为空的数据 ... where ut_id IS NULL
	
	models.UserInfo.objects.filter(ut__isnull=True)
##### contains 字段内容包含／不包含
字段是否包含内容 ... where name like '%a%' /... where not (name like '%a%' )

    models.UserInfo.objects.filter(name__contains="a")
    models.UserInfo.objects.filter(name__icontains="a")  # 大小写不敏感
    models.UserInfo.objects.exclude(name__icontains="a") 
##### range 范围
bettwen ... and ...
	
	models.UserInfo.objects.filter(id_range=[1, 3])

##### startswith/istartswith/endswith/iendswith
... where name like 'a%'

	models.UserInfo.objects.filter(name__startswith='a')
	models.UserInfo.objects.filter(name__istartswith='a')

##### 排序 order_by()
	
	- 筛选结果按照id ASC排序
		models.UserInfo.objects.filter(id__gt=1).order_by("id")
	- DESC排序
		models.UserInfo.objects.filter(id__gt=1).order_by("-id")
PS：order_by("id","name") 先按照id再按照name ASC排序
		
##### 分组 annotate()
	
	- import聚合函数
		from django.db.models import Count, Sum, Max, Min, Avg
	- annotate()必须配合values(),values()中是要分组的列
		models.UserInfo.objects.values("ut_id").annotate(x=Count('id'))
		对应sql：
		"SELECT ut_id， COUNT(id) AS x FROM UserInfo GROUP BY ut_id"
PS：分组后结果再filter()，类似于group by后的having操作
	
	models.UserInfo.objects.values("ut_id").annotate(x=Count('id')).filter(ut_id__gt=1)
	
##### select_related()
查询时，主动做连表操作，类似于inner join操作

	- select_related(要连接表的外键)
		q = models.UserInfo.objects.all().select_related('ut')
	- 执行原理
		" select * from UserInfo inner join UserType on UserInfo.ut = UserType.id "
	
	- 取值
		for row in q:
			print(row.name,row.ut.title)
	
##### prefetch_related()
查询时，不做连表，作多次单表查询，实现连表操作目的

	- prefetch_related(外键)
		q = models.UserInfo.objects.all().perfetch_related('ut')
	- 执行原理
		" select * from UserInfo "
		获取UserInfo全部ut组成列表[1,2,...]
		" select * from UserType where id in [1,2,...] "
		
	- 取值
		for row in q:
			print(row.id,row.ut.title)
		
PS：单表查询比连表操作查询性能高
	
	
##### sql语句 .query
通过.query 可以查看sql语句
	
	result = models.UserInfo.objects.all()
	print(result.query)
##### all()
	获取所有的数据对象
##### filter()
	 条件查询
	 条件可以是：参数，字典，Q
##### exclude()
	条件查询
	条件可以是：参数，字典，Q
##### annotate()
	实现聚合group by 查询
##### distinct()
	去重复
	models.UserInfo.objects.values('nid').distinct()
##### order_by()
	排序
##### extra（）
	构造额外查询条件或者映射
##### reverse()
	倒序
	models.UserInfo.objects.all().order_by('-id').reverse()
PS:如果存在order_by，reverse则是倒序，如果多个排序则一一倒序
##### defer()
	映射中排除某些列
	models.UserInfo.objects.defer('name','id')
	或者
	models.UserInfo.objects.filter(...).defer('name','id')
##### only()
	仅取某个表中的数据
	models.UserInfo.objects.only('name','id')
	或
	models.UserInfo.objects.filter(...).only('name','id')
PS:使用only效率比all高，但是使用only也可以点出未包括在only中的列，但是效率会比all第，所以，only哪些列就使用哪些列
##### using()
	制定使用的数据库，参数为别名，setting.py中设置的
##### raw()
	执行原生sql
	models.UserInfo.objects.raw('select * from userinfo')
	如果SQL是其他表时，必须将名字设置为当前UserInfo对象的主键列名
	models.UserInfo.objects.raw('select pid as id from 其他表')
	为原生SQL设置参数
	models.UserInfo.objects.raw('select id as nid from userinfo where nid>%s', params=[12,])
	将获取的到列名转换为指定列名
	name_map = {'first': 'first_name', 'last': 'last_name', 'bd': 'birth_date', 'pk': 'id'}
	Person.objects.raw('SELECT * FROM some_other_table', translations=name_map)
	指定数据库
	models.UserInfo.objects.raw('select * from userinfo', using="default")
##### values()
	获取每行数据为字典格式

##### values_list()
	获取每行数据为元组
##### dates()
	
	def dates(self, field_name, kind, order='ASC'):
    # 根据时间进行某一部分进行去重查找并截取指定内容
    # kind只能是："year"（年）, "month"（年-月）, "day"（年-月-日）
    # order只能是："ASC"  "DESC"
    # 并获取转换后的时间
        - year : 年-01-01
        - month: 年-月-01
        - day  : 年-月-日

    models.DatePlus.objects.dates('ctime','day','DESC')
#####  datetimes()

	def datetimes(self, field_name, kind, order='ASC', tzinfo=None):
    # 根据时间进行某一部分进行去重查找并截取指定内容，将时间转换为指定时区时间
    # kind只能是 "year", "month", "day", "hour", "minute", "second"
    # order只能是："ASC"  "DESC"
    # tzinfo时区对象
    models.UserInfo.objects.datetimes('ctime','hour',tzinfo=pytz.UTC)
    models.UserInfo.objects.datetimes('ctime','hour',tzinfo=pytz.timezone('Asia/Shanghai'))

    """
    pip3 install pytz
    import pytz
    pytz.all_timezones
    pytz.timezone(‘Asia/Shanghai’)
    """
    
##### none()

	def none(self):
    # 空QuerySet对象
##### aggregate()
	def aggregate(self, *args, **kwargs):
	   # 聚合函数，获取字典类型聚合结果
	   from django.db.models import Count, Avg, Max, Min, Sum
	   result = models.UserInfo.objects.aggregate(k=Count('u_id', distinct=True), n=Count('nid'))
	   ===> {'k': 3, 'n': 4}
##### count()
	def count(self):
	   # 获取个数
##### get()
	def get(self, *args, **kwargs):
	   # 获取单个对象
##### create()
	def create(self, **kwargs):
	   # 创建对象
##### bulk_create()
	def bulk_create(self, objs, batch_size=None):
	    # 批量插入
	    # batch_size表示一次插入的个数
	    objs = [
	        models.DDD(name='r11'),
	        models.DDD(name='r22')
	    ]
	    models.DDD.objects.bulk_create(objs, 10)
##### get_or_create()
	def get_or_create(self, defaults=None, **kwargs):
	    # 如果存在，则获取，否则，创建
	    # defaults 指定创建时，其他字段的值
	    obj, created = models.UserInfo.objects.get_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 2})
##### update_or_create()
	def update_or_create(self, defaults=None, **kwargs):
	    # 如果存在，则更新，否则，创建
	    # defaults 指定创建时或更新时的其他字段
	    obj, created = models.UserInfo.objects.update_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 1})
##### first()
	def first(self):
	   # 获取第一个
##### last()
	def last(self):
	   # 获取最后一个
##### in_bulk
	def in_bulk(self, id_list=None):
	   # 根据主键ID进行查找
	   id_list = [11,21,31]
	   models.DDD.objects.in_bulk(id_list)
##### delete()
	def delete(self):
	   # 删除
##### update()
	def update(self, **kwargs):
	    # 更新
##### exists()
	def exists(self):
	   # 是否有结果


#### 4.其他操作

##### F 
获取数据表中列字段进行操作

	- import F
		from django.db.models import F
	- F("age")+1 age列自加1，更新时取到原来的值
		v = models.UserInfo.objects.update(age=F('age') + 1)
		
PS:v取到被修改的行数
		
##### Q 字典传参数 (组合条件查询)
用于构造复杂的查询条件，两种用法，对象和方法

	- import Q
		from django.db.models import Q

**对象**
多用于一般条件查询，类似filter,所以不常用

	- Q对象，表示一个条件
	models.UserInfo.objects.filter(Q(id=1)) 
	- 多条件查询
		models.UserInfo.objects.filter(Q(id=1) | Q(id=2))

**方法** 
多用于组合条件查询，动态查询

同条件进行or，不同条件进行and，循环拼接查询条件

```py
    q1 = Q()
    q1.connector = "OR"
    q1.children.append(("id", 1))
    q1.children.append(("id", 2))
    q1.children.append(("id", 3))

    q2 = Q()
    q2.connector = "OR"
    q2.children.append(("name", "alex"))
    q2.children.append(("name", "tom"))
    q2.children.append(("name", "peter"))

    con = Q()
    con.add(q1, "AND")
    con.add(q2, "AND")

    result = models.UserInfo.objects.filter(con)
    print(result)
    for i in result:
        print(i.name, i.ut.title)
```
类似查询:
 
	select * from UserInfo where  (id=1 or id=2 or id =3)AND (name="alex" or name ="tom" or name = "peter")
	
##### extra 额外查询条件
在model基础上进行额外的查询条件以及相关表、排序

extra的参数有select,select_params,where,params,tables,order_by

	- select可以作为临时表的查询结果放在select映射部分，其中x为别名，select_params配合select使用
		models.UserInfo.objects.extra(select={"x": "select count(id) from app01_usertype where id >%s"},select_params=[1, ])
	- where可以作为where条件部分，params配合where使用
		models.UserInfo.objects.extra(where=['id>%s'], params=[1, ])
	- tables可以作为原表连接的新表，组成笛卡尔积
		models.UserInfo.objects.extra(tables=['app01_usertype'])
	- order_by可以对原表排序
		models.UserInfo.objects.extra(order_by=['-id'])

##### 原生sql语句
django原生sql已帮我们连接数据库，获取游标

	- 导入connection利用现成链接
		from django.db import connection, connections
	- 通过连接获取游标
		cursor = connection.cursor()  # 默认连接setting.py中key值为default DATABASES
	- 或者，可以选择其它数据库
		# cursor = connections['default'].cursor()  # default是选择setting.py中的DATABASES，可以根据需要变换数据库
	- sql语句
		cursor.execute("select * from app01_userinfo where id = %s", [1, ])
	- fetchone或者fetchall
    v = cursor.fetchone()



 
## Form组件

Django中Form组件主要功能：

* 生成HTML标签
* 验证用户数据（显示错误信息）
* HTML Form提交保留上次提交数据
* 初始化页面显示内容


### Form组件简单示例

1.创建Form类(确定字段约束)

```py
from django.forms import Form,fields

class LoginForm(Form):

	 username = fields.CharField(  # html标签的name属性 就是 Form类字段名
        max_length=18,
        min_length=6,
        required=True,
        error_messages={
            'required': '用户名不能为空',
            'min_length': '太短了',
            'max_length': '太长了',
        }
    )
```	
其中，
字段不可为空: required=True
字段长度: min_length～max_length
错误信息：error_messages

2.view函数(使用提交的数据)
			
```py	
# 获取POST对象
obj = LoginForm(request.POST)  

# 是否校验成功 True或False
v = obj.is_valid()  
	
# 所有错误信息 , 对象
obj.errors

# 获取字典格式数据，通过**dic可以直接操作数据表的增改查询
obj.cleaned_data
```

3.html（前端生成标签）

```html
{{ obj.username }} {{ obj.errors.username.0 }}
```

PS：

1.cleaned_data字典
对数据表中的某行数据的增、改、查操作可以使用字典作为传入参数，精简代码，所以对数据表的操作可以直接将obj.cleaned_data字典数据当作参数传入

		dic = {'name':'alex','age':12,...}
		
	- 增加
		models.Table.objects.create(**dic)
	- 更新
		models.Table.objects.filter(**dic)
	- 查询 查询条件间是and关系
		models.Table.objects.filter(**dic)

2.is_valid()原理

	1.实例化 LoginForm
	
	self.fields{
		'user':正则表达式，
		'pwd':正则表达式，
		...
	}
	
	2.循环 self.fields
	
	flag = True
	for k, v self.fields.items():  # k:user,v:正则表达式
		if request.POST.get(k)不符合v的正则表达式:
			flag = False
	return flag


### Form类

#### django内置字段

	Field
	    required=True,               是否允许为空
	    widget=None,                 HTML插件
	    label=None,                  用于生成Label标签或显示内容
	    initial=None,                初始值
	    help_text='',                帮助信息(在标签旁边显示)
	    error_messages=None,         错误信息 {'required': '不能为空', 'invalid': '格式错误'}
	    show_hidden_initial=False,   是否在当前插件后面再加一个隐藏的且具有默认值的插件（可用于检验两次输入是否一直）
	    validators=[],               自定义验证规则
	    localize=False,              是否支持本地化
	    disabled=False,              是否可以编辑
	    label_suffix=None            Label内容后缀
	 
	 
	CharField(Field)
	    max_length=None,             最大长度
	    min_length=None,             最小长度
	    strip=True                   是否移除用户输入空白
	 
	IntegerField(Field)
	    max_value=None,              最大值
	    min_value=None,              最小值
	 
	FloatField(IntegerField)
	    ...
	
	RegexField(CharField)
		regex,                      自定制正则表达式
		max_length=None,            最大长度
		min_length=None,            最小长度
		error_message=None,         忽略，错误信息使用 error_messages={'invalid': '...'}
	 
	DecimalField(IntegerField)
	    max_value=None,              最大值
	    min_value=None,              最小值
	    max_digits=None,             总长度
	    decimal_places=None,         小数位长度
	 
	BaseTemporalField(Field)
	    input_formats=None          时间格式化   
	 
	DateField(BaseTemporalField)    格式：2015-09-01
	TimeField(BaseTemporalField)    格式：11:12
	DateTimeField(BaseTemporalField)格式：2015-09-01 11:12
	 
	DurationField(Field)            时间间隔：%d %H:%M:%S.%f
	    ...
	 
	EmailField(CharField)      
	    ...
	 
	FileField(Field)
	    allow_empty_file=False     是否允许空文件
	 
	ImageField(FileField)      
	    ...
	    注：需要PIL模块，pip3 install Pillow
	    以上两个字典使用时，需要注意两点：
	        - form表单中 enctype="multipart/form-data"
	        - view函数中 obj = MyForm(request.POST, request.FILES)
	 
	URLField(Field)
	    ...
	 
	 
	BooleanField(Field)  
	    ...
	 
	NullBooleanField(BooleanField)
	    ...
	 
	ChoiceField(Field)
	    ...
	    choices=(),                选项，如：choices = ((0,'上海'),(1,'北京'),)
	    required=True,             是否必填
	    widget=None,               插件，默认select插件
	    label=None,                Label内容
	    initial=None,              初始值
	    help_text='',              帮助提示
	 
	 
	ModelChoiceField(ChoiceField)
	    ...                        django.forms.models.ModelChoiceField
	    queryset,                  # 查询数据库中的数据
	    empty_label="---------",   # 默认空显示内容
	    to_field_name=None,        # HTML中value的值对应的字段
	    limit_choices_to=None      # ModelForm中对queryset二次筛选
	     
	ModelMultipleChoiceField(ModelChoiceField)
	    ...                        django.forms.models.ModelMultipleChoiceField
	 
	 
	     
	TypedChoiceField(ChoiceField)
	    coerce = lambda val: val   对选中的值进行一次转换
	    empty_value= ''            空值的默认值
	 
	MultipleChoiceField(ChoiceField)
	    ...
	 
	TypedMultipleChoiceField(MultipleChoiceField)
	    coerce = lambda val: val   对选中的每一个值进行一次转换
	    empty_value= ''            空值的默认值
	 
	ComboField(Field)
	    fields=()                  使用多个验证，如下：即验证最大长度20，又验证邮箱格式
	                               fields.ComboField(fields=[fields.CharField(max_length=20), fields.EmailField(),])
	 
	MultiValueField(Field)
	    PS: 抽象类，子类中可以实现聚合多个字典去匹配一个值，要配合MultiWidget使用
	 
	SplitDateTimeField(MultiValueField)
	    input_date_formats=None,   格式列表：['%Y--%m--%d', '%m%d/%Y', '%m/%d/%y']
	    input_time_formats=None    格式列表：['%H:%M:%S', '%H:%M:%S.%f', '%H:%M']
	 
	FilePathField(ChoiceField)     文件选项，目录下文件显示在页面中
	    path,                      文件夹路径
	    match=None,                正则匹配
	    recursive=False,           递归下面的文件夹
	    allow_files=True,          允许文件
	    allow_folders=False,       允许文件夹
	    required=True,
	    widget=None,
	    label=None,
	    initial=None,
	    help_text=''
	 
	GenericIPAddressField
	    protocol='both',           both,ipv4,ipv6支持的IP格式
	    unpack_ipv4=False          解析ipv4地址，如果是::ffff:192.0.2.1时候，可解析为192.0.2.1， PS：protocol必须为both才能启用
	 
	SlugField(CharField)           数字，字母，下划线，减号（连字符）
	    ...
	 
	UUIDField(CharField)           uuid类型
	    ...

#### 补充

检测内置字段本身错误信息，例如，EmailField检测的就是字段不符合邮箱地址的格式

	fields.CharField(error_messages={'inalid','格式错误'})

针对IntegerField字段，字段范围

	number = fields.IntegerField(max_value=100,min_value=15)

	
生成标签有关字段

	t1 = fields.CharField(
		label='用户名',  # 标签的label命名
		lable_suffix=':'  # label内容后缀
		help_text=‘请入电话号码’,  # 标签帮助信息
		initital='username',  # input输入框显示默认值
		disabled=True，  # 输入框是否为可以编辑
		
		import django.forms import widgets
		widget=widgets.Select,  # 选择字段标签为select，默认是input
	)  
	
	html 调用
	
	{{ obj.t1.label }}{{ obj.t1.lable_suffix }}
	{{ obj.t1 }}{{ obj.t1.help_text }}
	
	其实，html 调用 只需要写如下就可以了
	
	{{ obj.as_p }}
	
PS：

有些浏览器会自动帮我们增加校验，却掉浏览器的校验给标签加 novalidate 属性

	<form action='...' method='...' movalidate >...</form>
	

### Form生成HTML标签、验证、不刷新、初始值全套

为了实现form提交不会刷新页面，我们之前使用ajax方法，同样的我们可以通过使用Django的form组件实现

#### ajax提交示例
通过ajax不刷新页面验证form，并且保留上次输入内容

views.py

```py
def ajax_login(request):
    import json
    ret = {'status': True, 'message': None}
    obj = loginForm(request.POST)
    if obj.is_valid():
        print(obj.cleaned_data)
    else:
        ret['status'] = False
        ret['message'] = obj.errors
    return HttpResponse(json.dumps(ret))
```
login.html 

```html
<script src="/static/jquery-3.2.1.js"></script>
<script>
    function ajaxForm() {
        $('.c1').remove();
        $.ajax({
            url: '{% url "ajax_login" %}',
            type: 'POST',
            data: $('#f1').serialize(),
            dataType: 'JSON',
            success: function (arg) {
                if (arg.status) {
                    location.href = '{% url index %}'
                } else {
                    $.each(arg.message, function (index, value) {
                        var tag = document.createElement('span');
                        tag.innerHTML = value[0];
                        tag.className = 'c1';
                        $('#f1').find("input[name='" + index + "']").after(tag)
                    })
                }
            }
        })
    }
</script>
```
login.html

```html
<form method="post" action="{% url 'login' %}" id="f1">
	...
	<a onclick="ajaxForm();" class="btn btn-default">ajax提交</a>
</form>
```

PS：

1.serialize()提交表单全部
ajax提交form表单时，可以将form表单内全部数据提交，使用serialize()方法

```html
data: $('#f1').serialize(),  # 其中<form id='f1'>
```
提交原理

	user=alex&pwd=123456&scrftoken=...

#### form提交示例
不刷新页面验证form，并且保留上次输入内容

1.创建form类

```py
from django.shortcuts import render, HttpResponse, redirect
from django.forms import Form, fields, widgets
from app01 import models

class loginfForm(Form):
    username = fields.CharField(
        label='用户名',
        widget=widgets.Input(attrs={'class': 'form-control'})
    )
    password = fields.CharField(
        label='密码',
        widget=widgets.Input(attrs={'class': 'form-control', 'type': 'password'})
    )
```
2.view函数处理

```py    
def loginf(request):
    if request.method == 'GET':
        obj = loginfForm()
        return render(request, 'loginf.html', {'obj': obj})
    else:
        obj = loginfForm(request.POST)
        if obj.is_valid():
            row = models.User.objects.filter(username=obj.cleaned_data['username'],
                                             password=obj.cleaned_data['password']).first()
            if row:
                return redirect('／index.html')
        return render(request, 'loginf.html', {'obj': obj})
```
3.loginf.html

```html
<form method="post" action="{% url 'loginf' %}" novalidate>
    {% csrf_token %}
    {{ obj.username.label }}{{ obj.username }} {{ obj.errors.username.0 }}
    {{ obj.password.label }} {{ obj.password }} {{ obj.errors.password.0 }}
    <input type="submit" value="登录">
</form>
```

PS：

单选：生成select字段,参数需是元组

	widget=widgets.Select(choices=[(1,'北京'),(2,'上海'),...])
	widget=widgets.Select(choices=models.City.objects.values_list('id','title'))
	
生成字段加css样式

	widget=widgets.TextInput(attrs={'class':'form-control'})



#### 初始值（data和initial）

生成html标签，并含有错误信息

	obj = funcForm(request.POST)
	省略了data，本质就是
	obj = funcForm(data = request.POST)
	
只生成html标签，并且可以附带默认值

	obj = funcForm({'username':'alex','cls_id':[1,2,3]})
	省略了initial,本质是
	obj = funcForm(initial = {'username':'alex','cls_id':[1,2,3]})
	
	
#### Form验证

1.首先，字段验证
2.然后，单独字段方法 clean__参数
3.最后，clean 方法

##### 1正则校验

a.字段本身正则
	
自定义正则表达式的字段有两种方式

	- 1.自定义验证规则，作为内置字段正则的补充
		fields.CharField( validators=[] ）               
	
	- 2.自定义正则字段
		fields.RegexField('010\d+')  # 010开头的数字
	
b.额外的正则

	from django.forms import Form,widgets,fields
	from django.core.validators import RegexValidator
	 
	class MyForm(Form):
		user = fields.CharField(
			validators=[RegexValidator(r'^[0-9]+$', '请输入数字'), RegexValidator(r'^159[0-9]+$', '数字必须以159开头')],
		)

##### 2验证字段是否已存在在数据库中

关键字：clean_字段 和 clean


	from django.core.exceptions import ValidationError
	
	class funcForm(Form):
		user = fields.CharField()
		pwd = fields.CharField()
		email = fields.CharField()
		
		# 单个字段校验
		def clean_user(self):
			v = self.cleand_data['user']
			if models.Student.objects.filter(name=v).count():
				# 主动异常
				raise ValidationError('用户名已存在')
			return self.cleaned_data['user']  # 必须返回，否则obj.cleaned_data['user']就被清空了
		
		def clean_pwd(self):)
			return self.cleaned_data['pwd']  # 必须返回
			
		# 联合校验
		def clean(self):
			user = self.cleand_data['user']
			email = self.cleand_data['email']
			if models.Student.objects.filter(name=user,email=email).count():
				raise ValidationError('用户名和邮箱联合已经存在')
			return self.cleaned_data  # 必须返回，否则obj.cleaned_data就被清空了

#### 制作插件
##### 单选下拉菜单

之前使用的IntegerField

	cls_id = fields.IntegerField(
		# widget=widgets.Select(choices=[(1,'上海'),(2,'北京')])
		widget=widgets.Select(choices=models.Classes.objects.values_list('id','title'),attrs={'class': 'form-control'})
	)

使用ChoiceField

	cls_id = fields.ChoiceField(
		choices = models.Classes.objects.values_list('id','title'),
		widget = widgets.Select()
	)

##### 可多选下拉菜单

首先尝试使用字段CharField

	cls_id = fields.CharField(
		widget = widgets.SelectMultiple(choices = models.Classes.objects.values_list('id','title'))
	)
	这样得到的cls_id结果是
	{ 'cls_id': "['1','2']" }
	其中，列表是SelectMultiple结果，但是列表又变成了字符串，那是因为CharField的作用，所以不能使用CharField
	
所以真正的单选是这样的，使用ChoiceField，并且要把其中的choices参数提取出来
	
	cls_id = fields.MultipleChoiceField(
		choices = models.Classes.objects.values_list('id','title'),
		widget = widgets.SelectMultiple
	)
	这样得到的cls_id结果是
	{ 'cls_id': ['1','2']}

下拉列表中的数据在程序启动时加入内存，无法动态显示数据库中的变化，为了修复这个Bug，刷新无法动态显示数据库内容：

方式一：

	class TeacherForm(Form):
		tname = fields.CharField(min_length=2)
		xx = form_model.ModelMultipleChoiceField(queryset=models.Classes.objects.all())
		# xx = form_model.ModelChoiceField(queryset=models.Classes.objects.all())

方式二：

	class TeacherForm(Form):
		tname = fields.CharField(min_length=2)
		xx = fields.MultipleChoiceField(
			# choices = models.Classes.objects.values_list('id','title') # 这句可注释
			widget=widgets.SelectMultiple
		)
		def __init__(self,*args,**kwargs):
			super(TeacherForm,self).__init__(*args,**kwargs)
			self.fields['xx'].choices = models.Classes.objects.values_list('id','title')
			
编辑界面如何将信息加载到页面

	def edit_teacher(request, nid):
		if request.method == 'GET':
			row = models.Teacher.object.filter(id=nid).first()
			class_ids = row.c2t.values_list(id)
			id_list = list(zip(*class_ids))[0] if list(zip(*class_ids)) else []
			obj = TeacherForm(initial={'tname':row.tname,'xx':id_list})
			return render(request,'edit_teacher.html',{'obj':obj})

##### checkbox

一个

	cb = fields.CharField(
		widget = widgets.CheckboxInput
	)

多个

	cbs = fields.MultipleChoiceField(
		choices = [(1,'篮球'),(2,'足球'),(3,'排球')],
		widget = widgets.CheckboxSelectMultiple
	)
	
##### radio

	r = fields.ChoiceField(
		choices = [(1,'篮球'),(2,'足球'),(3,'排球')],
		widget = widgets.RadioSelect
	)		

##### 文件上传

f1.html
需要在form标签增加 enctype="multipart/form-data"

```html

	<form method="post" action="/f1/" enctype="multipart/form-data">
	    {% csrf_token %}
	    <input type="file" name="fafafa">
	    <input type="submit" value="提交">
	</form>
```

普通上传

	def f1(request):
	    if request.method == "GET":
	        return render(request, 'f1.html')
	    else:
	        import os
	        file_obj = request.FILES.get('fafafa')
	        f = open(os.path.join('static', file_obj.name), 'wb')
	        for chunk in file_obj.chunks():
	            f.write(chunk)
	        f.close()
	        return render(request, 'f1.html')

基于form实现文件上传

	class f2Form(Form):
	    fafafa = fields.FileField()
	
	
	def f2(request):
	    if request.method == "GET":
	        obj = f2Form()
	        return render(request, 'f2.html', {'obj': obj})
	    else:
	        import os
	        obj = f2Form(files=request.FILES)
	
	        if obj.is_valid():
	            print(obj.cleaned_data.get('fafafa').name)
	            print(obj.cleaned_data.get('fafafa').size)
	            f = open(os.path.join('static', obj.cleaned_data.get('fafafa').name), 'wb')
	            for chunk in obj.cleaned_data.get('fafafa').chunks():
	                f.write(chunk)
	            f.close()
	
	        return render(request, 'f2.html', {'obj': obj})

## XSS攻击

恶意web用户将代码植入到提供给其它用户使用的页面中

通过页面评论等功能，插入js代码，获取cookie等信息，叫做跨站脚本攻击(Cross Site Scripting)
	
	<script>alert(123)</script>
django默认阻止传入页面带有标签的字符串，想要浏览器不阻止传入字符串的解析需要标注safe关键字

	- 前端
		<div>{{ msg | safe }}</div>
	- 后台
		from django.utils.safestring import mark_safe
		msg = "<a href='http://www.sdf.com'>sdf</a>"
		safe_msg = mark_safe(msg)
		return render(request,'conmment.html',{'msg':safe_msg})
所以，前端不要轻易加safe

但是，非要加safe，又想要避免被xss攻击，可以在后台接收字段时，判断是否有标签关键字

	v = request.POST.get('content')
	if 'script' in v:
		return render(request,"conmment.html",{'error':'输入内容存在安全隐患'})
		
PS:

* 慎用 safe和mark_safe
* 必须得用的话，一定要过滤关键字


## 跨站请求伪造

### CSRF攻击
CSRF（Cross-site request forgery）跨站请求伪造，通过伪装来自受信任用户的请求来利用受信任的网站。

一个网站用户Bob可能正在浏览聊天论坛，而同时另一个用户Alice也在此论坛中，并且后者刚刚发布了一个具有Bob银行链接的图片消息。设想一下，Alice编写了一个在Bob的银行站点上进行取款的form提交的链接，并将此链接作为图片src。如果Bob的银行在cookie中保存他的授权信息，并且此cookie没有过期，那么当Bob的浏览器尝试装载图片时将提交这个取款form和他的cookie，这样在没经Bob同意的情况下便授权了这次事务。

主要利用受害者刚好也在操作银行页面，cookie没有过期，攻击者利用图片链接中的隐藏的form提交POST请求给银行，银行验证cookie没问题，会执行转账操作。

这样通过其它网站直接向银行发送请求的操作应该被禁止，所以在用户登录银行系统时，银行会给用户一个随机字符串，在接下来的POST请求中，必须要有这个随机字符串，用户才可以正常操作，否则，银行认为操作操作违法。

### 启用CSRF验证
Diango中带有自动生成字符串的功能来避免被CSRF攻击

	- 启用CSRF验证
		settings.py中MIDDLEWAR的'django.middleware.csrf.CsrfViewMiddleware'表示启用验证
	- form表单中增加
		{% csrf_token %}
	- 页面加载后会有隐藏的input标签
		<input type="hidden" name="csrfmiddlewaretoken" value="dguniTLGtMB3xCoq3YFuyTzBtgOaCjdYygEf5QvzTwGqQuAyetpnCdvh6o1bIqNm">
	- 浏览器中的cookie将生成
		![](media/14980919010345/14986489795952.jpg)
		
### 禁用CSRF验证
 
	- 全站禁用
	将settings.py中'django.middleware.csrf.CsrfViewMiddleware',注释

	- 局部禁用 FBV 给函数加装饰器
		from django.views.decorators.csrf import csrf_exempt
		@csrf_exempt
		def func(request):
        ...
        
	- 局部使用
	 	# 'django.middleware.csrf.CsrfViewMiddleware',  # 注释
			
		from django.views.decorators.csrf import csrf_protect
		@csrf_protect
		def func(request):
			...
	
	- 局部使用 CBV csrf_protect不可给类中的方法加装饰器
		from django.views import View
		from django.views.decorators.csrf import csrf_protect
		from django.utils.decorators import method_decorator
		
		# @method_decorator(csrf_protect,name = 'dispatch')  # 方法二：给全部方法被装饰
		# @method_decorator(csrf_protect,name = 'post')  # 方法三：制定给某个方法加装饰器
		@method_decorator(csrf_protect)  # 方法一：给类加，全部方法被装饰
		class Login(View):
		    def get(self, request):
		        return render(request, "login.html")
		        
		    def post(self, request):
		        print(request.POST.get("user"))
		        return HttpResponse("login.post")
		        
		        
#### PS:CBV 如何加装饰器

Django中CBV不可直接加装饰器，需要通过method_decorator(装饰器)

		from django.views import View
		from django.utils.decorators import method_decorator
		
		# @method_decorator(auth,name = 'dispatch')  # 方法二：给全部方法被装饰
		# @method_decorator(auth,name = 'fun2')  # 方法三：制定给某个方法加装饰器
		@method_decorator(auth)  # 方法一：给类加，全部方法被装饰
		class Login(View):
		    def func1(self, request):
		        ...
		        
			# @method_decorator(auth)  # 方法四：给方法加，仅该方法被装饰
		    def fun2(self, request):
		        ...
		        
### Ajax提交数据时候，携带CSRF

#### 放在data中携带
 如果网站中带有csrftoken，发送ajax应该将csrftoken带进去
 
```js
<script src="/static/jquery-3.2.1.js"></script>
<script>
   function submitForm() {
       {#从页面中获取csrf值#}
       var csrf = $('input[name="csrfmiddlewaretoken"]').val();
       $.ajax({
           url:"/login.html",
           type:"POST",
           data:{'csrfmiddlewaretoken':csrf},
           success:function(arg){
               console.log(arg)
           }
       })
   }
</script>
```
#### 放在请求头
从cookie中获取csrf值

获取cookie的插件：jquery.cookie.js
获取token：$.cookie('csrftoken')
加入请求头：headers:{'X-CSRFToken':token}

```js
<script src="/static/jquery-3.2.1.js"></script>
    <script src="/static/jquery.cookie.js"></script>
    <script>
        function submitForm() {
            var token = $.cookie('csrftoken');
            $.ajax({
                url:"/login.html",
                type:"POST",
                data:{},
                {#加入请求头#}
                headers:{'X-CSRFToken':token},
                success:function(arg){
                    console.log(arg)
                }
            })
        }
    </script>
```


## Cookie

1.保存在浏览器端的“键值对”
2.服务端可以向用户浏览器端写cookie
3.客户端每次请求，会携带coolie

	- 发送cookie
		obj = redirect("/index/")
		obj.set_cookie("ticket","1qazxsw23edc",max_age = 10)  
		return obj
		
	- set_cookie方法：
		max_age = 10  //超时时间
		path = '/'  //指定url才可以后去cookie
		domain = None  //域名等级划分，多点登录，统一认证登录
		httponly = True  //只有http请求中才可用，js代码无法获取cookie
		secure = True  //只有https请求才可用，用户修改cookie没用
		
	- 接收cookie
		tk = request.COOKIES.get("ticket")

	- 加签名
	set_signed_cookie("ticket","1qazxsw23edc",salt='abc')  
		salt = '' //加盐

	- 接收cookie+解密
		tk = request.get_signed_cookie("ticket",salt='abc')


## Session

### 原理
cookie是保存在客户端浏览器上的键值对
Session是保存在服务端的数据（本质也是键值对）

http请求特点是短链接，无状态
session与cookie具有同样的功能：用于服务端验证客户端信息，启到保持会话作用，
并且session应用需要依赖cookie
session与cookie具有不同的是：session不会将敏感信息直接给客户端

### django实现
发送session步骤：
1.生成 随机字符串
2.通过cookie将 随机字符串 发送给客户端
3.服务端将 随机字符串 对应 客户端信息 键值对{ 随机字符串：{‘username’：‘alex’,...} }保存到某个地方
	
	- 语句
		def func(request):
			request.session['username']='alex'
			# request.session['user_info']={'username':'alex',...} #可以写成字典，作为session随机字符串的value
			
			return ...
获取session步骤：
1.获取客户端cookie中的 随机字符串
2.去session中查询有没有对应 随机字符串
3.去session对应的key的value中查看是否有username

	- 语句
		def func2(request):
			v = request.session.get('username')
			# v = request.session.get('user_info').get('username')
			if v:
				登录成功
			else：
				失败

前端：
	
	- 前端直接通过request.session获取字典
		{{ request.session.user_info.username }}
session其他操作：

```python
 def index(request):
        # 获取、设置、删除Session中数据
        request.session['k1']
        request.session.get('k1',None)
        
        request.session['k1'] = 123
        request.session.setdefault('k1',123) # 存在则不设置
        
        del request.session['k1'] # 仅删除随机字符串对应数据
 
        # 所有 键、值、键值对
        request.session.keys()
        request.session.values()
        request.session.items()
        request.session.iterkeys()
        request.session.itervalues()
        request.session.iteritems()
 
        # 用户session的随机字符串
        s_k = request.session.session_key # 获取随机字符串本身
 
        # 将所有Session失效日期小于当前日期的数据删除
        request.session.clear_expired()
 
        # 检查 用户session的随机字符串 在数据库中是否存在
        request.session.exists(s_k)
 
        # 删除 当前用户的所有Session数据
        request.session.delete(s_k) # 可以用作登出loginout
        
        # session超时
        request.session.clear() # 可以用作登出loginout
 			
 		 # 修改 session超时时间 	
        request.session.set_expiry(value)
            * 如果value是个整数，session会在些秒数后失效。
            * 如果value是个datatime或timedelta，session就会在这个时间后失效。
            * 如果value是0,用户关闭浏览器session就会失效。
            * 如果value是None,session会依赖全局session失效策略。
```					

配置文件设置session settings.py
     
    SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
    SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
    SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
    SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
    SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
    SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
    SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）只要用户重新操作页面，session超时时间就会重置，可以设置为True
 

session保存设置

session默认保存在数据库，django可以让session放在内存、缓存、数据库（默认）、文件、cookie等等
想要session存放于不同位置，只需要修改配置文件settings.py

数据库：
   
    SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）
    
缓存：（缓存相当于另外一台服务器的内存）
	
	SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
	SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置

文件：

	SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
	SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir()                                                            # 如：/var/folders/d3/j9tj0gz93dg06bmwxmhh6_xm0000gn/T
 

缓存+数据库(优先去缓存拿，缓存没有去数据库拿)

	SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎


加密cookie（相当于又放到coolie，没什么用）

	SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎


## 数据分页
数据查询到后显示到页面，如何实现分页，Django自带分页功能，但是有些缺陷，所以我们自定义一个分页组件
### 分页核心功能
ORM通过类似python中的切片功能实现分批获取数据

	models.UserInfo.objects.all()[0:10]  # 从0开始取到9,取头不取尾
	models.UserInfo.objects.all()[10:20]  # 从10开始取到19
	
### Django自带分页
Django自带分页适用于只有上一页，下一页的情况

views.py

```py
def index(request):
    '''
    分页
    :param request: 
    :return: 
    '''
    current_page = request.GET.get('page')
    user_list = models.UserInfo.objects.all()
    paginator = Paginator(user_list, 10) # 用户列表，每页显示10条数据
    # ---paginator方法---
    # page:     page对象
    # per_page: 每页显示条目数量
    # count:    数据总个数
    # num_pages:总页数
    # page_range:总页数的索引范围，如: (1,10),(1,200)
    try:
        posts = paginator.page(current_page)  # 新建page对象
        # ---posts方法---
        # has_next              是否有下一页
        # next_page_number      下一页页码
        # has_previous          是否有上一页
        # previous_page_number  上一页页码
        # object_list           分页之后的数据列表
        # number                当前页
        # paginator             paginator对象
    except PageNotAnInteger as e:  # 页码不是整型
        posts = paginator.page(1)
    except EmptyPage as e:  # 页码为空
        posts = paginator.page(1)
	    return render(request, 'index.html', {'posts': posts})
```
index.html

```py
<ul>
    {% for i in posts.object_list %}
        <li>{{ i.name }}</li>
    {% endfor %}
</ul>
<div>
    {% if posts.has_previous %}
        <a href="/index.html?page={{ posts.previous_page_number }}">上一页</a>
    {% endif %}
</div>
<div>
    {% if posts.has_next %}
        <a href="/index.html?page={{ posts.next_page_number }}">下一页</a>
    {% endif %}
</div>
```

### 自定义一个分页组件
views.py

```py
from utils.pager import PageInfo

def custom(request):
    total_row = models.UserInfo.objects.all().count()  # 全部数据行数
    page_info = PageInfo(request.GET.get('page'), total_row, 10, '/custom.html', 11)
    user_list = models.UserInfo.objects.all()[page_info.start():page_info.end()]
    return render(request, 'custom.html', {"userlist": user_list, "page_info": page_info})
```
utils.pager.py

```py
class PageInfo(object):
    def __init__(self, current_page, total_row, per_page, base_url, show_page=11):
        '''

        :param current_page: 当前页
        :param total_row: 总数据行数
        :param per_page: 每页显示几条数据
        :param base_url: url地址
        :param show_page: 显示几页
        '''
        try:
            self.current_page = int(current_page)
        except Exception as e:
            self.current_page = 1
        self.per_page = per_page
        # 求页码数
        x, y = divmod(total_row, per_page)
        if y:  # 如果余数不为0，页码+1
            x = x + 1
        self.total_page = x
        self.show_page = show_page
        self.base_url = base_url

    def start(self):
        return (self.current_page - 1) * self.per_page

    def end(self):
        return self.current_page * self.per_page

    def pager(self):
        page_list = []
        half_page = int((self.show_page - 1) / 2)  # 1／2显示几页

        # 总页数小于显示几页 <11
        if self.total_page < self.show_page:
            begin = 1
            stop = self.total_page + 1
        # 总页数大于显示几页 >11
        else:
            # 当前页小于一半
            if self.current_page <= half_page:
                begin = 1
                stop = self.show_page + 1
            # 当前页大于一半
            else:
                # 当前页+一半大于总页码
                if self.current_page + half_page > self.total_page:
                    begin = self.total_page - self.show_page + 1
                    stop = self.total_page + 1
                # 当前页+一半小于总页码
                else:
                    begin = self.current_page - half_page
                    stop = self.current_page + half_page + 1
        if self.current_page <= 1:
            prev = '<li><a href="#">上一页</a></li>'
        else:
            prev = '<li><a href="%s?page=%s">上一页</a></li>' % (self.base_url, self.current_page - 1,)
        page_list.append(prev)

        for i in range(begin, stop):
            if i == self.current_page:
                temp = '<li class="active"><a href="%s?page=%s">%s</a></li>' % (self.base_url, i, i)
            else:
                temp = '<li><a href="%s?page=%s">%s</a></li>' % (self.base_url, i, i)
            page_list.append(temp)

        if self.current_page >= self.total_page:
            nex = '<li><a href="#">下一页</a></li>'
        else:
            nex = '<li><a href="%s?page=%s">下一页</a></li>' % (self.base_url, self.current_page + 1,)
        page_list.append(nex)

        return ''.join(page_list)

```
custom.html

```html
<ul>
    {% for i in userlist %}
        <li>{{ i.name }}</li>
    {% endfor %}
</ul>
<nav aria-label="Page navigation">
    <ul class="pagination">
        {{ page_info.pager|safe }}
        {#        html调用函数不需要加括号pager(),浏览器信任插入字符串safe#}
    </ul>
</nav>
```


## 缓存

## 序列化

## 信号



