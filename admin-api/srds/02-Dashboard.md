# Dashboard

> 返回 [Admin 服务目录](./README.md)

---

## F-ADMIN-005: 获取统计数据

**描述**: 获取 Dashboard 统计卡片数据，包括用户数、资产总额、交易量等核心指标。

**HTTP 方法**: `GET /api/v1/admin/dashboard/statistics`

**请求参数**:
```
无
```

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "total_users": 12580,
    "active_users_today": 1256,
    "total_assets_usd": "15800000.50",
    "pending_withdrawals": 23,
    "pending_withdrawals_amount_usd": "125000.00",
    "today_transactions": 856,
    "today_volume_usd": "520000.00",
    "alerts_count": 3,
    "updated_at": "2025-12-10T10:00:00Z"
  }
}
```

**业务规则**:
- 数据缓存 5 分钟
- 金额统一转换为 USD 展示
- 实时数据通过 WebSocket 推送更新

**权限要求**:
- 角色: Admin, Finance

---

## F-ADMIN-006: 获取资金流趋势

**描述**: 获取指定时间范围内的资金流入流出趋势数据，用于绘制折线图。

**HTTP 方法**: `GET /api/v1/admin/dashboard/flow-trend`

**请求参数**:
```
?period=6m              // 时间范围: 7d, 30d, 3m, 6m, 1y
&granularity=daily      // 粒度: hourly, daily, weekly, monthly
```

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "period": "6m",
    "granularity": "daily",
    "series": [
      {
        "date": "2025-06-01",
        "inflow_usd": "50000.00",
        "outflow_usd": "35000.00",
        "net_flow_usd": "15000.00"
      },
      {
        "date": "2025-06-02",
        "inflow_usd": "62000.00",
        "outflow_usd": "41000.00",
        "net_flow_usd": "21000.00"
      }
    ],
    "summary": {
      "total_inflow_usd": "3200000.00",
      "total_outflow_usd": "2100000.00",
      "total_net_flow_usd": "1100000.00"
    }
  }
}
```

**权限要求**:
- 角色: Admin, Finance

---

## F-ADMIN-007: 获取转账批次趋势

**描述**: 获取最近 7 天的批量转账数据趋势。

**HTTP 方法**: `GET /api/v1/admin/dashboard/transfer-trend`

**请求参数**:
```
?days=7                 // 天数: 7, 14, 30
```

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "series": [
      {
        "date": "2025-12-04",
        "batch_count": 5,
        "total_recipients": 125,
        "total_amount_usd": "85000.00",
        "success_rate": 98.5
      },
      {
        "date": "2025-12-05",
        "batch_count": 3,
        "total_recipients": 89,
        "total_amount_usd": "52000.00",
        "success_rate": 100.0
      }
    ]
  }
}
```

**权限要求**:
- 角色: Admin, Finance

---

## F-ADMIN-008: 获取用户状态分布

**描述**: 获取用户账户状态分布数据，用于绘制饼图。

**HTTP 方法**: `GET /api/v1/admin/dashboard/user-status`

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "distribution": [
      {"status": "active", "count": 11500, "percentage": 91.4},
      {"status": "frozen", "count": 580, "percentage": 4.6},
      {"status": "pending_kyc", "count": 350, "percentage": 2.8},
      {"status": "terminated", "count": 150, "percentage": 1.2}
    ],
    "total": 12580
  }
}
```

**权限要求**:
- 角色: Admin, Finance

---

## F-ADMIN-009: 获取提现状态分布

**描述**: 获取提现申请状态分布数据。

**HTTP 方法**: `GET /api/v1/admin/dashboard/withdrawal-status`

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "distribution": [
      {"status": "pending", "count": 23, "amount_usd": "125000.00"},
      {"status": "approved", "count": 156, "amount_usd": "820000.00"},
      {"status": "processing", "count": 12, "amount_usd": "65000.00"},
      {"status": "completed", "count": 1850, "amount_usd": "9500000.00"},
      {"status": "rejected", "count": 45, "amount_usd": "180000.00"}
    ],
    "summary": {
      "pending_count": 23,
      "pending_amount_usd": "125000.00",
      "today_processed": 45
    }
  }
}
```

**权限要求**:
- 角色: Admin, Finance

---

## F-ADMIN-010: 获取币种余额分布

**描述**: 获取 Top 5 币种的余额分布数据。

**HTTP 方法**: `GET /api/v1/admin/dashboard/currency-balance`

**请求参数**:
```
?top=5                  // 返回 Top N 币种
```

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "currencies": [
      {
        "currency": "USDT",
        "balance": "8500000.000000",
        "balance_usd": "8500000.00",
        "percentage": 53.8
      },
      {
        "currency": "ETH",
        "balance": "1250.500000",
        "balance_usd": "3125000.00",
        "percentage": 19.8
      },
      {
        "currency": "BNB",
        "balance": "5600.000000",
        "balance_usd": "1680000.00",
        "percentage": 10.6
      },
      {
        "currency": "TRX",
        "balance": "12500000.000000",
        "balance_usd": "1250000.00",
        "percentage": 7.9
      },
      {
        "currency": "Others",
        "balance": "-",
        "balance_usd": "1245000.00",
        "percentage": 7.9
      }
    ],
    "total_usd": "15800000.00"
  }
}
```

**权限要求**:
- 角色: Admin, Finance

---

## F-ADMIN-011: 获取最近活动

**描述**: 获取最近的管理操作活动记录。

**HTTP 方法**: `GET /api/v1/admin/dashboard/recent-activities`

**请求参数**:
```
?limit=20               // 返回条数，默认 20
```

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "activities": [
      {
        "id": "act-001",
        "type": "withdrawal_approved",
        "description": "审批通过提现申请 #W20251210001",
        "operator": "admin@company.com",
        "target_user": "10001",
        "amount": "5000.00 USDT",
        "created_at": "2025-12-10T09:55:00Z"
      },
      {
        "id": "act-002",
        "type": "user_frozen",
        "description": "冻结用户账户",
        "operator": "admin@company.com",
        "target_user": "10256",
        "reason": "可疑交易行为",
        "created_at": "2025-12-10T09:50:00Z"
      },
      {
        "id": "act-003",
        "type": "vault_adjustment",
        "description": "Vault 余额调整申请",
        "operator": "finance@company.com",
        "amount": "+10000.00 USDT",
        "status": "pending_approval",
        "created_at": "2025-12-10T09:45:00Z"
      }
    ]
  }
}
```

**活动类型枚举**:

| 类型 | 描述 |
|------|------|
| withdrawal_approved | 提现审批通过 |
| withdrawal_rejected | 提现审批拒绝 |
| user_frozen | 用户冻结 |
| user_unfrozen | 用户解冻 |
| vault_adjustment | Vault 余额调整 |
| payroll_processed | 批量转账执行 |
| config_updated | 配置更新 |
| blacklist_added | 添加黑名单 |

**权限要求**:
- 角色: Admin, Finance

---

## F-ADMIN-012: 获取 Vault 余额

**描述**: 获取各网络 Vault 钱包的余额汇总。

**HTTP 方法**: `GET /api/v1/admin/dashboard/vault-balances`

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "networks": [
      {
        "network": "Ethereum",
        "chain_id": 1,
        "balances": [
          {"currency": "ETH", "balance": "125.500000", "balance_usd": "313750.00"},
          {"currency": "USDT", "balance": "2500000.000000", "balance_usd": "2500000.00"}
        ],
        "total_usd": "2813750.00",
        "status": "healthy",
        "last_sync": "2025-12-10T10:00:00Z"
      },
      {
        "network": "BSC",
        "chain_id": 56,
        "balances": [
          {"currency": "BNB", "balance": "560.000000", "balance_usd": "168000.00"},
          {"currency": "USDT", "balance": "3500000.000000", "balance_usd": "3500000.00"}
        ],
        "total_usd": "3668000.00",
        "status": "healthy",
        "last_sync": "2025-12-10T10:00:00Z"
      },
      {
        "network": "Tron",
        "chain_id": -1,
        "balances": [
          {"currency": "TRX", "balance": "12500000.000000", "balance_usd": "1250000.00"},
          {"currency": "USDT", "balance": "2500000.000000", "balance_usd": "2500000.00"}
        ],
        "total_usd": "3750000.00",
        "status": "healthy",
        "last_sync": "2025-12-10T10:00:00Z"
      }
    ],
    "grand_total_usd": "10231750.00"
  }
}
```

**权限要求**:
- 角色: Admin, Finance

---

## F-ADMIN-013: 获取告警阈值配置

**描述**: 获取当前的告警阈值配置。

**HTTP 方法**: `GET /api/v1/admin/dashboard/alerts`

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "thresholds": [
      {
        "id": "alert-001",
        "type": "low_balance",
        "network": "Ethereum",
        "currency": "ETH",
        "threshold": "10.000000",
        "current_value": "125.500000",
        "status": "normal",
        "enabled": true
      },
      {
        "id": "alert-002",
        "type": "pending_withdrawal_count",
        "threshold": "50",
        "current_value": "23",
        "status": "normal",
        "enabled": true
      },
      {
        "id": "alert-003",
        "type": "pending_withdrawal_amount",
        "threshold": "500000",
        "current_value": "125000",
        "status": "normal",
        "enabled": true
      }
    ],
    "active_alerts": [
      {
        "id": "active-001",
        "type": "high_withdrawal_volume",
        "message": "今日提现量超过阈值",
        "severity": "warning",
        "triggered_at": "2025-12-10T08:00:00Z"
      }
    ]
  }
}
```

**权限要求**:
- 角色: Admin

---

## F-ADMIN-014: 设置告警阈值

**描述**: 创建或更新告警阈值配置。

**HTTP 方法**: `POST /api/v1/admin/dashboard/alerts`

**请求 Schema**:
```json
{
  "type": "low_balance",           // 告警类型
  "network": "Ethereum",           // 网络 (部分类型需要)
  "currency": "ETH",               // 币种 (部分类型需要)
  "threshold": "20.000000",        // 阈值
  "enabled": true,                 // 是否启用
  "notify_email": true,            // 邮件通知
  "notify_webhook": true           // Webhook 通知
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "告警配置已更新",
  "data": {
    "id": "alert-001",
    "type": "low_balance",
    "threshold": "20.000000",
    "enabled": true
  }
}
```

**告警类型枚举**:

| 类型 | 描述 | 需要参数 |
|------|------|----------|
| low_balance | 余额过低 | network, currency, threshold |
| pending_withdrawal_count | 待审批提现数量 | threshold |
| pending_withdrawal_amount | 待审批提现金额 | threshold (USD) |
| high_withdrawal_volume | 单日提现量 | threshold (USD) |
| sync_delay | 同步延迟 | network, threshold (秒) |
| rpc_error_rate | RPC 错误率 | network, threshold (%) |

**权限要求**:
- 角色: Admin

---

## F-ADMIN-015: 获取统计数据 (基础版)

**描述**: 获取 Dashboard 核心统计数据 (API Gateway 定义的原始接口)。

**HTTP 方法**: `GET /api/v1/admin/dashboard/statistics`

**gRPC 方法**: `GetDashboardStatistics`

**响应 Schema**:
```json
{
  "success": true,
  "data": {
    "user_stats": {
      "total": 12580,
      "active_today": 1256,
      "new_today": 45,
      "frozen": 580
    },
    "transaction_stats": {
      "today_count": 856,
      "today_volume_usd": "520000.00",
      "pending_withdrawals": 23
    },
    "vault_stats": {
      "total_balance_usd": "15800000.00",
      "networks_healthy": 3,
      "networks_total": 3
    }
  }
}
```

**权限要求**:
- 角色: Admin, Finance
- 限流: 10/s

---

> 返回 [Admin 服务目录](./README.md)
