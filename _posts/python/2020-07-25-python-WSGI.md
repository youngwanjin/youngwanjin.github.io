---
layout: post
title: WSGI 基础介绍
categories: [Python]
description: python WSGI 学习
keywords: Python,WSGI
---

# WSGI

WSGI是一个**规范**，定义了Web服务器如何与Python应用程序进行交互，使得使用Python写的Web应用程序可以和Web服务器对接起来



WSGI 相当于是 web 服务器和python应用程序之间的桥梁（servlet）,其存在的目的有两个：

1. 让 web 服务器知道如何调用python应用程序，并把用户的请求告诉应用程序
2. 让Python应用程序知道用户的具体请求是什么，以及如何返回结果给Web服务器

> **WSGI** 接口有服务端和应用端两部分，服务端也可以叫网关端，应用端也叫框架端。服务端调用一个由应用端提供的可调用对象。如何提供这个对象，由服务端决定。例如某些服务器或者网关需要应用的部署者写一段脚本，以创建服务器或者网关的实例，并且为这个实例提供一个应用实例。另一些服务器或者网关则可能使用配置文件或其他方法以指定应用实例应该从哪里导入或获取。



### HTTP请求是如何到应用程序的?

 ![img](/images/python/python-web-server.png) 

**1、两级结构** 在这种结构里，uWSGI作为服务器，它用到了HTTP协议以及wsgi协议，flask应用作为application，实现了wsgi协议。当有客户端发来请求，uWSGI接受请求，调用flask app得到相应，之后响应给客户端。通常来说，Flask等web框架会自己附带一个wsgi服务器(这就是flask应用可以直接启动的原因)，但是这只是在开发阶段用到的，在生产环境是不够用的，所以用到了uwsgi这个性能高的wsgi服务器。

**2、三级结构** 这种结构里，uWSGI作为中间件，它用到了uwsgi协议(与nginx通信)，wsgi协议(调用Flask app)。当有客户端发来请求，nginx先做处理(静态资源是nginx的强项)，无法处理的请求(uWSGI)，最后的相应也是nginx回复给客户端的。 多了一层反向代理有什么好处？

提高web server性能(uWSGI处理静态资源不如nginx；nginx会在收到一个完整的http请求后再转发给wWSGI)

nginx可以做负载均衡(前提是有多个服务器)，保护了实际的web服务器(客户端是和nginx交互而不是uWSGI)



### WSGI 中的角色

#### 1. server （gateway）

​     web 服务器端

#### 2. application

​     应用程序端

> 1. 必须是一个可调用的对象
> 2. 接收两个必选参数environ、start_response
> 3. 返回值必须是可迭代对象，用来表示http body



server端会先收到用户的请求，然后会根据规范的要求调用application端，调用的结果会被封装成HTTP响应后再发送给客户端

 ![preview](/images/python/wsgi-server-app.png) 



### server 如何调用 application

每个**application**的入口只有一个，也就是所有的客户端请求都同一个入口进入到应用程序

接下来，**server**端需要知道去哪里找**application**的入口。这个需要在server端指定一个python模块，也就是python应用中的一个文件，并且这个模块中需要包含一个名称为**application**的可调用对象（函数和类都可以），这个**application**对象就是这个应用程序的唯一入口了。WSGI还定义了**application**对象的形式：

```python
def simple_app(environ, start_response):
      pass
```

> environ  和 start_response 是server调用application时传递的两个参数

当server按照WSGI的规范调用了application之后，application就可以开始处理客户端的请求了，处理请求之后，application对象需要返回处理结果给server端。处理请求和返回结果这两个事情，都和server调用application对象时传递的两个参数有关

#### 1. environ

environ参数是一个Python的字典，里面存放了所有和客户端相关的信息，这样application对象就能知道客户端请求的资源是什么，请求中带了什么数据等，environ中常用的成员：

首先是CGI规范中要求的变量：

- **REQUEST_METHOD**： 请求方法，是个字符串，'GET', 'POST','PUT','DELETE' 等
- **SCRIPT_NAME**： HTTP请求的path中的用于查找到application对象的部分，比如Web服务器可以根据path的一部分来决定请求由哪个virtual host处理
- **PATH_INFO**： HTTP请求的path中剩余的部分，也就是application要处理的部分
- **QUERY_STRING**： HTTP请求中的查询字符串，URL中**?**后面的内容
- **CONTENT_TYPE**： HTTP headers中的content-type内容
- **CONTENT_LENGTH**： HTTP headers中的content-length内容
- **SERVER_NAME**和**SERVER_PORT**： 服务器名和端口，这两个值和前面的SCRIPT_NAME, PATH_INFO拼起来可以得到完整的URL路径
- **SERVER_PROTOCOL**： HTTP协议版本，HTTP/1.0或者HTTP/1.1
- **HTTP_**：  和HTTP请求中的headers对应。

WSGI规范中还要求environ包含下列成员：

- **wsgi.version**：表示WSGI版本，一个元组(1, 0)，表示版本1.0
- **wsgi.url_scheme**：http或者https
- **wsgi.input**：一个类文件的输入流，application可以通过这个获取HTTP request body
- **wsgi.errors**：一个输出流，当应用程序出错时，可以将错误信息写入这里
- **wsgi.multithread**：当application对象可能被多个线程同时调用时，这个值需要为True
- **wsgi.multiprocess**：当application对象可能被多个进程同时调用时，这个值需要为True
- **wsgi.run_once**：当server期望application对象在进程的生命周期内只被调用一次时，该值为True

#### 2. start_response

start_response是一个可调用对象，接收两个必选参数和一个可选参数：

- **status**: 一个字符串，表示HTTP响应状态字符串
- **response_headers**: 一个列表，包含有如下形式的元组：(header_name, header_value)，用来表示HTTP响应的headers
- **exc_info**（可选）: 用于出错时，server需要返回给浏览器的信息

当application对象根据environ参数的内容执行完业务逻辑后，就需要返回结果给server端。我们知道HTTP的响应需要包含status，headers和body，所以在application对象将body作为返回值return之前，需要先调用`start_response()`，将status和headers的内容返回给server，这同时也是告诉server，application对象要开始返回body了

#### 3. application对象的返回值

application对象的返回值用于为HTTP响应提供body，如果没有body，那么可以返回None。如果有body的化，那么需要返回一个可迭代的对象。server端通过遍历这个可迭代对象可以获得body的全部内容。



### WSGI中间件

**WSGI Middleware**（中间件）也是WSGI规范的一部分。我们已经说明了WSGI的两个角色：server和application。middleware是一种运行在server和application中间的应用（一般都是Python应用）。middleware同时具备server和application角色，对于server来说，它是一个application；对于application来说，它是一个server。middleware并不修改server端和application端的规范，只是同时实现了这两个角色的功能而已。



middleware是如何工作的:

 ![preview](/images/python/wsgi-middleware.png) 

middleware 的工作流程：

1. Server收到客户端的HTTP请求后，生成了`environ_s`，并且已经定义了`start_response_s`
2. Server调用Middleware的application对象，传递的参数是`environ_s`和`start_response_s`
3. Middleware会根据`environ`执行业务逻辑，生成`environ_m`，并且已经定义了`start_response_m`
4. Middleware决定调用Application的application对象，传递参数是`environ_m`和`start_response_m`Application的application对象处理完成后，会调用`start_response_m`并且返回结果给Middleware，存放在`result_m`中
5. Middleware处理`result_m`，然后生成`result_s`，接着调用`start_response_s`，并返回结果`result_s`给Server端。Server端获取到result_s后就可以发送结果给客户端了

从上面的流程可以看出middleware应用的几个特点：

1. Server认为middleware是一个application
2. Application认为middleware是一个server
3. Middleware可以有多层

Middleware能过处理所有经过的request和response，所以要做什么都可以，没有限制。比如可以检查request是否有非法内容，检查response是否有非法内容，为request加上特定的HTTP header等，这些都是可以的



### WSGI的实现和部署

要使用WSGI，需要分别实现server角色和application角色。

Application端的实现一般是由Python的各种框架来实现的，比如Django, web.py等，一般开发者不需要关心WSGI的实现，框架会提供接口让开发者获取HTTP请求的内容以及发送HTTP响应。

Server端的实现会比较复杂一点，这个主要是因为软件架构的原因。一般常用的Web服务器，如Apache和nginx，都不会内置WSGI的支持，而是通过扩展来完成。比如Apache服务器，会通过扩展模块mod_wsgi来支持WSGI。Apache和mod_wsgi之间通过程序内部接口传递信息，mod_wsgi会实现WSGI的server端、进程管理以及对application的调用。Nginx上一般是用proxy的方式，用nginx的协议将请求封装好，发送给应用服务器，比如uWSGI，应用服务器会实现WSGI的服务端、进程管理以及对application的调用。



[Reference1]( https://segmentfault.com/a/1190000003069785 )

[Reference2]( https://juejin.im/post/5cff300a6fb9a07ef06f8a43 )

