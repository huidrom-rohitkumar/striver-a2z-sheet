## Resources

- [Striver Lecture 1](https://youtu.be/i05Ju7AftcM) | [Lecture 2](https://youtu.be/bLGZhJlt4y0) | [Lecture 3](https://youtu.be/WBgsABoClE0) | [Lecture 4](https://youtu.be/FWAIf_EVUKE) | [Lecture 5](https://youtu.be/wuVwUK25Rfc)
- [LC 131 — Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/)
- [LC 79 — Word Search](https://leetcode.com/problems/word-search/)
- [LC 51 — N-Queens](https://leetcode.com/problems/n-queens/)
- [GFG — Rat in a Maze](https://www.geeksforgeeks.org/problems/rat-in-a-maze-problem/1)
- [LC 139 — Word Break](https://leetcode.com/problems/word-break/)
- [GFG — M-Coloring Problem](https://www.geeksforgeeks.org/problems/m-coloring-problem-1587115620/1)
- [LC 37 — Sudoku Solver](https://leetcode.com/problems/sudoku-solver/)
- [LC 282 — Expression Add Operators](https://leetcode.com/problems/expression-add-operators/)

---

## Table of Contents

1. [Core Mental Model](#1-core-mental-model)
2. [The Universal Backtracking Template](#2-the-universal-backtracking-template)
3. [Palindrome Partitioning (LC 131)](#3-palindrome-partitioning-lc-131)
4. [Word Search (LC 79)](#4-word-search-lc-79)
5. [N-Queens (LC 51)](#5-n-queens-lc-51)
6. [Rat in a Maze (GFG)](#6-rat-in-a-maze-gfg)
7. [Word Break (LC 139)](#7-word-break-lc-139)
8. [M-Coloring Problem (GFG)](#8-m-coloring-problem-gfg)
9. [Sudoku Solver (LC 37)](#9-sudoku-solver-lc-37)
10. [Expression Add Operators (LC 282)](#10-expression-add-operators-lc-282)
11. [Pruning: The Difference Between Correct and Fast](#11-pruning-the-difference-between-correct-and-fast)
12. [Mathematical Invariants](#12-mathematical-invariants)
13. [Backtracking vs Dynamic Programming](#13-backtracking-vs-dynamic-programming)
14. [Common Mistakes and Pitfalls](#14-common-mistakes-and-pitfalls)
15. [Complexity Cheat Sheet](#15-complexity-cheat-sheet)
16. [Interview Reference](#16-interview-reference)
17. [Quick Revision Cheat Sheet](#17-quick-revision-cheat-sheet)

---

## 1. Core Mental Model

**Backtracking is not brute force.** Brute force enumerates all candidates and checks validity at the end. Backtracking checks validity *incrementally* and **abandons a candidate the moment it violates a constraint**, without exploring any of its descendants in the decision tree. This is called **pruning**.

Every backtracking problem is a **decision tree traversal**. Each node is a partial solution. Each edge is a choice. Each leaf is either a complete valid solution or a dead end. Backtracking is DFS on this tree with early termination whenever the current partial solution cannot be extended to a valid complete solution.

Three components are always present:

- **Choice:** what decisions can be made at the current step (which character, which direction, which color, which operator, which digit).
- **Constraint:** which choices are currently valid (palindrome check, boundary + match, conflict, leading-zero rule).
- **Goal:** when is the current path a complete solution (all string consumed, all rows placed, destination reached, board full).

**The invariant at every recursive call:** the state passed in equals the state when the call returns. Every `makeChoice` must be paired with `undoChoice`. Violating this invariant is the single most common backtracking bug.

Three architectural choices follow from this model:

1. **Controlled state mutation:** pass state by reference (O(1) memory) and explicitly undo after each recursive call. Passing by value (copying entire board on every call) is correct but dramatically slower.
2. **Constraint propagation / early pruning:** validate constraints *before* recursing. This is what converts an exponential search into a practically fast one.
3. **Directional ordering for lexicographic output:** when output must be sorted (e.g., Rat in a Maze), explore choices in lexicographic order (D < L < R < U) so the tree physically generates answers in the correct order, avoiding a post-sort.

---

## 2. The Universal Backtracking Template

Every problem in this file is an instantiation of this template. The specifics of `isValid`, `makeChoice`, `undoChoice`, and the base case differ per problem.

```cpp
void backtrack(State& state, Results& results /*, problem params */) {
    // Base case: goal reached
    if (goalReached(state)) {
        results.push_back(buildResult(state));
        return;
    }

    for (Choice c : getChoices(state)) {
        if (isValid(state, c)) {       // constraint check (pruning)
            makeChoice(state, c);      // mutate state
            backtrack(state, results); // recurse
            undoChoice(state, c);      // restore state — mandatory
        }
    }
}
```

For problems requiring only one valid solution (Sudoku, Word Search, M-Coloring feasibility), the function returns `bool` and short-circuits immediately on the first success:

```cpp
bool backtrack(State& state) {
    if (goalReached(state)) return true;
    for (Choice c : getChoices(state)) {
        if (isValid(state, c)) {
            makeChoice(state, c);
            if (backtrack(state)) return true;   // short-circuit
            undoChoice(state, c);
        }
    }
    return false;
}
```

---

## 3. Palindrome Partitioning (LC 131)

**Problem:** Given string `s`, return all partitions such that every substring is a palindrome.

**Constraints:** `1 <= s.length() <= 16`, lowercase only.

**Examples:**
- `"aab"` → `[["a","a","b"],["aa","b"]]`
- `"a"` → `[["a"]]`

**Decision tree:** At each `start`, try every ending index `end >= start`. If `s[start..end]` is a palindrome, include it and recurse from `end+1`. When `start == n`, record the partition.

### Approach 1 — On-the-fly Palindrome Check

```cpp
class Solution {
    bool isPalin(const string& s, int lo, int hi) {
        while (lo < hi) {
            if (s[lo] != s[hi]) return false;
            lo++; hi--;
        }
        return true;
    }

    void backtrack(const string& s, int start,
                   vector<string>& curr, vector<vector<string>>& res) {
        if (start == (int)s.size()) {
            res.push_back(curr);
            return;
        }
        for (int end = start; end < (int)s.size(); end++) {
            if (isPalin(s, start, end)) {
                curr.push_back(s.substr(start, end - start + 1));
                backtrack(s, end + 1, curr, res);
                curr.pop_back();   // backtrack
            }
        }
    }
public:
    vector<vector<string>> partition(string s) {
        vector<vector<string>> res;
        vector<string> curr;
        backtrack(s, 0, curr, res);
        return res;
    }
};
```

**Time:** O(n · 2^n) — up to 2^(n-1) partitions, each taking O(n) to build. **Space:** O(n) recursion depth.

### Approach 2 — DP Palindrome Precomputation (Optimal)

The same substring may be checked multiple times in Approach 1. Precompute `dp[i][j]` = true if `s[i..j]` is a palindrome. Each check in the backtracking loop becomes O(1).

**DP recurrence:** `dp[i][j] = (s[i] == s[j]) && (j - i < 2 || dp[i+1][j-1])`.

```cpp
class Solution {
public:
    vector<vector<string>> partition(string s) {
        int n = s.size();
        vector<vector<bool>> dp(n, vector<bool>(n, false));
        for (int i = 0; i < n; i++) dp[i][i] = true;
        for (int len = 2; len <= n; len++)
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                if (s[i] == s[j]) dp[i][j] = (len == 2) || dp[i+1][j-1];
            }

        vector<vector<string>> res;
        vector<string> curr;
        function<void(int)> bt = [&](int start) {
            if (start == n) { res.push_back(curr); return; }
            for (int end = start; end < n; end++) {
                if (dp[start][end]) {
                    curr.push_back(s.substr(start, end - start + 1));
                    bt(end + 1);
                    curr.pop_back();
                }
            }
        };
        bt(0);
        return res;
    }
};
```

**Time:** O(n^2) for DP + O(n · 2^n) for backtracking = O(n · 2^n). **Space:** O(n^2) for DP table.

### Dry Run — `s = "aab"`, Approach 1

```
bt(start=0, curr=[])
  end=0: "a" is palindrome -> curr=["a"], bt(1)
    end=1: "a" is palindrome -> curr=["a","a"], bt(2)
      end=2: "b" is palindrome -> curr=["a","a","b"], bt(3)
        start==3 -> record ["a","a","b"]
      pop "b". curr=["a","a"].
    pop "a". curr=["a"].
    end=2: "ab" NOT palindrome -> skip.
  pop "a". curr=[].
  end=1: "aa" is palindrome -> curr=["aa"], bt(2)
    end=2: "b" is palindrome -> curr=["aa","b"], bt(3)
      start==3 -> record ["aa","b"]
    pop "b". curr=["aa"].
  pop "aa". curr=[].
  end=2: "aab" NOT palindrome -> skip.

Result: [["a","a","b"], ["aa","b"]]
```

---

## 4. Word Search (LC 79)

**Problem:** Given an m×n character grid and string `word`, return `true` if `word` exists as a path of sequentially adjacent cells (horizontal/vertical). The same cell may not be used more than once.

**Constraints:** `1 <= m, n <= 6`, `1 <= word.length <= 15`.

**Examples:**
- `board=[["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]]`, `word="ABCCED"` → `true`
- Same board, `word="ABCB"` → `false`

**Decision tree:** At each matched cell, try 4 directions for the next character. Mark the current cell visited (overwrite with `'#'`) before recursing; restore after. Short-circuit with logical OR — the moment any branch returns `true`, propagate immediately without exploring siblings.

```cpp
class Solution {
    int m, n;
    bool dfs(vector<vector<char>>& board, const string& word, int r, int c, int idx) {
        if (idx == (int)word.size()) return true;
        if (r < 0 || r >= m || c < 0 || c >= n) return false;
        if (board[r][c] != word[idx]) return false;

        char tmp = board[r][c];
        board[r][c] = '#';   // mark visited

        bool found = dfs(board, word, r+1, c, idx+1)
                  || dfs(board, word, r-1, c, idx+1)
                  || dfs(board, word, r, c+1, idx+1)
                  || dfs(board, word, r, c-1, idx+1);

        board[r][c] = tmp;   // restore (backtrack)
        return found;
    }
public:
    bool exist(vector<vector<char>>& board, string word) {
        m = board.size(); n = board[0].size();
        // Frequency pruning: reject immediately if board lacks required characters
        vector<int> freq(128, 0);
        for (auto& row : board) for (char c : row) freq[(int)c]++;
        for (char c : word) { if (--freq[(int)c] < 0) return false; }

        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++)
                if (dfs(board, word, r, c, 0)) return true;
        return false;
    }
};
```

**Why short-circuit with `||`:** C++ evaluates `||` left to right and stops at the first `true`. When any direction finds the complete word, `found` becomes `true` and the remaining three calls are never made. This is the critical optimization for feasibility problems.

**Frequency pruning:** Before starting any DFS, count character frequencies. If the word requires more of any character than the board contains, return `false` immediately. This eliminates entire executions, not just subtrees.

**Time:** O(m · n · 4^L) where L = word length. **Space:** O(L) recursion depth.

### Dry Run — `board=[["A","B"],["C","D"]]`, `word="ABD"`

```
Try (0,0): 'A'==word[0]. Mark '#'. idx=1.
  (1,0): 'C'!=word[1]='B'. false.
  (-1,0): out of bounds. false.
  (0,1): 'B'==word[1]. Mark '#'. idx=2.
    (1,1): 'D'==word[2]. Mark '#'. idx=3.
      idx==3==word.size(). return true.
    Restore board[1][1]='D'.
  Restore board[0][1]='B'.
Restore board[0][0]='A'.
return true.
```

---

## 5. N-Queens (LC 51)

**Problem:** Place n queens on an n×n chessboard such that no two attack each other (same row, column, or diagonal). Return all distinct solutions.

**Constraints:** `1 <= n <= 9`.

**Key observation:** No two queens share a row, so exactly one queen goes per row. The decision at each row is which column to use. Constraints are column conflicts and diagonal conflicts.

**Diagonal encodings:**
- Left diagonal (top-left to bottom-right): all cells on the same diagonal share a constant `row - col`. Offset by `n-1` to make non-negative: index = `row - col + n - 1`. Range: `0` to `2n-2`.
- Right diagonal (top-right to bottom-left): all cells share `row + col`. Range: `0` to `2(n-1)`.

**Why these formulas are correct:** Moving one step down-right on the left diagonal: `(r+1) - (c+1) = r - c`. The difference is invariant. Moving one step down-left on the right diagonal: `(r+1) + (c-1) = r + c`. The sum is invariant.

### Approach 1 — Brute Force O(n) Conflict Check

Scan all previously placed queens for column and diagonal conflicts. O(n) per candidate placement.

### Approach 2 — O(1) Conflict Check with Boolean Arrays (Optimal)

Maintain three arrays: `col[c]` (column c occupied), `diag1[row-col+n-1]` (left diagonal occupied), `diag2[row+col]` (right diagonal occupied). Each check and update is O(1).

```cpp
class Solution {
    int n;
    vector<bool> col, diag1, diag2;

    void backtrack(int row, vector<string>& board, vector<vector<string>>& res) {
        if (row == n) { res.push_back(board); return; }
        for (int c = 0; c < n; c++) {
            int d1 = row - c + n - 1;   // left diagonal index
            int d2 = row + c;            // right diagonal index
            if (col[c] || diag1[d1] || diag2[d2]) continue;

            board[row][c] = 'Q';
            col[c] = diag1[d1] = diag2[d2] = true;

            backtrack(row + 1, board, res);

            board[row][c] = '.';
            col[c] = diag1[d1] = diag2[d2] = false;
        }
    }
public:
    vector<vector<string>> solveNQueens(int n) {
        this->n = n;
        col.assign(n, false);
        diag1.assign(2*n-1, false);
        diag2.assign(2*n-1, false);
        vector<string> board(n, string(n, '.'));
        vector<vector<string>> res;
        backtrack(0, board, res);
        return res;
    }
};
```

**Time:** O(n!) — n choices for row 0, at most n-1 for row 1 (one column blocked), etc.; diagonal pruning reduces this further in practice. **Space:** O(n) for board and arrays.

### Dry Run — `n=4`, tracing to first solution

```
Row 0, c=0: d1=3, d2=0. All free. Place Q at (0,0). col[0]=diag1[3]=diag2[0]=true.
Row 1, c=0: col[0] occupied. Skip.
       c=1: d1=1-1+3=3. diag1[3] occupied. Skip.
       c=2: d1=1-2+3=2, d2=1+2=3. All free. Place Q at (1,2).
Row 2, c=0: col[0] occupied. Skip.
       c=1: d2=2+1=3. diag2[3] occupied. Skip.
       c=2: col[2] occupied. Skip.
       c=3: d1=2-3+3=2. diag1[2] occupied. Skip.
       Row 2 exhausted. Backtrack (remove Q at (1,2)).
       c=3: d1=1-3+3=1, d2=1+3=4. All free. Place Q at (1,3).
Row 2, c=1: d1=2-1+3=4, d2=2+1=3. diag2[4] occupied? diag2[1+3=4]=true. Skip.
       c=1: Wait, d2[row+col]=d2[2+1]=d2[3]. diag2[1+3]=diag2[4]=true (from Q at (1,3)). Skip.
       Actually: for Q at (1,3): diag2[1+3=4]=true. For (2,1): d2=2+1=3, free. d1=2-1+3=4, diag1[4]? free (diag1[1-3+3]=diag1[1]=false). col[1]=false. Place Q at (2,1).
Row 3, c=0: d2=3+0=3, diag2[3]=true (Q at (1,2) was backtracked... wait, Q at (1,3): d2=4. Q at (2,1): d2=3). Skip c=0: d2[3]=true. Skip.
       c=2: d1=3-2+3=4. diag1[4]? false. d2=3+2=5. diag2[5]? false. col[2]? false. Place Q at (3,2).
Row 4: row==n. Record board:
  Row 0: Q at col 0 -> "Q..."
  Row 1: Q at col 3 -> "...Q"
  Row 2: Q at col 1 -> ".Q.."
  Row 3: Q at col 2 -> "..Q."
  Solution: ["Q...","...Q",".Q..","..Q."]
```

---

## 6. Rat in a Maze (GFG)

**Problem:** An N×N binary matrix (1=open, 0=blocked). Rat starts at (0,0), must reach (N-1,N-1). Find all paths. Moves: Down (D), Left (L), Right (R), Up (U). No cell revisited within a path. Return paths in lexicographic order.

**Constraints:** `2 <= N <= 5`.

**Lexicographic ordering without post-sort:** Explore directions in DLRU order (alphabetically: D < L < R < U). The DFS tree generates paths in sorted order automatically.

**Visited marking:** Temporarily set the current cell to 0 before recursing (blocks the cell from being revisited in the current path). Restore to 1 on return.

```cpp
class Solution {
    string dirStr = "DLRU";
    int dr[4] = {1, 0, 0, -1};
    int dc[4] = {0, -1, 1, 0};

    void dfs(vector<vector<int>>& mat, int r, int c, int n,
             string& path, vector<string>& res) {
        if (r == n-1 && c == n-1) { res.push_back(path); return; }
        mat[r][c] = 0;   // mark visited
        for (int i = 0; i < 4; i++) {
            int nr = r + dr[i], nc = c + dc[i];
            if (nr >= 0 && nr < n && nc >= 0 && nc < n && mat[nr][nc] == 1) {
                path.push_back(dirStr[i]);
                dfs(mat, nr, nc, n, path, res);
                path.pop_back();   // backtrack
            }
        }
        mat[r][c] = 1;   // restore
    }
public:
    vector<string> findPath(vector<vector<int>>& mat) {
        vector<string> res;
        int n = mat.size();
        if (mat[0][0] == 0 || mat[n-1][n-1] == 0) return res;
        string path = "";
        dfs(mat, 0, 0, n, path, res);
        return res;
    }
};
```

**Time:** O(4^(n²)) — 4 direction choices, path depth up to n². **Space:** O(n²) recursion depth.

### Dry Run — `mat=[[1,0,0,0],[1,1,0,1],[1,1,0,0],[0,1,1,1]]`, n=4

```
dfs(0,0, path=""):
  mat[0][0]=0 (mark).
  D:(1,0)=1 open. path="D". dfs(1,0):
    mat[1][0]=0. D:(2,0)=1. path="DD". dfs(2,0):
      mat[2][0]=0. D:(3,0)=0 blocked. L:(2,-1) OOB. R:(2,1)=1. path="DDR". dfs(2,1):
        mat[2][1]=0. D:(3,1)=1. path="DDRD". dfs(3,1):
          mat[3][1]=0. R:(3,2)=1. path="DDRDR". dfs(3,2):
            mat[3][2]=0. R:(3,3)=1. path="DDRDRR". dfs(3,3):
              At destination. Record "DDRDRR".
            Restore mat[3][2]. pop 'R'.
          Restore mat[3][1]. pop 'R'.
        Restore mat[2][1]. pop 'D'.
      Restore mat[2][0]. pop 'R'.
    R:(1,1)=1. path="DR". dfs(1,1): ... -> records "DRDDRR".
    Restore mat[1][0].
  Restore mat[0][0].

Result: ["DDRDRR", "DRDDRR"]
```

---

## 7. Word Break (LC 139)

**Problem:** Given string `s` and dictionary `wordDict`, return `true` if `s` can be segmented into dictionary words (words may be reused).

**Constraints:** `1 <= s.length() <= 300`, `1 <= wordDict.length <= 1000`, `1 <= wordDict[i].length <= 20`.

**Examples:**
- `"leetcode"`, `["leet","code"]` → `true`
- `"catsandog"`, `["cats","dog","sand","and","cat"]` → `false`

This problem has three approaches illustrating a progression from backtracking to DP.

### Approach 1 — Naive Backtracking (Exponential, TLE)

```cpp
bool backtrack(const string& s, unordered_set<string>& dict, int start) {
    if (start == (int)s.size()) return true;
    for (int end = start + 1; end <= (int)s.size(); end++)
        if (dict.count(s.substr(start, end - start)) && backtrack(s, dict, end))
            return true;
    return false;
}
```

**Time:** O(2^n) — exponential, many repeated subproblems.

### Approach 2 — Memoized Backtracking

`backtrack(start)` may be called many times for the same `start`. Cache results.

```cpp
bool wordBreak(string s, vector<string>& wordDict) {
    int n = s.size();
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    vector<int> memo(n + 1, -1);   // -1: unvisited, 0: false, 1: true

    function<bool(int)> solve = [&](int start) -> bool {
        if (start == n) return true;
        if (memo[start] != -1) return memo[start];
        for (int end = start + 1; end <= n; end++)
            if (dict.count(s.substr(start, end - start)) && solve(end))
                return memo[start] = 1;
        return memo[start] = 0;
    };
    return solve(0);
}
```

**Time:** O(n² · L) where L = max word length. **Space:** O(n).

### Approach 3 — Bottom-Up DP (Optimal)

`dp[i]` = true if `s[0..i-1]` can be segmented. `dp[0] = true` (empty string). For each `i`, check all `j < i`: if `dp[j]` and `s[j..i-1]` is in the dictionary, then `dp[i] = true`.

```cpp
bool wordBreak(string s, vector<string>& wordDict) {
    int n = s.size();
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    vector<bool> dp(n + 1, false);
    dp[0] = true;
    for (int i = 1; i <= n; i++)
        for (int j = 0; j < i; j++)
            if (dp[j] && dict.count(s.substr(j, i - j))) { dp[i] = true; break; }
    return dp[n];
}
```

**Time:** O(n² · L). **Space:** O(n).

### Dry Run — `s="leetcode"`, `dict={"leet","code"}`

```
dp = [T,F,F,F,F,F,F,F,F]

i=1: j=0: dp[0]=T, s[0..0]="l" not in dict. dp[1]=F.
i=2,3: similarly, dp[2]=dp[3]=F.
i=4: j=0: dp[0]=T, s[0..3]="leet" IN dict. dp[4]=T. Break.
i=5: j=0: "leetc" no. j=4: dp[4]=T, s[4..4]="c" no. dp[5]=F.
i=6,7: dp[6]=dp[7]=F.
i=8: j=0: "leetcode" no. j=4: dp[4]=T, s[4..7]="code" IN dict. dp[8]=T.

return dp[8] = true.
```

**When to use which approach:** DP is expected in placement interviews for the feasibility question. Backtracking (Approach 2, memoized) is required for LC 140 (Word Break II) which asks to return all valid sentences.

---

## 8. M-Coloring Problem (GFG)

**Problem:** Given an undirected graph with V vertices and E edges, determine if it can be colored with at most m colors such that no two adjacent vertices share the same color.

**Constraints:** `1 <= V <= 20`, `1 <= m <= V`.

**Examples:**
- 4 vertices, edges (0,1),(1,2),(2,3),(3,0),(0,2), m=3 → `true`
- Triangle (3 vertices, all connected), m=2 → `false`

```cpp
class Solution {
    bool isSafe(int vertex, int color, vector<vector<int>>& adj, vector<int>& colorArr) {
        for (int neighbor : adj[vertex])
            if (colorArr[neighbor] == color) return false;
        return true;
    }

    bool solve(int vertex, int v, int m, vector<vector<int>>& adj, vector<int>& colorArr) {
        if (vertex == v) return true;
        for (int c = 1; c <= m; c++) {
            if (isSafe(vertex, c, adj, colorArr)) {
                colorArr[vertex] = c;
                if (solve(vertex + 1, v, m, adj, colorArr)) return true;
                colorArr[vertex] = 0;   // backtrack
            }
        }
        return false;
    }
public:
    bool graphColoring(int v, vector<pair<int,int>>& edges, int m) {
        vector<vector<int>> adj(v);
        for (auto& [u, w] : edges) { adj[u].push_back(w); adj[w].push_back(u); }
        vector<int> colorArr(v, 0);
        return solve(0, v, m, adj, colorArr);
    }
};
```

**Time:** O(m^V · V) — m choices per vertex, V vertices, O(degree) ≤ O(V) per `isSafe`. **Space:** O(V).

### Dry Run — Triangle, m=3

```
solve(0): c=1, safe. colorArr[0]=1.
  solve(1): c=1, adj[1]={0,2}, color[0]=1. Unsafe. c=2, safe. colorArr[1]=2.
    solve(2): c=1, adj[2]={1,0}. color[0]=1. Unsafe. c=2, color[1]=2. Unsafe. c=3, safe.
      solve(3): vertex==v. return true.
Coloring: 0->1, 1->2, 2->3. return true.
```

### Dry Run — Triangle, m=2

```
solve(0): c=1, safe. colorArr[0]=1.
  solve(1): c=2, safe. colorArr[1]=2.
    solve(2): c=1, adj has 0 (color=1). Unsafe. c=2, adj has 1 (color=2). Unsafe. return false.
  colorArr[1]=0. return false.
solve(0): c=2, safe. colorArr[0]=2.
  solve(1): c=1, safe. colorArr[1]=1.
    solve(2): c=1, adj has 1 (color=1). Unsafe. c=2, adj has 0 (color=2). Unsafe. return false.
  return false.
return false.
```

---

## 9. Sudoku Solver (LC 37)

**Problem:** Fill a partially-filled 9×9 Sudoku board. Empty cells are `'.'`. Each row, column, and 3×3 box must contain digits 1–9 exactly once. The input is always a uniquely solvable valid puzzle.

**Constraint checking — box index formula:** For cell (r, c), box index `b = (r/3)*3 + (c/3)`.

```
(0,0)→b=0  (0,3)→b=1  (0,6)→b=2
(3,0)→b=3  (3,3)→b=4  (3,6)→b=5
(6,0)→b=6  (6,3)→b=7  (6,6)→b=8
```

### Approach 1 — O(9) Validity Check Per Placement

For each empty cell, scan its row, column, and 3×3 box for conflicts. The 3×3 box scan uses a single loop with index arithmetic:

```cpp
bool isValid(vector<vector<char>>& board, int row, int col, char c) {
    for (int i = 0; i < 9; i++) {
        if (board[row][i] == c) return false;        // row
        if (board[i][col] == c) return false;        // column
        // 3x3 box: (row/3)*3 + i/3 gives row offset, (col/3)*3 + i%3 gives col offset
        if (board[(row/3)*3 + i/3][(col/3)*3 + i%3] == c) return false;
    }
    return true;
}
```

**Why the box formula works:** `i` runs 0–8. `i/3` cycles 0,0,0,1,1,1,2,2,2 — row offset within the box. `i%3` cycles 0,1,2,0,1,2,0,1,2 — column offset. Adding to the box's top-left corner `(row/3)*3, (col/3)*3` covers all 9 cells exactly once.

### Approach 2 — O(1) Constraint Check with Precomputed Arrays (Optimal)

Maintain `row[r][d]`, `col[c][d]`, `box[b][d]` boolean arrays. Each validity check is three array lookups.

```cpp
class Solution {
    bool row[9][10]={}, col[9][10]={}, box[9][10]={};

    bool solve(vector<vector<char>>& board) {
        for (int r = 0; r < 9; r++) {
            for (int c = 0; c < 9; c++) {
                if (board[r][c] != '.') continue;
                int b = (r/3)*3 + (c/3);
                for (int d = 1; d <= 9; d++) {
                    if (row[r][d] || col[c][d] || box[b][d]) continue;
                    board[r][c] = '0' + d;
                    row[r][d] = col[c][d] = box[b][d] = true;
                    if (solve(board)) return true;
                    board[r][c] = '.';
                    row[r][d] = col[c][d] = box[b][d] = false;
                }
                return false;   // no digit works: trigger backtrack
            }
        }
        return true;   // no empty cell found: board complete
    }
public:
    void solveSudoku(vector<vector<char>>& board) {
        for (int r = 0; r < 9; r++)
            for (int c = 0; c < 9; c++)
                if (board[r][c] != '.') {
                    int d = board[r][c]-'0', b = (r/3)*3+(c/3);
                    row[r][d] = col[c][d] = box[b][d] = true;
                }
        solve(board);
    }
};
```

**Why `return false` inside the inner loop:** When the nested loops find an empty cell and no digit 1–9 passes the validity check, returning `false` immediately signals the caller to backtrack. Without this, the function would continue scanning the rest of the board as if the cell were filled.

**Why the outer loop `return true`:** When the nested loops complete without encountering any `'.'`, every cell is filled — the board is solved. Return `true` to propagate the success up the call stack.

**Time:** O(9^k) where k = number of empty cells. In practice, constraint pruning makes this extremely fast. **Space:** O(1) for the constraint arrays (fixed size 9×10); O(k) call stack.

---

## 10. Expression Add Operators (LC 282)

**Problem:** Given digit string `num` and integer `target`, return all expressions formed by inserting `+`, `-`, or `*` between digits that evaluate to `target`. No leading zeros in multi-digit operands.

**Constraints:** `1 <= num.length <= 10`, `-2^31 <= target <= 2^31 - 1`.

**Examples:**
- `"123"`, `target=6` → `["1+2+3","1*2*3"]`
- `"232"`, `target=8` → `["2*3+2","2+3*2"]`

**The multiplication precedence problem — the key difficulty:** You cannot track a simple running sum when multiplication is involved. `"1+2*3"` evaluates to 7, not 9. When inserting `* val` after the current total `calc`, you must "undo" the last addition and reapply it multiplicatively.

**Derivation:** Suppose the last operand was `tail` added to reach `calc`. After inserting `* val`:

```
old: calc = prev + tail
new: calc = prev + tail * val
     calc = (calc - tail) + tail * val
```

So: `new_calc = calc - tail + tail * val`, `new_tail = tail * val`.

For subtraction, `tail` was stored as negative: `calc += val` records `tail = val`; `calc -= val` records `tail = -val`. Then `* val` applied to the subtraction case: `new_calc = calc - (-val) + (-val)*val` — the formula is uniform.

```cpp
class Solution {
    void backtrack(const string& num, long target, int idx,
                   long calc, long tail, string& expr,
                   vector<string>& res) {
        if (idx == (int)num.size()) {
            if (calc == target) res.push_back(expr);
            return;
        }
        for (int len = 1; idx + len <= (int)num.size(); len++) {
            // Leading zero guard: break (not continue) — no longer prefix is valid
            if (len > 1 && num[idx] == '0') break;

            string part = num.substr(idx, len);
            long val = stoll(part);
            int savedLen = expr.size();

            if (idx == 0) {
                // First operand: no operator
                expr += part;
                backtrack(num, target, idx+len, val, val, expr, res);
                expr.resize(savedLen);
            } else {
                // '+' operator
                expr += '+'; expr += part;
                backtrack(num, target, idx+len, calc+val, val, expr, res);
                expr.resize(savedLen);

                // '-' operator
                expr += '-'; expr += part;
                backtrack(num, target, idx+len, calc-val, -val, expr, res);
                expr.resize(savedLen);

                // '*' operator: undo last tail, apply multiplication
                expr += '*'; expr += part;
                backtrack(num, target, idx+len,
                          calc - tail + tail*val, tail*val, expr, res);
                expr.resize(savedLen);
            }
        }
    }
public:
    vector<string> addOperators(string num, int target) {
        vector<string> res;
        string expr = "";
        backtrack(num, (long)target, 0, 0, 0, expr, res);
        return res;
    }
};
```

**Why `break` not `continue` for leading zeros:** If `num[idx] == '0'`, then every multi-character substring starting at `idx` would have a leading zero. Since `len > 1` implies a multi-digit number, and no subsequent longer `len` will fix the leading zero issue (they all start at the same `idx`), `break` correctly rejects all remaining lengths. `continue` would wrongly skip the current `len` but try `len+1`, producing `"023"` as a valid number.

**Use `long` throughout:** The product of two 10-digit numbers exceeds `int` range. All of `val`, `calc`, `tail`, and `target` must be `long`.

**Time:** O(n · 3^n) — 3 operator choices at each of n-1 positions; n factor from substring extraction. **Space:** O(n) recursion depth.

### Dry Run — `"123"`, `target=6`

```
bt(idx=0, calc=0, tail=0, expr=""):
  len=1: part="1", val=1. First operand. expr="1".
    bt(idx=1, calc=1, tail=1, expr="1"):
      len=1: part="2", val=2.
        '+': expr="1+2". bt(idx=2, calc=3, tail=2):
          len=1: part="3", val=3.
            '+': calc=6. idx=3==n. calc==target. Record "1+2+3". ✓
            '-': calc=0. Not 6.
            '*': calc=3-2+2*3=7. Not 6.
        '-': expr="1-2". bt(idx=2, calc=-1, tail=-2): no hits.
        '*': expr="1*2". bt(idx=2, calc=2, tail=2):
          len=1: part="3", val=3.
            '*': calc=2-2+2*3=6. Record "1*2*3". ✓
      len=2: part="23", val=23.
        '+': calc=24. Not 6. '-': calc=-22. '*': calc=23. None hit 6.
  len=2: part="12". bt(idx=2, calc=12, tail=12):
    '+': 15. '-': 9. '*': 36. None hit 6.
  len=3: part="123". calc=123. Not 6.

Result: ["1+2+3", "1*2*3"]
```

---

## 11. Pruning: The Difference Between Correct and Fast

Backtracking without pruning degenerates to brute force. The quality of pruning separates an acceptable solution from a fast one.

| Problem | Pruning technique | Benefit |
|---|---|---|
| Palindrome Partitioning | Recurse only when `s[start..end]` is a palindrome | Skip non-palindrome prefixes before any recursion |
| Palindrome Partitioning (DP) | Precompute all `dp[i][j]` in O(n²) | Each palindrome check becomes O(1) during backtracking |
| Word Search | In-place cell marking (`'#'`) | Prevents cycles; no auxiliary visited array |
| Word Search | Character frequency check before DFS | Reject impossible words in O(n+m) before any DFS |
| Word Search | Short-circuit `||` in direction checks | First successful branch aborts all siblings immediately |
| N-Queens | O(1) conflict check via col/diag arrays | Replaces O(n) board scan with 3 array lookups |
| N-Queens | One queen per row constraint | Eliminates entire class of invalid placements upfront |
| Rat in a Maze | DLRU ordering | No post-sort needed; output is lexicographic by construction |
| Word Break | Memoize `solve(start)` | Eliminates exponential repeated subproblems |
| Sudoku | Row/col/box tracking arrays | O(1) validity check; no sub-grid scan |
| Sudoku | Immediate `return false` on blocked cell | Prunes entire subtree without exploring other cells |
| M-Coloring | Adjacency list in `isSafe` | Check only actual neighbors, not all V vertices |
| Expression Add Operators | `break` on leading zero | Cuts all multi-digit numbers starting with '0' in one step |
| Expression Add Operators | Running `calc` and `tail` | No re-evaluation of the expression from scratch per call |

---

## 12. Mathematical Invariants

### N-Queens Diagonal Formulas

**Right diagonal (row + col = constant):** Moving one step down-right on this diagonal: `(r+1) + (c+1)` — wait, this diagonal goes down-left. Moving down-left: `(r+1) + (c-1) = r + c`. The sum is invariant. Maximum value: `(n-1) + (n-1) = 2n-2`. Array size needed: `2n-1`.

**Left diagonal (row - col = constant):** Moving down-right: `(r+1) - (c+1) = r - c`. The difference is invariant. Range: `-(n-1)` to `(n-1)`. Offset by `n-1` to make non-negative: index = `r - c + n - 1`. Array size: `2n-1`.

### Sudoku 3×3 Box Traversal Formula

`board[(row/3)*3 + i/3][(col/3)*3 + i%3]` for `i` in 0..8:

- `(row/3)*3`: top-left row of the box (0, 3, or 6).
- `i/3`: row offset within the box — cycles 0,0,0,1,1,1,2,2,2.
- `(col/3)*3`: top-left column of the box (0, 3, or 6).
- `i%3`: column offset — cycles 0,1,2,0,1,2,0,1,2.

This visits exactly the 9 cells of the box in one loop.

### Expression Operators: Multiplication Undo Formula

When inserting `* val` into the expression with current running value `calc` and last added operand `tail`:

```
calc_new = calc - tail + tail * val
tail_new = tail * val
```

For addition: `tail_new = +val`.
For subtraction: `tail_new = -val` (stored negative so the multiplication undo formula remains uniform).

---

## 13. Backtracking vs Dynamic Programming

Both explore subproblems, but they are suited to different problem types:

| Criterion | Backtracking | Dynamic Programming |
|---|---|---|
| Need all solutions | Yes | No — DP gives count or yes/no |
| Subproblems overlap | Memoize to convert to DP | Already DP by design |
| Order matters in the path | Yes | Usually no |
| Constraint satisfaction (hard constraints on partial solutions) | Core use case | Usually not applicable |
| Output is the path itself | Yes | No — DP records only optimal/feasible values |

LC 139 Word Break sits at the boundary: if the question asks "can it be segmented?" → DP is optimal. If it asks "return all possible sentences" (LC 140 Word Break II) → backtracking with memoization is required, because the path itself is part of the output.

**The three questions to ask before writing backtracking code:**

1. **What is the state?** (current index, current board, current path, running total)
2. **What are the choices?** (which character, which direction, which color, which operator)
3. **What are the constraints?** (palindrome check, bounds + match, no conflict, validity)

Answer these three and the code follows directly from the template.

---

## 14. Common Mistakes and Pitfalls

### Mistake 1: Forgetting to undo state after recursion

```cpp
// WRONG: state from one branch pollutes subsequent branches
if (isValid(state, c)) {
    makeChoice(state, c);
    backtrack(state, results);
    // FORGOT: undoChoice(state, c)
}

// CORRECT: always pair makeChoice with undoChoice
makeChoice(state, c);
backtrack(state, results);
undoChoice(state, c);
```

### Mistake 2: Passing state by value instead of by reference

```cpp
// WRONG: copies entire board (e.g., n*n matrix) on every recursive call
void backtrack(vector<vector<char>> board, ...) { ... }

// CORRECT: pass by reference, undo explicitly
void backtrack(vector<vector<char>>& board, ...) {
    board[r][c] = tmp;   // explicit undo after recursion
}
```

Copying an n×n board on every call turns O(n!) into O(n! × n²) — a catastrophic constant factor.

### Mistake 3: Recording result at wrong level

```cpp
// WRONG: records incomplete partitions (curr might not cover all of s)
void backtrack(int start, vector<string>& curr, ...) {
    results.push_back(curr);   // recorded unconditionally
    for (int end = start; ...) { ... }
}

// CORRECT: record only when goal is reached
if (start == n) { results.push_back(curr); return; }
```

### Mistake 4: Not short-circuiting for feasibility problems

```cpp
// WRONG for Sudoku/Word Search/M-Coloring: continues exploring after finding solution
backtrack(state);   // return value ignored

// CORRECT: propagate success immediately
if (backtrack(state)) return true;
```

### Mistake 5: `continue` vs `break` for leading zeros in LC 282

```cpp
// WRONG: skips current len but tries len+1, producing "01", "023" as valid numbers
if (len > 1 && num[idx] == '0') continue;

// CORRECT: break terminates all longer prefixes starting at idx
if (len > 1 && num[idx] == '0') break;
```

Once `num[idx] == '0'`, no prefix of length ≥ 2 starting at `idx` is valid. `continue` skips the current `len` but tries `len+1`, `len+2`, etc., all of which also start with `'0'`. `break` correctly discards all of them at once.

### Mistake 6: Using `int` instead of `long` in LC 282

```cpp
// WRONG: overflow when num="9999999999" and tail*val is computed
int calc = ..., tail = ..., val = ...;

// CORRECT: all arithmetic must use long (64-bit)
long calc = ..., tail = ..., val = ...;
```

### Mistake 7: Wrong diagonal index (negative index) in N-Queens

```cpp
// WRONG: row-col can be negative; cannot index a C++ array with negative index
int d1 = row - col;   // undefined behavior for negative index

// CORRECT: offset by n-1 to make all indices non-negative
int d1 = row - col + n - 1;   // range: 0 to 2n-2
```

### Mistake 8: Not marking the current cell as visited in grid DFS (Rat/Word Search)

```cpp
// WRONG: rat/DFS can revisit the current cell, causing infinite recursion
for (int i = 0; i < 4; i++) {
    int nr = r + dr[i], nc = c + dc[i];
    if (valid(nr, nc)) dfs(mat, nr, nc, ...);   // mat[r][c] still 1
}

// CORRECT: mark current cell before recursing, restore after
mat[r][c] = 0;
for (int i = 0; i < 4; i++) { ... dfs(mat, nr, nc, ...); }
mat[r][c] = 1;
```

### Mistake 9: Wrong recursive call bounds in Lomuto/Hoare (not backtracking — wrong notes context; skipping)

### Mistake 9: Recording the goal at destination without blocking (Rat in Maze)

Marking the destination cell before checking arrival means the destination's cell value becomes 0 at the moment you arrive. Since arrival is checked as `r==n-1 && c==n-1` before marking, this is not a bug in the template above — but some implementations check the destination *after* marking, causing the destination to appear blocked and never recorded. The check must come before marking.

---

## 15. Complexity Cheat Sheet

| Problem | Time | Space | Key pruning |
|---|---|---|---|
| LC 131 Palindrome Partitioning | O(n · 2^n) | O(n²) with DP, O(n) without | Palindrome check before recursing |
| LC 79 Word Search | O(m · n · 4^L) | O(L) | In-place marking; frequency pre-check |
| LC 51 N-Queens | O(n!) | O(n) | O(1) conflict via col/diag arrays |
| GFG Rat in a Maze | O(4^(n²)) | O(n²) | In-place cell blocking |
| LC 139 Word Break (DP) | O(n² · L) | O(n) | Memoization eliminates 2^n blowup |
| GFG M-Coloring | O(m^V · V) | O(V) | `isSafe` per adjacency |
| LC 37 Sudoku Solver | O(9^k) | O(1) arrays + O(k) stack | Row/col/box O(1) arrays; immediate false on blocked cell |
| LC 282 Expression Add Operators | O(n · 3^n) | O(n) | Running calc+tail; break on leading zero |

L = word length, k = empty cells, V = vertices.

---

## 16. Interview Reference

### Problem Taxonomy

| Category | Signal | Problems |
|---|---|---|
| Enumeration | "Return all possible..." | LC 131, LC 51, GFG Rat in Maze |
| Feasibility | "Does a valid X exist?" / returns bool | LC 139 (also DP), GFG M-Coloring |
| Construction | "Fill in the solution" | LC 37 Sudoku |
| Grid Search | "Find a path / word" | LC 79, GFG Rat in Maze |
| Arithmetic generation | "Insert operators to reach target" | LC 282 |

### Extension Problems

| Problem | Connection |
|---|---|
| LC 17 — Letter Combinations | Enumeration; choices from a fixed map |
| LC 39/40 — Combination Sum I/II | Enumeration with/without duplicate handling |
| LC 46/47 — Permutations I/II | Enumeration of arrangements; `used[]` array |
| LC 78/90 — Subsets I/II | Enumeration of subsets; no target constraint |
| LC 140 — Word Break II | Word Break but return all sentences; backtracking + memo |
| LC 52 — N-Queens II | Count solutions, not boards; same algorithm, counter instead |
| LC 212 — Word Search II | All words from dictionary in a grid; Trie + DFS |

---

## 17. Quick Revision Cheat Sheet

**Template** (memorise this):
```
backtrack(state):
  if goal: record; return
  for each choice:
    if valid:
      make choice
      backtrack(new state)
      undo choice          <- always paired with make choice
```

**For feasibility (return bool):**
```
if (backtrack(state)) return true;   <- short-circuit; don't continue loop
```

---

**LC 131 Palindrome Partitioning:**
```
for end = start..n-1:
  if isPalin(s, start, end):
    push s[start..end]; recurse(end+1); pop
base: start==n -> record curr
isPalin: two-pointer O(len); or precompute dp[i][j] in O(n²) for O(1) checks
```

**LC 79 Word Search:**
```
for each cell: start DFS if cell matches word[0]
dfs(r,c,idx): if idx==len return true
              if OOB or mismatch return false
              tmp=board[r][c]; board[r][c]='#'
              found = dfs(4 dirs with ||)
              board[r][c]=tmp; return found
Frequency pre-check before starting any DFS.
```

**LC 51 N-Queens:**
```
one queen per row; try each column c
d1 = row-c+n-1 (left diag), d2 = row+c (right diag)
if col[c]||diag1[d1]||diag2[d2]: skip
else: place Q; mark; recurse(row+1); undo
base: row==n -> record board
```

**GFG Rat in a Maze:**
```
directions DLRU: dr={1,0,0,-1}, dc={0,-1,1,0}  [alphabetical -> lexicographic output]
mat[r][c]=0 before recurse; mat[r][c]=1 after
base: r==n-1 && c==n-1 -> record path
```

**LC 139 Word Break (DP):**
```
dp[0]=true
for i=1..n: for j=0..i:
  if dp[j] && s[j..i] in dict: dp[i]=true; break
return dp[n]
```

**GFG M-Coloring:**
```
solve(vertex): if vertex==V return true
  for c=1..m: if isSafe(vertex,c): color[v]=c; if solve(v+1) return true; color[v]=0
  return false
isSafe: no neighbor of v has color c
```

**LC 37 Sudoku Solver:**
```
init row[r][d], col[c][d], box[b][d] from pre-filled cells; b=(r/3)*3+c/3
solve(): find first '.'; try d=1..9; check row/col/box; place; if solve() return true; undo
return false if no d works (triggers backtrack); return true if no '.' found (board complete)
Box formula: board[(r/3)*3 + i/3][(c/3)*3 + i%3] for i=0..8
```

**LC 282 Expression Add Operators:**
```
track: idx, calc (running eval), tail (last operand for * undo)
for len=1..n-idx:
  if len>1 && num[idx]=='0': break  [not continue]
  val = num[idx..idx+len-1]
  if idx==0: first operand, no operator
  else: +val (tail=val), -val (tail=-val),
        *val: calc=calc-tail+tail*val, tail=tail*val
Use long for all arithmetic.
```

**Multiplication precedence undo:**
```
new_calc = calc - tail + tail * val
new_tail = tail * val
```

**Key invariant:** state before == state after every recursive call. Every makeChoice must pair with undoChoice.
