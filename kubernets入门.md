# Kubernetes 入门

## 1. Kubernetes 是什么?

1. 一个全新的基于容器技术的分布式架构领先方案

2. 避免了着重于负载均衡器的选型和部署实施问题, 避免开发复杂的服务治理框架, 服务监控和故障处理模块的开发, 使得开发人员专注于业务
由于kubernetes 提供的强大的自动化机制, 降低后期的运维难度和成本

3. 开发平台: 不限制语言, 各种服务都可以被映射为 Kubernetes 的 Service 服务, 并通过 TCP 通信协议进行交互

4. 一个完备的分布式系统支撑平台

```分布式系统支撑平台
Kubernetes 具备完备的集群管理能力:
1. 多层次的安全防护和准入机制,

2. 多租户应用支撑能力.

3. 透明的服务注册和服务发现机制.

4. 智能负载均衡器

5. 强大的故障发现和自我修复能力

6. 滚动升级和在线扩容能力(灰度发布, 金丝雀发布, 蓝绿部署)

7. 可扩展的资源自动调度机制

8. 多粒度的资源配额管理能力

```

## 2. 使用 Kubernetes 的好处

1. 可以 "轻装上阵" 地开发复杂系统

2. 可以全面拥抱微服务

3. 可以随时随地的将系统整体上云

4. 在 Kubernetes 的架构方案中完全屏蔽了底层网络的细节, 基于 Service 的虚拟IP地址(Cluster IP)的设计思路让架构与底层的硬件拓扑无关

5. Kubernetes 内在的服务弹性扩容机制可以快速,轻松扩容

## 3. Kubernetes 相关概念介绍

![avatar](source\kubernetes.png)

1. Master

>> Master: 集群控制节点, 负责整个集群的管理和控制.

1.1 Kubernetes API Server (kube-apiserver)

```kube-apiserver
提供了HTTP Rest接口的关键服务进程, 是Kubernetes里所有资源的增,删,改,查等操作的唯一入口, 也是集群控制的入口进程
```

1.2 Kuberbetes Controller Manager (kube-controller-manager)

```kube-controller-manager
Kubernetes里所有资源对象的自动化控制中心
```

2.3 Kubernetes Scheduler (kube-scheduler)

```kube-scheduler
负责资源调度(Pod调度)的进程
```

2.4 ectd

```ectd
Kubernetes 里所有的资源对象的数据都被保存在 ectd 中.
```

2. Node

>> Node: 工作负载节点, 可以是物理机, 虚拟机. 每个Node节点都会被Master分配一些工作负载(Docker 容器), 当某个 Node 宕机时,
>> 其上的工作负载会被 Master 自动转移到其它节点

2.1 kubelet

```kubelet
负责Pod对应的容器的创建, 启停等任务, 同时与Master密切协作, 实现集群管理的基本功能.
Node 可以在运行期间动态增加到 Kubernetes 集群中, 前提是这个节点上已经正确安装,配置和启动上述关键进程, 在默认情况下 kubelet
会向 Master 注册自己. kubelet 会定时向 Master 汇报自身的情报(操作系统, Docker版本, 机器CPU内存情况, 当前有哪些Pod运行等).
Master 通过这些信息实现高效均衡的资源调度策略.当某个 Node 在超过指定时间不上报信息时, 会被 Master 判断为 "失联", Node 的
状态会被标记为不可用(Not Ready), 随后Master会触发"工作负载大转移" 的自动流程.
```

2.2 kube-proxy

```kube-proxy
实现Kubernetes Service 的通信与负载均衡机制的重要组件
```

2.3 Docker Engine (docker)

```docker engine
Docker 引擎, 负责本机的容器创建和管理工作
```

3. Pod

```Pod
一个Node 节点上可以有上百个 Pod, 每一个Pod中都运行着一个特殊的容器 Pause, 和其它业务容器(工作负载), 这些容器共享Pause的网络栈
和Volume挂载卷, 因此它们之间的通信和数据交换更为高效. 所以, 可以将一组密切相关的服务进程放入同一个 Pod. 另外, 之后提供服务的那
组 Pod 才会被映射为一个服务.

1. Pause 容器状态作为Pod的根容器, 它的状态代表整个容器组的状态

2. Pod里的多个业务容器共享 Pause容器的IP, 共享Pause容器挂接的Volume, 以解决业务容器之间的通信问题和文件共享问题.

3. Kubernetes 为每个Pod都分配的唯一的IP地址 Pod IP, 一个Pod 里的多个容器共享 Pod IP 地址. Kubernetes 要求底层网络支持集群内
任意两个Pod之间的TCP/IP直接通信.
```

3.1 Pod 类型

```Pod 类型
普通Pod:
一旦被创建就放入 ectd 中存储, 随后被 Kubernetes Master 调度到某个具体的Node 上, 进行绑定(Binding).
该Pod会被该Node上的 kubelet 进程实例化成一组相关的Docker容器启动. 默认情况下, 当Pod里的某个容器停止时,
Kubernetes 会自动检测到这个问题并重新启动这个Pod (重启Pod 里所有的容器). 若该Node宕机, 该Node上的所有
Pod 会被重新调度到其它节点上

静态Pod(Static Pod):
存放在某个具体的 Node 上的一个具体文件中, 并且只在此Node上启动, 运行.
```

3.2 Pod 定义文件示例 yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: myweb ## Pod 名称
    labels: ## 标签
        name: myweb
spec: ## Pod 中包含的容器
    containers:
        - name: myweb
          image: kubeguide/tmcat-app:v1
          ports:
            - containerPort: 8080 ## 容器启动端口
          env: ## 注入环境变量
            - name: MYSQL_SERVICE_HOST
              value: 'mysql'
            - name: MYSQL_SERVICE_PORT
              value: '3306'
         resources:
            requests: ## 该资源的最小申请量, 系统必须满足要求
                memory: "64Mi"
                cpu: "250m" ## 通常以千分之一来计算.
            limits: ## 该资源最大允许使用量, 当超过时, 容器可能会被Kubernetes "杀掉" 重启.
                memory: "128Mi"
                cpu: "500m"
```

4. label(标签)

```label
1) 一个 label 是一个 key=value 的键值对, 其中 key 与 value 由用户自己指定.

2) label 可以被附加到任何资源上, 如 Node, Pod, Service, RC.

3) 一个资源对象可以定义任意数量的label, 同一个label也可以被添加到任意数量的资源对象上. 可以动态添加和删除

4) 通过 label 来实现多维度的资源分组管理功能, 以便灵活,方便的进行资源分配, 调度, 配置, 部署等管理工作.
```

4.1 label selector(标签选择器)

```label selector
通过 label selector 管理一组资源对象.

1). 基于等式 =, !=

2). 基于集合 in, not in

3). matchLabels: 定义需要匹配的 label

4). matchExpressions: 通过 1) 与 2) 的方式匹配符合条件的 label

```

4.2 label selector 在 Kubernetes 中的重要使用场景

```使用场景
1). kube-controller 进程通过在资源对象RC上定义的label selector 来筛选要监控的Pod副本数量, 使Pod副本数量始终
符合预期设定的全自动控制流程

2). kube-proxy 进程通过 Service 的 label selector 来选择对应的 Pod, 自动建立每个 Service 到对应 Pod 的请求
转发路由表, 从而实现 Service 的智能负载均衡机制.

3). 通过对某些 Node 定义特定的 label, 并且在 Pod 定义文件中使用 NodeSelector 这种标签调度策略, kube-scheduler
进程可以实现Pod定向调度的特性.

总之, 使用 label 可以给对象创建多组标签, label 和 label selector 共同构成了 Kubernetes 系统中核心的应用模型,
使得被管理对象能够被精细的分组管理, 同时实现了整个集群的高可用性.
```

5. Replication Controller (RC)

```RC
定义了一个期望的场景, 即声明某种 Pod 的副本数量在任意时刻都符合某个预期值
1). Pod 期待的副本数量

2). 用于筛选目标 Pod 的 Label Selector

3). 当 Pod 副本数量小于预期数量时, 用于创建新的 Pod 的Pod模板. (template)

当我们定义了一个RC并将其提交到 Kubernetes 集群中后, Master上的 Controller Manager组件就得到通知, 定期巡检
系统中当前存活的目标Pod, 并确保目标Pod实例的数量刚好等于此RC的期望值. 自动化创建和销毁Pod.

1). RC Pod 动态缩放, kubectl scale

2). 滚动升级(Rolling Update), 可以通过每停止一个旧Pod, 然后升级一个新 Pod, 实现动态滚动升级. 实现灰度发布.
```

5.1 rc.yaml 示例

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
    name: frontend
spec:
    replicas: 1
    selector:
        tier: frontend
    template:
        metadata:
            labels:
                app: app-demo
                tier: fronetend
        spec:
            containers:
                - name: tomcat-demo
                  image: tomcat
                  imagePullPolicy: IfNotPresent
                  env:
                    - name: GET_HOSTS_FROM
                      value: dns
                  ports:
                    -containerPort: 80
```

5.2 Replica Set

```Replica Set
Replication Controller 的升级版本 (还有 Deployment)

Replication Controller 只支持基于等式的 label selector

Replica Set 支持基于集合的 label selector (matchLabels, matchExpressions)
```

5.3 特性总结

```特性总结
1). 在大多数情况下, 我们通过定义一个 RC 实现 Pod 的创建及副本数量的自动控制

2). 在 RC 里包括完整的 Pod 定义模板

3). RC 通过Label Selector 机制实现对 Pod 副本的自动控制

4). 通过改变RC里的Pod 副本数量, 可以实现Pod的扩容或缩容

5). 通过改变RC里Pod模板中的镜像版本, 可以实现Pod 的滚动升级.

```

6. Deployment

>> 可以更好地解决Pod的编排问题, Deployment 内部使用了 Replica Set来实现目的, 可以看作 RC 的一次升级, 两者的相似度超过90%.
>> 最大的升级时我们可以随时知道当前 Pod "部署" 的进度, 可以知道这个"部署过程" 的持续进度

6.1 Deployment 的典型使用场景

```使用场景
1. 创建一个 Deployment 对象来生成对应的 Replica Set 并完成 Pod 副本的创建

2. 检查 Deployment 的状态来看部署动作是否完成(Pod 副本数量是否达到预期的值)

3. 更新 Deployment 以创建新的 Pod (比如镜像升级)

4. 如果当前 Deployment 不稳定, 则回滚到一个早先的 Deployment 版本 (版本控制)

5. 暂停 Deployment 以便于一次性修改多个 PodTemplateSpace 的配置项, 之后再恢复 Deployment, 进行新的发布

6. 扩展 Deployment 以应对高负载

7. 查看 Deployment 的状态, 以此作为发布是否成功的指标.

8. 清理不再需要的旧版本 ReplicaSets
```

6.2 Deployment yaml 示例

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: frontend
spec:
  replica: 1
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [fronted]}
  template:
    metadata:
      labels:
        app: app-demo
        tier: frontend
    spec:
      containers:
        - name: tomcat-demo
          image: tomcat
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort:  8080
```

7. Horizontal Pod Autoscaler (HPA, Pod 横向自动扩展)

>> 自动化, 智能化, 能够根据当前负载自动触发水平扩容或缩容, 这一过程可能频繁发生, 不可预料

7.1 Pod负载度量指标方式(根据指标判断是否扩容)

```HPA
1. CPUUtilizationPercentage
CPUUtilizationPercentage => 算术平均值, 即目标Pod所有副本自身的CPU利用率的平均值.
一个Pod自身的CPU利用率是该Pod当前CPU的使用量除以它的Pod Request的值.

2. 应用程序自定义的度量指标(比如: 服务器在每秒内的相应请求数 TPS/QPS)
Kubernetes Monitoring Architecture中, Kubernetes 定义了一套标准化的API接口 Resource Metrics API, 以客户端应用程序从
Metrics Server 中获取目标资源对象的性能数据. (如: CPU和内存的使用数据)
```

7.2 HPA yaml 实例

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
  namespace: default
spec:
  maxReplicas: 10       ## 扩容时, 最大 Pod 数
  minReplicas: 1        ## 缩容时, 最小 Pod 数
  scaleTargetRef:
    kind: Deployment    ## 自动扩容目标对象类型
    name: php-apache    ## 自动扩容目标对象名称
  targetCPUUtilizationPercentage: 90    ## 当 Pod副本的 CPUUtilizationPercentage 超过 90% 时, 会触发自动动态扩容行为.
```

8. StatefulSet (有状态的Pod集)

>> 从本质上来说, 可以看作 Deployment/RC 的一个特殊变种

8.1 特性

```statefulSet
1. StatefulSet 里的每个Pod都有稳定, 唯一的网络标识, 可以用来发现集群内的其它成员.
假设 StatefulSet 的名称为 Kafka, 那么第1个 Pod 叫 kafka-0, 第2个 kafka-1, 以此类推

2. StatefulSet 控制的Pod 副本的启停顺序是受控的, 操作第n个Pod 时, 前 n-1 个 Pod 已经是运行且准备好的状态.

3. StatefulSet 里的Pod 采用稳定的持久化存储卷, 通过 PV 或 PVC 来实现, 删除 Pod 时默认不会删除与 StatefulSet 相关的存储卷
(为了保证数据的安全)
```

8.2 Headless Service

```Headless Service
StatefulSet 要与 Headless Service 配合使用, 即在每个 StatefulSet 定义中都要声明它属于哪个 Headless Service.

Headless Service 没有 Cluster IP, 若解析 Headless Service 的DNS 域名, 则返回的是该 Service 对应的全部 Pod Endpoint 列表.
StatefulSet 在 Headless Service的基础上又为了 StatefulSet 控制的每个Pod实例都创建了一个 DNS 域名.
格式: $(podname).$(headless service name)
```

9. Service (服务, Kubernetes 的核心资源对象之一)

![avatar](source\Service视图.png)

```service
Kubernetes 里的每个Service 其实就是我们经常提前的微服务架构中的一个微服务.

RC 的作用实际上是保证 Service 的服务能力和服务质量始终符合预期标准. (保证副本 Replica 个数)
```

9.1 Service (Name + Cluster IP) + Pod IP

```Service + Pod
每一个 Service 有一个全局唯一的 Cluster IP, 在 Service 整个生命周期内, Cluster IP 不会改变
通过Service Name 和 Cluster IP 地址做DNS域名映射, 接收请求, 再通过 kube-proxy 进程做负载均衡
将service 接收到的请求转发到 Pod 集群列表. 每个 Pod 都会被分配一个单独的 IP, (作为Service 转发列表)
且提供 Endpoint (Pod IP + ContainerPort)以被客户端访问.
```

9.2 Kubernetes的服务发现机制

```服务发现机制
1. 首先, 每个 Kubernetes 中的 Service 都有唯一的 Cluster IP 及唯一的名称

2. 如何通过 Service name 找到 Cluster IP

3. Kubernetes 通过 Add-On 增值包引入了 DNS 系统, 把服务名作为 DNS 域名, 这样程序就可以直接使用服务名建立通讯连接了.
```

9.3 外部系统访问 Service 的问题

```Service
Node IP: Node 的 IP 地址
Node 节点的物理网卡的IP地址, 真实物理网络地址

Pod IP: Pod 的 IP 地址
Docker Engine 根据docker0 网桥的IP地址段进行分配的, 通常是一个虚拟的二层网络. Kubernetes中, Pod里的容器访问另一个Pod
里的容器即通过虚拟二层网络, 但是真实 TCP/IP 流量是通过 Node IP 所在物理网卡流出的.

Cluster IP: Service 的 IP 地址
1). Cluster IP 仅仅作用于 Kubernetes Service 这个对象, 并有 Kubernetes 管理和分配 IP 地址(来源于 Cluster IP 地址池)

2). Cluster IP 无法被ping, 因为没有一个 "实体网络对象" 来响应.

3). Cluster IP 只能结合Service Port 组成一个具体的通信端口, 单独的 Cluster IP 不具备 TCP/IP 通信的基础, 并且它们属于
Kubernetes 集群封闭空间, 外部访问需特殊处理.

4). Kubernetes 集群内, Node IP网, Pod IP 网, Cluster 网之间的通信, 采用的是 Kubernetes 自己设计的特殊路由规则.
```

>> 外部访问Service 需通过真实IP, 如果 Node IP + NodePort, 或通过负载均衡器 (负载均衡)

10. Job (批处理类型应用)

```Job
1). Job 控制的 Pod 副本是短暂运行的, 可以将其视为一组 Docker 容器, 其中的每个 Docker 容器仅仅运行一次.

2). Job 所控制的 Pod 副本的工作模式能够多实例并行计算.
```

11. Volume (存储卷)

```volume
volume 是 Pod 中能够被多个容器访问的共享目录.
1). volume 被定义在 Pod 上.

2). 一个 Pod 里的多个容器挂载到具体的文件目录下.

3). volume 与 Pod 的生命周期相同, 但不相关, 容器重启或终止时, volume 中的数据不会丢失.

volume 使用
1). 在 Pod 声明一个 volume

2). 在容器里引用该 volume 并挂载到容器里的某个具体目录上.
```

12. Persistent Volume

```PV
PV 可以被理解成 Kubernetes 集群中的某个网络存储对应的一块存储, 它与 Volume 类似.
区别
1). PV 只能是网络存储, 不属于任何 Node, 但可以在每个 Node 上访问.

2). PV 并不是被定义在 Pod 上的, 而是独立于 Pod 之外定义的.

3). 支持各种网络 PV
```

12.1 PV yaml 示例

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  assessModes:
    - ReadWriteOnce
  nfs:
    path: /somepath
    server: 172.17.0.2
```

12.3 accessModes 属性

```accessModes
1. ReadWriteOnce: 读写权限, 并且只能被单个Node挂载

2. ReadOnlyMany: 只读权限, 允许被多个Node挂载.

3. ReadWriteMany: 读写权限, 允许被多个 Node 挂载.
```

13. namespace (命名空间)

>> Namespace 在很多情况下用于实现多租户的资源隔离. 可以在 metadata 中声明 namespace 属性, 表明属于哪一个 namespace

14. Annotation

>> 与 label 类似, 也使用 key/value 键值对的形式进行定义. 不同的是 Label 有严格的命名规范, 定义的是 Kubernetes 对象的
>> 元数据(metadata), 且用于 Label Selector
>> Annotation 是用户任意附加的数据, 用于外部工具查询

15. ConfigMap

```configMap
Kubernetes 将 所有的配置项都当做 key=value 字符串, 存储到 Map, 并持久化到 Kubernetes 的Ectd 数据库中.
Kubernetes 可以通过一种内建的方式, 将存储在 ectd 中的 ConfigMap 通过 Volume 映射的方式变成目标 Pod 内的配置文件.
而且, 如果 configMap 中数据被修改, 映射数据也会被修改.
```