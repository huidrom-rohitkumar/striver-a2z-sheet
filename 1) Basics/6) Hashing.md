
## Table of Contents

1. [The Mental Model: Why Hashing Exists](#1-the-mental-model-why-hashing-exists)
2. [Hash Functions and the Division Rule](#2-hash-functions-and-the-division-rule)
3. [Collisions and Collision Resolution](#3-collisions-and-collision-resolution)
4. [Load Factor and Rehashing](#4-load-factor-and-rehashing)
5. [Time Complexity: Average vs Worst Case](#5-time-complexity-average-vs-worst-case)
6. [C++ Standard Library: map vs unordered_map](#6-c-standard-library-map-vs-unordered_map)
7. [C++ Standard Library: set vs unordered_set](#7-c-standard-library-set-vs-unordered_set)
8. [Using unordered_map and unordered_set in Practice](#8-using-unordered_map-and-unordered_set-in-practice)
9. [The Hash Array Technique (Small Value Range)](#9-the-hash-array-technique-small-value-range)
10. [Problem: Counting Frequencies of Array Elements](#10-problem-counting-frequencies-of-array-elements)
11. [Problem: Frequency of the Most Frequent Element (LC 1838)](#11-problem-frequency-of-the-most-frequent-element-lc-1838)
12. [Anti-Hash Attack in Competitive Programming](#12-anti-hash-attack-in-competitive-programming)
13. [Advanced Hashing Concepts](#13-advanced-hashing-concepts)
14. [Related Problems Where Hashing is the Core Tool](#14-related-problems-where-hashing-is-the-core-tool)
15. [Interview Reference: Patterns and When to Reach for Hashing](#15-interview-reference-patterns-and-when-to-reach-for-hashing)
16. [Common Mistakes and Pitfalls](#16-common-mistakes-and-pitfalls)
17. [Complexity Cheat Sheet](#17-complexity-cheat-sheet)
18. [Quick Revision Cheat Sheet](#18-quick-revision-cheat-sheet)

---

## 1. The Mental Model: Why Hashing Exists

The fundamental problem hashing solves: given a key, answer "does this key exist, and what is its associated value?" in O(1) time rather than O(n) via linear scan or O(log n) via a binary search tree.

### The naive baselines

To count the frequency of every element in an array `arr` of size `n`:

- **Brute force nested loops:** For each element, scan the entire array to count matches. Time: O(n^2). Unusable for n = 10^5.
- **Sort + linear scan:** Sort (O(n log n)), then group adjacent identical elements in one pass. Total: O(n log n). Works, but destroys original order and adds a log factor.
- **Hashing:** One pass over the array, incrementing a counter for each element. O(n) time, O(n) space. Optimal.

### The core idea

Map every possible key to an integer index — the **hash** — and store the associated value at that index in an array. Looking up a key then costs one hash computation plus one array access, both O(1).

The challenge is that the universe of possible keys is enormous (all 64-bit integers, all strings) while the hash table is small. A **hash function** maps the large key space into a small range of indices. This mapping is many-to-one, so two distinct keys can map to the same index — a **collision**. Managing collisions correctly is the entire technical problem of hashing.

### The unifying primitive

Every hashing problem in this lecture reduces to: **build a frequency map (or presence map) in O(n), then query it in O(1) per query.** The map is one of:

- A raw array: when key range is small and non-negative (values 0 to ~10^6).
- `unordered_map<K, V>`: when key range is arbitrary or large, maximum speed needed.
- `map<K, V>`: when keys must be iterated in sorted order.

### The broader principle

Hashing is not just a data structure topic. It is a way of thinking:

```
Precompute information once  ->  avoid repeated work  ->  trade memory for time
```

This principle underlies frequency counting, two-sum, subarray-sum-equals-k, anagram grouping, memoization in DP, and graph adjacency using hash sets.

---

## 2. Hash Functions and the Division Rule

### What a hash function must do

A hash function `h(key)` maps a key to an index in `[0, m-1]` where `m` is the table size. Good hash functions satisfy:

1. **Deterministic:** Same key always maps to the same index.
2. **Uniform:** Keys spread evenly across all buckets, minimising collisions.
3. **Fast to compute:** O(1) in the key size.

### The Division Rule (Modular Hashing)

The simplest and most common hash function for integer keys:

```
h(key) = key % m
```

The key is taken modulo the table size `m`. The result is always in `[0, m-1]`.

**Example:** Table size m = 7.

```
h(23) = 23 % 7 = 2
h(14) = 14 % 7 = 0
h(35) = 35 % 7 = 0   <- collision with h(14)
h(9)  =  9 % 7 = 2   <- collision with h(23)
```

**Why choose m to be a prime?** If `m` is a power of 2, then `key % m` only uses the low-order bits of the key, completely ignoring high-order bits. If keys have patterns in the low bits (e.g., all multiples of 4), the distribution is terrible. A prime number has no divisibility structure, so the full key value influences the bucket. Choose a prime `m` not close to a power of 2 for the best distribution in practice.

**Example of bad m:** m = 10. All keys ending in the same digit collide. m = 101 (prime) is far better.

### Hash functions for strings

A common polynomial rolling hash:

```
h(s) = (s[0]*p^0 + s[1]*p^1 + ... + s[n-1]*p^(n-1)) % m
```

where `p` is a small prime (31 for lowercase letters, 37 for all ASCII). C++ `std::hash<std::string>` uses a similar scheme internally.

### The Multiplication Method (alternative)

```
h(key) = floor(m * frac(key * A))
```

where `A` is an irrational constant (often the golden ratio φ ≈ 0.618). Less sensitive to the choice of `m` but slightly slower to compute. Rarely used directly in competitive programming.

---

## 3. Collisions and Collision Resolution

A collision occurs when `h(k1) == h(k2)` for two distinct keys `k1 != k2`. No hash function can avoid all collisions — by the **pigeonhole principle**, mapping more keys than buckets forces at least one collision. Two primary strategies handle them.

### Strategy 1: Separate Chaining (Closed Addressing)

Each bucket holds a **linked list** (or dynamic array) of all key-value pairs that hash to it.

```
Bucket 0: [ (14, val_14) -> (35, val_35) -> null ]
Bucket 1: [ null ]
Bucket 2: [ (23, val_23) -> (9, val_9)  -> null ]
Bucket 3: [ null ]
```

- **Insert:** Compute `h(key)`, append to that bucket's list.
- **Lookup:** Compute `h(key)`, walk the list comparing keys.
- **Delete:** Compute `h(key)`, find and remove the node.

**This is exactly what C++ `std::unordered_map` uses internally.**

Advantages: simple, naturally handles load factors > 1, no constraint on table fill.
Disadvantages: each chain node is a heap allocation — poor cache locality when chains are long.

| Case | Complexity |
|------|------------|
| Average (uniform distribution) | O(1) |
| Worst (all keys in one bucket) | O(n) |

### Strategy 2: Open Addressing

All key-value pairs live inside the table array itself. On collision, probe for the next empty slot.

**Linear Probing:**
```
h(key, i) = (h(key) + i) % m     for i = 0, 1, 2, ...
```

**Quadratic Probing:**
```
h(key, i) = (h(key) + c1*i + c2*i^2) % m
```

**Double Hashing** (best among open addressing variants):
```
h(key, i) = (h1(key) + i * h2(key)) % m
```

**Primary clustering problem with linear probing:** When many keys hash to nearby slots, long contiguous runs of occupied slots form. Inserting into these runs requires scanning past the cluster, and the cluster grows with each insertion, progressively degrading performance.

**Why `unordered_map` uses chaining instead of open addressing:** Chaining supports a stable iterator API (open addressing invalidates iterators on rehash), handles arbitrary load factors, and is simpler to implement correctly. In practice, custom flat hash maps (like `gp_hash_table` in GCC's Policy-Based Data Structures) use open addressing and outperform `unordered_map` by 2–3x due to cache locality.

---

## 4. Load Factor and Rehashing

The **load factor** α is:

```
α = n / m       (elements stored / buckets in table)
```

- For **chaining**: α can exceed 1. Expected chain length = α. Lookup is O(1 + α). If α is bounded by a constant, all operations are O(1).
- For **open addressing**: α must stay below 1. Performance degrades sharply as α → 1. In practice, maintain α ≤ 0.7.

### Rehashing

When α exceeds a threshold (default 1.0 for `std::unordered_map`), a new array of roughly double the size (ideally the next prime ≥ 2m) is allocated and every existing key is re-inserted. This costs O(n) for one rehash, but since it occurs only when the element count doubles, the **amortised cost per insertion is O(1)**.

**Why amortised O(1)?** Starting from size 1, rehashes occur at element counts 1, 2, 4, 8, ..., n. Total work = n + n/2 + n/4 + ... = 2n = O(n). So n insertions cost O(n) total, O(1) amortised each.

### Controlling load factor in C++

```cpp
unordered_map<int,int> mp;
mp.reserve(1 << 20);       // pre-allocate enough buckets to avoid rehash
mp.max_load_factor(0.25);  // trigger rehash earlier for fewer collisions
```

`reserve` before bulk insertion eliminates the rehash overhead during build and reduces collision probability.

---

## 5. Time Complexity: Average vs Worst Case

This is the most commonly misunderstood point about hashing in interviews.

| Operation | Average Case | Worst Case | When worst case occurs |
|-----------|-------------|------------|------------------------|
| Insert | O(1) | O(n) | All keys collide into one bucket |
| Lookup | O(1) | O(n) | All keys collide into one bucket |
| Delete | O(1) | O(n) | All keys collide into one bucket |

**Why O(1) average?** Under the **Simple Uniform Hashing Assumption** — every key is equally likely to hash to any bucket, independently — the expected chain length at any bucket is α = n/m. If m = Θ(n), then α = O(1), so each operation takes O(1) expected time.

**Why O(n) worst case?** If all n keys happen to hash to the same bucket, the chain at that bucket has length n. Searching it is O(n). In competitive programming, adversaries can craft inputs that trigger this for any fixed hash function.

**Correct interview phrasing:** "O(1) average case, O(n) worst case." If asked why, explain: the average assumes uniform distribution; worst case occurs when all keys collide.

---

## 6. C++ Standard Library: map vs unordered_map

| Property | `map<K,V>` | `unordered_map<K,V>` |
|----------|-----------|----------------------|
| Internal structure | Red-Black Tree | Hash Table (chaining) |
| Insert | O(log n) | O(1) avg, O(n) worst |
| Lookup | O(log n) | O(1) avg, O(n) worst |
| Delete | O(log n) | O(1) avg, O(n) worst |
| Iteration order | Sorted by key | Unspecified |
| Memory per element | Higher (tree node with 3 pointers + color bit) | Lower |
| Iterator invalidation | Never on insert/erase (except erased element) | On rehash: all iterators invalidated |
| Custom key type requires | `operator<` | `operator==` and `std::hash<K>` specialisation |
| Header | `<map>` | `<unordered_map>` |

### When to use `map`

- Keys must be iterated in sorted order (range queries, ordered output).
- Guaranteed O(log n) worst case is required (adversarial inputs, competitive programming).
- Key type lacks a natural hash (`pair<int,int>`, `vector<int>`).

### When to use `unordered_map`

- Maximum speed on typical inputs.
- Key type is a built-in: `int`, `long long`, `string`, `char`.
- Sorted iteration is not needed.
- Constant factor matters: `unordered_map` is ~5–10x faster than `map` for integer keys.

### `map` usage

```cpp
#include <map>
map<int, int> mp;

mp[5] = 10;             // insert or update; creates entry if key absent
mp[5]++;                // increment
mp.count(5);            // 1 if key exists, 0 otherwise — O(log n), does NOT insert
mp.find(5);             // iterator to element, or mp.end() if not found
mp.erase(5);            // remove key 5

// Iteration is in sorted key order
for (auto& [key, val] : mp) {
    cout << key << " -> " << val << "\n";
}
```

### `unordered_map` usage

```cpp
#include <unordered_map>
unordered_map<int, int> mp;

mp[5] = 10;
mp[5]++;
mp.count(5);            // O(1) avg, does NOT insert
mp.find(5);             // O(1) avg
mp.erase(5);            // O(1) avg

// Iteration order is NOT sorted and NOT insertion order
for (auto& [key, val] : mp) {
    cout << key << " -> " << val << "\n";
}
```

### The default-initialisation trap — critical

Accessing a key that does not exist via `operator[]` **inserts** it with a default-constructed value (0 for `int`). This is intentional for frequency counting (`mp[x]++` works even if `x` was never seen), but it silently grows the map when you only want to check existence.

```cpp
unordered_map<int,int> freq;

// WRONG for existence check: inserts 99->0 if absent
if (freq[99] > 0) { ... }

// CORRECT: does not insert
if (freq.count(99)) { ... }
if (freq.find(99) != freq.end()) { ... }
```

---

## 7. C++ Standard Library: set vs unordered_set

When you need to track only **presence** (not a count), use a set.

| Property | `set<K>` | `unordered_set<K>` |
|----------|---------|---------------------|
| Internal structure | Red-Black Tree | Hash Table |
| Insert / Lookup / Delete | O(log n) | O(1) avg, O(n) worst |
| Iteration order | Sorted | Unspecified |
| Duplicates | Not stored | Not stored |

```cpp
#include <unordered_set>
unordered_set<int> seen;
seen.insert(5);
seen.count(5);     // 1 if present — does NOT insert
seen.erase(5);
bool exists = seen.find(5) != seen.end();
```

Use `unordered_set` when you only need "have I seen x?" and not a count. Use `set` when you need sorted iteration or guaranteed worst-case.

---

## 8. Using unordered_map and unordered_set in Practice

### Iterating over an unordered_map

```cpp
unordered_map<int,int> freq;
// ... populate ...

// C++17 structured binding (clearest)
for (auto& [key, val] : freq) {
    cout << key << ": " << val << "\n";
}

// Pre-C++17 with pair
for (auto& p : freq) {
    cout << p.first << ": " << p.second << "\n";
}
```

### Checking existence without inserting

```cpp
// CORRECT
if (freq.count(x)) { /* x is present, and freq[x] > 0 */ }
if (freq.find(x) != freq.end()) { /* x is present */ }

// WRONG: silently inserts x->0 if x was absent
if (freq[x]) { ... }
```

### Custom hash for pair<int,int> keys

The standard library does not provide `std::hash` for `pair`. Define a custom hash:

```cpp
struct PairHash {
    size_t operator()(const pair<int,int>& p) const {
        // Pack both ints into one 64-bit value, then hash
        return hash<long long>{}((long long)p.first << 32 | (unsigned int)p.second);
    }
};
unordered_map<pair<int,int>, int, PairHash> mp;
mp[{1, 2}] = 5;  // compiles and works
```

---

## 9. The Hash Array Technique (Small Value Range)

When all values are bounded (e.g., `1 <= arr[i] <= 10^6`), a direct array beats any hash map.

```cpp
const int MAXVAL = 1e6 + 5;
int freq[MAXVAL] = {};   // global zero-initialised array

void buildFreq(const vector<int>& arr) {
    for (int x : arr) freq[x]++;
}

int queryFreq(int x) {
    return freq[x];  // O(1), no hash computation
}
```

**Advantages over `unordered_map`:**
- Zero collision risk.
- Cache-friendly — contiguous memory access.
- No hash computation overhead.
- Faster constant factor: typically 5–10x faster than `unordered_map`.

**Disadvantages:**
- Memory: O(MAXVAL) regardless of how many distinct values appear. `MAXVAL = 10^9` means ~4 GB — impossible.
- Only works for non-negative integer keys in a known, small range.
- Cannot handle keys outside `[0, MAXVAL]`.

**Resetting between test cases efficiently:**

```cpp
// SLOW: memset over entire array every test case — O(MAXVAL)
memset(freq, 0, sizeof(freq));

// FAST: only reset positions you touched — O(n per test case)
vector<int> touched;
for (int x : arr) {
    if (freq[x] == 0) touched.push_back(x);
    freq[x]++;
}
// ... use freq ...
for (int x : touched) freq[x] = 0;  // selective reset
```

---

## 10. Problem: Counting Frequencies of Array Elements

### Problem Statement

Given an integer array `arr` of size `n`, print the frequency of each element. Each element and its frequency should appear exactly once. Values can be negative or arbitrarily large.

```
Input:  arr = [10, 20, 10, 5, 20]
Output: 10->2, 20->2, 5->1   (order may vary)

Input:  arr = [1, 2, 3, 2, 1, 3, 4, 5, 4]
Output: 1->2, 2->2, 3->2, 4->2, 5->1
```

Constraints: `1 <= n <= 10^5`, values can be arbitrary integers.

### Approach 1: Brute Force — Nested Loops

For each element, scan the rest of the array. Use a `visited` array to avoid recounting.

```cpp
void countFreqBrute(vector<int>& arr) {
    int n = arr.size();
    vector<bool> visited(n, false);

    for (int i = 0; i < n; i++) {
        if (visited[i]) continue;

        int count = 1;
        for (int j = i + 1; j < n; j++) {
            if (!visited[j] && arr[j] == arr[i]) {
                count++;
                visited[j] = true;
            }
        }
        cout << arr[i] << " -> " << count << "\n";
    }
}
```

**Dry Run (arr = [1, 2, 1, 3]):**

```
i=0: arr[0]=1, not visited
  j=1: arr[1]=2 != 1
  j=2: arr[2]=1 == 1, count=2, visited[2]=true
  j=3: arr[3]=3 != 1
  Print: 1 -> 2

i=1: arr[1]=2, not visited
  j=2: visited[2]=true, skip
  j=3: arr[3]=3 != 2
  Print: 2 -> 1

i=2: visited[2]=true, skip

i=3: arr[3]=3, not visited
  Print: 3 -> 1

Output: 1->2, 2->1, 3->1  (correct)
```

**Complexity:** Time O(n^2), Space O(n). Unusable for n > 10^4.

### Approach 2: Sort + Linear Scan

Sort the array (identical elements become adjacent), then count runs.

```cpp
void countFreqSort(vector<int> arr) {   // pass by value to avoid mutating original
    sort(arr.begin(), arr.end());
    int n = arr.size();
    for (int i = 0; i < n; ) {
        int j = i;
        while (j < n && arr[j] == arr[i]) j++;
        cout << arr[i] << " -> " << (j - i) << "\n";
        i = j;
    }
}
```

**Dry Run (arr = [3, 1, 2, 1, 3]):**

```
After sort: [1, 1, 2, 3, 3]

i=0: arr[0]=1, j advances to 2, count = 2-0 = 2  -> Print: 1->2
i=2: arr[2]=2, j advances to 3, count = 1         -> Print: 2->1
i=3: arr[3]=3, j advances to 5, count = 2         -> Print: 3->2
```

**Complexity:** Time O(n log n), Space O(1). Destroys original order.

### Approach 3: unordered_map (Optimal — arbitrary keys)

Single pass. For each element, increment its count in the map.

```cpp
void countFreqHash(const vector<int>& arr) {
    unordered_map<int, int> freq;
    for (int x : arr) freq[x]++;

    for (auto& [key, val] : freq) {
        cout << key << " -> " << val << "\n";
    }
}
// Time: O(n) avg, Space: O(n)
```

**If sorted output is required, use `map`:**

```cpp
void countFreqHashSorted(const vector<int>& arr) {
    map<int, int> freq;
    for (int x : arr) freq[x]++;
    for (auto& [key, val] : freq) {
        cout << key << " -> " << val << "\n";
    }
}
// Time: O(n log n) due to map insertions
```

**Dry Run — unordered_map (arr = [3, 1, 2, 1, 3, 2, 2]):**

```
Process 3: freq = {3:1}
Process 1: freq = {3:1, 1:1}
Process 2: freq = {3:1, 1:1, 2:1}
Process 1: freq = {3:1, 1:2, 2:1}
Process 3: freq = {3:2, 1:2, 2:1}
Process 2: freq = {3:2, 1:2, 2:2}
Process 2: freq = {3:2, 1:2, 2:3}

Output (any order): 3->2, 1->2, 2->3  (correct)
```

### Approach 4: Hash Array (small non-negative range)

```cpp
void countFreqArray(const vector<int>& arr, int MAXVAL) {
    vector<int> freq(MAXVAL + 1, 0);
    for (int x : arr) freq[x]++;
    for (int i = 0; i <= MAXVAL; i++) {
        if (freq[i] > 0) cout << i << " -> " << freq[i] << "\n";
    }
}
// Time: O(n + MAXVAL), Space: O(MAXVAL)
```

### Answering frequency queries after building the map

```cpp
unordered_map<int,int> freq;
for (int x : arr) freq[x]++;

// Safe query: returns 0 if key never appeared
auto query = [&](int x) -> int {
    auto it = freq.find(x);
    return (it != freq.end()) ? it->second : 0;
};
```

### Complexity Comparison

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute force nested loops | O(n^2) | O(n) | Unusable for n > 10^4 |
| Sort + linear scan | O(n log n) | O(1) | Destroys original order |
| `unordered_map` | O(n) avg | O(n) | Best general solution |
| `map` (sorted output) | O(n log n) | O(n) | Use when sorted output needed |
| Hash array (small range) | O(n + MAXVAL) | O(MAXVAL) | Fastest constant; only for bounded non-negative values |

### Edge Cases

| Input | Expected | Reason |
|-------|----------|--------|
| All elements identical | One entry with freq n | Map has one key |
| All elements distinct | n entries each with freq 1 | |
| Array of size 1 | One entry with freq 1 | |
| Negative values | Works with map/unordered_map; not with hash array | Array index cannot be negative |
| Values up to 10^9 | Must use map or unordered_map | Array of size 10^9 is ~4 GB |

---

## 11. Problem: Frequency of the Most Frequent Element (LC 1838)

### Problem Statement

**LeetCode 1838 — Frequency of the Most Frequent Element (Medium)**

The frequency of an element is the number of times it occurs. Given an integer array `nums` and integer `k`, in one operation you may choose any index of `nums` and increment the element there by 1. You may perform at most `k` operations total. Return the maximum possible frequency of any element after performing at most `k` operations.

```
Input: nums = [1,2,4],   k = 5   ->  Output: 3
  Explanation: Increment 1->4 (3 ops), 2->4 (2 ops). Total: 5 ops. All three = 4.

Input: nums = [1,4,8,13], k = 5  ->  Output: 2
  Explanation: Best is frequency 2. E.g., 1->4 (3 ops), giving [4,4,8,13].

Input: nums = [3,9,6],    k = 2  ->  Output: 1
  Explanation: Min gap is 9-6 = 3 > 2. Cannot make any two elements equal.
```

Constraints: `1 <= nums.length <= 10^5`, `1 <= nums[i] <= 10^5`, `1 <= k <= 10^5`.

### Key Observations

**Observation 1:** You can only increment, never decrement. To maximise the frequency of a target `t`, every element you bring to `t` must already be ≤ `t`. You should always bring smaller elements up, never try to bring a large element down.

**Observation 2:** The optimal target is always an existing value in the array, specifically the largest element in the candidate group. Choosing any non-array value as target wastes operations. Sorting makes this exploitable.

**Observation 3 — cost formula:** Given a sorted window `nums[l..r]`, the total number of increment operations needed to make every element equal to `nums[r]` is:

```
cost = nums[r] * (r - l + 1) - sum(nums[l..r])
```

This equals (target × window size) minus (current window sum) — the total number of increments needed across all elements.

**Why the formula works:** Each element `nums[i]` in the window needs `nums[r] - nums[i]` increments to reach `nums[r]`. Summing: `sum(nums[r] - nums[i]) = nums[r] * window_size - window_sum`.

**Observation 4:** We want the largest window `[l, r]` such that `cost <= k`.

### Approach 1: Brute Force — Try Each Element as Target

Sort the array. For each right endpoint `r`, expand left as far as possible while `cost <= k`.

```cpp
int maxFrequency_brute(vector<int>& nums, int k) {
    sort(nums.begin(), nums.end());
    int n = nums.size(), ans = 1;

    for (int r = 1; r < n; r++) {
        long long total = 0;
        for (int l = r - 1; l >= 0; l--) {
            total += (long long)(nums[r] - nums[l]);
            if (total <= k) ans = max(ans, r - l + 1);
            else break;  // further left only increases cost
        }
    }
    return ans;
}
// Time: O(n^2), Space: O(1) ignoring sort
```

**Why break early?** After sorting, `nums[l]` decreases as `l` moves left, so `nums[r] - nums[l]` increases monotonically. Once cost > k, no further left expansion can help.

### Approach 2: Sliding Window (Optimal)

Maintain a window `[l, r]` and a running window sum. Expand right (add `nums[r]`). If cost > k, shrink from the left (remove `nums[l]`). Track maximum window size.

The key insight enabling the sliding window: after sorting, as `r` increases, the window size is monotonically non-decreasing. We never need to shrink below the current best answer, so the window only ever slides right — the total movement of `l` across the entire loop is O(n).

```cpp
int maxFrequency(vector<int>& nums, int k) {
    sort(nums.begin(), nums.end());
    int n = nums.size();

    long long windowSum = 0;
    int l = 0, ans = 1;

    for (int r = 0; r < n; r++) {
        windowSum += nums[r];   // expand window to the right

        // Cost to make entire window equal to nums[r]:
        //   nums[r] * windowSize - windowSum
        // Shrink from left until cost <= k
        while ((long long)nums[r] * (r - l + 1) - windowSum > k) {
            windowSum -= nums[l];
            l++;
        }

        // Window [l, r] is valid: all can be raised to nums[r] with <= k ops
        ans = max(ans, r - l + 1);
    }
    return ans;
}
// Time: O(n log n), Space: O(1)
```

**Why the while loop is O(n) total:** `l` starts at 0 and only ever moves right. Across all iterations of the outer `for` loop, `l` advances at most `n` times total. Each element enters and leaves the window exactly once.

### Detailed Dry Run — Sliding Window (nums = [1, 2, 4], k = 5)

```
After sort: [1, 2, 4]
l=0, windowSum=0, ans=1

r=0: windowSum = 0+1 = 1
  cost = nums[0]*(0-0+1) - 1 = 1*1 - 1 = 0 <= 5, no shrink
  ans = max(1, 0-0+1) = 1
  Window: [1], target=1

r=1: windowSum = 1+2 = 3
  cost = nums[1]*(1-0+1) - 3 = 2*2 - 3 = 1 <= 5, no shrink
  ans = max(1, 1-0+1) = 2
  Window: [1,2], target=2  (1 op: 1->2)

r=2: windowSum = 3+4 = 7
  cost = nums[2]*(2-0+1) - 7 = 4*3 - 7 = 5 <= 5, no shrink
  ans = max(2, 2-0+1) = 3
  Window: [1,2,4], target=4  (5 ops: 1->4 costs 3, 2->4 costs 2)

Final ans = 3  (correct)
```

### Detailed Dry Run — Sliding Window (nums = [1, 4, 8, 13], k = 5)

```
After sort: [1, 4, 8, 13]
l=0, windowSum=0, ans=1

r=0: windowSum=1,  cost=1*1-1=0<=5, ans=1,  window=[1]
r=1: windowSum=5,  cost=4*2-5=3<=5, ans=2,  window=[1,4]
r=2: windowSum=13
  cost = 8*3 - 13 = 11 > 5 -> shrink:
    windowSum -= nums[0]=1 -> windowSum=12, l=1
  cost = 8*2 - 12 = 4 <= 5, ans=max(2,2)=2, window=[4,8]
r=3: windowSum=25
  cost = 13*3 - 25 = 14 > 5 -> shrink:
    windowSum -= nums[1]=4 -> windowSum=21, l=2
  cost = 13*2 - 21 = 5 <= 5, ans=max(2,2)=2, window=[8,13]

Final ans = 2  (correct)
```

### Approach 3: Binary Search on Answer (Alternative O(n log n))

Binary search on the target frequency `m`. For a given `m`, check if any sorted window of size exactly `m` has cost ≤ k using prefix sums. The feasibility function is monotone (if `m` is achievable, any smaller value is too), so binary search applies.

```cpp
bool canAchieve(vector<int>& nums, vector<long long>& prefix, int m, long long k) {
    int n = nums.size();
    for (int r = m - 1; r < n; r++) {
        int l = r - m + 1;
        long long windowSum = prefix[r + 1] - prefix[l];
        if ((long long)nums[r] * m - windowSum <= k) return true;
    }
    return false;
}

int maxFrequency_bsearch(vector<int>& nums, int k) {
    sort(nums.begin(), nums.end());
    int n = nums.size();

    vector<long long> prefix(n + 1, 0);
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] + nums[i];

    int lo = 1, hi = n, ans = 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (canAchieve(nums, prefix, mid, k)) {
            ans = mid;
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
}
// Time: O(n log n), Space: O(n) for prefix sums
```

The binary search approach is more expensive in space and no better in time than the sliding window. Prefer the sliding window. Know the binary search approach as an alternative for when the "sliding window is optimal" reasoning is not immediately obvious.

### Complexity Comparison

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Brute force | O(n^2) | O(1) | TLE for n = 10^5 |
| Sliding window | O(n log n) | O(1) | Optimal |
| Binary search on answer | O(n log n) | O(n) | Extra space for prefix; same time |

### Edge Cases

| Input | Expected | Reason |
|-------|----------|--------|
| `nums=[1], k=5` | 1 | Single element, trivially frequency 1 |
| `nums=[1,1,1], k=0` | 3 | All identical already, zero ops needed |
| `nums=[1,100000], k=1` | 1 | Gap 99999 > 1; cannot make equal |
| k very large | n | Entire array can be raised to max |
| `nums=[3,9,6], k=2` | 1 | Min gap between any two is 9-6=3 > 2 |

---

## 12. Anti-Hash Attack in Competitive Programming

### The attack

GCC's `unordered_map` uses modular hashing with a fixed internal prime (107897 or 126271 depending on the judge). An adversary who knows this prime can insert its multiples as keys — all map to bucket 0 — degrading every operation to O(n). An O(n) intended solution becomes O(n^2) and TLEs.

### The defence: splitmix64 custom hash

```cpp
struct SafeHash {
    static uint64_t splitmix64(uint64_t x) {
        x += 0x9e3779b97f4a7c15ULL;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9ULL;
        x = (x ^ (x >> 27)) * 0x94d049bb133111ebULL;
        return x ^ (x >> 31);
    }
    size_t operator()(uint64_t x) const {
        // FIXED_RANDOM changes every run, making the attack impossible
        static const uint64_t FIXED_RANDOM =
            chrono::steady_clock::now().time_since_epoch().count();
        return splitmix64(x + FIXED_RANDOM);
    }
};

unordered_map<int, int, SafeHash> mp;
unordered_set<int, SafeHash> st;
```

`FIXED_RANDOM` changes every run — the adversary cannot precompute collisions without knowing the seed. `splitmix64` is a high-quality bijective mixing function that spreads keys uniformly regardless of their structure.

**When to use this:** Any time `unordered_map` or `unordered_set` appears in a solution submitted to Codeforces or AtCoder, where adversarial test data is common. Not needed on LeetCode (test data is not adversarially crafted against hash functions).

**Alternative:** Use `map` when the O(log n) factor is acceptable — Red-Black Trees have guaranteed O(log n) worst case, immune to hash attacks.

---

## 13. Advanced Hashing Concepts

### String Hashing (Polynomial Rolling Hash)

Used in Rabin-Karp pattern matching and substring comparison. Computes a hash for every prefix of a string, enabling O(1) hash computation for any substring.

```cpp
// Precompute prefix hashes and powers
const long long MOD = 1e9 + 7, BASE = 31;
int n = s.size();
vector<long long> h(n + 1, 0), pw(n + 1, 1);
for (int i = 0; i < n; i++) {
    h[i + 1] = (h[i] * BASE + (s[i] - 'a' + 1)) % MOD;
    pw[i + 1] = pw[i] * BASE % MOD;
}
// Hash of substring s[l..r] (0-indexed, inclusive):
auto getHash = [&](int l, int r) -> long long {
    return (h[r + 1] - h[l] * pw[r - l + 1] % MOD + MOD * MOD) % MOD;
};
```

**Double hashing** (two independent hash functions with different MOD and BASE) reduces the probability of false positives to negligible levels.

### Coordinate Compression

Maps a large sparse set of values (e.g., values up to 10^9) into a compact range `[0, k-1]` where `k` is the number of distinct values. Enables using array-based hashing or segment trees on large values.

```cpp
vector<int> vals = {1, 1000000000, 500, 42};
sort(vals.begin(), vals.end());
vals.erase(unique(vals.begin(), vals.end()), vals.end());
// vals = {42, 500, 1000000000} (sorted, deduplicated)
// compressed index of x: lower_bound(vals.begin(), vals.end(), x) - vals.begin()
auto compress = [&](int x) {
    return (int)(lower_bound(vals.begin(), vals.end(), x) - vals.begin());
};
```

---

## 14. Related Problems Where Hashing is the Core Tool

### Two Sum (LC 1) — canonical O(n) hashing application

Store each value seen so far. For current element `x`, check if `target - x` has been seen.

```cpp
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int,int> seen;  // value -> index
    for (int i = 0; i < (int)nums.size(); i++) {
        int complement = target - nums[i];
        if (seen.count(complement)) return {seen[complement], i};
        seen[nums[i]] = i;
    }
    return {};
}
// Time: O(n), Space: O(n)
```

### Subarray Sum Equals K (LC 560) — prefix sum + hashing

Count subarrays with sum exactly `k`. For each prefix sum `S[i]`, the number of subarrays ending at `i` with sum `k` equals the count of previously seen prefix sums equal to `S[i] - k`.

```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int,int> prefixCount;
    prefixCount[0] = 1;   // empty prefix contributes sum 0
    int sum = 0, count = 0;
    for (int x : nums) {
        sum += x;
        count += prefixCount[sum - k];
        prefixCount[sum]++;
    }
    return count;
}
// Time: O(n), Space: O(n)
```

### Longest Consecutive Sequence (LC 128) — O(n) via unordered_set

Insert all elements into a set. For each element that is the start of a sequence (i.e., `x-1` not in set), count the sequence length.

```cpp
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> s(nums.begin(), nums.end());
    int ans = 0;
    for (int x : nums) {
        if (!s.count(x - 1)) {   // x is the start of a sequence
            int cur = x;
            while (s.count(cur + 1)) cur++;
            ans = max(ans, cur - x + 1);
        }
    }
    return ans;
}
// Time: O(n), Space: O(n)
```

### Group Anagrams (LC 49) — hash map with sorted string key

Sort each word's characters to produce a canonical key. Group words by key.

```cpp
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    unordered_map<string, vector<string>> groups;
    for (auto& s : strs) {
        string key = s;
        sort(key.begin(), key.end());
        groups[key].push_back(s);
    }
    vector<vector<string>> result;
    for (auto& [key, group] : groups) result.push_back(group);
    return result;
}
// Time: O(n * L log L) where L = max string length
```

### Sliding Window + Hash Map (Longest Substring Without Repeating Characters, LC 3)

Maintain a window where each character appears at most once. Use a hash map to track last-seen index.

```cpp
int lengthOfLongestSubstring(string s) {
    unordered_map<char,int> lastSeen;
    int l = 0, ans = 0;
    for (int r = 0; r < (int)s.size(); r++) {
        if (lastSeen.count(s[r]) && lastSeen[s[r]] >= l) {
            l = lastSeen[s[r]] + 1;  // shrink window past last occurrence
        }
        lastSeen[s[r]] = r;
        ans = max(ans, r - l + 1);
    }
    return ans;
}
// Time: O(n), Space: O(1) — only 128 possible ASCII chars
```

---

## 15. Interview Reference: Patterns and When to Reach for Hashing

### Pattern recognition table

| Problem signal | Appropriate technique |
|---------------|----------------------|
| "Check if element exists" | `unordered_set`, O(1) lookup |
| "Count occurrences of each element" | `unordered_map<val, count>`, O(n) build |
| "Find two elements summing to target" | `unordered_set` or `unordered_map`, O(n) |
| "First non-repeating / first duplicate" | `unordered_map`, one pass |
| "Group elements by some property" | `unordered_map<property, vector<element>>` |
| "Count subarrays with given sum / XOR" | Prefix + `unordered_map`, O(n) |
| "Longest subarray with constraint" | Sliding window + `unordered_map` |
| "k most frequent elements" | Frequency map + heap or bucket sort |
| "All values in range [0, MAXVAL]" | Direct hash array `freq[MAXVAL+1]` |
| "Need sorted output" | `map` (O(log n)), not `unordered_map` |
| "Adversarial / competitive programming" | `map` or `unordered_map` + custom hash |

### Data structure selection by key type

| Key type | Use |
|----------|-----|
| `int`, `long long` | `unordered_map<int, V>` directly |
| `string` | `unordered_map<string, V>` directly |
| `char` | `unordered_map<char, V>` or `int freq[128]` |
| `pair<int,int>` | Custom hash struct required |
| `vector<int>` | Custom hash struct required |
| Need sorted order | `map<K,V>` with `operator<` |

---

## 16. Common Mistakes and Pitfalls

### Mistake 1: Using operator[] for existence check — silently inserts

```cpp
unordered_map<int,int> freq;
// ... build freq ...

// WRONG: inserts 99->0 if key absent
if (freq[99] > 0) { cout << "exists\n"; }

// CORRECT
if (freq.count(99)) { cout << "exists\n"; }
if (freq.find(99) != freq.end()) { cout << "exists\n"; }
```

### Mistake 2: Using hash array for negative or large-range values

```cpp
int arr[] = {-5, 3, 100000000};
int freq[100] = {};    // WRONG: arr[-5] is undefined behaviour; 100000000 overflows buffer

// CORRECT: use unordered_map for arbitrary-range values
unordered_map<int,int> freq;
for (int x : arr) freq[x]++;
```

### Mistake 3: Integer overflow in the sliding window cost formula

```cpp
// WRONG: nums[r] * (r-l+1) overflows int when both are ~10^5
int cost = nums[r] * (r - l + 1) - windowSum;

// CORRECT: cast to long long before multiplication
long long cost = (long long)nums[r] * (r - l + 1) - windowSum;
```

### Mistake 4: Forgetting to sort before sliding window in LC 1838

The cost formula `nums[r] * windowSize - windowSum` is only meaningful when the array is sorted and `nums[r]` is the maximum in the window. On an unsorted array, the formula produces incorrect values.

```cpp
// ALWAYS sort before applying the sliding window for this problem
sort(nums.begin(), nums.end());
```

### Mistake 5: Stating "O(1) space" for a hash-map-based solution

The hash map itself takes O(n) space in the worst case (n distinct keys). O(n) space for the map is the correct answer, not O(1).

### Mistake 6: Claiming "unordered_map is O(1)" without qualification

The correct statement is "O(1) average case, O(n) worst case." Always say both in interviews and explain when the worst case is a concern.

### Mistake 7: Using pair<int,int> as unordered_map key without a custom hash

```cpp
// WRONG: does not compile — no std::hash<pair<int,int>> in standard library
unordered_map<pair<int,int>, int> mp;

// CORRECT: provide custom hash struct
struct PairHash {
    size_t operator()(const pair<int,int>& p) const {
        return hash<long long>{}((long long)p.first << 32 | (unsigned int)p.second);
    }
};
unordered_map<pair<int,int>, int, PairHash> mp;
```

### Mistake 8: Assuming unordered_map iteration order equals insertion order

```cpp
unordered_map<int,int> mp;
mp[3] = 1; mp[1] = 2; mp[2] = 3;
for (auto& [k,v] : mp) cout << k << " ";
// Output order is unspecified and hash-value-dependent; do NOT rely on it
```

### Mistake 9: Forgetting to handle the case where a queried key was never inserted

```cpp
// WRONG: returns 0 silently but INSERTS 99->0 into the map as a side effect
return freq[99];

// CORRECT
auto it = freq.find(99);
return (it != freq.end()) ? it->second : 0;
```

---

## 17. Complexity Cheat Sheet

### Counting Frequencies

| Approach | Time | Space |
|----------|------|-------|
| Brute force nested loops | O(n^2) | O(n) |
| Sort + linear scan | O(n log n) | O(1) |
| `unordered_map` | O(n) avg | O(n) |
| `map` (sorted output) | O(n log n) | O(n) |
| Hash array (small range) | O(n + MAXVAL) | O(MAXVAL) |

### Frequency of Most Frequent Element (LC 1838)

| Approach | Time | Space |
|----------|------|-------|
| Brute force | O(n^2) | O(1) |
| Sliding window | O(n log n) | O(1) |
| Binary search on answer | O(n log n) | O(n) |

### Container Operations

| Container | Insert | Lookup | Delete | Iteration | Order |
|-----------|--------|--------|--------|-----------|-------|
| `unordered_map` | O(1) avg, O(n) worst | O(1) avg, O(n) worst | O(1) avg | O(n) | None |
| `map` | O(log n) | O(log n) | O(log n) | O(n) | Sorted |
| `unordered_set` | O(1) avg | O(1) avg | O(1) avg | O(n) | None |
| `set` | O(log n) | O(log n) | O(log n) | O(n) | Sorted |
| Hash array | O(1) | O(1) | O(1) | O(MAXVAL) | Index order |

---

## 18. Quick Revision Cheat Sheet

This section is self-contained. Everything needed for same-day interview revision is here.

---

### Why Hashing

O(1) average insert / lookup / delete, vs O(log n) for BST and O(n) for linear scan. The tradeoff: O(n) extra space, and O(n) worst case if all keys collide.

---

### Hash Function

```
h(key) = key % m
```

Choose `m` as a prime not close to a power of 2 for best key distribution.

---

### Collision Resolution

```
Chaining:       linked list at each bucket; O(1) avg, O(n) worst; load factor can exceed 1
Open addressing: probe for next empty slot; O(1) avg, must keep load factor < 1
```

C++ `std::unordered_map` uses chaining.

---

### Load Factor and Rehash

```
α = n / m     (elements / buckets)
Rehash when α > threshold (default 1.0): allocate ~2m buckets, re-insert all keys
Rehash cost: O(n) per event, O(1) amortised per insertion
```

---

### Time Complexity (say both in interviews)

```
Insert / lookup / delete: O(1) average, O(n) worst
When worst case matters:  use map (O(log n) guaranteed)
```

---

### map vs unordered_map

```
map:           Red-Black Tree, O(log n) all ops, keys sorted, adversarial-safe
unordered_map: Hash Table, O(1) avg, keys unordered, ~5-10x faster on integers
```

---

### operator[] trap

```cpp
// INSERT if absent (intentional for freq counting)
freq[x]++;

// CHECK WITHOUT INSERTING
if (freq.count(x))          // preferred
if (freq.find(x) != freq.end())
```

---

### Frequency Counting Templates

```cpp
// Arbitrary keys
unordered_map<int,int> freq;
for (int x : arr) freq[x]++;
int count_x = freq.count(x) ? freq[x] : 0;

// Small non-negative keys (fastest)
int freq[MAXVAL + 1] = {};
for (int x : arr) freq[x]++;
int count_x = freq[x];
```

---

### LC 1838 — Sliding Window Core

```
1. Sort nums.
2. Window [l, r]: all elements can become nums[r] (the max) with <= k ops.
3. cost = nums[r] * (r - l + 1) - windowSum
4. If cost > k: shrink from left (l++, subtract nums[l] from windowSum).
5. ans = max(ans, r - l + 1) after each valid window.
6. Use long long for cost — int overflows.
```

---

### Anti-Hash Attack Defence (Competitive Programming)

```cpp
struct SafeHash {
    size_t operator()(uint64_t x) const {
        static const uint64_t R = chrono::steady_clock::now().time_since_epoch().count();
        x += R + 0x9e3779b97f4a7c15ULL;
        x = (x^(x>>30))*0xbf58476d1ce4e5b9ULL;
        x = (x^(x>>27))*0x94d049bb133111ebULL;
        return x^(x>>31);
    }
};
unordered_map<int, int, SafeHash> mp;
```

Use on Codeforces / AtCoder. Not needed on LeetCode.

---

### When to Reach for What

```
Values in [0, 10^6], counting:          hash array (fastest)
Arbitrary integer keys, max speed:      unordered_map + custom hash if adversarial
Need sorted output:                     map
Presence only (no count):               unordered_set or set
Pair / vector key:                      map or unordered_map + custom hash struct
```

---

### Key Formulas

```
Load factor:              α = n / m
Expected chain length:    O(1 + α)  under uniform hashing
Sliding window cost:      nums[r] * (r - l + 1) - windowSum
Amortised insert cost:    O(1)  via doubling rehash argument
```

---

### Hashing in the Broader DSA Context

```
Two Sum         ->  unordered_map, O(n)
Subarray sum k  ->  prefix sum + unordered_map, O(n)
Anagram group   ->  sorted-string key in map, O(n L log L)
Longest consec. ->  unordered_set, O(n)
Sliding window  ->  unordered_map for frequency in window
DP memoisation  ->  unordered_map<state, answer>
Graph adjacency ->  unordered_set per node (sparse graphs)
```
