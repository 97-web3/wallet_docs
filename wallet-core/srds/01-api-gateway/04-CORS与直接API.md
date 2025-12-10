# CORS 跨域处理与直接 API

## F-GW-008: CORS 配置

**描述**: 处理跨域资源共享请求。

**配置**:
```yaml
Cors:
  AllowOrigins:
    - "*"
  AllowMethods:
    - "GET"
    - "POST"
    - "PUT"
    - "DELETE"
    - "PATCH"
    - "OPTIONS"
  AllowHeaders:
    - "Content-Type"
    - "Authorization"
    - "X-Request-ID"
    - "X-Trace-ID"
  ExposeHeaders:
    - "X-Request-ID"
    - "X-Trace-ID"
  AllowCredentials: true
  MaxAge: 86400
```

**响应头**:
```
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 86400
```

---

## F-GW-009: 验证码服务

**描述**: 图形验证码生成与验证，用于防止机器人攻击。

**API 端点**:

| 端点 | 方法 | 描述 |
|------|------|------|
| `/api/v1/captcha` | GET | 生成验证码 |
| `/api/v1/captcha/verify` | POST | 验证验证码 |

**生成验证码响应**:
```json
{
  "captcha_id": "abc123",
  "captcha_image": "data:image/png;base64,..."
}
```

**验证验证码请求**:
```json
{
  "captcha_id": "abc123",
  "captcha_code": "A1B2"
}
```

---

## F-GW-010: 健康检查

**描述**: 服务健康状态检查端点。

**端点**: `GET /health`

**响应**:
```json
{
  "status": "ok",
  "timestamp": "2025-12-10T10:00:00Z"
}
```
