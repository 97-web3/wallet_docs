# 去中心化钱包 - 软件需求规格说明书索引

## 概述

本目录包含去中心化钱包（Web3 钱包）应用所有界面的软件需求规格说明书（SRD）。每个文档详细描述了对应界面的功能需求、UI组件、用户交互、数据需求、验证规则、技术考虑和验收标准。

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
| 项目名称 | 去中心化钱包（Web3 钱包） |
| 文档版本 | 1.2 |
| 创建日期 | 2025-12-08 |
| 最后更新 | 2025-12-09 |
| 总页面数 | 35 |
| 文档语言 | 中文 |

### V1.0 支持的区块链网络

| 网络 | 代币标准 |
|------|----------|
| TRX (Tron) | TRC-20 |
| BNB (BSC) | BEP-20 |
| ETH (Ethereum) | ERC-20 |

### 助记词规格
- 仅支持 **12 位**助记词 (BIP39 标准)

---

## 文档列表

### 一、钱包核心与首页（5个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 01 | [01-home-screen-requirements.md](./01-home-screen-requirements.md) | 首页/钱包主页 | 展示资产总览、快捷操作和代币列表 |
| 02 | [02-web3-wallet-page-requirements.md](./02-web3-wallet-page-requirements.md) | Web3钱包欢迎页 | 首次启动引导,创建或导入钱包入口 |
| 03 | [03-bottom-navigation-requirements.md](./03-bottom-navigation-requirements.md) | 底部导航栏 | 三大核心模块导航(资产/Web3钱包/我的) |
| 04 | [04-token-details-requirements.md](./04-token-details-requirements.md) | 代币详情页 | 单个代币的详细信息和价格走势 |
| 05 | [05-recent-transactions-requirements.md](./05-recent-transactions-requirements.md) | 最近交易记录 | 按时间展示所有交易历史 |

### 二、接收代币流程（2个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 06 | [06-receive-token-1-requirements.md](./06-receive-token-1-requirements.md) | 接收代币选择链页 | 选择要接收代币的区块链网络 |
| 07 | [07-receive-token-2-requirements.md](./07-receive-token-2-requirements.md) | 接收代币二维码页 | 显示二维码和完整接收地址 |

### 三、发送代币流程（6个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 08 | [08-send-token-select-requirements.md](./08-send-token-select-requirements.md) | 发送代币选择页 | 选择要发送的代币类型 |
| 09 | [09-send-token-1-requirements.md](./09-send-token-1-requirements.md) | 输入收款地址页 | 输入或扫描收款人地址 |
| 10 | [10-send-token-2-requirements.md](./10-send-token-2-requirements.md) | 输入金额页 | 输入发送数量和Gas费 |
| 11 | [11-send-token-3-requirements.md](./11-send-token-3-requirements.md) | 交易确认页 | 最终确认交易详情 |
| 12 | [12-send-token-4-requirements.md](./12-send-token-4-requirements.md) | 发送成功页 | 交易成功提示和区块链浏览器链接 |
| 13 | [13-send-token-5-requirements.md](./13-send-token-5-requirements.md) | 交易处理/失败页 | 处理发送中和失败状态 |

### 四、设置功能（2个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 14 | [14-currency-settings-requirements.md](./14-currency-settings-requirements.md) | 货币设置页 | 选择法定货币显示单位 |
| 15 | [15-language-settings-requirements.md](./15-language-settings-requirements.md) | 显示语言设置页 | 选择应用界面语言 |

### 五、其他页面（2个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 16 | [16-misc-screen-requirements.md](./16-misc-screen-requirements.md) | 未知组件 | 占位文档,原型图为空白 |
| 17 | [17-creation-success-requirements.md](./17-creation-success-requirements.md) | 钱包创建成功页 | 创建/导入成功确认页 |

### 六、钱包导入流程（2个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 18 | [18-import-wallet-mnemonic-requirements.md](./18-import-wallet-mnemonic-requirements.md) | 导入钱包-助记词 | 通过助记词导入现有钱包 |
| 19 | [19-import-wallet-private-key-requirements.md](./19-import-wallet-private-key-requirements.md) | 导入钱包-私钥 | 通过私钥导入现有钱包 |

### 七、助记词备份流程（4个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 20 | [20-generate-mnemonic-reminder-requirements.md](./20-generate-mnemonic-reminder-requirements.md) | 生成助记词提醒 | 创建钱包前的安全提示 |
| 21 | [21-backup-mnemonic-screenshot-warning-requirements.md](./21-backup-mnemonic-screenshot-warning-requirements.md) | 截图警告 | 禁止截屏助记词的安全警告 |
| 22 | [22-backup-mnemonic-1-requirements.md](./22-backup-mnemonic-1-requirements.md) | 备份助记词-步骤1 | 显示助记词让用户抄写 |
| 23 | [23-backup-mnemonic-2-requirements.md](./23-backup-mnemonic-2-requirements.md) | 备份助记词-步骤2 | 确认备份完成 |

### 八、助记词验证流程（2个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 24 | [24-verify-mnemonic-1-requirements.md](./24-verify-mnemonic-1-requirements.md) | 验证助记词 | 随机抽查3个单词输入验证 |
| 25 | [25-verify-mnemonic-status-requirements.md](./25-verify-mnemonic-status-requirements.md) | 验证状态 | 显示验证成功或失败 |

### 九、恢复短语查看（4个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 26 | [26-your-recovery-phrase-requirements.md](./26-your-recovery-phrase-requirements.md) | 恢复短语入口 | 查看恢复短语的入口页面 |
| 27 | [27-show-recovery-phrase-requirements.md](./27-show-recovery-phrase-requirements.md) | 显示恢复短语 | 验证后显示完整恢复短语 |
| 28 | [28-show-recovery-phrase-1-requirements.md](./28-show-recovery-phrase-1-requirements.md) | 显示恢复短语-变体 | 恢复短语显示的另一种布局 |
| 29 | [29-faceid-verification-requirements.md](./29-faceid-verification-requirements.md) | FaceID验证 | 生物识别验证 |

### 十、设置与账户管理（6个页面）

| 序号 | 文档 | 页面名称 | 描述 |
|------|------|----------|------|
| 30 | [30-settings-requirements.md](./30-settings-requirements.md) | 设置页面 | 应用设置主界面 |
| 31 | [31-add-account-requirements.md](./31-add-account-requirements.md) | 添加账户 | 添加新的钱包账户 |
| 32 | [32-remove-account-requirements.md](./32-remove-account-requirements.md) | 移除账户 | 删除现有钱包账户 |
| 33 | [33-manage-profile-requirements.md](./33-manage-profile-requirements.md) | 管理个人资料 | 用户信息管理 |
| 34 | [34-edit-bio-requirements.md](./34-edit-bio-requirements.md) | 编辑简介 | 修改个人简介 |
| 35 | [35-edit-username-requirements.md](./35-edit-username-requirements.md) | 编辑用户名 | 修改用户名 |

---

## 业务流程图

### 创建钱包流程
```
欢迎页 → 安全提醒 → 显示助记词 → 确认备份 → 验证助记词 → 创建成功 → 主页
```

### 导入钱包流程
```
欢迎页 → 选择导入方式 → 输入助记词/私钥 → 导入成功 → 主页
```

### 发送代币流程
```
主页 → 选择代币 → 输入地址 → 输入金额 → 确认交易 → 发送成功/失败
```

### 接收代币流程
```
主页 → 选择链网络 → 显示二维码和地址 → 复制/分享地址
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
| 钱包核心与首页 | 5 |
| 接收代币流程 | 2 |
| 发送代币流程 | 6 |
| 设置功能 | 2 |
| 其他页面 | 2 |
| 钱包导入流程 | 2 |
| 助记词备份流程 | 4 |
| 助记词验证流程 | 2 |
| 恢复短语查看 | 4 |
| 设置与账户管理 | 6 |
| **总计** | **35** |

---

## 核心功能特性

### 去中心化钱包特点
- ✅ 非托管模式，用户自持私钥
- ✅ 12 位助记词导入导出（BIP39 标准）
- ✅ V1.0 支持 TRX/BNB/ETH 三条链
- ✅ 链上直接交易
- ✅ 完整的交易记录追踪（已发起→交易中→已完成）

### 安全机制
- ✅ 替代验证模式（FaceID/2FA/资金密码任选其一）
- ✅ FaceID 生物识别验证（优先级 A）
- ✅ 助记词安全存储（本地加密）
- ✅ 截图保护（敏感页面）
- ✅ 地址格式验证
- ✅ ETH/BNB 地址二次网络确认
- ✅ 交易二次确认
- ✅ 重要安全提示和警告

### 用户体验
- ✅ 深色主题UI
- ✅ 多币种展示
- ✅ 搜索功能
- ✅ 二维码扫描和生成
- ✅ 一键复制地址
- ✅ 实时Gas费估算

---

## 技术栈

### 前端框架
- Flutter (跨平台移动应用)
- Dart 语言

### 区块链集成
- **Ethereum/EVM链**: web3dart
- **Solana**: solana (Dart package)
- **Bitcoin**: bitcoin_flutter
- **多链管理**: WalletConnect

### 关键依赖
- **状态管理**: Provider / Riverpod / BLoC
- **导航**: go_router / auto_route
- **图表**: fl_chart / syncfusion_flutter_charts
- **二维码**: qr_flutter / qr_code_scanner
- **国际化**: flutter_localizations / easy_localization
- **加密**: pointycastle, bip39, web3dart

### 后端服务
- **价格数据**: CoinGecko API / CoinMarketCap API
- **区块链节点**: Infura / Alchemy / QuickNode
- **Gas估算**: Etherscan API / BlockNative
- **交易索引**: The Graph / Covalent

---

## 安全考虑

### 核心安全要求
1. **私钥管理**:
   - 使用安全存储(Keychain/Keystore)
   - 永不上传到服务器
   - 加密存储助记词

2. **交易安全**:
   - 多重签名确认
   - 地址验证(Checksum)
   - Gas费合理性检查
   - 防重放攻击

3. **应用安全**:
   - 生物识别/PIN码锁定
   - 防截屏保护(敏感页面)
   - SSL Pinning
   - 代码混淆

4. **数据安全**:
   - 本地数据加密
   - 安全的剪贴板使用
   - 日志脱敏

---

## 开发优先级

### Phase 1 - MVP核心功能
- 钱包创建/导入 (文档02, 17-25)
- 资产展示 (文档01)
- 发送代币 (文档08-13)
- 接收代币 (文档06-07)

### Phase 2 - 增强功能
- 交易历史 (文档05)
- 代币详情 (文档04)
- 底部导航 (文档03)
- 恢复短语查看 (文档26-29)

### Phase 3 - 设置和优化
- 货币设置 (文档14)
- 语言设置 (文档15)
- 账户管理 (文档30-35)
- 性能优化
- 用户体验改进

---

## 测试要求

### 单元测试
- 密码学功能
- 地址验证
- 金额计算
- 业务逻辑

### 集成测试
- API集成
- 区块链交互
- 钱包操作流程
- 数据持久化

### E2E测试
- 完整用户流程
- 钱包创建到交易
- 多链操作
- 错误场景

### 安全测试
- 渗透测试
- 私钥安全审计
- 智能合约交互安全
- 第三方依赖审计

---

## 性能指标

### 响应时间要求
- 页面加载: < 2秒
- 交互响应: < 300毫秒
- 区块链查询: < 3秒
- 交易广播: < 5秒

### 稳定性要求
- 崩溃率: < 0.1%
- ANR率: < 0.1%
- 99%可用性

---

## 使用说明

1. **开发人员**: 参考各页面的功能需求和技术考虑进行开发
2. **测试人员**: 使用验收标准汇总作为测试用例基础
3. **产品经理**: 参考用户交互章节验证产品逻辑
4. **设计师**: 对照UI组件章节检查设计实现

---

## 相关文档

- [中心化钱包SRDs](../中心化钱包_SRDs/) - 托管钱包需求文档（51个页面）

---

## 更新记录

| 日期 | 版本 | 描述 |
|------|------|------|
| 2025-12-08 | 1.0 | 初始版本，包含35个页面的SRD文档 |
| 2025-12-09 | 1.2 | 更新助记词验证(24/25)改为抽查3词，恢复扫码导入功能(18) |

---
