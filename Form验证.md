Form组件作用：

	-对用户提交的数据进行校验
	
		-form提交（刷新，丢失上次数据）
	
		-ajax提交（不刷新，上次数据保留）

	-保存用户上一次输入的内容


form验证流程：

	-反射查找单独字段的验证功能

	-找到单独字段验证方法进行验证
		
	-整体验证
	

Django提供 Form组件：

			1. 定义规则
				from django.forms import Form
				from django.forms import fields
				class xxx(Form):
					xx = fields.CharField(required=True,max_lenght.,min,error_message=)
		
			2. 使用	
				
				obj = xxx(request.POST)

				# 是否校验成功  True 或者 False
				v = obj.is_valid()

				# html标签name属性 = Form类字段名
				# 所有错误信息
				obj.errors
				
				# 正确信息
				obj.cleaned_data
			3、HTML页面  form是后台传入前台的参数

				<form action="/" method="POST" enctype="multipart/form-data">
				    <p>{{ form.user }} {{ form.errors.user.0 }}</p>
				    <p>{{ form.gender }} {{ form.errors.gender.0  }}</p>
				    <p>{{ form.city }} {{ form.errors.city.0  }}</p>
				    <p>{{ form.pwd }} {{ form.errors.pwd.0  }}</p>
				    <input type="submit"/>
				</form>

			或者
				{{ form.as_p }}  将传入的html标签全部显示出来

下面自己定义一个用户名和密码的验证：

	from django.forms import Form
	from django.forms import fields

	class LoginForm(Form):
	    # 正则验证: 不能为空,长度6-18
	    username = fields.CharField(
	        max_length=18,   #最大长度18
	        min_length=6,    #最小长度6
	        required=True,   #不允许为空，False允许为空
	        error_messages={
	            'required': '用户名不能为空',  #自定义报错信息。
	            'min_length': '太短了',
	            'max_length': '太长了',
	        }
	    )
	    # 正则验证: 不能为空，16+
	    password = fields.CharField(min_length=16,required=True)


	    # email = fields.EmailField()
	    # email = fields.GenericIPAddressField()
	    # email = fields.IntegerField()
	
	
	def login(request):
	    if request.method == "GET":
	        return render(request,'login.html')
	    else:
	       obj = LoginForm(request.POST)
	       if obj.is_valid():
	           # 用户输入格式正确
	           print(obj.cleaned_data) # 字典类型,输入的信息
	           #{'username': '123123123', 'password': '123123123123123123123123123'}
	           return redirect('http://www.baidu.com')
	       else:
				print(obj.erros)
	           # 用户输入格式错误
	           #<tr><th>
	           # <label for="id_username">Username:</label>
	           # </th>
	           # <td>
	           # <ul class ="errorlist" > < li > 用户名不能为空 < / li > < / ul >
	           # <input type="text" name="username" value="123123123" maxlength="18" minlength="6" required id="id_username" />
	           # </td></tr>
	           return render(request,'login.html',{'obj':obj})


	HTML：

		<form action="/login/" method="POST" id=“form1”>
		    {% csrf_token %}
		    <p><input type="text" name="username">{{ obj.errors.username.0 }}</p>
		    <p><input type="text" name="password">{{ obj.errors.password.0 }}</p>
		    <input type="submit" value="登录">
		</form>

	PS：ajax获取form的数据可以用简单的方法：
			-data:{"username":$("input [name='username']").val()}

			-date:#("#form1").serialize()  #会把id=form1的form所有input数据打包 


  

		form验证生成的HTML标签代码。现在主流的浏览器都会自动加上验证，要不像让浏览器加上验证

		<form action="/login/" method="POST" id=“form1” novalidate>

		novalidate：这样浏览器就不会加上验证功能了。



		obj = LoginForm() 参数还可以传入初始值

		-obj = LoginForm(data={'title': 'asdfasdfasdfas'}） #给title字段设置初始值，该初始值也会进行form验证

		-obj = LoginForm(initial={'title': row.title})   #给title字段设置初始值，第一次不会验证
		 





Django内置字段：

	invalid    格式错误

	widgets 设置生成的HTML标签样式
	
		-TextInput
		-Select
		-PasswordInput

Field

	    required=True,               是否允许为空
	    widget=None,                 自动生成HTML标签

			from django.forms import widgets
			name = fields.CharField(
				widget=widgets.TextInput(attrs={'class': 'form-control'}) 
			)
			给name生成的HTML标签Textinput添加一个class样式叫form-control

			widget=widgets.Select(
				choices=models.Classes.objects.values_list('id','title'),attrs={'class': 'form-control'})
	
			从数据库中读取元组类型（id,值） 生成select下拉框。
	


	    label=None,                  用于生成Label标签或显示内容
	    initial=None,                标签初始值
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
 
RegexField(CharField) 自定制正则表达式

	    regex,                      自定制正则表达式
	    max_length=None,            最大长度
	    min_length=None,            最小长度
	    error_message=None,         忽略，错误信息使用 error_messages={'invalid': '...'}
	 
EmailField(CharField)  
    
	    ...
	 
FileField(Field)   上传文件

	    allow_empty_file=False     是否允许空文件
	 
ImageField(FileField)   上传图片
    
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

	    ...   django.forms.models.ModelMultipleChoiceField
	 
	 
	     
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
	 

Django内置插件：

	TextInput(Input)
	NumberInput(TextInput)
	EmailInput(TextInput)
	URLInput(TextInput)
	PasswordInput(TextInput)
	HiddenInput(TextInput)
	Textarea(Widget)
	DateInput(DateTimeBaseInput)
	DateTimeInput(DateTimeBaseInput)
	TimeInput(DateTimeBaseInput)
	CheckboxInput
	Select
	NullBooleanSelect
	SelectMultiple
	RadioSelect
	CheckboxSelectMultiple
	FileInput
	ClearableFileInput
	MultipleHiddenInput
	SplitDateTimeWidget
	SplitHiddenDateTimeWidget
	SelectDateWidget


常见插件：

	# 单radio，值为字符串
	# user = fields.CharField(
	#     initial=2,
	#     widget=widgets.RadioSelect(choices=((1,'上海'),(2,'北京'),))
	# )
	 
	# 单radio，值为字符串
	# user = fields.ChoiceField(
	#     choices=((1, '上海'), (2, '北京'),),
	#     initial=2,
	#     widget=widgets.RadioSelect
	# )
	 
	# 单select，值为字符串
	# user = fields.CharField(
	#     initial=2,
	#     widget=widgets.Select(choices=((1,'上海'),(2,'北京'),))
	# )
	 
	# 单select，值为字符串
	# user = fields.ChoiceField(
	#     choices=((1, '上海'), (2, '北京'),),
	#     initial=2,
	#     widget=widgets.Select
	# )
	 
	# 多选select，值为列表
	# user = fields.MultipleChoiceField(
	#     choices=((1,'上海'),(2,'北京'),),
	#     initial=[1,],
	#     widget=widgets.SelectMultiple
	# )
	 
	 
	# 单checkbox
	# user = fields.CharField(
	#     widget=widgets.CheckboxInput()
	# )
	 
	 
	# 多选checkbox,值为列表
	# user = fields.MultipleChoiceField(
	#     initial=[2, ],
	#     choices=((1, '上海'), (2, '北京'),),
	#     widget=widgets.CheckboxSelectMultiple
	# )



Select框：

	单选
		cls_id = fields.IntegerField(
			widget=widgets.Select(
				choices=models.Classes.objects.values_list('id','title'),attrs={'class': 'form-control'})
		)
		
		cls_id = fields.ChoiceField(
			choices=models.Classes.objects.values_list('id','title'),
			widget=widgets.Select(attrs={'class': 'form-control'})
		)
		
		
		obj = FooForm({'cls_id':1})
	多选
		xx = fields.MultipleChoiceField(
			choices=models.Classes.objects.values_list('id','title'),
			widget=widgets.SelectMultiple
		)
		
		obj = FooForm({'cls_id':[1,2,3]})


		
修复Bug，刷新无法动态显示数据库内容：

	方式一：
		from django.forms import models as form_model

		class TeacherForm(Form):
			tname = fields.CharField(min_length=2)
			# xx = form_model.ModelMultipleChoiceField(queryset=models.Classes.objects.all())

			xx = form_model.ModelChoiceField(queryset=models.Classes.objects.all())

	方式二：
		class TeacherForm(Form):
			tname = fields.CharField(min_length=2)

			xx = fields.MultipleChoiceField(
				widget=widgets.SelectMultiple
			)
			def __init__(self,*args,**kwargs):
				super(TeacherForm,self).__init__(*args,**kwargs)
				self.fields['xx'].choices = models.Classes.objects.values_list('id','title')

		PS:self.fields['xx'].choices和self.fields['xx'].widget.choices的区别

			不加widget是验证单选下拉框，即使能多选，传入到后台也是单值。

扩展

	- is_valid
		- 字段 = 默认正则表达式
			 - 额外的正则
				from django.forms import Form
				from django.forms import widgets
				from django.forms import fields
				from django.core.validators import RegexValidator
				 
				class MyForm(Form):
					user = fields.CharField(
						validators=[RegexValidator(r'^[0-9]+$', '请输入数字'), RegexValidator(r'^159[0-9]+$', '数字必须以159开头')],
					)
		- clean_字段，必须返回值


		- clean()   整体验证
			有返回值：cleaned_data = 返回值
			无返回值：cleaned_data = 原来的值


自定义规则：

1、

	from django.forms import Form
	from django.forms import widgets
	from django.forms import fields
	from django.core.validators import RegexValidator
	 
	class MyForm(Form):
	    user = fields.CharField(
	        validators=[RegexValidator(r'^[0-9]+$', '请输入数字'), RegexValidator(r'^159[0-9]+$', '数字必须以159开头')],
	    )
			
2、

	import re
	from django.forms import Form
	from django.forms import widgets
	from django.forms import fields
	from django.core.exceptions import ValidationError
	 
	 
	# 自定义验证规则
	def mobile_validate(value):
	    mobile_re = re.compile(r'^(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$')
	    if not mobile_re.match(value):
	        raise ValidationError('手机号码格式错误')
	 
	 
	class PublishForm(Form):
 
 
	    title = fields.CharField(max_length=20,
	                            min_length=5,
	                            error_messages={'required': '标题不能为空',
	                                            'min_length': '标题最少为5个字符',
	                                            'max_length': '标题最多为20个字符'},
	                            widget=widgets.TextInput(attrs={'class': "form-control",
	                                                          'placeholder': '标题5-20个字符'}))
	 
	 
	    # 使用自定义验证规则 validators：自定义验证器
	    phone = fields.CharField(validators=[mobile_validate, ],
	                            error_messages={'required': '手机不能为空'},
	                            widget=widgets.TextInput(attrs={'class': "form-control",
	                                                          'placeholder': u'手机号码'}))
	 
	    email = fields.EmailField(required=False,
	                            error_messages={'required': u'邮箱不能为空','invalid': u'邮箱格式错误'},
	                            widget=widgets.TextInput(attrs={'class': "form-control", 'placeholder': u'邮箱'}))				


3、

	from django import forms
    from django.forms import fields
    from django.forms import widgets
    from django.core.exceptions import ValidationError
    from django.core.validators import RegexValidator
 
    class FInfo(forms.Form):
        username = fields.CharField(max_length=5,
                                    validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid extension.', 'invalid')], )
        email = fields.EmailField()
 
        def clean_username(self):
            """
            Form中字段中定义的格式匹配完之后，执行此方法进行验证
            :return:
            """
            value = self.cleaned_data['username']
            if "666" in value:
                raise ValidationError('666已经被玩烂了...', 'invalid')
            return value


4、

	from django.forms import Form
	from django.forms import widgets
	from django.forms import fields
	 
	from django.core.validators import RegexValidator
	 
	 
	############## 自定义字段 ##############
	class PhoneField(fields.MultiValueField):
	    def __init__(self, *args, **kwargs):
	        # Define one message for all fields.
	        error_messages = {
	            'incomplete': 'Enter a country calling code and a phone number.',
	        }
	        # Or define a different message for each field.
	        f = (
	            fields.CharField(
	                error_messages={'incomplete': 'Enter a country calling code.'},
	                validators=[
	                    RegexValidator(r'^[0-9]+$', 'Enter a valid country calling code.'),
	                ],
	            ),
	            fields.CharField(
	                error_messages={'incomplete': 'Enter a phone number.'},
	                validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid phone number.')],
	            ),
	            fields.CharField(
	                validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid extension.')],
	                required=False,
	            ),
	        )
	        super(PhoneField, self).__init__(error_messages=error_messages, fields=f, require_all_fields=False, *args,**kwargs)
	 
	    def compress(self, data_list):
	        """
	        当用户验证都通过后，该值返回给用户
	        :param data_list:
	        :return:
	        """
	        return data_list
	 
	############## 自定义插件 ##############
	class SplitPhoneWidget(widgets.MultiWidget):
	    def __init__(self):
	        ws = (
	            widgets.TextInput(),
	            widgets.TextInput(),
	            widgets.TextInput(),
	        )
	        super(SplitPhoneWidget, self).__init__(ws)
	 
	    def decompress(self, value):
	        """
	        处理初始值，当初始值initial不是列表时，调用该方法
	        :param value:
	        :return:
	        """
	        if value:
	            return value.split(',')
	        return [None, None, None]