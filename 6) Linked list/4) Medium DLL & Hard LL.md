**Topic:** Advanced Linked List Patterns — DLL Operations, Reverse K-Group, Rotate, Flatten, Copy Random
**Patterns:** Multi-pointer Rewiring, DLL Two-Pointer, K-Group Reversal, Circular Conversion, Recursive Merge, Structural Cloning

**Resources:**
[GFG Delete DLL](https://www.geeksforgeeks.org/problems/delete-all-occurrences-of-a-given-key-in-a-doubly-linked-list/1) |
[GFG Pairs DLL](https://www.geeksforgeeks.org/problems/find-pairs-with-given-sum-in-doubly-linked-list/1) |
[GFG Remove Dups DLL](https://www.geeksforgeeks.org/problems/remove-duplicates-from-a-sorted-doubly-linked-list/1) |
[LC 25 Reverse K-Group](https://leetcode.com/problems/reverse-nodes-in-k-group) |
[LC 61 Rotate List](https://leetcode.com/problems/rotate-list) |
[GFG Flatten LL](https://www.geeksforgeeks.org/problems/flattening-a-linked-list/1) |
[LC 138 Copy Random](https://leetcode.com/problems/copy-list-with-random-pointer) |
[Striver DLL Ops](https://youtu.be/Mh0NH_SD92k) |
[Striver K-Group](https://youtu.be/YitR4dQsddE) |
[Striver Rotate](https://youtu.be/YJKVTnOJXSY) |
[Striver Flatten](https://youtu.be/lIar1skcQYI) |
[Striver Copy Random](https://youtu.be/uT7YI7XbTY8)

---

## Table of Contents

1. [Structural Overview — What Makes These Problems Hard](#1-structural-overview--what-makes-these-problems-hard)
2. [DLL Pointer Manipulation — The Universal Template](#2-dll-pointer-manipulation--the-universal-template)
3. [Problem 1 — Delete All Occurrences of a Key in DLL (GFG)](#3-problem-1--delete-all-occurrences-of-a-key-in-dll-gfg)
4. [Problem 2 — Find Pairs with Given Sum in DLL (GFG)](#4-problem-2--find-pairs-with-given-sum-in-dll-gfg)
5. [Problem 3 — Remove Duplicates from Sorted DLL (GFG)](#5-problem-3--remove-duplicates-from-sorted-dll-gfg)
6. [Problem 4 — Reverse Nodes in K-Group (LC 25)](#6-problem-4--reverse-nodes-in-k-group-lc-25)
7. [Problem 5 — Rotate List (LC 61)](#7-problem-5--rotate-list-lc-61)
8. [Problem 6 — Flattening a Linked List (GFG)](#8-problem-6--flattening-a-linked-list-gfg)
9. [Problem 7 — Copy List with Random Pointer (LC 138)](#9-problem-7--copy-list-with-random-pointer-lc-138)
10. [Complexity Cheat Sheet](#10-complexity-cheat-sheet)
11. [Edge Cases Master Table](#11-edge-cases-master-table)
12. [Common Mistakes and Pitfalls](#12-common-mistakes-and-pitfalls)
13. [Interview Reference — Patterns and Extensions](#13-interview-reference--patterns-and-extensions)
14. [Quick Revision Cheat Sheet](#14-quick-revision-cheat-sheet)

---

## 1. Structural Overview — What Makes These Problems Hard

These problems do not test whether you can traverse a list. They test whether you can reason about pointer state across multiple simultaneous modifications. The typical failure mode is not a wrong algorithm — it is a correct algorithm executed in the wrong order, causing a node's address to be lost, a cycle to be created, or a pointer to be dereferenced after the memory it addresses has been relinked.

The problems in this set use six distinct structural patterns:

| Pattern | Problem | Core action |
|---|---|---|
| DLL bidirectional relinking | Delete DLL, Pairs DLL, Remove Dups DLL | Fix both `prev` and `next` on every structural change |
| Two-pointer inward scan on sorted DLL | Find Pairs | `prev` pointer enables O(1) backward traversal |
| K-group boundary detection + reversal | LC 25 | Find kth node, disconnect group, reverse, reconnect |
| Circular list + break | LC 61 Rotate | Make circular, normalize k, find new tail, break |
| Recursive sorted merge via `bottom` pointers | GFG Flatten | Process right-to-left; merge two sorted `bottom`-lists |
| Identity-preserving structural copy | LC 138 Copy Random | Hash map (O(n) space) or interweaving (O(1) space) |

The rule that prevents most bugs: **save every pointer you will need after a modification before you make the modification**.

---

## 2. DLL Pointer Manipulation — The Universal Template

Every DLL deletion follows the same four-step order. The order matters because each step assumes the addresses from previous steps are still valid.

```cpp
// Universal DLL node deletion of curr:
Node* nextNode = curr->next;              // 1. save next BEFORE any modification
if (curr->prev) curr->prev->next = curr->next;  // 2. bypass curr from the left
if (curr->next) curr->next->prev = curr->prev;  // 3. bypass curr from the right
if (curr == head) head = curr->next;            // 4. update head if needed
delete curr;                                     // 5. free memory
curr = nextNode;                                 // 6. advance to saved next
```

Skipping the null checks on step 2 or 3 causes segfaults on head and tail nodes respectively. Deleting before saving `nextNode` causes undefined behavior on step 6. The order is non-negotiable.

---

## 3. Problem 1 — Delete All Occurrences of a Key in DLL (GFG)

**Problem:** Given a doubly linked list and a key `x`, delete all nodes with value equal to `x`. Return the (possibly new) head.
**Link:** [GFG](https://www.geeksforgeeks.org/problems/delete-all-occurrences-of-a-given-key-in-a-doubly-linked-list/1)
**Constraints:** `1 <= nodes <= 10^5`, `1 <= data <= 10^9`.

**Examples:**
- `1 <-> 2 <-> 3 <-> 2 <-> 4`, key=2 → `1 <-> 3 <-> 4`
- `2 <-> 2 <-> 2`, key=2 → empty list

### Approach — Single Pass with Universal Template

Apply the deletion template from Section 2 at every matching node.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    int data;
    Node* prev;
    Node* next;
    Node(int x) : data(x), prev(nullptr), next(nullptr) {}
};

Node* deleteAllOccurrences(Node* head, int key) {
    Node* curr = head;
    while (curr != nullptr) {
        Node* nextNode = curr->next;      // save before possible deletion
        if (curr->data == key) {
            if (curr->prev) curr->prev->next = curr->next;
            if (curr->next) curr->next->prev = curr->prev;
            if (curr == head) head = curr->next;
            delete curr;
        }
        curr = nextNode;
    }
    return head;
}
```

**Dry run:** `1 <-> 2 <-> 3 <-> 2 <-> 4`, key=2

```
curr=1: 1!=2, curr=2(first)
curr=2(first): match. next=3.
  1->next=3; 3->prev=1; head unchanged.
  delete 2. curr=3.
curr=3: 3!=2, curr=2(second)
curr=2(second): match. next=4.
  3->next=4; 4->prev=3.
  delete 2. curr=4.
curr=4: 4!=2, curr=null.

Result: 1 <-> 3 <-> 4  ✓
```

| Scenario | Expected |
|---|---|
| Head matches | `head = curr->next` triggers |
| All match | Returns `nullptr` |
| Key absent | List unchanged |
| Single matching node | Returns `nullptr` |

Time: O(n). Space: O(1).

---

## 4. Problem 2 — Find Pairs with Given Sum in DLL (GFG)

**Problem:** Given a **sorted** doubly linked list and a target sum, find all pairs of nodes whose values sum to the target. Report each pair once.
**Link:** [GFG](https://www.geeksforgeeks.org/problems/find-pairs-with-given-sum-in-doubly-linked-list/1)
**Constraints:** All values distinct. List sorted non-decreasing.

**Examples:**
- `1 <-> 2 <-> 4 <-> 5 <-> 6 <-> 8 <-> 9`, target=7 → `[(1,6),(2,5)]`
- `1 <-> 2 <-> 3`, target=10 → `[]`

### Why a Sorted DLL Enables Two-Pointer

In a sorted DLL, `head` holds the minimum and `tail` holds the maximum. The `prev` pointer enables O(1) backward movement from the tail, which is impossible on a singly linked list. This gives the same two-pointer inward scan used on sorted arrays.

### Approach — Two-Pointer Inward Scan

```cpp
vector<pair<int,int>> findPairsWithSum(Node* head, int target) {
    if (!head) return {};

    // Find tail
    Node* right = head;
    while (right->next) right = right->next;

    Node* left = head;
    vector<pair<int,int>> result;

    while (left != right && right->next != left) {
        int sum = left->data + right->data;
        if (sum == target) {
            result.push_back({left->data, right->data});
            left = left->next;
            right = right->prev;
        } else if (sum < target) {
            left = left->next;
        } else {
            right = right->prev;
        }
    }
    return result;
}
```

**Termination — why both conditions are needed:**

- `left != right`: handles odd-length lists. After the midpoint, both pointers land on the same node. A single node cannot form a valid pair.
- `right->next != left`: handles even-length lists. After the pointers cross, `right` is just behind `left`, so `right->next == left`. If we only checked `left != right`, we would continue past the crossing point, reporting invalid pairs.

**Dry run:** `1 <-> 2 <-> 4 <-> 5 <-> 6 <-> 8 <-> 9`, target=7

```
left=1, right=9
1+9=10>7 → right=8
1+8=9>7  → right=6
1+6=7    → pair(1,6). left=2, right=5.
2+5=7    → pair(2,5). left=4, right=4.
left==right → stop.

Result: [(1,6),(2,5)]  ✓
```

Time: O(n). Space: O(1) auxiliary (output excluded).

---

## 5. Problem 3 — Remove Duplicates from Sorted DLL (GFG)

**Problem:** Given a sorted DLL, remove all duplicate nodes so each value appears at most once. Return the head.
**Link:** [GFG](https://www.geeksforgeeks.org/problems/remove-duplicates-from-a-sorted-doubly-linked-list/1)

**Example:** `1 <-> 1 <-> 2 <-> 3 <-> 3` → `1 <-> 2 <-> 3`

### Approach — Forward Scan, Delete Next on Match

Because the list is sorted, all duplicates of any value are adjacent. Walk `curr` forward; when `curr->data == curr->next->data`, delete `curr->next` in place. Do not advance `curr` after a deletion — the new `curr->next` may also be a duplicate.

```cpp
Node* removeDuplicates(Node* head) {
    Node* curr = head;
    while (curr && curr->next) {
        if (curr->data == curr->next->data) {
            Node* dup = curr->next;
            curr->next = dup->next;
            if (dup->next) dup->next->prev = curr;
            delete dup;
            // Do NOT advance curr — next node may also be a duplicate
        } else {
            curr = curr->next;
        }
    }
    return head;
}
```

**Dry run:** `1 <-> 1 <-> 2 <-> 3 <-> 3`

```
curr=1: next=1(dup). Equal. dup=1(second).
  curr->next=2; 2->prev=1. delete. [1<->2<->3<->3]
curr=1: next=2. Not equal. curr=2.
curr=2: next=3. Not equal. curr=3.
curr=3: next=3(dup). Equal. dup=3(second).
  curr->next=null. delete. [1<->2<->3]
curr=3: next=null. Exit.

Result: 1 <-> 2 <-> 3  ✓
```

Time: O(n). Space: O(1).

---

## 6. Problem 4 — Reverse Nodes in K-Group (LC 25)

**Problem:** Reverse the nodes of a linked list k at a time. If fewer than k nodes remain at the end, leave them as-is. Must only modify nodes, not values.
**Link:** [LC 25](https://leetcode.com/problems/reverse-nodes-in-k-group)
**Constraints:** `1 <= k <= n <= 5000`, values in `[0,1000]`.

**Examples:**
- `[1,2,3,4,5]`, k=2 → `[2,1,4,3,5]`
- `[1,2,3,4,5]`, k=3 → `[3,2,1,4,5]`
- `[1,2,3,4,5]`, k=1 → `[1,2,3,4,5]`

### Mental Model

View the list as groups of k. For each group:
1. Check if k nodes exist (if not, leave the tail unchanged).
2. Reverse the k-node segment.
3. Connect: the previous group's tail must point to this group's new head; this group's new tail must point to the next group.

The key reconnection insight for the iterative version: start the reversal with `prev = groupNext` (the first node of the **next** group, not `nullptr`). This automatically wires the reversed group's tail to the next group without a separate step.

### Approach 1 — Recursive

```cpp
ListNode* reverseKGroup(ListNode* head, int k) {
    // Check if k nodes exist
    ListNode* check = head;
    int count = 0;
    while (check && count < k) { check = check->next; count++; }
    if (count < k) return head;   // fewer than k: leave as-is

    // Reverse k nodes
    ListNode* prev = nullptr, *curr = head;
    for (int i = 0; i < k; i++) {
        ListNode* nxt = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nxt;
    }
    // prev = new head of reversed group
    // head = last node of reversed group
    // curr = start of next group
    head->next = reverseKGroup(curr, k);
    return prev;
}
```

Time: O(n). Space: O(n/k) call stack.

### Approach 2 — Iterative with Dummy Node (Optimal, O(1) Space)

```cpp
// Returns kth node from start (nullptr if fewer than k nodes remain)
ListNode* getKth(ListNode* node, int k) {
    while (node && k > 0) { node = node->next; k--; }
    return node;
}

ListNode* reverseKGroup(ListNode* head, int k) {
    if (!head || k == 1) return head;

    ListNode dummy(0);
    dummy.next = head;
    ListNode* groupPrev = &dummy;

    while (true) {
        ListNode* kth = getKth(groupPrev, k);
        if (!kth) break;                          // fewer than k nodes left

        ListNode* groupNext = kth->next;

        // Reverse [groupPrev->next .. kth] with prev initialized to groupNext.
        // Starting prev at groupNext means after reversal the last node of
        // the reversed group automatically points to groupNext — no extra step.
        ListNode* prev = groupNext;
        ListNode* curr = groupPrev->next;
        while (curr != groupNext) {
            ListNode* nxt = curr->next;
            curr->next = prev;
            prev = curr;
            curr = nxt;
        }
        // prev = kth (new head of reversed group)
        // groupPrev->next (original first) = new tail of reversed group

        ListNode* tmp = groupPrev->next;   // save old first (now tail of group)
        groupPrev->next = prev;            // connect previous group to new head
        groupPrev = tmp;                   // advance groupPrev to end of this group
    }
    return dummy.next;
}
```

**Why `prev = groupNext` as the initial value for reversal:**

Reversing `[A -> B -> C]` (k=3) whose next group starts at `D`:
- With `prev=nullptr`: after reversal → `C -> B -> A -> nullptr`. Link to D is missing.
- With `prev=D` (`groupNext`): after reversal → `C -> B -> A -> D`. Link is established automatically.

This eliminates the need for a post-reversal reconnection step.

**Dry run:** `[1,2,3,4,5]`, k=2

```
dummy->1->2->3->4->5

Iter 1:
  groupPrev=dummy
  getKth(dummy, 2) = node 2  (dummy->1->2; advance 2)
  kth=2, groupNext=3
  Reverse [1->2] with prev=3:
    curr=1: nxt=2. 1->3. prev=1. curr=2.
    curr=2: nxt=3=groupNext. 2->1. prev=2. curr=3. stop.
  tmp = dummy->next = 1 (old first, now tail)
  dummy->next = 2 (new head)
  groupPrev = 1
  List: dummy->2->1->3->4->5

Iter 2:
  groupPrev=1
  getKth(1, 2) = node 4  (1->3->4; advance 2)
  kth=4, groupNext=5
  Reverse [3->4] with prev=5:
    curr=3: 3->5. prev=3. curr=4.
    curr=4: 4->3. prev=4. curr=5. stop.
  tmp = 3; 1->next=4; groupPrev=3
  List: dummy->2->1->4->3->5

Iter 3:
  getKth(3, 2): 3->5->null; null returned. Break.

Return: 2->1->4->3->5  ✓
```

Time: O(n). Space: O(1).

---

## 7. Problem 5 — Rotate List (LC 61)

**Problem:** Given the head of a linked list, rotate the list to the right by k places. Rotating right by 1 means the last element moves to the front.
**Link:** [LC 61](https://leetcode.com/problems/rotate-list)
**Constraints:** `0 <= k <= 2 * 10^9`, `0 <= n <= 500`.

**Examples:**
- `[1,2,3,4,5]`, k=2 → `[4,5,1,2,3]`
- `[0,1,2]`, k=4 → `[2,0,1]`

### Key Observations

1. Rotating by n (the list length) is a no-op, so the effective rotation is `k % n`.
2. After rotating right by k, the new head is the node at 0-indexed position `n - k`. The node just before it (position `n - k - 1`) becomes the new tail.
3. Making the list circular first and then breaking at the right position is cleaner than tracking two disconnected halves.

### Approach — Circular + Break

```cpp
ListNode* rotateRight(ListNode* head, int k) {
    if (!head || !head->next || k == 0) return head;

    // Step 1: find length and tail
    int length = 1;
    ListNode* tail = head;
    while (tail->next) { tail = tail->next; length++; }

    // Step 2: normalize k
    k = k % length;
    if (k == 0) return head;

    // Step 3: make circular
    tail->next = head;

    // Step 4: find new tail at position (length - k - 1) from head
    int stepsToNewTail = length - k - 1;
    ListNode* newTail = head;
    for (int i = 0; i < stepsToNewTail; i++) newTail = newTail->next;

    // Step 5: break circle
    ListNode* newHead = newTail->next;
    newTail->next = nullptr;
    return newHead;
}
```

**Why `stepsToNewTail = length - k - 1`:**

0-indexed positions: `0, 1, ..., n-1`. After rotating right by k, position `n-k` becomes the new head. The new tail is the node just before it: position `n-k-1`. Starting from position 0 (head), take `n-k-1` steps.

**Verification:** `[1,2,3,4,5]`, k=2: `length=5`, `stepsToNewTail=5-2-1=2`. Walk 2 from head: 1→2→3. `newTail=3`, `newHead=4`. Result: `4->5->1->2->3`. ✓

**Dry run — k larger than length:** `[1,2,3,4,5]`, k=7

```
length=5, k=7%5=2 (same as above)
tail=5, 5->next=1 (circular)
stepsToNewTail=2. Walk: 1->2->3. newTail=3, newHead=4.
3->next=null.
Result: 4->5->1->2->3  ✓
```

**Dry run — k is multiple of length:** `[1,2,3]`, k=3

```
length=3, k=3%3=0. Return head unchanged.
Result: 1->2->3  ✓
```

Time: O(n). Space: O(1).

---

## 8. Problem 6 — Flattening a Linked List (GFG)

**Problem:** A linked list where each node has a `next` pointer (horizontal) and a `bottom` pointer to another sorted linked list (vertical). All vertical lists are sorted. Flatten the entire structure into one sorted list using `bottom` pointers (set all `next` pointers to null).
**Link:** [GFG](https://www.geeksforgeeks.org/problems/flattening-a-linked-list/1)

```
Input:
5 -> 10 -> 19 -> 28
|    |      |     |
7    20     22   35
|           |
8          50
|
30

Output via bottom: 5->7->8->10->19->20->22->28->30->35->50
```

**Constraints:** `1 <= n <= 100` horizontal nodes, each vertical list up to 20 nodes.

### Node Structure

```cpp
struct Node {
    int data;
    Node* next;    // horizontal: next top-level node
    Node* bottom;  // vertical: sub-list below this node
};
```

### Core Insight: N Sorted Lists to Merge

The brute force (collect all values, sort, rebuild) costs O(NM log(NM)) and ignores the pre-sorted property of each vertical list. The optimal approach uses the merge step of merge sort: two sorted `bottom`-pointer lists can be merged in O(size) time. Flatten recursively from right to left, then merge the current node's vertical list with the already-flattened right portion.

### Approach 1 — Brute Force

```cpp
Node* flattenBrute(Node* head) {
    vector<int> vals;
    Node* curr = head;
    while (curr) {
        Node* bot = curr;
        while (bot) { vals.push_back(bot->data); bot = bot->bottom; }
        curr = curr->next;
    }
    sort(vals.begin(), vals.end());
    Node* newHead = new Node(vals[0]);
    Node* ptr = newHead;
    for (int i = 1; i < (int)vals.size(); i++) { ptr->bottom = new Node(vals[i]); ptr = ptr->bottom; }
    return newHead;
}
```

Time: O(NM log(NM)). Space: O(NM).

### Approach 2 — Recursive Merge (Optimal)

```cpp
// Merge two sorted lists connected via bottom pointers
Node* mergeSorted(Node* a, Node* b) {
    if (!a) return b;
    if (!b) return a;
    Node* result;
    if (a->data <= b->data) {
        result = a;
        result->bottom = mergeSorted(a->bottom, b);
    } else {
        result = b;
        result->bottom = mergeSorted(a, b->bottom);
    }
    result->next = nullptr;   // clear horizontal pointer
    return result;
}

Node* flatten(Node* root) {
    if (!root || !root->next) return root;
    root->next = flatten(root->next);     // flatten the right portion first
    root = mergeSorted(root, root->next); // merge current with flattened right
    return root;
}
```

**Why process right-to-left (recurse before merge):**

If we processed left-to-right, the first merge would be between `root` and `root->next` — but `root->next` still points to the un-flattened remainder. By recursing first, by the time we call `mergeSorted`, `root->next` already points to the fully flattened right portion. The merge then correctly produces a single sorted list.

**Why `result->next = nullptr`:** The merge traverses `bottom` pointers exclusively. The `next` pointers on the input nodes were part of the original horizontal structure and are stale after flattening. Clearing them ensures the output is a clean `bottom`-pointer list with no dangling `next` references.

**Dry run (simplified):** Horizontal: `5 -> 10 -> 19`. Verticals: `[5,7,8]`, `[10,20]`, `[19,22]`

```
flatten(5):
  flatten(10):
    flatten(19):
      19->next=null → base case, return 19(head of [19,22])
    merge([10,20], [19,22]):
      10<19 → result=10; 10->bottom=merge([20],[19,22])
        20>19 → result=19; 19->bottom=merge([20],[22])
          20<22 → result=20; 20->bottom=22. return 20
        return 19->20->22
      return 10->19->20->22
    root->next = 10->19->20->22; return 10->19->20->22

  merge([5,7,8], [10,19,20,22]):
    5<10 → result=5; 5->bottom=merge([7,8],[10,19,20,22])
      7<10 → result=7; 7->bottom=merge([8],[10,...])
        8<10 → result=8; 8->bottom=10->19->20->22
      return 7->8->10->19->20->22
    return 5->7->8->10->19->20->22  ✓
```

Time: O(N×M) — each node involved in at most one merge per recursion level, N levels. Space: O(N) recursion stack for `flatten` + O(depth) for `mergeSorted` recursion.

---

## 9. Problem 7 — Copy List with Random Pointer (LC 138)

**Problem:** Each node in a singly linked list has a `val`, a `next` pointer, and a `random` pointer (any node in the list, or null). Return a **deep copy** — completely new nodes with the same values, where `next` and `random` of new nodes mirror the original structure.
**Link:** [LC 138](https://leetcode.com/problems/copy-list-with-random-pointer)
**Constraints:** `0 <= n <= 1000`, `-10^4 <= val <= 10^4`.

```cpp
struct Node {
    int val;
    Node* next;
    Node* random;
};
```

**The core challenge:** `next` pointers follow list order and are easy to wire. `random` pointers are the problem: when copying node `A`, its `random` may point to node `B` which may not yet be copied. A naive approach would either create duplicate copies of `B` or require multiple passes to discover which copy of `B` to wire to.

### Approach 1 — Hash Map (O(n) Space)

**Two-pass:** First pass creates all clone nodes and maps each original to its clone. Second pass wires `next` and `random` using the map.

```cpp
Node* copyRandomList(Node* head) {
    if (!head) return nullptr;
    unordered_map<Node*, Node*> cloneMap;

    // Pass 1: create all clones
    Node* curr = head;
    while (curr) {
        cloneMap[curr] = new Node(curr->val);
        curr = curr->next;
    }

    // Pass 2: wire next and random
    curr = head;
    while (curr) {
        cloneMap[curr]->next   = cloneMap[curr->next];
        cloneMap[curr]->random = cloneMap[curr->random];
        curr = curr->next;
    }

    return cloneMap[head];
}
```

**Why `cloneMap[nullptr]` is safe:** For `unordered_map<Node*, Node*>`, accessing a key of `nullptr` returns the default-constructed value for `Node*`, which is `nullptr`. So `cloneMap[curr->next]` when `curr->next == nullptr` correctly returns `nullptr`.

Time: O(n). Space: O(n).

### Approach 2 — Interweaving (O(1) Space)

**The key insight:** Instead of a separate hash map, embed the mapping into the list itself. Insert each clone node immediately after its original: `A -> A' -> B -> B' -> ...`. Now `original->next` is always `original`'s clone. This makes `random` pointer assignment possible without any auxiliary structure: if `A->random = R`, then `A->random->next = R->next = R'` (R's clone).

**Three passes:**

Pass 1 — Interweave: Create each clone and insert it after its original.

Pass 2 — Set random: For each original `A`, `A->next` is `A'`. Set `A'->random = A->random ? A->random->next : nullptr`.

Pass 3 — Separate: Extract the clone list and restore the original.

```cpp
Node* copyRandomList(Node* head) {
    if (!head) return nullptr;

    // Pass 1: interweave
    Node* curr = head;
    while (curr) {
        Node* clone = new Node(curr->val);
        clone->next = curr->next;
        curr->next = clone;
        curr = clone->next;   // skip clone, advance to next original
    }

    // Pass 2: set random pointers
    curr = head;
    while (curr) {
        curr->next->random = curr->random ? curr->random->next : nullptr;
        curr = curr->next->next;   // skip clone, advance to next original
    }

    // Pass 3: separate the two lists
    Node* dummyClone = new Node(0);
    Node* cloneTail = dummyClone;
    curr = head;
    while (curr) {
        Node* clone = curr->next;
        Node* nextOriginal = clone->next;

        cloneTail->next = clone;
        cloneTail = clone;

        curr->next = nextOriginal;  // restore original
        curr = nextOriginal;
    }
    cloneTail->next = nullptr;

    Node* cloneHead = dummyClone->next;
    delete dummyClone;
    return cloneHead;
}
```

**Why `A->random->next` gives the clone of A's random target:**

After Pass 1, every original node `X` satisfies `X->next == X'` (X's clone). For any original `R`, `R->next == R'`. When `A->random == R`, then `A->random->next == R->next == R'`. This is the clone of R — exactly what `A'->random` should reference.

**Dry run:** `1(→3) -> 2(→1) -> 3(→null)` (arrow indicates `random`)

```
Pass 1 (interweave):
  1: clone=1'. 1'->next=2. 1->next=1'. curr=2.
  2: clone=2'. 2'->next=3. 2->next=2'. curr=3.
  3: clone=3'. 3'->next=null. 3->next=3'. curr=null.
  List: 1->1'->2->2'->3->3'->null

Pass 2 (random):
  curr=1: 1->random=3. 3->next=3'. 1'->random=3'. curr=2.
  curr=2: 2->random=1. 1->next=1'. 2'->random=1'. curr=3.
  curr=3: 3->random=null. 3'->random=null. curr=null.

Pass 3 (separate):
  curr=1: clone=1', next=2. cloneList: 1'. 1->next=2. curr=2.
  curr=2: clone=2', next=3. cloneList: 1'->2'. 2->next=3. curr=3.
  curr=3: clone=3', next=null. cloneList: 1'->2'->3'. 3->next=null.

Original: 1->2->3 (random pointers unchanged)
Clone: 1'->2'->3' with 1'->3', 2'->1', 3'->null  ✓
```

Time: O(n). Space: O(1) auxiliary (the new nodes are the output, not overhead).

---

## 10. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Delete all occurrences DLL | Single scan | O(n) | O(1) |
| Find pairs with sum DLL | Two-pointer inward | O(n) | O(1) |
| Remove duplicates sorted DLL | Forward scan | O(n) | O(1) |
| Reverse K-Group | Recursive | O(n) | O(n/k) stack |
| Reverse K-Group | Iterative (dummy + getKth) | O(n) | O(1) |
| Rotate list | Circular + break | O(n) | O(1) |
| Flatten LL | Brute force (collect + sort) | O(NM log NM) | O(NM) |
| Flatten LL | Recursive sorted merge | O(N×M) | O(N) stack |
| Copy random list | Hash map (two-pass) | O(n) | O(n) |
| Copy random list | Interweaving (three-pass) | O(n) | O(1) |

N = number of horizontal nodes (flatten), M = average vertical depth.

---

## 11. Edge Cases Master Table

| Problem | Input | Expected | Reason |
|---|---|---|---|
| Delete DLL | Key = head value | `head = curr->next` | Head-update branch fires |
| Delete DLL | All nodes match key | `nullptr` | Every deletion advances head; final head = null |
| Delete DLL | Key not present | Unchanged list | Deletion block never fires |
| Find pairs DLL | No valid pair | `[]` | Pointers cross without sum matching target |
| Find pairs DLL | Single node | `[]` | `left == right` at start |
| Remove dups DLL | Three+ consecutive dups | Single node remaining | `curr` stays while `curr->next` equals `curr->data` |
| Remove dups DLL | No duplicates | Unchanged | Else branch always fires |
| K-Group | k=1 | Unchanged list | `getKth` returns first node; reversing 1 node is a no-op |
| K-Group | k=n (full list) | Fully reversed | One group covers everything |
| K-Group | n not divisible by k | Last `n%k` nodes unchanged | `getKth` returns null for last partial group |
| Rotate | k=0 | Unchanged | Early return |
| Rotate | k=n | Unchanged | `k%n=0` early return |
| Rotate | Single node | Unchanged | `!head->next` guard |
| Flatten | Single top-level node | That node's vertical list | Base case returns immediately |
| Copy random | `head=null` | `null` | Guard at entry |
| Copy random | `random` points to self | Clone's random points to itself | Pass 2: `A->random=A`, so `A->random->next=A'` |

---

## 12. Common Mistakes and Pitfalls

### Mistake 1 — Accessing `curr->next` after freeing `curr`

```cpp
// WRONG: curr is freed; curr->next is undefined behavior
if (curr->data == key) {
    delete curr;
}
curr = curr->next;   // accessing freed memory

// CORRECT: save next BEFORE any modification
Node* nextNode = curr->next;
if (curr->data == key) { ...; delete curr; }
curr = nextNode;
```

### Mistake 2 — Missing null checks on `prev` and `next` in DLL

```cpp
// WRONG: head->prev is null; this segfaults
curr->prev->next = curr->next;

// CORRECT
if (curr->prev) curr->prev->next = curr->next;
if (curr->next) curr->next->prev = curr->prev;
```

### Mistake 3 — Wrong termination in two-pointer DLL pair search

```cpp
// WRONG: misses even-length crossing case
while (left != right) { ... }
// After last pair, left crosses right. In even-length list, left and right
// skip over each other without ever being equal → loop never terminates.

// CORRECT: check both same-node and crossed-over cases
while (left != right && right->next != left) { ... }
```

### Mistake 4 — Not normalizing k before rotate

```cpp
// WRONG: walk stepsToNewTail = length - k - 1 when k > length → negative steps
int stepsToNewTail = length - k - 1;   // negative if k > length

// CORRECT: normalize first
k = k % length;
if (k == 0) return head;
int stepsToNewTail = length - k - 1;   // always in [0, length-2]
```

### Mistake 5 — Reversing partial group in K-Group when fewer than k nodes remain

```cpp
// WRONG: reverses any remaining nodes, including partial groups
// (violates problem statement: "leave them as-is")

// CORRECT: check before every reversal
ListNode* kth = getKth(groupPrev, k);
if (!kth) break;   // fewer than k nodes: done, do not reverse
```

### Mistake 6 — Setting clone->random to the original node instead of the clone

```cpp
// WRONG (hash map approach):
cloneMap[curr]->random = curr->random;     // points to original, not clone!

// CORRECT:
cloneMap[curr]->random = cloneMap[curr->random];  // points to clone
```

### Mistake 7 — Advancing curr after duplicate deletion in sorted DLL

```cpp
// WRONG: advances curr after deletion, missing subsequent duplicates
// For [1,1,1]: deletes first pair 1,1 → [1], then advances → misses nothing here,
// but for [1,1,1,2]: deletes first pair → [1,2], advances → misses nothing here,
// but for: curr=1, delete 1(dup) → curr->next=1(third). Advancing curr to 1(third)
// then curr->next=2 which is not a dup — this case works by accident.
// Fail case: [1,1,2,2,2]: delete first 1 → [1,2,2,2], advance to 2.
// Now curr=2, next=2: delete → [1,2,2], but we do NOT advance, correct.
// The rule: only advance when you did NOT delete.

if (curr->data == curr->next->data) {
    // delete curr->next
    curr = curr->next;   // WRONG: must NOT advance here
} else {
    curr = curr->next;   // correct to advance only here
}
```

### Mistake 8 — Forgetting `result->next = nullptr` in flatten's merge

```cpp
// WRONG: merged nodes retain stale horizontal next pointers
if (a->data <= b->data) {
    result = a;
    result->bottom = mergeSorted(a->bottom, b);
    // MISSING: result->next = nullptr
}

// CORRECT: always clear next
result->bottom = mergeSorted(a->bottom, b);
result->next = nullptr;
```

### Mistake 9 — Not breaking the circular link after rotate

```cpp
// WRONG: tail->next = head is set to make the list circular, but never broken.
// The returned newHead starts an infinite cycle.

// CORRECT: always break BEFORE returning
ListNode* newHead = newTail->next;
newTail->next = nullptr;   // break the circle
return newHead;
```

---

## 13. Interview Reference — Patterns and Extensions

### Pattern Recognition Table

| Signal in problem | Technique |
|---|---|
| Delete/modify nodes in DLL | Save `nextNode`; fix `prev` and `next` both ways; update head if needed |
| Find pairs in sorted DLL with target sum | Two-pointer (head + tail); stop when same node or crossed |
| Reverse groups of k | Dummy node + `getKth` check + reverse with `prev=groupNext` |
| Rotate a linked list | Normalize k; make circular; `stepsToNewTail = n-k-1`; break |
| Multiple sorted lists → one sorted list | Recursive sorted merge on `bottom` pointers |
| Deep copy with non-linear pointers | Hash map (O(n) space) or interweaving (O(1) space) |

### Related Problems

**From DLL operations:**
- LC 146 LRU Cache: uses a DLL (O(1) removal from any position via node pointer) + hash map. The `prev` pointer is what enables O(1) removal of arbitrary nodes.
- LC 460 LFU Cache: uses multiple DLLs, one per frequency level.

**From Reverse K-Group:**
- LC 24 Swap Nodes in Pairs: k=2 special case.
- LC 92 Reverse Linked List II: reverse one sub-range `[left, right]` — same group-reversal logic, applied once.

**From Rotate List:**
- LC 189 Rotate Array: identical modular arithmetic (`k % n`) and split-then-reconnect structure, applied to arrays.

**From Flatten:**
- LC 430 Flatten a Multilevel DLL: `child` pointer instead of `bottom`. Iterative with a stack: push the `next` node when a `child` exists; flatten the child.
- LC 23 Merge K Sorted Lists: K sorted lists with head pointers → min-heap of size K. Each extraction gives the current minimum in O(log K). Total: O(N log K).

**From Copy Random Pointer:**
- LC 133 Clone Graph: same principle — use a hash map `{original → clone}` to avoid re-cloning. BFS or DFS the graph.
- LC 1485 Clone Binary Tree with Random Pointer: interweaving trick adapted to trees.

**The interweaving trick generalized:** Works whenever (a) each node has one non-trivial identity pointer, (b) you can temporarily embed the `original → clone` mapping into the list structure by making `original->next = clone`, (c) the structure has no multiple-parent sharing. Does not directly generalize to graphs.

---

## 14. Quick Revision Cheat Sheet

This section is fully self-contained for night-before revision.

**DLL Node Deletion — Universal Template**
```cpp
Node* next = curr->next;                     // save FIRST
if (curr->prev) curr->prev->next = curr->next;  // bypass from left
if (curr->next) curr->next->prev = curr->prev;  // bypass from right
if (curr == head) head = curr->next;            // update head
delete curr;
curr = next;
```

**Find Pairs DLL — Two-Pointer**
```
Find tail: right = tail node.
left=head, right=tail.
Stop: left==right (odd) OR right->next==left (even, crossed).
sum < target → left = left->next
sum > target → right = right->prev
sum == target → record pair; left++, right--
```

**Remove Duplicates Sorted DLL**
```
curr->data == curr->next->data → delete curr->next; do NOT advance curr.
Different values → advance curr.
```

**Reverse K-Group — Iterative**
```
dummy → [group1] → [group2] → ...
groupPrev = dummy

loop:
  kth = getKth(groupPrev, k)      // null if < k nodes remain
  if !kth: break
  groupNext = kth->next
  Reverse [groupPrev->next .. kth] starting with prev=groupNext
    (this auto-connects reversed group's tail to groupNext)
  tmp = groupPrev->next            // save old first (now tail of group)
  groupPrev->next = kth            // connect to new head (was kth)
  groupPrev = tmp                  // advance groupPrev to new tail
```

**Rotate List**
```
k = k % length              (normalize; early return if 0)
tail->next = head           (make circular)
stepsToNewTail = length-k-1
Walk stepsToNewTail from head → newTail
newHead = newTail->next
newTail->next = nullptr     (break — critical, do not forget)
return newHead
```

**Flatten LL — Recursive Merge**
```cpp
Node* flatten(Node* root) {
    if (!root || !root->next) return root;
    root->next = flatten(root->next);      // flatten right first
    root = mergeSorted(root, root->next);  // merge with flattened right
    return root;
}
// In mergeSorted: traverse bottom pointers; set result->next = nullptr
```

**Copy Random — Hash Map (O(n) space)**
```
Pass 1: for each original, create clone. Store {original → clone}.
Pass 2: clone->next = map[orig->next]; clone->random = map[orig->random].
```

**Copy Random — Interweave (O(1) space)**
```
Pass 1: A → A' → B → B' → ...  (clone inserted after each original)
Pass 2: A'->random = A->random ? A->random->next : null
        (A->random->next is the clone of A's random target)
Pass 3: Separate lists; restore original next pointers.
```

**Complexity Reference**

| Problem | Time | Space |
|---|---|---|
| Delete all occurrences DLL | O(n) | O(1) |
| Pairs with sum DLL | O(n) | O(1) |
| Remove duplicates DLL | O(n) | O(1) |
| Reverse K-Group (iterative) | O(n) | O(1) |
| Rotate List | O(n) | O(1) |
| Flatten LL | O(N×M) | O(N) |
| Copy Random (hash map) | O(n) | O(n) |
| Copy Random (interweave) | O(n) | O(1) |
