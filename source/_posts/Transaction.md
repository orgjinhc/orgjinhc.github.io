---
title: Transaction
author: 靳宏财
top: true
cover: true
toc: true
summary: 事务、分布式系统、分布式事务相关概念和实现方法
categories: 事务
tags:
  - 分布式系统
  - 分布式事务	
date: 2020年01月26日12:35:12

---

# 简介

做任何事前都需要先给自己定一个小目标，然后带着目标去学，在学的同时不断的提出问题，边学边想这样的学习方式最有效也记忆最深，学完后总结出来讲给身边的人，给他人讲明白，你才算是真正吸收了并转换为自己的东西。

# 目标

- 理解事物原则、实现原理

- 掌握Spring事物机制、实现

- 了解分布式系统

- 掌握分布式事务实现原理、方法、思想

# 问题

1. 事物是干什么的

2. 那些是事物要关心的问题

   - 数据的并发访问、修改

   - 不同请求之间的数据隔离

   - 多服务共同完成同一业务请求，保证都成功或都失败

   - 异常发生时数据的回滚

3. 单体架构如何解决数据一致性问题

4. 分布式架构下又如何解决

# 学习

先看下我们学习的路线 ![transaction](http://lion-heart.online/blog/2020-01-23-034516.png)

## 一、事物

### **事物是什么**

> 事物：通过一种**可靠、一致**的方式，访问和操作**数据库**中数据的**程序单元**

### **事物原则(ACID)**

> - **原子性（Atomicity）：事务中的所有操作作为一个整体像原子一样不可分割，要么全部成功,要么全部失败**
> - <font color=red>**一致性**(Consistency)</font>：**事务的执行结果必须使数据库从一个一致性状态到另一个一致性状态**
> - - **1.系统的状态满足数据的完整性约束(主码,参照完整性,check约束等)** 
>   - **2.系统的状态反应数据库本应描述的 现实世界的真实状态**
> - **隔离性（Isolation）：并发执行的事务不会相互影响,其对数据的影响和它们串行执行时一样**
> - **持久性（Durability）：事务一旦提交,其对数据库的更新就是持久的。任何事务或系统故障都不会导致数据丢失**

### 事务特点

1. **事务的四个原则中，一致性是事务的根本追求，而在某些情况下会对事务的一致性造成破坏**
   - **事务的并发执行**
   - **事务故障或系统故障**

2. **数据库系统通过`并发控制`和`日志恢复`来避免这种情况的发生**
   - **并发控制技术保证了事务的隔离性,使数据库的一致性状态不会因为并发执行的操作被破坏**
   - **日志恢复技术保证了事务的原子性,使一致性状态不会因事务或系统故障被破坏。同时使已提交的对数据库的修改不会因系统崩溃而丢失,保证了事务的持久性**

### 事务原则实现原理

- **事务的原子性是通过 undo log 来实现的**

- **事务的持久性是通过 redo log 来实现的**

- **事务的隔离性是通过 (读写锁+MVCC)来实现的**

- **事务的一致性是通过原子性，持久性，隔离性来实现的**

  

#### 原子性实现原理：Undo log

**在MySQL数据库InnoDB存储引擎中，还用Undo Log来实现`多版本并发控制(MVCC)`, 在操作任何数据之前，首先将数据备份到一个地方（这个存储数据备份的地方称为Undo Log）。然后进行数据的修改。如果出现了错误或者用户执行了ROLLBACK语句，系统可以利用Undo Log中的备份将数据恢复到事务开始之前的状态**

**注意：undo log是逻辑日志，可以理解为：**

- **当delete一条记录时，undo log中会记录一条对应的insert记录**
- **当insert一条记录时，undo log中会记录一条对应的delete记录**
- **当update一条记录时，它记录一条对应相反的update记录**



#### 持久性实现原理：Redo log

**和Undo Log相反，Redo Log记录的是新数据的备份。在事务提交前，只要将Redo Log持久化即可，不需要将数据持久化。当系统崩溃时，虽然数据没有持久化，但是Redo Log已经持久化。系统可以根据Redo Log的内容，将所有数据恢复到最新的状态**



#### 故障及故障恢复

- 事务的执行流程如下：
- - 系统会为每个事务开辟一个私有工作区
  - 事务读操作将从磁盘中拷贝数据项到工作区中,在执行写操作前所有的更新都作用于工作区中的拷贝. 
  - 事务的写操作将把数据输出到内存的缓冲区中,等到合适的时间再由缓冲区管理器将数据写入到磁盘

- 由于数据库存在立即修改和延迟修改,所以在事务执行过程中可能存在以下情况: 

- - 在事务提交前出现故障,但是事务对数据库的部分修改已经写入磁盘数据库中。这导致了事务的原子性

    被破坏

  - 在系统崩溃前事务已经提交,但数据还在内存缓冲区中,没有写入磁盘。系统恢复时将丢失此次已提交的

    修改。这是对事务持久性的破坏。





#### 故障及故障恢复

- 撤销事务undo:将事务更新的所有数据项恢复为日志中的旧值
- 重做事务redo:将事务更新的所有数据项恢复为日志中的新值
- 事务正常回滚/因事务故障中止将进行redo
- 系统从崩溃中恢复时将先进行redo再进行undo





### **事物隔离级别**

|    级别\问题     | 脏读 | 幻读 | 不可重复读 | 并发粒度 |
| :--------------: | :--: | :--: | :--------: | :------: |
| READ UNCOMMITTED |  ✅   |  ✅   |     ✅      | -------  |
|  READ COMMITTED  |      |  ✅   |     ✅      |  -----   |
| REPEATABLE READ  |      |      |     ✅      |   ---    |
|   SERIALIZABLE   |      |      |            |    -     |

#### **问题解释**

> + **脏读：事务读取到其他事务还`未提交的数据`，涉及到事务回滚，当前数据就是脏数据**
>
> - **不可重复读：两次读之间有其他事务对`共享数据`进行`修改`**
> - **幻读：两次读之间有其他事务进行增删动作，是在`可重复读`的事务隔离级别下会出现的一种问题，简单来说，`可重复读`保证了当前事务不会读取到其他事务已提交的 `UPDATE` 操作。但同时，也会导致当前事务无法感知到来自其他事务中的 `INSERT` 或 `DELETE` 操作，这就是`幻读`。**
> - **更新丢失：和别的事务读到相同的东西，各自写，自己的写被覆盖了。（谁写的快谁的更新就丢失了）**

#### **隔离级别如何做到解决上述问题的呢**

1. **LBCC：Lock Based Concurrency Control**
2. **MVCC：Multi Version Concurrency Control**

|    级别\读写     |             读             |             写             |
| :--------------: | :------------------------: | :------------------------: |
| READ UNCOMMITTED |           不加锁           |         行级共享锁         |
|  READ COMMITTED  |   行级共享锁（读完释放）   | 行级排他锁（事物结束释放） |
| REPEATABLE READ  | 行级共享锁（事物结束释放） | 行级排他锁（事物结束释放） |
|   SERIALIZABLE   |          表级共享          |          表级排他          |

上面的表格里又涉及到俩新概念：**`共享锁`、`排他锁`**

##### 锁

**定义：用于管理对`共享资源`的`访问控制`**

这里只介绍InnoDB存储引擎，InnoDB既支持行级锁（row-level locking），也支持表级锁（table-level locking），但默认情况下是采用行级锁，<font color = red size = 5 >**重点**</font>：InnoDB行锁基于索引，否则退化为表锁

##### 锁类型

| 锁粒度\区别 |                             特性                             |
| :---------: | :----------------------------------------------------------: |
|    表锁     | 开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低 |
|    行锁     | 开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高 |

***

|   锁类型   |                             定义                             |
| :--------: | :----------------------------------------------------------: |
|   共享锁   | 行级别读锁（S锁），多事物对共享资源共享同一把锁，阻塞其他类型的锁 |
|   排他锁   |              行级别写锁（X锁），事物独占排他锁               |
| 自增主键锁 |                                                              |
|   范围锁   |                                                              |
| 意向共享锁 |                表级别读锁（IS），提高加锁效率                |
| 意向排他锁 |                表级别写锁（IX），提高加锁效率                |

***

|            锁算法            |                             特性                             |
| :--------------------------: | :----------------------------------------------------------: |
|            记录锁            | 存在于包括`主键索引`在内的`唯一索引`中，锁定单条索引记录，可退化为临键锁 |
|        GAP（间隙）锁         | 存在于`非唯一索引`中，锁定`开区间`范围内的一段间隔，它是基于**临键锁**实现的 |
| Next-key（临键）锁：默认算法 | 特殊的**间隙锁**，通过**临键锁**可以解决`幻读`的问题。 每个数据行上的`非唯一索引列`上都会存在一把**临键锁**，锁住一段**左开右闭区间**的数据 |

**<font color = red size = 5>结论</font>：在根据`非唯一索引` 对记录行进行 `UPDATE \ FOR UPDATE \ LOCK IN SHARE MODE` 操作时，InnoDB 会获取该记录行的 `临键锁` ，并同时获取该记录行下一个区间的`间隙锁`**



## 二、Spring对事物的支持

### Spring事物机制

- **TransactionDefinition：`事物定义（隔离级别和传播行为）`**

  > ![TransactionDefinitionon](http://lion-heart.online/blog/2020-01-24-065702.png)

- **PlatformTransactionManager：`事物管理器`**

  > ![PlatformTransactionManager](http://lion-heart.online/blog/2020-01-24-065830.png)
  >
  > **几种常用的默认实现**
  >
  > > ```java
  > > DataSourceTransactionManager
  > > JpaTransactionManager
  > > JmsTransactionManager
  > > ```

- **TransactionStatus：`事物状态管理`**

  > ![TransactionStatus](http://lion-heart.online/blog/2020-01-24-065912.png)

### XA与JTA

**XA：一种协议或规范，XA协议采用`两阶段提交`方式来管`分布式事务`**

> 架构原理
>
> ![XA模型](http://lion-heart.online/blog/2020-01-24-080733.png)

**JTA：Java Transaction API Java根据XA规范提供的事务处理标准**

> **原理**
>
> ![JTA事物管理原理](http://lion-heart.online/blog/2020-01-24-075246.png)
>
> **核心组件**
>
> 1. AP
> 2. TransactionManager
> 3. ResourceManager
> 4. XID
>
> **弊端**
>
> 1. **两阶段提交**
> 2. **事务时间无法控制，很容易发生死锁或锁定时间太长**
> 3. **低性能、低吞吐量**
>
> **流程**
>
> 1. **第一阶段为 准备（prepare）阶段。即所有的参与者准备执行事务并锁住需要的资源。参与者ready时，向transaction manager报告已准备就绪。** 
>
> 2. **第二阶段为提交阶段（commit）。当transaction manager确认所有参与者都ready后，向所有参与者发送commit命令。** 
>
>    > **流程图以及流程如下**
>    >
>    > ![JTA原理](http://lion-heart.online/blog/2020-01-25-032734.png)
>    >
>    > ``` java
>    > 创建、启动TM
>    > o.s.t.jta.JtaTransactionManager          : Creating new transaction with name [类+方法]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT; ''
>    > c.a.icatch.imp.BaseTransactionManager    : createCompositeTransaction ( 10000 ): created new ROOT transaction with id 127.0.0.1.tm0000100003
>    > o.s.t.jta.JtaTransactionManager          : Participating in existing transaction
>    > c.a.icatch.imp.CompositeTransactionImp   : registerSynchronization ( com.atomikos.icatch.jta.Sync2Sync@66ea0a4d ) for transaction 127.0.0.1.tm0000100003
>    > 
>    > RM加入TM
>    > r$ExtendedEntityManagerInvocationHandler : Joined JTA transaction
>    > 
>    > 执行RM事务
>    > org.hibernate.SQL                        : insert into customer (id, password, role, user_name) values (null, ?, ?, ?)
>    > 
>    > 提交RM资源到TM
>    > c.atomikos.jdbc.AbstractDataSourceBean   : AtomikosDataSoureBean 'dataSource': getConnection ( null )...
>    > c.atomikos.jdbc.AbstractDataSourceBean   : AtomikosDataSoureBean 'dataSource': init...
>    > c.a.icatch.imp.CompositeTransactionImp   : addParticipant ( XAResourceTransaction: 3132372E302E302E312E746D30303030313030303033:3132372E302E302E312E746D31 ) for transaction 127.0.0.1.tm0000100003
>    > c.a.datasource.xa.XAResourceTransaction  : XAResource.start ( 3132372E302E302E312E746D30303030313030303033:3132372E302E302E312E746D31 , XAResource.TMNOFLAGS ) on resource dataSource represented by XAResource instance xads0: conn0: url=jdbc:h2:mem:testdb user=SA
>    > c.a.icatch.imp.CompositeTransactionImp   : registerSynchronization ( com.atomikos.jdbc.AtomikosConnectionProxy$JdbcRequeueSynchronization@7b60aac ) for transaction 127.0.0.1.tm0000100003
>    > c.atomikos.jdbc.AtomikosConnectionProxy  : atomikos connection proxy for conn1: url=jdbc:h2:mem:testdb user=SA: calling prepareStatement(insert into customer (id, password, role, user_name) values (null, ?, ?, ?),1)...
>    > c.atomikos.jdbc.AtomikosConnectionProxy  : atomikos connection proxy for conn1: url=jdbc:h2:mem:testdb user=SA: isClosed()...
>    > c.atomikos.jdbc.AtomikosConnectionProxy  : atomikos connection proxy for conn1: url=jdbc:h2:mem:testdb user=SA: calling getWarnings...
>    > c.atomikos.jdbc.AtomikosConnectionProxy  : atomikos connection proxy for conn1: url=jdbc:h2:mem:testdb user=SA: calling clearWarnings...
>    > c.atomikos.jdbc.AtomikosConnectionProxy  : atomikos connection proxy for conn1: url=jdbc:h2:mem:testdb user=SA: close()...
>    > c.a.datasource.xa.XAResourceTransaction  : XAResource.end ( 3132372E302E302E312E746D30303030313030303033:3132372E302E302E312E746D31 , XAResource.TMSUCCESS ) on resource dataSource represented by XAResource instance xads0: conn0: url=jdbc:h2:mem:testdb user=SA
>    > 
>    > TM提交阶段
>    > o.s.t.jta.JtaTransactionManager          : Initiating transaction commit
>    > com.atomikos.icatch.jta.Sync2Sync        : beforeCompletion() called on Synchronization: org.hibernate.resource.transaction.backend.jta.internal.synchronization.RegisteredSynchronization@40f07161
>    > c.a.icatch.imp.CompositeTransactionImp   : commit() done (by application) of transaction 127.0.0.1.tm0000100003
>    > c.a.datasource.xa.XAResourceTransaction  : XAResource.commit ( 3132372E302E302E312E746D30303030313030303033:3132372E302E302E312E746D31 , true ) on resource dataSource represented by XAResource instance xads0: conn0: url=jdbc:h2:mem:testdb user=SA
>    > com.atomikos.icatch.jta.Sync2Sync        : afterCompletion ( STATUS_COMMITTED ) called  on Synchronization: org.hibernate.resource.transaction.backend.jta.internal.synchronization.RegisteredSynchronization@40f07161
>    > o.j.s.OpenEntityManagerInViewInterceptor : Closing JPA EntityManager in OpenEntityManagerInViewInterceptor
>    > o.s.orm.jpa.EntityManagerFactoryUtils    : Closing JPA EntityManager
>    > ```
>
> **局限性**
>
> - **只适用单服务多数据源的场景**
>
> **扩展**
>
> > **多服务实现分布式事务架构**
> >
> > ![JTA实现多服务的分布式事务](http://lion-heart.online/blog/2020-01-25-071911.png)

**Spring分布式事务实现**

一致性选择

> 强一致性方案
>
> - JTA（性能差、只限于单服务内多数据场景）
>
> 弱、最终一致性方案
>
> - 最大努力一次提交（自己设计保证事务方法一致性）
> - 链式事物

场景选择

> - MQ - DB ： 最大努力一次提交
> - 多DB ： 链式事物

## 三、分布式系统

### 1.一致性理论

#### CAP理论

- **Consistency：一致性（分布式环境下多个节点的数据是否一致）**
- **Availability：可用性（分布式服务能一直保证可用状态。当用户发出一个请求后，服务能在有限时间内返回结果）**
- **Partition Tolerance：分区容错性（特指对网络分区的容忍性）**

#### BASE理论

- **基本可用（Basically Available）**：指分布式系统在出现故障时，允许损失部分的可用性来保证核心可用
- **软状态（Soft State）**：指允许分布式系统存在中间状态，该中间状态不会影响到系统的整体可用性
- **最终一致性（Eventual Consistency）**：指分布式系统中的所有副本数据经过一定时间后，最终能够达到一致的状态

### 2.一致性模型

- 强一致性：数据更新成功后，任意时刻所有节点的数据都是一致的，一般采用同步的方式实现。
- 弱一致性：数据更新成功后，系统不承诺立即可以读到最新写入的值，也不承诺具体多久之后可以读到。
- 最终一致性：弱一致性的一种形式，数据更新成功后，系统不承诺立即可以返回最新写入的值，但是保证最终会返回上一次更新操作的值

### 3.一致性引发的问题

> `一致性`其实又包含了**`数据一致性`**和**`事务一致性`**
>
> 数据一致性
>
> - 强一致性：数据更新成功后，任意时刻所有节点的数据都是一致的，一般采用同步的方式实现。
> - 弱一致性：数据更新成功后，系统不承诺立即可以读到最新写入的值，也不承诺具体多久之后可以读到。
> - 最终一致性：弱一致性的一种形式，数据更新成功后，系统不承诺立即可以返回最新写入的值，但是保证最终会返回上一次更新操作的值
>
> 事务一致性
>
> - 强一致性就是当前事务必须满足ACID所有事务原则
>
> - 弱一致性就是在外部服务没有完成之前，用户发起查询请求，可以查看单独的某个服务结果
>
> - 最终一致性是弱一致性的降级方案，如果某个事物发生不可逆的错误，最终要么要不回滚，要么人工介入解决问题

## 四、分布式事务

### 解决方案

1. 消息驱动的分布式事务实现（最终一致性）

   > 注意问题
   >
   > - 消息中间件需要支持事务
   > - 需要支持回滚操作
   > - 保证幂等性
   >
   > 问题处理方式
   >
   > 1. 定时任务获取所有超时订单，自动回滚操作
   > 2. 保存出错消息，人工处理

2. 事件溯源的分布式事务实现（最终一致性）

3. TCC（最终一致性）

   > - tryCharge()，参数检查、资源预留
   > - confirmCharge()，业务操作，保证幂等性
   > - commitCharge()，完成事务，保证幂等性

总结

- 事务同步
- 重试和幂等性
- 根据具体架构场景具体分析

## 五、分布式实践

- 保证高可用
- 保证事务同步
- 保证幂等性
- 合理设计流程

