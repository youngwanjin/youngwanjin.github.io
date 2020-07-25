---
layout: post
title: WSGI 实现
categories: [Python]
description: WSGI 本地实现
keywords: Python, WSGI
---

# WSGI 实现

### WSGI 相关概念

> 1. **WSGI：**WSGI不是服务器，python模块，框架，API或者任何软件，只是一种规范，描述 web server 如何与web application通信的规范。server和application的规范在PEP 3333中有具体描述。要实现WSGI协议，必须同时实现web server和web application，当前运行在WSGI协议之上的web框架有Torando，Flask，Django
>
> 2. **uwsgi：**是一种通信协议，是uWSGI服务器的独占协议，用于定义传输信息的类型(type of information)，每一个uwsgi packet前4 byte为传输信息类型的描述，与WSGI协议是两种东西，据说该协议是 fcgi 协议的10倍快。
>
> 3. **uWSGI：**是一个web服务器，实现了WSGI协议、uwsgi协议、http协议等。



**WSGI** 协议主要包括 **server** 和 **application** 两部分：

- Web server/gateway: 即 HTTP Server，处理 HTTP 协议，接受用户 HTTP 请求和提供并发，调用 web application 处理业务逻辑。通常采用 C/C++ 编写，代表：apache, nginx 和 IIS。WSGI server负责从客户端接收请求，将request转发给application，将application返回的response返回给客户端
- Python Web application/framework: WSGI application接收由server转发的request，处理请求，并将处理结果返回给server。application中可以包括多个栈式的中间件(middlewares)，这些中间件需要同时实现server与application，因此可以在WSGI服务器与WSGI应用之间起调节作用：对服务器来说，中间件扮演应用程序，对应用程序来说，中间件扮演服务器



### Application/Framework

app 必须是一个可调对象（ callable object ）可以是一个方法（method）、类（class）

+ 接受两个参数：  字典(environ)，回调函数(start_response，返回 HTTP status，headers 给 web server) 

+  返回一个可迭代的值 

callable  function : app.py

```python
def application(environ, start_response):
    start_response("200 OK", [('Content-Type', 'text/plain')])
    return ['This is a python application!']
```

callable  class: app.py

```python
class ApplicationClass(object):
    def __init__(self, environ, start_response):
        self.environ = environ
        self.start_response = start_response

    def __server__(self):
        self.start_response('200 OK', [('Content-type', 'text/plain')])
        yield "Hello world!n"
```



### Server/Gateway 

Server 端主要专注 HTTP 层面的业务，接收 HTTP 请求和提供并发。每当收到 HTTP 请求，server必须调用 callable object (app)：

- 接收 HTTP 请求，并不关心 HTTP url, HTTP method 
- 为 environ 提供必要的参数，实现一个回调函数 start_response，并传给 callable object
- 调用 callable object

call application : server.py

```python
from wsgiref.simple_server import make_server


# implement callback function
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return ['This is a python application!']


# server/gateway side
if __name__ == '__main__':
    server = make_server('127.0.0.1', 8080, application)
    server.serve_forever()
```



### Middleware: Components that Play Both Sides

Middleware 处于 server/gateway 和 application/framework 之间，对 server/gateway 来说，它相当于 application/framework；对 application/framework 来说，它相当于 server/gateway。每个 middleware 实现不同的功能，我们通常根据需求选择相应的 middleware 并组合起来，实现所需的功能。比如，可在 middleware 中实现以下功能：

- 根据 url 把用户请求调度到不同的 application 中
- 负载均衡，转发用户请求
- 预处理 XSL 等相关数据
- 限制请求速率，设置白名单

middleware ： mid.py

```python
class IPBlacklistMiddleware(object):
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        ip_addr = environ.get('HTTP_HOST').split(':')[0]
        if ip_addr not in ('127.0.0.1'):
            return forbidden(start_response)

        return self.app(environ, start_response)


def forbidden(start_response):
    start_response('403 Forbidden', [('Content-Type', 'text/plain')])
    return ['Forbidden']


def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain')])
    return ['This is a python application!']


if __name__ == '__main__':
    from wsgiref.simple_server import make_server

    application = IPBlacklistMiddleware(application)
    server = make_server('127.0.0.1', 8080, application)
    server.serve_forever()

```



### Path Dispatching

对于不同的url我们希望可以返回不同的结果，这就需要添加  path dispatch ，server 端只会处理 HTTP，所以需要 application 端对 url 和 method 进行处理 

参数 environ 带有请求信息：

+ **PATH_INFO** ： HTTP  url
+  **REQUEST_METHOD**：HTTP method 

 path dispatch  ：  path_dispatch.py 

```python
from wsgiref.simple_server import make_server


class IPBlacklistMiddleware(object):
    def __init__(self, app):
        self.app = app

    def __call__(self, environ, start_response):
        ip_addr = environ.get('HTTP_HOST').split(':')[0]
        if ip_addr not in ('127.0.0.1'):
            return forbidden(start_response)

        return self.app(environ, start_response)


def dog(start_response):
    start_response('200 OK', [('Content-Type', 'text/dog')])
    return ['This is dog!']


def cat(start_response):
    start_response('200 OK', [('Content-Type', 'text/cat')])
    return ['This is cat!']


def not_found(start_response):
    start_response('404 NOT FOUND', [('Content-Type', 'text/plain')])
    return ['Not Found']


def forbidden(start_response):
    start_response('403 Forbidden', [('Content-Type', 'text/plain')])
    return ['Forbidden']


def application(environ, start_response):
    path = environ.get('PATH_INFO', '').lstrip('/')
    mapping = {'dog': dog, 'cat': cat}

    call_back = mapping[path] if path in mapping else not_found
    return call_back(start_response)


if __name__ == '__main__':
    application = IPBlacklistMiddleware(application)
    server = make_server('127.0.0.1', 8080, application)
    server.serve_forever()

```



 [Reference1 >> ]( https://www.cnblogs.com/-wenli/p/10884168.html )

