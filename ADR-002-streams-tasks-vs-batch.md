# ADR-002: Streams and Tasks vs Batch

## Status
Accepted

## Context

The source system changes continuously and the platform needs timely propagation of inserts, updates, and deletes into downstream analytical structures. The white paper describes 13 Streams on the raw tables and a Task DAG with a 5-minute root scheduler to trigger stage refreshes only when data changes.

## Decision

We will use **Streams and Tasks** for change data capture and orchestration instead of a purely scheduled batch refresh pattern.

## Rationale

Streams capture row-level changes in the raw tables, and Tasks provide lightweight orchestration with conditional execution through `SYSTEM$STREAM_HAS_DATA`. This reduces unnecessary work, keeps refreshes aligned to actual data change, and fits the near-real-time design of the platform.

## Consequences

### Positive
- Less wasted compute than unconditional batch jobs
- Better alignment with data arrival patterns
- Clear trigger model for downstream refreshes
- Easier to reason about CDC across source entities

### Trade-offs
- More operational logic than a simple nightly batch
- Full-refresh stored procedures can be heavier than incremental merge logic
- Task orchestration must be maintained carefully as the platform grows

## Future Improvement

The current design works well for consistency, but the white paper recommends moving high-volume stage processing toward **MERGE-based incremental loading** over time.

## References

- Raw Streams: 13 delta streams on source tables
- Task root scheduler: `TASK_STG_ROOT_SCHEDULER`
- Conditional execution: `SYSTEM$STREAM_HAS_DATA`
