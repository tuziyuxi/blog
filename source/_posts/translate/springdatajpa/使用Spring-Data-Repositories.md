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
这章解释Spring Data repositories的核心概念和接口。这章信息来自Spring Data Commons模块。它使用Java Persistence API(JPA)模块的配置和代码示例。你应该调整XML命名空间声明和扩展类型为你使用特定模块的等效项。[Namespace reference](https://docs.spring.io/spring-data/jpa/docs/2.0.8.RELEASE/reference/html/#repositories.namespace-reference)覆盖XML配置，支持repository API的所有Spring Data模块都支持XML配置。[Repository query keywords](https://docs.spring.io/spring-data/jpa/docs/2.0.8.RELEASE/reference/html/#repository-query-keywords)覆盖了repository抽象大体支持的查询方法关键字。想要了解模块特定特征的详细信息，阅读该文档特定模块的章节。

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

    //保存实体
    <S extends T> S save(S var1);

    <S extends T> Iterable<S> saveAll(Iterable<S> var1);

    //返回ID标识的实体
    Optional<T> findById(ID var1);

    //判断是否存在给定ID的实体
    boolean existsById(ID var1);

    //返回所有实体
    Iterable<T> findAll();

    Iterable<T> findAllById(Iterable<ID> var1);

    //返回实体的个数
    long count();

    void deleteById(ID var1);

    //删除实体
    void delete(T var1);

    void deleteAll(Iterable<? extends T> var1);

    void deleteAll();
}

```

!我们还提供持久化特定技术的抽象类，例如JpaRepository和MongoRepository。除了相当通用的技术无关的持久化接口（如CrudRepository）,这些接口扩展了CrudRepository并且暴露了底层持久化技术的功能。

在CrudRepository之上，PagingAndSortingRepository抽象，添加额外方法来简化实体的分页访问。

例子4. PagingAndSortingRepository接口

```
package org.springframework.data.repository;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

@NoRepositoryBean
public interface PagingAndSortingRepository<T, ID> extends CrudRepository<T, ID> {
    Iterable<T> findAll(Sort var1);

    Page<T> findAll(Pageable var1);
}
```

访问User的第二页，每页20条记录，你可以像下面这样做：

```
PagingAndSortingRepository<User,Long> repository = //... 获得bean
Page<User> users = repository.findAll(new Pageable(1,20)); //从0开始
```

除了查询方法，还可以使用计数和删除查询的查询派生。以下列表展示了派生技术查询的接口定义：

例子 5. 派生
```
interface UserRepository extends CrudRepository<User, Long> {

  long countByLastname(String lastname);
}
```

## 4.2. 查询方法

标准的CRUD功能repositories通常对基础数据存储区进行查询。使用Spring Data，申明这些查询，有4个步骤：
    
1. 声明一个接口，实现Repository或它的子类，并指定要处理的域类和ID类型，如下示例：
```
interface PersonRepository extends Repository<Person, Long> { … }
```

2. 在接口声明查询方法
```
interface PersonRepository extends Repository<Person, Long> {
  List<Person> findByLastname(String lastname);
}
```

3. 设置Spring，为这些接口创建代理实例，不管是用JavaConfig或XML配置
a. 使用Java配置，创建一个类似下面的类：
```
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@EnableJpaRepositories
class Config {}
```
b.使用XML配置，定义一个类似如下的bean
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:jpa="http://www.springframework.org/schema/data/jpa"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/data/jpa
     http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

   <jpa:repositories base-package="com.acme.repositories"/>

</beans>
```

这个例子使用了JPA命名空间。如果你使用了任何其他存储的repository抽象，你需要修改为你存储模块合适的命名空间声明。换句话说，你需要替换jpa,例如改为mongodb.
另外还需注意，JavaConfig配置变量没有指定包，注解类的包被默认使用。如果需要指定包的扫描路径，使用特定数据存储的repository的@Enable${store}Repositories注解的basePackage...属性的一个。

4. 注入repository实例并使用它，如下示例：
```
class SomeClient {

  private final PersonRepository repository;

  SomeClient(PersonRepository repository) {
    this.repository = repository;
  }

  void doSomething() {
    List<Person> persons = repository.findByLastname("Matthews");
  }
}
```

如下的章节详细解释每一步骤：

* 定义Repository接口（4.3）
* 定义查询方法（4.4）
* 创建Repository实例（4.5）
* 自定义实现Spring Data Repository

## 4.3 定义Repository接口

首先，定义一个特殊域类接口。接口必须继承Repository,并且绑定域类和ID类型。如果你想要暴露域类的CRUD方法，继承CrudRepository代替Repository。

### 4.3.1 微调Repository定义

通常，你的repository接口继承Repository,CrudRepository,或者PagingAndSortingRepository。另外，如果你不想继承Spring Data接口，你也可以用@RepositoryDefinition注解你的repository接口。继承CrudRepository暴露一个完整的操作实体方法的集合。如果你更倾向于选择一些方法暴露，从CrudRepository复制一些方法到你的域类。

！这样做可以让你在提供的Spring Data Repositories功能上定义你自己的抽象类。

以下示例演示了如何选择暴露CRUD方法（在这例子里，暴露了findById和save方法）

例子7. 选择暴露CRUD方法

```
@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends Repository<T, ID> {

  Optional<T> findById(ID id);

  <S extends T> S save(S entity);
}

interface UserRepository extends MyBaseRepository<User, Long> {
  User findByEmailAddress(EmailAddress emailAddress);
}
```

在上面例子中，你为所有的域类repositories定义了公共基本接口，并且暴露了findById()和save()方法。这些方法被路由到Spring Data提供的你选择的存储的基本repository实现（例如：如果你使用JPA，实现就是SimpleJpaRepository）,因为它们匹配CrudRepository的签名。所以，现在UserRepository可以保存用户，通过id查询用户，并且触发查询通过邮件地址查询用户。

！中间的repository接口被@NoRepositoryBean注解。确保你给所有repository接口添加该注解，以保证Spring Data在运行时不会创建实例。

### 4.3.2  Repository方法的Null处理

从Spring Data 2.0开始，repository CRUD方法返回一个使用Java 8的Optional指示潜在空值的单个聚合实例。除此之外，Spring Data支持在查询方法返回如下包装类型：

* com.google.common.base.Optional
* scala.Option
* io.vavr.control.Option
* javaslang.control.Option(不推荐使用Javaslang)

或者，查询方法可以根本不使用包装类型。通过返回null来表明没有查询结果。保证返回集合，集合代替，包装类和流的Repository方法不会返回null，而是返回相应的空表示。查看“Repository查询返回类型”详情。

可空性注解

你可以通过使用Spring框架的可控性注解来表示repository方法的可空性约束。它们在运行时提供了友好的方法和opt-in null检查，如下：

* @NonNullApi:使用在包级别来声明参数和返回值的默认行为不接受和产生null值。
* @NonNull:用在参数和返回值，该值不能为null（参数和返回值不需要使用@NonNullApi）
* @Nullable:用在参数和返回值，该值可以为null

Spring注解是使用JSR 305注解的元注解（一种休眠但广泛传播的JSR）。JSR 305元注解让IDEA，Eclipse和Kotlin等工具厂商以通用方式提供null值安全的支持，而无须对Spring 注解进行硬编码支持。要对查询方法在运行时检查可空性约束，你需要在package-info.java使用Spring的@NonNullApi在包级别激活非可空性。如下示例：

例子8. 在package-info.java声明非可空性

```
@org.springframework.lang.NonNullApi
package com.acme;
```
一旦存在非空默认，repository查询方法在运行时获得可空性约束验证。如果查询执行结果返回了定义约束，会抛出异常。当方法返回null,却声明了非null时就会发生（默认情况下，注解定义在repository所在包上）。如果你想再次选择可以为空的结果，请在各个方法上使用@Nullable。使用本章开头提及的使用结果封装类可以继续如期望那样工作：一个空值转化为代表缺席的值。

如下示例展示了一些刚刚描述的技术：

例子9. 使用不同的可空性约束
```
//repository在一个我们已经定义飞空行为的包里
package com.acme;                                                       

import org.springframework.lang.Nullable;

interface UserRepository extends Repository<User, Long> {

  //当查询执行没有产生结果时抛出EmptyResultDataAccessException异常。当emailAddress为null时抛出IllegalArgumentException异常。
  User getByEmailAddress(EmailAddress emailAddress);                    

  //当执行没有产生结果返回null，emailAdress也可以接受null
  @Nullable
  User findByEmailAddress(@Nullable EmailAddress emailAdress);          

  //查询执行没有产生结果返回Optional.empty()(住:没有值的Optional,用isPresent()判断)
  //当emailAddress为null时，抛出IllegalArgumentException异常
  Optional<User> findOptionalByEmailAddress(EmailAddress emailAddress); 
}
```

基于Kotlin的Repositories可空性

（略）

### 使用多个Spring Data模块的Repositories

在你的应用程序中使用一个Spring Data模块使事情简单，因为在定义范围的所有repository接口都绑定到Spring Data模块。有时候，应用要求超过使用一个Spring Data模块。在这种情况下，repository在多个持久化技术之间必须区分定义。当在类路径下检测到多个repository工厂，Spring Data进入严格的repository配置模式。严格配置使用repository和域类详情来决定repository定义的Spring Data模块绑定：

1. 如果repository定义继承自特殊模块的repository,那么它是特定Spring Data模块的有效候选者。
2. 如果域类被特殊模块类型注解注解，那么它是特定Spring Data模块的有效候选者。Spring Data即接收第三方注解（例如JPA的@Entity），也会提供自己的注解（例如Spring Data MongoDB和Spring Data Elasticsearch的@Document）。

如下示例展示使用特定模块接口的repository(再这个例子中是JPA)：

例子11：使用特定模块接口的Repository定义
```
interface MyRepository extends JpaRepository<User, Long> { }

@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
  …
}

interface UserRepository extends MyBaseRepository<User, Long> {
  …
}
```
MyRepository和UserRepository在他们的类关系中继承了JpaRepository。他们是Spring Data JPA模块的有效候选者。

如下示例展示使用普通接口的repository:

例子12：使用通用接口的repository定义

```
interface AmbiguousRepository extends Repository<User, Long> {
 …
}

@NoRepositoryBean
interface MyBaseRepository<T, ID extends Serializable> extends CrudRepository<T, ID> {
  …
}

interface AmbiguousUserRepository extends MyBaseRepository<User, Long> {
  …
}
```
AmbiguousRepository和AmbiguousUserRepository在类关系中仅继承Repository和CrudRepository。当使用唯一的Spring Data模块这样是可行的，然而在多个模块无法区分这些repositories应该绑定到哪个特定的Spring Data。

如下示例展示使用注解域类的repository:

例子13：使用注解域类的Repository定义。
```
interface PersonRepository extends Repository<Person, Long> {
 …
}

@Entity
class Person {
  …
}

interface UserRepository extends Repository<User, Long> {
 …
}

@Document
class User {
  …
}
```
PersonRepository引用Person，它使用JPA的@Entity注解，所以这个repository明显属于Spring Data JPA。UserRepository引用User,它被Spring Data MongoDB的@Document注解。

下列不正确的示例展示一个repository使用混合注解的域类：

例子14.使用混合注解域类的Repository定义
```
interface JpaPersonRepository extends Repository<Person, Long> {
 …
}

interface MongoDBPersonRepository extends Repository<Person, Long> {
 …
}

@Entity
@Document
class Person {
  …
}
```
这个例子展示了一个域类即使用JPA注解，又使用Spring Data MongoDB注解。它定义了两个repositories,JpaPersonRepository和MongoDBPersonRepository。一个用于JPA，一个用于MongoDB。Spring Data不能够区分repositories部分，导致没有定义。
 
 
 严格repository配置使用repository类型详情和可区分的域类注解来鉴定Spring Data模块的repository候选人。在同一个域类使用多个持久化特定技术的注解是可能的，也允许跨多个持久化技术重用域类。然而，Spring Data不再能够决定绑到repository的唯一模块。
 
 区分repository的最后一种方法是通过限定repository基础包的范围。基础包定义了扫描repository接口定义的起点，意味着repository定义放置在相应的包中。默认情况下，注解驱动的配置使用配置类的包。基于XML配置中基础包是必须的。
 
 如下示例展示基础包的注解驱动的配置：
 
 例子15：基础包的注解驱动配置
 ```
@EnableJpaRepositories(basePackages = "com.acme.repositories.jpa")
@EnableMongoRepositories(basePackages = "com.acme.repositories.mongo")
interface Configuration { }
```

## 4.4.定义查询方法

