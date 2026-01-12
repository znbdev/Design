K8s
=====

在 Markdown 文档中绘制架构图，最通用且最强大的方式是使用 **Mermaid.js**。GitHub、GitLab、Obsidian 以及大多数现代 Markdown
编辑器都原生支持它。

以下为您提供两套不同视角的 K8s 部署架构图代码和说明：

1. **基础设施视角 (Infrastructure View)**：展示高可用 (HA) 集群的节点和组件布局。
2. **应用流量视角 (Traffic Flow View)**：展示请求如何从外部通过 Ingress 流向 Pod 和数据库。

-----

### 方案一：K8s 高可用集群架构 (Infrastructure)

这个图展示了一个生产环境级别的 K8s 集群，包含负载均衡器、3个控制平面（Master）节点和多个工作（Worker）节点。

您可以直接复制下面的代码块到您的 Markdown 文件中：

```mermaid
graph TD
    %% 定义样式
    classDef plain fill:#fff,stroke:#333,stroke-width:1px;
    classDef k8s fill:#326ce5,stroke:#fff,stroke-width:2px,color:#fff;
    classDef db fill:#e1f5fe,stroke:#0277bd,stroke-width:2px;

    subgraph External["外部访问 (External Access)"]
        User((用户/客户端))
        LB[("负载均衡器 (Load Balancer)<br/>VIP: 10.0.0.100")]:::plain
    end

    subgraph ControlPlane["控制平面 (Control Plane / Masters)"]
        direction TB
        subgraph Master1["Master Node 1"]
            API1[API Server]:::k8s
            ETCD1[etcd]:::db
            Sched1[Scheduler]:::plain
            CM1[Controller Mgr]:::plain
        end
        
        subgraph Master2["Master Node 2"]
            API2[API Server]:::k8s
            ETCD2[etcd]:::db
            Sched2[Scheduler]:::plain
            CM2[Controller Mgr]:::plain
        end
        
        subgraph Master3["Master Node 3"]
            API3[API Server]:::k8s
            ETCD3[etcd]:::db
            Sched3[Scheduler]:::plain
            CM3[Controller Mgr]:::plain
        end
    end

    subgraph DataPlane["数据平面 (Worker Nodes)"]
        subgraph Worker1["Worker Node 1"]
            Kubelet1[Kubelet]:::plain
            Proxy1[Kube-Proxy]:::plain
            Pod1((App Pod A)):::k8s
            Pod2((App Pod B)):::k8s
        end
        
        subgraph Worker2["Worker Node 2"]
            Kubelet2[Kubelet]:::plain
            Proxy2[Kube-Proxy]:::plain
            Pod3((App Pod A)):::k8s
            Pod4((App Pod C)):::k8s
        end
    end

    %% 连接关系
    User --> LB
    LB --> API1
    LB --> API2
    LB --> API3
    
    %% ETCD 集群通信
    ETCD1 <--> ETCD2
    ETCD2 <--> ETCD3
    ETCD1 <--> ETCD3

    %% API Server 通信
    API1 --> ETCD1
    API2 --> ETCD2
    API3 --> ETCD3

    %% Worker 连接 Master
    Kubelet1 -.-> LB
    Kubelet2 -.-> LB
    Proxy1 -.-> LB
    Proxy2 -.-> LB
```

-----

### 方案二：应用部署与流量流向图 (Application View)

这个图更适合开发人员，展示了一个典型的微服务应用是如何部署的，包括 Ingress、Service、Deployment、Pod 以及持久化存储（PVC）。

```mermaid
graph LR
%% 样式定义
    classDef ingress fill:#7c4dff,stroke:#fff,color:#fff;
    classDef svc fill:#ffca28,stroke:#fff,color:#333;
    classDef pod fill:#326ce5,stroke:#fff,color:#fff;
    classDef vol fill:#4caf50,stroke:#fff,color:#fff;

    User((互联网用户)) --> FW["防火墙/WAF"]

    subgraph K8s_Cluster["Kubernetes Cluster"]

        subgraph Namespace_App["Namespace: production"]
        %% 修复点：添加双引号包裹含有括号的文本
            ING["Ingress Controller<br/>(Nginx/Traefik)"]:::ingress

            subgraph Backend_Service
                SVC_BE["Service: Backend<br/>(ClusterIP)"]:::svc

                subgraph Deploy_BE["Deployment: Backend API"]
                    Pod1(Pod: Backend-v1):::pod
                    Pod2(Pod: Backend-v1):::pod
                    Pod3(Pod: Backend-v1):::pod
                end
            end

            subgraph Frontend_Service
                SVC_FE["Service: Frontend<br/>(ClusterIP)"]:::svc

                subgraph Deploy_FE["Deployment: Web UI"]
                    PodFE1(Pod: Web-v1):::pod
                    PodFE2(Pod: Web-v1):::pod
                end
            end

            subgraph Storage
                PVC[PersistentVolumeClaim]:::vol
                PV[PersistentVolume]:::vol
            end
        end
    end

    Database[("外部数据库 (RDS/Cloud SQL)")]

%% 流量路径
    FW --> ING
    ING --"/api"--> SVC_BE
    ING --"/"--> SVC_FE

SVC_BE --> Pod1 & Pod2 & Pod3
SVC_FE --> PodFE1 & PodFE2

%% 数据持久化与外部连接
Pod1 & Pod2 & Pod3 --> Database
Pod1 & Pod2 & Pod3 --"日志/文件"--> PVC
PVC -.-> PV
```

-----

### 核心组件说明

如果您需要在文档中解释这些架构，可以使用以下描述：

#### 1\. 控制平面 (Control Plane)

* **API Server:** 集群的统一入口，处理 REST 操作并更新 Etcd。
* **Etcd:** 高可用的键值存储，保存集群的所有状态数据。
* **Controller Manager:** 负责维护集群状态（如故障检测、自动扩缩容）。
* **Scheduler:** 负责决定将新建的 Pod 分配到哪个节点上运行。

#### 2\. 工作节点 (Worker Node)

* **Kubelet:** 节点上的代理，负责管理容器的生命周期。
* **Kube-proxy:** 维护网络规则，实现 Service 的负载均衡。
* **Pod:** K8s 的最小调度单元，通常包含一个或多个容器。

#### 3\. 网络与存储 (Network & Storage)

* **Ingress:** 管理外部访问集群内部服务的 HTTP/HTTPS 路由。
* **Service:** 定义一组 Pod 的逻辑集合和访问策略（ClusterIP, NodePort, LoadBalancer）。
* **PVC/PV:** 持久化存储系统，将存储资源与 Pod 解耦。

-----

