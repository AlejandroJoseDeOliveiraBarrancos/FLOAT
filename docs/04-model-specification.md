# Model Specification — LSTM Cash Recovery Forecasting Engine

Model-level specification of the PyTorch-based LSTM time-series model at the core of FLOAT.

---

## 1. Model Overview

* **Model type:** PyTorch-based LSTM (Long Short-Term Memory) time-series model.
* **Purpose:** Analyze multi-month insurer reimbursement cycles, calculate dynamic advance rates, and forecast portfolio-level cash inflows.
* **Design intent:** Replace constant-value fee assumptions (e.g., flat 85% advance rate, fixed 4% monthly fee) with sequence-aware predictive analytics. Every claim is treated as a sequence of time-sensitive interactions rather than an isolated, static event.

---

## 2. Input Specification

### A. Granularity & Shape

* Data is captured as **rich, time-stamped event sequences** per insurer / clinic partnership.
* Events are aggregated into **weekly or monthly time steps**.
* Input tensor shape: **`(batch_size, sequence_length, features)`** — a multi-month sequence of feature vectors.

### B. Feature Schema (Per Insurer / Clinic Partnership per Time Step)

| Feature Category | Features | Source |
| --- | --- | --- |
| **Claim Aging & Velocity** | Days Sales Outstanding (DSO) — rolling average days from invoice submission to full cash settlement; Aging brackets — volume of invoices at 0-30, 31-60, 61-90, 90+ days | Insurance claims submission and reconciliation interface |
| **Cash Realization & Haircuts** | Collection ratio (actual cash collected / initial invoice face value); Partial payment frequency; Write-off volume (uncollectible or permanently denied claims) | Insurer remittance advice (payment ledger) processing |
| **Procedure & Specialty Risk Weights** | Specialty mix ratio (high-scrutiny procedures such as complex orthopedic or cardiovascular surgeries vs. routine outpatient); Pre-authorization compliance rate | Doctor's initial procedure submission metadata |
| **Administrative Friction** | Denial / resubmission rate (claims rejected on first pass); Appeal turnaround time (average days to resolve disputed claims) | Back-office dispute and exception management workflow |
| **Macro / Calendar Indicators** | Fiscal quarter markers (binary flags for end-of-quarter / end-of-year insurance budget cycles); Holiday & seasonality weights (known elective surgery dips) | System-generated calendar features |

---

## 3. Output Specification

* **Primary output:** 30-, 60-, and 90-day **cash flow probability distributions** for outstanding factored surgical and procedural invoices (expected cash recovery curves).
* **Confidence bands:** projections are emitted with **50%, 80%, and 95% confidence intervals**, driven by the **LSTM's hidden state distributions** — displayed in the dashboard instead of a single static line.
* **Derived outputs:**
  * Risk-adjusted **dynamic advance rates** for new submissions (replacing the flat advance-rate constant).
  * **Payment velocity** signals per insurer feeding the insurer aging heatmap.
  * Liquidity runway inputs for the Runway Gauge (available liquid capital vs. projected inflows vs. upcoming doctor disbursement commitments).
  * Guardrail triggers (warnings or temporary advance-rate limits) when projected liquidity drops below safe operational thresholds.

---

## 4. Consumption & Simulation

* **Insurer bottleneck simulation:** treasury teams run what-if scenarios on the forecast (e.g., *"What happens to our liquidity if Insurer X slows down claim disbursements by 20% next month?"*) by perturbing insurer-specific sequence inputs and re-running inference.
* **Automated guardrails:** forecast outputs connect directly to the loan disbursement microservice to enforce advance-rate limits.
* **Portfolio view:** per-insurer/clinic forecasts aggregate to portfolio-level cash inflow projections across rolling 30/60/90-day windows.

---

## 5. Model Lifecycle Data Flow

```
[Doctor Submits Claim]
       |
       v
[Outbox Pattern / Event Bus]
       |
       v
[Feature Aggregation Service] (raw events -> weekly/monthly insurer-clinic time-series vectors)
       |
       v
[LSTM Inference Engine] (sequence tensor -> 30/60/90-day cash flow probability distributions)
       |
       v
[Treasury & Liquidity Dashboard UI] (runway, confidence bands, risk alerts)
```

* **Training/learning signal:** historical event sequences (claim aging, realization, friction, macro indicators) with realized settlement outcomes from remittance advices.
* **Inference path:** feature aggregation service produces the sequence tensor on each event batch; the FLOAT inference microservice emits updated probability distributions that refresh the dashboard and guardrail evaluation.
