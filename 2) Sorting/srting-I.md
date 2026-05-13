- [Striver — Sorting Part 1 (YouTube)](https://youtu.be/HGk_ypEuS24)

---

## Table of Contents

1. [Why Sorting Matters](#1-why-sorting-matters)
2. [The Unifying Mental Model](#2-the-unifying-mental-model)
3. [Inversions: The Hidden Metric](#3-inversions-the-hidden-metric)
4. [Selection Sort](#4-selection-sort)
5. [Bubble Sort](#5-bubble-sort)
6. [Insertion Sort](#6-insertion-sort)
7. [Recursive Variants](#7-recursive-variants)
8. [Stability: Formal Definition and Consequences](#8-stability-formal-definition-and-consequences)
9. [Comparative Analysis](#9-comparative-analysis)
10. [Edge Cases](#10-edge-cases)
11. [Common Mistakes and Pitfalls](#11-common-mistakes-and-pitfalls)
12. [Interview Patterns and Extensions](#12-interview-patterns-and-extensions)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. Why Sorting Matters

Sorting is not a topic — it is a prerequisite. A large fraction of interview problems become tractable only after the input is sorted, because sorting imposes structure that algorithms can exploit.

| Problem type | What sorting unlocks |
|---|---|
| Two Sum / K-Sum | Two-pointer approach |
| Merge Intervals | Overlapping intervals become adjacent |
| 3Sum, 4Sum | Duplicate skipping without a hash set |
| Median / order statistics | Direct index access |
| Frequency grouping | Equal elements cluster together |
| Greedy scheduling | Natural ordering reveals greedy choices |

The practical lesson: when a problem seems hard on an unsorted input, ask whether sorting first reduces it to something standard.

---

## 2. The Unifying Mental Model

All three algorithms — selection sort, bubble sort, and insertion sort — maintain a **growing sorted region** and extend it by exactly one element per pass.

At the start of pass `i`, a region of `i` elements is in its correct final configuration. Each pass extends that guarantee to `i+1` elements. After `n-1` passes, the entire array is sorted.

They differ only in *how* they extend the sorted region:

| Algorithm | Sorted region | Extension strategy |
|---|---|---|
| Selection Sort | Left prefix | Find minimum of unsorted suffix; swap to front |
| Bubble Sort | Right suffix | Bubble maximum of unsorted prefix to back via adjacent swaps |
| Insertion Sort | Left prefix | Extract first unsorted element; shift prefix right until correct slot |

This model makes the O(n²) nature immediate: n passes, each doing O(n) work. All differences in adaptivity, stability, and constant factors follow from the specific strategy.

---

## 3. Inversions: The Hidden Metric

An **inversion** is a pair of indices `(i, j)` with `i < j` and `arr[i] > arr[j]`. Inversions quantify exactly how unsorted an array is.

- Sorted array: 0 inversions.
- Reverse-sorted array of size n: n(n-1)/2 inversions (maximum).
- A swap or shift that corrects one out-of-order pair removes exactly one inversion.

This connects directly to adaptive sorting:

- **Bubble sort's total swap count equals the number of inversions.** Each adjacent swap corrects exactly one inversion. The algorithm finishes when inversions reach zero.
- **Insertion sort's total shift count equals the number of inversions.** Each right-shift of one position eliminates one inversion.
- **Selection sort has no such relationship.** Its comparison count is always n(n-1)/2 regardless of how many inversions exist. It cannot sense sortedness.

This is why bubble sort and insertion sort are **adaptive** — their running time is O(n + k) where k is the inversion count — while selection sort is not. For an already-sorted input, k = 0, giving O(n). For a reverse-sorted input, k = n(n-1)/2, giving O(n²).

**Interview application:** "How many swaps does bubble sort perform on this input?" — the answer is the inversion count of the input.

---

## 4. Selection Sort

### Intuition

You are placing exam sheets on a table in order. The strategy: scan the entire pile, find the sheet with the lowest marks, place it at position 0. Scan the rest, find the next lowest, place it at position 1. Repeat.

**You commit to placing exactly one element in its final position per pass, with certainty — by scanning everything that remains.** The inner loop cannot terminate early because you cannot declare a minimum until you have seen every remaining element. This is what removes all adaptive behavior.

### Algorithm

```
for i from 0 to n-2:
    minIdx = i
    for j from i+1 to n-1:
        if arr[j] < arr[minIdx]: minIdx = j
    swap arr[i] and arr[minIdx]
```

Outer loop: n-1 iterations (after n-1 passes, the last element is automatically placed).
Inner loop: finds the index of the minimum in `arr[i..n-1]`.
One swap places that minimum at `arr[i]`.

### C++ Implementation

```cpp
void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[minIdx]) {
                minIdx = j;
            }
        }
        if (minIdx != i) {
            swap(arr[i], arr[minIdx]);
        }
    }
}
```

The `if (minIdx != i)` guard avoids a redundant self-swap when the minimum is already at `i`. This does not change the comparison count, but reduces swaps to 0 on an already-sorted input.

### Dry Run

Input: `[64, 25, 12, 22, 11]`

```
Pass i=0: scan [64,25,12,22,11]
  j=1: 25 < 64  -> minIdx=1
  j=2: 12 < 25  -> minIdx=2
  j=3: 22 > 12  -> no change
  j=4: 11 < 12  -> minIdx=4
  swap arr[0] and arr[4]
  Array: [11, 25, 12, 22, 64]

Pass i=1: scan [25,12,22,64]
  j=2: 12 < 25  -> minIdx=2
  j=3: 22 > 12  -> no change
  j=4: 64 > 12  -> no change
  swap arr[1] and arr[2]
  Array: [11, 12, 25, 22, 64]

Pass i=2: scan [25,22,64]
  j=3: 22 < 25  -> minIdx=3
  j=4: 64 > 22  -> no change
  swap arr[2] and arr[3]
  Array: [11, 12, 22, 25, 64]

Pass i=3: scan [25,64]
  j=4: 64 > 25  -> no change
  minIdx == i, no swap
  Array: [11, 12, 22, 25, 64]

Total comparisons: 4+3+2+1 = 10 = n(n-1)/2 for n=5.
```

### Complexity Analysis

The inner loop for pass `i` runs `n-1-i` iterations. Total comparisons:

```
sum_{i=0}^{n-2} (n-1-i) = (n-1) + (n-2) + ... + 1 = n(n-1)/2 = O(n²)
```

This is **identical for all inputs** — sorted, reverse-sorted, random. Selection sort is the only elementary sort with no best-case advantage.

The number of swaps is at most `n-1` (one per outer iteration). This is O(n), making selection sort uniquely attractive when **writes are expensive** (e.g., flash memory or EEPROM), because every other O(n²) sort performs O(n²) writes in the worst case.

| Case | Comparisons | Swaps |
|---|---|---|
| Best (sorted) | n(n-1)/2 | 0 |
| Average | n(n-1)/2 | O(n) |
| Worst (reverse sorted) | n(n-1)/2 | n-1 |
| Space | — | O(1) |

### Why Selection Sort Is Not Stable

Consider `[3a, 3b, 1]` where `3a` and `3b` are both value 3 but we track original order.

Pass 0: minimum is `1` at index 2. Swap `arr[0]` with `arr[2]`.
Array becomes `[1, 3b, 3a]`.

`3a` now comes after `3b`, reversing their original order. The long-range swap moves `arr[0]` past the equal element `3b` without any opportunity for `3b` to maintain its position. Stability is broken.

Selection sort **can** be made stable by replacing the swap with a rotation: shift `arr[i..minIdx-1]` one position right and insert the minimum at `arr[i]`. This preserves relative order but costs O(n) writes per pass, making it functionally equivalent to insertion sort with more complexity.

---

## 5. Bubble Sort

### Intuition

Larger elements bubble rightward through adjacent swaps, like air bubbles rising in water. After pass 1, the largest element is at `arr[n-1]`. After pass 2, the second largest is at `arr[n-2]`. After pass k, the k largest elements occupy their final positions at the right end.

The local nature of swaps — only adjacent elements ever exchange — is precisely what makes bubble sort stable: equal elements can never swap past each other.

### Naive Implementation

```cpp
void bubbleSortNaive(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
            }
        }
    }
}
```

The inner loop runs to `n-1-i` because the last `i` elements are already sorted — no need to revisit them.

### Optimized Implementation (Early Termination)

**Key insight:** if a complete pass produces no swaps, the array is already sorted. No further passes can change anything.

```cpp
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        bool swapped = false;
        for (int j = 0; j < n - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```

This changes the best-case complexity to O(n): on a sorted input, the first pass makes zero swaps, `swapped` remains false, and the outer loop breaks after exactly n-1 comparisons.

### Dry Run

Input: `[5, 1, 4, 2, 8]`

```
Pass i=0: (j runs 0..3)
  j=0: 5>1  -> swap -> [1,5,4,2,8], swapped=true
  j=1: 5>4  -> swap -> [1,4,5,2,8], swapped=true
  j=2: 5>2  -> swap -> [1,4,2,5,8], swapped=true
  j=3: 5>8? No
  End of pass: 8 is locked at arr[4]

Pass i=1: (j runs 0..2)
  j=0: 1>4? No
  j=1: 4>2  -> swap -> [1,2,4,5,8], swapped=true
  j=2: 4>5? No
  End of pass: 5,8 locked

Pass i=2: (j runs 0..1)
  j=0: 1>2? No
  j=1: 2>4? No
  swapped=false -> break

Sorted: [1, 2, 4, 5, 8]  (terminated after 2 passes, not 4)
```

**Already-sorted input:** `[1, 2, 3, 4, 5]`

```
Pass i=0:
  j=0: 1>2? No. j=1: 2>3? No. j=2: 3>4? No. j=3: 4>5? No.
  swapped=false -> break immediately
Total comparisons: n-1 = 4 = O(n)
```

### Complexity Analysis

The worst case is a reverse-sorted array: every comparison triggers a swap. The total swap count equals the number of inversions, which is n(n-1)/2 for reverse-sorted input.

| Case | Description | Time |
|---|---|---|
| Best | Already sorted (with optimization) | O(n) |
| Average | Random input | O(n²) |
| Worst | Reverse sorted | O(n²) |
| Space | — | O(1) |

### Why Bubble Sort Is Stable

The swap condition is `arr[j] > arr[j+1]` (strict greater-than). When `arr[j] == arr[j+1]`, no swap occurs. Equal elements are never exchanged, so their original relative order is always preserved.

**Critical:** if you write `arr[j] >= arr[j+1]`, the sort becomes unstable because equal elements are swapped unnecessarily. The strict inequality is not arbitrary — it is what guarantees stability.

---

## 6. Insertion Sort

### Intuition

Insertion sort mirrors how most people sort a hand of playing cards. You hold a growing sorted hand on the left. Each time you pick up a new card, you slide it leftward through the sorted cards until it reaches its correct position.

Formally: at the start of iteration `i`, `arr[0..i-1]` is sorted (as a sorted subsequence). The element `arr[i]` is extracted, and `arr[0..i-1]` is scanned right-to-left, shifting elements right to make room, until the correct insertion point is found.

**The critical difference from selection sort:** insertion sort's inner loop can exit early. As soon as `arr[j] <= key`, the correct position is found and no further shifting is needed. This early-exit property is what makes insertion sort adaptive.

### Iterative Implementation

```cpp
void insertionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        int key = arr[i];   // must be saved before anything is overwritten
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j]; // shift right
            j--;
        }
        arr[j + 1] = key;   // j is one past the insertion point after exit
    }
}
```

**Why shifting instead of swapping?** Shifting overwrites `arr[j+1]` with `arr[j]` — one write per step. A swap requires three writes (via a temporary). Shifting is strictly cheaper and is why insertion sort has better constant factors than bubble sort despite identical asymptotic complexity.

**Why `arr[j+1] = key`?** After the while loop exits, `j` has been decremented one position past the correct insertion slot. The hole created by shifting is at `j+1`, which is always the correct placement index.

### Dry Run

Input: `[3, 1, 4, 2]`

```
i=1: key=1, j=0
  arr[0]=3 > 1  -> arr[1]=3, j=-1
  j < 0, exit while
  arr[0] = 1
  Array: [1, 3, 4, 2]

i=2: key=4, j=1
  arr[1]=3 > 4? No -> exit immediately
  arr[2] = 4 (no change)
  Array: [1, 3, 4, 2]

i=3: key=2, j=2
  arr[2]=4 > 2  -> arr[3]=4, j=1
  arr[1]=3 > 2  -> arr[2]=3, j=0
  arr[0]=1 > 2? No -> exit while
  arr[1] = 2
  Array: [1, 2, 3, 4]
```

### Complexity Analysis

**Best case — already sorted:** The condition `arr[j] > key` is immediately false for every `i` because `arr[i-1] <= arr[i]`. The while loop never executes. Each of the n-1 outer iterations does O(1) work. Total: **O(n)**.

**Worst case — reverse sorted:** For each `i`, the key must shift all the way to index 0. The while loop runs `i` iterations. Total shifts:

```
sum_{i=1}^{n-1} i = n(n-1)/2 = O(n²)
```

**Average case:** On random input, on average each key shifts halfway through the sorted prefix — about i/2 steps for element at index i. Total: n(n-1)/4 = O(n²).

**Nearly-sorted input (k inversions):** If the total number of inversions is k, total shifts = k. Runtime is O(n + k). If k = O(n), this is O(n) — extremely fast.

| Case | Time | Space |
|---|---|---|
| Best (sorted, 0 inversions) | O(n) | O(1) |
| Average | O(n²) | O(1) |
| Worst (reverse sorted) | O(n²) | O(1) |

### Why Insertion Sort Is Stable

The inner while loop condition is `arr[j] > key` (strict). When `arr[j] == key`, the loop stops and `key` is placed at `arr[j+1]` — immediately to the right of the equal element, preserving their original left-to-right order. Equal elements are never moved past each other.

### Why Insertion Sort Matters in Production

Despite O(n²) asymptotic complexity, insertion sort significantly outperforms both selection and bubble sort on random data in practice due to:

- **Lower constant factor:** shifting requires fewer writes than swapping.
- **Cache efficiency:** the inner loop accesses `arr[j]`, `arr[j+1]` sequentially — excellent spatial locality.
- **Fewest comparisons on average:** roughly n²/4 vs n²/2 for bubble and selection on random data.
- **Optimal on small arrays:** for n ≤ 10–16, the overhead of O(n log n) algorithms (recursion, cache misses, function call overhead) exceeds insertion sort's total cost.

This is why `std::sort` in C++ (introsort) and Python/Java's Timsort both switch to insertion sort for sub-arrays smaller than a threshold (typically 16–32 elements).

---

## 7. Recursive Variants

### Recursive Insertion Sort

The recursive structure expresses insertion sort's logic directly: to sort `arr[0..n-1]`, recursively sort `arr[0..n-2]`, then insert `arr[n-1]` into the already-sorted prefix.

**Recurrence:**

```
T(n) = T(n-1) + O(n),  T(1) = O(1)
```

Solving by substitution:

```
T(n) = T(n-1) + cn
     = T(n-2) + c(n-1) + cn
     ...
     = T(1) + c(2 + 3 + ... + n)
     = O(1) + c * n(n+1)/2
     = O(n²)
```

**Implementation (Striver's style — single function):**

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

Each call handles inserting element at index `i` into the sorted `arr[0..i-1]`, then delegates the rest.

**Space cost:** O(n) call stack — one stack frame per element. This is a meaningful drawback for large arrays relative to the O(1) iterative version. The asymptotic time behavior is identical.

### Recursive Bubble Sort

```cpp
void bubbleSortRecursive(vector<int>& arr, int n) {
    if (n == 1) return;
    bool swapped = false;
    for (int j = 0; j < n - 1; j++) {
        if (arr[j] > arr[j + 1]) {
            swap(arr[j], arr[j + 1]);
            swapped = true;
        }
    }
    if (!swapped) return;       // optimization: already sorted
    bubbleSortRecursive(arr, n - 1);
}
```

Recurrence: T(n) = T(n-1) + O(n) => O(n²). Call stack depth: O(n).

---

## 8. Stability: Formal Definition and Consequences

**Definition:** A sorting algorithm is **stable** if, for any two elements `a` and `b` with equal keys, if `a` appears before `b` in the input, then `a` appears before `b` in the output.

**Why it matters:** The standard multi-key sort idiom depends entirely on stability. To sort employees first by department, then by salary: sort by salary first (stably), then by department (stably). The second sort preserves the salary ordering within each department group. If either sort is unstable, the final ordering is unpredictable.

| Algorithm | Stable? | Reason |
|---|---|---|
| Selection Sort | **No** | Long-range swap can jump element past equal elements |
| Bubble Sort | **Yes** | Adjacent swaps only; equal elements never swapped (`>` not `>=`) |
| Insertion Sort | **Yes** | Shift stops at equal elements; key placed just to the right |
| Merge Sort | Yes | Merge step takes left element first on tie |
| Quick Sort (standard) | No | Partition swaps non-adjacent elements |
| Heap Sort | No | Heap operations break adjacency |

**Making selection sort stable:** Replace `swap(arr[i], arr[minIdx])` with a rotation — shift `arr[i..minIdx-1]` one position right and place the minimum at `arr[i]`. This costs O(n) writes per pass, turning selection sort into something functionally equivalent to insertion sort.

---

## 9. Comparative Analysis

### Full Complexity Table

| Algorithm | Best | Average | Worst | Space | Stable | Adaptive | Swaps/Shifts |
|---|---|---|---|---|---|---|---|
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No | No | O(n) swaps |
| Bubble Sort (naive) | O(n²) | O(n²) | O(n²) | O(1) | Yes | No | O(n²) |
| Bubble Sort (optimized) | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes | O(n²) |
| Insertion Sort (iterative) | O(n) | O(n²) | O(n²) | O(1) | Yes | Yes | O(n²) shifts |
| Insertion Sort (recursive) | O(n) | O(n²) | O(n²) | O(n) | Yes | Yes | O(n²) shifts |

### Decision Guide

| Situation | Best choice | Why |
|---|---|---|
| Writes are expensive (flash, EEPROM) | Selection Sort | At most n-1 swaps total |
| Input is already or nearly sorted | Insertion Sort | O(n + k) where k = inversions |
| Need to detect if array is sorted quickly | Optimized Bubble Sort | Single O(n) pass with zero swaps proves sorted |
| Stable sort needed, small n | Insertion Sort | Stable + lowest practical constant factor |
| Teaching / interview explanation | Selection Sort | Simplest invariant to state and prove |
| Sub-array in quicksort/mergesort | Insertion Sort | Low constant factor, cache-friendly |

---

## 10. Edge Cases

| Input | Notes |
|---|---|
| `[]` (empty) | n=0: outer loop never executes in all three. No error. |
| `[42]` (single element) | n=1: outer loop does not execute. Recursive base case fires immediately. |
| `[1,2,3,4,5]` (sorted) | Selection: O(n²) comparisons, 0 swaps. Bubble (opt): O(n), breaks after pass 0. Insertion: O(n), while loop never executes. |
| `[5,4,3,2,1]` (reverse sorted) | All three hit worst case. Bubble: n(n-1)/2 swaps. Insertion: n(n-1)/2 shifts. Selection: n-1 swaps but n(n-1)/2 comparisons. |
| `[3,3,3,3]` (all equal) | Selection: minIdx==i always, 0 swaps. Bubble: 0 swaps, terminates in 1 pass (O(n)). Insertion: while condition `arr[j]>key` is false immediately, 0 shifts. |
| `[2,1]` (two elements) | Minimal swap case. All three perform exactly 1 comparison and 1 swap/shift. |
| `[1,2,...,n,0]` (one element at end far out of place) | Insertion sort: O(n) — key=0 shifts through all n-1 elements. Selection: still O(n²) comparisons. Bubble: O(n²) — 0 triggers a chain of swaps in every pass. |

---

## 11. Common Mistakes and Pitfalls

### Mistake 1: Wrong inner loop bound in bubble sort

```cpp
// Wrong: j < n causes arr[j+1] to access arr[n], which is out of bounds
for (int j = 0; j < n; j++) {

// Also wrong: not reducing the upper bound each pass (re-examines sorted suffix)
for (int j = 0; j < n - 1; j++) {  // inside loop over i, but no -i

// Correct:
for (int j = 0; j < n - 1 - i; j++) {
```

### Mistake 2: Not saving `key` before shifting in insertion sort

```cpp
// Wrong: arr[i] is overwritten by the first shift before it is read
int j = i - 1;
while (j >= 0 && arr[j] > arr[i]) {  // arr[i] changes as arr[j+1]=arr[j] executes
    arr[j + 1] = arr[j];
    j--;
}
arr[j + 1] = arr[i];  // now reads a corrupted value

// Correct: save key first, before the loop touches anything
int key = arr[i];
int j = i - 1;
while (j >= 0 && arr[j] > key) {
    arr[j + 1] = arr[j];
    j--;
}
arr[j + 1] = key;
```

### Mistake 3: Missing `j >= 0` boundary check in insertion sort

```cpp
// Wrong: accesses arr[-1] when key is smaller than all previous elements
while (arr[j] > key) {

// Correct: short-circuit evaluation prevents out-of-bounds access
while (j >= 0 && arr[j] > key) {
```

C++ short-circuits `&&`: if `j >= 0` is false, `arr[j]` is never evaluated.

### Mistake 4: Off-by-one in placement after insertion sort shift loop

```cpp
// Wrong: j has been decremented one past the insertion point; arr[j] is too far left
arr[j] = key;

// Correct: the hole is always at j+1 after the loop exits
arr[j + 1] = key;
```

After the while loop body runs its last iteration, `j--` decrements `j` one step past the insertion slot. The correct index is always `j + 1`.

### Mistake 5: Using `>=` in insertion sort or bubble sort comparisons (breaks stability)

```cpp
// Wrong: swaps/shifts equal elements, destroying stability
while (j >= 0 && arr[j] >= key) {  // insertion sort: shifts past equal elements
if (arr[j] >= arr[j + 1]) { ... }  // bubble sort: swaps equal elements

// Correct:
while (j >= 0 && arr[j] > key) {
if (arr[j] > arr[j + 1]) { ... }
```

### Mistake 6: Inner loop of selection sort starting from 0 instead of i+1

```cpp
// Wrong: re-scans the already-sorted prefix; also may find a "minimum" in the sorted region
for (int j = 0; j < n; j++) {

// Correct: unsorted region starts at i+1
for (int j = i + 1; j < n; j++) {
```

### Mistake 7: Swapping inside the selection sort inner loop instead of tracking `minIdx`

```cpp
// Wrong: performs multiple swaps per pass, corrupting the sorted prefix
for (int j = i + 1; j < n; j++) {
    if (arr[j] < arr[i]) swap(arr[i], arr[j]);  // not selection sort

// Correct: track the minimum index; swap once, outside the inner loop
int minIdx = i;
for (int j = i + 1; j < n; j++) {
    if (arr[j] < arr[minIdx]) minIdx = j;
}
swap(arr[i], arr[minIdx]);
```

---

## 12. Interview Patterns and Extensions

### High-Frequency Interview Questions

| Question | Core point to make |
|---|---|
| Why is selection sort always O(n²)? | Inner loop cannot short-circuit; minimum is only known after scanning all remaining elements |
| Why is bubble sort O(n) best case? | Swap flag detects a zero-swap pass; that proves the array is sorted; outer loop breaks |
| Which of these are stable and why? | Bubble and insertion: yes. Selection: no. Explain the mechanism, not just the label |
| Why prefer insertion sort over bubble sort in practice? | Fewer comparisons on average, one write per shift vs three per swap, better cache behavior |
| How does insertion sort behave on nearly-sorted data? | O(n + k) where k = inversion count; optimal among comparison-based O(n²) sorts |
| Where is insertion sort used in production? | Base case of C++ introsort and Python/Java Timsort for sub-arrays below ~16–32 elements |
| What is the relationship between bubble sort swaps and inversions? | They are equal — each adjacent swap eliminates exactly one inversion |
| Can selection sort be made stable? | Yes, by rotating instead of swapping, at the cost of O(n) writes per pass |

### Connections to Harder Problems

**Inversion count:** The brute-force approach is O(n²) — directly implements the bubble/insertion sort logic. The optimal solution is O(n log n) via modified merge sort. Understanding the inversion interpretation of bubble sort is the conceptual bridge.

**Nearly-k-sorted arrays:** If each element is at most k positions from its final position, the array has at most O(nk) inversions. Insertion sort runs in O(nk). Using a min-heap of size k+1, the optimal solution runs in O(n log k). The insertion sort insight motivates the heap approach.

**Minimum swaps to sort an array:** Related to selection sort. The minimum number of swaps to sort a permutation equals n minus the number of cycles in the permutation's cycle decomposition. Computable in O(n) or O(n log n).

**Heapsort:** Conceptually "selection sort with the right data structure." Selection sort finds the minimum in O(n) using a linear scan. Replacing the unsorted suffix with a min-heap reduces the find-minimum step to O(log n), giving O(n log n) total. Heapsort is selection sort accelerated by a heap.

**Shell sort:** An extension of insertion sort that uses a gap sequence to perform insertion sort on interleaved sub-sequences. Elements far from their final position are moved there quickly in early passes with large gaps. Final pass (gap=1) is standard insertion sort on a nearly-sorted array. Achieves better than O(n²) average behavior without extra memory.

**Binary insertion sort:** The inner loop of insertion sort does two things: find the insertion position, and shift elements. Since the prefix `arr[0..i-1]` is sorted, binary search finds the insertion position in O(log i) comparisons instead of O(i). However, the shift step still costs O(i) writes, so total time is still O(n²). Binary insertion sort is useful when **comparisons are expensive** (complex objects, network calls) but writes are cheap.

```cpp
void binaryInsertionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int lo = 0, hi = i - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            if (arr[mid] > key) hi = mid - 1;
            else lo = mid + 1;
        }
        // lo is the insertion point; shift arr[lo..i-1] right
        for (int j = i - 1; j >= lo; j--) {
            arr[j + 1] = arr[j];
        }
        arr[lo] = key;
    }
}
// Comparisons: O(n log n).  Shifts: O(n²).  Total time: O(n²).
```

---

## 13. Quick Revision Cheat Sheet

This section is self-contained for night-before revision.

---

### Core Invariants

| Algorithm | What is guaranteed after k passes |
|---|---|
| Selection Sort | `arr[0..k-1]` contains the k smallest elements in sorted order |
| Bubble Sort | `arr[n-k..n-1]` contains the k largest elements in sorted order |
| Insertion Sort | `arr[0..k]` is a sorted permutation of the first k+1 elements |

---

### Selection Sort

**Strategy:** find minimum of unsorted suffix, swap to front.

```cpp
for (int i = 0; i < n - 1; i++) {
    int minIdx = i;
    for (int j = i + 1; j < n; j++)
        if (arr[j] < arr[minIdx]) minIdx = j;
    if (minIdx != i) swap(arr[i], arr[minIdx]);
}
```

- Always O(n²) comparisons, O(n) swaps — no adaptivity.
- Not stable (long-range swap moves past equal elements).
- Only sort where swap count is O(n) in all cases — preferred when writes are expensive.

---

### Bubble Sort

**Strategy:** bubble maximum to end via adjacent swaps; break on zero-swap pass.

```cpp
for (int i = 0; i < n - 1; i++) {
    bool swapped = false;
    for (int j = 0; j < n - 1 - i; j++) {
        if (arr[j] > arr[j + 1]) { swap(arr[j], arr[j+1]); swapped = true; }
    }
    if (!swapped) break;
}
```

- Best O(n) (sorted), worst O(n²).
- Stable (strict `>` condition; equal elements never swapped).
- Adaptive (early termination).
- Total swaps = number of inversions in input.

---

### Insertion Sort

**Strategy:** extract element, shift sorted prefix right until insertion point found.

```cpp
for (int i = 1; i < n; i++) {
    int key = arr[i];         // save before overwriting
    int j = i - 1;
    while (j >= 0 && arr[j] > key) { arr[j+1] = arr[j]; j--; }
    arr[j + 1] = key;         // j+1 is always the correct slot
}
```

- Best O(n) (sorted), worst O(n²). O(n + k) for k inversions.
- Stable (strict `>` stops at equal elements).
- Adaptive. Preferred for nearly-sorted data and small arrays.
- Total shifts = number of inversions in input.
- Recursive version: T(n) = T(n-1) + O(n) = O(n²); O(n) stack space.

---

### Complexity Reference

| Algorithm | Best | Worst | Space | Stable | Adaptive |
|---|---|---|---|---|---|
| Selection Sort | O(n²) | O(n²) | O(1) | No | No |
| Bubble Sort (opt) | O(n) | O(n²) | O(1) | Yes | Yes |
| Insertion Sort (iter) | O(n) | O(n²) | O(1) | Yes | Yes |
| Insertion Sort (recur) | O(n) | O(n²) | O(n) | Yes | Yes |

---

### Stability Reference

| Stable | Not Stable |
|---|---|
| Bubble Sort | Selection Sort |
| Insertion Sort | Quick Sort |
| Merge Sort | Heap Sort |
| Counting Sort | — |

---

### Inversion Count Connections

- 0 inversions → sorted → O(n) for bubble and insertion.
- n(n-1)/2 inversions → reverse sorted → O(n²) for all three.
- Bubble sort swap count = insertion sort shift count = inversion count of input.
- Selection sort is blind to inversions: always O(n²) comparisons.

---

### Production Context

- C++ `std::sort` (introsort): switches to insertion sort for sub-arrays ≤ ~16 elements.
- Python `list.sort()` and Java `Arrays.sort()` for objects (Timsort): uses insertion sort for runs ≤ ~32–64 elements.
- Reason: low constant factor, cache-friendly sequential access, zero overhead for tiny n.

---

### Key Interview One-Liners

- **Selection sort is always O(n²):** must confirm the minimum by scanning everything — inner loop cannot exit early.
- **Bubble sort best case is O(n):** a zero-swap pass proves the array is sorted; the swap flag captures this.
- **Insertion sort is O(n) on sorted input:** the while condition is false immediately for every element; zero shifts occur.
- **Insertion sort beats bubble sort in practice:** shifting costs one write per step; swapping costs three.
- **Selection sort minimizes writes:** at most n-1 swaps regardless of input — critical for write-expensive storage.
- **Stability in multi-key sort:** sort by secondary key first (stably), then primary key (stably) — ties in primary are broken correctly by secondary only if both sorts are stable.
