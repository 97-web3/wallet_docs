# API 网关 - 软件需求文档

## 1. 模块概述

### 1.1 用途

API 网关是 Internal-Wallet 系统的统一入口，负责将外部 HTTP 请求路由到内部 gRPC 微服务，同时提供认证、授权、限流、日志等横切关注点的统一处理。

### 1.2 系统架构中的角色

```
客户端 (Web/Mobile/API)
         │
         ▼ HTTP/HTTPS
┌─────────────────────────────────────┐
│           API Gateway               │
│  ┌───────────────────────────────┐  │
│  │        中间件链               │  │
│  │  Recovery → RequestID →       │  │
│  │  Logging → CORS → RateLimit → │  │
│  │  Auth → Router                │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
         │ gRPC
         ▼
┌────────┬────────┬────────┬────────┐
│Business│ChainRPC│ChainSync│ Signer │
│ Service│ Service│ Service │Service │
└────────┴────────┴────────┴────────┘
```

### 1.3 服务依赖

| 依赖 | 类型 | 用途 |
|------|------|------|
| ETCD | 基础设施 | 服务发现 |
| Redis | 基础设施 | 分布式限流 |
| Business Service | gRPC | 业务逻辑 |
| ChainRPC Service | gRPC | 区块链操作 |
| ChainSync Service | gRPC | 区块链同步 |
| Signer Service | gRPC | 交易签名 |

---

## 2. 已实现功能

### 2.1 HTTP 路由转发

#### F-GW-001: YAML 路由配置

**描述**: 基于 YAML 文件的声明式路由配置，支持动态路由映射。

**配置格式**:
```yaml
services:
  business:
    routes:
      - path: /api/v1/auth/login
        method: POST
        service: Business
        rpc: Login
        auth: false
        rate_limit: 5
        rate_limit_type: ip
```

**路由字段说明**:

| 字段 | 类型 | 必填 | 描述 |
|------|------|------|------|
| path | string | 是 | HTTP 路径 |
| method | string | 是 | HTTP 方法 (GET/POST/PUT/DELETE) |
| service | string | 是 | 目标 gRPC 服务名 |
| rpc | string | 是 | gRPC 方法名 |
| auth | bool | 否 | 是否需要认证 (默认 false) |
| rate_limit | int | 否 | 速率限制 (请求/秒) |
| rate_limit_type | string | 否 | 限流类型 (ip/user) |

#### F-GW-002: HTTP 到 gRPC 转换

**描述**: 自动将 HTTP 请求转换为 gRPC 调用。

**转换规则**:
- HTTP JSON Body → Protobuf Message
- HTTP Headers → gRPC Metadata
- HTTP Query Params → Protobuf Message Fields
- gRPC Response → HTTP JSON Response

---

### 2.2 认证中间件

#### F-GW-003: JWT Token 认证

**描述**: 基于 JWT 的无状态认证机制。

**认证流程**:
1. 从 `Authorization: Bearer <token>` 提取 Token
2. 验证 Token 签名和有效期
3. 解析用户信息注入请求上下文
4. 验证失败返回 401 Unauthorized

**请求头格式**:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Token Payload 结构**:
```json
{
  "user_id": "10001",
  "exp": 1702300800,
  "iat": 1702214400
}
```

**配置项**:

| 配置 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| AccessSecret | string | - | JWT 签名密钥 |
| AccessExpire | int | 86400 | Access Token 有效期 (秒) |
| RefreshSecret | string | - | Refresh Token 签名密钥 |
| RefreshExpire | int | 604800 | Refresh Token 有效期 (秒) |

---

### 2.3 速率限制

#### F-GW-004: 全局速率限制

**描述**: 基于令牌桶算法的全局请求限流。

**配置**:
```yaml
RateLimit:
  Enable: true
  GlobalRate: 1000    # 每秒请求数
  GlobalBurst: 2000   # 突发容量
```

#### F-GW-005: IP 级别限流

**描述**: 按客户端 IP 地址进行独立限流。

**实现**:
- 从 `X-Forwarded-For` 或 `X-Real-IP` 获取真实 IP
- 每个 IP 独立计数器
- 支持 Redis 分布式存储

**路由配置示例**:
```yaml
- path: /api/v1/auth/login
  rate_limit: 5
  rate_limit_type: ip
```

#### F-GW-006: 用户级别限流

**描述**: 按认证用户进行独立限流，需要开启认证。

**实现**:
- 从 JWT Token 提取用户 ID
- 每个用户独立计数器
- 需要 Redis 支持

**路由配置示例**:
```yaml
- path: /api/v1/transfer/internal
  auth: true
  rate_limit: 20
  rate_limit_type: user
```

---

### 2.4 CORS 跨域处理

#### F-GW-007: CORS 配置

**描述**: 处理跨域资源共享请求。

**配置**:
```yaml
Cors:
  AllowOrigins:
    - "*"
  AllowMethods:
    - "GET"
    - "POST"
    - "PUT"
    - "DELETE"
    - "PATCH"
    - "OPTIONS"
  AllowHeaders:
    - "Content-Type"
    - "Authorization"
    - "X-Request-ID"
    - "X-Trace-ID"
  ExposeHeaders:
    - "X-Request-ID"
    - "X-Trace-ID"
  AllowCredentials: true
  MaxAge: 86400
```

**响应头**:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

---

### 2.5 直接 API

#### F-GW-008: 验证码服务

**描述**: 图形验证码生成与验证，用于防止机器人攻击。

**API 端点**:

| 端点 | 方法 | 描述 |
|------|------|------|
| `/api/v1/captcha` | GET | 生成验证码 |
| `/api/v1/captcha/verify` | POST | 验证验证码 |

**生成验证码响应**:
```json
{
  "captcha_id": "abc123",
  "captcha_image": "data:image/png;base64,..."
}
```

**验证验证码请求**:
```json
{
  "captcha_id": "abc123",
  "captcha_code": "A1B2"
}
```

#### F-GW-009: 健康检查

**描述**: 服务健康状态检查端点。

**端点**: `GET /health`

**响应**:
```json
{
  "status": "ok",
  "timestamp": "2025-12-10T10:00:00Z"
}
```

---

### 2.6 日志与追踪

#### F-GW-010: 请求日志

**描述**: 记录所有 HTTP 请求的详细日志。

**日志字段**:
- 请求 ID (X-Request-ID)
- 请求方法和路径
- 客户端 IP
- 响应状态码
- 响应时间
- 用户 ID (认证后)

#### F-GW-011: 分布式追踪

**描述**: 集成 Jaeger 分布式追踪。

**追踪 ID 传递**:
- 请求头: `X-Trace-ID`
- gRPC Metadata: `trace-id`

---

### 2.7 异常处理

#### F-GW-012: Panic 恢复

**描述**: 捕获处理过程中的 panic，防止服务崩溃。

**行为**:
- 捕获 panic
- 记录堆栈信息
- 返回 500 Internal Server Error

#### F-GW-013: 统一错误响应

**描述**: 标准化的错误响应格式。

**错误响应格式**:
```json
{
  "code": 401,
  "message": "Unauthorized",
  "request_id": "abc123"
}
```

---

## 3. 技术接口

### 3.1 HTTP 路由列表

#### Business Service 路由

| 方法 | 路径 | gRPC 方法 | 认证 | 限流 |
|------|------|-----------|------|------|
| POST | `/api/v1/auth/register` | Register | 否 | 10/s (IP) |
| POST | `/api/v1/auth/login` | Login | 否 | 5/s (IP) |
| POST | `/api/v1/auth/logout` | Logout | 是 | 10/s |
| POST | `/api/v1/auth/refresh-token` | RefreshToken | 否 | 10/s |
| POST | `/api/v1/transfer/internal` | InternalTransfer | 是 | 20/s (User) |

#### ChainRPC Service 路由

| 方法 | 路径 | gRPC 方法 | 认证 | 限流 |
|------|------|-----------|------|------|
| POST | `/api/v1/chain/address/generate` | GenerateAddress | 是 | 100/s |
| POST | `/api/v1/chain/address/validate` | ValidateAddress | 否 | 200/s |
| POST | `/api/v1/chain/transaction/build/native` | BuildNativeTransfer | 是 | 50/s |
| POST | `/api/v1/chain/transaction/build/token` | BuildTokenTransfer | 是 | 50/s |
| POST | `/api/v1/chain/gas/estimate` | EstimateGas | 是 | 100/s |
| GET | `/api/v1/chain/gas/price` | GetGasPrice | 否 | 200/s |
| POST | `/api/v1/chain/transaction/broadcast` | BroadcastTransaction | 是 | 50/s (User) |
| GET | `/api/v1/chain/balance` | GetBalance | 是 | 100/s |
| GET | `/api/v1/chain/balance/token` | GetTokenBalance | 是 | 100/s |
| GET | `/api/v1/chain/transaction/status` | GetTransactionStatus | 是 | 200/s |
| GET | `/api/v1/chain/block/height` | GetBlockHeight | 否 | 100/s |
| GET | `/api/v1/chain/nonce` | GetNonce | 是 | 100/s |
| GET | `/api/v1/chain/token/info` | GetTokenInfo | 否 | 100/s |
| GET | `/api/v1/chain/health` | HealthCheck | 否 | 50/s |

#### ChainSync Service 路由

| 方法 | 路径 | gRPC 方法 | 认证 | 限流 |
|------|------|-----------|------|------|
| GET | `/api/v1/sync/status` | GetSyncStatus | 是 | 50/s |
| POST | `/api/v1/sync/start` | StartSync | 是 | 10/s |
| POST | `/api/v1/sync/stop` | StopSync | 是 | 10/s |
| POST | `/api/v1/sync/monitor/add` | AddAddressMonitor | 是 | 50/s |
| POST | `/api/v1/sync/monitor/remove` | RemoveAddressMonitor | 是 | 50/s |
| GET | `/api/v1/sync/monitor/list` | GetMonitoredAddresses | 是 | 50/s |
| GET | `/api/v1/sync/transaction` | GetTransaction | 是 | 100/s |
| GET | `/api/v1/sync/transactions/address` | GetAddressTransactions | 是 | 100/s |
| GET | `/api/v1/sync/balance/address` | GetAddressBalance | 是 | 100/s |
| POST | `/api/v1/sync/balance/multi` | GetMultiAddressBalance | 是 | 50/s |

---

## 4. 依赖

### 4.1 外部库

| 库 | 版本 | 用途 |
|----|------|------|
| github.com/zeromicro/go-zero | 1.6.0 | 微服务框架 |
| google.golang.org/grpc | 1.75.0 | gRPC 客户端 |
| github.com/golang-jwt/jwt/v4 | 4.5.1 | JWT 处理 |
| github.com/mojocn/base64Captcha | 1.3.8 | 验证码生成 |
| github.com/redis/go-redis/v9 | 9.17.0 | Redis 客户端 |

### 4.2 基础设施要求

| 组件 | 要求 | 用途 |
|------|------|------|
| ETCD | 3.5+ | 服务发现 |
| Redis | 7.x | 分布式限流 |

---

## 5. 配置

### 5.1 完整配置文件

**文件**: `api-gateway/etc/gateway.yaml`

```yaml
Name: api-gateway
Mode: dev

# HTTP 服务配置
Host: 0.0.0.0
Port: 8080

# 日志配置
Log:
  ServiceName: api-gateway
  Mode: file
  Path: logs/gateway
  Level: info
  Compress: true
  KeepDays: 7
  StackCooldownMillis: 100

# JWT 认证配置
Auth:
  AccessSecret: "your-secret-key-change-me-in-production"
  AccessExpire: 86400
  RefreshSecret: "your-refresh-secret-change-me-in-production"
  RefreshExpire: 604800

# RPC 服务发现
AccountRpc:
  Etcd:
    Hosts:
      - localhost:2379
    Key: account.rpc
  Timeout: 30000

# Redis 配置
Redis:
  Host: localhost
  Port: 6379
  Password: redis123456
  DB: 0

# 限流配置
RateLimit:
  Enable: true
  UseRedis: true
  GlobalRate: 1000
  GlobalBurst: 2000

# 请求超时
RequestTimeout: 30000

# CORS 配置
Cors:
  AllowOrigins:
    - "*"
  AllowMethods:
    - "GET"
    - "POST"
    - "PUT"
    - "DELETE"
    - "PATCH"
    - "OPTIONS"
  AllowHeaders:
    - "Content-Type"
    - "Authorization"
    - "X-Request-ID"
    - "X-Trace-ID"
  ExposeHeaders:
    - "X-Request-ID"
    - "X-Trace-ID"
  AllowCredentials: true
  MaxAge: 86400
```

### 5.2 环境变量

| 变量 | 描述 | 默认值 |
|------|------|--------|
| JWT_ACCESS_SECRET | JWT 签名密钥 | 配置文件值 |
| REDIS_HOST | Redis 地址 | localhost |
| REDIS_PORT | Redis 端口 | 6379 |
| REDIS_PASSWORD | Redis 密码 | - |

---

## 6. 安全考量

### 6.1 已知问题摘要

| 问题 ID | 级别 | 描述 | 参考 |
|---------|------|------|------|
| CRITICAL-02 | 严重 | 硬编码的加密密码和基础设施密钥 | `docs/security-issues/CRITICAL-02-*.md` |
| HIGH-02 | 高危 | JWT 密钥未强制执行或验证 | `docs/security-issues/HIGH-02-*.md` |
| MEDIUM-01 | 中危 | CORS 通配符配合凭证 | `docs/security-issues/MEDIUM-01-*.md` |
| MEDIUM-02 | 中危 | 基于 IP 的速率限制依赖不受信任的标头 | `docs/security-issues/MEDIUM-02-*.md` |

### 6.2 安全建议

1. **生产环境必须更换 JWT 密钥**: 使用强随机密钥替换默认配置
2. **限制 CORS 源**: 生产环境不应使用 `*`，应配置具体域名
3. **使用 HTTPS**: 生产环境必须启用 TLS
4. **配置反向代理**: 在可信反向代理后获取真实 IP

---

## 7. 中间件链顺序

```
请求进入
    │
    ▼
┌───────────────┐
│   Recovery    │  ← Panic 恢复
└───────────────┘
    │
    ▼
┌───────────────┐
│  Request ID   │  ← 生成/提取请求 ID
└───────────────┘
    │
    ▼
┌───────────────┐
│   Logging     │  ← 请求日志
└───────────────┘
    │
    ▼
┌───────────────┐
│     CORS      │  ← 跨域处理
└───────────────┘
    │
    ▼
┌───────────────┐
│  Rate Limit   │  ← 速率限制
└───────────────┘
    │
    ▼
┌───────────────┐
│     Auth      │  ← JWT 认证 (可选)
└───────────────┘
    │
    ▼
┌───────────────┐
│    Router     │  ← 路由匹配
└───────────────┘
    │
    ▼
┌───────────────┐
│  gRPC Call    │  ← 调用后端服务
└───────────────┘
    │
    ▼
响应返回
```
