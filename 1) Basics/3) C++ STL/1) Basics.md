### C++ STL

> **Based on Striver's A2Z DSA Sheet**
> [TakeUForward STL Reference](https://takeuforward.org/c/c-stl-tutorial-most-frequent-used-stl-containers)

---

## Table of Contents

**Part 1 — C++ Basics**
1. [Input / Output](#1-input--output)
2. [Data Types & Overflow](#2-data-types--overflow)
3. [Arrays](#3-arrays)
4. [Strings](#4-strings)
5. [Control Flow](#5-control-flow-if-else-switch)
6. [Loops](#6-loops)
7. [Functions & Pass by Reference](#7-functions--pass-by-reference)

**Part 2 — Complexity Analysis**

8. [Time Complexity](#8-time-complexity)
9. [Space Complexity](#9-space-complexity)

**Part 3 — C++ STL**

10. [STL Architecture](#10-stl-architecture)
11. [Pairs & Tuples](#11-pairs--tuples)
12. [Vectors](#12-vectors)
13. [List](#13-list)
14. [Deque](#14-deque)
15. [Stack](#15-stack)
16. [Queue](#16-queue)
17. [Priority Queue (Heap)](#17-priority-queue-heap)
18. [Set, Multiset, Unordered Set](#18-set-multiset-unordered-set)
19. [Map, Multimap, Unordered Map](#19-map-multimap-unordered-map)
20. [STL Algorithms](#20-stl-algorithms)
21. [Iterators](#21-iterators)

**Part 4 — Reference**

22. [Complexity Cheat Sheet](#22-complexity-cheat-sheet)
23. [Choosing the Right Container](#23-choosing-the-right-container)
24. [Common Interview Patterns](#24-common-interview-patterns)
25. [Common Mistakes & Pitfalls](#25-common-mistakes--pitfalls)
26. [Quick Revision Cheat Sheet](#26-quick-revision-cheat-sheet)

---

# PART 1 — C++ BASICS

---

## 1. Input / Output

```cpp
#include <iostream>
using namespace std;

int main() {
    int x;
    cin >> x;       // Standard input
    cout << x;      // Standard output
    cerr << "err";  // Error stream
}
```

### Fast I/O — Essential for Online Rounds

```cpp
ios::sync_with_stdio(false);
cin.tie(NULL);
```

**Why it matters:** By default, C++ streams synchronize with C streams — this is slow. Disabling it makes I/O ~3–5x faster.

> ⚠️ After enabling fast I/O, don't mix `printf`/`scanf` with `cin`/`cout`.

### `endl` vs `"\n"`

```cpp
cout << endl;   // Flushes buffer — SLOW (avoid in loops)
cout << "\n";   // Just newline — FAST (prefer this)
```

> 📌 **Placement Note:** Fast I/O is unnecessary in interviews but critical in online coding rounds (HackerRank, Codeforces, etc.)

---

## 2. Data Types & Overflow

### Size & Range

| Type | Size | Range | Use when |
|---|---|---|---|
| `int` | 4 bytes | ~±2×10⁹ | N ≤ 10⁹ |
| `long long` | 8 bytes | ~±9×10¹⁸ | N > 10⁹ |
| `char` | 1 byte | 0–255 | Characters |
| `float` | 4 bytes | ~7 decimal digits | Avoid in CP |
| `double` | 8 bytes | ~15 decimal digits | Decimals |
| `bool` | 1 byte | true / false | Flags |

### Integer Overflow — Most Common Bug

```cpp
// ❌ WRONG — overflows silently
int x = 100000, y = 100000;
int result = x * y;        // Overflow! (~10^10 doesn't fit in int)

// ✅ CORRECT
long long result = 1LL * x * y;
// OR
long long result = (long long)x * y;
```

### Constraint → Data Type Rule

```
N ≤ 10^9      → int is fine
N > 10^9      → use long long
Sum/Product   → almost always use long long
Indices       → int is fine (unless N > 2×10^9)
```

### Important Constants

```cpp
INT_MAX   = 2147483647     (~2 × 10^9)
INT_MIN   = -2147483648
LLONG_MAX = 9223372036854775807  (~9.2 × 10^18)
```

---

## 3. Arrays

### What Is An Array?

Contiguous memory block storing elements of the **same type**. Fixed size. O(1) random access.

```cpp
int arr[5] = {1, 2, 3, 4, 5};

// Memory: if arr[0] is at address 1000
// arr[1] = 1004, arr[2] = 1008  (int = 4 bytes each)
```

### Complexity

| Operation | Time |
|---|---|
| Access by index | O(1) |
| Search | O(N) |
| Insert at end | O(1) |
| Insert at middle | O(N) — shifting required |
| Delete | O(N) — shifting required |

### 2D Arrays

```cpp
int mat[3][4];                        // 3 rows, 4 columns
int mat[3][4] = {{1,2,3,4}, {5,6,7,8}, {9,10,11,12}};

mat[i][j];   // Access element at row i, column j
```

> 📌 **DSA Relevance:** Arrays underlie almost everything — heaps, hash tables, DP tables, segment trees, graphs (adjacency matrix). Master arrays first.

---

## 4. Strings

```cpp
string s = "hello";

s.length();         // 5
s.size();           // 5 (same as length)
s[0];               // 'h'
s += " world";      // Append
s.substr(1, 3);     // "ell" — substr(start, length)
s.find("lo");       // 3 — returns index, or string::npos if not found
s.empty();          // false
```

### Character Arithmetic (ASCII)

```cpp
'A' = 65,  'Z' = 90
'a' = 97,  'z' = 122
'0' = 48,  '9' = 57

// Useful conversions
char ch = '7';
int digit = ch - '0';     // 7

int idx = ch - 'a';       // 0-based index from 'a'
char upper = 'a' - 32;    // 'A'
```

### String ↔ Integer

```cpp
string s = to_string(42);      // int to string
int n = stoi("42");            // string to int
long long n = stoll("123456789012");  // string to long long
```

> 📌 **DSA Relevance:** Strings are key in sliding window, hashing, two pointers, KMP, tries, and palindrome problems.

---

## 5. Control Flow: If-Else, Switch

### If-Else

```cpp
if (condition) {
    // ...
} else if (other) {
    // ...
} else {
    // ...
}
```

### Operators

| Type | Operators |
|---|---|
| Comparison | `==`, `!=`, `<`, `>`, `<=`, `>=` |
| Logical | `&&` (AND), `\|\|` (OR), `!` (NOT) |
| Bitwise | `&`, `\|`, `^`, `~`, `<<`, `>>` |
| Ternary | `condition ? val_if_true : val_if_false` |

> ⚠️ **Classic Bug:** `if (a = b)` assigns b to a — always use `==` for comparison.

### Switch

```cpp
switch (x) {
    case 1:
        cout << "One";
        break;          // Without break, falls through to next case!
    case 2:
    case 3:
        cout << "Two or Three";  // Both cases share this code
        break;
    default:
        cout << "Other";
}
```

**Limitations of switch:** No ranges, no floats, no complex conditions. Use if-else for those.

---

## 6. Loops

### For Loop

```cpp
for (int i = 0; i < n; i++) { }      // Standard
for (int i = n-1; i >= 0; i--) { }   // Reverse
for (int i = 0; i < n; i += 2) { }   // Step of 2
```

### While Loop

```cpp
while (condition) {
    // Use when number of iterations is unknown
}

// Classic example: binary search manually
while (n > 1) {
    n /= 2;   // O(log N) — input halves each iteration
}
```

### Do-While

```cpp
do {
    // Executes at least once
} while (condition);
```

### Range-Based For (C++11)

```cpp
for (int x : arr) { }          // By value — copy
for (int& x : arr) { }         // By reference — can modify
for (const int& x : arr) { }   // By const ref — safe, no copy
```

### Nested Loop Complexity

```cpp
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)   // → O(N²)
        for (int k = 0; k < n; k++)  // → O(N³)
```

> ⚠️ **Most bugs and TLEs come from incorrect loop bounds or unexpected O(N) operations inside loops.**

---

## 7. Functions & Pass by Reference

### Syntax

```cpp
returnType functionName(parameters) {
    // body
    return value;
}
```

### Pass by Value vs Reference

```cpp
// Pass by VALUE — copy is made, original unchanged
void byValue(int x) { x = 100; }

// Pass by REFERENCE — original is modified
void byRef(int& x) { x = 100; }

// Pass by CONST REFERENCE — no copy, but can't modify
void byConstRef(const vector<int>& v) { /* read-only */ }
```

### Critical Rule for Containers

```cpp
// ❌ WRONG — copies entire vector: O(N) overhead
void solve(vector<int> v) { }

// ✅ CORRECT — no copy: O(1) overhead
void solve(vector<int>& v) { }
void solve(const vector<int>& v) { }  // if read-only
```

> 📌 Passing a `vector<int>` of size 10⁶ by value creates a hidden O(N) copy. This single mistake can turn an O(N log N) solution into TLE.

### Default Arguments & Overloading

```cpp
int power(int base, int exp = 2) { /* ... */ }  // Default arg

power(3);     // Uses exp = 2
power(3, 4);  // Uses exp = 4
```

---

# PART 2 — COMPLEXITY ANALYSIS

---

## 8. Time Complexity

### What It Is

Time complexity measures **how runtime grows as input size N increases**, not the actual time in seconds.

### Big O — Common Complexities

| Complexity | Name | Example | Max N (1 sec) |
|---|---|---|---|
| O(1) | Constant | Array access | Any |
| O(log N) | Logarithmic | Binary search | ~10^18 |
| O(N) | Linear | Single loop | ~10^8 |
| O(N log N) | Linearithmic | Merge sort | ~10^7 |
| O(N²) | Quadratic | Nested loops | ~10^4 |
| O(N³) | Cubic | Triple loops | ~10^3 |
| O(2^N) | Exponential | All subsets | ~20–25 |
| O(N!) | Factorial | All permutations | ~10–12 |

### Constraint → Expected Complexity (Critical Table)

| N (constraint) | Expected Algorithm |
|---|---|
| N ≤ 10 | O(N!) — brute force permutations |
| N ≤ 20 | O(2^N) — bitmask / subset DP |
| N ≤ 100 | O(N³) |
| N ≤ 1,000 | O(N²) |
| N ≤ 10^5 | O(N log N) |
| N ≤ 10^6 | O(N) |
| N ≤ 10^18 | O(log N) — math / binary search |

> 📌 **Always look at constraints before writing code.** This table tells you what complexity to target.

### Rules for Calculating Complexity

**Rule 1 — Drop constants**
```
O(3N) → O(N)
O(100) → O(1)
```

**Rule 2 — Drop smaller terms**
```
O(N² + N) → O(N²)
O(N log N + N) → O(N log N)
```

**Rule 3 — Sequential blocks add**
```cpp
for (i < n) { }    // O(N)
for (i < n) { }    // O(N)
// Total: O(N + N) = O(N)
```

**Rule 4 — Nested blocks multiply**
```cpp
for (i < n)        // O(N)
    for (j < n)    // O(N)
// Total: O(N × N) = O(N²)
```

### Hidden Complexity Traps

```cpp
// Each of these is NOT O(1):
v.erase(v.begin());          // O(N) — shifting
v.insert(v.begin(), x);      // O(N) — shifting
s.find("hello");             // O(N) — linear scan
sort(v.begin(), v.end());    // O(N log N)

// Accidentally quadratic:
for (int i = 0; i < n; i++)
    v.erase(v.begin());      // O(N) per iteration → O(N²) total!
```

### Amortized Complexity

Some operations are occasionally expensive but cheap on average.

```
vector::push_back() — usually O(1), sometimes O(N) for resize
Amortized over N operations: O(1) per push_back
```

---

## 9. Space Complexity

### What Is Counted

- Extra auxiliary memory used by your algorithm
- **Input storage is usually NOT counted**

```cpp
int arr[n];                  // Input — usually not counted
vector<int> temp(n, 0);      // Auxiliary — counted as O(N)
```

### Recursive Space

Each recursive call uses **stack frame memory**.

```cpp
void factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n-1);
}
// Call stack depth = N → O(N) space
```

### Common Space Complexities

| Code | Space |
|---|---|
| Fixed variables | O(1) |
| Array/vector of size N | O(N) |
| 2D array N×M | O(N×M) |
| Recursion depth D | O(D) |
| Hash map with N entries | O(N) |

---

# PART 3 — C++ STL

---

## 10. STL Architecture

The **Standard Template Library (STL)** provides ready-made, optimized data structures and algorithms.

```
STL
├── Containers   → Store data (vector, map, set, stack ...)
├── Algorithms   → Process data (sort, search, reverse ...)
└── Iterators    → Bridge between containers and algorithms
```

### The Magic Header

```cpp
#include <bits/stdc++.h>   // Includes everything — used in CP
using namespace std;
```

> ⚠️ `bits/stdc++.h` is non-standard and slow to compile. In interviews, prefer specific headers like `<vector>`, `<map>`, etc. Fine for online rounds.

### Internal Implementation

| Container | Internal Structure | Consequence |
|---|---|---|
| `vector` | Dynamic Array | O(1) random access |
| `list` | Doubly Linked List | O(1) insert given iterator |
| `deque` | Segmented Array | O(1) both ends |
| `stack` / `queue` | Adaptor (deque) | Restricted interface |
| `priority_queue` | Binary Heap | O(log N) push/pop |
| `set` / `map` | Red-Black Tree | O(log N) all ops, sorted |
| `unordered_set` / `unordered_map` | Hash Table | O(1) avg, unsorted |

---

## 11. Pairs & Tuples

### Pair

Stores **two values** of possibly different types.

```cpp
pair<int, string> p1 = {1, "hello"};
pair<int, int>    p2 = make_pair(3, 5);

p1.first;    // 1
p1.second;   // "hello"

// Nested pair
pair<int, pair<int, int>> nested = {1, {2, 3}};
nested.second.first;   // 2

// Comparison — lexicographic (first compared, then second)
pair<int,int> a = {1, 5}, b = {1, 3};
a > b;   // true (5 > 3 when firsts are equal)

// Vector of pairs
vector<pair<int,int>> vp;
vp.push_back({1, 2});
vp.emplace_back(3, 4);   // Slightly faster — constructs in-place
```

**Sorting vector of pairs:**

```cpp
vector<pair<int,int>> v = {{3,2},{1,5},{3,1}};

sort(v.begin(), v.end());  // Default: by first, then second (ascending)
// Result: {1,5}, {3,1}, {3,2}

// Custom: sort by second element descending
sort(v.begin(), v.end(), [](const auto& a, const auto& b){
    return a.second > b.second;
});
```

> 📌 **Interview Heavy-hitter:** Pairs appear in Dijkstra's `{dist, node}`, sorting with index `{value, index}`, interval problems `{start, end}`, and frequency maps `{freq, element}`.

### Tuple

Generalizes pair to **N values**.

```cpp
tuple<int, string, float> t = {1, "hello", 3.14f};

get<0>(t);   // 1
get<1>(t);   // "hello"
get<2>(t);   // 3.14

auto t2 = make_tuple(10, 'A', 3.5);

// Unpack with tie
int x; char c; double d;
tie(x, c, d) = t2;

// C++17 structured bindings
auto [a, b, f] = t;
```

---

## 12. Vectors

**Dynamic array.** Most used STL container.

### Declaration & Initialization

```cpp
vector<int> v1;                   // Empty
vector<int> v2(5);                // Size 5, all 0s
vector<int> v3(5, 10);           // Size 5, all 10s
vector<int> v4 = {1, 2, 3, 4, 5};
vector<int> v5(v4);              // Copy constructor
vector<int> v6(v4.begin(), v4.begin() + 3);  // {1, 2, 3}

// 2D vector
vector<vector<int>> mat(3, vector<int>(4, 0));  // 3×4 matrix of 0s
```

### Core Operations

```cpp
vector<int> v = {1, 2, 3, 4, 5};

// Size
v.size();        // 5 — number of elements
v.capacity();    // >= 5 — allocated memory
v.empty();       // false
v.resize(8);     // Size = 8, new elements = 0
v.reserve(20);   // Pre-allocate for 20 (avoids reallocation)

// Access
v[2];            // 3 — NO bounds check (UB if out of range)
v.at(2);         // 3 — WITH bounds check (throws out_of_range)
v.front();       // 1
v.back();        // 5

// Modify
v.push_back(6);                  // Add at end: O(1) amortized
v.pop_back();                    // Remove from end: O(1)
v.emplace_back(7);               // Construct in-place: slightly faster
v.insert(v.begin() + 2, 99);    // Insert at index 2: O(N)
v.erase(v.begin() + 2);         // Erase at index 2: O(N)
v.erase(v.begin(), v.begin()+3); // Erase range [0,3): O(N)
v.clear();                       // Remove all, size = 0

// Fill
fill(v.begin(), v.end(), 0);    // Fill all with 0
v.assign(5, -1);                // Replace with 5 copies of -1
```

### How Vector Grows

```
Initial: capacity = 1 (implementation-defined)
When full: capacity doubles (2x)

Amortized analysis:
  Total copies for N push_backs = 1+2+4+...+N/2 = N-1
  → O(N) total → O(1) per push_back
```

> ⚠️ After ANY reallocation, **all iterators, pointers, and references are invalidated**.

### Useful Tricks

```cpp
// Remove duplicates (sort first, then unique + erase)
sort(v.begin(), v.end());
v.erase(unique(v.begin(), v.end()), v.end());

// Flatten a 2D vector
vector<int> flat;
for (auto& row : mat)
    flat.insert(flat.end(), row.begin(), row.end());

// Swap two vectors (O(1))
v1.swap(v2);
```

---

## 13. List

**Doubly Linked List.** O(1) insertions/deletions at any position (given an iterator). No random access.

```cpp
#include <list>

list<int> l = {1, 2, 3, 4, 5};

l.push_front(0);   // O(1)
l.push_back(6);    // O(1)
l.pop_front();     // O(1)
l.pop_back();      // O(1)
l.front();         // First element
l.back();          // Last element

// Insert at iterator position — O(1) once you have the iterator
auto it = l.begin();
advance(it, 2);
l.insert(it, 99);

l.remove(3);       // Remove all occurrences of 3: O(N)
l.sort();          // Merge sort: O(N log N)
l.reverse();       // O(N)
l.unique();        // Remove consecutive duplicates (sort first!)
l.merge(l2);       // Merge two sorted lists: O(N)
```

**When to use list over vector:**
- Need O(1) insert/delete in the middle given an iterator
- Never need random access by index
- Classic use: LRU Cache implementation

---

## 14. Deque

**Double-Ended Queue.** O(1) insert/delete at both ends. O(1) random access.

```cpp
#include <deque>

deque<int> dq = {1, 2, 3};

dq.push_front(0);  // O(1)
dq.push_back(4);   // O(1)
dq.pop_front();    // O(1)
dq.pop_back();     // O(1)
dq[2];             // O(1) random access
dq.front();
dq.back();
```

> 📌 **Key Use Case:** Sliding window maximum uses a **monotonic deque**.

---

## 15. Stack

**LIFO — Last In, First Out.** Container adaptor over `deque`.

```cpp
#include <stack>

stack<int> st;

st.push(10);    // Add to top: O(1)
st.top();       // View top: O(1)  ← use BEFORE pop
st.pop();       // Remove top: O(1) — does NOT return value
st.empty();     // bool
st.size();      // count
```

> ⚠️ `pop()` does not return the value. Always call `top()` first.

### Key Problems Using Stack

| Problem | Technique |
|---|---|
| Valid Parentheses | Match open/close brackets |
| Next Greater Element | Monotonic Stack |
| Largest Rectangle in Histogram | Monotonic Stack |
| Min Stack | Auxiliary stack for minimum |
| Evaluate Postfix Expression | Operand stack |
| Celebrity Problem | Elimination |

### Monotonic Stack Pattern

```cpp
// Next Greater Element to the right
vector<int> nextGreater(vector<int>& arr) {
    int n = arr.size();
    vector<int> result(n, -1);
    stack<int> st;  // stores indices
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] < arr[i]) {
            result[st.top()] = arr[i];
            st.pop();
        }
        st.push(i);
    }
    return result;
}

// Previous Smaller Element
vector<int> prevSmaller(vector<int>& arr) {
    int n = arr.size();
    vector<int> res(n, -1);
    stack<int> st;
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] >= arr[i]) st.pop();
        if (!st.empty()) res[i] = arr[st.top()];
        st.push(i);
    }
    return res;
}
```

---

## 16. Queue

**FIFO — First In, First Out.**

```cpp
#include <queue>

queue<int> q;

q.push(10);    // Enqueue at back: O(1)
q.front();     // View front: O(1)
q.back();      // View back: O(1)
q.pop();       // Dequeue from front: O(1)
q.empty();     // bool
q.size();      // count

// ⚠️ Cannot initialize like vector:
// queue<int> q = {1,2,3};  ← DOES NOT COMPILE
```

> 📌 **BFS always uses a queue.** "Shortest path in unweighted graph" or "level-order traversal" → use queue.

---

## 17. Priority Queue (Heap)

Container adaptor giving the **highest priority element** at top. Uses a **Binary Heap** internally.

### Max Heap (Default)

```cpp
#include <queue>

priority_queue<int> pq;  // Max-heap

pq.push(10);
pq.push(30);
pq.push(20);

pq.top();    // 30 — largest: O(1)
pq.pop();    // Remove top: O(log N)
pq.push(5);  // Insert: O(log N)
```

### Min Heap

```cpp
// Method 1 — preferred
priority_queue<int, vector<int>, greater<int>> minPQ;

// Method 2 — negate values (hack)
priority_queue<int> pq;
pq.push(-10);           // Push negated
cout << -pq.top();      // Read actual value
```

### Priority Queue of Pairs

```cpp
// Max-heap by first element (default)
priority_queue<pair<int,int>> pq;

// Min-heap by first element
priority_queue<pair<int,int>, vector<pair<int,int>>,
               greater<pair<int,int>>> minPQ;
```

### Custom Comparator

```cpp
struct Compare {
    bool operator()(const pair<int,int>& a, const pair<int,int>& b) {
        return a.first > b.first;  // Min-heap by first
        // "return true" means a has LOWER priority than b
    }
};
priority_queue<pair<int,int>, vector<pair<int,int>>, Compare> pq;
```

### Heap Concepts (Interview Theory)

```
Max-Heap: Parent >= Children
         50
        /  \
       30   40
      / \   /
     10  20 35

Insert:     Add at end, bubble-up   → O(log N)
Delete top: Replace with last, bubble-down → O(log N)
Peek top:   O(1)
Build heap from N elements: O(N)   ← NOT O(N log N)! Interview trap.
```

> ⚠️ **Comparator Trap:** In `priority_queue`, `greater<int>` → **min-heap** (smallest at top). In `sort()`, `greater<int>` → **descending order**. Same comparator, opposite mental model.

### Key Problems Using Priority Queue

| Problem | Approach |
|---|---|
| K Largest Elements | Min-heap of size K |
| K Closest Points | Max-heap of size K |
| Merge K Sorted Lists | Min-heap of (value, list_index) |
| Dijkstra's Algorithm | Min-heap of (dist, node) |
| Top K Frequent Elements | Max-heap by frequency |
| Median in Data Stream | Two heaps (max + min) |

---

## 18. Set, Multiset, Unordered Set

### Set

Stores **unique, sorted** elements. Backed by Red-Black Tree.

```cpp
#include <set>

set<int> s = {3, 1, 4, 1, 5, 9, 2, 6};
// Stored: {1, 2, 3, 4, 5, 6, 9} — sorted, no duplicates

s.insert(7);          // O(log N)
s.erase(3);           // O(log N)
s.count(5);           // 0 or 1 — O(log N)
s.find(4);            // Iterator or s.end() — O(log N)
s.size();             // O(1)

// Range queries — use MEMBER functions (not std:: versions)
auto lb = s.lower_bound(4);  // First element >= 4 — O(log N)
auto ub = s.upper_bound(4);  // First element > 4  — O(log N)

*s.begin();    // Minimum element
*s.rbegin();   // Maximum element

// Membership check
if (s.count(5)) { /* found */ }
if (s.find(5) != s.end()) { /* found */ }
```

> ⚠️ **Critical:** For `set`/`map`, always use the **member function** `s.lower_bound()`, not `std::lower_bound()`. The generic version is O(N) for sets (no random access). The member version is O(log N).

### Multiset

Like `set` but **allows duplicates**.

```cpp
multiset<int> ms = {1, 2, 2, 3, 3, 3};

ms.insert(2);               // Now three 2s
ms.count(3);                // 3
ms.erase(2);                // ⚠️ Removes ALL 2s!
ms.erase(ms.find(2));       // ✅ Removes just ONE 2

ms.lower_bound(2);          // First position of 2
ms.upper_bound(2);          // First position after all 2s
```

> ⚠️ **Most common multiset mistake:** `ms.erase(value)` deletes **all** copies. Use `ms.erase(ms.find(value))` to delete just one.

### Unordered Set

Hash table. Average **O(1)**, no ordering.

```cpp
#include <unordered_set>

unordered_set<int> us = {3, 1, 4, 1, 5};
// Stored: unique, NO guaranteed order

us.insert(7);     // O(1) average
us.erase(3);      // O(1) average
us.count(5);      // O(1) average
us.find(4);       // O(1) average

// ⚠️ No lower_bound/upper_bound — not sorted
// ⚠️ Worst case O(N) due to hash collisions
```

### Comparison

| Feature | `set` | `unordered_set` |
|---|---|---|
| Order | Sorted | No order |
| Lookup | O(log N) | O(1) avg, O(N) worst |
| Insert | O(log N) | O(1) avg |
| lower_bound | ✅ O(log N) | ❌ Not available |
| Use when | Need sorted/range queries | Need fast lookup |

---

## 19. Map, Multimap, Unordered Map

### Map

**Key-value pairs, sorted by key, unique keys.** Red-Black Tree.

```cpp
#include <map>

map<string, int> m;

// Insertion
m["apple"] = 5;
m.insert({"banana", 3});
m.emplace("cherry", 7);

// Access
m["apple"];          // 5
m.at("apple");       // 5 — throws if key absent (safer)
m["newkey"];         // ⚠️ Creates entry with value 0!

// Safe existence check
m.count("apple");                    // 1 if exists, 0 if not
m.find("apple") != m.end();         // true if exists

// Erase
m.erase("apple");                   // O(log N)

// Iteration — always sorted by key
for (auto& [key, val] : m) {        // C++17 structured binding
    cout << key << ": " << val;
}

// Range queries
auto it = m.lower_bound("banana");  // Iterator to "banana" or next key
auto it2 = m.upper_bound("banana"); // Iterator after "banana"

m.begin()->first;    // Smallest key
m.rbegin()->first;   // Largest key
```

> ⚠️ **Critical:** `m["key"]` creates a default-value entry if key doesn't exist. Use `m.count("key")` or `m.find("key")` for safe checking.

### Multimap

Allows **duplicate keys**.

```cpp
multimap<int, string> mm;
mm.insert({1, "a"});
mm.insert({1, "b"});  // Duplicate key allowed

// Cannot use mm[key] — NOT defined for multimap!

// Get all values for a key
auto range = mm.equal_range(1);
for (auto it = range.first; it != range.second; it++)
    cout << it->second;   // "a", "b"
```

### Unordered Map

Hash table. Average **O(1).** Most commonly used in interviews.

```cpp
#include <unordered_map>

unordered_map<string, int> um;
um["a"] = 1;
um["b"] = 2;

um.count("a");    // O(1)
um.find("a");     // Iterator — O(1)
um.erase("a");    // O(1)

// Same pitfall: um["newkey"] creates entry
```

### Comparison

| Feature | `map` | `unordered_map` |
|---|---|---|
| Order | Sorted by key | No order |
| Lookup | O(log N) | O(1) avg |
| lower_bound | ✅ O(log N) | ❌ Not available |
| Use when | Need sorted keys, range queries | Need fast lookup (~4× faster) |

---

## 20. STL Algorithms

Header: `<algorithm>` (included in `bits/stdc++.h`)

### Sorting

```cpp
sort(v.begin(), v.end());                           // Ascending: O(N log N)
sort(v.begin(), v.end(), greater<int>());           // Descending
sort(v.begin(), v.begin() + 4);                     // Partial sort

// Custom comparator
sort(vp.begin(), vp.end(), [](const auto& a, const auto& b) {
    return a.second > b.second;   // Sort pairs by second, descending
});

stable_sort(v.begin(), v.end());   // Preserves relative order of equals
```

> 📌 `sort()` uses **IntroSort** — Quicksort + Heapsort + Insertion Sort. **Guaranteed O(N log N) worst case**, unlike pure Quicksort.

### Binary Search (Requires Sorted Input!)

```cpp
vector<int> v = {1, 2, 3, 4, 5, 6, 7};

binary_search(v.begin(), v.end(), 4);   // Returns bool

// lower_bound — first element >= val
auto lb = lower_bound(v.begin(), v.end(), 4);
*lb;                     // 4
lb - v.begin();          // 3 (index = count of elements < val)

// upper_bound — first element > val
auto ub = upper_bound(v.begin(), v.end(), 4);
*ub;                     // 5

// Count occurrences of val in sorted array
int cnt = upper_bound(v.begin(), v.end(), val)
        - lower_bound(v.begin(), v.end(), val);
```

**Visual:**
```
Array:  1  2  3  4  4  4  5  7
Index:  0  1  2  3  4  5  6  7

lower_bound(4) → index 3
upper_bound(4) → index 6
Count of 4s = 6 - 3 = 3  ✓
```

### Min & Max

```cpp
max(3, 5);                              // 5
min(3, 5);                              // 3
max({1, 5, 3});                         // 5 — initializer list (C++11)

*max_element(v.begin(), v.end());       // Max value — O(N)
*min_element(v.begin(), v.end());       // Min value — O(N)

int idx = max_element(v.begin(), v.end()) - v.begin();  // Index of max
```

### Reverse, Rotate, Fill

```cpp
reverse(v.begin(), v.end());                      // O(N)
rotate(v.begin(), v.begin() + 2, v.end());        // Bring v[2] to front
fill(v.begin(), v.end(), 0);                      // O(N)
fill_n(v.begin(), 3, -1);                         // Fill first 3 with -1
```

### Count, Find, Accumulate

```cpp
count(v.begin(), v.end(), 3);               // Count 3s: O(N)
find(v.begin(), v.end(), 5);                // First 5: O(N) — returns iterator
accumulate(v.begin(), v.end(), 0);          // Sum: O(N)
accumulate(v.begin(), v.end(), 0LL);        // Long long sum — use 0LL!
accumulate(v.begin(), v.end(), 1, multiplies<int>()); // Product

count_if(v.begin(), v.end(), [](int x){ return x % 2 == 0; });
find_if(v.begin(), v.end(), [](int x){ return x > 3; });
```

### Permutations

```cpp
vector<int> v = {1, 2, 3};  // Must be sorted for all permutations!

do {
    for (int x : v) cout << x << " ";
} while (next_permutation(v.begin(), v.end()));
// Prints all 6 permutations in lexicographic order

prev_permutation(v.begin(), v.end());  // Previous permutation
```

### Unique & Set Operations

```cpp
// unique — removes consecutive duplicates
sort(v.begin(), v.end());               // Sort first!
v.erase(unique(v.begin(), v.end()), v.end());

// Set operations on sorted arrays
vector<int> a = {1,2,3,4}, b = {3,4,5,6}, res;
set_intersection(a.begin(), a.end(), b.begin(), b.end(), back_inserter(res)); // {3,4}
set_union(a.begin(), a.end(), b.begin(), b.end(), back_inserter(res));        // {1,2,3,4,5,6}
set_difference(a.begin(), a.end(), b.begin(), b.end(), back_inserter(res));   // {1,2}
```

### Built-in Functions

```cpp
__gcd(12, 8);               // 4 — GCD
// C++17: #include <numeric>; gcd(12, 8);

__builtin_popcount(x);      // Count set bits (int)
__builtin_popcountll(x);    // Count set bits (long long)
__builtin_clz(x);           // Count leading zeros
__builtin_ctz(x);           // Count trailing zeros
__builtin_parity(x);        // Parity of set bits (0 = even, 1 = odd)
```

---

## 21. Iterators

Objects that act like **pointers** to container elements.

### Types

```
Input Iterator     → Read forward only
Output Iterator    → Write forward only
Forward Iterator   → Read/write forward (forward_list)
Bidirectional      → Both directions (list, set, map)
Random Access      → O(1) jump to any position (vector, deque)
```

### Common Operations

```cpp
vector<int> v = {1, 2, 3, 4, 5};

auto it = v.begin();     // First element
auto end = v.end();      // Sentinel — one PAST last element

*it;                     // Dereference: 1
++it;                    // Move forward
--it;                    // Move backward (bidirectional+)
it + 2;                  // Jump ahead (random access only)
it - v.begin();          // Index (distance from beginning)

auto rit = v.rbegin();   // Reverse: points to last element
auto rend = v.rend();    // Reverse sentinel

advance(it, 3);          // Move by 3: O(N) for bidirectional
distance(v.begin(), it); // Count between iterators
```

### `auto` Keyword

```cpp
auto x = 10;                   // int
auto it = v.begin();           // vector<int>::iterator
auto& val = v[0];              // int& (reference)

for (auto x : v) { }          // By value
for (auto& x : v) { }         // By reference — can modify
for (const auto& x : v) { }   // By const ref — safe
```

### Iterator Invalidation — Critical!

```cpp
vector<int> v = {1, 2, 3, 4, 5};
auto it = v.begin() + 2;

v.push_back(6);          // ⚠️ If reallocation: it is INVALID
v.insert(v.begin(), 0);  // ⚠️ it is INVALID
v.erase(v.begin());      // ⚠️ it and all iterators after it are INVALID

// ✅ Safe erase while iterating
for (auto it = v.begin(); it != v.end(); ) {
    if (*it % 2 == 0)
        it = v.erase(it);   // erase returns next valid iterator
    else
        ++it;
}
```

---

# PART 4 — REFERENCE

---

## 22. Complexity Cheat Sheet

### Container Operations

| Operation | vector | list | deque | set/map | unordered |
|---|---|---|---|---|---|
| Access by index | O(1) | O(N) | O(1) | ❌ | ❌ |
| Push back | O(1)* | O(1) | O(1) | — | — |
| Push front | O(N) | O(1) | O(1) | — | — |
| Insert middle | O(N) | O(1)† | O(N) | O(log N) | O(1) avg |
| Erase by value | O(N) | O(N) | O(N) | O(log N) | O(1) avg |
| Search | O(N) | O(N) | O(N) | O(log N) | O(1) avg |
| Min/Max | O(N) | O(N) | O(N) | O(1)‡ | O(N) |

\* Amortized | † Given iterator | ‡ Via `begin()`/`rbegin()`

### Priority Queue (Heap)

| Operation | Complexity |
|---|---|
| push | O(log N) |
| pop | O(log N) |
| top | O(1) |
| Build from N elements | **O(N)** ← Not O(N log N)! |

### Algorithms

| Algorithm | Complexity | Needs Sorted? |
|---|---|---|
| sort | O(N log N) | No (sorts it) |
| binary_search | O(log N) | ✅ Yes |
| lower_bound / upper_bound | O(log N) | ✅ Yes |
| min/max_element | O(N) | No |
| find / count | O(N) | No |
| reverse / rotate | O(N) | No |
| accumulate | O(N) | No |
| next_permutation | O(N) | Start sorted |
| unique | O(N) | Sort first for dedup |

---

## 23. Choosing the Right Container

```
Need ordered, unique elements with fast lookup?
  → set

Need ordered key-value pairs?
  → map

Need sorted but allow duplicates?
  → multiset / multimap

Need fastest lookup, don't care about order?
  → unordered_set / unordered_map

Need dynamic array with O(1) index access?
  → vector

Need O(1) insertions at both ends + random access?
  → deque

Need O(1) insertions/deletions anywhere (given iterator)?
  → list

Need LIFO access?
  → stack

Need FIFO access?
  → queue

Need to always access the min or max?
  → priority_queue
```

---

## 24. Common Interview Patterns

### Pattern 1: Frequency Count

```cpp
unordered_map<int, int> freq;
for (int x : arr) freq[x]++;

// Most frequent
int maxFreq = 0;
for (auto& [val, cnt] : freq) maxFreq = max(maxFreq, cnt);
```

### Pattern 2: K-th Largest / Smallest (Min-Heap of size K)

```cpp
priority_queue<int, vector<int>, greater<int>> minPQ;
for (int x : arr) {
    minPQ.push(x);
    if ((int)minPQ.size() > k) minPQ.pop();
}
return minPQ.top();  // K-th largest
```

### Pattern 3: Sliding Window (Distinct Elements)

```cpp
map<int, int> window;
for (int i = 0; i < k; i++) window[arr[i]]++;
cout << window.size() << "\n";

for (int i = k; i < n; i++) {
    window[arr[i]]++;
    if (--window[arr[i - k]] == 0) window.erase(arr[i - k]);
    cout << window.size() << "\n";
}
```

### Pattern 4: Two-Sum / Pair with Given Sum

```cpp
unordered_set<int> seen;
for (int x : arr) {
    if (seen.count(target - x)) { /* found pair */ }
    seen.insert(x);
}
```

### Pattern 5: Prefix Sum + HashMap (Subarray Sum = K)

```cpp
unordered_map<int, int> prefixCount;
prefixCount[0] = 1;
int sum = 0, count = 0;
for (int x : arr) {
    sum += x;
    if (prefixCount.count(sum - k)) count += prefixCount[sum - k];
    prefixCount[sum]++;
}
```

### Pattern 6: Monotonic Stack (NGE / NSE)

```cpp
// Next Greater Element
vector<int> nextGreater(vector<int>& arr) {
    int n = arr.size();
    vector<int> result(n, -1);
    stack<int> st;  // stores indices
    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] < arr[i]) {
            result[st.top()] = arr[i];
            st.pop();
        }
        st.push(i);
    }
    return result;
}
```

### Pattern 7: BFS with Queue

```cpp
queue<int> q;
vector<bool> visited(n, false);
q.push(start);
visited[start] = true;
while (!q.empty()) {
    int node = q.front(); q.pop();
    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            visited[neighbor] = true;
            q.push(neighbor);
        }
    }
}
```

### Pattern 8: Coordinate Compression

```cpp
vector<int> sorted_vals = arr;
sort(sorted_vals.begin(), sorted_vals.end());
sorted_vals.erase(unique(sorted_vals.begin(), sorted_vals.end()), sorted_vals.end());

auto getRank = [&](int val) {
    return (int)(lower_bound(sorted_vals.begin(), sorted_vals.end(), val)
                 - sorted_vals.begin());
};
```

---

## 25. Common Mistakes & Pitfalls

### ❌ Mistake 1: `map[key]` silently creates entry

```cpp
map<string, int> m;
if (m["missing"] == 0) { }   // ⚠️ Creates "missing" with value 0!

// ✅ Correct
if (m.count("missing") == 0) { }
if (m.find("missing") == m.end()) { }
```

### ❌ Mistake 2: `multiset.erase(val)` removes ALL copies

```cpp
multiset<int> ms = {1, 2, 2, 2, 3};
ms.erase(2);             // ⚠️ Removes ALL three 2s!
ms.erase(ms.find(2));    // ✅ Removes exactly one 2
```

### ❌ Mistake 3: Using `std::lower_bound` on `set`/`map`

```cpp
set<int> s = {1, 2, 3, 4, 5};
auto it = lower_bound(s.begin(), s.end(), 3);  // ⚠️ O(N)!
auto it = s.lower_bound(3);                    // ✅ O(log N)
```

### ❌ Mistake 4: Integer overflow in `accumulate`

```cpp
vector<int> v = {1000000000, 1000000000};
int sum = accumulate(v.begin(), v.end(), 0);    // ⚠️ OVERFLOW!
long long sum = accumulate(v.begin(), v.end(), 0LL);  // ✅
```

### ❌ Mistake 5: Priority queue comparator direction confusion

```cpp
// "greater<int>" in priority_queue = MIN-heap
priority_queue<int, vector<int>, greater<int>> minPQ;  // ✅

// "greater<int>" in sort() = descending order
sort(v.begin(), v.end(), greater<int>());   // descending ✅
```

### ❌ Mistake 6: Iterator invalidation

```cpp
// ⚠️ WRONG — erase invalidates it
for (auto it = v.begin(); it != v.end(); ++it)
    if (*it == 3) v.erase(it);   // UB!

// ✅ Correct
for (auto it = v.begin(); it != v.end(); )
    if (*it == 3) it = v.erase(it);
    else ++it;
```

### ❌ Mistake 7: `lower_bound` on unsorted array

```cpp
auto it = lower_bound(v.begin(), v.end(), 3);  // ⚠️ UB if unsorted!
sort(v.begin(), v.end());
auto it = lower_bound(v.begin(), v.end(), 3);  // ✅
```

### ❌ Mistake 8: `next_permutation` on unsorted input

```cpp
// ⚠️ Won't generate ALL permutations — starts from current state
vector<int> v = {2, 1, 3};
do { ... } while (next_permutation(v.begin(), v.end()));

// ✅ Sort first to get all permutations
sort(v.begin(), v.end());
do { ... } while (next_permutation(v.begin(), v.end()));
```

### Mistake 9: Passing large containers by value

```cpp
void solve(vector<int> v)    // ⚠️ O(N) copy — kills performance
void solve(vector<int>& v)   // ✅ O(1) reference
void solve(const vector<int>& v)  // ✅ O(1) const reference (read-only)
```

### Mistake 10: Using `int` when `long long` is needed

```cpp
int a = 1e9, b = 1e9;
int c = a * b;        // ⚠️ Overflow — result is ~10^18
long long c = 1LL * a * b;  // ✅
```

---

## 26. Quick Revision Cheat Sheet

### Container Headers

```cpp
#include <vector>
#include <list>
#include <deque>
#include <stack>
#include <queue>            // queue and priority_queue
#include <set>              // set and multiset
#include <map>              // map and multimap
#include <unordered_set>
#include <unordered_map>
#include <utility>          // pair
#include <tuple>
#include <algorithm>        // sort, binary_search, etc.
#include <numeric>          // accumulate, gcd (C++17)
```

### One-Liner Container Summary

```
vector      → dynamic array, O(1) access, O(1) amortized push_back
list        → doubly linked list, O(1) insert/delete at iterator
deque       → double-ended, O(1) push/pop both ends, O(1) access
stack       → LIFO: push/pop/top, all O(1)
queue       → FIFO: push/pop/front/back, all O(1)
pq (max)    → max at top, push O(log N), top O(1), pop O(log N)
pq (min)    → use greater<T> as third template argument
set         → sorted unique, O(log N) all ops, use s.lower_bound()
multiset    → sorted with dups, erase(find(x)) for single removal
map         → sorted key-value O(log N), never use [] for checking
unord_map   → hash key-value O(1) avg, ~4x faster than map
```

### Interview Quick-Fire Answers

| Q | A |
|---|---|
| How does vector grow? | Doubles capacity (2x), O(1) amortized push_back |
| Why is set O(log N)? | Backed by Red-Black Tree (Self-Balancing BST) |
| Why is unordered_map O(1)? | Hash Table with O(1) avg bucket access |
| Worst case of unordered_map? | O(N) — all keys hash to same bucket |
| Build heap from N elements? | **O(N)** — not O(N log N) |
| What does sort() use internally? | IntroSort = QuickSort + HeapSort + InsertionSort |
| What does stable_sort() use? | Merge Sort |
| lower_bound returns? | Iterator to first element **≥** val |
| upper_bound returns? | Iterator to first element **>** val |
| Delete one from multiset? | `ms.erase(ms.find(val))` |
| How to make min-heap? | `priority_queue<T, vector<T>, greater<T>>` |
| safe map existence check? | `m.count(key)` or `m.find(key) != m.end()` |
| lower_bound on set: which version? | Member function `s.lower_bound()` — O(log N) |

### Constraint → Algorithm Lookup

| N | Target Complexity | Typical Approach |
|---|---|---|
| ≤ 10 | O(N!) | Brute force, permutations |
| ≤ 20 | O(2^N) | Bitmask DP, subsets |
| ≤ 1,000 | O(N²) | Nested loops |
| ≤ 10^5 | O(N log N) | Sort, BST, heaps |
| ≤ 10^6 | O(N) | Hashing, two pointers, sliding window |
| ≤ 10^18 | O(log N) | Binary search, math |

### Memory Complexity

| Container | Space |
|---|---|
| vector, deque, list | O(N) |
| set, map (RB-tree) | O(N) — ~3–4 pointers per node |
| unordered_set/map | O(N) — hash table with load factor |
| priority_queue | O(N) |
| stack, queue | O(N) — adaptor overhead only |

---

