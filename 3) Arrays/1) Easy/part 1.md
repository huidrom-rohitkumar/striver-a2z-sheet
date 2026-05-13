## Resources

- [Arrays Easy Problems — Striver Part 1 (YouTube)](https://youtu.be/37E9ckMDdTk)
- [Arrays Easy Problems — Striver Part 2 (YouTube)](https://youtu.be/wvcQg43_V8U)

---

## Table of Contents

1. [Why Arrays Matter](#1-why-arrays-matter)
2. [Core Mental Models](#2-core-mental-models)
3. [Array Memory Layout and Complexity](#3-array-memory-layout-and-complexity)
4. [Problem: Largest Element in an Array](#4-problem-largest-element-in-an-array)
5. [Problem: Second Largest Element](#5-problem-second-largest-element)
6. [Problem: Check if Array is Sorted and Rotated (LC 1752)](#6-problem-check-if-array-is-sorted-and-rotated-lc-1752)
7. [Problem: Remove Duplicates from Sorted Array (LC 26)](#7-problem-remove-duplicates-from-sorted-array-lc-26)
8. [Problem: Rotate Array (LC 189)](#8-problem-rotate-array-lc-189)
9. [The Two-Pointer Pattern — Unified Reference](#9-the-two-pointer-pattern--unified-reference)
10. [The Reversal Pattern — Unified Reference](#10-the-reversal-pattern--unified-reference)
11. [Common Mistakes and Pitfalls](#11-common-mistakes-and-pitfalls)
12. [Interview Patterns and Extensions](#12-interview-patterns-and-extensions)
13. [Complexity Cheat Sheet](#13-complexity-cheat-sheet)
14. [Quick Revision Cheat Sheet](#14-quick-revision-cheat-sheet)

---

## 1. Why Arrays Matter

Arrays are the foundation of nearly all DSA. Every major topic later — heaps, hash tables, segment trees, DP tables, sliding window, binary search, matrices — is either built directly on arrays or relies on array traversal patterns.

A large fraction of interview problems are secretly array traversal with a carefully chosen invariant. The hard part is rarely the code; it is identifying the traversal strategy, pointer movement, and state to track.

**What sorting enables on arrays:**

| Problem type | What sorting enables |
|---|---|
| Two Sum, 3Sum | Two-pointer technique |
| Merge Intervals | Overlapping intervals become adjacent |
| Duplicate detection | Equal elements are adjacent |
| Greedy strategies | Natural ordering reveals local optima |
| Binary search | Requires sorted input |

---

## 2. Core Mental Models

### Mental Model 1 — Contiguous Memory Gives O(1) Access

Arrays store elements at consecutive memory addresses. Index access computes the address directly:

```
Address(i) = Base + i × element_size
```

The CPU jumps directly to any index in one operation. No traversal needed. This is why arrays are cache-friendly — sequential access loads multiple elements into cache simultaneously, making traversal faster than a linked list in practice even at equal asymptotic complexity.

### Mental Model 2 — Every Traversal Type Has a Purpose

| Traversal type | Primary use |
|---|---|
| Linear (single pointer) | Searching, tracking extrema, counting |
| Two pointers (slow/fast) | In-place rearrangement on sorted arrays |
| Two pointers (opposite ends) | Reversal, symmetric comparison |
| Sliding window | Subarray properties |
| Prefix/suffix | Cumulative computation |
| Circular (`(i+1) % n`) | Rotation, wrapped comparisons |

### Mental Model 3 — Maintain an Invariant During Traversal

Almost every optimal array solution is described by an invariant: a property that holds at every step of the traversal. Stating the invariant before writing code is both a correctness guarantee and the clearest way to communicate your thinking in an interview.

Examples from this set:

| Problem | Invariant |
|---|---|
| Largest element | `maxi` equals the maximum of `arr[0..i]` |
| Second largest | `largest` and `second` are the top two distinct values in `arr[0..i]` |
| Remove duplicates | `nums[0..slow]` contains exactly the unique elements seen so far |
| Sorted and rotated | Number of circular drops examined so far is ≤ 1 |

### Mental Model 4 — In-Place Means Two Regions

Most in-place array problems divide the array into two logical regions: a finalized/processed left side and an unprocessed right side. The slow pointer marks the boundary; the fast pointer scans. When the fast pointer finds something that belongs in the left region, it is moved there and the boundary advances.

---

## 3. Array Memory Layout and Complexity

### Static vs Dynamic Arrays

```cpp
int arr[5];         // static: stack memory, fixed size, no overhead
vector<int> v;      // dynamic: heap memory, resizable
```

`vector` doubles its capacity when full: allocates new memory, copies all elements, frees the old allocation. This costs O(n) for one resize but happens only when the element count doubles. Amortised over n pushbacks: O(1) per operation.

### Time Complexity of Array Operations

| Operation | Complexity | Why |
|---|---|---|
| Access by index | O(1) | Direct address computation |
| Update | O(1) | Direct address computation |
| Traversal | O(n) | Visit every element |
| Search (unsorted) | O(n) | Must check every element |
| Insert at end (vector) | O(1) amortised | Occasional resize costs O(n), amortised O(1) |
| Insert at front | O(n) | All elements must shift right |
| Delete from middle | O(n) | All elements to the right must shift left |

---

## 4. Problem: Largest Element in an Array

### Problem Statement

Given an array of integers, return the maximum element. Array has at least one element.

### Approach 1: Sorting

Sort the array; the last element is the largest.

```cpp
int largest(vector<int>& arr) {
    sort(arr.begin(), arr.end());
    return arr.back();
}
// Time: O(n log n), Space: O(1) in-place sort
```

Destroys original order. Overkill for this task.

### Approach 2: Linear Scan (Optimal)

Initialize the running maximum to `arr[0]` — not to `0` or `INT_MIN`. Starting from 0 is wrong on all-negative arrays. Starting from `INT_MIN` works but is less readable and unnecessary.

```cpp
int largest(vector<int>& arr) {
    int maxi = arr[0];
    for (int i = 1; i < (int)arr.size(); i++) {
        if (arr[i] > maxi) maxi = arr[i];
    }
    return maxi;
}
// Time: O(n), Space: O(1)
```

### Dry Run

```
arr = [3, 1, 9, 2, 7]

maxi = arr[0] = 3
i=1: arr[1]=1,  1>3? No  -> maxi=3
i=2: arr[2]=9,  9>3? Yes -> maxi=9
i=3: arr[3]=2,  2>9? No  -> maxi=9
i=4: arr[4]=7,  7>9? No  -> maxi=9

Return 9
```

**Why initializing to 0 fails:**

```
arr = [-5, -1, -3]
maxi = 0         <- wrong initial value
i=1: -5>0? No
i=2: -1>0? No
i=3: -3>0? No
Return 0         <- wrong. Correct answer is -1.
```

### STL Alternative

```cpp
return *max_element(arr.begin(), arr.end());   // O(n)
```

In interviews, the explicit loop demonstrates understanding. STL is acceptable when brevity is valued.

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[5]` | `5` | Single element |
| `[-3, -1, -4]` | `-1` | All negatives; do not initialize to 0 |
| `[7, 7, 7]` | `7` | All equal |
| `[INT_MIN, INT_MAX]` | `INT_MAX` | Boundary values |

---

## 5. Problem: Second Largest Element

### Problem Statement

Return the second largest **distinct** element. If it does not exist, return -1. Duplicates of the maximum do not count.

### Approach 1: Sorting

Sort, then scan from the right for the first element strictly less than the maximum.

```cpp
int secondLargest(vector<int>& arr) {
    sort(arr.begin(), arr.end());
    int n = arr.size();
    int largest = arr[n - 1];
    for (int i = n - 2; i >= 0; i--) {
        if (arr[i] != largest) return arr[i];
    }
    return -1;
}
// Time: O(n log n), Space: O(1)
```

### Approach 2: Two-Pass Linear Scan

First pass finds the largest; second pass finds the largest strictly less than it.

```cpp
int secondLargest(vector<int>& arr) {
    int largest = arr[0];
    for (int x : arr) largest = max(largest, x);

    int second = -1;
    for (int x : arr) {
        if (x != largest) second = max(second, x);
    }
    return second;
}
// Time: O(n), Space: O(1)
```

### Approach 3: Single-Pass (Optimal)

Maintain `largest` and `second` simultaneously. Three cases per element:

- `x > largest`: old `largest` becomes new `second`; `x` becomes new `largest`.
- `x < largest && x > second`: update `second` only.
- `x == largest`: skip — not a distinct second.

```cpp
int secondLargest(vector<int>& arr) {
    int largest = -1, second = -1;
    for (int x : arr) {
        if (x > largest) {
            second = largest;
            largest = x;
        } else if (x < largest && x > second) {
            second = x;
        }
        // x == largest: do nothing
    }
    return second;
}
// Time: O(n), Space: O(1)
```

### Dry Run (Single-Pass)

```
arr = [12, 35, 1, 10, 34, 1]
largest=-1, second=-1

x=12: 12>-1   -> second=-1,  largest=12
x=35: 35>12   -> second=12,  largest=35
x=1:  1<35, 1>-1? Yes -> second=1,   largest=35
x=10: 10<35, 10>1? Yes -> second=10,  largest=35
x=34: 34<35, 34>10? Yes -> second=34,  largest=35
x=1:  1<35, 1>34? No  -> no change

Return 34
```

**All-duplicate case:**

```
arr = [10, 10, 10]
x=10: 10>-1  -> second=-1, largest=10
x=10: 10==largest -> skip
x=10: 10==largest -> skip
Return -1   (correct: no distinct second)
```

**Duplicate max at start and end:**

```
arr = [2, 1, 2]
x=2: second=-1, largest=2
x=1: 1<2, 1>-1? Yes -> second=1, largest=2
x=2: ==largest -> skip
Return 1   (correct)
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[1]` | `-1` | Only one element |
| `[2, 2, 2]` | `-1` | No distinct second |
| `[1, 2]` | `1` | Two elements |
| `[5, 3, 5]` | `3` | Max duplicated; distinct second is 3 |
| `[-1, -3, -2]` | `-2` | All negatives work correctly |

### Generalization: Kth Largest

The pattern extends: maintain k variables to find the kth largest in one pass. For large k, use a min-heap of size k running in O(n log k).

---

## 6. Problem: Check if Array is Sorted and Rotated (LC 1752)

### Problem Statement

Return `true` if `nums` was originally sorted in non-decreasing order and then rotated some number of positions (including zero). Duplicates are allowed.

```
Input: [3,4,5,1,2]  ->  true   (rotation of [1,2,3,4,5])
Input: [2,1,3,4]    ->  false  (not a rotation of any sorted array)
Input: [1,1,1]      ->  true   (zero rotation, all equal)
```

### Key Insight

Define a **drop** (or descent) as a position where `nums[i] > nums[(i+1) % n]` — treating the array as circular (the element after the last is the first).

A non-decreasing array rotated by k positions has **at most one drop**: the rotation boundary, where the last element of the array (`arr[n-1]`, the original maximum) is followed by the first element (`arr[0]`, the original minimum). All other adjacent pairs are from the original sorted sequence and are non-decreasing.

If there are two or more drops, the array cannot be a rotation of any sorted array.

**Why circular indexing is necessary:** `[1, 2, 3]` (zero rotation) would show 0 drops across `(i, i+1)` pairs for i = 0 to n-2, but the circular pair `(nums[2]=3, nums[0]=1)` would give 1 drop — still ≤ 1, so it correctly returns `true`. Without the circular check, arrays like `[3, 4, 5]` (rotation of `[3,4,5]`, i.e., zero rotation) would pass trivially, but you would miss the structure needed for the correctness argument.

### Approach 1: Brute Force — Try Every Rotation

For each possible starting position, check if the resulting array is sorted.

```cpp
bool check(vector<int>& nums) {
    int n = nums.size();
    for (int start = 0; start < n; start++) {
        bool sorted = true;
        for (int i = 0; i < n - 1; i++) {
            if (nums[(i + start) % n] > nums[(i + start + 1) % n]) {
                sorted = false;
                break;
            }
        }
        if (sorted) return true;
    }
    return false;
}
// Time: O(n^2), Space: O(1)
```

### Approach 2: Count Circular Drops (Optimal)

```cpp
bool check(vector<int>& nums) {
    int n = nums.size();
    int drops = 0;
    for (int i = 0; i < n; i++) {
        if (nums[i] > nums[(i + 1) % n]) {
            drops++;
            if (drops > 1) return false;
        }
    }
    return true;
}
// Time: O(n), Space: O(1)
```

### Dry Run

```
nums = [3, 4, 5, 1, 2],  n=5

i=0: nums[0]=3, nums[1]=4 -> 3>4? No
i=1: nums[1]=4, nums[2]=5 -> 4>5? No
i=2: nums[2]=5, nums[3]=1 -> 5>1? Yes -> drops=1
i=3: nums[3]=1, nums[4]=2 -> 1>2? No
i=4: nums[4]=2, nums[0]=3 -> 2>3? No  (circular: (4+1)%5=0)

drops=1 <= 1 -> return true
```

```
nums = [2, 1, 3, 4],  n=4

i=0: 2>1? Yes -> drops=1
i=1: 1>3? No
i=2: 3>4? No
i=3: 4>nums[0]=2? Yes -> drops=2 -> return false immediately
```

```
nums = [1, 2, 3]  (zero rotation)

i=0: 1>2? No
i=1: 2>3? No
i=2: 3>nums[0]=1? Yes -> drops=1

drops=1 -> return true   (correct: zero rotation is valid)
```

### Formal Correctness Argument

A sorted array `[a0, a1, ..., an-1]` rotated by k positions produces `[ak, ..., an-1, a0, ..., ak-1]`. The original sorted sequence contributes 0 drops among its existing adjacencies. The rotation creates exactly one new adjacency: `(an-1, a0)`. Since `an-1` is the maximum and `a0` is the minimum of the sorted array, `an-1 >= a0`, creating exactly one drop at the rotation boundary. Therefore the drop count is exactly 1 (or 0 for zero rotation). Any count ≥ 2 means the array cannot be a rotation of a sorted array.

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[1]` | `true` | Single element; trivially valid |
| `[1, 2, 3]` | `true` | Sorted, 0 rotations |
| `[3, 1, 2]` | `true` | Rotation of `[1,2,3]` |
| `[2, 1, 3, 4]` | `false` | Two drops: (2,1) and (4,2) circular |
| `[1, 1, 1]` | `true` | All equal, no drops |
| `[3, 3, 1, 2]` | `true` | Duplicate at rotation boundary |

---

## 7. Problem: Remove Duplicates from Sorted Array (LC 26)

### Problem Statement

Given `nums` sorted in non-decreasing order, remove duplicates in-place so each unique element appears once. Return `k` = count of unique elements. The first `k` elements of `nums` must contain the unique values in order. The rest does not matter.

```
Input:  [0,0,1,1,1,2,2,3,3,4]
Output: k=5, nums = [0,1,2,3,4,...]
```

### The Key Observation

The array is **sorted**. All duplicates of any value are therefore **contiguous** — they appear next to each other. No hash set is needed to detect previously seen values. A single comparison `nums[j] != nums[i]` tells you whether `nums[j]` is a new unique value.

### Approach 1: Set-Based (Not Optimal)

```cpp
int removeDuplicates(vector<int>& nums) {
    set<int> unique(nums.begin(), nums.end());
    int i = 0;
    for (int x : unique) nums[i++] = x;
    return i;
}
// Time: O(n log n), Space: O(n)
```

Ignores the sorted structure. Not the intended solution.

### Approach 2: Two-Pointer In-Place (Optimal)

Use two pointers:
- `i` (slow): marks the end of the unique region. `nums[0..i]` always contains the unique elements found so far.
- `j` (fast): scans ahead to find the next new unique element.

When `nums[j] != nums[i]`, a new unique element is found. Advance `i` and copy `nums[j]` to `nums[i]`.

**Invariant:** `nums[0..i]` contains exactly the unique elements of `nums[0..j-1]`, in sorted order, at every step.

```cpp
int removeDuplicates(vector<int>& nums) {
    if (nums.empty()) return 0;
    int i = 0;
    for (int j = 1; j < (int)nums.size(); j++) {
        if (nums[j] != nums[i]) {
            i++;
            nums[i] = nums[j];
        }
        // nums[j] == nums[i]: j advances, i stays — duplicate skipped
    }
    return i + 1;   // count = index of last unique + 1
}
// Time: O(n), Space: O(1)
```

### Dry Run

```
nums = [0, 0, 1, 1, 1, 2, 2, 3, 3, 4]
i=0

j=1: nums[1]=0 == nums[0]=0 -> skip
j=2: nums[2]=1 != nums[0]=0 -> i=1, nums[1]=1   -> [0,1,1,1,1,2,2,3,3,4]
j=3: nums[3]=1 == nums[1]=1 -> skip
j=4: nums[4]=1 == nums[1]=1 -> skip
j=5: nums[5]=2 != nums[1]=1 -> i=2, nums[2]=2   -> [0,1,2,1,1,2,2,3,3,4]
j=6: nums[6]=2 == nums[2]=2 -> skip
j=7: nums[7]=3 != nums[2]=2 -> i=3, nums[3]=3   -> [0,1,2,3,1,2,2,3,3,4]
j=8: nums[8]=3 == nums[3]=3 -> skip
j=9: nums[9]=4 != nums[3]=3 -> i=4, nums[4]=4   -> [0,1,2,3,4,2,2,3,3,4]

Return i+1 = 5
First 5 elements: [0,1,2,3,4]  (correct)
```

### Why `j` traverses each element exactly once

`j` starts at 1 and increments on every iteration. It never moves backward. Total iterations = n-1. Each iteration does O(1) work. Total: O(n).

### Variant — Allow at Most K Occurrences (LC 80)

LeetCode 80 allows each element to appear at most twice. The slow pointer condition changes to compare `nums[j]` against `nums[i - (k-1)]`:

```cpp
int removeDuplicatesK(vector<int>& nums, int k = 2) {
    int i = 0;
    for (int x : nums) {
        if (i < k || nums[i - k] != x) {
            nums[i++] = x;
        }
    }
    return i;
}
// For k=1: LC 26. For k=2: LC 80.
```

### Edge Cases

| Input | Output | First k elements |
|---|---|---|
| `[]` | `0` | `[]` |
| `[1]` | `1` | `[1]` |
| `[1,1,1]` | `1` | `[1]` |
| `[1,2,3]` | `3` | `[1,2,3]` |
| `[1,1,2,2]` | `2` | `[1,2]` |

---

## 8. Problem: Rotate Array (LC 189)

### Problem Statement

Given `nums`, rotate it right by `k` steps. Rotating right by 1 moves the last element to the front.

```
nums = [1,2,3,4,5,6,7], k=3  ->  [5,6,7,1,2,3,4]
```

### Critical Preprocessing: Always Normalize k

```cpp
k = k % n;
```

Rotating an array of size n by exactly n positions returns it to its original state. Rotating by k is identical to rotating by `k % n`. If `k % n == 0`, return immediately. Without this: for `k = 10^5` and `n = 3`, the brute force would make 10^5 passes; the reversal approach would try `reverse(begin, begin + k)` with `k > n`, which is undefined behavior.

### Approach 1: Brute Force — Rotate One Step k Times

Each single right rotation moves the last element to the front by shifting everything right.

```cpp
void rotate_brute(vector<int>& nums, int k) {
    int n = nums.size();
    k %= n;
    for (int step = 0; step < k; step++) {
        int last = nums[n - 1];
        for (int i = n - 1; i > 0; i--) nums[i] = nums[i - 1];
        nums[0] = last;
    }
}
// Time: O(n*k) — TLE for k = n/2 and n = 10^5
// Space: O(1)
```

### Approach 2: Extra Array

Element at original index `i` lands at index `(i + k) % n` after rotation.

```cpp
void rotate_extra(vector<int>& nums, int k) {
    int n = nums.size();
    k %= n;
    vector<int> temp(n);
    for (int i = 0; i < n; i++) temp[(i + k) % n] = nums[i];
    nums = temp;
}
// Time: O(n), Space: O(n)
```

**Dry Run (nums = [1,2,3,4,5,6,7], k=3, n=7):**

```
i=0: temp[(0+3)%7]=temp[3]=1
i=1: temp[(1+3)%7]=temp[4]=2
i=2: temp[(2+3)%7]=temp[5]=3
i=3: temp[(3+3)%7]=temp[6]=4
i=4: temp[(4+3)%7]=temp[0]=5
i=5: temp[(5+3)%7]=temp[1]=6
i=6: temp[(6+3)%7]=temp[2]=7

temp = [5,6,7,1,2,3,4]  (correct)
```

### Approach 3: Reversal Algorithm (Optimal — O(n) time, O(1) space)

**Intuition:** Let the array be `[A | B]` where `A` = first `n-k` elements, `B` = last `k` elements. We want `[B | A]`.

```
Reverse all:          [rev(B) | rev(A)]
Reverse first k:      [B      | rev(A)]
Reverse last n-k:     [B      | A     ]   = desired
```

Each reversal undoes the order-reversal of its segment. The three operations compose into the exact permutation needed.

```cpp
void rotate(vector<int>& nums, int k) {
    int n = nums.size();
    k %= n;
    if (k == 0) return;
    reverse(nums.begin(), nums.end());          // reverse all
    reverse(nums.begin(), nums.begin() + k);    // reverse first k
    reverse(nums.begin() + k, nums.end());      // reverse last n-k
}
// Time: O(n) — each element swapped at most twice across the three passes
// Space: O(1)
```

### Detailed Dry Run (Reversal, nums = [1,2,3,4,5,6,7], k=3, n=7)

```
k = 3%7 = 3

Step 1 — reverse(0..6):
  swap(nums[0],nums[6]): [7,2,3,4,5,6,1]
  swap(nums[1],nums[5]): [7,6,3,4,5,2,1]
  swap(nums[2],nums[4]): [7,6,5,4,3,2,1]
  After step 1:           [7,6,5,4,3,2,1]

Step 2 — reverse(0..2):  ([7,6,5] reversed)
  swap(nums[0],nums[2]): [5,6,7,4,3,2,1]
  After step 2:           [5,6,7,4,3,2,1]

Step 3 — reverse(3..6):  ([4,3,2,1] reversed)
  swap(nums[3],nums[6]): [5,6,7,1,3,2,4]
  swap(nums[4],nums[5]): [5,6,7,1,2,3,4]
  After step 3:           [5,6,7,1,2,3,4]

Result: [5,6,7,1,2,3,4]  (correct)
```

### Approach 4: Cyclic Replacements (Alternative O(1) space)

Each element moves from index `i` to `(i + k) % n`. Follow each element to its destination, displacing whatever was there. When GCD(n, k) > 1, there are multiple independent cycles.

```cpp
void rotate_cyclic(vector<int>& nums, int k) {
    int n = nums.size();
    k %= n;
    if (k == 0) return;
    int count = 0;
    for (int start = 0; count < n; start++) {
        int current = start;
        int prev = nums[start];
        do {
            int next = (current + k) % n;
            int temp = nums[next];
            nums[next] = prev;
            prev = temp;
            current = next;
            count++;
        } while (current != start);
    }
}
// Time: O(n) — every element placed exactly once
// Space: O(1)
```

**Cycle structure:** There are exactly `GCD(n, k)` independent cycles, each of length `n / GCD(n, k)`. The outer loop runs GCD(n, k) times, once per cycle.

The reversal approach is preferred in interviews for clarity. Cyclic replacements demonstrate deeper understanding but are trickier to implement bug-free under pressure.

### Left vs Right Rotation

```
Right rotate by k  =  Left rotate by (n - k)

Left rotate by k:
  reverse(0, k-1)
  reverse(k, n-1)
  reverse(0, n-1)
```

### Approach Comparison

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force (k passes) | O(n*k) | O(1) | TLE for large k |
| Extra array | O(n) | O(n) | Simple but uses extra memory |
| Reversal | O(n) | O(1) | Preferred: clean and in-place |
| Cyclic replacements | O(n) | O(1) | Correct but complex |

### Edge Cases

| Input | k | Output | Reason |
|---|---|---|---|
| `[1,2,3,4,5,6,7]` | `0` | `[1,2,3,4,5,6,7]` | No rotation |
| `[1,2,3,4,5,6,7]` | `7` | `[1,2,3,4,5,6,7]` | k%n=0 |
| `[1,2,3,4,5,6,7]` | `10` | `[5,6,7,1,2,3,4]` | 10%7=3 |
| `[1]` | `5` | `[1]` | Single element |
| `[-1,-100,3,99]` | `2` | `[3,99,-1,-100]` | Negatives work correctly |

---

## 9. The Two-Pointer Pattern — Unified Reference

The two-pointer technique replaces a nested loop with two indices that together cover the array in a single pass.

### When to Apply

- The array is sorted (sorted structure enables logical deductions from position alone).
- You need in-place rearrangement without extra memory.
- There is a natural notion of a "processed" region (maintained by the slow pointer) and an "unprocessed" region (scanned by the fast pointer).

### The Slow-Fast Variant (used in Remove Duplicates)

```
slow (i): boundary of the result region — everything in [0..i] is finalized
fast (j): scanner moving forward through the unprocessed region

Invariant: nums[0..i] is always the valid processed output
```

```cpp
int i = 0;
for (int j = 1; j < n; j++) {
    if (/* nums[j] qualifies for the result */) {
        i++;
        nums[i] = nums[j];
    }
}
return i + 1;
```

### Problems Using This Exact Template

| Problem | Condition to advance i | LeetCode |
|---|---|---|
| Remove Duplicates I | `nums[j] != nums[i]` | 26 |
| Remove Duplicates II (allow 2) | `i < 2 || nums[j] != nums[i-2]` | 80 |
| Remove Element | `nums[j] != val` | 27 |
| Move Zeroes | `nums[j] != 0` | 283 |

### The Opposite-Ends Variant (used in Reversal)

```cpp
int left = 0, right = n - 1;
while (left < right) {
    swap(nums[left], nums[right]);
    left++;
    right--;
}
```

Used in: reverse array, valid palindrome, container with most water, trapping rain water.

---

## 10. The Reversal Pattern — Unified Reference

Reversing a subarray is a fundamental in-place manipulation.

### The Reversal Template

```cpp
void reverseRange(vector<int>& nums, int start, int end) {
    while (start < end) {
        swap(nums[start], nums[end]);
        start++;
        end--;
    }
}
// Time: O(end - start + 1), Space: O(1)
// Each element in the range is touched exactly once.
```

`std::reverse(nums.begin() + start, nums.begin() + end + 1)` is equivalent.

### Derivation of the Three-Reversal Rotation

Let the original array be `[A | B]` where `A = nums[0..n-k-1]` and `B = nums[n-k..n-1]`. Target: `[B | A]`.

```
Start:              [A     | B    ]
After reverse all:  [rev(B) | rev(A)]   -- both segments reversed
After reverse B:    [B      | rev(A)]   -- first k elements (which are rev(B)) reversed
After reverse A:    [B      | A    ]    -- last n-k elements (which are rev(A)) reversed
```

### Where the Reversal Pattern Appears

| Problem | Reversal steps |
|---|---|
| Rotate right by k | reverse all, reverse [0,k-1], reverse [k,n-1] |
| Rotate left by k | reverse [0,k-1], reverse [k,n-1], reverse all |
| Reverse words in a string | reverse all characters, reverse each word |
| Next Permutation | find rightmost descent, swap with successor, reverse suffix |

---

## 11. Common Mistakes and Pitfalls

### Mistake 1: Initializing max to 0 or INT_MIN

```cpp
// WRONG: fails on all-negative arrays
int maxi = 0;   // or INT_MIN
for (int x : arr) maxi = max(maxi, x);
// For arr=[-3,-1,-5], returns 0 — incorrect

// CORRECT: initialize to the first element
int maxi = arr[0];
for (int i = 1; i < (int)arr.size(); i++) maxi = max(maxi, arr[i]);
```

### Mistake 2: Second largest accepting duplicates of max

```cpp
// WRONG: allows x == largest in the else-if branch
else if (x > second) second = x;
// For arr=[5,5,3]: second becomes 5 — wrong. Should be 3.

// CORRECT: strict less-than ensures distinctness
else if (x < largest && x > second) second = x;
```

### Mistake 3: Missing `k %= n` before rotation

```cpp
// WRONG: k can exceed n, causing out-of-bounds in reverse calls
reverse(nums.begin(), nums.begin() + k);   // UB if k > n

// CORRECT: always normalize first
k %= n;
if (k == 0) return;
```

For `k = 10^5` and `n = 3`: `10^5 % 3 = 1`. Without normalization, the reversal boundaries are invalid.

### Mistake 4: Off-by-one in reversal boundaries

```cpp
// WRONG: reversing [0, k] instead of [0, k-1] — includes one extra element
reverse(nums.begin(), nums.begin() + k + 1);

// CORRECT: first k elements span indices [0, k-1]
reverse(nums.begin(), nums.begin() + k);         // [0, k-1]
reverse(nums.begin() + k, nums.end());            // [k, n-1]
```

### Mistake 5: Missing circular wrap in sorted-and-rotated check

```cpp
// WRONG: checks only n-1 pairs, misses the (last, first) circular pair
for (int i = 0; i < n - 1; i++) {
    if (nums[i] > nums[i + 1]) drops++;
}

// CORRECT: check all n circular pairs using modulo
for (int i = 0; i < n; i++) {
    if (nums[i] > nums[(i + 1) % n]) drops++;
}
```

Without the circular check, `[3, 4, 5]` would show 0 drops and pass, but the wrap-around `(5, 3)` is the legitimate rotation boundary that makes it valid — the algorithm still returns the right answer here, but only accidentally. For any genuinely invalid input, missing the circular check can return a wrong result.

### Mistake 6: Not saving key before the shift in insertion sort (applies here as context)

When implementing two-pointer overwrite patterns, always save the value being placed before any overwriting happens. In Remove Duplicates, `nums[i] = nums[j]` overwrites position `i`, not `j`, so no save is needed — but in rotation brute force:

```cpp
// WRONG: nums[n-1] is overwritten before being saved
for (int i = n - 1; i > 0; i--) nums[i] = nums[i - 1];
nums[0] = nums[n-1];   // nums[n-1] is already gone

// CORRECT: save the last element first
int last = nums[n - 1];
for (int i = n - 1; i > 0; i--) nums[i] = nums[i - 1];
nums[0] = last;
```

### Mistake 7: Using erasing from vector during iteration in Remove Duplicates

```cpp
// WRONG: erasing invalidates iterators — undefined behaviour
for (auto it = nums.begin(); it != nums.end(); ++it) {
    if (it != nums.begin() && *it == *(it - 1)) nums.erase(it);
}

// CORRECT: use the two-pointer overwrite approach; never erase during iteration
```

---

## 12. Interview Patterns and Extensions

### Pattern: Single-Pass Min/Max Tracking

The largest-element technique generalizes directly:

| Problem | Variables to track | Complexity |
|---|---|---|
| Largest | 1 (max) | O(n) |
| Second largest | 2 (max, second) | O(n) |
| Min and max simultaneously | 2 vars | O(n), not O(2n) |
| Range (max - min) | 2 vars | O(n) |
| Kth largest (small k) | k vars, bubble-down per element | O(nk) |
| Kth largest (large k) | min-heap of size k | O(n log k) |

Finding min and max together using only ~1.5n comparisons (compare pairs before updating global extrema):

```cpp
// Compare pairs first: 1.5n comparisons vs 2n for two separate passes
int globalMin = INT_MAX, globalMax = INT_MIN;
for (int i = 0; i + 1 < n; i += 2) {
    int lo = min(nums[i], nums[i + 1]);
    int hi = max(nums[i], nums[i + 1]);
    globalMin = min(globalMin, lo);
    globalMax = max(globalMax, hi);
}
if (n % 2 == 1) {  // handle odd-length array
    globalMin = min(globalMin, nums[n - 1]);
    globalMax = max(globalMax, nums[n - 1]);
}
```

### Pattern: Counting Structural Violations

The sorted-and-rotated problem teaches a general technique: instead of trying to reconstruct what the original structure was, count the number of times a structural property is violated. One violation is tolerated; more than one means the structure is fundamentally different.

This generalizes to:
- Count inversions (pairs where `arr[i] > arr[j]` for `i < j`) — measures "unsortedness". Brute force O(n^2); optimal via merge sort O(n log n).
- Count peaks or valleys for wave / bitonic array problems.

### Pattern: Two Pointers on Sorted Arrays

Remove Duplicates (LC 26) is the entry point. The same slow/fast pointer structure solves:

| Problem | Condition to advance slow | LeetCode |
|---|---|---|
| Remove Duplicates I | `nums[j] != nums[i]` | 26 |
| Remove Duplicates II | `i < 2 || nums[j] != nums[i-2]` | 80 |
| Remove Element | `nums[j] != val` | 27 |
| Move Zeroes | `nums[j] != 0` | 283 |
| Merge Sorted Array (in-place) | From the right end | 88 |

For two-pointer problems on sorted arrays where you need to find pairs, use opposite-end pointers (two sum, 3sum, container with most water, trapping rain water).

### Pattern: Reversal for In-Place Rearrangement

| Problem | Reversal steps |
|---|---|
| Rotate right by k | reverse all, reverse [0,k-1], reverse [k,n-1] |
| Rotate left by k | reverse [0,k-1], reverse [k,n-1], reverse all |
| Reverse words in string | reverse all, then reverse each word |
| Next Permutation | find pivot, swap with successor, reverse suffix |

### Harder Problems Built on These Foundations

| Harder problem | Foundation used |
|---|---|
| Find Minimum in Rotated Sorted Array (LC 153) | Sorted-and-rotated structure |
| Search in Rotated Sorted Array (LC 33) | Sorted-and-rotated structure |
| Container With Most Water (LC 11) | Opposite-end two pointers |
| 3Sum (LC 15) | Two pointers on sorted array |
| Trapping Rain Water (LC 42) | Two pointers + max prefix/suffix tracking |
| Kth Largest Element (LC 215) | Quickselect or heap; extends max-tracking |
| Counting Inversions | Extends bubble sort logic; solved via merge sort |
| Minimum Swaps to Sort | Extends selection sort logic; solved via cycle detection |

---

## 13. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Largest Element | Sort | O(n log n) | O(1) |
| Largest Element | Linear scan | O(n) | O(1) |
| Second Largest | Sort + scan | O(n log n) | O(1) |
| Second Largest | Two-pass scan | O(n) | O(1) |
| Second Largest | Single-pass | O(n) | O(1) |
| Sorted and Rotated | Brute (try all rotations) | O(n^2) | O(1) |
| Sorted and Rotated | Count circular drops | O(n) | O(1) |
| Remove Duplicates | Set-based | O(n log n) | O(n) |
| Remove Duplicates | Two pointers | O(n) | O(1) |
| Rotate Array | Brute (k passes) | O(n*k) | O(1) |
| Rotate Array | Extra array | O(n) | O(n) |
| Rotate Array | Reversal | O(n) | O(1) |
| Rotate Array | Cyclic replacements | O(n) | O(1) |

---

## 14. Quick Revision Cheat Sheet

This section is self-contained for same-day interview revision.

---

### Largest Element

```cpp
int maxi = arr[0];   // NOT 0 or INT_MIN — fails on all-negative arrays
for (int i = 1; i < (int)arr.size(); i++) maxi = max(maxi, arr[i]);
return maxi;
```

---

### Second Largest (Single Pass)

```cpp
int largest = -1, second = -1;
for (int x : arr) {
    if (x > largest)                { second = largest; largest = x; }
    else if (x < largest && x > second) { second = x; }   // x < largest is critical
    // x == largest: skip (not a distinct second)
}
return second;   // -1 if no distinct second exists
```

---

### Check Sorted and Rotated

```cpp
int drops = 0, n = nums.size();
for (int i = 0; i < n; i++) {
    if (nums[i] > nums[(i + 1) % n]) drops++;   // circular: % n
    if (drops > 1) return false;
}
return true;
// A sorted-rotated array has at most 1 circular drop
```

---

### Remove Duplicates from Sorted Array

```cpp
int i = 0;
for (int j = 1; j < (int)nums.size(); j++) {
    if (nums[j] != nums[i]) nums[++i] = nums[j];
}
return i + 1;
// i = slow (unique boundary), j = fast (scanner)
// Sorted => duplicates adjacent => no hash set needed
// Return i+1 not i (i is an index; count = index + 1)
```

---

### Rotate Array (Reversal)

```cpp
void rotate(vector<int>& nums, int k) {
    int n = nums.size();
    k %= n;               // MUST normalize — k can exceed n
    if (k == 0) return;
    reverse(nums.begin(), nums.end());           // step 1: reverse all
    reverse(nums.begin(), nums.begin() + k);     // step 2: reverse first k
    reverse(nums.begin() + k, nums.end());       // step 3: reverse last n-k
}
// [A|B] -> [rev(B)|rev(A)] -> [B|rev(A)] -> [B|A]
// Time: O(n), Space: O(1)
```

---

### Key Facts

```
Largest:             O(n), init max to arr[0] not 0
Second largest:      O(n), two vars, x < largest condition prevents counting duplicates of max
Sorted & rotated:    at most 1 circular drop, use (i+1)%n for wrap
Remove duplicates:   two pointers, sorted means adjacency check suffices, return i+1
Rotate right by k:   k %= n first, then three reversals
Rotate left by k:    equivalent to rotate right by (n-k)
Cyclic rotation:     GCD(n,k) independent cycles, each of length n/GCD(n,k)
```

---

### Invariants to State in Interviews

| Problem | Invariant |
|---|---|
| Largest element | `maxi` = maximum of `arr[0..current]` |
| Second largest | `largest` and `second` = top two distinct values in `arr[0..current]` |
| Sorted and rotated | circular drop count ≤ 1 at all checked positions |
| Remove duplicates | `nums[0..i]` = unique elements of `nums[0..j-1]`, in sorted order |
| Rotate (reversal) | after each step, the target permutation is closer to complete |

---

### Extensions to Mention

```
Remove Duplicates  ->  LC 80 (allow k occurrences): change condition to nums[j] != nums[i-k+1]
Largest element    ->  STL: *max_element(arr.begin(), arr.end())
Rotate right k     ->  Rotate left (n-k): reverse [0,k-1], [k,n-1], all
Sorted & rotated   ->  Binary search on result: Find Min (LC 153), Search (LC 33)
Second largest     ->  Kth largest: use min-heap of size k for O(n log k)
```
