title: OpenStack与DevStack初探
category: OpenStack
author: Yiqun Wang
date: 2015-11-9 17:00:00
tags: [openstack, devstack]

---

本文会简单介绍一下OpenStack平台，它的手动部署方法，以及使用devstack脚本自动化部署的方法。

<!--more-->

---

## OpenStack简介

OpenStack是一个开元的云计算管理平台项目，由几个主要的组件组合起来完成具体任务。OpenStack支持几乎所有类型的云环境，它的目标是提供实施简单、可大规模扩展、丰富、标准统一的云计算管理平台。它的主要组件及相应的功能如下图所示：

![](/images/openstack-service.png)


### 手动搭建OpenStack平台

我们选择的平台是Ubuntu14.04。根据[官方文档](http://docs.openstack.org/liberty/install-guide-ubuntu/)，我们可以手动部署OpenStack的各个服务组件。在部署服务之前，需要首先安装NTP，以及数据库和消息队列组件。其中NTP是为了保证集群中的所有结点有一个全局时钟。而数据库存放每个服务的数据等信息，OpenStack支持MySQL和MongoDB。消息队列被用来协调各个服务之间的操作和状态，一般用[RabbitMQ](http://www.rabbitmq.com/)实现。

每个服务的安装部署过程都比较类似。首先会建立一个数据库，然后建立一个用户，接着安装依赖的包等等。最后手动修改配置文件(主要为/etc/service_name/service_name.conf)。

## DevStack简介

DevStack实际上是一个shell脚本，可以用来快速搭建OpenStack的运行和开发环境，但是devstack并不适合用在生产环境中，它比较适合OpenStack开发者下载最新的OpenStack代码迅速在自己的笔记本上搭建一个开发环境出来。


### 使用DevStack脚本进行一键部署
使用devstack脚本很简单，安装git，然后下载devstack代码到本地，运行stack.sh脚本一次设定MySQL, RabbitMQ, OpenStack Dashboard和Keystone的密码，密码输入后stack.sh脚本会自动开始安装必要的软件包和库并下载最新的OpenStack及其组件代码，整个过程无需干预。devstack的下载方式：

```
git clone https://git.openstack.org/openstack-dev/devstack
```


Devstack的默认配置文件都保存在stackrc文件中，我们也可以创建一个localrc文件来修改他的默认设置。它的使用比较简单，但是它的脚本代码一点都不简单，因为它要维护支持多Linux发行版。通过阅读脚本，并且与stack.sh源码对照。将整个过程自动分为以下若干步骤：

```
清楚上次devstack脚本运行的缓存。
根据stackrc/localrc等配置文件加载一些系统变量及其他配置选项
载入本地变量，根据不同的Linux版本进行不同的操作。（详细的操作步骤目前还不是特别熟悉，需要的时候再看）
配置安装及log目录等信息。
预先设定错误处理过程，即如果在脚本运行过程中发生错误，需要在退出前进行哪些操作。
导入每个OpenStack服务所需要的plugins或是安装服务所用到的libraries及functions。
交互配置，及提供MySQL,RabbitMQ,Dashboard以及Keystone的密码。
安装database及队列queue。
安装配置本地的python环境。
安装service client。
安装service middleware（即每个需要安装的具体服务）。
初始化并启动这些服务。
保存安装过程退出后，在运行时可能用到的信息，如相关的系统变量等等。
打印安装完成的信息。
```

### 对比devstack部署以及手动部署的过程，以nova服务为例。
#### 手动部署
````
CREATE nova DATABASE;
CREATE nova Users;
CREATE compute service API endpoint;
apt-get install $(packages);
vim /etc/nova/nova.conf;
service $(*) restart;
````
#### 自动部署(stack.sh)
````
Line756, install_nova_client.
Line811, stack_install_nova, 这两个函数主要进行的就是git clone和pip install, 即下载安装所需依赖包。
Line815, configure_nova，主要对/etc/nova/nova.conf进行配置。
Line1131, init_nova, 主要进行初始化数据库等。
Line1252, start_nova，启动nova服务。
````
