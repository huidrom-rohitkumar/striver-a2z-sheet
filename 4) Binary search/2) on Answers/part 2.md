**Resources:**
[SPOJ Aggressive Cows](https://www.spoj.com/problems/AGGRCOW/) |
[GFG Book Allocation](https://www.geeksforgeeks.org/problems/allocate-minimum-number-of-pages0937/1) |
[LC 410 Split Array](https://leetcode.com/problems/split-array-largest-sum/) |
[InterviewBit Painter's Partition](https://www.interviewbit.com/problems/painters-partition-problem/) |
[LC 774 Gas Station](https://leetcode.com/problems/minimize-max-distance-to-gas-station/) |
[LC 4 Median](https://leetcode.com/problems/median-of-two-sorted-arrays/) |
[GFG Kth Element](https://www.geeksforgeeks.org/problems/k-th-element-of-two-sorted-array1317/1) |
[Striver Aggressive Cows](https://youtu.be/R_Mfw4ew-Vo) |
[Striver Book Allocation](https://youtu.be/Z0hwjftStI4) |
[Striver Split Array](https://youtu.be/thUd_WJn6wk) |
[Striver Gas Station](https://youtu.be/kMSBvlZ-_HA) |
[Striver Median](https://youtu.be/C2rRzz-JDk8) |
[Striver Kth Element](https://youtu.be/D1oDwWCq50g)

---

## Table of Contents

1. [The Unifying Mental Model](#1-the-unifying-mental-model)
2. [Generic Template: Binary Search on Answer](#2-generic-template-binary-search-on-answer)
3. [Problem 1 — Aggressive Cows (SPOJ)](#3-problem-1--aggressive-cows-spoj)
4. [Problem 2 — Allocate Minimum Pages (Book Allocation)](#4-problem-2--allocate-minimum-pages-book-allocation)
5. [Problem 3 — Painter's Partition Problem](#5-problem-3--painters-partition-problem)
6. [Problem 4 — Split Array Largest Sum (LC 410)](#6-problem-4--split-array-largest-sum-lc-410)
7. [Problem 5 — Minimize Max Distance to Gas Station (LC 774)](#7-problem-5--minimize-max-distance-to-gas-station-lc-774)
8. [Problem 6 — Median of Two Sorted Arrays (LC 4)](#8-problem-6--median-of-two-sorted-arrays-lc-4)
9. [Problem 7 — K-th Element of Two Sorted Arrays (GFG)](#9-problem-7--k-th-element-of-two-sorted-arrays-gfg)
10. [Why Binary Search Works on Monotonic Answer Spaces — Formal Argument](#10-why-binary-search-works-on-monotonic-answer-spaces--formal-argument)
11. [Taxonomy of Binary Search on Answer Problems](#11-taxonomy-of-binary-search-on-answer-problems)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Interview Reference — Patterns and Application Map](#13-interview-reference--patterns-and-application-map)
14. [Complexity Cheat Sheet](#14-complexity-cheat-sheet)
15. [Quick Revision Cheat Sheet](#15-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Model

Every problem in this set follows the same structure even though the surface stories differ — cows in stalls, books to students, painters on boards, gas stations on a highway, partitioning arrays. Recognizing this structure is the first skill.

**The pattern in one sentence:** You cannot search directly for the answer in an array, but you *can* search for a numeric value `x` such that `feasible(x)` is true, where `feasible` is a monotonic predicate over the answer space.

Break it down:

1. The answer lives in a **numeric range** `[lo, hi]`, not in any specific array index.
2. There exists a **feasibility predicate** `canDo(x)`: given a candidate answer `x`, can the problem be solved at or within this threshold?
3. The predicate is **monotone**: if `canDo(x)` is true, then `canDo(x')` is true for all `x'` on the "easier" side. This creates a clean `F F F T T T` (or `T T T F F F`) pattern.
4. **Binary search finds the boundary** in O(log(range)) evaluations of the predicate.
5. **Each predicate evaluation costs O(n)**, giving total O(n log(range)).

The critical insight beginners miss: you are not searching in the array. You are searching the space of possible answers. The array is only used inside the feasibility checker.

**Two mirror-image flavors:**

- **Maximize the minimum** (Aggressive Cows): the predicate is `canDo(mid)` = "can we achieve minimum distance ≥ mid?" Small mid is easy, large mid fails. Answer: the largest mid for which the predicate is true. When feasible, record and move `lo = mid + 1`.
- **Minimize the maximum** (Book Allocation, Painter's Partition, Split Array): the predicate is `canDo(mid)` = "can we partition such that no part exceeds mid?" Large mid is easy, small mid fails. Answer: the smallest mid for which the predicate is true. When feasible, record and move `hi = mid - 1`.

These two are duals. Master one and the other follows immediately.

**Two separate pattern families:**

Problems 1–5 belong to the **answer-space binary search** family. Problems 6–7 belong to a fundamentally different family: **partition binary search on two sorted arrays**. Both use binary search, but the search space and logic are different. Do not conflate them.

---

## 2. Generic Template: Binary Search on Answer

```cpp
// Template for "minimize the maximum" problems.
// For "maximize the minimum": swap the update directions (lo=mid+1 when feasible).

bool canDo(vector<int>& arr, long long mid, int k) {
    int parts = 1;
    long long cur = 0;
    for (int x : arr) {
        if ((long long)x > mid) return false;   // single element exceeds limit
        if (cur + x > mid) {
            parts++;
            cur = x;
            if (parts > k) return false;         // early exit
        } else {
            cur += x;
        }
    }
    return true;
}

long long binarySearchOnAnswer(vector<int>& arr, int k) {
    long long lo = *max_element(arr.begin(), arr.end());  // smallest possible answer
    long long hi = accumulate(arr.begin(), arr.end(), 0LL); // largest possible answer
    long long ans = hi;

    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (canDo(arr, mid, k)) {
            ans = mid;       // feasible: record and try smaller
            hi = mid - 1;
        } else {
            lo = mid + 1;   // infeasible: try larger
        }
    }
    return ans;
}
```

**Why `lo = max_element` and `hi = sum`?**

The answer cannot be smaller than the largest single element (you can never split one element across partitions), and cannot exceed the total sum (one group takes everything). These are tight bounds — using `lo = 0` or `hi = INT_MAX` introduces unnecessary iterations and risks division-by-zero in the gas station problem.

**Why `long long`?**

If `n = 10^5` and `arr[i] = 10^8`, then `sum = 10^13`, which overflows `int` (max ~2.1 × 10^9). Always use `long long` for sums and comparisons in these problems.

---

## 3. Problem 1 — Aggressive Cows (SPOJ)

### Problem Statement

Farmer John has a barn with `N` stalls at positions `x[0..N-1]` on a number line (not necessarily sorted). He has `C` aggressive cows to place in distinct stalls. He wants to maximize the minimum distance between any two cows. Return the largest possible minimum distance.

**Constraints:** `2 <= N <= 100,000`, `2 <= C <= N`, `0 <= x[i] <= 1,000,000,000`.

**Examples:**
- `stalls = [1,2,4,8,9]`, `C = 3` → `3` (place at 1, 4, 8 or 1, 4, 9)
- `stalls = [0,1000000000]`, `C = 2` → `1000000000`

### Approach 1 — Brute Force

Try every possible minimum distance from 1 to `max - min` and find the largest that works.

```cpp
bool canPlace(vector<int>& stalls, int minDist, int C) {
    int count = 1, last = stalls[0];
    for (int i = 1; i < (int)stalls.size(); i++)
        if (stalls[i] - last >= minDist) { count++; last = stalls[i]; }
    return count >= C;
}

int aggressiveCows_brute(vector<int> stalls, int C) {
    sort(stalls.begin(), stalls.end());
    int ans = 0;
    for (int d = 1; d <= stalls.back() - stalls.front(); d++) {
        if (canPlace(stalls, d, C)) ans = d;
        else break;
    }
    return ans;
}
```

Time: O(n log n + n * (max − min)). For max stall = 10^9, the linear sweep is unusable.

### Approach 2 — Binary Search on Answer (Optimal)

**Intuition:** The answer space is `[1, max_stall - min_stall]`. Feasibility is monotone: if distance `d` works, every smaller `d` also works (same placement is still valid). We want the largest feasible `d`. Binary search, record when feasible, move `lo = mid + 1`.

**Greedy feasibility:** Sort stalls. Place the first cow at `stalls[0]`. For each subsequent stall, place the next cow there only if its distance from the last placed cow is at least `minDist`. This greedy is optimal because placing each cow as early as possible leaves maximum room for all remaining cows — any delay can only make it harder to fit the rest.

**Why sorting is mandatory:** The stalls are at arbitrary positions. Only after sorting does "earliest valid stall" have meaning. Without sorting, the greedy would evaluate arbitrary orderings and fail.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canPlace(vector<int>& stalls, int minDist, int C) {
    int cows = 1, last = stalls[0];
    for (int i = 1; i < (int)stalls.size(); i++) {
        if (stalls[i] - last >= minDist) {
            cows++;
            last = stalls[i];
            if (cows == C) return true;   // early exit
        }
    }
    return cows >= C;
}

int aggressiveCows(vector<int> stalls, int C) {
    sort(stalls.begin(), stalls.end());
    int lo = 1;
    int hi = stalls.back() - stalls.front();
    int ans = 0;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (canPlace(stalls, mid, C)) {
            ans = mid;        // feasible: record, try larger
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}
```

**Dry run:** `stalls = [1, 2, 4, 8, 9]`, `C = 3` (already sorted)

```
lo=1, hi=8

mid=4: canPlace(4, 3)?
  last=1, cows=1
  stalls[1]=2: 2-1=1 < 4 → skip
  stalls[2]=4: 4-1=3 < 4 → skip
  stalls[3]=8: 8-1=7 >= 4 → cows=2, last=8
  stalls[4]=9: 9-8=1 < 4 → skip
  cows=2 < 3 → false
→ hi=3

lo=1, hi=3
mid=2: canPlace(2, 3)?
  last=1, cows=1
  stalls[1]=2: 2-1=1 < 2 → skip
  stalls[2]=4: 4-1=3 >= 2 → cows=2, last=4
  stalls[3]=8: 8-4=4 >= 2 → cows=3 → true (early exit)
→ ans=2, lo=3

lo=3, hi=3
mid=3: canPlace(3, 3)?
  last=1, cows=1
  stalls[1]=2: 2-1=1 < 3 → skip
  stalls[2]=4: 4-1=3 >= 3 → cows=2, last=4
  stalls[3]=8: 8-4=4 >= 3 → cows=3 → true
→ ans=3, lo=4

lo=4 > hi=3 → stop
Answer: 3
```

| Scenario | Expected | Reason |
|---|---|---|
| `C = 2` | `stalls.back() - stalls.front()` | Place at extreme ends |
| `C = N` | Min adjacent gap after sorting | Every stall must hold one cow |
| `C = 1` | 0 (any single stall) | No pair constraint |

Time: O(n log n + n log(max_dist)). Space: O(1) extra.

---

## 4. Problem 2 — Allocate Minimum Pages (Book Allocation)

### Problem Statement

`N` books with `arr[i]` pages each, allocated to `K` students such that: each student gets at least one book, books are allocated **contiguously**, and the maximum pages assigned to any student is minimized. Return this minimum possible maximum. Return `-1` if `K > N`.

**Constraints:** `1 <= N <= 10^5`, `1 <= arr[i] <= 10^8`, `1 <= K <= 10^5`.

**Examples:**
- `arr = [12, 34, 67, 90]`, `K = 2` → `113` (students: [12,34,67] and [90])
- `arr = [15, 17, 20]`, `K = 5` → `-1` (only 3 books for 5 students)

### Approach 1 — Brute Force

Try every value from `max(arr)` to `sum(arr)` and find the first feasible one.

```cpp
int allocateBooks_brute(vector<int>& arr, int K) {
    int n = arr.size();
    if (K > n) return -1;
    int lo = *max_element(arr.begin(), arr.end());
    long long hi = accumulate(arr.begin(), arr.end(), 0LL);
    for (long long m = lo; m <= hi; m++) {
        int students = 1; long long pages = 0;
        bool ok = true;
        for (int x : arr) {
            if (pages + x > m) { students++; pages = x; }
            else pages += x;
            if (students > K) { ok = false; break; }
        }
        if (ok) return (int)m;
    }
    return -1;
}
```

Time: O(n × (sum − max)), which can be O(n × 10^13). Completely infeasible.

### Approach 2 — Binary Search on Answer (Optimal)

**Intuition:** The minimum possible maximum (`lo`) is the largest single book (at least one student must read it alone). The maximum possible maximum (`hi`) is the total sum (one student reads all). Binary search this range. For each candidate `mid`, greedily count the minimum students required: accumulate pages for the current student; when the next book would push past `mid`, give it to a new student.

**Greedy correctness:** Assigning as many books as possible to the current student minimizes the number of students needed for a given `mid`. Any assignment that cuts earlier would require more or equal students for the remainder. Formally: by an exchange argument, any non-greedy assignment can be converted to the greedy assignment without increasing the partition count.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canAllocate(vector<int>& arr, long long maxPages, int K) {
    int students = 1;
    long long pages = 0;
    for (int x : arr) {
        if ((long long)x > maxPages) return false;   // single book exceeds limit
        if (pages + x > maxPages) {
            students++;
            pages = x;
            if (students > K) return false;
        } else {
            pages += x;
        }
    }
    return true;
}

int allocateBooks(vector<int>& arr, int K) {
    int n = arr.size();
    if (K > n) return -1;

    long long lo = *max_element(arr.begin(), arr.end());
    long long hi = accumulate(arr.begin(), arr.end(), 0LL);
    long long ans = hi;

    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (canAllocate(arr, mid, K)) {
            ans = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return (int)ans;
}
```

**Dry run:** `arr = [12, 34, 67, 90]`, `K = 2`

```
lo=90, hi=203

mid=146: students=1, pages=0
  +12→12; +34→46; +67→113; +90→203>146 → students=2, pages=90
  students=2 ≤ 2 → true
  ans=146, hi=145

mid=117:
  +12→12; +34→46; +67→113; +90→203>117 → students=2, pages=90
  true → ans=117, hi=116

mid=103:
  +12→12; +34→46; +67→113>103 → students=2, pages=67
  +90→157>103 → students=3 > 2 → false
  lo=104

mid=110:
  +12→12; +34→46; +67→113>110 → students=2, pages=67
  +90→157>110 → students=3 → false
  lo=111

mid=113:
  +12→12; +34→46; +67→113
  +90→203>113 → students=2, pages=90
  true → ans=113, hi=112

mid=111:
  +12→12; +34→46; +67→113>111 → students=2, pages=67
  +90→157>111 → students=3 → false
  lo=112

mid=112:
  +12→12; +34→46; +67→113>112 → students=2, pages=67
  +90→157>112 → students=3 → false
  lo=113

lo=113 > hi=112 → stop
Answer: 113
```

| Scenario | Expected | Reason |
|---|---|---|
| `K > N` | `-1` | Cannot give each student at least one book |
| `K = 1` | `sum(arr)` | One student reads everything |
| `K = N` | `max(arr)` | Each student gets exactly one book |
| Single very large element | That element | It is the tight lower bound `lo` |

Time: O(n log(sum)). Space: O(1).

---

## 5. Problem 3 — Painter's Partition Problem

### Problem Statement

`N` boards of lengths `arr[0..N-1]`. `K` painters each take 1 unit time per unit of board length. A painter paints only contiguous boards. Minimize the total time (since all paint simultaneously, time = max load assigned to any painter).

**Constraints:** `1 <= N <= 10^5`, `1 <= arr[i] <= 10^5`, `1 <= K <= 10^5`.

**Examples:**
- `arr = [10, 20, 30, 40]`, `K = 2` → `60` (painter 1: [10,20,30], painter 2: [40])
- `arr = [5, 10, 30, 20, 15]`, `K = 3` → `35`

**Key observation:** This is structurally **identical** to Book Allocation. Books → boards, students → painters, pages → lengths. The feasibility check, binary search bounds, and predicate are word-for-word the same algorithm. Recognizing isomorphic problems is a core interview skill.

The only surface differences: Painter's Partition has no "K > N" impossible case (painters may paint 0 boards), and the InterviewBit variant multiplies the answer by a time-per-unit `B` and takes modulo 10000003.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canFinish(vector<int>& arr, long long maxLoad, int K) {
    int painters = 1;
    long long load = 0;
    for (int x : arr) {
        if ((long long)x > maxLoad) return false;
        if (load + x > maxLoad) {
            painters++;
            load = x;
            if (painters > K) return false;
        } else {
            load += x;
        }
    }
    return true;
}

long long painterPartition(vector<int>& arr, int K) {
    long long lo = *max_element(arr.begin(), arr.end());
    long long hi = accumulate(arr.begin(), arr.end(), 0LL);
    long long ans = hi;

    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (canFinish(arr, mid, K)) {
            ans = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return ans;
}

// InterviewBit variant
int interviewBit(int K, int B, vector<int>& boards) {
    const int MOD = 10000003;
    long long minLen = painterPartition(boards, K);
    return (int)((minLen % MOD) * (B % MOD) % MOD);
}
```

**Dry run:** `arr = [5, 10, 30, 20, 15]`, `K = 3`

```
lo=30, hi=80

mid=55:
  +5→5; +10→15; +30→45; +20→65>55 → painters=2, load=20
  +15→35 → painters=2 ≤ 3 → true
  ans=55, hi=54

mid=42:
  +5→5; +10→15; +30→45>42 → painters=2, load=30
  +20→50>42 → painters=3, load=20
  +15→35 → painters=3 ≤ 3 → true
  ans=42, hi=41

mid=35:
  +5→5; +10→15; +30→45>35 → painters=2, load=30
  +20→50>35 → painters=3, load=20
  +15→35 → painters=3 ≤ 3 → true
  ans=35, hi=34

mid=32:
  +5→5; +10→15; +30→45>32 → painters=2, load=30
  +20→50>32 → painters=3, load=20
  +15→35>32 → painters=4 > 3 → false
  lo=33

(iterations 33, 34 also false) → lo=35 > hi=34 → stop
Answer: 35
```

Time: O(n log(sum)). Space: O(1).

---

## 6. Problem 4 — Split Array Largest Sum (LC 410)

### Problem Statement

Split integer array `nums` into exactly `k` non-empty contiguous subarrays to minimize the largest subarray sum. Return this minimized value.

**Constraints:** `1 <= nums.length <= 1000`, `0 <= nums[i] <= 10^6`, `1 <= k <= min(50, nums.length)`.

**Examples:**
- `nums = [7, 2, 5, 10, 8]`, `k = 2` → `18` (splits: [7,2,5] and [10,8])
- `nums = [1, 2, 3, 4, 5]`, `k = 2` → `9` (splits: [1,2,3,4] and [5])

**This is the same problem as Book Allocation and Painter's Partition**, with different variable names. Split array = allocate books = painter's partition = capacity to ship packages (LC 1011). The code below is identical to Section 4 with renamed variables.

### Approach 1 — Dynamic Programming

`dp[i][j]` = minimum largest sum when splitting `nums[0..i-1]` into `j` parts.

```cpp
int splitArray_dp(vector<int>& nums, int k) {
    int n = nums.size();
    vector<long long> prefix(n + 1, 0);
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + nums[i];

    vector<vector<long long>> dp(n + 1, vector<long long>(k + 1, LLONG_MAX));
    dp[0][0] = 0;

    for (int j = 1; j <= k; j++) {
        for (int i = j; i <= n; i++) {
            for (int m = j - 1; m < i; m++) {
                if (dp[m][j - 1] == LLONG_MAX) continue;
                long long cost = max(dp[m][j - 1], prefix[i] - prefix[m]);
                dp[i][j] = min(dp[i][j], cost);
            }
        }
    }
    return (int)dp[n][k];
}
```

Time: O(n^2 × k). For n=1000, k=50: ~50 million ops, borderline. Fails for n=10^5.
Space: O(n × k).

### Approach 2 — Binary Search on Answer (Optimal)

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canSplit(vector<int>& nums, long long mid, int k) {
    int parts = 1;
    long long cur = 0;
    for (int x : nums) {
        if ((long long)x > mid) return false;
        if (cur + x > mid) {
            parts++;
            cur = x;
            if (parts > k) return false;
        } else {
            cur += x;
        }
    }
    return true;
}

int splitArray(vector<int>& nums, int k) {
    long long lo = *max_element(nums.begin(), nums.end());
    long long hi = accumulate(nums.begin(), nums.end(), 0LL);
    long long ans = hi;

    while (lo <= hi) {
        long long mid = lo + (hi - lo) / 2;
        if (canSplit(nums, mid, k)) {
            ans = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return (int)ans;
}
```

**Dry run:** `nums = [7, 2, 5, 10, 8]`, `k = 2`

```
lo=10, hi=32

mid=21:
  +7→7; +2→9; +5→14; +10→24>21 → parts=2, cur=10
  +8→18 → parts=2 ≤ 2 → true
  ans=21, hi=20

mid=15:
  +7→7; +2→9; +5→14; +10→24>15 → parts=2, cur=10
  +8→18>15 → parts=3 > 2 → false
  lo=16

mid=18:
  +7→7; +2→9; +5→14; +10→24>18 → parts=2, cur=10
  +8→18 → parts=2 ≤ 2 → true
  ans=18, hi=17

mid=16, mid=17: similar analysis → false → lo=18 > hi=17 → stop
Answer: 18
```

Time: O(n log(sum)). Space: O(1).

---

## 7. Problem 5 — Minimize Max Distance to Gas Station (LC 774)

### Problem Statement

Gas stations are at positions `stations[0] < stations[1] < ... < stations[N-1]`. Add exactly `K` new gas stations at any real-number positions. Minimize the maximum distance between any two adjacent stations. Return the answer with error ≤ 10^{-6}.

**Constraints:** `10 <= stations.length <= 2000`, `0 <= stations[i] <= 10^8`, `1 <= K <= 10^6`.

**Example:** `stations = [1,2,3,...,10]`, `K = 9` → `0.5`

### Approach 1 — Priority Queue (Brute Force, O((n+k) log n))

Maintain a max-heap of current interval sizes (split each interval on demand). Too slow for k = 10^6.

### Approach 2 — Binary Search on Real Answer (Optimal)

**Intuition:** Binary search on the answer `d` (maximum allowed gap after placement). For a given `d`, the minimum number of new stations needed in a gap of length `g` is `floor(g / d)` — we split `g` into `floor(g/d) + 1` pieces each of size `g / (floor(g/d) + 1) <= d`. Summing over all gaps gives the total stations needed. If this total ≤ K, distance `d` is achievable.

The predicate is monotone: larger `d` is easier to achieve (fewer stations needed). So the feasibility function is `F F F T T T` as `d` increases, and we want the smallest feasible `d` — move `hi = mid` when feasible.

**Why use a fixed number of iterations instead of `while(hi - lo > 1e-6)`?** The floating-point termination condition can stall due to rounding precision — the difference may stay above `1e-6` indefinitely. Using 100 iterations with initial range `1e8` yields precision `1e8 / 2^100 ≈ 10^{-22}`, far exceeding the `10^{-6}` requirement.

**The counting formula explained:** For a gap `g` and maximum distance `d`, the number of new stations needed is `(int)(g / d)`. When `g / d` is an exact integer (say `g = 1.0, d = 0.5`), `(int)(1.0/0.5) = 2`, but we only need 1 station (split into two halves). The correct formula is `ceil(g/d) - 1`. For exact division: `ceil(g/d) - 1 = (g/d) - 1 = floor(g/d) - ... wait`. Let us re-derive: for gap `g`, if we insert `s` stations, the gap splits into `s+1` sub-intervals of length `g/(s+1)`. We need `g/(s+1) <= d`, i.e., `s+1 >= g/d`, i.e., `s >= g/d - 1`, i.e., `s = ceil(g/d - 1 + epsilon) = ceil(g/d) - 1` when `g/d` is not an integer, and `s = g/d - 1` when exact. In C++: `s = (int)ceil(g/d) - 1`. Equivalently, `s = (int)((g - 1e-9) / d)`.

In practice, since `mid` is a floating-point value produced by `(lo + hi) / 2.0` and almost never exactly divides any gap, `(int)(g / mid)` behaves correctly for essentially all iterations. For guaranteed correctness, use the safe form shown below.

```cpp
#include <bits/stdc++.h>
using namespace std;

bool canAchieve(vector<int>& stations, double maxDist, int k) {
    int needed = 0;
    for (int i = 0; i < (int)stations.size() - 1; i++) {
        double gap = stations[i + 1] - stations[i];
        needed += (int)(gap / maxDist);   // floor(gap/maxDist) new stations needed
        // Safe form: needed += (int)ceil(gap/maxDist) - 1;
    }
    return needed <= k;
}

double minmaxGasDist(vector<int>& stations, int k) {
    double lo = 0.0;
    double hi = 1e8;

    for (int iter = 0; iter < 100; iter++) {
        double mid = (lo + hi) / 2.0;
        if (canAchieve(stations, mid, k)) hi = mid;
        else                              lo = mid;
    }
    return lo;
}
```

| Scenario | Expected | Reason |
|---|---|---|
| `K = 0` | `max(consecutive gaps)` | No stations to add; just find existing max gap |
| All gaps equal, K = n-1 | `original_gap / 2` | Each gap gets one station |
| Very large K | Approaches 0.0 | Arbitrarily many stations fill every gap |

Time: O(100 × n) = O(n). Space: O(1).

---

## 8. Problem 6 — Median of Two Sorted Arrays (LC 4)

### Problem Statement

Given two sorted arrays `nums1` of size `m` and `nums2` of size `n`, return the median of their combined sorted array in O(log(m+n)) time.

**Constraints:** `0 <= m, n <= 1000`, `1 <= m + n <= 2000`, `-10^6 <= nums1[i], nums2[i] <= 10^6`.

**Examples:**
- `nums1 = [1,3]`, `nums2 = [2]` → `2.0` (merged: [1,2,3])
- `nums1 = [1,2]`, `nums2 = [3,4]` → `2.5` (merged: [1,2,3,4])

### Approach 1 — Merge and Find

Merge like merge sort, then pick the middle element(s).

```cpp
double findMedianSortedArrays_merge(vector<int>& nums1, vector<int>& nums2) {
    int m = nums1.size(), n = nums2.size();
    vector<int> merged;
    merged.reserve(m + n);
    int i = 0, j = 0;
    while (i < m && j < n) {
        if (nums1[i] <= nums2[j]) merged.push_back(nums1[i++]);
        else                       merged.push_back(nums2[j++]);
    }
    while (i < m) merged.push_back(nums1[i++]);
    while (j < n) merged.push_back(nums2[j++]);
    int total = m + n;
    if (total % 2 == 1) return merged[total / 2];
    return (merged[total / 2 - 1] + merged[total / 2]) / 2.0;
}
```

Time: O(m + n). Space: O(m + n). Violates the O(log(m+n)) requirement.

### Approach 2 — Binary Search on Partition (Optimal)

**The central idea — what the median means geometrically**

The median of a combined array of total length `T` is the element(s) at the middle of the sorted merged sequence. Instead of physically merging, we find where to *cut* each array so that the combined left halves contain exactly `(m+n+1)/2` elements and form the sorted left half of the merged array.

**Setting up the partition:** Let `cut1` = number of elements taken from `nums1` for the left partition. Then `cut2 = half - cut1` elements come from `nums2`, where `half = (m+n+1)/2`.

Define boundary elements with sentinels for empty partitions:
```
leftMax1  = (cut1 == 0) ? INT_MIN : nums1[cut1 - 1]
rightMin1 = (cut1 == m) ? INT_MAX : nums1[cut1]
leftMax2  = (cut2 == 0) ? INT_MIN : nums2[cut2 - 1]
rightMin2 = (cut2 == n) ? INT_MAX : nums2[cut2]
```

**Valid partition condition:** `leftMax1 <= rightMin2` AND `leftMax2 <= rightMin1`. When both hold, every element in the left partition is ≤ every element in the right partition — the partition correctly identifies the sorted left half. The median follows:
- Odd total: `max(leftMax1, leftMax2)` (the middle element)
- Even total: `(max(leftMax1, leftMax2) + min(rightMin1, rightMin2)) / 2.0`

**Binary search direction:**
- `leftMax1 > rightMin2`: `cut1` is too large (too many large elements from nums1 on the left). Move `hi = cut1 - 1`.
- `leftMax2 > rightMin1`: `cut1` is too small. Move `lo = cut1 + 1`.

**Why binary search on the smaller array?** We binary search `cut1` in `[0, m]`. Fixing `cut1` determines `cut2 = half - cut1`. Ensuring `nums1` is smaller guarantees `cut2` stays in `[0, n]` without extra bounds checking, and reduces the log factor to `O(log(min(m,n)))`.

**Why INT_MIN and INT_MAX as sentinels?** When `cut1 = 0`, the left partition of nums1 is empty. We pretend `leftMax1 = INT_MIN` so it never violates `leftMax1 <= rightMin2`. When `cut1 = m`, the right partition of nums1 is empty, so `rightMin1 = INT_MAX` never violates `leftMax2 <= rightMin1`. These sentinels model "no constraint from an empty partition."

```cpp
#include <bits/stdc++.h>
using namespace std;

double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
    if (nums1.size() > nums2.size()) swap(nums1, nums2);   // ensure nums1 is smaller

    int m = nums1.size(), n = nums2.size();
    int half = (m + n + 1) / 2;

    int lo = 0, hi = m;

    while (lo <= hi) {
        int cut1 = lo + (hi - lo) / 2;
        int cut2 = half - cut1;

        int leftMax1  = (cut1 == 0) ? INT_MIN : nums1[cut1 - 1];
        int rightMin1 = (cut1 == m) ? INT_MAX : nums1[cut1];
        int leftMax2  = (cut2 == 0) ? INT_MIN : nums2[cut2 - 1];
        int rightMin2 = (cut2 == n) ? INT_MAX : nums2[cut2];

        if (leftMax1 <= rightMin2 && leftMax2 <= rightMin1) {
            if ((m + n) % 2 == 1) {
                return max(leftMax1, leftMax2);
            } else {
                return (max(leftMax1, leftMax2) + min(rightMin1, rightMin2)) / 2.0;
            }
        } else if (leftMax1 > rightMin2) {
            hi = cut1 - 1;    // too much from nums1
        } else {
            lo = cut1 + 1;    // too little from nums1
        }
    }
    return 0.0;   // unreachable with valid input
}
```

**Dry run:** `nums1 = [1, 3]`, `nums2 = [2]`

```
nums1=[1,3] (m=2), nums2=[2] (n=1): m > n → swap
nums1=[2] (m=1), nums2=[1,3] (n=2)
half = (1+2+1)/2 = 2

lo=0, hi=1

cut1=0: cut2=2
  leftMax1=INT_MIN, rightMin1=nums1[0]=2
  leftMax2=nums2[1]=3, rightMin2=INT_MAX
  leftMax2(3) > rightMin1(2) → lo=1

cut1=1: cut2=1
  leftMax1=nums1[0]=2, rightMin1=INT_MAX
  leftMax2=nums2[0]=1, rightMin2=nums2[1]=3
  leftMax1(2) <= rightMin2(3) ✓
  leftMax2(1) <= rightMin1(INF) ✓
  (m+n)=3 odd → return max(2,1) = 2.0
```

**Dry run:** `nums1 = [1, 2]`, `nums2 = [3, 4]`

```
m=2, n=2, no swap. half = (2+2+1)/2 = 2

lo=0, hi=2
cut1=1: cut2=1
  leftMax1=1, rightMin1=2, leftMax2=3, rightMin2=4
  leftMax2(3) > rightMin1(2) → lo=2

cut1=2: cut2=0
  leftMax1=nums1[1]=2, rightMin1=INT_MAX
  leftMax2=INT_MIN, rightMin2=nums2[0]=3
  leftMax1(2) <= rightMin2(3) ✓; leftMax2(INT_MIN) <= rightMin1(INF) ✓
  (m+n)=4 even → (max(2,INT_MIN) + min(INF,3)) / 2.0 = (2+3)/2.0 = 2.5
```

| Scenario | Expected | Reason |
|---|---|---|
| `nums1 = []`, `nums2 = [1]` | `1.0` | Empty array handled by INT_MIN/INT_MAX |
| All of nums1 < all of nums2 | Boundary of nums1 | cut1 = m found early |
| Duplicates across arrays | Correct | Sentinels use extreme values, not array values |

Time: O(log(min(m, n))). Space: O(1).

---

## 9. Problem 7 — K-th Element of Two Sorted Arrays (GFG)

### Problem Statement

Given two sorted arrays `a[]` (size `n`) and `b[]` (size `m`), find the element at the k-th position (1-indexed) in the merged sorted array.

**Constraints:** `1 <= n, m <= 10^6`, `0 <= a[i], b[i] <= 10^9`, `1 <= k <= n + m`.

**Examples:**
- `a = [2,3,6,7,9]`, `b = [1,4,8,10]`, `k = 5` → `6` (merged: [1,2,3,4,6,7,8,9,10])
- `a = [100,112,256,349,770]`, `b = [72,86,113,119,265,445,892]`, `k = 7` → `256`

### Approach 1 — Pointer Merge

```cpp
int kthElement_merge(vector<int>& a, vector<int>& b, int k) {
    int i = 0, j = 0, cnt = 0;
    while (i < (int)a.size() && j < (int)b.size()) {
        int val = (a[i] <= b[j]) ? a[i++] : b[j++];
        if (++cnt == k) return val;
    }
    while (i < (int)a.size()) { if (++cnt == k) return a[i]; i++; }
    while (j < (int)b.size()) { if (++cnt == k) return b[j]; j++; }
    return -1;
}
```

Time: O(k). Worst case O(n + m). Space: O(1).

### Approach 2 — Binary Search on Partition (Optimal)

**Intuition:** The k-th element is the largest element in the left partition of the merged array when we place exactly `k` elements on the left. This is the median problem with a specific `k` instead of `(m+n)/2`. The partition logic is identical: find `cut1` such that the partition condition holds, then return `max(leftMax1, leftMax2)`.

**Why `lo = max(0, k-m)` and `hi = min(k, n)`?** `cut2 = k - cut1` must satisfy `0 <= cut2 <= m`: `cut2 >= 0` forces `cut1 <= k`, so `hi = min(k, n)`. `cut2 <= m` forces `cut1 >= k - m`, so `lo = max(0, k-m)`. These bounds ensure `cut2` is always a valid index into `b`.

```cpp
#include <bits/stdc++.h>
using namespace std;

int kthElement(vector<int>& a, vector<int>& b, int k) {
    int n = a.size(), m = b.size();
    if (n > m) return kthElement(b, a, k);   // always binary search on smaller

    int lo = max(0, k - m);
    int hi = min(k, n);

    while (lo <= hi) {
        int cut1 = lo + (hi - lo) / 2;
        int cut2 = k - cut1;

        int leftMax1  = (cut1 == 0) ? INT_MIN : a[cut1 - 1];
        int rightMin1 = (cut1 == n) ? INT_MAX : a[cut1];
        int leftMax2  = (cut2 == 0) ? INT_MIN : b[cut2 - 1];
        int rightMin2 = (cut2 == m) ? INT_MAX : b[cut2];

        if (leftMax1 <= rightMin2 && leftMax2 <= rightMin1) {
            return max(leftMax1, leftMax2);
        } else if (leftMax1 > rightMin2) {
            hi = cut1 - 1;
        } else {
            lo = cut1 + 1;
        }
    }
    return -1;   // unreachable with valid input
}
```

**Dry run:** `a = [2,3,6,7,9]` (n=5), `b = [1,4,8,10]` (m=4), `k = 5`

```
n=5 > m=4 → swap: a=[1,4,8,10] (n=4), b=[2,3,6,7,9] (m=5)

lo = max(0, 5-5) = 0
hi = min(5, 4) = 4

cut1=2: cut2=3
  leftMax1=a[1]=4, rightMin1=a[2]=8
  leftMax2=b[2]=6, rightMin2=b[3]=7
  4 <= 7 ✓; 6 <= 8 ✓
  → return max(4,6) = 6
```

Merged: [1,2,3,4,6,7,8,9,10]. 5th element = 6. Correct.

**Relationship to Median:** Median of combined array of length T is the kth element where k = (T+1)/2 for odd T, or the average of elements k and k+1 where k = T/2 for even T. The code is structurally identical to the median solution.

| Scenario | Expected | Reason |
|---|---|---|
| `k = 1` | `min(a[0], b[0])` | First element of merged array |
| `k = n + m` | `max(a[n-1], b[m-1])` | Last element |
| One array empty | Direct index into other | Sentinels handle cut = 0 or n |

Time: O(log(min(m, n))). Space: O(1).

---

## 10. Why Binary Search Works on Monotonic Answer Spaces — Formal Argument

**General correctness condition:** Binary search finds the first (or last) index where a predicate `f` transitions if and only if `f` is monotone over the search range.

**Why the partition predicates are monotone:**

For "minimize the maximum partition sum" (Book Allocation, Painter's Partition, Split Array):

Let `canDo(d)` = "can we partition the array into ≤ K parts, each with sum ≤ d?"

Claim: if `canDo(d)` is true, then `canDo(d')` is true for all `d' >= d`.

Proof: any valid partition for maximum `d` is also valid for `d' >= d`, since each part's sum ≤ d ≤ d'. The same partition witnesses feasibility. ∎

For "maximize the minimum distance" (Aggressive Cows):

Claim: if `canPlace(d)` is true (C cows placeable with minimum distance ≥ d), then `canPlace(d')` is true for all `d' <= d`.

Proof: any placement valid for minimum distance `d` is valid for `d' <= d`, since every pairwise distance ≥ d ≥ d'. ∎

**Why the greedy checker is optimal for partition problems:**

The greedy strategy (assign as many elements as possible to the current partition before starting a new one) minimizes the number of partitions used for a given budget `d`. Proof by exchange argument: suppose a non-greedy assignment uses the same or fewer partitions. Its first partition's sum is less than the greedy's. We can extend the first partition to match the greedy without violating the budget, and the remaining elements need the same or fewer partitions. By induction, the greedy assignment is always at least as efficient.

**Why the partition binary search (LC 4, GFG Kth) works:**

The valid partition condition (`leftMax1 <= rightMin2` AND `leftMax2 <= rightMin1`) characterizes a unique `cut1` value because:
- As `cut1` increases, `leftMax1` is non-decreasing and `leftMax2` is non-increasing. If `leftMax1 > rightMin2`, the left side of nums1 is too heavy, and we must decrease `cut1`. This creates a monotone search.
- The valid partition exists by a pigeonhole argument: at `cut1 = 0`, the condition `leftMax1 <= rightMin2` is trivially true (INT_MIN ≤ anything). At `cut1 = m`, `leftMax2 <= rightMin1` may fail. The transition happens exactly once.

---

## 11. Taxonomy of Binary Search on Answer Problems

| Problem | Flavor | lo | hi | Move when feasible |
|---|---|---|---|---|
| Aggressive Cows | Maximize minimum | 1 | max_pos − min_pos | `lo = mid + 1` |
| Book Allocation | Minimize maximum | `max(arr)` | `sum(arr)` | `hi = mid - 1` |
| Painter's Partition | Minimize maximum | `max(arr)` | `sum(arr)` | `hi = mid - 1` |
| Split Array | Minimize maximum | `max(arr)` | `sum(arr)` | `hi = mid - 1` |
| Gas Station | Minimize maximum (real) | `0.0` | `1e8` | `hi = mid` |
| Median (LC 4) | Partition search | 0 | `min(m,n)` | Depends on condition |
| Kth Element | Partition search | `max(0,k-m)` | `min(k,n)` | Depends on condition |

**Isomorphic problems (same code, different story):**

Book Allocation = Painter's Partition = Split Array Largest Sum = Capacity to Ship Packages (LC 1011) = Koko Eating Bananas (LC 875, slightly different bounds) = Find Smallest Divisor (LC 1283).

**Maximize minimum variants (same structure as Aggressive Cows):**

Magnetic Force Between Two Balls (LC 1552) = Divide Chocolate = Cutting Ribbons (LC 1891) = Maximum Candies per Child (LC 2226).

---

## 12. Common Mistakes and Pitfalls

### Mistake 1 — Wrong binary search bounds

```cpp
// WRONG — lo=0 allows mid=0 causing division-by-zero in gas station; also
// wastes ~log(max/max_element) = many iterations
int lo = 0, hi = INT_MAX;

// CORRECT — tight bounds reduce iterations and prevent degenerate mid values
long long lo = *max_element(arr.begin(), arr.end());
long long hi = accumulate(arr.begin(), arr.end(), 0LL);
```

### Mistake 2 — Integer overflow in accumulate

```cpp
// WRONG — overflows if n * arr[i] > INT_MAX
// e.g., n=10^5, arr[i]=10^8 → sum = 10^13 >> INT_MAX ≈ 2.1×10^9
int hi = accumulate(arr.begin(), arr.end(), 0);

// CORRECT — 0LL forces long long accumulation
long long hi = accumulate(arr.begin(), arr.end(), 0LL);
```

### Mistake 3 — Missing single-element validity check in feasibility function

```cpp
// WRONG — if arr[i] > maxPages, the greedy still assigns arr[i] to a new
// student and continues, potentially returning true falsely
bool canAllocate(vector<int>& arr, long long maxPages, int K) {
    int students = 1; long long pages = 0;
    for (int x : arr) {
        if (pages + x > maxPages) { students++; pages = x; }
        else pages += x;
    }
    return students <= K;
}

// CORRECT — single element exceeding the limit makes allocation impossible
for (int x : arr) {
    if ((long long)x > maxPages) return false;   // add this check first
    ...
}
```

### Mistake 4 — Wrong search direction for maximize-minimum vs minimize-maximum

```cpp
// WRONG — this is for minimize-maximum, not maximize-minimum
if (canPlace(stalls, mid, C)) { ans = mid; hi = mid - 1; }   // searching lower

// CORRECT for maximize-minimum (Aggressive Cows)
if (canPlace(stalls, mid, C)) { ans = mid; lo = mid + 1; }   // searching higher
```

The direction of movement when feasible is the single most important distinction between the two flavors.

### Mistake 5 — Forgetting K > N check in Book Allocation

```cpp
// Without this, the binary search runs and may return a wrong positive answer
if (K > n) return -1;
```

### Mistake 6 — Not swapping to ensure smaller array in Median / Kth Element

```cpp
// WRONG — if nums1 is larger, cut2 = half - cut1 may exceed n = nums2.size()
int lo = 0, hi = nums1.size();   // searching on larger array

// CORRECT
if (nums1.size() > nums2.size()) swap(nums1, nums2);
int lo = 0, hi = nums1.size();   // always search on smaller
```

### Mistake 7 — Wrong median formula for even total

```cpp
// WRONG — uses the elements at cut positions directly
return (nums1[cut1] + nums2[cut2]) / 2.0;

// CORRECT — median is at the boundary of the partition, not at the cut indices
return (max(leftMax1, leftMax2) + min(rightMin1, rightMin2)) / 2.0;
```

### Mistake 8 — Using `while(lo < hi)` for answer-space binary search

`while(lo <= hi)` with an explicit `ans` variable is the safest template. `while(lo < hi)` requires either upper-mid adjustment (`lo + (hi-lo+1)/2`) to avoid infinite loops, or careful termination analysis. For these problems, use `while(lo <= hi)` consistently.

### Mistake 9 — Sorting for Book Allocation / Painter's Partition

Never sort these arrays. The contiguous constraint means the original order is part of the problem definition. Sorting would change which partitions are valid.

---

## 13. Interview Reference — Patterns and Application Map

### When to reach for binary search on answer

| Signal in the problem | Likely approach |
|---|---|
| "Minimize the maximum..." | Binary search on answer (minimize-max template) |
| "Maximize the minimum..." | Binary search on answer (maximize-min template) |
| "K workers / painters / students" with contiguous constraint | Painter's Partition = Book Allocation |
| "...complete in at most T time..." | Binary search on T; feasibility = count completions |
| Two sorted arrays, find median or kth element | Binary search on partition position |
| Feasibility is monotone over a numeric range | Binary search on answer |

### Harder problems this pattern unlocks

- **Koko Eating Bananas (LC 875):** Minimize eating speed `s` such that all piles finish in `h` hours. Same as Book Allocation with `canDo(s)` = sum of `ceil(pile/s)` ≤ h.
- **Capacity to Ship Packages in D Days (LC 1011):** Identical to Book Allocation.
- **Magnetic Force Between Two Balls (LC 1552):** Identical to Aggressive Cows.
- **Divide Chocolate:** Maximize minimum sweetness with K+1 chunks. Same maximize-min structure.
- **Find Smallest Divisor (LC 1283):** Binary search on divisor d; predicate = sum of ceil(arr[i]/d) ≤ threshold.
- **Kth Smallest in Multiplication Table (LC 668):** Binary search on value d; predicate = count of elements ≤ d in table.
- **Median of K sorted arrays:** Binary search on value; predicate = count elements ≤ mid across all arrays.
- **LC 1095 Find in Mountain Array:** Find peak (LC 162), binary search ascending and descending halves. Partition logic from LC 4 generalizes here.

---

## 14. Complexity Cheat Sheet

| Problem | Brute Time | Optimal Time | Space | Search Range |
|---|---|---|---|---|
| Aggressive Cows | O(n log n + n × D) where D = max gap | O(n log n + n log D) | O(1) | [1, max_pos − min_pos] |
| Book Allocation | O(n × sum) | O(n log(sum)) | O(1) | [max(arr), sum(arr)] |
| Painter's Partition | O(n × sum) | O(n log(sum)) | O(1) | [max(arr), sum(arr)] |
| Split Array (LC 410) | O(n^2 × k) DP | O(n log(sum)) | O(1) | [max(arr), sum(arr)] |
| Gas Station (LC 774) | O((n+k) log n) heap | O(100n) = O(n) | O(1) | [0.0, 1e8] real |
| Median Two Arrays (LC 4) | O(m+n) merge | O(log(min(m,n))) | O(1) | partition [0, min(m,n)] |
| Kth Element (GFG) | O(k) pointer merge | O(log(min(m,n))) | O(1) | partition [max(0,k-m), min(k,n)] |

---

## 15. Quick Revision Cheat Sheet

This section is fully self-contained for night-before revision.

**Core template — minimize maximum:**
```
lo = max(arr), hi = sum(arr), ans = hi  [use long long]
while lo <= hi:
    mid = lo + (hi-lo)/2
    if canDo(arr, mid, k): ans = mid; hi = mid-1
    else: lo = mid+1
return ans
```

**Core template — maximize minimum:**
```
sort(stalls)
lo = 1, hi = stalls.back()-stalls.front(), ans = 0
while lo <= hi:
    mid = lo + (hi-lo)/2
    if canPlace(stalls, mid, C): ans = mid; lo = mid+1
    else: hi = mid-1
return ans
```

**Gas station (real-valued):**
```
for 100 iterations: mid = (lo+hi)/2
    if canAchieve(stations, mid, k): hi = mid
    else: lo = mid
return lo
Predicate: sum of (int)(gap/mid) <= k
```

**Feasibility checker (partition problems):**
```
parts=1, cur=0
for each x in arr:
    if x > mid: return false
    if cur+x > mid: parts++; cur=x; if parts>k: return false
    else: cur+=x
return true
```

**Median two arrays:**
```
ensure m <= n (swap if needed)
half = (m+n+1)/2; lo=0, hi=m
while lo <= hi:
    cut1 = (lo+hi)/2; cut2 = half-cut1
    leftMax1 = (cut1==0) ? INT_MIN : a[cut1-1]
    rightMin1 = (cut1==m) ? INT_MAX : a[cut1]
    leftMax2 = (cut2==0) ? INT_MIN : b[cut2-1]
    rightMin2 = (cut2==n) ? INT_MAX : b[cut2]
    if l1<=r2 && l2<=r1:
        odd: return max(l1,l2)
        even: return (max(l1,l2)+min(r1,r2))/2.0
    elif l1>r2: hi=cut1-1
    else: lo=cut1+1
```

**Kth element:** Same as median, but `cut2 = k - cut1`, `lo = max(0,k-m)`, `hi = min(k,n)`, return `max(leftMax1, leftMax2)` on valid partition.

**Monotonicity directions:**
- Minimize maximum → when feasible, shrink hi (try smaller)
- Maximize minimum → when feasible, grow lo (try larger)

**Overflow checklist:**
- Use `long long` for sums when `n × max_val > 2 × 10^9`
- `accumulate(arr.begin(), arr.end(), 0LL)` — the `0LL` is mandatory
- Mid formula: always `lo + (hi - lo) / 2`

**Isomorphic problems (one solution):**
Book Allocation = Painter's Partition = Split Array Largest Sum = Capacity to Ship = Koko Eating Bananas (bounds differ slightly)

**Kth Element is a generalization of Median:**
Median uses `k = (m+n)/2` or `(m+n+1)/2`; the partition code is identical.

**Two critical rules:**
1. Always sort before Aggressive Cows. Never sort for partition problems (order is part of the constraint).
2. Always check `K > N` for Book Allocation before running binary search.
