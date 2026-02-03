# Documentation Index

Complete navigation guide for the AI Payment Fraud Intelligence project.

## Quick Navigation

### By Role
- **Executives**: [QUICK_START.md#-executive--product-manager](QUICK_START.md) (30 min)
- **Tech Leads**: [QUICK_START.md#-tech-lead--architect](QUICK_START.md) (2-3 hrs)
- **ML Engineers**: [QUICK_START.md#-ml-engineer](QUICK_START.md) (3+ hrs)
- **Backend Engineers**: [QUICK_START.md#-backend--api-engineer](QUICK_START.md) (3 hrs)
- **DevOps**: [QUICK_START.md#-devops--infrastructure](QUICK_START.md) (2-3 hrs)

### By Topic
- **Problem & Solution**: [docs/OVERVIEW.md](docs/OVERVIEW.md)
- **System Architecture**: [architecture/HIGH_LEVEL_DESIGN.md](architecture/HIGH_LEVEL_DESIGN.md)
- **8 Components**: [architecture/COMPONENTS.md](architecture/COMPONENTS.md)
- **Data Flows**: [architecture/DATA_FLOW.md](architecture/DATA_FLOW.md)
- **Fraud Detection ML**: [design/FRAUD_DETECTION_STRATEGY.md](design/FRAUD_DETECTION_STRATEGY.md)
- **Approval Decisions**: [design/APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md)
- **REST API**: [specifications/API_SPEC.md](specifications/API_SPEC.md)
- **Terminology**: [docs/GLOSSARY.md](docs/GLOSSARY.md)

## Document Overview

### Root Level

| File | Purpose | Duration | Audience |
|------|---------|----------|----------|
| [README.md](README.md) | Project overview, metrics, timeline | 10 min | Everyone |
| [AT_A_GLANCE.md](AT_A_GLANCE.md) | Visual summary, key numbers | 5 min | Everyone |
| [QUICK_START.md](QUICK_START.md) | Role-based reading paths | 5 min | New readers |
| [ROADMAP.md](ROADMAP.md) | 16-week implementation plan | 20 min | Planning |
| [INDEX.md](INDEX.md) | This file | - | Navigation |

### docs/ Folder

| File | Purpose | Key Sections |
|------|---------|--------------|
| [OVERVIEW.md](docs/OVERVIEW.md) | Detailed problem statement, business case, success criteria | Problem, Solution, Target Users, Core Capabilities, Data Requirements, Financial Projections, Risk Mitigation, Competitive Analysis |
| [GLOSSARY.md](docs/GLOSSARY.md) | 50+ domain-specific terms | Business, Technical, Fraud-specific, Compliance, Architecture, ML/AI, Metrics |

### architecture/ Folder

| File | Purpose | Key Sections |
|------|---------|--------------|
| [HIGH_LEVEL_DESIGN.md](architecture/HIGH_LEVEL_DESIGN.md) | System architecture, component tiers, technology stack | Event-Driven Pattern, 4-Tier Architecture, Tech Stack, Latency Budget, Scalability, HA/DR |
| [COMPONENTS.md](architecture/COMPONENTS.md) | 8-component specifications with interfaces | API Gateway, Feature Engineering, Fraud Detection, Risk Scoring, Decision Engine, Notification, Rules Engine, Monitoring |
| [DATA_FLOW.md](architecture/DATA_FLOW.md) | Real-time and batch data processing | Real-Time Flow (T+0 to T+105ms), Batch Processing, Weekly Retraining, Storage Lifecycle |

### design/ Folder

| File | Purpose | Key Sections |
|------|---------|--------------|
| [FRAUD_DETECTION_STRATEGY.md](design/FRAUD_DETECTION_STRATEGY.md) | ML approach for fraud detection | 4-Model Ensemble, 8 Fraud Types, XGBoost/LSTM/Anomaly/Rules, Feature Engineering, Training, Monitoring, Feedback Loop, A/B Testing |
| [APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md) | Payment approval decision logic | Decision Framework, 3 Decision Categories, Business Rules, Decision Logic, Dynamic Thresholds, Appeal Process, Monitoring |

### specifications/ Folder

| File | Purpose | Key Sections |
|------|---------|--------------|
| [API_SPEC.md](specifications/API_SPEC.md) | REST API contract specifications | Auth & Security, 8 Endpoints, Error Codes, Webhooks, Rate Limiting, SLA Targets |

## Content Map

### Business & Strategy
- [README.md](README.md): Vision, problem, solution, metrics
- [AT_A_GLANCE.md](AT_A_GLANCE.md): Quick business case
- [docs/OVERVIEW.md](docs/OVERVIEW.md): Detailed problem, target market, ROI
- [ROADMAP.md](ROADMAP.md): Implementation phases and timeline

### Architecture & Design
- [architecture/HIGH_LEVEL_DESIGN.md](architecture/HIGH_LEVEL_DESIGN.md): System design, technology choices
- [architecture/COMPONENTS.md](architecture/COMPONENTS.md): Component specifications
- [architecture/DATA_FLOW.md](architecture/DATA_FLOW.md): Data processing pipelines
- [design/APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md): Decision logic

### Machine Learning
- [design/FRAUD_DETECTION_STRATEGY.md](design/FRAUD_DETECTION_STRATEGY.md): Complete ML strategy
- [docs/GLOSSARY.md](docs/GLOSSARY.md): ML terminology

### API & Integration
- [specifications/API_SPEC.md](specifications/API_SPEC.md): REST API contracts
- [architecture/COMPONENTS.md](architecture/COMPONENTS.md): Component interfaces

### Operations & Monitoring
- [design/APPROVAL_ENGINE.md](design/APPROVAL_ENGINE.md): Monitoring metrics
- [architecture/COMPONENTS.md](architecture/COMPONENTS.md): SLA targets
- [ROADMAP.md](ROADMAP.md): Deployment phases

## Key Sections by Topic

### Fraud Detection

| Question | Answer in |
|----------|-----------|
| What fraud types do we detect? | [FRAUD_DETECTION_STRATEGY.md - Fraud Types Detected](design/FRAUD_DETECTION_STRATEGY.md#fraud-types-detected) |
| How do the ML models work? | [FRAUD_DETECTION_STRATEGY.md - Detection Models](design/FRAUD_DETECTION_STRATEGY.md#detection-models) |
| How are models combined? | [FRAUD_DETECTION_STRATEGY.md - Ensemble Strategy](design/FRAUD_DETECTION_STRATEGY.md#ensemble-strategy) |
| What features do we use? | [FRAUD_DETECTION_STRATEGY.md - Feature Engineering](design/FRAUD_DETECTION_STRATEGY.md#feature-engineering) |
| How often do we retrain? | [FRAUD_DETECTION_STRATEGY.md - Training & Monitoring](design/FRAUD_DETECTION_STRATEGY.md#training--monitoring) |
| How do we monitor drift? | [FRAUD_DETECTION_STRATEGY.md - Retraining Triggers](design/FRAUD_DETECTION_STRATEGY.md#retraining-triggers) |

### Payment Approval

| Question | Answer in |
|----------|-----------|
| How do we make APPROVE/REVIEW/DECLINE decisions? | [APPROVAL_ENGINE.md - Decision Categories](design/APPROVAL_ENGINE.md#decision-categories) |
| What business rules apply? | [APPROVAL_ENGINE.md - Business Rules Framework](design/APPROVAL_ENGINE.md#business-rules-framework) |
| How do we adjust thresholds? | [APPROVAL_ENGINE.md - Dynamic Thresholds](design/APPROVAL_ENGINE.md#dynamic-thresholds) |
| What metrics do we track? | [APPROVAL_ENGINE.md - Monitoring & Alerts](design/APPROVAL_ENGINE.md#monitoring--alerts) |
| How do customer appeals work? | [APPROVAL_ENGINE.md - Appeal Process](design/APPROVAL_ENGINE.md#appeal-process) |

### System Design

| Question | Answer in |
|----------|-----------|
| How is the system architected? | [HIGH_LEVEL_DESIGN.md - 4-Tier Architecture](architecture/HIGH_LEVEL_DESIGN.md#4-tier-component-architecture) |
| What are the 8 components? | [COMPONENTS.md](architecture/COMPONENTS.md) |
| What technology stack? | [HIGH_LEVEL_DESIGN.md - Technology Stack](architecture/HIGH_LEVEL_DESIGN.md#technology-stack) |
| What's the latency budget? | [HIGH_LEVEL_DESIGN.md - Latency Budget](architecture/HIGH_LEVEL_DESIGN.md#latency-budget-breakdown) |
| How does data flow? | [DATA_FLOW.md](architecture/DATA_FLOW.md) |

### API & Integration

| Question | Answer in |
|----------|-----------|
| What endpoints are available? | [API_SPEC.md - Core Endpoints](specifications/API_SPEC.md#core-endpoints) |
| How do I evaluate a transaction? | [API_SPEC.md - POST /transactions/evaluate](specifications/API_SPEC.md#1-post-transactionsevaluate) |
| What are the error codes? | [API_SPEC.md - Error Codes](specifications/API_SPEC.md#error-codes) |
| What are the SLA targets? | [API_SPEC.md - SLA](specifications/API_SPEC.md#sla) for each endpoint |
| How do rate limits work? | [API_SPEC.md - Rate Limiting](specifications/API_SPEC.md#rate-limiting) |

### Implementation

| Question | Answer in |
|----------|-----------|
| What's the implementation timeline? | [ROADMAP.md](ROADMAP.md) |
| What's Phase 1 (Foundation)? | [ROADMAP.md - Phase 1](ROADMAP.md#phase-1-foundation-weeks-1-4) |
| What's Phase 2 (ML Intelligence)? | [ROADMAP.md - Phase 2](ROADMAP.md#phase-2-ml-intelligence-weeks-5-10) |
| What's Phase 3 (Production Ready)? | [ROADMAP.md - Phase 3](ROADMAP.md#phase-3-production-readiness-weeks-11-14) |
| What's the Go-Live plan? | [ROADMAP.md - Phase 4](ROADMAP.md#phase-4-go-live--optimization-weeks-15-16) |
| What happens after launch? | [ROADMAP.md - Phase 5](ROADMAP.md#phase-5-continuous-improvement-ongoing) |

### Business & Metrics

| Question | Answer in |
|----------|-----------|
| What's the business case? | [OVERVIEW.md - Problem & Solution](docs/OVERVIEW.md) |
| What's the financial impact? | [OVERVIEW.md - Financial Projections](docs/OVERVIEW.md#financial-projections) or [README.md - Financial Impact](README.md#financial-impact) |
| What are target metrics? | [README.md - Key Metrics](README.md#key-metrics) |
| Who are target customers? | [OVERVIEW.md - Target Users](docs/OVERVIEW.md#target-users) |
| How do we compare to competitors? | [OVERVIEW.md - Competitive Analysis](docs/OVERVIEW.md#competitive-analysis) |

## Search by Keyword

### "Latency"
- [HIGH_LEVEL_DESIGN.md - Latency Budget](architecture/HIGH_LEVEL_DESIGN.md#latency-budget-breakdown)
- [COMPONENTS.md - SLA Targets](architecture/COMPONENTS.md)
- [API_SPEC.md - SLA sections](specifications/API_SPEC.md)

### "Accuracy / Precision / Recall"
- [FRAUD_DETECTION_STRATEGY.md - Detection Models](design/FRAUD_DETECTION_STRATEGY.md#detection-models)
- [FRAUD_DETECTION_STRATEGY.md - Success Metrics](design/FRAUD_DETECTION_STRATEGY.md#success-metrics)

### "Approval Rate"
- [README.md - Key Metrics](README.md#key-metrics)
- [APPROVAL_ENGINE.md - Approval Rate Optimization](design/APPROVAL_ENGINE.md#approval-rate-optimization)

### "Feature Engineering"
- [FRAUD_DETECTION_STRATEGY.md - Feature Engineering](design/FRAUD_DETECTION_STRATEGY.md#feature-engineering)

### "Monitoring / Metrics"
- [APPROVAL_ENGINE.md - Monitoring & Alerts](design/APPROVAL_ENGINE.md#monitoring--alerts)
- [FRAUD_DETECTION_STRATEGY.md - Performance Monitoring](design/FRAUD_DETECTION_STRATEGY.md#performance-monitoring)

### "Deployment"
- [ROADMAP.md - Phase 4](ROADMAP.md#phase-4-go-live--optimization-weeks-15-16)
- [HIGH_LEVEL_DESIGN.md - Scalability](architecture/HIGH_LEVEL_DESIGN.md#scalability-strategy)

### "Data Pipeline"
- [DATA_FLOW.md - Real-Time Flow](architecture/DATA_FLOW.md#real-time-transaction-flow)
- [DATA_FLOW.md - Batch Processing](architecture/DATA_FLOW.md#daily-batch-processing)

### "Model Training"
- [FRAUD_DETECTION_STRATEGY.md - Training & Monitoring](design/FRAUD_DETECTION_STRATEGY.md#training--monitoring)
- [DATA_FLOW.md - Weekly Retraining](architecture/DATA_FLOW.md#weekly-model-retraining)

---

## Document Statistics

| Category | Count |
|----------|-------|
| Total Documents | 11 |
| Root-Level Files | 5 |
| Architecture Files | 3 |
| Design Files | 2 |
| Specification Files | 1 |
| Documentation Files | 2 (docs folder) |
| Total Content | 15,000+ words |
| Code Examples | 20+ |
| Diagrams | 15+ |

## Version Info

- **Project**: AI-Powered Payment Approval & Fraud Intelligence
- **Version**: 1.0
- **Status**: Production Ready
- **Last Updated**: February 2026

---

**Start Here**: Not sure where to begin? See [QUICK_START.md](QUICK_START.md) for role-based reading paths.