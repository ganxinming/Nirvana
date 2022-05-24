# mongo

mogodb 是一种 nosql 数据库， 不同于关系型数据库。 它以 BJON 格式进行存储数据， 类似于json。 用coolection替代了mysql中的table， document替代了row， filed替代了column。
正如书中所讲， mongodb 是面向文档的数据库。 用一条记录能很好的解决层级问题， 这是mysql 不容易做到的。 而且其设计理念非常符合面向对象的思想， 对于程序员来说非常容易入门。 且数据存储是基于内存的， 从而达到高度读写。 但是它不支持事务， 其实也可想而知，
如果基于事务了， 他的速度可能就没那么快了。 所以 mongodb 也主要是以文档形式做缓存辅助 mysql。

1.key可以使用大多数utf-8的字符。

2.documnet就一个，而collection可以多个。

3.插入的键值对是默认有序的。

#### 为什么使用？

对于关系型数据库，如果某个表关联东西太多了，导致需要建立特多的表。然后查询时，会造成频繁的join，减少效率。比如一篇博客，有多个标签，多条评论等。就这样需要建立两个表。而使用mongo，一个解决。

```
_id: POST_ID
   title: TITLE_OF_POST, 
   description: POST_DESCRIPTION,
   a 	uthor: POST_BY,
   tags: [TAG1, TAG2, TAG3],
   likes: TOTAL_LIKES, 
   comments: [	
      {
         user:'COMMENT_BY',
         message: TEXT,
         dateCreated: DATE_TIME,
      },
      {
         user:'COMMENT_BY',
         message: TEXT,
         dateCreated: DATE_TIME,
      }
   ]
```

优点：数据存储形式非常灵活宽松。而且key值每个document可以不同，即有的有，有的没有。JSON存数据，不需要转换。

- 1. 不同key-value需要用逗号隔开，而key:value中间是用冒号；
- 2. 如果一个key有多个value，value要用[]。哪怕当前只有一个value，也加上[]以备后续的添加；
- 3. 整个“数据块”要用{}括起来；



## java使用mongo

1.创建过程

```
ServerAddress serverAddress = new ServerAddress("192.168.248.128", 27017);
List<MongoCredential> credentialsList = new ArrayList<MongoCredential>();
MongoCredential mc = MongoCredential.createScramSha1Credential("rwuser","sang","123".toCharArray());
credentialsList.add(mc);
MongoClientOptions options = MongoClientOptions.builder()
        //设置连接超时时间为10s
        .connectTimeout(1000*10)
        //设置最长等待时间为10s
        .maxWaitTime(1000*10)
        .build();
//创建客户端
MongoClient client = new MongoClient(serverAddress,credentialsList,options);
//获得数据库
MongoDatabase sang = client.getDatabase("sang");
//获得表名
c = sang.getCollection("c1");
```

### 1.查询find()

1.简单的查询使用Filters就足够了。(当然里面还有很多其他的过滤条件，也很强大)

```
collection.find(Filters.eq("_id","12312"));
```

复杂的查询，使用BasicDBObject,其实很容易发现如果是过滤某个固定的值直接put(key,value)，而对于大于，小于，范围之类的过滤，需要put(key,new BasicDBObject())。注意类似范围查询，使用append()追加条件。

```
MongoCollection<Document> collection = mongoUtil.getCollection(driverCollectionName);
BasicDBObject queryObject = new BasicDBObject();
queryObject.put("datatime",new BasicDBObject("$gt",dateStart).append("lt",dateEnd));
queryObject.put("driverNo",new BasicDBObject("$ne",0));
queryObject.put("driverStatus",new BasicDBObject("$in",driverServingStatues()));
queryObject.put("driverType",1);
FindIterable<Document> documents = collection.find(queryObject);
```

#### 上面的仅仅可以进行查询，但不能分组聚合查询，aggregate可以进行分组。

```
 AggregateIterable<Document> aggregateIterable = collection.aggregate(
                    Arrays.asList(Aggregates.match(basicDBObject),
							Aggregates.group("$driverNo"),
							Aggregates.sort(new BasicDBObject("count",-1))));
或者(下面这种方式比较全面)
BasicDBObject queryCond = new BasicDBObject();
 
  queryCond.put("patient_id", new BasicDBObject("$in", patientTransferDepRepository.getPatientIdsByDep(depId)));
 Date startDate = null;
        Date endDate = null;
        if (month == null) {
            //统计当前年份 全部数据
            startDate = DateUtils.getYearFirst(year);
            endDate = DateUtils.getYearLast(year);
        } else {
            //统计当前 年-月 数据
            //手术时间在当年当月第一天和最后一天时间范围内(工具类month从0开始，此处month是从1开始，所以month-1)
            startDate = DateUtils.getFirstDayOfMonth(year, month-1);
            endDate = DateUtils.getLastDayOfMonth(year, month-1);
        }
        queryCond.put("survey_time", new BasicDBObject("$gte", DateUtils.parseDate(DateUtils.formatDate(startDate)+ " 00:00:00")).append("$lte", DateUtils.parseDate(DateUtils.formatDate(endDate)+ " 23:59:59")));
        aggregateCondList.add(new BasicDBObject("$match", queryCond));
 
        //unwind
        aggregateCondList.add(new BasicDBObject("$unwind", "$data"));
        // ZDLB 改为 手术名称的code(SSMC)
        aggregateCondList.add(new BasicDBObject("$match", new BasicDBObject("data.key", "SSMC")));
 
        //聚合条件 取出该年该月 手术名称：相应次数
        aggregateCondList.add(new BasicDBObject("$group", new BasicDBObject("_id", "$data.value").append("count" , new BasicDBObject("$sum", 1))));
 
        //排序，取手术名称对应次数前8的
        aggregateCondList.add(new BasicDBObject("$sort", new BasicDBObject("count", -1)));
        aggregateCondList.add(new BasicDBObject("$limit", 8));
 
        Map<String, Integer> resultMap = new LinkedHashMap<>(8);
        //遍历结果集
        MongoCursor<Document> cursor = mdcMongoTemplate.getCollection(CDR_DATA).aggregate(aggregateCondList).iterator();
 
```

常用管道：

- $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
- $limit：用来限制MongoDB聚合管道返回的文档数。
- $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- $group：将集合中的文档分组，可用于统计结果。
- $sort：将输入文档排序后输出。

### 2.增加(insertOne,insertMany)

```
Document d1 = new Document();
d1.append("name", "三国演义").append("author", "罗贯中");
c.insertOne(d1);

List<Document> collections = new ArrayList<Document>();
Document d1 = new Document();
d1.append("name", "三国演义").append("author", "罗贯中");
collections.add(d1);
Document d2 = new Document();
d2.append("name", "红楼梦").append("author", "曹雪芹");
collections.add(d2);
c.insertMany(collections);
```

### 3.修改$set(没有这个key，就创建，有就覆盖),$inc(只适用用数字型，在原有基础上增加)

```
c.updateOne(Filters.eq("author", "罗贯中"), new Document("$set", new Document("name", "三国演义123")));
```

### 4.删除

```
c.deleteOne(Filters.eq("author", "罗贯中"));
c.deleteMany(Filters.eq("author", "罗贯中"));
```

db.getCollection('scalping_order').find({'hitRule':'R28','useTime':{$gt:1610553600000}''}





# 注意

1.所有date类型的数据，存入mongo就变成了Long类型时间戳，取出来的时候也是Long，需要自己转换。

2.找找有没有能直接转换mongo的document和java对象的方法，一个个转太恶心了。



# 命令

## 索引

查询索引:db.collection.getIndexes(),
创建索引:db.collection.createIndex(),

```
db.collection.createIndex({name:1}) #1从小到大排序，-1从大到小
创建复合索引
db.collection.createIndex({name:1,balance:-1}) #从小到大排序
创建唯一索引，即字段不能重复
db.collection.createIndex({name:1},{unique:true}) #从小到大排序
db.containers.createIndex({name: 1},{unique:true, background: true})
Mongo提供两种建索引的方式foreground和background。
前台操作，它会阻塞用户对数据的读写操作直到index构建完毕；
后台模式，不阻塞数据读写操作，独立的后台线程异步构建索引，此时仍然允许对数据的读写操作。（一般选这个）

创建生命周期索引，即到期自动删除
db.collection.createIndex({crawl_time:1},{expireAfterSeconds:20,background: true}) 
创建文本索引，如果经常对某个字段进行字符串的搜索，为了提高速度可以使用该类型索引，可以对一个集合的一个或者多个字段设置文本索引，但是一个集合只能有一个文本索引。
db.collection.createIndex({ name: "text")
db.stores.createIndex( { name: "text", description: "text" } )
```

删除索引:db.collection.dropIndex()

```
db.stores.dropIndex("name_text")
```

## 查询(万事万物皆json)

```
db.COLLECTION_NAME.find().sort({KEY:1}) //按某字段排序
```

## 删除数据(未指定参数，删除所有)

```
db.test.remove({'title': 'MongoDB'})
db.collection.deleteMany ({})
```



# 慢查

```
db.getProfilingLevel()：命令来获取当前的Profile级别
db.setProfilingLevel(2) ：设置慢查级别
db.setProfilingLevel( 1 , 1000 )：设置级别同时设置默认超过毫秒为慢查

Profiling一共分为3个级别：
0 - 不开启。
1 - 记录慢命令 (默认为>100ms)
2 - 记录所有命令

开启profiling功能后，系统会把相关命令详细信息记录到当前数据库的system.profile集合里。查询方法也是跟普通的集合查询一样。
db.system.profile.find().pretty()

结构如下：
{
        "op" : "command",
        "ns" : "earthshake.driver_privatework_work_order",
        "command" : {
                "aggregate" : "driver_privatework_work_order",
                "pipeline" : [
                        {
                                "$match" : {
                                        "relativeViolationTrips.driverNo" : NumberLong("5481899511"),
                                        "relativeViolationTrips.beginTime" : {
                                                "$gte" : NumberLong("1626278400000"),
                                                "$lte" : NumberLong("1626364799999")
                                        }
                                }
                        },
                        {
                                "$unwind" : {
                                        "path" : "$relativeViolationTrips"
                                }
                        },
                        {
                                "$project" : {
                                        "relativeViolationTrips" : 1
                                }
                        }
                        {
                                "$limit" : 2147483647
                        },
                        {
                                "$skip" : 0
                        }
                ],
                "cursor" : {

                }
        },
        "keysExamined" : 686581,
        "docsExamined" : 433744,
        "cursorExhausted" : true,
        "numYield" : 5369,
        "locks" : {
                "Global" : {
                        "acquireCount" : {
                                "r" : NumberLong(10746)
                        }
                },
                "Database" : {
                        "acquireCount" : {
                                "r" : NumberLong(5373)
                        }
                },
                "Collection" : {
                        "acquireCount" : {
                                "r" : NumberLong(5372)
                        }
                }
        },
        "nreturned" : 1,
        "responseLength" : 950,
        "protocol" : "op_query",
        "millis" : 1939,
        "planSummary" : "IXSCAN { relativeViolationTrips.beginTime: -1, relativeViolationTrips.driverNo: 1 }",
        "ts" : ISODate("2021-07-26T07:25:14.618Z"),
        "client" : "172.17.200.10",
        "allUsers" : [
                {
                        "user" : "caocao",
                        "db" : "earthshake"
                }
        ],
        "user" : "caocao@earthshake"
}

op：操作类型
ns：被查的集合
commond：命令的内容
docsExamined：扫描文档数
keysExamined：扫描了多少次索引

nreturned：返回记录数
millis：耗时时间，单位毫秒
ts：命令执行时间
responseLength：返回内容长度

常用命令：说白就是针对profile里面的字段进行筛选
查询超过某一个耗时的慢查语句：
db.system.profile.find({millis:{$gt:50}})
查询最新的前几条耗时语句：
db.system.profile.find().sort({$natural:-1}).limit(3)
查询某个collection的慢查：
 db.system.profile.find({ns:'earthshake.order_current'}).limit(3)
 
```

## 慢查优化点：

1.docsExamined(扫描的记录数)远大于nreturned(返回结果的记录数)的话，那么我们就要考虑通过加索引来优化记录定位了。 　

2.responseLength 如果过大，那么说明我们返回的结果集太大了，这时请查看find函数的第二个参数是否只写上了你需要的属性名。(类似 于MySQL中不要总是select)

### Profiler 的效率

Profiling 功能肯定是会影响效率的，但是不太严重，原因是他使用的是system.profile 来记录，而system.profile 是一个capped collection 这种collection 在操作上有一些限制和特点，但是效率更高。



## explain分析执行计划

 explain的三种模式

2.1 queryPlanner

不会真正的执行查询，只是分析查询，选出winning plan。

2.2 executionStats

返回winning plan的关键数据。

executionTimeMillis该query查询的总体时间。

2.3 allPlansExecution

执行所有的plans。

```
db.getCollection('student').find({numIndex:{"$gt":50,"$lt":55}}).explain("executionStats")
```

执行后分析如下：

```
{        "queryPlanner" : {                "plannerVersion" : 1,
                "namespace" : "earthshake.driver_privatework_work_order",
                "indexFilterSet" : false,
                "parsedQuery" : {
                        "$and" : [
                                {
                                        "relativeViolationTrips.driverNo" : {
                                                "$eq" : NumberLong("5065809356")
                                        }
                                },
                                {
                                        "relativeViolationTrips.beginTime" : {
                                                "$lte" : NumberLong("1633622399999")
                                        }
                                },
                                {
                                        "relativeViolationTrips.beginTime" : {
                                                "$gte" : NumberLong("1633536000000")
                                        }
                                }
                        ]
                },
                "winningPlan" : {
                        "stage" : "FETCH",
                        "filter" : {
                                "$and" : [
                                        {
                                                "relativeViolationTrips.driverNo" : {
                                                        "$eq" : NumberLong("5065809356")
                                                }
                                        },
                                        {
                                                "relativeViolationTrips.beginTime" : {
                                                        "$lte" : NumberLong("1633622399999")
                                                }
                                        }
                                ]
                        },
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "keyPattern" : {
                                        "relativeViolationTrips.beginTime" : -1,
                                        "relativeViolationTrips.driverNo" : 1
                                },
                                "indexName" : "relativeViolationTrips.beginTime_-1_relativeViolationTrips.driverNo_1",
                                "isMultiKey" : true,
                                "multiKeyPaths" : {
                                        "relativeViolationTrips.beginTime" : [
                                                "relativeViolationTrips"
                                        ],
                                        "relativeViolationTrips.driverNo" : [
                                                "relativeViolationTrips"
                                        ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 1,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "relativeViolationTrips.beginTime" : [
                                                "[inf.0, 1633536000000]"
                                        ],
                                        "relativeViolationTrips.driverNo" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                }
                        }
                },
                "rejectedPlans" : [
                        {
                                "stage" : "FETCH",
                                "filter" : {
                                        "$and" : [
                                                {
                                                        "relativeViolationTrips.driverNo" : {
                                                                "$eq" : NumberLong("5065809356")
                                                        }
                                                },
                                                {
                                                        "relativeViolationTrips.beginTime" : {
                                                                "$gte" : NumberLong("1633536000000")
                                                        }
                                                }
                                        ]
                                },
                                "inputStage" : {
                                        "stage" : "IXSCAN",
                                        "keyPattern" : {
                                                "relativeViolationTrips.beginTime" : -1,
                                                "relativeViolationTrips.driverNo" : 1
                                        },
                                        "indexName" : "relativeViolationTrips.beginTime_-1_relativeViolationTrips.driverNo_1",
                                        "isMultiKey" : true,
                                        "multiKeyPaths" : {
                                                "relativeViolationTrips.beginTime" : [
                                                        "relativeViolationTrips"
                                                ],
                                                "relativeViolationTrips.driverNo" : [
                                                        "relativeViolationTrips"
                                                ]
                                        },
                                        "isUnique" : false,
                                        "isSparse" : false,
                                        "isPartial" : false,
                                        "indexVersion" : 1,
                                        "direction" : "forward",
                                        "indexBounds" : {
                                                "relativeViolationTrips.beginTime" : [
                                                        "[1633622399999, -inf.0]"
                                                ],
                                                "relativeViolationTrips.driverNo" : [
                                                        "[MinKey, MaxKey]"
                                                ]
                                        }
                                }
                        }
                ]
        },
        "executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 1,
                "executionTimeMillis" : 6,
                "totalKeysExamined" : 896,
                "totalDocsExamined" : 765,
                "executionStages" : {
                        "stage" : "FETCH",
                        "filter" : {
                                "$and" : [
                                        {
                                                "relativeViolationTrips.driverNo" : {
                                                        "$eq" : NumberLong("5065809356")
                                                }
                                        },
                                        {
                                                "relativeViolationTrips.beginTime" : {
                                                        "$lte" : NumberLong("1633622399999")
                                                }
                                        }
                                ]
                        },
                        "nReturned" : 1,
                        "executionTimeMillisEstimate" : 11,
                        "works" : 898,
                        "advanced" : 1,
                        "needTime" : 895,
                        "needYield" : 0,
                        "saveState" : 14,
                        "restoreState" : 14,
                        "isEOF" : 1,
                        "invalidates" : 0,
                        "docsExamined" : 765,
                        "alreadyHasObj" : 0,
                        "inputStage" : {
                                "stage" : "IXSCAN",
                                "nReturned" : 765,
                                "executionTimeMillisEstimate" : 0,
                                "works" : 897,
                                "advanced" : 765,
                                "needTime" : 131,
                                "needYield" : 0,
                                "saveState" : 14,
                                "restoreState" : 14,
                                "isEOF" : 1,
                                "invalidates" : 0,
                                "keyPattern" : {
                                        "relativeViolationTrips.beginTime" : -1,
                                        "relativeViolationTrips.driverNo" : 1
                                },
                                "indexName" : "relativeViolationTrips.beginTime_-1_relativeViolationTrips.driverNo_1",
                                "isMultiKey" : true,
                                "multiKeyPaths" : {
                                        "relativeViolationTrips.beginTime" : [
                                                "relativeViolationTrips"
                                        ],
                                        "relativeViolationTrips.driverNo" : [
                                                "relativeViolationTrips"
                                        ]
                                },
                                "isUnique" : false,
                                "isSparse" : false,
                                "isPartial" : false,
                                "indexVersion" : 1,
                                "direction" : "forward",
                                "indexBounds" : {
                                        "relativeViolationTrips.beginTime" : [
                                                "[inf.0, 1633536000000]"
                                        ],
                                        "relativeViolationTrips.driverNo" : [
                                                "[MinKey, MaxKey]"
                                        ]
                                },
                                "keysExamined" : 896,
                                "seeks" : 1,
                                "dupsTested" : 896,
                                "dupsDropped" : 131,
                                "seenInvalidated" : 0
                        }
                }
        },
        "serverInfo" : {
                "host" : "mongo-risk01",
                "port" : 15389,
                "version" : "3.4.18",
                "gitVersion" : "4410706bef6463369ea2f42399e9843903b31923"
        },
        "ok" : 1
}
```

#### queryPlanner

namespace：该值返回的是该query所查询的表
indexfilter：是否使用了索引过滤(index filter)
winningPlan：查询优化器针对该query所返回的最优执行计划的详细内容
winningPlan.stage：最优执行计划的stage
winningPlan.inputStage：explain.queryPlanner.winningPlan.stage的child stage
winningPlan.inputStage.keyPattern：所扫描的index内容，此处是w:1与n:1
winningPlan.inputStage.indexName：winning plan所选用的index
winningPlan.inputStage.isMultiKey：是否是Multikey，此处返回是false，如果索引建立在array上，此处将是true
winningPlan.inputStage.direction：此query的查询顺序，此处是forward，如果用了.sort({w:-1})将显示backward
winningPlan.inputStage.indexBounds：winningplan所扫描的索引范围。
rejectedPlans：其他执行计划（非最优而被查询优化器reject的）的详细返回，其中具体信息与winningPlan的返回中意义相同，故不在此赘述。

#### Stage 分类

COLLSCAN：扫描整个集合 

IXSCAN：索引扫描 

FETCH：根据索引去检索选择document
SHARD_MERGE：将各个分片返回数据进行merge
SORT：表明在内存中进行了排序（与老版本的scanAndOrder:true一致）
LIMIT：使用limit限制返回数
SKIP：使用skip进行跳过 IDHACK：针对_id进行查
SHARDING_FILTER：通过mongos对分片数据进行查询
COUNT：利用db.coll.explain().count()之类进行count
COUNTSCAN：count不使用用Index进行count时的stage返回
COUNT_SCAN：count使用了Index进行count时的stage返回 SUBPLA：未使用到索引的$or查询的stage返回
TEXT：使用全文索引进行查询时候的stage返回 PROJECTION：限定返回字段时候stage的返回

#### executionStats

executionSuccess：是否执行成功
nReturned：查询的返回条数
executionTimeMillis：整体执行时间
totalKeysExamined：扫描索引条目的数量
totalDocsExamined：扫描文档的数量
executionStages.nReturned：意义与nReturned一样
executionStages.executionTimeMillisEstimate：意义与executionTimeMillis一样
executionStages.docsExamined：意义与totalDocsExamined一样
executionStages.executionStats.inputStage中：的与上述理解方式相同



## mongo聚合管道



![image-20211011140203849](../../../Library/Application Support/typora-user-images/image-20211011140203849.png)

| 管道操作符 | Description                                           |
| :--------- | :---------------------------------------------------- |
| $project   | 增加、删除、重命名字段                                |
| $match     | 条件匹配。只满足条件的文档才能进入下 一阶段           |
| $limit     | 限制结果的数量                                        |
| $skip      | 跳过文档的数量                                        |
| $sort      | 条件排序。                                            |
| $group     | 条件组合结果 统计                                     |
| $lookup    | $lookup 操作符 用以引入其它集合的数 据 （表关联查询） |

**SQL 和 NOSQL 对比:**

| WHERE    | $match   |
| :------- | :------- |
| GROUP BY | $group   |
| HAVING   | $match   |
| SELECT   | $project |
| ORDER BY | $sort    |
| LIMIT    | $limit   |
| SUM()    | $sum     |
| COUNT()  | $sum     |
| join     | $lookup  |

**管道表达式:**

管道操作符作为“键”,所对应的“值”叫做管道表达式。

例如{$match:{status:"A"}}，$match 称为管道操作符，而 status:"A"称为管道表达式， 是管道操作符的操作数(Operand)。

每个管道表达式是一个文档结构，它是由字段名、字段值、和一些表达式操作符组成的。

### 生产示例：

```
db.driver_privatework_work_order.aggregate([ { "$match" : { "relativeViolationTrips.beginTime" : { "$gte" : NumberLong("1633536000000"), "$lte" : NumberLong("1633622399999") }, "relativeViolationTrips.driverNo" : NumberLong("5065809356") } }, { "$unwind" : { "path" : "$relativeViolationTrips" } }, { "$project" : { "relativeViolationTrips" : 1 } }, { "$match" : { "relativeViolationTrips.beginTime" : { "$gte" : NumberLong("1633536000000"), "$lte" : NumberLong("1633622399999") }, "relativeViolationTrips.driverNo" : NumberLong("5065809356") } }, { "$limit" : 2147483647 }, { "$skip" : 0 } ])
```



## **$project**

修改文档的结构，可以用来重命名、增加或删除文档中的字段。
要求查找 order 只返回文档中 trade_no 和 all_price 字段

```
db.order.aggregate([ { $project:{ trade_no:1, all_price:1 } } ])
```

## **$match**

**作用**

用于过滤文档。用法类似于 find() 方法中的参数。

```
db.order.aggregate([ 

  { $project:{ trade_no:1, all_price:1 } }, 

  { $match:{"all_price":{$gte:90}} } ]

)
```

## **$group**

将集合中的文档进行分组，可用于统计结果。

统计每个订单的订单数量，按照订单号分组

```
db.order_item.aggregate( [ 

  { $group: {_id: "$order_id", total: {$sum: "$num"}} } 

] )
```

####  **$lookup 表关联**

```
db.order.aggregate([ 
    { 
        $lookup: { from: "order_item", localField: "order_id", foreignField: "order_id", as: "items" }
     }
])
```

### $unwind（数组分散进行统计）

在aggregate中，常常会遇到一些字段属性是数组对象，然后又需要对这些数组对象进行统计。
这时候就需要用到$unwind操作符。这是一个常用的，又容易被忽略的一个操作。

```
{
    user_id:A_id ,
    bonus:[
        { type:a ,amount:1000 },
        { type:b ,amount:2000 },
        { type:b ,amount:3000 }
    ]
}
```

**unwind操作：**

```
db.user.aggregate([
    {$unwind:bonus}
])

//结果
{user_id : A_id , bonus:{type : a ,amount : 1000}}
{user_id : A_id , bonus:{type : b ,amount : 2000}}
{user_id : A_id , bonus:{type : b ,amount : 3000}}

db.user.aggregate([
    {$match: {user_id : A_id} },
    {$unwind:bonus},
    {$project:{ bonus:1 }}
])

//结果
{type : a ,amount : 1000}
{type : b ,amount : 2000}
{type : b ,amount : 3000}

```

### mongo架构

1.主从复制

主节点负责写(也可以读)，从节点负责读(只能读)，没有故障转移，主节点挂了，从节点不会自动选举。

2.副本集

这个是官方现在推荐的一种架构，实现主从复制，灾备，故障自动转移。
该架构的特点是没有所谓的主库，也没有所谓的从库，任何一个节点都可以充当主库或者从库，类似车轮战吧，一个累崩了，换一另外一个上。(但还是会选出一个主节点来负责整个副本集的读写)

主节点机负责整个副本集的读写，副本集定期同步数据备份，一但主节点挂掉，副本节点就会选举一个新的主服务器，这一切对于应用服务器不需要关心。副本集中的副本节点在主节点挂掉后通过心跳机制检测到后，就会在集群内发起主节点的选举机制，自动选举一位新的主服务器。

invoke com.caocao.risk.terminal.identify.api.AppletSubscribeNoticeApi.subscribe({"orderNo":123456,"uid":7890000,"status":0,"msgType":1,"originEnumCode":12,"customerMobile":"15180626533","requestTime":1648863891259,"class":"com.caocao.risk.terminal.identify.param.SubscribeNoticeParam"})
