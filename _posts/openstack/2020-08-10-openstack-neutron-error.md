---
layout: post
title: openstack neutron 错误
categories: [OpenStack, Neutron]
description: neutron 常见错误
keywords: OpenStack, Neutron
---

### 创建 network 失败

错误描述：No tenant network is available for allocation.

```shell
[root@controller ~]# neutron net-create ywj-net
neutron CLI is deprecated and will be removed in the future. Use openstack CLI instead.
Unable to create the network. No tenant network is available for allocation.
Neutron server returns request_ids: ['req-3f772b74-2791-4150-b007-187d0433841a']

```

查看日志：

```shell
[root@controller ml2]# vim /var/log/neutron/server.log

2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation [req-3f772b74-2791-4150-b007-187d0433841a d6aa5398f9984fe6ab59fb6bcfae4b86 e487f0b6dd5f49e4abf69649c9130560 - default default] POST failed.: NoNetworkAvailable: Unable to create the network. No tenant network is available for allocation.
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation Traceback (most recent call last):
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/pecan/core.py", line 683, in __call__
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     self.invoke_controller(controller, args, kwargs, state)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/pecan/core.py", line 574, in invoke_controller
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     result = controller(*args, **kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 139, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     setattr(e, '_RETRY_EXCEEDED', True)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 220, in __exit__
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     self.force_reraise()
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 196, in force_reraise
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     six.reraise(self.type_, self.value, self.tb)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 135, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return f(*args, **kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_db/api.py", line 154, in wrapper
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     ectxt.value = e.inner_exc
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 220, in __exit__
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     self.force_reraise()
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 196, in force_reraise
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     six.reraise(self.type_, self.value, self.tb)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_db/api.py", line 142, in wrapper
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return f(*args, **kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 183, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     LOG.debug("Retry wrapper got retriable exception: %s", e)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 220, in __exit__
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     self.force_reraise()
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 196, in force_reraise
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     six.reraise(self.type_, self.value, self.tb)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 179, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return f(*dup_args, **dup_kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron/pecan_wsgi/controllers/utils.py", line 76, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return f(*args, **kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron/pecan_wsgi/controllers/resource.py", line 163, in post
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return self.create(resources)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron/pecan_wsgi/controllers/resource.py", line 181, in create
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return {key: creator(*creator_args, **creator_kwargs)}
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron/common/utils.py", line 687, in inner
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return f(self, context, *args, **kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 233, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return method(*args, **kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 139, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     setattr(e, '_RETRY_EXCEEDED', True)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 220, in __exit__
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     self.force_reraise()
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 196, in force_reraise
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     six.reraise(self.type_, self.value, self.tb)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 135, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return f(*args, **kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_db/api.py", line 154, in wrapper
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     ectxt.value = e.inner_exc
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 220, in __exit__
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     self.force_reraise()
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 196, in force_reraise
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     six.reraise(self.type_, self.value, self.tb)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_db/api.py", line 142, in wrapper
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return f(*args, **kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 183, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     LOG.debug("Retry wrapper got retriable exception: %s", e)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 220, in __exit__
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     self.force_reraise()
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 196, in force_reraise
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     six.reraise(self.type_, self.value, self.tb)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron_lib/db/api.py", line 179, in wrapped
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     return f(*dup_args, **dup_kwargs)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron/plugins/ml2/plugin.py", line 1047, in create_network
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     result, mech_context = self._create_network_db(context, network)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron/plugins/ml2/plugin.py", line 1006, in _create_network_db
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     tenant_id)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron/plugins/ml2/managers.py", line 226, in create_network_segments
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     context, filters=filters)
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation   File "/usr/lib/python2.7/site-packages/neutron/plugins/ml2/managers.py", line 315, in _allocate_tenant_net_segment
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation     raise exc.NoNetworkAvailable()
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation NoNetworkAvailable: Unable to create the network. No tenant network is available for allocation.
2020-08-10 15:09:55.585 31524 ERROR neutron.pecan_wsgi.hooks.translation 
2020-08-10 15:09:55.616 31524 INFO neutron.wsgi [req-3f772b74-2791-4150-b007-187d0433841a d6aa5398f9984fe6ab59fb6bcfae4b86 e487f0b6dd5f49e4abf69649c9130560 - default default] 192.168.8.211 "POST /v2.0/networks HTTP/1.1" status: 503  len: 369 time: 1.1541748

```

错误原因，配置文件不对：

```shell
[root@controller ml2]# vim /etc/neutron/plugins/ml2/ml2_conf.ini

# 添加如下内容：
[ml2_type_vlan]
network_vlan_ranges = external:2259:2260,service:2246:2258,management

[ml2_type_vxlan]
vni_ranges = 10001:65535

```

修改完重启 neutron-server

```shell
[root@controller ~]# systemctl restart neutron-server.service
```

