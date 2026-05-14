## Resources

- [Two Sum — Striver (YouTube)](https://youtu.be/UXDSeD9mN-k)
- [Sort Colors — Striver (YouTube)](https://youtu.be/tp8JIuCXBaU)
- [Majority Element — Striver (YouTube)](https://youtu.be/nP_ns3uSh80)
- [Maximum Subarray / Kadane's — Striver (YouTube)](https://youtu.be/w_KEocd__20)
- [Best Time to Buy and Sell Stock — Striver (YouTube)](https://youtu.be/excAOvwF_Wk)
- [Rearrange by Sign — Striver (YouTube)](https://youtu.be/h4aBagy4Uok)
- [LeetCode 1 — Two Sum](https://leetcode.com/problems/two-sum)
- [LeetCode 75 — Sort Colors](https://leetcode.com/problems/sort-colors)
- [LeetCode 169 — Majority Element](https://leetcode.com/problems/majority-element)
- [LeetCode 53 — Maximum Subarray](https://leetcode.com/problems/maximum-subarray)
- [LeetCode 121 — Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock)
- [LeetCode 2149 — Rearrange Array Elements by Sign](https://leetcode.com/problems/rearrange-array-elements-by-sign)

---

## Table of Contents

1. [Why These Problems Matter](#1-why-these-problems-matter)
2. [Two Sum (LC 1)](#2-two-sum-lc-1)
3. [Sort Colors — Dutch National Flag (LC 75)](#3-sort-colors--dutch-national-flag-lc-75)
4. [Majority Element — Boyer-Moore Voting (LC 169)](#4-majority-element--boyer-moore-voting-lc-169)
5. [Maximum Subarray — Kadane's Algorithm (LC 53)](#5-maximum-subarray--kadanes-algorithm-lc-53)
6. [Best Time to Buy and Sell Stock (LC 121)](#6-best-time-to-buy-and-sell-stock-lc-121)
7. [Rearrange Array Elements by Sign (LC 2149)](#7-rearrange-array-elements-by-sign-lc-2149)
8. [Underlying Patterns — Unified Reference](#8-underlying-patterns--unified-reference)
9. [Common Mistakes and Pitfalls](#9-common-mistakes-and-pitfalls)
10. [Interview Extensions and Harder Problems](#10-interview-extensions-and-harder-problems)
11. [Complexity Cheat Sheet](#11-complexity-cheat-sheet)
12. [Quick Revision Cheat Sheet](#12-quick-revision-cheat-sheet)

---

## 1. Why These Problems Matter

These six problems introduce five distinct algorithmic ideas that recur across the entire DSA sheet. Unlike the easy array problems — which were all variations of single-pass tracking — each medium problem requires a genuinely different mental model.

| Problem | Core Idea |
|---|---|
| Two Sum | Complement search via hash map: trade O(n) space for O(n²) → O(n) time |
| Sort Colors | Three-way partitioning: classify into invariant regions in one pass |
| Majority Element | Pairwise cancellation: the majority survives all cancellations |
| Max Subarray | Local optimality via reset: a negative-sum prefix is never worth extending |
| Stock Profit | Greedy min-tracking: the best sell is always paired with the lowest seen buy |
| Rearrange Sign | Index arithmetic: use destination index parity to interleave without extra work |

The deepest insight common to all of them: **optimal algorithms succeed by identifying the minimal state that must be carried forward and discarding everything else.** Kadane needs only the current sum. Boyer-Moore needs only a candidate and a count. The stock problem needs only the running minimum. Recognizing this reduction is what separates strong interview answers from weak ones.

---

## 2. Two Sum (LC 1)

### Problem Statement

Given an array `nums` and integer `target`, return the indices of two distinct numbers that add up to `target`. Exactly one solution exists. You may not use the same element twice.

```
nums = [2, 7, 11, 15], target = 9  →  [0, 1]   (2+7=9)
nums = [3, 2, 4],      target = 6  →  [1, 2]   (2+4=6)
nums = [3, 3],         target = 6  →  [0, 1]   (same value, different indices)
```

### Approach 1: Brute Force

Check every pair (i, j) with i < j.

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    int n = nums.size();
    for (int i = 0; i < n - 1; i++)
        for (int j = i + 1; j < n; j++)
            if (nums[i] + nums[j] == target)
                return {i, j};
    return {};
}
// Time: O(n^2), Space: O(1)
```

### Approach 2: One-Pass Hash Map (Optimal)

**Insight:** For each element `x`, we need `target - x` (the complement). If the complement has already been seen, we have our answer. If not, store `x` and continue.

**Why insert after checking:** Inserting before checking would allow using the same element twice. When we reach index `i`, the hash map contains only elements at indices `0` to `i-1`, so `seen[complement]` is always a different index from `i`.

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int, int> seen;   // value → index

    for (int i = 0; i < (int)nums.size(); i++) {
        int complement = target - nums[i];

        if (seen.count(complement)) {
            return {seen[complement], i};
        }

        seen[nums[i]] = i;   // insert AFTER checking
    }
    return {};
}
// Time: O(n) average, Space: O(n)
```

### Dry Run

```
nums = [2, 7, 11, 15], target = 9

i=0: complement=7.  seen={}. Not found. Store seen[2]=0.
i=1: complement=2.  seen={2:0}. FOUND. Return {seen[2], 1} = {0, 1}.
```

```
nums = [3, 3], target = 6

i=0: complement=3.  seen={}. Not found. Store seen[3]=0.
i=1: complement=3.  seen={3:0}. FOUND. Return {0, 1}.
(Insert-after-check prevents returning {1, 1} — seen[3]=0 at i=1, not 1.)
```

```
nums = [3, 2, 4], target = 6

i=0: complement=3.  seen={}. Not found. Store seen[3]=0.
i=1: complement=4.  seen={3:0}. Not found. Store seen[2]=1.
i=2: complement=2.  seen={3:0, 2:1}. FOUND. Return {1, 2}.
```

### Variant: Two Sum II (Sorted Input, LC 167)

When the input is sorted, no hash map is needed. Use opposite-end two pointers.

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int sum = nums[lo] + nums[hi];
        if (sum == target) return {lo + 1, hi + 1};   // 1-indexed
        else if (sum < target) lo++;
        else hi--;
    }
    return {};
}
// Time: O(n), Space: O(1)
```

### Edge Cases

| Input | Target | Output | Note |
|---|---|---|---|
| `[2, 7, 11, 15]` | `9` | `[0, 1]` | Standard |
| `[3, 3]` | `6` | `[0, 1]` | Duplicate values, different indices |
| `[-1, -2, -3]` | `-5` | `[1, 2]` | Negative values |
| `[0, 4, 3, 0]` | `0` | `[0, 3]` | Zero values |

---

## 3. Sort Colors — Dutch National Flag (LC 75)

### Problem Statement

Given `nums` containing only 0s, 1s, and 2s, sort it in-place in a single pass without using library sort.

**Why this matters:** This is Dijkstra's **Dutch National Flag Problem** (1976). It is the canonical example of three-way partitioning — used in three-way quicksort and any problem requiring classification into three groups in one pass.

### Approach 1: Built-in Sort

`sort(nums.begin(), nums.end())` — O(n log n). Defeats the purpose.

### Approach 2: Counting Sort (Two Passes)

Count 0s, 1s, 2s. Overwrite. O(n) time, O(1) space.

```cpp
void sortColors(vector<int>& nums) {
    int c0 = 0, c1 = 0, c2 = 0;
    for (int x : nums) {
        if (x == 0) c0++;
        else if (x == 1) c1++;
        else c2++;
    }
    int i = 0;
    while (c0--) nums[i++] = 0;
    while (c1--) nums[i++] = 1;
    while (c2--) nums[i++] = 2;
}
// Time: O(n), Space: O(1), Two passes
```

Valid interview answer when the one-pass solution is not recalled.

### Approach 3: Dutch National Flag — One Pass (Optimal)

**Three-pointer invariant:**

```
[0, low)        →  all 0s  (confirmed)
[low, mid)      →  all 1s  (confirmed)
[mid, high]     →  unexamined (unknown)
(high, n-1]     →  all 2s  (confirmed)
```

`mid` scans left to right. At each step:
- `nums[mid] == 0`: swap with `nums[low]`, advance both `low` and `mid`. Safe to advance `mid` because whatever was at `low` before the swap came from the confirmed-1s region — a 1 moved to `mid`, which is fine.
- `nums[mid] == 1`: already in the correct region, advance `mid` only.
- `nums[mid] == 2`: swap with `nums[high]`, advance `high` backward. **Do NOT advance `mid`** — the element from `high` is unknown and must be examined next.

```cpp
void sortColors(vector<int>& nums) {
    int low = 0, mid = 0, high = (int)nums.size() - 1;

    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums[low], nums[mid]);
            low++;
            mid++;
        } else if (nums[mid] == 1) {
            mid++;
        } else {                        // nums[mid] == 2
            swap(nums[mid], nums[high]);
            high--;
            // mid does NOT advance: element from high is unexamined
        }
    }
}
// Time: O(n) — each element visited exactly once, Space: O(1)
```

### Dry Run

```
nums = [2, 0, 2, 1, 1, 0],  low=0, mid=0, high=5

Step 1: nums[0]=2 → swap(nums[0], nums[5]): [0,0,2,1,1,2], high=4
        mid stays: nums[0]=0 came from unexamined region

Step 2: nums[0]=0 → swap(nums[0], nums[0]): no change, low=1, mid=1

Step 3: nums[1]=0 → swap(nums[1], nums[1]): no change, low=2, mid=2

Step 4: nums[2]=2 → swap(nums[2], nums[4]): [0,0,1,1,2,2], high=3
        mid stays: nums[2]=1 is unexamined

Step 5: nums[2]=1 → mid=3

Step 6: nums[3]=1 → mid=4
        mid(4) > high(3) → STOP

Result: [0,0,1,1,2,2]  ✓
```

### Why `mid` Must Not Advance After Swapping with `high`

When `nums[mid]` (a 2) is swapped with `nums[high]`, the element arriving at `mid` came from the unexamined region `[mid, high]`. It could be 0, 1, or 2. Advancing `mid` would skip this element permanently, producing a wrong result. Contrast with the swap involving `low`: the element arriving at `mid` came from `[low, mid)` which is the confirmed-1s region — so it is definitely a 1, and advancing `mid` is safe.

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[0]` | `[0]` | Single element |
| `[2, 1, 0]` | `[0, 1, 2]` | Fully reversed |
| `[1, 1, 1]` | `[1, 1, 1]` | All 1s — mid advances through, low/high never move |
| `[0, 0, 0]` | `[0, 0, 0]` | All same |

---

## 4. Majority Element — Boyer-Moore Voting (LC 169)

### Problem Statement

Given an array of size n, return the element appearing more than `⌊n/2⌋` times. This element is **guaranteed to exist**.

```
[2,2,1,1,1,2,2]  →  2   (appears 4 times > 7/2=3)
[3,2,3]          →  3
[1]              →  1
```

### Approach 1: Brute Force O(n²)

For each element, count its occurrences. Return when count > n/2.

### Approach 2: Hash Map O(n) time, O(n) space

```cpp
int majorityElement(vector<int>& nums) {
    unordered_map<int, int> freq;
    int n = nums.size();
    for (int x : nums) {
        if (++freq[x] > n / 2) return x;
    }
    return -1;
}
```

### Approach 3: Sort — O(n log n)

After sorting, the majority element must occupy index `n/2` (it appears more than half the time, so it spans the median).

```cpp
int majorityElement(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    return nums[nums.size() / 2];
}
```

### Approach 4: Boyer-Moore Voting (Optimal)

**Core idea — pairwise cancellation:** The majority element appears more than all other elements combined. If we pair every majority vote with a non-majority vote and cancel them, majority votes remain. Boyer-Moore simulates this without storing all votes.

Maintain `candidate` and `count`:
- If `count == 0`: the previous candidate has been fully cancelled. Adopt the current element as new candidate with `count = 1`.
- If `x == candidate`: same side — increment count.
- If `x != candidate`: cancellation — decrement count.

```cpp
int majorityElement(vector<int>& nums) {
    int candidate = nums[0], count = 0;

    for (int x : nums) {
        if (count == 0) {
            candidate = x;
            count = 1;
        } else if (x == candidate) {
            count++;
        } else {
            count--;
        }
    }

    return candidate;
    // If majority is NOT guaranteed, add verification:
    // int verify = 0;
    // for (int x : nums) if (x == candidate) verify++;
    // return (verify > nums.size() / 2) ? candidate : -1;
}
// Time: O(n), Space: O(1)
```

### Dry Run

```
nums = [2, 2, 1, 1, 1, 2, 2]

x=2: count==0 → candidate=2, count=1
x=2: x==candidate → count=2
x=1: x!=candidate → count=1
x=1: x!=candidate → count=0
x=1: count==0 → candidate=1, count=1
x=2: x!=candidate → count=0
x=2: count==0 → candidate=2, count=1

Return 2  ✓
```

```
nums = [1, 1, 2, 2, 1]

x=1: count==0 → candidate=1, count=1
x=1: match → count=2
x=2: cancel → count=1
x=2: cancel → count=0
x=1: count==0 → candidate=1, count=1

Return 1  ✓ (1 appears 3 times > 5/2=2)
```

### Why Boyer-Moore Works — Formal Argument

Let majority element `m` appear more than n/2 times; let all other elements together appear fewer than n/2 times.

Each time `count` decrements to 0, a group ends where no element has a strict majority within that group. These groups of cancellations consume at most n/2 elements from `m` and at most n/2 from the rest. Since `m` appears more than n/2 times, it cannot be completely cancelled — at least one uncancelled occurrence remains, and its candidate value is `m`.

### Critical Caveat

Boyer-Moore only gives the correct result when a majority element is guaranteed. If the problem does not guarantee this, run a verification pass after the voting phase to confirm the candidate exceeds n/2 occurrences.

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[1]` | `1` | Single element |
| `[6, 6, 6]` | `6` | All same |
| `[-1, -1, 2]` | `-1` | Negatives work |

---

## 5. Maximum Subarray — Kadane's Algorithm (LC 53)

### Problem Statement

Given integer array `nums`, find the subarray with the largest sum and return the sum. A subarray must contain at least one element.

```
[-2, 1, -3, 4, -1, 2, 1, -5, 4]  →  6   (subarray [4,-1,2,1])
[5, 4, -1, 7, 8]                  →  23  (entire array)
[-1]                               →  -1  (only element)
```

### The Core Insight — When to Reset

At each index `i`, the question is: should the maximum subarray ending at `i` extend from a previous start, or start fresh at `i`?

**If the sum of all previous elements is negative, including them only decreases the total.** Starting fresh at `i` is strictly better. This is the single observation that makes Kadane's algorithm work.

The formal recurrence:

```
maxEndingHere[i] = max(nums[i], maxEndingHere[i-1] + nums[i])
```

If `maxEndingHere[i-1]` is negative, `max(nums[i], negative + nums[i]) = nums[i]` — fresh start wins. If positive, extend. The answer is `max(maxEndingHere[0..n-1])`.

### Approach 1: Brute Force O(n²)

```cpp
int maxSubArray(vector<int>& nums) {
    int n = nums.size(), maxSum = INT_MIN;
    for (int i = 0; i < n; i++) {
        int sum = 0;
        for (int j = i; j < n; j++) {
            sum += nums[j];
            maxSum = max(maxSum, sum);
        }
    }
    return maxSum;
}
```

### Approach 2: Kadane's Algorithm (Optimal)

```cpp
int maxSubArray(vector<int>& nums) {
    int curSum = nums[0];
    int maxSum = nums[0];

    for (int i = 1; i < (int)nums.size(); i++) {
        curSum = max(nums[i], curSum + nums[i]);  // extend or restart
        maxSum = max(maxSum, curSum);
    }

    return maxSum;
}
// Time: O(n), Space: O(1)
```

Equivalent reset formulation (both are identical logic):

```cpp
int maxSubArray(vector<int>& nums) {
    int curSum = 0, maxSum = INT_MIN;
    for (int x : nums) {
        curSum += x;
        maxSum = max(maxSum, curSum);   // update BEFORE reset
        if (curSum < 0) curSum = 0;    // discard negative prefix
    }
    return maxSum;
}
```

**Why update `maxSum` before resetting:** For all-negative arrays like `[-2, -1]`, `curSum` will go negative after each element. If you reset before updating `maxSum`, you lose the actual maximum (e.g., `-1`). Always update `maxSum` first.

### Detailed Dry Run

```
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

curSum = -2, maxSum = -2

i=1: x=1
  curSum = max(1, -2+1) = max(1, -1) = 1
  maxSum = max(-2, 1) = 1

i=2: x=-3
  curSum = max(-3, 1-3) = max(-3, -2) = -2
  maxSum = max(1, -2) = 1

i=3: x=4
  curSum = max(4, -2+4) = max(4, 2) = 4
  maxSum = max(1, 4) = 4

i=4: x=-1
  curSum = max(-1, 4-1) = max(-1, 3) = 3
  maxSum = max(4, 3) = 4

i=5: x=2
  curSum = max(2, 3+2) = max(2, 5) = 5
  maxSum = max(4, 5) = 5

i=6: x=1
  curSum = max(1, 5+1) = max(1, 6) = 6
  maxSum = max(5, 6) = 6

i=7: x=-5
  curSum = max(-5, 6-5) = max(-5, 1) = 1
  maxSum = max(6, 1) = 6

i=8: x=4
  curSum = max(4, 1+4) = max(4, 5) = 5
  maxSum = max(6, 5) = 6

Return 6  ✓  (subarray [4,-1,2,1])
```

### Variant: Return the Subarray Indices, Not Just the Sum

```cpp
pair<int,int> maxSubarrayIndices(vector<int>& nums) {
    int curSum = nums[0], maxSum = nums[0];
    int start = 0, end = 0, tempStart = 0;

    for (int i = 1; i < (int)nums.size(); i++) {
        if (nums[i] > curSum + nums[i]) {
            curSum = nums[i];
            tempStart = i;        // potential new start
        } else {
            curSum += nums[i];
        }
        if (curSum > maxSum) {
            maxSum = curSum;
            start = tempStart;
            end = i;
        }
    }
    return {start, end};
}
```

### Kadane's as DP

Kadane's is a DP with O(1) space because `dp[i]` depends only on `dp[i-1]`, compressing the dp array to one variable. The DP recurrence: `dp[i] = max(nums[i], dp[i-1] + nums[i])`. The answer is `max(dp[0..n-1])`.

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[-1]` | `-1` | Single negative — must include at least one element |
| `[-2, -1]` | `-1` | All negative — pick the least negative |
| `[5, 4, -1, 7, 8]` | `23` | Entire array is optimal |

---

## 6. Best Time to Buy and Sell Stock (LC 121)

### Problem Statement

Given `prices[i]` = stock price on day `i`, return the maximum profit from one buy-sell transaction. You must buy before you sell. Return 0 if no profit is possible.

```
prices = [7, 1, 5, 3, 6, 4]  →  5   (buy at 1 on day 1, sell at 6 on day 4)
prices = [7, 6, 4, 3, 1]     →  0   (prices only fall, no profitable transaction)
```

### Connection to Kadane's Algorithm

This problem is equivalent to Maximum Subarray on the **differences array** `diff[i] = prices[i] - prices[i-1]`. The profit from buying on day `i` and selling on day `j` is `prices[j] - prices[i] = sum(diff[i+1..j])`. Maximum profit over one transaction = maximum subarray sum of differences.

Direct Kadane's formulation:

```cpp
int maxProfit(vector<int>& prices) {
    int curGain = 0, maxGain = 0;
    for (int i = 1; i < (int)prices.size(); i++) {
        int dailyChange = prices[i] - prices[i-1];
        curGain = max(0, curGain + dailyChange);   // Kadane on differences
        maxGain = max(maxGain, curGain);
    }
    return maxGain;
}
```

The more intuitive formulation tracks the running minimum directly.

### Approach 1: Brute Force O(n²)

```cpp
int maxProfit(vector<int>& prices) {
    int n = prices.size(), maxP = 0;
    for (int i = 0; i < n - 1; i++)
        for (int j = i + 1; j < n; j++)
            maxP = max(maxP, prices[j] - prices[i]);
    return maxP;
}
```

### Approach 2: Greedy — Track Running Minimum (Optimal)

**Insight:** For any selling day `j`, the maximum profit is `prices[j] - min(prices[0..j-1])`. Maintain the running minimum as we scan, and compute the profit at each sell day in O(1).

```cpp
int maxProfit(vector<int>& prices) {
    int minPrice = INT_MAX, maxProfit = 0;

    for (int price : prices) {
        minPrice = min(minPrice, price);               // best buy price seen so far
        maxProfit = max(maxProfit, price - minPrice);  // best profit if selling today
    }

    return maxProfit;
}
// Time: O(n), Space: O(1)
// maxProfit initialized to 0 — handles no-profit case (strictly decreasing prices)
```

### Dry Run

```
prices = [7, 1, 5, 3, 6, 4]
minPrice=INT_MAX, maxProfit=0

price=7: minPrice=7,  maxProfit=max(0, 7-7)=0
price=1: minPrice=1,  maxProfit=max(0, 1-1)=0
price=5: minPrice=1,  maxProfit=max(0, 5-1)=4
price=3: minPrice=1,  maxProfit=max(4, 3-1)=4
price=6: minPrice=1,  maxProfit=max(4, 6-1)=5
price=4: minPrice=1,  maxProfit=max(5, 4-1)=5

Return 5  ✓
```

```
prices = [7, 6, 4, 3, 1]

All prices fall. At each step, price - minPrice <= 0.
maxProfit stays 0 throughout.

Return 0  ✓
```

### Why Greedy Works

`minPrice` tracks `min(prices[0..i-1])` when we evaluate selling at index `i`. The constraint that we must buy before we sell is automatically satisfied: `minPrice` is always from a strictly earlier day. At each day we compute the best possible profit for that sell day, and take the global maximum.

### Stock Problem Variant Series

| Variant | LeetCode | Constraint | Key Approach |
|---|---|---|---|
| LC 121 (this) | 121 | 1 transaction | Greedy min-tracking |
| Unlimited transactions | 122 | Unlimited | Greedy: sum all upward daily moves |
| At most 2 transactions | 123 | ≤ 2 | DP with 4 states |
| At most k transactions | 188 | ≤ k | DP with 2k states |
| With cooldown | 309 | Unlimited, 1-day cooldown | DP with hold/cooldown/sold states |
| With fee | 714 | Unlimited, fee per transaction | Greedy DP |

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[7,6,4,3,1]` | `0` | Strictly decreasing |
| `[1,2]` | `1` | Minimum case with profit |
| `[2,4,1]` | `2` | Buy day 0, sell day 1 |
| `[1]` | `0` | Single day, no sell possible |

---

## 7. Rearrange Array Elements by Sign (LC 2149)

### Problem Statement

Given `nums` of even length with equal counts of positive and negative integers, rearrange so positive and negative elements alternate (positive at index 0, negative at index 1, etc.), preserving the relative order within each sign group.

```
nums = [3, 1, -2, -5, 2, -4]
Output: [3, -2, 1, -5, 2, -4]

nums = [-1, 1]
Output: [1, -1]
```

Constraints: length is even, equal positives and negatives, all values non-zero.

### Approach 1: Separate and Merge

Collect positives and negatives into separate arrays. Interleave into result.

```cpp
vector<int> rearrangeArray(vector<int>& nums) {
    vector<int> pos, neg;
    for (int x : nums) {
        if (x > 0) pos.push_back(x);
        else neg.push_back(x);
    }
    vector<int> result(nums.size());
    for (int i = 0; i < (int)pos.size(); i++) {
        result[2 * i]     = pos[i];   // even indices: positives
        result[2 * i + 1] = neg[i];   // odd  indices: negatives
    }
    return result;
}
// Time: O(n), Space: O(n)
```

### Approach 2: Single Pass, Two Destination Indices (Optimal)

**Insight:** Positives belong at even indices (0, 2, 4, ...) and negatives at odd indices (1, 3, 5, ...). Maintain two pointers that advance by 2 each time. One pass places every element directly.

```cpp
vector<int> rearrangeArray(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n);
    int posIdx = 0;   // next even index for positives
    int negIdx = 1;   // next odd index for negatives

    for (int x : nums) {
        if (x > 0) {
            result[posIdx] = x;
            posIdx += 2;
        } else {
            result[negIdx] = x;
            negIdx += 2;
        }
    }

    return result;
}
// Time: O(n), Space: O(n) — output array unavoidable
```

### Dry Run

```
nums = [3, 1, -2, -5, 2, -4]
posIdx=0, negIdx=1, result=[_,_,_,_,_,_]

x=3:   pos → result[0]=3,  posIdx=2
x=1:   pos → result[2]=1,  posIdx=4
x=-2:  neg → result[1]=-2, negIdx=3
x=-5:  neg → result[3]=-5, negIdx=5
x=2:   pos → result[4]=2,  posIdx=6
x=-4:  neg → result[5]=-4, negIdx=7

result = [3, -2, 1, -5, 2, -4]  ✓
```

### Why Relative Order is Preserved

Positives are placed at even indices in the order they appear in `nums` (posIdx increases: 0, 2, 4, ...). Negatives are placed at odd indices in their original order (negIdx increases: 1, 3, 5, ...). Since we scan `nums` left to right and place into monotonically increasing index positions within each group, relative order is maintained.

### Note on O(n) Space

This problem inherently requires O(n) extra space. An in-place alternating arrangement would require moving elements to non-adjacent positions, which has no clean swap structure. The result array is the correct and expected approach.

### Variant: Unequal Counts of Positives and Negatives

When counts differ, interleave until the shorter group is exhausted, then append the remainder. This handles general cases not covered by LC 2149's constraints.

---

## 8. Underlying Patterns — Unified Reference

### Pattern 1: Complement Search via Hash Map

```
Problem: "Find two (or more) elements satisfying a sum/xor/product condition."
Key idea: For each element x, the "partner" is f(x) = complement(x).
          If f(x) has been seen, we have an answer.
          If not, store x and continue.
Critical: Insert AFTER checking to prevent using the same index twice.

Template:
  unordered_map<T, int> seen;
  for each x at index i:
      if seen.count(complement(x)): return answer
      seen[x] = i
```

| Problem | complement(x) | Notes |
|---|---|---|
| Two Sum (LC 1) | `target - x` | Return indices |
| Two Sum II (LC 167) | Two pointers (sorted) | O(1) space |
| 3Sum (LC 15) | Fix one, inner two-pointer | Sort first |
| Subarray Sum = K (LC 560) | Prefix sum + hash map | Count subarrays |

### Pattern 2: Three-Way Partitioning (Dutch National Flag)

```
Problem: "Classify elements into three groups in-place, single pass."
Key idea: Three pointers maintain invariant boundaries.

[0, lo) = group A | [lo, mid) = group B | [mid, hi] = unknown | (hi, n-1] = group C

if nums[mid] == A: swap(lo, mid); lo++; mid++   (safe to advance mid)
if nums[mid] == B: mid++
if nums[mid] == C: swap(mid, hi); hi--          (mid stays — unexamined element arrived)
```

Appears in: three-way quicksort, segregation problems, any "partition into three buckets" problem.

### Pattern 3: Pairwise Cancellation (Boyer-Moore Family)

```
Problem: "Find the dominant element."
Key idea: Each (dominant, non-dominant) pair cancels.
          The dominant element survives all cancellations.

For majority > n/2:   1 candidate, count
For majority > n/3:   2 candidates, 2 counts (LC 229)

Three-way cancellation: each triple (cand1, cand2, other) cancels one from each.
```

### Pattern 4: Greedy with Running Optimum (Kadane / Stock)

```
Problem: "Maximize objective over a subarray or interval."
Key idea: At each step, make the locally optimal choice.
          Kadane:  if extending the subarray hurts (negative prefix), restart.
          Stock:   track the minimum price seen so far.

Both reduce to:
  current = max(current + delta, restart_value)
  answer  = max(answer, current)

State needed: only the current running value (curSum or minPrice).
Everything else can be discarded.
```

### Pattern 5: Index Arithmetic for Interleaving

```
Problem: "Place elements from two groups into alternating positions."
Key idea: Positives → even indices (0, 2, 4, ...)
          Negatives → odd indices (1, 3, 5, ...)
          Two pointers advance by 2, not 1.

Generalizes to: k-spaced positions, round-robin assignments, circular interleaving.
```

### State Minimization — The Unifying Insight

| Algorithm | Minimal necessary state | What is discarded |
|---|---|---|
| Kadane's | Current subarray sum | All previous sums except the running value |
| Boyer-Moore | Candidate + count | Full frequency map |
| Stock greedy | Minimum price so far | All previous prices |
| Two Sum | Seen complements | No extra traversal needed |
| Dutch National Flag | lo, mid, hi boundaries | No counting needed |

---

## 9. Common Mistakes and Pitfalls

### Mistake 1: Two Sum — Checking map with `[]` instead of `.count()`

```cpp
// WRONG: creates a default entry (0) if key absent
if (seen[complement] != -1) { ... }
// For complement = 0: seen[0] returns 0 by default if absent, same as index 0 — false positive.

// CORRECT: use .count() or .find()
if (seen.count(complement)) { return {seen[complement], i}; }
```

### Mistake 2: Two Sum — Inserting before checking (same-index collision)

```cpp
// WRONG: for nums=[3,3], target=6, at i=0 we insert seen[3]=0,
//        then check: complement=3 is in seen! Returns {0,0} — same index used twice.
seen[nums[i]] = i;
if (seen.count(complement)) return {seen[complement], i};

// CORRECT: check first, then insert
if (seen.count(complement)) return {seen[complement], i};
seen[nums[i]] = i;
```

### Mistake 3: Dutch National Flag — Advancing `mid` after swapping with `high`

```cpp
// WRONG: element arriving from high is unexamined; skipping it corrupts the result
} else {   // nums[mid] == 2
    swap(nums[mid], nums[high]);
    high--;
    mid++;   // BUG: must NOT advance mid here

// CORRECT
} else {
    swap(nums[mid], nums[high]);
    high--;
    // mid does not change
}
```

### Mistake 4: Kadane's — Updating `maxSum` after the reset instead of before

```cpp
// WRONG: for all-negative arrays, maxSum never gets updated
if (curSum < 0) curSum = 0;
maxSum = max(maxSum, curSum);   // curSum is already 0 — wrong for [-2,-1]

// CORRECT: update maxSum BEFORE the reset
maxSum = max(maxSum, curSum);
if (curSum < 0) curSum = 0;
```

### Mistake 5: Boyer-Moore — Not setting count to 1 when adopting new candidate

```cpp
// WRONG: count stays 0 after adoption; then if x==candidate: count becomes -1 (wrong)
if (count == 0) candidate = x;   // missing: count = 1
if (x == candidate) count++;
else count--;

// CORRECT: set count to 1 when adopting
if (count == 0) { candidate = x; count = 1; }
else if (x == candidate) count++;
else count--;
```

### Mistake 6: Stock — Not handling the "no profit" case

```cpp
// WRONG: if all prices fall, this returns INT_MIN - INT_MAX (overflow)
int maxP = INT_MIN;
for (int p : prices) maxP = max(maxP, p - minPrice);

// CORRECT: initialize maxProfit to 0 (no transaction = 0 profit)
int maxProfit = 0;
for (int p : prices) maxProfit = max(maxProfit, p - minPrice);
```

### Mistake 7: Boyer-Moore — Applying it when majority is not guaranteed

Boyer-Moore always returns some candidate. If the majority element is not guaranteed to exist, the candidate returned may not be the majority at all. Always add a verification pass when the problem does not guarantee existence.

```cpp
// Verification pass (add when majority not guaranteed)
int verify = 0;
for (int x : nums) if (x == candidate) verify++;
return (verify > (int)nums.size() / 2) ? candidate : -1;
```

---

## 10. Interview Extensions and Harder Problems

### Two Sum Extensions

| Problem | LeetCode | Key extension |
|---|---|---|
| Two Sum II (sorted) | 167 | O(1) space via two pointers |
| 3Sum | 15 | Sort + fix one + two-pointer inner loop |
| 4Sum | 18 | Sort + fix two + two-pointer inner loop |
| Subarray Sum Equals K | 560 | Prefix sum + hash map; count, not find |
| Longest Subarray with Sum K | — | Prefix sum + first-seen hash map |

### Kadane Extensions

| Problem | LeetCode | Extension |
|---|---|---|
| Max Product Subarray | 152 | Track both max and min (negatives flip) |
| Maximum Sum Circular Subarray | 918 | `max(linear_max, total_sum - linear_min)` |
| Maximum Submatrix Sum | — | Kadane on column prefix sums (2D extension) |

### Stock Series

| Problem | LeetCode | Transactions | Approach |
|---|---|---|---|
| Buy/Sell I | 121 | 1 | Greedy min-tracking |
| Buy/Sell II | 122 | Unlimited | Sum all upward daily moves |
| Buy/Sell III | 123 | ≤ 2 | DP: 4 states |
| Buy/Sell IV | 188 | ≤ k | DP: 2k states |
| With cooldown | 309 | Unlimited | DP: hold/cooldown/sold |
| With fee | 714 | Unlimited | Greedy DP |

### Majority Element Extensions

| Problem | Threshold | Candidates | LeetCode |
|---|---|---|---|
| Majority Element | > n/2 | 1 | 169 |
| Majority Element II | > n/3 | 2 | 229 |

For LC 229, maintain two candidates and two counts. Any triple `(cand1, cand2, other)` cancels one from each candidate. At most two elements can exceed n/3, so two candidates suffice.

### Dutch National Flag Applications

- **Three-way quicksort partition:** Elements equal to the pivot go into a confirmed-equal middle section, eliminating O(n²) worst case on arrays with many equal elements.
- **Segregate negatives and positives:** Simplified two-group DNF (no middle class).
- **Sort array of 0s and 1s:** Even simpler: two-way partition with lo and hi.

---

## 11. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Two Sum | Brute force | O(n²) | O(1) |
| Two Sum | One-pass hash map | O(n) avg | O(n) |
| Two Sum (sorted) | Two pointers | O(n) | O(1) |
| Sort Colors | Counting sort (2 pass) | O(n) | O(1) |
| Sort Colors | Dutch National Flag | O(n) | O(1) |
| Majority Element | Brute force | O(n²) | O(1) |
| Majority Element | Hash map | O(n) | O(n) |
| Majority Element | Sort (median) | O(n log n) | O(1) |
| Majority Element | Boyer-Moore | O(n) | O(1) |
| Maximum Subarray | O(n²) brute | O(n²) | O(1) |
| Maximum Subarray | Kadane's | O(n) | O(1) |
| Stock Buy/Sell | Brute force | O(n²) | O(1) |
| Stock Buy/Sell | Greedy min-track | O(n) | O(1) |
| Rearrange by Sign | Single pass, 2 indices | O(n) | O(n) |

---

## 12. Quick Revision Cheat Sheet

This section is self-contained for same-day revision.

---

### Two Sum — One-Pass Hash Map

```cpp
unordered_map<int, int> seen;
for (int i = 0; i < (int)nums.size(); i++) {
    int comp = target - nums[i];
    if (seen.count(comp)) return {seen[comp], i};
    seen[nums[i]] = i;      // insert AFTER checking — prevents same-index use
}
```

---

### Sort Colors — Dutch National Flag

```cpp
int lo=0, mid=0, hi=(int)nums.size()-1;
while (mid <= hi) {
    if      (nums[mid] == 0) { swap(nums[lo++], nums[mid++]); }
    else if (nums[mid] == 1) { mid++; }
    else                     { swap(nums[mid], nums[hi--]); }
    // mid does NOT advance when swapping with hi
}
```

---

### Majority Element — Boyer-Moore

```cpp
int candidate = nums[0], count = 0;
for (int x : nums) {
    if      (count == 0)    { candidate = x; count = 1; }
    else if (x == candidate){ count++; }
    else                    { count--; }
}
return candidate;
// Only correct when majority is GUARANTEED. Add verification pass otherwise.
```

---

### Maximum Subarray — Kadane's

```cpp
int cur = nums[0], best = nums[0];
for (int i = 1; i < (int)nums.size(); i++) {
    cur = max(nums[i], cur + nums[i]);   // extend or restart
    best = max(best, cur);
}
return best;
// Initialized to nums[0], not 0 — handles all-negative arrays correctly
```

---

### Best Time to Buy and Sell Stock

```cpp
int minPrice = INT_MAX, maxProfit = 0;
for (int p : prices) {
    minPrice  = min(minPrice, p);
    maxProfit = max(maxProfit, p - minPrice);
}
return maxProfit;
// maxProfit = 0 handles no-profit case (prices only fall)
```

---

### Rearrange by Sign

```cpp
vector<int> res(n);
int posIdx = 0, negIdx = 1;
for (int x : nums) {
    if (x > 0) { res[posIdx] = x; posIdx += 2; }
    else        { res[negIdx] = x; negIdx += 2; }
}
return res;
// Even indices: positives. Odd indices: negatives. Relative order preserved.
```

---

### Critical Invariants

```
Two Sum:      seen = {value: index} for all elements strictly before current index
DNF:          [0,lo) = 0s | [lo,mid) = 1s | [mid,hi] = unknown | (hi,n-1] = 2s
Boyer-Moore:  After full pass, candidate is the majority (if guaranteed to exist)
Kadane's:     cur = maximum subarray sum ending exactly at the current index
Stock:        minPrice = min(prices[0..i-1]) when evaluating selling at index i
```

---

### The Minimal State Insight

```
The strongest array optimizations come from identifying:
"What is the minimal information that must be carried forward?"

Kadane's:     curSum only           (not all previous sums)
Boyer-Moore:  candidate + count     (not a full frequency map)
Stock:        minPrice only         (not all previous prices)
Two Sum:      hash map of complements (not re-scanning the array)
DNF:          lo/mid/hi boundaries  (not counting elements first)
```
