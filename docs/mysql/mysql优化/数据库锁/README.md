# 数据库锁

## 锁的分类

从对数据操作分为读锁和写锁

- 读锁(共享锁)：针对同一份数据，多个读操作可以同时进行而不会互相影响
- 写锁(排他锁)：当前写操作没有完成，不允许其他写操作和读操作

从数据操作粒度分为表锁和行锁



## 三锁

手动加锁

```mysql
lock table table_name1 read(write), table_name2 read(write), ..;
```

查看表加锁情况

```mysql
show open tables;
```

手动解锁

```mysql
unlock tables;
```



### 表锁(偏读)

偏向MyISAM存储引擎，开销小，加锁快；无死锁；

锁定粒度大，发生锁冲突概率最高，并发度最低。



#### 加读锁

session1为book表(**MyISAM引擎**)加了读锁后，可以读取当前表数据，但不可以修改当前表数据，也不可以查询其他表数据。此时session2可以读取book表数据，也可以读取其他表数据， 当想要修改book表数据会阻塞。

当session1为book释放锁之后，session阻塞的修改book表操作会执行。



#### 加写锁

session1为book表(**MyISAM引擎**)加了写锁后，可以读取和修改当前表数据，不可以查询其他表数据。此时其他session对book表的查询和修改会阻塞。



#### 总结

MyISAM在执行查询语句前，会自动给涉及的所有表加读锁。在执行增删查改前，会自动给涉及的表加写锁。



#### 分析表锁定

可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定。

```mysql
show status like 'table%';
```

- table_locks_waited：表示表级锁定争用而发生等待的次数(不能立即获得锁的次数，每等待一次锁值加1)，此值高则说明存在较严重的表级锁争用情况。
- table_locks_immediate：表示能够立即获得锁的次数

MyISAM的读写锁是写优先，这也是MyISAM不适合做写为主表的引擎。因为写锁后，其他线程不能做操作。高并发下使得查询很难获得锁，导致大量阻塞。



### 行表(偏写)

在InnoDB引擎，session1在修改一条记录的时候，会为这个记录加一个行锁。此时，其他session不能修改这个记录，但能够修改其他记录。

> 索引失效会导致行锁变表锁。



#### 锁定一行

```mysql
begin;
select * from student where id = 8 for update;
commit;
```



#### 总结

InnoDB存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能会比表锁要高一些，但是在整体并发处理能力是远远优于MyISAM的表级锁定的。当系统并发量较高的时候，InnoDB的整体性能和MyISAM相比就会有比较明显的优势了。



#### 行锁分析

通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况。

```mysql
show status like 'innodb_row_lock%';
```

对各个状态量的说明如下：

- Innodb_row_lock_current_waits：当前正在等待的数量
- Innodb_row_lock_time：从系统启动到现在锁定总时间长度
- Innodb_row_lock_time_avg：每次等待平均时间
- Innodb_row_lock_time_max：从系统启动到现在等待最长的时间
- Innodb_row_lock_waits：系统启动到现在总共等待的次数



#### 间隙锁

**什么是间隙锁？**

当我们用范围条件而不是相等条件检索数据，并请求共享锁或排他锁的时候，InnoDB会为符合条件的所有记录都加上锁。对于键值在这个范围，但是却不存在的记录，叫做间隙(GAP)。

InnoDB也会对这个间隙加锁，这个锁机制就是所谓的间隙锁(Next-Key锁)。



**危害**

间隙锁会有一个致命的弱点，间隙也会被加锁，导致锁定的时候无法插入检索范围内的记录。这在某些情况下可能会对性能造成很大的危害。



### 页锁

开销和加锁时间介于表锁行锁之间，会出现死锁。锁定粒度介于表锁行锁之间，并发度一般。