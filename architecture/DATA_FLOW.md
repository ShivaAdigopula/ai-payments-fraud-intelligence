# Data Flow Architecture

## Real-Time Transaction Processing

### Timeline: Transaction Arrival to Decision (T+0 to T+105ms)

```
T+0ms    ────────────────────────────────────────────────────────────
         │ Transaction arrives at API Gateway
         │ {transaction_id, amount, account_id, merchant_id, ...}
         │
T+5ms    │ API Gateway validates & authenticates
         │ ✓ Request format valid
         │ ✓ OAuth token verified
         │ ✓ Rate limit check passed
         │ → Routes to fraud detection service
         │
T+10ms   ├─ [PARALLEL PHASE STARTS]
         │ │
         │ ├─ Feature Engineering starts (branch A)
         │ │  └─ Query Redis for real-time features (cache)
         │ │  └─ Query PostgreSQL for historical features
         │ │  └─ Aggregate near-real-time metrics
         │ │
         │ ├─ Risk Scoring starts (branch B)
         │ │  └─ Lookup account risk profile (cache)
         │ │  └─ Lookup merchant risk profile (cache)
         │ │  └─ Calculate risk modifiers
         │ │
         │ ├─ Rules Engine starts (branch C)
         │ │  └─ Check velocity rules
         │ │  └─ Check impossible travel
         │ │  └─ Check card testing pattern
         │ │  └─ Check whitelist/blacklist
         │ │
T+50ms   │
         │ Feature Engineering completes (branch A done)
         │ → 50+ feature vector ready
         │
         │ Risk Scoring completes (branch B done)
         │ → Account risk: 0.25, Merchant risk: 0.18
         │
         │ Rules Engine completes (branch C done)
         │ → No rules triggered (rules_flag = false)
         │
T+55ms   │ Fraud Detection starts (parallel inference)
         │ ├─ XGBoost inference (5ms)
         │ ├─ LSTM inference (18ms)
         │ ├─ Isolation Forest (8ms)
         │ └─ Ensemble scoring (1ms)
         │
T+75ms   │ Fraud Detection completes
         │ → Fraud score: 0.76
         │ → Scores: [XGB: 0.78, LSTM: 0.72, Anom: 0.65, Rules: 0]
         │
T+78ms   │ Decision Engine processes
         │ ├─ Apply thresholds
         │ ├─ Adjust for account/merchant risk
         │ ├─ Generate decision metadata
         │ └─ Decision: APPROVE (fraud_score 0.28 < threshold 0.35)
         │
T+87ms   │ Response prepared with decision
         │ {
         │   "transaction_id": "txn_123",
         │   "decision": "APPROVE",
         │   "fraud_score": 0.28,
         │   "confidence": 0.92,
         │   "latency_ms": 87
         │ }
         │
T+90ms   │ Response sent to API client
         │ → Decision available to merchant/processor
         │
T+90-105ms (async):
         ├─ Notification Service sends confirmation email/SMS
         ├─ Transaction logged to PostgreSQL
         ├─ Metrics published to Prometheus
         ├─ Event published to Kafka for downstream processing
         └─ Monitoring updated
```

### Parallel Processing Details

**Branch A: Feature Engineering** (10-50ms)

```
Feature Request
    ↓
┌─────────────────────────┐
│ Real-Time Features      │
├─────────────────────────┤
│ Amount                  │ From request
│ Merchant ID             │ From request  
│ Card network            │ From request
│ IP address              │ From request
│ Device ID               │ From request
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│ Near-Real-Time Features │
├─────────────────────────┤
│ Redis Lookup (2ms)      │
│ - Velocity counts       │
│ - Amount aggregates     │
│ - Device diversity      │
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│ Historical Features     │
├─────────────────────────┤
│ PostgreSQL Lookup (5ms) │
│ - Account age           │
│ - Fraud history         │
│ - Device history        │
└─────────────────────────┘
    ↓
Feature Vector (50+ features)
Normalization
→ Ready for ML models
```

**Branch B: Risk Scoring** (10-30ms)

```
Account & Merchant lookup
    ↓
┌─────────────────────────┐
│ Account Risk Profile    │
├─────────────────────────┤
│ Redis Cache (2ms)       │
│ - Account age factor    │
│ - Fraud history %       │
│ - Pattern anomaly       │
│ → Risk Score: 0.25      │
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│ Merchant Risk Profile   │
├─────────────────────────┤
│ Redis Cache (2ms)       │
│ - Category risk         │
│ - Chargeback rate       │
│ - Volume factor         │
│ → Risk Score: 0.18      │
└─────────────────────────┘
```

**Branch C: Rules Engine** (10-15ms)

```
Transaction + History
    ↓
Check Rules (parallel):
├─ Velocity rule: 0 violations
├─ Impossible travel: No
├─ Card testing: No
├─ Whitelist: No
└─ Blacklist: No
    ↓
→ Rules Flag: false
```

### Request Flow Diagram

```
┌─────────────────────────────────────────────────────────┐
│                   API Gateway                           │
│  • Validate schema                                      │
│  • Verify OAuth token                                   │
│  • Rate limit check                                     │
│  [Latency: 5ms]                                         │
└────────────────┬────────────────────────────────────────┘
                 │
                 │ PARALLEL PROCESSING
        ┌────────┴────────┬──────────────┐
        │                 │              │
   [Branch A]        [Branch B]     [Branch C]
   Features          Risk           Rules
   (10-50ms)        (10-30ms)     (10-15ms)
        │                 │              │
        └────────┬────────┴──────────────┘
                 │
┌────────────────▼────────────────────────────────────────┐
│              Fraud Detection (Parallel)                 │
│  ├─ XGBoost (5ms): 0.78                                 │
│  ├─ LSTM (18ms): 0.72                                   │
│  ├─ Isolation Forest (8ms): 0.65                        │
│  └─ Ensemble Score (1ms): 0.76 → 0.28 (after rules)   │
│  [Total: 20ms for all models]                           │
└────────────────┬────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────┐
│              Decision Engine                            │
│  • Combine fraud_score + account_risk + merchant_risk   │
│  • Determine APPROVE/REVIEW/DECLINE                     │
│  • Generate decision metadata                           │
│  [Latency: 8ms]                                         │
└────────────────┬────────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────────┐
│              Response to Client                         │
│  • Transaction ID                                       │
│  • Decision (APPROVE/REVIEW/DECLINE)                    │
│  • Fraud score & confidence                             │
│  • Total latency: <100ms P99                            │
└────────────────────────────────────────────────────────┘
```

---

## Daily Batch Processing

### Timeline: 2:00 AM - 4:00 AM (Scheduled Daily)

**Purpose**: Aggregate transaction data for model retraining, generate reports

```
2:00 AM  ─────────────────────────────────────────────
         │ Batch job triggered by scheduler (Airflow)
         │
2:00-2:15 AM
         │ Extract Phase
         │ ├─ PostgreSQL → read all transactions (24 hours)
         │ ├─ Extract: 100,000-200,000 transactions
         │ ├─ Include decision outcomes
         │ ├─ Include chargeback data (delayed labels)
         │ └─ Output: Parquet files in staging directory
         │
2:15-2:45 AM
         │ Transform Phase
         │ ├─ Data validation & cleaning
         │ ├─ Feature engineering for batch
         │ ├─ Merge chargeback labels with predictions
         │ ├─ Calculate aggregate statistics
         │ │  ├─ Approval rate: 92.3%
         │ │  ├─ Fraud rate: 0.087%
         │ │  ├─ False positive rate: 0.42%
         │ │  ├─ Model accuracy: 98.2%
         │ │  └─ Latency (P95): 84ms
         │ └─ Output: Clean dataset → BigQuery
         │
2:45-3:00 AM
         │ Load Phase
         │ ├─ Write to BigQuery daily partition
         │ ├─ Update aggregate tables
         │ └─ Generate reports (fraud trends, patterns)
         │
3:00-3:15 AM
         │ Analysis Phase
         │ ├─ Detect anomalies in fraud patterns
         │ ├─ Alert if fraud spike detected
         │ ├─ Check data quality metrics
         │ └─ Validate thresholds still appropriate
         │
3:15-3:30 AM
         │ Cleanup Phase
         │ ├─ Archive completed transactions
         │ ├─ Compress old logs
         │ ├─ Purge temporary files
         │ └─ Job complete
         │
4:00 AM  Batch processing complete
         └─ Data ready for weekly model retraining
```

### Batch Data Pipeline

```
PostgreSQL
(Transaction DB)
    ↓
┌─────────────────────┐
│ Extract (2:00-2:15) │
│ 100K-200K txns/day  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Transform (2:15)    │
│ • Clean             │
│ • Validate          │
│ • Feature engineer  │
│ • Label merge       │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Load (2:45-3:00)    │
│ → BigQuery          │
│ • Daily partition   │
│ • Aggregate tables  │
└──────────┬──────────┘
           ↓
┌─────────────────────┐
│ Analysis (3:00)     │
│ • Fraud trends      │
│ • Pattern detection │
│ • Quality metrics   │
└──────────┬──────────┘
           ↓
Reporting Dashboard
Performance Metrics
```

---

## Weekly Model Retraining

### Timeline: Monday 3:00 AM - 1:00 PM (Weekly)

**Purpose**: Retrain ML models with latest labeled data, improve fraud detection

```
Monday 3:00 AM ──────────────────────────────────────
         │ Retraining pipeline starts
         │
3:00-3:30 AM
         │ Data Preparation Phase
         │ ├─ Load last 7 days transactions from BigQuery
         │ ├─ Load chargeback labels (delayed 30-180 days)
         │ ├─ Merge labels with predictions
         │ ├─ Handle class imbalance:
         │ │  ├─ Oversample fraud (minority) class
         │ │  ├─ Stratified split
         │ │  └─ Training data: 70%, Validation: 15%, Test: 15%
         │ └─ Output: Training dataset ready
         │
3:30-5:00 AM
         │ LSTM Model Retraining (90 minutes)
         │ ├─ Load previous LSTM checkpoint
         │ ├─ Prepare sequential transaction sequences
         │ ├─ Train on 7 days of new data
         │ ├─ Evaluate on validation set
         │ │  ├─ Precision: 94%
         │ │  ├─ Recall: 96%
         │ │  └─ F1 Score: 0.95
         │ ├─ Compare vs production model
         │ │  └─ If better: stage for deployment
         │ │  └─ If worse: keep current model
         │ └─ Save new model → Model Registry
         │
5:00-7:00 AM (Parallel with LSTM)
         │ XGBoost Model Retraining (120 minutes)
         │ ├─ Load feature matrix (7 days)
         │ ├─ Hyperparameter tuning
         │ ├─ Train new model
         │ ├─ Evaluate on validation set
         │ │  ├─ Precision: 96%
         │ │  ├─ Recall: 90%
         │ │  └─ AUC-ROC: 0.97
         │ ├─ Perform cross-validation
         │ └─ Save new model → Model Registry
         │
7:00-9:00 AM
         │ Ensemble & A/B Testing Setup
         │ ├─ Combine new models with latest ensemble weights
         │ ├─ Evaluate ensemble on test set
         │ ├─ Compare accuracy vs current production
         │ ├─ If improvement > 1%:
         │ │  ├─ Set up A/B test (10% traffic)
         │ │  ├─ Run for 7 days
         │ │  └─ If metrics good: deploy
         │ │  └─ If metrics bad: keep current
         │ └─ Document results
         │
9:00-10:00 AM
         │ Monitoring & Alerts
         │ ├─ Update monitoring thresholds
         │ ├─ Reset drift detection baseline
         │ ├─ Clear old model artifacts
         │ └─ Send report to ML team
         │
1:00 PM  Retraining complete
         └─ New model ready for production
```

### Model Retraining Pipeline

```
BigQuery (Historical Data)
├─ Transaction events (7 days)
├─ Chargeback labels (delayed)
└─ Merchant/Account profiles
    ↓
┌──────────────────────────┐
│ Data Preparation         │
│ • Merge labels           │
│ • Handle imbalance       │
│ • Split train/val/test   │
└─────────┬────────────────┘
          │
    ┌─────┴─────┐
    ↓           ↓
┌─────────┐  ┌─────────┐
│ LSTM    │  │ XGBoost │
│ 90 min  │  │ 120 min │
│         │  │         │
│ Model 1 │  │ Model 2 │
└────┬────┘  └────┬────┘
     │           │
     └─────┬─────┘
           ↓
    ┌─────────────┐
    │ Ensemble    │
    │ • Combine   │
    │ • Weight    │
    │ • A/B Test  │
    │ (10% traffic)
    └────┬────────┘
         ↓
    Model Registry (Versioned)
    ├─ LSTM v27
    ├─ XGBoost v45
    ├─ Isolation Forest v12
    └─ Rules v3
```

---

## Storage Lifecycle

### Hot Data (Real-Time, Last 7 Days)

**Storage**: Redis (in-memory cache)

**Data**:
- Recent transaction history
- Feature cache
- Account/merchant profiles
- Velocity counters

**Retention**: 7 days
**Purpose**: Fast feature computation
**Access Pattern**: 1000s/second reads

### Warm Data (Last 90 Days)

**Storage**: PostgreSQL (OLTP database)

**Data**:
- All transactions from last 90 days
- Account history
- Merchant history
- Rules engine lookups

**Retention**: 90 days
**Purpose**: Feature engineering, historical lookups
**Access Pattern**: 100s/second reads, continuous writes

### Cold Data (Historical, 90+ Days)

**Storage**: BigQuery (Data Warehouse)

**Data**:
- All transactions (archive)
- Chargeback labels
- Fraud patterns
- Model training data

**Retention**: 2+ years
**Purpose**: Model training, analysis, compliance
**Access Pattern**: Batch queries, analytical

### Archival Data (Compliance, 2+ Years)

**Storage**: Cloud Storage (S3, GCS, Azure Blob)

**Data**:
- Compressed transaction archives
- Audit logs
- Compliance records

**Retention**: 7 years (regulatory requirement)
**Purpose**: Legal/compliance
**Access Pattern**: Rarely accessed, if ever

### Data Movement Flow

```
Real-Time Transaction
    ↓ (1 second)
┌──────────┐
│  Redis   │ Hot data (7 days)
│ (1-10ms) │ • Velocity counts
└────┬─────┘ • Feature cache
     │
     ↓ (Continuous, async)
┌──────────┐
│PostgreSQL│ Warm data (90 days)
│  (10ms)  │ • Full history
└────┬─────┘ • Searchable
     │
     ↓ (Daily batch)
┌──────────┐
│ BigQuery │ Cold data (2+ years)
│ (1-10s)  │ • Training data
└────┬─────┘ • Analysis
     │
     ↓ (Monthly archival)
┌──────────┐
│ S3/GCS   │ Archive (7 years)
│(days)    │ • Legal hold
└──────────┘
```

---

## Event Streaming Architecture

### Kafka Topics

**Topic 1: transactions.evaluate**
- **Message**: Transaction event for fraud evaluation
- **Producers**: API Gateway
- **Consumers**: Feature Engineering, Fraud Detection, Decision Engine
- **Retention**: 7 days
- **Partitions**: 100+ (for parallelism)

**Topic 2: decisions.made**
- **Message**: Decision result (APPROVE/REVIEW/DECLINE)
- **Producers**: Decision Engine
- **Consumers**: Notification Service, Monitoring, Analytics
- **Retention**: 30 days

**Topic 3: chargebacks.received**
- **Message**: Chargeback event with fraud label
- **Producers**: External fraud/payment team
- **Consumers**: Data warehouse, model retraining
- **Retention**: 2 years

**Topic 4: reviews.submitted**
- **Message**: Manual review result from operator
- **Producers**: Manual review team
- **Consumers**: Feedback loop, model improvement
- **Retention**: 2 years

### Event Flow

```
Payment Network
    ↓
Transaction Event
    ├→ Kafka: transactions.evaluate
    │  ├→ Feature Engineering
    │  ├→ Fraud Detection
    │  └→ Decision Engine
    │
    └→ Decision Made
       ├→ Kafka: decisions.made
       │  ├→ Notification Service
       │  ├→ Monitoring (P99 latency, accuracy)
       │  └→ Analytics (approval rate, fraud rate)
       │
       └→ Transaction Logged
          ├→ PostgreSQL (OLTP)
          ├→ BigQuery (warehouse)
          └→ Kafka (audit trail)

(30-180 days later)
    ↓
Chargeback Received
    ├→ Kafka: chargebacks.received
    │  ├→ BigQuery (label update)
    │  └→ Weekly Retraining Pipeline
    │
    └→ Model Improved
       ├→ Production Deployment
       └→ Accuracy improves
```

---

## Data Consistency & Guarantees

### Transactional Consistency

**Level**: Eventually consistent (within 100ms for reads)

**Guarantees**:
- Fraud score for transaction never changes (immutable)
- Decision captured at decision_timestamp
- Chargeback labels added asynchronously (30-180 day delay)

### Exactly-Once Processing

**Achieved via**:
- Idempotent API (transaction_id deduplication)
- Kafka offsets (commit only after processing)
- Database transactions (ACID properties)

### Ordering Guarantees

**Within Account**: Maintained
- Transactions from same account processed in order
- Velocity checks based on chronological order

**Across Accounts**: Not guaranteed
- Transactions from different accounts can interleave
- Not a concern (independent decisions)

---

**Data Flow Version**: 1.0
**Last Updated**: February 2026
**Status**: Production Ready