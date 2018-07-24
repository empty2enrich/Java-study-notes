# Spring 事务

## 事务认知

事务的 ACID （Atomicity,Consistency,Isolation,Durability）

## 事务传播性

+ propagation_required : 如果当前没有事务,新建一个事务;若已存在,加入到存在的事务中,这是Spring 的默认配置
+ propagation_supports : 支持当前事务,若没有事务,以非事务方法执行
+ propagation_mandatory : 使用当前事务,若无事务就抛错。
+ propagation_required_new ： 新建事务,若存在当前事务,挂起当前事务
+ propagation_not_supported
+ propagation_never
+ propagation_nested

## 事务的隔离级别

+ read_uncommited
+ read_commited
+ repeatable read
+ serializable
+ 脏读、不可重复读、幻象读

## 事务的实现

+ 编程式事务管理
+ 基于 TransactionProxyFactoryBean 的声明式事务管理
+ 基于 @Transaction 的声明式事务管理
+ 基于 Aspectj AOP