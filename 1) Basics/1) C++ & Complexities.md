# C++ Basics & Complexity

---

# Topics Covered

1. Input / Output
2. Arrays and Strings
3. Data Types
4. If-Else Statements
5. Switch Statement
6. For Loops
7. While Loops
8. Functions
9. Pass by Value vs Reference
10. Time Complexity
11. Space Complexity

---

# 1. INPUT / OUTPUT

---

# What Input/Output Really Means

Programs need communication.

* Input → data enters program
* Output → program displays result

Without I/O, algorithms are useless.

---

# Standard Input/Output in C++

```cpp
#include <iostream>
using namespace std;

int main() {
    int x;
    cin >> x;

    cout << x;
}
```

---

# Important Streams

| Stream | Meaning          |
| ------ | ---------------- |
| cin    | Input stream     |
| cout   | Output stream    |
| cerr   | Error output     |
| endl   | New line + flush |

---

# Fast Input Output

Important for competitive programming.

```cpp
ios::sync_with_stdio(false);
cin.tie(NULL);
```

---

# Why Fast I/O Matters

Default C++ streams synchronize with C streams.

That synchronization slows execution.

Fast I/O disables it.

---

# Important Interview Insight

For placements:

* Fast I/O usually unnecessary in interviews
* Very important in online coding rounds

---

# Common Mistakes

## Forgetting cin

```cpp
int x;
cout << x;
```

Uninitialized variable.

---

## Using endl Excessively

```cpp
endl
```

flushes buffer every time.

Slower than:

```cpp
"\n"
```

---

# 2. ARRAYS

---

# What Is An Array?

An array stores multiple values of same type in contiguous memory.

---

# Example

```cpp
int arr[5] = {1,2,3,4,5};
```

---

# Key Properties

| Property          | Meaning                  |
| ----------------- | ------------------------ |
| Contiguous Memory | Elements stored together |
| Fixed Size        | Size usually fixed       |
| Indexed Access    | O(1) access              |

---

# Memory Layout

Suppose:

```cpp
int arr[5];
```

If:

```text
arr[0] address = 1000
```

Then:

```text
arr[1] = 1004
arr[2] = 1008
```

because integer takes 4 bytes.

---

# Why Arrays Are Extremely Important

Arrays are the foundation of:

* vectors
* heaps
* matrices
* hashing
* DP tables
* segment trees
* graphs (adjacency matrix)

You cannot become strong in DSA without array mastery.

---

# Array Traversal

```cpp
for(int i = 0; i < n; i++) {
    cout << arr[i];
}
```

---

# Complexity

| Operation     | Complexity |
| ------------- | ---------- |
| Access        | O(1)       |
| Search        | O(N)       |
| Insert at End | O(1)       |
| Insert Middle | O(N)       |
| Delete        | O(N)       |

---

# Real DSA Insight

Arrays dominate interviews because they test:

* indexing
* boundary handling
* optimization
* pattern recognition

---

# 3. STRINGS

---

# What Is A String?

A string is a sequence of characters.

In C++:

```cpp
string s = "hello";
```

---

# Important Operations

| Operation | Example    |
| --------- | ---------- |
| Length    | s.length() |
| Access    | s[i]       |
| Append    | s += 'a'   |
| Substring | s.substr() |

---

# String Memory Insight

Strings internally use arrays of characters.

Understanding this helps later in:

* sliding window
* hashing
* KMP
* tries
* suffix arrays

---

# Character Arithmetic

```cpp
'A' + 1 = 'B'
```

Because characters are ASCII integers internally.

---

# Important ASCII Values

| Character | ASCII |
| --------- | ----- |
| A         | 65    |
| a         | 97    |
| 0         | 48    |

---

# Useful Conversion

```cpp
char ch = '7';
int num = ch - '0';
```

---

# 4. DATA TYPES

---

# Why Data Types Matter

Wrong datatype selection causes:

* overflow
* memory waste
* incorrect answers
* hidden bugs

Strong programmers think about ranges.

---

# Common Data Types

| Type      | Size    | Range          |
| --------- | ------- | -------------- |
| int       | 4 bytes | ~10^9          |
| long long | 8 bytes | ~10^18         |
| char      | 1 byte  | character      |
| float     | 4 bytes | decimal        |
| double    | 8 bytes | larger decimal |
| bool      | 1 byte  | true/false     |

---

# Most Important Placement Insight

If constraints contain:

```text
10^10
```

then:

```cpp
int
```

will overflow.

Use:

```cpp
long long
```

---

# Integer Overflow Example

```cpp
int x = 100000;
int y = 100000;

cout << x * y;
```

Overflow occurs.

Correct:

```cpp
long long ans = 1LL * x * y;
```

---

# 5. IF ELSE STATEMENTS

---

# Purpose

Decision making.

Programs become intelligent only through conditions.

---

# Syntax

```cpp
if(condition) {

}
else {

}
```

---

# Important Operators

| Operator | Meaning       |
| -------- | ------------- |
| ==       | Equal         |
| !=       | Not equal     |
| >        | Greater       |
| <        | Smaller       |
| >=       | Greater equal |
| <=       | Smaller equal |

---

# Logical Operators

| Operator | Meaning |   |    |
| -------- | ------- | - | -- |
| &&       | AND     |   |    |
|          |         |   | OR |
| !        | NOT     |   |    |

---

# Common Interview Mistake

```cpp
if(a = b)
```

instead of:

```cpp
if(a == b)
```

This causes assignment.

Very common bug.

---

# 6. SWITCH STATEMENT

---

# Purpose

Cleaner alternative to multiple if-else cases.

---

# Syntax

```cpp
switch(x) {

    case 1:
        cout << "One";
        break;

    case 2:
        cout << "Two";
        break;

    default:
        cout << "Invalid";
}
```

---

# Important Concept — Fall Through

Without break:

execution continues into next case.

---

# Example

```cpp
case 1:
case 2:
    cout << "Small";
```

Both cases share logic.

---

# When NOT To Use Switch

Switch does not support:

* ranges
* floating point
* complex conditions

---

# 7. FOR LOOPS

---

# Why Loops Matter

Loops are one of the most important concepts in programming.

Most DSA problems rely heavily on iteration.

---

# Syntax

```cpp
for(initialization; condition; update)
```

---

# Example

```cpp
for(int i = 0; i < 5; i++) {
    cout << i;
}
```

---

# Mental Model

Loop contains:

1. start point
2. stop condition
3. movement

---

# Infinite Loop Example

```cpp
for(;;)
```

Infinite loop.

---

# Important DSA Insight

Most time complexity comes from loops.

Understanding loops deeply is critical.

---

# Nested Loops

```cpp
for(int i = 0; i < n; i++) {
    for(int j = 0; j < n; j++) {

    }
}
```

Complexity:

```text
O(N²)
```

---

# 8. WHILE LOOPS

---

# Syntax

```cpp
while(condition) {

}
```

---

# Difference From For Loop

Use while when number of iterations is unknown.

---

# Example

```cpp
int n;
cin >> n;

while(n > 0) {
    cout << n;
    n--;
}
```

---

# Real DSA Relevance

While loops are heavily used in:

* binary search
* linked lists
* trees
* graph traversal
* sliding window
* two pointers

---

# Common Mistake

Forgetting update statement.

Causes infinite loop.

---

# 9. FUNCTIONS

---

# Why Functions Matter

Functions allow:

* modular code
* reuse
* abstraction
* cleaner debugging

Large systems are impossible without functions.

---

# Syntax

```cpp
returnType functionName(parameters) {

}
```

---

# Example

```cpp
int add(int a, int b) {
    return a + b;
}
```

---

# Important Concepts

| Concept       | Meaning   |
| ------------- | --------- |
| Parameters    | Inputs    |
| Return Value  | Output    |
| Function Call | Execution |

---

# Real DSA Insight

Good decomposition matters.

Strong programmers split problems into smaller functions.

---

# 10. PASS BY VALUE VS PASS BY REFERENCE

---

# Pass By Value

Copy is created.

Original variable unchanged.

---

# Example

```cpp
void change(int x) {
    x = 100;
}
```

Original variable remains same.

---

# Pass By Reference

Original variable directly modified.

---

# Example

```cpp
void change(int &x) {
    x = 100;
}
```

---

# Why References Matter

Without references:

* copying large arrays becomes expensive
* performance drops

---

# Extremely Important Placement Insight

Never pass large vectors by value.

Bad:

```cpp
void solve(vector<int> v)
```

Good:

```cpp
void solve(vector<int>& v)
```

---

# Complexity Impact

Passing by value copies entire object.

That may become:

```text
O(N)
```

unexpectedly.

Huge hidden bug.

---

# 11. TIME COMPLEXITY

---

# Most Important Beginner Topic

Time complexity measures how runtime grows with input size.

This is the foundation of algorithm analysis.

---

# Why Complexity Matters

A correct algorithm that takes 100 years is useless.

Interviewers care about:

* correctness
* efficiency

Both matter.

---

# Big O Notation

Represents upper bound of runtime growth.

---

# Common Complexities

| Complexity | Name         |
| ---------- | ------------ |
| O(1)       | Constant     |
| O(log N)   | Logarithmic  |
| O(N)       | Linear       |
| O(N log N) | Linearithmic |
| O(N²)      | Quadratic    |
| O(2^N)     | Exponential  |
| O(N!)      | Factorial    |

---

# O(1)

Constant operations.

```cpp
int x = arr[5];
```

---

# O(N)

Single loop.

```cpp
for(int i = 0; i < n; i++)
```

---

# O(N²)

Nested loops.

```cpp
for(int i = 0; i < n; i++) {
    for(int j = 0; j < n; j++) {

    }
}
```

---

# O(log N)

Input reduces repeatedly.

Example:

```cpp
while(n > 1) {
    n /= 2;
}
```

---

# Most Important Complexity Skill

Learn counting operations.

Not memorization.

---

# Rules For Complexity Analysis

---

# Rule 1 — Ignore Constants

```text
O(2N) = O(N)
```

---

# Rule 2 — Ignore Smaller Terms

```text
O(N² + N) = O(N²)
```

Dominant term matters.

---

# Rule 3 — Sequential Loops Add

```cpp
for(int i=0;i<n;i++) {}
for(int i=0;i<n;i++) {}
```

Complexity:

```text
O(N + N) = O(N)
```

---

# Rule 4 — Nested Loops Multiply

```cpp
for(int i=0;i<n;i++) {
    for(int j=0;j<n;j++) {}
}
```

Complexity:

```text
O(N²)
```

---

# Important Constraint Analysis

---

# Typical Limits

| N    | Expected Complexity |
| ---- | ------------------- |
| 10   | O(N!) possible      |
| 20   | O(2^N) possible     |
| 10^2 | O(N³) maybe         |
| 10^3 | O(N²)               |
| 10^5 | O(N log N)          |
| 10^6 | O(N)                |

---

# This Table Is Extremely Important

Interview optimization decisions depend heavily on constraints.

---

# Hidden Complexity Trap

## STL Operations

Many beginners ignore STL complexity.

Example:

```cpp
vector.erase()
```

can become:

```text
O(N)
```

because shifting occurs.

---

# Amortized Complexity

Very important concept.

Example:

```cpp
vector.push_back()
```

Average:

```text
O(1)
```

Occasionally resizing occurs.

---

# 12. SPACE COMPLEXITY

---

# What Is Space Complexity?

Extra memory used by algorithm.

---

# Important Distinction

Input array memory usually not counted.

Extra memory is counted.

---

# Example

```cpp
int arr[n];
```

Input storage.

---

```cpp
vector<int> temp(n);
```

Extra space.

---

# Recursive Space Complexity

Recursion uses stack memory.

Example:

```cpp
factorial(n)
```

creates:

```text
O(N)
```

stack space.

---

# Why Space Complexity Matters

Sometimes memory limit exceeded occurs before TLE.

Strong programmers optimize both.

---

# Important Placement Insights

---

# Interviewers Usually Care About

1. Correctness
2. Clarity
3. Complexity
4. Edge cases
5. Communication

Not just final code.

---

# Strong Candidates Usually

* think aloud clearly
* estimate complexity early
* optimize step-by-step
* know tradeoffs

---

# Weak Candidates Usually

* jump into coding blindly
* ignore constraints
* cannot analyze complexity
* write messy loops

---

# Biggest Beginner Mistakes

---

# 1. Ignoring Constraints

Constraints determine algorithm choice.

---

# 2. Memorizing Complexity

Learn reasoning instead.

---

# 3. Weak Loop Understanding

Loops are the foundation of almost all DSA.

---

# 4. Ignoring Overflow

One of the most common bugs.

---

# 5. Passing Large Objects By Value

Creates hidden inefficiency.

---

# 6. Not Understanding Memory

Memory awareness matters.

---

# Final Reality Check

Most people underestimate fundamentals.

That becomes painful later.

When advanced topics arrive:

* recursion
* DP
* trees
* graphs
* segment trees

weak basics get exposed immediately.

---

# What You Should Actually Master From This Lecture

Not syntax.

You should deeply understand:

* loops
* conditions
* complexity
* arrays
* indexing
* references
* memory
* iteration
* runtime growth

These are the real foundations of DSA.

---

#
