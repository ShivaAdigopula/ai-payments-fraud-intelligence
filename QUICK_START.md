# Quick Start

Choose your reading path based on your role:

## 👔 Executive / Product Manager (30 minutes)

**Goal**: Understand business value and ROI

1. **Start**: [AT_A_GLANCE.md](AT_A_GLANCE.md) (5 min)
   - See 28x ROI, $26.7M value, 10,000x speed improvement

2. **Then**: [docs/OVERVIEW.md](docs/OVERVIEW.md) - "Problem Statement" & "Financial Projections" sections (10 min)
   - $6.2M fraud losses, competitive advantage, revenue model

3. **Finally**: [ROADMAP.md](ROADMAP.md) - Phase overview only (5 min)
   - 16-week timeline, deliverables, team requirements

**Key Takeaways**:
- 92% auto-approval vs 70% baseline = more transactions
- <0.5% false declines vs 5-8% = happier customers
- 75% cost reduction = $12M savings annually

---

## 🏗️ Tech Lead / Architect (2-3 hours)

**Goal**: Understand system design, components, integration points

1. **Start**: [README.md](README.md) (10 min)
   - Project vision, problem, metrics overview

2. **Then**: [architecture/HIGH_LEVEL_DESIGN.md](architecture/HIGH_LEVEL_DESIGN.md) (30 min)
   - 4-tier architecture, event-driven pattern, technology choices
   - Latency budget breakdown: 50ms API Gateway, 80ms features, 50ms fraud, 30ms risk, 20ms decision

3. **Next**: [architecture/COMPONENTS.md](architecture/COMPONENTS.md) (30 min)
   - 8 components with SLA targets, interfaces, dependencies
   - API Gateway, Feature Engineering, Fraud Detection, Risk Scoring, Decision Engine, Notification, Rules, Monitoring

4. **Then**: [architecture/DATA_FLOW.md](architecture/DATA_FLOW.md) (20 min)
   - Real-time transaction flow (T+0 to T+105ms)
   - Daily batch processing, weekly model retraining
   - Storage lifecycle (hot → warm → cold)

5. **Next**: [design/APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md) (20 min)
   - Decision framework: APPROVE/REVIEW/DECLINE logic
   - Business rules, thresholds, metadata output

6. **Finally**: [specifications/API_SPEC.md](specifications/API_SPEC.md) (10 min)
   - 8 main endpoints, authentication, rate limiting
   - Request/response examples, SLA targets

**Key Decisions**:
- Event-driven microservices (vs monolith)
- Real-time stream + batch pipelines
- Distributed ML inference for latency
- Multiple databases by access pattern

---

## 🤖 ML Engineer (3+ hours)

**Goal**: Understand fraud detection models, training, optimization

1. **Start**: [AT_A_GLANCE.md](AT_A_GLANCE.md) (5 min)
   - See the ensemble approach at high level

2. **Then**: [design/FRAUD_DETECTION_STRATEGY.md](design/FRAUD_DETECTION_STRATEGY.md) (80 min)
   - 4-model ensemble: XGBoost (96% precision), LSTM (96% recall), Isolation Forest (anomalies), Rules (hard constraints)
   - 8 fraud types detected with methods
   - 50+ feature engineering strategy
   - Training data requirements (12-24 months, 50M+ transactions)
   - Model monitoring, drift detection, retraining triggers
   - Feedback loops from chargebacks/reviews
   - A/B testing framework for deployment

3. **Next**: [design/APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md) - "Ensemble Strategy" section (10 min)
   - How individual model scores combine: weighted average
   - Threshold calibration for precision/recall trade-off

4. **Then**: [architecture/COMPONENTS.md](architecture/COMPONENTS.md) - "Fraud Detection Component" (10 min)
   - Inference latency target: <50ms
   - Expected throughput, SLA, dependencies

5. **Finally**: [docs/GLOSSARY.md](docs/GLOSSARY.md) (10 min)
   - ML-specific terms: AUC-ROC, precision, recall, F1, drift detection

**Key Modeling Decisions**:
- Supervised (XGBoost, LSTM) + Unsupervised (Isolation Forest) hybrid
- Weekly retraining for LSTM, monthly for XGBoost/Anomaly
- Weighted ensemble with production feedback
- Handle class imbalance via oversampling + weighted loss

---

## 🔧 Backend / API Engineer (3 hours)

**Goal**: Understand API design, system integration, deployment

1. **Start**: [README.md](README.md) (10 min)
   - Project overview, success metrics

2. **Then**: [specifications/API_SPEC.md](specifications/API_SPEC.md) (40 min)
   - 8 endpoints: evaluate, get status, batch, risk profiles, feedback, list, config
   - Authentication (OAuth 2.0), rate limiting (10k/min default)
   - Request/response contracts with JSON examples
   - 9 error codes, SLA targets (P50/P95/P99)
   - Webhooks, retry strategy, backward compatibility

3. **Next**: [architecture/HIGH_LEVEL_DESIGN.md](architecture/HIGH_LEVEL_DESIGN.md) - "Component Tiers" (20 min)
   - API Gateway tier: validation, auth, rate limiting

4. **Then**: [architecture/COMPONENTS.md](architecture/COMPONENTS.md) (30 min)
   - Integration points with other components
   - Latency budgets, throughput requirements

5. **Next**: [architecture/DATA_FLOW.md](architecture/DATA_FLOW.md) - "Real-Time Flow" (20 min)
   - How transactions flow through system
   - Dependencies and data passing

6. **Finally**: [design/APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md) (15 min)
   - Decision logic your API must enforce
   - Metadata output structure

**Key Implementation Details**:
- Evaluate endpoint: <120ms P99 latency
- Batch endpoint: <800ms P99 for 100 transactions
- Cache frequent lookups in Redis
- Circuit breakers for model service failures

---

## 🚀 DevOps / Infrastructure (2-3 hours)

**Goal**: Understand deployment, scaling, monitoring requirements

1. **Start**: [README.md](README.md) (5 min)
   - Overall metrics, team size

2. **Then**: [architecture/HIGH_LEVEL_DESIGN.md](architecture/HIGH_LEVEL_DESIGN.md) (30 min)
   - Technology stack: Kubernetes, Docker, AWS/GCP
   - HA/DR strategy, scalability approach
   - Latency budget implications for deployment

3. **Next**: [architecture/COMPONENTS.md](architecture/COMPONENTS.md) (20 min)
   - 8 components to deploy and scale independently
   - SLA targets per component
   - Resource requirements estimate

4. **Then**: [architecture/DATA_FLOW.md](architecture/DATA_FLOW.md) (20 min)
   - Daily batch jobs (2-4am), weekly retraining (Monday 3am, 10 hours)
   - Storage lifecycle management
   - Data pipeline orchestration

5. **Next**: [design/APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md) - "Monitoring & Alerts" (15 min)
   - Key metrics to monitor
   - Alert conditions for degradation

6. **Finally**: [ROADMAP.md](ROADMAP.md) - Phase 3 "Production Readiness" (10 min)
   - Load testing plan, DR setup, monitoring infrastructure

**Key Infrastructure Decisions**:
- Kubernetes for container orchestration
- Auto-scaling: 100k tps peak = need 50-100 pods per component
- Multi-region for HA (active-active)
- Canary deployment (1% → 10% → 50% → 100%)

---

## 📊 Data Analyst / BI (1-2 hours)

**Goal**: Understand data sources, analytics, monitoring

1. **Start**: [AT_A_GLANCE.md](AT_A_GLANCE.md) (5 min)
   - Impact metrics at a glance

2. **Then**: [docs/OVERVIEW.md](docs/OVERVIEW.md) - "Data Requirements" section (10 min)
   - Input data needed, historical requirements

3. **Next**: [architecture/DATA_FLOW.md](architecture/DATA_FLOW.md) (20 min)
   - Data sources, transformation pipeline
   - Real-time vs batch processing
   - Storage and warehousing

4. **Then**: [design/FRAUD_DETECTION_STRATEGY.md](design/FRAUD_DETECTION_STRATEGY.md) - "Training & Monitoring" (20 min)
   - Performance metrics to track
   - Retraining triggers and feedback loops

5. **Finally**: [design/APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md) - "Monitoring & Alerts" (10 min)
   - Business metrics to monitor

**Key Analytics Focus**:
- Fraud detection accuracy (precision, recall, F1)
- Approval rates and false positive rates
- Model drift detection
- Customer impact metrics

---

## 🔍 Security / Compliance (1 hour)

**Goal**: Understand security, privacy, regulatory requirements

1. **Start**: [specifications/API_SPEC.md](specifications/API_SPEC.md) - "Authentication & Security" (10 min)
   - OAuth 2.0, token validation, rate limiting

2. **Then**: [docs/GLOSSARY.md](docs/GLOSSARY.md) - Search for compliance terms (10 min)
   - Understand industry terminology

3. **Next**: [docs/OVERVIEW.md](docs/OVERVIEW.md) - "Risk Mitigation" section (10 min)
   - Security threats and mitigations

4. **Finally**: [architecture/HIGH_LEVEL_DESIGN.md](architecture/HIGH_LEVEL_DESIGN.md) - "HA/DR Strategy" (10 min)
   - Data protection, backup strategy

---

## 📚 Complete Documentation Path (6+ hours)

Read in this order for full context:

1. README.md (overview)
2. AT_A_GLANCE.md (summary)
3. docs/OVERVIEW.md (problem & solution)
4. docs/GLOSSARY.md (terminology)
5. architecture/HIGH_LEVEL_DESIGN.md (architecture)
6. architecture/COMPONENTS.md (8 components)
7. architecture/DATA_FLOW.md (data flows)
8. design/FRAUD_DETECTION_STRATEGY.md (ML approach)
9. design/APPROVAL_ENGINE.md (decision logic)
10. specifications/API_SPEC.md (API contracts)
11. ROADMAP.md (implementation timeline)

---

**Need Help Finding Something?**
→ See [INDEX.md](INDEX.md) for complete navigation and cross-references