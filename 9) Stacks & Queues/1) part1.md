**Topic:** Stack and Queue Implementations, Applications, and Expression Notation Conversions
**Patterns:** LIFO/FIFO Structures, Array/LL Implementation, Cross-Simulation, Parentheses Matching, Monotonic Min Tracking, Infix/Postfix/Prefix Conversion

**Resources:**
[GFG Stack Array](https://www.geeksforgeeks.org/problems/implement-stack-using-array/1) |
[GFG Queue Array](https://www.geeksforgeeks.org/problems/implement-queue-using-array/1) |
[LC 225 Stack via Queues](https://leetcode.com/problems/implement-stack-using-queues/) |
[LC 232 Queue via Stacks](https://leetcode.com/problems/implement-queue-using-stacks/) |
[GFG Stack LL](https://www.geeksforgeeks.org/problems/implement-stack-using-linked-list/1) |
[GFG Queue LL](https://www.geeksforgeeks.org/problems/implement-queue-using-linked-list/1) |
[LC 20 Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) |
[LC 155 Min Stack](https://leetcode.com/problems/min-stack/) |
[Striver Lecture 1](https://youtu.be/tqQ5fTamIN4) |
[Striver Lecture 2](https://youtu.be/xwjS0iZhw4I) |
[Striver Lecture 3](https://youtu.be/NdDIaH91P0g) |
[Striver Lecture 4](https://youtu.be/4pIc9UBHJtk)

---

## Table of Contents

1. [Core Mental Model — Stack and Queue as Access Discipline](#1-core-mental-model--stack-and-queue-as-access-discipline)
2. [Implementation 1 — Stack using Array](#2-implementation-1--stack-using-array)
3. [Implementation 2 — Queue using Array (Circular)](#3-implementation-2--queue-using-array-circular)
4. [Implementation 3 — Stack using Linked List](#4-implementation-3--stack-using-linked-list)
5. [Implementation 4 — Queue using Linked List](#5-implementation-4--queue-using-linked-list)
6. [Implementation 5 — Stack using Queues (LC 225)](#6-implementation-5--stack-using-queues-lc-225)
7. [Implementation 6 — Queue using Stacks (LC 232)](#7-implementation-6--queue-using-stacks-lc-232)
8. [Application 1 — Valid Parentheses (LC 20)](#8-application-1--valid-parentheses-lc-20)
9. [Application 2 — Min Stack (LC 155)](#9-application-2--min-stack-lc-155)
10. [Expression Notation — First Principles](#10-expression-notation--first-principles)
11. [Conversion 1 — Infix to Postfix](#11-conversion-1--infix-to-postfix)
12. [Conversion 2 — Infix to Prefix](#12-conversion-2--infix-to-prefix)
13. [Conversion 3 — Postfix to Infix](#13-conversion-3--postfix-to-infix)
14. [Conversion 4 — Postfix to Prefix](#14-conversion-4--postfix-to-prefix)
15. [Conversion 5 — Prefix to Infix](#15-conversion-5--prefix-to-infix)
16. [Conversion 6 — Prefix to Postfix](#16-conversion-6--prefix-to-postfix)
17. [The Unifying Map — All Six Conversions](#17-the-unifying-map--all-six-conversions)
18. [Why the Two-Stack Queue is Amortized O(1) — Formal Proof](#18-why-the-two-stack-queue-is-amortized-o1--formal-proof)
19. [Edge Cases Master Table](#19-edge-cases-master-table)
20. [Common Mistakes and Pitfalls](#20-common-mistakes-and-pitfalls)
21. [Complexity Cheat Sheet](#21-complexity-cheat-sheet)
22. [Interview Reference — Patterns and Applications](#22-interview-reference--patterns-and-applications)
23. [Quick Revision Cheat Sheet](#23-quick-revision-cheat-sheet)

---

## 1. Core Mental Model — Stack and Queue as Access Discipline

A stack and a queue are not fundamentally different data structures — they are different **access disciplines** applied to the same underlying sequence. Both store elements; what differs is which element is accessible at any moment.

- **Stack (LIFO — Last In First Out):** Only the most recently added element is accessible. Think of a call stack: the most recent function call runs now; it must complete before the caller resumes.
- **Queue (FIFO — First In First Out):** Only the least recently added element is accessible. Think of a print queue: jobs are dispatched in arrival order.

This distinction drives algorithm selection: when processing order must mirror arrival order, use a queue (BFS). When processing order must mirror the reverse of arrival, use a stack (DFS, expression parsing, undo).

**Internal implementation tradeoffs:**

| Backing store | Push/Enqueue | Pop/Dequeue | Space | Overflow risk |
|---|---|---|---|---|
| Static array | O(1) | O(1) | Fixed, pre-allocated | Yes |
| Dynamic array (vector) | O(1) amortized | O(1) | Grows on demand | No |
| Linked list | O(1) | O(1) | Per-node pointer overhead | No |

---

## 2. Implementation 1 — Stack using Array

**Problem:** Implement a stack supporting `push(x)`, `pop()`, and `peek()` using a fixed-size array. `pop()` and `peek()` on empty stack return `-1`.

**State invariant:** `top` is the index of the topmost element. `top == -1` means empty.

```cpp
class MyStack {
    int arr[1000];
    int top;
public:
    MyStack() { top = -1; }

    void push(int x) {
        if (top == 999) return;       // full; capacity guaranteed in contest
        arr[++top] = x;               // pre-increment: top points to current top
    }

    int pop() {
        if (top == -1) return -1;
        return arr[top--];            // post-decrement: read then decrement
    }

    int peek() {
        return (top == -1) ? -1 : arr[top];
    }

    bool isEmpty() { return top == -1; }
};
```

**Dry run** — `push(3)`, `push(7)`, `peek()`, `pop()`, `pop()`, `pop()`:

```
Initial:  top=-1

push(3):  arr[0]=3, top=0
push(7):  arr[1]=7, top=1
peek():   return arr[1]=7   (top unchanged)
pop():    return arr[1]=7, top=0
pop():    return arr[0]=3, top=-1
pop():    top==-1 → return -1
```

Time: O(1) all operations. Space: O(N) static.

---

## 3. Implementation 2 — Queue using Array (Circular)

**Problem:** Implement a queue supporting `push(x)` (enqueue) and `pop()` (dequeue). `pop()` on empty returns `-1`.

**Why circular array:** With a naive linear array, `rear` reaches the end even when front-portion positions are vacated by pops — wasting space. Modulo arithmetic wraps `rear` and `front` around, reusing emptied slots.

```cpp
class MyQueue {
    int arr[100005];
    int front, rear, sz;
public:
    MyQueue() : front(0), rear(0), sz(0) {}

    void push(int x) {
        if (sz == 100005) return;
        arr[rear] = x;
        rear = (rear + 1) % 100005;   // wrap around
        sz++;
    }

    int pop() {
        if (sz == 0) return -1;
        int val = arr[front];
        front = (front + 1) % 100005; // wrap around
        sz--;
        return val;
    }

    bool isEmpty() { return sz == 0; }
    int Front() { return sz == 0 ? -1 : arr[front]; }
};
```

**Dry run** — capacity=4: `push(1)`, `push(2)`, `pop()`, `push(3)`, `push(4)`, `push(5)`:

```
push(1): arr[0]=1, rear=1, sz=1
push(2): arr[1]=2, rear=2, sz=2
pop():   return arr[0]=1, front=1, sz=1
push(3): arr[2]=3, rear=3, sz=2
push(4): arr[3]=4, rear=0 (wrap), sz=3
push(5): arr[0]=5, rear=1 (wrap), sz=4   ← position 0 reused ✓
```

Time: O(1) all operations. Space: O(N).

---

## 4. Implementation 3 — Stack using Linked List

**Problem:** Implement a stack using a singly linked list. Return `-1` on pop of empty stack.

**Key insight:** The top of the stack maps to the **head** of the linked list. Push and pop both operate at the head — O(1) with no shifting. The list grows and shrinks dynamically, so there is no overflow.

```cpp
struct Node {
    int data;
    Node* next;
    Node(int val) : data(val), next(nullptr) {}
};

class MyStack {
    Node* top;
public:
    MyStack() : top(nullptr) {}

    void push(int x) {
        Node* node = new Node(x);
        node->next = top;   // new node points to current top
        top = node;         // new node becomes the new top
    }

    int pop() {
        if (!top) return -1;
        int val = top->data;
        Node* tmp = top;
        top = top->next;
        delete tmp;
        return val;
    }

    int peek() { return top ? top->data : -1; }
    bool isEmpty() { return top == nullptr; }

    ~MyStack() {
        while (top) { Node* t = top; top = top->next; delete t; }
    }
};
```

**Visual** after `push(3)`, `push(7)`, `push(1)`:

```
top → [1|→] → [7|→] → [3|null]
```

Pop returns 1; top advances to 7. Chain shortens from the front.

Time: O(1) push and pop. Space: O(n) with per-node pointer overhead.

---

## 5. Implementation 4 — Queue using Linked List

**Problem:** Implement a queue using a linked list. Enqueue at rear, dequeue from front, both O(1).

**Key insight:** Maintain both `front` and `rear` pointers. Appending to `rear` is O(1); removing from `front` is O(1). No wasted space, no capacity limit.

```cpp
struct Node {
    int data;
    Node* next;
    Node(int val) : data(val), next(nullptr) {}
};

class MyQueue {
    Node* front;
    Node* rear;
public:
    MyQueue() : front(nullptr), rear(nullptr) {}

    void push(int x) {
        Node* node = new Node(x);
        if (!rear) { front = rear = node; return; }
        rear->next = node;
        rear = node;
    }

    int pop() {
        if (!front) return -1;
        int val = front->data;
        Node* tmp = front;
        front = front->next;
        if (!front) rear = nullptr;   // queue became empty; must nullify rear
        delete tmp;
        return val;
    }

    bool isEmpty() { return front == nullptr; }
    int Front() { return front ? front->data : -1; }
};
```

**Dry run** — `push(1)`, `push(2)`, `pop()`, `push(3)`, `pop()`:

```
push(1): front→[1|null]←rear
push(2): front→[1|→]→[2|null]←rear
pop():   return 1, front→[2|null]←rear
push(3): front→[2|→]→[3|null]←rear
pop():   return 2, front→[3|null]←rear
```

Time: O(1) all operations. Space: O(n).

---

## 6. Implementation 5 — Stack using Queues (LC 225)

**Problem:** Implement a LIFO stack (`push`, `top`, `pop`, `empty`) using only queue operations (`push_back`, `front`, `pop_front`, `empty`).
**Link:** [LC 225](https://leetcode.com/problems/implement-stack-using-queues/)
**Constraints:** At most 100 calls; all `pop`/`top` calls are valid.

### Approach 1 — Push O(1), Pop O(n): Two Queues

Push to `q1` (O(1)). On pop or top, move `n-1` elements from `q1` to `q2`; the remaining element is the top; swap queues.

```cpp
class MyStack {
    queue<int> q1, q2;
public:
    void push(int x) { q1.push(x); }

    int pop() {
        while (q1.size() > 1) { q2.push(q1.front()); q1.pop(); }
        int val = q1.front(); q1.pop();
        swap(q1, q2);
        return val;
    }

    int top() {
        while (q1.size() > 1) { q2.push(q1.front()); q1.pop(); }
        int val = q1.front();
        q2.push(val);   // keep element for subsequent operations
        q1.pop();
        swap(q1, q2);
        return val;
    }

    bool empty() { return q1.empty(); }
};
```

### Approach 2 — Single Queue, Push O(n): Rotation (Optimal)

Push `x` to the back of the queue. Then rotate `size - 1` times, cycling all older elements behind `x`. Now `x` is at the front.

```cpp
class MyStack {
    queue<int> q;
public:
    void push(int x) {
        q.push(x);
        int sz = q.size();
        for (int i = 0; i < sz - 1; i++) {
            q.push(q.front());
            q.pop();
        }
    }

    int pop()  { int v = q.front(); q.pop(); return v; }
    int top()  { return q.front(); }
    bool empty(){ return q.empty(); }
};
```

**Dry run** — `push(1)`, `push(2)`, `push(3)`, `top()`, `pop()`:

```
push(1): q=[1]     (sz=1, 0 rotations)
push(2): q=[1,2] → rotate 1: q=[2,1]
push(3): q=[2,1,3] → rotate 2: q=[1,3,2] → q=[3,2,1]
top():   q.front()=3  ✓
pop():   return 3, q=[2,1]
```

Push: O(n). Pop/top/empty: O(1). Space: O(n).

---

## 7. Implementation 6 — Queue using Stacks (LC 232)

**Problem:** Implement a FIFO queue (`push`, `peek`, `pop`, `empty`) using only two stacks. Achieve amortized O(1) per operation.
**Link:** [LC 232](https://leetcode.com/problems/implement-queue-using-stacks/)

### Approach 1 — Push O(n): Make Push Costly

On each push, reverse everything from `s1` to `s2`, push to `s1`, reverse back. Bottom of `s1` is always the queue front.

Push: O(n). Pop/peek: O(1).

### Approach 2 — Lazy Transfer: Amortized O(1) (Optimal)

**Intuition:** Use `s_in` (inbox) for pushing and `s_out` (outbox) for popping. Transfer from `s_in` to `s_out` **only when `s_out` is empty**. This defers the expensive reversal until it is actually needed, and each element is moved at most once.

```cpp
class MyQueue {
    stack<int> s_in, s_out;

    void transfer() {
        if (s_out.empty())
            while (!s_in.empty()) { s_out.push(s_in.top()); s_in.pop(); }
    }
public:
    void push(int x) { s_in.push(x); }         // O(1)

    int pop() {
        transfer();
        int val = s_out.top(); s_out.pop();
        return val;
    }

    int peek() { transfer(); return s_out.top(); }
    bool empty() { return s_in.empty() && s_out.empty(); }
};
```

**Dry run** — `push(1)`, `push(2)`, `push(3)`, `pop()`, `push(4)`, `pop()`, `pop()`:

```
push(1): s_in=[1],     s_out=[]
push(2): s_in=[1,2],   s_out=[]
push(3): s_in=[1,2,3], s_out=[]

pop():   s_out empty → transfer: s_out=[3,2,1], s_in=[]
         pop s_out.top()=1 → s_out=[3,2]   return 1 ✓

push(4): s_in=[4],     s_out=[3,2]

pop():   s_out NOT empty → no transfer
         pop s_out.top()=2 → s_out=[3]     return 2 ✓

pop():   s_out NOT empty → no transfer
         pop s_out.top()=3 → s_out=[]      return 3 ✓
```

Each element is pushed to `s_in` once, moved to `s_out` once, popped from `s_out` once — 3 O(1) operations per element regardless of the call sequence. All operations are O(1) amortized, O(n) worst case for a single pop.

---

## 8. Application 1 — Valid Parentheses (LC 20)

**Problem:** Given string `s` containing only `(`, `)`, `{`, `}`, `[`, `]`, return `true` if every open bracket is closed by the same type in correct order.
**Link:** [LC 20](https://leetcode.com/problems/valid-parentheses/)
**Constraints:** `1 <= s.length <= 10^4`.

**Examples:** `"()"` → true, `"([)]"` → false, `"{[]}"` → true, `"((("` → false.

**Intuition:** An open bracket creates an obligation to be matched by a close bracket later. The most recent unmatched open bracket must be matched first — LIFO. Use a stack: push opens, pop and verify on closes.

```cpp
bool isValid(string s) {
    stack<char> st;
    unordered_map<char,char> match = {{')', '('}, {'}', '{'}, {']', '['}};
    for (char c : s) {
        if (match.count(c)) {                    // closing bracket
            if (st.empty() || st.top() != match[c]) return false;
            st.pop();
        } else {
            st.push(c);                          // opening bracket
        }
    }
    return st.empty();   // all open brackets must have been matched
}
```

**Dry run** — `s = "{[]}"`:

```
c='{': push → st=[{]
c='[': push → st=[{, []
c=']': match[']']='[', top='[' ✓, pop → st=[{]
c='}': match['}]='{', top='{' ✓, pop → st=[]
st.empty()=true → return true  ✓
```

**Dry run** — `s = "([)]"`:

```
c='(': push → st=[(]
c='[': push → st=[(, []
c=')': match[')']='(', top='[' ≠ '(' → return false  ✓
```

**Why `return st.empty()` and not `return true`:** After processing all characters, any unclosed open brackets remain on the stack. `"((("` has no close brackets — the loop exits without error, but the stack is non-empty. Returning `st.empty()` catches this case.

Time: O(n). Space: O(n) — worst case all opening brackets.

---

## 9. Application 2 — Min Stack (LC 155)

**Problem:** Implement a stack supporting `push(val)`, `pop()`, `top()`, and `getMin()` all in O(1) time. `getMin()` returns the current minimum.
**Link:** [LC 155](https://leetcode.com/problems/min-stack/)
**Constraints:** `-2^31 <= val <= 2^31 - 1`, at most `3 * 10^4` calls.

**The challenge:** After a pop, the minimum may change. We need to recover the previous minimum in O(1). A naive linear scan is O(n).

### Approach 1 — Auxiliary Min Stack (Cleaner)

Maintain a second stack `minSt` that mirrors the main stack. `minSt.top()` is always the current minimum. On push: push `min(val, minSt.top())` to `minSt`. On pop: pop both stacks.

```cpp
class MinStack {
    stack<int> st, minSt;
public:
    void push(int val) {
        st.push(val);
        int newMin = minSt.empty() ? val : min(val, minSt.top());
        minSt.push(newMin);
    }

    void pop() { st.pop(); minSt.pop(); }
    int top()   { return st.top(); }
    int getMin(){ return minSt.top(); }
};
```

**Dry run** — `push(5)`, `push(3)`, `push(7)`, `push(2)`, `getMin()`, `pop()`, `getMin()`:

```
push(5): st=[5],       minSt=[5]
push(3): st=[5,3],     minSt=[5,3]    (min(3,5)=3)
push(7): st=[5,3,7],   minSt=[5,3,3]  (min(7,3)=3)
push(2): st=[5,3,7,2], minSt=[5,3,3,2](min(2,3)=2)
getMin(): return minSt.top()=2 ✓
pop():   st=[5,3,7],   minSt=[5,3,3]
getMin(): return minSt.top()=3 ✓
```

Time: O(1) all operations. Space: O(n) — two stacks.

### Approach 2 — Single Stack with Encoded Minimum (Space-Optimized)

**Intuition:** When pushing a value smaller than `curMin`, store an encoded value `2*val - curMin` instead of `val`, and update `curMin = val`. The encoded value is always less than `curMin`, which acts as a sentinel: if `st.top() < curMin`, we know the top holds an encoded value. On pop, decode the previous minimum: `oldMin = 2 * curMin - encodedVal`.

```cpp
class MinStack {
    stack<long long> st;
    long long curMin;
public:
    void push(int val) {
        if (st.empty()) {
            st.push((long long)val);
            curMin = val;
        } else if (val >= curMin) {
            st.push((long long)val);
        } else {
            st.push(2LL * val - curMin);   // encoded; always < curMin
            curMin = val;
        }
    }

    void pop() {
        long long top = st.top(); st.pop();
        if (top < curMin)
            curMin = 2LL * curMin - top;   // recover previous minimum
    }

    int top() {
        long long t = st.top();
        return (int)(t < curMin ? curMin : t);   // encoded values appear as curMin
    }

    int getMin() { return (int)curMin; }
};
```

**Why `long long`:** `2 * val - curMin` can exceed `int` range when values are near `INT_MIN` or `INT_MAX`.

**Dry run** — `push(3)`, `push(5)`, `push(2)`, `top()`, `pop()`, `getMin()`:

```
push(3): st=[3], curMin=3
push(5): 5>=3 → st=[3,5], curMin=3
push(2): 2<3 → encoded=2*2-3=1; st=[3,5,1], curMin=2
top():   st.top()=1 < curMin=2 → return curMin=2 ✓
pop():   1 < curMin=2 → oldMin=2*2-1=3; curMin=3; st=[3,5]
getMin(): curMin=3 ✓
```

Time: O(1) all operations. Space: O(n) one stack — half the constant factor of Approach 1.

---

## 10. Expression Notation — First Principles

An expression combines operands (`A`, `B`, `3`) with binary operators (`+`, `-`, `*`, `/`, `^`). The three notations differ in where the operator is placed:

| Notation | Operator position | Example | Evaluation |
|---|---|---|---|
| **Infix** | Between operands | `A + B` | Requires precedence rules + parentheses |
| **Postfix** (Reverse Polish) | After operands | `A B +` | Unambiguous; no parentheses needed |
| **Prefix** (Polish) | Before operands | `+ A B` | Unambiguous; no parentheses needed |

**Why postfix/prefix matter:** Compilers convert infix source code to postfix for evaluation. A stack-based postfix evaluator is linear and requires no precedence lookups — it mirrors how CPUs naturally evaluate.

### Operator Precedence and Associativity

```
Precedence level 3 (highest):  ^    right-associative
Precedence level 2:            *, /  left-associative
Precedence level 1 (lowest):   +, -  left-associative
```

**Left-associative:** equal-precedence operators evaluate left-to-right. `A - B - C = (A - B) - C`.
**Right-associative:** equal-precedence operators evaluate right-to-left. `A ^ B ^ C = A ^ (B ^ C)`.

This affects the infix → postfix algorithm: for left-associative operators, an incoming operator of equal precedence pops the stack. For right-associative (`^`), it does not.

**Helper functions used in all conversion code:**

```cpp
int prec(char op) {
    if (op == '^')           return 3;
    if (op == '*' || op == '/') return 2;
    if (op == '+' || op == '-') return 1;
    return 0;   // '(' has lowest stack precedence
}

bool isRightAssoc(char op) { return op == '^'; }
bool isOperator(char c)    { return c=='+' || c=='-' || c=='*' || c=='/' || c=='^'; }
bool isOperand(char c)     { return isalpha(c) || isdigit(c); }
```

---

## 11. Conversion 1 — Infix to Postfix

**Algorithm** — scan left to right:
1. **Operand:** append directly to output.
2. **`(`:** push onto stack.
3. **`)`:** pop and append until `(` is popped; discard both parentheses.
4. **Operator `op`:** while (stack non-empty) AND (top ≠ `(`) AND (prec(top) > prec(op) OR (prec(top) == prec(op) AND op is left-associative)): pop and append. Then push `op`.
5. **End:** pop all remaining stack operators and append.

**Why this works:** Operands must appear in the output in their original order. Operators must appear after both their operands. The stack holds operators whose second operand has not yet been output. When a new operator with lower or equal (left-assoc) precedence arrives, the prior operator's second operand is complete — pop and output it.

```cpp
string infixToPostfix(const string& s) {
    stack<char> st;
    string res;
    for (char c : s) {
        if (isOperand(c)) {
            res += c;
        } else if (c == '(') {
            st.push(c);
        } else if (c == ')') {
            while (!st.empty() && st.top() != '(') { res += st.top(); st.pop(); }
            st.pop();   // discard '('
        } else {        // operator
            while (!st.empty() && st.top() != '('
                   && (prec(st.top()) > prec(c)
                       || (prec(st.top()) == prec(c) && !isRightAssoc(c)))) {
                res += st.top(); st.pop();
            }
            st.push(c);
        }
    }
    while (!st.empty()) { res += st.top(); st.pop(); }
    return res;
}
```

**Dry run** — `s = "a+b*c"`:

```
Token  Stack    Output   Reason
a      []       a        operand
+      [+]      a        stack empty → push
b      [+]      ab       operand
*      [+,*]    ab       prec(*)=2 > prec(+)=1 → push
c      [+,*]    abc      operand
END    []       abc*+    pop *, pop +
Result: abc*+  ✓
```

**Dry run** — `s = "a+b*c+d"`:

```
Token  Stack    Output   Reason
a      []       a
+      [+]      a
b      [+]      ab
*      [+,*]    ab       prec(*)>prec(+) → push
c      [+,*]    abc
+      [+]      abc*+    prec(+)==prec(+), left-assoc → pop *, pop old +; push new +
d      [+]      abc*+d
END    []       abc*+d+
Result: abc*+d+  ✓
```

**Dry run** — `s = "(a+b)*c"`:

```
Token  Stack    Output   Reason
(      [(]
a      [(]      a
+      [(,+]    a
b      [(,+]    ab
)      []       ab+      pop until '(', discard '('
*      [*]      ab+
c      [*]      ab+c
END    []       ab+c*
Result: ab+c*  ✓
```

**Right-associativity dry run** — `s = "a^b^c"`:

```
Token  Stack    Output   Reason
a      []       a
^      [^]      a
b      [^]      ab
^      [^,^]    ab       prec(^)==prec(^) but ^ is right-assoc → do NOT pop, push
c      [^,^]    abc
END    []       abc^^    pop second ^, pop first ^
Result: abc^^  ✓   (a^(b^c) in postfix)
```

Time: O(n). Space: O(n).

---

## 12. Conversion 2 — Infix to Prefix

**Key insight:** Prefix is the reverse of the postfix of the reversed (and bracket-flipped) infix string.

**Algorithm:**
1. Reverse the infix string. While reversing, swap `(` ↔ `)`.
2. Apply the infix-to-postfix algorithm to the reversed string, but treat all operators as right-associative (pop only on strictly greater precedence for equal-prec operators — the reversal changes the effective associativity).
3. Reverse the resulting postfix string — the result is prefix.

**Why this works:** Reversing infix mirrors the expression. Postfix on the mirror gives mirrored-postfix; reversing that gives prefix. The bracket swap corrects for `(A+B)` becoming `)B+A(` after reversal — swapping makes it `(B+A)`, correctly brackets the reversed sub-expression.

```cpp
string infixToPrefix(string s) {
    // Step 1: reverse and swap brackets
    reverse(s.begin(), s.end());
    for (char& c : s) {
        if (c == '(') c = ')';
        else if (c == ')') c = '(';
    }

    // Step 2: postfix on modified string, pop only on strictly greater precedence
    stack<char> st;
    string res;
    for (char c : s) {
        if (isOperand(c)) {
            res += c;
        } else if (c == '(') {
            st.push(c);
        } else if (c == ')') {
            while (!st.empty() && st.top() != '(') { res += st.top(); st.pop(); }
            st.pop();
        } else {
            while (!st.empty() && st.top() != '(' && prec(st.top()) > prec(c)) {
                res += st.top(); st.pop();
            }
            st.push(c);
        }
    }
    while (!st.empty()) { res += st.top(); st.pop(); }

    // Step 3: reverse the result
    reverse(res.begin(), res.end());
    return res;
}
```

**Dry run** — `s = "a+b*c"`:

```
Step 1: reverse "a+b*c" → "c*b+a" (no brackets)
Step 2: postfix of "c*b+a":
  c→c; *→push; b→cb; +: prec(*)>prec(+)→pop *; push + → output=cb*, stack=[+]
  a→cb*a; END→pop + → cb*a+
Step 3: reverse "cb*a+" → "+a*bc"
Result: +a*bc  ✓  (a+(b*c))
```

**Dry run** — `s = "(a+b)*c"`:

```
Step 1: "(a+b)*c" reversed char by char → "c*)b+a("
        swap brackets: "c*(b+a)"
Step 2: postfix of "c*(b+a)":
  c→c; *→push; (→push; b→cb; +→push; a→cba; )→pop + →cba+; discard (; END→pop *→cba+*
Step 3: reverse "cba+*" → "*+abc"
Result: *+abc  ✓  ((a+b)*c)
```

Time: O(n). Space: O(n).

---

## 13. Conversion 3 — Postfix to Infix

**Algorithm** — scan left to right:
1. **Operand:** push as string onto stack.
2. **Operator:** pop `op2 = top`, pop `op1 = second`. Push `"(" + op1 + op + op2 + ")"`.

**Intuition:** In postfix, an operator immediately follows both its operands. When encountered, both operands are already on the stack as strings. Wrap with parentheses to preserve correct infix precedence.

```cpp
string postfixToInfix(const string& s) {
    stack<string> st;
    for (char c : s) {
        if (isOperand(c)) {
            st.push(string(1, c));
        } else {
            string op2 = st.top(); st.pop();
            string op1 = st.top(); st.pop();
            st.push("(" + op1 + c + op2 + ")");
        }
    }
    return st.top();
}
```

**Dry run** — `s = "ab+c*"`:

```
a → ["a"]
b → ["a","b"]
+ → op2="b", op1="a" → push "(a+b)" → ["(a+b)"]
c → ["(a+b)","c"]
* → op2="c", op1="(a+b)" → push "((a+b)*c)" → ["((a+b)*c)"]
Result: ((a+b)*c)  ✓
```

Time: O(n). Space: O(n).

---

## 14. Conversion 4 — Postfix to Prefix

**Algorithm** — scan left to right:
1. **Operand:** push as string.
2. **Operator:** pop `op2 = top`, pop `op1 = second`. Push `op + op1 + op2`.

Prefix format: `operator first_operand second_operand`.

```cpp
string postfixToPrefix(const string& s) {
    stack<string> st;
    for (char c : s) {
        if (isOperand(c)) {
            st.push(string(1, c));
        } else {
            string op2 = st.top(); st.pop();
            string op1 = st.top(); st.pop();
            st.push(string(1, c) + op1 + op2);
        }
    }
    return st.top();
}
```

**Dry run** — `s = "AB+CD-*"`:

```
A → ["A"]
B → ["A","B"]
+ → op2="B", op1="A" → push "+AB" → ["+AB"]
C → ["+AB","C"]
D → ["+AB","C","D"]
- → op2="D", op1="C" → push "-CD" → ["+AB","-CD"]
* → op2="-CD", op1="+AB" → push "*+AB-CD"
Result: *+AB-CD  ✓
```

Time: O(n). Space: O(n).

---

## 15. Conversion 5 — Prefix to Infix

**Algorithm** — scan **right to left**:
1. **Operand:** push as string.
2. **Operator:** pop `op1 = top`, pop `op2 = second`. Push `"(" + op1 + op + op2 + ")"`.

**Why right to left:** In prefix, operators precede their operands. Scanning right to left means when an operator is encountered, its operands have already been processed and sit on the stack.

```cpp
string prefixToInfix(const string& s) {
    stack<string> st;
    for (int i = s.size() - 1; i >= 0; i--) {
        char c = s[i];
        if (isOperand(c)) {
            st.push(string(1, c));
        } else {
            string op1 = st.top(); st.pop();
            string op2 = st.top(); st.pop();
            st.push("(" + op1 + c + op2 + ")");
        }
    }
    return st.top();
}
```

**Dry run** — `s = "*+AB-CD"` (scan right to left: D,C,-,B,A,+,*):

```
D → ["D"]
C → ["D","C"]
- → op1="C", op2="D" → push "(C-D)" → ["(C-D)"]
B → ["(C-D)","B"]
A → ["(C-D)","B","A"]
+ → op1="A", op2="B" → push "(A+B)" → ["(C-D)","(A+B)"]
* → op1="(A+B)", op2="(C-D)" → push "((A+B)*(C-D))"
Result: ((A+B)*(C-D))  ✓
```

Time: O(n). Space: O(n).

---

## 16. Conversion 6 — Prefix to Postfix

**Algorithm** — scan **right to left**:
1. **Operand:** push as string.
2. **Operator:** pop `op1 = top`, pop `op2 = second`. Push `op1 + op2 + op`.

Postfix format: `first_operand second_operand operator`.

```cpp
string prefixToPostfix(const string& s) {
    stack<string> st;
    for (int i = s.size() - 1; i >= 0; i--) {
        char c = s[i];
        if (isOperand(c)) {
            st.push(string(1, c));
        } else {
            string op1 = st.top(); st.pop();
            string op2 = st.top(); st.pop();
            st.push(op1 + op2 + string(1, c));
        }
    }
    return st.top();
}
```

**Dry run** — `s = "*+AB-CD"` (scan right to left: D,C,-,B,A,+,*):

```
D → ["D"]
C → ["D","C"]
- → op1="C", op2="D" → push "CD-" → ["CD-"]
B → ["CD-","B"]
A → ["CD-","B","A"]
+ → op1="A", op2="B" → push "AB+" → ["CD-","AB+"]
* → op1="AB+", op2="CD-" → push "AB+CD-*"
Result: AB+CD-*  ✓
```

Time: O(n). Space: O(n).

---

## 17. The Unifying Map — All Six Conversions

```
              POSTFIX ◄──────────────► PREFIX
                  ▲                       ▲
                  │                       │
                  └──────── INFIX ────────┘
```

| Conversion | Scan direction | Stack type | On operator |
|---|---|---|---|
| Infix → Postfix | Left to right | Operator chars | Pop by precedence; append to output |
| Infix → Prefix | Reverse + flip → postfix → reverse | Operator chars | Pop on strict > only |
| Postfix → Infix | Left to right | Operand strings | Pop 2 → `(op1 OP op2)` |
| Postfix → Prefix | Left to right | Operand strings | Pop 2 → `OP op1 op2` |
| Prefix → Infix | Right to left | Operand strings | Pop 2 → `(op1 OP op2)` |
| Prefix → Postfix | Right to left | Operand strings | Pop 2 → `op1 op2 OP` |

**Memory device for string-stack conversions (when source is postfix or prefix):**

```
Source = postfix → scan LEFT  to right
Source = prefix  → scan RIGHT to left

Pop order:  op2 = top (came last), op1 = second (came first)
            "second comes off FIRST" (LIFO)

→ Target infix:   (op1 OP op2)
→ Target prefix:  OP op1 op2
→ Target postfix: op1 op2 OP
```

---

## 18. Why the Two-Stack Queue is Amortized O(1) — Formal Proof

**Claim:** The lazy-transfer queue using two stacks performs all n push, pop, and peek operations in O(1) amortized time.

**Proof using the accounting method:**

Assign a credit of 3 units to each `push` operation:
- 1 unit pays for the `s_in.push(x)`.
- 1 unit pays for the future move from `s_in` to `s_out` (the transfer).
- 1 unit pays for the future `s_out.pop()`.

Each element is pushed to `s_in` exactly once, moved to `s_out` at most once, and popped from `s_out` exactly once — 3 O(1) operations total per element. The `pop` and `peek` operations use pre-paid credit, so their actual work (the transfer, if triggered) is already accounted for. Total cost over n operations = 3n = O(n), so every operation is O(1) amortized.

**Worst case for a single pop:** O(n) — when `s_out` is empty and all n elements are transferred from `s_in`.

**Interview answer:** "O(1) amortized, O(n) worst case for a single operation." Saying only O(1) is wrong; saying only O(n) misses the amortized insight.

---

## 19. Edge Cases Master Table

| Scenario | Problem | Input | Expected | Reason |
|---|---|---|---|---|
| Pop empty stack | Stack array | pop() on empty | -1 | `top==-1` guard |
| Stack overflow | Stack array | push beyond capacity | ignore | Check `top < N-1` |
| Circular wrap | Queue array | push after several pop cycles | succeeds | Modulo indexing |
| Single element dequeue | Queue LL | pop() when one element | val, front=rear=null | Must set rear=null when front becomes null |
| `s = ")"` | Valid Parens | `")"` | false | Stack empty when `)` encountered |
| `s = "((("` | Valid Parens | `"((("` | false | Stack non-empty at end |
| `s = "([)]"` | Valid Parens | `"([)]"` | false | Top mismatch on `)` |
| `s = ""` | Valid Parens | `""` | true | Trivially valid |
| All same element | Min Stack | push(2),push(2),pop(),getMin() | 2 | Equal elements: `min(val, top)` preserves 2 |
| Single operand | Postfix→Infix | `"A"` | `"A"` | One string left on stack |
| Right-assoc `^` | Infix→Postfix | `"a^b^c"` | `"abc^^"` | Equal `^` does not pop |
| All same push/pop | Queue via 2 stacks | push(1),pop(),push(2),pop() | 1, then 2 | Transfer only when s_out empty |
| INT_MIN in Min Stack | Min Stack encoded | push(INT_MIN) | correct getMin | Use long long for `2*val - curMin` |

---

## 20. Common Mistakes and Pitfalls

### Mistake 1 — Stack array: mixing pre/post increment idioms

```cpp
// WRONG: inconsistent idiom causes off-by-one in edge cases
arr[top] = x; top++;    // push stores at old top, then increments
return arr[top];        // pop returns new top, not the one just stored

// CORRECT: pick one consistent idiom
arr[++top] = x;         // push: increment first, store at new top
return arr[top--];      // pop: read current top, then decrement
```

### Mistake 2 — Queue array: not using circular indexing

```cpp
// WRONG: rear hits end of array even with free space at front
arr[rear++] = x;

// CORRECT:
rear = (rear + 1) % capacity;
```

### Mistake 3 — Queue linked list: not nullifying rear when queue empties

```cpp
// WRONG: rear still points to freed node; next push re-attaches dangling pointer
front = front->next;
delete tmp;
// Missing: if (!front) rear = nullptr;

// CORRECT:
front = front->next;
if (!front) rear = nullptr;
```

### Mistake 4 — Two-stack queue: transferring when s_out is not empty

```cpp
// WRONG: overwrites existing s_out elements, breaking FIFO order
void pop() {
    while (!s_in.empty()) { s_out.push(s_in.top()); s_in.pop(); }  // always transfers
    s_out.pop();
}

// CORRECT: transfer only when s_out is empty
void pop() {
    if (s_out.empty())
        while (!s_in.empty()) { s_out.push(s_in.top()); s_in.pop(); }
    s_out.pop();
}
```

### Mistake 5 — Valid Parentheses: not checking `st.empty()` at the end

```cpp
// WRONG: "(((" returns true (loop exits without error)
for (char c : s) { ... }
return true;

// CORRECT
return st.empty();
```

### Mistake 6 — Min Stack encoded: forgetting long long

```cpp
// WRONG: 2*INT_MIN overflows int
st.push(2 * val - curMin);

// CORRECT
st.push(2LL * val - curMin);
```

### Mistake 7 — Infix to Postfix: wrong handling of equal-precedence right-associative `^`

```cpp
// WRONG: >= pops ^ on equal precedence, giving left-associative behavior
while (!st.empty() && prec(st.top()) >= prec(c)) { pop; }

// CORRECT: for right-associative ops, pop only on strictly greater
while (!st.empty() && st.top() != '(' &&
       (prec(st.top()) > prec(c) ||
       (prec(st.top()) == prec(c) && !isRightAssoc(c))))
    { pop; }
```

### Mistake 8 — Prefix/Postfix conversions: wrong operand order

The mnemonic is **"second comes off first" (LIFO)**: `op2 = st.top()` (most recently pushed = second operand in the expression), `op1 = st.second_top()` (first operand).

```cpp
// WRONG: op1 and op2 swapped
string op1 = st.top(); st.pop();   // actually op2 (second operand)
string op2 = st.top(); st.pop();   // actually op1 (first operand)
st.push(string(1,c) + op1 + op2); // wrong prefix order

// CORRECT
string op2 = st.top(); st.pop();   // top = second operand
string op1 = st.top(); st.pop();   // second = first operand
st.push(string(1,c) + op1 + op2); // OP first_op second_op
```

---

## 21. Complexity Cheat Sheet

| Problem / Operation | Time | Space | Notes |
|---|---|---|---|
| Stack (array): push, pop, top | O(1) | O(N) static | N = capacity |
| Queue (circular array): push, pop, front | O(1) | O(N) static | |
| Stack (linked list): push, pop, top | O(1) | O(n) dynamic | Pointer overhead |
| Queue (linked list): push, pop, front | O(1) | O(n) dynamic | |
| Stack using Queues (rotate, 1 queue) | push O(n), pop O(1) | O(n) | |
| Queue using Stacks (lazy transfer) | push O(1), pop O(1) amortized | O(n) | O(n) worst single pop |
| Valid Parentheses | O(n) | O(n) | Stack depth = max nesting |
| Min Stack (two stacks) | All O(1) | O(n) | |
| Min Stack (encoded single stack) | All O(1) | O(n) | Needs long long; half constant factor |
| Infix → Postfix / Prefix | O(n) | O(n) | |
| Postfix/Prefix → any other | O(n) | O(n) | String concat; use builder for O(n) |

---

## 22. Interview Reference — Patterns and Applications

### When to use a stack

| Signal | Reason |
|---|---|
| "Most recent / last added" processed first | Core LIFO |
| Matching pairs (brackets, tags, functions) | Stack stores unmatched opens; pop on close |
| Undo/redo | Push on action; pop on undo |
| DFS (iterative) | Stack replaces call stack |
| Expression evaluation / conversion | Operator precedence reversal |
| Next greater element | Monotonic stack |
| Largest rectangle in histogram | Monotonic stack |

### When to use a queue

| Signal | Reason |
|---|---|
| "First arrived" processed first | Core FIFO |
| BFS traversal | Queue holds current-level nodes |
| Level-order tree traversal | Queue per level |
| Sliding window maximum | Monotonic deque |

### Problems that build directly on this set

- **LC 84 Largest Rectangle in Histogram:** Monotonic stack. Push increasing bars; on shorter bar, pop and compute area.
- **LC 739 Daily Temperatures:** Monotonic stack for next greater element.
- **LC 150 Evaluate Reverse Polish Notation:** Postfix evaluation — operand: push; operator: pop two, compute, push.
- **LC 224/227 Basic Calculator I/II:** Infix evaluation using stack.
- **LC 42 Trapping Rain Water:** Monotonic stack version.
- **LC 907 Sum of Subarray Minimums:** Monotonic stack contribution counting.

### The Min Stack insight generalizes

The auxiliary-stack technique (tracking a secondary invariant at every stack level) applies to any per-level property: max stack (parallel `maxSt`), count stack (frequency tracking), sum stack (prefix sums for expression evaluation).

---

## 23. Quick Revision Cheat Sheet

**Stack array** (`top` starts at -1): `push`: `arr[++top]=x`. `pop`: `return arr[top--]`. Empty: `top==-1`.

**Queue circular array** (`front=rear=0, sz=0`): `push`: `arr[rear]=x; rear=(rear+1)%N; sz++`. `pop`: `val=arr[front]; front=(front+1)%N; sz--`.

**Stack LL:** push = insert at head. pop = remove head. `top → newest → ... → oldest → null`.

**Queue LL:** push = append to rear. pop = remove from front. Maintain `front` and `rear`. Set `rear=null` when front becomes null.

**Stack using 1 Queue (optimal):** push: enqueue x, rotate `size-1` times (bring x to front). pop/top: use `q.front()`.

**Queue using 2 Stacks (amortized O(1)):** push → `s_in`. pop/peek: if `s_out` empty, dump all `s_in` to `s_out`; use `s_out.top()`.

**Valid Parentheses:** push open brackets; on close: empty or top mismatch → false; return `st.empty()`.

**Min Stack (two stacks):** `minSt.top() = min(val, minSt.top())` on push; pop both together.

**Precedence:** `^`=3 (right-assoc), `*/`=2 (left-assoc), `+-`=1 (left-assoc).

**Infix → Postfix:** operand→output; `(`→push; `)`→pop until `(`; operator: pop while top has higher or equal-left-assoc prec, then push; end→pop all.

**Infix → Prefix:** reverse + swap brackets → postfix (pop only on strict `>`) → reverse result.

**Postfix/Prefix → other (string stack):**
```
Source postfix: scan LEFT  to right
Source prefix:  scan RIGHT to left
op2 = st.top() (second operand)
op1 = st.second (first operand)   ← "second comes off first"
→ Infix:   (op1 OP op2)
→ Prefix:  OP op1 op2
→ Postfix: op1 op2 OP
```

**Amortized O(1) queue proof:** Each element: 1 push to s_in + 1 move to s_out + 1 pop from s_out = 3 O(1) ops. Total O(n) for n elements. All operations O(1) amortized. Single worst-case pop: O(n).
