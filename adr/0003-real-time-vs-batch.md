# ADR-0003: Real-time vs batch

## Status
Accepted. Latency targets are assumptions to validate with load tests.

## Context
Each use case needs fresh data at a different speed:

| Use case | How fresh? | Who waits? |
| --- | --- | --- |
| Homepage / product page personalization | Seconds (what the user just clicked) | The user, while the page loads |
| Next-best action | Seconds to minutes | The user or the app |
| Cold start | Within the first session | The user |
| Campaign audiences and drafts | Hours to a day | The marketer, who reviews anyway (P2) |
| Model training | Daily or weekly | Nobody |

Campaign peaks bring up to 10x traffic.

## Decision driver
P3: real-time only where the user is waiting.

## Options considered
| Option | Benefits | Costs and constraints |
| --- | --- | --- |
| A: All batch (nightly scores in a table) | Simplest and cheapest | Ignores today's clicks; cold-start users get nothing useful until tomorrow |
| B: All real-time (compute everything per request) | Always fresh | Expensive; heavy features (purchase history) too slow per request; fragile at 10x peaks |
| C: Hybrid with a feature store | Heavy features computed in batch; session features streamed; light model scores at request time | Two pipelines; risk that training and serving compute features differently |

## Decision
Choose C:
- **Batch (nightly, Airflow):** purchase-history features, customer segments, model training, campaign audiences.
- **Near real-time (streaming, Kafka):** session events (views, add-to-cart) update the online feature store within seconds.
- **Real-time (on request):** ranking model reads the online feature store and responds with p95 < 150 ms; falls back to cached popular items if over budget (P3).
- **Campaign agent:** runs as a batch job; drafts wait for marketer approval (P2).

Use one feature store (e.g. Feast: offline + online) so training and serving share the same feature definitions.

## Trade-offs we accept
Two pipelines to operate and monitor. The feature store is a new component, but it removes training/serving skew.

## Revisit when
- A/B tests show session features do not lift take-rate (then simplify toward A).
- The p95 latency target is missed at 10x peak load.

## References
- Template: [MADR ADR template](https://github.com/adr/madr/blob/develop/template/adr-template.md)
- Format: Michael Nygard, [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) (2011), extended with Decision driver and Options considered from [MADR](https://adr.github.io/madr/).
- Principles P1–P5: [governance pack](../docs/01-governance-pack.md).
