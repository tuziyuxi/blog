---
title: Kafka官网学习笔记
date: 2019-04-10 20:43:50
categories:
- 大数据
tags:
- 大数据
- Kafka
---
Kafka版本2.0.0

Kafka用于构建实时数据管道和流应用程序。它具有水平可扩展性、容错性、快速性。

# 介绍

## Kafka是一个分布式流媒体平台。
有三个关键功能：
* 发布和订阅记录流
* 以容错的持久方式存储记录流
* 记录发生时处理流

通常用于构建两大类应用：
实时流数据管道和实时流应用程序

核心概念：

* Kafka作为一个集群运行在一个或多个可跨多个数据中心的服务器上
* Kafka集群以称为主题的类别存储记录流
* 每条记录由一个健，一个值和一个时间戳组成

4个核心API

* 生产者API

* 消费者API

* 流API
    允许应用程序充当流处理器，从一个或多个主题消耗的输入流，并产生一个输出流至一个或多个输出的主题，
    有效的变换所述输入流，以输出流
    
* 连接API
    允许构建和允许Kafka主题连接到现有的应用程序或数据系统中重用生产者或消费者。
    例如：关系数据库的连接器可能捕获对表的每个更改。
    
 客户端与服务端的通信协议：TCP。
 
 ## 主题和日志
 
 主题是发布记录的类别或订阅源名称。
 对于每个主题，Kafka集群都维护一个分区日志。
 使用可配置的保留期，持久保留所有已发布的记录。
 
 ## 分配
 
 分区分布在Kafka集群中的服务器上，每个服务器处理数据并请求分区的共享。
 每个分区都在可配置数量的服务器上进行复制，以实现容错。
 
 每个分区都有一个服务器充当“领导者”，零个或多个服务器充当“追随者”。领导者处理读写，追随者复制、备份。
 
 ## 地域复制
 使用MirrorMaker,可以跨多个数据中心或云区域复制消息
 
 ## 生产者
 将数据发布到他们选择的主题。生产者负责选择分配给主题中哪个分区的记录。
 
 ## 消费者
 消费者使用消费者组名称标记自己，并且发布到主题的每个记录被传递到每个订阅消费者组中的一个消费者实例。
 
 ## 多租户
 通过配置哪些主题可以生成或使用数据来启动多租户。（？）
 
 ## 担保
 
 ## Kafka作为消息系统
 
 ## Kafka作为存储系统
 
 ## Kafka用于流处理
 在Kafka中，流处理器是指从输入主题获取连续数据流，对此输入执行某些处理以及生成连续数据流以输出主题的任何内容。
 
 ## 把碎片放在一切
 消息系统+存储系统+流处理
 
 # 快速开始
  
  3 创建主题
  ```
kafka-topics --create --zookeeper ehdp-node-2:2181/kafka --replication-factor 1 --partitions 1 --topic petertest3
```
{% asset_img kafkatopic.png %}

{% asset_img kafkatopicsuc.png %}
 
 查看主题列表
 
 ```
kafka-topics --list --zookeeper ehdp-node-2:2181/kafka
```

4 发送一些消息
```
kafka-console-producer --broker-list ehdp-node-2:9092 --topic petertest
```
每行作为单独的消息发送，运行生产者，然后再控制台中输入消息以发送到服务器 
 
5 启动消费者

```
kafka-console-consumer --bootstrap-server ehdp-node-2:9092 --topic petertest --from-beginning
```

7 使用Kafka Connect导入/导出数据

Kafka Connect是Kafka附带的工具，可以向kafka导入和导出数据

8 使用Kafka Streams处理数据

Kafka Streams是一个客户端库，用于构建任务关键型实时应用程序和微服务，其中输入和输出数据存储在Kafka集群中。

# 2. API

生产者API、消费者API、流API、连接器API、管理客户端API

## 2.1 生产者API
```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.0.0</version>
</dependency>
```

## 2.2 消费者API

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.0.0</version>
</dependency>
```
## 2.3 流API
流API允许转化数据流从输入主题到输出主题（就是把输入主题的数据，经过处理，转化到输出主题）

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
    <version>2.0.0</version>
</dependency>
```

## 2.4 连接API


## 2.5 管理客户端API

支持管理和检查主题，代理，acls和其他Kafka对象。


# 3 配置

Kafka使用属性文件格式中的键值对进行配置。可以从文件或以编程方式提供这些值。

## 3.1 Broker(经纪人)配置
基本配置：
* broker.id : 此服务器的代理ID
* log.dirs : 保存日志数据的目录
* zookeeper.connect : Zookeeper主机字符串

## 3.2 topic级别的配置

话题相关的配置可以在创建时指定，也可以后面修改，没有指定，则使用服务器默认的配置。


## 3.3 生产者配置

## 3.4 消费者配置

## 3.4.1 新消费者配置

## 3.4.2 旧消费者配置

## 3.5 Kafka连接配置

## 3.6 Kafka流配置

## 3.7 管理客户端配置

# 4 设计

## 4.1 动机

把Kafka设计为实时数据处理统一平台

## 4.2 坚持

* 不要害怕文件系统

* 恒定的时间

## 4.3 效率

* 端到端批量压缩




































