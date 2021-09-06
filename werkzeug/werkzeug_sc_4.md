---
title: "Werkzeug 源码解析(4)"
date: 2020-08-05T22:46:55+08:00
tags: ["flask", "python", "werkzeug"]
keywords: 
- flask
- 源码
- werkzeug
- python
- source code
- hugo
- blog
categories: ["python", "werkzeug"]
draft: false
---

### wsgi.py开讲



`wsgi.py`封装了一系列方法，现在讲几个自认为是常用的方法。



**get_current_url方法**

（1）作用：

```
获取当前请求的URL
```

（2）使用案例：

```
	>>> from werkzeug.test import create_environ
    >>> env = create_environ("/?param=foo", "http://localhost/script")
    >>> get_current_url(env)
    'http://localhost/script/?param=foo'
    
    >>> get_current_url(env, root_only=True)
    'http://localhost/script/'
    
    >>> get_current_url(env, host_only=True)
    'http://localhost/'
    
    >>> get_current_url(env, strip_querystring=True)
    'http://localhost/script/'
```

（3）代码（**感觉没有什么内容可说**）

```python
def get_current_url(
    environ,
    root_only=False,
    strip_querystring=False,
    host_only=False,
    trusted_hosts=None,
):
    
    tmp = [environ["wsgi.url_scheme"], "://", get_host(environ, trusted_hosts)]
    # tmp.append， tmp是list类型，list.append方法，cat指向tmp.append
    cat = tmp.append
    if host_only:
        return uri_to_iri(f"{''.join(tmp)}/")
        
    cat(url_quote(environ.get("SCRIPT_NAME", "").encode("latin1")).rstrip("/"))
    cat("/")
    
    if not root_only:
        cat(url_quote(environ.get("PATH_INFO", "").encode("latin1").lstrip(b"/")))
        if not strip_querystring:
            qs = get_query_string(environ)
            if qs:
                cat(f"?{qs}")
    # uri_to_iri是把URI地址转换成IRI格式（IRI包含Unicode字符，URI是ASCII字符编码）
    return uri_to_iri("".join(tmp))
```



**host_is_trusted方法**
（1）作用

```
检查主机是否可行
```

（2）使用案例

```
wsgi.py get_host方法159行有用到
```

（3）代码

```python
def host_is_trusted(hostname, trusted_list):
    
    if not hostname:
        return False

	# str -> list
    if isinstance(trusted_list, str):
        trusted_list = [trusted_list]

    def _normalize(hostname):
    	# 把port去除，只获取host
        if ":" in hostname:
            hostname = hostname.rsplit(":", 1)[0]
        return _encode_idna(hostname)

    try:
        hostname = _normalize(hostname)
    except UnicodeError:
        return False
        
    for ref in trusted_list:
        if ref.startswith("."):
            ref = ref[1:]
            suffix_match = True
        else:
            suffix_match = False
            
        try:
            ref = _normalize(ref)
        except UnicodeError:
            return False
            
        if ref == hostname:
            return True
            
        if suffix_match and hostname.endswith(b"." + ref):
            return True
    return False
```



**get_host方法**

（1）作用

```
返回运行环境主机
```

（2）使用案例

```
pass
```

（3）代码

```python
def get_host(environ, trusted_hosts=None):
    
    if "HTTP_HOST" in environ:
        # HTTP_HOST 在environ
        rv = environ["HTTP_HOST"]
        if environ["wsgi.url_scheme"] == "http" and rv.endswith(":80"):
            # 获取除:80端口的主机号
            rv = rv[:-3]
        elif environ["wsgi.url_scheme"] == "https" and rv.endswith(":443"):
            # 获取除:443端口的主机号
            rv = rv[:-4]
    else:
        rv = environ["SERVER_NAME"]
        if (environ["wsgi.url_scheme"], environ["SERVER_PORT"]) not in (
            ("https", "443"),
            ("http", "80"),
        ):
            # 获取主机+端口
            rv += f":{environ['SERVER_PORT']}"
    
    # 判断是否可信
    if trusted_hosts is not None:
        # 用到了host_is_trusted方法了
        if not host_is_trusted(rv, trusted_hosts):
            from .exceptions import SecurityError

            raise SecurityError(f'Host "{rv}" is not trusted')
            
    return rv
```



**get_content_length方法**

（1）作用

```
获取来自WSGI环境的内容长度
```

（2）使用案例

```
pass
```

（3）代码，**实在没什么好讲的**

```python
def get_content_length(environ):
   	# 块读取，直接返回None
    if environ.get("HTTP_TRANSFER_ENCODING", "") == "chunked":
        return None

    content_length = environ.get("CONTENT_LENGTH")
    if content_length is not None:
        try:
            return max(0, int(content_length))
        except (ValueError, TypeError):
            pass
```



**get_query_string方法**

（1）作用

```
获取对应的URL字段
```

（2）使用案例

```
pass
```

（3）代码

```python
def get_query_string(environ):
    # 获取environ中的QUERY_STRING对应的值，并以latin1格式进行编码
    qs = environ.get("QUERY_STRING", "").encode("latin1")
    # QUERY_STRING really should be ascii safe but some browsers
    # will send us some unicode stuff (I am looking at you IE).
    # In that case we want to urllib quote it badly.
    return url_quote(qs, safe=":&%=+$!*'(),")
```



