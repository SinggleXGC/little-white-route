# 查询截取分析

## 查询优化

先来看看一条慢查询SQL的分析：

1. 让程序运行一天，查看生产的慢SQL情况
2. 开启慢查询日志，设置阙值。比如超过5秒钟的就是慢查询，把它抓取出来
3. explain + 慢SQL分析
4. show profile查询SQL在MySQL服务器里面的执行细节和生命周期情况
5. SQL数据库服务器的参数调优



### 小表驱动大表

小表驱动大表，即小的数据集驱动大的数据集



```mysql
select * from A where id in (select id from B);
```

B表驱动着A表，当B表的数据集小于A表的时候，使用in性能优于exists



```mysql
select * from A where exists (select 1 from B where B.id = A.id);
```

这种情况是A表驱动B表。当A表的数据集小于B表的时候，此时使用exists性能优于in



### order by关键字优化

order by子句，尽量使用index方式排序，避免使用FileSort方式排序。

MySQL支持两种方式的排序，FileSort和Index，Index效率高，它指MySQL扫描索引本身完成排序。FileSort方式效率较低。

order by满足两种条件会使用index排序：

1. order by 语句使用索引最左前列

2. 使用where子句与order by子句条件列组合满足索引最左前列。

3. 如果不在索引列，filesort有两种算法：单路排序和双路排序

   **双路排序**：MySQL4.1之前是使用双路排序，字面意思就是两次扫描磁盘，最终得到数据。

   读取行指针和order by列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取输出。

   从磁盘读取排序字段，在buffer进行排序，再从磁盘取其他字段。

   **单路排序**：再mysql4.1之后出现了第二种改进的算法，就是单路排序

   从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出。它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO，但是它会使用更多的空间。

   

   单路排序的性能会比多路排序的性能好一些，但是也存在问题：

   在sort_buffer中，单路排序要比多路排序占用很多空间，单路排序把所有字段取出，可能会出现取出数据大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据进行排序，排完再取sort_buffer容量大小的数据进行排序，...从而导致多次IO。这样性能反而会比多路排序更低。

   **解决方案**：

   1. 增大sort_buffer_size参数的设置
   2. 增大max_length_for_sort_data参数的设置

   

#### 提高order by速度

1. order by时，不要使用select *，尽量只查询需要的字段

   - 当查询字段大小总和小于max_length_for_sort_data，而且排序字段不是TEXT|BLOB类型时，会使用单路排序，否则使用双路排序。
   - 两种算法的数据都有可能超出sort_buffer的容量，超出之后会创建tmp文件进行合并排序，导致多次IO，但是用单路排序算法的风险会大点，所以要提高sort_buffer_size

2. 尝试提高sort_buffer_size

   不管用哪种算法，提高这个参数都会提高效率，当然，要根据系统的能力去提高。因为这个参数是针对每个线程的

3. 尝试提高max_length_for_sort_data

   提高这个参数，会增加使用单路排序的可能性。但如果设的太高，数据总容量超出sort_buffer_size的概率就增大，明显症状是高的磁盘IO活动和低的磁盘利用率。



### group by 优化

1. group by的实质是先排序后分组，遵循最左前缀
2. 当无法使用索引列，增加sort_buffer_size和max_length_for_sort_data
3. where高于having，能在where限定的条件不要在having写



## 慢查询日志

MySQL慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超出阈值的语句。具体指运行时间超出long_query_time值的SQL，会被记录到慢查询日志中。默认值是10秒。

默认慢查询日志是关闭的，如果不是调优需要，不建议开启。开启慢查询日志会带来性能消耗。



### 查看是否开启及如何开启

查看

```mysql
SHOW VARIABLES LIKE '%slow_query_log%';
```

开启，只对当前数据库生效，重启后失效

```mysql
set global low_query_log = 1;
```

永久生效，修改配置文件my.cnf

[mysqld]下增加参数，重启mysql服务器

```cnf
slow_query_log = 1
slow_query_log = /data/mysql/xgc-slow.log
```

默认慢查询日志hostname-slow.log



### long_query_time修改

查看

```mysql
SHOW VARIABLES LIKE 'long_query_time%';
```

**大于，而不是大于等于long_query_time**



设置3秒，需要重新连接才能看到修改值

```mysql
set global long_query_time = 3;
```



### 查看慢查询记录数

```mysql
show global status like '%Slow_queries%';
```



### 日志分析工具mysqldumpslow

- s：是表示按照何种方式排序
- c：访问次数
- l：锁定时间
- r：返回记录
- t：查询时间
- al：平均锁定时间
- ar：平均返回记录数
- at：平均查询时间
- t：及为返回前面多少条的数据
- g：后面搭配一个正则表达式，大小写不敏感的



#### 工作参考

1. 得到返回记录集最多的10个SQL

   ```mysql
   mysqldumpslow -s r -t 10 /opt/mysql/xgc-slow.log
   ```

2. 得到访问次数最多的10个SQL

   ```mysql
   mysqldumpslow -s c -t 10 /opt/mysql/xgc-slow.log
   ```

3. 得到按照时间排序的前10条里面含有左连接的查询语句

   ```mysql
   mysqldumpslow -s t -t 10 -g "left join" /opt/mysql/xgc-slow.log
   ```

4. 另外建议在使用这些命令时结合|和more使用，否则有可能出现爆屏情况

   ```mysql
   mysqldumpslow -s r -t 10 /opt/mysql/xgc-slow.log | more
   ```



## Show Profile

### 什么是show profile

show profile是mysql用来分析当前会话中语句执行的资源消耗情况。可以用于sql的调优的测量。

默认情况是关闭的，并保存最近15次的运行结果。



### 分析步骤

1. 查看当前mysql版本是否支持

   ```mysql
   show variables like 'profiling';
   ```

2. 开启

   ```mysql
   set profiling = on;
   ```

3. 运行SQL

4. 查看结果

   ```mysql
   show profiles;
   ```

   ```mysql
   show profile cpu, block io for query 3;
   ```

   3是show profiles;查询出来的SQL id号。

   > 参数备注：type
   >
   > 1. ALL：显示所有的开销
   > 2. BLOCK IO：显示块IO相关开销
   > 3. CONTEXT SWITCHES：上下文切换相关开销
   > 4. CPU：显示CPU相关开销信息
   > 5. IPC：显示发送和接受相关开销信息
   > 6. MEMORY：显示内存相关开销信息
   > 7. PAGE FAULTS：显示页面错误相关开销信息
   > 8. SOURCE：显示和Source_function、Source_file、Source_line相关的开销信息
   > 9. SWAPS：显示交换次数相关开销的信息



### 日常开发需要注意的结论

1. converting HEAP to MyISAM

   查询结果太大，内存都不够了往磁盘上搬

2. creating tmp table

   创建临时表，拷贝数据到临时表，再删除临时表

3. copying to tmp table on disk

   把内存中临时表复制到硬盘，危险

4. locked



## 全局查询日志

**只可以在测试环境用。**



### 配置开启

在mysql的my.cnf中设置

```cnf
#开启
general_log=1
#记录日志文件的路径
general_log_file=/path/logfile
#输出格式
log_output=FILE
```



### 编码开启

```mysql
set global general_log=1;
set global log_output='TABLE';
```

此后，我们所编写的SQL语句，将会记录在mysql库的general_log表，可以用下面命令查看

```mysql
select * from mysql.general_log;
```


