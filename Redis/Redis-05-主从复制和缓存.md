# 1.Redis主从复制

## 概述

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为**主节点**(master/leader)，后者称为**从节点**(slave/follower)；**数据的复制是单向的，只能由主节点到从节点**。

**默认情况下，每台Redis服务器都是主节点；且一个主节点可以有多个从节点(或没有从节点)，但一个从节点只能有一个主节点。**

主从复制的作用主要包括：

1. 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
2. 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
3. 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
4. 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。



**为什么要使用主从复制：**

一般来说Redis运行于工程项目中，只使用一台Redis是万万不够的：

- 从结构上，单个Redis服务器会发生单点故障，并且一台服务器需要处理所有的请求负载，压力较大
- 从容量上，单个Redis 服务器内存容量有限，就算一台服务器的内存容量为256G，也不能将所有的内存用作Redis存储内存，一般来说单台Redis最大使用内存不应该超过20G。

电商网站上的商品，一般都是一次上传，无数次浏览，也就是“多读少写”

![](.\image\主从复制Redis构架.png)

**主从复制，读写分离，架构中经常使用一主而从**



## 环境搭建

Redis启动默认即使master：

```bash
127.0.0.1:6379> info replication						#查看当前redis的信息
# Replication					
role:master												#角色默认是master
connected_slaves:0										#目前有0个从机
master_replid:6e3fdd2181076651702e73db4feb0a899a359674
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6379>
```

配置从机需要修改配置文件，本地环境搭建是在一台服务器上模拟多个Redis服务，所以需要多个redis.conf配置文件，每个配置文件对应一个端口，对应一个redis服务。

```bash
[root@centos7 h_config]# ll
总用量 256
-rw-r--r--. 1 root root 61798 5月   2 23:05 redis79.conf		#作为master
-rw-r--r--. 1 root root 61798 5月   2 23:04 redis80.conf		#作为slave1
-rw-r--r--. 1 root root 61798 5月   2 23:04 redis81.conf		#作为slave2
-rw-r--r--. 1 root root 61798 5月   2 17:32 redis.conf
```

MASTER配置文件的修改：

````bash
logfile "6379.log"			#日志文件，因为是多个redis服务，所以日志文件不能默认为空，方便查看
dbfilename dump6379.rdb		#rdb文件
````

SLAVE配置：

```bash
port 6380								#端口
pidfile /var/run/redis_6380.pid			#PID
logfile "6380.log"						
dbfilename dump6380.rdb					
#两个SLAVE都按照端口号修改即可
```

分别按照三个配置文件启动三个Redis 服务

```bash
#启动客户端的时候需要按照端口号启动对应的Redis客户端
[root@centos7 bin]# ./redis-cli -p 6380
```

![](.\image\Redis主从复制集群搭建完成.png)

**配置一主二从**

默认情况下，每台redis服务器都是主节点；我们一般情况下只需要配置从机即可！

这里将79端口的服务作为主节点，将80、81端口的服务作为从节点

配置命令：

```bash
SLAVEOF ip port
```

配置:

```bash
#6381的配置
127.0.0.1:6381> SLAVEOF 127.0.0.1 6379
OK
127.0.0.1:6381> info replication
# Replication
role:slave				#角色
master_host:127.0.0.1	#主节点IP
master_port:6379		#主节点端口
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
slave_repl_offset:70
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:09c5d9449469e16d23dd6e1b5adce3bc73be6e07
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:70
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:70

#6380也同样这么配置

#查看主节点的info
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2						#拥有两个从节点
slave0:ip=127.0.0.1,port=6380,state=online,offset=266,lag=0		#两台从节点的信息
slave1:ip=127.0.0.1,port=6381,state=online,offset=266,lag=1
master_replid:09c5d9449469e16d23dd6e1b5adce3bc73be6e07
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:266
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:266
127.0.0.1:6379>
```

**真是在开发中的配置是在配置文件中进行配置，这样配置就是永久的配置，而使用我们现在这种命令行的配置方式是临时的**

```bash
# replicaof <masterip> <masterport>   配置主机的IP和主机的端口号
```

## 测试

> 主机只能写，从机只能读不能写



```bash
#主机写入操作
127.0.0.1:6379> set k1 vi
OK
127.0.0.1:6379>

#从机
127.0.0.1:6380> keys *			#从机可以获取主机写入到数据
1) "k1"
127.0.0.1:6380> get k1
"vi"
127.0.0.1:6380> set k2
(error) ERR wrong number of arguments for 'set' command		#从机不能写入数据
127.0.0.1:6380>
```

- 如果主机宕机，从机依旧可以连接到主机，但是没有写操作，如果主机重新启动上线，从机依旧可以直接获取主机写入的信息
- 如果命令行配置的主从，从机重启了，那么此从机就不再是从机了又变回了主机的角色，与之前的主机没有任何联系了
- 如果从机重启后再次配置成为原主机的从机，那么立刻就能同步主机写入的数据



## 复制原理

- SLAVE启动成功连接到master后会发送一条sync指令
- MASTER接到指令后，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到SLAVE，并完成一次同步

**全量复制与增量复制**

- 全量复制：SLAVE服务在接收到数据库文件数据后，将其存盘并加载内存中
- 增量复制：MASTER继续将所有收集到的修改命令依次传递给slave，完成同步

只要是重新连接master，一次完全同步（全量同步）将被自动执行



## 主机宕机后手动配置主机



如果因为某种原因MASTER宕机了，那么就需要手动将其中一台SLAVE配置为主机，让其与其他SLAVE完成主从复制。

使用`SLAVEOF no one`命令来让自己称为主机，其他节点手动连接到这台最新的节点（手动）。

如果MASTER修复了，重新启动，那就重新连接到重启的主机。



# 2.哨兵模式（Sentinel）详解



## 哨兵模式概述

> **主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。**这不是一种推荐的方式，更多时候，我们优先考虑**哨兵模式**。
>
> 一言以蔽之：一种在MASTER宕机之后，自动选举新的MASTER的模式

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是**哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。**

![](.\image\哨兵模式.png)

这里的哨兵有两个作用

- 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
- 当哨兵监测到master宕机，会自动将slave切换成master，然后通过**发布订阅模式**通知其他的从服务器，修改配置文件，让它们切换主机

然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

![](.\image\多哨兵监控Redis.png)

**故障切换（failover）**的过程：

假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行faliover过程，仅仅是在哨兵1主观的认为主服务器不可用，这个现象称为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票结果由一个哨兵发起，进行faliover（故障转移）操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从从服务器实现切换主机，这个过程称为**客观下线**。这样对于客户端而言，一切都是透明的。



## 测试

我们目前的状态是一主二从。

**配置哨兵**

```bash
#新建一个配置文件
[root@centos7 h_config]$ vim sentinel.conf

#sentinel.conf文件内容

# 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义
sentinel monitor myredis 127.0.0.1 6379 1		#最后的"1"表示只有一个或一个以上的哨兵认为主服务器不可用的时候，才会进行failover操作

#启动启动哨兵
[root@centos7 bin]$ ./redis-sentinel ./h_config/sentinel.conf
#注意启动顺序。首先是主机的Redis服务进程，然后启动从机的服务进程，最后启动哨兵的服务进程。
```

现在手动挂掉主机，然后观察redis-sentinel进程输出的信息：

```bash
20497:X 03 May 2020 03:34:19.848 # +sdown master myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:19.848 # +odown master myredis 127.0.0.1 6379 #quorum 1/1
20497:X 03 May 2020 03:34:19.848 # +new-epoch 1
20497:X 03 May 2020 03:34:19.848 # +try-failover master myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:19.849 # +vote-for-leader eb7cda664385ca801eb7d180f589668b29841e52 1
20497:X 03 May 2020 03:34:19.849 # +elected-leader master myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:19.849 # +failover-state-select-slave master myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:19.905 # +selected-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:19.905 * +failover-state-send-slaveof-noone slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:19.964 * +failover-state-wait-promotion slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:20.238 # +promoted-slave slave 127.0.0.1:6380 127.0.0.1 6380 @ myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:20.238 # +failover-state-reconf-slaves master myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:20.309 * +slave-reconf-sent slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:21.240 * +slave-reconf-inprog slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:21.240 * +slave-reconf-done slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:21.296 # +failover-end master myredis 127.0.0.1 6379
20497:X 03 May 2020 03:34:21.296 # +switch-master myredis 127.0.0.1 6379 127.0.0.1 6380
20497:X 03 May 2020 03:34:21.296 * +slave slave 127.0.0.1:6381 127.0.0.1 6381 @ myredis 127.0.0.1 6380
20497:X 03 May 2020 03:34:21.296 * +slave slave 127.0.0.1:6379 127.0.0.1 6379 @ myredis 127.0.0.1 6380

#这些信息表明哨兵执行了failover
```

再看原先两个从服务器的信息：

```bash
#6380端口的Redis进程，从SLAVE变为了MASTER
127.0.0.1:6380> info replication
# Replication
role:master			#角色已经变化
connected_slaves:1
slave0:ip=127.0.0.1,port=6381,state=online,offset=36627,lag=1#此进程的从服务器变为了6381的Redis进程
master_replid:5f469abd58c498f7b4a2fdaaf70b768f982e40f0
master_replid2:09c5d9449469e16d23dd6e1b5adce3bc73be6e07
master_repl_offset:36627
second_repl_offset:31213
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:36627
127.0.0.1:6380>

#6381
127.0.0.1:6381> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6380		#此进程的MASTER已经变更为6380的进程
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:44583
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:5f469abd58c498f7b4a2fdaaf70b768f982e40f0
master_replid2:09c5d9449469e16d23dd6e1b5adce3bc73be6e07
master_repl_offset:44583
second_repl_offset:31213
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:44583
127.0.0.1:6381>
```

**如果主机重新上线，只能作为新主机下面的从机**



## Java中使用哨兵模式

```java
/**
 * 测试Redis哨兵模式
 * @author liu
 */
public class TestSentinels {
    @SuppressWarnings("resource")
    @Test
    public void testSentinel() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(10);
        jedisPoolConfig.setMaxIdle(5);
        jedisPoolConfig.setMinIdle(5);
        // 哨兵信息
        Set<String> sentinels = new HashSet<>(Arrays.asList("192.168.11.128:26379",
                "192.168.11.129:26379","192.168.11.130:26379"));
        // 创建连接池
        JedisSentinelPool pool = new JedisSentinelPool("mymaster", sentinels,jedisPoolConfig,"123456");
        // 获取客户端
        Jedis jedis = pool.getResource();
        // 执行两个命令
        jedis.set("mykey", "myvalue");
        String value = jedis.get("mykey");
        System.out.println(value);
    }
}
```



上面是通过Jedis进行使用的，同样也可以使用Spring进行配置RedisTemplate使用。

```xml
        <bean id = "poolConfig" class="redis.clients.jedis.JedisPoolConfig">
            <!-- 最大空闲数 -->
            <property name="maxIdle" value="50"></property>
            <!-- 最大连接数 -->
            <property name="maxTotal" value="100"></property>
            <!-- 最大等待时间 -->
            <property name="maxWaitMillis" value="20000"></property>
        </bean>
        
        <bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
            <constructor-arg name="poolConfig" ref="poolConfig"></constructor-arg>
            <constructor-arg name="sentinelConfig" ref="sentinelConfig"></constructor-arg>
            <property name="password" value="123456"></property>
        </bean>
        
        <!-- JDK序列化器 -->
        <bean id="jdkSerializationRedisSerializer" class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer"></bean>
        
        <!-- String序列化器 -->
        <bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer"></bean>
        
        <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
            <property name="connectionFactory" ref="connectionFactory"></property>
            <property name="keySerializer" ref="stringRedisSerializer"></property>
            <property name="defaultSerializer" ref="stringRedisSerializer"></property>
            <property name="valueSerializer" ref="jdkSerializationRedisSerializer"></property>
        </bean>
        
        <!-- 哨兵配置 -->
        <bean id="sentinelConfig" class="org.springframework.data.redis.connection.RedisSentinelConfiguration">
            <!-- 服务名称 -->
            <property name="master">
                <bean class="org.springframework.data.redis.connection.RedisNode">
                    <property name="name" value="mymaster"></property>
                </bean>
            </property>
            <!-- 哨兵服务IP和端口 -->
            <property name="sentinels">
                <set>
                    <bean class="org.springframework.data.redis.connection.RedisNode">
                        <constructor-arg name="host" value="192.168.11.128"></constructor-arg>
                        <constructor-arg name="port" value="26379"></constructor-arg>
                    </bean>
                    <bean class="org.springframework.data.redis.connection.RedisNode">
                        <constructor-arg name="host" value="192.168.11.129"></constructor-arg>
                        <constructor-arg name="port" value="26379"></constructor-arg>
                    </bean>
                    <bean class="org.springframework.data.redis.connection.RedisNode">
                        <constructor-arg name="host" value="192.168.11.130"></constructor-arg>
                        <constructor-arg name="port" value="26379"></constructor-arg>
                    </bean>
                </set>
            </property>
        </bean>
```

## 哨兵模式的其他配置项



| 配置项                           | 参数类型                     | 作用                                                         |
| -------------------------------- | ---------------------------- | ------------------------------------------------------------ |
| port                             | 整数                         | 启动哨兵进程端口                                             |
| dir                              | 文件夹目录                   | 哨兵进程服务临时文件夹，默认为/tmp，要保证有可写入的权限     |
| sentinel down-after-milliseconds | <服务名称><毫秒数（整数）>   | 指定哨兵在监控Redis服务时，当Redis服务在一个默认毫秒数内都无法回答时，单个哨兵认为的主观下线时间，默认为30000（30秒） |
| sentinel parallel-syncs          | <服务名称><服务器数（整数）> | 指定可以有多少个Redis服务同步新的主机，一般而言，这个数字越小同步时间越长，而越大，则对网络资源要求越高 |
| sentinel failover-timeout        | <服务名称><毫秒数（整数）>   | 指定故障切换允许的毫秒数，超过这个时间，就认为故障切换失败，默认为3分钟 |
| sentinel notification-script     | <服务名称><脚本路径>         | 指定sentinel检测到该监控的redis实例指向的实例异常时，调用的报警脚本。该配置项可选，比较常用 |



sentinel down-after-milliseconds配置项只是一个哨兵在超过规定时间依旧没有得到响应后，会自己认为主机不可用。对于其他哨兵而言，并不是这样认为。哨兵会记录这个消息，当拥有认为主观下线的哨兵达到sentinel monitor所配置的数量时，就会发起一次投票，进行failover，此时哨兵会重写Redis的哨兵配置文件，以适应新场景的需要。



````bash
# Example sentinel.conf
 
# 哨兵sentinel实例运行的端口 默认26379
port 26379
 
# 哨兵sentinel的工作目录
dir /tmp
 
# 哨兵sentinel监控的redis主节点的 ip port 
# master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
# quorum 配置多少个sentinel哨兵统一认为master主节点失联 那么这时客观上认为主节点失联了
# sentinel monitor <master-name> <ip> <redis-port> <quorum>
  sentinel monitor mymaster 127.0.0.1 6379 2
 
# 当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
# 设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
 
 
# 指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
# sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000
 
# 这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，
这个数字越小，完成failover所需的时间就越长，
但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。
可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
# sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1
 
 
 
# 故障转移的超时时间 failover-timeout 可以用在以下这些方面： 
#1. 同一个sentinel对同一个master两次failover之间的间隔时间。
#2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
#3.当想要取消一个正在进行的failover所需要的时间。  
#4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
# 默认三分钟
# sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000
 
# SCRIPTS EXECUTION
 
#配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
#对于脚本的运行结果有以下规则：
#若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
#若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
#如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
#一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
 
#通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，
这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，
一个是事件的类型，
一个是事件的描述。
如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
#通知脚本
# sentinel notification-script <master-name> <script-path>
  sentinel notification-script mymaster /var/redis/notify.sh
 
# 客户端重新配置主节点参数脚本
# 当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
# 以下参数将会在调用脚本时传给脚本:
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
# 目前<state>总是“failover”,
# <role>是“leader”或者“observer”中的一个。 
# 参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
# 这个脚本应该是通用的，能被多次调用，不是针对性的。
# sentinel client-reconfig-script <master-name> <script-path>
 sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
````



## 哨兵模式总结



优点：

- 哨兵集群，基于主从复制模式，所有的主从配置的优点，哨兵集群全都有
- 主从可以切换，故障可以转移，系统的可用性会更好
- 哨兵模式是主从复制模式的升级，手动到自动，系统更加的健壮



缺点：

- Redis不好在线扩容，集群容量到达上限，在线扩容就非常麻烦
- 实现实现哨兵模式的配置十分麻烦，里面有很多选择

# 3.Redis缓存穿透和雪崩



Redis缓存的使用，极大的提升了应用程序的性能和效率，特别是数据查询方面。但同时，它也带来了一些问题。其中，最要害的问题，就是数据的一致性问题，从严格意义上讲，这个问题无解。如果对数据的一致性要求很高，那么就不能使用缓存。

另外的一些典型问题就是，缓存穿透、缓存雪崩和缓存击穿。

## 缓存穿透

**概念：**

key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会到数据源，从而可能压垮数据源。比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

### 解决方案

**布隆过滤器**

布隆过滤器是一种数据结构，对所有可能查询的参数以hash形式存储，在控制层先进行校验，不符合则丢弃，从而避免了对底层存储系统的查询压力

![](.\image\布隆过滤器方式缓存穿透.png)

**缓存空对象**

当数据持久层不命中后，及时返回一个空对象缓存起来，同时设置一个过期时间，之后再访问这个数据就会从缓存中获取，保护了后端数据

![](.\image\缓存空对象解决缓存穿透.png)

但是这种方法存在有两个问题：

- 如果控制能够被缓存起来，这就意味着缓存需要更多的空间存储更多的键，因为者当中可能有很多空值的键
- 即使对空值设置了过期时间，还是会存在缓存层和数据持久层的数据会有一段时间窗口不一致，这就对需要保持一致性的业务会有一定的影响

采用这种粗暴方式的Java代码

````java
//伪代码
public object GetProductListNew() {
    int cacheTime = 30;
    String cacheKey = "product_list";

    String cacheValue = CacheHelper.Get(cacheKey);
    if (cacheValue != null) {
        return cacheValue;
    }

    cacheValue = CacheHelper.Get(cacheKey);
    if (cacheValue != null) {
        return cacheValue;
    } else {
        //数据库查询不到，为空
        cacheValue = GetProductListFromDB();
        if (cacheValue == null) {
            //如果发现为空，设置个默认值，也缓存起来
            cacheValue = string.Empty;
        }
        CacheHelper.Add(cacheKey, cacheValue, cacheTime);
        return cacheValue;
    }
}
````



## 缓存击穿

**概念：**

key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题。

### 解决方案

- 一个是设置热点key永不过期，从缓存层面来看，没有设置过期时间，所以不会出现热点key过期后产生的问题。
- 二是加互斥锁
	- 分布式锁：使用分布式锁，保证对于每个key同时只有一个线程去查询后端服务，其他线程没有获得分布式锁的权限，因此只需要等待即可。这种方式将高并发的压力转移到了分布式锁，因此对于分布式锁的压力比较大

业界比较常用的做法，是使用mutex。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法。

SETNX，是「SET if Not eXists」的缩写，也就是只有不存在的时候才设置，可以利用它来实现锁的效果。

```java
public String get(key) {
    String value = redis.get(key);
    if (value == null) { //代表缓存值过期
        //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
        if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
            value = db.get(key);
            redis.set(key, value, expire_secs);
            redis.del(key_mutex);
        } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
            sleep(50);
            get(key);  //重试
        }
    } else {
        return value;      
    }
}

//memcache代码:
if (memcache.get(key) == null) {  
    // 3 min timeout to avoid mutex holder crash  
    if (memcache.add(key_mutex, 3 * 60 * 1000) == true) {  
        value = db.get(key);  
        memcache.set(key, value);  
        memcache.delete(key_mutex);  
    } else {  
        sleep(50);  
        retry();  
    }  
}
```

## 缓存雪崩

### 概述

当缓存服务器重启或者大量缓存集中在某一个时间段失效，这样在失效的时候，也会给后端系统(比如DB)带来很大压力。

与缓存击穿的区别在于这里针对很多key缓存，前者则是某一个key。

缓存正常从Redis中获取，示意图如下：

![](.\image\缓存雪崩前.png)

缓存失效瞬间示意图如下:

![](.\image\缓存雪崩.png)

### 解决方案

缓存失效时的雪崩效应对底层系统的冲击非常可怕！

- 大多数系统设计者考虑用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。
- 还有一个简单方案就是将缓存失效时间分散开，比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

**加锁排队：**

```java
//伪代码
public object GetProductListNew() {
    int cacheTime = 30;
    String cacheKey = "product_list";
    String lockKey = cacheKey;

    String cacheValue = CacheHelper.get(cacheKey);
    if (cacheValue != null) {
        return cacheValue;
    } else {
        synchronized(lockKey) {
            cacheValue = CacheHelper.get(cacheKey);
            if (cacheValue != null) {
                return cacheValue;
            } else {
                //这里一般是sql查询数据
                cacheValue = GetProductListFromDB(); 
                CacheHelper.Add(cacheKey, cacheValue, cacheTime);
            }
        }
        return cacheValue;
    }
}
```

加锁排队只是为了减轻数据库的压力，并没有提高系统吞吐量。假设在高并发下，缓存重建期间key是锁着的，这是过来1000个请求999个都在阻塞的。同样会导致用户等待超时，这是个治标不治本的方法！

**注意：**加锁排队的解决方式分布式环境的并发问题，有可能还要解决分布式锁的问题；线程还会被阻塞，用户体验很差！因此，在真正的高并发场景下很少使用！



**设置随机值**

```java
//伪代码
public object GetProductListNew() {
    int cacheTime = 30;
    String cacheKey = "product_list";
    //缓存标记
    String cacheSign = cacheKey + "_sign";

    String sign = CacheHelper.Get(cacheSign);
    //获取缓存值
    String cacheValue = CacheHelper.Get(cacheKey);
    if (sign != null) {
        return cacheValue; //未过期，直接返回
    } else {
        CacheHelper.Add(cacheSign, "1", cacheTime);
        ThreadPool.QueueUserWorkItem((arg) -> {
            //这里一般是 sql查询数据
            cacheValue = GetProductListFromDB(); 
            //日期设缓存时间的2倍，用于脏读
            CacheHelper.Add(cacheKey, cacheValue, cacheTime * 2);                 
        });
        return cacheValue;
    }
} 

```

**解释说明：**

- 缓存标记：记录缓存数据是否过期，如果过期会触发通知另外的线程在后台去更新实际key的缓存；
- 缓存数据：它的过期时间比缓存标记的时间延长1倍，例：标记缓存时间30分钟，数据缓存设置为60分钟。这样，当缓存标记key过期后，实际缓存还能把旧数据返回给调用端，直到另外的线程在后台更新完成后，才会返回新缓存。



关于缓存崩溃的解决方法，这里提出了三种方案：使用锁或队列、设置过期标志更新缓存、为key设置不同的缓存失效时间，还有一种被称为“二级缓存”的解决方法

## 小结



针对业务系统，永远都是具体情况具体分析，没有最好，只有最合适。

最后也提一下三个词LRU、RDB、AOF，通常我们采用LRU策略处理溢出，Redis的RDB和AOF持久化策略来保证一定情况下的数据安全。