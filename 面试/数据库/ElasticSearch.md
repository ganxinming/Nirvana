# ElasticSearch

Elasticsearch 也是使用 Java 编写的，它的内部使用 Lucene 做索引与搜索，但是它的目的是使全文检索变得简单，**通过隐藏 Lucene 的复杂性，取而代之的提供一套简单一致的 RESTful API**。

(Luncene是信息检索工具包)

es是一个开源的高扩展分布式全文搜索引擎。近乎实时储存，检索数据。可扩展上百台服务器，处理PB(大数据)级别数据。

##### 目的：通过简单的RestFul API(/post/delete/put/get)操作，实现快速搜索。

==它被用作**全文检索**、**结构化搜索**、**分析**以及这三个功能的组合==

1.es不支持事务。

2.也没有oracle的用户的概念，es有安全控制是通过xpack实现，但是是针对访问全局的安全控制。

3.另外也不支持类似数据库中通过外键的复杂的多表关联操作。

除了搜索，结合Kibana、Logstash、Beats开源产品，Elastic Stack（简称ELK）还被广泛运用在大数据近实时分析领域，包括：**日志分析**、**指标监控**、**信息安全**等。它可以帮助你**探索海量结构化、非结构化数据，按需创建可视化报表，对监控数据设置报警阈值，通过使用机器学习，自动识别异常状况**。

### ElasticSearch的主要功能及应用场景

> 我们在哪些场景下可以使用ES呢？

- 主要功能：

1）海量数据的分布式存储以及集群管理，达到了服务与数据的高可用以及水平扩展；

2）近实时搜索，性能卓越。对结构化、全文、地理位置等类型数据的处理；

3）海量数据的近实时分析（聚合功能）

- 应用场景：

1）网站搜索、垂直搜索、代码搜索；

2）日志管理与分析、安全指标监控、应用性能监控、Web抓取舆情分析；

### 概述：

Es是面向文档。 一切都是json(仅支持json)

关系型数据库的对比

<img src="../../../Library/Application Support/typora-user-images/image-20220703231612192.png" alt="image-20220703231612192" style="zoom:25%;" />

| DB              | ES                        |
| --------------- | ------------------------- |
| 数据库 database | 索引(indices)             |
| 表 table        | Types（类型，过时）       |
| 行rows          | Documents(文档，一条数据) |
| 字段 columns    | Fields                    |

### 物理设计

##### es将每个索引划分多个分片，分片能在不同集群转移。一个人就是一个集群，默认的集群名称就是elasticSearch

> ### 类型

类型是文档的逻辑容器，像表格就是行的容器。指在一个索引中，可以索引不同类型的文档，如用户数据、博客数据。6.0.0 版本起已废弃，一个索引中只存放一类数据。

> ### 索引

就是数据库，索引是一个非常大的文档集合。将数据分散到不同分片，每个分片，有主分片和从分片(复制分片防止主分片宕机)，主从绝对不在同一节点。每个节点就是一个es进程。一个分片就是一个lucene索引，一个包含倒排索引的文件目录。所以不扫描全部文档，也能知道哪些数据包含关键字。

> ### 备份

- **Replication 备份**: 一个分片可以有多个备份（副本）

![image-20201111231046066](file:///Users/ganxinming/Desktop/TyporaBlogMAC/%E5%9B%BE/image-20201111231046066.png?lastModify=1646876925)

> ### 文档

一个文档包含对应的key和value对应的字段。可以是有层次，复杂的文档(json对象)，索引的基本信息单元，以JSON格式来表示。

# 倒排索引

这种结构适用于快速的全文搜索，一个索引由文档中所有不重复的列表构成，对于每一个词，都有一个包含它的文档列表。

==倒排索引，是通过分词策略，形成了词和文章的映射关系表，这种词典+映射表即为倒排索引。有了倒排索引，就能实现`o（1）时间复杂度`的效率检索文章了，极大的提高了检索效率。==

#### 添加索引：

普通方式：一般不会用（可以设置分片,副本等配置信息）

- **settings**: 用来设置分片,副本等配置信息

- mappings

  : 字段映射，类型等

  - **properties**: 由于type在后续版本中会被Deprecated, 所以无需被type嵌套

```
# PUT /索引名/类型名/文档id{请求体}
PUT /test2 (实际上一个put请求就可以，无需下面的mapping)
{ 
"settings": {
		"number_of_shards": 1,
		"number_of_replicas": 1
	},
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
       "cid": {
                    "index": "true",
                    "type": "long"
                },
      "customerMobile": {
                    "index": "true",
                    "type": "keyword"
                },
      "birthday": {
        "type": "date"
      }
    }
  }
}
//指定需要建立的倒排索引
```



## 索引模板

==索引模板是一种告诉Elasticsearch在创建索引时如何配置索引的方法。==

#### 模板类型

模板有两种类型：**索引模板**和**组件模板**。

1. **组件模板**是可重用的构建块，用于配置映射，设置和别名；它们不会直接应用于一组索引。
2. **索引模板**可以包含组件模板的集合，也可以直接指定设置，映射和别名。

#### 索引模板中的优先级

1. 可组合模板优先于旧模板。如果没有可组合模板匹配给定索引，则旧版模板可能仍匹配并被应用。
2. 如果使用显式设置创建索引并且该索引也与索引模板匹配，则创建索引请求中的设置将优先于索引模板及其组件模板中指定的设置。
3. 如果新数据流或索引与多个索引模板匹配，则使用优先级最高的索引模板。



#### 使用模板创建索引：es内部维护了template，template定义好了mapping，只要index的名称被template匹配到，那么该index的mapping就按照template中定义的mapping自动创建。而且template中定义了index的shard分片数量、replica副本数量等等属性。如果template编写错误重新执行put命令覆盖即可

（创建索引时进行模板匹配，1.满足index_patterns，2.多个模板时取order最小的优先级最高）

```
PUT _template/metric_datatest
 
{
    "index_patterns": ["metric_datatest-*"],
    "order": 0,//模板优先级，多个模板取最大的，有些版本字段是priority
   "settings": {
    "index": {
      "refresh_interval": "10s",//每10秒刷新
      "number_of_shards" : "5",//主分片数量
      "number_of_replicas" : "2",//副分片数量
      "translog": {
        "flush_threshold_size": "1gb",//内容容量到达1gb异步刷新
        "sync_interval": "30s",//间隔30s异步刷新（设置后无法更改）
        "durability": "async"//异步刷新
      }
    }
  },      
    "mappings": {
        "properties": {
            "ts": {
                "type": "long"
            },
            "name": {
                "type": "keyword"
            },
            "tags": {
                "type": "text",
                "analyzer": "tags_analyzer"
            },
            "count": {
                "type": "long"
            },
            "sum": {
                "type": "double"
            },
            "max": {
                "type": "double"
            },
            "min": {
                "type": "double"
            }
        }
    },
    "aliases": {
        "metric_datatest": {}
    }
}
```

### **索引别名**

  索引别名就是给一个索引或者多个索引起的另一个名字，索引别名是一个非常有用的功能，尤其在数据要重新导入时可以很优雅的切换数据为名为user_info的索引，创建别名user_info_alicas。

​	

### 索引类型：

```
字符串类型：text(可以被分词)、keyword(不可分词精确匹配)
数值类型：long、integer、short、byte、double、float、half float、scaled、float
日期类型：date
布尔值类型：boolean
二进制类型：binary
```



# 操作

##### 新建索引/插入数据：

http://115.159.202.204:9200/test1/type1/1 PUT请求（数据放在json里）

/新建索引名 test1/类型type1/文档id

类型默认_doc

##### 更新数据：

http://115.159.202.204:9200/test3/_doc/1  PUT请求

/索引名 test3/类型_doc/文档id  

##### 新版更新http://115.159.202.204:9200/test3/_doc/1/_update  POST请求（后面加个标识）

区别：PUT会将此次所提提交的数据全部覆盖原本的，而新版只修改提交修改的值。

##### 删除数据：

DELETE 请求

根据路径进行删除，如果是索引删除整个索引，如果某个文档删除文档

**查询数据：**

http://115.159.202.204:9200/test3/_doc/1  GET请求

**简单条件查询** http://115.159.202.204:9200/test3/_doc/_search?q=class:1234

_search?q=字段名:值

**复杂查询** 

GET请求带着请求体

```
{
    "query":{
        "match":{
            "name":"abc"
        }
    },
    "_source":["name"], //展示需要显示的字段，默认全部字段展示
    "sort":[           //按字段排序
      {
        "age" : "asc"
      }
    ],
    "from" : 0, //分页，跟limit参数类似
    "size" : 2
    
}
```

## bool query(布尔查询)

bool  通过布尔逻辑将较小的查询组合成较大的查询。

bool查询包含四种操作符，分别是must,should,must_not,filter。他们均是一种数组，数组里面是对应的判断条件。

- `must`： 多个查询条件的完全匹配,相当于 and 。 贡献算分
- `must_not`：过滤子句，必须不能匹配，相当于 not，但不贡献算分
- `should`： 选择性匹配，至少满足一条，相当于 or。贡献算分
- `filter`： 过滤子句，必须匹配，但不贡献算分

**filter**可以进行过滤(在query级别之下)

```
{
    "query":{
        "bool":{
          "must":[
              {
              "match":{
                  "name":"abc"
                 }
              },
              {
              "match":{
                  "age":18
                 }
              }
          ],
          "filter":{
            "range":{
              "age":{
                "gt":10,
                "lt":20
              }
            }
          }
        }
    }
}
```

#### 精确查询

- term(直接通过倒排索引指定的词条进行精确查找,精确匹配)，terms 跟 term 有点类似， 但 terms 允许指定多个匹配条件。 如果某个字段指定了多个
- match(使用分词器进行解析，利用倒排索引查)

(Keyword类型字段不会被分词器解析，换句话只能精确查，text：存储数据时候，会自动分词，并生成索引。)

#### 高亮查询 

相当于对搜索的高亮字段加了个html标签，当然也可以自定义高亮显示标签。

```
{
    "query":{
        "match":{
            "name":"abc"
        }
    },
    "highlight":{
      "fields":{
        "name":{}
      }
    }
}
```



# 2.ELK

##### elasticSearch，logstash，kibana。

搜集数据->清洗数据->分析展示







customerAllSuccessHermesOrderCount

customerAllSuccessHermesOrderCount

```
private Boolean is_success;
private Integer code;
private String msg;
private T data;
```





# 命令常用操作

http://服务器IP地址:9200/_cat/indices

下面直接是在kibana上操作命令，省略域名端口

```
1.GET /_cat/indices 查看索引表

2.GET /risk_event_request_result_20210818 查看某个表的详细信息(包括倒排索引)


1.插入数据数据
PUT /hxxy/user/1
{
  "name": "1号用户",
  "age": 23,
  "desc": "程序员",
  "tags": ["Java","Linux","Redis"]
}

# 根据id查询用户
GET hxxy/user/1
# 根据条件查询用户
# 所有用户得分（权重）相同
GET hxxy/user/_search?q=name:用户

2.复杂查询(后面增加_search)
# 普通根据字段查询
GET hxxy/user/_search
{
  "query": {
    "match": {
      "_id": "12332"
    }
  }
}

3.查看索引模板
GET /_template/risk_case_record*?pretty


```



# ES查询结果模板

```
{
  "took": 1, //查询耗时毫秒
  "timed_out": false,//是否超时
  "_shards": {//查询分片信息
    "total": 4,//总共4个
    "successful": 4,//成功4个
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,//文档数
    "max_score": null,
    "hits": [
      {
        "_index": "risk_event_request_result_20210818",//索引
        "_type": "risk_event_request",//表
        "_id": "348117892014841944",
        "_score": null,
        "_source": {//真正的数据
          "hitObserveRules": "R261009"
        },
        "sort": [
          "R261009"
        ]
      }
    ]
  }
}
```



# 查询命令模板

```
{
    "query":{
    //term使用精确查询，对于一般只查单个属性，term就够了
   		 "term":{
            "_id":"abc"
        },
        //多值匹配，能够匹配多个id
        "terms": {
          "_id": ["348117892014841944","348117892014841945"]
        }
        //match使用分词匹配
        "match":{
            "name":"abc"
        },
        //模糊匹配
        "wildcard": {
     			 "rule": "*R0001*"
   			 },
   			 //前缀匹配
   			 "prefix": {
      			"rule": "R00"
   			 }
    //一般用于多个条件组合异或非等，bool相当于最后等于true的才可以拿到
        "bool":{
        //里面的条件都是and
        	"must":[
        			{
        			"term":{
            			"name":"abc"
      			 		 }
        			},
        			{
        			"match":{
            			"age":18
      			 		 }
        			}
        	],
        	//过滤
        	"filter":{
        		"range":{
        			"age":{
        				"gt":10,
        				"lt":20
        			}
        		},
        		//或者精确匹配过滤
        		"term":{}
        	},
        	//里面的条件都是or
        	"should":{
        			"match":{
            			"name":"abc"
      			 		 }
        			}
        }
    },
    
    "_source":["name"], //展示需要显示的字段，默认全部字段展示
    "sort":[					 //按字段排序
    	{
    		"age" : "asc"
    	}
    ],
    "from" : 0, //分页，跟limit参数类似
    "size" : 2
    
}
```



# 总结下来就是：

1.query用于查询，sort排序，_source筛选字段，from，size分页，他们都是同等级

2.query中简单查询使用term|match即可，复杂查询使用bool，bool中可以带与或非条件和过滤器。

3.bool结构

```
{
   "bool" : {
      "must" :     [],
      "should" :   [],
      "must_not" : [],
      "filter":    []
   }
}
```