# Blockchain RPC Proxy API 文档

## 概述

本服务为 ETH、BSC、TRON 三个区块链节点提供统一的 HTTP RPC 代理入口，支持 API Key 身份认证。

## 服务信息

| 项目 | 值 |
|------|-----|
| 代理地址 | `http://YOUR_SERVER` |
| 认证方式 | API Key (Header) |
| 认证头 | `X-API-Key` |

## API Key

```

```

> 请妥善保管 API Key，如需更换请联系管理员。

---

## 端点列表

| 端点 | 链 | API 类型 | 说明 |
|------|-----|----------|------|
| `/health` | - | REST | 健康检查（无需认证） |
| `/eth` | Ethereum | JSON-RPC | 以太坊主网 |
| `/bsc` | BNB Chain | JSON-RPC | BSC 主网 |
| `/tron/*` | TRON | REST | 波场主网 |

---

## 健康检查

**无需认证**

```bash
curl http://YOUR_SERVER/health
```

**响应示例：**
```json
{"status":"ok","timestamp":"2025-12-11T03:23:41+00:00"}
```

---

## Ethereum (ETH)

### 端点
```
POST /eth
```

### 请求格式
JSON-RPC 2.0

### 示例请求

#### 获取最新区块号
```bash
curl -X POST http://YOUR_SERVER/eth \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_blockNumber",
    "params": [],
    "id": 1
  }'
```

**响应：**
```json
{"jsonrpc":"2.0","id":1,"result":"0x16e022e"}
```

#### 获取账户余额
```bash
curl -X POST http://YOUR_SERVER/eth \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_getBalance",
    "params": ["0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb2", "latest"],
    "id": 1
  }'
```

#### 获取交易详情
```bash
curl -X POST http://YOUR_SERVER/eth \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_getTransactionByHash",
    "params": ["0x交易哈希"],
    "id": 1
  }'
```

#### 获取区块详情
```bash
curl -X POST http://YOUR_SERVER/eth \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_getBlockByNumber",
    "params": ["latest", true],
    "id": 1
  }'
```

#### 获取 Gas 价格
```bash
curl -X POST http://YOUR_SERVER/eth \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_gasPrice",
    "params": [],
    "id": 1
  }'
```

### 支持的 RPC 方法

| 方法 | 说明 |
|------|------|
| `eth_blockNumber` | 获取最新区块号 |
| `eth_getBalance` | 获取账户余额 |
| `eth_getTransactionByHash` | 通过哈希获取交易 |
| `eth_getTransactionReceipt` | 获取交易回执 |
| `eth_getBlockByNumber` | 通过区块号获取区块 |
| `eth_getBlockByHash` | 通过哈希获取区块 |
| `eth_gasPrice` | 获取当前 Gas 价格 |
| `eth_chainId` | 获取链 ID |
| `eth_call` | 执行合约调用 |
| `eth_estimateGas` | 估算 Gas |
| `eth_getLogs` | 获取日志 |
| `eth_getCode` | 获取合约代码 |
| `eth_getStorageAt` | 获取存储数据 |
| `net_version` | 获取网络版本 |
| `web3_clientVersion` | 获取客户端版本 |

---

## BNB Smart Chain (BSC)

### 端点
```
POST /bsc
```

### 请求格式
JSON-RPC 2.0（与 ETH 相同）

### 示例请求

#### 获取链 ID
```bash
curl -X POST http://YOUR_SERVER/bsc \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_chainId",
    "params": [],
    "id": 1
  }'
```

**响应：**
```json
{"jsonrpc":"2.0","id":1,"result":"0x38"}
```

> 0x38 = 56，即 BSC 主网链 ID

#### 获取最新区块号
```bash
curl -X POST http://YOUR_SERVER/bsc \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_blockNumber",
    "params": [],
    "id": 1
  }'
```

#### 获取 BNB 余额
```bash
curl -X POST http://YOUR_SERVER/bsc \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "eth_getBalance",
    "params": ["0x地址", "latest"],
    "id": 1
  }'
```

### 支持的 RPC 方法

与 ETH 相同，BSC 兼容以太坊 JSON-RPC API。

---

## TRON

### 端点
```
/tron/*
```

### 请求格式
REST API

### 示例请求

#### 获取最新区块
```bash
curl http://YOUR_SERVER/tron/wallet/getnowblock \
  -H "X-API-Key: your_api_key_here"
```

**响应示例：**
```json
{
  "blockID": "...",
  "block_header": {
    "raw_data": {
      "number": 78206751,
      "timestamp": 1733888888000,
      ...
    }
  }
}
```

#### 获取账户信息
```bash
curl -X POST http://YOUR_SERVER/tron/wallet/getaccount \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "address": "TRX地址Base58格式"
  }'
```

#### 通过区块号获取区块
```bash
curl -X POST http://YOUR_SERVER/tron/wallet/getblockbynum \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "num": 78206751
  }'
```

#### 通过哈希获取交易
```bash
curl -X POST http://YOUR_SERVER/tron/wallet/gettransactionbyid \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "value": "交易ID"
  }'
```

#### 获取交易回执
```bash
curl -X POST http://YOUR_SERVER/tron/wallet/gettransactioninfobyid \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "value": "交易ID"
  }'
```

#### 获取节点信息
```bash
curl http://YOUR_SERVER/tron/wallet/getnodeinfo \
  -H "X-API-Key: your_api_key_here"
```

#### 查询 TRC20 代币余额
```bash
curl -X POST http://YOUR_SERVER/tron/wallet/triggerconstantcontract \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "owner_address": "持有者地址Hex格式",
    "contract_address": "合约地址Hex格式",
    "function_selector": "balanceOf(address)",
    "parameter": "0000000000000000000000000000000000000000000000000000000000000000"
  }'
```

### 常用 API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/wallet/getnowblock` | GET | 获取最新区块 |
| `/wallet/getblockbynum` | POST | 通过区块号获取区块 |
| `/wallet/getblockbyid` | POST | 通过区块哈希获取区块 |
| `/wallet/getaccount` | POST | 获取账户信息 |
| `/wallet/gettransactionbyid` | POST | 获取交易详情 |
| `/wallet/gettransactioninfobyid` | POST | 获取交易回执 |
| `/wallet/getnodeinfo` | GET | 获取节点信息 |
| `/wallet/getaccountbalance` | POST | 获取账户余额 |
| `/wallet/triggerconstantcontract` | POST | 调用合约只读方法 |
| `/wallet/getcontract` | POST | 获取合约信息 |

---

## 错误响应

### 认证错误 (401)
```json
{"error":"Invalid or missing API key","code":401}
```

**原因：** 缺少 `X-API-Key` 头或 API Key 无效

### 路径错误 (404)
```json
{"error":"Not found","endpoints":["/eth","/bsc","/tron/","/health"]}
```

**原因：** 请求了不存在的端点

### 速率限制 (503)
当请求过于频繁时，可能返回 503 错误。

**限制：** 每秒 500 请求 (per API Key)

---

