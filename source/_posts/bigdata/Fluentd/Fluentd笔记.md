---
title: Fluentd笔记
date: 2019-03-16 10:26:58
categories:
- 大数据
tags:
- 大数据
- 日志采集
- Fluentd
---

一 什么是Fluentd

Fluentd是一个开源数据收集器，它允许你统一数据收集和消耗，以便更好地使用和理解数据。

* 使用JSON进行统一日志记录
* 可插拔架构
* 所需的最低资源
* 内置可靠性

二 为什么使用Fluentd

* 统一记录层
* 简单易用而且灵活
* 开源
* 经验证的可靠性和性能
* 社区

三 快速入门指南

1）window安装Fluentd
[官网教程](https://docs.fluentd.org/v1.0/articles/install-by-msi)

# 连接到其他服务

在Fluentd中，数据输入/输出中最重要的部分是有插件管理的，每个插件都知道如何与外部端点连接，并负责管理传输数据流的管道。

插件以某种约定命名。

## 插件管理

使用td-agent-gem来管理Fluentd插件。

```shell
 sudo /usr/sbin/td-agent-gem install fluent-plugin-s3
```

# 配置语法

## 数据源

```xml
<source>

</source>
```

## 输出端点

```xml
<match>

</match>
```

# Fluentd事件

## 基本设置

## 事件结构

事件包括标签，时间和记录

## 处理事件

### 过滤器

事件遵循一步一步的循环，从上到下按顺序处理它们。

### 标签

解决配置文件的复杂性，并允许定义不遵循从上到下顺序的新路由部分，而是像链接引用一样。

### 缓冲区

## 结论

# 将Apache日志存储到MongoDB中

{% asset_img apacheToMongoDb.png.png %}

# 

