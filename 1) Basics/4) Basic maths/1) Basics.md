# Basic Maths for DSA

> Video Reference: [Striver — Basic Maths for DSA](https://youtu.be/1xNbjMdbjug)
>
> Problems Covered: Count Digits | Reverse Integer | Palindrome Number | Armstrong Numbers | GCD / HCF and LCM | Print All Divisors | Check for Prime

---

## Table of Contents

1. [Why Basic Maths Matters](#1-why-basic-maths-matters)
2. [The Core Primitive — Digit Extraction](#2-the-core-primitive--digit-extraction)
3. [Count Digits](#3-count-digits)
4. [Reverse a Number](#4-reverse-a-number)
5. [Palindrome Number](#5-palindrome-number)
6. [Armstrong Numbers](#6-armstrong-numbers)
7. [Print All Divisors](#7-print-all-divisors)
8. [Check for Prime](#8-check-for-prime)
9. [GCD / HCF and LCM](#9-gcd--hcf-and-lcm)
10. [Sieve of Eratosthenes](#10-sieve-of-eratosthenes)
11. [Number Theory Reference](#11-number-theory-reference)
12. [Complexity Cheat Sheet](#12-complexity-cheat-sheet)
13. [Common Mistakes and Edge Cases](#13-common-mistakes-and-edge-cases)
14. [Interview Pattern Recognition](#14-interview-pattern-recognition)
15. [Quick Revision Cheat Sheet](#15-quick-revision-cheat-sheet)

---

## 1. Why Basic Maths Matters

These problems look like warm-up exercises. They are not. They establish the foundations that reappear throughout DSA:

- **Number theory** — GCD, LCM, prime factorization, coprimality
- **Modular arithmetic** — the backbone of hashing, cryptography, combinatorics
- **Overflow reasoning** — a skill that separates correct from incorrect solutions at every level
- **Square root optimization** — a structural idea that scales to sieve, factorization, and geometry problems
- **Digit decomposition** — powers recursion intuition and the "pop and push" construction pattern

The real skill being trained here is not arithmetic. It is learning to break a number apart, process its pieces independently, exploit mathematical symmetry, and reduce unnecessary computation — all while reasoning formally about correctness and overflow.

---

## 2. The Core Primitive — Digit Extraction

Almost every problem in this section is powered by one operation: extracting digits one by one from a number. Internalizing this primitive fully is more important than memorizing any individual solution.

### The Two Operations

```
n % 10   →  gives the LAST (rightmost) digit
n / 10   →  removes the LAST digit (integer division, shifts all digits right by one position)
```

### Why This Works — Positional Number System

Any integer `n` in base 10 can be written as:

```
n = d_k * 10^k + d_(k-1) * 10^(k-1) + ... + d_1 * 10 + d_0
```

`n % 10` extracts `d_0` (the units digit) because all higher terms are divisible by 10.
`n / 10` produces `d_k * 10^(k-1) + ... + d_1`, which is the number with its last digit stripped.

### General Template

```cpp
while (n > 0) {
    int digit = n % 10;   // extract last digit
    n /= 10;              // remove last digit
    // process digit
}
```

### Dry Run for n = 4726

```
Iter 1: digit = 4726 % 10 = 6,  n = 4726 / 10 = 472
Iter 2: digit = 472  % 10 = 2,  n = 472  / 10 = 47
Iter 3: digit = 47   % 10 = 7,  n = 47   / 10 = 4
Iter 4: digit = 4    % 10 = 4,  n = 4    / 10 = 0
Stop: n == 0
```

### Behavior with Negative Numbers in C++

In C++, the result of `%` has the same sign as the dividend:

```cpp
-123 % 10  = -3
-123 / 10  = -12
```

**Safe pattern:** always work with `abs(n)` before digit extraction when negatives are possible, then restore the sign separately if needed.

### What the Primitive Powers

| Problem | Usage of digit extraction |
|---|---|
| Count Digits | Count the number of iterations |
| Reverse Number | Build the reversed value digit by digit |
| Palindrome | Compare reversed value with original |
| Armstrong | Accumulate sum of (digit ^ k) |
| Sum / Product of Digits | Accumulate over all digits |

---

## 3. Count Digits

**Problem:** Given an integer `n`, count the number of digits in it.

### Approach 1 — Digit Extraction Loop

**Intuition:** Each division by 10 removes one digit. The number of times you can divide before reaching 0 equals the digit count.

```cpp
int countDigits(int n) {
    if (n == 0) return 1;   // 0 has exactly one digit
    n = abs(n);             // handle negatives
    int count = 0;
    while (n > 0) {
        n /= 10;
        count++;
    }
    return count;
}
```

**Dry Run for n = 7789:**

```
count=0: n=7789 → n=778,  count=1
count=1: n=778  → n=77,   count=2
count=2: n=77   → n=7,    count=3
count=3: n=7    → n=0,    count=4
Return 4
```

**Time Complexity:** O(log10(n)) — a number with d digits runs exactly d iterations, and d = floor(log10(n)) + 1.
**Space Complexity:** O(1)

### Approach 2 — Logarithm Formula

**Intuition:** Any d-digit number n satisfies `10^(d-1) <= n < 10^d`. Taking log base 10: `d-1 <= log10(n) < d`, so `d = floor(log10(n)) + 1`.

```
log10(1)    = 0.0  → floor + 1 = 1
log10(9)    = 0.95 → floor + 1 = 1
log10(10)   = 1.0  → floor + 1 = 2
log10(4726) = 3.67 → floor + 1 = 4
```

```cpp
#include <cmath>
int countDigits(int n) {
    if (n == 0) return 1;
    return (int)floor(log10(abs(n))) + 1;
}
```

**Time Complexity:** O(1)
**Space Complexity:** O(1)

**Caution:** `log10` returns a `double`. For very large numbers near powers of 10, floating point precision can produce off-by-one errors (e.g., `log10(1000)` might return `2.9999...` instead of `3.0`). The loop approach is safer in competitive programming; use the formula only when a constant-time guarantee is explicitly required.

### Variant — Count Digits of N That Divide N

A common follow-up: given n, how many of its individual digits evenly divide n?

```cpp
int countDivisibleDigits(int n) {
    int temp = abs(n);
    int count = 0;
    while (temp > 0) {
        int digit = temp % 10;
        if (digit != 0 && n % digit == 0) count++;
        temp /= 10;
    }
    return count;
}
// n=1248: digits 1,2,4,8. 1248%1=0, 1248%2=0, 1248%4=0, 1248%8=0 → count=4
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| 0 | 1 | Zero has exactly one digit |
| -456 | 3 | Sign is not a digit |
| 1000 | 4 | Powers of 10 |

---

## 4. Reverse a Number

**Problem:** Given a signed 32-bit integer x, return x with its digits reversed. If the reversed integer overflows the 32-bit signed range `[-2^31, 2^31 - 1]`, return 0.

**LeetCode 7 — Reverse Integer**

### Core Insight — Pop and Push

Reversing is the inverse of extraction. You pop a digit from the right of x (via `x % 10`) and push it onto the right of a result being built from scratch:

```
result = result * 10 + digit
```

This shifts all previously pushed digits left by one position and appends the new digit at the units place.

### Approach 1 — Using long long (Recommended)

```cpp
int reverse(int x) {
    long long rev = 0;
    while (x != 0) {
        int digit = x % 10;    // handles negatives correctly: -123%10 = -3
        x /= 10;
        rev = rev * 10 + digit;
    }
    if (rev > INT_MAX || rev < INT_MIN) return 0;
    return (int)rev;
}
```

**Dry Run for x = 123:**

```
rev=0:   digit=3, x=12,  rev = 0*10 + 3  = 3
rev=3:   digit=2, x=1,   rev = 3*10 + 2  = 32
rev=32:  digit=1, x=0,   rev = 32*10 + 1 = 321
321 <= INT_MAX → return 321
```

**Dry Run for x = -123:**

```
digit = -123%10 = -3,  x=-12,  rev = 0*10 + (-3)  = -3
digit = -12%10  = -2,  x=-1,   rev = -3*10 + (-2) = -32
digit = -1%10   = -1,  x=0,    rev = -32*10 + (-1)= -321
return -321   (sign preserved automatically)
```

**Dry Run for x = 1534236469 (overflow case):**

```
After all iterations: rev = 9646324351 > INT_MAX (2147483647)
return 0
```

**Time Complexity:** O(log10(x)) — proportional to the number of digits.
**Space Complexity:** O(1)

### Approach 2 — Without 64-bit Integers (No long long)

Check for overflow **before** the multiply, not after (since the multiply itself is what overflows):

```cpp
int reverse(int x) {
    int rev = 0;
    while (x != 0) {
        int digit = x % 10;
        x /= 10;
        // rev * 10 + digit overflows if:
        if (rev > INT_MAX / 10 || (rev == INT_MAX / 10 && digit > 7))  return 0;
        if (rev < INT_MIN / 10 || (rev == INT_MIN / 10 && digit < -8)) return 0;
        rev = rev * 10 + digit;
    }
    return rev;
}
```

**Why 7 and -8?**

`INT_MAX = 2147483647` — its last digit is 7.
`INT_MIN = -2147483648` — its last digit is -8.

If `rev == INT_MAX / 10 = 214748364`, then `rev * 10 + digit` fits in an int only if `digit <= 7`. If `digit > 7`, it overflows.

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| 0 | 0 | Zero reversed is zero |
| 120 | 21 | Trailing zeros in original become leading zeros in reverse, dropped |
| -120 | -21 | Negative; trailing zero dropped |
| 1534236469 | 0 | Reversal exceeds INT_MAX |

---

## 5. Palindrome Number

**Problem:** Given an integer x, return true if x reads the same forwards and backwards.

**LeetCode 9 — Palindrome Number**

### Key Observations Before Writing Code

1. All negative numbers are not palindromes (`-121` reversed is `121-`, which is not a valid number).
2. Any number ending in 0, except 0 itself, cannot be a palindrome — the reversed number would start with 0, making it a different value.
3. `0` is a palindrome.

### Approach 1 — Reverse Full Number and Compare

```cpp
bool isPalindrome(int x) {
    if (x < 0) return false;
    if (x != 0 && x % 10 == 0) return false;

    int original = x;
    long long rev = 0;
    int temp = x;
    while (temp > 0) {
        rev = rev * 10 + (temp % 10);
        temp /= 10;
    }
    return rev == original;
}
```

**Time Complexity:** O(log10(x))
**Space Complexity:** O(1)

### Approach 2 — Reverse Only Half the Number (Optimal, No Overflow)

**Intuition:** We do not need to reverse the full number. Reverse only the second half, then compare with the first half. We know we have reached the halfway point when the reversed half is no longer smaller than the remaining front half.

```cpp
bool isPalindrome(int x) {
    if (x < 0) return false;
    if (x != 0 && x % 10 == 0) return false;

    int revHalf = 0;
    while (x > revHalf) {
        revHalf = revHalf * 10 + x % 10;
        x /= 10;
    }
    // Even digit count: 1221 → x=12, revHalf=12 → x == revHalf
    // Odd digit count:  121  → x=1,  revHalf=12 → x == revHalf/10 (middle digit ignored)
    return x == revHalf || x == revHalf / 10;
}
```

**Dry Run for x = 1221 (even digits):**

```
Start: x=1221, revHalf=0
Iter 1: revHalf = 0*10+1=1,  x=122   (122 > 1, continue)
Iter 2: revHalf = 1*10+2=12, x=12    (12 == 12, stop)
x(12) == revHalf(12) → return true
```

**Dry Run for x = 121 (odd digits):**

```
Start: x=121, revHalf=0
Iter 1: revHalf = 1,  x=12   (12 > 1, continue)
Iter 2: revHalf = 12, x=1    (1 < 12, stop)
x(1) == revHalf/10 (12/10=1) → return true
```

**Dry Run for x = 123:**

```
Iter 1: revHalf=3, x=12
Iter 2: revHalf=32, x=1  (stop)
x(1) != revHalf(32), x(1) != revHalf/10(3) → return false
```

**Time Complexity:** O(log10(x) / 2) — we process only half the digits.
**Space Complexity:** O(1)

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| 0 | true | Palindrome by definition |
| -121 | false | Negative |
| 10 | false | Ends in 0 |
| 11 | true | Two equal digits |

---

## 6. Armstrong Numbers

**Problem:** Return true if n equals the sum of each of its digits raised to the power of the total number of digits.

**LeetCode 1134 — Armstrong Number**

### Definition

For a number with k digits:

```
n is Armstrong  ⟺  d1^k + d2^k + ... + dk^k == n

153 (k=3): 1^3 + 5^3 + 3^3 = 1 + 125 + 27 = 153  ✓
9474 (k=4): 9^4 + 4^4 + 7^4 + 4^4 = 6561+256+2401+256 = 9474  ✓
10 (k=2):  1^2 + 0^2 = 1 ≠ 10  ✗
```

### Algorithm

Step 1: Count digits (k) using the digit extraction loop.
Step 2: For each digit, compute `digit^k` and accumulate.
Step 3: Compare sum with the original number.

```cpp
// Integer power — safer than pow() which returns double
long long intPow(int base, int exp) {
    long long result = 1;
    for (int i = 0; i < exp; i++) result *= base;
    return result;
}

bool isArmstrong(int n) {
    int original = n;

    // Step 1: count digits
    int k = 0, temp = n;
    while (temp > 0) { k++; temp /= 10; }

    // Step 2: sum of digit^k
    long long sum = 0;
    temp = n;
    while (temp > 0) {
        int digit = temp % 10;
        sum += intPow(digit, k);
        temp /= 10;
    }

    return sum == original;
}
```

**Why not use `pow()` from `<cmath>`?**

`pow(digit, k)` returns a `double`. For arguments like `pow(9, 4) = 6561.0`, floating point is fine, but for larger values the rounding can produce an incorrect integer result. Writing `intPow` with integer multiplication costs nothing and is always exact.

**Dry Run for n = 153:**

```
k = 3
digit=3: sum = 0  + 3^3 = 27
digit=5: sum = 27 + 5^3 = 152
digit=1: sum = 152 + 1^3 = 153
sum(153) == original(153) → true
```

**Time Complexity:** O(log n) for the digit count pass + O(log n) for the summation pass = O(log n)
**Space Complexity:** O(1)

### All 3-Digit Armstrong Numbers

```
153 = 1^3 + 5^3 + 3^3 = 1 + 125 + 27
370 = 3^3 + 7^3 + 0^3 = 27 + 343 + 0
371 = 3^3 + 7^3 + 1^3 = 27 + 343 + 1
407 = 4^3 + 0^3 + 7^3 = 64 + 0 + 343
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| 0 | true | 0^1 = 0 |
| 1 | true | 1^1 = 1 |
| 10 | false | 1^2 + 0^2 = 1 ≠ 10 |
| 153 | true | Classic Armstrong |

---

## 7. Print All Divisors

**Problem:** Given an integer n, print all of its divisors in sorted order.

### Key Mathematical Insight — Divisors Come in Pairs

If `d` divides `n`, then `n/d` also divides `n`. So divisors occur in pairs `(d, n/d)` around `sqrt(n)`.

```
n = 36, pairs: (1,36), (2,18), (3,12), (4,9), (6,6)
```

**Proof that no factor beyond sqrt(n) can exist without a paired factor below it:**

Suppose both `a > sqrt(n)` and `b > sqrt(n)`. Then `a * b > sqrt(n) * sqrt(n) = n`. But `a * b = n` is required for both to be factors. Contradiction. Therefore, in any factor pair `(a, b)` with `a <= b`, we have `a <= sqrt(n)`. Checking all `i` from 1 to `sqrt(n)` captures every pair.

### Approach 1 — Brute Force O(n)

```cpp
vector<int> getDivisors(int n) {
    vector<int> divs;
    for (int i = 1; i <= n; i++) {
        if (n % i == 0) divs.push_back(i);
    }
    return divs;
}
```

**Time Complexity:** O(n)

### Approach 2 — Optimal: Loop to sqrt(n) — O(sqrt(n))

```cpp
vector<int> getDivisors(int n) {
    vector<int> divs;
    for (int i = 1; (long long)i * i <= n; i++) {
        if (n % i == 0) {
            divs.push_back(i);
            if (i != n / i) divs.push_back(n / i);  // avoid adding sqrt(n) twice for perfect squares
        }
    }
    sort(divs.begin(), divs.end());
    return divs;
}
```

**Why `(long long)i * i <= n` and not `i <= sqrt(n)`?**

Two reasons. First, `sqrt(n)` returns a `double` and can have precision errors for large n — e.g., `sqrt(49)` might return `6.9999...`, causing you to miss the factor 7. Second, when n is close to `INT_MAX`, `i * i` as a plain `int` multiplication overflows. Casting to `long long` before comparing is both correct and safe.

**Dry Run for n = 36:**

```
i=1: 36%1==0 → add 1, add 36
i=2: 36%2==0 → add 2, add 18
i=3: 36%3==0 → add 3, add 12
i=4: 36%4==0 → add 4, add 9
i=5: 36%5!=0 → skip
i=6: 36%6==0 → add 6 (i == n/i = 6, so do NOT add again)
i=7: 7*7=49 > 36 → stop

Before sort: [1, 36, 2, 18, 3, 12, 4, 9, 6]
After sort:  [1, 2, 3, 4, 6, 9, 12, 18, 36]
```

**Time Complexity:** O(sqrt(n)) for the loop + O(d log d) for the sort, where d is the number of divisors.
**Space Complexity:** O(d)

### Number of Divisors — Formula from Prime Factorization

For `n = p1^a1 * p2^a2 * ... * pk^ak`, the number of divisors is:

```
tau(n) = (a1 + 1)(a2 + 1)...(ak + 1)

Example: 12 = 2^2 * 3^1
tau(12) = (2+1)(1+1) = 6
Divisors: 1, 2, 3, 4, 6, 12  ✓
```

This formula emerges because each divisor independently chooses an exponent for each prime from 0 up to its maximum.

### Sum of Divisors

```cpp
int sumOfDivisors(int n) {
    int sum = 0;
    for (int i = 1; (long long)i * i <= n; i++) {
        if (n % i == 0) {
            sum += i;
            if (i != n / i) sum += n / i;
        }
    }
    return sum;
}
```

### Related Number Classifications

**Perfect number:** sum of proper divisors equals the number itself.
```
28: proper divisors = {1, 2, 4, 7, 14}, sum = 28  → Perfect
```

**Abundant number:** sum of proper divisors exceeds the number (e.g., 12: 1+2+3+4+6=16 > 12).

**Deficient number:** sum of proper divisors is less than the number (most primes).

---

## 8. Check for Prime

**Problem:** Given n, determine if it is prime.

**Definition:** A prime number is a positive integer greater than 1 with no positive divisors other than 1 and itself.

### Approach 1 — Brute Force O(n)

```cpp
bool isPrime(int n) {
    if (n <= 1) return false;
    for (int i = 2; i < n; i++) {
        if (n % i == 0) return false;
    }
    return true;
}
```

### Approach 2 — Check to sqrt(n) — O(sqrt(n))

**Key insight:** The same pairing argument from divisors applies. If `n = p * q` with `p <= q`, then `p <= sqrt(n)`. If no factor exists up to `sqrt(n)`, no factor exists at all.

```cpp
bool isPrime(int n) {
    if (n <= 1) return false;
    if (n == 2) return true;
    if (n % 2 == 0) return false;
    for (int i = 3; (long long)i * i <= n; i += 2) {   // odd numbers only
        if (n % i == 0) return false;
    }
    return true;
}
```

**Dry Run for n = 29:**

```
n%2 != 0
i=3: 29%3=2 ≠ 0
i=5: 29%5=4 ≠ 0
i=7: 7*7=49 > 29 → stop
Return true
```

**Dry Run for n = 35:**

```
35%2 != 0
i=3: 35%3=2 ≠ 0
i=5: 35%5=0 → return false  (35 = 5 * 7)
```

### Approach 3 — 6k ± 1 Optimization — O(sqrt(n)) with Better Constant

**Insight:** All primes greater than 3 are of the form `6k + 1` or `6k - 1`. The reasoning:

```
Every integer falls in one of 6 residue classes mod 6: 0, 1, 2, 3, 4, 5
- 6k+0: divisible by 6
- 6k+2: divisible by 2
- 6k+3: divisible by 3
- 6k+4: divisible by 2
Only 6k+1 and 6k+5 (= 6k-1) can possibly be prime.
```

So instead of checking every odd i, we check only candidates of the form 6k±1, stepping by 6:

```cpp
bool isPrime(int n) {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    for (int i = 5; (long long)i * i <= n; i += 6) {
        if (n % i == 0 || n % (i + 2) == 0) return false;
    }
    return true;
}
```

This eliminates roughly two-thirds of candidates (multiples of 2 and 3), giving a constant factor improvement of about 3x over approach 2.

**Time Complexity:** O(sqrt(n)) — same asymptotic, but with a smaller constant.
**Space Complexity:** O(1)

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| 0 | false | By definition |
| 1 | false | By definition |
| 2 | true | Smallest prime |
| 3 | true | Prime |
| 4 | false | 4 = 2 * 2 |
| INT_MAX (2147483647) | true | Is a Mersenne prime |

---

## 9. GCD / HCF and LCM

**Problem:** Given two integers a and b, find their Greatest Common Divisor (GCD), also called Highest Common Factor (HCF).

### Definitions

**GCD(a, b):** The largest integer that divides both a and b.
**LCM(a, b):** The smallest positive integer divisible by both a and b.

```
GCD(12, 8):  divisors of 12 = {1,2,3,4,6,12}, divisors of 8 = {1,2,4,8} → largest common = 4
LCM(12, 8):  multiples of 12 = {12,24,...}, multiples of 8 = {8,16,24,...} → smallest common = 24
```

### The Key Relationship

```
GCD(a, b) * LCM(a, b) = a * b
```

This means once you have the GCD, LCM is O(1):

```
LCM(a, b) = (a / GCD(a, b)) * b
```

Always divide before multiplying to avoid overflow (computing `a * b` first can overflow if both are near `10^9`).

### Approach 1 — Brute Force O(min(a, b))

```cpp
int gcd(int a, int b) {
    int result = min(a, b);
    while (result > 0) {
        if (a % result == 0 && b % result == 0) return result;
        result--;
    }
    return 1;
}
```

### Approach 2 — Euclidean Algorithm

**The principle:**

```
GCD(a, b) = GCD(b, a % b)
```

**Why this works:**

Let `d = GCD(a, b)`. Then d divides a and d divides b. For any integers, if d divides two numbers it divides any linear combination of them. Since `a % b = a - floor(a/b) * b`, d also divides `a % b`. So every common divisor of (a, b) also divides (b, a%b), meaning `GCD(a, b)` divides `GCD(b, a%b)`. The same argument runs in reverse to show `GCD(b, a%b)` divides `GCD(a, b)`. Two positive integers that each divide the other must be equal, so `GCD(a, b) = GCD(b, a%b)`.

**Termination:** `a % b < b` always, so the second argument strictly decreases each step. Eventually it reaches 0, and `GCD(a, 0) = a` by definition.

### Iterative Euclidean (Recommended — O(1) space)

```cpp
int gcd(int a, int b) {
    while (b != 0) {
        int remainder = a % b;
        a = b;
        b = remainder;
    }
    return a;
}
```

### Recursive Euclidean

```cpp
int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a % b);
}
```

**Dry Run for GCD(48, 18):**

```
a=48, b=18: rem=48%18=12 → a=18, b=12
a=18, b=12: rem=18%12=6  → a=12, b=6
a=12, b=6:  rem=12%6=0   → a=6,  b=0
Return 6.   (48 = 6*8, 18 = 6*3 ✓)
```

**Dry Run for GCD(1002, 999):**

```
1002%999=3 → (999, 3)
999%3=0    → (3, 0)
Return 3.
```

**Time Complexity:** O(log(min(a, b)))

**Why log? Fibonacci worst case:** The worst-case input for the Euclidean algorithm is consecutive Fibonacci numbers: `GCD(F(n+1), F(n))` requires exactly n steps. Since `F(n) ≈ φ^n / sqrt(5)` where φ ≈ 1.618 (the golden ratio), solving for n gives `n ≈ log_φ(F(n)) = O(log F(n))`. In other words, the number of steps is proportional to the number of digits in the smaller input.

**Space Complexity:** O(log(min(a, b))) for the recursive call stack; O(1) for the iterative version.

### Computing LCM

```cpp
long long lcm(int a, int b) {
    return (long long)(a / gcd(a, b)) * b;   // divide FIRST to prevent overflow
}
```

```
a=12, b=8, GCD=4
LCM = (12/4) * 8 = 3 * 8 = 24  ✓
```

### GCD of Multiple Numbers

GCD is associative, so apply it pairwise:

```cpp
int gcdMultiple(vector<int>& nums) {
    int result = nums[0];
    for (int i = 1; i < (int)nums.size(); i++) {
        result = gcd(result, nums[i]);
        if (result == 1) break;   // early exit: GCD cannot decrease below 1
    }
    return result;
}
```

### STL Support (C++17)

```cpp
#include <numeric>
std::gcd(a, b);   // GCD
std::lcm(a, b);   // LCM
```

### GCD in Context — Applications

| Scenario | How GCD is Used |
|---|---|
| Simplify fraction a/b | Divide both numerator and denominator by GCD(a, b) |
| Check if two numbers are coprime | GCD == 1 |
| Rope cutting problem | Maximum piece length = GCD of all lengths |
| Water pouring puzzle | Solution exists iff target is a multiple of GCD |
| Modular inverse | Exists iff GCD(a, m) == 1 |

### GCD and LCM from Prime Factorization

For `a = 2^3 * 3^1` and `b = 2^2 * 3^2 * 5^1`:

```
GCD uses minimum exponents of each prime:
GCD = 2^min(3,2) * 3^min(1,2) = 2^2 * 3^1 = 12

LCM uses maximum exponents of each prime:
LCM = 2^max(3,2) * 3^max(1,2) * 5^max(0,1) = 2^3 * 3^2 * 5 = 360
```

---

## 10. Sieve of Eratosthenes

**Problem:** Find all prime numbers up to N efficiently.

This algorithm is the standard approach whenever you need primes in bulk, not for a single number.

### Algorithm

1. Create `isPrime[0..N]`, initialize all to `true`.
2. Set `isPrime[0] = isPrime[1] = false`.
3. For each `i` from 2 to `sqrt(N)`: if `isPrime[i]` is still `true`, mark all multiples of `i` starting from `i*i` as `false`.
4. All indices still `true` are prime.

**Why start marking from `i*i` and not `2*i`?**

All multiples `2*i, 3*i, ..., (i-1)*i` have already been marked as composite by earlier primes (2, 3, ..., i-1). The first unmarked multiple is `i*i`.

```cpp
vector<bool> sieve(int n) {
    vector<bool> isPrime(n + 1, true);
    isPrime[0] = isPrime[1] = false;
    for (int i = 2; (long long)i * i <= n; i++) {
        if (isPrime[i]) {
            for (int j = i * i; j <= n; j += i) {
                isPrime[j] = false;
            }
        }
    }
    return isPrime;
}

vector<int> getPrimes(int n) {
    auto isPrime = sieve(n);
    vector<int> primes;
    for (int i = 2; i <= n; i++) {
        if (isPrime[i]) primes.push_back(i);
    }
    return primes;
}
```

**Dry Run for N = 10:**

```
Initial:  [F,F,T,T,T,T,T,T,T,T,T]  (indices 0..10)
i=2: mark 4,6,8,10
      [F,F,T,T,F,T,F,T,F,T,F]
i=3: mark 9 (start from 3*3=9)
      [F,F,T,T,F,T,F,T,F,F,F]
i=4: 4*4=16 > 10 → outer loop stops
Primes: 2, 3, 5, 7
```

**Time Complexity:** O(N log log N)

The inner loop for each prime `p` runs `N/p` times. The total work is `N * (1/2 + 1/3 + 1/5 + 1/7 + ...)`, which by Mertens' theorem converges to `N * log log N`.

**Space Complexity:** O(N)

---

## 11. Number Theory Reference

### Divisibility Rules

| Divisor | Rule |
|---|---|
| 2 | Last digit even |
| 3 | Sum of digits divisible by 3 |
| 4 | Last two digits divisible by 4 |
| 5 | Last digit is 0 or 5 |
| 6 | Divisible by both 2 and 3 |
| 8 | Last three digits divisible by 8 |
| 9 | Sum of digits divisible by 9 |
| 10 | Last digit is 0 |
| 11 | Alternating sum of digits divisible by 11 |

### Properties of GCD

```
GCD(a, 0)       = a
GCD(0, 0)       = 0  (by convention)
GCD(a, b)       = GCD(b, a)            (commutative)
GCD(a, b, c)    = GCD(GCD(a, b), c)    (associative)
GCD(ka, kb)     = k * GCD(a, b)        (homogeneous)
GCD(a, b)       = GCD(a - b, b)        (subtraction form, basis of Euclidean algorithm)
```

### Coprime Numbers

Two numbers are **coprime** (relatively prime) if `GCD(a, b) = 1`.

```
GCD(8, 9) = 1 → coprime
GCD(6, 9) = 3 → not coprime
```

Coprimality is essential for: fraction simplification (after dividing by GCD, the resulting numerator and denominator are coprime), the Chinese Remainder Theorem, and modular inverses (the modular inverse of a mod m exists only when GCD(a, m) = 1).

### Fundamental Theorem of Arithmetic

Every integer greater than 1 has a unique prime factorization:

```
12  = 2^2 * 3
360 = 2^3 * 3^2 * 5
```

This uniqueness is the foundation for the number of divisors formula, GCD/LCM computation via exponents, and all of prime-based cryptography.

### Modular Arithmetic

```
(a + b) % m = ((a % m) + (b % m)) % m
(a - b) % m = ((a % m) - (b % m) + m) % m    ← add m to prevent negative result
(a * b) % m = ((a % m) * (b % m)) % m
(a / b) % m  requires modular inverse of b; cannot be reduced the same way
```

### Integer Size Reference

```
int range:       -2,147,483,648  to  2,147,483,647    (~2 * 10^9)
long long range: -9.2 * 10^18    to  9.2 * 10^18

Rules of thumb:
- If any intermediate result can exceed ~2 * 10^9, use long long.
- Product of two ints near 10^9 always overflows int; cast one operand to (long long).
- For (a * b) / gcd, compute as (a / gcd) * b to avoid overflow.
```

---

## 12. Complexity Cheat Sheet

| Problem | Brute Force | Optimal | Key Insight |
|---|---|---|---|
| Count Digits | O(log n) loop | O(1) via log10 formula | `floor(log10(n)) + 1` |
| Reverse Number | O(log n) | O(log n) | Pop-and-push digit construction |
| Palindrome Check | O(log n) | O(log n / 2) | Reverse only half the digits |
| Armstrong Check | O(log n) | O(log n) | Digit extraction + integer power |
| Print Divisors | O(n) | O(sqrt(n)) | Divisors pair around sqrt(n) |
| Sum of Divisors | O(n) | O(sqrt(n)) | Same pairing trick |
| Check Prime | O(n) | O(sqrt(n)) | No factor beyond sqrt(n) |
| Count Primes to N | O(n sqrt(n)) | O(N log log N) | Sieve of Eratosthenes |
| GCD | O(min(a,b)) | O(log(min(a,b))) | Euclidean algorithm |
| LCM | — | O(log(min(a,b))) | LCM = (a / GCD) * b |

---

## 13. Common Mistakes and Edge Cases

### Mistake 1 — Not Handling n = 0 in Digit Count

```cpp
// WRONG: the while loop never executes, returns 0
while (n > 0) { n /= 10; count++; }

// FIX: add this before the loop
if (n == 0) return 1;
```

### Mistake 2 — Integer Overflow in Reversal

```cpp
// WRONG: rev can exceed INT_MAX
int rev = 0;
rev = rev * 10 + digit;

// FIX: use long long for rev, check range at the end
long long rev = 0;
...
if (rev > INT_MAX || rev < INT_MIN) return 0;
```

### Mistake 3 — Overflow in the Divisor/Prime Loop Condition

```cpp
// WRONG: i*i overflows when n is large (e.g., n close to INT_MAX)
for (int i = 1; i * i <= n; i++)

// FIX: cast to long long before the multiplication
for (int i = 1; (long long)i * i <= n; i++)
```

### Mistake 4 — Using `sqrt()` for Loop Bound

```cpp
// RISKY: sqrt returns double; for large n, sqrt(n) may return n-0.0001, stopping one step early
for (int i = 2; i <= sqrt(n); i++)

// FIX: use the integer comparison i*i <= n instead
```

### Mistake 5 — Using `pow()` for Integer Exponentiation

```cpp
// WRONG: pow returns double; precision loss for large arguments
sum += pow(digit, k);

// FIX: write an integer power function
long long intPow(int base, int exp) {
    long long result = 1;
    for (int i = 0; i < exp; i++) result *= base;
    return result;
}
```

### Mistake 6 — LCM Overflow

```cpp
// WRONG: a * b overflows if both a and b are ~10^9
long long result = (a * b) / gcd(a, b);

// FIX: divide first
long long result = (long long)(a / gcd(a, b)) * b;
```

### Mistake 7 — Adding Sqrt Twice in Divisors

```cpp
// WRONG: for n=36, i=6: pushes 6 and 36/6=6 → 6 appears twice
divs.push_back(i);
divs.push_back(n / i);

// FIX: guard with equality check
if (i != n / i) divs.push_back(n / i);
```

### Mistake 8 — Prime Check Not Guarding n <= 1

```cpp
// WRONG: loop exits cleanly for n=0 and n=1, returning true
for (int i = 2; i * i <= n; i++) { if (n % i == 0) return false; }
return true;

// FIX: add at the top
if (n <= 1) return false;
```

### Edge Cases Summary

| Problem | Critical Edge Cases |
|---|---|
| Count Digits | n = 0 (1 digit), n < 0 (use abs) |
| Reverse | x = 0, trailing zeros (120 → 21), overflow at INT_MAX / INT_MIN |
| Palindrome | x < 0 (false), ends in 0 and ≠ 0 (false), x = 0 (true) |
| Armstrong | n = 0 (true), n = 1 (true), n = 10 (false) |
| Divisors | n = 1 (only divisor is 1), prime n (only 1 and n), perfect square (avoid duplicate) |
| Prime | n = 0 (false), n = 1 (false), n = 2 (true), n = 4 (false) |
| GCD | GCD(a, 0) = a, GCD(0, 0) = 0, negative inputs → use abs |

---

## 14. Interview Pattern Recognition

| Pattern | Problems it Appears In |
|---|---|
| Digit extraction (% 10, / 10) | Reverse, palindrome, Armstrong, count digits, digit DP |
| Build number from digits (rev = rev*10 + d) | Reverse integer, constructing numbers in general |
| Divisors pair around sqrt(n) | Print divisors, prime check, prime factorization |
| Euclidean reduction (GCD recurrence) | GCD, extended GCD, modular inverse, Bezout's identity |
| 6k ± 1 prime candidates | Fast prime filtering, segmented sieve |
| Sieve / prefix array for primes | Bulk prime queries, prime counting functions |
| Overflow pre-check before multiply | Reverse integer, power functions, LCM |
| Half-reversal for palindrome | Number palindrome, linked list palindrome (structural analogy) |

---

## 15. Quick Revision Cheat Sheet

This section is self-sufficient. Review only this before an interview.

### Digit Extraction Template

```cpp
while (n > 0) {
    int d = n % 10;   // last digit
    n /= 10;          // remove it
    // process d
}
```

### Count Digits

```cpp
// O(log n)
int count = 0;
if (n == 0) count = 1;
else { n = abs(n); while (n > 0) { count++; n /= 10; } }

// O(1) — use only when exact precision is not critical
int k = (n == 0) ? 1 : (int)floor(log10(abs(n))) + 1;
```

### Reverse Integer

```cpp
long long rev = 0;
while (x != 0) { rev = rev * 10 + x % 10; x /= 10; }
if (rev > INT_MAX || rev < INT_MIN) return 0;
return (int)rev;
```

### Palindrome Number

```cpp
if (x < 0 || (x % 10 == 0 && x != 0)) return false;
int rev = 0;
while (x > rev) { rev = rev * 10 + x % 10; x /= 10; }
return x == rev || x == rev / 10;
```

### Armstrong Number

```cpp
// intPow: integer power, avoids double precision issues
long long intPow(int base, int exp) {
    long long r = 1;
    for (int i = 0; i < exp; i++) r *= base;
    return r;
}

bool isArmstrong(int n) {
    int k = 0, temp = n;
    while (temp) { k++; temp /= 10; }
    long long sum = 0; temp = n;
    while (temp) { int d = temp%10; sum += intPow(d, k); temp /= 10; }
    return sum == n;
}
```

### Divisors (Optimal)

```cpp
vector<int> divs;
for (int i = 1; (long long)i*i <= n; i++) {
    if (n % i == 0) {
        divs.push_back(i);
        if (i != n/i) divs.push_back(n/i);
    }
}
sort(divs.begin(), divs.end());
```

### Prime Check (6k ± 1 Optimal)

```cpp
bool isPrime(int n) {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    for (int i = 5; (long long)i*i <= n; i += 6)
        if (n % i == 0 || n % (i+2) == 0) return false;
    return true;
}
```

### GCD (Euclidean)

```cpp
int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }
// or iterative:
while (b) { int r = a%b; a=b; b=r; } return a;
```

### LCM

```cpp
long long lcm(int a, int b) { return (long long)(a / gcd(a, b)) * b; }
```

### Sieve of Eratosthenes

```cpp
vector<bool> isPrime(n+1, true);
isPrime[0] = isPrime[1] = false;
for (int i = 2; (long long)i*i <= n; i++)
    if (isPrime[i])
        for (int j = i*i; j <= n; j += i)
            isPrime[j] = false;
```

### Key Facts to Memorize

```
INT_MAX  =  2,147,483,647   (2^31 - 1,  last digit 7)
INT_MIN  = -2,147,483,648   (−2^31,     last digit −8)
LLONG_MAX = 9.2 * 10^18     (2^63 - 1)

GCD(a, b) * LCM(a, b) = a * b
GCD worst case: consecutive Fibonacci numbers → O(log_φ(min(a,b))) steps, φ ≈ 1.618
tau(n) = (a1+1)(a2+1)...(ak+1)  for n = p1^a1 * p2^a2 * ... * pk^ak
All primes > 3 are of the form 6k ± 1
Number of primes up to N ≈ N / ln(N)  (Prime Number Theorem)
```

---

*Striver A2Z DSA Sheet — Step 1: Learn the Basics*
*Video: [Basic Maths for DSA — Striver](https://youtu.be/1xNbjMdbjug)*
