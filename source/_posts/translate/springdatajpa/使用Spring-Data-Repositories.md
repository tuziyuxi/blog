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

### 4.3.3. 使用多个Spring Data模块的Repositories

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

repository代理有两个方法从方法名派生出特殊存储的查询
* 直接从方法名派生查询
* 使用手工定义查询

可用选项取决于实际存储库。然而，必须有一个策略来决定被创建的实际查询。下一章节讨论可用选项。

### 4.4.1 查询查找策略

下列策略对于repository基础架构解决查询是可选择的。使用XML配置，你可以通过查询查找策略在命名空间配置策略。使用Java配置，你可以使用Enable${store}Repositories注解的查询查找策略属性。对于特定数据库有些策略可能不被支持。

* CREATE尝试从查询方法名称构建特殊存储库查询。一般方法是删除一组已知的方法名的前缀，并解析方法剩余部分。你可以在“Query Creation"读取更多关于查询构造的信息。
* USE_DECLARED_QUERY 尝试查找声明的查询，如果找不到，就抛出异常。查询可以通过某处的注解来定义，或其他方式声明。查询特定存储文档来查找该存储的有效的选项。如果repository基础架构在引导时如果不能找到方法的声明查询，就失败。
* CREATE_IF_NOT_FOUND(默认)结合了CREATE和USE_DECLARED_QUERY.它首先查找声明查询，然后如果声明查询没有找到，它创建一个基于名称查询的自定义方法。这是默认查找策略，因此，如果你没有明确配置任何策略，它就被使用。它允许通过方法名快速查询定义，还可以通过引入需要的声明查询来定义这些查询。

### 4.4.2. 查询创建

构建Spring Data repository基础结构的查询构建机制对于构建repository实体的约束查询是有用的。该机制从方法剥夺前缀（find...By,read...By,query...By,count...By,get...By）然后开始解析剩余部分。介绍的短语可以包含更多表达式，例如Distinct在被创建的查询中设置一个区别标志。但是，第一个By作为分隔符来标示查询条件的开始。在一个非常基础的级别，你可以在实体属性上定义条件，并用And和Or连接它们。下列例子展示如何创建一系列查询：

例子16.从方法名称查询构建
```
interface PersonRepository extends Repository<User, Long> {

  List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);

  // Enables the distinct flag for the query
 //可以在查询使用distinct标志
  List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
  List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);

  // Enabling ignoring case for an individual property
  //可以忽略单个属性的大小写
  List<Person> findByLastnameIgnoreCase(String lastname);
  // Enabling ignoring case for all suitable properties
  //可以忽略所有合适属性的大小写
  List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);

  // Enabling static ORDER BY for a query
  // 可以在查询使用静态ORDER
  List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
  List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
}
```

派生方法实际结果依赖你创建查询的持久化存储。然而，有一些共性要注意：
* 表达式通常是属性和可以被连接的操作相结合。你可以使用AND和OR连接属性表达式。你也可以获得属性表达式操作的支持，例如Between,LessThan,GreaterThan和Like。不同的数据库支持的操作不一样，所以要参考相关文档适当的部分。
* 方法解析器支持为各个属性设置IgnoreCase标志（例如，findByLastnameIgnoreCase(...)）或者为支持忽略大小写类型的所有属性（通常是String实例，例如，findByLastnameAndFirstnameAllIgnoreCase(...)）。是否支持忽略大小写对于不同存储会不相同，所以为特殊存储查询方法参考参考文档的相关章节。
* 你可以使用静态排序，通过添加OrderBy短语到引用一个属性的查询方法和提供一个排序指示（Ase或者Desc）.创建支持动态排序的查询方法，请看“Special parameter handling”。

### 4.4.3. 参数表达式

参数表达式可以只引用管理实体的直接属性，如之前例子展示那样。在查询创建时，你已经确定派生方法是管理域类的一个属性。但是，你也可以通过遍历嵌套属性来定义约束。考虑如下方法签名：
```
List<Person> findByAddressZipCode(ZipCode zipCode);
```
假设一个Person有一个带有ZipCode的Address。在这个例子中，方法创建属性遍历x.address.zipCode。解析算法首先把这个部分（AddressZipCode）当做属性，然后检查域类是否有这样一个属性（未大写）。如果算法成功，则使用该属性，如果不成功，算法在从右边驼峰大写部分拆分为一个头部和一个尾部，然后尝试找到相应的属性-在我们的例子中，分为AddresZip和Code。如果算法找到以那开头的属性，继续拿到尾部，从那里构建树，用刚才描述的方法拆分尾部。如果第一部分没有匹配，算法向左移动拆分点（Address,ZipCode）,然后继续。

尽管这在大多数情况下可行的，但是该算法还是有可能选择到了错误的属性。假设Person类也有一个addressZip属性。算法会在第一次拆分时匹配成功，选择到错的属性，然后失败（因为addressZip属性可能没有code属性）。

为解决这个模糊情况，你可以使用\_插入方法名以手工定义遍历点。所以我们的方法名会如下：
```
List<Person> findByAddress_ZipCode(ZipCode zipCode);
```

因为我们把下划线字符当作保留字符，我们强烈建议使用标准的Java命名规范（就是使用驼峰命名代替下划线在属性名中）。

### 4.4.4. 特殊参数处理

在查询中处理参数，在之前的例子中已经看到定义方法参数。除此之外，基础架构推荐确定的特殊类型，例如Pageable和Sort,用来在你的查询中动态分页和排序。如下示例展示了这些特征：

例子17： 在查询方法使用Pageable,Slice和Sort
```
Page<User> findByLastname(String lastname, Pageable pageable);

Slice<User> findByLastname(String lastname, Pageable pageable);

List<User> findByLastname(String lastname, Sort sort);

List<User> findByLastname(String lastname, Pageable pageable);
```
第一个方法让你给查询方法传递一个 org.springframework.data.domain.Pageable实例来给静态定义查询添加动态分页。Page有元素的总个数和可用的分页数。基础架构触发一个计数查询来计算实例的总数。这样可能是耗性能的（取决于使用的存储），你可以使用Slice代替。Slice仅知道是否有下一个Slice可用，这样在一个大的结果集中可能会高效点。

分页选项也可以通过Pageable实例处理。如果你只需要排序，给方法传递一个org.springframework.data.domain.Sort实例。正如你可以看到的，返回一个List也是可能的。在这种情况下，构建一个分页实例额外的元数据不会被创建（意味着必要的计数查询不会被发生）。相反，它限制查询查找实体给定的范围。

！找出一个实体查询你可以获得多少个分页，你需要触发额外的计数查询。默认情况下，这个查询从你实际触发查询中派生出来。

### 4.4.5. 限制查询结果

查找方法的结果可以通过使用first和top关键字来限制，这些关键字可以交换使用。一个可选的数值可以被添加到top和first来限定返回的最大结果。如果省略了数字，默认值是1。如下示例展示如何限制查询大小：

例子18： 限制使用Top和First查询的结果大小
```
User findFirstByOrderByLastnameAsc();

User findTopByOrderByAgeDesc();

Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);

Slice<User> findTop3ByLastname(String lastname, Pageable pageable);

List<User> findFirst10ByLastname(String lastname, Sort sort);

List<User> findTop10ByLastname(String lastname, Pageable pageable);
```
限制表达式也支持Distinct关键字。此外，对于查询限制结果集为一个实例，支持用Optional关键字包装结果。

如果pagination和slicing应用于限制查询分页（可用页数的计算），在限制结果中应用。

！通过使用Sort参数，限制结果结合动态查询，让你表达'K'最小和‘K'最大元素的查询方法。

### 4.4.6. 流式查询结果

查询方法的结果集可以通过使用Java 8的Stream<T>作为返回类型被递增处理。代替将查询结果封装在Stream，数据特殊存储方法用于执行流式查询，如下示例：

例子19. 使用Java 8 Stream<T>的流式查询结果
```
@Query("select u from User u")
Stream<User> findAllByCustomQueryAndStream();

Stream<User> readAllByFirstnameNotNull();

@Query("select u from User u")
Stream<User> streamAllPaged(Pageable pageable);
```

!Stream可能潜在包装底层数据特殊存储资源，所以必须在使用后关闭。你可以使用close()方法手动关闭Stream，也可以使用Java 7 的try-with-resources块，如下示例展示：

例子20. 在try-with-resources块中使用Stream<T>结果集
```
try (Stream<User> stream = repository.findAllByCustomQueryAndStream()) {
  stream.forEach(…);
}
```

!不是所有的Spring Data模块在现阶段支持Stream<T>作为返回类型

### 4.4.7. 异步查询结果

通过使用Spring的异步方法执行能力，repositories查询可以异步执行。这意味着方法调用时立即返回，而实际查询执行发生在被提交到Spring TaskExecutor的任务。异步查询执行和响应查询执行不同，并不能混合。参考特殊存储文档获得响应支持细节。如下示例展示一些异步查询：

```
//使用java.util.concurrent.Future作为返回类型
@Async
Future<User> findByFirstname(String firstname);               

//使用Java 8的java.util.concurrent.CompletableFuture作为返回类型
@Async
CompletableFuture<User> findOneByFirstname(String firstname); 

//使用org.springframework.util.concurrent.ListenableFuture作为返回类型
@Async
ListenableFuture<User> findOneByLastname(String lastname);    
```

## 4.5. 创建Repository实例
 
 在这章，为定义的repository接口创建实例和bean定义。一个方法是通过使用支持repository机制的Spring Data模块一起附带的Spring命名空间，尽管我们通常建议使用Java配置。
 
 ### 4.5.1 XML配置
 
 每个Spring Data模块包含一个让你定义Spring可以为你扫描的基本包的repositories元素，如下示例：
 例子21. 通过XML使用Spring Data repositories

```
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://www.springframework.org/schema/data/jpa"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/jpa
    http://www.springframework.org/schema/data/jpa/spring-jpa.xsd">

  <repositories base-package="com.acme.repositories" />

</beans:beans>
```

在前面的例子中，Spring扫描com.acme.repositories及子包下继承Repository或子接口的接口。对于每个找到的接口，基础结构会注册持久化特殊技术FactoryBean来创建处理查询方法调用的相应代理。每一个bean被用接口名注册，例如一个UserRepository接口会被注册名为userRepository的bean。base-package属性支持通配符，所以你可以定义扫描包的模式（正则表达式）。

使用过滤器

默认情况下，基础结构会选择位于配置基础包下继承持久化特殊技术Repository子接口的每一个接口，并为它创建bean实例。然而，你可能想要对某个接口为它们创建实例进行更细粒度的控制。这样做，在<repositories/>元素内使用<include-filter>和<exclude-filter>元素。语义完成等同于spring上下文中的命名空间。查看详情，为这些元素请看Spring 参考文档。

例子，从基础结构排除确认接口作为repository的bean,你可以使用如下配置：

例子22. 使用exclude-filter元素
```
<repositories base-package="com.acme.repositories">
  <context:exclude-filter type="regex" expression=".*SomeRepository" />
</repositories>
```
前面例子排除所有以SomeRepository结尾的接口被创建实例。

### 4.5.2. Java配置

repository基础结构也可以通过在Java配置类上使用特定存储的@Enable${store}Repositories注解来触发。对于基于Java的Spring容器配置的介绍，请看Spring参考文档中的Java配置。

启用Spring Data repositories的样例配置如下：

例子23. 基于注解repository配置的样例
```
@Configuration
@EnableJpaRepositories("com.acme.repositories")
class ApplicationConfiguration {

  @Bean
  EntityManagerFactory entityManagerFactory() {
    // …
  }
}
```

!前面的例子使用特殊的JPA注解，你可以根据你实际使用的存储模块改变。同样应用EntityManagerFactory bean的定义。查看特殊存储配置的章节。

### 4.5.3. 独立使用

你也可以在Spring容器外使用repository基础结构，例如在CDI环境。你依然需要一些Spring依赖在类路径下，但是，一般情况下，你也可以通过编程的方式构建repositories.提供的repository支持的Spring Data 模块附带一个持久化特定技术的你可以像下面使用的RepositoryFactory:

例子24. 单独使用repository工厂
```
RepositoryFactorySupport factory = … // Instantiate factory here
UserRepository repository = factory.getRepository(UserRepository.class);
```

## 4.6. Spring  Data Repositories的自定义实现

这章介绍repository自定义，以及碎片如何形成复合repository。

当一个查询方法要求不同的行为，或者不嗯能够从查询派生实现时，需要提供自定义实现。Spring Data让你提供自定义代码，并且整合通用的CRUD抽象和查询方法功能。

### 4.6.1. 定义单个repositories

使用自定义功能丰富repository,你首先要定义一个片段接口，然后实现自定义功能，如下列所示：

例子25. 自定义repository功能的接口
```
interface CustomizedUserRepository {
  void someCustomMethod(User user);
}
```
然后你可以让你的repository接口添加继承片段接口，如下示例：

例子26. 定义repository功能实现
```
class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  public void someCustomMethod(User user) {
    // Your custom implementation
  }
}
```

! 与片段接口对应的类名最重要的部分是Impl后缀

实现本身不依赖Spring Data,可以被当作普通的Spring bean。因此，你可以使用依赖注入行为注入引入到其他beans（例如一个JdbcTemplate）,在切面作为一部分，等等。

你可以让你的repository接口继承片段接口，如下示例：

例子27. 改变你的repository接口
```
interface UserRepository extends CrudRepository<User, Long>, CustomizedUserRepository {

  // Declare query methods here
}
```
使用repository接口扩展片段接口组合CRUD，自定义功能，并使它对客户端是可用的。

Spring Data repositories通过使用形成repository组合的片段来实现。Fragments(注：片段、碎片不知怎么翻译)是基本的repository，功能方面（例如QueryDsl）,自定义接口及其实现。每次你向你的repository接口添加一个接口时，通过添加一个fragment来增加组合。每个Spring Data模块都提供基本的repository和repository方面实现。

如下示例展示定义接口和它们的实现：

例子28. Fragments和它们的实现
```
interface HumanRepository {
  void someHumanMethod(User user);
}

class HumanRepositoryImpl implements HumanRepository {

  public void someHumanMethod(User user) {
    // Your custom implementation
  }
}

interface ContactRepository {

  void someContactMethod(User user);

  User anotherContactMethod(User user);
}

class ContactRepositoryImpl implements ContactRepository {

  public void someContactMethod(User user) {
    // Your custom implementation
  }

  public User anotherContactMethod(User user) {
    // Your custom implementation
  }
}
```

如下示例展示了继承了CrudRepository的自定义repository接口：

例子29. 改变你的repository接口
```
interface UserRepository extends CrudRepository<User, Long>, HumanRepository, ContactRepository {

  // Declare query methods here
}
```

Repositories可以被多个自定义实现组成，这些实现按声明的顺序导入。自定义实现有更高的优先级比起基本实现和repository方面。如果两个fragments有同样的方法签名，这个排序让你重写基本repository和切面方法和解决歧意。Repository fragments不被限制在一个repository接口使用。多个repositories也可以使用fragment接口，让你通过不同的repository重新使用自定义。

如下示例展示repository fragment和它的实现：

例子30. Fragments覆盖save(...)
```
interface CustomizedSave<T> {
  <S extends T> S save(S entity);
}

class CustomizedSaveImpl<T> implements CustomizedSave<T> {

  public <S extends T> S save(S entity) {
    // Your custom implementation
  }
}
```
如下示例展示使用之前repository fragment的repository:

例子31：自定义repository接口
```
interface UserRepository extends CrudRepository<User, Long>, CustomizedSave<User> {
}

interface PersonRepository extends CrudRepository<Person, Long>, CustomizedSave<Person> {
}
```

配置

如果你使用命名空间配置，repository基础结构尝试自动检测自定义实现fragments，通过扫描它找到repository包下的类。这些类需要遵循将命名空间元素的repository-impl-postfiex属性附加到fragment接口名的命名规范。这个后缀默认为Impl。如下示例展示了使用默认后缀的repository和自定义后缀值的repository.

例子32. 配置例子
```
<repositories base-package="com.acme.repository" />

<repositories base-package="com.acme.repository" repository-impl-postfix="MyPostfix" />
```

前面示例的第一个配置尝试查找一个叫com.acme.repository.CustomizedUserRepositoryImpl 类来作为自定义repository实现。第二个例子尝试查找com.acme.repository.CustomizedUserRepositoryMyPostfix。

解决歧义

如果多个匹配类名的实现在不同的包下找到，Spring Data使用bean名称去标识哪一个去使用。

给定前面展示的CustomizedUserRepository的如下两个自定义实现，第一个实现被使用，它的bean名称是customizedUserRepositoryImpl,匹配fragment接口（CustomizedUserRepository）添加的Impl后缀。

例子33.解决歧义实现
```
package com.acme.impl.one;

class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  // Your custom implementation
}
```
```
package com.acme.impl.two;

@Component("specialCustomImpl")
class CustomizedUserRepositoryImpl implements CustomizedUserRepository {

  // Your custom implementation
}
```
如果你用@Component("specialCustom")注解UserRepository,bean的名字加上Impl,然后就匹配定义在com.acme.impl.two的repository实现，从而代替第一个。

手动接线

如果你的自定义实现仅使用基于注解的配置和自动装配，前面展示的方法效果很好，因为它被当作任何一个其他Spring bean处理。如果你实现的fragment bean 需要特殊接线，你可以声明bean，并根据前面章节描述的约束命名它。然后基础结构通过名字手工定义bean的定义，而不是创建一个。如下示例展示如何手动连接自定义是实现：
 
 例子34. 手动连接自定义实现
```
<repositories base-package="com.acme.repository" />

<beans:bean id="userRepositoryImpl" class="…">
  <!-- further configuration -->
</beans:bean>
```

### 4.6.2. 自定义基础repository

前面章节介绍的方法要求当你想要自定义基本repository行为时自定义每个repository接口，以至于所有的repositories都受到影响。为了避免对所有repositories改变行为，你可以创建一个继承持久化特定技术的repository基本类的实现。然后这个类扮演一个repository代理的自定义的基本类，如下所示：

例子35. 自定义repository基础类
```
class MyRepositoryImpl<T, ID extends Serializable>
  extends SimpleJpaRepository<T, ID> {

  private final EntityManager entityManager;

  MyRepositoryImpl(JpaEntityInformation entityInformation,
                          EntityManager entityManager) {
    super(entityInformation, entityManager);

    // Keep the EntityManager around to used from the newly introduced methods.
    this.entityManager = entityManager;
  }

  @Transactional
  public <S extends T> S save(S entity) {
    // implementation goes here
  }
}
```

 ！类需要一个父类的构造器，特定存储repository工厂实现使用。如果repository基础类有多个构造器，覆盖采用EntityInformation和特定存储基础结构类的一个（例如一个EntityManager或者一个模板类）。
 
 最后一步是使Spring Data基础结构知道自定义的repository基础类。在Java配置中，你可以使用@Enable${store}Repositories注解的repositoryBaseClass属性来做，如下示例：
 
 例子36. 使用Java配置，配置自定义repository基础类
```
@Configuration
@EnableJpaRepositories(repositoryBaseClass = MyRepositoryImpl.class)
class ApplicationConfiguration { … }
```
在XML命名空间一个相应的属性是可用的，如下示例：

例子37. 使用XML配置一个自定义的repository基础类
```
<repositories base-package="com.acme.repository"
     base-class="….MyRepositoryImpl" />
```

## 4.7. 从聚合根发布事件

由repositories管理的实体是聚合根。在一个域驱动设计引用中，这些聚合根通常发布域事件。Spring Data 提供一个叫@DomainEvents的注解，你可以在你的聚合根的方法上使用它，尽可能使发布简单，如下示例：

例子38. 从聚合根暴露域事件
```
class AnAggregateRoot {

    //使用@DomainEvents方法可以返回单个事件实例或事件集合，它必须不带有任何参数
    @DomainEvents 
    Collection<Object> domainEvents() {
        // … return events you want to get published here
        // 返回要在此处发布的事件
    }

    // 所有事件都被发布后，我们有个被@AfterDomainEventPublication注解的方法。它可能被用来清理发布事件的列表（或其实用途）
    @AfterDomainEventPublication 
    void callbackMethod() {
       // … potentially clean up domain events list
    }
}
```
 
每次调用一个Spring Data repository的save(...)方法时，这些方法都会被调用。
 
## 4.8. Spring Data扩展

本章描述了一组Spring Data的扩展，可以使Spring Data在不同的上下文可以使用。目前，大多数集合都指向Spring MVC。

### 4.8.1 Querydsl扩展

Querydsl是一个框架，可以通过流式API构建静态类型SQL类查询。

几个Spring Data模块支持通过QuerydslPredicateExecutor整合Querydsl,如下示例：

例子39. QuerydslPredicateExecutor接口
```
package org.springframework.data.querydsl;

import com.querydsl.core.types.OrderSpecifier;
import com.querydsl.core.types.Predicate;
import java.util.Optional;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

public interface QuerydslPredicateExecutor<T> {

    //查找和返回匹配Predicate的单个实例
    Optional<T> findOne(Predicate var1);

    //查找和返回匹配Predicate的所有实例
    Iterable<T> findAll(Predicate var1);

    Iterable<T> findAll(Predicate var1, Sort var2);

    Iterable<T> findAll(Predicate var1, OrderSpecifier... var2);

    Iterable<T> findAll(OrderSpecifier... var1);

    Page<T> findAll(Predicate var1, Pageable var2);

    //返回匹配Predicate的实体个数
    long count(Predicate var1);

    // 返回是否存在匹配Predicate的实体
    boolean exists(Predicate var1);
}
```

确保支持Querydsl，继承QuerydslPredicateExecutor在你的repository接口，如下示例：

例子40. Quereydsl整合repositories
```
interface UserRepository extends CrudRepository<User, Long>, QuerydslPredicateExecutor<User> {
}
```

前面的例子让你使用Querydsl的Predicate实例写类型安全的查询，如下示例：
```
Predicate predicate = user.firstname.equalsIgnoreCase("dave")
	.and(user.lastname.startsWithIgnoreCase("mathews"));

userRepository.findAll(predicate);
```

### 4.8.2 Web支持

!本节包含Spring Data Web支持的文档，因为它在Spring Data Commons当前（或更高）版本实现。新引入的支持改变了很多东西，所以我们在[web.legacy]保留之前行为的文档。



