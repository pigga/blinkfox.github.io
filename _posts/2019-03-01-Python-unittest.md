---
title: OpenStack的Nova内unittest的几种方式
date: 2019-03-01 10:52 +0800
layout: post
Author: Yannis
permalink: /blog/2019/03/01/Python-unittest.html
categories:
  - Python
tags:
  - Python
  - 单元测试
  - mock
  - unittest
  - mox
  - OpenStack
  - Nova
---

最近在Python内使用单元测试时,发现了大概三种实现.在这里记录一下,以待后续使用
[mock官网地址](https://docs.python.org/3/library/unittest.mock.html)
###### 1. [mox]使用StubOutWithMock
```
from mox3 import mox

    def test_create(self):
        req = fakes.HTTPRequest.blank('/fake/xen')
        req.method = 'POST'
        req.headers["content-type"] = "application/json"
        req.environ['nova.context'] = context.RequestContext(
            user_id='fake_user_id', project_id='fake_project_id',
            is_admin=True,
            roles=['admin', '_members_'])
        body_json = {"xen": {"address": "10.160.60.254",
                             "username": "root", "password": "tuscloud",
                             "pool_name": "fake_pool_name",
                             "physical_host": "fake_host"}}
        self.mox.StubOutWithMock(compute_api.API, 'create_xen_pool')
        compute_api.API.create_xen_pool(mox.IgnoreArg(), mox.IgnoreArg())\
            .AndReturn(None)
        self.mox.ReplayAll()
        self.controller.create(req, body_json)
```
该方法使用`self.mox.StubOutWithMock`将`compute_api.API`内的`create_xen_pool`进行mock,后续使用
```
compute_api.API.create_xen_pool(mox.IgnoreArg(), mox.IgnoreArg())\
            .AndReturn(None)
```
mock了该方法的返回值.<br/>
并使用了
```
self.mox.ReplayAll()
```
将所有的mock生效.<br/>
_注意_: 
```
compute_api.API.create_xen_pool(mox.IgnoreArg(), mox.IgnoreArg())\
            .AndReturn(None)
```
以上代码的`create_xen_pool`方法内的参数个数与实际的参数个数一致.<br/>

**该种方法存在问题**:若想测试的方法内,同时调用了多次同一个方法.则无法使用该方式.`self.mox.StubOutWithMock(compute_api.API, 'create_xen_pool')`mock出来的方法,只能被一次有效调用.
###### 2. [stub]使用stubs
```
    def test_disable_host(self):
        def disable_xen_host(self, context, physical_host, host_uuid):
            return None
        req = fakes.HTTPRequest.blank(
            '/fake/xen/disable_host')
        req.method = 'POST'
        req.headers["content-type"] = "application/json"
        req.environ['nova.context'] = context.RequestContext(
            user_id='fake_user_id', project_id='fake_project_id',
            is_admin=True,
            roles=['admin', '_members_'])
        self.stubs.Set(objects.XenHost, 'get_host_by_filter',
                       self.fill_mock_host_enabled)
        self.stubs.Set(compute_api.API, 'disable_xen_host',
                       disable_xen_host)
        body_json = {"xen_host": "fake_host_id"}
        self.controller.disable_host(req, body_json)
        
    
    def fill_mock_host_enabled(self, context, host_uuid):
        _mock_host = objects.XenHost()
        _mock_host.id = 4
        _mock_host.uuid = 'fake_host_id'
        _mock_host.name = 'fake_host_name'
        _mock_host.description = 'for test'
        _mock_host.address = '10.160.59.112'
        _mock_host.username = 'root'
        _mock_host.password = 'tuscloud'
        _mock_host.physical_host = 'sk-xen'
        _mock_host.is_master = 1
        _mock_host.master_uuid = 'c6d72541-bcb6-20f5-f1f0-64c66d24df91'
        _mock_host.pool_uuid = 'fake_pool_id'
        _mock_host.enabled = 1
        _mock_host.status = 'active'
        _mock_host.power_state = 'on'
        return _mock_host

```
如你所见,我们想要对`objects.XenHost`的`get_host_by_filter`进行mock,仅需要一句
```
self.stubs.Set(objects.XenHost, 'get_host_by_filter',
                       self.fill_mock_host_enabled)
```
就可以了.不过这里的`self.fill_mock_host_enabled`对应的是mock方法的返回值.
_注意_:<br/>
在`self.fill_mock_host_enabled`方法内的入参个数,要与实际要mock的`get_host_by_filter`方法要求的参数个数一样.

###### 3. [mock]使用mock
该方法是我主要推荐的方法,最主要是因为写着简单,直接上示例
```
import mock

    @mock.patch('nova.objects.XenHost.get_host_by_filter')
    @mock.patch('nova.virt.tusxenapi.driver.XenAPIDriver._create_session')
    @mock.patch('nova.virt.tusxenapi.driver.XenAPIDriver._sync_pool_resources')
    def test_add_pool(self, _sync_pool_resources, _create_session, get_host_by_filter):
        pool_config = {
            "address": '192.168.1.1',
            "username": 'fake',
            "password": 'fake',
            "pool_name": 'test_pool_0',
            "physical_host": 'test-phy-host'
        }
        _create_session.return_value = self.create_fake_xen_session(pool_config.get('address', '127.0.0.1'))
        _sync_pool_resources.return_value = self._sync_pool_resources()
        get_host_by_filter.return_value = None
        self.driver.add_pool(context, pool_config)
```
如上,看着上面的代码有没有感觉很清新啊,基本内部没有什么其他的操作.唯一干的事情就是写上具体mock方法的返回值.<br/>
不过,需要注意的是.该方法使用的是装饰器,所以要求mock.patch的顺序与方法内的入参顺序是相反的.其他的就没什么了.你就可以愉快的玩耍了.

###### 额外福利
[如何来mock`__init__`方法内的方法.](https://stackoverflow.com/questions/54899511/how-to-mock-the-method-which-was-called-by-the-method-of-init/54899760#54899760)
```
import a

class Demo(object):
    def __init__(self):
        ......
        fun_return_value = a.methodB()
        ......

   def methodA(self):
       ......
```
这里给上使用mock的解决办法

```
class TestDemo(test.TestCase):
    def setUp(self):
        super(TestDemo, self).setUp()
        self.mocks = [mock.patch('a.methodB',
                                  mock.MagicMock(return_value=None))]
        for single_mock in self.mocks:
            single_mock.start()
            self.addCleanup(single_mock.stop)
```
