Web框架本质：socket

		- wsgiref+django
		- uwsgi+django


python标准库提供的独立WSGI服务器称为wsgiref。
	
	from wsgiref.simple_server import make_server
	 
	 
	def RunServer(environ, start_response):
	    start_response('200 OK', [('Content-Type', 'text/html')])
	    return [bytes('<h1>Hello, web!</h1>', encoding='utf-8'), ]
	 
	 
	if __name__ == '__main__':
	    httpd = make_server('', 8000, RunServer)
	    print("Serving HTTP on port 8000...")
	    httpd.serve_forever()




自定义一个web框架：

	from wsgiref.simple_server import make_server
	 
	def index():
	    return 'index'
	 
	def login():
	    return 'login'
	 

	def routers():
	   
	    urlpatterns = (
	        ('/index/',index),
	        ('/login/',login),
	    )
	     
	    return urlpatterns
	 
	def RunServer(environ, start_response):
	    start_response('200 OK', [('Content-Type', 'text/html')])
	    url = environ['PATH_INFO']
	    urlpatterns = routers()
	    func = None
	    for item in urlpatterns:
	        if item[0] == url:
	            func = item[1]
	            break
	    if func:
	        return func()
	    else:
	        return '404 not found'
	     
	if __name__ == '__main__':
	    httpd = make_server('', 8000, RunServer)
	    print "Serving HTTP on port 8000..."
	    httpd.serve_forever()