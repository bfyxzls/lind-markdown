# █ ActiveMQ 高级配置

# 一. ActiveMQ 的传输协议

## 1. connector

ActiveMQ 提供的用来实现连接通讯的组件, 包括两大类:

1. client - to - broker
2. broker - to - broker

ActiveMQ 允许客户端使用多种协议来连接, 可以在 `conf/activemq.xml` 下进行配置, 详见 [ActiveMQ手册](http://activemq.apache.org/configuring-version-5-transports.html)

ActiveMQ 支持以下的 Client - to - broker 通讯协议: tcp,nio, udp, ssl, http/https, vm

> < transportConnector name="openwire" uri="协议://允许的IP:端口?配置参数=VALUE" />

![1523515193320](ActiveMQ%20高级配置.assetsts/1523515193320.png)

## 2. 主要传输协议

### 1) TCP 协议

是默认的 broker 配置, 默认端口为 61616, 后面可以添加配置参数.

```xml
<transportConnector name="openwire" uri="tcp://0.0.0.0:61616?KEY=VALUE"/>
```

网络传输数据时, 需要对数据进行序列化/反序列化, 消息是通过一个叫 wire protocol 来完成序列化, ActiveMQ 把它称为 OpenWire, 它的目的是促使网络上的数据高效快速地交互.

1. tcp 协议传输可靠性高, 稳定性强
2. 高效性: 字节流方式传递, 效率很高
3. 有效性, 可用性: 应用广泛, 支持任何平台

### 2) NIO 协议

与 TCP 协议类似, 但更侧重于底层的访问操作, 允许开发人员对统一资源有更多的 client 调用, 且服务端能承担更高的负载

```xml
<transportConnector name="nio" uri="nio://0.0.0.0:61676?KEY=VALUE"/>
```

适用场景:

1. 可能有大量的 client 访问 broker: 

   通常情况下, 大量的client 去连接 broker 时会收到操作系统的线程数所限制, 而 NIO 的实现比 TCP 需要更少的线程

2. 可能对 broker 有一个很迟钝的网络传输

   NIO 比 TCP 能提供更好的性能

### 3) UDP 协议

```xml
<transportConnector name="udp" uri="udp://0.0.0.0:61619?KEY=VALUE"/>
```

TCP 是一个原始流的传输协议, 稳定可靠, 数据包在传递过程中不会被复制和丢失的, 而 UDP 并不保证数据包的传递

UDP 通常用在快速数据传递和不怕数据丢失的场景中

如果需要 ActiveMQ 通过防火墙, 也只能使用 UDP

### 4) SSL 协议

安全的连接协议

```xml
<transportConnector name="ssl" uri="ssl://0.0.0.0:61676?KEY=VALUE"/>
```

### 5) HTTP/HTTPS 协议

像 web 和 email 等服务需要通过防火墙来访问的, 可以使用 http 协议. 但 mq 一般作为中间件, 较少使用这种方式

```xml
<transportConnector name="http" uri="http://0.0.0.0:61676?KEY=VALUE"/>
```

### 6) VM 虚拟机通讯协议

[官方文档- vm通讯协议](http://activemq.apache.org/vm-transport-reference.html)

对于单系统项目使用嵌入式的 broker, VM 协议允许在虚拟机内部通讯, 从而避免了网络传输的开销. 

这是采用的不是 socket 连接, 而是直接的方法调用.

第一个创建 vm 连接的客户会启动一个 embed vm broker, 接下来所有使用相同 broker name 的 vm 连接都会使用这个 broker,

当这个 broker 上所有的链接都关闭的时候, 这个 broker 也会自动关闭



#### a. 简单 vm broker

可以启动一个简单的 vm broker, 但只支持少量的配置项

```xml
vm://brokerName?transportOptions
```

#### b. 高级 vm broker

在 java 中嵌入的方式, 定义了一个名为 BROKER 的broker, 并配置了一个 tcp 连接器

这种语法可以启动一个高级的 vm broker, 允许用户定义更多的配置项, 其中 broker 配置参考第7小节 **[通过 URI 配置 BROKER]**

> vm: broker配置 ? 传输配置

```xml
vm:(broker:(tcp://localhost)?brokerOptions)?transportOptions
vm:broker:(tcp://localhost)?brokerOptions
```

如下所示

```java
vm:broker:(tcp://localhost:6000)?brokerName=BROKER
```

#### c. 配置文件 vm broker

也可以通过加载一个配置文件来启动 broker

```xml
vm://localhost?brokerConfig=xbean:activemq.xml
```

### 7) 通过 URI 配置 BROKER

可以通过一个 URI 对 broker (尤其是嵌入式 broker) 参数进行配置, 支持以下三种方式:

#### a. xbean

最灵活最强大的配置方式, 通过一个额外的 xml 配置文件来进行配置, 但需要额外的 jar 包依赖

- 指向 classpath 的文件

  ```xml
  xbean:filename
  vm://localhost?brokerConfig=xbean:filename
  ```

- 指向其他路径文件

  ```xml
  xbean:file:filepath
  vm://localhost?brokerConfig=xbean:file:filepath
  ```

- 指向网络文件

  ```xml
  xbean:URL_path
  vm://localhost?brokerConfig=xbean:URL_path
  ```

#### b. broker

使用 broker URI 的格式配置 broker

```xml
broker:(tcp://localhost:61616,network:static:tcp://remotehost:61616)?persistent=false&useJmx=true
```

#### c. properties

 使用一个 properties 文件进行配置

```properties
# START SNIPPET: example
useJmx=false
persistent=false
brokerName=Cheese
# END SNIPPET: example
```



# 二. ActiveMQ 的消息持久化

ActiveMQ 不仅支持 **持久化**(persistent) 和**非持久化**(non-persistent) 两种方式, 还支持消息的**恢复**(recovery) 

1. 点对点模式

  队列的存储比较简单, 就是一个 FIFO(先进先出) 的 queue

  ![img](ActiveMQ%20高级配置.assetsts/897247-20170209150723041-822523364.png)

2. 发布/订阅模式

  对于持久化订阅主题, 每一个消费者将获得一个消息的复制

  ![img](ActiveMQ%20高级配置.assetsts/897247-20170209150745510-1362013534.png)

## 1. 插件式的消息存储

ActiveMQ 提供了一个插件式的消息存储, 通过不同的插件, 实现不同的持久化方式, 类似于消息的多点传播

可以在 `conf/activemq.xml` 文件下对持久化方式进行配置

1. AMQ消息存储, 基于文件的存储方式, 是旧版本 AMQ 的默认消息存储
2. KahaDB, 针对消息队列优化的数据库, 提供容量提升和恢复能力, 是当前版本的默认存储方式
3. JDBC, 基于外部数据库的消息存储方式
4. Memory, 基于内存的消息存储方式

## 2. kahaDB

### 1) 概述

kahaDB 是 ActiveMQ 的默认方式, 可用于任何场景, 提高了性能和恢复能力

使用一个事务日志和仅仅一个索引文件来存储它所有的地址

kahaDB 是一个专门针对消息持久化的解决方案, 对典型的消息使用模式进行了优化,.

在 kahaDB 中, 数据被追加到 datalogs 中, 当不在需要 log 文件中的数据的时候, log 文件会被丢弃

### 2) 配置

```xml
<persistenceAdapter>
	<kahaDB directory="${activemq.data}/kahadb"/>
</persistenceAdapter>
```
可用属性详见 [kahaDB参数文档](http://activemq.apache.org/kahadb.html)

### 3) 嵌入式broker

```java
public class KahaDBBroker {
    public static void main(String[] args) throws Exception{
        BrokerService broker = new BrokerService();
        
        // 设置 kahadb
        File dataFileDir = new File("target/amq/kahadb");
        KahaDBStore kaha = new KahaDBStore();
        kaha.setDirectory(dataFileDir);
        kaha.setJournalMaxFileLength(1024*100);
        kaha.setIndexWriteBatchSize(100);
        kaha.setEnableIndexWriteAsync(true);
        broker.setPersistenceAdapter(kaha);
        
        broker.setUseJmx(true);
        broker.addConnector("tcp://localhost:61666");
        broker.start();
    }
}
```

## 3. AMQ Message Store

```xml
<persistenceAdapter>
    <amqPersistenceAdapter directory="${activemq.base}/activemq-data" maxFileLength="32mb"/>
</persistenceAdapter>
<transportConnectors>
    <transportConnector uri="tcp://localhost:61616"/>
</transportConnectors>
```

## 4. JDBC

### 1) 概述

ActiveMQ 支持使用 JDBC 来持久化消息, 要求先在数据库中按规则创建好数据库表

使用 jdbc 进行持久化之前, 应该先将 jdbc 需要的包放到 `activemq/lib` 目录之下

比如使用 dbcp+mysql 进行持久化, 就需要以下三个包:

- commons-dbcp-1.4.jar
- commons-pool-1.4.jar
- mysql-connector-java-6.0.4.jar

### 2) 配置

如果使用 MySQL, 还应注意将 relaxAutoCommit 设为 true (避免不支持事务的 mysql 版本引发错误)

```xml
<!-- mysql-ds -->
<bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
  <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://localhost:3306/activemq?
                              useUnicode=true
                              &amp;characterEncoding=UTF-8
                              &amp;relaxAutoCommit=true"/>
  <property name="username" value="activemq"/>
  <property name="password" value="activemq"/>
  <property name="poolPreparedStatements" value="true"/>
</bean>

<broker>
    ...
    <persistenceAdapter>
        <!-- 添加 jdbcAdapter, 用井号引用 mysql-ds -->
        <jdbcPersistenceAdapter dataSource="#mysql-ds" />
    </persistenceAdapter>
    ...
</broker
```
### 3) 数据库表结构

在 actvemq 连接到数据库时, 会自动创建以下几个表

1. 消息表

   默认表名为 ACTIVEMQ_MSGS, queue 和 topic 都存在这个表里

   ```mysql
   CREATE TABLE `ACTIVEMQ_MSGS` (
     `ID` bigint(20) NOT NULL,
     `CONTAINER` varchar(250) NOT NULL,
     `MSGID_PROD` varchar(250) DEFAULT NULL,
     `MSGID_SEQ` bigint(20) DEFAULT NULL,
     `EXPIRATION` bigint(20) DEFAULT NULL,
     `MSG` longblob,
     `PRIORITY` bigint(20) DEFAULT NULL,
     `XID` varchar(250) DEFAULT NULL,
     PRIMARY KEY (`ID`),
     KEY `ACTIVEMQ_MSGS_MIDX` (`MSGID_PROD`,`MSGID_SEQ`),
     KEY `ACTIVEMQ_MSGS_CIDX` (`CONTAINER`),
     KEY `ACTIVEMQ_MSGS_EIDX` (`EXPIRATION`),
     KEY `ACTIVEMQ_MSGS_PIDX` (`PRIORITY`),
     KEY `ACTIVEMQ_MSGS_XIDX` (`XID`)
   ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
   ```

   ![1524195068617](ActiveMQ%20高级配置.assetsts/1524195068617.png)

2. 订阅表

   默认表名 ACTIVEMQ_ACKS, 存储持久订阅的消息和最后一个持久订阅接收的消息 ID

   ```mysql
   CREATE TABLE `ACTIVEMQ_ACKS` (
     `CONTAINER` varchar(250) NOT NULL,
     `SUB_DEST` varchar(250) DEFAULT NULL,
     `CLIENT_ID` varchar(250) NOT NULL,
     `SUB_NAME` varchar(250) NOT NULL,
     `SELECTOR` varchar(250) DEFAULT NULL,
     `LAST_ACKED_ID` bigint(20) DEFAULT NULL,
     `PRIORITY` bigint(20) NOT NULL DEFAULT '5',
     `XID` varchar(250) DEFAULT NULL,
     PRIMARY KEY (`CONTAINER`,`CLIENT_ID`,`SUB_NAME`,`PRIORITY`),
     KEY `ACTIVEMQ_ACKS_XIDX` (`XID`)
   ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
   ```

   ![1524195211128](ActiveMQ%20高级配置.assetsts/1524195211128.png)

3. 锁定表


   默认表名 ACTIVEMQ_LOCK, 用来确保在某一时刻, 只能有一个 broker 实例来访问数据库

   ```mysql
CREATE TABLE `ACTIVEMQ_LOCK` (
  `ID` bigint(20) NOT NULL,
  `TIME` bigint(20) DEFAULT NULL,
  `BROKER_NAME` varchar(250) DEFAULT NULL,
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
   ```

![1524195331527](ActiveMQ%20高级配置.assetsts/1524195331527.png)

## 5. Jdbc with journal

### 1) 概述

单纯使用 jdbc 时, 由于数据库读写速度较慢, 影响消息队列的性能

ActiveMQ 为此提供了一种 journal 机制, 使用快速缓存写入技术, 大大提高了性能

- JDBC with journal 的性能优于单独的 JDBC
- JDBC 可以用在 master/slave 模式的数据库分享
- JDBC with journal 不可用在 master/slave 模式
- 如果不需要使用集群, 推荐使用 JDBC with journal 

### 2) 配置

使用 journal, 需要在 `conf/activemq.xml`文件中修改持久化的设置, 将 persistenceAdapter **替换成** **persistenceFacotry**

```xml
<broker>
    ...
    <persistenceFactory> 
        <journalPersistenceAdapterFactory 
			journalLogFiles="5" 
             journalLogFileSize="32768"
             useJournal="true"
             useQuickJournal="true"
			dataDirectory="activemq-data" 
			dataSource="#mysql-ds"/> 
    </persistenceFactory> 
	...
</broker>
```

## 6. Memory Message Store

### 1) 概述

内存消息存储主要把须有持久化的消息存在内存中, 这里没有动态的缓存, 所以用户必须注意设置 broker 所在的虚拟机和内存限制

其实就是关闭持久化, 那么 ActiveMQ 就只会使用内存来作为消息存储空间

### 2) 配置

主要就是在broker 中将 **persistent 设为 false**, 并删除 persistenceAdapter 或 persistenceAdapterFactory 相关配置

```xml
<broker brokerName="localhost" persistent="false">...</broker>
```

###3) 嵌入式 broker

```java
public static void main(String[] args) throws Exception{
    BrokerService broker = new BrokerService();
    broker.setUseJmx(true);
    broker.setPersistent(false);
    broker.addConnector("tcp://localhost:61666");
    broker.start();
}
```

# 三. ActiveMQ 的网络连接

## 1. 启动多个 broker

1. 将 conf 文件复制多份

2. 修改 `activemq.xml` 

   1. brokerName 不能重复, 修改为 broker2

      ```xml
      <broker xmlns="http://activemq.apache.org/schema/core" brokerName="broker2" dataDirectory="${activemq.data}">
      ```

   2. 数据存放文件不能重复, 修改为 kahadb2

      ```xml
      <persistenceAdapter>
          <kahaDB directory="${activemq.data}/kahadb2"/>
      </persistenceAdapter>
      ```

   3. 所有端口都不能重复, 这里只保留 tcp 端口为 61617

      ```xml
      <transportConnectors>
          <transportConnector name="openwire" uri="tcp://0.0.0.0:61617?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
      </transportConnectors>
      ```

3. 修改 `jetty.xml`, 修改控制台访问端口为 8162

   ```xml
   <bean id="jettyPort" class="org.apache.activemq.web.WebConsolePort" init-method="start">
       <property name="host" value="0.0.0.0"/>
       <property name="port" value="8162"/>
   </bean>
   ```

   ​

4. 到 bin 目录下, 复制一个 `env` 

   修改其中的 `ACTIVEMQ_QUEUEMANAGERURL` , 与 `conf/activemq.xml` 中 tcp 端口保持一致 

   ```bash
   # Specify the queue manager URL for using "browse" option of sysv initscript
   if [ -z "$ACTIVEMQ_QUEUEMANAGERURL" ]; then
       ACTIVEMQ_QUEUEMANAGERURL="--amqurl tcp://localhost:61616"
   fi
   ```

5. 到 bin 目录下, 复制一个 `activemq`

   1. 修改程序的`ACTIVEMQ_PIDFILE` , 修改为 activemq2.pid

      ```sh
      # Location of the pidfile
      if [ -z "$ACTIVEMQ_PIDFILE" ]; then
        ACTIVEMQ_PIDFILE="$ACTIVEMQ_DATA/activemq2.pid"
      fi
      ```

   2. 修改配置文件路径 `ACTIVEMQ_CONF` , 指向 conf2 目录

      ```shell
      # Active MQ configuration directory
      if [ -z "$ACTIVEMQ_CONF" ] ; then

          # For backwards compat with old variables we let ACTIVEMQ_CONFIG_DIR set ACTIVEMQ_CONF
          if [ -z "$ACTIVEMQ_CONFIG_DIR" ] ; then
              ACTIVEMQ_CONF="$ACTIVEMQ_BASE/conf2"
          else
              ACTIVEMQ_CONF="$ACTIVEMQ_CONFIG_DIR"
          fi
      fi
      ```

   3. 修改环境配置`ACTIVEMQ_CONFIGS` , 指向 `bin/env2` 

      ```shell
      # LOAD CONFIGURATION

      # CONFIGURATION
      # For using instances
      if ( basename $0 | grep "activemq-instance-" > /dev/null);then
        INST="`basename $0|sed 's/^activemq-instance-//;s/\.sh$//'`"
        ACTIVEMQ_CONFIGS="/etc/default/activemq-instance-${INST} $HOME/.activemqrc-instance-${INST}"
        echo "INFO: Using alternative activemq configuration files: $ACTIVEMQ_CONFIGS"
      else
        ACTIVEMQ_CONFIGS="/etc/default/activemq $HOME/.activemqrc $ACTIVEMQ_HOME/bin/env"
      fi
      ```

## 2. 静态网络连接

在某些场景下, 需要多个 ActivMQ 的 broker 做集群, 就需要 broker - broker 的通信, 这个被称为 ActiveMQ 的 networkConnector.

ActiveMQ 的 networkConnector 默认是单向的, 称为桥接, 如图中 broker1 和 broker2

也支持双向连接, 创建一个双向的通道, 称为 duplex connector, 如图中 broker1 和 broker3

![1524214752386](ActiveMQ%20高级配置.assetsts/1524214752386.png)

### discovery

discovery 用来发现远程的服务, 客户端会去发现的所有可以利用的 broker, 另一层意思, 它是基于现有的网络 broker 去发现其他可用的broker

有两种配置 client 到broker 的链接方式:

1. client 通过 statically 配置的方式去链接broker
2. client 通过 discovery agent 来动态发现 broker

### 静态网络 Static networks

### 1) 单向桥接

![1524215778647](ActiveMQ%20高级配置.assetsts/1524215778647.png)

上图中, 两个brokers 是通过一个 static 的协议来网络连接的, 一个 consumer 连接到 brokerB 的一个地址上, 当 producer 在 brokerA 上以相同的目的地发送消息时, 消息将被转移到 brokerB 上, 也就是消息从 brokerA 转发给了 brokerB

Static networkConnector 是用于创建一个静态的配置, 去对应网络中的多个 broker

这种协议用于复合 url, 一个复合 url 包含多个 url 地址

> static:(uri1,uri2, uri3)?KEY=VALUE

在主 broker 中添加以下配置, 从 broker 不需修改

```xml
<networkConnectors>
    <networkConnector name="local_network" uri="static:(tcp://host1:61616,tcp://host2:61617)?KEY=VALUE" />
</networkConnectors>
```

### 2) 双向桥接

![1524540158902](ActiveMQ%20高级配置.assetsts/1524540158902.png)

双向桥接如上图所示, 

producer A1 发到 brokerA 的消息, 可以直接被 consumer A1 消费, 也可以经 brokerB 转发后被 consumerB1消费;

producer B1 发到 brokerB 的消息, 可以直接被 consumer B1 消费, 也可以经 brokerA 转发后被 consumerA1消费;

但经 brokerB 拉取后的消息, 不能回流到 brokerA 中被 consumer A1 消费.

如果需要消息回流, 需要另外的配置, 后面再讲.

双向桥接的配置与单向桥接类似, 在其中一个broker添加配置(不分主从), 但需增加一个 `duplex="true"` 的属性配置

```xml
<networkConnectors>
    <networkConnector name="local_network" duplex="true"
                      uri="static:(tcp://host1:61616,tcp://host2:61617)?KEY=VALUE" />
</networkConnectors>
```



## 3. 网络连接的属性

详见 [官方文档-网络连接属性](http://activemq.apache.org/networks-of-brokers.html)

- name, 默认为 bridge
- dynamicOnly, 默认为false, 启动时激活; 如果为true, 持久订阅被激活后才创建相应的网络持久订阅,
- decreaseNetworkConsumerPriority, 设定是否降低网络消费者优先权, 默认false. 优先级为0, true则优先级为-5 
- networkTTL, 网络中用于消息和订阅消费的broker 数量, 默认1
- messageTTL, 网络中用于消息的broker 数量, 默认1
- consumerTTL, 网络中用于订消费的 broker 数量, 默认1
- conduitSubscriptions, 是否把从属 broker 中多个consumer 当成一个来处理. 默认 true
- dynamicallyIncludedDestinations, 要包括的动态消息地址, 默认空
- staticallyIncludedDestinations, 要包括的静态消息地址, 默认空
- excludedDestinations, 要排除的地址, 默认空
- duplex, 是否双向通信, 默认 false
- prefetchSize, 预拉取的最大消息数量, 必须大于0, 默认1000
- suppressDuplicateQueueSubscriptions, 是否允许重复订阅, 默认 false, 重复订阅一产生就被阻止
- bridgeTempDestinations, 是否广播 advisory messages 来创建临时目的地, 默认 true
- alwaysSyncSend, 是否同步发送, 默认false, 如果 true, 则非吃u话消息也将使用 request/reply 方式发送到远程 broker
- staticBridge, 静态桥接, 默认false. 如true, 则只处理在 staticallyIncludedDestinations 中配置的目的地


## 4. 消息的丢失与回流

可以将 brokerB 视为BrokerA 中的一个特殊的消费者, 且比普通的消费者拥有更高的优先级.

brokerB 从 brokerA 中拉取消息时, 并不是一条一条地获取, 而是批量获取. 这可能会带来一个问题, 一批消息被 brokerB 取走, 但brokerB 上有消息却没有消费者, brokerA 上有消费者却没有消息

brokerA 和 brokerB 通过 networkConnection 连接, 一些 消费者连接到 brokerB, 消费 brokerA 上的消息.

消息先从 brokerA 转发到 brokerB, 然后转发给 brokerB 上的消费者.

但在消费者从 brokerB 消费部分消息时, brokerB 重启了, 消费者发现 brokerB 连接失败, 通过 failOver 连接到 brokerA 中. 但是因为 brokerA 上的消息已经被转发到 brokerB 上, brokerA 中已经没有消息, 这种情况下, 消息就好像消失了一般, 即便 brokerB 重启, 但若没有消费者重新连接到 brokerB 上, 就会出现 A有消费者无消息, B有消息无消费者的情况 

从 5.6 版本开始, 在 destinationPolicy 上新增的选项 replayWhenNoConsumers, 这个选项使得 brokerB 上有需要转发的消息但是没有消费者时, 把消息回流到原始的 brokerA, 同时把 enableAudit 设置为 false, 为了防止消息回流后被当作重复消息而不被分发.

消息的回流

需要在两个 broker 中都添加如下配置, 如果只配置了 A, 则只有 A 会将 B 的消息回流到 B, 而 B 拿走了 A 的消息后不会归还.

```xml
<destinationPolicy>
	<policyMap>
    	<policyEntries>
        	<policyEntry queue=">" enableAudit="false">
            	<networkBridgeFilterFactory>
                	<conditionalNetworkBridgeFilterFactory replayWhenNoConsumers="true" />
                </networkBridgeFilterFactory>
            </policyEntry>
        </policyEntries>
    </policyMap>
</destinationPolicy>
```

消息的丢失和回流, 只针对于点对点模式, 订阅模式由于每个broker 都会保存一份消息的拷贝, 不存在此问题.

## 5. 容错连接

在 client 尝试连接到 broker 时, 若 broker 挂了, client 有两个选择: 

1. 立刻结束 client
2. 尝试连接其他的 broker

failover 协议实现了自动重连的逻辑, 可以让 client 在一个 broker 失败后尝试连接另一个 broker, 可以用以下两种方式配置:

1. 提供一个静态的可用 broker 列表, client 从中选择一个去连接
2. 提供一个动态发现机制, 动态发现可用的 broker

### 1) static brokers 列表

>failover:(URI1, URI2, URI3)?KEY=VALUE
>
>failover: URI1, URI2, URI3...

默认情况下, failover 协议用于随机选择一个链接去链接, 如果连接失败了, 就会连接到其他的 broker 上

默认的配置定义了延迟重新连接, client 会在十秒后自动取连接其他可用的 broker

[参数列表](http://activemq.apache.org/failover-transport-reference.html)

![1524558552268](ActiveMQ%20高级配置.assetsts/1524558552268.png)

## 6. 动态网络链接

### 1) 多播协议 Multicast

ActiveMQ 使用 Multicast 协议将一个 service 和其他的 broker 的service 连接起来, 需要多台提供 activemq 的服务器组成网络, 才能使用多播协议

IP Multicast 是一个被用于网络中传输数据到其他一组接收者的技术

传功同年的概念称为组地址, 组地址是IP地址在 224.0.0.0 到 239.255.255.255 之间的 IP 地址

ActiveMQ 使用 multicast 协议去接力服务于远程的 broker 的服务的网络连接

基本格式配置:

> multicast://ipadaddress:port?KEY=VALUE

具体参数详见: [官方文档 - 多播协议参数配置](http://activemq.apache.org/multicast-transport-reference.html)

![1524558963634](ActiveMQ%20高级配置.assetsts/1524558963634.png)

默认是不可靠的多播, 数据包可能会丢失

> multicast://default

特定的ip 和端口

> muticast://224.1.2.3:6255

特定的 IP, port, group

> mutilcast:224.1.2.3:6255?group=GROUP_NAME

```XML
<broker xmlns="http://activemq.apache.org/schema/core" brokerName="multicast" dataDirectory="${activemq.data}">
    <networkConnector>
        <networkConnector name="default-nc" uri="mutilcast://default" />
    </networkConnector>
    <transportConnector>
        <transportConnector name="openwire" uri="tcp://0.0.0.0:61616" discoveryUri="multicast://default"/>
    </transportConnector>
</broker>
```

如何防止自动的寻找地址

1. 名称为 openwire 的 transport, 移除 discoveryUri="multicast://default" 即可

   传输链接用默认的名称 openwire 来配置 broker 的多点连接, 这将允许其他 broker 能够自动发现和链接到可用的broker 中

2. 名称为 default-nc 的 networkConnector, 注释掉或者删除即可

   ActiveMQ 默认的 networkConnector 基于 multicast 协议的链接名称为 default-nc, 而且自当的去发现其他的broker, 要停止这种行为, 只需要注销或者删除 default-nc 网络连接

3. 使用 唯一个 brokername, 可以唯一识别 broker 的实例, 默认为localhost

multicast 和 tcp 协议

区别在于 multicast 能够自动的发现其他的 broker, 从而替代了使用 static 列表 brokers, 

用 multicast 协议可以在网络中频繁的天界和删除 ip 也不会影响功能正常运作

优点: 能够适应动态变化的地址

缺点: 自动的连接地址, 过度消耗网络资源

Discovery

是在 multicast 协议的功能上定义的, 功能类似于 failover.

它将动态地发现 mutilcast 协议的broker 的链接, 并且随机连接其中一个 broker

peer 协议

fanout 协议

同时链接多个broker

## 7. ActiveMQ 的集群

### 1) queue consumer 的集群

ActiveMQ 支持 从consumer 对消息高可靠性的负载平衡消费, 

如果一个 consumer 挂了, 该消息会转发到其他 consumer 消费的queue 上, 

如果一个 consumer 获得消息比其他 consumer 快, 那么他将获得更多的消息, 

因此推荐 ActiveMQ 的 broker 和 client 使用 **failover://transport** 的方式来配置链接



默认情况下, 当有多个 consumer 连接到 broker 网络时, 则连接到同一个 broker 的多个 consumer, 会被认为是一个消费者, 即把 整个 broker 视为一个消费者, 这在某些情况下不利于负载均衡.

如果需要将连接到同一个 broker 的每个 consumer 都视为独立的,  需要在静态网络设置的地方, 添加 **"conduitSubscriptions=false"** 的设置

```xml
<networkConnectors>
    <networkConnector name="local_network" duplex="true" conduitSubscriptions="false"
                      uri="static:(tcp://localhost:61616,tcp://localhost:61617)" />
</networkConnectors>
```

### 2) broker 集群

大多数情况下, 会使用一系列的 broker, 和 client 链接到一起, 如果一个 broker 死掉了, client 可以自动连接到其他 broker 上, 实现以上行为需要用 failover 协议来配置 client.

如果启动了多个 broker, client 可以使用静态列表或者动态发现, 很容易地从一个 broker 切换到另一个 broker.

当一个 broker 上没有 consumer 时, 它的消息不会被消费, 但该 broker 会通过存储和转发的策略来把该消息发到其他 broker 上

特别注意: ActiveMQ 默认的两个broker , static 链接后是单方向的, brokerA 可以访问 brokerB 的消息, 如果要支持双向通讯, 需要添加 **duplex=true** 配置

- 静态网络连接
- 动态网络链接

### 3) Master-Slave 集群

ActiveMQ 支持多种主从备份的集群方式:

1. Shared File System Master Slave

   给予共享存储的的 Master-Slave, 多个 broker 实例使用一个存储文件, 谁拿到文件锁就是 master, 其他的处于待启动状态, 如果 master 挂掉了, 某个抢到文件锁的 slave 变成 master

2. JDBC Master Slave

   基于 JDBC 的master-slave, 使用同一个数据库, 拿到 LOCK 表的写锁的 broker 变成 master

3. Replicated LevelDB Store

   基于 zooKeeper 复制 LevelDB 存储的 master-slave 机制, 5.9 新加的功能, 暂不讲解

JDBC Master Slave

利用数据库作为数据源, 采用 master/slave 模式, 其中在启动的时候 master 首先获得独有锁, 其他 slave broker 等待获取独有锁.

推荐客户端使用 failover 来链接 broker

![1526284040469](ActiveMQ%20高级配置.assetsts/1526284040469.png)

master 失败

如果 master 失败, 则它释放独有锁, 其他 slave 获取独有锁, 升级为 master, 并启动所有的传输链接.

同时, client 将停止链接之前的master, 并轮询连接到其他可以利用的broker 即新的master

master 重启

任何时候去启动新的broker, 都是作为新的 slave 来加入集群, 直到当前master 挂掉以后才能获取独有锁

当多个使用 `<jdbcPersistenceAdapter/>` 来配置消息的持久化, 使用同一个数据库时, 自动就会使用 JDBC Master Slave方式

# 四. Destination 高级特性

## 1. 分层命名

ActiveMQ 支持 destination 的分层命名体系, 这不属于 JMS 中定义的规范, 是 ActiveMQ 对 JMS 的扩展

1. `.` 小数点, 用来作为分层命名的分隔符
2. `*` 星号, 通配符, 用来匹配当前层级的所有单词, 不包括 `.` 等下级路径
3. `>` 大于号, 通配符, 用来匹配前缀, 包括`.` 等下级路径, 单独的 `>` 表示所有 destination

## 2. 虚拟 Destination

虚拟 destination 用来创建逻辑 destination, 客户端可以通过这个虚拟的 destination 来生产和消费消息, 它会把消息映射到真是存在的物理destination,

ActiveMQ 支持两种虚拟 destination, 

- 虚拟主题: Virtual Topic
- 组合路径: Composite Destination

### 1) 虚拟主题 virtual topic

#### a. 为何使用虚拟主题

ActiveMQ 中, topic 只有在存在持久订阅者时, 才会将消息持久化

而持久订阅时, 每个持久订阅者, 都相当于有一个专属的 queue, 会收取所有的消息.

这会存在以下两个问题, 

1. 同一应用内的 consumer 负载均衡的问题, 

   同一个应用上的一个持久订阅, 不能使用多个 consumer 来共同承担消息处理功能, 因为每个 consumer 都会收到所有消息.

   queue模式可以解决这个问题, 但是broker 又不能将消息发送到多个应用端, 

   JMS 规范中无法实现既要发布订阅, 又能让消费者分组

2. 同一应用内 consumer 端 failover 的问题

   由于只能使用单个的持久订阅者, 如果这个订阅者出错挂掉, 则应用就无法处理消息了, 系统健壮性不高

为了解决这两个问题, Activemq 提供了虚拟 topic 的功能, 使得:

- destination 对发布者而言, 是一个 topic, 发布的消息可以被多个应用端使用
- destination 对某个应用端内的多个消费者而言, 是一个 queue, 多个消费者可以分工处理队列中的消息

注意, 使用虚拟主题时, 不可配置 组合destination 的 destinationInceptor, 否则会失效, **原因不明**

#### b. 用法

- **发布者**

  对于消息发布者来说, 这就是一个正常的 topic, 但名称必须以 `VirtualTopic.` 开头

  > Destination destination = session.createTopic("**VirtualTopic.>**");

  ```java
  // 创建会话, 不开启事务, 通过自动模式收发消息
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  Destination destination = session.createTopic("VirtualTopic.vt1");
  
  // 创建生产者
  MessageProducer producer = session.createProducer(destination);
  
  for(int i = 0; i< 5; i++){
      TextMessage message = session.createTextMessage();
      message.setText("这是VirtualTopic的消息"+i);
      producer.send(message);
      System.out.println("已发送消息:"+i);
  }
  ```

- **消费者**

  对于消费者来说, 将目的视为一个 queue, 而名称要求为 `Consumer.*.VirtualTopic.>`, 

  用 `Consumer.*` 前缀表明该 consumer 属于哪个应用, 后面的 `Virtual.>` 表示原始的虚拟主题

  > Destination destination = session.createQueue("**Consumer.*.VirtualTopic.>**");

  ```java
  // 创建会话, 不开启事务, 通过自动模式收发消息, 不需要手动签收
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  Destination destination = session.createQueue("Consumer.B.VirtualTopic.vt1");
  
  // 创建消费者
  MessageConsumer consumer = session.createConsumer(destination);
  
  System.out.println("B准备接受自动消息.....");
  for(int i = 0; i< 5; i++){
      TextMessage message = (TextMessage)consumer.receive();
      String msg = message.getText();
      System.out.println("Consumer.B.VirtualTopic.vt1:" + msg);
  }
  ```

#### c. 自定义前缀

可以自定义虚拟主题使用的前缀

- name, 指示何种前缀的的 topic 要视为虚拟主题处理, `>` 表示所有的 topic
- prefix, 指示消费者要从什么前缀的队列获取消息, 默认是 `Consumer.*.`, 下面设置为`VitrualTopicConsumers.*.`

```xml
<broker>
	<destinationInterceptors>
    	<virtualDestinationInterceptor>
        	<virtualDestinations>
            	<virtualTopic name=">" prefix="VitrualTopicConsumers.*." selectorAware="false" />
            </virtualDestinations>
        </virtualDestinationInterceptor>
    </destinationInterceptors>
</broker>
```

### 2) 组合 Destination

组合队列允许用一个虚拟的 destination 代表多个 destination, 这样就可以通过 composite destination , 在一个操作中同时向多个 destination 发送消息

#### a. 客户端实现

在创建 destination 时, 多个 destination 之间用 `,` 分割

默认目的地名与创建目的地方法一致, 如 createQueue 创建的是队列, createTopic 创建的是主题

如果需要使用不同的目的地类型, 则要加上相应的前缀 `queue://` 或 `topic://`

> Destination destination1 = session.createQueue("**目的地A,目的地B**");
>
> Destination destination2 = session.createQueue("**对列名,topic://主题名**");
>
> Destination destination3 = session.createTopic("**queue://队列名,主题名**");

- 消息生产者

  ```java
  // 创建会话, 不开启事务, 通过自动模式收发消息
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  Destination destination = session.createQueue("composite.queue,topic://composite.topic");
  
  // 创建生产者
  MessageProducer producer = session.createProducer(destination);
  
  for(int i = 0; i< 5; i++){
      TextMessage message = session.createTextMessage();
      message.setText("这是组合队列的消息"+i);
      producer.send(message);
      System.out.println("已发送消息:"+i);
  }
  
  // 关闭连接
  connection.close();
  ```

- 队列消费者

  ```java
  // 创建会话, 不开启事务, 通过自动模式收发消息, 不需要手动签收
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  Destination destination = session.createQueue("composite.queue");
  
  // 创建消费者
  MessageConsumer consumer = session.createConsumer(destination);
  
  System.out.println("准备接受自动消息.....");
  for(int i = 0; i< 5; i++){
      TextMessage message = (TextMessage)consumer.receive();
      String msg = message.getText();
      System.out.println("组合.queue:" + msg);
  }
  
  // 关闭连接
  connection.close();
  ```

- 主题消费者

  ```java
  // 创建会话, 不开启事务, 通过自动模式收发消息, 不需要手动签收
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  Destination destination = session.createTopic("composite.topic");
  
  // 创建消费者
  MessageConsumer consumer = session.createConsumer(destination);
  
  System.out.println("准备接受自动消息.....");
  for(int i = 0; i< 5; i++){
      TextMessage message = (TextMessage)consumer.receive();
      String msg = message.getText();
      System.out.println("组合.topic:" + msg);
  }
  
  // 关闭连接
  connection.close();
  ```

#### b. activemq.xml 中配置

也可以在 activeMQ 的配置文件中添加一个 destinationInterceptor, 在收到发送给组合目的地时, 转发到真正的目的地中去

其中 `compositeQueue` 也可以为`compositeTopic`, 在客户端使时注意匹配目的地类型

- activemq.xml

  ```xml
  <broker>
  	...
      <destinationInterceptors>
          <virtualDestinationInterceptor>
              <virtualDestinations> 
                  <compositeQueue name="COMPOSITE_QUEUE">
                      <forwardTo>
                          <queue physicalName="realQueue1" />
                          <topic physicalName="realTopic" />
                      </forwardTo>
                  </compositeQueue>
              </virtualDestinations>
          </virtualDestinationInterceptor>
      </destinationInterceptors>
      ...
  </broker>
  ```

- 消息生产者

  ```java
  // 创建会话, 不开启事务, 通过自动模式收发消息
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  Destination destination = session.createTopic("COMPOSITE_TOPIC");
  
  // 创建生产者
  MessageProducer producer = session.createProducer(destination);
  
  for(int i = 0; i< 5; i++){
      TextMessage message = session.createTextMessage();
      message.setText("这是xml组合队列的消息"+i);
      producer.send(message);
      System.out.println("已发送消息:"+i);
  }
  
  // 关闭连接
  connection.close();
  ```

- 队列消费者

  ```java
  // 创建会话, 不开启事务, 通过自动模式收发消息, 不需要手动签收
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  Destination destination = session.createQueue("realQueue1");
  
  // 创建消费者
  MessageConsumer consumer = session.createConsumer(destination);
  
  System.out.println("准备接受自动消息.....");
  for(int i = 0; i< 5; i++){
      TextMessage message = (TextMessage)consumer.receive();
      String msg = message.getText();
      System.out.println("xml.组合.queue:" + msg);
  }
  
  // 关闭连接
  connection.close();
  ```

- 主题消费者

  ```java
  // 创建会话, 不开启事务, 通过自动模式收发消息, 不需要手动签收
  Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
  Destination destination = session.createTopic("realTopic");
  
  // 创建消费者
  MessageConsumer consumer = session.createConsumer(destination);
  
  System.out.println("准备接受自动消息.....");
  for(int i = 0; i< 5; i++){
      TextMessage message = (TextMessage)consumer.receive();
      String msg = message.getText();
      System.out.println("xml.组合.topic:" + msg);
  }
  
  // 关闭连接
  connection.close();
  ```

### 3) FilteredDestination

  在 xml 中配置组合 destination 时, 可以通过选择器, 选择要转发到哪个目的地

  ```xml
<broker>
    ...
    <destinationInterceptors> 
        <virtualDestinationInterceptor>
            <virtualDestinations> 
                <compositeQueue name="MY.QUEUE">
                    <forwardTo>
                        <filteredDestination selector="odd = 'yes'" queue="FOO"/>
                        <filteredDestination selector="i = 5" topic="BAR"/>
                    </forwardTo>
                </compositeQueue>
            </virtualDestinations> 
        </virtualDestinationInterceptor>
    </destinationInterceptors>
    ...
</broker>
  ```

### 4) 避免重复消息

// TODO 什么情况下会有重复消息? 

在使用组合目的地时, 可能会出现重复的消息, 为了避免这种情况,可以使用如下设置

```xml
<networkConnectors> 
    <networkConnector uri="static://(tcp://localhost:61617)">
        <excludedDestinations> 
            <queue physicalName="Consumer.*.VirtualTopic.>"/> 
        </excludedDestinations> 
    </networkConnector>
</networkConnectors>
```

## 3. 其他设置

### 1) 启动时创建 destination

如果需要在 ActiveMQ 启动的时候就创建 destination , 可以在 activemq.xml 中添加如下配置内容

配置了启动destination 后, 在 activemq 启动时就会把 目的地创建好

```xml
<broker xmlns="http://activemq.apache.org/schema/core">
    ...
    <destinations>
        <queue physicalName="FOO.BAR" />
        <topic physicalName="SOME.TOPIC" />
    </destinations>
    ...
</broker>
```

### 2) 删除不活动 destination

一般情况下, broker 默认不会清理不活动的 destination, 可以通过 web 控制台或者是 JMX 方式来删除, 

也可以通过配置, 让 broker 自动检测并删除无用的 destination (一定时间内无客户端访问且为空) , 回收响应资源

- **schedulePeriodForDestinationPurge**: 检查不活动destination 间隔, 多长时间检查一次, 单位毫秒, 默认 0

- **gcInactiveDestinations**: 是否回收不活动destination, 默认 false
- **inactiveTimoutBeforeGC**: 超时时间, 多长时间不活动会被回收, 单位 毫秒, 默认60秒

```xml
<broker xmlns="http://activemq.apache.org/schema/core" schedulePeriodForDestinationPurge="10000">
    ...
    <destinationPolicy>
        <policyMap>
            <policyEntries>
                <policyEntry queue=">" gcInactiveDestinations="true" inactiveTimoutBeforeGC="30000"/>
            </policyEntries>
        </policyMap>
    </destinationPolicy>
    ...
</broker>
```

### 3) destination 选项

destination 选项是 JMS 规范之外添加的功能特性, 可以为 consumer 设置参数

在 destination 名称后面, 使用类似于带参数 URL 的语法添加多个选项

![1526367163088](ActiveMQ%20高级配置.assetsts/1526367163088.png)

```java
String str = "optionQueue?consumer.dispatchAsync=false&consumer.prefetchSize=10";
Destination destination = session.createQueue(str);
MessageProducer producer = session.createProducer(destination);
```

### 4) Mirrored Queues

queue 中的一条消息只能被一个 consumer 消费, 但有时候可能希望能够监视生产者和消费者之间的消息流;

可以通过虚拟立一个 virtual queue 来把消息转发到多个 queue 中, 但如果需要为系统中每个 queue 都进行配置会比较麻烦

ActiveMQ 支持 Mirrored Queues, broker 会把发送到特定 queue 的所有消息转发到一个名称类似的 topic, 因此监控程序只需要订阅这个 mirrored queue topic 

为了启用 mirrored queues , 需要在 activemq.xml 中对 broker 进行配置, 

1. 将 brokerService 的 useMirroredQueues 设为true, 
2. 通过 destinationInterceptors 设置其他属性
3. 设置 mirror topic 的前缀, 默认是 VirtualTopic.Mirror.

```xml
<broker>
	<destinationInterceptors>
    	<mirroredQueue copyMessage="true" postfix=".qmirror" prefix="mirror." />
    </destinationInterceptors>
</broker>
```

### 5) Per Destination Policies

ActiveMQ 支持多种个不同的策略, 可以为每一个不同的 destination 进行设置, 详见官方文档

# 五. Message Dispatch

## 1. Message cursor

从 5.0 开始, ActiveMQ 采用新的存储模型, 可以将消息分页存储到可用的存储空间中.

### 1) Store-based

消息处理的一种典型方式是: 当消费者在线时, 消息发送系统把存储的消息按批次发送给消费者, 发送完一个批次的消息后, 指针的标记位置只想下一批次待发送消息的位置, 进行后续的发送操作

这是一种比较健壮和灵活的消息发送方式, 但在存在快消费者情况下, 这种方式的效率并不高

从 5.0 开始, ActiveMQ 的消息发送系统默认采用一种混合型的发送模式, 能够满足大多数场景的使用要求, 同时支持废止旧消息的处理 

#### a. 持久化消息 + 快消费者

当消费者处于活跃状态且处理能力较强时, 消息持久化后直接发送到与消费者关联的发送队列, 如下图

![img](ActiveMQ%20高级配置.assetsts/DispatchFastConsumers.png) 

#### b. 持久化消息 + 慢消费者

当消息出现积压, 消费者再开始活跃, 或者消费者的消费速度比消息的发送速度慢时, 消息先被持久化, 然后从 pending cursor 中分批提取, 并发送到与消费者关联的发送队列, 如下图

![img](ActiveMQ%20高级配置.assetsts/DispatchSlowConsumers.png) 



#### c. 非持久化消息

在以前的 ActiveMQ 中, 使用非持久消息时经常会发生内存占用过高的问题, 因为消息都是存在内存中, 会占用大量的内存空间.

Store-based 方式内嵌了 File-based 模式, 用来处理非持久的消息.

非持久的消息会直接发送给 非持久游标, 根据消费者消费速度, 可以将消息发送给消费者的发送队列, 或缓存到磁盘临时文件

![img](ActiveMQ%20高级配置.assetsts/NonPersistentMsgs.png) 

### 2) VM

相关的消息引用存储在内存中, 当满足条件时, 消息直接被发送到消费者关联的发送队列, 处理速度非常快

但不适合会出现慢消费者或消费者长时间离线的情况下

![img](ActiveMQ%20高级配置.assetsts/VMCursor.png) 

### 3) File-based

这种方式源自于 vm 模式, 当内存达到设置的限制时, 消息会被存储到磁盘中的临时文件中.

这种模式通常用在消息的持久存储过于缓慢而消费者相对较快的情况下.

通过缓存到磁盘文件, 这种模式允许 broker 处理生产者突发性大量发送的消息, 而不需先经过缓慢的消息持久存储

![img](ActiveMQ%20高级配置.assetsts/FileCursor.png) 

### 4) 配置使用

默认情况下, activemq 会根据使用的 message Store 来决定使用何种类型的 Message cursors, 一般不用去修改

也可以在 `activemq.xml` 中 根据 destination 来配置使用的 message cursor

#### a. 对 topic Subscriber

每个主题的订阅者, 都有一个发送队列和等待游标, 可以为持久订阅者和临时订阅者配置不同的分发策略

```xml
<destinationPolicy>
    <policyMap>
        <policyEntries>
            <policyEntry topic="org.apache.>" producerFlowControl="false" memoryLimit="1mb">
                <dispatchPolicy>
                    <strictOrderDispatchPolicy />
                </dispatchPolicy>
                <deadLetterStrategy>
                    <individualDeadLetterStrategy  topicPrefix="Test.DLQ." />
                </deadLetterStrategy>
                <pendingSubscriberPolicy>
                    <vmCursor />
                </pendingSubscriberPolicy>
                <pendingDurableSubscriberPolicy>
                    <vmDurableCursor/>
                </pendingDurableSubscriberPolicy>
            </policyEntry>
        </policyEntries>
    </policyMap>
</destinationPolicy>
```

有效的 Subscriber types, 即 **pendingSubscriberPolicy**, 默认**store based cursor**, 可选值:

- **vmCursor** 
- **fileCursor**

有效的 Subscriber 的 cursor types , 即 pendingDurableSubscriberPolicy, 默认 **store based cursor** 

- **storeDurableSubscriberCursor**
- **vmDurableCursor**
- **fileDurableSubscriberCursor** 

#### b. 对于 queue

每个队列 destination, 只有一个 发送队列和等待游标, 可以进行以下配置

```xml
<destinationPolicy>
    <policyMap>
        <policyEntries>
            <policyEntry queue="org.apache.>">
                <deadLetterStrategy>
                    <individualDeadLetterStrategy queuePrefix="Test.DLQ."/>
                </deadLetterStrategy>
                <pendingQueuePolicy>
                    <vmQueueCursor />
                </pendingQueuePolicy>
            </policyEntry>
        </policyEntries>
    </policyMap>
</destinationPolicy>
```

有效的队列游标类型, 即 **pendingQueuePolicy**, 默认 **store based cursor** , 可选值:

- **storeCursor**
- **vmQueueCursor**
- **fileQueueCursor**

## 2. Async Sends 异步发送

### 1) 概述

ActiveMQ 支持异步和同发送消息,

通常对于快消费者, 是直接把消息同步发送过去, 但对慢消费者, 使用同步发送消息可能出现 producer 阻塞, 因此慢消费者适合异步发送.

ActiveMQ 大多数情况下都采用异步发送模式, 只有在 JMS 规范中明确要求, 即 **非事务+持久消息** 的情况下会使用同步发送.

在不使用事务且发送持久化消息时, producer 每次发送都是同步发送, 并且会阻塞直到 broker 给 producer 返回一个确认消息, 以确保消息已被正确持久化到存储中, 但这种方式会严重影响 producer 的效率. 

许多高性能的应用被设计成能接受失败时的少量消息丢失, 如果你的应用也是按此要求设计, 使用异步发送将大大提高发送持久消息时的吞吐量.

### 2) 配置使用

ActiveMQ 默认使用的 DispatcheAsync=true 是较好的性能设置, 如果你使用的是快消费者, 也可以设为 false

- 在 connectionURI 配置

  ```java
  cf = new ActiveMQConnectionFactory("tcp://locahost:61616?jms.useAsyncSend=true");
  ```

- 在 connectionFactory 配置

  ```java
  ((ActiveMQConnectionFactory)connectionFactory).setUseAsyncSend(true);
  ```

- 在 connection 配置

  ```java
  ((ActiveMQConnection)connection).setUseAsyncSend(true);
  ```

## 3. Dispatch policies



### 1) topic 的消息分发策略

topic 支持插件式的分发策略, 所有实现了 `org.apache.activemq.broker.region.policy.DispatchPolicy ` 的策略都可以使用

默认是分发策略是 `SimpleDispatchPolicy `, 会将消息分发给所有订阅者, activeMQ 还提供以下的消息分发策略

1. SimpleDispatchPolicy

   默认分发策略, 将消息分发给所有订阅者

2. PriorityDispatchPolicy

   优先级分发策略, 将消息按消费者优先级顺序, 分发消息给每个订阅者

3. PriorityNetworkDispatchPolicy

   网络优先级分发策略, 只将消息分发给优先级最高的订阅者, 忽略其他订阅者

4. RoundRobinDispatchPolicy

   轮询分发策略, 将消息分发给所有订阅者, 但比 SimpleDispatchPolicy 更注重分发消息时的平均性

5. StrictOrderDispatchPolicy

   严格顺序分发策略, 将详细分发给所有订阅者, 并保证所有订阅者收到的消息顺序是一样的

   通常 ActiveMQ 会保证 topic 的 consumer 以相同的顺序接收来自同一个 producer 的消息, 但由于多线程和异步处理, 不同的topic consumer 可能会以不同的顺序接收来自不同 producer 的消息, 如果需要保证不同的 topic  consumer 以相同的顺序来接收来自不同 producer 的消息, 可以使用 Strict order dispatch policy, 代价是性能上的损失

```xml
<destinationPolicy> 
    <policyMap> 
        <policyEntries> 
            <policyEntry topic="FOO.>"> 
                <dispatchPolicy> 
                    <roundRobinDispatchPolicy /> 
                </dispatchPolicy> 
                <subscriptionRecoveryPolicy> 
                    <lastImageSubscriptionRecoveryPolicy /> 
                </subscriptionRecoveryPolicy> 
            </policyEntry> 
            
            <policyEntry topic="ORDERS.>"> 
                <dispatchPolicy>
                    <strictOrderDispatchPolicy />
                </dispatchPolicy> 
                <!-- 1 minutes worth --> 
                <subscriptionRecoveryPolicy> 
                    <timedSubscriptionRecoveryPolicy recoverDuration="60000" /> 
                </subscriptionRecoveryPolicy> 
            </policyEntry> 
            
            <policyEntry topic="PRICES.>"> 
                <!-- lets force old messages to be discarded for slow consumers -->
                <pendingMessageLimitStrategy> 
                    <constantPendingMessageLimitStrategy limit="10"/>
                </pendingMessageLimitStrategy> 
                <!-- 10 seconds worth --> 
                <subscriptionRecoveryPolicy> 
                    <timedSubscriptionRecoveryPolicy recoverDuration="10000" /> 
                </subscriptionRecoveryPolicy> 
            </policyEntry> 
            <policyEntry tempTopic="true" advisoryForConsumed="true" /> 
            <policyEntry tempQueue="true" advisoryForConsumed="true" /> 
        </policyEntries> 
    </policyMap>
</destinationPolicy>
```

### 2) queue 的消息分发策略

插件式的分发策略只适用于 topic, queue 的分发策略要固定得多, 可选项有以下两种:

1. **round robin**

   轮询分发策略, 在此模式下, ActiveMQ 会将消息分配给每一个消费者, 但并不保证绝对的平均分配

2. **Strict order**

   严格顺序模式, 在此模式下, ActiveMQ 会先将消息发往同一个消费者, 直到该消费者的预取缓冲区填满后再给下一个消费者

注意: 由于 ActiveMQ 为了提高性能, 提供了"预取"机制, 且默认的预取大小较大, 因此在两种模式下, 在消费者多, 消息少的情况下, 都有可能只将消息发往某一个消费者, 而其他消费者没有消息. 如果要严格平均分配, 需要将预取缓冲区大小设置为1, 参考[官方文档 - 预取缓冲区](http://activemq.apache.org/what-is-the-prefetch-limit-for.html).

从 5.14.0 开始, 在单个消费者时, `strictOrderDispatch=true ` 会保证消费者消费消息的顺序与消息发送顺序一致

```xml
<policyEntry queue=">" strictOrderDispatch="false" /> 
```

## 4. prefetch size

### 1) 概述

ActiveMQ 的一个设计目标是成为一个高性能的消息总线, 它采用 SEDA ( 分阶段事件驱动架构 ) , 尽可能多地使用异步操作. 为了更有效的利用网络资源, ActiveMQ 使用推送( push )的方式将消息发送给 consumer . 这保证了每个 consumer 本地都有一个待处理消息的缓冲区. 另外一种方式就是 consumer 明确的从 broker 拉取消息, 这种单条消息的拉取方式效率不高, 并会增加处理每条消息的时间.

但是, 如果不限制 broker 向 consumer 推送的消息数量, 那么 consumer 的缓冲空间将很可能被耗尽, 这是当消息生产比消费快时的必然结果. 为了避免这种情况, ActiveMQ 采用了预取数量限制 ( prefetchSize ), 来限制每次发送给某个 consumer 的消息数量.  consumer 可以通过调整 prefetSize 来控制预取缓冲区的大小.

一旦 broker 给 consumer 发送了它 prefetchSize 数量的消息, 就不会再给这个 consumer 发送更多的消息, 直到该 consumer 确认消费完缓冲区中超过 50% 的消息. 当broker 收到确认回执后, 才会再给这个 consumer 发送新的消息, 去填满该 consumer 腾出来的那一半缓冲区.

为了更好的性能, 在有大量消息需要处理时建议使用更大的 prefetSize, 但如果是少量消息且单个消息耗时较长的情况, 可以将 prefetch设为1, 这样可以保证每个 consumer 在某个时刻只有一个消息需要处理, 有利于负载均衡. 而如果将 prefetchSize 设为 0, 则 broker 不会为 consumer 推送消息, 消费者只能从broker 拉取.

### 2) prefetch Policy

不同的使用场景有不同的默认 prefetchSize

- 持久消息队列: 1000
- 非持久消息队列: 1000
- 持久消息主题: 100
- 非持久消息主题: Short.MAX_VALUE-1

可以通过以下方式设置 prefetchSize

- 在 connectionURI 设置所有 prefetch Policy

  ```java
  tcp://localhost:61616?jms.prefetchPolicy.all=50
  ```

- 在 connection URI 设置队列或主题的 prefetchSize

  ```java
  tcp://localhost:61616?jms.prefetchPolicy.queuePrefetch=1 
  ```

- 在 destination Option 设置

  ```java
  queue = new ActiveMQQueue("TEST.QUEUE?consumer.prefetchSize=10");
  consumer = session.createConsumer(queue);
  ```

### 3) 消费者池与预取

如果使用消费者池, 那么消息预取可能会带来一些问题. 已被预取但未被消费的消息, 只会在 consumer 关闭的时候才会被释放以重发给别的 consumer, 但是在消费者池的模式下, 消费者不会被关闭, 直到整个消费者池关闭. 这样预取的消息就不会被消费, 直到该 consumer 重新被激活. 这个特性从性能角度来说是可取, 但可能会导致消息的乱序. 正是因为这个原因, `org.apache.activemq.pool.PooledConnectionFactory` 不会将 consumer 池化.

`Springs CachingConnectionFactory` 支持消费者池( 默认关闭 ). 如果您将 `CachingConnectionFactory` 与 `Spring  DefaultMessageListenerContainer`（DMLC）中配置的多个使用者线程一起使用，那么您要么关闭 `CachingConnectionFactory`中的使用者池（默认关闭），要么在池化 consumer 时使用预取值0. 这样的话, consumer 需要调用 receive(timeout) 来拉取消息. 通常我们建议关闭任何框架中的消费者池.

## 5. Optimize Acknowledgement

[参考文章 - ActiveMQ消息传送机制以及Ack机制](http://shift-alt-ctrl.iteye.com/blog/2020182)

ActiveMQ 支持批量确认消息, 但默认处于关闭状态. 批量确认可以提高性能, 但只在签收模式为 "**AUTO_ACKNOWLEDGE** " 才有效

如果需要在程序中启用, 可以采用如下方式:

- 在 connection URI 上启用 optimized acknowledge

  ```java
  cf = new ActiveMQConnectionFactory("tcp://locahost:61616?jms.optimizeAcknowledge=false");
  ```

- 在 connectionFactory 上启用 

  ```java
  ((ActiveMQConnectionFactory)connectionFactory).setOptimizeAcknowledge(true);
  ```

- 在 connection 上启用

  ```java
  ((ActiveMQConnection)connection).setOptimizeAcknowledge(true);
  ```

从 5.6 开始, 可以使用 setOptimizeAcknowledgeTimeOut 参数来配置确超时时间, 默认300ms, 0表示禁用

注意: AUTO_ACKNOWLEDGE + prefetchSize + optimizedAcknowledge + setOptimizeAcknowledgeTimeOut  的组合设置

由 consumer 自动控制确认时机, 在 optimizedAck 模式下, 通常会在 OptimizedAckTimeOut 时, 或已处理消息达到 prefetchSize*0.65 的时候给 broker 发送一个批量确认的消息, 一次性签收一批消息

## 6. producer flow control

### 1) 概述

生产者流量控制 [官方文档](http://activemq.apache.org/producer-flow-control.html)

当生产者生产消息的速度过快, 超过流量限制的时候, 生产者将会被阻塞, 直到资源可以继续使用, 或者抛出一个JMSException, 可以通过 `<SystemUsage>` 来配置.

同步发送消息的 producer  会自动使用 producer flow control

异步发送消息的 producer, 要使用流量控制, 需要先为 connection 配置一个 producerWindowSize 参数, 限制 producer 在发送消息的过程中个, 收到 broker 对于之前发送个消息的确认之前, 能够发送消息的最大字节数

```java
ActiveMQConnectionFactory connctionFactory = ...
connctionFactory.setProducerWindowSize(1024000);
```

也可以禁用流量控制, 需要在 `activemq.xml` 添加配置

```xml
<destinationPolicy>
  <policyMap>
    <policyEntries>
      <policyEntry topic="FOO.>" producerFlowControl="false"/>
    </policyEntries>
  </policyMap>
</destinationPolicy>
```

注意, 如果生产者的发送消息受到事务控制, 则在发送一批消息时, 可能会出现消息发送一半后被阻塞, 但由于该批次消息没有发完, 事务没有提交, 前面的消息并未真正发送完毕, 导致就是整个批次的消息都被阻塞.

### 2) 非持久队列的流量控制

5.0 引入新的消息游标后, 非持久化的消息被分流到了临沭文件存储中, 以此 来减少飞驰就话消息传送使用的内存总量.

因为游标并不需要占用太多的内存空间, 所以非持久队列的内存限制可能永远都不会达到, 生产者留恋个控制永远不会被触发.

如果需要将所有的非持久化消息存到内存中, 并在达到限制时停止生产者, 需要进行如下配置

```xml
<policyEntry queue=">" producerFlowControl="true" memoryLimit="1mb">    
    <pendingQueuePolicy>
        <vmQueueCursor/>
    </pendingQueuePolicy>
</policyEntry>
```

### 3) 配置客户端的异常

为了应对 broker 空间不足而导致不确定的阻塞生产者的 send() 方法, 可以对 broker 进行配置, 将 `sendFailIfNoSpace=true`,  broker 会在空间不足时引起 send() 方法的失败, 并抛出 `javax.jms.ResourceAllocationException` .

```xml
<systemUsage>
    <systemUsage sendFailIfNoSpace="true">
        <memoryUsage>
            <memoryUsage limit="20 mb"/>
        </memoryUsage>
    </systemUsage>
</systemUsage>
```

这么配置的好处是, 客户端可以不糊遗产个, 稍等一下后重试, 而不是无限期地等下去.

5.3.1 后, 引入了`sendFailIfNoSpaceAfterTimeout`  属性, 会在阻塞一定时间后再引起 send 方法的失败抛出异常, 单位毫秒

```xml
<systemUsage>
    <systemUsage sendFailIfNoSpaceAfterTimeout="3000">
        <memoryUsage>
            <memoryUsage limit="20 mb"/>
        </memoryUsage>
    </systemUsage>
</systemUsage>
```

### 4) System usage

可以通过 `<systemUsage>` 元素的一些属性来减慢生产者, 如下:

```xml
<systemUsage>
    <systemUsage>
        <memoryUsage>
            <memoryUsage limit="64 mb" />
        </memoryUsage>
        <storeUsage>
            <storeUsage limit="100 gb" />
        </storeUsage>
        <tempUsage>
            <tempUsage limit="10 gb" />
        </tempUsage>
    </systemUsage>
</systemUsage>
```

可以为非持久的消息设置内存限制, 为持久化消息设置磁盘空间, 以及为临时消息设置总的空间, broker 将在减慢生产者之前使用这些空间. 使用了上述的默认设置,  broker 将会一直阻塞 send 方法, 直到某些消息被消费, 有了可用的空间才会继续接收生产者的消息

# 六. Message 的高级特性

## 1. 消息属性 property

[官方文档 - 消息的属性说明](http://activemq.apache.org/activemq-message-properties.html)

![1526433962849](ActiveMQ%20高级配置.assetsts/1526433962849.png)

常见属性说明

1. 消息默认是持久化的
2. 默认优先级为 4
3. 消息发送时设置了时间戳
4. 消息的过期时间, 默认永不过期, 过期的消息会进入 DLQ, 可以配置 DLQ 及其处理策略
5. 如果消息是重发的, 会有重发标记
6. JMSReplyTo 标志回执消息要发送到哪个 Queue
7. JMSCorelationID 标志与此消息关联的消息ID, 可以用个这个标识把多个消息连接起来
8. JMS 记录了消息重发的次数, 默认重发6次
9. 如果有一组关联的消息需要处理, 可以分组: 只需要设置消息组的名字和这个消息是第几个消息
10. 如果消息中使用事务, 则会设置一个 TXID
11. ActiveMQ 在服务器端额外设置了消息入列和出列的时间戳
12. ActiveMQ 的消息属性的值, 不仅可以使用基本类型, 还可以使用 LIst 或者 Map

## 2. Advisory Message

### 1) 概述

Advisory Message 是 ActiveMQ 自身的系统消息地址, 可以监听该地址来获取 activeMQ 的系统消息.

目前支持以下的消息

1. consumers producers connections 的启动和停止
2. 创建和销毁 temporary destinations
3. topics 和 queues 的消息过期
4. brokers 可以发送消息给 destinations, 但是没有 consumer
5. connection 的启动和停止

### 2) 开启和关闭

1. 所有的 advisory 的 topic, 前缀是 `ActiveMQ.Advisory`

2. 所有的 Advisory 消息类型都是 `Advisory`, 所有消息都有的是消息属性有 `originBrokerId`, `oribinBrokerName`, `originBrokerURL`

3. 某些 Advisory 功能默认是关闭的, 需要在 `activemq.xml` 中配置打开

   ```xml
   <destinationPolicy>
       <policyMap>
           <policyEntries> 
               <policyEntry topic=">" advisoryForConsumed="true"/>
           </policyEntries>
       </policyMap>
   </destinationPolicy>
   ```

4. 具体支持的 topic 和 queue, 参考 [官方文档 - advisory message](http://activemq.apache.org/advisory-message.html)

Advisory 功能会占用一定的内存和网络资源, 如果不需要这个功能, 可以将其关闭

但应注意, advisory 功能在动态发现网络中是必须的, 如果禁用 advisory , 就必须使用静态网络配置

- 在 broker 配置中关闭

  ```xml
  <broker advisorySupport="false">
  ```

- 在 java 代码中关闭, 针对嵌入式的 broker

  ```java
  BrokerService broker = new BrokerService();
  broker.setAdvisorySupport(false);
  // ...
  broker.start();
  ```

- 在 connectionUri 关闭

  ```java
  tcp://localhost:61616?jms.watchTopicAdvisories=false
  ```

- 在 java 代码中设置 `ActiveMQConnectionFactory` 的 `watchTopicAdvisories`  属性

  ```java
  ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory();
  factory.setWatchTopicAdvisories(false);
  ```

### 3) advisory consumer 

1. 在配置文件中开启要使用的 advisory 功能

2. 生产者就是 broker, 不需要正在客户端设置生产者

3. 消费者根据要接收的信息类型, 来设置不同的 destination, 可以采用以下两种方式:

   - 使用完整的 destination 名订阅, 格式形如 **ActiveMQ.Advisory.Consumer.Topic.主题名**

     ```java
     Destination destination = session.createTopic("ActiveMQ.Advisory.Consumer.Topic.advisoryT");
     ```

   - 使用 AdvisorySupport 获取 advisory destination

     ```java
     Destination des = session.createTopic("advisoryT");
     Destination advisoruDes = AdvisorySupport.getProducerAdvisoryTopic(des);
     ```

4. 得到 advisory Destination 后, 就可以向普通消费者一样获取消息, 返回的消息都封装成 `ActiveMQMessage`, 

   再通过 `getDataStructure()` 方法进一步获取数据

   ```java
   MessageConsumer consumer = session.createConsumer(destination);  
   ActiveMQMessage message = (ActiveMQMessage) consumer.receive();
   DataStructure info =  message.getDataStructure();
   if(info instanceof ConsumerInfo){
       ConsumerInfo cInfo = (ConsumerInfo) info;
       System.out.println(message.getProperty("consumerCount"));
       System.out.println(cInfo.getConsumerId());
   }
   ```

5. 如果使用advisory topic, 则应先时启动advisory订阅者, 再启动要监控的 topic, 否则会丢失消息

## 3. 延迟和定时消息投递

有时候可能会不希望消息马上被 broker 投递出去, 而是想要消息在一定时间后再发给消费者, 或者我们想让消息每隔一定时间投递一次, 一共投递指定的次数...类似这样的需求, ActiveMQ 提供了一种 broker 端的消息定时调度机制

我们只需要把几个描述消息定时调度方式的参数作为属性添加到消息, broker 端的调度器就会按照我们想要的行为去处理消息

### 1) 消息属性

要使用延迟和定时消息投递, 首先需要在 `activemq.xml` 中设置 `schedulerSupport=true`

```xml
schedulerSupport="true"
```

可以使用以下的参数对消息进行配置, 将参数添加到消息的 properties 中即可

| 属性名               | 类型   | 描述                     |
| -------------------- | ------ | ------------------------ |
| AMQ_SCHEDULED_DELAY  | long   | 延迟投递时间             |
| AMQ_SCHEDULED_PERIOD | long   | 重复投递的时间间隔       |
| AMQ_SCHEDULED_REPEAT | int    | 重复投递次数, 不含第一次 |
| AMQ_SCHEDULED_CRON   | String | Cron 表达式              |

ActiveMQ 也提供了一个封装的消息类型 `org.apache.activemq.ScheduledMessage` , 可以使用这个类来辅助设置

使用参数进行配置

```java
// 延迟60秒投递
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("test msg");
long time = 60 * 1000;
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, time);
producer.send(message);

// 先延迟30秒, 然后投递10次, 间隔10秒
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("test msg");
long delay = 30 * 1000;
long period = 10 * 1000;
int repeat = 9;
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, delay);
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, period);
message.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT, repeat);
producer.send(message);
```

### 2) cron 表达式

也可以使用 cron 表达式进行配置

```java
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("test msg");
message.setStringProperty(ScheduledMessage.AMQ_SCHEDULED_CRON, "0 * * * *");
producer.send(message);
```

cron 表达式的优先级要高于另外三个参数, 如果在设置了 cron 的同时, 也有 repeat 和 period 参数, 则会在每次 cron 执行的时候, 重复投递 repeat 次, 每次间隔 period 

也就是说设置地式叠加地效果, 如下所示:

```java
// 每小时都会执行: 消息投递10次, 延迟1秒开始, 每次间隔1秒
MessageProducer producer = session.createProducer(destination);
TextMessage message = session.createTextMessage("test msg");
message.setStringProperty(ScheduledMessage.AMQ_SCHEDULED_CRON, "0 * * * *");
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, 1000);
message.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, 1000);
message.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT, 9);
producer.send(message);
```

## 4.  Blob Message

Blob, Binary Large Objects 消息, 即二进制大对象, 通常是文件.

### 1) 连接URI

// TODO 自定义文件服务器

配置 BLOB Transfer Policy, 可以在发送方的连接 URI 上设置文件服务器的地址,  ActiveMQ 本身也提供了一个这样的文件服务(已取消), 路径为 `IP:port/fileserver` , 完整连接 Uri 如下:

```java
tcp://192.168.1.106:61676?jms.blobTransferPolicy.uploadUri=http://192.168.1.106:8161/fileserver
```

### 2) 生产者

如果要发送的文件或者 URL 存在, 比如发给共享文件系统或者是 web 应用, 可以使用如下方式

```java
BlobMessage message = session.createBlobMessage(new URL("http://some.shared.site.com");
producer.send(message);
```

也可以在客户端动态的创建文件流, 可以将文件上传到文件服务器后再将 url 发送给消费者

```java
// 使用本地文件
BlobMessage message = session.createBlobMessage(new File("/foo/bar");
producer.send(message);

// 使用文件流
InputStream in = ...;
BlobMessage message = session.createBlobMessage(in);
producer.send(message);                       
```

### 3) 消费者

消费者从broker 得到消息后, 如果是 blobMessage, 可以获取到文件的 inputStream 并对文件流进行处理

```java
public void onMessage(Message message) {
    if (message instanceof BlobMessage) {
        BlobMessage blobMessage = (BlobMessage) message;
        InputStream in = blobMessage.getInputStream();
        // process the stream...
    }
}
```

## 5. transformation

有时候需要再 JMS provider 内部进行 message 的转换, 从4.2 开始,  ActiveMQ 提供了一个  MessageTransformer 接口用于进行消息转换, 可在以下对象上调用, 标准 JMS 并没有提供:

1. ActiveMQConnectionFactory
2. ActiveMQConnection
3. ActiveMQSession
4. ActiveMQMessageConsumer
5. ActiveMQMessageProducer

如果是生产者端进行消息转换, 即在消息发送到 broker 之前进行转换, 使用 `producerTransform`

如果是在消费者端进行消息转换, 即在 consumer 接收前进行转换, 使用 `consumerTransform`

MessageTransformer 只是一个接口, 具体实现要用户自己提供

- 生产者

  ```java
  ActiveMQMessageProducer messageProducer = 
      (ActiveMQMessageProducer)session.createProducer(destination);
  
  // 给生产者设置 transformer, 只需实现 producerTransform
  messageProducer.setTransformer(new MessageTransformer(){
  
      @Override
      public Message producerTransform(Session session, MessageProducer messageProducer, Message message) throws JMSException {
          if(message instanceof TextMessage){
              TextMessage textMsg = (TextMessage)message;
              MapMessage mapMsg = session.createMapMessage();
              mapMsg.setString("msg", textMsg.getText());
              System.out.println("生产者: textMessage 转换为 MapMessage 成功");
              return mapMsg;
          }
          return null;
      }
  
      @Override
      public Message consumerTransform(Session session, MessageConsumer messageConsumer, Message message) throws JMSException {
          return null;
      }
  });
  
  // 定义消息对象, 并发送
  TextMessage textMessage = session.createTextMessage();
  textMessage.setText("测试==消息类型转换");
  messageProducer.send(textMessage);
  ```

- 消费者

  ```java
  ActiveMQMessageConsumer consumer = 
      (ActiveMQMessageConsumer)session.createConsumer(destination);
  
  // 给消费者设置 transformer, 只需实现 consumerTransform
  consumer.setTransformer(new MessageTransformer() {
      @Override
      public Message producerTransform(Session session, MessageProducer messageProducer, Message message) throws JMSException {
          return null;
      }
  
      @Override
      public Message consumerTransform(Session session, MessageConsumer messageConsumer, Message message) throws JMSException {
          TextMessage textMsg = session.createTextMessage();
          if(message instanceof MapMessage){
              String mapMsg = ((MapMessage) message).getString("msg");
              textMsg.setText("转换成功:"+mapMsg);
              System.out.println("获取到MapMessage, 转换为TextMessage成功");
          }else {
              textMsg.setText("错误的消息类型");
          }
          return textMsg;
      }
  });
  
  TextMessage message = (TextMessage) consumer.receive();
  System.out.println("获取到消息:" + message.getText());
  ```


# 七. Consumer 高级特性

## 1. Exclusive Consumer

Queue 中的消息是按照发信顺序, 被分发到 consumer 的, 但是如果有多个 consumer 同时从这个 queue 中提取消息, 由于多线程的不确定性, 则无法保证消息是按顺序被消费的.

如果需要保证消息按顺序处理, ActiveMQ 从 4.x 版本起支持 Exclusive Consumer, 即独有消费者. broker 会从多个 consumer 候选人中挑一个, 把所有消息都交给这个 consumer 来处理. 如果这个 consumer 失效, broker 再从别的 consumer 中选取下一个处理者. 

可以通过 destination Option 来指定使用 exclusive Consumer

```java
queue = new ActiveMQQueue("TEST.QUEUE?consumer.exclusive=true");
consumer = session.createConsumer(queue);
```

还可以给 consumer 设置优先级, 一边针对网络情况进行优化

```java
queue = new ActiveMQQueue(queue?consumerexclusive=true&consumer.priority=10);
```

## 2. 消息分组

### 1) 概述

使用 exclusive Consumer 可以保证消息的消费顺序, 但由于只有一个 consumer 处于工作状态, 会对性能造成影响.

另一方面, 通常情况下并不需要对所有消息进行严格的顺序消费, 只需要对具有关联关系的几条消息保持顺序消费即可. 消费分组可以看成是一种并发的 excluseive comsumer ,  JMS 消息属性的 JMSXGroupID 可以用来区分消息所属的组, message group 特性保证所有具有相同 JMSXgroupID 的消息会被分发到相同的 consumer 去处理( 只要这个 consumer 保持活跃)

Message  group 特性也是一种负载均衡的机制, 在一个消息被分发到consumer 之前, broker 会检查消息的 JMSXGroupID 属性, 如果该属性有值, 则 broker 会寻找拥有这个 GroupID 的那个 consumer 来处理这条消息. 如果没有关联的 consumer, broker 会选择一个 consumer, 将 consumer 关联到 group, 以后都由这个 consumer 来处理该 group 的消息, 直到以下几种情况:

- consumer 被关闭
- group 被关闭: 可以发送一个消息, 将消息的 JMSXGroupSeq 设置为 -1

### 2) 创建消息组

创建一个消息组, 只需要在 message 上设置 groupID 即可

```java
Mesasge message = session.createTextMessage("<foo>hey</foo>");
message.setStringProperty("JMSXGroupID", "groupA");
...
producer.send(message);
```

### 3) 关闭消息组

关闭一个消息组, 只需要在最后一个消息上设置一个 `JMSXGroupSeq`的属性为-1

```java
Mesasge message = session.createTextMessage("<foo>hey</foo>");
message.setStringProperty("JMSXGroupID", "IBM_NASDAQ_20/4/05");
message.setIntProperty("JMSXGroupSeq", -1);
...
producer.send(message);
```

## 3. Consumer Dispatch Async

在 ActiveMQ 4.x 后, 可以选择 broker 同步或者异步地把消息分发给消费者, 可以设置 `dispatchAsync` 属性, 默认为`true`.

- 在 connection factory 设置

  ```java
  ((ActiveMQConnectionFactory)connectionFactory).setDispatchAsync(false);
  ```

- 在 connection 上设置

  ```java
  ((ActiveMQConnection)connection).setDispatchAsync(false);
  ```

- 在 consumer 上设置

  ```java
  queue = new ActiveMQQueue("TEST.QUEUE?consumer.dispatchAsync=false");
  consumer = session.createConsumer(queue);
  ```

- 在 broker 上设置全局禁用, 如果禁用,  connection factory, connection,  consumer 都无法单独启用

  ```xml
  <transportConnector name="openwire" uri="tcp://0.0.0.0:61616" disableAsyncDispatch="true"/>
  ```

## 4. Consumer Prioruty

可以为消费者制定优先级, 0-127, 默认为0, 最高优先级为127

```java
queue = new ActiveMQQueue("TEST.QUEUE?consumer.priority=10");
consumer = session.createConsumer(queue);
```

对于 queue, broker 将根据 consumer 的优先级进行消息分发, 先将高优先级 consumer 的 prefetch 装满后, 再给下一优先级的 consumer 分发消息

## 5. 管理持久订阅者

消息的持久化, 保证了消费者离线重连后不会丢失消息, 但这会消耗更多的资源, 5.6 开始, 可以对持久订阅进行管理

对于长时间离线的订阅者, 可以将其移除, 不再为它保留消息订阅, 需要在 `activemq.xml` 中对broker 进行配置

```xml
<broker name="localhost" offlineDurableSubscriberTimeout="86400000" offlineDurableSubscriberTaskSchedule="3600000">
```

| 属性                                 | 默认值 | 描述                                   |
| ------------------------------------ | ------ | -------------------------------------- |
| offlineDurableSubscriberTimeout      | -1     | 离线多久就删除, -1表示不删除, 单位毫秒 |
| offlineDurableSubscriberTaskSchedule | 300000 | 多长时间检查一次, 单位毫秒             |

## 6. 消息选择器

JMS Selectors 用在获取消息的时候, 可以基于消息属性和 XPath 语法对消息进行过滤, JMS Selectors 由 SQL92 语义定义

> consumer = session.createConsumer(destination, **选择器**)

```java
consumer = session.createConsumer(destination, "JMSType = 'car' AND color = 'blue' AND weight > 2500")
```

注意:

1. JMS Selectors 中, 可以使用 in, not in, like 等
2. JMS Selectors 中的日期和时间, 需要使用标准的 long 型毫秒值
3. 表达式中的属性不会自动进行类型转化, 比如 String 类型的 number 属性 "2", 不能用在 "number > 1" 的选择器
4. 消息组可以保证同组的消息被同一个 consumer 消费, 但不保证由哪个消费者消费. 如果需要指定 consumer , 可以在 consumer 端用选择器指定 group, 但这会导致 producer 和 consumer 的耦合, 且若 consumer 失效, 消息将被积压在 broker.

## 7. 消息的重新投递

### 1) 重传策略

ActiveMQ 在接收消息的客户端有以下操作的时候, 需要重新传递消息:

1. 客户端使用了事务, 在 session 中调用了 rollback()
2. 客户端使用了事务, 在commit() 之前关闭了
3. 客户端使用`CLIENT_ACKNOWLEDGE` 手动确认模式, 调用了 recover()
4. 客户端连接超时, (可能是因为消息处理时间比配置的超时时间更长)

可以通过设置 ActiveMQConnectionFactory 和 ActiveMQConnection 来定制要使用的重发策略:

| 属性                     | 默认值   | 描述                                                         |
| ------------------------ | -------- | ------------------------------------------------------------ |
| useExponentialBackOff    | false    | 启用时间间隔的指数倍增, 增加延迟时间                         |
| backOffMultiplier        | 5        | 重连时间间隔递增倍数, 只有值大于1, 启用 useExponentialBackOff 时有效 |
| useCollisionAvoidance    | false    | 启用防止冲突功能                                             |
| collisionAvoidanceFactor | 0.15     | 防止冲突范围的政府百分比, 启用useCollisionAvoidance 时有效   |
| initialRedeliveryDelay   | 1000L    | 初始重发延迟时间                                             |
| maximumRedeliveries      | 6        | 最大重发次数, 达到最大重发次数后抛出异常, 0不重发, -1无限重发 |
| maximumRedeliveryDelay   | -1(不限) | 最大传送延迟, useExponentialBackOff 启用时有效, 递增到最大值后不再增加 |
| redeliveryDelay          | 1000L    | 重发延迟时间, initialRedeliveryDelay=0 时有效                |

可以在`ActiveMQConnectionFactory ` 上修改重传策略

```java
ActiveMQConnectionFactory connectionFactory = new ActiveMQConnectionFactory(username, password, url);

RedeliveryPolicy queuePolicy = new RedeliveryPolicy();
queuePolicy.setMaximumRedeliveries(2);
connectionFactory.setRedeliveryPolicy(queuePolicy);
```

从 5.7 开始, `ActiveMQConnection ` 提供了一个 `RedeliveryPolicyMap` 属性,  可以为每个 destination 配置重传策略, 如下

```java
ActiveMQConnection connection ...  // Create a connection
 
// 为队列创建重传策略
RedeliveryPolicy queuePolicy = new RedeliveryPolicy();
queuePolicy.setInitialRedeliveryDelay(0);
queuePolicy.setRedeliveryDelay(1000);
queuePolicy.setUseExponentialBackOff(false);
queuePolicy.setMaximumRedeliveries(2);
 
// 为主题创建重传策略
RedeliveryPolicy topicPolicy = new RedeliveryPolicy();
topicPolicy.setInitialRedeliveryDelay(0);
topicPolicy.setRedeliveryDelay(1000);
topicPolicy.setUseExponentialBackOff(false);
topicPolicy.setMaximumRedeliveries(3);
 
// Receive a message with the JMS API
RedeliveryPolicyMap map = connection.getRedeliveryPolicyMap();
map.put(new ActiveMQTopic(">"), topicPolicy);
map.put(new ActiveMQQueue(">"), queuePolicy);
```

### 2) DLQ 死信队列

当消息试图重传的额次数超过配置中的最大重传次数时, broker 认为该消息是一个死消息, 并将该消息发往`dead letter queue`,  即 DLQ 死队列中.

ActiveMQ 的默认 DLQ 名为`ActiveMQ.DLQ`, 所有不能消费的消息都被发往该队列, 也可以在 `activemq.xml` 中为不同的 destination 配置各自的 `individualDeadLetterStrategy`

```xml
<broker>
    <destinationPolicy>
        <policyMap>
            <policyEntries>
                <policyEntry queue=">">
                    <deadLetterStrategy>
                        <individualDeadLetterStrategy queuePrefix="DLQ." useQueueForQueueMessages="true"/>
                    </deadLetterStrategy>
                </policyEntry>
            </policyEntries>
        </policyMap>
    </destinationPolicy>
</broker>
```

### 3) 非持久消息的 DLQ

默认情况下, ActiveMQ 不会将投递失败的非持久消息发送到 DLQ, 如果发件方认为该消息没有重要到需要持久化, 那么 ActiveMQ 认为该消息不重要, 故不会将其发往 DLQ 以便后续处理.

如果确实需要将非持久消息发往 DLQ, 可以进行以下配置, 将`processNonPersistent` 设为 true

```xml
<broker>
    <destinationPolicy>
        <policyMap>
            <policyEntries>
                <policyEntry queue=">">
                    <deadLetterStrategy>
                        <sharedDeadLetterStrategy processNonPersistent="true" />
                    </deadLetterStrategy>
                </policyEntry>
            </policyEntries>
        </policyMap>
    </destinationPolicy>
</broker>
```

### 4) 丢弃过期消息

某些情况下, 可能会希望将过期的消息直接丢弃, 而不是发往 DLQ, 这是得客户端只需要处理有问题的消息, 而不需要处理那些过期的消息.

如果需要直接丢弃过期消息, 可以将 `processExpired` 设为 false

```xml
<broker>
    <destinationPolicy>
        <policyMap>
            <policyEntries>
                <policyEntry queue=">">
                    <deadLetterStrategy>
                        <sharedDeadLetterStrategy processExpired="false" />
                    </deadLetterStrategy>
                </policyEntry>
            </policyEntries>
        </policyMap>
    </destinationPolicy>
</broker>
```

## 8. 慢消费者处理

非持久订阅的消费者, prefetch 会设置得比较大, broker 收到消息后直接发送给 consumer.

如果这个 consumer 消费速度慢, 则 broker 中积压的消息会越来越多. broker 会通知 producer 降低生产速度, 但这样会影响快消费者的效率; 另一个方法是将消息缓存到磁盘, 但这同样会减慢快消费者的效率.

现在 ActiveMQ 可以让用户配置这种情况下 broker 要保留的最大消息数量, 当超过其限制时, activeMQ 默认会丢弃旧消息, 以便能够继续接收新消息. 

### 1) 配置预取空间大小

通过在配置文件中的 destination map 中配置 **PendingMessageLimitStrategy** , 可以为不同的 topic 配置不同的策略

1. **ConstantPendingMessageLimitStrategy** 

   通过常量控制额外增加的预存数量

2. **PrefetchRatePendingMessageLimitStrategy** 

   通过比例来增加预存消息数量

以上两种方式, 0表示不增加预存大小, >1是表示增加大小, -1表示不增加预存空间大小, 但也禁止丢弃旧消息

```xml
<policyEntry>
    <pendingMessageLimitStrategy>
        <constantPendingMessageLimitStrategy limit="1000"/>
    </pendingMessageLimitStrategy>
</policyEntry>
```

### 2) 配置丢弃策略

可以选择丢弃消息的策略, 有以下三种方式

1. oldestMessageEvictionStrategy 

   丢弃最旧的消息, 默认

   ```xml
   <oldestMessageEvictionStrategy/> 
   ```

2. oldestMessageWithLowestPriorityEvictionStrategy 

   丢弃最旧, 且优先级最低的消息

   ```xml
   <oldestMessageWithLowestPriorityEvictionStrategy/> 
   ```

3. uniquePropertyMessageEvictionStrategy  

   根据选择器丢弃消息

   ```xml
   <uniquePropertyMessageEvictionStrategy propertyName="STOCK"/> 
   ```

# 八. ActiveMQ 优化和使用建议

## 1. 典型应用场景

异步调用

一对多通讯 topic

多个系统的集成, 同构/异构

作为 RPC/RMI 的替代

多个应用相互解耦

作为事件驱动架构的幕后支撑

提高系统的可伸缩性

## 2. 优化

### 1) 影响因素

1. 网络拓扑结构, 嵌入, 主从复制, 网络连接
2. transport 协议
3. service 的质量: topic or queue, 是否持久化, 是否重新投递, 消息超时等
4. 硬件, 网络, JVM, 操作系统等因素
5. 生产者, 消费者的数量
6. 消息分发要经过的 destination 数量, 消息大小等

### 2) 优化建议

- 非持久消息更快

  非持久化发送消息是异步的, producer 不需要等待 broker 的receipt 消息

  持久化需要先将消息存储起来, 然后再给 consumer 投递

- 尽量使用异步投递消息

- 事务处理比非事务处理更快, 因为是批量操作

- 考虑使用内嵌 broker, 应用与 broker 之间使用 VM 通讯, 速度更快

- 尽量使用基于文件的消息存储方案, 比如 kahaDB

- 调整 prefetch size

- 生产者流量控制

- 关闭消息的复制功能

- 调整 TCP 协议

  socketBufferSize, socket 缓存大小, 

  tcpNoDelay: 默认是 false

  ```java
  String url = "failover://(tcp://localhost:61616?tcpNoDelay=true)";
  ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory(url);
  ```

- 消息投递和消息确认

  官方建议使用自动确认的模式, 并开启优化确认的选项

- 通常 session 会在单独的线程将消息分发给消费者, 如果使用自动确认模式, 可以通过 session 直接分发消息

  ```java
  cf.setAlwaysSessionAsync(false);
  ```

- 如果使用 kahaDB, 进行一下优化

# 九. ActiveMQ 应用开发

使用场景:

1. 作为中间件使用, 多系统应用, 作为消息总线, 个系统解耦
2. 嵌入式 broker, 
3. 嵌入式 broker + vm 通讯: 单系统应用
4. 

