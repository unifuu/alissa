---
title: "About Locks"
date: "2025-12-10"
summary: "📝 Quick notes on Locks..."
tags: ["backend", "database", "locks"]
---

## Pessimistic Lock

- Conflicts will definitely happen. I should lock first:
    - `FOR UPDATE` locks the row, other TXN block until the lock is released.
``` sql
BEGIN;
SELECT coins FROM players WHERE id = 39 FOR UPDATE;
UPDATE players SET coins = coins + 10 WHERE id = 39;
COMMIT;
```

- When to use:
    - High chance of conflicts
    - Strong consistency required
- Disadvantage:
    - Deadlocks possible

## Optimistic Lock

- Conflicts probably won’t happen. If they do, retry:
    - Common implementation: version or updated_at column.
``` sql
-- 1. Read
SELECT coins, version FROM players WHERE id = 39; -- if version = 5

-- 2. Update:
UPDATE players
SET coins = coins + 10, version = version + 1
WHERE id = 39 AND version = 5;

-- 3. If affected rows = 0, it means conflict, just retry.
SELECT coins, version FROM players WHERE id = 39; -- if version = 6
UPDATE players
SET coins = coins + 10, version = version + 1
WHERE id = 39 AND version = 6;
```

## Distributed Lock

- Multiple servers exist, but this code must be executed by only one of them.
    - Prevent duplicate reward claims across servers
    - Prevent double inventory deduction
    - Ensure a scheduled job only runs once
- Types of distributed locks:
    - Redis Lock (Most common)