# Kubernetes教程

## 1. k8s集群介绍

Kubernetes（简称K8s）是一个开源的容器编排平台，用于自动化部署、扩展和管理容器化应用程序。它将组成应用程序的容器组合成逻辑单元，以便于管理和发现。

K8s集群分为Master节点和Worker节点，主要特征：

- **Master节点**：负责管理整个集群，协调集群中的所有活动，例如调度应用程序，维护应用程序的状态，扩展和更新应用程序。包含多个组件：API Server、Controller Manager、Scheduler、etcd。
- **Worker节点**：是VM（虚拟机）或物理计算机，充当k8s集群中的工作计算机。每个Worker节点都有一个Kubelet，它管理该Worker节点并负责与Master节点通信。该Worker节点还应具有用于处理容器操作的工具，例如Docker。

## 2. 基本概念

### 2.1. Deployment 部署

在k8s中，通过发布Deployment，可以创建应用程序（docker image）的实例（docker container），这个实例会被包含在称为Pod的概念中，Pod是k8s中最小可管理单元。

- k8s集群中发布Deployment后，Deployment将指示k8s如何创建和更新应用程序的实例，master节点将应用程序实例调度到集群中的具体的节点上。
- 创建应用程序实例后，Kubernetes Deployment Controller会持续监控这些实例，如果运行实例的worker节点关机或被删除，则Kubernetes Deployment Controller将在集群中资源最优的另一个worker节点上重新创建一个新的实例。这提供了一种自我修复机制来解决机器故障或维护问题。
- Deployment处于Master节点上，通过发布Deployment，Master节点会选择合适的Worker节点创建Container，Container会被包含在Pod里。

**Deployment的优势**：
- 自我修复：节点故障时自动在其他节点重新创建Pod
- 水平扩展：可以方便地增加或减少Pod数量
- 滚动更新：支持零停机更新应用程序
- 版本控制：可以回滚到之前的版本

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3  # 创建3个副本
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

### 2.2. Pods 容器组

Pod是k8s集群上的最基本的单元。当我们在k8s上创建Deployment时，会在集群上创建包含容器的Pod而不是直接创建容器。每个Pod都与运行它的节点Node绑定，并保持在那里直到被删除。如果节点发生故障，则会在集群中的其他可用节点上运行相同的Pod（从同样的镜像创建Container，使用同样的配置，IP地址不同，Pod名字不同）。

Pod容器组是一个k8s中一个抽象概念，用于存放一组container（可包含一个或多个），以及这些container容器的一些共享资源。这些资源包括：

- 共享存储，称为卷（Volumes）
- 网络，每个Pod在集群中有个唯一的IP，Pod中的container共享该IP地址。
- container的基本信息，例如容器的镜像版本，对外暴露的端口等。

**Pod的特点**：
- 容器的包装器，提供共享上下文
- 共享网络命名空间（同一Pod中的容器可通过localhost互相访问）
- 共享存储卷
- 共享生命周期（一起创建和销毁）

**应用场景**：
- 如果多个容器紧密耦合并且需要共享磁盘等资源，则他们应该被部署在同一个Pod容器组中。
- 主容器+辅助容器（如日志收集）
- 需要共享网络和存储的多个容器

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

### 2.3. Node节点

Node节点是kubernetes集群中的计算机，可以是虚拟机或物理机。每个Node节点都由Master管理，一个Node节点可以有多个Pod容器组，Kubernetes master会根据每个Node节点上可用资源情况，自动调度Pod容器组到Node节点上。

每个Kubernetes Node至少运行：

- Kubelet，负责Master节点和Worker节点之间通信的进程；管理Pod容器组和Pod容器组内运行的Container容器。
- 容器运行环境（如Docker）负责下载镜像、创建和运行容器等。
- Kube-proxy，维护网络规则，实现服务通信


## 3. PV 持久卷（PersisitentVolume）
### 3.1. 基本概念
- PV持久卷，其是k8s中对实际物理存储系统的抽象，作为k8s中集群层面的资源，与节点资源类似，PV不属于任何命名空间。
- PersistentVolumelClaim（PVC持久卷申领），其是k8s中用户对存储请求的抽象。类似地，Pod会消耗节点资源，而PVC则会消耗PV资源。
- PV实现了对底层真实存储系统的抽象，使得开发者只需要创建PVC资源即可。
- PVC可以作为卷在工作负载资源中进行引用，当集群中满足PVC要求的PV时，则会将该PV与PVC双向绑定。

**PV和PVC的工作流程**：
1. 管理员创建PV或配置动态供应
2. 用户创建PVC请求存储
3. K8s系统找到匹配的PV并绑定
4. Pod使用PVC作为卷挂载

```yaml
# PV示例
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-example
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data/pv0001

# PVC示例
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

### 3.2. 参数介绍

#### 3.2.1. accessModes（访问模式）

- **ReadWriteOnce（RWO）**：卷可以被一个节点以读写方式挂载，这种方式允许在同一个节点上的多个Pod访问卷
- **ReadOnlyMany（ROM）**：卷可以被多个节点以只读方式挂载
- **ReadWriteMany（RWM）**：卷可以被多个节点以读写方式挂载
- **ReadWriteOncePod（RWOP）**：在Kubernetes 1.22以上版本，卷可以被单个Pod以读写方式挂载

### 3.3. persistentVolumeReclaimPolicy（回收策略）

- **Retain（保留）**：当PVC资源被删除时，其绑定的PV不会被删除，但该PV状态会变为[Released已释放]，由于此时PV关联的物理存储存在历史数据，故该PV不能直接被申领
- **Delete（删除）**：不需要管理员介入，系统直接删除PV资源及数据
- **Recycle（回收）**：基本擦除（已弃用）

> TIP
> - 卷能都支持多种访问模式


## 4. ConfigMap

### 4.1. 基本概念

- ConfigMap是一种用于存储非敏感配置数据的资源对象，它允许你将配置信息从容器中分离出来，以便在不重新构建镜像的情况下进行配置更改。
- ConfigMap可以在多个Pod之间共享配置，从而简化了应用程序的配置管理。

### 4.2. 用法
- 创建ConfigMap，使用`kubectl create -f configmap.yaml`命令进行创建。

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  API_KEY: "sldkjf234kj2"
  DB_URL: "mongodb://localhost:27017"
  config.json: |
    {
      "timeout": 120,
      "cache_ttl": 300
    }
```

- 挂载到Pod，通过Pod的`spec.containers.env`或者`spec.volumes`中引用ConfigMap，Pod中的容器可以通过环境变量或文件访问配置数据。
- 配置文件，在Pod的`spec.volumes`中引用ConfigMap，可以将配置数据作为环境变量注入到容器中。

```yaml
# 在Pod中使用ConfigMap
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app-container
    image: myapp:1.0
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: DB_URL
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-config
```

### 4.3. 应用场景
- 将应用程序配置从镜像中分离，使镜像更具通用性
- 共享配置数据，确保多个Pod使用相同的配置
- 在不重新构建镜像的情况下，更改应用程序配置


## 5. Pod部署

### 5.1. 常用部署命令

| Command                                         | Desc                      |
|-------------------------------------------------|---------------------------|
| kubectl apply -f nginx-deployment.yaml          | 应用yaml文件              |
| kubectl get deployments                         | 查看Deployment            |
| kubectl get pods                                | 查看Pod                   |
| kubectl logs -f [pod_name]                      | 查看Pod容器的打印日志     |
| kubectl describe pods [pod_name] -n [namespace] | 查看pod详细信息           |
| kubectl exec -it [pod_name] /bin/bash           | 在Pod中容器环境中执行命令 |
| kubectl scale deployment/name --replicas=5      | 扩展Deployment副本数      |
| kubectl port-forward pod/pod-name 8080:80       | 将本地端口转发到Pod       |


### 5.2. yaml文件介绍

```yaml
apiVersion: apps/v1 # 与k8s集群版本有关，使用 kubectl api-versions 即可查看当前集群支持的版本
kind: Deployment # 该配置的类型，我们使用的是 Deployment
metadata:         # 译名为元数据，即 Deployment 的一些基本属性和信息
  name: nginx-deployment # Deployment 的名称
  labels:     # 标签，可以灵活定位一个或多个资源，其中key和value均可自定义，可以定义多组，目前不需要理解
    app: nginx # 为该Deployment设置key为app，value为nginx的标签
spec:         # 这是关于该Deployment的描述，可以理解为你期待该Deployment在k8s中如何使用
  replicas: 1 # 使用该Deployment创建一个应用程序实例
  selector:     # 标签选择器，与上面的标签共同作用，目前不需要理解
    matchLabels: # 选择包含标签app:nginx的资源
      app: nginx
  template:     # 这是选择或创建的Pod的模板
    metadata: # Pod的元数据
      labels: # Pod的标签，上面的selector即选择包含标签app:nginx的Pod
        app: nginx
    spec:     # 期望Pod实现的功能（即在pod中部署）
      containers: # 生成container，与docker中的container是同一种
      - name: nginx # container的名称
        image: nginx:1.7.9 # 使用镜像nginx:1.7.9创建container，该container默认80端口可访问
```

## 6. Kubernetes Service概述

事实上，Pod容器组由自己的生命周期，当Worker Node节点故障时，节点上运行的Pod也会消失。然后，Deployment可以通过创建新的Pod容器组来动态将集群调整回原来的状态，以使应用程序保持运行。

### 6.1. Why Service

假设有一个图像处理后端程序，具有3个副本。这3个副本可以替换的无状态应用，即使Pod消失并被重新创建，或者副本数由3增加到5，传统开发前端系统需要一直关注后端副本的变化，IP变换了就要切换到对应的IP才能进行访问，因为每次创建Pod都会一个新的IP地址，非常麻烦。Service的出现，屏蔽了前后端系统的Pod在销毁、创建过程中所带来的IP地址变化，无论后端如何变化，前端依旧访问就可以。

- Kubernetes中的Service提供了这样的一个抽象层，它选择具备某些特征Pod容器组并为它们定义一个访问方式。Service服务使Pod容器组之间的相互依赖解耦（原本从一个Pod中访问另外一个Pod，需要知道对方的IP地址）。
- 一个Service服务选定那些Pod容器组通常由LabelSelector（标签选择器）来决定。

**Service的核心功能**：
- 提供稳定的网络访问地址（DNS名称和IP）
- 实现负载均衡
- 服务发现
- 支持零停机部署

### 6.2. 服务方式

创建Service的时候，通过设置已配置文件中的spec.type字段的值，可以以不同方式向外部暴露应用程序。

- **ClusterIP**（默认），在集群中的内部IP上公布服务，这种方式的Service（服务）只在集群内部可以访问到。
- **NodePort**，使用NAT在集群中每个的同一端口公布服务，这种方式下，可以通过访问集群中任意节点+端口号的方式访问服务`<NodeIP>:<NodePort>`。此时ClusterIP的访问方式仍然可用。
- **LoadBalancer**，在云环境中创建一个集群外部的负载均衡器，并为使用该负载均衡器的IP地址作为服务的访问地址，此时ClusterIP和NodePort访问方式仍然可用。
- **ExternalName**，将服务映射到外部DNS名称，主要用于访问集群外部的服务。

```yaml
# NodePort示例
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # 在所有节点上开放30080端口
  type: NodePort
```

> <b>TIP</b>
> Service是一个抽象层，它通过LabelSelector选择了一组Pod容器组，把这些Pod的指定端口公布到集群外部，并支持负载均衡和服务发现。
> - 公布Pod的端口以使其可访问。
> - 在多个Pod间实现负载均衡。
> - 使用Label和LabelSelector

### 6.3. Labels、LabelSelector

Service使用Label、LabelSelector（标签和标签选择器）匹配一组Pod。Labels是附加到Kubernetes对象的键值对，其用途有多种：

- 将Kubernetes对象（Node、Deployment、Pod、Service等）指派作用于开发环境、测试环境或生产环境。
- 嵌入版本标签，使用标签区别不同应用软件版本。
- 使用标签对Kubernetes对象进行分类。
- Labels标签可以在创建Kubernetes对象时附加上去，也可以在创建之后再附加上去。任何时候都可以修改一个Kubernetes对象的Labels标签。

**标签的用途**：
- 区分环境（dev、test、prod）
- 标识应用版本（v1、v2、beta）
- 分类应用组件（frontend、backend、database）

**选择器**：
- 通过标签筛选资源
- 常用于Service选择目标Pod
- 支持等值选择和集合选择

```yaml
# 使用标签
metadata:
  labels:
    app: myapp
    tier: frontend
    version: v1.3

# 使用选择器
selector:
  matchLabels:
    app: myapp
    tier: frontend
```

### 6.4. 实战

- 创建nginx-service，yaml文件如下
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service	# Service 的名称
  labels:     	# Service 自己的标签
    app: nginx	# 为该 Service 设置 key 为 app，value 为 nginx 的标签
spec:	    #  这是关于该 Service 的定义，描述了 Service 如何选择 Pod，如何被访问
  selector:	    # 标签选择器
    app: nginx	# 选择包含标签 app:nginx 的 Pod
  ports:
  - name: nginx-port	# 端口的名字
    protocol: TCP	    # 协议类型 TCP/UDP
    port: 80	        # 集群内的其他容器组可通过 80 端口访问 Service
    nodePort: 32600   # 通过任意节点的 32600 端口访问 Service
    targetPort: 80	# 将请求转发到匹配 Pod 的 80 端口
  type: NodePort	# Serive的类型，ClusterIP/NodePort/LoaderBalancer
```

- 创建nginx的Deployment中定义Labels，如下：

```yaml
metadata:	# 译名为元数据，即Deployment的一些基本属性和信息
  name: nginx-deployment	# Deployment的名称
  labels:	# 标签，可以灵活定位一个或多个资源，其中key和value均可自定义，可以定义多组
    app: nginx	# 为该Deployment设置key为app，value为nginx的标签
```

## 7. K8s实用操作指南

### 7.1. Pod状态排查

- **Pending**：Pod已创建但未调度到节点
- **Running**：Pod已绑定到节点，所有容器都在运行
- **Succeeded**：所有容器成功终止
- **Failed**：至少一个容器以非零状态终止
- **Unknown**：无法获取Pod状态，通常是节点通信问题

### 7.2. 常见问题排查流程

1. **Pod无法启动**：
   - 检查Pod状态：`kubectl describe pod <pod-name>`
   - 查看Pod日志：`kubectl logs <pod-name>`
   - 检查资源限制：CPU/内存限制是否合理

2. **服务无法访问**：
   - 确认Service配置：`kubectl describe svc <service-name>`
   - 检查Service与Pod标签匹配
   - 测试Pod网络连通性：`kubectl exec <pod-name> -- curl <service-ip>:<port>`

3. **存储问题**：
   - 检查PV/PVC状态：`kubectl get pv,pvc`
   - 检查存储类是否可用：`kubectl get storageclass`
   - 查看详细信息：`kubectl describe pv <pv-name>`

### 7.3. K8s最佳实践

1. **资源限制**：
   - 为Pod设置适当的CPU和内存限制，避免资源争用
   - 使用命名空间隔离不同环境和团队的资源

2. **健康检查**：
   - 使用liveness探针检测应用是否运行
   - 使用readiness探针检测应用是否准备好接收流量

3. **网络策略**：
   - 使用NetworkPolicy限制Pod间通信
   - 遵循最小权限原则

4. **配置管理**：
   - 使用ConfigMap管理配置
   - 使用Secret管理敏感信息
   - 避免在容器镜像中硬编码配置

5. **高可用部署**：
   - 使用多副本确保应用高可用
   - 使用反亲和性规则将Pod分散到不同节点
   - 定期备份etcd数据

## 8. 常用命令

### 8.1. k8s系统命令

| Command                        | Desc                  |
|--------------------------------|-----------------------|
| kubectl config view            | 查看kubectl上下文信息 |
| kubectl config current-context | 查看kubectl上下文信息 |
| kubectl cluster-info           | 显示集群信息         |
| kubectl api-versions           | 查看API版本           |



| Command                        | Desc                |
|--------------------------------|---------------------|
| kubectl get nodes -owide       | 查看所有节点        |
| kubectl get cs                 | 查看健康状态        |
| kubectl get pv -A -owide       | 查看挂载的点        |
| kubectl get pvc -A -owide      | 查看挂载            |
| kubectl get cm -A -owide       | 查看所有ConfigMap   |
| kubectl get pod -A -owide      | 查看所有的pod容器   |
| kubectl get svc -A -owide      | 查看所有的服务      |
| kubectl get services -A -owide | 查看所有的服务      |
| kubectl get deploy -A -owide   | 查看所有的发布服务  |
| kubectl get ingress -A -owide  | 查看ingress域名代理 |
| kubectl top node               | 查看节点资源使用情况 |
| kubectl top pod                | 查看Pod资源使用情况  |

