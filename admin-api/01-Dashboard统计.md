# Dashboard 统计

> 返回 [后台管理服务目录](./README.md)

---

## F-ADMIN-001: 获取 Dashboard 统计数据

**描述**: 获取管理员 Dashboard 的汇总统计数据，包括充值/提现统计、托管余额、Vault 余额等。

**gRPC 方法**: `GetDashboardStatistics`

**权限要求**: Admin 或 Finance 角色

**请求 Schema**:
```json
{
  "time_window": "24h",       // 时间窗口: 24h/7d/30d/all
  "start_time": "",           // 可选: 自定义开始时间 (ISO 8601)
  "end_time": ""              // 可选: 自定义结束时间 (ISO 8601)
}
```

**响应 Schema**:
```json
{
  "success": true,
  "statistics": {
    "total_deposits": {
      "amount_usdt": "1000000.00",
      "count": 500,
      "change_percent": "12.5"     // 相比上一周期变化
    },
    "total_withdrawals": {
      "amount_usdt": "800000.00",
      "count": 300,
      "change_percent": "-5.2"
    },
    "web2_custodial_balance": {
      "total_usdt": "500000.00",
      "by_asset": [
        { "asset": "USDT", "amount": "300000.00", "valuation_usdt": "300000.00" },
        { "asset": "TRX", "amount": "1000000.00", "valuation_usdt": "100000.00" },
        { "asset": "ETH", "amount": "50.00", "valuation_usdt": "100000.00" }
      ]
    },
    "active_users": {
      "count": 1200,
      "change_percent": "8.3"
    },
    "pending_withdrawals": {
      "count": 15,
      "amount_usdt": "50000.00"
    },
    "pending_payroll": {
      "batch_count": 2,
      "total_amount_usdt": "100000.00"
    }
  },
  "generated_at": "2025-12-10T10:00:00Z"
}
```

**业务规则**:
- 数据缓存 1 分钟，避免频繁查询数据库
- 金额单位统一使用 USDT 等值
- 变化百分比基于相同时间窗口的前一周期计算
- P95 响应时间目标: ≤ 200ms

---

## F-ADMIN-002: 获取 Vault 余额

**描述**: 获取企业 Vault (热钱包/冷钱包) 的各链余额详情。

**gRPC 方法**: `GetVaultBalances`

**权限要求**: Admin 或 Finance 角色

**请求 Schema**:
```json
{
  "wallet_type": "all",       // hot/cold/all
  "chain": ""                 // 可选: 筛选链 (TRX/BSC/ETH)
}
```

**响应 Schema**:
```json
{
  "success": true,
  "vault_balances": {
    "hot_wallet": {
      "total_usdt": "200000.00",
      "balances": [
        {
          "chain": "TRX",
          "address": "T9yD14Nj9j7xAB4dbGeiX9h8unkKHxuWwb",
          "native_balance": "100000.00",
          "native_symbol": "TRX",
          "native_usdt": "10000.00",
          "tokens": [
            { "symbol": "USDT", "balance": "50000.00", "usdt_value": "50000.00" }
          ]
        },
        {
          "chain": "ETH",
          "address": "0x1234...5678",
          "native_balance": "50.00",
          "native_symbol": "ETH",
          "native_usdt": "112500.00",
          "tokens": [
            { "symbol": "USDT", "balance": "30000.00", "usdt_value": "30000.00" }
          ]
        },
        {
          "chain": "BSC",
          "address": "0xabcd...efgh",
          "native_balance": "100.00",
          "native_symbol": "BNB",
          "native_usdt": "31000.00",
          "tokens": [
            { "symbol": "USDT", "balance": "20000.00", "usdt_value": "20000.00" }
          ]
        }
      ]
    },
    "cold_wallet": {
      "total_usdt": "1000000.00",
      "balances": [...]
    }
  },
  "total_usdt": "1200000.00",
  "last_sync_at": "2025-12-10T10:00:00Z"
}
```

**业务规则**:
- 余额数据实时从 ChainSync 服务获取
- 热钱包地址用于日常出金
- 冷钱包地址用于大额存储
- 余额低于阈值时触发告警

---

## F-ADMIN-003: 获取告警阈值配置

**描述**: 获取当前的余额告警阈值配置。

**gRPC 方法**: `GetAlertThresholds`

**权限要求**: Admin 或 Finance 角色

**请求 Schema**:
```json
{}
```

**响应 Schema**:
```json
{
  "success": true,
  "thresholds": [
    {
      "id": 1,
      "chain": "TRX",
      "asset": "USDT",
      "wallet_type": "hot",
      "min_balance": "10000.00",
      "warning_balance": "20000.00",
      "enabled": true,
      "notify_roles": ["admin", "finance"],
      "notify_channels": ["email", "push"],
      "created_at": "2025-01-01T00:00:00Z",
      "updated_at": "2025-12-01T00:00:00Z"
    },
    {
      "id": 2,
      "chain": "ETH",
      "asset": "ETH",
      "wallet_type": "hot",
      "min_balance": "5.00",
      "warning_balance": "10.00",
      "enabled": true,
      "notify_roles": ["admin"],
      "notify_channels": ["email", "push", "sms"],
      "created_at": "2025-01-01T00:00:00Z",
      "updated_at": "2025-12-01T00:00:00Z"
    }
  ]
}
```

---

## F-ADMIN-004: 设置告警阈值配置

**描述**: 创建或更新余额告警阈值配置。

**gRPC 方法**: `SetAlertThreshold`

**权限要求**: Admin 角色

**请求 Schema**:
```json
{
  "id": 0,                    // 0 表示新建，>0 表示更新
  "chain": "TRX",
  "asset": "USDT",
  "wallet_type": "hot",       // hot/cold
  "min_balance": "10000.00",  // 触发告警的最低余额
  "warning_balance": "20000.00", // 触发警告的余额
  "enabled": true,
  "notify_roles": ["admin", "finance"],
  "notify_channels": ["email", "push"]
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "阈值配置已保存",
  "threshold_id": 1
}
```

**业务规则**:
- `min_balance`: 低于此值触发紧急告警
- `warning_balance`: 低于此值触发预警（不紧急）
- 必须 `min_balance < warning_balance`
- 配置变更记入审计日志
- 通知渠道支持: email, push, sms, webhook

---

## 告警通知规则

### 告警级别

| 级别 | 条件 | 通知频率 |
|------|------|----------|
| WARNING | 余额 < warning_balance | 每 4 小时一次 |
| CRITICAL | 余额 < min_balance | 每 30 分钟一次 |

### 告警消息格式

```json
{
  "alert_id": "ALERT-20251210-001",
  "level": "CRITICAL",
  "title": "热钱包余额不足",
  "message": "TRX 链 USDT 余额 (8,000) 低于最低阈值 (10,000)",
  "chain": "TRX",
  "asset": "USDT",
  "current_balance": "8000.00",
  "threshold": "10000.00",
  "wallet_address": "T9yD14...",
  "timestamp": "2025-12-10T10:00:00Z",
  "action_url": "https://admin.example.com/vault/TRX"
}
```

### 静默窗口

- 支持配置静默时间窗口（如凌晨 2-6 点不发送非紧急告警）
- CRITICAL 级别告警不受静默窗口影响

---

## 性能指标

| 指标 | 目标 |
|------|------|
| GetDashboardStatistics P95 | ≤ 200ms |
| GetVaultBalances P95 | ≤ 500ms |
| 数据缓存 TTL | 60 秒 |
| 告警检查间隔 | 5 分钟 |

---

> 返回 [后台管理服务目录](./README.md)
