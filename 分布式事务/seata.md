# █ seata 基本教程

# 一. 分布式事务概述

事务基础, ACID

分布式事务的 CAP 理论

柔性事务 BASE

常见的分布式事务模型

- 2PC
- 3PC
- TCC
- SAGA

seata 的 AT 模式, 是两阶段的 补偿型事务, 通过 select for update 锁来实现隔离性



# 二. seata 部署

seata 需要单独部署一个事务协调者的节点, 即 seata server, 从 github 下载

# server 端配置

### file.conf

里面有事务组配置，锁配置，事务日志存储等相关配置信息，由于此demo使用db存储事务信息，我们这里要修改store中的配置：

#### demo 使用本地file存储

```ini
## transaction log store, only used in seata-server
store {
  ## store mode: file、db、redis
  mode = "file"

  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    maxBranchSessionSize = 16384
    # globe session size , if exceeded throws exceptions
    maxGlobalSessionSize = 512
    # file buffer size , if exceeded allocate new buffer
    fileWriteBufferCacheSize = 16384
    # when recover batch read size
    sessionReloadReadSize = 100
    # async, sync
    flushDiskMode = async
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "mysql"
    password = "mysql"
    minConn = 5
    maxConn = 30
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }

  ## redis store property
  redis {
    host = "127.0.0.1"
    port = "6379"
    password = ""
    database = "0"
    minConn = 1
    maxConn = 10
    queryLimit = 100
  }
}
```



#### 应用 使用db管理事务存储

```ini
## transaction log store
store {
  ## store mode: file、db
  # █ 修改为db，表明事务信息用db存储
  mode = "db"

  ## file store 当mode=db时，此部分配置就不生效了，这是mode=file的配置
  file {
    dir = "sessionStore"

    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    max-branch-session-size = 16384
    # globe session size , if exceeded throws exceptions
    max-global-session-size = 512
    # file buffer size , if exceeded allocate new buffer
    file-write-buffer-cache-size = 16384
    # when recover batch read size
    session.reload.read_size = 100
    # async, sync
    flush-disk-mode = async
  }

  ## database store  mode=db时，事务日志存储会存储在这个配置的数据库里
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp) etc.
    datasource = "dbcp"
    ## mysql/oracle/h2/oceanbase etc.
    # █ 根据实际情况, 填写目标数据库配置
    db-type = "mysql"
    driver-class-name = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://116.62.62.26/seat-server"
    user = "root"
    password = "root"
    min-conn = 1
    max-conn = 3
    global.table = "global_table"
    branch.table = "branch_table"
    lock-table = "lock_table"
    query-limit = 100
  }
}
```

### registry.conf



## client 端配置

### file.conf

### registry.conf



关键在于将 DataSource 用 seata 进行托管



# Seata 基本概念

AT 模式

TM: 事务的发起者, 通常是业务入口的微服务, 如果该服务连接了数据库, 该服务会同时充当TM 和 RM

TC: 事务协调者, 即独立部署的 seata-server, 支持集, 也需要自己的一个数据库来管理事务

RM: 资源管理器, 数据库的入口,



## AT 模式的隔离原理

### 写隔离

- 一阶段本地事务提交前，需要确保先拿到 **全局锁** 。
- 拿不到 **全局锁** ，不能提交本地事务。
- 拿 **全局锁** 的尝试被限制在一定范围内，超出范围将放弃，并回滚本地事务，释放本地锁。

两个全局事务 tx1 和 tx2，分别对 a 表的 m 字段进行更新操作，m 的初始值 1000。

1. tx1 先开始，开启本地事务，拿到本地锁，更新操作 `m=1000-100=900`。

2. tx1 本地事务提交前，tx1 先拿到该记录的 **全局锁** ，本地提交释放本地锁。

3. tx2 后开始，开启本地事务，拿到本地锁，更新操作 `m=900-100=800`。

4. tx2 本地事务提交前，尝试拿该记录的**全局锁**，但此时, tx1未完成全局提交，该记录的全局锁被 tx1 持有，tx2 需要重试等待**全局锁**。

5. tx1 执行二阶段, 全局提交或全局回滚

   - 若 tx1 执行全局提交

     1. tx1 二阶段全局提交，释放**全局锁** 。
     2. tx2 拿到 **全局锁** 
     3. tx2 提交本地事务, 释放本地锁。

     ![Write-Isolation: Commit](seata.assets/seata_at-1.png)

   - 若 tx1 要执行全局回滚

     1. 此时 tx1 持有全局锁, 但需要重新获取本地锁，以进行反向补偿的更新操作，实现分支的回滚;
     2. 而 tx2 仍在等待该数据的 **全局锁**, 同时持有该记录的本地锁
     3. 则 tx1无法获取本地锁, tx1回滚会失败, 会一直重试，期间持有全局锁不会释放
     4. tx1 不释放全局锁, 则 tx2 **全局锁** 等锁超时，回滚本地事务释放本地锁
     5. tx1 获取到本地锁, 回滚成功
     6. tx1 释放本地锁, 释放全局锁

     ![Write-Isolation: Rollback](seata.assets/seata_at-2.png)

因为整个过程 **全局锁** 在 tx1 结束前一直是被 tx1 持有的，其他事务无法提交写操作, 所以不会发生 **脏写** 的问题。

### 读隔离

在数据库本地事务隔离级别 **读已提交（Read Committed）** 或以上的基础上，Seata（AT 模式）的默认全局隔离级别是 **读未提交（Read Uncommitted）** (*由于 seata AT 模式实际上是补偿型的分布式事务方案, 需要回滚时再做反向更新. 即便全局事务未提交, 分支事务也是已提交的, 已提交的分支事务会对别的事务可见*)

如果希望达到全局事务的读已提交,  seata 通过全局锁来阻塞别的事务(实际上是串行化 serializable 了), 具体地说, 是通过对 select for update 代理来实现:

1. SELECT FOR UPDATE 语句的执行会申请 **全局锁**
2. 如果 **全局锁** 被其他事务持有，则释放本地锁（回滚 SELECT FOR UPDATE 语句的本地执行）并重试。
3. 这个过程中，select 查询是被 block 住的，直到 **全局锁** 拿到，即读取的相关数据是 **已提交** 的，才返回

![Read Isolation: SELECT FOR UPDATE](seata.assets/seata_at-3.png)

出于总体性能上的考虑，Seata 目前的方案并没有对所有 SELECT 语句都进行代理，仅针对 FOR UPDATE 的 SELECT 语句。

问题

1. 全局锁的粒度? 行锁? 表锁? 间隙锁?

   通过id实现的行级锁? 非id 字段查询都被阻塞? 

2. 

# 常见问题

http://seata.io/zh-cn/docs/overview/faq.html

### Q: 4.怎么使用Seata框架，来保证事务的隔离性？

**A:** 因seata一阶段本地事务已提交，为防止其他事务脏读脏写需要加强隔离。

1. 脏读 select语句加for update，代理方法增加@GlobalLock+@Transactional或@GlobalTransaction
2. 脏写 必须使用@GlobalTransaction
    注：如果你查询的业务的接口没有GlobalTransactional 包裹，也就是这个方法上压根没有分布式事务的需求，这时你可以在方法上标注@GlobalLock+@Transactional 注解，并且在查询语句上加 for update。 如果你查询的接口在事务链路上外层有GlobalTransactional注解，那么你查询的语句只要加for update就行。设计这个注解的原因是在没有这个注解之前，需要查询分布式事务读已提交的数据，但业务本身不需要分布式事务。 若使用GlobalTransactional注解就会增加一些没用的额外的rpc开销比如begin 返回xid，提交事务等。GlobalLock简化了rpc过程，使其做到更高的性能。

### Q: 5.脏数据回滚失败如何处理?

**A:**

1. 脏数据需手动处理，根据日志提示修正数据或者将对应undo删除（可自定义实现FailureHandler做邮件通知或其他）
2. 关闭回滚时undo镜像校验，不推荐该方案。

```
注：建议事前做好隔离保证无脏数据
```

### Q: 10.为什么mybatis没有返回自增ID?

**A:** 方案1.需要修改mybatis的配置: 在`@Options(useGeneratedKeys = true, keyProperty = "id")`或者在xml中指定useGeneratedKeys 和 keyProperty属性
 方案2.删除undo_log表的id字段

### Q: 12.TC如何使用mysql8?

**A:** 1.修改file.conf的驱动配置store.db.driver-class-name;  2.lib目录下删除mysql5驱动,添加mysql8驱动
 ps: oracle同理;1.2.0支持mysql驱动多版本隔离，无需再添加驱动

### Q: 14.使用HikariDataSource报错如何解决?

```
异常1:ClassCastException: com.sun.proxy.$Proxy153 cannot be cast to com.zaxxer.hikari.HikariDataSource
原因: 自动代理时，实例类型转换错误，注入的是$Proxy153实例，不是HikariDataSource的本身或子类实例。
解决: seata自动代理数据源功能使用jdk proxy, 对DataSource进行代理，生成的代理类 extends Proxy implements DataSource, 接收方可改成DataSource接收实现。
1.1.0将同时支持jdk proxy和cglib，届时该问题还可切换cglib解决。
```

### Q: 15.是否可以不使用conf类型配置文件，直接将配置写入application.properties?

**A:** 目前seata-all是需要使用conf类型配置文件，后续会支持properties和yml类型文件。当前可以在项目中依赖seata-spring-boot-starter，然后将配置项写入到application .properties 这样可以不使用conf类型文件。

### Q: 17.Seata 支持哪些 RPC 框架?

**A:**

```
1. AT 模式支持Dubbo、Spring Cloud、Motan、gRPC 和 sofa-RPC。
2. TCC 模式支持Dubbo、Spring Cloud和sofa-RPC。
```

### Q: 20. 使用 AT 模式需要的注意事项有哪些 ？

**A:**

1. **必须使用代理数据源**，有 3 种形式可以代理数据源：
   - **依赖 seata-spring-boot-starter 时，自动代理数据源，无需额外处理。**
   - 依赖 seata-all 时，使用 @EnableAutoDataSourceProxy (since 1.1.0) 注解，注解参数可选择 jdk 代理或者 cglib 代理。
   - 依赖 seata-all 时，也可以手动使用 DatasourceProxy 来包装 DataSource。
2. 配置 GlobalTransactionScanner，使用 seata-all 时需要手动配置，**使用 seata-spring-boot-starter 时无需额外处理。**
3. **业务表中必须包含单列主键**，若存在复合主键，请参考问题 13 。
4. 每个**业务库中必须包含 undo_log 表**，若与分库分表组件联用，分库不分表。
5. 跨微服务链路的事务需要对相应 RPC 框架支持，目前 seata-all 中已经支持：Apache Dubbo、Alibaba  Dubbo、sofa-RPC、Motan、gRpc、httpClient，**对于 Spring Cloud 的支持，请大家引用  spring-cloud-alibaba-seata**。其他自研框架、异步模型、消息消费事务模型请结合 API 自行支持。
6. 目前AT模式支持的数据库有：**MySQL**、Oracle、PostgreSQL和 TiDB。
7. 使用注解开启分布式事务时，**若默认服务 provider 端加入 consumer 端的事务，provider 可不标注注解。但是，provider 同样需要相应的依赖和配置，仅可省略注解**。
8. 使用注解开启分布式事务时，**若要求事务回滚，必须将异常抛出到事务的发起方，被事务发起方的 @GlobalTransactional 注解感知到**。provide 直接抛出异常 或 定义错误码由 consumer 判断再抛出异常。

### Q: 22. AT 模式和 Spring @Transactional 注解连用时需要注意什么 ？

**A:**

`@Transactional` 可与 DataSourceTransactionManager 和  JTATransactionManager 连用, 分别表示本地事务和XA分布式事务，大家常用的是与本地事务结合。

当与本地事务结合时，`@Transactional`和`@GlobalTransaction`连用，`@Transactional` 只能位于标注在`@GlobalTransaction`的同一方法层次或者位于`@GlobalTransaction`  标注方法的**内层**。这里分布式事务的概念要大于本地事务，若将 `@Transactional`  标注在外层会导致分布式事务空提交，当`@Transactional` 对应的 connection  提交时会报全局事务正在提交或者全局事务的xid不存在。

### Q: 24. SpringCloud xid无法传递 ？

**A:**

1.首先确保你引入了**spring-cloud-alibaba-seata**的依赖.

2.如果xid还无法传递,请确认你是否实现了WebMvcConfigurer,如果是,请参考com.alibaba.cloud.seata.web.SeataHandlerInterceptorConfiguration#addInterceptors的方法.把`SeataHandlerInterceptor`加入到你的拦截链路中.

### 注意保证幂等性