# REST API Specification

Comprehensive API specification for the AI Payment Fraud Intelligence platform.

---

## Authentication & Security

### OAuth 2.0 Bearer Token

**Standard**:
- Protocol: OAuth 2.0 (RFC 6749)
- Grant Type: Client Credentials (B2B integrations)
- Bearer Token format: RFC 6750

**Request Header**:
```
Authorization: Bearer {access_token}
```

**Token Validation**:
- Verify JWT signature
- Check token expiration (default TTL: 3600 seconds)
- Verify scopes match requested operation
- Check client not revoked

**Example**:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Rate Limiting

**Purpose**: Prevent abuse, ensure fair usage

**Default Limits**:
- Standard endpoints: 10,000 requests/minute
- Evaluate endpoint: 100,000 requests/minute (critical)
- Batch endpoints: 1,000 requests/minute

**Response Headers**:
```
X-RateLimit-Limit: 10000
X-RateLimit-Remaining: 9998
X-RateLimit-Reset: 1642377600
```

**Behavior**:
- Track by API key + IP
- Sliding window (1-minute buckets)
- Return 429 (Too Many Requests) when exceeded

**Example**:
```
Request 10,001th request
↓
Response: 429 Too Many Requests
Headers:
  X-RateLimit-Remaining: 0
  X-RateLimit-Reset: 1642377660
  Retry-After: 60
```

---

## Core Endpoints

### 1. POST /transactions/evaluate

**Purpose**: Get real-time fraud assessment for a transaction

**Endpoint**: `https://api.fraud-intelligence.com/v1/transactions/evaluate`

**HTTP Method**: POST

**Authentication**: Required (Bearer token)

**Request Body**:

```json
{
  "transaction_id": "txn_abc123",
  "amount": 99.99,
  "currency": "USD",
  "account_id": "acc_xyz789",
  "merchant_id": "mch_def456",
  "card_token": "tok_abc123",
  "device_id": "dev_xyz789",
  "ip_address": "192.168.1.1",
  "metadata": {
    "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",
    "referer": "checkout.example.com"
  }
}
```

**Response (200 OK)**:

```json
{
  "transaction_id": "txn_abc123",
  "decision": "APPROVE",
  "fraud_score": 0.28,
  "confidence": 0.92,
  "latency_ms": 87,
  "factors": [
    "low_fraud_score",
    "trusted_account",
    "regular_merchant"
  ],
  "decision_at": "2024-01-15T10:23:45Z"
}
```

**Response (400 Bad Request)**:

```json
{
  "error": "INVALID_AMOUNT",
  "message": "Amount must be positive number",
  "code": "ERR_400_001"
}
```

**SLA**:
- P50: 45ms
- P95: 85ms
- P99: 120ms

**Rate Limit**: 100,000 req/min

---

### 2. GET /transactions/{transaction_id}

**Purpose**: Get decision details for previous transaction

**Endpoint**: `https://api.fraud-intelligence.com/v1/transactions/{transaction_id}`

**HTTP Method**: GET

**Path Parameters**:
- `transaction_id` (string, required): Unique transaction ID

**Response (200 OK)**:

```json
{
  "transaction_id": "txn_abc123",
  "decision": "APPROVE",
  "fraud_score": 0.28,
  "confidence": 0.92,
  "decision_at": "2024-01-15T10:23:45Z",
  "components": {
    "xgboost": 0.35,
    "lstm": 0.22,
    "anomaly": 0.15,
    "rules": false
  }
}
```

**Response (404 Not Found)**:

```json
{
  "error": "NOT_FOUND",
  "message": "Transaction not found",
  "code": "ERR_404_001"
}
```

**SLA**:
- P50: 20ms
- P95: 40ms
- P99: 80ms

**Rate Limit**: 10,000 req/min

---

### 3. POST /transactions/batch

**Purpose**: Evaluate multiple transactions in batch

**Endpoint**: `https://api.fraud-intelligence.com/v1/transactions/batch`

**HTTP Method**: POST

**Request Body**:

```json
{
  "transactions": [
    {
      "transaction_id": "txn_001",
      "amount": 50.00,
      "currency": "USD",
      "account_id": "acc_001",
      "merchant_id": "mch_001",
      "card_token": "tok_001"
    },
    {
      "transaction_id": "txn_002",
      "amount": 150.00,
      "currency": "USD",
      "account_id": "acc_002",
      "merchant_id": "mch_002",
      "card_token": "tok_002"
    }
  ]
}
```

**Response (200 OK)**:

```json
{
  "results": [
    {
      "transaction_id": "txn_001",
      "decision": "APPROVE",
      "fraud_score": 0.15
    },
    {
      "transaction_id": "txn_002",
      "decision": "REVIEW",
      "fraud_score": 0.55
    }
  ],
  "processed": 2,
  "failed": 0
}
```

**SLA**:
- P50: 150ms
- P95: 400ms
- P99: 800ms

**Rate Limit**: 1,000 req/min

---

### 4. GET /accounts/{account_id}/risk-profile

**Purpose**: Get aggregated risk profile for account

**Endpoint**: `https://api.fraud-intelligence.com/v1/accounts/{account_id}/risk-profile`

**HTTP Method**: GET

**Response (200 OK)**:

```json
{
  "account_id": "acc_xyz789",
  "risk_score": 0.18,
  "risk_level": "LOW",
  "stats": {
    "total_transactions": 1245,
    "fraud_transactions": 2,
    "approval_rate": 0.98,
    "avg_transaction": 125.50,
    "account_age_days": 387,
    "devices_seen": 3,
    "merchants_seen": 542
  },
  "trend": "stable",
  "last_updated": "2024-01-15T10:00:00Z"
}
```

**SLA**:
- P50: 30ms
- P95: 60ms
- P99: 100ms

**Rate Limit**: 10,000 req/min

---

### 5. GET /merchants/{merchant_id}/risk-profile

**Purpose**: Get risk profile for merchant

**Endpoint**: `https://api.fraud-intelligence.com/v1/merchants/{merchant_id}/risk-profile`

**HTTP Method**: GET

**Response (200 OK)**:

```json
{
  "merchant_id": "mch_def456",
  "name": "TechStore Inc",
  "risk_score": 0.12,
  "risk_level": "LOW",
  "stats": {
    "transaction_volume": 50000,
    "chargeback_rate": 0.002,
    "fraud_rate": 0.001,
    "avg_amount": 75.00,
    "customer_satisfaction": 4.8
  }
}
```

**SLA**:
- P50: 25ms
- P95: 50ms
- P99: 90ms

**Rate Limit**: 10,000 req/min

---

### 6. POST /transactions/{transaction_id}/feedback

**Purpose**: Submit feedback on transaction decision (post-resolution)

**Endpoint**: `https://api.fraud-intelligence.com/v1/transactions/{transaction_id}/feedback`

**HTTP Method**: POST

**Request Body**:

```json
{
  "feedback_type": "CHARGEBACK",
  "actual_result": "FRAUD",
  "feedback_at": "2024-01-20T15:30:00Z",
  "notes": "Customer disputed transaction"
}
```

**Feedback Types**:
- `CHARGEBACK`: Chargeback received from bank
- `MANUAL_REVIEW`: Result of manual fraud team review
- `CUSTOMER_REPORT`: Customer reported fraud
- `CUSTOMER_DENIAL`: Customer denies fraud (false positive)

**Response (202 Accepted)**:

```json
{
  "transaction_id": "txn_abc123",
  "feedback_accepted": true,
  "processing_id": "fb_xyz123"
}
```

**SLA**:
- P50: 100ms
- P95: 200ms
- P99: 300ms

**Rate Limit**: 10,000 req/min

---

### 7. GET /transactions

**Purpose**: List recent transactions with filters

**Endpoint**: `https://api.fraud-intelligence.com/v1/transactions`

**HTTP Method**: GET

**Query Parameters**:

```
GET /transactions?account_id=acc_001&decision=APPROVE&limit=100&offset=0
```

- `account_id` (string, optional): Filter by account
- `decision` (string, optional): Filter by decision (APPROVE, REVIEW, DECLINE)
- `fraud_score_min` (number, optional): Minimum fraud score [0, 1]
- `fraud_score_max` (number, optional): Maximum fraud score [0, 1]
- `limit` (number, optional): Results per page (default: 100, max: 1000)
- `offset` (number, optional): Pagination offset (default: 0)

**Response (200 OK)**:

```json
{
  "transactions": [
    {
      "transaction_id": "txn_abc123",
      "account_id": "acc_xyz789",
      "amount": 99.99,
      "currency": "USD",
      "decision": "APPROVE",
      "fraud_score": 0.28,
      "created_at": "2024-01-15T10:23:45Z"
    }
  ],
  "total": 5000,
  "limit": 100,
  "offset": 0
}
```

**SLA**:
- P50: 200ms
- P95: 500ms
- P99: 1000ms

**Rate Limit**: 10,000 req/min

---

### 8. POST /config/thresholds

**Purpose**: Update decision thresholds (admin only)

**Endpoint**: `https://api.fraud-intelligence.com/v1/config/thresholds`

**HTTP Method**: POST

**Permissions**: Admin only

**Request Body**:

```json
{
  "account_type": "standard",
  "approve_threshold": 0.30,
  "decline_threshold": 0.70,
  "effective_at": "2024-01-16T00:00:00Z"
}
```

**Response (200 OK)**:

```json
{
  "config_id": "cfg_abc123",
  "updated_at": "2024-01-15T10:23:45Z",
  "effective_at": "2024-01-16T00:00:00Z"
}
```

**SLA**:
- P50: 50ms
- P95: 100ms
- P99: 150ms

**Rate Limit**: 100 req/min (admin-only)

---

## Error Codes

### Standard HTTP Status Codes

| Status | Meaning | Description |
|--------|---------|-------------|
| 200 | OK | Request successful |
| 202 | Accepted | Request accepted for processing (async) |
| 400 | Bad Request | Invalid request parameters |
| 401 | Unauthorized | Invalid or missing authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Duplicate or conflicting resource |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server-side error |
| 503 | Service Unavailable | Service temporarily down |

### Application-Level Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| ERR_400_001 | 400 | Invalid request parameters |
| ERR_400_002 | 400 | Missing required field |
| ERR_400_003 | 400 | Invalid currency code |
| ERR_401_001 | 401 | Invalid authentication token |
| ERR_401_002 | 401 | Token expired |
| ERR_403_001 | 403 | Insufficient permissions |
| ERR_404_001 | 404 | Transaction not found |
| ERR_404_002 | 404 | Account not found |
| ERR_409_001 | 409 | Duplicate transaction ID |
| ERR_429_001 | 429 | Rate limit exceeded |
| ERR_500_001 | 500 | Internal server error |
| ERR_503_001 | 503 | Service unavailable |

### Error Response Format

```json
{
  "error": "INVALID_AMOUNT",
  "message": "Amount must be positive number",
  "code": "ERR_400_001",
  "details": {
    "field": "amount",
    "value": -50.00,
    "constraint": "positive_number"
  }
}
```

---

## Webhooks

**Purpose**: Real-time notifications for transaction events

**Event Types**:

```
transaction.approved
├─ Triggered: Decision is APPROVE
├─ Payload: Full transaction + decision
└─ Recipient: Merchant/processor

transaction.declined
├─ Triggered: Decision is DECLINE
├─ Payload: Transaction + fraud score
└─ Recipient: Merchant/processor

transaction.reviewed
├─ Triggered: Decision is REVIEW
├─ Payload: Transaction + assigned analyst
└─ Recipient: Internal review queue

fraud.detected
├─ Triggered: Fraud score > 0.8 (regardless of decision)
├─ Payload: Transaction + fraud indicators
└─ Recipient: Fraud team
```

**Webhook Format**:

```json
{
  "event_id": "evt_123456",
  "event_type": "transaction.approved",
  "timestamp": "2024-01-15T10:23:45Z",
  "transaction_id": "txn_abc123",
  "decision": "APPROVE",
  "fraud_score": 0.28
}
```

**Retry Strategy**:
- Immediate retry on failure
- Exponential backoff: 1s, 2s, 4s, 8s, 16s
- Max 5 retry attempts (total 31 seconds)
- Webhook timeout: 30 seconds

**Reliability**:
- At-least-once delivery
- Duplicate handling: Use event_id for idempotency
- Ordering: Per-transaction ordered delivery

---

## Backward Compatibility

### Versioning Strategy

**URL Versioning**:
```
/v1/transactions/evaluate
/v2/transactions/evaluate (future)
```

**Accept Header Versioning**:
```
Accept: application/vnd.fraud-api.v1+json
```

### Deprecation Policy

- **Notice Period**: 12 months before removal
- **Grace Period**: 6-month overlap (both versions active)
- **Communication**: Email + API dashboard announcement
- **Tools**: Migration guides and SDK updates provided

### Breaking Changes

- New major version (v2) for breaking changes
- Old version (v1) supported for 18 months minimum
- Automatic redirects from old to new (with deprecation header)

---

## SDK & Client Libraries

**Available Languages**:
- Python: `pip install fraud-intelligence-sdk`
- Node.js: `npm install @fraud-ai/sdk`
- Go: `go get github.com/fraud-ai/sdk-go`
- Java: Maven/Gradle support

**Example (Python)**:

```python
from fraud_intelligence import Client

client = Client(api_key="sk_test_123456")

response = client.transactions.evaluate(
    transaction_id="txn_abc123",
    amount=99.99,
    currency="USD",
    account_id="acc_xyz789",
    merchant_id="mch_def456",
    card_token="tok_abc123"
)

print(f"Decision: {response.decision}")
print(f"Fraud Score: {response.fraud_score}")
```

---

## Testing & Sandbox

### Sandbox Environment

- **URL**: `https://sandbox.fraud-intelligence.com/v1`
- **Purpose**: Test without affecting production
- **Data**: Isolated from production

**Test Cards**:
- `4000000000000002`: Always APPROVE
- `4000000000000010`: Always DECLINE
- `4000000000000028`: Always REVIEW

---

## SLA & Performance

### Service Level Agreement

| Metric | Target |
|--------|--------|
| Availability | 99.99% |
| P50 Latency | 45ms |
| P95 Latency | 85ms |
| P99 Latency | 120ms |
| Error Rate | <0.1% |

### Penalties (if not met)

- 99.95-99.99% uptime: 5% credit
- 99.0-99.95% uptime: 10% credit
- <99.0% uptime: 25% credit

---

**API Version**: 1.0
**Last Updated**: February 2026
**Status**: Production Ready