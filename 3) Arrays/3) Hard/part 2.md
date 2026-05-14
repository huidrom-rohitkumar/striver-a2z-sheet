> Video References: [Merge Intervals](https://youtu.be/IexN60k62jo) | [Merge Sorted Array](https://youtu.be/n7uwj04E0I4) | [Missing & Repeated](https://youtu.be/2D0D8HE6uak) | [Count Inversions](https://youtu.be/AseUmwVNaoY) | [Reverse Pairs](https://youtu.be/0e4bZaP3MDI) | [Max Product Subarray](https://youtu.be/hnswaLJvr6g)
> 
---

## Table of Contents

1. [The Unifying Mental Models](#1-the-unifying-mental-models)
2. [Merge Intervals (LC 56)](#2-merge-intervals-lc-56)
3. [Merge Sorted Array (LC 88)](#3-merge-sorted-array-lc-88)
4. [Set Mismatch and Find Missing & Repeated](#4-set-mismatch-and-find-missing--repeated)
5. [Count Inversions — Modified Merge Sort](#5-count-inversions--modified-merge-sort)
6. [Reverse Pairs (LC 493) — Modified Merge Sort](#6-reverse-pairs-lc-493--modified-merge-sort)
7. [Maximum Product Subarray (LC 152)](#7-maximum-product-subarray-lc-152)
8. [The Modified Merge Sort Framework](#8-the-modified-merge-sort-framework)
9. [Complexity Cheat Sheet](#9-complexity-cheat-sheet)
10. [Common Mistakes and Edge Cases](#10-common-mistakes-and-edge-cases)
11. [Interview Patterns and Extensions](#11-interview-patterns-and-extensions)
12. [Quick Revision Cheat Sheet](#12-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Models

Six problems. Four core ideas.

**Sort then sweep:** Merge Intervals. Sorting by start time creates a crucial invariant — if a new interval does not overlap the last merged one, it cannot overlap any previously merged interval. A two-dimensional problem (compare all pairs) collapses to a linear sweep.

**Reverse two-pointer to avoid clobbering:** Merge Sorted Array. Merging from the front overwrites elements before they are read. The tail of `nums1` is guaranteed empty; merging largest-first into those positions writes into space that has already been vacated.

**Two-equation system from expected vs actual aggregates:** Set Mismatch / Missing & Repeated. Two unknowns (duplicate `d`, missing `m`) require two independent equations. Sum gives `d - m`; sum-of-squares gives `d^2 - m^2 = (d-m)(d+m)`. From `d-m` and `d+m`, solve for both individually.

**Count during merge (the modified merge sort framework):** Inversions and Reverse Pairs. At the merge step, both halves are already sorted. A condition of the form `left[i] > f(right[j])` that is monotone in both halves can be counted in O(n) per level using a pointer that only advances. This turns O(n^2) brute force into O(n log n).

**Track both extremes for sign-flip dynamics:** Maximum Product Subarray. For sum, only the running maximum matters. For product, a negative running minimum becomes the new maximum when the next element is also negative. Both `curMax` and `curMin` must be tracked simultaneously at every step.

The meta-skill trained by this entire set: whenever brute force compares every pair, ask *"can sorting or merge sort reduce these comparisons?"*

---

## 2. Merge Intervals (LC 56)

**Problem:** Given an array of intervals `[start_i, end_i]`, merge all overlapping intervals and return the non-overlapping result.

```
Input:  [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]

Input:  [[1,4],[4,5]]
Output: [[1,5]]   (touching endpoints count as overlap)
```

**Constraints:** `1 <= intervals.length <= 10^4`, `0 <= start_i <= end_i <= 10^4`

### Why Sort is the Key Insight

Without sorting, detecting whether interval A overlaps interval B requires comparing against all others — O(n^2). After sorting by start time, a critical invariant holds: **if the next interval does not overlap the last merged interval, it cannot overlap any previously merged interval.** This is because all merged intervals end before or at the last merged interval's end, and all future intervals start at or after the next interval's start. The overlap check becomes a single comparison against `merged.back()`.

**Overlap condition:** Two intervals `[a, b]` and `[c, d]` overlap if and only if `c <= b`. When `c == b` (touching endpoints), LC 56 considers them overlapping.

**Why take `max` for the merged end:** A fully contained interval like `[2,3]` inside `[1,10]` — the merged end must be `max(10, 3) = 10`. Taking only the current interval's end would incorrectly shrink an already larger merged interval.

### Approach 1 — Brute Force

For each interval, repeatedly scan all others and merge overlaps until stable. O(n^2).

### Approach 2 — Optimal: Sort + Linear Sweep

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());  // sort by start time
    vector<vector<int>> merged;

    for (auto& interval : intervals) {
        if (merged.empty() || interval[0] > merged.back()[1]) {
            merged.push_back(interval);  // no overlap: start new merged block
        } else {
            merged.back()[1] = max(merged.back()[1], interval[1]);  // extend end
        }
    }
    return merged;
}
```

**Time:** O(n log n) for sort + O(n) for sweep = O(n log n).
**Space:** O(n) for output; O(log n) for sort stack.

### Dry Run

Input: `[[1,3],[2,6],[8,10],[15,18]]` (sorted)

```
merged=[]
[1,3]:  empty → push.          merged=[[1,3]]
[2,6]:  2<=3 (overlap) → end=max(3,6)=6.  merged=[[1,6]]
[8,10]: 8>6 (no overlap) → push.  merged=[[1,6],[8,10]]
[15,18]:15>10 → push.          merged=[[1,6],[8,10],[15,18]]
```

**Contained interval case:** `[[1,10],[2,5],[3,7]]`

```
sorted: [[1,10],[2,5],[3,7]]
[1,10]: push.
[2,5]:  2<=10 → end=max(10,5)=10. (no change)
[3,7]:  3<=10 → end=max(10,7)=10. (no change)
Result: [[1,10]]
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[[1,4]]` | `[[1,4]]` | Single interval |
| `[[1,4],[5,6]]` | `[[1,4],[5,6]]` | 5 > 4: no overlap |
| `[[1,4],[4,5]]` | `[[1,5]]` | 4 <= 4: touching endpoints merge |
| `[[1,10],[2,3]]` | `[[1,10]]` | Contained; max(10,3)=10 |
| `[[1,4],[2,3],[3,5]]` | `[[1,5]]` | Chain of overlaps |

---

## 3. Merge Sorted Array (LC 88)

**Problem:** `nums1` has `m` valid elements followed by `n` zeros. `nums2` has `n` elements. Both are sorted. Merge `nums2` into `nums1` in-place.

```
Input:  nums1=[1,2,3,0,0,0], m=3, nums2=[2,5,6], n=3
Output: nums1=[1,2,2,3,5,6]

Input:  nums1=[1], m=1, nums2=[], n=0
Output: [1]
```

### Why Merging from the Front Fails

If `nums2[0] < nums1[0]`, placing `nums2[0]` at `nums1[0]` overwrites `nums1[0]` before it has been written to the result. Even with shifting, this costs O(m*n) total.

### Why Merging from the Back Works

The tail of `nums1` (`positions m..m+n-1`) contains guaranteed placeholders. The largest element across both arrays belongs at `nums1[m+n-1]`, which is empty. Writing largest-first into the tail never overwrites an unread position.

**Formal proof of no clobbering:** At any step, the write pointer is at position `k = m+n-1-t` (where `t` = steps taken) and the read pointer for `nums1` is at `i <= m-1-t_1` where `t_1 <= t`. So `k >= i + n >= i`. The write pointer is always at or ahead of the read pointer.

### Approach 1 — Brute Force: Copy Then Sort

```cpp
void mergeBrute(vector<int>& nums1, int m, vector<int>& nums2, int n) {
    for (int i = 0; i < n; i++) nums1[m + i] = nums2[i];
    sort(nums1.begin(), nums1.end());
}
```

**Time:** O((m+n) log(m+n)). Discards sortedness of both arrays.

### Approach 2 — Optimal: Reverse Two-Pointer

```cpp
void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
    int i = m - 1;       // last valid element of nums1
    int j = n - 1;       // last element of nums2
    int k = m + n - 1;   // write position

    while (i >= 0 && j >= 0) {
        if (nums1[i] > nums2[j]) nums1[k--] = nums1[i--];
        else                      nums1[k--] = nums2[j--];
    }

    // Only nums2 needs draining — remaining nums1 elements are already in place
    while (j >= 0) nums1[k--] = nums2[j--];
}
```

**Why no drain for remaining `nums1`:** If `j < 0` but `i >= 0`, those `nums1` elements are at positions `0..i`, which is exactly where they belong in the final merged array. No action needed.

**Time:** O(m+n) — every element is written exactly once. **Space:** O(1).

### Dry Run

`nums1=[1,2,3,0,0,0]`, m=3, `nums2=[2,5,6]`, n=3

```
i=2(3), j=2(6), k=5: 3<6 → nums1[5]=6, j=1, k=4.  [1,2,3,0,0,6]
i=2(3), j=1(5), k=4: 3<5 → nums1[4]=5, j=0, k=3.  [1,2,3,0,5,6]
i=2(3), j=0(2), k=3: 3>2 → nums1[3]=3, i=1, k=2.  [1,2,3,3,5,6]
i=1(2), j=0(2), k=2: 2<=2 → nums1[2]=2, j=-1, k=1. [1,2,2,3,5,6]
j<0: exit. i=1 still in place at nums1[0..1]. Done.

Final: [1,2,2,3,5,6]
```

---

## 4. Set Mismatch and Find Missing & Repeated

**LC 645 — Set Mismatch:** Array originally `[1..n]`; one number duplicated, one missing. Return `[duplicate, missing]`.

```
Input:  [1,2,2,4]   Output: [2,3]
Input:  [1,1]       Output: [1,2]
```

**LC 2965 — Find Missing and Repeated Values:** n×n grid should contain `[1..n^2]`; one repeated, one missing. Return `[repeated, missing]`. (Flatten the grid and apply the same approaches.)

### The Two-Unknown System

Let `d` = duplicate, `m` = missing.

```
Expected sum:    n*(n+1)/2
Expected sq-sum: n*(n+1)*(2n+1)/6

From actual vs expected:
  d - m  = actualSum   - expectedSum    ... (I)
  d² - m² = actualSqSum - expectedSqSum

  (d-m)(d+m) = actualSqSum - expectedSqSum
  d + m = (actualSqSum - expectedSqSum) / (d - m)  ... (II)

Solving (I) and (II):
  d = ((d-m) + (d+m)) / 2
  m = ((d+m) - (d-m)) / 2
```

### Approach 1 — Frequency Count

```cpp
vector<int> findErrorNumsFreq(vector<int>& nums) {
    int n = nums.size();
    vector<int> freq(n + 1, 0);
    for (int x : nums) freq[x]++;
    int dup = -1, miss = -1;
    for (int i = 1; i <= n; i++) {
        if (freq[i] == 2) dup  = i;
        if (freq[i] == 0) miss = i;
    }
    return {dup, miss};
}
```

**Time:** O(n). **Space:** O(n).

### Approach 2 — Math: O(n) Time, O(1) Space

```cpp
vector<int> findErrorNums(vector<int>& nums) {
    long long n = nums.size();
    long long actualSum = 0, actualSqSum = 0;
    for (int x : nums) {
        actualSum   += x;
        actualSqSum += (long long)x * x;
    }
    long long expectedSum   = n * (n + 1) / 2;
    long long expectedSqSum = n * (n + 1) * (2 * n + 1) / 6;

    long long diffSum = actualSum - expectedSum;         // d - m
    long long diffSq  = actualSqSum - expectedSqSum;    // (d-m)(d+m)
    long long sumDM   = diffSq / diffSum;               // d + m

    int d = (int)((diffSum + sumDM) / 2);
    int m = (int)((sumDM - diffSum) / 2);
    return {d, m};
}
```

**Why `long long` throughout:** For n=10^4, `n^3 ~ 10^12` appears in the sum-of-squares formula, far exceeding `INT_MAX`.

**Time:** O(n). **Space:** O(1).

### Approach 3 — XOR: O(n) Time, O(1) Space, No Overflow Risk

XOR all array elements with all values `1..n`. Each non-duplicated element cancels, leaving `d ^ m`. Find the lowest set bit of `d ^ m` (the bit where d and m differ). Use it to partition all numbers into two groups; XOR within each group isolates d or m. A final scan distinguishes which is the duplicate.

```cpp
vector<int> findErrorNumsXOR(vector<int>& nums) {
    int n = nums.size(), xorr = 0;
    for (int i = 1; i <= n; i++) xorr ^= i;
    for (int x : nums) xorr ^= x;
    // xorr = d ^ m

    int bit = xorr & (-xorr);   // isolate lowest set bit
    int x = 0, y = 0;
    for (int i = 1; i <= n; i++) (i & bit) ? x ^= i : y ^= i;
    for (int v : nums)           (v & bit) ? x ^= v : y ^= v;

    // x and y are {d, m} in some order; verify which is the duplicate
    for (int v : nums) if (v == x) return {x, y};
    return {y, x};
}
```

**Time:** O(n). **Space:** O(1).

### Dry Run (Math Approach)

Input: `[1,2,2,4]`, n=4

```
actualSum = 9,  actualSqSum = 25
expectedSum = 10, expectedSqSum = 30

diffSum = 9-10 = -1     (d-m = -1)
diffSq  = 25-30 = -5    (d²-m² = -5)
sumDM   = -5/-1 = 5     (d+m = 5)

d = (-1+5)/2 = 2   (duplicate)
m = (5-(-1))/2 = 3 (missing)
Return: [2, 3]  ✓
```

---

## 5. Count Inversions — Modified Merge Sort

**Problem:** Count pairs (i, j) such that `i < j` and `arr[i] > arr[j]`.

```
Input:  [2, 4, 1, 3, 5]   Output: 3   (pairs: (2,1),(4,1),(4,3))
Input:  [5, 4, 3, 2, 1]   Output: 10  (every pair; n*(n-1)/2)
```

**Constraints:** `1 <= n <= 10^5`, elements can be any integer.

### Approach 1 — Brute Force

Nested loops check every pair. **Time:** O(n^2). TLE for n = 10^5.

### Approach 2 — Optimal: Modified Merge Sort

**Why merge sort works:** Every inversion is one of three types:
1. Both indices in the left half — counted recursively.
2. Both indices in the right half — counted recursively.
3. Left index in the left half, right index in the right half (cross-inversions) — counted during merge.

**Counting cross-inversions during merge:** At the merge step, the left subarray `L[lo..mid]` and right subarray `R[mid+1..hi]` are both sorted. When `L[i] > R[j]`, since L is sorted, every remaining left element `L[i], L[i+1], ..., L[mid]` is also greater than `R[j]`. All `(mid - i + 1)` of them form inversions with `R[j]`. Add this batch count in O(1).

**Why relative order within each half doesn't affect cross-inversion count:** Sorting within the left half only reorders pairs (i1, i2) where both i1, i2 are in the left half — these are already counted recursively. The cross-inversions between any left element and any right element depend only on their values, not their positions within each half.

```cpp
#include <bits/stdc++.h>
using namespace std;

long long mergeAndCount(vector<int>& arr, int lo, int mid, int hi) {
    vector<int> temp;
    int i = lo, j = mid + 1;
    long long count = 0;

    while (i <= mid && j <= hi) {
        if (arr[i] <= arr[j]) {
            temp.push_back(arr[i++]);
        } else {
            // arr[i] > arr[j]: ALL remaining left elements invert with arr[j]
            count += (long long)(mid - i + 1);
            temp.push_back(arr[j++]);
        }
    }
    while (i <= mid) temp.push_back(arr[i++]);
    while (j <= hi)  temp.push_back(arr[j++]);

    for (int k = lo; k <= hi; k++) arr[k] = temp[k - lo];
    return count;
}

long long mergeSort(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return 0;
    int mid = lo + (hi - lo) / 2;
    long long count = 0;
    count += mergeSort(arr, lo, mid);
    count += mergeSort(arr, mid + 1, hi);
    count += mergeAndCount(arr, lo, mid, hi);
    return count;
}

long long countInversions(vector<int>& arr) {
    return mergeSort(arr, 0, arr.size() - 1);
}
```

**Why `long long` for count:** Maximum inversions = n*(n-1)/2. For n=10^5, this is ~5*10^9, exceeding `INT_MAX`.

**Time:** O(n log n) — same recurrence as merge sort. **Space:** O(n) auxiliary array + O(log n) stack.

### Dry Run

Input: `[2, 4, 1, 3, 5]`

```
mergeSort(0,4), mid=2
  mergeSort(0,2), mid=1
    mergeSort(0,1), mid=0
      mergeSort(0,0): 0
      mergeSort(1,1): 0
      merge(0,0,1): L=[2],R=[4]. 2<=4 → no count. arr=[2,4,...]. return 0
    mergeSort(2,2): 0
    merge(0,1,2): L=[2,4],R=[1].
      2>1 → count+=2-0+1=2, take 1.
      drain L. arr=[1,2,4,3,5]. return 2

  mergeSort(3,4), mid=3
    mergeSort(3,3): 0
    mergeSort(4,4): 0
    merge(3,3,4): L=[3],R=[5]. 3<=5 → no count. return 0

  merge(0,2,4): L=[1,2,4],R=[3,5].
    1<=3 → take 1.
    2<=3 → take 2.
    4>3  → count+=2-2+1=1, take 3.
    4<=5 → take 4.
    drain R.
    arr=[1,2,3,4,5]. return 1

Total: 0+2+0+1=3  ✓
```

---

## 6. Reverse Pairs (LC 493) — Modified Merge Sort

**Problem:** Count pairs (i, j) where `i < j` and `nums[i] > 2 * nums[j]`.

```
Input:  [1,3,2,3,1]   Output: 2
Input:  [2,4,3,5,1]   Output: 3
```

**Constraints:** `1 <= nums.length <= 5*10^4`, `-2^31 <= nums[i] <= 2^31-1`

### The Critical Structural Difference from Inversions

For inversions, the counting condition `arr[i] > arr[j]` is identical to the merge order condition (take from right when left is greater). So counting and merging can be interleaved in the same loop.

For reverse pairs, the counting condition is `arr[i] > 2*arr[j]`, which is **not** the same as the merge condition `arr[i] > arr[j]`. If you count and merge in the same pass, the counting pointer and merging pointer go out of sync. **The fix: two separate passes — count first, then merge.**

**Why counting with a non-resetting pointer is O(n) per level:** After both halves are sorted, advance a right-half pointer `j` for each left-half element `i`. Since left is sorted (non-decreasing), whenever `nums[i] > 2*nums[j]`, `nums[i+1] >= nums[i] > 2*nums[j]` as well. So `j` for `i+1` starts where `j` for `i` stopped — it never goes backward. The inner while loop advances `j` at most n times across all values of `i`, giving O(n) total for the counting phase per merge level.

```cpp
#include <bits/stdc++.h>
using namespace std;

int countAndMerge(vector<int>& nums, int lo, int mid, int hi) {
    int count = 0;

    // Phase 1: COUNT — separate pass, j only advances
    int j = mid + 1;
    for (int i = lo; i <= mid; i++) {
        while (j <= hi && (long long)nums[i] > 2LL * nums[j]) j++;
        count += j - (mid + 1);
    }

    // Phase 2: MERGE — standard merge sort merge
    vector<int> temp;
    int p1 = lo, p2 = mid + 1;
    while (p1 <= mid && p2 <= hi) {
        if (nums[p1] <= nums[p2]) temp.push_back(nums[p1++]);
        else                       temp.push_back(nums[p2++]);
    }
    while (p1 <= mid) temp.push_back(nums[p1++]);
    while (p2 <= hi)  temp.push_back(nums[p2++]);
    for (int k = lo; k <= hi; k++) nums[k] = temp[k - lo];

    return count;
}

int mergeSort(vector<int>& nums, int lo, int hi) {
    if (lo >= hi) return 0;
    int mid = lo + (hi - lo) / 2;
    int count = 0;
    count += mergeSort(nums, lo, mid);
    count += mergeSort(nums, mid + 1, hi);
    count += countAndMerge(nums, lo, mid, hi);
    return count;
}

int reversePairs(vector<int>& nums) {
    return mergeSort(nums, 0, nums.size() - 1);
}
```

**Why `2LL * nums[j]` is mandatory:** `nums[j]` can be up to `2^31-1`. Multiplying by 2 gives `2^32-2`, which overflows a 32-bit int (max ~2.1*10^9). The `2LL` literal promotes the multiplication to `long long`.

**Time:** O(n log n). **Space:** O(n).

### Dry Run

Input: `[1,3,2,3,1]`

```
mergeSort(0,4), mid=2
  mergeSort(0,2): eventually produces sorted [1,2,3], count=0
  mergeSort(3,4): L=[3],R=[1]
    count phase: j=4. i=3: 3>2*1=2? Yes. j=5. count+=5-4=1.
    merge: [1,3]. nums=[1,2,3,1,3]. return 1

  countAndMerge(0,2,4): L=[1,2,3],R=[1,3]
    count: j=3.
      i=0: 1>2*1=2? No. count+=3-3=0.
      i=1: 2>2*1=2? No (2 is not strictly > 2). count+=3-3=0.
      i=2: 3>2*1=2? Yes. j=4. 3>2*3=6? No. count+=4-3=1.
    merge: [1,1,2,3,3]. count=1.

Total: 0+1+1=2  ✓
```

---

## 7. Maximum Product Subarray (LC 152)

**Problem:** Find the contiguous subarray with the largest product and return it.

```
Input:  [2,3,-2,4]    Output: 6     ([2,3])
Input:  [-2,0,-1]     Output: 0     ([0])
Input:  [-2,3,-4]     Output: 24    ([-2,3,-4])
```

**Constraints:** `1 <= n <= 2*10^4`, `-10 <= nums[i] <= 10`

### Why Standard Kadane Fails

Kadane's for sum: `curMax = max(nums[i], curMax + nums[i])`. This works because addition is monotone — a positive addend always helps; a negative always hurts.

For product: a large negative running minimum becomes the maximum when multiplied by the next negative. Tracking only `curMax` discards the information needed to make this flip.

**Example:** `[-3, -2, -4]`. After `-3`: curMax=-3, curMin=-3. After `-2`: curMax=max(-2, -3*-2=6)=6, curMin=min(-2, 6)=-2. After `-4`: curMax=max(-4, 6*-4=-24, -2*-4=8)=8 — the curMin (-2) yielded the winner. A single-max approach would give -4 here.

### Approach 1 — Brute Force

Two nested loops compute all subarray products. **Time:** O(n^2).

### Approach 2 — Optimal: Track curMax and curMin

**Recurrence:** At each position `i`, the maximum product ending at `i` is one of:
- `nums[i]` alone (start fresh),
- `prevMax * nums[i]` (extend the max-product subarray),
- `prevMin * nums[i]` (prevMin flips to max when nums[i] < 0).

Same three choices apply for the minimum. Save `prevMax` before updating so curMin can use the original value.

```cpp
int maxProduct(vector<int>& nums) {
    int n = nums.size();
    int curMax = nums[0], curMin = nums[0], result = nums[0];

    for (int i = 1; i < n; i++) {
        int tempMax = curMax;  // save before overwriting
        curMax = max({nums[i], curMax * nums[i], curMin * nums[i]});
        curMin = min({nums[i], tempMax * nums[i], curMin * nums[i]});
        result = max(result, curMax);
    }
    return result;
}
```

**Why `tempMax` is required:** When computing `curMin`, we need the OLD `curMax` (before it was updated for this iteration). Using the new `curMax` would introduce a forward dependency and produce wrong answers.

**Why initialize `result = nums[0]` and not 0 or `INT_MIN`:** A single-element array with a negative number must return that negative number. `INT_MIN` would work too, but `nums[0]` is cleaner.

**Time:** O(n). **Space:** O(1).

### Approach 3 — Prefix and Suffix Products

**Insight:** Any zero-free subarray achieves its maximum product by taking either the full subarray (even number of negatives) or the subarray minus a prefix up to the first negative, or minus a suffix from the last negative. A left-to-right scan (prefix) and right-to-left scan (suffix) together cover all cases. Zeros reset the running product.

```cpp
int maxProductPrefixSuffix(vector<int>& nums) {
    int n = nums.size();
    int result = *max_element(nums.begin(), nums.end());
    int prefix = 1, suffix = 1;

    for (int i = 0; i < n; i++) {
        if (prefix == 0) prefix = 1;
        if (suffix == 0) suffix = 1;
        prefix *= nums[i];
        suffix *= nums[n - 1 - i];
        result = max({result, prefix, suffix});
    }
    return result;
}
```

**Why reset to 1 after zero:** A zero product means the subarray so far contributes nothing. Fresh subarray starts at 1 (the identity for multiplication). If we didn't reset, all subsequent products would be zero regardless of what comes next.

**Time:** O(n). **Space:** O(1).

### Dry Run (curMax/curMin)

Input: `[2,3,-2,4]`

```
Initial: curMax=2, curMin=2, result=2

i=1, nums[1]=3:
  tempMax=2
  curMax=max(3, 2*3=6, 2*3=6)=6
  curMin=min(3, 2*3=6, 2*3=6)=3
  result=max(2,6)=6

i=2, nums[2]=-2:
  tempMax=6
  curMax=max(-2, 6*-2=-12, 3*-2=-6)=-2
  curMin=min(-2, 6*-2=-12, 3*-2=-6)=-12
  result=max(6,-2)=6

i=3, nums[3]=4:
  tempMax=-2
  curMax=max(4, -2*4=-8, -12*4=-48)=4
  curMin=min(4, -2*4=-8, -12*4=-48)=-48
  result=max(6,4)=6

Return: 6  ([2,3])
```

Input: `[-2,3,-4]`

```
Initial: curMax=-2, curMin=-2, result=-2

i=1, nums[1]=3:
  tempMax=-2
  curMax=max(3, -2*3=-6, -2*3=-6)=3
  curMin=min(3, -2*3=-6, -2*3=-6)=-6
  result=3

i=2, nums[2]=-4:
  tempMax=3
  curMax=max(-4, 3*-4=-12, -6*-4=24)=24
  curMin=min(-4, 3*-4=-12, -6*-4=24)=-12
  result=24

Return: 24  ([-2,3,-4])
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[-2]` | `-2` | Single negative; must return it |
| `[0]` | `0` | No non-zero subarray |
| `[0,2]` | `2` | Zero resets; `[2]` is the answer |
| `[-1,-2,-9,-6]` | `108` | `[-2,-9,-6]`; even negative count |
| `[-1,0,-2]` | `0` | Zero breaks; best single subarray is 0 |
| `[1,0,1]` | `1` | Zero between two 1s |

---

## 8. The Modified Merge Sort Framework

Count Inversions and Reverse Pairs are instances of a general pattern: **compute a function over cross-pairs (i in left half, j in right half) during merge sort, exploiting that both halves are sorted.**

```cpp
// Template
long long mergeSort(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return 0;
    int mid = lo + (hi - lo) / 2;
    long long count = 0;
    count += mergeSort(arr, lo, mid);
    count += mergeSort(arr, mid + 1, hi);
    count += countCrossAndMerge(arr, lo, mid, hi);
    return count;
}
```

**Inversions:** In `countCrossAndMerge`, the counting condition (`arr[i] > arr[j]`) is the same as the merge order condition. Count and merge in the same loop. When taking from the right: `count += (mid - i + 1)`.

**Reverse Pairs:** The counting condition (`arr[i] > 2*arr[j]`) differs from merge order. **Two separate passes:** first count with a dedicated non-resetting `j` pointer, then merge normally.

**When this framework applies:** Any condition `f(left[i], right[j])` that is monotone — if true for `(i, j)`, true for all `(i', j)` with `i' >= i` (left is sorted non-decreasing) — can be counted in O(n) per level. The inner pointer only advances, giving amortized O(n) across all outer iterations per merge.

**Problems solvable with this framework:**

| Problem | Condition | Counting method |
|---|---|---|
| Inversions | `left[i] > right[j]` | Combine with merge |
| Reverse Pairs | `left[i] > 2*right[j]` | Separate pass then merge |
| Count Smaller After Self (LC 315) | `left[i] > right[j]` | Track position shifts |
| Count of Range Sum (LC 327) | prefix sum in `[lo, hi]` | Modified merge on prefix sums |

---

## 9. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Merge Intervals | Brute force | O(n^2) | O(n) |
| Merge Intervals | Sort + linear sweep | O(n log n) | O(n) output |
| Merge Sorted Array | Copy then sort | O((m+n) log(m+n)) | O(1) |
| Merge Sorted Array | Reverse two-pointer | O(m+n) | O(1) |
| Set Mismatch | Frequency count | O(n) | O(n) |
| Set Mismatch | Sum + sum-of-squares | O(n) | O(1) |
| Set Mismatch | XOR | O(n) | O(1) |
| Count Inversions | Brute force | O(n^2) | O(1) |
| Count Inversions | Modified merge sort | O(n log n) | O(n) |
| Reverse Pairs | Brute force | O(n^2) | O(1) |
| Reverse Pairs | Modified merge sort | O(n log n) | O(n) |
| Max Product Subarray | Brute force | O(n^2) | O(1) |
| Max Product Subarray | curMax/curMin DP | O(n) | O(1) |
| Max Product Subarray | Prefix-suffix | O(n) | O(1) |

---

## 10. Common Mistakes and Edge Cases

### Mistake 1 — Not Taking `max` for Merged Interval End

```cpp
// WRONG: shrinks [1,10] when processing contained interval [2,3]
merged.back()[1] = interval[1];

// CORRECT:
merged.back()[1] = max(merged.back()[1], interval[1]);
```

### Mistake 2 — Forgetting to Sort Before Merging Intervals

```cpp
// WRONG: without sort, overlapping intervals may not be adjacent
for (auto& interval : intervals) { ... }

// CORRECT:
sort(intervals.begin(), intervals.end());
for (auto& interval : intervals) { ... }
```

### Mistake 3 — Merging Sorted Array from the Front

Merging left-to-right overwrites `nums1` values before they are placed in the result. Always merge from the back. If i >= 0 but j < 0 after the main loop, `nums1[0..i]` is already correct — no drain needed.

### Mistake 4 — Not Using `long long` in Set Mismatch Math

```cpp
// WRONG: n^3 term in expected sq-sum overflows int for n=10^4
int expectedSqSum = n*(n+1)*(2*n+1)/6;

// CORRECT:
long long n = nums.size();
long long expectedSqSum = n*(n+1)*(2*n+1)/6;
```

Also: `(long long)x * x` for individual elements, since squaring values near `n = 10^4` is fine but for larger n becomes necessary.

### Mistake 5 — Counting Inversions from the Wrong Side of the Merge

```cpp
// WRONG: taking from left does NOT create inversions
if (arr[i] <= arr[j]) {
    count += (hi - j + 1);   // WRONG: these are not inversions
    temp.push_back(arr[i++]);
}

// CORRECT: count only when taking from the RIGHT (right element is smaller)
if (arr[i] <= arr[j]) {
    temp.push_back(arr[i++]);
} else {
    count += (long long)(mid - i + 1);  // all remaining left elements > arr[j]
    temp.push_back(arr[j++]);
}
```

### Mistake 6 — Combining Count and Merge in Reverse Pairs

```cpp
// WRONG: counting condition (>2*arr[j]) differs from merge condition (>arr[j])
// Merging while counting causes the j pointer to advance for the wrong reason
while (i <= mid && j <= hi) {
    if ((long long)arr[i] > 2LL * arr[j]) {
        count += (mid - i + 1);
        temp.push_back(arr[j++]);  // WRONG: j advances on the counting condition, not merge
    }
    ...
}

// CORRECT: two completely separate loops — count first, then merge
```

### Mistake 7 — Integer Overflow in Reverse Pairs

```cpp
// WRONG: 2 * nums[j] overflows int when nums[j] near 2^31-1
if (nums[i] > 2 * nums[j])

// CORRECT:
if ((long long)nums[i] > 2LL * nums[j])
```

### Mistake 8 — Not Saving `tempMax` in Max Product DP

```cpp
// WRONG: curMin uses the newly computed curMax, not the old one
curMax = max({nums[i], curMax*nums[i], curMin*nums[i]});
curMin = min({nums[i], curMax*nums[i], curMin*nums[i]});  // uses NEW curMax!

// CORRECT:
int tempMax = curMax;
curMax = max({nums[i], curMax*nums[i], curMin*nums[i]});
curMin = min({nums[i], tempMax*nums[i], curMin*nums[i]});
```

### Mistake 9 — Initializing Max Product Result to 0

```cpp
// WRONG: array [-3,-2,-1] has answer -1, but 0 > -1
int result = 0;

// CORRECT: initialize to nums[0] (handles single negative element)
int result = nums[0];
```

### Edge Cases Summary

| Problem | Critical Edge Cases |
|---|---|
| Merge Intervals | Touching endpoints `[1,4],[4,5]` → merge; contained `[1,10],[2,3]` → max end; single interval |
| Merge Sorted Array | m=0 (drain all of nums2), n=0 (no drain needed), equal elements (use `<=` to maintain stability) |
| Set Mismatch | `[2,2]` → d=2, m=1; `[1,1]` → d=1, m=2; validate with `long long` |
| Count Inversions | Sorted input → 0; reverse sorted → n*(n-1)/2 (use `long long`) |
| Reverse Pairs | All zeros → 0 (0 > 2*0 is false); negative elements require `2LL` cast |
| Max Product | `[-2]` → -2; `[0,-3,1]` → 1; `[-1,-2,-9,-6]` → 108 (even negatives) |

---

## 11. Interview Patterns and Extensions

### Pattern Recognition Table

| Signal in problem | Algorithm |
|---|---|
| Merge / overlap intervals | Sort by start + linear sweep |
| In-place merge of two sorted arrays | Reverse two-pointer from end |
| One duplicate, one missing in [1..n] | Sum + sum-of-squares, or XOR |
| Count pairs (i<j) with `arr[i] > f(arr[j])` | Modified merge sort |
| Maximum product with potential negatives and zeros | curMax + curMin DP or prefix-suffix |

### Extensions

**Merge Intervals family:**
- LC 57 (Insert Interval): insert one interval into a sorted non-overlapping list. Three phases: add prefix (no overlap), merge overlap region, add suffix.
- LC 435 (Non-overlapping Intervals): minimum removals to eliminate all overlaps. Greedy: sort by end; greedily keep earliest-ending intervals.
- LC 1288 (Remove Covered Intervals): count intervals not covered by any other; sort by start descending, then by end descending.

**Merge Sorted Array:**
- Merge K sorted arrays: min-heap of size K, each entry storing (value, arrayIndex, elementIndex). O(n log k) total.
- LC 23 (Merge K Sorted Lists): same min-heap approach on linked list nodes.

**Set Mismatch / Missing & Repeated:**
- LC 287 (Find Duplicate, no modification): Floyd's cycle detection treating array as a linked list.
- LC 448 (Find All Missing Numbers): negate `nums[abs(nums[i])-1]` for each element; indices still positive are missing.
- LC 41 (First Missing Positive): cycle-sort elements to their correct positions; first mismatch is the answer.

**Modified Merge Sort:**
- LC 315 (Count Smaller After Self): for each element, count how many elements to its right are smaller. Track positions during merge.
- LC 327 (Count of Range Sum): count subarrays with sum in `[lower, upper]`. Build prefix sum array; count pairs `(i,j)` where `lower <= P[j]-P[i] <= upper` using modified merge sort on prefix sums.

**Maximum Product Subarray:**
- LC 53 (Maximum Subarray Sum): standard Kadane with single running max.
- LC 238 (Product of Array Except Self): prefix/suffix products without division.
- Circular maximum product: consider prefix products wrapping around.

---

## 12. Quick Revision Cheat Sheet

**Merge Intervals**

```
Sort by start. For each interval:
  if no overlap (interval[0] > back[1]): push new interval
  else: back[1] = max(back[1], interval[1])
O(n log n) time, O(n) space.
```

**Merge Sorted Array**

```
i=m-1, j=n-1, k=m+n-1
while(i>=0 && j>=0): place larger at nums1[k--]
while(j>=0): nums1[k--]=nums2[j--]
No drain for remaining nums1 — already in place.
O(m+n) time, O(1) space.
```

**Set Mismatch / Missing & Repeated**

```
diffSum = actualSum - n*(n+1)/2           // d-m
diffSq  = actualSqSum - n*(n+1)*(2n+1)/6  // (d-m)(d+m)
d+m = diffSq / diffSum
d = ((d-m)+(d+m))/2,  m = ((d+m)-(d-m))/2
Use long long throughout.
```

**Count Inversions**

```cpp
// In merge: when taking from right (arr[i] > arr[j])
count += (long long)(mid - i + 1);
// Return type: long long (max ~5*10^9 for n=10^5)
O(n log n) time, O(n) space.
```

**Reverse Pairs — Two Separate Passes**

```cpp
// Phase 1: count (j never resets)
int j = mid+1;
for (int i = lo; i <= mid; i++) {
    while (j <= hi && (long long)nums[i] > 2LL*nums[j]) j++;
    count += j-(mid+1);
}
// Phase 2: standard merge sort merge
// ALWAYS: 2LL* to avoid overflow
O(n log n) time, O(n) space.
```

**Maximum Product Subarray**

```cpp
int curMax=nums[0], curMin=nums[0], result=nums[0];
for (int i=1; i<n; i++) {
    int tmp = curMax;
    curMax = max({nums[i], curMax*nums[i], curMin*nums[i]});
    curMin = min({nums[i], tmp*nums[i], curMin*nums[i]});
    result = max(result, curMax);
}
// Save tempMax before updating. Init result to nums[0].
// Prefix-suffix alternative: reset to 1 after zeros, take max of all prefix/suffix values.
O(n) time, O(1) space.
```

**Modified Merge Sort Comparison**

| | Inversions | Reverse Pairs |
|---|---|---|
| Condition | `arr[i] > arr[j]` | `arr[i] > 2*arr[j]` |
| Count + merge | Same loop | Separate loops |
| Overflow | None | `2LL*arr[j]` |
| Count type | `long long` | `int` or `long long` |

**Complexity at a Glance**

| Problem | Optimal Time | Space |
|---|---|---|
| Merge Intervals | O(n log n) | O(n) |
| Merge Sorted Array | O(m+n) | O(1) |
| Set Mismatch (math) | O(n) | O(1) |
| Count Inversions | O(n log n) | O(n) |
| Reverse Pairs | O(n log n) | O(n) |
| Max Product Subarray | O(n) | O(1) |

---
