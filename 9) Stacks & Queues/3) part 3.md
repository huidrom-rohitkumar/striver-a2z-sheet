# stack-queue-and-cache-design.md

---

## Resources

- [LeetCode 239 — Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum)
- [LeetCode 901 — Online Stock Span](https://leetcode.com/problems/online-stock-span)
- [GFG — The Celebrity Problem](https://www.geeksforgeeks.org/problems/the-celebrity-problem/1)
- [LeetCode 146 — LRU Cache](https://leetcode.com/problems/lru-cache)
- [LeetCode 460 — LFU Cache](https://leetcode.com/problems/lfu-cache)
- [LeetCode 139 — Word Break](https://leetcode.com/problems/word-break)
- [Striver — Sliding Window Max (YouTube)](https://youtu.be/eay-zoSRkVc)
- [Striver — Stock Span (YouTube)](https://youtu.be/NwBvene4Imo)
- [Striver — Celebrity Problem (YouTube)](https://youtu.be/cEadsbTeze4)
- [Striver — LRU Cache (YouTube)](https://youtu.be/z9bJUPxzFOw)
- [Striver — LFU Cache (YouTube)](https://youtu.be/mzqHlAW7jeE)

---

## Table of Contents

1. [The Unifying Mental Model](#1-the-unifying-mental-model)
2. [Background: Monotonic Data Structures](#2-background-monotonic-data-structures)
3. [Problem: Sliding Window Maximum (LC 239)](#3-problem-sliding-window-maximum-lc-239)
4. [Problem: Online Stock Span (LC 901 / GFG)](#4-problem-online-stock-span-lc-901--gfg)
5. [Problem: The Celebrity Problem (GFG)](#5-problem-the-celebrity-problem-gfg)
6. [Problem: LRU Cache (LC 146)](#6-problem-lru-cache-lc-146)
7. [Problem: LFU Cache (LC 460)](#7-problem-lfu-cache-lc-460)
8. [Problem: Word Break (LC 139)](#8-problem-word-break-lc-139)
9. [Related Concepts and Interview Extensions](#9-related-concepts-and-interview-extensions)
10. [Interview Reference: Patterns and Follow-ups](#10-interview-reference-patterns-and-follow-ups)
11. [Common Mistakes and Pitfalls](#11-common-mistakes-and-pitfalls)
12. [Complexity Cheat Sheet](#12-complexity-cheat-sheet)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Model

These six problems fall into three families, each rooted in one core insight: **you do not need to keep every candidate around — only the ones that can still matter in the future.**

**Family 1 — Monotonic structures** (Sliding Window Max, Stock Span, Celebrity): A monotonic stack or deque enforces a sorted invariant as elements arrive. When a new element makes an older one permanently irrelevant, discard it immediately. This converts an O(n) scan per step into amortized O(1), giving O(n) total.

The governing question: *"If element X arrives and element Y is already here, is there any future query for which Y could be the answer but X could not?"* If no, discard Y immediately.

**Family 2 — Cache eviction structures** (LRU, LFU): A cache must answer "which item is eviction-worthy?" in O(1). The eviction criterion (recency for LRU, frequency for LFU) is maintained as a continuously updated ordering. Combine a hash map (O(1) lookup by key) with a doubly linked list (O(1) reorder/removal at arbitrary positions) — the hash map stores pointers directly into the list.

**Family 3 — String segmentation DP** (Word Break): The decision at index `i` — does any dictionary word starting here lead to a reachable position? — is independent of how we arrived at `i`. Memoization or bottom-up DP converts exponential backtracking into O(n²).

---

## 2. Background: Monotonic Data Structures

### What Is a Monotonic Stack?

A stack that maintains strictly increasing or decreasing order from bottom to top, enforced by popping violating elements when a new one arrives.

- **Monotonic increasing stack:** values increase bottom to top. Used for previous/next **smaller** element.
- **Monotonic decreasing stack:** values decrease bottom to top. Used for previous/next **greater** element (and range maximum).

### Why Amortized O(1)?

Each element is pushed at most once and popped at most once across the entire run. Total push+pop operations across n insertions ≤ 2n. So n insertions cost O(n) total — amortized O(1) per insertion.

### Deque vs Stack for Sliding Windows

A deque extends the stack idea: pop from the front (expire elements that left the window) and pop from the back (maintain the monotonic invariant). Needed whenever elements must be discarded from both ends.

### Should You Store Values or Indices?

- **Store indices when** you need to check if an element has left a window (`index <= i - k`). Values alone cannot encode position.
- **Store `(value, count)` pairs when** the structure spans queries without tracking individual positions (e.g., online Stock Span).

---

## 3. Problem: Sliding Window Maximum (LC 239)

### Problem Statement

Given array `nums` and integer `k`, find the maximum in every contiguous window of size `k`.

```
nums=[1,3,-1,-3,5,3,6,7], k=3  →  [3,3,5,5,6,7]
```

### Approach 1: Brute Force — O(n*k)

For each window, scan all k elements for the max.

### Approach 2: Max Heap with Lazy Deletion — O(n log k)

Push `(value, index)` pairs. Pop expired entries from the top lazily (when they surface) since heaps don't support O(1) arbitrary deletion.

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    int n = nums.size();
    vector<int> result;
    priority_queue<pair<int,int>> pq;   // (value, index), max-heap by value

    for (int i = 0; i < k; i++) pq.push({nums[i], i});
    result.push_back(pq.top().first);

    for (int i = k; i < n; i++) {
        pq.push({nums[i], i});
        while (pq.top().second <= i - k) pq.pop();   // lazy expiry
        result.push_back(pq.top().first);
    }
    return result;
}
// Time: O(n log k), Space: O(k)
```

### Approach 3: Monotonic Deque (Optimal) — O(n)

**Insight:** If `nums[j]` (j < i) is smaller than `nums[i]`, `nums[j]` can never again be the maximum of any window containing `nums[i]` — `nums[i]` will outlast it and is larger. Discard `nums[j]` immediately.

**Two invariants:**
1. **Expiry:** front index must be `>= i - k + 1`. Pop front if it falls outside the window.
2. **Monotonic:** values are non-increasing front to back. Pop back while its value `<= nums[i]`.

```cpp
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    int n = nums.size();
    vector<int> result;
    deque<int> dq;   // stores indices, non-increasing values front to back

    for (int i = 0; i < n; i++) {
        while (!dq.empty() && dq.front() <= i - k) dq.pop_front();         // expire
        while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();   // dominated
        dq.push_back(i);
        if (i >= k - 1) result.push_back(nums[dq.front()]);
    }
    return result;
}
// Time: O(n) — each index pushed once, popped once. Space: O(k)
```

### Dry Run

```
nums=[1,3,-1,-3,5,3,6,7], k=3

i=0: push 0. dq=[0]. (i<k-1, no output)
i=1: nums[0]=1<=3 → pop 0. push 1. dq=[1]. (i<k-1)
i=2: nums[1]=3>-1, no pop. push 2. dq=[1,2]. i=k-1 → output nums[1]=3. result=[3]
i=3: front=1, 1<=0? No. nums[2]=-1>-3, no pop. push 3. dq=[1,2,3].
     output nums[1]=3. result=[3,3]
i=4: front=1, 1<=1? Yes → pop. dq=[2,3].
     nums[3]=-3<=5 → pop. nums[2]=-1<=5 → pop. dq=[]. push 4. dq=[4].
     output nums[4]=5. result=[3,3,5]
i=5: front=4, 4<=2? No. nums[4]=5>3, no pop. push 5. dq=[4,5].
     output nums[4]=5. result=[3,3,5,5]
i=6: front=4, 4<=3? No. nums[5]=3<=6 → pop. nums[4]=5<=6 → pop. dq=[]. push 6.
     output nums[6]=6. result=[3,3,5,5,6]
i=7: front=6, 6<=4? No. nums[6]=6<=7 → pop. dq=[]. push 7.
     output nums[7]=7. result=[3,3,5,5,6,7]

Final: [3,3,5,5,6,7]  ✓
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `nums=[1], k=1` | `[1]` | Single window |
| `nums=[5,4,3,2,1], k=3` | `[5,4,3]` | Strictly decreasing: deque never shrinks from back |
| `nums=[1,2,3,4,5], k=3` | `[3,4,5]` | Strictly increasing: deque always size 1 |
| `nums=[1,1,1,1], k=2` | `[1,1,1]` | All equal: `<=` pops keep deque size 1 |

---

## 4. Problem: Online Stock Span (LC 901 / GFG)

### Problem Statement

**Offline (GFG):** Span of day `i` = number of consecutive days ending at `i` with price `<= arr[i]`.

```
arr=[100,80,60,70,75,85]  →  span=[1,1,1,2,3,5]
```

**Online (LC 901):** Implement `StockSpanner.next(price)` that processes prices one at a time.

### The Connection to "Previous Greater Element"

`span[i] = i - PGE[i]` where PGE[i] is the index of the nearest previous element strictly greater than `arr[i]`. If none exists, `span[i] = i + 1`. This is exactly what a monotonic decreasing stack computes.

### Approach 1: Brute Force — O(n²)

For each day, scan backward until a strictly greater price is found.

### Approach 2: Monotonic Stack — Offline (Store Indices) — O(n) amortized

Maintain a stack of indices with strictly decreasing prices. Pop all days with price `<= arr[i]` — they are absorbed into the span. The first remaining index is the previous greater day.

```cpp
vector<int> calculateSpan(vector<int>& arr) {
    int n = arr.size();
    vector<int> span(n);
    stack<int> st;   // indices, strictly decreasing prices bottom-to-top

    for (int i = 0; i < n; i++) {
        while (!st.empty() && arr[st.top()] <= arr[i]) st.pop();
        span[i] = st.empty() ? (i + 1) : (i - st.top());
        st.push(i);
    }
    return span;
}
// Time: O(n) amortized, Space: O(n)
```

### Dry Run

```
arr=[100,80,60,70,75,85]

i=0: stack empty. span[0]=1. push 0. stack=[0]
i=1: arr[0]=100>80, no pop. span[1]=1-0=1. push 1. stack=[0,1]
i=2: arr[1]=80>60, no pop. span[2]=2-1=1. push 2. stack=[0,1,2]
i=3: arr[2]=60<=70 → pop. arr[1]=80>70, stop. span[3]=3-1=2. push 3. stack=[0,1,3]
i=4: arr[3]=70<=75 → pop. arr[1]=80>75, stop. span[4]=4-1=3. push 4. stack=[0,1,4]
i=5: arr[4]=75<=85 → pop. arr[1]=80<=85 → pop. arr[0]=100>85, stop.
     span[5]=5-0=5. push 5. stack=[0,5]

Result: [1,1,1,2,3,5]  ✓
```

### Approach 3: Online Variant — Stack of (price, span) Pairs — O(n) amortized

Without indices (online), store cumulative span directly. When popping a smaller-or-equal price, absorb its span into the current count.

```cpp
class StockSpanner {
    stack<pair<int,int>> st;   // {price, cumulative_span}
public:
    int next(int price) {
        int span = 1;
        while (!st.empty() && st.top().first <= price) {
            span += st.top().second;
            st.pop();
        }
        st.push({price, span});
        return span;
    }
};
// Time: O(1) amortized per call, Space: O(n) worst case
```

### Dry Run

```
calls: next(100), next(80), next(60), next(70), next(75), next(85)

next(100): empty. span=1. push{100,1}. return 1.
next(80):  top={100,1}, 100>80. span=1. push{80,1}. return 1.
next(60):  top={80,1}, 80>60. span=1. push{60,1}. return 1.
next(70):  top={60,1}, 60<=70 → span=1+1=2. pop. top={80,1}, 80>70, stop.
           push{70,2}. return 2.
next(75):  top={70,2}, 70<=75 → span=1+2=3. pop. top={80,1}, 80>75, stop.
           push{75,3}. return 3.
next(85):  top={75,3}, 75<=85 → span=1+3=4. pop.
           top={80,1}, 80<=85 → span=4+1=5. pop.
           top={100,1}, 100>85, stop.
           push{85,5}. return 5.

Results: [1,1,1,2,3,5]  ✓
```

**Why the pair variant works without indices:** Storing the span directly allows constant-time accumulation — no index arithmetic needed.

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `[5,5,5,5]` | `[1,2,3,4]` | Equal prices extend span (`<=` condition) |
| `[1,2,3,4,5]` | `[1,2,3,4,5]` | Strictly increasing: every day includes all previous |
| `[5,4,3,2,1]` | `[1,1,1,1,1]` | Strictly decreasing: no day extends |

---

## 5. Problem: The Celebrity Problem (GFG)

### Problem Statement

Given `n x n` boolean matrix `mat` where `mat[i][j]=1` means person `i` knows person `j`. A **celebrity** is known by everyone but knows no one. Find their index, or -1 if none exists.

```
mat=[[1,1,0],[0,1,0],[0,1,1]]  →  1
```

### Why At Most One Celebrity Can Exist

Suppose A and B are both celebrities. Everyone knows B, so A knows B. But A is a celebrity and knows no one. Contradiction. At most one celebrity exists.

### Approach 1: Brute Force — In/Out Degree — O(n²)

Count in-degree and out-degree for every person. A celebrity has out-degree 1 (only self) and in-degree n (with the diagonal convention `mat[i][i]=1`).

### Approach 2: Stack-Based Elimination — O(n)

**Elimination principle:** Given any two persons A, B:
- If `mat[A][B]=1` (A knows B): A cannot be the celebrity. Eliminate A.
- If `mat[A][B]=0`: B cannot be the celebrity. Eliminate B.

Each comparison eliminates exactly one candidate with one lookup. After n-1 eliminations, exactly one candidate remains.

**Critical: the survivor is not guaranteed to be the celebrity — always verify.**

```cpp
int celebrity(vector<vector<int>>& mat) {
    int n = mat.size();
    stack<int> st;
    for (int i = 0; i < n; i++) st.push(i);

    while (st.size() > 1) {
        int a = st.top(); st.pop();
        int b = st.top(); st.pop();
        if (mat[a][b]) st.push(b);   // a knows b → a eliminated
        else            st.push(a);   // a doesn't know b → b eliminated
    }

    int candidate = st.top();

    for (int i = 0; i < n; i++) {
        if (i == candidate) continue;
        if (mat[candidate][i] || !mat[i][candidate]) return -1;
    }
    return candidate;
}
// Time: O(n), Space: O(n)
```

### Approach 3: Two-Pointer Elimination — O(n) time, O(1) space (Preferred)

Same elimination logic without the stack.

```cpp
int celebrity(vector<vector<int>>& mat) {
    int n = mat.size();
    int lo = 0, hi = n - 1;

    while (lo < hi) {
        if (mat[lo][hi]) lo++;   // lo knows hi → lo eliminated
        else              hi--;   // lo doesn't know hi → hi eliminated
    }

    int candidate = lo;

    for (int i = 0; i < n; i++) {
        if (i == candidate) continue;
        if (mat[candidate][i] || !mat[i][candidate]) return -1;
    }
    return candidate;
}
// Time: O(n), Space: O(1)
```

**Why verification is always necessary:** Elimination produces a survivor, not proof. Counter-example: `mat=[[0,0],[0,0]]` (nobody knows anybody). Two-pointer yields some candidate, but neither is a celebrity.

### Dry Run

```
mat=[[1,1,0],[0,1,0],[0,1,1]], n=3
lo=0, hi=2

mat[0][2]=0 → hi-- → hi=1
mat[0][1]=1 → lo++ → lo=1
lo==hi==1 → candidate=1

Verify:
  i=0: mat[1][0]=0 ✓ (1 doesn't know 0). mat[0][1]=1 ✓ (0 knows 1).
  i=2: mat[1][2]=0 ✓ (1 doesn't know 2). mat[2][1]=1 ✓ (2 knows 1).
All pass → return 1  ✓
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `n=1, mat=[[1]]` | `0` | Single person trivially celebrity |
| All off-diagonal zeros | `-1` | No one knows anyone |
| All off-diagonal ones | `-1` | Everyone knows everyone — no celebrity qualifies (everyone has nonzero out-degree) |

---

## 6. Problem: LRU Cache (LC 146)

### Problem Statement

Design a cache with O(1) `get(key)` and `put(key, value)`. On overflow, evict the **least recently used** key. Access via `get` or `put` counts as "used."

```
LRUCache(2)
put(1,1); put(2,2); get(1)→1 (1 now MRU); put(3,3) evicts 2; get(2)→-1
```

### The Core Insight

Three O(1) operations needed simultaneously:
1. **Lookup by key** — hash map.
2. **Move a node to MRU position** — needs O(1) arbitrary removal + O(1) insertion. A doubly linked list (DLL) gives both.
3. **Evict the LRU** — always the tail of the DLL.

The hash map stores `key → pointer to DLL node`. The DLL orders nodes by recency (head=MRU, tail=LRU).

**Why doubly linked, not singly:** Removing a middle node requires updating `prev.next`. Singly linked lists need O(n) to find `prev`; doubly linked nodes store `prev` directly, giving O(1) removal.

**Why the node stores its key:** When evicting the tail, you must also erase that key from the hash map — which requires knowing the key.

### Implementation

```cpp
struct Node {
    int key, val;
    Node *prev, *next;
    Node(int k=0, int v=0) : key(k), val(v), prev(nullptr), next(nullptr) {}
};

class LRUCache {
    int capacity;
    unordered_map<int, Node*> mp;
    Node *head, *tail;   // sentinels: head=MRU side, tail=LRU side

    void removeNode(Node* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }
    void insertAfterHead(Node* node) {
        node->next = head->next;
        node->prev = head;
        head->next->prev = node;
        head->next = node;
    }

public:
    LRUCache(int capacity) : capacity(capacity) {
        head = new Node(); tail = new Node();
        head->next = tail; tail->prev = head;
    }

    int get(int key) {
        if (!mp.count(key)) return -1;
        Node* node = mp[key];
        removeNode(node);
        insertAfterHead(node);   // access = promote to MRU
        return node->val;
    }

    void put(int key, int value) {
        if (mp.count(key)) {
            Node* node = mp[key];
            node->val = value;
            removeNode(node);
            insertAfterHead(node);
        } else {
            if ((int)mp.size() == capacity) {
                Node* lru = tail->prev;
                mp.erase(lru->key);
                removeNode(lru);
                delete lru;
            }
            Node* newNode = new Node(key, value);
            mp[key] = newNode;
            insertAfterHead(newNode);
        }
    }
};
// Time: O(1) per operation, Space: O(capacity)
```

### Cleaner Variant with `std::list`

`std::list::splice` moves a node to a new position in O(1) without invalidating iterators.

```cpp
class LRUCache {
    int cap;
    list<pair<int,int>> lst;   // front=MRU, back=LRU. {key, val}
    unordered_map<int, list<pair<int,int>>::iterator> mp;

public:
    LRUCache(int capacity) : cap(capacity) {}

    int get(int key) {
        if (!mp.count(key)) return -1;
        lst.splice(lst.begin(), lst, mp[key]);   // move to front, O(1)
        return mp[key]->second;
    }

    void put(int key, int value) {
        if (mp.count(key)) {
            mp[key]->second = value;
            lst.splice(lst.begin(), lst, mp[key]);
        } else {
            if ((int)lst.size() == cap) {
                mp.erase(lst.back().first);
                lst.pop_back();
            }
            lst.emplace_front(key, value);
            mp[key] = lst.begin();
        }
    }
};
```

**Important:** Iterators into `std::list` remain valid after `splice` — this is not true for `vector` or `deque`.

### Dry Run

```
capacity=2. Operations: put(1,1), put(2,2), get(1), put(3,3), get(2)

put(1,1): DLL: head↔N1{1,1}↔tail. map={1:N1}
put(2,2): DLL: head↔N2{2,2}↔N1{1,1}↔tail. map={1:N1,2:N2}
get(1): found N1. remove+reinsert at head.
  DLL: head↔N1{1,1}↔N2{2,2}↔tail. return 1.
put(3,3): not in map. size=2==capacity.
  Evict tail->prev=N2. erase key 2 from map. delete N2.
  DLL: head↔N1{1,1}↔tail. map={1:N1}.
  Insert N3{3,3}: DLL: head↔N3{3,3}↔N1{1,1}↔tail. map={1:N1,3:N3}.
get(2): not in map. return -1.
```

### Edge Cases

| Scenario | Behaviour |
|---|---|
| `get` on missing key | Return -1; do not modify recency |
| `put` on existing key | Move to MRU and update value; no eviction |
| `capacity = 1` | Every new key evicts the only existing key |

---

## 7. Problem: LFU Cache (LC 460)

### Problem Statement

Design a cache with O(1) `get`/`put`. On overflow, evict the **least frequently used** key. Tie-break: evict the **least recently used** among equal frequencies.

```
LFUCache(2)
put(1,1); put(2,2); get(1)→1 (freq[1]=2);
put(3,3) evicts key 2 (freq[2]=1 is min, only candidate)
```

### Why LFU Is Harder Than LRU

LRU needs one ordering dimension (recency). LFU needs two: frequency first, then recency within the same frequency. LRU needs one DLL; LFU needs one DLL **per frequency level**.

### Data Structure Design

1. `unordered_map<int, pair<int,int>> keyMap` — `key → {value, frequency}`.
2. `unordered_map<int, list<int>> freqMap` — `frequency → DLL of keys` (front=MRU, back=LRU within that frequency).
3. `unordered_map<int, list<int>::iterator> iterMap` — `key → iterator into its frequency's DLL`.
4. `int minFreq` — current minimum frequency across all keys.

**`get(key)` logic:**
1. Look up in `keyMap`. If absent, return -1.
2. Remove key from `freqMap[freq]`. If that bucket becomes empty and `freq == minFreq`, increment `minFreq`.
3. Insert key at the front of `freqMap[freq+1]`. Update `iterMap` and `keyMap[key].second`.
4. Return value.

**`put(key, value)` for a new key:**
1. If at capacity, evict `freqMap[minFreq].back()` (LRU among minimum frequency).
2. Insert the new key at `freqMap[1]`.
3. Set `keyMap[key] = {value, 1}`. Set `minFreq = 1`.

**Why `minFreq = 1` after insertion:** A new key always starts at frequency 1. No key can have frequency below 1, so the new global minimum is 1.

### Implementation

```cpp
class LFUCache {
    int capacity, minFreq, size;
    unordered_map<int, pair<int,int>> keyMap;        // key → {value, freq}
    unordered_map<int, list<int>> freqMap;            // freq → keys (MRU front)
    unordered_map<int, list<int>::iterator> iterMap;  // key → iterator

    void updateFreq(int key) {
        int freq = keyMap[key].second;
        freqMap[freq].erase(iterMap[key]);
        if (freqMap[freq].empty()) {
            freqMap.erase(freq);
            if (freq == minFreq) minFreq++;
        }
        int newFreq = freq + 1;
        freqMap[newFreq].push_front(key);
        iterMap[key] = freqMap[newFreq].begin();
        keyMap[key].second = newFreq;
    }

public:
    LFUCache(int capacity) : capacity(capacity), minFreq(0), size(0) {}

    int get(int key) {
        if (!keyMap.count(key)) return -1;
        updateFreq(key);
        return keyMap[key].first;
    }

    void put(int key, int value) {
        if (capacity == 0) return;

        if (keyMap.count(key)) {
            keyMap[key].first = value;
            updateFreq(key);
        } else {
            if (size == capacity) {
                int lruKey = freqMap[minFreq].back();
                freqMap[minFreq].pop_back();
                if (freqMap[minFreq].empty()) freqMap.erase(minFreq);
                keyMap.erase(lruKey);
                iterMap.erase(lruKey);
                size--;
            }
            keyMap[key] = {value, 1};
            freqMap[1].push_front(key);
            iterMap[key] = freqMap[1].begin();
            minFreq = 1;
            size++;
        }
    }
};
// Time: O(1) per operation, Space: O(capacity)
```

### Dry Run

```
capacity=2. Ops: put(1,1), put(2,2), get(1), put(3,3), get(2), get(3)

put(1,1): keyMap={1:{1,1}}, freqMap={1:[1]}, minFreq=1, size=1.
put(2,2): keyMap={1:{1,1},2:{2,1}}, freqMap={1:[2,1]}, minFreq=1, size=2.

get(1): updateFreq(1). Remove 1 from freqMap[1] → freqMap[1]=[2].
  minFreq stays 1 (bucket not empty). Insert 1 at freqMap[2].
  freqMap={1:[2], 2:[1]}. keyMap[1]={1,2}. return 1.

put(3,3): not in map. size=2==capacity.
  Evict freqMap[minFreq=1].back()=2. Erase 2 from all maps.
  freqMap[1] empty → erase. size=1.
  Insert 3: keyMap={1:{1,2},3:{3,1}}, freqMap={2:[1],1:[3]}, minFreq=1, size=2.

get(2): not in keyMap. return -1.

get(3): updateFreq(3). Remove 3 from freqMap[1] → empty → erase.
  1==minFreq → minFreq=2. Insert 3 at freqMap[2]. freqMap={2:[3,1]}.
  keyMap[3]={3,2}. return 3.
```

### Edge Cases

| Scenario | Behaviour |
|---|---|
| `capacity=0` | All puts ignored; all gets return -1 |
| Equal frequency tie | Evict the one not accessed most recently within that bucket |
| Updating existing key's value | Both value update and frequency promotion happen |

---

## 8. Problem: Word Break (LC 139)

### Problem Statement

Given string `s` and dictionary `wordDict`, determine if `s` can be segmented into space-separated dictionary words (reuse allowed).

```
s="leetcode", wordDict=["leet","code"]  →  true
s="catsandog", wordDict=["cats","dog","sand","and","cat"]  →  false
```

### Approach 1: Brute Force Recursion — O(n^n) worst case

At index `i`, try every prefix `s[i..j]`. If a dictionary word, recurse on `s[j+1..]`.

```cpp
bool helper(const string& s, unordered_set<string>& dict, int start) {
    if (start == (int)s.size()) return true;
    for (int end = start + 1; end <= (int)s.size(); end++) {
        if (dict.count(s.substr(start, end - start)) && helper(s, dict, end)) return true;
    }
    return false;
}
```

### Approach 2: Top-Down DP with Memoization — O(n² * L)

`helper(start)` has only n+1 distinct values. Memoize to avoid recomputation.

```cpp
bool helper(const string& s, unordered_set<string>& dict, vector<int>& memo, int start) {
    if (start == (int)s.size()) return true;
    if (memo[start] != -1) return memo[start];
    memo[start] = 0;
    for (int end = start + 1; end <= (int)s.size(); end++) {
        if (dict.count(s.substr(start, end - start)) && helper(s, dict, memo, end))
            return memo[start] = 1;
    }
    return memo[start];
}
```

### Approach 3: Bottom-Up Tabulation (Optimal) — O(n² * L)

**Recurrence:** `dp[i] = true` if `s[0..i-1]` can be segmented. `dp[0] = true` (empty prefix). `dp[i] = OR over j<i of (dp[j] AND s[j..i-1] ∈ dict)`.

```cpp
bool wordBreak(string s, vector<string>& wordDict) {
    int n = s.size();
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    vector<bool> dp(n + 1, false);
    dp[0] = true;

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            if (dp[j] && dict.count(s.substr(j, i - j))) { dp[i] = true; break; }
        }
    }
    return dp[n];
}
// Time: O(n^2 * L), Space: O(n)
```

### Optimization: Bound by Max Word Length

If `i - j > maxLen` (max dictionary word length), no point checking. Reduces inner loop from O(n) to O(maxLen).

```cpp
bool wordBreak(string s, vector<string>& wordDict) {
    int n = s.size();
    unordered_set<string> dict(wordDict.begin(), wordDict.end());
    int maxLen = 0;
    for (auto& w : wordDict) maxLen = max(maxLen, (int)w.size());

    vector<bool> dp(n + 1, false);
    dp[0] = true;
    for (int i = 1; i <= n; i++) {
        for (int j = max(0, i - maxLen); j < i; j++) {
            if (dp[j] && dict.count(s.substr(j, i - j))) { dp[i] = true; break; }
        }
    }
    return dp[n];
}
// Time: O(n * maxLen * L), Space: O(n)
```

### Dry Run

```
s="leetcode" (n=8), dict={"leet","code"}

dp[0]=T
i=1..3: no match. dp[1..3]=F.
i=4: j=0: dp[0]=T, s[0..3]="leet" ∈ dict → dp[4]=T.
i=5..7: no match.
i=8: j=4: dp[4]=T, s[4..7]="code" ∈ dict → dp[8]=T.

Return dp[8]=true  ✓
```

```
s="catsandog" (n=9), dict={"cats","dog","sand","and","cat"}

dp[0]=T
i=3: j=0: "cat" ∈ dict → dp[3]=T.
i=4: j=0: "cats" ∈ dict → dp[4]=T.
i=7: j=3: dp[3]=T, "sand" ∈ dict → dp[7]=T. (also j=4: dp[4]=T, "and" ∈ dict)
i=9: j=7: dp[7]=T, s[7..8]="og" not in dict.
     j=4: dp[4]=T, s[4..8]="andog" not in dict.
     j=3: dp[3]=T, s[3..8]="sandog" not in dict.
     No valid j → dp[9]=F.

Return false  ✓
```

### Why Greedy Matching Fails

Taking the first or longest matching word at each position is wrong. For `s="pineapple"`, `dict=["pine","apple","pineapple"]`: greedy might consume "pineapple" whole at the first step, missing that "pine"+"apple" also works, or vice versa fail on a wrong split. DP correctly tries all splits.

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `s="a"`, `dict=["a"]` | `true` | Single character match |
| `s="aaa"`, `dict=["a"]` | `true` | Word reuse allowed |
| `s="aaaaaab"`, `dict=["a","aa","aaa","aaaa"]` | `false` | Exponential trap without memo/DP |
| Empty dict | `false` | No words available |

---

## 9. Related Concepts and Interview Extensions

### From Sliding Window Maximum

- **Sliding Window Minimum:** Same algorithm, monotonic increasing deque (pop back when back value `>=` incoming).
- **Next Greater Element I & II (LC 496, 503):** Monotonic decreasing stack for "first greater element to the right."
- **Daily Temperatures (LC 739):** Days until a warmer temperature — monotonic decreasing stack of indices.
- **Largest Rectangle in Histogram (LC 84):** Nearest smaller bar on each side via monotonic increasing stack.
- **Trapping Rain Water (LC 42):** Solvable via monotonic stack or two pointers.

### From Stock Span

- **Sum of Subarray Minimums (LC 907):** For each element, count subarrays where it's the minimum — uses previous/next smaller element via monotonic stack.

### From Celebrity Problem

- **Find the Judge (LC 997):** Find the node with in-degree n-1, out-degree 0 in a directed graph — same structural idea.

### From LRU Cache

- **Design Browser History (LC 1472):** Simpler — only forward/back navigation.
- **All O(1) Data Structure (LC 432):** Similar design combining hash map with ordered structure.
- **Real-world LRU:** CPU page replacement, browser cache, database buffer pool, CDN edge caching.

### From LFU Cache

- **LFU vs LRU tradeoff:** LFU can retain stale-but-historically-popular items even when current traffic favors new keys. Adaptive Replacement Cache (ARC) blends both policies.

### From Word Break

- **Word Break II (LC 140):** Return all valid segmentations via DFS + memoization. Output can be exponential.
- **Concatenated Words (LC 472):** Each word must concatenate at least two shorter words from the same list — run Word Break per word.
- **Palindrome Partitioning (LC 131):** Same DP shape, different validity check (palindrome instead of dictionary membership).
- **Decode Ways (LC 91):** Same bottom-up shape: `dp[i] = dp[i-1] + dp[i-2]` under digit validity conditions.

---

## 10. Interview Reference: Patterns and Follow-ups

### Pattern Recognition Table

| Signal | Technique |
|---|---|
| Max/min in every window of size k | Monotonic deque, O(n) |
| Previous/next greater or smaller element | Monotonic stack, O(n) |
| Span/streak with comparison condition | Monotonic stack; (value,span) pairs for online queries |
| Unique element with in-degree n-1, out-degree 0 | Celebrity elimination, O(n) |
| Cache with O(1) ops, recency eviction | LRU: hash map + DLL |
| Cache with O(1) ops, frequency eviction | LFU: hash map + per-freq DLLs + minFreq |
| String segmentation into dictionary words | Bottom-up DP with hash set lookup |

### Common Follow-up Questions

**LRU Cache:**
- Thread safety? → Add a mutex around all operations.
- TTL support? → Store timestamp per node; check expiry on access; lazy-evict.
- Without explicit DLL? → Use `std::list` + `splice`, or Python `OrderedDict`.

**LFU Cache:**
- Why isn't LFU always better than LRU? → LFU can retain stale-but-frequently-used-in-the-past items even when current access patterns have shifted entirely to new keys.
- ARC? → Combines recency and frequency tracking in separate lists, adaptively weighting them.

**Monotonic Stack/Deque:**
- What invariant does the deque maintain? → Front-to-back non-increasing values (for max queries); front is the current maximum.
- Alternative for dynamic k? → Segment tree, O(n log n), supports changing window size.

**Celebrity Problem:**
- Multiple celebrities possible? → No — proven by contradiction (see Section 5).
- If `knows()` calls are expensive? → Track call count; elimination + verification uses at most 3(n-1) calls total.

**Word Break:**
- Relation to automata theory? → Dictionary words define a finite automaton; Word Break asks if `s` is accepted. Aho-Corasick speeds matching to O(n).
- Each word used at most once? → Becomes NP-complete in general (subset-sum-like).

---

## 11. Common Mistakes and Pitfalls

### Mistake 1: Sliding Window Max — Storing values instead of indices

```cpp
// WRONG: can't check expiry without position information
deque<int> dq;   // storing values
while (!dq.empty() && dq.front() <= i - k) dq.pop_front();   // comparing value to index — broken

// CORRECT: store indices
deque<int> dq;   // storing indices
while (!dq.empty() && dq.front() <= i - k) dq.pop_front();
```

### Mistake 2: Sliding Window Max — Using `<` instead of `<=` for back pop

```cpp
// WRONG: keeps redundant equal elements
while (!dq.empty() && nums[dq.back()] < nums[i]) dq.pop_back();

// CORRECT: equal elements are also dominated (new one lasts at least as long)
while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
```

### Mistake 3: Stock Span — Using `<` instead of `<=`

```cpp
// WRONG: undercounts days with equal prices
while (!st.empty() && arr[st.top()] < arr[i]) st.pop();

// CORRECT: span includes equal prices
while (!st.empty() && arr[st.top()] <= arr[i]) st.pop();
```

### Mistake 4: Celebrity Problem — Skipping verification

```cpp
// WRONG: elimination alone does not prove the candidate is a celebrity
return candidate;

// CORRECT: always verify
for (int i = 0; i < n; i++) {
    if (i == candidate) continue;
    if (mat[candidate][i] || !mat[i][candidate]) return -1;
}
return candidate;
```

### Mistake 5: LRU Cache — Forgetting to update recency on `get`

```cpp
// WRONG: get does not promote the node
int get(int key) {
    if (!mp.count(key)) return -1;
    return mp[key]->val;   // node stays at old position — violates LRU semantics
}

// CORRECT: get must move the node to MRU
int get(int key) {
    if (!mp.count(key)) return -1;
    Node* node = mp[key];
    removeNode(node);
    insertAfterHead(node);
    return node->val;
}
```

### Mistake 6: LRU Cache — Not storing the key inside the DLL node

```cpp
// WRONG: cannot evict from hash map without the key
struct Node { int val; Node *prev, *next; };
// On eviction: mp.erase(???)

// CORRECT: store both key and value
struct Node { int key, val; Node *prev, *next; };
// On eviction: mp.erase(lru->key);
```

### Mistake 7: LFU Cache — Not resetting `minFreq` to 1 on new insertion

```cpp
// WRONG: minFreq stays stale, eviction may target the wrong bucket
freqMap[1].push_front(key);
// missing: minFreq = 1;

// CORRECT
freqMap[1].push_front(key);
minFreq = 1;   // new key is always at the new global minimum
```

### Mistake 8: Word Break — Greedy matching instead of DP

```cpp
// WRONG: greedy fails when the first match leads to a dead end
// e.g., s="pineapple", dict=["pine","apple","pineapple"]
// Greedy taking "pineapple" first might miss valid splits, or taking "pine" first
// might fail to find "apple" if a different (wrong) word is tried first.

// CORRECT: DP explores all valid splits systematically
```

### Mistake 9: Word Break — dp array off-by-one

```cpp
// WRONG: dp[n-1] does not represent "whole string segmented"
vector<bool> dp(n, false);

// CORRECT: dp array of size n+1; dp[n] is the answer
vector<bool> dp(n + 1, false);
dp[0] = true;
```

---

## 12. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Sliding Window Max | Brute force | O(n*k) | O(1) |
| Sliding Window Max | Max heap (lazy) | O(n log k) | O(k) |
| Sliding Window Max | Monotonic deque | O(n) | O(k) |
| Stock Span | Brute force | O(n²) | O(1) |
| Stock Span | Monotonic stack (offline) | O(n) amortized | O(n) |
| Stock Span | (price,span) pairs (online) | O(n) amortized | O(n) |
| Celebrity | In/Out degree | O(n²) | O(n) |
| Celebrity | Stack elimination + verify | O(n) | O(n) |
| Celebrity | Two-pointer + verify | O(n) | O(1) |
| LRU Cache | Hash map + DLL | O(1) per op | O(capacity) |
| LFU Cache | Hash map + per-freq DLLs | O(1) per op | O(capacity) |
| Word Break | Brute recursion | O(n^n) | O(n) |
| Word Break | Memoization | O(n² * L) | O(n) |
| Word Break | Bottom-up DP | O(n² * L) | O(n) |
| Word Break | DP + maxLen bound | O(n * maxLen * L) | O(n) |

`L` = average word length, `n` = string or array length.

---

## 13. Quick Revision Cheat Sheet

Self-contained for same-day interview revision.

---

### Monotonic Deque (Sliding Window Max)

```cpp
deque<int> dq;   // indices, non-increasing values
for (int i = 0; i < n; i++) {
    while (!dq.empty() && dq.front() <= i - k) dq.pop_front();
    while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
    dq.push_back(i);
    if (i >= k - 1) result.push_back(nums[dq.front()]);
}
// Store INDICES not values. Use <= for both pops.
```

---

### Stock Span

```cpp
// Offline: span[i] = empty ? i+1 : i - st.top();  pop while arr[top]<=arr[i]
// Online:  stack of {price, span}; span = 1; absorb tops while top.price<=price
```

---

### Celebrity Problem

```cpp
int lo=0, hi=n-1;
while (lo < hi) {
    if (mat[lo][hi]) lo++; else hi--;
}
int candidate = lo;
// ALWAYS VERIFY with a full O(n) pass — elimination alone is not proof.
```

---

### LRU Cache

```cpp
// Hash map: key → DLL node. DLL: head=MRU, tail=LRU.
// Node MUST store key (needed for eviction from map).
// get: removeNode → insertAfterHead → return val.
// put (new, full): evict tail->prev from map+DLL; insert new node after head.
// put (existing): update val; removeNode → insertAfterHead.
```

---

### LFU Cache

```cpp
// keyMap{key→{val,freq}}, freqMap{freq→DLL}, iterMap{key→iterator}, minFreq.
// updateFreq: remove from freqMap[freq]; if empty && freq==minFreq: minFreq++;
//             insert at freqMap[freq+1].
// put (new, full): evict freqMap[minFreq].back(); insert at freqMap[1]; minFreq=1.
```

---

### Word Break

```cpp
vector<bool> dp(n+1, false); dp[0]=true;
for (int i=1; i<=n; i++)
    for (int j=max(0,i-maxLen); j<i; j++)
        if (dp[j] && dict.count(s.substr(j,i-j))) { dp[i]=true; break; }
return dp[n];
// dp array size n+1. Bound inner loop by maxLen for efficiency.
```

---

### Key Formulas

```
span[i] = i - PGE_index[i]   (or i+1 if no previous greater element)
dp[i]   = true if any dp[j] true AND s[j..i-1] ∈ dict
LRU evict = tail->prev
LFU evict = freqMap[minFreq].back()
Celebrity total comparisons ≤ 3(n-1)
Deque max size = k
```

---

### Templates to Recall Under Pressure

```cpp
// Monotonic deque
while (!dq.empty() && dq.front() <= i-k) dq.pop_front();
while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
dq.push_back(i);

// LRU helpers
void removeNode(Node* n) { n->prev->next=n->next; n->next->prev=n->prev; }
void insertAfterHead(Node* n) {
    n->next=head->next; n->prev=head;
    head->next->prev=n; head->next=n;
}

// Word Break DP
vector<bool> dp(n+1,false); dp[0]=true;
for (int i=1; i<=n; i++)
    for (int j=max(0,i-maxLen); j<i; j++)
        if (dp[j] && dict.count(s.substr(j,i-j))) { dp[i]=true; break; }
```
