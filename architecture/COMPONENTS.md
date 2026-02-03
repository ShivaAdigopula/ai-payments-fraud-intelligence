# 8-Component Architecture Specifications

## Component Overview

The system consists of 8 independent, scalable components working together to make fraud decisions:

```
Transaction → [API Gateway] → [Feature Engineering] → [Fraud Detection]
                                                             ↓
                                                      [Risk Scoring]
                                                             ↓
                                                      [Decision Engine]
                                                             ↓
                    [Notification Service] ← ← ← ← ← ← ← ←┤
                                                             ├→ [Monitoring]
                    [Rules Engine] ← ← ← ← ← ← ← ← ← ← ← ←┤
                                                             └→ Cache/DB
```

---

## Component 1: API Gateway

### Purpose
Validate requests, authenticate clients, rate limit traffic, route to fraud detection pipeline.

### Responsibilities
- Request validation (schema, data types)
- Authentication (OAuth 2.0 token verification)
- Rate limiting (10k-100k req/min depending on tier)
- Request/response logging
- Load balancing to fraud detection services

### Interface

**Input**:
```json
{
  "transaction_id": "txn_123",
  "account_id": "acc_456",
  "amount": 99.99,
  "currency": "USD",
  "merchant_id": "mch_789",
  "card_token": "tok_abc",
  "device_id": "dev_xyz",
  "ip_address": "192.168.1.1"
}
```

**Output**:
```json
{
  "transaction_id": "txn_123",
  "decision": "APPROVE",
  "fraud_score": 0.28,
  "confidence": 0.92,
  "latency_ms": 87
}
```

### SLA Targets
- **Throughput**: 100,000+ requests/second
- **Latency P50**: 5ms
- **Latency P99**: 40ms
- **Uptime**: 99.99%
- **Error Rate**: <0.1%

### Technologies
- FastAPI or Go (HTTP server)
- OAuth 2.0 libraries
- Rate limiting middleware (Redis)
- Load balancer (NGINX, HAProxy)

### Dependencies
- Redis (rate limit counters)
- Identity provider (OAuth)

### Scaling
- Horizontal scaling: Stateless design
- Auto-scale on CPU/memory
- CDN for static content

---

## Component 2: Feature Engineering

### Purpose
Compute real-time and near-real-time features for ML models.

### Feature Categories

**Real-Time Features** (computed immediately):
- Transaction amount
- Merchant ID
- Card network
- IP address
- Device ID

**Near-Real-Time Features** (1-24 hour aggregates):
- Velocity counts (transactions in last 5, 15, 60 minutes)
- Amount aggregates (sum/avg/max in periods)
- Device diversity (number of devices used)
- Merchant diversity (number of merchants)

**Historical Features** (pre-computed):
- Account age (days since creation)
- Lifetime transaction count
- Fraud rate for account
- Device history
- Merchant history

### Computation Strategy

```
Real-Time (cache lookup) ← 10ms
    ↓
Near-Real-Time (Redis aggregations) ← 30ms
    ↓
Historical (database lookup) ← 20ms
    ↓
Feature vector → Normalization → Feature store
```

### Input/Output

**Input**: Transaction event with account, merchant, card, device info

**Output**: 50+ feature vector for ML models

### SLA Targets
- **Latency P99**: 80ms
- **Feature Coverage**: 99%+ (missing features get defaults)
- **Uptime**: 99.95%

### Technologies
- Python/Go for feature computation
- Redis for caching
- Feature store (Feast, Tecton)
- PostgreSQL for historical lookups

### Dependencies
- Redis Cluster
- PostgreSQL
- Feature store

### Scaling
- Horizontal: Multiple workers
- Cache warming: Pre-compute common queries
- Batch processing: Nightly aggregations

---

## Component 3: Fraud Detection

### Purpose
Apply 4-model ensemble to detect fraudulent transactions.

### Models

**XGBoost Classifier**:
- 50+ engineered features
- Gradient boosting (supervised)
- 96% precision, 90% recall
- ~5-10ms inference

**LSTM Neural Network**:
- Sequential model of last 30 transactions
- 128→64 units with attention
- 94% precision, 96% recall
- ~15-20ms inference

**Isolation Forest**:
- Anomaly detection on normal patterns
- Detects novel fraud vectors
- ~5-10ms inference

**Rules Engine**:
- 20+ hard-coded rules
- Velocity, impossible travel, card testing
- Binary decision (fraud flag or not)
- <1ms evaluation

### Ensemble Scoring

```
Fraud Score = (XGBoost × 0.50) + (LSTM × 0.30) + 
              (Anomaly × 0.15) + (Rules × 0.05)

Result: Score in [0, 1] range
```

### Inference

**Latency Budget**: <50ms

```
Feature Vector (from Feature Eng)
    ↓
Parallel Inference:
├─ XGBoost (5ms)
├─ LSTM (18ms)
├─ Anomaly (8ms)
└─ Rules (1ms)
    ↓
Ensemble Score (100ms total with feature eng)
```

### Input/Output

**Input**: 50+ feature vector

**Output**:
```json
{
  "xgboost_score": 0.78,
  "lstm_score": 0.72,
  "anomaly_score": 0.65,
  "rules_flag": false,
  "ensemble_score": 0.76
}
```

### SLA Targets
- **Latency P99**: 50ms
- **Throughput**: 100,000+ tps
- **Model Accuracy**: 98%+
- **Uptime**: 99.99%

### Technologies
- XGBoost: scikit-learn package
- LSTM: TensorFlow/PyTorch
- Anomaly: scikit-learn Isolation Forest
- Model Serving: TensorFlow Serving or KServe
- GPU support for neural networks

### Dependencies
- Model registry (versioned models)
- Monitoring for model drift

### Scaling
- Model replicas for load distribution
- GPU instances for neural network inference
- Model caching in memory

---

## Component 4: Risk Scoring

### Purpose
Calculate account and merchant risk scores used for decision thresholds.

### Account Risk Score

**Factors**:
- Account age (newer = higher risk)
- Fraud history (% of transactions)
- Transaction patterns
- Device consistency
- Geographic patterns

**Calculation**:
```
account_risk = (account_age_factor × 0.3) + 
               (fraud_history_factor × 0.4) +
               (pattern_anomaly_factor × 0.3)
```

**Range**: [0, 1] where 0=lowest risk, 1=highest risk

### Merchant Risk Score

**Factors**:
- Merchant category (industry risk)
- Chargeback rate
- Transaction volume
- Customer satisfaction
- Industry violations

**Calculation**:
```
merchant_risk = (category_factor × 0.2) +
                (chargeback_rate_factor × 0.4) +
                (volume_factor × 0.2) +
                (satisfaction_factor × 0.2)
```

### Usage

Risk scores modify decision thresholds:
```
base_threshold = 0.50

if account_risk > 0.8:  # High risk account
    threshold -= 0.10  # Stricter (lower threshold = more declines)
    
if merchant_risk > 0.8:  # High risk merchant
    threshold -= 0.05
```

### Input/Output

**Input**: 
- Account ID
- Merchant ID
- Transaction amount

**Output**:
```json
{
  "account_risk_score": 0.25,
  "account_risk_level": "LOW",
  "merchant_risk_score": 0.18,
  "merchant_risk_level": "LOW"
}
```

### SLA Targets
- **Latency P99**: 30ms
- **Uptime**: 99.95%
- **Freshness**: Updated hourly

### Technologies
- Python/Go for calculations
- Redis for caching scores
- PostgreSQL for historical data

### Dependencies
- Account service
- Merchant service
- Historical transaction database

### Scaling
- Horizontal: Stateless computation
- Caching: Pre-computed scores updated hourly

---

## Component 5: Decision Engine

### Purpose
Combine fraud score and risk scores to make APPROVE/REVIEW/DECLINE decision.

### Decision Logic

```python
def make_decision(fraud_score, account_risk, merchant_risk):
    # Base thresholds
    approve_threshold = 0.30
    decline_threshold = 0.70
    
    # Risk modifiers
    threshold_adjust = (account_risk × 0.10) + (merchant_risk × 0.05)
    
    # Apply modifiers
    approve_threshold -= threshold_adjust
    decline_threshold -= threshold_adjust
    
    # Make decision
    if fraud_score < approve_threshold:
        return "APPROVE"
    elif fraud_score > decline_threshold:
        return "DECLINE"
    else:
        return "REVIEW"
```

### Decision Matrix

| Fraud Score | Account Risk | Merchant Risk | Decision |
|-------------|--------------|---------------|----------|
| 0.15 | LOW | LOW | APPROVE |
| 0.45 | LOW | LOW | REVIEW |
| 0.75 | LOW | LOW | DECLINE |
| 0.35 | HIGH | HIGH | DECLINE |
| 0.25 | LOW | LOW | APPROVE |

### Decision Metadata

```json
{
  "transaction_id": "txn_123",
  "decision": "APPROVE",
  "fraud_score": 0.28,
  "confidence": 0.92,
  "thresholds": {
    "approve_threshold": 0.35,
    "decline_threshold": 0.65,
    "fraud_score": 0.28
  },
  "decision_factors": [
    "low_fraud_score",
    "trusted_account",
    "regular_merchant"
  ],
  "latency_ms": 87,
  "decision_timestamp": "2024-01-15T10:23:45Z"
}
```

### Input/Output

**Input**:
- Fraud score from Component 3
- Account risk from Component 4
- Merchant risk from Component 4

**Output**:
- Decision (APPROVE/REVIEW/DECLINE)
- Confidence score
- Metadata for logging/monitoring

### SLA Targets
- **Latency P99**: 20ms
- **Uptime**: 99.99%
- **Accuracy**: Track actual vs predicted

### Technologies
- Python/Go for decision logic
- Rules engine for business logic

### Dependencies
- None (combines outputs from other components)

### Scaling
- Horizontal: Stateless computation
- No scaling issues (very fast)

---

## Component 6: Notification Service

### Purpose
Send notifications to customer and merchant about transaction decisions.

### Notification Types

**For APPROVE**:
- Optional: Email/SMS confirmation if high value
- Merchant: Order confirmation

**For DECLINE**:
- SMS: "Your transaction was declined. Contact support."
- Email: Detailed explanation
- Merchant: "Transaction declined"

**For REVIEW**:
- SMS: "Your transaction is being reviewed. Check back in 5-15 min."
- Email: What happened, timeline, next steps

### Channels
- SMS: Twilio, AWS SNS
- Email: SendGrid, AWS SES
- In-app: App notification service
- Webhooks: Merchant notifications

### Input/Output

**Input**:
- Transaction ID
- Decision (APPROVE/REVIEW/DECLINE)
- Customer ID
- Merchant ID

**Output**: Confirmation of notification sent

### SLA Targets
- **Latency**: <100ms (async, not blocking)
- **Delivery Rate**: 99%+ (retry on failures)
- **Uptime**: 99.95%

### Technologies
- Queue system: Apache Kafka, RabbitMQ
- Async workers: Celery, RQ
- Notification providers: Twilio, SendGrid

### Dependencies
- Kafka/message queue
- Notification providers

### Scaling
- Horizontal: Multiple worker processes
- Queue-based: Decoupled from decision engine

---

## Component 7: Rules Engine

### Purpose
Enforce hard business rules (velocity limits, impossible travel, etc.).

### Rules

**Velocity Rules**:
- Max 5 transactions per card in 5 minutes
- Max $5,000 per card daily
- Max 3 attempts to same merchant per hour

**Impossible Travel**:
- Flag if last transaction in different country
- Time gap insufficient for physical travel (>500km/hour)

**Card Testing Pattern**:
- Multiple small amounts ($1-10)
- Different merchants
- Within 30 minutes
- Action: DECLINE or REVIEW

**Whitelist/Blacklist**:
- Known fraud merchants: Auto-decline
- Trusted accounts: Can override REVIEW
- Verified merchants: Reduce friction

**Account Status**:
- New account (<7 days): Stricter checks
- Suspended account: Auto-decline
- VIP account: Relaxed limits

### Rule Format

```yaml
rules:
  - name: "max_daily_amount"
    condition: "same_card AND amount_in_24h > 5000"
    action: "DECLINE"
    
  - name: "card_testing"
    condition: "amount < 10 AND unique_merchants > 3 AND time_window < 30min"
    action: "DECLINE"
    
  - name: "impossible_travel"
    condition: "last_txn_country != current_country AND time_gap < travel_time"
    action: "REVIEW"
```

### Input/Output

**Input**:
- Transaction
- Account history
- Merchant info
- Last transactions

**Output**:
```json
{
  "rules_triggered": [
    "card_testing_pattern"
  ],
  "rules_action": "DECLINE",
  "rules_confidence": 0.95
}
```

### SLA Targets
- **Latency**: <10ms
- **Uptime**: 99.99%
- **Accuracy**: Rule coverage should catch 80%+ of detected fraud

### Technologies
- Rule engine: Drools, Easy Rules
- Python/Go for custom rules

### Dependencies
- Transaction history database
- Merchant database

### Scaling
- Horizontal: Stateless
- Caching: Pre-compute rule data

---

## Component 8: Monitoring & Alerting

### Purpose
Track system health, detect anomalies, alert on issues.

### Metrics to Monitor

**System Metrics**:
- Latency (P50, P95, P99)
- Throughput (transactions/second)
- Error rates
- Uptime

**Fraud Metrics**:
- Fraud detection rate (recall)
- False positive rate (precision)
- Approval rate
- Decline rate
- Review rate

**Business Metrics**:
- Fraud loss
- Customer satisfaction
- Appeal rate

### Alerts

**Critical** (page on-call):
- Latency P99 > 150ms
- Error rate > 0.5%
- Fraud detection drops below 95%
- System uptime < 99.9%

**Warning** (email):
- Latency P95 increasing
- Fraud rate trending up
- Model drift detected
- Review queue > 10,000

### Dashboards

**Operations Dashboard**:
- Real-time latency and throughput
- Error rates and uptime
- Component health status

**Fraud Dashboard**:
- Fraud detection accuracy
- Approval/decline/review breakdown
- Fraud loss trending

**Business Dashboard**:
- Approval rate vs target
- False decline rate
- Customer satisfaction
- Revenue impact

### Input/Output

**Input**: Metrics from all components

**Output**:
- Dashboards in Grafana
- Alerts via PagerDuty/Slack/Email
- Reports (daily, weekly, monthly)

### SLA Targets
- **Alert Latency**: <1 minute
- **Dashboard Update**: <10 second refresh
- **Reporting**: Automated daily

### Technologies
- Metrics: Prometheus
- Visualization: Grafana
- Alerting: AlertManager, PagerDuty
- Logging: ELK Stack or CloudWatch
- APM: Datadog, New Relic

### Dependencies
- All components send metrics

### Scaling
- Horizontal: Multiple monitoring instances
- Aggregation: Time-series database

---

## Component Interactions

### Happy Path (APPROVE)

```
1. API Gateway receives transaction
   ↓
2. Feature Engineering computes features (parallel)
3. Fraud Detection scores (parallel with Risk Scoring)
   ↓
4. Risk Scoring provides account/merchant risk
   ↓
5. Decision Engine combines scores → APPROVE
   ↓
6. Notification Service sends confirmation
   ↓
7. Monitoring tracks metrics
```

**Total Latency**: ~90ms

### Review Path

```
Same as above, but Decision Engine routes to REVIEW
   ↓
Notification Service sends review queue message
   ↓
Manual Review System (human) receives task
   ↓
Rules Engine validates against business rules
   ↓
Monitoring tracks review queue health
```

**Queue SLA**: <15 minutes

### Decline Path

```
Same as APPROVE, but Decision Engine routes to DECLINE
   ↓
Notification Service sends decline message
   ↓
Rules Engine confirms hard rules triggered
   ↓
Monitoring alerts if false decline detected
   ↓
Customer appeal process available (7 days)
```

---

## Deployment Strategy

### Canary Deployment
1. Deploy new version to 1 canary pod
2. Route 1% traffic to canary (99% to stable)
3. Monitor metrics for 2-3 hours
4. If healthy: proceed to Phase 1
5. If issues: automatic rollback

### Progressive Rollout
- Phase 1: 10% traffic (12 hours)
- Phase 2: 50% traffic (24 hours)
- Full: 100% traffic (if metrics good)

### Rollback
- Automatic: If error rate > 0.5% for 5 min
- Manual: On-call engineer can trigger
- Instant: Switch traffic to previous version

---

**Specification Version**: 1.0
**Last Updated**: February 2026
**Status**: Production Ready