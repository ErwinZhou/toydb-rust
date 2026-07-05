# Learning Path: Rebuilding toyDB from Scratch

Goal: learn Rust and database internals by rebuilding toyDB stage-by-stage on a fresh branch,
using the finished `main` branch only as a reference to check work against — never to copy from
before attempting a stage.

Setup: `main` is source of truth. Reference copy checked out via `git worktree` (see
[worktree-notes.md](worktree-notes.md)) so it can be browsed without touching the build branch.

How to use this file: check a box when a stage's milestone works. Add dated notes under each
stage as you go — surprises, bugs that taught you something, Rust concepts that clicked. This is
the living log, not just a checklist.

---

## Stage 0 — Rust fundamentals refresher

Prerequisite concepts before Stage 1: ownership & borrowing, `Result`/`?`, custom error enums,
traits & generics, `Box<dyn Trait>`, iterators (implementing `Iterator`, not just consuming it).
`Arc`/`Mutex`/channels can wait until Stage 9 (Raft).

- [ ] Comfortable with ownership/borrowing and `Result`/`?`
- [ ] Comfortable with traits, generics, and `Box<dyn Trait>`
- [ ] Comfortable implementing `Iterator` by hand

Notes:

---

## Stage 1 — Encoding

Files to build: `encoding/keycode.rs`, `encoding/bincode.rs`
Reference: `src/encoding/*.rs`, [docs/architecture/encoding.md](../architecture/encoding.md)

Order-preserving byte encoding for keys (keycode) and a value/message encoding (bincode wrapper).
No dependencies on any other component — smallest possible first exercise.

- [ ] Milestone: encode two values, byte-compare their encodings, confirm ordering matches value
      ordering (round-trip tests pass)

Notes:

---

## Stage 2 — Storage engine

Files to build: `storage/engine.rs`, `storage/memory.rs`, then `storage/bitcask.rs`
Reference: `src/storage/{engine,memory,bitcask}.rs`, [docs/architecture/storage.md](../architecture/storage.md)

Define an `Engine` trait (get/set/delete/scan), implement over `BTreeMap` first, then BitCask
(append-only log + in-memory index).

- [ ] Milestone: same test suite passes against both `Memory` and `BitCask` engines via the trait

Notes:

---

## Stage 3 — MVCC

Files to build: `storage/mvcc.rs`
Reference: `src/storage/mvcc.rs`, [docs/architecture/mvcc.md](../architecture/mvcc.md)

Transaction layer over any `Engine`, providing snapshot isolation via version chains.

- [ ] Milestone: two concurrent transactions don't see each other's uncommitted writes

Notes:

---

## Stage 4 — SQL types

Files to build: `sql/types/{value,row,schema}.rs`
Reference: `src/sql/types/*.rs`, [docs/architecture/sql-data.md](../architecture/sql-data.md)

Data modeling only: `Value` enum, `Row`, `Schema`/`Column`.

- [ ] Milestone: types compile and cover the basic SQL data types you plan to support

Notes:

---

## Stage 5 — Parser

Files to build: `sql/parser/{lexer,parser,ast}.rs`
Reference: `src/sql/parser/*.rs`, [docs/architecture/sql-parser.md](../architecture/sql-parser.md)

Recursive-descent parser, SQL text → AST.

- [ ] Milestone: parse `SELECT * FROM t WHERE x = 1` into an AST you can print/debug

Notes:

---

## Stage 6 — Planner

Files to build: `sql/planner/{plan,planner}.rs`
Reference: `src/sql/planner/*.rs`, [docs/architecture/sql-planner.md](../architecture/sql-planner.md)

AST → logical plan tree, using a `Catalog` trait for schema lookup. Skip optimization for now.

- [ ] Milestone: build a plan for a simple `SELECT ... WHERE ...` and print it

Notes:

---

## Stage 7 — Executor

Files to build: `sql/execution/*.rs`
Reference: `src/sql/execution/*.rs`, [docs/architecture/sql-execution.md](../architecture/sql-execution.md)

Iterator-based row streaming over plan nodes, bottoming out in MVCC/engine scans.

### 🎯 Big milestone — working single-node SQL engine

Wire parser → planner → executor → MVCC → memory engine (`sql/engine/local.rs`). A real, if
non-distributed, SQL database. Good place to pause before Raft — roughly the halfway point.

- [ ] Milestone: `CREATE TABLE`, `INSERT`, `SELECT` all work end-to-end against the in-memory engine

Notes:

---

## Stage 8 — Optimizer (optional polish)

Files to build: `sql/planner/optimizer.rs`
Reference: `src/sql/planner/optimizer.rs`, [docs/architecture/sql-optimizer.md](../architecture/sql-optimizer.md)

Filter pushdown, secondary index lookups, hash joins, constant folding. Can be deferred
indefinitely — not required for the rest of the roadmap.

- [ ] Milestone: `EXPLAIN` shows a filter pushed below a join

Notes:

---

## Stage 9 — Raft

Files to build: `raft/{log,node,message,state}.rs`
Reference: `src/raft/*.rs`, [docs/architecture/raft.md](../architecture/raft.md)

The hard part. Suggested sub-order: log storage → state machine trait → node roles/election →
log replication → client request routing. Test with a simulated (non-networked) harness first,
like the real `raft/testscripts`. Introduces `crossbeam` channels for message-passing concurrency.

- [ ] Sub-milestone: log storage works in isolation
- [ ] Sub-milestone: a 3-node simulated cluster elects a leader
- [ ] Sub-milestone: writes replicate to a majority and commit
- [ ] Milestone: simulated cluster tolerates a minority-node failure and keeps serving

Notes:

---

## Stage 10 — Distributed SQL engine

Files to build: `sql/engine/raft.rs`
Reference: `src/sql/engine/raft.rs`, [docs/architecture/sql-raft.md](../architecture/sql-raft.md)

Route SQL reads/writes through Raft instead of directly through local MVCC.

- [ ] Milestone: SQL statements against a simulated Raft cluster give the same results as Stage 7's
      local engine

Notes:

---

## Stage 11 — Server & client

Files to build: `server.rs`, `client.rs`, `bin/toydb.rs`, `bin/toysql.rs`
Reference: `src/{server,client}.rs`, `src/bin/*.rs`, [docs/architecture/server.md](../architecture/server.md), [docs/architecture/client.md](../architecture/client.md)

Real TCP networking, message serialization over the wire, a `rustyline`-based REPL.

- [ ] Milestone: `./cluster/run.sh`-style multi-node cluster of your own binary, connect with your
      own CLI client, run a SQL session end-to-end over the network

Notes:

---

## After each stage

1. Get it working and tested on your own first.
2. Diff your module against the same file in the `main` reference worktree.
3. Read that stage's page in `docs/architecture/` as a check, not a spec.
4. Log what surprised you in the Notes section above.
