

**Topic:** Binary Search on Modified Arrays — Rotated Arrays, Single Element, Peak Finding
**Pattern:** Exploiting Structural Guarantees on Partially Sorted Spaces

**Resources:**
[LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/) |
[LC 81](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) |
[LC 153](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) |
[GFG Rotation Count](https://www.geeksforgeeks.org/problems/rotation4723/1) |
[LC 540](https://leetcode.com/problems/single-element-in-a-sorted-array/) |
[LC 162](https://leetcode.com/problems/find-peak-element/) |
[Striver LC 33](https://youtu.be/5qGrJbHhqFs) |
[Striver LC 81](https://youtu.be/w2G2W8l__pc) |
[Striver LC 153](https://youtu.be/nhEMDKMB44g) |
[Striver Rotation Count](https://youtu.be/jtSiWTPLwd0) |
[Striver LC 540](https://youtu.be/AZOmHuHadxQ) |
[Striver LC 162](https://youtu.be/cXxmbemS6XM)

---

## Table of Contents

1. [Core Mental Model](#1-core-mental-model)
2. [Anatomy of a Rotated Sorted Array](#2-anatomy-of-a-rotated-sorted-array)
3. [Problem 1 — Search in Rotated Sorted Array (LC 33)](#3-problem-1--search-in-rotated-sorted-array-lc-33)
4. [Problem 2 — Search in Rotated Sorted Array II (LC 81)](#4-problem-2--search-in-rotated-sorted-array-ii-lc-81)
5. [Problem 3 — Find Minimum in Rotated Sorted Array (LC 153)](#5-problem-3--find-minimum-in-rotated-sorted-array-lc-153)
6. [Problem 4 — Find Rotation Count (GFG)](#6-problem-4--find-rotation-count-gfg)
7. [Problem 5 — Single Element in a Sorted Array (LC 540)](#7-problem-5--single-element-in-a-sorted-array-lc-540)
8. [Problem 6 — Find Peak Element (LC 162)](#8-problem-6--find-peak-element-lc-162)
9. [Why Binary Search Works on Unsorted Arrays — Formal Argument](#9-why-binary-search-works-on-unsorted-arrays--formal-argument)
10. [Why LC 81 Degrades to O(n) — Information-Theoretic Lower Bound](#10-why-lc-81-degrades-to-on--information-theoretic-lower-bound)
11. [Edge Cases Master Table](#11-edge-cases-master-table)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Complexity Cheat Sheet](#13-complexity-cheat-sheet)
14. [Interview Reference — Patterns and Applications](#14-interview-reference--patterns-and-applications)
15. [Quick Revision Cheat Sheet](#15-quick-revision-cheat-sheet)

---

## 1. Core Mental Model

The previous set of notes established that binary search works whenever there is a **monotone predicate** — a yes/no question that transitions from false to true exactly once across the search space.

This set extends that idea to arrays that are no longer globally sorted. The prerequisite for binary search is not "the array is sorted" — it is "a comparison at `mid` lets us determine in O(1) which half to discard, and the discarded half is proven to contain no valid answer."

Each problem in this set supplies a different structural guarantee that satisfies this requirement:

| Problem | Structural Guarantee | Predicate |
|---|---|---|
| LC 33 / LC 81 — Search Rotated | One of the two halves produced by any `mid` is always sorted | Is target in the sorted half? |
| LC 153 — Find Minimum | The minimum is at the "break point" between the two sorted halves | Is `nums[mid]` above `nums[hi]`? (Did the drop happen in the right half?) |
| GFG Rotation Count | Same as LC 153 | Same; return the index rather than the value |
| LC 540 — Single Element | Before the single element, pairs occupy (even, odd) slots; after, pairs occupy (odd, even) slots | Is the pair at `even_mid` intact? |
| LC 162 — Find Peak | The virtual boundary conditions `nums[-1] = nums[n] = -inf` guarantee a peak always exists in the direction of ascent | Is `nums[mid] < nums[mid+1]`? (Ascending slope?) |

The unifying strategy: at each step, identify what one comparison at `mid` tells you about the structure of each half, then discard the half that provably cannot contain the answer.

---

## 2. Anatomy of a Rotated Sorted Array

A sorted array `[1, 2, 3, 4, 5, 6, 7]` rotated at pivot index `k = 4` becomes `[5, 6, 7, 1, 2, 3, 4]`.

```
Original:   1   2   3   4   5   6   7
Rotated:    5   6   7   1   2   3   4
            ^                       ^
            lo                      hi
            |--- left sorted ---|--- right sorted ---|
```

Key structural facts:

- The array consists of exactly two sorted subarrays joined at a break point.
- The minimum element is at the break point — the first element of the right sorted half.
- When the array is actually rotated (not a 0-rotation): `arr[lo] > arr[hi]` always.
- For any midpoint `mid`: if `arr[mid] >= arr[lo]`, then `mid` is in the left sorted half. Otherwise, `mid` is in the right sorted half.
- **Rotation count** equals the index of the minimum element.

One half produced by any `mid` split is always sorted. This is the invariant that makes binary search possible on rotated arrays despite the global sortedness being broken.

---

## 3. Problem 1 — Search in Rotated Sorted Array (LC 33)

**Problem:** An integer array sorted in ascending order with **distinct** values is possibly rotated at an unknown pivot. Given the array and a `target`, return the index of `target` or `-1`.
**Link:** [LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/)
**Constraints:** `1 <= n <= 5000`, `-10^4 <= nums[i] <= 10^4`, all values unique. O(log n) required.

### Approach 1 — Brute Force Linear Scan

```cpp
int search(vector<int>& nums, int target) {
    for (int i = 0; i < (int)nums.size(); i++)
        if (nums[i] == target) return i;
    return -1;
}
```

Time: O(n). Space: O(1). Ignores the sorted structure entirely.

### Approach 2 — Find Pivot First, Then Binary Search

Find the index of the minimum (the pivot), which splits the array into two sorted subarrays. Determine which subarray `target` falls in, then run a standard binary search on that half.

```cpp
int findMinIndex(vector<int>& nums) {
    int lo = 0, hi = nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

int search(vector<int>& nums, int target) {
    int n = nums.size();
    int pivot = findMinIndex(nums);
    int lo, hi;
    if (target >= nums[pivot] && target <= nums[n - 1])
        lo = pivot, hi = n - 1;
    else
        lo = 0, hi = pivot - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

Time: O(log n) — two independent binary searches. Space: O(1). Correct but uses two passes.

### Approach 3 — Single-Pass Binary Search (Optimal)

**Intuition:** At every step, one of the two halves `[lo, mid]` or `[mid, hi]` is guaranteed to be sorted. We identify which by comparing `nums[lo]` with `nums[mid]`:

- If `nums[lo] <= nums[mid]`: the left half `[lo, mid]` is sorted.
- Otherwise: the right half `[mid, hi]` is sorted.

Once we know which half is sorted, checking whether `target` falls in that range is a single O(1) comparison. If it does, search there. If it does not, the target must be in the other half. One pass, O(log n).

```cpp
int search(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] == target) return mid;

        if (nums[lo] <= nums[mid]) {
            // Left half [lo, mid] is sorted
            if (nums[lo] <= target && target < nums[mid]) {
                hi = mid - 1;    // target in sorted left half
            } else {
                lo = mid + 1;    // target must be in right half
            }
        } else {
            // Right half [mid, hi] is sorted
            if (nums[mid] < target && target <= nums[hi]) {
                lo = mid + 1;    // target in sorted right half
            } else {
                hi = mid - 1;    // target must be in left half
            }
        }
    }
    return -1;
}
```

**Why `nums[lo] <= nums[mid]` uses `<=` and not `<`:** When `lo == mid` (two-element subarray), `nums[lo] == nums[mid]`. Using strict `<` would misclassify this as "right sorted" and the inner range check `nums[lo] <= target < nums[mid]` would be `x <= t < x` — always false — incorrectly forcing us to search the right half even when the left has one element. With `<=`, the left-sorted branch is entered and the inner check evaluates correctly.

**Why the range check for the left half is `target < nums[mid]` and not `target <= nums[mid]`:** The case `nums[mid] == target` is already handled at the top of the loop. So the left range is `[nums[lo], nums[mid])` — `nums[mid]` is excluded.

**Dry run — target present:** `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 0`

```
Indices:   0  1  2  3  4  5  6
Values:    4  5  6  7  0  1  2

Iter 1: lo=0, hi=6, mid=3, nums[3]=7 != 0
  nums[0]=4 <= nums[3]=7 → left half [0,3] sorted
  Is 0 in [4, 7)? → NO → lo = 4

Iter 2: lo=4, hi=6, mid=5, nums[5]=1 != 0
  nums[4]=0 <= nums[5]=1 → left half [4,5] sorted
  Is 0 in [0, 1)? → YES → hi = 4

Iter 3: lo=4, hi=4, mid=4, nums[4]=0 == 0 → return 4
```

**Dry run — target absent:** `nums = [4, 5, 6, 7, 0, 1, 2]`, `target = 3`

```
Iter 1: lo=0, hi=6, mid=3, nums[3]=7 != 3
  left half sorted: Is 3 in [4, 7)? → NO → lo = 4

Iter 2: lo=4, hi=6, mid=5, nums[5]=1 != 3
  nums[4]=0 <= nums[5]=1 → left half sorted
  Is 3 in [0, 1)? → NO → lo = 6

Iter 3: lo=6, hi=6, mid=6, nums[6]=2 != 3
  nums[6]=2 <= nums[6]=2 → left half sorted (single element)
  Is 3 in [2, 2)? → NO → lo = 7

lo=7 > hi=6 → return -1
```

Time: O(log n). Space: O(1).

---

## 4. Problem 2 — Search in Rotated Sorted Array II (LC 81)

**Problem:** Same as LC 33 but `nums` may contain **duplicates**. Return `true` if `target` exists, `false` otherwise.
**Link:** [LC 81](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/)
**Constraints:** `1 <= n <= 5000`, `-10^4 <= nums[i] <= 10^4`.

### Why Duplicates Break the LC 33 Logic

In LC 33, `nums[lo] <= nums[mid]` reliably identifies the sorted half because all elements are distinct. With duplicates, the critical failure case is `nums[lo] == nums[mid] == nums[hi]`. Consider:

```
nums = [1, 0, 1, 1, 1],  lo=0, hi=4, mid=2
nums[lo]=1, nums[mid]=1, nums[hi]=1
→ "left half sorted" branch entered
→ Check if 0 in [1, 1) → NO → lo = mid+1 = 3
→ Now search [3,4] = [1,1], target=0 never found
```

When all three boundary values are equal, we cannot determine which half is sorted — both halves appear structurally identical. The only safe action is to shrink both boundaries inward by 1: `lo++; hi--`.

### Approach — Modified Binary Search with Duplicate Handling

```cpp
bool search(vector<int>& nums, int target) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        if (nums[mid] == target) return true;

        // Ambiguous case: cannot determine sorted half
        if (nums[lo] == nums[mid] && nums[mid] == nums[hi]) {
            lo++;
            hi--;
            continue;
        }

        if (nums[lo] <= nums[mid]) {
            // Left half [lo, mid] is sorted
            if (nums[lo] <= target && target < nums[mid]) {
                hi = mid - 1;
            } else {
                lo = mid + 1;
            }
        } else {
            // Right half [mid, hi] is sorted
            if (nums[mid] < target && target <= nums[hi]) {
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
    }
    return false;
}
```

**Dry run — with duplicates, target present:** `nums = [1, 0, 1, 1, 1]`, `target = 0`

```
Indices:  0  1  2  3  4
Values:   1  0  1  1  1

Iter 1: lo=0, hi=4, mid=2, nums[2]=1 != 0
  nums[0]=1 == nums[2]=1 == nums[4]=1 → ambiguous → lo=1, hi=3

Iter 2: lo=1, hi=3, mid=2, nums[2]=1 != 0
  nums[1]=0, nums[2]=1, nums[3]=1
  nums[lo]=0 <= nums[mid]=1 → left half [1,2] sorted
  Is 0 in [0, 1)? → YES → hi = 1

Iter 3: lo=1, hi=1, mid=1, nums[1]=0 == 0 → return true
```

**Dry run — standard rotation, no ambiguity:** `nums = [2, 5, 6, 0, 0, 1, 2]`, `target = 0`

```
Iter 1: lo=0, hi=6, mid=3, nums[3]=0 == 0 → return true
```

Time: O(log n) average, **O(n) worst case** when all elements are equal. Space: O(1). (The O(n) worst case is a fundamental lower bound, not a flaw — see Section 10.)

---

## 5. Problem 3 — Find Minimum in Rotated Sorted Array (LC 153)

**Problem:** A sorted array of **unique** elements rotated 1 to n times. Find the minimum element. O(log n) required.
**Link:** [LC 153](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/)
**Constraints:** `1 <= n <= 5000`, `-5000 <= nums[i] <= 5000`, all elements unique.

### Approach 1 — Brute Force

```cpp
int findMin(vector<int>& nums) {
    return *min_element(nums.begin(), nums.end());
}
```

Time: O(n). Space: O(1).

### Approach 2 — Binary Search (Optimal)

**Intuition:** The minimum is at the break point — the only position where `arr[i] < arr[i-1]`. Binary search finds it by comparing `nums[mid]` with `nums[hi]`:

- If `nums[mid] > nums[hi]`: the break point is in the right half `[mid+1, hi]`. The entire left half `[lo, mid]` is sorted and lies above the minimum.
- If `nums[mid] <= nums[hi]`: the segment `[mid, hi]` is fully sorted ascending, so `nums[mid]` is its smallest element. The minimum of the entire array is either `nums[mid]` itself or somewhere in `[lo, mid-1]`. Use `hi = mid` (not `mid-1`) to keep `mid` in the search space.

The loop uses `lo < hi` (not `lo <= hi`) because we use `hi = mid` on one branch — if `lo == hi == mid`, that assignment would make no progress. The exit condition `lo == hi` identifies the single remaining candidate.

```cpp
int findMin(vector<int>& nums) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) {
            lo = mid + 1;    // minimum is strictly to the right of mid
        } else {
            hi = mid;        // minimum is at mid or to the left; keep mid
        }
    }
    return nums[lo];         // lo == hi at termination
}
```

**Optional early-exit optimization:** If `nums[lo] <= nums[hi]`, the current subarray is already sorted and `nums[lo]` is the minimum. This avoids unnecessary iterations on sorted subarrays.

```cpp
int findMin(vector<int>& nums) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        if (nums[lo] <= nums[hi]) return nums[lo];   // already sorted
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > nums[hi]) lo = mid + 1;
        else hi = mid;
    }
    return nums[lo];
}
```

**Why compare `mid` with `hi` and not with `lo`:** Comparing with `nums[lo]` is more fragile because `lo` can move past the original starting point of the left sorted half as the search progresses, making the comparison lose meaning. Comparing with `nums[hi]` is stable: `nums[hi]` is always the last element of the current search window, and `nums[mid] > nums[hi]` directly tells us the drop happened between `mid` and `hi`.

**Dry run:** `nums = [4, 5, 6, 7, 0, 1, 2]`

```
Indices:  0  1  2  3  4  5  6
Values:   4  5  6  7  0  1  2

Iter 1: lo=0, hi=6, mid=3, nums[3]=7 > nums[6]=2 → lo = 4
Iter 2: lo=4, hi=6, mid=5, nums[5]=1 <= nums[6]=2 → hi = 5
Iter 3: lo=4, hi=5, mid=4, nums[4]=0 <= nums[5]=1 → hi = 4
lo=4 == hi=4 → return nums[4] = 0
```

**Dry run:** `nums = [3, 4, 5, 1, 2]`

```
Iter 1: lo=0, hi=4, mid=2, nums[2]=5 > nums[4]=2 → lo = 3
Iter 2: lo=3, hi=4, mid=3, nums[3]=1 <= nums[4]=2 → hi = 3
lo=3 == hi=3 → return nums[3] = 1
```

**Dry run — not rotated:** `nums = [11, 13, 15, 17]`

```
Iter 1: lo=0, hi=3, nums[0]=11 <= nums[3]=17 → return 11  (early exit)
```

Time: O(log n). Space: O(1).

---

## 6. Problem 4 — Find Rotation Count (GFG)

**Problem:** A sorted array of distinct integers has been rotated clockwise `k` times. Find `k`.
**Link:** [GFG Rotation Count](https://www.geeksforgeeks.org/problems/rotation4723/1)
**Constraints:** `1 <= n <= 10^5`, distinct elements, `0 <= k < n`.

**Key insight:** The rotation count equals the **index of the minimum element**. After one rotation, the smallest element moves to index 1. After two rotations, to index 2. After `k` rotations, to index `k`. This is the same computation as LC 153, except we return the index instead of the value.

```cpp
int findKRotation(vector<int>& arr) {
    int lo = 0, hi = (int)arr.size() - 1;
    while (lo < hi) {
        if (arr[lo] <= arr[hi]) return lo;   // already sorted; min is at lo
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] > arr[hi]) {
            lo = mid + 1;    // minimum in right half
        } else {
            hi = mid;        // minimum at mid or in left half
        }
    }
    return lo;   // index of minimum = rotation count
}
```

**Dry run:** `arr = [15, 18, 2, 3, 6, 12]`

```
Indices:   0   1  2  3  4   5
Values:   15  18  2  3  6  12

Iter 1: lo=0, hi=5, arr[0]=15 > arr[5]=12 → not sorted
  mid=2, arr[2]=2 <= arr[5]=12 → hi = 2

Iter 2: lo=0, hi=2, arr[0]=15 > arr[2]=2 → not sorted
  mid=1, arr[1]=18 > arr[2]=2 → lo = 2

lo=2 == hi=2 → return 2
```

Rotation count = 2. Original was `[2, 3, 6, 12, 15, 18]`, rotated twice. Correct.

**Dry run — not rotated:** `arr = [7, 9, 11, 12, 15]`

```
Iter 1: arr[0]=7 <= arr[4]=15 → return 0
```

Time: O(log n). Space: O(1).

**Relationship to LC 153:** `findKRotation` and `findMin` are the same algorithm. `findMin` returns `arr[pivot]`; `findKRotation` returns `pivot`. Both compute the break point of the rotation.

---

## 7. Problem 5 — Single Element in a Sorted Array (LC 540)

**Problem:** A sorted array where every element appears exactly twice except for one element that appears exactly once. Find the single element. O(log n) time, O(1) space required.
**Link:** [LC 540](https://leetcode.com/problems/single-element-in-a-sorted-array/)
**Constraints:** `1 <= n <= 10^5`, `0 <= nums[i] <= 10^5`, `n` is always odd.

### Approach 1 — XOR (O(n))

XOR-ing a number with itself yields 0; XOR-ing with 0 yields the number. XOR-ing all elements cancels every pair, leaving only the single element.

```cpp
int singleNonDuplicate(vector<int>& nums) {
    int res = 0;
    for (int x : nums) res ^= x;
    return res;
}
```

Time: O(n). Space: O(1). Does not exploit sorted order; fails the O(log n) requirement.

### Approach 2 — Binary Search on Pair-Index Parity (Optimal)

**The key structural observation:** In a sorted array of all pairs with one single intruder, the pairing alignment shifts at the position of the single element.

- **Before the single element:** pairs occupy (even, odd) index slots. Pair starting at even index `k`: `arr[k] == arr[k+1]`.
- **After the single element:** the pairing is shifted by one. What was at an odd index is now first in its pair. Pair starting at odd index `k`: `arr[k] == arr[k+1]`.

To binary-search the boundary between "before" and "after": normalize `mid` to even by subtracting 1 if it is odd (`mid -= (mid % 2)`). This gives us the start of the pair that `mid` belongs to. Then:

- If `nums[mid] == nums[mid+1]`: the pair at `[mid, mid+1]` is intact — we are in the "before" region. The single element is strictly to the right. `lo = mid + 2`.
- If `nums[mid] != nums[mid+1]`: the pairing has broken — we are at the boundary or in the "after" region. The single element is at `mid` or to the left. `hi = mid`.

The loop uses `lo < hi`; exit when `lo == hi`.

```cpp
int singleNonDuplicate(vector<int>& nums) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (mid % 2 == 1) mid--;     // snap to start of pair (even index)
        if (nums[mid] == nums[mid + 1]) {
            lo = mid + 2;    // pair intact; single element to the right
        } else {
            hi = mid;        // pair broken; single element at mid or left
        }
    }
    return nums[lo];
}
```

**Dry run:** `nums = [1, 1, 2, 3, 3, 4, 4, 8, 8]`

```
Indices:  0  1  2  3  4  5  6  7  8
Values:   1  1  2  3  3  4  4  8  8

Iter 1: lo=0, hi=8, mid=4, even → no snap
  nums[4]=3, nums[5]=4 → not equal → hi = 4

Iter 2: lo=0, hi=4, mid=2, even → no snap
  nums[2]=2, nums[3]=3 → not equal → hi = 2

Iter 3: lo=0, hi=2, mid=1, odd → snap: mid=0
  nums[0]=1, nums[1]=1 → equal → pair intact → lo = 2

lo=2 == hi=2 → return nums[2] = 2
```

**Dry run:** `nums = [3, 3, 7, 7, 10, 11, 11]`

```
Indices:  0  1  2  3   4   5   6
Values:   3  3  7  7  10  11  11

Iter 1: lo=0, hi=6, mid=3, odd → snap: mid=2
  nums[2]=7, nums[3]=7 → equal → pair intact → lo = 4

Iter 2: lo=4, hi=6, mid=5, odd → snap: mid=4
  nums[4]=10, nums[5]=11 → not equal → hi = 4

lo=4 == hi=4 → return nums[4] = 10
```

Time: O(log n). Space: O(1).

**Alternative without snapping** — direct parity branching (equivalent, slightly more verbose):

```cpp
int singleNonDuplicate(vector<int>& nums) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (mid % 2 == 0) {
            // mid is even: pair partner should be mid+1
            if (nums[mid] == nums[mid + 1]) lo = mid + 2;
            else hi = mid;
        } else {
            // mid is odd: pair partner should be mid-1
            if (nums[mid] == nums[mid - 1]) lo = mid + 1;
            else hi = mid - 1;
        }
    }
    return nums[lo];
}
```

Both implementations are correct. The snapping version is easier to memorize for interviews.

**Note on `hi` initialization:** The snapping version initializes `hi = nums.size() - 1` (the last index). The answer is always at an even index (since `n` is odd), and the snapping guarantees we never access `nums[mid+1]` out of bounds: `mid` is always even after snapping, and `mid <= hi - 1` since `lo < hi` ensures the loop hasn't converged.

---

## 8. Problem 6 — Find Peak Element (LC 162)

**Problem:** A peak element is one that is strictly greater than its neighbors. Given `nums` where `nums[-1] = nums[n] = -inf`, find any peak element index. O(log n) required. Any valid peak is acceptable.
**Link:** [LC 162](https://leetcode.com/problems/find-peak-element/)
**Constraints:** `1 <= n <= 500`, `-2^31 <= nums[i] <= 2^31 - 1`, `nums[i] != nums[i+1]` for all valid `i`.

### Approach 1 — Linear Scan

```cpp
int findPeakElement(vector<int>& nums) {
    int n = nums.size();
    if (n == 1) return 0;
    if (nums[0] > nums[1]) return 0;
    if (nums[n-1] > nums[n-2]) return n - 1;
    for (int i = 1; i < n - 1; i++)
        if (nums[i] > nums[i-1] && nums[i] > nums[i+1])
            return i;
    return -1;
}
```

Time: O(n). Space: O(1).

### Approach 2 — Binary Search on Slope Direction (Optimal)

**Intuition and proof of correctness:** The virtual boundary conditions `nums[-1] = nums[n] = -inf` mean the array always "starts and ends at negative infinity." Wherever we are in the array, if we follow the ascending direction, we are guaranteed to eventually reach a peak before hitting the boundary.

The monotone predicate: "is `mid` on an ascending slope?" (i.e., `nums[mid] < nums[mid+1]`)

- If `nums[mid] < nums[mid+1]`: `mid` is on an ascending slope. A peak exists somewhere in `[mid+1, hi]` — the sequence must eventually stop rising before reaching `nums[n] = -inf`, and that stopping point is a peak. Discard `[lo, mid]`.
- If `nums[mid] > nums[mid+1]`: `mid` is on a descending slope. A peak exists somewhere in `[lo, mid]` — the sequence must have risen to reach this point from `nums[-1] = -inf`, and the top of that rise is a peak. Keep `mid` with `hi = mid` (do not use `hi = mid - 1`; `mid` itself could be the peak).

The array is not sorted, but the binary search is still valid because a peak is guaranteed to exist in the retained half at every step.

```cpp
int findPeakElement(vector<int>& nums) {
    int lo = 0, hi = (int)nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < nums[mid + 1]) {
            lo = mid + 1;    // ascending slope: peak is to the right
        } else {
            hi = mid;        // descending slope: peak is at mid or to the left
        }
    }
    return lo;   // lo == hi is the peak index
}
```

**Why `hi = mid` and not `hi = mid - 1`:** When `nums[mid] > nums[mid+1]`, `mid` is a candidate peak. Discarding it with `mid - 1` would skip a valid answer.

**Why `lo < hi` and not `lo <= hi`:** The `hi = mid` update does not reduce `hi` by at least 1 every iteration (it sets `hi` to `mid`, which equals `lo` when `lo == hi`). With `lo <= hi`, this would loop forever when `lo == hi`. With `lo < hi`, the loop terminates when `lo == hi`, and that single remaining element is the peak.

**Dry run:** `nums = [1, 2, 3, 1]`

```
Indices:  0  1  2  3
Values:   1  2  3  1

Iter 1: lo=0, hi=3, mid=1, nums[1]=2 < nums[2]=3 → lo = 2
Iter 2: lo=2, hi=3, mid=2, nums[2]=3 > nums[3]=1 → hi = 2
lo=2 == hi=2 → return 2  (nums[2]=3 is peak)
```

**Dry run:** `nums = [1, 2, 1, 3, 5, 6, 4]`

```
Iter 1: lo=0, hi=6, mid=3, nums[3]=3 < nums[4]=5 → lo = 4
Iter 2: lo=4, hi=6, mid=5, nums[5]=6 > nums[6]=4 → hi = 5
Iter 3: lo=4, hi=5, mid=4, nums[4]=5 < nums[5]=6 → lo = 5
lo=5 == hi=5 → return 5  (nums[5]=6 is peak)
```

**Dry run — strictly decreasing:** `nums = [5, 4, 3, 2, 1]`

```
Iter 1: lo=0, hi=4, mid=2, nums[2]=3 > nums[3]=2 → hi = 2
Iter 2: lo=0, hi=2, mid=1, nums[1]=4 > nums[2]=3 → hi = 1
Iter 3: lo=0, hi=1, mid=0, nums[0]=5 > nums[1]=4 → hi = 0
lo=0 == hi=0 → return 0  (nums[0]=5 > nums[-1]=-inf, so it is a peak)
```

Time: O(log n). Space: O(1).

---

## 9. Why Binary Search Works on Unsorted Arrays — Formal Argument

The prerequisite for binary search is not "the array is sorted." It is: a comparison at `mid` lets us determine in O(1) which half to discard, the discarded half is proven to contain no valid answer, and the retained half is proven to contain at least one valid answer. All three conditions must hold.

**LC 33 / LC 81:** One of the two halves produced by any `mid` split is always sorted (structural property of a singly-rotated array). Knowing which half is sorted lets us check in O(1) whether `target` is in the sorted range. If yes, search there (it may exist). If no, it cannot be there (sorted range fully visible), so search the other half.

**LC 153 / GFG Rotation Count:** The predicate `nums[mid] > nums[hi]` directly identifies on which side of the break point `mid` falls. If `mid` is above `nums[hi]`, the entire left half is above the minimum — discard it. The minimum is guaranteed to be in the right half.

**LC 540:** The pair-index parity is a monotone binary property. Before the single element, `nums[even_mid] == nums[even_mid+1]` holds (pair intact). After the single element, it does not. The single element is at the boundary of this transition — exactly the kind of transition point binary search is designed to find.

**LC 162:** The virtual boundary `nums[-1] = nums[n] = -inf` guarantees a peak exists in every non-trivial subarray. Following the ascending slope is guaranteed to lead toward a peak before hitting the boundary. This is sufficient: we never discard a half that might contain all peaks, because the retained half is guaranteed to contain at least one.

---

## 10. Why LC 81 Degrades to O(n) — Information-Theoretic Lower Bound

When all elements are equal (e.g., `[1,1,1,1,2,1,1,1]`), the condition `nums[lo] == nums[mid] == nums[hi]` holds at every step. We can only shrink by 1 from each end, so the algorithm takes O(n) iterations.

This is not a deficiency in the algorithm — it is a **fundamental lower bound**. Consider any input where all elements are 1 except one 2 at unknown position `k`. The only operation that distinguishes `k = i` from `k = j` is examining positions `i` and `j` directly. With `n` possible positions for 2, any algorithm (including any binary search variant) needs up to O(n) comparisons in the worst case. No algorithm can achieve O(log n) on this family of inputs without additional assumptions.

This is why LC 81 is correctly specified as O(log n) average, O(n) worst case.

---

## 11. Edge Cases Master Table

| Scenario | Problem | Input | Expected | Reason |
|---|---|---|---|---|
| Not rotated (0 rotation) | LC 33 | `[1,2,3,4,5]`, `t=3` | `2` | `nums[lo]<=nums[mid]` always; standard binary search |
| Single element, match | LC 33 | `[5]`, `t=5` | `0` | `lo=hi=mid=0`, direct match |
| Target at break point (minimum) | LC 33 | `[4,5,1,2,3]`, `t=1` | `2` | Right half sorted; target in `(nums[mid], nums[hi]]` |
| Target is `nums[lo]` | LC 33 | `[3,1,2]`, `t=3` | `0` | Left-sorted branch, `nums[lo]==target` handled by initial check |
| All duplicates, target present | LC 81 | `[1,1,1,1,2,1,1]`, `t=2` | `true` | Degrades to O(n) linear shrink |
| All duplicates, target absent | LC 81 | `[1,1,1,1]`, `t=2` | `false` | Degrades to O(n) linear shrink |
| Not rotated | LC 153 | `[1,2,3,4,5]` | `1` | Early exit: `arr[lo]<=arr[hi]` |
| Two elements | LC 153 | `[2,1]` | `1` | One binary search step |
| Rotated `n` times | LC 153 | `[3,4,5,1,2]` rotated 5x | `3` | `n`-rotation = 0-rotation; `arr[lo]<=arr[hi]` |
| Not rotated | GFG Count | `[7,9,11,12,15]` | `0` | Early exit |
| Single element | LC 540 | `[5]` | `5` | `lo==hi` immediately |
| Single element at start | LC 540 | `[2,3,3]` | `2` | Snap to even: `mid=0`; `nums[0]!=nums[1]`; `hi=0` |
| Single element at end | LC 540 | `[1,1,5]` | `5` | Snap to even: pair at 0 intact; `lo=2` |
| Strictly increasing | LC 162 | `[1,2,3,4,5]` | `4` | Keeps ascending until `lo==hi==n-1` |
| Strictly decreasing | LC 162 | `[5,4,3,2,1]` | `0` | First comparison: `nums[0]>nums[1]`; `hi` converges to `0` |
| Two elements | LC 162 | `[1,2]` | `1` | `nums[0]<nums[1]`; `lo=1` |
| Multiple peaks | LC 162 | `[1,3,2,4,1]` | `1` or `3` | Any peak acceptable; algorithm finds one deterministically |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1 — Using `nums[lo] < nums[mid]` (strict) instead of `nums[lo] <= nums[mid]`

```cpp
// WRONG — misclassifies when lo == mid (two-element subarray)
if (nums[lo] < nums[mid]) { /* left sorted */ }

// CORRECT
if (nums[lo] <= nums[mid]) { /* left sorted; handles lo==mid case */ }
```

When `lo == mid`, `nums[lo] == nums[mid]`. Strict `<` falls through to the right-sorted branch; the inner range check then gives a wrong result.

### Mistake 2 — Using `target <= nums[mid-1]` instead of `target < nums[mid]` for the sorted-half range

```cpp
// WRONG — off-by-one boundary
if (nums[lo] <= target && target <= nums[mid - 1]) hi = mid - 1;

// CORRECT — range is [nums[lo], nums[mid]); mid already checked at loop top
if (nums[lo] <= target && target < nums[mid]) hi = mid - 1;
```

### Mistake 3 — Using `hi = mid - 1` in `findMin` / `findPeak`

```cpp
// WRONG in findMin — excludes mid, which could be the minimum
if (nums[mid] <= nums[hi]) hi = mid - 1;

// CORRECT
if (nums[mid] <= nums[hi]) hi = mid;    // keep mid in search space
```

```cpp
// WRONG in findPeak — excludes mid, which could be the peak
if (nums[mid] > nums[mid+1]) hi = mid - 1;

// CORRECT
if (nums[mid] > nums[mid+1]) hi = mid;
```

### Mistake 4 — Using `lo <= hi` loop condition in `findMin` / `findPeak`

```cpp
// WRONG — infinite loop when lo==hi==mid, because hi=mid makes no progress
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] > nums[hi]) lo = mid + 1;
    else hi = mid;   // if lo==hi, hi=mid=lo → loop does not terminate
}

// CORRECT — use lo < hi; loop ends when lo==hi; that element is the answer
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] > nums[hi]) lo = mid + 1;
    else hi = mid;
}
```

The same applies to LC 540 and LC 162.

### Mistake 5 — Missing the ambiguous duplicate case in LC 81

```cpp
// WRONG — applies LC 33 logic directly without handling the ambiguous case
if (nums[lo] <= nums[mid]) { ... }   // no guard for nums[lo]==nums[mid]==nums[hi]

// CORRECT — add the ambiguity guard first
if (nums[lo] == nums[mid] && nums[mid] == nums[hi]) {
    lo++; hi--;
    continue;
}
if (nums[lo] <= nums[mid]) { ... }
```

Without this guard, when all three boundary values are equal, the algorithm may incorrectly conclude the left half is sorted and eliminate the wrong portion.

### Mistake 6 — Forgetting to snap `mid` to even in LC 540

```cpp
// WRONG — if mid is odd, mid+1 could point to the wrong pair partner
int mid = lo + (hi - lo) / 2;
if (nums[mid] == nums[mid + 1]) lo = mid + 2;   // broken when mid is odd

// CORRECT — snap first
int mid = lo + (hi - lo) / 2;
if (mid % 2 == 1) mid--;
if (nums[mid] == nums[mid + 1]) lo = mid + 2;
else hi = mid;
```

### Mistake 7 — Assuming the rotation pivot is at a fixed location

The pivot can be at any index from 0 (no rotation) to n-1 (rotated n-1 times). Never hardcode an assumption about pivot position.

---

## 13. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| LC 33 Search Rotated (distinct) | Linear scan | O(n) | O(1) |
| LC 33 Search Rotated (distinct) | Find pivot + binary search | O(log n) | O(1) |
| LC 33 Search Rotated (distinct) | Single-pass binary search | O(log n) | O(1) |
| LC 81 Search Rotated (duplicates) | Modified binary search | O(log n) avg, O(n) worst | O(1) |
| LC 153 Find Minimum | Linear scan | O(n) | O(1) |
| LC 153 Find Minimum | Binary search (compare with hi) | O(log n) | O(1) |
| GFG Rotation Count | Linear scan (find min index) | O(n) | O(1) |
| GFG Rotation Count | Binary search (same as LC 153) | O(log n) | O(1) |
| LC 540 Single Element | XOR all elements | O(n) | O(1) |
| LC 540 Single Element | Binary search on parity | O(log n) | O(1) |
| LC 162 Find Peak | Linear scan | O(n) | O(1) |
| LC 162 Find Peak | Binary search on slope | O(log n) | O(1) |

---

## 14. Interview Reference — Patterns and Applications

### Decision table

| Question asked | Strategy |
|---|---|
| Find target in rotated array (distinct) | Single-pass binary search: identify sorted half; check if target in range |
| Find target in rotated array (duplicates) | Same + ambiguity guard: `if nums[lo]==nums[mid]==nums[hi]: lo++; hi--` |
| Find minimum in rotated array | Compare `nums[mid]` with `nums[hi]`; move to the side that contains the drop |
| Find how many times rotated | Identical to find-minimum; return the index instead of the value |
| Find any peak element | Follow ascending slope; a peak is guaranteed to exist in that direction |
| Find single non-duplicate element | Normalize `mid` to even; check pair integrity; binary search the parity boundary |

### Connection map between problems

```
LC 153 (find min value)  ───────────────── same algorithm, different return
GFG rotation count (min index) ──────────┘

LC 33 (distinct elements) ────────────── add target check to sorted-half logic
LC 81 (with duplicates) ─────────────── add ambiguity guard to LC 33

LC 540 (single element) ── parity-based predicate; no rotation involved
LC 162 (peak element)   ── slope-based predicate; any-peak contract; boundary trick
```

### Follow-up problems

- **LC 154 — Find Minimum in Rotated Sorted Array II (with duplicates):** Extends LC 153 the same way LC 81 extends LC 33. When `nums[mid] == nums[hi]`, cannot determine which side the minimum is on → `hi--`. Worst case O(n).
- **LC 852 — Peak Index in a Mountain Array:** Simplified LC 162 on a globally unimodal (bitonic) array. Same slope-direction binary search.
- **LC 1095 — Find in Mountain Array:** Find the peak (LC 162), then binary search the ascending and descending halves separately.
- **LC 4 — Median of Two Sorted Arrays:** Binary search on a partition position, not element values. The structural reasoning ("discard the half that cannot satisfy the partition condition") is the same framework.
- **Bitonic array search:** Find the peak (LC 162), then binary search both halves. The peak divides the array into two sorted sequences.

### Loop condition summary for this problem set

```
lo <= hi  (with early return on match):  LC 33, LC 81
lo < hi   (with return nums[lo] after):  LC 153, GFG Count, LC 540, LC 162
```

The `lo < hi` variant is used whenever `hi = mid` appears (keeping `mid` in the search space), because `lo <= hi` with `hi = mid` can loop forever when `lo == hi`.

---

## 15. Quick Revision Cheat Sheet

This section is fully self-contained for night-before revision.

### LC 33 — Search Rotated (distinct)

```
Loop: lo <= hi, return mid on match.
nums[lo] <= nums[mid] → left half [lo,mid] sorted.
  Target in [nums[lo], nums[mid])? → hi = mid-1. Else → lo = mid+1.
Otherwise → right half [mid,hi] sorted.
  Target in (nums[mid], nums[hi]]? → lo = mid+1. Else → hi = mid-1.
```

### LC 81 — Search Rotated (duplicates)

```
Same as LC 33, but first check every iteration:
  if nums[lo] == nums[mid] == nums[hi]: lo++; hi--; continue.
Worst case O(n) when all elements equal.
```

### LC 153 — Find Minimum

```
Loop: lo < hi (not lo <= hi).
mid = lo + (hi-lo)/2.
nums[mid] > nums[hi] → lo = mid+1    (min in right half)
else              → hi = mid         (min at mid or left; keep mid)
Return nums[lo].
Optional: if nums[lo] <= nums[hi], return nums[lo] immediately.
```

### GFG Rotation Count

```
Identical to LC 153. Return lo (the INDEX of the minimum) instead of nums[lo].
```

### LC 540 — Single Element

```
Loop: lo < hi.
mid = lo+(hi-lo)/2; if (mid%2==1) mid--;   ← snap to even
nums[mid] == nums[mid+1] → lo = mid+2      ← pair intact; go right
else                     → hi = mid        ← pair broken; go left or stay
Return nums[lo].
```

### LC 162 — Find Peak

```
Loop: lo < hi.
mid = lo+(hi-lo)/2.
nums[mid] < nums[mid+1] → lo = mid+1   ← ascending: peak to the right
else                    → hi = mid     ← descending: peak at mid or left
Return lo.
```

### Midpoint formula (always)

```cpp
int mid = lo + (hi - lo) / 2;
```

### The one case that breaks all rotated-array logic

```
nums[lo] == nums[mid] == nums[hi]
Only safe action: lo++; hi--;
```

### Three-line correctness check before writing code

1. Does a comparison at `mid` tell me in O(1) which half to discard?
2. Is the discarded half proven to contain no valid answer?
3. Is the retained half proven to contain at least one valid answer?

If all three hold, binary search is correct on that problem regardless of whether the array is globally sorted.
