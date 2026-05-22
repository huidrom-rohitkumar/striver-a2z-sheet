# Monotonic Stack and Queue 


> Video References: [NGE](https://youtu.be/e7XQLtOQM3I) | [NGE II](https://youtu.be/7PrncD7v9YQ) | [Rainwater](https://youtu.be/1_5VuquLbXg) | [Subarray Mins](https://youtu.be/v0e8p9JCgRc) | [Asteroids](https://youtu.be/gIrMptNPf5M) | [Subarray Ranges](https://youtu.be/_eYGqw_VDR4) | [Remove K Digits](https://youtu.be/Bzat9vgD0fs) | [Histogram](https://youtu.be/jmbuRzYPGrg) | [Maximal Rect](https://youtu.be/ttVu6G7Ayik)

> Problems Covered: LC 496, 503, 42, 907, 735, 2104, 402, 84, 85 | GFG: Next Smaller, NGE Count

---

## Table of Contents

1. [The Unifying Mental Model](#1-the-unifying-mental-model)
2. [Core Primitives: NGE / NSE / PGE / PSE](#2-core-primitives-nge--nse--pge--pse)
3. [Next Smaller Element (GFG)](#3-next-smaller-element-gfg)
4. [Next Greater Element I (LC 496)](#4-next-greater-element-i-lc-496)
5. [Next Greater Element II — Circular Array (LC 503)](#5-next-greater-element-ii--circular-array-lc-503)
6. [Number of NGEs to the Right (GFG)](#6-number-of-nges-to-the-right-gfg)
7. [Trapping Rainwater (LC 42)](#7-trapping-rainwater-lc-42)
8. [Sum of Subarray Minimums (LC 907)](#8-sum-of-subarray-minimums-lc-907)
9. [Asteroid Collision (LC 735)](#9-asteroid-collision-lc-735)
10. [Sum of Subarray Ranges (LC 2104)](#10-sum-of-subarray-ranges-lc-2104)
11. [Remove K Digits (LC 402)](#11-remove-k-digits-lc-402)
12. [Largest Rectangle in Histogram (LC 84)](#12-largest-rectangle-in-histogram-lc-84)
13. [Maximal Rectangle (LC 85)](#13-maximal-rectangle-lc-85)
14. [The Contribution Technique — Mathematical Justification](#14-the-contribution-technique--mathematical-justification)
15. [Related Concepts and Extensions](#15-related-concepts-and-extensions)
16. [Interview Reference: Pattern Recognition Guide](#16-interview-reference-pattern-recognition-guide)
17. [Complexity Cheat Sheet](#17-complexity-cheat-sheet)
18. [Common Mistakes and Pitfalls](#18-common-mistakes-and-pitfalls)
19. [Quick Revision Cheat Sheet](#19-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Model

Every problem in this chapter is, at its core, about answering one of four **nearest boundary queries** for each element:

| Query | Name | What it finds |
|---|---|---|
| NGE | Next Greater Element | First element to the **right** that is **strictly greater** |
| NSE | Next Smaller Element | First element to the **right** that is **strictly smaller** |
| PGE | Previous Greater Element | First element to the **left** that is **strictly greater** |
| PSE | Previous Smaller Element | First element to the **left** that is **strictly smaller** |

The naive approach scans O(n) per element for O(n²) total. The **monotonic stack** answers all n queries simultaneously in O(n) by maintaining a stack whose elements are always ordered — either strictly increasing or strictly decreasing from bottom to top.

**The core insight:** When you push a new element and it violates the stack's ordering invariant, every element you pop has just found its answer — the new element is the boundary it was waiting for. Because every element is pushed once and popped at most once, the total work across all n insertions is O(n).

**Which stack direction for which query:**

| Stack invariant (bottom → top) | Right tool for |
|---|---|
| Increasing (small → large) | NSE and PGE |
| Decreasing (large → small) | NGE and PSE |

Think of the stack as holding "candidates whose answer has not yet been settled." The moment a new element settles an old candidate's answer, the candidate is popped and recorded. Candidates that survive to the end of the array have no answer on their queried side — they receive the default (−1 or n).

---

## 2. Core Primitives: NGE / NSE / PGE / PSE

These four building blocks appear inside almost every problem in this chapter. Implement them once and reuse everywhere.

### 2.1 Next Greater Element (right-to-left, decreasing stack)

```cpp
// nge[i] = index of the first element to the right strictly greater than arr[i]; n if none
vector<int> nextGreater(const vector<int>& arr) {
    int n = arr.size();
    vector<int> nge(n, n);
    stack<int> st;              // stores indices; values are decreasing top→bottom
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && arr[st.top()] <= arr[i])
            st.pop();           // these are not greater than arr[i]; useless
        if (!st.empty()) nge[i] = st.top();
        st.push(i);
    }
    return nge;
}
```

Left-to-right variant (records answer on pop — useful for contribution problems):

```cpp
vector<int> nextGreaterLR(const vector<int>& arr) {
    int n = arr.size();
    vector<int> nge(n, n);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] < arr[i]) {
            nge[st.top()] = i;   // i is the NGE for the popped index
            st.pop();
        }
        st.push(i);
    }
    return nge;
}
```

### 2.2 Next Smaller Element

```cpp
vector<int> nextSmaller(const vector<int>& arr) {
    int n = arr.size();
    vector<int> nse(n, n);
    stack<int> st;
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && arr[st.top()] >= arr[i])
            st.pop();
        if (!st.empty()) nse[i] = st.top();
        st.push(i);
    }
    return nse;
}
```

### 2.3 Previous Greater Element

```cpp
vector<int> prevGreater(const vector<int>& arr) {
    int n = arr.size();
    vector<int> pge(n, -1);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] <= arr[i])
            st.pop();
        if (!st.empty()) pge[i] = st.top();
        st.push(i);
    }
    return pge;
}
```

### 2.4 Previous Smaller Element

```cpp
vector<int> prevSmaller(const vector<int>& arr) {
    int n = arr.size();
    vector<int> pse(n, -1);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] >= arr[i])
            st.pop();
        if (!st.empty()) pse[i] = st.top();
        st.push(i);
    }
    return pse;
}
```

**Equality handling:** The `>=` vs `>` choice in the pop condition determines whether equal elements count as valid boundaries. This is problem-specific and is where subtle bugs appear — see [Section 18](#18-common-mistakes-and-pitfalls) and [Section 14](#14-the-contribution-technique--mathematical-justification).

---

## 3. Next Smaller Element (GFG)

**Problem:** For each element, find the first element to its right that is strictly smaller. Return -1 if none.

```
[4, 8, 5, 2, 25]  →  [2, 5, 2, -1, -1]
[13, 7, 6, 12]    →  [7, 6, -1, -1]
```

### Approach 1 — Brute Force O(n²)

```cpp
vector<long long> nextSmallerBrute(vector<long long>& arr) {
    int n = arr.size();
    vector<long long> res(n, -1);
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            if (arr[j] < arr[i]) { res[i] = arr[j]; break; }
    return res;
}
```

### Approach 2 — Monotonic Stack O(n)

**Intuition:** Traverse right to left. Maintain an increasing stack (smallest at top). For the current element, pop everything ≥ it — those cannot be NSE for anything at or before the current position, since the current element is closer and already smaller. The remaining stack top is the NSE.

```cpp
vector<long long> nextSmaller(vector<long long>& arr) {
    int n = arr.size();
    vector<long long> res(n, -1);
    stack<long long> st;                 // values, increasing bottom-to-top
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && st.top() >= arr[i])
            st.pop();
        if (!st.empty()) res[i] = st.top();
        st.push(arr[i]);
    }
    return res;
}
```

**Dry run on `[4, 8, 5, 2, 25]`:**

```
i=4: st=[],    → res[4]=-1, push 25. st=[25]
i=3: 25>=2 pop → st=[]. res[3]=-1, push 2. st=[2]
i=2: 2<5 stop  → res[2]=2,  push 5. st=[2,5]
i=1: 5<8 stop  → res[1]=5,  push 8. st=[2,5,8]
i=0: 8>=4 pop, 5>=4 pop, 2<4 stop → res[0]=2, push 4. st=[2,4]
Result: [2, 5, 2, -1, -1]  ✓
```

**Why O(n):** Every element is pushed once and popped at most once — at most 2n total operations.

**Time:** O(n). **Space:** O(n).

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[1,2,3,4]` | `[-1,-1,-1,-1]` | Strictly increasing — no NSE for any element |
| `[4,3,2,1]` | `[3,2,1,-1]` | Decreasing — each NSE is the immediate right neighbor |
| `[3,3,3]` | `[-1,-1,-1]` | Equal elements are NOT strictly smaller |

---

## 4. Next Greater Element I (LC 496)

**Problem:** `nums1` is a subset of `nums2` (all distinct). For each `nums1[i]`, find its NGE in `nums2` (first strictly greater element to its right in `nums2`). Return -1 if none.

```
nums1=[4,1,2], nums2=[1,3,4,2] → [-1,3,-1]
```

### Approach 1 — Brute Force O(m × n)

For each element of `nums1`, locate it in `nums2` then scan right.

### Approach 2 — Monotonic Stack + Hash Map O(n + m)

**Intuition:** Precompute the NGE for every element in `nums2` using a decreasing monotonic stack. Store results in a hash map (safe because all elements are distinct). Answer queries for `nums1` in O(1) each.

```cpp
vector<int> nextGreaterElement(vector<int>& nums1, vector<int>& nums2) {
    unordered_map<int, int> nge;    // value → its NGE value
    stack<int> st;                   // decreasing stack of values

    for (int i = nums2.size() - 1; i >= 0; i--) {
        while (!st.empty() && st.top() <= nums2[i])
            st.pop();
        nge[nums2[i]] = st.empty() ? -1 : st.top();
        st.push(nums2[i]);
    }

    vector<int> res;
    for (int x : nums1) res.push_back(nge[x]);
    return res;
}
```

**Dry run on `nums2 = [1,3,4,2]`:**

```
i=3 (2): st=[],  nge[2]=-1, push 2. st=[2]
i=2 (4): 2<=4 pop. st=[]. nge[4]=-1, push 4. st=[4]
i=1 (3): 4>3 stop. nge[3]=4, push 3. st=[4,3]
i=0 (1): 3>1 stop. nge[1]=3, push 1. st=[4,3,1]

nums1=[4,1,2] → [nge[4], nge[1], nge[2]] = [-1, 3, -1]
```

**Time:** O(n + m). **Space:** O(n).

---

## 5. Next Greater Element II — Circular Array (LC 503)

**Problem:** In a circular array, find the NGE for each element. Circularly means after the last element, the next element is `nums[0]`.

```
[1, 2, 1]      → [2, -1, 2]
[1, 2, 3, 4, 3] → [2, 3, 4, -1, 4]
```

### The Circular Trick

Simulate two passes by iterating `i` from `0` to `2n - 1`, using `i % n` to index the array. Only record answers during the first pass (`i < n`). During the second pass, elements act as the "wrap-around" portion.

```cpp
vector<int> nextGreaterElements(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, -1);
    stack<int> st;                       // decreasing stack of indices

    for (int i = 2 * n - 1; i >= 0; i--) {
        int idx = i % n;
        while (!st.empty() && nums[st.top()] <= nums[idx])
            st.pop();
        if (i < n && !st.empty())
            res[idx] = nums[st.top()];
        st.push(idx);
    }
    return res;
}
```

Left-to-right alternative (push only real indices, pop across both passes):

```cpp
vector<int> nextGreaterElementsV2(vector<int>& nums) {
    int n = nums.size();
    vector<int> res(n, -1);
    stack<int> st;
    for (int i = 0; i < 2 * n; i++) {
        while (!st.empty() && nums[st.top()] < nums[i % n]) {
            res[st.top()] = nums[i % n];
            st.pop();
        }
        if (i < n) st.push(i);   // only push real indices
    }
    return res;
}
```

**Dry run on `[1,2,1]` (n=3), right-to-left:**

```
i=5 (idx=2,val=1): st=[]. push 2. st=[2]
i=4 (idx=1,val=2): nums[2]=1<=2 pop. st=[]. push 1. st=[1]
i=3 (idx=0,val=1): nums[1]=2>1. push 0. st=[1,0]
i=2 (idx=2,val=1): i<3. nums[0]=1<=1 pop. nums[1]=2>1. res[2]=2. push 2. st=[1,2]
i=1 (idx=1,val=2): i<3. nums[2]=1<=2 pop. nums[1]=2<=2 pop. st=[]. res[1]=-1. push 1. st=[1]
i=0 (idx=0,val=1): i<3. nums[1]=2>1. res[0]=2. push 0. st=[1,0]
Result: [2, -1, 2]  ✓
```

**Time:** O(n). **Space:** O(n).

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[5,4,3,2,1]` | `[-1,5,5,5,5]` | Decreasing: every non-max wraps around to the global max |
| `[3,3,3]` | `[-1,-1,-1]` | All equal — no strictly greater element circularly |
| `[1]` | `[-1]` | Single element, nothing greater even circularly |

---

## 6. Number of NGEs to the Right (GFG)

**Problem:** Given array `arr` and Q queries (each is an index `i`), for each query return the count of elements strictly to the right of `i` that are greater than `arr[i]`.

```
arr=[3,4,2,7,5,8,10,6], queries=[0,5,4] → [6,1,2]
```

### Approach — Precompute with Sorted Suffix + Binary Search

Build a sorted list of the suffix `arr[i+1..n-1]` incrementally from right to left. Use binary search to count elements strictly greater than `arr[i]`.

```cpp
vector<int> countNGE(int n, vector<int>& arr, int q, vector<int>& queries) {
    vector<int> sorted_suffix;
    vector<int> count_nge(n, 0);

    for (int i = n - 1; i >= 0; i--) {
        // count elements in sorted_suffix strictly greater than arr[i]
        auto it = upper_bound(sorted_suffix.begin(), sorted_suffix.end(), arr[i]);
        count_nge[i] = sorted_suffix.end() - it;

        // insert arr[i] in sorted order
        it = lower_bound(sorted_suffix.begin(), sorted_suffix.end(), arr[i]);
        sorted_suffix.insert(it, arr[i]);
    }

    vector<int> res;
    for (int idx : queries) res.push_back(count_nge[idx]);
    return res;
}
```

**Time:** O(n² / 2) due to vector insertions. For n ≤ 10^5, a Fenwick tree or persistent segment tree achieves O(n log n + Q log n). For interview purposes, the sorted-suffix approach demonstrates the correct idea. **Space:** O(n).

---

## 7. Trapping Rainwater (LC 42)

**Problem:** Given heights of bars (each width 1), compute total water trapped after rain.

```
[0,1,0,2,1,0,1,3,2,1,2,1] → 6
[4,2,0,3,1,5]             → 10
```

### Core Formula

Water at position `i`:

```
water[i] = max(0, min(maxLeft[i], maxRight[i]) - height[i])
```

### Approach 1 — Brute Force O(n²)

For each position, scan left for max and right for max. O(n) per position.

### Approach 2 — Prefix/Suffix Max Arrays O(n) time, O(n) space

```cpp
int trapPrefixSuffix(vector<int>& height) {
    int n = height.size();
    vector<int> maxLeft(n), maxRight(n);

    maxLeft[0] = height[0];
    for (int i = 1; i < n; i++)
        maxLeft[i] = max(maxLeft[i-1], height[i]);

    maxRight[n-1] = height[n-1];
    for (int i = n-2; i >= 0; i--)
        maxRight[i] = max(maxRight[i+1], height[i]);

    int total = 0;
    for (int i = 0; i < n; i++)
        total += min(maxLeft[i], maxRight[i]) - height[i];
    return total;
}
```

### Approach 3 — Two Pointers O(n) time, O(1) space (Optimal)

**Intuition:** If `maxLeft < maxRight`, the water at the left pointer is determined by `maxLeft` regardless of what the right side looks like in detail. Process whichever side has the smaller boundary and advance that pointer inward.

```cpp
int trap(vector<int>& height) {
    int left = 0, right = height.size() - 1;
    int maxLeft = 0, maxRight = 0, total = 0;

    while (left < right) {
        if (height[left] <= height[right]) {
            if (height[left] >= maxLeft) maxLeft = height[left];
            else total += maxLeft - height[left];
            left++;
        } else {
            if (height[right] >= maxRight) maxRight = height[right];
            else total += maxRight - height[right];
            right--;
        }
    }
    return total;
}
```

**Dry run on `[0,1,0,2,1,0,1,3,2,1,2,1]`:**

```
l=0,r=11,mL=0,mR=0: h[0]=0<=h[11]=1 → 0>=mL(0)→mL=0. total+=0. l=1.
l=1,r=11: h[1]=1<=h[11]=1 → 1>=mL(0)→mL=1. l=2.
l=2,r=11: h[2]=0<=h[11]=1 → 0<mL(1)→total+=1. total=1. l=3.
l=3,r=11: h[3]=2>h[11]=1 → 1>=mR(0)→mR=1. r=10.
l=3,r=10: h[3]=2<=h[10]=2 → 2>=mL(1)→mL=2. l=4.
l=4,r=10: h[4]=1<=h[10]=2 → 1<mL(2)→total+=1. total=2. l=5.
l=5,r=10: h[5]=0→total+=2. total=4. l=6.
l=6,r=10: h[6]=1→total+=1. total=5. l=7.
l=7,r=10: h[7]=3>h[10]=2 → 2>=mR(1)→mR=2. r=9.
l=7,r=9: h[7]=3>h[9]=1 → 1<mR(2)→total+=1. total=6. r=8.
l=7,r=8: h[7]=3>h[8]=2 → 2>=mR(2)→mR=2. r=7.
l=7>=r=7: exit. Total=6.  ✓
```

### Approach 4 — Monotonic Stack O(n) time, O(n) space

Process left to right with a decreasing stack. When a taller bar is encountered, it is the right boundary; the stack-below element is the left boundary; the just-popped element is the valley bottom.

```cpp
int trapStack(vector<int>& height) {
    int n = height.size(), total = 0;
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && height[st.top()] < height[i]) {
            int bot = st.top(); st.pop();
            if (st.empty()) break;
            int left = st.top();
            int width = i - left - 1;
            int water_height = min(height[left], height[i]) - height[bot];
            total += width * water_height;
        }
        st.push(i);
    }
    return total;
}
```

The two-pointer approach is preferred in interviews for O(1) space.

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[3,0,3]` | `3` | Single valley, 3 units |
| `[1,2,3]` | `0` | Monotone increasing — no valleys |
| `[3,2,1]` | `0` | Water flows off edges |

---

## 8. Sum of Subarray Minimums (LC 907)

**Problem:** Sum of the minimums of all contiguous subarrays. Return modulo 10^9 + 7.

```
[3,1,2,4] → 17
```

### The Contribution Technique

Rather than computing the minimum of every subarray, ask: **for each element `arr[i]`, in how many subarrays is it the minimum?**

Define:
- `left[i]` = `i − PSE[i]`, where `PSE[i]` is the index of the Previous element that is **strictly smaller** (or −1). This is the count of left endpoints the subarray can start at while keeping `arr[i]` as the minimum.
- `right[i]` = `NSE[i] − i`, where `NSE[i]` is the index of the Next element that is **smaller or equal** (or n). This is the count of right endpoints.

Then `arr[i]` is the minimum of exactly `left[i] × right[i]` subarrays, contributing `arr[i] × left[i] × right[i]` to the total.

**Why asymmetric equality (strict on one side, non-strict on the other):** When `arr[i] == arr[j]` with `i < j`, the subarray `[i..j]` must be assigned to exactly one of them. Using strict `<` for PSE (so equal counts as "not smaller" — pop on `>=`) and non-strict `<=` for NSE (so equal counts as "smaller or equal" — pop on `>=` for NSE too, but from the right) assigns ownership to the leftmost duplicate. See [Section 14](#14-the-contribution-technique--mathematical-justification).

```cpp
int sumSubarrayMins(vector<int>& arr) {
    const long long MOD = 1e9 + 7;
    int n = arr.size();
    vector<int> pse(n), nse(n);

    // PSE: previous strictly smaller (pop when arr[top] >= arr[i])
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] >= arr[i]) st.pop();
        pse[i] = st.empty() ? -1 : st.top();
        st.push(i);
    }

    // NSE: next smaller or equal (pop when arr[top] >= arr[i])
    while (!st.empty()) st.pop();
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && arr[st.top()] >= arr[i]) st.pop();
        nse[i] = st.empty() ? n : st.top();
        st.push(i);
    }

    long long res = 0;
    for (int i = 0; i < n; i++) {
        long long left = i - pse[i];
        long long right = nse[i] - i;
        res = (res + (long long)arr[i] % MOD * left % MOD * right % MOD) % MOD;
    }
    return (int)res;
}
```

**Dry run on `[3,1,2,4]`:**

```
PSE (pop on >=):
  i=0(3): st=[]. pse[0]=-1. push 0.
  i=1(1): arr[0]=3>=1 pop. st=[]. pse[1]=-1. push 1.
  i=2(2): arr[1]=1<2. pse[2]=1. push 2.
  i=3(4): arr[2]=2<4. pse[3]=2. push 3.
pse = [-1, -1, 1, 2]

NSE (pop on >=, right-to-left):
  i=3(4): st=[]. nse[3]=4. push 3.
  i=2(2): arr[3]=4>=2 pop. st=[]. nse[2]=4. push 2.
  i=1(1): arr[2]=2>=1 pop. st=[]. nse[1]=4. push 1.
  i=0(3): arr[1]=1<3. nse[0]=1. push 0.
nse = [1, 4, 4, 4]

Contributions:
  i=0(3): left=1, right=1. 3*1*1=3
  i=1(1): left=2, right=3. 1*2*3=6
  i=2(2): left=1, right=2. 2*1*2=4
  i=3(4): left=1, right=1. 4*1*1=4
  Total = 3+6+4+4 = 17  ✓
```

**Time:** O(n). **Space:** O(n). **Note:** Use `long long` — `arr[i] × left × right` can reach ~3×10^4 × n² ≈ 3×10^13.

---

## 9. Asteroid Collision (LC 735)

**Problem:** Positive asteroids move right; negative ones move left. Same-direction asteroids never collide. When a right-mover meets a left-mover: smaller explodes; if equal, both explode. Find the final state.

```
[5,10,-5]  → [5,10]    (10 vs -5: 10 wins)
[8,-8]     → []        (equal: both explode)
[10,2,-5]  → [10]      (2 vs -5: -5 wins; 10 vs -5: 10 wins)
[-2,-1,1,2]→ [-2,-1,1,2] (no collision: left-movers and right-movers never meet here)
```

### Approach — Stack Simulation O(n)

**Intuition:** Right-moving asteroids accumulate on the stack. A left-moving asteroid can only collide with the stack top if the top is positive. Resolve collisions in a loop before deciding whether the left-mover survives.

Four sub-cases for a left-moving asteroid `a`:
1. Stack empty or top also negative → push `a` (no collision).
2. `|a| > top` → top is destroyed; continue loop.
3. `|a| == top` → both destroyed; `a` does not survive.
4. `|a| < top` → `a` is destroyed; stop.

```cpp
vector<int> asteroidCollision(vector<int>& asteroids) {
    vector<int> st;
    for (int a : asteroids) {
        bool alive = true;
        while (alive && a < 0 && !st.empty() && st.back() > 0) {
            if (st.back() < -a) {
                st.pop_back();           // right-mover destroyed; continue
            } else if (st.back() == -a) {
                st.pop_back();           // both destroyed
                alive = false;
            } else {
                alive = false;           // left-mover destroyed
            }
        }
        if (alive) st.push_back(a);
    }
    return st;
}
```

**Dry run on `[10, 2, -5]`:**

```
a=10: positive → push. st=[10]
a=2:  positive → push. st=[10,2]
a=-5: negative.
  Loop: top=2 > 0, 2 < 5 → pop. st=[10].
  Loop: top=10 > 0, 10 > 5 → alive=false.
  alive=false → don't push.
Result: [10]  ✓
```

**Time:** O(n) — each asteroid pushed and popped at most once. **Space:** O(n).

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[-1,-2,-3]` | `[-1,-2,-3]` | All left — no collisions |
| `[1,2,3]` | `[1,2,3]` | All right — no collisions |
| `[1,-1,1,-1]` | `[]` | Each pair annihilates |
| `[-5,10,5,-15]` | `[-5,-15]` | -15 destroys 5 then 10 |

---

## 10. Sum of Subarray Ranges (LC 2104)

**Problem:** The range of a subarray is `max − min`. Return the sum of ranges of all subarrays.

```
[1,2,3] → 4
[4,-2,-3,4,1] → 59
```

### Key Decomposition

```
Sum of ranges = Sum of subarray maximums − Sum of subarray minimums
```

This holds because `max[L..R] − min[L..R]` summed over all subarrays equals the sum of all maxima minus the sum of all minima.

Reuse the contribution technique twice: once for maximums (with PGE and NGE), once for minimums (with PSE and NSE). The equality handling is symmetric but flipped:

```cpp
long long sumMax(vector<int>& nums) {
    int n = nums.size();
    vector<int> pge(n), nge(n);

    // PGE: previous greater or equal (pop on <=)
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && nums[st.top()] <= nums[i]) st.pop();
        pge[i] = st.empty() ? -1 : st.top();
        st.push(i);
    }

    // NGE: next strictly greater (pop on <=)
    while (!st.empty()) st.pop();
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && nums[st.top()] <= nums[i]) st.pop();
        nge[i] = st.empty() ? n : st.top();
        st.push(i);
    }

    long long res = 0;
    for (int i = 0; i < n; i++) {
        long long left = i - pge[i];
        long long right = nge[i] - i;
        res += (long long)nums[i] * left * right;
    }
    return res;
}

long long sumMin(vector<int>& nums) {
    int n = nums.size();
    vector<int> pse(n), nse(n);

    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && nums[st.top()] >= nums[i]) st.pop();
        pse[i] = st.empty() ? -1 : st.top();
        st.push(i);
    }
    while (!st.empty()) st.pop();
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && nums[st.top()] >= nums[i]) st.pop();
        nse[i] = st.empty() ? n : st.top();
        st.push(i);
    }

    long long res = 0;
    for (int i = 0; i < n; i++) {
        long long left = i - pse[i];
        long long right = nse[i] - i;
        res += (long long)nums[i] * left * right;
    }
    return res;
}

long long subArrayRanges(vector<int>& nums) {
    return sumMax(nums) - sumMin(nums);
}
```

**Note:** `nums[i]` can be up to 10^9 and `left × right` up to n² ≈ 10^6. Product reaches 10^15 — use `long long` throughout.

**Time:** O(n). **Space:** O(n).

---

## 11. Remove K Digits (LC 402)

**Problem:** Remove exactly `k` digits from `num` to produce the smallest possible number. No leading zeros. Return "0" if empty.

```
"1432219", k=3 → "1219"
"10200",   k=1 → "200"
"10",      k=2 → "0"
```

### The Greedy Insight

A smaller digit in a higher (leftmost) position produces a smaller number. Whenever a digit in the result is larger than the digit immediately following it, removing it makes the result smaller. A **monotone increasing stack** enforces this: whenever the current digit is smaller than the stack top and `k > 0`, pop the top (remove that digit) and decrement `k`.

If after processing all digits `k > 0` (the number was non-decreasing), remove from the tail — those are the largest digits.

```cpp
string removeKdigits(string num, int k) {
    string st;
    for (char c : num) {
        while (k > 0 && !st.empty() && st.back() > c) {
            st.pop_back();
            k--;
        }
        st.push_back(c);
    }

    // If k > 0 after full traversal, remove from tail
    while (k-- > 0) st.pop_back();

    // Strip leading zeros
    int start = 0;
    while (start < (int)st.size() - 1 && st[start] == '0') start++;

    return st.empty() ? "0" : st.substr(start);
}
```

**Dry run on `"1432219"`, k=3:**

```
c='1': st=[]. push. st="1"
c='4': 1<4. push. st="14"
c='3': 4>3, k=3→2: pop. st="1". 1<3, push. st="13"
c='2': 3>2, k=2→1: pop. st="1". 1<2, push. st="12"
c='2': 2==2, push. st="122"
c='1': 2>1, k=1→0: pop. st="12". k=0, stop. push. st="121"
c='9': k=0, push. st="1219"
No trailing removal. No leading zeros.
Result: "1219"  ✓
```

**Dry run on `"10200"`, k=1:**

```
c='1': push. st="1"
c='0': 1>0, k=1→0: pop. st="". push. st="0"
c='2': k=0, push. st="02"
c='0': push. st="020"
c='0': push. st="0200"
Strip leading zeros: "200"  ✓
```

**Time:** O(n) — each character pushed and popped at most once. **Space:** O(n).

### Edge Cases

| Input | k | Output | Reason |
|---|---|---|---|
| `"10"` | 2 | `"0"` | All removed; return "0" |
| `"112"` | 1 | `"11"` | Non-decreasing; remove from tail |
| `"100"` | 1 | `"0"` | After stripping leading zeros from "00" |

---

## 12. Largest Rectangle in Histogram (LC 84)

**Problem:** Given bar heights (each width 1), find the area of the largest rectangle that fits within the histogram.

```
[2,1,5,6,2,3] → 10
[2,4]          → 4
```

### Core Insight

For each bar `i`, the largest rectangle with height `heights[i]` spans from just after the previous shorter bar to just before the next shorter bar:

```
width[i] = NSE[i] - PSE[i] - 1
area[i]  = heights[i] * width[i]
```

where `PSE[i]` = index of the previous strictly smaller bar (−1 if none), and `NSE[i]` = index of the next smaller or equal bar (n if none).

### Approach 1 — Brute Force O(n²)

For each pair (i, j), track the running minimum and compute area.

### Approach 2 — PSE + NSE Arrays (Two-Pass Stack) O(n)

```cpp
int largestRectangleArea(vector<int>& heights) {
    int n = heights.size();
    vector<int> pse(n, -1), nse(n, n);

    // PSE: previous strictly smaller (pop on >=)
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && heights[st.top()] >= heights[i]) st.pop();
        pse[i] = st.empty() ? -1 : st.top();
        st.push(i);
    }

    // NSE: next smaller or equal (pop on >)
    while (!st.empty()) st.pop();
    for (int i = n - 1; i >= 0; i--) {
        while (!st.empty() && heights[st.top()] > heights[i]) st.pop();
        nse[i] = st.empty() ? n : st.top();
        st.push(i);
    }

    int maxArea = 0;
    for (int i = 0; i < n; i++) {
        int width = nse[i] - pse[i] - 1;
        maxArea = max(maxArea, heights[i] * width);
    }
    return maxArea;
}
```

**Dry run on `[2,1,5,6,2,3]`:**

```
PSE: [-1,-1,1,2,1,4]
NSE: [1,6,4,4,6,6]

Areas:
i=0(2): width=1-(-1)-1=1.  area=2
i=1(1): width=6-(-1)-1=6.  area=6
i=2(5): width=4-1-1=2.     area=10  ← max
i=3(6): width=4-2-1=1.     area=6
i=4(2): width=6-1-1=4.     area=8
i=5(3): width=6-4-1=1.     area=3
Answer: 10  ✓
```

### Approach 3 — One-Pass Stack with Sentinel (Classic Interview Solution) O(n)

**Intuition:** Process left to right with an increasing stack. When a shorter bar arrives, the stack-top's bar has found its right boundary (the current bar) and its left boundary (the new stack top). A virtual bar of height 0 at index n flushes all remaining bars.

```cpp
int largestRectangleAreaOnePass(vector<int>& heights) {
    int n = heights.size(), maxArea = 0;
    stack<int> st;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];   // sentinel flushes remaining bars
        while (!st.empty() && heights[st.top()] > h) {
            int height = heights[st.top()]; st.pop();
            int width = st.empty() ? i : i - st.top() - 1;
            maxArea = max(maxArea, height * width);
        }
        st.push(i);
    }
    return maxArea;
}
```

**Why the sentinel:** Without it, bars that are never smaller than anything to their right would never be popped and their areas never computed. The virtual height-0 bar at index n acts as the right boundary for all remaining stack entries.

**Time:** O(n). **Space:** O(n).

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[5,5,5,5]` | `20` | Uniform height, full width |
| `[1,2,3,4,5]` | `9` | Best: bars 3,4,5 with height 3 → area=9 |
| `[0,0,0]` | `0` | All zeros |

---

## 13. Maximal Rectangle (LC 85)

**Problem:** Given a binary matrix, find the area of the largest rectangle containing only 1s.

```
[["1","0","1","0","0"],
 ["1","0","1","1","1"],
 ["1","1","1","1","1"],
 ["1","0","0","1","0"]]  → 6
```

### Core Reduction: Row-by-Row Histogram

For each row `r`, define `height[c]` = the number of consecutive 1s ending at row `r` in column `c`. This transforms each row into a histogram, and the problem reduces to running LC 84 on each histogram.

Height update:
- If `matrix[r][c] == '1'`: `height[c]++`
- If `matrix[r][c] == '0'`: `height[c] = 0`

```cpp
int maximalRectangle(vector<vector<char>>& matrix) {
    if (matrix.empty()) return 0;
    int rows = matrix.size(), cols = matrix[0].size();
    vector<int> heights(cols, 0);
    int maxArea = 0;

    for (int r = 0; r < rows; r++) {
        for (int c = 0; c < cols; c++)
            heights[c] = (matrix[r][c] == '1') ? heights[c] + 1 : 0;
        maxArea = max(maxArea, largestRectangleArea(heights));  // LC 84
    }
    return maxArea;
}
```

**Row-by-row trace on the example:**

```
Row 0: heights=[1,0,1,0,0] → max rect=1
Row 1: heights=[2,0,2,1,1] → max rect=3
Row 2: heights=[3,1,3,2,2] → max rect=6   ← global max
Row 3: heights=[4,0,0,3,0] → max rect=4
```

**Time:** O(rows × cols) — O(cols) per row for height update + O(cols) for LC 84. **Space:** O(cols).

---

## 14. The Contribution Technique — Mathematical Justification

### Claim

For element `arr[i]` with `left[i] = i − PSE[i]` and `right[i] = NSE[i] − i`, `arr[i]` is the minimum of exactly `left[i] × right[i]` subarrays.

### Proof

A subarray `[L..R]` has `arr[i]` as its minimum if and only if:
1. `L ∈ (PSE[i], i]` — the subarray starts at or after the previous smaller element.
2. `R ∈ [i, NSE[i])` — the subarray ends at or before the next smaller element.
3. All elements in `[L..R]` are ≥ `arr[i]` — guaranteed by definitions of PSE and NSE.

The number of valid (L, R) pairs is exactly `(i − PSE[i]) × (NSE[i] − i) = left[i] × right[i]`. This is a bijection: each valid pair corresponds to exactly one subarray where `arr[i]` is the minimum.

### Why Asymmetric Equality Prevents Double-Counting

Consider `arr = [2, 2]`. The subarray `[0..1]` must be assigned to exactly one element.

Using **pop on `>=` for PSE** (strictly smaller boundary) and **pop on `>=` for NSE but from the right** (smaller or equal boundary):
- `pse[0] = -1`, `nse[0] = 1` (index 1 is NSE for 0 because `arr[1] = 2 = arr[0]`)
- `pse[1] = -1`, `nse[1] = 2` (no NSE)

`left[0] × right[0] = 1 × 1 = 1` → arr[0] owns `[0..0]`.
`left[1] × right[1] = 2 × 1 = 2` → arr[1] owns `[0..1]` and `[1..1]`.

Total unique subarrays: `[0..0]`, `[0..1]`, `[1..1]` = 3. Correct. No double-counting.

The assignment rule: **the rightmost duplicate "owns" the subarray spanning both.** Using `>=` on both sides consistently but from different directions achieves this.

---

## 15. Related Concepts and Extensions

### Daily Temperatures (LC 739)

For each day, find how many days until a warmer temperature. Exactly NGE where the answer is `NGE[i] − i`. O(n) with a decreasing monotonic stack.

### Online Stock Span (LC 901)

For each day's stock price, how many consecutive preceding days had prices ≤ current. This is the "span" = how far back the PSE extends. Store (price, span) pairs on the stack. O(1) amortized per query.

### 132 Pattern (LC 456)

Find i < j < k with `nums[i] < nums[k] < nums[j]`. Process right to left with a decreasing stack; maintain a variable for the largest "valley" seen. O(n).

### Car Fleet (LC 853)

Cars converge toward a target. Count how many fleets form. Process by position, use a stack of arrival times.

### Sliding Window Maximum (LC 239) — Monotonic Deque

For the maximum in every window of size k, use a **deque** (double-ended queue) storing indices in decreasing order of value. The front is always the current window's maximum. Advance: pop from back if new element is ≥ back; pop from front if front index is outside window.

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;           // indices, decreasing values
    vector<int> res;
    for (int i = 0; i < (int)nums.size(); i++) {
        if (!dq.empty() && dq.front() <= i - k) dq.pop_front();
        while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
        dq.push_back(i);
        if (i >= k - 1) res.push_back(nums[dq.front()]);
    }
    return res;
}
```

**O(n) time, O(k) space.** Each element is added and removed from the deque at most once.

---

## 16. Interview Reference: Pattern Recognition Guide

| Signal in the problem | Likely tool |
|---|---|
| "First greater/smaller element to left/right" | Monotonic stack (NGE/NSE/PGE/PSE) |
| "Sum/count over all subarrays involving min or max" | Contribution technique + monotonic stack |
| "Largest area or rectangle in 1D histogram or 2D grid" | Histogram + PSE/NSE + one-pass stack |
| "Simulate events where later arrivals cancel earlier ones" | Stack (general simulation) |
| "Build lexicographically smallest/largest subsequence by removal" | Greedy + monotonic increasing stack |
| "Maximum/minimum in all windows of size k" | Monotonic deque |
| "Circular array, find NGE/NSE" | Double-pass with `i % n` |
| "Span: how far back does some condition hold" | Monotonic stack storing cumulative span |

### When to Use Which Stack Direction

| Goal | Stack invariant | Pop condition |
|---|---|---|
| NGE (first greater to right) | Decreasing (large bottom, small top) | `stack.top() <= current` |
| NSE (first smaller to right) | Increasing (small bottom, large top) | `stack.top() >= current` |
| PGE (first greater to left) | Process L→R, decreasing | Pop when `stack.top() <= current`, record then push |
| PSE (first smaller to left) | Process L→R, increasing | Pop when `stack.top() >= current`, record then push |
| Lexicographically smallest subsequence | Non-decreasing | Pop when `top > current && k > 0` |

---

## 17. Complexity Cheat Sheet

| Problem | Brute Force | Optimal Time | Optimal Space |
|---|---|---|---|
| Next Smaller Element | O(n²) | O(n) | O(n) |
| Next Greater Element I (LC 496) | O(m × n) | O(n + m) | O(n) |
| Next Greater Element II (LC 503) | O(n²) | O(n) | O(n) |
| Number of NGEs to Right | O(n × Q) | O(n log n + Q) | O(n) |
| Trapping Rainwater (LC 42) | O(n²) | O(n) | O(1) two-pointer |
| Sum of Subarray Minimums (LC 907) | O(n²) | O(n) | O(n) |
| Asteroid Collision (LC 735) | O(n²) | O(n) | O(n) |
| Sum of Subarray Ranges (LC 2104) | O(n²) | O(n) | O(n) |
| Remove K Digits (LC 402) | O(k × n) | O(n) | O(n) |
| Largest Rectangle in Histogram (LC 84) | O(n²) | O(n) | O(n) |
| Maximal Rectangle (LC 85) | O(r² × c²) | O(r × c) | O(c) |

---

## 18. Common Mistakes and Pitfalls

### Pitfall 1 — Wrong Equality Handling in Contribution Problems

Getting `>=` vs `>` wrong in the pop condition for PSE/NSE causes double-counting of subarrays when duplicates exist.

```cpp
// WRONG for LC 907: both sides use strict < → equal elements double-counted
while (!st.empty() && arr[st.top()] > arr[i]) st.pop();  // NSE: too permissive

// CORRECT: asymmetric handling
// PSE: pop on >=  (strictly smaller boundary)
while (!st.empty() && arr[st.top()] >= arr[i]) st.pop();
// NSE: pop on >=  from right (smaller-or-equal boundary)
while (!st.empty() && arr[st.top()] >= arr[i]) st.pop();
```

The key is consistency — pick one convention (leftmost or rightmost duplicate owns ties) and apply it asymmetrically across PSE and NSE.

### Pitfall 2 — Storing Values Instead of Indices

Histogram problems require computing widths. You cannot compute a width without knowing the position (index) of the boundary.

```cpp
// WRONG: stores values; cannot compute width
stack<int> st;
st.push(heights[i]);        // value — width computation impossible

// CORRECT: stores indices
st.push(i);
int width = i - st.top() - 1;  // possible because we have the index
```

### Pitfall 3 — Forgetting the Sentinel in One-Pass Histogram

Bars that are never shorter than anything to their right stay on the stack and never have their area computed.

```cpp
// WRONG: loop stops at i = n-1; bars remaining on stack are never popped
for (int i = 0; i < n; i++) { ... }

// CORRECT: loop goes to i = n; virtual bar of height 0 flushes everything
for (int i = 0; i <= n; i++) {
    int h = (i == n) ? 0 : heights[i];
    while (!st.empty() && heights[st.top()] > h) { ... }
    st.push(i);
}
```

### Pitfall 4 — Integer Overflow in Area Computation

In LC 84, heights and widths can each reach n ≈ 10^5. Their product reaches 10^10, overflowing `int`.

```cpp
// WRONG:
int area = heights[top] * width;

// CORRECT:
long long area = (long long)heights[top] * width;
```

Similarly in LC 907 and LC 2104: `arr[i] × left × right` can reach ~3×10^13. Always use `long long`.

### Pitfall 5 — Circular Array: Pushing the Same Index Twice Without Guard

In LC 503, pushing `i % n` for both passes causes an index to appear in the stack twice, which can produce incorrect NGE assignments.

```cpp
// RISKY: same index pushed twice
st.push(i % n);            // for i in [0, 2n-1]

// CORRECT: only push real indices; second pass just pops
if (i < n) st.push(i);    // or guard the push
```

### Pitfall 6 — Remove K Digits: Leading Zero Stripping Error

The strip loop must preserve the last character even if it is '0' (to avoid returning "" when the answer is "0").

```cpp
// WRONG: strips all zeros, returns "" for input "10" k=1
while (!st.empty() && st[0] == '0') st.erase(st.begin());

// CORRECT: keep at least one character
int start = 0;
while (start < (int)st.size() - 1 && st[start] == '0') start++;
return st.empty() ? "0" : st.substr(start);
```

---

## 19. Quick Revision Cheat Sheet

### Core Templates

```
NGE (decreasing stack, right-to-left):
  for i = n-1 to 0: pop while st.top() <= arr[i]; nge[i] = top or n; push i.

NSE (increasing stack, right-to-left):
  for i = n-1 to 0: pop while st.top() >= arr[i]; nse[i] = top or n; push i.

PGE (decreasing stack, left-to-right):
  for i = 0 to n-1: pop while st.top() <= arr[i]; pge[i] = top or -1; push i.

PSE (increasing stack, left-to-right):
  for i = 0 to n-1: pop while st.top() >= arr[i]; pse[i] = top or -1; push i.
```

### Problem-Specific Facts

| Problem | Key formula or trick |
|---|---|
| NGE / NSE | Traverse right-to-left; pop on violation; record then push. O(n). |
| LC 503 Circular NGE | Iterate `i` from `2n-1` to `0`; use `i % n`; only record/push when `i < n`. |
| LC 42 Rainwater | `water[i] = min(maxLeft, maxRight) - height[i]`; two-pointer achieves O(1) space. |
| LC 907 Subarray Mins | `arr[i] * (i - PSE[i]) * (NSE[i] - i)` per element; sum with mod 10^9+7. |
| LC 907 Equality rule | PSE uses pop-on-`>=`; NSE uses pop-on-`>=` from the right. Assign tie to leftmost. |
| LC 735 Asteroids | Push positives always. Left-movers collide with positive tops in a loop. |
| LC 2104 Subarray Ranges | `sumMax(nums) − sumMin(nums)`; apply contribution technique twice. |
| LC 402 Remove K Digits | Monotone increasing stack; pop top when `top > current && k > 0`; trim tail if k>0 left; strip leading zeros (keep at least one char). |
| LC 84 Histogram | `area[i] = heights[i] * (NSE[i] - PSE[i] - 1)`; use sentinel height-0 at index n to flush. |
| LC 85 Maximal Rect. | Build `heights[c]` row-by-row (`+1` on '1', reset to 0 on '0'); run LC 84 per row. |

### Invariants to Memorize

```
Decreasing stack → settles NGE (pop when current > top).
Increasing stack → settles NSE (pop when current < top).
Every element pushed once, popped once → O(n) total.
left[i] * right[i] = subarrays where arr[i] is the min/max (contribution technique).
Circular: iterate 2n steps with i%n; push only for i < n.
Histogram width: NSE[i] - PSE[i] - 1 (both boundaries are exclusive).
long long needed when height * width or value * span can exceed ~2 * 10^9.
LC 402: non-decreasing input → remove from tail; always strip leading zeros carefully.
LC 85: runs LC 84 in O(rows * cols) total, not O(rows * cols²).
```

---
