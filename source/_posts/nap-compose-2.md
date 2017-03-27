title: 跨主机的docker容器编排 nap-compose （二）
category: nap
author: Yuan & Ying
date: 2015-11-07  14:00:00
tags: [nap, docker, compose]
---

简化的跨主机的多容器应用编排和部署工具的实现。

<!--more-->

---

我们已经在[前一篇文章](http://nap-group.herokuapp.com/2015-11/nap-compose-1/)介绍了背景和思路，现在该写代码了。
我们准备用python 3来实现。先了解一下docker api和python ssh的库作为准备工作。主要部分在于梳理docker-compose的源码，然后在其基础上修改docker client相关的函数，改成我们的实现，从而重用compose已经实现的解析`docker-compose.yml`，生成容器间连接关系和获取容器域名的功能。

## [Docker Remote API](http://docs.docker.com/engine/reference/api/docker_remote_api_v1.21/)

### 概述
之前我们是通过定义fleet的service unit来管理多个主机上的docker daemon的。在service unit中可以直接写docker命令，fleet会把service分派给各主机的systemd来执行。
现在我们要通过docker提供的restful api来编程直接与docker daemon交互。restful api返回的是json格式的数据。有很多官方或非官方的repo把restful api包装成各种[主流编程语言的docker client库](http://docs.docker.com/engine/reference/api/remote_api_client_libraries/)，这样我们就不用自己写解析json的那部分代码了。

### 开启api访问的tcp端口
要远程访问docker api，首先要开启tcp端口。
对ubuntu 14.04系统，在`/etc/default/docker`中增加一行：
```
DOCKER_OPTS="-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock"
```

CoreOS和RHEL/CentOS使用systemd来管理系统服务，Ubuntu 15.04及以后的版本也改为systemd，设置方法有所不同，具体可以参考[CoreOS的文档](https://coreos.com/os/docs/latest/customizing-docker.html)和[dockone上的相关问题](http://www.dockone.io/question/616)

设置后重新启动docker daemon，即可从通过2375访问docker api。另外后面的`unix:///var/run/docker.sock`是给本机的docker client命令行使用的。我们实验的主机（VM）是ubuntu 15.10，IP是`10.1.1.5`，安装了docker 1.9.0，测试一下api：
```
curl http://10.1.1.5:2375/version
```

也可以用chrome访问这个地址。推荐安装json formatter这个chrome插件，它可以格式化显示json数据。

返回的结果如图：
<img src="/images/remote-api.png" height="240" alt=""/>

> 直接暴露tcp端口是有安全问题的。docker api支持登录验证，不过这里为了简便没有使用。

### docker-py简单示例

我们使用的[docker-py](https://github.com/docker/docker-py)是docker官方实现的。在开发机器上通过pip安装这个库。
```bash
apt-get install python3-pip  # 对ubuntu; windows在安装python时会同时安装pip
pip3 install docker-py
```

下面是启动容器、获取运行容器列表和容器ip的示例代码
```python
import docker

client = docker.Client(base_url= 'http://10.1.1.5:2375')

container = client.create_container(image='ubuntu:latest',command='echo hello')

print(container['Id'])
client.start(container)

containers = client.containers()           # /containers/json

for item in containers:
    info = client.inspect_container(item)  # /containers/{id}/json
    print("[{0}] {1} {2}".format(item['Id'][0:9],
                                 info['NetworkSettings']['IPAddress'],
                                 info['Name']))
    # print(info['HostsPath'])
```

运行的结果示例如下：
```cmd
C:\Python3\python.exe D:/Code/Py/main.py
[f4c28710d] 172.17.0.4 /shipyard-controller
[7edce4fce] 172.17.0.6 /shipyard-swarm-agent
[3c71749ce] 172.17.0.7 /shipyard-swarm-manager
[ea3527753] 172.17.0.5 /shipyard-proxy
[3e44ea102] 172.17.0.2 /shipyard-certs
[184391e5d] 172.17.0.3 /shipyard-discovery
[b9d60b44e] 172.17.0.8 /shipyard-rethinkdb

Process finished with exit code 0
```

这个库的函数与docker remote api文档有很直接的对应关系。函数的返回值也是与文档对应的json对象，可以通过字典的方式非常方便的获取相关字段的值。

## python ssh
通过ssh操作远程主机是不太规范的做法。原则上说，我们也可以不必使用remote api，而是直接ssh到各主机，然后执行docker命令，甚至做任何系统操作。不过我们不想集齐docker三件套，用docker engine来修改hosts，而又没有相关的api，所以只能用ssh了。

这里我们参考《Python自动化运维》，使用[paramiko](https://github.com/paramiko/paramiko)这个库。安装过程费了点周折。

先说Windows上的安装，直接用`pip install paramiko`是装不上的，会卡在依赖项`pycrypto`上。安装`pycrypto`过程中需要MSVC来编译，不过我没有安装MSVC，Google到的一个预编译的二进制文件不支持python 3.5，好在找到另一个预编译的，安装方法如下：
```
pip install --use-wheel --no-index --find-links=https://github.com/sfbahr/PyCrypto-Wheels/raw/master/pycrypto-2.6.1-cp35-none-win_amd64.whl pycrypto
pip install paramiko
```

在Ubuntu上还要麻烦些。在Ubuntu 15.10上，默认有python 2.7 和 python 3.4，在安装 `python3-pip` 时，会自动安装gcc，g++等开发工具，还有python3.5，不过没有安装python3.5-dev，导致无法编译`pycrypto`。

> PS, pip3默认使用python3.5，而不是系统自带的python3.4，所以最好把`/usr/bin/python3`这个符号链接从python3.4改成python3.5。
按照下面的步骤安装：
```
apt-get install python3.5-dev
pip3 install pycrypto
pip3 install paramiko
```

下面是Windows上的一个示例
```
import paramiko
import os

hostname='10.1.1.5'
username='root'
paramiko.util.log_to_file('D:\ssh.log')

ssh = paramiko.SSHClient()
ssh.load_system_host_keys()

key=paramiko.RSAKey.from_private_key_file(os.path.expanduser('D:\VM\ssh\id_rsa'))
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

ssh.connect(hostname=hostname, username=username, pkey=key)

stdin,stdout,stderr=ssh.exec_command('uname -a')
print(stdout.read())

ssh.close()
```

这个例子是通过ssh key登录到VM上，然后执行`uname -a`命令。返回的结果如下：
```
b'Linux x 4.2.0-16-generic #19-Ubuntu SMP Thu Oct 8 15:35:06 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux\n'
```

## 梳理docker-compose的源码
OK，准备工作基本就绪。下面梳理一下`docker-compose up`命令来创建、启动和连接多个容器涉及的代码，主要是参考[github上的源码](https://github.com/docker/compose)和《Docker容器与容器云》的6.1.2，下面的图参考了这本书的图6.1。
![](/images/compose.png)

+ project对象从compose.yml获取service列表，并根据link计算出各service的依赖关系，以此为顺序来创建各service。
+ docker-compose是针对单个docker engine的，project对象只关联了一个docker-client对象，首先我们要把它改成一个列表；
+ service对象也是只关联了一个docker-client对象，它是从project传递过来的，最终传给container对象来创建和启动容器；
+  从client列表中指派一个client给container对象，就是所谓的调度了。因为一个service会对应多个container，这些container应该可以跨Host，所以调度应该发生在service→container这个阶段；
+ 我们需要记录下来哪个docker-client创建了哪个service下面的container，容器的id，以及容器启动后的ip地址，这样我们再ssh到那个Host上去，修改对应容器id目录下的hosts文件；
+ 上面的操作可能会造成若干秒的延迟后才能正常地通过hostname来进行通信，为此，在容器的启动命令之前插入一个sleep命令，防止因为连接超时导致应用启动失败。
