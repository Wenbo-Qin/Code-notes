# Docker 入门指南

## 什么是 Docker？

Docker 是一个开源平台，可以让开发者和运维人员以容器的形式开发、部署和运行应用程序。容器是一种轻量级的虚拟化技术，可以将应用程序及其依赖项打包在一起。

可以把 Docker 想象成一个「集装箱」系统：
- 应用程序就像货物
- Docker 容器就像标准化的集装箱
- 这样就可以轻松地在任何地方运输和部署应用

## 为什么使用 Docker？

### 一致性
- 在开发、测试和生产环境中运行相同的应用
- 避免「在我机器上能运行」的问题

### 轻量级
- 相比传统虚拟机，容器更加轻便快速
- 启动时间从分钟级缩短到秒级

### 可移植性
- 容器可以在任何支持 Docker 的平台上运行
- 无论是在笔记本电脑、服务器还是云环境

### 效率
- 更高效地利用系统资源
- 单台主机可以运行更多的容器

## 核心概念

### Image（镜像）
镜像是一个只读模板，包含了运行应用程序所需的代码、运行时环境、库和配置文件。你可以把它看作是容器的「蓝图」。

### Container（容器）
容器是镜像的运行实例。可以启动、停止、移动和删除容器。

### Dockerfile
Dockerfile 是一个文本文件，其中包含了一系列指令，用来定义如何构建镜像。

### Registry（仓库）
存储和分发 Docker 镜像的服务，例如 Docker Hub。

## 简单示例

### 编写 Dockerfile

创建一个名为 `Dockerfile` 的文件：

```dockerfile
# 使用官方 Python 运行时作为父镜像
FROM python:3.8-slim

# 设置工作目录
WORKDIR /app

# 复制当前目录内容到工作目录
COPY . /app

# 安装 requirements.txt 中指定的包
RUN pip install --no-cache-dir -r requirements.txt

# 暴露端口 80 给外部
EXPOSE 80

# 定义环境变量
ENV NAME World

# 运行 app.py 时启动应用
CMD ["python", "app.py"]
```

### 常用命令

```bash
# 构建镜像
docker build -t my-app .

# 运行容器
docker run -p 4000:80 my-app

# 查看正在运行的容器
docker ps

# 停止容器
docker stop <container-id>

# 查看所有镜像
docker images

# 删除镜像
docker rmi <image-id>
```

## Docker Compose

当应用需要多个服务（如 Web 应用 + 数据库）时，可以使用 Docker Compose 来管理多容器应用。

示例 `docker-compose.yml`：

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

运行命令：
```bash
docker-compose up
```

## 总结

Docker 彻底改变了软件开发和部署的方式。通过容器化技术，它可以显著提高开发效率、简化部署流程并增强应用程序的可移植性。

对于初学者来说，建议从安装 Docker 开始，然后尝试容器化一个简单的应用，逐步掌握其核心概念和使用技巧。