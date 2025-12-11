# Swap 配置

> 返回 [Admin 服务目录](./README.md)

---

## F-ADMIN-038: 获取 Swap 全局配置

**描述**: 获取 Swap 功能的全局配置项。

**HTTP 方法**: `GET /api/v1/admin/swap/config`

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "global_settings": {
      "swap_enabled": true,
      "min_swap_amount_usd": "10.00",
      "max_swap_amount_usd": "100000.00",
      "daily_limit_usd": "500000.00",
      "fee_rate": "0.25%",
      "fee_min_usd": "1.00"
    },
    "slippage_settings": {
      "default_slippage": "0.5%",
      "max_slippage": "5.0%",
      "slippage_warning_threshold": "2.0%"
    },
    "price_settings": {
      "price_validity_seconds": 30,
      "price_source": "aggregated",
      "fallback_sources": ["coingecko", "coinmarketcap"]
    },
    "risk_settings": {
      "high_value_threshold_usd": "50000.00",
      "require_2fa_above_usd": "10000.00"
    }
  }
}
```

**权限要求**:
- 角色: Admin

---

## F-ADMIN-039: 更新 Swap 全局配置

**描述**: 更新 Swap 功能的全局配置项。

**HTTP 方法**: `PUT /api/v1/admin/swap/config`

**请求 Schema**:
```json
{
  "global_settings": {
    "swap_enabled": true,
    "min_swap_amount_usd": "10.00",
    "max_swap_amount_usd": "100000.00",
    "daily_limit_usd": "500000.00",
    "fee_rate": "0.25%",
    "fee_min_usd": "1.00"
  },
  "slippage_settings": {
    "default_slippage": "0.5%",
    "max_slippage": "5.0%",
    "slippage_warning_threshold": "2.0%"
  },
  "reason": "调整每日限额"
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "配置已更新",
  "data": {
    "updated_at": "2025-12-10T10:00:00Z",
    "updated_by": "admin@company.com",
    "changes": [
      {
        "field": "global_settings.daily_limit_usd",
        "old_value": "300000.00",
        "new_value": "500000.00"
      }
    ]
  }
}
```

**业务规则**:
- 配置变更立即生效
- 记录配置变更历史
- 敏感配置变更需要审批

**权限要求**:
- 角色: Admin

---

## F-ADMIN-040: 获取 DEX 提供商列表

**描述**: 获取所有配置的 DEX 提供商及其状态。

**HTTP 方法**: `GET /api/v1/admin/swap/providers`

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "providers": [
      {
        "id": "uniswap-v3",
        "name": "Uniswap V3",
        "network": "Ethereum",
        "chain_id": 1,
        "router_address": "0x68b3465833fb72A70ecDF485E0e4C7bD8665Fc45",
        "enabled": true,
        "priority": 1,
        "supported_pairs": 156,
        "fee_tier": "0.3%",
        "status": "healthy",
        "avg_slippage_24h": "0.45%",
        "volume_24h_usd": "125000.00",
        "last_trade_at": "2025-12-10T09:55:00Z",
        "health_check": {
          "latency_ms": 150,
          "success_rate_24h": "99.8%",
          "last_error": null
        }
      },
      {
        "id": "pancakeswap-v3",
        "name": "PancakeSwap V3",
        "network": "BSC",
        "chain_id": 56,
        "router_address": "0x13f4EA83D0bd40E75C8222255bc855a974568Dd4",
        "enabled": true,
        "priority": 1,
        "supported_pairs": 89,
        "fee_tier": "0.25%",
        "status": "healthy",
        "avg_slippage_24h": "0.38%",
        "volume_24h_usd": "85000.00",
        "last_trade_at": "2025-12-10T09:50:00Z"
      },
      {
        "id": "sunswap-v2",
        "name": "SunSwap V2",
        "network": "Tron",
        "chain_id": -1,
        "router_address": "TKzxdSv2FZKQrEqkKVgp5DcwEXBEKMg2Ax",
        "enabled": true,
        "priority": 1,
        "supported_pairs": 45,
        "fee_tier": "0.3%",
        "status": "healthy",
        "avg_slippage_24h": "0.52%",
        "volume_24h_usd": "65000.00"
      }
    ]
  }
}
```

**权限要求**:
- 角色: Admin

---

## F-ADMIN-041: 管理 DEX 提供商

**描述**: 启用/禁用 DEX 提供商或更新其配置。

**HTTP 方法**: `PUT /api/v1/admin/swap/providers/{id}`

**路径参数**:
- `id`: 提供商 ID

**请求 Schema**:
```json
{
  "enabled": true,
  "priority": 1,
  "fee_tier": "0.3%",
  "max_slippage": "3.0%",
  "daily_limit_usd": "200000.00",
  "reason": "调整优先级"
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "提供商配置已更新",
  "data": {
    "id": "uniswap-v3",
    "enabled": true,
    "priority": 1,
    "updated_at": "2025-12-10T10:00:00Z",
    "updated_by": "admin@company.com"
  }
}
```

**业务规则**:
- 至少保留一个启用的提供商
- 禁用提供商前检查是否有进行中的交易
- 记录配置变更日志

**权限要求**:
- 角色: Admin

**错误码**:
| 错误码 | HTTP 状态 | 描述 |
|--------|-----------|------|
| PROVIDER_NOT_FOUND | 404 | 提供商不存在 |
| LAST_PROVIDER | 400 | 不能禁用最后一个提供商 |
| PENDING_TRANSACTIONS | 409 | 存在进行中的交易 |

---

## F-ADMIN-042: 获取支持的交易对

**描述**: 获取系统支持的 Swap 交易对列表。

**HTTP 方法**: `GET /api/v1/admin/swap/pairs`

**请求参数**:
```
?network=                    // 网络筛选
&base_currency=              // 基础币种
&quote_currency=             // 计价币种
&enabled=true                // 启用状态
&page=1
&page_size=50
```

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "pairs": [
      {
        "id": "eth-usdt-ethereum",
        "base_currency": "ETH",
        "quote_currency": "USDT",
        "network": "Ethereum",
        "enabled": true,
        "providers": ["uniswap-v3"],
        "min_amount": "0.001",
        "max_amount": "100.000000",
        "current_rate": "2500.00",
        "rate_24h_change": "+2.5%",
        "volume_24h_usd": "85000.00",
        "liquidity_usd": "5000000.00"
      },
      {
        "id": "bnb-usdt-bsc",
        "base_currency": "BNB",
        "quote_currency": "USDT",
        "network": "BSC",
        "enabled": true,
        "providers": ["pancakeswap-v3"],
        "min_amount": "0.01",
        "max_amount": "500.000000",
        "current_rate": "300.00",
        "rate_24h_change": "-1.2%",
        "volume_24h_usd": "52000.00"
      }
    ]
  },
  "pagination": {
    "page": 1,
    "page_size": 50,
    "total": 45,
    "total_pages": 1
  }
}
```

**权限要求**:
- 角色: Admin

---

## F-ADMIN-043: 管理交易对代币

**描述**: 添加、更新或移除 Swap 支持的代币。

**HTTP 方法**: `POST /api/v1/admin/swap/tokens`

**请求 Schema**:
```json
{
  "action": "add",              // add, update, remove
  "token": {
    "symbol": "WETH",
    "name": "Wrapped Ether",
    "network": "Ethereum",
    "contract_address": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
    "decimals": 18,
    "enabled": true,
    "swap_enabled": true,
    "min_amount": "0.001",
    "max_amount": "100.000000",
    "icon_url": "https://cdn.example.com/tokens/weth.png"
  },
  "reason": "添加 WETH 支持"
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "代币已添加",
  "data": {
    "id": "weth-ethereum",
    "symbol": "WETH",
    "network": "Ethereum",
    "created_at": "2025-12-10T10:00:00Z",
    "created_by": "admin@company.com"
  }
}
```

**业务规则**:
- 添加代币前验证合约地址
- 移除代币前检查是否有持仓
- 记录代币配置变更

**权限要求**:
- 角色: Admin

**错误码**:
| 错误码 | HTTP 状态 | 描述 |
|--------|-----------|------|
| TOKEN_EXISTS | 409 | 代币已存在 |
| INVALID_CONTRACT | 400 | 合约地址无效 |
| TOKEN_HAS_BALANCE | 400 | 代币有用户持仓，无法移除 |

---

## F-ADMIN-044: 获取汇率配置

**描述**: 获取汇率来源和更新配置。

**HTTP 方法**: `GET /api/v1/admin/swap/rates/config`

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "primary_source": "aggregated",
    "sources": [
      {
        "id": "coingecko",
        "name": "CoinGecko",
        "enabled": true,
        "priority": 1,
        "weight": 0.4,
        "api_status": "healthy",
        "rate_limit": "50/min",
        "last_update": "2025-12-10T09:59:00Z"
      },
      {
        "id": "coinmarketcap",
        "name": "CoinMarketCap",
        "enabled": true,
        "priority": 2,
        "weight": 0.4,
        "api_status": "healthy",
        "rate_limit": "30/min",
        "last_update": "2025-12-10T09:59:00Z"
      },
      {
        "id": "binance",
        "name": "Binance",
        "enabled": true,
        "priority": 3,
        "weight": 0.2,
        "api_status": "healthy",
        "rate_limit": "1200/min",
        "last_update": "2025-12-10T09:59:30Z"
      }
    ],
    "update_interval_seconds": 30,
    "deviation_threshold": "1.0%",
    "fallback_behavior": "use_cache"
  }
}
```

**权限要求**:
- 角色: Admin

---

## F-ADMIN-045: 获取 Swap 历史统计

**描述**: 获取 Swap 交易的历史统计数据。

**HTTP 方法**: `GET /api/v1/admin/swap/statistics`

**请求参数**:
```
?period=30d                  // 时间范围: 7d, 30d, 90d, 1y
&group_by=day                // 分组: hour, day, week, month
&network=                    // 网络筛选
```

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "period": "30d",
    "summary": {
      "total_swaps": 12580,
      "total_volume_usd": "8500000.00",
      "total_fee_usd": "21250.00",
      "avg_swap_amount_usd": "675.60",
      "success_rate": "99.2%",
      "unique_users": 856
    },
    "by_network": [
      {
        "network": "Ethereum",
        "swap_count": 5200,
        "volume_usd": "4500000.00",
        "fee_usd": "11250.00"
      },
      {
        "network": "BSC",
        "swap_count": 4800,
        "volume_usd": "2800000.00",
        "fee_usd": "7000.00"
      },
      {
        "network": "Tron",
        "swap_count": 2580,
        "volume_usd": "1200000.00",
        "fee_usd": "3000.00"
      }
    ],
    "by_pair": [
      {
        "pair": "ETH/USDT",
        "swap_count": 3500,
        "volume_usd": "3200000.00"
      },
      {
        "pair": "BNB/USDT",
        "swap_count": 2800,
        "volume_usd": "1800000.00"
      }
    ],
    "trend": [
      {
        "date": "2025-12-01",
        "swap_count": 450,
        "volume_usd": "285000.00",
        "avg_slippage": "0.48%"
      }
    ]
  }
}
```

**权限要求**:
- 角色: Admin, Finance

---

## DEX 提供商状态枚举

| 状态 | 代码 | 描述 |
|------|------|------|
| 健康 | healthy | 正常运行 |
| 降级 | degraded | 部分功能受限 |
| 不可用 | unavailable | 服务不可用 |
| 维护中 | maintenance | 计划维护 |

## 价格来源枚举

| 来源 | 代码 | 描述 |
|------|------|------|
| 聚合 | aggregated | 多来源聚合 |
| CoinGecko | coingecko | CoinGecko API |
| CoinMarketCap | coinmarketcap | CoinMarketCap API |
| Binance | binance | Binance 行情 |
| DEX | dex | 链上 DEX 价格 |

---

> 返回 [Admin 服务目录](./README.md)
