---
layout:     post
title:      data model & data structure：I models
subtitle:   DDIA: Designing Data-Intensive Applications
date:       2018-09-08
author:     VickeeX
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - DDIA
    - data models
    - data structures
---


刚看完DDIA (Designing Data-Intensive Applications)第一部分，介绍了data system的基础，如：三大原则，基本/常见的数据模型 (models)，数据存储结构 (structures)。虽然暂时没有具体到很多细节或实例（后续），但也是理清了一些思路，不同于以前了解一个系统时看到一些名词的懵逼或混乱了。

我比较喜欢理清一些关键词的定义，再按树的结构往下走，文章组织形式以我所注重的点来吧~

# Definition
先弄清楚几个我之前有点混乱的概念，便于之后对数据系统进行学习。
**Storage engine**:数据系统内部用来进行数据增删改查的组件。

**Data Models**: 应用开发者所看到的数据形式，用户也将数据以这样的形式传递给数据库。如，用户看到的数据是K/V或表(table)形式。

**Data Structures**: 数据库层面(storage engine)的抽象结构，数据库内部进行存储和检索所依赖的数据组织形式。如，KV model的数据可能以“哈希索引”的方式进行组织索引。

**Data Orientation and clustering**: 关系数据库中可能以row-orientation(行方向)的方式存储数据，但也可以使用column-oriented(列方向)或correlational(相关性)的方式。这样将**不同类型的数据对象存在存储器中相邻位置的方式**可以有效地提高某些相关数据操作的性能，被称为"clustering"，集簇。

# Data Models
数据库常分为关系型数据库和NoSQL，较流行的NoSQL又主要有：K/V式，document(文档型), graph(图形), object(对象), tuple(元组)等等，见wiki。
**[** 注：此处我非常纠结，因为有一说将NoSQL主要分为四大类：KV，document, graph, column列式存储，然而我认为：NoSQL概念更倾向于指data models，列式存储为data structure的概念，不应将列式存储认为是一种NoSQL；后面还会涉及。**]**

先介绍两个概念: schema和query language，再根据书中的介绍简单列出几种数据模型的比较。

#### schema
   * explict, schema on write, 显示模式: 明确的数据格式，所有写入的数据都将遵循固定的形式（如，关系数据库中，所有数据都以表格、行的形式写入）
   * implict, schema on read, 隐式模式: 数据结构是隐式的，读取数据时才会遵循一定的数据格式进行处理，常用于数据形式不确定(动态)的情况

#### query language
   * declarative language，声明型查询：只需要指定所查询数据遵循的条件以及数据如何进行转换，如: **ans = σ<sub>types = “programmars”</sub> (person)**；由数据库的查询优化器决定如何选择索引、连接方法、各部分执行顺序等。适用于并行执行。
   * imperative language,命令式查询：按特定的数据顺序执行，难以实现多核或多主机上的并行，如:
```
def findProgrammars():
    ans = []
    for i in range(len(person)):
        if person[i].types == programmars:
            ans.append(person[i])
    return ans
```
#### compares

| mode | schema | query language | feature | relations fited |
| ------ | ------ | ------ | ------ | ----- |
| Relational | explict | declarative | 较好的支持连接(join)操作 | 适用于(数据间)多对一，简单的多对多关系 |
| Document | implict | imperative | 灵活、高效，更接近应用程序的数据结构 | 适用于一对多关系 |
| Graph | implict |  |  | 适用于多对多关系 |

