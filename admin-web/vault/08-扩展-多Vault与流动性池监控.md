# Vault / 扩展：多 Vault 与流动性池监控（规划）

> 返回 [Vault 资金管理索引](../10-Vault资金管理.md)

---

## 重要说明

本文件用于承载“企业 Treasury & Liquidity Monitoring”所需但**当前页面/接口未必已实现**的能力说明，避免把规划能力挤进核心文档，同时为后续产品化提供统一口径。

---

## 1. 多 Vault（Vault 维度）扩展

### 1.1 为什么需要

当需要监控 50+ Vault（热钱包/冷钱包/运营金/储备金/做市池），仅按网络展示会导致：

- 无法按资金用途与负责人分组
- 无法做跨 Vault 汇总报表（AUM、覆盖率、资金缺口）
- 权限无法按 Vault 边界隔离（合规风险）

### 1.2 推荐的 Vault 字段（文档级定义）

- `vault_id`：唯一标识
- `name`：Vault 名称
- `type`：hot/cold/ops/reserve/pool
- `owner_team`：责任团队
- `policy_tags[]`：策略标签（提款上限、冷却期、审批规则）
- `networks[]`：该 Vault 覆盖的链与地址/合约

### 1.3 UI 建议

- `/admin/vault` 保持入口不变，新增：
  - 顶部筛选：Vault Group / Owner / Tag
  - 列表视图：按 Vault 行展示（可展开网络明细）
  - 跨 Vault 汇总：总资产、可用资产、缺口、告警摘要

---

## 2. 流动性池监控（Liquidity Pool Monitoring）扩展

### 2.1 监控对象

- 资金被部署到链上协议（AMM、借贷、做市策略）形成的“池子/仓位”

### 2.2 核心指标（文档应定义口径）

- TVL / Position Value（USD）
- Utilization Rate（利用率）
- Reserve Ratio（储备率）
- Health Score（综合健康评分：阈值+异常+同步）
- 费用与收益（APR/APY、手续费收入）

### 2.3 告警与动作（建议）

- 利用率/储备率穿透阈值：warning/critical
- 大额异常流出、资产 depeg 风险
- 再平衡建议：从哪个 Vault/池子调拨多少到哪里（仅建议，不直接执行）
