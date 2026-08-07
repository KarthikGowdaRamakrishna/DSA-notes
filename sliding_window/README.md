# Topic 03 — Sliding Window

---

## The one idea

Two pointers looks at **the two endpoints**. A sliding window looks at **everything between them** — but never re-reads it.

The trick isn't the pointers. It's that you carry a **running summary** of the window (a sum, a count, a frequency map) and update it *incrementally*:

```
slide right → add one element to the summary
slide left  → remove one element from the summary
```

Recomputing the summary from scratch each time is O(k) per window → O(n·k) total. Updating it incrementally is O(1) per move → **O(n) total**.

That's the entire pattern. Everything below is bookkeeping.

---

## The precondition nobody states

Sliding window only works if **both** are true:

1. **The summary is incrementally updatable.** You can add on the right and remove on the left in O(1)-ish. Sum ✓. Count of distinct chars ✓. *Median* ✗ (that needs two heaps). *Max* ✗ in general — needs a monotonic deque, which is its own pattern.

2. **Shrinking from the left can restore validity.** If a window is invalid and removing the leftmost element can't possibly help, you can't slide — you'd have to restart. This is why sliding window needs **non-negative** numbers for sum problems. With negatives, shrinking can *increase* the sum, so "shrink until valid" never terminates correctly. That's a prefix-sum + hash map problem instead.

> If you only remember one thing from this file: **negatives kill sliding window on sum problems.** Interviewers use this as the trap.

---

## Two shapes

### Fixed size — window length `k` is given

Length never changes. Every step: drop one from the left, add one from the right.

Trigger words: *"of size k"*, *"every k-length block"*, *"max average of a subarray of length k"*.

### Dynamic size — window grows and shrinks

You're finding the longest / shortest range satisfying a condition.

Trigger words: *"longest ... such that"*, *"smallest ... with at least"*, *"at most K distinct"*.

---

## Templates

### Fixed — Python

```python
def fixed_window(nums, k):
    window = sum(nums[:k])           # build the first window
    best = window
    for r in range(k, len(nums)):
        window += nums[r] - nums[r-k]   # add new, drop old
        best = max(best, window)
    return best
```

### Fixed — Java

```java
int window = 0;
for (int i = 0; i < k; i++) window += nums[i];
int best = window;
for (int r = k; r < nums.length; r++) {
    window += nums[r] - nums[r - k];
    best = Math.max(best, window);
}
return best;
```

`r - k` is the index that just fell out. One line, no inner loop.

---

### Dynamic — and here is the split most people miss

**The two variants are not the same template.** Where you record the answer flips depending on whether you want the longest or the shortest.

#### Want the LONGEST valid window

Shrink *until valid*, then record. The window is valid at the moment you measure it.

```python
def longest_valid(s):
    l, best = 0, 0
    state = ...
    for r in range(len(s)):
        add(s[r], state)
        while not valid(state):        # invalid → shrink
            remove(s[l], state)
            l += 1
        best = max(best, r - l + 1)    # record AFTER the while
    return best
```

#### Want the SHORTEST valid window

Shrink *while still valid*, recording as you go — because the moment it stops being valid you've already passed the best one.

```python
def shortest_valid(nums, target):
    l, best = 0, float('inf')
    state = ...
    for r in range(len(nums)):
        add(nums[r], state)
        while valid(state):            # valid → record, then try smaller
            best = min(best, r - l + 1)
            remove(nums[l], state)
            l += 1
    return 0 if best == float('inf') else best
```

**Longest → record after the loop. Shortest → record inside the loop.** Burn that in; it's the single most common bug in this pattern.

---

### Why the total is O(n), not O(n²)

There's a `while` inside a `for`, which *looks* quadratic. It isn't: `l` only ever moves forward, and it can move at most `n` times across the entire run. So it's `n` right-moves + at most `n` left-moves = **O(2n) = O(n)**. Every element enters the window once and leaves once.

Say that out loud in an interview — it's the part people fumble.

---

## Worked problems

### 1. Maximum Sum Subarray of Size K — fixed

```python
def max_sum_k(nums, k):
    window = sum(nums[:k])
    best = window
    for r in range(k, len(nums)):
        window += nums[r] - nums[r-k]
        best = max(best, window)
    return best
```

**Max Average of Size K** is the identical function with `/ k` at the end. Don't divide inside the loop — you'd be doing n float divisions to answer one question.

---

### 2. Longest Substring Without Repeating Characters — dynamic, longest

```python
from collections import defaultdict

def length_of_longest_substring(s):
    count = defaultdict(int)
    l, best = 0, 0
    for r, ch in enumerate(s):
        count[ch] += 1
        while count[ch] > 1:            # only the NEW char can be the dup
            count[s[l]] -= 1
            l += 1
        best = max(best, r - l + 1)
    return best
```

**Why check only `count[ch] > 1` and not "is any count > 1"?** Because the window was valid before you added `s[r]`. The only possible violation is the character you just added. Checking one key instead of scanning the map is what keeps the inner work O(1).

**Alternative (last-seen-index jump):** store `char → last index` and jump `l` straight past the duplicate instead of stepping. Same O(n), fewer iterations, but you need `l = max(l, seen[ch] + 1)` — without the `max`, a stale index drags `l` *backwards* and silently breaks it. The counting version has no such trap; prefer it under pressure.

---

### 3. Longest Substring with At Most K Distinct Characters — dynamic, longest

The template with `len(count)` as the validity check.

```python
from collections import defaultdict

def longest_k_distinct(s, k):
    count = defaultdict(int)
    l, best = 0, 0
    for r, ch in enumerate(s):
        count[ch] += 1
        while len(count) > k:
            count[s[l]] -= 1
            if count[s[l]] == 0:
                del count[s[l]]          # MUST delete, or len() lies
            l += 1
        best = max(best, r - l + 1)
    return best
```

**The trap:** decrementing to zero isn't the same as removing. `len(count)` counts keys, not non-zero keys. Delete the key or your distinct-count is permanently wrong. (`Fruit Into Baskets` is this exact problem with `k = 2`.)

---

### 4. Longest Repeating Character Replacement — dynamic, longest, non-obvious validity

Window is valid if `window_length - count_of_most_frequent_char <= k` (i.e. the chars you'd have to replace fit within budget).

```python
from collections import defaultdict

def character_replacement(s, k):
    count = defaultdict(int)
    l, best, max_freq = 0, 0, 0
    for r, ch in enumerate(s):
        count[ch] += 1
        max_freq = max(max_freq, count[ch])
        while (r - l + 1) - max_freq > k:
            count[s[l]] -= 1
            l += 1
        best = max(best, r - l + 1)
    return best
```

**The famous confusion:** `max_freq` is never decreased when the window shrinks, so it can be stale (too high). Doesn't matter. A stale-high `max_freq` only makes the window look *more* valid, so the window never shrinks below the best size already found — and `best` is a running max, so it can't be corrupted. The window size becomes a non-shrinking high-water mark. Correct, and O(n).

You don't have to love this one. You do have to be able to state why it's safe.

---

### 5. Minimum Size Subarray Sum ≥ target — dynamic, **shortest**

Note the flipped template.

```python
def min_subarray_len(target, nums):
    l, total, best = 0, 0, float('inf')
    for r in range(len(nums)):
        total += nums[r]
        while total >= target:              # valid → record, then shrink
            best = min(best, r - l + 1)
            total -= nums[l]
            l += 1
    return 0 if best == float('inf') else best
```

**Requires non-negative `nums`.** With negatives, shrinking from the left can *raise* the sum, so "shrink while valid" doesn't converge. Negatives version → prefix sums + hash map (Topic 04). This is the precondition from the top of the file, in the wild.

---

### 6. Permutation in String / Find All Anagrams — fixed window + frequency match

Window size is fixed at `len(p)`. Question is whether the window's frequency map equals `p`'s.

```python
from collections import Counter

def find_anagrams(s, p):
    if len(p) > len(s): return []
    need, window = Counter(p), Counter(s[:len(p)])
    res = [0] if window == need else []
    for r in range(len(p), len(s)):
        window[s[r]] += 1
        left = s[r - len(p)]
        window[left] -= 1
        if window[left] == 0:
            del window[left]                # keep the dicts comparable
        if window == need:
            res.append(r - len(p) + 1)
    return res
```

`window == need` is O(26) for lowercase letters — a constant, so overall O(n). Say "O(1) because the alphabet is bounded" rather than hand-waving it.

`checkInclusion` (Permutation in String) is this function returning `True` on the first match.

---

### 7. Minimum Window Substring — dynamic, shortest, the hard one

Smallest window of `s` containing all chars of `t` (with multiplicity).

```python
from collections import Counter

def min_window(s, t):
    if not t or not s: return ""
    need = Counter(t)
    missing = len(t)                 # counts multiplicity, not distinct
    l, best = 0, (float('inf'), 0, 0)

    for r, ch in enumerate(s):
        if need[ch] > 0:
            missing -= 1             # only counts if still needed
        need[ch] -= 1                # can go negative = surplus

        while missing == 0:          # valid → record, then shrink
            if r - l + 1 < best[0]:
                best = (r - l + 1, l, r)
            need[s[l]] += 1
            if need[s[l]] > 0:       # crossed back into "needed"
                missing += 1
            l += 1

    return "" if best[0] == float('inf') else s[best[1]:best[2] + 1]
```

**The `missing` counter is the whole trick.** Naively you'd re-check the entire frequency map for validity every step — O(n·k). Collapsing validity into a single integer makes the check O(1).

**Why `need` is allowed to go negative:** a negative count means surplus. Surplus chars can be dropped from the left for free without breaking validity, and the `> 0` guards mean they don't touch `missing`. Letting it go negative is what makes the bookkeeping symmetric.

O(n) time, O(|t|) space.

---

## When it is *not* sliding window

- **Non-contiguous** subsequence → DP. Windows are contiguous by definition.
- **Sum problems with negative numbers** → prefix sum + hash map.
- **Max/min *of* the window** (not a count or sum) → monotonic deque (Sliding Window Maximum). The summary isn't O(1)-removable, so plain sliding window fails.
- **Median of the window** → two heaps.
- The window never needs to shrink → you didn't need a window, just a running variable (e.g. Kadane's).

---

## Recall card

```
build/extend right → update summary
                  ↓
   LONGEST:  while INVALID: shrink;  then record
   SHORTEST: while VALID:   record;  then shrink
```

| | trigger | summary | record where |
|---|---|---|---|
| **Fixed** | "of size k" | sum / freq | every step |
| **Dynamic, longest** | "longest ... such that" | freq map / distinct count | after the while |
| **Dynamic, shortest** | "smallest ... with at least" | running sum / missing counter | inside the while |

Four things to be able to say cold:

1. O(n) because `l` only moves forward — n right-moves + ≤ n left-moves.
2. Only the newly added element can break validity → check one key, not the whole map.
3. Deleting a zeroed key matters whenever `len(map)` is your validity check.
4. Sliding window needs non-negative values for sum problems; negatives → prefix sums.

---

## learndsa.org coverage

From **Topic 1 — Arrays and Strings**:

- Advanced Topics → Sliding Window Technique (Fixed Size, Variable Size) — fully covered
- Common Problems → #5 Longest Substring Without Repeating Characters — covered
- The `smallestSubarrayWithSum` implementation on the site — covered as #5 above, plus the negative-number caveat the site omits

Deferred to Topic 04 (Binary Search & Prefix Sums): prefix sums, range-sum queries, subarray-sum-equals-target with negatives, binary search, cyclic sort, missing number.
Owed to Topic 01 (Hashing): valid anagram, first non-repeating character, all-unique-characters, permutation check, string compression.
