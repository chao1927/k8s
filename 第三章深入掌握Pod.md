# 深入掌握 Pod

## 1. Pod 定义详解

### 1.1 YAML格式的 Pod 定义文件的完整内容

```yaml
apiVersion: v1              ## 版本号
kind: Pod                   ## Pod
metadata:                   ## 元数据
    name: string            ## Pod 名称
    namespace: string       ## Pod 所属命名空间, 默认值为 default
    labels:                 ## 自定义标签列表
        - name: string
    annotations:            ## 自定义注解列表
        - name: string
spec:                       ## Pod 容器的详细定义
    containers:             ## Pod 中的容器列表
        - name: string      ## 容器名称
          image: string     ## 容器镜像名称
          imagePullPolicy: [Always | Never | IfNotPresent]
          command: [string] ## 容器启动命令列表, 如果不指定, 则使用镜像打包时使用的启动命令
          args: [string]    ## 容器的启动参数列表
          workingDir: string    ## 容器的工作目录
          volumeMounts:         ## 挂载到容器的内部存储卷配置
            - name: string      ## 引用 Pod 定义的共享存储卷的名称, 需使用 volumes[] 部分定义的共享存储卷名称
              mountPath: string ## 存储卷在容器内Mount 的绝对路径
              readOnly: boolean ## 是否为只读模式, 默认为只读模式
          ports:                ## 容器需要暴露的端口号列表
            - name: string      ## 端口的名称
              containerPort: int    ## 容器需要监听的端口号
              hostPort: int         ## 容器所在主机需要监听的端口号, 默认与 containerPort 相同, 设置 hostPort 时, 同一台宿主机将无法启动该容器的第二份副本
              protocol: string  ## 端口协议: 支持 TCP/UDP, 默认 TCP
          env:                  ## 容器运行钱需要设置的环境变量列表
            - name: string      ## 环境变量的名称
              value: string     ## 环境变量的值
          resources:            ## 资源限制和资源请求的限制
            limits:             ## 资源限制
                cpu: string     ## cpu 限制, 单位可以为 core 数, 将用于 docker run --cpu-shares
                memory: string  ## 内存限制, 单位可以为 MiB, GiB, 将用于 docker run --memory
            requests:           ## 资源请求的设置
                cpu: string     ## cpu 请求, 单位可以为 core 数, 将用于 docker run --cpu-shares
                memory: string  ## 内存请求, 单位可以为 MiB, GiB, 将用于 docker run --memory
          livenessProbe:        ## 容器探测, 健康检查
            exec:
                command: [string]
            httpGet:
                path: string
                port: string
                host: string
                scheme: string
                httpHeaders:
                    - name: string
                      value: string
            tcpSocket:
                port: number
            initialDelaySeconds: 0
            timeoutSeconds: 0
            periodSeconds: 0
            successThreshold: 0
            failureThreshold: 0
            securityContext:
                privileged: false
    restartPolicy: [Always | Never | OnFailure]
    nodeSelector: Object
    imagePullSecrets:
        - name: string
    hostNetwork: false
    volumes:                ## 在该 Pod 上定义的共享存储卷列表
        - name: string
          emptyDir: {}
          hostPath:
            path: string
          secret:
            secretName: string
            items:
                - key: string
                  path: string
        configMap:
            name: string
            items:
                - key: string
                  path: string
```

## 2. Pod 的基本用法

```Pod 的基本用法
在使用 Docker 时, 可以使用 docker run 命令创建并启动一个容器, 而在 Kubernetes 系统中对长时间运行容器的要求是:
其主程序需要一直在前台执行. 如果常见的 Docker 镜像的启动命令是后台执行程序, 如:

    nohup ./start.sh &

则在 kubelet 创建包含这个容器的 Pod 之后运行完该命令, 即认为 Pod 执行结束, 将立刻销毁该 Pod. 如果为该 Pod 定义了
ReplicationController, 则系统会监控到该 Pod 已经终止, 之后根据RC定义中 Pod 的replicas 副本数量生成一个新的Pod.
一旦创建新的 Pod, 就在执行完启动命令后陷入无限循环的过程中.

1). 可以通过 Supervisor 辅助进行前台运行的功能.

2). Pod 可以由1个或多个容器组合而成.
```

## 3. 静态 Pod

```静态 Pod
静态 Pod 是由 kubelet 进行管理的仅存在于特定 Node 上的 Pod. 不能通过 API Server 进行管理, 无法与 ReplicationController
Deployment或者 DaemonSet 进行关联, 并且 kubelet 无法对它们进行健康检查. 静态 Pod 总是由 kubelet 创建的, 并且总在 kubelet
所在的 Node 上运行.
```

3.1 创建静态 Pod 的方式

```创建静态 Pod 的方式
1). 配置文件方式
设置kubelet的启动参数 "--config", 指定 kubelet 需要监控的配置文件所在目录, kubelet 会定期扫描该目录, 并根据目录下的
.yaml 或 .json 文件进行创建操作.

2). HTTP 方式
通过设置 kubelet 的启动参数 "--manifest-url", kubelet 将会定期从该 URL 地址下载 Pod 的定义文件, 并以 .yaml 或 .json
格式进行解析, 然后创建 Pod
```

## 4. Pod 容器共享Volume

```Pod 容器共享 Volume
同一个Pod中的多个容器能够共享Pod级别的存储卷Volume. Volume 可以被定义为各种类型, 多个容器各自进行挂载操作, 将一个Volume
挂载为容器内部需要的目录.
```

>> Pod 中多个容器共享Volume

4.1 Pod Volume 挂载示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
    - name: tomcat
      image: tomcat
      ports:
        - containerPort: 8080
      volumeMounts:
        - name: app-logs
          mountPath: /usr/local/tomcat/logs
        - name: busybox
          image: busybox
          command: ["sh", "-c", "tail -f /logs/catalina*.log"]
          volumeMounts:
            - name: app-logs
              mountPath: /logs
  volumes:
    - name: app-logs
      emptyDir: {}
```

## 5. Pod的配置管理

```ConfigMap
应用部署的一个最佳实践是将应用所需的配置信息与程序进行分离, 这样可以使应用程序可以更好的被复用, 通过不同的配置也能够实现更灵活的功能.
将应用打包为容器镜像后, 可以通过环境变量或者外挂文件的方式在创建容器时进行配置注入, 但是, 在大规模容器集群的环境中, 对多个容器进行不同
的配置将变得非常复杂.

通过将数据与处理过程分离, 使得程序更好地被复用.
```

5.1 ConfigMap 概述

```ConfigMap
ConfigMap 用法

1). 生成为容器内的环境变量

2). 设置容器启动命令的启动参数

3). 以 Volume 的形式挂载为容器内部的文件或目录
```

5.2 创建 ConfigMap 资源对象

1. 通过YAML配置文件方式创建

```yaml
## cm-appvars.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-appvars
data:
  apploglevel: info
  appdatadir: /var/data
```

```ConfigMap
执行 kubectl create -f cm-appvars.yaml

查看 kubectl get configmap
    /kubectl get configmap cm-appvars -o yaml
```

2. 通过 kubectl 命令行方式创建

1) 通过 --from-file 参数从文件中进行创建, 可以指定 key 的名称, 也可以在一个命令行中创建包含多个 key 的ConfigMap.

> kubectl create configmap NAME --from-file=[key=]source

2). 通过 --from-file 参数从目录中进行创建, 该目录下的每个配置文件名都被设置为 key, 文件的内容被设置为 value:

> kubectl create configmap NAME --from-file=config-files-dir

3). 使用 --from-literal 时会从文件中进行创建, 直接将指定的 key#=value# 创建为 ConfigMap 的内容:

> kubectl create configmap NAME --from-literal=key1=value1 --from-literal=key2=value2

3. 在 Pod 使用 ConfigMap

1). 通过环境变量方式使用 ConfigMap

1.1 使用示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-test-pod
spec:
  containers:
    - name: cm-test
      image: busybox
      command: ["/bin/sh", "-c", "env | grep APP" ]
      env:
        - name: APPLOGLEVEL     ## 定义环境变量的名称
          valueFrom:            ## key "apploglevel" 对应的值
            configMapKeyRef:
              name: cm-appvars  ## 环境变量的值取自 cm-appvars
              key: apploglevel  ## key 为 apploglevel
        - name: APPDATADIR      ## 定义环境变量名称
          valueFrom:            ## key "appdatadir" 对应的值
            configMapKeyRef:    ## 环境变量的值取自 cm-appvars
              name: cm-appvars  ## key 为 appdatadir
  restartPolicy: Never
```

> 创建 kubectl create -f cm-test-pod.yaml
> 查看 kubectl get pods --show-all

1.2 envFrom

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-test-pod
spec:
  containers:
    - name: cm-test
      image: busybox
      command: [ "/bin/sh", "-C", "env" ]
      envFrom:
        - configMapRef
          name: cm-appvars    ## 根据cm-appvars中的key=value自动生成环境变量
  restartPolicy: Never
```

2). 通过 volumeMount 使用 ConfigMap

示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-test-app
spec:
  containers:
    - name: cm-test-app
      image: kubeguide/tomcat-app:v1
      ports:
        - containers: 8080
      volumeMounts:
        - name: serverxml                 ## 引用Volume的名称
          mountPath: /configfiles         ## 挂载到容器内的目录
  volumes:
    - name: serverxml                     ## 定义Volume的名称
      configMap:
        name: cm-appconfigfiles           ## 使用 ConfigMap "cm-appconfigfiles"
        items:
          - key: key-serverxml            ## key=key-serverxml
            path: server.xml              ## value 将server.xml文件名进行挂载
          - key: key-loggingproperties    ## key=key-lohhingproties
            path: logging.properties      ## value 将logging.properties
```

> 创建 kubectl create -f cm-test-app.yaml

5.4 使用ConfigMap 的限制条件

```限制条件
1). ConfigMap 必须在 Pod 之前创建

2). ConfigMap 受 Namespace 限制, 只有处于相同 Namespace 中的 Pod 才可以引用它.

3). ConfigMap 中的配额管理还未能实现

4). kubelet 只支持可以被 API Server 管理的 Pod 使用 ConfigMap. (注意静态Pod)

5). 在 Pod 对 ConfigMap 进行挂载 (volumeMount) 操作时, 在容器内部只能挂载为 "目录", 无法挂载为 "文件".
```

## 6. 在容器内获取Pod信息 (Downward API)

```Downward API
Downward API 可以通过以下两种方式将Pod信息注入容器内部

1). 环境变量: 用于单个变量, 可以将Pod信息和 Container 信息注入容器

2). Volume 挂载: 将数组类信息生成为文件并挂载到容器内部.
```

6.1 环境变量方式: 将 Pod 信息注入为环境变量

```yaml
## dapi-test-pod.xml

apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: busybox
      command: [ "/bin/sh", "-C", "env"]
      env:
         - name:  MY_POD_NAME
           valueFrom:
              fieldRef:
                fieldPath: metadata.name
  restartPolicy: Never
```

目前 Downward API 提供的变量:
metadata 相关
metadata.name, status.podIP, metadata.namespace

resource 相关
requests.cpu, limits.cpu, requests.memory, limits.memory

6.2 Volume 挂载方式

6.3 Downware API 价值

```Downware API
在某些集群中, 集群中的每个节点都需要将自身的标识 (ID) 及进程绑定的IP地址等信息事先写入到配置文件中,
进程在启动时会读取这些信息, 然后将这些信息发布到某个雷士服务注册中心的地方, 以实现集群节点的自动发现
功能.

做法: 1. 编写一个预启动脚本或 Init Container, 通过环境变量或文件方式获取 Pod 自身的名称, IP 地址等信息.
      2. 将这些信息写入主程序的配置文件中, 最后启动主程序.
```

## 7. Pod生命周期和重启策略

7.1 Pod 状态

```Pod生命周期和重启策略
Pending: API Server 已经创建该 Pod, 但在Pod 内还有一个或多个容器的镜像没有创建, 包括正在下载镜像的过程
Running: Pod 内所有容器均已创建, 且至少有一个容器处于运行状态, 正在启动状态或正在重启状态
Succeeded: Pod 内所有容器均已成功执行后退出, 且不会再重启.
Failed: Pod 内所有容器均已退出, 但至少有一个容器退出为失败状态.
Unknown: 由于某种原因无法获取该 Pod 的状态, 可能由于网络通信不畅导致.
```

7.2 Pod 的重启策略

```Pod的重启策略
Always: 当容器失效时, 由kubelet 自动重启该容器.

OnFailure: 当容器终止且退出码不为0时, 有 kubelet 自动重启该容器.

Never: 不论容器运行状态如何, kubelet 都不会重启该容器.
```

7.3 RC, Job, DaemonSet 及静态Pod 对 Pod 的重启策略要求:

```RC, Job, DaemonSet 及静态Pod 对 Pod 的重启策略要求:
1). RC 和 DaemonSet: 必须设置为 Always, 需要保证该容器持续运行.

2). Job: OnFailure 或者 Never, 确保容器完成后不再重启.

3). kubelet: 在 Pod 失效时自动重启它, 不论将 RestartPolicy 设置为什么值, 也不会对 Pod 进行健康检查.
```
