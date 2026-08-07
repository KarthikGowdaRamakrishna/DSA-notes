# DSA Notes

Pattern-first notes for L5 SDE / SysDev interview prep. **Python first · Java equivalents · Rust follow-up.**

Not a problem list. Each topic teaches the *elimination argument* that makes a pattern work, then applies it to the problems that pattern owns. The goal is to be able to **regenerate** the code, not recall it.

---

## Topics

| # | Topic | Shapes / patterns | Status |
|---|---|---|---|
| 1 | [Arrays & Hashing](arrays_and_hashing/) | frequency count · complement lookup · canonical key · prefix/suffix · index-as-hash · delimiter design | ✅ |
| 2 | [Two Pointers](two_pointers/) | converging · read/write · fast-slow | ✅ |
| 3 | [Sliding Window](sliding_window/) | fixed size · dynamic longest · dynamic shortest | ✅ |
| 4 | Binary Search & Prefix Sums | vanilla · boundary / feasibility | ☐ |
| 5 | Linked Lists | reversal · merge · dummy-node rewiring | ☐ |
| 6 | Trees & Traversal | BFS levels · DFS recursion | ☐ |
| 7 | Graphs | BFS shortest path · DFS components · topo sort · union-find | ☐ |
| 8 | Backtracking | choose / explore / un-choose · pruning | ☐ |
| 9 | Heaps & Priority Queues | top-K · streaming median · Dijkstra | ☐ |
| 10 | Sorting | merge · quickselect · counting · custom comparators | ☐ |
| 11 | Dynamic Programming | 1-D · 2-D · knapsack · LIS | ☐ |
| 12 | Bit Manipulation | masks · XOR tricks · subsets | ☐ |

---

## Structure of each topic

Every topic file follows the same skeleton, so you always know where to look:

1. **The Mental Model** — the one idea, and what it replaces
2. **Core Operations & Complexity** — the table, plus gotchas worth saying aloud in interviews
3. **Syntax — Python ↔ Java ↔ Rust** — side-by-side, with ⚠ traps that change *correctness*, not just style
4. **The Shapes / Patterns** — each with a `*Signal:*` line and a template
5. **Worked Problems** — per problem: Approach → Algorithm → Code → Line notes → **Dry run table** → Complexity → Edge cases
6. **When it is NOT this pattern** — the boundary; knowing it is what makes matching fast
7. **Drill Plan** — one week, learn-then-drill, 25-min cap per problem
8. **Checkpoint** — things you must be able to *say* cold before moving on
9. **learndsa.org coverage** — which sections of the site each topic discharges

---

## How to use this

**Learn-then-drill.** Read the topic once. Then drill from the Drill Plan with a 25-minute cap per problem — if you blow the cap, read the worked solution, and put it back in the queue.

**Spaced re-solve: +3 / +7 / +21 days.** A problem isn't done when it passes; it's done when it still passes three weeks later, blind.

**Checkpoints are the floor.** Fail an item → redo it tomorrow, then move on. Reps over perfection.

**The dry-run tables are the point.** If a problem isn't clicking, don't re-read the prose — trace the table row by row against your own example. Every bug in these patterns is an off-by-one that a trace catches in thirty seconds.

---

## Conventions

- `⚠` marks a trap that changes **correctness**, not just style.
- `*Signal:*` lines are the trigger phrases that should make you reach for a pattern.
- Complexity is stated as `time / space`, space excluding the output.
- Python is the reference implementation. Java appears where the idiom differs. Rust appears in the syntax tables and where its ownership rules change the approach.

---

## Verification

Every Python and Java snippet in these notes was **executed**, not just written — deterministic tests plus randomized comparison against brute-force oracles where the correctness argument is subtle (greedy elimination, dedup, in-place partitioning).

Rust snippets are reasoned, not executed.
