Absolutely—here’s a tight, exam-style prep pack for the **Microsoft SQL Server** section (15 Q in \~10 min). It’s focused on what’s most likely to be tested, with crisp answers you can memorize fast.

---

# 🎯 What they’ll likely test

- **Schema & Keys:** PK/FK/unique/check; clustered vs nonclustered; composite index order; included columns; filtered & covering indexes.
- **Querying:** joins, set ops, window functions, GROUP BY/HAVING, NULL semantics, SARGability.
- **Transactions & Concurrency:** ACID, isolation levels (RC, RCSI, Snapshot, Repeatable Read, Serializable), locks, blocking, deadlocks.
- **Performance:** execution plans, statistics, parameter sniffing, temp table vs table variable, NOLOCK risks.
- **T-SQL basics:** TRY…CATCH, XACT_ABORT, MERGE pitfalls, paging, TOP.
- **Data types:** `datetime2` vs `datetime`, `varchar` vs `nvarchar`, `uniqueidentifier`, `rowversion`.
- **Admin essentials:** recovery models, backups, log growth, tempdb basics.
- **Security:** logins vs users, least privilege, SQL injection avoidance (parameters).

---

# ⚡ Rapid-fire Q\&A (exam style)

1. **Clustered vs nonclustered index?**
   Clustered = table’s physical order (one per table). Nonclustered = separate structure with key + pointer (RID/clustered key).

2. **Best column order for composite index?**
   Most selective & most-filtered/equality columns **first**; supports **leftmost prefix** lookups. Add wide, frequently-selected columns to **INCLUDE** for covering.

3. **SARGable predicate?**
   Search can use an index **Seek**. Avoid wrapping indexed column in functions (`WHERE LOWER(col)=…`), leading `%` in LIKE, mismatched types.

4. **Filtered index use case?**
   Sparse data, e.g., `WHERE IsDeleted = 0` or `WHERE Status='Open'`, to shrink index and speed common queries.

5. **COUNT(\*) vs COUNT(col)?**
   `COUNT(*)` counts rows. `COUNT(col)` ignores `NULL`s.

6. **INNER vs LEFT join (NULLs)?**
   INNER: only matches. LEFT: preserves left rows; unmatched right columns are `NULL`.

7. **Window functions (ROW_NUMBER/RANK/LAG/LEAD)?**
   Use `OVER (PARTITION BY … ORDER BY …)` for de-dupe, pagination, “previous/next row” without self-joins.

8. **GROUP BY vs HAVING?**
   `WHERE` filters **before** grouping. `HAVING` filters **after** aggregation.

9. **TRY…CATCH + XACT_ABORT?**
   Wrap transactions in `TRY…CATCH`; set `SET XACT_ABORT ON` so severe errors auto-rollback.

10. **Isolation levels quick map:**

- **RC:** default; nonrepeatable/phantoms possible.
- **RCSI:** readers use row versioning (tempdb); fewer reader/writer blocks.
- **Snapshot:** per-transaction snapshot; writers still block writers.
- **Repeatable/Serializable:** stronger; more blocking.

11. **NOLOCK (READ UNCOMMITTED) risk?**
    Dirty reads, missing/duplicated rows, “ghost” reads. Don’t use for correctness-critical queries.

12. **Deadlock—what to say?**
    Order object access consistently, keep transactions short, right indexes (to lock fewer rows), retry deadlock victim.

13. **Parameter sniffing—symptom & fix?**
    Plan good for one parameter, bad for others. Fix with **`OPTION (RECOMPILE)`**, local variables, or “optimize for”/plan guides; best = shape & stats.

14. **Temp table vs table variable?**
    `#temp` tables have **stats** (better plans), can be indexed; table variables **don’t** (until recent versions, still limited) → prefer `#temp` for medium/large sets.

15. **Paging?**
    `ORDER BY … OFFSET @skip ROWS FETCH NEXT @take ROWS ONLY` (ORDER BY is required for OFFSET).

16. **`datetime2` vs `datetime`?**
    `datetime2` = more precision, wider range, **prefer it**.

17. **`varchar` vs `nvarchar`?**
    `nvarchar` (UTF-16) = multilingual; uses 2 bytes/char. Don’t use `NVARCHAR` if you don’t need it.

18. **GUID PK pros/cons?**
    Easy uniqueness, but random GUIDs fragment clustered index. Use **`NEWSEQUENTIALID()`** or make GUID the **nonclustered** PK and cluster on an INT.

19. **`rowversion` use?**
    Concurrency token for optimistic concurrency; not a datetime.

20. **Execution plan hotspots?**
    Key lookups, scans on big tables, hash matches, sorts, spills to tempdb. Fix with covering indexes, better predicates, memory grants.

21. **Statistics—why they matter?**
    Cardinality estimates → plan quality. Keep **AUTO_UPDATE_STATISTICS ON**; consider manual updates for skew.

22. **MERGE pitfalls?**
    Known bugs & race conditions; if used, lock properly and include semicolons. Many teams prefer separate `UPDATE`/`INSERT`.

23. **Unique vs PK constraint?**
    Both enforce uniqueness; PK also disallows NULL and defines default clustered key (unless specified).

24. **Check constraint scope?**
    Validates per-row using deterministic expressions; cannot reference another table.

25. **Recovery models:**
    **FULL** (point-in-time, needs log backups), **SIMPLE** (no log backups; truncates log at checkpoints), **BULK_LOGGED** (min-logged bulk ops).

26. **Backup types:**
    Full, Diff, Log; restore chain allows point-in-time (FULL model).

27. **Security basics:**
    Create **login** (server) → **user** (db) → grant to **roles**. Avoid `sa`. Use parameterized queries to prevent injection.

28. **Apply operators:**
    `CROSS APPLY` to invoke TVF per row; great for top-N-per-group or JSON shredding.

29. **JSON in SQL Server:**
    `OPENJSON` to parse; `FOR JSON` to emit. Index persisted computed columns if querying JSON properties.

30. **Foreign keys & cascades:**
    `ON DELETE/UPDATE CASCADE` can be helpful but risky; be deliberate on large graphs to avoid surprise mass deletes.

---

# 🧪 Micro-snippets to memorize

**Covering index with INCLUDE**

```sql
CREATE INDEX IX_Order_Customer_Date
ON dbo.[Order](CustomerId, OrderDate)
INCLUDE (TotalAmount, Status);
```

**Filtered index**

```sql
CREATE INDEX IX_Tickets_Open
ON dbo.Tickets(Status)
WHERE Status = 'Open';
```

**ROW_NUMBER de-dupe (latest per key)**

```sql
WITH x AS (
  SELECT *, ROW_NUMBER() OVER(PARTITION BY CustomerId ORDER BY UpdatedAt DESC) AS rn
  FROM CustomerAddresses
)
SELECT * FROM x WHERE rn = 1;
```

**TRY…CATCH transaction**

```sql
SET XACT_ABORT ON;
BEGIN TRY
  BEGIN TRAN;
  -- work
  COMMIT;
END TRY
BEGIN CATCH
  IF @@TRANCOUNT > 0 ROLLBACK;
  THROW;
END CATCH;
```

**Snapshot / RCSI enable**

```sql
ALTER DATABASE MyDb SET ALLOW_SNAPSHOT_ISOLATION ON;
ALTER DATABASE MyDb SET READ_COMMITTED_SNAPSHOT ON;
```

**Parameter-safe search (SARGable)**

```sql
-- Bad: WHERE LOWER(Email) = LOWER(@email)
-- Good:
WHERE Email = @email COLLATE Latin1_General_CI_AS
```

**OFFSET/FETCH paging**

```sql
SELECT cols
FROM dbo.Items
ORDER BY CreatedAt DESC
OFFSET @Skip ROWS FETCH NEXT @Take ROWS ONLY;
```

---

# ❗ Common traps (they love these)

- Using functions on indexed columns in `WHERE` → kills seeks.
- Two body-like inputs in T-SQL? (e.g., misuse of variables breaking plan reuse.)
- `NOLOCK` assumed “just faster” → dirty/phantom reads.
- Table variables for big data → bad estimates → slow.
- Random GUID as clustered PK → fragmentation → slow inserts.
- Missing `ORDER BY` with `TOP` → nondeterministic results.
- Forgetting `WHERE` in UPDATE/DELETE examples 🤦—or cascade effects.

---

# ✅ Pocket checklist (pre-exam glance)

- Index craft: **(EQ filters first)**, INCLUDE for covering, **filtered** for sparse, avoid function-wrapped predicates.
- Concurrency: know **RC vs RCSI vs Snapshot**, deadlock basics, `rowversion`.
- Plans: watch for **scans/lookups/spills**, keep **stats** fresh, beware **parameter sniffing**.
- T-SQL hygiene: **TRY…CATCH + XACT_ABORT**, careful with MERGE, prefer `#temp` over table variables for sets.
- Types: prefer **`datetime2`**, right `varchar/nvarchar`, cautious with GUID clustering.
- Admin: recovery models & backups.

If you want, I can turn this into a **15-question timed mini-mock** for SQL Server right now to simulate the assessment pace.
