# Database Indexing

## What is an Index?

An index is a data structure that speeds up queries by providing 
a fast access path to the data — like a book index.

Without index: Full Table Scan — O(n)
With index: B-Tree Lookup — O(log n)

## Index Types in PostgreSQL

| Type | Good for | Example |
|------|----------|---------|
| B-Tree (default) | =, <, >, BETWEEN, ORDER BY | IP addresses, timestamps |
| Hash | = only (equality) | Exact lookups |
| GIN | JSONB, Arrays, Full-text | metadata columns |
| Partial | Subset of data | WHERE is_active = TRUE |

## Composite Index

```sql
CREATE INDEX idx ON table (col_a, col_b);
```

Works left to right only:
- WHERE col_a = X 
- WHERE col_a = X AND col_b = Y 
- WHERE col_b = Y  (index not used)

## Trade-off

More indexes = faster reads + slower writes.
Every INSERT/UPDATE must also update the index.

## When NOT to Index

- Small tables (<1000 rows)
- Low cardinality columns (boolean) — use Partial Index instead
- Columns rarely used in WHERE/ORDER BY

## Connection to ThreatStream

4 indexes on the events table:
- `type` → GROUP BY for attack-type chart
- `source_ip` → top-attackers query
- `timestamp` → timeline chart (range query)
- `event_id` → individual event lookup

Partial indexes on:
- `sessions.is_active WHERE TRUE` → only active sessions
- `threat_alerts.is_resolved WHERE FALSE` → only open alerts

## Practical Rule

Index columns that appear in WHERE, JOIN, ORDER BY and GROUP BY.
Use EXPLAIN ANALYZE to verify the index is actually being used.