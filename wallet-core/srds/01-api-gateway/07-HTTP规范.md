# HTTP 规范

## F-GW-016: 统一 HTTP 状态码

**描述**: 所有 API 响应使用统一的 HTTP 状态码规范。

### 成功响应

| 状态码 | 说明 | 使用场景 |
|--------|------|----------|
| 200 | OK | GET 请求成功、PUT/PATCH 更新成功 |
| 201 | Created | POST 创建资源成功 |
| 204 | No Content | DELETE 删除成功，无返回内容 |

### 客户端错误

| 状态码 | 说明 | 使用场景 |
|--------|------|----------|
| 400 | Bad Request | 请求参数格式错误、JSON 解析失败 |
| 401 | Unauthorized | 未认证、Token 无效或已过期 |
| 403 | Forbidden | 已认证但无权限、账户在黑名单 |
| 404 | Not Found | 资源不存在 |
| 409 | Conflict | 资源冲突 (如重复创建、已绑定等) |
| 422 | Unprocessable Entity | 业务规则校验失败 (如余额不足、超过限额) |
| 429 | Too Many Requests | 触发限流 |

### 服务端错误

| 状态码 | 说明 | 使用场景 |
|--------|------|----------|
| 500 | Internal Server Error | 服务端未知错误 |
| 502 | Bad Gateway | 上游 gRPC 服务不可用 |
| 503 | Service Unavailable | 服务维护中 |
| 504 | Gateway Timeout | 上游服务超时 |

---

## F-GW-018: 分页参数规范

**描述**: 所有列表接口使用统一的分页参数和响应结构。

**请求参数**:

| 参数 | 类型 | 必填 | 说明 | 默认值 |
|------|------|------|------|--------|
| page | int | 否 | 页码 (从 1 开始) | 1 |
| page_size | int | 否 | 每页数量 (1-100) | 20 |
| sort_by | string | 否 | 排序字段 | created_at |
| sort_order | string | 否 | 排序方向 (asc/desc) | desc |

**请求示例**:
```
GET /api/v1/transactions?page=1&page_size=20&sort_by=created_at&sort_order=desc
```

**响应结构**:
```json
{
  "success": true,
  "data": [
    { "id": 1, "..." },
    { "id": 2, "..." }
  ],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 100,
    "total_pages": 5,
    "has_next": true,
    "has_prev": false
  }
}
```

**分页字段说明**:

| 字段 | 类型 | 说明 |
|------|------|------|
| page | int | 当前页码 |
| page_size | int | 每页数量 |
| total | int | 总记录数 |
| total_pages | int | 总页数 |
| has_next | bool | 是否有下一页 |
| has_prev | bool | 是否有上一页 |

---

## F-GW-019: 错误响应 Schema

**描述**: 标准化的错误响应格式，包含详细的错误信息。

**标准错误响应结构**:
```json
{
  "success": false,
  "code": 10001,
  "message": "参数错误",
  "details": {
    "field": "amount",
    "reason": "金额必须大于 0",
    "value": "-100"
  },
  "request_id": "req-20251210-abc123",
  "timestamp": "2025-12-10T10:00:00Z"
}
```

**字段说明**:

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| success | bool | 是 | 始终为 false |
| code | int | 是 | 业务错误码 |
| message | string | 是 | 错误描述 (可展示给用户) |
| details | object/array | 否 | 详细错误信息 |
| request_id | string | 是 | 请求追踪 ID |
| timestamp | string | 是 | 错误发生时间 (ISO 8601) |

**多字段校验错误**:
```json
{
  "success": false,
  "code": 10001,
  "message": "参数校验失败",
  "details": [
    {"field": "amount", "reason": "金额必须大于 0"},
    {"field": "to_address", "reason": "地址格式不正确"}
  ],
  "request_id": "req-20251210-abc123",
  "timestamp": "2025-12-10T10:00:00Z"
}
```

**业务错误响应示例**:
```json
{
  "success": false,
  "code": 30001,
  "message": "余额不足",
  "details": {
    "available_balance": "50.00",
    "required_amount": "100.00",
    "asset": "USDT"
  },
  "request_id": "req-20251210-abc123",
  "timestamp": "2025-12-10T10:00:00Z"
}
```
