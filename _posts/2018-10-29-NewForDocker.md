---
layout:     post
title:      New For Docker
subtitle:   Background And Basic Concepts
date:       2018-10-29
author:     VickeeX
header-img: img/post-bg-doccker-logo.png
catalog: true
tags:
    - Docker
    - Container
    - Virtual Machine
---

很久很久以前（哈，其实也两三年啦），就知道了docker，看运维组的同学讨论得热火朝天，简单地搭建过，但一直没有具体了解过，于是开始一个新的系列：Docker~~！！以“Docker热迁移”为最终目的，逐步了解、使用、深挖。
【徐最近工作好辛苦啊，心xing疼。】

## Before Docker
##### 容器 && 虚拟机
**虚拟机**：用VMWare或者VirtualBox或者openStack等新建虚拟机的时候，需要选择硬件的配置，如：内存大小、磁盘大小、磁盘是否拆分等等，可以感知到这种情况下所新建的虚拟机是对硬件资源进行了划分和隔离，虚拟机间几乎不存在资源共享。

**容器**：则是操作系统级别的轻量级虚拟化，仍与主机共享操作系统内核，只不过使用软件实现进程间的隔离，容器间可以共享部分系统资源。正如Docker的可爱logo一样，容器可以理解为集装箱，即运输货物的时候我们不用关心集装里面装的是可爱的水果还是花花草草（运行的是什么程序），只需要关心集装箱的装卸、运输（容器的构建、运行）就行了。

也许你碰到过这些问题：
* 一台机子上运行两个Web程序，依赖的同一个软件的版本不一样、访问端口冲突等等，都是问题啦。开虚拟机解决冲突？这么重口的嘛<naive>，毕竟虚拟机开销太大了，而且资源被独占以后利用率不高。
* 若把一个Web服务从Ubuntu上迁移到CentOS上（其他不同系统的迁移也是一样的），环境部署......哇太麻烦的了！后台开发时，从开发环境迁移到生产环境时，即使系统版本一样、软件版本不一样的话或者即使弄到一样也有莫名奇妙的问题。

用容器的话，就可以把环境一起打包啦，然后扔给各服务器，随时随地都能运行~~

##### Docker && LXC
docker啊其实就是个容器引擎，让我们能对容器进行管理。**LXC，Linux Container**：Linux内核的虚拟化技术，实现了进程隔离，但特定的LXC配置下的应用程序的执行仍然依赖于机器的特定配置。
Docker基于LXC做了大量的工作（优化）：
* 将应用程序所依赖于机器的配置进行抽象并一起打包，实现了跨硬件、跨配置的运行。
* 层级镜像的自动化构建，可以将应用的软件依赖、构建工具、安装包等一起打包成镜像；镜像还使用了版本管理，便于增量构建、回退版本等。
* 共享：可通过Docker Hub进行镜像共享，对开发者十分友好。
但其实有些场景使用Linux内核容器更好，比如单容器多程序，各有各的好处吧。

## Docker
##### 安装Docker
参见官方文档即可：https://docs.docker.com/install/linux/docker-ce/ubuntu/

##### what's in Docker
* Docker client： 通过Docker命令或RESTful API发起请求。
* Docker daemon：dockerd，服务端，监听Docker API请求，并对Docker对象（镜像、容器、网络、数据卷等）进行管理。
* Registries：存放镜像的仓库，官方的即Docker Hub。
* Doker objects
   ** container：容器，可以理解为一个针对特定应用的完整的、隔离的运行环境。镜像的运行实例：有状态的镜像，用户进程。
  ** image：镜像，运行环境的“静态”体现，一个包含了{运行特定程序需要的所有东西}的可执行包。
  ** services：通过多个daemon进行扩展，以swarm形式一起工作。

##### 底层技术
* Namespaces：将内核的全局资源做封装，使得每个Namesapce都有一份独立的资源，不同的进程使用同一种资源时不会互扰。Docker会为每个容器创建一组namespaces，主要包括：pid namespace（进程隔离），net namespace（网络接口），ipc namespace（InterProcess Communication资源），mnt namespace（文件系统挂载），uts namespace（Unix Timesharing System）。
* Control groups：用于限制和隔离应用对系统资源的使用。
* Union file systems
* Container format：Docker将namespaces、control grous、UnionFS包装成一种默认的container形式——libcontainer。
