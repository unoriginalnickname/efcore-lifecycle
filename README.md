# EF Core — Full Lifecycle

> A comprehensive, single-page reference covering everything that happens from model definition to SQL on the wire in Entity Framework Core.

🔗 **Live site:** [efcore-lifecycle.vercel.app](https://efcore-lifecycle.vercel.app/)

---

## What This Is

A deep-dive reference guide through the complete lifecycle of EF Core — from design-time model building and migrations, through the query pipeline, change tracking, SaveChanges, relationships, concurrency, raw SQL, interceptors, connection management, and production performance tuning.

It's aimed at developers who already know the basics and want to understand *why* things work the way they do, not just *how* to use them.

---

## What's Covered

The guide walks through **12 numbered stages**, split into design-time and runtime phases:

### 🔧 Design-Time — runs once at development / deployment

| # | Stage | What it covers |
|---|-------|----------------|
| ① | **DbContext** | Class vs instance lifetime, `AddDbContext` registration, Scoped vs Singleton pitfalls, `OnConfiguring` for CLI/tests, MVC setup |
| ② | **Model** | Convention → annotation → Fluent API priority, `OnModelCreating`, global query filters, `IEntityTypeConfiguration<T>`, model compilation cache |
| ③ | **Migrations** | Code-first vs database-first scaffold, `migrations add/update/remove`, deployment strategies (MigrateAsync / SQL script / bundle), `__EFMigrationsHistory` |

### ⚡ Runtime — runs per operation in your application

| # | Stage | What it covers |
|---|-------|----------------|
| ④ | **Query Pipeline** | `IQueryable<T>` vs `IEnumerable<T>`, expression tree translation, query caching, projection with `Select()`, `AsNoTracking()`, `ToQueryString()` |
| ⑤ | **Change Tracker** | Entity state machine (Detached/Unchanged/Added/Modified/Deleted), snapshot diffing, identity map, `DetectChanges`, `AsNoTrackingWithIdentityResolution()` |
| ⑥ | **SaveChanges** | Internal pipeline (DetectChanges → batch → transaction → execute → snapshot refresh), `SavingChanges`/`SavedChanges` hooks, audit stamping, Unit of Work pattern |
| ⑦ | **Relationships** | Eager loading (`Include`/`ThenInclude`), explicit loading, lazy loading, N+1 problem, cartesian explosion, `AsSplitQuery()` |
| ⑧ | **Concurrency** | `[ConcurrencyToken]` vs `[Timestamp]`/`rowversion`, `DbUpdateConcurrencyException`, client-wins / server-wins / merge resolution strategies |
| ⑨ | **Raw SQL** | `FromSql()` (composable), `ExecuteUpdateAsync`/`ExecuteDeleteAsync` (bulk DML), `SqlQuery<T>()` for non-entity projections, injection safety |
| ⑩ | **Interceptors** | `IDbCommandInterceptor` (slow query logging), `ISaveChangesInterceptor` (audit, domain events), `IConnectionInterceptor`, `LogTo()` |
| ⑪ | **Connections** | ADO.NET connection pooling, implicit transactions, explicit `BeginTransactionAsync()`, sharing transactions across DbContexts, avoiding `TransactionScope` with async |
| ⑫ | **Performance** | Compiled queries (`EF.CompileAsyncQuery`), bulk operations, `AsSplitQuery()`, pre-compiled model (`dotnet ef dbcontext optimize`), query checklist |

---

## Key Concepts Explained

**DbContext class vs instance lifetime** — the class is compiled once and cached; the instance holds the Change Tracker and connection. Confusing these (especially registering as Singleton) is the root cause of most EF bugs.

**`IQueryable<T>` is not a result** — every `.Where()`, `.OrderBy()`, and `.Select()` builds an expression tree in memory. SQL fires only when you enumerate. Calling `.AsEnumerable()` mid-chain pulls the entire table into memory before filtering.

**The Change Tracker identity map** — loading the same entity twice in one scope returns the same C# instance. `FindAsync(5)` called twice hits the tracker on the second call with no database round-trip.

**Why lazy loading hides N+1 bugs** — accessing a navigation property inside a loop fires one SQL query per entity. 100 users → 101 queries. These only show up under production load, which is why lazy loading is disabled by default.

**`Down()` is not a production rollback strategy** — it's a local development convenience. Rolling back migrations on a live database with real data is dangerous. Always roll forward with a corrective migration.

**Why `SavingChanges` vs `SavedChanges` matters** — `SavingChanges` fires before the transaction commits (use for audit stamps, outbox messages that must be in the same transaction). `SavedChanges` fires after commit (use for domain events and cache invalidation that should only happen if the save succeeded).

**Cartesian explosion** — including two collection navigations with a JOIN multiplies rows multiplicatively. 100 users × 10 posts × 5 tags = 5,000 rows. `AsSplitQuery()` breaks this into separate SELECTs.

**`ExecuteUpdateAsync` vs tracked update** — updating 10,000 rows the tracked way allocates 10,000 entities with 10,000 snapshots, runs DetectChanges over all of them, and sends a large batch. `ExecuteUpdateAsync` sends a single `UPDATE … WHERE …` regardless of row count.

---

## Who It's For

- .NET developers who want a mental model of what EF Core actually does under the hood
- Developers debugging unexpected query behaviour, missing updates, or concurrency errors
- Engineers onboarding to a .NET codebase who want the "why" behind common EF patterns
- Anyone preparing for a .NET technical interview

---

## Related

Also see the companion reference: **[ASP.NET Core Full Request Lifecycle](https://aspnet-lifecycle.vercel.app/)** — covers the full HTTP request pipeline from TCP connection to JSON on the wire.

---

## Tech

- Static HTML/CSS — no build step, no framework
- Hosted on [Vercel](https://vercel.com)

---

## Contributing

Found something incorrect or missing? Open an issue or PR. The goal is accuracy — every stage should reflect current EF Core behaviour (EF Core 8+).

---

*Last updated: June 2026*
