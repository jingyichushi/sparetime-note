# MySQL深入（三）

> date:2018.11.14

## MySQL锁机制


锁是计算机协调多个进程或线程并发访问某一条资源的机制。

在数据库中，除传统的计算资源（CPU、RAM、I/O）的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所在有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。


**锁的分类：**

+ 从对数据操作的类型（读/写）分：读锁和写锁
  （1）读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响。
  （2）写锁（排它锁）：当前写操作没完成前，他会阻断其他写锁和读锁。

+ 从数据操作的粒度分：表锁和行锁



### 表锁

#### 表锁特点

表锁（偏读）：锁整张表。
特点：偏向MyISAM存储引擎，开销小，加锁快，无死锁；锁定力度大，发生锁冲突的概率最高，并发度最低。

#### 表锁案例分析

1.建表：

```sql
CREATE TABLE mylock(
    id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20)
) ENGINE myisam;

INSERT INTO mylock(name) VALUES('a');
INSERT INTO mylock(name) VALUES('b');
INSERT INTO mylock(name) VALUES('c');
INSERT INTO mylock(name) VALUES('d');
INSERT INTO mylock(name) VALUES('e');

```

2.手动增加表锁
lock table 表名字 read(write)， 表名字2 read(write) ,其它;

给mylock表上读锁，book表上写锁：
```sql
LOCK TABLE mylock read, book WRITE;
```


3.查看哪些表被加锁了
```sql
show open tables;
```


4.释放锁
```sql
unlock tables;
```

**5.给mylock上读锁**

```sql
LOCK TABLE mylock read;
```

上锁后在当前session查看mylock，看是否可读(可读,其他session也可读)
```sql
SELECT * FROM mylock;
```

上锁后在当前session尝试更新mylock，看是否报错（报错）
```sql
UPDATE mylock SET `name` = 'a2' WHERE id = 1;
```

尝试在当前session（加读锁的）读取其他未锁的表（报错）
```sql
SELECT * FROM book;
```

上锁后在其他session尝试更新mylock，看是否报错（**阻塞**，读锁解锁后立即更新）
```sql
UPDATE mylock SET `name` = 'a2' WHERE id = 1;
unlock tables;
```

**6.给mylock上写锁**
```sql
LOCK TABLE mylock write;
```

上锁后在当前session查看mylock，看是否可读(可读)
```sql
SELECT * FROM mylock;
```

上锁后在当前session尝试更新mylock，看是否报错（可以更新）
```sql
UPDATE mylock SET `name` = 'a2' WHERE id = 1;
```

尝试在当前session（加读锁的）读取其他未锁的表（报错）
```sql
SELECT * FROM book;
```

上锁后在其他session查看mylock，看是否可读(阻塞，读锁解锁后立即可读)
```sql
SELECT * FROM mylock;
```

尝试在其他session（加读锁的）读取其他未锁的表（可读）
```sql
SELECT * FROM book;
```

上锁后在其他session尝试更新mylock，看是否报错（阻塞，读锁解锁后立即更新）
```sql
UPDATE mylock SET `name` = 'a2' WHERE id = 1;
```



7.如何分析表锁定
可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定。
```sql
show status like 'table%';
```

table_locks_immediate: 产生表级锁定的**次数**
table_locks_waited：出现表级锁定争用而发生等待的**次数**，此值高则说明存在较严重的表级锁争用情况。

小结：
1.对MyISAM表的读操作（加读锁），不会阻塞其他进程对同一表的读请求，但会阻塞对同一表的写请求。只有当读锁释放后，才会执行其它进程的写操作。
2.对MyISAM表的写操作（加写锁），会阻塞其他进程对同一表的读和写操作，只有当写锁释放后，才会执行其它进程的读写操作。

简而言之，就是**读锁会阻塞写，但是不会阻塞读；而写锁会把读和写都阻塞。**

### 行锁

#### 行锁特点

偏向InnoDB存储引擎，开销大，加锁慢；会出现死锁，锁定粒度最小，发生锁冲突的概率最小，并发度最高.

InnoDB与MyISAM的最大不同有两点：
**一、支持事务；**
**二、采用了行级锁**

#### 事务

我们来复习一下事务：

事务是由一组SQL语句组成的逻辑处理单元，事务具有以下4个属性，通常简称为事务的ACID属性。

+ **原子性（Atomicity）**：事务是一个原子操作单元，其对数据的修改，要么全都执行，要么全都不执行。
+ **一致性（Consistent）**：在事务开始和完成时，数据都必须保持一致状态。这意味着所有相关的数据规则都必须应用于事务的修改，以保持数据的完整性；事务结束时，所有的内部数据结构（如B树索引或双向链表）也都必须是正确的。
+ **隔离性（Isolation）**：数据库系统提供一定的隔离机制，保证事务中不受外部并发操作影响到“独立”环境执行。这意味着事务处理过程中的中间状态对外部是不可见的，反之亦然。
+ **持久性（Durable）**：事务完成之后，它对于数据到修改是永久性的，即使出现系统故障也能够保持。

**并发事务处理带来的问题：**
更新丢失（Lost Update）：事务T1读取了数据，并执行了一些操作，然后更新数据。事务T2也做相同的事，则T1和T2更新数据时可能会覆盖对方的更新，从而引起错误。
脏读（Dirty Reads）：事务A读取到事务B【已修改但未提交的数据】。

不可重复读（Non—Repeatable Reads）：事务A读取到事务B【已提交的**修改**数据】。

幻读（Phantom Reads）：事务A读取到事务B【已提交的**新增**数据】


**事务隔离级别：**

![MySQL数据库深入（三）](http://img.bcoder.top/2018.02.22/1.png)

各具体数据库并不一定完全实现了上述４个隔离级别，例如，Oracle只提供Read committed和Serializable两个标准级别，另外还自己定义的Read only隔离级别：SQL Server除支持上述ISO/ANSI SQL92定义的４个级别外，还支持一个叫做＂快照＂的隔离级别，但严格来说它是一个用MVCC实现的Serializable隔离级别。ＭySQL支持全部４个隔离级别，但在具体实现时，有一些特点，比如在一些隔离级下是采用MVCC一致性读，但某些情况又不是。





查看当前数据库的事务隔离级别：
`show variables like 'tx_isolation';`

### 行锁案例分析

1.建表：
```sql
CREATE TABLE`test_innodb_lock` (
  `a` int(11) DEFAULT NULL,
  `b` varchar(16) DEFAULT NULL,
  KEY `test_innodb_lock_a_IDX` (`a`)
) ENGINE=InnoDB

insert into test_innodb_lock values(1,'b1');
insert into test_innodb_lock values(2,'b2');
insert into test_innodb_lock values(3,'3000');
insert into test_innodb_lock values(4,'4000');
insert into test_innodb_lock values(5,'5000');
insert into test_innodb_lock values(6,'6000');
insert into test_innodb_lock values(7,'7000');
insert into test_innodb_lock values(8,'8000');
insert into test_innodb_lock values(9,'9000');
create index test_innodb_a_ind on test_innodb_lock(a);
create index test_innodb_lock_b_ind on test_innodb_lock(b);
```


**实验一：**
打开两个MySQL客户端，
在客户端1执行:`set autocommit = 0;`
修改客户端1的事务提交方式为手动提交；

在客户端2执行：`set autocommit = 0;`
同样修改客户端2的事务提交方式为手动提交；

在客户端1执行：`update test_innodb_lock set b ='xxx' where  a = 1 and b = 'y';`
同时使用索引字段a和非索引字段b更新一条数据；

在客户端2执行：`mysql> update test_innodb_lock set b ='xxx' where a=1 and b = 'x';`

同时使用索引字段a（并且索引值同客户端1的值相同）和非索引字段 更新另外一条数据；

结果发现客户端2的update语句被阻塞，需要客户端1提交或回滚才能继续执行。说明， **虽然两个事务最终更新的数据不是同一条数据，但然后可能被锁定，这是因为两条SQL语句都使用了相同的索引值（a=1），行级锁上升为页级锁。**

**实验二：**
在客户端1执行：`rollback;`
回滚实验一的操作；

在客户端2执行：`rollback;`
回滚实验一的操作；

在客户端1执行：`update test_innodb_lock set b ='xxx' where  a = 1 and b = 'a';`

同时使用索引字段a和非索引字段b更新一条数据；

在客户端2执行：`update test_innodb_lock set b ='xxx' where a=2 and b = 'b';`

同时使用索引字段a（索引值不同于客户端1SQL语句的索引值）和非索引字段b更新一条数据；
更新顺利进行，执行并没有被阻塞；

说明，同是根据索引和非索引字段进行更新数据，当两个事务的SQL语句中的索引条件值不一样时，更新仍然能够顺利进行。

**实验三：**
在客户端1执行：`rollback;`
回滚实验二的操作；

在客户端2执行：`rollback;`
回滚实验二的操作；

 

在客户端1执行：`update test_innodb_lock set b ='xxx' where b = 'd';`
通过非索引字段更新唯一的一条数据记录

 

在客户端2执行：`update test_innodb_lock set b='xxx' where b ='e';`

通过非索引字段更新另外一条唯一的一条数据记录，update语句被阻塞；

说明， **一个事务根据非索引字段更新数据时，InnoDB会将整个表给锁住，行级锁此时上升为表级锁。**

**实验四：**
在客户端1执行：`rollback;`
回滚实验三的操作；

在客户端2执行：`rollback;`
回滚实验三的操作；


在客户端1执行：`update test_innodb_lock set b ='xxx' where a=4;`
只使用索引更新数据记录

 

在客户端2执行：`update test_innodb_lock set b ='xxx' where a=4;`
只使用索引更新数据记录，同时索引值与客户端1的索引值相同（a=4），此时，客户端2的update语句被阻塞。
说明，**这个现象的行级锁，于我们理解的行级锁一致，即真正只是锁定了一条记录。**

**实验五：**
在客户端1执行：`rollback;`
回滚实验四的操作；

在客户端2执行：`rollback;`
回滚实验四的操作；

在客户端1执行：`update test_innodb_lock set b ='xxx' where a=4;`
只使用索引更新数据记录


在客户端2执行：`update test_innodb_lock set b ='xxx' where b=’g’;`

只使用非索引字段更新数据记录，客户端2的update语句被阻塞，这是因为**客户端2的update语句由于没有使用索引，需要在数据表上加意向排他锁，但在a=4这条记录上，已经存在排他锁了，索引客户端2的update语句只能被阻塞。**


以上实验说明：
1、InnoDB的行级锁在有些情况下是会自动上升为页级锁和表级锁的，此时数据库的写性能会急剧下降，并可能出现大量的死锁（关于死锁的情况，很容易模仿出来，这里不在举例）；
2、真正的行级锁，只发生在所有的事务都是通过索引来进行检索数据的。

我们上面提到了意向排他锁，考虑到行级锁定均由各个存储引擎自行实现，而且具体实现也各有差别，而Innodb是目前事务型存储引擎中使用最为广泛的存储引擎，所以这里我们就主要分析一下Innodb的锁定特性。

总的来说，Innodb的锁定机制和Oracle数据库有不少相似之处。Innodb的行级锁定同样分为两种类型，共享锁和排他锁，而在锁定机制的实现过程中为了让行级锁定和表级锁定共存，Innodb也同样使用了意向锁（**表级锁定**）的概念，也就有了意向共享锁和意向排他锁这两种。

当一个事务需要给自己需要的某个资源加锁的时候，如果遇到一个共享锁正锁定着自己需要的资源的时候，自己可以再加一个共享锁，不过不能加排他锁。但是，如果遇到自己需要锁定的资源已经被一个排他锁占有之后，则只能等待该锁定释放资源之后自己才能获取锁定资源并添加自己的锁定。而意向锁的作用就是当一个事务在需要获取资源锁定的时候，如果遇到自己需要的资源已经被排他锁占用的时候，该事务可以需要锁定行的表上面添加一个合适的意向锁。如果自己需要一个共享锁，那么就在表上面添加一个意向共享锁。而如果自己需要的是某行（或者某些行）上面添加一个排他锁的话，则先在表上面添加一个意向排他锁。意向共享锁可以同时并存多个，但是意向排他锁同时只能有一个存在。所以，可以说Innodb的锁定模式实际上可以分为四种：共享锁（S），排他锁（X），意向共享锁（IS）和意向排他锁（IX），我们可以通过以下表格来总结上面这四种所的共存逻辑关系：

![MySQL数据库深入（三）](http://img.bcoder.top/2018.02.22/2.png)

虽然Innodb的锁定机制和Oracle有不少相近的地方，但是两者的实现确是截然不同的。总的来说就是Oracle锁定数据是通过需要锁定的某行记录所在的物理block上的事务槽上表级锁定信息，而Innodb的锁定则是通过在指向数据记录的第一个索引键之前和最后一个索引键之后的空域空间上标记锁定信息而实现的。Innodb的这种锁定实现方式被称为**“NEXT-KEYlocking”（间隙锁）**，因为Query执行过程中通过过范围查找的华，**它会锁定整个范围内所有的索引键值，即使这个键值并不存在**。


**间隙锁危害**：因为Query执行过程中通过过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害。
 默认情况下，InnoDB工作在可重复读隔离级别下，并且以Next-Key Lock的方式对数据行进行加锁，这样可以有效防止幻读的发生。Next-Key Lock是行锁与间隙锁的组合，这样，当InnoDB扫描索引记录的时候，会首先对选中的索引记录加上行锁（Record Lock），再对索引记录两边的间隙（向左扫描扫到第一个比给定参数小的值， 向右扫描扫描到第一个比给定参数大的值， 然后以此为界，构建一个区间）加上间隙锁（Gap Lock）。如果一个间隙被事务T1加了锁，其它事务是不能在这个间隙插入记录的。

下面我们继续实验测试间隙锁相关的情况：

**实验六：**
在客户端1执行：`rollback;`
回滚实验五的操作；

在客户端2执行：`rollback;`
回滚实验五的操作；

在客户端1执行：`update test_innodb_lock set b ='xxx' where a=8;`
通过索引更新数据记录，索引值为8；

 

在客户端2执行：`insert into test_innodb_lock(a,b) values(8,'xxx');`
向数据表中插入一条数据，插入数据的索引列的值与客户端1的SQL语句的索引值相同，都为8，此时，insert语句被阻塞。

 

在客户端1执行：
```sql
rollback;`
update test_innodb_lock set b ='xxx' where a=8;
```
通过索引更新数据记录，索引值为8；

 

在客户端2执行：
```sql
rollback;
insert into test_innodb_lock(a,b) values(5,'xxx');
```
向数据表中插入一条数据，插入数据的索引列的值小于客户端1的SQL语句的索引值，但大于或等于已有数据记录中最大小于检索索引（a=8）的索引值5，此时，insert语句被阻塞。


在客户端1执行:
```sql
rollback;
update test_innodb_lock set b ='xxx' where a=8;
```
通过索引更新数据记录，索引值为8；


在客户端2执行：
```sql
rollback;
insert into test_innodb_lock(a,b)values(9,'xxx');
```
向数据表中插入一条数据，插入数据的索引列的值大于客户端1的SQL语句的索引值，但 小于已有数据记录中最小大于检索索引（a=8）的索引值10，此时，insert语句被阻塞。

 

在客户端1执行：
```sql
rollback;
update test_innodb_lock set b ='xxx' where a=8;
```
通过索引更新数据记录，索引值为8；

 

在客户端2执行：
```sql
rollback;
insert into test_innodb_lock(a,b)values(10,'xxx');
```
向数据表中插入一条数据，插入数据的索引列的值大于客户端1的SQL语句的索引值，且 大于或等于已有数据记录中最小大于检索索引（a=8）的索引值10，此时，insert语句顺利执行。

 

以上系列的动作说明，当一个事务在通过索引更新数据时，它会将该索引的前后紧紧相邻的索引记录锁住，包括那些根本就不存在的索引值，锁定的区间为左闭右开区间，即[x,y），其中x为小于事务中SQL语句索引值的最大值，y为大于事务中SQL语句索引值的最小值，在本例中，事务中SQL语句索引值为8，索引其锁定的区间为[5,10），所以另外一个事务在做insert操作时，索引值大于或等于5且小于10的索引记录都将被阻塞。需要注意的是，当更新事务的索引值为已有记录中最大值时，这时所有大于该索引值的记录，其他事务的insert操作都将被阻塞。这就是InnoDB间隙锁的具体表现。所以说，InnoDB的间隙锁避免了部分幻读，但不是全部，因为它锁定的是一个区间，而不是整张表。

**实验七：**
在客户端1执行：`rollback;`
回滚实验六的操作

在客户端2执行：`rollback;`
回滚实验六的操作

 

在客户端1执行：`update test_innodb_lock set b ='xxx' where b=’a’;`
通过非索引字段更新一条记录；


在客户端2执行：`insert into test_innodb_lock(a,b)values(10,'xxx');`
插入一条完全不相关的数据，该insert语句被阻塞；

说明，当事务1通过 **非索引**字段更新一条数据是，整张表就会被锁住，即使是insert操作，也将被阻塞。



以上实验说明：

1、InnoDB的间隙锁是可以避免数据出现幻读，但只是避免部分出现幻读，当一个事务是通过索引来更新数据时，另外一个事务在前一个事务索引值前后的左闭右开区间是不能并行插入数据的，必须等待上一个事务提交或回滚；

2、当前一个事务不是通过索引字段来进行更新操作时，那么InnoDB的这种间隙锁就能够完全避免幻读的出现，因为它会将整个表锁住，在当前事务提交或回滚之前，阻塞所以insert操作。


**如何分析行锁定  ?**   注意：Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势了，但是，Innodb的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让Innodb的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况:
`show status like 'innodb_row_lock%';`

![MySQL数据库深入（三）](http://img.bcoder.top/2018.02.22/3.png)

参数分析:
Innodb_row_lock_current_waits：当前正在等待锁定的数量；     
Innodb_row_lock_time：从系统启动到现在锁定总时间长度； 
Innodb_row_lock_time_avg：每次等待所花平均时间；
Innodb_row_lock_time_max：从系统启动到现在等待最常的一次所花的时间；
Innodb_row_lock_waits：系统启动后到现在总共等待的次数；

说明：当等待次数很高，而且每次等待时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手指定优化计划。
​    



行锁（Innodb）优化建议:
① 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁；
② 合理设计索引，尽量缩小锁的范围；
③ 尽可能较少检索条件，避免间隙锁；
④ 尽量控制事务大小，减少锁定资源量和时间长度；
⑤ 尽可能低级别事务隔离

### 页锁

行锁锁指定行，表锁锁整张表，页锁是折中实现，即一次锁定相邻的一组记录。

特点：偏向BerkeleyDB存储引擎，开销和加锁速度介于表锁和行锁之间；会出现死锁；锁定粒度介于表锁和行锁之间，并发度一般。

在生产环境中使用表锁和行锁较多，所以页锁在此不详细介绍。

### 总结

从上述的特点来看，很难笼统的说哪种锁最好，只能根据具体应用的特点来说哪种锁更加合适。仅仅从锁的角度来说的话：

表锁更适用于以查询为主，只有少量按索引条件更新数据的应用，在MySQL数据库中，使用表级锁定的主要是MyISAM，Memory，CSV等一些非事务性存储引擎。

行锁更适用于有大量按索引条件并发更新少量不同数据，同时又有并发查询的应用，在MySQL数据库中，使用行级锁定的主要是Innodb存储引擎和NDBCluster存储引擎，

页级介于表锁和行锁之间，在MySQL数据库中，使用页级锁定主要是BerkeleyDB存储引擎的锁定方式。



## MySQL主从复制

### MySQL主从复制原理

MySQL复制的基本原理：slave会从master读取binlog(二进制日志文件)来进行数据同步.

![MySQL数据库深入（三）](http://img.bcoder.top/2018.02.22/4.png)

**MySQL主从复制步骤：**

1.master将改变记录到二进制日志（binary log）。这些记录的过程叫做二进制日志事件（binary log events）

2.slave 将 master 的binary log events拷贝到它的中继日志（relay log）

3.slace 重做中继日志中的事件，将改变应用到自己的数据库中。MySQL复制是异步的，并且是串行化的。

**MySQL主从复制的基本原则：**
1.每个slave 只有一个master
2.每个slave 只能有一个唯一的服务器ID
3.每个master 可以有多个salve

**MySQL主从复制的最大问题：**时延


### MySQL主从复制搭建

**主从复制的配置条件：**
1.MySQL版本一致且后台以服务运行
2.主从都配置在[mysqld]节点下，都是小写
3.两台机器可以ping的通

**主数据库Master配置：**
（1）server-id=1 (主服务器唯一id，可以自己设置) **必须配置**

（2）log-bin=mysql-bin 或者自定的本地路径/mysql-bin (必须启用二进制日志) **必须配置**

（3）log-err=/MySQL安装目录/data/mysqlerr (启用错误日志) **可选配置**

（4）basedir=/MySQL安装目录 (配置根目录) **可选配置**

（5）tmpdir=/MySQL安装目录 (临时目录) **可选配置**

（6）datadir=/MySQL安装目录/data (数据目录) **可选配置**

（7）read-only=0 (读写都可以) **可选配置**

（8）binlog-ignore-db=mysql (设置不要复制的数据库) **可选配置**

（9）binlog-do-db=需要复制的主数据库的名字 (设置需要复制的数据库) **可选配置**

**从数据库salve配置：**
（1）server-id=2 (从服务器唯一id，可以自己设置) 必须配置
（2）log-bin=mysql-bin (启用二进制日志) 可选配置


后序操作：
1.配置完成以后，重启MySQL服务器，并关闭防火墙
2.在Master上建立账户并授权slave:
`grant replication salve on *.* to 'user1'@'192.168.10.11' identified by '123456'`
建立user1的用户，密码为123456，并授权复制的功能给192.168.10.11的从机服务器。

3.执行命令来刷新:`flush privileges;`

4.查看master状态:`show master status;`可以看到file 和 position 的值（从position位置开始读取file文件进行复制）

5.在从机上配置需要复制的主机:
`change master to master_host='192.168.10.1' , master_user='user1' , master_password='123456' ,master_log_file='上一步的file' , master_log_pos = 上一步的position;`

6.启动从服务器的复制功能:
`start slave;`

7.验证：`show slaver sataus；`
如果 slaver_io_running=yes并且slaver_sql_running=yes则说明配置成功，如果有一个不是yes，则配置失败.

如果需要命令停止从服务器的复制功能：`stop slaver`;