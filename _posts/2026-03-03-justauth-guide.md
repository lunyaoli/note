---
layout: post
title: "JustAuth 实战指南 - 开箱即用的第三方登录组件"
date: 2026-03-03
categories: [Java, Spring Boot]
tags: [OAuth, 第三方登录, Java, Spring Boot, 开源]
---

## 简介

[JustAuth](https://www.justauth.cn/) 是一个开箱即用的第三方登录开源组件，让你脱离繁琐的第三方登录 SDK，让登录变得 So easy!

**项目地址：** [GitHub](https://github.com/justauth/JustAuth) | [Gitee](https://gitee.com/yadong.zhang/JustAuth)

**🏆 荣誉：** Gitee 最有价值开源项目

---

## 一、核心特点

### 1.1 支持 30+ 平台

**国内平台：**
- 微信（开放平台、公众号、企业微信）、QQ、微博
- 支付宝、淘宝、钉钉、飞书
- 百度、华为、小米、抖音、今日头条
- Gitee、Coding、CSDN、OSChina、美团、饿了么、京东

**国外平台：**
- GitHub、Google、Facebook、Twitter
- Microsoft、LinkedIn、Stack Overflow
- GitLab、Amazon、Slack、Line、Pinterest

### 1.2 设计理念

| 特性 | 说明 |
|------|------|
| 🌐 全 | 集成国内外数十家平台，持续扩展中 |
| 🔧 简 | API 极简设计，几行代码完成登录 |
| 🎯 灵 | 高度可定制：缓存、HTTP、OAuth、Scope |
| 📦 轻 | 无侵入式设计，不依赖具体 HTTP 实现 |

---

## 二、原生使用方式

### 2.1 添加依赖

**Maven：**

```xml
<dependency>
    <groupId>me.zhyd.oauth</groupId>
    <artifactId>JustAuth</artifactId>
    <version>1.16.6</version>
</dependency>
```

**Gradle：**

```groovy
implementation 'me.zhyd.oauth:JustAuth:1.16.6'
```

### 2.2 添加 HTTP 工具依赖（任选一种）

```xml
<!-- 推荐：hutool-http -->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-http</artifactId>
    <version>5.8.25</version>
</dependency>

<!-- 或 Apache HttpClient -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.14</version>
</dependency>

<!-- 或 OkHttp -->
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.12.0</version>
</dependency>
```

### 2.3 基础使用示例

```java
// 创建授权 request
AuthRequest authRequest = new AuthGithubRequest(AuthConfig.builder()
    .clientId("your-client-id")
    .clientSecret("your-client-secret")
    .redirectUri("http://localhost:8080/oauth/github/callback")
    .build());

// 生成授权 URL，重定向用户
String authorizeUrl = authRequest.authorize("state");

// 回调处理：用户授权后，用 code 换取用户信息
AuthResponse response = authRequest.login(AuthCallback.builder()
    .code("authorization-code")
    .state("state")
    .build());

if (response.ok()) {
    AuthUser user = (AuthUser) response.getData();
    System.out.println("用户昵称：" + user.getNickname());
    System.out.println("用户头像：" + user.getAvatar());
    System.out.println("来源平台：" + user.getSource());
}
```

### 2.4 动态配置

```java
// 从数据库或配置中心动态获取配置
AuthRequest authRequest = AuthRequestBuilder.builder()
    .source("github")
    .authConfig(source -> {
        // 根据 source 动态查询配置
        OAuthConfig config = configService.getBySource(source);
        return AuthConfig.builder()
            .clientId(config.getClientId())
            .clientSecret(config.getClientSecret())
            .redirectUri(config.getRedirectUri())
            .build();
    })
    .build();
```

---

## 三、Spring Boot 集成（推荐）⭐

使用 `justauth-spring-boot-starter` 实现零代码集成！

### 3.1 添加依赖

```xml
<dependency>
    <groupId>com.xkcoding.justauth</groupId>
    <artifactId>justauth-spring-boot-starter</artifactId>
    <version>1.4.0</version>
</dependency>
```

### 3.2 配置 application.yml

```yaml
justauth:
  enabled: true
  type:
    # GitHub 登录
    GITHUB:
      client-id: your-github-client-id
      client-secret: your-github-client-secret
      redirect-uri: http://localhost:8080/oauth/github/callback
    # Gitee 登录
    GITEE:
      client-id: your-gitee-client-id
      client-secret: your-gitee-client-secret
      redirect-uri: http://localhost:8080/oauth/gitee/callback
    # 微信开放平台
    WECHAT_OPEN:
      client-id: your-wechat-appid
      client-secret: your-wechat-secret
      redirect-uri: http://localhost:8080/oauth/wechat/callback
    # QQ 登录
    QQ:
      client-id: your-qq-appid
      client-secret: your-qq-secret
      redirect-uri: http://localhost:8080/oauth/qq/callback
      union-id: false  # 是否获取 unionId
    # 微博登录
    WEIBO:
      client-id: your-weibo-appkey
      client-secret: your-weibo-secret
      redirect-uri: http://localhost:8080/oauth/weibo/callback
  cache:
    type: default  # 默认内存缓存
```

### 3.3 控制器实现

```java
@Slf4j
@RestController
@RequestMapping("/oauth")
@RequiredArgsConstructor
public class OAuthController {
    
    private final AuthRequestFactory factory;
    
    /**
     * 获取支持的登录平台列表
     */
    @GetMapping
    public List<String> listPlatforms() {
        return factory.oauthList();
    }
    
    /**
     * 发起授权登录
     */
    @GetMapping("/login/{type}")
    public void login(@PathVariable String type, 
                      HttpServletResponse response) throws IOException {
        AuthRequest authRequest = factory.get(type);
        // 生成 state 并重定向
        String state = AuthStateUtils.createState();
        response.sendRedirect(authRequest.authorize(state));
    }
    
    /**
     * 授权回调处理
     */
    @RequestMapping("/{type}/callback")
    public AuthResponse callback(@PathVariable String type, 
                                  AuthCallback callback) {
        AuthRequest authRequest = factory.get(type);
        AuthResponse response = authRequest.login(callback);
        
        if (response.ok()) {
            AuthUser user = (AuthUser) response.getData();
            log.info("登录成功：{} - {}", user.getSource(), user.getNickname());
            // TODO: 保存用户信息到数据库，创建会话等
        }
        
        return response;
    }
}
```

### 3.4 前端使用

```html
<!-- 登录按钮 -->
<a href="/oauth/login/github">GitHub 登录</a>
<a href="/oauth/login/gitee">Gitee 登录</a>
<a href="/oauth/login/qq">QQ 登录</a>
<a href="/oauth/login/weibo">微博登录</a>
```

---

## 四、缓存配置

State 验证需要缓存支持，starter 内置 3 种缓存实现。

### 4.1 默认缓存（内存）

```yaml
justauth:
  cache:
    type: default
```

> ⚠️ 仅适合单机开发环境，多节点部署时 State 会跨机器失效。

### 4.2 Redis 缓存（推荐）

**添加依赖：**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

**配置：**

```yaml
justauth:
  cache:
    type: redis
    prefix: 'MYAPP:OAUTH:STATE:'  # 缓存前缀
    timeout: 5m                   # 过期时间

spring:
  redis:
    host: localhost
    port: 6379
    password: your-password
    timeout: 10000ms
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0
```

### 4.3 自定义缓存

适合使用其他缓存组件（如 Memcached、Caffeine 等）。

```java
// 1. 实现接口
public class MyAuthStateCache implements AuthStateCache {
    
    @Override
    public void cache(String key, String value) {
        // 存入缓存
    }
    
    @Override
    public void cache(String key, String value, long timeout) {
        // 存入缓存（指定过期时间）
    }
    
    @Override
    public String get(String key) {
        // 获取缓存
        return null;
    }
    
    @Override
    public boolean containsKey(String key) {
        // 判断是否存在
        return false;
    }
}

// 2. 注册 Bean
@Configuration
public class AuthStateConfiguration {
    @Bean
    public AuthStateCache authStateCache() {
        return new MyAuthStateCache();
    }
}
```

```yaml
justauth:
  cache:
    type: custom
```

---

## 五、高级特性

### 5.1 自定义 OAuth 平台

接入 JustAuth 未内置的 OAuth 平台。

```java
// 1. 定义平台枚举
public enum ExtendSource implements AuthSource {
    MY_PLATFORM {
        @Override
        public String authorize() {
            return "https://my-platform.com/oauth/authorize";
        }
        
        @Override
        public String accessToken() {
            return "https://my-platform.com/oauth/token";
        }
        
        @Override
        public String userInfo() {
            return "https://my-platform.com/api/user";
        }
    };
}

// 2. 实现请求处理
public class MyPlatformRequest extends AuthDefaultRequest {
    
    public MyPlatformRequest(AuthConfig config) {
        super(config, ExtendSource.MY_PLATFORM);
    }
    
    @Override
    protected AuthToken getAccessToken(AuthCallback callback) {
        // 实现获取 token 逻辑
    }
    
    @Override
    protected AuthUser getUserInfo(AuthToken token) {
        // 实现获取用户信息逻辑
    }
}
```

**配置：**

```yaml
justauth:
  extend:
    enum-class: com.example.oauth.ExtendSource
    config:
      MY_PLATFORM:
        request-class: com.example.oauth.MyPlatformRequest
        client-id: your-client-id
        client-secret: your-client-secret
        redirect-uri: http://localhost:8080/oauth/my-platform/callback
```

### 5.2 代理配置

国外平台（Google、Facebook、Twitter 等）需要代理。

```yaml
justauth:
  http-config:
    timeout: 30000
    proxy:
      GOOGLE:
        type: HTTP
        hostname: 127.0.0.1
        port: 7890
      FACEBOOK:
        type: SOCKS
        hostname: 127.0.0.1
        port: 1080
```

### 5.3 Scope 配置

自定义授权范围，获取更多用户信息。

```yaml
justauth:
  type:
    GITHUB:
      client-id: your-client-id
      client-secret: your-client-secret
      redirect-uri: http://localhost:8080/oauth/github/callback
      scopes:
        - user            # 用户信息
        - repo            # 仓库
        - read:org        # 组织只读
        - user:email      # 用户邮箱
```

---

## 六、实战最佳实践

### 6.1 完整登录流程

```java
@Service
@RequiredArgsConstructor
public class AuthService {
    
    private final AuthRequestFactory authRequestFactory;
    private final UserService userService;
    private final JwtService jwtService;
    
    /**
     * 处理 OAuth 回调
     */
    public LoginResult handleOAuthCallback(String type, AuthCallback callback) {
        // 1. 获取第三方用户信息
        AuthRequest authRequest = authRequestFactory.get(type);
        AuthResponse response = authRequest.login(callback);
        
        if (!response.ok()) {
            throw new OAuthException("授权失败：" + response.getMsg());
        }
        
        AuthUser oauthUser = (AuthUser) response.getData();
        
        // 2. 查找或创建本地用户
        User user = userService.findByOAuth(oauthUser.getSource(), oauthUser.getUuid())
            .orElseGet(() -> userService.createByOAuth(oauthUser));
        
        // 3. 生成 JWT Token
        String token = jwtService.generateToken(user);
        
        return LoginResult.builder()
            .token(token)
            .user(user)
            .isNewUser(user.isNew())
            .build();
    }
}
```

### 6.2 多平台账号绑定

```java
@Service
public class AccountBindingService {
    
    /**
     * 绑定第三方账号
     */
    public void bindOAuth(Long userId, String type, AuthCallback callback) {
        AuthRequest authRequest = authRequestFactory.get(type);
        AuthResponse response = authRequest.login(callback);
        
        if (!response.ok()) {
            throw new OAuthException("授权失败");
        }
        
        AuthUser oauthUser = (AuthUser) response.getData();
        
        // 检查是否已被其他用户绑定
        if (accountBindingRepository.existsByOAuth(oauthUser.getSource(), oauthUser.getUuid())) {
            throw new BusinessException("该账号已被绑定");
        }
        
        // 创建绑定关系
        AccountBinding binding = AccountBinding.builder()
            .userId(userId)
            .platform(oauthUser.getSource())
            .openId(oauthUser.getUuid())
            .nickname(oauthUser.getNickname())
            .avatar(oauthUser.getAvatar())
            .build();
        
        accountBindingRepository.save(binding);
    }
    
    /**
     * 解绑第三方账号
     */
    public void unbindOAuth(Long userId, String platform) {
        accountBindingRepository.deleteByUserIdAndPlatform(userId, platform);
    }
}
```

### 6.3 数据库表设计

```sql
-- 用户表
CREATE TABLE user (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE,
    nickname VARCHAR(100),
    avatar VARCHAR(500),
    email VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 第三方账号绑定表
CREATE TABLE account_binding (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    platform VARCHAR(20) NOT NULL,    -- GITHUB, WECHAT, QQ 等
    open_id VARCHAR(100) NOT NULL,     -- 第三方用户唯一标识
    union_id VARCHAR(100),             -- 微信 unionId
    nickname VARCHAR(100),
    avatar VARCHAR(500),
    access_token VARCHAR(500),         -- 可选：保存 token 用于 API 调用
    refresh_token VARCHAR(500),
    token_expires_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uk_platform_openid (platform, open_id),
    INDEX idx_userid (user_id)
);
```

---

## 七、常见问题

### Q1: State 验证失败？

**原因：** State 过期或跨机器失效。

**解决：** 使用 Redis 缓存，确保多节点共享 State。

### Q2: 微信获取不到 unionId？

**解决：** 配置 `union-id: true`，并在微信开放平台申请对应权限。

### Q3: 支付宝登录失败？

**解决：** 必须配置 `alipay-public-key`（支付宝公钥）。

### Q4: 企业微信登录失败？

**解决：** 必须配置 `agent-id`（网页应用ID）。

### Q5: Google/Facebook 登录超时？

**解决：** 配置 HTTP 代理。

---

## 八、相关资源

| 资源 | 地址 |
|------|------|
| 📖 官方文档 | [https://www.justauth.cn](https://www.justauth.cn) |
| 💻 GitHub | [justauth/JustAuth](https://github.com/justauth/JustAuth) |
| 🛠️ Spring Boot Starter | [justauth-spring-boot-starter](https://github.com/justauth/justauth-spring-boot-starter) |
| 📝 完整示例 | [spring-boot-demo-social](https://github.com/xkcoding/spring-boot-demo/tree/master/demo-social) |
| 💬 QQ群 | 230017570 |
| 💬 微信群 | justauth（备注：justauth 或 ja） |

---

## 总结

JustAuth 是一个**小而全而美**的第三方登录组件：

- ✅ **开箱即用** - 无需深入研究各平台 SDK
- ✅ **平台丰富** - 30+ 平台开箱支持
- ✅ **Spring Boot 集成** - 配置即用，零代码
- ✅ **高度可扩展** - 自定义缓存、OAuth、HTTP
- ✅ **生产可用** - Redis 缓存支持分布式部署

**推荐做法：** Spring Boot 项目直接使用 `justauth-spring-boot-starter`，配置 `application.yml` 即可快速接入！
