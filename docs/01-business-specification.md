# Business Specification — FLOAT

Business-level specification of the FLOAT platform.

---

## 1. Domain Context & Market Landscape

* **Target Market:** Medical Financing & Working Capital in Healthcare.

* **Product:** FLOAT (Factor Liquidity Optimization & Analytics Tool).

### Core Counterparties

* **The Sellers (Clients):** Doctors, surgeons, private clinics, and medical centers providing specialized surgeries, elective operations, and outpatient procedures.
* **The Debtors (Payers):** Private Health Insurance Companies, Managed Care Organizations, and Health Maintenance Organizations (HMOs).
* **The Factoring Platform (Us):** The financial technology intermediary providing non-recourse or limited-recourse invoice discounting and instant working capital.

---

## 2. Current Product Baseline (As-Is State)

### Operational Strengths

* **Established Core Lifecycle:** An operational web platform exists to manage claim submissions, doctor accounts, document uploads, and direct integrations with insurer interfaces.
* **Asynchronous Event Architecture:** The platform uses domain-driven backend modules and outbox event publishing patterns to process claims and sync with insurer portals asynchronously.

### Financial Engine Vulnerabilities (The Problem Area)

* **Oversimplified Pricing & Static Risk:** Financial variables are hardcoded as static constants across the board (e.g., a flat 85% advance rate and a fixed 4% monthly fee regardless of procedure type, doctor track record, or insurer behavior).
* **Opaque Portfolio Exposure:** The platform treats every claim as an isolated, static event rather than a sequence of time-sensitive interactions.
* **Unmonitored Liquidity Risk:** The treasury function lacks dynamic predictive visibility into actual payout turnaround times from health insurers, exposing the business to sudden cash conversion bottlenecks.

---

## 3. Business Model Evolution (To-Be State)

The business transitions from a simple **Flat-Fee Cash Advance Platform** to an **AI-Driven Dynamic Liquidity & Risk Management Engine for Healthcare**.

```
 +-------------------------------------------------------------------------+
 |                            DOCTOR / CLINIC                              |
 | Performs procedure -> Generates invoice -> Submits claim on portal      |
 +-------------------------------------------------------------------------+
                                      |
                                      v
 +-------------------------------------------------------------------------+
 |                       OUR FACTORING PLATFORM                            |
 |                                                                         |
 |  1. Instant Cash Advance (e.g., 80%-90%) paid to Doctor today         |
 |  2. Platform assumes duty of collection from Insurance Company          |
 |  3. Remaining balance paid to Doctor minus Dynamic Fee upon settlement  |
 +-------------------------------------------------------------------------+
                                      |
                                      v
 +-------------------------------------------------------------------------+
 |                       HEALTH INSURANCE CARRIER                          |
 |  Processes claim over 30-120 days -> Remits final payment to Platform   |
 +-------------------------------------------------------------------------+
```

---

## 4. Core Value Propositions by Stakeholder

| Stakeholder | Pain Point | Our Solution |
| --- | --- | --- |
| **Doctors / Surgeons** | Insurance companies take 30 to 120 days to pay, creating payroll and supply crunches. | Receive immediate liquidity (within 24 hours) for performed surgeries, turning unpaid insurance claims into instant cash flow. |
| **Factoring Treasury & Executive Team** | Insurer delays and rejections lead to unpredictable cash burn and potential insolvency. | **FLOAT** predicts cash inflows across 30-, 60-, and 90-day windows to prevent over-advancing capital. |
| **Institutional Lenders / Debt Partners** | High perceived risk in healthcare claim receivables limits funding lines. | Dynamic LSTM risk modeling provides continuous portfolio transparency, lowering the platform's cost of capital. |

---

## 5. Business Justification & Long-Term Value

### A. Prevention of the "Liquidity Trap"

* **The Win:** In medical factoring, insolvency rarely happens overnight; it happens when cash outflows (advancing money to doctors today) outpace cash inflows (waiting for insurance reimbursements) due to hidden delays.
* **Business Impact:** By providing an accurate, sequence-aware forecast of when insurance money will actually hit the bank, the business avoids sudden liquidity crunches, protecting operational continuity without needing to maintain excessively large, idle cash buffers.

### B. Lower Cost of Institutional Capital

* **The Win:** Factoring companies typically fund their operations via institutional credit lines or debt facilities. Institutional lenders demand rigorous visibility into portfolio cash conversion cycles.
* **Business Impact:** Demonstrating a predictive treasury dashboard powered by advanced time-series modeling builds immense credibility with lenders, allowing the business to negotiate lower interest rates on credit facilities and scale its lending capacity.

### C. Optimized Capital Allocation & Yield Maximization

* **The Win:** Knowing your exact liquidity runway allows you to deploy capital more aggressively during high-yield periods (e.g., peak elective surgery seasons) while tightening parameters during predictable insurer budget freezes.
* **Business Impact:** Maximizes the velocity of capital turnover, directly increasing return on invested capital (ROIC).
