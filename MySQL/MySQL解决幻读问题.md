## Mysql解决幻读问题

> https://www.cnblogs.com/jian0110/p/15080603.html

# 前言

> 　　我们知道MySQL在可重复读隔离级别下别的事物提交的内容，是看不到的。而可提交隔离级别下是可以看到别的事务提交的。而如果我们的业务场景是在事物内同样的两个查询我们需要看到的数据都是一致的，不能被别的事物影响，就使用可重复读隔离级别。这种情况下RR级别下的普通查询（快照读）依靠MVCC解决“幻读”问题，如果是“当前读”的情况需要依靠什么解决“幻读”问题呢？这就是本博文需要探讨的。
>
> 　　在探讨前可以看下之前的博文（[MySQL是如何实现事务隔离？](https://www.cnblogs.com/jian0110/p/14840809.html)），主要介绍隔离级别的具体技术细节，读过以后看此篇文章可能更有帮助。
>
> 　　注：本博文讨论的“幻读”都是指在“可重复读”隔离级别下进行。

　　

------

#  

# 一、什么是幻读？

　　假设我们有表t结构如下，里面的初始数据行为：(0,0,0),(1,1,1),(2,2,2),(3,3,3),(4,4,4),(5,5,5)



```
CREATE TABLE `t`
(
    `id` INT(11) NOT NULL,
    `key`  INT(11) DEFAULT NULL,
    `value`  INT(11) DEFAULT NULL,
    PRIMARY KEY (`id`),
    KEY `value` (`value`)
) ENGINE = InnoDB;
INSERT INTO t
VALUES (0, 0, 0),
       (1, 1, 1),
       (2, 2, 2),
       (3, 3, 3),
       (4, 4, 4),
       (5, 5, 5)
```

　　假设select * from where value=1 for update，只在这一行加锁（注意这只是假设），其它行不加锁，那么就会出现如下场景：

　　[![img](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730152035987-457472370.png)](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730152035987-457472370.png)

Session A的三次查询Q1-Q3都是select * from where value=1 for update，查询的value=1的所有row。

- T1：Q1只返回一行(1,1,1)；
- T2：session B更新id=0的value为1，此时表t中value=1的数据有两行
- T3：Q3返回两行(0,0,1),(1,1,1)
- T4：session C插入一行(6,6,1)，此时表t中value=1的数据有三行
- T5：Q3返回三行(0,0,1),(1,1,1),(6,6,1)
- T6：session A事物commit。

其中Q3读到value=1这一样的现象，就称之为幻读，**幻读指的是一个事务在前后两次查询同一个范围的时候，后一次查询看到了前一次查询没有看到的行**。

先对“幻读”做出如下解释：

- **在可重复读隔离级别下，普通的查询是快照读，是不会看到别的事务插入的数据的。因此， 幻读在“当前读”下才会出现（三个查询都是for update表示当前读）；**
- **上面session B的修改update结果，被session A之后的select语句用“当前读”看到，不能称为幻读，幻读仅专指“新插入的行”**。

# 二、幻读有什么问题？

**（1）需要单独解决**

　　众所周知，select ...for update语句就是将相应的数据行锁住，比如session A在T1时刻的Q1查询语句：select * from where value=1 for update就是将value=1的数据行锁住，但显然如果是上述的场景发生，此时的for update语义被破坏了（并没有锁住value=1的数据行）。

　　**即使把所有的记录都加上锁，还是阻止不了新插入的记录，所以“幻读”问题要单独拿出来解决。没法依靠MVCC或者行锁机制来解决。这就引出“间隙锁”，是另外一种加锁机制**。

**（2）间隙锁引发的并发度**

　　间隙锁引入以后，可能会导致同样语句锁住更大的范围，这可能就会影响了并发度。具体请看下面介绍

# 三、如何解决幻读？

　　**产生幻读的原因是，行锁只能锁住行，但是新插入记录这个动作，要更新的是记录之间的“间隙”。因此，为了解决幻读问题，InnoDB只好引入新的锁，也就是间隙锁(Gap Lock)**。

　　间隙：比如表中加入6个记录，0,5,10,15,20,25。则产生7个间隙：

 [![img](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730170533957-139933393.png)](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730170533957-139933393.png)

　　**在一行行扫描的过程中，不仅将给行加上了行锁，还给行两边的空隙也加上了间隙锁。这样就确保了无法再插入新的记录。**

　　**间隙锁和行锁合称next-key lock，每个next-key lock是前开后闭区间（间隙锁开区间，next-key lock前开后闭区间）**：

　　间隙锁与间隙锁之间是不存在冲突的，冲突的是往间隙里插入一条记录。 

　[![img](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730164120518-792535247.png)](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730164120518-792535247.png)

　　表t中是没有value=7这个数据的，所以Q1加的间隙锁(1,5)，而Q2也是加的这个间隙锁，两者不冲突都是为了保护这个间隙不允许插入值。

　　在表t初始化后，假设表的数据如下：

　　[![img](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730164147130-862136177.png)](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730164147130-862136177.png)

　　如果用select * from for update执行，则会把整个表所有记录锁起来，就形成了7个next-key lock，分别是(-∞,0]、(0,2]、(2,4]、(4,6]、(6,8]、(8, 10]、(10, +supremum]

　　间隙锁的引入，可能会导致同样的语句锁住更大的范围，是会影响了并发度

　　假设发生如下场景：

 [![img](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730164221792-1257424981.png)](https://img2020.cnblogs.com/blog/1352849/202107/1352849-20210730164221792-1257424981.png)

　则明显发生了死锁，分析如下：

- Q1：执行select …for update语句，由于id=9这一行并不存在，因此会加上间隙锁 (8,10);
- Q2：执行select …for update语句，同样会加上间隙锁(8,10)，间隙锁之间不会冲突，因 此这个语句可以执行成功；
- session B 试图插入一行(9,9,9)，被session A的间隙锁挡住了，只好进入等待；
- session A试图插入一行(9,9,9)，被session B的间隙锁挡住了。 

　　有上述可知间隙锁的引入，可能会导致同样语句锁住更大的范围，这其实是影响了并发度。

　　**为了解决幻读问题可以采用读可提交隔离级别，间隙锁是在可重复读隔离级别下才会生效的。所以如果把隔离级别设置为读提交的话， 就没有间隙锁了。但同时，你要解决可能出现的数据和日志不一致问题，需要把binlog格式设置为row，****也就是说采用“RC隔离级别+日志格式binlog_format=row”组合**。

# 三、总结

- **RR隔离级别下间隙锁才有效，RC隔离级别下没有间隙锁；**
- **RR隔离级别下为了解决“幻读”问题：“快照读”依靠MVCC控制，“当前读”通过间隙锁解决；**
- **间隙锁和行锁合称next-key lock，每个next-key lock是前开后闭区间；**
- **间隙锁的引入，可能会导致同样语句锁住更大的范围，影响并发度。**