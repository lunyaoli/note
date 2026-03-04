---
title: "Ubuntu 22.04 Docker 安装指南"
date: 2026-03-04 16:00:00 +0800
categories: [DevOps, Docker]
tags: [docker, ubuntu, installation, tutorial]
author: Claude Code
---

本文记录了在 Ubuntu 22.04 上安装 Docker 的完整过程及配置。

## 安装环境

- **操作系统**: Ubuntu 22.04.5 LTS (Jammy Jellyfish)
- **安装日期**: 2026-03-04
- **Docker版本**: 28.2.2
- **Docker Compose版本**: 1.29.2

---

## 安装步骤

### 1. 环境准备

清理可能的 apt 锁和冲突进程：

```bash
sudo killall -9 apt-get apt 2>/dev/null
sudo rm -f /var/lib/dpkg/lock-frontend /var/lib/apt/lists/lock /var/cache/apt/archives/lock
```

### 2. 使用 Ubuntu 官方源安装

由于网络环境限制，使用 Ubuntu 官方仓库安装最为稳定：

```bash
sudo apt update
sudo apt install -y docker.io docker-compose
```

### 3. 启动并启用 Docker 服务

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

### 4. 配置国内镜像加速器

创建 daemon 配置文件：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json << 'EOF'
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com"
  ]
}
EOF
```

重启 Docker 使配置生效：

```bash
sudo systemctl restart docker
```

---

## 安装验证

### 检查 Docker 版本

```bash
$ docker --version
Docker version 28.2.2, build 28.2.2-0ubuntu1~22.04.1
```

### 检查 Docker Compose 版本

```bash
$ docker-compose --version
docker-compose version 1.29.2, build unknown
```

### 检查服务状态

```bash
$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Active: active (running) since Wed 2026-03-04 15:48:45 CST
```

---

## 安装结果

| 组件 | 版本 | 状态 |
|------|------|------|
| Docker Engine | 28.2.2 | 已安装 ✓ |
| Docker Compose | 1.29.2 | 已安装 ✓ |
| 服务自启动 | enabled | 已配置 ✓ |
| 国内镜像加速 | 3个源 | 已配置 ✓ |

---

## 常见问题

### 1. 依赖冲突

如果出现 `google-chrome` 等依赖问题，先修复：

```bash
sudo apt --fix-broken install -y
```

### 2. apt 锁被占用

```bash
sudo killall -9 apt-get apt
sudo rm -f /var/lib/dpkg/lock-frontend
```

### 3. 无法拉取镜像

如果网络受限无法连接 Docker Hub，可以：
- 使用已配置的国内镜像源
- 使用离线导入的镜像包：`docker load -i image.tar`
- 配置内部私有镜像仓库

---

## 命令速查

| 命令 | 说明 |
|------|------|
| `docker ps` | 查看运行中的容器 |
| `docker ps -a` | 查看所有容器 |
| `docker images` | 查看本地镜像 |
| `docker pull <image>` | 拉取镜像 |
| `docker run <image>` | 运行容器 |
| `docker stop <id>` | 停止容器 |
| `docker rm <id>` | 删除容器 |
| `docker rmi <id>` | 删除镜像 |
| `docker-compose up -d` | 后台启动服务 |
| `docker-compose down` | 停止并删除服务 |
| `docker-compose logs -f` | 实时查看日志 |

---

## 后续配置

### 使用免 sudo

将当前用户加入 docker 组：

```bash
sudo usermod -aG docker $USER
```

> 注：需要重新登录后生效。

---

*本文档由 Claude Code 自动生成*
