

**Topic:** Binary Search Fundamentals
**Pattern:** Search Space Reduction on Monotonic Structures
**Language:** C++


---

## Table of Contents

1. [Core Mental Model](#1-core-mental-model)
2. [Why Binary Search is Correct — Formal Argument](#2-why-binary-search-is-correct--formal-argument)
3. [The Overflow Bug — Integer Midpoint](#3-the-overflow-bug--integer-midpoint)
4. [Two Canonical Loop Variants](#4-two-canonical-loop-variants)
5. [Problem 1 — Binary Search (LC 704)](#5-problem-1--binary-search-lc-704)
6. [The lower\_bound Primitive](#6-the-lower_bound-primitive)
7. [The upper\_bound Primitive](#7-the-upper_bound-primitive)
8. [Problem 2 — Floor in a Sorted Array](#8-problem-2--floor-in-a-sorted-array)
9. [Problem 3 — Ceil in a Sorted Array](#9-problem-3--ceil-in-a-sorted-array)
10. [Problem 4 — Floor and Ceil Together](#10-problem-4--floor-and-ceil-together)
11. [Problem 5 — Search Insert Position (LC 35)](#11-problem-5--search-insert-position-lc-35)
12. [Problem 6 — First and Last Position (LC 34)](#12-problem-6--first-and-last-position-lc-34)
13. [Problem 7 — Count Occurrences](#13-problem-7--count-occurrences)
14. [The Unifying Abstraction](#14-the-unifying-abstraction)
15. [C++ STL: lower\_bound and upper\_bound](#15-c-stl-lower_bound-and-upper_bound)
16. [Edge Cases Master Table](#16-edge-cases-master-table)
17. [Common Mistakes and Pitfalls](#17-common-mistakes-and-pitfalls)
18. [Complexity Analysis](#18-complexity-analysis)
19. [Interview Pattern Recognition](#19-interview-pattern-recognition)
20. [Quick Revision Cheat Sheet](#20-quick-revision-cheat-sheet)

---

## 1. Core Mental Model

Binary search is not a search algorithm — it is a **decision framework for eliminating half of a search space in O(1) per step**. The only prerequisite is a **monotone predicate**: a yes/no question whose answer transitions from `false` to `true` exactly once across the search space. Sorted order is the most common source of such a predicate, but the same framework applies to any problem where "can the answer be X?" is monotone in X.

Every problem in this topic reduces to one of two questions:

- "Does a specific value exist, and where?" — answered by exact binary search.
- "What is the leftmost / rightmost position satisfying a condition?" — answered by `lower_bound` / `upper_bound`.

The entire set of seven problems is unified by two primitives: `lower_bound` and `upper_bound`. Once you can implement both from scratch, every other problem is at most a two-line consequence.

**Loop invariant:** At all times during binary search, the answer (if it exists) lies in `[lo, hi]`. Every step shrinks this interval by at least 1. When `lo > hi`, the interval is empty, so no answer exists in the original range.

**Why binary search is fast:** For `n = 1,000,000`, linear search costs up to 1,000,000 comparisons. Binary search costs at most 20. This gap is why binary search is used everywhere: searching, optimization, answer-space problems, monotonic predicates, scheduling, and allocation.

---

## 2. Why Binary Search is Correct — Formal Argument

**Claim:** A standard binary search on a sorted array of `n` elements terminates in at most `ceil(log2(n + 1))` iterations and either finds the target or correctly concludes it is absent.

**Proof sketch:**

Let the size of the current search space be `n = hi - lo + 1`.

At each step, `mid = lo + (hi - lo) / 2`, which satisfies `lo <= mid <= hi`.

- If `arr[mid] == target`, return immediately.
- If `arr[mid] < target`, move `lo = mid + 1`. New space size is at most `hi - mid <= n/2`.
- If `arr[mid] > target`, move `hi = mid - 1`. New space size is at most `mid - lo <= n/2`.

In each non-returning step the space size strictly decreases by at least half. Starting from `n`, after `k` steps the space is at most `n / 2^k`. Setting `n / 2^k = 1` gives `k = log2(n)`, so the loop terminates in `O(log n)` steps.

**Correctness of the negative result:** The loop invariant is "if the target exists in the original array, it lies in `arr[lo..hi]`." When `lo > hi`, the interval is empty. The invariant then forces the conclusion: target is absent. The invariant is maintained because the sorted property guarantees that when `arr[mid] < target`, all elements in `arr[lo..mid]` are also less than target, so they can all be discarded without losing the target.

---

## 3. The Overflow Bug — Integer Midpoint

**Wrong:**
```cpp
int mid = (lo + hi) / 2;   // lo + hi can overflow int when both are near INT_MAX
```

**Correct:**
```cpp
int mid = lo + (hi - lo) / 2;
```

`hi - lo` is always non-negative (loop condition ensures `lo <= hi`) and at most `INT_MAX`, so no overflow occurs. This is a **critical interview trap** — always use the second form.

---

## 4. Two Canonical Loop Variants

There are exactly two loop structures in practice. Mixing them is the most common source of off-by-one errors. Pick one per function and be consistent.

**Variant 1 — Closed interval `[lo, hi]` with `lo <= hi`** (used for equality search and boundary searches with a result variable):

```cpp
int lo = 0, hi = n - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (arr[mid] == target) return mid;
    else if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}
// target not found
```

When `lo == hi`, one element remains — it is checked. When `lo > hi`, the space is empty. This is the variant used throughout all examples in this file.

**Variant 2 — Half-open interval `[lo, hi)` with `lo < hi`** (natural for `lower_bound` / `upper_bound` when the result can equal `n`):

```cpp
int lo = 0, hi = n;         // hi is one past the last valid index
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (condition(mid)) hi = mid;   // mid could be the answer; narrow from right
    else lo = mid + 1;              // mid is definitely not the answer
}
// lo == hi is the answer (may equal n meaning "not found")
```

Both variants are correct. The examples below use Variant 1 with a result variable to make the logic explicit.

---

## 5. Problem 1 — Binary Search (LC 704)

**Problem:** Given a sorted ascending array `nums` of distinct integers and a `target`, return the index of `target`, or `-1` if absent.
**Links:** [LC 704](https://leetcode.com/problems/binary-search/)
**Constraints:** `1 <= n <= 10^4`, `-10^4 < nums[i], target < 10^4`, all elements unique.

### Approach 1 — Brute Force Linear Scan

Ignores the sorted property. Check every element.

```cpp
int search(vector<int>& nums, int target) {
    for (int i = 0; i < (int)nums.size(); i++)
        if (nums[i] == target) return i;
    return -1;
}
```

Time: O(n). Space: O(1). Fails the O(log n) requirement.

### Approach 2 — Iterative Binary Search (Optimal)

**Intuition:** Because the array is sorted, when we examine `arr[mid]`, we know every element to the left is smaller and every element to the right is larger. One comparison with `target` tells us which half can possibly contain `target`. Discarding the other half and repeating this halves the search space each step, yielding O(log n).

```cpp
int search(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

**Dry run — target found:** `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`

```
Indices:   0    1   2   3   4    5
Values:   -1    0   3   5   9   12

Iter 1: lo=0, hi=5, mid=2, nums[2]=3  < 9  → lo = 3
Iter 2: lo=3, hi=5, mid=4, nums[4]=9 == 9  → return 4
```

**Dry run — target absent:** `nums = [-1, 0, 3, 5, 9, 12]`, `target = 2`

```
Iter 1: lo=0, hi=5, mid=2, nums[2]=3  > 2  → hi = 1
Iter 2: lo=0, hi=1, mid=0, nums[0]=-1 < 2  → lo = 1
Iter 3: lo=1, hi=1, mid=1, nums[1]=0  < 2  → lo = 2
lo=2 > hi=1 → exit → return -1
```

Time: O(log n). Space: O(1).

### Approach 3 — Recursive Binary Search

Identical logic expressed recursively. Each call reduces the problem to a subarray.

```cpp
int helper(vector<int>& nums, int lo, int hi, int target) {
    if (lo > hi) return -1;
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] == target) return mid;
    if (nums[mid] < target) return helper(nums, mid + 1, hi, target);
    return helper(nums, lo, mid - 1, target);
}

int search(vector<int>& nums, int target) {
    return helper(nums, 0, (int)nums.size() - 1, target);
}
```

Time: O(log n). Space: O(log n) — call stack depth. Strictly worse than iterative for this problem; prefer the iterative version in interviews.

---

## 6. The lower\_bound Primitive

`lower_bound(arr, x)` returns the **index of the first element `>= x`**. If all elements are less than `x`, it returns `n`.

This is the most important primitive in this topic. It directly solves or is a component of: ceil, search insert position, first occurrence, and count of occurrences.

**Intuition:** We want the leftmost position where `arr[pos] >= x`. Initialize `ans = n` (default: no valid position). Every time we see `arr[mid] >= x`, `mid` is a valid candidate — record it, then search left for an even earlier valid position. Every time `arr[mid] < x`, `mid` and everything to its left are too small, so move right.

```cpp
// Returns the index of the first element >= target.
// Returns n if all elements are < target.
int lowerBound(vector<int>& arr, int target) {
    int lo = 0, hi = (int)arr.size() - 1;
    int ans = (int)arr.size();   // default: target would go at the end
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] >= target) {
            ans = mid;       // valid candidate; try to go further left
            hi = mid - 1;
        } else {
            lo = mid + 1;    // arr[mid] < target; discard left half including mid
        }
    }
    return ans;
}
```

**Dry run — x not present:** `arr = [2, 3, 7, 10, 11, 11, 25]`, `target = 9`

```
Indices:  0  1  2   3   4   5   6
Values:   2  3  7  10  11  11  25
ans = 7 (n)

Iter 1: lo=0, hi=6, mid=3, arr[3]=10 >= 9 → ans=3, hi=2
Iter 2: lo=0, hi=2, mid=1, arr[1]=3  < 9  → lo=2
Iter 3: lo=2, hi=2, mid=2, arr[2]=7  < 9  → lo=3
lo=3 > hi=2 → exit

ans = 3  (arr[3]=10 is the first element >= 9)
```

**Dry run — x present with duplicates:** `arr = [2, 3, 7, 10, 11, 11, 25]`, `target = 11`

```
Iter 1: lo=0, hi=6, mid=3, arr[3]=10 < 11  → lo=4
Iter 2: lo=4, hi=6, mid=5, arr[5]=11 >= 11 → ans=5, hi=4
Iter 3: lo=4, hi=4, mid=4, arr[4]=11 >= 11 → ans=4, hi=3
lo=4 > hi=3 → exit

ans = 4  (first occurrence of 11)
```

Time: O(log n). Space: O(1).

---

## 7. The upper\_bound Primitive

`upper_bound(arr, x)` returns the **index of the first element strictly greater than `x`**. If all elements are `<= x`, it returns `n`.

**Intuition:** Identical structure to `lower_bound`, but the condition changes from `>= x` to `> x`. Every time we find a position where `arr[mid] > x`, it is a valid candidate — record it and search left for an even earlier one.

```cpp
// Returns the index of the first element > target.
// Returns n if all elements are <= target.
int upperBound(vector<int>& arr, int target) {
    int lo = 0, hi = (int)arr.size() - 1;
    int ans = (int)arr.size();   // default: no element strictly greater
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] > target) {
            ans = mid;       // valid candidate; try to go further left
            hi = mid - 1;
        } else {
            lo = mid + 1;    // arr[mid] <= target; discard left half including mid
        }
    }
    return ans;
}
```

**Dry run:** `arr = [2, 3, 7, 10, 11, 11, 25]`, `target = 11`

```
Iter 1: lo=0, hi=6, mid=3, arr[3]=10 <= 11 → lo=4
Iter 2: lo=4, hi=6, mid=5, arr[5]=11 <= 11 → lo=6
Iter 3: lo=6, hi=6, mid=6, arr[6]=25 > 11  → ans=6, hi=5
lo=6 > hi=5 → exit

ans = 6  (arr[6]=25 is the first element > 11)
```

**Relationship between lower\_bound and upper\_bound:**

```
arr = [ ..., 11, 11, 11, ... ]
              ^           ^
        lower_bound    upper_bound
        (first >= 11)  (first > 11)

count of 11 = upper_bound(11) - lower_bound(11)
```

Everything between `lower_bound` and `upper_bound` is exactly equal to `target`. This is the foundation of count-occurrences and range queries.

Time: O(log n). Space: O(1).

---

## 8. Problem 2 — Floor in a Sorted Array

**Problem:** Given a sorted array `arr[]` and an integer `x`, find the index of the largest element `<= x`. Return `-1` if no such element exists.
**Links:** [GFG Floor in Sorted Array](https://www.geeksforgeeks.org/problems/floor-in-a-sorted-array-1587115620/1)
**Constraints:** `1 <= n <= 10^7`, `0 <= arr[i], x <= 10^18`.

**Definition:** Floor of `x` is the greatest element in the array that does not exceed `x`.

**Intuition:** Maintain `ans = -1`. Every time `arr[mid] <= x`, `mid` is a valid floor candidate — record it, then search right for a potentially larger valid candidate. Every time `arr[mid] > x`, no element from `mid` onward can be the floor, so move left.

Note the direction: floor searches right after a match (we want the largest valid), while ceil searches left after a match (we want the smallest valid). Getting this direction right is the key to not confusing floor and ceil.

```cpp
int findFloor(vector<long long>& arr, long long x) {
    int lo = 0, hi = (int)arr.size() - 1;
    int ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] <= x) {
            ans = mid;       // valid floor candidate; try to find a larger one
            lo = mid + 1;
        } else {
            hi = mid - 1;    // arr[mid] > x; no element from mid onward is a floor
        }
    }
    return ans;
}
```

**Dry run — x between two elements:** `arr = [1, 2, 8, 10, 10, 12, 19]`, `x = 5`

```
Indices:  0  1  2   3   4   5   6
Values:   1  2  8  10  10  12  19
ans = -1

Iter 1: lo=0, hi=6, mid=3, arr[3]=10 > 5  → hi=2
Iter 2: lo=0, hi=2, mid=1, arr[1]=2  <= 5 → ans=1, lo=2
Iter 3: lo=2, hi=2, mid=2, arr[2]=8  > 5  → hi=1
lo=2 > hi=1 → exit

ans = 1  (arr[1]=2 is the largest element <= 5)
```

**Dry run — x smaller than all elements:** `arr = [1, 2, 8, 10, 10, 12, 19]`, `x = 0`

```
Iter 1: lo=0, hi=6, mid=3, arr[3]=10 > 0  → hi=2
Iter 2: lo=0, hi=2, mid=1, arr[1]=2  > 0  → hi=0
Iter 3: lo=0, hi=0, mid=0, arr[0]=1  > 0  → hi=-1
lo=0 > hi=-1 → exit

ans = -1  (no element <= 0)
```

Time: O(log n). Space: O(1).

---

## 9. Problem 3 — Ceil in a Sorted Array

**Problem:** Given a sorted array `arr[]` and an integer `x`, find the smallest element `>= x`. Return `-1` if no such element exists.
**Links:** [GFG Ceil in Sorted Array](https://www.geeksforgeeks.org/problems/ceil-in-a-sorted-array/1), [GFG Ceil the Floor](https://www.geeksforgeeks.org/problems/ceil-the-floor2802/1)
**Constraints:** `1 <= n <= 10^5`.

**Definition:** Ceil of `x` is the smallest element in the array that is at least `x`. This is structurally identical to `lower_bound` — return `arr[lowerBound(x)]` if the index is within bounds, else `-1`.

**Intuition:** Maintain `ans = -1`. Every time `arr[mid] >= x`, it is a valid ceil candidate — record the value and search left for a potentially smaller one.

```cpp
// Returns the ceil value (not the index) of x in arr.
// Returns -1 if all elements are less than x.
int findCeil(vector<int>& arr, int x) {
    int lo = 0, hi = (int)arr.size() - 1;
    int ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] >= x) {
            ans = arr[mid];  // valid ceil candidate; try to find a smaller one
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return ans;
}
```

**Dry run — x between two elements:** `arr = [1, 2, 8, 10, 10, 12, 19]`, `x = 5`

```
Iter 1: lo=0, hi=6, mid=3, arr[3]=10 >= 5 → ans=10, hi=2
Iter 2: lo=0, hi=2, mid=1, arr[1]=2  < 5  → lo=2
Iter 3: lo=2, hi=2, mid=2, arr[2]=8  >= 5 → ans=8, hi=1
lo=2 > hi=1 → exit

ans = 8
```

**Dry run — x larger than all elements:** `arr = [1, 2, 8, 10, 10, 12, 19]`, `x = 20`

```
Iter 1: lo=0, hi=6, mid=3, arr[3]=10 < 20  → lo=4
Iter 2: lo=4, hi=6, mid=5, arr[5]=12 < 20  → lo=6
Iter 3: lo=6, hi=6, mid=6, arr[6]=19 < 20  → lo=7
lo=7 > hi=6 → exit

ans = -1  (no element >= 20)
```

Time: O(log n). Space: O(1).

---

## 10. Problem 4 — Floor and Ceil Together

**Problem:** Given a sorted array `arr[]` and an integer `x`, return `{floor(x), ceil(x)}` where floor is the greatest `<= x` and ceil is the smallest `>= x`. Return `-1` for either if not found.
**Links:** [GFG Ceil the Floor](https://www.geeksforgeeks.org/problems/ceil-the-floor2802/1)

**Intuition:** Run the floor search and the ceil search as two independent binary searches. Do not attempt to find both in a single pass — it complicates the logic without saving time, since both passes are already O(log n). Built-in consistency check: when `x` is present in the array, `floor(x) == ceil(x) == x`.

```cpp
pair<int,int> getFloorAndCeil(vector<int>& arr, int x) {
    int n = arr.size();
    int lo, hi, mid;

    // Floor: largest element <= x
    int floor_val = -1;
    lo = 0; hi = n - 1;
    while (lo <= hi) {
        mid = lo + (hi - lo) / 2;
        if (arr[mid] <= x) { floor_val = arr[mid]; lo = mid + 1; }
        else hi = mid - 1;
    }

    // Ceil: smallest element >= x
    int ceil_val = -1;
    lo = 0; hi = n - 1;
    while (lo <= hi) {
        mid = lo + (hi - lo) / 2;
        if (arr[mid] >= x) { ceil_val = arr[mid]; hi = mid - 1; }
        else lo = mid + 1;
    }

    return {floor_val, ceil_val};
}
```

Time: O(log n). Space: O(1).

---

## 11. Problem 5 — Search Insert Position (LC 35)

**Problem:** Given a sorted array of distinct integers `nums` and a `target`, return the index if `target` is found. If not found, return the index where it would be inserted to maintain sorted order.
**Links:** [LC 35](https://leetcode.com/problems/search-insert-position/)
**Constraints:** `1 <= n <= 10^4`, `-10^4 <= nums[i], target <= 10^4`, all distinct.

**Intuition:** "The index where it would be inserted" is exactly `lower_bound(target)`: the first index where `arr[index] >= target`. If `arr[lb]` equals `target`, the element was found there. If not, `lb` is the insertion point. Either way, `lower_bound` is the direct answer. This is the cleanest demonstration that `lower_bound` is more fundamental than a plain equality search — exact search is just `lower_bound` plus a verification.

```cpp
int searchInsert(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    int ans = (int)nums.size();  // default: insert at end
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= target) {
            ans = mid;
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return ans;
}
```

**Dry run — target present:** `nums = [1, 3, 5, 6]`, `target = 5`

```
Indices: 0  1  2  3
Values:  1  3  5  6
ans = 4

Iter 1: lo=0, hi=3, mid=1, nums[1]=3 < 5  → lo=2
Iter 2: lo=2, hi=3, mid=2, nums[2]=5 >= 5 → ans=2, hi=1
lo=2 > hi=1 → exit

ans = 2  (found at index 2)
```

**Dry run — target absent, insert in middle:** `nums = [1, 3, 5, 6]`, `target = 2`

```
Iter 1: lo=0, hi=3, mid=1, nums[1]=3 >= 2 → ans=1, hi=0
Iter 2: lo=0, hi=0, mid=0, nums[0]=1 < 2  → lo=1
lo=1 > hi=0 → exit

ans = 1  (insert between index 0 and 1)
```

**Dry run — target larger than all elements:** `nums = [1, 3, 5, 6]`, `target = 7`

```
Iter 1: lo=0, hi=3, mid=1, nums[1]=3 < 7  → lo=2
Iter 2: lo=2, hi=3, mid=2, nums[2]=5 < 7  → lo=3
Iter 3: lo=3, hi=3, mid=3, nums[3]=6 < 7  → lo=4
lo=4 > hi=3 → exit

ans = 4  (insert at end; default value was correct)
```

Time: O(log n). Space: O(1).

---

## 12. Problem 6 — First and Last Position (LC 34)

**Problem:** Given `nums` sorted in non-decreasing order (duplicates allowed) and a `target`, find the starting and ending position of `target`. Return `[-1, -1]` if absent.
**Links:** [LC 34](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/), [GFG Number of Occurrence](https://www.geeksforgeeks.org/problems/number-of-occurrence2259/1)
**Constraints:** `0 <= n <= 10^5`, `-10^9 <= nums[i] <= 10^9`.

**Key insight:** With duplicates, a single binary search lands on some occurrence of `target`, not necessarily the first or last. Two separate searches are required: one that biases left (first occurrence) and one that biases right (last occurrence).

### First Occurrence

**Intuition:** When `arr[mid] == target`, record `mid` as a candidate but keep searching left (`hi = mid - 1`) because an earlier occurrence may exist.

```cpp
int firstOccurrence(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    int ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) {
            ans = mid;
            hi = mid - 1;    // keep searching left
        } else if (nums[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}
```

### Last Occurrence

**Intuition:** When `arr[mid] == target`, record `mid` but keep searching right (`lo = mid + 1`) because a later occurrence may exist.

```cpp
int lastOccurrence(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    int ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) {
            ans = mid;
            lo = mid + 1;    // keep searching right
        } else if (nums[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}
```

### Combined Solution

```cpp
vector<int> searchRange(vector<int>& nums, int target) {
    int first = firstOccurrence(nums, target);
    if (first == -1) return {-1, -1};
    return {first, lastOccurrence(nums, target)};
}
```

**Dry run:** `nums = [5, 7, 7, 8, 8, 10]`, `target = 8`

```
Indices:  0  1  2  3  4   5
Values:   5  7  7  8  8  10

First occurrence:
  ans = -1
  Iter 1: lo=0, hi=5, mid=2, nums[2]=7  < 8  → lo=3
  Iter 2: lo=3, hi=5, mid=4, nums[4]=8 == 8  → ans=4, hi=3
  Iter 3: lo=3, hi=3, mid=3, nums[3]=8 == 8  → ans=3, hi=2
  lo=3 > hi=2 → exit  →  first = 3

Last occurrence:
  ans = -1
  Iter 1: lo=0, hi=5, mid=2, nums[2]=7  < 8  → lo=3
  Iter 2: lo=3, hi=5, mid=4, nums[4]=8 == 8  → ans=4, lo=5
  Iter 3: lo=5, hi=5, mid=5, nums[5]=10 > 8  → hi=4
  lo=5 > hi=4 → exit  →  last = 4

Result: [3, 4]
```

**Dry run — target absent:** `nums = [5, 7, 7, 8, 8, 10]`, `target = 6`

```
First occurrence:
  Iter 1: lo=0, hi=5, mid=2, nums[2]=7 > 6 → hi=1
  Iter 2: lo=0, hi=1, mid=0, nums[0]=5 < 6 → lo=1
  Iter 3: lo=1, hi=1, mid=1, nums[1]=7 > 6 → hi=0
  lo=1 > hi=0 → exit  →  first = -1  →  return [-1, -1]
```

**Alternative formulation using lower\_bound and upper\_bound directly:**

The first occurrence equals `lower_bound(target)` (if `arr[lb] == target`). The last occurrence equals `upper_bound(target) - 1`. This formulation makes the connection to the two primitives explicit.

```cpp
vector<int> searchRange(vector<int>& nums, int target) {
    int n = nums.size();

    // lower_bound: first index with value >= target
    int lo = 0, hi = n - 1, lb = n;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= target) { lb = mid; hi = mid - 1; }
        else lo = mid + 1;
    }
    if (lb == n || nums[lb] != target) return {-1, -1};

    // upper_bound: first index with value > target
    lo = 0; hi = n - 1;
    int ub = n;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > target) { ub = mid; hi = mid - 1; }
        else lo = mid + 1;
    }
    return {lb, ub - 1};
}
```

Time: O(log n) — two independent binary searches. Space: O(1).

---

## 13. Problem 7 — Count Occurrences

**Problem:** Given a sorted array `arr[]` and a number `target`, find the total count of `target` in `arr[]`. Return 0 if absent.
**Links:** [GFG Number of Occurrence](https://www.geeksforgeeks.org/problems/number-of-occurrence2259/1)
**Constraints:** `1 <= n <= 10^6`.

**Intuition:** The occurrences of `target` form a contiguous block in a sorted array. This block starts at `lower_bound(target)` and ends just before `upper_bound(target)`. Therefore: count = `upper_bound(target) - lower_bound(target)`. No explicit existence check is needed — if `target` is absent, `lower_bound` and `upper_bound` point to the same index and the count is 0.

```cpp
int countOccurrences(vector<int>& arr, int target) {
    int n = arr.size();

    // lower_bound: first index with arr[i] >= target
    int lo = 0, hi = n - 1, lb = n;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] >= target) { lb = mid; hi = mid - 1; }
        else lo = mid + 1;
    }

    // upper_bound: first index with arr[i] > target
    lo = 0; hi = n - 1;
    int ub = n;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] > target) { ub = mid; hi = mid - 1; }
        else lo = mid + 1;
    }

    return ub - lb;
}
```

**Dry run — target present multiple times:** `arr = [1, 1, 2, 2, 2, 2, 3]`, `target = 2`

```
Indices:  0  1  2  3  4  5  6
Values:   1  1  2  2  2  2  3

lower_bound(2):
  Iter 1: lo=0, hi=6, mid=3, arr[3]=2 >= 2 → lb=3, hi=2
  Iter 2: lo=0, hi=2, mid=1, arr[1]=1 < 2  → lo=2
  Iter 3: lo=2, hi=2, mid=2, arr[2]=2 >= 2 → lb=2, hi=1
  exit → lb = 2

upper_bound(2):
  Iter 1: lo=0, hi=6, mid=3, arr[3]=2 <= 2 → lo=4
  Iter 2: lo=4, hi=6, mid=5, arr[5]=2 <= 2 → lo=6
  Iter 3: lo=6, hi=6, mid=6, arr[6]=3 > 2  → ub=6, hi=5
  exit → ub = 6

count = ub - lb = 6 - 2 = 4
```

**Dry run — target absent:** `arr = [1, 1, 2, 2, 2, 2, 3]`, `target = 4`

```
lower_bound(4): lb = 7 (n, since all elements < 4)
upper_bound(4): ub = 7 (n, since all elements <= 4... wait, 4 > all, so ub = n)
count = 7 - 7 = 0
```

Time: O(log n). Space: O(1).

---

## 14. The Unifying Abstraction

Every problem in this topic reduces to at most two calls to `lower_bound` and `upper_bound`. This table is the conceptual core — internalize it.

| Problem | Reduction |
|---|---|
| Binary Search (exact) | `lb = lower_bound(target)`. Return `lb` if `arr[lb] == target`, else `-1`. |
| Search Insert Position | Return `lower_bound(target)` directly. |
| Ceil | `lb = lower_bound(target)`. Return `arr[lb]` if `lb < n`, else `-1`. |
| Floor | `ub = upper_bound(target)`. Floor index = `ub - 1` if `ub > 0`, else `-1`. |
| First Occurrence | `lb = lower_bound(target)`. Return `lb` if `arr[lb] == target`, else `-1`. |
| Last Occurrence | `ub = upper_bound(target)`. Return `ub - 1` after verifying target exists. |
| Count Occurrences | `upper_bound(target) - lower_bound(target)`. |
| Count in range [L, R] | `upper_bound(R) - lower_bound(L)`. |

The single template underlying all of these:

```cpp
// Adjust the condition on arr[mid] to get the desired boundary.
// lower_bound:    arr[mid] >= target  →  record ans, hi = mid - 1
// upper_bound:    arr[mid] > target   →  record ans, hi = mid - 1
// floor:          arr[mid] <= target  →  record ans, lo = mid + 1
// last occurrence: arr[mid] == target →  record ans, lo = mid + 1

int lo = 0, hi = n - 1, ans = <default>;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (<condition involving arr[mid] and target>) {
        ans = mid;                      // record candidate
        <move toward more candidates>;  // hi = mid-1 (search left) or lo = mid+1 (search right)
    } else {
        <move away from candidates>;
    }
}
return ans;
```

The direction of movement after recording `ans` is the only thing that differs between these variants:

- Want the **leftmost** valid position (lower\_bound, ceil, first occurrence): `hi = mid - 1` after match.
- Want the **rightmost** valid position (floor, last occurrence): `lo = mid + 1` after match.

---

## 15. C++ STL: lower\_bound and upper\_bound

The C++ STL provides `std::lower_bound` and `std::upper_bound` in `<algorithm>`. They operate on sorted random-access containers and run in O(log n).

```cpp
#include <algorithm>
#include <vector>
using namespace std;

vector<int> v = {2, 3, 7, 10, 11, 11, 25};

// Iterator to first element >= 9
auto lb = lower_bound(v.begin(), v.end(), 9);
int lb_index = lb - v.begin();          // 3

// Iterator to first element > 11
auto ub = upper_bound(v.begin(), v.end(), 11);
int ub_index = ub - v.begin();          // 6

// Count occurrences of 11
int count = upper_bound(v.begin(), v.end(), 11)
          - lower_bound(v.begin(), v.end(), 11);   // 2

// Check existence and get index
auto it = lower_bound(v.begin(), v.end(), 9);
bool found = (it != v.end() && *it == 9);   // false (9 is not in v)
```

**Important details:**

- `lower_bound` and `upper_bound` return **iterators**. Subtract `v.begin()` to get a 0-based integer index.
- If the returned iterator equals `v.end()`, the result is `n` — no valid position was found.
- `std::binary_search(v.begin(), v.end(), target)` returns a `bool` (true if present). It does not return an index. For index queries, use `lower_bound` with a verification step.
- **Interview gotcha:** `lower_bound({1,3,5}, 2)` returns an iterator to `3` (index 1), not to `end()`. It returns the insertion point, not "not found."

**Custom comparator** for non-standard orderings:
```cpp
// Descending sorted vector
vector<int> desc = {25, 11, 11, 10, 7, 3, 2};
auto it = lower_bound(desc.begin(), desc.end(), 9, greater<int>());
// Returns iterator to first element <= 9 in a descending array (i.e., value 7)
```

---

## 16. Edge Cases Master Table

| Scenario | Input | Expected | Reason |
|---|---|---|---|
| Target smaller than all elements | `arr=[5,10,15]`, `x=2` | floor=-1, ceil=5, insert=0 | No element <= 2; first element >= 2 is 5 |
| Target larger than all elements | `arr=[5,10,15]`, `x=20` | floor=15, ceil=-1, insert=3 | All elements <= 20; no element >= 20 |
| Target exactly equals element | `arr=[5,10,15]`, `x=10` | floor=10, ceil=10, first=1, last=1 | floor == ceil when element is present |
| All elements equal, target matches | `arr=[7,7,7,7]`, `x=7` | first=0, last=3, count=4 | Entire array is one contiguous block |
| All elements equal, target absent | `arr=[7,7,7,7]`, `x=5` | first=-1, last=-1, count=0 | lb and ub point to same index; ub-lb=0 |
| Single element, match | `arr=[3]`, `x=3` | found=0, floor=0, ceil=3 | Trivially handled; loop runs once |
| Single element, no match (x > elem) | `arr=[3]`, `x=5` | found=-1, floor=3, ceil=-1 | arr[0] < x, so floor=arr[0]; no ceil |
| Single element, no match (x < elem) | `arr=[3]`, `x=1` | found=-1, floor=-1, ceil=3 | arr[0] > x, so no floor; ceil=arr[0] |
| Empty array | `arr=[]`, `x=5` | found=-1, floor=-1, ceil=-1 | Loop never executes; defaults returned |
| Target between two elements | `arr=[1,3]`, `x=2` | floor=1(val=1), ceil=3, insert=1 | 1 < 2 < 3 |
| Duplicates at boundaries | `arr=[1,2,2,2,3]`, `x=2` | first=1, last=3, count=3 | Block spans indices 1 to 3 |

---

## 17. Common Mistakes and Pitfalls

### Mistake 1 — Integer overflow in midpoint calculation

```cpp
// WRONG — overflows when lo and hi are both near INT_MAX
int mid = (lo + hi) / 2;

// CORRECT
int mid = lo + (hi - lo) / 2;
```

### Mistake 2 — Wrong loop condition for exact search

```cpp
// WRONG — misses the single remaining element when lo == hi
while (lo < hi) { ... }

// CORRECT for closed-interval exact search
while (lo <= hi) { ... }

// Note: while (lo < hi) IS correct for the half-open variant — pick one and
// be consistent within a function. Never mix them.
```

### Mistake 3 — Stopping immediately on match instead of recording and continuing

```cpp
// WRONG — finds some occurrence but not necessarily the first
int firstOcc(vector<int>& arr, int target) {
    int lo = 0, hi = arr.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;   // stops at any occurrence
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}

// CORRECT — records candidate and keeps going left
int firstOcc(vector<int>& arr, int target) {
    int lo = 0, hi = arr.size() - 1, ans = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) { ans = mid; hi = mid - 1; }
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return ans;
}
```

### Mistake 4 — Returning lower\_bound result without verifying target is present

```cpp
// WRONG — lb could point to a different element when target is absent
int search(vector<int>& nums, int target) {
    int lb = lowerBound(nums, target);
    return lb;

// CORRECT
int search(vector<int>& nums, int target) {
    int lb = lowerBound(nums, target);
    if (lb < (int)nums.size() && nums[lb] == target) return lb;
    return -1;
}
```

### Mistake 5 — Infinite loop from missing pointer update on match

```cpp
// WRONG — when searching for last occurrence, forgetting lo = mid+1
// causes an infinite loop when lo == mid
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (arr[mid] == target) { ans = mid; /* BUG: lo not updated */ }
    else if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}

// CORRECT
if (arr[mid] == target) { ans = mid; lo = mid + 1; }
```

### Mistake 6 — Calling binary search on unsorted data

`lower_bound`, `upper_bound`, and all variants are only correct on **sorted (non-decreasing)** data. On unsorted data, the comparisons do not give valid information about which half to discard. The result is undefined behavior (in STL) or silently wrong answers (in manual implementations).

### Mistake 7 — Using `upper_bound - 1` for last occurrence without first verifying existence

```cpp
// WRONG — ub - 1 could be a different element if target is absent
int last = upperBound(arr, target) - 1;

// CORRECT — verify existence first
int lb = lowerBound(arr, target);
if (lb == n || arr[lb] != target) return -1;
int last = upperBound(arr, target) - 1;
```

---

## 18. Complexity Analysis

| Problem | Brute Force | Optimal | Space |
|---|---|---|---|
| Binary Search (exact) | O(n) | O(log n) iterative | O(1) |
| Binary Search (exact) | O(n) | O(log n) recursive | O(log n) |
| Floor in Sorted Array | O(n) | O(log n) | O(1) |
| Ceil in Sorted Array | O(n) | O(log n) | O(1) |
| Floor and Ceil Together | O(n) | O(log n) — two passes | O(1) |
| Search Insert Position | O(n) | O(log n) | O(1) |
| First Occurrence | O(n) | O(log n) | O(1) |
| Last Occurrence | O(n) | O(log n) | O(1) |
| Count Occurrences | O(n) | O(log n) | O(1) |

**Why O(log n)?** After each iteration, the search space of size `n` becomes `n/2`. After `k` iterations, size = `n / 2^k`. Setting this to 1 gives `k = log2(n)`. The number of iterations is therefore bounded by `ceil(log2(n))`, which is O(log n).

The base of the logarithm does not matter for big-O: `log2(n) = log10(n) / log10(2)`, and `1/log10(2)` is a constant.

---

## 19. Interview Pattern Recognition

### Signals that binary search applies

| Signal in the problem | What it means |
|---|---|
| "Array is sorted; find a value" | Exact binary search or lower\_bound with verification |
| "Find the first / last occurrence" | Biased binary search (lower\_bound / upper\_bound variant) |
| "Minimum X such that condition(X) holds" | Binary search on the answer; condition must be monotone in X |
| "Maximum X such that condition(X) holds" | Same, but record ans on condition-true and search right |
| "Count elements in a range [L, R]" | `upper_bound(R) - lower_bound(L)` |
| "Smallest / largest satisfying a property" | lower\_bound / upper\_bound on the property |
| "Allocate bandwidth / minimize maximum / maximize minimum" | Binary search on answer space with feasibility check |

### Two-pointer vs binary search

Both work on sorted arrays. Use binary search when finding a single specific boundary or value in a large space. Use two pointers when finding pairs satisfying a condition (two-sum on sorted array) or maintaining a sliding window.

### Extension problems that build directly on these primitives

| Problem | Core primitive used |
|---|---|
| Search in Rotated Sorted Array (LC 33) | Binary search with extra check for which half is sorted |
| Minimum in Rotated Sorted Array (LC 153) | Binary search using rotation invariant |
| Find Peak Element (LC 162) | Binary search on condition `arr[mid] > arr[mid+1]` |
| Integer Square Root (LC 69) | Binary search on `[1, x]` with condition `mid*mid <= x` |
| Koko Eating Bananas (LC 875) | Binary search on eating speed; feasibility check is O(n) |
| Capacity to Ship Packages (LC 1011) | Binary search on capacity; feasibility check is O(n) |
| Aggressive Cows (GFG) | Binary search on minimum distance; feasibility check is O(n) |
| Allocate Books (GFG) | Binary search on maximum pages; feasibility check is O(n) |
| Median of Two Sorted Arrays (LC 4) | Partition binary search on the shorter array |
| Count pairs with sum == target | Fix one element, binary search for complement |
| Count elements in range [L, R] | `upper_bound(R) - lower_bound(L)` |

The pattern for "binary search on the answer" is: define a predicate `feasible(mid)` that is monotone in `mid`, then run binary search to find the transition point. The O(n) feasibility check wrapped in O(log(answer_space)) gives O(n log n) overall.

---

## 20. Quick Revision Cheat Sheet

This section is self-contained for night-before revision.

### Core definitions

- `lower_bound(x)` — first index where `arr[i] >= x`. Returns `n` if none.
- `upper_bound(x)` — first index where `arr[i] > x`. Returns `n` if none.
- `floor(x)` — largest `arr[i] <= x`. Index = `upper_bound(x) - 1` (if `ub > 0`).
- `ceil(x)` — smallest `arr[i] >= x`. Index = `lower_bound(x)` (if `lb < n`).

### Midpoint formula — always use this

```cpp
int mid = lo + (hi - lo) / 2;   // never (lo + hi) / 2 — integer overflow
```

### Loop condition

```
Exact search:     while (lo <= hi)  with lo = 0, hi = n - 1
Boundary search:  same, but track ans and continue instead of returning
```

### Pointer update rules

```
arr[mid] < target        →  lo = mid + 1   (discard left half)
arr[mid] > target        →  hi = mid - 1   (discard right half)
arr[mid] == target (first occ)  →  ans = mid; hi = mid - 1  (search left)
arr[mid] == target (last occ)   →  ans = mid; lo = mid + 1  (search right)
arr[mid] >= target (lower_bound) →  ans = mid; hi = mid - 1  (search left)
arr[mid] > target  (upper_bound) →  ans = mid; hi = mid - 1  (search left)
arr[mid] <= target (floor)       →  ans = mid; lo = mid + 1  (search right)
```

### The unifying table

| Goal | Condition for recording ans | Direction after recording |
|---|---|---|
| lower\_bound (first >= x) | `arr[mid] >= x` | `hi = mid - 1` (go left) |
| upper\_bound (first > x) | `arr[mid] > x` | `hi = mid - 1` (go left) |
| floor (largest <= x) | `arr[mid] <= x` | `lo = mid + 1` (go right) |
| first occurrence | `arr[mid] == target` | `hi = mid - 1` (go left) |
| last occurrence | `arr[mid] == target` | `lo = mid + 1` (go right) |

### Key formulas

```
count of x          =  upper_bound(x) - lower_bound(x)
search insert pos   =  lower_bound(target)
first occurrence    =  lower_bound(target), verify arr[lb] == target
last occurrence     =  upper_bound(target) - 1, verify target exists first
count in [L, R]     =  upper_bound(R) - lower_bound(L)
```

### Complexity

All binary search operations: **O(log n) time, O(1) space (iterative)**.
Recursive binary search: O(log n) time, O(log n) space (call stack).

### STL one-liners

```cpp
auto lb  = lower_bound(v.begin(), v.end(), x);  // iterator to first >= x
auto ub  = upper_bound(v.begin(), v.end(), x);  // iterator to first > x
int  lb_idx = lb - v.begin();
int  ub_idx = ub - v.begin();
int  cnt    = ub - lb;
bool found  = (lb != v.end() && *lb == x);
```

### Default values for ans

```
lower_bound / upper_bound / ceil → ans = n   (target beyond end)
floor / first occurrence / last occurrence → ans = -1  (not found)
search insert position → ans = n  (insert at end)
```

### The three bugs that fail binary search

1. `mid = (lo + hi) / 2` overflows — use `lo + (hi - lo) / 2`.
2. `while (lo < hi)` in exact search — misses the last element; use `lo <= hi`.
3. Returning immediately on match instead of recording `ans` and continuing — gives some occurrence, not first or last.
