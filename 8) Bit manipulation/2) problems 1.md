
**Problems covered:**
- [LC 2220 — Minimum Bit Flips to Convert Number](https://leetcode.com/problems/minimum-bit-flips-to-convert-number/)
- [LC 136 — Single Number](https://leetcode.com/problems/single-number/)
- [LC 78 — Subsets](https://leetcode.com/problems/subsets/)
- [GFG — Find XOR of Numbers from L to R](https://www.geeksforgeeks.org/problems/find-xor-of-numbers-from-l-to-r/1)
- [LC 260 — Single Number III](https://leetcode.com/problems/single-number-iii/)
- [GFG — Finding the Numbers (Two Odd Occurring)](https://www.geeksforgeeks.org/problems/finding-the-numbers0215/1)
- [GFG — Prime Factors](https://www.geeksforgeeks.org/problems/prime-factors5052/1)
- [GFG — Sum of All Divisors from 1 to N](https://www.geeksforgeeks.org/problems/sum-of-all-divisors-from-1-to-n4738/1)
- [LC 204 — Count Primes](https://leetcode.com/problems/count-primes/)
- [LC 50 — Pow(x, n)](https://leetcode.com/problems/powx-n/)

**Striver Videos:**
- [Bit Manipulation Basics](https://youtu.be/OOdrmcfZXd8) | [Single Number / XOR tricks](https://youtu.be/LqKaUv1G3_I) | [Subsets via Bit Masking](https://youtu.be/sFBCAl8yBfE) | [Single Number III](https://youtu.be/UA5JnV1J2sI) | [Prime Factors](https://youtu.be/WqGb7159h7Q) | [Sum of Divisors 1 to N](https://youtu.be/nttpF8kwgd4) | [Count Primes / Sieve](https://youtu.be/5Bb2nqA40JY)

---

## Table of Contents

1. [Bit Manipulation Fundamentals](#1-bit-manipulation-fundamentals)
2. [Number Theory Fundamentals](#2-number-theory-fundamentals)
3. [Problem 1 — Minimum Bit Flips to Convert Number (LC 2220)](#3-problem-1--minimum-bit-flips-to-convert-number-lc-2220)
4. [Problem 2 — Single Number (LC 136)](#4-problem-2--single-number-lc-136)
5. [Problem 3 — Subsets via Bit Masking (LC 78)](#5-problem-3--subsets-via-bit-masking-lc-78)
6. [Problem 4 — XOR from L to R (GFG)](#6-problem-4--xor-from-l-to-r-gfg)
7. [Problem 5 — Single Number III (LC 260)](#7-problem-5--single-number-iii-lc-260)
8. [Problem 6 — Two Numbers with Odd Occurrences (GFG)](#8-problem-6--two-numbers-with-odd-occurrences-gfg)
9. [Problem 7 — Prime Factors (GFG)](#9-problem-7--prime-factors-gfg)
10. [Problem 8 — Sum of All Divisors from 1 to N (GFG)](#10-problem-8--sum-of-all-divisors-from-1-to-n-gfg)
11. [Problem 9 — Count Primes (LC 204)](#11-problem-9--count-primes-lc-204)
12. [Problem 10 — Pow(x, n) (LC 50)](#12-problem-10--powx-n-lc-50)
13. [Related Concepts and Extensions](#13-related-concepts-and-extensions)
14. [Common Mistakes and Pitfalls](#14-common-mistakes-and-pitfalls)
15. [Interview Reference — Pattern Map](#15-interview-reference--pattern-map)
16. [Complexity Cheat Sheet](#16-complexity-cheat-sheet)
17. [Quick Revision Cheat Sheet](#17-quick-revision-cheat-sheet)

---

## 1. Bit Manipulation Fundamentals

### The Six Core Operators

| Operator | Symbol | Rule | Example (4-bit) |
|---|---|---|---|
| AND | `&` | 1 only if both bits are 1 | `0110 & 1010 = 0010` |
| OR | `\|` | 1 if either bit is 1 | `0110 \| 1010 = 1110` |
| XOR | `^` | 1 if bits differ | `0110 ^ 1010 = 1100` |
| NOT | `~` | Flip all bits | `~0110 = ...11111001` (two's complement) |
| Left shift | `<<` | Multiply by 2^k | `0110 << 2 = 011000` |
| Right shift | `>>` | Divide by 2^k (floor for non-negative) | `1100 >> 2 = 0011` |

### XOR Properties — The Core Identity Toolkit

These five identities underlie almost every XOR-based problem:

```
a ^ a = 0          (self-cancellation)
a ^ 0 = a          (identity element)
a ^ b = b ^ a      (commutative)
(a^b)^c = a^(b^c)  (associative)
a ^ b ^ a = b      (double XOR cancels: a^b^a = (a^a)^b = 0^b = b)
```

Because XOR is commutative and associative, you can freely reorder and regroup any XOR expression. This means paired elements cancel regardless of where they appear in the sequence.

### Essential Bit Tricks

```cpp
// Check if bit i is set in x
bool isSet = (x >> i) & 1;

// Set bit i
x |= (1 << i);

// Clear bit i
x &= ~(1 << i);

// Toggle bit i
x ^= (1 << i);

// Count set bits (popcount / Hamming weight)
__builtin_popcount(x);         // GCC built-in, O(1), treats x as unsigned
__builtin_popcountll(x);       // for long long

// Isolate rightmost (lowest) set bit
int lsb = x & (-x);           // two's complement: -x = ~x + 1
// Proof: if x = ...abc1000, then -x = ...xyz1000 (only rightmost 1 and trailing zeros survive AND)

// Clear rightmost set bit
x &= (x - 1);                  // Brian Kernighan's step

// Count set bits manually (Brian Kernighan) — O(popcount(x))
int cnt = 0;
while (x) { x &= (x - 1); cnt++; }

// Check if x is a power of 2
bool isPow2 = x > 0 && (x & (x - 1)) == 0;

// Iterate over all non-empty subsets of bitmask m
for (int s = m; s > 0; s = (s - 1) & m) { /* process s */ }
```

### Two's Complement and `~x`

In C++ with 32-bit `int`: `~x = -(x + 1)`. So `~0 = -1`, `~3 = -4`. This means `x & (-x)` isolates the lowest set bit: since `-x = ~x + 1`, the addition propagates a carry that flips all trailing zeros back to zero and leaves only the rightmost 1 unchanged.

---

## 2. Number Theory Fundamentals

### Why √n is the Divisor Threshold

**Claim:** If `d` is a divisor of `n` and `d > √n`, then `n/d < √n` is also a divisor. So divisors come in pairs `(d, n/d)`, at least one of which is ≤ √n.

**Proof:** Suppose `n = d * q`. If both `d > √n` and `q > √n`, then `d * q > n` — contradiction. So at least one of them must be ≤ √n. ∎

This reduces trial division from O(n) to O(√n).

### Sieve of Eratosthenes

Find all primes ≤ n in O(n log log n):

1. `isPrime[0..n]` = all true; set `isPrime[0] = isPrime[1] = false`.
2. For each `p` from 2 to √n: if `isPrime[p]`, mark multiples `p*p, p*p+p, ...` as false.
3. All remaining `true` entries are prime.

**Why start marking from `p*p`?** Every multiple `k*p` for `k < p` has a factor `k < p`, so it was already marked by that smaller prime. Starting from `p^2` avoids all redundant work.

**Time:** O(n log log n) — from Mertens' theorem: the sum `n/2 + n/3 + n/5 + n/7 + ...` over all primes converges to O(n log log n).

### Smallest Prime Factor (SPF) Sieve

For fast prime factorization of many numbers ≤ N:

```cpp
const int MAXN = 1e6 + 1;
int spf[MAXN];

void buildSPF() {
    for (int i = 0; i < MAXN; i++) spf[i] = i;
    for (int i = 2; (long long)i * i < MAXN; i++) {
        if (spf[i] == i) {  // i is prime
            for (int j = i * i; j < MAXN; j += i)
                if (spf[j] == j) spf[j] = i;
        }
    }
}

// Factorize n in O(log n) after O(N log log N) precomputation
vector<int> primeFactors_spf(int n) {
    vector<int> factors;
    while (n > 1) { factors.push_back(spf[n]); n /= spf[n]; }
    return factors;
}
```

---

## 3. Problem 1 — Minimum Bit Flips to Convert Number (LC 2220)

**Problem:** Given integers `start` and `goal`, return the minimum number of single-bit flips to convert `start` to `goal`.

**Constraints:** `0 <= start, goal <= 10^9`.

**Examples:** `start=10` (1010), `goal=7` (0111) → `3`. `start=3` (011), `goal=4` (100) → `3`.

### Core Insight

A bit needs to be flipped if and only if it differs between `start` and `goal`. XOR outputs 1 at exactly the positions where its inputs differ. Therefore:

**Minimum flips = Hamming distance = `__builtin_popcount(start ^ goal)`.**

This is the definition of Hamming distance: the number of positions at which two bit strings differ.

### Approaches

**Brute force — bit-by-bit:**

```cpp
int minBitFlips_brute(int start, int goal) {
    int flips = 0;
    for (int i = 0; i < 32; i++)
        if (((start >> i) & 1) != ((goal >> i) & 1)) flips++;
    return flips;
}
```

**Optimal — XOR + popcount:**

```cpp
int minBitFlips(int start, int goal) {
    return __builtin_popcount(start ^ goal);
}
```

**Time:** O(1). **Space:** O(1).

### Dry Run

`start=10` (1010), `goal=7` (0111):
```
start ^ goal = 1010 ^ 0111 = 1101   (bits 0, 2, 3 differ)
popcount(1101) = 3
Answer: 3 ✓
```

**Edge cases:**

| Input | Expected | Reason |
|---|---|---|
| `start == goal` | 0 | `x ^ x = 0`, popcount(0) = 0 |
| `start=1, goal=2` | 2 | `01 ^ 10 = 11`, 2 bits differ |
| `start=0, goal=INT_MAX` | 31 | All 31 non-sign bits differ |

---

## 4. Problem 2 — Single Number (LC 136)

**Problem:** Every element in a non-empty array appears exactly twice except one. Find the unique element. Must be O(n) time and O(1) space.

**Constraints:** `1 <= nums.length <= 3*10^4`, values in `[-3*10^4, 3*10^4]`.

**Examples:** `[2,2,1]` → `1`. `[4,1,2,1,2]` → `4`.

### Approach: XOR of All Elements

**Intuition:** XOR all elements. Every pair cancels to 0 (`a ^ a = 0`). The unique element XORed with 0 survives.

**Formal proof:** Let the unique element be `x` and pairs be `a1, a1, ..., ak, ak`. XOR of all = `x ^ (a1^a1) ^ ... ^ (ak^ak) = x ^ 0 ^ ... ^ 0 = x`. ∎

```cpp
int singleNumber(vector<int>& nums) {
    int result = 0;
    for (int x : nums) result ^= x;
    return result;
}
```

**Time:** O(n). **Space:** O(1).

### Dry Run

`[4, 1, 2, 1, 2]`:
```
0 ^ 4 = 4
  ^ 1 = 5   (100 ^ 001 = 101)
  ^ 2 = 7   (101 ^ 010 = 111)
  ^ 1 = 6   (111 ^ 001 = 110)
  ^ 2 = 4   (110 ^ 010 = 100)
Answer: 4 ✓
```

Because XOR is commutative and associative, we can regroup: `4 ^ (1^1) ^ (2^2) = 4 ^ 0 ^ 0 = 4`. The arrival order is irrelevant.

---

## 5. Problem 3 — Subsets via Bit Masking (LC 78)

**Problem:** Given an array of **unique** elements, return all possible subsets (power set).

**Constraints:** `1 <= nums.length <= 10`, all elements distinct.

**Example:** `[1,2,3]` → `[[], [1], [2], [1,2], [3], [1,3], [2,3], [1,2,3]]`.

### The Bijection Between Binary Numbers and Subsets

For `n` elements, every subset is uniquely identified by an n-bit bitmask: bit `i` = 1 means "include `nums[i]`". Iterating mask from 0 (empty set) to `2^n - 1` (full set) generates every subset exactly once.

### Approach 1 — Recursive Backtracking

```cpp
void backtrack(vector<int>& nums, int start, vector<int>& cur, vector<vector<int>>& res) {
    res.push_back(cur);
    for (int i = start; i < (int)nums.size(); i++) {
        cur.push_back(nums[i]);
        backtrack(nums, i + 1, cur, res);
        cur.pop_back();
    }
}
vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> res; vector<int> cur;
    backtrack(nums, 0, cur, res);
    return res;
}
```

### Approach 2 — Bit Masking (Iterative, Optimal)

```cpp
vector<vector<int>> subsets(vector<int>& nums) {
    int n = nums.size();
    int total = 1 << n;   // 2^n subsets
    vector<vector<int>> result;
    result.reserve(total);
    for (int mask = 0; mask < total; mask++) {
        vector<int> subset;
        for (int i = 0; i < n; i++)
            if (mask & (1 << i))   // bit i set → include nums[i]
                subset.push_back(nums[i]);
        result.push_back(subset);
    }
    return result;
}
```

**Time:** O(2^n * n). **Space:** O(2^n * n) for output.

### Dry Run — `nums=[1,2,3]`, n=3

```
mask=0 (000): no bits → []
mask=1 (001): bit 0   → [1]
mask=2 (010): bit 1   → [2]
mask=3 (011): bits 0,1 → [1,2]
mask=4 (100): bit 2   → [3]
mask=5 (101): bits 0,2 → [1,3]
mask=6 (110): bits 1,2 → [2,3]
mask=7 (111): bits 0,1,2 → [1,2,3]
```

All 8 subsets generated. ✓

**Why bit masking is preferred for interviews:** No recursion stack to trace, no backtracking logic to explain. The bijection between binary numbers and subsets is a clean mathematical argument.

**Limitation:** Works for `n ≤ 30` with `int` (use `1 << i`), or `n ≤ 63` with `long long` (use `1LL << i`). For `n > 63`, use backtracking.

---

## 6. Problem 4 — XOR from L to R (GFG)

**Problem:** Find `L ^ (L+1) ^ ... ^ R`. Expected O(1) time.

**Constraints:** `1 <= L <= R <= 10^9`.

**Examples:** `L=4, R=8` → `8`. `L=2, R=4` → `5`.

### Approach 1 — Brute Force Linear Loop

```cpp
int findXOR_brute(int L, int R) {
    int ans = 0;
    for (int i = L; i <= R; i++) ans ^= i;
    return ans;
}
```

**Time:** O(R - L + 1). For R = 10^9: ~10^9 iterations — infeasible.

### Approach 2 — O(1) Using Prefix XOR and the N%4 Pattern

**Key observation:** XOR from 1 to N follows a repeating 4-cycle.

**Pattern:**

| N mod 4 | XOR(1..N) |
|---|---|
| 0 | N |
| 1 | 1 |
| 2 | N + 1 |
| 3 | 0 |

**Proof of the pattern:** Consider consecutive integers in groups of 4: {4k+1, 4k+2, 4k+3, 4k+4}.

The last two bits of these four are 01, 10, 11, 00. Their XOR on the last two bits: `01^10^11^00 = 00`. The higher bits all equal 4k, so they XOR to zero as well (even count). Each complete group of 4 XORs to 0.

Therefore XOR(1..N) depends only on the remainder of N elements after complete groups of 4:
```
Verify:
XOR(1..4) = 1^2^3^4 = (1^2)^(3^4) = 3^7 = 4 = N ✓   (N%4=0)
XOR(1..5) = 4^5 = 1                                   ✓   (N%4=1)
XOR(1..6) = 1^6 = 7 = N+1                             ✓   (N%4=2)
XOR(1..7) = 7^7 = 0                                   ✓   (N%4=3)
XOR(1..8) = 0^8 = 8 = N                               ✓   (N%4=0)
```

**Prefix XOR to compute range:** `XOR(L..R) = XOR(1..R) ^ XOR(1..L-1)`.

This works because XOR is its own inverse: XOR-ing the prefix through R and the prefix through L-1 cancels all elements from 1 to L-1, leaving exactly L through R.

**Why `L-1` and not `L`:** We want L included. `XOR(1..R) ^ XOR(1..L-1)` cancels elements 1 through L-1, leaving L through R — correct. Using `XOR(1..L)` would cancel L itself, giving us (L+1) through R — wrong by one.

```cpp
int xorUpTo(int n) {
    if (n < 0) return 0;
    switch (n % 4) {
        case 0: return n;
        case 1: return 1;
        case 2: return n + 1;
        case 3: return 0;
    }
    return 0;
}

int findXOR(int L, int R) {
    return xorUpTo(R) ^ xorUpTo(L - 1);
}
```

**Time:** O(1). **Space:** O(1).

### Dry Runs

`L=4, R=8`:
```
xorUpTo(8): 8%4=0 → 8
xorUpTo(3): 3%4=3 → 0
Result: 8 ^ 0 = 8 ✓
```

`L=2, R=4`:
```
xorUpTo(4): 4%4=0 → 4
xorUpTo(1): 1%4=1 → 1
Result: 4 ^ 1 = 5 ✓   (2^3^4 = 5^4 = 1... wait: 2^3=1, 1^4=5) ✓
```

---

## 7. Problem 5 — Single Number III (LC 260)

**Problem:** In an array, exactly two elements appear once; all others appear exactly twice. Find both unique elements in O(n) time and O(1) space.

**Constraints:** `2 <= nums.length <= 3*10^4`, values in `[-2^31, 2^31-1]`.

**Examples:** `[1,2,1,3,2,5]` → `[3,5]`. `[-1,0]` → `[-1,0]`.

### Why Single Number I's Trick Alone Fails

XOR-ing all elements gives `xorAB = a ^ b`. But this is a mixed value — neither `a` nor `b` — so we cannot recover either directly. We need to separate the two unique elements into independent subproblems.

### The Two-Step Algorithm

**Step 1:** XOR all elements → `xorAB = a ^ b`. Since `a ≠ b`, `xorAB ≠ 0`, so at least one bit is set.

**Step 2:** Find any bit position where `a` and `b` differ. That bit is set in `xorAB`. Use it to partition all elements into two groups:
- Group 1: elements with that bit set.
- Group 2: elements with that bit unset.

Since `a` and `b` differ at this bit, they land in different groups. Every paired element falls in the same group as its twin (both have the same value at that bit). XOR-ing each group independently cancels all pairs, leaving one unique element per group.

**Isolating the rightmost set bit:** `lsb = xorAB & (-xorAB)`.

```cpp
vector<int> singleNumber(vector<int>& nums) {
    // Step 1: XOR all → a ^ b
    int xorAB = 0;
    for (int x : nums) xorAB ^= x;

    // Step 2: isolate rightmost set bit where a and b differ
    int lsb = xorAB & (-xorAB);

    // Step 3: partition into two groups; XOR each group
    int a = 0, b = 0;
    for (int x : nums) {
        if (x & lsb) a ^= x;
        else          b ^= x;
    }
    return {a, b};
}
```

**Time:** O(n). **Space:** O(1) extra.

### Dry Run — `[1, 2, 1, 3, 2, 5]`

```
Step 1: XOR all
  0^1=1, ^2=3, ^1=2, ^3=1, ^2=3, ^5=6
  xorAB = 6 = 110

Step 2: isolate LSB
  -6 in two's complement:
    6  = ...0110
    -6 = ...1010  (flip: ...1001, +1: ...1010)
  lsb = 6 & (-6) = 0110 & 1010 = 0010 = 2  (bit 1)

Step 3: partition by bit 1
  1 (001): bit1=0 → b ^= 1 → b=1
  2 (010): bit1=1 → a ^= 2 → a=2
  1 (001): bit1=0 → b ^= 1 → b=0
  3 (011): bit1=1 → a ^= 3 → a=1  (010^011=001)
  2 (010): bit1=1 → a ^= 2 → a=3  (001^010=011)
  5 (101): bit1=0 → b ^= 5 → b=5

  a=3, b=5
Answer: [3, 5] ✓
```

---

## 8. Problem 6 — Two Numbers with Odd Occurrences (GFG)

**Problem:** In an unsorted array where all numbers except two appear an even number of times, find the two numbers that appear an odd number of times. Return in decreasing order.

**Constraints:** `2 <= arr.size() <= 10^6`, `1 <= arr[i] <= 10^8`.

**Examples:** `[12,23,34,12,12,23,12,45]` → `[45,34]`. `[2,3,3,4,4,5]` → `[5,2]`.

**This is structurally identical to Single Number III.** The only differences are output order (decreasing) and that elements may appear any even number of times (4, 6, 8, ...). XOR cancels any even count: `a ^ a ^ a ^ a = (a^a) ^ (a^a) = 0 ^ 0 = 0`. The algorithm is identical.

```cpp
vector<long long> twoOddNum(vector<long long> arr) {
    long long xorAll = 0;
    for (long long x : arr) xorAll ^= x;

    long long lsb = xorAll & (-xorAll);

    long long a = 0, b = 0;
    for (long long x : arr) {
        if (x & lsb) a ^= x;
        else          b ^= x;
    }

    if (a < b) swap(a, b);   // return in decreasing order
    return {a, b};
}
```

**Time:** O(n). **Space:** O(1).

---

## 9. Problem 7 — Prime Factors (GFG)

**Problem:** Given N, find all distinct prime factors in sorted (ascending) order.

**Constraints:** `1 <= N <= 10^6` (GFG); algorithm works for single queries up to 10^9.

**Examples:** `N=12` → `[2,3]`. `N=100` → `[2,5]`. `N=13` → `[13]`.

### Approach 1 — Naive: Trial Division up to N

```cpp
vector<int> primeFactors_naive(int n) {
    vector<int> factors;
    for (int i = 2; i <= n; i++) {
        if (n % i == 0) {
            factors.push_back(i);
            while (n % i == 0) n /= i;
        }
    }
    return factors;
}
```

**Time:** O(n).

### Approach 2 — Trial Division up to √N (Optimal for Single Query)

**Key insight:** After dividing out all prime factors ≤ √n, if the remaining `n > 1`, it must itself be prime (it has no factors other than 1 and itself, since all smaller prime factors have been divided out). This remaining prime is > √(original n) and can appear at most once (since its square exceeds the original n).

```cpp
vector<int> primeFactors(int n) {
    vector<int> factors;

    // Handle factor 2 separately; lets us step by 2 in the main loop
    if (n % 2 == 0) {
        factors.push_back(2);
        while (n % 2 == 0) n /= 2;
    }

    // Check odd factors from 3 to √n
    for (int i = 3; (long long)i * i <= n; i += 2) {
        if (n % i == 0) {
            factors.push_back(i);
            while (n % i == 0) n /= i;   // divide out all copies
        }
    }

    // If n > 1 remains, it is a prime factor > √(original n)
    if (n > 1) factors.push_back(n);

    return factors;
}
```

**Time:** O(√N). **Space:** O(log N) for output (at most log₂(N) distinct prime factors).

### Dry Runs

`N=100`:
```
n=100. 100%2=0 → push 2. Divide: 100→50→25. n=25.
i=3: 25%3≠0.
i=5: 5*5=25 ≤ 25, 25%5=0 → push 5. Divide: 25→5→1. n=1.
i=7: 7*7=49 > 1 → loop exits.
n=1, not > 1 → no extra factor.
Result: [2, 5] ✓
```

`N=13` (prime):
```
13%2≠0.
i=3: 3*3=9 ≤ 13, 13%3≠0.
i=5: 5*5=25 > 13 → loop exits.
n=13 > 1 → push 13.
Result: [13] ✓
```

### Approach 3 — SPF Sieve for Multiple Queries

If factorizing many numbers ≤ N: precompute the SPF sieve (Section 2), factorize in O(log n) per query. Total: O(N log log N) precompute + O(log n) per query.

### When to Use Which Approach

| Scenario | Approach | Time |
|---|---|---|
| Single query, N ≤ 10^9 | Trial division to √N | O(√N) |
| Many queries, N ≤ 10^6 | SPF sieve | O(N log log N) precompute + O(log N) per query |

---

## 10. Problem 8 — Sum of All Divisors from 1 to N (GFG)

**Problem:** Compute `F(1) + F(2) + ... + F(N)` where `F(i)` = sum of all divisors of `i`.

**Constraints:** `1 <= N <= 10^6`.

**Examples:** `N=4` → `15` (F(1)=1, F(2)=3, F(3)=4, F(4)=7). `N=5` → `21`.

### The Contribution Trick

Instead of computing `F(i)` for each `i`, ask: **how many times does divisor `d` contribute to the total sum?**

Divisor `d` is a factor of all multiples of `d` in `[1, N]`: namely `d, 2d, 3d, ..., floor(N/d)*d`. There are `floor(N/d)` such multiples. So `d` contributes `d * floor(N/d)` to the total.

Total sum = `sum over d from 1 to N of d * floor(N/d)`.

### Approach 1 — Naive (O(N√N), TLE)

For each `i` from 1 to N, find all divisors of `i` by trial division up to √i and sum them.

**Time:** O(N√N). For N=10^6: ~10^9 operations — TLE.

### Approach 2 — Contribution Trick (O(N))

```cpp
long long sumDivisors(int N) {
    long long sum = 0;
    for (int d = 1; d <= N; d++)
        sum += (long long)d * (N / d);   // cast before multiply to avoid overflow
    return sum;
}
```

**Time:** O(N). **Space:** O(1).

### Dry Run — `N=4`

```
d=1: 1 * (4/1) = 1 * 4 = 4
d=2: 2 * (4/2) = 2 * 2 = 4
d=3: 3 * (4/3) = 3 * 1 = 3
d=4: 4 * (4/4) = 4 * 1 = 4
Total = 4 + 4 + 3 + 4 = 15 ✓
```

**Verification via definition:** F(1)=1, F(2)=1+2=3, F(3)=1+3=4, F(4)=1+2+4=7. Sum = 1+3+4+7 = 15. ✓

**Why `(long long)d * (N/d)` not `d * (N/d)`?** The total sum can reach roughly `N * ln(N)` which for N=10^6 is ~1.4*10^7 — fits in int. But for larger N constraints or intermediate products, the cast protects against overflow. Always cast one factor to `long long` before multiplication when the product might be large.

---

## 11. Problem 9 — Count Primes (LC 204)

**Problem:** Return the number of primes strictly less than `n`.

**Constraints:** `0 <= n <= 5*10^6`.

**Examples:** `n=10` → `4` (primes: 2, 3, 5, 7). `n=0, n=1` → `0`.

### Approach 1 — Naive (O(n√n), Infeasible)

For each number 2 to n-1, check primality by trial division up to its square root. **Time:** O(n√n). For n=5*10^6: ~10^10 operations — completely infeasible.

### Approach 2 — Sieve of Eratosthenes (Optimal)

```cpp
int countPrimes(int n) {
    if (n < 2) return 0;

    vector<bool> isPrime(n, true);
    isPrime[0] = isPrime[1] = false;

    for (int p = 2; (long long)p * p < n; p++) {
        if (isPrime[p]) {
            // Mark multiples starting from p*p (smaller ones already marked)
            for (int j = p * p; j < n; j += p)
                isPrime[j] = false;
        }
    }

    return count(isPrime.begin(), isPrime.end(), true);
}
```

**Time:** O(n log log n). **Space:** O(n).

### Dry Run — `n=10`

```
isPrime = [F,F,T,T,T,T,T,T,T,T]  (indices 0-9)

p=2: 2*2=4 < 10. Mark 4, 6, 8.
  isPrime = [F,F,T,T,F,T,F,T,F,T]

p=3: 3*3=9 < 10. Mark 9 (6 already false).
  isPrime = [F,F,T,T,F,T,F,T,F,F]

p=4: 4*4=16 ≥ 10. Loop ends.

Count true: indices 2,3,5,7 → 4 primes ✓
```

### Memory Optimization with `bitset`

For large n (n > 10^7), `vector<bool>` uses ~1 byte per element. `bitset` uses 1 bit per element — 8× less memory.

```cpp
#include <bitset>
const int MAXN = 5e6 + 1;
bitset<MAXN> isComposite;   // 0 = prime (default)

int countPrimes_bitset(int n) {
    isComposite[0] = isComposite[1] = 1;
    for (int p = 2; (long long)p * p < n; p++) {
        if (!isComposite[p])
            for (int j = p * p; j < n; j += p)
                isComposite[j] = 1;
    }
    int cnt = 0;
    for (int i = 2; i < n; i++) if (!isComposite[i]) cnt++;
    return cnt;
}
```

For n=5*10^6: `bitset` uses ~625 KB vs. ~5 MB for `vector<bool>`.

---

## 12. Problem 10 — Pow(x, n) (LC 50)

**Problem:** Compute x raised to the power n.

**Constraints:** `-100.0 < x < 100.0`, `-2^31 <= n <= 2^31 - 1`.

### Core Idea: Binary Exponentiation

Every exponent `n` has a binary representation. Write `n = b_k * 2^k + ... + b_1 * 2 + b_0`. Then:

```
x^n = x^(b_k * 2^k) * ... * x^(b_1 * 2) * x^b_0
```

We iterate through the bits of `n` from LSB to MSB. At each step, we square `x` (computing x, x^2, x^4, x^8, ...). Whenever the current bit is 1, we multiply `x`'s current value into the result.

```cpp
double myPow(double x, int n) {
    long long N = n;            // MUST use long long: negating INT_MIN (-2^31) overflows int
    if (N < 0) { x = 1.0 / x; N = -N; }

    double result = 1.0;
    while (N > 0) {
        if (N & 1) result *= x; // current bit is set: include this power of x
        x *= x;                 // square the base for next bit position
        N >>= 1;                // move to next bit
    }
    return result;
}
```

**Time:** O(log |n|). **Space:** O(1).

**The INT_MIN edge case:** `n` can be `-2^31`. Negating `-2^31` in `int` overflows (no positive 32-bit integer has that magnitude). Storing `n` in a `long long` before negating avoids undefined behavior: `-(long long)(-2^31) = 2^31`, which fits in `long long`.

### Dry Run — `x=2, n=10` (binary: 1010)

```
N=10 (1010), x=2, result=1

Bit 0 (N=10=1010): N&1=0. x=2*2=4. N=5.
Bit 1 (N=5=101):   N&1=1. result=1*4=4. x=4*4=16. N=2.
Bit 2 (N=2=10):    N&1=0. x=16*16=256. N=1.
Bit 3 (N=1=1):     N&1=1. result=4*256=1024. x=256*256. N=0.

Result: 1024 = 2^10 ✓
```

---

## 13. Related Concepts and Extensions

### From XOR — Find Duplicate in O(1) Space

Array of n+1 elements with values in [1, n], one value repeated. XOR all elements with XOR of [1..n]. Pairs cancel; the duplicate remains.

### From Subsets — Sum of XOR of All Pairs

For array `arr`, the sum of `arr[i] ^ arr[j]` for all `i < j` equals `(sum over each bit b of: count_zeros * count_ones * 2^b)`. Uses the bitmask understanding that each bit contributes independently.

### From Subsets — Sum of All Subset XORs (LC 1863)

The total equals `(XOR of all elements) * 2^(n-1)`. Each element that is set in the XOR of all elements contributes to exactly half of all subsets.

### From Sieve — Euler's Totient Function

`phi(n)` = count of integers in [1,n] coprime to n. Computed via a sieve: `phi[i] = i`, then for each prime `p`, multiply all multiples by `(1 - 1/p)`. Used in Fermat's little theorem: `a^phi(n) ≡ 1 (mod n)` for gcd(a, n) = 1. Powers in modular arithmetic use binary exponentiation with `% mod` in every multiply.

### From Prime Factors — Sum of Divisors Formula

For `n = p1^a1 * p2^a2 * ... * pk^ak`, the sum of all divisors is the product of geometric series:
```
sigma(n) = (p1^(a1+1) - 1)/(p1 - 1) * (p2^(a2+1) - 1)/(p2 - 1) * ...
```
n is a **perfect number** iff `sigma(n) = 2n`.

### From Count Primes — Segmented Sieve

For ranges [L, R] with R up to 10^12 (too large for a standard sieve): generate primes up to √R with the standard sieve, then use them to mark composites in each segment of size √R. Memory: O(√R) instead of O(R).

### Binary Exponentiation for Modular Arithmetic

```cpp
long long modpow(long long base, long long exp, long long mod) {
    long long result = 1;
    base %= mod;
    while (exp > 0) {
        if (exp & 1) result = result * base % mod;
        base = base * base % mod;
        exp >>= 1;
    }
    return result;
}
```

Used in: Count Good Numbers (LC 1922), Fermat's little theorem modular inverse (`a^(p-2) % p`), matrix exponentiation for Fibonacci.

---

## 14. Common Mistakes and Pitfalls

### Mistake 1: Integer overflow in `p * p` in sieve loop

```cpp
// Wrong: p*p overflows int when p is near 46,340 and n is large
for (int p = 2; p * p < n; p++) { ... }

// Correct: cast one operand to long long
for (int p = 2; (long long)p * p < n; p++) { ... }
```

### Mistake 2: Starting sieve inner loop at `2*p` instead of `p*p`

```cpp
// Inefficient but correct:
for (int j = 2 * p; j < n; j += p) isPrime[j] = false;

// Optimal: start at p*p; multiples 2p, 3p, ..., (p-1)*p already marked by earlier primes
for (int j = p * p; j < n; j += p) isPrime[j] = false;
```

### Mistake 3: XOR from L to R — using `xorUpTo(L)` instead of `xorUpTo(L-1)`

```cpp
// Wrong: cancels L itself, giving XOR of (L+1) to R
return xorUpTo(R) ^ xorUpTo(L);

// Correct: L-1 cancels everything before L; L is included in the result
return xorUpTo(R) ^ xorUpTo(L - 1);
```

### Mistake 4: Not using `long long` for `n` in `myPow` before negation

```cpp
// Wrong: negating INT_MIN in int causes undefined behavior (overflow)
if (n < 0) { x = 1.0/x; n = -n; }   // UB when n == INT_MIN

// Correct:
long long N = n;
if (N < 0) { x = 1.0/x; N = -N; }
```

### Mistake 5: Sum of divisors — missing `long long` cast before multiplication

```cpp
// Potentially wrong: d*(N/d) overflows int for large N
sum += d * (N / d);

// Correct:
sum += (long long)d * (N / d);
```

For N=10^6: individual terms reach 10^6; the total sum reaches ~1.4*10^7 — fits in int. But for N=10^7 or larger, the total exceeds int range. Always cast when the result might grow large.

### Mistake 6: Missing the residual `n > 1` case in prime factors

```cpp
// After the loop, if n > 1 remains, it is a prime factor > sqrt(original n)
// Forgetting this gives wrong output for primes and numbers like 2*p where p is large

if (n > 1) factors.push_back(n);  // this line is mandatory
```

### Mistake 7: `i * i` overflow in prime factors trial division

```cpp
// Wrong: i*i overflows when i > ~46,000 and original n is large
for (int i = 3; i * i <= n; i += 2) { ... }

// Correct:
for (int i = 3; (long long)i * i <= n; i += 2) { ... }
```

### Mistake 8: Using `1 << i` instead of `1 << i` in subsets for `n` near 32

```cpp
// Wrong: 1<<31 and 1<<32 produce undefined behavior with int
int mask; for (mask = 0; mask < (1 << n); mask++)

// Correct for n <= 30 with int; for n up to 63 use 1LL:
for (long long mask = 0; mask < (1LL << n); mask++)
    for (int i = 0; i < n; i++)
        if (mask & (1LL << i)) ...
```

### Mistake 9: Single Number III — INT_MIN in `lsb = xorAB & (-xorAB)`

When `xorAB = INT_MIN`, `-xorAB` overflows in signed int (INT_MIN has no positive 32-bit signed counterpart). Safe workarounds:

```cpp
// Option A: scan for any set bit explicitly
int lsb = 1;
while (!(xorAB & lsb)) lsb <<= 1;

// Option B: use unsigned arithmetic
unsigned int u = (unsigned int)xorAB;
int lsb = (int)(u & (-u));
```

---

## 15. Interview Reference — Pattern Map

| Signal in problem | Technique |
|---|---|
| Count differing bits between two numbers | `__builtin_popcount(a ^ b)` — Hamming distance |
| Find one element appearing odd times, all others even | XOR all elements; pairs cancel |
| Find two elements appearing odd times | XOR all → `a^b`; isolate a set bit (`lsb = xorAB & -xorAB`); partition; XOR each group |
| XOR of a range [L, R] | Prefix XOR: `xorUpTo(R) ^ xorUpTo(L-1)` with N%4 formula |
| Generate all 2^n subsets | Bitmask 0 to `(1<<n)-1`; bit `i` set → include `nums[i]` |
| Sum of all divisors 1 to N | Contribution: each `d` appears `floor(N/d)` times; `sum += d*(N/d)` |
| All prime factors of N, single query | Trial division to √N; push residual if > 1 |
| All prime factors, many queries, N ≤ 10^6 | SPF sieve: O(N log log N) precompute + O(log N) per query |
| Count primes ≤ N | Sieve of Eratosthenes; start inner loop at `p*p` |
| x^n for large n | Binary exponentiation; `long long N = n` before negating |
| Isolate rightmost set bit | `x & (-x)` |
| Clear rightmost set bit | `x & (x-1)` |
| Check if power of 2 | `x > 0 && (x & (x-1)) == 0` |
| Iterate all non-empty subsets of mask m | `for (s = m; s > 0; s = (s-1) & m)` |

---

## 16. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Min Bit Flips (LC 2220) | `popcount(a ^ b)` | O(1) | O(1) |
| Single Number (LC 136) | XOR all | O(n) | O(1) |
| Subsets (LC 78) | Bitmask 0 to 2^n-1 | O(2^n * n) | O(2^n * n) output |
| XOR from L to R | Prefix XOR + N%4 formula | O(1) | O(1) |
| Single Number III (LC 260) | XOR all → partition → XOR groups | O(n) | O(1) |
| Two Odd Occurring (GFG) | Same as Single Number III | O(n) | O(1) |
| Prime Factors, single (GFG) | Trial division to √N | O(√N) | O(log N) |
| Prime Factors, many queries | SPF sieve | O(N log log N) pre + O(log N)/query | O(N) |
| Sum Divisors 1 to N (GFG) | Contribution trick | O(N) | O(1) |
| Count Primes (LC 204) | Sieve of Eratosthenes | O(n log log n) | O(n) |
| Pow(x, n) (LC 50) | Binary exponentiation | O(log n) | O(1) |

---

## 17. Quick Revision Cheat Sheet

**XOR identities:** `a^a=0`, `a^0=a`, commutative, associative. Double XOR cancels: `a^b^a=b`.

**Hamming distance = `popcount(a^b)`.**  Minimum bit flips = Hamming distance.

**Single Number I:** XOR all. Pairs cancel to 0. One element survives.

**Single Number III (two odd elements):**
1. XOR all → `xorAB = a ^ b`
2. `lsb = xorAB & (-xorAB)` — isolates rightmost bit where a and b differ
3. Partition all elements by that bit; XOR each group → `a` and `b` separately

**XOR from L to R:** `xorUpTo(R) ^ xorUpTo(L-1)`
```
xorUpTo(N):
  N%4==0 → N
  N%4==1 → 1
  N%4==2 → N+1
  N%4==3 → 0
```

**Subsets via bitmask:** Iterate `mask` from 0 to `(1<<n)-1`. Bit `i` set → include `nums[i]`.

**Key bit tricks:**
- Isolate lowest set bit: `x & (-x)`
- Clear lowest set bit: `x & (x-1)`
- Power of 2: `x > 0 && (x & (x-1)) == 0`
- Check bit i: `(x >> i) & 1`

**Prime factors (single query):** Check 2 separately, then odd `i` from 3 to √n. If `n > 1` after loop, `n` is a prime factor.

**Sum of divisors 1 to N:** `sum += (long long)d * (N/d)` for `d` from 1 to N. O(N). Divisor `d` appears `floor(N/d)` times across all numbers in [1,N].

**Sieve of Eratosthenes:** Mark `isPrime[0]=isPrime[1]=false`. For each prime `p`, mark multiples starting from `p*p` (smaller multiples already marked). Loop condition: `(long long)p*p < n`.

**Binary exponentiation:**
```cpp
long long N = n;
if (N < 0) { x = 1.0/x; N = -N; }   // long long handles INT_MIN
while (N > 0) {
    if (N & 1) result *= x;
    x *= x;
    N >>= 1;
}
```

**SPF sieve:** `spf[i] = i` initially. For each prime `p` (when `spf[p]==p`), mark `spf[j] = p` for multiples `j = p*p, p*p+p, ...` not yet marked. Factorize n in O(log n): repeatedly divide by `spf[n]`.

**Overflow checkpoints:**
- `p*p` in sieve/trial division: `(long long)p * p`
- `d * (N/d)`: `(long long)d * (N/d)`
- `n` in Pow(x,n): `long long N = n` before negating
