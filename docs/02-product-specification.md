# Product Specification — FLOAT

Product-level specification of the FLOAT platform.

---

## 1. Product Overview & Objectives

* **Product Name:** **FLOAT (Factor Liquidity Optimization & Analytics Tool)**
* **Target Users:** Chief Financial Officer (CFO), Head of Risk, and Treasury Operations Managers at the factoring firm.
* **Core Objective:** Replace static, oversimplified financial assumptions with a dynamic, LSTM-powered forecasting engine that accurately predicts cash inflows from insurance companies over a **30-, 60-, and 90-day horizon**.
* **The Problem It Solves:** Factoring companies advance immediate cash to doctors for performed procedures, taking on the burden of collecting from health insurers. If an insurer's processing timeline quietly stretches from 45 days to 90 days, a flat financial model leads to severe liquidity crunches. FLOAT provides real-time visibility into actual cash realization curves, preventing the platform from over-advancing funds.

---

## 2. Functional Requirements & User Experience

### A. Core Functional Capabilities

1. **Probabilistic Cash Inflow Projections:** Generates expected cash recovery curves across rolling 30-, 60-, and 90-day windows for all outstanding factored surgical and procedural invoices.
2. **Insurer Bottleneck Simulation:** Allows treasury teams to run "what-if" scenarios (e.g., *"What happens to our liquidity if Insurer X slows down claim disbursements by 20% next month?"*).
3. **Automated Liquidity Guardrails:** Connects directly with the loan disbursement microservice to trigger automated warnings — or temporary limits — on advance rates when projected liquidity drops below safe operational thresholds.

### B. Dashboard Interface Components

* **The Runway Gauge:** A high-level visual indicator showing current available liquid capital versus projected cash inflows and upcoming doctor disbursement commitments.
* **Confidence Bands (Probabilistic Forecasts):** Instead of a single static line, cash inflow projections are displayed with 50%, 80%, and 95% confidence intervals driven by the LSTM's hidden state distributions.
* **Insurer Aging Heatmap:** A grid breaking down each insurance carrier by their current payment velocity, error/rejection rates, and volume of outstanding factored capital.

---

## 3. Complete System Interaction Flow

1. **Claim Capture & Parsing:** A doctor performs an operation, attaches the supporting documentation (pre-authorizations, surgical logs, itemized billing), and submits the claim to the portal.
2. **Sequential Risk Scoring:** The claim metadata is passed to the feature aggregation pipeline, which combines historical insurer payment velocity, procedure risk profiles, and clinic quality metrics into a sequence vector.
3. **LSTM Cash Inflow Forecasting:** The **FLOAT** inference microservice consumes the sequence vector to predict the exact probability curve of cash settlement (30, 60, and 90 days out).
4. **Treasury Runway Adjustment:** The forecast updates the executive dashboard, dynamically adjusting platform liquidity buffers and recommending risk-adjusted advance rates for new submissions.
5. **Automated Execution & Portal Submission:** The claim is submitted to the insurer portal via automated pipelines, and funds are disbursed to the doctor's bank account.

---

## 4. Data Capture Specification

To power the LSTM model behind this dashboard, the factoring software must transition from storing static invoice attributes to capturing **rich, time-stamped event sequences**.

Data is aggregated into weekly or monthly time steps to form the input tensor: `(batch_size, sequence_length, features)`.

### A. Data Capture Schema (Per Insurer / Clinic Partnership per Time Step)

| Feature Category | Data Points to Log / Capture | Description & Collection Source |
| --- | --- | --- |
| **Claim Aging & Velocity** | **Days Sales Outstanding (DSO):** Rolling average days from invoice submission to full cash settlement.<br>**Aging Brackets:** Volume of invoices sitting at 0-30, 31-60, 61-90, and 90+ days. | Captured via the insurance claims submission and reconciliation interface. |
| **Cash Realization & Haircuts** | **Collection Ratio:** Actual cash collected divided by the initial invoice face value.<br>**Partial Payment Frequency:** Rate at which insurers pay a percentage rather than the full amount.<br>**Write-Off Volume:** Total monetary value of uncollectible or permanently denied claims. | Logged when the insurer remittance advice (payment ledger) is processed. |
| **Procedure & Specialty Risk Weights** | **Specialty Mix Ratio:** Proportion of claims tied to high-scrutiny procedures (e.g., complex orthopedic or cardiovascular surgeries) vs. routine outpatient procedures.<br>**Pre-Authorization Compliance Rate:** Percentage of claims with pre-approval flags matched successfully. | Extracted from the doctor's initial procedure submission metadata. |
| **Administrative Friction** | **Denial / Resubmission Rate:** Volume of claims rejected on first pass requiring administrative correction.<br>**Appeal Turnaround Time:** Average days required to resolve disputed claims with the insurer. | Logged via the back-office dispute and exception management workflow. |
| **Macro / Calendar Indicators** | **Fiscal Quarter Markers:** Binary flags for end-of-quarter or end-of-year insurance budget cycles.<br>**Holiday & Seasonality Weights:** Indicators tracking known elective surgery dips (e.g., holiday seasons). | System-generated calendar features. |

### B. Data Pipeline Integration Workflow

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
