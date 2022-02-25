进入正题前先简单看看MySQL的逻辑架构，相信我用的着。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-00f49587cdc38a28bff4e552ac5e9349.png)MySQL逻辑架构

MySQL的逻辑架构大致可以分为三层：

- 第一层：处理客户端连接、授权认证，安全校验等。
- 第二层：服务器`server`层，负责对SQL解释、分析、优化、执行操作引擎等。
- 第三层：存储引擎，负责MySQL中数据的存储和提取。

> 我们要知道MySQL的服务器层是不管理事务的，事务是由存储引擎实现的，而MySQL中支持事务的存储引擎又属`InnoDB`使用的最为广泛，所以后续文中提到的存储引擎都以`InnoDB`为主。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-d0316602fa70cf53ff29810e01571268.png)MySQL数据更新流程

**记住！** **记住！** **记住！** 上边这张图，她是MySQL更新数据的基础流程，其中包括`redo log`、`bin log`、`undo log`三种日志间的大致关系，好了闲话少说直奔主题。

**Contents**  [hide](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#) 

1 redo log（重做日志）

[1.1 MySQL宕机](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#MySQL_dang_ji)

[1.2 大小固定](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#da_xiao_gu_ding)

[1.3 crash-safe](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#crash-safe)

2 undo log（回滚日志）

[2.1 回滚](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#hui_gun)

[2.2 前滚](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#qian_gun)

3 bin log（归档日志）

[3.1 主从同步](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#zhu_cong_tong_bu)

[3.2 基于时间点还原](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#ji_yu_shi_jian_dian_hai_yuan)

[4 relay log（中继日志）](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#relay_log_zhong_ji_ri_zhi)

[5 slow query log](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#slow_query_log)

[6 general query log](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#general_query_log)

[7 error log](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#error_log)

8 总结

[8.1 EditorT](https://www.teqng.com/2021/08/18/mysql不会丢失数据的秘密，就藏在它的-7种日志里/#EditorT)

### redo log（重做日志）

`redo log`属于MySQL存储引擎`InnoDB`的事务日志。

MySQL的数据是存放在磁盘中的，每次读写数据都需做磁盘IO操作，如果并发场景下性能就会很差。为此MySQL提供了一个优化手段，引入缓存`Buffer Pool`。这个缓存中包含了磁盘中**部分**数据页（`page`）的映射，以此来缓解数据库的磁盘压力。

当从数据库读数据时，首先从缓存中读取，如果缓存中没有，则从磁盘读取后放入缓存；当向数据库写入数据时，先向缓存写入，此时缓存中的数据页数据变更，这个数据页称为**脏页**，`Buffer Pool`中修改完数据后会按照设定的更新策略，定期刷到磁盘中，这个过程称为**刷脏页**。

#### MySQL宕机

如果刷脏页还未完成，可MySQL由于某些原因宕机重启，此时`Buffer Pool`中修改的数据还没有及时的刷到磁盘中，就会导致数据丢失，无法保证事务的持久性。

为了解决这个问题引入了`redo log`，redo Log如其名侧重于重做！它记录的是数据库中每个页的修改，而不是某一行或某几行修改成怎样，可以用来恢复提交后的物理数据页，且只能恢复到最后一次提交的位置。

`redo log`用到了`WAL`（Write-Ahead Logging）技术，这个技术的核心就在于修改记录前，一定要先写日志，并保证日志先落盘，才能算事务提交完成。

有了redo log再修改数据时，InnoDB引擎会把更新记录先写在redo log中，在修改`Buffer Pool`中的数据，当提交事务时，调用`fsync`把redo log刷入磁盘。至于缓存中更新的数据文件何时刷入磁盘，则由后台线程异步处理。

> **注意**：此时redo log的事务状态是`prepare`，还未真正提交成功，要等`bin log`日志写入磁盘完成才会变更为`commit`，事务才算真正提交完成。

这样一来即使刷脏页之前MySQL意外宕机也没关系，只要在重启时解析redo log中的更改记录进行重放，重新刷盘即可。

#### 大小固定

redo log采用固定大小，循环写入的格式，当redo log写满之后，重新从头开始如此循环写，形成一个环状。

那为什么要如此设计呢？

因为redo log记录的是数据页上的修改，如果`Buffer Pool`中数据页已经刷磁盘后，那这些记录就失效了，新日志会将这些失效的记录进行覆盖擦除。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-56435803bef0450e698552f22e4aba45.png)

上图中的`write pos`表示redo log当前记录的日志序列号`LSN`(log sequence number)，写入还未刷盘，循环往后递增；`check point`表示redo log中的修改记录已刷入磁盘后的LSN，循环往后递增，这个LSN之前的数据已经全落盘。

`write pos`到`check point`之间的部分是redo log空余的部分（绿色），用来记录新的日志；`check point`到`write pos`之间是redo log已经记录的数据页修改数据，此时数据页还未刷回磁盘的部分。当`write pos`追上`check point`时，会先推动`check point`向前移动，空出位置（刷盘）再记录新的日志。

> **注意**：redo log日志满了，在擦除之前，需要确保这些要被擦除记录对应在内存中的数据页都已经刷到磁盘中了。擦除旧记录腾出新空间这段期间，是不能再接收新的更新请求的，此刻MySQL的性能会下降。所以在并发量大的情况下，合理调整redo log的文件大小非常重要。

#### crash-safe

因为redo log的存在使得`Innodb`引擎具有了`crash-safe`的能力，即MySQL宕机重启，系统会自动去检查redo log，将修改还未写入磁盘的数据从redo log恢复到MySQL中。

MySQL启动时，不管上次是正常关闭还是异常关闭，总是会进行恢复操作。会先检查数据页中的`LSN`，如果这个 LSN 小于 redo log 中的LSN，即`write pos`位置，说明在`redo log`上记录着数据页上尚未完成的操作，接着就会从最近的一个`check point`出发，开始同步数据。

简单理解，比如：redo log的`LSN`是500，数据页的`LSN`是300，表明重启前有部分数据未完全刷入到磁盘中，那么系统则将redo log中`LSN`序号300到500的记录进行重放刷盘。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-d0da942bc8ca6bcb877ffb028a0ef28a.png)

### undo log（回滚日志）

`undo log`也是属于MySQL存储引擎InnoDB的事务日志。

`undo log`属于逻辑日志，如其名主要起到回滚的作用，它是保证事务原子性的关键。记录的是数据修改前的状态，在数据修改的流程中，同时会记录一条与当前操作相反的逻辑日志到`undo log`中。

我们举个栗子：假如更新ID=1记录的name字段，name原始数据为小富，现改name为程序员内点事

事务执行`update X set name = 程序员内点事 where id =1`语句时，先会在`undo log`中记录一条相反逻辑的`update X set name = 小富 where id =1`记录，这样当某些原因导致服务异常事务失败，就可以借助`undo log`将数据回滚到事务执行前的状态，保证事务的完整性。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-7dccf9ce123e4519d767102643a3a30d.png)

那可能有人会问：同一个事物内的一条记录被多次修改，那是不是每次都要把数据修改前的状态都写入`undo log`呢？

答案是不会的！

`undo log`只负责记录事务开始前要修改数据的原始版本，当我们再次对这行数据进行修改，所产生的修改记录会写入到`redo log`，`undo log`负责完成回滚，`redo log`负责完成前滚。

#### 回滚

未提交的事务，即事务未执行`commit`。但该事务内修改的脏页中，可能有一部分脏块已经刷盘。如果此时数据库实例宕机重启，就需要用回滚来将先前那部分已经刷盘的脏块从磁盘上撤销。

#### 前滚

未完全提交的事务，即事务已经执行`commit`，但该事务内修改的脏页中只有一部分数据被刷盘，另外一部分还在`buffer pool`缓存上，如果此时数据库实例宕机重启，就需要用前滚来完成未完全提交的事务。将先前那部分由于宕机在内存上的未来得及刷盘数据，从`redo log`中恢复出来并刷入磁盘。

> 数据库实例恢复时，先做前滚，后做回滚。

如果你仔细看过了上边的 `MySQL数据更新流程图` 就会发现，`undo log`、`redo log`、`bin log`三种日志都是在刷脏页之前就已经刷到磁盘了的，相互协作最大限度保证了用户提交的数据不丢失。

### bin log（归档日志）

`bin log`是一种数据库Server层（和什么引擎无关），以二进制形式存储在磁盘中的逻辑日志。`bin log`记录了数据库所有`DDL`和`DML`操作（不包含 `SELECT` 和 `SHOW`等命令，因为这类操作对数据本身并没有修改）。

默认情况下，二进制日志功能是关闭的。可以通过以下命令查看二进制日志是否开启：

```
mysql> SHOW VARIABLES LIKE 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+
```

`bin log`也被叫做`归档日志`，因为它不会像`redo log`那样循环写擦除之前的记录，而是会一直记录日志。一个`bin log`日志文件默认最大容量`1G`（也可以通过`max_binlog_size`参数修改），单个日志超过最大值，则会新创建一个文件继续写。

```
mysql> show binary logs;
+-----------------+-----------+
| Log_name        | File_size |
+-----------------+-----------+
| mysq-bin.000001 |      8687 |
| mysq-bin.000002 |      1445 |
| mysq-bin.000003 |      3966 |
| mysq-bin.000004 |       177 |
| mysq-bin.000005 |      6405 |
| mysq-bin.000006 |       177 |
| mysq-bin.000007 |       154 |
| mysq-bin.000008 |       154 |
```

`bin log`日志的内容格式其实就是执行SQL命令的反向逻辑，这点和`undo log`有点类似。一般来说开启`bin log`都会给日志文件设置过期时间（`expire_logs_days`参数，默认永久保存），要不然日志的体量会非常庞大。

```
mysql> show variables like 'expire_logs_days';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| expire_logs_days | 0     |
+------------------+-------+
1 row in set

mysql> SET GLOBAL expire_logs_days=30;
Query OK, 0 rows affected
```

`bin log`主要应用于MySQL主从模式（`master-slave`）中，主从节点间的数据同步；以及基于时间点的数据还原。

#### 主从同步

通过下图MySQL的主从复制过程，来了解下`bin log`在主从模式下的应用。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-7a8c2cd20af2a0c9f5d0f38a6bd37ec5.png)

- 用户在主库`master`执行`DDL`和`DML`操作，修改记录顺序写入`bin log`;
- 从库`slave`的I/O线程连接上Master，并请求读取指定位置`position`的日志内容;
- `Master`收到从库`slave`请求后，将指定位置`position`之后的日志内容，和主库bin log文件的名称以及在日志中的位置推送给从库;
- slave的I/O线程接收到数据后，将接收到的日志内容依次写入到`relay log`文件最末端，并将读取到的主库bin log文件名和位置`position`记录到`master-info`文件中，以便在下一次读取用;
- slave的SQL线程检测到`relay log`中内容更新后，读取日志并解析成可执行的SQL语句，这样就实现了主从库的数据一致;

#### 基于时间点还原

我们看到`bin log`也可以做数据的恢复，而`redo log`也可以，那它们有什么区别？

- 层次不同：redo log 是InnoDB存储引擎实现的，bin log 是MySQL的服务器层实现的，但MySQL数据库中的任何存储引擎对于数据库的更改都会产生bin log。
- 作用不同：redo log 用于碰撞恢复（`crash recovery`），保证MySQL宕机也不会影响持久性；bin log 用于时间点恢复（`point-in-time recovery`），保证服务器可以基于时间点恢复数据和主从复制。
- 内容不同：redo log 是物理日志，内容基于磁盘的页`Page`；bin log的内容是二进制，可以根据`binlog_format`参数自行设置。
- 写入方式不同：redo log 采用循环写的方式记录；binlog 通过追加的方式记录，当文件大小大于给定值后，后续的日志会记录到新的文件上。
- 刷盘时机不同：bin log在事务提交时写入；redo log 在事务开始时即开始写入。

bin log 与 redo log 功能并不冲突而是起到相辅相成的作用，需要二者同时记录，才能保证当数据库发生宕机重启时，数据不会丢失。

### relay log（中继日志）

`relay log`日志文件具有与`bin log`日志文件相同的格式，从上边MySQL主从复制的流程可以看出，`relay log`起到一个中转的作用，`slave`先从主库`master`读取二进制日志数据，写入从库本地，后续再异步由`SQL线程`读取解析`relay log`为对应的SQL命令执行。

### slow query log

慢查询日志（`slow query log`）: 用来记录在 MySQL 中执行时间超过指定时间的查询语句，在 SQL 优化过程中会经常使用到。通过慢查询日志，我们可以查找出哪些查询语句的执行效率低，耗时严重。

出于性能方面的考虑，一般只有在排查慢SQL、调试参数时才会开启，默认情况下，慢查询日志功能是关闭的。可以通过以下命令查看是否开启慢查询日志：

```
mysql> SHOW VARIABLES LIKE 'slow_query%';
+---------------------+--------------------------------------------------------+
| Variable_name       | Value                                                  |
+---------------------+--------------------------------------------------------+
| slow_query_log      | OFF                                                    |
| slow_query_log_file | /usr/local/mysql/data/iZ2zebfzaequ90bdlz820sZ-slow.log |
+---------------------+--------------------------------------------------------+
```

通过如下命令开启慢查询日志后，我发现 `iZ2zebfzaequ90bdlz820sZ-slow.log` 日志文件里并没有内容啊，可能因为我执行的 SQL 都比较简单没有超过指定时间。

```
mysql>  SET GLOBAL slow_query_log=ON;
Query OK, 0 rows affected
```

上边提到超过 `指定时间` 的查询语句才算是慢查询，那么这个时间阈值又是多少嘞？我们通过 `long_query_time` 参数来查看一下，发现默认是 10 秒。

```
mysql> SHOW VARIABLES LIKE 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```

这里我们将 `long_query_time` 参数改小为 0.001秒再次执行查询SQL，看看慢查询日志里是否有变化。

```
mysql> SET GLOBAL long_query_time=0.001;
Query OK, 0 rows affected
```

果然再执行 SQL 的时，执行时间大于 0.001秒，发现慢查询日志开始记录了。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-8a1daaefc92371bd52801d352c12a6fe.png)慢查询日志

### general query log

一般查询日志（`general query log`）：用来记录用户的**所有**操作，包括客户端何时连接了服务器、客户端发送的所有`SQL`以及其他事件，比如 `MySQL` 服务启动和关闭等等。`MySQL`服务器会按照它接收到语句的先后顺序写入日志文件。

由于一般查询日志记录的内容过于详细，开启后 Log 文件的体量会非常庞大，所以出于对性能的考虑，默认情况下，该日志功能是关闭的，通常会在排查故障需获得详细日志的时候才会临时开启。

我们可以通过以下命令查看一般查询日志是否开启，命令如下：

```
mysql> show variables like 'general_log';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| general_log   | OFF   |
+---------------+-------+
```

下边开启一般查询日志并查看日志存放的位置。

```
mysql> SET GLOBAL general_log=on;
Query OK, 0 rows affected
mysql> show variables like 'general_log_file';
+------------------+---------------------------------------------------+
| Variable_name    | Value                                             |
+------------------+---------------------------------------------------+
| general_log_file | /usr/local/mysql/data/iZ2zebfzaequ90bdlz820sZ.log |
+------------------+---------------------------------------------------+
```

执行一条查询 SQL 看看日志内容的变化。

```
mysql> select * from t_config;
+---------------------+------------+---------------------+---------------------+
| id                  | remark     | create_time         | last_modify_time    |
+---------------------+------------+---------------------+---------------------+
| 1325741604307734530 | 我是广播表 | 2020-11-09 18:06:44 | 2020-11-09 18:06:44 |
+---------------------+------------+---------------------+---------------------+
```

我们看到日志内容详细的记录了所有执行的命令、SQL、SQL的解析过程、数据库设置等等。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-e27ab25949bc2f26ad2c540d719f4007.png)一般查询日志

### error log

错误日志（`error log`）: 应该是 MySQL 中最好理解的一种日志，主要记录 MySQL 服务器每次启动和停止的时间以及诊断和出错信息。

默认情况下，该日志功能是开启的，通过如下命令查找错误日志文件的存放路径。

```
mysql> SHOW VARIABLES LIKE 'log_error';
+---------------+----------------------------------------------------------------+
| Variable_name | Value                                                          |
+---------------+----------------------------------------------------------------+
| log_error     | /usr/local/mysql/data/LAPTOP-UHQ6V8KP.err |
+---------------+----------------------------------------------------------------+
```

**注意**：错误日志中记录的可并非全是错误信息，像 MySQL 如何启动 `InnoDB` 的表空间文件、如何初始化自己的存储引擎，初始化 `buffer pool` 等等，这些也记录在错误日志文件中。

![img](https://www.teqng.com/wp-content/uploads/2021/08/wxsync-2021-08-082da1b97446c50334332bb918472405.png)

### 总结

MySQL作为我们工作中最常接触的中间件，熟练使用只算是入门，如果要在简历写上一笔精通，还需要深入了解其内部工作原理，而这7种日志也只是深入学习过程中的一个起点，学无止境，兄嘚干就完了！