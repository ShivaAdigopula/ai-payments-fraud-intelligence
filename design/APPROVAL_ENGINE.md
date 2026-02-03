# Payment Approval Engine

## Decision Framework

### Three-Tier Decision Model

The core of instant payment decisions: score-based routing to APPROVE, REVIEW, or DECLINE

```
┌────────────────────────────────────────────┐
│                                            │
│  HIGH CONFIDENCE APPROVE                   │
│  (Low fraud score, trusted patterns)       │
│  → Instant approval, <20ms                 │
│  → 92% of transactions                     │
│                                            │
├────────────────────────────────────────────┤
│                                            │
│  UNCERTAIN ZONE (REVIEW)                   │
│  (Borderline fraud score)                  │
│  → Queue for human review                  │
│  → 5-15 minute SLA                         │
│  → 6% of transactions                      │
│                                            │
├────────────────────────────────────────────┤
│                                            │
│  HIGH CONFIDENCE DECLINE                   │
│  (High fraud score, fraud patterns)        │
│  → Auto-blocked, instant notification      │
│  → 2% of transactions                      │
│                                            │
└────────────────────────────────────────────┘
```

---

## Decision Categories

### APPROVE (Auto-Approve, 92%)

**Criteria**:
- Fraud score < 0.30 (low fraud probability)
- No hard rules triggered
- Account status healthy

**Action**:
- Transaction automatically approved
- No manual review needed
- Instant decision (<20ms)

**Notification**:
- Optional: Email/SMS confirmation for high-value
- Merchant: Order confirmation

**Threshold Modifiers**:
- VIP account: -5% (more lenient, easier to approve)
- High velocity: +2% (stricter)
- New merchant: +3% (stricter)

**Example Decision**:
```
Fraud Score: 0.28 (from ensemble)
Base Threshold: 0.30
Account Modifier: -0.05 (VIP)
Final Threshold: 0.35
Decision: 0.28 < 0.35 → APPROVE ✓
```

### REVIEW (Manual Review, 6%)

**Criteria**:
- Fraud score 0.30-0.70 (uncertain)
- Hard rules triggered but not critical
- Requires human judgment

**Action**:
- Transaction queued for manual review
- Fraud analyst evaluates within 5-15 minutes
- Decision context provided (50+ signals)

**Notification**:
- Customer: "Your transaction is being reviewed. Check back in 5-15 minutes."
- Merchant: "Transaction pending manual review"

**Review Dashboard Features**:
- Transaction details
- Customer history
- Merchant information
- Fraud score breakdown (by model)
- Similar transactions from same account
- Device & location information
- Analyst decision options: APPROVE / DECLINE / ESCALATE

**SLA**:
- 50% reviewed within 5 minutes
- 95% reviewed within 15 minutes
- Escalation alert if queue > 10,000

**Example Decision**:
```
Fraud Score: 0.55 (uncertain)
Base Threshold: 0.30 (approve) - 0.70 (decline)
Fraud Score 0.55 falls in REVIEW zone
→ Queue for manual analyst
→ Decision within 15 minutes
```

### DECLINE (Auto-Decline, 2%)

**Criteria**:
- Fraud score > 0.70 (high fraud confidence)
- Critical hard rules triggered
- Immediate threat to account

**Action**:
- Transaction automatically declined
- Payment blocked instantly
- Fraud team alerted

**Notification**:
- SMS: "Your transaction was declined. Contact us if this was you."
- Email: Detailed explanation, why declined
- Merchant: "Transaction declined"

**Customer Appeal**:
- Available: Within 7 days
- Process: Contact support, provide proof
- Resolution: 24-48 hours

**Example Decision**:
```
Fraud Score: 0.82 (high fraud confidence)
Base Threshold: 0.70
Decline Threshold: 0.75 (after modifiers)
Decision: 0.82 > 0.75 → DECLINE ✓
Alert: Fraud team notified
```

---

## Business Rules Framework

### Rule 1: Velocity Rules

**Purpose**: Prevent card testing and rapid-fire fraud

```
Card velocity rules:
├─ Same card: Max 5 transactions in 5 minutes
├─ Same card: Max $5,000 daily ($0 for new cards)
├─ Same merchant: Max 3 attempts per hour
├─ New card: Apply -20% threshold for first 24 hours
└─ New card: Apply -$1,000 daily limit for first 7 days
```

**Example**:
- Card used 6 times in 5 minutes → Rule triggered → DECLINE
- Card has 4 transactions totaling $4,800 today, 6th transaction $500 → Rule triggered → DECLINE

### Rule 2: Impossible Travel

**Purpose**: Detect account takeover via geographic impossibility

```
Detect impossible travel:
├─ If last transaction in different country
├─ Calculate required travel speed
├─ If speed > 900 km/hour (plane max speed):
│  ├─ Flag as REVIEW (human judgment)
│  └─ Or DECLINE if amount is high
└─ Allow 6 hours buffer for flight delays
```

**Example**:
- Last transaction in Tokyo at 10:00 AM
- Current transaction attempt in New York at 11:00 AM (1 hour later)
- Distance: 10,800 km, Required speed: 10,800 km/hr (impossible)
- → Rule triggered → REVIEW

### Rule 3: Card Testing Pattern

**Purpose**: Stop attackers testing stolen cards

```
Card testing detection:
├─ Amount: $1-10 (test amounts)
├─ Multiple merchants: 3+ different merchants
├─ Time window: All within 30 minutes
└─ If all conditions true:
   └─ DECLINE (high confidence fraud)
```

**Example**:
- Transaction 1: $2.50 to Merchant A at 10:00
- Transaction 2: $5.00 to Merchant B at 10:15
- Transaction 3: $3.75 to Merchant C at 10:25
- → Pattern matches → DECLINE

### Rule 4: Whitelist/Blacklist

**Purpose**: Override model decisions for known good/bad entities

**Whitelist** (Auto-approve regardless of score):
- VIP merchant (e.g., Amazon, Uber)
- Known good account with 5+ years history
- Corporate card with verified employer
- Action: Skip fraud checks, instant APPROVE

**Blacklist** (Auto-decline regardless of score):
- Compromised merchant (ongoing fraud investigation)
- Known fraud ring
- Suspended account
- Action: Immediate DECLINE

**Example**:
```
Account on whitelist (VIP, 10 years history)
Fraud Score: 0.80 (high fraud score)
Decision: APPROVE (whitelist override)
Reason: Known good account overrides model
```

### Rule 5: Account Status

**Purpose**: Adjust strictness based on account maturity and status

**New Account** (<7 days):
- Apply -10% threshold adjustment (stricter)
- Lower daily limits
- Higher review rate

**Emerging Account** (7-30 days):
- Apply -5% threshold adjustment
- Gradual limit increases
- Standard review rate

**Regular Account** (30+ days):
- Standard thresholds
- Standard daily limits
- Standard review rate

**VIP Account** (>1 year, low fraud history):
- Apply +5% threshold adjustment (more lenient)
- Higher daily limits
- Lower review rate

**Suspended Account** (fraud investigation):
- All transactions: DECLINE
- No exceptions
- Investigation team review

---

## Decision Logic

### Algorithm

```python
def make_decision(transaction, fraud_score, account, merchant):
    """
    Combine fraud score and business context for decision
    """
    
    # Base thresholds
    approve_threshold = 0.30
    decline_threshold = 0.70
    
    # 1. Get account risk level
    account_age = calculate_account_age(account.created_date)
    account_fraud_rate = account.lifetime_fraud_count / account.lifetime_txn_count
    account_risk = 0.3 * (min(account_age, 365) / 365) + 0.7 * account_fraud_rate
    
    # 2. Get merchant risk level
    merchant_chargeback_rate = merchant.chargebacks / merchant.total_transactions
    merchant_fraud_rate = merchant.fraud_txns / merchant.total_transactions
    merchant_risk = 0.6 * merchant_chargeback_rate + 0.4 * merchant_fraud_rate
    
    # 3. Apply modifiers based on context
    threshold_adjust = 0.0
    
    # Account age modifier
    if account_age < 7:
        threshold_adjust -= 0.10  # Stricter (lower threshold)
    elif account_age > 365:
        threshold_adjust += 0.05  # More lenient (higher threshold)
    
    # Transaction amount modifier
    if transaction.amount > 1000:
        threshold_adjust -= 0.05  # Stricter for high value
    elif transaction.amount > 5000:
        threshold_adjust -= 0.10  # Much stricter for very high value
    
    # Merchant risk modifier
    if merchant_risk > 0.05:
        threshold_adjust -= (merchant_risk * 0.10)
    
    # Apply account type adjustments
    if account.is_vip:
        threshold_adjust += 0.05
    if account.is_new_device:
        threshold_adjust -= 0.03
    
    # Final thresholds
    adjusted_approve_threshold = approve_threshold + threshold_adjust
    adjusted_decline_threshold = decline_threshold + threshold_adjust
    
    # 4. Check business rules first (hard constraints)
    if is_on_blacklist(account, merchant):
        return DECLINE
    
    if is_on_whitelist(account, merchant) and fraud_score < 0.60:
        return APPROVE
    
    if check_velocity_rules(account, transaction):
        return DECLINE
    
    if check_card_testing_pattern(account):
        return DECLINE
    
    if check_impossible_travel(account, transaction):
        if transaction.amount > 500:
            return DECLINE
        else:
            return REVIEW
    
    # 5. Make decision based on fraud score and thresholds
    if fraud_score < adjusted_approve_threshold:
        return APPROVE
    elif fraud_score > adjusted_decline_threshold:
        return DECLINE
    else:
        return REVIEW
```

### Decision Matrix Examples

| Fraud Score | Account Age | Amount | Merchant Risk | Decision |
|-------------|------------|--------|----------------|----------|
| 0.15 | 5 years | $50 | Low (0.01) | APPROVE |
| 0.35 | 5 years | $50 | Low (0.01) | REVIEW |
| 0.75 | 5 years | $50 | Low (0.01) | DECLINE |
| 0.35 | 2 days | $50 | Low (0.01) | DECLINE (new acct) |
| 0.25 | 2 days | $50 | Low (0.01) | REVIEW (new acct) |
| 0.35 | 5 years | $5000 | Low (0.01) | DECLINE (high amt) |
| 0.15 | VIP | $5000 | Low (0.01) | APPROVE (VIP) |
| 0.50 | 5 years | $50 | High (0.08) | DECLINE (merchant) |

---

## Dynamic Thresholds

### Threshold Configuration Table

| Parameter | Approve Threshold | Decline Threshold | Notes |
|-----------|------------------|------------------|-------|
| New Account (<7d) | 0.20 | 0.60 | Stricter (lower=more fraud) |
| Emerging (7-30d) | 0.25 | 0.65 | Still strict |
| Regular (30d-1yr) | 0.30 | 0.70 | Standard |
| Established (>1yr) | 0.35 | 0.75 | More lenient |
| VIP Account | 0.40 | 0.80 | Very lenient |
| High Amount (>$5k) | 0.20 | 0.60 | Stricter |
| New Device | 0.25 | 0.65 | Stricter |
| High Merchant Risk | 0.25 | 0.65 | Stricter |
| Trusted Merchant | 0.35 | 0.75 | More lenient |

---

## Approval Rate Optimization

### Target Metrics

```
Auto-Approval Rate: 92% (target)
├─ Business goal: Maximize frictionless transactions
├─ Current baseline: 70% (manual system)
├─ Improvement: +22 percentage points
└─ Expected additional revenue: $8.5M/year at $1B volume

Manual Review Rate: 6% (target)
├─ Business goal: Manual team capacity
├─ SLA: 5-15 minutes per review
├─ Estimated manual reviewers: 50-60 people
└─ Reduction from 200+ people in manual system

Auto-Decline Rate: 2% (target)
├─ Business goal: Block clear fraud
├─ Trade-off: Few false declines (<0.5%)
└─ Prevents chargebacks and fraud losses
```

### Optimization Approach

**1. Baseline Phase** (Week 1):
- Start: 80% approval, 15% review, 5% decline
- Thresholds: Conservative
- Monitor fraud loss rate

**2. Iterative Tuning** (Weeks 2-4):
- Monthly threshold adjustments
- Track approval rate vs target (92%)
- Monitor fraud leakage (fraud not caught)
- Decision: Adjust if rates drift

**3. Validation** (Weeks 5-8):
- Run A/B test with new thresholds
- 10% traffic on new thresholds, 90% on baseline
- Measure: Fraud rate, approval rate, customer satisfaction
- Decision: Keep new or revert

**4. Stabilization** (Ongoing):
- Once optimal thresholds found
- Lock in configuration
- Monthly monitoring for drift
- Quarterly reoptimization

---

## Decision Metadata

### Complete Output Structure

```json
{
  "transaction_id": "txn_12345",
  "decision": "APPROVE",
  "decision_timestamp": "2024-01-15T10:23:45.123Z",
  
  "fraud_score": 0.28,
  "confidence": 0.92,
  
  "fraud_components": {
    "xgboost_score": 0.35,
    "lstm_score": 0.22,
    "anomaly_score": 0.15,
    "rules_flag": false
  },
  
  "decision_factors": [
    "low_fraud_score",
    "trusted_account",
    "regular_merchant",
    "normal_amount",
    "usual_device"
  ],
  
  "risk_signals": [
    "amount_slightly_higher_than_usual"
  ],
  
  "thresholds_applied": {
    "base_approve": 0.30,
    "base_decline": 0.70,
    "account_age_modifier": 0.05,
    "amount_modifier": 0.0,
    "merchant_modifier": -0.02,
    "device_modifier": 0.0,
    "final_approve_threshold": 0.33,
    "final_decline_threshold": 0.68
  },
  
  "context": {
    "account_age_days": 387,
    "account_verification": "verified",
    "is_vip": false,
    "account_fraud_rate": 0.001,
    "merchant_fraud_rate": 0.0008,
    "transaction_amount": 99.99,
    "device_is_new": false,
    "impossible_travel_score": 0.0,
    "velocity_flag": false
  },
  
  "latency_ms": 87,
  "latency_breakdown": {
    "api_gateway": 5,
    "feature_engineering": 42,
    "fraud_detection": 25,
    "risk_scoring": 8,
    "decision_engine": 7
  }
}
```

---

## Appeal Process

### Customer Appeal

**Eligibility**:
- Timeline: Within 7 days of transaction
- Reason codes: "Transaction blocked", "Transaction declined"
- Requirement: Proof customer initiated transaction

**Appeal Process**:

1. **Submission**:
   - Customer contacts support
   - Provides transaction ID and explanation
   - Support creates appeal ticket

2. **Initial Review** (24 hours):
   - Fraud team reviews transaction
   - Checks account history
   - Verifies merchant legitimacy

3. **Decision** (24-48 hours):
   - Approve appeal: Reverse decline, reattempt transaction
   - Deny appeal: Confirm fraud
   - Escalate: Further investigation needed

4. **Communication**:
   - Email: Appeal decision + explanation
   - Support: Direct follow-up if needed

### Override Capabilities

**Merchant Support Override**:
- Authority: Merchant ops team
- Condition: With justification (CSAT issue, VIP customer, etc.)
- Audit: Logged for review

**Customer Service Override**:
- Authority: Support manager
- Condition: Customer escalation (retention risk)
- Audit: Escalation ticket required

**Fraud Team Override**:
- Authority: Senior fraud analyst
- Condition: Investigation findings
- Audit: Case notes mandatory

---

## Monitoring & Alerts

### Key Metrics to Monitor

**Operational Metrics**:
```
Approval Rate (Target: 92%)
├─ If < 88%: Warning alert
├─ If < 85%: Critical alert
└─ Action: Review thresholds

Decline Rate (Target: 2%)
├─ If > 5%: Warning alert
└─ Action: Review thresholds

Review Queue Size (SLA: <5 min wait)
├─ If > 5,000: Warning alert
├─ If > 10,000: Critical alert
└─ Action: Auto-callback, hire more reviewers
```

**Fraud Metrics**:
```
Fraud Loss Rate (Target: <0.01% of volume)
├─ If > 0.015%: Warning alert
├─ If > 0.02%: Critical alert
└─ Action: Lower decline threshold

False Decline Rate (Target: <0.5%)
├─ If > 0.7%: Warning alert
└─ Action: Raise approve threshold
```

**Customer Metrics**:
```
Customer Satisfaction (Target: 4.7+/5)
├─ If < 4.5: Warning alert
├─ If < 4.2: Critical alert
└─ Action: Review false declines

Appeal Rate (Target: <1%)
├─ If > 1.5%: Warning alert
└─ Action: Check thresholds
```

### Alert Conditions

| Condition | Severity | Action |
|-----------|----------|--------|
| Approval rate drops below 88% | Warning | Check thresholds |
| Fraud loss exceeds $50k/day | Critical | Reduce decline threshold |
| Review queue backlog > 5,000 | Warning | Add reviewers |
| False decline rate > 0.7% | Warning | Raise approve threshold |
| System latency P99 > 150ms | Warning | Investigate bottleneck |
| Decision latency P95 > 100ms | Critical | Page on-call |

---

## Performance Targets

### Latency SLOs

| Metric | P50 | P95 | P99 |
|--------|-----|-----|-----|
| Decision Latency | 45ms | 85ms | 120ms |
| Queue Wait Time (REVIEW) | 2 min | 8 min | 15 min |
| Appeal Resolution | 12 hrs | 24 hrs | 48 hrs |

### Accuracy SLOs

| Metric | Target |
|--------|--------|
| Decision Accuracy | 98.2% |
| Approval Rate | 92% |
| Fraud Catch Rate | 98%+ |
| False Decline Rate | <0.5% |

### Availability SLOs

| Metric | Target |
|--------|--------|
| System Uptime | 99.99% |
| API Availability | 99.99% |
| Manual Review Queue | 100% |

---

**Engine Version**: 1.0
**Last Updated**: February 2026
**Status**: Production Ready