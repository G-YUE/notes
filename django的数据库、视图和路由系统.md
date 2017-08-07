---
title:django的数据库操作、COOKIE和SESSION、和路由系统
data:2017年6月22日 08:47:59
categories:django
---
# 路由系统

本质: url----->函数

一一对应： 

	^index/  ------>def index

支持正则: 

	^login/(\d+)/  ---->def login(request,a1)

	^login/(?p<a1>\w+)/(?p<a2>\w+)/  ---->def login(request,a1,a2)

	以上两种不能混用，否则会接受不全	参数

伪静态界面：
	
	^login/(\d+).html$  ---->def login(request,a1)

根据app写url系统

	主：
	
		from django.conf.urls import url,include
	
		url(r'^app01/', include("app01.urls")),
	
	app01中urls
		
		url(r'^index.html/', views.index),

	这样输入url:app01/index.html   匹配到app01会到app01中的urls中匹配url 匹配到index.html会跳转到views.index函数


默认或者输错url跳转到指定页面

	url(r'^', views.default),


在函数中获取对应的url,反射出url

	^index/  ------>views.index,name="n1"

	from django.urls import reverse
	def index(request):
		url = reverse("n1")
		print(url)   #/index/

	def index(request):
		url = reverse("n1"，args=("123123"))
		print(url)   #/index/123123
	
	在HTML中{% url "n1"  %}----->/index/



# ORM数据库操作

## 数据库

操作表：

	创建表
	修改表
	删除表

操作数据行：

	增删改查
	增删改，参数都可以换成字典形式。

利用pymysql等第三方工具连接数据库


django默认:sqlite

Mysql默认：mysql--->MySQLDB

	如果用python3 则需要更改连接方式

	DATABASES = {
	    'default': {
	    'ENGINE': 'django.db.backends.mysql',
	    'NAME':'dbname',
	    'USER': 'root',
	    'PASSWORD': 'gaoyue123',
	    'HOST': 'localhost',
	    'PORT': 3306,
	    }
	}

	在与工程名相同的文件夹的init中写入

	import pymysql
	pymysql.install_as_MySQLdb()   #将MySQLdb替换为pymysql

注册app

settings中 INSTALLED_APPS 最后一行加入app名字

	INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app01',
	]

在models 中创建一个类

	from django.db import models

    ##一对多

	class Userinfo(models.Model):
	    nid = models.AutoField(primary_key=True)  # 自增整型,主键
	    username = models.CharField(max_length=32)  # 字符串类型，长度是32
	    password = models.CharField(max_length=64)  # 字符串长度64


		# ug_id  实际存储到数据库的列名是这个
        ug = models.ForeignKey("UserGroup",null=True)  设置外面 允许为空

		#如果一个表中有2个外键，而且连的是同一个表，可以给每个列取个别名
		ug=models.ForeignKey("userGroup",related_name="a")
		obj对象.a.all()

		ug=models.ForeignKey("userGroup",related_query_name="a")
		obj对象.a_set.all()
		obj对象.a__id.all()

	class Usertype(models.Model):
	    id = models.AutoField(primary_key=True)  # 自增整型,主键
	    title = models.CharField(max_length=32)  # 字符串类型，长度是32
	    




	##多对多


	a、手动创建关系表

		class Boy(models.Model):
			id=models.AutoField(primary_key=True)
			name=models.CharField(max_length=32)
	
		class Gril(models.Model):
			id=models.AutoField(primary_key=True)
			name=models.CharField(max_length=32)
	
		class Love(models.Model):
			b=models.ForeignKey("Boy")
			g=models.ForeignKey("Gril")
	
			##联合唯一索引
			class Meta:
				unique_together=[
					("b","g"),
				]


		查询与男生A有关系的女生姓名
		1、
			res=models.Boy.objects.filter(name="A").first()
			g_list=res.love_set.all()
			for i in g_list:
				print(i.g.name)
	
		2、
			res=models.Love.objects.filter(b__name="A")
			for i in res:
				print(i.g.name)
	
		3、
			res=models.Love.objects.filter(b__name="A").values("g__name")
			for i in res:
				print(i["g__name"])
		4.
			res=models.Love.objects.filter(b__name="A").select_related("g")
			for i in res:
				print(i.g.name)


	b、ManyToManyField 来让系统自己创建关系表

		class Boy(models.Model):
			id=models.AutoField(primary_key=True)
			name=models.CharField(max_length=32)
			m=models.ManyToManyField("Gril")
	
		class Girl(models.Model):
			id=models.AutoField(primary_key=True)
			name=models.CharField(max_length=32)
	
		这样在数据库中会创建3张表   boy表、gril表、boy_m表即boy和gril的关系表
	
		boy_m表中的结构：列名：id,boyid,gril_id
	
		这样我们再完成上面的操作
	
		obj=models.Boy.objects.filter(name="A").first()
		
		add() 添加
	    # obj.m.add(2)
	    # obj.m.add(2,4)
	    # obj.m.add(*[1,])
	
		remove() 删除
	    # obj.m.remove(1)
	    # obj.m.remove(2,3)
	    # obj.m.remove(*[4,])
	
		set() 重置
	    # obj.m.set([1,])
	
		all() 如果是通过boy调用，则返回的是gril对象列表
			正向
		    # q = obj.m.all()
		    # # [Girl对象]
		    # print(q)
		
		    # obj = models.Boy.objects.filter(name='方少伟').first()
		    # girl_list = obj.m.all()
		    # girl_list = obj.m.filter(nick='小鱼')
		    # print(girl_list)
		
			反向
		    # obj = models.Girl.objects.filter(nick='小鱼').first()
		    # v = obj.boy_set.all()
		    # print(v)
	
	
	    clear() 清空
	    # obj.m.clear()

	c、两种混用
		class Boy(models.Model):
			id=models.AutoField(primary_key=True)
			name=models.CharField(max_length=32)
			m=models.ManyToManyField("Gril")
	
		class Girl(models.Model):
			id=models.AutoField(primary_key=True)
			name=models.CharField(max_length=32)
	
		class Love(models.Model):
			b=models.ForeignKey("Boy")
			g=models.ForeignKey("Gril")

		这样会产生4个表 boy表、gril表、boy_m表即boy和gril的关系表  和 love表

		但是这时候就不能用m来进行add和remove来进行操作了，但是可以用clear

		这就需要建立m的时候这样写：
			m = models.ManyToManyField('Girl',through="Love",through_fields=('b','g',))
			through：表示从哪个表中找
			through_fields：表示找哪个字段

		这样在生成表的时候生成的就是3个表 boy表、gril表 和 love表
		
		查询可以通过m快速查询，例如拿上述问题举例：

		1、
			res=models.Boy.objects.filter(name="A").first()
			g_list=res.love_set.all()
			for i in g_list:
				print(i.g.name)
		
		2、

		 	obj = models.Boy.objects.filter(name='A').first()
	        g_list = obj.m.all()   拿的是gril的对象
 			for i in g_list:
				print(i.name)

ManyToManyField自关联特性：

			obj = models.UserInfo.objects.filter(id=1).first()
			# from_userinfo_id
			obj.m          => select xx from xx where from_userinfo_id = 1
			
			# to_userinfo_id
			obj.userinfo_set => select xx from xx where to_userinfo_id = 1
			
		
		定义：
			# 前面列：男生ID
			# 后面列：女生ID
			
		应用：
			# 男生对象
			obj = models.UserInfo.objects.filter(id=1).first()
			# 根据男生ID=1查找关联的所有的女神
			obj.m.all()
			
			# 女生
			obj = models.UserInfo.objects.filter(id=4).first()
			# 根据女生ID=4查找关联的所有的男生
			obj.userinfo_set.all()
			
		
ForeignKey自关联：

			class Comment(models.Model):
				"""
				评论表
				"""
				news_id = models.IntegerField()            # 新闻ID
				content = models.CharField(max_length=32)  # 评论内容
				user = models.CharField(max_length=32)     # 评论者
				reply = models.ForeignKey('Comment',null=True,blank=True,related_name='xxxx')
			"""
			   新闻ID                         reply_id
			1   1        别比比    root         null
			2   1        就比比    root         null
			3   1        瞎比比    shaowei      null
			4   2        写的正好  root         null
			5   1        拉倒吧    由清滨         2
			6   1        拉倒吧1    xxxxx         2
			7   1        拉倒吧2    xxxxx         5
			"""
			"""
			新闻1
				别比比
				就比比
					- 拉倒吧
						- 拉倒吧2
					- 拉倒吧1
				瞎比比
			新闻2：
				写的正好
			"""


字段和属性

字段类型

	字符串
			EmailField(CharField)：   
			IPAddressField(Field)   
			URLField(CharField)
			SlugField(CharField)
			UUIDField(Field)
			FilePathField(Field)
			FileField(Field)
			ImageField(FileField)
			CommaSeparatedIntegerField(CharField)
	时间类：
			models.DateTimeField(null=True)
	数字：
			num = models.IntegerField()
			num = models.FloatField()
			mum = models.DecimalField(max_digits=30,decimal_places=10)
	枚举（Django）：
			color_list = (
				(1,'黑色'),
				(2,'白色'),
				(3,'蓝色')
			)
			color = models.IntegerField(choices=color_list)   只能从color_list里选值
		
			1. 自己操作：
				自己取，自己用
			2. 给Django admin使用
				
			应用场景：选项固定
			
			PS: FK选项动态，男女选择
	
	字段参数：
		null=True,       允许为空
		default='1111',
		db_index=True,   是否创建索引
		unique=True      是否创建唯一索引
		
		class Meta:      创建唯一索引
			# unique_together = (
			#     ('email','ctime'),
			# )
			# index_together = (
			#     ('email','ctime'),
			# )
			db_table=("tablename")   #定义数据表名
	
		DjangoAdmin提供的参数：
			verbose_name        Admin中显示的字段名称
			4tr4  t`blank      editable            Admin中是否可以编辑
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



创建数据表

	命令行输入：
		python3 manage.py makemigrations # 生成配置文件-迁移到migrations
		python3 manage.py migrate        # 同步数据库


操作数据库：增删改查

	def index(request):
		import app01 import models
		#增加
		models.Userinfo.objects.create(username="egon",password="123456")

		obj = models.UserGroup.objects.filter(id=1).first()
        models.UserInfo.objects.create(user='root',password='123',age=10,ug=obj)




		一次性增加多条:bulk_create
		models.Userinfo.objects.bulk_create([models.Userinfo(user="a"),models.Userinfo(user="a"),....],5)

		[models.Userinfo(user="a"),models.Userinfo(user="a"),....]	 一堆数据，列表类型

		5  每次提交5条


		#查
		models.Userinfo.objects.all()

            group_list = models.UserInfo.objects.all()
            group_list获取的是QuerySet，内部元素是UserInfo对象，每一个对象代指一行数据，
            对象中如果包含ForeignKey，则代指与其关联表的一行数据

            for row in group_list:
                print(row.user)
                print(row.age)
                print(row.ug_id)    # 用户信息关联的部门ID
                print(row.ug.title) # 用户信息关联的部门名称


		通过group反向找出userinfo中的数据：
			obj = models.Group.objects.all().first() #取到group中第一条数据

			for i in obj.userinfo_set.all(): 找到group第一条数据相关联的userinfo中的数据
				print(i.id,i.username,i.password,i.ug_id)  打印userinfo中的数据
			
			models.Group.objects.values("id","title","userinfo 将group当外键表的小写表名") 


			models.Group.objects.values("id","title","userinfo_username") 


		values:  
		不能跨表：res=models.Userinfo.objects.values("id","username")  #只拿到3列 ug双下划线
				 #res--->  queryset   [{"id":1,"username":"aaa"},]

 				 res=models.Userinfo.objects.all().values_list("id","username")
 				 #res---> queryset   [(1,"aaa"),]
 		
		能跨表：res=models.Userinfo.objects.values("id","username"，"ug__name")  #只拿到3列
			   #res--->  queryset   [{"id":1,"username":"aaa","ug__name":对象},]

 			   res=models.Userinfo.objects.all().values_list("id","username")
 			   #res---> queryset   [(1,"aaa",对象),]

		models.Userinfo.objects.filter(id=1)
	
		models.Userinfo.objects.filter(id=1,name="egon") 默认and

		models.Userinfo.objects.filter(id_gt=1) id>1

		models.Userinfo.objects.filter(id_lt=1) id<1

		models.Userinfo.objects.filter(id_in=[1,2,3]) id in [1,2,3]

		
		models.Userinfo.objects.exclude(id=1)  id != 1

		#删

		models.Userinfo.objects.filter(id=1).del()

		#更新

		models.Userinfo.objects.filter(id=1).update(username="alex")



		#排序
	
		models.Userinfo.objects.all().order_by("id") #按照ID从小到大排列

		models.Userinfo.objects.all().order_by("-id") #按照ID从大到小排列


		#分组 

		from django.db.models import Count,Max,Min,Sum,Avg
		obj=models.Userinfo.objects.values("ut_id").annotate(别名=Count("id"))
		“select userinfo.ut_id,Count("id") as 别名  from userinfo group by userinfo.id”

		obj.query   #显示对应的sql原生语句


## django自带分页

	from django.core.paginator import Paginator,Page


	def index(request):


		current_page = request.GET.get('page') #获取页码

	    user_list = models.UserInfo.objects.all()  #从数据库中查询数据
	    paginator = Paginator(user_list,10)      #10条数据一组分组

	    # per_page: 每页显示条目数量
	    # count:    数据总个数
	    # num_pages:总页数
	    # page_range:总页数的索引范围，如: (1,10),(1,200)
	    # page:     page对象
	    try:
	        posts = paginator.page(current_page)
	    except PageNotAnInteger as e:
	        posts = paginator.page(1)
	    except EmptyPage as e:
	        posts = paginator.page(1)
	    # has_next              是否有下一页
	    # next_page_number      下一页页码
	    # has_previous          是否有上一页
	    # previous_page_number  上一页页码
	    # object_list           分页之后的数据列表
	    # number                当前页
	    # paginator             paginator对象
	    return render(request,'index.html',{'posts':posts})
	

		html:
			{% for i in posts.object_list %}
				{{ i }}
			{% endfor %}

## 自定义分页

	def newpage(request):
		page=request.GET.get("page")

		1   0 -10
		2   10-20


		start_page=(page-1)*10
		end_page=page*10

		user_list=models.Userinfo.objects.all()[start_page,end_page]
		
		return render(request,"user.html",{"userlist":user_list})

# COOKIE和SESSION

## COOKIE   向服务器发送存在请求头里。服务器发送到浏览器存在响应头里。

	a、保存在浏览器端的"键值对"，服务端可以向浏览器端写cookie
	b、浏览器每次发送请求时，会携带cookie	

	应用：
		a、投票
		b、用户登录

	登录时，如果用户名和密码正确，可以写
		obj=render(request,"index.html")
		obj.set_cookie("键"，"值",max_age=10，path="/")  #max_age超时时间，浏览器保存的cookie有效时间。 10秒

											            	#或者expires 他跟的参数是2017年6月21日 11:50:58
											            	#path 指定某个url可以使用当前的cookie path="/index/"    /表示所有url都可以用

		return obj


		obj=set_signed_cookie("键","值",salt="加盐操作")

	接收端接收cookie
		cook=request.COOKIES.get("上面中的键")
		cook=request.get_signed_cookie("键",salt="加盐")

## 	SESSION

	a、保存在服务器端的数据（本质是键值对）

	b、依赖cookie

	c、保持会话（web网站）

	好处：敏感信息不会直接给客户端

### 存放位置

	1、数据库中   django默认存放在数据库中

		Django默认支持Session，并且默认是将Session数据存储在数据库中，即：django_session 表中。
 
		a. 配置 settings.py
 
		    SESSION_ENGINE = 'django.contrib.sessions.backends.db'   # 引擎（默认）
		     
		    SESSION_COOKIE_NAME ＝ "sessionid"                       # Session的cookie保存在浏览器上时的key，即：sessionid＝随机字符串（默认）
		    SESSION_COOKIE_PATH ＝ "/"                               # Session的cookie保存的路径（默认）
		    SESSION_COOKIE_DOMAIN = None                             # Session的cookie保存的域名（默认）
		    SESSION_COOKIE_SECURE = False                            # 是否Https传输cookie（默认）
		    SESSION_COOKIE_HTTPONLY = True                           # 是否Session的cookie只支持http传输（默认）
		    SESSION_COOKIE_AGE = 1209600                             # Session的cookie失效日期（2周）（默认）
		    SESSION_EXPIRE_AT_BROWSER_CLOSE = False                  # 是否关闭浏览器使得Session过期（默认）
		    SESSION_SAVE_EVERY_REQUEST = False                       # 是否每次请求都保存Session，默认修改之后才保存（默认）
		    


	2、缓存中

		a. 配置 settings.py
			SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 引擎
    		SESSION_CACHE_ALIAS = 'default'                            # 使用的缓存别名（默认内存缓存，也可以是memcache），此处别名依赖缓存的设置
			
		其他同上

	3、文件中

		a. 配置 settings.py
			SESSION_ENGINE = 'django.contrib.sessions.backends.file'    # 引擎
    		SESSION_FILE_PATH = None                                    # 缓存文件路径，如果为None，则使用tempfile模块获取一个临时地址tempfile.gettempdir() 
		其他同上

	4、加密的cookie中
		a. 配置 settings.py
    		SESSION_ENGINE = 'django.contrib.sessions.backends.signed_cookies'   # 引擎

	5、缓存+数据库

		a. 配置 settings.py
    		SESSION_ENGINE = 'django.contrib.sessions.backends.cached_db'        # 引擎

### 查询、设置、删除、修改

	# 获取、设置、删除Session中数据
	    request.session['k1']     #不存在会报错
	    request.session.get('k1',None)
	    request.session['k1'] = 123
	    request.session.setdefault('k1',123) # 存在则不设置


	    del request.session['k1']
		request.session.delete(request.session.session_key)  #删除session
		request.session.clear()   #删除cookie
		
    # 所有 键、值、键值对
	    request.session.keys()
	    request.session.values()
	    request.session.items()
	    request.session.iterkeys()
	    request.session.itervalues()
	    request.session.iteritems()


    # 用户session的随机字符串
    	request.session.session_key

    # 将所有Session失效日期小于当前日期的数据删除
    	request.session.clear_expired()

    # 检查 用户session的随机字符串 在数据库中是否
    	request.session.exists("session_key")

    # 删除当前用户的所有Session数据
    	request.session.delete("session_key")

	# 设置失效期
	    request.session.set_expiry(value)
	        * 如果value是个整数，session会在些秒数后失效。
	        * 如果value是个datatime或timedelta，session就会在这个时间后失效。
	        * 如果value是0,用户关闭浏览器session就会失效。
	        * 如果value是None,session会依赖全局session失效策略。
	




# CBV 和 FBV

## CBV  url--->类

	url(r'^index.html$', views.Login.as_view()),

	app01.views

	from django.views import View
	class Login(View):
		'''
			get     查
			post    创建
			put	    更新
			delete  删除
		'''
		def dispath(self,request,*args,**kwargs):
			obj= super(Login,self).dispatj(request,*args,**kwargs)
			return obj

		def get(self,request):  如果是以get方式访问的执行这个函数
			return HttpResponse("get")

		def post(self,request):如果是以post方式访问的执行这个函数
			return HttpResponse("post")