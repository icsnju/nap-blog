title: Hotswap Agent 学习笔记
category: TVDCR
date: 2015-12-18
tags: [spring, transacion, dynamic reconfiguration]
author: Hooting
---
热部署是在不重启Java虚拟机的前提下，能自动侦测到class文件的变化，更新运行时class的行为。[Hotswap Agent](https://github.com/HotswapProjects/HotswapAgent)是一个支持热部署的开源项目，该项目的主要目的是为了避免低效的change->restart + wait->check开发生命周期。

<!--more-->

---

## Hotswap Agent 介绍
热部署是在不重启Java虚拟机的前提下，能自动侦测到class文件的变化，更新运行时class的行为。[Hotswap Agent](https://github.com/HotswapProjects/HotswapAgent)是一个支持热部署的开源项目，该项目的主要目的是为了避免低效的change->restart + wait->check开发生命周期。Hotswap Agent支持所有主要的框架， 比如：Hibernate、Spring、Jetty、ZooKeeper等。Hotswap Agent拥有如下特性。

* **增强的Java HotSwap** - 支持改变方法体，添加/重命名方法或属性。唯一不支持的操作是层次结构的变化（改变父类或删除接口）
* **自动配置** - 所有的本地类和已知的正在运行的Java应用程序的资源可以自动监控并重新加载（不包括在Jar文件里的类）
* **额外的类路径** - 需要改变存在依赖的jar包中的类，使用extraClasspath属性，添加任何目录作为类路径。
* **更改后重新加载资源** - 使用watchResources属性添加任何目录来监视资源的变化。
* **框架的支持** - 通过plugin system，Hotswap Agent支持许多框架。并且新的plugin可以很容易地添加。
* **快速** - 只有plugin被初始化消耗资源，其余时间它不消耗任何资源或减慢应用程序

## Hotswap Agent 安装

* 下载最新的[DCEVM Java Patch](https://github.com/dcevm/dcevm/releases)
* 运行下载的jar包(e.g. java -jar XXX-installer-light.jar)
* 选择本地JDK的安装路径。
* 下载最新的[Hotswap Agent jar](https://github.com/HotswapProjects/HotswapAgent/releases)，将它放在任意目录，比如：/home

## Hotswap Agent 使用
在运行java应用程序时，添加如下参数：`-XXaltjvm=dcevm -javaagent:PATH_TO_AGENT/hotswap-agent.jar=autoHotswap=true`
其中，参数`autoHotswap`指定当应用程序的某个class变化时，自动动态加载。

