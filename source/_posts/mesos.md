title: Mesos Framework 解析
category: nap
author: Jie CHEN
date: 2015-11-07 10:00:00
tags: [mesos]
---

Mesos是Apache旗下的开源集群资源计算框架，它能将多台机器（包括物理机和虚拟机）的资源，包括CPU，内存，硬盘等计算资源抽象出来，进行资源隔离和共享，构建高容错性、高可用性的分布式资源管理系统。

<!--more-->

---

## 介绍

### Mesos Framework

Mesos采用两层架构，所有计算框架均在底层，即Mesos Master负责对任务的资源分配给各个框架；在计算框架之上为Mesos Framework，它可用是通用的框架，也可用为特定软件编写的框架，如Framework一个长时间运行的服务（比如JobTracker等），也可以是一个短生命周期的Job或者Application。这样设计的好处是，**作为底层的Mesos不需要知道顶层的应用的细节，只负责根据slave资源状况提供offers，而调度由顶层的Framework进行实现**，这样底层的Mesos对所有应用（分布式or集中式，长时间服务or短暂性任务等）均可适用。

Framework可以根据是否符合任务对资源的约束，选择接受或拒绝资源邀约。一旦资源邀约被接受，Framework将与Master协作调度任务，并在数据中心的相应Slave节点上运行任务。

Framework包含两大部分：

* **Scheduler**
	Scheduler负责管理框架所获得的资源，按照作业的输入量，将其分解成若干任务，并未这些任务申请资源，监控这些任务的运行状态，一旦发现某个任务运行失败则重新为其申请资源。
* **Executor**
	Executor负责执行调度器所分配的任务。

Framework结构如图：
![Framework Structure](/images/mesos-arch.jpg)
<!-- http://cdn1.infoqstatic.com/statics_s2_20151020-0055-2/resource/articles/analyse-mesos-part-04/zh/resources/0604001.jpg -->

## 分析

### Framework任务分配过程

Framework任务分配流程图如下：
![Framework Task](http://s2.51cto.com/wyfs02/M01/09/F3/wKiom1LKbc-DCmXcAACKJCK6gs4287.jpg)

Mesos的资源分配行为发生在：**新框架注册**，**框架注销**，**增加节点**，**出现空闲资源**等情况，对Mesos Framework进行资源分配的流程如下：

1.  Mesos Master中的`Allocator`模块将要为某个Framework分配资源，则将资源封装到`ResourceOffersMessage`中，通过网络传输发送给`SchedulerProcess`；
2.  `SchedulerProcess`调用用户在Scheduler中编写的`resourceOffers`函数，告知有新资源可用；
3.  Framework的Scheduler调用`MesosSchedulerDriver`中的`launchTasks()`函数，告知将要启动的任务；
4.  Mesos Master将待启动的任务分装成`RunTaskMessage`发送给各个Mesos Slave；
5.  Mesos Slave收到`RunTaskMessage`消息后，将其发送给对应的`ExecutorProcess`；
6.  `ExecutorProcess`收到消息后，进行资源本地化，并准备任务环境，最终调度用户编写的Exector中的`launchTask`任务启动任务。

### Framework开放接口及作用

Mesos向开发者开放了API，API隐藏了Mesos底层和分布式交互的具体的细节，使得开发者可方便地用根据相应使用环境编写对应的Framework，提供的接口支持`C++`，`Go`，`Haskell`，`Java`，`Python`，和`Scala`语言。

**Scheduler API**

| 方法                   | 说明                                   |
| -                      | -                                      |
| `registered()`         | 当Scheduler成功向Master注册时被调用    |
| `reregistered()`       | 当Scheduler再次注册成功时被调用        |
| `disconnected()`       | 当Scheduler和Master失去连接时被调用    |
| **`resourcesOffers()`**| 当Framework收到Mesos提供的资源时被调用 |
| `offerRescinded()`     | 当某个offer不再适用时调用(Slave丢失或者其他Framework正在适用这个offer),master撤销发送给Framework的资源 |
| `statusUpdate()`       | 当Mesos向框架发送消息时被调用          |
| `frameworkMessage()`   | 用来向Scheduler传递Executor发送的消息  |
| `slaveLost()`          | 通知Mesos无法和指定ID的slave通信       |
| `executorLost()`       | 通知Scheduler指定的Executor已经以某个状态结束了 |
| `error()`              | 当Scheduler出现可恢复的错误时被调用    |

其中最重要的为
`resourceOffers(SchedulerDriver,  List<Offer>)`方法。Master向Framework提供资源offer，每一份资源offer包含从某个slave上获取的资源列表（包括CPU，内存，硬盘等）。开发者重写该方法，可用根据特定的情况接受资源offer，并利用offersId交给Executor启动任务，或者拒绝该offer。**一个offer会提供给多个Framework，但是第一个使用offer启动任务的Framework会获胜，其他的Framework将受到`offerRescinded()`发出的消息。**

**Scheduler API**

| 方法                   | 说明                                      |
| -                      | -                                         |
| `registered()`         | 当Executor驱动成功和Mesos连接上时调用     |
| `reregistered()`       | 当Executor再次向某个重启的slave注册时调用 |
| `disconnected()`       | 当Executor和slave失去连接时调用           |
| **`launchTask()`**     | 当Executor启动某个任务时调用              |
| `killTask()`           | 当Executor杀死某个task时被调用            |
| `frameworkMessage()`   | 当Framework的信息到达Executor时被调用     |
| `shutdown()`           | 当Executor需要终止所有task时被调用        |
| `error()`              | 当Scheduler出现不可恢复的错误时被调用     |

同样，Executor中最核心的方法是`launchTask(ExecutorDriver, TaskInfo)`。可重写该方法实现执行任务，任务将在offer来源的slave上执行。**执行任务在方法调用时会被阻塞，在回调完成前，该Executor无法执行其他的任务。** 因此如果要执行一个长时间的任务（如Web应用)，可用把任务放在线程里，通过线程的方式启动。

## Framework实例分析

Mesos上已有许多Framework，包括支持MPI，Hadoop，Spark，Storm等应用的。这里主要利用简单的Framework分析如何实现一个Framework。

* [mesos-hydra](https://github.com/mesosphere/mesos-hydra)，MPI Framework，该Framework是利用python进行的实现。

* [RENDLER](https://github.com/mesosphere/RENDLER),一个利用Mesos框架实现的分布式爬虫

```
挖坑，待看完全部代码后补上源码分析
```

## Framework设计模式
从现有的Mesos Framework分配调度使用资源的方式中，可用总结出以下几种模式。

* 用于协调资源：设计的Framework作为其他Framework的资源协调者，作为服务运行，如`Mesos-DNS`。
* 基于负载状况：设计的Framework根据负载来调节资源使用，如`Marathon`和`Aurora`，会根据约定自动进行扩容和缩容。
* 基于预留的框架：这里主要指，某些分布式应用，在设计时已经预留了资源调度的接口，如`Hadoop`，`Spark`，`Storm`，因此只要实现接口，就可以将应用从原有的调度框架（如Hadoop原有框架为`YARN`)移植到Mesos上来。

-----
## 相关资源
+ [Mesos项目官网](http://mesos.apache.org/)
+ [数人云翻译的Mesos文档](http://www.mesoscn.cn/)
+ [Mesosphere](https://mesosphere.com/)
+ [Marathon](https://mesosphere.github.io/marathon/)
+ [Aurora](http://aurora.apache.org/)
+ [Chronos](https://mesos.github.io/chronos/)
+ [InfoQ的Mesos迷你书](http://www.infoq.com/cn/minibooks/analyse-mesos)
+ [NSDI 2011 - Mesos: A Platform for Fine-Grained Resource Sharing in the Data Center ](http://people.csail.mit.edu/matei/papers/2011/nsdi_mesos.pdf)
