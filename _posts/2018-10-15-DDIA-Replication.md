---
layout:     post
title:      Replication In Distributed Systems
subtitle:   Designing Data-Intensive Applications
date:       2018-10-15
author:     VickeeX
header-img:
catalog: true
tags:
    - DDIA
    - replication
    - leader-based replication
    - multi-leader replication
    - leaderless replication
---

正文之前小吐槽下：上课很心累，作业太多了，而且很多作业的意义不大(mei you yi yi)。某课程的实验要求写的莫名其妙，连给的镜像资源也莫名其妙。不过好像这样也许有让我更为珍惜和利用可以自主学习的时间吧。最近还有简单看（仅限于了解）强化学习，希望后面也能写一点小记录。

这两天看了些数据的备份方式，总结下leader-based replication(主从复制)和leaderless replication(去中心复制)，前者又包括single-leader和multi-leader两类。先说说为什么需要副本。
 ***scalability***: 数据集过大或读写请求过多时，单节点负载过大，通过多节点来均衡负载。
***fault tolerance / high availability***: 系统中一个或多个节点崩溃、网络或某个datacenter故障时，冗余备份的节点可以接任其工作，系统继续正常运行。
***latency***: 物理上更靠近用户的数据备份能有效减少延时。

## Leader-based Replication
每个存储了一份数据拷贝的节点被称作replica。某个replica被指定为leader(master / primary)，负责客户端的写请求，并最先进行数据更新。其他的副本则被称为followers(slaves / secondaries)。leader会将writes以replicaton log或change stream的方式发送给followers，以便各从节点进行本地的数据修改。读请求可发送给leader或followers。

**小八卦**：在 Twitter 炮轰下，Redis 作者被迫将 Master/Slave 架构改名为 Master/Replica，以避免让人联想到奴隶制。有趣的知乎链接：https://www.zhihu.com/question/294200413/answer/489899388

#### synchronously and asynchronously replication
一个系统中可能会有一些从节点采用同步复制方式，另一些采用异步复制方式。
**sync**: 主节点等待从节点完成数据写入并返回写的确认回执，才向写请求的客户端发送回执。主从一致，但从节点崩溃或网络故障都容易导致主节点被阻塞。
**async**: 主节点发送给从节点写的信息后，立即向客户端发送确认回执，不需要等待从节点的回复。主节点崩溃后，还未写入从节点的writes永久丢失。
[ semi-sync: 系统中有一个节点为同步复制，其余为异步，若同步的节点down掉了，则选择一个异步节点改为同步设置。 ]
* ***replication lag***：若用户从一个异步的节点中读取数据，可能由于该节点落后于leader而得到过时的数据，即数据不一致。虽然这种不一致是暂时的，会达到最终一致。

#### setting up new followers
扩大数据备份量或有节点崩溃时，需要加入新的从节点，一般不会直接通过拷贝所有数据来完成新的从节点的设置。通常步骤如下：
* 在某些时间点对leader数据进行一致性快照；
* 将快照复制给新的从节点；
* 该从节点向leader请求快照之后的所有数据修改。快照一般与replication log的某个位置相关联，以便进行定位操作；
* 该从节点完成快照后的所有数据修改后，即认为是追赶上进度了(caught up)，可以开始工作。

#### handling node outages
若从节点故障，通过catch-up recovery，即根据本地log文件，向leader请求错失过的操作，补回来。主节点故障，就麻烦了，通过failover(故障转移)修复：
* 确认leader挂掉，大部分系统都通过timeout机制来确认：超时未回复则认为节点挂了。
* 选举一个新的leader，一般选取数据最接近已挂主节点的那个节点。
* 重新配置系统以便新leader开始工作：新的写请求发送给新leader，从节点也需要更新leader信息以便交互。
[ 其实failover机制挺复杂的，想深入需要另外具体去了解。 ]

#### replication logs
replication logs的实现有多种方法，将常用的几种简单介绍下。
* **statement-based replication**
基于语句的复制（关系数据库中所用），leader记录下执行的每个写请求，并将写语句发送给从节点。但若语句中存在一些不确定因素(如：调用时间、随机数等不确定函数，或与环境相关的语句等)，复制会失败。可以通过替换不确定函数来解决这个问题。

* **WAL shipping**
Write-ahead log shipping, 预写日志传输。写操作追加到一个log中，write-ahead是指先将操作记录写入log再将数据更改写入磁盘。主节点将日志复制给从节点，从节点根据日志操作生成一份数据副本。但日志通常将数据描述的很详细，如某磁盘块的某字节被改变，若主从节点的存储引擎等版本不一致，则会出错，这个小细节不利于运维。

* **logical log replication**
解决以上WAL的主从存储引擎版本不一致的问题，将日志更改为logical log(逻辑日志)，与存储引擎的物理数据表示所不同。比如关系数据库，粒度为行，一条记录可以为：插入/删除/更新一行；即一个修改多行的transaction也会被分开为基于单行的好几条记录。

* **trigger-based replication**
基于触发器的备份。以上各方法都在数据库内部完成，没有应用层的代码，而有时会需要更灵活的处理。如数据冲突时，将replication移到应用层以便添加解决冲突的逻辑代码。不过，灵活性也会带来更大的开销或其他问题。

******
助教下课太早了，这两个小时的收获就是骑了两趟自行车当运动，以及王一多帮我做了两张专属表情包。回来继续。
******

## Single-leader Replication
没有什么需要特别单独说的了，基本同上，在leader-based中讲了。

## Multi-leader Replication
系统中有多个数据中心(datacenters)，每个datacenter有一个leader。
[建议了解下概念：cluster, datacenter, cloud。没错，我没写就是因为我还没弄明白。]

#### conflict resolution
在多个leader的情况下，若vickee同学更改了某条数据为“你很漂亮”，写请求由leader A完成，同时xu同学更改了该条数据为“你很帅”并且写请求发送到leader B处理，那么AB都会在写完后向对方发送自己的write操作进行更新，冲突就来了！到底是漂亮还是帅？
解决冲突一般以单行或一个单独的文档作为基本单位，而不是基于transaction，即transaction里的writes被认为是解决数据冲突时相互独立的单位。

* **sync and async detection**
同步方式即确认数据复制到了其他datacenter上，才返回给用户写成功的消息，不过这样就失去了多leader的优势了——各leader不再相互独立。
异步方式即两个leader上的写入都会成功，在之后的某时间点检测到冲突再解决，不过一般这时候再叫用户来解决就为时已晚啦。

* **conflict avoidance**
聪明的做法是把冲突扼杀在摇篮里：将针对某个record的写入全路由到同一个leader上，由此避免冲突。但当某个datacenter挂掉或用户物理位置改变时，需要重新路由到另一个leader，也可能发生并发写入。

* **converging toward a consistent state**
让数据达到一致，常用方法如下:
** 给每个write赋予一个ID(比如：时间戳、随机数等)，数据冲突时，ID更大的那个为winner（last write wins: ），即为保留的数据。
** 给每个replica一个ID，ID越高则其对应的数据也有更高优先级，不过也意味着某些数据的无条件丢失。
** 合并：将冲突的值排序并连接起来，如“你很漂亮/帅”。
** 记录冲突及所有信息，在之后通过应用代码解决。

* **custom conflict resolution logic**
用户编写代码自定义解决冲突的逻辑，分为on write（写时解决）和 on read（读时解决）。
** on write: 修改数据时若检测到了冲突，则立即调用conflict handler处理。
** on read: 检测到冲突时，将不同的writes都存储下来，有读请求时再将这多个版本一并返回给用户，由用户决定如何解决。

* **额外一提: automatic conflict resolution，可能会是未来的一种趋势，有兴趣可以去了解一下**

#### topologies
节点之间发送writes的路径结构，有circular topology（一个单向路径圈），star topology（一个中心节点，与其他所有节点互相传送），all-to-all topology（两两之间全是路径）。

#### comparison
**performance**: single-leader中，所有writes都经过一个leader，延迟会很大。multi-leader中，writes请求由本地的datacenter请求，并异步复制到其它datacenters。
**tolerance of datacenter outages**: single-leader中，leader一旦挂掉，需要另推选一个leader，会很难受。而multi-leader中，某个leader挂掉了，其他的datacenter还能继续独立完成工作。
**tolerance of network problems**: datacenter之间网络堵塞时，single-leader中的leader会比较难受，因为有同步写操作。而multi-leader就无所谓了，采用异步复制，不会受什么影响。

## Leaderless Replication
去中心化复制中，客户端写请求会发送给多个节点，多数写成功则视为成功。同样地，读请求也会从多个节点上读取，通过版本对比采纳最新的那份数据。那么多少个节点写成功算成功了，读的时候请求几个节点又该有多少份数据为最新的才能认为数据可以采用呢？**Quorums for reading and writing**，了解一下，这儿我就先不写了，可能以后也不会写......
如果一个节点崩溃了（leaderless没有failover），恢复正常时有以下两种方式进行数据修复：
**read repair**: 读请求会发送给多个节点，读请求操作会自带检测操作，若发现某节点的数据版本过时了，就将刚读到的最新数据发送给它进行更新。
**anti-entropy**: 后台有个进程会持续地检查各副本间的数据差异，并通过复制拷贝消除这种不一致，但数据复制前会有明显的间隔时间。