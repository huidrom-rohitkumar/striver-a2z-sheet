> Video References: [Pascal's Triangle](https://youtu.be/bR7mQgwQ_o8) | [Majority Element II](https://youtu.be/vwZj1K0e9U8) | [3Sum](https://youtu.be/DhFh8Kw7ymk) | [4Sum](https://youtu.be/eD95WRfh81c) | [Largest 0-Sum Subarray](https://youtu.be/xmguZ6GbatA) | [Subarray XOR](https://youtu.be/eZr-6p0B7ME)
>
> Problems Covered: Pascal's Triangle (LC 118) | Majority Element II (LC 229) | 3Sum (LC 15) | 4Sum (LC 18) | Largest Subarray with 0 Sum (GFG) | Count Subarrays with Given XOR (InterviewBit)

---

## Table of Contents

1. [The Unifying Mental Model](#1-the-unifying-mental-model)
2. [Pascal's Triangle](#2-pascals-triangle)
3. [Majority Element II — Boyer-Moore Extended](#3-majority-element-ii--boyer-moore-extended)
4. [3Sum](#4-3sum)
5. [4Sum](#5-4sum)
6. [Largest Subarray with 0 Sum](#6-largest-subarray-with-0-sum)
7. [Count Subarrays with Given XOR](#7-count-subarrays-with-given-xor)
8. [The Prefix Hash Unification: Sum and XOR](#8-the-prefix-hash-unification-sum-and-xor)
9. [Complexity Cheat Sheet](#9-complexity-cheat-sheet)
10. [Common Mistakes and Edge Cases](#10-common-mistakes-and-edge-cases)
11. [Interview Patterns and Extensions](#11-interview-patterns-and-extensions)
12. [Quick Revision Cheat Sheet](#12-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Model

These six problems look unrelated. They are all instances of one meta-skill: eliminating brute-force enumeration by exploiting hidden structure in the problem.

| Problem | Hidden Structure | How It Is Exploited |
|---|---|---|
| Pascal's Triangle | Combinatorial recurrence | `C(n,r) = C(n-1,r-1) + C(n-1,r)` replaces factorial computation |
| Majority Element II | Frequency dominance — at most k-1 elements exceed n/k | Vote cancellation survives true majority |
| 3Sum / 4Sum | Sorted order groups duplicates adjacently | Fix k-2 elements; two-pointer finds remaining 2 in O(n) |
| Largest 0-Sum Subarray | Repeated prefix sum implies a zero-sum span between them | Hash map stores first occurrence; answer is the gap |
| Subarray XOR = B | Prefix XOR transforms the question to a lookup | `xor(i+1..j) = prefXOR[j] ^ prefXOR[i]`; count prior prefixes |

The underlying skill is: before writing a loop, ask *"is there a mathematical identity that replaces this search?"* Internalize the five identities above and you can derive every solution from first principles under interview conditions.

---

## 2. Pascal's Triangle

**Problem:** Given `numRows`, return the first `numRows` rows of Pascal's Triangle. Element at row `r`, column `c` (both 0-indexed) equals the binomial coefficient `C(r, c)`.

**LeetCode 118 — Pascal's Triangle**

```
Input:  numRows = 5
Output: [[1],[1,1],[1,2,1],[1,3,3,1],[1,4,6,4,1]]
```

### Mathematical Foundation

**Additive recurrence:** `C(n, r) = C(n-1, r-1) + C(n-1, r)`

This is the basis of the row-by-row construction: every interior element is the sum of the two elements directly above it. Row boundaries are always 1 (`C(n,0) = C(n,n) = 1`).

**Multiplicative formula for consecutive elements within one row:**

```
C(n, r) = C(n, r-1) * (n - r + 1) / r
```

Starting from `C(n,0) = 1`, this lets you compute each subsequent element in O(1) from the previous one, using only integer arithmetic. The division is always exact because `C(n,r)` is an integer — the product of any `r` consecutive integers is divisible by `r!`.

**Connection to combinatorics:** Row `n` (0-indexed) contains the coefficients of `(x+y)^n`. Element at position `r` in row `n` is `C(n,r) = n! / (r! * (n-r)!)`.

### Approach 1 — Brute Force: nCr per Element

For each element, compute `C(r-1, c-1)` from scratch using the multiplicative formula.

```cpp
long long nCr(int n, int r) {
    long long res = 1;
    for (int i = 0; i < r; i++) {
        res = res * (n - i) / (i + 1);  // multiply then divide to stay integer
    }
    return res;
}

vector<vector<int>> generateBrute(int numRows) {
    vector<vector<int>> triangle;
    for (int r = 1; r <= numRows; r++) {
        vector<int> row;
        for (int c = 1; c <= r; c++) {
            row.push_back((int)nCr(r - 1, c - 1));
        }
        triangle.push_back(row);
    }
    return triangle;
}
```

**Time:** O(n^3) — n rows, up to n elements per row, each nCr call is O(n).
**Space:** O(1) auxiliary.

### Approach 2 — Optimal: Row-by-Row Additive Construction

Use the additive recurrence directly. Each row is initialized to all 1s; interior elements are filled from the previous row.

```cpp
vector<vector<int>> generate(int numRows) {
    vector<vector<int>> triangle;
    for (int i = 0; i < numRows; i++) {
        vector<int> row(i + 1, 1);  // size i+1, all initialized to 1
        for (int j = 1; j < i; j++) {
            row[j] = triangle[i - 1][j - 1] + triangle[i - 1][j];
        }
        triangle.push_back(row);
    }
    return triangle;
}
```

**Time:** O(n^2) — total elements = 1 + 2 + ... + n = n(n+1)/2 = O(n^2).
**Space:** O(n^2) for storing the full triangle.

### Generating Row r Alone (Multiplicative Formula)

When you only need one specific row (LC 119 — Pascal's Triangle II):

```cpp
// Returns 0-indexed row r
vector<int> getRow(int r) {
    vector<int> row;
    row.push_back(1);          // C(r,0) = 1
    long long prev = 1;
    for (int c = 1; c <= r; c++) {
        prev = prev * (r - c + 1) / c;  // C(r,c) from C(r,c-1)
        row.push_back((int)prev);
    }
    return row;
}
```

**Why `long long` for `prev`:** The intermediate product `prev * (r - c + 1)` can temporarily exceed `INT_MAX` before the division, even though the final value `C(r,c)` fits in an `int` for `numRows <= 30`.

**Time:** O(r) per row; O(n^2) for all rows.

### Dry Run (Row-by-Row, numRows = 4)

```
i=0: row=[1]              triangle=[[1]]
i=1: row=[1,1]            (interior loop: j from 1 to 0, empty)
i=2: row=[1,_,1]
     j=1: row[1] = triangle[1][0] + triangle[1][1] = 1+1 = 2
     row=[1,2,1]
i=3: row=[1,_,_,1]
     j=1: row[1] = triangle[2][0] + triangle[2][1] = 1+2 = 3
     j=2: row[2] = triangle[2][1] + triangle[2][2] = 2+1 = 3
     row=[1,3,3,1]
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `numRows=1` | `[[1]]` | Single row, single element |
| `numRows=2` | `[[1],[1,1]]` | Second row has no interior elements; j loop is empty |
| `numRows=30` | valid int values | C(29,14)=77,558,760 fits in int; intermediate `long long` handles overflow |

---

## 3. Majority Element II — Boyer-Moore Extended

**Problem:** Given an integer array `nums` of size n, return all elements appearing **more than `floor(n/3)` times**.

**LeetCode 229 — Majority Element II**

```
Input:  [3, 2, 3]                    Output: [3]
Input:  [1, 2]                       Output: [1, 2]
Input:  [1, 1, 1, 3, 3, 2, 2, 2]    Output: [1, 2]
Input:  [1, 2, 3]                    Output: []
```

### Key Observation: At Most 2 Elements Can Qualify

**Proof:** Suppose 3 or more distinct elements each appear more than n/3 times. Then their combined count exceeds `3 * (n/3) = n`, which is impossible since the array has exactly n elements. By contradiction, at most 2 elements can appear more than n/3 times.

**Generalization:** At most `k-1` elements can appear more than `n/k` times. For Majority Element I (n/2), at most 1 candidate; for n/3, at most 2.

### Approach 1 — Hash Map

Count all frequencies; return elements with count > n/3.

```cpp
vector<int> majorityElementHash(vector<int>& nums) {
    unordered_map<int, int> freq;
    for (int x : nums) freq[x]++;
    int n = nums.size();
    vector<int> result;
    for (auto& [val, cnt] : freq) {
        if (cnt > n / 3) result.push_back(val);
    }
    return result;
}
```

**Time:** O(n). **Space:** O(n).

### Approach 2 — Optimal: Extended Boyer-Moore Voting

**Intuition:** Maintain 2 candidates and 2 vote counts. When an element differs from both candidates and both counts are positive, cancel one vote from each candidate (as if this element "consumes" one instance of each). The true majority elements appear too frequently to be fully cancelled.

**Phase 1 — Candidate election:**

```cpp
// Check candidate match FIRST, then adopt, then cancel.
// This order is critical (see Mistake 1 in section 10).
if      (x == cand1)  cnt1++;
else if (x == cand2)  cnt2++;
else if (cnt1 == 0)   { cand1 = x; cnt1 = 1; }
else if (cnt2 == 0)   { cand2 = x; cnt2 = 1; }
else                  { cnt1--; cnt2--; }
```

**Phase 2 — Verification (mandatory):** The vote phase guarantees only that true majority elements survive as candidates. It does not guarantee that all survivors are true majority elements. Always verify by counting actual frequencies in a second pass.

```cpp
vector<int> majorityElement(vector<int>& nums) {
    int n = nums.size();
    int cand1 = INT_MIN, cand2 = INT_MIN;
    int cnt1 = 0, cnt2 = 0;

    // Phase 1: candidate election
    for (int x : nums) {
        if      (x == cand1)  cnt1++;
        else if (x == cand2)  cnt2++;
        else if (cnt1 == 0)   { cand1 = x; cnt1 = 1; }
        else if (cnt2 == 0)   { cand2 = x; cnt2 = 1; }
        else                  { cnt1--; cnt2--; }
    }

    // Phase 2: verify actual frequencies
    cnt1 = cnt2 = 0;
    for (int x : nums) {
        if      (x == cand1) cnt1++;
        else if (x == cand2) cnt2++;
    }

    vector<int> result;
    if (cnt1 > n / 3) result.push_back(cand1);
    if (cnt2 > n / 3) result.push_back(cand2);
    return result;
}
```

**Time:** O(n) — two passes. **Space:** O(1).

**Why the order of checks in Phase 1 matters:** The match checks (`x == cand1`, `x == cand2`) must come before the adoption checks (`cnt == 0`). If `cnt1 == 0` were checked first and `x` happened to equal `cand2`, we would wrongly overwrite `cand1` with `x` instead of incrementing `cnt2`.

### Dry Run

Input: `[1, 1, 1, 3, 3, 2, 2, 2]`, n=8, threshold = floor(8/3) = 2

```
Phase 1:
x=1: cnt1=0      → cand1=1, cnt1=1
x=1: x==cand1   → cnt1=2
x=1: x==cand1   → cnt1=3
x=3: cnt2=0      → cand2=3, cnt2=1
x=3: x==cand2   → cnt2=2
x=2: no match; cnt1>0, cnt2>0 → cnt1=2, cnt2=1
x=2: no match; cnt1>0, cnt2>0 → cnt1=1, cnt2=0
x=2: cnt2=0      → cand2=2, cnt2=1

Candidates: cand1=1, cand2=2

Phase 2 actual counts:
1 appears 3 times; 3 > 2 → include
2 appears 3 times; 3 > 2 → include

Result: [1, 2]
```

**Counter-example showing why verification is needed:**
Input `[1, 2, 3, 4, 5, 6, 7]` (n=7, threshold=2): no element appears more than twice. The vote phase might leave candidates 6 and 7 with count 1 each. The verification pass correctly rejects both since neither count exceeds 2.

---

## 4. 3Sum

**Problem:** Given an integer array `nums`, return all unique triplets `[a, b, c]` such that `a + b + c == 0`. No duplicate triplets.

**LeetCode 15 — 3Sum**

```
Input:  [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]

Input:  [0, 0, 0]
Output: [[0, 0, 0]]

Input:  [0, 1, 1]
Output: []
```

### Approach 1 — Brute Force: Three Nested Loops

Fix all three indices; use a set of sorted triplets to deduplicate.

**Time:** O(n^3 log n). Not acceptable.

### Approach 2 — Better: Two Loops + Hash Set

Fix two indices; use a hash set to find the third in O(1) amortized.

**Time:** O(n^2). **Space:** O(n). Still uses extra space and an outer set for deduplication adds log factor.

### Approach 3 — Optimal: Sort + Two-Pointer

**Key insight:** Sorting puts equal elements adjacent, enabling structural deduplication without a set. After sorting, fix the leftmost element `i` and use two pointers `lo = i+1`, `hi = n-1` to find pairs summing to `-nums[i]`. Moving pointers based on whether the current sum is too small or too large is correct because the array is sorted — increasing `lo` increases the sum, decreasing `hi` decreases it.

```cpp
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    int n = nums.size();
    vector<vector<int>> result;

    for (int i = 0; i < n - 2; i++) {
        if (nums[i] > 0) break;  // sorted: all remaining elements are positive; sum > 0

        // Skip duplicate values for i
        if (i > 0 && nums[i] == nums[i - 1]) continue;

        int lo = i + 1, hi = n - 1;
        while (lo < hi) {
            int sum = nums[i] + nums[lo] + nums[hi];
            if (sum == 0) {
                result.push_back({nums[i], nums[lo], nums[hi]});
                // Skip duplicates for lo and hi BEFORE advancing
                while (lo < hi && nums[lo] == nums[lo + 1]) lo++;
                while (lo < hi && nums[hi] == nums[hi - 1]) hi--;
                lo++;
                hi--;
            } else if (sum < 0) {
                lo++;
            } else {
                hi--;
            }
        }
    }
    return result;
}
```

**Why the duplicate-skip logic works:** After finding a valid triplet, `lo` points at the last element of a run of equal values (the inner while exhausts the run). Then `lo++` moves it to the first element of the next distinct value. Same for `hi`. This prevents producing identical triplets from equal elements at the same positions.

**Time:** O(n log n) + O(n^2) = O(n^2).
**Space:** O(1) auxiliary (output not counted; in-place sort).

### Dry Run

Input: `[-1, 0, 1, 2, -1, -4]`, sorted: `[-4, -1, -1, 0, 1, 2]`

```
i=0: nums[0]=-4, -4 <= 0, proceed.
  lo=1,hi=5: -4+(-1)+2=-3 < 0 → lo++
  lo=2,hi=5: -4+(-1)+2=-3 < 0 → lo++
  lo=3,hi=5: -4+0+2=-2    < 0 → lo++
  lo=4,hi=5: -4+1+2=-1    < 0 → lo++
  lo=5: lo==hi, exit.  No triplets with -4.

i=1: nums[1]=-1, -1 <= 0, proceed.
  lo=2,hi=5: -1+(-1)+2=0 == 0 → push [-1,-1,2]
    skip lo: nums[2]==nums[3]? -1==0? No.
    skip hi: nums[5]==nums[4]? 2==1? No.
    lo=3, hi=4.
  lo=3,hi=4: -1+0+1=0 == 0 → push [-1,0,1]
    skip lo: nums[3]==nums[4]? 0==1? No.
    skip hi: nums[4]==nums[3]? 1==0? No.
    lo=4, hi=3. lo>=hi, exit.

i=2: nums[2]=-1, i>0 && nums[2]==nums[1]? -1==-1 → skip.

i=3: nums[3]=0, 0 <= 0, proceed.
  lo=4,hi=5: 0+1+2=3 > 0 → hi--
  lo=4,hi=4: lo==hi, exit.

Result: [[-1,-1,2], [-1,0,1]]
```

---

## 5. 4Sum

**Problem:** Given array `nums` and integer `target`, return all unique quadruplets `[a, b, c, d]` with `a+b+c+d == target`.

**LeetCode 18 — 4Sum**

```
Input:  nums=[1,0,-1,0,-2,2], target=0
Output: [[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]

Input:  nums=[2,2,2,2,2], target=8
Output: [[2,2,2,2]]
```

### Approach — Sort + Two Outer Loops + Two-Pointer

4Sum is 3Sum with one more fixed element. Fix `i` and `j` with two outer loops; use two-pointer for the remaining pair. This reduces O(n^4) brute force to O(n^3).

```cpp
vector<vector<int>> fourSum(vector<int>& nums, int target) {
    sort(nums.begin(), nums.end());
    int n = nums.size();
    vector<vector<int>> result;

    for (int i = 0; i < n - 3; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue;

        for (int j = i + 1; j < n - 2; j++) {
            // j's dedup condition: j > i+1 (not j > 0) because j starts at i+1
            if (j > i + 1 && nums[j] == nums[j - 1]) continue;

            int lo = j + 1, hi = n - 1;
            while (lo < hi) {
                // MANDATORY: long long — four values near 10^9 each overflow int
                long long sum = (long long)nums[i] + nums[j] + nums[lo] + nums[hi];

                if (sum == target) {
                    result.push_back({nums[i], nums[j], nums[lo], nums[hi]});
                    while (lo < hi && nums[lo] == nums[lo + 1]) lo++;
                    while (lo < hi && nums[hi] == nums[hi - 1]) hi--;
                    lo++;
                    hi--;
                } else if (sum < target) {
                    lo++;
                } else {
                    hi--;
                }
            }
        }
    }
    return result;
}
```

**Why the j dedup condition is `j > i+1`, not `j > 0`:** `j` starts at `i+1`. If `j == i+1` and `nums[j] == nums[i]`, we still need to process this `j` value as the second fixed element — it is a valid second choice. We only skip `j` if it has already been used as the second element at least once in this i-iteration, meaning `j > i+1`.

**Why `long long` is mandatory:** `nums[i]` can be up to `10^9` in magnitude. The sum of four such values can reach `4 * 10^9`, exceeding `INT_MAX ≈ 2.1 * 10^9`. This causes undefined behavior in C++. Casting one operand to `long long` before the addition promotes the entire expression.

**Time:** O(n^3). **Space:** O(1) auxiliary.

### Dry Run

sorted: `[-2, -1, 0, 0, 1, 2]`, target=0

```
i=0 (−2), j=1 (−1):
  lo=2,hi=5: −2+(−1)+0+2=−1 <0 → lo++
  lo=3,hi=5: −2+(−1)+0+2=−1 <0 → lo++
  lo=4,hi=5: −2+(−1)+1+2=0  → push [−2,−1,1,2]; lo=5,hi=4. exit.

i=0 (−2), j=2 (0):
  lo=3,hi=5: −2+0+0+2=0 → push [−2,0,0,2]; lo=4,hi=4. exit.

i=0 (−2), j=3 (0): j>i+1 && nums[3]==nums[2] → skip.

i=1 (−1), j=2 (0):
  lo=3,hi=5: −1+0+0+2=1 >0 → hi--
  lo=3,hi=4: −1+0+0+1=0 → push [−1,0,0,1]; lo=4,hi=3. exit.

i=1 (−1), j=3 (0): j>i+1 && nums[3]==nums[2] → skip.

Remaining i/j combinations produce sums > 0 or lo>=hi immediately.

Result: [[−2,−1,1,2],[−2,0,0,2],[−1,0,0,1]]
```

### Generalization: k-Sum Recursive Pattern

```
kSum(nums, target, k, start):
    if k == 2:
        two-pointer on nums[start..n-1]
    else:
        for i from start to n-k:
            skip if duplicate (i > start && nums[i] == nums[i-1])
            recurse: kSum(nums, target - nums[i], k-1, i+1)
```

Time: O(n^(k-1)) for any k. Space: O(k) recursion depth.

---

## 6. Largest Subarray with 0 Sum

**Problem:** Given an array of positive and negative integers, find the length of the largest subarray whose sum is 0.

**GFG — Largest Subarray with 0 Sum**

```
Input:  [15, -2, 2, -8, 1, 7, 10, 23]
Output: 5   (subarray [-2,2,-8,1,7] from index 1 to 5)

Input:  [1, 2, 3]
Output: 0

Input:  [0]
Output: 1
```

### Approach 1 — Brute Force

Check all O(n^2) subarrays; compute the running sum.

```cpp
int maxLenBrute(vector<int>& arr) {
    int n = arr.size(), maxLen = 0;
    for (int i = 0; i < n; i++) {
        int sum = 0;
        for (int j = i; j < n; j++) {
            sum += arr[j];
            if (sum == 0) maxLen = max(maxLen, j - i + 1);
        }
    }
    return maxLen;
}
```

**Time:** O(n^2). **Space:** O(1).

### Approach 2 — Optimal: Prefix Sum + Hash Map

**Core insight:** Define prefix sum `P[i] = arr[0] + ... + arr[i]`, with `P[-1] = 0`. The sum of subarray `arr[i+1..j]` equals `P[j] - P[i]`. This sum is 0 if and only if `P[j] == P[i]`.

Therefore: **a repeated prefix sum value at positions i and j means the subarray arr[i+1..j] has sum 0.** The length of this subarray is `j - i`.

To find the **longest** such subarray, we want the smallest possible `i` for each repeated prefix sum, so we store only the **first** occurrence of each prefix sum value.

**Algorithm:**
1. Initialize `map = {0: -1}` (prefix sum 0 "occurred" at the virtual index -1, before the array).
2. For each index `i`, update `prefSum += arr[i]`.
3. If `prefSum` is already in the map at index `prev`: update `maxLen = max(maxLen, i - prev)`.
4. Else: store `map[prefSum] = i`. Do NOT overwrite existing entries.
5. Return `maxLen`.

```cpp
int maxLen(vector<int>& arr) {
    int n = arr.size();
    unordered_map<int, int> firstSeen;
    firstSeen[0] = -1;   // virtual index before the array

    int prefSum = 0, maxLen = 0;
    for (int i = 0; i < n; i++) {
        prefSum += arr[i];
        if (firstSeen.count(prefSum)) {
            maxLen = max(maxLen, i - firstSeen[prefSum]);
        } else {
            firstSeen[prefSum] = i;   // store first occurrence only
        }
    }
    return maxLen;
}
```

**Why `firstSeen[0] = -1`:** If `prefSum == 0` at index `i`, the entire subarray `arr[0..i]` has sum 0. Its length is `i - (-1) = i + 1`. Without this seed, subarrays starting from index 0 would be missed.

**Why not overwrite existing entries:** If `P[j] == P[k]` for `j < k`, then for any later index `l > k`, the subarray `arr[j+1..l]` is longer than `arr[k+1..l]`. Keeping the smallest (earliest) index maximizes the length.

**Time:** O(n) average — single pass; hash map O(1) average per operation.
**Space:** O(n) — up to n distinct prefix sums.

### Dry Run

Input: `[15, -2, 2, -8, 1, 7, 10, 23]`

```
firstSeen={0:-1}, prefSum=0, maxLen=0

i=0: arr[0]=15,  prefSum=15.  Not in map. firstSeen={0:-1, 15:0}
i=1: arr[1]=-2,  prefSum=13.  Not in map. firstSeen={..., 13:1}
i=2: arr[2]=2,   prefSum=15.  In map at 0.  len=2-0=2.  maxLen=2.
i=3: arr[3]=-8,  prefSum=7.   Not in map. firstSeen={..., 7:3}
i=4: arr[4]=1,   prefSum=8.   Not in map. firstSeen={..., 8:4}
i=5: arr[5]=7,   prefSum=15.  In map at 0.  len=5-0=5.  maxLen=5.
i=6: arr[6]=10,  prefSum=25.  Not in map.
i=7: arr[7]=23,  prefSum=48.  Not in map.

Return 5.
Subarray arr[1..5] = [-2,2,-8,1,7]; sum = 0.  ✓
```

---

## 7. Count Subarrays with Given XOR

**Problem:** Given array `A` and integer `B`, count the total number of subarrays whose XOR equals `B`.

**InterviewBit — Subarray with Given XOR**

```
Input:  A=[4,2,2,6,4], B=6
Output: 4
(Subarrays: [4,2], [2,2,6], [6], [4,2,2,6,4])

Input:  A=[5,6,7,8,9], B=5
Output: 2
(5^6^7^8^9=5, and [5] alone)
```

### Key Mathematical Property

Define prefix XOR: `X[i] = A[0] ^ A[1] ^ ... ^ A[i]`, with `X[-1] = 0`.

The XOR of subarray `A[i+1..j]` equals:

```
X[j] ^ X[i]
```

This works because XOR is self-inverse: XOR-ing the prefix up to `i` with itself cancels it, leaving only the contribution from `A[i+1..j]`.

We want subarrays where `X[j] ^ X[i] == B`. XOR-ing both sides with `X[j]`:

```
X[i] == X[j] ^ B
```

So for each position `j`, the count of subarrays ending at `j` with XOR equal to `B` is the number of times the prefix XOR value `X[j] ^ B` has appeared before position `j`.

This is the XOR analog of the prefix-sum trick: instead of `P[j] - P[i] == 0`, we have `X[j] ^ X[i] == B`.

### Approach 1 — Brute Force

```cpp
int countBrute(vector<int>& A, int B) {
    int n = A.size(), count = 0;
    for (int i = 0; i < n; i++) {
        int xorVal = 0;
        for (int j = i; j < n; j++) {
            xorVal ^= A[j];
            if (xorVal == B) count++;
        }
    }
    return count;
}
```

**Time:** O(n^2). **Space:** O(1).

### Approach 2 — Optimal: Prefix XOR + Hash Map

```cpp
int solve(vector<int>& A, int B) {
    unordered_map<int, int> freq;
    freq[0] = 1;   // one empty prefix with XOR value 0 exists before the array

    int prefXOR = 0, count = 0;
    for (int val : A) {
        prefXOR ^= val;

        // Count previous prefixes whose XOR with current prefix equals B
        int target = prefXOR ^ B;
        if (freq.count(target)) count += freq[target];

        // Record current prefix XOR AFTER counting (not before)
        freq[prefXOR]++;
    }
    return count;
}
```

**Why `freq[0] = 1`:** If the prefix XOR at position `j` equals `B`, then the subarray `A[0..j]` itself has XOR equal to `B`. This is the case where `X[i] = 0` (the empty prefix). Without `freq[0] = 1`, such subarrays would be missed.

**Why count before updating freq:** We want to count *previous* prefixes. If we updated `freq[prefXOR]++` first, then when `B == 0`, we would query `freq[prefXOR ^ 0] = freq[prefXOR]` and count the current prefix as its own match, which is incorrect.

**Time:** O(n) average. **Space:** O(n).

### Dry Run

Input: `A = [4, 2, 2, 6, 4]`, `B = 6`

```
freq={0:1}, prefXOR=0, count=0

val=4: prefXOR=4.  target=4^6=2.  freq[2]=0.  count=0. freq={0:1,4:1}
val=2: prefXOR=6.  target=6^6=0.  freq[0]=1.  count=1. freq={0:1,4:1,6:1}
val=2: prefXOR=4.  target=4^6=2.  freq[2]=0.  count=1. freq={0:1,4:2,6:1}
val=6: prefXOR=2.  target=2^6=4.  freq[4]=2.  count=3. freq={0:1,4:2,6:1,2:1}
val=4: prefXOR=6.  target=6^6=0.  freq[0]=1.  count=4. freq={0:1,4:2,6:2,2:1}

Return 4.
```

**Verification of the 4 subarrays:**
- `[4,2]` (idx 0-1): 4^2=6 ✓
- `[2,2,6]` (idx 1-3): 2^2^6=6 ✓
- `[6]` (idx 3): 6=6 ✓
- `[4,2,2,6,4]` (idx 0-4): 4^2^2^6^4=6 ✓

---

## 8. The Prefix Hash Unification: Sum and XOR

Problems 6 and 7 are the same algorithm structure applied to two different operations. Recognizing this unification lets you handle any similar variant on sight.

| Dimension | Prefix Sum (0-sum subarray) | Prefix XOR (XOR=B count) |
|---|---|---|
| Running value | `prefSum += arr[i]` | `prefXOR ^= arr[i]` |
| Subarray value formula | `sum(i+1..j) = P[j] - P[i]` | `xor(i+1..j) = X[j] ^ X[i]` |
| Condition for match | `P[j] == P[i]` | `X[i] == X[j] ^ B` |
| Map stores | first occurrence index | frequency count |
| Map query | look up `prefSum` to find prior index | look up `prefXOR ^ B` to find prior count |
| Map seed | `{0: -1}` — virtual index before array | `{0: 1}` — one empty prefix exists |
| Output | maximum subarray length | count of subarrays |

**Why the seeds differ:** The 0-sum problem asks for length, so it needs the *earliest index* where each prefix sum was seen. The XOR problem asks for a count, so it needs the *number of times* each prefix XOR has appeared. The empty prefix (before index 0) contributes one occurrence of XOR value 0, hence `{0:1}`. For length, the empty prefix sits at virtual index -1, hence `{0:-1}`.

**General principle:** For any associative, invertible operation `op` with identity `e`, if you can compute the prefix aggregate and query a hash map, you can answer subarray questions in O(n). Sum uses subtraction as its inverse; XOR uses XOR itself (self-inverse). Product works too, for non-zero elements, using division.

---

## 9. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Pascal's Triangle | Brute (nCr per element) | O(n^3) | O(1) |
| Pascal's Triangle | Row-by-row additive | O(n^2) | O(n^2) output |
| Pascal's Triangle (single row r) | Multiplicative formula | O(r) | O(r) |
| Majority Element II | Hash map | O(n) | O(n) |
| Majority Element II | Boyer-Moore extended | O(n) | O(1) |
| 3Sum | Three nested loops + set | O(n^3 log n) | O(triplets) |
| 3Sum | Two loops + hash set | O(n^2) | O(n) |
| 3Sum | Sort + two-pointer | O(n^2) | O(1) |
| 4Sum | Four nested loops | O(n^4) | O(1) |
| 4Sum | Sort + two loops + two-pointer | O(n^3) | O(1) |
| Largest 0-Sum Subarray | All subarrays | O(n^2) | O(1) |
| Largest 0-Sum Subarray | Prefix sum + hash map | O(n) avg | O(n) |
| Count Subarrays XOR=B | All subarrays | O(n^2) | O(1) |
| Count Subarrays XOR=B | Prefix XOR + hash map | O(n) avg | O(n) |

---

## 10. Common Mistakes and Edge Cases

### Mistake 1 — Wrong Check Order in Boyer-Moore Phase 1

```cpp
// WRONG: checking cnt==0 before candidate match replaces a live candidate
if (cnt1 == 0)         { cand1 = x; cnt1 = 1; }   // might fire when x==cand2!
else if (x == cand1)   cnt1++;
...

// CORRECT: check candidate match first, then adopt, then cancel
if      (x == cand1)  cnt1++;
else if (x == cand2)  cnt2++;
else if (cnt1 == 0)   { cand1 = x; cnt1 = 1; }
else if (cnt2 == 0)   { cand2 = x; cnt2 = 1; }
else                  { cnt1--; cnt2--; }
```

### Mistake 2 — Skipping Duplicates in 3Sum at Wrong Position

```cpp
// WRONG: skips duplicates after advancing, comparing against already-moved position
lo++;
while (lo < hi && nums[lo] == nums[lo - 1]) lo++;   // lo-1 is the position we just left
                                                      // this works but is confusing; watch bounds

// CORRECT canonical form: skip while NEXT equals CURRENT, then advance
while (lo < hi && nums[lo] == nums[lo + 1]) lo++;   // exhaust run
while (lo < hi && nums[hi] == nums[hi - 1]) hi--;   // exhaust run
lo++; hi--;                                          // move past run
```

### Mistake 3 — Missing `long long` in 4Sum

```cpp
// WRONG: sum of four values near 10^9 overflows int
int sum = nums[i] + nums[j] + nums[lo] + nums[hi];

// CORRECT: promote before summing
long long sum = (long long)nums[i] + nums[j] + nums[lo] + nums[hi];
```

### Mistake 4 — Overwriting First Occurrence in 0-Sum Subarray

```cpp
// WRONG: always updates, so map stores the latest index, giving shortest subarrays
firstSeen[prefSum] = i;   // unconditional

// CORRECT: store only first occurrence to maximize subarray length
if (!firstSeen.count(prefSum)) firstSeen[prefSum] = i;
```

### Mistake 5 — Updating freq Before Counting in Subarray XOR

```cpp
// WRONG: when B==0, prefXOR^0==prefXOR; counting after update includes current prefix
freq[prefXOR]++;                     // update first
count += freq.count(prefXOR ^ B)
       ? freq[prefXOR ^ B] : 0;     // counts current prefix against itself when B==0

// CORRECT: count first, then update
count += freq.count(prefXOR ^ B) ? freq[prefXOR ^ B] : 0;
freq[prefXOR]++;
```

### Mistake 6 — Not Verifying Boyer-Moore Candidates

```cpp
// WRONG: returning candidates directly; they may not exceed n/3
return {cand1, cand2};

// CORRECT: verify with a second pass
cnt1 = cnt2 = 0;
for (int x : nums) {
    if      (x == cand1) cnt1++;
    else if (x == cand2) cnt2++;
}
vector<int> result;
if (cnt1 > n / 3) result.push_back(cand1);
if (cnt2 > n / 3) result.push_back(cand2);
return result;
```

### Mistake 7 — Integer Overflow in Pascal's Row Generation

```cpp
// WRONG for large rows: intermediate product overflows int
int prev = 1;
for (int c = 1; c <= r; c++)
    prev = prev * (r - c + 1) / c;   // prev*(r-c+1) can overflow int

// CORRECT: intermediate as long long, cast result to int
long long prev = 1;
for (int c = 1; c <= r; c++) {
    prev = prev * (r - c + 1) / c;
    row.push_back((int)prev);
}
```

### Edge Cases Summary

| Problem | Critical Edge Cases |
|---|---|
| Pascal's Triangle | numRows=1 → `[[1]]`; numRows=2 → interior loop empty; numRows=30 → use long long for intermediate |
| Majority Element II | `[1,2,3]` → none exceed n/3, return `[]`; `[1,2]` → both appear 1 time > 0 = floor(2/3), return `[1,2]` |
| 3Sum | `[0,0,0,0]` → `[[0,0,0]]` only; all positive/negative → `[]`; many equal elements → duplicate skips handle |
| 4Sum | `[2,2,2,2,2]`, target=8 → `[[2,2,2,2]]`; large values → long long |
| 0-Sum Subarray | `[0]` → 1 (prefSum=0 at i=0; 0-(-1)=1); `[1,2,3]` → 0 (no prefix repeats); entire array sums to 0 → n |
| Subarray XOR=B | B=0 → count subarrays with XOR=0 (self-canceling pairs); single-element array A=[B] → 1 |

---

## 11. Interview Patterns and Extensions

### Pattern Recognition Table

| Signal in problem | Algorithm |
|---|---|
| "All unique triplets / quadruplets summing to target" | Sort + fix (k-2) outer elements + two-pointer |
| "Elements appearing more than n/k times" | Boyer-Moore with k-1 candidates + verification |
| "Longest subarray with sum 0 / sum = target" | Prefix sum + hash map, store first index, seed `{0:-1}` |
| "Count subarrays with sum = target" | Prefix sum + hash map, store frequency, seed `{0:1}` |
| "Count subarrays with XOR = B" | Prefix XOR + hash map, store frequency, seed `{0:1}` |
| "Element at row r, col c of Pascal's Triangle" | `C(r-1, c-1)` via multiplicative formula |

### Extensions

**From 3Sum:**
- LC 16 (3Sum Closest): same sort + two-pointer; track minimum `|sum - target|` instead of exact match.
- LC 259 (3Sum Smaller): count triplets with sum < target; when sum < target, `hi - lo` triplets with current `lo` all qualify.
- k-Sum (general): recursive reduction, fix one element per level, two-pointer at the base.

**From Majority Element II:**
- LC 169 (Majority Element I): Boyer-Moore with 1 candidate; verification optional when majority is guaranteed.
- General n/k majority: maintain k-1 candidates; O(n) two passes.

**From 0-Sum Subarray:**
- LC 560 (Subarray Sum Equals K): same prefix-sum hash map but store frequency (not just first index), seed `{0:1}`, output count not length.
- LC 974 (Subarray Sums Divisible by K): prefix sum mod K; two subarrays with same prefix mod K span a subarray divisible by K.
- Equal 0s and 1s: replace 0 with -1; reduces to longest 0-sum subarray.

**From Subarray XOR:**
- LC 421 (Maximum XOR of Two Numbers): trie-based, O(32n).
- LC 260 (Single Number III): XOR all → a^b; find differing bit; partition by that bit; XOR each partition.

### Complexity of k-Sum

| k | Optimal Time | Approach |
|---|---|---|
| 2 | O(n) unsorted with hash, O(n) sorted with two-pointer | Hash or two-pointer |
| 3 | O(n^2) | Fix 1 element, 2Sum on the rest |
| 4 | O(n^3) | Fix 2 elements, 2Sum on the rest |
| k | O(n^(k-1)) | Recursive: fix one, recurse on k-1 |

---

## 12. Quick Revision Cheat Sheet

**Pascal's Triangle**

```cpp
// Full triangle: O(n^2)
for (int i = 0; i < numRows; i++) {
    vector<int> row(i + 1, 1);
    for (int j = 1; j < i; j++)
        row[j] = triangle[i-1][j-1] + triangle[i-1][j];
    triangle.push_back(row);
}

// Single row r: O(r), use long long for intermediate
long long prev = 1;
for (int c = 1; c <= r; c++) {
    prev = prev * (r - c + 1) / c;
    row.push_back((int)prev);
}
```

**Boyer-Moore Majority (at most 2 elements exceed n/3)**

```
// Candidate check order: match → adopt → cancel
if      (x==cand1) cnt1++;
else if (x==cand2) cnt2++;
else if (cnt1==0)  {cand1=x; cnt1=1;}
else if (cnt2==0)  {cand2=x; cnt2=1;}
else               {cnt1--; cnt2--;}
// Always verify with second pass — Phase 1 is necessary but not sufficient.
```

**3Sum — Sort + Two-Pointer**

```cpp
sort(nums.begin(), nums.end());
for (int i = 0; i < n-2; i++) {
    if (nums[i] > 0) break;
    if (i > 0 && nums[i] == nums[i-1]) continue;
    int lo = i+1, hi = n-1;
    while (lo < hi) {
        int s = nums[i]+nums[lo]+nums[hi];
        if (s == 0) {
            push triplet;
            while (lo<hi && nums[lo]==nums[lo+1]) lo++;
            while (lo<hi && nums[hi]==nums[hi-1]) hi--;
            lo++; hi--;
        } else if (s < 0) lo++;
        else hi--;
    }
}
```

**4Sum — Two Outer Loops + Two-Pointer**

```cpp
// Same structure; j dedup: j > i+1 (not j > 0)
// MANDATORY: long long sum = (long long)nums[i] + nums[j] + nums[lo] + nums[hi];
```

**Largest 0-Sum Subarray**

```cpp
unordered_map<int,int> firstSeen;
firstSeen[0] = -1;   // ← seed: empty prefix at virtual index -1
int prefSum = 0, maxLen = 0;
for (int i = 0; i < n; i++) {
    prefSum += arr[i];
    if (firstSeen.count(prefSum)) maxLen = max(maxLen, i - firstSeen[prefSum]);
    else firstSeen[prefSum] = i;   // ← store FIRST occurrence only
}
```

**Count Subarrays with XOR = B**

```cpp
unordered_map<int,int> freq;
freq[0] = 1;   // ← seed: one empty prefix with XOR=0
int prefXOR = 0, count = 0;
for (int val : A) {
    prefXOR ^= val;
    count += freq.count(prefXOR ^ B) ? freq[prefXOR ^ B] : 0;  // ← count BEFORE update
    freq[prefXOR]++;
}
```

**Prefix Hash Comparison (commit this to memory)**

```
Problem              Seed        Map stores    Query         Update rule
0-sum subarray       {0:-1}      first index   prefSum       only if absent
Subarray sum=K       {0:1}       frequency     prefSum-K     always
Subarray XOR=B       {0:1}       frequency     prefXOR^B     always (after query)
```

**Deduplication Template for k-Sum**

```cpp
// Outer index i (starting from `start`):
if (i > start && nums[i] == nums[i-1]) continue;

// Inner lo/hi after finding valid result:
while (lo < hi && nums[lo] == nums[lo+1]) lo++;
while (lo < hi && nums[hi] == nums[hi-1]) hi--;
lo++; hi--;
```
---
