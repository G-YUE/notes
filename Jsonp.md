JSONP

		技巧，技术。。。
		Ajax存在：
			访问自己域名URL - 顺利
			访问其他域名URL - 被阻止
		
		浏览器：同源策略，
			- 禁止：Ajax跨域发送请求时，再回来时浏览器拒绝接受
			- 允许：script标签没禁止 src属性没限制
		
		JSONP：钻空子
			# http://www.baidu.com?p=1&name=xx
			0. function list(arg){
				console.log(arg);
			  }
			
			1. 发送：
				把数据拼接成，script放在html
				<script src='http://www.jxntv.cn/data/jmd-jxtv2.html?callback=list&_=1454376870403'></script>


开发需求：向其他网站发送HTTP请求

	1、浏览器直接发送请求（考虑同源）
		
		浏览器->服务端->发送请求

	  a、jsonp

		<a onclick="sendMsg();">发送</a>

	    <script>
	        function sendMsg(){
	            var tag = document.createElement('script');
	            tag.src = "http://127.0.0.1:8000/index/`";
	            document.head.appendChild(tag);
	        }
	
			function list(arg){
				console.log(arg)
				}
    	</script>

		


		def index(request):
		    a=json.dumps("123")
		    x="list(%s)"%a
		    print(x)
		    return HttpResponse(x)

 		b、jquary ajax jsonp
	
			http://www.s4.com:8001/users/?funcname=bbb


		
![Markdown](http://i2.kiimg.com/1949/b52466e397bf1376.png)



		其他
	
			--只能发送GET请求。即使写的是POST，后台也会自动改成GET
			--双方需要约定好特殊规则

		JSONP是一种方式，目的解决跨域问题


		c、cors 跨域资源共享  服务器返回数据的时候在响应头中加入一条数据，会发送2次请求

		HTML：
			
			<input type="button" value="获取用户列表" onclick="getUsers();" />
		    <ul id="user_list">
		
		    </ul>
		    <script src="/static/jquery-1.12.4.js"></script>
		    <script>
		        function getUsers(){
		            $.ajax({
		                url: 'http://www.s4.com:8001/new_users/',
		                type:"DELETE",
		                success:function(arg){
		                    console.log(arg);
		                }
		            })
		        }
		    </script>




		目标服务器：

			def new_users(request):
			    print(request.method)
			    if request.method == "OPTIONS":
			        obj = HttpResponse()
			        obj['Access-Control-Allow-Origin'] = "允许跨域访问的域名"
			        obj['Access-Control-Allow-Methods'] = "DELETE"
			        return obj
			
			    obj = HttpResponse('asdfasdf')
			    obj['Access-Control-Allow-Origin'] = "*"   允许跨域访问的域名，这样浏览器就可以不阻止
			    return obj





	2、后端服务器发送请求返回给浏览器

	