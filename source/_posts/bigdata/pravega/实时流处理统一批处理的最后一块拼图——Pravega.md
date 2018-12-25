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

批处理和流处理系统使用不同的




