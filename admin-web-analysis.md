# Admin Web 分析与后端接口映射

> 本文基于仓库中的前端页面规范文档(`admin-web/*.md`)与现有 SRDS(`wallet-core/srds`)进行系统化梳理，不臆测未在代码/文档出现的接口。所有映射均可溯源到具体文件与行号。

---

## 页面清单与路径

- `/login` 登录与认证（admin-web/01-登录与认证.md:9）
- `/admin/dashboard` Dashboard（admin-web/02-Dashboard.md:9）
- `/admin/users` 用户管理（admin-web/03-用户管理.md:9）
- `/admin/withdrawals` 提现审批（admin-web/04-提现审批.md:9）
- `/admin/ledger` 账本与交易（admin-web/05-账本与交易.md:9）
- `/admin/swap` Swap 配置（admin-web/06-Swap配置.md:9）
- `/admin/settings` 系统设置（admin-web/07-系统设置.md:9）
- `/admin/reports` 报表与导出（admin-web/08-报表与导出.md:9）
- `/admin/web3users` Web3 用户管理（admin-web/09-Web3用户管理.md:9）
- `/admin/vault` Vault 资金管理（admin-web/10-Vault资金管理.md:9）
- `/admin/transfer` 转账管理（admin-web/11-转账管理.md:9）
- `/admin/blacklist` 黑名单地址管理（admin-web/12-黑名单地址管理.md:9）
- `/admin/currencies` 币种管理（admin-web/13-币种管理.md:9）
- `/admin/status` 系统状态监控（admin-web/14-系统状态监控.md:9）

备注：admin-web/README.md 也提供了页面索引与说明（admin-web/README.md:29）。

---

## 后端接口来源

- Admin Web 页面内显式列出的 API（仅 Dashboard 明确列出）
  - admin-web/02-Dashboard.md:998 起“API 接口”小节列出 7 个 GET 接口。
- API Gateway 路由清单（含 Admin Service 路由）
  - wallet-core/srds/01-api-gateway/09-路由列表.md:83 起，枚举了 Admin 相关 HTTP → gRPC 映射与角色要求。

两者合并后形成本次可确认的 Admin 后端接口集合。尚未在文档出现的页面（如黑名单、报表等）的具体接口不可据此臆测，已在下文标注为“未在代码中出现”。

---

## 页面 → 接口映射

- Dashboard `/admin/dashboard`
  - GET `/api/v1/admin/dashboard/statistics`（admin-web/02-Dashboard.md:1004）
  - GET `/api/v1/admin/dashboard/flow-trend`（admin-web/02-Dashboard.md:1005）
  - GET `/api/v1/admin/dashboard/transfer-trend`（admin-web/02-Dashboard.md:1006）
  - GET `/api/v1/admin/dashboard/user-status`（admin-web/02-Dashboard.md:1007）
  - GET `/api/v1/admin/dashboard/withdrawal-status`（admin-web/02-Dashboard.md:1008）
  - GET `/api/v1/admin/dashboard/currency-balance`（admin-web/02-Dashboard.md:1009）
  - GET `/api/v1/admin/dashboard/recent-activities`（admin-web/02-Dashboard.md:1010）
  - GET `/api/v1/admin/dashboard/vault-balances`（wallet-core/srds/01-api-gateway/09-路由列表.md:92）
  - GET `/api/v1/admin/dashboard/alerts`（wallet-core/srds/01-api-gateway/09-路由列表.md:93）
  - POST `/api/v1/admin/dashboard/alerts`（wallet-core/srds/01-api-gateway/09-路由列表.md:94）

- 用户管理 `/admin/users`
  - GET `/api/v1/admin/users`（wallet-core/srds/01-api-gateway/09-路由列表.md:100）
  - POST `/api/v1/admin/users/{uid}/freeze`（wallet-core/srds/01-api-gateway/09-路由列表.md:101）
  - POST `/api/v1/admin/users/{uid}/unfreeze`（wallet-core/srds/01-api-gateway/09-路由列表.md:102）
  - POST `/api/v1/admin/users/{uid}/terminate`（wallet-core/srds/01-api-gateway/09-路由列表.md:103）
  - PUT `/api/v1/admin/users/{uid}/role`（wallet-core/srds/01-api-gateway/09-路由列表.md:104）

- 转账管理（含薪资发放）`/admin/transfer`
  - POST `/api/v1/admin/payroll/import`（wallet-core/srds/01-api-gateway/09-路由列表.md:110）
  - POST `/api/v1/admin/payroll/{batch_id}/process`（wallet-core/srds/01-api-gateway/09-路由列表.md:111）
  - GET `/api/v1/admin/payroll/history`（wallet-core/srds/01-api-gateway/09-路由列表.md:112）
  - GET `/api/v1/admin/payroll/{batch_id}/export`（wallet-core/srds/01-api-gateway/09-路由列表.md:113）

- 提现审批 `/admin/withdrawals`
  - GET `/api/v1/admin/withdraw/pending`（wallet-core/srds/01-api-gateway/09-路由列表.md:119）
  - POST `/api/v1/admin/withdraw/{id}/approve`（wallet-core/srds/01-api-gateway/09-路由列表.md:120）
  - POST `/api/v1/admin/withdraw/{id}/reject`（wallet-core/srds/01-api-gateway/09-路由列表.md:121）

- Swap 配置 `/admin/swap`
  - GET `/api/v1/admin/swap/config`（wallet-core/srds/01-api-gateway/09-路由列表.md:127）
  - PUT `/api/v1/admin/swap/config`（wallet-core/srds/01-api-gateway/09-路由列表.md:128）
  - PUT `/api/v1/admin/swap/providers/{id}`（wallet-core/srds/01-api-gateway/09-路由列表.md:129）
  - POST `/api/v1/admin/swap/tokens`（wallet-core/srds/01-api-gateway/09-路由列表.md:130）

- 角色与权限（系统设置）`/admin/settings/roles`
  - GET `/api/v1/admin/roles`（wallet-core/srds/01-api-gateway/09-路由列表.md:136）
  - POST `/api/v1/admin/roles/assign`（wallet-core/srds/01-api-gateway/09-路由列表.md:137）
  - GET `/api/v1/admin/users/{uid}/permissions`（wallet-core/srds/01-api-gateway/09-路由列表.md:138）

- 账本与 Vault `/admin/ledger`、`/admin/vault`
  - GET `/api/v1/admin/ledger`（wallet-core/srds/01-api-gateway/09-路由列表.md:144）
  - POST `/api/v1/admin/ledger/export`（wallet-core/srds/01-api-gateway/09-路由列表.md:145）
  - POST `/api/v1/admin/vault/adjust`（wallet-core/srds/01-api-gateway/09-路由列表.md:146）
  - GET `/api/v1/admin/vault/adjustments`（wallet-core/srds/01-api-gateway/09-路由列表.md:147）
  - POST `/api/v1/admin/vault/adjustments/{id}/approve`（wallet-core/srds/01-api-gateway/09-路由列表.md:148）

- 下列页面未在现有代码/文档中找到明确的 Admin 接口：
  - `/admin/blacklist` 黑名单地址管理（admin-web/12-黑名单地址管理.md）
  - `/admin/reports` 报表与导出（admin-web/08-报表与导出.md）
  - `/admin/currencies` 币种管理（admin-web/13-币种管理.md）
  - `/admin/status` 系统状态监控（admin-web/14-系统状态监控.md）
  - `/admin/web3users` Web3 用户管理（admin-web/09-Web3用户管理.md）

> 以上模块虽有前端页面说明，但未在 API Gateway 管理路由或其它 SRDS 文件中出现对应 HTTP 路由；建议后续补充。

---

## 接口目的摘要

- Dashboard：统计卡片、趋势图、分布图、最近动态、Vault 余额与告警阈值（admin-web/02-Dashboard.md:998；wallet-core/srds/01-api-gateway/09-路由列表.md:91）
- 用户管理：用户分页查询、冻结/解冻、离职、更新角色（wallet-core/srds/01-api-gateway/09-路由列表.md:100）
- 转账/薪资：批次导入、执行、历史、导出（wallet-core/srds/01-api-gateway/09-路由列表.md:110）
- 提现审批：待审批列表、通过、拒绝（wallet-core/srds/01-api-gateway/09-路由列表.md:119）
- Swap 配置：配置项与 Provider/Token 维护（wallet-core/srds/01-api-gateway/09-路由列表.md:127）
- 角色与权限：角色列表、分配、用户权限查询（wallet-core/srds/01-api-gateway/09-路由列表.md:136）
- 账本与 Vault：账本分页与导出、余额调整流程与审批（wallet-core/srds/01-api-gateway/09-路由列表.md:144）

---

## 备注与建议

- 敏感操作遵循幂等键规范（见 wallet-core/srds/01-api-gateway/07-HTTP规范.md:120），涉及接口：
  - POST `/api/v1/admin/vault/adjust`
  - POST `/api/v1/admin/withdraw/{id}/approve`
  - POST `/api/v1/admin/payroll/{batch_id}/process`
- 列表接口使用统一分页结构（wallet-core/srds/01-api-gateway/07-HTTP规范.md:18）。
- 建议为“黑名单、报表、币种、状态、Web3 用户”补齐 Admin 路由与 SRDS 以闭环对齐前端页面。

