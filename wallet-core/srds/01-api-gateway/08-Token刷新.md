# Token 刷新机制

## F-GW-020: Token 刷新

**描述**: Access Token 过期后使用 Refresh Token 刷新，支持自动轮换。

### Token 类型

| 类型 | 有效期 | 用途 |
|------|--------|------|
| Access Token | 2 小时 (7200 秒) | API 请求认证 |
| Refresh Token | 7 天 (604800 秒) | 刷新 Access Token |

### 刷新端点

`POST /api/v1/auth/refresh-token`

**请求 Schema**:
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

**响应 Schema**:
```json
{
  "success": true,
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "expires_in": 7200,
  "token_type": "Bearer"
}
```

### 刷新规则

| 规则 | 说明 |
|------|------|
| 主动刷新 | Access Token 过期前 10 分钟可主动刷新 |
| 被动刷新 | Access Token 过期后，使用 Refresh Token 刷新 |
| Token 轮换 | Refresh Token 每次使用后自动轮换，旧 Token 失效 |
| 单次使用 | 同一 Refresh Token 只能使用一次，重复使用将使整个会话失效 |
| 过期处理 | Refresh Token 过期需重新登录 |

### Token 失效场景

| 场景 | Access Token | Refresh Token | 处理方式 |
|------|--------------|---------------|----------|
| 正常过期 | 失效 | 有效 | 使用 Refresh Token 刷新 |
| 用户登出 | 失效 | 失效 | 需重新登录 |
| 密码修改 | 失效 | 失效 | 需重新登录 |
| 设备被移除 | 失效 | 失效 | 需重新登录 |
| 账户被拉黑 | 失效 | 失效 | 无法登录 |
| 异地登录 | 失效 | 失效 | 需重新登录 (如开启异地登录保护) |

### 错误响应

| 错误码 | HTTP 状态 | 说明 |
|--------|-----------|------|
| 1002 | 401 | Refresh Token 已过期 |
| 1003 | 401 | Refresh Token 无效 |
| 1005 | 403 | 账户已被禁用 |
