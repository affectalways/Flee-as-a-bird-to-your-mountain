---
title: "Werkzeug 源码解析(2)"
tags: ["flask", "python", "werkzeug"]
keywords: 
- flask
- 源码
- werkzeug
- python
- source code
- hugo
- blog
date: 2020-06-23T23:08:16+08:00
categories: ["python", "werkzeug"]
draft: false
---

# 从哪开始呢？

1. 熟悉目录层级

2. 参照[Werkzeug](https://werkzeug.palletsprojects.com/en/1.0.x/)官方文档给出的[示例](https://werkzeug.palletsprojects.com/en/1.0.x/tutorial/#introducing-shortly)，重点关注`Request`,`Response`,`run_simple`

   ```python
   from werkzeug.wrappers import Request, Response
   
   def application(environ, start_response):
       request = Request(environ)
       text = 'Hello %s!' % request.args.get('name', 'World')
       response = Response(text, mimetype='text/plain')
       return response(environ, start_response)
   
   
   
   """
   	省略超多内容
   """
   class Shortly(object):
   
       def __init__(self, config):
           self.redis = redis.Redis(config['redis_host'], config['redis_port'])
   
       def dispatch_request(self, request):
           return Response('Hello World!')
   
       def wsgi_app(self, environ, start_response):
           request = Request(environ)
           response = self.dispatch_request(request)
           return response(environ, start_response)
   
       def __call__(self, environ, start_response):
           return self.wsgi_app(environ, start_response)
   
   
   def create_app(redis_host='localhost', redis_port=6379, with_static=True):
       app = Shortly({
           'redis_host':       redis_host,
           'redis_port':       redis_port
       })
       if with_static:
           app.wsgi_app = SharedDataMiddleware(app.wsgi_app, {
               '/static':  os.path.join(os.path.dirname(__file__), 'static')
           })
       return app
   
   
   if __name__ == '__main__':
       from werkzeug.serving import run_simple
       app = create_app()
       run_simple('127.0.0.1', 5000, app, use_debugger=True, use_reloader)
   ```





# Request、Response、run_simple（最不重要的）

之后的文章就重点关注以上三个，而且肯定会延伸到Werkzeug的其他地方（源码），不必担心，肯定会有所涉猎。

讲真，感觉run_simple可以不重点关注，希望不要打脸

