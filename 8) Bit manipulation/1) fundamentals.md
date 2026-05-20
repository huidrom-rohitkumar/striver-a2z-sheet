
**Topic:** Bit Manipulation Fundamentals and Interview Tricks
**Patterns:** Bit Masking, XOR Tricks, Set Bit Counting, Shift Arithmetic, Bitwise Greedy Construction


**Resources:**
[TakeUForward Bits Intro](https://takeuforward.org/plus/dsa/problems/introduction-to-bits-and-tricks) |
[TakeUForward Check ith Bit](https://takeuforward.org/plus/dsa/problems/check-if-the-i-th-bit-is-set-or-not) |
[LC 231 Power of Two](https://leetcode.com/problems/power-of-two/) |
[GFG Count Total Set Bits](https://www.geeksforgeeks.org/problems/count-total-set-bits-1587115620/1) |
[HackerRank Maximizing XOR](https://www.hackerrank.com/challenges/maximizing-xor) |
[GFG Swap Numbers](https://www.geeksforgeeks.org/problems/swap-two-numbers3844/1) |
[LC 29 Divide Two Integers](https://leetcode.com/problems/divide-two-integers/) |
[Striver Bit Manipulation 1](https://youtu.be/qQd-ViW7bfk) |
[Striver Bit Manipulation 2](https://youtu.be/nttpF8kwgd4) |
[Striver Bit Manipulation 3](https://youtu.be/pBD4B1tzgVc)

---

## Table of Contents

1. [Binary Number System — Foundation](#1-binary-number-system--foundation)
2. [Bitwise Operators — Complete Reference](#2-bitwise-operators--complete-reference)
3. [Two's Complement and Negative Numbers](#3-twos-complement-and-negative-numbers)
4. [Core Bit Tricks — The Toolkit](#4-core-bit-tricks--the-toolkit)
5. [Problem 1 — Check if the ith Bit is Set](#5-problem-1--check-if-the-ith-bit-is-set)
6. [Problem 2 — Set, Clear, and Toggle the ith Bit](#6-problem-2--set-clear-and-toggle-the-ith-bit)
7. [Problem 3 — Check if a Number is Odd or Even](#7-problem-3--check-if-a-number-is-odd-or-even)
8. [Problem 4 — Power of Two (LC 231)](#8-problem-4--power-of-two-lc-231)
9. [Problem 5 — Count Total Set Bits from 1 to N (GFG)](#9-problem-5--count-total-set-bits-from-1-to-n-gfg)
10. [Problem 6 — Maximizing XOR (HackerRank)](#10-problem-6--maximizing-xor-hackerrank)
11. [Problem 7 — Swap Two Numbers Without Temp (GFG)](#11-problem-7--swap-two-numbers-without-temp-gfg)
12. [Problem 8 — Divide Two Integers (LC 29)](#12-problem-8--divide-two-integers-lc-29)
13. [Extended Bit Patterns and Applications](#13-extended-bit-patterns-and-applications)
14. [C++ Built-in Bit Functions](#14-c-built-in-bit-functions)
15. [Complexity Cheat Sheet](#15-complexity-cheat-sheet)
16. [Common Mistakes and Pitfalls](#16-common-mistakes-and-pitfalls)
17. [Quick Revision Cheat Sheet](#17-quick-revision-cheat-sheet)

---

## 1. Binary Number System — Foundation

Every integer is stored in binary (base 2). Each binary digit is a **bit**. Understanding the bit representation is the prerequisite for all bit manipulation.

### Decimal to Binary

Repeatedly divide by 2 and collect remainders from bottom to top:

```
13 ÷ 2 = 6  remainder 1  ← bit 0 (LSB)
 6 ÷ 2 = 3  remainder 0  ← bit 1
 3 ÷ 2 = 1  remainder 1  ← bit 2
 1 ÷ 2 = 0  remainder 1  ← bit 3 (MSB)

13 = 1101₂ = 1×2³ + 1×2² + 0×2¹ + 1×2⁰ = 8+4+0+1 = 13  ✓
```

### Binary to Decimal

```
1101₂ = 1×8 + 1×4 + 0×2 + 1×1 = 13
```

### Powers of 2 (Must Memorize)

```
2⁰  = 1          2⁸  = 256
2¹  = 2          2⁹  = 512
2²  = 4          2¹⁰ = 1024     (~10³)
2³  = 8          2¹⁶ = 65536
2⁴  = 16         2²⁰ = 1048576  (~10⁶)
2⁵  = 32         2³⁰ = ~10⁹
2⁶  = 64         2³¹ - 1 = 2147483647 = INT_MAX
2⁷  = 128        2³¹ = 2147483648 = INT_MAX + 1
```

### Bit Numbering Convention

```
Number 13 = 0...01101  (32-bit int)
Bit index: 31...3210

Bit 0 (LSB): rightmost, value 1
Bit 1:       second from right, value 2
Bit 2:       third from right, value 4
Bit 3:       leftmost set bit of 13, value 8
Bit 31 (MSB):leftmost bit of 32-bit int, sign bit
```

---

## 2. Bitwise Operators — Complete Reference

### AND (`&`)

Both bits must be 1 for the result to be 1.

```
  1010  (10)
& 1100  (12)
------
  1000  (8)

Truth table: 0&0=0, 0&1=0, 1&0=0, 1&1=1
```

Uses: check if a bit is set (`n & (1<<i)`), clear a bit (`n & ~(1<<i)`), extract LSB (`n & 1`), masking.

### OR (`|`)

At least one bit must be 1 for the result to be 1.

```
  1010  (10)
| 1100  (12)
------
  1110  (14)

Truth table: 0|0=0, 0|1=1, 1|0=1, 1|1=1
```

Uses: set a bit to 1 (`n | (1<<i)`), combine flags.

### XOR (`^`)

Bits must be different for the result to be 1.

```
  1010  (10)
^ 1100  (12)
------
  0110  (6)

Truth table: 0^0=0, 0^1=1, 1^0=1, 1^1=0
```

**XOR properties — critical for interviews:**

```
a ^ 0 = a          identity
a ^ a = 0          self-cancellation
a ^ b ^ a = b      cancellation (key for swap and find-unique)
a ^ b = b ^ a      commutative
(a^b)^c = a^(b^c)  associative
```

Uses: swap without temp, find single non-repeating element, toggle a bit, XOR of a range.

### NOT (`~`)

Flips all bits (one's complement). In two's complement: `~n = -(n+1)`.

```
~5 = ~(0...00101) = 1...11010 = -6
In general: ~n = -(n+1)
```

### Left Shift (`<<`)

Shifts bits left, fills right with zeros. Equivalent to multiplying by 2^k.

```
5 << 1 = 10    (5 × 2¹)
5 << 2 = 20    (5 × 2²)
1 << i = 2^i   (bit mask for bit i)

0101 << 1 = 1010
```

**Warning:** Left-shifting into or past the sign bit of a signed integer is undefined behavior in C++. Use `1LL << i` instead of `1 << i` when `i >= 31`.

### Right Shift (`>>`)

Shifts bits right. For positive numbers: equivalent to integer division by 2^k.

```
20 >> 1 = 10   (20 ÷ 2¹)
20 >> 2 = 5    (20 ÷ 2²)
n >> i  = n ÷ 2^i (integer division)

1010 >> 1 = 0101
```

**Note:** Right shift on signed integers in C++ is implementation-defined (usually arithmetic, i.e., sign-extending). For unsigned integers, it is logical (fills with 0). Use unsigned or non-negative values for predictable behavior.

---

## 3. Two's Complement and Negative Numbers

Modern CPUs store negative integers in **two's complement**. The MSB (bit 31 for `int`) is the sign bit: 0 = positive, 1 = negative.

**Two's complement of n = `~n + 1 = -n`**

```
 5 = 0000...0101
~5 = 1111...1010  (one's complement)
-5 = 1111...1011  (two's complement = ~5 + 1)

Check: -5 + 5 = 1111...1011 + 0000...0101 = 1|0000...0000
The overflow bit is discarded → result = 0  ✓
```

### The `n & -n` Trick — Isolate Rightmost Set Bit

`n & -n` produces a number with only the rightmost set bit of `n` set; all others become 0.

```
n  = 12 = 1100
-n = -12 (two's complement) = 0100
n & -n = 1100 & 0100 = 0100 = 4

n  = 10 = 1010
-n = -10 = 0110
n & -n = 1010 & 0110 = 0010 = 2
```

**Why it works:** Two's complement flips all bits and adds 1. The carry from adding 1 propagates through all trailing zeros and stops at the rightmost set bit, which stays 1 in both `n` and `-n`. All bits above it are flipped in `-n` relative to `n`. Only the rightmost set bit survives the AND.

---

## 4. Core Bit Tricks — The Toolkit

All the fundamental operations in one place, with the reasoning behind each.

```cpp
// ─── Check ────────────────────────────────────────────────────────
bool isSet(int n, int i)     { return (n >> i) & 1; }          // right-shift method
bool isSet2(int n, int i)    { return (n & (1 << i)) != 0; }   // mask method
bool isOdd(int n)            { return n & 1; }
bool isPowerOf2(int n)       { return n > 0 && !(n & (n - 1)); }

// ─── Modify ───────────────────────────────────────────────────────
int setBit(int n, int i)     { return n | (1 << i); }
int clearBit(int n, int i)   { return n & ~(1 << i); }
int toggleBit(int n, int i)  { return n ^ (1 << i); }

// ─── Extract ──────────────────────────────────────────────────────
int lastSetBit(int n)        { return n & (-n); }              // isolate rightmost 1
int removeLastBit(int n)     { return n & (n - 1); }           // clear rightmost 1

// ─── Count ────────────────────────────────────────────────────────
int countBits(int n) {          // Brian Kernighan
    int cnt = 0;
    while (n) { n &= n - 1; cnt++; }
    return cnt;
}
// OR: __builtin_popcount(n)

// ─── Arithmetic ───────────────────────────────────────────────────
// x << k = x * 2^k   |   x >> k = x / 2^k (integer division, positive x)
```

### The `n & (n-1)` Pattern — Clear Rightmost Set Bit

Subtracting 1 from `n` flips the rightmost set bit to 0 and all bits below it to 1. AND-ing with `n` clears the rightmost set bit while leaving everything above it unchanged.

```
n   = 1100  (12)       n   = 1000  (8, a power of 2)
n-1 = 1011  (11)       n-1 = 0111  (7)
n&(n-1)=1000 (8)       n&(n-1)=0000 (0) ← zero iff n was a power of 2
```

**Applications:** power-of-2 check, Brian Kernighan bit-counting, check exactly k bits set.

---

## 5. Problem 1 — Check if the ith Bit is Set

**Problem:** Given `n` and `i`, return `true` if bit `i` of `n` is 1 (set), `false` if it is 0.

**Examples:**
```
n=13 (1101), i=0 → true    (bit 0 = 1)
n=13 (1101), i=1 → false   (bit 1 = 0)
n=13 (1101), i=2 → true    (bit 2 = 1)
```

### Method 1 — Right-Shift n

Shift `n` right by `i` positions so bit `i` is now at position 0. Check if the LSB is 1.

```cpp
bool isIthBitSet(int n, int i) {
    return (n >> i) & 1;
}
```

```
n=13=1101, i=2: n>>2 = 0011. 0011&1 = 1 → true  ✓
n=13=1101, i=1: n>>1 = 0110. 0110&1 = 0 → false ✓
```

### Method 2 — Left-Shift the Mask

Create a mask with only bit `i` set (`1 << i`). AND with `n`; result is non-zero iff bit `i` is 1.

```cpp
bool isIthBitSet(int n, int i) {
    return (n & (1 << i)) != 0;
}
```

```
n=13=1101, i=2: mask=0100. 1101&0100=0100 ≠ 0 → true  ✓
```

Both methods are O(1) time, O(1) space. Method 1 (`(n>>i)&1`) is slightly more idiomatic.

---

## 6. Problem 2 — Set, Clear, and Toggle the ith Bit

**Set bit i (force to 1):** OR with mask — OR guarantees 1 regardless of original value.
```cpp
int setBit(int n, int i)   { return n | (1 << i); }
// 1101 | 0010 = 1111  (set bit 1)
```

**Clear bit i (force to 0):** AND with inverted mask — the 0 in the mask forces a 0 at position i.
```cpp
int clearBit(int n, int i) { return n & ~(1 << i); }
// ~(0100) = 1011. 1101 & 1011 = 1001  (clear bit 2)
```

**Toggle bit i (flip 0↔1):** XOR with mask — XOR with 1 flips; XOR with 0 leaves unchanged.
```cpp
int toggleBit(int n, int i){ return n ^ (1 << i); }
// 1101 ^ 0100 = 1001  (toggle bit 2: was 1, now 0)
// 1001 ^ 0100 = 1101  (toggle again: back to 1)
```

**Dry run for n=13 (1101):**
```
setBit(13, 1):   1101 | 0010 = 1111 = 15
clearBit(13, 2): 1101 & ~(0100) = 1101 & 1011 = 1001 = 9
toggleBit(13,1): 1101 ^ 0010 = 1111 = 15
toggleBit(13,2): 1101 ^ 0100 = 1001 = 9
```

Time: O(1). Space: O(1) for all.

---

## 7. Problem 3 — Check if a Number is Odd or Even

**Bitwise method:**
```cpp
bool isOdd(int n)  { return n & 1; }    // true if LSB is 1
bool isEven(int n) { return !(n & 1); }
```

**Why:** In binary, every odd number has LSB = 1 (it is `2k+1`), and every even number has LSB = 0 (it is `2k`). `n & 1` extracts only the LSB.

```
5  = 101  → 5 & 1 = 1 → odd   ✓
8  = 1000 → 8 & 1 = 0 → even  ✓
-3 = 1...1101 (two's complement) → -3 & 1 = 1 → odd  ✓
-4 = 1...1100 → -4 & 1 = 0 → even  ✓
```

`n & 1` works correctly for negative numbers in two's complement because the LSB of any odd integer (positive or negative) is 1.

Time: O(1). Space: O(1).

---

## 8. Problem 4 — Power of Two (LC 231)

**Problem:** Return `true` if `n` is a power of 2.
**Link:** [LC 231](https://leetcode.com/problems/power-of-two/)
**Difficulty:** Easy

**Examples:**
```
n=1  → true  (2⁰)    n=3  → false    n=0  → false
n=16 → true  (2⁴)    n=-16→ false
```

### The Insight — Powers of 2 Have Exactly One Set Bit

```
1=0001, 2=0010, 4=0100, 8=1000   (exactly one bit each)
3=0011, 5=0101, 6=0110            (multiple bits — not powers)
```

Clearing the single set bit gives 0: `n & (n-1) == 0` iff n is a power of 2.

```
8=1000, 8-1=0111, 8&7=0000 → true   ✓
6=0110, 6-1=0101, 6&5=0100 → false  ✓
```

### Approach 1 — Repeated Division (Brute Force)

```cpp
bool isPowerOfTwo(int n) {
    if (n <= 0) return false;
    while (n > 1) {
        if (n % 2 != 0) return false;
        n /= 2;
    }
    return true;
}
```

Time: O(log n). Space: O(1).

### Approach 2 — `n & (n-1)` Trick (Optimal)

```cpp
bool isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

Time: O(1). Space: O(1). The `n > 0` check is mandatory — without it, n=0 incorrectly returns true (`0 & -1 = 0`).

### Approach 3 — Popcount

```cpp
bool isPowerOfTwo(int n) {
    return n > 0 && __builtin_popcount(n) == 1;
}
```

| Input | Expected | Reason |
|---|---|---|
| 0 | false | 0 is not 2^k for any k |
| 1 | true | 2⁰ |
| -16 | false | Negative numbers are not powers of 2 |
| INT_MAX | false | 2³¹ - 1 has 31 set bits |

---

## 9. Problem 5 — Count Total Set Bits from 1 to N (GFG)

**Problem:** Count the total number of set bits in the binary representations of all integers from 1 to N.
**Link:** [GFG](https://www.geeksforgeeks.org/problems/count-total-set-bits-1587115620/1)
**Difficulty:** Medium

**Examples:**
```
N=4: nums=[1,2,3,4]=[1,10,11,100] → set bits=1+1+2+1=5
N=6: 1+1+2+1+2+2=9
N=17: 35
```

### Approach 1 — Brute Force O(N log N)

```cpp
int countSetBits(int N) {
    int count = 0;
    for (int i = 1; i <= N; i++) count += __builtin_popcount(i);
    return count;
}
```

### Approach 2 — Bit-Position Contribution Formula O(log N)

**Key insight:** Count how many times each bit position contributes a 1, summed over all numbers from 1 to N.

For bit position `b`, the pattern of 0s and 1s repeats with period `2^(b+1)`: first `2^b` numbers have bit b = 0, next `2^b` numbers have bit b = 1.

```
Bit 0: period 2:  0,1,0,1,0,1,...
Bit 1: period 4:  0,0,1,1,0,0,1,1,...
Bit 2: period 8:  0,0,0,0,1,1,1,1,...
```

For N numbers, the 1s contributed by bit `b`:
```
full_cycles = (N+1) / 2^(b+1)          → ones from complete cycles
remainder   = (N+1) % 2^(b+1)          → partial cycle
partial_1s  = max(0, remainder - 2^b)  → ones in partial cycle
contribution = full_cycles × 2^b + partial_1s
```

```cpp
int countSetBits(int N) {
    int count = 0;
    for (int b = 0; (1 << b) <= N; b++) {
        int cycle = 1 << (b + 1);              // period = 2^(b+1)
        int full  = (N + 1) / cycle;           // complete cycles
        int rem   = (N + 1) % cycle;           // remaining numbers
        count += full * (1 << b);              // ones from full cycles
        count += max(0, rem - (1 << b));       // ones from partial cycle
    }
    return count;
}
```

**Dry run for N=6:**
```
b=0: cycle=2, full=7/2=3, rem=7%2=1
  count += 3*1=3; max(0,1-1)=0. count=3.

b=1: cycle=4, full=7/4=1, rem=7%4=3
  count += 1*2=2; max(0,3-2)=1. count=6.

b=2: cycle=8, full=7/8=0, rem=7%8=7
  count += 0*4=0; max(0,7-4)=3. count=9.

b=3: 1<<3=8 > 6. Stop.
Return 9  ✓
```

Time: O(log N). Space: O(1).

### Approach 3 — Recursive (Elegant for Interviews) O(log² N)

**Observation:** `f(2^k - 1) = k × 2^(k-1)` — for the range [0, 2^k − 1], each of k bit positions has exactly 2^(k-1) numbers with that bit set.

For arbitrary N with highest bit at position k (2^k ≤ N < 2^(k+1)):
```
f(N) = f(2^k - 1) + (N - 2^k + 1) + f(N - 2^k)

Where:
  f(2^k - 1) = k × 2^(k-1)    — set bits in [1, 2^k - 1]
  N - 2^k + 1                  — bit k is set for every number in [2^k, N]
  f(N - 2^k)                   — lower bits of the numbers in [2^k, N]
```

```cpp
int countSetBits(int N) {
    if (N == 0) return 0;
    if (N == 1) return 1;
    int x = (int)log2(N);
    int highestPow = 1 << x;
    int fullBlock  = x * (highestPow >> 1);   // f(2^x - 1)
    int msbContrib = N - highestPow + 1;       // bit x set for [2^x .. N]
    int rest       = countSetBits(N - highestPow);
    return fullBlock + msbContrib + rest;
}
```

**Dry run for N=11:**
```
x=3, highestPow=8
fullBlock = 3*4 = 12   (f(7) = 1+1+2+1+2+2+3 = 12)
msbContrib = 11-8+1 = 4
rest = countSetBits(3):
  x=1, highestPow=2
  fullBlock=1*1=1, msbContrib=3-2+1=2, rest=countSetBits(1)=1
  return 1+2+1=4
return 12+4+4 = 20

Verify: 0+1+1+2+1+2+2+3+1+2+2+3 = 20  ✓
```

Time: O(log² N) — O(log N) recursive calls, each O(1). Space: O(log N) call stack.

---

## 10. Problem 6 — Maximizing XOR (HackerRank)

**Problem:** Given two integers L and R, find the maximum value of `a XOR b` for any pair where `L ≤ a, b ≤ R`.
**Link:** [HackerRank](https://www.hackerrank.com/challenges/maximizing-xor)

**Examples:**
```
L=10, R=15:  maximum XOR = 7
L=11, R=100: maximum XOR = 127
```

### Key Insight — Highest Differing Bit Determines the Answer

The maximum XOR between any two numbers in [L, R] is determined by the highest bit position where L and R differ. Once that bit position k is identified, we can always find two numbers in [L, R] whose XOR has all bits from 0 to k set to 1. The answer is `2^(k+1) - 1` — all ones from bit 0 through bit k.

Why: since L and R differ at bit k, there exist numbers in [L, R] that agree on bits above k but have any combination of bits 0 through k. We can pick them such that their XOR at every position 0 through k is 1.

`L XOR R` gives the bits where L and R differ. The highest set bit of `L XOR R` is bit k. The answer is all 1s from bit 0 through bit k.

```cpp
int maxXor(int l, int r) {
    int diff = l ^ r;               // bits that differ
    if (diff == 0) return 0;        // l == r; best XOR is l^l = 0
    int pos = 31 - __builtin_clz(diff);  // position of highest set bit
    return (1 << (pos + 1)) - 1;         // all bits 0..pos set to 1
}
```

**Dry run for L=10, R=15:**
```
L=10=1010, R=15=1111
diff = 1010^1111 = 0101 = 5
Highest bit of 0101 = bit 2 (value 4)
Answer = (1 << 3) - 1 = 7 = 111₂  ✓

Verify: 11^12 = 1011^1100 = 0111 = 7  ✓
```

**Dry run for L=11, R=100:**
```
diff = 11^100 = 0001011^1100100 = 1101111 = 111
Highest bit = bit 6 (value 64)
Answer = (1 << 7) - 1 = 127 = 1111111₂  ✓
```

Time: O(1) (using `__builtin_clz`). Space: O(1).

---

## 11. Problem 7 — Swap Two Numbers Without Temp (GFG)

**Problem:** Swap two integers `a` and `b` without using a third variable.
**Link:** [GFG](https://www.geeksforgeeks.org/problems/swap-two-numbers3844/1)

### Method 1 — XOR Swap

```cpp
void xorSwap(int& a, int& b) {
    if (&a != &b) {     // critical guard — see below
        a = a ^ b;      // a stores XOR of both
        b = a ^ b;      // b = (a^b)^b = a  (original a)
        a = a ^ b;      // a = (a^b)^a = b  (original b)
    }
}
```

**Step-by-step for a=5, b=3:**
```
Initial: a=5(101), b=3(011)
Step 1: a = 101^011 = 110  (a=6, stores XOR)
Step 2: b = 110^011 = 101  (b=5, original a)
Step 3: a = 110^101 = 011  (a=3, original b)
Final: a=3, b=5  ✓
```

**Mathematical proof:**
```
After step 1: a₁ = a⊕b
After step 2: b₂ = a₁⊕b = (a⊕b)⊕b = a⊕(b⊕b) = a⊕0 = a  ✓
After step 3: a₃ = a₁⊕b₂ = (a⊕b)⊕a = (a⊕a)⊕b = 0⊕b = b  ✓
```

**The critical limitation — same-address failure:**
```
If &a == &b (same memory location, e.g., swap(arr[i], arr[i])):
  a = a^a = 0   (a is now 0)
  b = 0^0 = 0   (b is also 0)
  a = 0^0 = 0   (both are 0 — WRONG)
```

Always guard with `if (&a != &b)` before XOR swap.

### Method 2 — Arithmetic Swap

```cpp
a = a + b;  b = a - b;  a = a - b;
```

Risk: integer overflow if `a + b > INT_MAX`.

### Method 3 — `std::swap` (Preferred in Production)

```cpp
swap(a, b);   // O(1), no overflow risk, works for any type
```

**Interview note:** The XOR swap is the expected answer in a bit manipulation discussion. `std::swap` is the correct answer for production code. Know both.

---

## 12. Problem 8 — Divide Two Integers (LC 29)

**Problem:** Divide `dividend` by `divisor` without using `*`, `/`, or `%`. Truncate toward zero. If result overflows 32-bit integer, return INT_MAX.
**Link:** [LC 29](https://leetcode.com/problems/divide-two-integers/)
**Difficulty:** Medium

**Examples:**
```
10 / 3  = 3    (truncated from 3.33)
7 / -3  = -2
-2147483648 / -1 = 2147483647   (overflow: return INT_MAX)
```

### Approach 1 — Repeated Subtraction (Too Slow)

Subtract `divisor` from `dividend` repeatedly and count iterations. O(quotient) time — TLE for large inputs like `INT_MIN / 1`.

### Approach 2 — Exponential Doubling O(log² N)

**Insight:** Instead of subtracting `divisor` one at a time, double it as much as possible (`divisor × 2^k`), subtract that, and record `2^k` in the quotient. This is binary representation of the quotient being built greedily from the most significant bit down.

```cpp
int divide(int dividend, int divisor) {
    // Handle overflow case: INT_MIN / -1 = 2^31, which overflows int
    if (dividend == INT_MIN && divisor == -1) return INT_MAX;

    // Work in long long to avoid abs(INT_MIN) overflow
    long long a = abs((long long)dividend);
    long long b = abs((long long)divisor);

    int sign = ((dividend > 0) == (divisor > 0)) ? 1 : -1;
    long long quotient = 0;

    while (a >= b) {
        long long temp = b;
        long long multiple = 1;

        // Double temp until doubling would exceed a
        while ((temp << 1) <= a) {
            temp     <<= 1;   // temp = b * 2^k
            multiple <<= 1;   // multiple = 2^k
        }

        a         -= temp;      // subtract largest fitting multiple
        quotient  += multiple;  // record that 2^k went into the quotient
    }

    return (int)(sign * quotient);
}
```

**Dry run for dividend=10, divisor=3:**
```
a=10, b=3, sign=+1, quotient=0

Iteration 1:
  temp=3, multiple=1
  temp<<1=6 <= 10: temp=6, multiple=2
  temp<<1=12 > 10: stop
  a=10-6=4, quotient=2

Iteration 2:
  temp=3, multiple=1
  temp<<1=6 > 4: stop immediately
  a=4-3=1, quotient=3

a(1) < b(3): outer loop exits
Return 1 * 3 = 3  ✓
```

**The two critical edge cases:**

Edge case 1 — `INT_MIN / -1`:
```
INT_MIN = -2³¹ = -2147483648
-INT_MIN = 2³¹ = 2147483648 > INT_MAX (2³¹ - 1)
Must check and return INT_MAX before any computation.
```

Edge case 2 — `abs(INT_MIN)` overflow:
```cpp
abs(INT_MIN)             // undefined behavior — INT_MIN has no positive counterpart in int
abs((long long)INT_MIN)  // = 2147483648 — safe in long long
```

**Sign determination:**
```cpp
int sign = ((dividend > 0) == (divisor > 0)) ? 1 : -1;
// Same sign → positive quotient; different sign → negative quotient
```

| dividend | divisor | output | Reason |
|---|---|---|---|
| INT_MIN | -1 | INT_MAX | Overflow |
| 0 | 5 | 0 | Zero dividend |
| -2147483648 | 1 | -2147483648 | INT_MIN / 1 = INT_MIN |
| 7 | -3 | -2 | Truncation toward zero |

Time: O(log² N) — outer loop O(log N) iterations; each inner doubling loop O(log N). Space: O(1).

---

## 13. Extended Bit Patterns and Applications

### XOR Applications

```cpp
// Find single non-repeating element (all others appear exactly twice)
int singleNumber(vector<int>& nums) {
    int res = 0;
    for (int x : nums) res ^= x;  // pairs cancel to 0; single survives
    return res;
}

// XOR from 1 to n — four-case pattern based on n % 4
int xorUpTo(int n) {
    switch (n % 4) {
        case 0: return n;
        case 1: return 1;
        case 2: return n + 1;
        case 3: return 0;
    }
    return -1;
}
// Why: XOR(1,2)=3, XOR(1,2,3)=0, XOR(1,2,3,4)=4; pattern repeats with period 4.

// XOR of [L, R]
int xorRange(int L, int R) {
    return xorUpTo(R) ^ xorUpTo(L - 1);
}

// Count bits to flip A to B
int countFlips(int A, int B) {
    return __builtin_popcount(A ^ B);
    // XOR gives bits that differ; popcount counts them
}
```

### Bitmask Subset Enumeration

```cpp
// All 2^n subsets of n elements
for (int mask = 0; mask < (1 << n); mask++) {
    for (int i = 0; i < n; i++) {
        if (mask & (1 << i)) {
            // element i is in this subset
        }
    }
}

// All submasks of a given mask (iterate over non-empty submasks)
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // process submask 'sub'
    // Terminates because (sub-1)&mask strictly decreases sub each iteration
}
```

### Miscellaneous Tricks

```cpp
// Brian Kernighan — count set bits O(number of set bits)
int count = 0;
while (n) { n &= n - 1; count++; }

// Round up to next power of 2
int nextPow2(int n) {
    n--;
    n |= n >> 1;  n |= n >> 2;  n |= n >> 4;  n |= n >> 8;  n |= n >> 16;
    return n + 1;
}
// Fills all bits below the highest bit, then +1 carries into the next power.

// Position of rightmost set bit (0-indexed)
int rightmostBit(int n) { return __builtin_ctz(n); }   // count trailing zeros
// Equivalently: log2(n & -n)

// Position of leftmost set bit (0-indexed from right)
int leftmostBit(int n) { return 31 - __builtin_clz(n); }  // 31 - leading zeros
```

---

## 14. C++ Built-in Bit Functions

```cpp
// Count set bits
__builtin_popcount(int)           // int
__builtin_popcountl(long)         // long
__builtin_popcountll(long long)   // long long

// Count trailing zeros (position of rightmost set bit)
__builtin_ctz(unsigned int)       // undefined for input 0!
__builtin_ctzl(unsigned long)
__builtin_ctzll(unsigned long long)

// Count leading zeros (from MSB)
__builtin_clz(unsigned int)       // undefined for input 0!
__builtin_clzll(unsigned long long)
// Use: 31 - __builtin_clz(n) = position of highest set bit

// Parity (1 if odd set bits, 0 if even)
__builtin_parity(unsigned int)
__builtin_parityll(unsigned long long)
```

**Usage examples:**
```cpp
__builtin_popcount(13)    // 13=1101 → 3
__builtin_ctz(12)         // 12=1100 → 2 trailing zeros
__builtin_clz(8)          // 8=1000 in 32-bit → 28 leading zeros
31 - __builtin_clz(8)     // → 3 (bit 3 is highest set bit of 8)
```

**Warning:** `__builtin_ctz(0)` and `__builtin_clz(0)` are undefined. Always guard with `if (n != 0)` before calling these.

---

## 15. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Check ith bit | `(n>>i)&1` or `n&(1<<i)` | O(1) | O(1) |
| Set / Clear / Toggle bit | Single bitwise op | O(1) | O(1) |
| Odd/Even | `n&1` | O(1) | O(1) |
| Power of two | `n>0 && !(n&(n-1))` | O(1) | O(1) |
| Count set bits (single n) | Brian Kernighan | O(set bits) | O(1) |
| Count set bits (single n) | `__builtin_popcount` | O(1) | O(1) |
| Total set bits 1..N | Brute force | O(N log N) | O(1) |
| Total set bits 1..N | Bit-position formula | O(log N) | O(1) |
| Total set bits 1..N | Recursive | O(log² N) | O(log N) |
| Maximize XOR in [L,R] | `L^R` + highest bit | O(1) | O(1) |
| Swap without temp | XOR swap (with guard) | O(1) | O(1) |
| Divide integers | Repeated subtraction | O(Q) | O(1) |
| Divide integers | Exponential doubling | O(log² N) | O(1) |

---

## 16. Common Mistakes and Pitfalls

### Mistake 1 — Operator precedence: `&` has lower priority than `==`

```cpp
// WRONG: parsed as n & (1 == 0) = n & 0 = 0 — always 0
if (n & 1 == 0) { /* even */ }

// CORRECT: parenthesize bitwise operations
if ((n & 1) == 0) { /* even */ }
```

The precedence order (high to low): `~` > `<<`, `>>` > `&` > `^` > `|` > `==`, `!=`. Always use parentheses around bitwise expressions when mixing with comparisons.

### Mistake 2 — Left shift overflow for `int`

```cpp
// WRONG: 1 << 31 is undefined behavior for signed int
int mask = 1 << 31;

// CORRECT: use long long literal for shifts >= 31
long long mask = 1LL << 31;
```

### Mistake 3 — XOR swap fails when a and b are the same variable

```cpp
// WRONG: if swap(arr[i], arr[i]), both become 0
void xorSwap(int& a, int& b) {
    a ^= b; b ^= a; a ^= b;  // no address guard
}

// CORRECT:
void xorSwap(int& a, int& b) {
    if (&a != &b) { a ^= b; b ^= a; a ^= b; }
}
```

### Mistake 4 — `abs(INT_MIN)` undefined behavior

```cpp
// WRONG: abs(INT_MIN) overflows since -INT_MIN > INT_MAX
long long a = abs(dividend);  // if dividend == INT_MIN, UB

// CORRECT: cast to long long BEFORE abs
long long a = abs((long long)dividend);
```

### Mistake 5 — Not checking `n > 0` for power-of-two test

```cpp
// WRONG: n=0 gives 0 & -1 = 0 & 0xFFFFFFFF = 0 → incorrectly returns true
bool isPow2 = !(n & (n - 1));

// CORRECT:
bool isPow2 = n > 0 && !(n & (n - 1));
```

### Mistake 6 — Forgetting `INT_MIN / -1` overflow in divide

```cpp
// WRONG: INT_MIN / -1 = 2^31, which overflows int → UB
// Must check BEFORE any arithmetic including abs()
if (dividend == INT_MIN && divisor == -1) return INT_MAX;  // FIRST thing
```

### Mistake 7 — Right shift on signed negative numbers

```cpp
int n = -8;
int m = n >> 1;  // implementation-defined in C++ (usually -4 by sign extension)

// Safe: cast to unsigned if you want logical right shift (fill with 0)
unsigned int m = (unsigned int)n >> 1;
```

### Mistake 8 — `__builtin_ctz(0)` and `__builtin_clz(0)` are undefined

```cpp
// WRONG: undefined for input 0
int pos = __builtin_ctz(n);

// CORRECT: guard first
if (n != 0) { int pos = __builtin_ctz(n); }
```

---

## 17. Quick Revision Cheat Sheet

This section is fully self-contained for night-before revision.

### The Six Operators

```
&   AND:  both 1 → 1.  Masking, checking, clearing bits.
|   OR:   any 1 → 1.   Setting bits, combining flags.
^   XOR:  different → 1. Toggle, swap, find unique, range XOR.
~   NOT:  flip all. ~n = -(n+1).
<<  LEFT SHIFT:  ×2^k. Use 1LL<<i when i >= 31.
>>  RIGHT SHIFT: ÷2^k for non-negative values.
```

### Core Bit Tricks (One-Liners)

```cpp
(n >> i) & 1           // check bit i
n | (1 << i)           // set bit i to 1
n & ~(1 << i)          // clear bit i to 0
n ^ (1 << i)           // toggle bit i
n & 1                  // extract LSB (1=odd, 0=even)
n & (n - 1)            // clear rightmost set bit
n & -n                 // isolate rightmost set bit
n > 0 && !(n & (n-1))  // check power of 2
```

### XOR Properties

```
a ^ 0 = a              identity
a ^ a = 0              self-cancellation
a ^ b ^ a = b          cancellation
a ^ b = b ^ a          commutative
(a^b)^c = a^(b^c)      associative
```

### Problem → Approach

```
Check bit i:           (n >> i) & 1
Odd/Even:              n & 1
Power of 2:            n > 0 && !(n & (n-1))
Count set bits:        __builtin_popcount(n)  OR  Brian Kernighan
Total bits 1..N:       bit-position formula, O(log N)
Maximize XOR [L,R]:    fill all bits below highest bit of (L^R)
XOR from 1 to n:       switch n%4: {n, 1, n+1, 0}
Swap without temp:     XOR swap (guard &a != &b)
Find unique element:   XOR all elements (pairs cancel)
Divide without /:      exponential doubling, O(log² N)
Count bits to flip A→B:__builtin_popcount(A ^ B)
Enumerate all subsets: for mask in [0, 2^n)
```

### Operations Table

| Operation | Formula | When |
|---|---|---|
| Check bit i | `(n>>i)&1` | Test |
| Set bit i | `n|(1<<i)` | Force to 1 |
| Clear bit i | `n&~(1<<i)` | Force to 0 |
| Toggle bit i | `n^(1<<i)` | Flip |
| Check odd | `n&1` | Parity |
| Clear rightmost set | `n&(n-1)` | Power-of-2 check, bit count |
| Isolate rightmost set | `n&(-n)` | Lowest set bit |

### Critical Gotchas

```
1. Parenthesize bitwise ops: (n & mask) == 0, not n & mask == 0
2. Use 1LL << i when i >= 31 (avoid signed int UB)
3. XOR swap: guard if (&a != &b) before swapping
4. abs(INT_MIN): cast to long long first
5. n > 0 in power-of-2 check: n=0 is a false positive otherwise
6. INT_MIN/-1 check in divide: check before any arithmetic
7. __builtin_ctz(0) and __builtin_clz(0): undefined — guard with n != 0
```
