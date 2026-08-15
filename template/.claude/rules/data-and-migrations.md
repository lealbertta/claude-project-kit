---
paths:
  - "<persistence-dir>/**/*"
  - "<schema-dir>/**/*"
  - "<fixtures-dir>/**/*"
---

# Data and Migration Rules

- Every persisted root carries a schema version.
- Persisted records use stable IDs and logical values. Never serialize live
  objects, framework handles, or object graphs.
- Write to a temporary location, validate, then atomically replace the active
  data, retaining the last known-good copy.
- Treat interruption (crash, kill, power loss, connection drop) as a normal event.
  Any committed state must be safe to persist at any moment.
- Every schema change requires a migration and at least one fixture test using a
  real file written by a released version.
- Unknown or unavailable records load as explicit placeholders that preserve the
  original bytes. Do not drop what you cannot interpret — re-serialize it intact.
- Migrations are forward-only, deterministic, and idempotent where practical.
- Never delete a migration that a released or shared build may still need.
- Widening a constraint is compatible; narrowing one is a migration. Lowering a
  limit that existing data already exceeds must not destroy that data — decide
  explicitly whether old data is grandfathered or converted, and write it down.
- Repair-on-load must be reported, not silent. A record repaired in memory and
  re-serialized unchanged must not claim a repair the next load will not see.
