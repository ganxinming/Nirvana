# Hbase

大数据做的三件事：海量数据的传输+计算+存储(Hbase,Big Table)

实现了HDFS随机写操作，在几十亿数据中实现秒级增删改查。

#### 1.什么是HBSE

HBase的原型是Google的BigTable论文，受到了该论文思想的启发，目前作为Hadoop的子项目来开发维护，用于支持结构化的数据存储。

HBase是一个分布式，可扩展，支持海量数据存储的Nosql数据库。

HBase的目标是存储并处理大型的数据，更具体来说是仅需使用普通的硬件配置，就能够处理由成千上万的行和列所组成的大型数据。

#### 2.HBase特点

**1****）海量存储**

Hbase适合存储PB级别的海量数据，在PB级别的数据以及采用廉价PC存储的情况下，能在几十到百毫秒内返回数据。这与Hbase的极易扩展性息息相关。正式因为Hbase良好的扩展性，才为海量数据的存储提供了便利。

**2****）列式存储**

这里的列式存储其实说的是列族存储，Hbase是根据列族来存储数据的。列族下面可以有非常多的列，列族在创建表的时候就必须指定。

(通过rowkey，就是直接取出一列族下的所有列)

**3****）极易扩展**

Hbase的扩展性主要体现在两个方面，一个是基于上层处理能力（RegionServer）的扩展，一个是基于存储的扩展（HDFS）。
通过横向添加RegionSever的机器，进行水平扩展，提升Hbase上层的处理能力，提升Hbsae服务更多Region的能力。

备注：RegionServer的作用是管理region、承接业务的访问，这个后面会详细的介绍通过横向添加Datanode的机器，进行存储层扩容，提升Hbase的数据存储能力和提升后端存储的读写能力。

**4****）高并发**

由于目前大部分使用Hbase的架构，都是采用的廉价PC，因此单个IO的延迟其实并不小，一般在几十到上百ms之间。这里说的高并发，主要是在并发的情况下，Hbase的单个IO延迟下降并不多。能获得高并发、低延迟的服务。

**5****）稀疏**

稀疏主要是针对Hbase列的灵活性，在列族中，你可以指定任意多的列，在列数据为空的情况下，是不会占用存储空间的。



#### 3.HBase数据模型

逻辑上和关系型数据很像，一张表中有行有列，但从底层来看是K-V结构，更像是一个多维的map。

#### 逻辑结构：

rowkey(行键)：建表时必须创建rowkey，唯一的，相当于主键。(键值按字典排序)

column-family(列族)：将多个列放在一组，形成列族。不同的列族放在不同文件夹。

Region(切片)：表的横向切分。根据rowkey的数据量进行划分。

store：被列族和切片，分成不同的区域。是真正的储存内容，存在HDFS

![image-20201102225214502](../../../Desktop/TyporaBlogMAC/图/image-20201102225214502.png)

高表行多(水平切分)，宽表列多(垂直切分)

#### 物理结构：

对应过来，相当于每列值都对应一行数据。

TimeStamp ： 时间戳。用于储存操作的时间。(特别重要，影响增删改查)

Type：操作类型。put：插入，修改。delete：删除。get：查

Column Qualifier：相当于列名。

storeFile：就是将这些数据放在文件中存储的。

cell：(图中没有标注，由唯一{rowkey,columnFamily,columnQualifier,TimeStamp}来确定唯一的单元) 需要靠时间戳来确定版本

最小的储存结构 cell中的数据没有类型，都是==字节码byte[]==。

(更改就是put一条新的数据，取时间戳最大的。删除就是新增一条delete操作，时间戳最大就是删除。有点像MVCC)

TTL：每条数据都有一个过期时间。

多版本：可以维护最近N次记录。

![image-20201102230044027](../../../Desktop/TyporaBlogMAC/图/image-20201102230044027.png)



#### Hbase(简化版架构)

master：管理表的增删改查。(只管表，需要zookeeper帮忙管理数组) DDL

zookeeper：管理数据的增删改查。(master挂了依然可以使用) DML

RegionServer：对region进行管理，提供对数据的增删改查。支持region分开与合并

(天然高可用性，不用配置什么slave，谁挂了选举什么的。直接全是master，谁抢到了资源就是谁的，但是对于资源的更新需要通知zk去通知其他master更新)

![image-20201103194142118](../../../Desktop/TyporaBlogMAC/图/image-20201103194142118.png)



# shell命令

1.查询所有表：list 

2.扫描：scan ”表名“

3.建表:  create "rtdw:risk_payAccount_related_customer", "cf" 



create "rtdw:risk_first_login", "cf" 

4.插入数据

 put '表名' ,'行键' ,'列族:列名' , '值'

 put 'rtdw:uic_customer_exchange_history_test' ,'475smdid_123456_10000' ,'cf:customer_no' , '999'

 put 'rtdw:uic_customer_exchange_history_test' ,'478deviceId_123456_10000' ,'cf:customer_no' , '999'

 put 'rtdw:uic_customer_exchange_history_test' ,'a89imei_123456_10000' ,'cf:customer_no' , '999'

 put 'rtdw:uic_customer_exchange_history_test' ,'2afcustomerMobile_123456_10000' ,'cf:customer_no' , '999'



## MySQL 和 HBase对比？

MySQL解决应用的在线事务问题，侧重于在线应用。

Hbase解决大数据场景的海量存储问题，侧重于大数据用户画像等。

引擎数据结构不一样：**B+Tree 和 LSM** 

