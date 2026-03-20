---
name: slow-plan-optimization
description: Template skill for slow plan optimization in TiDB. Use when adding or refining workflow instructions, references, and scripts for diagnosing and improving slow or regressed execution plans.
---

# Slow Plan Optimization

## Required Inputs

- Require the original SQL text (exact query sent to TiDB).
- Prefer real execution plans from `EXPLAIN ANALYZE` with runtime details (time cost, rows, loops, memory/disk when available).
- Ask whether the user can provide the related table schema (`SHOW CREATE TABLE` for involved tables).
- If the user only provides `EXPLAIN` (no runtime stats), ask for `EXPLAIN ANALYZE` before giving detailed optimization advice.
- Ask for missing required inputs before giving optimization advice.

## Workflow

- Confirm required inputs first (SQL, `EXPLAIN ANALYZE`, and schema if available).
- Identify the bottleneck operator in the execution plan first.
  - Define bottleneck operator as the operator that accounts for the largest time cost in the plan.
- Decide whether this is a query plan problem before proposing plan tuning.
  - If the bottleneck operator processes only a small number of rows but still takes a long time, treat it as likely not a query plan problem.
- If it is not a query plan problem, or there is no clear room for plan-side improvement, do not provide plan optimization suggestions.
  - Output a clear conclusion such as: "Can't improve this query from the plan perspective; this might be a <component> problem."
- Use bottleneck-targeted methods first if and only if it is a query plan problem.
- Figure out optimization methods that directly target that bottleneck operator.
- Output suggestions in this strict order:
  1. Create SQL binding or add optimizer hints.
  2. Add new indexes.
  3. Rewrite the query.
- Re-run `EXPLAIN ANALYZE` to verify each applied change.

## Guardrails

- Keep recommendation priority fixed: binding/hints -> indexes -> query rewrite.
- Keep analysis priority fixed: identify bottleneck operator -> optimize bottleneck -> output suggestions.
- Keep diagnosis priority fixed: determine plan problem vs non-plan problem before plan changes.
- If diagnosis says non-plan problem, stop plan tuning and return only the non-plan conclusion.
- Do not jump to index or SQL rewrite suggestions before considering binding/hints.
- Use runtime evidence from `EXPLAIN ANALYZE` to justify each recommendation.

## Non-Plan Cases

- `PointGet` or `BatchPointGet` as bottleneck usually means the plan is already optimal; suspect TiKV hotspot or storage-side latency.
- Small `IndexRangeScan`/`TableRangeScan` (for example ~100 rows) but long latency usually points to TiKV-side issues, not plan shape.
- Small-row `Hash`/`Agg`/`Sort` (for example ~1000 rows) but long latency is usually not a query plan problem.

## Useful References

- TODO: Add reference links.
