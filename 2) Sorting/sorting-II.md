**Prerequisites:** basic-sorting(Selection Sort, Bubble Sort, Insertion Sort)

## Resources

- [Striver — Merge Sort (YouTube)](https://youtu.be/ogjf7ORKfd8)
- [Striver — Quick Sort (YouTube)](https://youtu.be/WIrA4YexLRQ)

---

## Table of Contents

1. [Why O(n log n) and the Lower Bound](#1-why-on-log-n-and-the-lower-bound)
2. [The Divide-and-Conquer Mental Model](#2-the-divide-and-conquer-mental-model)
3. [Recursive Bubble Sort](#3-recursive-bubble-sort)
4. [Recursive Insertion Sort](#4-recursive-insertion-sort)
5. [Merge Sort](#5-merge-sort)
6. [Quick Sort](#6-quick-sort)
7. [Lomuto vs Hoare: Detailed Comparison](#7-lomuto-vs-hoare-detailed-comparison)
8. [Recurrence Relations and Master Theorem](#8-recurrence-relations-and-master-theorem)
9. [Stability, Space, and In-Place Properties](#9-stability-space-and-in-place-properties)
10. [Comprehensive Comparison Table](#10-comprehensive-comparison-table)
11. [Edge Cases](#11-edge-cases)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Interview Patterns and Hard Problems](#13-interview-patterns-and-hard-problems)
14. [Quick Revision Cheat Sheet](#14-quick-revision-cheat-sheet)

---

## 1. Why O(n log n) and the Lower Bound

The O(n²) algorithms from basic sorting are disqualifying at interview-scale inputs. For n = 10⁵:

- O(n²) = 10¹⁰ operations — far beyond any time limit.
- O(n log n) ≈ 1.7 × 10⁶ operations — comfortably within limits.

This is not just a practical concern — it reflects a fundamental theoretical boundary.

**The O(n log n) lower bound for comparison-based sorting:** Any algorithm that sorts by comparing pairs of elements requires at least Ω(n log n) comparisons in the worst case. The proof uses a decision tree argument: there are n! possible orderings of n elements, so a decision tree that distinguishes all of them must have at least n! leaves and therefore height at least log₂(n!) ≈ n log n (by Stirling's approximation). No comparison-based algorithm can do better than this in the worst case.

This means merge sort and quick sort (in its average case) are asymptotically optimal for comparison-based sorting.

Beyond raw speed, the algorithms in this file teach ideas that reappear throughout DSA:

| Concept | Where it reappears |
|---|---|
| Divide and conquer | Binary search, segment trees, FFT, Karatsuba multiplication |
| Partition | Quickselect, Dutch National Flag, k-th order statistics |
| Merge | External sort, counting inversions, merge k sorted lists |
| Recursion trees | All divide-and-conquer complexity analysis |
| Randomization | Randomized algorithms broadly |

---

## 2. The Divide-and-Conquer Mental Model

Divide and conquer decomposes a problem of size n into subproblems of size roughly n/2, solves them recursively, and combines the results. The key insight: if the recursion tree has O(log n) levels and the work per level is O(n), the total is O(n log n).

Merge sort and quick sort both achieve this, but from structurally opposite directions:

| | Merge Sort | Quick Sort |
|---|---|---|
| Divide step | Trivial: split at midpoint — O(1) | Hard: partition around pivot — O(n) |
| Combine step | Hard: merge two sorted halves — O(n) | Trivial: nothing — O(1) |
| Result placement | Elements reach correct positions during combine | Elements reach correct positions during divide (partition) |
| Space | O(n) auxiliary for merge | O(1) auxiliary; O(log n) call stack |
| Worst case | Always O(n log n) | O(n²) on degenerate pivot choices |

This structural difference is the root cause of every difference between them: stability, space, cache behavior, and practical performance.

---

## 3. Recursive Bubble Sort

### From Iterative to Recursive

Iterative bubble sort has two nested loops: an outer pass counter and an inner pass that bubbles the current maximum to its final position. The recursion transforms the outer loop: after one complete pass, `arr[n-1]` is in its final position. Recursively sort the remaining prefix of size n-1.

```
bubbleSortRecursive(arr, n):
    if n == 1: return
    one_pass(arr, n)              // bubbles max of arr[0..n-1] to arr[n-1]
    bubbleSortRecursive(arr, n-1) // sort arr[0..n-2]
```

### C++ Implementation

```cpp
void bubbleSortRecursive(vector<int>& arr, int n) {
    if (n <= 1) return;

    bool swapped = false;
    for (int j = 0; j < n - 1; j++) {
        if (arr[j] > arr[j + 1]) {
            swap(arr[j], arr[j + 1]);
            swapped = true;
        }
    }

    // If no swap in this pass, array is already sorted
    if (!swapped) return;

    bubbleSortRecursive(arr, n - 1);
}
```

The `swapped` flag preserves the O(n) best-case behavior from the iterative optimized version.

### Dry Run

Input: `[5, 1, 4, 2, 8]`

```
Call n=5:
  j=0: 5>1  -> swap -> [1,5,4,2,8], swapped=true
  j=1: 5>4  -> swap -> [1,4,5,2,8]
  j=2: 5>2  -> swap -> [1,4,2,5,8]
  j=3: 5>8? No
  arr[4]=8 locked. Recurse n=4.

Call n=4: arr = [1,4,2,5 | 8]
  j=0: 1>4? No
  j=1: 4>2  -> swap -> [1,2,4,5,8], swapped=true
  j=2: 4>5? No
  arr[3]=5 locked. Recurse n=3.

Call n=3: arr = [1,2,4 | 5,8]
  j=0: 1>2? No
  j=1: 2>4? No
  swapped=false -> return immediately.

Final: [1, 2, 4, 5, 8]
```

### Complexity Analysis

```
T(n) = T(n-1) + O(n),  T(1) = O(1)
```

Expanding: total work = (n-1) + (n-2) + ... + 1 = n(n-1)/2 = **O(n²)**.

With the `swapped` flag: **O(n) best case** (already sorted — first pass makes zero swaps, returns immediately), **O(n²) worst case**.

**Space:** O(n) call stack. Each of the n recursive frames is on the stack simultaneously.

**Comparison to iterative:** Identical time behavior but uses O(n) stack space versus O(1) for the iterative version. The recursive form is purely pedagogical.

---

## 4. Recursive Insertion Sort

### From Iterative to Recursive

Iterative insertion sort processes elements left to right: for each position i, insert `arr[i]` into the sorted prefix `arr[0..i-1]`. The recursion inverts this: sort the first n-1 elements recursively, then insert `arr[n-1]` into the correct position in the sorted result.

### C++ Implementation

**Form 1 — backward recursion (sort n-1, then insert nth):**

```cpp
void insertionSortRecursive(vector<int>& arr, int n) {
    if (n <= 1) return;
    insertionSortRecursive(arr, n - 1);  // sort arr[0..n-2]

    int last = arr[n - 1];
    int j = n - 2;
    while (j >= 0 && arr[j] > last) {
        arr[j + 1] = arr[j];
        j--;
    }
    arr[j + 1] = last;
}
// Call as: insertionSortRecursive(arr, arr.size());
```

**Form 2 — forward recursion (Striver's style; insert i-th, recurse on rest):**

```cpp
void insertionSortRecursive(vector<int>& arr, int i, int n) {
    if (i == n) return;
    int key = arr[i];
    int j = i - 1;
    while (j >= 0 && arr[j] > key) {
        arr[j + 1] = arr[j];
        j--;
    }
    arr[j + 1] = key;
    insertionSortRecursive(arr, i + 1, n);
}
// Call as: insertionSortRecursive(arr, 1, arr.size());
```

Form 2 is often cleaner in interviews: the recursion manages "which position to process next," and the insertion logic is identical to the iterative version — making the relationship explicit.

### Dry Run (Form 2)

Input: `[5, 3, 1, 4, 2]`, call `(arr, 1, 5)`

```
i=1: key=3, j=0
  arr[0]=5 > 3  -> arr[1]=5, j=-1
  arr[0]=3
  Array: [3, 5, 1, 4, 2]. Recurse i=2.

i=2: key=1, j=1
  arr[1]=5 > 1  -> arr[2]=5, j=0
  arr[0]=3 > 1  -> arr[1]=3, j=-1
  arr[0]=1
  Array: [1, 3, 5, 4, 2]. Recurse i=3.

i=3: key=4, j=2
  arr[2]=5 > 4  -> arr[3]=5, j=1
  arr[1]=3 > 4? No -> stop
  arr[2]=4
  Array: [1, 3, 4, 5, 2]. Recurse i=4.

i=4: key=2, j=3
  arr[3]=5 > 2  -> arr[4]=5, j=2
  arr[2]=4 > 2  -> arr[3]=4, j=1
  arr[1]=3 > 2  -> arr[2]=3, j=0
  arr[0]=1 > 2? No -> stop
  arr[1]=2
  Array: [1, 2, 3, 4, 5]. Recurse i=5.

i=5: i==n -> return.

Final: [1, 2, 3, 4, 5]
```

### Complexity Analysis

```
T(n) = T(n-1) + O(n),  T(1) = O(1)
```

Solved by substitution:

```
T(n) = T(n-1) + cn
     = T(n-2) + c(n-1) + cn
     ...
     = O(1) + c(2 + 3 + ... + n)
     = c * n(n+1)/2 = O(n²)
```

| Case | Time | Reason |
|---|---|---|
| Best (sorted) | O(n) | Each insert step exits the while loop immediately |
| Average | O(n²) | |
| Worst (reverse sorted) | O(n²) | Each element shifts all the way to index 0 |

**Space:** O(n) call stack. Use the iterative version for large n in production.

---

## 5. Merge Sort

### Core Insight: Why Merging Sorted Halves Works

Given two sorted arrays A and B, you can merge them into one sorted array of size |A| + |B| in exactly O(|A| + |B|) time. Why: at each step, the next element in the merged output must be either the front of A or the front of B — these are the only two candidates. One comparison per output element. This is optimal; you cannot do better without reading both fronts.

Once this merge primitive exists, the full sort is immediate: split in half (trivial), recursively sort each half, merge the two sorted halves (O(n)). The base case — a single element — is trivially sorted. The recursion tree has O(log n) levels; each level's total merge work is O(n); total: **O(n log n)**.

### The Merge Subroutine

The merge subroutine is the critical component. Given `arr[lo..mid]` sorted and `arr[mid+1..hi]` sorted, produce a merged sorted result in `arr[lo..hi]`.

```cpp
void merge(vector<int>& arr, int lo, int mid, int hi) {
    vector<int> temp;
    temp.reserve(hi - lo + 1);

    int i = lo;        // pointer into left half  [lo..mid]
    int j = mid + 1;   // pointer into right half [mid+1..hi]

    while (i <= mid && j <= hi) {
        if (arr[i] <= arr[j]) {
            // <= not <: left half wins on ties, preserving stability
            temp.push_back(arr[i++]);
        } else {
            temp.push_back(arr[j++]);
        }
    }

    // Drain whichever half has remaining elements
    while (i <= mid) temp.push_back(arr[i++]);
    while (j <= hi)  temp.push_back(arr[j++]);

    // Copy merged result back into the original array
    for (int k = 0; k < (int)temp.size(); k++) {
        arr[lo + k] = temp[k];
    }
}
```

**Why `arr[i] <= arr[j]` (not `<`):** When elements are equal, the left half contains the element that appeared earlier in the original array. Taking it first preserves the original relative order of equal elements. This single character difference (`<=` vs `<`) is the **entire stability mechanism** of merge sort. Changing it to `<` makes the sort unstable.

**Time per merge call:** O(hi - lo + 1). Each element is compared at most once, pushed to `temp` once, and copied back once.

**Space per merge call:** O(hi - lo + 1) for the temporary buffer. Across any single level of the recursion tree, total temporary space is O(n). Only one level's merges are active at a time (the others have already returned and freed their buffers), so peak auxiliary space is O(n).

### Full Merge Sort Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

void merge(vector<int>& arr, int lo, int mid, int hi) {
    vector<int> temp;
    temp.reserve(hi - lo + 1);
    int i = lo, j = mid + 1;
    while (i <= mid && j <= hi) {
        if (arr[i] <= arr[j]) temp.push_back(arr[i++]);
        else                   temp.push_back(arr[j++]);
    }
    while (i <= mid) temp.push_back(arr[i++]);
    while (j <= hi)  temp.push_back(arr[j++]);
    for (int k = 0; k < (int)temp.size(); k++) arr[lo + k] = temp[k];
}

void mergeSort(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;                   // base case: 0 or 1 elements

    int mid = lo + (hi - lo) / 2;          // safe midpoint (no overflow)

    mergeSort(arr, lo, mid);                // sort left half
    mergeSort(arr, mid + 1, hi);            // sort right half
    merge(arr, lo, mid, hi);               // merge the two sorted halves
}

int main() {
    vector<int> arr = {38, 27, 43, 3, 9, 82, 10};
    mergeSort(arr, 0, arr.size() - 1);
    for (int x : arr) cout << x << " ";
    // Output: 3 9 10 27 38 43 82
}
```

**Critical detail:** `mid = lo + (hi - lo) / 2` instead of `(lo + hi) / 2`. When `lo` and `hi` are both large integers (e.g., 10⁹), their sum overflows a 32-bit int. The subtraction form is always safe.

**Why `mergeSort(arr, mid+1, hi)` not `mergeSort(arr, mid, hi)`:** If you pass `mid` instead of `mid+1` as the `lo` of the right half, the element at `mid` is included in both recursive calls. For a two-element array `[lo=0, hi=1]`, `mid=0`, and `mergeSort(arr, 0, 1)` would call `mergeSort(arr, 0, 0)` and `mergeSort(arr, 0, 1)` — infinite recursion.

### Dry Run

Input: `[38, 27, 43, 3]`, call `mergeSort(arr, 0, 3)`

```
mergeSort(0,3): mid=1
  mergeSort(0,1): mid=0
    mergeSort(0,0): lo==hi, return   [38 is a sorted singleton]
    mergeSort(1,1): lo==hi, return   [27 is a sorted singleton]
    merge(0,0,1):
      i=0 (38), j=1 (27): 38>27 -> temp=[27], j=2
      j>hi=1, drain left: temp=[27,38]
      arr=[27,38,43,3]

  mergeSort(2,3): mid=2
    mergeSort(2,2): lo==hi, return   [43 is a sorted singleton]
    mergeSort(3,3): lo==hi, return   [3  is a sorted singleton]
    merge(2,2,3):
      i=2 (43), j=3 (3): 43>3 -> temp=[3], j=4
      j>hi=3, drain left: temp=[3,43]
      arr=[27,38,3,43]

  merge(0,1,3):
    i=0 (27), j=2 (3):  27>3  -> temp=[3],      j=3
    i=0 (27), j=3 (43): 27<43 -> temp=[3,27],   i=1
    i=1 (38), j=3 (43): 38<43 -> temp=[3,27,38],i=2
    i>mid=1, drain right: temp=[3,27,38,43]
    arr=[3,27,38,43]

Final: [3, 27, 38, 43]
```

### Complexity Analysis

**Recurrence:** `T(n) = 2T(n/2) + cn`, `T(1) = O(1)`

**Recursion tree method** (assume n is a power of 2):

```
Level 0: 1 subproblem of size n         -> merge work = cn
Level 1: 2 subproblems of size n/2      -> merge work = 2 * c(n/2) = cn
Level 2: 4 subproblems of size n/4      -> merge work = 4 * c(n/4) = cn
...
Level k: 2^k subproblems of size n/2^k  -> merge work = 2^k * c(n/2^k) = cn
...
Level log(n): n subproblems of size 1   -> base cases, no merge
```

Every level contributes exactly `cn` work. The tree has `log₂(n)` merge levels (plus the base level). Total: `cn * log₂(n) = O(n log n)`.

This is **identical for all inputs** — merge sort has no best-case advantage. A sorted input still splits, recurses, and merges at every level.

| Case | Time | Space |
|---|---|---|
| Best | O(n log n) | O(n) + O(log n) stack |
| Average | O(n log n) | O(n) |
| Worst | O(n log n) | O(n) |

**For linked lists:** merge sort is the natural choice. You can find the midpoint using fast/slow pointers, and merge by relinking nodes — no auxiliary array needed. Space becomes O(log n) for the call stack only. Quick sort is impractical on linked lists because it requires random access for efficient partitioning.

---

## 6. Quick Sort

### Core Insight: The Partition Invariant

Quick sort is built around a single operation: **partition**. Given a pivot, rearrange the array so that:

- Every element to the left of the pivot is ≤ the pivot.
- Every element to the right of the pivot is ≥ the pivot.
- The pivot itself is in its **final correct position**.

After a single partition, the pivot is permanently placed and will never move again. Recursively sort the left and right sub-arrays independently. No merge step is needed — once both sub-arrays are sorted and the pivot is already in place, the whole array is sorted.

Two classical partitioning schemes exist and both appear in interviews.

### Lomuto Partition Scheme

**Strategy:** Use the last element as pivot. Maintain a boundary pointer `i` such that `arr[lo..i]` contains elements ≤ pivot. Scan `j` from `lo` to `hi-1`; whenever `arr[j] <= pivot`, extend the left region by incrementing `i` and swapping `arr[i]` with `arr[j]`. After the scan, place the pivot at `arr[i+1]`.

**Loop invariant:** at the start of each iteration, `arr[lo..i] <= pivot`, `arr[i+1..j-1] > pivot`, `arr[j..hi-1]` unexamined.

```cpp
int lomutoPartition(vector<int>& arr, int lo, int hi) {
    int pivot = arr[hi];  // pivot is the last element
    int i = lo - 1;       // right boundary of the "<=pivot" region (initially empty)

    for (int j = lo; j < hi; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    // Place pivot at its correct position
    swap(arr[i + 1], arr[hi]);
    return i + 1;  // pivot's final index; arr[lo..i+1-1] <= pivot <= arr[i+1+1..hi]
}
```

**What the return value means:** `arr[i+1]` is the pivot in its final correct position. `arr[lo..i] <= pivot`. `arr[i+2..hi] >= pivot`. The pivot is excluded from both recursive calls.

### Full Quick Sort — Lomuto

```cpp
#include <bits/stdc++.h>
using namespace std;

int lomutoPartition(vector<int>& arr, int lo, int hi) {
    int pivot = arr[hi];
    int i = lo - 1;
    for (int j = lo; j < hi; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[hi]);
    return i + 1;
}

void quickSort(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;                       // 0 or 1 elements: already sorted
    int pivotIdx = lomutoPartition(arr, lo, hi);
    quickSort(arr, lo, pivotIdx - 1);           // sort left of pivot (pivot excluded)
    quickSort(arr, pivotIdx + 1, hi);           // sort right of pivot (pivot excluded)
}

int main() {
    vector<int> arr = {10, 7, 8, 9, 1, 5};
    quickSort(arr, 0, arr.size() - 1);
    for (int x : arr) cout << x << " ";
    // Output: 1 5 7 8 9 10
}
```

### Hoare Partition Scheme

**Strategy:** Use the first element as pivot. Two pointers: `i` starting just left of `lo` and `j` starting just right of `hi`. Move `i` rightward until `arr[i] >= pivot`. Move `j` leftward until `arr[j] <= pivot`. If `i < j`, swap and continue. When `i >= j`, return `j`.

**Critical difference from Lomuto:** Hoare's partition does **not** place the pivot at its final position. It returns an index `j` such that `arr[lo..j]` are all ≤ elements in `arr[j+1..hi]`, but the pivot may be anywhere in `arr[lo..j]`. The recursive calls are on `arr[lo..j]` and `arr[j+1..hi]` — the left call includes `j`, not `j-1`.

```cpp
int hoarePartition(vector<int>& arr, int lo, int hi) {
    int pivot = arr[lo];  // pivot is the first element
    int i = lo - 1;
    int j = hi + 1;

    while (true) {
        do { i++; } while (arr[i] < pivot);   // find element >= pivot from left
        do { j--; } while (arr[j] > pivot);   // find element <= pivot from right
        if (i >= j) return j;                  // pointers crossed: done
        swap(arr[i], arr[j]);
    }
}
```

### Full Quick Sort — Hoare

```cpp
void quickSortHoare(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;
    int j = hoarePartition(arr, lo, hi);
    quickSortHoare(arr, lo, j);        // NOTE: lo to j (not j-1)
    quickSortHoare(arr, j + 1, hi);
}
```

**Why `(lo, j)` not `(lo, j-1)`:** `j` is not the pivot's final position; it is a partition boundary. `arr[j]` still needs to be sorted in the left partition. Using `j-1` leaves one element unsorted.

### Dry Run (Lomuto)

Input: `[10, 7, 8, 9, 1, 5]`, call `quickSort(arr, 0, 5)`

```
lomutoPartition(0,5): pivot=arr[5]=5, i=-1
  j=0: arr[0]=10 > 5, skip
  j=1: arr[1]=7  > 5, skip
  j=2: arr[2]=8  > 5, skip
  j=3: arr[3]=9  > 5, skip
  j=4: arr[4]=1  <= 5 -> i=0, swap(arr[0],arr[4]) -> [1,7,8,9,10,5]
  End: swap(arr[1],arr[5]) -> [1,5,8,9,10,7]
  pivotIdx=1.  Pivot 5 is at final position (index 1).

quickSort(0,0): lo==hi, return.   [1] is sorted.
quickSort(2,5): pivot=arr[5]=7, i=1
  j=2: 8>7, skip. j=3: 9>7, skip. j=4: 10>7, skip.
  End: swap(arr[2],arr[5]) -> [1,5,7,9,10,8]
  pivotIdx=2.  Pivot 7 at final position.

quickSort(2,1): lo>hi, return.
quickSort(3,5): pivot=arr[5]=8, i=2
  j=3: 9>8, skip. j=4: 10>8, skip.
  End: swap(arr[3],arr[5]) -> [1,5,7,8,10,9]
  pivotIdx=3.  Pivot 8 at final position.

quickSort(3,2): lo>hi, return.
quickSort(4,5): pivot=arr[5]=9, i=3
  j=4: 10>9, skip.
  End: swap(arr[4],arr[5]) -> [1,5,7,8,9,10]
  pivotIdx=4.  Pivot 9 at final position.

quickSort(4,3): lo>hi, return.
quickSort(5,5): lo==hi, return.

Final: [1, 5, 7, 8, 9, 10]
```

Note: this input (already sorted in ascending order except for the last element) was not worst case here, but for a fully sorted input `[1,5,7,8,9,10]` with Lomuto's last-element pivot, every pivot would be the maximum of its subarray, producing O(n²).

### Complexity Analysis

**Best case:** pivot always splits the array exactly in half.

```
T(n) = 2T(n/2) + O(n)  =>  O(n log n)   (same recurrence as merge sort)
```

**Worst case:** pivot is always the minimum or maximum — e.g., sorted input with last-element Lomuto pivot.

```
T(n) = T(0) + T(n-1) + O(n) = T(n-1) + O(n)  =>  O(n²)
```

The recursion tree degenerates from a balanced binary tree (depth log n) to a chain of depth n.

**Average case:** O(n log n). Even with random pivots, the expected depth of recursion is O(log n). A formal argument: any split where the pivot lands between the 25th and 75th percentile is a "good split" (sub-arrays at most 3n/4). The probability of a good split with a random pivot is 1/2. Alternating good and bad splits still gives O(log n) depth, yielding O(n log n) total. More precisely, the expected comparison count is `2n ln n ≈ 1.38 n log₂ n`.

| Case | Time | Call Stack Depth |
|---|---|---|
| Best | O(n log n) | O(log n) |
| Average | O(n log n) expected | O(log n) expected |
| Worst | O(n²) | O(n) |

**Space:** O(1) auxiliary (in-place partition). Call stack: O(log n) average, O(n) worst case. This is the main space advantage over merge sort's O(n).

### Why Quick Sort Is Not Stable

Consider `[3a, 3b, 1]` with Lomuto (pivot = arr[2] = 1):
- `j=0`: arr[0]=3a > 1, skip.
- `j=1`: arr[1]=3b > 1, skip.
- End: swap pivot to position 0 → `[1, 3b, 3a]`.

`3a` and `3b` are now reversed. The long-range swap during partition moves elements past non-adjacent positions, breaking the relative order of equal elements.

### Randomized Quick Sort

To prevent worst-case O(n²) on sorted or adversarially crafted inputs:

```cpp
int randomizedPartition(vector<int>& arr, int lo, int hi) {
    int randIdx = lo + rand() % (hi - lo + 1);
    swap(arr[randIdx], arr[hi]);    // move random pivot to last position
    return lomutoPartition(arr, lo, hi);
}

void quickSortRandomized(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;
    int pivotIdx = randomizedPartition(arr, lo, hi);
    quickSortRandomized(arr, lo, pivotIdx - 1);
    quickSortRandomized(arr, pivotIdx + 1, hi);
}
```

With a random pivot, no input can consistently produce O(n²) behavior — the adversary cannot predict which element will be chosen.

**Median-of-three:** Another strategy — pick the median of `arr[lo]`, `arr[mid]`, `arr[hi]` as pivot. Avoids worst case on sorted and reverse-sorted inputs with O(1) additional cost. Used in most production quicksort implementations.

### Three-Way Partition (Dutch National Flag)

Standard Lomuto and Hoare both degrade to O(n²) on all-equal arrays (all elements are "equal to pivot" so all go to one side). Three-way partition handles this in O(n):

```cpp
// Partition into arr[lo..lt-1] < pivot, arr[lt..gt] == pivot, arr[gt+1..hi] > pivot
pair<int,int> threeWayPartition(vector<int>& arr, int lo, int hi) {
    int pivot = arr[lo];
    int lt = lo, gt = hi, i = lo;
    while (i <= gt) {
        if      (arr[i] < pivot) swap(arr[lt++], arr[i++]);
        else if (arr[i] > pivot) swap(arr[i], arr[gt--]);
        else                     i++;
    }
    return {lt, gt};
}

void quickSort3Way(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;
    auto [lt, gt] = threeWayPartition(arr, lo, hi);
    quickSort3Way(arr, lo, lt - 1);
    quickSort3Way(arr, gt + 1, hi);
}
```

On an all-equal array, the first call places every element in the `== pivot` region, and both recursive calls immediately return. O(n) total.

---

## 7. Lomuto vs Hoare: Detailed Comparison

| Aspect | Lomuto | Hoare |
|---|---|---|
| Pivot choice | Last element | First element |
| Does pivot end at returned index? | **Yes** — `arr[ret]` is pivot, in final position | **No** — `ret` is a boundary, pivot may be anywhere in `arr[lo..ret]` |
| Recursive call bounds | `(lo, ret-1)` and `(ret+1, hi)` — pivot excluded from both | `(lo, ret)` and `(ret+1, hi)` — left call includes `ret` |
| Average swaps | ~3× more than Hoare | ~3× fewer |
| All-equal input behavior | O(n²) — all elements go left | Handles better, but still not O(n); use 3-way for this |
| Stability | Not stable | Not stable |
| Implementation complexity | Simpler; single pointer | Slightly more complex (do-while loops, careful crossing condition) |
| Common use | Textbooks, interviews (clearer to explain) | Production libraries |
| Most dangerous bug | Using `pivotIdx` instead of `pivotIdx-1` for left call (includes pivot in recursion) | Using `j-1` instead of `j` for left call (misses an element) |

**The single most common interview mistake:** mixing the recursive call bounds from one scheme with the partition function of the other. Hoare returns a boundary `j` where `arr[lo..j]` are all ≤ elements in `arr[j+1..hi]`, but the pivot is not necessarily at `j`. The left call **must** be `(lo, j)`, not `(lo, j-1)`.

---

## 8. Recurrence Relations and Master Theorem

### Common Recurrences

| Algorithm | Recurrence | Solution |
|---|---|---|
| Merge Sort | T(n) = 2T(n/2) + O(n) | O(n log n) |
| Quick Sort (best/average) | T(n) = 2T(n/2) + O(n) | O(n log n) |
| Quick Sort (worst) | T(n) = T(n-1) + O(n) | O(n²) |
| Recursive Bubble / Insertion | T(n) = T(n-1) + O(n) | O(n²) |
| Binary Search | T(n) = T(n/2) + O(1) | O(log n) |

### Master Theorem

For recurrences of the form `T(n) = aT(n/b) + O(n^k)` where `a ≥ 1`, `b > 1`:

```
Case 1: a > b^k  =>  T(n) = O(n^(log_b a))    [root-heavy tree]
Case 2: a = b^k  =>  T(n) = O(n^k * log n)    [balanced tree — each level costs the same]
Case 3: a < b^k  =>  T(n) = O(n^k)            [leaf-heavy tree]
```

**Merge sort:** `T(n) = 2T(n/2) + O(n¹)`. Here a=2, b=2, k=1. `b^k = 2^1 = 2 = a`. Case 2: T(n) = **O(n log n)**.

**Why Case 2 applies:** at every level of the recursion tree, the total work is exactly `cn` (shown in the recursion tree derivation above). A balanced tree means no level dominates, so you accumulate `O(n)` across each of the `O(log n)` levels.

**Quick sort worst case:** `T(n) = T(n-1) + O(n)`. This is not in Master Theorem form (b must be > 1 to have `n/b` structure). Solve by substitution: sum gives n(n+1)/2 = **O(n²)**.

---

## 9. Stability, Space, and In-Place Properties

### Stability Mechanisms

| Algorithm | Stable? | Why |
|---|---|---|
| Recursive Bubble Sort | Yes | Inherits iterative bubble sort's adjacent-swap stability |
| Recursive Insertion Sort | Yes | Inherits iterative insertion sort's `>` stopping condition |
| Merge Sort | Yes | `arr[i] <= arr[j]` in merge: left half wins on ties, preserving original order |
| Quick Sort (both schemes) | No | Long-range swaps in partition move elements past equal elements |

**The one-character stability test for merge sort:** `arr[i] <= arr[j]` → stable. `arr[i] < arr[j]` → unstable. The only difference is whether the left or right half wins when elements are equal.

### Space Properties

| Algorithm | Auxiliary Space | Call Stack | Total |
|---|---|---|---|
| Merge Sort | O(n) — temp arrays | O(log n) | O(n) |
| Quick Sort (Lomuto/Hoare) | O(1) — in-place partition | O(log n) avg, O(n) worst | O(log n) avg |
| Recursive Bubble/Insertion | O(1) | O(n) | O(n) |

**Merge sort on linked lists:** merge can be performed by relinking nodes — no auxiliary array needed. Space drops to O(log n) for the call stack. This is a strong reason to prefer merge sort for linked-list sorting.

### Practical Performance: Why Quick Sort Often Wins

Despite identical O(n log n) average complexity, quick sort typically outperforms merge sort on random in-memory data because:

- **Cache locality:** quick sort accesses data sequentially during partition; merge sort's copy-back step accesses two separate regions, causing more cache misses.
- **No auxiliary allocation:** merge sort allocates O(n) memory per sort call; allocation has overhead.
- **In-place:** quick sort's working set fits in cache better.
- **Constants:** quick sort's inner loop is extremely tight (one comparison, one conditional swap).

This is why `std::sort` in C++ (introsort) uses quick sort as its primary strategy.

---

## 10. Comprehensive Comparison Table

| Algorithm | Best | Average | Worst | Space | Stable | In-Place | Adaptive |
|---|---|---|---|---|---|---|---|
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No | Yes | No |
| Bubble Sort (opt) | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes | Yes |
| Recursive Bubble | O(n) | O(n²) | O(n²) | O(n) | Yes | Yes | Yes |
| Recursive Insertion | O(n) | O(n²) | O(n²) | O(n) | Yes | Yes | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | No | No |
| Quick Sort (rand) | O(n log n) | O(n log n) | O(n²) | O(log n) | No | Yes | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No | Yes | No |

### Decision Guide

| Requirement | Algorithm | Reason |
|---|---|---|
| Guaranteed O(n log n) worst case | Merge Sort or Heap Sort | Quick sort can degrade |
| Stable sort required | Merge Sort | Quick and heap are not stable |
| In-place O(n log n) | Heap Sort | Merge sort needs O(n) |
| Best average-case speed | Randomized Quick Sort | Cache-friendly, O(1) aux space |
| Sorting a linked list | Merge Sort | No random access needed; O(1) aux |
| Nearly-sorted input | Insertion Sort | O(n + k) for k inversions |
| Many duplicate keys | 3-Way Quick Sort | O(n) on all-equal input |
| Sub-arrays (< ~16 elements) | Insertion Sort | Low constants dominate |
| External sort (data on disk) | Merge Sort | Sequential access pattern |
| Find k-th element | Quickselect | O(n) average, no full sort needed |

### STL Internals

- **`std::sort` in C++:** uses **introsort** — starts with randomized quick sort; if recursion depth exceeds `2 * log(n)`, switches to heap sort (guaranteeing O(n log n) worst case); uses insertion sort for sub-arrays ≤ ~16 elements.
- **`std::stable_sort` in C++:** merge-sort based; guarantees stability and O(n log n).
- **`list.sort()` in Python / `Arrays.sort()` for objects in Java:** use **Timsort** — exploits natural runs in real-world data; uses insertion sort for short runs (< ~32–64 elements); merges runs bottom-up.

---

## 11. Edge Cases

| Input | Behavior |
|---|---|
| `[]` empty | All: `lo >= hi` (or `n <= 1`) fires immediately. No work done. |
| `[x]` single element | All: base case, return immediately. |
| `[1,2,3,...,n]` sorted | Merge: O(n log n) unchanged — still fully recurses and merges. Quick (Lomuto, last-element pivot): **O(n²)** — pivot is always the maximum, producing partitions of size 0 and n-1. Insertion: O(n). |
| `[n,...,2,1]` reverse sorted | Merge: O(n log n). Quick (Lomuto, last-element pivot): **O(n²)** — pivot is always the minimum. Insertion: O(n²). |
| `[k,k,...,k]` all equal | Merge: O(n log n). Lomuto: **O(n²)** — every element satisfies `arr[j] <= pivot`, all go left, pivot always placed at last position of subarray. 3-Way Quick: **O(n)** — entire array classified as `== pivot` in first call. |
| `[2,1]` two elements | All: one comparison, at most one swap. |
| Large n on recursive quick sort without tail recursion | Risk of stack overflow on worst-case O(n) depth. Mitigate: randomize pivot, or use iterative version with explicit stack. |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1: Integer overflow in midpoint computation

```cpp
// Wrong: lo+hi overflows when both are near INT_MAX
int mid = (lo + hi) / 2;

// Correct:
int mid = lo + (hi - lo) / 2;
```

### Mistake 2: Off-by-one in merge sort recursion — passing mid instead of mid+1

```cpp
// Wrong: right half starts at mid, overlapping with left half
mergeSort(arr, lo, mid);
mergeSort(arr, mid, hi);    // mid is in both calls -> infinite recursion on [lo,lo+1]

// Correct:
mergeSort(arr, lo, mid);
mergeSort(arr, mid + 1, hi);
```

### Mistake 3: Forgetting to drain remaining elements in merge

```cpp
// Wrong: stops after one half is exhausted, leaving the other uncopied
while (i <= mid && j <= hi) { ... }
// Missing: while(i<=mid) and while(j<=hi) drain loops

// Correct: always add both drain loops after the main while
while (i <= mid) temp.push_back(arr[i++]);
while (j <= hi)  temp.push_back(arr[j++]);
```

### Mistake 4: Forgetting to copy temp array back in merge

```cpp
// Wrong: temp is built but arr is never updated
void merge(vector<int>& arr, int lo, int mid, int hi) {
    vector<int> temp;
    // ... fill temp correctly ...
    // MISSING: copy temp back!
}

// Correct: always copy back
for (int k = 0; k < (int)temp.size(); k++) arr[lo + k] = temp[k];
```

### Mistake 5: Using < instead of <= in merge (breaks stability)

```cpp
// Wrong: right half wins on ties -> unstable
if (arr[i] < arr[j]) temp.push_back(arr[i++]);

// Correct: left half wins on ties -> stable
if (arr[i] <= arr[j]) temp.push_back(arr[i++]);
```

### Mistake 6: Wrong base case in quick sort

```cpp
// Wrong: misses lo==hi; also if lo>hi is possible from malformed calls
if (lo > hi) return;   // misses the valid single-element case lo==hi which is fine but
                        // more importantly, a wrong partition could call quickSort(lo, lo)
                        // and re-enter partition unnecessarily

// Correct:
if (lo >= hi) return;  // handles both lo==hi (single element) and lo>hi (empty range)
```

### Mistake 7: Using Lomuto bounds with Hoare's partition (or vice versa)

```cpp
// WRONG: Hoare returns j where arr[j] is NOT necessarily the pivot's final position
int j = hoarePartition(arr, lo, hi);
quickSort(arr, lo, j - 1);  // misses arr[j], which still needs to be sorted on the left
quickSort(arr, j + 1, hi);

// CORRECT for Hoare:
quickSort(arr, lo, j);      // j is a boundary, not the pivot index
quickSort(arr, j + 1, hi);

// CORRECT for Lomuto:
int p = lomutoPartition(arr, lo, hi);
quickSort(arr, lo, p - 1);  // pivot is at p; exclude it from both calls
quickSort(arr, p + 1, hi);
```

This is the highest-frequency correctness error on quick sort in interviews.

### Mistake 8: Including pivot in Lomuto recursive calls

```cpp
// Wrong: pivot is already at its final position; including it causes infinite recursion
quickSort(arr, lo, pivotIdx);      // includes pivotIdx
quickSort(arr, pivotIdx, hi);      // includes pivotIdx

// Correct:
quickSort(arr, lo, pivotIdx - 1);
quickSort(arr, pivotIdx + 1, hi);
```

### Mistake 9: Off-by-one in Hoare termination condition

```cpp
// Wrong: strict > misses the i==j case; can cause an infinite swap loop
if (i > j) return j;

// Correct: >= handles the case where pointers meet exactly
if (i >= j) return j;
```

---

## 13. Interview Patterns and Hard Problems

### Problems That Use Merge Sort

| Problem | How merge sort applies |
|---|---|
| Count inversions in array | Count cross-inversions during merge: when `arr[j]` (right) is taken before `arr[i]` (left), add `mid - i + 1` to the count. O(n log n). |
| Count smaller numbers after self (LC 315) | Modified merge sort tracking how far each element moves during merges |
| Reverse pairs (LC 493) | Count pairs during merge where `left > 2 * right` |
| Sort linked list (LC 148) | Merge sort with fast/slow pointer split; O(n log n) time, O(log n) space |
| Merge k sorted arrays/lists | k-way merge using a min-heap: O(n log k) |
| External sort | Merge passes over disk-resident data |

**Counting inversions — the canonical merge sort extension:**

```cpp
long long mergeAndCount(vector<int>& arr, int lo, int mid, int hi) {
    long long count = 0;
    vector<int> temp;
    int i = lo, j = mid + 1;
    while (i <= mid && j <= hi) {
        if (arr[i] <= arr[j]) {
            temp.push_back(arr[i++]);
        } else {
            count += (mid - i + 1);  // arr[i..mid] all form inversions with arr[j]
            temp.push_back(arr[j++]);
        }
    }
    while (i <= mid) temp.push_back(arr[i++]);
    while (j <= hi)  temp.push_back(arr[j++]);
    for (int k = 0; k < (int)temp.size(); k++) arr[lo + k] = temp[k];
    return count;
}

long long countInversions(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return 0;
    int mid = lo + (hi - lo) / 2;
    long long count = countInversions(arr, lo, mid)
                    + countInversions(arr, mid + 1, hi)
                    + mergeAndCount(arr, lo, mid, hi);
    return count;
}
```

When a right-half element `arr[j]` is taken during merge, every remaining left-half element `arr[i..mid]` is greater than `arr[j]` and appears before it in the original array — all `mid - i + 1` of them are inversions with `arr[j]`.

### Problems That Use Quick Sort / Partition

| Problem | How partition applies |
|---|---|
| K-th largest/smallest element (LC 215) | Quickselect: partition and recurse only on the side containing index k |
| Sort Colors / Dutch National Flag (LC 75) | 3-way partition directly solves it in O(n) one pass |
| Top K frequent elements (LC 347) | Quickselect on frequency array |
| Wiggle sort (LC 280, 324) | Partition-based element placement |

### Quickselect — O(n) Average Selection

Quick sort's partition can find the k-th smallest element without fully sorting:

```cpp
// Returns the kth smallest element (k is 0-indexed) in arr[lo..hi]
// Modifies arr in place
int quickSelect(vector<int>& arr, int lo, int hi, int k) {
    if (lo == hi) return arr[lo];
    int pivotIdx = lomutoPartition(arr, lo, hi);
    if (pivotIdx == k)       return arr[pivotIdx];
    else if (pivotIdx > k)   return quickSelect(arr, lo, pivotIdx - 1, k);
    else                     return quickSelect(arr, pivotIdx + 1, hi, k);
}
```

- **Average:** O(n). Expected subproblem sizes: n → n/2 → n/4 → ... → sum = 2n.
- **Worst:** O(n²) on degenerate pivot choices.
- **With randomized pivot:** O(n) expected, regardless of input.

### Pattern Recognition Summary

| Signal in problem | Algorithm to reach for |
|---|---|
| "count pairs satisfying order condition" | Modified merge sort |
| "k-th smallest / largest" | Quickselect (O(n) avg) |
| "stable sort required" | Merge sort |
| "sort linked list" | Merge sort |
| "external sort, data on disk" | Merge sort |
| "3 colors / 3-way partition around value" | Dutch National Flag / 3-way quick sort |
| "nearly sorted, few inversions" | Insertion sort (O(n + k)) |
| "guaranteed O(n log n), no extra space" | Heap sort |

---

## 14. Quick Revision Cheat Sheet

This section is self-contained for same-night revision.

---

### The Divide-and-Conquer Split

| | Merge Sort | Quick Sort |
|---|---|---|
| Hard step | Combine (merge) | Divide (partition) |
| Easy step | Divide (split at midpoint) | Combine (nothing) |
| Space | O(n) aux | O(1) aux, O(log n) stack |
| Worst case | Always O(n log n) | O(n²) on degenerate pivot |
| Stable | Yes (`<=` in merge) | No |

---

### Recursive Bubble Sort

```cpp
void bubbleSortRecursive(vector<int>& arr, int n) {
    if (n <= 1) return;
    bool swapped = false;
    for (int j = 0; j < n - 1; j++)
        if (arr[j] > arr[j+1]) { swap(arr[j], arr[j+1]); swapped = true; }
    if (!swapped) return;
    bubbleSortRecursive(arr, n - 1);
}
```

- T(n) = T(n-1) + O(n) => O(n²); O(n) best (swapped flag).
- O(n) stack space. Purely pedagogical.

---

### Recursive Insertion Sort

```cpp
void insertionSortRecursive(vector<int>& arr, int i, int n) {
    if (i == n) return;
    int key = arr[i]; int j = i - 1;
    while (j >= 0 && arr[j] > key) { arr[j+1] = arr[j]; j--; }
    arr[j+1] = key;
    insertionSortRecursive(arr, i + 1, n);
}
// Call: insertionSortRecursive(arr, 1, arr.size());
```

- T(n) = T(n-1) + O(n) => O(n²); O(n) best on sorted input.
- O(n) stack space. Use iterative for large n.

---

### Merge Sort

```cpp
void merge(vector<int>& arr, int lo, int mid, int hi) {
    vector<int> temp; int i=lo, j=mid+1;
    while (i<=mid && j<=hi)
        temp.push_back(arr[i]<=arr[j] ? arr[i++] : arr[j++]); // <= for stability
    while (i<=mid) temp.push_back(arr[i++]);
    while (j<=hi)  temp.push_back(arr[j++]);
    for (int k=0; k<(int)temp.size(); k++) arr[lo+k]=temp[k]; // copy back
}
void mergeSort(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi-lo)/2;     // safe midpoint
    mergeSort(arr, lo, mid);
    mergeSort(arr, mid+1, hi);    // mid+1, NOT mid
    merge(arr, lo, mid, hi);
}
```

- T(n) = 2T(n/2) + O(n) => O(n log n) via Master Theorem Case 2 (a = b^k = 2).
- O(n log n) in ALL cases. No best/worst distinction.
- Space: O(n) aux + O(log n) stack = O(n).
- Stability mechanism: `arr[i] <= arr[j]` — left half wins on equal keys.

---

### Quick Sort — Lomuto

```cpp
int lomutoPartition(vector<int>& arr, int lo, int hi) {
    int pivot = arr[hi], i = lo - 1;
    for (int j = lo; j < hi; j++)
        if (arr[j] <= pivot) { i++; swap(arr[i], arr[j]); }
    swap(arr[i+1], arr[hi]);
    return i+1;  // pivot's FINAL position
}
void quickSort(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;
    int p = lomutoPartition(arr, lo, hi);
    quickSort(arr, lo, p-1);    // pivot EXCLUDED from both calls
    quickSort(arr, p+1, hi);
}
```

- Pivot is at `arr[ret]` in its final position after partition.
- Recursive calls: `(lo, p-1)` and `(p+1, hi)` — pivot never revisited.

---

### Quick Sort — Hoare

```cpp
int hoarePartition(vector<int>& arr, int lo, int hi) {
    int pivot=arr[lo], i=lo-1, j=hi+1;
    while (true) {
        do { i++; } while (arr[i] < pivot);
        do { j--; } while (arr[j] > pivot);
        if (i >= j) return j;
        swap(arr[i], arr[j]);
    }
}
void quickSortHoare(vector<int>& arr, int lo, int hi) {
    if (lo >= hi) return;
    int j = hoarePartition(arr, lo, hi);
    quickSortHoare(arr, lo, j);      // j INCLUDED in left call (NOT j-1)
    quickSortHoare(arr, j+1, hi);
}
```

- `ret` is NOT the pivot's final position; it is a partition boundary.
- Left call must be `(lo, j)` — using `(lo, j-1)` misses an element.
- ~3× fewer swaps than Lomuto.

---

### Quick Sort Critical Points

- NOT stable (long-range swaps reorder equal elements).
- O(log n) avg stack space; O(n) worst case stack space.
- Worst case on: sorted input (Lomuto last-pivot), reverse-sorted, all-equal.
- Randomize pivot to eliminate adversarial worst cases.
- 3-way partition for arrays with many duplicates → O(n) on all-equal.
- Quickselect: partition + recurse on one side only → O(n) average k-th element.

---

### Master Theorem Quick Reference

```
T(n) = aT(n/b) + O(n^k):
  a > b^k  =>  O(n^{log_b a})
  a = b^k  =>  O(n^k * log n)   <- Merge Sort: a=b=2, k=1 -> O(n log n)
  a < b^k  =>  O(n^k)
```

---

### Stability Reference

| Stable | Not Stable |
|---|---|
| Merge Sort | Quick Sort (both schemes) |
| Bubble Sort | Heap Sort |
| Insertion Sort | Selection Sort |
| Counting / Radix / Timsort | — |

---

### Inversion Count via Merge Sort

When right-half element `arr[j]` is taken before left-half element `arr[i]` during merge:
all remaining `arr[i..mid]` are inversions with `arr[j]` → add `mid - i + 1` to count.

---

### Full Complexity Reference

| Algorithm | Best | Average | Worst | Space | Stable |
|---|---|---|---|---|---|
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort (rand) | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Recursive Bubble | O(n) | O(n²) | O(n²) | O(n) | Yes |
| Recursive Insertion | O(n) | O(n²) | O(n²) | O(n) | Yes |
| Insertion Sort (iter) | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |

---

### Decision Tree (Interview)

```
Need O(n log n) guaranteed worst case?
  Yes, and need stable?       -> Merge Sort
  Yes, need in-place?         -> Heap Sort
  No guarantee needed?        -> Randomized Quick Sort (fastest in practice)

Sorting a linked list?         -> Merge Sort

External sort?                 -> Merge Sort

Nearly sorted, few inversions? -> Insertion Sort

Many duplicates?               -> 3-Way Quick Sort

Finding k-th element?          -> Quickselect (O(n) avg)

n <= 16?                       -> Insertion Sort
```
