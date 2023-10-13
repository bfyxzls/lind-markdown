* https://blog.51cto.com/wangguishe/5789239
* https://blog.csdn.net/qq_29974229/article/details/127190476
* https://blog.51cto.com/wangguishe/5789239
*
在 Kubernetes 中使用 Envoy 作为负载均衡器通常涉及以下步骤：

1. **创建 Envoy 部署：** 首先，你需要在 Kubernetes 中创建一个 Envoy 的 Deployment，这将运行 Envoy 代理的实例。你可以使用 YAML 文件定义 Deployment，确保在容器规范中将 Envoy 作为容器运行。

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: envoy-deployment
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: envoy
     template:
       metadata:
         labels:
           app: envoy
       spec:
         containers:
             - name: envoy
               image: envoyproxy/envoy
               ports:
                - name: http
                  containerPort: 80 #envoy路由的端口
                - name: envoy-admin
                  containerPort: 9901 #envoy后台管理系统端口
               volumeMounts:
                - name: envoy-config-101
                  mountPath: "/etc/envoy"
                  readOnly: true
         volumes:
          - name: envoy-config-101
            configMap:
              name: envoy-config-101
              
   ```

2. **创建 Envoy 服务：** 为 Envoy 创建一个 Kubernetes Service，以便其他应用程序可以通过该 Service 连接到 Envoy 负载均衡器。这通常是一个 ClusterIP Service。

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: envoy-service
   spec:
     selector:
       app: envoy
     type: NodePort
     ports:
     - protocol: TCP
       port: 9901 
       name: http
     - protocol: TCP
       port: 80 
       name: admin
   ```

3. **配置 Envoy：** 你需要提供 Envoy 配置文件，以定义路由、后端服务和其他负载均衡规则。通常，你会使用 ConfigMap 来存储 Envoy 配置。

4. **Sidecar 模式：** 另一种常见的做法是将 `Envoy 作为应用容器的 Sidecar 容器部署`，这使得每个应用容器都有一个附加的 Envoy 容器，用于处理负载均衡。

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
   spec:
     containers:
     - name: my-app
       image: my-app-image
       ports:
       - containerPort: 8080
     - name: envoy
       image: envoyproxy/envoy
       ports:
       - containerPort: 80
   ```

5. **配置服务发现：** 为了让 Envoy 知道要负载均衡的后端服务，你需要配置服务发现。这通常包括使用 Kubernetes 的服务名和端口。
* envoy.yaml代码如下
```
admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 } #envoy后台系统的端口

static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 80 } #envoy路由的端口
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        config:
          stat_prefix: ingress_http
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: {cluster: myapp_cluster, timeout: 60s }
          http_filters:
          - name: envoy.router
  clusters:
  - name: myapp_cluster
    connect_timeout: 0.25s
    type: STRICT_DNS
    dns_lookup_family: V4_ONLY
    lb_policy: LEAST_REQUEST
    hosts: #具体应用myapp的地址和端口
    - socket_address:
        address: 192.168.60.136
        port_value: 30021
    - socket_address:
        address: 192.168.60.146
        port_value: 30022
    # 健康检查配置，以下配置之后，会把不在httpstatus不在200~399之间的服务离线，不让它请求流量
    health_checks:
      - timeout: 1s
        interval: 10s
        unhealthy_threshold: 3
        healthy_threshold: 2
        http_health_check:
          path: "/"
          expected_statuses: 
            start: 200
            end: 399


```
* kubectl create configmap envoy-config-101 --from-file=envoy.yaml

6. **监控和健康检查：** 可以配置 Envoy 以执行健康检查，并监控后端服务的可用性。这有助于确保只有健康的实例接收流量。

7. **部署应用：** 最后，将你的应用部署到 Kubernetes 集群，并确保它们使用 Envoy 服务进行负载均衡。

这是一个高级概述，实际配置和部署取决于你的具体需求和环境。你可能需要创建适合你应用程序的 Envoy 配置文件，定义路由规则，配置健康检查等。同时，Envoy 提供了丰富的文档，可以帮助你深入了解如何在 Kubernetes 中使用它作为负载均衡器。

# envoy健康检查配置
Envoy可以使用健康检查来确定服务实例的可用性，如果服务实例被标记为不健康，Envoy将停止将流量路由到该实例。以下是如何配置Envoy来执行健康检查的一般步骤：

1. **定义健康检查配置**:
   在Envoy的配置文件中，你需要定义健康检查的配置。通常，这是通过`cluster`配置完成的，其中包括`health_checks`字段。

```yaml
static_resources:
  clusters:
    - name: my_service
      connect_timeout: 0.25s
      type: STATIC
      hosts:
        - socket_address:
            address: 192.168.60.136
            port_value: 30021
        - socket_address:
            address: 192.168.60.136
            port_value: 8080
      health_checks:
          - timeout: 1s
            interval: 10s
            unhealthy_threshold: 3
            healthy_threshold: 2
            http_health_check:
              path: "/"
              expected_statuses: 
                start: 200
                end: 399
```

在上述示例中，我们为名为`my_service`的集群定义了健康检查。它将定期发送HTTP请求检查每个服务实例的健康状况。

2. **指定健康检查协议**:
   你可以根据需要选择适当的健康检查协议，如HTTP、TCP或gRPC。在上述示例中，我们使用了HTTP健康检查。

3. **定义健康检查路径和间隔**:
   在健康检查配置中，你需要指定要发送的健康检查请求路径和检查的时间间隔。这些参数可以根据你的需求进行配置。

4. **处理失败**:
   如果健康检查失败，Envoy将标记服务实例为不健康，并停止将流量路由到该实例。你可以配置失败的阈值、重试次数等参数。

5. **监控和日志**:
   你可以使用监控和日志工具来监视健康检查的结果，以便了解服务实例的状态和健康情况。

通过这些配置，Envoy将根据健康检查的结果自动管理流量路由，确保不健康的服务实例不再接收流量，从而提高系统的稳定性和可用性。

#  hosts和load_assignment的使用场景
在Envoy的配置中，hosts 和 load_assignment 是两个不同的部分，用于定义集群的成员和它们的负载分配。

## hosts
hosts 部分用于直接指定集群的成员（后端服务的 IP 地址和端口）。
你需要显式地列出每个后端服务的 IP 地址和端口。
这是一种静态定义方式，通常用于管理一组已知的后端服务，其中的成员不会经常变化。

* 示例：

```
hosts:
  - socket_address:
      address: 192.168.1.1
      port_value: 8080
  - socket_address:
      address: 192.168.1.2
      port_value: 8080
```
## load_assignment

load_assignment 部分用于更动态地定义集群的成员，通常与服务发现系统集成，如Consul、etcd、ZooKeeper等。
它不需要显式列出每个后端服务，而是通过服务发现来动态获取成员信息。
这是一种灵活的方式，适用于环境中服务实例的动态变化，因为它能够自动感知新的服务实例并将它们添加到集群。

* 示例：
```
    load_assignment:
      cluster_name: my_cluster
      endpoints:
        - lb_endpoints:
            - endpoint:
                address:
                  socket_address:
                    address: my-service.example.com
                    port_value: 8080
            - endpoint:
                address:
                  socket_address:
                    address: my-service2.example.com
                    port_value: 8080
```
总之，hosts 是一种手动定义后端服务的静态方式，而 load_assignment 则适用于与服务发现系统集成，以便更动态地管理集群成员。选择哪种方式取决于你的环境和需求。

# cluster.type配置项
在 Envoy 中，`clusters` 是用于定义与后端服务通信的配置部分。`clusters` 可以采用不同的类型，包括 `STRICT_DNS` 和 `static`：

1. **STRICT_DNS**:
   - `STRICT_DNS` 是一种 Cluster 类型，它允许 Envoy 根据 DNS 查询来动态地解析后端服务的主机名。
   - 这意味着你可以使用主机名来配置后端服务，而不需要为每个后端服务器指定静态 IP 地址或端口。
   - Envoy将根据DNS查询的结果动态更新后端服务器的地址。

2. **static**:
   - `static` 是另一种 Cluster 类型，它允许你明确地定义后端服务的 IP 地址和端口。
   - 这意味着你需要手动指定后端服务器的详细信息，包括 IP 地址和端口。
   - 这种配置适用于静态集群，其中后端服务器的地址不会经常更改。

选择使用哪种 Cluster 类型取决于你的应用需求。如果你的后端服务的 IP 地址可能会发生变化，或者你想要动态地扩展或缩减后端服务，那么`STRICT_DNS` 是一个不错的选择。而如果你的后端服务是静态的，并且你愿意手动配置它们的 IP 地址和端口，那么 `static` 可能更适合。

总之，Cluster 配置允许你有效地管理 Envoy 与后端服务之间的通信，无论是通过 DNS 动态解析还是通过静态配置。

# clusters.lb_policy配置项
在 Envoy 的 Cluster 配置中，`lb_policy` 是负载均衡策略的设置，用于决定如何分发请求给后端服务的成员。以下是一些可能的 `lb_policy` 选项以及它们的含义：

1. **round_robin**（默认值）：
   - 这是最常见的负载均衡策略。
   - 它按照顺序逐个将请求分发给后端服务的成员，确保每个成员都接收到大致相等数量的请求。

2. **least_request**：
   - 这个策略会将请求分发给当前具有最少请求的后端服务成员。
   - 这对于负载不均衡的情况非常有用，可以确保将请求发送到负载较低的服务器。

3. **ring_hash**：
   - 这个策略使用哈希函数来将请求映射到后端服务器。
   - 这可以确保特定的请求将一致地路由到相同的后端服务器，适用于会话保持等情况。

4. **random**：
   - 这个策略会随机地选择后端服务器来处理请求。
   - 这种策略在某些特定的场景下可能有用，但通常不太常见。

5. **original_dst**：
   - 这个策略基于请求中的原始目标地址来路由请求。
   - 它适用于代理透明性或 L4/L3 代理的情况，通常在 TCP/UDP 代理中使用。

选择合适的负载均衡策略取决于你的应用需求和后端服务的特性。例如，如果后端服务的负载不均衡，你可以选择`least_request`。如果需要会话保持，可以使用`ring_hash`。通常情况下，`round_robin` 是一种简单且有效的策略，适用于多数情况。

注意，具体的负载均衡策略的可用性可能取决于 Envoy 版本，所以确保查阅与你所使用的 Envoy 版本相关的文档以获取详细信息。
