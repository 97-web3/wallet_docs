# Swap / Provider 管理（聚合通道）

> 返回 [Swap 配置索引](../06-Swap配置.md)

---

## 定位与原则

- **目标**：面向运维/运营团队，对接并管理第三方 Swap 流动性聚合通道（Provider/Channel），而非自建资金池。
- **重要说明**：本模块支持 **EVM 链**（Ethereum/BSC/Polygon 等）与 **Tron**；Provider 可按链、按交易对进行路由与降级。
- **术语约定**：
  - **Provider/Channel**：第三方聚合器或报价/交易通道（例如 1inch/0x/ParaSwap 类）。
  - **DEX**：Provider 返回的底层路由路径中可能包含的去中心化交易所名称（例如 UniswapV3/Curve）。

---

## 页面入口

- **页面路径**：`/admin/swap`
- **建议信息架构**：在 Swap 配置页面中，将“DEX 管理”语义升级为 **Provider 管理**（不改变页面入口，仅改变模块命名与字段解释）。

---

## 1. Provider 列表

### 1.1 UI 结构（参考）

```
┌──────────────────────────────────────────────────────────────────────────┐
│  Provider 管理                                             [添加 Provider] │
├──────────────────────────────────────────────────────────────────────────┤
│ │Provider名称│链/网络│类型│Endpoint│状态│优先级│配额│操作(编辑/禁用)│  │
│ ├───────────┼──────┼────┼────────┼────┼──────┼────┼───────────────┤  │
│ │1inch      │EVM   │agg │https://…│健康│1     │…   │[编辑] [禁用]  │  │
│ │0x         │EVM   │agg │https://…│降级│2     │…   │[编辑] [禁用]  │  │
│ │SunSwapAgg │TRON  │agg │https://…│不可用│1  │…   │[编辑] [重试]  │  │
└──────────────────────────────────────────────────────────────────────────┘
```

### 1.2 建议表格字段（生产必需）

- **基础信息**
  - Provider ID（唯一标识）
  - Provider 名称
  - network + chain_id（Tron 在 SRD 示例中 `chain_id = -1`）
  - 类型：Aggregator / Direct DEX / RFQ（如适用）
  - Endpoint（Base URL）
- **开关与路由**
  - enabled（启用/禁用）
  - priority（优先级，数字越小优先级越高）
  - 适用范围：全局/按链/按交易对
- **健康与配额（运维核心）**
  - 状态：healthy / degraded / unavailable / maintenance
  - 健康检查（与 SRD 字段对齐）：
    - `health_check.latency_ms`
    - `health_check.success_rate_24h`
    - `health_check.last_error`
  - （可选扩展指标）错误率、P95/P99 延迟、分链成功率等
  - 配额/限流：当前窗口用量、上限、触发策略（告警/自动降级）

#### 1.2.1 SRD 字段对齐（来源：`GET /api/v1/admin/swap/providers` 与 `PUT /api/v1/admin/swap/providers/{id}`）

| UI/文档字段 | SRD 路径/字段 | 备注 |
|---|---|---|
| Provider ID | providers[].id / path `{id}` | 与 SRD 一致 |
| Provider 名称 | providers[].name | 与 SRD 一致 |
| network | providers[].network | 与 SRD 一致 |
| chain_id | providers[].chain_id | Tron 示例为 -1 |
| router_address | providers[].router_address | 仅直连 DEX 场景常见；聚合通道可为空或不展示 |
| enabled | providers[].enabled / (PUT).enabled | 与 SRD 一致 |
| priority | providers[].priority / (PUT).priority | 与 SRD 一致 |
| fee_tier | providers[].fee_tier / (PUT).fee_tier | 可选；直连 DEX 常用 |
| max_slippage | (PUT).max_slippage | Provider 级覆盖（如后端启用该策略） |
| daily_limit_usd | (PUT).daily_limit_usd | Provider 级限额（如后端启用该策略） |
| status | providers[].status | 枚举见 SRD（healthy/degraded/unavailable/maintenance） |
| health_check.* | providers[].health_check.* | latency_ms / success_rate_24h / last_error |

> 说明：原 `router_address/factory_address/quoter` 等“合约级字段”仅适用于直连 DEX 场景；在第三方聚合模型中，运营侧更需要 **Endpoint + Auth + 限流/配额 + 路由策略 + 健康指标**。

### 1.3 配置字段优先级（建议落地顺序）

| 优先级 | 字段/能力 | 目的 | 缺失风险 |
|---|---|---|---|
| P0 | Provider ID、名称、网络/链、启用开关、优先级 | 可识别、可路由、可关停 | 无法控制流量走向/无法应急 |
| P0 | Endpoint（Base URL） | 可访问 Provider | 全部报价/执行失败 |
| P0 | 认证方式 + `secret_ref`（密钥引用） | 安全接入与可轮换 | 密钥泄露/无法轮换 |
| P0 | 超时/重试（含退避） | 稳定性与降级基础 | 尖峰期失败率飙升 |
| P0 | 限流/配额参数（窗口、burst） | 成本控制与稳定性 | 触发 429 导致大面积失败 |
| P1 | 健康指标（latency/success_rate/error_rate） | 可观测性 | 事故定位缓慢 |
| P1 | 熔断阈值与降级策略 | 自动保护 | 雪崩/级联失败 |
| P1 | 按链/按交易对适用范围 | 精细化路由 | 某些链/对长期不可用 |
| P2 | 配额用量展示与告警阈值 | 运维效率 | 事后发现配额耗尽 |

---

## 1.4 Provider 上线最小闭环（Checklist）

用于指导 Ops/工程在接入新 Provider 时按顺序完成验证：

1. 创建 Provider（填写 P0 字段）并先置为“禁用”。
2. 配置认证（仅保存 `secret_ref`，不在文档/页面展示明文密钥）。
3. 设置超时/重试/限流的初始值（以 Provider 官方建议为准）。
4. 使用“测试查询”对至少 3 个典型交易对进行测试（稳定币对/主流币对/长尾币对）。
5. 校验返回：报价、折算口径、路由路径、预估费用、错误码映射。
6. 小流量启用（优先级设低或仅对特定 Pair 生效），观察 30–60 分钟。
7. 确认健康指标稳定后，再提升优先级/扩大适用范围。

---

## 2. 添加/编辑 Provider

### 2.1 表单结构（建议）

```
┌───────────────────────────────────────────┐
│ 添加/编辑 Provider                   [×]   │
├───────────────────────────────────────────┤
│ Provider 名称 *                          │
│ [________________________]               │
│ Provider ID *                            │
│ [________________________]               │
│ 网络/链 *                                │
│ [Ethereum ▼]  (network / chain_id)       │
│ 类型 *                                   │
│ [Aggregator ▼]                           │
│ Endpoint *                               │
│ [https://____________________]           │
│ 认证方式 *                               │
│ [API Key ▼] / OAuth / Signed Request     │
│ 认证信息（引用/占位）                    │
│ [secret_ref: vault://swap/...]           │
│ 超时(ms) / 重试次数 / 重试退避           │
│ [____]  [__]  [exponential]              │
│ 限流/配额                                │
│ [50/min] [burst 100]                     │
│ 优先级 *                                 │
│ [1]                                      │
│ 启用状态                                 │
│ [启用] [禁用]                            │
│ 备注                                     │
│ [________________________]               │
│              [取消]  [保存]              │
└───────────────────────────────────────────┘
```

### 2.2 验证规则（建议）

| 字段 | 规则 | 错误提示 |
|---|---|---|
| Provider 名称 | 必填，2-50 字符 | 请输入 Provider 名称 |
| Provider ID | 必填，字母/数字/`-`，全局唯一 | Provider ID 不合法或已存在 |
| 网络/链 | 必选 | 请选择网络/链 |
| Endpoint | 必填，URL 格式 | 请输入有效的 Endpoint |
| 认证方式 | 必选 | 请选择认证方式 |
| 优先级 | 必填，1-10（建议） | 优先级范围 1-10 |

---

## 3. 路由与降级（Fallback）策略（建议写入文档）

> 这一部分是生产必需，但当前 admin-web/06 文档未覆盖。

- **Provider 选择规则**（从运维角度可配置）
  - 按链映射：例如 ETH 优先 1inch，BSC 优先 0x，Tron 优先 SunSwapAgg。
  - 按交易对覆盖：某些 token pair 指定 Provider，避免不稳定渠道。
- **降级/熔断**
  - 连续失败次数阈值触发 `degraded/unavailable`
  - 延迟超阈值自动降级
  - 配额耗尽自动切换备用 Provider
- **回切策略**
  - 手动回切
  - 自动回切（成功率恢复/延迟恢复）

---

## 4. 健康检查与测试

### 4.1 健康状态指示（建议）

| 状态 | 说明 |
|---|---|
| healthy | 可稳定报价/可执行 |
| degraded | 报价可用但执行受限，或延迟偏高 |
| unavailable | 不可用，应触发降级与告警 |
| maintenance | 计划维护，不参与路由 |

### 4.2 “测试查询”对话框（沿用既有能力）

- 用途：在不实际执行交易的情况下发起报价/路径查询，用于验证 Provider 配置。
- 输出建议包含：
  - 预计获得数量
  - 报价（用于**折算**展示）
  - 价格影响
  - 路由路径（底层 DEX 组成）
  - 预估 Gas
  - Provider 返回码/错误码（失败时）

---

## 5. 对应后端接口（参考）

> 仅做文档映射，具体字段以 Admin API SRD 为准。

- `GET /api/v1/admin/swap/providers`
- `PUT /api/v1/admin/swap/providers/{id}`

（详见 `wallet_docs/admin-api/srds/06-Swap配置.md`）

---

> 返回 [Swap 配置索引](../06-Swap配置.md)
