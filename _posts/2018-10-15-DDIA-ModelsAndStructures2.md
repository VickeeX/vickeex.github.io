---
layout:     post
title:      Data Models And Data Structures: Structures
subtitle:   Designing Data-Intensive Applications
date:       2018-10-15
author:     VickeeX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - DDIA
    - data models
    - data structures
---


上一篇留下了data structure部分，在这里继续完成。开始前，先介绍涉及到的一个概念：数据仓库。
## Data Warehouse数据仓库
**数据库**：服务于业务，进行基本的事务处理如数据的增删改查等操作。
**数据仓库**：常用于商业用途，提供复杂的数据分析和决策支持，提供直观的查询结果

**OLTP, online transaction processing**: 基于数据库的基本操作，对实时性要求比较高，需要高并发度。比如，四六级系统‘本应’可以很多人同时进行成绩查询。
**OLAP, online analytical processing**: 数据仓库的主要应用，一般基于比较大的数据量进行操作分析，不要求实时性。

ETL: extract-transform-load, 从数据库中抽取数据并进行系列处理（清洗、转换）且加载到数据仓库的技术。

## Data Structure
data structure，或者说是面向storage engine的数据结构，主要介绍以下两种类的三种具体结构：
* Log-structured school, 基于log的结构: **hash indexes**, **LSM-trees**。Log, 只可追加的一系列记录(records)，将系列的数据操作写入所存储的log文件。
* Page-oriented school, 基于分页的结构: **B-trees**

### Hash Indexes
最简单的哈希索引方式即：每个操作记录为一个键值对，键映射至数据所存储的位移量(byte offsets)。log文件则由这些键值对组成。
由于不断地追加写入log文件，可能使得磁盘空间不足，故将log文件分为segments(段)。当前写入的segment达到固定大小后，关闭此segment，后续的写操作又从另一个新的segment开始。
##### compaction and merging
将log分为segment后，通过(compaction)和合并(merging)操作来减少对磁盘空间的占用。
**compaction**: 在一个segment中，某几个写记录都是针对于同一个键，那么仅用保留最后一个。![compaction operation.png](https://upload-images.jianshu.io/upload_images/13962746-111cfabb0e367ad4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**merging**: 两个已经关闭的segment中可能存在对相同键的写入，则将两个segment合并为一个新的segment，对相同键的写记录保留最新的一次即可。![merging operation.png](https://upload-images.jianshu.io/upload_images/13962746-0b420385989cdd7f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

优点：append和merge都是有序的写操作，写速率很高；只可追加且关闭后不再修改的segment，使得并发控制和故障修复更为简单；不断的merge操作也避免了数据文件的分散。


### LSM-trees
先介绍LSM-trees(log-structed merge-tree)中的重要概念SSTables。

##### SSTables
不同于hash index中的log segment, SSTable(sortet string table)结构中，每个segment中的数据是有序的。优点如下：
* 更简单高效地完成segment merging。使用类似mergesort的算法实现，类似于有序数组的合并。
* 内存中不需要再保存所有的键索引，持有一部分即可。键查询时，根据已知索引的键进行有序范围内的查询即可。
* 进行数据压缩以节省磁盘空间和I/O带宽。根据内存中持有的索引，将键值对分组成块并进行压缩，压缩后再写入磁盘；每个内存中的索引指向一个压缩块的起始。

##### operations
LSM-trees包括**内存中的memtable**和**磁盘上的SSTable**两部分；memtable是一个平衡树的结构(如: 红黑树)。操作基本如下：
* write: 将数据追加到memtabel中，若memtable的大小达到了阈值，就将其写为SSTable文件以写入磁盘中；后续的新write则将数据写到一个新的memtable实例中。
* read: memtable --> 最新的segment --> 次新的segment --> ......依次往回查询。
* 后台一直运行着compaction 和 merging进程。

LSM-tree的特点在后面与B-tree比较时进行介绍。

### B-trees
以上两种为基于追加式log的方式，而page-oriented则是将数据分为page/block，在B-tree结构中:
* 有一个page为树的根页；页可以指向其它的页，页中包含多个键以及子页的索引；子页则负责以父页中某两个相邻键为边界的连续范围内的所有键。
* branch factor: 页中含有的子页的索引个数。
* 含有n个键的树的深度级别为O(log<sup>**n**</sup>)
![B-tree structure.png](https://upload-images.jianshu.io/upload_images/13962746-31c26e8d605703e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进行write操作时，从 root page向下定位到该key对应范围所在的page，若添加新key后该页的空间不足，则将其分为两个新页，并在父页中添加新页的索引；若父页的空间不足，则以类似的方式继续拆分且向上添加。![growing a B-tree by splitting a page.png](https://upload-images.jianshu.io/upload_images/13962746-1eecd49a0f6ad5c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由于这种方式是对数据进行原地修改，若进行写入时需要对多个页进行修改，修改到一半时，系统崩溃，就GG了。所以一般会采用**WAL**(write ahead log, 预写日志)：当有写操作时，先追加式地写入WAL，再写入磁盘。
此外，还可以采用latches(轻量级锁)来保证并发控制下的树的一致性。

### 比较LSM-trees和B-trees
* **读写速率**：LSM-trees有更快的写速率，B-trees有更快的读速率。
因为LSM-tree的进行追加写入即可，读取却要依次往前回溯；需要注意，LSM-tree后台的压缩合并进程可能会影响正在进行的读写操作的性能，并且占用磁盘带宽。B-trees的写可能会进行多个page的改动，速率较慢，但读取时进行有序键范围内的向下索引，效率较高。
* **空间使用**：LSM-trees进行数据压缩，比B-tree占用更少的磁盘空间；B-tree分割页时，新页的某些空间尚未被使用，还会产生一些空间碎片。
* **数据副本**：log式的存储会持有一个key的多个副本，而B-tree中每个key只存有一份，故B-tree适用于需要强事务语义(transaction semanctics，后续还会经常提及)的数据库。


more: 看完之后决定先读下基于LSM的Google Bigtable论文。