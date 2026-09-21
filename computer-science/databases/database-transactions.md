# Database Transactions

A transaction groups several operations so they succeed or fail together.

## Example

Transferring money between two accounts:

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;   -- or ROLLBACK; to undo everything since BEGIN
```

If the server crashes after the first `UPDATE` but before `COMMIT`, the first change is not kept. Without the transaction you'd have money removed and never added.

## ACID

| Letter | Meaning |
|---|---|
| Atomicity | all or nothing |
| Consistency | constraints hold before and after (foreign keys, checks, uniqueness) |
| Isolation | concurrent transactions don't see each other's half-finished work (to a configurable degree) |
| Durability | once committed, it survives a crash |

## Isolation levels

Isolation is the part that has knobs. Lower levels allow more anomalies in return for less locking/coordination.

| Anomaly | What happens |
|---|---|
| Dirty read | you read another transaction's uncommitted data |
| Non-repeatable read | same row read twice in a transaction gives different values |
| Phantom read | same query returns different *sets* of rows |

Standard levels: Read Uncommitted, Read Committed, Repeatable Read, Serializable. Actual behavior differs per database. For example, PostgreSQL defaults to Read Committed and MySQL InnoDB defaults to Repeatable Read, and each implements the levels in its own way (MVCC, locking). Read the docs for the specific database.

## Common mistakes

- Not using a transaction for multi-step writes that must stay consistent.
- Check-then-act races: read a value, decide in application code, write later. Another transaction can change it in between. Use atomic updates (`UPDATE ... SET x = x - 1 WHERE x > 0`), locking reads (`SELECT ... FOR UPDATE`), or a stricter isolation level.
- Keeping transactions open while doing slow work (HTTP calls, user input). Holds locks, blocks others.
- Forgetting to `ROLLBACK` on error in application code, leaving a connection stuck in a transaction (connection pools make this nasty).
- Assuming a transaction covers external side effects. Sending an email or calling an API is not rolled back.
- Deadlocks: two transactions lock rows in opposite order. Databases detect them and abort one; the application should retry. Consistent lock ordering reduces them.

## Practical notes

- Autocommit: many clients commit each statement immediately unless you start a transaction.
- Most ORMs and drivers have a transaction helper; use it so commit/rollback is guaranteed even when errors are thrown.
- Keep transactions short.
- Serializable errors and deadlocks are expected in busy systems; code should be able to retry.

## When to use it

- Multiple writes that must be consistent together.
- Read-modify-write sequences.

## When not to use it

- Single-statement operations that are already atomic on their own.

## Remember

- BEGIN … COMMIT/ROLLBACK.
- Short transactions.
- Isolation level details are database-specific; look them up.

## Related

- [Database indexes](database-indexes.md)

## References

- PostgreSQL docs: Transaction Isolation
- MySQL docs: InnoDB Transaction Isolation Levels
