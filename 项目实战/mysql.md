

##### 场景：msql默认字符集utfmb3,以三个字符为一个字，但是这有个坏处，对于表情无法插入，需要修改成umb4

```
1.查询当前数据库的字符集
show full columns from penalty_order;
2.表设置成utfmb4
ALTER TABLE  penalty.penalty_order convert to character set  utf8mb4;
3.如果这样的话，还是不行，因为其实数据库本身的连接还是utfumb3,还需要设计连接时，指定umb4，以下是druid连接池的配置：
<property name="connectionInitSqls" value="set names utf8mb4;"/>

或者不需要第三步只需要将数据库的：
character_set_client、character_set_connection、character_set_results
这三个参数的编码设置为utf8mb4即可；

执行以下sql：
set character_set_client='utf8mb4';
set character_set_connection = 'utf8mb4';
set character_set_results = 'utf8mb4';
show variables like 'character%';
```