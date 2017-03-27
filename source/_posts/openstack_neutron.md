title: OpenStack平台Neutron服务初探
category: OpenStack
tags: [OpenStack, Neutron]
date: 2015-12-02
author: Yiqun Wang

---

>之前搭建了一个两个结点的OpenStack集群，老板说太简单了太low了，于是最近又重新搭建了一个四个结点的OpenStack集群。其中包含一个Controller结点，一个Network结点和两个Compute结点。其中Network结点主要运行OpenStack中的网络服务，即Neutron组件。下面对这个集群的部署过程做一下总结，并且简单介绍一下Neutron服务。

<!--more-->

## 扩展Compute结点

扩展Compute结点相对来说是比较简单的。并且所有Computer节点的部署过程都是相同的，详细信息请参考上一篇。

## 部署Network结点

### 开始之前

在上一篇中，我们将Neutron服务的所有代理都放在了Controller组件里面，这些代理包括：

```
	Linux网桥代理没建立layer-2虚拟网络，为实例提供VXLAN通道并处理安全组问题(security groups)。
	DHCP代理，用于在创建实例时分配IP地址。
	L3代理，为虚拟网络提供路由和NAT服务。
	Metadata代理，提供配置信息，如实例的证书等。
```

我们想把上述代理都放在Network结点中，即将Neutron组件单独运行在一个结点中。我们的解决思路是尝试着分别将每个代理依次从Controller结点迁移到Network结点中。在此之前，我们首先将与各个代理相关的配置文件项列出来，并给予一些说明。

* ML2插件配置项(配置文件为/etc/neutron/plugins/ml2/ml2_conf.ini)

```
	type_drivers = flat,vlan,vxlan
	tenant_network_types = vxlan
	mechanism_drivers = linuxbridge,l2population
	extension_drivers = port_security
	flat_networks = public
	vni_ranges = 1:1000
	enable_ipset = True
```

在ml2插件的配置项中，主要是开启了flat,vlan,vxlan网络，其中私有网络(tenant)使用的是vxlan隧道，具体为下图所示。我们看到VXLAN隧道在网桥代理中，我们领management network的ip地址赋给VXLAN的网卡。而flat网络是OpenStack网络中的一种工作机制。Flat模式指定一个子网，规定虚拟机实例能使用的ip范围，也就是一个ip池，创建实例时，从ip池中取出一个有效的ip为虚拟机分配，并且在虚拟机启动时注入虚拟机镜像。在这里，给Flat网命名为public。

![](/images/openstack-network-connect.png)

* Linux网桥代理配置

```
	physical_interface_mappings = public:eth0 
	enable_vxlan = True 
	local_ip = OVERLAY_INTERFACE_IP_ADDRESS
	l2_population = True
	prevent_arp_spoofing = True
	enable_security_group = True
	firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```

在Linux网桥的配置项中，我们开启了VXLAN通道，并且指定ip为management work中的ip，并将public网络绑定在eth0网卡(public network)。

* L3代理配置

```
	interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
	external_network_bridge =

```
* DHCP代理配置

```
	interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
	dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
	enable_isolated_metadata = True
```

在上述两个代理中，我们都在interface_driver项中选择了Linux网桥的驱动。

* Metadata代理配置，主要为与keystone和nova之间关于认证的一些[配置项](http://docs.openstack.org/liberty/install-guide-ubuntu/neutron-controller-install.html)，没有太多需要说明的。

### 将L3代理迁移到Network结点

1. 首先在Controller上面停掉L3代理，在Network结点上面启动L3代理，其实配置项都是一样的。
2. 然后需要在Network结点配置好Linux网桥代理，原因不用说，只有L3代理不就相当于一个孤点么，没有任何作用。
3. 重新建立一个网络，运行一个实例，配置一个floating-ip，然后ssh进去就可以啦！

### 将DHCP代理迁移到Network结点

1. 跟L3代理一样，首先还是在Network结点中启动DHCP代理，然后停掉Controller结点上面的DHCP代理。
2. 本以为建立好一个网络后，一切还是很正常的运行，但是发现事实不是这样子的。当我写下这篇文章的时候，我只记得我当时去看了实例的log，发现已经分配了正确的ip地址，于是就断定DHCP代理是没有错的，并且查询了一下neutron的log，发现有几个是刚刚更新过的，发现在metadata的log里面，没有将请求或操作连接到正确的消息url中，即一直都在请求127.0.0.1的本地消息队列，而Network结点是没有装消息队列的，所以肯定是不对的。
3. 最后，直接一次性将metadata代理也迁移了过去。
4. 建立好一个网络，运行实例，配置一个floating-ip，然后ssh进去就可以啦！

### 结果截个图

1. Neutron组件代理列表：
	![](/images/openstack-neutron-agentlist.png)
	
	可以看到，Neutron组件的所有代理目前都跑在了Network结点上面。
	
2. Floating-ip测试：

	建立好一个虚拟机实例。
	![](/images/openstack-neutron-novalist.png)
	
	然后ssh进去访问它。
	![](/images/openstack-neutron-floatingip.png)
	
	可以看到我们最后是在Network结点上进行访问的，并且需要输入密码，这点我们会在后面详细解释一下。

## 在OpenStack中建立网络

之前说了那么多，下面具体讲一下在OpenStack中如何建立网络，看完这个过程，你会发现里面有一些东西会跟前面的某些名词对应上。


```
	source admin-openrc.sh
	neutron net-create public --shared --provider:physical_network public --provider:network_type flat
```

这条命令会建立一个类型为flat，名称为public的共享虚拟网络。

```
	neutron subnet-create public 10.0.2.0/24 --name public --allocation-pool start=10.0.2.110,end=10.0.2.200 --dns-nameserver 10.0.2.3 --gateway 10.0.2.2
```

这条命令会为public网络分配ip池，可以看到分配了90个ip，并且有相对应的dns服务器和网关。

```
	source demo-openrc.sh
	neutron net-create private
	neutron subnet-create private 172.16.1.0/24 --name private --dns-nameserver 10.0.2.3 --gateway 172.16.1.1
```

这两条命令是demo用户创建了一个私有的虚拟网络，名字为private。

```
	source admin-openrc.sh
	neutron net-update public --router:external
	source demo-openrc.sh
	neutron router-create router
	neutron router-interface-add router private
	neutron router-gateway-set router public
```

上述命令会建立一个路由器router，并且将private网络作为router上面的一个网卡，将public网络当做router上面的网关。建好了网络，在建立实例之前，首先验证一下。在Controller结点上面列出端口信息。并且查看每个端口的详细信息。

```
	source admin-openrc.sh
	neutron router-port-list router
	neutron port-show $(id)
```

截图如下：
![](/images/openstack-router-port-list.png)

可以看到这些端口绑定的host_id是neutron，即Network结点。这是因为L3代理是在Network结点上面的。这个时候检查一下网络的名空间，就应该在Network结点，而不是Controller结点了。

![](/images/openstack-ip-netns.png)

并且这个router的网关10.0.2.112只能在Network结点上ping通。并且，建立好实例后，目前只能在Network结点ping通分配好的floating-ip，在ssh进去的时候是需要输入密码的，这是因为默认的ssh-key是controller结点上的。其实，按照官方文档的说法，只要在public网络下面的任何结点都是可以通过ssh的方式进入虚拟机实例的，而我们目前只能在Network结点上，这是因为我们新建的路由器是绑定在Network结点上面的，而这是因为L3代理是运行在Network结点上面的。感觉能够通过添加路由转发规则的方式，令public子网内所有的主机都能通过floating-ip访问虚拟机实例。这一点会在本文写好之后进行尝试。

## 总结
经过这两天的尝试，感觉OpenStack的灵活性很高。就像我们这一次把Neutron服务从Controller结点迁移到Network结点，都是直接把网络代理从这边关闭，到那边打开就可以。给我的感觉就像是传说中的那种可以在硬件层面上组装的手机，需要什么零件自己选择。

还有一点就是，在出现错误时，我目前都是看log的。log的目录是在/var/log/$(service)/*.log中，每次都查看最近更新过的log，然后看里面有什么错误之类的。