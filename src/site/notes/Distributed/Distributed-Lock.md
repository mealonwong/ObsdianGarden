---
{"dg-publish":true,"permalink":"/distributed/distributed-lock/"}
---


## 什么是分布式锁

- 在分布式模型下，数据只有一份或不允许多个线程同时修改数据，此时需要利用锁的技术控制某一时刻仅有一个线程可修改该数据；
- 与在单机模式下的锁不仅需要保证进程可见，还需要保证进程与锁之间的网络问题
- 分布式锁是将标记存在内存，只是该内存不是某个进程分配的内存而是公共内存例如Redis等，。至于利用数据库、文件、zookeeper等作锁的实现是一样的，只要保证标记互斥就可以。

## 需要怎样的分布式锁

- 保证分布式环境中，同一个方法在同一时间仅可以被一台机器上的一个线程获取到
- 最好是可重入锁，可以比避免死锁
- 阻塞锁（根据业务判断是否需要）
- 公平锁（根据业务判断是否需要）
- 有高可用的获取锁和释放锁的功能
- 获取锁和释放锁的性能要好

## 分布式锁的实现

### 基于数据库的分布式锁

#### 悲观锁的实现
可以使用select ... for update来实现分布式锁

表结构
```SQL
CREATE TABLE `t_resource_lock` (
  `key_resource` varchar(45) COLLATE utf8_bin NOT NULL DEFAULT '资源主键',
  `status` char(1) COLLATE utf8_bin NOT NULL DEFAULT '' COMMENT 'S,F,P',
  `lock_flag` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '1是已经锁 0是未锁',
  `begin_time` datetime DEFAULT NULL COMMENT '开始时间',
  `end_time` datetime DEFAULT NULL COMMENT '结束时间',
  `client_ip` varchar(45) COLLATE utf8_bin NOT NULL DEFAULT '抢到锁的IP',
  `time` int(10) unsigned NOT NULL DEFAULT '60' COMMENT '方法生命周期内只允许一个结点获取一次锁，单位：分钟',
  PRIMARY KEY (`key_resource`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin
```

加Lock的伪代码实现
```Java
@Transcational //一定要加事务
public boolean lock(String keyResource，int time){
   resourceLock = 'select * from t_resource_lock where key_resource ='#{keySource}' for update';
   
   try{
    if(resourceLock==null){
      //插入锁的数据
      resourceLock = new ResourceLock();
      resourceLock.setTime(time);
      resourceLock.setLockFlag(1);  //上锁
      resourceLock.setStatus(P); //处理中
      resourceLock.setBeginTime(new Date());
      int count = "insert into resourceLock"; 
      if(count==1){
         //获取锁成功
         return true;
      }
      return false;
   }
   }catch(Exception x){
      return false;
   }
   
   //没上锁并且锁已经超时，即可以获取锁成功
   if(resourceLock.getLockFlag=='0'&&'S'.equals(resourceLock.getstatus)
    && new Date()>=resourceLock.addDateTime(resourceLock.getBeginTime(,time)){
      resourceLock.setLockFlag(1);  //上锁
      resourceLock.setStatus(P); //处理中
      resourceLock.setBeginTime(new Date());
      //update resourceLock;
      return true;
   }else if(new Date()>=resourceLock.addDateTime(resourceLock.getBeginTime(,time)){
     //超时未正常执行结束,获取锁失败
     return false;
   }else{
     return false;
   } 
}


```

unlock

```Java
public void unlock(String v，status){
      resourceLock.setLockFlag(0);  //解锁
      resourceLock.setStatus(status); S:表示成功，F表示失败
      //update resourceLock;
      return ;
}

```

整体流程
```Java
try{
if(lock(keyResource,time)){ //加锁
   status = process();//你的业务逻辑处理。
 }
} finally{
    unlock(keyResource,status); //释放锁
}

```
其实这个悲观锁实现的分布式锁，整体的流程还是比较清晰的。就是先select ... for update 锁住主键key_resource那个记录，如果为空，则可以插入一条记录，如果已有记录判断下状态和时间，是否已经超时。这里需要注意一下哈，必须要加事务哈。

#### 乐观锁的实现

除了悲观锁，还可以用乐观锁实现分布式锁。乐观锁，顾名思义，就是很乐观，每次更新操作，都觉得不会存在并发冲突，只有更新失败后，才重试。它是基于CAS思想实现的。

以扣减金额为例
搞个version字段，每次更新修改，都会自增加一，然后去更新余额时，把查出来的那个版本号，带上条件去更新，如果是上次那个版本号，就更新，如果不是，表示别人并发修改过了，就继续重试。

```SQL
select version,balance from account where user_id ='666';
```

逻辑判断
```Java
if(balance<扣减金额){
   return；
}

left_balance = balance - 扣减金额;

```

```SQL
update account set balance = #{left_balance} ,version = version+1 where version 
= #{oldVersion} and balance>= #{left_balance} and user_id ='666';

```
这种方式适合并发不高的场景，一般需要设置一下重试的次数。
### 基于Redis的分布式锁

Redis实现分布式锁一般有以下几种方式
- setnx + expire。
- setnx + value值是过期时间。
- set的扩展命令(set ex px nx)。
- set ex px nx + 校验唯一随机值,再删除。
- Redisson。
- Redisson + RedLock。

#### setnx+expire
```Java
if（jedis.setnx(key,lock_value) == 1）{ //setnx加锁
    expire（key，100）; //设置过期时间
    try {
        do something  //业务处理
    }catch(){
    }
  finally {
       jedis.del(key); //释放锁
    }
}

```
这种方式可以加锁成功，但是存在问题。因为加锁和设置超时时间的操作是分开的，如果加完锁后，在设置超时时间时进程crash掉或者重启维护了，那么这个锁就永远不会过期，其他进程就永远拿不到这个锁。

#### setnx+value值是过期时间
```Java
long expires = System.currentTimeMillis() + expireTime; //系统时间+设置的过期时间
String expiresStr = String.valueOf(expires);

// 如果当前锁不存在，返回加锁成功
if (jedis.setnx(key, expiresStr) == 1) {
        return true;
} 
// 如果锁已经存在，获取锁的过期时间
String currentValueStr = jedis.get(key);

// 如果获取到的过期时间，小于系统当前时间，表示已经过期
if (currentValueStr != null && Long.parseLong(currentValueStr) < System.currentTimeMillis()) {

     // 锁已过期，获取上一个锁的过期时间，并设置现在锁的过期时间（不了解redis的getSet命令的小伙伴，可以去官网看下哈）
    String oldValueStr = jedis.getSet(key, expiresStr);
    
    if (oldValueStr != null && oldValueStr.equals(currentValueStr)) {
         // 考虑多线程并发的情况，只有一个线程的设置值和当前值相同，它才可以加锁
         return true;
    }
}
        
//其他情况，均返回加锁失败
return false;
}

```

这么实现分布式锁仍旧存在以下缺点：
- 过期时间是客户端自己生成的，分布式环境下，每个客户端的时间必须同步。
- 没有保存持有者的唯一标识，可能被别的客户端释放/解锁。
- 锁过期的时候，并发多个客户端同时请求过来，都执行了jedis.getSet()，最终只能有一个客户端加锁成功，但是该客户端锁的过期时间，可能被别的客户端覆盖。

#### set的扩展命令（set ex px nx）
命令的意思
``` redis
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```
- EX second ：设置键的过期时间为second秒。
- PX millisecond ：设置键的过期时间为millisecond毫秒。
- NX ：只在键不存在时，才对键进行设置操作。
- XX ：只在键已经存在时，才对键进行设置操作。

```Java
if（jedis.set(key, lock_value, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch(){
  }
  finally {
       jedis.del(key); //释放锁
    }
}

```

该方式存在以下问题
- 锁过期释放了，业务还没执行完。
- 锁被别的线程误删。
> 锁为什么会被误删？
> 假设线程A和B，都想用key加锁，最后A抢到锁加锁成功，但是由于执行业务逻辑的耗时很长，超过了设置的超时时间100s。这时候，Redis就自动释放了key锁。这时候线程B就可以加锁成功了，接下啦，它也执行业务逻辑处理。假设碰巧这时候，A执行完自己的业务逻辑，它就去释放锁，但是它就把B的锁给释放了。

#### set ex px nx + 校验唯一随机值,再删除
为了解决锁被别的线程误删问题。可以在set ex px nx的基础上，加上个校验的唯一随机值，如下：
```Java
（jedis.set(key, uni_request_id, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch(){
  }
  finally {
       //判断是不是当前线程加的锁,是才释放
       if (uni_request_id.equals(jedis.get(key))) {
          jedis.del(key); //释放锁
        }
    }
}

```
在这里，判断当前线程加的锁和释放锁不是一个原子操作。如果调用jedis.del()释放锁的时候，可能这把锁已经不属于当前客户端，会解除他人加的锁。

一般可以用lua脚本来包一下。lua脚本如下：
```lua
if redis.call('get',KEYS[1]) == ARGV[1] then 
   return redis.call('del',KEYS[1]) 
else
   return 0
end;
```

这种方式比较不错了，一般情况下，已经可以使用这种实现方式。但是还是存在：锁过期释放了，业务还没执行完的问题。

#### Redisson
对于可能存在锁过期释放，业务没执行完的问题。我们可以稍微把锁过期时间设置长一些，大于正常业务处理时间就好啦。如果你觉得不是很稳，还可以给获得锁的线程，开启一个定时守护线程，每隔一段时间检查锁是否还存在，存在则对锁的过期时间延长，防止锁过期提前释放。

当前开源框架Redisson解决了这个问题。可以看下Redisson底层原理图:
![](https://pic.imgdb.cn/item/6538c6c9c458853aefd49ceb.png)

只要线程一加锁成功，就会启动一个watch dog看门狗，它是一个后台线程，会每隔10秒检查一下，如果线程1还持有锁，那么就会不断的延长锁key的生存时间。因此，Redisson就是使用watch dog解决了锁过期释放，业务没执行完问题。

#### Redisson + RedLock

前面六种方案都只是基于Redis单机版的分布式锁讨论，还不是很完美。因为Redis一般都是集群部署的：
![](https://pic.imgdb.cn/item/6538c719c458853aefd58033.png)

如果线程一在Redis的master节点上拿到了锁，但是加锁的key还没同步到slave节点。恰好这时，master节点发生故障，一个slave节点就会升级为master节点。线程二就可以顺理成章获取同个key的锁啦，但线程一也已经拿到锁了，锁的安全性就没了。

为了解决这个问题，Redis作者antirez提出一种高级的分布式锁算法：Redlock。它的核心思想是这样的：

部署多个Redis master，以保证它们不会同时宕掉。并且这些master节点是完全相互独立的，相互之间不存在数据同步。同时，需要确保在这多个master实例上，是与在Redis单实例，使用相同方法来获取和释放锁。

我们假设当前有5个Redis master节点，在5台服务器上面运行这些Redis实例。

![](https://pic.imgdb.cn/item/6538c745c458853aefd5f794.png)

RedLock的实现步骤:

- 获取当前时间，以毫秒为单位。
- 按顺序向5个master节点请求加锁。客户端设置网络连接和响应超时时间，并且超时时间要小于锁的失效时间。(假设锁自动失效时间为10秒，则超时时间一般在5-50毫秒之间,我们就假设超时时间是50ms吧)。如果超时，跳过该master节点，尽快去尝试下一个master节点。
- 客户端使用当前时间减去开始获取锁时间(即步骤1记录的时间)，得到获取锁使用的时间。当且仅当超过一半(N/2+1，这里是5/2+1=3个节点)的Redis master节点都获得锁，并且使用的时间小于锁失效时间时，锁才算获取成功。(如上图，10s> 30ms+40ms+50ms+4m0s+50ms)。
- 如果取到了锁，key的真正有效时间就变啦，需要减去获取锁所使用的时间。
- 如果获取锁失败(没有在至少N/2+1个master实例取到锁，有或者获取锁时间已经超过了有效时间)，客户端要在所有的master节点上解锁(即便有些master节点根本就没有加锁成功，也需要解锁，以防止有些漏网之鱼)。

简化下步骤就是：

- 按顺序向5个master节点请求加锁。
- 根据设置的超时时间来判断，是不是要跳过该master节点。
- 如果大于等于3个节点加锁成功，并且使用的时间小于锁的有效期，即可认定加锁成功啦。
- 如果获取锁失败，解锁!

Redisson实现了redLock版本的锁，有兴趣的小伙伴，可以去了解一下哈~
### 基于ZK的分布式锁

Zookeeper的节点Znode有四种类型：

- 持久节点：默认的节点类型。创建节点的客户端与zookeeper断开连接后，该节点依旧存在。
- 持久节点顺序节点：所谓顺序节点，就是在创建节点时，Zookeeper根据创建的时间顺序给该节点名称进行编号，持久节点顺序节点就是有顺序的持久节点。
- 临时节点：和持久节点相反，当创建节点的客户端与zookeeper断开连接后，临时节点会被删除。
- 临时顺序节点：有顺序的临时节点。

Zookeeper分布式锁实现应用了临时顺序节点。这里不贴代码啦，来讲下zk分布式锁的实现原理吧。

### ZK获取锁的过程
当第一个客户端请求过来时，Zookeeper客户端会创建一个持久节点locks。如果它(Client1)想获得锁，需要在locks节点下创建一个顺序节点lock1
![](https://pic.imgdb.cn/item/6538c8ccc458853aefda54ab.png)
接着，客户端Client1会查找locks下面的所有临时顺序子节点，判断自己的节点lock1是不是排序最小的那一个，如果是，则成功获得锁。
![](https://pic.imgdb.cn/item/6538c8fbc458853aefdb002a.png)
 这时候如果又来一个客户端client2前来尝试获得锁，它会在locks下再创建一个临时节点lock2。
![](https://pic.imgdb.cn/item/6538c90cc458853aefdb3b66.png)
客户端client2一样也会查找locks下面的所有临时顺序子节点，判断自己的节点lock2是不是最小的，此时，发现lock1才是最小的，于是获取锁失败。获取锁失败，它是不会甘心的，client2向它排序靠前的节点lock1注册Watcher事件，用来监听lock1是否存在，也就是说client2抢锁失败进入等待状态。
![](https://pic.imgdb.cn/item/6538c91cc458853aefdb6ef2.png)
此时，如果再来一个客户端Client3来尝试获取锁，它会在locks下再创建一个临时节点lock3。
![](https://pic.imgdb.cn/item/6538c92ec458853aefdbabe2.png)
同样的，client3一样也会查找locks下面的所有临时顺序子节点，判断自己的节点lock3是不是最小的，发现自己不是最小的，就获取锁失败。它也是不会甘心的，它会向在它前面的节点lock2注册Watcher事件，以监听lock2节点是否存在。
![](https://pic.imgdb.cn/item/6538c93dc458853aefdbd638.png)
### 释放锁

 我们再来看看释放锁的流程，Zookeeper的客户端业务完成或者发生故障，都会删除临时节点，释放锁。如果是任务完成，Client1会显式调用删除lock1的指令。
![](https://pic.imgdb.cn/item/6538c994c458853aefdce75c.png) 如果是客户端故障了，根据临时节点得特性，lock1是会自动删除的。
![](https://pic.imgdb.cn/item/6538c9a3c458853aefdd10ff.png)
lock1节点被删除后，Client2可开心了，因为它一直监听着lock1。lock1节点删除，Client2立刻收到通知，也会查找locks下面的所有临时顺序子节点，发下lock2是最小，就获得锁。
![](https://pic.imgdb.cn/item/6538c9b3c458853aefdd3d2c.png)
1. 同理，Client2获得锁之后，Client3也对它虎视眈眈
- Zookeeper设计定位就是分布式协调，简单易用。如果获取不到锁，只需添加一个监听器即可，很适合做分布式锁。
- Zookeeper作为分布式锁也缺点：如果有很多的客户端频繁的申请加锁、释放锁，对于Zookeeper集群的压力会比较大。


## 分布式锁对比

#### 5.1 数据库分布式锁实现

优点：

- 简单，使用方便，不需要引入Redis、zookeeper等中间件。

缺点：

- 不适合高并发的场景。
- db操作性能较差，有锁表的风险。

#### 5.2 Redis分布式锁实现

优点：

- 性能好，适合高并发场景。
- 较轻量级。
- 有较好的框架支持，如Redisson。

缺点：

- 过期时间不好控制。
- 需要考虑锁被别的线程误删场景。

#### 5.3 Zookeeper分布式锁实现

缺点：

- 性能不如redis实现的分布式锁。
- 比较重的分布式锁。

优点：

- 有较好的性能和可靠性。
- 有封装较好的框架，如Curator。

#### 5.4 对比汇总

- 从性能角度(从高到低)Redis > Zookeeper >= 数据库;
- 从理解的难易程度角度(从低到高)数据库 > Redis > Zookeeper;
- 从实现的复杂性角度(从低到高)Zookeeper > Redis > 数据库;
- 从可靠性角度(从高到低)Zookeeper > Redis > 数据库。


原文链接
[聊聊分布式锁的多种实现！](https://www.51cto.com/article/705985.html)