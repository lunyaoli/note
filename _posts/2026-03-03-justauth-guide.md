---
layout: post
title: "JustAuth - 开箱即用的第三方登录组件"
date: 2026-03-03
categories: [Java, 开源项目]
tags: [OAuth, 第三方登录, Java, 开源]
---

## 简介

[JustAuth](https://www.justauth.cn/) 是一个开箱即用的第三方登录开源组件，让你脱离繁琐的第三方登录 SDK，让登录变得 So easy!

**项目地址：** [GitHub](https://github.com/justauth/JustAuth) | [Gitee](https://gitee.com/yadong.zhang/JustAuth)

## 核心特点

### 🌐 全 - 支持数十家平台

已集成的第三方平台（国内外常用的基本都已包含）：

**国内平台：**
- Gitee、支付宝、新浪微博、微信、钉钉、百度、QQ、淘宝
- 腾讯云开发者平台、OSChina、Coding、华为、企业微信
- 抖音、今日头条、小米、美团、饿了么、飞书、京东、阿里云、喜马拉雅

**国外平台：**
- GitHub、Google、Facebook、Twitter、StackOverflow
- Pinterest、领英、微软、GitLab、Amazon、Slack、Line

### 🔧 简 - API 设计极简

```java
// 创建授权 request
AuthRequest authRequest = new AuthGiteeRequest(AuthConfig.builder()
    .clientId("clientId")
    .clientSecret("clientSecret")
    .redirectUri("redirectUri")
    .build());

// 生成授权页面
authRequest.authorize("state");

// 授权登录
AuthResponse response = authRequest.login(callback);
```

### 🎯 灵 - 高度可定制

| 特性 | 说明 |
|------|------|
| 自定义 State 缓存 | 支持各种分布式缓存组件（Redis、Memcached 等） |
| 自定义 OAuth 平台 | 更容易适配自有的 OAuth 服务 |
| 自定义 Http 实现 | 不单独依赖某一具体实现，可自由选择 |
| 自定义 Scope | 支持更完善的授权体系 |

## 快速开始

### 1. 添加依赖

**Maven：**

```xml
<dependency>
    <groupId>me.zhyd.oauth</groupId>
    <artifactId>JustAuth</artifactId>
    <version>{latest-version}</version>
</dependency>
```

**Gradle：**

```groovy
implementation 'me.zhyd.oauth:JustAuth:{latest-version}'
```

### 2. 添加 HTTP 工具依赖（任选一种）

```xml
<!-- hutool-http -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-http</artifactId>
    <version>5.7.7</version>
</dependency>

<!-- 或 httpclient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.13</version>
</dependency>

<!-- 或 okhttp -->
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.1</version>
</dependency>
```

### 3. 使用示例

**静态配置：**

```java
AuthRequest authRequest = AuthRequestBuilder.builder()
    .source("github")
    .authConfig(AuthConfig.builder()
        .clientId("clientId")
        .clientSecret("clientSecret")
        .redirectUri("redirectUri")
        .build())
    .build();

// 生成授权 URL
String authorizeUrl = authRequest.authorize("state");

// 授权回调后登录
AuthResponse response = authRequest.login(callback);
```

**动态配置：**

```java
AuthRequest authRequest = AuthRequestBuilder.builder()
    .source("gitee")
    .authConfig((source) -> {
        // 可以从数据库或配置文件中动态获取
        return AuthConfig.builder()
            .clientId("clientId")
            .clientSecret("clientSecret")
            .redirectUri("redirectUri")
            .build();
    })
    .build();
```

## 版本说明

### 分支

| 分支 | 说明 |
|------|------|
| `master` | 稳定版代码，用于生产环境 |
| `dev` | 最新代码，实时发布 SNAPSHOT |

### 版本号

- **稳定版（release）**：格式 `x.x.x`，推荐生产环境使用
- **快照版（snapshots）**：格式 `x.x.x-SNAPSHOT`，仅用于开发尝鲜

> ⚠️ **注意：快照版不可用于生产环境！**

### 使用快照版

```xml
<repositories>
    <repository>
        <id>sonatype-nexus-snapshots</id>
        <name>Sonatype Nexus Snapshots</name>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

## 高级特性

### 自定义 OAuth 平台

```java
AuthRequest authRequest = AuthRequestBuilder.builder()
    .extendSource(AuthExtendSource.values())  // 自定义 AuthSource
    .source("other")
    .authConfig(AuthConfig.builder()
        .clientId("clientId")
        .clientSecret("clientSecret")
        .redirectUri("redirectUri")
        .build())
    .build();
```

### 自定义 State 缓存

支持集成 Redis、Memcached 等分布式缓存，避免 State 跨机器失效问题。

## 相关资源

- 📖 **官方文档：** [https://www.justauth.cn](https://www.justauth.cn)
- 💬 **QQ 群：** 230017570
- 💬 **微信群：** justauth（备注 justauth 或 ja）
- 🔗 **GitHub：** [justauth/JustAuth](https://github.com/justauth/JustAuth)
- 🔗 **Gitee：** [yadong.zhang/JustAuth](https://gitee.com/yadong.zhang/JustAuth)

## 总结

JustAuth 是一个**小而全而美**的第三方登录组件：

- ✅ **开箱即用** - 无需深入研究各平台 SDK
- ✅ **平台丰富** - 国内外数十家平台开箱支持
- ✅ **API 简洁** - 几行代码完成授权登录
- ✅ **高度可扩展** - 支持自定义各种组件

适合任何需要第三方登录的 Java 项目，强烈推荐！
