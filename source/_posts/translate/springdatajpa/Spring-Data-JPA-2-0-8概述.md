---
title: Spring Data JPA 2.0.8概述
date: 2018-07-07 22:01:00
categories:
- Spring Data JPA 2.0.8 参考指南翻译
tags:
- Spring Data JPA
- 翻译
---

# 概述

Spring Data JPA是Spring Data大家族中的一员，可以轻松实现基于JPA的存储库。这个模块处理基于JPA的数据访问层的增强支持。
它使构建使用数据访问技术的Spring驱动的应用程序变得更加容易。

在相当长的一段时间内，实现应用程序的数据访问层是相当麻烦的。必须编写太多的样板代码来执行简单的查询，以及执行分页和审计。
Spring Data JPA旨在减少实际的工作量来显著改善数据访问层的实现。作为开发人员，你编写存储库接口，包括自定义查询方法，然后
Spring会自动提供实现。

# 特征

* 对构建基于Spring和JPA的存储库的复杂支持
* 支持Querydsl谓词，从而支持类型安全的JPA查询
* 透明的域类审计
* 分页支持，动态查询执行，集成自定义数据访问代码的能力
* 在引导时验证带@Query注释的查询
* 支持基于XML的实体映射
* 通过引入@EnableJpaRepositories,启用基于Java配置的存储库配置

# 快速开始

通过Spring Initializr引导你的应用程序



