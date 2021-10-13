# MySQL 中 MyISAM 和 InnoDB 的区别有哪些？

### **区别：**

1.  **InnoDB 支持事务**，MyISAM 不支持事务。这是 MySQL 将默认存储引擎从 MyISAM 变成 InnoDB 的重要原因之一；

2.  **InnoDB 支持外键**，而 MyISAM 不支持。对一个包含外键的 InnoDB 表转为 MYISAM 会失败；  

3.  **InnoDB 是聚集索引，MyISAM 是非聚集索引**。聚簇索引的文件存放在主键索引的叶子节点上，因此 InnoDB 必须要有主键，通过主键索引效率很高。但是辅助索引需要两次查询，先查询到主键，然后再通过主键查询到数据。因此，主键不应该过大，因为主键太大，其他索引也都会很大。而 MyISAM 是非聚集索引，数据文件是分离的，索引保存的是数据文件的指针。主键索引和辅助索引是独立的。 

4.  **InnoDB 不保存表的具体行数**，执行 select count(*) from table 时需要全表扫描。而MyISAM 用一个变量保存了整个表的行数，执行 select count(*) from table 时只需要读出该变量即可，速度很快；    

5.  **InnoDB 最小的锁粒度是行锁，MyISAM 最小的锁粒度是表锁。**MyISAM 只支持表级锁，用户在操作myisam表时，select，update，delete，insert语句都会给表自动加锁。**InnoDB：** 支持事务和行级锁，是innodb的最大特色。行锁大幅度提高了多用户并发操作的新能。但是InnoDB的行锁，只是在WHERE的主键是有效的，非主键的WHERE都会锁全表的。

6. **MyISAM支持FULLTEXT类型的全文索引，InnoDB不支持FULLTEXT类型的全文索引**。但是innodb可以使用sphinx插件支持全文索引，并且效果更好。

7. **MyISAM：**允许没有任何索引和主键的表存在，索引都是保存行的地址。**InnoDB：**如果没有设定主键或者非空唯一索引，就会自动生成一个6字节的主键(用户不可见)。

   

### **如何选择：**

1. 是否要支持事务，如果要请选择 InnoDB，如果不需要可以考虑 MyISAM；
2.  如果表中绝大多数都只是读查询，可以考虑 MyISAM，如果既有读写也挺频繁，请使用InnoDB。
3. 系统奔溃后，MyISAM恢复起来更困难，能否接受，不能接受就选 InnoDB；
4.  MySQL5.5版本开始Innodb已经成为Mysql的默认引擎(之前是MyISAM)，说明其优势是有目共睹的。如果你不知道用什么存储引擎，那就用InnoDB，至少不会差。

