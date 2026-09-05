# Data & Storage

Questions about SQL, schemas, indexing, partitioning, migrations, and analytical data systems.

## Atlassian

### 5. SQL / Data Manipulation

**1. Write a query to identify all products that have had at least two price changes within a rolling 90-day window.**
Model answer:
```sql
SELECT product_id
FROM (
  SELECT product_id, changed_at,
         COUNT(*) OVER (
           PARTITION BY product_id
           ORDER BY changed_at
           RANGE BETWEEN INTERVAL '90 days' PRECEDING AND CURRENT ROW
         ) AS changes_in_window
  FROM price_changes
) t
WHERE changes_in_window >= 2
GROUP BY product_id;
```
I'd flag that "rolling" needs a precise definition (any 90-day window ending anywhere, vs. the last 90 days from today) since that changes the query significantly.

**2. How would you design a schema to efficiently support both transactional writes and analytical rollups at scale?**
Model answer: I'd keep the transactional (OLTP) schema normalized and optimized for write correctness and point lookups, and stream changes (via CDC) into a separate analytical store (columnar warehouse) optimized for scans and aggregation. Trying to serve both patterns from one schema usually compromises both — wide denormalized tables hurt transactional write performance, and normalized tables hurt analytical scan performance.

**3. Explain your approach to indexing strategy for a table with high write throughput and frequent range queries.**
Model answer: Every index adds write cost, so I'd index deliberately for the specific range queries that matter, likely a composite index ordered to match the most common filter + range pattern, rather than indexing every column defensively. For very high write throughput, I'd also consider whether some indexes can be maintained asynchronously (e.g., in a secondary read-optimized store) rather than synchronously on every write.

**4. How would you detect and prevent data duplication in a system fed by multiple upstream event sources?**
Model answer: Prevention is better than detection — I'd require idempotency keys or natural dedup keys on ingestion, and use upsert semantics keyed on that identifier rather than blind inserts. For detection after the fact, I'd run periodic reconciliation jobs comparing counts/hashes against expected source-of-truth totals, since silent duplication often isn't caught until a downstream report looks wrong.

**5. Write a query approach to compute a sliding-window moving average of active users over the last 7 days.**
Model answer:
```sql
SELECT day,
       AVG(daily_active_users) OVER (
         ORDER BY day
         ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
       ) AS moving_avg_7d
FROM daily_active_user_counts
ORDER BY day;
```
At real scale, I'd precompute daily active user counts incrementally rather than recomputing distinct-user counts from raw event data on every query.

**6. How do you decide between denormalization and normalization when designing a schema for a high-scale reporting system?**
Model answer: I lean denormalized for reporting/analytical workloads where read performance and query simplicity matter more than storage cost or update anomalies, since reporting data is typically append-only or batch-updated rather than subject to frequent in-place edits. I keep the source-of-truth transactional data normalized and treat the denormalized reporting schema as a derived, rebuildable artifact.

**7. What's your strategy for schema migrations on a table with billions of rows and zero downtime tolerance?**
Model answer: I'd avoid any migration that locks the table or rewrites it in place; instead, add new columns as nullable, backfill asynchronously in batches with rate limiting to avoid replication lag or lock contention, and only enforce constraints (NOT NULL, etc.) once backfill is verified complete. For structural changes, I'd use a shadow-table/dual-write approach and cut over once validated, similar to the general strangler-migration pattern.

**8. How would you design a data model to support hierarchical permissions queries efficiently (e.g., "can user X access resource Y")?**
Model answer: I'd use a closure table or materialized path pattern to avoid recursive graph traversal on every permission check, so "is Y a descendant of any resource X has access to" becomes a simple indexed lookup rather than a recursive query. I'd pair this with the effective-permissions cache described earlier for the hottest paths.

**9. Explain how you'd approach data partitioning strategy for a multi-tenant system with wildly uneven tenant sizes.**
Model answer: Naive hash-based tenant partitioning breaks down when one tenant is 1000x larger than another — that tenant's shard becomes a hotspot. I'd use a hybrid: most tenants share pooled partitions via hash/range partitioning, but oversized tenants get dedicated partitions (or their own database), decided by a monitored size/traffic threshold rather than statically at onboarding.

**10. As a Principal Engineer, how do you evaluate whether a data problem needs a new datastore vs. better modeling in the existing one?**
Model answer: I push teams to articulate the specific failure mode of the current datastore (a query pattern it can't serve efficiently, a scale ceiling being approached, a consistency model mismatch) before considering a new one — "this would be easier in X" isn't sufficient justification given the operational cost of running another datastore. If better indexing, partitioning, or a read replica solves it, that's almost always cheaper than introducing new infrastructure with its own on-call burden and expertise requirement.

