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
