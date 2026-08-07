# Topic 2 — Two Pointers
*Python first · Java equivalents · Rust follow-up*

---

## 1. The Mental Model

A nested loop asks *"for every `i`, check every `j`"* — every step **adds** work → O(n²).

Two pointers asks a different question:

> **"Given `l` and `r` right now, can I prove one of them can never be part of a better answer?"**

If yes, discard it. Every step **deletes** work → O(n).

Topic 1 killed the inner search with a hash map (spend memory, buy time). Two pointers kills it with **structure** — usually sortedness — and spends *nothing*. This is the only family that improves time **and** space at once.

**Precondition:** a comparison at `(l, r)` must tell you something *definitive* about one whole side. No elimination argument ⇒ not two pointers.

The decision that comes up every time:

| Input | Use | Cost |
|---|---|---|
| Already sorted | two pointers | O(n) / **O(1)** |
| Not sorted | hash map (Topic 1) | O(n) / O(n) |
| Not sorted, but O(1) space demanded | sort, then two pointers | O(n log n) / O(1) |

Sorting to force two pointers costs `O(n log n)` — worse than the map's `O(n)`. Only do it when space is the binding constraint, or when you need *all* pairs (3Sum), not one.

---

## 2. Core Operations & Complexity

| Operation | Time | Space | Note |
|---|---|---|---|
| Converging scan (both ends → middle) | O(n) | O(1) | each index visited once |
| Read/write scan (in-place filter) | O(n) | O(1) | `w ≤ r` invariant |
| Fast/slow scan | O(n) | O(1) | fast covers 2n, still linear |
| Fixed-gap scan (k-th from end) | O(n) | O(1) | one pass, no length count |
| Sort, then converge | O(n log n) | O(1)–O(n) | sort dominates |
| Fix one index + converge (3Sum) | O(n²) | O(1) | n anchors × O(n) scan |

Gotchas worth saying aloud in interviews:
- **`w ≤ r` always** in read/write — that inequality is *why* in-place overwriting is safe, not a coincidence.
- Two pointers on linked lists gives O(1) space where a hash set gives O(n) — that's the whole reason Floyd's algorithm is famous.
- `while l < r` vs `l <= r`: use `<` when an element can't pair with itself (converging), `<=` when every index must be examined (Dutch flag).
- 3Sum is `O(n²)` and that *is* the optimal answer — don't apologise for it.

---

## 3. Syntax — Python ↔ Java ↔ Rust

### Pointer mechanics

| Task | Python | Java | Rust |
|---|---|---|---|
| Init converging | `l, r = 0, len(a)-1` | `int l=0, r=a.length-1;` | `let (mut l, mut r) = (0usize, a.len()-1);` |
| Advance both | `l += 1; r -= 1` | `l++; r--;` | `l += 1; r -= 1;` |
| Swap | `a[i],a[j] = a[j],a[i]` | `int t=a[i];a[i]=a[j];a[j]=t;` | `a.swap(i, j);` |
| In-place write | `a[w] = a[r]` | `a[w++] = a[r];` | `let v=a[r]; a[w]=v; w+=1;` |
| Index type | `int` (signed) | `int` (signed) | **`usize` (unsigned)** |
| Null guard | `while fast and fast.next:` | `while (fast!=null && fast.next!=null)` | `Option` / `while let` |
| Short-circuit | `and` / `or` | `&&` / `\|\|` | `&&` / `\|\|` |

⚠ **`a[-1]` in Python is legal and wrong.** `a[w-1]` when `w == 0` silently reads the **last element of the array** — no error, plausible-looking wrong output. Java throws `ArrayIndexOutOfBoundsException`. Rust panics on `usize` underflow. This single difference is why `w == 0` guards exist (§5.6).

⚠ **Rust indices are `usize`.** `w - 1` with `w == 0` is "attempt to subtract with overflow" — a panic, not a negative number. The guard is mandatory, not stylistic.

### Strings & chars

| Task | Python | Java | Rust |
|---|---|---|---|
| Char at i | `s[i]` | `s.charAt(i)` | `s.as_bytes()[i]` |
| Alphanumeric? | `c.isalnum()` | `Character.isLetterOrDigit(c)` | `c.is_alphanumeric()` |
| Lowercase | `c.lower()` | `Character.toLowerCase(c)` | `c.to_ascii_lowercase()` |
| To char list | `list(s)` | `s.toCharArray()` | `s.chars().collect::<Vec<char>>()` |

⚠ Rust strings are UTF-8 — `s[i]` doesn't compile. `as_bytes()[i]` is fine (and faster) for ASCII problems, which is all of LeetCode.

### Linked lists

| Task | Python | Java |
|---|---|---|
| Node | `class ListNode: val, next` | `class ListNode { int val; ListNode next; }` |
| Dummy head | `dummy = ListNode(0, head)` | `ListNode dummy = new ListNode(0, head);` |
| Identity compare | `slow is fast` | `slow == fast` |
| Value compare | `slow.val == fast.val` | `slow.val == fast.val` |

⚠ Compare **identity**, not `.val` — duplicate values must not register as a cycle collision.

⚠ Rust linked lists are `Option<Box<ListNode>>`; two pointers into one list = two mutable borrows, which the borrow checker rejects. Real Rust needs `Rc<RefCell<>>` or raw pointers. **Do §5.10–5.13 in Python or Java — the Rust version teaches you Rust, not the pattern.**

---

## 4. The Shapes

### Shape A — Converging (opposite ends)

*Signal: sorted array + "find a pair/triplet"; mirror comparison (palindrome); brute force would check all pairs.*

```
l →                    ← r
[ _ _ _ _ _ _ _ _ _ _ _ ]
```

Move rule, always the same: **move the pointer that cannot participate in a better answer.** Everything else is problem-specific dressing.

```python
l, r = 0, len(a) - 1
while l < r:
    if   <hit>:         return ...
    elif <need bigger>: l += 1
    else:               r -= 1
```

```java
int l = 0, r = a.length - 1;
while (l < r) {
    if (/* hit */) return /* ... */;
    else if (/* need bigger */) l++;
    else r--;
}
```

---

### Shape B1 — Read / write

*Signal: "in place", "return k", remove / dedupe / partition with order preserved.*

```
      w              ← next free slot
[ k k | ? ? ? ? ? ]
        r            ← scanner
```

Two cursors, **asymmetric jobs**: `r` decides *what to look at*, `w` decides *where it lands*.

| | job | when it moves |
|---|---|---|
| `r` | scan | every iteration, unconditionally |
| `w` | mark the end of the kept region | **only on a keep** |

**Invariant:** `w = (seen) − (dropped)`. The gap between `r` and `w` **is** the discard count.

**Safety:** `w` advances only when `r` does ⇒ `w ≤ r` always ⇒ every slot you overwrite has already been read. Nothing unread can be destroyed.

`nums[0:w]` is the answer zone. `nums[w-1]` is the last committed value. `nums[w]` is the next free slot.

```python
w = 0
for r in range(len(a)):
    if keep(a[r]):
        a[w] = a[r]
        w += 1
return w
```

```java
int w = 0;
for (int r = 0; r < a.length; r++)
    if (keep(a[r])) a[w++] = a[r];
return w;
```

```rust
let mut w = 0usize;
for r in 0..a.len() {
    let val = a[r];            // read, then write — keeps the borrow checker quiet
    if keep(val) { a[w] = val; w += 1; }
}
w
```

**Swap the `keep` predicate, get a different problem.** That's the entire pattern:

| Problem | `keep` |
|---|---|
| Remove Element (27) | `a[r] != val` |
| Move Zeroes (283) | `a[r] != 0` (swap, not assign) |
| Remove Duplicates (26) | `w == 0 or a[r] != a[w-1]` |
| Remove Duplicates II (80) | `w < 2 or a[r] != a[w-2]` |

⚠ The first two have a **self-contained** `keep` — it reads only `a[r]`. The last two peek *backward* into the answer zone at `a[w-1]`. That backward peek is the only reason #26 is harder than #27. **Learn §5.7 first, then read §5.6 as "the same thing with a backward-looking keep."**

---

### Shape B2 — Fast / slow

*Signal: linked list + "middle", "cycle", "k-th from the end"; O(1) space demanded.*

**Two-speed variant** (`fast` moves 2, `slow` moves 1):

**Midpoint for free** — when `fast` has covered `n`, `slow` has covered `n/2`. When `fast` falls off the end, `slow` is on the middle. No length pass.

**Cycle detection for free** — once both are inside a loop of length `L`, `fast` gains exactly 1 net position per step, so the gap shrinks by exactly 1 each step. A value confined to `[0, L)` that decreases by 1 per step **must** reach 0 ⇒ collision is forced. No cycle ⇒ `fast` hits `null` first.

**Fixed-gap variant** — start `fast` `n` steps ahead, advance both at speed 1. When `fast` reaches the last node, `slow` is exactly `n` from the end.

```python
slow = fast = head
while fast and fast.next:
    slow, fast = slow.next, fast.next.next
```

```java
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) { slow = slow.next; fast = fast.next.next; }
```

⚠ `while fast and fast.next` — both checks, **in that order**. `fast.next.next` needs `fast.next`; `fast.next` needs `fast`. Short-circuit `and` makes the order load-bearing.

---

## 5. Worked Problems

Format per problem: **Approach → Algorithm → Code → Line notes → Dry run → Complexity → Edge cases.**

---

### 5.1 Valid Palindrome — LC 125 — Shape A

**Problem.** Is `s` a palindrome, considering only alphanumerics, ignoring case?

**Approach.** Mirror comparison ⇒ converging. Skip junk *before* comparing, both sides. Building a cleaned copy would cost O(n) space; this is O(1).

**Algorithm.**
1. `l=0`, `r=n-1`.
2. Advance `l` past non-alphanumerics; retreat `r` past non-alphanumerics.
3. Compare lowercased. Mismatch → `False`.
4. Step both inward. Survive the loop → `True`.

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        l, r = 0, len(s) - 1
        while l < r:
            while l < r and not s[l].isalnum(): l += 1
            while l < r and not s[r].isalnum(): r -= 1
            if s[l].lower() != s[r].lower():
                return False
            l += 1; r -= 1
        return True
```

```java
public boolean isPalindrome(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;
        while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
        if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r))) return false;
        l++; r--;
    }
    return true;
}
```

**Line notes.**
- ⚠ The `l < r` inside the **inner** loops is load-bearing. Without it, `",,,"` walks `l` off the end → `IndexError`.
- `.lower()` on both sides — never compare raw chars.

**Dry run.** `s = "0P"`

| l | r | s[l] | s[r] | compare | action |
|---|---|---|---|---|---|
| 0 | 1 | `0` | `P` | `'0'` vs `'p'` | mismatch → **False** |

This is the test case that kills naive solutions: `'0'` and `'P'` differ by 32 in ASCII, so any `tolower`-by-arithmetic trick returns `True` here. `.lower()` on a digit is a no-op, which is correct.

**Complexity.** Time O(n) — each index passed once, by exactly one pointer. Space O(1).

**Edge cases** (verified):

| Input | Result | Why |
|---|---|---|
| `""` | `True` | loop never entered |
| `" "` | `True` | inner loops collapse `l` onto `r` |
| `".,"` | `True` | all skipped |
| `"0P"` | `False` | digit vs letter |
| `"race a car"` | `False` | genuine mismatch |

---

### 5.2 Two Sum II — LC 167 — Shape A

**Problem.** `numbers` sorted ascending. Return the **1-indexed** positions of the two values summing to `target`. Exactly one solution.

**Approach.** Sorted ⇒ converging. Start at the widest pair, narrow.

**The elimination argument.** If `numbers[l] + numbers[r] < target`, then `numbers[l]` paired with *anything at or left of* `r` is also too small — every such value is ≤ `numbers[r]`. So `l` is dead. One move deletes an entire row of pairs.

**Algorithm.**
1. `l=0`, `r=n-1`.
2. `s = numbers[l] + numbers[r]`.
3. `s == target` → return `[l+1, r+1]`; `s < target` → `l++`; `s > target` → `r--`.

```python
class Solution:
    def twoSum(self, numbers: list[int], target: int) -> list[int]:
        l, r = 0, len(numbers) - 1
        while l < r:
            s = numbers[l] + numbers[r]
            if s == target:  return [l + 1, r + 1]
            elif s < target: l += 1
            else:            r -= 1
        return []
```

```java
public int[] twoSum(int[] numbers, int target) {
    int l = 0, r = numbers.length - 1;
    while (l < r) {
        int s = numbers[l] + numbers[r];
        if (s == target) return new int[]{l + 1, r + 1};
        else if (s < target) l++;
        else r--;
    }
    return new int[]{};
}
```

**Line notes.**
- ⚠ `return [l+1, r+1]` — LC 167 is **1-indexed**; LC 1 is 0-indexed. Only difference between them, and it costs people the submission.
- `while l < r`, not `<=` — an element can't pair with itself.

**Dry run.** `numbers = [2,7,11,15]`, `target = 9`

| l | r | sum | vs target | action |
|---|---|---|---|---|
| 0 | 3 | 2+15 = 17 | > 9 | `r--` |
| 0 | 2 | 2+11 = 13 | > 9 | `r--` |
| 0 | 1 | 2+7 = 9 | = 9 | return `[1,2]` |

**Complexity.** Time O(n), Space O(1).

**Edge cases.** Negatives fine (`[-1,0], -1 → [1,2]`) — the ordering argument never assumed positivity. Duplicates fine. Answer at the extremes fine.

**Contrast with Topic 1.** LC 1 (unsorted) → hash map, O(n)/O(n). Sorting it to force two pointers costs O(n log n) *and* destroys the original indices you were asked to return.

---

### 5.3 Container With Most Water — LC 11 — Shape A

**Problem.** `height[i]` is a vertical line. Maximise `min(h[l], h[r]) * (r - l)`.

**Approach.** Start at maximum width; move inward. The greedy move is always **discard the shorter wall**.

**Why that's provably safe** — this *is* the problem:

Say `h[l] < h[r]`. Take any other container using `l`, with right wall `r'` where `l < r' < r`:
- width `r' - l` is **strictly smaller** than `r - l`
- height `min(h[l], h[r'])` is **≤ `h[l]`** — capped by the short wall regardless of `h[r']`

Both factors ≤, one strictly. Every remaining pair involving `l` is strictly worse than the one just measured. `l` is exhausted.

**Algorithm.**
1. `l=0`, `r=n-1`, `best=0`.
2. `best = max(best, min(h[l],h[r]) * (r-l))`.
3. Move the shorter wall inward.

```python
class Solution:
    def maxArea(self, height: list[int]) -> int:
        l, r, best = 0, len(height) - 1, 0
        while l < r:
            best = max(best, min(height[l], height[r]) * (r - l))
            if height[l] < height[r]: l += 1
            else:                     r -= 1
        return best
```

```java
public int maxArea(int[] h) {
    int l = 0, r = h.length - 1, best = 0;
    while (l < r) {
        best = Math.max(best, Math.min(h[l], h[r]) * (r - l));
        if (h[l] < h[r]) l++; else r--;
    }
    return best;
}
```

**Line notes.**
- Measure **before** moving — the current pair is a candidate.
- Ties (`h[l] == h[r]`): move either. Both walls are exhausted by the same argument; the `else` handles it.

**Dry run.** `height = [1,8,6,2,5,4,8,3,7]`

| l | r | h[l] | h[r] | width | area | best | move |
|---|---|---|---|---|---|---|---|
| 0 | 8 | 1 | 7 | 8 | 8 | 8 | `l++` (1<7) |
| 1 | 8 | 8 | 7 | 7 | **49** | 49 | `r--` (8≥7) |
| 1 | 7 | 8 | 3 | 6 | 18 | 49 | `r--` |
| 1 | 6 | 8 | 8 | 5 | 40 | 49 | `r--` (tie) |
| 1 | 5 | 8 | 4 | 4 | 16 | 49 | `r--` |
| 1 | 4 | 8 | 5 | 3 | 15 | 49 | `r--` |
| 1 | 3 | 8 | 2 | 2 | 4 | 49 | `r--` |
| 1 | 2 | 8 | 6 | 1 | 6 | 49 | `r--` |

`l == r` → **49**.

**Complexity.** Time O(n) — the pointers move `n-1` times total. Space O(1). Brute force is O(n²).

**Edge cases.** `n == 2` → one measurement. Zeros → area 0, no crash. All-equal heights → widest pair wins on iteration 1. *(Randomised vs brute force, 400 cases — matches.)*

---

### 5.4 3Sum — LC 15 — Shape A + outer loop

**Problem.** All **unique** triplets summing to 0.

**Approach.** Sort, then **fix `i` and converge on the suffix** for `target = -nums[i]`. Sorting does double duty: it enables the converging scan *and* makes duplicates adjacent, so skipping them is O(1).

**Algorithm.**
1. Sort.
2. For each `i`: break if `nums[i] > 0`; skip if `nums[i] == nums[i-1]`.
3. Converge `l = i+1`, `r = n-1` on sum 0.
4. On a hit: record, advance `l` past duplicates, decrement `r`.

```python
class Solution:
    def threeSum(self, nums: list[int]) -> list[list[int]]:
        nums.sort()
        res = []
        for i in range(len(nums) - 2):
            if nums[i] > 0: break
            if i > 0 and nums[i] == nums[i - 1]: continue
            l, r = i + 1, len(nums) - 1
            while l < r:
                s = nums[i] + nums[l] + nums[r]
                if s < 0:   l += 1
                elif s > 0: r -= 1
                else:
                    res.append([nums[i], nums[l], nums[r]])
                    l += 1
                    while l < r and nums[l] == nums[l - 1]: l += 1
                    r -= 1
        return res
```

```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (nums[i] > 0) break;
        if (i > 0 && nums[i] == nums[i - 1]) continue;
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int s = nums[i] + nums[l] + nums[r];
            if (s < 0) l++;
            else if (s > 0) r--;
            else {
                res.add(Arrays.asList(nums[i], nums[l], nums[r]));
                l++;
                while (l < r && nums[l] == nums[l - 1]) l++;
                r--;
            }
        }
    }
    return res;
}
```

**Line notes — dedup is the entire difficulty. Two separate rules, both required:**
- `if i > 0 and nums[i] == nums[i-1]: continue` — skip repeated **anchors**. ⚠ Compare **backward** (`i-1`), never forward: comparing to `i+1` skips the *first* occurrence, which is the one you need.
- `while l < r and nums[l] == nums[l-1]: l += 1` — after recording, skip repeated **lefts**. Runs *after* `l += 1`, so `nums[l-1]` is the value just consumed.
- `r -= 1` sits outside that while — `r` needs no dedup loop; a fixed `i` and fixed `l` admit exactly one `r`.
- `if nums[i] > 0: break` — sorted, so `l` and `r` are positive too; the sum can't reach 0. Prunes only; correctness unaffected.

**Dry run.** `nums = [-1,0,1,2,-1,-4]` → sorted `[-4,-1,-1,0,1,2]`

| i | nums[i] | l | r | sum | action |
|---|---|---|---|---|---|
| 0 | -4 | 1 | 5 | -4-1+2 = -3 | `l++` |
| 0 | -4 | 2 | 5 | -4-1+2 = -3 | `l++` |
| 0 | -4 | 3 | 5 | -4+0+2 = -2 | `l++` |
| 0 | -4 | 4 | 5 | -4+1+2 = -1 | `l++` → `l==r`, exit |
| 1 | -1 | 2 | 5 | -1-1+2 = **0** | record `[-1,-1,2]`; `l→3`; `r→4` |
| 1 | -1 | 3 | 4 | -1+0+1 = **0** | record `[-1,0,1]`; `l→4`; `r→3`; exit |
| 2 | -1 | — | — | — | **skipped** — `nums[2]==nums[1]` |
| 3 | 0 | 4 | 5 | 0+1+2 = 3 | `r--` → `l==r`, exit |

Return `[[-1,-1,2], [-1,0,1]]`.

Row `i=2` is the payoff: without the anchor-skip you'd emit both triplets a second time.

**Complexity.** Time O(n²) — O(n log n) sort + n anchors × O(n) scan; n² dominates. Space O(1) excluding output (O(log n)–O(n) for sort internals). Brute force is O(n³).

**Edge cases** (verified, plus 400 randomised vs brute force):

| Input | Output | Why |
|---|---|---|
| `[0,1,1]` | `[]` | nothing sums to 0 |
| `[0,0,0]` | `[[0,0,0]]` | one triplet |
| `[0,0,0,0]` | `[[0,0,0]]` | anchor-skip prevents the duplicate |
| `[]`, `[1]`, `[1,2]` | `[]` | `range(len-2)` empty — no negative-length bug |

---

### 5.5 Trapping Rain Water — LC 42 — Shape A

**Problem.** Water trapped between bars.

**Approach.** Water above bar `i` is `min(maxLeft(i), maxRight(i)) - h[i]`. The obvious fix is two prefix-max arrays → O(n) space. Two pointers gets O(1).

**The key argument.** Track running maxima `lmax`, `rmax` from each end. If `lmax <= rmax`, then for the **left** pointer the binding constraint is *certainly* `lmax` — because the true right maximum is at least `rmax`, which is at least `lmax`. You never need to know the actual right maximum. So position `l` can be settled now.

**Algorithm.**
1. `l=0`, `r=n-1`, `lmax=h[l]`, `rmax=h[r]`, `total=0`.
2. `lmax <= rmax` → `l++`, update `lmax`, add `lmax - h[l]`.
3. Else → `r--`, update `rmax`, add `rmax - h[r]`.

```python
class Solution:
    def trap(self, height: list[int]) -> int:
        if not height: return 0
        l, r = 0, len(height) - 1
        lmax, rmax, total = height[l], height[r], 0
        while l < r:
            if lmax <= rmax:
                l += 1
                lmax = max(lmax, height[l])
                total += lmax - height[l]
            else:
                r -= 1
                rmax = max(rmax, height[r])
                total += rmax - height[r]
        return total
```

```java
public int trap(int[] h) {
    if (h.length == 0) return 0;
    int l = 0, r = h.length - 1, lmax = h[0], rmax = h[h.length - 1], total = 0;
    while (l < r) {
        if (lmax <= rmax) { l++; lmax = Math.max(lmax, h[l]); total += lmax - h[l]; }
        else              { r--; rmax = Math.max(rmax, h[r]); total += rmax - h[r]; }
    }
    return total;
}
```

**Line notes.**
- ⚠ Order: **move first**, then update the max, then add. Done this way, `lmax ≥ h[l]` holds by construction, so `lmax - h[l]` is never negative and no `max(0, …)` clamp is needed.
- ⚠ `<=` not `<`. With `<`, equal maxima send you to the `else` branch forever on a flat array. Ties must break one way consistently.

**Dry run.** `height = [4,2,0,3,2,5]`

| l | r | lmax | rmax | branch | after move | added | total |
|---|---|---|---|---|---|---|---|
| 0 | 5 | 4 | 5 | `4≤5` L | `l=1`, `lmax=4` | 4−2 = 2 | 2 |
| 1 | 5 | 4 | 5 | `4≤5` L | `l=2`, `lmax=4` | 4−0 = 4 | 6 |
| 2 | 5 | 4 | 5 | `4≤5` L | `l=3`, `lmax=4` | 4−3 = 1 | 7 |
| 3 | 5 | 4 | 5 | `4≤5` L | `l=4`, `lmax=4` | 4−2 = 2 | 9 |
| 4 | 5 | 4 | 5 | `4≤5` L | `l=5`, `lmax=5` | 5−5 = 0 | 9 |

`l == r` → **9**.

**Complexity.** Time O(n), Space O(1). The prefix-array version is O(n)/O(n).

**Edge cases.** `[]` → 0 (explicit guard; without it `height[0]` throws). `[5]` → 0 (`l==r`, loop skipped). Monotonic input → 0. *(Randomised vs brute force, 400 cases — matches.)*

---

### 5.6 Remove Duplicates from Sorted Array — LC 26 — Shape B1

**Problem.** Sorted array. Remove duplicates in place; return `k` = unique count, with `nums[0:k]` holding them in order.

**Approach.** Read/write. Sorted ⇒ duplicates are adjacent ⇒ compare only against the **last kept** value, `nums[w-1]`.

**Algorithm.**
1. `w = 0`.
2. Keep `nums[r]` if `w == 0` (nothing kept yet) **or** `nums[r] != nums[w-1]`.
3. On keep: `nums[w] = nums[r]`, `w++`.

```python
class Solution:
    def removeDuplicates(self, nums: list[int]) -> int:
        w = 0
        for r in range(len(nums)):
            if w == 0 or nums[r] != nums[w - 1]:
                nums[w] = nums[r]
                w += 1
        return w
```

```java
public int removeDuplicates(int[] nums) {
    int w = 0;
    for (int r = 0; r < nums.length; r++)
        if (w == 0 || nums[r] != nums[w - 1]) nums[w++] = nums[r];
    return w;
}
```

```rust
pub fn remove_duplicates(nums: &mut Vec<i32>) -> i32 {
    let mut w = 0usize;
    for r in 0..nums.len() {
        let val = nums[r];
        if w == 0 || val != nums[w - 1] { nums[w] = val; w += 1; }
    }
    w as i32
}
```

**Line notes — read all four, this is the problem people get wrong:**

- **`w` is a count, not a value.** It holds *how many elements you've kept*. It is never an array element. `w == 0` means "kept nothing yet" — it has **nothing to do with the number `0` appearing in the data**. `[0,0,0,1,1]` → `[0,1]`, correct. Rename it `kept_so_far` in your head if that helps.
- ⚠ **The `w == 0` guard is mandatory.** Without it, `nums[w-1]` is `nums[-1]` = the **last element of the array**, read silently in Python. `[2,2,2]` returns `[]` instead of `[2]`. Java: `ArrayIndexOutOfBoundsException`. Rust: `usize` underflow panic. And note the bug only shows when `nums[0] == nums[-1]` — `[1,1,2,2,2,3]` gives the right answer either way, so it passes a casual test and fails in the interview.
- ⚠ **Guard order matters.** `or` short-circuits, so `w == 0` must be **left** of `nums[r] != nums[w-1]`, or `nums[-1]` evaluates before the guard can stop it. Same reason you write `if node and node.val`.
- **Source is `r`, not `w+1`.** `nums[w] = nums[r]` — `r` is often far ahead. In the trace below the final write is `nums[4] = nums[9]`, five slots apart.
- Compare against `nums[w-1]` (last **kept**), not `nums[r-1]` (last **seen**). Identical on sorted input; only `w-1` generalises.

**Dry run.** `nums = [0,0,1,1,1,2,2,3,3,4]`

| r | nums[r] | w | `w==0`? | nums[w-1] | keep | array after | w' |
|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | yes | *(not read)* | ✓ | `[0,0,1,1,1,2,2,3,3,4]` | 1 |
| 1 | 0 | 1 | no | `nums[0]`=0 | ✗ | unchanged | 1 |
| 2 | 1 | 1 | no | `nums[0]`=0 | ✓ | `[0,`**`1`**`,1,1,1,2,2,3,3,4]` | 2 |
| 3 | 1 | 2 | no | `nums[1]`=1 | ✗ | unchanged | 2 |
| 4 | 1 | 2 | no | `nums[1]`=1 | ✗ | unchanged | 2 |
| 5 | 2 | 2 | no | `nums[1]`=1 | ✓ | `[0,1,`**`2`**`,1,1,2,2,3,3,4]` | 3 |
| 6 | 2 | 3 | no | `nums[2]`=2 | ✗ | unchanged | 3 |
| 7 | 3 | 3 | no | `nums[2]`=2 | ✓ | `[0,1,2,`**`3`**`,1,2,2,3,3,4]` | 4 |
| 8 | 3 | 4 | no | `nums[3]`=3 | ✗ | unchanged | 4 |
| 9 | 4 | 4 | no | `nums[3]`=3 | ✓ | `[0,1,2,3,`**`4`**`,2,2,3,3,4]` | 5 |

Return **5**, `nums[:5] = [0,1,2,3,4]`. Tail `[2,2,3,3,4]` is garbage by design.

Watch the `nums[w-1]` column: `0 → 1 → 2 → 3`. It always names the most recent commit, because committing is exactly what moves `w`.

**Complexity.** Time O(n), Space O(1).

**Edge cases** (verified):

| Input | k | `nums[:k]` | Why |
|---|---|---|---|
| `[]` | 0 | `[]` | loop never runs |
| `[5]` | 1 | `[5]` | guard fires |
| `[2,2,2]` | 1 | `[2]` | **the case that breaks without the guard** |
| `[-3,-3,0,0,7]` | 3 | `[-3,0,7]` | negatives and literal `0`s fine |
| `[1,2,3]` | 3 | `[1,2,3]` | every write is `nums[r]=nums[r]`, a harmless no-op |

**Variant you'll see online.** LC guarantees `len ≥ 1`, so most solutions start `w = 1` and drop the guard. Shorter, crashes on `[]`, hides the general shape. `w = 0` is the version that adapts.

---

### 5.7 Remove Element — LC 27 — Shape B1

**Problem.** Remove every occurrence of `val` in place. Return `k`. Order of the remainder is irrelevant.

**Approach.** Read/write with the simplest possible `keep` — it inspects only `nums[r]`, nothing else. **This is the pattern with nothing layered on top. Get this one solid before §5.6 makes sense.**

```python
class Solution:
    def removeElement(self, nums: list[int], val: int) -> int:
        w = 0
        for r in range(len(nums)):
            if nums[r] != val:
                nums[w] = nums[r]
                w += 1
        return w
```

```java
public int removeElement(int[] nums, int val) {
    int w = 0;
    for (int r = 0; r < nums.length; r++)
        if (nums[r] != val) nums[w++] = nums[r];
    return w;
}
```

**Line notes.** No `w == 0` guard needed — `keep` never reads `nums[w-1]`. That is *precisely* why this is easier than §5.6.

**Dry run.** `nums = [0,1,2,2,3,0,4,2]`, `val = 2`

| r | nums[r] | keep | action | array after | w |
|---|---|---|---|---|---|
| 0 | 0 | ✓ | `nums[0]=0` (no-op) | `[0,1,2,2,3,0,4,2]` | 1 |
| 1 | 1 | ✓ | `nums[1]=1` (no-op) | unchanged | 2 |
| 2 | 2 | ✗ | — | unchanged | 2 |
| 3 | 2 | ✗ | — | unchanged | 2 |
| 4 | 3 | ✓ | `nums[2]=3` | `[0,1,`**`3`**`,2,3,0,4,2]` | 3 |
| 5 | 0 | ✓ | `nums[3]=0` | `[0,1,3,`**`0`**`,3,0,4,2]` | 4 |
| 6 | 4 | ✓ | `nums[4]=4` | `[0,1,3,0,`**`4`**`,0,4,2]` | 5 |
| 7 | 2 | ✗ | — | unchanged | 5 |

Return **5**, `nums[:5] = [0,1,3,0,4]`.

Rows 0–1: `w == r`, so the write is a self-assignment. Harmless, not special-cased. `w` first falls behind at `r=2` — exactly when the first element is dropped. That's the `w = seen − dropped` invariant, visible.

**Complexity.** Time O(n), Space O(1).

**Edge cases.** `val` absent → `k == n`, array unchanged. All elements are `val` → `k == 0`. `[]` → 0.

**Alternative when removals are rare:** swap `nums[r]` with the last element and shrink — fewer writes, scrambles order. Legal here only because LC 27 doesn't require order.

---

### 5.8 Move Zeroes — LC 283 — Shape B1

**Problem.** Move all `0`s to the end **in place**, preserving relative order of non-zeros.

**Approach.** Read/write, but **swap instead of overwrite**. Overwriting loses the zeroes; swapping carries them to the back automatically, so no second pass to pad.

```python
class Solution:
    def moveZeroes(self, nums: list[int]) -> None:
        w = 0
        for r in range(len(nums)):
            if nums[r] != 0:
                nums[w], nums[r] = nums[r], nums[w]
                w += 1
```

```java
public void moveZeroes(int[] nums) {
    int w = 0;
    for (int r = 0; r < nums.length; r++)
        if (nums[r] != 0) { int t = nums[w]; nums[w] = nums[r]; nums[r] = t; w++; }
}
```

**Line notes.**
- Swap, not assign. `nums[w]` always holds either a zero or — when `w == r` — the element itself, so the swap is safe and self-cleaning.
- Order is preserved because non-zeros are written in scan order and never reordered among themselves.

**Dry run.** `nums = [0,1,0,3,12]`

| r | nums[r] | non-zero | swap | array after | w |
|---|---|---|---|---|---|
| 0 | 0 | ✗ | — | `[0,1,0,3,12]` | 0 |
| 1 | 1 | ✓ | `nums[0]↔nums[1]` | `[`**`1`**`,`**`0`**`,0,3,12]` | 1 |
| 2 | 0 | ✗ | — | unchanged | 1 |
| 3 | 3 | ✓ | `nums[1]↔nums[3]` | `[1,`**`3`**`,0,`**`0`**`,12]` | 2 |
| 4 | 12 | ✓ | `nums[2]↔nums[4]` | `[1,3,`**`12`**`,0,`**`0`**`]` | 3 |

Final `[1,3,12,0,0]`.

**Complexity.** Time O(n), Space O(1). Two writes per non-zero.

**Edge cases.** `[0]` → `[0]`. No zeroes → every swap is `nums[r]↔nums[r]`, a no-op. All zeroes → no swaps at all. *(Randomised, 400 cases — matches.)*

---

### 5.9 Sort Colors — LC 75 — Shape B1, three pointers

**Problem.** Array of `0`,`1`,`2`. Sort in place, one pass, no library sort. (Dutch National Flag.)

**Approach.** Three regions ⇒ three pointers. `low` = end of the 0s block, `mid` = scanner, `high` = start of the 2s block.

```
[ 0 0 0 | 1 1 1 | ? ? ? | 2 2 2 ]
         low     mid    high
```

**Algorithm.**
- `nums[mid] == 0` → swap with `low`, advance **both**.
- `nums[mid] == 1` → already placed, advance `mid`.
- `nums[mid] == 2` → swap with `high`, decrement `high`, **do not advance `mid`**.

```python
class Solution:
    def sortColors(self, nums: list[int]) -> None:
        low, mid, high = 0, 0, len(nums) - 1
        while mid <= high:
            if nums[mid] == 0:
                nums[low], nums[mid] = nums[mid], nums[low]
                low += 1; mid += 1
            elif nums[mid] == 1:
                mid += 1
            else:
                nums[mid], nums[high] = nums[high], nums[mid]
                high -= 1
```

```java
public void sortColors(int[] nums) {
    int low = 0, mid = 0, high = nums.length - 1;
    while (mid <= high) {
        if (nums[mid] == 0) { int t=nums[low]; nums[low]=nums[mid]; nums[mid]=t; low++; mid++; }
        else if (nums[mid] == 1) mid++;
        else { int t=nums[mid]; nums[mid]=nums[high]; nums[high]=t; high--; }
    }
}
```

**Line notes — the asymmetry IS the problem:**
- ⚠ Swapping with `high` pulls in an **unexamined** value from the right. Advancing `mid` would skip it unchecked. **Don't advance.**
- Swapping with `low` pulls in a value from the left that was **already examined** — it can only be a `1` (0s already pushed left, 2s already pushed right). Safe to advance.
- ⚠ `while mid <= high`, not `<` — when `mid == high` there's still one unexamined element.

**Dry run.** `nums = [2,0,2,1,1,0]`

| low | mid | high | nums[mid] | action | array after |
|---|---|---|---|---|---|
| 0 | 0 | 5 | 2 | swap mid↔high, `high→4` | `[0,0,2,1,1,`**`2`**`]` |
| 0 | 0 | 4 | 0 | swap low↔mid (self), `low→1`, `mid→1` | `[0,0,2,1,1,2]` |
| 1 | 1 | 4 | 0 | swap low↔mid (self), `low→2`, `mid→2` | `[0,0,2,1,1,2]` |
| 2 | 2 | 4 | 2 | swap mid↔high, `high→3` | `[0,0,`**`1`**`,1,`**`2`**`,2]` |
| 2 | 2 | 3 | 1 | `mid→3` | unchanged |
| 2 | 3 | 3 | 1 | `mid→4` | unchanged |

`mid > high` → `[0,0,1,1,2,2]`.

Row 1 is the payoff: after swapping in a `2`, `mid` stayed put and immediately re-examined the incoming `0`. Advancing would have stranded a `0` in the middle region.

**Complexity.** Time O(n) — `mid` and `high` together move `n` times. Space O(1). One pass beats counting sort's two.

**Edge cases.** `[]`, `[0]`, `[2]` → `high = -1` or `mid == high`; loop terminates correctly. *(Randomised vs `sorted()`, 600 cases — matches.)*

---

### 5.10 Middle of the Linked List — LC 876 — Shape B2

**Problem.** Return the middle node. Even length → the **second** middle.

**Approach.** Fast/slow at 2×. When fast falls off, slow is at the middle. One pass, no length count, O(1) space.

```python
class Solution:
    def middleNode(self, head: ListNode) -> ListNode:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        return slow
```

```java
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) { slow = slow.next; fast = fast.next.next; }
    return slow;
}
```

**Line notes.**
- ⚠ `while fast and fast.next` — both checks, in this order (see §4 B2).
- Want the **first** middle on even length? Loop on `fast.next and fast.next.next`.

**Dry run — odd.** `[1,2,3,4,5]`

| slow | fast | condition | action |
|---|---|---|---|
| 1 | 1 | `fast=1`, `fast.next=2` ✓ | slow→2, fast→3 |
| 2 | 3 | `fast=3`, `fast.next=4` ✓ | slow→3, fast→5 |
| 3 | 5 | `fast=5`, `fast.next=null` ✗ | exit |

Return node `3`. ✓

**Dry run — even.** `[1,2,3,4,5,6]`

| slow | fast | condition | action |
|---|---|---|---|
| 1 | 1 | ✓ | slow→2, fast→3 |
| 2 | 3 | ✓ | slow→3, fast→5 |
| 3 | 5 | `fast=5`, `fast.next=6` ✓ | slow→4, fast→null |
| 4 | null | ✗ | exit |

Return node `4` — the second middle. ✓

**Complexity.** Time O(n), Space O(1).

**Edge cases.** Single node → loop never runs, returns head. Two nodes → one iteration, returns the second.

---

### 5.11 Linked List Cycle — LC 141 — Shape B2

**Problem.** Does the list contain a cycle?

**Approach.** Floyd. See §4 B2 for why collision is forced.

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
```

```java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next; fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Line notes.**
- ⚠ Compare **after** moving. Both start at `head`; checking before the first move returns `True` for every list.
- ⚠ `is` (Python) / `==` (Java) compares **identity**, not `.val`.

**Dry run.** `[3,2,0,-4]`, tail → index 1 (the node holding `2`)

| step | slow | fast | equal? |
|---|---|---|---|
| start | 3 | 3 | — |
| 1 | 2 | 0 | no |
| 2 | 0 | 2 | no |
| 3 | -4 | -4 | **yes** → `True` |

**Complexity.** Time O(n), Space O(1). The hash-set alternative is O(n) space — that gap is the entire reason this trick is worth memorising.

**Edge cases.** `null` head → loop skipped, `False`. Single node, no cycle → `False`. Single node pointing at itself → step 1 puts both on it, `True`.

---

### 5.12 Linked List Cycle II — LC 142 — Shape B2

**Problem.** Return the node where the cycle **begins**, or `null`.

**Approach.** Detect the collision (§5.11), then reset one pointer to `head` and advance **both at speed 1**. They meet at the entrance.

**Why.** Let `a` = head→entrance, `b` = entrance→meeting point, `L` = cycle length. At collision slow has travelled `a + b` and fast `2(a + b)`; fast's surplus is a whole number of loops, so `a + b = kL`, hence `a = kL − b`. The distance head→entrance equals the distance meeting-point→entrance (mod `L`). Two pointers at speed 1 cover those equal distances and arrive together.

```python
class Solution:
    def detectCycle(self, head: ListNode) -> ListNode:
        slow = fast = head
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
            if slow is fast:
                p = head
                while p is not slow:
                    p = p.next
                    slow = slow.next
                return p
        return None
```

**Line notes.**
- ⚠ Phase 2 runs **inside** the `if`, using the collision point. Restarting the scan afterwards loses it.
- ⚠ Both move at **1** in phase 2. Keeping fast at 2 breaks the distance identity.
- `while p is not slow` — if the entrance *is* `head` (whole list is a loop), the body never runs and `head` is returned immediately. Correct.

**Dry run.** `[3,2,0,-4]`, tail → index 1. Here `a = 1`; the entrance is node `2`.

Phase 1 (from §5.11): collision at node `-4`.

Phase 2:

| p | slow | equal? |
|---|---|---|
| 3 (head) | -4 | no → advance both |
| 2 | 2 | **yes** → return node `2` |

Entrance is node `2`. ✓ *(Verified exhaustively for n = 1..8 × every cycle position including none.)*

**Complexity.** Time O(n), Space O(1).

**Edge cases.** No cycle → `None`. Entire list is a cycle (`a = 0`) → phase 2 returns `head` without looping. Self-loop single node → returns that node.

⚠ **Memorise the phase-2 move.** The proof is short but you don't want to derive it live.

---

### 5.13 Remove Nth Node From End — LC 19 — Shape B2, fixed gap

**Problem.** Remove the n-th node from the end, one pass.

**Approach.** Not 2× speed — **fixed gap**. Put `fast` `n` steps ahead, then advance both at 1. When `fast` reaches the last node, `slow` sits on the node *before* the target.

**Dummy node.** Removing the head is otherwise a special case (no predecessor to rewire). A dummy before `head` gives every real node a predecessor and deletes the special case entirely.

```python
class Solution:
    def removeNthFromEnd(self, head: ListNode, n: int) -> ListNode:
        dummy = ListNode(0, head)
        slow = fast = dummy
        for _ in range(n):
            fast = fast.next
        while fast.next:
            slow = slow.next
            fast = fast.next
        slow.next = slow.next.next
        return dummy.next
```

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0, head), slow = dummy, fast = dummy;
    for (int i = 0; i < n; i++) fast = fast.next;
    while (fast.next != null) { slow = slow.next; fast = fast.next; }
    slow.next = slow.next.next;
    return dummy.next;
}
```

**Line notes.**
- ⚠ Both start at `dummy`, not `head`. That one-node offset is what leaves `slow` on the **predecessor** rather than the target.
- ⚠ `while fast.next`, not `while fast` — stop **on** the last node, not past it.
- ⚠ `return dummy.next`, not `head` — if `head` was removed, `head` is a dangling reference to a deleted node.

**Dry run.** `[1,2,3,4,5]`, `n = 2`. List is `dummy→1→2→3→4→5`.

Phase 1 — advance `fast` 2 steps:

| step | fast |
|---|---|
| 0 | dummy |
| 1 | 1 |
| 2 | 2 |

Phase 2 — advance both until `fast.next` is null:

| slow | fast | `fast.next` | action |
|---|---|---|---|
| dummy | 2 | 3 | advance |
| 1 | 3 | 4 | advance |
| 2 | 4 | 5 | advance |
| 3 | 5 | null | stop |

`slow` is node `3`; `slow.next` is node `4` — the 2nd from the end. `slow.next = slow.next.next` → `3→5`.

Return `dummy.next` = `[1,2,3,5]`. ✓

**Complexity.** Time O(n), one pass. Space O(1).

**Edge cases** (verified):

| Input | n | Output | Why |
|---|---|---|---|
| `[1]` | 1 | `[]` | dummy makes head-removal ordinary |
| `[1,2]` | 2 | `[2]` | removes head |
| `[1,2]` | 1 | `[1]` | removes tail |
| `[1,2,3,4,5]` | 2 | `[1,2,3,5]` | middle |

---

### 5.14 Two-input variant — merge & intersect

*Signal: two sorted arrays, one output.* Same idea; the pointers just live in different arrays, and both advance by comparison.

```python
def merge_sorted(a, b):
    i = j = 0; out = []
    while i < len(a) and j < len(b):
        if a[i] <= b[j]: out.append(a[i]); i += 1
        else:            out.append(b[j]); j += 1
    out += a[i:]; out += b[j:]
    return out

def intersect_sorted(a, b):
    i = j = 0; out = []
    while i < len(a) and j < len(b):
        if a[i] == b[j]:  out.append(a[i]); i += 1; j += 1
        elif a[i] < b[j]: i += 1
        else:             j += 1
    return out
```

⚠ **LC 88** (merge `b` into `a`'s tail buffer) — **fill from the back.** Writing forward clobbers unread values in `a`; writing backward only touches space already consumed. Same `w ≤ r` safety argument, mirrored.

Unsorted intersection → hash set, O(n)/O(n) (Topic 1). Sorted → this, O(n)/O(1). Know which lever you're pulling.

---

## 6. When it is NOT Two Pointers

| Situation | Use instead | Why |
|---|---|---|
| Unsorted, need one pair | hash map (Topic 1) | sorting costs O(n log n) > map's O(n) |
| Need a running **summary** of the range between the pointers (sum / count / frequency) | **sliding window** (Topic 3) | two pointers reads only the two endpoints |
| Non-contiguous subsequence | DP | can't skip and remember |
| Can halve the space each step, not just shave it | binary search (Topic 4) | stronger monotonic condition available |
| No elimination argument exists | brute force / other | the precondition fails |

---

## 7. Drill Plan (Week 2)

Learn-then-drill, 25-min cap per problem, then the +3/+7/+21 re-solve schedule. Order is deliberate: **§5.7 before §5.6** — simple `keep` before backward-looking `keep`.

| Day | Problem | Shape | Difficulty |
|---|---|---|---|
| 1 | Valid Palindrome | A | Easy |
| 1 | Two Sum II | A | Medium |
| 2 | Remove Element | B1 | Easy |
| 2 | Remove Duplicates from Sorted Array | B1 | Easy |
| 3 | Move Zeroes | B1 | Easy |
| 3 | Container With Most Water | A | Medium |
| 4 | Middle of the Linked List | B2 | Easy |
| 4 | Linked List Cycle | B2 | Easy |
| 5 | 3Sum | A + outer loop | Medium |
| 5 | Sort Colors | B1 ×3 | Medium |
| 6 | Remove Nth Node From End | B2 fixed-gap | Medium |
| 6 | Linked List Cycle II | B2 | Medium |
| 7 | **Re-solve day:** all Day 1–4 problems blind, 15 min each | — | — |

Stretch (only if Day 1–6 were clean): **Trapping Rain Water** — the O(1)-space version, not the prefix-array one.

Java rep (during +7): redo Valid Palindrome and Sort Colors in Java.
Rust rep (Week 5+): Remove Element and Remove Duplicates in Rust — the `usize` underflow on `w - 1` will make the `w == 0` guard permanent knowledge.

---

## 8. Checkpoint (pass before Week 3)

You're done with this topic when, cold, you can:

1. Solve **Valid Palindrome + Remove Element** in under 10 min combined, bug-free first run.
2. State the **Container With Most Water** elimination argument in one sentence — why the shorter wall can be discarded.
3. Explain why in-place overwriting is safe, using the phrase **`w ≤ r`**.
4. Say what `w == 0` means without using the word "zero-value," and name what breaks without it in each of Python / Java / Rust.
5. Give both **3Sum dedup rules** and say why the anchor check compares backward (`i-1`), not forward.
6. State the **Floyd termination argument**: inside a cycle the gap shrinks by exactly 1 per step, so collision is forced.
7. Say why **`mid` doesn't advance** after a `high` swap in Dutch flag.
8. Given an unsorted array and "find a pair summing to target," say in 5 seconds whether you'd reach for two pointers or a hash map — **and why**.

Fail any → redo that item tomorrow, then move on. Reps over perfection; the checkpoint is the floor.

---

## 9. learndsa.org — Topic 1 coverage

| Site section | Covered by |
|---|---|
| Common Array Patterns → In-place Modification | §5.6, §5.7, §5.8 |
| Common Array Patterns → Traversing from Both Ends | §5.1–§5.5 |
| Two Pointer Technique → Opposite Directional | §5.2, §5.3 |
| Two Pointer Technique → Same Directional | §5.6, §5.7 |
| Two Pointer Technique → Partition (Dutch flag) | §5.9 |
| Common Problems → #4 Valid Palindrome | §5.1 |
| Practice Exercises → move zeros to end | §5.8 |
| Practice Exercises → merge two sorted arrays | §5.14 |
| Practice Exercises → intersection of two arrays | §5.14 |

**Still owed to Topic 1 (hashing):** two sum unsorted, valid anagram, first non-repeating char, all-unique-characters, permutation check, string compression.
**Topic 3 (sliding window):** longest substring without repeating characters, min size subarray sum.
**Topic 4 (binary search & prefix sums):** binary search, cyclic sort, missing number, prefix sums, rotate array.

---

## 10. Verification

Every Python and Java snippet in this file was **executed**, not just written.

- Deterministic: all 13 problems plus every edge case listed above.
- Randomised vs brute force: Container With Water (400), 3Sum (400), Trapping Rain Water (400), Move Zeroes (400), Sort Colors (600).
- Exhaustive: Cycle II for n = 1..8 × every cycle position including none.
- Rust snippets are reasoned, not executed — no `rustc` in the sandbox.

---

**Next topic:** Sliding Window (Week 3) — two pointers plus a *running summary* of everything between them. The new constraint: the summary must be incrementally updatable, and shrinking from the left must be able to restore validity.
