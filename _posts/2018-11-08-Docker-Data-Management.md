---
layout:     post
title:      Docker Data Management
subtitle:   
date:       2018-11-08
author:     VickeeX
header-img: img/post-bg-docker-logo-2.png
catalog: true
tags:
    - Docker
    - Storage driver
    - CoW
    - Copy-on-write
    - Volumes
    - Data Volumes
    - Bind Mounts
---


对image、container有了基本认知之后，继续学习下docker的数据管理。呼，天气变化很大，爱护自己，注意保暖，如果不注意可能就会感冒或者生病，啊有点小难受的。么么哒想想徐就不难受了。

先说说container内部的文件管理吧。

### Storage Driver
Container中创建的文件默认存储在可写层上，通过存储驱动（storage driver）进行管理。storage driver对镜像层和可写层都会进行管理，虽然具体实现有所不同，但都使用***层叠式（stackable）***的镜像层和***写时复制（copy-on-write, CoW）***策略。

##### container size
size: 每个容器的可写层所使用的磁盘数据总量
virtual size: 容器的只读镜像所使用的数据总量+可写层所使用的数据总量

若容器键存在镜像的共享，则总计使用的磁盘空间不需要重复计算共享镜像的大小。磁盘空间也不包括数据卷、checkpoints文件等。

##### CoW
Copy-on-write是一个文件共享和复制的高效策略。若镜像层/可写层需要从的较低层读取文件或目录，直接访问即可，若需要更改文件的话则先将该文件复制到自己的层再进行修改操作（第一次进行复制即可）。
* 共享：镜像层共享能够有效地减少磁盘空间、I/O带宽的占用，而且新创建的镜像也会只需要进行增量修改从而拥有较小的size。
* 复制：容器在需要进行文件更改时才拷贝到可写层，故可写层的大小也会保持比较小的规模。拷贝的具体实现根据storage driver而有所不同，操作顺序大致如下：
      *  从上至下搜索镜像层，找到需要更新的文件，并添加至cache；
      *  执行copy_up操作，将文件拷贝至可写层；
      *  对文件进行改动后，容器无法再看到底层镜像层中的该文件（只能从可写层中获取）。

##### different storage drivers
Docker自定义了不同存储驱动的使用优先级。首先选择使用较少配置的驱动，如btrfs或zfs，依赖于已配置完备的文件系统；其次选择性能和稳定性较好的，overlay2（docker CE的默认方式）、overlay、devicemapper。
选择合适的storage driver，需要先以所使用的docker版本、操作系统、分布式与否作为依据，另外一些存储驱动要求使用文件系统的特定格式或者额外的配置，此后再根据所需要的工作负载和稳定性进行选择。

### Data Volumes
容器可写层的文件读写速度很慢，而且在container退出后即丢失，同时容器外的进程很难获取容器内部文件，可写层也不适合用于大量写操作的应用，故docker还提供了**持久化存储**文件的方式——直接写入主机的文件系统。主要有两种方式，volumes和bind mounts(绑定挂载)，Linux系统还哈哈哈额外赠送了另一种tmpfs mount方式。
* **Volumes**: 存储在docker管理的主机文件系统上（如 /var/lib/docker/volumes），docker外部进程无法更改这些文件 —— 是比较好的数据持久化方式。
* **Bind mounts**: 可以存储在主机上任何位置，docker containers或外部进程都可以进行更改。
* **tmpfs mounts**: 写入主机内存(memory)，但不写入文件系统。
![data_volumes.png](https://upload-images.jianshu.io/upload_images/13962746-78792a0570b17c20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### volumes
Volumes是由docker进行管理的，和主机文件系统隔离开了。多个containers可以共同使用一个volume，要注意volume的存在是不依附于container的。
在创建container或者使用service时可通过选项参数进行volumes创建，也可单独使用docker volume create命令。如下，第一条命令创建了一个名为memeda的volume，第二条命令则运行容器时在memeda下又创建了一个/hh文件夹：
```
$ docker volume creat memeda
$ docker run -d -v memeda:/hh busybox ls/hh
```
volume通过名字区分，container之间共用volume的时候不要弄混了以至于文件覆盖咯。docker create命令提供 "--driver" 选项以使用volume driver，还可以通过 "--label" 设置信息。volume还有以下操作：

|operation | description | options |
| ------ | ------ | ------ |
| ls | 列出docker下的所有volumes | --filter 根据提供值(对driver/label/name等)进行筛选，--quiet 只列出volume名，--format |
| inspect | 获取volumes的详细信息 | --format |
| prune | 删除没有使用的volumes，即没有与任何containers进行关联 | --filter，--force 无需确认直接删除 |
| rm | 删除volumes，但无法删除正在被containers使用的volumes | --force |


##### Bind mounts




