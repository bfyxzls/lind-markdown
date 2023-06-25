Nacos：一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

# 导入配和共享配置
## 导入配置
```yaml
spring:
  config:
    import:
    - nacos:application-dev.yml # 这个功能类型于config-server中的思想，将共享配置写到统一的application.yml里
```
## 共享配置
```yaml
spring:
  cloud:
    nacos:
      username: ${nacos.username}
      password: ${nacos.password}
      config:
        server-addr: ${nacos.server-addr}
        namespace: ${nacos.namespace}
        # 用于共享的配置文件
        shared-configs:
          - data-id: common-mysql.yaml
            group: SPRING_CLOUD_EXAMPLE_GROUP
          - data-id: common-redis.yaml
            group: SPRING_CLOUD_EXAMPLE_GROUP
          - data-id: common-base.yaml
            group: SPRING_CLOUD_EXAMPLE_GROUP
```
## 扩展配置(extension-configs)
```yaml
spring:
  cloud:
    nacos:
      config:
        shared-configs:
         - data-id: mysql.yaml
           refresh: true
        extension-configs:
         - data-id: mysql.yaml
           group: ddd-demo
           refresh: true
```
## 优先级
下面是正确的优先级排序：

1. `import`优先级最高：当导入配置中同时存在`import`和其他配置属性（如`shared-configs`和`extension-configs`）时，`import`的优先级最高。导入配置中的项将覆盖其他属性中相同的配置项。

2. `extension-configs`次之：如果导入配置中不存在某个配置项，Nacos将首先查找`extension-configs`中的对应项。`extension-configs`用于扩展Nacos的配置处理能力，它可以提供更高级的配置定制和处理。

3. `shared-config`优先级最低：如果导入配置和`extension-configs`中都不存在某个配置项，Nacos将尝试查找`shared-configs`中的对应项。`shared-configs`是一组共享配置，可以被多个应用共同引用和使用。

因此，正确的优先级排序应为：`import` > `extension-configs` > `shared-configs`。

# 网络分区
网络分区是指在一个网络中，部分节点之间的连接中断或无法通信，形成了相互隔离的子网络。网络分区可能是由于网络故障、硬件故障、拓扑变化或其他因素引起的。

网络分区可能对分布式系统和应用产生重要影响，主要有以下几个方面：

1. 可用性影响：当网络分区发生时，被分区的节点无法与其他节点进行通信，可能导致这些节点无法提供服务，从而影响整个系统的可用性。

2. 数据一致性问题：当网络分区发生时，分区内的节点之间可以相互通信，但与其他分区的节点无法进行通信。这可能导致数据在不同分区之间的不一致性，因为在分区内的更新可能无法传播到其他分区。

3. 容错性挑战：网络分区对系统的容错能力提出了挑战。分布式系统需要设计相应的容错机制，例如使用副本和冗余节点来保证服务的可用性和数据的一致性。

4. 分区恢复：一旦网络分区被解决，系统需要进行分区恢复以恢复整个系统的正常运行。这可能涉及到数据的同步、重新连接节点和重新平衡负载等操作。

为了应对网络分区问题，分布式系统通常采用一些技术和策略，如使用一致性哈希算法、复制和分片、故障切换、心跳机制等，以确保系统在网络分区期间仍能提供服务并保持数据一致性。

总结而言，网络分区是指网络中部分节点之间的连接中断或无法通信，它对分布式系统的可用性、数据一致性和容错能力都带来重要影响，需要系统设计和容错机制来应对和解决。

# AP还是CP
Nacos是一个分布式的配置中心和服务发现系统，它既支持AP（可用性和分区容忍性）模型，也支持CP（一致性和分区容忍性）模型，具体取决于您的部署方式和配置。

在Nacos中，有两种主要的部署模式：

1. 单机模式（Standalone Mode）：在单机模式下，Nacos会以AP模型运行。这意味着它在面对网络分区时会保持可用性，但在某些情况下，可能会出现数据的不一致性。

2. 集群模式（Cluster Mode）：在集群模式下，您可以使用Nacos的集群部署功能来实现高可用性和一致性。当以集群模式运行时，您可以选择将Nacos配置为AP或CP模型。

对于AP模型，Nacos将优先保持可用性，即使在网络分区的情况下也可以继续提供服务。这可能导致在分区恢复后，数据的一致性需要一定的时间来同步。

对于CP模型，Nacos将优先保证数据的一致性，即使在面临网络分区时也会牺牲可用性。这意味着在网络分区期间，部分节点可能会暂时失去可用性，直到网络分区解决并达到一致性。

选择AP模型还是CP模型取决于您的应用需求和对可用性与一致性的重要性权衡。如果您的应用更关注可用性和分区容忍性，可以选择AP模型。如果您的应用更关注数据的一致性和完整性，可以选择CP模型。

请注意，无论是AP模型还是CP模型，Nacos都提供了一系列的配置和调优选项，以便根据具体需求进行自定义配置。具体的配置和部署方式可以参考Nacos的官方文档以获取更详细的信息。
