---
title: 使用Spring Data Repositories
date: 2018-07-07 22:01:00
categories:
- Spring Data JPA 2.0.8 参考指南翻译
tags:
- Spring Data JPA
- 翻译
---

# 4. 使用Spring Data Repositories  

Spring Data repository抽象的目的是极大减少要求实现不同持久化存储的数据访问层的样板代码的数量。

！Spring Data repository文档和你的模块
这章解释Spring Data repositories的核心概念和接口。这章信息来自Spring Data Commons模块。它使用Java Persistence API(JPA)模块的配置和代码示例。你应该调整XML命名空间声明和扩展类型为你使用特定模块的等效项。[Namespace reference](https://docs.spring.io/spring-data/jpa/docs/2.0.8.RELEASE/reference/html/#repositories.namespace-reference)覆盖XML配置，支持repository API的所有Spring Data模块都支持XML配置。[Repository query keywords](https://docs.spring.io/spring-data/jpa/docs/2.0.8.RELEASE/reference/html/#repository-query-keywords)覆盖了repository抽象大体支持的查询方法关键字。想要了解模块特定特征的详细信息，阅读该文档特定模块的章节。（注：感觉有点狗屁不通，却又无能为力。）

## 4.1 核心概念

Spring Data repository abstraction核心接口是Repository。 它将域类和域类的ID类型作为参数进行管理。此接口作为一个标记接口，用来获取使用的类型，并用来帮助你发现集成该类的接口。CrudRepository为正在被管理的类提供复杂的CRUD功能。

注开始：Repository接口，通过泛型，指定实体类和实体类ID类型，代码如下

```
package org.springframework.data.repository;

import org.springframework.stereotype.Indexed;

@Indexed
public interface Repository<T, ID> {
}
```

实现该接口的类有：

{% asset_img repository.png  %}

注结束。

例子3：CrudRepository接口

```
package org.springframework.data.repository;

import java.util.Optional;

@NoRepositoryBean
public interface CrudRepository<T, ID> extends Repository<T, ID> {

    //
    <S extends T> S save(S var1);

    <S extends T> Iterable<S> saveAll(Iterable<S> var1);

    Optional<T> findById(ID var1);

    boolean existsById(ID var1);

    Iterable<T> findAll();

    Iterable<T> findAllById(Iterable<ID> var1);

    long count();

    void deleteById(ID var1);

    void delete(T var1);

    void deleteAll(Iterable<? extends T> var1);

    void deleteAll();
}

```
