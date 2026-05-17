> Problems Covered: String to Integer atoi (LC 8) | Pow(x, n) (LC 50) | Count Good Numbers (LC 1922) | Sort a Stack (GFG) | Reverse a Stack (TakeUForward)
>
> Video Reference: [Recursion on Stacks — Striver](https://youtu.be/l0YC3876qxg)

---

## Table of Contents

1. [The Unifying Mental Models](#1-the-unifying-mental-models)
2. [Binary Exponentiation — The Mathematical Foundation](#2-binary-exponentiation--the-mathematical-foundation)
3. [String to Integer — Atoi (LC 8)](#3-string-to-integer--atoi-lc-8)
4. [Pow(x, n) (LC 50)](#4-powx-n-lc-50)
5. [Count Good Numbers (LC 1922)](#5-count-good-numbers-lc-1922)
6. [Recursion on Stacks — The Core Idea](#6-recursion-on-stacks--the-core-idea)
7. [Reverse a Stack Using Recursion](#7-reverse-a-stack-using-recursion)
8. [Sort a Stack Using Recursion](#8-sort-a-stack-using-recursion)
9. [Why Reverse and Sort Stack Are Structurally Identical](#9-why-reverse-and-sort-stack-are-structurally-identical)
10. [Mathematical Proofs and Invariants](#10-mathematical-proofs-and-invariants)
11. [Complexity Cheat Sheet](#11-complexity-cheat-sheet)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Interview Pattern Reference](#13-interview-pattern-reference)
14. [Quick Revision Cheat Sheet](#14-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Models

This set bridges two conceptually distinct but equally important algorithmic ideas.

**Cluster 1 — Simulating math with precision (Problems 1–3).**

All three math problems involve implementing something that looks simple but conceals a minefield of edge cases:

`atoi` is a finite state machine over a string. The rules are fully specified; the challenge is applying them in the correct order without any case falling through, and detecting overflow before it happens.

`Pow(x, n)` has a naive O(|n|) solution that is killed by `n = INT_MIN`. The correct O(log n) algorithm is **binary exponentiation** — a foundational algorithm that also underpins modular arithmetic, matrix exponentiation, Fermat's little theorem, and cryptography.

Count Good Numbers is a counting problem that reduces to two independent exponentiation sub-problems. Because `n` can be up to `10^15`, both must use **modular binary exponentiation** — applying `% MOD` at every multiplication step to prevent 64-bit overflow.

The connecting thread: whenever you need `x^n` for large `n` or under a modulus, binary exponentiation is the answer.

**Cluster 2 — Recursion on stacks (Problems 4–5).**

Both "reverse a stack" and "sort a stack" impose the constraint: no extra data structure, only recursion. The insight is that **the call stack is itself a data structure**. When you recursively pop all elements off the input stack, the call stack temporarily holds them in its frames. On the way back up (unwinding), you insert each held element at the correct position — at the bottom for reversal, or in sorted order for sorting.

Both problems share the same two-function skeleton: a main driver that strips elements one by one, and a helper that inserts one element into a stack already in the desired state.

---

## 2. Binary Exponentiation — The Mathematical Foundation

Binary exponentiation computes `x^n` in O(log n) multiplications instead of O(n). It is used directly in Pow(x, n) and Count Good Numbers, and appears throughout competitive programming.

**The recurrence:**

```
x^n = 1                    if n == 0
x^n = (x^(n/2))^2         if n is even
x^n = x * (x^((n-1)/2))^2 if n is odd
```

At each step, `n` is halved. The number of halvings before reaching 0 is `floor(log2(n))` — hence O(log n).

**Bit-interpretation:** Write `n` in binary as `b_k b_{k-1} ... b_1 b_0`. Then:

```
x^n = x^(b_k * 2^k + ... + b_1 * 2 + b_0)
    = product of x^(2^i) for each bit i that is set in n
```

The algorithm computes `x^1, x^2, x^4, x^8, ...` (successive squarings) and multiplies into `result` only when the corresponding bit of `n` is 1.

**Iterative implementation:**

```cpp
long long binpow(long long x, long long n) {
    long long result = 1;
    while (n > 0) {
        if (n & 1) result *= x;   // bit is set: fold current power into result
        x *= x;                    // square: move to next power of x
        n >>= 1;                   // examine next bit
    }
    return result;
}
```

**Trace for x=2, n=10 (binary 1010):**

```
n=10 (1010): bit0=0 → result=1.0,    x→4,    n=5
n=5  (0101): bit0=1 → result=4,      x→16,   n=2
n=2  (0010): bit0=0 → result=4,      x→256,  n=1
n=1  (0001): bit0=1 → result=1024,   n=0
Answer: 1024 = 2^10  ✓
```

**Modular binary exponentiation** (used when results must be taken mod `M`):

```cpp
long long modpow(long long x, long long n, long long M) {
    long long result = 1;
    x %= M;                               // reduce base before starting
    while (n > 0) {
        if (n & 1) result = result * x % M;
        x = x * x % M;
        n >>= 1;
    }
    return result;
}
```

**Why `x %= M` before the loop:** After reduction, `x < M ≤ 10^9 + 7`, so `x * x < (10^9 + 7)^2 ≈ 10^18 < LLONG_MAX ≈ 9.2 * 10^18`. Every subsequent squaring stays within `long long`.

**The core modular identity:** `(a * b) % M = ((a % M) * (b % M)) % M`. Modulo distributes over multiplication, so applying it at every step yields the same final result as computing in infinite precision and applying modulo at the end.

---

## 3. String to Integer — Atoi (LC 8)

**Problem:** Implement `myAtoi(string s)` that converts a string to a 32-bit signed integer following these exact rules in order:

1. Skip leading `' '` characters.
2. Read one optional `'+'` or `'-'` sign. If neither, assume positive.
3. Read consecutive digit characters until the first non-digit or end of string. If no digits are read, return 0.
4. Clamp the result to `[INT_MIN, INT_MAX] = [-2147483648, 2147483647]`.

```
" -042"          → -42       (leading spaces, negative sign, leading zero)
"1337c0d3"       → 1337      (stops at 'c')
"words and 987"  → 0         (first non-space character is not sign or digit)
"+-12"           → 0         ('+' consumed as sign, '-' is non-digit → no digits read)
"-91283472332"   → -2147483648  (underflows INT_MIN; clamped)
"-2147483648"    → -2147483648  (exactly INT_MIN; valid, not clamped)
```

### Approach — Sequential State Implementation

**Overflow detection:** The input can be up to 200 characters. A 200-digit number far overflows even `long long` (max ~9.2 * 10^18, 19 digits). Accumulate in `long long` and break early when the magnitude exceeds `INT_MAX + 1` (= 2147483648 = `|INT_MIN|`). This threshold allows exactly INT_MIN to pass through when the sign is negative.

```cpp
int myAtoi(string s) {
    int i = 0, n = s.size(), sign = 1;
    long long result = 0;

    // Step 1: Skip leading whitespace
    while (i < n && s[i] == ' ') i++;

    // Step 2: Read optional sign (exactly one)
    if (i < n && (s[i] == '+' || s[i] == '-')) {
        sign = (s[i] == '-') ? -1 : 1;
        i++;
    }

    // Step 3: Read digits, break on overflow
    while (i < n && isdigit(s[i])) {
        result = result * 10 + (s[i] - '0');
        if (result > (long long)INT_MAX + 1) break;  // magnitude exceeds any valid 32-bit int
        i++;
    }

    // Step 4: Apply sign and clamp
    result *= sign;
    if (result > INT_MAX) return INT_MAX;
    if (result < INT_MIN) return INT_MIN;
    return (int)result;
}
```

**Why `INT_MAX + 1` as the early break threshold:** `|INT_MIN| = 2147483648 = INT_MAX + 1`. If sign is `-1` and magnitude is exactly `INT_MAX + 1`, the result is `INT_MIN` — a valid 32-bit int that must not be clamped. Breaking only when magnitude strictly exceeds `INT_MAX + 1` allows this case through correctly.

**Time:** O(n) — each character visited at most once. **Space:** O(1).

### Dry Run

`s = "  -042"`:

```
i=0,1: spaces → skip. i=2.
i=2: '-' → sign=-1, i=3.
i=3: '0' → result=0*10+0=0.  i=4.
i=4: '4' → result=4.         i=5.
i=5: '2' → result=42.        i=6. End of string.
result = 42 * (-1) = -42. In range → return -42.
```

`s = "-91283472332"` (overflow):

```
sign=-1. Digits accumulate: 9,91,...,91283472332.
After '3' (the digit making it 91283472332): result=91283472332 > INT_MAX+1=2147483648 → break.
result *= -1 = -91283472332. < INT_MIN → return INT_MIN.
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `""` | `0` | No digits read |
| `"   "` | `0` | Only spaces |
| `"+-12"` | `0` | One sign consumed, next char `-` is not a digit |
| `"0032"` | `32` | Leading zeros handled naturally |
| `"2147483648"` | `2147483647` | One above INT_MAX; clamped |
| `"-2147483648"` | `-2147483648` | Exactly INT_MIN; valid |
| `"   +0 123"` | `0` | Stops at the space after `'0'` |

---

## 4. Pow(x, n) (LC 50)

**Problem:** Implement `pow(x, n)` — compute `x` raised to the power `n`.

**Constraints:** `-100.0 < x < 100.0`, `-2^31 ≤ n ≤ 2^31 - 1`.

```
x=2.0,  n=10   → 1024.0
x=2.1,  n=3    → 9.261
x=2.0,  n=-2   → 0.25   (= 1/4)
```

### Approach 1 — Brute Force: Linear Multiplication

Multiply `x` by itself `|n|` times.

**Time:** O(|n|). For `n = 2^31 - 1 ≈ 2.1 * 10^9`, this is 2 billion multiplications — TLE.

### Approach 2 — Recursive Binary Exponentiation

```cpp
double myPow(double x, long long n) {
    if (n == 0) return 1.0;
    if (n < 0)  return myPow(1.0 / x, -n);

    double half = myPow(x, n / 2);      // compute ONCE
    if (n % 2 == 0) return half * half;
    else            return half * half * x;
}

double myPow(double x, int n) {
    return myPow(x, (long long)n);  // cast before any operation
}
```

**Time:** O(log |n|). **Space:** O(log |n|) recursion depth.

### Approach 3 — Iterative Binary Exponentiation (Optimal)

```cpp
double myPow(double x, int n) {
    long long N = n;      // cast FIRST, before negation
    if (N < 0) { x = 1.0 / x; N = -N; }

    double result = 1.0;
    while (N > 0) {
        if (N & 1) result *= x;
        x *= x;
        N >>= 1;
    }
    return result;
}
```

**Time:** O(log |n|). **Space:** O(1).

### The INT_MIN Edge Case — Why the Cast is Non-Negotiable

`n = INT_MIN = -2147483648`. If you write `int n = INT_MIN; n = -n;`, the negation produces `2147483648`, which overflows a signed 32-bit integer (max = 2147483647). In C++ this is **undefined behavior**, and in practice the value wraps back to `-2147483648`, causing an infinite loop or wrong answer.

Fix: `long long N = n; N = -N;` — `long long` holds `2147483648` comfortably (max ~9.2 * 10^18).

### Dry Run

`x=2.0, n=10` (N=10, binary=1010):

```
N=10: bit0=0 → result=1.0;    x→4.0;    N=5
N=5:  bit0=1 → result=4.0;   x→16.0;   N=2
N=2:  bit0=0 → result=4.0;   x→256.0;  N=1
N=1:  bit0=1 → result=1024.0; N=0
Answer: 1024.0  ✓
```

`x=2.0, n=-2` (x→0.5, N=2, binary=10):

```
N=2: bit0=0 → result=1.0; x→0.25; N=1
N=1: bit0=1 → result=0.25; N=0
Answer: 0.25  ✓
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `x=2.0, n=0` | `1.0` | x^0 = 1 for any x |
| `x=1.0, n=INT_MIN` | `1.0` | 1 to any power = 1 |
| `x=-1.0, n=INT_MIN` | `1.0` | Even exponent; (-1)^even = 1 |
| `n=INT_MIN` | Handle with `long long` | Negation overflows 32-bit int |

**Mistake to avoid:** Computing `myPow(x, n/2)` twice:

```cpp
// WRONG: calls myPow twice — total work is O(n), not O(log n)
return myPow(x, n/2) * myPow(x, n/2);

// CORRECT: compute once and reuse
double half = myPow(x, n/2);
return half * half;
```

---

## 5. Count Good Numbers (LC 1922)

**Problem:** A digit string of length `n` (0-indexed) is **good** if:
- Every digit at an **even index** is an even digit: `{0, 2, 4, 6, 8}` — 5 choices.
- Every digit at an **odd index** is a prime digit: `{2, 3, 5, 7}` — 4 choices.

Return the count of good digit strings of length `n`, modulo `10^9 + 7`.

**Constraints:** `1 ≤ n ≤ 10^15`.

```
n=1  → 5    (index 0 only: even index → 5 choices)
n=4  → 400  (indices 0,2 even: 5^2=25; indices 1,3 odd: 4^2=16; 25*16=400)
n=50 → 564908303
```

### Counting the Positions

For a 0-indexed string of length `n`:
- **Even positions** (0, 2, 4, ...): `ceil(n/2) = (n+1)/2` positions.
- **Odd positions** (1, 3, 5, ...): `floor(n/2) = n/2` positions.

**Proof by cases:**
- `n` even (n=2k): even positions = k = (2k+1)/2 (integer division) = (n+1)/2. ✓
- `n` odd (n=2k+1): even positions = k+1 = (2k+2)/2 = (n+1)/2. ✓

**Why positions are independent:** Each position's digit is chosen from a fixed set with no constraint between positions. Total count = product of choices at each position (multiplication rule of counting).

```
Answer = 5^((n+1)/2) * 4^(n/2)   (mod 10^9 + 7)
```

### Why Binary Exponentiation is Mandatory

For `n = 10^15`, the exponent `(n+1)/2 ≈ 5 * 10^14`. Computing `5^(5*10^14)` by loop takes `5 * 10^14` iterations — completely infeasible. Binary exponentiation does it in `O(log(5 * 10^14)) ≈ 49` iterations.

### Approach — Modular Binary Exponentiation

```cpp
class Solution {
    const long long MOD = 1e9 + 7;

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

public:
    int countGoodNumbers(long long n) {
        long long evenPos = (n + 1) / 2;   // ceil(n/2)
        long long oddPos  = n / 2;           // floor(n/2)

        long long ans = modpow(5, evenPos, MOD) * modpow(4, oddPos, MOD) % MOD;
        return (int)ans;
    }
};
```

**Time:** O(log n) — two modular exponentiations each O(log n). **Space:** O(1).

**Note:** `n` in the function signature must be `long long`, not `int`. `n` can reach `10^15`, which far exceeds `INT_MAX ≈ 2.1 * 10^9`.

### Dry Run

`n = 4`:

```
evenPos = (4+1)/2 = 2   oddPos = 4/2 = 2
modpow(5, 2, MOD): 5^2 = 25
modpow(4, 2, MOD): 4^2 = 16
ans = 25 * 16 % MOD = 400  ✓
```

`n = 1`:

```
evenPos = (1+1)/2 = 1   oddPos = 1/2 = 0
modpow(5, 1, MOD) = 5
modpow(4, 0, MOD) = 1   (x^0 = 1)
ans = 5 * 1 = 5  ✓
```

---

## 6. Recursion on Stacks — The Core Idea

### The Constraint and Its Implication

A standard stack exposes only `push`, `pop`, `top`, and `empty`. No indexing, no iteration over internal elements. To access any element below the top, you must pop — and to not lose that element, you must store it somewhere. If no extra data structure is allowed, the only option is the **function call stack**.

When you write:

```cpp
void f(stack<int>& st) {
    int top = st.top(); st.pop();   // hold 'top' in this frame's local variable
    f(st);                           // recurse: this frame's 'top' stays alive
    // ... at this point, st has been processed without 'top' ...
    st.push(top);                    // on unwind, restore 'top'
}
```

the sequence of `top` values across all frames forms an implicit array, ordered top-to-bottom of the original stack. When recursion unwinds, each frame's `top` is re-inserted wherever the problem requires. This is the complete mechanism for both Reverse Stack and Sort Stack.

### The Two-Function Pattern

Both problems use exactly this skeleton:

```
mainFunction(stack):
    if stack empty: return
    top = pop stack
    mainFunction(stack)        // recurse on remaining (now top is in this frame)
    helperFunction(stack, top) // on unwind: insert top at desired position

helperFunction(stack, element):
    if stack empty OR insertion condition: push element; return
    top = pop stack
    helperFunction(stack, element)   // go deeper to find correct position
    push top                          // restore top above element
```

The helper finds the correct insertion position by recursively peeling the stack until the condition is met, then restores all popped elements above.

---

## 7. Reverse a Stack Using Recursion

**Problem:** Reverse a stack in-place. No extra data structure. Only recursion.

```
Input  (top→bottom): [3, 2, 1]  (3 is top)
Output (top→bottom): [1, 2, 3]  (1 is top)
```

### The Two Functions

**`reverseStack(st)`:** Pops elements one by one into recursive frames. On the way back up, calls `insertAtBottom` for each, placing each old top at the new bottom.

**`insertAtBottom(st, x)`:** Inserts `x` at the bottom of `st`. If `st` is empty, push directly (this is the bottom). Otherwise, save the top, recurse to reach the bottom, push `x`, then restore the top above it.

```cpp
void insertAtBottom(stack<int>& st, int x) {
    if (st.empty()) {
        st.push(x);
        return;
    }
    int top = st.top(); st.pop();
    insertAtBottom(st, x);   // reach the bottom
    st.push(top);             // restore top above x
}

void reverseStack(stack<int>& st) {
    if (st.empty()) return;
    int top = st.top(); st.pop();
    reverseStack(st);         // reverse remaining stack
    insertAtBottom(st, top);  // old top goes to new bottom
}
```

### Full Dry Run

Stack notation: `[bottom ... top]`. Input: `[1, 2, 3]` (1 is bottom, 3 is top).

```
reverseStack([1,2,3]):
  top=3; stack=[1,2]
  reverseStack([1,2]):
    top=2; stack=[1]
    reverseStack([1]):
      top=1; stack=[]
      reverseStack([]):  ← base case
      insertAtBottom([], 1): empty → push(1); stack=[1]
    insertAtBottom([1], 2):
      top=1; pop; stack=[]
      insertAtBottom([], 2): push(2); stack=[2]
      push(1); stack=[2,1]     (2 bottom, 1 top)
  insertAtBottom([2,1], 3):
    top=1; pop; stack=[2]
    insertAtBottom([2], 3):
      top=2; pop; stack=[]
      insertAtBottom([], 3): push(3); stack=[3]
      push(2); stack=[3,2]     (3 bottom, 2 top)
    push(1); stack=[3,2,1]     (3 bottom, 1 top)

Final: [3, 2, 1]  (3 is bottom, 1 is top)  ✓ — reversed!
```

**Correctness intuition:** `reverseStack` processes elements top-to-bottom (3, 2, 1). As recursion unwinds, `insertAtBottom` places each at the bottom. The first element processed (3) is placed at the bottom first; the last (1) is placed at the bottom last, but all subsequent elements are restored above it — so 1 ends up on top. Exactly reversed.

**Time:** O(n^2) — `reverseStack` makes n calls; each triggers `insertAtBottom` making up to n calls. Total: 1 + 2 + ... + n = O(n^2).
**Space:** O(n) — maximum recursion depth is O(n) for `reverseStack`; `insertAtBottom` adds another O(n) depth inside, but not simultaneously.

---

## 8. Sort a Stack Using Recursion

**Problem:** Sort a stack in ascending order (smallest at bottom, largest at top). No extra data structure. Only recursion.

```
Input  (top→bottom): [11, 2, 32, 3, 41]  (11 is top)
Output (top→bottom): [41, 32, 11, 3, 2]  (41 is top, 2 is bottom)
```

### The Two Functions

**`sortStack(st)`:** Identical structure to `reverseStack`. Peels elements into frames, then calls `sortedInsert` for each on unwind.

**`sortedInsert(st, x)`:** Inserts `x` into an already-sorted stack (ascending: smallest at bottom) while maintaining sorted order. If `st` is empty or `st.top() ≤ x`, push `x` directly — it belongs on top (largest so far). Otherwise, the current top must go above `x`, so save it, recurse deeper, then restore it.

```cpp
void sortedInsert(stack<int>& st, int x) {
    // Base: empty stack, or x belongs at or above current top
    if (st.empty() || st.top() <= x) {
        st.push(x);
        return;
    }
    // Current top is larger than x; it must end up above x
    int top = st.top(); st.pop();
    sortedInsert(st, x);   // find x's correct position in remaining stack
    st.push(top);           // restore top above x (top > x, so this maintains sorted order)
}

void sortStack(stack<int>& st) {
    if (st.empty()) return;
    int top = st.top(); st.pop();
    sortStack(st);           // sort remaining stack
    sortedInsert(st, top);   // insert held element in sorted position
}
```

### Full Dry Run

Input: `[2, 1, 3]` (bottom to top: 2, 1, 3; 3 is top).

```
sortStack([2,1,3]):
  top=3; stack=[2,1]
  sortStack([2,1]):
    top=1; stack=[2]
    sortStack([2]):
      top=2; stack=[]
      sortStack([]):  ← base case
      sortedInsert([], 2): empty → push(2); stack=[2]
    sortedInsert([2], 1):
      st.top()=2 > 1 → top=2, pop, stack=[]
      sortedInsert([], 1): empty → push(1); stack=[1]
      push(2) → stack=[1,2]     (1 bottom, 2 top)
  sortedInsert([1,2], 3):
    st.top()=2 ≤ 3 → push(3); stack=[1,2,3]  ✓

Final: [1, 2, 3]  (1 bottom, 3 top)  ✓
```

**`sortedInsert` inserting into the middle:**

`sortedInsert([1, 3], 2)` (insert 2 into sorted stack [1,3]):

```
st.top()=3 > 2 → top=3, pop, stack=[1]
sortedInsert([1], 2):
  st.top()=1 ≤ 2 → push(2); stack=[1,2]
push(3) → stack=[1,2,3]  ✓ (sorted: 1,2,3)
```

**Time:** O(n^2) — same analysis as reverse. **Space:** O(n).

---

## 9. Why Reverse and Sort Stack Are Structurally Identical

The same recursive skeleton powers both problems. Only the helper's insertion condition differs:

| Dimension | Reverse Stack | Sort Stack |
|---|---|---|
| Main function body | Pop, recurse, call helper | Pop, recurse, call helper |
| Helper name | `insertAtBottom` | `sortedInsert` |
| Helper base condition | `st.empty()` | `st.empty() || st.top() <= x` |
| What helper achieves | Always inserts at bottom | Inserts in sorted position |
| Result | Elements reversed | Elements sorted ascending |

**Why the structural identity exists:** Both problems reduce to "process elements one by one; for each, insert at a position determined by a predicate." For reversal the predicate is "always bottom" (unconditional); for sorting it is "position where sorted order is maintained." The recursion mechanism for reaching that position is the same in both cases.

This is the general template for **stack transformation without an extra data structure**: peel elements into recursive frames, reinsert each at the desired position using a recursive helper.

---

## 10. Mathematical Proofs and Invariants

### INT_MIN and Two's Complement Negation

In 32-bit two's complement, the range is `[-2147483648, 2147483647]`. The negative boundary is exactly 1 greater in magnitude than the positive boundary. Negating `INT_MIN = -2^31` should yield `2^31`, but that value does not fit in a 32-bit signed int (max = `2^31 - 1`). The operation causes signed integer overflow — undefined behavior in C++. In practice on most hardware, the result wraps back to `INT_MIN`.

**Invariant:** Always cast to a wider type (`long long`) before negating or performing arithmetic that could exceed 32-bit range.

### Modulo Distributes Over Multiplication

**Theorem:** `(a * b) % M = ((a % M) * (b % M)) % M`

This holds because modular arithmetic is a ring: multiplication and addition respect the equivalence classes. Applying `% M` continuously during iterative exponentiation gives the same mathematical result as computing in arbitrary precision and applying `% M` at the end. The advantage: all intermediate values remain bounded by `M^2`, which for `M = 10^9 + 7` gives `M^2 ≈ 10^18 < LLONG_MAX`.

### Counting Positions in Count Good Numbers

For `n`-length string, 0-indexed:
- `n` even (n=2k): even positions {0,2,...,2k-2} = k positions = `(2k+1)/2 = (n+1)/2` (integer division, since `2k+1` is odd and division truncates).
- `n` odd (n=2k+1): even positions {0,2,...,2k} = k+1 positions = `(2k+2)/2 = k+1 = (n+1)/2`.

Both cases: even position count = `(n+1)/2`. Odd position count = `n - (n+1)/2 = n/2`.

---

## 11. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| atoi | Sequential scan | O(n) | O(1) |
| Pow(x, n) | Brute force loop | O(\|n\|) | O(1) |
| Pow(x, n) | Recursive binary exp | O(log\|n\|) | O(log\|n\|) call stack |
| Pow(x, n) | Iterative binary exp | O(log\|n\|) | O(1) |
| Count Good Numbers | Brute loop | O(n) | O(1) |
| Count Good Numbers | Modular binary exp | O(log n) | O(1) |
| Reverse Stack | Two-function recursion | O(n^2) | O(n) call stack |
| Sort Stack | Two-function recursion | O(n^2) | O(n) call stack |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1 — Not Casting `n` to `long long` Before Negation in Pow(x, n)

```cpp
// WRONG: UB when n = INT_MIN
if (n < 0) { x = 1.0 / x; n = -n; }

// CORRECT:
long long N = n;
if (N < 0) { x = 1.0 / x; N = -N; }
```

### Mistake 2 — Computing the Half Power Twice in Recursive Pow

```cpp
// WRONG: O(n) total — the recursion branches rather than halves
return myPow(x, n/2) * myPow(x, n/2);

// CORRECT: O(log n) — compute once and reuse
double half = myPow(x, n/2);
return half * half;
```

### Mistake 3 — Using `int` for `n` in Count Good Numbers

```cpp
// WRONG: n up to 10^15 doesn't fit in int (max ~2.1 * 10^9)
int countGoodNumbers(int n) { ... }

// CORRECT:
int countGoodNumbers(long long n) { ... }
```

### Mistake 4 — Forgetting `% MOD` After Multiplication in modpow

```cpp
// WRONG: can overflow long long when base ~ 10^9
if (n & 1) result = result * base;

// CORRECT:
if (n & 1) result = result * base % MOD;
```

### Mistake 5 — Not Using `long long` for atoi Accumulation

```cpp
// WRONG: overflows int before the comparison
int result = 0;
result = result * 10 + digit;
if (result > INT_MAX) ...   // comparison happens after overflow

// CORRECT:
long long result = 0;
result = result * 10 + digit;
if (result > (long long)INT_MAX + 1) break;
```

### Mistake 6 — Wrong Base Condition in `sortedInsert` (Strict `<` Instead of `<=`)

```cpp
// WRONG: strict < causes issues with equal elements (infinite recursion or wrong order)
if (st.empty() || st.top() < x) { st.push(x); return; }

// CORRECT:
if (st.empty() || st.top() <= x) { st.push(x); return; }
```

### Mistake 7 — Passing Stack by Value to Recursive Functions

```cpp
// WRONG: modifications in recursive frames affect only a local copy
void sortStack(stack<int> st) { ... }

// CORRECT:
void sortStack(stack<int>& st) { ... }
```

### Mistake 8 — Trying to Insert at Bottom by Just Calling `push`

`insertAtBottom` is not just `st.push(x)`. It recursively peels the entire stack to reach the bottom, pushes `x` there, then restores all elements above. Calling `st.push(x)` directly at the call site pushes to the current top — completely wrong.

---

## 13. Interview Pattern Reference

| Problem signature | Technique | Core algorithm |
|---|---|---|
| Implement `x^n`, n potentially very large | Binary exponentiation | `while n: if n&1: result*=x; x*=x; n>>=1` |
| Count combinations mod M with large exponent | Modular binary exponentiation | Same + `% M` at each multiply |
| Parse string to integer with constraints | Sequential state machine | Skip spaces → sign → digits → clamp |
| Transform stack without extra DS | Recursion on stack | Main peels + helper inserts |
| Reverse stack with recursion | `insertAtBottom` | Always insert at bottom |
| Sort stack with recursion | `sortedInsert` | Insert in sorted position |
| Fibonacci or linear recurrences in O(log n) | Matrix exponentiation | Binary exponentiation on 2×2 matrices |
| Modular division (a/b mod M) | Fermat's little theorem | `a * modpow(b, M-2, M) % M` |

### `long long` Decision Rules

| Situation | Needed? |
|---|---|
| Negating an int that might be `INT_MIN` | Yes |
| Accumulating digits from a long string | Yes |
| Product of two values each up to `10^9` | Yes (product can reach `10^18`) |
| Product of two values each up to `10^9 + 7` | Yes, but safe — `(10^9+7)^2 ≈ 10^18 < LLONG_MAX` |
| Product of two values each up to `10^10` | `__int128` or restructure — overflows `long long` |

---

## 14. Quick Revision Cheat Sheet

**Binary Exponentiation (iterative):**

```cpp
result=1;
while (n>0) {
    if (n & 1) result *= x;   // or result = result * x % MOD
    x *= x;                    // or x = x * x % MOD
    n >>= 1;
}
```

**Pow(x, n) key facts:**
- Cast `n` to `long long` before any operation. `int n = INT_MIN; -n` is undefined behavior.
- Negative `n`: set `x = 1.0/x`, then negate `N`.
- Compute `half = myPow(x, n/2)` exactly once; return `half*half` or `half*half*x`.

**Count Good Numbers formula:**

```
evenPos = (n+1)/2   (ceil, integer division)
oddPos  = n/2       (floor, integer division)
Answer = (modpow(5, evenPos, MOD) * modpow(4, oddPos, MOD)) % MOD
```

`n` must be `long long`. Apply `% MOD` after the final multiplication too.

**atoi rules (in order):** skip spaces → one optional sign → digits until non-digit → clamp.
- Accumulate in `long long`. Break when `result > INT_MAX + 1LL`.
- `"+-12"` → 0. `" -042"` → -42. `"-2147483648"` → -2147483648 (valid, not clamped).

**Reverse Stack (two functions):**

```
reverseStack:   if empty: return; top=pop; reverseStack(st); insertAtBottom(st, top)
insertAtBottom: if empty: push(x); return; top=pop; insertAtBottom(st,x); push(top)
```

**Sort Stack (two functions):**

```
sortStack:     if empty: return; top=pop; sortStack(st); sortedInsert(st, top)
sortedInsert:  if empty OR st.top()<=x: push(x); return; top=pop; sortedInsert(st,x); push(top)
```

**Both stack problems: O(n^2) time, O(n) space.** Same recursive skeleton — only the helper's base condition differs.

**Modular arithmetic identity:** `(a * b) % M = ((a % M) * (b % M)) % M`

**Key integer limits:**

```
INT_MAX  =  2,147,483,647   (2^31 - 1)
INT_MIN  = -2,147,483,648   (-2^31)
|INT_MIN| = INT_MAX + 1 = 2,147,483,648
LLONG_MAX ≈ 9.2 * 10^18
(10^9 + 7)^2 ≈ 10^18 < LLONG_MAX — safe for one multiplication
```
