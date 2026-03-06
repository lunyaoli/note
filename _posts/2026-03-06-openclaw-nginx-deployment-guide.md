---
title: OpenClaw 远程访问与 Nginx 反向代理部署指南
description: >-
  详细介绍 OpenClaw 的安装部署、Nginx 反向代理配置、远程访问设置，以及常见问题的完整解决方案。
author: lunyaoli
date: 2026-03-06 19:25:00 +0800
categories: [OpenClaw, DevOps]
tags: [openclaw, nginx, deployment, remote-access]
---

## 环境信息

| 项目 | 值 |
|------|-----|
| 服务器 | Ubuntu |
| 公网 IP | 116.176.58.172 |
| OpenClaw 端口 | 18789（本地） |
| Nginx 端口 | 11111（公网，原 80 端口已修改） |
| 安装目录 | `~/.openclaw/` |

---

## 一、安装部署

### 1.1 安装 OpenClaw

```bash
# 使用官方安装脚本
curl -fsSL https://get.openclaw.app | sh

# 安装目录
ls ~/.openclaw/
```

### 1.2 创建 systemd 服务

创建文件 `/etc/systemd/system/openclaw-gateway.service`：

```ini
[Unit]
Description=OpenClaw Gateway Service
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root
ExecStart=/root/.openclaw/bin/openclaw gateway
Restart=always
RestartSec=10
StandardOutput=append:/var/log/openclaw-gateway.log
StandardError=append:/var/log/openclaw-gateway.log

[Install]
WantedBy=multi-user.target
```

启用服务：

```bash
systemctl daemon-reload
systemctl enable openclaw-gateway
```

---

## 二、Nginx 反向代理配置

OpenClaw Gateway 默认监听本地端口 18789，通过 Nginx 反向代理实现：

1. 统一外部访问端口
2. 支持 WebSocket 长连接
3. 配置 SSL/HTTPS（可选）
4. 设置访问控制和日志

### 2.1 创建 Nginx 配置

创建配置文件 `/etc/nginx/sites-available/openclaw`：

```nginx
server {
    listen 11111 default_server;
    listen [::]:11111 default_server;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # 标准代理头
        proxy_set_header Host $host;
        proxy_set_header Origin $http_origin;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 长连接超时
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

### 2.2 配置详解

| 配置项 | 说明 | 推荐值 |
|--------|------|--------|
| `listen` | 监听端口 | 11111 (可自定义) |
| `proxy_pass` | 代理目标地址 | `http://127.0.0.1:18789` |
| `proxy_http_version` | HTTP 版本 | 1.1 (WebSocket 需要) |
| `Upgrade` / `Connection` | WebSocket 升级头 | 必须 |
| `proxy_read_timeout` | 读取超时 | 86400s (24小时) |
| `proxy_send_timeout` | 发送超时 | 86400s (24小时) |

### 2.3 启用配置

```bash
# 创建软链接
ln -sf /etc/nginx/sites-available/openclaw /etc/nginx/sites-enabled/

# 测试配置语法
nginx -t

# 重载 Nginx
systemctl reload nginx
```

### 2.4 开放防火墙端口

```bash
# 检查 iptables
iptables -L -n

# 如需开放端口
iptables -I INPUT -p tcp --dport 11111 -j ACCEPT

# 云平台安全组也需要开放 11111 端口
```

---

## 三、服务管理命令

### OpenClaw Gateway 服务

| 操作 | 命令 |
|------|------|
| 启动 | `systemctl start openclaw-gateway` |
| 停止 | `systemctl stop openclaw-gateway` |
| 重启 | `systemctl restart openclaw-gateway` |
| 状态 | `systemctl status openclaw-gateway` |
| 开机自启 | `systemctl enable openclaw-gateway` |
| 查看日志 | `tail -f /var/log/openclaw-gateway.log` |

### Nginx 服务

| 操作 | 命令 |
|------|------|
| 重载配置（不中断） | `systemctl reload nginx` |
| 重启服务（中断） | `systemctl restart nginx` |
| 测试配置 | `nginx -t` |

---

## 四、OpenClaw 配置

### 4.1 基础配置

```bash
# 查看当前配置
openclaw config get gateway

# 设置 Gateway 模式
openclaw config set gateway.mode local
```

### 4.2 远程访问配置（关键）

```bash
# 信任 nginx 反向代理
openclaw config set gateway.trustedProxies '["127.0.0.1", "::1"]'

# 允许的 Origin（公网地址）
openclaw config set gateway.controlUi.allowedOrigins '["http://116.176.58.172:11111"]'

# 允许非安全上下文认证（HTTP）
openclaw config set gateway.controlUi.allowInsecureAuth true

# 禁用设备认证（仅开发环境）
openclaw config set gateway.controlUi.dangerouslyDisableDeviceAuth true
```

### 4.3 一键配置脚本

```bash
# 替换为你的公网IP和端口
PUBLIC_IP="116.176.58.172"
PORT="11111"

openclaw config set gateway.mode local
openclaw config set gateway.trustedProxies '["127.0.0.1", "::1"]'
openclaw config set gateway.controlUi.allowedOrigins "[\"http://${PUBLIC_IP}:${PORT}\"]"
openclaw config set gateway.controlUi.allowInsecureAuth true
openclaw config set gateway.controlUi.dangerouslyDisableDeviceAuth true

# 重启生效
systemctl restart openclaw-gateway
```

---

## 五、验证与访问

### 5.1 检查服务状态

```bash
# 查看 OpenClaw Gateway 端口
ss -tlnp | grep 18789

# 查看 Nginx 端口
ss -tlnp | grep 11111

# 测试本地访问
curl -I http://127.0.0.1:18789

# 测试代理访问
curl -I http://127.0.0.1:11111
```

### 5.2 远程访问

- **地址**: `http://116.176.58.172:11111/`
- **方式**: 直接在浏览器打开上述地址

---

## 六、问题排查

### 6.1 常见错误

#### 错误：origin not allowed

**原因**: Origin 未在 `allowedOrigins` 中配置

**解决**:
```bash
openclaw config set gateway.controlUi.allowedOrigins '["http://你的公网IP:端口"]'
systemctl restart openclaw-gateway
```

#### 错误：device identity required

**原因**: 需要安全上下文（HTTPS）或设备认证

**解决**（开发环境）:
```bash
openclaw config set gateway.controlUi.allowInsecureAuth true
openclaw config set gateway.controlUi.dangerouslyDisableDeviceAuth true
systemctl restart openclaw-gateway
```

#### 问题：外部无法访问但本地可以

**排查清单**:
- [ ] iptables 防火墙: `iptables -L -n`
- [ ] 云平台安全组是否开放端口
- [ ] `trustedProxies` 是否正确配置
- [ ] Nginx 配置是否生效: `nginx -t`

### 6.2 日志排查

```bash
# OpenClaw 日志
tail -100 /var/log/openclaw-gateway.log

# Nginx 日志
tail -100 /var/log/nginx/error.log
tail -100 /var/log/nginx/access.log
```

---

## 七、安全说明

> **警告** 以下配置仅适合开发测试环境：
> - `allowInsecureAuth: true`
> - `dangerouslyDisableDeviceAuth: true`
> - `allowedOrigins: ["*"]`

### 生产环境建议

| 方案 | 说明 |
|------|------|
| Tailscale Serve | 提供 HTTPS 访问，无需公网暴露 |
| SSH 隧道 | 本地端口转发，安全访问 |
| HTTPS + 域名 | 配置 SSL 证书，限制 `allowedOrigins` |

---

## 八、快速速查表

```bash
# 一键启动
systemctl start openclaw-gateway && systemctl start nginx

# 一键重启（配置修改后）
systemctl restart openclaw-gateway && systemctl reload nginx

# 查看所有状态
systemctl status openclaw-gateway
systemctl status nginx
ss -tlnp | grep -E '18789|11111'

# 快速测试
curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:11111
```

---

## 九、配置文件位置汇总

| 文件 | 路径 |
|------|------|
| OpenClaw 安装目录 | `~/.openclaw/` |
| OpenClaw 配置 | `~/.openclaw/config/` |
| systemd 服务 | `/etc/systemd/system/openclaw-gateway.service` |
| Nginx 配置 | `/etc/nginx/sites-available/openclaw` |
| OpenClaw 日志 | `/var/log/openclaw-gateway.log` |
| Nginx 日志 | `/var/log/nginx/` |

---

## 十、WebSocket 连接说明

OpenClaw Gateway 使用 WebSocket 进行实时通信，Nginx 代理配置中必须包含：

```nginx
# WebSocket 升级头
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";

# HTTP/1.1 版本（WebSocket 需要）
proxy_http_version 1.1;

# 长连接超时
proxy_read_timeout 86400s;
proxy_send_timeout 86400s;
```

这些配置确保：
- WebSocket 握手成功
- 长连接不会超时断开
- 实时通信稳定可靠

---

**文档创建时间**：2026-03-06  
**作者**：Claw (OpenClaw Agent)
