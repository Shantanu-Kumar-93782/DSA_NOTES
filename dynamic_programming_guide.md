# Dynamic Programming — The Complete Revision Guide 🧠

## Table of Contents

| # | Pattern | Key Problems |
|---|---|---|
| 1 | [The DP Thinking Framework](#1-the-dp-thinking-framework) | How to approach ANY DP problem |
| 2 | [1D DP](#2-1d-dp--linear-dp) | Fibonacci, Climbing Stairs, House Robber |
| 3 | [2D DP / Grid DP](#3-2d-dp--grid-dp) | Unique Paths, Min Path Sum |
| 4 | [Knapsack Family](#4-knapsack-family) | 0/1, Unbounded, Subset Sum, Coin Change |
| 5 | [Longest Increasing Subsequence](#5-longest-increasing-subsequence-lis) | LIS, Russian Dolls, Box Stacking |
| 6 | [String DP](#6-string-dp) | LCS, Edit Distance, Distinct Subsequences |
| 7 | [Interval DP](#7-interval-dp) | Burst Balloons, Matrix Chain, Palindrome Partition |
| 8 | [State Machine DP](#8-state-machine-dp) | All Buy & Sell Stock problems |
| 9 | [DP on Trees](#9-dp-on-trees) | House Robber III, Diameter, Max Path Sum |
| 10 | [DP with Bitmask](#10-dp-with-bitmask) | TSP, Assignment, Shortest Path Visiting All |
| 11 | [Digit DP](#11-digit-dp) | Count numbers with property in [L, R] |
| 12 | [Game Theory DP](#12-game-theory-dp) | Stone Game, Predict the Winner |
| 13 | [Partition DP](#13-partition-dp) | Palindrome Partitioning II, MCM |
| 14 | [Space Optimization](#14-space-optimization-techniques) | Rolling array, 1D from 2D |
| 15 | [Master Pattern Recognition](#15-master-pattern-recognition-cheat-sheet) | Decision tree for any DP problem |

---

## 1. The DP Thinking Framework

Every DP problem follows the same 5-step recipe. If you nail this, you can solve any DP problem.

### Step 1: Define the State
> "What information do I need to uniquely describe a subproblem?"

```
dp[i]       = answer considering elements 0..i
dp[i][j]    = answer considering first i items with capacity j
dp[i][j]    = answer for substring s[i..j]
dp[mask]    = answer when the set of visited nodes is 'mask'
dp[i][k]    = answer at position i with k transactions done
```

### Step 2: Write the Recurrence (Transition)
> "How does dp[current] relate to dp[smaller subproblems]?"

```
dp[i] = dp[i-1] + dp[i-2]                         // Fibonacci-like
dp[i] = max(dp[i-1], dp[i-2] + val[i])            // Take or skip
dp[i][j] = dp[i-1][j-1] + 1  if match             // LCS
dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt] + val)   // Knapsack
```

### Step 3: Identify the Base Case
> "What's the smallest subproblem I already know the answer to?"

```
dp[0] = 1        // one way to do nothing
dp[0][0] = 0     // zero items, zero weight = zero value
dp[i][i] = 1     // a single character is a palindrome of length 1
```

### Step 4: Determine the Order of Computation
> "Which subproblems need to be solved first?"

```
Bottom-up: small → large (most common: left to right, bottom to top)
Top-down:  large → small (recursion + memoization handles it automatically)
```

### Step 5: Extract the Answer
> "Where in my DP table is the final answer?"

```
dp[n]              // answer for the whole array
dp[n][W]           // answer for all items with full capacity
dp[0][n-1]         // answer for the entire string
max(dp[i])         // the best answer across all states
```

### Top-Down vs Bottom-Up

| | Top-Down (Memoization) | Bottom-Up (Tabulation) |
|---|---|---|
| **Style** | Recursive + cache | Iterative + table |
| **Pros** | Natural to think, only solves needed subproblems | No recursion overhead, easier to optimize space |
| **Cons** | Stack overflow risk, overhead of hashmap/recursion | Must figure out correct iteration order |
| **When to use** | Complex state spaces, tree DP, when you're unsure of order | Simple 1D/2D DP, when space optimization matters |

```java
// Top-Down Template
Map<String, Integer> memo = new HashMap<>();

int solve(int i, int j) {
    if (/* base case */) return /* base value */;
    String key = i + "," + j;
    if (memo.containsKey(key)) return memo.get(key);

    int result = /* recurrence using solve(smaller) */;
    memo.put(key, result);
    return result;
}

// Bottom-Up Template
int[][] dp = new int[n+1][m+1];
// fill base cases
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= m; j++) {
        dp[i][j] = /* recurrence using dp[smaller] */;
    }
}
return dp[n][m];
```

---

## 2. 1D DP / Linear DP

> **Pattern**: `dp[i]` depends on a few previous values like `dp[i-1]`, `dp[i-2]`, etc.

### Template — "Take or Skip" (House Robber Pattern)

```java
// LC #198 House Robber
// dp[i] = max money robbing houses 0..i
int rob(int[] nums) {
    int n = nums.length;
    if (n == 1) return nums[0];

    int prev2 = nums[0];                    // dp[i-2]
    int prev1 = Math.max(nums[0], nums[1]); // dp[i-1]

    for (int i = 2; i < n; i++) {
        int curr = Math.max(prev1,          // skip house i
                            prev2 + nums[i]); // rob house i
        prev2 = prev1;
        prev1 = curr;
    }

    return prev1;
}
```

### Template — "Count Ways" (Climbing Stairs Pattern)

```java
// LC #70 Climbing Stairs / LC #91 Decode Ways
// dp[i] = number of ways to reach step i
int climbStairs(int n) {
    if (n <= 2) return n;
    int prev2 = 1, prev1 = 2;

    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;  // can come from i-1 or i-2
        prev2 = prev1;
        prev1 = curr;
    }

    return prev1;
}
```

### Template — "Best ending at i" (Kadane's / Max Subarray)

```java
// LC #53 Maximum Subarray
// dp[i] = max subarray sum ENDING at index i
int maxSubArray(int[] nums) {
    int maxEndingHere = nums[0];
    int maxSoFar = nums[0];

    for (int i = 1; i < nums.length; i++) {
        maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
        // either start new subarray at i, or extend previous
        maxSoFar = Math.max(maxSoFar, maxEndingHere);
    }

    return maxSoFar;
}
```

### Common 1D DP Problems

| Problem | State | Recurrence |
|---|---|---|
| Fibonacci | `dp[i]` = ith fib | `dp[i-1] + dp[i-2]` |
| Climbing Stairs (LC #70) | `dp[i]` = ways to reach i | `dp[i-1] + dp[i-2]` |
| House Robber (LC #198) | `dp[i]` = max profit from 0..i | `max(dp[i-1], dp[i-2] + nums[i])` |
| Decode Ways (LC #91) | `dp[i]` = ways to decode s[0..i] | `dp[i-1] + dp[i-2]` (if valid) |
| Max Subarray (LC #53) | `dp[i]` = max sum ending at i | `max(nums[i], dp[i-1] + nums[i])` |
| Min Cost Climbing (LC #746) | `dp[i]` = min cost to reach i | `min(dp[i-1]+cost[i-1], dp[i-2]+cost[i-2])` |
| Jump Game (LC #55) | `dp[i]` = can reach i? | `any(dp[j] && j+nums[j] >= i)` |

---

## 3. 2D DP / Grid DP

> **Pattern**: `dp[i][j]` represents a subproblem on a grid or with two changing parameters.

### Template — Grid Paths

```java
// LC #62 Unique Paths
int uniquePaths(int m, int n) {
    int[][] dp = new int[m][n];

    // Base: first row and first column are all 1 (only one way to reach them)
    for (int i = 0; i < m; i++) dp[i][0] = 1;
    for (int j = 0; j < n; j++) dp[0][j] = 1;

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[i][j] = dp[i-1][j] + dp[i][j-1];  // from top + from left
        }
    }

    return dp[m-1][n-1];
}
```

### Template — Min/Max Path on Grid

```java
// LC #64 Minimum Path Sum
int minPathSum(int[][] grid) {
    int m = grid.length, n = grid[0].length;
    int[][] dp = new int[m][n];
    dp[0][0] = grid[0][0];

    for (int i = 1; i < m; i++) dp[i][0] = dp[i-1][0] + grid[i][0];
    for (int j = 1; j < n; j++) dp[0][j] = dp[0][j-1] + grid[0][j];

    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];

    return dp[m-1][n-1];
}
```

> [!TIP]
> **Grid DP direction rule**: If you can only move **right and down**, fill the DP table **left to right, top to bottom**. The iteration order always follows the direction of movement.

---

## 4. Knapsack Family

This is the **most important DP pattern family**. Almost every DP problem is a knapsack variant in disguise.

### 4.1 — 0/1 Knapsack (Each item used at most once)

> "Given items with weight and value, maximize value within capacity W."

```java
int knapsack01(int[] weights, int[] values, int W) {
    int n = weights.length;
    int[][] dp = new int[n + 1][W + 1];
    // dp[i][w] = max value using first i items with capacity w

    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i-1][w];                         // skip item i
            if (w >= weights[i-1])
                dp[i][w] = Math.max(dp[i][w],
                           dp[i-1][w - weights[i-1]] + values[i-1]); // take item i
        }
    }

    return dp[n][W];
}

// ⭐ Space-optimized (1D array — iterate W backwards!)
int knapsack01_optimized(int[] weights, int[] values, int W) {
    int[] dp = new int[W + 1];

    for (int i = 0; i < weights.length; i++) {
        for (int w = W; w >= weights[i]; w--) {    // ⚠️ BACKWARDS to prevent reuse
            dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
        }
    }

    return dp[W];
}
```

> [!CAUTION]
> **0/1 Knapsack space optimization**: Iterate capacity **backwards** (W → 0). Going forwards would let you "pick the same item twice" because `dp[w - wt]` would already be updated.

### 4.2 — Unbounded Knapsack (Each item used unlimited times)

```java
// Same as 0/1, but iterate capacity FORWARDS (allows reuse)
int unboundedKnapsack(int[] weights, int[] values, int W) {
    int[] dp = new int[W + 1];

    for (int i = 0; i < weights.length; i++) {
        for (int w = weights[i]; w <= W; w++) {    // ⚠️ FORWARDS allows reuse
            dp[w] = Math.max(dp[w], dp[w - weights[i]] + values[i]);
        }
    }

    return dp[W];
}
```

### 4.3 — Subset Sum (Can we make exact sum?)

```java
// LC #416 Partition Equal Subset Sum
boolean canPartition(int[] nums) {
    int total = Arrays.stream(nums).sum();
    if (total % 2 != 0) return false;
    int target = total / 2;

    boolean[] dp = new boolean[target + 1];
    dp[0] = true;  // sum 0 is always possible (empty subset)

    for (int num : nums) {
        for (int s = target; s >= num; s--) {    // backwards (0/1 knapsack)
            dp[s] = dp[s] || dp[s - num];
        }
    }

    return dp[target];
}
```

### 4.4 — Coin Change (Fewest coins / Count ways)

```java
// LC #322 Coin Change — Minimum coins (Unbounded Knapsack variant)
int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);  // impossibly large
    dp[0] = 0;

    for (int coin : coins) {
        for (int a = coin; a <= amount; a++) {     // forwards (unbounded)
            dp[a] = Math.min(dp[a], dp[a - coin] + 1);
        }
    }

    return dp[amount] > amount ? -1 : dp[amount];
}

// LC #518 Coin Change 2 — Count ways (order doesn't matter)
int change(int amount, int[] coins) {
    int[] dp = new int[amount + 1];
    dp[0] = 1;

    for (int coin : coins) {               // ⭐ coins in outer loop → combinations
        for (int a = coin; a <= amount; a++) {
            dp[a] += dp[a - coin];
        }
    }

    return dp[amount];
}

// LC #377 Combination Sum IV — Count ways (order matters = permutations)
int combinationSum4(int[] nums, int target) {
    int[] dp = new int[target + 1];
    dp[0] = 1;

    for (int t = 1; t <= target; t++) {     // ⭐ target in outer loop → permutations
        for (int num : nums) {
            if (t >= num) dp[t] += dp[t - num];
        }
    }

    return dp[target];
}
```

> [!IMPORTANT]
> **The loop order trick for counting**:
> - **Combinations** (order doesn't matter): items in **outer** loop → `{1,2}` and `{2,1}` counted once
> - **Permutations** (order matters): target in **outer** loop → `{1,2}` and `{2,1}` counted separately

### Knapsack Family Summary

| Variant | Item Use | Loop Direction | Example |
|---|---|---|---|
| 0/1 Knapsack | Once | **Backwards** | Subset Sum, Partition Equal (LC #416) |
| Unbounded | Unlimited | **Forwards** | Coin Change (LC #322, #518) |
| Subset Sum | Once | **Backwards** | Target Sum (LC #494) |
| Count Combinations | Unlimited | Items outer | Coin Change 2 (LC #518) |
| Count Permutations | Unlimited | Target outer | Combination Sum IV (LC #377) |

---

## 5. Longest Increasing Subsequence (LIS)

### Template — O(n²) DP

```java
// LC #300 Longest Increasing Subsequence
int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];  // dp[i] = LIS ending at index i
    Arrays.fill(dp, 1);     // every element is a subsequence of length 1

    int maxLen = 1;
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
        maxLen = Math.max(maxLen, dp[i]);
    }

    return maxLen;
}
```

### Template — O(n log n) with Binary Search ⭐

```java
int lengthOfLIS_fast(int[] nums) {
    List<Integer> tails = new ArrayList<>();
    // tails[i] = smallest tail element of all increasing subsequences of length i+1

    for (int num : nums) {
        int pos = Collections.binarySearch(tails, num);
        if (pos < 0) pos = -(pos + 1);   // insertion point

        if (pos == tails.size()) {
            tails.add(num);               // extends longest subsequence
        } else {
            tails.set(pos, num);          // replace to keep smallest tail
        }
    }

    return tails.size();
}
```

> [!TIP]
> The `tails` array is NOT the actual LIS — it's just a tool to compute the length. To reconstruct the LIS, you need to track parent pointers.

### LIS Variants

| Problem | Twist |
|---|---|
| LC #300 LIS | Standard |
| LC #354 Russian Doll Envelopes | Sort by width ↑, then LIS on height (sort height ↓ for same width!) |
| LC #1964 Longest Obstacle Course | LIS allowing equal elements (use `upper_bound`) |
| LC #673 Number of LIS | Track count along with length |

---

## 6. String DP

### 6.1 — Longest Common Subsequence (LCS)

```java
// LC #1143 Longest Common Subsequence
int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    // dp[i][j] = LCS of text1[0..i-1] and text2[0..j-1]

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1.charAt(i-1) == text2.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1] + 1;       // ⭐ match: extend
            } else {
                dp[i][j] = Math.max(dp[i-1][j],     // skip from text1
                                    dp[i][j-1]);     // skip from text2
            }
        }
    }

    return dp[m][n];
}
```

### 6.2 — Edit Distance (Levenshtein)

```java
// LC #72 Edit Distance
int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    // dp[i][j] = min ops to convert word1[0..i-1] to word2[0..j-1]

    // Base cases: converting to/from empty string
    for (int i = 0; i <= m; i++) dp[i][0] = i;  // delete all
    for (int j = 0; j <= n; j++) dp[0][j] = j;  // insert all

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i-1) == word2.charAt(j-1)) {
                dp[i][j] = dp[i-1][j-1];           // match: no operation
            } else {
                dp[i][j] = 1 + Math.min(dp[i-1][j-1],   // replace
                               Math.min(dp[i-1][j],       // delete
                                        dp[i][j-1]));     // insert
            }
        }
    }

    return dp[m][n];
}
```

### 6.3 — Longest Palindromic Subsequence (LPS)

```java
// LC #516 Longest Palindromic Subsequence
// Trick: LPS(s) = LCS(s, reverse(s))
// OR use interval DP:
int longestPalindromeSubseq(String s) {
    int n = s.length();
    int[][] dp = new int[n][n];
    // dp[i][j] = LPS in s[i..j]

    for (int i = 0; i < n; i++) dp[i][i] = 1;  // single char

    for (int len = 2; len <= n; len++) {           // ⚠️ iterate by length
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            if (s.charAt(i) == s.charAt(j)) {
                dp[i][j] = dp[i+1][j-1] + 2;      // both ends match
            } else {
                dp[i][j] = Math.max(dp[i+1][j],    // skip left
                                    dp[i][j-1]);    // skip right
            }
        }
    }

    return dp[0][n-1];
}
```

### 6.4 — Distinct Subsequences

```java
// LC #115 Distinct Subsequences
// How many subsequences of s equal t?
int numDistinct(String s, String t) {
    int m = s.length(), n = t.length();
    int[][] dp = new int[m + 1][n + 1];
    // dp[i][j] = ways to form t[0..j-1] from s[0..i-1]

    for (int i = 0; i <= m; i++) dp[i][0] = 1;  // empty t: one way (skip everything)

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            dp[i][j] = dp[i-1][j];              // skip s[i-1]
            if (s.charAt(i-1) == t.charAt(j-1))
                dp[i][j] += dp[i-1][j-1];       // use s[i-1] to match t[j-1]
        }
    }

    return dp[m][n];
}
```

### String DP Pattern Summary

| Problem | State | Key Recurrence |
|---|---|---|
| LCS (LC #1143) | `dp[i][j]` = LCS of s1[0..i], s2[0..j] | Match → `dp[i-1][j-1]+1`, else `max(dp[i-1][j], dp[i][j-1])` |
| Edit Dist (LC #72) | `dp[i][j]` = min ops s1[0..i] → s2[0..j] | Match → `dp[i-1][j-1]`, else `1 + min(replace, delete, insert)` |
| LPS (LC #516) | `dp[i][j]` = LPS in s[i..j] | Ends match → `dp[i+1][j-1]+2`, else `max(dp[i+1][j], dp[i][j-1])` |
| Distinct Subseq (LC #115) | `dp[i][j]` = ways t[0..j] in s[0..i] | Match → `dp[i-1][j-1] + dp[i-1][j]`, else `dp[i-1][j]` |

---

## 7. Interval DP

> **Pattern**: `dp[i][j]` = optimal answer for the subproblem on range `[i, j]`.
> **Key**: Iterate by **length** of interval, not by index.

### Template
```java
// General interval DP structure
for (int len = 2; len <= n; len++) {          // length of interval
    for (int i = 0; i <= n - len; i++) {      // start index
        int j = i + len - 1;                  // end index

        for (int k = i; k < j; k++) {         // split point
            dp[i][j] = best(dp[i][j],
                            dp[i][k] + dp[k+1][j] + cost(i, k, j));
        }
    }
}
```

### Burst Balloons (LC #312) ⭐

```java
int maxCoins(int[] nums) {
    int n = nums.length;
    int[] arr = new int[n + 2];  // pad with 1s on both sides
    arr[0] = arr[n + 1] = 1;
    for (int i = 0; i < n; i++) arr[i + 1] = nums[i];

    int[][] dp = new int[n + 2][n + 2];
    // dp[i][j] = max coins bursting all balloons in (i, j) exclusive

    for (int len = 1; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len + 1;
            for (int k = i + 1; k < j; k++) {   // k is the LAST balloon to burst
                dp[i][j] = Math.max(dp[i][j],
                    dp[i][k] + dp[k][j] + arr[i] * arr[k] * arr[j]);
            }
        }
    }

    return dp[0][n + 1];
}
```

> [!TIP]
> **Burst Balloons trick**: Think of `k` as the **last** balloon to burst in range (i, j), not the first. This makes the subproblems independent — the left and right subproblems don't affect each other.

### Matrix Chain Multiplication

```java
// Minimum cost to multiply matrices A1 × A2 × ... × An
// dims[i-1] × dims[i] = dimensions of matrix i
int matrixChain(int[] dims) {
    int n = dims.length - 1;  // number of matrices
    int[][] dp = new int[n][n];

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i; k < j; k++) {
                dp[i][j] = Math.min(dp[i][j],
                    dp[i][k] + dp[k+1][j] + dims[i] * dims[k+1] * dims[j+1]);
            }
        }
    }

    return dp[0][n-1];
}
```

---

## 8. State Machine DP

> **The pattern that solves ALL buy/sell stock problems with one framework.**

### The Universal State Machine

```
                buy              sell
  [NOT HOLDING] ───→ [HOLDING] ───→ [NOT HOLDING]
       ↑                                   │
       └───────────── cooldown ────────────┘
```

### States
```
dp[i][0] = max profit on day i, NOT holding stock
dp[i][1] = max profit on day i, HOLDING stock
```

### LC #121 — At Most 1 Transaction

```java
int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE;
    int maxProfit = 0;

    for (int price : prices) {
        minPrice = Math.min(minPrice, price);
        maxProfit = Math.max(maxProfit, price - minPrice);
    }

    return maxProfit;
}
```

### LC #122 — Unlimited Transactions

```java
int maxProfit(int[] prices) {
    int notHolding = 0;
    int holding = Integer.MIN_VALUE;

    for (int price : prices) {
        int prevNotHolding = notHolding;
        notHolding = Math.max(notHolding, holding + price);   // sell
        holding = Math.max(holding, prevNotHolding - price);   // buy
    }

    return notHolding;
}
```

### LC #309 — With Cooldown

```java
int maxProfit(int[] prices) {
    int notHolding = 0;
    int holding = Integer.MIN_VALUE;
    int cooldown = 0;

    for (int price : prices) {
        int prevNotHolding = notHolding;
        notHolding = Math.max(notHolding, cooldown);           // stay or exit cooldown
        cooldown = holding + price;                             // sell → enter cooldown
        holding = Math.max(holding, prevNotHolding - price);   // buy
    }

    return Math.max(notHolding, cooldown);
}
```

### LC #714 — With Transaction Fee

```java
int maxProfit(int[] prices, int fee) {
    int notHolding = 0;
    int holding = Integer.MIN_VALUE;

    for (int price : prices) {
        int prevNotHolding = notHolding;
        notHolding = Math.max(notHolding, holding + price - fee); // sell (pay fee)
        holding = Math.max(holding, prevNotHolding - price);       // buy
    }

    return notHolding;
}
```

### LC #123 / #188 — At Most K Transactions

```java
int maxProfit(int k, int[] prices) {
    if (k >= prices.length / 2) return unlimitedProfit(prices); // optimization

    int[][] dp = new int[k + 1][2];
    // dp[j][0] = max profit with j transactions done, not holding
    // dp[j][1] = max profit with j transactions done, holding
    for (int j = 0; j <= k; j++) dp[j][1] = Integer.MIN_VALUE;

    for (int price : prices) {
        for (int j = k; j >= 1; j--) {
            dp[j][0] = Math.max(dp[j][0], dp[j][1] + price);     // sell (completes txn j)
            dp[j][1] = Math.max(dp[j][1], dp[j-1][0] - price);   // buy (starts txn j)
        }
    }

    return dp[k][0];
}
```

### Stock Problems Summary

| Problem | Transactions | Extra Rule | Key Addition |
|---|---|---|---|
| LC #121 | 1 | - | Track min price |
| LC #122 | Unlimited | - | Two states: holding / not holding |
| LC #123 | At most 2 | - | Add transaction count dimension |
| LC #188 | At most k | - | Generalized #123 |
| LC #309 | Unlimited | 1-day cooldown | Add cooldown state |
| LC #714 | Unlimited | Fee per sell | Subtract fee on sell |

---

## 9. DP on Trees

> **Pattern**: DFS post-order. Compute DP values for children first, then combine at parent.

### Template

```java
int[] dp;

void dfs(int node, int parent, List<List<Integer>> tree) {
    for (int child : tree.get(node)) {
        if (child == parent) continue;
        dfs(child, node, tree);  // solve children first

        // Combine child's answer into node's answer
        dp[node] = combine(dp[node], dp[child]);
    }
}
```

### House Robber III (LC #337)

```java
// dp returns {maxWithout, maxWith} for each node
int rob(TreeNode root) {
    int[] result = dfs(root);
    return Math.max(result[0], result[1]);
}

int[] dfs(TreeNode node) {
    if (node == null) return new int[]{0, 0};

    int[] left = dfs(node.left);
    int[] right = dfs(node.right);

    // Rob this node: can't rob children
    int robThis = node.val + left[0] + right[0];

    // Skip this node: take best of each child
    int skipThis = Math.max(left[0], left[1]) + Math.max(right[0], right[1]);

    return new int[]{skipThis, robThis};
}
```

### Tree Diameter (LC #543)

```java
int diameter = 0;

int diameterOfBinaryTree(TreeNode root) {
    depth(root);
    return diameter;
}

int depth(TreeNode node) {
    if (node == null) return 0;

    int left = depth(node.left);
    int right = depth(node.right);

    diameter = Math.max(diameter, left + right);  // update global answer

    return 1 + Math.max(left, right);  // return height for parent
}
```

### Max Path Sum (LC #124)

```java
int maxSum = Integer.MIN_VALUE;

int maxPathSum(TreeNode root) {
    dfs(root);
    return maxSum;
}

int dfs(TreeNode node) {
    if (node == null) return 0;

    int left = Math.max(0, dfs(node.left));    // ignore negative paths
    int right = Math.max(0, dfs(node.right));

    maxSum = Math.max(maxSum, left + right + node.val);  // path through node

    return node.val + Math.max(left, right);   // best single path to return
}
```

> [!TIP]
> **Tree DP pattern**: The DFS returns what the **parent needs** (usually max single path / height), but internally updates a **global answer** with the best "two-branch" result at each node.

---

## 10. DP with Bitmask

> **When**: The state involves a **set** of items (visited nodes, selected elements), and n ≤ 20.
> **How**: Represent the set as a bitmask integer.

### Template

```java
// dp[mask] = answer when the set of selected items is represented by mask
int n = items.length;
int[] dp = new int[1 << n];  // 2^n states
Arrays.fill(dp, INF);
dp[0] = baseCase;

for (int mask = 0; mask < (1 << n); mask++) {
    if (dp[mask] == INF) continue;

    for (int i = 0; i < n; i++) {
        if ((mask & (1 << i)) != 0) continue;  // i already selected

        int newMask = mask | (1 << i);
        dp[newMask] = best(dp[newMask], dp[mask] + cost(mask, i));
    }
}
```

### Shortest Path Visiting All Nodes (LC #847)

```java
int shortestPathLength(int[][] graph) {
    int n = graph.length;
    int FULL = (1 << n) - 1;

    // BFS: state = (node, visited mask)
    int[][] dist = new int[n][1 << n];
    for (int[] row : dist) Arrays.fill(row, Integer.MAX_VALUE);

    Queue<int[]> queue = new LinkedList<>();
    for (int i = 0; i < n; i++) {
        queue.offer(new int[]{i, 1 << i});
        dist[i][1 << i] = 0;
    }

    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int u = curr[0], mask = curr[1];

        if (mask == FULL) return dist[u][mask];

        for (int v : graph[u]) {
            int newMask = mask | (1 << v);
            if (dist[u][mask] + 1 < dist[v][newMask]) {
                dist[v][newMask] = dist[u][mask] + 1;
                queue.offer(new int[]{v, newMask});
            }
        }
    }

    return -1;
}
```

### Bitmask Operations Cheat Sheet

| Operation | Code | Meaning |
|---|---|---|
| Check if bit i is set | `(mask >> i) & 1` | Is item i selected? |
| Set bit i | `mask \| (1 << i)` | Select item i |
| Clear bit i | `mask & ~(1 << i)` | Deselect item i |
| Toggle bit i | `mask ^ (1 << i)` | Flip item i |
| Count set bits | `Integer.bitCount(mask)` | How many selected |
| Lowest set bit | `mask & (-mask)` | Isolate rightmost 1 |
| Remove lowest set bit | `mask & (mask - 1)` | Clear rightmost 1 |
| All bits set for n items | `(1 << n) - 1` | All items selected |
| Iterate subsets of mask | `for(int s=mask; s>0; s=(s-1)&mask)` | All subsets |

---

## 11. Digit DP

> **Problem type**: "Count numbers in range [L, R] with some digit property."
> **Trick**: `count(R) - count(L-1)` converts range query into prefix query.

### Template

```java
// Count numbers from 0 to num with some property
// State: position, tight constraint, other parameters
char[] digits;
Integer[][][] memo;

int countUpTo(int num) {
    digits = String.valueOf(num).toCharArray();
    int n = digits.length;
    memo = new Integer[n][2][/* extra dims */];
    return solve(0, true, /* initial params */);
}

int solve(int pos, boolean tight, /* other params */) {
    if (pos == digits.length) {
        return /* is this a valid number? */ ? 1 : 0;
    }

    if (memo[pos][tight ? 1 : 0][/* params */] != null)
        return memo[pos][tight ? 1 : 0][/* params */];

    int limit = tight ? (digits[pos] - '0') : 9;  // ⭐ upper bound on digit
    int result = 0;

    for (int d = 0; d <= limit; d++) {
        result += solve(pos + 1,
                        tight && (d == limit),     // ⭐ tight only if we're at the limit
                        /* updated params */);
    }

    memo[pos][tight ? 1 : 0][/* params */] = result;
    return result;
}

// Answer for [L, R] = countUpTo(R) - countUpTo(L - 1)
```

> [!IMPORTANT]
> **The "tight" parameter**: If `tight = true`, the digit at this position can be at most `digits[pos]`. If `false`, any digit 0-9 is allowed. Once we place a digit strictly less than the limit, all subsequent positions become "free" (tight = false).

**LeetCode**: LC #233 Number of Digit One, LC #357 Count Numbers with Unique Digits, LC #902 Numbers At Most N Given Digit Set

---

## 12. Game Theory DP

> **Pattern**: Two players play optimally. "If I play optimally and my opponent plays optimally, what's the best I can do?"

### Template — Min-Max

```java
// dp[i][j] = best score difference (current player - opponent) for subarray [i..j]
int[][] dp = new int[n][n];

for (int i = 0; i < n; i++) dp[i][i] = nums[i];  // only one choice

for (int len = 2; len <= n; len++) {
    for (int i = 0; i <= n - len; i++) {
        int j = i + len - 1;
        dp[i][j] = Math.max(nums[i] - dp[i+1][j],   // take left
                             nums[j] - dp[i][j-1]);   // take right
    }
}

// dp[0][n-1] > 0 means first player wins
```

> [!TIP]
> **The insight**: `dp[i][j]` stores the **score difference** (current player minus opponent), not the absolute score. When the opponent plays, their best is subtracted because they're trying to maximize their own advantage.

### Stone Game (LC #877)

```java
boolean stoneGame(int[] piles) {
    int n = piles.length;
    int[][] dp = new int[n][n];

    for (int i = 0; i < n; i++) dp[i][i] = piles[i];

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            dp[i][j] = Math.max(piles[i] - dp[i+1][j],
                                 piles[j] - dp[i][j-1]);
        }
    }

    return dp[0][n-1] > 0;  // does player 1 have a positive advantage?
}
```

### Predict the Winner (LC #486)
Same code as Stone Game, but return `dp[0][n-1] >= 0` (ties allowed).

---

## 13. Partition DP

> **Pattern**: "What is the minimum cost to partition the array/string into valid segments?"

### Template

```java
// dp[i] = min cost to partition s[0..i-1]
int[] dp = new int[n + 1];
Arrays.fill(dp, Integer.MAX_VALUE);
dp[0] = 0;

for (int i = 1; i <= n; i++) {
    for (int j = 0; j < i; j++) {
        if (isValid(s, j, i - 1)) {  // is s[j..i-1] a valid segment?
            dp[i] = Math.min(dp[i], dp[j] + cost(j, i - 1));
        }
    }
}

return dp[n];
```

### Palindrome Partitioning II (LC #132)

```java
int minCut(String s) {
    int n = s.length();

    // Precompute: is s[i..j] a palindrome?
    boolean[][] isPalin = new boolean[n][n];
    for (int len = 1; len <= n; len++)
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            isPalin[i][j] = s.charAt(i) == s.charAt(j)
                && (len <= 2 || isPalin[i+1][j-1]);
        }

    // dp[i] = min cuts for s[0..i]
    int[] dp = new int[n];
    Arrays.fill(dp, Integer.MAX_VALUE);

    for (int i = 0; i < n; i++) {
        if (isPalin[0][i]) {
            dp[i] = 0;                  // whole prefix is palindrome
        } else {
            for (int j = 1; j <= i; j++) {
                if (isPalin[j][i]) {
                    dp[i] = Math.min(dp[i], dp[j-1] + 1);
                }
            }
        }
    }

    return dp[n - 1];
}
```

### Word Break (LC #139)

```java
boolean wordBreak(String s, List<String> wordDict) {
    Set<String> words = new HashSet<>(wordDict);
    int n = s.length();
    boolean[] dp = new boolean[n + 1];
    dp[0] = true;  // empty string

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && words.contains(s.substring(j, i))) {
                dp[i] = true;
                break;
            }
        }
    }

    return dp[n];
}
```

---

## 14. Space Optimization Techniques

### Technique 1: Rolling Array (2 rows → 2 variables)

```java
// If dp[i] only depends on dp[i-1] and dp[i-2]:
int prev2 = base0, prev1 = base1;
for (int i = 2; i <= n; i++) {
    int curr = f(prev1, prev2);
    prev2 = prev1;
    prev1 = curr;
}
// Answer: prev1
```

### Technique 2: 2D → 1D (Row compression)

```java
// If dp[i][j] only depends on dp[i-1][...]:
// Replace dp[i-1][j] with dp[j], process in correct order

// 0/1 Knapsack: iterate j BACKWARDS
for (int i = 0; i < n; i++)
    for (int j = W; j >= weight[i]; j--)  // ← backwards
        dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i]);

// LCS: need to save dp[i-1][j-1] before overwriting
for (int i = 1; i <= m; i++) {
    int prev = 0;  // saves dp[i-1][j-1]
    for (int j = 1; j <= n; j++) {
        int temp = dp[j];  // this will become dp[i-1][j-1] for next j
        if (s1.charAt(i-1) == s2.charAt(j-1))
            dp[j] = prev + 1;
        else
            dp[j] = Math.max(dp[j], dp[j-1]);
        prev = temp;
    }
}
```

### When Can You Optimize Space?

| Current DP depends on... | Optimization | Direction |
|---|---|---|
| `dp[i-1]` only | 1D array | Any direction |
| `dp[i-1]` and `dp[i-2]` | 2 variables | N/A |
| `dp[i-1][j]` and `dp[i][j-1]` | 1D array | Left to right |
| `dp[i-1][j]` and `dp[i-1][j-w]` (0/1) | 1D array | **Right to left** ⚠️ |
| `dp[i][j-w]` (unbounded) | 1D array | Left to right |
| `dp[i+1]` and `dp[i-1]` (interval) | ❌ Can't easily optimize | — |

---

## 15. Master Pattern Recognition Cheat Sheet

### "I see a DP problem. Which pattern is it?"

```
What does the problem ask?
│
├── Count ways / paths?
│   ├── Linear sequence → 1D DP (Climbing Stairs)
│   ├── Two strings → String DP (LCS, Distinct Subseq)
│   ├── Grid → Grid DP
│   └── Making a sum from items → Knapsack (Coin Change 2)
│
├── Optimize (min/max)?
│   ├── Take or skip items in sequence → 1D DP (House Robber)
│   ├── Select items with capacity → 0/1 Knapsack
│   ├── Partition into segments → Partition DP
│   ├── Merge / split ranges → Interval DP (Burst Balloons)
│   ├── Buy/sell with rules → State Machine DP
│   └── Visit all nodes → Bitmask DP (TSP)
│
├── Longest / shortest subsequence?
│   ├── Single sequence → LIS
│   ├── Two sequences → LCS
│   └── Palindromic → LPS (Interval DP on string)
│
├── Can we do it? (True/False)
│   ├── Make a sum → Subset Sum (0/1 Knapsack)
│   ├── Split string into words → Partition DP (Word Break)
│   └── Win a game → Game Theory DP
│
├── Count in range [L, R]?
│   └── Digit property → Digit DP
│
└── Optimal play / game?
    └── Both players optimal → Game Theory DP (Min-Max)
```

### Pattern → Template Quick Reference

| Pattern | State Shape | Time | Key Trick |
|---|---|---|---|
| **1D Linear** | `dp[i]` | O(n) | Often optimizable to O(1) space |
| **Grid** | `dp[i][j]` | O(m×n) | Fill in direction of movement |
| **0/1 Knapsack** | `dp[i][w]` → `dp[w]` | O(n×W) | Iterate capacity **backwards** |
| **Unbounded Knapsack** | `dp[w]` | O(n×W) | Iterate capacity **forwards** |
| **LIS** | `dp[i]` or tails[] | O(n²) or O(n log n) | Binary search on tails for O(n log n) |
| **LCS / Edit Dist** | `dp[i][j]` | O(m×n) | Match → diagonal, else max/min of neighbors |
| **Interval** | `dp[i][j]` length-based | O(n³) | Iterate by **length**, try all split points |
| **State Machine** | `dp[i][state]` | O(n×states) | Draw the state transition diagram first |
| **Tree DP** | `dp[node]` via DFS | O(n) | Post-order: solve children first |
| **Bitmask** | `dp[mask]` | O(n×2ⁿ) | n ≤ 20, represent sets as integers |
| **Digit** | `dp[pos][tight][...]` | O(digits × states) | `tight` flag limits digit choices |
| **Game Theory** | `dp[i][j]` | O(n²) | Store score **difference**, subtract on opponent's turn |
| **Partition** | `dp[i]` | O(n²) | Try all valid last segments ending at i |

---

## TL;DR — The 7 DP Commandments

1. **Define the state first.** If you can't explain what `dp[i]` means in one sentence, start over.
2. **0/1 Knapsack?** Iterate capacity **backwards**. Unbounded? **Forwards**. Mess this up and you'll get wrong answers silently.
3. **Combinations vs Permutations?** Items in outer loop = combinations. Target in outer loop = permutations.
4. **Interval DP?** Always iterate by **length**, not by start index.
5. **Stock problems?** Draw the state machine. States = {holding, not holding, cooldown}. Done.
6. **Two strings?** Build a 2D table. Match → come from diagonal. No match → come from top or left.
7. **Can't figure out the pattern?** Ask: "What's the last decision I make?" The answer reveals the recurrence.

Master these, and DP becomes pattern matching, not problem solving. 🎯
