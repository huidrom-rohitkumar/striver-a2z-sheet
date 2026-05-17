**Topic:** Recursion and Subsequences — Constrained Generation, Include-Exclude, DP on Endings, Combinatorics
**Patterns:** Backtracking with State, Include-Exclude Binary Tree, DP on Character Endings, Sort + Two-Pointer + Combinatorics

**Resources:**
[TakeUForward Binary Strings](https://takeuforward.org/plus/dsa/problems/generate-binary-strings-without-consecutive-1s) |
[LC 22 Generate Parentheses](https://leetcode.com/problems/generate-parentheses) |
[LC 78 Subsets](https://leetcode.com/problems/subsets) |
[GFG Subsequence Sum K](https://www.geeksforgeeks.org/problems/check-if-there-exists-a-subsequence-with-sum-k/1) |
[LC 1498 Subsequences Sum Condition](https://leetcode.com/problems/number-of-subsequences-that-satisfy-the-given-sum-condition) |
[LC 940 Distinct Subsequences II](https://leetcode.com/problems/distinct-subsequences-ii) |
[Striver Binary Strings + Parentheses](https://youtu.be/b7AYbpM5YrE) |
[Striver Subsets + Subsequence Sum](https://youtu.be/eQCS_v3bw0Q)

---

## Table of Contents

1. [The Unifying Mental Models](#1-the-unifying-mental-models)
2. [The Backtracking Framework](#2-the-backtracking-framework)
3. [Problem 1 — Generate Binary Strings Without Consecutive 1s](#3-problem-1--generate-binary-strings-without-consecutive-1s)
4. [Problem 2 — Generate Parentheses (LC 22)](#4-problem-2--generate-parentheses-lc-22)
5. [Problem 3 — Subsets / Power Set (LC 78)](#5-problem-3--subsets--power-set-lc-78)
6. [Problem 4 — Check if a Subsequence with Sum K Exists (GFG)](#6-problem-4--check-if-a-subsequence-with-sum-k-exists-gfg)
7. [Problem 5 — Distinct Subsequences II (LC 940)](#7-problem-5--distinct-subsequences-ii-lc-940)
8. [Problem 6 — Number of Subsequences That Satisfy Sum Condition (LC 1498)](#8-problem-6--number-of-subsequences-that-satisfy-sum-condition-lc-1498)
9. [The Subsequence Taxonomy](#9-the-subsequence-taxonomy)
10. [Complexity Cheat Sheet](#10-complexity-cheat-sheet)
11. [Common Mistakes and Pitfalls](#11-common-mistakes-and-pitfalls)
12. [Interview Reference — Patterns and Extensions](#12-interview-reference--patterns-and-extensions)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Models

Four distinct algorithmic ideas appear across this problem set. Recognizing which one applies is the interview skill.

**Constrained generation (backtracking):** Build a string or collection one element at a time. At each position, enumerate valid choices given the current state. Recurse. Undo the choice after the recursive call returns. The recursion tree has at most O(branching^depth) leaves; pruning eliminates entire subtrees when a constraint is violated early. Used in Generate Binary Strings, Generate Parentheses, and Subsets.

**Include-exclude binary decision tree:** For subsequence problems, at every index make a binary decision: take element `i` into the current subsequence, or skip it. This produces all 2^n subsequences. Adding early-exit or pruning makes it efficient. Used in Check Subsequence Sum K and as the conceptual basis for Subsets.

**DP on subsequence endings:** When counting distinct subsequences is the goal, brute force generates exponentially many strings — infeasible for n=2000. Instead, track how many distinct subsequences end with each of the 26 characters. When a new character `c` arrives, every existing distinct subsequence can be extended by `c`, forming a new distinct subsequence. Overwrite the count for `c` (not accumulate) to avoid counting duplicates from prior occurrences of `c`. Used in Distinct Subsequences II.

**Sort + two-pointer + combinatorics:** When a subsequence is characterized purely by its minimum and maximum values (not element order), sort the array — this is valid because min/max are order-independent properties. Fix the left pointer as the minimum and shrink the right pointer as the maximum until the condition is satisfied. Every element strictly between left and right can be freely included or excluded: `2^(right - left)` valid subsequences. Precompute powers of 2 modulo 10^9 + 7. Used in LC 1498.

**The critical distinction:** Never confuse these four patterns. The sorting trick (Pattern 4) is only valid when the subsequence condition depends on unordered set properties (min, max, sum) — never when the original relative order matters (e.g., subsequence matching, LCS).

---

## 2. The Backtracking Framework

The master template that underlies Problems 1, 2, and 3:

```cpp
void backtrack(State& state, int depth, Result& result) {
    if (base_case_reached(state, depth)) {
        result.push_back(state);    // record complete configuration
        return;
    }
    for (each valid choice at current depth) {
        apply(choice, state);               // choose
        backtrack(state, depth + 1, result); // explore
        undo(choice, state);                 // unchoose (backtrack)
    }
}
```

**Two implementation styles for string/array building:**

Style 1 — Pass-by-reference with explicit undo:
```cpp
path.push_back(x);
backtrack(path, ...);
path.pop_back();    // must undo
```

Style 2 — Pass-by-value (implicit copy at each call):
```cpp
backtrack(path + x, ...);   // no undo needed; each call gets its own copy
```

Style 1 is O(1) per step for state management but requires explicit undo. Style 2 creates a new copy per call — O(current_size) — and is simpler to write at the cost of memory. For interviews: Style 1 is preferred for performance; Style 2 is acceptable when strings are short and clarity matters more.

**When to add to results:**
- Only at leaves (base case): permutations, complete strings, constrained strings.
- At every node: subsets — because every partial configuration is a valid subset.

---

## 3. Problem 1 — Generate Binary Strings Without Consecutive 1s

**Problem:** Given integer n, generate all binary strings of length n with no two consecutive `1`s, in lexicographic order.
**Link:** [TakeUForward](https://takeuforward.org/plus/dsa/problems/generate-binary-strings-without-consecutive-1s)
**Constraints:** `1 <= n <= 20`

**Examples:**
- n=1 → `["0","1"]`
- n=2 → `["00","01","10"]`
- n=3 → `["000","001","010","100","101"]`

### Why the Count Is a Fibonacci Number

Let `f(n)` = count of valid strings of length n.
- Strings ending in `0`: first n-1 characters are any valid string of length n-1. Count: `f(n-1)`.
- Strings ending in `1`: must be preceded by `0`, so last two characters are `01`. First n-2 characters are any valid string of length n-2. Count: `f(n-2)`.

Recurrence: `f(n) = f(n-1) + f(n-2)`, with `f(1) = 2` and `f(2) = 3`. This is the Fibonacci sequence (shifted). For n=3: 5, n=4: 8. Counting only (no generation) is O(n) DP.

### Approach 1 — Flag-Based Backtracking

**Intuition:** Carry a boolean `canPlace1` indicating whether the previous character was `0` (or the string is empty). If `canPlace1` is false, we must place `0`. Otherwise either character is valid.

```cpp
#include <bits/stdc++.h>
using namespace std;

void solve(int i, int n, string& s, vector<string>& result, bool canPlace1) {
    if (i == n) {
        result.push_back(s);
        return;
    }
    // '0' is always valid
    s[i] = '0';
    solve(i + 1, n, s, result, true);

    // '1' is valid only if previous was not '1'
    if (canPlace1) {
        s[i] = '1';
        solve(i + 1, n, s, result, false);
        s[i] = '0';  // backtrack: restore for any parent frame
    }
}

vector<string> generateBinaryStrings(int n) {
    string s(n, '0');
    vector<string> result;
    solve(0, n, s, result, true);
    return result;
}
```

### Approach 2 — Skip-Based Backtracking (Preferred)

**Intuition:** When placing `1` at position `i`, skip position `i+1` entirely (jump to `i+2`). The string is initialized to all `'0'`, so skipped positions remain `'0'` automatically.

```cpp
void solve(int i, int n, string& s, vector<string>& result) {
    if (i >= n) {         // >= handles both exact and overshoot when jumping by 2
        result.push_back(s);
        return;
    }
    // Place '0': proceed to next position
    solve(i + 1, n, s, result);

    // Place '1': skip i+1 to prevent consecutive 1s
    s[i] = '1';
    solve(i + 2, n, s, result);
    s[i] = '0';           // backtrack
}
```

**Why `i >= n` instead of `i == n`:** When we place `1` at position `n-1` (the last position) and jump by 2, we land at `i = n+1`, not `n`. Using `>=` captures both the clean exit (`i == n`) and the overshoot (`i == n+1`).

**Dry run — Skip-Based, n=3, s initialized to "000":**

```
solve(0, 3):
  Place '0': solve(1):
    Place '0': solve(2):
      Place '0': solve(3): i>=3 → push "000"
      Place '1': s[2]='1'; solve(4): i>=3 → push "001". s[2]='0'.
    Place '1': s[1]='1'; solve(3): i>=3 → push "010". s[1]='0'.
  Place '1': s[0]='1'; solve(2):
    Place '0': solve(3): push "100"
    Place '1': s[2]='1'; solve(4): push "101". s[2]='0'.
  s[0]='0'.

Output: ["000","001","010","100","101"]  ✓
```

Time: O(f(n) * n) where f(n) = Fibonacci(n+2). Space: O(n) recursion stack + O(n) for string.

---

## 4. Problem 2 — Generate Parentheses (LC 22)

**Problem:** Given n pairs of parentheses, generate all combinations of well-formed parentheses strings of length 2n.
**Link:** [LC 22](https://leetcode.com/problems/generate-parentheses)
**Constraints:** `1 <= n <= 8`

**Examples:**
- n=1 → `["()"]`
- n=2 → `["(())", "()()"]`
- n=3 → `["((()))","(()())","(())()","()(())","()()()"]`

### The Two Validity Constraints

A string of `(` and `)` is valid if and only if: (1) it contains exactly n `(`s and n `)`s, and (2) at no prefix does the `)` count exceed the `(` count. These two conditions translate directly into pruning rules:

- Place `(` if `open < n` (haven't exhausted the opening brackets).
- Place `)` if `close < open` (have unclosed opening brackets; closing now is valid).
- Base case: `open == n && close == n`.

Every complete string satisfying these two pruning rules is valid, and every valid string is generated exactly once.

### Implementation

Strings are built by value concatenation — no explicit undo needed, each recursive call operates on its own copy.

```cpp
#include <bits/stdc++.h>
using namespace std;

void solve(int n, int open, int close, string current, vector<string>& result) {
    if (open == n && close == n) {
        result.push_back(current);
        return;
    }
    if (open < n) {
        solve(n, open + 1, close, current + "(", result);
    }
    if (close < open) {
        solve(n, open, close + 1, current + ")", result);
    }
}

vector<string> generateParenthesis(int n) {
    vector<string> result;
    solve(n, 0, 0, "", result);
    return result;
}
```

**Count of valid strings:** The nth Catalan number `C(n) = C(2n,n) / (n+1)`. Values: C(1)=1, C(2)=2, C(3)=5, C(4)=14, C(5)=42.

**Dry run — n=2:**

```
solve(2, 0, 0, ""):
  open<2: solve(2, 1, 0, "("):
    open<2: solve(2, 2, 0, "(("):
      close<2: solve(2, 2, 1, "(()"):
        close<2: solve(2, 2, 2, "(())") → push "(())"
    close<1: solve(2, 1, 1, "()"):
      open<2: solve(2, 2, 1, "()("):
        close<2: solve(2, 2, 2, "()()") → push "()()"

Output: ["(())", "()()"]  ✓ (Catalan(2) = 2)
```

Time: O(C(n) * 2n) — generating each valid string takes O(2n); there are C(n) strings. Space: O(2n) recursion depth.

---

## 5. Problem 3 — Subsets / Power Set (LC 78)

**Problem:** Given an integer array `nums` of unique elements, return all possible subsets (the power set). No duplicate subsets.
**Link:** [LC 78](https://leetcode.com/problems/subsets)
**Constraints:** `1 <= nums.length <= 10`, all elements unique, `-10 <= nums[i] <= 10`.

**Example:** `[1,2,3]` → `[[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]`

### Key Insight: Every Node Is a Valid Subset

In permutation problems, only leaves are results. In the subsets problem, every partial configuration — at every depth of the recursion tree — is a valid subset. Add to results at every call, not just the base case.

### Approach 1 — Backtracking (Iterate from `start`) — Preferred

**Intuition:** Process elements left to right using a `start` index. At each node, add the current `path` to results. Then iterate `i` from `start` to `n-1`: include `nums[i]`, recurse with `i+1` as the new start, then pop `nums[i]` (backtrack).

The `start` index prevents revisiting earlier elements, ensuring each subset is generated exactly once.

```cpp
#include <bits/stdc++.h>
using namespace std;

void backtrack(int start, vector<int>& nums, vector<int>& path, vector<vector<int>>& result) {
    result.push_back(path);   // every prefix is a valid subset
    for (int i = start; i < (int)nums.size(); i++) {
        path.push_back(nums[i]);
        backtrack(i + 1, nums, path, result);
        path.pop_back();      // backtrack
    }
}

vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> result;
    vector<int> path;
    backtrack(0, nums, path, result);
    return result;
}
```

**Dry run — `[1,2,3]`:**

```
backtrack(0, []):
  push []. i=0: path=[1].
    backtrack(1, [1]):
      push [1]. i=1: path=[1,2].
        backtrack(2, [1,2]):
          push [1,2]. i=2: path=[1,2,3].
            backtrack(3): push [1,2,3]. no loop. return.
          pop 3. i done.
        pop 2. i=2: path=[1,3].
          backtrack(3): push [1,3]. return.
        pop 3.
      pop 1. (i done for start=1 loop in backtrack(1))
    pop 1. i=1: path=[2]. (similar) ...
    i=2: path=[3]. push [3]. return.

Result: [[],[1],[1,2],[1,2,3],[1,3],[2],[2,3],[3]]  ✓
```

### Approach 2 — Include-Exclude Binary Tree

Make a binary decision at each index: include or exclude. Base case collects the complete subset.

```cpp
void solve(int index, vector<int>& nums, vector<int>& path, vector<vector<int>>& result) {
    if (index == (int)nums.size()) {
        result.push_back(path);
        return;
    }
    // Exclude nums[index]
    solve(index + 1, nums, path, result);
    // Include nums[index]
    path.push_back(nums[index]);
    solve(index + 1, nums, path, result);
    path.pop_back();
}
```

### Approach 3 — Bit Masking

Represent each subset as a bitmask `mask` from `0` to `2^n - 1`. Bit `j` set means `nums[j]` is included.

```cpp
vector<vector<int>> subsetsBit(vector<int>& nums) {
    int n = nums.size();
    vector<vector<int>> result;
    for (int mask = 0; mask < (1 << n); mask++) {
        vector<int> subset;
        for (int j = 0; j < n; j++)
            if (mask & (1 << j)) subset.push_back(nums[j]);
        result.push_back(subset);
    }
    return result;
}
```

### Approach 4 — Iterative Cascading

Start with `[[]]`. For each element, duplicate all existing subsets and append the element to the duplicates.

```cpp
vector<vector<int>> subsetsIterative(vector<int>& nums) {
    vector<vector<int>> result = {{}};
    for (int num : nums) {
        int sz = result.size();
        for (int i = 0; i < sz; i++) {
            result.push_back(result[i]);
            result.back().push_back(num);
        }
    }
    return result;
}
```

**Cascading dry run for `[1,2,3]`:**
```
Start: [[]]
num=1: [[],[1]]
num=2: [[],[1],[2],[1,2]]
num=3: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

**When to choose which:**
- Backtracking (Approach 1): most general; extends to Subsets II (with duplicates) and combination-sum problems.
- Bit masking (Approach 3): concise for small n without constraints.
- Cascading (Approach 4): simplest iterative; fixed O(n * 2^n).

Time: O(n * 2^n) for all approaches — 2^n subsets, each copied in O(n) worst case. Space: O(n) recursion depth (Approaches 1-2); O(1) auxiliary for Approach 3.

---

## 6. Problem 4 — Check if a Subsequence with Sum K Exists (GFG)

**Problem:** Given array `arr[]` and integer `k`, return `true` if any subsequence of `arr` sums to exactly `k`.
**Link:** [GFG](https://www.geeksforgeeks.org/problems/check-if-there-exists-a-subsequence-with-sum-k/1)
**Constraints:** `1 <= n <= 35`, `0 <= arr[i] <= 10^9`, `0 <= k <= 10^13`

**Examples:**
- `[1,2,3]`, k=5 → `true` (subsequence [2,3])
- `[1,2,3]`, k=7 → `false`

### Approach 1 — Recursive Include-Exclude with Short-Circuit

**Intuition:** At each index, include or exclude the element. Carry a running sum. Return `true` immediately when sum hits k — this short-circuits all remaining branches without exploring the entire 2^n tree.

```cpp
bool solve(int index, long long sum, vector<int>& arr, long long k) {
    if (sum == k) return true;
    if (index == (int)arr.size()) return false;
    if (sum > k) return false;   // pruning: only valid for non-negative arrays

    if (solve(index + 1, sum + arr[index], arr, k)) return true;
    return solve(index + 1, sum, arr, k);
}

bool checkSubsequenceSum(vector<int>& arr, long long k) {
    return solve(0, 0, arr, k);
}
```

**Pruning `sum > k`:** Valid only when all elements are non-negative. Once the sum exceeds k, no future inclusion can reduce it. For arrays with negative elements, remove this pruning.

**Dry run — `arr=[1,2,3]`, k=5:**

```
solve(0, 0):
  Include 1: solve(1, 1):
    Include 2: solve(2, 3):
      Include 3: solve(3, 6): 6 > 5 → false
      Exclude 3: solve(3, 3): 3 != 5, index==n → false
    Exclude 2: solve(2, 1):
      Include 3: solve(3, 4): 4 != 5 → false
      Exclude 3: solve(3, 1): false
  Exclude 1: solve(1, 0):
    Include 2: solve(2, 2):
      Include 3: solve(3, 5): 5 == 5 → true ✓ (bubbles up instantly)
```

Time: O(2^n) worst case. Short-circuit makes it much faster in practice. Space: O(n).

### Approach 2 — Meet in the Middle (for n up to 35)

When n approaches 35, O(2^35) ≈ 34 billion — infeasible. Meet in the middle reduces this to O(2^(n/2) * n/2) ≈ O(2^17 * 17) ≈ 2 million.

Split the array in half. Generate all subset sums of each half. Sort the second half's sums. For each sum `s` in the first half, binary-search for `k - s` in the second half.

```cpp
#include <bits/stdc++.h>
using namespace std;

void generateSums(vector<int>& arr, int lo, int hi, vector<long long>& sums) {
    int len = hi - lo;
    for (int mask = 0; mask < (1 << len); mask++) {
        long long sum = 0;
        for (int j = 0; j < len; j++)
            if (mask & (1 << j)) sum += arr[lo + j];
        sums.push_back(sum);
    }
}

bool checkSubsequenceSumMeetMiddle(vector<int>& arr, long long k) {
    int n = arr.size();
    int mid = n / 2;
    vector<long long> left, right;
    generateSums(arr, 0, mid, left);
    generateSums(arr, mid, n, right);
    sort(right.begin(), right.end());
    for (long long s : left)
        if (binary_search(right.begin(), right.end(), k - s)) return true;
    return false;
}
```

**When to use:** n ≤ 20: simple recursion. n ≤ 35: meet in the middle. The GFG problem has n ≤ 35 so meet in the middle is the intended optimal.

Time: O(2^(n/2) * log(2^(n/2))). Space: O(2^(n/2)).

---

## 7. Problem 5 — Distinct Subsequences II (LC 940)

**Problem:** Given string `s`, return the number of distinct non-empty subsequences of `s`, modulo 10^9 + 7.
**Link:** [LC 940](https://leetcode.com/problems/distinct-subsequences-ii)
**Constraints:** `1 <= s.length <= 2000`, lowercase English letters only.

**Examples:**
- `"abc"` → 7 (`"a","b","c","ab","ac","bc","abc"`)
- `"aba"` → 6 (`"a","b","ab","ba","aa","aba"`)
- `"aaa"` → 3 (`"a","aa","aaa"`)

### Why Brute Force Fails

For n=2000, there are up to 2^2000 possible subsequences. Any approach that enumerates them is infeasible. DP is required.

### The Key Insight: Track by Ending Character

**Core observation:** When we process a new character `c`, every existing distinct subsequence can be extended by `c` to form a new distinct subsequence. Additionally, `c` alone forms one new subsequence. So the count of distinct subsequences ending with `c` becomes `(total existing) + 1`.

**The duplicate problem:** If `c` appeared before, the old count of subsequences ending with `c` would be double-counted — those subsequences are now included as extensions of the new `c`. The fix: **overwrite** `endsWith[c]` with the new count, do not accumulate.

**Why overwriting avoids duplicates:** The new `endsWith[c]` represents: {`c`} union {every existing distinct subsequence extended by `c`}. The old `endsWith[c]` (subsequences ending with a previous `c`) is a strict subset of this new set — because every one of those old subsequences can also be formed by taking all characters up to but not including the current `c`, then appending `c`. Setting `endsWith[c] = total + 1` replaces the old count with the new (larger) count, correctly deduplicating.

**State:** `endsWith[26]` where `endsWith[c]` = number of distinct non-empty subsequences ending with character `c`. Running total = sum of all `endsWith[c]`.

**Transition when processing character `c` at position `i`:**
```
prev         = endsWith[c]
new          = total + 1
total        = total - prev + new    (remove old, add new)
endsWith[c]  = new
```

```cpp
#include <bits/stdc++.h>
using namespace std;

int distinctSubseqII(string s) {
    const int MOD = 1e9 + 7;
    vector<long long> endsWith(26, 0);
    long long total = 0;

    for (char ch : s) {
        int idx = ch - 'a';
        long long prev = endsWith[idx];
        long long newCount = (total + 1) % MOD;
        total = ((total - prev + newCount) % MOD + MOD) % MOD;  // +MOD guards negatives
        endsWith[idx] = newCount;
    }
    return (int)total;
}
```

**The `+ MOD` guard:** In C++, `(a - b) % MOD` can be negative when `a < b` (e.g., `-5 % 3 == -2`). Adding MOD before taking modulo maps the result back into `[0, MOD-1]`. This is mandatory whenever a subtraction precedes a modulo operation.

**Time:** O(n). Space: O(26) = O(1).

### Dry Run — `s = "aba"`

```
Initial: endsWith=[0]*26, total=0

ch='a' (idx=0):
  prev=0. newCount=(0+1)%MOD=1.
  total = (0-0+1)%MOD = 1. endsWith[a]=1.
  Subsequences: {"a"}  ✓ count=1

ch='b' (idx=1):
  prev=0. newCount=(1+1)%MOD=2.
  total = (1-0+2)%MOD = 3. endsWith[b]=2.
  Subsequences: {"a","b","ab"}  ✓ count=3

ch='a' (idx=0):
  prev=endsWith[a]=1. newCount=(3+1)%MOD=4.
  total = (3-1+4)%MOD = 6. endsWith[a]=4.
  Subsequences ending with 'a': {"a","ba","aba","aa"} (4) ✓
  Subsequences ending with 'b': {"b","ab"} (2) ✓
  All 6: {"a","b","ab","ba","aa","aba"}  ✓

Return: 6  ✓
```

**Why `prev` is subtracted from `total`:** When `a` appears the second time, the old `endsWith[a]=1` counted `{"a"}` from the first occurrence. The new `endsWith[a]=4` counts `{"a","ba","aba","aa"}` (all existing extended by this new `a`, plus `a` alone). Without subtracting `prev=1`, `total` would add `4` on top of the already-counted `1`, giving `3 - 0 + 4 = 7` — wrong. By removing the old contribution and adding the new, `total` stays accurate.

---

## 8. Problem 6 — Number of Subsequences That Satisfy Sum Condition (LC 1498)

**Problem:** Given array `nums` and integer `target`, return the number of non-empty subsequences of `nums` such that the sum of the minimum and maximum element in each subsequence is `<= target`. Return modulo 10^9 + 7.
**Link:** [LC 1498](https://leetcode.com/problems/number-of-subsequences-that-satisfy-the-given-sum-condition)
**Constraints:** `1 <= nums.length <= 10^5`, `0 <= nums[i] <= 10^6`, `1 <= target <= 10^6`

**Examples:**
- `[3,5,6,7]`, target=9 → 4
- `[3,3,6,8]`, target=10 → 6
- `[2,3,3,4,6,7]`, target=12 → 61

### Why Sorting Is Valid Here

A subsequence's minimum and maximum depend only on **which elements are selected**, not their order. Swapping the order of elements in a subsequence does not change its min or max. Therefore we can sort the array without changing the set of valid (min, max) pairs — the same combinations of elements exist, just enumerated in sorted order. **This is only valid because the condition checks an unordered property of the subset.**

### Why `2^(right - left)` and not `2^(right - left + 1)`

After sorting, fix `left` as the minimum (it must be included — it is the designated smallest element). Elements at indices `left+1` through `right` are optional: each can be included or excluded. That is `right - left` elements, each with 2 choices: `2^(right - left)` subsequences. The element at `left` is not optional — do not count it as a free choice.

### Why Precompute Powers of 2

`2^(right - left)` with `right - left` up to `n - 1 = 10^5 - 1` must be taken modulo 10^9 + 7. `std::pow` returns doubles (precision loss for large exponents) and does not support modular arithmetic. Precompute `pow2[i] = 2^i % MOD` for all `i` from `0` to `n-1` in O(n), then look up in O(1).

```cpp
#include <bits/stdc++.h>
using namespace std;

int numSubseq(vector<int>& nums, int target) {
    const int MOD = 1e9 + 7;
    int n = nums.size();
    sort(nums.begin(), nums.end());

    // Precompute powers of 2 modulo MOD
    vector<long long> pow2(n);
    pow2[0] = 1;
    for (int i = 1; i < n; i++) pow2[i] = (pow2[i - 1] * 2) % MOD;

    int left = 0, right = n - 1;
    long long ans = 0;

    while (left <= right) {
        if (nums[left] + nums[right] <= target) {
            // nums[left] is the min; any subset of (right-left) elements between
            // left and right is valid — gives 2^(right-left) subsequences
            ans = (ans + pow2[right - left]) % MOD;
            left++;
        } else {
            right--;   // max too large; shrink window
        }
    }
    return (int)ans;
}
```

**Dry run — `[3,5,6,7]`, target=9:**

```
Sorted: [3,5,6,7]. pow2=[1,2,4,8].
left=0, right=3.

3+7=10>9 → right=2.
3+6=9<=9 → ans += pow2[2-0]=4. ans=4. left=1.
  (Subsequences: {3},{3,5},{3,6},{3,5,6} — all have min=3, max<=6 ✓)
5+6=11>9 → right=1.
5+5=10>9 → right=0.
left=1 > right=0. Exit.

Return: 4  ✓
```

**Dry run — `[3,3,6,8]`, target=10:**

```
Sorted: [3,3,6,8]. pow2=[1,2,4,8].
3+8=11>10 → right=2.
3+6=9<=10 → ans+=pow2[2]=4. ans=4. left=1.
3+6=9<=10 → ans+=pow2[1]=2. ans=6. left=2.
6+6=12>10 → right=1.
left=2 > right=1. Exit.

Return: 6  ✓
```

Time: O(n log n) for sort + O(n) for two-pointer + O(n) for precomputation = O(n log n). Space: O(n) for `pow2`.

---

## 9. The Subsequence Taxonomy

Understanding the type of problem determines which approach applies:

| Problem type | Key question | Approach |
|---|---|---|
| Generate all subsequences | List all 2^n | Backtracking (include-exclude) |
| Check if one subsequence satisfies condition | Exists? | Recursion with pruning + short-circuit; meet-in-middle for large n |
| Count distinct subsequences | How many unique? | DP on character endings (endsWith[26]) |
| Count valid subsequences (min+max condition) | How many have min+max <= target? | Sort + two-pointer + precomputed powers |
| Count subarrays with sum/XOR condition | Contiguous elements | Prefix sum/XOR + hash map |
| Longest common subsequence | LCS of two strings | 2D DP |

**Subsequence vs subarray vs subset:**
- **Subarray:** contiguous elements in original order — `arr[i..j]`.
- **Subsequence:** elements in original order, not necessarily contiguous — delete some elements without reordering the rest.
- **Subset:** any selection of elements, order irrelevant — every subset is a subsequence when sorted; every subsequence is a subset.

---

## 10. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Binary Strings (generate) | Backtracking | O(Fib(n) * n) | O(n) |
| Binary Strings (count only) | Fibonacci DP | O(n) | O(1) |
| Generate Parentheses | Backtracking | O(C(n) * 2n) | O(n) |
| Subsets (backtracking) | Iterate from start | O(n * 2^n) | O(n) |
| Subsets (bit masking) | Enumerate masks | O(n * 2^n) | O(1) |
| Subsets (cascading) | Iterative duplicate | O(n * 2^n) | O(2^n * n) |
| Subsequence Sum K (n ≤ 20) | Include-exclude recursion | O(2^n) | O(n) |
| Subsequence Sum K (n ≤ 35) | Meet in the middle | O(2^(n/2) * n) | O(2^(n/2)) |
| Distinct Subsequences II | DP on endsWith[26] | O(n) | O(1) |
| Subsequences Sum Condition | Sort + two-pointer | O(n log n) | O(n) |

C(n) = Catalan(n) = C(2n,n)/(n+1). Fib(n) grows as φ^n/√5 where φ ≈ 1.618.

---

## 11. Common Mistakes and Pitfalls

### Mistake 1 — Forgetting `pop_back` (not backtracking)

```cpp
// WRONG: path is never restored; all future branches see a corrupted state
for (int i = start; i < n; i++) {
    path.push_back(nums[i]);
    backtrack(i + 1, nums, path, result);
    // MISSING: path.pop_back();
}

// CORRECT:
path.push_back(nums[i]);
backtrack(i + 1, nums, path, result);
path.pop_back();   // restore
```

### Mistake 2 — Using `i` instead of `i+1` as the start in subsets backtracking

```cpp
// WRONG: nums[i] can be used again, causing infinite recursion or duplicates
backtrack(i, nums, path, result);

// CORRECT:
backtrack(i + 1, nums, path, result);
```

### Mistake 3 — Adding to results only at the base case in subsets

```cpp
// WRONG: misses all non-empty non-maximal subsets
void backtrack(int start, ...) {
    if (start == n) { result.push_back(path); return; }   // only leaves counted
    ...
}

// CORRECT: add at every node (every prefix is a valid subset)
void backtrack(int start, ...) {
    result.push_back(path);   // add before the loop
    for (int i = start; ...) { ... }
}
```

### Mistake 4 — Accumulating instead of overwriting in Distinct Subsequences II

```cpp
// WRONG: old subsequences ending in 'a' + new ones = duplicates
endsWith[idx] += total + 1;

// CORRECT: entirely replace old count with new
endsWith[idx] = total + 1;
// (and update running total: total = total - prev + new)
```

### Mistake 5 — Using `2^(right - left + 1)` instead of `2^(right - left)` in LC 1498

```cpp
// WRONG: treats nums[left] as optional; it must be included as the minimum
ans += pow2[right - left + 1];

// CORRECT: nums[left] is mandatory; only right-left elements between left and right are optional
ans += pow2[right - left];
```

### Mistake 6 — Not taking modulo at every step in LC 1498

```cpp
// WRONG: overflows long long after ~63 iterations
pow2[i] = pow2[i - 1] * 2;

// CORRECT:
pow2[i] = (pow2[i - 1] * 2) % MOD;
// Also: ans = (ans + pow2[right - left]) % MOD;
```

### Mistake 7 — Wrong pruning condition in Generate Parentheses

```cpp
// WRONG: allows ')' when no unmatched '(' exists; produces invalid strings like ")("
if (close < n) solve(n, open, close + 1, ...);

// CORRECT:
if (close < open) solve(n, open, close + 1, ...);
```

### Mistake 8 — Pruning `sum > k` on arrays with negative numbers

```cpp
// WRONG for negative arrays: sum can decrease after exceeding k
if (sum > k) return false;

// CORRECT for non-negative arrays: valid pruning
// For arrays with negatives: remove this pruning entirely
```

### Mistake 9 — Missing `+ MOD` guard on subtraction in modular arithmetic

```cpp
// WRONG in C++: (-5) % MOD can return -2 (negative)
total = (total - prev + newCount) % MOD;

// CORRECT: add MOD before taking modulo to ensure non-negative result
total = ((total - prev + newCount) % MOD + MOD) % MOD;
```

---

## 12. Interview Reference — Patterns and Extensions

### Pattern Recognition Table

| Signal in problem | Approach |
|---|---|
| "Generate all X without Y" | Backtracking with constraint as pruning |
| "All subsets / power set" | Backtracking (add at every node) or bit masking |
| "Does any subsequence sum to k?" (n ≤ 20) | Include-exclude recursion with short-circuit |
| "Does any subsequence sum to k?" (n ≤ 35) | Meet in the middle |
| "Count distinct subsequences" (possible duplicates in string) | DP with endsWith[26]; overwrite on repeat char |
| "Count subsequences where min + max ≤ target" | Sort + two-pointer + precomputed 2^k powers |
| Count binary strings length n without consecutive 1s | Fibonacci recurrence f(n) = f(n-1) + f(n-2) |
| Count valid parenthesizations of n pairs | Catalan number C(2n,n)/(n+1) |

### Extensions

**Binary Strings → Generalize:** Strings over {0,1,...,k-1} without two consecutive identical characters — flag-based backtracking with `prevChar` state instead of a boolean.

**Generate Parentheses → Related:**
- LC 32 Longest Valid Parentheses: DP or stack to find longest valid substring.
- LC 301 Remove Invalid Parentheses: BFS removing minimal characters.
- Multiple bracket types: extend `open`/`close` counters to one pair per type.

**Subsets → Related:**
- LC 90 Subsets II (with duplicates): sort first; skip `nums[i] == nums[i-1]` when `i > start`.
- LC 77 Combinations: choose exactly k elements — add only when `path.size() == k`.
- LC 39 Combination Sum: can reuse elements — pass `i` (not `i+1`) to next call.

**Distinct Subsequences II → Related:**
- LC 115 Distinct Subsequences: count occurrences of string `t` as subsequence in `s`. 2D DP: `dp[i][j]` = ways to form `t[0..j-1]` using `s[0..i-1]`.

**Subsequences Sum Condition → Related:**
- LC 167 Two Sum II: same two-pointer on sorted array, return single pair.
- Count subsequences with exact min + max = target: change `<=` to `==` in the condition.

---

## 13. Quick Revision Cheat Sheet

This section is fully self-contained for night-before revision.

**Backtracking Template**
```cpp
void backtrack(state, start, result) {
    result.push_back(state);   // for subsets: add at every node
    // OR if base case: if (done) { result.push_back(state); return; }
    for (each valid choice from start) {
        apply(choice);
        backtrack(state, next, result);
        undo(choice);           // pop_back / restore
    }
}
```

**Binary Strings No Consecutive 1s**
- Count = f(n) = f(n-1) + f(n-2), f(1)=2, f(2)=3 (Fibonacci)
- Skip approach: place '1' at `i` → jump to `i+2` (skip `i+1`). Use `i >= n` as base case.
- Always place '0' at every position; place '1' only conditionally.

**Generate Parentheses**
- State: `(open, close)`. Place `(` if `open < n`; place `)` if `close < open`.
- Base: `open == n && close == n`. No explicit undo — string passed by value.
- Count = Catalan(n): 1, 2, 5, 14, 42, ...

**Subsets (LC 78)**
- Backtrack: `result.push_back(path)` at every node; loop `i` from `start`; push, recurse `i+1`, pop.
- Bit mask: for mask in [0, 2^n): include `nums[j]` if bit j set.
- O(n * 2^n) time for all approaches.

**Subsequence Sum K**
- Recursion: at each index, include or exclude. Short-circuit on match. Pruning: `sum > k` (positive arrays only).
- n ≤ 35: meet in the middle. Split half/half; generate sums for each; sort one; binary search.

**Distinct Subsequences II (LC 940) — Critical**
```
endsWith[26] = {0}, total = 0
For each char c (idx = c - 'a'):
    prev       = endsWith[idx]
    newCount   = (total + 1) % MOD
    total      = ((total - prev + newCount) % MOD + MOD) % MOD
    endsWith[idx] = newCount
Return total % MOD
```
- Overwrite, not accumulate. The `+MOD` guard prevents negative modulo in C++.
- O(n) time, O(1) space.

**Subsequences Sum Condition (LC 1498)**
```
Sort nums.
Precompute pow2[i] = 2^i % MOD for i in [0, n-1].
left=0, right=n-1, ans=0.
While left <= right:
    if nums[left] + nums[right] <= target:
        ans = (ans + pow2[right - left]) % MOD   // NOT right-left+1
        left++
    else:
        right--
```
- `2^(right-left)`: nums[left] is mandatory; right-left elements are optional.
- O(n log n) time, O(n) space.

**Complexity Summary**

| Problem | Time | Space |
|---|---|---|
| Binary Strings (generate) | O(Fib(n) * n) | O(n) |
| Generate Parentheses | O(Catalan(n) * 2n) | O(n) |
| Subsets | O(n * 2^n) | O(n) |
| Subsequence Sum K | O(2^n) or O(2^(n/2)*n) | O(n) or O(2^(n/2)) |
| Distinct Subsequences II | O(n) | O(1) |
| Subsequences Sum Condition | O(n log n) | O(n) |

**Three numbers to remember:**
- Catalan(n) = C(2n,n)/(n+1): parenthesization, triangulation, Dyck paths.
- Fibonacci: binary strings without consecutive 1s, staircase counting.
- 2^k: subsets of k elements, binary strings of length k.
