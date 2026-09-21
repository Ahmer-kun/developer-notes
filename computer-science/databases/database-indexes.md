# Database Indexes

An index is a separate data structure that lets the database find rows without scanning the whole table. It's the same idea as a book's index.

## What it is

Most relational databases use B-tree indexes by default (PostgreSQL, MySQL/InnoDB). A B-tree keeps keys sorted, so it can do equality lookups, range queries, and ordered scans efficiently.

```sql
CREATE INDEX idx_users_email ON users (email);

-- can now use the index:
SELECT * FROM users WHERE email = 'a@example.com';
```

Primary keys and unique constraints get an index automatically in most databases. Foreign key columns are **not** automatically indexed in PostgreSQL (MySQL/InnoDB does index them). Check your database.

## Why it matters

Without a usable index, the database does a full table scan: read every row and check the condition. That's fine for small tables and bad for big ones.

## Checking whether it's used

```sql
EXPLAIN SELECT * FROM users WHERE email = 'a@example.com';
-- PostgreSQL: EXPLAIN ANALYZE also runs the query and shows real timings
```

Look for an index scan / seek versus a sequential/full scan. Output format differs by database.

## Composite indexes

```sql
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at);
```

Column order matters. This index helps with:

- `WHERE user_id = ?`
- `WHERE user_id = ? AND created_at > ?`
- `WHERE user_id = ? ORDER BY created_at`

It generally does **not** help a query that filters only on `created_at` (the leftmost-prefix rule). Put equality columns first, range columns later.

## Common mistakes

- Indexing everything. Every index costs disk space and slows down `INSERT`/`UPDATE`/`DELETE`, since it must be maintained.
- Wrapping the indexed column in a function, which prevents use of a plain index:

```sql
WHERE lower(email) = 'a@example.com'   -- plain index on email won't help
-- fix: an expression index on lower(email), where supported
```

- Leading wildcards: `LIKE '%abc'` can't use a B-tree efficiently. `LIKE 'abc%'` can (depending on collation and settings).
- Indexing low-selectivity columns (a boolean with 50/50 values). The planner may ignore it.
- Assuming the planner will use your index. With small tables or queries returning a large share of rows, a scan may genuinely be cheaper.
- Implicit type conversion in comparisons (comparing a text column to a number) can bypass the index.

## Practical notes

- Add indexes based on real slow queries and `EXPLAIN`, not guesses.
- Foreign keys used in joins are strong candidates.
- A unique index also enforces uniqueness, not only speed.
- Covering indexes (index contains all needed columns) can avoid touching the table; syntax and support vary.
- Other index types exist (hash, GIN, GiST, full-text). Learn them when a problem calls for them.

## When to use it

- Columns used often in `WHERE`, `JOIN`, `ORDER BY` on large tables.

## When not to use it

- Tiny tables, or write-heavy tables where the read benefit is small.

## Remember

- Index = faster reads, slower writes, more storage.
- Composite order matters.
- Verify with `EXPLAIN`.

## Related

- [Database transactions](database-transactions.md)

## References

- PostgreSQL docs: Indexes
- MySQL docs: How MySQL uses indexes
