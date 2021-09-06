---
title: "Werkzeug 源码解析(3)"
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

# Werkzeug Request 源码解析

代码示例：

```
from werkzeug.wrappers import Request, Response

def application(environ, start_response):
    request = Request(environ)
    text = 'Hello %s!' % request.args.get('name', 'World')
    response = Response(text, mimetype='text/plain')
    return response(environ, start_response)

```

请注意，之后`Request`和`Response`的相关源码都会围绕`Werkzeug`给出的以上代码讲解。





# Request 类

### 文件定位

`src/werkzeug/wrappers/request.py`



### 作用

根据`Request`类的名称其实就可以知道，`Request`类是处理请求的，实际上，通过阅读相关代码，`Request`类的作用确实如此



### 分析

```Python
class Request(
    BaseRequest,
    AcceptMixin,
    ETagRequestMixin,
    UserAgentMixin,
    AuthorizationMixin,
    CORSRequestMixin,
    CommonRequestDescriptorsMixin,
):
    """Full featured request object implementing the following mixins:

    -   :class:`AcceptMixin` for accept header parsing
    -   :class:`ETagRequestMixin` for etag and cache control handling
    -   :class:`UserAgentMixin` for user agent introspection
    -   :class:`AuthorizationMixin` for http auth handling
    -   :class:`~werkzeug.wrappers.cors.CORSRequestMixin` for Cross
        Origin Resource Sharing headers
    -   :class:`CommonRequestDescriptorsMixin` for common headers

    """
```

​	根据注释可知，除了`BaseRequest`类之外，其他的`Mixin`类都是作为添加`高级方法`的类。而且`Request`类也没有`初始化`方法，所以可以将注意力从`Request`转移到`BaseRequest`类上面。





# BaseRequest 类

截取以上代码示例的代码

```
request = Request(environ)
```

调用父类`BaseRequest`的`__init__`方法

```python
    def __init__(self, environ, populate_request=True, shallow=False):
        self.environ = environ
        if populate_request and not shallow:
            self.environ["werkzeug.request"] = self
        self.shallow = shallow
```

