# String

(底层是可动态变化容量，每次插入数据时，如果小于1M，容量翻倍，)

字符串键是 Redis 最基本的键值对类型，这种类型的键值对会在数据库里面把单独的一个键和单独的一个值关联起来，被关联的键和值既可以是普通的文字数据，也可以是图片、视频、音频、压缩文件等更为复杂的二进制数据。

作为例子，图 2-1 展示了数据库视角下的四个字符串键，其中：

- 与键 `"message"` 相关联的值是 `"hello world"` ；
- 与键 `"number"` 相关联的值是 `"10086"` ；
- 与键 `"homepage"` 相关联的值是 `"redis.io"` ；
- 与键 `"redis-logo.jpg"` 相关联的值是二进制数据 `"\xff\xd8\xff\xe0\x00\x10JFIF\x00…"` 。

### Redis构建了一个叫做简单动态字符串（Simple Dynamic String），简称SDS

#### SDS 代码结构

```
struct sdshdr{  
    //  记录已使用长度  
    int len;  
    // 记录空闲未使用的长度  
    int free;  
    // 字符数组  
    char[] buf;  
};  
Redis的字符串也会遵守C语言的字符串的实现规则，即最后一个字符为空字符。然而这个空字符不会被计算在len里头

动态扩展：
当修改后的字符串长度len < 1M,则会分配与len相同长度的未使用的空间(free)(即翻倍)
当修改后的字符串长度len >= 1M,则会分配1M长度的未使用的空间(free)(每次增长1M)
刚开始s1 只有5个空闲位子，后面需要追加' world' 6个字符，很明显是不够的。那咋办？Redis会做以下三个操作：

计算出大小是否足够
开辟空间至满足所需大小
开辟与已使用大小len相同长度的空闲free空间（如果len < 1M）开辟1M长度的空闲free空间（如果len >= 1M）

```

#### 结构优势

```
快速获取字符串长度：
当想获取字符串长度时直接返回len即可

避免缓冲区溢出:
对一个C语言字符串进行strcat追加字符串的时候需要提前开辟需要的空间，如果不开辟空间的话可能会造成缓冲区溢出，而影响程序其他代码。如下图，有一个字符串s1="hello" 和 字符串s2="baby",现在要执行strcat(s1,"world"),并且执行前未给s1开辟空间，所以造成了缓冲区溢出

于Redis而言由于每次追加字符串时都会检查空间是否够用，所以不会存在缓冲区溢出问题.(即每次追加字符会检查空间)

降低空间分配次数提升内存使用效率:
1.空间预分配
即上面的动态扩展
2.惰性空间回收
惰性空间回收适用于字符串缩减操作。比如有个字符串s1="hello world"，对s1进行sdstrim(s1," world")操作，执行完该操作之后Redis不会立即回收减少的部分，而是会分配给下一个需要内存的程序
```

#### 对于字符串的解释

每当用户将一个值储存到字符串键里面的时候，Redis 都会对这个值进行检测，如果这个值能够被解释为以下两种类型的其中一种，那么 Redis 就会把这个值当做数字来处理：

==简单的说就是，对于正负数，且符合整数，浮点数的范围，就会被解释成相应的类型==

表 1-2 一些能够被 Redis 解释为数字的例子

| 值                     | Redis 解释这个值的方式                                       |
| :--------------------- | :----------------------------------------------------------- |
| `10086`                | 解释为整数。                                                 |
| `+894`                 | 解释为整数。                                                 |
| `-123`                 | 解释为整数。                                                 |
| `3.14`                 | 解释为浮点数。                                               |
| `+2.56`                | 解释为浮点数。                                               |
| `-5.12`                | 解释为浮点数。                                               |
| `12345678901234567890` | 这个值虽然是整数，但是因为它的大小超出了 `long long int` 类型能够容纳的范围，所以只能被解释为字符串。 |
| `3.14e5`               | 因为 Redis 不能解释使用科学记数法表示的浮点数，所以这个值只能被解释为字符串。 |
| `"one"`                | 解释为字符串。                                               |
| `"123abc"`             | 解释为字符串。                                               |



使用命令



==总结：==

==1.常规get,set.(参数扩展：可判断是否有值情况下进行操作)==

==2.批量mget,mset(减少IO，也支持判断是否有值情况下进行操作，MSETNX)==

==3.GETSET 获取原先值，并设置新值==

==4.支持字符串的一系列操作，获取字符串长度，根据位置获取|设置范围内字符，以及追加字符==

==5.对于整数浮点数，支持加减法，自加自减==



```
1.SET key value
SET 命令在成功创建字符串键之后将返回 OK 作为结果。比如说，通过执行以下命令，我们可以创建出一个字符串键
redis> SET song_title "Get Wild"
OK

2.SET key value [NX|XX]
如果在SET时，给定了NX，那么 SET 命令只会在键没有值的情况下执行设置操作，并返回 OK 表示设置成功；如果键已经存在，那么 SET 命令将放弃执行设置操作，并返回空值 nil 表示设置失败
redis> SET password "123456" NX
OK    -- 对尚未有值的 password 键进行设置，成功
redis> SET password "999999" NX
(nil)    -- password 键已经有了值，设置失败
如果用户在执行 SET 命令时给定了 XX 选项，那么 SET 命令只会在键已经有值的情况下执行设置操作，并返回 OK 表示设置成功；如果给定的键并没有值，那么 SET 命令将放弃执行设置操作，并返回空值表示设置失败。
redis> SET mysql-homepage "mysql.org"
OK    -- 为键 mysql-homepage 设置一个值
redis> SET mysql-homepage "mysql.com" XX
OK    -- 对键的值进行更新

3.GET key
GET 命令接受一个字符串键作为参数，然后返回与该键相关联的值。
redis> GET message
"hello world"
另一方面，如果用户给定的字符串键在数据库中并没有与之相关联的值，那么 GET 命令将返回一个空值：
redis> GET date
(nil)

4.GETSET key new_value
GETSET 命令就像 GET 命令和 SET 命令的组合版本，它首先获取字符串键目前已有的值，接着为键设置新值，最后把之前获取到的旧值返回给用户
redis> GET number    -- number 键现在的值为 "10086"
"10086"
redis> GETSET number "12345"
"10086"    -- 返回旧值
redis> GET number    -- number 键的值已被更新为 "12345"
"12345"

5.MSET key value [key value ...]
MSET 命令可以一次为多个字符串键设置值，此外，如果给定的字符串键已经有相关联的值，那么 MSET 命令也会直接使用新值去覆盖已有的旧值。
通过使用一条 MSET 命令去代替多条 SET 命令，可以将原本所需的多次网络通信降低为只需一次网络通信，从而有效地减少程序执行多个设置操作时所需的时间
redis> MSET message "hello world" number "10086" homepage "redis.io"
OK
redis> GET message
"hello world"
redis> GET number
"10086"
redis> GET homepage
"redis.io"

6.MGET key [key ...]
MGET 命令就是一个多键版本的 GET 命令，它接受一个或多个字符串键作为参数，并返回这些字符串键的值
可以将原本所需的多次网络通信降低为只需一次网络通信，从而有效地减少程序执行多个设置操作时所需的时间
redis> MGET message number homepage
1) "hello world"    -- message 键的值
2) "10086"          -- number 键的值
3) "redis.io"       -- homepage 键的值

7.MSETNX key value [key value ...]
MSETNX 跟 MSET 的主要区别在于 MSETNX 只会在所有给定键都不存在的情况下对键进行设置，而不会像 MSET 那样直接覆盖键已有的值：如果在给定键当中，有哪怕一个键已经有值了，那么 MSETNX 命令也会放弃对所有给定键的设置操作。MSETNX 命令在成功执行设置操作时返回 1 ，在放弃执行设置操作时则返回 0 。
redis> MSETNX k1 "one" k2 "two" k3 "three" k4 "four"
(integer) 0    -- 因为键 k4 已存在，所以 MSETNX 未能执行设置操作

8.STRLEN key
STRLEN：获取字符串值的字节长度
redis> GET number
"10086"
redis> STRLEN number    -- number 键的值长 5 字节
(integer) 5

9.GETRANGE key start end
通过使用 GETRANGE 命令，用户可以获取字符串值从 start 索引开始，直到 end 索引为止的所有内容
索引顺序，有正负之分，正：左边0开始，负：右边从-1开始
redis> GETRANGE message 0 4     -- 获取字符串值索引 0 至索引 4 上的内容
"hello"
redis> GETRANGE message 6 10    -- 获取字符串值索引 6 至索引 10 上的内容
"world"
redis> GETRANGE message -11 -7  -- 使用负数索引获取指定内容
"hello"

10.SETRANGE key index substitute
用户可以将字符串键的值从索引 index 开始的部分替换为指定的新内容，被替换内容的长度取决于新内容的长度：
redis> GET message
"hello world"
redis> SETRANGE message 6 "Redis"
(integer) 11    -- 字符串值当前的长度为 11 字节
redis> GET message
"hello Redis"

11.APPEND key suffix
通过调用 APPEND 命令，用户可以将给定的内容追加到字符串键已有值的末尾，APPEND 命令在执行追加操作之后，会返回字符串值当前的长度作为返回值。如果用户给定的键并不存在，那么 APPEND 命令会先将键的值初始化为空字符串 "" ，然后再执行追加操作
redis> GET description
"Redis"
edis> APPEND description " is a database"
(integer) 19    -- 追加操作执行完毕之后，值的长度

12.对于整数|不存在的键(默认0)，可以加减任意数，对于字符串|浮点数使用该命令返回错误error
INCRBY key increment
DECRBY key increment
redis> GET number
"100"
redis> INCRBY number 300     -- 将键的值加上 300
(integer) 400
和上面命令一样，但是是自加自减
INCR key
DECR key

13.对于浮点数|不存在的键(默认0)，可以加减任意数
INCRBYFLOAT key increment

```





