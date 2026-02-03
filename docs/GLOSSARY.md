# Glossary

Domain-specific terminology for the AI Payment Fraud Intelligence project.

## Business Terms

**APPROVE**: Automated decision to accept a transaction without manual review.

**CHARGEBACK**: Customer dispute of a transaction, typically resulting in fraud label after investigation.

**DECLINE**: Automated decision to reject a transaction and block payment.

**FRAUD LOSS**: Total financial loss from undetected fraudulent transactions.

**FALSE POSITIVE**: Legitimate transaction incorrectly classified as fraud (unnecessary decline).

**FALSE NEGATIVE**: Fraudulent transaction incorrectly classified as legitimate.

**REVIEW**: Manual human review of a transaction with borderline fraud score.

**MERCHANT**: Business or entity accepting customer payments (e.g., online retailer, service provider).

**TRANSACTION**: Individual payment event (card swipe, online purchase, money transfer, etc.).

**VELOCITY**: Speed or frequency of transactions (e.g., 5 transactions in 5 minutes).

---

## Technical Terms

**ALGORITHM**: Step-by-step procedure for solving a problem or making decisions.

**API**: Application Programming Interface; standardized way for systems to communicate.

**BATCH PROCESSING**: Grouping transactions for processing together at scheduled times (e.g., daily).

**CACHE**: Fast-access memory storing frequently used data (e.g., Redis).

**LATENCY**: Time delay between request and response (measured in milliseconds).

**THROUGHPUT**: Number of transactions processed per unit time (e.g., 100k tps = 100k per second).

**REAL-TIME**: Immediate or near-immediate processing without significant delay.

**SLA**: Service Level Agreement; guaranteed performance metrics (e.g., <100ms latency).

**UPTIME**: Percentage of time system is operational (e.g., 99.99% = 52 minutes downtime/year).

**FEATURE**: Input variable used in machine learning (e.g., transaction amount, device ID).

---

## Machine Learning Terms

**ACCURACY**: Percentage of correct predictions (both fraud and legitimate correctly identified).

**AUC-ROC**: Area Under the Receiver Operating Characteristic curve; measure of model discrimination (0=random, 1=perfect).

**CLASSIFICATION**: Task of predicting discrete categories (e.g., FRAUD vs LEGITIMATE).

**CROSS-VALIDATION**: Technique for evaluating model by splitting data multiple ways.

**DRIFT DETECTION**: Monitoring for changes in data or model performance over time.

**ENSEMBLE**: Combining multiple models for better predictions.

**FEATURE ENGINEERING**: Creating new variables from raw data to improve model performance.

**HYPERPARAMETER**: Parameter set before training that controls model learning (e.g., learning rate).

**INFERENCE**: Using trained model to make predictions on new data.

**METRIC**: Measurement of model performance (precision, recall, F1, AUC-ROC).

**MODEL**: Mathematical representation learned from data to make predictions.

**OVERFITTING**: Model memorizing training data rather than learning generalizable patterns.

**PRECISION**: Percentage of predicted fraud cases that are actually fraudulent (TP / (TP + FP)).

**RECALL**: Percentage of actual fraudulent cases correctly identified (TP / (TP + FN)).

**F1 SCORE**: Harmonic mean of precision and recall; balanced accuracy metric.

**TRAINING**: Process of adjusting model parameters using labeled historical data.

**VALIDATION**: Evaluating model performance on held-out data not used for training.

**WEIGHTED ENSEMBLE**: Combining models where different weights control each model's influence.

---

## Fraud-Specific Terms

**ACCOUNT TAKEOVER (ATO)**: Criminal gains control of legitimate customer account.

**CARD TESTING**: Small transactions sent to multiple merchants to identify valid stolen cards.

**CHARGEBACK RATE**: Percentage of transactions disputed by customers.

**FRIENDLY FRAUD**: Customer falsely disputes legitimate transaction to get refund.

**IMPOSSIBLE TRAVEL**: Transaction from account from geographically impossible locations within short timeframe.

**PHISHING**: Social engineering attack to steal credentials or financial information.

**REFUND ABUSE**: Customer repeatedly purchasing, receiving, and returning items to steal goods/money.

**STOLEN CARD**: Fraudulent use of payment card obtained through theft or data breach.

**SYNTHETIC IDENTITY FRAUD**: Criminals create fake identity using mix of real and fabricated information.

**VELOCITY CHECK**: Rule detecting multiple transactions from same card in short period.

---

## Compliance & Security Terms

**COMPLIANCE**: Adherence to laws, regulations, and industry standards (PCI-DSS, AML, KYC, etc.).

**KYC**: Know Your Customer; process of verifying customer identity and assessing risk.

**AML**: Anti-Money Laundering; processes to detect and prevent money laundering.

**PCI-DSS**: Payment Card Industry Data Security Standard; standard for handling card data.

**GDPR**: General Data Protection Regulation; EU privacy law.

**CCPA**: California Consumer Privacy Act; US privacy law.

**OAUTH 2.0**: Industry-standard authorization protocol for secure authentication.

**RATE LIMITING**: Restricting number of API requests per time period to prevent abuse.

**AUTHENTICATION**: Verifying identity of user or system.

**AUTHORIZATION**: Granting permissions to authenticated user/system.

---

## Architecture Terms

**MICROSERVICES**: Architecture where system divided into small, independent services.

**EVENT-DRIVEN**: Architecture responding to events (transactions, user actions) rather than polling.

**DISTRIBUTED**: System spread across multiple servers/regions for reliability and scale.

**HORIZONTAL SCALING**: Adding more servers to handle increased load.

**LOAD BALANCING**: Distributing traffic across multiple servers for efficiency.

**FAILOVER**: Automatic switching to backup system when primary fails.

**HIGH AVAILABILITY (HA)**: System designed to remain operational with minimal downtime.

**DISASTER RECOVERY (DR)**: Procedures and systems for recovering from major failures.

**MULTI-REGION**: Services deployed across geographic regions for redundancy and performance.

**STATELESS**: Service that doesn't store state between requests (allows scaling).

**CONTAINERIZATION**: Packaging application and dependencies in isolated container (Docker).

**ORCHESTRATION**: Automated management of containers across servers (Kubernetes).

---

## Specific Model Terms

**XGBOOST**: Gradient boosting framework optimized for speed and performance; used for supervised fraud detection.

**LSTM**: Long Short-Term Memory neural network; captures sequential patterns (transactions over time).

**ISOLATION FOREST**: Unsupervised anomaly detection algorithm; identifies unusual transactions.

**GRADIENT BOOSTING**: Ensemble technique combining weak learners iteratively.

**DECISION TREE**: Model making predictions through series of if-then rules.

**NEURAL NETWORK**: Model inspired by biological neurons; foundation for deep learning.

**ATTENTION MECHANISM**: Technique allowing model to focus on important parts of input.

**ANOMALY DETECTION**: Identifying data points significantly different from normal patterns.

---

## Metrics & Performance Terms

**P50**: 50th percentile; median latency (half requests faster, half slower).

**P95**: 95th percentile; 95% of requests faster, 5% slower.

**P99**: 99th percentile; 99% of requests faster, 1% slower.

**THROUGHPUT**: Number of transactions processed per unit time.

**LATENCY BUDGET**: Total allowable delay distributed across system components.

**JITTER**: Variability in latency (inconsistent response times).

**TAIL LATENCY**: Performance of slowest requests (P99, P99.9).

**SERVICE LEVEL OBJECTIVE (SLO)**: Target reliability level for a service.

**SERVICE LEVEL INDICATOR (SLI)**: Actual measurement of service reliability.

**MEAN TIME TO RECOVERY (MTTR)**: Average time to fix system failure.

**MEAN TIME BETWEEN FAILURES (MTBF)**: Average time between system failures.

---

## Data Terms

**FEATURE STORE**: Centralized repository for features accessible by ML models.

**DATA WAREHOUSE**: Centralized database for analytics and reporting (BigQuery).

**OLTP**: Online Transaction Processing; optimized for high-volume transactions (PostgreSQL).

**OLAP**: Online Analytical Processing; optimized for complex queries (BigQuery).

**TIME-SERIES DATA**: Data with timestamp, tracking metrics over time (InfluxDB).

**DATA PIPELINE**: Process moving data from source through transformations to destination.

**ETL**: Extract, Transform, Load; process for data integration.

**SCHEMA**: Structure/format of data (what fields, types, constraints).

**REPLICATION**: Copying data across multiple locations for redundancy.

**CONSISTENCY**: All copies of data are synchronized.

---

## Testing & Deployment Terms

**UNIT TEST**: Testing individual functions/components in isolation.

**INTEGRATION TEST**: Testing multiple components working together.

**LOAD TEST**: Testing system behavior under expected traffic volume.

**STRESS TEST**: Testing system beyond expected load to find breaking point.

**SMOKE TEST**: Quick test checking basic system functionality.

**REGRESSION TEST**: Testing that changes don't break existing functionality.

**A/B TEST**: Comparing two versions by showing different users different versions.

**CANARY DEPLOYMENT**: Gradually rolling out changes to small subset first.

**BLUE-GREEN DEPLOYMENT**: Switching between two identical production environments.

**ROLLING DEPLOYMENT**: Gradually updating servers without full downtime.

**ROLLBACK**: Reverting to previous version if new version has issues.

**CI/CD**: Continuous Integration/Continuous Delivery; automated testing and deployment.

---

## Decision Engine Terms

**THRESHOLD**: Score cutoff determining decision (e.g., score >0.7 = DECLINE).

**MODIFIER**: Adjustment to threshold based on context (e.g., -5% for VIP account).

**DECISION BOUNDARY**: Score range determining decision category (APPROVE/REVIEW/DECLINE).

**CONFIDENCE**: Probability/certainty of decision being correct.

**DECISION FACTOR**: Input influencing final decision (e.g., "low_fraud_score", "trusted_account").

**DECISION METADATA**: Information about how decision was made (scores, factors, thresholds).

**BUSINESS RULE**: Hard constraint enforced regardless of model score (e.g., velocity limit).

---

## Monitoring Terms

**ALERT**: Notification triggered when metric exceeds threshold.

**DASHBOARD**: Visual display of key metrics and status.

**LOG**: Record of system events and transactions.

**TRACE**: Detailed record of request path through system.

**METRIC**: Quantitative measurement of system performance.

**ANOMALY ALERT**: Alert triggered when metric behaves unexpectedly (statistical detection).

**ESCALATION**: Routing alert to higher-level team if not resolved quickly.

**ON-CALL**: Engineer responsible for responding to alerts outside business hours.

**RUNBOOK**: Documented procedures for responding to specific incidents.

---

## Customer Experience Terms

**APPROVAL RATE**: Percentage of transactions automatically approved without review.

**DECLINE RATE**: Percentage of transactions automatically declined.

**REVIEW RATE**: Percentage of transactions sent for manual review.

**CUSTOMER SATISFACTION (CSAT)**: Survey metric rating customer satisfaction (1-5 scale).

**NET PROMOTER SCORE (NPS)**: Likelihood customer would recommend service (0-100 scale).

**FALSE DECLINE**: Legitimate transaction incorrectly declined (hurts customer satisfaction).

**APPEAL**: Customer requesting review of declined transaction.

---

## Financial Terms

**ROI**: Return on Investment; profit generated relative to cost.

**TCO**: Total Cost of Ownership; all costs associated with system.

**PAYBACK PERIOD**: Time for system to generate profits equal to implementation cost.

**FRAUD LOSS**: Financial loss from undetected fraudulent transactions.

**OPERATIONAL COST**: Cost of running system (personnel, infrastructure).

**REVENUE IMPACT**: Additional revenue or cost savings from system.

---

## Common Abbreviations

| Abbr | Meaning |
|------|---------|
| ATO | Account Takeover |
| AUC | Area Under Curve |
| AML | Anti-Money Laundering |
| API | Application Programming Interface |
| CI/CD | Continuous Integration/Continuous Delivery |
| CSAT | Customer Satisfaction |
| DR | Disaster Recovery |
| ELK | Elasticsearch-Logstash-Kibana |
| ETL | Extract Transform Load |
| FN | False Negative |
| FP | False Positive |
| FTE | Full-Time Equivalent |
| HA | High Availability |
| KYC | Know Your Customer |
| LSTM | Long Short-Term Memory |
| MTTR | Mean Time To Recovery |
| NPS | Net Promoter Score |
| OLAP | Online Analytical Processing |
| OLTP | Online Transaction Processing |
| P50/P95/P99 | Percentile latencies |
| PCI | Payment Card Industry |
| ROI | Return on Investment |
| SLA | Service Level Agreement |
| SLO | Service Level Objective |
| SLI | Service Level Indicator |
| tps | Transactions Per Second |
| TP | True Positive |
| TN | True Negative |

---

**Glossary Version**: 1.0
**Last Updated**: February 2026