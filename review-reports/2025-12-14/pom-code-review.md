# pom 开发者代码审查报告（2025-12-14）

## 一、审查范围

本次审查基于 wallet-admin-api 仓库 main 分支，重点关注提交：

- 提交号：`2e174ef`
- 提交说明：`1.币种管理 2.后台基础管理参考srd优化`

主要涉及模块：

- 币种管理：`internal/api/currency.go`、`internal/service/currency_service.go`、`internal/dto/currency.go`、`internal/dao/currency.go`、`internal/router/router.go`、`internal/ecode/*` 等
- 登录 / Token 与登录安全：`common/redis/login.go`、`internal/service/auth_service.go`、`internal/api/login.go`
- 管理员管理等基础模块：`internal/api/admin.go`、`internal/service/admin_service.go` 等

---

## 二、币种管理模块实现情况

### 2.1 结构设计

- 分层清晰：API / Service / DTO / DAO 划分明确，路由通过 `CurrencyRouterGroup` 进行集中注册，可维护性较好。
- 数据模型：
  - `dao.Currency` 将 `features`、`limits`、`fees` 以 JSON 文本持久化，并在 DAO 层提供 `Get*/Set*` 方法封装序列化/反序列化，接口层直接使用结构体，设计合理。
  - 状态字段：`Status` 采用 `active/inactive` 字符串，与业务含义直观匹配。

### 2.2 接口与业务逻辑

1. 列表 & 详情
   - `CurrencyListReq` 支持分页、network/status/type/keyword 筛选，对应 `dao.CurrencyListParams`，分页查询通过通用 `QueryPageWithDB` 实现，逻辑清晰。
   - `GetCurrencyList` 将 DAO 模型转换为 `CurrencyItem`，前后端字段一致性较好。
   - `GetCurrencyDetail` 基于 ID 查询并返回 `CurrencyDetailResp`，封装合理。

2. 创建币种
   - `CreateCurrency` 中对同一 network + symbol 做存在性校验（`ExistsBySymbolAndNetwork`），可以有效避免重复创建。
   - 对 `type=token` 且 `contract_address` 为空时返回 `ErrInvalidContract`，基本满足 token 类型约束。
   - 新增币种默认状态设为 `inactive`，并记录 `CreatedBy/UpdatedBy`，符合上线流程的预期。

3. 更新币种
   - `UpdateCurrency` 允许修改 `name/icon_url/limits/fees`，使用指针类型避免整体覆盖，增量更新逻辑合理。
   - Service 层返回值只区分 `ErrCurrencyNotFound` 与其他错误，在 API 层映射到统一错误码，行为一致。

4. 删除币种
   - `DeleteCurrency`：
     - 如果为 `native` 且未设置 `force`，直接返回 `ErrCurrencyIsNative`，阻止误删链原生币，保护较好。
     - 当前实现未对“币种有余额”的情况做检查，仅在错误码中预留了 `CodeCurrencyHasBalance`，存在业务保护缺失风险（详见改进建议）。

5. 状态切换 & 功能开关
   - `ToggleCurrencyStatus` 只负责更新 `Status`，并记录 `UpdatedBy`，实现简单直接。
   - `UpdateCurrencyFeatures` 支持 `deposit_enabled/withdraw_enabled/swap_enabled` 的局部更新，与前端开关型配置契合。

### 2.3 代码质量与可维护性

**优点：**

- 代码风格整体统一，命名清晰、职责边界明确。
- 错误码使用集中在 `ecode` 包，Service 层返回业务错误，API 层做翻译，便于后续国际化或文案调整。
- DTO/DAO 分离良好，未将 ORM 结构直接暴露给外部接口。

**改进建议：**

1. 删除币种的安全校验
   - 问题：当前 `DeleteCurrency` 仅校验 `native` 类型，未校验是否存在用户余额或账本余额，容易导致误删仍在使用中的币种。
   - 建议：
     - 在 Service 层增加余额或引用检查，例如查询业务 wallet/账本 表中是否存在该币种的余额记录；
     - 若存在余额，则返回 `CodeCurrencyHasBalance`，并要求手动迁移/结算后再删除或走“逻辑下架”而非物理删除。

2. 价格与总余额字段
   - 当前 `PriceUSD`、`TotalBalance` 由 DAO 字段直接暴露，但 `GetCurrencySummary` 中 `TotalBalanceUSD` 直接写死为 `"0.00"`，易与真实数据不一致。
   - 建议：
     - 若暂未对接行情/余额统计，可在注释中明确“暂未实现统计聚合，前端忽略字段”；
     - 或在汇总接口中显式标注 TODO，避免误解为实现完备。

3. 时间格式硬编码
   - `toCurrencyItem` 中使用固定格式 `"2006-01-02T15:04:05Z"`，与系统其它接口可能存在差异。
   - 建议：抽出到 `consts.TimeFormatISO` 之类的常量，统一时间序列化格式。

---

## 三、登录与安全相关改动

### 3.1 Redis 中的 Token 与登录尝试控制（common/redis/login.go）

**实现亮点：**

- 使用 `admin_login_token:{adminId}` 作为 key 存储单点登录 token，并在 `ValidateToken` 时通过 Redis 校验，有效支持“服务端强制下线”等场景。
- 登录失败计数与锁定：
  - `LoginAttempt` 结合 `failKey` 与 `lockKey` 区分失败次数与锁定状态；
  - 超过阈值后自动写入锁定 key，并设置过期时间，实现简单的防暴力破解。
- `NeedCaptcha` 逻辑解耦到 Redis 工具层，Service 层只关心是否需要验证码，职责分离较好。

**风险与建议：**

1. JWT 秘钥硬编码
   - 当前 `SigningKey` 为常量字符串写死在代码中，不利于生产环境安全管理。
   - 建议：改为读取配置（如 `config.C.Auth.SigningKey`），并通过环境变量/配置中心注入。

2. Redis 连接错误静默处理
   - 多处 `if rs == nil { return }` 或忽略 `Do` 返回错误，会导致在 Redis 不可用时静默失败，登录逻辑变为“无状态 + 无锁定”。
   - 建议：
     - 至少在 Service 层对关键路径增加日志记录，便于排查依赖故障；
     - 或在 Redis 失败时给出降级策略（例如禁止登录）。

### 3.2 登录服务层（internal/service/auth_service.go）

**优点：**

- 登录流程较完整：密码校验、失败计数、验证码校验、账户锁定、Token 生成与持久化、权限列表下发均已串联好。
- reCAPTCHA 校验对开发环境做了特殊处理（未配置 secret 且 debug 时跳过），提升了本地调试体验。
- 刷新 Token 时通过剩余有效期判断是否允许刷新，并校验 Redis 中旧 token 的有效性，防止非法刷新。

**改进建议：**

1. 错误信息区分
   - 当前对 `auth.ParseToken` 的错误基本直接透传或转换为 `ErrNotFound`，无法区分“过期”“签名错误”“格式错误”。
   - 建议：在 `ParseToken` 层归类错误类型，便于安全审计与日志分析。

2. 密码生成函数
   - `generateRandomPassword` 目前使用固定 charset + 按序取模的方式构造密码，随机性不足。
   - 建议：使用 `crypto/rand` 生成真随机索引，避免模式化密码。

---

## 四、后台基础管理相关改动（节选）

### 4.1 管理员相关接口（internal/api/admin.go & internal/service/admin_service.go）

**优点：**

- 列表、增删改、禁用/启用、重置密码、权限获取等接口拆分清晰，对应 Service 的方法命名规范。
- 禁用管理员时：
  - 防止禁用自己（`ErrCannotDisableSelf`）；
  - 检查是否为最后一个超级管理员，避免误操作锁死系统（`ErrLastSuperAdmin`）；
  - 禁用成功后清理 Redis 中 token，确保立刻失效。

**建议：**

- 审计与操作日志：从 AdminActionLog 相关中间件来看已有记录机制，建议确保所有敏感操作（禁用、重置密码、角色变更）均写入统一审计流水，并在文档中标明。

### 4.2 登录接口（internal/api/login.go）

**优点：**

- 根据不同错误类型（账号锁定、验证码错误、密码错误）给出差异化响应，并在需要验证码场景下通过 `NeedCaptcha` 字段指导前端行为，交互较友好。
- 使用 `notifylogger.ErrorWithRobotNotice` 对异常登录失败进行告警，提升运维可观测性。

**建议：**

- 对返回 `Message` 文本的地方（例如“账号或密码错误”）后续可统一走多语言文案配置，避免硬编码。

---

## 五、整体评价

- **完成度**：本次提交中，币种管理模块的基础 CRUD、状态切换与功能开关已完整实现，能支撑后台日常配置与管理需求；登录与管理员管理模块也进行了较系统的安全与体验优化。
- **代码质量**：整体结构清晰、命名规范，错误码与业务错误分离良好，可维护性较好。
- **主要风险点**：
  1. 币种删除未做余额关联检查，存在误删仍在使用币种的可能；
  2. JWT 签名秘钥硬编码在代码中；
  3. Redis 依赖异常时缺乏显式告警与降级策略。

---

## 六、后续建议（供 PO/开发参考）

1. **补充币种风险控制**：
   - 在删除/下架流程中增加余额检查与业务确认；
   - 对“强制删除”行为增加二次确认与审计记录。

2. **配置与安全治理**：
   - 将 JWT 秘钥、Google reCAPTCHA Secret 等敏感信息统一迁移到配置中心/环境变量；
   - 对 Redis 关键操作增加错误日志或指标上报。

3. **文档与前端协同**：
   - 将币种管理接口的字段与行为在团队内部 Wiki 中固化，约定好哪些字段暂时是占位（如统计类），避免前端误用。

> 本报告仅基于当前仓库 main 分支与提交 2e174ef 的代码快照进行静态审查，未包含运行期验证和联调结果。