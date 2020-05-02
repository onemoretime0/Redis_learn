# 1.Redis.conf详解

> Redis的启动是通过redis.conf配置文件来启动的



**redis.conf中可以配置的信息：**

- 基本的单位的配置
- 配置文件导入的配置
- 网络配置
- 通用配置
- 持久化配置
	- rdb
	- aof
- 主从复制
- 安全
- 客户端
- 内存

```bash
###############################第一部分：单位################################
#单位，并且redis对大小写是不敏感的，1GB 1Gb 1gB作为单位都是可以的
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.

###############################第二部分：配置文件导入（INCLUDES）################################
#Redis可以包含多个配置文件，可以将其他配置文件引入 
# include /path/to/local.conf
# include /path/to/other.conf

###############################第三部分：网络配置（NETWORK）################################
#网络配置，可以访问其他服务器上的Redis服务
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
bind 127.0.0.1			#原始的配置文件已经配置了一个本地的
protected-mode yes		#是否受保护的模式，默认开启
port 6379				#端口号默认6379

###############################第四部分：通用配置（GENERAL）################################

daemonize yes							#以守护进程的方式运行，默认是no，需要手动开启
pidfile /var/run/redis_6379.pid			#如果以守护进程的方式运行，需要指定的PID文件

#日志级别
# Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)			debug用于测试和开发阶段
# verbose (many rarely useful info, but not a mess like the debug level)	
# notice (moderately verbose, what you want in production probably)				生产环境
# warning (only very important / critical messages are logged)	
loglevel notice							#配置日志级别
logfile ""								#生成的日志文件名

databases 16							#数据库数量，默认是16

always-show-logo yes					#是否显示log，启动redis服务时的log

###############################第四部分：快照（SNAPSHOTTING）################################

#持久化，在规定的时间内执行了多少次操作，则会持久化到文件，.rdb和.aof文件，

#这里配置的是持久化规则
save 900 1			#如果 900 秒内，至少有 1 个key及进行了修改，就进行持久化操作
save 300 10			#如果 300 秒内，至少有 10 个key及进行了修改，就进行持久化操作
save 60 10000		#如果 60 秒内，至少有 10000 个key及进行了修改，就进行持久化操作

stop-writes-on-bgsave-error yes			#持久化出现错误，是否继续工作

rdbcompression yes						#是否压缩.rdb文件，需要消耗一些CPU资源

rdbchecksum yes							#保存rdb文件的时候，是否进行错误的检查校验

dir ./									#持久化生成文件保存的目录，默认是当前目录下

###############################第五部分：主从复制（REPLICATION）################################

# replicaof <masterip> <masterport>   配置主机的IP和主机的端口号

###############################第六部分：安全（SERCITY）################################
#在这部分可以设置密码，Redis默认时没有密码的

# requirepass foobared		密码
#但是一般情况下我们会通过命令来设置密码：config set requirepass password 
#获取密码：config get requirepass
#登录命令：auth password

###############################第七部分：客户端（CLIENTS）################################

# maxclients 10000		默认的最大客户端连接数

###############################第八部分：内存（MEMORY MANAGEMENT）################################
#关于内存的配置		
# maxmemory <bytes>		Redis配置的最大内存容量

# maxmemory-policy noeviction			内存到达上限的处理策略
	#六种策略：
		#volatile-lru：只对设置了过期时间的key进行LRU（默认值） 
        #allkeys-lru ： 删除lru算法的key   
        #volatile-random：随机删除即将过期key   
        #allkeys-random：随机删除   
        #volatile-ttl ： 删除即将过期的   
        #noeviction ： 永不过期，返回错误
        
###############################第九部分：AOF设置（APPEND ONLY MODE）##############################
#AOF也是一种持久化策略
appendonly no		#默认是不开启的，默认是使用rdb持久化方式的，在大多数情况下rdb完全够用
appendfilename "appendonly.aof"		#持久化文件的名字

# appendfsync always		#每次修改都会sync，消耗资源
appendfsync everysec		#每秒执行一次sync，可能会丢失一秒的数据		Redis默认
# appendfsync no			#不会执行sync， 这个时候操作系统自己同步数据，速度最快的
```

# 2.Redis持久化

==**面试和工作中,持久化都是重点**==

> Redis是内存数据库,如果不将内存中的数据保存到硬盘中,那么一旦服务器进程推退出,服务器的数据库状态也会消失,所以Redis提供共了持久化功能

## RDB(Redis DataBase)



### RDB介绍



在指定的时间间隔内将内存中共的数据集快照写入硬盘中,也就是行话说的Snapshot快照,它恢复时是将快照文件直接读到内存中.

- Redis会单独创建(fork)一个子进程来进行持久化，先将数据写入到一个临时文件中，待持久化过程都结束了 ，在用这个临时文件替换上次持久化好的文件。
- 整个过程中，主进程是不进行任何IO操作，这就确保了极高的性能。
- 如果需要大规模的数据的恢复，且对数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加高效
- RDB的缺点是最后一次持久化的数据可能会丢失

==RDB持久化保存的文件就是dump.rdb==在生产环境中，一般会将这个文件进行备份，如果这个文件被破坏了可以使用redis-check-rdb来进行修复

![](.\image\RDB持久化过程.png)

### 配置RDB

RDB都是在redis.conf配置文件中进行配置的

```bash
#这里配置的是持久化规则
save 900 1			#如果 900 秒内，至少有 1 个key及进行了修改，就进行持久化操作
save 300 10			#如果 300 秒内，至少有 10 个key及进行了修改，就进行持久化操作
save 60 10000		#如果 60 秒内，至少有 10000 个key及进行了修改，就进行持久化操作

stop-writes-on-bgsave-error yes			#持久化出现错误，是否继续工作

rdbcompression yes						#是否压缩.rdb文件，需要消耗一些CPU资源

rdbchecksum yes							#保存rdb文件的时候，是否进行错误的检查校验

dir ./									#持久化生成文件保存的目录，默认是当前目录下
```

### RDB触发机制

> 在什么情况下会创建dump.rdb文件

- saved的规则满足的情况下，会触发rdb规则
- 执行了flushall命令也会触发EDB规则，生成dump.rdb文件
- 退出Redis也会触发RDB规则，生成dump.rdb文件

**如何恢复rdb文件？**

只需要将rdb文件放在redis.conf配置文件中配置的rdb文件目录中即可，Redis启动的时候会自动检查dump.rdb文件并恢复其中的数据

```bash
#可以通过命令查看rdb文件存放的位置
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/bin"			#只要这个下有dump.rdb文件，启动Redis就会自动回复其中的数据
127.0.0.1:6379>
```

### RDB的优缺点

优点：

- 适合大规模的数据恢复
- 如果对数据的完整性不高可以使用RDB

缺点：

- 需要一定的时间间隔进行操作，如果redis宕机了，那么最后依次修改的数据就没有了
- fork进程的时候，会占用一定的内存空间



## AOF（Append Only File）



### 介绍

> AOF会将我们所有的命令都记录下来，类似于history,恢复的时候会将所有的写命令执行一遍

AOF是以日志的形式记录每一个写操作，将Redis执行过程中的所有命令都记录下来（读操作不记录），只许追加文件但不可以改写文件，Redis启动之处初会读取文件重新构建数据，换言之，Redis重启是根据日志文件的内容将指令从前到后依次完成数据的恢复工作。

==AOF保存的是appendonly.aof文件==

![](.\image\AOF持久化.png)

### 配置



Redis中AOF默认是不开启的需要手动进行配置

```bash
#AOF也是一种持久化策略
appendonly no		#默认是不开启的，默认是使用rdb持久化方式的，在大多数情况下rdb完全够用
appendfilename "appendonly.aof"		#持久化文件的名字

#AOF持久化的策略
# appendfsync always		#每次修改都会sync，消耗资源
appendfsync everysec		#每秒执行一次sync，可能会丢失一秒的数据		Redis默认
# appendfsync no			#不会执行sync， 这个时候操作系统自己同步数据，速度最快的

#重写的规则
no-appendfsync-on-rewrite no	#重写的规则，重写的时候是否运行append，可以保证数据的安全性

auto-aof-rewrite-percentage 100		
auto-aof-rewrite-min-size 64mb		#如果aof文件大于64m，就fork一个新的进程来将我们的文件进行重写
```

在开启了AOF之后，其他的AOF配置项在一般的需求下保持默认即可，除非有特殊的需求

### 测试

配置完AOF后重启Redis服务

```bash
#向reids中set三个key
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> set k3 v3

##########################################
#查看appendonly.aof文件
[root@centos7 bin]# cat appendonly.aof
*2
$6
SELECT
$1
0
*3
$3
set
$2
k1
$2
v1
*3
$3
set
$2
k2
$2
v2
*3
$3
set
$2
k3
$2
v3
#appendonly.aof文件中记录了完整的set操作
```

### 修复appendonly.aof文件

redis-check-aof可以用来修复appendonly.aof文件

测试：认为的破坏修改appendonly.aof文件之后，Redis是无法启动的。

![](.\image\破坏appendont文件后redis无法启动.png)

修复：

```bash
[root@centos7 bin]# ./redis-check-aof --fix appendonly.aof
'x              6e: Expected prefix '*', got: '
AOF analyzed: size=128, ok_up_to=110, diff=18
This will shrink the AOF from 128 bytes, with 18 bytes, to 110 bytes
Continue? [y/N]: yes
Successfully truncated AOF
[root@centos7 bin]# vim appendonly.aof

#再次查看appendonly.aof文件，我们认为添加的东西以及被删除了
```

再次启动Redis

```bash
[root@centos7 bin]# ./redis-server ./h_config/redis.conf
13509:C 02 May 2020 17:26:13.127 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
13509:C 02 May 2020 17:26:13.127 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=13509, just started
13509:C 02 May 2020 17:26:13.127 # Configuration loaded
[root@centos7 bin]# ./redis-cli
127.0.0.1:6379> keys *
1) "k1"
2) "k3"
3) "k2"
127.0.0.1:6379>
```

### AOF的优缺点

- 优点：每一次修改都同步，文件的完整性更加好
- 缺点：相对于数据文件来说，aof远远大于rdb，修复速度比rdb慢很多
	- AOF的运行速度也比RDB慢，



## 扩展

- RDB持久化方式能够在指定的时间间隔内对数据进行快照存储
- AOF持久化方式是记录每一次写操作，当服务器重启的时候会重新执行这些命令来恢复原始的数据，AOF命令以Redis协议追加保存每次写的操作到文件末尾，Redis还能对AOF进行后台重写，使得AOF文件的替吉不至于过大
- 只做缓存，如果希望数据在服务器运行的时候存在，可以不用任何持久化
- 同时开启两种缓存方式
	- 在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始数据，因为在通常情况下AOF保存的数据集要比RDB保存的数据集要完整
	- RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只是用AOF呢？建议不要，因为RDB更合适用于备份数据库（AOF在不断的变化不利于备份），快速重启，而且不会有AOF可能潜在的BUG，留着作为一个万一的手段
- 性能建议
	- 因为RDB文件只作为后被用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留``save 900 1``这条规则。
	- 如果Enable AOF ，好处是最恶劣的情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件即可，代价一是带来了持续的IO，二是AOF rewrite 过程中产生的数据写到新文件造成的阻塞几乎不可避免。只要硬盘许可，因该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值是64M太小了，可以设到5G以上，默认超过原大小100%重写可以改到适当的值
	- 如果不Enable AOF ，仅靠Master-Slave Repllcation 实现高可用性也可以，能省掉一大笔IO，也减少了rewrite是带来的系统波动。代价是如果Master/Slave 同时断电，会丢失十几分钟的数据，启动脚本的也要比较两个Master/Slave中的RDB文件，载入较新的哪个（微博就是这种构架）



# 3.发布订阅

## 简介

Redis发布订阅（pub/sub）是一种消息通信模式：发送者（pub）发送消息，订阅者（sub）接收消息

Redis客户端可以订阅任意数量的频道。

- 消息发送者
- 频道
- 消息订阅者

![](.\image\发布订阅.png)

下图展示了频道channel1，以及订阅这个频道的三个客户端：client1、client2、client5之间的关系：

![](.\image\channel和三个订阅者.png)

当有先的消息通过PUBLISH命令发送给频道channel1时，这个消息就会被发送给订阅它的三个客户端：

![](.\image\频道发送消息.png)

**命令：**

| 命令                                         | 描述                               |
| -------------------------------------------- | ---------------------------------- |
| `PSUBSCRIBE pattern [pattern...]`            | 订阅一个或多个符合给定模式的频道。 |
| `PUBSUB subcommand [argument [argument...]]` | 查看订阅与发布系统状态。           |
| `PUBLISH channel message`                    | 将信息发送到指定的频道             |
| `PUNSUBSCRIBE [pattern [pattern...]]`        | 退订所有给定模式的频道。           |
| `SUBSCRIBE channel [channel...]`             | 订阅给定的一个或多个频道的信息。   |
| `UNSUBSCRIBE [channel [channel...]]`         | 指退订给定的频道。                 |

## 使用

```bash
#消息订阅者
127.0.0.1:6379> psubscribe test				#订阅频道test，等待接收消息
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "test"
3) (integer) 1								#等待读取推送的信息

#消息发送者
127.0.0.1:6379> publish test "Hello,World"			#向test频道发送了一条消息"Hello,World"
(integer) 1
127.0.0.1:6379>

#查看订阅者接收的消息
127.0.0.1:6379> psubscribe test
Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "test"
3) (integer) 1
1) "pmessage"							#消息		
2) "test"								#频道
3) "test"
4) "Hello,World"						#消息的具体内容

```

## 原理

Redis是使用C实现的，通过分析Redis源码里的publish.c文件，了解发布和订阅机制的底层实现，加深Redis的理解

Redis通过`PUBLISH`、`SUBSCRIBE`、`PSUBCEIBE`等命令实现发布和订阅功能。

通过`SUBSCRIBE`命令订阅某个频道后，redis-server里维护了一个字典，字典的键就是一个一个的channel，而字典的的值则是一个链表，链表中保存了所有订阅这个频道的客户端，`SUBSCRIBE`命令的关键就是将客户端添加到指定的channel的订阅链表中

通过`PUBLISH`命令向订阅者发送消息，redis-server会使用给定的频道作为键，在它所维护的channel字典中查找记录了订阅这个频道的所有客户端的链表，遍历这个链表，将消息发送给所有订阅者。

PUB/SUB从字面上理解就是发布（publish）和订阅（subscribe），在Redis中，可以设定对某一个key值进行消息发布以及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会接收到相应的消息。这一功能最明显的就是用来做实时消息系统，比如普通的即时聊天记录，群聊功能。



## 使用场景

- 实时消息系统
- 实时聊天
- 订阅关注系统

稍微复杂的场景及使用消息中间件来做