# Google Kubernetes Engine (GKE) 介绍

## 什么是 GKE？

Google Kubernetes Engine (GKE) 是 Google Cloud 提供的托管 Kubernetes 服务。它让你能够在 Google Cloud 上轻松部署、管理和扩展 Kubernetes 集群，而无需手动设置和维护底层基础设施。

简单来说，GKE 是 Google Cloud 上的 Kubernetes，它消除了运行 Kubernetes 集群的复杂性。

## GKE、Kubernetes 和 Docker 的关系

### Docker 的角色
- Docker 是容器化平台，用于构建和运行容器
- 开发者使用 Docker 将应用程序及其依赖打包成容器镜像
- 这些镜像随后可以在任何支持 Docker 的环境中运行

### Kubernetes 的角色
- Kubernetes 是容器编排工具，负责管理多个容器
- 它提供了部署、扩展、负载均衡、自我修复等功能
- Kubernetes 可以管理由 Docker 创建的容器

### GKE 的角色
- GKE 是托管的 Kubernetes 服务，运行在 Google Cloud 上
- 它自动处理 Kubernetes 控制平面的设置和维护
- 用户只需关注部署应用程序，而不需要担心底层基础设施

下面是三者之间关系的图解：

```
+------------------+     +-------------------+     +------------------+
|   Docker         |     |   Kubernetes      |     |   GKE            |
|                  |     |                   |     |                  |
|  容器化应用       |---->|  容器编排         |---->|  托管 Kubernetes |
|  (创建容器镜像)   |     |  (管理容器)       |     |  服务            |
+------------------+     +-------------------+     +------------------+
```

## GKE 的主要优势

### 全托管服务
- Google 自动管理控制平面（master 节点）
- 自动进行安全更新和补丁管理
- 无需担心集群的维护工作

### 安全性
- 默认启用基于角色的访问控制（RBAC）
- 集成 Google Cloud IAM
- 支持私有集群和授权插件

### 可扩展性
- 自动扩缩容功能（HPA 和 CA）
- 支持垂直和水平 Pod 扩展
- 节点池管理

### 集成性
- 与 Google Cloud 服务深度集成
- 日志和监控集成（Cloud Logging 和 Monitoring）
- 负载均衡、存储等云服务支持

## GKE 工作流程示例

### 1. 构建 Docker 镜像
```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

构建镜像：
```bash
docker build -t gcr.io/my-project/my-app:v1 .
```

### 2. 推送镜像到容器注册表
```bash
docker push gcr.io/my-project/my-app:v1
```

### 3. 在 GKE 中部署应用
创建 deployment.yaml：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: gcr.io/my-project/my-app:v1
        ports:
        - containerPort: 3000
```

部署到 GKE：
```bash
kubectl apply -f deployment.yaml
```

### 4. 暴露服务
创建 service.yaml：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

应用服务配置：
```bash
kubectl apply -f service.yaml
```

## GKE 与其他 Kubernetes 发行版的区别

| 特性 | GKE | 自建 Kubernetes |
|------|-----|----------------|
| 基础设施管理 | Google 管理 | 用户自行管理 |
| 更新和补丁 | 自动处理 | 手动处理 |
| 成本 | 按节点付费 | 基础设施成本 |
| 集成性 | 与 Google Cloud 深度集成 | 需要自己集成云服务 |
| 学习曲线 | 较低 | 较高 |

## 最佳实践

### 镜像管理
- 使用 Google Container Registry (GCR) 或 Artifact Registry
- 为镜像打标签以便版本管理
- 定期扫描镜像漏洞

### 集群管理
- 使用节点池分离不同类型的工作负载
- 启用集群自动扩缩容
- 定期备份 etcd 数据

### 安全性
- 使用命名空间隔离不同环境
- 配置网络策略限制流量
- 启用 Workload Identity 管理服务账户

## 总结

GKE 是连接 Docker 和 Kubernetes 的桥梁，它使得在云环境中运行容器化应用变得更加简单。通过 GKE，你可以专注于应用开发而不是基础设施管理。

Docker 负责容器化应用，Kubernetes 提供编排能力，而 GKE 则提供了托管的 Kubernetes 环境，三者结合形成了完整的容器化解决方案生态系统。

开始使用 GKE 最好的方式是创建一个 Google Cloud 账户，并跟随官方快速入门指南创建第一个集群。