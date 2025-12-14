# Swap / Token 与交易对管理

> 返回 [Swap 配置索引](../06-Swap配置.md)

---

## 定位

本部分用于管理：
- **可参与 Swap 的 Token（按链）**：白名单/黑名单、合约地址与精度、启用状态。
- **可参与 Swap 的交易对（Pair）**：支持范围、是否启用、最小/最大交易量、可用 Provider。

> 说明：这部分是生产必要能力，用于降低恶意 Token、错误合约、低流动性资产导致的运维事故。

---

## 1. Token 管理（按链）

### 1.1 核心字段（建议）

| 字段 | 说明 |
|---|---|
| network | Ethereum/BSC/Polygon/Tron 等 |
| symbol/name | 代币符号与名称 |
| contract_address | EVM: 0x…；Tron: TRC20 合约地址格式 |
| decimals | 精度 |
| enabled | 是否在资产列表/系统内启用（与 swap_enabled 区分） |
| swap_enabled | 是否允许参与 Swap |
| min_amount / max_amount | 单笔最小/最大交易量（可按 Token 或按 Pair 覆盖） |
| risk_tag | 风险标签（例如 high_risk / stable / memecoin） |

#### 1.1.1 SRD 字段对齐（来源：`POST /api/v1/admin/swap/tokens`）

> 说明：以下为 SRD 已定义字段；如需新增 `risk_tag` 等扩展字段，应以“后端是否已支持”为准。

| UI/文档字段 | SRD 路径 | 备注 |
|---|---|---|
| network | token.network | 与 SRD 一致 |
| symbol | token.symbol | 文档中的 `symbol/name` 建议拆分展示 |
| name | token.name | 同上 |
| contract_address | token.contract_address | 多链校验规则见 1.2 |
| decimals | token.decimals | 建议链上读取校验 |
| enabled | token.enabled | 与 `swap_enabled` 区分：是否在资产列表启用 vs 是否允许 Swap |
| swap_enabled | token.swap_enabled | 与 SRD 一致 |
| min_amount | token.min_amount | 与 SRD 一致 |
| max_amount | token.max_amount | 与 SRD 一致 |
| icon_url | token.icon_url | 可选展示 |
| risk_tag | （SRD 未定义） | 可选扩展字段（需后端支持） |

### 1.2 合约地址校验（多链差异）

- EVM：`0x` 开头、长度、checksum（如采用）
- Tron：地址格式与编码规则不同；建议复用 [98-地址校验规范](../98-地址校验规范.md)
- 建议增加“链上读取校验”（文档层面要求）：
  - 读取 `symbol/decimals/name` 与录入是否一致
  - 合约是否存在/是否可调用

---

## 2. 交易对（Pair）管理

### 2.1 列表与筛选（建议）

- 筛选：network、base_currency、quote_currency、enabled、Provider
- 列表字段建议：
  - Pair ID（全局唯一）
  - base/quote（注意：这里 quote 指“计价币种”，用于**折算**与展示）
  - enabled
  - providers（该 Pair 可用 Provider 集合）
  - min_amount / max_amount
  - 近 24h：成交量（USD）、成功率、平均滑点

#### 2.1.1 SRD 字段对齐（来源：`GET /api/v1/admin/swap/pairs`）

| UI/文档字段 | SRD 路径 | 备注 |
|---|---|---|
| Pair ID | pairs[].id | 与 SRD 一致 |
| base_currency | pairs[].base_currency | 文档中的 base 对应该字段 |
| quote_currency | pairs[].quote_currency | 文档中的 quote 为计价币种（折算口径） |
| network | pairs[].network | 与 SRD 一致 |
| enabled | pairs[].enabled | 与 SRD 一致 |
| providers | pairs[].providers | Provider ID 列表 |
| min_amount | pairs[].min_amount | 与 SRD 一致 |
| max_amount | pairs[].max_amount | 与 SRD 一致 |
| current_rate | pairs[].current_rate | 展示字段（注意与“折算价源”口径区分） |
| rate_24h_change | pairs[].rate_24h_change | 展示字段 |
| volume_24h_usd | pairs[].volume_24h_usd | 统计字段 |
| liquidity_usd | pairs[].liquidity_usd | 可选展示字段（SRD 示例中出现） |

### 2.2 Pair 级路由覆盖（建议）

- 某些 Pair 指定固定 Provider（例如稳定币对优先特定渠道）
- 某些 Pair 禁用某些 Provider（例如对某些 Token 报价不稳定）

---

## 3. 黑白名单策略（建议）

- **Token 黑名单**：
  - 风险资产、钓鱼合约、被监管限制资产
- **Pair 黑名单**：
  - 流动性不足、滑点过高、失败率高
- **用户地址黑名单**：
  - 建议与“黑名单地址管理”模块联动（本文件仅定义联动需求，不展开实现）

---

## 3.1 上线前检查清单（10 条以内，建议强制执行）

1. 合约地址格式校验（EVM/Tron 分别校验）。
2. 链上读取校验：`symbol/decimals/name` 与录入一致。
3. 风险核对：是否在 Token/Pairs 黑名单、是否需要标记 `risk_tag`。
4. 配置 Token 的 `swap_enabled` 初始为关闭；先完成测试再开启。
5. 设置最小/最大交易量（Token 级 + Pair 级覆盖策略要明确）。
6. 为关键 Pair 指定可用 Provider 列表（至少 1 主 + 1 备）。
7. 配置默认计价币种（用于折算与限额口径）并确认价格来源可用。
8. 通过“测试查询”验证至少 3 个金额档位（小额/中额/接近上限）。
9. 检查失败率与滑点是否在可接受区间（否则先调整 Provider 或禁用 Pair）。
10. 灰度启用（先限定链/对/范围），观察稳定后再全量。

---

## 4. 对应后端接口（参考）

- `POST /api/v1/admin/swap/tokens`（add/update/remove）
- `GET /api/v1/admin/swap/pairs`

（详见 `wallet_docs/admin-api/srds/06-Swap配置.md`）

---

> 返回 [Swap 配置索引](../06-Swap配置.md)
