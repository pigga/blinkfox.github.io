---
title: Linux打包Python文件为RPM格式
date: 2018-12-21 13:57:00 +0800
layout: post
permalink: /blog/2018/12/21/Linux下打包Python的rpm包.html
categories:
  - Linux
tags:
  - Linux
  - Rpm
---

花费了将近一周的时间,才把rpm打包弄好.不能说已经了然于胸,但至少踩过了很多坑.接下来就顺顺在打包过程中的二三事.刚开始接到任务的时候,一脸懵逼.
![1.jpeg](https://cdn.nlark.com/yuque/0/2018/jpeg/228600/1545364635210-c542e2ba-aa01-4825-990f-b2c14ee81201.jpeg)


作为专业的`JAVA Web`程序员,确定要让我干这么跨界的事情吗?之前自己虽然也安装过其他的rpm包,但我保证,我只是看说明书,跟着一路弄下来的.并且之后对它可是有多远躲多远的.不过考虑到自己的title,我知道那不应该是我关心的事情.那么问题来了,什么是rpm呢?
# 什么是RPM
> *__RPM（RPM软件包管理器）__*
> RPM^ ^ 是Red-Hat Package Manager（RPM软件包管理器）的缩写，这一文件格式名称虽然打上了RedHat的标志，但是其原始设计理念是开放式的，现在包括OpenLinux、SuSE.以及Turbo Linux等Linux的分发版本都有采用，可以算是公认的行业标准了。
看到了rpm是什么,我们就来了解一下RPM相关的命令
本来准备直接 `rpm -help`让大家看看相关参数的,但是考虑到内容太多,感兴趣的[自行点击](https://www.yuque.com/yannis/linux/fuczr7).这里放几个常用的命令.
```
# rpm 安装(后面的.rpm为你需要安装的rpm包)
rpm -ivh autoinstall.rpm
# 查看是否安装了某个rpm,注意后面的名称可以是安装的服务的部分名称
rpm -qa | grep autoinstall
# rpm 卸载.(后面的名称即上面查看服务里的服务名称)
rpm -e autoinstall-1.0.0.dev33-1.noarch
```

至此,基本的rpm基础已经了解到了.那么关键的问题来了,如何制作RPM
# 如何制作RPM
把制作RPM,道上传闻有多种方式,我自己测过的有两种
```
#第一种
python setup.py bdist_rpm
#第二种
rpmbuild -bb autoinstall.spec
```
注意:无论上述那种都需要`setup.py`及`setup.cfg`文件,描述相关的软件信息.
我自己使用的是第二种rpmbuild,
- 首先安装

```
yum install rpm-build
```

- 文件`setup.py`

```
import setuptools

# In python < 2.7.4, a lazy loading of package `pbr` will break
# setuptools if some other modules registered functions in `atexit`.
# solution from: http://bugs.python.org/issue15881#msg170215
try:
    import multiprocessing  # noqa
except ImportError:
    pass

setuptools.setup(
    setup_requires=['pbr'],
    pbr=True)
```

- 文件`setup.cfg`,想了解更多,[请点击](https://www.yuque.com/python_study/environment/dn25ov)

```

[metadata]
name = autoinstall
version = 1.0
summary = autoinstall
author = Yannis
author-email = openstack-dev@lists.openstack.org
home-page = http://www.openstack.org/
classifier =
    Environment :: TusCloud
    Intended Audience :: Information Technology
    Intended Audience :: System Administrators
    License :: OSI Approved :: Apache Software License
    Operating System :: POSIX :: Linux
    Programming Language :: Python
    Programming Language :: Python :: 2
    Programming Language :: Python :: 2.7

[files]
packages =
    autoinstall

[pbr]
autodoc_index_modules = True

[build_sphinx]
all_files = 1
build-dir = doc/build
source-dir = doc/source

[egg_info]
tag_build =
tag_date = 0
tag_svn_revision = 0

[wheel]
universal = 1
```
* 使用rpm安装不可缺少的spec文件,想了解更多spec,[请点击](https://www.yuque.com/python_study/environment/ts4gsu)

```
%if 0%{?rhel} && 0%{?rhel} <= 6
%{!?__python2: %global __python2 /usr/bin/python2}
%{!?python2_sitearch: %global python2_sitearch %(%{__python2} -c "from distutils.sysconfig import get_python_lib; print(get_python_lib(1))")}
%{!?python2_sitelib: %global python2_sitelib %(%{__python2} -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())")}
%else
%global __python2 /usr/bin/python
%endif

%define name autoinstall
%define version 1.0
%define unmangled_version 1.0
%define unmangled_version 1.0
%define release 1
Summary: auto install scripts for tusCloud
Name: %{name}
Version: %{version}
Release: %{release}
Source0: %{name}-%{unmangled_version}.tar.gz
License: UNKNOWN
Group: Development/Libraries
BuildRoot: %{_tmppath}/%{name}-%{version}-%{release}-buildroot
Prefix: %{_prefix}
BuildArch: noarch
Vendor: UNKNOWN <UNKNOWN>
Provides: provide-files

%description
UNKNOWN

%prep
%setup -n %{name}-%{unmangled_version} -n %{name}-%{unmangled_version}

%build
%{__python2} setup.py build

%install
%{__python2} setup.py install -O1 --skip-build --root=$RPM_BUILD_ROOT
mkdir -p %{buildroot}%{_sysconfdir}/%{name}/
mkdir -p %{buildroot}%{_bindir}/
mkdir -p %{buildroot}/var/lib/autoinstall
mkdir -p %{buildroot}/var/log/autoinstall
mv %{name}/initconfig.yaml %{buildroot}%{_sysconfdir}/%{name}/
touch %{buildroot}/var/log/autoinstall/auto_install.log
touch %{buildroot}%{_sysconfdir}/%{name}/exec.txt

install -p -D -m 755 %{_builddir}/%{name}-%{version}/bin/* %{buildroot}%{_bindir}/

%clean
rm -rf $RPM_BUILD_ROOT

%files
%defattr(-,root,root)
%{python_sitelib}/autoinstall*
%config(noreplace) %attr(-,root,root) %{_sysconfdir}/%{name}/
%{_bindir}/*
/var/log/autoinstall
/var/lib/autoinstall

```

* 必须要有的requirements.txt(因为缺少这个文件遇到过问题)

```
# The order of packages is significant, because pip processes them in the order
# of appearance. Changing the order has an impact on the overall integration
# process, which may cause wedges in the gate later.

pbr>=1.3,<2.0
requests
datetime
bs4
PyYAML
abc
```

* 描述文件PKG-INFO

```
Metadata-Version: 1.1
Name: autoinstall
Version: 1.0
Summary: autoinstall
Home-page: http://www.openstack.org/
Author: Yannis
Author-email: openstack-dev@lists.openstack.org
License: UNKNOWN
Description: False
Platform: UNKNOWN
Classifier: Environment :: TusCloud
Classifier: Intended Audience :: Information Technology
Classifier: Intended Audience :: System Administrators
Classifier: License :: OSI Approved :: Apache Software License
Classifier: Operating System :: POSIX :: Linux
Classifier: Programming Language :: Python
Classifier: Programming Language :: Python :: 2
Classifier: Programming Language :: Python :: 2.7

```

* 整体的文件夹结构


![2018-12-21 14-17-31屏幕截图.png](https://cdn.nlark.com/yuque/0/2018/png/228600/1545373105968-0d20c868-ced5-461e-8c35-df66e4a0e624.png)

至此就可以欢快的进行打包了.执行

```
rpmbuild -bb autoinstall.spec
```

以上命令运行成功后会在当前用户目录下生成rpmbuild目录，该目录会包含以下子目录

--BUILD #编译之前，如解压包后存放的路径
--BUILDROOT #编译后存放的路径
--RPMS #打包完成后rpm包存放的路径
--SOURCES #源包所放置的路径
--SPECS #spec文档放置的路径
--SPRMS #源码rpm包放置的路径
* 执行安装
```
rpm -ivh autoinstall.rpm
```
安装成功后,即可直接执行相关命令了.

# 必要条件
* [x] 具体的软件对应的文件在autoinstall下的autoinstall下,目前感觉应该名称一致
* [x] 必须有个bin目录,存放需放置至/usr/bin下的文件
* [x] requirements.txt
* [x] PKG-INFO
* [x] setup.cfg
* [x] setup.py
* [x] \*.spec
* [x] \*.tar.gz
# 坑坑不息
1. Python文件路径,在Python内使用相对路径,由于在做编排文件的时候调整了目录结构,如上图,原本仅有一层的autoinstall文件夹,后面在制作spec的时候,要求必须要有一个bin目录,所以又加了一层文件夹.但加完之后,发现路径找不到了.后,修改项目内关于其他文件的引用为绝对路径;坑深度 <span data-type="color" style="color:#1890FF">**** </span>
2. 缺少PKG-INFO文件.之前未接触过相关东西,直接参照其他项目.其他项目没有该文件正常能编译,但自己制作的怎么也不成功.后来使用了`python setup.py bdist_rpm`可以正常打包,只是打包的包名及其他的格式与要求不符,就未采用.不过把这种方式对应的tar.gz包,放到环境上,执行rpmbuild打包,竟然正常.比对发现,我的项目内缺少了PKG-INFO文件.copy该文件,重新执行,就可以打包了;坑深度  <span data-type="color" style="color:#1890FF">*****</span>
3. bin内的autoinstall调用的主文件与外层文件夹名称一致,造成无法正确获取.最好要修改主文件的名称,不要跟文件夹名称一致;坑深度  <span data-type="color" style="color:#1890FF">*****</span>
4. 当所有安装在测试环境成功后,执行命令,尽然发现对应代码引用一直为我本地的路径.百思不得其解.后来意外发现在/usr/bin下除了有个autoinstall的文件外,还有一个autoinstall.pyc的文件.删除.pyc的文件,执行即可;坑深度  <span data-type="color" style="color:#1890FF">*****</span>
5. 在spec内有一个`%{python_sitelib}`获取的是本地python的路径,造成在ubuntu下不能正确在rpmbuild的路径下找到python路径,不过在centos下正常;坑深度  <span data-type="color" style="color:#F5222D">*****</span>
# 附录
rpmbuild
> rpmbuild
> -ba 既生成src.rpm又生成二进制rpm
> -bs 只生成src的rpm
> -bb 只生二进制的rpm
> -bp 执行到pre
> -bc 执行到 build段
> -bi 执行install段
> -bl 检测有文件没包含



