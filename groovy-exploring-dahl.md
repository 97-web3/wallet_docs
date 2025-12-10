# Enterprise Internal Cryptocurrency Wallet Platform - Admin Backend System Feature Specification V1.0

---

## Executive Summary

This document provides a comprehensive feature specification for the Admin Management Backend System of an enterprise internal cryptocurrency wallet platform. The platform operates a dual-wallet architecture supporting both Web2 (custodial) and Web3 (non-custodial) wallets across TRX, BNB, and ETH blockchains.

**Key Admin Roles:** Administrator, Finance
**Primary Functions:** User management, payroll distribution, withdrawal approval, vault management, swap configuration, compliance monitoring

---

## 1. Dashboard & Monitoring

### 1.1 Real-Time Metrics Dashboard

#### 1.1.1 Total Deposit/Withdrawal Statistics Widget

**Feature Name:** Total Deposit/Withdrawal Statistics Widget

**Description:** Displays aggregated deposit and withdrawal volumes across all supported chains (TRX/BNB/ETH) with real-time updates. Shows total counts, total values in USDT equivalent, and trend indicators comparing to previous periods.

**Business Value:**
- Provides immediate visibility into platform fund flows for finance team oversight
- Enables quick identification of unusual activity patterns
- Supports daily operational decision-making and liquidity planning
- ROI: Reduces manual report generation time by 80%

**Target User Roles:**
- **Finance:** Primary user - monitors daily flows, identifies anomalies, reports to management
- **Administrator:** Secondary user - overview for system health assessment

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Full transaction visibility including internal transfers (划转), deposits to child addresses, and withdrawals from hot wallet. Admin has complete transaction data access.
- **Web3 (Non-custodial):** Read-only monitoring of linked Web3 addresses. Only displays transactions for addresses bound to platform accounts. Limited to on-chain observable data.
- **Linked Wallets:** Aggregated view showing both Web2 balance and linked Web3 balance for users with bound addresses.

**Security Considerations:**
- Authentication: Requires valid admin session with Finance or Administrator role
- Data encryption: TLS for transit, AES-256 for sensitive data at rest
- Audit logging: All dashboard views logged with timestamp and user ID
- Rate limiting: Prevent data scraping via excessive refresh requests

**Integration Requirements:**
- Blockchain nodes: Web3.js/ethers.js for ETH/BNB, TronWeb for TRX
- Price oracle: DeFiLlama/CoinGecko API for USD conversion rates
- Internal database: Read replica for real-time queries without impacting primary DB
- Cache layer: Redis for sub-second dashboard refresh

**Technical Architecture Notes:**
- Pattern: CQRS with dedicated read model for dashboard metrics
- Scalability: Pre-aggregated metrics updated via event streaming
- Performance: <500ms page load, <2s for full refresh
- Data consistency: Eventually consistent (acceptable 30s lag)

**Priority Level:** Critical
- **Justification:** PRD Section 四.1 explicitly requires "总充值、总提现统计" as core Dashboard functionality

**Dependencies:**
- Transaction processing system must emit events for aggregation
- Price oracle integration for USDT conversion

**Success Criteria:**
- Dashboard loads within 500ms
- Metrics update within 30 seconds of transaction confirmation
- 99.9% accuracy compared to actual ledger totals

---

#### 1.1.2 Web2 Total Custodial Balance Widget

**Feature Name:** Web2 Total Custodial Balance Widget

**Description:** Displays the total balance held across all Web2 custodial wallets, broken down by token (USDT, TRX, BNB, ETH) with USD equivalent values. Shows aggregate of all user child address balances managed by the platform.

**Business Value:**
- Critical for financial auditing and reconciliation
- Enables compliance reporting on custodial asset volumes
- Supports capacity planning for hot/cold wallet distribution
- ROI: Eliminates 4+ hours of daily manual balance aggregation

**Target User Roles:**
- **Finance:** Primary user - daily reconciliation, audit preparation
- **Administrator:** Oversight and system configuration

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Full balance visibility and control. Balance represents off-chain ledger totals backed by vault holdings.
- **Web3 (Non-custodial):** Not applicable - Web3 balances are user-controlled and not part of custodial totals.
- **Linked Wallets:** Displays only the Web2 portion; linked Web3 balances shown separately.

**Security Considerations:**
- Access restricted to Finance and Administrator roles only
- Balance data considered highly sensitive financial information
- Audit trail for every balance view access
- No balance modification through this widget (read-only)

**Integration Requirements:**
- Off-chain ledger database for Web2 balance totals
- Blockchain nodes for on-chain vault balance verification
- Reconciliation service comparing ledger vs. actual balances

**Technical Architecture Notes:**
- Pattern: Materialized view updated on transaction commits
- Scalability: Index on token type and chain for fast aggregation
- Performance: <1s query response for total balance
- Data consistency: Strong consistency required - balance must match ledger

**Priority Level:** Critical
- **Justification:** PRD Section 四.1 explicitly requires "Web2 钱包总托管余额"

**Dependencies:**
- Off-chain ledger system (Feature 8.1)
- User wallet allocation system

**Success Criteria:**
- Balance accuracy: 100% match with audited ledger
- Real-time updates within 5 seconds of transaction completion
- Historical balance snapshots available for any date

---

#### 1.1.3 Enterprise Vault Balance Widget

**Feature Name:** Enterprise Vault Balance Widget

**Description:** Displays real-time balances of enterprise hot wallets (vaults) across all supported tokens (TRX/BNB/ETH/USDT) on their respective chains. Includes available balance, pending outflows, and reserved amounts.

**Business Value:**
- Ensures operational liquidity for withdrawal processing
- Prevents failed withdrawals due to insufficient vault balance
- Enables proactive fund management and rebalancing
- ROI: Prevents potential losses from withdrawal delays

**Target User Roles:**
- **Finance:** Monitors vault health, initiates rebalancing, manages cold storage transfers
- **Administrator:** System oversight, emergency response

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Vault directly supports Web2 withdrawals and payroll distributions. Admin can initiate vault operations.
- **Web3 (Non-custodial):** No vault involvement - users control their own funds.
- **Linked Wallets:** Vault funds cover linked address withdrawal requests from Web2 accounts.

**Security Considerations:**
- HSM integration for vault private key operations
- Multi-signature requirements for large transfers
- Real-time alerting on unexpected balance changes
- Audit logging of all vault balance queries

**Integration Requirements:**
- Blockchain nodes: Direct RPC calls for balance queries
- Hot wallet service: Internal service managing vault operations
- Cold storage system: Integration for rebalancing operations
- Alert service: Push notifications for threshold breaches

**Technical Architecture Notes:**
- Pattern: Event sourcing for complete vault operation history
- Scalability: Separate vault per chain to isolate operations
- Performance: Balance query <200ms, refresh every 10 seconds
- Data consistency: Strong consistency - must reflect on-chain state

**Priority Level:** Critical
- **Justification:** PRD Section 四.1 explicitly requires "企业 vault 余额（TRX / BNB / ETH / USDT）"

**Dependencies:**
- Blockchain node infrastructure
- HSM/key management system

**Success Criteria:**
- Balance reflects on-chain state within 1 block confirmation
- 100% accuracy verified against blockchain explorers
- Zero discrepancies in monthly audits

---

#### 1.1.4 Balance Threshold Alert System

**Feature Name:** Balance Threshold Alert System

**Description:** Configurable alert system that monitors vault balances and triggers notifications when balances fall below defined thresholds. Supports per-chain, per-token threshold configuration with role-based alert routing.

**Business Value:**
- Prevents operational disruption from insufficient liquidity
- Enables proactive fund management before critical shortages
- Reduces manual monitoring burden
- ROI: Prevents potential withdrawal delays and user complaints

**Target User Roles:**
- **Finance:** Configures thresholds, receives alerts, takes corrective action
- **Administrator:** Configures alert recipients and escalation rules

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Alerts based on vault balance vs. pending Web2 withdrawal queue
- **Web3 (Non-custodial):** Not applicable - no platform-controlled funds
- **Linked Wallets:** Include linked address withdrawal requests in threshold calculations

**Security Considerations:**
- Alert configuration requires Finance role minimum
- Alert data (balance levels) is sensitive financial information
- Secure notification channels (encrypted email, authenticated push)
- Audit log of all threshold changes

**Integration Requirements:**
- Notification service: Email, SMS, webhook integrations
- Vault monitoring service: Real-time balance tracking
- Alert rule engine: Configurable threshold and routing rules
- Escalation service: Automatic escalation for unacknowledged alerts

**Technical Architecture Notes:**
- Pattern: Observer pattern with configurable threshold rules
- Scalability: Event-driven alerts, no polling
- Performance: Alert triggered within 30 seconds of threshold breach
- Data consistency: Alert based on confirmed on-chain balance

**Priority Level:** Critical
- **Justification:** PRD Section 四.1 explicitly requires "余额低于阈值发预警（角色及阈值可配置）"

**Dependencies:**
- Vault Balance Widget (1.1.3)
- Role-Based Access Control (Feature 12)
- Notification service infrastructure

**Success Criteria:**
- Alerts delivered within 60 seconds of threshold breach
- Zero missed alerts in 12-month period
- 95% alert acknowledgment within 30 minutes

---

### 1.2 Multi-Chain Status Monitoring

#### 1.2.1 Blockchain Network Health Dashboard

**Feature Name:** Blockchain Network Health Dashboard

**Description:** Displays real-time health status of connected blockchain networks (TRX, BNB, ETH) including node connectivity, block height, sync status, and network congestion metrics.

**Business Value:**
- Enables proactive identification of network issues affecting transactions
- Supports informed decision-making on transaction timing
- Reduces support inquiries about delayed transactions
- ROI: Improved operational awareness and faster incident response

**Target User Roles:**
- **Administrator:** Primary user - monitors infrastructure health
- **Finance:** Awareness of network conditions affecting operations

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Network health directly impacts withdrawal processing and deposit crediting
- **Web3 (Non-custodial):** Network health affects user transaction visibility
- **Linked Wallets:** Same implications as Web2/Web3 depending on transaction type

**Security Considerations:**
- Node endpoint information is sensitive infrastructure data
- Read-only monitoring - no configuration changes through this dashboard
- Audit logging of dashboard access

**Integration Requirements:**
- Blockchain RPC nodes: eth_syncing, eth_blockNumber, etc.
- TronWeb: Similar health check endpoints
- Uptime monitoring: External service for node availability
- Gas price oracles: For network congestion assessment

**Technical Architecture Notes:**
- Pattern: Health check aggregator with circuit breaker
- Scalability: Distributed health checks across regions
- Performance: Status updates every 15 seconds
- Data consistency: Eventually consistent acceptable

**Priority Level:** High
- **Justification:** Essential for operational stability though not explicitly mentioned in PRD

**Dependencies:**
- Blockchain node infrastructure
- Network monitoring service

**Success Criteria:**
- Network issues detected within 30 seconds
- False positive rate <5%
- Dashboard availability 99.9%

---

## 2. User Management

### 2.1 User Administration

#### 2.1.1 User List and Search

**Feature Name:** User List and Search

**Description:** Comprehensive user directory displaying all registered accounts with search and filter capabilities by UID, phone number, email, and identity type (employee/non-employee). Supports pagination, sorting, and bulk selection.

**Business Value:**
- Enables efficient user lookup for support and administration
- Facilitates compliance investigations and audits
- Supports operational workflows requiring user identification
- ROI: Reduces average user lookup time from minutes to seconds

**Target User Roles:**
- **Administrator:** Full access to all user data, can modify user status
- **Finance:** Read access for transaction-related user lookups

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Shows Web2 wallet status, balance, and assigned child address
- **Web3 (Non-custodial):** Shows linked Web3 address if bound (read-only)
- **Linked Wallets:** Indicates binding status between Web2 and Web3 wallets

**Security Considerations:**
- PII data (phone, email) requires appropriate access level
- Search queries logged for audit purposes
- Data masking for partial display (e.g., email: u***@domain.com)
- Export restrictions based on role

**Integration Requirements:**
- User database: Primary user data store
- Authentication service: Validate admin session
- Audit service: Log all search and access events

**Technical Architecture Notes:**
- Pattern: Repository pattern with search specification
- Scalability: Elasticsearch for full-text search, database for exact match
- Performance: <500ms search response, pagination required >100 results
- Data consistency: Read from replica acceptable

**Priority Level:** Critical
- **Justification:** PRD Section 四.3 explicitly requires "用户列表：uid、手机号、邮箱、身份（员工/非员工）"

**Dependencies:**
- User registration system
- Authentication system

**Success Criteria:**
- Search returns results within 500ms
- Supports 100,000+ user records without degradation
- 100% accuracy in search results

---

#### 2.1.2 User Account Status Management

**Feature Name:** User Account Status Management

**Description:** Administrative controls for managing user account states including freeze (temporary suspension), unfreeze (restore access), and employment termination handling. Each action requires justification and is logged for audit.

**Business Value:**
- Enables rapid response to security incidents or policy violations
- Supports HR processes for employee offboarding
- Provides compliance capability for regulatory requirements
- ROI: Reduces security incident response time significantly

**Target User Roles:**
- **Administrator:** Can freeze, unfreeze, and process terminations
- **Finance:** Read-only view of user status history

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Freeze action locks Web2 wallet - prevents deposits, withdrawals, and transfers
- **Web3 (Non-custodial):** Cannot freeze Web3 wallet (user-controlled), but can restrict platform features for linked Web3
- **Linked Wallets:** Freeze status applies to the linkage - prevents linked address withdrawal exemption

**Security Considerations:**
- Freeze/unfreeze actions require Administrator role
- Dual approval for termination processing (recommended)
- Complete audit trail with timestamp, actor, and justification
- Immediate effect on active sessions (force logout on freeze)

**Integration Requirements:**
- User database: Status field updates
- Session service: Invalidate active sessions on freeze
- Notification service: Alert user of status change
- HR system: Integration for employment status changes

**Technical Architecture Notes:**
- Pattern: State machine for account lifecycle
- Scalability: Status check on every API request (cached)
- Performance: Status change effective within 5 seconds
- Data consistency: Strong consistency for status changes

**Priority Level:** Critical
- **Justification:** PRD Section 四.3 requires "冻结、解冻、离职处理"

**Dependencies:**
- User List (2.1.1)
- Session management system
- Audit logging system

**Success Criteria:**
- Status change takes effect within 5 seconds
- 100% of status changes have complete audit records
- Zero unauthorized freeze/unfreeze actions

---

#### 2.1.3 Employee/Non-Employee Identity Management

**Feature Name:** Employee/Non-Employee Identity Management

**Description:** Capability to assign and modify user identity classification (employee vs. non-employee) with role tagging. Ensures proper segregation of employee accounts for independent management and applies appropriate privilege levels.

**Business Value:**
- Enables employee-specific features like payroll distribution
- Supports compliance with employment-related regulations
- Facilitates proper access control and audit trails
- ROI: Streamlines HR-related crypto operations

**Target User Roles:**
- **Administrator:** Can modify user identity type and role tags
- **Finance:** Read access to verify employee status for payroll

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Employee status enables salary receipt capability, may affect withdrawal limits
- **Web3 (Non-custodial):** Identity does not affect Web3 functionality (user-controlled)
- **Linked Wallets:** Employee linked address may have different approval thresholds

**Security Considerations:**
- Identity changes require Administrator role
- Integration with HR system for employment verification
- Audit trail for all identity changes
- Cannot self-modify identity type

**Integration Requirements:**
- HR system: Sync employee status changes
- User database: Identity type and role tag fields
- Audit service: Log all identity changes

**Technical Architecture Notes:**
- Pattern: Role-based with attribute-based access control
- Scalability: Identity check cached per session
- Performance: Identity lookup <100ms
- Data consistency: Strong consistency for identity changes

**Priority Level:** Critical
- **Justification:** PRD Section 四.3 requires "角色标签管理，确保员工账户独立管理"

**Dependencies:**
- User List (2.1.1)
- RBAC system (Feature 12)
- HR system integration (optional)

**Success Criteria:**
- Identity changes reflected immediately in all systems
- 100% sync accuracy with HR system (if integrated)
- Complete audit trail for all identity changes

---

### 2.2 Blacklist Management

#### 2.2.1 UID-Based Account Blacklist

**Feature Name:** UID-Based Account Blacklist

**Description:** System for adding user accounts to a blacklist by UID, preventing login, forcing logout of active sessions, and blocking all platform operations. Includes reason tracking, effective dates, and appeal workflow support.

**Business Value:**
- Critical security control for fraud prevention
- Compliance requirement for regulatory blocking
- Protects platform from malicious actors
- ROI: Prevents financial losses from blocked bad actors

**Target User Roles:**
- **Administrator:** Can add/remove users from blacklist
- **Finance:** Can view blacklist for transaction context

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Complete platform lockout - cannot access Web2 wallet
- **Web3 (Non-custodial):** Prevents binding/unbinding, but cannot control user's Web3 wallet directly
- **Linked Wallets:** Blacklisting prevents linked address exemptions

**Security Considerations:**
- Blacklist management requires Administrator role
- Immediate session invalidation on blacklisting
- Cannot blacklist self or other administrators
- Appeals must go through separate secure channel

**Integration Requirements:**
- User database: Blacklist status field
- Session service: Immediate session termination
- Login service: Block authentication attempts
- Audit service: Complete action logging

**Technical Architecture Notes:**
- Pattern: Bloom filter for fast blacklist check, database for authoritative source
- Scalability: Blacklist loaded in memory for O(1) lookup
- Performance: Blacklist check <10ms on every request
- Data consistency: Strong consistency for blacklist changes

**Priority Level:** Critical
- **Justification:** PRD Section 三 "特殊场景处理" explicitly requires blacklist handling

**Dependencies:**
- User Management (2.1)
- Session management system

**Success Criteria:**
- Blacklisted user blocked within 5 seconds
- Zero bypasses of blacklist
- 100% audit trail coverage

---

## 3. Wallet Management

### 3.1 Web2 Wallet (Custodial) Administration

#### 3.1.1 Enterprise Vault/Hot Wallet Management

**Feature Name:** Enterprise Vault/Hot Wallet Management

**Description:** Administrative interface for managing the enterprise master wallets (hot wallets/vaults) that hold pooled user funds. Enables viewing balances, initiating transfers to/from cold storage, and monitoring vault operations.

**Business Value:**
- Central control point for custodial fund security
- Enables optimal fund distribution between hot and cold storage
- Supports compliance with custody requirements
- ROI: Reduces risk exposure through proper fund segregation

**Target User Roles:**
- **Finance:** View vault balances, initiate approved transfers
- **Administrator:** Full vault configuration access

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - vault holds all Web2 wallet funds
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** Vault funds service linked address withdrawals

**Security Considerations:**
- HSM integration for private key protection
- Multi-signature requirements for outbound transfers
- Air-gapped cold storage for majority of funds
- 24/7 monitoring for unauthorized transactions
- Disaster recovery and key backup procedures

**Integration Requirements:**
- HSM: Hardware security module for key operations
- Cold storage: Offline signing system integration
- Blockchain nodes: Transaction submission and monitoring
- Alert system: Real-time anomaly detection

**Technical Architecture Notes:**
- Pattern: Command pattern with audit for all operations
- Scalability: Per-chain vault isolation
- Performance: Transfer initiation <30s, confirmation tracking real-time
- Data consistency: Strong consistency, double-entry bookkeeping

**Priority Level:** Critical
- **Justification:** PRD Section 四.7 requires vault management: "Vault 资金操作记录" and "财务在 admin 调整 vault 金额"

**Dependencies:**
- HSM/Key management infrastructure
- Blockchain node infrastructure
- Cold storage system

**Success Criteria:**
- Zero unauthorized vault transactions
- Hot wallet holds <20% of total custodial funds
- All vault operations have complete audit trail

---

#### 3.1.2 Child Address Allocation and Monitoring

**Feature Name:** Child Address Allocation and Monitoring

**Description:** System for managing HD wallet-derived child addresses assigned to users for deposits. Tracks address allocation, monitors incoming deposits, and handles address recycling for inactive accounts.

**Business Value:**
- Enables unique deposit addresses per user for attribution
- Supports scalable user onboarding
- Facilitates deposit reconciliation
- ROI: Automates address management for unlimited users

**Target User Roles:**
- **Finance:** View address allocations, monitor deposit status
- **Administrator:** Configure allocation rules, troubleshoot issues

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - child addresses are the Web2 wallet "addresses"
- **Web3 (Non-custodial):** Not applicable - users have their own addresses
- **Linked Wallets:** Child address can be linked to user's Web3 address

**Security Considerations:**
- HD derivation path securely stored
- Child private keys derived on-demand, not stored
- Address reuse prevention
- Deposit monitoring for unauthorized access attempts

**Integration Requirements:**
- HD wallet service: BIP-32/44 derivation
- Blockchain nodes: Deposit monitoring via event subscription
- User database: Address-to-user mapping
- Transaction service: Credit user balance on deposit

**Technical Architecture Notes:**
- Pattern: Factory pattern for address generation
- Scalability: Batch address pre-generation for performance
- Performance: Address generation <100ms, deposit detection <1 block
- Data consistency: Strong consistency for address-user mapping

**Priority Level:** Critical
- **Justification:** PRD Section 一.1 specifies HD wallet architecture with child addresses

**Dependencies:**
- HD wallet infrastructure
- Blockchain node infrastructure

**Success Criteria:**
- Unique address per user guaranteed
- Deposit credited within 3 block confirmations
- Zero address collision incidents

---

#### 3.1.3 Web2 Wallet Freeze/Unfreeze Operations

**Feature Name:** Web2 Wallet Freeze/Unfreeze Operations

**Description:** Capability to freeze individual user Web2 wallets, preventing all fund movements (deposits credited but not spendable, withdrawals and transfers blocked). Supports partial freeze (e.g., withdrawals only) and unfreezing with proper authorization.

**Business Value:**
- Essential for responding to suspicious activity
- Supports AML compliance requirements
- Enables dispute resolution without full account suspension
- ROI: Targeted intervention without complete account lockout

**Target User Roles:**
- **Administrator:** Can freeze/unfreeze wallets
- **Finance:** Can request freeze for suspicious transactions

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Full freeze capability - platform controls funds
- **Web3 (Non-custodial):** Cannot freeze (user-controlled)
- **Linked Wallets:** Freeze can block linked address exemption privileges

**Security Considerations:**
- Freeze actions require documented justification
- Automatic notification to user (unless under investigation)
- Time-limited freezes with mandatory review
- Audit trail for all freeze/unfreeze actions

**Integration Requirements:**
- Wallet database: Freeze status field
- Transaction service: Check freeze status before processing
- Notification service: User alerts
- Audit service: Action logging

**Technical Architecture Notes:**
- Pattern: State pattern for wallet status
- Scalability: Freeze check in every transaction path
- Performance: Freeze status check <10ms
- Data consistency: Strong consistency for freeze state

**Priority Level:** Critical
- **Justification:** PRD Section 一.1 explicitly mentions "冻结异常账号" as Web2 wallet use case

**Dependencies:**
- User Account Status Management (2.1.2)
- Transaction processing system

**Success Criteria:**
- Freeze takes effect within 5 seconds
- Zero transactions processed for frozen wallets
- Complete audit trail for all actions

---

### 3.2 Web3 Wallet (Non-custodial) Monitoring

#### 3.2.1 Linked Address Tracking

**Feature Name:** Linked Address Tracking

**Description:** Read-only monitoring of Web3 wallet addresses that users have linked to their Web2 accounts. Displays balance, recent transactions, and binding status. Manages the Web2 ↔ Web3 linking workflow.

**Business Value:**
- Enables linked address withdrawal exemption feature
- Provides complete picture of user's platform engagement
- Supports compliance monitoring across wallet types
- ROI: Streamlines withdrawals to verified addresses

**Target User Roles:**
- **Administrator:** View all linked addresses, manage binding rules
- **Finance:** View linked addresses for transaction context

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Shows which Web3 addresses are linked
- **Web3 (Non-custodial):** The linked addresses being tracked
- **Linked Wallets:** Core feature - manages the 1:1 binding

**Security Considerations:**
- Linking requires strong verification (biometric + verification code per PRD)
- Only one Web3 address can link to one Web2 account (1:1)
- Unlinking requires same verification as linking
- Linked status affects withdrawal approval workflow

**Integration Requirements:**
- User database: Link status storage
- Verification service: Multi-factor authentication
- Blockchain nodes: Web3 balance/transaction queries (read-only)
- Withdrawal service: Check linked status for exemptions

**Technical Architecture Notes:**
- Pattern: Observer pattern for link status changes
- Scalability: Cached link status for performance
- Performance: Link verification <5s, status check <100ms
- Data consistency: Strong consistency for link state

**Priority Level:** Critical
- **Justification:** PRD Section 三 "首页：Web2 钱包" requires "Web2 在转账给我们的关联 Web3 地址时，可以免审批"

**Dependencies:**
- User Management (2.1)
- Verification system
- Withdrawal Approval System (Feature 6)

**Success Criteria:**
- 1:1 binding enforced with zero violations
- Linked address exemption works correctly
- Complete audit trail for all binding changes

---

### 3.3 Wallet Address Whitelisting

#### 3.3.1 Address Whitelist Management

**Feature Name:** Address Whitelist Management

**Description:** System for managing trusted external wallet addresses that bypass certain restrictions or receive expedited processing. Includes address verification, categorization (exchange, partner, etc.), and expiration management.

**Business Value:**
- Streamlines operations with known trusted addresses
- Reduces friction for legitimate high-volume addresses
- Supports partnership integrations
- ROI: Faster processing for verified addresses

**Target User Roles:**
- **Administrator:** Add/remove addresses, configure whitelist rules
- **Finance:** View whitelist for transaction context

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Whitelist affects withdrawal destination approval
- **Web3 (Non-custodial):** Read-only context for transaction monitoring
- **Linked Wallets:** Linked addresses are effectively whitelisted for the user

**Security Considerations:**
- Whitelist additions require Administrator approval
- Regular whitelist review and expiration
- Separate from user-level blacklist
- Audit trail for all whitelist changes

**Integration Requirements:**
- Address database: Whitelist storage
- Withdrawal service: Check whitelist during approval
- Audit service: Change logging

**Technical Architecture Notes:**
- Pattern: Repository pattern with caching
- Scalability: In-memory cache for fast lookup
- Performance: Whitelist check <10ms
- Data consistency: Eventually consistent acceptable (5s max)

**Priority Level:** Medium
- **Justification:** Implied by approval workflow but not explicitly required in PRD V1.0

**Dependencies:**
- Withdrawal Approval System (Feature 6)

**Success Criteria:**
- Whitelist lookup performance <10ms
- Zero unauthorized whitelist additions
- Regular whitelist review completed monthly

---

## 4. Transaction Management

### 4.1 Deposit Processing

#### 4.1.1 Real-Time Deposit Monitoring

**Feature Name:** Real-Time Deposit Monitoring

**Description:** System that monitors blockchain transactions to child addresses, detects deposits, verifies confirmations, and triggers automatic crediting to user Web2 wallets. Displays pending and completed deposits with full transaction details.

**Business Value:**
- Enables automatic deposit processing without manual intervention
- Provides transparency on deposit status for support
- Supports reconciliation and audit requirements
- ROI: Eliminates manual deposit processing effort

**Target User Roles:**
- **Finance:** Monitor deposit flow, investigate issues
- **Administrator:** Configure confirmation requirements

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - deposits to child addresses credit Web2 balance
- **Web3 (Non-custodial):** Monitoring only - user controls their Web3 wallet
- **Linked Wallets:** Deposits to linked addresses shown for context

**Security Considerations:**
- Confirmation requirements per chain (prevent double-spend)
- Amount validation (minimum 10 USDT per PRD)
- Source address screening (blacklist check)
- Deposit crediting is irreversible - require high confidence

**Integration Requirements:**
- Blockchain nodes: Event subscription for incoming transactions
- Transaction database: Deposit records
- User balance service: Credit on confirmation
- Price oracle: USD conversion for minimum check

**Technical Architecture Notes:**
- Pattern: Event-driven with idempotent processing
- Scalability: Kafka/SQS for transaction event streaming
- Performance: Detection within 1 block, credit within configured confirmations
- Data consistency: Strong consistency for balance updates

**Priority Level:** Critical
- **Justification:** PRD Section 三 资产页面 specifies complete deposit workflow

**Dependencies:**
- Child Address Allocation (3.1.2)
- Blockchain node infrastructure
- User balance system

**Success Criteria:**
- Deposits credited within 3 confirmations
- Zero missed deposits
- Zero double-crediting incidents

---

### 4.2 Withdrawal Management

#### 4.2.1 Withdrawal Request Queue

**Feature Name:** Withdrawal Request Queue

**Description:** Displays all pending withdrawal requests with filtering by status, amount, user, and date. Provides detailed view of each request including amount, destination address, user information, and risk indicators.

**Business Value:**
- Central management point for withdrawal processing
- Enables efficient batch processing of approvals
- Supports risk-based prioritization
- ROI: Streamlines withdrawal operations

**Target User Roles:**
- **Finance:** Process withdrawal approvals/rejections
- **Administrator:** Configure queue rules, troubleshoot issues

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** All withdrawals from Web2 wallet appear here
- **Web3 (Non-custodial):** No queue - users execute directly on-chain
- **Linked Wallets:** Withdrawals to linked addresses may skip queue

**Security Considerations:**
- Queue access requires Finance role minimum
- Request details include risk scores for informed decisions
- Approval actions logged with approver identity
- SLA tracking for timely processing

**Integration Requirements:**
- Withdrawal database: Request storage
- Risk scoring service: AML/fraud indicators
- User database: User context for approval
- Approval workflow: Integration with approval engine

**Technical Architecture Notes:**
- Pattern: Priority queue with status state machine
- Scalability: Indexed queries for filtering
- Performance: Queue load <500ms for 1000 items
- Data consistency: Strong consistency for status changes

**Priority Level:** Critical
- **Justification:** PRD Section 四.5 specifies withdrawal approval workflow

**Dependencies:**
- User Management (Feature 2)
- Approval Workflow (Feature 6)

**Success Criteria:**
- Queue updates in real-time
- Approval SLA: 95% within 24 hours
- Zero unauthorized status changes

---

#### 4.2.2 Blockchain Transaction Execution

**Feature Name:** Blockchain Transaction Execution

**Description:** Service that executes approved withdrawals by submitting transactions to the blockchain. Handles gas estimation, nonce management, transaction signing via HSM, and status tracking through confirmation.

**Business Value:**
- Automated execution reduces operational overhead
- Proper gas management prevents stuck transactions
- Reliable execution ensures user satisfaction
- ROI: Eliminates manual transaction submission

**Target User Roles:**
- **Finance:** Trigger execution for approved withdrawals
- **Administrator:** Configure gas limits, troubleshoot failures

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - platform executes from vault
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** Same as Web2 withdrawals

**Security Considerations:**
- HSM signing for all outbound transactions
- Gas limit caps to prevent excessive fees
- Transaction replay protection
- Monitoring for front-running or MEV extraction

**Integration Requirements:**
- HSM: Transaction signing
- Blockchain nodes: Transaction submission and monitoring
- Gas price oracle: Optimal fee estimation
- Transaction database: Status tracking

**Technical Architecture Notes:**
- Pattern: Command pattern with retry logic
- Scalability: Queue-based parallel execution
- Performance: Submission within 30s of approval
- Data consistency: Idempotent submission with nonce tracking

**Priority Level:** Critical
- **Justification:** Required to complete withdrawal flow from PRD

**Dependencies:**
- Vault Management (3.1.1)
- Approval Workflow (Feature 6)
- HSM infrastructure

**Success Criteria:**
- 99% of transactions confirmed within 10 minutes
- Zero double-spend incidents
- Failed transaction retry success rate >95%

---

### 4.3 Internal Transfers (划转)

#### 4.3.1 Internal Transfer Processing

**Feature Name:** Internal Transfer Processing

**Description:** System for handling Web2 wallet-to-wallet transfers (划转) within the platform. Updates off-chain ledger balances without blockchain transactions. Supports search by UID, instant settlement, and zero fees.

**Business Value:**
- Free instant transfers within platform ecosystem
- Reduces blockchain transaction costs
- Enables payroll and internal payment use cases
- ROI: Significant gas fee savings for internal operations

**Target User Roles:**
- **Finance:** Monitor transfer volumes, reconcile ledger
- **Administrator:** Configure transfer limits, troubleshoot issues

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - ledger-based transfer between Web2 wallets
- **Web3 (Non-custodial):** Not applicable - Web3 transfers are on-chain
- **Linked Wallets:** No special handling - transfers are between Web2 accounts

**Security Considerations:**
- Sender balance verification before transfer
- Double-entry bookkeeping for auditability
- No fee but may have rate limits
- Verification required per PRD (biometric/2FA/fund password)

**Integration Requirements:**
- Ledger database: Balance updates
- User database: UID lookup for recipient
- Verification service: Transfer authorization
- Audit service: Transaction logging

**Technical Architecture Notes:**
- Pattern: Transaction script with ACID guarantees
- Scalability: Database partitioning by user
- Performance: Transfer completion <3 seconds
- Data consistency: Strong consistency required

**Priority Level:** Critical
- **Justification:** PRD Section 三 资产页面 specifies complete 划转 workflow

**Dependencies:**
- User Management (Feature 2)
- Ledger System (Feature 8)
- Verification System

**Success Criteria:**
- Transfer completion <3 seconds
- 100% ledger accuracy
- Zero balance discrepancies

---

### 4.4 Transaction History

#### 4.4.1 Comprehensive Transaction History

**Feature Name:** Comprehensive Transaction History

**Description:** Unified view of all transactions (deposits, withdrawals, transfers, swaps) across all users with advanced filtering by user, date range, status, chain, token, and transaction type. Supports export to CSV/Excel.

**Business Value:**
- Essential for audit and compliance
- Enables efficient support and investigation
- Supports financial reporting requirements
- ROI: Reduces report generation time significantly

**Target User Roles:**
- **Finance:** Primary user for reconciliation and reporting
- **Administrator:** Investigation and troubleshooting

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Off-chain transactions (transfers) and on-chain (deposits/withdrawals)
- **Web3 (Non-custodial):** On-chain transactions for linked addresses (read-only)
- **Linked Wallets:** Combined view of Web2 and linked Web3 transactions

**Security Considerations:**
- PII protection in transaction details
- Export controls based on role
- Audit logging of history access
- Data retention compliance

**Integration Requirements:**
- Transaction database: Unified transaction store
- Blockchain indexer: On-chain transaction data
- Export service: CSV/Excel generation
- User database: User context for display

**Technical Architecture Notes:**
- Pattern: CQRS with read-optimized projection
- Scalability: Time-partitioned tables, indexed queries
- Performance: <1s for typical queries, export handled async
- Data consistency: Eventually consistent for queries

**Priority Level:** Critical
- **Justification:** PRD Section 四.7 requires "Web2 内部划转记录" and "Web2 → Onchain 出金记录"

**Dependencies:**
- All transaction processing features
- Export infrastructure

**Success Criteria:**
- Query response <1s for 10,000 records
- Export completes within 5 minutes for 100,000 records
- 100% transaction visibility

---

#### 4.4.2 Blockchain Explorer Integration

**Feature Name:** Blockchain Explorer Integration

**Description:** Deep links to blockchain explorers (Tronscan, BscScan, Etherscan) for on-chain transaction verification. One-click navigation to transaction details, address pages, and block information.

**Business Value:**
- Enables independent transaction verification
- Improves user trust and transparency
- Supports investigation and troubleshooting
- ROI: Reduces support time for transaction verification

**Target User Roles:**
- **Finance:** Verify transaction status on-chain
- **Administrator:** Troubleshoot blockchain issues

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Links for deposits and withdrawals
- **Web3 (Non-custodial):** Links for all on-chain transactions
- **Linked Wallets:** Same as above

**Security Considerations:**
- Outbound links only (no external data ingestion)
- No sensitive data in URL parameters
- Open in new tab to maintain session

**Integration Requirements:**
- Explorer URL configuration per chain
- Transaction hash storage
- UI integration for link rendering

**Technical Architecture Notes:**
- Pattern: URL builder based on chain type
- Scalability: Stateless link generation
- Performance: Instant link generation
- Data consistency: N/A (external links)

**Priority Level:** High
- **Justification:** PRD Section 三 资产页面 specifies "区块浏览器：点击后跳转至对应的区块链浏览器页面"

**Dependencies:**
- Transaction History (4.4.1)

**Success Criteria:**
- Correct explorer URL for each chain
- Links work for all transaction types
- Zero broken links

---

## 5. Payroll & Batch Operations

### 5.1 Bulk Salary Distribution

#### 5.1.1 Excel/CSV Import and Validation

**Feature Name:** Excel/CSV Import and Validation

**Description:** Import interface for bulk salary distribution files with template download. Validates file format, required fields (UID, token, amount), data integrity, and recipient eligibility. Provides detailed error reports for failed validations.

**Business Value:**
- Enables efficient mass payroll processing
- Reduces manual entry errors
- Supports existing HR/finance Excel workflows
- ROI: Processes thousands of payments in minutes vs. hours

**Target User Roles:**
- **Finance:** Primary user - uploads payroll files

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Target wallets are Web2 accounts
- **Web3 (Non-custodial):** Not applicable - payroll is to Web2 wallets
- **Linked Wallets:** Recipients receive in Web2 wallet regardless of linking

**Security Considerations:**
- File upload validation (type, size limits)
- Input sanitization to prevent injection
- Preview before execution
- Cannot modify after approval submission

**Integration Requirements:**
- File parsing service: Excel/CSV parsing
- User database: UID validation
- Token configuration: Valid token verification
- Balance service: Vault balance check

**Technical Architecture Notes:**
- Pattern: Import pipeline with validation stages
- Scalability: Streaming parser for large files
- Performance: 10,000 row file processed in <30 seconds
- Data consistency: Atomic batch creation

**Priority Level:** Critical
- **Justification:** PRD Section 四.2 explicitly requires "批量导入 Excel（字段：uid、币种、金额）"

**Dependencies:**
- User Management (Feature 2)
- Token Configuration (Feature 11)

**Success Criteria:**
- File validation completes in <30 seconds
- Clear error messages for all validation failures
- Zero false positives in eligibility checks

---

#### 5.1.2 Batch Approval Workflow

**Feature Name:** Batch Approval Workflow

**Description:** Multi-level approval process for payroll batches with configurable thresholds. Supports batch preview, partial approval/rejection, and approval routing based on total batch value.

**Business Value:**
- Control over large financial disbursements
- Separation of duties for compliance
- Audit trail for all payroll decisions
- ROI: Reduced risk of unauthorized disbursements

**Target User Roles:**
- **Finance:** Submits and approves payroll batches

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - approves Web2 balance credits
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** No special handling

**Security Considerations:**
- Maker-checker principle (submitter cannot self-approve)
- Approval timeout for unprocessed batches
- Cannot modify batch after submission
- Complete audit trail

**Integration Requirements:**
- Approval engine: Workflow orchestration
- User database: Approver authorization
- Notification service: Approval request alerts
- Audit service: Decision logging

**Technical Architecture Notes:**
- Pattern: State machine for batch lifecycle
- Scalability: Queue-based approval processing
- Performance: Approval decision recorded in <3 seconds
- Data consistency: Strong consistency for approval state

**Priority Level:** Critical
- **Justification:** PRD Section 四.2 requires "支持批量审批及发放"

**Dependencies:**
- Excel Import (5.1.1)
- Approval Workflow Engine (Feature 6)
- RBAC (Feature 12)

**Success Criteria:**
- Zero unauthorized batch approvals
- Approval SLA: 95% within 24 hours
- Complete audit trail for all decisions

---

#### 5.1.3 Payment Execution Engine

**Feature Name:** Payment Execution Engine

**Description:** Executes approved payroll batches by crediting recipient Web2 wallet balances. Supports sequential or parallel processing, handles partial failures gracefully, and provides detailed execution logs.

**Business Value:**
- Automated execution of approved payrolls
- Reliable handling of large batch operations
- Clear visibility into execution status
- ROI: Eliminates manual payment processing

**Target User Roles:**
- **Finance:** Monitor execution, handle failures
- **Administrator:** Configure execution parameters

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Off-chain ledger updates - no blockchain transactions
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** No special handling - updates Web2 balance

**Security Considerations:**
- Only approved batches can execute
- Vault balance verification before execution
- Idempotent execution (prevent double payments)
- Execution cannot be cancelled mid-batch

**Integration Requirements:**
- Ledger database: Balance updates
- Batch database: Execution status tracking
- Vault service: Balance verification
- Notification service: Execution completion alerts

**Technical Architecture Notes:**
- Pattern: Saga pattern for distributed transaction
- Scalability: Parallel execution with configurable concurrency
- Performance: 1,000 payments/minute
- Data consistency: Eventual consistency with reconciliation

**Priority Level:** Critical
- **Justification:** Required to complete payroll workflow from PRD Section 四.2

**Dependencies:**
- Batch Approval (5.1.2)
- Ledger System (Feature 8)
- Vault Management (Feature 3.1)

**Success Criteria:**
- 99.9% execution success rate
- Zero duplicate payments
- Failed payments identified and retriable

---

#### 5.1.4 Distribution Logs and Export

**Feature Name:** Distribution Logs and Export

**Description:** Comprehensive logging of all payroll distributions with status tracking per recipient. Supports filtering, detailed views, and export to Excel/CSV for accounting integration.

**Business Value:**
- Audit trail for all salary distributions
- Integration with accounting systems
- Supports reconciliation and compliance
- ROI: Reduces accounting reconciliation effort

**Target User Roles:**
- **Finance:** Primary user - reporting and export

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** All payroll transactions logged
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** No special handling

**Security Considerations:**
- PII in logs requires appropriate access
- Export controls based on role
- Retention policy compliance
- Audit of log access

**Integration Requirements:**
- Distribution database: Log storage
- Export service: File generation
- Audit service: Access logging

**Technical Architecture Notes:**
- Pattern: Event sourcing for complete history
- Scalability: Time-partitioned storage
- Performance: Export <5 minutes for 100,000 records
- Data consistency: Eventually consistent for queries

**Priority Level:** Critical
- **Justification:** PRD Section 四.2 explicitly requires "发放日志记录与可导出报表"

**Dependencies:**
- Payment Execution (5.1.3)
- Export Infrastructure

**Success Criteria:**
- Complete log for every distribution
- Export completes within 5 minutes
- 100% reconciliation accuracy

---

## 6. Withdrawal Approval System

### 6.1 Approval Workflow Configuration

#### 6.1.1 Amount-Based Approval Thresholds

**Feature Name:** Amount-Based Approval Thresholds

**Description:** Configurable thresholds that determine approval requirements based on withdrawal amount. Supports automatic approval below threshold, single approval for medium amounts, and multi-level approval for large amounts.

**Business Value:**
- Risk-proportionate approval burden
- Faster processing for small withdrawals
- Enhanced control for large amounts
- ROI: Optimizes approval resource allocation

**Target User Roles:**
- **Administrator:** Configure threshold rules
- **Finance:** View current thresholds for context

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Thresholds apply to all Web2 withdrawals
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** May have different (higher) auto-approval threshold

**Security Considerations:**
- Threshold changes require Administrator approval
- Audit trail for all configuration changes
- Cannot set thresholds that bypass all approval
- Per-token and per-chain configuration

**Integration Requirements:**
- Configuration database: Threshold storage
- Approval engine: Threshold evaluation
- Audit service: Configuration change logging

**Technical Architecture Notes:**
- Pattern: Strategy pattern for threshold evaluation
- Scalability: Cached configuration
- Performance: Evaluation <100ms
- Data consistency: Eventual consistency for config, strong for decisions

**Priority Level:** Critical
- **Justification:** PRD Section 四.5 specifies approval workflow with AML verification

**Dependencies:**
- Withdrawal Queue (4.2.1)
- RBAC (Feature 12)

**Success Criteria:**
- Correct threshold application 100% of time
- Configuration changes audited
- No approval bypasses

---

#### 6.1.2 Linked Address Exemption Rules

**Feature Name:** Linked Address Exemption Rules

**Description:** Special approval rules for withdrawals to addresses linked to the user's Web3 wallet. Configurable exemption allowing auto-approval for verified Web2 ↔ Web3 bindings up to specified limits.

**Business Value:**
- Frictionless experience for users with linked wallets
- Reduces approval queue volume for low-risk transfers
- Encourages platform ecosystem usage
- ROI: Faster user experience, lower operational cost

**Target User Roles:**
- **Administrator:** Configure exemption rules
- **Finance:** View exemption status in approval context

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - withdrawal from Web2 wallet
- **Web3 (Non-custodial):** Destination is user's linked Web3 address
- **Linked Wallets:** This feature exists specifically for linked wallets

**Security Considerations:**
- Exemption requires verified linking (biometric + verification code)
- Daily/monthly limits on exempted amounts
- Address verification on each withdrawal
- Exemption can be revoked if linking status changes

**Integration Requirements:**
- Link database: Binding status verification
- Withdrawal service: Exemption evaluation
- Configuration: Exemption limits
- Audit service: Exemption usage logging

**Technical Architecture Notes:**
- Pattern: Decorator pattern for approval rules
- Scalability: Cached link status
- Performance: Exemption check <100ms
- Data consistency: Strong consistency for link state

**Priority Level:** Critical
- **Justification:** PRD Section 三 "首页：Web2 钱包" explicitly states "Web2 在转账给我们的关联 Web3 地址时，可以免审批"

**Dependencies:**
- Linked Address Tracking (3.2.1)
- Amount-Based Thresholds (6.1.1)

**Success Criteria:**
- Exemption applies correctly for verified links
- Zero exemptions for unverified addresses
- Complete audit trail of exemptions

---

### 6.2 AML/KYC Integration

#### 6.2.1 Automated Risk Scoring

**Feature Name:** Automated Risk Scoring

**Description:** Integration with AML screening services to automatically score withdrawal requests based on destination address risk, user history, and transaction patterns. High-risk scores trigger manual review.

**Business Value:**
- Compliance with AML regulations
- Automated risk triage reduces manual review burden
- Protects platform from regulatory penalties
- ROI: Prevents potential compliance fines

**Target User Roles:**
- **Finance:** Views risk scores during approval
- **Administrator:** Configure risk thresholds

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** All withdrawals scored
- **Web3 (Non-custodial):** Monitoring only
- **Linked Wallets:** Score may be lower for verified links

**Security Considerations:**
- AML service credentials secured
- Risk data is compliance-sensitive
- Cannot bypass AML check
- Retention of risk scoring records

**Integration Requirements:**
- AML service: External provider (Chainalysis, Elliptic, etc.)
- Risk database: Score storage
- Withdrawal service: Integrate scoring in workflow
- Configuration: Risk threshold settings

**Technical Architecture Notes:**
- Pattern: Chain of responsibility for risk evaluation
- Scalability: Async scoring with SLA
- Performance: Score returned within 5 seconds
- Data consistency: Score immutable once assigned

**Priority Level:** Critical
- **Justification:** PRD Section 四.5 explicitly includes "AML 验证" in approval workflow

**Dependencies:**
- External AML service provider
- Withdrawal Queue (4.2.1)

**Success Criteria:**
- 100% of withdrawals scored
- AML response within 5 seconds
- Zero unscored withdrawals processed

---

### 6.3 Approval Queue Management

#### 6.3.1 Approval Queue with SLA Tracking

**Feature Name:** Approval Queue with SLA Tracking

**Description:** Centralized queue for all pending approvals with SLA monitoring. Supports assignment, reassignment, escalation for overdue items, and filtering by approver, status, and age.

**Business Value:**
- Ensures timely processing of withdrawals
- Visibility into approval backlog
- Automated escalation prevents stuck requests
- ROI: Improved user satisfaction through faster processing

**Target User Roles:**
- **Finance:** Process approvals from queue
- **Administrator:** Configure SLA rules, view escalations

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Withdrawals pending approval
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** Fewer items due to exemptions

**Security Considerations:**
- Approval access restricted by role
- Cannot approve own requests
- SLA data for operational metrics only
- Audit trail for all approvals

**Integration Requirements:**
- Approval database: Queue storage
- SLA engine: Tracking and escalation
- Notification service: Escalation alerts
- Assignment service: Work distribution

**Technical Architecture Notes:**
- Pattern: Priority queue with aging
- Scalability: Indexed queries for filtering
- Performance: Queue load <500ms
- Data consistency: Strong consistency for assignments

**Priority Level:** High
- **Justification:** Required for operational efficiency though not explicitly in PRD

**Dependencies:**
- Withdrawal Request Queue (4.2.1)
- Approval Workflow (6.1)

**Success Criteria:**
- SLA: 95% approved within 24 hours
- Escalation triggers at correct thresholds
- Zero lost approval requests

---

#### 6.3.2 Rejection Handling with Reason Codes

**Feature Name:** Rejection Handling with Reason Codes

**Description:** Standardized rejection workflow with required reason codes and optional detailed comments. Triggers automatic user notification explaining rejection and any remediation options.

**Business Value:**
- Clear communication to users
- Standardized rejection reasons for analytics
- Supports appeal and remediation processes
- ROI: Reduces support inquiries about rejections

**Target User Roles:**
- **Finance:** Select reason codes during rejection
- **Administrator:** Configure reason code options

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Rejections return funds to Web2 balance
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** Same handling as Web2

**Security Considerations:**
- Rejection reasons stored for audit
- User notification must not reveal sensitive info
- Rejection is final but can request again
- Audit trail required

**Integration Requirements:**
- Reason code configuration: Managed list
- Notification service: User alerts
- Withdrawal service: Status update
- Analytics: Rejection reason aggregation

**Technical Architecture Notes:**
- Pattern: Command pattern with validation
- Scalability: Stateless rejection processing
- Performance: Rejection recorded in <3 seconds
- Data consistency: Strong consistency for status

**Priority Level:** High
- **Justification:** PRD Section 三 资产页面 mentions "已驳回" status

**Dependencies:**
- Approval Queue (6.3.1)
- Notification Service

**Success Criteria:**
- Reason code required for all rejections
- User notified within 1 minute of rejection
- Complete audit trail

---

## 7. Security & Risk Control

### 7.1 Blacklist Management

#### 7.1.1 Wallet Address Blacklist

**Feature Name:** Wallet Address Blacklist

**Description:** System for blocking transactions to/from flagged wallet addresses. Supports manual additions with reason tracking, integration with external blacklist feeds, and expiration dates for temporary blocks.

**Business Value:**
- Prevents transactions with known bad actors
- Compliance with sanctions requirements
- Protects platform from regulatory risk
- ROI: Prevents potential compliance penalties

**Target User Roles:**
- **Administrator:** Manage blacklist entries
- **Finance:** View blacklist for transaction context

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Blocks withdrawals to blacklisted addresses, blocks deposits from blacklisted sources
- **Web3 (Non-custodial):** Read-only - cannot block user-controlled transactions, but can flag for monitoring
- **Linked Wallets:** Cannot link to blacklisted address

**Security Considerations:**
- Blacklist updates require Administrator role
- Integration with OFAC and other sanctions lists
- Real-time enforcement on all transactions
- Audit trail for all changes

**Integration Requirements:**
- External blacklist feeds: OFAC, Chainalysis, etc.
- Address database: Blacklist storage
- Transaction service: Real-time checking
- Audit service: Change logging

**Technical Architecture Notes:**
- Pattern: Bloom filter + database for fast lookup
- Scalability: In-memory blacklist cache
- Performance: Check <10ms per address
- Data consistency: Eventually consistent with short refresh

**Priority Level:** Critical
- **Justification:** PRD Section 三 "特殊场景处理" mentions wallet address blacklisting

**Dependencies:**
- Transaction Processing (Feature 4)
- External blacklist services

**Success Criteria:**
- 100% transaction coverage
- Check performance <10ms
- Zero transactions to blacklisted addresses

---

### 7.2 Fraud Detection

#### 7.2.1 Velocity and Pattern Detection

**Feature Name:** Velocity and Pattern Detection

**Description:** Rule-based and ML-powered detection of unusual transaction patterns including rapid successive transactions, unusual amounts, new address patterns, and behavioral anomalies.

**Business Value:**
- Early detection of account compromise
- Prevents financial losses from fraud
- Protects user funds proactively
- ROI: Prevents potential fraud losses

**Target User Roles:**
- **Administrator:** Configure detection rules
- **Finance:** Respond to alerts

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Full monitoring and blocking capability
- **Web3 (Non-custodial):** Monitoring only, alert generation
- **Linked Wallets:** Combined pattern analysis

**Security Considerations:**
- Detection rules are sensitive (prevent gaming)
- False positive handling important
- Graduated response (alert vs. block)
- Regular rule tuning based on patterns

**Integration Requirements:**
- Transaction stream: Real-time event feed
- Rules engine: Pattern matching
- ML service: Anomaly detection
- Alert service: Notification on detection

**Technical Architecture Notes:**
- Pattern: Complex event processing (CEP)
- Scalability: Stream processing (Kafka Streams, Flink)
- Performance: Detection within 30 seconds of transaction
- Data consistency: Eventually consistent acceptable

**Priority Level:** High
- **Justification:** Essential for platform security though not explicitly in PRD

**Dependencies:**
- Transaction Processing (Feature 4)
- Alert Service (Feature 7.3)

**Success Criteria:**
- Detection within 30 seconds
- False positive rate <10%
- Zero undetected fraud incidents

---

### 7.3 Security Audit Logs

#### 7.3.1 Admin Action Audit Trail

**Feature Name:** Admin Action Audit Trail

**Description:** Comprehensive, immutable logging of all administrative actions including user management, configuration changes, approvals, and system access. Supports search, filtering, and export for compliance reviews.

**Business Value:**
- Compliance with audit requirements
- Investigation support for security incidents
- Accountability for administrative actions
- ROI: Reduces audit preparation time

**Target User Roles:**
- **Administrator:** Primary viewer for investigation
- **Finance:** View own actions and related transactions

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Logs for all custodial operations
- **Web3 (Non-custodial):** Logs for monitoring configuration changes
- **Linked Wallets:** Logs for binding/unbinding actions

**Security Considerations:**
- Logs must be tamper-proof
- Separate storage from operational data
- Retention per compliance requirements
- Access to logs restricted

**Integration Requirements:**
- Audit database: Append-only log storage
- All services: Event emission on actions
- Search service: Log query capability
- Export service: Compliance report generation

**Technical Architecture Notes:**
- Pattern: Event sourcing with immutable store
- Scalability: Time-partitioned storage
- Performance: Write <100ms, query <5 seconds for typical searches
- Data consistency: Append-only, eventually consistent reads

**Priority Level:** Critical
- **Justification:** Required for compliance and security

**Dependencies:**
- All administrative features
- Immutable storage infrastructure

**Success Criteria:**
- 100% action coverage
- Zero log tampering capability
- Search returns in <5 seconds

---

## 8. Compliance & Audit

### 8.1 Off-chain Ledger Management

#### 8.1.1 Web2 Transaction Ledger

**Feature Name:** Web2 Transaction Ledger

**Description:** Complete double-entry ledger system recording all Web2 wallet operations including internal transfers, balance adjustments, payroll credits, and the off-chain portion of deposits/withdrawals.

**Business Value:**
- Auditable record of all fund movements
- Supports reconciliation with on-chain state
- Foundation for financial reporting
- ROI: Meets accounting and audit requirements

**Target User Roles:**
- **Finance:** Primary user - reconciliation and reporting
- **Administrator:** Troubleshooting and investigation

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - all Web2 transactions recorded
- **Web3 (Non-custodial):** Not applicable (on-chain is the ledger)
- **Linked Wallets:** Ledger entries for Web2 side of linked operations

**Security Considerations:**
- Immutable entries (corrections via adjusting entries only)
- Double-entry validation on every entry
- Access restricted to appropriate roles
- Backup and disaster recovery

**Integration Requirements:**
- Ledger database: Specialized accounting database
- Transaction services: Entry creation on all operations
- Reconciliation service: Comparison with on-chain/vault
- Reporting service: Financial report generation

**Technical Architecture Notes:**
- Pattern: Double-entry bookkeeping with journal/ledger
- Scalability: Time-partitioned tables
- Performance: Entry creation <100ms, query performance varies
- Data consistency: Strong consistency with ACID transactions

**Priority Level:** Critical
- **Justification:** PRD Section 四.7 explicitly requires "Web2 内部划转记录" and account book management

**Dependencies:**
- All transaction processing features
- Database infrastructure with ACID support

**Success Criteria:**
- Double-entry balance: debits = credits always
- Reconciliation to vault balance daily
- Zero unexplained discrepancies

---

#### 8.1.2 Balance Reconciliation Tools

**Feature Name:** Balance Reconciliation Tools

**Description:** Automated and manual reconciliation between off-chain ledger totals and actual on-chain vault balances. Identifies discrepancies, supports investigation, and generates reconciliation reports.

**Business Value:**
- Ensures ledger accuracy
- Early detection of accounting issues
- Supports audit requirements
- ROI: Prevents undetected financial discrepancies

**Target User Roles:**
- **Finance:** Run reconciliation, investigate discrepancies

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Reconcile ledger total vs. vault balance
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** No special handling

**Security Considerations:**
- Reconciliation results are sensitive
- Discrepancies trigger alerts
- Cannot auto-adjust without approval
- Audit trail for reconciliation runs

**Integration Requirements:**
- Ledger database: Balance summation
- Vault service: Actual balance query
- Alert service: Discrepancy notifications
- Report service: Reconciliation report generation

**Technical Architecture Notes:**
- Pattern: Batch processing with comparison logic
- Scalability: Can run for subset of accounts
- Performance: Full reconciliation in <5 minutes
- Data consistency: Point-in-time snapshot comparison

**Priority Level:** Critical
- **Justification:** Essential for financial integrity

**Dependencies:**
- Ledger (8.1.1)
- Vault Management (3.1.1)

**Success Criteria:**
- Daily reconciliation completes successfully
- Discrepancy detection within 1 block of cause
- Investigation tools identify root cause

---

### 8.2 On-chain Transaction Tracking

#### 8.2.1 Confirmation Status Monitoring

**Feature Name:** Confirmation Status Monitoring

**Description:** Real-time tracking of blockchain confirmation status for all outbound transactions (withdrawals, payroll on-chain if any). Updates internal status as confirmations progress, handles reorgs gracefully.

**Business Value:**
- Accurate transaction status for users
- Supports reliable withdrawal processing
- Enables proper ledger entry timing
- ROI: Reduced support inquiries about status

**Target User Roles:**
- **Finance:** Monitor transaction completion
- **Administrator:** Troubleshoot confirmation issues

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Tracks outbound withdrawals from vault
- **Web3 (Non-custodial):** Read-only monitoring of linked address transactions
- **Linked Wallets:** Same tracking for withdrawals to linked addresses

**Security Considerations:**
- Confirmation count per chain configured appropriately
- Reorg handling to prevent double-counting
- Final status only after sufficient confirmations
- No status manipulation possible

**Integration Requirements:**
- Blockchain nodes: Transaction receipt queries
- Transaction database: Status updates
- Event stream: Confirmation events
- Notification service: Status change alerts

**Technical Architecture Notes:**
- Pattern: Polling with exponential backoff
- Scalability: Batch queries for pending transactions
- Performance: Status update within 1 block
- Data consistency: Eventually consistent with blockchain

**Priority Level:** Critical
- **Justification:** Required for accurate transaction processing

**Dependencies:**
- Blockchain node infrastructure
- Transaction Processing (Feature 4)

**Success Criteria:**
- Status reflects blockchain within 1 block
- Zero missed confirmations
- Reorg handling without double-entry

---

## 9. Financial Operations

### 9.1 Vault Balance Management

#### 9.1.1 Manual Balance Adjustments

**Feature Name:** Manual Balance Adjustments

**Description:** Capability for authorized Finance personnel to make manual adjustments to vault balances with required justification, approval workflow, and complete audit trail. Supports both increases and decreases.

**Business Value:**
- Correct discrepancies from manual operations
- Record external fund movements (cold storage transfers)
- Maintain accurate books despite exceptional circumstances
- ROI: Enables proper accounting for edge cases

**Target User Roles:**
- **Finance:** Initiate adjustments with justification
- **Administrator:** Approve adjustments (if configured)

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Adjusts vault balance (actual on-chain assets unchanged)
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** No special handling

**Security Considerations:**
- Dual approval recommended for adjustments
- Detailed justification required
- Audit trail is immutable
- Alerts for large adjustments

**Integration Requirements:**
- Ledger database: Adjusting entries
- Approval service: Adjustment approval workflow
- Audit service: Complete logging
- Alert service: Anomaly detection

**Technical Architecture Notes:**
- Pattern: Command pattern with approval workflow
- Scalability: Low volume operation
- Performance: Adjustment recorded in <3 seconds
- Data consistency: Strong consistency required

**Priority Level:** Critical
- **Justification:** PRD Section 四.7 explicitly requires "财务在 admin 调整 vault 金额"

**Dependencies:**
- Ledger System (8.1)
- Approval Workflow (Feature 6)

**Success Criteria:**
- All adjustments have justification
- Approval workflow enforced
- Complete audit trail

---

### 9.2 Fee Configuration

#### 9.2.1 Withdrawal Fee Settings

**Feature Name:** Withdrawal Fee Settings

**Description:** Configuration interface for withdrawal fees supporting flat fee, percentage-based, or tiered structures. Per-token and per-chain configuration with effective date management.

**Business Value:**
- Revenue generation from withdrawal services
- Cost recovery for blockchain fees
- Competitive pricing flexibility
- ROI: Directly impacts platform revenue

**Target User Roles:**
- **Finance:** Configure and update fee structures
- **Administrator:** Approve fee changes

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Fees applied to all Web2 withdrawals
- **Web3 (Non-custodial):** No platform fees (users pay gas)
- **Linked Wallets:** Same fees as regular withdrawals

**Security Considerations:**
- Fee changes require approval
- Historical fee preservation for audit
- Clear disclosure to users
- Cannot set exploitative fees

**Integration Requirements:**
- Configuration database: Fee structure storage
- Withdrawal service: Fee calculation integration
- User interface: Fee display before confirmation
- Audit service: Fee change logging

**Technical Architecture Notes:**
- Pattern: Strategy pattern for fee calculation
- Scalability: Cached configuration
- Performance: Fee calculation <50ms
- Data consistency: Eventually consistent acceptable

**Priority Level:** High
- **Justification:** PRD Section 三 资产页面 mentions "手续费：根据后台配置手续费展示"

**Dependencies:**
- Withdrawal Processing (Feature 4.2)
- Configuration Management (Feature 11)

**Success Criteria:**
- Correct fee calculation 100% of time
- Fee displayed accurately to users
- All changes audited

---

### 9.3 Transaction Limits

#### 9.3.1 Minimum Transaction Configuration

**Feature Name:** Minimum Transaction Configuration

**Description:** Configuration of minimum transaction amounts per operation type (deposit, withdrawal, transfer) and per token. Validation enforced at transaction initiation.

**Business Value:**
- Prevents spam transactions
- Ensures economic viability of transactions
- Supports operational efficiency
- ROI: Reduces processing cost for tiny transactions

**Target User Roles:**
- **Administrator:** Configure minimum limits

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Enforced for all Web2 operations
- **Web3 (Non-custodial):** Read-only - minimums are blockchain-determined
- **Linked Wallets:** Web2 minimums apply

**Security Considerations:**
- Minimum changes require approval
- Cannot set minimums that lock user funds
- Historical configuration preserved
- Clear user messaging

**Integration Requirements:**
- Configuration database: Limit storage
- Transaction services: Limit validation
- UI: Error messaging for below-minimum
- Price oracle: USD conversion for 10 USDT minimum

**Technical Architecture Notes:**
- Pattern: Validation middleware
- Scalability: Cached configuration
- Performance: Validation <50ms
- Data consistency: Eventually consistent

**Priority Level:** High
- **Justification:** PRD specifies minimum 10 USDT for deposits and withdrawals

**Dependencies:**
- Transaction Processing (Feature 4)
- Price Oracle integration

**Success Criteria:**
- All transactions validated against minimums
- Clear error messages for violations
- Configuration changes audited

---

## 10. DEX/Swap Configuration

### 10.1 DEX Integration Management

#### 10.1.1 DEX Protocol Configuration

**Feature Name:** DEX Protocol Configuration

**Description:** Management interface for configuring supported DEX protocols (Uniswap, PancakeSwap, etc.) including RPC endpoints, router contract addresses, and enable/disable toggles per chain.

**Business Value:**
- Enables swap functionality for users
- Multi-DEX support for best rates
- Flexible configuration without code changes
- ROI: Enables swap revenue opportunity

**Target User Roles:**
- **Administrator:** Configure DEX integrations

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Swaps NOT supported per PRD (Web3 only feature)
- **Web3 (Non-custodial):** Core feature - swaps execute from user's Web3 wallet
- **Linked Wallets:** No special handling

**Security Considerations:**
- Router addresses must be verified contracts
- RPC endpoints security (no sensitive data in requests)
- Enable/disable for rapid response to DEX issues
- Audit trail for all configuration changes

**Integration Requirements:**
- Configuration database: DEX settings storage
- RPC nodes: Connection health monitoring
- Smart contract integration: Router ABI
- Admin UI: Configuration interface

**Technical Architecture Notes:**
- Pattern: Adapter pattern for multi-DEX support
- Scalability: Per-chain isolation
- Performance: Configuration cached
- Data consistency: Eventually consistent acceptable

**Priority Level:** High
- **Justification:** PRD Section 四.4 specifies "接入 DEX 清单管理"

**Dependencies:**
- RPC Node infrastructure
- Smart contract integration capabilities

**Success Criteria:**
- DEX configuration without code deployment
- Enable/disable effective within 1 minute
- All changes audited

---

#### 10.1.2 Slippage and Parameter Settings

**Feature Name:** Slippage and Parameter Settings

**Description:** Configuration for swap parameters including default slippage tolerance, user-adjustable slippage range (0.1%-50%), preset options (0.5%/1%/3%), and transaction deadline settings.

**Business Value:**
- Protects users from excessive slippage
- Provides sensible defaults while allowing customization
- Enables platform-appropriate risk parameters
- ROI: Better user experience, fewer failed swaps

**Target User Roles:**
- **Administrator:** Configure slippage defaults and limits

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Not applicable (no swaps)
- **Web3 (Non-custodial):** Slippage applied to user's swap transactions
- **Linked Wallets:** No special handling

**Security Considerations:**
- Maximum slippage cap to prevent exploitation
- Clear disclosure of slippage to users
- Cannot set dangerous default values
- Audit trail for changes

**Integration Requirements:**
- Configuration database: Slippage settings
- Swap service: Parameter application
- User interface: Slippage display and selection
- API: Endpoint for slippage config per PRD

**Technical Architecture Notes:**
- Pattern: Configuration-driven behavior
- Scalability: Cached settings
- Performance: N/A (configuration)
- Data consistency: Eventually consistent

**Priority Level:** High
- **Justification:** PRD Section 四.4 requires "配置汇率接口和滑点规则" and specifically mentions slippage API

**Dependencies:**
- DEX Configuration (10.1.1)

**Success Criteria:**
- Default slippage applied correctly
- User overrides within allowed range
- All settings changes audited

---

#### 10.1.3 Swap Transaction History

**Feature Name:** Swap Transaction History

**Description:** Record and display of all swap transactions executed through the platform including tokens swapped, amounts, rates achieved, slippage actual vs. expected, and transaction status.

**Business Value:**
- User transparency on swap history
- Support investigation capability
- Analytics on swap usage patterns
- ROI: Improved user experience and support

**Target User Roles:**
- **Administrator:** View all swap history
- **Finance:** Analytics and reporting

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Not applicable
- **Web3 (Non-custodial):** All user swaps recorded
- **Linked Wallets:** Swaps from linked addresses shown

**Security Considerations:**
- Transaction data is user financial information
- Access restricted appropriately
- Export controls
- Retention policy

**Integration Requirements:**
- Transaction database: Swap records
- Blockchain indexer: On-chain swap data
- Analytics service: Usage reporting
- Export service: Data extraction

**Technical Architecture Notes:**
- Pattern: Event sourcing for swap events
- Scalability: Time-partitioned storage
- Performance: Query <1s for typical searches
- Data consistency: Eventually consistent

**Priority Level:** High
- **Justification:** PRD Section 四.4 requires "查看历史闪兑交易记录"

**Dependencies:**
- DEX Integration (10.1.1)
- Transaction History (4.4)

**Success Criteria:**
- Complete record of all swaps
- Accurate rate and amount data
- Query performance <1s

---

## 11. System Configuration

### 11.1 Blockchain & Token Management

#### 11.1.1 Supported Token Configuration

**Feature Name:** Supported Token Configuration

**Description:** Management of supported tokens including adding new tokens, configuring contract addresses, decimal precision, display name, and enable/disable status per chain.

**Business Value:**
- Flexible token support without code changes
- Quick response to new token requirements
- Consistent token handling across platform
- ROI: Reduced development time for new tokens

**Target User Roles:**
- **Administrator:** Configure token settings

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Tokens available for deposit, withdrawal, transfer
- **Web3 (Non-custodial):** Tokens displayed in user's wallet
- **Linked Wallets:** Same token support

**Security Considerations:**
- Contract address verification critical
- Decimal precision must be accurate
- Cannot add tokens without proper validation
- Audit trail for all changes

**Integration Requirements:**
- Configuration database: Token settings
- Blockchain nodes: Contract verification
- All services: Token configuration integration
- Price oracle: Token price feeds

**Technical Architecture Notes:**
- Pattern: Registry pattern for tokens
- Scalability: Cached configuration
- Performance: Token lookup <10ms
- Data consistency: Eventually consistent

**Priority Level:** Critical
- **Justification:** PRD specifies multi-token support (USDT, TRX, BNB, ETH)

**Dependencies:**
- Blockchain node infrastructure

**Success Criteria:**
- Token configuration without code deployment
- Correct handling of decimals
- All changes audited

---

#### 11.1.2 Network/Chain Configuration

**Feature Name:** Network/Chain Configuration

**Description:** Configuration of supported blockchain networks including chain ID, RPC endpoints (primary and backup), block explorer URLs, native token, and confirmation requirements.

**Business Value:**
- Multi-chain support with centralized configuration
- Quick RPC endpoint updates without deployment
- Network-specific settings management
- ROI: Operational flexibility and resilience

**Target User Roles:**
- **Administrator:** Configure network settings

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Chains where Web2 deposits/withdrawals are supported
- **Web3 (Non-custodial):** Chains where Web3 wallets can operate
- **Linked Wallets:** Same chain support

**Security Considerations:**
- RPC endpoint changes can affect security
- Chain ID verification prevents cross-chain issues
- Backup endpoints for resilience
- Audit trail for changes

**Integration Requirements:**
- Configuration database: Network settings
- All blockchain services: Configuration consumption
- Health monitoring: RPC endpoint status
- Admin UI: Configuration interface

**Technical Architecture Notes:**
- Pattern: Multi-chain abstraction layer
- Scalability: Per-chain service isolation
- Performance: Configuration cached
- Data consistency: Eventually consistent

**Priority Level:** Critical
- **Justification:** PRD specifies TRX/BNB/ETH support

**Dependencies:**
- RPC node infrastructure

**Success Criteria:**
- Multi-chain operations work correctly
- RPC failover functions properly
- All changes audited

---

### 11.2 Security Settings

#### 11.2.1 Verification Method Configuration

**Feature Name:** Verification Method Configuration

**Description:** Configuration of user verification methods including 2FA requirements, biometric options, fund password rules, and priority order for verification challenges.

**Business Value:**
- Security policy customization
- Compliance with security standards
- User experience optimization
- ROI: Balance between security and usability

**Target User Roles:**
- **Administrator:** Configure verification policies

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Verification required for deposits, withdrawals, transfers
- **Web3 (Non-custodial):** Verification for send, receive, swap per PRD
- **Linked Wallets:** Verification for binding/unbinding

**Security Considerations:**
- Cannot disable all verification methods
- Password policy enforcement
- 2FA requirement for sensitive operations
- Audit trail for policy changes

**Integration Requirements:**
- Configuration database: Policy settings
- Authentication service: Policy enforcement
- User interface: Verification flow handling
- Audit service: Policy change logging

**Technical Architecture Notes:**
- Pattern: Policy pattern for verification rules
- Scalability: Cached policy
- Performance: Policy check <100ms
- Data consistency: Eventually consistent

**Priority Level:** Critical
- **Justification:** PRD Section 一.1 specifies detailed verification requirements

**Dependencies:**
- Authentication infrastructure
- 2FA service (Google Authenticator)

**Success Criteria:**
- Verification policy enforced correctly
- Policy changes take effect within 1 minute
- All changes audited

---

### 11.3 Notification & Alert Rules

#### 11.3.1 Alert Configuration Management

**Feature Name:** Alert Configuration Management

**Description:** Configuration interface for all system alerts including balance thresholds, transaction volume alerts, security alerts, and system health notifications. Supports per-alert recipient configuration and notification channels.

**Business Value:**
- Customizable alerting for operational needs
- Appropriate alert routing to right personnel
- Reduced alert fatigue through tuning
- ROI: Faster incident response

**Target User Roles:**
- **Administrator:** Configure alert rules and recipients

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Alerts for vault balance, transaction anomalies
- **Web3 (Non-custodial):** Alerts for linked address activity
- **Linked Wallets:** No special handling

**Security Considerations:**
- Alert configuration access restricted
- Cannot disable critical alerts
- Recipient verification
- Audit trail for changes

**Integration Requirements:**
- Configuration database: Alert rules
- Monitoring service: Alert evaluation
- Notification service: Alert delivery
- Recipient management: Contact verification

**Technical Architecture Notes:**
- Pattern: Observer pattern with configurable rules
- Scalability: Event-driven alerting
- Performance: Alert triggered within 30 seconds
- Data consistency: Eventually consistent

**Priority Level:** High
- **Justification:** PRD Section 四.1 requires configurable balance alerts

**Dependencies:**
- Monitoring infrastructure
- Notification service (email, SMS, webhook)

**Success Criteria:**
- Alerts triggered correctly
- Delivery within 60 seconds
- All configuration changes audited

---

### 11.4 Localization

#### 11.4.1 Language and Display Settings

**Feature Name:** Language and Display Settings

**Description:** Configuration of supported languages, default language, currency display preferences, timezone settings, and number formatting rules.

**Business Value:**
- Multi-region support
- Consistent user experience
- Localization without code changes
- ROI: Broader user accessibility

**Target User Roles:**
- **Administrator:** Configure localization settings

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Applied to all Web2 interfaces
- **Web3 (Non-custodial):** Applied to Web3 interfaces
- **Linked Wallets:** Consistent across wallet types

**Security Considerations:**
- Language changes should not affect security
- Currency display is informational only
- Audit trail for changes

**Integration Requirements:**
- Configuration database: Localization settings
- All services: Setting consumption
- Translation service: Language strings
- UI framework: Locale support

**Technical Architecture Notes:**
- Pattern: Internationalization (i18n) pattern
- Scalability: Cached settings
- Performance: Locale lookup <50ms
- Data consistency: Eventually consistent

**Priority Level:** Medium
- **Justification:** PRD Section 三 mentions "语言切换：支持中英两种"

**Dependencies:**
- Translation files
- UI framework i18n support

**Success Criteria:**
- Language switching works correctly
- No hardcoded strings
- All changes audited

---

## 12. Role-Based Access Control (RBAC)

### 12.1 Role Definitions

#### 12.1.1 Administrator Role

**Feature Name:** Administrator Role

**Description:** Full system access role with all permissions including user management, configuration changes, approval workflows, and audit log access. Restricted to senior operations personnel.

**Business Value:**
- Clear administrative accountability
- Complete system control capability
- Supports emergency operations
- ROI: Enables efficient platform management

**Target User Roles:**
- **Administrator:** Role holder

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Full control over Web2 operations
- **Web3 (Non-custodial):** Configuration and monitoring for Web3
- **Linked Wallets:** Full visibility and configuration

**Security Considerations:**
- Limited number of administrators
- All actions logged
- Cannot self-elevate permissions
- Regular access review

**Integration Requirements:**
- RBAC database: Role definition
- All services: Permission checking
- Audit service: Action logging
- SSO: Enterprise authentication integration

**Technical Architecture Notes:**
- Pattern: RBAC with attribute-based additions
- Scalability: Cached permissions
- Performance: Permission check <50ms
- Data consistency: Eventually consistent for permissions

**Priority Level:** Critical
- **Justification:** PRD Section 四.6 defines Administrator role with specific permissions

**Dependencies:**
- Authentication infrastructure
- Audit logging

**Success Criteria:**
- Complete permission coverage
- No permission bypass
- All actions audited

---

#### 12.1.2 Finance Role

**Feature Name:** Finance Role

**Description:** Financial operations role with permissions for payroll initiation, withdrawal approval, vault monitoring, and financial reporting. Cannot modify system configuration or user management.

**Business Value:**
- Separation of duties
- Financial process efficiency
- Compliance with control requirements
- ROI: Reduced risk, improved operations

**Target User Roles:**
- **Finance:** Role holder

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Payroll, approvals, vault management
- **Web3 (Non-custodial):** Monitoring and reporting only
- **Linked Wallets:** View linked status for approvals

**Security Considerations:**
- Cannot approve own payroll
- Transaction limits per role
- Audit trail for all financial actions
- Regular permission review

**Integration Requirements:**
- RBAC database: Role definition
- Financial services: Permission integration
- Approval workflow: Role-based routing
- Reporting: Access control

**Technical Architecture Notes:**
- Same as Administrator Role (12.1.1)

**Priority Level:** Critical
- **Justification:** PRD Section 四.6 defines Finance role with specific permissions

**Dependencies:**
- Administrator Role (12.1.1)
- All financial features

**Success Criteria:**
- Finance can perform all specified duties
- Cannot access restricted functions
- All actions audited

---

### 12.2 Permission Management

#### 12.2.1 Granular Permission Configuration

**Feature Name:** Granular Permission Configuration

**Description:** Fine-grained permission management at feature and operation level. Supports custom roles with specific permission combinations beyond standard roles.

**Business Value:**
- Least privilege principle enforcement
- Flexible access control
- Future role expansion support
- ROI: Enhanced security and flexibility

**Target User Roles:**
- **Administrator:** Configure permissions

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Permissions for custodial operations
- **Web3 (Non-custodial):** Permissions for monitoring functions
- **Linked Wallets:** Specific permissions for link management

**Security Considerations:**
- Permission changes require approval
- Cannot create permission that bypasses controls
- Audit trail for all changes
- Regular permission review

**Integration Requirements:**
- RBAC database: Permission storage
- All services: Permission check integration
- Admin UI: Permission configuration
- Audit service: Change logging

**Technical Architecture Notes:**
- Pattern: Hierarchical RBAC with attributes
- Scalability: Cached permission trees
- Performance: Authorization check <50ms
- Data consistency: Eventually consistent

**Priority Level:** High
- **Justification:** PRD Section 四.6 specifies detailed permission system

**Dependencies:**
- Role Definitions (12.1)

**Success Criteria:**
- Granular permissions work correctly
- Custom roles function properly
- All changes audited

---

## 13. Reporting & Analytics

### 13.1 Transaction Reports

#### 13.1.1 Transaction Volume Analytics

**Feature Name:** Transaction Volume Analytics

**Description:** Comprehensive reporting on transaction volumes by chain, token, user segment, and time period. Includes trend analysis, comparison to previous periods, and drill-down capabilities.

**Business Value:**
- Business intelligence on platform usage
- Trend identification for planning
- KPI tracking for operations
- ROI: Data-driven decision making

**Target User Roles:**
- **Finance:** Primary user for business metrics
- **Administrator:** Platform health assessment

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** All transaction types included
- **Web3 (Non-custodial):** Linked address transactions only
- **Linked Wallets:** Separate segment analysis

**Security Considerations:**
- Aggregated data (no individual transaction exposure)
- Report access based on role
- Export controls
- Data retention compliance

**Integration Requirements:**
- Analytics database: Pre-aggregated metrics
- Transaction database: Source data
- Reporting service: Report generation
- Visualization: Charts and dashboards

**Technical Architecture Notes:**
- Pattern: OLAP cube or materialized views
- Scalability: Pre-computed aggregations
- Performance: Report generation <30 seconds
- Data consistency: Eventually consistent

**Priority Level:** High
- **Justification:** Supports Dashboard requirements from PRD Section 四.1

**Dependencies:**
- Transaction History (Feature 4.4)
- Analytics infrastructure

**Success Criteria:**
- Accurate volume calculations
- Report generation <30 seconds
- Historical data available

---

### 13.2 Financial Reconciliation Reports

#### 13.2.1 Daily Reconciliation Report

**Feature Name:** Daily Reconciliation Report

**Description:** Automated daily report comparing off-chain ledger balances with on-chain vault balances. Highlights any discrepancies with detail for investigation.

**Business Value:**
- Ensures financial accuracy
- Early discrepancy detection
- Audit preparation support
- ROI: Prevents financial errors

**Target User Roles:**
- **Finance:** Review and investigation

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Core feature - ledger vs vault
- **Web3 (Non-custodial):** Not applicable
- **Linked Wallets:** No special handling

**Security Considerations:**
- Sensitive financial information
- Automated generation reduces manual error
- Discrepancy alerts to finance
- Historical reports retained

**Integration Requirements:**
- Ledger system: Balance extraction
- Vault service: Balance query
- Report service: Report generation
- Alert service: Discrepancy notification

**Technical Architecture Notes:**
- Pattern: Batch processing with scheduling
- Scalability: Per-token/chain parallel processing
- Performance: Complete within 15 minutes
- Data consistency: Point-in-time snapshot

**Priority Level:** Critical
- **Justification:** Essential for financial integrity

**Dependencies:**
- Ledger System (Feature 8.1)
- Vault Management (Feature 3.1)

**Success Criteria:**
- Report generated daily at scheduled time
- All tokens/chains covered
- Discrepancies flagged immediately

---

### 13.3 Scheduled Reports

#### 13.3.1 Automated Report Distribution

**Feature Name:** Automated Report Distribution

**Description:** Scheduling system for automatic report generation and distribution via email. Supports daily, weekly, and monthly schedules with configurable recipients and report selection.

**Business Value:**
- Reduces manual reporting effort
- Consistent stakeholder communication
- Time-based compliance reporting
- ROI: Significant time savings

**Target User Roles:**
- **Administrator:** Configure schedules and recipients
- **Finance:** Receive and review reports

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Reports on custodial operations
- **Web3 (Non-custodial):** Monitoring reports if configured
- **Linked Wallets:** Included in relevant reports

**Security Considerations:**
- Report data sensitivity classification
- Recipient verification
- Secure email transmission
- Report archive retention

**Integration Requirements:**
- Scheduler service: Job scheduling
- Report service: Report generation
- Email service: Distribution
- Archive: Report storage

**Technical Architecture Notes:**
- Pattern: Job scheduling with retry
- Scalability: Distributed job execution
- Performance: Reports complete before deadline
- Data consistency: Point-in-time for scheduled time

**Priority Level:** Medium
- **Justification:** Operational efficiency improvement

**Dependencies:**
- All reporting features
- Email infrastructure

**Success Criteria:**
- Reports delivered on schedule
- Zero missed scheduled reports
- Recipients configurable without code changes

---

## 14. Customer Support Tools

### 14.1 User Account Lookup

#### 14.1.1 Comprehensive User Search

**Feature Name:** Comprehensive User Search

**Description:** Search interface for support personnel to find users by UID, email, phone number, or wallet address. Returns complete user profile including account status, wallet balances, transaction history, and linked addresses.

**Business Value:**
- Efficient user support capability
- Reduced average handle time
- Complete context for support decisions
- ROI: Improved support efficiency

**Target User Roles:**
- **Administrator:** Full access
- **Finance:** Transaction-related access

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Full balance and transaction visibility
- **Web3 (Non-custodial):** Linked address information only
- **Linked Wallets:** Binding status and history

**Security Considerations:**
- PII access requires appropriate role
- Search queries logged
- Data masking for sensitive fields
- Cannot modify through lookup

**Integration Requirements:**
- User database: Search indexing
- Transaction database: History query
- Wallet service: Balance query
- Audit service: Access logging

**Technical Architecture Notes:**
- Pattern: Aggregator pattern for composite view
- Scalability: Elasticsearch for search
- Performance: Search results <1s
- Data consistency: Eventually consistent for read

**Priority Level:** High
- **Justification:** Essential for customer support operations

**Dependencies:**
- User Management (Feature 2)
- Transaction History (Feature 4.4)

**Success Criteria:**
- Search returns in <1 second
- Complete user context available
- All searches logged

---

### 14.2 Transaction Investigation

#### 14.2.1 Transaction Tracing Tools

**Feature Name:** Transaction Tracing Tools

**Description:** Tools for investigating specific transactions including detailed status, blockchain verification, related transactions, and fund flow tracing for dispute resolution.

**Business Value:**
- Efficient dispute resolution
- Fraud investigation capability
- User complaint handling
- ROI: Reduced escalation rate

**Target User Roles:**
- **Administrator:** Full investigation access
- **Finance:** Transaction-focused investigation

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Off-chain and on-chain tracing
- **Web3 (Non-custodial):** On-chain tracing only
- **Linked Wallets:** Cross-wallet tracing

**Security Considerations:**
- Investigation data is sensitive
- Access logged for audit
- Cannot modify transactions
- Time-limited access for investigations

**Integration Requirements:**
- Transaction database: Detailed records
- Blockchain indexer: On-chain data
- Explorer integration: Verification links
- Case management: Investigation tracking

**Technical Architecture Notes:**
- Pattern: Graph traversal for fund flows
- Scalability: Pre-indexed transaction relationships
- Performance: Trace query <10 seconds
- Data consistency: Eventual consistency acceptable

**Priority Level:** High
- **Justification:** Required for support and compliance

**Dependencies:**
- Transaction History (Feature 4.4)
- Blockchain Explorer Integration (Feature 4.4.2)

**Success Criteria:**
- Complete transaction context available
- Fund flow visible
- Investigation time reduced

---

### 14.3 Account Recovery Assistance

#### 14.3.1 Account Recovery Tools

**Feature Name:** Account Recovery Tools

**Description:** Administrative tools for account recovery including password reset assistance, 2FA reset with enhanced verification, and account unlock procedures. All actions require proper verification and are fully logged.

**Business Value:**
- User self-service limitation recovery
- Secure recovery process
- Reduced support escalations
- ROI: Improved user retention

**Target User Roles:**
- **Administrator:** Full recovery capabilities

**Web2 vs Web3 Implementation Differences:**
- **Web2 (Custodial):** Account recovery restores Web2 wallet access
- **Web3 (Non-custodial):** Cannot recover - user controls keys (per PRD: "助记词丢失则不可找回")
- **Linked Wallets:** Recovery does not affect linking status

**Security Considerations:**
- Enhanced verification for recovery
- Cool-down periods after recovery
- Multiple verification channels
- Complete audit trail

**Integration Requirements:**
- Authentication service: Password reset
- 2FA service: TOTP reset
- Verification service: Identity confirmation
- Notification service: User alerts

**Technical Architecture Notes:**
- Pattern: Workflow with verification gates
- Scalability: Low volume operation
- Performance: Recovery completion <5 minutes
- Data consistency: Strong consistency for credentials

**Priority Level:** High
- **Justification:** Essential user support capability

**Dependencies:**
- Authentication infrastructure
- Verification system

**Success Criteria:**
- Secure recovery process
- User verified before recovery
- All recovery actions logged

---

## Implementation Priority Summary

### Critical (V1.0 Launch Required)
1. Dashboard & Monitoring: All widgets (1.1.1-1.1.4)
2. User Management: User list, status management, identity management, blacklist (2.1, 2.2)
3. Wallet Management: Vault management, child addresses, freeze/unfreeze (3.1)
4. Transaction Management: Deposits, withdrawals, transfers, history (4.1-4.4.1)
5. Payroll: Import, approval, execution, logs (5.1)
6. Withdrawal Approval: Thresholds, linked exemptions, AML (6.1, 6.2)
7. Security: Address blacklist, audit logs (7.1.1, 7.3)
8. Compliance: Ledger, reconciliation, confirmation tracking (8.1, 8.2)
9. Financial Operations: Vault adjustments (9.1)
10. System Configuration: Tokens, networks, verification (11.1, 11.2)
11. RBAC: Administrator and Finance roles (12.1)

### High Priority (V1.1)
- Network health dashboard (1.2)
- Fraud detection (7.2)
- Fee configuration (9.2)
- Transaction limits (9.3)
- DEX configuration (10.1)
- Approval queue SLA (6.3)
- Granular permissions (12.2)
- Transaction analytics (13.1)
- User search and investigation tools (14.1, 14.2)
- Account recovery (14.3)

### Medium Priority (V1.2+)
- Address whitelist (3.3)
- Localization settings (11.4)
- Alert configuration (11.3)
- Scheduled reports (13.3)
- Custom report builder (13.2)
- Dispute resolution (14.3)

---

## Architecture Recommendations

### Technology Stack
- **Backend:** Node.js/TypeScript or Go for high-performance services
- **Database:** PostgreSQL for ACID-compliant ledger, Redis for caching
- **Search:** Elasticsearch for user/transaction search
- **Queue:** Kafka for event streaming, RabbitMQ for task queues
- **Blockchain:** ethers.js (ETH/BNB), TronWeb (TRX)
- **Security:** HSM for key management, Vault for secrets

### Design Patterns
- **CQRS:** Separate read/write models for dashboard and reporting
- **Event Sourcing:** Audit trail and ledger operations
- **Saga:** Multi-step transactions (payroll, withdrawals)
- **Circuit Breaker:** External service resilience (blockchain nodes, AML)

### Security Architecture
- **Key Management:** HSM-backed vault keys, HD derivation for child addresses
- **Access Control:** RBAC with attribute-based policy extension
- **Audit:** Immutable, append-only audit logs
- **Network:** Zero-trust architecture, API gateway with rate limiting

---

## Appendix: PRD Reference Mapping

| PRD Section | Feature Coverage |
|-------------|------------------|
| 四.1 Dashboard | Features 1.1.1-1.1.4 |
| 四.2 财务薪资发放 | Features 5.1.1-5.1.4 |
| 四.3 用户管理 | Features 2.1.1-2.2.1 |
| 四.4 Swap 配置 | Features 10.1.1-10.1.3 |
| 四.5 提现审批流 | Features 6.1.1-6.3.2 |
| 四.6 角色与权限 | Features 12.1.1-12.2.1 |
| 四.7 账本管理 | Features 8.1.1-8.2.1 |
| 一.1 钱包账户体系 | Features 3.1, 3.2 |
| 三 特殊场景处理 | Features 2.2.1, 7.1.1 |
