# Kubernetes (K8s) 入门指南

## 什么是 Kubernetes？

Kubernetes，通常缩写为 K8s，是一个开源的容器编排平台。它可以帮助你自动化部署、扩展和管理容器化的应用程序。

简单来说，如果你有很多个容器（比如 Docker 容器），Kubernetes 可以帮你管理这些容器，确保它们正常运行，并在出现问题时自动修复。

## 为什么需要 Kubernetes？

想象一下，你的应用运行在多个容器中：

- 如果某个容器崩溃了怎么办？
- 如果流量突然增加，需要更多容器来处理请求怎么办？
- 如何更新应用而不中断服务？

Kubernetes 就是为了解决这些问题而设计的。

## 核心概念

### Cluster（集群）
集群是 Kubernetes 的基础，它是一组节点（机器）的集合，用于运行你的应用程序。

### Node（节点）
节点是集群中的单台机器（物理机或虚拟机）。每个节点都有一个或多个 Pod。

### Pod
Pod 是 Kubernetes 中最小的部署单元。一个 Pod 包含一个或多个紧密相关的容器。

### Service（服务）
Service 定义了一组 Pod 的访问方式，提供稳定的网络端点。

### Deployment（部署）
Deployment 负责创建和更新 Pod 实例，确保指定数量的 Pod 副本始终在运行。

## 简单示例

下面是一个简单的 Deployment 示例：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
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

这个例子会创建一个包含 3 个 nginx 容器副本的部署。

## 常用命令

```bash
# 查看集群信息
kubectl cluster-info

# 查看所有 Pod
kubectl get pods

# 查看所有 Deployment
kubectl get deployments

# 创建资源
kubectl apply -f filename.yaml

# 删除资源
kubectl delete -f filename.yaml
```

## 总结

Kubernetes 是现代云原生应用的核心技术之一。虽然学习曲线较陡峭，但它提供了强大的容器管理和编排能力，能帮助你在生产环境中更可靠地运行应用。

开始使用 Kubernetes 最好的方法是在本地搭建一个测试环境，如 Minikube 或 kind。