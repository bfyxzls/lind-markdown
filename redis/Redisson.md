# Redisson

redisson 是 redis 的一款 Java 平台的客户端, 与 jedis 等客户端不同, redisson 并不是直接暴露 redis api, 而是在 redis api 的基础上, 将 redis 的各种数据类型映射成 java 中的对象, 然后通过这些对象提供了对 redis 的数据操作.

将原生的Redis [`Hash`](http://redis.cn/topics/data-types-intro.html#hashes)，[`List`](http://redis.cn/topics/data-types-intro.html#redis-lists)，[`Set`](http://redis.cn/topics/data-types-intro.html#sets)，[`String`](http://redis.cn/topics/data-types-intro.html#redis-strings)，[`Geo`](http://redis.cn/commands/geoadd.html)，[`HyperLogLog`](http://redis.cn/topics/data-types-intro.html#hyperloglogs)等数据结构封装为Java里大家最熟悉的[`映射（Map）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#71-映射map)，[`列表（List）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#77-列表list)，[`集（Set）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#73-集set)，[`通用对象桶（Object Bucket）`](https://github.com/redisson/redisson/wiki/6.-分布式对象#61-通用对象桶object-bucket)，[`地理空间对象桶（Geospatial Bucket）`](https://github.com/redisson/redisson/wiki/6.-分布式对象#62-地理空间对象桶geospatial-bucket)，[`基数估计算法（HyperLogLog）`](https://github.com/redisson/redisson/wiki/6.-分布式对象#68-基数估计算法hyperloglog)等结构

除了将 redis api 进行封装, redisson 还提供了许多有用的操作, 比如分布式锁,

在这基础上还提供了分布式的[`多值映射（Multimap）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#72-多值映射multimap)，[`本地缓存映射（LocalCachedMap）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#712-本地缓存映射localcachedmap)，[`有序集（SortedSet）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#74-有序集sortedset)，[`计分排序集（ScoredSortedSet）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#75-计分排序集scoredsortedset)，[`字典排序集（LexSortedSet）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#76-字典排序集lexsortedset)，[`列队（Queue）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#78-列队queue)，[`阻塞队列（Blocking Queue）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#710-阻塞队列blocking-queue)，[`有界阻塞列队（Bounded Blocking Queue）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#711-有界阻塞列队bounded-blocking-queue)，[`双端队列（Deque）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#79-双端队列deque)，[`阻塞双端列队（Blocking Deque）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#712-阻塞双端列队blocking-deque)，[`阻塞公平列队（Blocking Fair Queue）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#713-阻塞公平列队blocking-fair-queue)，[`延迟列队（Delayed Queue）`](https://github.com/redisson/redisson/wiki/7.-分布式集合#714-延迟列队delayed-queue)，[`布隆过滤器（Bloom Filter）`](https://github.com/redisson/redisson/wiki/6.-分布式对象#67-布隆过滤器bloom-filter)，[`原子整长形（AtomicLong）`](https://github.com/redisson/redisson/wiki/6.-分布式对象#64-原子整长形atomiclong)，[`原子双精度浮点数（AtomicDouble）`](https://github.com/redisson/redisson/wiki/6.-分布式对象#65-原子双精度浮点数atomicdouble)，[`BitSet`](https://github.com/redisson/redisson/wiki/6.-分布式对象#63-bitset)等Redis原本没有的分布式数据结构。

不仅如此，Redisson还实现了Redis[文档中提到](http://www.redis.cn/topics/distlock.html)像分布式锁[`Lock`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#81-可重入锁reentrant-lock)这样的更高阶应用场景。

事实上Redisson并没有不止步于此，在分布式锁的基础上还提供了[`联锁（MultiLock）`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#83-联锁multilock)，[`读写锁（ReadWriteLock）`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#85-读写锁readwritelock)，[`公平锁（Fair Lock）`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#82-公平锁fair-lock)，[`红锁（RedLock）`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#84-红锁redlock)，[`信号量（Semaphore）`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#86-信号量semaphore)，[`可过期性信号量（PermitExpirableSemaphore）`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#87-可过期性信号量permitexpirablesemaphore)和[`闭锁（CountDownLatch）`](https://github.com/redisson/redisson/wiki/8.-分布式锁和同步器#88-闭锁countdownlatch)这些实际当中对多线程高并发应用至关重要的基本部件。正是通过实现基于Redis的高阶应用方案，使Redisson成为构建分布式系统的重要工具。

# 项目集成

## 直接集成

## Spring集成

## SpringBoot集成

### 添加starter

redisson 提供了 spring boot starter, [官网地址](https://github.com/redisson/redisson/tree/master/redisson-spring-boot-starter)

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.15.6</version>
</dependency>
```

`redisson-spring-boot-starter` 中通过一个 `redisson-spring-data` 的模块来实现对 spring boot 不同版本的兼容, 默认使用最高版本, 目前(2021.6)是`redisson-spring-data-24`, 适用于 spring boot 2.4.x

### 测试代码

```java
private String lock5Second(String orderId) {
    //初始化锁的对象
    RLock rLock = redissonClient.getLock("lock_" + orderId);
    try {
        //尝试加锁, 最多等待5秒
        boolean lock = rLock.tryLock(20, -1, TimeUnit.SECONDS);
        if (lock) {
            log.info("获取到锁，执行支付流程");
            //延时15s
            Thread.sleep(15000);
            log.info("支付完成");
            return "支付完成";
        } else {
            log.info("请稍等，有人正在支付");
            return "请稍等，有人正在支付";
        }
    } catch (InterruptedException e) {
        log.error("获取锁异常 e:{}", e.getMessage());
        return "获取锁异常";
    } finally {
        //是锁定状态，并且是当前执行线程的锁，释放锁
        if (rLock.isLocked() && rLock.isHeldByCurrentThread()) {
            rLock.unlock();
        }
    }
}
```

如果你使用的想在旧版本的 spring boot 中使用 redisson, 你还需要将`redisson-spring-data` 替换为指定版本


| redisson-spring-data module name | Spring Boot version |
| -------------------------------- | ------------------- |
| redisson-spring-data-16          | 1.3.x               |
| redisson-spring-data-17          | 1.4.x               |
| redisson-spring-data-18          | 1.5.x               |
| redisson-spring-data-20          | 2.0.x               |
| redisson-spring-data-21          | 2.1.x               |
| redisson-spring-data-22          | 2.2.x               |
| redisson-spring-data-23          | 2.3.x               |
| redisson-spring-data-24          | 2.4.x               |

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.15.6</version>
    <exclusions>
        <exclusion>
            <artifactId>redisson-spring-data-24</artifactId>
            <groupId>org.redisson</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-data-21</artifactId>
    <version>3.15.6</version>
</dependency>
```

### 配置yaml

可以使用 spring 提供的 redis 通用配置, 对 redisson 进行简单配置, 如下所示

```yaml
spring:
  redis:
    database: 1
    host: 192.168.0.101
    port: 6379
    password: myredis
```

如果需要对 redisson 进行更精细化的配置, 就需要使用 redisson 自己的配置, redisson 的配置与 spring 提供的通用配置不可同时使用, redisson 的配置会覆盖通用配置

```yaml
spring:
  redis:
    redisson:
      # 注意这里有个竖线|, 指示后续内容都是 redisson 的配置
      config: |
        singleServerConfig:
          address: "redis://192.168.0.101:6379"
          database: 1
        codec: !<org.redisson.client.codec.StringCodec> {}
```

注意 `spring.redis.redisson.config` 后有个竖线, 指示后续的下级内容都是 redisson 的配置

如果不清楚如何编写 redisson 的配置内容, 可以借助 `Config` 类来生成

```java
Config config = new Config();
config.setCodec(new StringCodec());
config.useSingleServer()
    .setAddress("redis//192.168.0.101:6379")
    .setDatabase(1);
System.out.println(config.toYAML())
```

生成的yaml 内容如下, 添加到配置文件中时, 可以省略其他默认值的内容

```yaml
---
singleServerConfig:
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  subscriptionsPerConnection: 5
  sslEnableEndpointIdentification: true
  sslProvider: "JDK"
  pingConnectionInterval: 30000
  keepAlive: false
  tcpNoDelay: true
  nameMapper: !<org.redisson.api.DefaultNameMapper> {}
  address: "redis://192.168.0.101:6379"
  subscriptionConnectionMinimumIdleSize: 1
  subscriptionConnectionPoolSize: 50
  connectionMinimumIdleSize: 24
  connectionPoolSize: 64
  database: 1
  dnsMonitoringInterval: 5000
threads: 16
nettyThreads: 32
codec: !<org.redisson.client.codec.StringCodec> {}
referenceEnabled: true
transportMode: "NIO"
lockWatchdogTimeout: 30000
reliableTopicWatchdogTimeout: 600000
keepPubSubOrder: true
useScriptCache: false
minCleanUpDelay: 5
maxCleanUpDelay: 1800
cleanUpKeysAmount: 100
nettyHook: !<org.redisson.client.DefaultNettyHook> {}
useThreadClassLoader: true
addressResolverGroupFactory: !<org.redisson.connection.DnsAddressResolverGroupFactory> {}
```

对于 config 生成的内容, 可以直接添加到 `application.yml` 文件里, 放在 `spring.redis.redisson.config` 后面(注意 config 后面需要添加竖线), 如前面个的例子所示

也可以将配置放置在一个独立的 yaml 文件中,

```yaml
# redisson.yml
singleServerConfig:
  address: "redis://192.168.0.101:6379"
  database: 1
codec: !<org.redisson.client.codec.StringCodec> {}
```

然后在 `application.yml` 中通过 `spring.redis.redisson.file` 指定文件路径

```yaml
# application.yml
spring:
  redis:
   redisson: 
      file: classpath:redisson.yaml
```

### java config

除了使用配置文件, 也可以在 java 代码中构造 Config 对象, 并用 config 对象构造一个 `RedissonClient` 的 `Bean`

```java
@Bean
public RedissonClient redisson() {
    Config config = new Config();
    config.setCodec(new StringCodec());
    config.useSingleServer()
        .setAddress("redis://192.168.0.101:6379")
        .setDatabase(1);
    return Redisson.create(config);
}
```
