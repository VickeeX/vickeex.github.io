---
layout:     post
title:      Docker Checkpoint/Restore
subtitle:   Docker Container Checkpoint/Restore with CRIU
date:       2018-11-18
author:     VickeeX
header-img: img/post-bg-docker-logo-2.png
catalog: true
tags:
    - Docker
    - Docker Container
    - CRIU
    - Checkpoint
    - Restore
    - Checkpoint/Restore
---


唔，暂时小记一下checkpoint / restore，希望之后能回顾并深入认识下目前的问题。

### CRIU
CRIU全称“Checkpoint / Restore in Userspace”，是一个为Linux提供检查点/恢复功能的工具，主要是对运行中的应用进行冻结(freeze)再基于其在磁盘上的所有文件建立检查点，并根据checkpoint恢复冻结时状态并继续运行。CRIU可以运用到场景包括：应用热迁移（live migration）、快照、远程调试（debugging）等等。CRIU为OpenVZ、LXZ/LXD、Docker等都提供了很好的支持。

##### Checkpoint
/proc是一个基于内存的文件系统，包括CPU、内存、分区划分、[I/O地址]、直接内存访问通道和正在运行的进程等等，Linux通过/proc访问内核内部数据结构及更改内核设置等。Checkpoint很大程度上是基于/proc文件系统进行的，主要依赖/proc获取文件描述符信息、管道参数、内存映射等。
Checkpoint通过进程转存器(process dumper)进行以下步骤：
* 收集进程树并进行冻结
* 收集任务资源并进行转存（写入转存文件）
* 清理战场~~

##### Restore
Restore恢复过程主要进行以下步骤：
* 解决共享资源：CRIU读取镜像文件找出哪些进程共享哪些资源，共享资源由某个进程恢复后，其他进程继承或以其他方式获取。
* fork进程树：通过fork()函数创建待恢复的进程，但此时并没有对进程进行恢复。
* 恢复基本的任务资源：打开文件，准备namespaces，映射内存区域，创建套接字等。但是以下几类资源的恢复会等到下一个阶段：内存映射的确切位置，计时器，证书，线程。
* 切换到恢复点的上下文，恢复并继续执行。


### CRIU for Docker
Docker container实际上也是一个进程，故CRIU实质上是对容器进程进行checkpoint/restore。

##### 使用之前
源码装CRIU有一丢丢麻烦，记得把官网说的那些库都下完整哦。
docker虽然提供了checkpoint，但切换至experimental下才能用，新建/etc/docker/daemon.json文件，（docker的配置文件，默认没有）。
```
$ echo "{\"experimental\": true}" >> /etc/docker/daemon.json
$ systemctl restart docker
```
若该文件参数更改很多，就会起冲突......解决办法：尽量只将自己需要更改的配置参数写入就好，若还冲突，就启动docker时手动指定参数或脚本启动吧。

另外，我使用docker 18及之后的版本时，checkpoint无法正常使用，主要出现以下问题：
* 对一个运行的容器进行checkpoint后，该容器并没有自己停，马不停蹄地继续跑继续跑......但该容器/指定目录下还是有了对应的checkpoint文件夹。
* 执行checkpoint ls 时，无法正常获取指定容器已有的checkpoint及信息，显示：
```
Error response from daemon: open /var/lib/docker/containers/[CONTAINER_ID]/checkpoints/[CHECKPOINT_ID]/config.json: no such file or directory
```
* 尝试根据checkpoint恢复也不行，错误信息为：
```
Error response from daemon: failed to retrieve OCI runtime container pid: open /run/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/[CONTAINER_ID]/init.pid: no such file or directory: UNKOWN 
```
据说是???moby的原因，但看Stackflow上的问题也还是open的，关闭了一个但感觉他关的莫名其妙；有一个问题下，开发人员说解决了，但还未推到新版本。我的解决办法：试验之后，**建议使用较新版本17.06进行checkpoint/restore**，可以正常使用，可能18版本（小生年方18，尚未婚娶）太新了脚跟还没站稳。

##### Docker  CLI for checkpoint
现在可以开始愉快地使用docker checkpoint了！！Docker CLI提供了checkpoint命令。

**create**
```
$ docker checkpoint create [OPTIONS] CONTAINER CHECKPOINT
```
* --leave-running=false，checkpoint完成之后，容器继续运行还是停止（默认false）
 * --checkpoint-dir DIR_PATH，使用指定的目录（就不用存放在docker下了，好找些）

**ls**
```
$ docker checkpoint ls CONTAINER
```
**rm** 无话可说
**start**
启动时没有单独的命令，但在container start可以指定checkpoint选项参数，如将容器从/home/vickee/chkps/目录下的chkp0恢复：
```
$ docker start --checkpoint chkp0 --checkpoint-dir /home/vickee/chkps/[CONTAINER_FULL_ID]/checkpoints CONTAINER
```
注意：在创建checkpoint时，若我们指定的路径为/home/PATH，则恢复时还需要具体指定到该路径下的/home/PATH/[CONTAINER_FULL_ID]/checkpoints。因为恢复时，我们可能新建容器，或者将另一个容器从别的容器的checkpoint恢复，故需自己根据checkpoint信息进行路径完善。

##### limitations
CRIU对最新内核的支持有限，且好像在较新版本中，移除了--checkpoint-dir即指定目录这一特性。
若容器运行时有用external terminal（```docker run -t```），checkpoint会失败的。[ 参数-t 让docker分配一个伪终端并绑定到容器的标准输入上, -i 则让容器的标准输入保持打开，常一起使用。]

***links：***
https://criu.org/Docker
https://criu.org/Installation
https://criu.org/Checkpoint/Restore



