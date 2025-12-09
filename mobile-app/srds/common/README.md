# 公共规格文档 (Common SRDs)

本目录包含适用于中心化钱包和去中心化钱包的公共规格文档。

## 文档索引

| 文档ID | 文档名称 | 说明 |
|--------|----------|------|
| COMMON-001 | [代币精度规则](./token-precision-rules.md) | 代币数量/价值的精度和格式化规则 |
| COMMON-002 | [验证方式规则](./verification-method-rules.md) | FaceID/2FA/资金密码的优先级和验证逻辑 |
| COMMON-003 | [区块链网络规格](./blockchain-networks.md) | TRX/BNB/ETH 网络支持及扩展性设计 |
| COMMON-004 | [钱包切换逻辑](./wallet-switch-logic.md) | Web2↔Web3 钱包切换流程 |
| COMMON-005 | [实时汇率集成](./price-feed-integration.md) | Debank 价格源及缓存策略 |
| COMMON-006 | [术语表](./glossary.md) | PRD/SRD 统一术语定义 |
| COMMON-007 | [推送通知规则](./notification-rules.md) | 通知触发条件、渠道和内容模板 |
| COMMON-008 | [错误码规格](./error-codes.md) | 错误码命名规范和分类详情 |
| COMMON-011 | [多语言/国际化规格](./i18n-specifications.md) | 支持语言、文案规范和格式化规则 |

## 适用范围

这些公共规格适用于以下钱包类型：
- **中心化钱包 (平台账户/Web2 钱包)**
- **去中心化钱包 (Web3 钱包)**

## 引用方式

在其他 SRD 文档中引用公共规格时，使用以下格式：

```markdown
> 验证方式详见 [COMMON-002 验证方式规则](../common/verification-method-rules.md)
```

## 版本信息

| 属性 | 值 |
|------|-----|
| 版本 | 1.2 |
| 创建日期 | 2025-12-08 |
| 最后更新 | 2025-12-09 |

---

## 更新记录

| 日期 | 版本 | 描述 |
|------|------|------|
| 2025-12-08 | 1.0 | 初始版本，包含 COMMON-001 至 COMMON-005 |
| 2025-12-09 | 1.1 | 新增术语表(COMMON-006)、通知规则(COMMON-007)、错误码(COMMON-008) |
| 2025-12-09 | 1.2 | 新增多语言/国际化规格(COMMON-011) |

---

*此目录由 PRD-SRD 分析修正任务创建*
