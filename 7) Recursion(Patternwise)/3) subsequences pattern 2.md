## Resources

- [Striver — Combination Sum I & Subset Sums (YouTube)](https://youtu.be/OyZFFqQtu98)
- [Striver — Combination Sum II & Subsets II (YouTube)](https://youtu.be/G1fRTGRxXU8)
- [Striver — Combination Sum III (YouTube)](https://youtu.be/rYkfBRtMJr8)
- [Striver — Letter Combinations of Phone Number (YouTube)](https://youtu.be/RIn3gOkbhQE)
- [LeetCode 39 — Combination Sum](https://leetcode.com/problems/combination-sum)
- [LeetCode 40 — Combination Sum II](https://leetcode.com/problems/combination-sum-ii)
- [LeetCode 216 — Combination Sum III](https://leetcode.com/problems/combination-sum-iii)
- [GFG — Subset Sums](https://www.geeksforgeeks.org/problems/subset-sums2234/1)
- [LeetCode 90 — Subsets II](https://leetcode.com/problems/subsets-ii)
- [LeetCode 17 — Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number)

---

## Table of Contents

1. [The Two Backtracking Archetypes](#1-the-two-backtracking-archetypes)
2. [The Duplicate-Handling Mechanism — Formal Explanation](#2-the-duplicate-handling-mechanism--formal-explanation)
3. [The Three Key Dials](#3-the-three-key-dials)
4. [Problem: Subset Sums (GFG)](#4-problem-subset-sums-gfg)
5. [Problem: Subsets II (LC 90)](#5-problem-subsets-ii-lc-90)
6. [Problem: Combination Sum I (LC 39) — Unlimited Reuse](#6-problem-combination-sum-i-lc-39--unlimited-reuse)
7. [Problem: Combination Sum II (LC 40) — No Reuse, Duplicates Possible](#7-problem-combination-sum-ii-lc-40--no-reuse-duplicates-possible)
8. [Problem: Combination Sum III (LC 216) — Exactly k from 1–9](#8-problem-combination-sum-iii-lc-216--exactly-k-from-19)
9. [Problem: Letter Combinations of a Phone Number (LC 17)](#9-problem-letter-combinations-of-a-phone-number-lc-17)
10. [The Combination Sum Family — Unified Comparison](#10-the-combination-sum-family--unified-comparison)
11. [Edge Cases](#11-edge-cases)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Interview Reference: Patterns and Extensions](#13-interview-reference-patterns-and-extensions)
14. [Complexity Cheat Sheet](#14-complexity-cheat-sheet)
15. [Quick Revision Cheat Sheet](#15-quick-revision-cheat-sheet)

---

## 1. The Two Backtracking Archetypes

Every problem in this set is a structural variation of one of two archetypes. Recognizing which one applies is the first step.

### Archetype A — Include-or-Skip at each index

At position `i`, make exactly two choices: include `arr[i]` or skip it. Then strictly advance to `i+1`.

```cpp
void solve(int i, ...) {
    if (base_case) { record; return; }
    // Skip arr[i]
    solve(i + 1, ...);
    // Include arr[i]
    current.push_back(arr[i]);
    solve(i + 1 or i, ...);   // i+1 if no reuse, i if reuse allowed
    current.pop_back();
}
```

Used in: Subset Sums, Combination Sum I (include-or-skip variant).

### Archetype B — For-loop over choices starting from `start`

At each recursive call, iterate over remaining choices from `start` to `end` using a `for` loop. Each choice is tried, recursed, and backtracked.

```cpp
void solve(int start, ...) {
    if (base_case or record_always) { record; return; }
    for (int i = start; i < n; i++) {
        if (duplicate_condition) continue;   // skip duplicate siblings
        if (pruning_condition) break;
        current.push_back(arr[i]);
        solve(i + 1 or i, ...);   // i+1 if no reuse, i if reuse allowed
        current.pop_back();
    }
}
```

Used in: Combination Sum I (loop variant), Combination Sum II, Combination Sum III, Subsets II, Letter Combinations.

**Why both archetypes produce the same results:** They traverse the same decision tree described differently. Archetype A's two recursive branches (skip / include) correspond to Archetype B's for-loop iterations: the loop over `i = start..end` explores all "include" choices; the implicit "skip" is captured by the loop body not starting over from 0.

---

## 2. The Duplicate-Handling Mechanism — Formal Explanation

This is the single most important concept in this set. It appears in Combination Sum II and Subsets II, and must be understood precisely.

**Setup:** Input array has duplicate elements (e.g., `[1,1,2]` after sorting). We want `[1,1,2]` as a valid combination (using both 1s) but not `[1,2]` produced twice through different index choices.

**The skip rule:**

```cpp
if (i > start && arr[i] == arr[i - 1]) continue;
```

**What `i > start` means precisely:**

`start` is the beginning index of the current loop at the current recursion depth. When `i == start`, this is the first element being considered at this depth — it is always a fresh valid choice. When `i > start`, we are considering the second (or later) element at this depth — a "sibling" in the decision tree. If it equals the previous sibling, including it would produce the same subtree as the previous sibling and therefore the same combinations.

**Proof by example:** `candidates = [1,1,2]`, target = 3, sorted.

Without skip:
- At depth 0, i=0: take `1(first)`. Recurse with start=1.
- At depth 0, i=1: take `1(second)`. Recurse with start=2. → produces same combinations as above.

With skip `i > start && arr[i] == arr[i-1]`:
- At depth 0, i=0: `1(first)`, take it. start=0, i=0: `i > start` is `0 > 0` → false. Allowed.
- At depth 0, i=1: `arr[1]=1 == arr[0]=1` AND `1 > 0` → skip. The entire duplicate subtree is pruned.
- At depth 1 (called with start=1), i=1: `i > start` is `1 > 1` → false. `1(second)` is allowed here. This is why `[1,1,2]` is still found — the second `1` is taken at a deeper level.

**The invariant in plain language:** Skip a sibling if it has the same value as the previous sibling at this same recursion depth. Never skip based on absolute index — that would incorrectly prevent using duplicates at deeper levels.

**Why sort is mandatory:** The skip rule compares `arr[i]` with `arr[i-1]`. If equal elements are not adjacent, the comparison fails. Sorting is the prerequisite.

---

## 3. The Three Key Dials

These three decisions fully determine any problem's structure:

**Dial 1: `i` or `i+1` in the recursive call?**
- `i`: same element may be picked again (unlimited reuse) — Combination Sum I.
- `i+1`: each element used at most once — everything else.

**Dial 2: Sort + duplicate skip or not?**
- Required when: input contains duplicates AND you want unique combinations.
- Combination Sum II, Subsets II: yes.
- Combination Sum I, Letter Combinations: no.

**Dial 3: Record at every node, or only at leaves meeting a constraint?**
- Every node is a valid result (subset generation): `result.push_back(current)` before the loop — Subsets II, Subset Sums.
- Only leaves meeting a constraint (combination generation): record only when `target == 0` or `size == k` — Combination Sum I/II/III.

---

## 4. Problem: Subset Sums (GFG)

### Problem Statement

Given an array `arr` of N integers, find the sum of every possible subset (including the empty subset) and return all sums in sorted order.

```
arr=[3,1,2]   →  [0,1,2,3,3,4,5,6]
arr=[2,3]     →  [0,2,3,5]
```

### Approach: Include-Exclude Recursion

Every element is either included or excluded. For N elements there are 2^N subsets. Pass the running sum as a parameter — no need to maintain a current array.

```cpp
void solve(int index, int sum, vector<int>& arr, vector<int>& result) {
    if (index == (int)arr.size()) {
        result.push_back(sum);
        return;
    }
    // Include arr[index]
    solve(index + 1, sum + arr[index], arr, result);
    // Exclude arr[index]
    solve(index + 1, sum, arr, result);
}

vector<int> subsetSums(vector<int>& arr) {
    vector<int> result;
    solve(0, 0, arr, result);
    sort(result.begin(), result.end());
    return result;
}
// Time: O(2^n) recursion + O(2^n * n) sort, Space: O(n) stack + O(2^n) result
```

### Dry Run

```
arr=[3,1,2]

solve(0, 0):
  Include 3: solve(1, 3):
    Include 1: solve(2, 4):
      Include 2: push 6
      Exclude 2: push 4
    Exclude 1: solve(2, 3):
      Include 2: push 5
      Exclude 2: push 3
  Exclude 3: solve(1, 0):
    Include 1: solve(2, 1):
      Include 2: push 3
      Exclude 2: push 1
    Exclude 1: solve(2, 0):
      Include 2: push 2
      Exclude 2: push 0

Unsorted: [6,4,5,3,3,1,2,0]
Sorted:   [0,1,2,3,3,4,5,6]  ✓
```

---

## 5. Problem: Subsets II (LC 90)

### Problem Statement

Given an integer array `nums` that may contain duplicates, return all possible subsets (power set) with no duplicate subsets.

```
[1,2,2]   →  [[],[1],[1,2],[1,2,2],[2],[2,2]]
[0]       →  [[],[0]]
```

### Approach: Sort + Backtracking with Duplicate Skip

Sort first. Record the current state at every node (before the for loop). Apply duplicate skip in the loop.

```cpp
void backtrack(int start, vector<int>& nums, vector<int>& current,
               vector<vector<int>>& result) {
    result.push_back(current);   // every state is a valid subset
    for (int i = start; i < (int)nums.size(); i++) {
        if (i > start && nums[i] == nums[i - 1]) continue;   // skip duplicate siblings
        current.push_back(nums[i]);
        backtrack(i + 1, nums, current, result);
        current.pop_back();
    }
}

vector<vector<int>> subsetsWithDup(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    vector<int> current;
    backtrack(0, nums, current, result);
    return result;
}
// Time: O(n log n + n * 2^n), Space: O(n) stack
```

### Dry Run

```
nums=[1,2,2] sorted

backtrack(0, []):
  push [].
  i=0: take 1. backtrack(1, [1]):
    push [1].
    i=1: take 2. backtrack(2, [1,2]):
      push [1,2].
      i=2: nums[2]=2==nums[1]=2, i(2)>start(2)? NO (i==start). Take 2.
           backtrack(3, [1,2,2]): push [1,2,2]. return.
      pop 2.
    pop 2.
    i=2: nums[2]=2==nums[1]=2, i(2)>start(1)? YES. SKIP.
  pop 1.
  i=1: take 2. backtrack(2, [2]):
    push [2].
    i=2: nums[2]=2==nums[1]=2, i(2)>start(2)? NO. Take 2.
         backtrack(3, [2,2]): push [2,2]. return.
    pop 2.
  pop 2.
  i=2: nums[2]=2==nums[1]=2, i(2)>start(0)? YES. SKIP.

Result: [[],[1],[1,2],[1,2,2],[2],[2,2]]  ✓
```

---

## 6. Problem: Combination Sum I (LC 39) — Unlimited Reuse

### Problem Statement

Given an array of **distinct** integers `candidates` and a target, return all unique combinations where chosen numbers sum to `target`. The same number may be used **an unlimited number of times**. Combinations are unique regardless of order.

```
candidates=[2,3,6,7], target=7   →  [[2,2,3],[7]]
candidates=[2,3,5], target=8    →  [[2,2,2,2],[2,3,3],[3,5]]
```

### The Reuse Mechanism

When we include `candidates[i]`, recurse with `i` (not `i+1`). Staying at the same index allows the same element to be picked again in the next call. To prevent permutations (e.g., `[2,3]` and `[3,2]` both summing to 5), the `start` parameter ensures we never look backward.

### Approach: Backtracking with For-Loop

```cpp
void backtrack(int start, int target, vector<int>& candidates,
               vector<int>& current, vector<vector<int>>& result) {
    if (target == 0) { result.push_back(current); return; }
    for (int i = start; i < (int)candidates.size(); i++) {
        if (candidates[i] > target) break;   // sorted: all remaining also > target
        current.push_back(candidates[i]);
        backtrack(i, target - candidates[i], candidates, current, result);   // i, not i+1
        current.pop_back();
    }
}

vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
    sort(candidates.begin(), candidates.end());   // for early termination via break
    vector<vector<int>> result;
    vector<int> current;
    backtrack(0, target, candidates, current, result);
    return result;
}
// Time: O(n^(t/m)) where t=target, m=smallest candidate. Space: O(t/m) stack depth.
```

**Why `break` (not `continue`) when `candidates[i] > target`:** After sorting, all subsequent elements are also `> target`. No further exploration is possible at this depth.

### Dry Run

```
candidates=[2,3,6,7] sorted, target=7

backtrack(0, 7, []):
  i=0: cand=2. [2]. backtrack(0, 5, [2]):
    i=0: cand=2. [2,2]. backtrack(0, 3, [2,2]):
      i=0: cand=2. [2,2,2]. backtrack(0, 1, [2,2,2]):
        i=0: 2>1. break. return.
      pop 2.
      i=1: cand=3. [2,2,3]. backtrack(1, 0, [2,2,3]): target==0 → push [2,2,3]. ✓
      pop 3.
      i=2: cand=6>3. break.
    pop 2.
    i=1: cand=3. [2,3]. backtrack(1, 2, [2,3]):
      i=1: 3>2. break. return.
    pop 3.
    i=2: cand=6>5. break.
  pop 2.
  i=1: cand=3. [3]. backtrack(1, 4, [3]):
    i=1: 3. [3,3]. backtrack(1, 1, [3,3]): i=1: 3>1. break.
    pop 3. i=2: 6>4. break.
  pop 3.
  i=2: 6. [6]. backtrack(2, 1): i=2: 6>1. break.
  i=3: 7. [7]. backtrack(3, 0): push [7]. ✓

Result: [[2,2,3],[7]]  ✓
```

---

## 7. Problem: Combination Sum II (LC 40) — No Reuse, Duplicates Possible

### Problem Statement

Given a collection of candidate numbers (may contain duplicates) and a target, find all unique combinations where each number is used **at most once**.

```
candidates=[10,1,2,7,6,1,5], target=8   →  [[1,1,6],[1,2,5],[1,7],[2,6]]
candidates=[2,5,2,1,2], target=5        →  [[1,2,2],[5]]
```

### Two Differences from Combination Sum I

1. No reuse: recurse with `i+1`.
2. Input has duplicates: apply the skip rule `if (i > start && candidates[i] == candidates[i-1]) continue`.

```cpp
void backtrack(int start, int target, vector<int>& candidates,
               vector<int>& current, vector<vector<int>>& result) {
    if (target == 0) { result.push_back(current); return; }
    for (int i = start; i < (int)candidates.size(); i++) {
        if (i > start && candidates[i] == candidates[i - 1]) continue;   // skip duplicate siblings
        if (candidates[i] > target) break;   // sorted: all remaining also > target
        current.push_back(candidates[i]);
        backtrack(i + 1, target - candidates[i], candidates, current, result);   // i+1
        current.pop_back();
    }
}

vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
    sort(candidates.begin(), candidates.end());   // mandatory for skip rule
    vector<vector<int>> result;
    vector<int> current;
    backtrack(0, target, candidates, current, result);
    return result;
}
// Time: O(2^n * k) where k = avg combination length, Space: O(n) stack
```

### Dry Run

```
candidates=[1,1,2,5,6,7,10] sorted, target=8

backtrack(0, 8, []):
  i=0: cand=1. [1]. backtrack(1, 7, [1]):
    i=1: cand=1. i(1)>start(1)? NO. [1,1]. backtrack(2, 6, [1,1]):
      i=2: 2. [1,1,2]. backtrack(3,4): i=3: 5>4. break. No valid.
      i=3: 5. [1,1,5]. backtrack(4,1): i=4: 6>1. break. No.
      i=4: 6. [1,1,6]. backtrack(5,0): push [1,1,6]. ✓
      i=5: 7>6. break.
    pop 1.
    i=2: cand=2. [1,2]. backtrack(3,5):
      i=3: 5. [1,2,5]. backtrack(4,0): push [1,2,5]. ✓
      i=4: 6>5. break.
    pop 2.
    i=3: 5. [1,5]. backtrack(4,2): 6>2. break. No.
    i=4: 6. [1,6]. backtrack(5,1): 7>1. break. No.
    i=5: 7. [1,7]. backtrack(6,0): push [1,7]. ✓
    i=6: 10>7. break.
  pop 1.
  i=1: cand=1==candidates[0]=1, i(1)>start(0)? YES. SKIP.
  i=2: cand=2. [2]. backtrack(3,6):
    i=3: 5. [2,5]. backtrack(4,1): 6>1. break. No.
    i=4: 6. [2,6]. backtrack(5,0): push [2,6]. ✓
    i=5: 7>6. break.
  ...

Result: [[1,1,6],[1,2,5],[1,7],[2,6]]  ✓
```

---

## 8. Problem: Combination Sum III (LC 216) — Exactly k from 1–9

### Problem Statement

Find all valid combinations of **exactly k distinct numbers** from 1–9 that sum to n. Each number may be used at most once.

```
k=3, n=7   →  [[1,2,4]]
k=3, n=9   →  [[1,2,6],[1,3,5],[2,3,4]]
k=4, n=1   →  []   (minimum possible sum: 1+2+3+4=10 > 1)
```

### Two Hard Constraints, Checked Together

Unlike Combination Sum I/II where only the sum is checked, here there are two constraints simultaneously: exactly `k` elements **and** sum equals `n`. The base case checks both.

The pool is implicit (always 1 through 9), so no input array is needed.

```cpp
void backtrack(int start, int k, int remaining, vector<int>& current,
               vector<vector<int>>& result) {
    // Both constraints met simultaneously
    if ((int)current.size() == k && remaining == 0) {
        result.push_back(current);
        return;
    }
    // Prune: too many elements selected, or sum exceeded
    if ((int)current.size() >= k || remaining < 0) return;

    for (int i = start; i <= 9; i++) {
        if (i > remaining) break;   // i is smallest remaining; all larger also exceed
        current.push_back(i);
        backtrack(i + 1, k, remaining - i, current, result);
        current.pop_back();
    }
}

vector<vector<int>> combinationSum3(int k, int n) {
    vector<vector<int>> result;
    vector<int> current;
    backtrack(1, k, n, current, result);   // start from 1
    return result;
}
// Time: O(C(9,k)) — choose k from 9 distinct values. Space: O(k) stack.
```

**Why `start` begins at 1:** Numbers are from 1 to 9 by definition.

**Why `break` when `i > remaining`:** Since `i` increases and the numbers are already in order, if even the current (smallest) number exceeds the remaining sum, no valid combination exists with any larger number.

### Dry Run

```
k=3, n=7. backtrack(1, 3, 7, [])

i=1: [1]. backtrack(2, 3, 6, [1]):
  i=2: [1,2]. backtrack(3, 3, 4, [1,2]):
    i=3: [1,2,3]. size=3==k, remaining=1≠0. return.
    i=4: [1,2,4]. size=3==k, remaining=0. push [1,2,4]. ✓
    i=5: [1,2,5]. remaining=4-5=-1<0. return.
    (break when i>remaining=4, so i=5: 5>4 → break)
  pop 2.
  i=3: [1,3]. backtrack(4, 3, 3, [1,3]):
    i=4: 4>3. break. return.
  i=4: [1,4]. backtrack(5, 3, 2, [1,4]):
    i=5: 5>2. break. return.
  ... (no more valid for [1,...])
pop 1.
i=2: [2]. backtrack(3, 3, 5, [2]):
  i=3: [2,3]. backtrack(4, 3, 2, [2,3]): i=4: 4>2. break. No.
  i=4: [2,4]. backtrack(5,3,1): i=5: 5>1. break. No.
  ... No valid.
... etc.

Result: [[1,2,4]]  ✓
```

---

## 9. Problem: Letter Combinations of a Phone Number (LC 17)

### Problem Statement

Given a string containing digits 2–9, return all possible letter combinations the number could represent based on phone button mappings.

```
digits="23"   →  ["ad","ae","af","bd","be","bf","cd","ce","cf"]
digits=""     →  []
digits="2"    →  ["a","b","c"]
```

### Structural Difference: No "Skip" Choice

This is pure Cartesian product generation, not subset selection. For each digit position, every mapped letter must be tried — there is no option to skip a digit. The branching factor at position `i` is the number of letters mapped to `digits[i]` (3 or 4).

Total combinations = product of branching factors at each position.

### Approach 1: Backtracking

```cpp
void backtrack(int index, const string& digits, string& current,
               vector<string>& result, const vector<string>& phone) {
    if (index == (int)digits.size()) {
        result.push_back(current);
        return;
    }
    const string& letters = phone[digits[index] - '0'];
    for (char c : letters) {
        current.push_back(c);
        backtrack(index + 1, digits, current, result, phone);
        current.pop_back();
    }
}

vector<string> letterCombinations(string digits) {
    if (digits.empty()) return {};   // explicit guard: empty input → empty output
    vector<string> phone = {"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    vector<string> result;
    string current;
    backtrack(0, digits, current, result, phone);
    return result;
}
// Time: O(4^n * n), Space: O(n) stack
```

### Approach 2: BFS / Iterative Queue

Start with `[""]`. For each digit, expand every existing combination by appending each mapped letter.

```cpp
vector<string> letterCombinationsBFS(string digits) {
    if (digits.empty()) return {};
    vector<string> phone = {"","","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
    vector<string> result = {""};

    for (char digit : digits) {
        vector<string> next;
        for (const string& combo : result) {
            for (char c : phone[digit - '0']) {
                next.push_back(combo + c);
            }
        }
        result = next;
    }
    return result;
}
// Time: O(4^n * n), Space: O(4^n * n) for expanding result
```

### Dry Run

```
digits="23", phone[2]="abc", phone[3]="def"

Backtracking:
  index=0, current="":
    c='a': backtrack(1, "a"):
      c='d': push "ad". c='e': push "ae". c='f': push "af".
    c='b': backtrack(1, "b"):
      c='d': push "bd". c='e': push "be". c='f': push "bf".
    c='c': backtrack(1, "c"):
      c='d': push "cd". c='e': push "ce". c='f': push "cf".

BFS:
  Start: [""]
  digit='2': ["a","b","c"]
  digit='3': ["ad","ae","af","bd","be","bf","cd","ce","cf"]

Result: ["ad","ae","af","bd","be","bf","cd","ce","cf"]  ✓
```

---

## 10. The Combination Sum Family — Unified Comparison

| Feature | Subset Sums | Subsets II | Combo Sum I | Combo Sum II | Combo Sum III | Letter Combo |
|---|---|---|---|---|---|---|
| Input has duplicates? | No | Yes | No | Yes | No (1–9 fixed) | N/A |
| Reuse allowed? | Once each | Once each | Unlimited | Once each | Once each | N/A (Cartesian) |
| Sort required? | No (result sorted after) | Yes | Yes (for pruning) | Yes (mandatory) | No (1–9 already ordered) | No |
| Duplicate skip rule? | No | Yes | No | Yes | No | No |
| Recurse with `i` or `i+1`? | `i+1` | `i+1` | `i` (reuse) | `i+1` | `i+1` | `i+1` |
| Base case: record when | index==n | every node | target==0 | target==0 | size==k AND sum==n | index==len(digits) |
| Two constraints? | No | No | Sum only | Sum only | Size AND sum | None |

---

## 11. Edge Cases

| Problem | Input | Output | Reason |
|---|---|---|---|
| Subset Sums | `arr=[0]` | `[0, 0]` | Two subsets: {} sum=0, {0} sum=0 |
| Subsets II | `[1,1,1]` | `[[],[1],[1,1],[1,1,1]]` | Three 1s; deduplicated |
| Combo Sum I | `candidates=[7], target=7` | `[[7]]` | Single element equals target |
| Combo Sum I | `candidates=[2], target=3` | `[]` | 2 cannot sum to 3 |
| Combo Sum I | `candidates=[1], target=40` | `[[1,1,...,1]]` (40 ones) | Deepest recursion; O(t) depth |
| Combo Sum II | `candidates=[1,1], target=1` | `[[1]]` | Two 1s in input but only one `[1]` in output |
| Combo Sum III | `k=4, n=1` | `[]` | Minimum sum: 1+2+3+4=10 > 1 |
| Combo Sum III | `k=9, n=45` | `[[1,2,3,4,5,6,7,8,9]]` | Only one valid combination |
| Letter Combo | `digits=""` | `[]` | Explicit guard returns empty, not `[""]` |
| Letter Combo | `digits="7"` | `["p","q","r","s"]` | 7 maps to 4 letters |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1: Using `i+1` instead of `i` in Combination Sum I (breaks reuse)

```cpp
// WRONG: passes i+1, element can never be chosen again
backtrack(i + 1, target - candidates[i], candidates, current, result);

// CORRECT: pass i to allow reuse
backtrack(i, target - candidates[i], candidates, current, result);
```

### Mistake 2: Using `i` instead of `i+1` in Combination Sum II (allows illegal reuse)

```cpp
// WRONG: allows the same physical element to be used twice
backtrack(i, target - candidates[i], candidates, current, result);

// CORRECT: advance to i+1
backtrack(i + 1, target - candidates[i], candidates, current, result);
```

### Mistake 3: Using `i > 0` instead of `i > start` in the duplicate skip

```cpp
// WRONG: skips valid deeper usage of the second duplicate
if (i > 0 && candidates[i] == candidates[i-1]) continue;
// Problem: for [1,1,2] with target=4, at depth where start=1:
// i=1, 1(second) matches i>0 and arr[1]==arr[0] → wrongly skipped.
// This misses [1,1,2] as a valid combination.

// CORRECT
if (i > start && candidates[i] == candidates[i-1]) continue;
// i==start means "first element at this depth" → never skip.
// i>start means "sibling element at this depth" → skip if duplicate value.
```

### Mistake 4: Forgetting to sort before applying the duplicate skip

```cpp
// WRONG: skip rule fails on unsorted input because equal elements may not be adjacent
combinationSum2_wrong(candidates, target);  // missing sort

// CORRECT: sort is mandatory
sort(candidates.begin(), candidates.end());
```

### Mistake 5: Using `break` instead of `continue` for the duplicate skip

```cpp
// WRONG: breaks entire loop when a duplicate is found, stopping all remaining exploration
if (i > start && candidates[i] == candidates[i-1]) break;

// CORRECT: continue skips just this duplicate sibling, exploration continues at i+1
if (i > start && candidates[i] == candidates[i-1]) continue;
```

### Mistake 6: Recording result at wrong point in Subsets II

```cpp
// WRONG: only records at leaf (misses intermediate subsets)
void backtrack(int start, ...) {
    if (start == nums.size()) { result.push_back(current); return; }

// CORRECT: record at every node, before the for loop
void backtrack(int start, ...) {
    result.push_back(current);   // every prefix is a valid subset
    for (int i = start; ...)
```

### Mistake 7: Not handling empty `digits` in Letter Combinations

```cpp
// WRONG: if result starts as {""}, returns [""] for empty input instead of []
vector<string> result = {""};

// CORRECT: explicit guard at the start
if (digits.empty()) return {};
```

### Mistake 8: Using a `set` to deduplicate instead of the skip rule (causes TLE)

```cpp
// WRONG: O(N log(2^N)) per insertion into set — instant TLE on large inputs
set<vector<int>> seen;
void backtrack(...) {
    if (target==0) { seen.insert(current); return; }
    ...
}

// CORRECT: sort + skip rule prevents duplicates at generation time — O(1) per skip
if (i > start && candidates[i] == candidates[i-1]) continue;
```

### Mistake 9: Checking only one constraint in Combination Sum III

```cpp
// WRONG: records combinations regardless of count
if (remaining == 0) { result.push_back(current); return; }
// May include combinations with fewer than k elements.

// CORRECT: both conditions simultaneously
if ((int)current.size() == k && remaining == 0) {
    result.push_back(current);
    return;
}
```

---

## 13. Interview Reference: Patterns and Extensions

### Pattern Recognition Table

| Problem signal | Key structural decisions |
|---|---|
| "Combinations summing to target, reuse allowed" | Sort; loop from `start`; recurse with `i`; `break` when `candidates[i] > target` |
| "Combinations summing to target, each used once, duplicates in input" | Sort; loop from `start`; recurse with `i+1`; skip rule `i > start`; `break` when `candidates[i] > target` |
| "Exactly k numbers from 1–9 summing to n" | Loop `i` from `start` to 9; two-condition base case `size==k && sum==n` |
| "All subsets, input has duplicates" | Sort; record at every node before loop; skip rule `i > start` |
| "All subset sums" | Include-or-skip; collect sums at base; sort result |
| "Cartesian product per digit/position" | No skip; pure enumeration; record when index reaches end |

### Extensions and Harder Problems

**From Combination Sum I → Coin Change family:**
- LC 518 (Coin Change II): count ways to make amount (DP: `dp[amount] += dp[amount - coin]`). Same problem as Combo Sum I but counting, not enumerating.
- LC 322 (Coin Change): minimum coins (DP: `dp[amount] = min(dp[amount], dp[amount - coin] + 1)`).

**From Combination Sum II → Permutations II:**
- LC 47 (Permutations II): same duplicate-skip logic applied to permutations — sort and skip duplicates at each depth of the permutation tree.

**From Subsets II → Power Set problems:**
- Count distinct subset sums: unbounded knapsack DP where `dp[s] = true` if any subset sums to `s`.

**From Letter Combinations → Multi-domain enumeration:**
- Word search, decode ways with multiple valid interpretations per token — same Cartesian product structure.

### Backtracking Problem Taxonomy

```
Backtracking problems
├── Subset generation (record at every node)
│   ├── No duplicates → LC 78 (Subsets)
│   └── Duplicates → LC 90 (Subsets II): sort + skip rule
├── Combination generation (record only at leaves meeting constraint)
│   ├── Sum constraint
│   │   ├── Reuse allowed → LC 39 (Combo Sum I): recurse with i
│   │   ├── No reuse, duplicates → LC 40 (Combo Sum II): recurse with i+1, skip rule
│   │   └── Fixed count + sum → LC 216 (Combo Sum III): two-condition base case
│   └── Subset sums → GFG: include-or-skip, sort result
├── Permutation generation (track visited)
│   └── LC 46, 47 (Permutations I/II)
└── Cartesian product (no skip, one mandatory choice per position)
    └── LC 17 (Letter Combinations)
```

---

## 14. Complexity Cheat Sheet

| Problem | Time | Space | Notes |
|---|---|---|---|
| Subset Sums | O(2^n * n) | O(n) stack + O(2^n) result | Sorting the result adds n factor |
| Subsets II | O(n * 2^n) | O(n) stack | Sort first: O(n log n) |
| Combo Sum I | O(n^(t/m)) | O(t/m) stack | t=target, m=smallest candidate |
| Combo Sum II | O(2^n * k) | O(n) stack | k=avg combination length |
| Combo Sum III | O(C(9,k)) | O(k) stack | Fixed: choose k from 9 numbers |
| Letter Combinations | O(4^n * n) | O(n) stack | n=digits length; max 4 letters/digit |

---

## 15. Quick Revision Cheat Sheet

Self-contained for same-day interview revision.

---

### The `i` vs `i+1` Rule

```
Reuse allowed (Combo Sum I):     recurse with i     (same element can appear again)
No reuse (everything else):      recurse with i+1   (advance to next element)
```

---

### The Duplicate Skip Rule

```cpp
// Required for: Combo Sum II, Subsets II
// Prerequisite: sort the input first

if (i > start && arr[i] == arr[i - 1]) continue;

// i == start  means "first element at this recursion depth"  → always valid, never skip
// i > start   means "sibling element at this same depth"     → skip if same value as previous sibling
// This prevents duplicate subtrees at the same level without blocking deeper valid usage
```

---

### When to Record in Result

```
Subsets (every prefix is valid):  result.push_back(current) BEFORE the loop
Combinations (target at leaf):    record only at base case (target==0 or size==k)
```

---

### Per-Problem Summary

```cpp
// Subset Sums: include-or-skip, push sum at base, sort result after
void solve(int i, int sum, ...):
    if i==n: push sum; return
    solve(i+1, sum+arr[i], ...);  solve(i+1, sum, ...)

// Subsets II: sort first, push at every node, skip duplicate siblings
sort(nums). result.push_back(current) before loop.
if (i > start && nums[i] == nums[i-1]) continue;
backtrack(i+1, ...)

// Combo Sum I: sort for pruning, recurse with i for reuse
sort(candidates). if (candidates[i] > target) break.
backtrack(i, target - candidates[i], ...)   // NOTE: i not i+1

// Combo Sum II: sort mandatory, skip duplicates, recurse i+1
sort(candidates). if (i > start && candidates[i] == candidates[i-1]) continue.
if (candidates[i] > target) break.
backtrack(i+1, target - candidates[i], ...)

// Combo Sum III: loop 1..9, two-condition base
if (current.size()==k && remaining==0) record.
if (current.size()>=k || remaining<0) return.
for i=start to 9: if i>remaining: break.

// Letter Combinations: guard empty input, no skip, record at leaf
if (digits.empty()) return {}.
for each letter mapped to digits[index]: backtrack(index+1, ...)
```

---

### Complexity Summary

```
Subset Sums:        O(2^n * n)         O(n) stack
Subsets II:         O(n * 2^n)         O(n) stack
Combo Sum I:        O(n^(t/m))         O(t/m) stack
Combo Sum II:       O(2^n * k)         O(n) stack
Combo Sum III:      O(C(9,k))          O(k) stack
Letter Combos:      O(4^n * n)         O(n) stack
```

---

### Critical Rules

```
1. Always sort before applying the duplicate skip rule.
2. Use i > start, never i > 0, in the skip condition.
3. Use continue (not break) for the duplicate skip.
4. Use break (not continue) when candidates[i] > target in a sorted array.
5. Never use a set to deduplicate — it causes TLE on large inputs.
6. Guard: if (digits.empty()) return {} in Letter Combinations.
7. Two conditions simultaneously in Combo Sum III: size==k AND remaining==0.
```
