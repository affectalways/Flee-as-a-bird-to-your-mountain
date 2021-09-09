#### 前言：InnoDB 架构

![](https://dev.mysql.com/doc/refman/5.7/en/images/innodb-architecture.png)

> ​	从图中可以看出，change buffer 是 buffer pool 中的一部分，虽然 Change Buffer 的名字叫做 buffer。但是他是可以进行持久化的，在右边的磁盘中的 System Tablespace （系统表空间 -- idbata1）可以看到持久化 Change Buffer 的空间以及把这个 Change Buffer 页 的改动记录在 redo log 里。





**什么是 Change Buffer**

> Change Buffer 是一种特殊的数据结构，是buffer pool中独立的一个区域，当需要修改的**二级索引页**不在**buffer pool**中时，InnoDB引擎会将**对数据的操作**缓存在change buffer中，这样就省去了从磁盘随机IO读这个数据页。

​	在MySQL 5.5 之前的版本中，由于只支持缓存insert操作，所以最初叫做 Insert Buffer，在后来的版本中支持了更多的操作类型缓冲，改叫 Change Buffer（写缓存）

> 



#### change buffer架构

![](https://dev.mysql.com/doc/refman/5.7/en/images/innodb-change-buffer.png)

​		图中详细的描述了Change Buffer的功能，Change Buffer中的数据最终还是会刷到数据所在的原始数据页中。Change Buffer数据应用到原始数据页，得到新的数据页的过程称之为**merge**。merge过程中只会将Change Buffer中与原始数据页有关的数据应用到原始数据页。

**触发merge操作有一下几种情况：**

1. 原始数据页加载到buffer pool时（访问数据页会触发merge）
2. 后台线程定时merge操作
3. MySQL数据库正常关闭时

> 题外话：
>
> 与[聚簇索引](https://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_clustered_index)不同，二级索引通常是非唯一的，并且以相对随机的顺序插入二级索引。同样，删除和更新可能会影响索引树中不相邻的二级索引页。将数据页从磁盘读入内存中涉及到随机IO访问，这里也是数据库中成本最高的操作之一，而利用 Change Buffer 可以减少 随机IO 访问操作，从而提升数据库性能。	
>
> mysql更新语句流程：当需要更新一条数据的时候，并不是将需要更新的那一条记录从磁盘中读出来，而是以页为单位，将整个页读入内存（在InnoDB中，最小存储单元是页，每个数据页的大小是16KB），也就是说，如果引擎找到了这一行数据，数据所在的数据页就都在内存里。



#### Change Buffer的相关设置：

Change Buffer 是可以通过命令参数进行控制的，MySQL 数据库提供了两个 change buffer 的参数。

![](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20210412_075de674-9b7b-11eb-b379-00163e068ecd.png)

1. **innodb_change_buffer_max_size**

   innodb_change_buffer_max_size 表示 Change Buffer 最大大小占 Buffer Pool 的百分比，默认为 25%。最大可以设置为 50%。

2. **innodb_change_buffering**

   innodb_change_buffering 参数用来控制对哪些操作启用 Change Buffer 功能，默认是：all。innodb_change_buffering 参数有以下几种选择：

   ```
   all        默认值。开启buffer
   none       不开启change buffer
   inserts：  只是开启buffer insert操作
   deletes:   只是开delete-marking操作
   changes：  开启buffer insert操作和delete-marking操作
   purges：   对只是在后台执行的物理删除操作开启buffer功能
   ```





#### **什么条件下可以使用 change buffer 呢？**

​		了解了change buffer，那 change buffer什么时候使用呢？是所有的情况下都可以使用吗？

![](D:\Users\sangfor\Desktop\MySQL笔记\InnoDB的索引组织架构.png)

接下来，我们就从这两种索引对查询语句和更新语句的性能影响来进行分析。

##### 查询过程

假设，执行查询的语句是 select id from T where k=5。这个查询语句在索引树上查找的过程，先是通过 B+ 树从树根开始，按层搜索到叶子节点，也就是图中右下角的这个数据页，然后可以认为数据页内部通过二分法来定位记录。

- 对于普通索引来说，查找到满足条件的第一个记录 (5,500) 后，需要查找下一个记录，直到碰到第一个不满足 k=5 条件的记录。
- 对于唯一索引来说，由于索引定义了唯一性，查找到第一个满足条件的记录后，就会停止继续检索。

那么，这个不同带来的性能差距会有多少呢？答案是，微乎其微。

> InnoDB 的数据是按数据页为单位来读写的。也就是说，当需要读一条记录的时候，并不是将这个记录本身从磁盘读出来，而是以页为单位，将其整体读入内存。在 InnoDB 中，每个数据页的大小默认是 16KB。
>
> 因为引擎是按页读写的，所以说，当找到 k=5 的记录的时候，它所在的数据页就都在内存里了。那么，对于普通索引来说，要多做的那一次“查找和判断下一条记录”的操作，就只需要一次指针寻找和一次计算。
>
> 当然，如果 k=5 这个记录刚好是这个数据页的最后一个记录，那么要取下一个记录，必须读取下一个数据页，这个操作会稍微复杂一些。
>
> 但是，对于整型字段，一个数据页可以放近千个 key，因此出现这种情况的概率会很低。所以，我们计算平均性能差异时，仍可以认为这个操作成本对于现在的 CPU 来说可以忽略不计。



##### 更新过程

对于唯一索引来说，所有的更新操作都要先判断这个操作是否违反唯一性约束。比如，要插入 (4,400) 这个记录，就要先判断现在表中是否已经存在 k=4 的记录，而这必须要将数据页读入内存才能判断。如果都已经读入到内存了，那直接更新内存会更快，就没必要使用 change buffer 了。

因此，唯一索引的更新就不能使用 change buffer，实际上也只有普通索引可以使用。（**前面已经提到只有非聚簇索引会用到change buffer**）

**如果要在这张表中插入一个新记录 (4,400) 的话，InnoDB 的处理流程是怎样的。**

第一种情况是，**这个记录要更新的目标页在内存中**。这时，InnoDB 的处理流程如下：

- 对于唯一索引来说，找到 3 和 5 之间的位置，判断到没有冲突，插入这个值，语句执行结束；
- 对于普通索引来说，找到 3 和 5 之间的位置，插入这个值，语句执行结束。

这样看来，普通索引和唯一索引对更新语句性能影响的差别，只是一个判断，只会耗费微小的 CPU 时间。

但，这不是我们关注的重点。

第二种情况是，**这个记录要更新的目标页不在内存中**。这时，InnoDB 的处理流程如下：

- 对于唯一索引来说，需要将数据页读入内存，判断到没有冲突，插入这个值，语句执行结束；
- 对于普通索引来说，则是将更新记录在 change buffer，语句执行就结束了。





#### Change Buffer案例讲解

是不是有点迷糊？讲解个案例

假设，我们有一个主键列为 ID 的表，表中有字段 k，并且在 k 上有索引。

这个表的建表语句是：

```sql
mysql> create table T(
id int primary key, 
k int not null, 
name varchar(16),
index (k))engine=InnoDB;

```

首先我们向数据库中插入两条数据：

```sql
mysql> insert into t(id,k) values(id1,k1),(id2,k2);
```

结合下面这张图来分析这两条插入语句。

![](D:\Users\sangfor\Desktop\MySQL笔记\change buffer更新过程.png)

分析这条更新语句，你会发现它涉及了四个部分：内存、redo log（ib_log_fileX）、 数据表空间（t.ibd，就是一个个的表数据文件，对应的磁盘文件就是“表名.ibd”）、系统表空间（ibdata1，用来放系统信息，如数据字典等，对应的磁盘文件是“ibdata1”）。

这条更新语句做了如下的操作（按照图中的数字顺序）：

1. Page 1 在内存中，直接更新内存；
2. Page 2 没有在内存中，就在内存的 change buffer 区域，记录下“我要往 Page 2 插入一行”这个信息，这个地方及其关键，并没有从磁盘中将 page2 加载到内存。
3. 将上述两个动作记入 redo log 中（图中 3 和 4）。
4. 后台线程会定时将 page1 和 Change Buffer 中的数据持久化

做完上面这些，事务就可以完成了。所以，你会看到，执行这条更新语句的成本很低，就是写了两处内存，然后写了一处磁盘（两次操作合在一起写了一次磁盘），而且还是顺序写的。

ps: 图中的两个虚线箭头，是后台操作，不影响更新的响应时间。

> **主要地方在步骤2**，这就是写缓存（Change Buffer）提高性能的地方，虽然 page2 并没有在内存中，但是并没有妨碍我们往数据库 page2 中插入数据，先将数据写入到写缓存（Change Buffer）中，再后台通过 merge 操作将插入的数据写入到数据页 page2 。 这就是写缓存（Change Buffer）的巧妙之处，也是写缓存（Change Buffer）提高 MySQL 的地方。

比如，我们现在要执行 select * from t where k in (k1, k2)。 以下是读请求的流程图。

如果读语句发生在更新语句后不久，内存中的数据都还在，那么此时的这两个读操作就与系统表空间（ibdata1）和 redo log（ib_log_fileX）无关了。所以，图中就没画出这两部分。

![](D:\Users\sangfor\Desktop\MySQL笔记\change buffer读过程.png)

从图中可以看到：

1. 读 Page 1 的时候，直接从内存返回。
2. 要读 Page 2 的时候，需要把 Page 2 从磁盘读入内存中，然后应用 change buffer 里面的操作日志，生成一个正确的版本并返回结果。**这里会有一次merge操作。**

**从此可以看出，更新的时候，并不急着将数据刷回磁盘。而是先用change buffer缓存起来。当你要执行SELECT的时候，那么再才将磁盘上的数据页缓存到内存中，然后应用这个页面相关的change buffer日志，可以返回可客户端，并且将最新值merge回磁盘。**



#### **触发 Change Buffer 持久化操作有以下几种情况：**

1. 数据库空闲时，后台有线程定时持久化
2. 数据库缓冲池不够用时。
3. 数据库正常关闭
4. redo log写满





#### 小知识点

1. **如果在此时，机器断电重启了，会不会导致 change buffer 丢失呢？change buffer 丢失可不是小事情，再从磁盘读出来的话就没有办法进行merge了，相当于数据丢失了？**

   > 是不会丢失的。虽然只是更新内存，但是在事务提交的时候，change buffer 的操作也会记录到 redo log 里面，所以崩溃回复的时候，change buffer 也是可以找回来的。

2. **merge过程是否会把数据直接写回磁盘？**

   > 先看下merge流程是什么样子的吧：
   >
   >  1、从磁盘读入数据页到内存中（老版本的数据页）
   >
   >  2、从change buffer 里找出这个数据页的 change buffer 记录，可能是多个，以此按照顺序进行应用，得到最新的版本数据页。
   >
   >  3、写redo log。这个redo log 包含了 数据的变更 和 change buffer 的 变更。
   >
   > 在此时 merge 的操作就结束了。这时候数据页 和 内存中的 change buffer 对应的磁盘位置都还没有进行修改， 属于脏页。之后各自在刷回自己的物理数据，就是另一个过程 
   >
   > 解释：
   >
   > ​    更新先是在内存中的数据页更新；完了 Commit 的时候，将更新写入 Redo Log；Redo Log 的写走的是两阶段写协议，保证 Redo Log 和 Binlog 对更新达成共识；Redo Log 的 Commit 阶段执行成功，再响应客户端；但是！！！脏页刷盘的时候，是将内存中数据页的最新值，写入磁盘中的数据文件（表文件），而不是 Redo Log 中；正是由于 Redo Log 的存在，可以让内存中出现大量的脏页。

   