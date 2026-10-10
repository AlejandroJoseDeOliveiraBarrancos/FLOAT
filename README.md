# FLOAT
FLOAT is a PyTorch-based LSTM time-series model with domain-driven outbox architecture that analyzes multi-month insurer reimbursement cycles, calculate dynamic advance rates, and forecast portfolio-level cash inflows. Designed to replace constant-value fee assumptions with sequence-aware predictive analytics.

FLOAT stands for **Factor Liquidity Optimization & Analytics Tool**.


## Documentation

Full project specifications are available in the [docs/](docs/) folder:

* [Business specification](docs/01-business-specification.md) — domain context, stakeholders, business model evolution, value justification
* [Product specification](docs/02-product-specification.md) — functional capabilities, dashboard components, interaction flows, data capture
* [Software architecture](docs/03-software-architecture.md) — event-driven architecture, outbox pattern, service decomposition, data pipeline
* [Model specification](docs/04-model-specification.md) — LSTM design, feature schema, tensor format, forecast outputs
