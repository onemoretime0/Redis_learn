# 1.NoSQL的四大分类

**KV键值对：**

- 新浪：Redis
- 美团：Redis+Tair
- 阿里、百度：Redis+memecache

**文档型数据库（bson格式，和json格式相同）**

- MongoDB
	- MongoDB是一个基于分布式文件存储的数据库，C++编写，主要用来处理大量的文档数据
	- MongoDB是一个介于关系型数据库和非关系型数据库中间的产品
	- MongoDB是NoSQL数据库中功能最丰富，最像关系型数据库的

**列存储数据库**

- Hbase
- 分布式文件系统

**图关系数据库**

![](.\image\社交拓扑图.png)

## 

- 它不是存储图形的，而是存放的是关系，比如：朋友圈社交，广告推荐
- Neo4j、infoGrid

# 2.Redis

## Redis是什么？

Redis（Remote Dictionary Server )，即远程字典服务，是一个开源的使用ANSI [C语言](https://baike.baidu.com/item/C语言)编写、支持网络、可基于内存亦可持久化的日志型、Key-Value[数据库](https://baike.baidu.com/item/数据库/103728)，并提供多种语言的API。

redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步

官方提供测试数据，50个并发执行100000个请求,读的速度是110000次/s,写的速度是81000次/s ，且Redis通过提供多种键值数据类型来适应不同场景下的存储需求

> 官方:Redis是一个开源（BSD许可），内存存储的数据结构服务器，可用作数据库，高速缓存和消息队列代理。它支持[字符串](https://www.redis.net.cn/tutorial/3508.html)、[哈希表](https://www.redis.net.cn/tutorial/3509.html)、[列表](https://www.redis.net.cn/tutorial/3510.html)、[集合](https://www.redis.net.cn/tutorial/3511.html)、[有序集合](https://www.redis.net.cn/tutorial/3512.html)，[位图](https://www.redis.net.cn/tutorial/3508.html)，[hyperloglogs](https://www.redis.net.cn/tutorial/3513.html)等数据类型。内置复制、[Lua脚本](https://www.redis.net.cn/tutorial/3516.html)、LRU收回、[事务](https://www.redis.net.cn/tutorial/3515.html)以及不同级别磁盘持久化功能，同时通过Redis Sentinel提供高可用，通过Redis Cluster提供自动[分区](https://www.redis.net.cn/tutorial/3524.html)。



## Reids支持的数据结构

- 字符串（string）
- 哈希类型（hash）
- 列表类型（list）
- 集合类型（set）
- 有序集合类型（sortset）

## Rdis可以干什么?

- 内存存储、持久化，【因为内存是断电即失的，所以持久化非常重要（rdb、aof）】
- Redis效率高、可以用于高速缓存
- 发布订阅系统
- 地图信息分析
- 计时器、计数器（微信、微博的浏览量）
- 聊天室的在线好友列表
- 任务队列。（秒杀、抢购、12306等等）
- 应用排行榜
- 网站访问统计
- 数据过期处理（可以精确到毫秒
- 分布式集群架构中的session分离

## Redis的特性

- 多样化的数据结构
- 持久化
- 集群
- 事务

。。。。

## Redi在Linux下的安装配置

> 安装参考以前的笔记。

安装好的路径默认在`usr/local/bin`目录：

![](.\image\linxu下redis的安装路径.png)

在此目录先新建一个目录`h_config`用来存放我们的配置文件，将解压目录下的redis.conf配置文件，复制一份到`h_config`目录下，之后的配置都是基于`/usr/local/bin/h_config/redis.conf`进行配置



### 配置后台启动

> redis默认不是后台启动的，所以需要配置后台启动

修改redis.conf配置文件设置后台启动

````bash
#将配置文件中的这一项改成yes，保存并推出
daemonize yes
````

启动服务：

```shell
/usr/local/bin/redis-server /usr/local/bin/h_config/redis-conf
#指定配置文件启动
#关闭redis
shutdown
#这样在redis-cli中执行此命令，会关闭redis的进程
```

> 配置开机自启查看之前的笔记



### redis-benchmark性能测试

redis-benchmark是一个而压力测试工具，`/usr/local/bin`路径下

使用：

```shell
redis-benchmark 参数
```

| 序号 | 选项      | 描述                                       | 默认值    |
| :--- | :-------- | :----------------------------------------- | :-------- |
| 1    | **-h**    | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**    | 指定服务器端口                             | 6379      |
| 3    | **-s**    | 指定服务器 socket                          |           |
| 4    | **-c**    | 指定并发连接数                             | 50        |
| 5    | **-n**    | 指定请求数                                 | 10000     |
| 6    | **-d**    | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| 7    | **-k**    | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**    | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**    | 通过管道传输 <numreq> 请求                 | 1         |
| 10   | **-q**    | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **--csv** | 以 CSV 格式输出                            |           |
| 12   | **-l**    | 生成循环，永久执行测试                     |           |
| 13   | **-t**    | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | **-I**    | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

# 3.Redis的基本知识

- redis有16个数据库，默认使用第0个数据库
	- 可以使用`select`命令来切换数据库
	- 使用`DBSIZE`查看但钱数据库大小
- 清空当前数据库：`flushdb`
- 清空所有的数据库：`FLUSHALL`



**Redis是单线程的**

> Redis的速度是非常快的，官方表示，Redis是基于内存操作的，CPU并不是Redis的性能瓶颈。Redis的瓶颈是有机器的内存和网络决定的。既然可以使用单线程来实现，那么就使用单线程来实现

**Redis为什么单线程还这么快？**

> 误区1：很多人认为高新能的服务器一定是多线程的
>
> 误区2：多线程一定比单线程效率高

- Redis是将所有的数据放在内存中的，所以说使用单线程去操作效率是最高的
- 多线程会产CPU的上下文切换（这是一个耗时的操作），对于内存来说，如果没有上下问切换效率就是最高的。
- 多次读写都是在一个CPU上的，在相同内存的情况下，这就是最佳方案！

# 4.Redis-key命令

| 命令                | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| `DEL key`           | 该命令用于在 key 存在是删除 key。                            |
| `DUMP key`          | 序列化给定 key ，并返回被序列化的值。                        |
| `EXISTS key`        | 检查给定 key 是否存在。                                      |
| `EXPIRE key`        | seconds 为给定 key 设置过期时间。                            |
| `KEYS pattern`      | 查找所有符合给定模式( pattern)的 key 。                      |
| `MOVE key db`       | 将当前数据库的 key 移动到给定的数据库 db 当中。              |
| `PERSIST key`       | 移除 key 的过期时间，key 将持久保持。                        |
| `TTL key`           | 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 |
| `RANDOMKEY`         | 从当前数据库中随机返回一个 key                               |
| `TYPE key`          | 返回 key 所储存的值的类型。                                  |
| `RENAME key newkey` | 修改 key 的名称                                              |

 

日常使用这些命令已经足够了，更过详情查看官方文档：[官方文档](https://www.redis.net.cn/tutorial/3507.html)

# 5.基本数据类型详解

## String

**基础操作：**

```bash
#基础操作
APPEND key value 			#如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原								来的值的末尾。如果key不存在，则相当于set key value 
STRLEN key 					#返回 key 所储存的字符串值的长度。 
```

**改变值的大小：**

````bash
#改变的值的大小==>自增，自减，步长
INCR key 					#将 key 中储存的数字值增一。	==>(i++)
DECR key 					#将 key 中储存的数字值减一	==>(i--)
INCRBY key increment 		#将 key 所储存的值加上给定的增量值（increment）	==> (i+=increment)
DECRBY key decrement 		#key 所储存的值减去给定的减量值（decrement）	==>(i-=increment)
````

**字符串范围：**

````bash
#字符串范围 range
#获取
GETRANGE key start end 		#返回 key 中字符串值的子字符，从索引start 到end（0,-1）查看所有字符串

#替换
SETRANGE key offset value 	#用 value 参数替换给定 key 所储存的字符串值，从offset开始
````

**serex和setnx：**

```bash
#setnx(存在) 和 setex(存在)
SETNX key value 			#只有在 key 不存在时设置 key 的值。(在分布式锁中常用)
SETEX key seconds value 	#将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。
127.0.0.1:6379> setex k2 20 'Hello'	#设置的k2的值,并设置了过期时间
OK
127.0.0.1:6379> setnx k2 'world'	#使用SETNX重新设置k2的值，k2的已经过期之后才会将'world'赋值到k2
(integer) 1
127.0.0.1:6379> get k2
"world"
127.0.0.1:6379>
```

**批量操作：**

```bash
#批量操作	mset	mget
MSET key value [key value ...] 		#同时设置一个或多个 key-value 对。
MGET key1 [key2..] 					#获取所有(一个或多个)给定 key 的值。
MSETNX key value [key value ...] 	#同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。

127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k1"
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k4 v4 k1 v_other		#只有当所有的key都不存在的时候，才会设置成功，否则就会失败
(integer) 0
127.0.0.1:6379> keys *		#msetnx是一个原子性草走
1) "k2"
2) "k3"
3) "k1"
127.0.0.1:6379> msetnx k4 v4 k5 v5
(integer) 1
127.0.0.1:6379> keys *
1) "k2"
2) "k3"
3) "k1"
4) "k4"
5) "k5"
127.0.0.1:6379>

#对象
set user:1 {name:'张三'，age:18}	#设置一个user:1对象，值为json字符串来保存一个对象

127.0.0.1:6379> mset user:1:name zhangsan user:1:age 18
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "18"
127.0.0.1:6379>

#这里的key是一个巧妙的设计：user:{id}:{name},这样设计在redis中是完全OK的！
```

**组合命令：**

```bash
#组合命令 getset ==>先get 再 set
GETSET key value 将给定 key 的值设为 value ，并返回 key 的旧值(old value)。

127.0.0.1:6379> getset db redis		#key不存在，返回nil,并设置key的值为vlaue
(nil)
127.0.0.1:6379> get db
"redis"
127.0.0.1:6379> getset db MongoDB		#如果key存在，覆盖key的值，并返回value 的旧值(oldValue)
"redis"
127.0.0.1:6379>127.0.0.1:6379> get db
"MongoDB"
127.0.0.1:6379>
```

**string的使用场景：**value可以是字符串，也可以是数字

- 计数器
- 统计多单位的数量

## List

在redis中可以将list玩成队列、栈、阻塞队列。

>- Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）
>- 一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。



**基础命令：**

```bash
#基础命令
#添加
LPUSH key value1 [value2] 		#将一个或多个值插入到列表头部
LRANGE key start stop 			#获取列表指定范围内的元素
RPUSH key value1 [value2] 		#在列表中添加一个或多个值(在尾部添加)
############################################################################################
#移除
LPOP key 						#移出并获取列表的第一个元素（头部）
RPOP key 						#移除并获取列表最后一个元素（尾部）
#移除指定的值
LREM key count value 			#移除列表元素==>count表示要删除的数量，value表示要删除的具体的值

127.0.0.1:6379> lpush lit one two three four three one one one	#像列表中添加元素，元素可以重复
(integer) 8
127.0.0.1:6379> lrem lit 1 two		#删除一个value = two元素
(integer) 1
127.0.0.1:6379> lrange lit 0 -1
1) "one"
2) "one"
3) "one"
4) "three"
5) "four"
6) "three"
7) "one"
127.0.0.1:6379> lrem lit 3 one		#删除三个value = one的元素
(integer) 3
127.0.0.1:6379> lrange lit 0 -1
1) "three"
2) "four"
3) "three"
4) "one"
127.0.0.1:6379>

############################################################################################
#获取值
LINDEX key index 				#通过索引获取列表中的元素
LRANGE key start stop 			#获取列表指定范围内的元素
############################################################################################
#列表长度
LLEN key  						#获取列表长度
```



**对列表修剪：**

```bash
#对列表进行修剪，只保留一部分
LTRIM key start stop 		#对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区							间之内的元素都将被删除。

```

**移除一个元素并添加到另外一个列表:**

````bash
RPOPLPUSH source destination 			#移除列表的最后一个元素，并将该元素添加到另一个列表并返回

127.0.0.1:6379> rpush list one two three four		#这操作就像是从栈顶弹出一个元素，然后压到另一个列表中
(integer) 4
127.0.0.1:6379> lrange list 0 -1
1) "one"
2) "two"
3) "three"
4) "four"
127.0.0.1:6379> rpoplpush list listother
"four"
127.0.0.1:6379> lrange listother 0 -1
1) "four"
127.0.0.1:6379>
````

**更新操作：**

```bash
LSET key index value 		#通过索引设置列表元素的值

127.0.0.1:6379> lset lirt 0 item
(error) ERR no such key		#报错，说明并没有这个list这个列表,
127.0.0.1:6379>				#list中没有直接进行判断的命令，但是可以使用这种方法来进行判断存不存在
127.0.0.1:6379> lpush list value1		#创建list列表，并赋值
(integer) 1
127.0.0.1:6379> lrange list 0 0
1) "value1"
127.0.0.1:6379> lset list 0 item		#修改index = 0的值为item
OK
127.0.0.1:6379> lrange list 0 0
1) "item"
127.0.0.1:6379>
```

**插入值：**

```bash
LINSERT key BEFORE|AFTER pivot value 		#在列表的元素前或者后插入元素
```

> list小结
>
> - redis中的list实际上是一个链表
> - 如果key不存在，则创建新的链表
> - 如果key存在，新增内容
> - 如果移除了所有值，空链表，也代表不存在
> - 在两边插入值或改动值，效率最高，中间元素，相对来说效率要低一点
> - 使用list可以做做消息队列（LPUSH RPOP）



## Set



> - Redis的Set是string类型的无序集合。集合成员是唯一的，这就意味着集合中不能出现重复的数据。
> - Redis 中 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。
> - 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)



**基础命令:**

```bash
#添加操作
SADD key member1 [member2] 				#向集合添加一个或多个成员

################################################################################################

#获取操作
SMEMBERS key 							#返回集合中的所有成员
SCARD key 								#获取集合的成员数
SRANDMEMBER key [count] 				#返回集合中一个或多个随机数

127.0.0.1:6379> sadd myset onr two three zhangsan		#添加
(integer) 4
127.0.0.1:6379> smembers myset							#查看所有
1) "two"
2) "onr"
3) "three"
4) "zhangsan"
127.0.0.1:6379> scard myset								#查看元素个数
(integer) 4
127.0.0.1:6379> srandmember myset 2						#获取随机
1) "three"
2) "zhangsan"
127.0.0.1:6379> srandmember myset 2						#获取随机
1) "onr"
2) "two"
127.0.0.1:6379>

################################################################################################

#判断
SISMEMBER key member 						#判断 member 元素是否是集合 key 的成员

127.0.0.1:6379> sismember myset lisi		#myset中不存在lisi
(integer) 0
127.0.0.1:6379> sismember myset two			#myset存在two
(integer) 1
127.0.0.1:6379>

################################################################################################

#移除
SREM key member1 [member2] 					#移除集合中一个或多个成员
SPOP key 									#移除并返回集合中的一个随机元素

127.0.0.1:6379> srem myset onr			#移除onr元素
(integer) 1
127.0.0.1:6379> spop myset 1			#移除一个随机元素
1) "zhangsan"
127.0.0.1:6379> smembers myset
1) "two"
2) "three"
127.0.0.1:6379>
```



**并集：**

```bash
SUNION key1 [key2] 							#返回所有给定集合的并集
SUNIONSTORE destination key1 [key2] 		#所有给定集合的并集存储在 destination 集合中

127.0.0.1:6379> sadd set1 one two three zhangsan lisi		#集合1
(integer) 5
127.0.0.1:6379> sadd set2 one hello world redis MongoDB		#集合2
(integer) 5
127.0.0.1:6379> sunion set1 set2							#返回两个集合的并集(只返回了一个one元素)
1) "lisi"
2) "one"
3) "hello"
4) "world"
5) "zhangsan"
6) "redis"
7) "two"
8) "three"
9) "MongoDB"
127.0.0.1:6379> sunionstore doubleset set1 set2			#将set1和set2的并集存储到doubleset中
(integer) 9										
127.0.0.1:6379> smembers doubleset						#doubleset中只有一个one元素
1) "lisi"
2) "one"
3) "hello"
4) "world"
5) "zhangsan"
6) "redis"
7) "two"
8) "three"
9) "MongoDB"
127.0.0.1:6379>
```

**交集:**

````bash
SINTER key1 [key2] 									#返回给定所有集合的交集
SINTERSTORE destination key1 [key2] 				#返回给定所有集合的交集并存储在 destination 中

````

**差集：**

```bash
SDIFF key1 [key2] 									#返回给定所有集合的差集
SDIFFSTORE destination key1 [key2] 					#返回给定所有集合的差集并存储在 destination 中

127.0.0.1:6379> sdiff set1 set2				#返回的是set1不与set2相交的部分
1) "two"
2) "lisi"
3) "three"
4) "zhangsan"
127.0.0.1:6379> sdiff set2 set1				#返回的是set2中不与set1相交的部分
1) "redis"
2) "MongoDB"
3) "world"
4) "hello"
127.0.0.1:6379> sdiffstore diffset set1 set2
(integer) 4
127.0.0.1:6379> smembers diffset
1) "two"
2) "lisi"
3) "three"
4) "zhangsan"
127.0.0.1:6379>
```

## Hash

> - value中存储的是map
> - Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。
> - Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）



**添加操作：**

```bash
#添加操作
HSET key field value 						#将哈希表 key 中的字段 field 的值设为 value 。
HMSET key field1 value1 [field2 value2 ] 	#同时将多个 field-value (域-值)对设置到哈希表 key 中。

HSETNX key field value 						#只有在字段 field 不存在时，设置哈希表字段的值。
```

**获取操作：**

````bash
HLEN key 						#获取哈希表中字段的数量

HVALS key 						#获取哈希表中所有值

HKEYS key 						#获取所有哈希表中的字段
HGETALL key 					#获取在哈希表中指定 key 的所有字段和值

HGET key field 					#获取存储在哈希表中指定字段的值
HMGET key field1 [field2] 		#获取所有给定字段的值

````

**删除操作：**

```bash
HDEL key field2 [field2]		#删除一个或多个哈希表字段
```

**判断:**

```bash
HEXISTS key field 				#查看哈希表 key 中，指定的字段是否存在
```

**更新操作：**

```bash
HINCRBY key field increment 			#为哈希表 key 中的指定字段的整数值加上增量 increment 。
HINCRBYFLOAT key field increment 		#为哈希表 key 中的指定字段的浮点数值加上增量 increment 。


```

**迭代：**

```bash
HSCAN key cursor [MATCH pattern] [COUNT count] 		#迭代哈希表中的键值对。
```



## sorted Set(Zset)

>- Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。
>- 不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。
>- 有序集合的成员是唯一的,但分数(score)却可以重复。
>- 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。



**添加操作：**

```bash
ZADD key score1 member1 [score2 member2] 		#向有序集合添加一个或多个成员，或者更新已存在成员的分数
```

**获取操作：**

```bash
#成员数的获取
ZCARD key 							#获取有序集合的成员数
ZCOUNT key min max 					#计算在有序集合中指定区间分数的成员数
ZLEXCOUNT key min max 				#在有序集合中计算指定字典区间内成员数量

#获取成员
ZRANGE key start stop [WITHSCORES] 					#通过索引区间返回有序集合成指定区间内的成员
ZRANGEBYLEX key min max [LIMIT offset count] 		#通过字典区间返回有序集合的成员
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] 		#通过分数返回有序集合指定区间内的成员
#排列
ZREVRANGE key start stop [WITHSCORES] 			   #通过索引返回有序集中指定区间内的成员，分数从高到底
ZREVRANGEBYSCORE key max min [WITHSCORES] 		   #返回有序集中指定分数区间内的成员，分数从高到低排序

ZREVRANK key member 				#返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序

#返回member
ZSCORE key member 							#返回有序集中，成员的分数值

#返回索引值
ZRANK key member 							#返回有序集合中指定成员的索引

```

**移除操作：**

```bash
ZREM key member [member ...] 					#移除有序集合中的一个或多个成员
#通过给定字典区间
ZREMRANGEBYLEX key min max 						#移除有序集合中给定的字典区间的所有成员
#通过排名
ZREMRANGEBYRANK key start stop 					#移除有序集合中给定的排名区间的所有成员
#通过分数
ZREMRANGEBYSCORE key min max 					#移除有序集合中给定的分数区间的所有成员
```

**更新操作：**

````bash
ZINCRBY key increment member 				#有序集合中对指定成员的分数加上增量 increment
````

**交集：**

```bash
ZINTERSTORE destination numkeys key [key ...] 
#计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中
```

**并集：**

```bash
UNIONSTORE destination numkeys key [key ...] 
#计算给定的一个或多个有序集的并集，并存储在新的 key 中
```

**迭代：**

```bash
ZSCAN key cursor [MATCH pattern] [COUNT count] 
#迭代有序集合中的元素（包括元素成员和元素分值）
```



> sorted set 应用思路：普通消息 1 ，重要消息 2 ，加权判断
>
> ​	排行榜



# 6.三种特殊数据类型

## geospatial 地理位置

> 思考:微信的朋友定位,附近的人以及滴滴打车时候距离的计算都是怎么实现的?

地球上的地理位置是使用二维的经纬度表示，经度范围 (-180, 180]，纬度范围 (-90, 90]，只要我们确定一个点的经纬度就可以名曲他在地球的位置

例如打车，最直观的操作就是实时记录更新各个车的位置，然后去我们要找车时，在数据库中查找距离我们(坐标x0,y0)附近r公里的车辆,使用如下SQL即可：`select cart from position where x0-r < x < x0+r and y0-r < y < y0+r`

但是这样会有什么问题呢？

- 一是查询性能问题，如果并发高，数据量大这种查询是要搞垮数据库的
- 二是这个查询的是一个矩形访问，而不是以我为中心r公里为半径的圆形访问。
- 三是精准度的问题，我们知道地球不是平面坐标系，而是一个圆球，这种矩形计算在长距离计算时会有很大误差

Redis在3.2版本以后增加了地理位置的处理，其提供了6个地理位置相关的命令

```bash
GEOADD 					#将给定的空间元素（纬度、经度、名字）添加到指定的键里面
GEOPOS 					#从键里面返回所有给定位置元素的位置（经度和纬度）
GEODIST 				#返回两个给定位置之间的距离。
GEORADIUS 				#以给定的经纬度为中心， 返回与中心的距离不超过给定最大距离的所有位置元素。
GEORADIUSBYMEMBER 		#跟GEORADIUS类似
GEOHASH 				#返回一个或多个位置元素的 Geohash 表示。
```

官方文档:www.redis.net.cn/order/3685.html

### GEOADD

> - 有效的经度从-180度到180度。
> - 有效的纬度从-85.05112878度到85.05112878度。
>
> 当坐标位置超出上述指定范围时，该命令将会返回一个错误。

````bash
GEOADD 					#将给定的空间元素（纬度、经度、名字）添加到指定的键里面
#两极无法直接添加,一般会下载城市数据,然后使用java程序直接导入
#参数:key value(经度,纬度,名称) 
127.0.0.1:6379> geoadd china:city 116.405285 39.904989 beijing
(integer) 1
127.0.0.1:6379> geoadd china:city 118.326443 35.065282 linyi
(integer) 1
127.0.0.1:6379> geoadd china:city 121.472644 31.231706 shanghai
(integer) 1
127.0.0.1:6379>
````



### GEOPOS（获得当前的位置）

```bash
GEOPOS 					#从键里面返回所有给定位置元素的位置（经度和纬度）
```

**返回值**:GEOPOS 命令返回一个数组， 数组中的每个项都由两个元素组成： 第一个元素为给定位置元素的经度， 而第二个元素则为给定位置元素的纬度。

当给定的位置元素不存在时， 对应的数组项为空值

```bash
127.0.0.1:6379> geopos china:city shanghai							#查询一个
1) 1) "121.47264629602432251"
   2) "31.23170490709807012"
127.0.0.1:6379> geopos china:city shanghai beijing linyi			#一次性查询多个
1) 1) "121.47264629602432251"
   2) "31.23170490709807012"
2) 1) "116.40528291463851929"
   2) "39.9049884229125027"
3) 1) "118.32644194364547729"
   2) "35.06528309123243758"
127.0.0.1:6379>
```

### GEODIST

```bash
GEODIST 				#返回两个给定位置之间的距离。
```

**参数：**

- 如果两个位置之间的其中一个位置不存在，那么那么命令就会返回空值
- 指定单位的参数unit必须是以下单位的其中一个
	- m
	- km
	- mi：表示单位为英里
	- ft：表示单位为英尺
	- 如果没有显式的指定参数，默认为m

> `GEODIST` 命令在计算距离时会假设地球为完美的球形， 在极限情况下， 这一假设最大会造成 0.5% 的误差。

**返回值：**

计算出的距离会以双精度浮点数的形式被返回。 如果给定的位置元素不存在， 那么命令返回空值。

```bash
127.0.0.1:6379> geodist china:city shanghai beijing			#使用GEODIST命令默认的单位
"1067597.9668"
127.0.0.1:6379> geodist china:city shanghai beijing km		#指定单位km
"1067.5980"
127.0.0.1:6379>
```

### GEORADIUS

```bash
GEORADIUS 				#以给定的经纬度为中心， 返回与中心的距离不超过给定最大距离的所有位置元素。
```

单位的设定与GEODIST命令相同。

**WITH可选项（在给定以下可选项的时候，命令会返回额外的信息）**：

- `WITHDIST`：在返回位置元素的同时，将位置元素与中心位置之间的距离一并返回，距离单位与用户给定的单位设置一致
- `WITHCOORD`：将位置元素与经度和纬度一并返回
- `WITHHASH`： 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。

**参数指定排序方式：**

命令默认返回未排序的位置元素。 通过以下两个参数， 用户可以指定被返回位置元素的排序方式

- `ASC`：根据中心的位置，按照由近到远的顺序返回元素
- `DESC`：根据中心位置，按照从远到近的顺序返回元素

在默认情况下， GEORADIUS 命令会返回所有匹配的位置元素。 虽然用户可以使用 **COUNT `<count>`** 选项去获取前 N 个匹配元素， 但是因为命令在内部可能会需要对所有被匹配的元素进行处理， 所以在对一个非常大的区域进行搜索时， 即使只使用 `COUNT` 选项去获取少量元素， 命令的执行速度也可能会非常慢。 但是从另一方面来说， 使用 `COUNT` 选项去减少需要返回的元素数量， 对于减少带宽来说仍然是非常有用的。

**返回值：**

- 在没有给定任何 `WITH` 选项的情况下， 命令只会返回一个像 [“New York”,”Milan”,”Paris”] 这样的线性（linear）列表。
- 在指定了 `WITHCOORD` 、 `WITHDIST` 、 `WITHHASH` 等选项的情况下， 命令返回一个二层嵌套数组， 内层的每个子数组就表示一个元素。
	- 在返回嵌套数组时， 子数组的第一个元素总是位置元素的名字。 至于额外的信息， 则会作为子数组的后续元素， 按照以下顺序被返回：
		- 以浮点数格式返回的中心与位置元素之间的距离， 单位与用户指定范围时的单位一致
		- geohash 整数。
		- 由两个元素组成的坐标，分别为经度和纬度。

```bash
127.0.0.1:6379> georadius china:city 111 31 1000 km	#查询东经111，北纬31为中心，1000km范围内的城市
1) "shanghai"
2) "linyi"

#查询东经111，北纬31为中心，1000km范围内的城市，并返回距离
127.0.0.1:6379> georadius china:city 111 31 1000 km withdist
1) 1) "shanghai"
   2) "997.2023"
2) 1) "linyi"
   2) "818.8696"

##查询东经111，北纬31为中心，1000km范围内的城市，并返回位置的坐标经纬度
127.0.0.1:6379> georadius china:city 111 31 1000 km withcoord
1) 1) "shanghai"
   2) 1) "121.47264629602432251"
      2) "31.23170490709807012"
2) 1) "linyi"
   2) 1) "118.32644194364547729"
      2) "35.06528309123243758"
      
###查询东经111，北纬31为中心，1000km范围内的城市，并返回位置的坐标经纬度，只显示一条数据
127.0.0.1:6379> georadius china:city 111 31 1000 km withcoord count 1
1) 1) "linyi"
   2) 1) "118.32644194364547729"
      2) "35.06528309123243758"
127.0.0.1:6379>
```

### GEOHASH

```bash
GEOHASH 				#返回一个或多个位置元素的 Geohash 表示。
```

该命令将返回11个字符的Geohash字符串，所以没有精度Geohash，损失相比，使用内部52位表示。返回的geohashes具有以下特性：

- 他们可以缩短从右边的字符。它将失去精度，但仍将指向同一地区。
- 可以在 `geohash.org` 网站使用，网址 `http://geohash.org/`。查询例子：http://geohash.org/sqdtr74hyu0.
- 与类似的前缀字符串是附近，但相反的是不正确的，这是可能的，用不同的前缀字符串附近。

**返回值：**

此命令返回一个**标准的Geohash**

一个数组， 数组的每个项都是一个 geohash 。 命令返回的 geohash 的位置与用户给定的位置元素的位置一一对应。

```bash
127.0.0.1:6379> geohash china:city beijing
1) "wx4g0b7xrt0"
127.0.0.1:6379>
```

### GEORADIUSBYMEMBER

```bash
GEORADIUSBYMEMBER 		#跟GEORADIUS类似
```

这个命令和 `GEORADIUS`命令一样， 都可以找出位于指定范围内的元素， 但是 `GEORADIUSBYMEMBER` 的中心点是由给定的位置元素决定的， 而不是像 `GEORADIUS`那样， 使用输入的经度和纬度来决定中心点

**指定成员的位置被用作查询的中心。**

```BASH
127.0.0.1:6379> georadiusbymember china:city beijing 1000 km
1) "linyi"
2) "beijing"
127.0.0.1:6379>
```

## HyperLogLog 

### Hyperloglog 是什么？

> 什么是基数？
>
> 基数（cardinal number）在数学上，是[集合论](https://baike.baidu.com/item/集合论/494533)中刻画任意[集合](https://baike.baidu.com/item/集合/2908117)大小的一个概念。两个能够建立元素间一一对应的集合称为互相对等集合。例如3个人的集合和3匹马的集合可以建立[一一](https://baike.baidu.com/item/一一/2702379)[对应](https://baike.baidu.com/item/对应)，是两个[对等](https://baike.baidu.com/item/对等/4198791)的集合。
>
> 根据对等这种关系对[集合](https://baike.baidu.com/item/集合)进行分类，凡是互相对等的集合就划入同一类。这样，每一个集合都被划入了某一类。任意一个集合A所属的类就称为集合A的基数，记作|A|（或cardA)。这样，当A 与B同属一个类时，A与B 就有相同的基数，即|A|=|B|。而当 A与B不同属一个类时，它们的基数也不同。

redis 2.8.9版本加入了Hyperloglog数据结构~

Hyperloglog是用来做技术统计的算法

**优点：**Hyperloglog占用的内存是固定的，保存2^64不同的基数，只需要12kb

**具体的应用场景分析：**

例如在同统计页面的访问量UV（一个人访问一个网站多个，网站的浏览量还是算一个人）

- 传统的方式，set保存用户的id，然后就可以统计set中元素的数量作为标准，但是这种方式如果保存大量的用户id就会比较麻烦。我们的目的是计数，而不是保存用户的id

### Pfadd

> Pfadd 命令将所有元素参数添加到 HyperLogLog 数据结构中。

**语法：**

```bash
PFADD key element [element ...]
```

```bash
127.0.0.1:6379> pfadd mykey a b c f g as d  g
(integer) 1
127.0.0.1:6379> pfcount mykey
(integer) 7
127.0.0.1:6379>
```

###  Pfcount

> Pfcount 命令返回给定 HyperLogLog 的基数估算值。

### Pgmerge

> Pgmerge 命令将多个 HyperLogLog 合并为一个 HyperLogLog ，合并后的 HyperLogLog 的基数估算值是通过对所有 给定 HyperLogLog 进行并集计算得出的。

**语法：**

```bash
PFMERGE destkey sourcekey [sourcekey ...]
```

```bash
127.0.0.1:6379> pfadd key1 v b b as d j t d e
(integer) 1
127.0.0.1:6379> pfadd key2 a b v d d g i o p t
(integer) 1
127.0.0.1:6379> pfcount key1
(integer) 7
127.0.0.1:6379> pfcount key2
(integer) 8
127.0.0.1:6379> pfmerge key3 key1 key2		#key3是合并的key1 和key2 ，是并集
OK
127.0.0.1:6379> pfcount key3
(integer) 11
127.0.0.1:6379>
```

## Bitmap

> 位图，也是一种数据结构，（位存储），所有的操作都是来操作二进制为来进行记录的，只有0和1两个状态

**应用场景分析：**

只有两个状态的都可以用Bitmap来存储，像qq中用户是否登录，员工的打卡状态等等。。。

基本使用：

- `setbit key offset value`
- `getbit key offset`
- `bitcount key [start end]`



```bash
127.0.0.1:6379> setbit sign 0 1				#周一已打卡
(integer) 0
127.0.0.1:6379> setbit sign 1 0				#周二没有打卡
(integer) 0
127.0.0.1:6379> setbit sign 2 0
(integer) 0
127.0.0.1:6379> setbit sign 3 1
(integer) 0
127.0.0.1:6379> setbit sign 4 1
(integer) 0
127.0.0.1:6379> setbit sign 5 1
(integer) 0
127.0.0.1:6379> setbit sign 6 1
(integer) 0
127.0.0.1:6379> getbit sign 2		#查看周三有没有打卡
(integer) 0
127.0.0.1:6379> bitcount sign		#统计打卡天数（还可以设置从start end 的值）
(integer) 5
127.0.0.1:6379>
```

