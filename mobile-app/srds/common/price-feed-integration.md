# 实时汇率集成 - 公共规格文档

## 1. 文档标识

| 属性 | 值 |
|------|-----|
| 文档ID | COMMON-005 |
| 文档名称 | 实时汇率集成 |
| 版本 | 1.0 |
| 更新日期 | 2025-12-08 |
| 适用范围 | 中心化钱包、去中心化钱包 |

---

## 2. 概述

系统需要实时获取代币价格用于：
- 显示代币法币估值
- 计算总资产价值
- 验证最小金额 (10 USDT 等值)
- 闪兑汇率参考

---

## 3. 数据源

### 3.1 主数据源: Debank API

| 属性 | 值 |
|------|------|
| 提供商 | Debank |
| API 类型 | RESTful |
| 数据格式 | JSON |
| 认证方式 | API Key |

### 3.2 备用数据源

当主数据源不可用时，按以下顺序尝试备用源：
1. CoinGecko API
2. 本地缓存数据

---

## 4. 价格更新策略

### 4.1 更新频率

| 场景 | 更新频率 |
|------|----------|
| 首页资产列表 | 每 30 秒 |
| 交易确认页 | 实时获取 |
| 后台刷新 | 每 60 秒 |

### 4.2 缓存策略

```javascript
const PRICE_CACHE_CONFIG = {
  // 缓存有效期
  TTL: 30 * 1000, // 30 秒

  // 最大过期时间（网络异常时使用过期数据）
  MAX_STALE: 5 * 60 * 1000, // 5 分钟

  // 缓存存储
  storage: 'localStorage', // 或 'memory'
};
```

### 4.3 更新逻辑

```javascript
/**
 * 获取代币价格
 * @param {string} tokenSymbol - 代币符号
 * @param {string} network - 网络代码
 * @returns {Promise<number|null>} 价格 (USDT)
 */
async function getTokenPrice(tokenSymbol, network) {
  const cacheKey = `price_${tokenSymbol}_${network}`;
  const cached = getFromCache(cacheKey);

  // 检查缓存
  if (cached && !isExpired(cached, PRICE_CACHE_CONFIG.TTL)) {
    return cached.price;
  }

  try {
    // 从 API 获取最新价格
    const price = await fetchPriceFromAPI(tokenSymbol, network);

    // 更新缓存
    setCache(cacheKey, {
      price,
      timestamp: Date.now(),
    });

    return price;
  } catch (error) {
    // 网络异常，尝试使用过期缓存
    if (cached && !isExpired(cached, PRICE_CACHE_CONFIG.MAX_STALE)) {
      console.warn('Using stale price data');
      return cached.price;
    }

    // 无可用数据
    return null;
  }
}
```

---

## 5. 价格显示规则

### 5.1 正常状态

```
ETH     1.2345678     ≈ $2,469.13
        ▲ 代币数量     ▲ 法币估值
```

### 5.2 数据加载中

```
ETH     1.2345678     ≈ --
```

### 5.3 数据获取失败

```
ETH     1.2345678     ≈ --
```

显示 Toast 提示: "价格获取失败，请检查网络"

### 5.4 价格为零或无效

```
ETH     1.2345678     ≈ $0.00
```

---

## 6. 最小金额验证

### 6.1 规则

- 最小充值/提现/划转金额: **10 USDT 等值**
- 非 USDT 代币通过实时汇率换算

### 6.2 验证逻辑

```javascript
/**
 * 验证金额是否满足最小要求
 * @param {number} amount - 输入金额
 * @param {string} tokenSymbol - 代币符号
 * @param {string} network - 网络代码
 * @returns {Promise<Object>} 验证结果
 */
async function validateMinAmount(amount, tokenSymbol, network) {
  const MIN_USDT = 10;

  // USDT 直接比较
  if (tokenSymbol === 'USDT') {
    return {
      valid: amount >= MIN_USDT,
      usdtEquivalent: amount,
      error: amount < MIN_USDT ? `最小金额不能低于 ${MIN_USDT} USDT` : null,
    };
  }

  // 获取代币价格
  const price = await getTokenPrice(tokenSymbol, network);

  if (price === null) {
    return {
      valid: false,
      usdtEquivalent: null,
      error: '无法获取当前价格，请稍后重试',
    };
  }

  const usdtEquivalent = amount * price;

  return {
    valid: usdtEquivalent >= MIN_USDT,
    usdtEquivalent,
    error: usdtEquivalent < MIN_USDT
      ? `最小金额不能低于 ${MIN_USDT} USDT 等值（当前约 ${usdtEquivalent.toFixed(2)} USDT）`
      : null,
  };
}
```

### 6.3 UI 反馈

```
┌────────────────────────────────────────┐
│  输入金额                               │
│  ┌──────────────────────────────────┐  │
│  │ 0.001                         ETH│  │
│  └──────────────────────────────────┘  │
│  ≈ 2.47 USDT                          │
│  ⚠️ 最小金额不能低于 10 USDT 等值        │
└────────────────────────────────────────┘
```

---

## 7. 总资产计算

### 7.1 计算公式

```javascript
/**
 * 计算总资产价值
 * @param {Array} tokens - 代币列表 [{symbol, balance, network}]
 * @returns {Promise<Object>} 总资产信息
 */
async function calculateTotalAssets(tokens) {
  let totalUSDT = 0;
  let hasError = false;

  for (const token of tokens) {
    const price = await getTokenPrice(token.symbol, token.network);

    if (price === null) {
      hasError = true;
      continue;
    }

    totalUSDT += token.balance * price;
  }

  return {
    totalUSDT,
    displayValue: formatTokenValue(totalUSDT), // 参考 COMMON-001
    hasError,
    errorMessage: hasError ? '部分代币价格获取失败' : null,
  };
}
```

### 7.2 显示规则

| 状态 | 显示 |
|------|------|
| 正常 | `$12,345.67` |
| 部分失败 | `$12,345.67 *` (带星号提示) |
| 全部失败 | `--` |

---

## 8. API 错误处理

### 8.1 错误类型

| 错误码 | 说明 | 处理方式 |
|--------|------|----------|
| NETWORK_ERROR | 网络连接失败 | 使用缓存 + 提示 |
| RATE_LIMIT | 请求频率超限 | 延迟重试 |
| API_ERROR | API 返回错误 | 使用缓存 + 上报 |
| INVALID_TOKEN | 不支持的代币 | 显示 "--" |

### 8.2 用户提示

| 场景 | 提示 |
|------|------|
| 价格加载失败 | Toast: "价格获取失败，显示数据可能不准确" |
| 持续失败 (>5分钟) | Banner: "网络异常，部分数据无法更新" |
| 恢复正常 | 自动刷新，无提示 |

---

## 9. 验收标准

- [ ] 代币价格每 30 秒自动刷新
- [ ] 缓存机制正常工作
- [ ] 网络异常时使用缓存数据
- [ ] 最小金额验证使用实时汇率
- [ ] 总资产正确计算
- [ ] 价格获取失败时显示 "--"
- [ ] 错误状态有明确提示

---

*文档创建日期: 2025-12-08*
