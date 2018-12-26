---
title: ''Kudu介绍''
date: 2018-10-06 08:09:10
categories:
- Kudu 1.7.1 官方文档
tags:
- Kudu
- 翻译
---

# Apache Kudu介绍

Kudu是为Apache Hadoop平台开发的柱状存储管理器。Kudu有Hadoop生态系统应用的普通技术属性：运行在商业硬件上，可以横向扩展，支持高可用操作。

Kudu的设计使它与众不同，Kudu的一些好处包括：
* 快速处理OLAP工作负载
* 与MapReduce、Spark和其他Hadoop生态系统组件整合
* 与Apache Impala紧密整合，使它成为在Apache Parquet使用HDFS的一个好的，可变的替换方案
* 强大但灵活的一致性模型，允许你根据每个请求选择一致性要求，包括严格可序列化一致性选项
* 同时运行顺序和随机工作负载的强大性能
* 对于管理员，使用Cloudera Manager容易管理
* 高可用性。Tablet服务和Master使用筏共识算法，只要确保超过一半的可用副本的数量，tablet就可以读写。
* 结构化数据模型

通过整合所有这些属性，Kudu的目标是支持在当代Hadoop存储技术难以或无法实现的应用系列，Kudu是一个很好解决方案的应用的一些例子：
* 新到达的数据需要立即被提供到最终用户的报告（报表？）应用
* 必须同时支持时间序列应用
    
    查询大量历史数据
    
    关于必须非常快速返回的单个实体的粒度查询
    
* 使用预测模型做实时决策的应用程序，基于所有历史数据定期刷新预测模型

关于这些或其他场景的更多信息，请看示例用例


## Kudu-Impala整合特征
CREATE/ALTER/DROP TABLE
Impala支持使用Kudu作为持久层创建、修改和删除表，这些表和Impala其他表使用同样的内部和外部方法，允许复杂的数据整合和查询。

INSERT
可以使用和任何其他Impala表一样的语法将数据插入到Impala的Kudu表中，例如使用HDFS或HBase作为持久层的表

UPDATE/DELETE
Impala支持UPDATE和DELETE SQL命令逐行或批量修改Kudu表中存在的数据。SQL语法与现有标准尽可能相同。除了简单DELETE和UPDATE命令，你可以在子查询中使用FROM字句指定特殊连接。

灵活分区
和Hive表分区类似，Kudu允许你通过哈希或范围动态预分表为预定数量的tablets,以便在集群中均匀读写。你可以通过任意数量的主键列，任意数量的哈希，和可选的拆分行列表。详情看表设计

并行扫描
为了在现代硬件上达到最高可用性能，Impala使用Kudu客户端并行扫描多个tablets。

高效查询
在可能情况下，Impala推送谓词评估到Kudu，以便尽可能接近谓词评估的数据，在很多工作负载下，查询性能和Parquet相当。

关于使用Impala查询存储在Kudu中的数据的更多细节，请参考Impala文档。

## 概念和术语

列式数据存储
Kudu是列式数据存储，列式数据存储存储存储强类型列。通过适当设计，在分析和数据仓库工作负载而言是优越的，基于几个原因。

读取效率
对于分析查询，你可以读取单独一列，或一列的一部分，而忽略其他列。这意味着你可以在磁盘上读取最下数量的块来满足你的查询。使用基于行的存储，你需要读取整一行，即便你只需要返回几列的数据。

数据压缩
因为给定的列只包含一个类型的数据，基于模型的压缩比使用基于行的解决方案包含混合数据类型的压缩效率高几个数据级。结合从列读取数据的效率，压缩允许你从磁盘读取更少的块满足查询。

Table
Table是你数据存储在Kudu的位置。Table有模式和完全有序的主键，一个Table被拆分为称为tablets的片段

Tablet
一个Tablet是一个table的连续片段，类似一个分区在其他数据存储引擎或关系型数据库。给定的tablet在多个tablet服务间复制，在任何一个给定的时间点，这些副本中的一个被认为是领导tablet。任何副本可以被读取，然而写入需要在tablet服务中达成共识。

Tablet Server
Tablet Server为客户存储和提供tablet。对于一个给定的tablet,一个tablet server扮演领导，其他的扮演跟随副本。只有领导server 服务写请求，然而领导和跟随server都可以服务读请求，领导通过Raft Consensus Algorithm（筏共识算法）删选出来。一个tablet server可以服务多个tablets,一个tablet可以被多个tablet server服务

Master
master追踪所有tablets,tablet servers, Catalog Table,和集群中的其他元数据。在一个给定的时间点，只有一个master，如果这个master消失了，另一个master会通过筏共识算法选出来。

master也和客户端协调元数据操作。例如，当创建一个新表，客户端内部发送请求到master。master为新表写元数据到Catalog table,并协调在tablet servers上创建tablets过程。

所有master的数据存储在一个tablet,可以被所有其他候选master备份。

Tablet servers间隔一段时间对master进行心跳检查（默认一秒一次）

筏共识算法
Kudu使用筏共识算法作为一个方法保证容错和一致性，不管是规则的tablets还是master数据。通过筏，多个tablets备份选择一个领导，负责接受和写入跟随副本。一旦写入持久到多个副本中，就会通知到客户端。一个给定的有N个备份的组能够接受最多（N-1）/ 2个错误副本的写入。

Catalog Table
Catalog Table是Kudu元数据的中央位置。它存储关于tables和tablets的信息。Catalog Table不能直接读或写。代替的，只能通过暴露在客户端API的元数据操作来访问。

Catalog Table存储两类元数据类型：

* Tables 
    table模式、位置和状态
* Tablets
    存在tablets的列表，包括tablet servers每一个tablet的副本，tablet当前的状态，开始和结束关键字。
    
逻辑备份
Kudu备份操作，而不是磁盘操作。这称为逻辑复制，而不是物理复制。这样有几个好处：
* 尽管插入和更新操作通过网络传输数据，删除操作不需要移动任何数据。删除操作发送到每个tablet服务，执行本地删除。

* 物理操作，例如压缩，在Kudu中不需要通过网络传送数据。这和使用HDFS的存储系统是不同的，数据块需要通过网络传送来满足所需数量的副本

* Tablet不需要同时或在同一时间执行压缩，又或者在物理存储层保持同步。这降低了所有tablet servers在同时遇到高延迟的可能性，由于压缩和大量写入负载。
    
## 架构概述
下图显示了一个Kudu集群，拥有三个Master和多个tablet servers，每一个server服务多个tablets。它说明如何使用筏共识来允许master和tablet servers的领导和跟随者。此外，一个tablet server可以成为一些tablets的领导，和其他的跟随。
![](https://kudu.apache.org/docs/images/kudu-architecture-2.png)

## 例子用例

* 具有近实时可用性的流输入

* 具有各种访问模式的时间序列应用程序

* 预测建模

* 将Kudu中的数据与遗留系统相结合
    
    
    
    
    