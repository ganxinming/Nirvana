#### InnoDB和MyISAM区别：

1.默认支持事物(可重复读)，commit和rollback/不支持事物

2.支持行级锁(能并发/支持表级锁)/写的时候给整个表加锁

3.支持崩溃后恢复/不支持

4.支持外键(默认主索引:聚簇索引)/不支持外键

### MyISAM特性:

设计简单，数据以紧密格式存储，支持压缩表、空间数据索引。适用于读多写少的情况。

### InnoDB特性：

MVCC(多版本并发控制) 通过行级锁和版本号控制并发:在可重复读隔离级别下，通过多版本并发控制（MVCC）+ Next-Key Locking 防止幻影读。

#### 类型：

整型:

TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT 分别使用 8, 16, 24, 32, 64 位存储空间，一般情况下越小的列越好。INT(11) 中的数字只是规定了交互工具显示字符的个数，对于存储和计算来说是没有意义的。

浮点数:

FLOAT 和 DOUBLE 为浮点类型，DECIMAL 为高精度小数类型。CPU 原生支持浮点运算，但是不支持 DECIMAl 类型的计算，因此 DECIMAL 的计算比浮点类型需要更高的代价。

FLOAT、DOUBLE 和 DECIMAL 都可以指定列宽，例如 DECIMAL(18, 9) 表示总共 18 位，取 9 位存储小数部分，剩下 9 位存储整数部分。

字符串:

主要有 CHAR 和 VARCHAR 两种类型，一种是定长的，一种是变长的。在进行存储和检索时，会保留 VARCHAR 末尾的空格，而会删除 CHAR 末尾的空格。

时间和日期：

dateTime:保存1000年到9999年的日期和时间，8字节。timestamp:只能表示从 1970 年到 2038 年。4字节。应该尽量使用 TIMESTAMP，因为它比 DATETIME 空间效率更高。

文本：

blob和text：BLOB能用来保存二进制数据，比如照片(不过一般不存照片)；而TEXT只能保存字符数据。必须指定索引前缀的长度

## 数据量越来越大时：分表分库(两者都可以水平切分，垂直切分)

**水平切分**：将同一个表中的记录拆分到多个相同表结构中。但是单独把表分离出来没啥意义，还在一个数据库里并没有缓解压力。此时应该分库，将分出来的表放置不同库中。(一般是根据ID进行取模来决定数据库)

- 优化单一表数据量过大而产生的性能问题

- 避免IO争抢并减少锁表的几率

  库内的水平分表，解决了单一表数据量过大的问题，分出来的小表中只包含一部分数据，从而使得单个表的数据量变小，提高检索性能。

  **通常我们按以下原则进行水平拆分:**

  1.单库存储量及性能瓶颈。

  2.当一个应用难以再细粒度的垂直切分，或切分后数据量行数巨大，存在单库读写、存储性能瓶颈，这时候就需要进行水平分库了，经过水平切分的优化。

**垂直切分**:垂直切分是将一张表按列切分成多个表，通常是按照列的**关系密集程度**进行切分,相关联的在一起，也可以利用垂直切分将经常被使用的列和不经常被使用的列切分到不同的表中。

- 解决业务层面的耦合，业务清晰

- 能对不同业务的数据进行分级管理、维护、监控、扩展等

- 高并发场景下，垂直分库一定程度的提升IO、数据库连接数、降低单机硬件资源的瓶颈

  垂直分库通过将表按业务分类，然后分布在不同数据库，并且可以将这些数据库部署在不同服务器上，从而达到多个服务器共同分摊压力的效果，但是依然没有解决单表数据量过大的问题。

**通常我们按以下原则进行垂直拆分:**

1. 把不常用的字段单独放在一张表;

2. 把text，blob等大字段拆分出来放在附表中;

3. 经常组合查询的列放在一张表中;

   **垂直分库是指按照业务将表进行分类，分布到不同的数据库上面，每个库可以放在不同的服务器上，它的核心理念是专库专用。**

### （其实可以认为分表是单机版垂直水平切分，分库是将分出来的表独立成一个库。本质就是不在一个服务器上了）

读写分离(读从服务器，写主服务器)、主从复制:

使用前开启bin-log功能，日志文件用于记录数据库的读写增删

将一个数据库master复制到另一个slave数据库上。在Master和Slave之间实现整个主从复制的过程是由三个线程参与完成。

需要开启3个线程，master IO线程，slave开启 IO线程 SQL线程，
Slave 通过IO线程连接master，并且请求某个bin-log，position之后的内容。
MASTER服务器收到slave IO线程发来的日志请求信息，io线程去将bin-log内容，position返回给slave IO线程。
slave服务器收到bin-log日志内容，将bin-log日志内容写入relay-log中继日志，创建一个master.info的文件，该文件记录了master ip 用户名 密码 master bin-log名称，bin-log position。
slave端开启SQL线程，实时监控relay-log日志内容是否有更新，解析文件中的SQL语句，在slave数据库中去执行。

(总的来说就是，slave启动一个IO线程去读取master的binlog，master发现有人请求，则启动IO线程将文件发送给slave，之后slave启动sql线程relay-log(中继日志)，如果更新了执行里面的sql语句)

#### 三范式3NF：

**第一范式：保证每列的原子性(每列不可再分)**

第一范式是最基本的范式。如果**数据库表中的所有字段值都是不可分解的原子值**，就说明该数据库满足了第一范式。

**第二范式：所有非主键列依赖主键**

这是通俗的说法，用第二范式的定义描述第二范式，说的是在满足第一范式的基础上，数据库表中不存在非关键字段对任一候选关键字段的部分函数依赖，也即**所有非关键字段都完全依赖于任一组候选关键字**。

**第三范式----保证每列都和主键直接相关，不存在传递依赖**

第三范式又和第二范式相关，用第三范式的定义描述第三范式就是，数据库表中如果不存在非关键字段任一候选关键字段的传递函数依赖则符合第三范式，所谓传递函数依赖指的是如果存在"A-->B-->C"的决定关系，则C传递函数依赖于A。也就是说表中的字段和主键直接对应不依靠其他中间字段，说白了就是，决定某字段值的必须是主键。


#### mysql优化：

1.不是使用子查询(使用关联查询)。底层会创建临时表，且该临时表没有索引，最后还得删除临时表，这样浪费太多时间了。

2.开启慢查询，优先优化那些查询慢的语句。

3.使用explain查看索引使用情况，表连接情况，扫描行数等。

4.表结构使用：是使用范式或者反范式结构。类型的选择

5.索引的优化，正确的建立合适的索引。

6.SQL语句的优化。

#### 主要索引优化：

作用：1.减少扫描量2.帮组避免排序和创建临时表3.使随机IO，变成顺序IO。

三星索引：1.相关记录在一起2.排序和查找顺序相同3.索引中列包含查询所有列。

1.索引字段建立在where之后。 索引个数不是越多越好，最好不要超过6个。索引应该建立在不经常变化的字段。

2.不要使用select *，可以用覆盖索引代替。

3.like的%加右边，左边容易失效。

4.最左前缀规则，假如abc，只能a,ab,abc，不能单独两列。

5.使用不等于（！= 或者<>）或者is null 或者 is not null 不能使用索引

6.对索引不要进行计算，不要对索引使用or。容易失效。



### sql语句题：

**复制表结构( 只复制结构, 源表名：a新表名：b)** 

1.create table   b   like a;

2.create table b select * from a  where limit 0 ;

**复制整个表**

create table b select * from a;

**拷贝表数据 ( 拷贝数据, 源表名：a目标表名：b)** 

 insert into b  select  *  from a; 

**三表连接**

select * from a left join  b on a.id=b.id left join c a.id=c.id;

##### 将一个表两列数据连接起来在一列

select concat (id, name, score) as info from tt2;

使用concat方法实现字符串拼接。默认是直接连接，可以加空格或逗号分开

select concat (id, ' ',name,',', score) as info from tt2;

##### mysql插入数据

insert into表示插入数据，数据库会检查主键（PrimaryKey），如果出现重复会报错；

replace into表示插入替换数据，需求表中有PrimaryKey，或者unique索引的话，如果数据库已经存在数据，则用新数据替换，如果没有数据效果则和insert into一样；（）

insert ignore  忽略数据库中已经存在 的数据，如果数据库没有数据，就插入新的数据



隔离级别的实现：

为什么需要隔离级别？在并发环境下，事务的隔离性很难保证，因此会出现很多并发一致性问题。

1.丢失修改:事物A修改数据，事物B修改数据覆盖A的修改。

2.脏读:事务A读到事务B未提交数据。

3.不可重复读:T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同。可重复读是更改表中行级数据.

4.幻读:强调的是两次读到的数据的数量不一致。幻读是增加/减少表中行级数据

解决办法：

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| -------- | ---- | ---------- | ------ |
| 未提交读 | √    | √          | √      |
| 提交读   | ×    | √          | √      |
| 可重复读 | ×    | ×          | √      |
| 可串行化 | ×    | ×          | ×      |

那么这些隔离级别是怎么实现的？

两种方式：**悲观锁和乐观锁**

## 悲观锁：

在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现。往往依靠数据库提供的锁机制，读取数据时给加锁，其他事务无法改动这些数据。改动删除数据时也要加锁，其他事务无法读取这些数据。依靠读写锁来完成这些隔离级别。

##### 读写锁

- 排它锁（Exclusive），简写为 X 锁，又称写锁。(对于数据只能完成当前事务的操作，对于其他带锁的事务都不能进行操作，不带锁的select可以)
- 共享锁（Shared），简写为 S 锁，又称读锁。(其实就是给数据加了只能读锁，对于所有事务只读都是允许，不能更改)

有以下两个规定：

- 一个事务对数据对象 A 加了 X 锁，就可以对 A 进行读取和更新。加锁期间其它事务不能对 A 加任何锁。
- 一个事务对数据对象 A 加了 S 锁，可以对 A 进行读取操作，但是不能进行更新操作。加锁期间其它事务能对 A 加 S 锁，但是不能加 X 锁。

锁的兼容关系如下：

| -     | X    | S    |
| ----- | ---- | ---- |
| **X** | ×    | ×    |
| **S** | ×    | √    |

在mysql中，delete，update，insert都会默认加排它锁，而select是默认不加锁。所以对于加了X或S锁，都是可以进行select的(无锁)。

如果想对select进行加锁操作可以添加字段

排他锁：select ...for update语句

共享锁：select ... lock in share mode语句



通过三级封锁协议来达到目的：

**一级封锁协议**

事务 T 要修改数据 A 时必须加 X 锁，直到 T 结束才释放锁。

**可以解决丢失修改问题**，因为不能同时有两个事务对同一个数据进行修改，那么事务的修改就不会被覆盖。

**二级封锁协议**

在一级的基础上，要求读取数据 A 时必须加 S 锁，读取完马上释放 S 锁。(写数据时不能读数据)

**可以解决读脏数据问题**，因为如果一个事务在对数据 A 进行修改，根据 1 级封锁协议，会加 X 锁，那么就不能再加 S 锁了，也就是不会读入数据。

**三级封锁协议**

在二级的基础上，要求读取数据 A 时必须加 S 锁，直到事务结束了才能释放 S 锁。(表示读数据时，不能加入x锁，所以读数据时就不能修改数据了)

**可以解决不可重复读的问题**，因为读 A 时，其它事务不能对 A 加 X 锁，从而避免了在读的期间数据发生改变。

## 乐观锁：

相对悲观锁而言，乐观锁机制採取了更加宽松的加锁机制。悲观锁大多数情况下依靠数据库的锁机制实现，以保证操作最大程度的独占性。但随之而来的就是数据库性能的大量开销。特别是对长事务而言，这种开销往往无法承受。

而乐观锁机制在一定程度上攻克了这个问题。乐观锁，大多是基于数据版本号（ Version ）记录机制实现。何谓数据版本号？即为数据添加一个版本号标识，在基于数据库表的版本号解决方式中，通常是通过为数据库表添加一个 “version” 字段来实现。

即MVCC：用于实现提交读和可重复读这两种隔离级别。本身并不能解决幻读，而是通过next-key解决幻读(行级锁+间隙锁)。

CAS等

#### MVCC：早期只有读读之间可以并发，读写，写读，写写都要阻塞。引入多版本之后，只有写写之间相互阻塞，其他三种操作都可以并行。

MVCC只在 Read Committed 和 Repeatable Read两个隔离级别下工作。其他两个隔离级别和MVCC不兼容，Read Uncommitted总是读取最新的记录行，而不是符合当前事务版本的记录行；Serializable 则会对所有读取的记录行都加锁。

(读写，读读，写写：这里指的是事务操作，而不是读锁，写锁)



#### 乐观锁实现方式：

- 取出记录时，获取当前version
- 更新时，带上这个version
- 执行更新时， set version = newVersion where version = oldVersion
- 如果version不对，就更新失败

乐观锁：1. 先查询，获得版本号 version = 1

-- A线程update user set name = "codewei",version = version +1where id =2 and version = 1

-- B线程抢先完成，这个时候 version=2，会导致A修改失败！

update user set name = "codewei" ,version = version+1where id =2 and version = 1





### 磁盘中的结构：

所有数据存放在data目录下:

库名/包含表结构文件，表结构包含.frm+索引结构.ibd(innodb) 或者.myd.myi(myISAM)

数据并不是和表结构放在一起，InnoDB 类型的表数据统一存放于 data 目录下的 **ibdata1** 文件中

（因为innodb是共享表空间的，所以存在一起。如果不想共享可以使用独享空间，只有独享空间会生成.ibd文件。）

（实际上ibata1是存放innodb索引结构的地方，但是由于innodb默认是聚簇索引

​	ibd是表空间的数据文件，以段区页方式规划存储数据行和索引)

1. 聚簇索引并不是一种单独的索引类型，而是一种数据数据存储方式；

2. 当表有聚簇索引是，它的数据行实际上存在放在索引的叶子页(leaf page)中；

3. 叶子页包含了行的全部数据；

   **所以在mysql中数据始终和索引放在一块，并不存在什么光存数据的地方。即使没有索引，也还是可以，别忘了聚簇索引说的：没有主键也能随机取或帮我们创建**



##### 聚簇索引：id(主键)作为B+树的下标，用于IO深度搜索，值：整个行数据。

##### 非聚簇索引：选择指定的索引作为下标，值：主键。查询流程：先查非聚簇索引拿到主键id，然后根据主键id，拿到行数据。(两次查询)

##### 组合索引：多个索引，取(选择度高)第一个作为下标，值：主键id。(所以才会出现需要按顺序取索引，如果是第二索引则不走索引了)

##### 覆盖索引：其实就是说，索引中已经包含了需要的字段，比如：这里组合索引，就默认覆盖了id。（减少回表的手段）

##### 回表：根据查询出主键id，在根据id去查询主表。(回表的次数也可以用来衡量性能的好坏，回表越多，性能越差)建议覆盖索引

==(在未使用组合索引，使用两个单索引时，只会走一个索引，然后根据id找到相应的记录，去比较另一个字段的值)==

##### 索引下推：5.6及之前的版本，组合索引只会生效第一个索引，然后通过取id，拿主表数据判断后面的索引条件。有了索引下推，可以直接将索引条件作用在组合索引上。

比如：select * from a name="1" and age>18;组合索引(name,age)

5.6以前组合索引只会生效第一个索引，找出所有name=1的，然后根据id去查所有数据，并判断age>18。

5.6以后，找出所有name=1的，然后判断age>18，在回表返回所有数据

==(由于组合索引只生效第一个)==

### Mysql如何实现ACID特性：

**原子性**：利用回滚的机制完成恢复到未执行事务之前。通过**Undo log**日志实现恢复。为了满足原子性，在做任何操作之前，首先将数据备份到undo log中。然后才开始事务，完成对数据操作。如果进行回滚，则将undo log日志恢复到之前的状态。

**持久性**：执行完事务操作，数据保存在数据库中。那他怎么保证事务操作后，数据一定保存在数据库。使用redolog。

**重做日志** redo log 分为两部分：一部分是内存中的重做日志缓冲（redo log buffer），是易丢失的；二部分是重做日志文件（redo log file），是持久的。InnoDB通过Force Log at Commit机制来实现持久性，当commit时，必须先将事务的所有日志写到重做日志文件进行持久化，待commit操作完成才算完成。

(将事务操作过程记录在缓冲中，待提交commit时，将缓冲写入到redo file中进行持久化)

**隔离性：**锁，MVCC等

**一致性：**undo log+redo log保证事务的一致性。(其他三个特性就是来保证一致性的，保证了其他的三个性质就保证了一致性)

#### redo是物理逻辑日志，记录的是页的物理修改(提交前记录数据的修改，整个数据)操作。undo是逻辑日志，根据每行记录进行记录(记录每一步步骤，好回滚)。



**InnoDB 中的锁**

InnoDB 的标准实现的锁只有 2 类，一种是行级锁，一种是意向锁。

共享锁（读锁 S Lock），允许事务读一行数据。

排它锁（写锁 X Lock），允许事务删除一行数据或者更新一行数据。

意向共享锁（读锁 IS Lock），事务想要获取一张表的几行数据的共享锁，事务在给一个数据行加共享锁前必须先取得该表的 IS 锁。

意向排他锁（写锁 IX Lock），事务想要获取一张表中几行数据的排它锁，事务在给一个数据行加排它锁前必须先取得该表的 IX 锁。

InnoDB 有 3 种行锁的算法：

- Record Lock：单个行记录上的锁。(即共享锁和排它锁)

  select * from t_logs where id = 1 for update 其中增删改操作自动上行锁，相当于上了写锁（排它锁）。

  select * from t_logs where id = 1 lock in share mode 相当于上了读锁（共享锁）。

- Gap Lock：间隙锁，锁定一个范围，而非记录本身。update t_permission set description = '666' where id > 0 and id < 6;

- Next-Key Lock：结合 Gap Lock 和 Record Lock，锁定一个范围，并且锁定记录本身。主要解决的问题是 RR 隔离级别下的幻读。

#### 隔离级别：

**Serializable(串行化)**：强制事务排序，串行化执行事务。

**Repeatable Read(可重复读)**：解决不可重复读。啥叫不可重复读，即在一个事务里，读了两次数据，但是两次数据都不一致。可重复读则是保证可以重复读，而数据不会出错。利用MVCC进行版本控制。

MVCC：维持两个版本号，创建版本号和删除版本号(还有个系统版本号，可以认为是最近操作的事务版本号)

1.insert:插入一条新数据，创建版本号=原系统版本号+1（即当前事务的版本号）；

2.delete:删除一条数据，删除版本号=原系统版本号+1（即当前事务的版本号）；

3.select:查询只能查询出<=当前事务版本号的数，和删除版本>当前事务版本号或删除版本不存在的(即未删除).

4.update:开始就在想，如果第一个事务开始查询数据版本号为5，如果update了，此时版本号为6了，那么怎么可能select到呢？当然从开始这样想就错了。其实自己就没仔细考虑update这个动作，update如果是在原有基础上直接更改版本号，那么就会导致该数据被当前查询不到，显然不是这样有漏洞。

update正确做法：1.首先删除数据,删除版本号更新(这个并不影响上个事务查询)。2.插入一条新数据(更新为当前版本号)。所以此时数据库是有两条数据的，可能会在事务结束时才去删除带有删除版本号的数据。

**Read Committed(读已提交)**：解决脏读：避免读取到别人未提交的数据。mvcc默认解决这个问题，未提交数据版本号未更新。

**Read Uncommitted(读未提交)**：事务中的修改，即使没有提交，对其他事务也都是可见的。



in和exists区别:

exists是对外表做loop循环，每次loop循环再对内表（子查询）进行查询，那么因为对内表的查询使用的索引（内表效率高，故可用大表），而外表有多大都需要遍历，不可避免（尽量用小表），故内表大的使用exists，可加快效率.当 exists( 查询 )中的查询存在结果时则返回真， 否则返回假。

in是把外表和内表做hash连接，先查询内表，再把内表结果与外表匹配，对外表使用索引（外表效率高，可用大表），而内表多大都需要查询，不可避免，故外表大的使用in，可加快效率.s

#### (总结:则子查询表大的用exists，子查询表小的用in.

#### in是在内存里遍历比较，而exists需要查询数据库，所以当B表数据量较大时，exists效率优于in。)

in的优化：

sql:表连接真的有效，使用join代替in。记住结果集使用 (别名.*) ,如果全部显示，速度很慢。
(说白了还是显示时只显示有用数据。)

java代码:比较好的两个方法:1.使用线程池，多线程处理list集合。2.lambda表达式流的处理(本质也是多线程)



#### 1.添加索引:(格式:alter table name add 索引方法名(列名))

**1.添加PRIMARY KEY（主键索引）** 

mysql>ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` ) 

**2.添加UNIQUE(唯一索引)** 
mysql>ALTER TABLE `table_name` ADD UNIQUE ( `column` ) 

**3.添加INDEX(普通索引)** 

mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column` ) 

**4.添加FULLTEXT(全文索引)** 

mysql>ALTER TABLE `table_name` ADD FULLTEXT ( `column`)  

**5.添加多列索引** 
mysql>ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )



建立索引原则:(索引建立需要查询的字段)

1.保证唯一性。(如果不是唯一的，也可以建立普通索引，但是基本没啥效果。像之前的查询的username就是重复的)

2.限制索引数目，不能过多。

3.最左前缀，a，ab，abc(头到尾，中间不能断)

4.索引不能进行计算。

5.经常查询的字段，需要排序，分组的字段建立索引。

6.选择度尽量大。去重字段/总数据 ，唯一索引选择度是1。

(总结:使用explain观察语句时，发现没有使用索引，可能是本身就没有建立索引。此时，建立索引时就需要考虑索引的原则了)

sql执行顺序:

1、 FROM：对 FROM 子句中的前两个表执行笛卡尔积(交叉联接)，生成虚拟表 VT1。
2、 ON：对 VT1 应用 ON 筛选器，只有那些使为真才被插入到 TV2。(因为表连接当然得先查询条件啊)
3、 OUTER (JOIN):如果指定了 OUTER JOIN(相对于 CROSS JOIN 或 INNER JOIN)，保留表中未找到匹配的行将作为外部行添加到 VT2，生成 TV3。如果 FROM子句包含两个以上的表，则对上一个联接生成的结果表和下一个表重复执行步骤 1 到步骤 3，直到处理完所有的表位置。
4、 WHERE：对 TV3 应用 WHERE 筛选器，只有使为 true 的行才插入 TV4。
5、 GROUP BY：按 GROUP BY 子句中的列列表对 TV4 中的行进行分组，生成 TV5。
6、 CUTE|ROLLUP：把超组插入 VT5，生成 VT6。
7、 HAVING：对 VT6 应用 HAVING 筛选器，只有使为 true 的组插入到 VT7。
8、 SELECT：处理 SELECT 列表，产生 VT8。
9、 DISTINCT：将重复的行从 VT8 中删除，产品 VT9。
10、 ORDER BY：将 VT9 中的行按 ORDER BY 子句中的列列表顺序，生成一个游标(VC10)。
11、 TOP：从 VC10 的开始处选择指定数量或比例的行，生成表 TV11，并返回给调用者。

(on和where区别，on是表连接条件。where就是最后结果的筛选)



#### 今天在使用Navicat for mysql设计表时，在设置外键的时候，删除时和更新时两列有四个值可以选择：CASCADE、NO ACTION、RESTRICT、SET NULL，自己全亲自试了一遍，它们的区别如下：

##### (主表:其中存在主键(primary key)用于与其它表相关联，并且作为在主表中的唯一性标识)

##### (从表：以主表的主键(primary key)值为外键 (Foreign Key)的表，可以通过外键与主表进行关联查询)

CASCADE：父表delete、update的时候，子表会delete、update掉关联记录;(所以当删除主表时，从表也删除)
SET NULL：父表delete、update的时候，子表会将关联记录的外键字段所在列设为null，所以注意在设计子表时外键不能设为not null；
RESTRICT：如果想要删除父表的记录时，而在子表中有关联该父表的记录，则不允许删除父表中的记录；
NO ACTION：同 RESTRICT，也是首先先检查外键；





# SQL调优

### 查询sql操作次数

```
show  status  like'Com_______' //当前会话连接下次数
show global status  like'Com_______'  //全局，自从数据库开启
Com_binlog,0
Com_commit,0
Com_delete,0
Com_insert,0
Com_repair,0
Com_revoke,0
Com_select,36
Com_signal,0
Com_update,0
Com_xa_end,0

show status like 'Innodb_rows_%'; //查询innodb引擎的操作次数

Innodb_rows_deleted,2
Innodb_rows_inserted,1024
Innodb_rows_read,1701
Innodb_rows_updated,3

```

### 定位低效率执行SQL

1.**开始慢查询日志**，记录下查询慢的sql。用—log-slow-queries[=ﬁle_name]选项启动时，mysqld 写一个包含所有执行时间超过 long_query_time 秒的 SQL 语句的日志文件。

2.**show processlist** ：可以使用show processlist命令查看当前MySQL在进行的线程，包括线程的状态、是否锁表等，可以实时地查看 SQL 的执行情况，同时对一些锁表操作进行优化

- id列，用户登录mysql时，系统分配的”connection_id”，可以使用函数connection_id()查看
- user列，显示当前用户。如果不是root，这个命令就只显示用户权限范围的sql语句
- host列，显示这个语句是从哪个ip的哪个端口上发的，可以用来跟踪出现问题语句的用户
- db列，显示这个进程目前连接的是哪个数据库
- command列，显示当前连接的执行的命令，一般取值为休眠（sleep），查询（query），连接
  （connect）等
- **time列**，显示这个状态持续的时间，单位是秒
- **state列**，显示使用当前连接的sql语句的状态，很重要的列。state描述的是语句执行中的某一个状态。一
  个sql语句，以查询为例，可能需要经过copying to tmp table、sorting result、sending data等状态才可以完成
- info列，显示这个sql语句，是判断问题语句的一个重要依据



### 分析慢查询SQL

#### 1.使用explain 语句

| id            | select查询的序列号，是一组数字，表示的是查询中执行select子句或者是操作表的顺序。 |
| ------------- | ------------------------------------------------------------ |
| select_type   | 表示 SELECT 的类型，常见的取值有 SIMPLE（简单表，即不使用表连接或者子查询）、PRIMARY（主查询，即外层的查询）、UNION（UNION 中的第二个或者后面的查询语句）、SUBQUERY（子查询中的第一个 SELECT）等 |
| table         | 输出结果集的表                                               |
| type          | 表示表的连接类型，性能由好到差的连接类型为( system —-> const ——-> eq_ref ———> ref———-> ref_or_null——> index_merge —-> index_subquery ——-> range ——-> index ———> all ) |
| possible_keys | 表示查询时，可能使用的索引                                   |
| key           | 表示实际使用的索引                                           |
| key_len       | 索引字段的长度                                               |
| rows          | 扫描行的数量                                                 |
| extra         | 执行情况的说明和描述                                         |

- **id 相同，**加载表的顺序是从上到下，即：A->B。id 不同id值越大，优先级越高。比如三次子查询，最内部的优先级越高。

  ![image-20210324094616198](../../../Library/Application Support/typora-user-images/image-20210324094616198.png)

- **select_type**

| **select_type** | **含义**                                                     |
| --------------- | ------------------------------------------------------------ |
| SIMPLE          | 简单的select查询，查询中不包含子查询或者UNION                |
| PRIMARY         | **查询中若包含任何复杂的子查询，最外层查询标记为该标识**     |
| SUBQUERY        | 在SELECT 或 WHERE 列表中包含了子查询                         |
| DERIVED         | 在FROM 列表中包含的子查询，被标记为 DERIVED（衍生） MYSQL会递归执行这些子查询，把结果放在临时表中 |
| UNION           | 若第二个SELECT出现在UNION之后，则标记为UNION ； 若UNION包含在FROM子句的子查询中，外层SELECT将被标记为 ： DERIVED |
| UNION RESULT    | 从UNION表获取结果的SELECT                                    |

- **type**

| NULL     | MySQL不访问任何表，索引，直接返回结果                        |
| -------- | ------------------------------------------------------------ |
| system   | 表只有一行记录(等于系统表)，这是`const`类型的特例，一般不会出现 |
| `const`  | 表示通过索引一次就找到了，`const`用于比较primary key 或者 unique 索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL 就能将该查询转换为一个常亮。`const`于将**”主键” 或 “唯一”** 索引的所有部分与常量值进行比较。(只在主键索引和唯一索引可能看到) |
| `eq_ref` | 类似ref，区别在于使用的是唯一索引，使用主键的关联查询，关联查询出的记录只有一条。常见于主键或唯一索引扫描 |
| ref      | 非唯一性索引扫描，返回匹配某个单独值的所有行。本质上也是一种索引访问，返回所有匹配某个单独值的所有行（多个） |
| range    | 只检索给定返回的行，使用一个索引来选择行。 where 之后出现 between ， < , > , in 等操作。 |
| index    | index 与 ALL的区别为 index 类型只是遍历了索引树， 通常比ALL 快， ALL 是遍历数据文件。 |
| all      | 将遍历全表以找到匹配的行                                     |

- **extra**

| `using ﬁlesort`   | 说明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取， 称为“文件排序”, 效率低。 |
| ----------------- | ------------------------------------------------------------ |
| using temporary   | 使用了临时表保存中间结果，MySQL在对查询结果排序时使用临时表。常见于 order by 和group by； 效率低 |
| using index       | 表示相应的select操作使用了覆盖索引， 避免访问表的数据行， 效率不错。索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据;当能通过读取索引就可以得到想要的数据，那就不需要读取行了。 |
| Using where       | 表明使用了where过滤                                          |
| using join buffer | 使用了连接缓存：                                             |
| impossible where  | where子句的值总是false，不能用来获取任何元组                 |



### 2.show profile分析SQL



1. ```
   select @@have_profiling;    //查询是否支持profile
   set profiling = 1;					//设置session级别，开启profie
   select @@profiling;					//查看profile是否开启，1为开启
   
   //执行完目标sql然后执行 
   show profiles 
   show profile for query 5  //queryId，这条sql操作有问题，还需查询使用方法
   ```

![image-20210324100738931](../../../Library/Application Support/typora-user-images/image-20210324100738931.png)



### 3.trace分析优化器执行计划

1.打开trace设置格式为 JSON，并设置trace最大能够使用的内存大小，避免解析过程中因为默认内存过小而不能够完整展示。

```
SET  optimizer_trace="enabled=on",end_markers_in_json=on;
set  optimizer_trace_max_mem_size=1000000;
```

```
select * from  information_schema.optimizer_trace; //查看
```





# 索引



### 1.添加索引

```
create index idx_seller_name_sta_addr on tb_seller (name, status, address); //多个就是组合索引，也可以单个索引
//注意添加索引，时间会很长
```



## 2.避免索引失效

#### 1.最左前缀匹配

如果索引了多列，要遵守最左前缀法则。指的是查询从索引的最左前列开始，并且不跳过中间的列，

如果跳过中间列，那么只会使用最左列的索引。

（可以ABC：走三个索引,AB：走两个索引,AC：走一个索引。<font color='red'>(全值匹配时，即使顺序不一样，sql优化器会自动优化顺序)</font>>）

#### 2.全值匹配

全值匹配，**对索引中的所有列都指定具体值。**

该情况下，索引生效，执行效率高。<font color='red'>(全值匹配时，即使顺序不一样，sql优化器会自动优化顺序)</font>>

#### 3.避免索引失效—范围查询右边的列，不能使用索引

> 范围查询右边的列将不走索引，例如`status`使用了范围查询，那么只走 `name` 和 `status` 的索引，不走 `address` 的索引。

```
explain select * from tb_seller where name = '小米科技' and status > '1' and address = '西安市';
```

(三个索引，有一个使用了范围索引，则他的右边列不能使用索引)

#### 4.避免索引失效—不要在索引列上进行运算操作

如果在索引列上进行运算操作，那么索引将失效

```
explain select * from tb_seller where substring(name,3,2) = '科技';
```

#### 5.避免索引失效—字符串必须加单引号(其实说白了就是保持存的和查的类型一致)

字符串不加单引号，造成索引失效。

```
explain select * from tb_seller where name = '小米科技' and status = '1';
explain select * from tb_seller where name = '小米科技' and status = 1; //没单引号，其实说白了就是保持存的和查的类型一致
```

#### 6.尽量使用覆盖索引，避免select *

> 尽量使用覆盖索引（只访问索引的查询（索引列完全包含查询列）），减少select *
>
> 如果查询列，超出索引列，也会降低性能。（例如多查询了一个password会导致性能下降）
>
> TIPS:(在explain的extra可以看到)

- using index ：使用覆盖索引的时候就会出现。(select的列在索引列内)
- using where：在查找使用索引的情况下，需要回表去查询所需的数据
- using index condition：查找使用了索引，但是需要回表查询数据
- using index ; using where：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据

#### 7.or分隔的条件

or分割开的条件， 如果**or前的条件中的列有索引**，而**后面的列中没有索引**，那么**涉及的索引都不会被用到**。

(尽量不使用or)

```
explain select    from tb_seller where name='黑马程序员' or createtime = '2088-01-01 12:00:00';
```

#### 8.以%开头的Like模糊查询，索引失效。

如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

```
explain select * from tb_seller where name like '%黑马';//失效

explain select * from tb_seller where name like '黑马%'; //有效
```

解决办法：

使用覆盖索引

```
explain select name,status,address from tb_seller where name like '%黑马%'; //使用了覆盖索引就可以检索到索引
```

#### 9.is NULL、is NOT NULL `有时`索引失效。

```
explain select * from tb_seller where address is null;//走索引
explain select * from tb_seller where address is not null;//不走索引
```

走不走索引，取决于数据库自己的判断。如果数据为null的数据少，则走索引。null的数据多，则不走索引。

总之取决于数据，走不走索引总是取决于，数据为null的是不是少。如果为null的很少，数据库走索引，因为这个时候不为null的很多，不然会查询很多数据。

#### 10.In、not In `有时` 不使用索引

```
explain select * from tb_seller where sellerid in ('oppo','xiaomi','sina');
```

**In、not in 使不使用索引不是一刀切的。**

#### 负向查询

==包含：!=、<>、not in、not like、!>、!<等，都不走索引。==

```
SELECT * FROM student WHERE age != 7;
```

#### 索引字段函数操作和运算符操作，索引失效

```
SELECT * FROM student WHERE left(name,1) = '小';
SELECT * FROM student WHERE age+1 = 7;
```

普通索引的不等于不会走索引

\- !=

select * from tb1 where email != 'alex'

==--特别的：如果是主键，则还是会走索引==

select * from tb1 where nid != 123

#### 11.单列索引和复合索引

尽量使用复合索引，而少使用单列索引。==如果创建单索引，只会走一个索引==。

```
create index idx_name_sta_address on tb_seller(name, status, address);

相当于创建了三个索引：

name
name + status
name + status + address
```

## 优化

#### 1.GROUP BY如何优化(默认根据后面字段排序，1.排序字段为索引类型 2.禁止排序)

不知道你有没有使用关键字`EXPLAIN`去查看`GROUP BY`操作的执行计划，你会发现在`EXTRA`字段中出现类似`filesort`的关键字。这是因为默认情况下，MySQL对所有GROUP BY col1，col2….的字段进行排序，类似在查询中指定 ORDER BY col1，col2…一样。因此，`GROUP BY`是默认排序的。

因此，我们可以让`GROUP BY`后的字段利用索引排序，或者你的业务场景不需要排序的情况下，可以使用以下语句禁用默认排序：

```
SELECT age,count(*) 
  FROM student 
GROUP BY age 
ORDER BY NULL;
```

## 2.ORDER BY如何优化

orderBy排序，如果能利用好索引的排序，也能提升很多性能。

因此，MySQL 可以使用一个索引来满足`ORDER BY`子句，而不需要额外的排序。但需要遵守以下三个原则：

- WHERE 条件和 ORDER BY 使用相同的索引。
- ORDER BY 字段的顺序和索引顺序一致。
- ORDER BY 的字段都是升序或者都是降序。

```
SELECT * 
  FROM student 
WHERE age = 7
ORDER BY age ASC,name ASC;
```

### 3.分页性能优化

深分页的时候，MYSQL查询几秒钟的情况，你遇到过吗？不知道MYSQL在分页时处于何种考虑，`LIMIT n,m`,这个操作跳过n条数据需要进行回表，导致我们下面这个SQL需要回表10万次。

(查询age=10的索引，拿到的只是id，需要进行回表操作)

```
SELECT * FROM student where age = 10 LIMIT 100000,10
```

办法总是有的，可换种思路避免这10万次回表，来看SQL的优化吧：

```
SELECT *
  FROM student s1
INNER JOIN(
  SELECT id FROM student where age = 10 LIMIT 100000,10
) s2 on s1.id = s2.id ;
```

这里相当于利用了索引上的id值，然后用in进行查询。(则不会回表查询)

### 4.JOIN性能优化

OIN也是多表关联的常用的关键字，有`LEFT JOIN`、`RIGHT JOIN`、`JOIN`等。在了解JOIN性能优化前，需要明确：`驱动表`和`被驱动表`。

- LEFT JOIN
  左表是驱动表，右表是被驱动表
- RIGHT JOIN
  右表时驱动表，左表是被驱动表
- INNER JOIN
  MYSQL会选择数据量比较小的表作为驱动表，大表作为被驱动表（==小表驱动大表==）

我们用这个SQL来观察三种算法的执行过程：

```
SELECT t1.*,t2.* 
  FROM table1 t1 
  LEFT JOIN table2 t2 on t1.a=t2.a;
```

> 假设：table1有100行数据，table2有1000行数据。

1.假如a在两张表都存在索引(因为都走索引，t1的每找到一条数据，带着索引去t2表寻找(一次就找到了)，然后两个数据组成一条数据)

总扫描行数=包括遍历t1表的100次(==遍历次数==)和t2嵌套查询idx_a索引(==查找索引顺序==)的100次=200

==(所以都有索引的时候，取决于小表的数量，决定查询次数)==

2.如果都没有索引(mysql并没有采用笛卡尔积)

总扫描=t1的100次遍历+100*1000的循环遍历=扫描次数为100100次

3.如果都没有索引(分块嵌套查询连接)

1.把表t1的数据读入线程内存join_buffer中

2.扫描表t2，把表t2中的每一行取出来，跟join_buffer中的数据做对比，满足join条件的，作为结果集的一部分返回。

因此，尽量比对次数是10万次，但表扫描次数为1100次，是table1和table2的数据总行数。

(因为线程内存join_buffer，不可能无限大，所以也是会分其他小块)

那么，最后我们总结下优化Join的手段有：

- 将小表作为驱动表
  无论是否使用索引，小表作为驱动表都能够减少扫描次数。
- 调整join_buffer_size大小
  MYSQL该参数的默认值大小为512k,调整该参数的大小，可以减少分块嵌套查询的块数，能够成倍的减少扫描次数。
- 关联时使用索引
  关联时使用索引避免扫描和笛卡尔判断，是提升join性能的绝对杀手锏！

# 查看索引使用情况

#### 1.查看当前连接（会话）的索引情况

```
show status like 'Handler_read%';
show global status like 'Handler_read%'; //查看全局
```

<img src="../../../Desktop/TyporaBlogMAC/图/image-20210324160237145.png" alt="image-20210324160237145" style="zoom:50%;" />

- Handler_read_first：索引中第一条被读的次数。如果较高，表示服务器正执行大量全索引扫描（这个值越低
  越好）。
- Handler_read_key：如果索引正在工作，这个值代表一个行被索引值读的次数，如果值越低，表示索引得到的
  性能改善不高，因为索引不经常使用（这个值越高越好）。
- Handler_read_next ：按照键顺序读下一行的请求数。如果你用范围约束或如果执行索引扫描来查询索引列，
  该值增加。
- Handler_read_prev：按照键顺序读前一行的请求数。该读方法主要用于优化ORDER BY … DESC。
- Handler_read_rnd ：根据固定位置读一行的请求数。如果你正执行大量查询并需要对结果进行排序该值较高。你可能使用了大量需要MySQL扫描整个表的查询或你的连接没有正确使用键。这个值较高，意味着运行效率低，应
  该建立索引来补救。
- Handler_read_rnd_next：在数据文件中读下一行的请求数。如果你正进行大量的表扫描，该值较高。通常说明你的表索引不正确或写入的查询没有利用索引。



# 回表

插入几条测试数据。

```
INSERT INTO xttblog(id, k, name) VALUES(1, 2, 'xttblog'),
    (2, 1, '业余草'),
    (3, 3, '业余草公众号');
```

假设，现在我们要查询出 id 为 2 的数据。那么执行 select * from xttblog where ID = 2; 这条 SQL 语句就不需要回表。原因是根据主键的查询方式，则只需要搜索 ID 这棵 B+ 树。主键是唯一的，根据这个唯一的索引，MySQL 就能确定搜索的记录。

但当我们使用 k 这个索引来查询 k = 2 的记录时就要用到回表。select * from xttblog where k = 2; 原因是通过 k 这个普通索引查询方式，则需要先搜索 k 索引树，然后得到主键 ID 的值为 1，再到 ID 索引树搜索一次。这个过程虽然用了索引，但实际上底层进行了两次索引查询，这个过程就称为回表。

(回表：先查普通索引，得到主键索引，在查主键索引。(所以其实其他普通索引最后也还是回到主键索引))

```
[
    {
        "code":"ip_geo_equal_cost_city",
        "name":"IP归属城市和订单起点城市不一致",
        "relatedFeatures":[
            "ip_geo_equal_cost_city"
        ],
        "expression":"return ip_geo_equal_cost_city "
    },
{
        "code":"phone_geo_same_area",
        "name":"手机归属和订单启始地不一致",
        "relatedFeatures":[
            "phone_geo_same_area"
        ],
        "expression":"return phone_geo_same_area"
    },
{
        "code":"userCallMoveSpeed",
        "name":"用户叫车移动速度超过20米/秒",
        "relatedFeatures":[
            "userCallMoveSpeed"
        ],
        "expression":"userCallMoveSpeed > 20"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    },
{
        "code":"z_1_day_call_order_city_num",
        "name":"用户1天叫车关联城市数大于5",
        "relatedFeatures":[
            "z_1_day_call_order_city_num"
        ],
        "expression":"z_1_day_call_order_city_num > 5"
    }

]

{"context":{"estimate":3234,"customerNo":937218008,"afterDiscountEstimate":3234},"customerMobile":"13036669088","customerNo":937218008,"filterId":"2","getOffTime":1647438802000,"orderNo":303260185513008}
1647438802000



{
    "2":{
        "filterScript":"bizCode =~[1,13] && orderTypeCode =~ [1,2,3,4] && originEnumCode =~ [12] && costCity =~[\"0871\",\"020\",\"028\",\"0898\",\"023\"] && customerNo % 100 < 100",
        "modelTemplate":"risk_applet_alert_model_script",
        "hitModel":0,
        "msgTemplate":"risk-jstorm-SMS-NOTICE-stop-bill-alert"
    },
    "1":{
        "filterScript":"bizCode =~[1,13] && orderTypeCode =~ [1,2,3,4] && originEnumCode =~ [9] && costCity =~[\"0871\",\"020\",\"028\",\"0898\",\"023\"] && customerNo % 100 < 50 && customerNo % 10 < 5",
        "modelTemplate":"risk_applet_alert_model_script",
        "hitModel":0,
        "msgTemplate":"risk-jstorm-SMS-NOTICE-alipay-stop-bill-alert"
    },
    "3":{
        "filterScript":"bizCode =~[1,13] && orderTypeCode =~ [1,2,3,4] && originEnumCode =~ [9] && costCity =~[\"0871\",\"020\",\"028\",\"0898\",\"023\"] && customerNo % 100 < 50 && customerNo % 10 > 4",
        "modelTemplate":"risk_applet_alert_model_script",
        "hitModel":1,
        "msgTemplate":"risk-jstorm-SMS-NOTICE-alipay-stop-bill-alert"
    }
}


```

