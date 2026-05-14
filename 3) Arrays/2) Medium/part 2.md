## Resources

- [Next Permutation — Striver (YouTube)](https://youtu.be/JDOXKqF60RQ)
- [Leaders in an Array (YouTube)](https://youtu.be/cHrH9CQ8pmY)
- [Longest Consecutive Sequence (YouTube)](https://youtu.be/oO5uLE7EUlM)
- [Set Matrix Zeroes (YouTube)](https://youtu.be/N0MgLvceX7M)
- [Rotate Image (YouTube)](https://youtu.be/Z0R2u6gd3GU)
- [Spiral Matrix (YouTube)](https://youtu.be/3Zv-s9UUrFM)
- [Subarray Sum Equals K (YouTube)](https://youtu.be/xvNwoz-ufXA)
- [LeetCode 31 — Next Permutation](https://leetcode.com/problems/next-permutation)
- [GFG — Leaders in an Array](https://www.geeksforgeeks.org/problems/leaders-in-an-array-1587115620/1)
- [LeetCode 128 — Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence)
- [LeetCode 73 — Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes)
- [LeetCode 48 — Rotate Image](https://leetcode.com/problems/rotate-image)
- [LeetCode 54 — Spiral Matrix](https://leetcode.com/problems/spiral-matrix)
- [LeetCode 560 — Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k)

---

## Table of Contents

1. [Core Mental Models in This Set](#1-core-mental-models-in-this-set)
2. [Next Permutation (LC 31)](#2-next-permutation-lc-31)
3. [Leaders in an Array](#3-leaders-in-an-array)
4. [Longest Consecutive Sequence (LC 128)](#4-longest-consecutive-sequence-lc-128)
5. [Set Matrix Zeroes (LC 73)](#5-set-matrix-zeroes-lc-73)
6. [Rotate Image (LC 48)](#6-rotate-image-lc-48)
7. [Spiral Matrix (LC 54)](#7-spiral-matrix-lc-54)
8. [Subarray Sum Equals K (LC 560)](#8-subarray-sum-equals-k-lc-560)
9. [The Prefix Sum Pattern — Unified Reference](#9-the-prefix-sum-pattern--unified-reference)
10. [The Matrix Manipulation Pattern — Unified Reference](#10-the-matrix-manipulation-pattern--unified-reference)
11. [Common Mistakes and Pitfalls](#11-common-mistakes-and-pitfalls)
12. [Interview Pattern Recognition](#12-interview-pattern-recognition)
13. [Complexity Cheat Sheet](#13-complexity-cheat-sheet)
14. [Quick Revision Cheat Sheet](#14-quick-revision-cheat-sheet)

---

## 1. Core Mental Models in This Set

Every problem here requires a non-obvious structural insight that collapses an intractable brute force into an elegant linear or near-linear solution. The pattern across all seven problems is the same: **find the hidden structure and exploit it directly.**

| Problem | Hidden Structure | Insight |
|---|---|---|
| Next Permutation | Non-increasing suffix is already maximum | Find the first violation from the right; make the minimal increase |
| Leaders in Array | Max-from-right determines leadership in O(1) | Right-to-left scan with running maximum |
| Longest Consecutive | Each sequence has a unique leftmost start | Only extend from starts (num-1 absent); each element touched once total |
| Set Matrix Zeroes | First row/col can serve as marker storage | Use matrix itself; handle first row/col state separately |
| Rotate Image | 90° CW rotation = Transpose + Row reversal | Decompose into two simple in-place operations |
| Spiral Matrix | Traversal = shrinking rectangular boundary | Maintain four boundary pointers; guard inner two traversals |
| Subarray Sum = K | sum(i,j) = prefix[j] - prefix[i] | Count how many earlier prefix sums equal prefix[j] - k |

---

## 2. Next Permutation (LC 31)

### Problem Statement

Given `nums`, rearrange it into the **next lexicographically greater permutation**. If the array is sorted descending (last permutation), rearrange to the first (sorted ascending). In-place, O(1) extra space.

```
[1,2,3]     →  [1,3,2]
[3,2,1]     →  [1,2,3]   (last permutation wraps to first)
[1,1,5]     →  [1,5,1]
[2,4,1,7,5,0]  →  [2,4,5,0,1,7]
```

### The Structural Observation

Scan from right to left. The suffix that is **non-increasing** is already in its maximum possible arrangement — no rearrangement of those elements can produce a larger permutation.

```
[2, 4, 1, 7, 5, 0]
         ^--------  non-increasing suffix: [7,5,0] — already maximum
     ^              pivot: index 2, value 1 (first position where sequence breaks)
```

To find the next permutation: increase the pivot by the smallest possible amount, then arrange the suffix in its minimum order.

### The Three-Step Algorithm

**Step 1 — Find the pivot:** Scan right to left. The first index `i` where `nums[i] < nums[i+1]` is the pivot.

**Step 2 — Find the successor:** In the suffix `nums[i+1..n-1]`, find the rightmost element strictly greater than `nums[i]`. Since the suffix is non-increasing, scanning right-to-left gives the rightmost (= smallest) element that is still greater than the pivot.

**Step 3 — Swap and reverse:** Swap pivot with successor. Reverse the suffix `nums[i+1..n-1]`. The suffix was non-increasing before the swap (and still is after — swapping with the rightmost larger element preserves this). Reversing it produces the minimum arrangement.

**Special case:** If no pivot exists (entire array is non-increasing), it is the last permutation. `pivot = -1`, step 2 is skipped, step 3 reverses from index 0 = entire array = first permutation. The code handles this automatically without a special branch.

```cpp
void nextPermutation(vector<int>& nums) {
    int n = nums.size(), pivot = -1;

    // Step 1: Find pivot (rightmost i where nums[i] < nums[i+1])
    for (int i = n - 2; i >= 0; i--) {
        if (nums[i] < nums[i + 1]) { pivot = i; break; }
    }

    // Step 2: Find rightmost successor > pivot value, swap
    if (pivot != -1) {
        for (int j = n - 1; j > pivot; j--) {
            if (nums[j] > nums[pivot]) { swap(nums[pivot], nums[j]); break; }
        }
    }

    // Step 3: Reverse the suffix (from pivot+1 to end)
    // If pivot == -1, this reverses the entire array (first permutation)
    reverse(nums.begin() + pivot + 1, nums.end());
}
// Time: O(n), Space: O(1)
```

### Dry Run — Standard Case

```
nums = [2, 4, 1, 7, 5, 0],  n=6

Step 1 (find pivot, scan right to left):
  i=4: nums[4]=5, nums[5]=0. 5<0? No.
  i=3: nums[3]=7, nums[4]=5. 7<5? No.
  i=2: nums[2]=1, nums[3]=7. 1<7? YES. pivot=2.

Step 2 (find rightmost successor > nums[2]=1):
  j=5: nums[5]=0 > 1? No.
  j=4: nums[4]=5 > 1? YES. swap(nums[2], nums[4]).
  Array after swap: [2, 4, 5, 7, 1, 0]

Step 3 (reverse suffix from index 3): [7,1,0] → [0,1,7]
  Array: [2, 4, 5, 0, 1, 7]

Result: [2,4,5,0,1,7]  ✓
```

### Dry Run — Already Descending (Last Permutation)

```
nums = [3, 2, 1]

Step 1: i=1: 2<1? No. i=0: 3<2? No. pivot=-1.
Step 2: skipped.
Step 3: reverse from index 0: [1, 2, 3]

Result: [1,2,3]  ✓
```

### Dry Run — Ascending (First Permutation)

```
nums = [1, 2, 3]

Step 1: i=1: nums[1]=2 < nums[2]=3. YES. pivot=1.
Step 2: j=2: nums[2]=3 > nums[1]=2. swap. [1, 3, 2]
Step 3: reverse suffix from index 2: [2] → [2] (one element)

Result: [1,3,2]  ✓
```

### Why j Scans from n-1 in Step 2

The suffix is non-increasing. Scanning from the right finds the **rightmost** element greater than the pivot, which is also the **smallest** such element in the suffix (because rightmost = earliest in the descending order). Swapping with the smallest valid successor gives the minimum possible increase.

### STL

```cpp
next_permutation(nums.begin(), nums.end());   // returns false if already last permutation
prev_permutation(nums.begin(), nums.end());   // previous permutation
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[1]` | `[1]` | Single element |
| `[1,1,5]` | `[1,5,1]` | Duplicates: pivot=1 (val 1), successor=index 2 (val 5) |
| `[2,3,1,3,3]` | `[2,3,3,1,3]` | Rightmost valid successor among duplicates |

---

## 3. Leaders in an Array

### Problem Statement

Return all **leaders** in `arr`. An element is a leader if it is **greater than or equal to all elements to its right**. The rightmost element is always a leader.

```
arr = [16, 17, 4, 3, 5, 2]  →  [17, 5, 2]
arr = [10, 22, 12, 3, 0, 6] →  [22, 12, 6]
arr = [3, 2, 1]             →  [3, 2, 1]   (descending: every element is a leader)
```

### Approach 1: Brute Force O(n²)

For each element, scan everything to its right. If nothing is larger, it is a leader.

```cpp
vector<int> leaders(vector<int>& arr) {
    int n = arr.size();
    vector<int> result;
    for (int i = 0; i < n; i++) {
        bool isLeader = true;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] > arr[i]) { isLeader = false; break; }
        }
        if (isLeader) result.push_back(arr[i]);
    }
    return result;
}
// Time: O(n^2), Space: O(1)
```

### Approach 2: Right-to-Left Scan with Running Maximum (Optimal)

**Insight:** `arr[i]` is a leader if and only if `arr[i] >= max(arr[i+1..n-1])`. The maximum to the right of index `i` is exactly the running maximum maintained while scanning right to left. One pass gives all leaders.

Since we find leaders right to left but want them left to right, reverse the result at the end.

```cpp
vector<int> leaders(vector<int>& arr) {
    int n = arr.size();
    vector<int> result;
    int maxFromRight = arr[n - 1];
    result.push_back(arr[n - 1]);   // rightmost is always a leader

    for (int i = n - 2; i >= 0; i--) {
        if (arr[i] >= maxFromRight) {
            result.push_back(arr[i]);
            maxFromRight = arr[i];
        }
    }

    reverse(result.begin(), result.end());
    return result;
}
// Time: O(n), Space: O(k) for k leaders in output
```

### Dry Run

```
arr = [16, 17, 4, 3, 5, 2],  n=6

maxFromRight=2, result=[2]

i=4: arr[4]=5.  5 >= 2? YES → result=[2,5],  maxFromRight=5
i=3: arr[3]=3.  3 >= 5? No
i=2: arr[2]=4.  4 >= 5? No
i=1: arr[1]=17. 17 >= 5? YES → result=[2,5,17], maxFromRight=17
i=0: arr[0]=16. 16 >= 17? No

After reverse: [17, 5, 2]  ✓
```

### The Key Insight: Reverse Traversal Converts Future to Past

Scanning from the right means the running maximum represents information about elements to the **right** of the current index — exactly what we need. The problem requires knowing the future (all elements to the right); traversing backward converts that future information into already-computed past information.

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[1, 2, 3]` | `[3]` | Only the max (rightmost) is a leader |
| `[3, 2, 1]` | `[3, 2, 1]` | Descending: every element is a leader |
| `[5]` | `[5]` | Single element |
| `[2, 2, 2]` | `[2, 2, 2]` | All equal: >= condition makes all leaders |

---

## 4. Longest Consecutive Sequence (LC 128)

### Problem Statement

Given an unsorted integer array, return the length of the longest consecutive sequence. You must write an **O(n) algorithm**.

```
[100, 4, 200, 1, 3, 2]      →  4   ([1,2,3,4])
[0, 3, 7, 2, 5, 8, 4, 6, 0, 1]  →  9   ([0..8])
```

### Why Sorting Does Not Satisfy the Requirement

Sorting gives O(n log n). The problem explicitly demands O(n), which forces a hash-based approach.

### Approach 1: Sort and Scan — O(n log n)

```cpp
int longestConsecutive(vector<int>& nums) {
    if (nums.empty()) return 0;
    sort(nums.begin(), nums.end());
    int longest = 1, current = 1;
    for (int i = 1; i < (int)nums.size(); i++) {
        if (nums[i] == nums[i-1]) continue;           // skip duplicates
        if (nums[i] == nums[i-1] + 1) current++;
        else current = 1;
        longest = max(longest, current);
    }
    return longest;
}
```

### Approach 2: Hash Set + Sequence-Start Detection (Optimal)

**The key insight:** A number `num` is the **start of a consecutive sequence** if and only if `num - 1` is **not** in the set. If we only begin counting from sequence starts, we process each sequence exactly once.

**Why this is O(n) and not O(n²):** Each element is visited in the inner while loop **at most once total** across all outer iterations. Since we only enter the while loop at sequence starts, every number from that sequence is consumed by exactly one call to the while loop. Total while-loop iterations across all `num` values = n.

```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> numSet(nums.begin(), nums.end());
    int longest = 0;

    for (int num : numSet) {                          // iterate over unique values
        if (!numSet.count(num - 1)) {                 // only start from sequence beginnings
            int currentNum = num;
            int currentLen = 1;

            while (numSet.count(currentNum + 1)) {
                currentNum++;
                currentLen++;
            }

            longest = max(longest, currentLen);
        }
    }

    return longest;
}
// Time: O(n) average, Space: O(n)
```

### Dry Run

```
nums = [100, 4, 200, 1, 3, 2]
numSet = {100, 4, 200, 1, 3, 2}

num=100: 99 in set? No → start. 101 in set? No. len=1. longest=1.
num=4:   3 in set? YES → skip (not a start).
num=200: 199 in set? No → start. 201? No. len=1.
num=1:   0 in set? No → start.
  while 2 in set? YES → currentNum=2, len=2
  while 3 in set? YES → currentNum=3, len=3
  while 4 in set? YES → currentNum=4, len=4
  while 5 in set? No → stop. longest=4.
num=3: 2 in set? YES → skip.
num=2: 1 in set? YES → skip.

Return 4  ✓
```

### Why Iterate Over `numSet` and Not `nums`

Iterating over `nums` would attempt to start the same sequence from duplicate values multiple times (though start-detection would skip them, it wastes iterations). Iterating over `numSet` guarantees each unique value is tried exactly once.

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[]` | `0` | Empty array |
| `[1]` | `1` | Single element |
| `[1,2,0,1]` | `3` | Duplicate 1; set deduplicates |
| `[-1,0,1]` | `3` | Negatives work |

---

## 5. Set Matrix Zeroes (LC 73)

### Problem Statement

Given an `m x n` matrix, if any element is `0`, set its **entire row and column** to `0`. In-place.

**The core difficulty:** You cannot zero elements as you scan — a new zero might incorrectly cause additional rows/columns to be zeroed. All rows/columns to be zeroed must be identified **before** zeroing begins.

### Approach 1: Extra Space — O(m+n)

Use two boolean arrays to record which rows and columns contain zeros.

```cpp
void setZeroes(vector<vector<int>>& matrix) {
    int m = matrix.size(), n = matrix[0].size();
    vector<bool> zeroRow(m, false), zeroCol(n, false);

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (matrix[i][j] == 0) { zeroRow[i] = true; zeroCol[j] = true; }

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (zeroRow[i] || zeroCol[j]) matrix[i][j] = 0;
}
// Time: O(m*n), Space: O(m+n)
```

### Approach 2: O(1) Space — Use First Row and Column as Markers

**Insight:** Instead of extra arrays, use `matrix[i][0]` to mark row `i` and `matrix[0][j]` to mark column `j`. The problem: `matrix[0][0]` is shared between row 0 and column 0. Use a separate boolean `firstColZero` to independently track whether column 0 should be zeroed.

**The order of operations is critical and must not be changed:**

1. Check if the first row originally contains any zero (`firstRowZero`).
2. Check if the first column originally contains any zero (`firstColZero`).
3. Scan interior `[1..m-1][1..n-1]`: for any zero, mark `matrix[i][0] = 0` (row marker) and `matrix[0][j] = 0` (col marker).
4. Use markers to zero out interior cells `[1..m-1][1..n-1]`.
5. If `firstRowZero`, zero the entire first row.
6. If `firstColZero`, zero the entire first column.

Steps 5 and 6 **must be last** — the first row and column contain markers that steps 1-4 depend on. Zeroing them earlier destroys those markers.

```cpp
void setZeroes(vector<vector<int>>& matrix) {
    int m = matrix.size(), n = matrix[0].size();

    // Step 1-2: Record original state of first row and first column
    bool firstRowZero = false, firstColZero = false;
    for (int j = 0; j < n; j++) if (matrix[0][j] == 0) firstRowZero = true;
    for (int i = 0; i < m; i++) if (matrix[i][0] == 0) firstColZero = true;

    // Step 3: Mark rows/cols using first row and first column as storage
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            if (matrix[i][j] == 0) {
                matrix[i][0] = 0;   // mark row i
                matrix[0][j] = 0;   // mark col j
            }
        }
    }

    // Step 4: Zero out interior cells based on markers
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            if (matrix[i][0] == 0 || matrix[0][j] == 0) matrix[i][j] = 0;

    // Step 5: Zero first row if it originally had a zero
    if (firstRowZero) for (int j = 0; j < n; j++) matrix[0][j] = 0;

    // Step 6: Zero first column if it originally had a zero
    if (firstColZero) for (int i = 0; i < m; i++) matrix[i][0] = 0;
}
// Time: O(m*n), Space: O(1)
```

### Dry Run

```
matrix = [[1,1,1],[1,0,1],[1,1,1]],  m=3, n=3

Steps 1-2: firstRowZero=false, firstColZero=false.

Step 3 (scan interior [1..2][1..2]):
  (1,1): matrix[1][1]=0 → mark matrix[1][0]=0, matrix[0][1]=0.
  Matrix: [[1,0,1],[0,0,1],[1,1,1]]

Step 4 (zero interior using markers):
  (1,1): matrix[1][0]=0 → zero. (1,2): matrix[1][0]=0 → zero.
  (2,1): matrix[0][1]=0 → zero.
  Matrix: [[1,0,1],[0,0,0],[1,0,1]]

Steps 5-6: firstRowZero=false, firstColZero=false → skip.

Result: [[1,0,1],[0,0,0],[1,0,1]]  ✓
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[[0]]` | `[[0]]` | Single zero cell |
| `[[1,0],[1,1]]` | `[[0,0],[1,0]]` | Zero in row 0 and col 1 |
| No zeros | Unchanged | Both booleans false, no markers set |

---

## 6. Rotate Image (LC 48)

### Problem Statement

Rotate an `n x n` matrix **90° clockwise** in-place.

```
[1,2,3]         [7,4,1]
[4,5,6]  →      [8,5,2]
[7,8,9]         [9,6,3]
```

### The Mathematical Mapping

After 90° clockwise rotation, the element at `(i, j)` moves to `(j, n-1-i)`.

```
(0,0)=1 → (0,2):  output[0][2]=1  ✓
(0,1)=2 → (1,2):  output[1][2]=2  ✓
(1,0)=4 → (0,1):  output[0][1]=4  ✓
```

### Approach 1: Extra Matrix — O(n²) space

```cpp
void rotate(vector<vector<int>>& matrix) {
    int n = matrix.size();
    vector<vector<int>> temp(n, vector<int>(n));
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n; j++)
            temp[j][n-1-i] = matrix[i][j];
    matrix = temp;
}
// Violates in-place constraint.
```

### Approach 2: Transpose + Reverse Rows (Optimal, In-Place)

**Decomposition:** 90° clockwise = Transpose then reverse each row.

**Why this works:**
- After transpose: element originally at `(i,j)` is now at `(j,i)`.
- After reversing row `j`: element at `(j,i)` moves to `(j, n-1-i)`.
- Net effect: `(i,j) → (j, n-1-i)` — exactly the 90° clockwise mapping.

**Critical: transpose swaps only the upper triangle** (`j` starts at `i+1`). If `j` starts at 0, each pair gets swapped twice, reverting to the original.

```cpp
void rotate(vector<vector<int>>& matrix) {
    int n = matrix.size();

    // Step 1: Transpose (upper triangle only)
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)   // j starts at i+1, NOT 0
            swap(matrix[i][j], matrix[j][i]);

    // Step 2: Reverse each row
    for (int i = 0; i < n; i++)
        reverse(matrix[i].begin(), matrix[i].end());
}
// Time: O(n^2), Space: O(1)
```

### Dry Run

```
matrix = [[1,2,3],[4,5,6],[7,8,9]]

After Transpose (swap [i][j] with [j][i] for j>i):
  swap [0][1]↔[1][0]: 2↔4
  swap [0][2]↔[2][0]: 3↔7
  swap [1][2]↔[2][1]: 6↔8
  Result: [[1,4,7],[2,5,8],[3,6,9]]

After Reverse each row:
  Row 0: [1,4,7] → [7,4,1]
  Row 1: [2,5,8] → [8,5,2]
  Row 2: [3,6,9] → [9,6,3]

Final: [[7,4,1],[8,5,2],[9,6,3]]  ✓
```

### Other Rotation Variants

```
90° counter-clockwise  =  Transpose + Reverse each COLUMN
                        =  Reverse each row + Transpose

180°  =  (i,j) → (n-1-i, n-1-j)
       =  Reverse all rows top-to-bottom + Reverse each row
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[[1]]` | `[[1]]` | 1×1, no change |
| `[[1,2],[3,4]]` | `[[3,1],[4,2]]` | 2×2 |

---

## 7. Spiral Matrix (LC 54)

### Problem Statement

Return all elements of an `m x n` matrix in spiral (clockwise) order, starting from the top-left.

```
[[1,2,3],[4,5,6],[7,8,9]]  →  [1,2,3,6,9,8,7,4,5]
```

### The Shrinking Boundary Approach

Maintain four boundaries: `top`, `bottom`, `left`, `right`. Each complete traversal of one boundary shrinks it inward. Repeat until boundaries cross.

```
Direction order:
  Left → Right  along the top row    →  top++
  Top → Bottom  along the right col  →  right--
  Right → Left  along the bottom row →  bottom--   (guard: only if top <= bottom)
  Bottom → Top  along the left col   →  left++      (guard: only if left <= right)
```

**Why the guards on directions 3 and 4:** After traversing the top row and right column, there may be no rows or columns left (e.g., a single-row or single-column matrix). Without guards, we would traverse the bottom row or left column again, outputting elements twice.

```cpp
vector<int> spiralOrder(vector<vector<int>>& matrix) {
    int m = matrix.size(), n = matrix[0].size();
    int top = 0, bottom = m - 1, left = 0, right = n - 1;
    vector<int> result;

    while (top <= bottom && left <= right) {
        // Left → Right along top row
        for (int j = left; j <= right; j++) result.push_back(matrix[top][j]);
        top++;

        // Top → Bottom along right column
        for (int i = top; i <= bottom; i++) result.push_back(matrix[i][right]);
        right--;

        // Right → Left along bottom row (guard: still a row to traverse)
        if (top <= bottom) {
            for (int j = right; j >= left; j--) result.push_back(matrix[bottom][j]);
            bottom--;
        }

        // Bottom → Top along left column (guard: still a column to traverse)
        if (left <= right) {
            for (int i = bottom; i >= top; i--) result.push_back(matrix[i][left]);
            left++;
        }
    }

    return result;
}
// Time: O(m*n) — every element visited exactly once, Space: O(1) excluding output
```

### Detailed Dry Run — 3×3 Matrix

```
matrix = [[1,2,3],[4,5,6],[7,8,9]]
top=0, bottom=2, left=0, right=2

Iteration 1:
  Left→Right (top=0, j=0..2):   push 1,2,3. top=1.
  Top→Bottom (right=2, i=1..2): push 6,9.   right=1.
  Guard top<=bottom: 1<=2 YES. Right→Left (bottom=2, j=1..0): push 8,7. bottom=1.
  Guard left<=right: 0<=1 YES. Bottom→Top (left=0, i=1..1):  push 4.   left=1.

  State: top=1, bottom=1, left=1, right=1. result=[1,2,3,6,9,8,7,4]

Iteration 2:
  Left→Right (top=1, j=1..1):   push 5. top=2.
  Top→Bottom (right=1, i=2..1): nothing (top>bottom). right=0.
  Guard top<=bottom: 2<=1? No. Skip.
  Guard left<=right: 1<=0? No. Skip.

  top=2 > bottom=1 → loop ends.

result = [1,2,3,6,9,8,7,4,5]  ✓
```

### Dry Run — 1×4 Matrix

```
matrix = [[1,2,3,4]],  top=0, bottom=0, left=0, right=3

Iteration 1:
  Left→Right: push 1,2,3,4. top=1.
  Top→Bottom: i=1..0 → nothing. right=2.
  Guard top<=bottom: 1<=0? No. Skip.
  Guard left<=right: 0<=2? Yes. Bottom→Top: i=0..1 → nothing (bottom=0, top=1, loop empty). left=1.

  top=1 > bottom=0 → end.

result = [1,2,3,4]  ✓
```

### Spiral Matrix II — Generate (LC 59)

Fill an n×n matrix with 1 to n² in spiral order.

```cpp
vector<vector<int>> generateMatrix(int n) {
    vector<vector<int>> mat(n, vector<int>(n, 0));
    int top=0, bottom=n-1, left=0, right=n-1, num=1;
    while (top <= bottom && left <= right) {
        for (int j=left; j<=right; j++)  mat[top][j] = num++;  top++;
        for (int i=top; i<=bottom; i++)  mat[i][right] = num++;  right--;
        if (top<=bottom) { for (int j=right; j>=left; j--) mat[bottom][j] = num++;  bottom--; }
        if (left<=right) { for (int i=bottom; i>=top; i--) mat[i][left] = num++;  left++; }
    }
    return mat;
}
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[[1]]` | `[1]` | Single cell |
| `[[1],[2],[3]]` | `[1,2,3]` | Single column |
| `[[1,2,3]]` | `[1,2,3]` | Single row |
| `[[1,2],[3,4]]` | `[1,2,4,3]` | 2×2 |

---

## 8. Subarray Sum Equals K (LC 560)

### Problem Statement

Given `nums` and integer `k`, return the total number of **subarrays whose sum equals k**. Array may contain negative integers.

```
nums = [1,1,1], k=2   →  2   ([1,1] at [0,1] and [1,2])
nums = [1,2,3], k=3   →  2   ([3], [1,2])
nums = [-1,-1,1], k=0 →  1   ([-1,1])
```

### Why Sliding Window Fails

Sliding window requires that expanding the window increases the sum and shrinking decreases it — true only for non-negative arrays. With negative numbers, a longer window can have a smaller sum. The sliding window approach breaks entirely.

### The Prefix Sum Insight

Define `prefix[i]` = sum of `nums[0..i-1]` (with `prefix[0] = 0`).

Sum of subarray `nums[i..j]` = `prefix[j+1] - prefix[i]`.

For this to equal `k`:
```
prefix[j+1] - prefix[i] = k
prefix[i] = prefix[j+1] - k
```

At each index `j`, count how many earlier indices `i` have `prefix[i] = prefix[j+1] - k`. Maintain a **frequency map** of all prefix sums seen so far.

### Approach 1: Brute Force O(n²)

```cpp
int subarraySum(vector<int>& nums, int k) {
    int n = nums.size(), count = 0;
    for (int start = 0; start < n; start++) {
        int sum = 0;
        for (int end = start; end < n; end++) {
            sum += nums[end];
            if (sum == k) count++;
        }
    }
    return count;
}
```

### Approach 2: Prefix Sum + Hash Map (Optimal)

```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int, int> prefixCount;
    prefixCount[0] = 1;   // CRITICAL: empty prefix has sum 0, occurred once

    int sum = 0, count = 0;
    for (int x : nums) {
        sum += x;                               // running prefix sum
        int complement = sum - k;               // need this prefix to have appeared before
        if (prefixCount.count(complement)) {
            count += prefixCount[complement];
        }
        prefixCount[sum]++;                     // record AFTER querying
    }

    return count;
}
// Time: O(n), Space: O(n)
```

### Why `prefixCount[0] = 1` Is Critical

Consider `nums = [3], k = 3`. At index 0: `sum = 3`, `complement = 0`. If `prefixCount[0]` is not seeded as 1, we miss this subarray. The base case represents the empty prefix (the sum before index 0 begins), which exists exactly once. Any subarray starting from index 0 whose sum equals `k` corresponds to `prefix[j] - prefix[0] = k`, requiring `prefix[0] = 0` with count 1 in the map.

### Why We Query Before Inserting

```cpp
// Query first, then insert.
count += prefixCount[sum - k];
prefixCount[sum]++;
```

Inserting before querying would allow the current prefix sum to form a pair with itself. For `k = 0`, `sum - k = sum`, and inserting before querying would count a subarray from "index i to index i" — a zero-length subarray, which is invalid.

### Detailed Dry Run

```
nums = [1,1,1], k=2
prefixCount={0:1}, sum=0, count=0

x=1: sum=1. complement=1-2=-1. -1 in map? No.  map={0:1, 1:1}.
x=1: sum=2. complement=2-2=0.   0 in map? YES (count 1). count=1.  map={0:1,1:1,2:1}.
x=1: sum=3. complement=3-2=1.   1 in map? YES (count 1). count=2.  map={0:1,1:1,2:1,3:1}.

Return 2  ✓  ([1,1] at [0..1] and [1..2])
```

```
nums = [1,2,3], k=3
prefixCount={0:1}, sum=0, count=0

x=1: sum=1. comp=-2. No.  map={0:1,1:1}
x=2: sum=3. comp=0.  YES(1). count=1.  map={0:1,1:1,3:1}
x=3: sum=6. comp=3.  YES(1). count=2.  map={0:1,1:1,3:2}

Return 2  ✓  ([1,2] and [3])
```

```
nums = [0,0,0], k=0
prefixCount={0:1}, sum=0

x=0: sum=0. comp=0. YES(1). count=1. map={0:2}
x=0: sum=0. comp=0. YES(2). count=3. map={0:3}
x=0: sum=0. comp=0. YES(3). count=6. map={0:4}

Return 6  ✓  (every subarray sums to 0: (0),(0),(0),(0,0),(0,0),(0,0,0) = 6)
```

### Prefix Sum vs Longest Subarray — Key Difference

| Goal | What to store in map | Map value |
|---|---|---|
| Count subarrays with sum k | Frequency of each prefix sum | `prefixCount[sum]++` |
| Find longest subarray with sum k | First occurrence of each prefix sum | `if not in map: map[sum] = i` |

For counting, multiple subarrays can share the same right endpoint but have different left endpoints, all producing sum k. Every occurrence of `complement` contributes a valid subarray. For longest subarray, you want the leftmost previous occurrence to maximize length.

### Edge Cases

| Input | k | Output | Note |
|---|---|---|---|
| `[1,1,1]` | `2` | `2` | Two overlapping subarrays |
| `[0,0,0]` | `0` | `6` | Every subarray is valid |
| `[1,-1,1,-1]` | `0` | `4` | Multiple zero-sum subarrays |
| `[1]` | `0` | `0` | No subarray sums to 0 |

---

## 9. The Prefix Sum Pattern — Unified Reference

**Definition:**

```
prefix[0] = 0   (empty prefix — always seed this in the hash map!)
prefix[i] = prefix[i-1] + arr[i-1]

sum(arr[l..r]) = prefix[r+1] - prefix[l]
```

**The hash map extension:** To count pairs `(i, j)` where `prefix[j] - prefix[i] = target`, rewrite as: for each `j`, count how many `i` have `prefix[i] = prefix[j] - target`. Store prefix frequencies in a hash map for O(1) lookup per query.

### Problem Family

| Problem | Hash Map Key | Value Stored | LeetCode |
|---|---|---|---|
| Subarray Sum = K | Prefix sum | Frequency | 560 |
| Subarray Sum Divisible by K | Prefix sum mod k | Frequency | 974 |
| Longest Subarray with Sum K | Prefix sum | First occurrence index | — |
| Count Equal 0s and 1s | Replace 0→-1, prefix sum | Frequency | — |
| Subarray XOR = K | Prefix XOR | Frequency | 1442 |

### Template

```cpp
unordered_map<int, int> mp;
mp[identity] = 1;    // seed: identity value of the operation (0 for sum, 0 for XOR)
int running = identity;
int answer = 0;

for (int x : arr) {
    running = combine(running, x);       // update running value (sum: +=x; XOR: ^=x)
    answer += mp[running - target];      // count (for sum) or query (for XOR: running ^ target)
    mp[running]++;                       // record AFTER querying
}
```

---

## 10. The Matrix Manipulation Pattern — Unified Reference

### Index Transformations for Rotations

```
Original (i, j)  →  90° clockwise:         (j, n-1-i)
Original (i, j)  →  90° counter-clockwise: (n-1-j, i)
Original (i, j)  →  180°:                  (n-1-i, n-1-j)
```

### Decomposing Rotations into In-Place Operations

```
90° clockwise  =  Transpose (j from i+1) + Reverse each row
90° CCW        =  Transpose (j from i+1) + Reverse each column
               =  Reverse each row + Transpose
180°           =  Reverse each row + Reverse all rows top-to-bottom
```

### Spiral Traversal Template

```
top=0, bottom=m-1, left=0, right=n-1
while top<=bottom and left<=right:
  traverse top row left to right   →  top++
  traverse right col top to bottom →  right--
  IF top<=bottom: traverse bottom row right to left  →  bottom--
  IF left<=right: traverse left col bottom to top    →  left++
```

### Using the Matrix Itself as Marker Storage

When a problem asks to mark rows/columns without extra space:
1. Record the state of the first row and first column (two booleans) **before** any marking.
2. Use `matrix[i][0]` and `matrix[0][j]` as markers for rows and columns.
3. Process the interior first (`rows 1..m-1, cols 1..n-1`).
4. Handle the first row and first column **last** using the stored booleans.

---

## 11. Common Mistakes and Pitfalls

### Mistake 1: Next Permutation — `j` scanning from `pivot+1` instead of `n-1`

```cpp
// WRONG: finds the LEFTMOST element > pivot, not the rightmost
for (int j = pivot + 1; j < n; j++) {
    if (nums[j] > nums[pivot]) { swap(...); break; }
}
// For [1, 3, 2], pivot=0(val=1). j starts at 1, finds 3. Swaps: [3,1,2].
// Reverse suffix: [3,1,2]. WRONG. Should be [2,1,3].

// CORRECT: find the RIGHTMOST element > pivot (smallest in suffix)
for (int j = n - 1; j > pivot; j--) {
    if (nums[j] > nums[pivot]) { swap(...); break; }
}
// For [1,3,2]: j=2 finds 2>1. Swap: [2,3,1]. Reverse suffix [3,1]→[1,3]. Result: [2,1,3] ✓
```

### Mistake 2: Set Matrix Zeroes — Zeroing first row/column before step 4

```cpp
// WRONG: zero first row early, destroying markers
if (firstRowZero) for (int j=0; j<n; j++) matrix[0][j] = 0;  // too early!
// Now all matrix[0][j] are 0. In step 4, every column gets zeroed. Catastrophic.

// CORRECT: zero first row and column only after step 4 is complete.
```

### Mistake 3: Set Matrix Zeroes — Detecting `firstRowZero` after the marking loop

```cpp
// WRONG: check first row AFTER the marking loop
for (int i=1; i<m; i++) for (int j=1; j<n; j++) if (matrix[i][j]==0) matrix[0][j]=0;
bool firstRowZero = false;
for (int j=0; j<n; j++) if (matrix[0][j]==0) firstRowZero = true;
// matrix[0][j] may now be 0 from markers, not original zeros. Wrong!

// CORRECT: detect firstRowZero before ANY modifications.
```

### Mistake 4: Rotate Image — Transpose includes entire square (double-swap)

```cpp
// WRONG: j starts at 0 — each pair swapped twice (reverts to original)
for (int j = 0; j < n; j++) swap(matrix[i][j], matrix[j][i]);

// CORRECT: j starts at i+1 (upper triangle only, each pair swapped once)
for (int j = i + 1; j < n; j++) swap(matrix[i][j], matrix[j][i]);
```

### Mistake 5: Spiral Matrix — Missing guards on directions 3 and 4

```cpp
// WRONG: traverse bottom row without checking if there's still a bottom row
for (int j=right; j>=left; j--) result.push_back(matrix[bottom][j]);
bottom--;
// For a 1-row matrix: after traversing top row (top=1), top > bottom=0.
// This would re-traverse the first row elements. Output would double.

// CORRECT
if (top <= bottom) {
    for (int j=right; j>=left; j--) result.push_back(matrix[bottom][j]);
    bottom--;
}
```

### Mistake 6: Subarray Sum = K — Not seeding `prefixCount[0] = 1`

```cpp
// WRONG: empty map
unordered_map<int,int> prefixCount;
// For nums=[k], k=k: sum=k, complement=0. 0 not in map → count=0. Should be 1.

// CORRECT:
unordered_map<int,int> prefixCount;
prefixCount[0] = 1;
```

### Mistake 7: Subarray Sum = K — Inserting before querying

```cpp
// WRONG: insert first, then query
prefixCount[sum]++;
count += prefixCount[sum - k];
// For k=0: sum - k = sum. The current prefix is already in the map.
// At the first element: count incorrectly includes a zero-length subarray.

// CORRECT: query first, insert after
count += prefixCount[sum - k];
prefixCount[sum]++;
```

### Mistake 8: Longest Consecutive — O(n) lookup inside the while loop

```cpp
// WRONG: linear search for num+1 inside while loop → O(n^2)
while (find(nums.begin(), nums.end(), currentNum + 1) != nums.end()) { ... }

// CORRECT: use unordered_set for O(1) lookup
while (numSet.count(currentNum + 1)) { ... }
```

---

## 12. Interview Pattern Recognition

| Problem | Hidden Structure | Pattern Name |
|---|---|---|
| Next Permutation | Non-increasing suffix already maximum | Lexicographic suffix analysis |
| Leaders in Array | Max-from-right decides leadership | Reverse traversal with running optimum |
| Longest Consecutive | Only sequence starts need checking | Hash set with start detection |
| Set Matrix Zeroes | First row/col as dual-purpose storage | Matrix-as-marker encoding |
| Rotate Image | Rotation = two reversals | Decompose into simpler in-place operations |
| Spiral Matrix | Rectangular boundary shrinks each cycle | Boundary simulation |
| Subarray Sum = K | sum(i,j) = prefix[j] - prefix[i] | Prefix frequency hashing |

### Harder Problems Built on These Foundations

| Harder Problem | Foundation |
|---|---|
| LC 46/47 — Permutations | Next Permutation logic |
| LC 974 — Subarray Sum Divisible by K | Prefix mod k + hash map |
| LC 325 — Longest Subarray with Sum K | Prefix sum + first-occurrence map |
| LC 59 — Spiral Matrix II | Same boundary simulation |
| LC 2013 — Detect Squares | Matrix coordinate hashing |
| LC 1480 — Running Sum | Prefix sum base case |

---

## 13. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Next Permutation | Three-step algorithm | O(n) | O(1) |
| Leaders in Array | Brute force | O(n²) | O(1) |
| Leaders in Array | Right-to-left scan | O(n) | O(k) output |
| Longest Consecutive | Sort + scan | O(n log n) | O(1) |
| Longest Consecutive | Hash set + start detection | O(n) avg | O(n) |
| Set Matrix Zeroes | Extra row/col arrays | O(m*n) | O(m+n) |
| Set Matrix Zeroes | First row/col as markers | O(m*n) | O(1) |
| Rotate Image | Extra matrix | O(n²) | O(n²) |
| Rotate Image | Transpose + reverse rows | O(n²) | O(1) |
| Spiral Matrix | Boundary shrinking | O(m*n) | O(1) |
| Subarray Sum = K | Brute force | O(n²) | O(1) |
| Subarray Sum = K | Prefix sum + hash map | O(n) | O(n) |

---

## 14. Quick Revision Cheat Sheet

Self-contained for same-day interview revision.

---

### Next Permutation — Three Steps

```cpp
void nextPermutation(vector<int>& nums) {
    int n = nums.size(), pivot = -1;
    // Step 1: rightmost i where nums[i] < nums[i+1]
    for (int i = n-2; i >= 0; i--) if (nums[i] < nums[i+1]) { pivot = i; break; }
    // Step 2: rightmost j where nums[j] > nums[pivot], swap
    if (pivot != -1)
        for (int j = n-1; j > pivot; j--) if (nums[j] > nums[pivot]) { swap(nums[pivot], nums[j]); break; }
    // Step 3: reverse suffix from pivot+1
    reverse(nums.begin() + pivot + 1, nums.end());
    // pivot=-1 → reverses entire array (first permutation) ✓
}
```

---

### Leaders in Array

```cpp
vector<int> leaders(vector<int>& arr) {
    int n = arr.size();
    vector<int> res;
    int mx = arr[n-1];
    res.push_back(mx);
    for (int i = n-2; i >= 0; i--)
        if (arr[i] >= mx) { mx = arr[i]; res.push_back(mx); }
    reverse(res.begin(), res.end());
    return res;
}
// arr[i] is leader iff arr[i] >= max(arr[i+1..n-1])
// Scan right to left with running max. Reverse at end for left-to-right order.
```

---

### Longest Consecutive Sequence

```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> s(nums.begin(), nums.end());
    int best = 0;
    for (int num : s) {
        if (!s.count(num - 1)) {        // only start from sequence beginnings
            int len = 1;
            while (s.count(num + len)) len++;
            best = max(best, len);
        }
    }
    return best;
}
// Each element enters while loop at most once total → O(n)
```

---

### Set Matrix Zeroes — O(1) Space

```cpp
void setZeroes(vector<vector<int>>& matrix) {
    int m = matrix.size(), n = matrix[0].size();
    bool fr = false, fc = false;
    // Record first row/col state FIRST
    for (int j=0; j<n; j++) if (matrix[0][j]==0) fr=true;
    for (int i=0; i<m; i++) if (matrix[i][0]==0) fc=true;
    // Mark interior using first row/col
    for (int i=1; i<m; i++) for (int j=1; j<n; j++)
        if (matrix[i][j]==0) { matrix[i][0]=0; matrix[0][j]=0; }
    // Zero interior using markers
    for (int i=1; i<m; i++) for (int j=1; j<n; j++)
        if (matrix[i][0]==0 || matrix[0][j]==0) matrix[i][j]=0;
    // Zero first row and col LAST
    if (fr) for (int j=0; j<n; j++) matrix[0][j]=0;
    if (fc) for (int i=0; i<m; i++) matrix[i][0]=0;
}
```

---

### Rotate Image — Transpose + Reverse

```cpp
void rotate(vector<vector<int>>& matrix) {
    int n = matrix.size();
    for (int i=0; i<n; i++) for (int j=i+1; j<n; j++) swap(matrix[i][j], matrix[j][i]);
    for (int i=0; i<n; i++) reverse(matrix[i].begin(), matrix[i].end());
}
// 90° CW = Transpose (j from i+1, NOT 0) + Reverse each row
// 90° CCW = Transpose + Reverse each column
```

---

### Spiral Matrix — Shrinking Boundaries

```cpp
vector<int> spiralOrder(vector<vector<int>>& matrix) {
    int top=0, bottom=matrix.size()-1, left=0, right=matrix[0].size()-1;
    vector<int> res;
    while (top<=bottom && left<=right) {
        for (int j=left; j<=right; j++) res.push_back(matrix[top][j]); top++;
        for (int i=top; i<=bottom; i++) res.push_back(matrix[i][right]); right--;
        if (top<=bottom) { for (int j=right; j>=left; j--) res.push_back(matrix[bottom][j]); bottom--; }
        if (left<=right) { for (int i=bottom; i>=top; i--) res.push_back(matrix[i][left]); left++; }
    }
    return res;
}
// Guards on directions 3 and 4 prevent double-counting in non-square matrices.
```

---

### Subarray Sum = K — Prefix Sum + Map

```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int,int> mp;
    mp[0] = 1;        // seed: empty prefix sum = 0, occurred once
    int sum = 0, count = 0;
    for (int x : nums) {
        sum += x;
        count += mp[sum - k];   // query before insert
        mp[sum]++;              // insert after query
    }
    return count;
}
// sum(i,j) = prefix[j] - prefix[i] = k → need prefix[i] = prefix[j] - k
// Negative numbers: sliding window fails; prefix sum works.
// mp[0]=1 handles subarrays starting from index 0.
```

---

### Critical Invariants and Facts

```
Next Permutation:
  Non-increasing suffix is at its maximum — can only change the element just before it.
  j from n-1 (not pivot+1) finds the smallest valid successor.
  Reversing a non-increasing suffix gives the minimum arrangement.

Leaders: arr[i] >= maxFromRight ↔ leader. Scan right to left.

Longest Consecutive: start counting only when (num-1) is absent. O(n) total while-loop work.

Set Matrix Zeroes: record first row/col state first; zero them last.
  matrix[0][0] serves only row 0's marker; col 0 needs separate boolean.

Rotate 90° CW: (i,j) → (j, n-1-i). Decompose: Transpose (j>i) then reverse rows.

Spiral: 4 boundaries. Guards prevent double traversal when boundaries collapse asymmetrically.

Subarray Sum = K:
  seed mp[0]=1. query before insert. works with negatives. sliding window does not.
  count subarrays: store frequency. longest subarray: store first index.
```
