# Swap 配置 - Token 与交易对

> 返回 [Swap 配置索引](../06-Swap配置.md)

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

> 返回 [Swap 配置索引](../06-Swap配置.md)

