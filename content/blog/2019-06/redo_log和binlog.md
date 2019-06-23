---
date: "2019-06-23T11:42:48+08:00"
draft: false
title: "redo log和binlog"
tags: ["Mysql", "sql"]
series: ["mysql"]
categories: ["Mysql"]
toc: true
---

## 序
前面我们介绍了一条查询sql的执行过程[《一条SQL查询语句是如何执行的》](https://xp329486175.github.io/blog/2019-06/%E4%B8%80%E6%9D%A1sql%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E7%9A%84/)。
那如果是一条更新sql呢？会是一个怎样的执行过程？

其实一条更新语句与查询语句执行过程基本一样。经过连接器、分析器、优化器、执行器等功能模块，最后到达存储引擎。    
但是它还是有差别的。差别就在`redo log`和`binlog`。

## redo log和binlog
前面我们说过：mysql可以看成两块：server层和存储引擎层。    
而作为mysql中最重要的两个日志。`物理日志redo log工作在存储引擎层，逻辑日志binlog工作在server层。`
也叫重做日志和归档日志。

### redo log
简单来说，redo log的作用就是记录数据页"做了哪些活动"。它主要用到的是mysql里面经常说到的WAL技术。
WAL全称是Write-Ahead Logging。它的关键点就是先写日志，再写磁盘。也就是说，当mysql处理请求时，
存储引擎会将"做了哪些事"先记录到redo log，并更新内存，然后等mysql空闲时，再更新到磁盘。

redo log是InnoDB特有的。有了redo log，InnoDB就可以保证数据库即使异常重启，之前的提交也不会丢失，这种能力称为crash-safe。

那redo log到底是怎么工作的呢？会不会写满？

答案是会被写满。而且redo log是固定大小的，比如可以配置一组为4个文件，每个文件大小为1GB，那么redo log总的大小就为4GB。

再贴一个redo log工作原理图：
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/mysql_03.png" width='200px'/>{{% /center %}}
write pos是当前记录的位置，一边写一边后移。写到第3号文件末尾，就要回到0号文件开头。     
check point是当前要清除日志的位置，也是往后移并且循环的。清除记录时会先把记录更新到数据文件。

write log和check point直接空着的部分，可以用来记录新的操作。当write log追上check point时，表示redo log满了，
这时候不能再执行新的更新，需要清除一些记录，把check point推进一下。

### binlog
mysql-binlog是MySQL数据库的二进制日志，用于记录用户对数据库操作的SQL语句（(除了数据查询语句）信息。
可以使用mysqlbin命令查看二进制日志的内容。

binlog的格式有三种：STATEMENT、ROW、MIXED 。

1、`STATMENT模式`：基于SQL语句的复制(statement-based replication, SBR)，每条修改数据的sql语句都会记录到binlog中。   
优点：不需要记录每一条SQL语句与每行的数据变化，这样子binlog的日志也会比较少，减少了磁盘IO，提高性能。          
缺点：在某些情况下会导致master-slave中的数据不一致(如sleep()函数， last_insert_id()，以及user-defined functions(udf)等会出现问题)。

2、`ROW模式`：基于行的复制(row-based replication, RBR)：不记录每一条SQL语句的上下文信息，仅需记录哪条数据被修改了，修改成了什么样子了。       
优点：不会出现某些特定情况下的存储过程、或function、或trigger的调用和触发无法被正确复制的问题。      
缺点：会产生大量的日志，尤其是alter table的时候会让日志暴涨。

3、`MIXED模式`：混合模式复制(mixed-based replication, MBR)：以上两种模式的混合使用，一般的复制使用STATEMENT模式保存binlog，
对于STATEMENT模式无法复制的操作使用ROW模式保存binlog，MySQL会根据执行的SQL语句选择日志保存方式。
### 两种日志的区别
为什么会有redolog和binlog两份日志呢？     
因为最开始mysql并没有InnoDB引擎。之前的mysql默认引擎是MyIASM。但是MyIASM并没有crash-safe的能力，binlog只能用于归档。
所以后来以插件的形式引入了InnDB，而InnoDB使用另一套日志系统。也就是redo log来实现crash-safe能力。

总结来说：redo log和binlog的区别主要有三点：

>`1、redo log是InnoDB引擎特有的；binlog是MySQL的Server层实现的，所有引擎都可以使用。`             
`2、redo log是物理日志，记录的是“在某个数据页上做了什么修改”；
binlog是逻辑日志，记录的是这个语句的原始逻辑，比如“给ID=2这一行的v字段加1”。`                 
`3、redo log是循环写的，空间固定会用完；binlog是可以追加写入的。
“追加写”是指binlog文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。`
                                                             
### update语句内部执行流程                                                             
理解来两个日志的概念，我们再来看执行器InnoDB引擎在执行这个简单的update语句时的内部流程。
```sql
update test set v=v+1 where ID=2;
```
1、执行器先找引擎取id=2这一行。id是主键，引擎直接用树搜索找到这一行。如果这一行本来就在内存中，就直接返回给执行器；
否则，需要先从磁盘读入内存，然后再返回。    
2、执行器拿到引擎给的行数据，把这个值加上1，比如原来是N，现在就是N+1，得到新的一行数据，再调用引擎接口写入这行新数据。     
3、引擎将这行新数据更新到内存中，同时将这个更新操作记录到redo log里面，此时redo log处于prepare状态。然后告知执行器执行完成了，
随时可以提交事务。      
4、执行器生成这个操作的binlog，并把binlog写入磁盘。     
5、执行器调用引擎的提交事务接口，引擎把刚刚写入的redo log改成提交（commit）状态，更新完成。
                                                       
下面是这个update 语句的执行流程图，图中浅色框表示是在 InnoDB 内部执行的，深色框表示是在执行器中执行的。                        
{{% center %}}<img name="touchbar-config" src="/images/blog/2019-06/mysql_02.png" width='400px'/>{{% /center %}}

### 两阶段提交
在上面最update执行流程图中，后三步将redo log的写入拆成了两个步骤：prepare和commit，这就是两阶段提交。
                               
为什么会有两阶段提交呢？这是为了让两个日志之间的逻辑保持一致。这里我们用上面的例子推导一下。

通常我们可以通过binlog恢复最近半个月任意一秒的状态。比如某天下午两点发现中午十二点有一次误删表，需要找回数据，
那我们可以这么做：

>首先，找到最近的一次全量备份，如果你运气好，可能就是昨天晚上的一个备份，从这个备份恢复到临时库；      
然后，从备份的时间点开始，将备份的binlog依次取出来，重放到中午误删表之前的那个时刻。
          
假设当前ID=2的行，字段v的值是0，再假设执行update语句过程中在写完第一个日志后，第二个日志还没有写完期间发生了crash，会出现什么情况呢？     

1、**先写reredo log后写binlog。**假设在redo log写完，binlog还没有写完的时候，MySQL进程异常重启。由于我们前面说过的，redo log写完之后，系统即使崩溃。
仍然能够把数据恢复回来，所以恢复后这一行v的值是 1。
但是由于binlog没写完就crash了，这时候binlog 里面就没有记录这个语句。因此，之后备份日志的时候，存起来的 binlog 里面就没有这条句。 
然后你会发现，如果需要用这个binlog来恢复临时库的话，由于这个语句的binlog丢失，这个临时库就会少了这一次更新，恢复出来的这一行v的值就是0，与原库的值不同。

2、**先写binlog后写redo log。**如果在binlog写完之后crash，由于 redo log 还没写，崩溃恢复以后这个事务无效，所以这一行v的值是0。
但是binlog里面已经记录了"把v从0改成1"这个日志。所以，在之后用binlog来恢复的时候就多了一个事务出来，恢复出来的这一行v的值就是1，与原库的值不同。

可以看到，如果不使用“两阶段提交”，那么数据库的状态就有可能和用它的日志恢复出来的库的状态不一致。



