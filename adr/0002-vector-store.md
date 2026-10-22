# ADR-0002: Vector store

## Status
Accepted. Evidence is reasoning, not measurement.

## Context
Two features need similarity search over embeddings:
- **Cold start**: "products similar to what this new user just viewed" over ~50k product embeddings.
- **Campaign agent RAG**: retrieve brand guidelines, past campaigns and product facts so copy is accurate (no invented prices or discounts).

Both are small (under 1M vectors). Results must be filtered by business rules such as in stock, eligible category and language (P5). The team already runs PostgreSQL.

## Decision driver
Simplicity (lecture: "pick the simplest design that meets the requirements") and filtering for P5.

## Options considered
| Option | Benefits | Costs and constraints |
| --- | --- | --- |
| A: pgvector in existing PostgreSQL | No new system; vectors and product data in one place, so filters are plain SQL; team already knows it | Slower than dedicated engines at very large scale |
| B: Dedicated vector DB (Qdrant, Milvus, Pinecone) | Fast at large scale; rich vector features | New system to run, secure and back up; product data must be synced into it |
| C: Elasticsearch / OpenSearch hybrid | Keyword + vector search; good Arabic analyzers | Heaviest to operate; more than we need now |

## Decision
Choose A: pgvector with an HNSW index, in the existing in-Kingdom PostgreSQL.

## Trade-offs we accept
Lower peak performance than a dedicated engine. Keyword (exact name/SKU) search is weaker; we accept it because users do not search through this component.

## Revisit when
- Vectors grow past ~10M (e.g. per-user embeddings) or p95 query latency exceeds 50 ms.
- Agent RAG quality is poor because exact keyword matching is needed (then consider C).
- The embedding model changes: re-embed all products and documents, since queries and stored vectors must use the same model.

## References
- Template: [MADR ADR template](https://github.com/adr/madr/blob/develop/template/adr-template.md)
- Format: Michael Nygard, [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) (2011), extended with Decision driver and Options considered from [MADR](https://adr.github.io/madr/).
- Principles P1–P5: [governance pack](../docs/01-governance-pack.md).
