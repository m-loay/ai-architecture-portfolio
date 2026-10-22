# ai-architecture-portfolio

AI Architecture Design course by MLOps MENA Academy

```text
ai-architecture-portfolio/
├── README.md        # your scenario, assumptions, how to read the repo
├── docs/            # one file per session: 01-framing.md, 02-building-blocks.md
├── diagrams/        # exported PNG/SVG + source files
├── adr/             # one file per decision: 0001-model-hosting.md
├── guardrails/      # automated checks run in CI
└── capstone/        # final architecture PDF + slides
```

## Scenario: Marketing Automation with AI & Agents

A retailer selling online (web + mobile app) in Saudi Arabia wants AI-driven marketing:

| Use case | What it does |
| --- | --- |
| Personalization | Homepage and product pages ranked per user, using a feature store |
| Next-best action (NBA) | Picks the single best offer/message for a customer right now |
| Cold start | Sensible recommendations for new users with no history |
| Agentic campaigns | An AI agent drafts campaigns (audience, message, channel, timing) for marketers |

**KPIs:** increase sales take-rate by 80%, increase CSAT to 70%, close the feedback loop (every outcome feeds back into models).
**Non-functional:** latency (real-time / near real-time / batch), scale at campaign peaks, explainability and business control, security, PDPL.

## Assumptions

Written down because the brief does not specify them; each one should be confirmed with stakeholders.

- Around 2M customers, 50k products, up to 10x traffic during campaigns (e.g. White Friday, Ramadan).
- Customer data lives in a CRM, an orders database and a web/app event stream.
- Customers and content are Arabic and English.
- The company already runs PostgreSQL and a cloud platform with an in-Kingdom region.
- Marketing content (copy for SMS, email, push) is public once sent; customer personal data is the sensitive asset.

## Sessions

### Session 1 - Governance pack

| Deliverable | File |
| --- | --- |
| Governance pack (wiki page) | [docs/01-governance-pack.md](docs/01-governance-pack.md) |
| Diagram | [diagrams/src/01-governance-architecture.drawio](diagrams/src/01-governance-architecture.drawio) |
| ADR 0001 | [LLM hosting](adr/0001-llm-hosting.md) |
| ADR 0002 | [Vector store](adr/0002-vector-store.md) |
| ADR 0003 | [Real-time vs batch](adr/0003-real-time-vs-batch.md) |
| Guardrail | [guardrails/check_governance.py](guardrails/check_governance.py) |

## References and credits
- Templates: [TOGAF principle template](https://www.opengroup.org/architecture/doc-review/protected-d3/htmlclean/chap23.html#tagtcjh_2) · [MADR ADR template](https://github.com/adr/madr/blob/develop/template/adr-template.md)
- Course brief: AI Architecture Design course, MLOps MENA Academy (Eng. Sameh Amin).
- Sources: [TOGAF architecture principles](https://www.opengroup.org/architecture/doc-review/protected-d3/htmlclean/chap23.html), [Nygard ADRs](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions), [MADR](https://adr.github.io/madr/). Full list in the [governance pack](docs/01-governance-pack.md#references).
- Repo and page layout adapted from a sample course portfolio for a different system (Bonod, a contract review assistant) <!-- TODO: add author/link -->. All content here is written for the Marketing Automation scenario.
