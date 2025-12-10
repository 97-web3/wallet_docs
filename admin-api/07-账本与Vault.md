# 账本与 Vault

> 返回 [后台管理服务目录](./README.md)

---

## 模块概述

账本与 Vault 模块为管理员和财务人员提供账本查询、导出及 Vault 余额调整功能。

---

## F-ADMIN-050: 获取账本记录

**描述**: 查询系统账本流水记录。

**gRPC 方法**: `GetLedgerRecords`

**权限要求**: Finance 或 Admin 角色

**请求 Schema**:
```json
{
  "page": 1,
  "page_size": 50,
  "type": "",                      // 可选: deposit/withdraw/transfer/swap/payroll/adjustment
  "asset": "",                     // 可选: USDT/TRX/BNB/ETH
  "chain": "",                     // 可选: TRX/BSC/ETH
  "user_uid": "",                  // 可选: 按用户筛选
  "start_time": "",
  "end_time": "",
  "min_amount": "",
  "max_amount": "",
  "search": "",                    // 搜索交易ID/地址
  "sort_by": "created_at",
  "sort_order": "desc"
}
```

**响应 Schema**:
```json
{
  "success": true,
  "records": [
    {
      "ledger_id": "L20251210000001",
      "type": "deposit",
      "type_text": "充值",
      "user_uid": "U20251210ABCD",
      "user_nickname": "用户昵称",
      "asset": "USDT",
      "chain": "TRX",
      "amount": "1000.00",
      "amount_usdt": "1000.00",
      "fee": "0.00",
      "balance_before": "5000.00",
      "balance_after": "6000.00",
      "related_id": "D20251210001",        // 关联的业务ID
      "related_type": "deposit",
      "tx_hash": "abc123...",
      "from_address": "T9yD14Nj...",
      "to_address": "TWd4WrZ9...",
      "status": "completed",
      "status_text": "已完成",
      "memo": "",
      "created_at": "2025-12-10T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 50,
    "total": 10000,
    "total_pages": 200
  },
  "summary": {
    "total_in": "500000.00",
    "total_out": "450000.00",
    "net_flow": "50000.00"
  }
}
```

**业务规则**:
- 支持多维度筛选和组合查询
- 敏感字段（用户手机/邮箱）不在此接口返回
- 账本记录不可修改，仅供查询
- 查询结果默认按时间倒序

---

## F-ADMIN-051: 导出账本

**描述**: 导出账本数据为 Excel 或 CSV 文件。

**gRPC 方法**: `ExportLedger`

**权限要求**: Finance 或 Admin 角色

**请求 Schema**:
```json
{
  "type": "",                      // 可选: 同查询筛选条件
  "asset": "",
  "chain": "",
  "user_uid": "",
  "start_time": "2025-12-01T00:00:00Z",
  "end_time": "2025-12-10T23:59:59Z",
  "format": "xlsx",                // xlsx/csv
  "include_summary": true          // 是否包含汇总行
}
```

**响应 Schema**:
```json
{
  "success": true,
  "download_url": "https://...",
  "file_name": "ledger_20251201_20251210.xlsx",
  "record_count": 5000,
  "expires_at": "2025-12-10T11:00:00Z"
}
```

**导出限制**:
| 限制项 | 值 |
|--------|-----|
| 单次最大导出记录 | 100,000 条 |
| 导出文件有效期 | 1 小时 |
| 并发导出限制 | 每用户 1 个任务 |

**导出列**:
| 列名 | 说明 |
|------|------|
| 序号 | 行号 |
| 账本ID | 唯一标识 |
| 类型 | 交易类型 |
| 用户UID | 用户标识 |
| 币种 | 资产类型 |
| 链 | 区块链网络 |
| 金额 | 交易金额 |
| USDT等值 | 换算值 |
| 手续费 | 费用 |
| 变动前余额 | |
| 变动后余额 | |
| 关联ID | 业务单号 |
| 交易哈希 | 链上哈希 |
| 状态 | 交易状态 |
| 创建时间 | |

---

## F-ADMIN-052: 调整 Vault 余额

**描述**: 管理员手动调整 Vault 余额（用于补充资金或纠正差异）。

**gRPC 方法**: `AdminAdjustVaultBalance`

**权限要求**: Admin 角色（大额调整需双重审批）

**请求 Schema**:
```json
{
  "asset": "USDT",
  "chain": "TRX",
  "adjustment_type": "increase",   // increase/decrease
  "amount": "50000.00",
  "reason": "补充 Hot Wallet 资金",
  "reference_tx": "abc123...",     // 可选: 关联的链上交易
  "attachments": []                // 可选: 附件证明
}
```

**请求头**:
```
Idempotency-Key: <UUID>
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "Vault 余额已调整",
  "adjustment_id": "ADJ20251210001",
  "asset": "USDT",
  "chain": "TRX",
  "adjustment_type": "increase",
  "amount": "50000.00",
  "balance_before": "100000.00",
  "balance_after": "150000.00",
  "adjusted_at": "2025-12-10T10:00:00Z",
  "requires_approval": false        // 是否需要二次审批
}
```

**业务规则**:
- 调整金额 > 100,000 USDT 需要双重审批（见 [角色与权限](./06-角色与权限.md)）
- 必须提供调整原因
- 建议关联链上交易作为证明
- 所有调整记入审计日志和账本
- 使用 Idempotency-Key 防止重复调整

**双重审批流程**:
```
发起调整 (> 100,000 USDT)
    │
    ▼
┌─────────────────┐
│  待第二审批     │
│  status: pending │
└────────┬────────┘
         │
    ┌────┴────┐
    │ 审批通过 │ 审批拒绝
    ▼         ▼
  执行调整   取消调整
```

**错误码**:
| 错误码 | 说明 |
|--------|------|
| 81050 | 调整金额无效 |
| 81051 | 余额不足（减少时） |
| 81052 | 需要双重审批 |
| 81053 | 调整原因不能为空 |
| 81054 | 重复操作 |

---

## F-ADMIN-053: 获取 Vault 调整历史

**描述**: 查询 Vault 余额调整历史记录。

**gRPC 方法**: `GetVaultAdjustmentHistory`

**权限要求**: Finance 或 Admin 角色

**请求 Schema**:
```json
{
  "page": 1,
  "page_size": 20,
  "asset": "",
  "chain": "",
  "adjustment_type": "",           // increase/decrease
  "status": "",                    // pending/approved/rejected/completed
  "start_time": "",
  "end_time": "",
  "operator_id": ""                // 按操作人筛选
}
```

**响应 Schema**:
```json
{
  "success": true,
  "adjustments": [
    {
      "adjustment_id": "ADJ20251210001",
      "asset": "USDT",
      "chain": "TRX",
      "adjustment_type": "increase",
      "adjustment_type_text": "增加",
      "amount": "50000.00",
      "balance_before": "100000.00",
      "balance_after": "150000.00",
      "reason": "补充 Hot Wallet 资金",
      "reference_tx": "abc123...",
      "status": "completed",
      "status_text": "已完成",
      "operator_id": "U20251001ADMIN",
      "operator_name": "管理员张三",
      "approver_id": "U20251001SUPER",
      "approver_name": "超管李四",
      "created_at": "2025-12-10T10:00:00Z",
      "approved_at": "2025-12-10T10:05:00Z",
      "completed_at": "2025-12-10T10:05:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 50,
    "total_pages": 3
  }
}
```

---

## F-ADMIN-054: 审批 Vault 调整

**描述**: 审批或拒绝待审批的 Vault 调整请求。

**gRPC 方法**: `ApproveVaultAdjustment`

**权限要求**: Super Admin 角色（或根据金额要求的审批人）

**请求 Schema**:
```json
{
  "adjustment_id": "ADJ20251210001",
  "action": "approve",             // approve/reject
  "comment": "确认资金来源合规"
}
```

**响应 Schema**:
```json
{
  "success": true,
  "message": "调整已审批",
  "adjustment_id": "ADJ20251210001",
  "new_status": "completed",
  "balance_after": "150000.00",
  "approved_at": "2025-12-10T10:05:00Z"
}
```

---

## 账本类型定义

| 类型 | 代码 | 说明 | 金额方向 |
|------|------|------|----------|
| 充值 | deposit | 用户充值到账 | + |
| 提现 | withdraw | 用户提现 | - |
| 内部转账-转出 | transfer_out | 内部转账发起方 | - |
| 内部转账-转入 | transfer_in | 内部转账接收方 | + |
| 闪兑-卖出 | swap_sell | Swap 卖出币种 | - |
| 闪兑-买入 | swap_buy | Swap 买入币种 | + |
| 薪资发放 | payroll | 薪资入账 | + |
| Vault调整 | adjustment | 管理员手动调整 | +/- |
| 手续费 | fee | 手续费扣除 | - |
| 冻结 | freeze | 资产冻结 | - |
| 解冻 | unfreeze | 资产解冻 | + |

---

## 审计日志

所有账本与 Vault 操作记录以下信息：

| 字段 | 说明 |
|------|------|
| action | 操作类型 (query/export/adjust/approve) |
| operator_id | 操作人 UID |
| operator_role | 操作人角色 |
| target_asset | 目标资产 |
| target_chain | 目标链 |
| amount | 涉及金额 |
| query_params | 查询参数（导出时） |
| client_ip | 客户端 IP |
| operated_at | 操作时间 |

---

## 性能指标

| 指标 | 目标 |
|------|------|
| 账本查询 P95 | ≤ 500ms |
| 导出任务启动 | ≤ 2s |
| 余额调整 P95 | ≤ 300ms |
| 并发查询支持 | ≥ 100 QPS |

---

> 返回 [后台管理服务目录](./README.md)
