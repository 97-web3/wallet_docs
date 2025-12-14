# Vault 资金管理（索引）

> 返回 [后台管理前端目录](./README.md)

---

## 页面概述

- **页面路径**：`/admin/vault`
- **权限要求**：Admin / Finance
- **模块目标**：面向企业资金管理（Treasury），对多链 Vault 资金进行**余额监控、阈值告警、同步状态观测、资金调整（Maker-Checker）**等操作。

---

## 文档结构（拆分后的目录）

为避免单文件过于拥挤，已将 Vault 资金管理文档拆分为以下子文档（仍然只对应 `/admin/vault` 这一页面入口）：

> 说明：文档按主题拆分并不意味着前端必须新增 Tab。页面可继续保持“总览 + 网络卡片”的结构，其余内容以“页面内区块/折叠面板/抽屉/弹窗”承载即可。

1. [vault/01-页面概览与布局.md](vault/01-页面概览与布局.md)
2. [vault/02-数据模型与字段.md](vault/02-数据模型与字段.md)
3. [vault/03-钱包地址管理.md](vault/03-钱包地址管理.md)
4. [vault/04-余额监控与阈值告警.md](vault/04-余额监控与阈值告警.md)
5. [vault/05-同步与链上可观测性.md](vault/05-同步与链上可观测性.md)
6. [vault/06-余额调整与审批.md](vault/06-余额调整与审批.md)
7. [vault/07-审计、权限与合规.md](vault/07-审计、权限与合规.md)
8. [vault/08-扩展-多Vault与流动性池监控.md](vault/08-扩展-多Vault与流动性池监控.md)
9. [vault/09-性能、SLA与应急预案.md](vault/09-性能、SLA与应急预案.md)
10. [vault/10-告警与通知.md](vault/10-告警与通知.md)

---

## 与后端接口的映射（参考）

以 `wallet_docs/admin-api/srds/10-Vault资金管理.md` 为字段与校验范围的唯一来源。前端文档中的字段与枚举应尽量与 SRD 对齐，避免“页面字段漂移”。

- `GET /api/v1/admin/vault/overview`
  - `summary.*`（总资产/网络健康/待审批/告警数）
  - `networks[]`（按链/网络的 Vault 概览）
- `GET /api/v1/admin/vault/{chain_id}`
  - `balances[]`（含 `available_balance` / `locked_balance` / `threshold_config`）
  - `recent_transactions[]`
  - `sync_info.*`（区块同步、落后高度）
  - `gas_info.*`（手续费币余额与可支持笔数估算）
- `PUT /api/v1/admin/vault/thresholds`
  - `thresholds.low` / `thresholds.critical`
  - `notifications.*`（email/webhook）
- `POST /api/v1/admin/vault/adjust`
- `GET /api/v1/admin/vault/adjustments`
- `POST /api/v1/admin/vault/adjustments/{id}/approve`
- `POST /api/v1/admin/vault/{chain_id}/sync`

---

> 返回 [后台管理前端目录](./README.md)
