> Video References: [Arrays Part 2 — Striver](https://youtu.be/bYWLJb3vCWY) | [Arrays Part 3 — Striver](https://youtu.be/wvcQg43_V8U)
>
> Problems Covered: Linear Search | Move Zeroes | Union of Two Sorted Arrays | Missing Number | Max Consecutive Ones | Single Number

---

## Table of Contents

1. [The Unifying Mental Model](#1-the-unifying-mental-model)
2. [XOR — The Bit-Manipulation Foundation](#2-xor--the-bit-manipulation-foundation)
3. [Linear Search](#3-linear-search)
4. [Move Zeroes](#4-move-zeroes)
5. [Union of Two Sorted Arrays](#5-union-of-two-sorted-arrays)
6. [Missing Number](#6-missing-number)
7. [Max Consecutive Ones](#7-max-consecutive-ones)
8. [Single Number](#8-single-number)
9. [Complexity Cheat Sheet](#9-complexity-cheat-sheet)
10. [Common Mistakes and Edge Cases](#10-common-mistakes-and-edge-cases)
11. [Interview Patterns and Extensions](#11-interview-patterns-and-extensions)
12. [Quick Revision Cheat Sheet](#12-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Model

Every problem in this set is an instance of one of four fundamental traversal patterns. Recognizing which pattern applies is the primary skill — the code follows mechanically.

**Pattern 1 — Single-pointer scan with scalar state:** Walk the array once, maintaining one or two running values. No extra structure needed. Used in Linear Search and Max Consecutive Ones.

**Pattern 2 — Read/write two-pointer (same array):** A read pointer `i` visits every element. A write pointer `j` tracks where the next valid element should be placed. Used in Move Zeroes. This is also called the stable partition pattern.

**Pattern 3 — Merge-step two-pointer (two sorted arrays):** One pointer per array. Always advance the pointer pointing at the smaller front element. This is literally the merge step of merge sort. Used in Union of Two Sorted Arrays.

**Pattern 4 — Mathematical or bitwise reduction:** Combine all elements using a commutative, associative operation so that unwanted contributions cancel and the answer remains. Used in Missing Number (Gauss sum) and Single Number (XOR).

The key observation about Pattern 4 is that it converts an O(n) problem of "finding something" into an O(n) problem of "computing something" — no searching, just accumulation.

**The invariant principle:** Array problems solved in one pass always maintain an invariant — a statement about the array that is true after every iteration. Writing down what the invariant is before writing code is the most reliable way to get the algorithm right.

| Problem | Invariant during traversal |
|---|---|
| Linear Search | `result = -1` or the first matching index seen so far |
| Move Zeroes | `nums[0..j-1]` contains all non-zero elements encountered so far, in order |
| Union | `result` contains all distinct elements from `a[0..i-1]` and `b[0..j-1]`, sorted |
| Missing Number | `xorResult` = XOR of all indices and elements processed so far |
| Max Consecutive Ones | `curStreak` = length of the current unbroken run of 1s ending at the current position |
| Single Number | `result` = XOR of all elements seen so far |

---

## 2. XOR — The Bit-Manipulation Foundation

XOR appears in both Missing Number and Single Number. Its properties must be known without hesitation.

### XOR Truth Table

```
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0    ← same bits cancel
```

XOR produces a 1 in each bit position where the inputs **differ**, and a 0 where they are the **same**. The cancellation property (`1 ^ 1 = 0`) is why XOR is powerful for finding unpaired elements.

### The Four Properties You Must Know

| Property | Formula | Example |
|---|---|---|
| Self-cancellation | `a ^ a = 0` | `5 ^ 5 = 0` |
| Identity | `a ^ 0 = a` | `7 ^ 0 = 7` |
| Commutativity | `a ^ b = b ^ a` | `3 ^ 5 = 5 ^ 3` |
| Associativity | `(a ^ b) ^ c = a ^ (b ^ c)` | Order of grouping doesn't matter |

### Why These Properties Make XOR Powerful

If you XOR a multiset of integers where every value appears an **even** number of times, the result is 0 — all contributions cancel. If exactly one value appears an **odd** number of times, the result is that value. This works because XOR operates bit by bit: bit position k in the result is 1 if and only if an odd number of operands have bit k set.

Commutativity and associativity together mean the order of XOR operations does not matter at all. You can regroup and reorder freely when reasoning about correctness.

### XOR vs Arithmetic Sum — When to Prefer Which

XOR never overflows — it is a bitwise operation defined on the binary representation of integers, with no concept of carry beyond each bit. Arithmetic sum can overflow for large n. For the problems in this set the constraints are small enough that sum is safe, but XOR is always the safer choice and is equally efficient.

### Bit-Level Illustration

```
5 = 101 (binary)
3 = 011 (binary)
    ---
5 ^ 3 = 110 = 6
```

---

## 3. Linear Search

**Problem:** Given an array `arr` of n elements and a target `key`, return the index of the first occurrence of `key`, or -1 if absent. The array is unsorted.

**LeetCode equivalent:** Various; appears as a component in nearly every array problem.

### Intuition

Without sorted order or any additional structure, the element we are looking for could be at any position. An adversary can always arrange input so the key is at the last position checked. Therefore, in the worst case, any correct algorithm must examine every element. Linear search is asymptotically optimal for this problem.

### C++ Implementation

```cpp
int linearSearch(const vector<int>& arr, int key) {
    int n = arr.size();
    for (int i = 0; i < n; i++) {
        if (arr[i] == key) return i;
    }
    return -1;
}
```

### Dry Run

Input: `arr = [5, 3, 8, 1, 9, 2]`, `key = 8`

```
i=0: arr[0]=5, 5==8? No
i=1: arr[1]=3, 3==8? No
i=2: arr[2]=8, 8==8? Yes → return 2
```

Input: same array, `key = 7`

```
i=0..5: no match
Loop ends → return -1
```

### Complexity Analysis

| Case | Time | Condition |
|---|---|---|
| Best | O(1) | Key is at index 0 |
| Average | O(n) | Key is somewhere in the middle |
| Worst | O(n) | Key is at the last index or absent |

**Space:** O(1)

### Variants

**Find all occurrences (do not return early):**

```cpp
vector<int> linearSearchAll(const vector<int>& arr, int key) {
    vector<int> indices;
    for (int i = 0; i < (int)arr.size(); i++) {
        if (arr[i] == key) indices.push_back(i);
    }
    return indices;
}
```

**Find last occurrence:** Run the full loop and update `result` on every match, returning `result` after the loop.

**Sentinel optimization:** Place `key` at the end of the array before the loop. Then the loop termination condition is just `arr[i] == key` — no index bound check. This halves the number of comparisons per iteration at the cost of needing one extra slot and a pre-write.

---

## 4. Move Zeroes

**Problem:** Move all `0`s in `nums` to the end, maintaining the relative order of all non-zero elements. Do this **in-place** without making a copy.

**LeetCode 283 — Move Zeroes**

```
Input:  [0, 1, 0, 3, 12]
Output: [1, 3, 12, 0, 0]
```

### Approach 1 — Brute Force: Extra Array

Collect non-zeroes into a temp array, then copy back followed by zeros.

```cpp
void moveZeroesBrute(vector<int>& nums) {
    vector<int> temp;
    for (int x : nums)
        if (x != 0) temp.push_back(x);
    int i = 0;
    for (int x : temp) nums[i++] = x;
    while (i < (int)nums.size()) nums[i++] = 0;
}
```

**Time:** O(n) — two passes.
**Space:** O(n) — violates the in-place constraint.

### Approach 2 — Optimal: Two-Pointer Swap

**Intuition:** `j` is a write pointer that always points to the position of the leftmost zero (or to `i` if no zero has been seen). When we find a non-zero element at `i`, we swap it to position `j` and advance `j`. This moves the non-zero to its correct forward position and pushes the zero backward. Since `i` moves strictly left to right and we only swap when we find a non-zero, the relative order of non-zero elements is preserved.

When `j == i` (no zero has been encountered yet), the swap is a no-op — swapping an element with itself — so the algorithm is safe even when there are no zeros.

```cpp
void moveZeroes(vector<int>& nums) {
    int j = 0;  // write pointer: next position for a non-zero element
    for (int i = 0; i < (int)nums.size(); i++) {
        if (nums[i] != 0) {
            swap(nums[j], nums[i]);
            j++;
        }
    }
}
```

**Invariant after each iteration:** `nums[0..j-1]` contains all non-zero elements from `nums[0..i]`, in their original relative order. All positions from `j` onward are zeros or have not yet been finalized.

### Dry Run

Input: `[0, 1, 0, 3, 12]`

```
j=0
i=0: nums[0]=0, zero — skip.           [0, 1, 0, 3, 12]
i=1: nums[1]=1, non-zero.
     swap(nums[j=0], nums[i=1])         [1, 0, 0, 3, 12], j=1
i=2: nums[2]=0, zero — skip.           [1, 0, 0, 3, 12]
i=3: nums[3]=3, non-zero.
     swap(nums[j=1], nums[i=3])         [1, 3, 0, 0, 12], j=2
i=4: nums[4]=12, non-zero.
     swap(nums[j=2], nums[i=4])         [1, 3, 12, 0, 0], j=3

Result: [1, 3, 12, 0, 0]
```

**Time:** O(n) — single pass.
**Space:** O(1) — in-place.

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[0]` | `[0]` | j=0, i=0, zero — skip; nothing changes |
| `[1]` | `[1]` | j=0, i=0, non-zero; swap(0,0) is a no-op |
| `[1, 2, 3]` | `[1, 2, 3]` | No zeros; j==i always; all swaps are no-ops |
| `[0, 0, 0]` | `[0, 0, 0]` | All zeros; j never advances; no swaps |
| `[0, 0, 1]` | `[1, 0, 0]` | j=0 when i reaches 2; swap(0, 2) |

---

## 5. Union of Two Sorted Arrays

**Problem:** Given two sorted arrays `a` and `b` (possibly containing duplicates), return their union as a sorted array of distinct elements.

```
Input:  a = [1, 1, 2, 3, 4],  b = [2, 3, 4, 4, 5]
Output: [1, 2, 3, 4, 5]
```

### Approach 1 — Brute Force: Set

Insert all elements from both arrays into an ordered set (which handles deduplication and sorting), then dump to a vector.

```cpp
vector<int> unionBrute(const vector<int>& a, const vector<int>& b) {
    set<int> s(a.begin(), a.end());
    s.insert(b.begin(), b.end());
    return vector<int>(s.begin(), s.end());
}
```

**Time:** O((m+n) log(m+n)) — each set insertion is O(log k).
**Space:** O(m+n).

This approach is correct but ignores the fact that the inputs are already sorted — it does extra work to re-sort them via the set's internal structure.

### Approach 2 — Optimal: Two-Pointer Merge

**Intuition:** Since both arrays are sorted, the globally smallest unprocessed element is always at the front of one of them. Walk both arrays simultaneously using two pointers, always taking the smaller front. Skip duplicates by comparing each candidate against `result.back()` before adding.

This is the merge step of merge sort, extended with deduplication.

**Deduplication rule:** Before pushing any value to `result`, check: is `result` empty, or is the value different from `result.back()`? This single check eliminates duplicates both within one array and across the two arrays uniformly.

```cpp
vector<int> findUnion(const vector<int>& a, const vector<int>& b) {
    int m = a.size(), n = b.size();
    int i = 0, j = 0;
    vector<int> result;

    auto addIfNew = [&](int val) {
        if (result.empty() || result.back() != val)
            result.push_back(val);
    };

    while (i < m && j < n) {
        if (a[i] <= b[j]) addIfNew(a[i++]);
        else               addIfNew(b[j++]);
        // When a[i] == b[j], we take a[i] and advance i.
        // On the next iteration b[j] still holds the same value;
        // addIfNew will skip it because result.back() already equals it.
    }

    // Drain remaining elements
    while (i < m) addIfNew(a[i++]);
    while (j < n) addIfNew(b[j++]);

    return result;
}
```

**Why the equal case is handled correctly:** When `a[i] == b[j]`, the condition `a[i] <= b[j]` fires (taking `a[i]` and advancing `i`). On the next iteration, `b[j]` is still the same value. `addIfNew` checks `result.back() != b[j]` — they are equal, so the duplicate is skipped. `j` advances normally in a subsequent iteration when `b[j]` becomes smaller or on the drain. This is correct, verified by tracing:

```
a=[1,2], b=[2,3]:
  a[0]=1 <= b[0]=2 → addIfNew(1), i=1.  result=[1]
  a[1]=2 <= b[0]=2 → addIfNew(2), i=2.  result=[1,2]
  i==m, drain b:
    addIfNew(b[0]=2) → result.back()=2, skip. j=1.
    addIfNew(b[1]=3) → add.            result=[1,2,3]
```

### Dry Run

`a = [1, 1, 3, 5]`, `b = [1, 2, 3]`

```
i=0,j=0: a[0]=1 <= b[0]=1 → addIfNew(1) → add.    result=[1], i=1
i=1,j=0: a[1]=1 <= b[0]=1 → addIfNew(1) → skip.   i=2
i=2,j=0: a[2]=3 >  b[0]=1 → addIfNew(1) → skip.   j=1
i=2,j=1: a[2]=3 >  b[1]=2 → addIfNew(2) → add.    result=[1,2], j=2
i=2,j=2: a[2]=3 <= b[2]=3 → addIfNew(3) → add.    result=[1,2,3], i=3
i=3,j=2: a[3]=5 >  b[2]=3 → addIfNew(3) → skip.   j=3
j==n, drain a:
  addIfNew(5) → add.  result=[1,2,3,5]

Final result: [1, 2, 3, 5]
```

**Time:** O(m+n) — each element from both arrays is examined at most once.
**Space:** O(m+n) for the result array (output space, unavoidable).

### Edge Cases

| Scenario | Behavior |
|---|---|
| One array empty | The main while loop exits immediately; drain handles the other array |
| Both arrays identical | Every element past the first is skipped by `addIfNew`; each unique value appears once |
| No overlap between arrays | No equal-element case fires; straight merge |
| Consecutive duplicates within one array | `addIfNew` skips them since `result.back()` matches |

---

## 6. Missing Number

**Problem:** Given an array `nums` containing `n` distinct numbers from the range `[0, n]`, return the one number in that range that is missing.

**LeetCode 268 — Missing Number**

```
Input:  nums = [3, 0, 1]     (n=3, range [0..3])
Output: 2

Input:  nums = [9,6,4,2,3,5,7,0,1]  (n=9, range [0..9])
Output: 8
```

### Approach 1 — Sort and Scan

Sort the array. In a complete sorted range [0..n], index i should hold value i. Scan for the first mismatch.

```cpp
int missingNumberSort(vector<int> nums) {
    sort(nums.begin(), nums.end());
    for (int i = 0; i < (int)nums.size(); i++) {
        if (nums[i] != i) return i;
    }
    return nums.size(); // missing number is n itself
}
```

**Time:** O(n log n). **Space:** O(1) for in-place sort.

### Approach 2 — Hash Set

Mark every seen number. Scan [0..n] and return the first unmarked.

```cpp
int missingNumberHash(const vector<int>& nums) {
    unordered_set<int> seen(nums.begin(), nums.end());
    int n = nums.size();
    for (int i = 0; i <= n; i++) {
        if (!seen.count(i)) return i;
    }
    return -1; // unreachable
}
```

**Time:** O(n) average. **Space:** O(n).

### Approach 3 — Gauss Formula (Sum)

**Intuition:** The complete set [0..n] has a known sum: `n*(n+1)/2`. The actual array is missing exactly one element. The difference is the missing number.

```
missing = expectedSum - actualSum = n*(n+1)/2 - sum(nums)
```

```cpp
int missingNumberSum(const vector<int>& nums) {
    int n = nums.size();
    long long expected = (long long)n * (n + 1) / 2;
    long long actual = 0;
    for (int x : nums) actual += x;
    return (int)(expected - actual);
}
```

**Overflow note:** For the given constraint `n <= 10^4`, `n*(n+1)/2 <= 5*10^7`, which fits in a 32-bit int. Using `long long` costs nothing and is the safe default — always cast before multiplying when n could be larger.

**Time:** O(n). **Space:** O(1).

### Approach 4 — XOR (Optimal — No Overflow Risk)

**Intuition:** XOR all indices [0..n] and all elements of `nums`. Every number that appears in both sequences cancels with itself (`a ^ a = 0`). The missing number appears in the index sequence but not in `nums`, so it survives without a partner.

```
xorResult = (0 ^ 1 ^ ... ^ n) ^ (nums[0] ^ nums[1] ^ ... ^ nums[n-1])
           = missing
```

```cpp
int missingNumber(const vector<int>& nums) {
    int n = nums.size();
    int xorResult = n;  // start with the extra index n
    for (int i = 0; i < n; i++) {
        xorResult ^= i ^ nums[i];  // XOR index i and element at i together
    }
    return xorResult;
}
```

**Why starting with `n` and folding index and element into one loop works:** We need to XOR all of `{0, 1, ..., n}` and all of `nums`. Initializing with `n` accounts for the extra index. The loop handles indices `0..n-1` and simultaneously XORs the corresponding array elements. This is algebraically equivalent to the two-loop version.

**Time:** O(n). **Space:** O(1).

### Dry Run (XOR Approach)

Input: `nums = [3, 0, 1]`, n=3

```
xorResult = 3
i=0: xorResult ^= 0 ^ nums[0]=3  → 3^0^3=0
i=1: xorResult ^= 1 ^ nums[1]=0  → 0^1^0=1
i=2: xorResult ^= 2 ^ nums[2]=1  → 1^2^1=2

Return 2.

Verification: (0^1^2^3) ^ (3^0^1) = rearranging...
= (0^0) ^ (1^1) ^ 2 ^ (3^3) = 0^0^2^0 = 2  ✓
```

### Which Approach to Use in Interviews

| Priority | Approach | Reason |
|---|---|---|
| Primary answer | Sum formula | Simple to explain; O(n), O(1); Gauss formula is universally known |
| Follow-up / bonus | XOR | Shows bitwise fluency; eliminates overflow concern entirely |

---

## 7. Max Consecutive Ones

**Problem:** Given a binary array `nums`, return the maximum number of consecutive `1`s.

**LeetCode 485 — Max Consecutive Ones**

```
Input:  [1, 1, 0, 1, 1, 1]
Output: 3

Input:  [1, 0, 1, 1, 0, 1]
Output: 2
```

### Intuition

A `0` acts as a barrier that resets the current streak of `1`s. We need to track how long the current unbroken run of `1`s is and maintain the global maximum across all runs. A single pass suffices.

**The critical design choice** is when to update `maxStreak`. Updating inside the `if (x == 1)` branch handles both mid-array resets and trailing 1s at the end without any special post-loop logic. This is the safest implementation.

### C++ Implementation

```cpp
int findMaxConsecutiveOnes(const vector<int>& nums) {
    int curStreak = 0, maxStreak = 0;
    for (int x : nums) {
        if (x == 1) {
            curStreak++;
            maxStreak = max(maxStreak, curStreak);
        } else {
            curStreak = 0;
            // maxStreak is already up to date; no update needed here
        }
    }
    return maxStreak;
}
```

**Why not update `maxStreak` at the `else` (zero) branch?** Because the array might end with a streak of 1s. If the loop ends without hitting a 0, the last streak never gets compared. Updating inside `if (x==1)` avoids this entirely — the comparison happens the moment `curStreak` grows.

### Dry Run

Input: `[1, 1, 0, 1, 1, 1]`

```
x=1: curStreak=1, maxStreak=max(0,1)=1
x=1: curStreak=2, maxStreak=max(1,2)=2
x=0: curStreak=0
x=1: curStreak=1, maxStreak=max(2,1)=2
x=1: curStreak=2, maxStreak=max(2,2)=2
x=1: curStreak=3, maxStreak=max(2,3)=3
Return 3
```

Input: `[1, 1, 1, 1]` (all ones — trailing streak case)

```
x=1: curStreak=1, maxStreak=1
x=1: curStreak=2, maxStreak=2
x=1: curStreak=3, maxStreak=3
x=1: curStreak=4, maxStreak=4
Return 4   ← correct; no post-loop update needed
```

**Time:** O(n). **Space:** O(1).

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `[0]` | `0` | No 1s; `curStreak` never increments |
| `[1]` | `1` | Single 1 |
| `[1,1,1,1]` | `4` | All ones; `maxStreak` updated inside the loop |
| `[0,0,0]` | `0` | No 1s |
| `[0,1,0]` | `1` | Single 1 sandwiched by zeros |

---

## 8. Single Number

**Problem:** Every element in `nums` appears exactly twice except for one element which appears exactly once. Find that element. Must solve in O(n) time and O(1) space.

**LeetCode 136 — Single Number**

```
Input:  [2, 2, 1]        → Output: 1
Input:  [4, 1, 2, 1, 2]  → Output: 4
Input:  [1]              → Output: 1
```

### Approach 1 — Brute Force: Nested Loop

For each element, count its occurrences across the whole array. Return the one with count 1.

**Time:** O(n^2). **Space:** O(1).

### Approach 2 — Hash Map

Count frequencies. Return the element with frequency 1.

**Time:** O(n). **Space:** O(n). — violates the O(1) space constraint.

### Approach 3 — Optimal: XOR

**Intuition:** XOR all elements together. Every element that appears twice contributes `x ^ x = 0`. Since XOR is commutative and associative, we can regroup any order of operations, and all paired elements cancel. The single element, unpaired, survives as `x ^ 0 = x`.

```cpp
int singleNumber(const vector<int>& nums) {
    int result = 0;
    for (int x : nums) result ^= x;
    return result;
}
```

**Formal proof of correctness:**

Let the paired elements be `a1, a2, ..., ak` (each appearing twice) and the single element be `s`.

```
XOR of all elements = a1^a1 ^ a2^a2 ^ ... ^ ak^ak ^ s
                    = 0 ^ 0 ^ ... ^ 0 ^ s
                    = s
```

Commutativity and associativity guarantee that the physical order of elements in the array does not matter — we can always regroup to pair up duplicates.

### Dry Run

Input: `[4, 1, 2, 1, 2]`

```
result = 0
x=4: result = 0 ^ 4 = 4    (binary: 000 ^ 100 = 100)
x=1: result = 4 ^ 1 = 5    (binary: 100 ^ 001 = 101)
x=2: result = 5 ^ 2 = 7    (binary: 101 ^ 010 = 111)
x=1: result = 7 ^ 1 = 6    (binary: 111 ^ 001 = 110)
x=2: result = 6 ^ 2 = 4    (binary: 110 ^ 010 = 100)
Return 4

Pair cancellation view:
4 ^ (1^1) ^ (2^2) = 4 ^ 0 ^ 0 = 4  ✓
```

**Time:** O(n). **Space:** O(1).

### XOR Variations — Extensions

| Problem | What Changes | Approach |
|---|---|---|
| LC 137 — Single Number II | Every element appears 3 times except one | For each bit position, sum all bits modulo 3; surviving bits form the answer |
| LC 260 — Single Number III | Two elements appear once; rest appear twice | XOR all to get `a^b`; find any set bit (differing bit); partition array into two groups by that bit; XOR each group independently |
| LC 268 — Missing Number | Range [0..n], one missing | XOR indices [0..n] with all array elements |
| LC 389 — Find the Difference | Extra character added to shuffled string | XOR all characters of both strings |

---

## 9. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Linear Search | Single scan | O(n) | O(1) |
| Move Zeroes | Extra array (brute) | O(n) | O(n) |
| Move Zeroes | Two-pointer swap (optimal) | O(n) | O(1) |
| Union of Sorted Arrays | Set insertion (brute) | O((m+n) log(m+n)) | O(m+n) |
| Union of Sorted Arrays | Two-pointer merge (optimal) | O(m+n) | O(m+n) output |
| Missing Number | Sort and scan | O(n log n) | O(1) |
| Missing Number | Hash set | O(n) avg | O(n) |
| Missing Number | Gauss sum formula | O(n) | O(1) |
| Missing Number | XOR | O(n) | O(1) |
| Max Consecutive Ones | Single scan | O(n) | O(1) |
| Single Number | Nested loop (brute) | O(n^2) | O(1) |
| Single Number | Hash map | O(n) | O(n) |
| Single Number | XOR (optimal) | O(n) | O(1) |

---

## 10. Common Mistakes and Edge Cases

### Mistake 1 — Max Consecutive Ones: Updating `maxStreak` Only at Zeros

```cpp
// WRONG: if array ends with a streak of 1s, the loop exits without a 0
// to trigger the update, and maxStreak is never updated for the last run.
for (int x : nums) {
    if (x == 1) curStreak++;
    else { maxStreak = max(maxStreak, curStreak); curStreak = 0; }
}
return maxStreak;   // WRONG for input [1,1,1]: returns 0

// FIX OPTION A: update maxStreak inside the if(x==1) branch (preferred)
if (x == 1) { curStreak++; maxStreak = max(maxStreak, curStreak); }

// FIX OPTION B: add one line after the loop
maxStreak = max(maxStreak, curStreak);
```

### Mistake 2 — Union: Forgetting the Drain Loops

```cpp
// WRONG: stops after the main while loop; remaining elements from one
// array are silently dropped.
while (i < m && j < n) { ... }
return result;  // WRONG

// FIX: always include both drain loops
while (i < m) addIfNew(a[i++]);
while (j < n) addIfNew(b[j++]);
```

### Mistake 3 — Union: Advancing Only One Pointer When Elements Are Equal

```cpp
// WRONG: when a[i] == b[j], advancing only i leaves b[j] at the same
// value; in the addIfNew approach this is actually handled correctly,
// but in an explicit equal-branch this causes a duplicate:
if (a[i] == b[j]) {
    result.push_back(a[i]);
    i++;       // b[j] still holds the same value → added again in drain
}

// FIX when using an explicit equal-branch: advance both
if (a[i] == b[j]) { result.push_back(a[i]); i++; j++; }
// Or use the addIfNew pattern, which handles this automatically.
```

### Mistake 4 — Missing Number: Integer Overflow in Sum

```cpp
// RISKY for very large n: n*(n+1) overflows int before dividing by 2
int expected = n * (n + 1) / 2;

// SAFE: cast one operand to long long before multiplying
long long expected = (long long)n * (n + 1) / 2;
```

For `n <= 10^4` the constraint is safe with `int`, but the habit of casting matters for larger inputs.

### Mistake 5 — Move Zeroes: Using an Extra Array (Missing In-Place Requirement)

The problem explicitly requires in-place modification. An extra array solution is wrong on LeetCode even if it produces correct output in isolation. Always re-read constraints.

### Mistake 6 — Confusing `^` (XOR) with `|` (OR)

These are different operators with different truth tables. XOR cancels identical bits; OR accumulates them. Mixing them produces wrong results silently — the compiler will not warn you.

### Mistake 7 — Linear Search: Returning `0` for "Not Found" Instead of `-1`

Index 0 is a valid result (the key might be at position 0). The "not found" sentinel must be a value that cannot be a valid index — conventionally `-1`.

### Edge Cases Summary

| Problem | Critical Edge Cases |
|---|---|
| Linear Search | key not present (return -1), key at index 0, duplicate keys (return first) |
| Move Zeroes | all zeros (j never advances), no zeros (all no-op swaps), single element |
| Union | one empty array, both identical arrays, no overlap, all duplicates |
| Missing Number | missing is 0, missing is n, n=1 |
| Max Consecutive Ones | all zeros (return 0), all ones (update inside loop), single element |
| Single Number | n=1 (result = 0 ^ nums[0] = nums[0]), negative numbers (XOR works on signed ints) |

---

## 11. Interview Patterns and Extensions

### Pattern Recognition Table

| Signal in problem statement | Applicable Pattern | Key mechanism |
|---|---|---|
| Unsorted array, find element | Linear scan | Return on first match |
| In-place partition (zeros/negatives/condition) | Two-pointer read/write | `j` = write head; swap on valid element |
| Merge / combine two sorted arrays | Two-pointer merge | Compare fronts; `addIfNew` for dedup |
| Range [0..n], one number missing | Mathematical / XOR reduction | Gauss sum or XOR cancellation |
| Binary array, longest run | Counter with reset | `curStreak`, `maxStreak` |
| All values appear even times except one | XOR | Pair cancellation |

### Extensions to Know

**Move Zeroes → Sort Colors (Dutch National Flag, LC 75):** Three-way partition with `lo`, `mid`, `hi` pointers. Same read/write invariant, extended to three regions (red/white/blue or 0/1/2).

**Union of Two → Merge K Sorted Arrays (LC 23):** Use a min-heap of size K. Each entry holds (value, arrayIndex, elementIndex). Pop minimum, push next from that array. O(n log k) total.

**Missing Number → Find All Missing Numbers (LC 448):** For each `nums[i]`, negate `nums[abs(nums[i])-1]`. Indices with still-positive values are missing. O(n) time, O(1) space.

**Missing Number → Find Duplicate (LC 287):** Use Floyd's cycle detection on the array as a linked list, treating each value as a "next" pointer. O(n) time, O(1) space.

**Single Number → Single Number III (LC 260):** Two distinct singles; rest appear twice. XOR all to get `a^b`. Find any set bit (where a and b differ). Partition all numbers into two groups by that bit and XOR each group independently. Each group reduces to one of the two singles.

**Max Consecutive Ones → Max Consecutive Ones III (LC 1004):** Allow flipping at most K zeros. Sliding window: expand right; when the window contains more than K zeros, shrink from the left. O(n), O(1).

---

## 12. Quick Revision Cheat Sheet

This section is self-sufficient for last-minute review.

### XOR Properties (Must Know Cold)

```
a ^ a = 0    (self-cancellation)
a ^ 0 = a    (identity)
Commutative: a ^ b = b ^ a
Associative: (a ^ b) ^ c = a ^ (b ^ c)
```

### Linear Search

```cpp
for (int i = 0; i < n; i++)
    if (arr[i] == key) return i;
return -1;
```
O(n) time, O(1) space. Optimal for unsorted arrays.

### Move Zeroes (Two-Pointer Swap)

```cpp
int j = 0;
for (int i = 0; i < n; i++)
    if (nums[i] != 0) { swap(nums[j], nums[i]); j++; }
```
O(n) time, O(1) space. Stable (relative order of non-zeros preserved). Swap is no-op when j==i.

### Union of Two Sorted Arrays

```cpp
auto addIfNew = [&](int v) {
    if (result.empty() || result.back() != v) result.push_back(v);
};
while (i < m && j < n) {
    if (a[i] <= b[j]) addIfNew(a[i++]);
    else               addIfNew(b[j++]);
}
while (i < m) addIfNew(a[i++]);
while (j < n) addIfNew(b[j++]);
```
O(m+n) time. When a[i]==b[j], take a[i]; b[j] is skipped by addIfNew next iteration.

### Missing Number

```cpp
// Sum formula (simpler to explain):
return (int)((long long)n*(n+1)/2 - accumulate(nums.begin(),nums.end(),0LL));

// XOR (no overflow risk):
int x = n;
for (int i = 0; i < n; i++) x ^= i ^ nums[i];
return x;
```
Both O(n) time, O(1) space.

### Max Consecutive Ones

```cpp
int cur = 0, maxi = 0;
for (int x : nums) {
    if (x == 1) { cur++; maxi = max(maxi, cur); }
    else cur = 0;
}
return maxi;
```
O(n) time, O(1) space. Updating `maxi` inside `if(x==1)` handles trailing 1s automatically.

### Single Number

```cpp
int result = 0;
for (int x : nums) result ^= x;
return result;
```
O(n) time, O(1) space. Pairs cancel (`a^a=0`); single survives (`x^0=x`).

### Complexity at a Glance

| Problem | Optimal Time | Optimal Space |
|---|---|---|
| Linear Search | O(n) | O(1) |
| Move Zeroes | O(n) | O(1) |
| Union of Sorted Arrays | O(m+n) | O(m+n) output |
| Missing Number | O(n) | O(1) |
| Max Consecutive Ones | O(n) | O(1) |
| Single Number | O(n) | O(1) |

### Core Invariants (The "Why" Behind Each Algorithm)

| Algorithm | Invariant |
|---|---|
| Move Zeroes | `nums[0..j-1]` contains all non-zero elements seen so far, in original order |
| Union merge | `result` contains all distinct elements from both arrays up to the current pointers, sorted |
| Missing Number (XOR) | `xorResult` = XOR of all indices and array elements processed so far; unpaired index survives |
| Max Consecutive Ones | `curStreak` = length of the current unbroken run of 1s ending at current position |
| Single Number | `result` = XOR of all elements seen; paired elements cancel as they are encountered |

---
