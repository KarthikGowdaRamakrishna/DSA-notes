# Topic 1 — Arrays & Hashing
*Python first · Java equivalents · Rust follow-up*

---

## 1. The Mental Model

An **array** is contiguous memory: index → address is one multiplication, so reads are O(1) and the CPU prefetcher loves sequential scans (this is why arrays beat linked lists in practice even when big-O ties — you know this from cache lines).

A **hash map** trades memory for time: it turns "search" (O(n) scan) into "lookup" (O(1) average) by computing where a key *should* live. Almost every problem in this topic is one idea:

> **Replace a nested loop (O(n²) "for each element, search for something") with a single pass + hash lookup (O(n)).**

When you see a brute force with an inner search — ask *"what am I searching for, and can I have seen it already?"* If yes: hash map.

---

## 2. Core Operations & Complexity

| Operation | Array / List | Hash Map | Hash Set |
|---|---|---|---|
| Access by index/key | O(1) | O(1) avg | O(1) avg |
| Search by value | O(n) | O(1) avg (by key) | O(1) avg |
| Insert at end | O(1) amortized | O(1) avg | O(1) avg |
| Insert at front/middle | O(n) | — | — |
| Delete | O(n) (shift) | O(1) avg | O(1) avg |
| Sort | O(n log n) | — | — |

Gotchas worth saying aloud in interviews: hash worst case is O(n) (collisions); dict keys must be hashable/immutable (Python: tuple yes, list no); iteration order — Python dicts preserve insertion order (3.7+), Java HashMap does not (use LinkedHashMap), Rust HashMap does not.

---

## 3. Syntax — Python ↔ Java ↔ Rust

### Arrays / dynamic lists

| Task | Python | Java | Rust |
|---|---|---|---|
| Create | `a = [1, 2, 3]` | `List<Integer> a = new ArrayList<>(List.of(1,2,3));` | `let mut a = vec![1, 2, 3];` |
| Append | `a.append(4)` | `a.add(4)` | `a.push(4);` |
| Index | `a[0]`, `a[-1]` | `a.get(0)`, `a.get(a.size()-1)` | `a[0]`, `a[a.len()-1]` or `a.last()` |
| Slice | `a[1:3]` | `a.subList(1, 3)` | `&a[1..3]` |
| Length | `len(a)` | `a.size()` | `a.len()` |
| Sort in place | `a.sort()` | `Collections.sort(a)` | `a.sort();` |
| Sort by key | `a.sort(key=lambda x: x[1])` | `a.sort(Comparator.comparingInt(x -> x[1]))` | `a.sort_by_key(\|x\| x.1);` |
| Reverse | `a[::-1]` or `a.reverse()` | `Collections.reverse(a)` | `a.reverse();` |
| Sum / max | `sum(a)`, `max(a)` | `a.stream().mapToInt(i->i).sum()` | `a.iter().sum::<i32>()`, `a.iter().max()` |
| Iterate w/ index | `for i, x in enumerate(a):` | `for (int i = 0; i < a.size(); i++)` | `for (i, x) in a.iter().enumerate()` |
| List of zeros, size n | `[0] * n` | `new int[n]` | `vec![0; n]` |
| 2-D grid | `[[0]*c for _ in range(r)]` | `new int[r][c]` | `vec![vec![0; c]; r]` |
| Comprehension / map | `[x*x for x in a]` | `a.stream().map(x->x*x).toList()` | `a.iter().map(\|x\| x*x).collect::<Vec<_>>()` |

⚠ Python trap: `[[0]*c] * r` aliases rows — always use the comprehension form.
⚠ Rust: indexing out of bounds panics; `a.get(i)` returns `Option<&T>` for safe access.

### Hash maps

| Task | Python | Java | Rust |
|---|---|---|---|
| Create | `d = {}` | `Map<String,Integer> d = new HashMap<>();` | `let mut d: HashMap<String,i32> = HashMap::new();` |
| Insert / update | `d[k] = v` | `d.put(k, v)` | `d.insert(k, v);` |
| Get w/ default | `d.get(k, 0)` | `d.getOrDefault(k, 0)` | `*d.get(&k).unwrap_or(&0)` |
| Contains key | `k in d` | `d.containsKey(k)` | `d.contains_key(&k)` |
| Count idiom | `d[k] = d.get(k, 0) + 1` | `d.merge(k, 1, Integer::sum)` | `*d.entry(k).or_insert(0) += 1;` |
| Delete | `del d[k]` / `d.pop(k)` | `d.remove(k)` | `d.remove(&k);` |
| Iterate | `for k, v in d.items():` | `for (var e : d.entrySet())` | `for (k, v) in &d` |
| Default-value map | `defaultdict(list)` | `d.computeIfAbsent(k, x -> new ArrayList<>()).add(v)` | `d.entry(k).or_insert_with(Vec::new).push(v);` |
| Frequency counter | `Counter(a)` | *(no direct; use merge loop)* | *(entry loop above)* |

`collections.Counter` is your Python superpower: `Counter(s) == Counter(t)` is an anagram check in one line; `.most_common(k)` gives top-k.

### Hash sets

| Task | Python | Java | Rust |
|---|---|---|---|
| Create from list | `s = set(a)` | `Set<Integer> s = new HashSet<>(a);` | `let s: HashSet<_> = a.iter().collect();` |
| Add / contains | `s.add(x)` / `x in s` | `s.add(x)` / `s.contains(x)` | `s.insert(x);` / `s.contains(&x)` |
| Dedup check | `len(set(a)) < len(a)` | `s.size() < a.size()` | `s.len() < a.len()` |

### Strings (immutable in all three)

| Task | Python | Java | Rust |
|---|---|---|---|
| Build efficiently | `"".join(parts)` | `StringBuilder sb; sb.append(...)` | `String::new()` + `push_str` |
| Sort chars (canonical key) | `''.join(sorted(s))` | `char[] c = s.toCharArray(); Arrays.sort(c); new String(c)` | `let mut c: Vec<char> = s.chars().collect(); c.sort();` |
| char → index | `ord(ch) - ord('a')` | `ch - 'a'` | `(ch as u8 - b'a') as usize` |
| Split / join | `s.split(",")` / `",".join(a)` | `s.split(",")` / `String.join(",", a)` | `s.split(',')` / `a.join(",")` |

⚠ Rust strings are UTF-8; you can't index `s[i]`. Use `s.chars()` or `s.as_bytes()` (fine for ASCII problems, and faster).

---

## 4. The Patterns

### Pattern A — Frequency counting
*Signal: "anagram", "count occurrences", "same characters/elements".*
Map each element → count; compare or query the map.

```python
def is_anagram(s: str, t: str) -> bool:
    return Counter(s) == Counter(t)          # O(n) time, O(k) space
```
Manual version (know it — interviewers may ban Counter): array of size 26, `+1` for s, `-1` for t, check all zeros.

### Pattern B — Complement lookup (seen-so-far)
*Signal: "find a pair that sums/matches", "have I seen this before?"*
One pass; before inserting the current element, ask the map if its *complement* is already there.

```python
def two_sum(nums, target):
    seen = {}                                 # value -> index
    for i, x in enumerate(nums):
        if target - x in seen:
            return [seen[target - x], i]
        seen[x] = i
```
The invariant to say aloud: *"`seen` holds everything to my left, so each pair is checked exactly once."*

### Pattern C — Canonical key grouping
*Signal: "group items that are equivalent under some transformation".*
Design a key such that equivalent items collide: sorted string, char-count tuple, normalized fraction (slope), tuple of diffs.

```python
def group_anagrams(strs):
    groups = defaultdict(list)
    for s in strs:
        key = tuple(sorted(s))                # or 26-tuple of counts → O(n·k)
        groups[key].append(s)
    return list(groups.values())
```
**Choosing the key IS the problem.** If you can state the key, the code is mechanical.

### Pattern D — Prefix / suffix accumulation
*Signal: "product/sum of everything except…", "range sums", no-division constraints.*
`answer[i] = prefix[i] · suffix[i]` — sweep left-to-right, then right-to-left.

```python
def product_except_self(nums):
    n = len(nums)
    res = [1] * n
    pre = 1
    for i in range(n):
        res[i] = pre
        pre *= nums[i]
    suf = 1
    for i in range(n - 1, -1, -1):
        res[i] *= suf
        suf *= nums[i]
    return res
```
Prefix sums (`pre[i+1] = pre[i] + a[i]`, range sum = `pre[j+1]-pre[i]`) reappear constantly in later topics — own this now.

### Pattern E — Index as hash (bucket trick)
*Signal: "top K frequent" in O(n), "longest consecutive", values bounded by n.*
When counts/values are bounded by n, an array indexed by count/value replaces sorting.

```python
def top_k_frequent(nums, k):
    count = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]   # index = frequency
    for x, c in count.items():
        buckets[c].append(x)
    res = []
    for c in range(len(buckets) - 1, 0, -1):
        res.extend(buckets[c])
        if len(res) >= k:
            return res[:k]
```

Longest Consecutive Sequence uses the set variant: only start counting from `x` when `x-1` is not in the set — that guard is what makes it O(n).

```python
def longest_consecutive(nums):
    s = set(nums)
    best = 0
    for x in s:
        if x - 1 not in s:                    # x starts a run
            y = x
            while y + 1 in s:
                y += 1
            best = max(best, y - x + 1)
    return best
```

### Pattern F — Design with delimiters (Encode/Decode Strings)
*Signal: serialize variable-length data unambiguously.*
Length-prefix framing: `"4#neet4#code"`. Delimiters alone fail because data may contain them — the same reason TCP messages need length prefixes or framing (say this in the interview; it's your systems background showing).

---

## 5. Drill Plan (Week 1)

Learn-then-drill, 25-min cap per problem, then the +3/+7/+21 re-solve schedule. You've likely "done" several — treat first attempt as a blind audit.

| Day | Problem | Pattern | Difficulty |
|---|---|---|---|
| 1 | Contains Duplicate | E/set | Easy |
| 1 | Valid Anagram | A | Easy |
| 2 | Two Sum | B | Easy |
| 2 | Group Anagrams | C | Medium |
| 3 | Top K Frequent Elements | E | Medium |
| 4 | Product of Array Except Self | D | Medium |
| 4 | Valid Sudoku | A/C (keys = row/col/box sets) | Medium |
| 5 | Encode and Decode Strings | F | Medium |
| 6 | Longest Consecutive Sequence | E | Medium |
| 7 | **Re-solve day:** all Day 1–4 problems blind, 15 min each | — | — |

Java rep (during +7): redo Two Sum and Group Anagrams in Java.
Rust rep (Week 5+): Two Sum and Top K Frequent in Rust — `entry().or_insert()` will click.

---

## 6. Checkpoint (pass before Week 2)

You're done with this topic when, cold, you can:

1. Solve Two Sum + Valid Anagram in under 10 min combined, bug-free first run.
2. State the canonical key for Group Anagrams two ways (sorted string vs 26-count tuple) and their complexities.
3. Explain why Longest Consecutive is O(n) despite the nested while loop.
4. Write the Counter idiom in all three languages from memory:
   - Python: `d[k] = d.get(k, 0) + 1`
   - Java: `d.merge(k, 1, Integer::sum)`
   - Rust: `*d.entry(k).or_insert(0) += 1;`

Fail any → redo that item tomorrow, then move on. Reps over perfection — but the checkpoint is the floor.

**Next topic:** Two Pointers (Week 2) — sorted-array structure replaces the hash map as the search-killer.
