# 多语言/国际化规格 - 通用规格文档

## 文档标识

| 属性 | 值 |
|------|-----|
| 文档ID | COMMON-011 |
| 文档名称 | 多语言/国际化规格 |
| 版本 | 1.0 |
| 创建日期 | 2025-12-09 |
| 状态 | 草稿 |

### 概述

本文档定义了钱包应用的多语言支持规范，包括支持的语言列表、语言切换规则、文案规范和格式化规则。

> 相关规格: [COMMON-006 术语表](./glossary.md) | [PRD 第五章 文案](../../prd/企业内部钱包V1.0.md)

---

## 1. 支持语言

### 1.1 语言列表

| 语言代码 | 语言名称 | 本地化名称 | 状态 |
|---------|---------|-----------|------|
| zh-CN | 简体中文 | 简体中文 | 默认语言 |
| en-US | English (US) | English | 支持 |

### 1.2 语言优先级

1. 用户手动设置的语言
2. 系统/设备语言（如匹配支持列表）
3. 默认语言（zh-CN）

---

## 2. 语言切换规则

### 2.1 切换入口

| 位置 | 说明 |
|------|------|
| 个人中心 > 语言 | 主要切换入口，显示当前语言 |

### 2.2 切换行为

| 规则 | 说明 |
|------|------|
| 即时生效 | 切换后立即应用到全局，无需重启 |
| 持久化存储 | 语言设置保存到本地，下次启动自动应用 |
| 适用范围 | Web2 钱包和 Web3 钱包全局生效 |

---

## 3. 文案规范表

### 3.1 Web3 钱包页面

| 中文 (zh-CN) | 英文 (en-US) | 适用场景 |
|-------------|--------------|---------|
| 平台账户 | Platform Account | 钱包切换 |
| Web3 钱包 | Web3 Wallet | 钱包切换 |
| 创建新钱包 | Create New Wallet | Landing页面 |
| 导入已有钱包 | Import Wallet | Landing页面 |

### 3.2 助记词相关

| 中文 (zh-CN) | 英文 (en-US) | 适用场景 |
|-------------|--------------|---------|
| 重要：不要冒丢失钱包的风险，在信任的地点保存您的私钥助记词，以此保护您的钱包。如果您被应用锁定或更换新设备，这是找回钱包的唯一途径 | Important: Do not risk losing your wallet. Store your mnemonic phrase in a trusted and secure location to protect your wallet. If you are locked out of the app or switch to a new device, this is the only way to recover your wallet. | 助记词备份提示 |
| 点击即可显示，请确保没有人在看您的屏幕 | Tap to reveal. Please ensure no one is viewing your screen. | 助记词蒙层 |
| 这是您的私钥助记词，请按正确顺序写下它并妥善保管。如果任何人拥有您的私钥助记词，就能访问您的钱包。切勿与任何人共享该助记词 | This is your mnemonic phrase. Write it down in the correct order and keep it securely. Anyone with access to this phrase can access your wallet. Never share it with anyone. | 助记词文案提示 |
| 验证 | Verify | 验证按钮 |

### 3.3 首页功能

| 中文 (zh-CN) | 英文 (en-US) | 适用场景 |
|-------------|--------------|---------|
| 总资产 | Total Asset | 首页资产显示 |
| 发送 | Send | 功能按钮 |
| 接收 | Receive | 功能按钮 |
| 闪兑 | Swap | 功能按钮 |
| 交易记录 | Transaction Record | 功能入口 |
| 全部 | All | 筛选选项 |

### 3.4 交易记录

| 中文 (zh-CN) | 英文 (en-US) | 适用场景 |
|-------------|--------------|---------|
| 交易记录 | Transaction History | 页面标题 |
| 接收 xxx | Receive xxx | 交易类型 |
| 发送 xxx | Send xxx | 交易类型 |
| 闪兑 xxx | Swap xxx | 交易类型 |

### 3.5 设置页面

| 中文 (zh-CN) | 英文 (en-US) | 说明 |
|-------------|--------------|------|
| 生物识别 | Biometric Authentication | 支持指纹和人脸 |
| 地址簿 | Address Book | 保存的钱包地址 |
| 语言 | Language | 显示当前语言 |
| 关联平台账户 | Linked Platform Account | 关联的钱包地址可以免审批提币 |

### 3.6 交易状态

| 中文 (zh-CN) | 英文 (en-US) | 状态码 |
|-------------|--------------|--------|
| 已发起 | Initiated | pending |
| 处理中 / 交易中 | Processing | processing |
| 已完成 | Completed | completed |
| 已驳回 | Rejected | rejected |
| 失败 | Failed | failed |

---

## 4. 格式化规则

### 4.1 数字格式

| 类型 | zh-CN | en-US | 示例 |
|------|-------|-------|------|
| 千分位 | , | , | 1,234,567 |
| 小数点 | . | . | 0.1234567 |
| 代币数量 | 7位小数 | 7位小数 | 123.4567890 |
| 代币价值 | 2位小数 | 2位小数 | $1,234.56 |

### 4.2 日期时间格式

| 类型 | zh-CN | en-US |
|------|-------|-------|
| 完整日期 | YYYY年MM月DD日 | MM/DD/YYYY |
| 完整时间 | HH:mm:ss | HH:mm:ss |
| 日期时间 | YYYY-MM-DD HH:mm:ss | MM/DD/YYYY HH:mm:ss |
| 相对时间 | X分钟前 / X小时前 | X minutes ago / X hours ago |

### 4.3 货币格式

| 货币 | zh-CN | en-US |
|------|-------|-------|
| CNY | ¥1,234.56 | ¥1,234.56 |
| USD | $1,234.56 | $1,234.56 |

---

## 5. 实现规范

### 5.1 文案存储

```json
{
  "key": "wallet.create",
  "translations": {
    "zh-CN": "创建新钱包",
    "en-US": "Create New Wallet"
  }
}
```

### 5.2 动态参数

```json
{
  "key": "transaction.receive",
  "translations": {
    "zh-CN": "接收 {amount} {token}",
    "en-US": "Receive {amount} {token}"
  }
}
```

### 5.3 复数形式

```json
{
  "key": "wallet.count",
  "translations": {
    "zh-CN": "{count} 个钱包",
    "en-US": {
      "one": "{count} wallet",
      "other": "{count} wallets"
    }
  }
}
```

---

## 6. 验收标准

1. [ ] 支持 zh-CN 和 en-US 两种语言
2. [ ] 语言切换即时生效
3. [ ] 语言设置持久化存储
4. [ ] 所有UI文案有对应翻译
5. [ ] 数字格式化符合规范
6. [ ] 日期时间格式化符合规范
7. [ ] 动态参数正确替换
8. [ ] 缺失翻译时回退到默认语言

---

*文档创建日期: 2025-12-09*
