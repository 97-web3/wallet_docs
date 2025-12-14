# Swap 配置（索引）

> 返回 [后台管理前端目录](./README.md)

---

## 页面概述

- **页面路径**：`/admin/swap`
- **权限要求**：Admin
- **适用钱包**：仅 Web3 钱包支持 Swap（Web2 托管钱包不支持）

## 模块定位（重要）

本仓库的 Swap/闪兑为 **多链第三方聚合通道集成模型**：

- 支持网络：EVM（Ethereum/BSC/Polygon 等）与 Tron
- 集成模式：接入第三方 Swap/报价/路由提供商（类似 1inch/0x/ParaSwap 等），**不自营资金池**

因此，本页面的“配置能力”应以 **Provider/Channel 管理、Token/交易对治理、费率与折算、滑点与交易参数、监控与审计** 为核心。

> 术语要求：在价值追踪、计价币种、限额、统计、对账等语境下，统一使用“折算”，避免与“兑换（Swap 执行）”混用。

---

## 文档结构（拆分后的目录）

为避免单文件过于拥挤，已将 Swap 配置文档拆分为以下子文档（仍然只对应 `/admin/swap` 这一页面入口）：

> 说明：文档按主题拆分并不意味着前端必须新增 Tab。页面可继续保持 3 个 Tab（例如 Provider/滑点/历史），其余内容以“页面内配置区块/折叠面板”承载即可。

1. [swap/01-Provider管理.md](swap/01-Provider管理.md)
2. [swap/02-Token与交易对.md](swap/02-Token与交易对.md)
3. [swap/03-费率与折算.md](swap/03-费率与折算.md)
4. [swap/04-滑点与交易参数.md](swap/04-滑点与交易参数.md)
5. [swap/05-监控与历史.md](swap/05-监控与历史.md)
6. [swap/06-告警与审计.md](swap/06-告警与审计.md)

---

## 与后端接口的映射（参考）

### 字段命名与 SRD 归属（避免字段漂移）

以 `wallet_docs/admin-api/srds/06-Swap配置.md` 为字段与校验范围的唯一来源。文档中的字段应尽量写成“分组路径”，例如：

- `GET/PUT /api/v1/admin/swap/config`
	- `global_settings.*`（开关、限额、fee）
	- `slippage_settings.*`（默认滑点、最大滑点、告警阈值）
	- `price_settings.*`（`price_validity_seconds`、`price_source`、`fallback_sources`）
	- `risk_settings.*`（高额阈值、2FA 阈值）
- `GET /api/v1/admin/swap/rates/config`
	- `primary_source`、`sources[]`、`update_interval_seconds`、`deviation_threshold`、`fallback_behavior`
- `GET /api/v1/admin/swap/providers` / `PUT /api/v1/admin/swap/providers/{id}`
	- Provider 字段（含 `health_check.*`）
- `GET /api/v1/admin/swap/pairs` / `POST /api/v1/admin/swap/tokens`
	- Pair/Token 字段

以下接口在 SRD 中已定义，建议前端文档与之对齐（字段与校验范围以 SRD 为准）：

- `GET /api/v1/admin/swap/config`
- `PUT /api/v1/admin/swap/config`
- `GET /api/v1/admin/swap/providers`
- `PUT /api/v1/admin/swap/providers/{id}`
- `GET /api/v1/admin/swap/pairs`
- `POST /api/v1/admin/swap/tokens`
- `GET /api/v1/admin/swap/rates/config`
- `GET /api/v1/admin/swap/statistics`

详见 `wallet_docs/admin-api/srds/06-Swap配置.md`。

---

> 返回 [后台管理前端目录](./README.md)
