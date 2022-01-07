![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1628174249461-a0e76cc8-7ef8-45fc-8c57-012b24a6b932.png)

## Services 和 Pods

Kubernetes[Pods](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/concepts/workloads/pods/pod-overview/)是有生命周期的。他们可以被创建，而且销毁不会再启动。 如果您使用[Deployment](https://link.zhihu.com/?target=https%3A//kubernetes.io/docs/concepts/workloads/controllers/deployment/)来运行您的应用程序，则它可以动态创建和销毁 Pod。

一个Kubernetes的`Service`是一种抽象，它定义了一组`Pods`的逻辑集合和一个用于访问它们的策略 - 有的时候被称之为微服务。一个`Service`的目标`Pod`集合通常是由`Label Selector` 来决定的（下面有讲一个没有选择器的`Service` 有什么用处）。

举个例子，想象一个处理图片的后端运行了三个副本。这些副本都是可以替代的 - 前端不关心它们使用的是哪一个后端。尽管实际组成后端集合的`Pod`可能会变化，前端的客户端却不需要知道这个变化，也不需要自己有一个列表来记录这些后端服务。`Service`抽象能让你达到这种解耦。

不像 `Pod` 的 IP 地址，它实际路由到一个固定的目的地，`Service` 的 IP 实际上不能通过单个主机来进行应答。 相反，我们使用 `iptables`（Linux 中的数据包处理逻辑）来定义一个虚拟IP地址（VIP），它可以根据需要透明地进行重定向。 当客户端连接到 VIP 时，它们的流量会自动地传输到一个合适的 Endpoint。 环境变量和 DNS，实际上会根据 `Service` 的 VIP 和端口来进行填充。

kube-proxy支持三种代理模式: 用户空间，iptables和IPVS；它们各自的操作略有不同。

### Userspace

作为一个例子，考虑前面提到的图片处理应用程序。 当创建 backend `Service` 时，Kubernetes master 会给它指派一个虚拟 IP 地址，比如 10.0.0.1。 假设 `Service` 的端口是 1234，该 `Service` 会被集群中所有的 `kube-proxy` 实例观察到。 当代理看到一个新的 `Service`， 它会打开一个新的端口，建立一个从该 VIP 重定向到新端口的 iptables，并开始接收请求连接。

当一个客户端连接到一个 VIP，iptables 规则开始起作用，它会重定向该数据包到 `Service代理` 的端口。 `Service代理` 选择一个 backend，并将客户端的流量代理到 backend 上。

这意味着 `Service` 的所有者能够选择任何他们想使用的端口，而不存在冲突的风险。 客户端可以简单地连接到一个 IP 和端口，而不需要知道实际访问了哪些 `Pod`。

### iptables

再次考虑前面提到的图片处理应用程序。 当创建 backend `Service` 时，Kubernetes 控制面板会给它指派一个虚拟 IP 地址，比如 10.0.0.1。 假设 `Service` 的端口是 1234，该 `Service` 会被集群中所有的 `kube-proxy` 实例观察到。 当代理看到一个新的 `Service`， 它会配置一系列的 iptables 规则，从 VIP 重定向到 per-`Service` 规则。 该 per-`Service` 规则连接到 per-`Endpoint` 规则，该 per-`Endpoint` 规则会重定向（目标 NAT）到 backend。

当一个客户端连接到一个  VIP，iptables 规则开始起作用。一个 backend 会被选择（或者根据会话亲和性，或者随机），数据包被重定向到这个  backend。 不像 userspace 代理，数据包从来不拷贝到用户空间，kube-proxy 不是必须为该 VIP 工作而运行，并且客户端  IP 是不可更改的。 当流量打到 Node 的端口上，或通过负载均衡器，会执行相同的基本流程，但是在那些案例中客户端 IP 是可以更改的。

### IPVS

在大规模集群（例如10,000个服务）中，iptables  操作会显着降低速度。 IPVS 专为负载平衡而设计，并基于内核内哈希表。 因此，您可以通过基于 IPVS 的 kube-proxy  在大量服务中实现性能一致性。 同时，基于 IPVS 的 kube-proxy 具有更复杂的负载平衡算法（最小连接，局部性，加权，持久性）。

下面我们详细说下k8s支持的4种类型的Service。

### ClusterIP

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/10386414/1628174249472-31a5325b-ac6a-4734-a963-7db65318841e.jpeg)

创建ClusterIP的Service yaml如下：

```plain
apiVersion: v1
kind: Service
metadata:
  name: service-python
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 443
  selector:
    run: pod-python
  type: ClusterIP
```

使用 `kuebctl get svc` :

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1628174249479-cbb3055b-6bc5-48f9-8aa9-b857ad9405da.png)

类型为ClusterIP的service，这个service有一个Cluster-IP，其实就一个VIP。具体实现原理依靠kubeproxy组件，通过iptables或是ipvs实现。

这种类型的service 只能在集群内访问。

### NodePort

我们的场景不全是集群内访问，也需要集群外业务访问。那么ClusterIP就满足不了了。NodePort当然是其中的一种实现方案。

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/10386414/1628174249487-eb66bb38-984c-4d47-9a5a-050700297aec.jpeg)

创建NodePort 类型service 如下：

```plain
apiVersion: v1
kind: Service
metadata:
  name: service-python
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 443
    nodePort: 30080
  selector:
    run: pod-python
  type: NodePort
```

使用 `kuebctl get svc` :

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1628174249488-405a31d7-cf38-4f5e-b52e-96f62cb5b938.png)

此时我们可以通过`http://4.4.4.1:30080或http://4.4.4.2:30080` 对pod-python访问。该端口有一定的范围，比如默认Kubernetes 控制平面将在`--service-node-port-range`标志指定的范围内分配端口（默认值：30000-32767）。

## LoadBalancer

LoadBalancer类型的service 是可以实现集群外部访问服务的另外一种解决方案。不过并不是所有的k8s集群都会支持，大多是在公有云托管集群中会支持该类型。负载均衡器是异步创建的，关于被提供的负载均衡器的信息将会通过`Service`的`status.loadBalancer`字段被发布出去。

创建 LoadBalancer service 的yaml 如下：

```plain
apiVersion: v1
kind: Service
metadata:
  name: service-python
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 443
    nodePort: 30080
  selector:
    run: pod-python
  type: LoadBalancer
```

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/10386414/1628174249488-55dad710-49e7-4b16-bcad-9a796735cf07.jpeg)

使用 `kuebctl get svc` :

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1628174249486-cda8e1f9-ef7a-41e0-acbc-23129b9e406d.png)

可以看到external-ip。我们就可以通过该ip来访问了。

当然各家公有云支持诸多的其他设置。大多是公有云负载均衡器的设置参数，都可以通过svc的注解来设置，例如下面的aws：

```plain
metadata:
      name: my-service
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-access-log-enabled: "true"
        # Specifies whether access logs are enabled for the load balancer
        service.beta.kubernetes.io/aws-load-balancer-access-log-emit-interval: "60"
        # The interval for publishing the access logs. You can specify an interval of either 5 or 60 (minutes).
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name: "my-bucket"
        # The name of the Amazon S3 bucket where the access logs are stored
        service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix: "my-bucket-prefix/prod"
        # The logical hierarchy you created for your Amazon S3 bucket, for example `my-bucket-prefix/prod`
```

## ExternalName

类型为 ExternalName 的service将服务映射到 DNS 名称，而不是典型的选择器，例如`my-service`或者`cassandra`。 您可以使用`spec.externalName`参数指定这些服务。

创建 ExternalName 类型的服务的 yaml 如下：

```plain
kind: Service
apiVersion: v1
metadata:
  name: service-python
spec:
  ports:
  - port: 3000
    protocol: TCP
    targetPort: 443
  type: ExternalName
  externalName: remote.server.url.com
```

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/10386414/1628174249497-57ef9a06-734c-483b-941c-9e04c9bbe6c1.jpeg)

**说明：**您需要 CoreDNS 1.7 或更高版本才能使用`ExternalName`

类型。

当查找主机 `service-python.default.svc.cluster.local`时，集群DNS服务返回CNAME记录，其值为`my.database.example.com`。 访问`service-python`的方式与其他服务的方式相同，但主要区别在于重定向发生在 DNS 级别，而不是通过代理或转发。 

将生产工作负载迁移到Kubernetes集群并不容易。大多数我们不可以停止所有服务并在Kubernetes集群上启动它们。有时，尝试迁移轻量且不会破坏你服务的服务是很好的。在此过程中，一个可能不错的解决方案是使用现有的有状态服务（例如DB），并首先从无状态容器开始。

从Pod中访问外部服务的最简单正确的方法是创建`ExternalName` service。例如，如果您决定保留AWS RDS，但您还希望能够将MySQL容器用于测试环境。让我们看一下这个例子：

```plain
kind: Service
apiVersion: v1
metadata:
  name: test-service
  namespace: default
spec:
  type: ExternalName
  externalName: test.database.example.com
```

你已将Web应用程序配置为使用URL测试服务访问数据库，但是在生产集群上，数据库位于AWS RDS上，并且具有以下URL `test.database.example.com`。创建ExternalName service 并且你的Web Pod尝试访问test-service上的数据库之后，Kubernetes DNS服务器将返回值为`test.database.example.com`的CNAME记录。问题解决了。



ExternalName service 也可以用于从其他名称空间访问服务。例如：

```plain
kind: Service
apiVersion: v1
metadata:
  name: test-service-1
  namespace: namespace-a
spec:
  type: ExternalName
  externalName: test-service-2.namespace-b.svc.cluster.local
  ports:
  - port: 80
```

在这里，我可以使用名称空间a中定义的`test-service-1`访问命名空间b中的服务

`test-service-2`。

这个意义在哪里？

ExternalName service  也是一种service，那么ingress controller 会支持，那么就可以实现跨namespace的ingress。