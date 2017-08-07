# 1、Ajax

	a、原生Ajax，XMLHttpRequest	

				GET请求：
					var xhr = new XMLHttpRequest();
					xhr.onreadystatechange = function(){
						if(xhr.readyState == 4){
							alert(xhr.responseText);
						}
					};
					xhr.open('GET','/add2/?i1=12&i2=19');
					xhr.send();
				
				
				POST请求：
					var xhr = new XMLHttpRequest();
					xhr.onreadystatechange = function(){
						if(xhr.readyState == 4){
							alert(xhr.responseText);
						}
					};
					xhr.open('POST','/add2/');
			****    xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded');
			****	xhr.send("i1=12&i2=19");


	b、jQuery Ajax  ：jQuary 本身没有Ajax，他内部基于原生"Ajax"

		$.ajax(
			{
			url:"/index.html/",	
			type:"POST",
			data:{"name":"gaoyue"},
			dataType:"Json",
			success:function(arg){
				alert("成功！！！")	
				}
	
			｝
		)

## 原生Ajax-->XmlHttpRequest对象介绍

### XmlHttpRequest对象的主要方法：


a. void open(String method,String url,Boolen async)

	   用于创建请求
	    
	   参数：
	       method： 请求方式（字符串类型），如：POST、GET、DELETE...
	       url：    要请求的地址（字符串类型）
	       async：  是否异步（布尔类型）
 
b. void send(String body)

    用于发送请求
 
    参数：
        body： 要发送的数据（字符串类型）
 
c. void setRequestHeader(String header,String value)

    用于设置请求头
 
    参数：
        header： 请求头的key（字符串类型）
        vlaue：  请求头的value（字符串类型）
 
d. String getAllResponseHeaders()

    获取所有响应头
 
    返回值：
        响应头数据（字符串类型）
 
e. String getResponseHeader(String header)

    获取响应头中指定header的值
 
    参数：
        header： 响应头的key（字符串类型）
 
    返回值：
        响应头中指定的header对应的值
 
f. void abort()
 
    终止请求

g. append()

     添加数据，如
	formData.append('k1','v1')
	formData.append('fafafa',document.getElementById('i1').files[0]);


### XmlHttpRequest对象的主要属性：

a. (Number) readyState

      状态值（整数）
 
	   详细：
	      0-未初始化，尚未调用open()方法；
	      1-启动，调用了open()方法，未调用send()方法；
	      2-发送，已经调用了send()方法，未接收到响应；
	      3-接收，已经接收到部分响应数据；
	      4-完成，已经接收到全部响应数据；
 
b. (Function) onreadystatechange

   	当readyState的值改变时自动触发执行其对应的函数（回调函数）
 
c. (String) responseText

   	服务器返回的数据（字符串类型）
 
d. (XmlDocument) responseXML

   	服务器返回的数据（Xml对象）
 
e. (Number) states

   	状态码（整数），如：200、404...
 
f. (String) statesText

   	状态文本（字符串），如：OK、NotFound...


# 2、伪造Ajax

		技术：
			iframe标签，不刷新发送HTTP请求
			<form>....</form>
		
		示例：
			<form id="f1" method="POST" action="/fake_ajax/" target="ifr">
				<iframe id="ifr" name="ifr" style="display: none"></iframe>
				<input type="text" name="user" />
				<a onclick="submitForm();">提交</a>
			</form>

			<script>
				function submitForm(){
					document.getElementById('ifr').onload = loadIframe;
					document.getElementById('f1').submit();

				}
				function loadIframe(){
					var content = document.getElementById('ifr').contentWindow.document.body.innerText;
					alert(content);
				}
			</script>



