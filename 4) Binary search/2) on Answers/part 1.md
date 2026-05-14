## Resources

- [Sqrt(x) — Striver (YouTube)](https://youtu.be/Bsv3FPUX_BA)
- [Nth Root of M (YouTube)](https://youtu.be/rjEJeYCasHs)
- [Koko Eating Bananas (YouTube)](https://youtu.be/qyfekrNni90)
- [Minimum Days to Make M Bouquets (YouTube)](https://youtu.be/TXAuxeYBTdg)
- [Smallest Divisor Given Threshold (YouTube)](https://youtu.be/UvBKTVaG6U8)
- [Capacity to Ship Packages (YouTube)](https://youtu.be/MG-Ac4TAvTY)
- [Kth Missing Positive Number (YouTube)](https://youtu.be/uZ0N_hZpyps)
- [LeetCode 69 — Sqrt(x)](https://leetcode.com/problems/sqrtx)
- [GFG — Nth Root of M](https://www.geeksforgeeks.org/problems/find-nth-root-of-m5843/1)
- [LeetCode 875 — Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas)
- [LeetCode 1482 — Minimum Days to Make M Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets)
- [LeetCode 1283 — Find the Smallest Divisor Given a Threshold](https://leetcode.com/problems/find-the-smallest-divisor-given-a-threshold)
- [LeetCode 1011 — Capacity to Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days)
- [LeetCode 1539 — Kth Missing Positive Number](https://leetcode.com/problems/kth-missing-positive-number)

---

## Table of Contents

1. [The Core Mental Model](#1-the-core-mental-model)
2. [The Universal Template](#2-the-universal-template)
3. [Sqrt(x) — Integer Square Root (LC 69)](#3-sqrtx--integer-square-root-lc-69)
4. [Nth Root of M](#4-nth-root-of-m)
5. [Koko Eating Bananas (LC 875)](#5-koko-eating-bananas-lc-875)
6. [Minimum Days to Make M Bouquets (LC 1482)](#6-minimum-days-to-make-m-bouquets-lc-1482)
7. [Find the Smallest Divisor Given a Threshold (LC 1283)](#7-find-the-smallest-divisor-given-a-threshold-lc-1283)
8. [Capacity to Ship Packages Within D Days (LC 1011)](#8-capacity-to-ship-packages-within-d-days-lc-1011)
9. [Kth Missing Positive Number (LC 1539)](#9-kth-missing-positive-number-lc-1539)
10. [How to Identify a Binary Search on Answer Problem](#10-how-to-identify-a-binary-search-on-answer-problem)
11. [Common Mistakes and Pitfalls](#11-common-mistakes-and-pitfalls)
12. [Complexity Cheat Sheet](#12-complexity-cheat-sheet)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. The Core Mental Model

Classical binary search finds a **target value in a sorted array**. Binary search on the answer is a generalization: the thing being searched is an implicit ordered range of **possible answer values**, and the comparison is a **feasibility check** instead of equality.

### The Two Conditions

**Condition 1 — Monotonicity of feasibility.** There exists a threshold T such that:
- Every answer value below T is infeasible (cannot satisfy the constraints).
- Every answer value at or above T is feasible (can satisfy the constraints).

This creates a boolean sequence over the answer space:

```
[F, F, F, ..., F, T, T, T, ..., T]   ← find the leftmost T (minimum feasible)
[T, T, T, ..., T, F, F, F, ..., F]   ← find the rightmost T (maximum feasible)
```

**Condition 2 — Verifiable in polynomial time.** Given a candidate answer `mid`, you can efficiently check whether `mid` is feasible — typically in O(n).

### The Conceptual Leap

```
Classical binary search: find value x in sorted array
Binary search on answer: find minimum (or maximum) value m in [lo, hi]
                         such that feasible(m) = true
```

The answer space is always a concrete range with deterministic bounds. Every problem here has lo and hi derivable directly from the problem constraints — not guessed.

### Answer Spaces at a Glance

| Problem | Answer variable | lo | hi |
|---|---|---|---|
| Sqrt(x) | floor(sqrt(x)) | 1 | x |
| Nth Root of M | integer nth root | 1 | m |
| Koko Eating Bananas | minimum speed | 1 | max(piles) |
| Min Days Bouquets | minimum days | 1 | max(bloomDay) |
| Smallest Divisor | minimum divisor | 1 | max(nums) |
| Ship Packages | minimum capacity | max(weights) | sum(weights) |
| Kth Missing (binary) | answer index | 0 | n-1 |

---

## 2. The Universal Template

All seven problems use the same binary search skeleton. Only the feasibility function and the answer range change.

### Template: Find Minimum Feasible Value

```cpp
int binarySearchOnAnswer(/* problem parameters */) {
    int lo = /* minimum possible answer */;
    int hi = /* maximum possible answer */;
    int answer = hi;   // default to worst case

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;   // overflow-safe: never use (lo+hi)/2

        if (feasible(mid)) {
            answer = mid;      // mid works; try something smaller
            hi = mid - 1;
        } else {
            lo = mid + 1;      // mid does not work; need larger
        }
    }

    return answer;
}
```

### Template: Find Maximum Feasible Value

```cpp
int binarySearchOnAnswerMax(/* problem parameters */) {
    int lo = /* minimum possible answer */;
    int hi = /* maximum possible answer */;
    int answer = lo;   // default to worst case

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (feasible(mid)) {
            answer = mid;      // mid works; try something larger
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }

    return answer;
}
```

### Why `lo + (hi - lo) / 2` Not `(lo + hi) / 2`

When lo and hi are both close to INT_MAX (e.g., sum-of-weights problems), `lo + hi` overflows a 32-bit integer. `lo + (hi - lo) / 2` is algebraically identical but overflow-safe.

```
lo = 2*10^9, hi = 2*10^9 + 1
lo + hi = 4*10^9 + 1  → overflows int (INT_MAX ≈ 2.1*10^9)
lo + (hi - lo) / 2 = 2*10^9 + 0  → safe
```

### Designing the `feasible()` Function

The hardest part of each problem is writing the correct feasibility check:

```cpp
bool feasible(int candidate, /* other params */) {
    // Simulate: given this candidate answer, can the problem constraints be satisfied?
    // Return true if yes, false if no.
    // This function is typically O(n).
}
```

---

## 3. Sqrt(x) — Integer Square Root (LC 69)

### Problem Statement

Return `floor(sqrt(x))` — the largest integer `r` such that `r * r <= x`. Do not use built-in sqrt functions.

```
x=4   → 2   (2²=4=x exactly)
x=8   → 2   (2²=4≤8, 3²=9>8)
x=0   → 0
x=1   → 1
```

### Why Binary Search Applies

`f(m) = m²` is monotonically increasing. We want the **largest m** such that `m² ≤ x` — the "find maximum feasible" pattern where `feasible(m)` means `m² ≤ x`.

```
For x=8: feasible(1)=T, feasible(2)=T, feasible(3)=F → answer=2
```

### Approach 1: Linear Scan — O(sqrt(x))

```cpp
int mySqrt(int x) {
    if (x == 0) return 0;
    long long i = 1;
    while (i * i <= x) i++;
    return (int)(i - 1);
}
```

### Approach 2: Binary Search — O(log x)

```cpp
int mySqrt(int x) {
    if (x == 0) return 0;
    long long lo = 1, hi = x, answer = 1;

    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (mid * mid <= (long long)x) {
            answer = mid;      // mid feasible; try larger
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }

    return (int)answer;
}
// Time: O(log x), Space: O(1)
```

### Dry Run (x = 8)

```
lo=1, hi=8, answer=1

mid=4: 16 > 8 → hi=3
mid=2: 4 <= 8 → answer=2, lo=3
mid=3: 9 > 8  → hi=2

lo(3) > hi(2) → stop. Return 2  ✓
```

### Dry Run (x = 25)

```
lo=1, hi=25

mid=13: 169>25 → hi=12
mid=6:  36>25  → hi=5
mid=3:  9<=25  → answer=3, lo=4
mid=4:  16<=25 → answer=4, lo=5
mid=5:  25<=25 → answer=5, lo=6

lo(6) > hi(5) → stop. Return 5  ✓
```

### Approach 3: Newton-Raphson — O(log log x)

Newton's method (Babylonian/Heron's method) for integer sqrt. Each iteration roughly doubles the number of correct digits — quadratic convergence.

```cpp
int mySqrt(int x) {
    if (x == 0) return 0;
    long long r = x;
    while (r > x / r) {        // overflow-safe: equivalent to r*r > x
        r = (r + x / r) / 2;   // integer division at each step
    }
    return (int)r;
}
// Time: O(log log x) — quadratic convergence, Space: O(1)
```

**Why `r > x / r` instead of `r * r > x`:** Avoids overflow when r starts near INT_MAX.

### Edge Cases

| x | Output | Note |
|---|---|---|
| `0` | `0` | Special case |
| `1` | `1` | Perfect square |
| `2` | `1` | floor(1.414) = 1 |
| `INT_MAX` | `46340` | 46340² < INT_MAX < 46341² |

---

## 4. Nth Root of M

### Problem Statement

Find the integer nth root of m. Return it if it exists exactly; return -1 if no integer solution.

```
n=2, m=9  → 3   (3²=9)
n=3, m=27 → 3   (3³=27)
n=2, m=5  → -1  (√5 is irrational)
n=3, m=8  → 2   (2³=8)
```

**Key difference from Sqrt:** Return -1 if no exact integer root exists (unlike floor sqrt).

### The Power Function with Overflow Guard

Computing `mid^n` naively overflows for large `mid` and `n`. Add an early exit when the result exceeds `m`.

```cpp
// Returns: 1 if mid^n == m exactly
//          0 if mid^n < m
//         -1 if mid^n > m (overflow or excess)
int checkPow(long long mid, int n, long long m) {
    long long result = 1;
    for (int i = 0; i < n; i++) {
        result *= mid;
        if (result > m) return -1;   // early exit: already exceeds m
    }
    return (result == m) ? 1 : 0;
}
```

Without the early exit, `result` overflows `long long` for large `mid` and `n`, giving incorrect comparisons.

### Optimal Approach — Binary Search

```cpp
int nthRoot(int n, int m) {
    int lo = 1, hi = m;

    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        int check = checkPow(mid, n, (long long)m);

        if (check == 1) return (int)mid;    // exact root found
        else if (check == 0) lo = mid + 1;  // mid^n < m → need larger
        else hi = mid - 1;                  // mid^n > m → need smaller
    }

    return -1;   // no integer nth root exists
}
// Time: O(n log m), Space: O(1)
```

### Dry Run (n=3, m=27)

```
lo=1, hi=27

mid=14: 14^3=2744>27 → hi=13
mid=7:  7^3=343>27   → hi=6
mid=3:  3^3=27==27   → return 3  ✓
```

### Dry Run (n=2, m=5)

```
lo=1, hi=5

mid=3: 9>5  → hi=2
mid=1: 1<5  → lo=2
mid=2: 4<5  → lo=3

lo(3) > hi(2) → stop. Return -1  ✓
```

### Edge Cases

| n | m | Output | Note |
|---|---|---|---|
| 1 | 10 | 10 | 1st root of anything is itself |
| 2 | 1 | 1 | 1²=1 |
| 3 | 9 | -1 | ∛9 ≈ 2.08, not integer |

---

## 5. Koko Eating Bananas (LC 875)

### Problem Statement

There are n piles of bananas. Guards return in `h` hours. Each hour, Koko picks one pile and eats up to `k` bananas from it. Find the **minimum integer eating speed `k`** such that she finishes all bananas within `h` hours.

```
piles=[3,6,7,11], h=8   → 4
piles=[30,11,23,4,20], h=5 → 30
```

### Why Binary Search Applies

Answer space: `[1, max(piles)]`. Feasibility is monotonic: if speed `k` works, any speed `k' > k` also works. Pattern: `[F,...,F,T,...,T]`. Find **leftmost T** (minimum feasible speed).

### Hours to Eat a Pile at Speed k

```
ceil(pile / k) = (pile + k - 1) / k    ← integer arithmetic, no floating point needed
```

### Feasibility Check

```cpp
bool canFinish(vector<int>& piles, int k, int h) {
    long long hours = 0;
    for (int p : piles) {
        hours += (p + k - 1) / k;    // ceil(p / k)
        if (hours > h) return false;  // early exit
    }
    return hours <= h;
}
```

### Brute Force — O(max(piles) × n)

Try every speed from 1 upward. Return the first that works.

### Optimal — Binary Search O(n log max(piles))

```cpp
int minEatingSpeed(vector<int>& piles, int h) {
    int lo = 1;
    int hi = *max_element(piles.begin(), piles.end());
    int answer = hi;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (canFinish(piles, mid, h)) {
            answer = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }

    return answer;
}
```

### Dry Run

```
piles=[3,6,7,11], h=8
lo=1, hi=11, answer=11

mid=6: hours = ceil(3/6)+ceil(6/6)+ceil(7/6)+ceil(11/6) = 1+1+2+2 = 6 ≤ 8 ✓
  answer=6, hi=5

mid=3: hours = 1+2+3+4 = 10 > 8 ✗ → lo=4

mid=4: hours = 1+2+2+3 = 8 ≤ 8 ✓
  answer=4, hi=3

lo(4) > hi(3) → stop. Return 4  ✓
```

### Why hi = max(piles) Is Sufficient

At speed `max(piles)`, each pile takes exactly 1 hour. With n piles, she finishes in exactly n hours. Since `h ≥ n` is always implied (otherwise impossible), this is always feasible. No larger speed is ever needed.

### Edge Cases

| piles | h | Output | Note |
|---|---|---|---|
| `[1,1,1,1]` | `4` | `1` | Minimum speed 1 suffices |
| `[30]` | `1` | `30` | Must eat entire pile in 1 hour |

---

## 6. Minimum Days to Make M Bouquets (LC 1482)

### Problem Statement

`bloomDay[i]` is the day flower `i` will bloom. Make `m` bouquets, each needing `k` **adjacent** bloomed flowers. Find the **minimum number of days** needed. Return -1 if impossible.

```
bloomDay=[1,10,3,10,2], m=3, k=1   → 3
bloomDay=[1,10,3,10,2], m=3, k=2   → -1
bloomDay=[7,7,7,7,12,7,7], m=2, k=3 → 12
```

### Impossibility Check (Do This First)

Total flowers needed = `m * k`. If `m * k > n`, impossible regardless of days.

```cpp
if ((long long)m * k > (int)bloomDay.size()) return -1;
```

Use `long long` — `m * k` overflows `int` when both are large.

### Why Binary Search Applies

Answer space: `[1, max(bloomDay)]`. More days → more flowers bloomed → easier to form bouquets. Pattern: `[F,...,T,T]`. Find **minimum T**.

### Feasibility Check

Scan the array. Count consecutive flowers bloomed by day `d`. Every `k` consecutive gives one bouquet. Reset count when a non-bloomed flower is encountered.

```cpp
bool canMakeBouquets(vector<int>& bloomDay, int m, int k, int day) {
    int bouquets = 0, consecutive = 0;
    for (int b : bloomDay) {
        if (b <= day) {
            consecutive++;
            if (consecutive == k) { bouquets++; consecutive = 0; }
        } else {
            consecutive = 0;
        }
        if (bouquets >= m) return true;
    }
    return bouquets >= m;
}
```

### Optimal — Binary Search

```cpp
int minDays(vector<int>& bloomDay, int m, int k) {
    if ((long long)m * k > (int)bloomDay.size()) return -1;

    int lo = 1;
    int hi = *max_element(bloomDay.begin(), bloomDay.end());
    int answer = hi;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (canMakeBouquets(bloomDay, m, k, mid)) {
            answer = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }

    return answer;
}
// Time: O(n log max(bloomDay)), Space: O(1)
```

### Dry Run

```
bloomDay=[7,7,7,7,12,7,7], m=2, k=3
n=7, m*k=6 ≤ 7 → possible.
lo=1, hi=12, answer=12.

mid=6:  No flowers bloom (all ≥ 7 > 6). 0 bouquets. ✗ → lo=7.
mid=9:  7✓,7✓,7✓ → bouq=1. 7✓,12✗→reset. 7✓,7✓ → consecutive=2.
        bouquets=1 < 2. ✗ → lo=10.
mid=11: 7✓,7✓,7✓ → bouq=1. 7✓,12✗→reset. 7✓,7✓ → consecutive=2.
        bouquets=1 < 2. ✗ → lo=12.
mid=12: 7✓,7✓,7✓ → bouq=1. 7✓,12✓,7✓ → consecutive=3, bouq=2 ≥ 2. ✓
        answer=12, hi=11.

lo(12) > hi(11) → stop. Return 12  ✓
```

---

## 7. Find the Smallest Divisor Given a Threshold (LC 1283)

### Problem Statement

Find the **smallest positive integer divisor `d`** such that `sum(ceil(nums[i] / d)) ≤ threshold`.

```
nums=[1,2,5,9], threshold=6
  d=5: ceil(1/5)+ceil(2/5)+ceil(5/5)+ceil(9/5) = 1+1+1+2 = 5 ≤ 6 ✓
  d=4: 1+1+2+3 = 7 > 6 ✗
  Answer: 5
```

### Why Binary Search Applies

As `d` increases, each `ceil(nums[i]/d)` decreases (or stays the same). Total sum decreases as d increases. Pattern: `[F,...,T,T]`. Find **minimum T**.

**Answer space:** `[1, max(nums)]`. At d = max(nums), every ceil = 1, total sum = n ≤ threshold (guaranteed by problem).

### Optimal — Binary Search

```cpp
bool isFeasible(vector<int>& nums, int d, int threshold) {
    long long sum = 0;
    for (int x : nums) {
        sum += (x + d - 1) / d;   // ceil(x / d)
        if (sum > threshold) return false;
    }
    return sum <= threshold;
}

int smallestDivisor(vector<int>& nums, int threshold) {
    int lo = 1;
    int hi = *max_element(nums.begin(), nums.end());
    int answer = hi;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (isFeasible(nums, mid, threshold)) {
            answer = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }

    return answer;
}
// Time: O(n log max(nums)), Space: O(1)
```

### Dry Run

```
nums=[1,2,5,9], threshold=6
lo=1, hi=9, answer=9

mid=5: 1+1+1+2=5 ≤ 6 ✓  → answer=5, hi=4
mid=2: 1+1+3+5=10 > 6 ✗ → lo=3
mid=3: 1+1+2+3=7 > 6 ✗  → lo=4
mid=4: 1+1+2+3=7 > 6 ✗  → lo=5

lo(5) > hi(4) → stop. Return 5  ✓
```

---

## 8. Capacity to Ship Packages Within D Days (LC 1011)

### Problem Statement

Find the **minimum ship capacity** to ship all packages (in order) within `days` days. Packages must be loaded contiguously each day.

```
weights=[1,2,3,4,5,6,7,8,9,10], days=5  → 15
weights=[3,2,2,4,1,4],           days=3  → 6
```

### Answer Space

- **lo = max(weights):** Any capacity less than some package's weight means that package can never be loaded.
- **hi = sum(weights):** With this capacity, ship everything in 1 day.

### Why Binary Search Applies

Larger capacity → fewer days needed. Pattern: `[F,...,T,T]`. Find **minimum T**.

### Feasibility Check — Greedy Packing

Greedily fill each day: add packages until the next would exceed capacity; then start a new day.

```cpp
bool canShip(vector<int>& weights, int cap, int days) {
    int daysNeeded = 1, currentLoad = 0;
    for (int w : weights) {
        if (currentLoad + w > cap) {
            daysNeeded++;
            currentLoad = 0;
        }
        currentLoad += w;
        if (daysNeeded > days) return false;   // early exit
    }
    return daysNeeded <= days;
}
```

### Optimal — Binary Search

```cpp
int shipWithinDays(vector<int>& weights, int days) {
    int lo = *max_element(weights.begin(), weights.end());
    int hi = 0;
    for (int w : weights) hi += w;

    int answer = hi;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (canShip(weights, mid, days)) {
            answer = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }

    return answer;
}
// Time: O(n log(sum(weights))), Space: O(1)
```

### Dry Run

```
weights=[1,2,3,4,5,6,7,8,9,10], days=5
lo=10, hi=55, answer=55

mid=32: Day1:[1,2,3,4,5,6,7]=28. 28+8=36>32→Day2. Day2:[8,9]=17. 17+10=27→Day2. [8,9,10]=27.
        2 days ≤ 5 ✓ → answer=32, hi=31.

(After several iterations...)

mid=15: Day1:[1,2,3,4,5]=15. +6=21>15→Day2. Day2:[6,7]=13. +8=21>15→Day3.
        Day3:[8]. +9=17>15→Day4. Day4:[9]. +10=19>15→Day5. Day5:[10].
        5 days ≤ 5 ✓ → answer=15, hi=14.

mid=12: Day1:[1,2,3,4]=10. +5=15>12→Day2.  ...  6 days > 5 ✗ → lo=13.

(Converges to 15.)
Return 15  ✓
```

### Structural Similarity to Koko

| | Koko | Ship Packages |
|---|---|---|
| Minimize | speed k | capacity cap |
| lo | 1 | max(weights) |
| hi | max(piles) | sum(weights) |
| Feasibility | total hours ≤ h | days needed ≤ days |
| Per-item cost | ceil(pile/k) | greedy contiguous packing |

Both are "minimize the maximum workload" problems. Recognizing this family makes the solution immediate.

---

## 9. Kth Missing Positive Number (LC 1539)

### Problem Statement

Given a **strictly increasing sorted** array `arr` of positive integers and integer `k`, return the k-th missing positive number.

```
arr=[2,3,4,7,11], k=5  → 9   (missing: 1,5,6,8,9,... → 5th = 9)
arr=[1,2,3,4],    k=2  → 6   (missing: 5,6,7,... → 2nd = 6)
```

### Key Observation — Counting Missing Numbers

At index `i` (0-indexed), if there were no missing numbers, `arr[i]` would equal `i+1`. The number of positive integers **missing in the range [1, arr[i]]** is:

```
missing(i) = arr[i] - (i + 1) = arr[i] - i - 1
```

```
arr = [2, 3, 4, 7, 11]
  i=0: missing = 2-0-1 = 1    (only 1 is missing before position 0)
  i=1: missing = 3-1-1 = 1
  i=2: missing = 4-2-1 = 1
  i=3: missing = 7-3-1 = 3    (1,5,6 are missing up to here)
  i=4: missing = 11-4-1 = 6   (1,5,6,8,9,10 are missing up to here)
```

`missing(i)` is monotonically non-decreasing — binary search applies.

### Approach 1: Linear Scan — O(n)

As we encounter array elements ≤ current target k, shift k forward because those elements are "occupying" a slot.

```cpp
int findKthPositive(vector<int>& arr, int k) {
    for (int x : arr) {
        if (x <= k) k++;   // x takes a slot; push target forward
        else break;        // x > k means the k-th missing is exactly k
    }
    return k;
}
```

**Dry Run (arr=[2,3,4,7,11], k=5):**
```
x=2:  2<=5 → k=6
x=3:  3<=6 → k=7
x=4:  4<=7 → k=8
x=7:  7<=8 → k=9
x=11: 11<=9? No → break.
Return 9  ✓
```

### Approach 2: Binary Search — O(log n)

**Goal:** Find the smallest index `lo` such that `missing(lo) >= k`. The k-th missing number lies in the gap before `arr[lo]`.

**Answer formula:** After the binary search, `lo` is the number of array elements exhausted before the k-th missing. The answer is `lo + k`.

**Why `lo + k` works:** After the loop, `lo` array elements (indices 0 to lo-1) all appear before the answer, each shifting the sequence by 1. The k-th missing integer in the natural numbers, accounting for these lo occupied slots, is `lo + k`.

```cpp
int findKthPositive(vector<int>& arr, int k) {
    int lo = 0, hi = (int)arr.size() - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] - mid - 1 < k) {
            lo = mid + 1;   // not enough missing to the left, go right
        } else {
            hi = mid - 1;   // enough missing here or to the left, go left
        }
    }

    return lo + k;   // lo = number of arr elements before the answer
}
// Time: O(log n), Space: O(1)
```

### Dry Run (arr=[2,3,4,7,11], k=5)

```
lo=0, hi=4

mid=2: arr[2]=4. missing=4-2-1=1 < 5 → lo=3
mid=3: arr[3]=7. missing=7-3-1=3 < 5 → lo=4
mid=4: arr[4]=11. missing=11-4-1=6 >= 5 → hi=3

lo(4) > hi(3) → stop.
Answer = lo + k = 4 + 5 = 9  ✓
```

### Dry Run (arr=[1,2,3,4], k=2)

```
lo=0, hi=3

mid=1: arr[1]=2. missing=2-1-1=0 < 2 → lo=2
mid=2: arr[2]=3. missing=0 < 2 → lo=3
mid=3: arr[3]=4. missing=0 < 2 → lo=4

lo(4) > hi(3) → stop.
Answer = 4 + 2 = 6  ✓  (missing beyond the array: 5, 6, ... → 2nd = 6)
```

### Dry Run (arr=[2,3,4,7,11], k=1)

```
mid=2: missing=1 >= 1 → hi=1
mid=0: missing=2-0-1=1 >= 1 → hi=-1

lo(0) > hi(-1) → stop.
Answer = 0 + 1 = 1  ✓  (1 is the 1st missing number)
```

### Edge Cases

| arr | k | Output | Note |
|---|---|---|---|
| `[2,3,4,7,11]` | `5` | `9` | Standard case |
| `[1,2,3,4]` | `2` | `6` | All small numbers present; answer beyond array |
| `[1]` | `1` | `2` | 1 is in array; 1st missing is 2 |
| `[1000]` | `1` | `1` | 1 is missing and precedes the array |

---

## 10. How to Identify a Binary Search on Answer Problem

### Signal Phrases

```
"minimum ... such that ..."
"maximum ... such that ..."
"smallest X that satisfies ..."
"largest X that satisfies ..."
"minimize the maximum ..."
"maximize the minimum ..."
```

### Decision Checklist

```
1. Is there a monotonic relationship between the answer variable and feasibility?
   (Larger speed → fewer hours. Larger capacity → fewer days needed.)
   → YES: candidate for binary search on answer.

2. Can you write feasible(candidate) that runs in O(n)?
   → YES: total complexity will be O(n log range).

3. What are lo and hi for the answer range?
   lo = smallest value that could possibly be valid (from problem constraints)
   hi = largest value that could possibly be valid (sum, max, or known bound)

4. Are you finding minimum feasible or maximum feasible?
   Minimum feasible: when feasible, store ans=mid and go left (hi=mid-1)
   Maximum feasible: when feasible, store ans=mid and go right (lo=mid+1)
```

### Full Problem Family

| Problem | LeetCode | Answer | Pattern |
|---|---|---|---|
| Sqrt(x) | 69 | Floor sqrt | Max m: m²≤x |
| Nth Root of M | GFG | Integer nth root | Exact m: mⁿ=x |
| Koko Eating Bananas | 875 | Min speed | Min k: total hours≤h |
| Min Days Bouquets | 1482 | Min days | Min d: bouquets(d)≥m |
| Smallest Divisor | 1283 | Min divisor | Min d: sum_ceil≤threshold |
| Capacity to Ship | 1011 | Min capacity | Min c: days_needed≤days |
| Kth Missing Positive | 1539 | Kth missing | Binary on missing count |
| Split Array Largest Sum | 410 | Min max subarray sum | Min c: splits≤m |
| Magnetic Force Between Balls | 1552 | Max min distance | Max d: placements≥m |
| Book Allocation | GFG | Min max pages | Min p: students≤m |
| Aggressive Cows | GFG | Max min distance | Max d: cows fit in ≥k stalls |

---

## 11. Common Mistakes and Pitfalls

### Mistake 1: Wrong lo/hi for the Answer Range

```cpp
// WRONG: lo=1 for Ship Packages
int lo = 1, hi = sum;
// Capacity of 1 can't ship any package with weight > 1.
// The canShip function would compute wrong day counts.

// CORRECT: lo = max(weights)
int lo = *max_element(weights.begin(), weights.end());
```

### Mistake 2: Integer Overflow in mid*mid

```cpp
// WRONG: mid is int; mid*mid overflows when mid is large
int mid = (lo + hi) / 2;
if (mid * mid <= x) ...   // overflows for x near INT_MAX

// CORRECT: use long long for mid
long long mid = lo + (hi - lo) / 2;
if (mid * mid <= (long long)x) ...
```

### Mistake 3: Using (lo + hi) / 2 Instead of lo + (hi - lo) / 2

```cpp
// WRONG: overflows when lo and hi are both near INT_MAX
int mid = (lo + hi) / 2;

// CORRECT: overflow-safe
int mid = lo + (hi - lo) / 2;
```

### Mistake 4: Floor Division Instead of Ceiling Division

```cpp
// WRONG: floor division gives incorrect hour count
hours += pile / speed;

// CORRECT: ceiling division
hours += (pile + speed - 1) / speed;
// Alternative: (pile - 1) / speed + 1   (equivalent for positive integers)
```

### Mistake 5: Missing Impossibility Check in Bouquets

```cpp
// WRONG: skipping the impossible check causes incorrect binary search behavior
// when m*k > n (no amount of days can help)
int minDays(vector<int>& bloomDay, int m, int k) {
    // Missing: if ((long long)m * k > bloomDay.size()) return -1;
    ...
}

// CORRECT: always check before binary searching
if ((long long)m * k > (int)bloomDay.size()) return -1;
```

### Mistake 6: Off-by-One in Kth Missing Formula

```cpp
// WRONG
return hi + k;      // off by 1 — hi ends at lo-1, not lo
return lo + k - 1;  // off by 1 in the other direction

// CORRECT: after the binary search loop with lo <= hi template,
// lo is the first index where missing(lo) >= k, and the answer is lo + k.
return lo + k;
```

### Mistake 7: Not Using Early Exit in Feasibility Checks

```cpp
// SLOW: always processes all elements
bool canFinish(vector<int>& piles, int k, int h) {
    long long hours = 0;
    for (int p : piles) hours += (p + k - 1) / k;
    return hours <= h;   // computes even when already > h
}

// FAST: early exit as soon as infeasible
bool canFinish(vector<int>& piles, int k, int h) {
    long long hours = 0;
    for (int p : piles) {
        hours += (p + k - 1) / k;
        if (hours > h) return false;   // no need to continue
    }
    return true;
}
```

### Mistake 8: Incorrect Power Function for Nth Root (No Overflow Guard)

```cpp
// WRONG: overflows for large mid and n
long long result = 1;
for (int i = 0; i < n; i++) result *= mid;  // no early exit

// CORRECT: exit as soon as result exceeds m
for (int i = 0; i < n; i++) {
    result *= mid;
    if (result > m) return -1;   // overflow guard
}
```

---

## 12. Complexity Cheat Sheet

| Problem | lo | hi | Feasibility check | Total time | Space |
|---|---|---|---|---|---|
| Sqrt(x) | 1 | x | O(1) | O(log x) | O(1) |
| Sqrt(x) Newton | — | — | O(1) | O(log log x) | O(1) |
| Nth Root of M | 1 | m | O(n) | O(n log m) | O(1) |
| Koko Eating Bananas | 1 | max(piles) | O(n) | O(n log max) | O(1) |
| Min Days Bouquets | 1 | max(bloomDay) | O(n) | O(n log max) | O(1) |
| Smallest Divisor | 1 | max(nums) | O(n) | O(n log max) | O(1) |
| Capacity to Ship | max(w) | sum(w) | O(n) | O(n log sum) | O(1) |
| Kth Missing (linear) | — | — | — | O(n) | O(1) |
| Kth Missing (binary) | 0 | n-1 | O(1) | O(log n) | O(1) |

**General rule:**

```
Total time = O(feasibility_check_cost × log(answer_range_size))
```

---

## 13. Quick Revision Cheat Sheet

Self-contained for same-day interview revision.

---

### The One Idea

```
Binary search on answer:
  Search the range of possible answer VALUES (not array indices).
  For each candidate mid, run feasible(mid) to determine direction.
  Monotonicity guarantee: feasible(x) stays true (or false) once it flips.
```

---

### Universal Templates

```cpp
// Find MINIMUM feasible value
int lo=minPossible, hi=maxPossible, ans=hi;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (feasible(mid)) { ans=mid; hi=mid-1; }
    else lo=mid+1;
}
return ans;

// Find MAXIMUM feasible value
int lo=minPossible, hi=maxPossible, ans=lo;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (feasible(mid)) { ans=mid; lo=mid+1; }
    else hi=mid-1;
}
return ans;
```

---

### Per-Problem Reference

```cpp
// Sqrt(x)   — find max m: m*m <= x
long long lo=1, hi=x, ans=0;
// feasible(mid): mid*mid <= x  [O(1)]

// Nth Root of M   — find exact m: m^n == M, else -1
// feasible(mid): checkPow(mid,n,m)==1  [O(n), with overflow guard]

// Koko Eating Bananas   — find min k: sum(ceil(p/k)) <= h
int lo=1, hi=max(piles);
// feasible(k): sum((p+k-1)/k for p in piles) <= h  [O(n)]

// Min Days Bouquets   — find min d: count_bouquets(d) >= m
// Impossibility check first: if (m*k > n) return -1;
int lo=1, hi=max(bloomDay);
// feasible(d): count consecutive bloomed groups of k  [O(n)]

// Smallest Divisor   — find min d: sum(ceil(x/d)) <= threshold
int lo=1, hi=max(nums);
// feasible(d): sum((x+d-1)/d for x in nums) <= threshold  [O(n)]

// Capacity to Ship   — find min cap: days_needed <= days
int lo=max(weights), hi=sum(weights);
// feasible(cap): greedy packing, count days  [O(n)]

// Kth Missing (binary)   — answer = lo + k after binary search
int lo=0, hi=n-1;
while (lo<=hi) {
    int mid=lo+(hi-lo)/2;
    if (arr[mid]-mid-1 < k) lo=mid+1;
    else hi=mid-1;
}
return lo + k;
```

---

### Ceil Division

```cpp
// ceil(a / b) for positive integers a, b:
(a + b - 1) / b     // most common form
(a - 1) / b + 1     // equivalent alternative
```

---

### Answer Space Quick Reference

```
Sqrt(x):          lo=1,            hi=x
Nth Root:         lo=1,            hi=m
Koko:             lo=1,            hi=max(piles)
Bouquets:         lo=1,            hi=max(bloomDay)
Smallest Divisor: lo=1,            hi=max(nums)
Ship Packages:    lo=max(weights), hi=sum(weights)
Kth Missing (BS): lo=0(idx),       hi=n-1(idx) → return lo+k
```

---

### Critical Implementation Rules

```
1. Always use lo + (hi - lo) / 2, never (lo + hi) / 2.
2. Always use long long for mid * mid, sums, and powers.
3. Always use ceil division: (a + b - 1) / b, never a / b.
4. For bouquets, check impossibility (m*k > n) before binary searching.
5. In power function for nth root, exit early when result > m.
6. For feasibility checks with sums, add early exit when sum exceeds limit.
7. Kth missing answer: lo + k after the binary search terminates.
```
