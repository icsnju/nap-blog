title: 跨主机的docker容器编排 nap-compose （一）
category: nap
author: Yuan & Ying
date: 2015-11-07  11:00:00
tags: [nap, docker, compose]
---

docker三件套（compose + swarm + engine）可以实现跨主机的多容器应用编排和部署。我们要实现一个功能类似的，简化一些的工具，这里介绍一下实现的思路。

<!--more-->

---

## 容器编排和docker-compose

### 概述
要想在**单个**主机上部署多个容器相关联的应用，
- 一种方法是依次执行docker命令，依次启动各容器。要特别注意容器的启动顺序：如果容器B依赖A，但容器B比A先启动，B会可能因为无法访问A而异常停止。
- 另一种是方法使用[docker-compose](https://docs.docker.com/compose/)工具，它的前身是fig。compose是python实现的，被打包成可执行文件发布。

docker-compose是简化多容器应用编排（orchestration）和部署的工具，它允许开发者把一个应用中多个容器及它们之间的依赖关系都写到一个 `docker-compose.yml` 配置文件中，将其解析后自动按顺序执行docker（通过docker API），启动整个应用。

### guestbook示例
下面是一个 `docker-compose.yml` 的[例子](http://yingz.info/2015-10/compose-guestbook/)，来自[kubernetes(k8s)](http://k8s.io/)中的[guestbook示例](https://github.com/kubernetes/kubernetes/tree/master/examples/guestbook)。它是一个简单的web留言板，共有3个组件，分别是

- php前端，运行在apache服务器；
- redis master，作为写入数据库；
- redis-slave，作为读取数据库。

```
web:  
  image: guestbook
  ports:
  - 80:80
  links:
  - redis-master:redis-master
  - redis-slave:redis-slave

redis-master:  
  image: redis
  ports:
  - 6379

redis-slave:  
  image: redis-slave
  ports:
  - 6379
  links:
  - redis-master:redis-master
```

### service 和 links
compose中有下面2个概念
- project：一个`docker-compose.yml`描述的就是一个project，也就是开发者的一个应用；一个project由若干service组成。
- service：是yml中的顶层节点，上面的例子里是web， redis-master，redis-slave 。看起来每个service的子节点都是通过`docker run`运行一个容器的参数，不过service是比容器高一层的抽象，一个service对应于多个容器（container）。比如运行`docker-compose scale web=3`命令可以横向扩展web这个service，让它有3个容器。
> service这样简单地横向扩展在实际中显然是不够的，如果是web应用，应该有一个负载均衡节点作为入口。此外，横向扩展的容器的hostname应该有一个约定格式，这样才能与它们通信。

compose除了在一个yml文件里描述多容器以外，还要把它们连接起来，这是通过yml中的`links`来描述的。compose能够让各容器之间通过域名来互相访问。

使用默认的docker0网桥时，开发者只能在docker启动之后才能获取容器的ip，而且这个ip不是固定的，在容器重启后可能会变化，这给跨容器的通信造成了不便。在考虑横向扩展和失败转移时，会动态增减容器，它们的ip也是不能事先知道的。
另外，硬编码ip地址不是好的做法，最好是通过域名来屏蔽ip的变化。

我们不需要配置DNS来实现域名解析，docker的`link`参数可以实现在单主机上容器间的静态域名解析。
```
docker run --link=容器A:A的别名 镜像B
```

这个命令**将要**运行的容器B与一个**已经**运行的容器A（通过`--name`来指定容器名）连接起来：修改容器B的环境变量及`/etc/hosts`，在`hosts`中添加类似下面项。
```
<A的IP> A的别名
```

这样容器B中就可以通过`A的别名`这个域名来访问容器A了。
必须要求容器B先于容器A启动，这样docker才能知道容器A的IP。

此外，关于volumns的内容以后再写文章讨论。

## 跨主机编排

### docker三件套

如果要将有多个容器的应用部署到多个主机上，直接使用docker-compose就困难了。

一种解决方案是使用docker三件套：compose，[swarm](https://docs.docker.com/swarm/)和docker engine（也就是docker daemon），此外还需要新增的[networking](http://docs.docker.com/engine/userguide/networking/)。

swarm是集群管理工具，目标是把多个主机上的docker服务整合为集群，提供**单一**的入口。主要解决2个问题：
+ 组成集群：之前我们的实验都是基于CoreOS的fleet来管理集群的。[fleet](https://coreos.com/using-coreos/clustering/)使用[etcd](https://coreos.com/etcd/)作为集群信息的存储，[systemd](https://zh.wikipedia.org/wiki/Systemd)作为主机上的执行代理。swarm与fleet比较相似。swarm在主机上的执行代理自然是docker engine（也就是docker daemon，通过docker API），集群信息存储除了使用etcd，还支持zookeeper和consul。
+ 分发调度：既然swarm把多个主机上的docker抽象为单一的入口，那就要解决任务如何分配到各个主机上的问题，也就是所谓的调度问题。目前swarm支持的有随机和binpack（装箱，根据CPU和内存）调度。调度是集群管理都需要解决的问题。mesos是进行调度管理的项目，swarm也有与**mesos的整合**。

networking是解决跨主机容器联网问题的库，与flannel类似，也是采用的覆盖（overlay）网络，docker 1.9版增加了`docker network`命令，用来管理跨主机的网络。

另外，docker machine是在虚拟机，IaaS云平台上从零开始部署docker的，有点像chef，puppet这类自动化工具或vagrant这种虚拟机开发环境构建工具。它还可以ssh到各主机执行命令。不过如果已经安装好了docker，machine就不太必要了。

### 跨主机的覆盖网络
覆盖网络是跨主机的docker容器间联网问题的一种解决方案。
跨主机的docker容器间联网，关键是实现容器间的**路由**，也就是说让各主机都知道每个主机上运行了哪些容器，或者说每个容器都运行在哪个主机上。知道了这些信息，容器间跨主机通信时，容器发出的数据包先发给本主机，然后发给目标容器所在的主机，目标主机再转发给目标容器。
要想获取容器分布信息，
+ 一种方案是将其都放到各主机都能访问的集中存储上去，然后各主机上安装代理来同步这些信息，并负责转发数据包；
+ 另一种是主机上安装一个虚拟路由器，通过路由协议来交换信息；或者通过`iptables`来实现类似的功能；

>传统的虚拟机支持桥接网络，此桥接非彼docker的桥接，docker的桥接在虚拟机技术中对应的叫NAT网络（本来在docker中也是NAT，但不叫NAT，却被称为桥接。。。）。虚拟机的桥接，是将VM的虚拟网卡与物理网卡绑定，基于物理网卡的[混杂模式](https://zh.wikipedia.org/wiki/%E6%B7%B7%E6%9D%82%E6%A8%A1%E5%BC%8F)或支持虚拟化的网卡的[SR-IOV](https://msdn.microsoft.com/en-us/library/windows/hardware/hh440235%28v=vs.85%29.aspx)，从外部网络看来，虚拟网卡与物理网卡没有什么区别，从而虚拟机与物理主机在网络上的地位也是对等的，已有的网络管理系统像管理物理主机一样管理虚拟机，也就不存在跨主机的VM通信问题了。

>docker也是可以通过[自定义网桥](http://blog.oddbit.com/2014/08/11/four-ways-to-connect-a-docker/)来使用虚拟机的桥接网络模式的。

CoreOS的[flannel](https://github.com/coreos/flannel)采用的是第一种方案，它使用etcd做为集中存储，以ip in ip的形式来转发数据包。docker networking与flannel类似。

### 设置flannel
先获取[etcd(v2.0.12)](https://github.com/coreos/etcd/releases/download/v2.0.12/etcd-v2.0.12-linux-amd64.tar.gz)和[flannel(v0.5.3)](https://github.com/coreos/flannel/releases/download/v0.5.3/flannel-0.5.3-linux-amd64.tar.gz)的二进制文件，并解压到**相关**主机的 `/usr/bin`（或`$PATH`中的其它路径）。

> 注意，flannel安装在需要组成覆盖网络的主机上，etcd可以单独设置主机，不必在flannel主机里。下面的示例etcd安装在ip为`10.1.1.12`的主机上。
> 可以将etcd安装在多台主机上[组成集群](https://coreos.com/etcd/docs/latest/clustering.html)，以提高可用性。

```
# 在10.1.1.12主机上，启动etcd，监听本机所有ip的4001端口
etcd -listen-client-urls http://0.0.0.0:4001 -advertise-client-urls http://127.0.0.1:4001  &>/dev/null &

# 设置flannel的配置key（设置一次即可），使用 10.4.0.0/16 网段
etcdctl rm /coreos.com/network --recursive  &>/dev/null
etcdctl mk /coreos.com/network/config '{"Network":"10.4.0.0/16"}'

# 在任意主机上，检查etcd是否可用
curl http://10.1.1.12:4001/version

# 在需要组成覆盖网络的主机上，启动flannel，并设置docker0网桥
flanneld -iface="eth1" -etcd-endpoints="http://10.1.1.12:4001" &>/dev/null  &
sleep 1

service docker stop              # 对ubuntu 14.04，停止docker daemon

. "/run/flannel/subnet.env"      # 获取下面用到的${FLANNEL_*}相关环境变量

ifconfig docker0 ${FLANNEL_SUBNET}
docker daemon --bip=${FLANNEL_SUBNET} --mtu=${FLANNEL_MTU}  &  # 启动docker daemon
```

设置完成后，flannel在主机上添加了一个flannel0网桥，它与docker0网桥不同：bridge utils的`brctl show`会显示docker0网桥，但不会显示flannel0网桥。上面的命令将docker0网桥的ip设置在flannel0的ip段内，启动的容器也在这个ip段，这样就可以通过flannel0来转发跨主机的数据包了。

### 不同主机容器的 `/etc/hosts`

虽然设置了flannel覆盖网络，可以让跨主机的容器间通过IP地址来通信，但是还不能直接使用域名。
我们可以增加一个容器来专门提供DNS服务，也可以用类似`docker --link`的做法，分别修改各容器的hosts。docker三件套再加上networking可以实现这样的功能，不过我们希望能自己实现一下这样的功能。

>注意，由于compose只能与一个docker daemon交互，不能与多个主机上的docker daemon交互，无法实现跨主机的容器间由域名通信。

我们先不考虑解析`compose.yml`，生成容器间连接关系和如何获取容器域名。假设已经知道了要启动哪些容器及这些容器应该是什么域名，容器可以运行在不同的主机上，这些主机间已经有flannel的覆盖网络允许容器间的通信。启动容器后我们可以得到各主机上运行的容器 id 和 ip，然后要把域名和ip这一对信息分别写道所有相关容器内的`/etc/hosts`文件里，这样容器间就可以使用域名通信了。

用docker API编程地访问多个主机的docker daemon，可以创建、启动容器，获取容器的信息，包括ip地址和hosts在对应主机的路径`/var/lib/docker/containers/{容器id}/hosts`，将域名信息用 ssh 写入到各主机的相应`hosts`文件即可。

> docker 三件套是最终通过 engine来写入 `hosts` 的，但并没有提供该功能的API。

我们在下一篇介绍具体的做法。
