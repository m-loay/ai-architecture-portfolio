# ADR-0001: LLM hosting

## Status
Accepted. Evidence is reasoning, not measurement; the Arabic quality eval below confirms or reverses it.

## Context
The campaign agent needs an LLM to plan campaigns and write Arabic and English copy. Copy becomes public once sent. Personal data is the sensitive asset, and P1 already keeps it out of prompts: the agent sees segment statistics ("12,400 customers, 60% bought electronics"), never individual customers. Usage is bursty: heavy before campaigns, near zero otherwise. Personalization and NBA scoring do not use the LLM (see ADR-0003).

## Decision driver
P1 (no personal data leaves) and cost at bursty volume.

## Options considered
| Option | Benefits | Costs and constraints |
| --- | --- | --- |
| A: Managed LLM API (frontier model) | Best Arabic copy and agent reasoning; no GPUs to run; pay per use suits bursty load | Data leaves our platform; vendor lock-in; per-token cost |
| B: Self-hosted open model (e.g. Llama, Qwen, Jais on vLLM) | Data stays inside; full control | GPUs idle most of the month; ops team needed; weaker Arabic copy and tool use |
| C: Hybrid: API for generation, small self-hosted model for PII redaction | Strong quality plus a local safety net | Two stacks to operate |

## Decision
Choose A, called only through an internal **LLM gateway** that removes PII, enforces a token budget per campaign, logs prompts and responses (without PII), and lets us swap vendors. Prefer an in-Kingdom region or a provider that satisfies PDPL transfer rules (to confirm with the DPO).

## Trade-offs we accept
Vendor dependency and per-token cost. Prompts (segment stats, product facts, brand guidelines) leave the platform, which is acceptable only because P1 removes personal data first.

## Revisit when
- Legal decides even pseudonymized segment data cannot leave the Kingdom.
- Monthly token cost exceeds the cost of a dedicated GPU node.
- An open model matches the API on our Arabic copy eval (then consider option B or C).

## References
- Template: [MADR ADR template](https://github.com/adr/madr/blob/develop/template/adr-template.md)
- Format: Michael Nygard, [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) (2011), extended with Decision driver and Options considered from [MADR](https://adr.github.io/madr/).
- Principles P1–P5: [governance pack](../docs/01-governance-pack.md).
