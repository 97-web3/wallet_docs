# 中心化钱包 - 软件需求规格说明书索引

## 概述

本目录包含中心化钱包（平台账户/Web2 钱包）应用所有界面的软件需求规格说明书（SRD）。每个文档详细描述了对应界面的功能需求、UI组件、用户交互、数据需求、验证规则、技术考虑和验收标准。

---

## 公共规格文档

以下公共规格适用于中心化钱包和去中心化钱包，请优先参阅：

| 文档 | 说明 |
|------|------|
| [COMMON-001 代币精度规则](../common/token-precision-rules.md) | 代币数量 7 位小数，价值 2 位小数 |
| [COMMON-002 验证方式规则](../common/verification-method-rules.md) | FaceID > 2FA > 资金密码（替代验证） |
| [COMMON-003 区块链网络规格](../common/blockchain-networks.md) | V1.0 支持 TRX/BNB/ETH |
| [COMMON-004 钱包切换逻辑](../common/wallet-switch-logic.md) | Web2 ↔ Web3 切换流程 |
| [COMMON-005 实时汇率集成](../common/price-feed-integration.md) | Debank 价格源 |
| [COMMON-006 术语表](../common/glossary.md) | PRD/SRD 统一术语定义 |
| [COMMON-007 推送通知规则](../common/notification-rules.md) | 通知触发和内容模板 |
| [COMMON-008 错误码规格](../common/error-codes.md) | 错误码规范和分类 |

---

## 项目信息

| 属性 | 值 |
|------|-----|
| 项目名称 | 中心化钱包（平台账户/Web2 钱包） |
| 文档版本 | 1.2 |
| 创建日期 | 2025-12-08 |
| 最后更新 | 2025-12-09 |
| 总页面数 | 51 |
| 文档语言 | 中文 |

### V1.0 支持的区块链网络

| 网络 | 代币标准 |
|------|----------|
| TRX (Tron) | TRC-20 |
| BNB (BSC) | BEP-20 |
| ETH (Ethereum) | ERC-20 |

---

## 文档列表

### 一、登录与认证（6个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 01 | [01-login-requirements.md](./01-login-requirements.md) | 登录页面 | CW-001 |
| 30 | [30-input-password-requirements.md](./30-input-password-requirements.md) | 输入密码登录 | CW-030 |
| 31 | [31-input-password-filled-requirements.md](./31-input-password-filled-requirements.md) | 输入密码登录(已填写) | CW-031 |
| 32 | [32-input-verification-code-requirements.md](./32-input-verification-code-requirements.md) | 输入验证码登录 | CW-032 |
| 33 | [33-input-error-requirements.md](./33-input-error-requirements.md) | 输入错误状态 | CW-033 |
| 34 | [34-input-error-variant-requirements.md](./34-input-error-variant-requirements.md) | 输入错误状态(验证码) | CW-034 |

### 二、钱包主页与资产（2个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 02 | [02-centralized-wallet-home-requirements.md](./02-centralized-wallet-home-requirements.md) | 中心化钱包主页 | CW-002 |
| 03 | [03-token-details-requirements.md](./03-token-details-requirements.md) | 代币详情 | CW-003 |

### 三、交易记录（3个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 04 | [04-transaction-list-requirements.md](./04-transaction-list-requirements.md) | 交易列表 | CW-004 |
| 05 | [05-transaction-detail-deposit-requirements.md](./05-transaction-detail-deposit-requirements.md) | 交易详情-充值 | CW-005 |
| 06 | [06-transaction-detail-2-requirements.md](./06-transaction-detail-2-requirements.md) | 交易详情-变体 | CW-006 |

### 四、充值流程（4个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 07 | [07-deposit-select-token-requirements.md](./07-deposit-select-token-requirements.md) | 充值-选择代币 | CW-007 |
| 08 | [08-select-network-requirements.md](./08-select-network-requirements.md) | 选择网络 | CW-008 |
| 09 | [09-view-deposit-address-requirements.md](./09-view-deposit-address-requirements.md) | 查看充值地址 | CW-009 |
| 10 | [10-deposit-details-requirements.md](./10-deposit-details-requirements.md) | 充值详情 | CW-010 |

### 五、提现流程（12个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 11 | [11-withdraw-select-token-requirements.md](./11-withdraw-select-token-requirements.md) | 提现-选择代币 | CW-011 |
| 12 | [12-withdraw-select-token-2-requirements.md](./12-withdraw-select-token-2-requirements.md) | 提现-选择代币2 | CW-012 |
| 13 | [13-select-withdraw-method-requirements.md](./13-select-withdraw-method-requirements.md) | 选择提现方式 | CW-013 |
| 14 | [14-internal-withdraw-requirements.md](./14-internal-withdraw-requirements.md) | 内部划转 | CW-014 |
| 15 | [15-withdraw-select-network-requirements.md](./15-withdraw-select-network-requirements.md) | 提现-选择网络 | CW-015 |
| 16 | [16-chain-recipient-requirements.md](./16-chain-recipient-requirements.md) | 链上提现-收款方 | CW-016 |
| 17 | [17-chain-withdraw-details-requirements.md](./17-chain-withdraw-details-requirements.md) | 链上提现详情 | CW-017 |
| 18 | [18-chain-withdraw-error-requirements.md](./18-chain-withdraw-error-requirements.md) | 链上提现-错误 | CW-018 |
| 19 | [19-chain-confirm-requirements.md](./19-chain-confirm-requirements.md) | 链上提现确认 | CW-019 |
| 20 | [20-chain-confirm-2-requirements.md](./20-chain-confirm-2-requirements.md) | 链上提现确认2 | CW-020 |
| 21 | [21-chain-confirm-3-requirements.md](./21-chain-confirm-3-requirements.md) | 链上提现确认3 | CW-021 |
| 22 | [22-confirm-requirements.md](./22-confirm-requirements.md) | 提现确认 | CW-022 |

### 六、安全验证（7个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 23 | [23-faceid-verification-requirements.md](./23-faceid-verification-requirements.md) | FaceID验证 | CW-023 |
| 24 | [24-faceid-verification-success-requirements.md](./24-faceid-verification-success-requirements.md) | FaceID验证成功 | CW-024 |
| 25 | [25-faceid-verification-scanning-requirements.md](./25-faceid-verification-scanning-requirements.md) | FaceID扫描中 | CW-025 |
| 26 | [26-security-verification-requirements.md](./26-security-verification-requirements.md) | 安全验证(2FA) | CW-026 |
| 27 | [27-security-verification-filled-requirements.md](./27-security-verification-filled-requirements.md) | 安全验证(已输入) | CW-027 |
| 28 | [28-security-verification-error-requirements.md](./28-security-verification-error-requirements.md) | 安全验证(错误) | CW-028 |
| 29 | [29-verification-loading-requirements.md](./29-verification-loading-requirements.md) | 验证加载中 | CW-029 |

### 七、收款方管理（3个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 35 | [35-edit-recipient-requirements.md](./35-edit-recipient-requirements.md) | 编辑收款方 | CW-035 |
| 36 | [36-edit-recipient-filled-requirements.md](./36-edit-recipient-filled-requirements.md) | 编辑收款方(已填写) | CW-036 |
| 37 | [37-address-book-internal-requirements.md](./37-address-book-internal-requirements.md) | 通讯录(内部) | CW-037 |

### 八、通知与消息（2个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 38 | [38-notifications-requirements.md](./38-notifications-requirements.md) | 通知列表 | CW-038 |
| 39 | [39-notifications-empty-requirements.md](./39-notifications-empty-requirements.md) | 通知(空状态) | CW-039 |

### 九、系统组件与其他（5个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 40 | [40-alert-popup-requirements.md](./40-alert-popup-requirements.md) | 弹窗提示 | CW-040 |
| 41 | [41-resend-requirements.md](./41-resend-requirements.md) | 重新发送 | CW-041 |
| 42 | [42-frame-component-requirements.md](./42-frame-component-requirements.md) | 框架组件 | CW-042 |
| 43 | [43-rectangle-component-requirements.md](./43-rectangle-component-requirements.md) | 矩形组件 | CW-043 |
| 44 | [44-misc-component-requirements.md](./44-misc-component-requirements.md) | 其他组件 | CW-044 |

### 十、账户绑定与安全设置（2个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 45 | [45-platform-account-binding-requirements.md](./45-platform-account-binding-requirements.md) | 平台账户绑定 | CW-045 |
| 46 | [46-ga-binding-requirements.md](./46-ga-binding-requirements.md) | Google Authenticator 绑定 | CW-046 |

### 十一、闪兑功能（5个页面）

| 序号 | 文档 | 页面名称 | 页面ID |
|------|------|----------|--------|
| 47 | [47-swap-select-token-requirements.md](./47-swap-select-token-requirements.md) | 闪兑-选择代币 | CW-047 |
| 48 | [48-swap-details-requirements.md](./48-swap-details-requirements.md) | 闪兑详情 | CW-048 |
| 49 | [49-swap-confirm-requirements.md](./49-swap-confirm-requirements.md) | 闪兑确认 | CW-049 |
| 50 | [50-swap-result-requirements.md](./50-swap-result-requirements.md) | 闪兑结果 | CW-050 |
| 51 | [51-swap-history-requirements.md](./51-swap-history-requirements.md) | 闪兑记录 | CW-051 |

---

## 业务流程图

### 充币流程
```
主页 → 选择币种 → 选择网络 → 查看充值地址和二维码 → 完成链上转账 → 交易记录
```

### 提币流程（内部划转）
```
主页 → 选择币种 → 选择方式（内部划转） → 输入UID/邮箱/手机号和数量 → 安全验证 → 确认信息 → 提交成功
```
> 注：安全验证采用替代模式（FaceID/2FA/资金密码任选其一）

### 提币流程（链上转账）
```
主页 → 选择币种 → 选择方式（链上转账） → 选择网络 → 输入地址和数量 → 确认信息 → 安全验证 → 提交成功
```

### 闪兑流程
```
主页 → 选择代币对 → 输入金额（查看汇率/滑点） → 安全验证 → 确认兑换 → 等待结果 → 完成/失败
```

---

## 文档结构

每个SRD文档包含以下8个标准章节：

1. **页面标识** - 页面ID、名称、版本、状态等元数据
2. **UI组件** - 布局概述和组件详情表格
3. **功能需求** - 详细的功能需求规格（FR-XXX格式）
4. **用户交互** - 用户工作流、导航流程、手势交互
5. **数据需求** - 显示数据和输入数据规格
6. **验证规则** - 输入验证和业务规则
7. **技术考虑** - API集成、状态管理、安全考虑
8. **验收标准汇总** - 可测试的验收标准清单

---

## 功能模块统计

| 功能模块 | 文档数量 |
|----------|----------|
| 登录与认证 | 6 |
| 钱包主页与资产 | 2 |
| 交易记录 | 3 |
| 充值流程 | 4 |
| 提现流程 | 12 |
| 安全验证 | 7 |
| 收款方管理 | 3 |
| 通知与消息 | 2 |
| 系统组件与其他 | 5 |
| 账户绑定与安全设置 | 2 |
| 闪兑功能 | 5 |
| **总计** | **51** |

---

## 核心功能特性

### 中心化钱包特点
- ✅ 托管模式，平台管理私钥
- ✅ 支持内部划转（0 手续费，需审核）
- ✅ 支持链上转账（需手续费，无需审核）
- ✅ 多币种支持，V1.0 支持 TRX/BNB/ETH 网络
- ✅ 完整的交易记录追踪（已发起→交易中→已完成）

### 安全机制
- ✅ 替代验证模式（FaceID/2FA/资金密码任选其一）
- ✅ FaceID 生物识别验证（优先级 A）
- ✅ Google Authenticator 2FA 验证（优先级 B）
- ✅ 资金密码验证（优先级 C）
- ✅ 地址格式验证
- ✅ 余额验证
- ✅ 二次确认
- ✅ 重要提示和警告

### 用户体验
- ✅ 深色主题UI
- ✅ 热门币种推荐
- ✅ 搜索功能
- ✅ 地址管理（保存常用地址）
- ✅ 二维码扫描
- ✅ 一键复制地址
- ✅ 实时费用计算

---

## 技术栈

### 前端框架
- Flutter (跨平台移动应用)
- Dart 语言

### 关键依赖
- **状态管理**: Provider / Riverpod / BLoC
- **导航**: go_router / auto_route
- **网络请求**: dio / http
- **二维码**: qr_flutter / qr_code_scanner
- **国际化**: flutter_localizations / easy_localization
- **本地存储**: shared_preferences / hive / sqflite
- **生物识别**: local_auth

### 后端服务
- **价格数据**: CoinGecko API / CoinMarketCap API
- **用户认证**: 自建后端服务
- **交易服务**: 自建后端服务
- **通知服务**: Firebase Cloud Messaging

---

## 使用说明

1. **开发人员**: 参考各页面的功能需求和技术考虑进行开发
2. **测试人员**: 使用验收标准汇总作为测试用例基础
3. **产品经理**: 参考用户交互章节验证产品逻辑
4. **设计师**: 对照UI组件章节检查设计实现

---

## 相关文档

- [去中心化钱包SRDs](../去中心化钱包_SRDs/) - 非托管钱包需求文档（35个页面）

---

## 更新记录

| 日期 | 版本 | 描述 |
|------|------|------|
| 2025-12-08 | 1.0 | 初始版本，包含44个页面的SRD文档 |
| 2025-12-08 | 1.1 | 新增平台账户绑定(CW-045)和GA绑定(CW-046)，共46个页面 |
| 2025-12-09 | 1.2 | 新增闪兑模块(CW-047至CW-051)，更新CW-014审核流程，共51个页面 |

---
