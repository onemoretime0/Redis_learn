# 1.Redis事务

## 简介

Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

**Redis事务没有隔离级别的概念**

　　批量操作在发送 EXEC 命令前被放入队列缓存，并不会被实际执行，也就不存在事务内的查询要看到事务里的更新，事务外查询不能看到。

​	所有的命令在事务中，并没有直接被执行，只有发起执行命令的时候才会被执行

**Redis事务不保证原子性：**

Redis中，单条命令是原子性执行的，但事务不保证原子性，且没有回滚。==事务中任意命令执行失败，其余的命令仍会被执行==

**一个事务从开始到执行会经历以下三个阶段：**

- 开始事务。
- 命令入队。
- 执行事务。

**Redis事务的本质：**一组命令的集合，一个事务中的所有命令都会被序列化，zz爱十五执行过程中，会按照顺序执行

**Redis事务相关命令：**

- ``EXEC``：取消事务
- `WATCH`：监视命令
- `DISCARD`：取消事务
- `UNWATCH`：取消监视
- `MULTI`：标记一个事务块的开始

## 事务操作

### 正常的事务执行

```bash
127.0.0.1:6379> multi					#开启一个事务块
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec					#执行事务，三条命令将依次被执行
1) OK
2) OK
3) "v2"
127.0.0.1:6379>
```

### 取消事务

```bash
#Discard 命令用于取消事务，放弃执行事务块内的所有命令。

127.0.0.1:6379> multi			#开启事务
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> discard			#取消事务
OK
127.0.0.1:6379> 
```

### 事务的两种异常

- 编译时异常（代码有问题，命令有错），事务中的所有命令都不会被执行
- 运行时异常，如果事务中存在语法性，那么执行命令的时候，其他命令时可以正常执行，错误命令抛出异常

**编译时异常：**

```bash
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> getset k1					#错误的命令，直接报错
(error) ERR wrong number of arguments for 'getset' command
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec						#执行事务也会报错
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k2						#事务中的命令都没有被执行
(nil)
127.0.0.1:6379> get k1
(nil)
127.0.0.1:6379>
```

**运行时异常：**

```bash
127.0.0.1:6379> set k1 "admin"					#k1 是一个字符串
OK
127.0.0.1:6379> multi							#开启事务
OK
127.0.0.1:6379> incr k1		#对k1自增加一，在语法上是没有错误的但是k1是字符串，这样做就会产生运行时错误
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec							#执行事务就会报错
1) (error) ERR value is not an integer or out of range
2) OK											#但是事务中的其他命令依然会执行
3) "v2"											
127.0.0.1:6379>
```

## Redis实现乐观锁



### 乐观锁和悲观锁

**悲观锁：**

> 总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程）。
>
> 传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现。

**乐观锁：**

> 总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制和CAS算法实现。
>
> 乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于write_condition机制，其实都是提供的乐观锁。在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

**两种锁的使用场景：**

- ==乐观锁适用于写操作较少的情况下（多读场景）==，即即冲突真的很少发生的时候，这样可以省去了锁的开销，加大了系统的整个吞吐量。
- ==写操作比较多的情况下悲观锁就比较合适==

**在redis中实现乐观锁依赖于两条命令：`WITCH`和`UNWATCH`**

### 实现乐观锁（面试常问）

**正常的执行：**

```bash
127.0.0.1:6379> set money 100					#原始的账户金额
OK
127.0.0.1:6379> set out 0						#用掉的金额
OK
127.0.0.1:6379> watch money						#监视money账户
OK
127.0.0.1:6379> multi							#开启事务
OK
127.0.0.1:6379> decrby money 20					#账户金额减20元
QUEUED
127.0.0.1:6379> incrby out 20					#用掉的金额加20元
QUEUED
127.0.0.1:6379> exec							#执行事务--->执行成功
1) (integer) 80
2) (integer) 20
127.0.0.1:6379>
```

**多线程修改值：**

```bash
#线程一：监视money ，并开启事务
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> decrby money 10
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> 					#事务并没有开始执行

#########################################################################################

#在线程一还没有开始执行事务的时候，此时新的线程修改了money的值
127.0.0.1:6379> set money 1000
OK
127.0.0.1:6379>

######################################################################################

#在线程二修改money的值后，线程一执行事务
127.0.0.1:6379> exec
(nil)									#事务执行失败了
127.0.0.1:6379> get money				#余额账户中的数值已经被线程二修改二零
"1000"
127.0.0.1:6379>
```

**WAICH可以作为Redis乐观锁操作**



**使用UNWATCH解决这种问题**:

- 如果发现事务执行失败，就是用unwatch解锁
- 然后再次watch监视获取最新的值
- 重新执行事务

```bash
127.0.0.1:6379> unwatch					#取消监控
OK
127.0.0.1:6379> watch money				#再次监视，获得了money最新值
OK
127.0.0.1:6379> multi					#再次开启事务
OK
127.0.0.1:6379> decrby money 10			
QUEUED
127.0.0.1:6379> incrby out 10
QUEUED
127.0.0.1:6379> exec					#执行成功
1) (integer) 990
2) (integer) 30
127.0.0.1:6379> get money				#余额还剩了990
"990"
127.0.0.1:6379> get out					#总共花了三十块
"30"
127.0.0.1:6379>
```

# 2.Jedis

> Jedis是Redis官方推荐的的Redis连接开发工具，时使用Java操作redis的中间件

依赖：

```xml
<!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```

## 使用jedis

要使用jedis 首先要创建jedis的对象，创建Jedis对象时可以指定ip和端口号。

Jedis包含多种构造器，可以根据需求创建Jedis对象

```java
Jedis jedis = new Jedis("127.0.0.1",6379);
```

Jedis中所有的方法就是我们之前学习的那些命令

````java
    //1.获取链接
    Jedis jedis=new Jedis("localhost",6379);
    //2.操作
    jedis.set("username","水濑祈");
    //3.关闭连接
    jedis.close();
````

**操作数据：**

```java

/**
 * Jedis操作string
 */
@Test
public void testString(){
    Jedis jedis=new Jedis();//如果使用空参的构造函数，默认主机是localhost，端口号是6379

    jedis.set("name","zhangsan");
    System.out.println( jedis.get("name"));

    //使用setex()方法存储可以指定过期时间的 key value
    jedis.setex("setexcode",20,"hehe");//将setexcode:hehe键值对存入redis,并且20秒后自动删除该键值对
    
    jedis.close();
}

/**
 * Jedis操作hash
 */
@Test
public void testHash(){
    Jedis jedis=new Jedis();//如果使用空参的构造函数，默认主机是localhost，端口号是6379

    jedis.hset("user","name","zhangsan");
    jedis.hset("user","age","23");
    jedis.hset("user","gender","male");

    //获取
    String hget = jedis.hget("user", "name");

    Map<String, String> user = jedis.hgetAll("user");

    System.out.println(hget);
    //遍历集合
    Set<String> keySet = user.keySet();
    for (String key : keySet) {
        System.out.println(key+": "+user.get(key));
    }
    jedis.close();
}

/**
 * jedis操作list
 */
@Test
public void testList(){
    Jedis jedis=new Jedis();//如果使用空参的构造函数，默认主机是localhost，端口号是6379

    jedis.lpush("username","zhangsan");
    jedis.lpush("username","01");
    jedis.lpush("username","02");

    jedis.rpush("username","lisi");

    List<String> username = jedis.lrange("username", 1, 3);
    System.out.println(username);

    String leftusername = jedis.lpop("username");
    String rightusernmae = jedis.rpop("username");

    System.out.println(leftusername);
    System.out.println(rightusernmae);
    
    jedis.close();
    
}

```

## Jedis操作事务

**正常执行没有异常抛出:**

```java
public class TestTx {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("localhost", 6379);
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello", "world");
        jsonObject.put("name", "zhagnsan");
        String s = jsonObject.toJSONString();

        Transaction multi = jedis.multi();//开启事务
        
        try {
            //事务操作
            multi.set("user1", s);
            multi.set("user2", s);

            multi.exec();

        } catch (Exception e) {
            multi.discard();       //放弃事务
            e.printStackTrace();
        } finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();  //关闭连接
        }
    }
}

/*
{"name":"zhagnsan","hello":"world"}
{"name":"zhagnsan","hello":"world"}
*/

```

**运行时异常：**

```java
public class TestTx {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("localhost", 6379);
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("hello", "world");
        jsonObject.put("name", "zhagnsan");
        String s = jsonObject.toJSONString();

        Transaction multi = jedis.multi();//开启事务

        try {
            //事务操作
            multi.set("user1", s);
            int i = 1 / 0;         //制造运行时异常
            multi.set("user2", s);
            multi.exec();

        } catch (Exception e) {
            multi.discard();       //放弃事务
            e.printStackTrace();
        } finally {
            System.out.println(jedis.get("user1"));
            System.out.println(jedis.get("user2"));
            jedis.close();  //关闭连接
        }
    }
}

/*
java.lang.ArithmeticException: / by zero
	at com.jedis.TestTx.main(TestTx.java:23)
null
null

报异常之后，就直接进入到catch块中放弃了事务
*/
```

