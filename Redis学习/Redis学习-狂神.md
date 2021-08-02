MyCat 分库

# Redis学习

> Remote dictionary Server 远程字典服务
>
> [Redis官网](https://redis.io/)、[Redis中文网](redis.cn)

```
Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。
```



## 一、安装过程

```shell
#安装redis 默认是稳定版 静默安装完成 使用mac安装
brew install redis

#redis使用
#启动Redis
方式一: brew services start redis@3.2 
方式二: redis-server /usr/local/etc/redis.conf
#查看ps(或者其他应用进程)
ps axu | grep redis(其他进程名)
isole            84721   0.0  0.0  4267768    900 s000  S+   10:08上午   0:00.00 grep redis
isole            84501   0.0  0.0  4309180   1568   ??  Ss   10:06上午   0:00.07 redis-server 127.0.0.1:6379
#关闭redis服务(直接kill +进程号)
方法一：kill 84721   
方法二：brew services stop redis
#连接redis客户端
redis-cli -h 127.0.0.1 -p 6379 如下: 127.0.0.1:6379> get("123")
#关闭redis客户端
redis-cli shutdown
#关闭服务(杀死进程)
sudo pkill redis-server
```

```sh
#安装redis linux安装 copy对应的离线包 解压包
tar xvf  redis-6.2.1.tar 
#安装所需环境
yum install gcc-c++
#安装
make install
#在对应的安装目录（默认/usr/local/bin） 启动redis服务 修改了配置文件的地址
redis-server redis-config/redis.conf 
#连接redis
redis-cli -h 127.0.0.1 -p 6379
```

### 1、Redis能干嘛？？？

1. 作为缓存，效率高， 高速缓存
2. 发布订阅系统
3. 地图信息更新











1、内存存储。持久化。内存中是断电就没有 需要持久化 rdb aof

redis-benchmark redis 性能测试 





**redis 是 单线程**

redis的瓶颈时根据机器的内存和网络带宽，cpu不是redis的瓶颈。

redis为什么是单线程还这么快？

1、误区1 ：高性能的服务器一定是多线程的？

2、误区2：多线程（CPU会上下文切换） 一定比单线程效率高？

核心：redis是把所有的数据都放到内存中，所以说单线程去操作内存效率最高，多线程（CPU上下文切换：耗时的操作） 对于内存系统来说 如果没有上下文切换效率就是最高的 多次读写都是在一个CPU上，在内存情况下 单线程是最佳方案。



## 二、五大数据类型

## 1、String

常用的命令

[命令官网地址](https://redis.io/commands#string)

```bash
# 设置一个value到redis中
127.0.0.1:6379> set key value
# 获取对应的key的value
127.0.0.1:6379> get key
"value"
# 设置多个value到redis中 
127.0.0.1:6379> mset key1 value1 key2 value2
# 获取多个对应的key的value 原子操作 要么一起成功 一起失败
127.0.0.1:6379> mget key1 key2
1) "value1"
2) "value2"
# 追加某个key的value
127.0.0.1:6379> APPEND key appendvlaue
(integer) 16
127.0.0.1:6379> get key
"valueappendvlaue"
127.0.0.1:6379> set number 100
#数字递减
127.0.0.1:6379> DECR number
(integer) 99
#数字递减指定值
127.0.0.1:6379> DECRBY number  10
(integer) 89
127.0.0.1:6379> 
#获取某个范围的值
127.0.0.1:6379> GETRANGE key 1 10
"alueappend"
#返回原本的值并且修改对应的value
127.0.0.1:6379> GETSET key newvaluebygetset
"valueappendvlaue"
#数字递增加
127.0.0.1:6379> INCR number
(integer) 90
#数字递增指定值
127.0.0.1:6379> INCRBY number 10
(integer) 100
#数字递增浮点数
127.0.0.1:6379> INCRBYFLOAT number 1.1
"101.1"
#如果这个key不存在就set值 可以一次性set多个 且是原子操作
127.0.0.1:6379> MSETNX key valuemsetnx
(integer) 0
127.0.0.1:6379> MSETNX key100 valuemsetnx
(integer) 1
#设置一个值的过期时间 单位ms
127.0.0.1:6379> PSETEX key100 10000000 value
#查看过期时间
127.0.0.1:6379> ttl key100
(integer) 9998
#设置一个值的过期时间 单位s
127.0.0.1:6379> SETEX key1200 100 svalue
127.0.0.1:6379> ttl key1200
(integer) 95
#如果key不存在，这种情况下等同SET命令。 当key存在时，什么也不做。SETNX是”SET if Not eXists”的简写
127.0.0.1:6379> SETNX key100 valeu
(integer) 0
127.0.0.1:6379> SETNX key1000 valeu
(integer) 1
#从第几位开始新增
127.0.0.1:6379> SETRANGE key 10 sahsaks11111111111111111111 
(integer) 37
#获取一个key对应的长度 不是string报错
127.0.0.1:6379> STRLEN key2
(integer) 21
```

### String使用场景

- 缓存，value对应的数字、字符串--对象缓存存储
- 计数器 逐渐累加 递减操作--粉丝数
- 统计多单位的数量，如 uid:99898:follow 0

### 底层原理 TODO



## 2、List  

> 双向链表 、先进后出
>
> **value有序可重复**

常用的命令

[官网地址](http://www.redis.cn/commands.html#list)

```bash
#往list中存值 从左插入
127.0.0.1:6379> LPUSH list 1 2 2 3 3 4 4 5 6 
(integer) 9
#往list中存值 从右插入
127.0.0.1:6379> RPUSH list 11 22  33  44  55  66
(integer) 13
#查看所有值
127.0.0.1:6379> LRANGE list 0 -1
1) "6"
2) "5"
3) "4"
4) "4"
5) "3"
6) "3"
7) "2"
8) "2"
9) "1"
#找到第一个值 找到指定下标的值
127.0.0.1:6379> LINDEX list 0
"5"
#把 value 插入存于 key 的列表中在基准值 pivot 的前面或后面。当 key 不存在时，这个list会被看作是空list，任何操作都不会发生。存list报错
127.0.0.1:6379> LINSERT list before 55 555
127.0.0.1:6379> LINSERT list after 55 55555
# key对应的list的长度。
127.0.0.1:6379> LLEN list
(integer) 15
# 如果这个list存在就在头部插入value
127.0.0.1:6379> LPUSHX list aaaaa
(integer) 16
# 如果这个list存在就在尾部插入value
127.0.0.1:6379> RPUSHX list bbbbb
# 移除并且返回 key 对应的 list 的第一个元素。 从左
127.0.0.1:6379> LPOP list
"aaaaa"
# 移除并且返回 key 对应的 list 的第一个元素。 从右
127.0.0.1:6379> RPOP list
"bbbbb"
#从存于 key 的列表里移除前 count 次出现的值为 value 的元素 不存在就是返回0
#count > 0: 从头往尾移除值为 value 的元素。
#count < 0: 从尾往头移除值为 value 的元素。
#count = 0: 移除所有值为 value 的元素。
127.0.0.1:6379> LREM list 2 2
(integer) 2
127.0.0.1:6379> LRANGE list 0 -1
 1) "5"
 2) "4"
 3) "4"
 4) "3"
 5) "3"
 6) "11"
 7) "22"
 8) "33"
 9) "44"
10) "555"
11) "55"
12) "55555"
13) "66"
#设置 index 位置的list元素的值为 value 从0开始
127.0.0.1:6379> LSET list 0 88888
OK
127.0.0.1:6379> LRANGE list 0 -1
 1) "88888"
 2) "4"
 3) "4"
 4) "3"
 5) "3"
 6) "11"
 7) "22"
 8) "33"
 9) "44"
10) "555"
11) "55"
12) "55555"
13) "66"


127.0.0.1:6379> LRANGE list 0 -1
 1) "88888"
 2) "4"
 3) "4"
 4) "3"
 5) "3"
 6) "11"
 7) "22"
 8) "33"
 9) "44"
10) "555"
11) "55"
12) "55555"
#修剪(trim)一个已存在的 list，这样 list 就会只包含指定范围的指定元素
127.0.0.1:6379> ltrim list 3 -3
OK
127.0.0.1:6379> LRANGE list 0 -1
1) "3"
2) "3"
3) "11"
4) "22"
5) "33"
6) "44"
7) "555"
# 从右边拼一个 移动到另一个list中
#  如果 source 和 destination 是同样的，那么这个操作等同于移除列表最后一个元素并且把该元素放在列表头部， 所以这个命令也可以当作是一个旋转列表的命令
127.0.0.1:6379> RPOPLPUSH list otherlist
"555"
127.0.0.1:6379> LRANGE otherlist 0 -1
1) "555"
#BRPOPLPUSH 是 RPOPLPUSH 的阻塞版本。 当 source 包含元素的时候，这个命令表现得跟 RPOPLPUSH 一模一样。 当 source 是空的时候，Redis将会阻塞这个连接，直到另一个客户端 push 元素进入或者达到 timeout 时限。 timeout 为 0 能用于无限期阻塞客户端。
127.0.0.1:6379> BRPOPLPUSH list1 otherlist 1
(nil)
(1.05s)
127.0.0.1:6379> BRPOPLPUSH list1 otherlist 0
"list1"
(46.43s)
# 另开客户端 插入值
127.0.0.1:6379> LPUSH list1 list1

# BLPOP 是阻塞式列表的弹出原语
127.0.0.1:6379> BLPOP list2 0
1) "list2"
2) "1"
(8.25s)
#另开客户端 插入值
127.0.0.1:6379> lpush list2 1
(integer) 1
#尝试多个值 RRPOP同理
127.0.0.1:6379> BLPOP list1 list2 list3 0
1) "list2"
2) "111"
(72.77s)

```



### 常见用途

- lindex  通过下标获取值 生产、消费者   生产者逐渐插入 消费值逐渐消费 
- 可以把list变成 栈、队列、阻塞队列 队列  **栈（先进后出 push pop 弹夹 只允许在一端操作）**、**队列（先进先出 排队  只允许在一端插入 另一端删除）**
- 消息排队！ 消息队列！

### 底层原理TODO



## 3、Set

> **无序不能重复**

[官网命令](http://www.redis.cn/commands.html#set)

```bash
#添加一个或多个指定的member元素到集合的 key中
127.0.0.1:6379> SADD set 1 2 3 4 5 6 7 1 1 1 2 3 3 4 5 
(integer) 7
#查看成员
127.0.0.1:6379> SMEMBERS set
1) "1"
2) "2"
3) "3"
4) "4"
5) "5"
6) "6"
7) "7"
#查看set的值
127.0.0.1:6379> SCARD set
(integer) 7


127.0.0.1:6379> SADD set1 11 123099 192836 218 2
(integer) 5
#查看set set1 的差值
127.0.0.1:6379> SDIFF set set1 
1) "1"
2) "3"
3) "4"
4) "5"
5) "6"
6) "7"
7) "28"
8) "23099"
9) "92836"

#查看set set1 的差值 并且把这个放到ste3 中 如果ste3存在 那么将重写
127.0.0.1:6379> SDIFFSTORE ste3 set set1
(integer) 9
127.0.0.1:6379> SMEMBERS ste3
1) "1"
2) "3"
3) "4"
4) "5"
5) "6"
6) "7"
7) "28"
8) "23099"
9) "92836"
#返回指定所有的集合的成员的交集.
127.0.0.1:6379> SINTER set set1
1) "2"
#查看set set1 的交集 并且把这个放到set3中 如果set3存在 那么将重写
127.0.0.1:6379> SINTERSTORE set3 set1 set
(integer) 1
127.0.0.1:6379> SMEMBERS set3
1) "2"

#集合中是否包含成员 存在1 不存在0
127.0.0.1:6379> SISMEMBER set 1
(integer) 1
127.0.0.1:6379> SISMEMBER set 11111
(integer) 0



127.0.0.1:6379> SMEMBERS set
 1) "1"
 2) "2"
 3) "3"
 4) "4"
 5) "5"
 6) "6"
 7) "7"
 8) "28"
 9) "23099"
10) "92836"
127.0.0.1:6379> SMEMBERS set1
1) "2"
2) "11"
3) "218"
4) "123099"
5) "192836"
#从原地址移动一个值到目标集合
127.0.0.1:6379> SMOVE set set1 28
(integer) 1
#随机pop一个值出去
127.0.0.1:6379> SPOP set
"3"
#随机pop指定个值出去
127.0.0.1:6379> SPOP set 2
1) "1"
2) "5"


#随机返回key集合中的一个元素.
127.0.0.1:6379> SRANDMEMBER set 1
1) "2"
#随机返回key集合中的n个元素. 如果大于size 取全部
127.0.0.1:6379> SRANDMEMBER set 122
1) "2"
2) "4"
3) "6"
4) "7"
5) "92836"
#随机返回key集合中的N个元素. 取绝对值
127.0.0.1:6379> SRANDMEMBER set -1
1) "2"
#随机返回key集合中的N个元素. 取绝对值
127.0.0.1:6379> SRANDMEMBER set -2
1) "6"
2) "4"

#在key集合中移除指定的元素
127.0.0.1:6379> srem set 2
(integer) 1
127.0.0.1:6379> srem set 4 6 7
(integer) 3

#取并集
127.0.0.1:6379> SMEMBERS set1
1) "2"
2) "11"
3) "28"
4) "218"
5) "123099"
6) "192836"
127.0.0.1:6379> SMEMBERS set
1) "92836"
127.0.0.1:6379> SUNION set set1
1) "2"
2) "11"
3) "28"
4) "218"
5) "92836"
6) "123099"
7) "192836"

#取并集 结果存储在destination集合中.
127.0.0.1:6379> SUNIONSTORE set001 set  set1
(integer) 7
127.0.0.1:6379> SRANDMEMBER set001 1000
1) "2"
2) "11"
3) "28"
4) "218"
5) "92836"
6) "123099"
7) "192836"

```

### 常见用途

- 微博 b站 共同关注 共同爱好 共同等字眼
- 抽随机 抽奖
- 差集、交集、并集

### 底层原理TODO



## 4、Hash

Hset hashmap key value set 值

```bash
#设置 key 指定的哈希集中指定字段的值
127.0.0.1:6379> hset hashmap key1 value1 key2 value2 key3 value23
(integer) 2
#返回 key 指定的哈希集中所有的字段和值
127.0.0.1:6379> HGETALL hashmap
1) "key1"
2) "value1"
3) "key2"
4) "value2"
5) "key3"
6) "value23"
#返回 key 指定的哈希集中该字段所关联的值
127.0.0.1:6379> hget hashmap key1
"value1"
#移除指定的key 成功1 失败0
127.0.0.1:6379> HDEL hashmap key1
(integer) 1
127.0.0.1:6379> HDEL hashmap key11
(integer) 0
#是否存在key 成功1 失败0
127.0.0.1:6379> HEXISTS hashmap key1
(integer) 0
127.0.0.1:6379> HEXISTS hashmap key2
(integer) 1
# 指定值增加int
127.0.0.1:6379> HSET hashmap key5 5
(integer) 1
127.0.0.1:6379> HINCRBY hashmap key5 1
(integer) 6
127.0.0.1:6379> HINCRBY hashmap key5 -1
(integer) 5

# 指定值增加float
127.0.0.1:6379> HINCRBYFLOAT hashmap key5 1.0
"6"
127.0.0.1:6379> HINCRBYFLOAT hashmap key5 1.01
"7.01"
#查看所有的key值
127.0.0.1:6379> HKEYS hashmap
1) "key2"
2) "key3"
3) "key5"
#返回 key 指定的哈希集包含的字段的数量
127.0.0.1:6379> HLEN hashmap
(integer) 3
#获取指定key值
127.0.0.1:6379> HMGET hashmap key1 key3
1) (nil)
2) "value23"
#设置 key 指定的哈希集中指定字段的值
127.0.0.1:6379> HMSET hashmap key11 value11 key22 value22
OK
#如果这个key不存在的时候设置值 1 赋值成功 0 已经有字段 赋值失败
127.0.0.1:6379> HSETNX hashmap key 11
(integer) 1
127.0.0.1:6379> HSETNX hashmap key 11
(integer) 0
#返回指定字段的长度
127.0.0.1:6379> HSTRLEN hashmap  key
(integer) 2
#获取所有value
127.0.0.1:6379> HVALS hashmap
1) "value2"
2) "value23"
3) "7.01"
4) "value11"
5) "value22"
6) "11"
```



### 常见用途

- 用户信息保存  userid  name age  经常变动的信息 适合对象的存储

  

### 底层原理TODO



## 5、Zset 

> [官网地址](http://www.redis.cn/commands/zadd.html)
>
> 有序集合  有权重 排序 score



## 

```bash
#添加值
127.0.0.1:6379> ZADD zset 1 "value1" 2 "value2" 33 "value33"
(integer) 3
#查看所有的值
127.0.0.1:6379> ZRANGE zset 0 -1
1) "value1"
2) "value2"
3) "value33"
#查看部分的值
127.0.0.1:6379> ZRANGE zset 0 1
1) "value1"
2) "value2"
#返回返回key的有序集元素个数。
127.0.0.1:6379> ZCARD zset
(integer) 3
#指定分数范围的元素个数。
#正负无穷
127.0.0.1:6379> ZCOUNT zset -inf +inf
(integer) 3
#指定区间
127.0.0.1:6379> ZCOUNT zset 0 2
(integer) 2

#给指定的value分数增加
127.0.0.1:6379> ZINCRBY zset 10 value1
"11"
127.0.0.1:6379> ZRANGE zset 0 -1
1) "value2"
2) "value1"
3) "value33"

# 交集放到新的目标地址中 需要设置几个key的交集   weights 后面是乘法因子 每个key都要乘以对应的 
127.0.0.1:6379> ZINTERSTORE zset12 2 zset1 zset WEIGHTS 3 4
(integer) 2
127.0.0.1:6379> ZRANGE zset12 0 -1 WITHSCORES
1) "value2"
2) "14"
3) "value33"
4) "141"

#指定成员中的成员个数 最小最大分值
127.0.0.1:6379> ZLEXCOUNT zset12  - +
(integer) 2
#指定成员之间的个数
127.0.0.1:6379> ZLEXCOUNT zset12  [value2 [value33
(integer) 2
#区间中的
127.0.0.1:6379> ZLEXCOUNT zset12  [value2 +
(integer) 2
127.0.0.1:6379> ZLEXCOUNT zset12 -  [value2 
(integer) 1

#弹出最高分数的几个值
127.0.0.1:6379> ZPOPMAX zset1 2
1) "value12"
2) "12"
3) "value33"
4) "3"
#弹出最低分数的几个值
127.0.0.1:6379> ZPOPMIN zset 2
1) "value2"
2) "2"
3) "value1"
4) "11"

#返回指定成员区间内的成员，按成员字典正序排序 顺序
127.0.0.1:6379> ZRANGEBYLEX  zset  - +
1) "value33"

#获取指定区间或者指定对象的的前后最后的值 可分页 “(“ 符号作为开头表示小于 顺序
127.0.0.1:6379> zadd zset11 10 a 11 b 12 c 33 v 13 n
(integer) 5
127.0.0.1:6379> ZRANGEBYLEX zset11 - + limit 1 2
1) "b"
2) "c"
127.0.0.1:6379> ZRANGEBYLEX zset11 [c + limit 0 10
1) "c"
2) "n"
3) "v"


#获取指定分数的前后最后的值 可分页 “(“ 符号作为开头表示小于
127.0.0.1:6379> ZRANGEBYSCORE zset11 0 10  limit 0 1
1) "a"
127.0.0.1:6379> ZRANGEBYSCORE zset11 (0 (30  limit 0 3
1) "a"
2) "b"
3) "c"

#返回有序集key中成员member的排名
127.0.0.1:6379> zrank zset11 c
(integer) 2

#移除指定member
127.0.0.1:6379> ZREM zset11 v
(integer) 1
127.0.0.1:6379> ZRANGE zset11 0 -1 WITHSCORES
1) "a"
2) "10"
3) "b"
4) "11"
5) "c"
6) "12"
7) "n"
8) "13"
#所以指定区间 [到指定元素
127.0.0.1:6379> ZREMRANGEBYLEX zset11 - [c
(integer) 3
#移除所有
127.0.0.1:6379> ZREMRANGEBYLEX zset11 - +
(integer) 1
#所以指定区间 (小于
127.0.0.1:6379> ZREMRANGEBYLEX zset11 - (value33
(integer) 0
# 移除有序集key中，指定排名(rank)区间内的所有成员
127.0.0.1:6379> zadd zset11 1 a 2 b 3 c 4 d 5 f 6 h 7 j
(integer) 7
127.0.0.1:6379> ZREMRANGEBYRANK zset11 0 5
(integer) 6
127.0.0.1:6379> ZRANGE zset11 0 -1
1) "j"
# 移除有序集key中，指定分数区间内的所有成员
127.0.0.1:6379> ZREMRANGEBYSCORE zset11 3 5
(integer) 3
127.0.0.1:6379> ZRANGE zset11 0 -1
1) "a"
2) "b"
3) "h"
4) "j"
#返回有序集key中，指定区间内的成员 倒序
127.0.0.1:6379> ZREVRANGE zset11 1 4
1) "h"
2) "f"
3) "d"
4) "c"
#指定成员区间内的成员  - + 要反过来
127.0.0.1:6379> ZREVRANGEBYLEX zset11 + - limit 0 10
1) "j"
2) "h"
3) "f"
4) "d"
5) "c"
6) "b"
7) "a"
#指定范围
127.0.0.1:6379> ZREVRANGEBYLEX zset11 + [f limit 0 10
1) "j"
2) "h"
3) "f"
#指定成员
127.0.0.1:6379> ZREVRANGEBYLEX zset11 + (f limit 0 10
1) "j"
2) "h"


#指定分数之间的值
127.0.0.1:6379> ZREVRANGEBYSCORE zset11 10 5
1) "j"
2) "h"
3) "f"
#小于某个段位
127.0.0.1:6379> ZREVRANGEBYSCORE zset11 10 (5
1) "j"
2) "h"
#全部
127.0.0.1:6379> ZREVRANGEBYSCORE zset11 +inf -inf
1) "j"
2) "h"
3) "f"
4) "d"
5) "c"
6) "b"
7) "a"



#倒序排序
127.0.0.1:6379> ZREVRANGE zset11 0 -1
1) "j"
2) "h"
3) "f"
4) "d"
5) "c"
6) "b"
7) "a"

#返回有序集key中，成员member的score值
127.0.0.1:6379> ZSCORE zset11 a
"1"

#计算给定的numkeys个有序集合的并集，并且把结果放到destination中。 要设置几个key
127.0.0.1:6379> ZUNIONSTORE des 2 zset11 zset WEIGHTS 1 10
(integer) 8
127.0.0.1:6379> ZRANGE des 0 -1
1) "a"
2) "b"
3) "c"
4) "d"
5) "f"
6) "h"
7) "j"
8) "value33"
#3个key
127.0.0.1:6379> ZUNIONSTORE des 3 zset11 zset1 zset WEIGHTS 1 10 10
(integer) 10


```

### 常见用途

- 存储班级成绩表
- 工资表排序 set排序
- 排行榜 
- 重要消息  带权重进行判断

### 底层原理TODO



## 三、特殊类型：

## 1、Geospatial ⭐️

> 地理位置 geo 
>
> [命令地址](http://www.redis.cn/commands/geoadd.html)

- 有效的经度从-180度到180度。

- 有效的纬度从-85.05112878度到85.05112878度

- 南北极不能输入

  ```bash
  #添加位置记录 合肥 成都
  127.0.0.1:6379> geoadd chinaMap 117.27 31.86 hf 104.06 30.67 cd
  (integer) 2
  #查看两地位置 [m|km|ft|mi] 可以设置单位
  127.0.0.1:6379> GEODIST chinaMap hf bj km
  "897.6757"
  # 返回的 geohash 的位置与用户给定的位置元素的位置一一对应 
  127.0.0.1:6379> GEOHASH chinaMap bj
  1) "wx4fbxxfke0"
  #返回对应的经纬度
  127.0.0.1:6379> GEOPOS chinaMap bj hf
  1) 1) "116.39999896287918091"
     2) "39.90000009167092543"
  2) 1) "117.27000027894973755"
     2) "31.85999891446920884"
  #以指定的纬度作为中心 方圆多大半径的member
  127.0.0.1:6379> GEORADIUS chinaMap 110 37 10000 km
  1) "cd"
  2) "hf"
  3) "tj"
  4) "bj"
  #指定位置中心的元素
  127.0.0.1:6379> GEORADIUSBYMEMBER chinaMap bj 1000 km
  1) "tj"
  2) "bj"
  3) "hf"
  ```

### 常见用途

- 朋友的定位
- 附近的人  
- 打车距离计算 直线距离

### 底层原理TODO

- 底层原理是Zset



## 2、Hyperloglog ⭐️

> 在数学上，基数或势，即集合中包含的元素的“个数”（参见势的比较），是日常交流中基数的概念在数学上的精确化（并使之不再受限于有限情形）
>
> [官网命令](http://www.redis.cn/commands/pfmerge.html)

可以顺带学一下布隆过滤器 

A{1,3,5,7,8,7}

A{1,3,5,7,8,}

什么是基数：集合内不重复的元素 =5 可以接受误差





```bash
# 添加参数 唯一对象的基数
127.0.0.1:6379> PFADD hlllog a b c d e f g a b c d e f g
(integer) 1
#如果存在 就返回0 无效添加
127.0.0.1:6379> PFADD hlllog a c  d
(integer) 0
#如果不存在 就返回1
127.0.0.1:6379> PFADD hlllog a c  d k
(integer) 1

#查看当参数为一个key时,返回存储在HyperLogLog结构体的该变量的近似基数，如果该变量不存在,则返回0.
127.0.0.1:6379> PFCOUNT hlllog
(integer) 7

127.0.0.1:6379> PFCOUNT hlllog
(integer) 8

127.0.0.1:6379> PFMERGE hll1 hlllog hll
OK
127.0.0.1:6379> PFCOUNT hll1
(integer) 12
#将多个 HyperLogLog 合并（merge）为一个 HyperLogLog ， 合并后的 HyperLogLog 的基数接近于所有输入 HyperLogLog 的可见集合（observed set）的并集.
127.0.0.1:6379> PFMERGE hll1 hlllog hll
OK
127.0.0.1:6379> PFCOUNT hll1
(integer) 12
```

Hyperloglog  占用的内存是固定 2^64 不同的元素的技术 只需要12kb内存 如果要从内存角度来比较的话 hyperloglog首选

传统的方式 set保存用户的id 然后可以统计set中的元素的数量做为标准  这个方式如果保存了大量的用户id 会占用很多内存

### 常见用途

- 网页的UV 一个人访问一个网站多次 还是算作一个人
- 0.81% 的错误率  统计UV任务 可以忽略不计

### 底层原理TODO





## 3、Bitmap 位运算🌟

> 位图计算 bitmaps 位图 数据结构 都是操作二进制 来进行记录 就只有0 1 两个状态

```bash
#设置或者清空key的value(字符串)在offset处的bit值。
127.0.0.1:6379> SETBIT biykey 1 1
(integer) 0
#获取offset的bit值
127.0.0.1:6379> GETBIT bitkey 1
(integer) 1
#获取值
127.0.0.1:6379> get bitkey
"p\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\b"
#统计字符串被设置为1的bit数.
127.0.0.1:6379> SETBIT bitkey 1001 0
(integer) 0
127.0.0.1:6379> BITCOUNT bitkey
(integer) 3
#返回一个位置，把字符串当做一个从左到右的字节数组，第一个符合条件的在位置0，其次在位置8，
BITPOS
#TODO
BITFIELD
#对一个或多个保存二进制位的字符串 key 进行位元操作，并将结果保存到 destkey  AND 、 OR 、 NOT 、 XOR
127.0.0.1:6379> set bitkeyy1 abcdjs
OK
127.0.0.1:6379> set bitkeyy2 abcdaasskllkjs
OK
127.0.0.1:6379> BITOP and  dest bitkeyy1 bitkeyy2
(integer) 14
127.0.0.1:6379> get dest
"abcd`a\x00\x00\x00\x00\x00\x00\x00\x00
```



### 常见用途

- 统计用户信息 活跃 不活跃 登陆 未登陆 打卡  365天打卡 两个状态都是可以使用bitmaps
- 打卡 使用bitmap来记录周一到周四的打卡



## 四、事务

> 事务 ACID 同时成功 同时失败 原子性
>
> [事务命令](http://www.redis.cn/commands.html#transactions)

Redis事务 单条保存是保存原子性的。但是事务不保存原子性 

redis事务 ：一组命令的集合 一个事务中的所有命令都会被序列化 在事务执行中 会按照顺序执行

```bash
#标记一个事务块的开始。 随后的指令将在执行EXEC时作为一个原子执行
127.0.0.1:6379> MULTI
OK
#执行操作 命令入队 
127.0.0.1:6379(TX)> SADD v1 aaa
QUEUED
127.0.0.1:6379(TX)> Sadd n 1111
QUEUED
127.0.0.1:6379(TX)> SADD kkkkk 11111
QUEUED
#执行事务
127.0.0.1:6379(TX)> EXEC
1) (integer) 1
2) (integer) 1
3) (integer) 1


127.0.0.1:6379> MULTI
OK
127.0.0.1:6379(TX)> SADD keu1111 11111
QUEUED
#取消事物 队列里面的都取消
127.0.0.1:6379(TX)> DISCARD
OK

```

#### 错误类型

- ​	编译型错误   代码有问题 命令报错  事务中所有的命令都不会被执行

- ```bash
  127.0.0.1:6379> MULTI
  OK
  127.0.0.1:6379(TX)> sahh ajsj 
  (error) ERR unknown command `sahh`, with args beginning with: `ajsj`, 
  127.0.0.1:6379(TX)> SADD lqooi1 111
  QUEUED
  127.0.0.1:6379(TX)> EXEC
  (error) EXECABORT Transaction discarded because of previous errors.
  
  ```

- ​	运行时异常  如果事务中存在语法性 那么执行命令的时候 其他命令是可以正常执行



乐观锁

监视 watch

悲观锁：很悲观 什么时候都回出现问题

乐观锁：很乐观 什么时候都不会出现问题  所以不会上锁  更新数据的时候去判断一下  在此期间是否有人呢修改过这个数据 

​	1、获取version版本

​	2、更新的时候比较version

redis监视测试

redis 乐观锁 通过使用watch 监控 

​		事务执行完毕后回自动解锁 无需手动unwatch解锁



## 五、Java实战

### 1、Java操作redis

#### 1）Jedis操作

> jedis 是官方推荐的Java连接开发工具  **老的**
>
> jedis:采用直连  多个线程操作不安全 为了避免不安全 采用jedispool连接池  BIO模式

```java
public class RawJedisTest {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("localhost", 6379);
//        localhost.auth("12345678");
        jedis.set("keg", "zhangsan");
        String key = jedis.get("keg");
        System.out.println(jedis.ping());
        System.out.println(key);
    }
}



public class TestHash {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        Map<String, String> map = new HashMap<String, String>();
        map.put("key1", "value1");
        map.put("key2", "value2");
        map.put("key3", "value3");
        map.put("key4", "value4");
        //添加名称为hash（key）的hash元素
        jedis.hmset("hash", map);
        //向名称为hash的hash中添加key为key5，value为value5元素
        jedis.hset("hash", "key5", "value5");
        //return Map<String,String>
        System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
        //return Set<String>
        System.out.println("散列hash的所有键为：" + jedis.hkeys("hash"));
        //return List<String>
        System.out.println("散列hash的所有值为：" + jedis.hvals("hash"));
        System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6：" + jedis.hincrBy("hash", "key6", 6));
        System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
        System.out.println("将key6保存的值加上一个整数，如果key6不存在则添加key6：" + jedis.hincrBy("hash", "key6", 3));
        System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
        System.out.println("删除一个或者多个键值对：" + jedis.hdel("hash", "key2"));
        System.out.println("散列hash的所有键值对为：" + jedis.hgetAll("hash"));
        System.out.println("散列hash中键值对的个数：" + jedis.hlen("hash"));
        System.out.println("判断hash中是否存在key2：" + jedis.hexists("hash", "key2"));
        System.out.println("判断hash中是否存在key3：" + jedis.hexists("hash", "key3"));
        System.out.println("获取hash中的值：" + jedis.hmget("hash", "key3"));
        System.out.println("获取hash中的值：" + jedis.hmget("hash", "key3", "key4"));
    }
}


public class TestKey {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);

        System.out.println("清空数据：" + jedis.flushDB());
        System.out.println("判断某个键是否存在：" + jedis.exists("username"));
        System.out.println("新增<'username','kuangshen'>的键值对：" + jedis.set("username", "kuangshen"));
        System.out.println("新增<'password','password'>的键值对：" + jedis.set("password", "password"));
        System.out.print("系统中所有的键如下：");
        Set<String> keys = jedis.keys("*");
        System.out.println(keys);
        System.out.println("删除键password:" + jedis.del("password"));
        System.out.println("判断键password是否存在：" + jedis.exists("password"));
        System.out.println("查看键username所存储的值的类型：" + jedis.type("username"));
        System.out.println("随机返回key空间的一个：" + jedis.randomKey());
        System.out.println("重命名key：" + jedis.rename("username", "name"));
        System.out.println("取出改后的name：" + jedis.get("name"));
        System.out.println("按索引查询：" + jedis.select(0));
        System.out.println("删除当前选择数据库中的所有key：" + jedis.flushDB());
        System.out.println("返回当前数据库中key的数目：" + jedis.dbSize());
        System.out.println("删除所有数据库中的所有key：" + jedis.flushAll());
    }
}

public class TestList {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("===========添加一个list===========");
        jedis.lpush("collections", "ArrayList", "Vector", "Stack", "HashMap", "WeakHashMap", "LinkedHashMap");
        jedis.lpush("collections", "HashSet");
        jedis.lpush("collections", "TreeSet");
        jedis.lpush("collections", "TreeMap");
        //-1代表倒数第一个元素，-2代表倒数第二个元素,end为-1表示查询全部
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("collections区间0-3的元素：" + jedis.lrange("collections", 0, 3));
        System.out.println("===============================");
        // 删除列表指定的值 ，第二个参数为删除的个数（有重复时），后add进去的值先被删，类似于出栈
        System.out.println("删除指定元素个数：" + jedis.lrem("collections", 2, "HashMap"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("删除下表0-3区间之外的元素：" + jedis.ltrim("collections", 0, 3));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("collections列表出栈（左端）：" + jedis.lpop("collections"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("collections添加元素，从列表右端，与lpush相对应：" + jedis.rpush("collections", "EnumMap"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("collections列表出栈（右端）：" + jedis.rpop("collections"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("修改collections指定下标1的内容：" + jedis.lset("collections", 1, "LinkedArrayList"));
        System.out.println("collections的内容：" + jedis.lrange("collections", 0, -1));
        System.out.println("===============================");
        System.out.println("collections的长度：" + jedis.llen("collections"));
        System.out.println("获取collections下标为2的元素：" + jedis.lindex("collections", 2));
        System.out.println("===============================");
        jedis.lpush("sortedList", "3", "6", "2", "0", "7", "4");
        System.out.println("sortedList排序前：" + jedis.lrange("sortedList", 0, -1));
        System.out.println(jedis.sort("sortedList"));
        System.out.println("sortedList排序后：" + jedis.lrange("sortedList", 0, -1));
    }
}


public class TestMulti {
    public static void main(String[] args) {
        //创建客户端连接服务端，redis服务端需要被开启
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello", "world");
        jsonObject.put("name", "java");
        //开启事务
        Transaction multi = jedis.multi();
        String result = jsonObject.toJSONString();
        try {
            //向redis存入一条数据
            multi.set("json", result);
            //再存入一条数据
            multi.set("json2", result);
            //这里引发了异常，用0作为被除数
            int i = 100 / 0;
            //如果没有引发异常，执行进入队列的命令
            multi.exec();
        } catch (Exception e) {
            e.printStackTrace();
            //如果出现异常，回滚
            multi.discard();
        } finally {
            System.out.println(jedis.get("json"));
            System.out.println(jedis.get("json2"));
            //最终关闭客户端
            jedis.close();
        }
    }
}


package study.redis.test;

import redis.clients.jedis.Jedis;

public class TestSet {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("============向集合中添加元素（不重复）============");
        System.out.println(jedis.sadd("eleSet", "e1", "e2", "e4", "e3", "e0", "e8", "e7", "e5"));
        System.out.println(jedis.sadd("eleSet", "e6"));
        System.out.println(jedis.sadd("eleSet", "e6"));
        System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
        System.out.println("删除一个元素e0：" + jedis.srem("eleSet", "e0"));
        System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
        System.out.println("删除两个元素e7和e6：" + jedis.srem("eleSet", "e7", "e6"));
        System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
        System.out.println("随机的移除集合中的一个元素：" + jedis.spop("eleSet"));
        System.out.println("随机的移除集合中的一个元素：" + jedis.spop("eleSet"));
        System.out.println("eleSet的所有元素为：" + jedis.smembers("eleSet"));
        System.out.println("eleSet中包含元素的个数：" + jedis.scard("eleSet"));
        System.out.println("e3是否在eleSet中：" + jedis.sismember("eleSet", "e3"));
        System.out.println("e1是否在eleSet中：" + jedis.sismember("eleSet", "e1"));
        System.out.println("e1是否在eleSet中：" + jedis.sismember("eleSet", "e5"));
        System.out.println("=================================");
        System.out.println(jedis.sadd("eleSet1", "e1", "e2", "e4", "e3", "e0", "e8", "e7", "e5"));
        System.out.println(jedis.sadd("eleSet2", "e1", "e2", "e4", "e3", "e0", "e8"));
        //移到集合元素
        System.out.println("将eleSet1中删除e1并存入eleSet3中：" + jedis.smove("eleSet1", "eleSet3", "e1"));
        System.out.println("将eleSet1中删除e2并存入eleSet3中：" + jedis.smove("eleSet1", "eleSet3", "e2"));
        System.out.println("eleSet1中的元素：" + jedis.smembers("eleSet1"));
        System.out.println("eleSet3中的元素：" + jedis.smembers("eleSet3"));
        System.out.println("============集合运算=================");
        System.out.println("eleSet1中的元素：" + jedis.smembers("eleSet1"));
        System.out.println("eleSet2中的元素：" + jedis.smembers("eleSet2"));
        System.out.println("eleSet1和eleSet2的交集:" + jedis.sinter("eleSet1", "eleSet2"));
        System.out.println("eleSet1和eleSet2的并集:" + jedis.sunion("eleSet1", "eleSet2"));
        //eleSet1中有，eleSet2中没有
        System.out.println("eleSet1和eleSet2的差集:" + jedis.sdiff("eleSet1", "eleSet2"));
        //求交集并将交集保存到dstkey的集合
        jedis.sinterstore("eleSet4", "eleSet1", "eleSet2");
        System.out.println("eleSet4中的元素：" + jedis.smembers("eleSet4"));
    }
}


public class TestString {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("127.0.0.1", 6379);
        jedis.flushDB();
        System.out.println("===========增加数据===========");
        System.out.println(jedis.set("key1", "value1"));
        System.out.println(jedis.set("key2", "value2"));
        System.out.println(jedis.set("key3", "value3"));
        System.out.println("删除键key2:" + jedis.del("key2"));
        System.out.println("获取键key2:" + jedis.get("key2"));
        System.out.println("修改key1:" + jedis.set("key1", "value1Changed"));
        System.out.println("获取key1的值：" + jedis.get("key1"));
        System.out.println("在key3后面加入值：" + jedis.append("key3", "End"));
        System.out.println("key3的值：" + jedis.get("key3"));
        System.out.println("增加多个键值对：" + jedis.mset("key01", "value01", "key02", "value02", "key03", "value03"));
        System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03"));
        System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03", "key04"));
        System.out.println("删除多个键值对：" + jedis.del("key01", "key02"));
        System.out.println("获取多个键值对：" + jedis.mget("key01", "key02", "key03"));

        jedis.flushDB();
        System.out.println("===========新增键值对防止覆盖原先值==============");
        System.out.println(jedis.setnx("key1", "value1"));
        System.out.println(jedis.setnx("key2", "value2"));
        System.out.println(jedis.setnx("key2", "value2-new"));
        System.out.println(jedis.get("key1"));
        System.out.println(jedis.get("key2"));

        System.out.println("===========新增键值对并设置有效时间=============");
        System.out.println(jedis.setex("key3", 2, "value3"));
        System.out.println(jedis.get("key3"));
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(jedis.get("key3"));

        System.out.println("===========获取原值，更新为新值==========");
        System.out.println(jedis.getSet("key2", "key2GetSet"));
        System.out.println(jedis.get("key2"));

        System.out.println("获得key2的值的字串：" + jedis.getrange("key2", 2, 4));
    }
}

```





Spring推荐使用lettuce连接

**lettuce：采用netty  实例可以在多个线程中进行共享 不存在线程不安全的情况 减少线程数 更像NIO模式**

使用redisTemplate操作

```properties
# redis 配置
spring.redis.lettuce.pool.max-active= 8
spring.redis.port=6379
spring.redis.password=
spring.redis.ssl=false
spring.redis.host=
spring.redis.database=0
spring.redis.client-type=lettuce
spring.redis.lettuce.pool.max-idle=8
spring.redis.lettuce.pool.max-wait=-1ms
spring.redis.lettuce.shutdown-timeout=3000ms
spring.redis.lettuce.pool.min-idle=0
```







### 2、Redis config配置

```shell
# Redis configuration file example.
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
#
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.

################################## INCLUDES ###################################

# Include one or more other config files here.  This is useful if you
# have a standard template that goes to all Redis servers but also need
# to customize a few per-server settings.  Include files can include
# other files, so use this wisely.
#
# Note that option "include" won't be rewritten by command "CONFIG REWRITE"
# from admin or Redis Sentinel. Since Redis always uses the last processed
# line as value of a configuration directive, you'd better put includes
# at the beginning of this file to avoid overwriting config change at runtime.
#
# If instead you are interested in using includes to override configuration
# options, it is better to use include as the last line.
#
# include /path/to/local.conf
# include /path/to/other.conf

################################## MODULES #####################################

# Load modules at startup. If the server is not able to load modules
# it will abort. It is possible to use multiple loadmodule directives.
#
# loadmodule /path/to/my_module.so
# loadmodule /path/to/other_module.so

################################## NETWORK #####################################

# By default, if no "bind" configuration directive is specified, Redis listens
# for connections from all available network interfaces on the host machine.
# It is possible to listen to just one or multiple selected interfaces using
# the "bind" configuration directive, followed by one or more IP addresses.
# Each address can be prefixed by "-", which means that redis will not fail to
# start if the address is not available. Being not available only refers to
# addresses that does not correspond to any network interfece. Addresses that
# are already in use will always fail, and unsupported protocols will always BE
# silently skipped.
#
# Examples:
#
# bind 192.168.1.100 10.0.0.1     # listens on two specific IPv4 addresses
# bind 127.0.0.1 ::1              # listens on loopback IPv4 and IPv6
# bind * -::*                     # like the default, all available interfaces
#
# ~~~ WARNING ~~~ If the computer running Redis is directly exposed to the
# internet, binding to all the interfaces is dangerous and will expose the
# instance to everybody on the internet. So by default we uncomment the
# following bind directive, that will force Redis to listen only on the
# IPv4 and IPv6 (if available) loopback interface addresses (this means Redis
# will only be able to accept client connections from the same host that it is
# running on).
#
# IF YOU ARE SURE YOU WANT YOUR INSTANCE TO LISTEN TO ALL THE INTERFACES
# JUST COMMENT OUT THE FOLLOWING LINE.
# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
bind 127.0.0.1 -::1

# Protected mode is a layer of security protection, in order to avoid that
# Redis instances left open on the internet are accessed and exploited.
#
# When protected mode is on and if:
#
# 1) The server is not binding explicitly to a set of addresses using the
#    "bind" directive.
# 2) No password is configured.
#
# The server only accepts connections from clients connecting from the
# IPv4 and IPv6 loopback addresses 127.0.0.1 and ::1, and from Unix domain
# sockets.
#
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured, nor a specific set of interfaces
# are explicitly listed using the "bind" directive.
protected-mode yes

# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6379

# TCP listen() backlog.
#
# In high requests-per-second environments you need a high backlog in order
# to avoid slow clients connection issues. Note that the Linux kernel
# will silently truncate it to the value of /proc/sys/net/core/somaxconn so
# make sure to raise both the value of somaxconn and tcp_max_syn_backlog
# in order to get the desired effect.
tcp-backlog 511

# Unix socket.
#
# Specify the path for the Unix socket that will be used to listen for
# incoming connections. There is no default, so Redis will not listen
# on a unix socket when not specified.
#
# unixsocket /run/redis.sock
# unixsocketperm 700

# Close the connection after a client is idle for N seconds (0 to disable)
timeout 0

# TCP keepalive.
#
# If non-zero, use SO_KEEPALIVE to send TCP ACKs to clients in absence
# of communication. This is useful for two reasons:
#
# 1) Detect dead peers.
# 2) Force network equipment in the middle to consider the connection to be
#    alive.
#
# On Linux, the specified value (in seconds) is the period used to send ACKs.
# Note that to close the connection the double of the time is needed.
# On other kernels the period depends on the kernel configuration.
#
# A reasonable value for this option is 300 seconds, which is the new
# Redis default starting with Redis 3.2.1.
tcp-keepalive 300

################################# TLS/SSL #####################################

# By default, TLS/SSL is disabled. To enable it, the "tls-port" configuration
# directive can be used to define TLS-listening ports. To enable TLS on the
# default port, use:
#
# port 0
# tls-port 6379

# Configure a X.509 certificate and private key to use for authenticating the
# server to connected clients, masters or cluster peers.  These files should be
# PEM formatted.
#
# tls-cert-file redis.crt 
# tls-key-file redis.key

# Normally Redis uses the same certificate for both server functions (accepting
# connections) and client functions (replicating from a master, establishing
# cluster bus connections, etc.).
#
# Sometimes certificates are issued with attributes that designate them as
# client-only or server-only certificates. In that case it may be desired to use
# different certificates for incoming (server) and outgoing (client)
# connections. To do that, use the following directives:
#
# tls-client-cert-file client.crt
# tls-client-key-file client.key

# Configure a DH parameters file to enable Diffie-Hellman (DH) key exchange:
#
# tls-dh-params-file redis.dh

# Configure a CA certificate(s) bundle or directory to authenticate TLS/SSL
# clients and peers.  Redis requires an explicit configuration of at least one
# of these, and will not implicitly use the system wide configuration.
#
# tls-ca-cert-file ca.crt
# tls-ca-cert-dir /etc/ssl/certs

# By default, clients (including replica servers) on a TLS port are required
# to authenticate using valid client side certificates.
#
# If "no" is specified, client certificates are not required and not accepted.
# If "optional" is specified, client certificates are accepted and must be
# valid if provided, but are not required.
#
# tls-auth-clients no
# tls-auth-clients optional

# By default, a Redis replica does not attempt to establish a TLS connection
# with its master.
#
# Use the following directive to enable TLS on replication links.
#
# tls-replication yes

# By default, the Redis Cluster bus uses a plain TCP connection. To enable
# TLS for the bus protocol, use the following directive:
#
# tls-cluster yes

# By default, only TLSv1.2 and TLSv1.3 are enabled and it is highly recommended
# that older formally deprecated versions are kept disabled to reduce the attack surface.
# You can explicitly specify TLS versions to support.
# Allowed values are case insensitive and include "TLSv1", "TLSv1.1", "TLSv1.2",
# "TLSv1.3" (OpenSSL >= 1.1.1) or any combination.
# To enable only TLSv1.2 and TLSv1.3, use:
#
# tls-protocols "TLSv1.2 TLSv1.3"

# Configure allowed ciphers.  See the ciphers(1ssl) manpage for more information
# about the syntax of this string.
#
# Note: this configuration applies only to <= TLSv1.2.
#
# tls-ciphers DEFAULT:!MEDIUM

# Configure allowed TLSv1.3 ciphersuites.  See the ciphers(1ssl) manpage for more
# information about the syntax of this string, and specifically for TLSv1.3
# ciphersuites.
#
# tls-ciphersuites TLS_CHACHA20_POLY1305_SHA256

# When choosing a cipher, use the server's preference instead of the client
# preference. By default, the server follows the client's preference.
#
# tls-prefer-server-ciphers yes

# By default, TLS session caching is enabled to allow faster and less expensive
# reconnections by clients that support it. Use the following directive to disable
# caching.
#
# tls-session-caching no

# Change the default number of TLS sessions cached. A zero value sets the cache
# to unlimited size. The default size is 20480.
#
# tls-session-cache-size 5000

# Change the default timeout of cached TLS sessions. The default timeout is 300
# seconds.
#
# tls-session-cache-timeout 60

################################# GENERAL #####################################

# By default Redis does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis.pid when daemonized.
# When Redis is supervised by upstart or systemd, this parameter has no impact.
#houtaiqidong
#守护进程 yes
daemonize yes

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#                        requires "expect stop" in your upstart job config
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#                        on startup, and updating Redis status on a regular
#                        basis.
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous pings back to your supervisor.
#
# The default is "no". To run under upstart/systemd, you can simply uncomment
# the line below:
#
# supervised auto

# If a pid file is specified, Redis writes it where specified at startup
# and removes it at exit.
#
# When the server runs non daemonized, no pid file is created if none is
# specified in the configuration. When the server is daemonized, the pid file
# is used even if not specified, defaulting to "/var/run/redis.pid".
#
# Creating a pid file is best effort: if Redis is not able to create it
# nothing bad happens, the server will start and run normally.
#
# Note that on modern Linux systems "/run/redis.pid" is more conforming
# and should be used instead.
# 指定pid文件 守护进程
pidfile /var/run/redis_6379.pid

# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably) 生产环境
# warning (only very important / critical messages are logged)
# 日志级别
loglevel notice

# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
# 文件名
logfile ""

# To enable logging to the system logger, just set 'syslog-enabled' to yes,
# and optionally update the other syslog parameters to suit your needs.
# syslog-enabled no

# Specify the syslog identity.
# syslog-ident redis

# Specify the syslog facility. Must be USER or between LOCAL0-LOCAL7.
# syslog-facility local0

# To disable the built in crash log, which will possibly produce cleaner core
# dumps when they are needed, uncomment the following:
#
# crash-log-enabled no

# To disable the fast memory check that's run as part of the crash log, which
# will possibly let redis terminate sooner, uncomment the following:
#
# crash-memcheck-enabled no

# Set the number of databases. The default database is DB 0, you can select
# a different one on a per-connection basis using SELECT <dbid> where
# dbid is a number between 0 and 'databases'-1
# 数据库数量
databases 16

# By default Redis shows an ASCII art logo only when started to log to the
# standard output and if the standard output is a TTY and syslog logging is
# disabled. Basically this means that normally a logo is displayed only in
# interactive sessions.
#
# However it is possible to force the pre-4.0 behavior and always show a
# ASCII art logo in startup logs by setting the following option to yes.
# 是否显示log
always-show-logo no

# By default, Redis modifies the process title (as seen in 'top' and 'ps') to
# provide some runtime information. It is possible to disable this and leave
# the process name as executed by setting the following to no.
# 快照配置
set-proc-title yes

# When changing the process title, Redis uses the following template to construct
# the modified title.
#
# Template variables are specified in curly brackets. The following variables are
# supported:
#
# {title}           Name of process as executed if parent, or type of child process.
# {listen-addr}     Bind address or '*' followed by TCP or TLS port listening on, or
#                   Unix socket if only that's available.
# {server-mode}     Special mode, i.e. "[sentinel]" or "[cluster]".
# {port}            TCP port listening on, or 0.
# {tls-port}        TLS port listening on, or 0.
# {unixsocket}      Unix domain socket listening on, or "".
# {config-file}     Name of configuration file used.
#
proc-title-template "{title} {listen-addr} {server-mode}"

################################ SNAPSHOTTING  ################################

# Save the DB to disk.
#
# save <seconds> <changes>
#
# Redis will save the DB if both the given number of seconds and the given
# number of write operations against the DB occurred.
#
# Snapshotting can be completely disabled with a single empty string argument
# as in following example:
#
# save ""
#
# Unless specified otherwise, by default Redis will save the DB:
#   * After 3600 seconds (an hour) if at least 1 key changed
#   * After 300 seconds (5 minutes) if at least 100 keys changed
#   * After 60 seconds if at least 10000 keys changed
#
# You can set these explicitly by uncommenting the three following lines.
# 快照次数配置 一个小时 一个key进行改变就进行配置
# save 3600 1
# save 300 100
# save 60 10000

# By default Redis will stop accepting writes if RDB snapshots are enabled
# (at least one save point) and the latest background save failed.
# This will make the user aware (in a hard way) that data is not persisting
# on disk properly, otherwise chances are that no one will notice and some
# disaster will happen.
#
# If the background saving process will start working again Redis will
# automatically allow writes again.
#
# However if you have setup your proper monitoring of the Redis server
# and persistence, you may want to disable this feature so that Redis will
# continue to work as usual even if there are problems with disk,
# permissions, and so forth.
# 持久化出现错误的时候是否停止写入
stop-writes-on-bgsave-error yes

# Compress string objects using LZF when dump .rdb databases?
# By default compression is enabled as it's almost always a win.
# If you want to save some CPU in the saving child set it to 'no' but
# the dataset will likely be bigger if you have compressible values or keys.
# 是否压缩文件
rdbcompression yes

# Since version 5 of RDB a CRC64 checksum is placed at the end of the file.
# This makes the format more resistant to corruption but there is a performance
# hit to pay (around 10%) when saving and loading RDB files, so you can disable it
# for maximum performances.
#
# RDB files created with checksum disabled have a checksum of zero that will
# tell the loading code to skip the check.
#rdb是否做校验
rdbchecksum yes

# Enables or disables full sanitation checks for ziplist and listpack etc when
# loading an RDB or RESTORE payload. This reduces the chances of a assertion or
# crash later on while processing commands.
# Options:
#   no         - Never perform full sanitation
#   yes        - Always perform full sanitation
#   clients    - Perform full sanitation only for user connections.
#                Excludes: RDB files, RESTORE commands received from the master
#                connection, and client connections which have the
#                skip-sanitize-payload ACL flag.
# The default should be 'clients' but since it currently affects cluster
# resharding via MIGRATE, it is temporarily set to 'no' by default.
#
# sanitize-dump-payload no

# The filename where to dump the DB
#
dbfilename dump.rdb

# Remove RDB files used by replication in instances without persistence
# enabled. By default this option is disabled, however there are environments
# where for regulations or other security concerns, RDB files persisted on
# disk by masters in order to feed replicas, or stored on disk by replicas
# in order to load them for the initial synchronization, should be deleted
# ASAP. Note that this option ONLY WORKS in instances that have both AOF
# and RDB persistence disabled, otherwise is completely ignored.
#
# An alternative (and sometimes better) way to obtain the same effect is
# to use diskless replication on both master and replicas instances. However
# in the case of replicas, diskless is not always an option.
rdb-del-sync-files no

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
# 目录位置
dir ./

################################# REPLICATION #################################

# Master-Replica replication. Use replicaof to make a Redis instance a copy of
# another Redis server. A few things to understand ASAP about Redis replication.
#
#   +------------------+      +---------------+
#   |      Master      | ---> |    Replica    |
#   | (receive writes) |      |  (exact copy) |
#   +------------------+      +---------------+
#
# 1) Redis replication is asynchronous, but you can configure a master to
#    stop accepting writes if it appears to be not connected with at least
#    a given number of replicas.
# 2) Redis replicas are able to perform a partial resynchronization with the
#    master if the replication link is lost for a relatively small amount of
#    time. You may want to configure the replication backlog size (see the next
#    sections of this file) with a sensible value depending on your needs.
# 3) Replication is automatic and does not need user intervention. After a
#    network partition replicas automatically try to reconnect to masters
#    and resynchronize with them.
#
# replicaof <masterip> <masterport>

# If the master is password protected (using the "requirepass" configuration
# directive below) it is possible to tell the replica to authenticate before
# starting the replication synchronization process, otherwise the master will
# refuse the replica request.
#
# masterauth <master-password>
#
# However this is not enough if you are using Redis ACLs (for Redis version
# 6 or greater), and the default user is not capable of running the PSYNC
# command and/or other commands needed for replication. In this case it's
# better to configure a special user to use with replication, and specify the
# masteruser configuration as such:
#
# masteruser <username>
#
# When masteruser is specified, the replica will authenticate against its
# master using the new AUTH form: AUTH <username> <password>.

# When a replica loses its connection with the master, or when the replication
# is still in progress, the replica can act in two different ways:
#
# 1) if replica-serve-stale-data is set to 'yes' (the default) the replica will
#    still reply to client requests, possibly with out of date data, or the
#    data set may just be empty if this is the first synchronization.
#
# 2) If replica-serve-stale-data is set to 'no' the replica will reply with
#    an error "SYNC with master in progress" to all commands except:
#    INFO, REPLICAOF, AUTH, PING, SHUTDOWN, REPLCONF, ROLE, CONFIG, SUBSCRIBE,
#    UNSUBSCRIBE, PSUBSCRIBE, PUNSUBSCRIBE, PUBLISH, PUBSUB, COMMAND, POST,
#    HOST and LATENCY.
#
replica-serve-stale-data yes

# You can configure a replica instance to accept writes or not. Writing against
# a replica instance may be useful to store some ephemeral data (because data
# written on a replica will be easily deleted after resync with the master) but
# may also cause problems if clients are writing to it because of a
# misconfiguration.
#
# Since Redis 2.6 by default replicas are read-only.
#
# Note: read only replicas are not designed to be exposed to untrusted clients
# on the internet. It's just a protection layer against misuse of the instance.
# Still a read only replica exports by default all the administrative commands
# such as CONFIG, DEBUG, and so forth. To a limited extent you can improve
# security of read only replicas using 'rename-command' to shadow all the
# administrative / dangerous commands.
replica-read-only yes

# Replication SYNC strategy: disk or socket.
#
# New replicas and reconnecting replicas that are not able to continue the
# replication process just receiving differences, need to do what is called a
# "full synchronization". An RDB file is transmitted from the master to the
# replicas.
#
# The transmission can happen in two different ways:
#
# 1) Disk-backed: The Redis master creates a new process that writes the RDB
#                 file on disk. Later the file is transferred by the parent
#                 process to the replicas incrementally.
# 2) Diskless: The Redis master creates a new process that directly writes the
#              RDB file to replica sockets, without touching the disk at all.
#
# With disk-backed replication, while the RDB file is generated, more replicas
# can be queued and served with the RDB file as soon as the current child
# producing the RDB file finishes its work. With diskless replication instead
# once the transfer starts, new replicas arriving will be queued and a new
# transfer will start when the current one terminates.
#
# When diskless replication is used, the master waits a configurable amount of
# time (in seconds) before starting the transfer in the hope that multiple
# replicas will arrive and the transfer can be parallelized.
#
# With slow disks and fast (large bandwidth) networks, diskless replication
# works better.
repl-diskless-sync no

# When diskless replication is enabled, it is possible to configure the delay
# the server waits in order to spawn the child that transfers the RDB via socket
# to the replicas.
#
# This is important since once the transfer starts, it is not possible to serve
# new replicas arriving, that will be queued for the next RDB transfer, so the
# server waits a delay in order to let more replicas arrive.
#
# The delay is specified in seconds, and by default is 5 seconds. To disable
# it entirely just set it to 0 seconds and the transfer will start ASAP.
repl-diskless-sync-delay 5

# -----------------------------------------------------------------------------
# WARNING: RDB diskless load is experimental. Since in this setup the replica
# does not immediately store an RDB on disk, it may cause data loss during
# failovers. RDB diskless load + Redis modules not handling I/O reads may also
# cause Redis to abort in case of I/O errors during the initial synchronization
# stage with the master. Use only if you know what you are doing.
# -----------------------------------------------------------------------------
#
# Replica can load the RDB it reads from the replication link directly from the
# socket, or store the RDB to a file and read that file after it was completely
# received from the master.
#
# In many cases the disk is slower than the network, and storing and loading
# the RDB file may increase replication time (and even increase the master's
# Copy on Write memory and salve buffers).
# However, parsing the RDB file directly from the socket may mean that we have
# to flush the contents of the current database before the full rdb was
# received. For this reason we have the following options:
#
# "disabled"    - Don't use diskless load (store the rdb file to the disk first)
# "on-empty-db" - Use diskless load only when it is completely safe.
# "swapdb"      - Keep a copy of the current db contents in RAM while parsing
#                 the data directly from the socket. note that this requires
#                 sufficient memory, if you don't have it, you risk an OOM kill.
repl-diskless-load disabled

# Replicas send PINGs to server in a predefined interval. It's possible to
# change this interval with the repl_ping_replica_period option. The default
# value is 10 seconds.
#
# repl-ping-replica-period 10

# The following option sets the replication timeout for:
#
# 1) Bulk transfer I/O during SYNC, from the point of view of replica.
# 2) Master timeout from the point of view of replicas (data, pings).
# 3) Replica timeout from the point of view of masters (REPLCONF ACK pings).
#
# It is important to make sure that this value is greater than the value
# specified for repl-ping-replica-period otherwise a timeout will be detected
# every time there is low traffic between the master and the replica. The default
# value is 60 seconds.
#
# repl-timeout 60

# Disable TCP_NODELAY on the replica socket after SYNC?
#
# If you select "yes" Redis will use a smaller number of TCP packets and
# less bandwidth to send data to replicas. But this can add a delay for
# the data to appear on the replica side, up to 40 milliseconds with
# Linux kernels using a default configuration.
#
# If you select "no" the delay for data to appear on the replica side will
# be reduced but more bandwidth will be used for replication.
#
# By default we optimize for low latency, but in very high traffic conditions
# or when the master and replicas are many hops away, turning this to "yes" may
# be a good idea.
repl-disable-tcp-nodelay no

# Set the replication backlog size. The backlog is a buffer that accumulates
# replica data when replicas are disconnected for some time, so that when a
# replica wants to reconnect again, often a full resync is not needed, but a
# partial resync is enough, just passing the portion of data the replica
# missed while disconnected.
#
# The bigger the replication backlog, the longer the replica can endure the
# disconnect and later be able to perform a partial resynchronization.
#
# The backlog is only allocated if there is at least one replica connected.
#
# repl-backlog-size 1mb

# After a master has no connected replicas for some time, the backlog will be
# freed. The following option configures the amount of seconds that need to
# elapse, starting from the time the last replica disconnected, for the backlog
# buffer to be freed.
#
# Note that replicas never free the backlog for timeout, since they may be
# promoted to masters later, and should be able to correctly "partially
# resynchronize" with other replicas: hence they should always accumulate backlog.
#
# A value of 0 means to never release the backlog.
#
# repl-backlog-ttl 3600

# The replica priority is an integer number published by Redis in the INFO
# output. It is used by Redis Sentinel in order to select a replica to promote
# into a master if the master is no longer working correctly.
#
# A replica with a low priority number is considered better for promotion, so
# for instance if there are three replicas with priority 10, 100, 25 Sentinel
# will pick the one with priority 10, that is the lowest.
#
# However a special priority of 0 marks the replica as not able to perform the
# role of master, so a replica with priority of 0 will never be selected by
# Redis Sentinel for promotion.
#
# By default the priority is 100.
replica-priority 100

# It is possible for a master to stop accepting writes if there are less than
# N replicas connected, having a lag less or equal than M seconds.
#
# The N replicas need to be in "online" state.
#
# The lag in seconds, that must be <= the specified value, is calculated from
# the last ping received from the replica, that is usually sent every second.
#
# This option does not GUARANTEE that N replicas will accept the write, but
# will limit the window of exposure for lost writes in case not enough replicas
# are available, to the specified number of seconds.
#
# For example to require at least 3 replicas with a lag <= 10 seconds use:
#
# min-replicas-to-write 3
# min-replicas-max-lag 10
#
# Setting one or the other to 0 disables the feature.
#
# By default min-replicas-to-write is set to 0 (feature disabled) and
# min-replicas-max-lag is set to 10.

# A Redis master is able to list the address and port of the attached
# replicas in different ways. For example the "INFO replication" section
# offers this information, which is used, among other tools, by
# Redis Sentinel in order to discover replica instances.
# Another place where this info is available is in the output of the
# "ROLE" command of a master.
#
# The listed IP address and port normally reported by a replica is
# obtained in the following way:
#
#   IP: The address is auto detected by checking the peer address
#   of the socket used by the replica to connect with the master.
#
#   Port: The port is communicated by the replica during the replication
#   handshake, and is normally the port that the replica is using to
#   listen for connections.
#
# However when port forwarding or Network Address Translation (NAT) is
# used, the replica may actually be reachable via different IP and port
# pairs. The following two options can be used by a replica in order to
# report to its master a specific set of IP and port, so that both INFO
# and ROLE will report those values.
#
# There is no need to use both the options if you need to override just
# the port or the IP address.
#
# replica-announce-ip 5.5.5.5
# replica-announce-port 1234

############################### KEYS TRACKING #################################

# Redis implements server assisted support for client side caching of values.
# This is implemented using an invalidation table that remembers, using
# a radix key indexed by key name, what clients have which keys. In turn
# this is used in order to send invalidation messages to clients. Please
# check this page to understand more about the feature:
#
#   https://redis.io/topics/client-side-caching
#
# When tracking is enabled for a client, all the read only queries are assumed
# to be cached: this will force Redis to store information in the invalidation
# table. When keys are modified, such information is flushed away, and
# invalidation messages are sent to the clients. However if the workload is
# heavily dominated by reads, Redis could use more and more memory in order
# to track the keys fetched by many clients.
#
# For this reason it is possible to configure a maximum fill value for the
# invalidation table. By default it is set to 1M of keys, and once this limit
# is reached, Redis will start to evict keys in the invalidation table
# even if they were not modified, just to reclaim memory: this will in turn
# force the clients to invalidate the cached values. Basically the table
# maximum size is a trade off between the memory you want to spend server
# side to track information about who cached what, and the ability of clients
# to retain cached objects in memory.
#
# If you set the value to 0, it means there are no limits, and Redis will
# retain as many keys as needed in the invalidation table.
# In the "stats" INFO section, you can find information about the number of
# keys in the invalidation table at every given moment.
#
# Note: when key tracking is used in broadcasting mode, no memory is used
# in the server side so this setting is useless.
#
# tracking-table-max-keys 1000000

################################## SECURITY ###################################

# Warning: since Redis is pretty fast, an outside user can try up to
# 1 million passwords per second against a modern box. This means that you
# should use very strong passwords, otherwise they will be very easy to break.
# Note that because the password is really a shared secret between the client
# and the server, and should not be memorized by any human, the password
# can be easily a long string from /dev/urandom or whatever, so by using a
# long and unguessable password no brute force attack will be possible.

# Redis ACL users are defined in the following format:
#
#   user <username> ... acl rules ...
#
# For example:
#
#   user worker +@list +@connection ~jobs:* on >ffa9203c493aa99
#
# The special username "default" is used for new connections. If this user
# has the "nopass" rule, then new connections will be immediately authenticated
# as the "default" user without the need of any password provided via the
# AUTH command. Otherwise if the "default" user is not flagged with "nopass"
# the connections will start in not authenticated state, and will require
# AUTH (or the HELLO command AUTH option) in order to be authenticated and
# start to work.
#
# The ACL rules that describe what a user can do are the following:
#
#  on           Enable the user: it is possible to authenticate as this user.
#  off          Disable the user: it's no longer possible to authenticate
#               with this user, however the already authenticated connections
#               will still work.
#  skip-sanitize-payload    RESTORE dump-payload sanitation is skipped.
#  sanitize-payload         RESTORE dump-payload is sanitized (default).
#  +<command>   Allow the execution of that command
#  -<command>   Disallow the execution of that command
#  +@<category> Allow the execution of all the commands in such category
#               with valid categories are like @admin, @set, @sortedset, ...
#               and so forth, see the full list in the server.c file where
#               the Redis command table is described and defined.
#               The special category @all means all the commands, but currently
#               present in the server, and that will be loaded in the future
#               via modules.
#  +<command>|subcommand    Allow a specific subcommand of an otherwise
#                           disabled command. Note that this form is not
#                           allowed as negative like -DEBUG|SEGFAULT, but
#                           only additive starting with "+".
#  allcommands  Alias for +@all. Note that it implies the ability to execute
#               all the future commands loaded via the modules system.
#  nocommands   Alias for -@all.
#  ~<pattern>   Add a pattern of keys that can be mentioned as part of
#               commands. For instance ~* allows all the keys. The pattern
#               is a glob-style pattern like the one of KEYS.
#               It is possible to specify multiple patterns.
#  allkeys      Alias for ~*
#  resetkeys    Flush the list of allowed keys patterns.
#  &<pattern>   Add a glob-style pattern of Pub/Sub channels that can be
#               accessed by the user. It is possible to specify multiple channel
#               patterns.
#  allchannels  Alias for &*
#  resetchannels            Flush the list of allowed channel patterns.
#  ><password>  Add this password to the list of valid password for the user.
#               For example >mypass will add "mypass" to the list.
#               This directive clears the "nopass" flag (see later).
#  <<password>  Remove this password from the list of valid passwords.
#  nopass       All the set passwords of the user are removed, and the user
#               is flagged as requiring no password: it means that every
#               password will work against this user. If this directive is
#               used for the default user, every new connection will be
#               immediately authenticated with the default user without
#               any explicit AUTH command required. Note that the "resetpass"
#               directive will clear this condition.
#  resetpass    Flush the list of allowed passwords. Moreover removes the
#               "nopass" status. After "resetpass" the user has no associated
#               passwords and there is no way to authenticate without adding
#               some password (or setting it as "nopass" later).
#  reset        Performs the following actions: resetpass, resetkeys, off,
#               -@all. The user returns to the same state it has immediately
#               after its creation.
#
# ACL rules can be specified in any order: for instance you can start with
# passwords, then flags, or key patterns. However note that the additive
# and subtractive rules will CHANGE MEANING depending on the ordering.
# For instance see the following example:
#
#   user alice on +@all -DEBUG ~* >somepassword
#
# This will allow "alice" to use all the commands with the exception of the
# DEBUG command, since +@all added all the commands to the set of the commands
# alice can use, and later DEBUG was removed. However if we invert the order
# of two ACL rules the result will be different:
#
#   user alice on -DEBUG +@all ~* >somepassword
#
# Now DEBUG was removed when alice had yet no commands in the set of allowed
# commands, later all the commands are added, so the user will be able to
# execute everything.
#
# Basically ACL rules are processed left-to-right.
#
# For more information about ACL configuration please refer to
# the Redis web site at https://redis.io/topics/acl

# ACL LOG
#
# The ACL Log tracks failed commands and authentication events associated
# with ACLs. The ACL Log is useful to troubleshoot failed commands blocked 
# by ACLs. The ACL Log is stored in memory. You can reclaim memory with 
# ACL LOG RESET. Define the maximum entry length of the ACL Log below.
acllog-max-len 128

# Using an external ACL file
#
# Instead of configuring users here in this file, it is possible to use
# a stand-alone file just listing users. The two methods cannot be mixed:
# if you configure users here and at the same time you activate the external
# ACL file, the server will refuse to start.
#
# The format of the external ACL user file is exactly the same as the
# format that is used inside redis.conf to describe users.
#
# aclfile /etc/redis/users.acl

# IMPORTANT NOTE: starting with Redis 6 " requirepass" is just a compatibility
# layer on top of the new ACL system. The option effect will be just setting
# the password for the default user. Clients will still authenticate using
# AUTH <password> as usually, or more explicitly with AUTH default <password>
# if they follow the new protocol: both will work.
#
# The requirepass is not compatable with aclfile option and the ACL LOAD
# command, these will cause requirepass to be ignored.
#设置密码

# requirepass foobared
requirepass JackGao5210


# New users are initialized with restrictive permissions by default, via the
# equivalent of this ACL rule 'off resetkeys -@all'. Starting with Redis 6.2, it
# is possible to manage access to Pub/Sub channels with ACL rules as well. The
# default Pub/Sub channels permission if new users is controlled by the 
# acl-pubsub-default configuration directive, which accepts one of these values:
#
# allchannels: grants access to all Pub/Sub channels
# resetchannels: revokes access to all Pub/Sub channels
#
# To ensure backward compatibility while upgrading Redis 6.0, acl-pubsub-default
# defaults to the 'allchannels' permission.
#
# Future compatibility note: it is very likely that in a future version of Redis
# the directive's default of 'allchannels' will be changed to 'resetchannels' in
# order to provide better out-of-the-box Pub/Sub security. Therefore, it is
# recommended that you explicitly define Pub/Sub permissions for all users
# rather then rely on implicit default values. Once you've set explicit
# Pub/Sub for all exisitn users, you should uncomment the following line.
#
# acl-pubsub-default resetchannels

# Command renaming (DEPRECATED).
#
# ------------------------------------------------------------------------
# WARNING: avoid using this option if possible. Instead use ACLs to remove
# commands from the default user, and put them only in some admin user you
# create for administrative purposes.
# ------------------------------------------------------------------------
#
# It is possible to change the name of dangerous commands in a shared
# environment. For instance the CONFIG command may be renamed into something
# hard to guess so that it will still be available for internal-use tools
# but not available for general clients.
#
# Example:
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
#
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
# rename-command CONFIG ""
#
# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to replicas may cause problems.

################################### CLIENTS ####################################

# Set the max number of connected clients at the same time. By default
# this limit is set to 10000 clients, however if the Redis server is not
# able to configure the process file limit to allow for the specified limit
# the max number of allowed clients is set to the current file limit
# minus 32 (as Redis reserves a few file descriptors for internal uses).
#
# Once the limit is reached Redis will close all the new connections sending
# an error 'max number of clients reached'.
#
# IMPORTANT: When Redis Cluster is used, the max number of connections is also
# shared with the cluster bus: every node in the cluster will use two
# connections, one incoming and another outgoing. It is important to size the
# limit accordingly in case of very large clusters.
# 客户端连接数
# maxclients 10000

############################## MEMORY MANAGEMENT ################################

# Set a memory usage limit to the specified amount of bytes.
# When the memory limit is reached Redis will try to remove keys
# according to the eviction policy selected (see maxmemory-policy).
#
# If Redis can't remove keys according to the policy, or if the policy is
# set to 'noeviction', Redis will start to reply with errors to commands
# that would use more memory, like SET, LPUSH, and so on, and will continue
# to reply to read-only commands like GET.
#
# This option is usually useful when using Redis as an LRU or LFU cache, or to
# set a hard memory limit for an instance (using the 'noeviction' policy).
#
# WARNING: If you have replicas attached to an instance with maxmemory on,
# the size of the output buffers needed to feed the replicas are subtracted
# from the used memory count, so that network problems / resyncs will
# not trigger a loop where keys are evicted, and in turn the output
# buffer of replicas is full with DELs of keys evicted triggering the deletion
# of more keys, and so forth until the database is completely emptied.
#
# In short... if you have replicas attached it is suggested that you set a lower
# limit for maxmemory so that there is some free RAM on the system for replica
# output buffers (but this is not needed if the policy is 'noeviction').
# 最大内存
# maxmemory <bytes>

# MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
# is reached. You can select one from the following behaviors:
#
# volatile-lru -> Evict using approximated LRU, only keys with an expire set.
# allkeys-lru -> Evict any key using approximated LRU.
# volatile-lfu -> Evict using approximated LFU, only keys with an expire set.
# allkeys-lfu -> Evict any key using approximated LFU.
# volatile-random -> Remove a random key having an expire set.
# allkeys-random -> Remove a random key, any key.
# volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
# noeviction -> Don't evict anything, just return an error on write operations.
#
# LRU means Least Recently Used
# LFU means Least Frequently Used
#
# Both LRU, LFU and volatile-ttl are implemented using approximated
# randomized algorithms.
#
# Note: with any of the above policies, when there are no suitable keys for
# eviction, Redis will return an error on write operations that require
# more memory. These are usually commands that create new keys, add data or
# modify existing keys. A few examples are: SET, INCR, HSET, LPUSH, SUNIONSTORE,
# SORT (due to the STORE argument), and EXEC (if the transaction includes any
# command that requires memory).
#
# The default is:
# 内存达到上限之后的处理策略方式
# maxmemory-policy noeviction

# LRU, LFU and minimal TTL algorithms are not precise algorithms but approximated
# algorithms (in order to save memory), so you can tune it for speed or
# accuracy. By default Redis will check five keys and pick the one that was
# used least recently, you can change the sample size using the following
# configuration directive.
#
# The default of 5 produces good enough results. 10 Approximates very closely
# true LRU but costs more CPU. 3 is faster but not very accurate.
#
# maxmemory-samples 5

# Eviction processing is designed to function well with the default setting.
# If there is an unusually large amount of write traffic, this value may need to
# be increased.  Decreasing this value may reduce latency at the risk of 
# eviction processing effectiveness
#   0 = minimum latency, 10 = default, 100 = process without regard to latency
#
# maxmemory-eviction-tenacity 10

# Starting from Redis 5, by default a replica will ignore its maxmemory setting
# (unless it is promoted to master after a failover or manually). It means
# that the eviction of keys will be just handled by the master, sending the
# DEL commands to the replica as keys evict in the master side.
#
# This behavior ensures that masters and replicas stay consistent, and is usually
# what you want, however if your replica is writable, or you want the replica
# to have a different memory setting, and you are sure all the writes performed
# to the replica are idempotent, then you may change this default (but be sure
# to understand what you are doing).
#
# Note that since the replica by default does not evict, it may end using more
# memory than the one set via maxmemory (there are certain buffers that may
# be larger on the replica, or data structures may sometimes take more memory
# and so forth). So make sure you monitor your replicas and make sure they
# have enough memory to never hit a real out-of-memory condition before the
# master hits the configured maxmemory setting.
#
# replica-ignore-maxmemory yes

# Redis reclaims expired keys in two ways: upon access when those keys are
# found to be expired, and also in background, in what is called the
# "active expire key". The key space is slowly and interactively scanned
# looking for expired keys to reclaim, so that it is possible to free memory
# of keys that are expired and will never be accessed again in a short time.
#
# The default effort of the expire cycle will try to avoid having more than
# ten percent of expired keys still in memory, and will try to avoid consuming
# more than 25% of total memory and to add latency to the system. However
# it is possible to increase the expire "effort" that is normally set to
# "1", to a greater value, up to the value "10". At its maximum value the
# system will use more CPU, longer cycles (and technically may introduce
# more latency), and will tolerate less already expired keys still present
# in the system. It's a tradeoff between memory, CPU and latency.
#
# active-expire-effort 1

############################# LAZY FREEING ####################################

# Redis has two primitives to delete keys. One is called DEL and is a blocking
# deletion of the object. It means that the server stops processing new commands
# in order to reclaim all the memory associated with an object in a synchronous
# way. If the key deleted is associated with a small object, the time needed
# in order to execute the DEL command is very small and comparable to most other
# O(1) or O(log_N) commands in Redis. However if the key is associated with an
# aggregated value containing millions of elements, the server can block for
# a long time (even seconds) in order to complete the operation.
#
# For the above reasons Redis also offers non blocking deletion primitives
# such as UNLINK (non blocking DEL) and the ASYNC option of FLUSHALL and
# FLUSHDB commands, in order to reclaim memory in background. Those commands
# are executed in constant time. Another thread will incrementally free the
# object in the background as fast as possible.
#
# DEL, UNLINK and ASYNC option of FLUSHALL and FLUSHDB are user-controlled.
# It's up to the design of the application to understand when it is a good
# idea to use one or the other. However the Redis server sometimes has to
# delete keys or flush the whole database as a side effect of other operations.
# Specifically Redis deletes objects independently of a user call in the
# following scenarios:
#
# 1) On eviction, because of the maxmemory and maxmemory policy configurations,
#    in order to make room for new data, without going over the specified
#    memory limit.
# 2) Because of expire: when a key with an associated time to live (see the
#    EXPIRE command) must be deleted from memory.
# 3) Because of a side effect of a command that stores data on a key that may
#    already exist. For example the RENAME command may delete the old key
#    content when it is replaced with another one. Similarly SUNIONSTORE
#    or SORT with STORE option may delete existing keys. The SET command
#    itself removes any old content of the specified key in order to replace
#    it with the specified string.
# 4) During replication, when a replica performs a full resynchronization with
#    its master, the content of the whole database is removed in order to
#    load the RDB file just transferred.
#
# In all the above cases the default is to delete objects in a blocking way,
# like if DEL was called. However you can configure each case specifically
# in order to instead release memory in a non-blocking way like if UNLINK
# was called, using the following configuration directives.

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

# It is also possible, for the case when to replace the user code DEL calls
# with UNLINK calls is not easy, to modify the default behavior of the DEL
# command to act exactly like UNLINK, using the following configuration
# directive:

lazyfree-lazy-user-del no

# FLUSHDB, FLUSHALL, and SCRIPT FLUSH support both asynchronous and synchronous
# deletion, which can be controlled by passing the [SYNC|ASYNC] flags into the
# commands. When neither flag is passed, this directive will be used to determine
# if the data should be deleted asynchronously.

lazyfree-lazy-user-flush no

################################ THREADED I/O #################################

# Redis is mostly single threaded, however there are certain threaded
# operations such as UNLINK, slow I/O accesses and other things that are
# performed on side threads.
#
# Now it is also possible to handle Redis clients socket reads and writes
# in different I/O threads. Since especially writing is so slow, normally
# Redis users use pipelining in order to speed up the Redis performances per
# core, and spawn multiple instances in order to scale more. Using I/O
# threads it is possible to easily speedup two times Redis without resorting
# to pipelining nor sharding of the instance.
#
# By default threading is disabled, we suggest enabling it only in machines
# that have at least 4 or more cores, leaving at least one spare core.
# Using more than 8 threads is unlikely to help much. We also recommend using
# threaded I/O only if you actually have performance problems, with Redis
# instances being able to use a quite big percentage of CPU time, otherwise
# there is no point in using this feature.
#
# So for instance if you have a four cores boxes, try to use 2 or 3 I/O
# threads, if you have a 8 cores, try to use 6 threads. In order to
# enable I/O threads use the following configuration directive:
#
# io-threads 4
#
# Setting io-threads to 1 will just use the main thread as usual.
# When I/O threads are enabled, we only use threads for writes, that is
# to thread the write(2) syscall and transfer the client buffers to the
# socket. However it is also possible to enable threading of reads and
# protocol parsing using the following configuration directive, by setting
# it to yes:
#
# io-threads-do-reads no
#
# Usually threading reads doesn't help much.
#
# NOTE 1: This configuration directive cannot be changed at runtime via
# CONFIG SET. Aso this feature currently does not work when SSL is
# enabled.
#
# NOTE 2: If you want to test the Redis speedup using redis-benchmark, make
# sure you also run the benchmark itself in threaded mode, using the
# --threads option to match the number of Redis threads, otherwise you'll not
# be able to notice the improvements.

############################ KERNEL OOM CONTROL ##############################

# On Linux, it is possible to hint the kernel OOM killer on what processes
# should be killed first when out of memory.
#
# Enabling this feature makes Redis actively control the oom_score_adj value
# for all its processes, depending on their role. The default scores will
# attempt to have background child processes killed before all others, and
# replicas killed before masters.
#
# Redis supports three options:
#
# no:       Don't make changes to oom-score-adj (default).
# yes:      Alias to "relative" see below.
# absolute: Values in oom-score-adj-values are written as is to the kernel.
# relative: Values are used relative to the initial value of oom_score_adj when
#           the server starts and are then clamped to a range of -1000 to 1000.
#           Because typically the initial value is 0, they will often match the
#           absolute values.
oom-score-adj no

# When oom-score-adj is used, this directive controls the specific values used
# for master, replica and background child processes. Values range -2000 to
# 2000 (higher means more likely to be killed).
#
# Unprivileged processes (not root, and without CAP_SYS_RESOURCE capabilities)
# can freely increase their value, but not decrease it below its initial
# settings. This means that setting oom-score-adj to "relative" and setting the
# oom-score-adj-values to positive values will always succeed.
oom-score-adj-values 0 200 800


#################### KERNEL transparent hugepage CONTROL ######################

# Usually the kernel Transparent Huge Pages control is set to "madvise" or
# or "never" by default (/sys/kernel/mm/transparent_hugepage/enabled), in which
# case this config has no effect. On systems in which it is set to "always",
# redis will attempt to disable it specifically for the redis process in order
# to avoid latency problems specifically with fork(2) and CoW.
# If for some reason you prefer to keep it enabled, you can set this config to
# "no" and the kernel global to "always".

disable-thp yes

############################## APPEND ONLY MODE ###############################

# By default Redis asynchronously dumps the dataset on disk. This mode is
# good enough in many applications, but an issue with the Redis process or
# a power outage may result into a few minutes of writes lost (depending on
# the configured save points).
#
# The Append Only File is an alternative persistence mode that provides
# much better durability. For instance using the default data fsync policy
# (see later in the config file) Redis can lose just one second of writes in a
# dramatic event like a server power outage, or a single write if something
# wrong with the Redis process itself happens, but the operating system is
# still running correctly.
#
# AOF and RDB persistence can be enabled at the same time without problems.
# If the AOF is enabled on startup Redis will load the AOF, that is the file
# with the better durability guarantees.
#
# Please check http://redis.io/topics/persistence for more information.
#AOF 模式配置
appendonly no

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof"

# The fsync() call tells the Operating System to actually write data on disk
# instead of waiting for more data in the output buffer. Some OS will really flush
# data on disk, some other OS will just try to do it ASAP.
#
# Redis supports three different modes:
#
# no: don't fsync, just let the OS flush the data when it wants. Faster.
# always: fsync after every write to the append only log. Slow, Safest.
# everysec: fsync only one time every second. Compromise.
#
# The default is "everysec", as that's usually the right compromise between
# speed and data safety. It's up to you to understand if you can relax this to
# "no" that will let the operating system flush the output buffer when
# it wants, for better performances (but if you can live with the idea of
# some data loss consider the default persistence mode that's snapshotting),
# or on the contrary, use "always" that's very slow but a bit safer than
# everysec.
#
# More details please check the following article:
# http://antirez.com/post/redis-persistence-demystified.html
#
# If unsure, use "everysec".

# appendfsync always
# 每秒同步一次数据
appendfsync everysec
# appendfsync no

# When the AOF fsync policy is set to always or everysec, and a background
# saving process (a background save or AOF log background rewriting) is
# performing a lot of I/O against the disk, in some Linux configurations
# Redis may block too long on the fsync() call. Note that there is no fix for
# this currently, as even performing fsync in a different thread will block
# our synchronous write(2) call.
#
# In order to mitigate this problem it's possible to use the following option
# that will prevent fsync() from being called in the main process while a
# BGSAVE or BGREWRITEAOF is in progress.
#
# This means that while another child is saving, the durability of Redis is
# the same as "appendfsync none". In practical terms, this means that it is
# possible to lose up to 30 seconds of log in the worst scenario (with the
# default Linux settings).
#
# If you have latency problems turn this to "yes". Otherwise leave it as
# "no" that is the safest pick from the point of view of durability.

no-appendfsync-on-rewrite no

# Automatic rewrite of the append only file.
# Redis is able to automatically rewrite the log file implicitly calling
# BGREWRITEAOF when the AOF log size grows by the specified percentage.
#
# This is how it works: Redis remembers the size of the AOF file after the
# latest rewrite (if no rewrite has happened since the restart, the size of
# the AOF at startup is used).
#
# This base size is compared to the current size. If the current size is
# bigger than the specified percentage, the rewrite is triggered. Also
# you need to specify a minimal size for the AOF file to be rewritten, this
# is useful to avoid rewriting the AOF file even if the percentage increase
# is reached but it is still pretty small.
#
# Specify a percentage of zero in order to disable the automatic AOF
# rewrite feature.

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# An AOF file may be found to be truncated at the end during the Redis
# startup process, when the AOF data gets loaded back into memory.
# This may happen when the system where Redis is running
# crashes, especially when an ext4 filesystem is mounted without the
# data=ordered option (however this can't happen when Redis itself
# crashes or aborts but the operating system still works correctly).
#
# Redis can either exit with an error when this happens, or load as much
# data as possible (the default now) and start if the AOF file is found
# to be truncated at the end. The following option controls this behavior.
#
# If aof-load-truncated is set to yes, a truncated AOF file is loaded and
# the Redis server starts emitting a log to inform the user of the event.
# Otherwise if the option is set to no, the server aborts with an error
# and refuses to start. When the option is set to no, the user requires
# to fix the AOF file using the "redis-check-aof" utility before to restart
# the server.
#
# Note that if the AOF file will be found to be corrupted in the middle
# the server will still exit with an error. This option only applies when
# Redis will try to read more data from the AOF file but not enough bytes
# will be found.
aof-load-truncated yes

# When rewriting the AOF file, Redis is able to use an RDB preamble in the
# AOF file for faster rewrites and recoveries. When this option is turned
# on the rewritten AOF file is composed of two different stanzas:
#
#   [RDB file][AOF tail]
#
# When loading, Redis recognizes that the AOF file starts with the "REDIS"
# string and loads the prefixed RDB file, then continues loading the AOF
# tail.
aof-use-rdb-preamble yes

################################ LUA SCRIPTING  ###############################

# Max execution time of a Lua script in milliseconds.
#
# If the maximum execution time is reached Redis will log that a script is
# still in execution after the maximum allowed time and will start to
# reply to queries with an error.
#
# When a long running script exceeds the maximum execution time only the
# SCRIPT KILL and SHUTDOWN NOSAVE commands are available. The first can be
# used to stop a script that did not yet call any write commands. The second
# is the only way to shut down the server in the case a write command was
# already issued by the script but the user doesn't want to wait for the natural
# termination of the script.
#
# Set it to 0 or a negative value for unlimited execution without warnings.
lua-time-limit 5000

################################ REDIS CLUSTER  ###############################

# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
#
# cluster-enabled yes

# Every cluster node has a cluster configuration file. This file is not
# intended to be edited by hand. It is created and updated by Redis nodes.
# Every Redis Cluster node requires a different cluster configuration file.
# Make sure that instances running in the same system do not have
# overlapping cluster configuration file names.
#
# cluster-config-file nodes-6379.conf

# Cluster node timeout is the amount of milliseconds a node must be unreachable
# for it to be considered in failure state.
# Most other internal time limits are a multiple of the node timeout.
#
# cluster-node-timeout 15000

# A replica of a failing master will avoid to start a failover if its data
# looks too old.
#
# There is no simple way for a replica to actually have an exact measure of
# its "data age", so the following two checks are performed:
#
# 1) If there are multiple replicas able to failover, they exchange messages
#    in order to try to give an advantage to the replica with the best
#    replication offset (more data from the master processed).
#    Replicas will try to get their rank by offset, and apply to the start
#    of the failover a delay proportional to their rank.
#
# 2) Every single replica computes the time of the last interaction with
#    its master. This can be the last ping or command received (if the master
#    is still in the "connected" state), or the time that elapsed since the
#    disconnection with the master (if the replication link is currently down).
#    If the last interaction is too old, the replica will not try to failover
#    at all.
#
# The point "2" can be tuned by user. Specifically a replica will not perform
# the failover if, since the last interaction with the master, the time
# elapsed is greater than:
#
#   (node-timeout * cluster-replica-validity-factor) + repl-ping-replica-period
#
# So for example if node-timeout is 30 seconds, and the cluster-replica-validity-factor
# is 10, and assuming a default repl-ping-replica-period of 10 seconds, the
# replica will not try to failover if it was not able to talk with the master
# for longer than 310 seconds.
#
# A large cluster-replica-validity-factor may allow replicas with too old data to failover
# a master, while a too small value may prevent the cluster from being able to
# elect a replica at all.
#
# For maximum availability, it is possible to set the cluster-replica-validity-factor
# to a value of 0, which means, that replicas will always try to failover the
# master regardless of the last time they interacted with the master.
# (However they'll always try to apply a delay proportional to their
# offset rank).
#
# Zero is the only value able to guarantee that when all the partitions heal
# the cluster will always be able to continue.
#
# cluster-replica-validity-factor 10

# Cluster replicas are able to migrate to orphaned masters, that are masters
# that are left without working replicas. This improves the cluster ability
# to resist to failures as otherwise an orphaned master can't be failed over
# in case of failure if it has no working replicas.
#
# Replicas migrate to orphaned masters only if there are still at least a
# given number of other working replicas for their old master. This number
# is the "migration barrier". A migration barrier of 1 means that a replica
# will migrate only if there is at least 1 other working replica for its master
# and so forth. It usually reflects the number of replicas you want for every
# master in your cluster.
#
# Default is 1 (replicas migrate only if their masters remain with at least
# one replica). To disable migration just set it to a very large value.
# A value of 0 can be set but is useful only for debugging and dangerous
# in production.
#
# cluster-migration-barrier 1

# By default Redis Cluster nodes stop accepting queries if they detect there
# is at least a hash slot uncovered (no available node is serving it).
# This way if the cluster is partially down (for example a range of hash slots
# are no longer covered) all the cluster becomes, eventually, unavailable.
# It automatically returns available as soon as all the slots are covered again.
#
# However sometimes you want the subset of the cluster which is working,
# to continue to accept queries for the part of the key space that is still
# covered. In order to do so, just set the cluster-require-full-coverage
# option to no.
#
# cluster-require-full-coverage yes

# This option, when set to yes, prevents replicas from trying to failover its
# master during master failures. However the replica can still perform a
# manual failover, if forced to do so.
#
# This is useful in different scenarios, especially in the case of multiple
# data center operations, where we want one side to never be promoted if not
# in the case of a total DC failure.
#
# cluster-replica-no-failover no

# This option, when set to yes, allows nodes to serve read traffic while the
# the cluster is in a down state, as long as it believes it owns the slots. 
#
# This is useful for two cases.  The first case is for when an application 
# doesn't require consistency of data during node failures or network partitions.
# One example of this is a cache, where as long as the node has the data it
# should be able to serve it. 
#
# The second use case is for configurations that don't meet the recommended  
# three shards but want to enable cluster mode and scale later. A 
# master outage in a 1 or 2 shard configuration causes a read/write outage to the
# entire cluster without this option set, with it set there is only a write outage.
# Without a quorum of masters, slot ownership will not change automatically. 
#
# cluster-allow-reads-when-down no

# In order to setup your cluster make sure to read the documentation
# available at http://redis.io web site.

########################## CLUSTER DOCKER/NAT support  ########################

# In certain deployments, Redis Cluster nodes address discovery fails, because
# addresses are NAT-ted or because ports are forwarded (the typical case is
# Docker and other containers).
#
# In order to make Redis Cluster working in such environments, a static
# configuration where each node knows its public address is needed. The
# following two options are used for this scope, and are:
#
# * cluster-announce-ip
# * cluster-announce-port
# * cluster-announce-bus-port
#
# Each instructs the node about its address, client port, and cluster message
# bus port. The information is then published in the header of the bus packets
# so that other nodes will be able to correctly map the address of the node
# publishing the information.
#
# If the above options are not used, the normal Redis Cluster auto-detection
# will be used instead.
#
# Note that when remapped, the bus port may not be at the fixed offset of
# clients port + 10000, so you can specify any port and bus-port depending
# on how they get remapped. If the bus-port is not set, a fixed offset of
# 10000 will be used as usual.
#
# Example:
#
# cluster-announce-ip 10.1.1.5
# cluster-announce-port 6379
# cluster-announce-bus-port 6380

################################## SLOW LOG ###################################

# The Redis Slow Log is a system to log queries that exceeded a specified
# execution time. The execution time does not include the I/O operations
# like talking with the client, sending the reply and so forth,
# but just the time needed to actually execute the command (this is the only
# stage of command execution where the thread is blocked and can not serve
# other requests in the meantime).
#
# You can configure the slow log with two parameters: one tells Redis
# what is the execution time, in microseconds, to exceed in order for the
# command to get logged, and the other parameter is the length of the
# slow log. When a new command is logged the oldest one is removed from the
# queue of logged commands.

# The following time is expressed in microseconds, so 1000000 is equivalent
# to one second. Note that a negative number disables the slow log, while
# a value of zero forces the logging of every command.
slowlog-log-slower-than 10000

# There is no limit to this length. Just be aware that it will consume memory.
# You can reclaim memory used by the slow log with SLOWLOG RESET.
slowlog-max-len 128

################################ LATENCY MONITOR ##############################

# The Redis latency monitoring subsystem samples different operations
# at runtime in order to collect data related to possible sources of
# latency of a Redis instance.
#
# Via the LATENCY command this information is available to the user that can
# print graphs and obtain reports.
#
# The system only logs operations that were performed in a time equal or
# greater than the amount of milliseconds specified via the
# latency-monitor-threshold configuration directive. When its value is set
# to zero, the latency monitor is turned off.
#
# By default latency monitoring is disabled since it is mostly not needed
# if you don't have latency issues, and collecting data has a performance
# impact, that while very small, can be measured under big load. Latency
# monitoring can easily be enabled at runtime using the command
# "CONFIG SET latency-monitor-threshold <milliseconds>" if needed.
latency-monitor-threshold 0

############################# EVENT NOTIFICATION ##############################

# Redis can notify Pub/Sub clients about events happening in the key space.
# This feature is documented at http://redis.io/topics/notifications
#
# For instance if keyspace events notification is enabled, and a client
# performs a DEL operation on key "foo" stored in the Database 0, two
# messages will be published via Pub/Sub:
#
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# It is possible to select the events that Redis will notify among a set
# of classes. Every class is identified by a single character:
#
#  K     Keyspace events, published with __keyspace@<db>__ prefix.
#  E     Keyevent events, published with __keyevent@<db>__ prefix.
#  g     Generic commands (non-type specific) like DEL, EXPIRE, RENAME, ...
#  $     String commands
#  l     List commands
#  s     Set commands
#  h     Hash commands
#  z     Sorted set commands
#  x     Expired events (events generated every time a key expires)
#  e     Evicted events (events generated when a key is evicted for maxmemory)
#  t     Stream commands
#  m     Key-miss events (Note: It is not included in the 'A' class)
#  A     Alias for g$lshzxet, so that the "AKE" string means all the events
#        (Except key-miss events which are excluded from 'A' due to their
#         unique nature).
#
#  The "notify-keyspace-events" takes as argument a string that is composed
#  of zero or multiple characters. The empty string means that notifications
#  are disabled.
#
#  Example: to enable list and generic events, from the point of view of the
#           event name, use:
#
#  notify-keyspace-events Elg
#
#  Example 2: to get the stream of the expired keys subscribing to channel
#             name __keyevent@0__:expired use:
#
#  notify-keyspace-events Ex
#
#  By default all notifications are disabled because most users don't need
#  this feature and the feature has some overhead. Note that if you don't
#  specify at least one of K or E, no events will be delivered.
notify-keyspace-events ""

############################### GOPHER SERVER #################################

# Redis contains an implementation of the Gopher protocol, as specified in
# the RFC 1436 (https://www.ietf.org/rfc/rfc1436.txt).
#
# The Gopher protocol was very popular in the late '90s. It is an alternative
# to the web, and the implementation both server and client side is so simple
# that the Redis server has just 100 lines of code in order to implement this
# support.
#
# What do you do with Gopher nowadays? Well Gopher never *really* died, and
# lately there is a movement in order for the Gopher more hierarchical content
# composed of just plain text documents to be resurrected. Some want a simpler
# internet, others believe that the mainstream internet became too much
# controlled, and it's cool to create an alternative space for people that
# want a bit of fresh air.
#
# Anyway for the 10nth birthday of the Redis, we gave it the Gopher protocol
# as a gift.
#
# --- HOW IT WORKS? ---
#
# The Redis Gopher support uses the inline protocol of Redis, and specifically
# two kind of inline requests that were anyway illegal: an empty request
# or any request that starts with "/" (there are no Redis commands starting
# with such a slash). Normal RESP2/RESP3 requests are completely out of the
# path of the Gopher protocol implementation and are served as usual as well.
#
# If you open a connection to Redis when Gopher is enabled and send it
# a string like "/foo", if there is a key named "/foo" it is served via the
# Gopher protocol.
#
# In order to create a real Gopher "hole" (the name of a Gopher site in Gopher
# talking), you likely need a script like the following:
#
#   https://github.com/antirez/gopher2redis
#
# --- SECURITY WARNING ---
#
# If you plan to put Redis on the internet in a publicly accessible address
# to server Gopher pages MAKE SURE TO SET A PASSWORD to the instance.
# Once a password is set:
#
#   1. The Gopher server (when enabled, not by default) will still serve
#      content via Gopher.
#   2. However other commands cannot be called before the client will
#      authenticate.
#
# So use the 'requirepass' option to protect your instance.
#
# Note that Gopher is not currently supported when 'io-threads-do-reads'
# is enabled.
#
# To enable Gopher support, uncomment the following line and set the option
# from no (the default) to yes.
#
# gopher-enabled no

############################### ADVANCED CONFIG ###############################

# Hashes are encoded using a memory efficient data structure when they have a
# small number of entries, and the biggest entry does not exceed a given
# threshold. These thresholds can be configured using the following directives.
hash-max-ziplist-entries 512
hash-max-ziplist-value 64

# Lists are also encoded in a special way to save a lot of space.
# The number of entries allowed per internal list node can be specified
# as a fixed maximum size or a maximum number of elements.
# For a fixed maximum size, use -5 through -1, meaning:
# -5: max size: 64 Kb  <-- not recommended for normal workloads
# -4: max size: 32 Kb  <-- not recommended
# -3: max size: 16 Kb  <-- probably not recommended
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# Positive numbers mean store up to _exactly_ that number of elements
# per list node.
# The highest performing option is usually -2 (8 Kb size) or -1 (4 Kb size),
# but if your use case is unique, adjust the settings as necessary.
list-max-ziplist-size -2

# Lists may also be compressed.
# Compress depth is the number of quicklist ziplist nodes from *each* side of
# the list to *exclude* from compression.  The head and tail of the list
# are always uncompressed for fast push/pop operations.  Settings are:
# 0: disable all list compression
# 1: depth 1 means "don't start compressing until after 1 node into the list,
#    going from either the head or tail"
#    So: [head]->node->node->...->node->[tail]
#    [head], [tail] will always be uncompressed; inner nodes will compress.
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#    2 here means: don't compress head or head->next or tail->prev or tail,
#    but compress all nodes between them.
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# etc.
list-compress-depth 0

# Sets have a special encoding in just one case: when a set is composed
# of just strings that happen to be integers in radix 10 in the range
# of 64 bit signed integers.
# The following configuration setting sets the limit in the size of the
# set in order to use this special memory saving encoding.
set-max-intset-entries 512

# Similarly to hashes and lists, sorted sets are also specially encoded in
# order to save a lot of space. This encoding is only used when the length and
# elements of a sorted set are below the following limits:
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

# HyperLogLog sparse representation bytes limit. The limit includes the
# 16 bytes header. When an HyperLogLog using the sparse representation crosses
# this limit, it is converted into the dense representation.
#
# A value greater than 16000 is totally useless, since at that point the
# dense representation is more memory efficient.
#
# The suggested value is ~ 3000 in order to have the benefits of
# the space efficient encoding without slowing down too much PFADD,
# which is O(N) with the sparse encoding. The value can be raised to
# ~ 10000 when CPU is not a concern, but space is, and the data set is
# composed of many HyperLogLogs with cardinality in the 0 - 15000 range.
hll-sparse-max-bytes 3000

# Streams macro node max size / items. The stream data structure is a radix
# tree of big nodes that encode multiple items inside. Using this configuration
# it is possible to configure how big a single node can be in bytes, and the
# maximum number of items it may contain before switching to a new node when
# appending new stream entries. If any of the following settings are set to
# zero, the limit is ignored, so for instance it is possible to set just a
# max entries limit by setting max-bytes to 0 and max-entries to the desired
# value.
stream-node-max-bytes 4096
stream-node-max-entries 100

# Active rehashing uses 1 millisecond every 100 milliseconds of CPU time in
# order to help rehashing the main Redis hash table (the one mapping top-level
# keys to values). The hash table implementation Redis uses (see dict.c)
# performs a lazy rehashing: the more operation you run into a hash table
# that is rehashing, the more rehashing "steps" are performed, so if the
# server is idle the rehashing is never complete and some more memory is used
# by the hash table.
#
# The default is to use this millisecond 10 times every second in order to
# actively rehash the main dictionaries, freeing memory when possible.
#
# If unsure:
# use "activerehashing no" if you have hard latency requirements and it is
# not a good thing in your environment that Redis can reply from time to time
# to queries with 2 milliseconds delay.
#
# use "activerehashing yes" if you don't have such hard requirements but
# want to free memory asap when possible.
activerehashing yes

# The client output buffer limits can be used to force disconnection of clients
# that are not reading data from the server fast enough for some reason (a
# common reason is that a Pub/Sub client can't consume messages as fast as the
# publisher can produce them).
#
# The limit can be set differently for the three different classes of clients:
#
# normal -> normal clients including MONITOR clients
# replica  -> replica clients
# pubsub -> clients subscribed to at least one pubsub channel or pattern
#
# The syntax of every client-output-buffer-limit directive is the following:
#
# client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
#
# A client is immediately disconnected once the hard limit is reached, or if
# the soft limit is reached and remains reached for the specified number of
# seconds (continuously).
# So for instance if the hard limit is 32 megabytes and the soft limit is
# 16 megabytes / 10 seconds, the client will get disconnected immediately
# if the size of the output buffers reach 32 megabytes, but will also get
# disconnected if the client reaches 16 megabytes and continuously overcomes
# the limit for 10 seconds.
#
# By default normal clients are not limited because they don't receive data
# without asking (in a push way), but just after a request, so only
# asynchronous clients may create a scenario where data is requested faster
# than it can read.
#
# Instead there is a default limit for pubsub and replica clients, since
# subscribers and replicas receive data in a push fashion.
#
# Both the hard or the soft limit can be disabled by setting them to zero.
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# Client query buffers accumulate new commands. They are limited to a fixed
# amount by default in order to avoid that a protocol desynchronization (for
# instance due to a bug in the client) will lead to unbound memory usage in
# the query buffer. However you can configure it here if you have very special
# needs, such us huge multi/exec requests or alike.
#
# client-query-buffer-limit 1gb

# In the Redis protocol, bulk requests, that are, elements representing single
# strings, are normally limited to 512 mb. However you can change this limit
# here, but must be 1mb or greater
#
# proto-max-bulk-len 512mb

# Redis calls an internal function to perform many background tasks, like
# closing connections of clients in timeout, purging expired keys that are
# never requested, and so forth.
#
# Not all tasks are performed with the same frequency, but Redis checks for
# tasks to perform according to the specified "hz" value.
#
# By default "hz" is set to 10. Raising the value will use more CPU when
# Redis is idle, but at the same time will make Redis more responsive when
# there are many keys expiring at the same time, and timeouts may be
# handled with more precision.
#
# The range is between 1 and 500, however a value over 100 is usually not
# a good idea. Most users should use the default of 10 and raise this up to
# 100 only in environments where very low latency is required.
hz 10

# Normally it is useful to have an HZ value which is proportional to the
# number of clients connected. This is useful in order, for instance, to
# avoid too many clients are processed for each background task invocation
# in order to avoid latency spikes.
#
# Since the default HZ value by default is conservatively set to 10, Redis
# offers, and enables by default, the ability to use an adaptive HZ value
# which will temporarily raise when there are many connected clients.
#
# When dynamic HZ is enabled, the actual configured HZ will be used
# as a baseline, but multiples of the configured HZ value will be actually
# used as needed once more clients are connected. In this way an idle
# instance will use very little CPU time while a busy instance will be
# more responsive.
dynamic-hz yes

# When a child rewrites the AOF file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
aof-rewrite-incremental-fsync yes

# When redis saves RDB file, if the following option is enabled
# the file will be fsync-ed every 32 MB of data generated. This is useful
# in order to commit the file to the disk more incrementally and avoid
# big latency spikes.
rdb-save-incremental-fsync yes

# Redis LFU eviction (see maxmemory setting) can be tuned. However it is a good
# idea to start with the default settings and only change them after investigating
# how to improve the performances and how the keys LFU change over time, which
# is possible to inspect via the OBJECT FREQ command.
#
# There are two tunable parameters in the Redis LFU implementation: the
# counter logarithm factor and the counter decay time. It is important to
# understand what the two parameters mean before changing them.
#
# The LFU counter is just 8 bits per key, it's maximum value is 255, so Redis
# uses a probabilistic increment with logarithmic behavior. Given the value
# of the old counter, when a key is accessed, the counter is incremented in
# this way:
#
# 1. A random number R between 0 and 1 is extracted.
# 2. A probability P is calculated as 1/(old_value*lfu_log_factor+1).
# 3. The counter is incremented only if R < P.
#
# The default lfu-log-factor is 10. This is a table of how the frequency
# counter changes with a different number of accesses with different
# logarithmic factors:
#
# +--------+------------+------------+------------+------------+------------+
# | factor | 100 hits   | 1000 hits  | 100K hits  | 1M hits    | 10M hits   |
# +--------+------------+------------+------------+------------+------------+
# | 0      | 104        | 255        | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 1      | 18         | 49         | 255        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 10     | 10         | 18         | 142        | 255        | 255        |
# +--------+------------+------------+------------+------------+------------+
# | 100    | 8          | 11         | 49         | 143        | 255        |
# +--------+------------+------------+------------+------------+------------+
#
# NOTE: The above table was obtained by running the following commands:
#
#   redis-benchmark -n 1000000 incr foo
#   redis-cli object freq foo
#
# NOTE 2: The counter initial value is 5 in order to give new objects a chance
# to accumulate hits.
#
# The counter decay time is the time, in minutes, that must elapse in order
# for the key counter to be divided by two (or decremented if it has a value
# less <= 10).
#
# The default value for the lfu-decay-time is 1. A special value of 0 means to
# decay the counter every time it happens to be scanned.
#
# lfu-log-factor 10
# lfu-decay-time 1

########################### ACTIVE DEFRAGMENTATION #######################
#
# What is active defragmentation?
# -------------------------------
#
# Active (online) defragmentation allows a Redis server to compact the
# spaces left between small allocations and deallocations of data in memory,
# thus allowing to reclaim back memory.
#
# Fragmentation is a natural process that happens with every allocator (but
# less so with Jemalloc, fortunately) and certain workloads. Normally a server
# restart is needed in order to lower the fragmentation, or at least to flush
# away all the data and create it again. However thanks to this feature
# implemented by Oran Agra for Redis 4.0 this process can happen at runtime
# in a "hot" way, while the server is running.
#
# Basically when the fragmentation is over a certain level (see the
# configuration options below) Redis will start to create new copies of the
# values in contiguous memory regions by exploiting certain specific Jemalloc
# features (in order to understand if an allocation is causing fragmentation
# and to allocate it in a better place), and at the same time, will release the
# old copies of the data. This process, repeated incrementally for all the keys
# will cause the fragmentation to drop back to normal values.
#
# Important things to understand:
#
# 1. This feature is disabled by default, and only works if you compiled Redis
#    to use the copy of Jemalloc we ship with the source code of Redis.
#    This is the default with Linux builds.
#
# 2. You never need to enable this feature if you don't have fragmentation
#    issues.
#
# 3. Once you experience fragmentation, you can enable this feature when
#    needed with the command "CONFIG SET activedefrag yes".
#
# The configuration parameters are able to fine tune the behavior of the
# defragmentation process. If you are not sure about what they mean it is
# a good idea to leave the defaults untouched.

# Enabled active defragmentation
# activedefrag no

# Minimum amount of fragmentation waste to start active defrag
# active-defrag-ignore-bytes 100mb

# Minimum percentage of fragmentation to start active defrag
# active-defrag-threshold-lower 10

# Maximum percentage of fragmentation at which we use maximum effort
# active-defrag-threshold-upper 100

# Minimal effort for defrag in CPU percentage, to be used when the lower
# threshold is reached
# active-defrag-cycle-min 1

# Maximal effort for defrag in CPU percentage, to be used when the upper
# threshold is reached
# active-defrag-cycle-max 25

# Maximum number of set/hash/zset/list fields that will be processed from
# the main dictionary scan
# active-defrag-max-scan-fields 1000

# Jemalloc background thread for purging will be enabled by default
jemalloc-bg-thread yes

# It is possible to pin different threads and processes of Redis to specific
# CPUs in your system, in order to maximize the performances of the server.
# This is useful both in order to pin different Redis threads in different
# CPUs, but also in order to make sure that multiple Redis instances running
# in the same host will be pinned to different CPUs.
#
# Normally you can do this using the "taskset" command, however it is also
# possible to this via Redis configuration directly, both in Linux and FreeBSD.
#
# You can pin the server/IO threads, bio threads, aof rewrite child process, and
# the bgsave child process. The syntax to specify the cpu list is the same as
# the taskset command:
#
# Set redis server/io threads to cpu affinity 0,2,4,6:
# server_cpulist 0-7:2
#
# Set bio threads to cpu affinity 1,3:
# bio_cpulist 1,3
#
# Set aof rewrite child process to cpu affinity 8,9,10,11:
# aof_rewrite_cpulist 8-11
#
# Set bgsave child process to cpu affinity 1,10,11
# bgsave_cpulist 1,10-11

# In some cases redis will emit warnings and even refuse to start if it detects
# that the system is in bad state, it is possible to suppress these warnings
# by setting the following config which takes a space delimited list of warnings
# to suppress
#
# ignore-warnings ARM64-COW-BUG

```











### 3、Redis持久化

> 快照-持久化 在规定的时间内 执行了多少次的操作 则会持久化到文件.rdb .aof 中
>
> 在指定的时间间隔内将内存中的数据集快照写入磁盘中 也就是进行snapshot快照  它恢复时是将快照文件直接读到内存中

#### 1）RDB（Redis Database）

Redis会单独创建fork一个子进程来进行持久化 会先将数据写入到一个临时文件中 待持久化过程都结束了 在用这个临时文件替换上次持久化好的文件 整个过程  主进程不进行任何的IO操作 这就确保了极高的性能 如果需要大规模数据的恢复 且对于数据恢复的完整性不是非常敏感 那RDB模式要比AOF方式更加的高效， **RDB的缺点时最后一次持久化后的数据可能会丢失**。默认模式就是RDB模式。

文件名是dump.rdb

```bash
# Unless specified otherwise, by default Redis will save the DB:
#   * After 3600 seconds (an hour) if at least 1 key changed
#   * After 300 seconds (5 minutes) if at least 100 keys changed
#   * After 60 seconds if at least 10000 keys changed
#
# You can set these explicitly by uncommenting the three following lines.
# 快照次数配置 一个小时 一个key进行改变就进行配置
# save 3600 1
# save 300 100
# save 60 10000
```

#### 触发机制

- save命令
- 执行flushDbALl 也会触发
- 退出redis 也会产生rdb文件

#### 恢复机制

- 只需要把rdb文件放到redis启动目录就可以 redis启动的时候会自动检查dump.rdb恢复其中的数据。

#### 优点

- 适合大规模的数据恢复

- 对数据的完整性不高 

#### 缺点

- 需要一定的时间间隔操作 如果redis宕机 **最后一次的保存的数据会消失**
- fork子线程需要内存

![image-20210321120251137](https://tva1.sinaimg.cn/large/008eGmZEly1gore8hoff5j30qw0g442u.jpg)





### 2）AOF（append only file）

> 将我们的所有的命令都记录下来。

以日志的形式记录每个写的操作 将redis执行过的所有指令记录下来（**读操作不记录**）**只许追加文件但不可以改写文件** redis启动之初会读取该文件重新构建数据  换言之 redis重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。默认是不开启的。

aof保存的是 appendonly.aof文件

```bash
#AOF 模式配置
appendonly no
# The name of the append only file (default: "appendonly.aof")
appendfilename "appendonly.aof"

# If unsure, use "everysec".
# appendfsync always
# 每秒同步一次数据
appendfsync everysec

#从不同步 效率最高
# appendfsync no
```



#### 

#### 触发机制：

- 每次写操作就会触发

#### 恢复机制

- 重启时会自动读取aof文

**Tips**:如果 appendonly.aof 被破坏 重启redis是失败的 redis提供工具 redis-check-aof --fix 修复

#### 缺点

- 恢复时间：在大数据的时候恢复很慢 aof文件大小远大于rdb文件
- 运行效率：aof运行效率也要比rdb满 频繁io读写
- 每秒同步一次 可能会丢失一秒的数据

#### 优点：

- ​	数据完整性：每一次修改都保存 文件完整性更好



![image-20210321123127285](https://tva1.sinaimg.cn/large/008eGmZEly1gorf27pp4ij30qu0giq72.jpg)





### 	持久化总结：

```java
1、RDB持久化方式能够在指定的时间间隔内对数据进行快照存储
2、AOF持久化方式记录每次对服务器写的操作，当服务器重启对时候会重新执行这些命令来恢复原始的数据。AOF命令以Redis协议追加保存每次写的操作到文件末尾，Redis还能对AOF文件进行后台重写，使得AOF文件的体积不至于过大。
3、只做缓存，如果你只希望你的数据在服务器运行的时候存在，你也可以不使用任何持久化。
4、同时开启两种持久化方式
	.在这种情况下，当Redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
	.RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢 作者建议不要，因为RDB更适合用于备份数据库（AOF不断变化不好备份），快速		重启，而且不会有AOF可能潜在的Bug，可以做备份手段。
5、性能建议
	.因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留 save 900 1 这条规则。
	.如果Enable AOF ，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以，代价是带了持续的IO，二是AOF rewrite的最后		将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要磁盘允许 ，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可	 以设置到5G以上，默认超过原大小100%，重写可以改到适当的大小。
	.如果不Enable AOF，仅靠Master-Slave Replication 实现高可用也行，能省掉一大笔IO，也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时宕		机，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个文件，微博就是这种架构。
	
```



## 六、Redis发布与订阅

> [commands](http://www.redis.cn/commands.html#pubsub)
>
> redis发布订阅 pub/sub 是一种消息通信模式 发送者pub发送消息 订阅者接受消息 

使用三个因素：

- 第一个：消息发送者 
- 第二个 发送频道  
- 第三个 消息接收者

```bash
#发送消息 往一个渠道
127.0.0.1:6379> PUBLISH channel newMessage
1) "punsubscribe"
2) "aaaa"
3) (integer) 0
127.0.0.1:6379> PUBLISH channel newMessage1
(integer) 1
127.0.0.1:6379> PUBLISH channel newMessage12
(integer) 1
127.0.0.1:6379> PUBLISH channel newMessage12
(integer) 1
127.0.0.1:6379> PUBLISH channel newMessage123
(integer) 1
127.0.0.1:6379> 

#接受消息 订阅频道
127.0.0.1:6379> SUBSCRIBE channel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "channel"
3) (integer) 1

1) "message"
2) "channel"
3) "newMessage"

1) "message"
2) "channel"
3) "newMessage1"

1) "message"
2) "channel"
3) "newMessage12"

1) "message"
2) "channel"
3) "newMessage12"

```

![image-20210321135107623](https://tva1.sinaimg.cn/large/008eGmZEly1gorhd40lokj30r60f5q94.jpg)

#### 底层原理

Redis是通过C实现的，通过分析Redis源码里的pubsub.c文件，了解发布和订阅机制的底层实现，借此加深对Redis的理解。

Redis通过Publish subscribe psubscribe 等实现发布和订阅功能。

通过Subscribe命令订阅某频道后，redis-server里维护了一个字典，字典的key就是一个个channel，而字典的value则是一个链表，链表保存了所有订阅这个channel的客户端。subscribe命令的关键就是将客户端添加到给定的channel的订阅链表中。

通过publish命令向订阅者发送消息，redis-server会使用给定的频道作为key，在它所维护的channel字典中查找记录了这个频道的所有客户端的链表，遍历这个链表，将消息发布给所有的订阅者。

Pub/Sub从字面上理解就是发布帝和订阅，在Redis中，你可以设定对某个key值进行消息发布和消息订阅，当一个key值进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。

#### 常见用途

- 实时消息系统
- 实时聊天 聊天室 消息回显。websocket
- 订阅 关注系统 ---微信微博关注系统

  



## 七、Redis主从复制

> 主从复制 读写分离,主机只做写操作，从机负责读操作。

概念：

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器，前者称为主节点（master/leader），后者称为从节点（slave/follower；数据的复制是单向的，只能由主节点到从节点。master节点已写为主，Slave以读为主。

默认情况下，**每台Redis服务器都是主节点；**且一个主节点可以有多个从节点，但是一个从节点只能有一个主节点。

主从复制到作用：

- 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式
- 故障恢复：当主节点出现问题的时候，可以由从节点提供服务，实现快速的故障恢复；是一种服务的冗余
- 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其在写少读多的场景下，通过多个从节点来分担读负载，可以大大提高Redis服务器的并非量
- 高可用基石：主从复制还是哨兵和集群能够实施的基础，因此说主从复制时Redis高可用的基础

Tips

一般来说，要将Redis运用于工程项目中，只使用一台Redis是万万不能的，原因如下：

- 才能结构上来说，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大
- 从容量上来说，单个Redis服务器内存容量有限，不能将所有的内存作为Redis存储内存，单台服务器的Redis内存不应该超过20G

电商网站上的商品，一般都是一次上传，无数次浏览的，少写多读。

![image-20210321140449974](/Users/gaoshang/Library/Application Support/typora-user-images/image-20210321140449974.png)

最少三台服务器。一主两从。主作为写。从负责读。

##### 1、查看服务主从信息

```bash
	#查看客户端信息 主从信息
	127.0.0.1:6380> INFO replication
# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:8a2f97a36339b902281a29f3fce8a6f9e92d1b04
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

```

##### 2、修改从机配置-搭集群

```bash
#修改文件-搭集群 一台服务
#修改端口
# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6382

#修改pidfile
# Note that on modern Linux systems "/run/redis.pid" is more conforming
# and should be used instead.
pidfile /var/run/redis_6382.pid

#修改log文件
# Specify the log file name. Also the empty string can be used to force
# Redis to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile "6382.log"

#修改rdb文件名
# The filename where to dump the DB
dbfilename dump-6382.rdb
```

##### 3、配置从机-master 

需要slave找master

```bash

#从机上使用命令 暂时性
slaveof 127.0.0.1 6380

#使用配置文件 永久性的
#设置主机ip和端口
replicaof 127.0.0.1 6379
#设置主机密码
masterauth JackGao1996



#在主机上查看 发现有两个从机
127.0.0.1:6379> INFO replication
# Replication
role:master
connected_slaves:2
slave0:ip=127.0.0.1,port=6381,state=online,offset=126,lag=0
slave1:ip=127.0.0.1,port=6382,state=online,offset=126,lag=1
master_replid:5702bdf0328ccc5ff3bd0be812a815095405ec97
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:126
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:126
127.0.0.1:6379> 

```

##### 4、 读写测试

主机负责写 从机负责读

```bash
#主机设置一个值
127.0.0.1:6379> set key 001
OK

#从机读取出
127.0.0.1:6381> get key
"001"
#从机无法写操作
127.0.0.1:6381> set key 11
(error) READONLY You can't write against a read only replica.
```

如果主机宕机，没有设置哨兵模式 从机还是可以拿到对应的值 主机恢复后 从机还是拿到值

如果从机宕机，没有设置哨兵模式 从机可以宕机后到开机的值 会重新从主机读取数据 数据是单向的

##### 5、复制原理

- Slave启动成功连接master后会发送一个sync命令
- Master接到命令，启动后台的存盘进程，同时收集所有接受到的用于修改数据集命令，在后台进行执行完毕之后，master将传送整个数据呀文件到slave，并完成一次完全同步
- 全量复制：slave服务在接收到数据库文件数据后，将其存盘并加载到内存中
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave 完成同步
- tips：只要是重新连接master 全量同步将被自动执行



##### 6、层层链路方式（不会使用）：

![image-20210403220130761](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6wmfajorj31c20oytuz.jpg)

##### 7、主机宕机，从机手动变为master（不会使用）

```bash
#从机手动变为master
slaveof  no one 
```

##### 8、哨兵模式

> 自动选举主节点

###### 0、概念：

```
主从切换技术的方式是：当master宕机后，需要手动把一台slave切换为master。如果不用哨兵，需要人工干预。
Redis从2.8版本正式推出Sentinel哨兵架构来解决这个问题。
从机手动为master的自动版本，能够从后台监控主机是否故障，如果故障了根据投票来自动将从库切换为主库。
哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，他会独立运行。其原理是哨兵通过发送命令，等待redis服务器响应，从而监控运行多个Redis实例
```

###### 0-1、单哨兵架构图：

![image-20210403203054115](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6u052t4uj30fe0csju2.jpg)

###### 0-2、多哨兵架构图：

![image-20210403210203346](/Users/gaoshang/Library/Application Support/typora-user-images/image-20210403210203346.png)

假设主服务器宕机，哨兵1先检测这个结果，系统此时不会立即进行failover过程，仅仅是哨兵1主观认为这个服务器不可用，这个现象成为主观下线。当后面的哨兵也检测主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover【故障转移】操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器切换为主服务器，这个过程称为客观下线。



###### 1、配置sentinel.conf

```bash
 # 禁止保护模式
protected-mode no
# 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，
#192.168.11.128代表监控的主服务器，6379代表端口，
#2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
sentinel monitor redis-master 127.0.0.1 6379 1
# sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass redis-master JackGao1996



#启动哨兵
[root@iz2zead4m1v827c36ss5ibz src]# ./redis-sentinel  redis-config/redis-sentinel.conf 
24156:X 03 Apr 2021 21:13:02.033 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
24156:X 03 Apr 2021 21:13:02.033 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=24156, just started
24156:X 03 Apr 2021 21:13:02.033 # Configuration loaded
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.8 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in sentinel mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 26379
 |    `-._   `._    /     _.-'    |     PID: 24156
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

24156:X 03 Apr 2021 21:13:02.034 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
24156:X 03 Apr 2021 21:13:02.038 # Sentinel ID is 1214b29cbf66a6b6011e86ad2accc1596c0985fc
24156:X 03 Apr 2021 21:13:02.038 # +monitor master redis-master 127.0.0.1 6379 quorum 1
24156:X 03 Apr 2021 21:13:02.039 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ redis-master 127.0.0.1 6379
24156:X 03 Apr 2021 21:13:02.042 * +slave slave 127.0.0.1:6382 127.0.0.1 6382 @ redis-master 127.0.0.1 6379


#选举成功 6382端口服务作为主节点
24156:X 03 Apr 2021 21:13:42.638 # +selected-slave slave 127.0.0.1:6382 127.0.0.1 6382 @ redis-master 127.0.0.1 6379
24156:X 03 Apr 2021 21:13:42.638 * +failover-state-send-slaveof-noone slave 127.0.0.1:6382 127.0.0.1 6382 @ redis-master 127.0.0.1 6379
24156:X 03 Apr 2021 21:13:42.704 * +failover-state-wait-promotion slave 127.0.0.1:6382 127.0.0.1 6382 @ redis-master 127.0.0.1 6379



#6381 认6382 为主节点
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6382
master_link_status:down
master_last_io_seconds_ago:-1
master_sync_in_progress:0
slave_repl_offset:6997
master_link_down_since_seconds:257
slave_priority:100
slave_read_only:1
connected_slaves:0
master_failover_state:no-failover
master_replid:5702bdf0328ccc5ff3bd0be812a815095405ec97
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:6997
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:6997

```

```bash
#sentinel端口26379
[root@iz2zead4m1v827c36ss5ibz src]# ps -ef|grep redis
root      1037 13690  0 22:23 pts/0    00:00:00 grep --color=auto redis
root      4808     1  0 21:19 ?        00:00:13 redis-server *:6379
root      8408     1  0 21:21 ?        00:00:09 redis-server 127.0.0.1:6382
root     12691     1  0 19:58 ?        00:00:23 redis-server 127.0.0.1:6381
root     16612     1  0 19:29 ?        00:00:28 redis-server 127.0.0.1:6380
root     24156 17209  0 21:13 pts/2    00:00:07 ./redis-sentinel *:26379 [sentinel]

```



###### 2、sentinel.conf

```bash
# Example sentinel.conf

# *** IMPORTANT ***
#
# By default Sentinel will not be reachable from interfaces different than
# localhost, either use the 'bind' directive to bind to a list of network
# interfaces, or disable protected mode with "protected-mode no" by
# adding it to this configuration file.
#
# Before doing that MAKE SURE the instance is protected from the outside
# world via firewalling or other means.
#
# For example you may use one of the following:
#
# bind 127.0.0.1 192.168.1.1
#
# protected-mode no

# port <sentinel-port>
# The port that this sentinel instance will run on
port 26379

# By default Redis Sentinel does not run as a daemon. Use 'yes' if you need it.
# Note that Redis will write a pid file in /var/run/redis-sentinel.pid when
# daemonized.
daemonize no

# When running daemonized, Redis Sentinel writes a pid file in
# /var/run/redis-sentinel.pid by default. You can specify a custom pid file
# location here.
pidfile /var/run/redis-sentinel.pid

# Specify the log file name. Also the empty string can be used to force
# Sentinel to log on the standard output. Note that if you use standard
# output for logging but daemonize, logs will be sent to /dev/null
logfile ""

# sentinel announce-ip <ip>
# sentinel announce-port <port>
#
# The above two configuration directives are useful in environments where,
# because of NAT, Sentinel is reachable from outside via a non-local address.
#
# When announce-ip is provided, the Sentinel will claim the specified IP address
# in HELLO messages used to gossip its presence, instead of auto-detecting the
# local address as it usually does.
#
# Similarly when announce-port is provided and is valid and non-zero, Sentinel
# will announce the specified TCP port.
#
# The two options don't need to be used together, if only announce-ip is
# provided, the Sentinel will announce the specified IP and the server port
# as specified by the "port" option. If only announce-port is provided, the
# Sentinel will announce the auto-detected local IP and the specified port.
#
# Example:
#
# sentinel announce-ip 1.2.3.4

# dir <working-directory>
# Every long running process should have a well-defined working directory.
# For Redis Sentinel to chdir to /tmp at startup is the simplest thing
# for the process to don't interfere with administrative tasks such as
# unmounting filesystems.
dir /tmp

# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# Tells Sentinel to monitor this master, and to consider it in O_DOWN
# (Objectively Down) state only if at least <quorum> sentinels agree.
#
# Note that whatever is the ODOWN quorum, a Sentinel will require to
# be elected by the majority of the known Sentinels in order to
# start a failover, so no failover can be performed in minority.
#
# Replicas are auto-discovered, so you don't need to specify replicas in
# any way. Sentinel itself will rewrite this configuration file adding
# the replicas using additional configuration options.
# Also note that the configuration file is rewritten when a
# replica is promoted to master.
#
# Note: master name should not include special characters or spaces.
# The valid charset is A-z 0-9 and the three characters ".-_".
sentinel monitor mymaster 127.0.0.1 6379 2

# sentinel auth-pass <master-name> <password>
#
# Set the password to use to authenticate with the master and replicas.
# Useful if there is a password set in the Redis instances to monitor.
#
# Note that the master password is also used for replicas, so it is not
# possible to set a different password in masters and replicas instances
# if you want to be able to monitor these instances with Sentinel.
#
# However you can have Redis instances without the authentication enabled
# mixed with Redis instances requiring the authentication (as long as the
# password set is the same for all the instances requiring the password) as
# the AUTH command will have no effect in Redis instances with authentication
# switched off.
#
# Example:
#
# sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# Number of milliseconds the master (or any attached replica or sentinel) should
# be unreachable (as in, not acceptable reply to PING, continuously, for the
# specified period) in order to consider it in S_DOWN state (Subjectively
# Down).
#
# Default is 30 seconds.
sentinel down-after-milliseconds mymaster 30000

# sentinel parallel-syncs <master-name> <numreplicas>
#
# How many replicas we can reconfigure to point to the new replica simultaneously
# during the failover. Use a low number if you use the replicas to serve query
# to avoid that all the replicas will be unreachable at about the same
# time while performing the synchronization with the master.
sentinel parallel-syncs mymaster 1

# sentinel failover-timeout <master-name> <milliseconds>
#
# Specifies the failover timeout in milliseconds. It is used in many ways:
#
# - The time needed to re-start a failover after a previous failover was
#   already tried against the same master by a given Sentinel, is two
#   times the failover timeout.
#
# - The time needed for a replica replicating to a wrong master according
#   to a Sentinel current configuration, to be forced to replicate
#   with the right master, is exactly the failover timeout (counting since
#   the moment a Sentinel detected the misconfiguration).
#
# - The time needed to cancel a failover that is already in progress but
#   did not produced any configuration change (SLAVEOF NO ONE yet not
#   acknowledged by the promoted replica).
#
# - The maximum time a failover in progress waits for all the replicas to be
#   reconfigured as replicas of the new master. However even after this time
#   the replicas will be reconfigured by the Sentinels anyway, but not with
#   the exact parallel-syncs progression as specified.
#
# Default is 3 minutes.
sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION
#
# sentinel notification-script and sentinel reconfig-script are used in order
# to configure scripts that are called to notify the system administrator
# or to reconfigure clients after a failover. The scripts are executed
# with the following rules for error handling:
#
# If script exits with "1" the execution is retried later (up to a maximum
# number of times currently set to 10).
#
# If script exits with "2" (or an higher value) the script execution is
# not retried.
#
# If script terminates because it receives a signal the behavior is the same
# as exit code 1.
#
# A script has a maximum running time of 60 seconds. After this limit is
# reached the script is terminated with a SIGKILL and the execution retried.

# NOTIFICATION SCRIPT
#
# sentinel notification-script <master-name> <script-path>
# 
# Call the specified notification script for any sentinel event that is
# generated in the WARNING level (for instance -sdown, -odown, and so forth).
# This script should notify the system administrator via email, SMS, or any
# other messaging system, that there is something wrong with the monitored
# Redis systems.
#
# The script is called with just two arguments: the first is the event type
# and the second the event description.
#
# The script must exist and be executable in order for sentinel to start if
# this option is provided.
#
# Example:
#
# sentinel notification-script mymaster /var/redis/notify.sh

# CLIENTS RECONFIGURATION SCRIPT
#
# sentinel client-reconfig-script <master-name> <script-path>
#
# When the master changed because of a failover a script can be called in
# order to perform application-specific tasks to notify the clients that the
# configuration has changed and the master is at a different address.
# 
# The following arguments are passed to the script:
#
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#
# <state> is currently always "failover"
# <role> is either "leader" or "observer"
# 
# The arguments from-ip, from-port, to-ip, to-port are used to communicate
# the old address of the master and the new address of the elected replica
# (now a master).
#
# This script should be resistant to multiple invocations.
#
# Example:
#
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh

# SECURITY
#
# By default SENTINEL SET will not be able to change the notification-script
# and client-reconfig-script at runtime. This avoids a trivial security issue
# where clients can set the script to anything and trigger a failover in order
# to get the program executed.

sentinel deny-scripts-reconfig yes

# REDIS COMMANDS RENAMING
#
# Sometimes the Redis server has certain commands, that are needed for Sentinel
# to work correctly, renamed to unguessable strings. This is often the case
# of CONFIG and SLAVEOF in the context of providers that provide Redis as
# a service, and don't want the customers to reconfigure the instances outside
# of the administration console.
#
# In such case it is possible to tell Sentinel to use different command names
# instead of the normal ones. For example if the master "mymaster", and the
# associated replicas, have "CONFIG" all renamed to "GUESSME", I could use:
#
# SENTINEL rename-command mymaster CONFIG GUESSME
#
# After such configuration is set, every time Sentinel would use CONFIG it will
# use GUESSME instead. Note that there is no actual need to respect the command
# case, so writing "config guessme" is the same in the example above.
#
# SENTINEL SET can also be used in order to perform this configuration at runtime.
#
# In order to set a command back to its original name (undo the renaming), it
# is possible to just rename a command to itsef:
#
# SENTINEL rename-command mymaster CONFIG CONFIG

```

使用redis-sentinel 启动哨兵服务



###### 3、优点

- 哨兵集群，基于主从复制模式，所有的主从配置他都有
- 主从切换，故障可以转移，系统的可用性会更好
- 哨兵模式就是主从模式的升级，手动到自动，更加健壮



## 八、Redis缓存穿透和雪崩

服务器高可用

查不到

![image-20210403232501378](/Users/gaoshang/Library/Application Support/typora-user-images/image-20210403232501378.png)



解决办法



![image-20210403232533178](/Users/gaoshang/Library/Application Support/typora-user-images/image-20210403232533178.png)



![image-20210403232556125](/Users/gaoshang/Library/Application Support/typora-user-images/image-20210403232556125.png)





量太大

![image-20210403232649062](/Users/gaoshang/Library/Application Support/typora-user-images/image-20210403232649062.png)



![image-20210403232721157](/Users/gaoshang/Library/Application Support/typora-user-images/image-20210403232721157.png)





![image-20210403232813258](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6z4mdkqvj30tx0iitgt.jpg)



![image-20210403232910752](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6z5m77nvj30sy07ftdt.jpg)



![image-20210403233011872](https://tva1.sinaimg.cn/large/008eGmZEgy1gp6z6nqx1uj30so06vdk9.jpg)





[ElasticSearch](https://www.bilibili.com/video/BV17a4y1x7zq?p=1)





























