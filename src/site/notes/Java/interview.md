---
{"dg-publish":true,"permalink":"/java/interview/"}
---




## mybatis
### mybits缓存原理，用了哪个二级缓存，为什么用？

1. **一级缓存原理**
	- **作用域与生命周期**：一级缓存是SqlSession实例级别的本地缓存，生命周期与SqlSession绑定。当SqlSession关闭、提交或回滚时，缓存自动清空。
	- - **存储与命中机制**：缓存存储在内存HashMap中，Key由SQL语句、参数、MappedStatement ID和环境信息组成。同一SqlSession内，相同查询（SQL+参数）直接返回缓存结果，避免数据库访问。
	- **失效规则**：执行增删改操作（INSERT/UPDATE/DELETE）或手动调用`clearCache()`时，缓存立即清空，确保数据一致性。
	- **配置方式**：默认开启，无法关闭；可通过`localCacheScope`参数设置作用范围为SESSION（默认）或STATEMENT（语句级别）。
2. **二级缓存原理**
	- - **作用域与共享**：二级缓存是Mapper级别（namespace级别）的全局缓存，多个SqlSession可共享同一Mapper的缓存数据，作用域更大。
	- **配置与启用**：需手动配置：在MyBatis全局配置中开启`cacheEnabled=true`（默认），并在Mapper XML文件添加`<cache/>`标签或使用`@CacheNamespace`注解。
	- **存储与命中**：缓存Key由查询ID、参数等生成，查询时优先检查二级缓存。命中后直接返回结果，不访问数据库；未命中则查询数据库并存入缓存。
	- **失效与策略**：增删改操作会触发对应namespace缓存失效；支持自定义回收策略（如LRU、FIFO）和序列化对象存储。
3. **两级缓存协作流程**
	- 查询时，MyBatis优先检查二级缓存（跨会话共享），未命中则检查一级缓存（会话内）。一级缓存未命中才访问数据库，结果先存入一级缓存，再根据配置存入二级缓存。缓存存储基于装饰器模式（如PerpetualCache为基础实现，LRU/FIFO为装饰器）。
4. **作用** ：跨会话共享缓存数据，降低数据库查询压力，提升系统性能。
### MyBatis 动态标签

- `<if>`：条件判断，满足条件则拼接对应 SQL 片段。
- `<where>`：自动处理拼接条件时的`AND/OR`前缀，避免语法错误。
- `<choose><when><otherwise>`：类似 Java 的`switch-case`，多条件分支选择。
- `<foreach>`：遍历集合，常用于批量插入、`IN`条件查询等场景。
- `<set>`：用于更新语句，自动处理字段后的逗号，避免语法错误。

## Redis相关

[[redis/RedisInterview\|RedisInterview]]

### Redis高可用

[[redis/RedisColony\|RedisColony]]
#### 主从复制模式

主从复制是最基础的高可用方案，通过一个主节点（Master）处理写操作，多个从节点（Slave）异步复制数据并提供读服务12。这种模式实现了数据冗余和读写分离，但存在明显的单点故障风险，主节点故障后需要手动干预进行故障转移，无法实现自动恢复3。

**工作原理：**

- 全量同步：从节点首次连接时，主节点生成RDB快照并传输给从节点
- 增量同步：基于复制偏移量和复制积压缓冲区，实现断点续传
- 异步复制：主节点写操作后立即返回，不等待从节点同步

#### 哨兵模式（Sentinel）

哨兵模式在主从复制基础上增加了监控和自动故障转移能力。哨兵节点周期性地向主从节点发送PING请求，监测节点健康状态，当多数哨兵节点达成共识认为主节点不可用时，会自动选举新的主节点并通知客户端。

**核心功能：**

- 故障检测：通过主观下线（SDOWN）和客观下线（ODOWN）机制
- 自动故障转移：选举哨兵领导者，选择最优从节点提升为主节点
- 服务发现：客户端通过SENTINEL命令获取最新主节点地址

**部署建议：**  
至少部署3个哨兵节点，采用奇数个节点避免脑裂，哨兵节点应部署在不同物理机或可用区。

#### 集群模式（Cluster）

Redis Cluster是原生的分布式解决方案，将数据自动分片存储到多个节点。整个Key空间划分为16384个哈希槽（hash slots），通过CRC16(key) % 16384算法确定key的槽位，每个主节点负责一部分槽位。

**高可用机制：**

- 主从复制：每个主节点有多个从节点备份数据
- 自动故障转移：主节点故障时，从节点自动提升为主节点
- Gossip协议：节点间通过Gossip交换状态信息，维护集群拓扑
### Redis 大 Key 查找与处理

- **查找方法**：
    - 用`redis-cli --bigkeys`命令扫描大 Key，按数据类型统计最大 Key。
    - 通过`DEBUG OBJECT key`查看 Key 的内存占用情况。
- **处理方式**：
    - 拆分大 Key：如将大 Hash 拆分为多个小 Hash，大 List 按范围拆分。
    - 异步删除：使用`UNLINK`命令替代`DEL`，避免阻塞主线程。
    - 调整过期策略：合理设置过期时间，避免大 Key 长期占用内存。

### Redis使用场景

- 缓存：热点数据缓存（如商品信息），减轻数据库压力。
- 分布式锁：基于`SET NX EX`命令实现，解决并发问题。
- 计数器：如文章阅读量、秒杀库存计数，支持原子操作。
- 消息队列：基于 List 的`LPUSH`/`RPOP`实现简单消息队列。
- 分布式会话：存储用户会话信息，实现多服务共享。
- 地理位置：基于 Geo 类型实现附近的人、地址定位功能。

### Redis 数据类型

- 字符串（String）：存储文本、数字，支持原子操作（`INCR`、`DECR`）。
- 哈希（Hash）：存储键值对集合，适合存储对象。
- 列表（List）：有序字符串列表，支持首尾插入删除（`LPUSH`、`RPOP`）。
- 集合（Set）：无序不重复集合，支持交集、并集、差集操作。
- 有序集合（Sorted Set）：按分数排序的集合，适合排行榜场景。
- 其他类型：Geo（地理位置）、BitMap（位图）、HyperLogLog（基数统计）。

## MySQL

### 优化

[[MySql/SQL-optimize\|SQL-optimize]]

###  数据库乐观锁与悲观锁

- **悲观锁**：
    - 原理：假设并发冲突频繁，操作前先锁定资源，禁止其他事务操作，直至当前事务结束。
    - 实现：数据库层面用`SELECT ... FOR UPDATE`语句，Java 层面用`synchronized`、`ReentrantLock`等。
    - 适用场景：写操作频繁、冲突严重的场景。
- **乐观锁**：
    - 原理：假设并发冲突较少，操作时不锁定资源，提交时检查数据是否被修改（通过版本号或时间戳），若修改则重试。
    - 实现：数据库层面添加版本号字段，更新时判断版本号是否一致；Java 层面用`Atomic`系列原子类。
    - 适用场景：读操作频繁、冲突较少的场景。

## 线程池工作原理

[[Java/multiThread/线程池\|线程池]]


## 接口幂等

- **定义**：同一请求多次提交，结果一致，不会因重复提交产生副作用。
- **实现方式**：
    - 基于唯一标识：如请求 ID、订单号，首次处理时存储标识，重复请求时校验并直接返回结果。
    - 乐观锁：通过版本号控制，重复提交时版本号不匹配则拒绝处理。
    - 幂等令牌：客户端先获取令牌，提交时携带令牌，服务端校验令牌有效性。


## JVM

### 垃圾回收算法

- **标记 - 清除算法**：先标记需要回收的对象，再统一清除，优点简单，缺点产生内存碎片。
- **标记 - 复制算法**：将内存分为两块，存活对象复制到另一块，清除原块，优点无内存碎片，缺点内存利用率低。
- **标记 - 整理算法**：标记存活对象后，将其移动到内存一端，再清除剩余部分，兼顾无碎片和高利用率，适用于老年代。
- **分代收集算法**：结合以上算法，新生代用标记 - 复制算法，老年代用标记 - 整理算法，优化回收效率。


### JVM 调优与问题定位

 [从实际案例聊聊Java应用的GC优化](https://tech.meituan.com/2017/12/29/jvm-optimize.html)

- **调优目标**：减少 Full GC 频率、降低内存占用、提升应用响应速度。
- **调优参数**：
    - 内存配置：`-Xms`（初始堆内存）、`-Xmx`（最大堆内存）、`-Xmn`（新生代内存）。
    - 垃圾回收器：G1、ZGC 等，通过`-XX:+UseG1GC`指定。
- **问题定位**：
    - 内存问题：通过`jmap`生成堆 Dump 文件，用 MAT 工具分析内存泄漏。
    - CPU 过高：用`top`命令找到高 CPU 线程，结合`jstack`生成线程栈日志，分析线程阻塞或循环问题。
    - GC 问题：通过`jstat`监控 GC 状态，结合 GC 日志分析回收效率。


## Spring

[[Spring/循环依赖\|循环依赖]]
[[Spring/AOP\|AOP]]
[[Spring/IOC\|IOC]]
[[Spring/Spring事务传播行为\|Spring事务传播行为]]


## Springboot

###  SpringBoot 配置文件加载顺序

- 优先级从高到低：
    1. 命令行参数（`java -jar xxx.jar --server.port=8080`）。
    2. 系统环境变量。
    3. `application-{profile}.properties/yaml`（激活的环境配置）。
    4. `application.properties/yaml`（默认配置）。
    5. 类路径下的`application.properties/yaml`。

### SpringBoot 默认线程数

- 内置 Tomcat 容器：默认核心线程数 10，最大线程数 200，队列容量 10000。
- 异步任务线程池（`@Async`）：默认核心线程数 8，最大线程数`Integer.MAX_VALUE`，队列容量`Integer.MAX_VALUE`。
- 可通过`server.tomcat.threads.core`、`spring.task.execution.pool.core-size`等配置自定义。

###  SpringBoot 全局异常处理

- 实现方式：通过`@RestControllerAdvice`+`@ExceptionHandler`注解。
- 流程：
    1. 定义全局异常处理类，添加`@RestControllerAdvice`注解。
    2. 用`@ExceptionHandler`注解指定处理的异常类型，在方法中统一返回异常响应结果。
- 优势：集中处理异常，减少代码冗余，统一响应格式。

### SpringBoot 处理 HTTP 请求流程

1. 客户端发送 HTTP 请求，由内置 Tomcat 容器接收。
2. Tomcat 将请求封装为`ServletRequest`和`ServletResponse`对象，交给 DispatcherServlet。
3. DispatcherServlet 根据请求 URL 查找对应的 Handler（通过 HandlerMapping）。
4. 调用 HandlerAdapter 执行 Handler 方法，处理业务逻辑。
5. Handler 返回 ModelAndView 或 JSON 数据，DispatcherServlet 通过 ViewResolver 解析视图并渲染（或直接返回 JSON）。
6. 将响应结果返回给客户端。

### SpringBoot 常用注解

- 核心注解：`@SpringBootApplication`（组合`@SpringBootConfiguration`、`@EnableAutoConfiguration`、`@ComponentScan`）。
- 控制器注解：`@RestController`（返回 JSON 数据）、`@RequestMapping`（映射请求路径）、`@GetMapping`/`@PostMapping`（指定请求方式）。
- 依赖注入注解：`@Autowired`（自动注入）、`@Resource`（按名称注入）、`@Value`（注入配置属性）。
- 事务注解：`@Transactional`（声明事务）。

### SpringBoot 事务注解失效场景

- 非 public 方法：`@Transactional`仅对 public 方法生效。
- 方法内部调用：同一类中无事务方法调用有事务方法，事务不生效。
- 异常类型不匹配：默认只捕获 RuntimeException，checked 异常需指定`rollbackFor`属性。
- 数据源未配置事务管理器：未注入`PlatformTransactionManager`。
- 事务传播机制配置不当：如配置为`PROPAGATION_NOT_SUPPORTED`（不支持事务）。

### SpringBoot 分页插件使用与失效处理

- **使用方式**（以 MyBatis-Plus 分页插件为例）：
    1. 配置分页插件：通过`@Bean`注入`MybatisPlusInterceptor`，添加`PaginationInnerInterceptor`。
    2. 代码中使用：用`Page`对象作为方法参数，查询后返回分页结果。
- **失效场景与处理**：
    - 自定义 SQL 未使用分页参数：需手动在 SQL 中添加`LIMIT #{offset}, #{size}`。
    - 插件未配置或扫描路径错误：检查插件配置是否正确，确保拦截器生效。
    - 多表关联查询未处理：确保分页插件支持多表查询，必要时手动构建分页 SQL。

###  SpringBoot 常用组件

- 数据库操作：Spring Data JPA、MyBatis-Plus、Druid 连接池。
- 缓存：Spring Cache（集成 Redis、Ehcache）。
- 消息队列：Spring Kafka、Spring RabbitMQ。
- 服务调用：Feign、OpenFeign。
- 任务调度：`@Scheduled`（定时任务）、Quartz。
- 文档生成：SpringDoc、Swagger。

## Java基础

[Java基础面试16问](https://mp.weixin.qq.com/s/-xFSHf7Gz3FUcafTJUIGWQ)


## Kafka

[[kafka/KafkaRelatedBlog\|KafkaRelatedBlog]]


## git


- **常用命令**：
    - 基础操作：`git init`（初始化仓库）、`git add`（添加文件）、`git commit -m "注释"`（提交）、`git push`（推送远程）。
    - 分支操作：`git branch`（查看分支）、`git checkout -b 分支名`（创建并切换分支）、`git merge`（合并分支）。
    - 版本回退：`git log`（查看日志）、`git reset --hard 版本号`（回退版本）。

- **三次提交合并为一次**：`git rebase -i HEAD~3`，在编辑界面将后两次提交的`pick`改为`squash`，保存后编辑合并注释，完成合并。


## 配置

配置更新实时化的核心目标是：**无需重启应用 / 服务，让修改后的配置快速、一致地生效于所有目标节点**，适用于微服务、分布式系统、客户端应用等场景（如 Java 后端服务的参数调优、金融系统的风控规则更新、APP 的功能开关等）。其原理可拆解为「配置存储 - 变更感知 - 分发推送 - 本地生效」四大核心环节，结合技术实现方案和关键细节如下：

### 一、核心原理框架（四大环节闭环）

#### 1. 配置存储：统一数据源（解决 “配置存在哪”）

- **核心要求**：集中式存储、高可用、支持版本控制 / 回滚、权限管控（避免非法修改）。
- **常见方案**：
    - 专业配置中心：Nacos、Apollo、Spring Cloud Config（Java 后端主流）、Consul；
    - 分布式 KV：etcd、ZooKeeper（依赖 Watcher 机制）；
    - 简单场景：Redis（适合轻量配置，需配合发布订阅）、数据库（如 MySQL，需轮询 + 缓存优化）。
- **存储设计**：配置按「应用 - 环境 - 集群 - 节点」分级（如`app=user-service, env=prod, cluster=shanghai, key=timeout`），支持加密存储（敏感配置如数据库密码、API 密钥）。

#### 2. 变更感知：实时发现配置修改（解决 “怎么知道配置变了”）

这是 “实时化” 的核心触发点，主流有 3 种实现方式，对比如下：

|感知方式|原理|优点|缺点|典型场景|
|---|---|---|---|---|
|推模式（Push）|配置中心主动向订阅节点推送变更（基于长连接 / 消息队列）|实时性最高（毫秒级）、无冗余请求|需维护长连接、节点下线处理复杂|Nacos/Apollo（默认）、etcd|
|拉模式（Pull）|客户端定期轮询配置中心（如每 30 秒），对比本地配置版本号是否更新|实现简单、无需维护连接|实时性差（取决于轮询周期）、浪费带宽|Spring Cloud Config 默认|
|长轮询（Long Poll）|客户端发起请求后，服务端若无变更则 hold 连接（如 30 秒），有变更则立即响应|兼顾实时性（秒级）和低开销|服务端需处理大量 hold 连接，需做连接池优化|Nacos（可选）、Apollo（降级方案）|

- **关键细节**：
    - 版本控制：每个配置项关联版本号（如递增整数、时间戳），客户端通过对比版本号判断是否变更，避免重复处理；
    - 变更校验：配置中心需校验修改的合法性（如格式、取值范围），避免非法配置下发。

#### 3. 分发推送：确保配置一致性（解决 “怎么把新配置发下去”）

- **核心要求**：可靠投递（不丢配置）、顺序性（按修改顺序生效）、一致性（所有目标节点拿到相同配置）。
- **实现方案**：
    - 长连接分发：配置中心与客户端建立 TCP 长连接（如 Netty），变更后通过连接直接推送（Nacos/Apollo 核心方案）；
    - 消息队列分发：配置中心将变更事件写入 MQ（如 Kafka、RabbitMQ），客户端消费 MQ 消息获取新配置（适合大规模集群）；
    - 广播机制：基于 UDP 广播（局域网场景）或组播（如 ZooKeeper 的 Watcher 通知），适用于小型同网段集群。
- **一致性保障**：
    - 重试机制：客户端未收到配置或校验失败时，主动重试拉取（如最多重试 3 次，间隔指数退避）；
    - 分批推送：大规模集群（如千级节点）时，分批次推送（避免服务端压力过载），支持灰度发布（先推部分节点验证）。

#### 4. 本地生效：应用无重启加载配置（解决 “新配置怎么用起来”）

这是 Java 后端面试的重点，核心是「避免重启应用，动态替换内存中的配置对象」，实现方式分 3 类：

##### （1）配置注入层面：基于注解 / 动态代理

- 原理：通过自定义注解（如`@RefreshScope`、`@DynamicConfig`）标记需要动态更新的 Bean，结合 Spring 的 Bean 生命周期，在配置变更时重新创建 Bean 并注入新配置。
- 典型案例（Spring Cloud Config + Spring Cloud Bus）：
	- 用`@RefreshScope`标注 Controller/Service
```Java
	@RestController @RefreshScope // 开启动态刷新 
	public class UserController { 
		@Value("${user.service.timeout:3000}") // 配置项 
		private int timeout; // 接口使用timeout配置 
		@GetMapping("/timeout") 
		public int getTimeout() { 
			return timeout; 
		} 
	}
```
-   配置变更时，Spring Cloud Bus 接收事件，触发`ContextRefresher`刷新`@RefreshScope`标记的 Bean，重新绑定配置值。

##### （2）API 层面：主动获取最新配置

- 原理：封装配置中心客户端 API（如 Nacos 的`ConfigService.getConfigAndSignListener`），应用在关键流程主动调用 API 获取最新配置，或注册监听器回调。
- 典型案例（Nacos 客户端）：
```Java
 // 初始化Nacos配置客户端 
 ConfigService configService = NacosFactory.createConfigService(properties); // 注册配置变更监听器 
	 configService.addListener("user-service-prod", "DEFAULT_GROUP", new Listener() { 
	 @Override 
	 public void receiveConfigInfo(String newConfig) { // 配置变更时的回调：解析新配置并更新本地变量 
		 JSONObject configJson = JSONObject.parseObject(newConfig); 
		 timeout = configJson.getIntValue("user.service.timeout"); 
		 System.out.println("配置更新：timeout=" + timeout);
	} 
	 @Override 
	 public Executor getExecutor() { 
		 return Executors.newSingleThreadExecutor(); // 回调线程池 
	 } 
 });
```

##### （3）底层组件层面：支持动态配置的组件

- 原理：直接使用支持动态更新的中间件 / 组件，避免手动处理配置刷新。
    
    - 连接池：如 HikariCP（数据库连接池）支持动态修改最大连接数（通过`HikariDataSource.setMaximumPoolSize()`）；
    - 缓存：RedisTemplate 的超时时间可通过配置动态更新（重新设置`expire`参数）；
    - 定时任务：Quartz 定时任务的执行周期，可通过更新触发器配置动态生效（无需重启任务）。
- **关键注意事项**：
    
    - 不可变对象：若配置注入到不可变对象（如`final`变量、无 setter 的 POJO），需通过重新创建对象实现更新；
    - 线程安全：更新配置时需加锁（如`ReentrantLock`），避免多线程读取到中间状态的配置；
    - 生效范围：部分配置（如 JVM 参数`-Xms`、端口号）依赖底层系统，无法动态生效，需提前规划可配置项范围。

| 方案                        | 核心组件                            | 实时性         | 无重启生效             | 适用场景           | 优点                    | 缺点               |
| ------------------------- | ------------------------------- | ----------- | ----------------- | -------------- | --------------------- | ---------------- |
| Spring Cloud Config + Bus | Git/SVN（存储）+ RabbitMQ/Kafka（总线） | 中（轮询 / 长轮询） | 支持（@RefreshScope） | Spring 生态微服务   | 无缝集成 Spring、版本控制完善    | 实时性一般、需额外部署总线    |
| Nacos Config              | Nacos Server（存储 + 推送）           | 高（推模式）      | 支持（API / 注解）      | 微服务、分布式系统      | 实时性强、部署简单、支持灰度        | 生态较新、大型集群需优化     |
| Apollo（阿波罗）               | 数据库（存储）+ 长连接（推送）                | 高（推模式）      | 支持（自动刷新）          | 中大型企业、复杂配置场景   | 功能全面（权限 / 灰度 / 回滚）、稳定 | 部署复杂（需多组件）、资源占用高 |
| ZooKeeper + Watcher       | ZooKeeper（存储 + 通知）              | 高（推模式）      | 需自定义实现            | 分布式协调场景（如服务发现） | 一致性强、实时性高             | 配置管理功能弱（需自定义）    |

### 三、面试高频问题延伸

1. **配置更新时如何保证数据一致性？**
    
    - 版本号校验：客户端只处理比本地版本高的配置，避免重复更新；
    - 原子更新：配置中心下发配置时，一次性推送完整配置（而非部分字段），客户端原子替换本地配置；
    - 灰度发布：先推送少量节点，验证配置正确性后再全量推送，避免批量故障。
2. **如果配置中心宕机，应用会受影响吗？**
    
    - 不会直接影响：客户端会缓存本地配置（如磁盘文件 + 内存），宕机后继续使用缓存配置；
    - 容错机制：配置中心恢复后，客户端自动重连并同步最新配置；部分方案支持本地配置文件兜底（如 Nacos 的`spring.cloud.nacos.config.file-extension`）。
3. **如何处理敏感配置（如数据库密码）？**
    
    - 加密存储：配置中心支持 AES/DES 加密（如 Nacos 的加密配置、Apollo 的敏感配置加密），存储密文；
    - 解密流程：客户端启动时通过密钥（如环境变量、本地密钥文件）解密，避免明文传输和存储；
    - 权限管控：配置中心设置细粒度权限（如只读 / 可写权限），仅授权用户可修改敏感配置。
4. **大规模集群（万级节点）如何优化配置推送？**
    
    - 分批推送：按集群 / 机房分批下发，每批间隔一定时间（如 1 秒），避免服务端连接过载；
    - 连接复用：客户端使用长连接池，减少 TCP 连接创建销毁开销；
    - 压缩传输：配置内容较大时（如 JSON/XML），使用 Gzip 压缩后传输，降低带宽消耗；
    - 增量更新：只推送变更的配置字段（而非全量配置），减少数据传输量。