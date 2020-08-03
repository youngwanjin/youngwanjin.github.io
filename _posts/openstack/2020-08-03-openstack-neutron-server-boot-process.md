---
layout: post
title: OpenStack Neutron Server 启动过程
categories: [OpenStack, Neutron]
description: neutron server 启动 url路由 创建 network 流程
keywords: OpenStack, Neutron
---

### 1. neutron server 启动过程

> **pike** 版本

**neutron-server** 入口：`setup.cfg` 中 `neutron-server = neutron.cmd.eventlet.server:main`

1. `neutron\cmd\eventlet\server\__init__.py`

```python
def main():
    server.boot_server(wsgi_eventlet.eventlet_wsgi_server)


def main_rpc_eventlet():
    server.boot_server(rpc_eventlet.eventlet_rpc_server)
```

`boot_server()`  方法在  `neutron.server.__init__.py`  中，该函数主要是加载配置文件 `_init_configuration()` ；调用  `server_func()` 即： `wsgi_eventlet.eventlet_wsgi_server`



2. `neutron\server\__init__.py`:

```python
def _init_configuration():
    # the configuration will be read into the cfg.CONF global data structure
    config.init(sys.argv[1:])
    config.setup_logging()
    config.set_config_defaults()
    if not cfg.CONF.config_file:
        sys.exit(_("ERROR: Unable to find configuration file via the default"
                   " search paths (~/.neutron/, ~/, /etc/neutron/, /etc/) and"
                   " the '--config-file' option!"))


def boot_server(server_func):
    _init_configuration()
    try:
        server_func()
    except KeyboardInterrupt:
        pass
    except RuntimeError as e:
        sys.exit(_("ERROR: %s") % e)


def get_application():
    _init_configuration()
    profiler.setup('neutron-server', cfg.CONF.host)
    return config.load_paste_app('neutron')
```

`eventlet_wsgi_server()` 函数在 `neutron\server\wsgi_eventlet.py` 文件中，该方法中一部分是 WSGI 一部分是 rpc，将 neutron api 封装成 class `NeutronApiService`



3. `neutron\server\wsgi_eventlet.py`

```python
def eventlet_wsgi_server():
    neutron_api = service.serve_wsgi(service.NeutronApiService)
    start_api_and_rpc_workers(neutron_api)


def start_api_and_rpc_workers(neutron_api):
    try:
        worker_launcher = service.start_all_workers()

        pool = eventlet.GreenPool()
        api_thread = pool.spawn(neutron_api.wait)
        plugin_workers_thread = pool.spawn(worker_launcher.wait)

        # api and other workers should die together. When one dies,
        # kill the other.
        api_thread.link(lambda gt: plugin_workers_thread.kill())
        plugin_workers_thread.link(lambda gt: api_thread.kill())

        pool.waitall()
    except NotImplementedError:
        LOG.info("RPC was already started in parent process by "
                 "plugin.")

        neutron_api.wait()
```

在`serve_wsgi()` 函数中调用了 `class`  `NeutronApiService` 中的 `create`f方法来创建实例，然后使用`start`启动服务



```python
def serve_wsgi(cls):

    try:
        service = cls.create()
        service.start()
    except Exception:
        with excutils.save_and_reraise_exception():
            LOG.exception('Unrecoverable error: please check log '
                          'for details.')

    registry.notify(resources.PROCESS, events.BEFORE_SPAWN, service)
    return service
```

`NeutronApiService` 在文件 `neutron\service.py` 中，`NeutronApiService` 是继承了 `class` `WsgiService`，说明 neutron server 是一个 WSGI 服务



```python
class WsgiService(object):
    """Base class for WSGI based services.

    For each api you define, you must also define these flags:
    :<api>_listen: The address on which to listen
    :<api>_listen_port: The port on which to listen

    """

    def __init__(self, app_name):
        self.app_name = app_name
        self.wsgi_app = None

    def start(self):
        self.wsgi_app = _run_wsgi(self.app_name)

    def wait(self):
        self.wsgi_app.wait()
```

在`NeutronApiService` 只是简单的记录了服务名称，`neutron`，`start` 函数里面真真正的加载了 WSGI APP 



```python
def _run_wsgi(app_name):
    app = config.load_paste_app(app_name)
    if not app:
        LOG.error('No known API applications configured.')
        return
    return run_wsgi_app(app)
```

`_run_wsgi()` 函数中加载了 paste 定义的 WSGI 应用

```python
def run_wsgi_app(app):
    server = wsgi.Server("Neutron")
    server.start(app, cfg.CONF.bind_port, cfg.CONF.bind_host,
                 workers=_get_api_workers())
    LOG.info("Neutron service started, listening on %(host)s:%(port)s",
             {'host': cfg.CONF.bind_host, 'port': cfg.CONF.bind_port})
    return server
```

`run_wsgi_app()` 该函数会启动一个 neutron server 



```python
def load_paste_app(app_name):
    """Builds and returns a WSGI app from a paste config file.

    :param app_name: Name of the application to load
    """
    loader = wsgi.Loader(cfg.CONF)
    app = loader.load_app(app_name)
    return app
```

`wsgi.Loader()` 函数从 `neutron.conf` 中读取deploy配置文件的路径，然后根据文件来加载 app，一般路径为`etc/neutron/api-paste.ini` 然后使用  `deploy.loadapp`  来加载 app，这个 `deploy`就是`PasteDeploy`



```python
class Loader(object):
    """Used to load WSGI applications from paste configurations."""

    def __init__(self, conf):
        """Initialize the loader, and attempt to find the config.

        :param conf: Application config
        :returns: None

        """
        conf.register_opts(_options.wsgi_opts)
        self.config_path = None

        config_path = conf.api_paste_config
        if not os.path.isabs(config_path):
            self.config_path = conf.find_file(config_path)
        elif os.path.exists(config_path):
            self.config_path = config_path

        if not self.config_path:
            raise ConfigNotFound(path=config_path)

    def load_app(self, name):
        """Return the paste URLMap wrapped WSGI application.

        :param name: Name of the application to load.
        :returns: Paste URLMap object wrapping the requested application.
        :raises: PasteAppNotFound

        """
        try:
            LOG.debug("Loading app %(name)s from %(path)s",
                      {'name': name, 'path': self.config_path})
            return deploy.loadapp("config:%s" % self.config_path, name=name)
        except LookupError:
            LOG.exception("Couldn't lookup app: %s", name)
            raise PasteAppNotFound(name=name, path=self.config_path)
```

`/etc/neutron/api-paste.ini`

```ini
[composite:neutron]
use = egg:Paste#urlmap
/: neutronversions
/v2.0: neutronapi_v2_0

[composite:neutronapi_v2_0]
use = call:neutron.auth:pipeline_factory
noauth = cors request_id catch_errors extensions neutronapiapp_v2_0
keystone = cors request_id catch_errors authtoken keystonecontext extensions neutronapiapp_v2_0

[filter:request_id]
paste.filter_factory = oslo_middleware:RequestId.factory

[filter:catch_errors]
paste.filter_factory = oslo_middleware:CatchErrors.factory

[filter:cors]
paste.filter_factory = oslo_middleware.cors:filter_factory
oslo_config_project = neutron

[filter:keystonecontext]
paste.filter_factory = neutron.auth:NeutronKeystoneContext.factory

[filter:authtoken]
paste.filter_factory = keystonemiddleware.auth_token:filter_factory

[filter:extensions]
paste.filter_factory = neutron.api.extensions:plugin_aware_extension_middleware_factory

[app:neutronversions]
paste.app_factory = neutron.api.versions:Versions.factory

[app:neutronapiapp_v2_0]
paste.app_factory = neutron.api.v2.router:APIRouter.factory
```



### 2. 通过 paste 和 paste.deploy 加载 WSGI application

paste 配置文件是由一个个 section 构成的，下面对 api-paste.ini 配置文件做一定的解释：

每个 section 格式： [type:name]

type 有以下几种类型：

+ 应用：app，application
+ 过滤器：filter，filte-app
+ 管道：pipeline
+ 组合：composite

```ini
[composite:neutron]
use = egg:Paste#urlmap
/: neutronversions
/v2.0: neutronapi_v2_0
```

[composite:neutron] 是一个 sectiion ，表示这是一个组合类型配置，名字是 neutron ，request 请求进来会先通过第一个 section ，表示要将一个 http request 调度到一个或者多个 application 上

使用 key `use` 表明将使用 Paste agg 包中的 paste.urlmap 这个中间件功能，这个中间件可以根据不同的 url 请求前缀路由给不同的 WSGI 应用

`/` 配置表示将对 `/` 的访问路由到  **neutronversions** 这个 app 

`/v2.0` 配置表示将对 `/v2.0` 的访问路由交给 **neutronapi_v2_0** 这个 app 处理

> **use** 可以使用以下几种形式
>
> +  egg ：  使用一个 URL 指定的 egg 包中的对象
> + call ：  使用某个模块中的可调用对象
> + config ：   使用另外一个配置文件 

```ini
[app:neutronversions]
paste.app_factory = neutron.api.versions:Versions.factory
```

使用 key  `paste.app_factory ` 表示后面是一个工厂函数，指明了加载的模块和方法

neutron\api\versions.py：

```python
class Versions(object): 

    @classmethod
    def factory(cls, global_config, **local_config):
        if cfg.CONF.web_framework == 'pecan':
            return pecan_app.versions_factory(global_config, **local_config)
        return cls(app=None)

    @webob.dec.wsgify(RequestClass=wsgi.Request)
    def __call__(self, req):
        """Respond to a request for all Neutron API versions."""
        version_objs = [
            {
                "id": "v2.0",
                "status": "CURRENT",
            },
        ]

        if req.path != '/':
            if self.app:
                return req.get_response(self.app)
            language = req.best_match_language()
            msg = _('Unknown API version specified')
            msg = oslo_i18n.translate(msg, language)
            return webob.exc.HTTPNotFound(explanation=msg)

        builder = versions_view.get_view_builder(req)
        versions = [builder.build(version) for version in version_objs]
        response = dict(versions=versions)
        metadata = {}

        content_type = req.best_match_content_type()
        body = (wsgi.Serializer(metadata=metadata).
                serialize(response, content_type))

        response = webob.Response()
        response.content_type = content_type
        response.body = wsgi.encode_body(body)

        return response

    def __init__(self, app):
        self.app = app
```

该模块使用了工厂方法来构造对应的 WSGI 应用对象，这样对于访问`/`的 URL 就会交给 factory  方法构造一个 callable 应用对象，该对象有一个 **\_\_call\_\_**  方法，就是在处理发送方请求时调用的方法 

> 这里 `@webob.dec.wsgify(RequestClass=wsgi.Request)` 使用了装饰器，**webob** 

```ini
[composite:neutronapi_v2_0]
use = call:neutron.auth:pipeline_factory
noauth = cors request_id catch_errors extensions neutronapiapp_v2_0
keystone = cors request_id catch_errors authtoken keystonecontext extensions neutronapiapp_v2_0
```

这个 section 用来处理访问 URL `/v2.0` 访问的 application，`use = call:neutron.auth:pipeline_factory`,根据 value 判断这是一个 call 类型的值，说明使用一个 callable 对象来构造 WSGI 应用

neutron\auth.py：

```python
class NeutronKeystoneContext(base.ConfigurableMiddleware):
    """Make a request context from keystone headers."""

    @webob.dec.wsgify
    def __call__(self, req):
        ctx = context.Context.from_environ(req.environ)

        if not ctx.user_id:
            LOG.debug("X_USER_ID is not found in request")
            return webob.exc.HTTPUnauthorized()

        # Inject the context...
        req.environ['neutron.context'] = ctx

        return self.application


def pipeline_factory(loader, global_conf, **local_conf):
    """Create a paste pipeline based on the 'auth_strategy' config option."""
    pipeline = local_conf[cfg.CONF.auth_strategy]
    pipeline = pipeline.split()
    filters = [loader.get_filter(n) for n in pipeline[:-1]]
    app = loader.get_app(pipeline[-1])
    filters.reverse()
    for filter in filters:
        app = filter(app)
    return app
```

该方法会根据不同的配置选择不同的授权策略，`noauth` 和 `keystone` 然后读取不同的 filter 和 app， `cors request_id catch_errors authtoken keystonecontext extensions neutronapiapp_v2_0 `这是一个 filter 列表，一般默认为 `keystone` 所以一般对于访问 URL 为 `v2.0` 都会对 `neutronapiapp_v2_0 `  这个 app 经过所有的 filter 后返回

```ini
# cors request_id catch_errors authtoken keystonecontext extensions neutronapiapp_v2_0
[filter:cors]
paste.filter_factory = oslo_middleware.cors:filter_factory
oslo_config_project = neutron

[filter:request_id]
paste.filter_factory = oslo_middleware:RequestId.factory

[filter:catch_errors]
paste.filter_factory = oslo_middleware:CatchErrors.factory

[filter:authtoken]
paste.filter_factory = keystonemiddleware.auth_token:filter_factory

[filter:keystonecontext]
paste.filter_factory = neutron.auth:NeutronKeystoneContext.factory

[filter:extensions]
paste.filter_factory = neutron.api.extensions:plugin_aware_extension_middleware_factory

[app:neutronapiapp_v2_0]
paste.app_factory = neutron.api.v2.router:APIRouter.factory
```

> **filter** 和 **app** 的 key 有两种格式 `paste.filter_factory` 和 `paste.app_factory` , filter_factory 和 app_factory 是非常类似的，filter_factory  返回 filter 而不是 WSGI app ，返回的 filter 必须是 callable , 接受 WSGI app 作为唯一参数，即：返回处理过的 application。

[filter:extensions]  -->  neutron\api\extensions.py:

```python
def plugin_aware_extension_middleware_factory(global_config, **local_config):
    """Paste factory."""
    def _factory(app):
        ext_mgr = PluginAwareExtensionManager.get_instance()
        return ExtensionMiddleware(app, ext_mgr=ext_mgr)
    return _factory
```

可以发现 filter 接受一个 app 返回一个处理后的 app 

`filter_factory` 是最常见 factory ，接受配置参数，用来返回一个 WSGI 应用，全局配置参数通过字典形式传入，局部配置参数通过 关键字参数 传入



### 3. URL 如何路由到具体的 applictaion

对于访问 URL 为 `/v2.0` 为前缀的请求，最终都交由 `neutron.api.v2.router:APIRouter.factory` 来负责实例化对象，这个工厂又是如何将不同的具体的请求发送给不同的 application 的呢？这里就需要介绍 **routers** 和 **pecan** 这两个库了

paste.app_factory -->  neutron\api\v2\router.py : 

```python
class APIRouter(base_wsgi.Router):

    @classmethod
    def factory(cls, global_config, **local_config):
        if cfg.CONF.web_framework == 'pecan':
            return pecan_app.v2_factory(global_config, **local_config)
        return cls(**local_config)

    def __init__(self, **local_config):
        mapper = routes_mapper.Mapper()
        manager.init()
        plugin = directory.get_plugin()
        ext_mgr = extensions.PluginAwareExtensionManager.get_instance()
        ext_mgr.extend_resources("2.0", attributes.RESOURCE_ATTRIBUTE_MAP)

        col_kwargs = dict(collection_actions=COLLECTION_ACTIONS,
                          member_actions=MEMBER_ACTIONS)

        def _map_resource(collection, resource, params, parent=None):
            allow_bulk = cfg.CONF.allow_bulk
            controller = base.create_resource(
                collection, resource, plugin, params, allow_bulk=allow_bulk,
                parent=parent, allow_pagination=True,
                allow_sorting=True)
            path_prefix = None
            if parent:
                path_prefix = "/%s/{%s_id}/%s" % (parent['collection_name'],
                                                  parent['member_name'],
                                                  collection)
            mapper_kwargs = dict(controller=controller,
                                 requirements=REQUIREMENTS,
                                 path_prefix=path_prefix,
                                 **col_kwargs)
            return mapper.collection(collection, resource,
                                     **mapper_kwargs)

        mapper.connect('index', '/', controller=Index(RESOURCES))
        for resource in RESOURCES:
            _map_resource(RESOURCES[resource], resource,
                          attributes.RESOURCE_ATTRIBUTE_MAP.get(
                              RESOURCES[resource], dict()))
            resource_registry.register_resource_by_name(resource)

        for resource in SUB_RESOURCES:
            _map_resource(SUB_RESOURCES[resource]['collection_name'], resource,
                          attributes.RESOURCE_ATTRIBUTE_MAP.get(
                              SUB_RESOURCES[resource]['collection_name'],
                              dict()),
                          SUB_RESOURCES[resource]['parent'])

        # Certain policy checks require that the extensions are loaded
        # and the RESOURCE_ATTRIBUTE_MAP populated before they can be
        # properly initialized. This can only be claimed with certainty
        # once this point in the code has been reached. In the event
        # that the policies have been initialized before this point,
        # calling reset will cause the next policy check to
        # re-initialize with all of the required data in place.
        policy.reset()
        super(APIRouter, self).__init__(mapper)
```

可以看到如果配置文件使用 `pecan` 则会使用 `v2_factory()` 函数，即：pecan 库来实现 application 的路由，默认则会使用 `routers` 库实现具体的 applicatiion 进行路由 ，`pecan` 暂时先不看，对于如何使用 `routers` 进行路由需要看 `__init__()` 中具体是如何实现的

在`__init__()` 中可以看到构造 `mapper` 映射主要是通过内部 `_map_resource` 函数实现，通过遍历 `RESOURCES` 和 `SUB_RESOURCES` 来添加 `routers` 对于不同的 URL 请求，通过 `base.create_resource()` 来创建相应的 `controller` 

neutron\api\v2\base.py : 

```python
def create_resource(collection, resource, plugin, params, allow_bulk=False,
                    member_actions=None, parent=None, allow_pagination=False,
                    allow_sorting=False):
    controller = Controller(plugin, collection, resource, params, allow_bulk,
                            member_actions=member_actions, parent=parent,
                            allow_pagination=allow_pagination,
                            allow_sorting=allow_sorting)

    return wsgi_resource.Resource(controller, FAULT_MAP)
```

该方法动态的处理不同的 URL 所需的 controller ，`collection` 和 `member_actions` 决定了 API 所支持的动作

```python
COLLECTION_ACTIONS = ['index', 'create']
MEMBER_ACTIONS = ['show', 'update', 'delete']
```



最后介绍一下 webob，该库提供装饰器来将我们的函数包装成 WSGI 应用

neutron\api\versions.py ：（对于 `/` 的路由处理）

```python
	@webob.dec.wsgify(RequestClass=wsgi.Request)
    def __call__(self, req):
        """Respond to a request for all Neutron API versions."""
        version_objs = [
            {
                "id": "v2.0",
                "status": "CURRENT",
            },
        ]

        if req.path != '/':
            if self.app:
                return req.get_response(self.app)
            language = req.best_match_language()
            msg = _('Unknown API version specified')
            msg = oslo_i18n.translate(msg, language)
            return webob.exc.HTTPNotFound(explanation=msg)

        builder = versions_view.get_view_builder(req)
        versions = [builder.build(version) for version in version_objs]
        response = dict(versions=versions)
        metadata = {}

        content_type = req.best_match_content_type()
        body = (wsgi.Serializer(metadata=metadata).
                serialize(response, content_type))

        response = webob.Response()
        response.content_type = content_type
        response.body = wsgi.encode_body(body)

        return response
```

`@webob.dec.wsgify(RequestClass=wsgi.Request)` 其作用是将函数封装成符合 WSGI 规范的 application ，这样调用 `__call__` 方法时，就会按照如下方式调用：

app = obj(environ, start_response) ，obj 是一个 Versions 对象



#### 4. 创建 network  流程

上面讲到会根据不同的 URL 动态的创建 controller ，具体的创建由 `Resource` 实现

neutron\api\v2\resource.py ：

```python
def Resource(controller, faults=None, deserializers=None, serializers=None,
             action_status=None):
    """Represents an API entity resource and the associated serialization and
    deserialization logic
    """
    default_deserializers = {'application/json': wsgi.JSONDeserializer()}
    default_serializers = {'application/json': wsgi.JSONDictSerializer()}
    format_types = {'json': 'application/json'}
    action_status = action_status or dict(create=201, delete=204)

    default_deserializers.update(deserializers or {})
    default_serializers.update(serializers or {})

    deserializers = default_deserializers
    serializers = default_serializers
    faults = faults or {}

    @webob.dec.wsgify(RequestClass=Request)
    def resource(request):
        route_args = request.environ.get('wsgiorg.routing_args')
        if route_args:
            args = route_args[1].copy()
        else:
            args = {}

        # NOTE(jkoelker) by now the controller is already found, remove
        #                it from the args if it is in the matchdict
        args.pop('controller', None)
        fmt = args.pop('format', None)
        action = args.pop('action', None)
        content_type = format_types.get(fmt,
                                        request.best_match_content_type())
        language = request.best_match_language()
        deserializer = deserializers.get(content_type)
        serializer = serializers.get(content_type)

        try:
            if request.body:
                args['body'] = deserializer.deserialize(request.body)['body']

            # Routes library is dumb and cuts off everything after last dot (.)
            # as format. At the same time, it doesn't enforce format suffix,
            # which combined makes it impossible to pass a 'id' with dots
            # included (the last section after the last dot is lost). This is
            # important for some API extensions like tags where the id is
            # really a tag name that can contain special characters.
            #
            # To work around the Routes behaviour, we will attach the suffix
            # back to id if it's not one of supported formats (atm json only).
            # This of course won't work for the corner case of a tag name that
            # actually ends with '.json', but there seems to be no better way
            # to tackle it without breaking API backwards compatibility.
            if fmt is not None and fmt not in format_types:
                args['id'] = '.'.join([args['id'], fmt])

            revision_number = api_common.check_request_for_revision_constraint(
                request)
            if revision_number is not None:
                request.context.set_transaction_constraint(
                    controller._collection, args['id'], revision_number)

            method = getattr(controller, action)
            result = method(request=request, **args)
        except Exception as e:
            mapped_exc = api_common.convert_exception_to_http_exc(e, faults,
                                                                  language)
            if hasattr(mapped_exc, 'code') and 400 <= mapped_exc.code < 500:
                LOG.info('%(action)s failed (client error): %(exc)s',
                         {'action': action, 'exc': mapped_exc})
            else:
                LOG.exception('%(action)s failed: %(details)s',
                              {
                                  'action': action,
                                  'details': utils.extract_exc_details(e),
                              }
                              )
            raise mapped_exc

        status = action_status.get(action, 200)
        body = serializer.serialize(result)
        # NOTE(jkoelker) Comply with RFC2616 section 9.7
        if status == 204:
            content_type = ''
            body = None

        return webob.Response(request=request, status=status,
                              content_type=content_type,
                              body=body)
    # NOTE(blogan): this is something that is needed for the transition to
    # pecan.  This will allow the pecan code to have a handle on the controller
    # for an extension so it can reuse the code instead of forcing every
    # extension to rewrite the code for use with pecan.
    setattr(resource, 'controller', controller)
    setattr(resource, 'action_status', action_status)
    return resource
```

所有的请求都会先交给 `resource` 进行处理，反序列化和获取请求参数之后，再交由 controller 处理

`action = args.pop('action', None)`

`method = getattr(controller, action)`

`result = method(request=request, **args)`

对于具体的请求 `/networks` `/subnets`  `/subnetpools` `/ports` 最终都会交给对应的 action 函数处理，以 ` create_network `  为例：

生成对应的 controller 后，会调用 controller  的 `create`方法

neutron\api\v2\base.py

```python
    def create(self, request, body=None, **kwargs):
        self._notifier.info(request.context,
                            self._resource + '.create.start',
                            body)
        return self._create(request, body, **kwargs)
```

然后调用 `_create` 函数：

neutron\api\v2\base.py

```python
	@db_api.retry_db_errors
    def _create(self, request, body, **kwargs):
        """Creates a new instance of the requested entity."""
        parent_id = kwargs.get(self._parent_id_name)
        body = Controller.prepare_request_body(request.context,
                                               body, True,
                                               self._resource, self._attr_info,
                                               allow_bulk=self._allow_bulk)
        action = self._plugin_handlers[self.CREATE]
        # Check authz
        if self._collection in body:
            # Have to account for bulk create
            items = body[self._collection]
        else:
            items = [body]
        # Ensure policy engine is initialized
        policy.init()
        # Store requested resource amounts grouping them by tenant
        # This won't work with multiple resources. However because of the
        # current structure of this controller there will hardly be more than
        # one resource for which reservations are being made
        request_deltas = collections.defaultdict(int)
        for item in items:
            self._validate_network_tenant_ownership(request,
                                                    item[self._resource])
            policy.enforce(request.context,
                           action,
                           item[self._resource],
                           pluralized=self._collection)
            if 'tenant_id' not in item[self._resource]:
                # no tenant_id - no quota check
                continue
            tenant_id = item[self._resource]['tenant_id']
            request_deltas[tenant_id] += 1
        # Quota enforcement
        reservations = []
        try:
            for (tenant, delta) in request_deltas.items():
                reservation = quota.QUOTAS.make_reservation(
                    request.context,
                    tenant,
                    {self._resource: delta},
                    self._plugin)
                reservations.append(reservation)
        except n_exc.QuotaResourceUnknown as e:
            # We don't want to quota this resource
            LOG.debug(e)

        def notify(create_result):
            # Ensure usage trackers for all resources affected by this API
            # operation are marked as dirty
            with db_api.context_manager.writer.using(request.context):
                # Commit the reservation(s)
                for reservation in reservations:
                    quota.QUOTAS.commit_reservation(
                        request.context, reservation.reservation_id)
                resource_registry.set_resources_dirty(request.context)

            notifier_method = self._resource + '.create.end'
            self._notifier.info(request.context,
                                notifier_method,
                                create_result)
            registry.notify(self._resource, events.BEFORE_RESPONSE, self,
                            context=request.context, data=create_result,
                            method_name=notifier_method,
                            collection=self._collection,
                            action=action, original={})
            return create_result

        def do_create(body, bulk=False, emulated=False):
            kwargs = {self._parent_id_name: parent_id} if parent_id else {}
            if bulk and not emulated:
                obj_creator = getattr(self._plugin, "%s_bulk" % action)
            else:
                obj_creator = getattr(self._plugin, action)
            try:
                if emulated:
                    return self._emulate_bulk_create(obj_creator, request,
                                                     body, parent_id)
                else:
                    if self._collection in body:
                        # This is weird but fixing it requires changes to the
                        # plugin interface
                        kwargs.update({self._collection: body})
                    else:
                        kwargs.update({self._resource: body})
                    return obj_creator(request.context, **kwargs)
            except Exception:
                # In case of failure the plugin will always raise an
                # exception. Cancel the reservation
                with excutils.save_and_reraise_exception():
                    for reservation in reservations:
                        quota.QUOTAS.cancel_reservation(
                            request.context, reservation.reservation_id)

        if self._collection in body and self._native_bulk:
            # plugin does atomic bulk create operations
            objs = do_create(body, bulk=True)
            # Use first element of list to discriminate attributes which
            # should be removed because of authZ policies
            fields_to_strip = self._exclude_attributes_by_policy(
                request.context, objs[0])
            return notify({self._collection: [self._filter_attributes(
                obj, fields_to_strip=fields_to_strip)
                for obj in objs]})
        else:
            if self._collection in body:
                # Emulate atomic bulk behavior
                objs = do_create(body, bulk=True, emulated=True)
                return notify({self._collection: objs})
            else:
                obj = do_create(body)
                return notify({self._resource: self._view(request.context,
                                                          obj)})
```

在该函数内部会从 plugin 里面取出操作映射的 action ,`action = self._plugin_handlers[self.CREATE]`  这个映射是在 controller 的构造函数中创建的

neutron\api\v2\base.py : 

```python
self._plugin_handlers = {
            self.LIST: 'get%s_%s' % (parent_part, self._collection),
            self.SHOW: 'get%s_%s' % (parent_part, self._resource)
        }
 for action in [self.CREATE, self.UPDATE, self.DELETE]:
      self._plugin_handlers[action] = '%s%s_%s' % (action, parent_part,
                                                         self._resource)
```

当 `self._resource` 为 `network` `port` 这些 resource 时，对应的 create 方法就是 create_network，create_port，从源码可以看到，在 _create() 方法中，调用了  do_create () 方法

neutron\api\v2\base.py : 

```python
        def do_create(body, bulk=False, emulated=False):
            kwargs = {self._parent_id_name: parent_id} if parent_id else {}
            if bulk and not emulated:
                obj_creator = getattr(self._plugin, "%s_bulk" % action)
            else:
                obj_creator = getattr(self._plugin, action)
            try:
                if emulated:
                    return self._emulate_bulk_create(obj_creator, request,
                                                     body, parent_id)
                else:
                    if self._collection in body:
                        # This is weird but fixing it requires changes to the
                        # plugin interface
                        kwargs.update({self._collection: body})
                    else:
                        kwargs.update({self._resource: body})
                    return obj_creator(request.context, **kwargs)
            except Exception:
                # In case of failure the plugin will always raise an
                # exception. Cancel the reservation
                with excutils.save_and_reraise_exception():
                    for reservation in reservations:
                        quota.QUOTAS.cancel_reservation(
                            request.context, reservation.reservation_id)
```

会从 `self._plugin` 里面获取对应的 action，这个 `_plugin` 就是核心插件 Ml2Plugin，因此所有的核心操作最终都会交给 Ml2Plugin 里面对应的 create_network ，create_port 等方法执行具体逻辑，即：核心资源的创建、删除 操作最终都是交给 Ml2Plugin 实现

neutron\plugins\ml2\plugin.py ：

```python
    def __init__(self):
        # First load drivers, then initialize DB, then initialize drivers
        self.type_manager = managers.TypeManager()
        self.extension_manager = managers.ExtensionManager()
        self.mechanism_manager = managers.MechanismManager()
        super(Ml2Plugin, self).__init__()
        self.type_manager.initialize()
        self.extension_manager.initialize()
        self.mechanism_manager.initialize()
        self._setup_dhcp()
        self._start_rpc_notifiers()
        self.add_agent_status_check_worker(self.agent_health_check)
        self.add_workers(self.mechanism_manager.get_workers())
        self._verify_service_plugins_requirements()
        LOG.info("Modular L2 Plugin initialization complete")
```

`plugin` 中首先初始化了 `type_manager` `extension_manager` `mechanism_manager` 这几个管理器，`type_manager` 和 `mechanism_manager`  分别用来管理 **type** 和 **mechanism** , 不同的网络拓扑类型对应着不同的 **Type Driver** 但是网络的实现机制对应着 **Mechanism Driver** 。这两个管理器都是通过 stevedor 进行管理的，这样就可以像使用标准库一样来使用  Type 和 Mechanism Driver 

 其中 Type 插件的加载会以 `neutron.ml2.type_drivers` 作为命名空间，Mechanism 插件的加载会以 `neutron.ml2.mechanism_drivers` 作为命名空间，实际上 Ml2Plugin 的不同操作会交给不同的 type ，mechanism 插件处理

neutron\plugins\ml2\plugin.py：

```python
    def create_network(self, context, network):
        self._before_create_network(context, network)
        result, mech_context = self._create_network_db(context, network)
        return self._after_create_network(context, result, mech_context)
    
     def _after_create_network(self, context, result, mech_context):
        kwargs = {'context': context, 'network': result}
        registry.notify(resources.NETWORK, events.AFTER_CREATE, self, **kwargs)
        try:
            self.mechanism_manager.create_network_postcommit(mech_context)
        except ml2_exc.MechanismDriverError:
            with excutils.save_and_reraise_exception():
                LOG.error("mechanism_manager.create_network_postcommit "
                          "failed, deleting network '%s'", result['id'])
                self.delete_network(context, result['id'])

        return result
```

通过源码可以看到，创建网络最终会交给 `mechanism_manager` 处理

APIRouter 流程 resource --> Controller --> Ml2Plugin --> Type , Mechanism 这样我们只用实现具体的 type 和 mechanism Driver 



[Reference1>> ]( https://blog.csdn.net/happyanger6/article/details/54586463 )

[Reference2>> ]( https://blog.csdn.net/happyanger6/article/details/54518491 )

[Reference3>> ]( https://routes.readthedocs.io/en/latest/ )