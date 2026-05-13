**Resources:**
- [YouTube — Striver: Brute to Optimal](https://youtu.be/frf7qxiN2qU)

---

## Table of Contents

1. [Why This Problem Matters](#1-why-this-problem-matters)
2. [The Subarray Sum Identity: Prefix Sum Foundation](#2-the-subarray-sum-identity-prefix-sum-foundation)
3. [Problem Statement and Constraints](#3-problem-statement-and-constraints)
4. [Approach 1 — Brute Force](#4-approach-1--brute-force)
5. [Approach 2 — Prefix Sum with Hash Map (All Integers)](#5-approach-2--prefix-sum-with-hash-map-all-integers)
6. [Approach 3 — Sliding Window (Non-Negative Arrays Only)](#6-approach-3--sliding-window-non-negative-arrays-only)
7. [Why Sliding Window Fails for Negative Integers](#7-why-sliding-window-fails-for-negative-integers)
8. [Why Store Only the First Occurrence: Correctness Proof](#8-why-store-only-the-first-occurrence-correctness-proof)
9. [The {0: -1} Pre-Insert: Why It Is Necessary](#9-the-0--1-pre-insert-why-it-is-necessary)
10. [Variants and Extensions](#10-variants-and-extensions)
11. [Edge Cases](#11-edge-cases)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Interview Patterns](#13-interview-patterns)
14. [Quick Revision Cheat Sheet](#14-quick-revision-cheat-sheet)

---

## 1. Why This Problem Matters

This problem is not primarily about the specific question of finding the longest subarray with sum k. It is the entry point to a family of techniques — prefix sums, hash-based range queries, and the difference-of-prefix-sums identity — that underlie dozens of harder problems.

| Problem enabled by this foundation | Connection |
|---|---|
| Count subarrays with sum k (LC 560) | Frequency map instead of index map |
| Longest subarray with equal 0s and 1s (LC 525) | Transform 0→-1, find sum = 0 |
| Subarray sum divisible by k (LC 974) | Prefix sum modulo k |
| Maximum size subarray sum equals k (LC 325) | Identical to this problem |
| Prefix XOR problems | Replace sum with XOR |
| Difference arrays, Fenwick trees, segment trees | Generalizations of prefix sums |

The unifying idea behind all of them: **every subarray sum is the difference of two prefix sums**, which converts a repeated range computation into a constant-time hash map lookup.

---

## 2. The Subarray Sum Identity: Prefix Sum Foundation

Define the running prefix sum `S[i]` as the sum of all elements from index 0 through index i:

```
S[-1] = 0                              (virtual empty prefix)
S[i]  = arr[0] + arr[1] + ... + arr[i]
```

**The identity:** The sum of subarray `arr[l..r]` equals:

```
sum(arr[l..r]) = S[r] - S[l-1]
```

**Proof:** `S[r] = arr[0]+...+arr[r]` and `S[l-1] = arr[0]+...+arr[l-1]`. Subtracting cancels the common prefix, leaving `arr[l]+...+arr[r]`.

**Reframing the problem:** We want `sum(arr[l..r]) = k`, which by the identity is:

```
S[r] - S[l-1] = k
=>  S[l-1] = S[r] - k
```

For each right endpoint `r`, the question becomes: does there exist an earlier index `j = l-1` such that `S[j] = S[r] - k`? If yes, the subarray `arr[j+1..r]` has sum `k` and length `r - j`.

To **maximise** length, we want the **earliest** such `j`. This single insight drives the entire O(n) hash map solution.

**Example:**

```
arr = [10, 5, 2, 7, 1, -10],  k = 15

Prefix sums (index -1 through 5):
S[-1]=0, S[0]=10, S[1]=15, S[2]=17, S[3]=24, S[4]=25, S[5]=15

At i=5: S[5]=15, rem = 15-15 = 0 = S[-1].
  Subarray from (-1)+1=0 to 5, length 5-(-1)=6.
  arr[0..5] = [10,5,2,7,1,-10], sum = 15. Correct.
```

---

## 3. Problem Statement and Constraints

Given an array `arr[]` of `n` integers (may contain negatives and zeros) and an integer `k`, find the **length of the longest contiguous subarray** whose elements sum to exactly `k`. Return 0 if none exists.

```
Input:  arr = [10, 5, 2, 7, 1, -10],  k = 15
Output: 6     (entire array sums to 15)

Input:  arr = [-5, 8, -14, 2, 4, 12],  k = -5
Output: 5     (arr[0..4] = [-5,8,-14,2,4] sums to -5)

Input:  arr = [10, -10, 20, 30],  k = 5
Output: 0     (no subarray sums to 5)

Input:  arr = [1, 2, 3, 1, 1, 1, 1],  k = 3
Output: 5     (arr[2..6] = [3,1,1,1,1]? No. Let us verify via prefix map.)
```

**GFG Constraints:** `1 <= n <= 10^5`, `-10^4 <= arr[i] <= 10^4`, `-10^9 <= k <= 10^9`.

Maximum prefix sum with these constraints: `10^4 * 10^5 = 10^9`, which fits in a 32-bit int. Use `long long` as a safety habit when constraints are not immediately obvious.

---

## 4. Approach 1 — Brute Force

### Intuition

For every starting index `l`, extend the right boundary `r` one step at a time and maintain a running sum. Whenever the sum equals `k`, update the maximum length. No early termination is possible if the array contains negatives — a future negative element could reduce a sum that currently exceeds `k` back down to exactly `k`.

### Code

```cpp
int longestSubarray(vector<int>& arr, int k) {
    int n = arr.size();
    int maxLen = 0;

    for (int l = 0; l < n; l++) {
        long long sum = 0;
        for (int r = l; r < n; r++) {
            sum += arr[r];
            if (sum == k) {
                maxLen = max(maxLen, r - l + 1);
            }
            // Cannot break when sum > k: a future negative could bring sum back to k
        }
    }
    return maxLen;
}
```

**For positive-only arrays**, you can break early when `sum > k`:

```cpp
// Valid ONLY when all arr[i] > 0
if (sum > k) break;
```

### Dry Run

`arr = [10, 5, 2, 7, 1, -10]`, `k = 15`:

```
l=0: sum=10 | sum=15 -> maxLen=2 | sum=17 | sum=24 | sum=25 | sum=15 -> maxLen=6
l=1: sum=5  | sum=7  | sum=14   | sum=15 -> len=4 (no update) | sum=6
l=2: sum=2  | sum=9  | sum=10   | sum=0
l=3: sum=7  | sum=8  | sum=-2
l=4: sum=1  | sum=-9
l=5: sum=-10
maxLen = 6
```

### Complexity

| Metric | Value | Reason |
|---|---|---|
| Time | O(n²) | Two nested loops, each up to n |
| Space | O(1) | No auxiliary structure |

For n = 10⁵, this is 10¹⁰ operations — infeasible within any time limit.

---

## 5. Approach 2 — Prefix Sum with Hash Map (All Integers)

### Intuition

From Section 2: for each index `i` with running prefix sum `S`, we want to know whether any earlier index `j` has `S[j] = S - k`. A hash map stores each prefix sum value mapped to the **earliest index** at which it occurred, giving O(1) lookup.

For each `i`:
1. Update `prefSum += arr[i]`.
2. Check if `prefSum - k` is in the map. If yes, the subarray from `prefMap[prefSum-k]+1` to `i` has sum `k` and length `i - prefMap[prefSum-k]`.
3. If `prefSum` is not yet in the map, store `prefMap[prefSum] = i`. **Never overwrite** — first occurrence gives maximum length (proved in Section 8).

Pre-inserting `{0: -1}` handles the case where `arr[0..i]` itself sums to `k` (see Section 9).

### Code

```cpp
int longestSubarray(vector<int>& arr, int k) {
    unordered_map<long long, int> prefMap;
    prefMap[0] = -1;   // virtual empty prefix at index -1 has sum 0

    long long prefSum = 0;
    int maxLen = 0;

    for (int i = 0; i < (int)arr.size(); i++) {
        prefSum += arr[i];

        long long rem = prefSum - k;
        if (prefMap.count(rem)) {
            maxLen = max(maxLen, i - prefMap[rem]);
        }

        // Store first occurrence only — never overwrite
        if (!prefMap.count(prefSum)) {
            prefMap[prefSum] = i;
        }
    }
    return maxLen;
}
```

### Dry Run 1: `arr = [10, 5, 2, 7, 1, -10]`, `k = 15`

```
prefMap = {0:-1}, prefSum = 0, maxLen = 0

i=0, arr[0]=10:
  prefSum=10. rem=10-15=-5. Not in map.
  10 not in map -> prefMap[10]=0.
  Map: {0:-1, 10:0}

i=1, arr[1]=5:
  prefSum=15. rem=15-15=0. In map at -1.
  maxLen = max(0, 1-(-1)) = 2.
  15 not in map -> prefMap[15]=1.
  Map: {0:-1, 10:0, 15:1}

i=2, arr[2]=2:
  prefSum=17. rem=17-15=2. Not in map.
  prefMap[17]=2.

i=3, arr[3]=7:
  prefSum=24. rem=24-15=9. Not in map.
  prefMap[24]=3.

i=4, arr[4]=1:
  prefSum=25. rem=25-15=10. In map at 0.
  maxLen = max(2, 4-0) = 4.
  prefMap[25]=4.

i=5, arr[5]=-10:
  prefSum=15. rem=15-15=0. In map at -1.
  maxLen = max(4, 5-(-1)) = 6.
  15 already in map at 1 -> do NOT overwrite (preserving first occurrence).

Final maxLen = 6.   arr[0..5] sums to 15. Correct.
```

### Dry Run 2: `arr = [-5, 8, -14, 2, 4, 12]`, `k = -5`

```
prefMap = {0:-1}, prefSum = 0, maxLen = 0

i=0: prefSum=-5.  rem=-5-(-5)=0.  In map at -1. maxLen=0-(-1)=1. prefMap[-5]=0.
i=1: prefSum=3.   rem=3-(-5)=8.   Not in map.   prefMap[3]=1.
i=2: prefSum=-11. rem=-11-(-5)=-6. Not in map.  prefMap[-11]=2.
i=3: prefSum=-9.  rem=-9-(-5)=-4. Not in map.   prefMap[-9]=3.
i=4: prefSum=-5.  rem=-5-(-5)=0.  In map at -1. maxLen=max(1,4-(-1))=5.
     -5 already in map at 0 -> do NOT overwrite.
i=5: prefSum=7.   rem=7-(-5)=12.  Not in map.   prefMap[7]=5.

Final maxLen = 5.   arr[0..4] = [-5,8,-14,2,4], sum=-5. Correct.
```

### Dry Run 3: `arr = [1, 2, 3, 1, 1, 1, 1]`, `k = 3`

```
prefMap = {0:-1}, prefSum = 0, maxLen = 0

i=0: prefSum=1.  rem=1-3=-2. Not in map.  prefMap[1]=0.
i=1: prefSum=3.  rem=3-3=0.  In map at -1. maxLen=1-(-1)=2. prefMap[3]=1.
i=2: prefSum=6.  rem=6-3=3.  In map at 1. maxLen=max(2,2-1)=2.
     6 not in map -> prefMap[6]=2.
i=3: prefSum=7.  rem=7-3=4.  Not in map.  prefMap[7]=3.
i=4: prefSum=8.  rem=8-3=5.  Not in map.  prefMap[8]=4.
i=5: prefSum=9.  rem=9-3=6.  In map at 2. maxLen=max(2,5-2)=3. prefMap[9]=5.
i=6: prefSum=10. rem=10-3=7. In map at 3. maxLen=max(3,6-3)=3. prefMap[10]=6.

Final maxLen = 3.
Subarrays with sum 3: [1,2](len 2), [3](len 1), [1,1,1] at idx 4..6 (len 3). Correct.
```

### Complexity

| Metric | Value | Reason |
|---|---|---|
| Time | O(n) average | Single pass; hash map operations O(1) average |
| Space | O(n) | Map stores at most n+1 distinct prefix sums |

**Worst-case time:** O(n²) if all n prefix sums hash to the same bucket (hash collision). In competitive programming, use a custom hash or `map` (O(n log n)) to avoid adversarial inputs.

---

## 6. Approach 3 — Sliding Window (Non-Negative Arrays Only)

### Intuition

When all elements are non-negative (≥ 0), the prefix sum is **monotonically non-decreasing**. This means:

- Expanding the right pointer `r` can only increase or maintain the window sum.
- Shrinking the left pointer `l` can only decrease or maintain the window sum.

This monotone property makes the two-pointer technique valid: if the current window sum exceeds `k`, safely shrink from the left — no future right expansion can make the discarded left portion necessary again, because all elements are non-negative.

### Code

```cpp
int longestSubarrayNonNeg(vector<int>& arr, int k) {
    int n = arr.size();
    int l = 0;
    long long sum = 0;
    int maxLen = 0;

    for (int r = 0; r < n; r++) {
        sum += arr[r];   // expand right

        // Shrink left while sum exceeds k
        // Safe because arr[l] >= 0: removing it can only decrease (or maintain) sum
        while (sum > k && l <= r) {
            sum -= arr[l];
            l++;
        }

        if (sum == k) {
            maxLen = max(maxLen, r - l + 1);
        }
    }
    return maxLen;
}
```

### Why O(n) and Not O(n²)

Each element is added to the window exactly once (when `r` passes it) and removed at most once (when `l` passes it). Total pointer movements across the entire execution: at most `2n`. This is the standard amortized argument for all two-pointer algorithms.

### Dry Run: `arr = [1, 2, 3, 1, 1, 1, 1]`, `k = 3`

```
l=0, sum=0, maxLen=0

r=0: sum=1. 1<=3. 1!=3.
r=1: sum=3. 3<=3. sum==3. maxLen=max(0,1-0+1)=2.
r=2: sum=6. 6>3: sum-=arr[0]=1->sum=5,l=1. 5>3: sum-=arr[1]=2->sum=3,l=2.
     sum==3. maxLen=max(2,2-2+1)=2. (No improvement.)
r=3: sum=4. 4>3: sum-=arr[2]=3->sum=1,l=3. 1!=3.
r=4: sum=2. 2<=3. 2!=3.
r=5: sum=3. sum==3. maxLen=max(2,5-3+1)=3.
r=6: sum=4. 4>3: sum-=arr[3]=1->sum=3,l=4. sum==3. maxLen=max(3,6-4+1)=3.

Final maxLen = 3.  Correct (matches prefix sum result).
```

### Complexity

| Metric | Value | Reason |
|---|---|---|
| Time | O(n) | Each element enters and exits window at most once |
| Space | O(1) | Two pointers and a running sum only |

---

## 7. Why Sliding Window Fails for Negative Integers

**Counterexample:** `arr = [1, -1, 5, -2, 3]`, `k = 3`.

Correct answer: `arr[0..3] = [1,-1,5,-2]`, sum = 3, length = 4.

**Trace of sliding window on this array:**

```
l=0, sum=0

r=0: sum=1.  1<=3.  1!=3.
r=1: sum=0.  0<=3.  0!=3.
r=2: sum=5.  5>3: sum-=arr[0]=1->sum=4,l=1.  (valid removal)
             4>3: sum-=arr[1]=-1->sum=5,l=2.  (removed -1, sum INCREASED)
             5>3: sum-=arr[2]=5->sum=0,l=3.
             0!=3.
r=3: sum=-2. -2!=3.
r=4: sum=1.  1!=3.

Sliding window answer: 0.   WRONG. Correct answer: 4.
```

**Why it failed:** At `r=2`, the algorithm removed `arr[1] = -1` to shrink the window. But subtracting a negative element increases the sum (5 - (-1) = 6). The invariant — "shrinking the window reduces the sum" — is violated. The algorithm overshot past the valid window `arr[0..3]` and could never recover.

**The formal argument:** The sliding window relies on:

> If `sum(arr[l..r]) > k`, then shrinking to `sum(arr[l+1..r]) < sum(arr[l..r])`.

This holds if and only if `arr[l] > 0`. When `arr[l] <= 0`, removing it increases or maintains the sum, destroying the invariant.

**Conclusion:**

| Array contents | Correct algorithm |
|---|---|
| All non-negative (≥ 0) | Sliding window — O(n) time, O(1) space |
| Any negatives present | Prefix sum + hash map — O(n) avg, O(n) space |

---

## 8. Why Store Only the First Occurrence: Correctness Proof

This is the most commonly misunderstood detail of the hash map approach.

**Claim:** When the same prefix sum value `v` appears at multiple indices, the hash map should store only the **earliest** (smallest) index. Storing any later occurrence will produce a shorter or incorrect answer.

**Proof:** Suppose prefix sum value `v` first appears at index `j1` and again at `j2 > j1`. For a fixed right endpoint `i > j2`, both windows below satisfy sum = k:

```
arr[j1+1 .. i]  has sum = S[i] - S[j1] = S[i] - v = k,  length = i - j1
arr[j2+1 .. i]  has sum = S[i] - S[j2] = S[i] - v = k,  length = i - j2
```

Since `j1 < j2`, we have `i - j1 > i - j2`. The window starting at `j1+1` is strictly longer. We must prefer `j1`.

If the map stored `j2` (the later occurrence), we would compute length `i - j2` and miss the longer valid window of length `i - j1`.

**Concrete example:**

```
arr = [0, 1, 2, -3, 4],  k = 4

Prefix sums: S[-1]=0, S[0]=0, S[1]=1, S[2]=3, S[3]=0, S[4]=4

Value 0 appears at: index -1 (pre-insert) and index 0 and index 3.
First occurrence: -1.

At i=4: rem = S[4]-k = 4-4 = 0. First occurrence at -1.
  Length = 4 - (-1) = 5. arr[0..4] = [0,1,2,-3,4], sum=4. Correct.

If map stored last occurrence of 0 (index 3):
  Length = 4 - 3 = 1. arr[4..4] = [4], sum=4, length 1. Correct sum, wrong (shorter) length.

If map stored second occurrence of 0 (index 0):
  Length = 4 - 0 = 4. arr[1..4] = [1,2,-3,4], sum=4, length 4. Better but still not 5.
```

**Implementation rule:** Always check before inserting. Never overwrite.

```cpp
if (!prefMap.count(prefSum)) {
    prefMap[prefSum] = i;
}
```

---

## 9. The {0: -1} Pre-Insert: Why It Is Necessary

**The case it handles:** When `arr[0..i]` itself (the entire prefix from index 0) sums to `k`.

For this case, we need `S[l-1] = S[i] - k = 0` at some earlier index `j = l-1`. The "empty prefix" (no elements, before the array starts) has sum 0 and conceptually lives at index `-1`. Pre-inserting `prefMap[0] = -1` makes this available in the map.

**Without the pre-insert, this case is silently missed:**

```
arr = [3, 1, 2],  k = 3.

Without pre-insert:
  i=0: prefSum=3. rem=3-3=0. prefMap.count(0)=0. Missed. prefMap[3]=0.
  i=1: prefSum=4. rem=4-3=1. Not in map.
  i=2: prefSum=6. rem=6-3=3. In map at 0. maxLen=2-0=2.
  Answer: 2 (arr[1..2]=[1,2]). WRONG. Correct answer: 3 (arr[0..0]=[3], but also [3,1,2] is wrong, 3+1+2=6).
  Wait: arr=[3,1,2], k=3. arr[0..0]=[3], sum=3, length=1. arr[0..1]=[3,1], sum=4. arr[0..2]=[3,1,2], sum=6. arr[1..2]=[1,2], sum=3, length=2. Correct answer IS 2.
  
  But for arr=[3], k=3: without pre-insert, answer=0. With pre-insert: rem=0 in map at -1, length=0-(-1)=1. Correct.
```

**Two equivalent implementations:**

```cpp
// Option A (cleaner): pre-insert {0: -1}, handle all cases uniformly
prefMap[0] = -1;
// No explicit check for prefSum == k needed

// Option B: no pre-insert, explicit separate check
if (prefSum == k) maxLen = max(maxLen, i + 1);
// Then check prefMap for prefSum - k
```

Option A is standard and recommended because it unifies all cases: when `prefSum == k`, `rem = prefSum - k = 0`, which is found at index `-1`, giving length `i - (-1) = i + 1`. No separate branch needed.

---

## 10. Variants and Extensions

### Variant 1 — Count Subarrays with Sum K (LC 560)

**Difference from main problem:** Count all valid subarrays, not just find the longest. Store the **frequency** of each prefix sum (how many times it has appeared), not its first index. Update count by `+= prefCount[prefSum - k]` each time.

```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int, int> prefCount;
    prefCount[0] = 1;   // empty prefix: 1 way to have sum 0

    int prefSum = 0, count = 0;

    for (int x : nums) {
        prefSum += x;
        count += prefCount[prefSum - k];  // default 0 if absent
        prefCount[prefSum]++;             // update every occurrence
    }
    return count;
}
// Time: O(n), Space: O(n)
```

**Key difference from longest subarray:**

| Goal | Map value | Update rule |
|---|---|---|
| Longest subarray | First index | Insert only if not present |
| Count subarrays | Frequency | Increment every time |

### Variant 2 — Longest Subarray with Equal 0s and 1s (LC 525)

**Reduction:** Replace every 0 with -1. "Equal count of 0s and 1s" becomes "subarray sum = 0." Apply the main algorithm with k = 0.

```cpp
int findMaxLength(vector<int>& nums) {
    for (int& x : nums) if (x == 0) x = -1;   // transform

    unordered_map<int, int> prefMap;
    prefMap[0] = -1;
    int prefSum = 0, maxLen = 0;

    for (int i = 0; i < (int)nums.size(); i++) {
        prefSum += nums[i];
        if (prefMap.count(prefSum)) {
            maxLen = max(maxLen, i - prefMap[prefSum]);
        } else {
            prefMap[prefSum] = i;
        }
    }
    return maxLen;
}
// Time: O(n), Space: O(n)
```

**Dry run:** `nums = [0, 1, 0, 1, 1, 0]` → transformed: `[-1, 1, -1, 1, 1, -1]`

```
prefMap={0:-1}, prefSum=0

i=0: prefSum=-1. -1 not in map. prefMap[-1]=0.
i=1: prefSum=0.  0 in map at -1. maxLen=1-(-1)=2.
i=2: prefSum=-1. -1 in map at 0. maxLen=max(2,2-0)=2.
i=3: prefSum=0.  0 in map at -1. maxLen=max(2,3-(-1))=4.
i=4: prefSum=1.  1 not in map. prefMap[1]=4.
i=5: prefSum=0.  0 in map at -1. maxLen=max(4,5-(-1))=6.

Answer: 6. Entire array has 3 zeros and 3 ones. Correct.
```

### Variant 3 — Longest Subarray with Sum Divisible by K

**Key identity:** `sum(arr[l..r])` is divisible by `k` iff `prefSum[r] % k == prefSum[l-1] % k`. Store the first index of each `prefSum % k` value.

**Handling negative prefix sums in C++:** `(-7) % 3` evaluates to `-1` in C++11+ (truncation toward zero). Use `((sum % k) + k) % k` to normalise to `[0, k-1]`.

```cpp
int longestSubarrayDivK(vector<int>& arr, int k) {
    unordered_map<int, int> prefMap;
    prefMap[0] = -1;

    int sum = 0, maxLen = 0;

    for (int i = 0; i < (int)arr.size(); i++) {
        sum += arr[i];
        int rem = ((sum % k) + k) % k;   // always non-negative

        if (prefMap.count(rem)) {
            maxLen = max(maxLen, i - prefMap[rem]);
        } else {
            prefMap[rem] = i;
        }
    }
    return maxLen;
}
// Time: O(n), Space: O(k) — only k distinct remainders possible
```

### Variant 4 — Minimum Size Subarray Sum >= K (LC 209, positive arrays)

Find the **shortest** subarray with sum at least `k`. Valid for positive arrays only (same monotonicity argument as sliding window). Shrink whenever `sum >= target` (not just `> target`), and update minimum length during the shrink.

```cpp
int minSubArrayLen(int target, vector<int>& nums) {
    int l = 0, sum = 0, minLen = INT_MAX;
    for (int r = 0; r < (int)nums.size(); r++) {
        sum += nums[r];
        while (sum >= target) {
            minLen = min(minLen, r - l + 1);
            sum -= nums[l++];
        }
    }
    return (minLen == INT_MAX) ? 0 : minLen;
}
// Time: O(n), Space: O(1)
```

### Generalisation: The Prefix Aggregate Pattern

The prefix sum identity generalises to any associative operation where the subarray condition can be expressed as a function of a running aggregate:

| Subarray condition | Running aggregate | Map key |
|---|---|---|
| Sum = k | Prefix sum | prefSum - k |
| Sum divisible by k | Prefix sum mod k | (prefSum mod k) |
| Equal 0s and 1s (binary) | Prefix sum (0 → -1) | prefSum - 0 |
| XOR = k | Prefix XOR | prefXOR XOR k |

---

## 11. Edge Cases

| Input | Expected | Note |
|---|---|---|
| `arr = [k]`, single element equals k | 1 | `prefSum = k`, rem = 0, in map at -1, length = 0-(-1) = 1 |
| `arr = [0, 0, 0]`, k = 0 | 3 | All prefix sums are 0; first occurrence at -1; at i=2, length = 2-(-1) = 3 |
| All negatives, k negative | Correct if sum exists | Prefix hash map handles; sliding window fails |
| k > total sum of array | 0 | rem never found in map |
| k < minimum possible subarray sum | 0 | Same |
| All equal elements, k = one element | Subarray of length 1 | Also check for longer sums equaling k |
| Large values: arr[i] = 10⁴, n = 10⁵ | — | Max prefix sum = 10⁹, fits `int`; use `long long` defensively |
| arr[i] can be 0, k > 0 | Window may not shrink (sliding) | Zeros alone do not break sliding window; only negatives do |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1: Applying sliding window when array contains negatives

```cpp
// WRONG for arr = [1, -1, 5, -2, 3], k = 3
// Returns 0 instead of 4 (see Section 7 for the full trace)
while (sum > k && l <= r) {
    sum -= arr[l++];   // removing a negative INCREASES sum; invariant broken
}

// CORRECT: use prefix sum + hash map when negatives are possible
```

### Mistake 2: Overwriting existing map entries (last occurrence instead of first)

```cpp
// WRONG: always overwrites, losing the earliest occurrence
prefMap[prefSum] = i;

// CORRECT: insert only if not already present
if (!prefMap.count(prefSum)) {
    prefMap[prefSum] = i;
}
```

If prefix sum `v` first appears at `j1` and later at `j2 > j1`, the wrong code stores `j2`. For a later right endpoint `i`, it computes length `i - j2`, missing the correct maximum `i - j1`.

### Mistake 3: Omitting the {0: -1} pre-insert and not adding a substitute explicit check

```cpp
// WRONG: misses the case where arr[0..i] itself sums to k
unordered_map<int,int> prefMap;
// Missing: prefMap[0] = -1;
for (int i = 0; i < n; i++) {
    prefSum += arr[i];
    if (prefMap.count(prefSum - k))       // when prefSum==k, looks for 0, not found
        maxLen = max(maxLen, i - prefMap[prefSum - k]);
    if (!prefMap.count(prefSum)) prefMap[prefSum] = i;
}

// CORRECT option A: pre-insert
prefMap[0] = -1;

// CORRECT option B: explicit check instead of pre-insert
if (prefSum == k) maxLen = max(maxLen, i + 1);
```

**Concrete failure:** `arr = [3]`, `k = 3`. Without fix: answer = 0. Correct: 1.

### Mistake 4: Off-by-one in the length formula

```cpp
// The subarray from index (j+1) to index i has length i - j, NOT i - j + 1.
int j = prefMap[prefSum - k];  // j is the index where S[j] was recorded
int len = i - j;               // CORRECT: subarray starts at j+1, ends at i

// Wrong: i - j + 1 counts j itself, which is BEFORE the subarray starts
```

The map stores the index where a prefix sum ends. The subarray starts at the **next** index. So the length is `i - j` (end minus one-before-start), not `i - j + 1`.

### Mistake 5: Using prefMap[key] for existence check (creates entry if absent)

```cpp
// WRONG: operator[] inserts a default-value (0) entry if key is absent
if (prefMap[prefSum - k] != 0) { ... }  // 0 could be a valid index

// CORRECT: use .count() or .find()
if (prefMap.count(prefSum - k)) { ... }
```

### Mistake 6: Forgetting long long for prefix sums

```cpp
// POTENTIALLY WRONG: int overflow when arr[i] ~ 10^9 and n ~ 10^5
int prefSum = 0;   // max ~ 10^14, overflows int

// CORRECT:
long long prefSum = 0;
unordered_map<long long, int> prefMap;
```

For GFG constraints (`arr[i] <= 10^4`, `n <= 10^5`), max prefix sum is `10^9` which fits `int`. For safety in general, always use `long long`.

### Mistake 7: Confusing subarray (contiguous) with subsequence (non-contiguous)

The problem requires contiguous elements. Prefix sums only work for contiguous subarrays. Non-contiguous subsequences cannot be expressed as a prefix difference.

---

## 13. Interview Patterns

### Decision Tree

```
Is the problem about a contiguous subarray with a sum condition?
  YES -> Consider prefix sum identity: sum(arr[l..r]) = S[r] - S[l-1]
         Can elements be negative?
           NO  (all non-negative): Sliding window. O(n) time, O(1) space.
           YES (negatives possible): Prefix sum + hash map. O(n) avg, O(n) space.
         
         Is the goal to maximise length?
           YES -> Store first occurrence in map. Never overwrite.
         Is the goal to count subarrays?
           YES -> Store frequency in map. Update every occurrence.
```

### What to Say in an Interview

1. Ask: "Can elements be negative?"
2. If no negatives: offer sliding window as the optimal approach, mention the prefix hash map also works.
3. If negatives possible: go directly to prefix sum + hash map. Do not attempt sliding window.
4. State explicitly: "I will pre-insert `{0: -1}` to handle the case where the entire prefix sums to k."
5. State explicitly: "I will store only the first occurrence of each prefix sum to maximise subarray length."
6. Declare `long long` for prefix sums as a precaution.

### Connections to Harder Problems

| Harder problem | How this problem connects |
|---|---|
| Count subarrays with sum k (LC 560) | Change map from index to frequency |
| Longest subarray equal 0s/1s (LC 525) | Transform + apply this exact algorithm with k=0 |
| Minimum window subarray (LC 209) | Sliding window, minimise instead of maximise |
| Subarray sum divisible by k (LC 974) | Prefix sum modulo k, same map structure |
| Range sum queries (Fenwick, segment tree) | Prefix sum is the foundation |
| Two Sum (LC 1) | Structurally identical: find two values summing to target using hash map |

---

## 14. Quick Revision Cheat Sheet

**Core identity:** `sum(arr[l..r]) = S[r] - S[l-1]`

Reframed: we want `S[l-1] = S[r] - k`. For each `r`, look up whether `S[r] - k` was seen earlier.

---

**Algorithm selection:**

| Array type | Algorithm | Time | Space |
|---|---|---|---|
| All non-negative | Sliding window | O(n) | O(1) |
| Any negatives | Prefix sum + hash map | O(n) avg | O(n) |

---

**Prefix sum + hash map template:**

```cpp
int longestSubarray(vector<int>& arr, int k) {
    unordered_map<long long, int> prefMap;
    prefMap[0] = -1;          // empty prefix at virtual index -1

    long long prefSum = 0;
    int maxLen = 0;

    for (int i = 0; i < (int)arr.size(); i++) {
        prefSum += arr[i];
        if (prefMap.count(prefSum - k))
            maxLen = max(maxLen, i - prefMap[prefSum - k]);
        if (!prefMap.count(prefSum))   // first occurrence only, never overwrite
            prefMap[prefSum] = i;
    }
    return maxLen;
}
```

---

**Sliding window template (non-negative arrays only):**

```cpp
int longestSubarrayNonNeg(vector<int>& arr, int k) {
    int l = 0; long long sum = 0; int maxLen = 0;
    for (int r = 0; r < (int)arr.size(); r++) {
        sum += arr[r];
        while (sum > k && l <= r) sum -= arr[l++];
        if (sum == k) maxLen = max(maxLen, r - l + 1);
    }
    return maxLen;
}
```

---

**Three rules that must not be violated:**

1. **Pre-insert `{0: -1}`** — or add explicit check `if (prefSum == k) maxLen = max(maxLen, i+1)`. Without it, the algorithm misses subarrays that start at index 0.
2. **Never overwrite map entries** — first occurrence gives maximum length. Later occurrence gives shorter subarray for the same right endpoint.
3. **Length = `i - prefMap[rem]`**, not `i - prefMap[rem] + 1` — the map stores the index before the subarray starts.

---

**Why sliding window fails with negatives:** The invariant "shrinking the window decreases the sum" requires `arr[l] > 0`. When `arr[l] <= 0`, removing it maintains or increases the sum, and the algorithm overshoots valid windows it can never recover.

---

**Variant cheat sheet:**

| Variant | Map value | Update rule | Pre-insert |
|---|---|---|---|
| Longest subarray sum = k | First index | Insert if absent | `{0: -1}` |
| Count subarrays sum = k | Frequency | Always increment | `{0: 1}` |
| Longest equal 0s and 1s | First index | Insert if absent | `{0: -1}` after 0→-1 transform |
| Longest sum divisible by k | First index | Insert if absent | `{0: -1}` with `((sum%k)+k)%k` |

---

**Overflow note:** Use `long long` for prefix sums when `arr[i]` and `n` are both large. For GFG default constraints (arr[i] ≤ 10⁴, n ≤ 10⁵), max prefix sum = 10⁹ fits `int`, but `long long` is safer.
