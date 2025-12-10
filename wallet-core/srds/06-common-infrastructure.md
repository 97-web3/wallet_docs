# 通用库与基础设施 - 软件需求文档

## 1. 模块概述

### 1.1 用途

通用库与基础设施模块提供系统所有服务共享的基础功能，包括中间件、工具函数、数据模型、缓存封装、常量定义、错误码、数据访问层等。此外，本文档还涵盖系统运行所需的基础设施组件。

### 1.2 系统架构中的角色

```
┌─────────────────────────────────────────────────────────────────┐
│                         服务层                                   │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────┐        │
│  │ Business  │ │ ChainRPC  │ │ ChainSync │ │  Signer   │        │
│  │  Service  │ │  Service  │ │  Service  │ │  Service  │        │
│  └─────┬─────┘ └─────┬─────┘ └─────┬─────┘ └─────┬─────┘        │
│        │             │             │             │               │
└────────┼─────────────┼─────────────┼─────────────┼───────────────┘
         │             │             │             │
         └─────────────┴──────┬──────┴─────────────┘
                              │
┌─────────────────────────────┼────────────────────────────────────┐
│                        common/ 模块                               │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │                    middleware/                            │   │
│  │  ┌─────────┐ ┌──────┐ ┌───────────┐ ┌──────────┐ ┌─────┐ │   │
│  │  │  Auth   │ │ CORS │ │ RateLimit │ │ Recovery │ │ Log │ │   │
│  │  └─────────┘ └──────┘ └───────────┘ └──────────┘ └─────┘ │   │
│  └───────────────────────────────────────────────────────────┘   │
│  ┌───────────────────────────────────────────────────────────┐   │
│  │                      utils/                               │   │
│  │  ┌────────┐ ┌──────────────┐ ┌───────────┐ ┌───────────┐  │   │
│  │  │ Crypto │ │ CryptoSigner │ │ Snowflake │ │ Validator │  │   │
│  │  └────────┘ └──────────────┘ └───────────┘ └───────────┘  │   │
│  └───────────────────────────────────────────────────────────┘   │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────────┐    │
│  │  model/   │ │  cache/   │ │ constants/│ │  repository/  │    │
│  └───────────┘ └───────────┘ └───────────┘ └───────────────┘    │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌───────────────┐    │
│  │ errcode/  │ │    db/    │ │   code/   │ │ interceptor/  │    │
│  └───────────┘ └───────────┘ └───────────┘ └───────────────┘    │
└──────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────┼────────────────────────────────────┐
│                        基础设施层                                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
│  │ MariaDB  │ │  Redis   │ │  Kafka   │ │   ETCD   │ │ Jaeger │ │
│  │  10.11   │ │    7     │ │   7.5    │ │  3.5.9   │ │        │ │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └────────┘ │
│  ┌──────────────────────────┐ ┌──────────────────────────────┐   │
│  │       Prometheus         │ │          Grafana             │   │
│  └──────────────────────────┘ └──────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────┘
```

### 1.3 服务依赖

| 依赖项 | 版本 | 用途 |
|--------|------|------|
| github.com/zeromicro/go-zero | v1.6.0 | 微服务框架 |
| gorm.io/gorm | latest | ORM 框架 |
| github.com/redis/go-redis/v9 | v9 | Redis 客户端 |
| github.com/golang-jwt/jwt/v4 | v4 | JWT 处理 |
| golang.org/x/crypto/pbkdf2 | latest | 密钥派生 |
| golang.org/x/time/rate | latest | 限流 |

---

## 2. 已实现功能

### 2.1 中间件 (middleware/)

#### F-MW-001: JWT 认证中间件

- **描述**: 基于 JWT 的用户身份认证
- **文件**: `common/middleware/auth.go`
- **功能**:
  - 从 Authorization Header 提取 Bearer Token
  - 验证 JWT 签名和有效期
  - 将用户信息 (user_id, email) 注入 Context
  - 支持可选认证模式 (OptionalJWTAuth)

**JWT Claims 结构**:
```go
type JWTClaims struct {
    UserID string `json:"user_id"`
    Email  string `json:"email"`
    jwt.RegisteredClaims
}
```

**使用方式**:
```go
// 必须认证
router.Use(middleware.JWTAuth(jwtSecret))

// 可选认证
router.Use(middleware.OptionalJWTAuth(jwtSecret))

// 从 Context 获取用户信息
userID := middleware.GetUserID(ctx)
email := middleware.GetEmail(ctx)
```

#### F-MW-002: CORS 跨域中间件

- **描述**: 处理跨域资源共享请求
- **文件**: `common/middleware/cors.go`
- **功能**:
  - 配置允许的 Origin、Methods、Headers
  - 处理 OPTIONS 预检请求
  - 支持通配符域名匹配
  - 可配置凭证和缓存时间

**默认配置**:
```go
CORSConfig{
    AllowOrigins:     []string{"*"},
    AllowMethods:     []string{"GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"},
    AllowHeaders:     []string{"Content-Type", "Authorization", "X-Request-ID", "X-Trace-ID"},
    ExposeHeaders:    []string{"X-Request-ID", "X-Trace-ID"},
    AllowCredentials: true,
    MaxAge:           86400,  // 24小时
}
```

#### F-MW-003: 全局速率限制

- **描述**: 基于令牌桶算法的全局限流
- **文件**: `common/middleware/ratelimit.go`
- **功能**:
  - 控制系统整体请求速率
  - 可配置每秒请求数和突发容量
  - 返回 429 Too Many Requests

**使用方式**:
```go
// 每秒 1000 个请求，突发 2000
router.Use(middleware.RateLimit(1000, 2000))
```

#### F-MW-004: IP 级速率限制

- **描述**: 基于客户端 IP 的限流
- **文件**: `common/middleware/ratelimit.go`
- **功能**:
  - 每个 IP 独立的限流器
  - 自动清理不活跃的限流器
  - 支持 X-Forwarded-For 解析

**使用方式**:
```go
ipLimiter := middleware.NewIPRateLimiter(100, 200, time.Hour)
router.Use(ipLimiter.Middleware())
```

#### F-MW-005: 用户级速率限制

- **描述**: 基于用户 ID 的限流（需在 JWT 中间件之后使用）
- **文件**: `common/middleware/ratelimit.go`
- **功能**:
  - 每个用户独立的限流器
  - 未认证请求跳过限流
  - 防止单用户滥用

**使用方式**:
```go
userLimiter := middleware.NewUserRateLimiter(50, 100)
router.Use(userLimiter.Middleware())
```

#### F-MW-006: 异常恢复中间件

- **描述**: 捕获 Panic 并返回友好错误
- **文件**: `common/middleware/recovery.go`
- **功能**:
  - 捕获所有 Panic
  - 记录堆栈信息到日志
  - 返回 500 Internal Server Error
  - 支持自定义处理器

**日志记录字段**:
- error: Panic 信息
- stack: 堆栈追踪
- method: HTTP 方法
- path: 请求路径
- ip: 客户端 IP

#### F-MW-007: 请求日志中间件

- **描述**: 记录 HTTP 请求日志
- **文件**: `common/middleware/logging.go`
- **功能**:
  - 基础日志: 方法、路径、状态码、耗时
  - 详细日志: 包含请求头、User-Agent、Referer
  - 访问日志: Nginx 风格格式

**日志字段**:
```json
{
    "method": "POST",
    "path": "/api/v1/login",
    "query": "redirect=/dashboard",
    "status": 200,
    "duration": 45,
    "size": 1024,
    "ip": "192.168.1.1",
    "user_agent": "Mozilla/5.0...",
    "user_id": "123456789"
}
```

---

### 2.2 工具函数 (utils/)

#### F-UTIL-001: 通用加密哈希

- **描述**: MD5 和 SHA256 哈希工具
- **文件**: `common/utils/crypto.go`
- **功能**:
  - MD5 哈希
  - SHA256 哈希
  - 密码哈希 (SHA256 + Salt)
  - 密码验证

**函数签名**:
```go
func MD5Hash(data string) string
func SHA256Hash(data string) string
func HashPassword(password, salt string) string
func VerifyPassword(password, salt, hashedPassword string) bool
```

#### F-UTIL-002: 种子加密工具

- **描述**: HD 钱包种子的安全加密/解密
- **文件**: `common/utils/crypto_signer.go`
- **功能**:
  - AES-256-GCM 加密
  - PBKDF2 密钥派生 (100,000 次迭代)
  - 32 字节随机盐值
  - SHA256 完整性校验

**加密流程**:
1. 生成 32 字节随机盐值
2. 使用 PBKDF2 从密码派生 32 字节密钥
3. 创建 AES-256-GCM 加密器
4. 生成随机 Nonce
5. 加密数据
6. 计算原始种子的 SHA256 哈希

**加密结果结构**:
```go
type EncryptSeedResult struct {
    Encrypted []byte // 加密后的数据
    Salt      []byte // 盐值
    IV        []byte // IV/Nonce
    Hash      string // SHA256哈希
}
```

**函数签名**:
```go
func EncryptSeed(seed []byte, password string) (*EncryptSeedResult, error)
func DecryptSeed(encrypted []byte, salt []byte, password string) ([]byte, error)
func HashSeed(seed []byte) string
func VerifySeedHash(seed []byte, expectedHash string) bool
func GenerateRandomSeed(seedLength int) ([]byte, error)
func GenerateStrongPassword(length int) (string, error)
```

#### F-UTIL-003: 雪花 ID 生成器

- **描述**: 分布式唯一 ID 生成
- **文件**: `common/utils/snowflake.go`
- **功能**:
  - 41 位时间戳 + 10 位机器 ID + 12 位序列号
  - 支持 1024 台机器
  - 每毫秒最多 4096 个 ID
  - 时钟回拨检测

**ID 结构** (63 位):
```
┌─────────────────────────────────────────────────────────────────┐
│ 1 位符号 │    41 位时间戳     │  10 位机器ID │   12 位序列号    │
└─────────────────────────────────────────────────────────────────┘
```

**使用方式**:
```go
// 服务启动时初始化 (每个服务使用不同的 machineID)
utils.Init(1)  // business-rpc: 1

// 生成 ID
id := utils.GenerateID()           // int64
idStr := utils.GenerateIDString()  // string (推荐，避免 JS 精度问题)
```

**推荐的机器 ID 分配**:
| 服务 | Machine ID |
|------|------------|
| business-rpc | 1 |
| chainrpc-rpc | 2 |
| chainsync-rpc | 3 |
| signer-rpc | 4 |

#### F-UTIL-004: 验证码生成器

- **描述**: 验证码生成、存储和验证
- **文件**: `common/code/code.go`
- **功能**:
  - 生成随机数字验证码
  - 存储到 Redis 并设置过期时间
  - 验证用户输入
  - 验证后删除（一次性使用）

**Redis Key 格式**:
```
{codeType}:code:{recipient}:{scene}
例如: email:code:user@example.com:register
```

**支持的场景**:
- register: 注册
- login: 登录
- reset_password: 重置密码
- withdraw: 提现

---

### 2.3 数据模型 (model/)

#### F-MODEL-001: 基础模型

- **描述**: 所有业务模型的基类
- **文件**: `common/model/base_model.go`
- **功能**:
  - 雪花 ID 主键
  - 自动创建/更新时间
  - 软删除标记
  - 乐观锁版本号
  - GORM 钩子自动生成 ID

**BaseModel 结构**:
```go
type BaseModel struct {
    ID          int64     `gorm:"column:id;primaryKey;autoIncrement:false" json:"id"`
    CreatedTime time.Time `gorm:"column:created_time;autoCreateTime" json:"created_time"`
    UpdatedTime time.Time `gorm:"column:updated_time;autoUpdateTime" json:"updated_time"`
    Deleted     int       `gorm:"column:deleted;default:0;index" json:"deleted"`
    Version     int       `gorm:"column:version;default:1" json:"version"`
}
```

**BaseModelWithUser 结构** (带用户信息):
```go
type BaseModelWithUser struct {
    BaseModel
    CreatedUserID int64  `gorm:"column:created_user_id" json:"created_user_id"`
    UpdatedUserID *int64 `gorm:"column:updated_user_id" json:"updated_user_id,omitempty"`
}
```

---

### 2.4 缓存封装 (cache/)

#### F-CACHE-001: Redis 缓存封装

- **描述**: 基于 go-redis/v9 的全功能缓存封装
- **文件**: `common/cache/redis_cache.go`
- **功能**:
  - 键前缀管理
  - 字符串操作 (Set/Get/Delete/Exists)
  - JSON 序列化存储
  - 原子计数 (Incr/Decr)
  - Hash 操作
  - List 操作
  - Set 操作
  - Sorted Set 操作
  - 分布式锁

**分布式锁**:
```go
// 获取锁
unlock, err := cache.Lock(ctx, "resource_key", time.Second*30)
if err != nil {
    // 获取锁失败
}
defer unlock()  // 释放锁

// 带重试的锁
unlock, err := cache.LockWithRetry(ctx, "resource_key", time.Second*30, 3, time.Millisecond*100)
```

**支持的操作**:

| 类别 | 方法 |
|------|------|
| 字符串 | Set, Get, SetJSON, GetJSON, Delete, Exists, Expire, TTL |
| 计数 | Incr, IncrBy, Decr, DecrBy |
| Hash | HSet, HGet, HGetAll, HDel, HExists |
| List | LPush, RPush, LPop, RPop, LRange, LLen |
| Set | SAdd, SRem, SMembers, SIsMember, SCard |
| Sorted Set | ZAdd, ZRem, ZRange, ZRevRange, ZRangeByScore, ZCard, ZScore |
| 分布式锁 | Lock, LockWithRetry, SetNX |

---

### 2.5 常量定义 (constants/)

#### F-CONST-001: 业务状态常量

- **文件**: `common/constants/constants.go`
- **包含**:
  - 用户状态 (正常/冻结/禁用)
  - 账户类型 (现货/合约)
  - 订单方向 (买入/卖出)
  - 订单类型 (限价/市价)
  - 订单状态 (待成交/部分成交/完全成交/已撤销)
  - 充值状态 (待确认/已确认/失败)
  - 提现状态 (待审核/处理中/已完成/已拒绝)
  - 转账状态 (成功/失败)
  - 风险等级 (低/中/高/严重)
  - Kafka Topic 名称
  - 通用错误码

**错误码范围**:
| 范围 | 用途 |
|------|------|
| 0 | 成功 |
| 10001-10999 | 通用错误 |
| 20001-20999 | 业务错误 |
| 30001-30999 | 风控错误 |

#### F-CONST-002: Redis Key 常量

- **文件**: `common/constants/redis.go`
- **包含**:
  - 行情 Ticker Hash
  - 深度数据 Hash
  - 分时数据前缀
  - K线数据前缀

---

### 2.6 错误码定义 (errcode/)

#### F-ERR-001: 签名服务错误码

- **文件**: `common/errcode/signer.go`
- **范围**: 2000-2999

**通用错误 (2000-2099)**:
| 错误码 | 名称 | 说明 |
|--------|------|------|
| 0 | SignerSuccess | 成功 |
| 2001 | SignerInvalidParams | 参数错误 |
| 2002 | SignerInternalError | 内部错误 |
| 2003 | SignerDatabaseError | 数据库错误 |
| 2004 | SignerUnauthorized | 未授权 |
| 2005 | SignerRateLimitError | 请求限流 |
| 2006 | SignerServiceUnavail | 服务不可用 |

**Seed 错误 (2100-2199)**:
| 错误码 | 名称 | 说明 |
|--------|------|------|
| 2101 | SignerSeedNotFound | Seed不存在 |
| 2102 | SignerSeedAlreadyExist | Seed已存在 |
| 2103 | SignerSeedDecryptFail | Seed解密失败 |
| 2104 | SignerSeedInactive | Seed未激活 |
| 2105 | SignerSeedEncryptFail | Seed加密失败 |
| 2106 | SignerSeedHashMismatch | Seed哈希校验失败 |
| 2107 | SignerSeedInvalidPassword | Seed密码错误 |
| 2108 | SignerSeedDeprecated | Seed已废弃 |

**公钥导出错误 (2200-2299)**:
| 错误码 | 名称 | 说明 |
|--------|------|------|
| 2201 | SignerInvalidPath | 派生路径无效 |
| 2202 | SignerPubkeyExportFail | 公钥导出失败 |
| 2203 | SignerChainNotSupport | 链不支持 |
| 2204 | SignerInvalidIndex | 索引无效 |
| 2205 | SignerBatchTooLarge | 批量数量过大 |
| 2206 | SignerDerivationFail | HD派生失败 |

**签名错误 (2300-2399)**:
| 错误码 | 名称 | 说明 |
|--------|------|------|
| 2301 | SignerSignatureFailed | 签名失败 |
| 2302 | SignerDuplicateRequest | 重复请求 |
| 2303 | SignerInvalidTxData | 交易数据无效 |
| 2304 | SignerAmountExceedLimit | 金额超过限额 |
| 2305 | SignerApprovalRequired | 需要审批 |
| 2306 | SignerInvalidSignature | 签名验证失败 |
| 2307 | SignerPrivateKeyNotFound | 私钥不存在 |
| 2308 | SignerInvalidAddress | 地址无效 |

---

### 2.7 数据访问层 (repository/)

#### F-REPO-001: 泛型基础 Repository

- **描述**: 基于 GORM 的泛型数据访问层
- **文件**: `common/repository/base_repository.go`
- **功能**:
  - 泛型 CRUD 操作
  - 软删除支持
  - 乐观锁版本控制
  - 分页查询
  - 事务支持

**接口定义**:
```go
type BaseRepository[T any] interface {
    // 基础 CRUD
    Create(ctx context.Context, entity *T) error
    CreateMultiple(ctx context.Context, entities []*T) error
    FindByID(ctx context.Context, id int64) (*T, error)
    Update(ctx context.Context, entity *T) error
    UpdateFields(ctx context.Context, id int64, fields map[string]interface{}) error
    Delete(ctx context.Context, id int64) error
    SoftDelete(ctx context.Context, id int64) error

    // 查询
    FindAll(ctx context.Context) ([]*T, error)
    FindByCondition(ctx context.Context, condition map[string]interface{}) ([]*T, error)
    Count(ctx context.Context, condition map[string]interface{}) (int64, error)
    Exists(ctx context.Context, id int64) (bool, error)

    // 分页
    FindWithPagination(ctx context.Context, page, pageSize int, condition map[string]interface{}) ([]*T, int64, error)

    // 事务支持
    WithTx(tx *gorm.DB) BaseRepository[T]
    GetDB() *gorm.DB
}
```

**使用方式**:
```go
// 创建 Repository
userRepo := repository.NewBaseRepository[User](db)

// CRUD 操作
user, err := userRepo.FindByID(ctx, 123)
err := userRepo.Create(ctx, &user)
err := userRepo.SoftDelete(ctx, 123)

// 分页查询
users, total, err := userRepo.FindWithPagination(ctx, 1, 10, map[string]interface{}{
    "status = ?": 1,
})

// 事务
tx := db.Begin()
txRepo := userRepo.WithTx(tx)
// ... 操作
tx.Commit()
```

---

### 2.8 数据库连接 (db/)

#### F-DB-001: GORM 初始化

- **描述**: GORM 数据库连接初始化
- **文件**: `common/db/gorm.go`
- **功能**:
  - MySQL/MariaDB 连接
  - 连接池配置
  - 慢查询日志
  - 自定义日志集成 (go-zero logx)

**配置项**:
| 参数 | 说明 | 默认值 |
|------|------|--------|
| Host | 数据库主机 | - |
| Port | 端口 | 3306 |
| Username | 用户名 | - |
| Password | 密码 | - |
| Database | 数据库名 | - |
| MaxIdleConns | 最大空闲连接 | - |
| MaxOpenConns | 最大打开连接 | - |
| ConnMaxLifetime | 连接最大生命周期 (秒) | - |
| SlowThreshold | 慢查询阈值 (毫秒) | 0 |
| LogLevel | 日志级别 | 1 |

---

## 3. 基础设施

### 3.1 Docker Compose 配置

**文件**: `deploy/docker/docker-compose.yml`

#### 3.1.1 MariaDB

- **镜像**: mariadb:10.11
- **端口**: 3306
- **功能**: 主数据库
- **配置**:
  - 数据库: crypto_exchange
  - 字符集: utf8mb4
  - 时区: Asia/Shanghai
  - 数据持久化: mariadb_data volume

#### 3.1.2 Redis

- **镜像**: redis:7-alpine
- **端口**: 6379
- **功能**: 缓存、速率限制、验证码存储
- **配置**:
  - AOF 持久化
  - 密码保护
  - 数据持久化: redis_data volume

#### 3.1.3 Kafka

- **镜像**: confluentinc/cp-kafka:7.5.0
- **端口**: 9092 (内部), 9093 (外部)
- **功能**: 异步消息队列
- **依赖**: Zookeeper
- **配置**:
  - 自动创建 Topic
  - 单副本模式

#### 3.1.4 Zookeeper

- **镜像**: confluentinc/cp-zookeeper:7.5.0
- **端口**: 2181
- **功能**: Kafka 协调服务

#### 3.1.5 ETCD

- **镜像**: quay.io/coreos/etcd:v3.5.9
- **端口**: 2379 (客户端), 2380 (对等)
- **功能**: 服务发现、配置中心
- **配置**:
  - 单节点模式
  - 数据持久化: etcd_data volume

#### 3.1.6 Prometheus

- **镜像**: prom/prometheus:latest
- **端口**: 9090
- **功能**: 指标收集和存储
- **配置**:
  - 配置文件: prometheus/prometheus.yml
  - 数据持久化: prometheus_data volume

#### 3.1.7 Grafana

- **镜像**: grafana/grafana:latest
- **端口**: 3000
- **功能**: 监控仪表板
- **配置**:
  - 禁用用户注册
  - 数据持久化: grafana_data volume

#### 3.1.8 Jaeger

- **镜像**: jaegertracing/all-in-one:latest
- **端口**:
  - 16686: Web UI
  - 6831/6832: Agent (UDP)
  - 14268: Collector HTTP
  - 14250: Collector gRPC
- **功能**: 分布式链路追踪

### 3.2 网络配置

所有服务通过 `crypto-network` 桥接网络互联。

### 3.3 数据卷

| Volume | 用途 |
|--------|------|
| mariadb_data | MariaDB 数据 |
| redis_data | Redis 持久化数据 |
| zookeeper_data | Zookeeper 数据 |
| zookeeper_logs | Zookeeper 日志 |
| kafka_data | Kafka 数据 |
| prometheus_data | Prometheus 指标数据 |
| grafana_data | Grafana 配置和仪表板 |
| etcd_data | ETCD 数据 |

---

## 4. 配置文件

### 4.1 数据库配置示例

```yaml
MySQL:
  Host: localhost
  Port: 3306
  Username: root
  Password: root123456
  Database: crypto_exchange
  MaxIdleConns: 10
  MaxOpenConns: 100
  ConnMaxLifetime: 3600
  SlowThreshold: 200
  LogLevel: 2
```

### 4.2 Redis 配置示例

```yaml
Redis:
  Host: localhost:6379
  Password: redis123456
  DB: 0
```

### 4.3 ETCD 配置示例

```yaml
Etcd:
  Hosts:
    - localhost:2379
  Key: business.rpc
```

---

## 5. 目录结构

```
common/
├── cache/
│   └── redis_cache.go        # Redis 缓存封装
├── captcha/
│   └── captcha.go            # 图形验证码
├── code/
│   └── code.go               # 验证码生成器
├── constants/
│   ├── constants.go          # 业务常量
│   ├── redis.go              # Redis Key 常量
│   └── signer.go             # 签名服务常量
├── db/
│   ├── config.go             # 数据库配置
│   └── gorm.go               # GORM 初始化
├── enum/
│   └── accountType.go        # 账户类型枚举
├── errcode/
│   └── signer.go             # 签名服务错误码
├── interceptor/
│   ├── example_usage.go      # 拦截器示例
│   └── metadata.go           # gRPC 元数据拦截器
├── middleware/
│   ├── auth.go               # JWT 认证
│   ├── common.go             # 通用工具函数
│   ├── cors.go               # CORS 中间件
│   ├── logging.go            # 日志中间件
│   ├── metadata.go           # 元数据中间件
│   ├── ratelimit.go          # 限流中间件
│   ├── recovery.go           # 异常恢复
│   └── redis_ratelimit.go    # Redis 限流
├── model/
│   └── base_model.go         # 基础模型
├── models/
│   └── wallet.go             # 钱包模型
├── repository/
│   └── base_repository.go    # 泛型 Repository
└── utils/
    ├── config_finder.go      # 配置文件查找
    ├── crypto.go             # 通用加密
    ├── crypto_signer.go      # 种子加密
    ├── response.go           # HTTP 响应工具
    ├── snowflake.go          # 雪花 ID
    ├── tracing.go            # 链路追踪
    └── validator.go          # 输入验证
```

---

## 6. 安全考量

### 6.1 已知安全问题

> 以下安全问题引用自 `docs/SECURITY_ISSUES.md`

#### CRITICAL-02: 硬编码的加密密码

- **位置**: `common/utils/crypto_signer.go` (概念上)
- **描述**: 如果密码在代码或配置中硬编码，种子加密将失去意义
- **影响**: 加密种子可被轻易解密
- **建议**:
  - 使用 HSM 或密钥管理服务存储密码
  - 实施密码轮换机制

#### HIGH-01: 弱用户密码哈希

- **位置**: `common/utils/crypto.go`
- **描述**: 使用 SHA256 + Salt 而非 bcrypt/argon2
- **影响**: 密码哈希易受暴力破解
- **建议**: 迁移到 bcrypt 或 argon2id

**当前实现**:
```go
func HashPassword(password, salt string) string {
    return SHA256Hash(password + salt)
}
```

**建议实现**:
```go
func HashPassword(password string) (string, error) {
    return bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
}
```

#### HIGH-02: JWT 密钥未强制执行

- **位置**: `common/middleware/auth.go`
- **描述**: JWT 密钥依赖配置，可能使用弱密钥
- **影响**: 弱密钥可被暴力破解，导致 Token 伪造
- **建议**:
  - 强制最小密钥长度 (32 字节)
  - 启动时验证密钥强度

#### MEDIUM-03: Docker 暴露弱凭证

- **位置**: `deploy/docker/docker-compose.yml`
- **描述**: 使用弱默认密码
  - MariaDB: root123456
  - Redis: redis123456
  - Grafana: admin123456
- **影响**: 开发环境凭证泄露到生产环境
- **建议**:
  - 使用环境变量或 secrets 管理
  - 强制生产环境密码策略

### 6.2 安全最佳实践

1. **密钥管理**:
   - 种子加密密码应存储在 HSM 或密钥管理服务
   - JWT 密钥应定期轮换
   - 避免在代码或配置文件中硬编码密钥

2. **密码存储**:
   - 迁移到 bcrypt (cost factor >= 10)
   - 或使用 argon2id

3. **速率限制**:
   - 登录接口应有更严格的限制
   - 敏感操作应有独立的限制配置

4. **日志安全**:
   - 不记录敏感信息 (密码、密钥、种子)
   - 实施日志访问控制

---

## 7. 附录

### 7.1 Kafka Topic 定义

| Topic | 用途 |
|-------|------|
| trade-events | 交易事件 |
| matching-events | 撮合事件 |
| wallet-events | 钱包事件 |
| market-events | 市场事件 |
| settlement-events | 结算事件 |
| risk-events | 风控事件 |

### 7.2 环境变量

| 变量 | 说明 | 示例 |
|------|------|------|
| MYSQL_ROOT_PASSWORD | MariaDB root 密码 | (使用强密码) |
| REDIS_PASSWORD | Redis 密码 | (使用强密码) |
| GF_SECURITY_ADMIN_PASSWORD | Grafana 管理员密码 | (使用强密码) |
| JWT_SECRET | JWT 签名密钥 | (至少 32 字节随机字符串) |
| SEED_ENCRYPTION_PASSWORD | 种子加密密码 | (至少 32 字符强密码) |
