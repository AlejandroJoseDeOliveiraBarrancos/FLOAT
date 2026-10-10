# Software Architecture Specification — FLOAT

Software-level specification of the FLOAT platform: architecture, service decomposition, integration flows, and the data pipeline.

---

## 1. Architectural Overview

FLOAT is an **asynchronous, event-driven medical factoring engine and dynamic liquidity manager**. It integrates a PyTorch-based LSTM time-series model with a **domain-driven outbox architecture** to analyze multi-month insurer reimbursement cycles, calculate dynamic advance rates, and forecast portfolio-level cash inflows.

### Core Architectural Principles

* **Asynchronous, event-driven processing:** Claims, remittances, disputes, and forecasts are propagated as domain events rather than synchronous calls, decoupling the doctor-facing portal from insurer-portal latency (30-120 day cycles).
* **Domain-Driven Design (DDD):** Backend is organized around domain modules (claims, doctors/clinics, insurers, disbursements, treasury).
* **Outbox Pattern / Event Bus:** Domain events are published via an outbox to guarantee reliable, at-least-once event delivery to downstream consumers (feature aggregation, insurer portal sync, dashboards).
* **Predictive microservice:** The FLOAT LSTM inference engine runs as a dedicated consumer of the feature pipeline, producing probability distributions instead of point estimates.

---

## 2. High-Level Architecture

```
[Doctor Submits Claim]
       |
       v
[Outbox Pattern / Event Bus]
       |
       v
[Feature Aggregation Service] (Transforms raw events into weekly insurer/clinic time-series vectors)
       |
       v
[LSTM Inference Engine] (Consumes sequence tensor -> outputs 30/60/90-day cash flow probability distributions)
       |
       v
[Treasury & Liquidity Dashboard UI] (Displays runway, confidence bands, and risk alerts)
```

Supporting integrations:

* **Insurer portal sync:** claims are submitted to insurer portals via automated pipelines; remittance advices flow back into the event bus.
* **Loan disbursement microservice:** receives forecast-driven guardrail signals (warnings or temporary advance-rate limits) and disburses funds to doctor bank accounts.

---

## 3. Service & Component Decomposition

| Component | Responsibility |
| --- | --- |
| **Doctor / Clinic Web Portal** | Claim submission, account management, document uploads (pre-authorizations, surgical logs, itemized billing), procedure metadata capture. |
| **Domain Backend Modules (DDD)** | Claims, doctor/clinic, insurer, disbursement, and treasury domains; publish domain events to the outbox. |
| **Outbox / Event Bus** | Reliable asynchronous publication of domain events (claim submitted, claim parsed, remittance processed, denial/resubmission, appeal resolved, forecast produced). |
| **Insurer Portal Integration Pipeline** | Automated submission of claims to health insurance carriers and ingestion of payment/reconciliation responses. |
| **Feature Aggregation Service** | Transforms raw events into weekly/monthly insurer-clinic time-series vectors; combines historical insurer payment velocity, procedure risk profiles, and clinic quality metrics into sequence vectors shaped `(batch_size, sequence_length, features)`. |
| **LSTM Inference Engine (FLOAT microservice)** | Consumes the sequence tensor and outputs 30/60/90-day cash flow probability distributions with 50%, 80%, and 95% confidence intervals; computes risk-adjusted dynamic advance rates. |
| **Loan Disbursement Microservice** | Disburses instant cash advances (e.g., 80%-90%) to doctors within 24 hours; enforces automated liquidity guardrails (warnings or temporary advance-rate limits) driven by forecasts. |
| **Treasury & Liquidity Dashboard UI** | Renders the Runway Gauge, confidence bands, and the insurer aging heatmap; exposes what-if insurer bottleneck simulation to treasury users. |

---

## 4. Complete System Interaction Flow

1. **Claim Capture & Parsing:** A doctor performs an operation, attaches supporting documentation, and submits the claim to the portal.
2. **Sequential Risk Scoring:** Claim metadata is passed to the feature aggregation pipeline, which combines historical insurer payment velocity, procedure risk profiles, and clinic quality metrics into a sequence vector.
3. **LSTM Cash Inflow Forecasting:** The FLOAT microservice consumes the sequence vector to predict the exact probability curve of cash settlement (30, 60, and 90 days out).
4. **Treasury Runway Adjustment:** The forecast updates the executive dashboard, dynamically adjusting platform liquidity buffers and recommending risk-adjusted advance rates for new submissions.
5. **Automated Execution & Portal Submission:** The claim is submitted to the insurer portal via automated pipelines, and funds are disbursed to the doctor's bank account.

---

## 5. Event Taxonomy (Domain Events on the Bus)

| Event | Source | Consumers |
| --- | --- | --- |
| Claim submitted / parsed | Doctor portal, claims domain | Feature aggregation service, insurer portal pipeline |
| Pre-authorization matched / failed | Claims domain | Feature aggregation service |
| Claim denied / resubmitted | Insurer portal integration | Feature aggregation service, dispute workflow |
| Appeal resolved | Dispute & exception management workflow | Feature aggregation service |
| Remittance advice processed | Insurer portal integration / payment ledger | Feature aggregation service (collection ratio, partial payments, write-offs) |
| Feature vector produced | Feature aggregation service | LSTM inference engine |
| Forecast produced | LSTM inference engine (FLOAT) | Dashboard UI, loan disbursement microservice (guardrails) |
| Guardrail warning / advance-rate limit | Treasury rules on forecast output | Loan disbursement microservice, treasury dashboard |

---

## 6. Data Architecture

* **Event store / outbox:** durable record of domain events enabling replay and audit of the full claim lifecycle.
* **Feature store:** weekly or monthly time-step vectors per insurer/clinic partnership, shaped `(batch_size, sequence_length, features)`.
* **Forecast store:** 30/60/90-day cash flow probability distributions with confidence intervals, consumed by the dashboard and guardrail evaluation.
* **Operational data:** doctor accounts, uploaded documents, insurer interfaces, and disbursement ledgers managed by the existing platform.

---

## 7. Key Integration Points

* **Health insurance carrier portals:** automated claim submission, reconciliation, and remittance ingestion over 30-120 day cycles.
* **Loan disbursement microservice:** direct connection for automated liquidity guardrails and doctor fund disbursement.
* **Doctor bank accounts:** disbursement of instant cash advances and residual balance (minus dynamic fee) upon settlement.
* **Treasury & liquidity dashboard:** real-time consumption of forecasts, confidence bands, and risk alerts by CFO, Head of Risk, and Treasury Operations Managers.
