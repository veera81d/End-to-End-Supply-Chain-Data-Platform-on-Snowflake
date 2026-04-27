# ADR-001: Medallion Architecture

## Status
Accepted

## Context

The platform ingests 13 source entities from Amazon S3 and must support auditability, transformation, quality control, and business-ready analytics. The white paper shows a three-layer architecture with Raw, Stage, and Curated layers, each serving a distinct role in the data lifecycle.

## Decision

We will use a **medallion architecture** with three layers:

- **Raw** for exact source replication and auditability
- **Stage** for denormalized transformation logic and business rules
- **Curated** for dimensional modeling, semantic views, and AI-ready analytics

## Rationale

The layered approach keeps source fidelity separate from transformation logic and analytical modeling. It also supports clearer ownership, simpler debugging, and a stable downstream contract for semantic and AI consumption.

## Consequences

### Positive
- Easier traceability from source to insight
- Clean separation of concerns
- Better support for governance and data quality
- Flexible downstream consumption through semantic and AI layers

### Trade-offs
- More orchestration and storage than a single-layer design
- Requires discipline in promoting logic to the correct layer
- Full-refresh stage processing may need to evolve for scale

## References

- Raw layer: `DATA_RAW.SUPPLY_CHAIN`
- Stage layer: `DATA_STAGE.SUPPLY_CHAIN`
- Curated layer: `DATA_CURATED.SUPPLY_CHAIN`
