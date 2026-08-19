# Topic 02 — Two Pointers
*Pattern first · Python primary · Java equivalent · Rust follow-up*

---

## 1. The one idea

A nested loop asks: *for every `i`, check every `j`.* O(n²), because every step **adds** work.

Two pointers asks: *given what I know about `l` and `r` right now, can I rule out an entire side?*

If yes, every step **deletes** work. O(n²) → O(n).

The pattern lives or dies on that question. If nothing can be ruled out, two pointers does not apply.

**Precondition:** an *ordering* — usually sorted, not always. What you actually need: a comparison at `(l, r)` must say something **definitive** about everything inward of one of them.

> Same monotonic-condition idea that powers binary search. Binary search halves the space per step; two pointers shaves it. Same family, different aggression.

**The sentence to say in an interview:** "Moving this pointer is safe because no better answer can involve it."

---

## 2. Core Operations & Complexity

### 2.1 What the pattern buys you

| Problem class | Brute force | Two pointers | What made it work |
|---|---|---|---|
| Pair summing to target, **sorted** | O(n²) / O(1) | **O(n) / O(1)** | sum is monotonic in each pointer |
| Pair summing to target, **unsorted** | O(n²) | O(n) / O(n) **hash map** | ← not two pointers (Topic 01) |
| Max container / max area | O(n²) | **O(n) / O(1)** | short wall is provably exhausted |
| Triple summing to 0 | O(n³) | **O(n²) / O(1)**\* | sort → fix one → converge |
| In-place filter / dedupe / partition | O(n) / O(n) | **O(n) / O(1)** | write cursor lags read cursor |
| Midpoint of linked list | 2 passes | **1 pass / O(1)** | fast covers 2× |
| Cycle detection | O(n) / O(n) set | **O(n) / O(1)** | gap shrinks by 1 per step |
| Merge / intersect two **sorted** arrays | O(n·m) | **O(n+m) / O(1)** | both inputs monotonic |

\* excluding the O(log n) recursion stack of the sort.

### 2.2 Underlying operation costs (what you're actually paying for)

| Operation | Cost | Note |
|---|---|---|
| Index read `a[i]` | O(1) | contiguous memory, prefetcher-friendly |
| Swap `a[i], a[j]` | O(1) | the workhorse of read/write + Dutch flag |
| Pointer step `l += 1` | O(1) | total moves across a run ≤ n per pointer |
| **Sort to enable the pattern** | **O(n log n)** | the tax — see below |
| Python slice `a[i:j]` | O(j−i) **copy** | silently turns O(1) loops into O(n²) |
| Java `subList` | O(1) **view** | not a copy — different from Python |
| Rust `&a[i..j]` | O(1) **borrow** | not a copy |
| Linked-list `node.next` | O(1) | but cache-hostile; no random access |

**The tax rule, memorise it:** if the input is *already* sorted, two pointers wins (O(n) / O(1)). If you have to sort it yourself, you pay O(n log n) — and a hash map at O(n) / O(n) usually beats it. Sortedness is a gift, not something you manufacture. Exception: when you need *all* results with dedup (3Sum), sorting pays for itself because it makes duplicates adjacent.

**Amortisation argument (say it out loud):** a `while` nested in a `for` looks quadratic. It isn't, when each pointer only moves forward: n right-moves + ≤ n left-moves = O(2n) = **O(n)**.

---

## 3. Syntax — Python ↔ Java ↔ Rust

### 3.1 Pointer mechanics

| Task | Python | Java | Rust |
|---|---|---|---|
| Init converging | `l, r = 0, len(a) - 1` | `int l = 0, r = a.length - 1;` | `let (mut l, mut r) = (0usize, a.len() - 1);` |
| Loop | `while l < r:` | `while (l < r) {` | `while l < r {` |
| Step both | `l, r = l + 1, r - 1` | `l++; r--;` | `l += 1; r -= 1;` |
| Swap | `a[i], a[j] = a[j], a[i]` | `int t=a[i]; a[i]=a[j]; a[j]=t;` | `a.swap(i, j);` |
| Midpoint (overflow-safe) | `(l + r) // 2` | `l + (r - l) / 2` | `l + (r - l) / 2` |
| Empty-input guard | `if not a: return 0` | `if (a.length == 0) return 0;` | `if a.is_empty() { return 0; }` |

⚠ **Rust `usize` trap:** `r = a.len() - 1` panics on an empty vec (underflow, not −1). Guard first, or use `i32` indices, or iterate with `.iter().rev()`.
⚠ **Java:** `l + (r - l) / 2`, never `(l + r) / 2` — the latter overflows on large arrays. Python has bignums so it doesn't matter there; say the Java form anyway, interviewers listen for it.

### 3.2 In-place array work (read/write cursor)

| Task | Python | Java | Rust |
|---|---|---|---|
| Overwrite-keep | `a[w] = a[r]; w += 1` | `a[w++] = a[r];` | `a[w] = a[r]; w += 1;` |
| Swap-keep | `a[w], a[r] = a[r], a[w]` | *(manual temp)* | `a.swap(w, r);` |
| Return new length | `return w` | `return w;` | `return w;` |
| Truncate to length | `del a[w:]` | *(n/a — return length)* | `a.truncate(w);` |
| Iterate with index | `for r in range(len(a)):` | `for (int r=0;r<a.length;r++)` | `for r in 0..a.len() {` |

⚠ **Rust borrow checker:** you cannot hold `&mut a[i]` and `&a[j]` at once. Use `a.swap(i, j)`, or `split_at_mut`, or copy the values out first (`let (x, y) = (a[i], a[j]);`). This is *the* reason two-pointer code feels harder in Rust.

### 3.3 Character work (palindrome-class problems)

| Task | Python | Java | Rust |
|---|---|---|---|
| Alphanumeric test | `s[i].isalnum()` | `Character.isLetterOrDigit(s.charAt(i))` | `s[i].is_ascii_alphanumeric()` |
| Lowercase | `s[i].lower()` | `Character.toLowerCase(c)` | `c.to_ascii_lowercase()` |
| Index a string | `s[i]` | `s.charAt(i)` | ❌ — use `s.as_bytes()[i]` |
| To index-able form | *(already is)* | `s.toCharArray()` | `let b = s.as_bytes();` |
| Reverse | `s[::-1]` (O(n) space) | `new StringBuilder(s).reverse()` | `s.chars().rev().collect::<String>()` |

⚠ **Rust strings are UTF-8** — no `s[i]`. For ASCII interview problems `s.as_bytes()` is correct *and* faster.
⚠ **Python:** `s[::-1] == s` is a correct palindrome check but O(n) **space**. Two pointers is O(1). Interviewers ask for the O(1) one.

### 3.4 Linked list (fast/slow)

| Task | Python | Java | Rust |
|---|---|---|---|
| Node field | `node.next` | `node.next` | `node.next` (`Option<Box<Node>>`) |
| Two-step guard | `while fast and fast.next:` | `while (fast != null && fast.next != null)` | `while let Some(f) = fast { ... }` |
| Advance | `slow, fast = slow.next, fast.next.next` | `slow=slow.next; fast=fast.next.next;` | *(needs `Rc<RefCell<>>`)* |
| Identity compare | `slow is fast` | `slow == fast` (reference eq) | `Rc::ptr_eq(&s, &f)` |

⚠ **Python:** use `is`, not `==`. `==` may invoke `__eq__` and compare *values*, which is wrong for cycle detection.
⚠ **Java:** `==` on objects *is* reference equality — the correct thing here. Do not call `.equals()`.
⚠ **Rust:** cyclic linked lists fight ownership. Don't do these in Rust for interview prep; do them in Python/Java and learn Rust on the array problems.

### 3.5 Two inputs (merge / intersect)

| Task | Python | Java | Rust |
|---|---|---|---|
| Dual bound | `while i < len(a) and j < len(b):` | `while (i<a.length && j<b.length)` | `while i < a.len() && j < b.len() {` |
| Drain rest | `out.extend(a[i:])` | `while(i<a.length) out.add(a[i++]);` | `out.extend_from_slice(&a[i..]);` |
| Append | `out.append(x)` | `out.add(x)` | `out.push(x);` |

---

## 4. The three shapes

### Shape A — Converging (opposite ends)

```
l →                    ← r
[ _ _ _ _ _ _ _ _ _ _ _ ]
```

`l = 0`, `r = n−1`, move inward until they meet.

**Move rule — always the same question:**
> Which pointer, if moved, could *possibly* improve things? Move that one. The other is already as good as it gets on its side.

Everything else is problem-specific dressing.

**Triggers:** sorted + find a pair/triple hitting a target · mirrored comparison (palindrome) · brute force would check all pairs.
**Cost:** O(n) time, O(1) space. One of the few patterns that improves time *and* space.

---

### Shape B1 — Same direction: read / write cursor

```
        w
[ k k k ? ? ? ? ? ]
          r
```

`w` = where the next **kept** element goes. `r` scans everything. In-place filtering, no extra array.

**Invariant:** `a[0 .. w-1]` is the finished answer; `a[w .. r-1]` is garbage you're allowed to overwrite; `a[r ..]` is unexamined.

**Triggers:** remove / dedupe / partition **in place**, order preserved.

---

### Shape B2 — Same direction: fast / slow at 2×

`fast` moves 2, `slow` moves 1.

**Why you get the midpoint free:** when `fast` has covered `n`, `slow` has covered `n/2`. When `fast` falls off the end, `slow` stands on the middle. No length pass.

**Why you get cycle detection free:** once both are inside a loop of length `L`, `fast` gains exactly 1 net step per tick, so the gap shrinks by 1 every step. A quantity that decreases by 1 per step and lives in `[0, L)` **must** hit 0. Collision is forced. No cycle → `fast` hits `None` first.

That is the entire proof of Floyd's algorithm. Be able to say it out loud.

---

### Shape C — Three pointers (partition into 3 regions)

Dutch National Flag. `low` = end of region-0, `mid` = scanner, `high` = start of region-2. Covered as problem 7.

---

## 5. Templates

### Converging

```python
def converge(arr):
    l, r = 0, len(arr) - 1
    while l < r:
        if <condition met at (l, r)>:
            return ...
        elif <need bigger>:
            l += 1          # only the left can help
        else:
            r -= 1          # only the right can help
    return <not found>
```

```java
int l = 0, r = arr.length - 1;
while (l < r) {
    if      (/* condition   */) return /* ... */;
    else if (/* need bigger */) l++;
    else                        r--;
}
```

```rust
let (mut l, mut r) = (0usize, arr.len() - 1);   // guard empty first!
while l < r {
    if /* condition */      { return /* ... */; }
    else if /* need bigger */ { l += 1; }
    else                      { r -= 1; }
}
```

### Read / write

```python
def filter_in_place(arr, keep):
    w = 0
    for r in range(len(arr)):
        if keep(arr[r]):
            arr[w] = arr[r]
            w += 1
    return w                 # new logical length
```

```java
int w = 0;
for (int r = 0; r < arr.length; r++)
    if (keep(arr[r])) arr[w++] = arr[r];
return w;
```

```rust
let mut w = 0;
for r in 0..arr.len() {
    if keep(arr[r]) { arr[w] = arr[r]; w += 1; }
}
arr.truncate(w);
```

### Fast / slow

```python
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
# slow == middle (2nd middle if even length)
```

```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;
    fast = fast.next.next;
}
```

---

## 6. Worked problems

Every problem below runs the full solve format: **Understanding → Approach → Algorithm → Code → Line-by-line → Dry run → Complexity → Edge cases.** The dry runs are the point — read them, don't skim them.

---

### 6.1 Valid Palindrome (LC 125) — converging

#### Problem Understanding
Given a string, decide if it reads the same forwards and backwards **after** discarding every non-alphanumeric character and ignoring case. `"A man, a plan, a canal: Panama"` → `true`.

#### Approach
Mirror comparison. Instead of building a cleaned copy (O(n) extra space), keep two pointers at the ends and **skip junk in place** before each comparison. Any mismatch at a mirrored pair is a proof of failure — return immediately.

#### Algorithm
1. `l = 0`, `r = n − 1`.
2. While `l < r`: advance `l` past non-alphanumerics; retreat `r` past non-alphanumerics.
3. Compare lowercased `s[l]` vs `s[r]`. Mismatch → `False`.
4. Step both inward.
5. Survived the loop → `True`.

#### Code
```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        l, r = 0, len(s) - 1
        while l < r:
            while l < r and not s[l].isalnum():
                l += 1
            while l < r and not s[r].isalnum():
                r -= 1
            if s[l].lower() != s[r].lower():
                return False
            l, r = l + 1, r - 1
        return True
```

#### Line-by-Line Explanation
- `l, r = 0, len(s) - 1` — endpoints of the still-unverified region.
- `while l < r:` — stop when they meet or cross; a single middle char is trivially a palindrome, no need to check it.
- `while l < r and not s[l].isalnum(): l += 1` — skip junk on the left. **The `l < r` is load-bearing** — without it a string of pure punctuation walks off the end.
- `while l < r and not s[r].isalnum(): r -= 1` — same on the right.
- `if s[l].lower() != s[r].lower(): return False` — one mirrored mismatch disproves the whole string.
- `l, r = l + 1, r - 1` — this pair is verified; shrink.
- `return True` — every mirrored pair matched.

#### Dry Run
Input: `s = "0P"` (the classic failing test case)

| Step | l | r | s[l] | s[r] | Action |
|---|---|---|---|---|---|
| 1 | 0 | 1 | `0` | `P` | both alnum; `'0' != 'p'` → **return False** ✅ |

Input: `s = "a."`

| Step | l | r | Action |
|---|---|---|---|
| 1 | 0 | 1 | `s[1]='.'` not alnum → `r → 0` |
| 2 | 0 | 0 | loop guard `l < r` fails → **return True** |

Input: `s = "aba"`

| Step | l | r | Compare | Action |
|---|---|---|---|---|
| 1 | 0 | 2 | `a` vs `a` ✓ | l→1, r→1 |
| 2 | 1 | 1 | — | `l < r` false → **True** |

#### Complexity
- Time **O(n)** — each pointer moves forward at most n times total.
- Space **O(1)** — this is the whole reason to prefer it over `cleaned == cleaned[::-1]`, which is O(n) space.

#### Edge Cases
- `""` → `True` (loop never runs).
- `" "` / `",.;"` → `True`, but **only** because of the inner `l < r` guards.
- Single char → `True`.
- Mixed case → handled by `.lower()`.
- Digits → `isalnum()` includes them; `"0P"` must be `False`.

---

### 6.2 Two Sum II — Input Array Is Sorted (LC 167) — converging

#### Problem Understanding
A **sorted** array and a target. Return the **1-indexed** positions of the two numbers that sum to target. Exactly one solution exists; you may not use the same element twice. Required: O(1) extra space — which is precisely what rules out the hash map.

#### Approach
Start with the widest pair. `nums[l] + nums[r]` is monotonic: raising `l` can only raise the sum, lowering `r` can only lower it. So the comparison against `target` tells you exactly which pointer is wrong, and moving it eliminates an entire row of pairs at once.

#### Algorithm
1. `l = 0`, `r = n − 1`.
2. `s = nums[l] + nums[r]`.
3. `s == target` → return `[l+1, r+1]`.
4. `s < target` → need bigger → `l += 1`.
5. `s > target` → need smaller → `r -= 1`.

#### Code
```python
class Solution:
    def twoSum(self, numbers: list[int], target: int) -> list[int]:
        l, r = 0, len(numbers) - 1
        while l < r:
            s = numbers[l] + numbers[r]
            if s == target:
                return [l + 1, r + 1]
            elif s < target:
                l += 1
            else:
                r -= 1
        return []
```

#### Line-by-Line Explanation
- `l, r = 0, len(numbers) - 1` — smallest and largest candidates; the sum starts at the widest possible spread.
- `while l < r:` — `l == r` would reuse one element, which is banned.
- `s = numbers[l] + numbers[r]` — compute once, compare thrice.
- `if s == target: return [l + 1, r + 1]` — `+1` because the problem is 1-indexed. This is the most common submission bug.
- `elif s < target: l += 1` — **the elimination step.** If `numbers[l] + numbers[r]` (the *largest* partner available to `l`) is still too small, then `l` cannot pair with anything. Delete the whole row.
- `else: r -= 1` — symmetric: `r` paired with the smallest partner is still too big, so `r` is dead.
- `return []` — unreachable given the guarantee; keep it so the function is total.

#### Dry Run
Input: `numbers = [2, 3, 4], target = 6`

| Step | l | r | numbers[l] | numbers[r] | s | vs target | Action |
|---|---|---|---|---|---|---|---|
| 1 | 0 | 2 | 2 | 4 | 6 | = | **return [1, 3]** ✅ |

Input: `numbers = [2, 7, 11, 15], target = 18`

| Step | l | r | s | vs target | Action |
|---|---|---|---|---|---|
| 1 | 0 | 3 | 2+15=17 | < 18 | too small → `l → 1` (2 is dead: 2+15 was its best shot) |
| 2 | 1 | 3 | 7+15=22 | > 18 | too big → `r → 2` (15 is dead) |
| 3 | 1 | 2 | 7+11=18 | = | **return [2, 3]** ✅ |

#### Complexity
- Time **O(n)** — `l` and `r` together move at most n times.
- Space **O(1)** — the sortedness is what buys this.

#### Edge Cases
- Negatives — fine, monotonicity doesn't care about sign.
- Duplicates (`[1,1,1], target=2`) — fine, first valid pair returned.
- Two elements — one iteration.
- **Unsorted input → this is a different problem.** Use the hash-map Two Sum (Topic 01, Pattern B). Sorting first would cost O(n log n) *and* destroy the original indices.

---

### 6.3 Container With Most Water (LC 11) — the one that makes it click

#### Problem Understanding
Heights `h[0..n-1]` are vertical walls on a flat surface. Pick two walls; the water held is `min(h[l], h[r]) * (r − l)` — capped by the **shorter** wall, widened by the distance. Maximise it.

#### Approach
Start at maximum width and move in. Every inward move **loses** width, so it only pays off if height can grow — and height is capped by the shorter wall. Therefore the shorter wall is the only one worth abandoning.

**The proof — this is the part interviewers want:**
Say `h[l] < h[r]`. Consider any other pair `(l, r')` with `r' < r`:
- width is strictly smaller (`r' − l < r − l`), and
- height is still ≤ `h[l]` (the min can never exceed the short wall).

So every remaining pair involving `l` is **strictly worse** than the one just measured. `l` is provably exhausted — discard it. That argument, not the code, is the skill.

#### Algorithm
1. `l = 0`, `r = n − 1`, `best = 0`.
2. Measure `min(h[l], h[r]) * (r − l)`; keep the max.
3. Move the pointer at the **shorter** wall inward.
4. Repeat until `l == r`.

#### Code
```python
class Solution:
    def maxArea(self, height: list[int]) -> int:
        l, r, best = 0, len(height) - 1, 0
        while l < r:
            best = max(best, min(height[l], height[r]) * (r - l))
            if height[l] < height[r]:
                l += 1
            else:
                r -= 1
        return best
```

#### Line-by-Line Explanation
- `l, r, best = 0, len(height) - 1, 0` — widest container first; `best` is a running max so order of visits doesn't matter.
- `while l < r:` — a container needs two distinct walls.
- `min(height[l], height[r]) * (r - l)` — water level is the short wall; `r - l` is the base.
- `best = max(best, ...)` — record before moving, because the pointer we're about to drop can never be revisited.
- `if height[l] < height[r]: l += 1` — the short wall is exhausted (proof above).
- `else: r -= 1` — covers `h[l] > h[r]` and the tie. **On a tie either move is safe**: both walls are capped at the same value and any inward pair is narrower, so both are exhausted.
- `return best`.

#### Dry Run
Input: `height = [1, 8, 6, 2, 5, 4, 8, 3, 7]` (n = 9)

| Step | l | r | h[l] | h[r] | width | area | best | Move |
|---|---|---|---|---|---|---|---|---|
| 1 | 0 | 8 | 1 | 7 | 8 | 1×8 = 8 | 8 | h[l] smaller → l→1 |
| 2 | 1 | 8 | 8 | 7 | 7 | 7×7 = **49** | 49 | h[r] smaller → r→7 |
| 3 | 1 | 7 | 8 | 3 | 6 | 3×6 = 18 | 49 | r→6 |
| 4 | 1 | 6 | 8 | 8 | 5 | 8×5 = 40 | 49 | tie → else branch → r→5 |
| 5 | 1 | 5 | 8 | 4 | 4 | 4×4 = 16 | 49 | r→4 |
| 6 | 1 | 4 | 8 | 5 | 3 | 5×3 = 15 | 49 | r→3 |
| 7 | 1 | 3 | 8 | 2 | 2 | 2×2 = 4 | 49 | r→2 |
| 8 | 1 | 2 | 8 | 6 | 1 | 6×1 = 6 | 49 | r→1 |
| — | 1 | 1 | — | — | — | — | 49 | loop ends → **return 49** ✅ |

Note step 4: the tie at 8 vs 8. We measured 40 *before* moving, so nothing is lost.

#### Complexity
- Time **O(n)** — exactly `n − 1` iterations, one pointer moves each time.
- Space **O(1)**.

#### Edge Cases
- Two bars — single measurement.
- All equal heights → the answer is always the widest pair; the algorithm finds it on step 1.
- A zero height → area 0, harmless.
- **Trap:** greedily moving toward the taller wall is wrong; you can skip past the optimum.
- **Not the same problem as Trapping Rain Water (LC 42)** — that one sums water over *every* index and needs prefix-max/suffix-max (or a smarter two-pointer with `leftMax`/`rightMax`). Don't confuse them.

---

### 6.4 3Sum (LC 15) — sort, fix one, converge

#### Problem Understanding
Return **all unique** triplets summing to 0. Uniqueness is by *value*, not index — `[-1,0,1]` must appear once even if the input contains several `-1`s.

#### Approach
Reduce three pointers to two: **sort**, then fix an anchor `i` and run converging two pointers on the suffix looking for `−nums[i]`. Sorting does two jobs: it enables the converge step, *and* it makes duplicates adjacent, which turns dedup into a one-line skip.

#### Algorithm
1. Sort ascending.
2. For each anchor `i` from 0 to n−3:
   - `nums[i] > 0` → break (everything right is ≥ it, so no sum can reach 0).
   - `i > 0 and nums[i] == nums[i-1]` → continue (this anchor's triplets were already emitted).
   - `l = i+1`, `r = n−1`; converge on `s = nums[i] + nums[l] + nums[r]`.
   - `s < 0` → `l += 1`; `s > 0` → `r -= 1`; else record, then advance `l` past duplicates and `r -= 1`.

#### Code
```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        res = []
        for i in range(len(nums) - 2):
            if nums[i] > 0:
                break
            if i > 0 and nums[i] == nums[i - 1]:
                continue
            l, r = i + 1, len(nums) - 1
            while l < r:
                s = nums[i] + nums[l] + nums[r]
                if s < 0:
                    l += 1
                elif s > 0:
                    r -= 1
                else:
                    res.append([nums[i], nums[l], nums[r]])
                    l += 1
                    while l < r and nums[l] == nums[l - 1]:
                        l += 1
                    r -= 1
        return res
```

#### Line-by-Line Explanation
- `nums.sort()` — O(n log n), and it's the enabler for both converging *and* dedup.
- `for i in range(len(nums) - 2)` — stop at n−3; an anchor needs two elements after it.
- `if nums[i] > 0: break` — sorted, so `nums[l], nums[r] ≥ nums[i] > 0`; the sum can never be 0. Not just an optimisation, it's a clean early exit.
- `if i > 0 and nums[i] == nums[i-1]: continue` — **anchor dedup.** The previous identical anchor already scanned a *superset* of this suffix.
- `l, r = i + 1, len(nums) - 1` — search only to the right, which is what prevents reusing `i` and prevents permutation duplicates.
- `s = nums[i] + nums[l] + nums[r]` — one addition, three branches.
- `if s < 0: l += 1` — need bigger; only `l` can grow the sum.
- `elif s > 0: r -= 1` — need smaller; only `r` can shrink it.
- `res.append([...])` — a hit.
- `l += 1` then `while l < r and nums[l] == nums[l-1]: l += 1` — **left dedup.** After a hit, any identical `nums[l]` would re-emit the same triplet.
- `r -= 1` — with `l` moved and the sum previously exactly 0, `r` must also move or the sum goes positive. Moving both is correct and skips a wasted iteration.

#### Dry Run
Input: `nums = [-1, 0, 1, 2, -1, -4]` → sorted: `[-4, -1, -1, 0, 1, 2]` (n = 6)

| i | nums[i] | l | r | s | Action |
|---|---|---|---|---|---|
| 0 | −4 | 1 | 5 | −4−1+2 = −3 | < 0 → l→2 |
| 0 | −4 | 2 | 5 | −4−1+2 = −3 | < 0 → l→3 |
| 0 | −4 | 3 | 5 | −4+0+2 = −2 | < 0 → l→4 |
| 0 | −4 | 4 | 5 | −4+1+2 = −1 | < 0 → l→5, loop ends |
| 1 | −1 | 2 | 5 | −1−1+2 = 0 | **hit → `[-1,-1,2]`**; l→3 (`nums[3]=0 ≠ nums[2]=-1`, no skip); r→4 |
| 1 | −1 | 3 | 4 | −1+0+1 = 0 | **hit → `[-1,0,1]`**; l→4; r→3; loop ends |
| 2 | −1 | — | — | — | `nums[2] == nums[1]` → **continue** (dedup fires) |
| 3 | 0 | 4 | 5 | 0+1+2 = 3 | > 0 → r→4, loop ends |

Result: `[[-1,-1,2], [-1,0,1]]` ✅ — and without the `i` dedup at row 7, `[-1,0,1]` would appear twice.

#### Complexity
- Time **O(n²)** — O(n log n) sort + n anchors × O(n) converge. That *is* the good answer, down from O(n³).
- Space **O(1)** excluding the output (Python's Timsort uses O(n) auxiliary; Java's dual-pivot quicksort on primitives is O(log n) — mention this if pushed).

#### Edge Cases
- Fewer than 3 elements → `range(negative)` is empty → `[]`.
- All zeros `[0,0,0,0]` → one triplet `[0,0,0]`; the anchor dedup stops the rest.
- All positives / all negatives → `[]`; the `break` makes the positive case near-instant.
- Many duplicates → both dedup rules are needed; drop either and you get repeats.
- **4Sum** is this with one more nested anchor loop → O(n³).

---

### 6.5 Remove Duplicates from Sorted Array (LC 26) — read/write

#### Problem Understanding
Sorted array, remove duplicates **in place** so each value appears once, preserving order. Return the new length `k`; the first `k` slots must hold the answer. Whatever is beyond `k` doesn't matter.

#### Approach
`w` is the write cursor — the length of the answer built so far. `r` scans. Because the array is sorted, a duplicate can only ever be adjacent to the last thing kept, so a single comparison against `nums[w-1]` decides everything.

#### Algorithm
1. Empty → return 0.
2. `w = 1` (the first element is always kept).
3. For `r` from 1 to n−1: if `nums[r] != nums[w-1]`, write it at `w` and increment `w`.
4. Return `w`.

#### Code
```python
class Solution:
    def removeDuplicates(self, nums: list[int]) -> int:
        if not nums:
            return 0
        w = 1
        for r in range(1, len(nums)):
            if nums[r] != nums[w - 1]:
                nums[w] = nums[r]
                w += 1
        return w
```

#### Line-by-Line Explanation
- `if not nums: return 0` — guards the `w = 1` assumption.
- `w = 1` — element 0 has no predecessor, so it's kept unconditionally.
- `for r in range(1, len(nums))` — scan from the second element.
- `if nums[r] != nums[w - 1]` — compare against **the last thing kept**, not `nums[r-1]`. Same answer here because the input is sorted, but the `w-1` form is the one that generalises (e.g. LC 80, "allow at most 2", becomes `nums[r] != nums[w-2]`).
- `nums[w] = nums[r]` — write into a slot already consumed. `w ≤ r` always, so this never clobbers unread data.
- `w += 1` — grow the answer.
- `return w` — the new logical length.

#### Dry Run
Input: `nums = [0, 0, 1, 1, 1, 2, 2, 3, 3, 4]`

| r | nums[r] | w | nums[w−1] | equal? | Action | array prefix |
|---|---|---|---|---|---|---|
| 1 | 0 | 1 | 0 | yes | skip | `[0]` |
| 2 | 1 | 1 | 0 | no | `nums[1]=1`, w→2 | `[0,1]` |
| 3 | 1 | 2 | 1 | yes | skip | `[0,1]` |
| 4 | 1 | 2 | 1 | yes | skip | `[0,1]` |
| 5 | 2 | 2 | 1 | no | `nums[2]=2`, w→3 | `[0,1,2]` |
| 6 | 2 | 3 | 2 | yes | skip | `[0,1,2]` |
| 7 | 3 | 3 | 2 | no | `nums[3]=3`, w→4 | `[0,1,2,3]` |
| 8 | 3 | 4 | 3 | yes | skip | `[0,1,2,3]` |
| 9 | 4 | 4 | 3 | no | `nums[4]=4`, w→5 | `[0,1,2,3,4]` |

**return 5**, `nums[:5] == [0,1,2,3,4]` ✅

#### Complexity
- Time **O(n)** — single pass.
- Space **O(1)** — no auxiliary array. `list(dict.fromkeys(nums))` is also correct but O(n) space and doesn't satisfy "in place".

#### Edge Cases
- `[]` → 0 (guard).
- `[1]` → 1 (loop body never runs).
- All identical `[2,2,2]` → 1.
- Already unique → `w` tracks `r`; every element is rewritten onto itself, harmless.
- **Follow-up LC 80** (keep at most two): change the test to `w < 2 or nums[r] != nums[w-2]`. Same shape, different lag.

---

### 6.6 Move Zeroes (LC 283) — read/write with swap

#### Problem Understanding
Move all `0`s to the end **in place**, keeping the relative order of the non-zero elements. Minimise writes.

#### Approach
Same read/write cursor, but **swap** instead of overwrite. Overwriting would drop values on the floor and need a second pass to backfill zeroes; swapping carries the zeroes to the back automatically, so one pass finishes the job.

#### Algorithm
1. `w = 0`.
2. For each `r`: if `nums[r] != 0`, swap `nums[w]` and `nums[r]`, then `w += 1`.
3. Done — no return value; the mutation is the answer.

#### Code
```python
class Solution:
    def moveZeroes(self, nums: list[int]) -> None:
        w = 0
        for r in range(len(nums)):
            if nums[r] != 0:
                nums[w], nums[r] = nums[r], nums[w]
                w += 1
```

#### Line-by-Line Explanation
- `w = 0` — the boundary: `nums[0..w-1]` are the non-zeroes, in order.
- `for r in range(len(nums))` — scan everything.
- `if nums[r] != 0:` — only non-zeroes trigger work.
- `nums[w], nums[r] = nums[r], nums[w]` — `nums[w]` is either a zero (so the zero gets pushed right, exactly where it belongs) or `r == w` (a self-swap, a no-op). Both cases are correct, which is why no `if w != r` guard is needed.
- `w += 1` — extend the non-zero prefix.

**Why swap beats overwrite:** the overwrite version (`nums[w] = nums[r]; w += 1`) leaves stale values in `nums[w..n-1]` and needs a second loop to zero-fill. Swapping does it in one.

#### Dry Run
Input: `nums = [0, 1, 0, 3, 12]`

| r | nums[r] | w | Action | array after |
|---|---|---|---|---|
| 0 | 0 | 0 | zero → skip | `[0,1,0,3,12]` |
| 1 | 1 | 0 | swap idx 0↔1, w→1 | `[1,0,0,3,12]` |
| 2 | 0 | 1 | zero → skip | `[1,0,0,3,12]` |
| 3 | 3 | 1 | swap idx 1↔3, w→2 | `[1,3,0,0,12]` |
| 4 | 12 | 2 | swap idx 2↔4, w→3 | `[1,3,12,0,0]` |

**Result `[1,3,12,0,0]`** ✅ — order of `1,3,12` preserved.

#### Complexity
- Time **O(n)**, at most n swaps.
- Space **O(1)**.

#### Edge Cases
- No zeroes → every swap is `r == w`, a self-swap. Correct, though it does n redundant writes; add `if w != r` if the interviewer asks you to minimise writes.
- All zeroes → `w` stays 0, nothing moves, already correct.
- `[]` / `[0]` / `[1]` → trivially fine.
- **Order matters** — this is why you can't just use the converging shape and swap from both ends.

---

### 6.7 Sort Colors / Dutch National Flag (LC 75) — three pointers

#### Problem Understanding
Array of `0`, `1`, `2` only. Sort it in place, one pass, O(1) space, without a library sort and without a two-pass counting sort.

#### Approach
Three regions, so three pointers. Maintain the invariant:

```
[ 0 0 0 | 1 1 1 | ? ? ? ? | 2 2 2 ]
         low     mid       high
```

- `nums[0 .. low-1]` = all 0s
- `nums[low .. mid-1]` = all 1s
- `nums[mid .. high]` = unexamined
- `nums[high+1 .. n-1]` = all 2s

`mid` scans; every element it sees gets routed to the correct region.

#### Algorithm
1. `low = mid = 0`, `high = n − 1`.
2. While `mid <= high`:
   - `nums[mid] == 0` → swap with `low`; `low += 1`, `mid += 1`.
   - `nums[mid] == 1` → already in place; `mid += 1`.
   - `nums[mid] == 2` → swap with `high`; `high -= 1`; **do not move `mid`**.

#### Code
```python
class Solution:
    def sortColors(self, nums: list[int]) -> None:
        low, mid, high = 0, 0, len(nums) - 1
        while mid <= high:
            if nums[mid] == 0:
                nums[low], nums[mid] = nums[mid], nums[low]
                low += 1
                mid += 1
            elif nums[mid] == 1:
                mid += 1
            else:
                nums[mid], nums[high] = nums[high], nums[mid]
                high -= 1
```

#### Line-by-Line Explanation
- `low, mid, high = 0, 0, len(nums) - 1` — all three regions start empty; everything is unexamined.
- `while mid <= high:` — **`<=`, not `<`.** `nums[high]` is unexamined and must be processed; `<` leaves the last element unsorted.
- `if nums[mid] == 0:` — belongs at the front.
- `nums[low], nums[mid] = ...` — whatever sat at `low` was necessarily a `1` (or `mid == low`), so it lands safely at the front of the 1s region.
- `low += 1; mid += 1` — **both advance.** The incoming value is known-good, so `mid` may move on.
- `elif nums[mid] == 1: mid += 1` — it's already in the 1s region by construction.
- `else:` — it's a `2`; swap it to the back.
- `high -= 1` and **no `mid += 1`** — this is the whole problem. The value swapped in from `high` came from the *unexamined* zone. Advance `mid` and you skip it unchecked.

**The asymmetry in one line:** swapping with `low` brings in an already-examined value (advance `mid`); swapping with `high` brings in an unexamined value (don't).

#### Dry Run
Input: `nums = [2, 0, 2, 1, 1, 0]` (n = 6)

| Step | low | mid | high | nums | nums[mid] | Action |
|---|---|---|---|---|---|---|
| 1 | 0 | 0 | 5 | `[2,0,2,1,1,0]` | 2 | swap 0↔5, high→4, **mid stays** |
| 2 | 0 | 0 | 4 | `[0,0,2,1,1,2]` | 0 | swap 0↔0 (self), low→1, mid→1 |
| 3 | 1 | 1 | 4 | `[0,0,2,1,1,2]` | 0 | swap 1↔1 (self), low→2, mid→2 |
| 4 | 2 | 2 | 4 | `[0,0,2,1,1,2]` | 2 | swap 2↔4, high→3, **mid stays** |
| 5 | 2 | 2 | 3 | `[0,0,1,1,2,2]` | 1 | mid→3 |
| 6 | 2 | 3 | 3 | `[0,0,1,1,2,2]` | 1 | mid→4 |
| — | 2 | 4 | 3 | `[0,0,1,1,2,2]` | — | `mid > high` → stop ✅ |

Step 1 is the payoff: had `mid` advanced there, the `0` swapped in from index 5 would never be examined and the output would be `[0,0,2,1,1,2]`-ish garbage.

#### Complexity
- Time **O(n)** — `mid` only rises, `high` only falls; they meet after ≤ n steps.
- Space **O(1)**.
- Beats the two-pass counting sort (also O(n)) on the one-pass constraint, and beats `nums.sort()`'s O(n log n).

#### Edge Cases
- `[]` → `high = -1`, loop never runs.
- `[0]` / `[2]` → single step.
- All same value → one region fills, others stay empty.
- Already sorted → still one pass, all self-swaps.
- **Generalises** to the partition step of quicksort with duplicate pivots (3-way partition).

---

### 6.8 Middle of the Linked List (LC 876) — fast/slow

#### Problem Understanding
Return the middle node. On even length, return the **second** middle. Single pass, no length precomputation.

#### Approach
`fast` moves twice per `slow` move. Distance is linear in steps, so when `fast` has covered the whole list, `slow` has covered exactly half.

#### Algorithm
1. `slow = fast = head`.
2. While `fast` and `fast.next` both exist: `slow = slow.next`, `fast = fast.next.next`.
3. Return `slow`.

#### Code
```python
class Solution:
    def middleNode(self, head: ListNode) -> ListNode:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        return slow
```

#### Line-by-Line Explanation
- `slow = fast = head` — both start at the same node; the 2:1 ratio does all the work.
- `while fast and fast.next:` — **both checks, in this order.** `fast` guards against `None.next`; `fast.next` guards against `None.next.next`. Python short-circuits `and`, so the order matters.
- `slow = slow.next` — one step.
- `fast = fast.next.next` — two steps.
- `return slow` — when `fast` runs out, `slow` is at index `⌊n/2⌋`.

**Which middle you get, and how to flip it:** the loop above lands on the *second* middle for even n. For the *first* middle, loop on `while fast.next and fast.next.next:`.

#### Dry Run
Input: `1 → 2 → 3 → 4 → 5` (odd, n = 5)

| Step | slow | fast | guard `fast and fast.next` |
|---|---|---|---|
| start | 1 | 1 | ✓ |
| 1 | 2 | 3 | ✓ |
| 2 | 3 | 5 | `fast.next` is None → stop |

**return node 3** ✅ (index 2 = ⌊5/2⌋)

Input: `1 → 2 → 3 → 4 → 5 → 6` (even, n = 6)

| Step | slow | fast | guard |
|---|---|---|---|
| start | 1 | 1 | ✓ |
| 1 | 2 | 3 | ✓ |
| 2 | 3 | 5 | ✓ |
| 3 | 4 | None | `fast` is None → stop |

**return node 4** ✅ — the second middle (nodes 3 and 4 are both "middle").

#### Complexity
- Time **O(n)** — `fast` traverses once.
- Space **O(1)** — vs. O(n) for the "dump to a list and index" approach.

#### Edge Cases
- `head = None` → loop guard fails immediately, returns `None`.
- One node → returns it.
- Two nodes → returns the second.
- **Same machinery** as "remove Nth from end" (gap pointers) and "reorder list" (split at middle) — this loop is a building block, not a one-off.

---

### 6.9 Linked List Cycle (LC 141) + Cycle II (LC 142) — Floyd

#### Problem Understanding
141: does the list contain a cycle? 142: if so, return the node where the cycle *begins*. O(1) space required — which is exactly what rules out the hash set.

#### Approach
Run `fast` at 2× and `slow` at 1×. No cycle → `fast` falls off the end. Cycle → they must collide.

**The proof — say this out loud:** once both pointers are inside a loop of length `L`, `fast` gains exactly 1 net position per tick, so the gap between them decreases by exactly 1 each step. A non-negative integer that decreases by 1 every step and lives in `[0, L)` **must** reach 0. Collision is forced — it can't be jumped over, because the decrement is 1, not more.

#### Algorithm (141)
1. `slow = fast = head`.
2. While `fast and fast.next`: step slow 1, fast 2; if `slow is fast` → `True`.
3. Fell out → `False`.

#### Algorithm (142)
4. After the meeting, reset one pointer to `head`; advance **both at 1 step**; they meet at the cycle entrance.

#### Code
```python
class Solution:
    def hasCycle(self, head: ListNode) -> bool:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                return True
        return False


class SolutionII:
    def detectCycle(self, head: ListNode) -> ListNode:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:                 # phase 1: find a meeting point
                slow = head                  # phase 2: find the entrance
                while slow is not fast:
                    slow = slow.next
                    fast = fast.next
                return slow
        return None
```

#### Line-by-Line Explanation
- `slow = fast = head` — same start.
- `while fast and fast.next:` — the only way to exit without a cycle.
- `slow = slow.next` / `fast = fast.next.next` — the 2:1 ratio.
- `if slow is fast:` — **`is`, not `==`.** `==` may call `__eq__` and compare node *values*; two distinct nodes holding `3` would falsely "collide". In Java, `==` on references is correct and `.equals()` is the bug.
- **Check after moving, not before** — both start at `head`, so a pre-move check would report a cycle on step 0 for every list.
- `slow = head` (142) — reset one pointer, keep the other at the meeting point.
- `while slow is not fast: slow = slow.next; fast = fast.next` — both at 1× now.
- `return slow` — the cycle entrance.

**Why phase 2 works (the distance algebra):** let `F` = distance head→entrance, `a` = distance entrance→meeting point, `L` = cycle length. When they meet, slow travelled `F + a`, fast travelled `2(F + a)`, and fast's extra distance is a whole number of laps: `F + a = kL`. So `F = kL − a` — the distance from head to the entrance equals the distance from the meeting point forward to the entrance (modulo whole laps). Walking both at 1× therefore lands them together at the entrance. **Memorise the result; don't try to re-derive it live.**

#### Dry Run
Input: `3 → 2 → 0 → −4 → (back to node index 1, value 2)`. Nodes: `n0=3, n1=2, n2=0, n3=-4`; `n3.next = n1`. Cycle length `L = 3`, entrance = `n1`.

*Phase 1:*

| Step | slow | fast | collide? |
|---|---|---|---|
| start | n0 | n0 | (not checked) |
| 1 | n1 | n2 | no |
| 2 | n2 | n1 (n2→n3→n1) | no |
| 3 | n3 | n3 (n1→n2→n3) | **yes** — meeting point = n3 |

*Phase 2:* reset `slow = n0`, keep `fast = n3`.

| Step | slow | fast | equal? |
|---|---|---|---|
| start | n0 | n3 | no |
| 1 | n1 | n1 (n3→n1) | **yes** |

**return n1** ✅ — the entrance, value 2.

Check the algebra: `F = 1` (n0→n1), `a = 2` (n1→n2→n3), `L = 3`. `F + a = 3 = 1·L` ✓.

No-cycle input `1 → 2`: step 1 → `slow = n1`, `fast = None`; guard fails → **False** ✅.

#### Complexity
- Time **O(n)** — at most `F + L` steps in phase 1, `F` in phase 2.
- Space **O(1)** — the entire reason this trick is worth memorising. The hash-set version is O(n) space.

#### Edge Cases
- `head = None` → `False` / `None`.
- Single node, no cycle → guard fails immediately.
- Single node pointing at itself → step 1: slow = fast = the node → `True`.
- Duplicate values in the list → why `is` matters.
- Cycle of length 1 (self-loop at the tail) → gap shrinks to 0 in one step.

---

### 6.10 Merge Sorted Array (LC 88) — two inputs, fill from the back

#### Problem Understanding
`nums1` has length `m + n`: the first `m` slots hold sorted data, the last `n` are zero-padding. `nums2` holds `n` sorted values. Merge into `nums1` **in place**, O(1) extra space.

#### Approach
Forward merging would overwrite unread values in `nums1`. Merging **backwards** — largest first, writing into the tail — always writes into padding or into a slot whose value has already been consumed. The write cursor can never overtake the read cursor.

#### Algorithm
1. `i = m − 1` (last real value in `nums1`), `j = n − 1`, `w = m + n − 1`.
2. While `j >= 0`: write the larger of `nums1[i]` / `nums2[j]` at `w`; decrement that pointer and `w`.
3. When `j < 0` you're done — remaining `nums1` values are already in place.

#### Code
```python
class Solution:
    def merge(self, nums1: list[int], m: int, nums2: list[int], n: int) -> None:
        i, j, w = m - 1, n - 1, m + n - 1
        while j >= 0:
            if i >= 0 and nums1[i] > nums2[j]:
                nums1[w] = nums1[i]
                i -= 1
            else:
                nums1[w] = nums2[j]
                j -= 1
            w -= 1
```

#### Line-by-Line Explanation
- `i, j, w = m - 1, n - 1, m + n - 1` — three cursors, all at their respective tails.
- `while j >= 0:` — **loop on `j` only.** If `nums2` is exhausted, everything left in `nums1` is already sorted and in the right place — no copying needed. Looping on `i` too would be a wasted branch.
- `if i >= 0 and nums1[i] > nums2[j]:` — the `i >= 0` guard handles `nums1` running out first (all remaining `nums2` values then flow through the `else`).
- `nums1[w] = nums1[i]; i -= 1` — take from `nums1`.
- `else: nums1[w] = nums2[j]; j -= 1` — take from `nums2`. Using `>` (not `>=`) with the `else` branch keeps the merge **stable**.
- `w -= 1` — one write per iteration, unconditionally.

**Why backwards is safe, precisely:** at every moment `w ≥ i`, because `w − i = (number of nums2 values not yet written) ≥ 0`. So `nums1[w]` is never a slot holding unread data.

#### Dry Run
Input: `nums1 = [1, 2, 3, 0, 0, 0], m = 3`, `nums2 = [2, 5, 6], n = 3`

| Step | i | j | w | nums1[i] | nums2[j] | Pick | nums1 after |
|---|---|---|---|---|---|---|---|
| 1 | 2 | 2 | 5 | 3 | 6 | nums2 (6) → j→1 | `[1,2,3,0,0,6]` |
| 2 | 2 | 1 | 4 | 3 | 5 | nums2 (5) → j→0 | `[1,2,3,0,5,6]` |
| 3 | 2 | 0 | 3 | 3 | 2 | nums1 (3) → i→1 | `[1,2,3,3,5,6]` |
| 4 | 1 | 0 | 2 | 2 | 2 | tie → `else` → nums2 (2), j→−1 | `[1,2,2,3,5,6]` |
| — | 1 | −1 | 1 | — | — | `j < 0` → stop | `[1,2,2,3,5,6]` ✅ |

Step 3 is the one to notice: it wrote `3` into index 3, and index 3 was padding. Step 4 wrote into index 2, whose old value (`3`) had already been copied out. Never a clobber.

#### Complexity
- Time **O(m + n)** — one write per output slot.
- Space **O(1)** — the point of the exercise. `nums1[m:] = nums2; nums1.sort()` also passes LeetCode but is O((m+n)log(m+n)) and misses the intended answer.

#### Edge Cases
- `m = 0` → `i = −1`, the `i >= 0` guard sends everything down the `else`; `nums2` is copied wholesale.
- `n = 0` → `j = −1`, loop never runs, `nums1` untouched. ✅
- All of `nums2` smaller than all of `nums1` → `nums1` drains first, then the guard takes over.
- Duplicates across arrays → the `>` / `else` split keeps it stable.
- **Two separate output arrays** (not in-place) → merge *forwards*, which is the classic merge-sort step:

```python
def merge_sorted(a, b):
    i = j = 0
    out = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]:
            out.append(a[i]); i += 1
        else:
            out.append(b[j]); j += 1
    out.extend(a[i:]); out.extend(b[j:])
    return out
```

---

### 6.11 Intersection of Two Arrays II (LC 350) — two inputs

#### Problem Understanding
Return the intersection of two arrays, where **each element appears as many times as it does in both** (multiset intersection). Order doesn't matter.

#### Approach
If both are sorted: two pointers, O(1) extra space. Equal → emit and advance both. Otherwise advance whichever is smaller — it can never match anything further along in the other array.

#### Algorithm
1. `i = j = 0`.
2. While both in bounds: equal → append, `i += 1`, `j += 1`. `a[i] < b[j]` → `i += 1`. Else → `j += 1`.

#### Code
```python
class Solution:
    def intersect(self, nums1: list[int], nums2: list[int]) -> list[int]:
        nums1.sort()
        nums2.sort()
        i = j = 0
        out = []
        while i < len(nums1) and j < len(nums2):
            if nums1[i] == nums2[j]:
                out.append(nums1[i])
                i += 1
                j += 1
            elif nums1[i] < nums2[j]:
                i += 1
            else:
                j += 1
        return out
```

#### Line-by-Line Explanation
- `nums1.sort(); nums2.sort()` — required by the pattern. **If the inputs arrive sorted, delete these two lines and the solution is O(n+m) / O(1).**
- `while i < len(nums1) and j < len(nums2):` — either exhausted → no more matches possible.
- `if nums1[i] == nums2[j]:` — a match; advance **both**, which is what gives the correct multiplicity (three `2`s in one and two in the other yield exactly two `2`s).
- `elif nums1[i] < nums2[j]: i += 1` — `nums1[i]` is smaller than everything remaining in `nums2`; it can never match. Discard.
- `else: j += 1` — symmetric.

#### Dry Run
Input: `nums1 = [4,9,5]`, `nums2 = [9,4,9,8,4]` → sorted: `[4,5,9]` and `[4,4,8,9,9]`

| Step | i | j | nums1[i] | nums2[j] | Compare | Action | out |
|---|---|---|---|---|---|---|---|
| 1 | 0 | 0 | 4 | 4 | = | emit 4; i→1, j→1 | `[4]` |
| 2 | 1 | 1 | 5 | 4 | > | j→2 | `[4]` |
| 3 | 1 | 2 | 5 | 8 | < | i→2 | `[4]` |
| 4 | 2 | 2 | 9 | 8 | > | j→3 | `[4]` |
| 5 | 2 | 3 | 9 | 9 | = | emit 9; i→3, j→4 | `[4,9]` |
| — | 3 | 4 | — | — | — | `i` out of bounds → stop | `[4,9]` ✅ |

#### Complexity
- Inputs already sorted: **O(n + m)** time, **O(1)** extra space.
- Inputs unsorted (code above): **O(n log n + m log m)** — dominated by the sorts.
- **Hash-map alternative** (`Counter(nums1) & Counter(nums2)`): O(n + m) time, O(min(n,m)) space.

**Know which lever you're pulling.** Unsorted input → the Counter version is asymptotically better. Sorted input, or memory-constrained, or "the arrays are on disk and you can only stream them" (the classic follow-up) → two pointers wins, because it needs no random access and no memory proportional to the input.

#### Edge Cases
- Either array empty → `[]`.
- No overlap → `[]`; one pointer walks off the end.
- Full overlap with duplicates → multiplicity is `min(count_a, count_b)` per value, which the advance-both rule produces automatically.
- **LC 349** ("Intersection of Two Arrays", *distinct* results) → same loop, plus skip duplicates after an emit: `while i < len(a) and a[i] == a[i-1]: i += 1`.

---

## 7. When it is *not* two pointers

| Symptom | Actual pattern |
|---|---|
| Unsorted, need a pair | **Hash map** (Topic 01). Sorting to force two pointers costs more. |
| Non-contiguous subsequence | **DP** |
| Need a running summary of everything *between* the pointers (sum/count/freq) | **Sliding window** (Topic 03). Two pointers only reads the two endpoints. |
| Sum problem with **negative** numbers | **Prefix sum + hash map** (Topic 04) — shrinking can raise the sum, so monotonicity dies |
| Need max/min *of* a range | **Monotonic deque** |
| Need the k-th / median of a range | **Two heaps** |
| Search for one value in a sorted array | **Binary search** — halving beats shaving |

**The single test:** *can a comparison at `(l, r)` prove that one side contains no better answer?* No → it's not this pattern.

---

## 8. Drill plan (Week 2)

25-min cap per problem, then the +3 / +7 / +21 re-solve schedule.

| Day | Problem | Shape | Difficulty |
|---|---|---|---|
| 1 | Valid Palindrome (125) | A | Easy |
| 1 | Two Sum II (167) | A | Medium |
| 2 | Remove Duplicates (26) | B1 | Easy |
| 2 | Move Zeroes (283) | B1 | Easy |
| 3 | Container With Most Water (11) | A | Medium |
| 4 | 3Sum (15) | A + anchor | Medium |
| 4 | Sort Colors (75) | C | Medium |
| 5 | Middle of Linked List (876) | B2 | Easy |
| 5 | Linked List Cycle (141) → Cycle II (142) | B2 | Easy → Medium |
| 6 | Merge Sorted Array (88) | two inputs | Easy |
| 6 | Intersection of Two Arrays II (350) | two inputs | Easy |
| 7 | **Re-solve blind:** 11, 15, 75, 142 at 15 min each | — | — |

Java rep (during +7): Container With Most Water and Sort Colors.
Rust rep (Week 5+): Move Zeroes and Sort Colors — `a.swap(i, j)` sidesteps the borrow checker and the pattern will click.

---

## 9. Checkpoint (pass before Week 3)

Cold, no notes:

1. Solve Two Sum II + Valid Palindrome in under 10 minutes combined, bug-free first run.
2. State the Container With Most Water exhaustion argument in two sentences.
3. Explain why `mid` does **not** advance on the `high` swap in Dutch flag.
4. Prove Floyd's cycle detection terminates (gap shrinks by exactly 1).
5. Explain why LC 88 fills from the back.
6. Say, for any given problem, whether sorting to enable two pointers is worth the O(n log n).

Fail any → redo that item tomorrow, then move on.

---

## 10. Recall card

```
CONVERGING   l=0, r=n-1        → move the pointer that can't be in a better answer
READ/WRITE   w lags r          → w advances only on a keep
FAST/SLOW    fast 2, slow 1    → midpoint free, cycle free
THREE-WAY    low / mid / high  → don't advance mid on the high swap
```

| | trigger | move rule | cost |
|---|---|---|---|
| **Converging** | sorted + pair/triple, or mirror comparison | move the exhausted pointer | O(n) / O(1) |
| **Read/write** | in-place filter, dedupe, partition | `w` advances only on a keep | O(n) / O(1) |
| **Fast/slow 2×** | midpoint, cycle | fast 2, slow 1 | O(n) / O(1) |
| **Three-way** | partition into 3 buckets | `mid` frozen on the `high` swap | O(n) / O(1) |

Five things to say cold:

1. Two pointers works because each move **provably eliminates** possibilities — not because it's "two loops in one."
2. Container: the shorter wall is exhausted because width can only shrink and height is already capped.
3. Floyd: the gap shrinks by exactly 1 per step inside a cycle, so collision is forced.
4. Dutch flag: the `high` swap imports an unexamined value, so `mid` must not advance.
5. LC 88 fills from the back because forward writing clobbers unread data.

---

## 11. learndsa.org coverage

From **Topic 1 — Arrays and Strings**, fully covered above:

- Common Array Patterns → In-place Modification, Traversing from Both Ends
- Two Pointer Technique → Opposite Directional, Same Directional, Partition (Dutch flag)
- Common Problems → #4 Valid Palindrome
- Practice Exercises → merge two sorted arrays, move all zeros to the end, intersection of two arrays, remove duplicates

Still owed to Topic 01 (hash-map territory): two sum unsorted, valid anagram, first non-repeating char, unique characters, permutation check.
Deferred to Topic 03: sliding window, longest substring without repeating characters.
Deferred to Topic 04: binary search, cyclic sort, missing number, prefix sums.
