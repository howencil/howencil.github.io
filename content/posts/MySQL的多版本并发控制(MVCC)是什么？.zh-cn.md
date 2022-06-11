---
title: "MySQL的多版本并发控制(MVCC)是什么？"
date: 2022-06-09T14:51:20+08:00
draft: false
author: 插画师
tags: ["MySQL"]
categories: ["Tech"]
---

# 1.MVCC的实现

**MVCC是通过每行记录后面保存2个隐藏的列来实现的。这两个列，一个保存了行的创建时间，一个保存了行的过期时间（或过期时间）。当然存储的并不是实际的时间值，而是系统版本号。每开始一个新的事务，学生版本号都会自动递增。事务开始时刻的系统版本号会作为事务的版本号，用来和查询的每行记录的版本号进行比较。**

以**可重复读**隔离级别为例，MVCC具体是如何操作的

- **SELECT**

  InnoDB会根据以下2个条件检查每行记录：

  1. InnoDB只查找版本早于当前事务版本的数据行（也就是说，行的系统版本号小于或等于事务的系统版本号），这样可以确保事务读取的行，要么是在事务开始前就已经存在的，要么是事务自身插入或者修改过的。
  2. 行的删除版本要么未定义，要么大于当前事务版本号。这样可以确保事务读取的行，在事务开始之前未被删。除

- **INSERT**

  InnoDB为新插入的每一行保存当前系统版本号作为行版本号。

- **DELETE**

  InnoDB为删除的每一行保存当前系统版本号作为行删除标识。

- **UPDATE**

  InnoDB为插入的一行新记录，保存当前系统版本号作为行版本号，同时保存当前系统版本号到原来的行作为删除标识。保存这2个额外的系统版本号，使大多数读操作都可以不用加锁。这样设计使得读数据操作很简单，性能很好，并且也能保证只会读取到符合标准的行，不足之处是每行记录都需要额外的存储空间，需要做更多的行检查工作，以及一些额外的维护工作。

# 2.举例说明

```sql
create table mvcctest( 
id int primary key auto_increment, 
name varchar(20));
```

**transaction 1:**

```sql
start transaction;
insert into mvcctest values(NULL,'mi');
insert into mvcctest values(NULL,'kong');
commit;
```

假设系统初始事务ID为1；

| ID   | NAME | 创建时间 | 过期时间  |
| ---- | ---- | -------- | --------- |
| 1    | mi   | 1        | undefined |
| 2    | kong | 1        | undefined |

**transaction 2:**

```sql
start transaction;
select * from mvcctest;  //(1)
select * from mvcctest;  //(2)
commit
```

#### SELECT

假设当执行事务2的过程中，准备执行语句(2)时，开始执行事务3：

**transaction 3:**

```sql
start transaction;
insert into mvcctest values(NULL,'qu');
commit;
```

| ID   | NAME | 创建时间 | 过期时间  |
| ---- | ---- | -------- | --------- |
| 1    | mi   | 1        | undefined |
| 2    | kong | 1        | undefined |
| 3    | qu   | 3        | undefined |

事务3执行完毕，开始执行事务2的语句2，由于事务2只能查询创建时间小于等于2的，所以事务3新增的记录在事务2中是查询不出来的，这就通过乐观锁的方式避免了幻读的产生。

#### UPDATE

假设当执行事务2的过程中，准备执行语句(2)时，开始执行事务4：

**transaction session 4:**

```sql
start transaction;
update mvcctest set name = 'fan' where id = 2;
commit;
```

InnoDB执行UPDATE，实际上是新插入了一行记录，并保存其创建时间为当前事务的ID，同时保存当前事务处ID到UPDATE的行的删除时间。

| ID   | NAME | 创建时间 | 过期时间  |
| ---- | ---- | -------- | --------- |
| 1    | mi   | 1        | undefined |
| 2    | kong | 1        | 4         |
| 2    | fan  | 4        | undefined |

 事务4执行完毕，开始执行事务2的语句2，由于事务2只能查询到创建时间小于等于2的，所以事务修改的记录在事务2中是查询不到的，这样就保证了在两次读取时读取到的数据的状态或者说是内容是一致的。

#### DELETE

假设当执行事务2的过吹水，准备执行语句(2)时，开始执行事务5：

**transaction session 5:**

```sql
start transaction;
delete from mvcctest where id = 2;
commit;
```

| ID   | NAME | 创建时间 | 过期时间  |
| ---- | ---- | -------- | --------- |
| 1    | mi   | 1        | undefined |
| 2    | kong | 1        | 5         |

事务5执行完毕后，开始执行事务2的语句2，由于事务2只能查询创建时间小于等于2，并且过期时间大于等于2的，所以id=2的记录在事务2的语句2中，也是可以查询出来的。这样就保证了事务在两次读取时读取到的数据的状态或者说是内容是一致的。

> 参考文章：[MySQL MVCC的实现原理](https://www.jianshu.com/p/f692d4f8a53e)
