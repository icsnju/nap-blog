title: OpenStack平台手动搭建以及floating-ip的使用
category: OpenStack
date: 2015-11-20
tags: [OpenStack, floating-ip]
author: Yiqun Wang

---

> 最近尝试着根据OpenStack官方网站的教程搭建一个简单的OpenStack集群，并且使用floating-ip机制。下面简单对这些工作进行介绍。

<!--more-->

---



## 搭建OpenStack平台

### 开始之前

在上一篇中，对OpenStack平台做了简单的介绍。这一次，我们根据OpenStack官网上面的[文档](http://docs.openstack.org/liberty/install-guide-ubuntu/)，手动部署OpenStack中的服务组件。并在搭建好的平台上运行实例。平台的设计布局如下图所示：

![](/images/openstack-layout.png)

在上图中，主要包括一个Controller节点和Compute节点，以及一些Storage节点。在本例中我们仅仅关注Controller节点和Compute节点。从图中可以看出Controller和Compute节点均包括两个子网，分别为management network和public network。

我们使用[Vagrant](https://www.vagrantup.com/)工具初始化两个节点。Vagrantfile具体为：

```
Vagrant.configure(2) do |config|
	config.vm.box = "ubuntu/trusty64"
 	config.vm.hostname = "controller"
	config.ssh.username = "vagrant"
	config.ssh.password = "vagrant"
   
	config.vm.network "private_network", ip: "10.0.0.11"
	config.vm.network "public_network"
	
	config.vm.provider "virtualbox" do |vb|
		vb.gui = false
		vb.memory = "6144"
		vb.cpus = 2
	end
end
	
```

我们建立的虚拟机是Ubuntu14.04 64位，还有一些关于主机名，用户密码，内存，CPU的简单设置。我们设置了两个网络，分别对应于management work和public network。

### 准备工作

在安装OpenStack的各个服务组件之前，需要预先安装一些软件。

* [NTP协议](http://docs.openstack.org/liberty/install-guide-ubuntu/environment-ntp.html)，用于同步不同节点之间的状态及服务。
* [OpenStack包](http://docs.openstack.org/liberty/install-guide-ubuntu/environment-packages.html)，安装最新的OpenStack包，这里我们选取的是最新版Liberty。
* [SQL数据库](http://docs.openstack.org/liberty/install-guide-ubuntu/environment-sql-database.html)，在Controller节点上安装SQL数据库，大多数OpenStack服务使用SQL数据库用来存储信息。
* [NoSQL数据库](http://docs.openstack.org/liberty/install-guide-ubuntu/environment-nosql-database.html)，在Controller节点上安装，Telemetry服务使用NoSQL数据库(一般为MongoDB)来存储消息。
* [消息队列](http://docs.openstack.org/liberty/install-guide-ubuntu/environment-messaging.html)，在Controller节点上安装，OpenStack平台使用消息队列来协调各个服务之间的操作和状态信息。

### 安装Keystone组件

OpenStack平台中的Keystone服务主要用于身份管理，Keystone的作用类似与一个服务总线，其他服务都通过Keystone来注册器服务的Endpoint，针对这些服务的任何调用都需要经过Keystone的身份认证才可以。安装的步骤按照[文档](http://docs.openstack.org/liberty/install-guide-ubuntu/common/get_started_identity.html)中进行即可。

### 安装Glance组件

OpenStack平台中的Glance服务主要管理镜像。这点跟Docker中的image类似。即我们可以上传当前的镜像以便日后使用。安装的步骤按照[文档](http://docs.openstack.org/liberty/install-guide-ubuntu/glance.html)中进行即可。

### 安装Nova组件

OpenStack平台中的Nova服务是OpenStack平台的核心服务，它用于抽象虚拟机实例，并控制着虚拟机的状态便签，管理着它们的资源分配。安装的步骤按照[文档](http://docs.openstack.org/liberty/install-guide-ubuntu/nova.html)中进行，其中有一个地方文档写错了，我们会在后面详细说一下。

### 安装Neutron组件

OpenStack平台中的Neutron服务将OpenStack平台所在的物理网络泛化为网络资源池，通过对物理网络资源的灵活划分与管理，Neutron组件能为同一物理网络上的每个tenant提供独立的虚拟网络环境。安装的步骤按照[文档](http://docs.openstack.org/liberty/install-guide-ubuntu/neutron.html)中进行即可。安装后的网络模块大致如下：

![](/images/openstack-network-overview.png)

其中Controller中有网桥代理，Metadata代理，DHCP代理和L3代理，而Compute节点中只有网桥代理。

### 安装Horizon组件

OpenStack平台中的Horizon提供了Dashboard，即一个简洁方便用户友好的控制界面，让开发者和使用者更好的浏览并操作属于自己的计算资源，安装步骤按照[文档](http://docs.openstack.org/liberty/install-guide-ubuntu/horizon.html)中进行即可。

### 运行一个例子

在此之前，我们按照文档已经安装好了OpenStack平台最基本的一些服务，下面我们就可以运行一个[例子](http://docs.openstack.org/liberty/install-guide-ubuntu/launch-instance.html)。在安装Neutron服务后，集群的网络拓扑为下图。

![](/images/openstack-network-connect.png)

其中public provider network和public physical network应该在同一个网段内，management physical network是启动虚拟机的Vagrantfile中已经预先定义好了。Private project network需要自己手动创建，在文档中有例子，这个网段应该可以是任意指定的。 我们一共运行了两个实例，一个为公共实例，一个为私有实例，公共实例可以通过ip地址直接ssh访问(因为在同一子网内)，但是私有实例则需要指定floating-ip，才能访问。把floating-ip的介绍的[链接1](https://www.rdoproject.org/networking/difference-between-floating-ip-and-private-ip/) [链接2](http://docs.openstack.org/user-guide/cli_manage_ip_addresses.html)放出来，供大家参考。

![](/images/openstack-network-instance.png)

如果想要访问私有实例private-instance，由于分配了floating-ip，所以它的访问过程如下图所示：

![](/images/openstack-floating-ip.png)

## 安装过程遇到的问题总结

* 首先在安装完Compute服务后，发现在Controller节点上并没有显示出Compute节点上面的nova-compute服务，在网上找了一下原因。有的大神提出来在配置文件/etc/nova/nova.conf中，关于消息队列的三个配置项不应该在`[oslo_messaging_rabbit]`这个section中，应该在`[DEFAULT]`中，修改后重启nova-compute服务即可。在重启后，在Controller节点上发现nova-compute服务，状态为up，但是过了一会儿状态变为down了。不知道为什么，在网上找了很久也没有解决。于是把Compute节点删掉，重新装了一遍，就好了。好了。了。
* 安装完DashBoard组件后，可以正常登陆，但是在打开`Instance`列表时，右上角会出现**Unable to Connect to Neutron**的提示，而且不能通过在DashBoard页面创建实例等。这是因为我在刚开始装Neutron组件的时候选择了[Provider networks](http://docs.openstack.org/liberty/install-guide-ubuntu/neutron-controller-install-option1.html)这个选项，在配置文件/etc/neutron/neutron.conf文件中，它禁用了service_plugins这个选项，即把它赋值为空。但是在第二个选项[Self-service networks](http://docs.openstack.org/liberty/install-guide-ubuntu/neutron-controller-install-option2.html)中，这一项赋值为router。由于后来要使用floating-ip机制，所以需要选择第二个网络拓扑，就把这一项填充好了。然后之前页面上的问题也成功解决了。
* 在运行的例子中，由于我们最后选择的是第二项网络拓扑，所以运行的实例不论是共有(public instance)的还是私有(private instance)的，都应该可以正常运行。但是我们只能成功运行私有实例，并可以使用floating-ip机制访问进新建的private虚拟机实例。而公共实例却不能正常访问，这是因为虚拟机在启动的时候DHCP没有分配好IP地址，所以尽管我们在DashBoard中发现这个实例已经有了IP地址，但是通过console进入虚拟机之后发现并没有分配正确的ip地址。通过实例的log分析，应该是DHCP的配置出了一些问题，也在网上找了很久，都没有找到解决办法。