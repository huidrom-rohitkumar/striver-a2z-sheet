> Video References: [LL Introduction](https://youtu.be/Nq7ok-OyEpg) | [LL in C++](https://youtu.be/VaECK03Dz-g) | [LL Insertion](https://youtu.be/0eKMU10uEDI) | [LL Deletion](https://youtu.be/u3WUW2qe6ww)

> Topics Covered: What is a Linked List | Node Structure | SLL: Creation, Traversal, Insertion, Deletion, Search, Count | Delete Node Without Head (LC 237) | Doubly Linked List: Structure, Insertion, Deletion, Reversal

---

## Table of Contents

1. [Why Linked Lists — The Mental Model](#1-why-linked-lists--the-mental-model)
2. [Memory Internals — How Linked Lists Actually Work](#2-memory-internals--how-linked-lists-actually-work)
3. [Node Structure](#3-node-structure)
4. [Creating a Singly Linked List](#4-creating-a-singly-linked-list)
5. [Traversal and Length](#5-traversal-and-length)
6. [Search in a Linked List](#6-search-in-a-linked-list)
7. [Insertion in a Singly Linked List](#7-insertion-in-a-singly-linked-list)
8. [Deletion in a Singly Linked List](#8-deletion-in-a-singly-linked-list)
9. [Delete Node Without Head (LC 237)](#9-delete-node-without-head-lc-237)
10. [Doubly Linked List — Structure](#10-doubly-linked-list--structure)
11. [Insertion in a Doubly Linked List](#11-insertion-in-a-doubly-linked-list)
12. [Deletion in a Doubly Linked List](#12-deletion-in-a-doubly-linked-list)
13. [Reverse a Doubly Linked List](#13-reverse-a-doubly-linked-list)
14. [Arrays vs Linked Lists — When to Use Which](#14-arrays-vs-linked-lists--when-to-use-which)
15. [Complexity Cheat Sheet](#15-complexity-cheat-sheet)
16. [Common Mistakes and Pitfalls](#16-common-mistakes-and-pitfalls)
17. [Quick Revision Cheat Sheet](#17-quick-revision-cheat-sheet)

---

## 1. Why Linked Lists — The Mental Model

An array stores elements in **contiguous memory**. Its strength is O(1) random access by index. Its weaknesses are that insertion and deletion in the middle require shifting all subsequent elements — O(n) — and its size is fixed at allocation time.

A linked list stores elements in **non-contiguous heap memory**. Each element knows only the address of the next one. You trade O(1) index access for O(1) insertion and deletion at any already-known position.

```
Array — contiguous blocks:
[10][20][30][40][50]
 ↑
Access by index: O(1). Insert at position k: O(n) shift.

Singly Linked List — scattered blocks connected by pointers:
[10|*] → [20|*] → [30|*] → [40|NULL]
 ↑
Access by index: O(n) traverse. Insert after a known node: O(1).
```

**The four variants:**

```
Singly Linked List (SLL):    each node has one pointer — next
Doubly Linked List (DLL):    each node has two pointers — prev and next
Circular Singly LL:          last node's next points back to head
Circular Doubly LL:          both circular and doubly linked
```

This module covers SLL and DLL in full — every other linked list problem in placements is built on top of these two.

**Real-world uses of linked lists:** LRU cache (DLL + hash map), graph adjacency lists, hash chaining, undo/redo systems, memory allocators. They appear everywhere that dynamic, pointer-connected structures are needed.

---

## 2. Memory Internals — How Linked Lists Actually Work

When you write `new Node(5)` in C++, the OS allocates memory on the **heap** at some runtime address, say `0x1A3F`. The `next` pointer field inside that node holds the address of the next node — say `0x2B7C`. The `head` variable (typically on the stack) holds the address of the first node.

```
Stack:               Heap (addresses are random — non-contiguous):
head = 0x1A3F       0x1A3F: [data=10 | next=0x2B7C]
                    0x2B7C: [data=20 | next=0x4D1E]
                    0x4D1E: [data=30 | next=NULL  ]
```

This non-contiguous layout is why:
- There is no formula for the address of element `i` — you must traverse from head.
- Insertion and deletion only change a constant number of pointers, regardless of list size.
- Nodes can be added one at a time on demand — no pre-allocation.

**Cache consequence:** Arrays benefit from spatial locality — nearby elements load into the same cache line. Linked lists suffer from pointer chasing — each `node->next` is a random address, causing cache misses. For traversal-heavy workloads, arrays are faster in practice even when the asymptotic complexity is the same.

### Memory Management Responsibility

In C++, nodes created with `new` live on the heap and are NOT automatically freed. When you logically delete a node (by unlinking it), you must call `delete node` to return its memory to the OS. Failing to do so causes **memory leaks**.

```cpp
// CORRECT: unlink then free
Node* temp = head;
head = head->next;
delete temp;
temp = nullptr;  // optional defensive practice

// WRONG: leaks the old head node
head = head->next;  // old head is unreachable but still occupies heap memory
```

---

## 3. Node Structure

### Singly Linked List Node

```cpp
struct Node {
    int data;
    Node* next;

    Node(int val) : data(val), next(nullptr) {}
    Node(int val, Node* nxt) : data(val), next(nxt) {}
};
```

### Doubly Linked List Node

```cpp
struct Node {
    int data;
    Node* prev;
    Node* next;

    Node(int val) : data(val), prev(nullptr), next(nullptr) {}
    Node(int val, Node* p, Node* n) : data(val), prev(p), next(n) {}
};
```

**Why constructors matter:** Without them, every node creation requires three separate assignments. With them: `new Node(5)` is one clean, atomic operation with no risk of forgetting to initialize `next`.

**`nullptr` vs `NULL`:** `nullptr` is the C++11 type-safe null pointer constant. `NULL` is a C macro that expands to `0`. Always use `nullptr` in modern C++. In interview code, `NULL` is universally accepted.

---

## 4. Creating a Singly Linked List

### Manual Node Construction

```cpp
Node* head = new Node(10);
head->next = new Node(20);
head->next->next = new Node(30);
// head → [10] → [20] → [30] → NULL
```

### From an Array (Practical Pattern)

```cpp
Node* createLinkedList(vector<int>& arr) {
    if (arr.empty()) return nullptr;
    Node* head = new Node(arr[0]);
    Node* curr = head;
    for (int i = 1; i < (int)arr.size(); i++) {
        curr->next = new Node(arr[i]);
        curr = curr->next;
    }
    return head;
}
```

**Dry Run for arr = [1, 2, 3, 4]:**

```
head = [1|NULL], curr = head

i=1: curr->next = [2|NULL], curr = [2]   →  1→2→NULL
i=2: curr->next = [3|NULL], curr = [3]   →  1→2→3→NULL
i=3: curr->next = [4|NULL], curr = [4]   →  1→2→3→4→NULL

Return head (pointing to node 1)
```

**Time:** O(n). **Space:** O(n) — n nodes on heap.

### The Tail Pointer Optimization

Maintaining a separate `tail` pointer enables O(1) append instead of O(n) traversal-to-end:

```cpp
Node* head = new Node(1);
Node* tail = head;

// O(1) append:
tail->next = new Node(2);
tail = tail->next;
```

---

## 5. Traversal and Length

### The Canonical Traversal Pattern

```cpp
Node* curr = head;
while (curr != nullptr) {
    // process curr->data
    curr = curr->next;
}
```

**Never use `while (curr->next != nullptr)` for general traversal** — this stops one node before the end and misses the last node. Reserve `curr->next != nullptr` only for situations where you need to look ahead (e.g., finding the second-to-last node for tail deletion).

### Print All Elements

```cpp
void printList(Node* head) {
    Node* curr = head;
    while (curr != nullptr) {
        cout << curr->data;
        if (curr->next) cout << " → ";
        curr = curr->next;
    }
    cout << " → NULL" << endl;
}
```

### Count Nodes

```cpp
int countNodes(Node* head) {
    int count = 0;
    Node* curr = head;
    while (curr != nullptr) {
        count++;
        curr = curr->next;
    }
    return count;
}
```

**Dry Run for 1 → 2 → 3 → NULL:**

```
curr=1: count=1, curr=2
curr=2: count=2, curr=3
curr=3: count=3, curr=NULL
Stop. Return 3.
```

**Time:** O(n). **Space:** O(1).

---

## 6. Search in a Linked List

```cpp
bool searchKey(Node* head, int key) {
    Node* curr = head;
    while (curr != nullptr) {
        if (curr->data == key) return true;
        curr = curr->next;
    }
    return false;
}
```

**Time:** O(n) — must examine every element in the worst case.
**Space:** O(1) iterative.

Recursive version (O(n) call stack):

```cpp
bool searchKey(Node* head, int key) {
    if (head == nullptr) return false;
    if (head->data == key) return true;
    return searchKey(head->next, key);
}
```

---

## 7. Insertion in a Singly Linked List

Every insertion reduces to: create a new node, set its `next`, then update the predecessor's `next`. The pointer assignment order matters — always set the new node's pointers before severing existing links.

### 7.1 Insert at Head — O(1)

```cpp
Node* insertAtHead(Node* head, int val) {
    Node* newNode = new Node(val);
    newNode->next = head;    // new node points to old head
    return newNode;          // new node is the new head
}
```

```
Before: head → [10] → [20] → NULL
Insert 5 at head:
  newNode = [5|NULL]
  newNode->next = [10]   →   [5|→10] → [10] → [20] → NULL
  return newNode (= new head)
After: head → [5] → [10] → [20] → NULL
```

**Why return head?** The head pointer itself changes. The caller must receive the new head.

### 7.2 Insert at Tail — O(n)

```cpp
Node* insertAtTail(Node* head, int val) {
    Node* newNode = new Node(val);
    if (head == nullptr) return newNode;

    Node* curr = head;
    while (curr->next != nullptr) curr = curr->next;  // reach last node
    curr->next = newNode;
    return head;
}
```

**Dry Run inserting 40 at tail of 1 → 2 → 3 → NULL:**

```
curr=1: next=2, not NULL → advance
curr=2: next=3, not NULL → advance
curr=3: next=NULL → stop
curr->next = [40|NULL]
List: 1 → 2 → 3 → 40 → NULL
```

### 7.3 Insert at Position k (1-indexed) — O(k)

```cpp
Node* insertAtPosition(Node* head, int val, int pos) {
    if (pos == 1) {
        Node* newNode = new Node(val);
        newNode->next = head;
        return newNode;
    }

    Node* curr = head;
    for (int i = 1; i < pos - 1 && curr != nullptr; i++)
        curr = curr->next;

    if (curr == nullptr) return head;  // position out of range

    Node* newNode = new Node(val);
    newNode->next = curr->next;   // save rest of list FIRST
    curr->next = newNode;         // then link current to new node
    return head;
}
```

**Dry Run inserting 99 at position 3 in 1 → 2 → 3 → 4 → NULL:**

```
Traverse to pos-1=2: curr arrives at [2].
newNode->next = curr->next = [3]    (rest of list saved)
curr->next = newNode = [99]         (link updated)
List: 1 → 2 → 99 → 3 → 4 → NULL  ✓
```

**Critical: pointer order matters.**

```cpp
// WRONG — loses the tail:
curr->next = newNode;        // curr's old next is now inaccessible
newNode->next = curr->next;  // curr->next is already newNode — circular!

// CORRECT:
newNode->next = curr->next;  // save the rest first
curr->next = newNode;
```

---

## 8. Deletion in a Singly Linked List

Every deletion: find the node to delete and its predecessor, update predecessor's `next`, then `delete` the node to free memory.

### 8.1 Delete the Head — O(1)

```cpp
Node* deleteHead(Node* head) {
    if (head == nullptr) return nullptr;
    Node* temp = head;
    head = head->next;
    delete temp;
    return head;
}
```

### 8.2 Delete the Tail — O(n)

```cpp
Node* deleteTail(Node* head) {
    if (head == nullptr) return nullptr;
    if (head->next == nullptr) { delete head; return nullptr; }

    Node* curr = head;
    while (curr->next->next != nullptr) curr = curr->next;  // stop at second-to-last
    delete curr->next;
    curr->next = nullptr;
    return head;
}
```

**Why `curr->next->next != nullptr`?** We need to arrive at the node **before** the tail so we can set its `next` to `nullptr`. If we stop at the tail itself, we have no reference to its predecessor.

### 8.3 Delete at Position k — O(k)

```cpp
Node* deleteAtPosition(Node* head, int pos) {
    if (head == nullptr) return nullptr;
    if (pos == 1) return deleteHead(head);

    Node* curr = head;
    for (int i = 1; i < pos - 1 && curr->next != nullptr; i++)
        curr = curr->next;

    if (curr->next == nullptr) return head;  // position out of range

    Node* toDelete = curr->next;
    curr->next = toDelete->next;
    delete toDelete;
    return head;
}
```

### 8.4 Delete by Value — O(n)

```cpp
Node* deleteByValue(Node* head, int val) {
    if (head == nullptr) return nullptr;
    if (head->data == val) return deleteHead(head);

    Node* curr = head;
    while (curr->next != nullptr) {
        if (curr->next->data == val) {
            Node* toDelete = curr->next;
            curr->next = toDelete->next;
            delete toDelete;
            return head;  // delete first occurrence only
        }
        curr = curr->next;
    }
    return head;  // value not found
}
```

**Dry Run deleting 20 from 10 → 20 → 30 → NULL:**

```
curr=10: curr->next->data=20 == val
  toDelete = [20]
  curr->next = [30]
  delete [20]
List: 10 → 30 → NULL  ✓
```

---

## 9. Delete Node Without Head (LC 237)

**Problem:** You are given a pointer directly to a node (guaranteed NOT to be the tail). Delete it from the list. You have no access to the head.

**LeetCode 237 — Delete Node in a Linked List**

### Why Standard Deletion Fails

Standard deletion requires the **predecessor** of the target node (to update `prev->next`). Without the head, you cannot traverse to find the predecessor. This appears impossible.

### The Copy-and-Skip Trick

You cannot remove the current node, but you can make it disappear by copying the next node's value into the current node, then deleting the next node — whose predecessor you do know (it's the current node).

```
Before: ... → [4] → [5] → [1] → [9] → NULL
                      ↑
                node to delete (val=5)

Step 1: Copy next node's value into current:
        ... → [4] → [1] → [1] → [9] → NULL

Step 2: Skip (delete) the next node:
        ... → [4] → [1] → [9] → NULL
```

The result is equivalent to having deleted the node containing 5.

```cpp
void deleteNode(ListNode* node) {
    ListNode* toDelete = node->next;
    node->val = toDelete->val;    // copy next node's value here
    node->next = toDelete->next;  // skip next node
    delete toDelete;              // free memory
}
```

**Time:** O(1). **Space:** O(1).

**Why the problem guarantees "not the tail":** If the node were the tail, `node->next` would be `nullptr`. Dereferencing `nullptr->val` is undefined behavior. The guarantee exists precisely to make this trick valid.

### Edge Case Analysis

| Scenario | Behavior |
|---|---|
| Node is in the middle | Works correctly |
| Node is second-to-last | Works: copies tail's value, removes tail |
| Node is the tail | Undefined — guaranteed not to happen |
| List has exactly 2 nodes; delete first | Copies second's value into first, removes second. Result: one-node list. |

---

## 10. Doubly Linked List — Structure

A **Doubly Linked List (DLL)** gives each node a `prev` pointer in addition to `next`, enabling O(1) traversal in both directions and O(1) deletion of any node given only its pointer.

```
NULL ← [prev|10|next] ↔ [prev|20|next] ↔ [prev|30|next] → NULL
          ↑                                     ↑
         head                                  tail
```

### Advantages of DLL over SLL

| Operation | SLL | DLL |
|---|---|---|
| Delete a given node (with its pointer) | O(n) — need predecessor via traversal | O(1) — prev pointer IS the predecessor |
| Traverse backward | Not possible | O(n) |
| Insert before a given node | O(n) — need predecessor | O(1) |
| Memory per node | data + 1 pointer | data + 2 pointers |

### Creating a DLL from an Array

```cpp
Node* createDLL(vector<int>& arr) {
    if (arr.empty()) return nullptr;
    Node* head = new Node(arr[0]);
    Node* curr = head;
    for (int i = 1; i < (int)arr.size(); i++) {
        Node* newNode = new Node(arr[i]);
        newNode->prev = curr;
        curr->next = newNode;
        curr = newNode;
    }
    return head;
}
```

---

## 11. Insertion in a Doubly Linked List

For every DLL insertion, you must update **both** `prev` and `next` pointers on the new node and update neighbors' `prev`/`next` to point to it. Always guard against `nullptr` before dereferencing.

### 11.1 Insert at Head — O(1)

```cpp
Node* insertAtHeadDLL(Node* head, int val) {
    Node* newNode = new Node(val);
    newNode->next = head;
    if (head != nullptr) head->prev = newNode;
    return newNode;  // new node is new head
}
```

```
Before: head → [10] ↔ [20] → NULL
Insert 5 at head:
  newNode = [5|NULL|NULL]
  newNode->next = [10]         [5|NULL|→10]
  head->prev = newNode         [5|NULL|→10] ↔ [←5|10|→20]
  return newNode
After: head → [5] ↔ [10] ↔ [20] → NULL
```

### 11.2 Insert at Tail — O(n) without tail pointer, O(1) with

```cpp
Node* insertAtTailDLL(Node* head, int val) {
    Node* newNode = new Node(val);
    if (head == nullptr) return newNode;

    Node* curr = head;
    while (curr->next != nullptr) curr = curr->next;
    curr->next = newNode;
    newNode->prev = curr;
    return head;
}
```

### 11.3 Insert After a Given Node — O(1) (Given Pointer)

```cpp
void insertAfterDLL(Node* node, int val) {
    if (node == nullptr) return;
    Node* newNode = new Node(val);
    newNode->prev = node;
    newNode->next = node->next;
    if (node->next != nullptr) node->next->prev = newNode;
    node->next = newNode;
}
```

**The four-pointer update checklist for inserting after node X:**

```
1. newNode->prev = X
2. newNode->next = X->next
3. if X->next != nullptr: X->next->prev = newNode
4. X->next = newNode
```

Always perform steps 1 and 2 (setting newNode's own pointers) before steps 3 and 4 (updating neighbors), since step 4 overwrites `X->next`.

### 11.4 Insert Before a Given Node — O(1) (Given Pointer)

```cpp
void insertBeforeDLL(Node* node, int val) {
    if (node == nullptr) return;
    Node* newNode = new Node(val);
    newNode->next = node;
    newNode->prev = node->prev;
    if (node->prev != nullptr) node->prev->next = newNode;
    node->prev = newNode;
}
```

---

## 12. Deletion in a Doubly Linked List

### 12.1 Delete the Head — O(1)

```cpp
Node* deleteHeadDLL(Node* head) {
    if (head == nullptr) return nullptr;
    Node* temp = head;
    head = head->next;
    if (head != nullptr) head->prev = nullptr;
    delete temp;
    return head;
}
```

### 12.2 Delete the Tail — O(n) without tail pointer

```cpp
Node* deleteTailDLL(Node* head) {
    if (head == nullptr) return nullptr;
    if (head->next == nullptr) { delete head; return nullptr; }

    Node* curr = head;
    while (curr->next != nullptr) curr = curr->next;  // reach tail
    curr->prev->next = nullptr;
    delete curr;
    return head;
}
```

### 12.3 Delete a Given Node — O(1) (Given Pointer)

This is DLL's defining advantage over SLL: no traversal needed to find the predecessor.

```cpp
Node* deleteGivenNodeDLL(Node* head, Node* del) {
    if (head == nullptr || del == nullptr) return head;

    if (del == head) return deleteHeadDLL(head);

    if (del->prev != nullptr) del->prev->next = del->next;
    if (del->next != nullptr) del->next->prev = del->prev;

    delete del;
    return head;
}
```

**Dry Run for deleting [20] from NULL ← [10] ↔ [20] ↔ [30] → NULL:**

```
del = [20], del->prev = [10], del->next = [30]

del->prev->next = del->next   →  [10]->next = [30]
del->next->prev = del->prev   →  [30]->prev = [10]
delete [20]

Result: NULL ← [10] ↔ [30] → NULL  ✓
```

**Time:** O(1). **Space:** O(1).

**The deletion checklist for DLL node D:**

```
1. if D == head: new head = D->next, new head->prev = nullptr
2. if D->prev != nullptr: D->prev->next = D->next
3. if D->next != nullptr: D->next->prev = D->prev
4. delete D
```

---

## 13. Reverse a Doubly Linked List

**Problem:** Reverse the DLL so the first node becomes last and vice versa.

```
1 ↔ 2 ↔ 3 ↔ 4 → NULL   →   4 ↔ 3 ↔ 2 ↔ 1 → NULL
```

### Core Insight

Swapping `prev` and `next` pointers on every node effectively reverses all links. After processing all nodes, the original last node (whose `prev` now points to its former predecessor — which is its new `next`) becomes the new head.

### Approach 1 — Swap prev/next on Every Node (Optimal)

```cpp
Node* reverseDLL(Node* head) {
    if (head == nullptr || head->next == nullptr) return head;

    Node* curr = head;
    Node* newHead = nullptr;

    while (curr != nullptr) {
        swap(curr->prev, curr->next);
        // After swap:
        //   curr->prev holds what was curr->next (i.e., the next node to visit)
        //   curr->next holds what was curr->prev (i.e., the previous node)
        newHead = curr;
        curr = curr->prev;   // advance forward via the swapped pointer
    }

    return newHead;
}
```

**Detailed Dry Run for 1 ↔ 2 ↔ 3 → NULL:**

```
Initial: NULL←[1]↔[2]↔[3]→NULL

curr=[1] (prev=NULL, next=[2]):
  swap(prev,next): prev=[2], next=NULL
  newHead=[1], curr=curr->prev=[2]

curr=[2] (prev=[1], next=[3]):
  swap(prev,next): prev=[3], next=[1]
  newHead=[2], curr=curr->prev=[3]

curr=[3] (prev=[2], next=NULL):
  swap(prev,next): prev=NULL, next=[2]
  newHead=[3], curr=curr->prev=NULL → stop

Return newHead=[3]

Verify: [3].prev=NULL, [3].next=[2]
        [2].prev=[3],  [2].next=[1]
        [1].prev=[2],  [1].next=NULL

NULL ← [3] ↔ [2] ↔ [1] → NULL  ✓
```

**Time:** O(n) — one pass. **Space:** O(1).

### Approach 2 — Swap Data Values (Simpler, Less Preferred)

```cpp
Node* reverseDLL(Node* head) {
    Node* left = head, *right = head;
    while (right->next != nullptr) right = right->next;  // find tail

    while (left != right && left->prev != right) {
        swap(left->data, right->data);
        left = left->next;
        right = right->prev;
    }
    return head;
}
```

**Time:** O(n). **Space:** O(1). Simpler to implement but swaps data, not structure. When nodes contain large objects, swapping data is expensive. The pointer-swap approach is preferred.

---

## 14. Arrays vs Linked Lists — When to Use Which

| Feature | Array | Singly Linked List | Doubly Linked List |
|---|---|---|---|
| Memory layout | Contiguous | Scattered | Scattered |
| Access by index | O(1) | O(n) | O(n) |
| Search by value | O(n) | O(n) | O(n) |
| Insert at head | O(n) shift | O(1) | O(1) |
| Insert at tail (no tail ptr) | O(1) amortized | O(n) | O(n) |
| Insert at tail (with tail ptr) | O(1) | O(1) | O(1) |
| Delete at head | O(n) shift | O(1) | O(1) |
| Delete at tail | O(1) | O(n) | O(1) with tail ptr |
| Delete given node (with pointer) | N/A | O(n) need predecessor | O(1) use prev |
| Cache performance | Excellent | Poor | Poor |
| Dynamic sizing | No | Yes | Yes |
| Memory per element | `sizeof(data)` | +1 pointer | +2 pointers |

**Choose linked lists when:** frequent insertions/deletions at head or at known positions; size is unknown or highly dynamic; implementing stacks, queues, adjacency lists, or LRU cache.

**Choose arrays when:** frequent random access by index; cache performance matters; size is known; binary search is needed.

---

## 15. Complexity Cheat Sheet

### Singly Linked List

| Operation | Time | Space | Notes |
|---|---|---|---|
| Create from array | O(n) | O(n) | n nodes on heap |
| Traverse / Print | O(n) | O(1) | |
| Count nodes | O(n) | O(1) | |
| Search by value | O(n) | O(1) | |
| Insert at head | O(1) | O(1) | |
| Insert at tail (no tail ptr) | O(n) | O(1) | |
| Insert at tail (with tail ptr) | O(1) | O(1) | |
| Insert at position k | O(k) | O(1) | |
| Delete at head | O(1) | O(1) | |
| Delete at tail | O(n) | O(1) | Need second-to-last node |
| Delete at position k | O(k) | O(1) | |
| Delete by value | O(n) | O(1) | |
| Delete given node (LC 237) | O(1) | O(1) | Copy-next-skip; node must not be tail |

### Doubly Linked List

| Operation | Time | Space | Notes |
|---|---|---|---|
| Insert at head | O(1) | O(1) | |
| Insert at tail (with tail ptr) | O(1) | O(1) | |
| Delete at head | O(1) | O(1) | |
| Delete at tail (with tail ptr) | O(1) | O(1) | |
| Delete given node (with pointer) | O(1) | O(1) | Key advantage over SLL |
| Reverse (pointer swap) | O(n) | O(1) | |
| Traverse backward | O(n) | O(1) | |

---

## 16. Common Mistakes and Pitfalls

### Mistake 1 — Wrong Pointer Order During SLL Insertion

```cpp
// WRONG: overwrites curr->next before saving it
curr->next = newNode;        // curr's old next is now unreachable
newNode->next = curr->next;  // curr->next is now newNode — circular!

// CORRECT: save the rest of the list first
newNode->next = curr->next;
curr->next = newNode;
```

### Mistake 2 — Not Freeing Deleted Nodes (Memory Leak)

```cpp
// WRONG: logically removes but leaks
head = head->next;   // old head node still in heap, now unreachable

// CORRECT:
Node* temp = head;
head = head->next;
delete temp;
```

### Mistake 3 — Dereferencing Null Pointer

```cpp
// WRONG: crashes on empty list
head->next = newNode;

// CORRECT: guard before dereferencing
if (head == nullptr) { head = newNode; return; }
head->next = newNode;
```

### Mistake 4 — Wrong Stopping Condition for Tail Deletion

```cpp
// WRONG: stops at tail, no way to update predecessor's next
while (curr->next != nullptr) curr = curr->next;

// CORRECT: stop at second-to-last
while (curr->next->next != nullptr) curr = curr->next;
delete curr->next;
curr->next = nullptr;
```

### Mistake 5 — Forgetting to Update prev in DLL

```cpp
// WRONG DLL insertion after X: links next but not prev
newNode->next = X->next;
X->next = newNode;
// newNode->next->prev still points to X, not newNode!

// CORRECT:
newNode->next = X->next;
if (X->next != nullptr) X->next->prev = newNode;  // update successor's prev
newNode->prev = X;
X->next = newNode;
```

### Mistake 6 — Wrong Traversal Stopping Condition

```cpp
// WRONG: misses the last node
while (curr->next != nullptr) {
    process(curr);       // last node never processed!
    curr = curr->next;
}

// CORRECT for processing all nodes:
while (curr != nullptr) {
    process(curr);
    curr = curr->next;
}
// Use curr->next != nullptr ONLY when you need to look ahead
```

### Mistake 7 — DLL: Not Handling head == del in deleteGivenNode

```cpp
// WRONG: del->prev is nullptr when del is head — segfault
del->prev->next = del->next;

// CORRECT: check for head first
if (del == head) {
    head = del->next;
    if (head) head->prev = nullptr;
    delete del;
    return;
}
del->prev->next = del->next;
if (del->next) del->next->prev = del->prev;
delete del;
```

### Mistake 8 — LC 237: Trying to Delete the Actual Node

```cpp
// WRONG: frees memory but does not update any list structure
delete node;

// CORRECT: copy-next-skip trick
node->val = node->next->val;
ListNode* toDelete = node->next;
node->next = toDelete->next;
delete toDelete;
```

---

## 17. Quick Revision Cheat Sheet

### Node Definitions

```cpp
// SLL
struct Node { int data; Node* next; Node(int v):data(v),next(nullptr){} };

// DLL
struct Node { int data; Node* prev; Node* next; Node(int v):data(v),prev(nullptr),next(nullptr){} };
```

### SLL Core Operations

```cpp
// Create from array
Node* head = new Node(arr[0]);
Node* curr = head;
for (int i=1; i<n; i++) { curr->next = new Node(arr[i]); curr = curr->next; }

// Traverse
for (Node* c = head; c; c = c->next) cout << c->data << " ";

// Count
int cnt=0; for (Node* c=head; c; c=c->next) cnt++;

// Search
for (Node* c=head; c; c=c->next) if (c->data==key) return true;

// Insert at head
newNode->next = head; head = newNode;

// Insert at tail (no tail ptr)
Node* c=head; while(c->next) c=c->next; c->next=newNode;

// Delete head
Node* temp=head; head=head->next; delete temp;

// Delete tail
Node* c=head; while(c->next->next) c=c->next;
delete c->next; c->next=nullptr;

// Delete by value
while(curr->next) {
    if(curr->next->data==val) { Node* t=curr->next; curr->next=t->next; delete t; break; }
    curr=curr->next;
}
```

### Delete Given Node Without Head (LC 237)

```cpp
void deleteNode(ListNode* node) {    // node is guaranteed non-tail
    ListNode* toDelete = node->next;
    node->val = toDelete->val;       // copy next's value here
    node->next = toDelete->next;     // skip next node
    delete toDelete;
}
```

### DLL Core Operations

```cpp
// Insert at head
newNode->next = head;
if (head) head->prev = newNode;
head = newNode;

// Delete given node D (O(1) — the key DLL advantage)
if (D->prev) D->prev->next = D->next;
if (D->next) D->next->prev = D->prev;
delete D;

// Reverse (swap prev/next on every node)
Node* curr=head; Node* newHead=nullptr;
while (curr) {
    swap(curr->prev, curr->next);
    newHead = curr;
    curr = curr->prev;   // after swap, prev holds the old next
}
return newHead;
```

### Pointer Update Rules

```
SLL: Insert after node X
  1. newNode->next = X->next    ← save rest of list first
  2. X->next = newNode

DLL: Insert after node X
  1. newNode->prev = X
  2. newNode->next = X->next
  3. if X->next: X->next->prev = newNode
  4. X->next = newNode

DLL: Delete node D
  1. if D->prev: D->prev->next = D->next
  2. if D->next: D->next->prev = D->prev
  3. delete D
  (handle head case separately: D == head → new head = D->next)
```

### Key Facts

```
SLL traversal:  while(curr != nullptr) — never while(curr->next != nullptr)
SLL tail delete: stop at curr->next->next == nullptr, not curr->next == nullptr

SLL delete given node: O(n) — must traverse to find predecessor
DLL delete given node: O(1) — prev pointer IS the predecessor

LC 237: no head given → copy-next-skip → O(1) (node must not be tail)
DLL reverse: swap(prev, next) per node; advance via curr->prev after swap; last node = new head

Memory: always delete freed nodes; nullptr-check before dereferencing
Order: always set newNode's pointers BEFORE updating existing nodes' pointers
```

---

*Videos: [LL Introduction](https://youtu.be/Nq7ok-OyEpg) | [LL in C++](https://youtu.be/VaECK03Dz-g) | [LL Insertion](https://youtu.be/0eKMU10uEDI) | [LL Deletion](https://youtu.be/u3WUW2qe6ww)*
