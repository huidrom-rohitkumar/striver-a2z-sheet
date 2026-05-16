**Topic:** Linked List Structural Transformations, Arithmetic, Sorting, and Intersections

## Resources

- [Striver — Remove Nth Node from End](https://youtu.be/3kMKYQ2wNIU)
- [Striver — Delete Middle Node](https://youtu.be/ePpV-_pfOeI)
- [Striver — Sort Linked List](https://youtu.be/8ocB7a_c-Cc)
- [Striver — Sort 0s 1s 2s](https://youtu.be/gRII7LhdJWc)
- [Striver — Intersection of Two Linked Lists](https://youtu.be/XmRrGzR6udg)
- [Striver — Add 1 to Number Represented by LL](https://youtu.be/aXQWhbvT3w0)
- [Striver — Add Two Numbers](https://youtu.be/0DYoPz2Tpt4)
- [LC 19 — Remove Nth Node From End](https://leetcode.com/problems/remove-nth-node-from-end-of-list)
- [LC 2095 — Delete the Middle Node](https://leetcode.com/problems/delete-the-middle-node-of-a-linked-list)
- [LC 148 — Sort List](https://leetcode.com/problems/sort-list)
- [LC 160 — Intersection of Two Linked Lists](https://leetcode.com/problems/intersection-of-two-linked-lists)
- [LC 2 — Add Two Numbers](https://leetcode.com/problems/add-two-numbers)
- [GFG — Sort LL of 0s 1s and 2s](https://www.geeksforgeeks.org/problems/given-a-linked-list-of-0s-1s-and-2s-sort-it/1)

---

## Table of Contents

1. [The Unifying Mental Models](#1-the-unifying-mental-models)
2. [The Fast-Slow Pointer Framework](#2-the-fast-slow-pointer-framework)
3. [The Dummy Node Technique](#3-the-dummy-node-technique)
4. [Remove Nth Node From End (LC 19)](#4-remove-nth-node-from-end-lc-19)
5. [Delete the Middle Node (LC 2095)](#5-delete-the-middle-node-lc-2095)
6. [Sort Linked List (LC 148)](#6-sort-linked-list-lc-148)
7. [Sort LL of 0s, 1s and 2s — Change Links (GFG)](#7-sort-ll-of-0s-1s-and-2s--change-links-gfg)
8. [Intersection of Two Linked Lists (LC 160)](#8-intersection-of-two-linked-lists-lc-160)
9. [Add 1 to a Number Represented by LL (GFG)](#9-add-1-to-a-number-represented-by-ll-gfg)
10. [Add Two Numbers (LC 2)](#10-add-two-numbers-lc-2)
11. [Edge Cases](#11-edge-cases)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Interview Patterns and Extensions](#13-interview-patterns-and-extensions)
14. [Quick Revision Cheat Sheet](#14-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Models

Every problem in this file falls into one of five fundamental patterns. Recognising which pattern applies is the core interview skill.

**Gap pointer (fast leads by n+1 from a dummy):** When you need to reach the n-th node from the end without knowing the total length. The gap ensures slow is always one step behind the target when fast hits null. A dummy node absorbs the head-deletion case.

**Fast-slow with head start (for stopping before the middle):** When you need to act on the node *before* the middle, give fast a 2-step head start. When fast hits null, slow is one before the middle. The head start biases slow one position earlier than the standard stop.

**Merge sort on linked lists:** Linked lists have no random access, making quicksort and heapsort impractical. Merge sort is natural: split at the middle (fast-slow pointer), sort recursively, merge by relinking. O(n log n) time, O(log n) stack space. The merge step requires no auxiliary array — just pointer relinking.

**Three dummy heads for k-way partition:** When sorting or filtering into k fixed categories, create k dummy head nodes. Walk the list once, appending each node to the correct bucket by pointer manipulation. Concatenate the bucket tails. O(n) time, O(1) space. The Dutch National Flag idea at the pointer level.

**Path equalisation for intersection:** Two lists sharing a common tail. Walking list A then list B, and list B then list A, both cover total distance `a + b + c` (where c is the shared suffix length). They meet exactly at the intersection node — or both reach null simultaneously if no intersection. No length calculation needed.

**Carry propagation on lists:** Adding digits stored in nodes where the most significant digit is at the head. Carry flows right-to-left, but list traversal is left-to-right. Resolution: reverse the list (so units digit is at head), propagate carry naturally, reverse back. Alternatively, use recursion to simulate right-to-left processing via the call stack.

---

## 2. The Fast-Slow Pointer Framework

All position-finding problems in this file use a fast-slow variant. The configuration changes based on where slow must stop.

| Goal | fast initialisation | slow initialisation | Loop condition | slow stops at |
|---|---|---|---|---|
| At the middle node | `head` | `head` | `fast && fast->next` | Middle (`ceil(n/2)` index) |
| One before middle (delete middle) | `head->next->next` | `head` | `fast && fast->next` | `floor(n/2) - 1` index |
| One before n-th from end (remove nth) | `dummy`, then advance `n+1` steps | `dummy` | `fast != null` | One before n-th from end |
| One before right half (merge sort split) | `head->next` | `head` | `fast && fast->next` | Last node of left half |

**The unifying principle:** Fast moves at 2x speed. Slow stops at approximately `floor(total/2)`. Every step of head-start given to fast shifts slow one position earlier. Every step of head-start given to slow shifts slow one position later.

---

## 3. The Dummy Node Technique

A dummy (sentinel) node placed before the head eliminates special-casing for head modifications.

**Without a dummy node:** Deleting the head requires a separate code path. Any operation that needs a "node before the target" fails when the target is the head, because no such node exists.

**With a dummy node:** All deletions are uniform: `prev->next = prev->next->next`. The returned head is always `dummy.next`.

**Standard pattern:**

```cpp
ListNode dummy(0);
dummy.next = head;
// ... operate using &dummy as starting point ...
return dummy.next;
```

The dummy can be stack-allocated (as shown) or heap-allocated. Stack allocation avoids a memory leak and is cleaner.

---

## 4. Remove Nth Node From End (LC 19)

### Problem Statement

Remove the n-th node from the end of the list in one pass and return the head.

```
[1,2,3,4,5], n=2  ->  [1,2,3,5]
[1], n=1          ->  []
[1,2], n=1        ->  [1]
```

**Constraints:** `1 <= n <= length`. n is always valid.

### Why the Dummy Is Non-Negotiable

When `n == length` (the head must be deleted), slow must point to the node *before* the head. Without a dummy, there is no such node. With a dummy at position -1, slow lands on the dummy and `slow->next = slow->next->next` correctly removes the head.

### Brute Force — Two Passes

Count length L in the first pass. Navigate to node at position `L - n` in the second pass and unlink its successor.

```cpp
ListNode* removeNthFromEndBrute(ListNode* head, int n) {
    ListNode dummy(0);
    dummy.next = head;
    int length = 0;
    ListNode* curr = head;
    while (curr) { length++; curr = curr->next; }
    curr = &dummy;
    for (int i = 0; i < length - n; i++) curr = curr->next;
    curr->next = curr->next->next;
    return dummy.next;
}
```

**Time:** O(n), two passes. **Space:** O(1).

### Optimal — One-Pass Gap Pointer

**Core insight:** If fast is exactly n+1 positions ahead of slow (both starting from the dummy), then when fast becomes null, slow is at the node immediately before the target. One link skip removes the target.

**Why n+1 steps, not n:** We need slow at the node *before* the target. Starting both at the dummy and advancing fast by n+1 creates a gap of n+1 between their starting positions. When fast reaches null, slow is at position `L - n` from the dummy, which is one before the n-th node from the end.

```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy(0);
    dummy.next = head;

    ListNode* fast = &dummy;
    ListNode* slow = &dummy;

    // Create gap of n+1 between dummy-relative positions
    for (int i = 0; i <= n; i++) {
        fast = fast->next;
    }

    while (fast) {
        slow = slow->next;
        fast = fast->next;
    }

    ListNode* toDelete = slow->next;
    slow->next = slow->next->next;
    delete toDelete;

    return dummy.next;
}
```

**Time:** O(n), one pass. **Space:** O(1).

### Dry Run

`[1,2,3,4,5]`, n=2

```
dummy->1->2->3->4->5->null

Advance fast n+1=3 times from dummy:
  Step 0 (i=0): fast=1
  Step 1 (i=1): fast=2
  Step 2 (i=2): fast=3

fast=3, slow=dummy.

Synchronised walk:
  Iter 1: slow=1, fast=4
  Iter 2: slow=2, fast=5
  Iter 3: slow=3, fast=null -> stop

slow=node(3). slow->next=node(4) is target.
slow->next = node(5).

Result: 1->2->3->5  ✓
```

`[1,2]`, n=2 (remove head)

```
Advance fast 3 times from dummy:
  i=0: fast=1.  i=1: fast=2.  i=2: fast=null.

fast is null after advance. While loop body never runs.
slow = dummy. slow->next = node(1) is target.
slow->next = node(2). dummy.next = node(2).

Result: [2]  ✓
```

---

## 5. Delete the Middle Node (LC 2095)

### Problem Statement

Delete the middle node (0-indexed at `floor(n/2)`) and return the head. Return null if the list has one node.

```
[1,3,4,7,1,2,6]  ->  [1,3,4,1,2,6]   (remove node(7) at index 3)
[1,2,3,4]         ->  [1,2,4]          (remove node(3) at index 2)
[2,1]             ->  [2]              (remove node(1) at index 1)
[1]               ->  null
```

### Approach: Fast-Slow with 2-Step Head Start

**Standard fast-slow** (both start at head): slow lands AT the middle. That is wrong here — we need the node *before* the middle to perform the deletion.

**With 2-step head start for fast** (fast starts at `head->next->next`): fast is 2 steps ahead at the start. Since fast moves at 2x and slow at 1x, the 2 extra fast steps correspond to 1 extra slow step that does not occur. Slow ends up 1 position earlier — exactly one before the middle.

```cpp
ListNode* deleteMiddle(ListNode* head) {
    // Single node: the middle is the only node; delete it
    if (!head->next) return nullptr;

    ListNode* slow = head;
    ListNode* fast = head->next->next;  // 2-step head start

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }

    // slow is one before the middle
    ListNode* toDelete = slow->next;
    slow->next = slow->next->next;
    delete toDelete;

    return head;
}
```

**Time:** O(n). **Space:** O(1).

### Dry Run

`[1,3,4,7,1,2,6]` — n=7, middle at index 3 (value 7)

```
slow=node(1), fast=node(4)  [head->next->next skips 1 and 3]

Iter 1: slow=node(3), fast=node(1)  [fast: 4->7->1]
Iter 2: slow=node(4), fast=node(6)  [fast: 1->2->6]
Iter 3: fast->next=null -> stop

slow=node(4). slow->next=node(7) is middle.
slow->next = node(1).

Result: [1,3,4,1,2,6]  ✓
```

`[1,2]` — n=2, middle at index 1 (value 2)

```
head->next exists (node(2)). head->next->next = null.
fast = null initially.
While loop: fast is null -> does not execute.
slow = node(1). slow->next = node(2).
slow->next = null. Delete node(2).

Result: [1]  ✓
```

---

## 6. Sort Linked List (LC 148)

### Problem Statement

Sort a linked list in ascending order. Target: O(n log n) time.

```
[4,2,1,3]     ->  [1,2,3,4]
[-1,5,3,4,0]  ->  [-1,0,3,4,5]
```

### Why Merge Sort, Not Quicksort or Heapsort

Quicksort on linked lists requires either random-access pivot selection (impossible without traversal) or O(n) traversal per partition, degrading the algorithm. Heapsort requires building a heap structure that does not map to pointers naturally. Merge sort is canonical for lists:

- **Split at midpoint:** fast-slow pointer finds it in O(n/2).
- **Merge step:** only pointer relinking — no auxiliary array, O(n) work.
- **Space:** O(log n) recursion stack; O(1) auxiliary per level.
- **Guaranteed O(n log n):** unlike quicksort which degrades to O(n²).

### Helper: Find the Node Before the Midpoint

`getMid` returns the last node of the left half, so the right half starts at `mid->next`. We use the head-start variant (fast = `head->next`) to bias slow toward the left, preventing infinite recursion on 2-element lists.

```cpp
ListNode* getMid(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head->next;   // 1-step head start
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;   // last node of left half
}
```

### Helper: Merge Two Sorted Lists

```cpp
ListNode* mergeSorted(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (l1 && l2) {
        if (l1->val <= l2->val) { tail->next = l1; l1 = l1->next; }
        else                    { tail->next = l2; l2 = l2->next; }
        tail = tail->next;
    }
    tail->next = l1 ? l1 : l2;
    return dummy.next;
}
```

`<=` (not `<`) makes the sort stable: equal elements from the left list go first, preserving their original relative order.

### Full Merge Sort

```cpp
ListNode* sortList(ListNode* head) {
    if (!head || !head->next) return head;   // base: 0 or 1 nodes

    ListNode* mid       = getMid(head);
    ListNode* rightHead = mid->next;
    mid->next = nullptr;                     // sever into two halves

    ListNode* left  = sortList(head);
    ListNode* right = sortList(rightHead);

    return mergeSorted(left, right);
}
```

`mid->next = nullptr` before the recursive calls is mandatory. Without it, the left half still contains a pointer into the right half, causing the recursion to sort a longer list than intended.

**Time:** O(n log n). **Space:** O(log n) recursion stack.

### Dry Run

`[4,2,1,3]`

```
sortList([4,2,1,3]):
  getMid: slow=4, fast=2. Step: slow=2, fast=null (4->next->next=null). mid=node(2).
  left=[4,2], right=[1,3]. mid->next=null.

  sortList([4,2]):
    getMid: slow=4, fast=2(=head->next). fast->next=null. Stop. mid=node(4).
    left=[4], right=[2]. node(4)->next=null.
    merge([4],[2]): 2<4->take 2; drain [4]. Return [2,4].

  sortList([1,3]):
    Similarly: merge([1],[3])=[1,3].

  merge([2,4],[1,3]):
    2>1: take 1. 2<3: take 2. 4>3: take 3. drain [4].
    Return [1,2,3,4].  ✓
```

---

## 7. Sort LL of 0s, 1s and 2s — Change Links (GFG)

### Problem Statement

Sort a linked list containing only 0s, 1s, and 2s by **changing links**, not data values. (Data swapping is disallowed — the problem models scenarios where modifying node values is impossible or expensive.)

```
1->0->2->1->0->2->1  ->  0->0->1->1->1->2->2
```

### Brute Force — Count and Overwrite (Violates Constraint)

Count occurrences of each value, then overwrite node data in order. O(n) time, O(1) space, but forbidden by the link-only constraint.

### Optimal — Three Dummy Heads

**Intuition:** This is the Dutch National Flag problem at the pointer level. Instead of swapping array values to partition into three regions, we partition by pointer relinking: maintain three separate chains (one per value), walk the original list once, append each node to the correct chain, then concatenate.

```cpp
Node* segregate(Node* head) {
    Node d0(0), d1(1), d2(2);
    Node* t0 = &d0, *t1 = &d1, *t2 = &d2;

    Node* curr = head;
    while (curr) {
        Node* nxt   = curr->next;   // save before relinking
        curr->next  = nullptr;      // detach from original list
        if      (curr->data == 0) { t0->next = curr; t0 = t0->next; }
        else if (curr->data == 1) { t1->next = curr; t1 = t1->next; }
        else                      { t2->next = curr; t2 = t2->next; }
        curr = nxt;
    }

    // Concatenate: 0-chain -> 1-chain -> 2-chain
    t0->next = d1.next ? d1.next : d2.next;  // skip empty 1-chain
    t1->next = d2.next;
    // t2's last node already has next=null (from curr->next=nullptr above)

    return d0.next;
}
```

**Why `curr->next = nullptr` before appending:** When `t0->next = curr` is executed, `curr` should be the tail of the 0-chain so far. If its `next` is not nulled, the old list connections remain, corrupting the rebuilt chain.

**Why `t0->next = d1.next ? d1.next : d2.next`:** `d1` is the dummy node itself (not a real list node). `d1.next` is the first real 1-valued node. The ternary skips the 1-chain entirely if it is empty.

**Time:** O(n). **Space:** O(1) — three dummy nodes on the stack.

### Dry Run

Input: `1->0->2->1->0`

```
d0->null, d1->null, d2->null

curr=node(1,val=1): nxt=node(0). curr->next=null. t1->next=curr. t1=node(1). curr=node(0).
curr=node(0,val=0): nxt=node(2). curr->next=null. t0->next=curr. t0=node(0). curr=node(2).
curr=node(2,val=2): nxt=node(1b). curr->next=null. t2->next=curr. t2=node(2). curr=node(1b).
curr=node(1b,val=1): nxt=node(0b). curr->next=null. t1->next=curr. t1=node(1b). curr=node(0b).
curr=node(0b,val=0): nxt=null. curr->next=null. t0->next=curr. t0=node(0b). curr=null. Done.

0-chain: d0->node(0)->node(0b)->null
1-chain: d1->node(1)->node(1b)->null
2-chain: d2->node(2)->null

Connect:
  t0->next = d1.next = node(1)     [end of 0-chain -> start of 1-chain]
  t1->next = d2.next = node(2)     [end of 1-chain -> start of 2-chain]

Result: d0.next = 0->0->1->1->2  ✓
```

---

## 8. Intersection of Two Linked Lists (LC 160)

### Problem Statement

Find the node where two singly linked lists first intersect. Intersection means the same *node object* (same memory address), not same value. Return null if no intersection.

```
A: 4->1-\
          8->4->5->null       intersect at node(8)
B: 5->6->1-/
```

**Constraints:** No cycles. If they intersect, they share all nodes from the intersection point onward.

### Approach 1 — Brute Force: Nested Loops

For every node in A, scan all of B for pointer equality. **Time:** O(m·n). **Space:** O(1).

### Approach 2 — Hash Set

Walk A, inserting each node pointer. Walk B, returning the first pointer found in the set.

```cpp
ListNode* getIntersectionHash(ListNode* headA, ListNode* headB) {
    unordered_set<ListNode*> seen;
    for (ListNode* curr = headA; curr; curr = curr->next) seen.insert(curr);
    for (ListNode* curr = headB; curr; curr = curr->next)
        if (seen.count(curr)) return curr;
    return nullptr;
}
```

**Time:** O(m+n). **Space:** O(m).

### Approach 3 — Length Difference Alignment

Compute lengths. Advance the longer list's pointer by the difference. Then walk both together until pointer equality.

**Time:** O(m+n). **Space:** O(1). Requires two passes.

### Approach 4 — Optimal: Path Switching

**Mathematical insight:** Let list A have length `a + c` and list B have length `b + c`, where `c` is the length of the shared suffix (the common tail from the intersection onward).

- Pointer `pA` walks A to the end, then switches to headB. Total distance to intersection: `a + c + b` — at that point, it has consumed the unique parts of A (`a`), the shared part (`c`), and the unique prefix of B (`b`).
- Pointer `pB` walks B to the end, then switches to headA. Total distance to intersection: `b + c + a`.

Both travel `a + b + c` steps before reaching the intersection node. They arrive simultaneously.

**If no intersection:** Both reach null after `a + b + c` steps (since `c = 0`, both walk `a + b` steps total and land at null simultaneously). The loop condition `pA != pB` becomes `null != null` = false, and the loop exits returning null.

```cpp
ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
    ListNode* a = headA;
    ListNode* b = headB;

    while (a != b) {
        a = a ? a->next : headB;   // redirect to headB when A is exhausted
        b = b ? b->next : headA;   // redirect to headA when B is exhausted
    }
    return a;   // intersection node, or null if no intersection
}
```

**Time:** O(m+n). **Space:** O(1).

### Dry Run

A: 4->1->8->4->5 (length 5, intersection at node(8), index 2 of A)
B: 5->6->1->8->4->5 (length 6, intersection at node(8), index 3 of B)

Each pointer walks `5 + 6 = 11` steps total. The intersection is at distance `2 + 3 = 5` steps into the combined path for both pointers (`a=2` unique steps in A, `b=3` unique steps in B), after which both are at node(8). They meet there after step 5 of the combined traversal.

```
Step 1: a=1,    b=6
Step 2: a=8,    b=1
Step 3: a=4(A), b=8     ->  a!=b
Step 4: a=5,    b=4
Step 5: a=null->headB=5, b=5
Step 6: a=6,    b=null->headA=4
Step 7: a=1,    b=1
Step 8: a=8,    b=8     ->  a==b  -> return node(8)  ✓
```

---

## 9. Add 1 to a Number Represented by LL (GFG)

### Problem Statement

A number is stored in a linked list with the most significant digit at the head. Add 1 and return the modified list.

```
4->5->6  ->  4->5->7
9->9->9  ->  1->0->0->0
```

**Constraints:** No leading zeros except a single-node [0].

### The Core Difficulty

Carry propagates from the least significant digit (tail) toward the head. Linked list traversal moves head to tail. These directions conflict.

### Approach 1 — Brute Force: Array Copy

Copy values to a vector, add 1 from the back with carry, rebuild the list. O(n) time, O(n) space.

### Approach 2 — Reverse, Add, Reverse Back (Optimal)

Reverse the list so the units digit is at the head. Add 1 with carry propagation (now left-to-right). Reverse again to restore order.

```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr, *curr = head;
    while (curr) {
        ListNode* nxt = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nxt;
    }
    return prev;
}

ListNode* addOne(ListNode* head) {
    head = reverseList(head);

    ListNode* curr = head;
    int carry = 1;

    while (curr && carry) {
        int sum    = curr->val + carry;
        curr->val  = sum % 10;
        carry      = sum / 10;
        if (carry && !curr->next) {
            curr->next = new ListNode(0);   // will be set to 1 in next iteration
        }
        curr = curr->next;
    }

    return reverseList(head);
}
```

**Time:** O(n). **Space:** O(1).

### Approach 3 — Recursive (Implicit Right-to-Left via Call Stack)

The recursion stack processes from tail to head naturally. Each call returns the carry from its subtree.

```cpp
int addOneHelper(ListNode* curr) {
    if (!curr) return 1;                        // past the end: initial carry = 1
    int carry = addOneHelper(curr->next);
    int sum   = curr->val + carry;
    curr->val = sum % 10;
    return sum / 10;
}

ListNode* addOne(ListNode* head) {
    int carry = addOneHelper(head);
    if (carry) {
        ListNode* newHead = new ListNode(1);
        newHead->next = head;
        return newHead;
    }
    return head;
}
```

**Time:** O(n). **Space:** O(n) recursion stack. Use the reverse approach in interviews when O(1) space is required.

### Dry Run (Reverse approach)

Input: `9->9->9`

```
After reverse: 9->9->9 (palindrome)

curr=node(9), carry=1:
  sum=10. val=0. carry=1. curr->next exists. curr=node(9) (second).
curr=node(9), carry=1:
  sum=10. val=0. carry=1. curr->next exists. curr=node(9) (third).
curr=node(9), carry=1:
  sum=10. val=0. carry=1. curr->next=null -> append node(0). curr=node(0).
curr=node(0), carry=1:
  sum=1. val=1. carry=0. Stop.

Reversed result: 0->0->0->1
After re-reverse: 1->0->0->0  ✓
```

---

## 10. Add Two Numbers (LC 2)

### Problem Statement

Two non-empty linked lists represent non-negative integers with the **least significant digit at the head**. Add the two numbers and return the sum as a linked list in the same format.

```
l1=[2,4,3] (342) + l2=[5,6,4] (465)  ->  [7,0,8] (807)
l1=[9,9,9,9,9,9,9] + l2=[9,9,9,9]   ->  [8,9,9,9,0,0,0,1]
```

### Key Advantage of Reverse Storage

The list already stores digits with least significant first — exactly the order needed for grade-school addition. No reversal is needed, unlike the "Add 1" problem where the list is stored most-significant-first.

### Algorithm

Walk both lists simultaneously, computing digit sums with carry. Continue until both lists are exhausted **and** carry is zero.

```cpp
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;
    int carry = 0;

    while (l1 || l2 || carry) {
        int sum = carry;
        if (l1) { sum += l1->val; l1 = l1->next; }
        if (l2) { sum += l2->val; l2 = l2->next; }

        carry      = sum / 10;
        tail->next = new ListNode(sum % 10);
        tail       = tail->next;
    }

    return dummy.next;
}
```

**Why `while (l1 || l2 || carry)`:** After both lists are exhausted, a remaining carry (e.g., from l1=[5], l2=[5] giving carry=1) must generate one more digit node. Including `carry` in the condition handles this without special-casing.

**Why the dummy node:** We always append to `tail->next` and advance `tail`. The first appended node becomes `dummy.next`. No special handling for the first node.

**Time:** O(max(m, n)) — each digit is processed once, plus at most one extra for carry.
**Space:** O(max(m, n) + 1) for the result list.

### Dry Run

`l1=[2,4,3]`, `l2=[5,6,4]`:

```
carry=0.

Step 1: sum=0+2+5=7.   carry=0. node(7). l1->4, l2->6.
Step 2: sum=0+4+6=10.  carry=1. node(0). l1->3, l2->4.
Step 3: sum=1+3+4=8.   carry=0. node(8). l1=null, l2=null.
l1=null, l2=null, carry=0 -> exit.

Result: 7->0->8  (807 = 342+465)  ✓
```

`l1=[9,9,9]`, `l2=[9,9,9]`:

```
Step 1: 9+9+0=18. carry=1. node(8).
Step 2: 9+9+1=19. carry=1. node(9).
Step 3: 9+9+1=19. carry=1. node(9).
Step 4: l1=null, l2=null, carry=1. sum=1. carry=0. node(1).

Result: 8->9->9->1  (1998 = 999+999)  ✓
```

---

## 11. Edge Cases

| Problem | Input | Expected | Note |
|---|---|---|---|
| Remove Nth from End | n = list length | Head removed | dummy absorbs; slow stays at dummy |
| Remove Nth from End | Single node, n=1 | Empty list | dummy.next becomes null |
| Delete Middle | Single node | null | Guard: `!head->next -> return nullptr` |
| Delete Middle | Two nodes [1,2] | [1] | fast=null after head start; slow=node(1); delete node(2) |
| Sort LL | Already sorted | Order unchanged | getMid still splits; merge is identity |
| Sort LL | Empty or single node | Unchanged | Base case: `!head || !head->next -> return head` |
| Sort 0s1s2s | All same value | Same structure, no reordering | Two of the three bucket chains remain empty |
| Sort 0s1s2s | No 0s in list | 1-chain connected to 2-chain | d0.next=null (not returned); `d1.next ? d1.next : d2.next` handles it |
| Intersection | No intersection | null | Both pointers reach null simultaneously |
| Intersection | Lists share same head | head | First comparison: a==b immediately, return a |
| Add 1 to LL | All 9s | Extra leading node(1) | Carry propagates entire length; new head prepended |
| Add 1 to LL | [0] | [1] | 0+1=1, carry=0 |
| Add Two Numbers | Different lengths | Handles via null-checks | `if (l1) sum += l1->val` is safe when l1 is null |
| Add Two Numbers | Both [9,...,9] | Extra digit at tail | `carry` in loop condition generates final node |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1: Advancing fast by n steps (not n+1) in Remove Nth from End

```cpp
// WRONG: fast is n steps ahead of slow. When fast=null, slow is AT the target,
// not one before it. slow->next is out of the subarray we want to modify.
for (int i = 0; i < n; i++) fast = fast->next;

// CORRECT: n+1 steps from dummy; slow lands one before the target
for (int i = 0; i <= n; i++) fast = fast->next;
```

### Mistake 2: Not using a dummy node in Remove Nth from End

```cpp
// WRONG: when n == list length, target is the head. slow would be null
// (one before the head does not exist), causing segfault.
ListNode* slow = head, *fast = head;
// ... slow->next = slow->next->next;  -> segfault if slow is null

// CORRECT: dummy makes slow start one before the head
ListNode dummy(0); dummy.next = head;
ListNode* slow = &dummy, *fast = &dummy;
```

### Mistake 3: Wrong fast starting position in Delete Middle

```cpp
// WRONG: standard fast=head. slow lands AT the middle. No pointer to the previous node.
ListNode* fast = head;   // standard start — slow stops too late

// CORRECT: 2-step head start for fast; slow stops one before middle
ListNode* fast = head->next->next;
```

### Mistake 4: Accessing head->next->next without guarding for single-node list

```cpp
// WRONG: if head->next is null (single node), head->next->next is a segfault
ListNode* fast = head->next->next;

// CORRECT: guard before this line
if (!head->next) return nullptr;
ListNode* fast = head->next->next;
```

### Mistake 5: Not nulling mid->next before recursive calls in Sort LL

```cpp
// WRONG: left half still points into the right half; recursion sorts an
// oversized sublist and may loop indefinitely
ListNode* rightHead = mid->next;
// Missing: mid->next = nullptr;
ListNode* left = sortList(head);  // head's list still extends through rightHead

// CORRECT:
ListNode* rightHead = mid->next;
mid->next = nullptr;              // sever before recursing
ListNode* left  = sortList(head);
ListNode* right = sortList(rightHead);
```

### Mistake 6: Not nulling curr->next before appending in Sort 0s1s2s

```cpp
// WRONG: curr->next still points into the original list. The bucket chain
// becomes corrupted because the appended node drags its old successor with it.
if (curr->data == 0) { t0->next = curr; t0 = t0->next; }

// CORRECT: detach before appending
curr->next = nullptr;
if (curr->data == 0) { t0->next = curr; t0 = t0->next; }
```

### Mistake 7: Not connecting all three sublists in Sort 0s1s2s

```cpp
// WRONG: 2-chain is disconnected; result is only 0s and 1s
t0->next = d1.next;
// Missing: t1->next = d2.next;

// CORRECT:
t0->next = d1.next ? d1.next : d2.next;  // handle empty 1-chain
t1->next = d2.next;
// t2's last node's next is already null from the curr->next=nullptr step
```

### Mistake 8: Null-dereference in intersection pointer switching

```cpp
// WRONG: a->next crashes when a is already null
a = a->next;
if (!a) a = headB;   // too late

// CORRECT: check before advancing
a = a ? a->next : headB;
```

### Mistake 9: Not including carry in the Add Two Numbers loop condition

```cpp
// WRONG: exits loop when both lists are null but carry remains
while (l1 || l2) { ... }
// carry=1 after exhausting both -> last digit is lost

// CORRECT:
while (l1 || l2 || carry) { ... }
```

### Mistake 10: Confusing node address equality with value equality in intersection

The intersection problem compares pointers (node addresses), not values. Two nodes with the same value at the same position are not an intersection unless they are the same node object in memory.

```cpp
// WRONG: compares values
while (a->val != b->val) { ... }

// CORRECT: compares addresses
while (a != b) { ... }
```

---

## 13. Interview Patterns and Extensions

### Pattern Recognition

| Signal in problem | Pattern to apply |
|---|---|
| "n-th from end" without known length, one pass | Gap pointer: fast leads by n+1 from dummy |
| "Delete middle" / "stop one before middle" | Fast-slow with 2-step head start for fast |
| "Sort linked list in O(n log n)" | Merge sort: getMid + sever + recurse + merge |
| "Sort by category without changing values" | Three dummy heads, append by category, concatenate |
| "Find common node of two lists" | Path-switching two pointer |
| "Right-to-left carry on left-to-right list" | Reverse → operate → reverse back |
| "Add digits with carry, lists may differ in length" | Simultaneous walk; `while (l1 || l2 || carry)` |

### Extensions and Harder Problems

**Remove Nth from End:**
- LC 61 (Rotate List): use the gap pointer to find the new tail (L-k-th node from start).
- GFG variant: delete k-th from middle — combine midpoint and gap logic.

**Delete Middle:**
- LC 876 (Middle of Linked List): standard fast-slow from head; return slow.

**Sort Linked List:**
- Bottom-up merge sort (O(1) space, no recursion): merge in place with step sizes 1, 2, 4, ..., n/2. Eliminates the O(log n) stack space but is significantly harder to implement.
- LC 23 (Merge K Sorted Lists): extend mergeSorted to k lists via a min-heap, or divide-and-conquer pairwise merging.

**Sort 0s1s2s:**
- LC 75 (Sort Colors) for arrays: Dutch National Flag with three-way in-place swapping.
- The three-dummy-head trick generalises to any k-way partitioning of a linked list by category.
- LC 86 (Partition List): two dummy heads (< pivot and >= pivot), same pattern.

**Intersection:**
- LC 141/142 (Cycle Detection/Entry): Floyd's algorithm — fast=2x, slow=1x, different problem (detect cycle) but same speed structure.
- With modifiable lists: if you can mark visited nodes, mark all of one list, then scan the other.

**Add Two Numbers:**
- LC 445 (Add Two Numbers II): digits stored most-significant first. Use two stacks to reverse order, add, build result. Or reverse both lists, use this problem's solution, reverse result.
- LC 67 (Add Binary): same carry logic on string inputs; same conceptual structure.
- Add k to a LL number: initialise carry = k in the "Add 1" approach.

---

## 14. Quick Revision Cheat Sheet

**Remove Nth Node From End — One Pass**
```
dummy->list. fast = slow = dummy.
Advance fast (n+1) times from dummy.
While fast: slow = slow->next; fast = fast->next.
slow->next = slow->next->next.
Return dummy.next.
```
O(n) time, O(1) space. Dummy absorbs head-deletion case.

---

**Delete Middle Node — Head-Start Fast**
```
if (!head->next) return null.
slow = head. fast = head->next->next.  [2-step head start]
While fast && fast->next: slow++; fast+=2.
slow->next = slow->next->next.
Return head.
```
O(n) time, O(1) space. Head start stops slow one before middle.

---

**Sort Linked List — Merge Sort**
```
getMid(head): slow=head, fast=head->next.
  While fast && fast->next: slow++; fast+=2. Return slow.

sortList(head):
  if !head || !head->next: return head.
  mid = getMid(head). right = mid->next. mid->next = null.
  return merge(sortList(head), sortList(right)).

merge(l1, l2): dummy; tail=dummy.
  While l1 && l2: take smaller; advance. tail->next = remaining.
  return dummy.next.
```
O(n log n) time, O(log n) stack space.

---

**Sort 0s1s2s — Three Dummy Heads**
```
d0, d1, d2 = dummy nodes. t0, t1, t2 = their tails.
For each curr: nxt=curr->next; curr->next=null; append to correct tail; curr=nxt.
t0->next = d1.next ?: d2.next.
t1->next = d2.next.
Return d0.next.
```
O(n) time, O(1) space. `curr->next=null` before appending is mandatory.

---

**Intersection — Path Switching**
```
a = headA. b = headB.
While a != b: a = a ? a->next : headB; b = b ? b->next : headA.
Return a.
```
O(m+n) time, O(1) space. Compares node addresses, not values. No intersection: both reach null simultaneously.

---

**Add 1 to LL — Reverse Approach**
```
reverse list. curr=head, carry=1.
While curr && carry:
  sum=curr->val+carry. curr->val=sum%10. carry=sum/10.
  if carry && !curr->next: append node(0).
  curr=curr->next.
reverse list again. Return head.
```
O(n) time, O(1) space.

---

**Add Two Numbers**
```
dummy. tail=dummy. carry=0.
While l1 || l2 || carry:
  sum = carry + (l1?l1->val:0) + (l2?l2->val:0).
  carry = sum/10. tail->next = node(sum%10). tail++.
  if l1: l1++. if l2: l2++.
Return dummy.next.
```
O(max(m,n)) time, O(max(m,n)) space.

---

**Fast-Slow Pointer Configurations**

| Need slow to stop at | fast init | Loop |
|---|---|---|
| Middle node | `head` | `fast && fast->next` |
| One before middle | `head->next->next` | `fast && fast->next` |
| Last of left half (sort split) | `head->next` | `fast && fast->next` |
| One before n-th from end | `dummy`, advance `n+1` steps | `fast != null` |

---

**Complexity Reference**

| Problem | Time | Space |
|---|---|---|
| Remove Nth from End | O(n) | O(1) |
| Delete Middle Node | O(n) | O(1) |
| Sort Linked List | O(n log n) | O(log n) |
| Sort 0s1s2s | O(n) | O(1) |
| Intersection (path switching) | O(m+n) | O(1) |
| Add 1 to LL (reverse) | O(n) | O(1) |
| Add Two Numbers | O(max(m,n)) | O(max(m,n)) |
