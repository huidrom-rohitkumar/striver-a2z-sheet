
**Resources:**
[GFG Row Max 1s](https://www.geeksforgeeks.org/problems/row-with-max-1s0023/1) |
[LC 74 Search 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) |
[LC 240 Search 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/) |
[LC 1901 Find Peak II](https://leetcode.com/problems/find-a-peak-element-ii/) |
[GFG Median Row-Wise](https://www.geeksforgeeks.org/problems/median-in-a-row-wise-sorted-matrix1527/1) |
[Striver Row Max 1s](https://youtu.be/SCz-1TtYxDI) |
[Striver LC 74](https://youtu.be/JXU4Akft7yk) |
[Striver LC 240](https://youtu.be/9ZbB397jU4k) |
[Striver LC 1901](https://youtu.be/nGGp5XBzC4g) |
[Striver GFG Median](https://youtu.be/Q9wXgdxJq48)

---

## Table of Contents

1. [Core Mental Model — Search Space Reduction in 2D](#1-core-mental-model--search-space-reduction-in-2d)
2. [Index Mapping: 2D to 1D and Back](#2-index-mapping-2d-to-1d-and-back)
3. [Comparison of Matrix Structures — The Critical Distinction Table](#3-comparison-of-matrix-structures--the-critical-distinction-table)
4. [Problem 1 — Row with Maximum 1s (GFG)](#4-problem-1--row-with-maximum-1s-gfg)
5. [Problem 2 — Search a 2D Matrix (LC 74)](#5-problem-2--search-a-2d-matrix-lc-74)
6. [Problem 3 — Search a 2D Matrix II (LC 240)](#6-problem-3--search-a-2d-matrix-ii-lc-240)
7. [Problem 4 — Find a Peak Element II (LC 1901)](#7-problem-4--find-a-peak-element-ii-lc-1901)
8. [Problem 5 — Median in Row-Wise Sorted Matrix (GFG)](#8-problem-5--median-in-row-wise-sorted-matrix-gfg)
9. [Why the Staircase Search is Correct — Formal Argument](#9-why-the-staircase-search-is-correct--formal-argument)
10. [Why Binary Search on Value Space Finds the Median — Formal Argument](#10-why-binary-search-on-value-space-finds-the-median--formal-argument)
11. [Edge Cases Master Table](#11-edge-cases-master-table)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Complexity Cheat Sheet](#13-complexity-cheat-sheet)
14. [Interview Reference — Patterns and Applications](#14-interview-reference--patterns-and-applications)
15. [Quick Revision Cheat Sheet](#15-quick-revision-cheat-sheet)

---

## 1. Core Mental Model — Search Space Reduction in 2D

In 1D binary search, every comparison discards half the remaining elements. In 2D, the same principle applies but the "half" is now a submatrix or a count in the value range. The challenge is identifying which structural property of the matrix allows systematic elimination.

The five problems in this set use four distinct strategies:

| Strategy | Problem | What is eliminated per step |
|---|---|---|
| Staircase from corner — track best row | GFG Row Max 1s | One full row or one column position |
| Flatten to 1D; standard binary search | LC 74 | Half of `m*n` elements |
| Staircase from top-right corner | LC 240 | One full row or one full column |
| Binary search on columns; max of mid-column | LC 1901 | Half the columns |
| Binary search on value space; `upper_bound` count | GFG Median | Half the value range |

Each strategy works because the matrix's sorted property provides a **monotone predicate** at each decision point. The predicate differs per problem. Identifying the correct predicate is the interview skill — not memorizing code.

**The one rule that prevents all category errors:** Before writing any code, identify which sorted property the problem guarantees and look it up in the table in Section 3. The wrong algorithm for the wrong structure is a common and fatal mistake.

---

## 2. Index Mapping: 2D to 1D and Back

For an `m x n` matrix (m rows, n columns), flat index `k` maps to:
```
row = k / n
col = k % n
```

And conversely, cell `(i, j)` maps to flat index `k = i * n + j`.

```
Matrix (m=3, n=4):        Flat indices:
[  1  3  5  7 ]           0  1  2  3
[ 10 11 16 20 ]           4  5  6  7
[ 23 30 34 60 ]           8  9 10 11

k=6 → row=6/4=1, col=6%4=2 → mat[1][2] = 16
mat[2][1] = 30 → k = 2*4 + 1 = 9
```

**Critical:** divide by the number of **columns** (`n`), not the number of rows. When `m != n`, using `m` instead of `n` gives wrong results.

This mapping is the entire trick for LC 74. A matrix satisfying "first element of each row > last element of previous row" is **isomorphic** to a 1D sorted array. Binary search on the flat index directly gives O(log(m*n)).

---

## 3. Comparison of Matrix Structures — The Critical Distinction Table

This table is the most important reference in this file. Misidentifying the matrix structure leads to applying the wrong algorithm.

| Matrix property | Sorted globally? | Optimal search strategy | Complexity |
|---|---|---|---|
| Rows sorted; first of row > last of previous (LC 74) | Yes | Flatten to 1D binary search | O(log(m*n)) |
| Rows sorted + columns sorted; NOT globally sorted (LC 240) | No | Staircase from top-right or bottom-left | O(m+n) |
| Binary rows (0s then 1s); rows sorted (GFG Row Max 1s) | No need | Staircase from top-right, track `ansRow` | O(m+n) |
| Rows sorted only; columns not necessarily sorted (GFG Median) | No | Binary search on value space | O(32 * n log m) |
| Arbitrary; no two adjacent cells equal (LC 1901) | No | Binary search on columns; find max in mid-column | O(m log n) |

**Why LC 74 cannot be solved by staircase and why LC 240 cannot be solved by flattening:**
- LC 74's globally sorted property makes flattening lossless. Staircase also works but costs O(m+n) instead of O(log(mn)) — strictly worse.
- LC 240's matrix is not globally sorted. Flattening destroys the ordering: the flat sequence `[1,4,7,11,2,5,8,12,...]` is unsorted. Applying binary search on it produces wrong answers.

---

## 4. Problem 1 — Row with Maximum 1s (GFG)

**Problem:** Given a 2D binary matrix where each row is sorted (0s followed by 1s), find the index of the row with the maximum number of 1s. If no row has a 1, return `-1`. If multiple rows tie, return the first one.
**Link:** [GFG Row with Max 1s](https://www.geeksforgeeks.org/problems/row-with-max-1s0023/1)
**Constraints:** `1 <= n, m <= 10^3`. Only 0s and 1s. Rows sorted.

**Examples:**
- `mat = [[0,0,1,1],[0,1,1,1],[1,1,1,1],[0,0,0,0]]` → `2`
- `mat = [[0,0],[0,0]]` → `-1`

### Approach 1 — Brute Force

```cpp
int rowWithMax1s(vector<vector<int>>& mat) {
    int n = mat.size(), m = mat[0].size();
    int maxOnes = 0, ansRow = -1;
    for (int i = 0; i < n; i++) {
        int cnt = 0;
        for (int j = 0; j < m; j++) cnt += mat[i][j];
        if (cnt > maxOnes) { maxOnes = cnt; ansRow = i; }
    }
    return ansRow;
}
```

Time: O(n*m). Space: O(1). Ignores sorted property.

### Approach 2 — Binary Search (lower\_bound) per Row

**Intuition:** Since each row is sorted, the count of 1s equals `m - lower_bound(row, 1)`. Run `lower_bound` for every row and track the maximum count.

```cpp
int rowWithMax1s(vector<vector<int>>& mat) {
    int n = mat.size(), m = mat[0].size();
    int maxOnes = 0, ansRow = -1;
    for (int i = 0; i < n; i++) {
        int firstOne = (int)(lower_bound(mat[i].begin(), mat[i].end(), 1) - mat[i].begin());
        int cnt = m - firstOne;
        if (cnt > maxOnes) { maxOnes = cnt; ansRow = i; }
    }
    return ansRow;
}
```

Time: O(n log m). Space: O(1).

### Approach 3 — Staircase from Top-Right (Optimal)

**Intuition:** Start at the top-right corner `(0, m-1)`. Column `col` tracks the leftmost 1 seen across all rows examined so far — the row owning that leftmost 1 has the most 1s (since rows are sorted). Moving further left means more 1s.

- If `mat[row][col] == 1`: record `ansRow = row` and move left (`col--`) to look for a 1 even further left.
- If `mat[row][col] == 0`: this row has no 1 at column `col` or earlier, so it cannot beat the current best. Move down (`row++`).

```cpp
int rowWithMax1s(vector<vector<int>>& mat) {
    int n = mat.size(), m = mat[0].size();
    int row = 0, col = m - 1;
    int ansRow = -1;
    while (row < n && col >= 0) {
        if (mat[row][col] == 1) {
            ansRow = row;
            col--;
        } else {
            row++;
        }
    }
    return ansRow;
}
```

**Dry run:** `mat = [[0,0,1,1],[0,1,1,1],[1,1,1,1],[0,0,0,0]]`

```
Indices:   col=0 col=1 col=2 col=3
Row 0:       0     0     1     1
Row 1:       0     1     1     1
Row 2:       1     1     1     1
Row 3:       0     0     0     0

Start: row=0, col=3

Step 1: mat[0][3]=1 → ansRow=0, col=2
Step 2: mat[0][2]=1 → ansRow=0, col=1
Step 3: mat[0][1]=0 → row=1
Step 4: mat[1][1]=1 → ansRow=1, col=0
Step 5: mat[1][0]=0 → row=2
Step 6: mat[2][0]=1 → ansRow=2, col=-1
col<0 → exit

ansRow = 2
```

Time: O(n + m) — at most `n` downward moves and `m` leftward moves. Space: O(1).

| Scenario | Expected | Reason |
|---|---|---|
| No 1s anywhere | `-1` | `ansRow` never updated |
| All 1s | `0` | Row 0 encountered first at rightmost column |
| Last row has most 1s | Last row index | Staircase reaches it after traversing earlier rows |

---

## 5. Problem 2 — Search a 2D Matrix (LC 74)

**Problem:** `m x n` matrix with: (1) each row sorted ascending, (2) first element of each row strictly greater than last element of previous row. Return `true` if `target` exists, else `false`. O(log(m*n)) required.
**Link:** [LC 74](https://leetcode.com/problems/search-a-2d-matrix/)
**Constraints:** `1 <= m, n <= 100`, `-10^4 <= mat[i][j], target <= 10^4`.

**Examples:**
- `mat = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `target = 3` → `true`
- Same matrix, `target = 13` → `false`

### Approach 1 — Brute Force

Scan every element. Time: O(m*n).

### Approach 2 — Two-Pass Binary Search (row then column)

Find the candidate row by binary searching the first column for the last row whose first element is `<= target`. Then binary search within that row.

```cpp
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
    int lo = 0, hi = m - 1, row = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (matrix[mid][0] <= target) { row = mid; lo = mid + 1; }
        else hi = mid - 1;
    }
    if (row == -1) return false;
    lo = 0; hi = n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (matrix[row][mid] == target) return true;
        else if (matrix[row][mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

Time: O(log m + log n) = O(log(m*n)). Space: O(1). Correct but uses two passes.

### Approach 3 — Flatten to 1D Binary Search (Optimal, Single Pass)

**Intuition:** Property (2) means the elements, read row by row left to right, form a single sorted sequence. Binary search the flat index space `[0, m*n-1]`. Convert flat index `k` to `(k/n, k%n)` at each step.

```cpp
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
    int lo = 0, hi = m * n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n];
        if (val == target) return true;
        else if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
```

**Dry run — target found:** `mat = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `target = 3`

```
m=3, n=4, total 12 elements.
Flat layout: [1, 3, 5, 7, 10, 11, 16, 20, 23, 30, 34, 60]

Iter 1: lo=0, hi=11, mid=5 → mat[5/4][5%4] = mat[1][1] = 11 > 3 → hi=4
Iter 2: lo=0, hi=4,  mid=2 → mat[2/4][2%4] = mat[0][2] = 5  > 3 → hi=1
Iter 3: lo=0, hi=1,  mid=0 → mat[0/4][0%4] = mat[0][0] = 1  < 3 → lo=1
Iter 4: lo=1, hi=1,  mid=1 → mat[1/4][1%4] = mat[0][1] = 3 == 3 → true
```

**Dry run — target absent:** `target = 13`

```
Iter 1: lo=0, hi=11, mid=5  → mat[1][1]=11 < 13 → lo=6
Iter 2: lo=6, hi=11, mid=8  → mat[2][0]=23 > 13 → hi=7
Iter 3: lo=6, hi=7,  mid=6  → mat[1][2]=16 > 13 → hi=5
lo=6 > hi=5 → false
```

| Scenario | Expected | Reason |
|---|---|---|
| Single element, matches | `true` | `mid=0`, direct match |
| Target smaller than all | `false` | `lo` exceeds `hi` before match |
| Target larger than all | `false` | Same |

Time: O(log(m*n)). Space: O(1).

---

## 6. Problem 3 — Search a 2D Matrix II (LC 240)

**Problem:** `m x n` matrix where: (1) each row is sorted ascending left to right, (2) each column is sorted ascending top to bottom. The matrix is **not** globally sorted. Return `true` if `target` exists.
**Link:** [LC 240](https://leetcode.com/problems/search-a-2d-matrix-ii/)
**Constraints:** `1 <= m, n <= 300`, `-10^9 <= mat[i][j] <= 10^9`.

**Examples:**
- `mat = [[1,4,7,11,15],[2,5,8,12,19],[3,6,9,16,22],[10,13,14,17,24],[18,21,23,26,30]]`, `target = 5` → `true`
- Same, `target = 20` → `false`

### Approach 1 — Brute Force

Time: O(m*n).

### Approach 2 — Binary Search per Row

```cpp
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    for (auto& row : matrix) {
        auto it = lower_bound(row.begin(), row.end(), target);
        if (it != row.end() && *it == target) return true;
    }
    return false;
}
```

Time: O(m log n). Uses row-sorted property but ignores column sorting.

### Approach 3 — Staircase from Top-Right (Optimal)

**Intuition:** Start at `(0, n-1)`. This corner has a unique property: everything to its left is smaller (row sorted), everything below it is larger (column sorted). So at any cell `(r, c)`:

- `mat[r][c] == target` → found.
- `mat[r][c] > target` → target cannot be in column `c` from row `r` downward (all such values are `>= mat[r][c] > target`). Move left: `c--`.
- `mat[r][c] < target` → target cannot be in row `r` at column `c` or to its left (all such values are `<= mat[r][c] < target`). Move down: `r++`.

Each step eliminates one full column or one full row. After at most `m + n - 1` steps, the matrix is exhausted.

```cpp
bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
    int r = 0, c = n - 1;
    while (r < m && c >= 0) {
        if (matrix[r][c] == target) return true;
        else if (matrix[r][c] > target) c--;
        else r++;
    }
    return false;
}
```

**Dry run — target found:** `target = 5`

```
       c=0  c=1  c=2  c=3  c=4
r=0  [  1    4    7   11   15 ]
r=1  [  2    5    8   12   19 ]

Start: r=0, c=4
Step 1: mat[0][4]=15 > 5 → c=3
Step 2: mat[0][3]=11 > 5 → c=2
Step 3: mat[0][2]=7  > 5 → c=1
Step 4: mat[0][1]=4  < 5 → r=1
Step 5: mat[1][1]=5 == 5 → return true
```

**Dry run — target absent:** `target = 20`

```
Start: r=0, c=4
mat[0][4]=15 < 20 → r=1
mat[1][4]=19 < 20 → r=2
mat[2][4]=22 > 20 → c=3
mat[2][3]=16 < 20 → r=3
mat[3][3]=17 < 20 → r=4
mat[4][3]=26 > 20 → c=2
mat[4][2]=23 > 20 → c=1
mat[4][1]=21 > 20 → c=0
mat[4][0]=18 < 20 → r=5
r=5 >= m=5 → false
```

**Why bottom-left also works:** Starting at `(m-1, 0)` provides the same property in reverse — right is larger, up is smaller. Move right when `< target`, move up when `> target`.

**Why top-left and bottom-right do not work:** From `(0, 0)`, both right and down lead to larger values — no way to decide direction. From `(m-1, n-1)`, both left and up lead to smaller values — same problem.

| Scenario | Expected | Reason |
|---|---|---|
| Single row | Uses only leftward moves | Column-sorted constraint trivially satisfied |
| Single column | Uses only downward moves | Row-sorted constraint trivially satisfied |
| Target at `(0,0)` | `true` | Staircase exhausts left moves and reaches it |
| Target at `(m-1,n-1)` | `true` | No left/up moves; down/right traversal leads to it |

Time: O(m + n). Space: O(1).

---

## 7. Problem 4 — Find a Peak Element II (LC 1901)

**Problem:** A 2D peak is an element strictly greater than all four neighbors. The grid is surrounded by virtual `-1` on all sides. No two adjacent cells are equal. Return any `[row, col]` that is a peak. O(m log n) required.
**Link:** [LC 1901](https://leetcode.com/problems/find-a-peak-element-ii/)
**Constraints:** `1 <= m, n <= 500`, `1 <= mat[i][j] <= 10^5`. No two adjacent cells equal.

**Examples:**
- `mat = [[1,4],[3,2]]` → `[0,1]` or `[1,0]` (4 and 3 are both valid peaks)
- `mat = [[10,20,15],[21,30,14],[7,16,32]]` → `[1,1]` (30) or `[2,2]` (32)

### Approach 1 — Brute Force

Check every cell against its four neighbors.

```cpp
vector<int> findPeakGrid(vector<vector<int>>& mat) {
    int m = mat.size(), n = mat[0].size();
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            int v = mat[i][j];
            bool ok = (i == 0   || mat[i-1][j] < v)
                   && (i == m-1 || mat[i+1][j] < v)
                   && (j == 0   || mat[i][j-1] < v)
                   && (j == n-1 || mat[i][j+1] < v);
            if (ok) return {i, j};
        }
    }
    return {-1, -1};
}
```

Time: O(m*n). Space: O(1).

### Approach 2 — Binary Search on Columns (Optimal)

**Intuition:** Binary search over column index. For a middle column `mid`, find `maxRow` — the row that contains the maximum element in column `mid`. Compare `mat[maxRow][mid]` with its left and right neighbors:

- **Both neighbors are smaller:** `mat[maxRow][mid]` is a 2D peak. It beats its top and bottom neighbors because it is the maximum of its column (all column elements `<= mat[maxRow][mid]`). It beats left and right by the comparison.
- **Left neighbor is larger:** A peak exists in `[lo, mid-1]`. The maximum of column `mid-1` is `>= mat[maxRow][mid-1] > mat[maxRow][mid]`. Following this argument rightward from column `mid-1` (comparing with left), the chain must terminate at a peak before the virtual `-1` left border.
- **Right neighbor is larger:** Symmetric; a peak exists in `[mid+1, hi]`.

```cpp
vector<int> findPeakGrid(vector<vector<int>>& mat) {
    int m = mat.size(), n = mat[0].size();
    int lo = 0, hi = n - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;

        // Find the row with the maximum value in column mid
        int maxRow = 0;
        for (int i = 1; i < m; i++)
            if (mat[i][mid] > mat[maxRow][mid]) maxRow = i;

        int leftVal  = (mid > 0)     ? mat[maxRow][mid - 1] : -1;
        int rightVal = (mid < n - 1) ? mat[maxRow][mid + 1] : -1;

        if (mat[maxRow][mid] > leftVal && mat[maxRow][mid] > rightVal) {
            return {maxRow, mid};
        } else if (mat[maxRow][mid] < leftVal) {
            hi = mid - 1;
        } else {
            lo = mid + 1;
        }
    }
    return {-1, -1};
}
```

**Dry run:** `mat = [[10,20,15],[21,30,14],[7,16,32]]`

```
       col=0  col=1  col=2
row=0  [  10    20    15 ]
row=1  [  21    30    14 ]
row=2  [   7    16    32 ]

lo=0, hi=2

Iter 1: mid=1 (column 1)
  Column 1: [20, 30, 16] → maxRow=1 (mat[1][1]=30)
  leftVal  = mat[1][0] = 21
  rightVal = mat[1][2] = 14
  30 > 21 ✓, 30 > 14 ✓ → return [1, 1]
```

**Dry run:** `mat = [[1,4],[3,2]]`

```
lo=0, hi=1

Iter 1: mid=0 (column 0)
  Column 0: [1, 3] → maxRow=1 (mat[1][0]=3)
  leftVal  = -1 (mid=0)
  rightVal = mat[1][1] = 2
  3 > -1 ✓, 3 > 2 ✓ → return [1, 0]
```

**Symmetry:** Binary search on rows instead of columns yields O(n log m). Use whichever dimension is smaller for the inner linear scan.

Time: O(m log n). Space: O(1).

| Scenario | Expected | Reason |
|---|---|---|
| Single element | `[0,0]` | Surrounded by virtual `-1`; `mat[0][0] > -1` |
| Strictly increasing left-to-right rows | Rightmost column contains peak | Binary search pushes `lo` rightward |
| Peak at (0,0) or (m-1,n-1) | That corner | Column max may be at boundary; virtual `-1` handles it |

---

## 8. Problem 5 — Median in Row-Wise Sorted Matrix (GFG)

**Problem:** Given a row-wise sorted matrix `mat[n][m]` where `n*m` is always odd, find the median.
**Link:** [GFG Median Row-Wise](https://www.geeksforgeeks.org/problems/median-in-a-row-wise-sorted-matrix1527/1)
**Constraints:** `1 <= n, m <= 500`, `1 <= mat[i][j] <= 10^5`. `n*m` is odd.

**Examples:**
- `mat = [[1,3,5],[2,6,9],[3,6,9]]` → `5`
- `mat = [[1],[2],[3]]` → `2`

### Approach 1 — Brute Force: Flatten, Sort, Return Middle

```cpp
int median(vector<vector<int>>& mat, int n, int m) {
    vector<int> arr;
    for (auto& row : mat) for (int x : row) arr.push_back(x);
    sort(arr.begin(), arr.end());
    return arr[(n * m) / 2];
}
```

Time: O(n*m*log(n*m)). Space: O(n*m). Expensive for large inputs.

### Approach 2 — Binary Search on Value Space (Optimal)

**The key insight:** The median of an odd-size collection is the smallest value `x` such that the count of elements `<= x` is `>= (n*m)/2 + 1`. Binary search this value directly over `[min_val, max_val]`.

For each candidate `mid`, count elements `<= mid` efficiently: each row is sorted, so `upper_bound(row, mid)` gives the count in that row in O(log m). Summing over all rows gives the total count in O(n log m).

**Predicate monotonicity:** The function `count(x) = number of elements <= x` is non-decreasing in `x`. This gives a clean `F F F T T T` pattern. Binary search finds the leftmost `T`.

**Finding min and max:** `lo = min(first element of each row)`, `hi = max(last element of each row)`.

```cpp
int median(vector<vector<int>>& mat, int n, int m) {
    int lo = INT_MAX, hi = INT_MIN;
    for (int i = 0; i < n; i++) {
        lo = min(lo, mat[i][0]);
        hi = max(hi, mat[i][m - 1]);
    }

    int desired = (n * m) / 2 + 1;   // the median is the desired-th smallest element

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;

        int cnt = 0;
        for (int i = 0; i < n; i++)
            cnt += (int)(upper_bound(mat[i].begin(), mat[i].end(), mid) - mat[i].begin());

        if (cnt < desired) {
            lo = mid + 1;   // median is larger than mid
        } else {
            hi = mid;       // mid is a candidate; try smaller
        }
    }
    return lo;
}
```

**Dry run:** `mat = [[1,3,5],[2,6,9],[3,6,9]]`, n=3, m=3

```
Row 0: [1,3,5]  Row 1: [2,6,9]  Row 2: [3,6,9]
lo = min(1,2,3) = 1
hi = max(5,9,9) = 9
desired = 9/2+1 = 5

Flat sorted: [1,2,3,3,5,6,6,9,9], median = index 4 → value 5.

Iter 1: mid=5
  Row 0: upper_bound(5) = 3 (all of 1,3,5 <= 5)
  Row 1: upper_bound(5) = 1 (only 2 <= 5)
  Row 2: upper_bound(5) = 1 (only 3 <= 5)
  cnt=5 >= desired=5 → hi=5

Iter 2: mid=3
  Row 0: upper_bound(3) = 2 (1,3 <= 3)
  Row 1: upper_bound(3) = 1 (2 <= 3)
  Row 2: upper_bound(3) = 1 (3 <= 3)
  cnt=4 < desired=5 → lo=4

Iter 3: mid=4
  Row 0: upper_bound(4) = 2; Row 1: 1; Row 2: 1 → cnt=4 < 5 → lo=5

lo=5 == hi=5 → return 5
```

| Scenario | Expected | Reason |
|---|---|---|
| Single element | `mat[0][0]` | lo=hi from start |
| All elements same | That value | `lo=hi=value` immediately |
| `lo == hi` from start | `lo` | Value range is a single point |

Time: O(log(max-min) * n log m). Since values ≤ 10^5, `log(max-min) <= 17`, giving effectively O(17 * n * log m). Space: O(1).

---

## 9. Why the Staircase Search is Correct — Formal Argument

**Claim:** Starting from `(0, n-1)` in an `m x n` matrix with rows and columns individually sorted ascending, the staircase search (move left if `> target`, move down if `< target`) finds `target` if present and correctly returns false if absent.

**Proof by invariant:** Maintain: "if `target` exists in the matrix, it must lie in submatrix `mat[r..m-1][0..c]`."

*Base case:* Initially `r=0, c=n-1`. Submatrix = entire matrix. ✓

*Inductive step:*
- `mat[r][c] > target`: Column `c`, rows `r..m-1` all have values `>= mat[r][c] > target` (column sorted ascending). Target cannot be in column `c` at rows `r..m-1`. Eliminate column `c`: `c--`. Invariant maintained for `mat[r..m-1][0..c-1]`.
- `mat[r][c] < target`: Row `r`, columns `0..c` all have values `<= mat[r][c] < target` (row sorted ascending). Target cannot be in row `r` at columns `0..c`. Eliminate row `r`: `r++`. Invariant maintained for `mat[r+1..m-1][0..c]`.

*Termination:* Each step reduces submatrix size by at least one row or one column. Starting from `m*n`, the size decreases to 0 in at most `m+n-1` steps.

*Correctness:* When `mat[r][c] == target`, target is found. When the submatrix is empty (`r >= m` or `c < 0`), the invariant says target is absent. ∎

---

## 10. Why Binary Search on Value Space Finds the Median — Formal Argument

**Why the value-space search finds the correct value:**

Define `f(x) = count of elements in the matrix <= x`. `f` is non-decreasing (integers, sorted rows). The median is the smallest `x*` such that `f(x*) >= desired` where `desired = (n*m)/2 + 1`.

The binary search with `while (lo < hi)`, `cnt < desired → lo = mid+1`, `cnt >= desired → hi = mid` finds exactly this `x*`:
- When `mid < x*`: `f(mid) < desired` → `lo = mid+1`. Keeps `x*` in `[lo, hi]`.
- When `mid >= x*`: `f(mid) >= desired` → `hi = mid`. Keeps `x*` in `[lo, hi]`.
- When `lo == hi`: both point to `x*`.

**Why `x*` must be a value present in the matrix:**

`f(x*) >= desired` and `f(x*-1) < desired`. So at least one element equals `x*` (otherwise `f(x*) = f(x*-1)`, contradiction). Therefore `lo` at termination is guaranteed to be an actual matrix value, not a spurious value from the search range. ∎

**Why `desired = (n*m)/2 + 1` and not `(n*m)/2`:**

For `n*m = 9`, the median is the 5th smallest (1-indexed). `desired = 9/2 + 1 = 5`. Using `9/2 = 4` returns the 4th smallest, which is one below the median.

---

## 11. Edge Cases Master Table

| Scenario | Problem | Input | Expected | Reason |
|---|---|---|---|---|
| No 1s in matrix | GFG Row Max 1s | `[[0,0],[0,0]]` | `-1` | `ansRow` never updated |
| All 1s in every row | GFG Row Max 1s | `[[1,1],[1,1]]` | `0` | Row 0 found first when col reaches 0 |
| Single element, matches | LC 74 | `[[5]]`, `t=5` | `true` | mid=0, direct match |
| Target smaller than all | LC 74 | `[[3,5],[7,9]]`, `t=1` | `false` | lo exceeds hi before match |
| Target at `(0,0)` | LC 240 | `[[1,4],[2,5]]`, `t=1` | `true` | Staircase reaches it after leftward moves |
| Target at `(m-1,n-1)` | LC 240 | `[[1,4],[2,5]]`, `t=5` | `true` | Staircase reaches it via downward moves |
| Single row | LC 240 | `[[1,2,3,4]]`, `t=3` | `true` | Staircase uses only leftward moves |
| Single column | LC 240 | `[[1],[2],[3]]`, `t=2` | `true` | Staircase uses only downward moves |
| Single element, peak | LC 1901 | `[[5]]` | `[0,0]` | Virtual `-1` on all sides; 5 > -1 |
| Peak at left border | LC 1901 | `[[3,1],[2,4]]` | `[0,0]` or `[1,0]` | `mid=0`, leftVal=-1; column max beats right |
| Single element matrix | GFG Median | `[[7]]` | `7` | lo=hi=7 immediately |
| All elements same | GFG Median | `[[3,3],[3,3],[3,3]]` | `3` | All counts jump at exactly 3; lo=hi=3 |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1 — Flattening LC 240 (applying LC 74's trick to the wrong matrix)

```cpp
// WRONG for LC 240 — matrix is NOT globally sorted, flat sequence is unsorted
int val = matrix[mid / n][mid % n];   // traverses in wrong order
```

The flat sequence of LC 240's matrix is `[1, 4, 7, 11, ..., 2, 5, 8, ...]` — not sorted. Binary search on it gives wrong answers. Use the staircase instead.

### Mistake 2 — Starting staircase from the wrong corner

```cpp
// WRONG — from top-left, both right and down lead to larger values
int r = 0, c = 0;
if (mat[r][c] < target) c++;   // no way to know whether to go right or down
```

From `(0,0)`, a single comparison cannot tell you which dimension to discard. Only top-right `(0, n-1)` or bottom-left `(m-1, 0)` provide the required bidirectional elimination property.

### Mistake 3 — Wrong `desired` value for median

```cpp
// WRONG — returns the ((n*m)/2)-th element, which is one below the median
int desired = (n * m) / 2;

// CORRECT
int desired = (n * m) / 2 + 1;
```

For a 9-element collection, the median is the 5th smallest (1-indexed). `9/2 = 4` gives the wrong position.

### Mistake 4 — Using `lower_bound` instead of `upper_bound` for median count

```cpp
// WRONG — lower_bound counts elements < mid, missing elements equal to mid
cnt += (int)(lower_bound(mat[i].begin(), mat[i].end(), mid) - mat[i].begin());

// CORRECT — upper_bound counts elements <= mid
cnt += (int)(upper_bound(mat[i].begin(), mat[i].end(), mid) - mat[i].begin());
```

The median predicate requires counting elements `<= mid`. `upper_bound(mid)` gives the first index past `mid`, so `upper_bound(mid) - begin` is the count of elements `<= mid`. `lower_bound(mid) - begin` counts elements `< mid`, which incorrectly excludes all copies of `mid` itself.

### Mistake 5 — Off-by-one in LC 74's index mapping (using row count instead of column count)

```cpp
// WRONG — when m != n, this gives a wrong row/column calculation
int val = matrix[mid / m][mid % m];   // m is number of rows, should use n (columns)

// CORRECT
int n_cols = matrix[0].size();
int val = matrix[mid / n_cols][mid % n_cols];
```

Dividing by row count instead of column count shifts every access to the wrong cell when the matrix is non-square.

### Mistake 6 — Returning `maxRow` without checking left/right neighbors in LC 1901

```cpp
// WRONG — column max beats top and bottom, but may NOT beat left/right
int maxRow = argmax(column[mid]);
return {maxRow, mid};   // may not be a 2D peak
```

The column maximum is greater than all elements above and below it in that column, but its left and right neighbors are independent. Always compare with `mat[maxRow][mid-1]` and `mat[maxRow][mid+1]` before returning.

### Mistake 7 — Using `while(lo <= hi)` in the median binary search

The median search uses `while(lo < hi)` with `hi = mid` (not `hi = mid - 1`) on the feasible branch. Using `while(lo <= hi)` with `hi = mid` causes an infinite loop when `lo == hi == mid`. The correct loop condition is `lo < hi`; at termination, `lo == hi` is the answer.

---

## 13. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| GFG Row Max 1s | Brute force | O(n*m) | O(1) |
| GFG Row Max 1s | `lower_bound` per row | O(n log m) | O(1) |
| GFG Row Max 1s | Staircase | O(n + m) | O(1) |
| LC 74 Search 2D Matrix | Brute force | O(m*n) | O(1) |
| LC 74 Search 2D Matrix | Two-pass binary search | O(log m + log n) | O(1) |
| LC 74 Search 2D Matrix | Flatten to 1D binary search | O(log(m*n)) | O(1) |
| LC 240 Search 2D Matrix II | Brute force | O(m*n) | O(1) |
| LC 240 Search 2D Matrix II | Binary search per row | O(m log n) | O(1) |
| LC 240 Search 2D Matrix II | Staircase | O(m + n) | O(1) |
| LC 1901 Find Peak II | Brute force | O(m*n) | O(1) |
| LC 1901 Find Peak II | Binary search on columns | O(m log n) | O(1) |
| GFG Median Matrix | Flatten + sort | O(nm log(nm)) | O(nm) |
| GFG Median Matrix | Binary search on value space | O(32 * n log m) | O(1) |

Note: `log(m*n) = log m + log n`, so the two-pass binary search for LC 74 is asymptotically equivalent to the single-pass flatten approach.

---

## 14. Interview Reference — Patterns and Applications

### Decision table: which algorithm for which structure

| Property observed | Correct approach |
|---|---|
| Rows sorted; first of row > last of previous | Flatten to 1D binary search |
| Rows AND columns sorted; NOT globally sorted | Staircase from top-right or bottom-left |
| Binary rows sorted; need row with most 1s | Staircase from top-right; track `ansRow` |
| Rows sorted only; need median (odd total size) | Binary search on value space with `upper_bound` count |
| 2D grid; no two adjacent equal; need any peak | Binary search on columns; return max of mid-column |

### Problems this set enables

- **LC 378 — Kth Smallest Element in Sorted Matrix:** Same binary search on value space as median, but `desired = k` and the matrix has both rows and columns sorted. Count using `upper_bound` per row identically.
- **LC 668 — Kth Smallest Number in Multiplication Table:** Same value-space binary search; count formula changes (closed-form instead of `upper_bound`).
- **LC 719 — Find Kth Smallest Pair Distance:** Binary search on value space; feasibility count uses two pointers.
- **LC 162 Find Peak (1D):** Base case of LC 1901 — same slope argument but no column scan needed.

### The staircase as a general 2D decision template

The staircase works whenever you have a corner where one direction strictly increases and the other strictly decreases. Applications beyond sorted matrices:

- Counting pairs `(i, j)` with `a[i] + b[j] == target` (two sorted arrays): start at `(0, n-1)`, staircase.
- Counting pairs satisfying a monotone condition over two sorted arrays.
- Closest pair in two sorted arrays.

---

## 15. Quick Revision Cheat Sheet

This section is fully self-contained for night-before revision.

**GFG Row Max 1s (optimal — staircase):**
```
Start (row=0, col=m-1).
mat[row][col]==1 → ansRow=row, col--
mat[row][col]==0 → row++
Return ansRow (or -1 if never updated)
```

**LC 74 (flatten):**
```
lo=0, hi=m*n-1
mid → mat[mid/n_cols][mid%n_cols]
Standard binary search. Return true/false.
```

**LC 240 (staircase):**
```
Start (r=0, c=n-1).
== target → return true
>  target → c--    (eliminate column c from r downward)
<  target → r++    (eliminate row r from c leftward)
Exit when r>=m or c<0 → return false
```

**LC 1901 (binary search on columns):**
```
lo=0, hi=n-1.
For mid column: find maxRow = argmax(column[mid])
If mat[maxRow][mid] > leftVal and > rightVal → return {maxRow, mid}
If < leftVal → hi = mid-1
Else         → lo = mid+1
leftVal  = (mid>0)   ? mat[maxRow][mid-1] : -1
rightVal = (mid<n-1) ? mat[maxRow][mid+1] : -1
```

**GFG Median (binary search on value space):**
```
lo = min of first elements, hi = max of last elements.
desired = n*m/2 + 1.
while (lo < hi):
    mid = lo + (hi-lo)/2
    cnt = sum over rows of upper_bound(row, mid) - row.begin()
    cnt < desired → lo = mid+1
    else          → hi = mid
return lo
```

**Index mapping for LC 74:** flat `k` → row = `k / n_cols`, col = `k % n_cols`.

**Count elements `<= x` in sorted row:** `upper_bound(row.begin(), row.end(), x) - row.begin()`.

**Corner choice for staircase:**
- Top-right `(0, n-1)`: left decreases, down increases. Move left on `>`, down on `<`.
- Bottom-left `(m-1, 0)`: right increases, up decreases. Move right on `<`, up on `>`.
- Top-left / bottom-right: both neighbors go same direction — cannot eliminate.

**Loop conditions:**
- LC 74, LC 240: `while (lo <= hi)` with early return.
- LC 1901: `while (lo <= hi)` with early return.
- GFG Median: `while (lo < hi)` with `hi = mid` on feasible branch (not `mid-1`).

**The three fatal category errors:**
1. Flattening LC 240 (only rows+columns sorted, not globally sorted).
2. Starting staircase from top-left or bottom-right.
3. Using `lower_bound` instead of `upper_bound` for median count.
