# Swap 配置

> 返回 [后台管理服务目录](./README.md)

---

## 模块概述

Swap 配置模块为管理员提供闪兑功能的配置管理，包括 DEX 提供商管理、滑点规则配置、汇率接口设置等。

---

## F-ADMIN-040: 获取 Swap 配置

**描述**: 获取当前的 Swap 系统配置。

**gRPC 方法**: `GetSwapConfig`

**权限要求**: Admin 角色

**请求 Schema**:
```json
{}
```

**响应 Schema**:
```json
{
  "success": true,
  "config": {
    "enabled": true,
    "default_slippage": 0.005,        // 默认滑点 0.5%
    "min_slippage": 0.001,            // 最小滑点 0.1%
    "max_slippage": 0.50,             // 最大滑点 50%
    "preset_slippage_options": [0.001, 0.005, 0.01, 0.03],
    "min_swap_amount_usdt": "10.00",  // 最小兑换金额
    "max_swap_amount_usdt": "100000.00",  // 单笔最大金额
    "daily_swap_limit_usdt": "500000.00", // 单用户每日限额
    "platform_fee_rate": 0.003,       // 平台手续费 0.3%
    "quote_expiration_seconds": 30,   // 报价有效期
    "high_price_impact_threshold": 0.05,  // 高价格影响阈值 5%
    "providers": [
      {
        "id": "1inch",
        "name": "1inch",
        "enabled": true,
        "priority": 1,
        "supported_chains": ["ETH", "BSC"],
        "api_url": "https://api.1inch.io/v5.0",
        "status": "healthy",
        "last_health_check": "2025-12-10T10:00:00Z"
      },
      {
        "id": "paraswap",
        "name": "Paraswap",
        "enabled": true,
        "priority": 2,
        "supported_chains": ["ETH", "BSC"],
        "api_url": "https://apiv5.paraswap.io",
        "status": "healthy",
        "last_health_check": "2025-12-10T10:00:00Z"
      }
    ],
    "supported_tokens": {
      "ETH": ["ETH", "USDT", "USDC", "DAI", "WETH"],
      "BSC": ["BNB", "USDT", "BUSD", "WBNB"],
      "TRX": ["TRX", "USDT"]
    },
    "blacklisted_tokens": ["SCAM_TOKEN_1"],
    "updated_at": "2025-12-01T00:00:00Z",
    "updated_by": "U20251001ADMIN"
  }
}
```

---

## F-ADMIN-041: 更新 Swap 配置

**描述**: 更新 Swap 系统配置。

**gRPC 方法**: `UpdateSwapConfig`

**权限要求**: Admin 角色

**请求 Schema**:
```json
{
  "updates": {
    "enabled": true,
    "default_slippage": 0.005,
    "min_slippage": 0.001,
    "max_slippage": 0.50,
    "preset_slippage_options": [0.001, 0.005, 0.01, 0.03],
    "min_swap_amount_usdt": "10.00",
    "max_swap_amount_usdt": "100000.00",
    "daily_swap_limit_usdt": "500000.00",
    "platform_fee_rate": 0.003,
    "quote_expiration_seconds": 30,
    "high_price_impact_threshold": 0.05
  },
  "reason": "调整滑点范围以适应市场波动"
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "Swap 配置已更新",
  "updated_fields": ["default_slippage", "max_slippage"],
  "updated_at": "2025-12-10T10:00:00Z"
}
```

**业务规则**:
- 部分字段更新，只传需要修改的字段
- 配置变更立即生效
- 所有变更记入审计日志
- 滑点范围校验: min < default < max
- 费率范围: 0 ≤ rate ≤ 0.1 (最高 10%)

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 81040 | 配置参数无效 |
| 81041 | 滑点范围不合法 |
| 81042 | 费率超出允许范围 |

---

## F-ADMIN-042: 管理 DEX 提供商

**描述**: 启用/禁用 DEX 提供商或调整优先级。

**gRPC 方法**: `ManageSwapProvider`

**权限要求**: Admin 角色

**请求 Schema**:
```json
{
  "provider_id": "1inch",
  "action": "update",             // enable/disable/update
  "updates": {
    "enabled": true,
    "priority": 1,
    "api_url": "https://api.1inch.io/v5.0",
    "api_key": "xxx",             // 可选: 部分提供商需要 API Key
    "timeout_ms": 5000,
    "max_retries": 3
  },
  "reason": "提升 1inch 优先级"
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "提供商配置已更新",
  "provider": {
    "id": "1inch",
    "name": "1inch",
    "enabled": true,
    "priority": 1,
    "status": "healthy"
  }
}
```

**业务规则**:
- 至少保留一个启用的提供商
- 优先级数值越小越优先
- 禁用提供商后，该提供商的报价不再返回
- 提供商健康检查每 5 分钟执行一次

---

## F-ADMIN-043: 管理代币白名单

**描述**: 添加或移除可兑换的代币。

**gRPC 方法**: `ManageSwapTokens`

**权限要求**: Admin 角色

**请求 Schema**:
```json
{
  "action": "add",                // add/remove/blacklist
  "chain": "ETH",
  "token": {
    "symbol": "NEW_TOKEN",
    "contract_address": "0x1234...5678",
    "decimals": 18,
    "name": "New Token"
  },
  "reason": "添加新代币支持"
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "代币已添加到白名单",
  "token": {
    "symbol": "NEW_TOKEN",
    "chain": "ETH",
    "status": "active"
  }
}
```

**业务规则**:
- 添加代币前会自动验证合约地址有效性
- 黑名单代币无法参与兑换
- 移除代币不会影响进行中的订单

---

## 配置参数说明

### 滑点配置

| 参数 | 默认值 | 范围 | 说明 |
|------|--------|------|------|
| default_slippage | 0.5% | - | 用户未设置时的默认值 |
| min_slippage | 0.1% | - | 允许设置的最小滑点 |
| max_slippage | 50% | - | 允许设置的最大滑点 |
| preset_options | [0.1%, 0.5%, 1%, 3%] | - | 前端预设选项 |

> **PRD 对齐**: 滑点范围为 0.1% ~ 50%，预设选项为 0.5%/1%/3%

### 金额限制

| 参数 | 默认值 | 说明 |
|------|--------|------|
| min_swap_amount_usdt | 10 USDT | 最小兑换金额 |
| max_swap_amount_usdt | 100,000 USDT | 单笔最大金额 |
| daily_swap_limit_usdt | 500,000 USDT | 单用户每日限额 |

### 费用配置

| 参数 | 默认值 | 说明 |
|------|--------|------|
| platform_fee_rate | 0.3% | 平台收取的手续费 |
| quote_expiration_seconds | 30 | 报价有效期 |

### 风险控制

| 参数 | 默认值 | 说明 |
|------|--------|------|
| high_price_impact_threshold | 5% | 超过此值需用户确认 |

---

## DEX 提供商健康检查

### 检查指标

| 指标 | 健康标准 | 检查方式 |
|------|----------|----------|
| API 可用性 | 响应码 2xx | HTTP 请求 |
| 响应时间 | < 5s | 平均响应时间 |
| 报价成功率 | > 95% | 最近 100 次请求 |

### 状态定义

| 状态 | 说明 |
|------|------|
| healthy | 正常可用 |
| degraded | 部分功能受限 |
| unhealthy | 不可用，已自动禁用 |

---

## 审计日志

配置变更记录以下信息：

| 字段 | 说明 |
|------|------|
| config_type | 配置类型 (slippage/provider/token) |
| action | 操作类型 |
| before_value | 变更前值 |
| after_value | 变更后值 |
| operator_id | 操作人 |
| reason | 变更原因 |
| operated_at | 操作时间 |

---

> 返回 [后台管理服务目录](./README.md)
