---
layout:     post
title:      Docker Container
subtitle:   
date:       2018-11-06
author:     VickeeX
header-img: img/post-bg-docker-logo-2.png
catalog: true
tags:
    - Docker
    - Container
    - Docker Container
    - Storage driver
---

上一篇讲了docker image，接下来讲讲container。感觉“container 是image的运行实例”的说法有偏差，因为container不一定处于running/stopped状态，初始创建后的状态为“created”，就把container理解为运行某个应用程序所依赖的环境及其他资源的封闭集合吧。

### create a container
创建一个container，实际是在指定镜像的镜像层的最顶端添加一个可写层，也称为“container layer”。container所做的修改都会写入container layer，比如写/改/删新文件。这些镜像层和container层都由storage driver（存储驱动）负责管理。
使用create命令进行container创建，完成后返回container id。create命令的部分常用参数如下表，完成参数选项的请用--help查看或参考CLI docs。docker还提供了run命令，创建一个container并立即启动该容器，用法相似。

|options | usage | description |
| ------ | ------ | ------ |
| -a | --attach value | 关联至STDIN/STDOUT/STDERR |
| -c | --cpu-shares int | 共享CPU |
| -e | --env value | 设定环境变量 |
|  | --expose value | 暴露一个或一组端口 |
| -h | --hostname string | container host name |
| -i | --interactive | 如果没有attach则一直保持STDIN开放 |
|  | --ip/ip6 string | ipv4/ipv6地址 |
| -l | label value | 为container设置元信息 |
|  | --link value | 链接至另一个容器（依赖于其存在） |
| -m | --memory string | 设置内存大小 |
|  | --mount value | 挂载文件系统 |
|  | --name string | 指定容器名 |
|  | --network string | 网络连接方式：none / bridge / container / host / user-defined network |
| -P | --publish value | 将容器的端口发布(映射)至主机 |
| -p | --publish-all | 将容器所有端口映射至主机上随机的端口 |
|  | --read-only | 容器的根文件系统设为只读 |
|  | --rm | 容器退出时立即删除 |
| -u | --user string | username or UID |
| -v | --volume value | 绑定挂载数据卷 |
| -w | --workdir string | 指定工作路径 |
注：本来相对--link选项参数拓展讲一下容器的链接（即容器间存在依赖关系，container A的运行依附于container B的已经运行），然后感觉不会怎么用上，就此吧。


### other operations

| command | description | options |
| ------ | ------ | ------ |
| attach | 将输入/输出/错误流关联至一个运行中的容器 | --no-stdin 不关联输入 |
| exec | 正在运行的容器中执行一个额外的命令 |  --env=[], --user, --workdir |
| start | 启动一个或多个停止状态的容器 | --interactive |
| pause | 挂起容器中的所有进程，通过cgroups freezer实现 |  |
| stop | 发送"SIGTERM"信号停止容器，超时则直接kill掉 |  |
| kill | 发送"SIGKILL"（或其他信号）给一个或多个容器 | --signal 指定发送的其他信号, 默认为KILL |
| wait | 等待容器退出时返回状态码 |  |
| export | 以tar压缩包方式导出容器的文件系统 | --outputs 指定输出到文件而不是STDOUT |
| rm | 删除一个或一组容器 | --force 强制删除(通过"SIGKILL") , --link 删除链接, --volumes 删除容器卷|
| cp | 在容器和本地文件系统间进行文件/文件夹的拷贝 | --archive 拷贝所有的uid/gid信息, --follow-link 遵循源路径的符号链接 |
| rename | 重命名 |  |
| restart | 重启| --time 重启前等待kill的时间 |
| logs | 获取容器的日志信息 | --details, --since/until, --tail, --timestamps 显示时间戳 |
| top | 显示容器内运行的进程 |  |
| port | 列出容器的端口映射信息 |  |
| stats | 显示容器资源使用情况统计信息的实时流数据 | --all, --format |
| diff | 列出容器文件系统的文件/目录变化 |  |


### network
在创建container时提供了network的选项参数，网络连接正是容器最重要的配置之一。docker的网络系统可插拔，以网络驱动的形式进行管理，主要有以下几种方式：
* bridge: 默认的网络驱动方式，常在单独工作模式下的(standalone)容器里的应用需要连接时使用。
   *default: 若容器间使用了link，则可以共享环境变量。
   *user-defined: 比默认桥接拥有更好的隔离性和互操作性，提供自动DNS服务。
* host: 容器直接使用主机的网络，容器和主机间不再具有网络隔离性。
* overlay: 多个docker daemons或swarm服务间能互相通信，
* macvlan: 可以为container指定一个MAC地址以使其以物理设备的形式显示在网络上。
* none: 哈哈哈不用网络也是一种网络连接的方式。
* network plugins: 安装并使用第三方网络插件。

