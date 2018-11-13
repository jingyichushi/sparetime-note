# MySQL数据库深入（二）

> data:2018.11.06



## 索引优化案例


### 单表索引优化

#### 建表
建立本次优化案例中所需的数据库及数据表

```sql
CREATE DATABASE db02;
USE db02;

CREATE TABLE `db02`.`article`(  
  `id` INT(11) NOT NULL AUTO_INCREMENT,
  `author_id` INT(11) UNSIGNED NOT NULL,
  `category_id` INT(11) UNSIGNED NOT NULL,
  `views` INT(11) UNSIGNED NOT NULL,
  `comments` INT(11) UNSIGNED NOT NULL,
  `title` VARCHAR(255) NOT NULL,
  `content` TEXT NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=INNODB CHARSET=utf8;


INSERT INTO `db02`.`article` (`id`, `author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES (NULL, '1', '1', '1', '1', '1', '1');
INSERT INTO `db02`.`article` (`id`, `author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES (NULL, '2', '2', '2', '2', '2', '2');
INSERT INTO `db02`.`article` (`id`, `author_id`, `category_id`, `views`, `comments`, `title`, `content`) VALUES (NULL, '1', '1', '3', '3', '3', '3');
```

#### 单表索引分析
下面我们来执行这条sql：查询category_id为1，且comments大于1的情况下，views最多的article_id

```sql
SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/1.png)



通过explain命令来查看sql查询优化信息:
```sql
EXPLAIN SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
```

sql查询优化信息:

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/2.png)

很显然type是ALL，即最坏情况。Extra里还出现Using filesort（文件内排序），也是最坏情况，所以优化是必须的。

#### 优化

建立索引的SQL语句:

```sql
CREATE INDEX idx_article_ccv ON article (category_id,comments,views);
```

再次执行查询分析sql:
```sql
EXPLAIN SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/3.png)


type变成了range，这是可以忍受的。但是extra里使用了Using filesort 仍然是无法接受的。 

但是我们已经建立的索引，为啥没有用呢？ 
这是因为按照BTree索引的工作原理先排序category_id，如果遇到相同的category_id则再排序comments，如果遇到相同的commetns则再排序views.
当comments字段在联合索引中处于中间位置时， 因为comments > 1 条件是一个范围值（所谓的range），**Mysql无法利用索引再对后面的views部分进行检索，即range类型查询字段后面索引无效。**


#### 第二次优化

删除不合适的索引:
```sql
DROP INDEX idx_article_ccv ON article;
```

重新建立索引:
```sql
CREATE INDEX idx_article_cv ON article(category_id,views);
```

重新执行查询分析
```sql
EXPLAIN SELECT id,author_id FROM article WHERE category_id = 1 AND comments > 1 ORDER BY views DESC LIMIT 1;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/4.png)

根据MySQL的查询分析报告可知，使用当前建立的索引，达到了type=ref,且extra中没有出现Using filesort，因此，我们现在使用的索引结构达到了最优的情况。

### 两表索引优化

#### 建表

```sql
create table if not EXISTS `class`(
	`id` int(10) UNSIGNED not null auto_increment,
	`card` int(10) UNSIGNED not NULL,
	PRIMARY KEY(`id`)
);

CREATE TABLE if NOT EXISTS `book`(
	`bookid` int(10) UNSIGNED not null auto_increment,
	`card` int(10) UNSIGNED not null,
	PRIMARY key(`bookid`)
);

INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO class(card) VALUES(FLOOR(1+(RAND()*20)));

INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
INSERT INTO book(card) VALUES(FLOOR(1+(RAND()*20)));
```

#### 两表索引分析
下面我们来执行这条sql：


```sql
SELECT * from class LEFT JOIN book ON class.card = book.card;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/5.png)

通过explain命令来查看sql查询优化信息:
```sql
EXPLAIN SELECT * from class LEFT JOIN book ON class.card = book.card;
```

sql查询优化信息:

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/6.png)

结论：type 有All ，需要优化

#### 优化



建立索引的SQL语句:

```sql
ALTER TABLE `book` ADD INDEX Y(`card`);
```

再次执行查询分析sql:
```sql
explain SELECT * from class LEFT JOIN book ON class.card = book.card;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/7.png)


可以看到第二行的 type 变为了 ref , rows 也变成了 1 优化比较明显。
这是由左连接特性决定的。LEFT JOIN条件用于确定从右表搜索行，左边一定都有，
所以右边是我们的关键点，一定需要建立索引。




#### 再次分析



左连接改成右连接：
```sql
explain SELECT * from class RIGHT JOIN book ON class.card = book.card
```
![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/8.png)


删除索引:

```sql
drop index Y on book;
```
重建索引
```sql
create index X on class(card);
```
再次分析
```sql
explain SELECT * from class RIGHT JOIN book ON class.card = book.card
```
![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/9.png)


优化比较明显。这是因为RIGHT JOIN 条件用于确定如何从左表搜索行，右边一定都有，所以左边是我们的关键点，一定需要建立索引。

综上所述 ,我们得到以下结论:
**左连接：索引建在右表上。
右连接：索引建在左表上。**

### 三表索引优化

#### 建表

```sql
CREATE TABLE IF NOT EXISTS `phone` (  
`phoneid` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,  
`card` INT(10) UNSIGNED NOT NULL,  
PRIMARY KEY (`phoneid`)  
) ENGINE = INNODB;  
  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20)));  
INSERT INTO phone(card) VALUES(FLOOR(1 + (RAND() * 20))); 
```

#### 三表索引分析

接下来给表建立索引
```sql
ALTER TABLE `phone` ADD INDEX Z ( `card`);
ALTER TABLE `book` ADD INDEX Y ( `card`);

```

下面我们来执行这条sql：

```sql
SELECT * FROM class LEFT JOIN book ON class.card=book.card LEFT JOIN phone ON book.card = phone.card;
```
![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/10.png)

通过explain命令来查看sql查询优化信息:
```sql
EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card=book.card LEFT JOIN phone ON book.card = phone.card;
```

sql查询优化信息:

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/11.png)

后 2 行的 type 都是 ref 且总 rows 优化很好,效果不错。因此索引最好设置在需要经常查询的字段中。


结论：

1.尽可能减少Join语句中的NestedLoop的循环总次数：“永远用小结果集驱动大的结果集”(小表驱动大表)。

2.Join语句的优化，优先优化NestedLoop的内层循环；

**3.保证Join语句中被驱动表上Join条件字段已经被索引；**

当无法保证被驱动表的Join条件字段被索引且内存资源充足的前提下，不要太吝惜JoinBuffer的设置；此属性在my.cnf中设置。



## 索引失效

首先我们先来建表、插入数据以及增加索引：
```sql
CREATE TABLE IF NOT EXISTS staffs(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(24) NOT NULL DEFAULT "" COMMENT'姓名',
    age INT NOT NULL DEFAULT 0 COMMENT'年龄',
    pos VARCHAR(20) NOT NULL DEFAULT "" COMMENT'职位',
    add_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT'入职事件'
) CHARSET utf8 COMMENT'员工记录表';

INSERT INTO `staffs` (`name`, `age`, `pos`, `add_time`) VALUES ('z3', 22, 'manager', now());
INSERT INTO `staffs` (`name`, `age`, `pos`, `add_time`) VALUES ('July', 23, 'dev', now());
INSERT INTO `staffs` (`name`, `age`, `pos`, `add_time`) VALUES ('2000', 23, 'dev', now());

ALTER TABLE staffs ADD INDEX idx_staffs_nameAgePos(name, age, pos);

```

**避免索引失效：**
1.最好使用全值匹配（索引都用在where后作为条件）
建立几个复合索引字段，最好就用上几个字段。**且按照顺序来用**。

我们下面来看下面三个语句：

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July';
EXPLAIN SELECT * FROM staffs WHERE name = 'July' AND age=25;
EXPLAIN SELECT * FROM staffs WHERE name = 'July' AND age=25 AND pos='dev';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/12.png)

2.最佳左前缀法则（指的是从索引的左前列开始并且不跳过索引的列）
如果索引了多列，要遵守**最左前缀法则**，指的是查询从索引的最左前列开始，不跳过索引中间的列。（带头大哥不能死，中间兄弟不能丢）


比如下面这个就索引失效了：
```sql
EXPLAIN SELECT * FROM staffs age=25 AND pos='dev';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/13.png)



3.不要在索引上做任何操作（计算、函数、（自动or手动）类型转换），会导致索引失效而转向全表扫描

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July';
EXPLAIN SELECT * FROM staffs WHERE LEFT(name, 4) = 'July';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/14.png)


4.储存引擎不能使用索引中范围条件右边的列（用了>,<,like等右边的索引全部失效）

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July' AND age = 25 AND pos = 'dev';
EXPLAIN SELECT * FROM staffs WHERE name = 'July' AND age < 25 AND pos = 'dev';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/15.png)

将age字段条件从=改成了<，查出的是个范围，所以可发现第三个字段pos索引失效了，因为type类型低了，key_len短了。ref也变成了range。



5.尽量使用覆盖索引（只访问索引的查询），减少select *

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July';
EXPLAIN SELECT name FROM staffs WHERE name = 'July';
EXPLAIN SELECT name,age FROM staffs WHERE name = 'July';
```
![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/16.png)

可以发现，若将替换成索引列的话会用到Using index，直接从索引读，效果更佳，数据量大的时候更明显

```sql
EXPLAIN SELECT * FROM staffs WHERE name = 'July' AND age < 25 AND pos = 'dev';
EXPLAIN SELECT name, age FROM staffs WHERE name = 'July' AND age < 25 AND pos = 'dev';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/17.png)


可以发现，范围查找时，若将替换成索引列的话不仅会用到Using index索引级别还会是ref，key_len也短，效果更佳，数据量大的时候更明显.


6.Mysql在使用不等于(!=、<>)或like的左模糊的时候无法试用索引会导致全表扫描。
```sql
 EXPLAIN SELECT * FROM staffs WHERE name != 'July';
EXPLAIN SELECT * FROM staffs WHERE name LIKE '%July';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/18.png)



7.is null，is not null也无法使用索引

```sql
EXPLAIN SELECT * FROM staffs WHERE name IS NULL;
EXPLAIN SELECT * FROM staffs WHERE name IS NOT NULL;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/19.png)



8.like以通配符开头（’%abc….’)mysql索引失效会变成全表扫描的操作（%尽量放在右边，要使用%开头则要使用覆盖索引）
```sql
EXPLAIN SELECT * FROM staffs WHERE name LIKE '%July%';
EXPLAIN SELECT * FROM staffs WHERE name LIKE '%July';
EXPLAIN SELECT * FROM staffs WHERE name LIKE 'July%';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/20.png)

**解决 like `%str%` 索引失效的方法？**
使用覆盖索引：
```sql
EXPLAIN SELECT name,age FROM staffs WHERE name LIKE '%July%';
EXPLAIN SELECT id,name,age FROM staffs WHERE name LIKE '%July%';
EXPLAIN SELECT name,age,add_time FROM staffs WHERE name LIKE '%July%';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/21.png)

需要注意的是，select 后面的字段要和索引沾边，如果不沾边，就变为全表扫描。


9.字符串不加单引号索引失效

```sql
EXPLAIN SELECT * FROM staffs WHERE name = '2000';
EXPLAIN SELECT * FROM staffs WHERE name = 2000;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/22.png)

10.少用or，用它来连接时会索引失效
```sql
EXPLAIN SELECT * FROM staffs WHERE name = '2000' or name = 'z3';
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/23.png)

总结：

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/24.png)


一道关于索引的小题目：

1.建表：

```
create table test03(
id int primary key not null auto_increment,
c1 char(10),
c2 char(10),
c3 char(10),
c4 char(10),
c5 char(10)
);

insert into test03(c1,c2,c3,c4,c5) values('a1','a2','a3','a4','a5');
insert into test03(c1,c2,c3,c4,c5) values('b1','b2','b3','b4','b5');
insert into test03(c1,c2,c3,c4,c5) values('c1','c2','c3','c4','c5');
insert into test03(c1,c2,c3,c4,c5) values('d1','d2','d3','d4','d5');
insert into test03(c1,c2,c3,c4,c5) values('e1','e2','e3','e4','e5');

```

2.建索引：

```sql
create index idx_test03_c1234 on test03(c1,c2,c3,c4);
show index from test03;
```

3.问题：
我们创建了复合索引idx_test03_c1234，根据以下SQL分析下索引使用情况？
```sql
explain select * from test03 where c1='a1'; 
explain select * from test03 where c1='a1' and c2='a2'; 
explain select * from test03 where c1='a1' and c2='a2' and c3='a3'; 
explain select * from test03 where c1='a1' and c2='a2' and c3='a3' and c4='a4'; 
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/25.png)

匹配的精度越来越高。


```sql
explain select * from test03 where c4='a4' and c3='a3' and c2='a2' and c1='a1'; 
explain select * from test03 where c4='a4' and c3='a3' and c2='a2' and c1='a1'; 
explain select * from test03 where c1='a1' and c2='a2' and c3>'a3' and c4='a4'; 
explain select * from test03 where c1='a1' and c2='a2' and c4>'a4' and c3='a3'; 
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/26.png)

第三条c4无法使用。

```sql
explain select * from test03 where c1='a1' and c2='a2' and c4='a4' order by c3;
```
![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/27.png)

c1,c2用到索引，c3在这里的作用是排序而不是查找。


```sql
explain select * from test03 where c1='a1' and c2='a2' order by c3;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/28.png)

同上一条执行计划一样，与c4='a4'没有关系

```sql
explain select * from test03 where c1='a1' and c2='a2' order by c4;
```
![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/29.png)

c1,c2用到索引，Extra中出现了Using filesort。

```sql
explain select * from test03 where c1='a1' and c5='a5' order by c2,c3;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/30.png)

只用c1一个字段索引，但是c2、c3用于排序，无Using filesort

```sql
explain select * from test03 where c1='a1' and c5='a5' order by c3,c2;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/31.png)

有Using filesort，我们建的索引是c1,c2,c3,c4，它没有按照顺序来，3和2颠倒了。


```sql
explain select * from test03 where c1='a1' and c2='a2' order by c2,c3;
explain select * from test03 where c1='a1' and c2='a2' and c5='a5' order by c2,c3;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/32.png)

用c1、c2两个字段索引，但是c2、c3用于排序，无Using filesort


```sql
explain select * from test03 where c1='a1' and c2='a2' and c5='a5' order by c3,c2;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/33.png)

本例有常量c2(排序字段)的情况，所以无Using filesort，与`explain select * from test03 where c1='a1' and c5='a5' order by c3,c2;`例子不同。

```sql
explain select * from test03 where c1='a1' and c4='a4' group by c2,c3;
explain select * from test03 where c1='a1' and c4='a4' group by c3,c2;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/34.png)

第二条有Using temporary，有Using filesort。

我们对这个题目进行个小结：
a. group by 分组之前必排序，所以说它与order by排序法则与索引优化原则几乎是一致的。
b. 在索引分析的时候，定值为常量，范围之后是索引失效，最终看排序，一般order by是给个范围。
c. group by 基本上都需要进行排序，会有临时表产生。



**优化总结口诀**：
全值匹配我最爱，最左前缀要遵守； 
带头大哥不能死，中间兄弟不能断； 
索引列上少计算，范围之后全失效； 
Like百分写最右，覆盖索引不写星； 
不等空值还有or，索引失效要少用； 
VAR引号不可丢，SQL高级也不难；


那么我们如何进行一步步分析呢？
1.观察，至少跑1天，看看生产的慢SQL情况。
2.开启慢查询日志，设置阙值，比如超过5秒钟的就是慢SQL，并将它抓取出来。
3.explain+慢SQL分析。
4.show profile。
5.运维经理或者DBA进行SQL数据库服务器的参数调优。

提取一下就是：
0.为了避免出现问题，我们需要进行查询优化。
1.慢查询日志的开启与捕获。
2.explain+慢SQL分析。
3.show profile查询SQL在MySQL服务器中的执行细节和生命周期情况。
4.SQL数据库服务器的参数调优。

## 查询优化

### 小表驱动大表

小表驱动大表，即小的数据集驱动大的数据集。

1). 当B表的数据集小于A表的数据集时，用in优于exists.
select * from A where id in (select id from B);
等价于for循环:
for select id from B;
for select * from A where A.id = B.id;

2). 当B表的数据集大于A表的数据集时，用exists优于in.
select * from A where exists (select 1 from B where B.id=A.id);
等价于for循环:
for select * from A;
for select * from B where B.id = A.id;

in 是把外表和内表作hash 连接；
exists 是对外表作loop循环，每次loop循环再对内表进行查询。

总结起来就是：in+小表，exists+大表

注意：A表与B表的ID字段应建立索引。

### order by关键字优化

1.建表和索引：
```sql
CREATE TABLE tblA(
    #id INT PRIMARY KEY NOT NULL AUTO_INCREMENT,
    age INT,
    birth TIMESTAMP NOT NULL
);

INSERT INTO tblA(age, birth) VALUES(22, NOW());
INSERT INTO tblA(age, birth) VALUES(23, NOW());
INSERT INTO tblA(age, birth) VALUES(24, NOW());

CREATE INDEX indx_A_ageBirth ON tblA(age, birth);
```

2.执行SQL：
（1）
```sql
EXPLAIN SELECT * FROM tblA WHERE age > 20 ORDER BY age;
EXPLAIN SELECT * FROM tblA WHERE age > 20 ORDER BY age, birth;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/35.png)

age一直在，所以不会产生Using filesort，因为索引从age开始，而order by正好也是从age开始，age索引可以用上。   

（2）
```sql
EXPLAIN SELECT * FROM tblA WHERE age > 20 ORDER BY birth;
EXPLAIN SELECT * FROM tblA WHERE age > 20 ORDER BY birth, age;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/36.png)

age不再order by范围内了,前面where条件还是大于号（范围值），索引断了，所以排序的时候无法用到索引，产生了Using filesort。

（三）
```sql
EXPLAIN SELECT * FROM tblA ORDER BY birth;
EXPLAIN SELECT * FROM tblA WHERE birth > '2011-11-11 11:11:11' ORDER BY birth;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/37.png)

age完全没用上，产生了Using filesort。

（四）
```sql
EXPLAIN SELECT * FROM tblA WHERE birth > '2011-11-11 11:11:11' ORDER BY age;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/38.png)

不难发现，order by开头用到了age，索引不会产生Using filesort

（五）
```sql
EXPLAIN SELECT * FROM tblA ORDER BY age ASC, birth DESC;
EXPLAIN SELECT * FROM tblA ORDER BY age DESC, birth ASC;
EXPLAIN SELECT * FROM tblA ORDER BY age ASC, birth ASC;
EXPLAIN SELECT * FROM tblA ORDER BY age DESC, birth DESC;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/39.png)

排序的话要么都升序，要么都降序，否则mysql内部会不知道怎么排序，排序也是索引的一部分（Btree），所以不一致时会产生Using filesort

（六）
```sql
EXPLAIN SELECT * FROM tblA WHERE age = 20 ORDER BY birth;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/40.png)

没有Using filesort，是因为where 条件是个常量，而不是个范围，所以会自动传递到order by，所以order by 起始排序字段还是age。

order by 支持两种排序方式：Index（索引排序）和FileSort（文件排序），但是index效率高。因此尽量使用Using Index方式排序，避免使用Using FileSort方式排序、
尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀的原则

ORDER BY满足两情况，会使用Index方式排序：
①ORDER BY 语句使用索引最左前列；
②使用Where子句与Order BY子句条件列组合满足索引最左前列。

如果不在索引列上，filesort有两种算法：mysql就要启动**双路排序和单路排序**
双路排序：MySQL 4.1之前是使用双路排序,字面意思就是两次扫描磁盘，最终得到数据.
单路排序 : 从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，**它的效率更快一些**，避免了第二次读取数据。并且把随机IO变成了顺序IO,但是它会使用更多的空间，因为它把每一行都保存在内存中了。

由于单路是后出的，总体而言好过双路。但是用单路有问题

在sort_buffer中，方法B比方法A要多占用很多空间，因为方法B是把所有字段都取出, 所以有可能取出的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并），排完再取sort_buffer容量大小，再排…… 从而多次I/O。

**解决方案：**
1.order by 时select * 是一个大忌，只select 需要的字段，而不要用 *，当Query的字段大小总和小于max_length_for_sort_data而且排序字段不是text|bolb类型，会用改进后的算法-单路排序，否则会用老算法-双路排序。两种算法的数据都有可能超出sort_buffer的容量，超出之后，会创建temp文件进行合并排序，导致多次I/O，使用单路排序算法的风险会大一些，所以要提高sort_buffer_size。

2. 提高sort_buffer_size，不管用哪种算法，提高这个参数，都会提高效率，当然要根据系统的能力去提高，因为这个参数是针对每个进程的

3.提高max_length_for_sort_data，会增加用改进算法的概率，但是如果设置太高，数据总量超过sort_buffer_size的概率就会增大，明显症状是，高的磁盘I/O活动和低的cup使用率。

### group by关键字优化

1.group by 是先排序后分组，用到索引跟order by 一样，遵从索引最佳左前缀原则
2.当无法使用索引列时，要加大sort_buffer_size和max_length_for_sort_data的值
3.where高于having，能写到where限定条件里的，就不要去having中限定。

## 慢查询日志

### 慢查询日志介绍

MySQL的慢查询日志是MySQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过**long_query_time**值的SQL，则会被记录到慢查询日志中。

具体指运行时间超过long_query_time值的SQL，则会被记录到慢查询日志中。long_query_time的默认值为10，意思是运行10秒以上的语句。


### 慢查询日志使用

**mysql数据库中开启慢查询：**

默认slow_query_log的值为off,表示慢查询日志是禁用的：
`show variables like '%slow_query_log%';`

开启:(只对当前开启的数据库当时生效，如果mysql重启后就失效了)
`set global slow_query_log=1;`

查看慢查询阙值时间:
`show variables like 'long_query_time%';`

设置阙值时间:
`set global long_query_time=3;`

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/41.png)

为什么设置阙值时间后，还是默认的10秒？
需要重新连接或者新开一个会话才能看到修改值。

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/42.png)




查看当前系统有多少条慢查询记录:
`show global status like '%slow_queries%';`

如果要以上参数永久生效，需要修改my.cnf配置文件
```
[mysqld]
slow_query_log=1
slow_query_log_file=/var/lib/mysql/zln117-slow.log
long_query_time=3;
log_output=FILE
```
slow_query_log_file表示慢查询日志存放的位置。

我们使用select sleep(4);

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/43.png)

### 日志分析工具mysqldumpslow

使用mysqldumpslow命令可以非常明确的得到各种我们需要的查询语句，对MySQL查询语句的监控、分析、优化是MySQL优化的第一步，也是非常重要的一步。


mysqldumpslow的帮助信息：
-s 表示按照何种方式排序
-c 访问次数
-l 锁定时间
-r 返回记录
-t 查询时间
-al 平均锁定时间
-ar 平均返回记录数
-at 平均查询时间
-t 即为返回前面多少条的数据
-g 后边搭配一个正则匹配模式，大小写不敏感的

示例参考：
1.得到返回记录集最多的10个SQL:
`mysqldumpslow -s r -t 10 /var/lib/mysql/zln117-slow.log`


2.得到访问次数最多的10个SQL:
`mysqldumpslow -s c -t 10 /var/lib/mysql/zln117-slow.log`

3.得到按照时间排序的前10条里面含有左连接的查询语句:
`mysqldumpslow -s t -t 10 -g "left join" /var/lib/mysql/zln117-slow.log`

4.另外建议在使用这些命令时结合|和more使用，否则有可能出现爆爆的情况:
`mysqldumpslow -s r -t 10 /var/lib/mysql/zln117-slow.log | more`

## 批量数据脚本

1.建库建表

```sql
#建库
create database bigdata;
use bigdata;

#创建部门表dept
create table dept(
id int unsigned primary key auto_increment,
deptno mediumint unsigned not null default 0,
deptname varchar(20) not null default '',
loc varchar(13) not null default ''
)engine=innodb default charset=gbk;

#创建员工表emp
create table emp(
id int unsigned primary key auto_increment,
empno mediumint unsigned not null default 0, /*编号*/
empname varchar(20) not null default '', /*名字*/
job varchar(9) not null default '', /*工作*/
mgr mediumint unsigned not null default 0, /*上级编号*/
hiredate date not null, /*入职时间*/
sal decimal(7,2) not null, /*薪水*/
comm decimal(7,2) not null, /*红利*/
deptno mediumint unsigned not null default 0 /*部门编号*/
) engine=innodb default charset=gbk;
```

2.设置参数log_bin_trust_function_creators
创建函数时，如果报错：`ERROR 1418 (HY000): This function has none of DETERMINISTIC, NO SQL, or READS SQL DATA in its declaration  and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable)`

由于开启过慢查询日志，因为我们开启了bin-log，我们就必须为我们的function指定一个参数。:
```
show variables like 'log_bin_trust_function_creators';
set global log_bin_trust_function_creators=1;
```
这样添加了参数以后，如果mysqld重启，上述参数又会消失，使其永久有效的做法是：

在windows下配置my.ini或者在linux下配置my.cnf：
```
[mysqld]
log_bin_trust_function_creators=1
```

3.创建函数，保证每条数据都不同
```sql
#随机产生字符串
delimiter $$

create function rand_string(n int) returns varchar(255)

begin

declare init_str varchar(100) default 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
declare return_str varchar(255) default '';
declare i int default 0;

while i < n do
set return_str = concat(return_str, 
substring(init_str,floor(1+rand()*52),1));
set i=i+1;
end while;
return return_str;

end $$


#假如要删除
#drop function rand_string;

#随机产生部门编号
delimiter $$
create function rand_num()
returns int(5)
begin
declare i int default 0;
set i = floor(100+rand()*10);
return i;
end $$

#假如要删除
#drop function rand_num;
```


4.创建存储过程

```sql
delimiter $$
create procedure insert_emp(in start_num int(10),in max_num int(10))

begin
declare i int default 0;
## 把自动提交写为0，commit提交
set autocommit=0;
repeat
set i = i+1;
insert into emp(empno,empname,job,mgr,hiredate,sal,comm,deptno) values((start_num+i),rand_string(6),'salesman',0001,curdate(),2000,400,rand_num());
until i = max_num
end repeat;
commit;
end $$

delimiter $$
create procedure insert_dept(in start_num int(10),in max_num int(10))
begin
declare i int default 0;
set autocommit=0;
repeat
set i = i+1;
insert into dept(deptno,deptname,loc) values((start_num+i),rand_string(10),rand_string(8));
until i = max_num
end repeat;
commit;
end $$

```

5.调用存储过程

```sql

delimiter ;
call insert_dept(100,10);

#执行存储过程，往emp表添加50万条数据
delimiter ;
call insert_emp(100001,500000);
```

至此50万条数据插入完毕。


## Show Profile分析SQL
mysql提供用来分析当前会话中语句执行的资源消耗情况，可以用于SQL的调优测量。

分析步骤：
1.是否支持，看看当前的mysql版本是否支持：
`show variables like 'profiling%';`

2.开启功能，默认是关闭的，使用前需要开启
`set profiling=on;`

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/44.png)

3.运行sql:
```
select * from emp group by id%10 limit 150000;
select * from emp group by id%20 order by 5;
```

4.查看结果:
```
show profiles;
```

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/45.png)

5.诊断SQL:  
`show profile cpu,block io for query id;`
(id是上一步通过show profiles查询的Query_ID数值)；

![MySQL数据库深入（二）](http://img.bcoder.top/2018.02.21/46.png)

show profile 后面还可以有很多参数，来查看执行的详细情况，其中我们主要用的是cpu，block io。详细参数介绍：
all ：显示所有的开销信息
block io ： 显示io相关开销
context switches ： 上下文切换相关开销
cpu ： 显示cpu相关开销
ipc ： 显示发送和接收相关开销信息
memory ： 显示内存相关开销信息
page faults ： 显示页面错误相关开销信息
source ：显示source_function,source_file,source_line相关的开销信息
swaps ： 显示交换次数相关的开销信息


6.日常开发需要注意到结论:
1). status 出现converting HEAP to MyISAM 查询结果太大，内存都不够用了只能往磁盘上搬了。
2). status 出现creating tmp table 创建临时表
拷贝数据到临时表；用完再删除。
3). status 出现copying to tmp table on disk 把内存中临时表复制到磁盘，危险！！！
4). status 出现locked 锁表


**全局查询日志：**

开启全局查询日志以后，所有的sql都会被记录下来（**生产环境不要开启这个**）

可以在my.cnf中配置开启，设置
```
general_log=1，
general_log_file=/路径，
log_output=FILE或者log_output=TABLE，
```

设置了输出到文件中，那么所有的sql都被记录到了general_log_file中配置的路径里面，如果设置了输出到table，那么所有的记录都被记录到了mysql库中的general_log表中。

可以使用命令开启，命令开启的mysql重启以后将失效； 
执行命令:

```sql
set global general_log = 1;
set global log_output='TABLE';
```

之后所有的sql都将被记录到general_log中。使用select * from mysql.general_log 将查处所有的sql。