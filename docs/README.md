# FLOAT — Documentation

FLOAT (Factor Liquidity Optimization & Analytics Tool) is an asynchronous, event-driven medical factoring engine and dynamic liquidity manager. It integrates a PyTorch-based LSTM time-series model with a domain-driven outbox architecture to analyze multi-month insurer reimbursement cycles, calculate dynamic advance rates, and forecast portfolio-level cash inflows — replacing constant-value fee assumptions with sequence-aware predictive analytics.

## Specification Index

| Level | Document | Contents |
| --- | --- | --- |
| Business | [01-business-specification.md](01-business-specification.md) | Domain context, counterparties, as-is baseline, business model evolution, stakeholder value propositions, business justification |
| Product | [02-product-specification.md](02-product-specification.md) | Objectives, functional capabilities, dashboard components, system interaction flows, data capture specification |
| Software | [03-software-architecture.md](03-software-architecture.md) | Event-driven architecture, outbox pattern, service decomposition, integration flows, event taxonomy, data pipeline |
| Model | [04-model-specification.md](04-model-specification.md) | LSTM model design, feature schema, tensor format, forecast outputs, confidence intervals |
