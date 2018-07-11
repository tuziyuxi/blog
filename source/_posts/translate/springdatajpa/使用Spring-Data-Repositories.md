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


