# Redis基础知识（一）

> data:2018.11.07

## NoSQL简介

### NoSQL分类

NoSQL，泛指非关系型的数据库，NoSQL数据库的四大分类：

**键值(Key-Value)存储数据库**：这一类数据库主要会使用到一个哈希表，这个表中有一个特定的键和一个指针指向特定的数据
相关产品： `Redis`，`Voldemort`，`Oracle BDB`、`Tokyo Cabinet/Tyrant`、`Berkeley DB`
**典型应用**： 内容缓存，主要用于处理大量数据的高访问负载。
**数据模型**： 一系列键值对
**优势**： 快速查询
**劣势**： 存储的数据缺少结构化


**列存储数据库**：这部分数据库通常是用来应对分布式存储的海量数据。键仍然存在，但是，它们的特点是指向了多个列。
**相关产品**：`Cassandra`, `HBase`, `Riak`
**典型应用**：分布式的文件系统
**数据模型**：以列簇式存储，将同一列数据存在一起
**优势**：查找速度快，可扩展性强，更容易进行分布式扩展
**劣势**：功能相对局限

**文档型数据库**:该类型的数据模型是版本化的文档，半结构化的文档以特定的格式存储，比如JSON。文档型数据库可以看作是键值数据库的升级版，允许之间嵌套键而且文档型数据库比键值数据库的查询效率更高。
**相关产品**：`CouchDB`、`MongoDB`
**典型应用**：Web应用（与Key-Value类似，Value是结构化的）
**数据模型**： 一系列键值对
**优势**：数据结构要求不严格
**劣势**： 查询性能不高，而且缺乏统一的查询语法

**图形(Graph)数据库**：图形结构的数据库同其他行列以及刚性结构的SQL数据库不同，它是使用灵活的图形模型，并且能够扩展到多个服务器上。NoSQL数据库没有标准的查询语言(SQL)，因此进行数据库查询需要制定数据模型。许多NoSQL数据库都由REST式的数据接口或者查询API。
**相关产品**：`Neo4J`、`InfoGrid`、`Infinite Graph`
**典型应用**：社交网络
**数据模型**：图结构
**优势**：利用图结构相关算法。
**劣势**：需要对整个图做计算才能得出结果，不容易做分布式的集群方案


### 非关系型数据库特点

1、数据模型比较简单。

2、需要灵活性更强的IT系统。

3、对数据库性能要求较高。

4、不需要高度的数据一致性。

5、对于给定key，比较容易映射复杂值的环境。

## Redis简介


是以**key-value**形式存储，和传统的关系型数据库不一样，不一定遵循传统数据库的一些基本要求(非关系型的，分布式的，开源的，水平可扩展的)

### Redis优缺点

优点：
对数据高并发读写
对海量数据的高效率存储和访问
对数据的可扩展性和高可用性
Redis具备一定的可靠性

缺点：
redis(ACID处理非常简单)
无法做到太复杂的关系数据库模型

Redis是以key-value store存储，data structure service 数据结构服务器。键可以包含：**(string)字符串，哈希，(list)链表，(set)集合，(zset)有序集合**。这些数据集合都支持push/pop、add/remove及取交集和并集以及更丰富的操作，redis支持各种不同的方式排序，为了保证效率，数据都是缓存在内存中，它也可以周期性的把更新的数据写入磁盘或者把修改操作写入追加到文件里。

下面我们对有点进行简略的分析：

### 高并发读写

我们来看下面一个图：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/1.png)

通常，我们会将Redis服务器进行集群，并且采用主从模式（多个主节点和多个从节点），主节点主要负责数据的写（也可以进行读数据操作），从节点只负责读取数据，主节点会将数据定时或者实时（根据业务需要进行设计）的同步到从节点。从而实现数据的一致性，保证了并发访问。

### 可扩展性

我们来看下面这张图：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/2.png)

在我们的生产环境中，有时候我们的Redis服务器的硬盘会存在数据满载的情况，如果继续进行写入，主节点将会出现意想不到的故障，那么我们就要对服务器进行拓展，一般有两种方式，水平拓展和垂直拓展，

我们先来看一下水平拓展，水平拓展简单来说，就是加机器，添加若干台主节点和从节点，新写入的数据放入新的主从节点中。

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/3.png)

垂直拓展的意思就是直接往相关的机器中添加硬盘。

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/4.png)

### 高可用性

主从模式的好处显而易见，但是坏处也是显而易见的，也就是会出现单点故障的问题（单台主节点故障，导致整个主从网络无法正常工作）

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/5.png)

而我们所说的高可用性，就是不止一个主节点（比如两个主节点，另一个主节点为备份主节点），当主节点宕机时，备份主节点会迅速接替主节点的工作，成为新的主节点，从而保证其正常运行。

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/6.png)

### Redis可靠性

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/7.png)

Redis可靠性主要体现在对缓存数据的持久化。Redis支持两种持久化的方式：RDB和AOF,其中，RDB是定期将数据存到硬盘，AOF是实时存Redis的操作命令（相当于保存操作的脚本），一般来说，我们选用AOF这种方式来保证Redis可靠性。

### Redis模式

Redis主要有三种模式，分别为主从模式，哨兵模式以及集群模式。

之前我们就已经见过主从模式，主从模式是指由一个主节点，若干个从节点。优点之前也说过，能够保证高并发读写，缺点就是单点失效。

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/8.png)

为了避免单点失效的问题，产生了一种新的模式，叫做哨兵模式，哨兵模式如下图所示：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/9.png)


如上图所示，我们有一台哨兵节点，用来监控三台机器，一旦主节点故障，哨兵会进行选举从节点转变成主节点，原先的主节点恢复后，变成从节点。
这种方式的优点是解决了单点失效的问题，缺点是在主节点故障时选举新的主节点需要一定时间，同时数据也可以会丢失。

下面来介绍一下集群模式：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/10.jpg)

M1，M2，M3为redis三个主节点，S1，S2，S3为redis三个从节点，分别为M1，M2，M3备份数据以及故障切换使用。APP访问数据库可以通过连接任意一个Master节点实现。在三个Master节点的redis集群中，只容许有一个Master出故障，当多于一个Master宕机时，redis即不可用。当其中一个Master出现故障，其对应的Slave会接管故障Master的服务，保证redis 数据库的正常使用。


## Redis安装

详细的安装步骤：

1 首先需要安装gcc，把下载好的redis-3.0.0-rc2.tar.gz 放到linux /usr/local文件夹下（下载地址http://redis.io/download）

2 进行解压:`tar -zxvf redis-3.0.0-rc2.tar.gz`

3 进入到redis-3.0.0目录下，进行编译 make
编译完成后，如下图所示：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/11.png)

4 进入到src下进行安装`make install`验证(ll查看src下的目录，有redis-server 、redis-cil即可)

5 建立2个文件夹存放redis命令和配置文件
`mkdir -p /usr/local/redis/etc`
`mkdir -p /usr/local/redis/bin`

6 把redis-3.0.0下的redis.conf 移动到/usr/local/redis/etc下，
`cp redis.conf /usr/local/redis/etc/`

7 把redis-3.0.0/src里的mkreleasehdr.sh、redis-benchmark、redis-check-aof、redis-check-dump、redis-cli、redis-server 
文件移动到bin下，命令：
`mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-server /usr/local/redis/bin`

最后redis目录如下图:

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/12.png)

8 启动时并指定配置文件：`./redis-server /usr/local/redis/etc/redis.conf`
（注意要使用后台启动，所以修改redis.conf里的 daemonize 改为yes)

9 验证启动是否成功：
`ps -ef | grep redis`查看是否有redis服务
或者查看端口：`netstat -tunpl | grep 6379`

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/13.png)

进入redis客户端 ./redis-cli 退出客户端quit
退出redis服务： 
（1）pkill redis-server 
（2）kill 进程号
（3）/usr/local/redis/bin/redis-cli shutdown

## Redis 数据类型

redis一共分为5种基本数据类型：**String、Hash、List、Set、Zset**

### String类型

String类型是包含很多种类型的特殊类型，并且是二进制安全的。比如序列化的对象进行存储，比如一张图片进行二进制存储，比如一个简单的字符串、数值等等.


设置值**set**：`set name zln` 
取值**get**: `get name` (说明 设置name多次会覆盖)
删除值**del**：`del name`

使用**setnx** (not exist)
`setnx name` 如果不存在进行设置，存在就不需要进行设置，返回0

使用**setex** (expired)
`setex color 10 red` 设置color的有效期为10秒，10秒后返回nil (在redis里nil表示空)

使用**setrange** 替换字符串:
`set email aaabbbccc@qq.com`
`setrange email 10 ww` (10表示从第几位开始替换，后面跟上替换的字符串)

操作如下：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/14.png)


使用一次性设置多个和获取多个值的**mset**，**mget**方法：
`mset name zln address nanjing sex man`
对应的`mget name address sex`

对应的也有**msetnx**和**mget**方法。

一次性设置和取值的**getset**方法；
`set key cc`
`getset key changchun` 返回旧值并设置新值的方法。

**incr和decr**方法：对某一个值进行递增和递减

**incrby和decrby**方法：对某个值进行指定长度的递增和递减 `incrby key 【步长】`

**append** [name]方法：字符串追加方法

**strlen** [name]方法：获取字符串的长度

操作如下：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/15.png)

### Hash类型

Hash类型是String类型的filed和value的映射表，或者说一个String集合。它特别适合存储对象，相比较而言，将一个对象类型存储在Hash类型里要比存储在String类型里占用更少的内存空间，并方便存取整个对象。

形如：`hset myhash field hello`（含义是**hset**是hash集合，myhash是集合的名字 filed1是字段名 hello是其值）

使用`hget myhash field`获取内容，也可以存储多个值。

**hmset**可以进行批量存储多个键值对:`hmset myhash sex man addr beijing`

也可以使用**hmget**进行批量获取多个键值对：`hmget myhash sex addr`

同样也有**hsetnx**,和**setnx**类似；

**hincrby**集合递增和递减

**hexists** 是否存在key，如果存在返回，不存在返回0

**hlen** 返回hash几个里的所有的键数值

**hdel** 删除指定hash的field

**hkeys** 返回hash里所有的字段

**hvals** 返回hash的所有value

**hgetall** 返回hash里所有的key和value

操作如下：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/16.png)

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/17.png)

### List类型

List类型是一个链表结构的集合，其主要功能有push、pop、获取元素等。更详细地说，List类型是一个双端链表结构。我们可以通过相关操作进行集合的头部或尾部添加删除元素，List的设计非常简单巧妙，既可以作为栈，又可以作为队列，满足绝大多数需求。

注：list中元素可重复


常用命令

1）**lpush**:从头部加入元素（栈） 先进后出
`lpush list1 "hello"`
`lpush  list1 "world"`

2）**rpush**:从尾部加入元素（队列） 先进先出
`rpush list2 1`
`rpush list2 "2"`

3）**linsert**:插入元素
`linsert list3 before [集合的元素] [插入的元素]`

4）**lrange**取值
`lrange list1 0 -1` (表示从头取到末尾)

5）**lset**:替换元素
`lset list1 0 10` ， 把第0个元素替换为10

6）删除元素
**lrem**方法：删除多个重复元素：`lrem list2 2 1` ，删除2个1元素
**lpop**方法：从list的头部删除元素，并返回删除元素
**rpop**方法：从list的尾部删除元素，并返回删除元素
**ltrim**方法： 保留指定key的值范围内的数据

7）其他命令
**rpoplpush**方法： 第一步从尾部删除元素，然后第二步并从头部加入元素
**lindex**方法： 返回名称为key的list中 index位置的元素
**llen**方法：返回元素的个数

操作如下：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/18.png)

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/19.png)

### Set类型

集合类型常用操作是像集合中加入或删除元素、判断元素是否存在等，内部使用值为空的散列表实现，所以这些操作的复杂度都为O(1)，多个集合类型的键之间还可以进行并集、交集、差集的运算。

常用命令

1）添加、查看元素：**sadd** ，**smembers**

2）删除元素
**srem**删除方法：srem set1 aaa  //删除aaa元素
**spop**随机删除方法：spop set1//随机删除某个元素，返回删除的key

3）集合之间的比较操作
**sdiff**方法：返回俩个集合的不同元素(哪个集合在前面就以哪个集合为标准)
**sdiffstore**方法： 将返回的不同元素存储到另外一个集合里
**sinter**方法：返回集合的交集
**sinterstroe**方法： 返回交集结果，存入set3中
**sunion**方法： 取并集
**sunionstore**方法：取得并集，存入set3中
**smove**方法： 从一个set集合移动到另一个set集合里  
`smove set1 set2 1`  //将set1中的元素aaa移动到set2中（相当于剪切复制）
**scard**方法： 查看集合里元素个数
**sismember**方法： 判断某元素是否为集合中的元素:返回1代表是集合中的元素，0代表不是
**srandmember**方法： 随机返回一个元素

操作如下：

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/20.png)

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/21.png)


### Zset类型

它与集合类型区别在于“有序”二字，有序集合在某些方面与列表类型也比较**类似**：
1）二者都是有序的；
2）二者都可以获得某一范围内的元素；

当然也有明显的**区别**：
1）List通过链表实现，获取两端数据极快，而取中间的数据较慢；
2）Zset是使用散列表和跳跃表实现，即使位于中间的数据读取也很快，复杂度为： O(log(N))；
3）List不能简单的调整元素的位置，但是Zset可以；
4）Zset比List更耗费内存。

常用命令：

1）**zadd**添加方法
`zadd zset1 1 one`
向有序集合中添加一个元素，该元素如果存在，则更新顺序，在重复插入的时候 会根据顺序属性更新，其中的“1”为插入时指定的下标。

2）删除方法
**zrem** 删除名称为key的zset中的元素member
**zremrangebyrank** 删除 1到1（只删除索引1）
**zremrangebyscore** 删除指定序号

3）**zincrby** 以指定值去自动递增或者减少，用法和之前的incrby类似

4）**zrangebyscore**  找到指定区间范围的数据进行返回

5）排序索引
**zrank** 返回排序索引 从小到大排序（升序排序之后再找索引） 注意 一个是顺序号 一个是索引 zrank返回的是索引
**zrevrank** 返回排序索引 从大到小排序（降序排序之后再找索引）

6）**zcard** 返回集合里所有元素的个数

7）**zcount**  返回集合中score在给定区间中的数量

8）**zremrangebyrank** zset [from] [to]（删除索引）

9）**zremrangebyscore** zset [from] [to] (删除指定序号)

![Redis基础知识（一）](http://img.bcoder.top/2017.11.24/22.png)