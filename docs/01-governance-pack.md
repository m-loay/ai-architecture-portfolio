# Session 1 — Governance pack: Marketing Automation with AI & Agents

## Purpose
Govern an AI marketing platform (personalization, next-best action, cold start, campaign agent) so it grows sales take-rate and CSAT without breaking customer trust or PDPL. Scenario and assumptions: [README](../README.md).

## Principles
| Principle | Rule | Business goal | Verification |
| --- | --- | --- | --- |
| P1 Consent-first, minimum data | Use only data the customer consented to for marketing. Pseudonymize customer IDs at ingestion; no direct identifiers (name, phone, email, national ID) in features, prompts or logs. | PDPL compliance; customer trust (CSAT 70%). | Consent filter test; PII scanner on prompts and logs; DPO sign-off. |
| P2 Agents propose, humans approve | The campaign agent only drafts. A marketer approves audience, content and budget before anything is sent. Agent tools are read-only or draft-only, with spend and volume caps. | Brand safety and business control; no spam (CSAT). | No send tool exposed to the agent; approval recorded in audit log; cap tests. |
| P3 Right speed for the job | Real-time only where the user is waiting (homepage, NBA); everything else is batch or near real-time. Pages must render even if the model is down (fallback to popular items). | Take-rate +80% at controlled cost; survive campaign peaks. | p95 latency and load tests at 10x traffic; fallback test. |
| P4 Every decision is measured | Log every recommendation and campaign with model version and outcome. New models and campaigns ship behind A/B tests with a control group. | Proves the +80% take-rate; feeds the feedback loop. | A/B report per release; outcome events joined to decisions. |
| P5 Explainable and overridable | Each recommendation carries reason codes. Business rules (eligibility, stock, frequency caps, exclusions) run after the model and always win. | Explainability and business control; fewer irrelevant messages (CSAT). | Reason code present on 100% of responses; rule-override test. |

## ADR summary
| ADR | Decision | Driver |
| --- | --- | --- |
| [0001](../adr/0001-llm-hosting.md) LLM hosting | Managed LLM API behind an internal LLM gateway (PII removal, token budget, vendor swap) | P1, cost |
| [0002](../adr/0002-vector-store.md) Vector store | pgvector in existing PostgreSQL | Simplicity, P5 |
| [0003](../adr/0003-real-time-vs-batch.md) Real-time vs batch | Hybrid: batch + streaming into a feature store; real-time scoring only for homepage and NBA | P3 |

## ARB checklist
Each answer needs evidence (test report, dashboard, sign-off). **Release blockers** (any "no" stops launch): S2, D1, D2, E3, H1, H2.

| ID | Question | Principle |
| --- | --- | --- |
| **Security** | | |
| S1 | Marketer console uses SSO and role-based access; only approvers can approve sends? | P2 |
| S2 | Agent tools are read-only or draft-only; no tool can send messages or change prices? | P2 |
| S3 | Product reviews and other user text are treated as data, not instructions (prompt-injection test set passed)? | P2 |
| S4 | Secrets (API keys) in a vault; LLM calls only through the gateway? | P1 |
| **Privacy (PDPL)** | | |
| D1 | Only customers with marketing consent are used; opt-out takes effect within 24 h? | P1 |
| D2 | No direct identifiers in prompts, features or logs (PII scanner passed)? | P1 |
| D3 | Data stored in-Kingdom; any transfer outside is approved by the DPO? | P1 |
| D4 | Retention period defined; customer deletion removes features and logs? | P1 |
| **Evals** | | |
| E1 | Ranking model beats current baseline offline (e.g. NDCG@10) on a holdout set? | P4 |
| E2 | A/B test with control group planned, with the take-rate target written down before launch? | P4 |
| E3 | Generated copy: zero invented prices or offers, brand and Arabic quality pass on 50-sample LLM-as-judge plus human review? | P2, P5 |
| E4 | Cold-start recommendations beat "most popular" baseline? | P3, P4 |
| **Cost** | | |
| C1 | Cost per 1,000 recommendations and LLM cost per campaign estimated and monitored? | P3, ADR-0001 |
| C2 | Token budget per campaign enforced by the gateway; alert at 80%? | ADR-0001 |
| C3 | Load test at 10x peak passed within budget; autoscaling limits set? | P3 |
| **Human oversight** | | |
| H1 | Every campaign has a recorded human approval before send? | P2 |
| H2 | Frequency caps and a kill switch to stop a campaign or fall back to non-personalized pages? | P2, P3 |
| H3 | Every recommendation has reason codes visible to marketers? | P5 |
| H4 | Customer complaints and unsubscribes feed back into review and retraining? | P4 |

D1, D3 and D4 must be confirmed by someone with PDPL legal expertise (the DPO).

## Automated guardrail (CI)
[`guardrails/check_governance.py`](../guardrails/check_governance.py) runs on every pull request ([workflow](../.github/workflows/governance.yml)) and reviews the design documents against this pack:

| Check | Fails when |
| --- | --- |
| Principles linked to goals | A principle row has an empty business goal or verification |
| ADR completeness | An ADR is missing a required section (Status, Context, Options considered, Decision, Trade-offs we accept, Revisit when) or has fewer than 2 options |
| ADR traceability | An ADR does not cite at least one principle that exists here |
| ARB coverage | The checklist lacks one of Security, Privacy (PDPL), Evals, Cost, Human oversight, or a question has no principle |

It checks that the documents are complete and connected. It does not prove the running system obeys them; the ARB evidence does that.

## Diagram
[Editable draw.io architecture](../diagrams/src/01-governance-architecture.drawio): components tagged with the principle (P1–P5) and ADR they follow.

## References
Formats are adapted from public templates; the scenario, principles, decisions, checklist items and diagram are original to this system.
- **Templates:** [TOGAF principle template](https://www.opengroup.org/architecture/doc-review/protected-d3/htmlclean/chap23.html#tagtcjh_2) · [MADR ADR template](https://github.com/adr/madr/blob/develop/template/adr-template.md)
- **Brief and governance approach:** AI Architecture Design course, MLOps MENA Academy (Eng. Sameh Amin), slides 8, 22, 24, 26 and 36.
- **Principles format:** TOGAF architecture principle template (Name, Statement, Rationale, Implications), The Open Group, [Architecture Principles](https://www.opengroup.org/architecture/doc-review/protected-d3/htmlclean/chap23.html). Here Rule = Statement, Business goal = Rationale, Verification = Implications.
- **ADR format:** Michael Nygard, [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) (2011), and [MADR](https://adr.github.io/madr/) (Decision Drivers, Considered Options).
- **ARB checklist:** categories from the course brief (slide 36); security items informed by [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/).
- **Architecture patterns:** Eugene Yan, [System Design for Recommendations and Search](https://eugeneyan.com/writing/system-design-for-discovery/); [Feast](https://docs.feast.dev/) feature store; Anthropic, [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents).
- **Page layout:** section order and table layout adapted from a sample course portfolio for a different system (Bonod, a contract review assistant) <!-- TODO: add author/link -->. No principles, decisions or checklist items were reused from it.
