# Swap 配置 - Provider 管理

> 返回 [Swap 配置索引](../06-Swap配置.md)

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

> 返回 [Swap 配置索引](../06-Swap配置.md)

