# Enterprise Payroll Disbursement System - Module Analysis & Extension Plan

## Executive Summary

This document provides a comprehensive analysis of the Internal-Wallet system documented in `wallet-core/srds/README.md` and detailed recommendations for extending it to support enterprise cryptocurrency payroll disbursement operations across TRON, BSC, and Ethereum networks.

### User Requirements (Confirmed)

| Requirement | Selection | Implications |
|-------------|-----------|--------------|
| **Scale** | 1,000-5,000 employees | Existing architecture sufficient with optimizations; chunked batch processing |
| **Approval Model** | Multi-level by amount | Different approval chains based on payment amount thresholds |
| **Fee Priority** | Speed first | Prioritize fast confirmations, use FAST gas settings, accept higher fees |

---

# 1. Current Module Analysis

## 1.1 Module Inventory

Based on the documentation in `wallet-core/srds/README.md`, the system consists of **7 core modules**:

| # | Module Name | Chinese Name | Location | Functions |
|---|-------------|--------------|----------|-----------|
| 01 | API Gateway | API 网关 | `01-api-gateway/` | 20+ HTTP routes |
| 02 | Business Service | 业务服务 | `02-business-service/` | 48 RPC |
| 03 | ChainRPC Service | 链RPC服务 | `03-chainrpc-service/` | 28 RPC |
| 04 | ChainSync Service | 链同步服务 | `04-chainsync-service/` | 17 RPC |
| 05 | Signer Service | 签名服务 | `05-signer-service/` | 14 RPC |
| 06 | Common Infrastructure | 通用库与基础设施 | `06-common-infrastructure/` | Shared utilities |
| 07 | Admin Service | 后台管理服务 | `07-admin-service/` (documented in `admin-api/`) | 30+ RPC |

**Total: 107+ RPC functions across 22 database tables**

---

## 1.2 Module Responsibilities

### Module 01: API Gateway (`01-api-gateway/`)

**Primary Purpose:** Unified HTTP entry point that routes requests to microservices via gRPC

**Core Technical Responsibilities:**
- HTTP to gRPC protocol conversion
- JWT token authentication and validation
- Distributed rate limiting (global, IP-based, user-based)
- CORS handling for cross-origin requests
- Request logging and distributed tracing (Jaeger)
- Panic recovery and exception handling
- Blacklist checking (IP and user identifier)

**Key APIs/Interfaces:**
- 20+ HTTP routes under `/api/v1/*`
- Middleware chain: Recovery → Request ID → Logging → CORS → Rate Limit → Auth → Blacklist → Router

**Data Models:**
- No persistent storage (stateless)
- Redis for distributed rate limiting state

**Current Chain Support:** Chain-agnostic (routes to backend services)

---

### Module 02: Business Service (`02-business-service/`)

**Primary Purpose:** Core business logic for user management, authentication, assets, and transactions

**Core Technical Responsibilities:**
- User registration, authentication, session management
- Asset balance management and transaction history
- Deposit address generation and monitoring
- Withdrawal request creation and audit workflow
- Internal user-to-user transfers
- Security settings (2FA, device management, biometrics)
- Notification management (push, SMS, email, in-app)
- Token swap functionality (DEX integration)

**Key APIs/Interfaces:** 48 RPC functions across 12 sub-modules:
- `Register`, `Login`, `Logout`, `RefreshToken`, `ResetPassword`
- `GetUserOverview`, `GetPersonalInfo`, `UpdatePersonalInfo`
- `GetAssetOverview`, `GetTransactionRecords`, `GetBalanceChanges`
- `GetDepositInfo`, `CreateDepositAddress`
- `GetWithdrawInfo`, `CreateWithdrawRequest`
- `GetTransferInfo`, `CreateTransfer`
- Security, notification, preference, address book, login history, swap functions

**Data Models (12 tables):**
- `users` - User accounts with KYC level, 2FA status
- `user_sessions` - Session tokens with device tracking
- `verification_codes` - OTP codes for various purposes
- `login_logs` - Login history with risk assessment
- `user_roles` - RBAC (USER, FINANCE, ADMIN)
- `blacklists` - Blocked identifiers
- `platform_bindings` - Web3 wallet connections
- `withdrawal_audits` - Withdrawal approval workflow
- `trusted_devices` - Device fingerprinting
- `transfer_audits` - Internal transfer audit trail
- `payroll_records` - Salary/bonus tracking
- `swap_transactions` - Swap history

**Current Chain Support:** TRON, BSC, Ethereum (via ChainRPC/Signer services)

---

### Module 03: ChainRPC Service (`03-chainrpc-service/`)

**Primary Purpose:** Blockchain interaction layer for address generation, transaction building, and broadcasting

**Core Technical Responsibilities:**
- Multi-chain address generation and validation
- Native token transfer transaction building
- Token (TRC20/BEP20/ERC20) transfer transaction building
- Batch transfer construction
- Gas price fetching and estimation
- Transaction broadcasting, acceleration, and cancellation
- On-chain balance queries (native and token)
- Transaction status and receipt retrieval
- Block height and information queries
- RPC node health monitoring and failover

**Key APIs/Interfaces:** 28 RPC functions:
- Address: `GenerateAddress`, `GenerateAddressBatch`, `ValidateAddress`, `ConvertPubKeyToAddress`, `CheckInternalAddress`
- Transaction: `BuildNativeTransfer`, `BuildTokenTransfer`, `BuildBatchTransfer`
- Gas: `EstimateGas`, `GetGasPrice`
- Broadcast: `BroadcastTransaction`, `BroadcastBatch`, `SpeedUpTransaction`, `CancelTransaction`
- Query: `GetBalance`, `GetTokenBalance`, `GetMultiAddressBalance`
- Transaction Query: `GetTransactionDetails`, `GetTransactionStatus`, `GetTransactionReceipt`
- Block: `GetBlockHeight`, `GetBlockInfo`, `GetAddressNonce`
- Token/Node: `GetTokenInfo`, `GetNodeHealth`, `SwitchActiveNode`, `GetNodeStatus`, `GetSupportedTokens`, `GetChainConfig`

**Data Models:**
- No persistent tables (stateless queries)
- Caches gas prices and balances in Redis

**Current Chain Support:**
- TRON (TRX/TRC20) - Coin Type 195
- BSC (BNB/BEP20) - Coin Type 60, Account 0
- Ethereum (ETH/ERC20) - Coin Type 60, Account 1
- Polygon (MATIC/ERC20) - Available but disabled in V1

---

### Module 04: ChainSync Service (`04-chainsync-service/`)

**Primary Purpose:** Real-time blockchain synchronization, deposit monitoring, and balance tracking

**Core Technical Responsibilities:**
- Block synchronization state management (start/stop/reset)
- Address monitoring for deposit detection
- Transaction monitoring and confirmation tracking
- Real-time balance tracking across addresses
- RPC provider health monitoring and failover
- Kafka event publishing for deposit/balance events
- Blockchain reorganization (reorg) detection

**Key APIs/Interfaces:** 17 RPC functions:
- Sync: `StartSync`, `StopSync`, `GetSyncStatus`, `ResetSyncState`
- Address: `AddMonitoredAddress`, `RemoveMonitoredAddress`, `ListMonitoredAddresses`
- Transaction: `GetTransactionDetails`, `GetAddressTransactionHistory`, `GetBlockTransactions`
- Balance: `GetAddressBalance`, `GetMultiAddressBalances`
- Provider: `GetProviderStatus`, `SwitchProvider`, `TestRPCEndpoint`
- Stats: `GetSyncStatistics`, `GetPerformanceMetrics`

**Data Models (6 tables):**
- `blocks` - Synced block data with hash, timestamp
- `transactions` - Confirmed transactions with status
- `address_monitors` - Watched addresses with webhook callbacks
- `balance_records` - Address balance snapshots
- `sync_tasks` - Background sync job tracking
- `provider_status` - RPC health/performance metrics

**Kafka Event Topics:**
- `chainsync.deposit.detected` - Raw deposit detection
- `chainsync.deposit.confirmed` - Confirmed deposits
- `chainsync.balance.changed` - Balance updates
- `chainsync.block.synced` - Block sync completion
- `chainsync.tx.status` - Transaction status changes

**Confirmation Requirements:**
| Chain | Standard | Large (≥10k USDT) |
|-------|----------|-------------------|
| TRON | 20 | 30 |
| BSC | 15 | 25 |
| Ethereum | 12 | 20 |

**Current Chain Support:** TRON, BSC, Ethereum

---

### Module 05: Signer Service (`05-signer-service/`)

**Primary Purpose:** Core security module for HD wallet key management and transaction signing

**Core Technical Responsibilities:**
- BIP39 seed generation and mnemonic management
- BIP44 hierarchical deterministic address derivation
- AES-256-GCM seed encryption with PBKDF2 key derivation
- Multi-chain address generation from single seed
- Transaction signing (single and batch)
- Message signing for verification
- Public key export (without exposing private keys)
- Hot/cold wallet temperature separation
- Comprehensive signature audit logging

**Key APIs/Interfaces:** 14 RPC functions:
- Seed: `InitSeed`, `InitSeedFromMnemonic`, `ListSeeds`, `UpdateSeedStatus`
- Address: `GenerateUserDepositAddresses`, `DerivePath`, `GenerateMultiChainAddresses`
- Public Key: `ExportPublicKey`, `ExportPublicKeyBatch`
- Signing: `SignTransaction`, `SignBatch`, `SignMessage`
- Audit: `GetSignatureLogs`, `HealthCheck`

**Data Models (4 tables):**
- `master_seeds` - Encrypted seeds with temperature (hot=1/cold=2), daily limits
- `deposit_addresses` - Generated addresses with BIP44 paths per chain
- `signature_logs` - Complete audit trail of all signing operations
- `pub_key_export_logs` - Public key export audit

**BIP44 Derivation Paths:**
| Chain | Coin Type | Account | Path Format |
|-------|-----------|---------|-------------|
| TRON | 195 | 0 | m/44'/195'/0'/0/{user_id} |
| BSC | 60 | 0 | m/44'/60'/0'/0/{user_id} |
| Ethereum | 60 | 1 | m/44'/60'/1'/0/{user_id} |

**Cryptography:**
- Seed encryption: AES-256-GCM + PBKDF2 (100,000 iterations)
- Key derivation: secp256k1 elliptic curve
- Address generation: Keccak256 hash → EIP-55 checksum (ETH/BSC), Base58Check (TRON)

**Security Level:** CRITICAL - Should NOT be exposed via public API gateway

**Current Chain Support:** TRON, BSC, Ethereum

---

### Module 06: Common Infrastructure (`06-common-infrastructure/`)

**Primary Purpose:** Shared libraries, utilities, middleware, and infrastructure configuration

**Core Technical Responsibilities:**
- Middleware components (JWT auth, CORS, rate limiting, recovery, logging)
- Utility functions (crypto hashing, seed encryption, Snowflake ID generation, captcha)
- Base data models (BaseModel, BaseModelWithUser)
- Redis cache wrapper with distributed locks
- Business constants and Redis key patterns
- Error code registry
- Generic repository pattern (CRUD operations)
- GORM database connection and pooling
- Docker Compose infrastructure configuration

**Key Components:**
```
common/
├── cache/redis_cache.go
├── captcha/captcha.go
├── code/code.go
├── constants/{constants,redis,signer}.go
├── db/{config,gorm}.go
├── enum/accountType.go
├── errcode/signer.go
├── interceptor/{example_usage,metadata}.go
├── middleware/{auth,common,cors,logging,metadata,ratelimit,recovery,redis_ratelimit}.go
├── model/base_model.go
├── models/wallet.go
├── repository/base_repository.go
└── utils/{config_finder,crypto,crypto_signer,response,snowflake,tracing,validator}.go
```

**Infrastructure Components:**
- MariaDB 10.11 (primary database)
- Redis 7.x (caching, rate limiting, sessions)
- Kafka 7.5.0 (event streaming)
- ETCD 3.5.9 (service discovery)
- Prometheus + Grafana (monitoring)
- Jaeger (distributed tracing)

---

### Module 07: Admin Service (`admin-api/`)

**Primary Purpose:** Back-office administration for dashboard, user management, payroll, approvals, and RBAC

**Core Technical Responsibilities:**
- Dashboard statistics and vault balance monitoring
- User management (freeze, unfreeze, terminate, role assignment)
- Payroll batch import, processing, and reporting
- Withdrawal approval workflow (with AML integration)
- Swap configuration management
- Ledger records and vault adjustments
- Role-based access control (RBAC)

**Key APIs/Interfaces:** 30+ RPC functions across 8 sub-modules:
- Dashboard: `GetDashboardStatistics`, `GetVaultBalances`, `GetAlertThresholds`, `SetAlertThreshold`
- User: `AdminListUsers`, `AdminFreezeUser`, `AdminUnfreezeUser`, `AdminTerminateUser`, `AdminUpdateUserRole`
- Payroll: `BatchImportPayroll`, `ProcessPayrollBatch`, `GetPayrollHistory`, `ExportPayrollReport`
- Withdrawal: `AdminListPendingWithdraws`, `AdminApproveWithdraw`, `AdminRejectWithdraw`
- Swap: `GetSwapConfig`, `UpdateSwapConfig`
- Ledger: `GetLedgerRecords`, `ExportLedger`, `AdminAdjustVaultBalance`, `GetVaultAdjustmentHistory`, `ApproveVaultAdjustment`
- Role: `GetRoles`, `AssignRole`, `GetUserPermissions`

**Data Models:**
- `vault_balances` - Hot/cold wallet balances per chain/asset
- `vault_adjustments` - Manual balance adjustments with approval
- `payroll_batches` - Salary batch records
- `payroll_items` - Individual salary line items
- `ledger_entries` - Complete transaction ledger (immutable)
- `admin_audit_logs` - All admin operations

**RBAC Hierarchy:**
```
super_admin → admin → finance → employee
```

**Existing Payroll Capabilities:**
- Excel file upload (max 10MB, 1000 rows)
- Batch validation and deduplication (batch_hash)
- Approval/rejection workflow
- Partial failure handling (skip_invalid flag)
- Internal transfer creation per employee
- XLSX/CSV export with transfer IDs
- 24-hour batch expiration

---

## 1.3 Dependency Mapping

### Service Dependency Graph

```
                    ┌─────────────────────────────────────────┐
                    │         API Gateway (Port 8080)         │
                    │    HTTP → gRPC, Auth, Rate Limiting     │
                    └─────────────────────────────────────────┘
                                        │
          ┌─────────────────┬───────────┼───────────┬─────────────────┐
          │                 │           │           │                 │
          ▼                 ▼           ▼           ▼                 ▼
┌─────────────────┐ ┌──────────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐
│ Business Service│ │ ChainRPC     │ │ChainSync │ │ Signer   │ │ Admin Service│
│   (48 RPC)      │ │ (28 RPC)     │ │(17 RPC)  │ │ (14 RPC) │ │  (30+ RPC)   │
└─────────────────┘ └──────────────┘ └──────────┘ └──────────┘ └──────────────┘
         │                 │              │             │              │
         │                 │              │             │              │
         ├─────────────────┼──────────────┼─────────────┤              │
         │                 │              │             │              │
         │                 └──────────────┼─────────────┘              │
         │                                │                            │
         ▼                                ▼                            ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Common Infrastructure                                 │
│   middleware/ │ cache/ │ db/ │ repository/ │ utils/ │ constants/ │ errcode/ │
└─────────────────────────────────────────────────────────────────────────────┘
         │                                │                            │
         ▼                                ▼                            ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           Infrastructure Layer                               │
│     MariaDB  │  Redis  │  Kafka  │  ETCD  │  Prometheus  │  Jaeger          │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Direct Dependencies

| Module | Depends On | Dependency Type |
|--------|------------|-----------------|
| API Gateway | Business, ChainRPC, ChainSync, Signer, Admin | gRPC calls |
| API Gateway | ETCD | Service discovery |
| API Gateway | Redis | Rate limiting state |
| Business Service | ChainRPC | Address generation queries |
| Business Service | Signer | HD wallet derivation |
| Business Service | MariaDB | User data persistence |
| Business Service | Redis | Sessions, verification codes |
| Business Service | Kafka | Event distribution (optional) |
| ChainRPC Service | Signer | Transaction signing |
| ChainRPC Service | RPC Nodes | Blockchain interaction |
| ChainRPC Service | Redis | Gas price/balance caching |
| ChainSync Service | RPC Nodes | Blockchain data fetching |
| ChainSync Service | MariaDB | Block/transaction storage |
| ChainSync Service | Redis | Sync state caching |
| ChainSync Service | Kafka | Event publishing |
| Signer Service | MariaDB | Seed/address storage |
| Signer Service | Redis | Temporary data |
| Admin Service | Business | User operations |
| Admin Service | ChainRPC | Balance queries |
| Admin Service | MariaDB | Admin data persistence |

### Upstream/Downstream Relationships

**Upstream Services (Provide services to others):**
1. **Signer Service** - Provides key management to Business, ChainRPC
2. **ChainRPC Service** - Provides blockchain operations to Business, Admin
3. **ChainSync Service** - Provides sync/monitoring to Business (via Kafka)
4. **Common Infrastructure** - Provides utilities to ALL services

**Downstream Services (Consume services from others):**
1. **API Gateway** - Consumes ALL backend services
2. **Business Service** - Consumes ChainRPC, Signer, ChainSync events
3. **Admin Service** - Consumes Business, ChainRPC

### Coupling Issues Identified

1. **Tight Coupling:** Business Service has direct dependency on both ChainRPC and Signer for address operations (could be abstracted)

2. **Potential Circular Risk:** Admin → Business → ChainRPC, Admin → ChainRPC (no actual circular dependency, but complex call paths)

3. **Missing Abstraction:** No unified "Wallet Service" layer abstracting multi-chain operations from business logic

4. **Event Coupling:** ChainSync publishes to Kafka, but consumer coupling is implicit (not documented which services consume which topics)

### External Dependencies

| Dependency | Purpose | Used By |
|------------|---------|---------|
| QuickNode | RPC provider | ChainRPC, ChainSync |
| Infura | RPC provider (ETH) | ChainRPC, ChainSync |
| Alchemy | RPC provider | ChainRPC, ChainSync |
| Ankr | RPC provider | ChainRPC, ChainSync |
| Moralis | RPC provider | ChainRPC, ChainSync |
| TronGrid | TRON API | ChainRPC, ChainSync |
| 1inch | DEX aggregator | Business (swap) |
| Paraswap | DEX aggregator | Business (swap) |
| Chainalysis/Elliptic/Coinfirm | AML screening | Admin (withdrawal approval) |

---

## 1.4 Architecture Pattern Assessment

### Self-Custodial Components (Employee Wallets)

**Current Implementation:** LIMITED

The current system is primarily designed as a **custodial exchange wallet**, not self-custodial. However, the architecture supports extension:

**Existing Self-Custodial Elements:**
1. **BIP44 HD Derivation** (Signer Service) - Can derive unique addresses per user
2. **Per-User Address Generation** - Each user gets dedicated deposit addresses
3. **Address Monitoring** (ChainSync) - Watches user-specific addresses

**Missing Self-Custodial Features:**
- Client-side key generation and storage
- User-controlled mnemonic backup/recovery
- Hardware wallet integration for users
- User-initiated transaction signing (all signing is server-side)

**Self-Custodial Boundary:** The `deposit_addresses` table stores addresses but NOT private keys for individual users. Private keys remain in `master_seeds` (company-controlled).

### Centralized Wallet Components (Company Treasury)

**Current Implementation:** COMPREHENSIVE

The system excels at centralized wallet management:

1. **Hot Wallet Management:**
   - `vault_balances` tracks hot wallet balances per chain/asset
   - `AdminAdjustVaultBalance` for manual top-ups
   - Alert thresholds for low balance warnings

2. **Cold Wallet Support:**
   - Signer Service `temperature` field (hot=1, cold=2)
   - Separate seeds for hot vs cold storage
   - No automated cold-to-hot transfers documented

3. **Treasury Operations:**
   - Payroll batch disbursement from vault
   - Vault adjustment with dual-approval
   - Complete ledger tracking

4. **Key Management:**
   - Master seeds encrypted with AES-256-GCM
   - Seed status management (active/suspended/deprecated)
   - Daily signing limits (partially implemented)

### Separation Strategy

**Current Separation:**
```
┌─────────────────────────────────────────────────────────────────────┐
│                    CENTRALIZED (Company-Controlled)                  │
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────────────┐ │
│  │master_seeds │    │ vault_      │    │ Signer Service          │ │
│  │(encrypted)  │    │ balances    │    │ (internal gRPC only)    │ │
│  └─────────────┘    └─────────────┘    └─────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ Derived addresses (public keys only)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    SEMI-CUSTODIAL (User-Facing)                     │
│                                                                     │
│  ┌─────────────────┐    ┌─────────────────┐                        │
│  │deposit_addresses│    │ user account    │                        │
│  │(pub keys only)  │    │ balances        │                        │
│  └─────────────────┘    └─────────────────┘                        │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

**Isolation Mechanisms:**
1. **Network Isolation:** Signer Service should only accept internal gRPC (documented as "should NOT be exposed via public API gateway")
2. **Database Isolation:** Seeds stored encrypted, accessible only by Signer Service
3. **API Isolation:** No public endpoints expose signing operations directly

### Hybrid Patterns

**Documented Hybrid Model (from PRD):**
- **Web2 Wallet (Custodial):** Company-controlled, salary distribution, internal transfers
- **Web3 Wallet (Non-custodial):** Employee-controlled for external operations

**Current Implementation Gap:** The non-custodial (Web3) wallet component is conceptually documented but not fully implemented in the SRDs. The system currently operates primarily in custodial mode.

### Security Boundaries

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ TRUST BOUNDARY 1: Public Internet ←→ API Gateway                            │
│ Protection: JWT auth, rate limiting, CORS, blacklist                        │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ TRUST BOUNDARY 2: API Gateway ←→ Backend Services                           │
│ Protection: Internal gRPC, service mesh (recommended)                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ TRUST BOUNDARY 3: Services ←→ Signer Service (CRITICAL)                     │
│ Protection: Network isolation, no public access, audit logging              │
│ Risk: Currently accessible from API Gateway (CRITICAL-01 security issue)    │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ TRUST BOUNDARY 4: Signer Service ←→ Master Seeds                            │
│ Protection: AES-256-GCM encryption, PBKDF2 key derivation, TEE (optional)   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

# 2. Proposed Extensions

## 2.A Multi-Chain Support Enhancements

### 2.A.1 TRON Network Integration (Enhancements)

**Current State:** Implemented (TRX/TRC20)

**Proposed Enhancements:**

| Enhancement | Affected Modules | New Modules | Technical Requirements |
|-------------|------------------|-------------|------------------------|
| Energy/bandwidth management | ChainRPC | None | TronGrid API for resource estimation |
| Resource delegation support | ChainRPC, Business | `TronResourceManager` | Stake/unstake TRX for energy |
| TRC20 approval optimization | ChainRPC | None | Batch approve for payroll efficiency |
| Multi-sig TRON support | ChainRPC, Signer | `TronMultiSigAdapter` | TRON multi-sig contract deployment |

**Integration Points:**
- ChainRPC `BuildTokenTransfer` → Add `energy_estimate` and `bandwidth_estimate` fields
- Gas estimation → Convert to energy/bandwidth for TRON
- New RPC: `GetTronResources(address)` → Returns energy/bandwidth balance

**Chain-Specific Considerations:**
- TRON uses protobuf for transaction serialization (different from EVM RLP)
- Energy rental market integration for large payroll batches
- 3-second block time vs 12 seconds for Ethereum

### 2.A.2 BSC Network Integration (Enhancements)

**Current State:** Implemented (BNB/BEP20)

**Proposed Enhancements:**

| Enhancement | Affected Modules | New Modules | Technical Requirements |
|-------------|------------------|-------------|------------------------|
| BSC-specific gas optimization | ChainRPC | None | Gas price oracle integration |
| Faster confirmation tracking | ChainSync | None | 3-second block handling |
| BEP20 batch transfer contract | ChainRPC | `BSCBatchTransfer` | Deploy gas-efficient batch contract |

**Integration Points:**
- ChainSync confirmation config → Reduce from 15 to 10 for standard transactions
- Gas price oracle → BSCScan API integration

### 2.A.3 Ethereum Network Integration (Enhancements)

**Current State:** Implemented (ETH/ERC20)

**Proposed Enhancements:**

| Enhancement | Affected Modules | New Modules | Technical Requirements |
|-------------|------------------|-------------|------------------------|
| EIP-1559 full support | ChainRPC | None | Base fee + priority fee handling |
| Gas price prediction | ChainRPC | `GasPricePredictor` | Historical analysis for scheduling |
| Layer 2 preparation | ChainRPC, ChainSync | `L2Adapter` interface | Polygon/Arbitrum/Optimism stubs |
| MEV protection | ChainRPC | `FlashbotsAdapter` | Private transaction submission |

**Integration Points:**
- `GetGasPrice` → Return `baseFee`, `priorityFee`, `maxFee` for EIP-1559
- Transaction building → Support `maxFeePerGas` and `maxPriorityFeePerGas`

### 2.A.4 Cross-Chain Transaction Orchestration

**Affected Modules:** ChainRPC, Business, Admin

**New Module:** `PayrollOrchestrator`

**Technical Requirements:**
```go
type PayrollOrchestrator interface {
    // Submit payroll across multiple chains
    SubmitMultiChainPayroll(batch PayrollBatch) (MultiChainResult, error)

    // Get aggregated status
    GetPayrollStatus(batchID string) (MultiChainStatus, error)

    // Retry failed transactions with chain fallback
    RetryWithFallback(txID string, fallbackChain ChainType) error
}
```

**Integration Points:**
- Admin `ProcessPayrollBatch` → Calls `PayrollOrchestrator.SubmitMultiChainPayroll`
- Parallel submission to TRON, BSC, ETH based on employee preferences
- Unified error handling with chain-specific retry logic

### 2.A.5 Multi-Chain Fee Optimization

**Affected Modules:** ChainRPC, Admin

**New Module:** `FeeOptimizer`

**Technical Requirements:**
```go
type FeeOptimizer interface {
    // Compare fees across chains for same transfer
    CompareFees(amount *big.Int, token string) ([]ChainFeeEstimate, error)

    // Recommend optimal chain for payment
    RecommendChain(amount *big.Int, urgency FeeUrgency) (ChainType, error)

    // Get historical fee analysis
    GetFeeHistory(chain ChainType, period time.Duration) (FeeHistory, error)
}
```

**Integration Points:**
- Pre-payroll analysis → Show cost breakdown by chain
- Employee address management → Suggest preferred chain based on historical fees

### 2.A.6 Aggregated Balance Reporting

**Affected Modules:** Admin, ChainRPC

**New Module:** `BalanceAggregator`

**Technical Requirements:**
- Consolidated balance view: `GetAggregatedBalances()` → All chains, all tokens, single response
- Real-time sync from ChainSync events
- USD/USDT equivalent calculation for all balances
- Hot vs cold wallet breakdown

**Database Schema Addition:**
```sql
CREATE TABLE aggregated_balances (
    id BIGINT PRIMARY KEY,
    wallet_type ENUM('hot', 'cold', 'user') NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    token_address VARCHAR(64),
    token_symbol VARCHAR(20) NOT NULL,
    balance DECIMAL(36, 18) NOT NULL,
    usdt_value DECIMAL(20, 8),
    last_updated DATETIME NOT NULL,
    INDEX idx_wallet_chain (wallet_type, chain)
);
```

### 2.A.7 Token Standard Abstraction

**Affected Modules:** ChainRPC

**New Module:** `TokenAdapter`

**Technical Requirements:**
```go
type TokenAdapter interface {
    // Chain-agnostic transfer
    Transfer(ctx context.Context, req TransferRequest) (TxHash, error)

    // Chain-agnostic balance
    BalanceOf(ctx context.Context, address string, token string) (*big.Int, error)

    // Chain-agnostic approval
    Approve(ctx context.Context, spender string, amount *big.Int) (TxHash, error)
}

// Implementations
type TRC20Adapter struct { /* TRON-specific */ }
type BEP20Adapter struct { /* BSC-specific */ }
type ERC20Adapter struct { /* ETH-specific */ }
```

**Integration Points:**
- ChainRPC internally uses `TokenAdapter` interface
- Factory pattern: `GetTokenAdapter(chain ChainType) TokenAdapter`

---

## 2.B Self-Custodial Wallet Features (Employee Wallets)

### 2.B.1 HD Wallet Derivation Management

**Affected Modules:** Signer

**Enhancements:**

| Feature | Implementation |
|---------|----------------|
| Per-employee derivation paths | `m/44'/{coin}'/0'/0/{employee_id}` (current) |
| Derivation path registry | New table `derivation_registry` |
| Path collision detection | Check before assignment |

**Database Schema:**
```sql
CREATE TABLE derivation_registry (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    seed_id BIGINT NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    derivation_path VARCHAR(100) NOT NULL,
    address VARCHAR(64) NOT NULL,
    assigned_at DATETIME NOT NULL,
    UNIQUE KEY uk_employee_chain (employee_id, chain),
    UNIQUE KEY uk_path (seed_id, derivation_path)
);
```

### 2.B.2 Secure Key Generation (Client-Side Extension)

**New Module:** `ClientKeyManager` (mobile/web SDK)

**Technical Requirements:**
- CSPRNG using platform secure random (iOS: SecRandomCopyBytes, Android: SecureRandom)
- Minimum 256-bit entropy validation
- Key generation audit logging (timestamp only, NOT keys)
- Air-gapped key generation option (QR code transport)

**Note:** This is a client-side component, not server-side. Server never sees employee private keys in true self-custodial mode.

### 2.B.3 Key Storage Mechanisms (Client-Side)

**New Module:** `SecureKeyStore` (mobile SDK)

**Technical Requirements:**
- iOS: Keychain Services with `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`
- Android: AndroidKeyStore with hardware-backed keys
- Encryption: AES-256-GCM with biometric-protected key
- Key wrapping for backup

### 2.B.4 Backup and Recovery Workflows

**Affected Modules:** Business (new endpoints)

**New APIs:**
- `InitiateBackupVerification` - Generate challenge for mnemonic verification
- `VerifyBackup` - Confirm user has recorded mnemonic correctly
- `RequestRecovery` - Initiate time-locked recovery (e.g., 48-hour delay)

**Technical Requirements:**
- BIP-39 mnemonic display (12 or 24 words)
- Screenshot prevention (FLAG_SECURE on Android, UITextField.isSecureTextEntry on iOS)
- Recovery phrase verification (user re-enters 3 random words)
- Optional: Shamir's Secret Sharing for enterprise key recovery

### 2.B.5 Multi-Chain Address Generation (Enhancement)

**Affected Modules:** Signer

**Current:** Already supports multi-chain from single seed

**Enhancement:**
- Add `GenerateEmployeeWalletSet(employee_id)` → Returns all 3 chain addresses at once
- Address labeling: Store employee name, department in `deposit_addresses.metadata`

---

## 2.C Centralized Wallet Account Features (Company Treasury)

### 2.C.1 Hot Wallet Management (Enhancement)

**Affected Modules:** Admin, Business

**New Module:** `HotWalletManager`

**Technical Requirements:**
```go
type HotWalletManager interface {
    // Check if hot wallet needs replenishment
    CheckReplenishmentNeeded(chain ChainType) (bool, *big.Int, error)

    // Trigger replenishment from cold wallet
    RequestReplenishment(chain ChainType, amount *big.Int) (RequestID, error)

    // Rotate hot wallet (generate new, transfer balance)
    RotateHotWallet(chain ChainType) error

    // Emergency drain to cold wallet
    EmergencyDrain(chain ChainType) error
}
```

**Database Schema:**
```sql
CREATE TABLE hot_wallet_config (
    id BIGINT PRIMARY KEY,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    wallet_address VARCHAR(64) NOT NULL,
    min_balance DECIMAL(36, 18) NOT NULL,
    target_balance DECIMAL(36, 18) NOT NULL,
    max_balance DECIMAL(36, 18) NOT NULL,
    auto_replenish BOOLEAN DEFAULT FALSE,
    last_rotation DATETIME,
    UNIQUE KEY uk_chain (chain)
);
```

### 2.C.2 Cold Wallet Integration

**Affected Modules:** Signer, Admin

**New Module:** `ColdWalletBridge`

**Technical Requirements:**
- Offline signing workflow: Export unsigned tx → Sign on air-gapped device → Import signature
- Multi-signature support for cold wallet transfers
- Cold wallet balance monitoring (watch-only)

**New APIs:**
- `ExportUnsignedTransaction(tx)` → Returns QR code or file
- `ImportSignedTransaction(signature)` → Broadcasts signed tx
- `GetColdWalletBalance(chain)` → Read-only balance check

### 2.C.3 Automated Fund Allocation

**New Module:** `TreasuryAutomation`

**Technical Requirements:**
```go
type TreasuryAutomation interface {
    // Configure auto-replenishment rules
    SetReplenishmentRule(rule ReplenishmentRule) error

    // Predict funding needs based on payroll schedule
    PredictFundingNeeds(nextDays int) ([]FundingPrediction, error)

    // Execute scheduled fund movements
    ExecuteScheduledTransfers() error
}

type ReplenishmentRule struct {
    Chain           ChainType
    TriggerBalance  *big.Int  // Trigger when below this
    TargetBalance   *big.Int  // Replenish to this amount
    SourceWallet    string    // Cold wallet address
    RequiresApproval bool
    ApproverCount   int       // For multi-sig
}
```

### 2.C.4 Internal Account Ledger (Enhancement)

**Affected Modules:** Admin (`ledger_entries` table)

**Enhancements:**
- Double-entry bookkeeping enforcement
- Account hierarchy support (company → department → cost center)
- Accrual accounting (record liability at approval, not disbursement)

**Database Schema Addition:**
```sql
CREATE TABLE ledger_accounts (
    id BIGINT PRIMARY KEY,
    account_code VARCHAR(20) NOT NULL UNIQUE,
    account_name VARCHAR(100) NOT NULL,
    account_type ENUM('asset', 'liability', 'equity', 'revenue', 'expense') NOT NULL,
    parent_id BIGINT,
    is_active BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (parent_id) REFERENCES ledger_accounts(id)
);

-- Enhance existing ledger_entries
ALTER TABLE ledger_entries ADD COLUMN debit_account_id BIGINT;
ALTER TABLE ledger_entries ADD COLUMN credit_account_id BIGINT;
```

---

## 2.D Payroll-Specific Capabilities

### 2.D.1 Batch Payment Processing (Enhancement)

**Affected Modules:** Admin

**Current State:** Supports up to 1000 recipients per batch

**Enhancements:**

| Feature | Implementation |
|---------|----------------|
| Increase to 10,000+ recipients | Chunked processing, background jobs |
| Multi-chain batch operations | Parallel submission to 3 chains |
| Nonce management | Per-chain nonce sequencing |
| Partial completion tracking | Item-level status in `payroll_items` |

**Technical Requirements:**
```go
type BatchProcessor interface {
    // Process large batch with chunking
    ProcessBatch(batch PayrollBatch, options ProcessOptions) (BatchResult, error)

    // Resume failed batch from checkpoint
    ResumeBatch(batchID string) (BatchResult, error)

    // Get real-time progress
    GetBatchProgress(batchID string) (BatchProgress, error)
}

type ProcessOptions struct {
    ChunkSize       int           // Items per chunk (default: 100)
    ParallelChains  bool          // Submit to multiple chains simultaneously
    RetryCount      int           // Retries per failed item
    ContinueOnError bool          // Don't stop on individual failures
}
```

### 2.D.2 Scheduled/Recurring Payments

**Affected Modules:** Admin, Business

**New Module:** `PayrollScheduler`

**Technical Requirements:**
```go
type PayrollScheduler interface {
    // Create recurring schedule
    CreateSchedule(schedule PayrollSchedule) (ScheduleID, error)

    // Modify existing schedule
    UpdateSchedule(scheduleID string, updates ScheduleUpdates) error

    // Pause/resume schedule
    SetScheduleStatus(scheduleID string, status ScheduleStatus) error

    // Get upcoming executions
    GetUpcomingExecutions(days int) ([]ScheduledExecution, error)

    // Manual trigger
    TriggerManually(scheduleID string) error
}

type PayrollSchedule struct {
    ID              string
    Name            string
    CronExpression  string            // "0 9 1,15 * *" for 1st and 15th
    Timezone        string            // "America/New_York"
    TemplateID      string            // References payment template
    PreExecutionHours int             // Hours before to validate
    AutoApprove     bool              // For routine payroll
    AutoApproveLimit *big.Int         // Max amount for auto-approve
}
```

**Database Schema:**
```sql
CREATE TABLE payroll_schedules (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    cron_expression VARCHAR(50) NOT NULL,
    timezone VARCHAR(50) NOT NULL DEFAULT 'UTC',
    template_id BIGINT NOT NULL,
    pre_execution_hours INT DEFAULT 24,
    auto_approve BOOLEAN DEFAULT FALSE,
    auto_approve_limit DECIMAL(20, 8),
    status ENUM('active', 'paused', 'completed') DEFAULT 'active',
    last_execution DATETIME,
    next_execution DATETIME,
    created_by BIGINT NOT NULL,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    FOREIGN KEY (template_id) REFERENCES payroll_templates(id)
);

CREATE TABLE scheduled_executions (
    id BIGINT PRIMARY KEY,
    schedule_id BIGINT NOT NULL,
    scheduled_at DATETIME NOT NULL,
    executed_at DATETIME,
    status ENUM('pending', 'executing', 'completed', 'failed', 'skipped') DEFAULT 'pending',
    batch_id BIGINT,
    error_message TEXT,
    FOREIGN KEY (schedule_id) REFERENCES payroll_schedules(id),
    FOREIGN KEY (batch_id) REFERENCES payroll_batches(id)
);
```

### 2.D.3 Payment Template Management

**Affected Modules:** Admin

**New Module:** `TemplateManager`

**Technical Requirements:**
```go
type PayrollTemplate struct {
    ID              string
    Name            string
    Version         int
    Recipients      []TemplateRecipient
    DefaultChain    ChainType
    DefaultToken    string
    GasSettings     GasSettings
    Variables       map[string]VariableDefinition  // For dynamic amounts
    CreatedBy       string
    CreatedAt       time.Time
    UpdatedAt       time.Time
}

type TemplateRecipient struct {
    EmployeeID      string
    Chain           ChainType       // Override per employee
    Token           string          // Override per employee
    AmountType      string          // "fixed" or "variable"
    FixedAmount     *big.Int
    VariableKey     string          // Reference to Variables map
}
```

**Database Schema:**
```sql
CREATE TABLE payroll_templates (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    version INT NOT NULL DEFAULT 1,
    template_data JSON NOT NULL,  -- Stores full TemplateRecipient array
    default_chain ENUM('TRON', 'BSC', 'ETH'),
    default_token VARCHAR(64),
    gas_settings JSON,
    variables JSON,
    is_active BOOLEAN DEFAULT TRUE,
    created_by BIGINT NOT NULL,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    UNIQUE KEY uk_name_version (name, version)
);

CREATE TABLE template_history (
    id BIGINT PRIMARY KEY,
    template_id BIGINT NOT NULL,
    version INT NOT NULL,
    changed_by BIGINT NOT NULL,
    changed_at DATETIME NOT NULL,
    change_type ENUM('create', 'update', 'deactivate') NOT NULL,
    previous_data JSON,
    new_data JSON,
    FOREIGN KEY (template_id) REFERENCES payroll_templates(id)
);
```

### 2.D.4 Multi-Currency/Multi-Token Support

**Affected Modules:** Business, Admin

**Current State:** Partial (supports multiple tokens per chain)

**Enhancements:**
- Per-employee token preference storage
- Automatic token conversion via DEX (if needed)
- Exchange rate locking at batch creation

**Database Schema Addition:**
```sql
CREATE TABLE employee_payment_preferences (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    preferred_chain ENUM('TRON', 'BSC', 'ETH'),
    preferred_token VARCHAR(64),
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    UNIQUE KEY uk_employee (employee_id)
);

ALTER TABLE payroll_items
ADD COLUMN exchange_rate DECIMAL(20, 8),
ADD COLUMN exchange_locked_at DATETIME,
ADD COLUMN original_token VARCHAR(64),
ADD COLUMN original_amount DECIMAL(36, 18);
```

### 2.D.5 Payment Approval Workflows (Enhancement)

**Affected Modules:** Admin

**Current State:** Basic approval (maker-checker for large batches)

**Enhancements:**

| Feature | Implementation |
|---------|----------------|
| Multi-level approval | Configurable approval chain |
| Department-based routing | Different approvers per department |
| Approval delegation | Temporary delegate assignment |
| Time-based auto-approval | For routine payroll under threshold |
| Approval expiration | Re-approval required after X days |

**Database Schema:**
```sql
CREATE TABLE approval_chains (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE,
    created_at DATETIME NOT NULL
);

CREATE TABLE approval_chain_levels (
    id BIGINT PRIMARY KEY,
    chain_id BIGINT NOT NULL,
    level_order INT NOT NULL,
    required_approvers INT NOT NULL DEFAULT 1,
    timeout_hours INT,  -- Auto-escalate after timeout
    FOREIGN KEY (chain_id) REFERENCES approval_chains(id)
);

CREATE TABLE approval_chain_approvers (
    id BIGINT PRIMARY KEY,
    level_id BIGINT NOT NULL,
    approver_user_id BIGINT,
    approver_role VARCHAR(50),  -- Can approve by role
    FOREIGN KEY (level_id) REFERENCES approval_chain_levels(id)
);

CREATE TABLE payroll_approvals (
    id BIGINT PRIMARY KEY,
    batch_id BIGINT NOT NULL,
    level_id BIGINT NOT NULL,
    approver_id BIGINT,
    status ENUM('pending', 'approved', 'rejected', 'delegated', 'expired') DEFAULT 'pending',
    comments TEXT,
    approved_at DATETIME,
    expires_at DATETIME,
    delegated_to BIGINT,
    FOREIGN KEY (batch_id) REFERENCES payroll_batches(id),
    FOREIGN KEY (level_id) REFERENCES approval_chain_levels(id)
);
```

### 2.D.6 Failed Transaction Retry

**Affected Modules:** Admin, ChainRPC

**New Module:** `RetryManager`

**Technical Requirements:**
```go
type RetryManager interface {
    // Configure retry policy
    SetRetryPolicy(policy RetryPolicy) error

    // Automatic retry with backoff
    AutoRetry(txID string) error

    // Manual retry with options
    ManualRetry(txID string, options RetryOptions) error

    // Get retry history
    GetRetryHistory(txID string) ([]RetryAttempt, error)

    // Move to dead letter queue
    MoveToDeadLetter(txID string, reason string) error
}

type RetryPolicy struct {
    MaxRetries          int
    InitialBackoff      time.Duration
    MaxBackoff          time.Duration
    BackoffMultiplier   float64
    RetryableErrors     []string        // Error codes that trigger retry
    GasIncreasePct      int             // Increase gas by X% on retry
    EnableChainFallback bool            // Try alternate chain on repeated failure
}
```

**Database Schema:**
```sql
CREATE TABLE transaction_retries (
    id BIGINT PRIMARY KEY,
    original_tx_id VARCHAR(100) NOT NULL,
    payroll_item_id BIGINT,
    attempt_number INT NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    tx_hash VARCHAR(100),
    status ENUM('pending', 'submitted', 'confirmed', 'failed') NOT NULL,
    error_code VARCHAR(50),
    error_message TEXT,
    gas_price_used DECIMAL(20, 8),
    attempted_at DATETIME NOT NULL,
    FOREIGN KEY (payroll_item_id) REFERENCES payroll_items(id)
);

CREATE TABLE dead_letter_transactions (
    id BIGINT PRIMARY KEY,
    original_tx_id VARCHAR(100) NOT NULL,
    payroll_item_id BIGINT,
    total_attempts INT NOT NULL,
    final_error_code VARCHAR(50),
    final_error_message TEXT,
    moved_at DATETIME NOT NULL,
    resolution_status ENUM('unresolved', 'manual_resolved', 'refunded') DEFAULT 'unresolved',
    resolution_notes TEXT,
    resolved_by BIGINT,
    resolved_at DATETIME
);
```

### 2.D.7 Payment Status Tracking (Enhancement)

**Affected Modules:** Admin, ChainSync

**Current State:** Basic status tracking

**Enhancements:**

| Feature | Implementation |
|---------|----------------|
| Real-time status updates | WebSocket push to admin dashboard |
| Confirmation depth tracking | Show 3/12 confirmations |
| Webhook notifications | POST to external systems |
| Employee status portal | Read-only view for employees |
| Push notifications | Email, SMS, in-app |

**New APIs:**
- WebSocket: `ws://api/v1/payroll/stream/{batch_id}`
- Webhook config: `POST /api/v1/admin/webhooks`
- Employee portal: `GET /api/v1/employee/payments`

**Database Schema:**
```sql
CREATE TABLE webhook_configs (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    url VARCHAR(500) NOT NULL,
    secret_key VARCHAR(100) NOT NULL,
    events JSON NOT NULL,  -- ["payment.confirmed", "payment.failed"]
    is_active BOOLEAN DEFAULT TRUE,
    created_by BIGINT NOT NULL,
    created_at DATETIME NOT NULL
);

CREATE TABLE webhook_deliveries (
    id BIGINT PRIMARY KEY,
    webhook_id BIGINT NOT NULL,
    event_type VARCHAR(50) NOT NULL,
    payload JSON NOT NULL,
    response_code INT,
    response_body TEXT,
    delivered_at DATETIME,
    retry_count INT DEFAULT 0,
    FOREIGN KEY (webhook_id) REFERENCES webhook_configs(id)
);
```

---

## 2.E Security Enhancements

### 2.E.1 Multi-Signature Support

**Affected Modules:** Signer, Admin

**New Module:** `MultiSigManager`

**Technical Requirements:**

| Chain | Implementation |
|-------|----------------|
| Ethereum/BSC | Gnosis Safe contract |
| TRON | TRON multi-sig contract |
| All chains | Off-chain approval with single on-chain signature |

```go
type MultiSigManager interface {
    // Create multi-sig configuration
    CreateMultiSig(config MultiSigConfig) (MultiSigID, error)

    // Request signatures
    RequestSignatures(txID string) error

    // Submit signature
    SubmitSignature(txID string, signerID string, signature []byte) error

    // Check if threshold met
    CheckThreshold(txID string) (bool, error)

    // Finalize and broadcast
    Finalize(txID string) (TxHash, error)
}

type MultiSigConfig struct {
    Name            string
    Threshold       int         // M in M-of-N
    TotalSigners    int         // N in M-of-N
    SignerIDs       []string
    AmountThreshold *big.Int    // Only require multi-sig above this amount
}
```

**Database Schema:**
```sql
CREATE TABLE multisig_configs (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    threshold INT NOT NULL,
    total_signers INT NOT NULL,
    amount_threshold DECIMAL(20, 8),
    chain ENUM('TRON', 'BSC', 'ETH', 'ALL') NOT NULL,
    contract_address VARCHAR(64),  -- For on-chain multi-sig
    is_active BOOLEAN DEFAULT TRUE,
    created_at DATETIME NOT NULL
);

CREATE TABLE multisig_signers (
    id BIGINT PRIMARY KEY,
    config_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    signer_address VARCHAR(64),
    FOREIGN KEY (config_id) REFERENCES multisig_configs(id)
);

CREATE TABLE multisig_requests (
    id BIGINT PRIMARY KEY,
    config_id BIGINT NOT NULL,
    transaction_id VARCHAR(100) NOT NULL,
    raw_transaction TEXT NOT NULL,
    status ENUM('pending', 'threshold_met', 'executed', 'expired', 'cancelled') DEFAULT 'pending',
    expires_at DATETIME,
    created_at DATETIME NOT NULL,
    FOREIGN KEY (config_id) REFERENCES multisig_configs(id)
);

CREATE TABLE multisig_signatures (
    id BIGINT PRIMARY KEY,
    request_id BIGINT NOT NULL,
    signer_id BIGINT NOT NULL,
    signature TEXT NOT NULL,
    signed_at DATETIME NOT NULL,
    FOREIGN KEY (request_id) REFERENCES multisig_requests(id),
    FOREIGN KEY (signer_id) REFERENCES multisig_signers(id)
);
```

### 2.E.2 Hardware Wallet Integration

**Affected Modules:** Signer (cold wallet operations), Admin (UI)

**New Module:** `HardwareWalletBridge`

**Technical Requirements:**
- Ledger support via Ledger Live API or HID
- Trezor support via Trezor Connect
- USB/Bluetooth connectivity handling
- Firmware version verification

**Integration Points:**
- Cold wallet signing: Export unsigned tx → Sign on hardware device → Import signature
- Admin UI: Hardware wallet connection wizard

### 2.E.3 Biometric Authentication

**Affected Modules:** Business (API), Mobile SDK

**Current State:** Partial (trusted_devices table exists)

**Enhancements:**
- Require biometric re-authentication for high-value approvals
- Biometric template hash storage (never store actual biometric data)
- Fallback to PIN/password on biometric failure

### 2.E.4 Role-Based Access Control (Enhancement)

**Affected Modules:** Admin

**Current State:** Basic roles (super_admin, admin, finance, employee)

**Enhancements:**

| Role | New Permissions |
|------|-----------------|
| Treasury Manager | Hot/cold wallet management, fund allocation |
| Payroll Operator | Create/modify batches, cannot approve own batches |
| Auditor | Read-only access to all records, export capabilities |

**Database Schema Enhancement:**
```sql
-- Granular permissions
CREATE TABLE permissions (
    id BIGINT PRIMARY KEY,
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL,  -- payroll, treasury, audit, user_management
    description TEXT
);

-- Role-permission mapping
CREATE TABLE role_permissions (
    id BIGINT PRIMARY KEY,
    role_id BIGINT NOT NULL,
    permission_id BIGINT NOT NULL,
    UNIQUE KEY uk_role_permission (role_id, permission_id)
);

-- Sample permissions
INSERT INTO permissions (code, name, category) VALUES
('payroll.create', 'Create Payroll Batch', 'payroll'),
('payroll.approve', 'Approve Payroll Batch', 'payroll'),
('payroll.approve_own', 'Approve Own Payroll Batch', 'payroll'),  -- Usually denied
('treasury.view', 'View Treasury Balances', 'treasury'),
('treasury.transfer', 'Execute Treasury Transfers', 'treasury'),
('audit.export', 'Export Audit Logs', 'audit');
```

### 2.E.5 Transaction Velocity Limits

**Affected Modules:** Business, Admin

**New Module:** `VelocityLimiter`

**Technical Requirements:**
```go
type VelocityLimiter interface {
    // Check if transaction exceeds limits
    CheckLimits(userID string, amount *big.Int, txType string) (LimitResult, error)

    // Get current usage
    GetCurrentUsage(userID string) (UsageSummary, error)

    // Configure limits
    SetLimits(userID string, limits LimitConfig) error
}

type LimitConfig struct {
    DailyLimit      *big.Int
    WeeklyLimit     *big.Int
    MonthlyLimit    *big.Int
    PerTxLimit      *big.Int
    RequireApprovalAbove *big.Int
}
```

**Database Schema:**
```sql
CREATE TABLE velocity_limits (
    id BIGINT PRIMARY KEY,
    scope_type ENUM('user', 'role', 'wallet', 'global') NOT NULL,
    scope_id BIGINT,  -- user_id, role_id, wallet_id, or NULL for global
    daily_limit DECIMAL(20, 8),
    weekly_limit DECIMAL(20, 8),
    monthly_limit DECIMAL(20, 8),
    per_tx_limit DECIMAL(20, 8),
    require_approval_above DECIMAL(20, 8),
    is_active BOOLEAN DEFAULT TRUE,
    created_at DATETIME NOT NULL
);

CREATE TABLE velocity_usage (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    period_type ENUM('daily', 'weekly', 'monthly') NOT NULL,
    period_start DATE NOT NULL,
    total_amount DECIMAL(20, 8) NOT NULL DEFAULT 0,
    tx_count INT NOT NULL DEFAULT 0,
    last_updated DATETIME NOT NULL,
    UNIQUE KEY uk_user_period (user_id, period_type, period_start)
);
```

### 2.E.6 Comprehensive Audit Logging (Enhancement)

**Affected Modules:** All

**Current State:** `admin_audit_logs` and `signature_logs` exist

**Enhancements:**
- Immutable audit log storage (append-only)
- Cryptographic hash chain for tamper evidence
- Log retention: 7 years minimum
- SIEM integration (Splunk, ELK Stack export)

**Database Schema Enhancement:**
```sql
CREATE TABLE audit_log_chain (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    previous_hash VARCHAR(64) NOT NULL,  -- SHA-256 of previous entry
    entry_hash VARCHAR(64) NOT NULL,     -- SHA-256 of this entry
    user_id BIGINT,
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50) NOT NULL,
    resource_id VARCHAR(100),
    details JSON,
    ip_address VARCHAR(45),
    user_agent TEXT,
    timestamp DATETIME(6) NOT NULL,      -- Microsecond precision
    INDEX idx_user_action (user_id, action),
    INDEX idx_resource (resource_type, resource_id),
    INDEX idx_timestamp (timestamp)
) ENGINE=InnoDB ROW_FORMAT=COMPRESSED;

-- Trigger to enforce append-only
DELIMITER //
CREATE TRIGGER audit_log_chain_immutable
BEFORE UPDATE ON audit_log_chain
FOR EACH ROW
BEGIN
    SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Audit log is immutable';
END //
DELIMITER ;
```

---

## 2.F Operational Features

### 2.F.1 Transaction History Management (Enhancement)

**Affected Modules:** Admin

**Current State:** Basic filtering and export

**Enhancements:**
- Full-text search on transaction metadata
- Advanced filtering (amount range, confirmation depth)
- Bulk operations (export selection, retry selection)

**New APIs:**
```go
type TransactionHistoryQuery struct {
    Chains          []ChainType
    Tokens          []string
    Statuses        []TxStatus
    DateFrom        time.Time
    DateTo          time.Time
    AmountMin       *big.Int
    AmountMax       *big.Int
    RecipientSearch string      // Full-text search
    SenderSearch    string
    ConfirmationMin int
    Page            int
    PageSize        int
    SortBy          string
    SortOrder       string
}
```

### 2.F.2 Real-Time Balance Monitoring (Enhancement)

**Affected Modules:** Admin, ChainSync

**Enhancements:**
- WebSocket-based real-time dashboard updates
- Historical balance charts (line graph over time)
- Predicted balance (current - pending payroll)

**New APIs:**
- WebSocket: `ws://api/v1/admin/balances/stream`
- Historical: `GET /api/v1/admin/balances/history?period=30d`

### 2.F.3 Gas Fee Optimization

**Affected Modules:** ChainRPC, Admin

**New Module:** `GasOptimizer`

**Technical Requirements:**
```go
type GasOptimizer interface {
    // Get current gas prices across chains
    GetCurrentPrices() (map[ChainType]GasPrice, error)

    // Predict gas price at future time
    PredictPrice(chain ChainType, atTime time.Time) (GasPrice, error)

    // Recommend best time to submit
    RecommendSubmissionTime(chain ChainType, urgency Urgency) (time.Time, error)

    // Estimate total fees for payroll batch
    EstimateBatchFees(batch PayrollBatch) (FeeEstimate, error)
}
```

**Integration Points:**
- Pre-payroll validation: Show estimated fees before approval
- Scheduled payroll: Optionally defer execution to lower-fee window

### 2.F.4 Transaction Confirmation Tracking (Enhancement)

**Affected Modules:** ChainSync, Admin

**Current State:** Confirmation requirements defined per chain

**Enhancements:**
- Real-time confirmation progress: "3/12 confirmations"
- Blockchain reorg detection and notification
- Stuck transaction detection (pending > 1 hour)
- Transaction acceleration UI (increase gas)

### 2.F.5 Reconciliation Tools

**Affected Modules:** Admin

**New Module:** `ReconciliationEngine`

**Technical Requirements:**
```go
type ReconciliationEngine interface {
    // Run reconciliation for period
    Reconcile(from, to time.Time) (ReconciliationReport, error)

    // Find discrepancies
    FindDiscrepancies(chain ChainType) ([]Discrepancy, error)

    // Mark discrepancy as resolved
    ResolveDiscrepancy(discrepancyID string, resolution string) error

    // Export reconciliation report
    ExportReport(reportID string, format string) ([]byte, error)
}

type Discrepancy struct {
    ID              string
    Type            string  // "missing_on_chain", "missing_in_ledger", "amount_mismatch"
    Chain           ChainType
    TxHash          string
    LedgerEntryID   *int64
    OnChainAmount   *big.Int
    LedgerAmount    *big.Int
    DetectedAt      time.Time
    Status          string
}
```

**Database Schema:**
```sql
CREATE TABLE reconciliation_reports (
    id BIGINT PRIMARY KEY,
    period_start DATETIME NOT NULL,
    period_end DATETIME NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH', 'ALL') NOT NULL,
    total_on_chain_txs INT NOT NULL,
    total_ledger_entries INT NOT NULL,
    matched_count INT NOT NULL,
    discrepancy_count INT NOT NULL,
    status ENUM('pending', 'completed', 'failed') NOT NULL,
    created_at DATETIME NOT NULL,
    completed_at DATETIME
);

CREATE TABLE reconciliation_discrepancies (
    id BIGINT PRIMARY KEY,
    report_id BIGINT NOT NULL,
    type ENUM('missing_on_chain', 'missing_in_ledger', 'amount_mismatch', 'status_mismatch') NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    tx_hash VARCHAR(100),
    ledger_entry_id BIGINT,
    on_chain_amount DECIMAL(36, 18),
    ledger_amount DECIMAL(36, 18),
    status ENUM('unresolved', 'resolved', 'ignored') DEFAULT 'unresolved',
    resolution_notes TEXT,
    resolved_by BIGINT,
    resolved_at DATETIME,
    detected_at DATETIME NOT NULL,
    FOREIGN KEY (report_id) REFERENCES reconciliation_reports(id)
);
```

### 2.F.6 Employee Wallet Address Management (Enhancement)

**Affected Modules:** Admin, Business

**Current State:** Basic address book (`wallet_addresses` table)

**Enhancements:**
- Per-chain address storage for each employee
- Address verification workflow (send $0.01 to verify ownership)
- Address change approval (require finance approval)
- Bulk import/export (CSV)
- Address whitelist enforcement

**Database Schema:**
```sql
CREATE TABLE employee_wallet_addresses (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    address VARCHAR(64) NOT NULL,
    label VARCHAR(100),
    is_verified BOOLEAN DEFAULT FALSE,
    verified_at DATETIME,
    verification_tx_hash VARCHAR(100),
    is_active BOOLEAN DEFAULT TRUE,
    requires_approval BOOLEAN DEFAULT TRUE,
    approval_status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
    approved_by BIGINT,
    approved_at DATETIME,
    created_at DATETIME NOT NULL,
    updated_at DATETIME NOT NULL,
    UNIQUE KEY uk_employee_chain (employee_id, chain),
    INDEX idx_address (address)
);

CREATE TABLE address_change_requests (
    id BIGINT PRIMARY KEY,
    employee_id BIGINT NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    old_address VARCHAR(64),
    new_address VARCHAR(64) NOT NULL,
    reason TEXT,
    status ENUM('pending', 'approved', 'rejected') DEFAULT 'pending',
    reviewed_by BIGINT,
    reviewed_at DATETIME,
    review_notes TEXT,
    created_at DATETIME NOT NULL,
    FOREIGN KEY (employee_id) REFERENCES users(id)
);
```

---

## 2.G Monitoring and Reporting

### 2.G.1 Payroll Disbursement Reports

**Affected Modules:** Admin

**New Reports:**
- Summary by pay period, department, cost center
- Breakdown by chain and token
- Individual employee payment history (YTD)
- Comparison reports (current vs previous period, budget vs actual)

**New APIs:**
```go
type ReportGenerator interface {
    // Generate payroll summary
    GeneratePayrollSummary(period PayPeriod, options ReportOptions) (Report, error)

    // Generate department breakdown
    GenerateDepartmentBreakdown(period PayPeriod) (Report, error)

    // Generate employee YTD
    GenerateEmployeeYTD(employeeID string, year int) (Report, error)

    // Generate comparison
    GenerateComparison(period1, period2 PayPeriod) (Report, error)
}
```

### 2.G.2 Cost Analysis Reports

**Affected Modules:** Admin

**New Reports:**
- Total transaction fees by chain and period
- Fee percentage of total disbursement
- Cost comparison across chains
- Fee forecasting based on historical data

### 2.G.3 Balance Alerts (Enhancement)

**Affected Modules:** Admin

**Current State:** Basic alert thresholds

**Enhancements:**
- Predictive alerts ("Balance will be insufficient in 3 days")
- Alert escalation (if not acknowledged, escalate)
- Multi-channel notifications (email, SMS, Slack, PagerDuty)
- Alert history and acknowledgment tracking

**Database Schema:**
```sql
CREATE TABLE balance_alerts (
    id BIGINT PRIMARY KEY,
    alert_type ENUM('low_balance', 'high_balance', 'predicted_low', 'anomaly') NOT NULL,
    chain ENUM('TRON', 'BSC', 'ETH') NOT NULL,
    wallet_type ENUM('hot', 'cold') NOT NULL,
    current_balance DECIMAL(36, 18) NOT NULL,
    threshold_balance DECIMAL(36, 18),
    predicted_shortfall DECIMAL(36, 18),
    severity ENUM('info', 'warning', 'critical') NOT NULL,
    status ENUM('active', 'acknowledged', 'resolved') DEFAULT 'active',
    acknowledged_by BIGINT,
    acknowledged_at DATETIME,
    resolved_at DATETIME,
    created_at DATETIME NOT NULL,
    INDEX idx_status_severity (status, severity)
);

CREATE TABLE alert_notifications (
    id BIGINT PRIMARY KEY,
    alert_id BIGINT NOT NULL,
    channel ENUM('email', 'sms', 'slack', 'pagerduty', 'webhook') NOT NULL,
    recipient VARCHAR(255) NOT NULL,
    sent_at DATETIME,
    delivery_status ENUM('pending', 'sent', 'failed') NOT NULL,
    error_message TEXT,
    FOREIGN KEY (alert_id) REFERENCES balance_alerts(id)
);
```

### 2.G.4 Failed Transaction Reporting

**Affected Modules:** Admin

**New Reports:**
- Failed transaction dashboard (real-time)
- Root cause analysis (categorize by error type)
- Retry status tracking
- Failed transaction resolution workflow

**New APIs:**
- Dashboard: `GET /api/v1/admin/transactions/failed/dashboard`
- Analysis: `GET /api/v1/admin/transactions/failed/analysis?period=30d`

---

# 3. Prioritization Matrix

## 3.1 Tier 1 (MVP - Must Have)

| # | Feature | Justification | Complexity | Dependencies | Risk if Not Implemented |
|---|---------|---------------|------------|--------------|------------------------|
| 1 | **Multi-chain batch payment processing** | Core payroll function - must support TRON, BSC, ETH simultaneously | High | ChainRPC, Signer already support multi-chain | Cannot execute basic payroll |
| 2 | **Employee wallet address management** | Must know where to send payments | Medium | Users table, ChainRPC validation | Cannot identify payment recipients |
| 3 | **Payment status tracking** | Finance team must track disbursement status | Medium | ChainSync events | No visibility into payment success/failure |
| 4 | **Basic approval workflow** | Maker-checker for financial controls | Low | Already exists, enhance | SOX compliance failure, fraud risk |
| 5 | **Transaction retry mechanism** | Blockchain transactions fail; must have recovery | Medium | ChainRPC | Stuck payments, manual intervention required |
| 6 | **Vault balance monitoring** | Must ensure sufficient funds before payroll | Low | Already exists | Payroll failure due to insufficient funds |
| 7 | **Audit logging** | Regulatory and compliance requirement | Low | Mostly exists, enhance immutability | Audit failure, legal risk |
| 8 | **Gas fee estimation** | Must show costs before approval | Low | ChainRPC has this | Unexpected costs, approval without cost visibility |

## 3.2 Tier 2 (Phase 2 - Should Have)

| # | Feature | Justification | Complexity | Dependencies | Risk if Not Implemented |
|---|---------|---------------|------------|--------------|------------------------|
| 9 | **Scheduled/recurring payments** | Most payroll is recurring (bi-weekly, monthly) | Medium | Tier 1 batch processing | Manual batch creation each pay period |
| 10 | **Payment templates** | Reuse configurations, reduce errors | Medium | Employee address management | Manual data entry errors |
| 11 | **Multi-level approval workflow** | Different approval for different amounts | Medium | Basic approval | Inadequate controls for large payments |
| 12 | **Gas fee optimization** | Reduce costs across chains | Medium | Gas estimation | Higher than necessary transaction costs |
| 13 | **Aggregated balance reporting** | Single view across chains | Medium | ChainSync | Fragmented view of treasury |
| 14 | **Webhook notifications** | Integration with HR/ERP systems | Low | Status tracking | Manual data export to other systems |
| 15 | **Reconciliation tools** | Match on-chain to internal records | High | Ledger, ChainSync | Manual reconciliation, audit findings |
| 16 | **Velocity limits** | Prevent runaway transactions | Medium | Audit logging | Fraud or error could drain treasury |

## 3.3 Tier 3 (Future - Nice to Have)

| # | Feature | Justification | Complexity | Dependencies | Risk if Not Implemented |
|---|---------|---------------|------------|--------------|------------------------|
| 17 | **Multi-signature support** | Additional security for large transfers | High | Signer, Admin | Reduced security (single point of failure) |
| 18 | **Hardware wallet integration** | Enhanced cold wallet security | High | Signer | Software-only key management |
| 19 | **Hot wallet auto-replenishment** | Reduce manual treasury operations | Medium | Vault monitoring | Manual hot wallet management |
| 20 | **Advanced reporting** | Department breakdowns, YTD, comparisons | Medium | Basic reporting | Manual Excel analysis |
| 21 | **Employee self-service portal** | Employees view their own payment status | Medium | Status tracking | Support tickets for status inquiries |
| 22 | **Predictive balance alerts** | Proactive treasury management | Medium | Balance monitoring | Reactive rather than proactive |
| 23 | **Layer 2 support** | Lower fees on Polygon/Arbitrum | High | Multi-chain architecture | Higher fees on L1 only |
| 24 | **Cross-chain fee comparison** | Optimize chain selection per payment | Medium | Fee estimation | Suboptimal chain selection |

---

# 4. Architecture Recommendations

## 4.1 Multi-Chain Architecture

### Abstraction Layer Design

**Proposed Chain-Agnostic Interface:**
```go
// pkg/chain/interface.go
package chain

type ChainAdapter interface {
    // Identity
    GetChainType() ChainType
    GetChainName() string

    // Address operations
    ValidateAddress(address string) (bool, error)
    GenerateAddress(pubKey []byte) (string, error)

    // Balance operations
    GetNativeBalance(address string) (*big.Int, error)
    GetTokenBalance(address, tokenContract string) (*big.Int, error)

    // Transaction building
    BuildNativeTransfer(from, to string, amount *big.Int, gasPrice *big.Int) (*UnsignedTx, error)
    BuildTokenTransfer(from, to, tokenContract string, amount *big.Int, gasPrice *big.Int) (*UnsignedTx, error)

    // Fee estimation
    EstimateGas(tx *UnsignedTx) (uint64, error)
    GetGasPrice(speed GasSpeed) (*big.Int, error)

    // Transaction submission
    BroadcastTransaction(signedTx []byte) (txHash string, error)
    GetTransactionStatus(txHash string) (*TxStatus, error)
    GetTransactionReceipt(txHash string) (*TxReceipt, error)

    // Chain-specific
    GetConfirmationRequirement(amount *big.Int) int
}

// Common data structures
type UnsignedTx struct {
    ChainType    ChainType
    RawTx        []byte      // Chain-specific serialization
    From         string
    To           string
    Value        *big.Int
    GasPrice     *big.Int
    GasLimit     uint64
    Nonce        uint64
    Data         []byte
}

type TxStatus struct {
    Hash          string
    Status        TransactionStatus  // pending, confirmed, failed
    Confirmations int
    BlockNumber   uint64
    Timestamp     time.Time
}
```

### Chain Adapter Pattern

**Implementation Structure:**
```
pkg/chain/
├── interface.go          # ChainAdapter interface
├── factory.go            # GetAdapter(chainType) ChainAdapter
├── types.go              # Shared types (UnsignedTx, TxStatus, etc.)
├── ethereum/
│   ├── adapter.go        # EthereumAdapter implements ChainAdapter
│   ├── gas.go            # EIP-1559 gas handling
│   └── client.go         # go-ethereum client wrapper
├── bsc/
│   ├── adapter.go        # BSCAdapter implements ChainAdapter
│   └── client.go         # BSC-specific client
└── tron/
    ├── adapter.go        # TronAdapter implements ChainAdapter
    ├── resource.go       # Energy/bandwidth handling
    └── client.go         # gotron-sdk wrapper
```

**Factory Pattern:**
```go
// pkg/chain/factory.go
func GetAdapter(chainType ChainType, config ChainConfig) (ChainAdapter, error) {
    switch chainType {
    case ChainTypeTRON:
        return tron.NewAdapter(config.TronConfig)
    case ChainTypeBSC:
        return bsc.NewAdapter(config.BSCConfig)
    case ChainTypeEthereum:
        return ethereum.NewAdapter(config.EthConfig)
    default:
        return nil, ErrUnsupportedChain
    }
}
```

### Transaction Routing

**PayrollOrchestrator Implementation:**
```go
// pkg/payroll/orchestrator.go
type PayrollOrchestrator struct {
    adapters     map[ChainType]chain.ChainAdapter
    signer       signer.SignerClient
    statusTracker *StatusTracker
}

func (o *PayrollOrchestrator) SubmitMultiChainPayroll(ctx context.Context, batch PayrollBatch) (*MultiChainResult, error) {
    // Group payments by chain
    byChain := groupByChain(batch.Items)

    // Submit to each chain in parallel
    var wg sync.WaitGroup
    results := make(chan ChainResult, len(byChain))

    for chainType, items := range byChain {
        wg.Add(1)
        go func(ct ChainType, payItems []PayrollItem) {
            defer wg.Done()
            result := o.submitToChain(ctx, ct, payItems)
            results <- result
        }(chainType, items)
    }

    wg.Wait()
    close(results)

    return aggregateResults(results), nil
}
```

### State Synchronization

**Event-Driven State Updates:**
```go
// Consume ChainSync Kafka events
func (s *StateSync) HandleDepositConfirmed(event DepositConfirmedEvent) error {
    // Update internal balance
    return s.ledger.RecordDeposit(LedgerEntry{
        UserID:    event.UserID,
        Chain:     event.Chain,
        TxHash:    event.TxHash,
        Amount:    event.Amount,
        Status:    "confirmed",
        Timestamp: event.Timestamp,
    })
}

// Polling for transaction status (fallback)
func (s *StateSync) PollTransactionStatus(txHash string, chain ChainType) {
    adapter := s.adapters[chain]
    ticker := time.NewTicker(10 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-ticker.C:
            status, err := adapter.GetTransactionStatus(txHash)
            if err != nil {
                continue
            }
            if status.Status == Confirmed {
                s.updateInternalStatus(txHash, status)
                return
            }
        case <-s.stopCh:
            return
        }
    }
}
```

## 4.2 Separation of Concerns

### Module Boundaries

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        PAYROLL DOMAIN BOUNDARY                               │
│                                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                │
│  │ PayrollService │  │ TemplateService│  │ ScheduleService│                │
│  │                │  │                │  │                │                │
│  │ - BatchCreate  │  │ - CRUD         │  │ - CronTrigger  │                │
│  │ - BatchProcess │  │ - Versioning   │  │ - NextRun      │                │
│  │ - StatusQuery  │  │ - Validation   │  │ - Pause/Resume │                │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘                │
│          │                   │                   │                          │
└──────────┼───────────────────┼───────────────────┼──────────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        TREASURY DOMAIN BOUNDARY                              │
│                                                                             │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐                │
│  │TreasuryService │  │ HotWalletMgr   │  │ ColdWalletMgr  │                │
│  │                │  │                │  │                │                │
│  │ - VaultBalance │  │ - Replenish    │  │ - OfflineSign  │                │
│  │ - Allocation   │  │ - Rotate       │  │ - ImportSig    │                │
│  │ - Ledger       │  │ - Drain        │  │ - WatchOnly    │                │
│  └───────┬────────┘  └───────┬────────┘  └───────┬────────┘                │
│          │                   │                   │                          │
└──────────┼───────────────────┼───────────────────┼──────────────────────────┘
           │                   │                   │
           ▼                   ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                      BLOCKCHAIN DOMAIN BOUNDARY                              │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │                      BlockchainGatewayService                       │    │
│  │                                                                     │    │
│  │  - Chain-agnostic transaction submission                           │    │
│  │  - Balance queries across chains                                   │    │
│  │  - Fee estimation and optimization                                 │    │
│  │  - Transaction status tracking                                     │    │
│  └───────────────────────────────┬────────────────────────────────────┘    │
│                                  │                                          │
│        ┌─────────────────────────┼─────────────────────────┐               │
│        ▼                         ▼                         ▼               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                     │
│  │ TronAdapter  │  │  BSCAdapter  │  │  EthAdapter  │                     │
│  └──────────────┘  └──────────────┘  └──────────────┘                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       SIGNING DOMAIN BOUNDARY (ISOLATED)                     │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │                         SignerService                               │    │
│  │                                                                     │    │
│  │  - Seed management (encrypted)                                     │    │
│  │  - Address derivation                                              │    │
│  │  - Transaction signing                                             │    │
│  │  - NO external network access                                      │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
│  SECURITY: Internal gRPC only, no public access, separate network segment   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Shared Services

**Services used by both domains:**
- `NotificationService` - Email, SMS, push notifications
- `AuditService` - Centralized audit log collection
- `ConfigService` - Feature flags, system configuration

### API Boundaries

**Inter-Service Communication:**
```protobuf
// proto/payroll/v1/payroll.proto
service PayrollService {
    rpc CreateBatch(CreateBatchRequest) returns (CreateBatchResponse);
    rpc ProcessBatch(ProcessBatchRequest) returns (ProcessBatchResponse);
    rpc GetBatchStatus(GetBatchStatusRequest) returns (GetBatchStatusResponse);
}

// proto/treasury/v1/treasury.proto
service TreasuryService {
    rpc GetVaultBalance(GetVaultBalanceRequest) returns (GetVaultBalanceResponse);
    rpc RequestReplenishment(ReplenishmentRequest) returns (ReplenishmentResponse);
}

// proto/blockchain/v1/blockchain.proto
service BlockchainGatewayService {
    rpc SubmitTransaction(SubmitTxRequest) returns (SubmitTxResponse);
    rpc GetBalance(GetBalanceRequest) returns (GetBalanceResponse);
    rpc EstimateFee(EstimateFeeRequest) returns (EstimateFeeResponse);
}
```

## 4.3 Database Schema Changes

### New Tables Summary

| Table | Purpose | Tier |
|-------|---------|------|
| `payroll_schedules` | Recurring payment schedules | Tier 2 |
| `scheduled_executions` | Scheduled execution history | Tier 2 |
| `payroll_templates` | Saved payment configurations | Tier 2 |
| `template_history` | Template version history | Tier 2 |
| `employee_wallet_addresses` | Per-chain employee addresses | Tier 1 |
| `address_change_requests` | Address change approval | Tier 1 |
| `approval_chains` | Multi-level approval config | Tier 2 |
| `approval_chain_levels` | Approval level definitions | Tier 2 |
| `approval_chain_approvers` | Approvers per level | Tier 2 |
| `payroll_approvals` | Batch approval records | Tier 2 |
| `transaction_retries` | Retry attempt tracking | Tier 1 |
| `dead_letter_transactions` | Failed transaction queue | Tier 1 |
| `velocity_limits` | Transaction limits config | Tier 2 |
| `velocity_usage` | Limit usage tracking | Tier 2 |
| `multisig_configs` | Multi-sig configurations | Tier 3 |
| `multisig_signers` | Multi-sig signer list | Tier 3 |
| `multisig_requests` | Pending multi-sig requests | Tier 3 |
| `multisig_signatures` | Collected signatures | Tier 3 |
| `reconciliation_reports` | Reconciliation runs | Tier 2 |
| `reconciliation_discrepancies` | Found discrepancies | Tier 2 |
| `balance_alerts` | Alert instances | Tier 2 |
| `alert_notifications` | Alert delivery tracking | Tier 2 |
| `webhook_configs` | External webhook config | Tier 2 |
| `webhook_deliveries` | Webhook delivery history | Tier 2 |
| `audit_log_chain` | Immutable audit chain | Tier 1 |
| `hot_wallet_config` | Hot wallet settings | Tier 1 |
| `aggregated_balances` | Cross-chain balance view | Tier 2 |
| `derivation_registry` | HD path assignments | Tier 1 |
| `employee_payment_preferences` | Employee chain/token prefs | Tier 2 |
| `ledger_accounts` | Account hierarchy | Tier 3 |
| `permissions` | Granular permissions | Tier 2 |
| `role_permissions` | Role-permission mapping | Tier 2 |

### Schema Modifications

```sql
-- Enhance payroll_items with retry and chain info
ALTER TABLE payroll_items
ADD COLUMN chain ENUM('TRON', 'BSC', 'ETH') NOT NULL DEFAULT 'TRON',
ADD COLUMN token_address VARCHAR(64),
ADD COLUMN retry_count INT DEFAULT 0,
ADD COLUMN last_retry_at DATETIME,
ADD COLUMN gas_used DECIMAL(20, 8),
ADD COLUMN actual_fee DECIMAL(20, 8),
ADD COLUMN confirmation_depth INT DEFAULT 0;

-- Enhance ledger_entries for double-entry
ALTER TABLE ledger_entries
ADD COLUMN debit_account_id BIGINT,
ADD COLUMN credit_account_id BIGINT,
ADD COLUMN chain ENUM('TRON', 'BSC', 'ETH');

-- Enhance users for employee wallet addresses
-- (using separate employee_wallet_addresses table instead)
```

### Indexing Strategy

```sql
-- Transaction history performance
CREATE INDEX idx_payroll_items_status_chain ON payroll_items(status, chain, created_at);
CREATE INDEX idx_payroll_items_employee ON payroll_items(employee_id, created_at);

-- Batch processing
CREATE INDEX idx_payroll_batches_status_scheduled ON payroll_batches(status, scheduled_at);

-- Retry management
CREATE INDEX idx_transaction_retries_status ON transaction_retries(status, attempted_at);

-- Audit queries
CREATE INDEX idx_audit_log_chain_user ON audit_log_chain(user_id, timestamp);
CREATE INDEX idx_audit_log_chain_resource ON audit_log_chain(resource_type, resource_id, timestamp);

-- Balance monitoring
CREATE INDEX idx_aggregated_balances_wallet ON aggregated_balances(wallet_type, chain);

-- Address lookup
CREATE INDEX idx_employee_addresses_address ON employee_wallet_addresses(address);
```

### Data Partitioning

```sql
-- Partition transaction_retries by month
ALTER TABLE transaction_retries
PARTITION BY RANGE (YEAR(attempted_at) * 100 + MONTH(attempted_at)) (
    PARTITION p202512 VALUES LESS THAN (202601),
    PARTITION p202601 VALUES LESS THAN (202602),
    -- ... add partitions as needed
    PARTITION pmax VALUES LESS THAN MAXVALUE
);

-- Archive old payroll data (after 2 years)
CREATE TABLE payroll_items_archive LIKE payroll_items;
-- Scheduled job to move old data
```

## 4.4 Service Architecture

### Microservices vs Monolith Recommendation

**Recommendation: Modular Monolith initially, extract as needed**

**Rationale:**
1. Current system is already Go-Zero microservices - maintain pattern
2. Payroll is a new domain - start as module within Admin Service
3. Extract to separate service if: independent scaling, separate team, different deployment cycle

**Proposed Service Structure:**
```
services/
├── api-gateway/          # Existing
├── business/             # Existing - user operations
├── chainrpc/             # Existing - blockchain operations
├── chainsync/            # Existing - blockchain monitoring
├── signer/               # Existing - key management
├── admin/                # Existing - enhance with payroll
│   ├── payroll/          # NEW: Payroll module
│   │   ├── batch.go
│   │   ├── schedule.go
│   │   ├── template.go
│   │   └── approval.go
│   ├── treasury/         # NEW: Treasury module
│   │   ├── vault.go
│   │   ├── hot_wallet.go
│   │   └── reconciliation.go
│   └── reporting/        # NEW: Reporting module
└── notification/         # Extract if high volume
```

### Inter-Service Communication

**Synchronous (gRPC):**
- Admin → ChainRPC: Balance queries, transaction submission
- Admin → Signer: Address generation (for employee addresses)
- Business → Admin: Employee data lookup

**Asynchronous (Kafka):**
- ChainSync → Admin: Transaction confirmed events
- Admin → Notification: Send alerts
- Admin → Audit: Log events

**Event Schema:**
```json
{
  "event_id": "EVT-20251210-ABC123",
  "event_type": "payroll.batch.processed",
  "timestamp": "2025-12-10T09:00:00Z",
  "payload": {
    "batch_id": "PAY-20251210-001",
    "total_items": 150,
    "success_count": 148,
    "failed_count": 2,
    "total_amount": "150000.00"
  },
  "metadata": {
    "operator_id": "10001",
    "correlation_id": "REQ-XYZ"
  }
}
```

## 4.5 Scalability Design

### Batch Processing Scalability

```go
type BatchWorker struct {
    workerID      int
    inputCh       chan PayrollItem
    resultCh      chan ProcessResult
    adapters      map[ChainType]chain.ChainAdapter
    signer        signer.SignerClient
    concurrency   int  // transactions per chain concurrently
}

func (s *BatchService) ProcessLargeBatch(ctx context.Context, batch PayrollBatch) error {
    // Chunk batch into worker-sized pieces
    chunks := chunkItems(batch.Items, s.chunkSize)  // 100 items per chunk

    // Create worker pool
    pool := NewWorkerPool(s.workerCount)  // 10 workers

    // Submit chunks
    for _, chunk := range chunks {
        pool.Submit(func() error {
            return s.processChunk(ctx, chunk)
        })
    }

    // Wait with timeout
    return pool.Wait(ctx)
}
```

### Horizontal Scaling

**Stateless Services (can scale horizontally):**
- API Gateway (load balanced)
- Admin Service (read replicas for queries)
- ChainRPC Service (multiple instances per chain)
- Notification Service

**Stateful Services (careful scaling):**
- Signer Service: Single instance preferred (key consistency)
- ChainSync Service: Partition by chain (one instance per chain)

### Caching Strategy

```go
// Redis caching configuration
type CacheConfig struct {
    // Balance cache - 30 second TTL
    BalanceTTL: 30 * time.Second,

    // Gas price cache - 10 second TTL
    GasPriceTTL: 10 * time.Second,

    // Exchange rate cache - 60 second TTL
    ExchangeRateTTL: 60 * time.Second,

    // Employee address cache - 5 minute TTL
    EmployeeAddressTTL: 5 * time.Minute,

    // Payroll template cache - 10 minute TTL
    TemplateTTL: 10 * time.Minute,
}

// Cache invalidation events
func (c *CacheManager) InvalidateOnEvent(event Event) {
    switch event.Type {
    case "balance.changed":
        c.Delete(fmt.Sprintf("balance:%s:%s", event.Chain, event.Address))
    case "template.updated":
        c.Delete(fmt.Sprintf("template:%s", event.TemplateID))
    }
}
```

### Queue Management

```go
// Queue configuration
type QueueConfig struct {
    // Transaction submission queue
    TxSubmission: QueueSettings{
        Name:        "payroll.tx.submit",
        Concurrency: 10,
        MaxRetries:  3,
        RetryDelay:  time.Minute,
    },

    // Transaction monitoring queue
    TxMonitor: QueueSettings{
        Name:        "payroll.tx.monitor",
        Concurrency: 50,
        MaxRetries:  100,  // Keep monitoring for hours
        RetryDelay:  10 * time.Second,
    },

    // High priority queue for urgent payments
    HighPriority: QueueSettings{
        Name:        "payroll.tx.priority",
        Concurrency: 5,
        MaxRetries:  5,
        RetryDelay:  30 * time.Second,
    },
}
```

## 4.6 Security Architecture

### Key Management Service (KMS)

**Recommendation: Cloud KMS for key encryption keys, local HSM for high-value operations**

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Key Hierarchy                                 │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Master Key (AWS KMS / Google Cloud KMS / HashiCorp Vault)  │   │
│  │  - Never leaves KMS                                         │   │
│  │  - Used to encrypt/decrypt DEKs                             │   │
│  └─────────────────────────────┬───────────────────────────────┘   │
│                                │                                    │
│                                ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Data Encryption Keys (DEKs) - Stored encrypted in DB       │   │
│  │  - Seed encryption keys                                     │   │
│  │  - Backup encryption keys                                   │   │
│  └─────────────────────────────┬───────────────────────────────┘   │
│                                │                                    │
│                                ▼                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │  Protected Data                                              │   │
│  │  - master_seeds.seed_encrypted                              │   │
│  │  - Backup files                                             │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Encryption at Rest

| Data Type | Encryption | Storage |
|-----------|------------|---------|
| Master seeds | AES-256-GCM + PBKDF2 | MariaDB (encrypted column) |
| Backup mnemonics | AES-256-GCM | Secure file storage |
| Sensitive PII | AES-256-GCM | MariaDB (encrypted column) |
| Audit logs | Transparent encryption | MariaDB TDE |
| Redis data | Redis encryption at rest | Redis Enterprise |

### Encryption in Transit

- All API traffic: TLS 1.3
- Service-to-service: mTLS (mutual TLS)
- Database connections: TLS required
- Redis connections: TLS required

### Signing Service Isolation

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRODUCTION NETWORK                                │
│                                                                     │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐             │
│  │ API Gateway │    │   Admin     │    │  ChainRPC   │             │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘             │
│         │                  │                  │                     │
│         └──────────────────┴──────────────────┘                     │
│                            │                                        │
│                     Internal Load Balancer                          │
│                            │                                        │
└────────────────────────────┼────────────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    │  Firewall/SG    │
                    │  - Only gRPC    │
                    │  - IP whitelist │
                    └────────┬────────┘
                             │
┌────────────────────────────┼────────────────────────────────────────┐
│                    ISOLATED SIGNING NETWORK                          │
│                                                                     │
│         ┌──────────────────────────────────────────┐               │
│         │           Signer Service                  │               │
│         │                                          │               │
│         │  - No outbound internet access           │               │
│         │  - Only accepts internal gRPC            │               │
│         │  - HSM integration for cold keys         │               │
│         │  - Audit logging all operations          │               │
│         └──────────────────────────────────────────┘               │
│                            │                                        │
│                    ┌───────┴───────┐                               │
│                    │   HSM/TEE     │                               │
│                    │ (Optional)    │                               │
│                    └───────────────┘                               │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Secrets Management

**Recommendation: HashiCorp Vault or cloud-native secrets manager**

```go
// Secrets to manage
type SecretsConfig struct {
    // Database credentials
    DBHost     string `vault:"secret/data/db#host"`
    DBUser     string `vault:"secret/data/db#username"`
    DBPassword string `vault:"secret/data/db#password"`

    // Redis credentials
    RedisPassword string `vault:"secret/data/redis#password"`

    // JWT signing key
    JWTSecret string `vault:"secret/data/jwt#secret"`

    // Encryption password for seeds
    SeedEncryptionPassword string `vault:"secret/data/signer#encryption_password"`

    // RPC provider API keys
    InfuraAPIKey    string `vault:"secret/data/rpc#infura_key"`
    AlchemyAPIKey   string `vault:"secret/data/rpc#alchemy_key"`
    QuickNodeAPIKey string `vault:"secret/data/rpc#quicknode_key"`

    // Notification service credentials
    SMTPPassword    string `vault:"secret/data/notification#smtp_password"`
    TwilioAuthToken string `vault:"secret/data/notification#twilio_token"`
}
```

---

# 5. Implementation Roadmap

## Phase 1: MVP (Weeks 1-8)

**Objective:** Basic multi-chain payroll disbursement capability

### Week 1-2: Foundation
- [ ] Employee wallet address management (CRUD, validation)
- [ ] Hot wallet configuration per chain
- [ ] Enhanced audit logging (immutable chain)

### Week 3-4: Multi-Chain Batch Processing
- [ ] Chain adapter abstraction layer
- [ ] PayrollOrchestrator implementation
- [ ] Parallel multi-chain transaction submission
- [ ] Nonce management per chain

### Week 5-6: Status & Retry
- [ ] Real-time payment status tracking
- [ ] Transaction retry mechanism
- [ ] Dead letter queue for failed transactions
- [ ] Basic dashboard updates

### Week 7-8: Testing & Hardening
- [ ] Integration testing all chains
- [ ] Load testing (1000+ recipients)
- [ ] Security audit
- [ ] Documentation

**Deliverables:**
- Multi-chain payroll batch processing
- Employee address management
- Transaction retry and status tracking
- Enhanced audit logging

**Success Criteria:**
- Successfully process 1000-recipient batch across 3 chains
- 99% transaction success rate
- Complete audit trail for all operations

**Team Composition:**
- 2 Senior Backend Engineers (Go)
- 1 Blockchain Engineer (multi-chain experience)
- 1 QA Engineer
- 0.5 DevOps Engineer

---

## Phase 2: Enhancement (Weeks 9-16)

**Objective:** Operational efficiency and compliance features

### Week 9-10: Scheduling & Templates
- [ ] PayrollScheduler implementation
- [ ] Cron-based recurring payments
- [ ] Payment template management
- [ ] Template versioning

### Week 11-12: Advanced Approvals
- [ ] Multi-level approval workflow
- [ ] Approval delegation
- [ ] Time-based auto-approval
- [ ] Approval audit trail

### Week 13-14: Optimization & Monitoring
- [ ] Gas fee optimization
- [ ] Aggregated balance dashboard
- [ ] Velocity limits
- [ ] Balance alerts with escalation

### Week 15-16: Integration & Reconciliation
- [ ] Webhook notifications
- [ ] Basic reconciliation tools
- [ ] Enhanced reporting
- [ ] ERP integration stubs

**Deliverables:**
- Scheduled/recurring payroll
- Payment templates
- Multi-level approvals
- Gas optimization
- Reconciliation tools

**Success Criteria:**
- Automated bi-weekly payroll execution
- 20% reduction in manual operations
- Zero reconciliation discrepancies
- SOX-compliant approval workflow

**Team Composition:**
- 2 Senior Backend Engineers
- 1 Frontend Engineer (dashboard)
- 1 QA Engineer
- 0.5 DevOps Engineer

---

## Phase 3: Optimization (Weeks 17-24)

**Objective:** Advanced security and scalability

### Week 17-18: Multi-Signature
- [ ] Multi-sig configuration
- [ ] Signature collection workflow
- [ ] Gnosis Safe integration (ETH/BSC)
- [ ] TRON multi-sig contract

### Week 19-20: Advanced Security
- [ ] Hardware wallet integration
- [ ] Enhanced RBAC
- [ ] Anomaly detection
- [ ] Security dashboard

### Week 21-22: Scalability
- [ ] Horizontal scaling implementation
- [ ] Database partitioning
- [ ] Cache optimization
- [ ] Performance tuning

### Week 23-24: Future Prep
- [ ] Layer 2 preparation (interfaces)
- [ ] Advanced reporting
- [ ] Employee self-service portal
- [ ] Documentation & training

**Deliverables:**
- Multi-signature support
- Hardware wallet integration
- Production-grade scalability
- Employee portal

**Success Criteria:**
- Multi-sig operational for >$100K transactions
- Support 10,000+ employees
- 99.9% uptime
- Full employee self-service

**Team Composition:**
- 2 Senior Backend Engineers
- 1 Security Engineer
- 1 Frontend Engineer
- 1 QA Engineer
- 1 DevOps/SRE Engineer

---

## Dependencies Between Phases

```
Phase 1 (MVP)
├── Employee Address Management ─────────────────────────────────┐
├── Multi-Chain Batch Processing ─────┐                          │
├── Transaction Retry ────────────────┼──────────────────────────│
└── Audit Logging ────────────────────│                          │
                                      │                          │
Phase 2 (Enhancement)                 ▼                          │
├── Scheduling ◄──────────────── Batch Processing               │
├── Templates ◄───────────────── Batch Processing               │
├── Multi-Level Approval ◄────── Audit Logging                  │
├── Gas Optimization ◄────────── Multi-Chain Batch              │
├── Reconciliation ◄──────────── Audit Logging                  │
└── Balance Alerts ◄──────────── Vault Monitoring               │
                                      │                          │
Phase 3 (Optimization)                ▼                          │
├── Multi-Sig ◄───────────────── Approval Workflow              │
├── Hardware Wallet ◄─────────── Multi-Sig                      │
├── Scalability ◄─────────────── All Phase 1 & 2               │
└── Employee Portal ◄─────────── Address Management ◄───────────┘
```

---

# 6. Critical Files Reference

## Existing Files to Modify

| File/Location | Modification |
|---------------|--------------|
| `admin-api/03-薪资发放.md` | Enhance with multi-chain, retry, scheduling |
| `admin-api/07-账本与Vault.md` | Add reconciliation, enhanced alerts |
| `admin-api/99-数据模型与配置.md` | Add new tables |
| `wallet-core/srds/03-chainrpc-service/02-交易构建.md` | Enhance batch building |
| `wallet-core/srds/05-signer-service/02-地址生成.md` | Employee address generation |
| `wallet-core/srds/06-common-infrastructure/06-错误码.md` | Add payroll error codes |

## New Files to Create

| File/Location | Purpose |
|---------------|---------|
| `pkg/chain/interface.go` | Chain adapter interface |
| `pkg/chain/factory.go` | Adapter factory |
| `pkg/chain/ethereum/adapter.go` | Ethereum implementation |
| `pkg/chain/bsc/adapter.go` | BSC implementation |
| `pkg/chain/tron/adapter.go` | TRON implementation |
| `services/admin/payroll/orchestrator.go` | Multi-chain orchestration |
| `services/admin/payroll/scheduler.go` | Recurring payments |
| `services/admin/payroll/template.go` | Template management |
| `services/admin/treasury/reconciliation.go` | Reconciliation engine |
| `migrations/20251210_payroll_extensions.sql` | Database migrations |

---

# 7. Summary

This plan provides a comprehensive analysis of the Internal-Wallet system and a detailed roadmap for extending it to support enterprise cryptocurrency payroll disbursement across TRON, BSC, and Ethereum networks.

**Key Findings:**
1. The existing system has strong foundations with 107+ RPC functions, multi-chain support, and enterprise-grade architecture
2. Payroll-specific features already exist but need enhancement for multi-chain, scheduling, and advanced approvals
3. The Signer Service provides secure key management but needs network isolation in production
4. The modular microservices architecture supports the proposed extensions

**Critical Success Factors:**
1. Chain adapter abstraction for maintainable multi-chain support
2. Robust transaction retry mechanism for blockchain reliability
3. Comprehensive audit logging for compliance
4. Secure key management with proper isolation

**Risk Mitigation:**
1. Start with modular monolith, extract services as needed
2. Extensive testing on testnets before mainnet deployment
3. Security audit before each phase launch
4. Gradual rollout with feature flags
