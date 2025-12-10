# Gas 管理

> 返回 [ChainRPC 服务目录](./README.md)

---

## F-CRPC-008: 估算 Gas

**描述**: 估算交易所需的 Gas 费用，并检查原生币余额是否足够。

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
  "estimated_fee_usd": "0.5",
  "native_balance": "50000000000000",   // 原生币余额
  "native_symbol": "TRX",               // 原生币符号
  "insufficient_gas": false,            // 原生币是否不足以支付 Gas
  "required_native_balance": "21000000000000",  // 所需原生币数量
  "shortfall": "0"                      // 缺少的原生币数量 (insufficient_gas=true 时)
}
```

**业务规则**:
- 当 `insufficient_gas=true` 时，提示用户先充值原生币
- 返回的 `shortfall` 表示需要补充的原生币数量
- 各链原生币: TRX (Tron), BNB (BSC), ETH (Ethereum)

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
