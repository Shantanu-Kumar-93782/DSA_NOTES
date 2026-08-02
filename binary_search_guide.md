# Binary Search — The Only Framework You Need

## The Root Cause of Confusion

The confusion happens because there are **two fundamentally different templates**, and mixing pieces from one into the other creates bugs. Pick one, internalize it, and you'll never be confused again.

---

## Template 1: `while (lo <= hi)` — "Search for Exact Match"

> **Use when**: You need to find a specific target value or check if something exists.

```java
int binarySearch(int[] nums, int target) {
    int lo = 0, hi = nums.length - 1;

    while (lo <= hi) {                 // ← loop runs while search space is non-empty
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] == target) {
            return mid;                // ← found it, return immediately
        } else if (nums[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }

    return -1;                         // ← exhausted search space, not found
}
```

### Why `lo <= hi`?
The search space is `[lo, hi]` (both inclusive). When `lo == hi`, there's still **one element** left to check. Using `lo < hi` here would skip it.

### Why return `-1` outside?
If the loop ends, it means `lo > hi` — the search space is empty, and we never found the target.

### Mental model
> "Shrink the window until I find it or the window is empty."

---

## Template 2: `while (lo < hi)` — "Search for a Boundary / Condition"

> **Use when**: You need to find the **first/last** element satisfying a condition (lower bound, upper bound, minimum in rotated array, capacity problems, etc.)

```java
// Find the FIRST index where condition(mid) is true
int binarySearch(int[] nums) {
    int lo = 0, hi = nums.length - 1;

    while (lo < hi) {                  // ← loop runs until lo and hi CONVERGE
        int mid = lo + (hi - lo) / 2;

        if (condition(mid)) {
            hi = mid;                  // ← mid might be the answer, keep it
        } else {
            lo = mid + 1;             // ← mid is definitely NOT the answer, discard it
        }
    }

    return lo;                         // ← lo == hi, they converged to the answer
}
```

### Why `lo < hi`?
We're **not** looking for an exact match. We're narrowing down to a single candidate. When `lo == hi`, they've **converged** — that's our answer. No need to check further.

### Why return `lo` (or `hi`) outside?
The loop guarantees `lo == hi` at exit. That's the boundary we were searching for.

### Why `hi = mid` (not `hi = mid - 1`)?
Because `mid` **might be the answer**! If `condition(mid)` is true, we can't discard it. We can only say "the answer is at `mid` or to the left."

---

## The Decision Cheat Sheet

| Question | Template 1: `lo <= hi` | Template 2: `lo < hi` |
|---|---|---|
| **What am I searching for?** | An exact value | A boundary / first-last occurrence |
| **Loop condition** | `while (lo <= hi)` | `while (lo < hi)` |
| **Branches** | 3-way: `==`, `<`, `>` | 2-way: `condition` true/false |
| **How does mid move?** | `lo = mid+1` / `hi = mid-1` | `lo = mid+1` / `hi = mid` |
| **Where is the answer?** | Found inside the loop (return mid) | Found after the loop (return lo) |
| **Return on failure** | `-1` after loop | `lo` (may need a validity check) |

---

## Template 2 Variants

### Variant A: Find the **first** element where `condition` is true

```java
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;   // ← left-biased mid
    if (condition(mid)) {
        hi = mid;       // answer is at mid or LEFT of mid
    } else {
        lo = mid + 1;   // answer is RIGHT of mid
    }
}
return lo;
```

### Variant B: Find the **last** element where `condition` is true

```java
while (lo < hi) {
    int mid = lo + (hi - lo + 1) / 2;  // ← RIGHT-biased mid (⚠️ critical!)
    if (condition(mid)) {
        lo = mid;       // answer is at mid or RIGHT of mid
    } else {
        hi = mid - 1;   // answer is LEFT of mid
    }
}
return lo;
```

> [!CAUTION]
> In Variant B, you **must** use `mid = lo + (hi - lo + 1) / 2` (ceiling division).
> Otherwise `lo = mid` with a left-biased mid causes an **infinite loop** when `lo + 1 == hi`.

---

## The Golden Rule to Avoid Infinite Loops

| Assignment in branch | Mid formula |
|---|---|
| `lo = mid + 1` and `hi = mid` | `mid = lo + (hi - lo) / 2` (floor — default) |
| `lo = mid` and `hi = mid - 1` | `mid = lo + (hi - lo + 1) / 2` (ceiling) |

> **Rule**: If you ever write `lo = mid`, you MUST use ceiling mid. Otherwise infinite loop.

---

## Clean Return Patterns (No Messy If-Else!)

### ❌ What NOT to do:
```java
// Messy — return logic scattered everywhere
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] == target) {
        if (mid == 0 || nums[mid-1] != target) {
            return mid;
        } else {
            hi = mid - 1;
        }
    } else if (nums[mid] < target) {
        lo = mid + 1;
    } else {
        hi = mid - 1;
    }
}
return -1;
```

### ✅ What TO do — use Template 2 instead:
```java
// Clean — find lower bound of target
int lo = 0, hi = nums.length - 1;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] >= target) {   // condition: "is nums[mid] at least target?"
        hi = mid;
    } else {
        lo = mid + 1;
    }
}
// Single clean check at the end
return (lo < nums.length && nums[lo] == target) ? lo : -1;
```

> [!TIP]
> **The secret to clean code**: Move ALL result validation to a single check AFTER the loop.
> The loop's only job is to narrow down to one candidate. Then check that candidate once.

---

## Applying This to Real LeetCode Problems

### Problem 1: Search Insert Position (LC #35)
> Find where `target` would be inserted in a sorted array.

**Think**: "Find the **first** index where `nums[mid] >= target`" → Template 2A.

```java
int searchInsert(int[] nums, int target) {
    int lo = 0, hi = nums.length;  // hi = length because insert pos could be at end

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= target) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }

    return lo;  // clean, no if-else needed
}
```

---

### Problem 2: First and Last Position (LC #34)
> Find the first and last position of `target`.

**Think**: Two binary searches. Both use Template 2.

```java
int[] searchRange(int[] nums, int target) {
    if (nums.length == 0) return new int[]{-1, -1};

    // 1) Find FIRST occurrence: first index where nums[mid] >= target
    int lo = 0, hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= target) hi = mid;
        else lo = mid + 1;
    }
    if (nums[lo] != target) return new int[]{-1, -1};
    int first = lo;

    // 2) Find LAST occurrence: last index where nums[mid] <= target
    hi = nums.length - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo + 1) / 2;   // ⚠️ ceiling mid!
        if (nums[mid] <= target) lo = mid;
        else hi = mid - 1;
    }

    return new int[]{first, lo};
}
```

---

### Problem 3: Minimum in Rotated Sorted Array (LC #153)
> Find the minimum element in a rotated sorted array.

**Think**: "Find the **first** element that is `<= nums[last]`" → Template 2A.

```java
int findMin(int[] nums) {
    int lo = 0, hi = nums.length - 1;

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] <= nums[hi]) {  // mid is in the right (sorted) half
            hi = mid;                 // min is at mid or left
        } else {
            lo = mid + 1;             // min is right of mid
        }
    }

    return nums[lo];  // converged to the minimum
}
```

---

### Problem 4: Koko Eating Bananas (LC #875)
> Find the minimum speed `k` such that Koko can eat all bananas in `h` hours.

**Think**: "Find the **first** speed where `canFinish(speed, h)` is true" → Template 2A.

```java
int minEatingSpeed(int[] piles, int h) {
    int lo = 1, hi = max(piles);

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canFinish(piles, mid, h)) {  // can finish at this speed?
            hi = mid;                     // try slower
        } else {
            lo = mid + 1;                // need faster
        }
    }

    return lo;  // minimum valid speed
}
```

---

## Quick Reference — Which Template?

| Problem Type | Template | Key Condition |
|---|---|---|
| Find exact value | `lo <= hi` | `nums[mid] == target` |
| Lower bound / first occurrence | `lo < hi` | `nums[mid] >= target` → `hi = mid` |
| Upper bound / last occurrence | `lo < hi` | `nums[mid] <= target` → `lo = mid` (⚠️ ceiling mid) |
| Search insert position | `lo < hi` | `nums[mid] >= target` → `hi = mid` |
| Min in rotated array | `lo < hi` | `nums[mid] <= nums[hi]` → `hi = mid` |
| Capacity / speed / min-max problems | `lo < hi` | `feasible(mid)` → `hi = mid` |
| Peak element | `lo < hi` | `nums[mid] > nums[mid+1]` → `hi = mid` |

---

## TL;DR — The 3 Rules

1. **Exact match?** → `while (lo <= hi)`, return inside loop, return `-1` after.
2. **Finding a boundary?** → `while (lo < hi)`, return `lo` after loop.
3. **Writing `lo = mid`?** → Use ceiling mid: `lo + (hi - lo + 1) / 2`. Always.

Master these three rules and you'll never second-guess a binary search again. 🎯
