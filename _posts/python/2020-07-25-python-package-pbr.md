---
layout: post
title: python包管理和pbr
categories: [Python]
description: python 包管理工具学习
keywords: Python, pbr
---

# python包管理和pbr

### 1. 包管理工具发展历史

1. distutils  -->  setuptools --> disutils2 -->  distlib  

### 2. pbr (Python Build Reasonableness) 

> ops 项目使用的包管理工具 

1. pbr 是一个管理 python setuptools 的工具库（为了方便使用setuptools），pbr 模块读入 setup.cfg 文件的信息，并且给 setuptools 中的 setup hook 函数填写默认参数，提供更加有意义的行为。pbr只需要最小化的setup.py 文件

  使用 pbr :

```python
import setuptools

setuptools.setup(setup_requires=['pbr'], pbr=True)
```

> 1. setup_requires : setup函数在执行之前需要依赖的包的列表。这里的依赖的包的功能可以理解为生成setup的实际参数




2. setup.py 所需的实际元数据存储在setup.cfg

```python
[metadata]  # 包信息
name = neutron  # 软件包名称  
summary = OpenStack Networking   # 简介 
description-file = README.rst  # 描述文件  
author = OpenStack  # 作者
author-email = openstack-dev@lists.openstack.org  # 作者邮箱
home-page = https://docs.openstack.org/neutron/latest/ 
classifier =
    Environment :: OpenStack
    Intended Audience :: Information Technology  # 功用,平台 
    Intended Audience :: System Administrators
    License :: OSI Approved :: Apache Software License
    Operating System :: POSIX :: Linux
    Programming Language :: Python
    Programming Language :: Python :: 2
    Programming Language :: Python :: 2.7
    Programming Language :: Python :: 3
    Programming Language :: Python :: 3.5   # python 版本 

[files]  # 文件段
packages =
    neutron  #包名 递归Python包层次结构并安装,如果未指定packages，则默认为[metadata]部分中给出的name字段的值;注意会根据__init__.py进行递归扫描
data_files =
    etc/neutron =
        etc/api-paste.ini
        etc/policy.json
        etc/rootwrap.conf
    etc/neutron/rootwrap.d = etc/neutron/rootwrap.d/*
scripts =
    bin/neutron-rootwrap-xen-dom0

[entry_points]  # 模块入口
wsgi_scripts =
    neutron-api = neutron.server:get_application
console_scripts =  # 可执行脚本，在linux上/usr/local/bin，在windows上在python的Scripts中生成
    neutron-db-manage = neutron.db.migration.cli:main
    neutron-debug = neutron.debug.shell:main
    neutron-dhcp-agent = neutron.cmd.eventlet.agents.dhcp:main
    neutron-keepalived-state-change = neutron.cmd.keepalived_state_change:main
    neutron-ipset-cleanup = neutron.cmd.ipset_cleanup:main
    neutron-l3-agent = neutron.cmd.eventlet.agents.l3:main
    neutron-linuxbridge-agent = neutron.cmd.eventlet.plugins.linuxbridge_neutron_agent:main
    neutron-linuxbridge-cleanup = neutron.cmd.linuxbridge_cleanup:main
    neutron-macvtap-agent = neutron.cmd.eventlet.plugins.macvtap_neutron_agent:main
    neutron-metadata-agent = neutron.cmd.eventlet.agents.metadata:main
    neutron-netns-cleanup = neutron.cmd.netns_cleanup:main
    neutron-openvswitch-agent = neutron.cmd.eventlet.plugins.ovs_neutron_agent:main
    neutron-ovs-cleanup = neutron.cmd.ovs_cleanup:main
    neutron-pd-notify = neutron.cmd.pd_notify:main
    neutron-server = neutron.cmd.eventlet.server:main
    neutron-rpc-server = neutron.cmd.eventlet.server:main_rpc_eventlet
    neutron-rootwrap = oslo_rootwrap.cmd:main
    neutron-rootwrap-daemon = oslo_rootwrap.cmd:daemon
    neutron-usage-audit = neutron.cmd.eventlet.usage_audit:main
    neutron-metering-agent = neutron.cmd.eventlet.services.metering_agent:main
    neutron-sriov-nic-agent = neutron.cmd.eventlet.plugins.sriov_nic_neutron_agent:main
    neutron-sanity-check = neutron.cmd.sanity_check:main
neutron.core_plugins =
    ml2 = neutron.plugins.ml2.plugin:Ml2Plugin
neutron.service_plugins =
    dummy = neutron.tests.unit.dummy_plugin:DummyServicePlugin
    router = neutron.services.l3_router.l3_router_plugin:L3RouterPlugin
    metering = neutron.services.metering.metering_plugin:MeteringPlugin
    qos = neutron.services.qos.qos_plugin:QoSPlugin
    tag = neutron.services.tag.tag_plugin:TagPlugin
    flavors = neutron.services.flavors.flavors_plugin:FlavorsPlugin
    auto_allocate = neutron.services.auto_allocate.plugin:Plugin
    segments = neutron.services.segments.plugin:Plugin
    network_ip_availability = neutron.services.network_ip_availability.plugin:NetworkIPAvailabilityPlugin
    revisions = neutron.services.revisions.revision_plugin:RevisionPlugin
    timestamp = neutron.services.timestamp.timestamp_plugin:TimeStampPlugin
    trunk = neutron.services.trunk.plugin:TrunkPlugin
    loki = neutron.services.loki.loki_plugin:LokiPlugin
    logapi = neutron.services.logapi.logging_plugin:LoggingPlugin
neutron.ml2.type_drivers =
    flat = neutron.plugins.ml2.drivers.type_flat:FlatTypeDriver
    local = neutron.plugins.ml2.drivers.type_local:LocalTypeDriver
    vlan = neutron.plugins.ml2.drivers.type_vlan:VlanTypeDriver
    geneve = neutron.plugins.ml2.drivers.type_geneve:GeneveTypeDriver
    gre = neutron.plugins.ml2.drivers.type_gre:GreTypeDriver
    vxlan = neutron.plugins.ml2.drivers.type_vxlan:VxlanTypeDriver
neutron.ml2.mechanism_drivers =
    logger = neutron.tests.unit.plugins.ml2.drivers.mechanism_logger:LoggerMechanismDriver
    test = neutron.tests.unit.plugins.ml2.drivers.mechanism_test:TestMechanismDriver
    linuxbridge = neutron.plugins.ml2.drivers.linuxbridge.mech_driver.mech_linuxbridge:LinuxbridgeMechanismDriver
    macvtap = neutron.plugins.ml2.drivers.macvtap.mech_driver.mech_macvtap:MacvtapMechanismDriver
    openvswitch = neutron.plugins.ml2.drivers.openvswitch.mech_driver.mech_openvswitch:OpenvswitchMechanismDriver
    l2population = neutron.plugins.ml2.drivers.l2pop.mech_driver:L2populationMechanismDriver
    sriovnicswitch = neutron.plugins.ml2.drivers.mech_sriov.mech_driver.mech_driver:SriovNicSwitchMechanismDriver
    fake_agent = neutron.tests.unit.plugins.ml2.drivers.mech_fake_agent:FakeAgentMechanismDriver
    faulty_agent = neutron.tests.unit.plugins.ml2.drivers.mech_faulty_agent:FaultyAgentMechanismDriver
neutron.ml2.extension_drivers =
    test = neutron.tests.unit.plugins.ml2.drivers.ext_test:TestExtensionDriver
    testdb = neutron.tests.unit.plugins.ml2.drivers.ext_test:TestDBExtensionDriver
    port_security = neutron.plugins.ml2.extensions.port_security:PortSecurityExtensionDriver
    qos = neutron.plugins.ml2.extensions.qos:QosExtensionDriver
    dns = neutron.plugins.ml2.extensions.dns_integration:DNSExtensionDriverML2
    data_plane_status = neutron.plugins.ml2.extensions.data_plane_status:DataPlaneStatusExtensionDriver
    dns_domain_ports = neutron.plugins.ml2.extensions.dns_integration:DNSDomainPortsExtensionDriver
neutron.ipam_drivers =
    fake = neutron.tests.unit.ipam.fake_driver:FakeDriver
    internal = neutron.ipam.drivers.neutrondb_ipam.driver:NeutronDbPool
neutron.agent.l2.extensions =
    qos = neutron.agent.l2.extensions.qos:QosAgentExtension
    fdb = neutron.agent.l2.extensions.fdb_population:FdbPopulationAgentExtension
neutron.qos.agent_drivers =
    ovs = neutron.plugins.ml2.drivers.openvswitch.agent.extension_drivers.qos_driver:QosOVSAgentDriver
    sriov = neutron.plugins.ml2.drivers.mech_sriov.agent.extension_drivers.qos_driver:QosSRIOVAgentDriver
    linuxbridge = neutron.plugins.ml2.drivers.linuxbridge.agent.extension_drivers.qos_driver:QosLinuxbridgeAgentDriver
neutron.agent.linux.pd_drivers =
    dibbler = neutron.agent.linux.dibbler:PDDibbler
neutron.services.external_dns_drivers =
    designate = neutron.services.externaldns.drivers.designate.driver:Designate
oslo.config.opts =
    neutron = neutron.opts:list_opts
    neutron.agent = neutron.opts:list_agent_opts
    neutron.az.agent = neutron.opts:list_az_agent_opts
    neutron.base.agent = neutron.opts:list_base_agent_opts
    neutron.db = neutron.opts:list_db_opts
    neutron.dhcp.agent = neutron.opts:list_dhcp_agent_opts
    neutron.extensions = neutron.opts:list_extension_opts
    neutron.l3.agent = neutron.opts:list_l3_agent_opts
    neutron.metadata.agent = neutron.opts:list_metadata_agent_opts
    neutron.metering.agent = neutron.opts:list_metering_agent_opts
    neutron.ml2 = neutron.opts:list_ml2_conf_opts
    neutron.ml2.linuxbridge.agent = neutron.opts:list_linux_bridge_opts
    neutron.ml2.macvtap.agent = neutron.opts:list_macvtap_opts
    neutron.ml2.ovs.agent = neutron.opts:list_ovs_opts
    neutron.ml2.sriov.agent = neutron.opts:list_sriov_agent_opts
    neutron.ml2.xenapi = neutron.opts:list_xenapi_opts
    nova.auth = neutron.opts:list_auth_opts
oslo.config.opts.defaults =
    neutron = neutron.common.config:set_cors_middleware_defaults
neutron.db.alembic_migrations =
    neutron = neutron.db.migration:alembic_migrations
neutron.interface_drivers =
    ivs = neutron.agent.linux.interface:IVSInterfaceDriver
    linuxbridge = neutron.agent.linux.interface:BridgeInterfaceDriver
    null = neutron.agent.linux.interface:NullDriver
    openvswitch = neutron.agent.linux.interface:OVSInterfaceDriver
neutron.agent.firewall_drivers =
    noop = neutron.agent.firewall:NoopFirewallDriver
    iptables = neutron.agent.linux.iptables_firewall:IptablesFirewallDriver
    iptables_hybrid = neutron.agent.linux.iptables_firewall:OVSHybridIptablesFirewallDriver
    openvswitch = neutron.agent.linux.openvswitch_firewall:OVSFirewallDriver
neutron.services.metering_drivers =
    noop = neutron.services.metering.drivers.noop.noop_driver:NoopMeteringDriver
    iptables = neutron.services.metering.drivers.iptables.iptables_driver:IptablesMeteringDriver
tempest.test_plugins =
    neutron_tests = neutron.tests.tempest.plugin:NeutronTempestPlugin

[build_sphinx]  # 文档build相关信息
all_files = 1
build-dir = doc/build
source-dir = doc/source
warning-is-error = 1

[extract_messages] 
keywords = _ gettext ngettext l_ lazy_gettext
mapping_file = babel.cfg
output_file = neutron/locale/neutron.pot

[compile_catalog]
directory = neutron/locale
domain = neutron neutron-log-error neutron-log-info neutron-log-warning

[update_catalog]
domain = neutron
output_dir = neutron/locale
input_file = neutron/locale/neutron.pot

[wheel]
universal = 1

```

> 1.  [entry_points]   可以通过注册功能模块的方式，在一个包中引入不同的模块，并使其相互调用 ; 如果不使用这种方式，使用sys.path.append()引入自定义模块，将非常复杂 
> 2.  默认部署在  /usr/lib/python2.7/dist-packages/.  目录下
> 3.  三种打包格式
>    +  **tar.gz ** 格式：这个就是标准压缩格式，里面包含了项目元数据和代码，使用`python setup.py sdist`命令生成 
>    +  **.egg** 格式：本质上是一个压缩文件，只是扩展名换了，里面也包含了项目元数据以及源代码。可以通过命令`python setup.py bdist_egg` 命令生成
>    +  **.whl** 格式：这个是Wheel包，也是一个压缩文件，只是扩展名换了，里面也包含了项目元数据和代码。可以通过命令`python setup.py bdist_wheel`生成 

3. requirements.txt  指定项目的依赖包

   > 会根据git自动生成源码归档

   

   

   

   

