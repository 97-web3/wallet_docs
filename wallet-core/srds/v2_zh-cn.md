# 企业薪资支付系统 - 模块分析与扩展计划

## 执行摘要

本文档提供了对 `wallet-core/srds/README.md` 中记录的内部钱包系统的全面分析，并提出了支持企业跨 TRON、BSC 和以太坊网络进行加密货币薪资支付操作的扩展建议。

### 用户需求（已确认）

| 需求 | 选择 | 影响 |
|---|---|---|
| **规模** | 1,000-5,000 名员工 | 现有架构在优化后足够；分块批处理 |
| **审批模式** | 多级按金额 | 基于支付金额阈值的不同审批链 |
| **费用优先级** | 速度优先 | 优先考虑快速确认，使用 FAST Gas 设置，接受更高费用 |

---

# 1. 当前模块分析

## 1.1 模块清单

根据 `wallet-core/srds/README.md` 中的文档，系统由 **7 个核心模块** 组成：

| # | 模块名称 | Chinese Name | 位置 | 功能 |
|---|---|---|---|---|
| 01 | API Gateway | API 网关 | `01-api-gateway/` | 20+ HTTP 路由 |
| 02 | Business Service | 业务服务 | `02-business-service/` | 48 RPC |
| 03 | ChainRPC Service | 链RPC服务 | `03-chainrpc-service/` | 28 RPC |
| 04 | ChainSync Service | 链同步服务 | `04-chainsync-service/` | 17 RPC |
| 05 | Signer Service | 签名服务 | `05-signer-service/` | 14 RPC |
| 06 | Common Infrastructure | 通用库与基础设施 | `06-common-infrastructure/` | 共享工具 |
| 07 | Admin Service | 后台管理服务 | `07-admin-service/` (在 `admin-api/` 中记录) | 30+ RPC |

**总计：22 个数据库表上的 107+ RPC 功能**

---

## 1.2 模块职责

### 模块 01：API 网关 (`01-api-gateway/`)

**主要目的：** 通过 gRPC 将请求路由到微服务的统一 HTTP 入口点

**核心技术职责：**
- HTTP 到 gRPC 协议转换
- JWT 令牌认证和验证
- 分布式限流（全局、基于 IP、基于用户）
- CORS 处理跨域请求
- 请求日志记录和分布式跟踪 (Jaeger)
- Panic 恢复和异常处理
- 黑名单检查（IP 和用户标识符）

**关键 API/接口：**
- `/api/v1/*` 下的 20 多个 HTTP 路由
- 中间件链：恢复 → 请求 ID → 日志记录 → CORS → 速率限制 → 认证 → 黑名单 → 路由器

**数据模型：**
- 无持久化存储（无状态）
- Redis 用于分布式限流状态

**当前链支持：** 链无关（路由到后端服务）

---

### 模块 02：业务服务 (`02-business-service/`)

**主要目的：** 用户管理、认证、资产和交易的核心业务逻辑

**核心技术职责：**
- 用户注册、认证、会话管理
- 资产余额管理和交易历史
- 存款地址生成和监控
- 提款请求创建和审计工作流程
- 内部用户间转账
- 安全设置（2FA、设备管理、生物识别）
- 通知管理（推送、短信、邮件、应用内）
- 代币交换功能（DEX 集成）

**关键 API/接口：** 12 个子模块上的 48 个 RPC 功能：
- `Register`, `Login`, `Logout`, `RefreshToken`, `ResetPassword`
- `GetUserOverview`, `GetPersonalInfo`, `UpdatePersonalInfo`
- `GetAssetOverview`, `GetTransactionRecords`, `GetBalanceChanges`
- `GetDepositInfo`, `CreateDepositAddress`
- `GetWithdrawInfo`, `CreateWithdrawRequest`
- `GetTransferInfo`, `CreateTransfer`
- 安全、通知、偏好、地址簿、登录历史、交换功能

**数据模型（12 个表）：**
- `users` - 带有 KYC 级别、2FA 状态的用户账户
- `user_sessions` - 带有设备跟踪的会话令牌
- `verification_codes` - 用于各种目的的 OTP 代码
- `login_logs` - 带有风险评估的登录历史
- `user_roles` - RBAC（用户、财务、管理员）
- `blacklists` - 被阻止的标识符
- `platform_bindings` - Web3 钱包连接
- `withdrawal_audits` - 提款审批工作流程
- `trusted_devices` - 设备指纹识别
- `transfer_audits` - 内部转账审计跟踪
- `payroll_records` - 薪资/奖金跟踪
- `swap_transactions` - 交换历史

**当前链支持：** TRON、BSC、以太坊（通过 ChainRPC/Signer 服务）

---

### 模块 03：链RPC服务 (`03-chainrpc-service/`)

**主要目的：** 用于地址生成、交易构建和广播的区块链交互层

**核心技术职责：**
- 多链地址生成和验证
- 原生代币转账交易构建
- 代币 (TRC20/BEP20/ERC20) 转账交易构建
- 批量转账构造
- Gas 价格获取和估算
- 交易广播、加速和取消
- 链上余额查询（原生和代币）
- 交易状态和收据检索
- 区块高度和信息查询
- RPC 节点健康监控和故障转移

**关键 API/接口：** 28 个 RPC 功能：
- 地址：`GenerateAddress`, `GenerateAddressBatch`, `ValidateAddress`, `ConvertPubKeyToAddress`, `CheckInternalAddress`
- 交易：`BuildNativeTransfer`, `BuildTokenTransfer`, `BuildBatchTransfer`
- Gas：`EstimateGas`, `GetGasPrice`
- 广播：`BroadcastTransaction`, `BroadcastBatch`, `SpeedUpTransaction`, `CancelTransaction`
- 查询：`GetBalance`, `GetTokenBalance`, `GetMultiAddressBalance`
- 交易查询：`GetTransactionDetails`, `GetTransactionStatus`, `GetTransactionReceipt`
- 区块：`GetBlockHeight`, `GetBlockInfo`, `GetAddressNonce`
- 代币/节点：`GetTokenInfo`, `GetNodeHealth`, `SwitchActiveNode`, `GetNodeStatus`, `GetSupportedTokens`, `GetChainConfig`

**数据模型：**
- 无持久表（无状态查询）
- 缓存 Gas 价格和余额在 Redis 中

**当前链支持：**
- TRON (TRX/TRC20) - 币种类型 195
- BSC (BNB/BEP20) - 币种类型 60，账户 0
- 以太坊 (ETH/ERC20) - 币种类型 60，账户 1
- Polygon (MATIC/ERC20) - 可用但 V1 中禁用

---

### 模块 04：链同步服务 (`04-chainsync-service/`)

**主要目的：** 实时区块链同步、存款监控和余额跟踪

**核心技术职责：**
- 区块同步状态管理（启动/停止/重置）
- 地址监控以检测存款
- 交易监控和确认跟踪
- 跨地址的实时余额跟踪
- RPC 提供者健康监控和故障转移
- Kafka 事件发布用于存款/余额事件
- 区块链重组 (reorg) 检测

**关键 API/接口：** 17 个 RPC 功能：
- 同步：`StartSync`, `StopSync`, `GetSyncStatus`, `ResetSyncState`
- 地址：`AddMonitoredAddress`, `RemoveMonitoredAddress`, `ListMonitoredAddresses`
- 交易：`GetTransactionDetails`, `GetAddressTransactionHistory`, `GetBlockTransactions`
- 余额：`GetAddressBalance`, `GetMultiAddressBalances`
- 提供者：`GetProviderStatus`, `SwitchProvider`, `TestRPCEndpoint`
- 统计：`GetSyncStatistics`, `GetPerformanceMetrics`

**数据模型（6 个表）：**
- `blocks` - 已同步的区块数据，包含哈希、时间戳
- `transactions` - 已确认的交易，包含状态
- `address_monitors` - 监控的地址，带有 Webhook 回调
- `balance_records` - 地址余额快照
- `sync_tasks` - 后台同步任务跟踪
- `provider_status` - RPC 健康/性能指标

**Kafka 事件主题：**
- `chainsync.deposit.detected` - 原始存款检测
- `chainsync.deposit.confirmed` - 已确认的存款
- `chainsync.balance.changed` - 余额更新
- `chainsync.block.synced` - 区块同步完成
- `chainsync.tx.status` - 交易状态变化

**确认要求：**

| 链 | 标准 | 大额 (≥10k USDT) |
|---|---|---|
| TRON | 20 | 30 |
| BSC | 15 | 25 |
| 以太坊 | 12 | 20 |

**当前链支持：** TRON、BSC、以太坊

---

### 模块 05：签名服务 (`05-signer-service/`)

**主要目的：** 用于 HD 钱包密钥管理和交易签名的核心安全模块

**核心技术职责：**
- BIP39 种子生成和助记词管理
- BIP44 分层确定性地址派生
- 使用 PBKDF2 密钥派生的 AES-256-GCM 种子加密
- 单一种子生成多链地址
- 交易签名（单笔和批量）
- 消息签名以供验证
- 公钥导出（不暴露私钥）
- 热/冷钱包温度分离
- 全面的签名审计日志记录

**关键 API/接口：** 14 个 RPC 功能：
- 种子：`InitSeed`, `InitSeedFromMnemonic`, `ListSeeds`, `UpdateSeedStatus`
- 地址：`GenerateUserDepositAddresses`, `DerivePath`, `GenerateMultiChainAddresses`
- 公钥：`ExportPublicKey`, `ExportPublicKeyBatch`
- 签名：`SignTransaction`, `SignBatch`, `SignMessage`
- 审计：`GetSignatureLogs`, `HealthCheck`

**数据模型（4 个表）：**
- `master_seeds` - 加密种子，带有温度（热=1/冷=2），每日限额
- `deposit_addresses` - 每个链按 BIP44 路径生成的地址
- `signature_logs` - 所有签名操作的完整审计跟踪
- `pub_key_export_logs` - 公钥导出审计

**BIP44 派生路径：**

| 链 | 币种类型 | 账户 | 路径格式 |
|---|---|---|---|
| TRON | 195 | 0 | m/44'/195'/0'/0/{user_id} |
| BSC | 60 | 0 | m/44'/60'/0'/0/{user_id} |
| 以太坊 | 60 | 1 | m/44'/60'/1'/0/{user_id} |

**加密方法：**
- 种子加密：AES-256-GCM + PBKDF2（100,000 次迭代）
- 密钥派生：secp256k1 椭圆曲线
- 地址生成：Keccak256 哈希 → EIP-55 校验和 (ETH/BSC)，Base58Check (TRON)

**安全级别：** 关键 - 不应通过公共 API 网关暴露

**当前链支持：** TRON、BSC、以太坊

---

### 模块 06：通用基础设施 (`06-common-infrastructure/`)

**主要目的：** 共享库、工具、中间件和基础设施配置

**核心技术职责：**
- 中间件组件（JWT 认证、CORS、速率限制、恢复、日志记录）
- 工具函数（加密哈希、种子加密、Snowflake ID 生成、验证码）
- 基础数据模型（BaseModel, BaseModelWithUser）
- Redis 缓存包装器，带分布式锁
- 业务常量和 Redis 键模式
- 错误代码注册表
- 通用存储库模式（CRUD 操作）
- GORM 数据库连接和池
- Docker Compose 基础设施配置

**关键组件：**
```
common/
├── cache/redis_cache.go
├── captcha/captcha.go
├── code/code.go
├── constants/{constants,redis,signer}.go
├── db/{config,gorm}.go
├── enum/accountType.go
├── errcode/signer.go
├── interceptor/{example_usage,metadata}.go
├── middleware/{auth,common,cors,logging,metadata,ratelimit,recovery,redis_ratelimit}.go
├── model/base_model.go
├── models/wallet.go
├── repository/base_repository.go
└── utils/{config_finder,crypto,crypto_signer,response,snowflake,tracing,validator}.go
```

**基础设施组件：**
- MariaDB 10.11（主数据库）
- Redis 7.x（缓存、速率限制、会话）
- Kafka 7.5.0（事件流）
- ETCD 3.5.9（服务发现）
- Prometheus + Grafana（监控）
- Jaeger（分布式跟踪）

---

### 模块 07：后台服务 (`admin-api/`)

**主要目的：** 用于仪表板、用户管理、薪资、审批和 RBAC 的后台管理

**核心技术职责：**
- 仪表板统计数据和金库余额监控
- 用户管理（冻结、解冻、终止、角色分配）
- 薪资批量导入、处理和报告
- 提款审批工作流程（带 AML 集成）
- 交换配置管理
- 分录记录和金库调整
- 基于角色的访问控制 (RBAC)

**关键 API/接口：** 8 个子模块上的 30 多个 RPC 功能：
- 仪表板：`GetDashboardStatistics`, `GetVaultBalances`, `GetAlertThresholds`, `SetAlertThreshold`
- 用户：`AdminListUsers`, `AdminFreezeUser`, `AdminUnfreezeUser`, `AdminTerminateUser`, `AdminUpdateUserRole`
- 薪资：`BatchImportPayroll`, `ProcessPayrollBatch`, `GetPayrollHistory`, `ExportPayrollReport`
- 提款：`AdminListPendingWithdraws`, `AdminApproveWithdraw`, `AdminRejectWithdraw`
- 交换：`GetSwapConfig`, `UpdateSwapConfig`
- 分录：`GetLedgerRecords`, `ExportLedger`, `AdminAdjustVaultBalance`, `GetVaultAdjustmentHistory`, `ApproveVaultAdjustment`
- 角色：`GetRoles`, `AssignRole`, `GetUserPermissions`

**数据模型：**
- `vault_balances` - 每个链/资产的热/冷钱包余额
- `vault_adjustments` - 带有审批的手动余额调整
- `payroll_batches` - 薪资批次记录
- `payroll_items` - 个人薪资行项目
- `ledger_entries` - 完整的交易分类账（不可变）
- `admin_audit_logs` - 所有管理员操作

**RBAC 层次结构：**
```
super_admin → admin → finance → employee
```

**现有薪资能力：**
- Excel 文件上传（最大 10MB，1000 行）
- 批次验证和去重（batch_hash）
- 审批/拒绝工作流程
- 部分失败处理 (skip_invalid 标志)
- 为每位员工创建内部转账
- 带有转账 ID 的 XLSX/CSV 导出
- 24 小时批次有效期

---

## 1.3 依赖关系映射

### 服务依赖图

```
                    ┌─────────────────────────────────────────┐
                    │         API Gateway (端口 8080)         │
                    │    HTTP → gRPC, 认证, 速率限制        │
                    └─────────────────────────────────────────┘
                                        │
          ┌─────────────────┬───────────┼───────────┬─────────────────┐
          │                 │           │           │                 │
          ▼                 ▼           ▼           ▼                 ▼
┌─────────────────┐ ┌──────────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐
│ 业务服务        │ │ ChainRPC     │ │ChainSync │ │ Signer   │ │ 管理服务     │
│   (48 RPC)      │ │ (28 RPC)     │ │(17 RPC)  │ │ (14 RPC) │ │  (30+ RPC)   │
└─────────────────┘ └──────────────┘ └──────────┘ └──────────┘ └──────────────┘
         │                 │              │             │              │
         │                 │              │             │              │
         ├─────────────────┼──────────────┼─────────────┤              │
         │                 │              │             │              │
         │                 └──────────────┼─────────────┘              │
         │                                │                            │
         ▼                                ▼                            ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        通用基础设施                                         │
│   中间件/ │ 缓存/ │ 数据库/ │ 存储库/ │ 工具/ │ 常量/ │ 错误码/ │
└─────────────────────────────────────────────────────────────────────────────┘
         │                                │                            │
         ▼                                ▼                            ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           基础设施层                                       │
│     MariaDB  │  Redis  │  Kafka  │  ETCD  │  Prometheus  │  Jaeger          │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 直接依赖关系

| 模块 | 依赖于 | 依赖类型 |
|---|---|---|
| API 网关 | 业务, ChainRPC, ChainSync, Signer, Admin | gRPC 调用 |
| API 网关 | ETCD | 服务发现 |
| API 网关 | Redis | 速率限制状态 |
| 业务服务 | ChainRPC | 地址生成查询 |
| 业务服务 | Signer | HD 钱包派生 |
| 业务服务 | MariaDB | 用户数据持久化 |
| 业务服务 | Redis | 会话、验证码 |
| 业务服务 | Kafka | 事件分发（可选） |
| ChainRPC 服务 | Signer | 交易签名 |
| ChainRPC 服务 | RPC 节点 | 区块链交互 |
| ChainRPC 服务 | Redis | Gas 价格/余额缓存 |
| ChainSync 服务 | RPC 节点 | 区块链数据获取 |
| ChainSync 服务 | MariaDB | 区块/交易存储 |
| ChainSync 服务 | Redis | 同步状态缓存 |
| ChainSync 服务 | Kafka | 事件发布 |
| Signer 服务 | MariaDB | 种子/地址存储 |
| Signer 服务 | Redis | 临时数据 |
| Admin 服务 | 业务 | 用户操作 |
| Admin 服务 | ChainRPC | 余额查询 |
| Admin 服务 | MariaDB | 管理数据持久化 |

### 上游/下游关系

**上游服务（提供服务给其他服务）：**
1. **Signer Service** - 向 Business、ChainRPC 提供密钥管理
2. **ChainRPC Service** - 向 Business、Admin 提供区块链操作
3. **ChainSync Service** - 通过 Kafka 向 Business 提供同步/监控
4. **Common Infrastructure** - 向所有服务提供工具

**下游服务（从其他服务消费服务）：**
1. **API Gateway** - 消费所有后端服务
2. **Business Service** - 消费 ChainRPC、Signer、ChainSync 事件
3. **Admin Service** - 消费 Business、ChainRPC

### 识别出的耦合问题

1. **紧密耦合：** 业务服务同时直接依赖 ChainRPC 和 Signer 进行地址操作（可以抽象化）

2. **潜在循环风险：** Admin → Business → ChainRPC，Admin → ChainRPC（没有实际循环依赖，但调用路径复杂）

3. **缺少抽象：** 没有统一的“钱包服务”层来从业务逻辑中抽象出多链操作

4. **事件耦合：** ChainSync 发布到 Kafka，但消费者耦合是隐性的（未记录哪些服务消费哪些主题）

### 外部依赖

| 依赖项 | 目的 | 使用方 |
|---|---|---|
| QuickNode | RPC 提供者 | ChainRPC, ChainSync |
| Infura | RPC 提供者 (ETH) | ChainRPC, ChainSync |
| Alchemy | RPC 提供者 | ChainRPC, ChainSync |
| Ankr | RPC 提供者 | ChainRPC, ChainSync |
| Moralis | RPC 提供者 | ChainRPC, ChainSync |
| TronGrid | TRON API | ChainRPC, ChainSync |
| 1inch | DEX 聚合器 | Business (交换) |
| Paraswap | DEX 聚合器 | Business (交换) |
| Chainalysis/Elliptic/Coinfirm | AML 筛选 | Admin (提款审批) |

---

## 1.4 架构模式评估

### 自托管组件（员工钱包）

**当前实现：** 有限

当前系统主要设计为**托管式交易所钱包**，而非自托管。然而，该架构支持扩展：

**现有自托管元素：**
1. **BIP44 HD 派生** (Signer Service) - 可以为每个用户派生唯一的地址
2. **按用户地址生成** (ChainSync) - 监控用户特定地址
3. **地址监控** (ChainSync) - 监控用户特定地址

**缺失的自托管功能：**
- 客户端密钥生成和存储
- 用户控制的助记词备份/恢复
- 针对用户的硬件钱包集成
- 用户发起的交易签名（所有签名都是服务器端）

**自托管边界：** `deposit_addresses` 表存储地址，但不存储单个用户的私钥。私钥保留在 `master_seeds`（公司控制）中。

### 集中式钱包组件（公司金库）

**当前实现：** 全面

系统在集中式钱包管理方面表现出色：

1. **热钱包管理：**
   - `vault_balances` 跟踪每个链/资产的热钱包余额
   - `AdminAdjustVaultBalance` 用于手动充值
   - 低余额警告的阈值警报

2. **冷钱包支持：**
   - Signer Service `temperature` 字段（热=1，冷=2）
   - 热存储和冷存储的独立种子
   - 未记录冷到热的自动转账

3. **金库操作：**
   - 从金库进行薪资批量支付
   - 带有双重审批的金库调整
   - 完整的分类账跟踪

4. **密钥管理：**
   - 使用 AES-256-GCM 加密的主种子
   - 种子状态管理（活动/暂停/弃用）
   - 每日签名限制（部分实现）

### 分离策略

**当前分离：**
```
┌─────────────────────────────────────────────────────────────────────┐
│                    集中式（公司控制）                  │
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ │
│  │master_seeds │    │ vault_      │    │ Signer Service          │ │
│  │（加密）     │    │ balances    │    │ （内部 gRPC 专用）     │ │
│  └─────────────┘    └─────────────┘    └─────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ 派生的地址（仅公钥）
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    半托管式（面向用户）                     │
│                                                                     │
│  ┌─────────────────┐    ┌─────────────────┐                        │
│  │deposit_addresses│    │ user account    │                        │
│  │（仅公钥）       │    │ balances        │                        │
│  └─────────────────┘    └─────────────────┘                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**隔离机制：**
1. **网络隔离：** Signer Service 应仅接受内部 gRPC（记录为“不应通过公共 API 网关暴露”）
2. **数据库隔离：** 种子加密存储，仅 Signer Service 可访问
3. **API 隔离：** 没有公共端点直接暴露签名操作

### 混合模式

**记录的混合模型（来自 PRD）：**
- **Web2 钱包（托管）：** 公司控制，薪资分配，内部转账
- **Web3 钱包（非托管）：** 员工控制，用于外部操作

**当前实现差距：** 概念上记录的非托管（Web3）钱包组件在 SRD 中尚未完全实现。系统目前主要以托管模式运行。

### 安全边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ 信任边界 1：公共互联网 ←→ API 网关                            │
│ 保护：JWT 认证、速率限制、CORS、黑名单                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 信任边界 2：API 网关 ←→ 后端服务                           │
│ 保护：内部 gRPC、服务网格（推荐）                         │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 信任边界 3：服务 ←→ 签名服务 (关键)                      │
│ 保护：网络隔离、无公共访问、审计日志                      │
│ 风险：目前可从 API 网关访问 (关键-01 安全问题)          │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 信任边界 4：签名服务 ←→ 主种子                            │
│ 保护：AES-256-GCM 加密、PBKDF2 密钥派生、TEE（可选）    │
└─────────────────────────────────────────────────────────────────────────────┘
```

# 2. 建议的扩展

## 2.A 多链支持增强

### 2.A.1 TRON 网络集成（增强）

**当前状态：** 已实现 (TRX/TRC20)

**建议的增强：**

| 增强 | 影响模块 | 新模块 | 技术要求 |
|---|---|---|---|
| 能量/带宽管理 | ChainRPC | 无 | TronGrid API 用于资源估算 |
| 资源委托支持 | ChainRPC, Business | `TronResourceManager` | 质押/取消质押 TRX 以获取能量 |
| TRC20 批准优化 | ChainRPC | 无 | 薪资效率的批量批准 |
| 多签 TRON 支持 | ChainRPC, Signer | `TronMultiSigAdapter` | TRON 多签合约部署 |

**集成点：**
- ChainRPC `BuildTokenTransfer` → 添加 `energy_estimate` 和 `bandwidth_estimate` 字段
- Gas 估算 → 转换为 TRON 的能量/带宽
- 新 RPC：`GetTronResources(address)` → 返回能量/带宽余额

**链特定考虑因素：**
- TRON 使用 protobuf 进行交易序列化（与 EVM RLP 不同）
- 大型薪资批次的能量租赁市场集成
- 3 秒区块时间 vs 以太坊的 12 秒

### 2.A.2 BSC 网络集成（增强）

**当前状态：** 已实现 (BNB/BEP20)

**建议的增强：**

| 增强 | 影响模块 | 新模块 | 技术要求 |
|---|---|---|---|
| BSC 特定 Gas 优化 | ChainRPC | 无 | Gas 价格预言机集成 |
| 更快的确认跟踪 | ChainSync | 无 | 3 秒区块处理 |
| BEP20 批量转账合约 | ChainRPC | `BSCBatchTransfer` | 部署 Gas 高效的批量合约 |

**集成点：**
- ChainSync 确认配置 → 标准交易从 15 降至 10
- Gas 价格预言机 → BSCScan API 集成

### 2.A.3 以太坊网络集成（增强）

**当前状态：** 已实现 (ETH/ERC20)

**建议的增强：**

| 增强 | 影响模块 | 新模块 | 技术要求 |
|---|---|---|---|
| EIP-1559 完全支持 | ChainRPC | 无 | Base 费率 + 优先级费率处理 |
| Gas 价格预测 | ChainRPC | `GasPricePredictor` | 历史分析用于调度 |
| Layer 2 准备 | ChainRPC, ChainSync | `L2Adapter` 接口 | Polygon/Arbitrum/Optimism 存根 |
| MEV 保护 | ChainRPC | `FlashbotsAdapter` | 私有交易提交 |

**集成点：**
- `GetGasPrice` → 返回 `baseFee`, `priorityFee`, `maxFee` 以用于 EIP-1559
- 交易构建 → 支持 `maxFeePerGas` 和 `maxPriorityFeePerGas`

### 2.A.4 跨链交易编排

**影响模块：** ChainRPC, Business, Admin

**新模块：** `PayrollOrchestrator`

**技术要求：**
```go
type PayrollOrchestrator interface {
    // 跨多链提交薪资
    SubmitMultiChainPayroll(batch PayrollBatch) (MultiChainResult, error)

    // 获取聚合状态
    GetPayrollStatus(batchID string) (MultiChainStatus, error)

    // 带有链回退的重试失败交易
    RetryWithFallback(txID string, fallbackChain ChainType) error
}
```

**集成点：**
- Admin `ProcessPayrollBatch` → 根据员工偏好并行调用 `PayrollOrchestrator.SubmitMultiChainPayroll` 到 TRON, BSC, ETH
- 统一错误处理和链特定重试逻辑

### 2.A.5 多链费用优化

**影响模块：** ChainRPC, Admin

**新模块：** `FeeOptimizer`

**技术要求：**
```go
type FeeOptimizer interface {
    // 比较同一笔交易跨链的费用
    CompareFees(amount *big.Int, token string) ([]ChainFeeEstimate, error)

    // 推荐支付的最优链
    RecommendChain(amount *big.Int, urgency FeeUrgency) (ChainType, error)

    // 获取历史费用分析
    GetFeeHistory(chain ChainType, period time.Duration) (FeeHistory, error)
}
```

**集成点：**
- 薪资前分析 → 按链显示成本明细
- 员工地址管理 → 根据历史费用建议首选链

### 2.A.6 聚合余额报告

**影响模块：** Admin, ChainRPC

**新模块：** `BalanceAggregator`

**技术要求：**
- 汇总余额视图：`GetAggregatedBalances()` → 所有链，所有代币，单个响应
- 从 ChainSync 事件实时同步
- 所有余额的美元/USDT 等值计算
- 热钱包与冷钱包明细划分

**数据库模式添加：**
```sql
CREATE TABLE aggregated_balances (
    id BIGINT PRIMARY KEY,
    wallet_type ENUM('hot', 'cold', 'user') NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    token_address VARCHAR(64),
    token_symbol VARCHAR(20) NOT NULL,
    balance DECIMAL(36, 18) NOT NULL,
    usdt_value DECIMAL(20, 8),
    last_updated DATETIME NOT NULL,
    INDEX idx_wallet_chain (wallet_type, chain)
);
```

### 2.A.7 代币标准抽象

**影响模块：** ChainRPC

**新模块：** `TokenAdapter`

**技术要求：**
```go
type TokenAdapter interface {
    // 链无关转账
    Transfer(ctx context.Context, req TransferRequest) (TxHash, error)

    // 链无关余额查询
    BalanceOf(ctx context.Context, address string, token string) (*big.Int, error)

    // 链无关批准
    Approve(ctx context.Context, spender string, amount *big.Int) (TxHash, error)
}

// 实现
type TRC20Adapter struct { /* TRON 特有 */ }
type BEP20Adapter struct { /* BSC 特有 */ }
type ERC20Adapter struct { /* ETH 特有 */ }
```

**集成点：**
- ChainRPC 内部使用 `TokenAdapter` 接口
- 工厂模式：`GetTokenAdapter(chain ChainType) TokenAdapter`

---

## 2.B 自托管钱包功能（员工钱包）

### 2.B.1 HD 钱包派生管理

**影响模块：** Signer

**增强：**

| 功能 | 实现 |
|---|---|
| 每个员工的派生路径 | `m/44'/{coin}'/0'/0/{employee_id}` (当前) |
| 路径注册表 | 新表 `derivation_registry` |
| 路径冲突检测 | 分配前检查 |

**数据库模式：**
```sql
CREATE TABLE derivation_registry (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    seed_id BIGINT NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    derivation_path VARCHAR(100) NOT NULL,
    address VARCHAR(64) NOT NULL,
    assigned_at DATETIME NOT NULL,
    UNIQUE KEY uk_employee_chain (employee_id, chain),
    UNIQUE KEY uk_path (seed_id, derivation_path)
);
```

### 2.B.2 安全密钥生成（客户端）

**新模块：** `ClientKeyManager` (移动/Web SDK)

**技术要求：**
- 使用平台安全随机数 (iOS: SecRandomCopyBytes, Android: SecureRandom) 的 CSPRNG
- 最小 256 位熵验证
- 密钥生成审计日志（仅时间戳，不记录密钥）
- 气隙密钥生成选项（二维码传输）

**注意：** 这是一个客户端组件，不是服务器端组件。在真正的自托管模式下，服务器永远不会看到员工私钥。

### 2.B.3 密钥存储机制（客户端）

**新模块：** `SecureKeyStore` (移动 SDK)

**技术要求：**
- iOS：Keychain Services，使用 `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`
- Android：AndroidKeyStore，使用硬件支持的密钥
- 加密：使用生物识别保护的密钥的 AES-256-GCM
- 密钥包装用于备份

### 2.B.4 备份和恢复工作流程

**影响模块：** Business (新端点)

**新 API：**
- `InitiateBackupVerification` - 生成助记词验证挑战
- `VerifyBackup` - 确认用户已正确记录助记词
- `RequestRecovery` - 启动定时锁定恢复（例如，48 小时延迟）

**技术要求：**
- BIP-39 助记词显示（12 或 24 个单词）
- 屏幕截图阻止（Android 上的 FLAG_SECURE，iOS 上的 UITextField.isSecureTextEntry）
- 恢复短语验证（用户重新输入 3 个随机单词）
- 可选：用于企业密钥恢复的 Shamir 秘密共享

### 2.B.5 多链地址生成（增强）

**影响模块：** Signer

**当前：** 已支持从单个种子生成多链地址

**增强：**
- `GenerateEmployeeWalletSet(employee_id)` → 一次性返回所有 3 个链的地址
- 地址标签：在 `deposit_addresses.metadata` 中存储员工姓名、部门

---

## 2.C 中央钱包账户功能（公司金库）

### 2.C.1 热钱包管理（增强）

**影响模块：** Admin, Business

**新模块：** `HotWalletManager`

**技术要求：**
```go
type HotWalletManager interface {
    // 检查热钱包是否需要补充
    CheckReplenishmentNeeded(chain ChainType) (bool, *big.Int, error)

    // 从冷钱包触发补给
    RequestReplenishment(chain ChainType, amount *big.Int) (RequestID, error)

    // 轮换热钱包（生成新钱包，转移余额）
    RotateHotWallet(chain ChainType) error

    // 紧急清空到冷钱包
    EmergencyDrain(chain ChainType) error
}
```

**数据库模式：**
```sql
CREATE TABLE hot_wallet_config (
    id BIGINT PRIMARY KEY,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    wallet_address VARCHAR(64) NOT NULL,
    min_balance DECIMAL(36, 18) NOT NULL,
    target_balance DECIMAL(36, 18) NOT NULL,
    max_balance DECIMAL(36, 18) NOT NULL,
    auto_replenish BOOLEAN DEFAULT FALSE,
    last_rotation DATETIME,
    UNIQUE KEY uk_chain (chain)
);
```

### 2.C.2 冷钱包集成

**影响模块：** Signer, Admin

**新模块：** `ColdWalletBridge`

**技术要求：**
- 离线签名工作流程：导出未签名交易 → 在气隙设备上签名 → 导入签名
- 冷钱包转账的多重签名支持
- 冷钱包余额监控（仅监视）

**新 API：**
- `ExportUnsignedTransaction(tx)` → 返回二维码或文件
- `ImportSignedTransaction(signature)` → 广播已签名交易
- `GetColdWalletBalance(chain)` → 只读余额检查

### 2.C.3 自动资金分配

**新模块：** `TreasuryAutomation`

**技术要求：**
```go
type TreasuryAutomation interface {
    // 配置自动补给规则
    SetReplenishmentRule(rule ReplenishmentRule) error

    // 根据薪资时间表预测资金需求
    PredictFundingNeeds(nextDays int) ([]FundingPrediction, error)

    // 执行计划的资金转移
    ExecuteScheduledTransfers() error
}

type ReplenishmentRule struct {
    Chain           ChainType
    TriggerBalance  *big.Int  // 低于此金额时触发
    TargetBalance   *big.Int  // 补给到此金额
    SourceWallet    string    // 冷钱包地址
    RequiresApproval bool
    ApproverCount   int       // 用于多签
}
```

### 2.C.4 内部账户分类账（增强）

**影响模块：** Admin (`ledger_entries` 表)

**增强：**
- 实施复式记账法
- 支持账户层级结构（公司 → 部门 → 成本中心）
- 权责发生制会计（在审批时记录负债，而不是在支付时）

**数据库模式添加：**
```sql
CREATE TABLE ledger_accounts (
    id BIGINT PRIMARY KEY,
    account_code VARCHAR(20) NOT NULL UNIQUE,
    account_name VARCHAR(100) NOT NULL,
    account_type ENUM('asset', 'liability', 'equity', 'revenue', 'expense') NOT NULL,
    parent_id BIGINT,
    is_active BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (parent_id) REFERENCES ledger_accounts(id)
);

-- 增强现有 ledger_entries
ALTER TABLE ledger_entries ADD COLUMN debit_account_id BIGINT;
ALTER TABLE ledger_entries ADD COLUMN credit_account_id BIGINT;
```

---

## 2.D 薪资特定功能

### 2.D.1 批量支付处理（增强）

**影响模块：** Admin

**当前状态：** 支持每个批次最多 1000 个接收者

**增强：**

| 功能 | 实现 |
|---|---|
| 增加到 10,000+ 接收者 | 分块处理，后台作业 |
| 多链批量操作 | 并行提交到 3 条链 |
| Nonce 管理 | 每条链的 Nonce 排序 |
| 部分完成跟踪 | 在 `payroll_items` 中跟踪项目级别状态 |

**技术要求：**
```go
type BatchProcessor interface {
    // 使用分块处理大批量
    ProcessBatch(batch PayrollBatch, options ProcessOptions) (BatchResult, error)

    // 从检查点恢复失败的批次
    ResumeBatch(batchID string) (BatchResult, error)

    // 获取实时进度
    GetBatchProgress(batchID string) (BatchProgress, error)
}

type ProcessOptions struct {
    ChunkSize       int           // 每个块的项目数（默认：100）
    ParallelChains  bool          // 并行提交到多个链
    RetryCount      int           // 每个失败项目的重试次数
    ContinueOnError bool          // 遇到单个失败时不停止
}
```

### 2.D.2 定期/循环支付

**影响模块：** Admin, Business

**新模块：** `PayrollScheduler`

**技术要求：**
```go
type PayrollScheduler interface {
    // 创建定期计划
    CreateSchedule(schedule PayrollSchedule) (ScheduleID, error)

    // 修改现有计划
    UpdateSchedule(scheduleID string, updates ScheduleUpdates) error

    // 暂停/恢复计划
    SetScheduleStatus(scheduleID string, status ScheduleStatus) error

    // 获取即将执行的计划
    GetUpcomingExecutions(days int) ([]ScheduledExecution, error)

    // 手动触发
    TriggerManually(scheduleID string) error
}

type PayrollSchedule struct {
    ID              string
    Name            string
    CronExpression  string            // "0 9 1,15 * *" 表示 1 号和 15 号
    Timezone        string            // "America/New_York"
    TemplateID      string            // 引用支付模板
    PreExecutionHours int             // 执行前的小时数以进行验证
    AutoApprove     bool              // 用于例行薪资
    AutoApproveLimit *big.Int         // 自动批准的最大金额
}
```

**数据库模式：**
```sql
CREATE TABLE payroll_schedules (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    cron_expression VARCHAR(50) NOT NULL,
    timezone VARCHAR(50) NOT NULL DEFAULT 'UTC',
    template_id BIGINT NOT NULL,
    pre_execution_hours INT DEFAULT 24,
    auto_approve BOOLEAN DEFAULT FALSE,
    auto_approve_limit DECIMAL(20, 8),
    status ENUM('active', 'paused', 'completed') DEFAULT 'active',
    last_execution DATETIME,
    next_execution DATETIME,
    created_by BIGINT NOT NULL,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    FOREIGN KEY (template_id) REFERENCES payroll_templates(id)
);

CREATE TABLE scheduled_executions (
    id BIGINT PRIMARY KEY,
    schedule_id BIGINT NOT NULL,
    scheduled_at DATETIME NOT NULL,
    executed_at DATETIME,
    status ENUM('pending', 'executing', 'completed', 'failed', 'skipped') DEFAULT 'pending',
    batch_id BIGINT,
    error_message TEXT,
    FOREIGN KEY (schedule_id) REFERENCES payroll_schedules(id),
    FOREIGN KEY (batch_id) REFERENCES payroll_batches(id)
);
```

### 2.D.3 支付模板管理

**影响模块：** Admin

**新模块：** `TemplateManager`

**技术要求：**
```go
type PayrollTemplate struct {
    ID              string
    Name            string
    Version         int
    Recipients      []TemplateRecipient
    DefaultChain    ChainType
    DefaultToken    string
    GasSettings     GasSettings
    Variables       map[string]VariableDefinition  // 用于动态金额
    CreatedBy       string
    CreatedAt       time.Time
    UpdatedAt       time.Time
}

type TemplateRecipient struct {
    EmployeeID      string
    Chain           ChainType       // 每位员工覆盖
    Token           string          // 每位员工覆盖
    AmountType      string          // "fixed" 或 "variable"
    FixedAmount     *big.Int
    VariableKey     string          // 引用 Variables 映射
}
```

**数据库模式：**
```sql
CREATE TABLE payroll_templates (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    version INT NOT NULL DEFAULT 1,
    template_data JSON NOT NULL,  -- 存储完整的 TemplateRecipient 数组
    default_chain ENUM('TRON', 'BSC', 'ETH'),
    default_token VARCHAR(64),
    gas_settings JSON,
    variables JSON,
    is_active BOOLEAN DEFAULT TRUE,
    created_by BIGINT NOT NULL,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    UNIQUE KEY uk_name_version (name, version)
);

CREATE TABLE template_history (
    id BIGINT PRIMARY KEY,
    template_id BIGINT NOT NULL,
    version INT NOT NULL,
    changed_by BIGINT NOT NULL,
    changed_at DATETIME NOT NULL,
    change_type ENUM('create', 'update', 'deactivate') NOT NULL,
    previous_data JSON,
    new_data JSON,
    FOREIGN KEY (template_id) REFERENCES payroll_templates(id)
);
```

### 2.D.4 多币种/多代币支持

**影响模块：** Business, Admin

**当前状态：** 部分（支持每条链的多种代币）

**增强：**
- 存储每位员工的首选代币
- 如果需要，通过 DEX 自动进行代币转换
- 在批次创建时锁定汇率

**数据库模式添加：**
```sql
CREATE TABLE employee_payment_preferences (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    preferred_chain ENUM('TRON', 'BSC', 'ETH'),
    preferred_token VARCHAR(64),
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    UNIQUE KEY uk_employee (employee_id)
);

ALTER TABLE payroll_items
ADD COLUMN exchange_rate DECIMAL(20, 8),
ADD COLUMN exchange_locked_at DATETIME,
ADD COLUMN original_token VARCHAR(64),
ADD COLUMN original_amount DECIMAL(36, 18);
```

### 2.D.5 支付审批工作流程（增强）

**影响模块：** Admin

**当前状态：** 基本审批（大额批次的挂牌-核对）

**增强：**

| 功能 | 实现 |
|---|---|
| 多级审批 | 可配置的审批链 |
| 基于部门的路由 | 不同的审批人针对不同的部门 |
| 审批委托 | 临时委托分配 |
| 基于时间的自动批准 | 针对低于阈值的例行薪资 |
| 审批过期 | X 天后需要重新审批 |

**数据库模式：**
```sql
CREATE TABLE approval_chains (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at DATETIME NOT NULL
);

CREATE TABLE approval_chain_levels (
    id BIGINT PRIMARY KEY,
    chain_id BIGINT NOT NULL,
    level_order INT NOT NULL,
    required_approvers INT NOT NULL DEFAULT 1,
    timeout_hours INT,  -- 超时后自动升级
    FOREIGN KEY (chain_id) REFERENCES approval_chains(id)
);

CREATE TABLE approval_chain_approvers (
    id BIGINT PRIMARY KEY,
    level_id BIGINT NOT NULL,
    approver_user_id BIGINT,
    approver_role VARCHAR(50),  -- 可按角色批准
    FOREIGN KEY (level_id) REFERENCES approval_chain_levels(id)
);

CREATE TABLE payroll_approvals (
    id BIGINT PRIMARY KEY,
    batch_id BIGINT NOT NULL,
    level_id BIGINT NOT NULL,
    approver_id BIGINT,
    status ENUM('pending', 'approved', 'rejected', 'delegated', 'expired') DEFAULT 'pending',
    comments TEXT,
    approved_at DATETIME,
    expires_at DATETIME,
    delegated_to BIGINT,
    FOREIGN KEY (batch_id) REFERENCES payroll_batches(id),
    FOREIGN KEY (level_id) REFERENCES approval_chain_levels(id)
);
```

### 2.D.6 失败交易重试

**影响模块：** Admin, ChainRPC

**新模块：** `RetryManager`

**技术要求：**
```go
type RetryManager interface {
    // 配置重试策略
    SetRetryPolicy(policy RetryPolicy) error

    // 自动重试带退避
    AutoRetry(txID string) error

    // 带选项的手动重试
    ManualRetry(txID string, options RetryOptions) error

    // 获取重试历史
    GetRetryHistory(txID string) ([]RetryAttempt, error)

    // 移至死信队列
    MoveToDeadLetter(txID string, reason string) error
}

type RetryPolicy struct {
    MaxRetries          int
    InitialBackoff      time.Duration
    MaxBackoff          time.Duration
    BackoffMultiplier   float64
    RetryableErrors     []string        // 触发重试的错误代码
    GasIncreasePct      int             // 重试时 Gas 增加 X%
    EnableChainFallback bool            // 重试失败时尝试备用链
}
```

**数据库模式：**
```sql
CREATE TABLE transaction_retries (
    id BIGINT PRIMARY KEY,
    original_tx_id VARCHAR(100) NOT NULL,
    payroll_item_id BIGINT,
    attempt_number INT NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    tx_hash VARCHAR(100),
    status ENUM('pending', 'submitted', 'confirmed', 'failed') NOT NULL,
    error_code VARCHAR(50),
    error_message TEXT,
    gas_price_used DECIMAL(20, 8),
    attempted_at DATETIME NOT NULL,
    FOREIGN KEY (payroll_item_id) REFERENCES payroll_items(id)
);

CREATE TABLE dead_letter_transactions (
    id BIGINT PRIMARY KEY,
    original_tx_id VARCHAR(100) NOT NULL,
    payroll_item_id BIGINT,
    total_attempts INT NOT NULL,
    final_error_code VARCHAR(50),
    final_error_message TEXT,
    moved_at DATETIME NOT NULL,
    resolution_status ENUM('unresolved', 'manual_resolved', 'refunded') DEFAULT 'unresolved',
    resolution_notes TEXT,
    resolved_by BIGINT,
    resolved_at DATETIME
);
```

### 2.D.7 支付状态跟踪（增强）

**影响模块：** Admin, ChainSync

**当前状态：** 基本状态跟踪

**增强：**
- 实时状态更新 → WebSocket 推送到管理仪表板
- 确认深度跟踪 → 显示 3/12 次确认
- Webhook 通知 → POST 到外部系统
- 员工状态门户 → 员工的只读视图
- 推送通知 → 电子邮件、短信、应用内

**新 API：**
- WebSocket：`ws://api/v1/payroll/stream/{batch_id}`
- Webhook 配置：`POST /api/v1/admin/webhooks`
- 员工门户：`GET /api/v1/employee/payments`

**数据库模式：**
```sql
CREATE TABLE webhook_configs (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    url VARCHAR(500) NOT NULL,
    secret_key VARCHAR(100) NOT NULL,
    events JSON NOT NULL,  -- ["payment.confirmed", "payment.failed"]
    is_active BOOLEAN DEFAULT TRUE,
    created_by BIGINT NOT NULL,
    created_at DATETIME NOT NULL
);

CREATE TABLE webhook_deliveries (
    id BIGINT PRIMARY KEY,
    webhook_id BIGINT NOT NULL,
    event_type VARCHAR(50) NOT NULL,
    payload JSON NOT NULL,
    response_code INT,
    response_body TEXT,
    delivered_at DATETIME,
    retry_count INT DEFAULT 0,
    FOREIGN KEY (webhook_id) REFERENCES webhook_configs(id)
);
```

---

## 2.E 安全增强

### 2.E.1 多重签名支持

**影响模块：** Signer, Admin

**新模块：** `MultiSigManager`

**技术要求：**

| 链 | 实现 |
|---|---|
| Ethereum/BSC | Gnosis Safe 合约 |
| TRON | TRON 多签合约 |
| 所有链 | 链下审批，链上单个签名 |

```go
type MultiSigManager interface {
    // 创建多签配置
    CreateMultiSig(config MultiSigConfig) (MultiSigID, error)

    // 请求签名
    RequestSignatures(txID string) error

    // 提交签名
    SubmitSignature(txID string, signerID string, signature []byte) error

    // 检查阈值是否满足
    CheckThreshold(txID string) (bool, error)

    // 最终确定并广播
    Finalize(txID string) (TxHash, error)
}

type MultiSigConfig struct {
    Name            string
    Threshold       int         // M of N 中的 M
    TotalSigners    int         // N of N 中的 N
    SignerIDs       []string
    AmountThreshold *big.Int    // 超过此金额才需要多签
}
```

**数据库模式：**
```sql
CREATE TABLE multisig_configs (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    threshold INT NOT NULL,
    total_signers INT NOT NULL,
    amount_threshold DECIMAL(20, 8),
    chain ENUM('TRON', 'BSC', 'ETH', 'ALL') NOT NULL,
    contract_address VARCHAR(64),  -- 用于链上多签
    is_active BOOLEAN DEFAULT TRUE,
    created_at DATETIME NOT NULL
);

CREATE TABLE multisig_signers (
    id BIGINT PRIMARY KEY,
    config_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    signer_address VARCHAR(64),
    FOREIGN KEY (config_id) REFERENCES multisig_configs(id)
);

CREATE TABLE multisig_requests (
    id BIGINT PRIMARY KEY,
    config_id BIGINT NOT NULL,
    transaction_id VARCHAR(100) NOT NULL,
    raw_transaction TEXT NOT NULL,
    status ENUM('pending', 'threshold_met', 'executed', 'expired', 'cancelled') DEFAULT 'pending',
    expires_at DATETIME,
    created_at DATETIME NOT NULL,
    FOREIGN KEY (config_id) REFERENCES multisig_configs(id)
);

CREATE TABLE multisig_signatures (
    id BIGINT PRIMARY KEY,
    request_id BIGINT NOT NULL,
    signer_id BIGINT NOT NULL,
    signature TEXT NOT NULL,
    signed_at DATETIME NOT NULL,
    FOREIGN KEY (request_id) REFERENCES multisig_requests(id),
    FOREIGN KEY (signer_id) REFERENCES multisig_signers(id)
);
```

### 2.E.2 硬件钱包集成

**影响模块：** Signer (冷钱包操作), Admin (UI)

**新模块：** `HardwareWalletBridge`

**技术要求：**
- 通过 Ledger Live API 或 HID 支持 Ledger
- 通过 Trezor Connect 支持 Trezor
- USB/蓝牙连接处理
- 固件版本验证

**集成点：**
- 冷钱包签名：导出未签名交易 → 在硬件设备上签名 → 导入签名
- 管理界面：硬件钱包连接向导

### 2.E.3 生物识别认证

**影响模块：** Business (API), 移动 SDK

**当前状态：** 部分（存在 `trusted_devices` 表）

**增强：**
- 对高价值审批要求生物识别重新认证
- 生物识别模板哈希存储（从不存储实际生物识别数据）
- 生物识别失败时的备用 PIN/密码

### 2.E.4 基于角色的访问控制（增强）

**影响模块：** Admin

**当前状态：** 基本角色（超级管理员、管理员、财务、员工）

**增强：**

| 角色 | 新权限 |
|---|---|
| 财务经理 | 热/冷钱包管理、资金分配 |
| 薪资操作员 | 创建/修改批次，不能批准自己的批次 |
| 审计员 | 所有记录的只读访问，导出功能 |

**数据库模式增强：**
```sql
-- 粒度权限
CREATE TABLE permissions (
    id BIGINT PRIMARY KEY,
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,  -- payroll, treasury, audit, user_management
    description TEXT
);

-- 角色-权限映射
CREATE TABLE role_permissions (
    id BIGINT PRIMARY KEY,
    role_id BIGINT NOT NULL,
    permission_id BIGINT NOT NULL,
    UNIQUE KEY uk_role_permission (role_id, permission_id)
);

-- 示例权限
INSERT INTO permissions (code, name, category) VALUES
('payroll.create', '创建薪资批次', 'payroll'),
('payroll.approve', '批准薪资批次', 'payroll'),
('payroll.approve_own', '批准自己的薪资批次', 'payroll'),  -- 通常被拒绝
('treasury.view', '查看金库余额', 'treasury'),
('treasury.transfer', '执行金库转账', 'treasury'),
('audit.export', '导出审计日志', 'audit');
```

### 2.E.5 交易速度限制

**影响模块：** Business, Admin

**新模块：** `VelocityLimiter`

**技术要求：**
```go
type VelocityLimiter interface {
    // 检查交易是否超过限制
    CheckLimits(userID string, amount *big.Int, txType string) (LimitResult, error)

    // 获取当前使用情况
    GetCurrentUsage(userID string) (UsageSummary, error)

    // 配置限制
    SetLimits(userID string, limits LimitConfig) error
}

type LimitConfig struct {
    DailyLimit      *big.Int
    WeeklyLimit     *big.Int
    MonthlyLimit    *big.Int
    PerTxLimit      *big.Int
    RequireApprovalAbove *big.Int
}
```

**数据库模式：**
```sql
CREATE TABLE velocity_limits (
    id BIGINT PRIMARY KEY,
    scope_type ENUM('user', 'role', 'wallet', 'global') NOT NULL,
    scope_id BIGINT,  -- user_id, role_id, wallet_id, 或 NULL 表示全局
    daily_limit DECIMAL(20, 8),
    weekly_limit DECIMAL(20, 8),
    monthly_limit DECIMAL(20, 8),
    per_tx_limit DECIMAL(20, 8),
    require_approval_above DECIMAL(20, 8),
    is_active BOOLEAN DEFAULT TRUE,
    created_at DATETIME NOT NULL
);

CREATE TABLE velocity_usage (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    period_type ENUM('daily', 'weekly', 'monthly') NOT NULL,
    period_start DATE NOT NULL,
    total_amount DECIMAL(20, 8) NOT NULL DEFAULT 0,
    tx_count INT NOT NULL DEFAULT 0,
    last_updated DATETIME NOT NULL,
    UNIQUE KEY uk_user_period (user_id, period_type, period_start)
);
```

### 2.E.6 全面审计日志（增强）

**影响模块：** 所有

**当前状态：** 存在 `admin_audit_logs` 和 `signature_logs`

**增强：**
- 不可变审计日志存储（追加写入）
- 用于篡改证据的加密哈希链
- 日志保留：最少 7 年
- SIEM 集成（Splunk, ELK Stack 导出）

**数据库模式增强：**
```sql
CREATE TABLE audit_log_chain (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    previous_hash VARCHAR(64) NOT NULL,  -- 前一个条目的 SHA-256
    entry_hash VARCHAR(64) NOT NULL,     -- 当前条目的 SHA-256
    user_id BIGINT,
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    resource_id VARCHAR(100),
    details JSON,
    ip_address VARCHAR(45),
    user_agent TEXT,
    timestamp DATETIME(6) NOT NULL,      -- 微秒精度
    INDEX idx_user_action (user_id, action),
    INDEX idx_resource (resource_type, resource_id),
    INDEX idx_timestamp (timestamp)
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED;

-- 触发器以强制追加写入
DELIMITER //
CREATE TRIGGER audit_log_chain_immutable
BEFORE UPDATE ON audit_log_chain
FOR EACH ROW
BEGIN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = '审计日志是不可变的';
END //
DELIMITER ;
```

---

## 2.F 运营功能

### 2.F.1 交易历史管理（增强）

**影响模块：** Admin

**当前状态：** 基本过滤和导出

**增强：**
- 交易元数据的全文搜索
- 高级过滤（金额范围、确认深度）
- 批量操作（导出选择，重试选择）

**新 API：**
```go
type TransactionHistoryQuery struct {
    Chains          []ChainType
    Tokens          []string
    Statuses        []TxStatus
    DateFrom        time.Time
    DateTo          time.Time
    AmountMin       *big.Int
    AmountMax       *big.Int
    RecipientSearch string      // 全文搜索
    SenderSearch    string
    ConfirmationMin int
    Page            int
    PageSize        int
    SortBy          string
    SortOrder       string
}
```

### 2.F.2 实时余额监控（增强）

**影响模块：** Admin, ChainSync

**增强：**
- 基于 WebSocket 的仪表板实时更新
- 历史余额图表（随时间变化的线图）
- 预测余额（当前 - 待定薪资）

**新 API：**
- WebSocket：`ws://api/v1/admin/balances/stream`
- 历史记录：`GET /api/v1/admin/balances/history?period=30d`

### 2.F.3 Gas 费优化

**影响模块：** ChainRPC, Admin

**新模块：** `GasOptimizer`

**技术要求：**
```go
type GasOptimizer interface {
    // 获取跨链当前 Gas 价格
    GetCurrentPrices() (map[ChainType]GasPrice, error)

    // 预测未来时间的 Gas 价格
    PredictPrice(chain ChainType, atTime time.Time) (GasPrice, error)

    // 推荐最佳提交时间
    RecommendSubmissionTime(chain ChainType, urgency Urgency) (time.Time, error)

    // 估算薪资批次的总体费用
    EstimateBatchFees(batch PayrollBatch) (FeeEstimate, error)
}
```

**集成点：**
- 薪资前验证：在审批前显示估计费用
- 计划薪资：可以选择推迟执行到费用较低的时段

### 2.F.4 交易确认跟踪（增强）

**影响模块：** ChainSync, Admin

**当前状态：** 按链定义了确认要求

**增强：**
- 实时确认进度：“3/12 次确认”
- 区块链重组检测和通知
- 交易卡住检测（挂起时间 > 1 小时）
- 交易加速 UI（增加 Gas）

### 2.F.5 对账工具

**影响模块：** Admin

**新模块：** `ReconciliationEngine`

**技术要求：**
```go
type ReconciliationEngine interface {
    // 运行期间对账
    Reconcile(from, to time.Time) (ReconciliationReport, error)

    // 查找差异
    FindDiscrepancies(chain ChainType) ([]Discrepancy, error)

    // 将差异标记为已解决
    ResolveDiscrepancy(discrepancyID string, resolution string) error

    // 导出对账报告
    ExportReport(reportID string, format string) ([]byte, error)
}

type Discrepancy struct {
    ID              string
    Type            string  // "missing_on_chain", "missing_in_ledger", "amount_mismatch"
    Chain           ChainType
    TxHash          string
    LedgerEntryID   *int64
    OnChainAmount   *big.Int
    LedgerAmount    *big.Int
    DetectedAt      time.Time
    Status          string
}
```

**数据库模式：**
```sql
CREATE TABLE reconciliation_reports (
    id BIGINT PRIMARY KEY,
    period_start DATETIME NOT NULL,
    period_end DATETIME NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH', 'ALL') NOT NULL,
    total_on_chain_txs INT NOT NULL,
    total_ledger_entries INT NOT NULL,
    matched_count INT NOT NULL,
    discrepancy_count INT NOT NULL,
    status ENUM('pending', 'completed', 'failed') NOT NULL,
    created_at DATETIME NOT NULL,
    completed_at DATETIME
);

CREATE TABLE reconciliation_discrepancies (
    id BIGINT PRIMARY KEY,
    report_id BIGINT NOT NULL,
    type ENUM('missing_on_chain', 'missing_in_ledger', 'amount_mismatch', 'status_mismatch') NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    tx_hash VARCHAR(100),
    ledger_entry_id BIGINT,
    on_chain_amount DECIMAL(36, 18),
    ledger_amount DECIMAL(36, 18),
    status ENUM('unresolved', 'resolved', 'ignored') DEFAULT 'unresolved',
    resolution_notes TEXT,
    resolved_by BIGINT,
    resolved_at DATETIME,
    detected_at DATETIME NOT NULL,
    FOREIGN KEY (report_id) REFERENCES reconciliation_reports(id)
);
```

### 2.F.6 员工钱包地址管理（增强）

**影响模块：** Admin, Business

**当前状态：** 基本地址簿 (`wallet_addresses` 表)

**增强：**
- 为每位员工存储每条链的地址
- 地址验证工作流程（向地址发送 $0.01 以验证所有权）
- 地址变更审批（需要财务审批）
- 批量导入/导出（CSV）
- 地址白名单强制执行

**数据库模式：**
```sql
CREATE TABLE employee_wallet_addresses (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    address VARCHAR(64) NOT NULL,
    label VARCHAR(100),
    is_verified BOOLEAN DEFAULT FALSE,
    verified_at DATETIME,
    verification_tx_hash VARCHAR(100),
    is_active BOOLEAN DEFAULT TRUE,
    requires_approval BOOLEAN DEFAULT TRUE,
    approval_status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
    approved_by BIGINT,
    approved_at DATETIME,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    UNIQUE KEY uk_employee_chain (employee_id, chain),
    INDEX idx_address (address)
);

CREATE TABLE address_change_requests (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    old_address VARCHAR(64),
    new_address VARCHAR(64) NOT NULL,
    reason TEXT,
    status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
    reviewed_by BIGINT,
    reviewed_at DATETIME,
    review_notes TEXT,
    created_at DATETIME NOT NULL,
    FOREIGN KEY (employee_id) REFERENCES users(id)
);
```

---

## 2.G 监控与报告

### 2.G.1 薪资支付报告

**影响模块：** Admin

**新报告：**
- 按薪资周期、部门、成本中心汇总
- 按链和代币分类细分
- 员工个人支付历史 (YTD)
- 对比报告（当前与上一个周期，预算与实际）

**新 API：**
```go
type ReportGenerator interface {
    // 生成薪资汇总
    GeneratePayrollSummary(period PayPeriod, options ReportOptions) (Report, error)

    // 生成部门细分
    GenerateDepartmentBreakdown(period PayPeriod) (Report, error)

    // 生成员工 YTD
    GenerateEmployeeYTD(employeeID string, year int) (Report, error)

    // 生成对比报告
    GenerateComparison(period1, period2 PayPeriod) (Report, error)
}
```

### 2.G.2 成本分析报告

**影响模块：** Admin

**新报告：**
- 按链和周期划分的总交易费用
- 费用占总支付额的百分比
- 跨链费用比较
- 基于历史数据的费用预测

### 2.G.3 余额警报（增强）

**影响模块：** Admin

**当前状态：** 基本警报阈值

**增强：**
- 预测性警报（“3 天后余额将不足”）
- 警报升级（如果未确认，则升级）
- 多渠道通知（电子邮件、短信、Slack、PagerDuty）
- 警报历史和确认跟踪

**数据库模式：**
```sql
CREATE TABLE balance_alerts (
    id BIGINT PRIMARY KEY,
    alert_type ENUM('low_balance', 'high_balance', 'predicted_low', 'anomaly') NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    wallet_type ENUM('hot', 'cold') NOT NULL,
    current_balance DECIMAL(36, 18) NOT NULL,
    threshold_balance DECIMAL(36, 18),
    predicted_shortfall DECIMAL(36, 18),
    severity ENUM('info', 'warning', 'critical') NOT NULL,
    status ENUM('active', 'acknowledged', 'resolved') DEFAULT 'active',
    acknowledged_by BIGINT,
    acknowledged_at DATETIME,
    resolved_at DATETIME,
    created_at DATETIME NOT NULL,
    INDEX idx_status_severity (status, severity)
);

CREATE TABLE alert_notifications (
    id BIGINT PRIMARY KEY,
    alert_id BIGINT NOT NULL,
    channel ENUM('email', 'sms', 'slack', 'pagerduty', 'webhook') NOT NULL,
    recipient VARCHAR(255) NOT NULL,
    sent_at DATETIME,
    delivery_status ENUM('pending', 'sent', 'failed') NOT NULL,
    error_message TEXT,
    FOREIGN KEY (alert_id) REFERENCES balance_alerts(id)
);
```

### 2.G.4 失败交易报告

**影响模块：** Admin

**新报告：**
- 失败交易仪表板（实时）
- 根本原因分析（按错误类型分类）
- 重试状态跟踪
- 失败交易解决工作流程

**新 API：**
- 仪表板：`GET /api/v1/admin/transactions/failed/dashboard`
- 分析：`GET /api/v1/admin/transactions/failed/analysis?period=30d`

---

# 3. 优先级矩阵

## 3.1 Tier 1 (MVP - 必须拥有)

| # | 功能 | 理由 | 复杂度 | 依赖项 | 不实施的风险 |
|---|---|---|---|---|---|
| 1 | **多链批量支付处理** | 核心薪资功能 - 必须支持 TRON, BSC, ETH 同时进行 | 高 | ChainRPC, Signer 已支持多链 | 无法执行基本薪资 |
| 2 | **员工钱包地址管理** | 必须知道发送到哪里 | 中 | Users 表, ChainRPC 验证 | 无法识别支付接收者 |
| 3 | **支付状态跟踪** | 财务团队必须跟踪支付状态 | 中 | ChainSync 事件 | 对支付成功/失败没有可见性 |
| 4 | **基本审批工作流程** | 金融控制的挂牌-核对 | 低 | 已存在，增强 | SOX 合规失败，欺诈风险 |
| 5 | **交易重试机制** | 区块链交易会失败；必须有恢复机制 | 中 | ChainRPC | 支付卡住，需要手动干预 |
| 6 | **金库余额监控** | 在薪资前必须确保有足够资金 | 低 | 已存在 | 因资金不足导致薪资失败 |
| 7 | **审计日志** | 合规性要求 | 低 | 主要已存在，增强不变性 | 审计失败，法律风险 |
| 8 | **Gas 费估算** | 审批前必须显示成本 | 低 | ChainRPC 具备此功能 | 成本意外，无成本可见性的审批 |

## 3.2 Tier 2 (第 2 阶段 - 应该拥有)

| # | 功能 | 理由 | 复杂度 | 依赖项 | 不实施的风险 |
|---|---|---|---|---|---|
| 9 | **定期/循环支付** | 大多数薪资是定期发生的（双周、月度） | 中 | Tier 1 批量处理 | 每个发薪日手动创建批次 |
| 10 | **支付模板** | 重复使用配置，减少错误 | 中 | 员工地址管理 | 手动数据输入错误 |
| 11 | **多级审批** | 针对不同金额的不同审批 | 中 | 基本审批 | 对大额支付控制不足 |
| 12 | **Gas 费优化** | 降低跨链成本 | 中 | Gas 估算 | 交易成本高于必要水平 |
| 13 | **聚合余额报告** | 跨链的单一视图 | 中 | ChainSync | 金库视图分散 |
| 14 | **Webhook 通知** | 与 HR/ERP 系统集成 | 低 | 状态跟踪 | 手动导出数据到其他系统 |
| 15 | **对账工具** | 将链上记录与内部记录匹配 | 高 | Ledger, ChainSync | 手动对账，审计发现 |
| 16 | **速度限制** | 防止失控交易 | 中 | 审计日志 | 欺诈或错误可能耗尽金库 |

## 3.3 Tier 3 (未来 - 拥有更好)

| # | 功能 | 理由 | 复杂度 | 依赖项 | 不实施的风险 |
|---|---|---|---|---|---|
| 17 | **多重签名支持** | 对大额转账提供额外安全性 | 高 | Signer, Admin | 安全性降低（单点故障） |
| 18 | **硬件钱包集成** | 增强冷钱包安全性 | 高 | Signer | 仅软件密钥管理 |
| 19 | **热钱包自动补给** | 减少手动金库操作 | 中 | 金库监控 | 手动热钱包管理 |
| 20 | **高级报告** | 部门细分、YTD、对比 | 中 | 基本报告 | 手动 Excel 分析 |
| 21 | **员工自助门户** | 员工查看自己的支付状态 | 中 | 状态跟踪 | 询问状态的支持工单 |
| 22 | **预测余额警报** | 主动金库管理 | 中 | 余额监控 | 被动而非主动 |
| 23 | **Layer 2 支持** | 在 Polygon/Arbitrum 上降低费用 | 高 | 多链架构 | 仅在 L1 上支付高额费用 |
| 24 | **跨链费用比较** | 为每笔支付优化链选择 | 中 | 费用估算 | 链选择次优 |

---

# 4. 架构建议

## 4.1 多链架构

### 抽象层设计

**建议的链无关接口：**
```go
// pkg/chain/interface.go
package chain

type ChainAdapter interface {
    // 身份识别
    GetChainType() ChainType
    GetChainName() string

    // 地址操作
    ValidateAddress(address string) (bool, error)
    GenerateAddress(pubKey []byte) (string, error)

    // 余额操作
    GetNativeBalance(address string) (*big.Int, error)
    GetTokenBalance(address, tokenContract string) (*big.Int, error)

    // 交易构建
    BuildNativeTransfer(from, to string, amount *big.Int, gasPrice *big.Int) (*UnsignedTx, error)
    BuildTokenTransfer(from, to, tokenContract string, amount *big.Int, gasPrice *big.Int) (*UnsignedTx, error)

    // 费用估算
    EstimateGas(tx *UnsignedTx) (uint64, error)
    GetGasPrice(speed GasSpeed) (*big.Int, error)

    // 交易提交
    BroadcastTransaction(signedTx []byte) (txHash string, error)
    GetTransactionStatus(txHash string) (*TxStatus, error)
    GetTransactionReceipt(txHash string) (*TxReceipt, error)

    // 链特定
    GetConfirmationRequirement(amount *big.Int) int
}

// 共享数据结构
type UnsignedTx struct {
    ChainType    ChainType
    RawTx        []byte      // 链特定序列化
    From         string
    To           string
    Value        *big.Int
    GasPrice     *big.Int
    GasLimit     uint64
    Nonce        uint64
    Data         []byte
}

type TxStatus struct {
    Hash          string
    Status        TransactionStatus  // pending, confirmed, failed
    Confirmations int
    BlockNumber   uint64
    Timestamp     time.Time
}
```

### 链适配器模式

**实现结构：**
```
pkg/chain/
├── interface.go          # ChainAdapter 接口
├── factory.go            # GetAdapter(chainType) ChainAdapter
├── types.go              # 共享类型 (UnsignedTx, TxStatus, etc.)
├── ethereum/
│   ├── adapter.go        # EthereumAdapter 实现 ChainAdapter
│   ├── gas.go            # EIP-1559 Gas 处理
│   └── client.go         # go-ethereum 客户端包装器
├── bsc/
│   ├── adapter.go        # BSCAdapter 实现 ChainAdapter
│   └── client.go         # BSC 特定客户端
└── tron/
    ├── adapter.go        # TronAdapter 实现 ChainAdapter
    ├── resource.go       # 能量/带宽处理
    └── client.go         # gotron-sdk 包装器
```

**工厂模式：**
```go
// pkg/chain/factory.go
func GetAdapter(chainType ChainType, config ChainConfig) (ChainAdapter, error) {
    switch chainType {
    case ChainTypeTRON:
        return tron.NewAdapter(config.TronConfig)
    case ChainTypeBSC:
        return bsc.NewAdapter(config.BSCConfig)
    case ChainTypeEthereum:
        return ethereum.NewAdapter(config.EthConfig)
    default:
        return nil, ErrUnsupportedChain
    }
}
```

### 交易路由

**PayrollOrchestrator 实现：**
```go
// pkg/payroll/orchestrator.go
type PayrollOrchestrator struct {
    adapters     map[ChainType]chain.ChainAdapter
    signer       signer.SignerClient
    statusTracker *StatusTracker
}

func (o *PayrollOrchestrator) SubmitMultiChainPayroll(ctx context.Context, batch PayrollBatch) (*MultiChainResult, error) {
    // 按链分组支付项目
    byChain := groupByChain(batch.Items)

    // 并行提交到每条链
    var wg sync.WaitGroup
    results := make(chan ChainResult, len(byChain))

    for chainType, items := range byChain {
        wg.Add(1)
        go func(ct ChainType, payItems []PayrollItem) {
            defer wg.Done()
            result := o.submitToChain(ctx, ct, payItems)
            results <- result
        }(chainType, items)
    }

    wg.Wait()
    close(results)

    return aggregateResults(results), nil
}
```

### 状态同步

**事件驱动的状态更新：**
```go
// 消费 ChainSync Kafka 事件
func (s *StateSync) HandleDepositConfirmed(event DepositConfirmedEvent) error {
    // 更新内部余额
    return s.ledger.RecordDeposit(LedgerEntry{
        UserID:    event.UserID,
        Chain:     event.Chain,
        TxHash:    event.TxHash,
        Amount:    event.Amount,
        Status:    "confirmed",
        Timestamp: event.Timestamp,
    })
}

// 用于后备的交易状态轮询
func (s *StateSync) PollTransactionStatus(txHash string, chain ChainType) {
    adapter := s.adapters[chain]
    ticker := time.NewTicker(10 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            status, err := adapter.GetTransactionStatus(txHash)
            if err != nil {
                continue
            }
            if status.Status == Confirmed {
                s.updateInternalStatus(txHash, status)
                return
            }
        case <-s.stopCh:
            return
        }
    }
}
```

## 4.2 关注点分离

### 模块边界

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        薪资领域边界                                         │
│                                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                │
│  │ PayrollService │  │ TemplateService│  │ ScheduleService│                │
│  │                │  │                │  │                │                │
│  │ - BatchCreate  │  │ - CRUD         │  │ - CronTrigger  │                │
│  │ - BatchProcess │  │ - Versioning   │  │ - NextRun      │                │
│  │ - StatusQuery  │  │ - Validation   │  │ - Pause/Resume │                │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘                │
│          │                   │                   │                          │
└──────────┼───────────────────┼───────────────────┼──────────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        金库领域边界                                        │
│                                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                │
│  │TreasuryService │  │ HotWalletMgr   │  │ ColdWalletMgr  │                │
│  │                │  │                │  │                │                │
│  │ - VaultBalance │  │ - Replenish    │  │ - OfflineSign  │                │
│  │ - Allocation   │  │ - Rotate       │  │ - ImportSig    │                │
│  │ - Ledger       │  │ - Drain        │  │ - WatchOnly    │                │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘                │
│          │                   │                   │                          │
└──────────┼───────────────────┼───────────────────┼──────────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      区块链领域边界                                        │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │                      BlockchainGatewayService                       │    │
│  │                                                                     │    │
│  │  - 链无关交易提交                                                  │    │
│  │  - 跨链余额查询                                                    │    │
│  │  - 费用估算和优化                                                  │    │
│  │  - 交易状态跟踪                                                    │    │
│  └───────────────────────────────┬────────────────────────────────────┘    │
│                                  │                                          │
│        ┌─────────────────────────┼─────────────────────────┐               │
│        ▼                         ▼                         ▼               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                     │
│  │ TronAdapter  │  │  BSCAdapter  │  │  EthAdapter  │                     │
│  └──────────────┘  └──────────────┘  └──────────────┘                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      签名领域边界 (隔离)                                   │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │                         SignerService                               │    │
│  │                                                                     │    │
│  │  - 种子管理（加密）                                              │    │
│  │  - 地址派生                                                        │    │
│  │  - 交易签名                                                        │    │
│  │  - 无出站网络访问                                                  │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  安全：仅内部 gRPC，无公共访问，单独的网络段                       │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 共享服务

**被两个域使用的服务：**
- `NotificationService` - 电子邮件、短信、推送通知
- `AuditService` - 集中式审计日志收集
- `ConfigService` - 功能标志、系统配置

### API 边界

**服务间通信：**
```protobuf
// proto/payroll/v1/payroll.proto
service PayrollService {
    rpc CreateBatch(CreateBatchRequest) returns (CreateBatchResponse);
    rpc ProcessBatch(ProcessBatchRequest) returns (ProcessBatchResponse);
    rpc GetBatchStatus(GetBatchStatusRequest) returns (GetBatchStatusResponse);
}

// proto/treasury/v1/treasury.proto
service TreasuryService {
    rpc GetVaultBalance(GetVaultBalanceRequest) returns (GetVaultBalanceResponse);
    rpc RequestReplenishment(ReplenishmentRequest) returns (ReplenishmentResponse);
}

// proto/blockchain/v1/blockchain.proto
service BlockchainGatewayService {
    rpc SubmitTransaction(SubmitTxRequest) returns (SubmitTxResponse);
    rpc GetBalance(GetBalanceRequest) returns (GetBalanceResponse);
    rpc EstimateFee(EstimateFeeRequest) returns (EstimateFeeResponse);
}
```

## 4.3 数据库模式更改

### 新表总结

| 表 | 目的 | 级别 |
|---|---|---|
| `payroll_schedules` | 定期支付计划 | Tier 2 |
| `scheduled_executions` | 计划执行历史 | Tier 2 |
| `payroll_templates` | 保存的支付配置 | Tier 2 |
| `template_history` | 模板版本历史 | Tier 2 |
| `employee_wallet_addresses` | 每条链的员工地址 | Tier 1 |
| `address_change_requests` | 地址变更审批 | Tier 1 |
| `approval_chains` | 多级审批配置 | Tier 2 |
| `approval_chain_levels` | 审批级别定义 | Tier 2 |
| `approval_chain_approvers` | 每级的审批人 | Tier 2 |
| `payroll_approvals` | 批次审批记录 | Tier 2 |
| `transaction_retries` | 重试尝试跟踪 | Tier 1 |
| `dead_letter_transactions` | 失败交易队列 | Tier 1 |
| `velocity_limits` | 交易限制配置 | Tier 2 |
| `velocity_usage` | 限制使用情况跟踪 | Tier 2 |
| `multisig_configs` | 多签配置 | Tier 3 |
| `multisig_signers` | 多签签名者列表 | Tier 3 |
| `multisig_requests` | 待处理的多签请求 | Tier 3 |
| `multisig_signatures` | 收集到的签名 | Tier 3 |
| `reconciliation_reports` | 对账运行 | Tier 2 |
| `reconciliation_discrepancies` | 发现的差异 | Tier 2 |
| `balance_alerts` | 警报实例 | Tier 2 |
| `alert_notifications` | 警报交付跟踪 | Tier 2 |
| `webhook_configs` | 外部 Webhook 配置 | Tier 2 |
| `webhook_deliveries` | Webhook 交付历史 | Tier 2 |
| `audit_log_chain` | 不可变审计链 | Tier 1 |
| `hot_wallet_config` | 热钱包设置 | Tier 1 |
| `aggregated_balances` | 跨链余额视图 | Tier 2 |
| `derivation_registry` | HD 路径分配 | Tier 1 |
| `employee_payment_preferences` | 员工链/代币偏好 | Tier 2 |
| `ledger_accounts` | 账户层级结构 | Tier 3 |
| `permissions` | 粒度权限 | Tier 2 |
| `role_permissions` | 角色-权限映射 | Tier 2 |

### 模式修改

```sql
-- 增强 payroll_items 带有重试和链信息
ALTER TABLE payroll_items
ADD COLUMN chain ENUM('TRON', 'BSC', 'ETH') NOT NULL DEFAULT 'TRON',
ADD COLUMN token_address VARCHAR(64),
ADD COLUMN retry_count INT DEFAULT 0,
ADD COLUMN last_retry_at DATETIME,
ADD COLUMN gas_used DECIMAL(20, 8),
ADD COLUMN actual_fee DECIMAL(20, 8),
ADD COLUMN confirmation_depth INT DEFAULT 0;

-- 增强 ledger_entries 以实现复式记账
ALTER TABLE ledger_entries
ADD COLUMN debit_account_id BIGINT,
ADD COLUMN credit_account_id BIGINT,
ADD COLUMN chain ENUM('TRON', 'BSC', 'ETH');

-- 增强 users 以实现员工钱包地址
-- (使用单独的 employee_wallet_addresses 表代替)
```

### 索引策略

```sql
-- 交易历史性能
CREATE INDEX idx_payroll_items_status_chain ON payroll_items(status, chain, created_at);
CREATE INDEX idx_payroll_items_employee ON payroll_items(employee_id, created_at);

-- 批处理
CREATE INDEX idx_payroll_batches_status_scheduled ON payroll_batches(status, scheduled_at);

-- 重试管理
CREATE INDEX idx_transaction_retries_status ON transaction_retries(status, attempted_at);

-- 审计查询
CREATE INDEX idx_audit_log_chain_user ON audit_log_chain(user_id, timestamp);
CREATE INDEX idx_audit_log_chain_resource ON audit_log_chain(resource_type, resource_id, timestamp);

-- 余额监控
CREATE INDEX idx_aggregated_balances_wallet ON aggregated_balances(wallet_type, chain);

-- 地址查找
CREATE INDEX idx_employee_addresses_address ON employee_wallet_addresses(address);
```

### 数据分区

```sql
-- 按月分区 transaction_retries
ALTER TABLE transaction_retries
PARTITION BY RANGE (YEAR(attempted_at) * 100 + MONTH(attempted_at)) (
    PARTITION p202512 VALUES LESS THAN (202601),
    PARTITION p202601 VALUES LESS THAN (202602),
    -- ... 根据需要添加分区
    PARTITION pmax VALUES LESS THAN MAXVALUE
);

-- 归档旧的薪资数据（2 年后）
CREATE TABLE payroll_items_archive LIKE payroll_items;
-- 计划任务以移动旧数据
```

## 4.4 服务架构

### 微服务与单体推荐

**建议：初始采用模块化单体，按需提取**

**理由：**
1. 当前系统已经是 Go-Zero 微服务 - 保持该模式
2. 薪资是一个新领域 - 作为 Admin Service 的模块启动
3. 如果满足以下条件，则提取为独立服务：独立扩展、不同团队、不同部署周期

**建议的服务结构：**
```
services/
├── api-gateway/          # 现有
├── business/             # 现有 - 用户操作
├── chainrpc/             # 现有 - 区块链操作
├── chainsync/            # 现有 - 区块链监控
├── signer/               # 现有 - 密钥管理
├── admin/                # 现有 - 增强薪资功能
│   ├── payroll/          # NEW: 薪资模块
│   │   ├── batch.go
│   │   ├── schedule.go
│   │   ├── template.go
│   │   └── approval.go
│   ├── treasury/         # NEW: 金库模块
│   │   ├── vault.go
│   │   ├── hot_wallet.go
│   │   └── reconciliation.go
│   └── reporting/        # NEW: 报告模块
└── notification/         # 如果流量高则提取
```

### 服务间通信

**同步 (gRPC)：**
- Admin → ChainRPC：余额查询、交易提交
- Admin → Signer：地址生成（用于员工地址）
- Business → Admin：员工数据查找

**异步 (Kafka)：**
- ChainSync → Admin：交易确认事件
- Admin → Notification：发送警报
- Admin → Audit：记录事件

**事件模式：**
```json
{
  "event_id": "EVT-20251210-ABC123",
  "event_type": "payroll.batch.processed",
  "timestamp": "2025-12-10T09:00:00Z",
  "payload": {
    "batch_id": "PAY-20251210-001",
    "total_items": 150,
    "success_count": 148,
    "failed_count": 2,
    "total_amount": "150000.00"
  },
  "metadata": {
    "operator_id": "10001",
    "correlation_id": "REQ-XYZ"
  }
}
```

## 4.5 可扩展性设计

### 批量处理可扩展性

```go
type BatchWorker struct {
    workerID      int
    inputCh       chan PayrollItem
    resultCh      chan ProcessResult
    adapters      map[ChainType]chain.ChainAdapter
    signer        signer.SignerClient
    concurrency   int  // 每条链并发处理的交易数
}

func (s *BatchService) ProcessLargeBatch(ctx context.Context, batch PayrollBatch) error {
    // 将批次分块
    chunks := chunkItems(batch.Items, s.chunkSize)  // 每个块 100 个项目

    // 创建工作池
    pool := NewWorkerPool(s.workerCount)  // 10 个工作进程

    // 提交块
    for _, chunk := range chunks {
        pool.Submit(func() error {
            return s.processChunk(ctx, chunk)
        })
    }

    // 带超时等待
    return pool.Wait(ctx)
}
```

### 水平扩展

**无状态服务（可水平扩展）：**
- API 网关（负载均衡）
- Admin Service（读取副本用于查询）
- ChainRPC Service（每条链多个实例）
- Notification Service

**有状态服务（小心扩展）：**
- Signer Service：首选单个实例（密钥一致性）
- ChainSync Service：按链分区（每条链一个实例）

### 缓存策略

```go
// Redis 缓存配置
type CacheConfig struct {
    // 余额缓存 - 30 秒 TTL
    BalanceTTL: 30 * time.Second,

    // Gas 价格缓存 - 10 秒 TTL
    GasPriceTTL: 10 * time.Second,

    // 汇率缓存 - 60 秒 TTL
    ExchangeRateTTL: 60 * time.Second,

    // 员工地址缓存 - 5 分钟 TTL
    EmployeeAddressTTL: 5 * time.Minute,

    // 薪资模板缓存 - 10 分钟 TTL
    TemplateTTL: 10 * time.Minute,
}

// 缓存失效事件
func (c *CacheManager) InvalidateOnEvent(event Event) {
    switch event.Type {
    case "balance.changed":
        c.Delete(fmt.Sprintf("balance:%s:%s", event.Chain, event.Address))
    case "template.updated":
        c.Delete(fmt.Sprintf("template:%s", event.TemplateID))
    }
}
```

### 队列管理

```go
// 队列配置
type QueueConfig struct {
    // 交易提交队列
    TxSubmission: QueueSettings{
        Name:        "payroll.tx.submit",
        Concurrency: 10,
        MaxRetries:  3,
        RetryDelay:  time.Minute,
    },

    // 交易监控队列
    TxMonitor: QueueSettings{
        Name:        "payroll.tx.monitor",
        Concurrency: 50,
        MaxRetries:  100,  // 保持监控数小时
        RetryDelay:  10 * time.Second,
    },

    // 紧急支付的高优先级队列
    HighPriority: QueueSettings{
        Name:        "payroll.tx.priority",
        Concurrency: 5,
        MaxRetries:  5,
        RetryDelay:  30 * time.Second,
    },
}
```

## 4.6 安全架构

### 密钥管理服务 (KMS)

**建议：使用云 KMS 来加密密钥，使用本地 HSM 进行高价值操作**

```
┌─────────────────────────────────────────────────────────────────────┐
│                        密钥层级                                    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  主密钥 (AWS KMS / Google Cloud KMS / HashiCorp Vault)      │   │
│  │  - 从不离开 KMS                                             │   │
│  │  - 用于加密/解密 DEK                                        │   │
│  └─────────────────────────────┬───────────────────────────────┘   │
│                                │                                    │
│                                ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  数据加密密钥 (DEK) - 加密后存储在 DB 中                 │   │
│  │  - 种子加密密钥                                             │   │
│  │  - 备份加密密钥                                             │   │
│  └─────────────────────────────┬───────────────────────────────┘   │
│                                │                                    │
│                                ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  受保护的数据                                              │   │
│  │  - master_seeds.seed_encrypted                              │   │
│  │  - 备份文件                                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 静态数据加密

| 数据类型 | 加密 | 存储 |
|---|---|---|
| 主种子 | AES-256-GCM + PBKDF2 | MariaDB（加密列） |
| 备份助记词 | AES-256-GCM | 安全文件存储 |
| 敏感 PII | AES-256-GCM | MariaDB（加密列） |
| 审计日志 | 透明加密 | MariaDB TDE |
| Redis 数据 | Redis 静态加密 | Redis Enterprise |

### 传输中加密

- 所有 API 流量：TLS 1.3
- 服务间：mTLS（双向 TLS）
- 数据库连接：需要 TLS
- Redis 连接：需要 TLS

### 签名服务隔离

```
┌─────────────────────────────────────────────────────────────────────┐
│                        生产网络                                    │
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │ API 网关    │    │   Admin     │    │  ChainRPC   │             │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘             │
│         │                  │                  │                     │
│         └──────────────────┴──────────────────┘                     │
│                            │                                        │
│                     内部负载均衡器                                  │
│                            │                                        │
└────────────────────────────┼────────────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    │  防火墙/安全组  │
                    │  - 仅 gRPC      │
                    │  - IP 白名单    │
                    └────────┬────────┘
                             │
┌────────────────────────────┼────────────────────────────────────────┐
│                    隔离签名网络                                    │
│                                                                     │
│         ┌──────────────────────────────────────────┐               │
│         │           Signer Service                  │               │
│         │                                          │               │
│         │  - 无出站互联网访问                        │               │
│         │  - 仅接受内部 gRPC                       │               │
│         │  - 冷密钥的 HSM 集成                     │               │
│         │  - 审计所有操作                            │               │
│         └──────────────────────────────────────────┘               │
│                            │                                        │
│                    ┌───────┴───────┐                               │
│                    │   HSM/TEE     │                               │
│                    │ （可选）      │                               │
│                    └───────────────┘                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 密钥管理

**建议：HashiCorp Vault 或云原生密钥管理器**

```go
// 需要管理的密钥
type SecretsConfig struct {
    // 数据库凭证
    DBHost     string `vault:"secret/data/db#host"`
    DBUser     string `vault:"secret/data/db#username"`
    DBPassword string `vault:"secret/data/db#password"`

    // Redis 凭证
    RedisPassword string `vault:"secret/data/redis#password"`

    // JWT 签名密钥
    JWTSecret string `vault:"secret/data/jwt#secret"`

    // 种子加密密码
    SeedEncryptionPassword string `vault:"secret/data/signer#encryption_password"`

    // RPC 提供者 API 密钥
    InfuraAPIKey    string `vault:"secret/data/rpc#infura_key"`
    AlchemyAPIKey   string `vault:"secret/data/rpc#alchemy_key"`
    QuickNodeAPIKey string `vault:"secret/data/rpc#quicknode_key"`

    // 通知服务凭证
    SMTPPassword    string `vault:"secret/data/notification#smtp_password"`
    TwilioAuthToken string `vault:"secret/data/notification#twilio_token"`
}
```

---

# 5. 实施路线图

## 阶段 1：MVP（第 1-8 周）

**目标：** 实现基本的跨链薪资支付能力

### 第 1-2 周：基础建设
- [ ] 员工钱包地址管理（CRUD，验证）
- [ ] 每条链的热钱包配置
- [ ] 增强的审计日志（不可变链）

### 第 3-4 周：多链批量处理
- [ ] 链适配器抽象层
- [ ] PayrollOrchestrator 实现
- [ ] 跨 3 条链的并行交易提交
- [ ] 每条链的 Nonce 管理

### 第 5-6 周：状态与重试
- [ ] 实时支付状态跟踪
- [ ] 交易重试机制
- [ ] 失败交易的死信队列
- [ ] 基本仪表板更新

### 第 7-8 周：测试与加固
- [ ] 所有链的集成测试
- [ ] 负载测试（1000+ 接收者）
- [ ] 安全审计
- [ ] 文档编制

**交付成果：**
- 跨链薪资批量处理
- 员工地址管理
- 交易重试和状态跟踪
- 增强的审计日志

**成功标准：**
- 成功处理跨 3 条链的 1000 接收者批次
- 99% 的交易成功率
- 所有操作的完整审计跟踪

**团队构成：**
- 2 名高级后端工程师 (Go)
- 1 名区块链工程师（多链经验）
- 1 名 QA 工程师
- 0.5 名 DevOps 工程师

---

## 阶段 2：增强（第 9-16 周）

**目标：** 运营效率和合规性功能

### 第 9-10 周：调度与模板
- [ ] PayrollScheduler 实现
- [ ] 基于 Cron 的定期支付
- [ ] 支付模板管理
- [ ] 模板版本控制

### 第 11-12 周：高级审批
- [ ] 多级审批工作流程
- [ ] 审批委托
- [ ] 基于时间的自动批准
- [ ] 审批审计跟踪

### 第 13-14 周：优化与监控
- [ ] Gas 费优化
- [ ] 聚合余额仪表板
- [ ] 速度限制
- [ ] 带有升级机制的余额警报

### 第 15-16 周：集成与对账
- [ ] Webhook 通知
- [ ] 基本对账工具
- [ ] 高级报告
- [ ] ERP 集成存根

**交付成果：**
- 定期/循环薪资
- 支付模板
- 多级审批
- Gas 优化
- 对账工具

**成功标准：**
- 自动执行双周薪资
- 手动操作减少 20%
- 对账无差异
- SOX 合规的审批工作流程

**团队构成：**
- 2 名高级后端工程师
- 1 名前端工程师（仪表板）
- 1 名 QA 工程师
- 0.5 名 DevOps 工程师

---

## 阶段 3：优化（第 17-24 周）

**目标：** 高级安全和可扩展性

### 第 17-18 周：多重签名
- [ ] 多签配置
- [ ] 签名收集工作流程
- [ ] Gnosis Safe 集成 (ETH/BSC)
- [ ] TRON 多签合约

### 第 19-20 周：高级安全
- [ ] 硬件钱包集成
- [ ] 增强的 RBAC
- [ ] 异常检测
- [ ] 安全仪表板

### 第 21-22 周：可扩展性
- [ ] 水平扩展实现
- [ ] 数据库分区
- [ ] 缓存优化
- [ ] 性能调优

### 第 23-24 周：未来准备
- [ ] Layer 2 准备（接口）
- [ ] 高级报告
- [ ] 员工自助门户
- [ ] 文档和培训

**交付成果：**
- 多重签名支持
- 硬件钱包集成
- 生产级可扩展性
- 员工门户

**成功标准：**
- 多签操作用于 >$100K 的交易
- 支持 10,000+ 员工
- 99.9% 的正常运行时间
- 完整的员工自助服务

**团队构成：**
- 2 名高级后端工程师
- 1 名安全工程师
- 1 名前端工程师
- 1 名 QA 工程师
- 1 名 DevOps/SRE 工程师

---

## 依赖关系图

```
第 1 阶段 (MVP)
├── 员工地址管理 ───────────────────────────────────┐
├── 多链批量处理 ───────────────────────────┐            │
├── 交易重试 ───────────────────────────────┼──────────│
└── 审计日志 ──────────────────────────────────│            │
                                      │            │
第 2 阶段 (增强)                      ▼            │
├── 调度 ◄───────────────── 批量处理               │
├── 模板 ◄───────────────── 批量处理               │
├── 多级审批 ◄────────────── 审计日志               │
├── Gas 优化 ◄────────────── 多链批量               │
├── 对账 ◄─────────────── 审计日志               │
└── 余额警报 ◄────────────── 金库监控               │
                                      │            │
第 3 阶段 (优化)                      ▼            │
├── 多签 ◄───────────────── 审批工作流程             │
├── 硬件钱包 ◄────────────── 多签                   │
├── 可扩展性 ◄────────────── 第 1 & 2 阶段所有内容   │
└── 员工门户 ◄────────────── 地址管理 ◄───────────┘
```

---

# 6. 关键文件参考

## 需要修改的现有文件

| 文件/位置 | 修改内容 |
|---|---|
| `admin-api/03-薪资发放.md` | 增强以支持多链、重试、调度 |
| `admin-api/07-账本与Vault.md` | 添加对账、增强警报 |
| `admin-api/99-数据模型与配置.md` | 添加新表 |
| `wallet-core/srds/03-chainrpc-service/02-交易构建.md` | 增强批量构建 |
| `wallet-core/srds/05-signer-service/02-地址生成.md` | 员工地址生成 |
| `wallet-core/srds/06-common-infrastructure/06-错误码.md` | 添加薪资错误代码 |

## 需要创建的新文件

| 文件/位置 | 目的 |
|---|---|
| `pkg/chain/interface.go` | 链适配器接口 |
| `pkg/chain/factory.go` | 适配器工厂 |
| `pkg/chain/ethereum/adapter.go` | 以太坊实现 |
| `pkg/chain/bsc/adapter.go` | BSC 实现 |
| `pkg/chain/tron/adapter.go` | TRON 实现 |
| `services/admin/payroll/orchestrator.go` | 多链编排器 |
| `services/admin/payroll/scheduler.go` | 定期支付 |
| `services/admin/payroll/template.go` | 模板管理 |
| `services/admin/treasury/reconciliation.go` | 对账引擎 |
| `migrations/20251210_payroll_extensions.sql` | 数据库迁移 |

---
