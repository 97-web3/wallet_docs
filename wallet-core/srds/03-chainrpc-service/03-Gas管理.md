# Gas 管理

> 返回 [ChainRPC 服务目录](./README.md)

---

## F-CRPC-008: 估算 Gas

**描述**: 估算交易所需的 Gas 费用。

**gRPC 方法**: `EstimateGas`

**请求 Schema**:
```json
{
  "chain": 1,
  "from_address": "TKvjY...",
  "to_address": "TAbcD...",
  "value": "1000000",
  "data": "",                           // 可选：交易数据
  "token_contract": ""                  // 可选：代币合约
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "",
  "gas_limit": 21000,
  "gas_price": "1000000000",
  "estimated_fee": "21000000000000",
  "estimated_fee_usd": "0.5"
}
```

---

## F-CRPC-009: 获取 Gas 价格

**描述**: 获取当前推荐的 Gas 价格。

**gRPC 方法**: `GetGasPrice`

**请求 Schema**:
```json
{
  "chain": 1,
  "speed": 2                            // GasSpeed 枚举
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "",
  "prices": [
    {
      "speed": 1,                       // SLOW
      "gas_price": "500000000",
      "gas_price_gwei": "0.5",
      "estimated_time_seconds": 300
    },
    {
      "speed": 2,                       // STANDARD
      "gas_price": "1000000000",
      "gas_price_gwei": "1",
      "estimated_time_seconds": 60
    },
    {
      "speed": 3,                       // FAST
      "gas_price": "2000000000",
      "gas_price_gwei": "2",
      "estimated_time_seconds": 15
    },
    {
      "speed": 4,                       // INSTANT
      "gas_price": "5000000000",
      "gas_price_gwei": "5",
      "estimated_time_seconds": 5
    }
  ],
  "current_block": "12345678",
  "timestamp": 1702300800
}
```

---

## Gas 速度枚举

| 值 | 名称 | 描述 |
|----|------|------|
| 1 | GAS_SPEED_SLOW | 慢速 (省钱) |
| 2 | GAS_SPEED_STANDARD | 标准 |
| 3 | GAS_SPEED_FAST | 快速 |
| 4 | GAS_SPEED_INSTANT | 极速 |
