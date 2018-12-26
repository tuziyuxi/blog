---
title: 实时流处理统一批处理的最后一块拼图——Pravega
date: 2018-12-20 17:04:49
categories:
- 大数据
tags:
- 大数据
- 实时流处理
- Pravega
---
原文地址[https://mp.weixin.qq.com/s/HK1bu1R3GpdrOkUYmCXgxQ]

Pravega 
当前的大数据处理系统无论是何种架构都面临一个共同的问题，即：“计算是原生的流计算，而存储却不是原生的流存储” 。Pravega 团队重新思考了这一基本的数据处理和存储规则，为这一场景重新设计了一种新的存储类型，即原生的流存储，命名为”Pravega”，取梵语中“Good Speed”之意。

{% asset_img pravega_icon.png  %}

## 1 在 Pravega 之前的流数据处理

### 1.1 流数据处理发展过程

1）批处理作业阶段

批处理作业：在一个或多个大数据集上运行的分布式计算

2） 微批（micro-batch）处理方法

在较短时间内累计的较小块上运行作业

Spark Streaming为代表的微批处理会以秒级增量对流进行缓冲，然后在内存中进行计算。

因缓存机制的存在，微批处理仍然有较高的延迟。


3）原生的流处理平台

* S4和Apache Storm

* Heron

与Storm兼容的同时性能更优

* Apache Samza和Kafka Stream

基于Apache Kafka消息系统来实现高效的流处理

### 1.2 Lambda大数据处理架构

{% asset_img lambda.png  %}
(详细设计后面补充)

批处理和流处理系统使用不同的框架。

将大数据平台分隔成了批处理层、流处理层和应用服务层。Lambda架构遵循读写分离，复杂性隔离的原则，整合了离线计算和实时计算，集合Hadoop，Kafka,Storm,Spark,Hbase等各类大数据组件，使得两种处理能够在高容错，低延时和可扩展的条件下平稳运行。

### 1.3 流和批本来就应该没有界限

* Kappa架构，将批处理层、流处理层简化为一致性的流处理
* Apache Flink新一代流处理框架的翘楚，设计遵循Dataflow模型，从根本上统一了批处理和流处理
* Apache Spark推翻了之前微批处理的设计，推出了Structured Streaming,使用表和SQL的概念进行处理的统一

Pravega设计宗旨是成为流的实时存储解决方案。应用程序将数据持久化存储到Pravega中，Pravega的Stream可以有无限制的数量并且持久化存储任意长时间，使用同样的Reader API提供尾读（tail read）和追赶读（catch-up read）功能，能够有效满足两种处理方式的统一。

通过将Pravega流存储与Apache Flink有状态流处理器相结合，所有写、处理、读和存储都是独立的、弹性的，并可以根据到达数据进行实时动态扩展。


## 2 流式存储的要求

* 能够将数据视为连续和无限的，而不是有限和静态的
* 能够通过自动弹性伸缩数据采集、存储和处理能力，与负载保持协调一致，持续快速地交付结果
* 即使在延迟到达或出现乱序数据的情况下，也能持续交互准确的处理结果

以Kafka和Pravega对比，看Pravega如何以今天存储无法实现的方式实现它们。

### 2.1 将数据视为连续和无限的

### 2.2 基于负载的自动（zero-touch）弹性伸缩特性（scale up/scale down）

### 2.3 连续处理数据生成准确的结果


## 3 系列文章















