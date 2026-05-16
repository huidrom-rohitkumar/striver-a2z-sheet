**Topic:** Linked List Core Interview Patterns — Slow/Fast Pointers, Reversal, Cycle Detection, Palindrome, Segregation
**Patterns:** Tortoise-Hare, In-Place Pointer Reversal, Floyd Cycle Detection, Half Reversal, Two-Chain Segregation


**Resources:**
[Middle LL](https://youtu.be/D2vI2DNJGd8) |
[Reverse Iterative](https://youtu.be/7LjQ57RqgEc) |
[Reverse Recursive](https://youtu.be/wiOo4DC5GGA) |
[Detect Loop](https://youtu.be/I4g1qbkTPus) |
[Find Loop Start](https://youtu.be/2Kd0KKmmHFc) |
[Loop Length](https://youtu.be/lRY_G-u_8jk) |
[Palindrome + Odd-Even](https://youtu.be/qf6qp7GzD5Q) |
[GFG Loop Length](https://www.geeksforgeeks.org/problems/find-length-of-loop/1)

**Problems:** LC 876 Middle | LC 206 Reverse | LC 141 Detect Cycle | LC 142 Cycle Start | GFG Loop Length | LC 234 Palindrome | LC 328 Odd-Even

---

## Table of Contents

1. [The Unifying Mental Model](#1-the-unifying-mental-model)
2. [Problem 1 — Middle of a Linked List (LC 876)](#2-problem-1--middle-of-a-linked-list-lc-876)
3. [Problem 2 — Reverse Linked List — Iterative (LC 206)](#3-problem-2--reverse-linked-list--iterative-lc-206)
4. [Problem 3 — Reverse Linked List — Recursive (LC 206)](#4-problem-3--reverse-linked-list--recursive-lc-206)
5. [Problem 4 — Detect a Loop in a Linked List (LC 141)](#5-problem-4--detect-a-loop-in-a-linked-list-lc-141)
6. [Problem 5 — Find the Starting Point of a Loop (LC 142)](#6-problem-5--find-the-starting-point-of-a-loop-lc-142)
7. [Problem 6 — Length of Loop in a Linked List (GFG)](#7-problem-6--length-of-loop-in-a-linked-list-gfg)
8. [Problem 7 — Check if a Linked List is a Palindrome (LC 234)](#8-problem-7--check-if-a-linked-list-is-a-palindrome-lc-234)
9. [Problem 8 — Segregate Odd and Even Nodes (LC 328)](#9-problem-8--segregate-odd-and-even-nodes-lc-328)
10. [Floyd's Algorithm — Complete Mathematical Proof](#10-floyds-algorithm--complete-mathematical-proof)
11. [Complexity Cheat Sheet](#11-complexity-cheat-sheet)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Model

Half the problems in this set share one primitive: **two pointers moving at different speeds through the same list**. The slow pointer (tortoise) moves one node per step; the fast pointer (hare) moves two. This 2:1 speed ratio creates predictable arithmetic relationships between distances that power multiple problems.

| Application | What the 2:1 ratio gives you |
|---|---|
| Find middle | When fast reaches end, slow is at midpoint |
| Detect cycle | If cycle exists, fast laps slow and they meet |
| Find cycle start | Reset one pointer to head after meeting; both reach cycle start simultaneously |
| Find cycle length | Count steps from meeting point back to meeting point |
| Palindrome check | Find middle, reverse second half, compare |

The other three problems — reverse (iterative and recursive), and odd-even segregation — use a different primitive: **relinking pointers in a single pass** while tracking the previous, current, and next nodes. No speed ratio; just disciplined pointer management.

Recognizing which primitive applies is the interview skill. The code follows mechanically once you see it.

---

## 2. Problem 1 — Middle of a Linked List (LC 876)

**Problem:** Return the middle node of a singly linked list. If there are two middle nodes (even length), return the second one.
**Link:** [LC 876](https://leetcode.com/problems/middle-of-the-linked-list/)
**Difficulty:** Easy

**Examples:**
- `1 → 2 → 3 → 4 → 5` → Node 3 (middle of 5)
- `1 → 2 → 3 → 4` → Node 3 (second middle of 4)

### Approach 1 — Count and Traverse (Two Pass)

Count nodes to get `n`, then traverse to index `n/2`.

```cpp
ListNode* middleNode(ListNode* head) {
    int n = 0;
    ListNode* curr = head;
    while (curr) { n++; curr = curr->next; }
    curr = head;
    for (int i = 0; i < n / 2; i++) curr = curr->next;
    return curr;
}
```

Time: O(n). Space: O(1). Correct but visits every node twice.

### Approach 2 — Tortoise and Hare (Single Pass, Optimal)

**Intuition:** At step `t`, slow has moved `t` nodes and fast has moved `2t` nodes. When fast reaches the end (`2t ≈ n`), slow is at position `t ≈ n/2` — the midpoint. One pass, no counting.

```cpp
ListNode* middleNode(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;
}
```

**Loop condition `fast && fast->next`:**
- `fast != nullptr`: prevents a null dereference when fast lands exactly on NULL (even-length lists).
- `fast->next != nullptr`: prevents `fast->next->next` from crashing when fast is at the last node (odd-length lists).

**Dry run — odd length:** `1 → 2 → 3 → 4 → 5`

```
slow=1, fast=1

Step 1: slow=2, fast=3   (fast: 1→2→3)
Step 2: slow=3, fast=5   (fast: 3→4→5)
        fast->next = NULL → stop

return slow = 3  ✓
```

**Dry run — even length:** `1 → 2 → 3 → 4`

```
slow=1, fast=1

Step 1: slow=2, fast=3
Step 2: slow=3, fast=NULL   (fast: 3→4→NULL)
        fast == NULL → stop

return slow = 3  ✓ (second middle)
```

**Why second middle for even-length?** The condition `fast && fast->next` allows fast to land on NULL (past the last node). For a 4-element list, fast passes through positions 1→3→NULL, meaning slow passes 1→2→3. Slow lands on the second middle. This is the correct behavior per LC 876.

**If you need the first middle** (for palindrome problems where you must split after the first half): use `while (fast->next && fast->next->next)`. This stops slow one position earlier.

| Input | Output | Reason |
|---|---|---|
| `[1]` | Node 1 | Single node |
| `[1,2]` | Node 2 | Even: second middle |
| `[1,2,3]` | Node 2 | Odd: exact middle |
| `[1,2,3,4]` | Node 3 | Even: second middle |

Time: O(n). Space: O(1).

---

## 3. Problem 2 — Reverse Linked List — Iterative (LC 206)

**Problem:** Reverse a singly linked list in-place. Return the new head.
**Link:** [LC 206](https://leetcode.com/problems/reverse-linked-list/)
**Difficulty:** Easy

### The Three-Pointer Technique

**Intuition:** To redirect each node's `next` pointer backward, you need three pointers simultaneously: `prev` (the growing reversed list behind you), `curr` (the node being processed), and `next` (saved copy of `curr->next` before you overwrite it). Without saving `next` first, the forward chain is permanently broken.

At each step:
1. Save `curr->next` in `next`.
2. Redirect: `curr->next = prev`.
3. Advance: `prev = curr`, `curr = next`.

When `curr` becomes NULL, `prev` is the new head of the fully reversed list.

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* curr = head;
    while (curr != nullptr) {
        ListNode* next = curr->next;   // 1. save
        curr->next = prev;             // 2. redirect
        prev = curr;                   // 3a. advance prev
        curr = next;                   // 3b. advance curr
    }
    return prev;
}
```

**Loop invariant:** At the start of each iteration, all nodes from the original head up to and including `prev` are correctly reversed. All nodes from `curr` onward are still in original order.

**Dry run:** `1 → 2 → 3 → 4 → 5 → NULL`

```
prev=NULL, curr=1

Iter 1: next=2; 1→NULL; prev=1, curr=2   | Reversed: [1]  | Remaining: 2→3→4→5
Iter 2: next=3; 2→1→NULL; prev=2, curr=3 | Reversed: [2→1] | Remaining: 3→4→5
Iter 3: next=4; 3→2→1→NULL; prev=3, curr=4
Iter 4: next=5; 4→3→2→1→NULL; prev=4, curr=5
Iter 5: next=NULL; 5→4→3→2→1→NULL; prev=5, curr=NULL

curr==NULL → stop. Return prev=5.
Final: 5→4→3→2→1→NULL  ✓
```

Time: O(n). Space: O(1).

---

## 4. Problem 3 — Reverse Linked List — Recursive (LC 206)

**Difficulty:** Easy

### The Recursive Insight

**Intuition:** Assume `reverseList(head->next)` correctly reverses everything from `head->next` onward and returns the new head. After this call, `head->next` is the **last node** of the reversed sublist. Attach `head` after it: `head->next->next = head`. Then nullify `head->next` (head is now the tail). Return `newHead` unchanged.

```cpp
ListNode* reverseList(ListNode* head) {
    if (head == nullptr || head->next == nullptr) return head;

    ListNode* newHead = reverseList(head->next);

    head->next->next = head;   // attach head after the end of the reversed sublist
    head->next = nullptr;      // head is the new tail; cut the old forward link

    return newHead;            // new head propagates unchanged up the call stack
}
```

**Dry run:** `1 → 2 → 3 → NULL`

```
reverseList(1) calls reverseList(2) calls reverseList(3)
  Base: return 3 (newHead=3)

Back in reverseList(2):
  newHead=3
  head(2)->next(3)->next = head(2)  →  3→2
  head(2)->next = NULL              →  3→2→NULL
  return 3

Back in reverseList(1):
  newHead=3
  head(1)->next(2)->next = head(1)  →  2→1
  head(1)->next = NULL              →  2→1→NULL
  return 3

Final: 3→2→1→NULL  ✓
```

**Critical detail:** `head->next->next = head` works because `head->next` still points to the original successor (which is now the last node of the reversed sublist) — we set it before nullifying.

**Why iterative is preferred:** Recursive version has O(n) call stack depth. For n = 100,000, this risks stack overflow. Interviewers expect you to know both; when O(1) space is required, use iterative.

Time: O(n). Space: O(n) call stack.

---

## 5. Problem 4 — Detect a Loop in a Linked List (LC 141)

**Problem:** Given the head of a linked list, return `true` if it contains a cycle, `false` otherwise.
**Link:** [LC 141](https://leetcode.com/problems/linked-list-cycle/)
**Difficulty:** Easy

### Approach 1 — Hash Set

Store visited node addresses. If a node is seen twice, a cycle exists.

```cpp
bool hasCycle(ListNode* head) {
    unordered_set<ListNode*> visited;
    while (head) {
        if (visited.count(head)) return true;
        visited.insert(head);
        head = head->next;
    }
    return false;
}
```

Time: O(n). Space: O(n).

### Approach 2 — Floyd's Cycle Detection (Optimal)

**Intuition:** Without a cycle, fast reaches NULL and the loop exits. With a cycle, fast is trapped inside it. Once both pointers are in the cycle, consider the distance from fast to slow going forward in the cycle. Fast gains 1 step on slow every iteration (fast moves 2, slow moves 1, net gain = 1). The distance decreases by 1 each step and must eventually reach 0 — they meet. The formal proof is in Section 10.

```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

**Dry run — cycle present:** `3 → 2 → 0 → -4 → (back to 2)`

```
Nodes (indexed): 3[0] → 2[1] → 0[2] → -4[3] → 2[1] (cycle)

slow=3, fast=3
Step 1: slow=2[1], fast=0[2]    (fast: 3→2→0)
Step 2: slow=0[2], fast=2[1]    (fast: 0→-4→2)
Step 3: slow=-4[3], fast=-4[3]  (fast: 2→0→-4)
slow == fast → return true  ✓
```

**Dry run — no cycle:** `1 → 2 → NULL`

```
Step 1: slow=2, fast=NULL  → fast==NULL → stop → return false  ✓
```

Time: O(n). Space: O(1).

---

## 6. Problem 5 — Find the Starting Point of a Loop (LC 142)

**Problem:** Given a linked list with a cycle, return the node where the cycle begins. Return `nullptr` if no cycle.
**Link:** [LC 142](https://leetcode.com/problems/linked-list-cycle-ii/)
**Difficulty:** Medium

### Algorithm

**Step 1:** Run Floyd's detection to find the meeting point inside the cycle.

**Step 2:** Reset slow to `head`. Keep fast at the meeting point. Advance both at speed 1. They will meet exactly at the cycle's starting node.

**Why this works:** Let L = distance from head to cycle start, C = cycle length, d = distance from cycle start to meeting point. From the proof in Section 10: `L = nC - d`. After resetting slow to head, slow needs L steps to reach the cycle start. Fast, from the meeting point, needs `C - d` steps to reach the cycle start going forward, and then `(n-1)` more full cycles — a total of `(n-1)C + (C-d) = nC - d = L` steps. Both arrive at the cycle start simultaneously.

```cpp
ListNode* detectCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;

    // Step 1: find meeting point
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) break;
    }

    // Check if there was actually a cycle
    if (fast == nullptr || fast->next == nullptr) return nullptr;

    // Step 2: find cycle start
    slow = head;
    while (slow != fast) {
        slow = slow->next;
        fast = fast->next;
    }

    return slow;   // slow == fast == cycle start
}
```

**Dry run:** `3 → 2 → 0 → -4 → (back to 2)`, cycle start = node 2

Step 1 (detect, from earlier dry run): meeting point = node `-4`.

Step 2:
```
slow = head = 3, fast = -4(meeting point)

Iter 1: slow=2, fast=2   (fast: -4 → 2)
slow == fast → return slow = 2  ✓
```

| Scenario | Expected |
|---|---|
| No cycle | `nullptr` |
| Entire list is a loop (tail → head) | `head` |
| Cycle starting at last node pointing to itself | That node |
| Cycle starting at first node after head | Second node |

Time: O(n). Space: O(1).

---

## 7. Problem 6 — Length of Loop in a Linked List (GFG)

**Problem:** Given a linked list, if a loop exists, find its length. Return 0 if no loop.
**Link:** [GFG Loop Length](https://www.geeksforgeeks.org/problems/find-length-of-loop/1)

### Algorithm

After finding the meeting point inside the cycle (Floyd's Step 1), keep one pointer fixed at the meeting point and advance the other node by node until it returns. Count the steps.

```cpp
int countLoop(Node* meetingPoint) {
    Node* curr = meetingPoint->next;
    int count = 1;
    while (curr != meetingPoint) {
        count++;
        curr = curr->next;
    }
    return count;
}

int countNodesinLoop(Node* head) {
    Node* slow = head;
    Node* fast = head;
    while (fast != nullptr && fast->next != nullptr) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return countLoop(slow);
    }
    return 0;
}
```

**Dry run:** Loop is `3 → 4 → 5 → 3` (length 3). Meeting point = node 4.

```
countLoop(node 4):
  curr=5, count=1
  curr=3, count=2
  curr=4 == meetingPoint → stop
  return 3  ✓
```

**Alternative:** Find the cycle start (LC 142 approach), then count nodes until you return to the start. Both are O(n) time, O(1) space. The meeting-point approach above is simpler since it avoids the separate cycle-start computation.

Time: O(n). Space: O(1).

---

## 8. Problem 7 — Check if a Linked List is a Palindrome (LC 234)

**Problem:** Return `true` if the linked list is a palindrome. O(n) time, O(1) space.
**Link:** [LC 234](https://leetcode.com/problems/palindrome-linked-list/)
**Difficulty:** Easy (O(1) space constraint makes it Medium in practice)

**Examples:**
- `1 → 2 → 2 → 1` → `true`
- `1 → 2 → 3` → `false`
- `[1]` → `true`

### Approach 1 — Copy to Array

```cpp
bool isPalindrome(ListNode* head) {
    vector<int> vals;
    for (ListNode* curr = head; curr; curr = curr->next) vals.push_back(curr->val);
    int l = 0, r = vals.size() - 1;
    while (l < r) if (vals[l++] != vals[r--]) return false;
    return true;
}
```

Time: O(n). Space: O(n).

### Approach 2 — Find Middle + Reverse Second Half (Optimal)

**The three-step algorithm:**
1. Find the **last node of the first half** using the left-leaning middle (condition `fast->next && fast->next->next`).
2. Reverse the second half (from `slow->next` onward).
3. Compare the first half and the reversed second half node by node.
4. Optionally restore the list.

**Why the left-leaning middle condition here:** The palindrome check requires the two halves to be of equal length. The condition `fast->next && fast->next->next` (rather than `fast && fast->next`) stops slow one position earlier, at the last node of the first half. The second half starts at `slow->next`.

```
Standard:      fast && fast->next   → slow lands at second middle (right-leaning)
For palindrome: fast->next && fast->next->next  → slow lands at end of first half (left-leaning)

For [1,2,2,1]:
  Standard:        slow=2(second) → reverse from 1(last) → [1] → compare [1,2,2] vs [1] → WRONG
  Left-leaning:    slow=2(first)  → reverse from 2(second)→1 → [1,2] → compare [1,2] vs [1,2] → correct
```

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr, *curr = head;
    while (curr) { ListNode* next = curr->next; curr->next = prev; prev = curr; curr = next; }
    return prev;
}

bool isPalindrome(ListNode* head) {
    if (!head || !head->next) return true;

    // Step 1: find end of first half (left-leaning)
    ListNode* slow = head, *fast = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    // slow is now the last node of the first half

    // Step 2: reverse second half
    ListNode* secondHalf = reverseList(slow->next);
    slow->next = nullptr;   // cut (aids comparison loop termination)

    // Step 3: compare
    ListNode* p1 = head, *p2 = secondHalf;
    bool isPalin = true;
    while (p2) {
        if (p1->val != p2->val) { isPalin = false; break; }
        p1 = p1->next;
        p2 = p2->next;
    }

    // Step 4: restore
    slow->next = reverseList(secondHalf);

    return isPalin;
}
```

**Detailed dry run:** `1 → 2 → 2 → 1 → NULL`

```
Step 1: slow=1, fast=1
  fast->next(2) && fast->next->next(2): YES → slow=2(first), fast=2(third)
  fast->next(1) && fast->next->next(NULL): NO → stop
  slow = 2 (first 2 = end of first half)

Step 2: secondHalf = reverse(slow->next = 2(second)→1)
  reverse([2→1]) = 1→2→NULL
  slow->next = nullptr → first half becomes 1→2→NULL

Step 3: p1=1, p2=1: match; p1=2, p2=2: match; p2->next=NULL → stop
  isPalin = true

Step 4: restore. Return true.  ✓
```

**Dry run — not palindrome:** `1 → 2 → 3`

```
Step 1: slow=1, fast=1
  fast->next(2) && fast->next->next(3): YES → slow=2, fast=3
  fast->next(NULL): NO → stop
  slow = 2 (middle)

Step 2: secondHalf = reverse(3) = [3]; slow->next = nullptr → [1→2]

Step 3: p1=1, p2=3 → 1 ≠ 3 → isPalin=false. Return false.  ✓
```

Time: O(n). Space: O(1).

---

## 9. Problem 8 — Segregate Odd and Even Nodes (LC 328)

**Problem:** Group all odd-indexed nodes followed by all even-indexed nodes. Preserve relative order within each group. Indices are 1-based (node 1 is odd, node 2 is even, etc.). This is about **index parity**, not **value parity**.
**Link:** [LC 328](https://leetcode.com/problems/odd-even-linked-list/)
**Difficulty:** Medium

**Examples:**
- `1 → 2 → 3 → 4 → 5` → `1 → 3 → 5 → 2 → 4`
- `2 → 1 → 3 → 5 → 6 → 4 → 7` → `2 → 3 → 6 → 7 → 1 → 5 → 4`

### Algorithm — Two-Chain Single Pass

**Intuition:** Build two separate chains simultaneously: `odd` for nodes at odd indices and `even` for nodes at even indices. At each iteration, extend both chains by one node. Connect the odd chain's tail to the even chain's head when done.

```cpp
ListNode* oddEvenList(ListNode* head) {
    if (!head || !head->next) return head;

    ListNode* odd = head;             // current tail of odd chain
    ListNode* even = head->next;      // current tail of even chain
    ListNode* evenHead = head->next;  // save head of even chain

    while (even != nullptr && even->next != nullptr) {
        odd->next = even->next;    // odd chain: skip over current even node
        odd = odd->next;           // advance odd pointer
        even->next = odd->next;    // even chain: skip over the odd node we just added
        even = even->next;         // advance even pointer
    }

    odd->next = evenHead;          // join: last odd node → first even node
    return head;
}
```

**Loop invariant:** At the start of each iteration, `odd` is the tail of the complete odd chain so far, and `even` is the tail of the complete even chain so far.

**Detailed dry run:** `1 → 2 → 3 → 4 → 5 → NULL`

```
odd=1, even=2, evenHead=2

Iter 1:
  odd->next = even->next = 3    →  odd chain: 1→3
  odd = 3
  even->next = odd->next = 4   →  even chain: 2→4
  even = 4
  State: odd=3, even=4

Iter 2:
  odd->next = even->next = 5    →  odd chain: 1→3→5
  odd = 5
  even->next = odd->next = NULL →  even chain: 2→4→NULL
  even = NULL
  even==NULL → stop

odd->next = evenHead = 2       →  5→2
Return head=1.
Final: 1→3→5→2→4→NULL  ✓
```

**Dry run — even length:** `1 → 2 → 3 → 4 → NULL`

```
odd=1, even=2, evenHead=2

Iter 1:
  odd->next = 3, odd=3
  even->next = 4, even=4

even(4) != NULL, even->next(NULL) == NULL → stop

odd->next = evenHead=2 → 3→2
Final: 1→3→2→4→NULL  ✓
```

| Input | Output | Reason |
|---|---|---|
| `[1]` | `[1]` | Single node, no even nodes |
| `[1,2]` | `[1,2]` | Already segregated |
| `[1,2,3]` | `[1,3,2]` | Two odd, one even |

Time: O(n). Space: O(1).

---

## 10. Floyd's Algorithm — Complete Mathematical Proof

### Variable Definitions

```
L  = number of nodes from head to the cycle start (head to cycle start, not counting cycle start itself)
C  = number of nodes in the cycle
d  = number of nodes from the cycle start to the meeting point (forward direction)
```

```
head → [L nodes] → cycleStart → [d nodes] → meetingPoint → [C-d nodes] → cycleStart
```

### Part 1 — Why They Always Meet

When slow enters the cycle (after L steps), fast has traveled 2L steps. Fast has been inside the cycle for `L mod C` positions ahead of the cycle start. The distance from fast to slow going forward (how far slow must travel to reach fast) is `C - (L mod C)`.

Each step, fast gains 1 on slow (fast moves 2, slow moves 1; net = 1 within the cycle). The gap decreases by 1 per step. Since the gap is a positive integer that decreases by exactly 1 each step, it reaches 0 in finite time. Meeting is guaranteed.

**Why speed 1 and 2 (not other combinations):** If fast moved at speed k instead of 2, the relative gain per step is `k - 1`. If `k - 1 > 1`, the gap decreases by more than 1 per step and can skip over 0, meaning they never meet. Speed 2 gives gain 1, guaranteeing the gap always reaches exactly 0.

### Part 2 — The Meeting Point Formula

At the meeting point:
- Slow has traveled `L + d` total steps.
- Fast has traveled `2(L + d)` total steps (twice the speed).
- Fast traveled an additional `nC` compared to slow (extra full laps inside the cycle):

```
2(L + d) - (L + d) = nC   for some positive integer n
L + d = nC
L = nC - d
L = (n-1)C + (C - d)
```

### Part 3 — Why Resetting One Pointer to Head Finds the Cycle Start

After the meeting point, reset slow to head. Keep fast at the meeting point. Both now move at speed 1.

- Slow needs exactly `L` steps to reach the cycle start.
- Fast, from the meeting point, needs `C - d` steps to reach the cycle start going forward. Then `(n-1)` full laps bring fast back to the same position. Total: `(n-1)C + (C-d) = nC - d = L` steps.

Both pointers take exactly `L` steps from their respective starting positions and arrive at the cycle start simultaneously. They meet there.

### Summary Table

| Variable | Meaning | Value at meeting |
|---|---|---|
| Slow distance | Nodes traversed by slow | L + d |
| Fast distance | Nodes traversed by fast | 2(L + d) |
| Key equation | Derived from slow × 2 = fast | L = nC − d |
| Steps after reset | For both to reach cycle start | L |

---

## 11. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Middle of LL | Count + traverse | O(n) | O(1) |
| Middle of LL | Tortoise-Hare (single pass) | O(n) | O(1) |
| Reverse LL | Iterative (3-pointer) | O(n) | O(1) |
| Reverse LL | Recursive | O(n) | O(n) call stack |
| Detect Cycle | Hash set | O(n) | O(n) |
| Detect Cycle | Floyd's | O(n) | O(1) |
| Find Cycle Start | Hash set (first repeat) | O(n) | O(n) |
| Find Cycle Start | Floyd's + reset | O(n) | O(1) |
| Length of Loop | Floyd's + count | O(n) | O(1) |
| Palindrome LL | Copy to array | O(n) | O(n) |
| Palindrome LL | Stack | O(n) | O(n) |
| Palindrome LL | Middle + reverse half | O(n) | O(1) |
| Odd-Even Segregation | Two-chain pointer | O(n) | O(1) |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1 — Wrong middle condition for palindrome check

```cpp
// WRONG: standard condition gives second (right-leaning) middle
while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
// For [1,2,2,1]: slow stops at 2(second) → reverse only [1] → compare [1,2,2] vs [1] → WRONG

// CORRECT for palindrome: left-leaning condition stops slow at end of first half
while (fast->next && fast->next->next) { slow = slow->next; fast = fast->next->next; }
// For [1,2,2,1]: slow stops at 2(first) → reverse [2,1] → compare [1,2] vs [1,2] → correct
```

### Mistake 2 — Checking cycle equality before moving pointers

```cpp
// WRONG: checking before moving means slow==fast==head at step 0, falsely returns true
while (fast && fast->next) {
    if (slow == fast) return true;   // fires on first iteration even with no cycle
    slow = slow->next;
    fast = fast->next->next;
}

// CORRECT: move first, then check
while (fast && fast->next) {
    slow = slow->next;
    fast = fast->next->next;
    if (slow == fast) return true;
}
```

### Mistake 3 — Reversing pointer: wrong order of operations

```cpp
// WRONG: overwrites curr->next before saving it — permanently loses the rest of the list
curr->next = prev;
ListNode* next = curr->next;   // next is now prev, not the original successor

// CORRECT: always save next BEFORE redirecting
ListNode* next = curr->next;   // save first
curr->next = prev;             // then redirect
```

### Mistake 4 — Recursive reverse: forgetting to nullify the old forward link

```cpp
// WRONG: creates a cycle
ListNode* newHead = reverseList(head->next);
head->next->next = head;   // head->next still points forward → cycle formed
// Missing: head->next = nullptr;

// CORRECT: always nullify after reattaching
head->next->next = head;
head->next = nullptr;      // head is the new tail; cut the old link
```

### Mistake 5 — Odd-even: not saving `evenHead` before the loop

```cpp
// WRONG: even advances during the loop, ends at the last even node, not the first
ListNode* even = head->next;
// ... loop runs, even advances to last even node ...
odd->next = even;   // connects to last even node, not the head of even chain

// CORRECT: save the head of even chain separately before the loop
ListNode* evenHead = head->next;   // immutable reference to even chain head
// ... loop runs ...
odd->next = evenHead;              // always connects to the first even node
```

### Mistake 6 — Null check before `fast->next->next`

```cpp
// WRONG: segfault when fast or fast->next is NULL
fast = fast->next->next;

// CORRECT: guard with two conditions
while (fast != nullptr && fast->next != nullptr) {
    fast = fast->next->next;
}
```

### Mistake 7 — Not handling the "no cycle" case after Floyd's detection in LC 142

```cpp
// WRONG: after the loop, assumes fast == slow (met inside cycle)
// but the loop could have exited because fast reached NULL
slow = head;
while (slow != fast) { ... }   // if fast==NULL, this loops forever or crashes

// CORRECT: check whether loop exited due to meeting or due to NULL
while (fast && fast->next) {
    slow = slow->next; fast = fast->next->next;
    if (slow == fast) break;
}
if (fast == nullptr || fast->next == nullptr) return nullptr;  // no cycle
// Only reach here if slow == fast (cycle found)
slow = head;
while (slow != fast) { slow = slow->next; fast = fast->next; }
return slow;
```

---

## 13. Quick Revision Cheat Sheet

This section is fully self-contained for night-before revision.

### Middle of LL (standard — second middle for even)

```cpp
ListNode *slow = head, *fast = head;
while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
return slow;
```

### Middle of LL (left-leaning — for palindrome, split after first half)

```cpp
while (fast->next && fast->next->next) { slow = slow->next; fast = fast->next->next; }
// slow = last node of first half; second half starts at slow->next
```

### Reverse LL — Iterative

```cpp
ListNode *prev = nullptr, *curr = head;
while (curr) {
    ListNode* next = curr->next;  // save first — most common mistake
    curr->next = prev;
    prev = curr; curr = next;
}
return prev;   // new head
```

### Reverse LL — Recursive

```cpp
if (!head || !head->next) return head;
ListNode* newHead = reverseList(head->next);
head->next->next = head;   // reattach head at end of reversed sublist
head->next = nullptr;      // head is now the tail — must nullify
return newHead;
```

### Detect Cycle (Floyd's)

```cpp
ListNode *slow = head, *fast = head;
while (fast && fast->next) {
    slow = slow->next; fast = fast->next->next;
    if (slow == fast) return true;   // move THEN check
}
return false;
```

### Find Cycle Start (Floyd's + reset)

```cpp
// Step 1: find meeting point
ListNode *slow = head, *fast = head;
while (fast && fast->next) {
    slow = slow->next; fast = fast->next->next;
    if (slow == fast) break;
}
if (!fast || !fast->next) return nullptr;   // no cycle guard

// Step 2: reset slow, advance both at speed 1
slow = head;
while (slow != fast) { slow = slow->next; fast = fast->next; }
return slow;   // cycle start
```

### Length of Loop

```cpp
// After finding meeting point:
Node* curr = meetingPoint->next;
int count = 1;
while (curr != meetingPoint) { count++; curr = curr->next; }
return count;
```

### Palindrome LL (O(1) space)

```cpp
// 1. Left-leaning middle
while (fast->next && fast->next->next) { slow = slow->next; fast = fast->next->next; }
// 2. Reverse second half
ListNode* secondHalf = reverseList(slow->next); slow->next = nullptr;
// 3. Compare
ListNode *p1 = head, *p2 = secondHalf;
bool ok = true;
while (p2) { if (p1->val != p2->val) { ok = false; break; } p1 = p1->next; p2 = p2->next; }
// 4. Restore: slow->next = reverseList(secondHalf);
return ok;
```

### Odd-Even Segregation

```cpp
ListNode *odd = head, *even = head->next, *evenHead = head->next;
while (even && even->next) {
    odd->next = even->next; odd = odd->next;
    even->next = odd->next; even = even->next;
}
odd->next = evenHead;
return head;
```

### Floyd's Algorithm — Key Facts

```
Variables:  L = head to cycle start,  C = cycle length,  d = cycle start to meeting point
Key equation:  L + d = nC  →  L = nC - d  =  (n-1)C + (C-d)
Why they meet: fast gains 1 on slow per step; gap reaches 0 in finite time
Why speed 2: relative gain = 2-1 = 1; any faster gain can skip over 0
After reset: slow needs L steps from head; fast needs L steps from meeting point; arrive together
```

### Common patterns built on these primitives

| Advanced problem | Core primitives used |
|---|---|
| Reorder List (LC 143) | Middle + Reverse second half + Merge |
| Merge Sort on LL | Middle (split) + Merge |
| Intersection of Two LLs (LC 160) | Pointer synchronization (two-pass length) |
| Rotate List (LC 61) | Find new tail + cycle link + break |
| Copy List with Random Pointer (LC 138) | Weaving / pointer interleaving |
