# ElasticSearch

基于lucene做了一些封装和增强。

(Luncene是信息检索工具包)

es是一个开源的高扩展分布式全文搜索引擎。近乎实时储存，检索数据。可扩展上百台服务器，处理PB(大数据)级别数据。

目的：通过简单的RestFul API(/post/delete/put/get)操作，实现快速搜索。



### 概述：

Es是面向文档。 一切都是json(仅支持json)

关系型数据库的对比

| DB              | ES                        |
| --------------- | ------------------------- |
| 数据库 database | 索引(indices)             |
| 表 table        | Types（类型，过时）       |
| 行rows          | Documents(文档，一条数据) |
| 字段 columns    | Fields                    |

### 物理设计

##### es将每个索引划分多个分片，分片能在不同集群转移。一个人就是一个集群，默认的集群名称就是elasticSearch

> ### 类型

类型是文档的逻辑容器，想表格就是行的容器。

> ### 索引

就是数据库，索引是一个非常大的文档集合。将数据分散到不同分片，每个分片，有主分片和从分片，主从绝对不在同一节点。每个节点就是一个es进程。一个分片就是一个lucene索引，一个包含倒排索引的文件目录。所以不扫描全部文档，也能知道哪些数据包含关键字。

![image-20201111231046066](../../../Desktop/TyporaBlogMAC/图/image-20201111231046066.png)

> ### 文档

一个文档包含对应的key和value对应的字段。可以是有层次，复杂的文档(json对象)



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
    "sort":[					 //按字段排序
    	{
    		"age" : "asc"
    	}
    ],
    "from" : 0, //分页，跟limit参数类似
    "size" : 2
    
}
```

**must**相当于and，多个条件查询。**should**相当于or。**must_not**不是某个值相当于not.

**filter**可以进行过滤

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

- term(精确匹配)
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