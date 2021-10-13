# 记录锁、间隙锁和临键锁

### Record Lock

> A record lock is a lock on an index record.
> For example, SELECT c1 FROM t WHERE c1 = 10 FOR UPDATE; 
> prevents any other transaction from inserting, updating, or deleting rows where the value of t.c1 is 10.

记录锁锁住的是索引记录。如果使用索引作为条件命中了记录，那么就是记录锁，被锁住的记录不能被别的事务插入相同的索引键值，修改和删除。

我们在表中插入4条记录，主键分别是1、4、7、10。

我们用主键或者唯一索引作为条件等值查询的时候，命中记录就是加的记录锁，如：
select * from xx where id = 1;
命中记录，所以id = 1这条记录就加了记录锁。

### Gap Lock

> A gap lock is a lock on a gap between index records, or a lock on the gap before the first or after the last index record.

间隙锁是锁在**索引**之间或者第一个索引前面或者最后一个索引后面。

当我们使用索引无论是等值还是范围查询，没有命中一条记录时候，加的就是间隙锁。
还是拿上面的例子，我们在表中插入4条记录，主键分别是1、4、7、10。

![](F:\Flee-as-a-bird-to-your-mountain\MySQL\pictures\间隙锁1.png)

图中的范围区间就会被锁住，都是左开右开的区间。

> select * from xx where id = 6 或者 select * from xx were id >4 and id < 7;

没有命中任何一条记录，会锁住（4，7）区间，另一个事务插入id = 6则会阻塞；

select * from xx where id > 20没有命中，会锁住（10，正无穷），另一个事务插入id = 11会阻塞。

> **间隙锁只在可重复读隔离基本中存在**。

### Next-key Lock

> A next-key lock is a combination of a record lock on the index record and a gap lock on the gap before the index record.

当我们使用索引进行范围查询，命中了记录的情况下，就是使用了临键锁，他相当于记录锁+间隙锁。

两种退化的情况：

1. 唯一性索引，等值查询匹配到一条记录的时候，退化成记录锁。
2. 没有匹配到任何记录的时候，退化成间隙锁。

**左开右闭**区间，**目的是为了解决幻读的问题**。

![](F:\Flee-as-a-bird-to-your-mountain\MySQL\pictures\临键锁.png)



select * from xx where id > 5 and id < 9;

上面的sql命中了id = 7的记录，也包含了记录不存在的区间，所以他锁住（4，7]和（7，10]区间，在这区间，别的事务插入不了数据，所以解决了幻读问题。


