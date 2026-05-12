# Striver A2Z DSA Sheet — Important Pattern Problems Master Notes

---

# Why Pattern Problems Matter

Most beginners think pattern problems are useless because they only print stars or numbers.

That is completely wrong.

Pattern problems are not about printing.
They are about building the mental framework required for:

* nested loop mastery
* visualization skills
* index mathematics
* symmetry recognition
* matrix traversal intuition
* simulation problem solving
* decomposition of complex logic
* debugging discipline
* converting observations into formulas

These skills directly transfer into:

* arrays
* matrices
* graphs
* dynamic programming
* recursion
* geometry problems
* implementation-heavy interview questions

The people who become strong in DSA are usually good at visualizing structures.
Pattern problems are the first training ground for that skill.

---

# The Real Goal While Solving Patterns

DO NOT memorize patterns.

Instead, train yourself to answer:

1. What changes row by row?
2. What changes column by column?
3. How many spaces are needed?
4. How many symbols are needed?
5. Is the pattern symmetric?
6. Is the pattern growing or shrinking?
7. Can the problem be broken into smaller sections?
8. Is there a mathematical formula hidden inside?

If you learn to derive patterns instead of memorizing them, later DSA becomes dramatically easier.

---

# Universal Pattern Solving Framework

Before coding ANY pattern:

## Step 1 — Count Rows

Outer loop almost always represents rows.

```cpp
for(int i = 0; i < n; i++)
```

---

## Step 2 — Analyze Each Row

Ask:

* how many spaces?
* how many stars?
* what values?
* increasing or decreasing?

---

## Step 3 — Build Formula

Examples:

| Quantity              | Formula   |
| --------------------- | --------- |
| Increasing stars      | i + 1     |
| Decreasing stars      | n - i     |
| Pyramid stars         | 2*i + 1   |
| Pyramid spaces        | n - i - 1 |
| Reverse pyramid stars | 2*(n-i)-1 |

---

## Step 4 — Divide Into Sections

Complex patterns usually contain:

* left part
* middle spaces
* right part

OR

* upper half
* lower half

Breaking patterns into sections is a huge programming skill.

---

## Step 5 — Dry Run on Paper

Strong programmers mentally simulate loops.

Always test:

* first row
* middle row
* last row

---

# Complexity Analysis of Pattern Problems

Most pattern problems:

* Time Complexity: O(N²)
* Space Complexity: O(1)

Because nested loops dominate.

This matters because matrix problems later also commonly become O(N²).

---

# IMPORTANT PATTERN 1 — Right Triangle Star Pattern

## Pattern

```text
*
**
***
****
*****
```

---

# Observation

For row i:

* number of stars = i + 1

---

# Intuition

Each row adds one extra star.

This is the first exposure to:

* nested loops
* row-column dependency

---

# Approach

Outer loop controls rows.
Inner loop prints stars.

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 0; i < n; i++) {
        for(int j = 0; j <= i; j++) {
            cout << "*";
        }
        cout << endl;
    }
}
```

---

# Complexity

* Time: O(N²)
* Space: O(1)

---

# Interview Relevance

This pattern teaches:

* loop boundaries
* visualization
* incremental growth

Those concepts appear everywhere later.

---

# Common Mistakes

## Wrong Loop Bound

```cpp
j < i
```

instead of:

```cpp
j <= i
```

This misses one star.

---

# IMPORTANT PATTERN 2 — Number Triangle

## Pattern

```text
1
12
123
1234
12345
```

---

# Observation

For row i:

* numbers go from 1 to i+1

---

# Core Learning

Now output value differs from loop index logic.

You begin separating:

* loop control
* printed value

This distinction becomes critical in advanced DSA.

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 1; i <= n; i++) {
        for(int j = 1; j <= i; j++) {
            cout << j;
        }
        cout << endl;
    }
}
```

---

# Transferable Skill

Useful later in:

* matrix filling
* simulation problems
* coordinate generation

---

# IMPORTANT PATTERN 3 — Repeated Number Triangle

## Pattern

```text
1
22
333
4444
55555
```

---

# Observation

For row i:

* print number i exactly i times

---

# Key Concept

The row number itself becomes the printed value.

This teaches:

* row-state dependency
* fixed repeated outputs

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 1; i <= n; i++) {
        for(int j = 1; j <= i; j++) {
            cout << i;
        }
        cout << endl;
    }
}
```

---

# IMPORTANT PATTERN 4 — Reverse Star Triangle

## Pattern

```text
*****
****
***
**
*
```

---

# Observation

For row i:

* stars = n - i

---

# Core Learning

Shrinking loops.

This is important because many DSA algorithms reduce search space gradually.

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 0; i < n; i++) {
        for(int j = 0; j < n - i; j++) {
            cout << "*";
        }
        cout << endl;
    }
}
```

---

# IMPORTANT PATTERN 5 — Pyramid Pattern

## Pattern

```text
    *
   ***
  *****
 *******
*********
```

---

# This Is One Of The Most Important Patterns

Why?

Because this introduces:

* symmetry
* spaces + stars together
* center alignment
* mathematical relationships

---

# Observation

For row i:

| Quantity | Formula |
| -------- | ------- |
| Spaces   | n-i-1   |
| Stars    | 2*i+1   |

---

# Mental Model

Imagine expanding from center.

Each row:

* loses one space from left
* gains two stars

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 0; i < n; i++) {

        for(int j = 0; j < n - i - 1; j++) {
            cout << " ";
        }

        for(int j = 0; j < 2*i + 1; j++) {
            cout << "*";
        }

        cout << endl;
    }
}
```

---

# Real DSA Relevance

This thinking appears later in:

* matrix diagonals
* geometry
* image transformations
* coordinate systems
* tree visualization

---

# Common Mistake

Using:

```cpp
2*i
```

instead of:

```cpp
2*i + 1
```

That breaks symmetry.

---

# IMPORTANT PATTERN 6 — Inverted Pyramid

## Pattern

```text
*********
 *******
  *****
   ***
    *
```

---

# Observation

| Quantity | Formula   |
| -------- | --------- |
| Spaces   | i         |
| Stars    | 2*(n-i)-1 |

---

# Key Skill

Reverse-engineering logic.

Interviewers love checking whether you can invert logic cleanly.

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 0; i < n; i++) {

        for(int j = 0; j < i; j++) {
            cout << " ";
        }

        for(int j = 0; j < 2*(n-i)-1; j++) {
            cout << "*";
        }

        cout << endl;
    }
}
```

---

# IMPORTANT PATTERN 7 — Diamond Pattern

## Pattern

```text
    *
   ***
  *****
 *******
*********
 *******
  *****
   ***
    *
```

---

# Why This Pattern Matters

This teaches decomposition.

The pattern is actually:

```cpp
upperPyramid();
lowerPyramid();
```

This mindset is massively important in software engineering.

Complex systems are built from smaller reusable components.

---

# Approach

1. Print normal pyramid
2. Print inverted pyramid

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 0; i < n; i++) {

        for(int j = 0; j < n - i - 1; j++) {
            cout << " ";
        }

        for(int j = 0; j < 2*i + 1; j++) {
            cout << "*";
        }

        cout << endl;
    }

    for(int i = 0; i < n; i++) {

        for(int j = 0; j < i; j++) {
            cout << " ";
        }

        for(int j = 0; j < 2*(n-i)-1; j++) {
            cout << "*";
        }

        cout << endl;
    }
}
```

---

# IMPORTANT PATTERN 8 — Hollow Square

## Pattern

```text
*****
*   *
*   *
*   *
*****
```

---

# Core Concept

Print star only on boundaries.

---

# Boundary Conditions

A cell belongs to boundary if:

* first row
* last row
* first column
* last column

---

# This Pattern Is Extremely Valuable

Because boundary checking appears everywhere:

* matrix traversal
* BFS
* DFS
* flood fill
* graph problems
* simulation

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 0; i < n; i++) {
        for(int j = 0; j < n; j++) {

            if(i == 0 || i == n-1 || j == 0 || j == n-1)
                cout << "*";
            else
                cout << " ";
        }

        cout << endl;
    }
}
```

---

# IMPORTANT PATTERN 9 — Binary Number Triangle

## Pattern

```text
1
01
101
0101
10101
```

---

# Hidden Concept

Parity.

---

# Observation

Value depends on:

```cpp
(i + j) % 2
```

---

# Why This Is Important

Parity logic appears everywhere:

* chessboard problems
* graph coloring
* alternating sequences
* bit manipulation
* game theory

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 0; i < n; i++) {
        for(int j = 0; j <= i; j++) {
            cout << (i + j) % 2;
        }

        cout << endl;
    }
}
```

---

# IMPORTANT PATTERN 10 — Palindrome Pyramid

## Pattern

```text
1      1
12    21
123  321
12344321
```

---

# This Is One Of The Best Thinking Patterns

Why?

Because it combines:

* left increasing sequence
* middle spaces
* right decreasing sequence

Multiple regions exist in one row.

---

# Core Learning

Complex patterns are combinations of simpler structures.

This is exactly how advanced DSA works.

---

# Observation

For row i:

1. print increasing numbers
2. print spaces
3. print decreasing numbers

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 4;

    for(int i = 1; i <= n; i++) {

        for(int j = 1; j <= i; j++) {
            cout << j;
        }

        for(int j = 1; j <= 2*(n-i); j++) {
            cout << " ";
        }

        for(int j = i; j >= 1; j--) {
            cout << j;
        }

        cout << endl;
    }
}
```

---

# Transferable Skills

Useful later in:

* two pointers
* string manipulation
* bidirectional traversal
* deque problems

---

# IMPORTANT PATTERN 11 — Continuous Number Triangle

## Pattern

```text
1
2 3
4 5 6
7 8 9 10
```

---

# Important Concept

State persistence.

Unlike previous patterns, values continue across rows.

---

# Why This Matters

This teaches maintaining information across iterations.

That idea becomes fundamental later in:

* prefix sums
* DP
* streaming problems
* cumulative computations

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 4;
    int num = 1;

    for(int i = 1; i <= n; i++) {

        for(int j = 1; j <= i; j++) {
            cout << num << " ";
            num++;
        }

        cout << endl;
    }
}
```

---

# IMPORTANT PATTERN 12 — Character Triangle

## Pattern

```text
A
AB
ABC
ABCD
ABCDE
```

---

# Core Concept

ASCII arithmetic.

---

# Key Learning

Characters are internally integers.

```cpp
'A' + 1 = 'B'
```

Understanding this becomes important later in:

* hashing
* string algorithms
* parsing
* encoding

---

# Code

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n = 5;

    for(int i = 0; i < n; i++) {

        for(char ch = 'A'; ch <= 'A' + i; ch++) {
            cout << ch;
        }

        cout << endl;
    }
}
```

---

# IMPORTANT PATTERN 13 — Butterfly Pattern

## Pattern

```text
*      *
**    **
***  ***
********
***  ***
**    **
*      *
```

---

# One Of The Most Important Patterns

This combines:

* increasing stars
* decreasing spaces
* symmetry
* upper/lower halves

---

# Core Observation

Upper Half:

| Quantity    | Formula |
| ----------- | ------- |
| Left stars  | i       |
| Spaces      | 2*(n-i) |
| Right stars | i       |

Lower half mirrors this.

---

# Why This Pattern Is Valuable

This develops:

* decomposition ability
* visualization strength
* coordinate reasoning

These are advanced implementation skills.

---

# IMPORTANT PATTERN 14 — Concentric Number Pattern

## Pattern

```text
4 4 4 4 4 4 4
4 3 3 3 3 3 4
4 3 2 2 2 3 4
4 3 2 1 2 3 4
4 3 2 2 2 3 4
4 3 3 3 3 3 4
4 4 4 4 4 4 4
```

---

# One Of The Most Important Matrix Thinking Problems

---

# Core Insight

Each cell value depends on distance from boundary.

---

# Formula

For cell (i,j):

```cpp
min(
    min(i, j),
    min(n-1-i, n-1-j)
)
```

---

# Why This Pattern Matters

This develops:

* coordinate thinking
* geometry intuition
* layered matrix reasoning

Those skills later appear in:

* spiral traversal
* image processing
* simulation
* matrix rotations
* geometry problems

---

# Common Beginner Mistakes In Pattern Problems

## 1. Memorizing Instead of Deriving

Bad approach.

Always derive formulas.

---

## 2. Not Drawing Rows/Columns

Visualization matters.

Draw the grid.

---

## 3. Ignoring Symmetry

Most hard patterns are symmetric.

Exploit symmetry.

---

## 4. Mixing Spaces And Stars

Treat them separately.

---

## 5. Poor Dry Runs

Always manually test:

* first row
* middle row
* last row

---

# Real Interview Relevance

Pattern problems themselves are rarely asked.

But the thinking absolutely matters.

Strong candidates usually:

* visualize better
* debug faster
* derive formulas quicker
* handle matrix problems more confidently

because they trained these skills early.

---

# Final Important Takeaway

Pattern problems are not important because of stars.

They are important because they teach:

* structured thinking
* loop control
* decomposition
* symmetry
* visualization
* formula derivation
* implementation discipline

Those skills compound throughout your entire DSA journey.

If you master the thinking behind patterns, later topics feel much less overwhelming.
