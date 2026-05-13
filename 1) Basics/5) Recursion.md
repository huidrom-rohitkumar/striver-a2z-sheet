## Resources

- [Striver — Introduction to Recursion (TUF)](https://takeuforward.org/recursion/introduction-to-recursion-understand-recursion-by-printing-something-n-times/)
- [LeetCode 344 — Reverse String](https://leetcode.com/problems/reverse-string/)
- [LeetCode 125 — Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)
- [LeetCode 509 — Fibonacci Number](https://leetcode.com/problems/fibonacci-number/)
- [YouTube — Striver Recursion Playlist](https://youtu.be/kvRjNm4rVBE)

---

## Table of Contents

1. [What Recursion Actually Is](#1-what-recursion-actually-is)
2. [How the Call Stack Works](#2-how-the-call-stack-works)
3. [Anatomy of a Recursive Function](#3-anatomy-of-a-recursive-function)
4. [Parameterised vs Functional Recursion](#4-parameterised-vs-functional-recursion)
5. [Head Recursion vs Tail Recursion](#5-head-recursion-vs-tail-recursion)
6. [Problem: Print Name N Times](#6-problem-print-name-n-times)
7. [Problem: Print 1 to N](#7-problem-print-1-to-n)
8. [Problem: Print N to 1](#8-problem-print-n-to-1)
9. [Problem: Sum of First N Numbers](#9-problem-sum-of-first-n-numbers)
10. [Problem: Factorial of N](#10-problem-factorial-of-n)
11. [Problem: Reverse an Array / String (LC 344)](#11-problem-reverse-an-array--string-lc-344)
12. [Problem: Valid Palindrome (LC 125)](#12-problem-valid-palindrome-lc-125)
13. [Problem: Fibonacci Number (LC 509)](#13-problem-fibonacci-number-lc-509)
14. [Analysing Complexity via Recursion Trees](#14-analysing-complexity-via-recursion-trees)
15. [Stack Overflow: When Recursion Breaks](#15-stack-overflow-when-recursion-breaks)
16. [Backtracking vs Recursion](#16-backtracking-vs-recursion)
17. [Where Recursion Leads: Later Topics](#17-where-recursion-leads-later-topics)
18. [Common Mistakes and Pitfalls](#18-common-mistakes-and-pitfalls)
19. [Complexity Cheat Sheet](#19-complexity-cheat-sheet)
20. [Quick Revision Cheat Sheet](#20-quick-revision-cheat-sheet)

---

## 1. What Recursion Actually Is

Recursion is the technique of solving a problem by reducing it to a **smaller instance of the same problem**, trusting that the smaller instance will be solved correctly, and combining the result.

The critical mental shift: you do not need to simulate the entire call stack in your head to write correct recursive code. You only need to answer three questions:

1. What is the smallest input that has a trivially known answer? — This is the **base case**.
2. How does the problem of size `n` relate to the problem of size `n-1`? — This is the **recurrence**.
3. Does my base case plus my recurrence produce the correct answer? — Verify by induction.

This is called the **inductive leap of faith**: assume `f(n-1)` returns the correct answer for `n-1`, then use it to build the answer for `f(n)`. Every correct recursive algorithm is exactly a proof by mathematical induction:

- Base case of the recursion = base case of the induction.
- Recursive step = inductive step.

### Deferred computation

Each recursive call **pauses its current work** and hands the problem to the next call. The current frame cannot finish until the deeper call returns.

```
sum(3)
  waits for sum(2)
    waits for sum(1)
      waits for sum(0) -> returns 0
    returns 1 + 0 = 1
  returns 2 + 1 = 3
returns 3 + 3 = 6
```

This creates two distinct phases in every recursion:

| Phase | Meaning |
|-------|---------|
| Going down | Recursive calls expanding, frames pushed onto the stack |
| Coming back up | Frames returning, stack unwinding |

**Where you place work relative to the recursive call determines in which phase it executes.** This single insight explains print-1-to-N vs print-N-to-1, preorder vs postorder tree traversal, and the construction phase in backtracking.

---

## 2. How the Call Stack Works

Every function call creates a **stack frame** that stores: local variables, parameters, and the return address (where to resume when this call finishes). The call stack is a fixed region of memory — typically 1 to 8 MB.

```
main() calls f(3)
  f(3) is pushed onto the stack
  f(3) calls f(2)
    f(2) is pushed onto the stack
    f(2) calls f(1)
      f(1) is pushed onto the stack
      f(1) hits base case, returns
    f(1) is popped; f(2) resumes from where it paused
  f(2) is popped; f(3) resumes
f(3) is popped; main resumes
```

For a recursion of depth `n`, the call stack holds `n` frames simultaneously at peak. This is why naive recursion **always** contributes at least O(n) space complexity, even when you allocate no auxiliary data structures.

Exceeding the stack limit causes a **stack overflow** — a runtime crash, not a graceful error. On most systems, recursion depth beyond roughly 10,000 to 20,000 is unsafe (see Section 15).

---

## 3. Anatomy of a Recursive Function

Every recursive function has exactly three logical components.

```cpp
ReturnType solve(parameters) {

    // 1. Base case — the smallest input with a directly known answer.
    //    Without this, recursion never terminates.
    if (base_condition) {
        return base_value;
    }

    // 2. Self work — the contribution of the current call.
    //    May be before the recursive call (head style) or after (tail style).

    // 3. Recursive call — the same problem on a strictly smaller input.
    //    Must move closer to the base case on every call.
    return something involving solve(smaller_input);
}
```

The single most important design rule: **every recursive call must move strictly closer to the base case**. If your call is `f(n+1)` when your base case is `f(0)`, you will never terminate.

---

## 4. Parameterised vs Functional Recursion

These are two distinct styles. Knowing which to use, and why, is essential for fluency.

### Parameterised Recursion

The **state of the computation is carried as a parameter**. The function is `void` and performs side effects (printing, modifying a container). There is no meaningful return value.

```cpp
// The running total is passed along; the function prints and returns nothing.
void sumParam(int n, int acc) {
    if (n == 0) {
        cout << acc;
        return;
    }
    sumParam(n - 1, acc + n);
}
// Call: sumParam(n, 0);
```

Use when: you need to print a sequence, accumulate into a data structure, or carry changing state without returning a value.

### Functional Recursion

The **function returns a value** built by combining the current element with the return value of the recursive call.

```cpp
// The function returns the answer for input n.
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 1); // combine current with result of smaller problem
}
```

Use when: you need to compute and return a number, string, bool, or any aggregated value.

### Converting between styles

Any functional recursion can be converted to parameterised by introducing an accumulator. This also makes the function tail-recursive (the recursive call is the very last operation in the function body):

```cpp
// Functional
int sum(int n) {
    if (n == 0) return 0;
    return n + sum(n - 1);  // NOT tail-recursive: addition happens after the call
}

// Parameterised (tail-recursive)
int sumTail(int n, int acc) {
    if (n == 0) return acc;
    return sumTail(n - 1, acc + n);  // tail-recursive: call is last operation
}
// Call: sumTail(n, 0)
```

Tail-recursive functions can theoretically be optimised by the compiler into a loop (tail call optimisation, TCO). C++ does not guarantee TCO, but GCC performs it with `-O2` in practice. Do not rely on this in competitive programming — treat any recursion of depth > 10^4 as a stack overflow risk.

---

## 5. Head Recursion vs Tail Recursion

This distinction determines whether work executes on the way **down** or on the way **back up** the call stack.

### Tail Recursion (work before the recursive call)

```cpp
void tail(int n) {
    if (n == 0) return;
    cout << n << " ";   // work FIRST, on the way down
    tail(n - 1);        // recurse AFTER
}
// Output for n=3: 3 2 1   (top-down order — calls are made in this order)
```

### Head Recursion (work after the recursive call)

```cpp
void head(int n) {
    if (n == 0) return;
    head(n - 1);        // recurse FIRST, go all the way down
    cout << n << " ";   // work AFTER, on the way back up
}
// Output for n=3: 1 2 3   (bottom-up order — work executes on return)
```

### Summary

| Property | Tail Recursion | Head Recursion |
|----------|---------------|----------------|
| Work relative to call | Before | After |
| Execution phase | Going down | Coming back up |
| Output order for print-n | Top-down (N to 1) | Bottom-up (1 to N) |
| Maps naturally to | Iterative loop | Postorder traversal, DP on trees |

Print N to 1 and Print 1 to N are **the same function** with one change: whether the print is before or after the recursive call. This is the most important concrete illustration of head vs tail recursion.

---

## 6. Problem: Print Name N Times

### Problem Statement

Print a given name exactly N times using recursion. No loop allowed.

### Intuition

The iterative version uses a counter: `for (int i = 1; i <= n; i++) cout << name`. In recursion, the counter becomes a parameter. Each call prints once and increments the counter toward `n`. When the counter exceeds `n`, stop.

This is the minimal parameterised recursion pattern — the only state is the current count.

### Code

```cpp
void printName(int i, int n, const string& name) {
    if (i > n) return;          // base case: done printing
    cout << name << "\n";
    printName(i + 1, n, name);  // tail recursion: print before call
}
// Call: printName(1, n, "Raj");
```

### Dry Run (n = 3, name = "Raj")

```
printName(1, 3) -> prints "Raj", calls printName(2, 3)
  printName(2, 3) -> prints "Raj", calls printName(3, 3)
    printName(3, 3) -> prints "Raj", calls printName(4, 3)
      printName(4, 3) -> 4 > 3, returns   [base case]
    returns
  returns
returns

Output: Raj / Raj / Raj
```

### Complexity

| Metric | Value | Justification |
|--------|-------|---------------|
| Time | O(n) | Exactly n recursive calls, O(1) work each |
| Space | O(n) | n frames on the call stack at peak depth |

### Edge Cases

| Input | Output | Reason |
|-------|--------|--------|
| n = 0 | (nothing) | First call: 1 > 0, base case fires immediately |
| n = 1 | name once | One call before base case |

---

## 7. Problem: Print 1 to N

### Problem Statement

Given N, print 1, 2, ..., N in order using recursion.

### Two Approaches

**Approach A — Parameterised (count up):** Pass the current number. Print it, recurse with `i+1`, stop when `i > n`. Work before the call — tail recursion.

**Approach B — Functional / Backtracking style (print on return):** Recurse all the way down to `f(1)`, then print while returning. Work after the call — head recursion. Since `f(n-1)` prints 1 through `n-1` before returning, printing `n` afterward gives 1 through `n`.

Approach B is more instructive because it shows work happening on the way back up.

### Approach A: Parameterised

```cpp
void print1toN_param(int i, int n) {
    if (i > n) return;
    cout << i << " ";
    print1toN_param(i + 1, n);
}
// Call: print1toN_param(1, n);
```

### Approach B: Functional (backtracking style)

```cpp
void print1toN_functional(int n) {
    if (n == 0) return;
    print1toN_functional(n - 1); // recurse first — go all the way down
    cout << n << " ";            // print on the way back up
}
// Call: print1toN_functional(n);
```

### Dry Run — Approach B (n = 4)

```
print1toN_functional(4)
  print1toN_functional(3)
    print1toN_functional(2)
      print1toN_functional(1)
        print1toN_functional(0) -> returns  [base case]
        prints 1                            [on return from depth 0]
      prints 2                              [on return from depth 1]
    prints 3                                [on return from depth 2]
  prints 4                                  [on return from depth 3]

Output: 1 2 3 4
```

The print statement is after the recursive call, so it executes **in reverse order of how the calls were made** — bottom of the stack prints first.

### Complexity

| Approach | Time | Space |
|----------|------|-------|
| Parameterised | O(n) | O(n) call stack |
| Functional | O(n) | O(n) call stack |

Both are identical in complexity. The difference is purely conceptual — which phase the work happens in.

---

## 8. Problem: Print N to 1

### Problem Statement

Given N, print N, N-1, ..., 1 in order using recursion.

### Two Approaches

Exact mirror of Print 1 to N — swap the position of the print relative to the recursive call.

### Approach A: Parameterised (count down)

```cpp
void printNto1_param(int n) {
    if (n == 0) return;
    cout << n << " ";
    printNto1_param(n - 1);
}
```

### Approach B: Functional (print before recursing)

```cpp
void printNto1_functional(int i, int n) {
    if (i > n) return;
    printNto1_functional(i + 1, n); // recurse first
    cout << i << " ";               // print on return — gives reverse order
}
// Call: printNto1_functional(1, n);
```

### Dry Run — Approach A (n = 4)

```
printNto1_param(4) -> prints 4, calls (3)
  printNto1_param(3) -> prints 3, calls (2)
    printNto1_param(2) -> prints 2, calls (1)
      printNto1_param(1) -> prints 1, calls (0)
        printNto1_param(0) -> returns  [base case]

Output: 4 3 2 1
```

### The Symmetry

```
Print before recursive call  ->  output top-down  ->  N to 1
Print after recursive call   ->  output bottom-up ->  1 to N
```

This symmetry is the most important concrete illustration in this entire lecture. It reappears as preorder vs postorder in tree traversal.

### Complexity

| Metric | Value |
|--------|-------|
| Time | O(n) |
| Space | O(n) call stack |

---

## 9. Problem: Sum of First N Numbers

### Problem Statement

Given N, return 1 + 2 + ... + N.

### Three Approaches

**Approach 1 — Functional recursion:** `sum(n) = n + sum(n-1)`.

**Approach 2 — Iterative:** Accumulate in a loop. O(n) time, O(1) space.

**Approach 3 — Closed-form formula:** O(1) time, O(1) space. Always state this in interviews.

### Approach 1: Functional Recursion

```cpp
int sum(int n) {
    if (n == 0) return 0;       // base case: sum of nothing is 0
    return n + sum(n - 1);      // current + sum of everything before
}
```

### Approach 2: Iterative

```cpp
int sumIterative(int n) {
    int total = 0;
    for (int i = 1; i <= n; i++) total += i;
    return total;
}
// Time: O(n), Space: O(1)
```

### Approach 3: Formula (optimal)

```cpp
long long sumFormula(long long n) {
    return n * (n + 1) / 2;
}
// Time: O(1), Space: O(1)
```

**Why the formula works:** Pair elements from both ends — (1, n), (2, n-1), (3, n-2), ... Each pair sums to `n+1`. There are `n/2` such pairs. Total = `n/2 * (n+1) = n(n+1)/2`. (Gauss's trick.)

**Overflow warning:** For large `n`, `n * (n+1)` overflows a 32-bit integer. Always use `long long` or cast: `(long long)n * (n + 1) / 2`.

### Dry Run — Recursive (n = 4)

```
sum(4)
  = 4 + sum(3)
         = 3 + sum(2)
                = 2 + sum(1)
                       = 1 + sum(0)
                              = 0   [base case]
                       returns 1 + 0 = 1
                returns 2 + 1 = 3
         returns 3 + 3 = 6
  returns 4 + 6 = 10

Answer: 10  (= 1+2+3+4, correct)
```

### Complexity

| Approach | Time | Space |
|----------|------|-------|
| Recursive | O(n) | O(n) call stack |
| Iterative | O(n) | O(1) |
| Formula | O(1) | O(1) |

### Edge Cases

| Input | Expected | Reason |
|-------|----------|--------|
| n = 0 | 0 | Base case |
| n = 1 | 1 | 1 + sum(0) = 1 |

---

## 10. Problem: Factorial of N

### Problem Statement

Given N, return N! = N × (N-1) × ... × 1. Note: 0! = 1.

### Intuition

Factorial defines itself recursively: `factorial(n) = n * factorial(n-1)`. The base case is `factorial(0) = 1`. The base case value must be `1`, not `0` — returning `0` would zero out the entire product chain.

**Why is 0! = 1?** Combinatorially: there is exactly one way to arrange zero objects in a row — the empty arrangement. Algebraically: `1! = 1 * 0!` and `1! = 1`, so `0! = 1`.

### Recursive Code

```cpp
long long factorial(int n) {
    if (n == 0) return 1LL;                    // 0! = 1
    return (long long)n * factorial(n - 1);
}
```

### Iterative Code

```cpp
long long factorialIterative(int n) {
    long long result = 1;
    for (int i = 2; i <= n; i++) result *= i;
    return result;
}
// Time: O(n), Space: O(1)
```

### Dry Run — Recursive (n = 5)

```
factorial(5)
  = 5 * factorial(4)
         = 4 * factorial(3)
                = 3 * factorial(2)
                       = 2 * factorial(1)
                              = 1 * factorial(0)
                                     = 1   [base case]
                              returns 1 * 1 = 1
                       returns 2 * 1 = 2
                returns 3 * 2 = 6
         returns 4 * 6 = 24
  returns 5 * 24 = 120

Answer: 120  (correct)
```

### Overflow Analysis

| n | n! | Fits int32? | Fits int64? |
|---|-----|------------|-------------|
| 12 | 479,001,600 | Yes (barely) | Yes |
| 13 | 6,227,020,800 | No | Yes |
| 20 | ~2.4 × 10^18 | No | Yes |
| 21 | ~5.1 × 10^19 | No | No |

**Always use `long long` for factorial.** For n > 20, use `__int128` or arbitrary-precision libraries. The constraint `0 <= n <= 12` in a problem means the setter expects `int`; `0 <= n <= 20` means `long long`.

### Complexity

| Approach | Time | Space |
|----------|------|-------|
| Recursive | O(n) | O(n) call stack |
| Iterative | O(n) | O(1) |

### Edge Cases

| Input | Expected | Reason |
|-------|----------|--------|
| n = 0 | 1 | 0! = 1 by definition |
| n = 1 | 1 | 1 × 0! = 1 |
| n = 13 | overflow if int32 | Use long long |

---

## 11. Problem: Reverse an Array / String (LC 344)

### Problem Statement

**LeetCode 344 — Reverse String (Easy)**

Reverse `vector<char>& s` in-place with O(1) extra memory.

```
Input:  ['h','e','l','l','o']
Output: ['o','l','l','e','h']
```

### Intuition

Swap the outermost pair (`s[left]` and `s[right]`), then recursively reverse the interior `s[left+1 .. right-1]`. Base case: when `left >= right`, the subarray has zero or one element and is already reversed. This is the **two-pointer shrink** pattern expressed recursively.

### Approach 1: Iterative Two Pointers (optimal — satisfies O(1) space)

```cpp
void reverseString(vector<char>& s) {
    int left = 0, right = (int)s.size() - 1;
    while (left < right) {
        swap(s[left], s[right]);
        left++;
        right--;
    }
}
// Time: O(n), Space: O(1)
```

### Approach 2: Recursive Two Pointers

```cpp
void reverseHelper(vector<char>& s, int left, int right) {
    if (left >= right) return;               // base case: 0 or 1 element left
    swap(s[left], s[right]);                 // fix outermost pair
    reverseHelper(s, left + 1, right - 1);  // recurse on interior
}

void reverseString(vector<char>& s) {
    reverseHelper(s, 0, (int)s.size() - 1);
}
// Time: O(n), Space: O(n/2) = O(n) call stack
```

### Dry Run — Recursive (s = ['h','e','l','l','o'])

```
reverseHelper(s, 0, 4):
  swap(s[0], s[4])  ->  ['o','e','l','l','h']
  calls reverseHelper(s, 1, 3)

    reverseHelper(s, 1, 3):
      swap(s[1], s[3])  ->  ['o','l','l','e','h']
      calls reverseHelper(s, 2, 2)

        reverseHelper(s, 2, 2):
          2 >= 2, returns  [base case]

      returns
  returns

Final: ['o','l','l','e','h']
```

### Space Complexity Note

The LeetCode problem requires O(1) **extra memory** in the sense of not allocating a new array. The iterative solution satisfies this. The recursive solution uses O(n/2) call stack space — it does NOT satisfy the strict O(1) space constraint. In an interview, implement iteratively for optimal space, and mention the recursive version to demonstrate understanding of the pattern.

### Complexity

| Approach | Time | Space |
|----------|------|-------|
| Iterative two pointers | O(n) | O(1) |
| Recursive two pointers | O(n) | O(n) call stack |

### Generalisation

The same helper works for any element type:

```cpp
void reverseArray(vector<int>& arr, int left, int right) {
    if (left >= right) return;
    swap(arr[left], arr[right]);
    reverseArray(arr, left + 1, right - 1);
}
```

### Edge Cases

| Input | Expected | Reason |
|-------|----------|--------|
| `['a']` | `['a']` | Single element, base case fires immediately |
| `['a','b']` | `['b','a']` | One swap, then base case |
| `[]` | `[]` | Empty — cast `s.size()` to `int` first: `s.size()-1` underflows as `size_t` if empty |

---

## 12. Problem: Valid Palindrome (LC 125)

### Problem Statement

**LeetCode 125 — Valid Palindrome (Easy)**

After converting to lowercase and removing all non-alphanumeric characters, check if the string reads the same forward and backward.

```
Input:  "A man, a plan, a canal: Panama"  ->  true
Input:  "race a car"                       ->  false
Input:  " "                                ->  true  (empty after cleaning)
```

### Intuition

A string is a palindrome iff every character at position `i` from the left equals the character at position `i` from the right. The recursive version: if the outermost valid characters match, check the interior. Base case: when pointers cross, no mismatch was found.

Two sub-problems specific to LC 125: skip non-alphanumeric characters, and compare case-insensitively.

### Approach 1: Brute Force — Build Cleaned String, Then Check

```cpp
bool isPalindrome_brute(string s) {
    string cleaned;
    for (char c : s)
        if (isalnum(c)) cleaned += tolower(c);
    string rev = cleaned;
    reverse(rev.begin(), rev.end());
    return cleaned == rev;
}
// Time: O(n), Space: O(n)
```

### Approach 2: Two Pointers (optimal)

```cpp
bool isPalindrome(string s) {
    int left = 0, right = (int)s.size() - 1;
    while (left < right) {
        while (left < right && !isalnum(s[left]))  left++;
        while (left < right && !isalnum(s[right])) right--;
        if (tolower(s[left]) != tolower(s[right])) return false;
        left++;
        right--;
    }
    return true;
}
// Time: O(n), Space: O(1)
```

### Approach 3: Recursive Two Pointers

```cpp
bool checkPalindrome(const string& s, int left, int right) {
    if (left >= right) return true;             // base case: no mismatch found
    if (!isalnum(s[left]))  return checkPalindrome(s, left + 1, right);
    if (!isalnum(s[right])) return checkPalindrome(s, left, right - 1);
    if (tolower(s[left]) != tolower(s[right])) return false;
    return checkPalindrome(s, left + 1, right - 1);
}

bool isPalindrome_recursive(string s) {
    return checkPalindrome(s, 0, (int)s.size() - 1);
}
// Time: O(n), Space: O(n) call stack
```

### Dry Run — Two Pointers (s = "race a car")

```
s = "race a car"
     0123456789

left=0 ('r'), right=9 ('r'): tolower match -> left=1, right=8
left=1 ('a'), right=8 ('a'): tolower match -> left=2, right=7
left=2 ('c'), right=7 ('c'): tolower match -> left=3, right=6
left=3 ('e'), right=6 (' '): s[6] not alnum -> right=5
left=3 ('e'), right=5 ('a'): tolower 'e' != 'a' -> return false

Output: false  (correct)
```

### Complexity

| Approach | Time | Space |
|----------|------|-------|
| Brute force (clean + reverse) | O(n) | O(n) |
| Two pointers | O(n) | O(1) |
| Recursive two pointers | O(n) | O(n) call stack |

### Edge Cases

| Input | Expected | Reason |
|-------|----------|--------|
| `" "` | true | After removing non-alnum, empty; empty is palindrome |
| `"a"` | true | Single character always palindrome |
| `"0P"` | false | '0' != 'p' after tolower |
| `".,!"` | true | All non-alnum; cleaned string is empty |

### Interview Note

The recursive palindrome check is fundamentally a **divide-from-outside-inward** pattern. It reappears in: linked list palindrome (with a stack or reverse), tree symmetry check (LC 101), and recursive string parsing.

---

## 13. Problem: Fibonacci Number (LC 509)

### Problem Statement

**LeetCode 509 — Fibonacci Number (Easy)**

F(0) = 0, F(1) = 1, F(n) = F(n-1) + F(n-2). Given n, return F(n). Constraints: 0 <= n <= 30.

### Intuition

Fibonacci is not important because interviews ask Fibonacci. It matters because it is the canonical example of a recursive definition that **directly translates to code but hides exponential redundancy**. The naive recursive solution is correct and clean, but recomputes the same subproblems an exponential number of times. This problem motivates memoization and dynamic programming — Fibonacci is the gateway to DP.

### Approach 1: Naive Recursion (Brute Force)

```cpp
int fib(int n) {
    if (n <= 1) return n;       // F(0)=0, F(1)=1
    return fib(n-1) + fib(n-2);
}
// Time: O(2^n), Space: O(n) call stack depth
```

### Why O(2^n)?

Each call spawns two more calls. The recursion tree is a near-complete binary tree of depth n.

```
                    fib(5)
                  /        \
            fib(4)          fib(3)
           /      \         /    \
       fib(3)   fib(2)   fib(2)  fib(1)
       /    \   /    \   /    \
   fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)
   /    \
fib(1) fib(0)

fib(3) is computed TWICE. fib(2) is computed THREE TIMES.
```

The recurrence `T(n) = T(n-1) + T(n-2) + O(1)` resolves to `T(n) = O(φ^n)` where `φ ≈ 1.618` (the golden ratio). This is exponential.

### Approach 2: Memoization — Top-Down DP

Cache the result of each subproblem. Each subproblem is then computed exactly once.

```cpp
int fib_memo(int n, vector<int>& memo) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];      // already computed
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo);
    return memo[n];
}

int fib(int n) {
    vector<int> memo(n + 1, -1);
    return fib_memo(n, memo);
}
// Time: O(n), Space: O(n) memo + O(n) call stack = O(n)
```

### Approach 3: Bottom-Up DP (Tabulation)

Build from base cases upward, no recursion.

```cpp
int fib(int n) {
    if (n <= 1) return n;
    vector<int> dp(n + 1);
    dp[0] = 0; dp[1] = 1;
    for (int i = 2; i <= n; i++)
        dp[i] = dp[i-1] + dp[i-2];
    return dp[n];
}
// Time: O(n), Space: O(n)
```

### Approach 4: Space-Optimised Iteration (optimal for most purposes)

Only the previous two values are ever needed.

```cpp
int fib(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
// Time: O(n), Space: O(1)
```

### Approach 5: Matrix Exponentiation (O(log n))

Using the identity:

```
[F(n+1)]   [1 1]^n   [F(1)]
[F(n)  ] = [1 0]   * [F(0)]
```

Fast matrix exponentiation computes the nth power in O(log n) matrix multiplications, each O(1) for 2×2 matrices.

```cpp
typedef vector<vector<long long>> Matrix;

Matrix multiply(const Matrix& A, const Matrix& B) {
    Matrix C(2, vector<long long>(2, 0));
    for (int i = 0; i < 2; i++)
        for (int j = 0; j < 2; j++)
            for (int k = 0; k < 2; k++)
                C[i][j] += A[i][k] * B[k][j];
    return C;
}

Matrix matpow(Matrix M, int p) {
    Matrix result = {{1,0},{0,1}};   // 2x2 identity
    while (p > 0) {
        if (p & 1) result = multiply(result, M);
        M = multiply(M, M);
        p >>= 1;
    }
    return result;
}

int fib(int n) {
    if (n <= 1) return n;
    Matrix M = {{1,1},{1,0}};
    return (int)matpow(M, n)[0][1];
}
// Time: O(log n), Space: O(1)
```

### Dry Run — Space-Optimised (n = 6)

```
Target: F(6) = 8

prev2=0, prev1=1
i=2: curr=0+1=1,   prev2=1, prev1=1
i=3: curr=1+1=2,   prev2=1, prev1=2
i=4: curr=1+2=3,   prev2=2, prev1=3
i=5: curr=2+3=5,   prev2=3, prev1=5
i=6: curr=3+5=8,   prev2=5, prev1=8

return prev1 = 8  (correct: 0,1,1,2,3,5,8)
```

### Complexity

| Approach | Time | Space |
|----------|------|-------|
| Naive recursion | O(2^n) | O(n) |
| Memoization | O(n) | O(n) |
| Bottom-up DP | O(n) | O(n) |
| Space-optimised iteration | O(n) | O(1) |
| Matrix exponentiation | O(log n) | O(1) |

### Edge Cases

| Input | Expected | Reason |
|-------|----------|--------|
| n = 0 | 0 | F(0) = 0 by definition |
| n = 1 | 1 | F(1) = 1 by definition |
| n = 30 | 832040 | Maximum constraint |

---

## 14. Analysing Complexity via Recursion Trees

The recursion tree is the standard tool for deriving time complexity. The rule is:

```
Time Complexity = (number of nodes in the recursion tree) x (work done per node)
```

### Linear Recursion (one call per level, shrinks by 1)

```
f(n) -> f(n-1) -> f(n-2) -> ... -> f(0)
```

Tree is a chain of depth n. Each node does O(1) work. Total: **O(n)**.

Applies to: factorial, sum, print-1-to-N, reverse string, palindrome.

### Binary Recursion (two calls per level, shrinks by 1)

```
f(n) -> f(n-1) and f(n-2)
```

Tree is a near-complete binary tree of depth n. Nodes at level k: ~2^k. Total nodes: ~2^0 + 2^1 + ... + 2^n = **O(2^n)**. Work per node: O(1). Total: **O(2^n)**.

Applies to: naive Fibonacci.

### Divide-and-Conquer (two calls, shrinks by half)

```
f(n) -> f(n/2) and f(n/2)
```

Tree has depth log n. Nodes at level k: 2^k, each processing n/2^k elements. Work at each level: O(n). Total: **O(n log n)**.

Applies to: merge sort (covered in later lectures).

### The Master Theorem (reference for divide-and-conquer)

For `T(n) = a * T(n/b) + f(n)`, let `c = log_b(a)`:

| Condition | Result |
|-----------|--------|
| `f(n) = O(n^(c-ε))` | `T(n) = Θ(n^c)` |
| `f(n) = Θ(n^c)` | `T(n) = Θ(n^c * log n)` |
| `f(n) = Ω(n^(c+ε))` | `T(n) = Θ(f(n))` |

---

## 15. Stack Overflow: When Recursion Breaks

A stack overflow occurs when recursion depth exceeds available call stack memory. The runtime crashes — this is not a graceful error.

**Estimate safe depth:** Each stack frame is typically 50–200 bytes. With a 1 MB stack: `1,000,000 / 100 ≈ 10,000` frames. For competitive programming, treat recursion depth greater than ~10^4 as a stack overflow risk.

**Most common causes:**

1. Missing base case — recursion never terminates.
2. Recursion depth too large for the input size — even correct recursion fails at scale.

```cpp
// WRONG — will crash for any n > 0
int broken(int n) {
    return n + broken(n - 1);   // no base case
}

// CORRECT
int fixed(int n) {
    if (n == 0) return 0;
    return n + fixed(n - 1);
}
```

**When to use iterative instead of recursive:**

- When n > 10^4 and recursion is linear depth.
- When the problem involves deep graph/tree DFS with large inputs.
- When the problem explicitly asks for O(1) space.

Converting recursion to iteration: replace the implicit call stack with an explicit `stack<State>` data structure. Pop a state, process it, push next states.

---

## 16. Backtracking vs Recursion

Students conflate these. They are not the same.

**Recursion** is a function calling itself to solve a smaller problem. It explores one path to the solution.

**Backtracking** is recursion plus **undoing choices**. After a recursive call returns, you restore the state to what it was before the call, then try a different choice. This allows exploring all possible paths through a search space.

```
Backtracking template:
  Make a choice
  Recurse (explore with that choice)
  Undo the choice  <-- this is what makes it backtracking
```

| Property | Recursion | Backtracking |
|----------|-----------|--------------|
| Purpose | Solve a smaller version of the same problem | Explore all possibilities in a search space |
| Paths explored | Typically one | Multiple branches |
| State modification | Usually not required | Required — state must be restored |
| Example problems | Factorial, sum, Fibonacci | Subsets, permutations, N-Queens, Sudoku |

The print-1-to-N backtracking style (recurse first, print after) is the simplest possible glimpse of this — after the inner call returns, execution continues in the current frame. No state to undo there, but the structural concept is identical.

---

## 17. Where Recursion Leads: Later Topics

Every major DSA topic after this point uses recursion as its foundation.

| Topic | How recursion applies |
|-------|-----------------------|
| Trees | DFS traversals (preorder/inorder/postorder), height, diameter, path problems |
| Graphs | DFS, connected components, cycle detection |
| Dynamic Programming | Memoized recursion (top-down DP) |
| Backtracking | Subsets, permutations, N-Queens, Sudoku |
| Divide and Conquer | Merge sort, quick sort, binary search |
| Binary Search Trees | Insertion, deletion, LCA |
| Segment Trees | Build, query, update |
| Tries | Insert, search |

The pre/post-order output distinction from Print 1 to N / Print N to 1 directly maps to tree traversals. The two-pointer pattern from reverse and palindrome directly maps to recursive string and linked-list problems. The exponential recursion tree of Fibonacci directly motivates memoization and DP.

---

## 18. Common Mistakes and Pitfalls

### Mistake 1: Missing base case

```cpp
// WRONG — infinite recursion, stack overflow
int sum_broken(int n) {
    return n + sum_broken(n - 1);
}

// CORRECT
int sum_fixed(int n) {
    if (n == 0) return 0;
    return n + sum_fixed(n - 1);
}
```

### Mistake 2: Wrong base case value for factorial

```cpp
// WRONG — base case returns 0, zeroes out the entire product
long long factorial_broken(int n) {
    if (n == 0) return 0;               // 0! should be 1
    return n * factorial_broken(n - 1);
}

// CORRECT
long long factorial_fixed(int n) {
    if (n == 0) return 1;               // 0! = 1
    return n * factorial_fixed(n - 1);
}
```

### Mistake 3: Not moving toward the base case

```cpp
// WRONG — n grows away from base case 0, never terminates
void broken(int n) {
    if (n == 0) return;
    broken(n + 1);      // should be n - 1
}
```

### Mistake 4: Unsigned integer underflow in size-1 expressions

```cpp
// WRONG — s.size() is size_t (unsigned). If s is empty,
// s.size() - 1 wraps to a huge positive number.
void reverse_broken(vector<char>& s) {
    reverseHelper(s, 0, s.size() - 1);  // DANGEROUS if s is empty
}

// CORRECT
void reverse_fixed(vector<char>& s) {
    reverseHelper(s, 0, (int)s.size() - 1);  // cast to int first
}
```

### Mistake 5: Forgetting non-alphanumeric handling in palindrome

```cpp
// WRONG — raw comparison fails for "A man, a plan, a canal: Panama"
bool broken(string s, int l, int r) {
    if (l >= r) return true;
    if (s[l] != s[r]) return false;     // compares ',' against 'm', etc.
    return broken(s, l+1, r-1);
}

// CORRECT
bool fixed(string s, int l, int r) {
    if (l >= r) return true;
    if (!isalnum(s[l])) return fixed(s, l+1, r);
    if (!isalnum(s[r])) return fixed(s, l, r-1);
    if (tolower(s[l]) != tolower(s[r])) return false;
    return fixed(s, l+1, r-1);
}
```

### Mistake 6: Using int for factorial (overflow at n = 13)

```cpp
// WRONG — overflows for n >= 13
int factorial_int(int n) {
    if (n == 0) return 1;
    return n * factorial_int(n - 1);    // silent overflow
}

// CORRECT
long long factorial_ll(int n) {
    if (n == 0) return 1LL;
    return (long long)n * factorial_ll(n - 1);
}
```

### Mistake 7: Claiming O(1) space for recursive algorithms

Recursive stack depth always contributes O(depth) space. Stating "Space: O(1)" for a recursive solution is wrong. The correct answer for a linear recursion of depth n is O(n) space, even if no auxiliary data structures are allocated.

### Mistake 8: Using naive Fibonacci recursion in a performance context

For `n = 30` (the LC 509 constraint), naive recursion makes approximately 2.7 million calls — fine for a single call. If called many times in a loop or for larger n, it becomes unacceptable. Default to the iterative O(n) O(1) space version.

---

## 19. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---------|----------|------|-------|
| Print name N times | Parameterised recursion | O(n) | O(n) |
| Print 1 to N | Parameterised (count up) | O(n) | O(n) |
| Print 1 to N | Functional (print after call) | O(n) | O(n) |
| Print N to 1 | Parameterised (count down) | O(n) | O(n) |
| Print N to 1 | Functional (print before call) | O(n) | O(n) |
| Sum 1..N | Recursive | O(n) | O(n) |
| Sum 1..N | Iterative | O(n) | O(1) |
| Sum 1..N | Formula n(n+1)/2 | O(1) | O(1) |
| Factorial | Recursive | O(n) | O(n) |
| Factorial | Iterative | O(n) | O(1) |
| Reverse string | Iterative two pointers | O(n) | O(1) |
| Reverse string | Recursive two pointers | O(n) | O(n) |
| Valid palindrome | Two pointers | O(n) | O(1) |
| Valid palindrome | Brute (clean + reverse) | O(n) | O(n) |
| Valid palindrome | Recursive two pointers | O(n) | O(n) |
| Fibonacci | Naive recursion | O(2^n) | O(n) |
| Fibonacci | Memoization | O(n) | O(n) |
| Fibonacci | Bottom-up DP | O(n) | O(n) |
| Fibonacci | Space-optimised iteration | O(n) | O(1) |
| Fibonacci | Matrix exponentiation | O(log n) | O(1) |

---

## 20. Quick Revision Cheat Sheet

This section is self-contained. Everything needed for a same-day interview revision is here.

---

### Core Structure of Every Recursive Function

```cpp
ReturnType solve(params) {
    if (base_condition) return base_value;  // ALWAYS write this first
    // optional: work before call (executes on the way DOWN)
    solve(smaller_input);
    // optional: work after call (executes on the way BACK UP)
}
```

---

### The Two Phases

```
Going DOWN  -> calls are being made, frames pushed onto stack
Coming UP   -> calls are returning, frames popped, deferred work executes
```

Work before the recursive call executes going down. Work after the recursive call executes coming back up. This controls output order for printing problems, and shapes DFS traversal in trees.

---

### Parameterised vs Functional

```
Parameterised: state in params, function is void, uses side effects
Functional:    function returns a value built from recursive call's return
```

---

### Head vs Tail at a Glance

```
cout << n;       // print BEFORE call -> top-down output (N to 1)
recurse(n-1);

recurse(n-1);
cout << n;       // print AFTER call -> bottom-up output (1 to N)
```

---

### Key Recurrences

```
sum(n)         = n + sum(n-1)           |  sum(0) = 0
factorial(n)   = n * factorial(n-1)     |  factorial(0) = 1
fib(n)         = fib(n-1) + fib(n-2)   |  fib(0)=0, fib(1)=1
reverse(l,r)   = swap(l,r), recurse(l+1, r-1)  |  base: l >= r
isPalin(l,r)   = s[l]==s[r] && recurse(l+1, r-1)  |  base: l >= r
```

---

### Complexity Rules

```
Linear recursion  (1 call, depth n):        O(n) time,  O(n) space
Binary recursion  (2 calls, depth n):       O(2^n) time, O(n) space
Divide and halve  (2 calls, depth log n):   O(n log n) time, O(log n) space
```

Every recursion contributes O(depth) space from the call stack, even with no auxiliary allocations.

---

### Critical Numeric Facts

```
Sum formula:          n(n+1)/2   — use long long for large n
Factorial overflow:   int overflows at n=13; long long overflows at n=21
Fibonacci:            use iterative O(n) O(1) by default; naive is O(2^n)
Safe recursion depth: ~10^4 on most systems
```

---

### Fibonacci Hierarchy (fastest to slowest)

```
Matrix exponentiation  O(log n)  O(1)    — for very large n
Space-optimised iter   O(n)      O(1)    — use this by default
Bottom-up DP           O(n)      O(n)
Memoization            O(n)      O(n)
Naive recursion        O(2^n)    O(n)    — only for demonstration
```

---

### Interview Procedure for Any Recursion Problem

```
1. Write the recurrence relation (one line, mathematically).
2. Identify all base cases explicitly.
3. Write code directly from the recurrence.
4. State time = nodes in recursion tree x work per node.
5. State space = max recursion depth + any auxiliary storage.
6. Mention iterative alternative if depth > ~10^4.
```

---

### Pattern Map

| Situation | Pattern to reach for |
|-----------|----------------------|
| Print sequence in order | Parameterised recursion, count up |
| Print sequence in reverse | Work before call, count down |
| Compute value from subresults | Functional recursion |
| Two-pointer shrink (reverse, palindrome) | Recursive helper with left/right params |
| Explore all possibilities | Backtracking |
| Overlapping subproblems, optimal value | Memoized recursion (DP) |
| Divide problem in half | Divide and conquer |

---

### How Recursion Maps to Later Topics

```
Work before call (pre-order)   ->  preorder tree traversal
Work after call  (post-order)  ->  postorder traversal, DP on trees
Two-pointer shrink             ->  linked list palindrome, tree symmetry
Exponential recursion tree     ->  Fibonacci -> memoization -> all of DP
Explore all branches           ->  backtracking: subsets, permutations, N-Queens
Recursive call on halves       ->  binary search, merge sort, quick sort
```
