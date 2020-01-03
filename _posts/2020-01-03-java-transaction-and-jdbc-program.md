---
layout: post
title: JAVA中JDBC的事务编程:| java
category: java
tag: [java]
---

这篇文章主要介绍了JDBC的事务编程相关。
本文分为以下几个部分：
1. JDBC中的事务编程
2. Spring中的事务编程


# 1.JDBC中的事务编程


## 1.1 自动提交

事务处理一定程度上和数据库实现相关，数据库有个auto-commit属性，一般而言，MySQL默认开启，Oracle默认关闭，另外auto-commit属性分为全局和会话级别，全局级别影响的是所有会话，单个会话级别只针对当前会话生效，这个属性的设置影响到了jdbc事务编程的方式，因为在自动提交模式下，任何一个语句都会作为一个单独事务立马提交，这样编程人员无法将多条语句作为一个事务统一提交，所以，在jdbc编程中，一般模式是将autocommit首先设置为false。也就是说，一个connection被创建时，默认是auto-commit模式，也即一个sql statement作为一个事务，执行完成后自动commit。如果支持多个statement组成一个事务，则要禁止auto-commit模式：
```
con.setAutoCommit(false);
```

## 1.2 事务隔离级别

Connection支持了标准的四种隔离级别，如果没有设置，则查询数据库的默认隔离级别，也可以用setTransactionIsolation方法手动设置：

```
conn.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);
```

JDBC对应的四种隔离级别由低到高如下：
```
(1) Read Uncommitted(未授权读取)：Connection.TRANSACTION_READ_UNCOMMITTED;
(2) Read Committed(授权读取)：Connection.TRANSACTION_READ_COMMITTED;
(3) Repeatable Read(可重复读取): Connection.TRANSACTION_REPEATABLE_READ;
(4) Serializable(序列化): Connection.TRANSACTION_SERIALIZABLE
```

## 1.3 事务提交与回滚

Connection使用commit和rollback方法支持事务的提交和回滚：

```
try {
	conn.setAutoCommit(false);
	// -- do query and update operation
	conn.commit();
} catch (Exception e) {
	conn.rollback();
}
```

## 1.4 保存点的设置和释放

JDBC使用setSavepoint和releaseSavepoint方法支持保存点的设置和释放。

```
try {
	conn.setAutoCommit(false);
	// -- update 1
	Savepoint s1 = conn.setSavepoint();
	try {
		// -- update2
	} catch (Exception e) {
		conn.releaseSavepoint(s1);
	}
	conn.commit();
} catch (Exception e) {
	conn.rollback();
}
```

# 2.Spring中的事务编程

事务的定义包含：事务的隔离级别、事务的传播属性、超时时间设置、是否只读

## 2.1 事务管理方式

spring支持编程式事务管理和声明式事务管理两种方式。

- 编程式事务：使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。
- 声明式事务：是建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。

显然声明式事务管理要优于编程式事务管理，这正是spring倡导的非侵入式的开发方式。声明式事务管理使业务代码不受污染，一个普通的POJO对象，只要加上注解就可以获得完全的事务支持。和编程式事务相比，声明式事务唯一不足地方是，它的最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。但是即便有这样的需求，也存在很多变通的方法，比如，可以将需要进行事务管理的代码块独立为方法等等。

声明式事务管理也有两种常用的方式，一种是基于tx和aop名字空间的xml配置文件，另一种就是基于@Transactional注解。显然基于注解的方式更简单易用，更清爽。

### 2.2 事务隔离级别

隔离级别是指若干个并发的事务之间的隔离程度。TransactionDefinition 接口中定义了五个表示隔离级别的常量：

- TransactionDefinition.ISOLATION_DEFAULT：这是默认值，表示使用底层数据库的默认隔离级别。对大部分数据库而言，通常这值就是TransactionDefinition.ISOLATION_READ_COMMITTED。
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED：该隔离级别表示一个事务可以读取另一个事务修改但还没有提交的数据。该级别不能防止脏读，不可重复读和幻读，因此很少使用该隔离级别。比如PostgreSQL实际上并没有此级别。
- TransactionDefinition.ISOLATION_READ_COMMITTED：该隔离级别表示一个事务只能读取另一个事务已经提交的数据。该级别可以防止脏读，这也是大多数情况下的推荐值。
- TransactionDefinition.ISOLATION_REPEATABLE_READ：该隔离级别表示一个事务在整个过程中可以多次重复执行某个查询，并且每次返回的记录都相同。该级别可以防止脏读和不可重复读。
- TransactionDefinition.ISOLATION_SERIALIZABLE：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

### 2.3 事务传播行为

事务的传播属性则是Spring自己为我们提供的功能，数据库事务没有事务的传播属性这一说法。

所谓事务的传播行为是指，如果在开始当前事务之前，一个事务上下文已经存在，此时有若干选项可以指定一个事务性方法的执行行为。在TransactionDefinition定义中包括了如下几个表示传播行为的常量：

- TransactionDefinition.PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。这是默认值。
- TransactionDefinition.PROPAGATION_REQUIRES_NEW：创建一个新的事务，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_SUPPORTS：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
- TransactionDefinition.PROPAGATION_NOT_SUPPORTED：以非事务方式运行，如果当前存在事务，则把当前事务挂起。
- TransactionDefinition.PROPAGATION_NEVER：以非事务方式运行，如果当前存在事务，则抛出异常。
- TransactionDefinition.PROPAGATION_MANDATORY：如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。
- TransactionDefinition.PROPAGATION_NESTED：如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。

### 2.4 事务超时问题

所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 TransactionDefinition 中以 int 的值来表示超时时间，其单位是秒。

默认设置为底层事务系统的超时值，如果底层数据库事务系统没有设置超时值，那么就是none，没有超时限制。

### 2.5 事务回滚规则

默认配置下，spring只有在抛出的异常为运行时unchecked异常时才回滚该事务，也就是抛出的异常为RuntimeException的子类(Errors也会导致事务回滚)，而抛出checked异常则不会导致事务回滚。可以明确的配置在抛出哪些异常时回滚事务，包括checked异常。也可以明确定义那些异常抛出时不回滚事务。还可以编程性的通过setRollbackOnly()方法来指示一个事务必须回滚，在调用完setRollbackOnly()后你所能执行的唯一操作就是回滚。

## 2.6 编程式事务

### 2.6.1 Spring事务管理的三大接口

- PlatformTransactionManager ： 事务管理器

```
public interface PlatformTransactionManager {

    //根据事务定义TransactionDefinition，获取事务
    TransactionStatus getTransaction(TransactionDefinition definition);

    //提交事务
    void commit(TransactionStatus status);

    //回滚事务
    void rollback(TransactionStatus status);
}
```

- TransactionDefinition ： 事务的一些基础信息，如超时时间、隔离级别、传播属性等。

常用的TransactionStatus接口实现为DefaultTransactionStatus：

- TransactionStatus ： 事务的一些状态信息，如是否是一个新的事务、是否已被标记为回滚

### 2.6.2 使用PlatformTransactionManager

```
DefaultTransactionDefinition definition =  new DefaultTransactionDefinition();
//TransactionDefinition.ISOLATION_READ_COMMITTED: 只能读取另一个事务已经提交的数据
definition.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
//TransactionDefinition.PROPAGATION_REQUIRED:当前没有事务的时候，创建一个，如果有则使用当前事务
definition.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
definition.setTimeout(3600);
PlatformTransactionManager transactionManager = new DataSourceTransactionManager(dataSource);
TransactionStatus status = transactionManager.getTransaction(definition);

try {
	//................业务代码区
} catch (TransactionException e) {
	transactionManager.rollback(status);
	throw e;
} catch (Exception e) {
	transactionManager.rollback(status);
	throw e;
}
```

### 2.6.3 使用TransactionTemplate

使用TransactionTemplate 不需要显式地开始事务，甚至不需要显式地提交事务。这些步骤都由模板完成。但出现异常时，应通过TransactionStatus 的setRollbackOnly 显式回滚事务。 TransactionTemplate 的execute 方法接收一个TransactionCallback或TransactionCallbackWithoutResult的实例。

- TransactionCallback 回调方式

TransactionCallback 包含如下抽象方法
``` 
Object dolnTransaction(TransactionStatus status)
```
该方法的方法体就是事务的执行体。  

- TransactionCallbackWithoutResult 回调方式

如果事务的执行体没有返回值，则可以使用TransactionCallbackWithoutResult类的实例。它包含如下抽象方法: 
```
void dolnTransactionWithoutResult(TransactionStatus status)
```
该方法与TransactionCallback的dolnTransaction 的效果非常相似，区别在于该方法没有返回值，即事务执行体无须返回值。 

定义如下：
```
@Autowired
TransactionTemplate transactionTemplate；
```
使用如下：
```
transactionTemplate.execute(new TransactionCallback() { 

	public Object doInTransaction(TransactionStatus status) { 
	try{ 
		//................业务代码区
	}catch (Exception e) { 
	   status.setRollbackOnly(); 
	} 
	
}); 
```

## 2.7 声明式事务

声明式事务有两种方式，一种是在配置文件（xml）中做相关的事务规则声明，另一种是基于 @Transactional 注解的方式。

**步骤一**：在 xml 配置文件中添加如清单 1 的事务配置信息。除了用配置文件的方式，@EnableTransactionManagement 注解也可以启用事务管理功能。这里以简单的 DataSourceTransactionManager 为例。

```
<tx:annotation-driven />
   <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
       <property name="dataSource" ref="dataSource" />
   </bean>
```

或者在启动类上开启事物支持 @EnableTransactionManagement

```
@EnableTransactionManagement //开启事物支持
@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootStudyApplication.class, args);
	}

}
```

提示：@EnableTransactionManagement注解其实在大多数情况下，不是必须的，因为SpringBoot在TransactionAutoConfiguration类里为我们自动配置启用了@EnableTransactionManagement注解。不过自动启用该注解有两个前提条件，分别是：@ConditionalOnBean(PlatformTransactionManager.class)和@ConditionalOnMissingBean(AbstractTransactionManagementConfiguration.class)，而一般情况下，这两个条件都是满足的，所以一般的，我们在启动类上写不写@EnableTransactionManagement都行。本人这里还是建议写出来。

**步骤二**：在业务逻辑层接口的实现类中的相关类或方法上使用@Transactional注解

注意事项：

- 1. Transactional 注解只能应用到 public 可见度的方法上。 如果应用在protected、private或者 package可见度的方法上，也不会报错，但是实际上事务设置不会起作用。

- 2. 使用@Transactional注释具体类（以及具体类的方法），而不是注释接口；注释具体类 ，则该类下public方法事务生效。

- 3. Transactional 注解的事物所管理的方法中，如果方法抛出运行时异常或error，那么会进行事务回滚；如果方法抛出的是非运行时异常，那么不会回滚。

- 4. 在很多时候，我们除了catch一般的异常或自定义异常外，我们还习惯于catch住Exception异常；然后再抛出Exception异常。但是Exception异常属于非运行时异常(即：检查异常)，因为默认是运行时异常时事物才进行回滚，那么这种情况下，是不会回滚的。我们可以在@Transacional注解中，通过rollbackFor = {Exception.class} 来解决这个问题。即：设置当Exception异常或Exception的所有任意子类异常时事物会进行回滚。











