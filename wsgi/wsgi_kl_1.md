---
title: "WSGI 理解（1）"
tags: ["flask", "python", "wsgi"]
keywords: 
- flask
- 源码
- python
- source code
- hugo
- wsgi
date: 2020-06-20T11:24:01+08:00
categories: ["python", "WSGI"]
draft: false
---

## WSGI 是个什么东西？

实际的生产环境中，Python应用程序是放在服务器的http server（比如Apache、Nginx等）上的。现在的问题是http server（之后以服务器代称）怎么把接收到的请求传递给Python应用程序？这就是WSGI做的事情。

`WSGI（Web Server Gateway Interface）`即Web服务器网关接口，解耦了`服务器（Apache、Nginx等）`和`Python应用程序`，是Python开发者只需要关注Python应用程序的开发。

> Web Server：即HTTP Server，接收用户的请求并返回响应信息；分为以下两部分：
>
> 1. 服务器，如Apache、Nginx等
> 2. Python应用程序，负责处理业务逻辑

![wsgi_framework.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/wsgi/wsgi_1/wsgi_1_framework.png?raw=true)

![wsgi.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/wsgi/wsgi_1/wsgi_1_wsgi.png?raw=true)



## HTTP Server 实现

`服务器`每接收到一个请求就调用一次`Python Application`。`服务器`作用如下

- 接收HTTP请求
- 提供`environ`和回调函数`start_response`，并传给`callable object`
- 调用`callable object`

以下是[PEP-3333](https://www.python.org/dev/peps/pep-3333/#the-application-framework-side)提供的示例

```
import os, sys

enc, esc = sys.getfilesystemencoding(), 'surrogateescape'

def unicode_to_wsgi(u):
    # Convert an environment variable to a WSGI "bytes-as-unicode" string
    return u.encode(enc, esc).decode('iso-8859-1')

def wsgi_to_bytes(s):
    return s.encode('iso-8859-1')


def run_with_cgi(application):
	"""
		application: 是Python Application中的callable object
	"""
    # 构造environ变量，dict类型，里面的内容是一次HTTP请求的环境变量
    environ = {k: unicode_to_wsgi(v) for k,v in os.environ.items()}
    environ['wsgi.input']        = sys.stdin.buffer
    environ['wsgi.errors']       = sys.stderr
    environ['wsgi.version']      = (1, 0)
    environ['wsgi.multithread']  = False
    environ['wsgi.multiprocess'] = True
    environ['wsgi.run_once']     = True

    if environ.get('HTTPS', 'off') in ('on', '1'):
        environ['wsgi.url_scheme'] = 'https'
    else:
        environ['wsgi.url_scheme'] = 'http'

    headers_set = []
    headers_sent = []

    # 把响应信息写到终端
    def write(data):
        out = sys.stdout.buffer

        if not headers_set:
             raise AssertionError("write() before start_response()")

        elif not headers_sent:
             # Before the first output, send the stored headers
             status, response_headers = headers_sent[:] = headers_set
             out.write(wsgi_to_bytes('Status: %s\r\n' % status))
             for header in response_headers:
                 out.write(wsgi_to_bytes('%s: %s\r\n' % header))
             out.write(wsgi_to_bytes('\r\n'))

        out.write(data)
        out.flush()

    # 定义start_response回调函数
    def start_response(status, response_headers, exc_info=None):
        if exc_info:
            try:
                if headers_sent:
                    # Re-raise original exception if headers sent
                    raise exc_info[1].with_traceback(exc_info[2])
            finally:
                exc_info = None     # avoid dangling circular ref
        elif headers_set:
            raise AssertionError("Headers already set!")

        headers_set[:] = [status, response_headers]

        # Note: error checking on the headers should happen here,
        # *after* the headers are set.  That way, if an error
        # occurs, start_response can only be re-called with
        # exc_info set.

        return write

    result = application(environ, start_response)
    try:
        # 处理application返回的结果（可迭代）
        for data in result:
            if data:    # don't send headers until body appears
                write(data)
        if not headers_sent:
            write('')   # send headers now if body was empty
    finally:
        if hasattr(result, 'close'):
            result.close()
```





## 中间件Middleware

`Middlerware`是位于`Http Server`和`Python Application`之间的功能组件。

对于`Http Server`而言，`Middlerware`就是应用程序；对于`Python Application`而言，`Middlerware`就是`Http Server`。`Middleware`对`Http Server`和`Python Application`是透明的，把从`Http Server`接收到的请求进行处理并向后传递，一直传递给`Python Application`，最后把`Python Application`的处理结果返回给`Http Server`。如下图：

![wsgi_middlerware.png](https://github.com/affectalways/affectalways.github.io/blob/master/images/wsgi/wsgi_1/wsgiframeworkmiddleware.png?raw=true)

`Middlerware`组件可执行以下功能：

- 根据 url 把用户请求调度到不同的 Python Application 中。
- 负载均衡，转发用户请求
- 预处理 XSL 等相关数据
- 限制请求速率，设置白名单

> PS：WSGI 的 middleware 体现了 unix 的哲学之一：do one thing and do it well。

本例实现了一个关于异常处理的 middleware（[摘自](https://lucumr.pocoo.org/2007/5/21/getting-started-with-wsgi/)）：

```
from sys import exc_info
from traceback import format_tb

class ExceptionMiddleware(object):
    """The middleware we use."""

    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        """Call the application can catch exceptions."""
        appiter = None
        # just call the application and send the output back
        # unchanged but catch exceptions
        try:
            appiter = self.app(environ, start_response)
            for item in appiter:
                yield item
        # if an exception occours we get the exception information
        # and prepare a traceback we can render
        except:
            e_type, e_value, tb = exc_info()
            traceback = ['Traceback (most recent call last):']
            traceback += format_tb(tb)
            traceback.append('%s: %s' % (e_type.__name__, e_value))
            # we might have not a stated response by now. try
            # to start one with the status code 500 or ignore an
            # raised exception if the application already started one.
            try:
                start_response('500 INTERNAL SERVER ERROR', [
                               ('Content-Type', 'text/plain')])
            except:
                pass
            yield '\n'.join(traceback)

        # wsgi applications might have a close function. If it exists
        # it *must* be called.
        if hasattr(appiter, 'close'):
            appiter.close()
```





## Python Application

`Python Application`端必须定义一个 `callable object`，`callable object` 可以是以下三者之一：

- `function/method`
- `class`
- `instance with a __call__ method`

`callable object`必须满足以下两个条件：

- 接收两个参数：environ（字典，WSGI的环境信息）、start_response（响应请求的函数, 返回HTTP status、headers给server）
- 返回一个可迭代的值（iterable）

> 重点内容：
>
> 1. `environ`和`start_response`由`http server`提供并实现
> 2. `environ`变量是包含环境变量的字典
> 3. `Python Application`内部在返回前调用`start_response`
> 4. `start_response`也是一个callable，接收两个必要的参数，`status`和`response_headers`



#### callable object代码实现

##### 1.function/method

```
def application(environ, start_response):
	# 调用服务器程序提供的 start_response，填入两个参数
	start_response('200 OK', [('Content-Type', 'text/json')])
	return []
```

##### 2.class

```
class ApplicationClass(object):
	def __init__(self, environ, start_response):
		self.environ = environ
		self.start_response = start_response
	
	def __iter__(self):
		self.start_response('200 OK', [('Content-Type', 'text/json')])
		yield "随便"
```

> 使用方式
>
> ```
> for result in ApplicationClass(environ, start_response):
>     do_somthing(result)
> ```

##### 3.instance with a __call__ method

```
class ApplicationClass(object):
	def __init__(self):
		pass
		
	def __call__(self, environ, start_response):
		start_response('200 OK', [('Content-Type', 'text/json')])
		yield "anything"
```

> 使用方式
>
> ```
> app = ApplicationClass()
> for result in app(environ, start_response):
> 	do_something(result)
> ```
>
> 





### 参考链接

[PEP-3333](<https://www.python.org/dev/peps/pep-3333/#the-application-framework-side>)

[巨佬](https://lucumr.pocoo.org/2007/5/21/getting-started-with-wsgi/)

