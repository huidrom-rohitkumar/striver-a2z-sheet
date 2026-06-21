# Sliding Window & Two Pointer

**Playlist:** [Striver's Sliding Window & Two Pointer Playlist](https://youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL)
- Lecture 3 — Longest Substring Without Repeating Characters → already in `arrays-math-misc.md`
- Lecture 4 — Max Consecutive Ones III (LC 1004)
- Lecture 5 — Fruit Into Baskets (LC 904)
- Lecture 8 — Longest Repeating Character Replacement (LC 424)
- Lecture 9 — Binary Subarrays With Sum (LC 930)
- Lecture 10 — Count Number of Nice Subarrays (LC 1248)

**LeetCode Links:**
- [LC 1004 — Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii)
- [LC 904 — Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets)
- [LC 424 — Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement)
- [LC 930 — Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum)
- [LC 1248 — Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays)

---

## Table of Contents

1. [The Sliding Window Framework](#1-the-sliding-window-framework)
2. [The "Exactly K = AtMost K − AtMost K−1" Identity](#2-the-exactly-k--atmost-k--atmost-k1-identity)
3. [Max Consecutive Ones III (LC 1004)](#3-max-consecutive-ones-iii-lc-1004)
4. [Fruit Into Baskets (LC 904)](#4-fruit-into-baskets-lc-904)
5. [Longest Repeating Character Replacement (LC 424)](#5-longest-repeating-character-replacement-lc-424)
6. [Binary Subarrays With Sum (LC 930)](#6-binary-subarrays-with-sum-lc-930)
7. [Count Number of Nice Subarrays (LC 1248)](#7-count-number-of-nice-subarrays-lc-1248)
8. [The Full Sliding Window Taxonomy](#8-the-full-sliding-window-taxonomy)
9. [Complexity Cheat Sheet](#9-complexity-cheat-sheet)
10. [Edge Cases](#10-edge-cases)
11. [Common Mistakes and Pitfalls](#11-common-mistakes-and-pitfalls)
12. [Interview Reference: Patterns and Extensions](#12-interview-reference-patterns-and-extensions)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. The Sliding Window Framework

A sliding window maintains a **contiguous subarray or substring** defined by two pointers `left` and `right`. The window is always the range `[left, right]`. `right` expands the window (adds elements); `left` shrinks it (removes elements).

**Two types:**

**Fixed-size window:** Window size is always `k`. When `right - left + 1 > k`, advance `left` by 1. Used when the problem specifies a fixed window length.

**Variable-size window:** Window size changes based on a validity condition. Expand `right` freely; when the condition is violated, shrink from `left` until valid again.

**Master template (variable window):**

```cpp
int left = 0;
// window state (frequency map, count, sum, etc.)
for (int right = 0; right < n; right++) {
    // 1. Expand: add s[right] / nums[right] to window state
    update_state(arr[right]);

    // 2. Shrink: while window is invalid, remove from left
    while (WINDOW_INVALID) {
        undo_state(arr[left]);
        left++;
    }

    // 3. Record: window is now valid
    // For "longest": maxLen = max(maxLen, right - left + 1)
    // For "count":  ans += right - left + 1  (all subarrays ending at right)
}
```

**Why `right - left + 1` for counting:** When the window `[left, right]` is valid, every subarray `[left, right]`, `[left+1, right]`, ..., `[right, right]` is also valid (since it is a sub-window of a valid window for at-most problems). There are `right - left + 1` such subarrays.

**Critical distinctions between problems:**

| What changes | Problem type |
|---|---|
| "Maximum length" of a valid window | Expand greedily; shrink only when invalid |
| "Count of subarrays" with exactly k | Use the AtMost identity: `f(k) - f(k-1)` |
| The validity condition | Changes per problem; determines what "invalid" means |

---

## 2. The "Exactly K = AtMost K − AtMost K−1" Identity

This is the single most reusable insight in this problem set. It applies whenever:
1. You need to **count** subarrays satisfying an **exact** constraint.
2. The problem becomes easier if you relax the constraint to **at most** K.
3. The window can be managed cleanly with a sliding window under the at-most constraint.

**Why the sliding window works for "at most" but not "exactly":**

For "at most K" constraints: when the window becomes invalid (constraint exceeded), shrinking from the left always recovers validity. The window can only get "more valid" as elements are removed. This monotonicity is required for the sliding window.

For "exactly K" constraints: removing an element from the left might make the window satisfy K-1, K-2, etc. There is no clean "shrink until valid" step because the valid condition (exactly K) can be both overshot and undershot. The window is not monotone in the valid sense.

**The identity:**
```
count(exactly K) = count(at most K) - count(at most K-1)
```

**Why this works:** Every subarray with sum/count at most K is either:
- Exactly K → counted in `atMost(K)` but not in `atMost(K-1)`, so it survives the subtraction.
- Less than K → counted in both `atMost(K)` and `atMost(K-1)`, so it cancels out.

The difference leaves exactly the subarrays with sum/count = K.

**Problems where this identity applies in this set:**
- Binary Subarrays With Sum: exactly goal → atMost(goal) - atMost(goal-1)
- Count Nice Subarrays: exactly k odd numbers → atMost(k) - atMost(k-1)

---

## 3. Max Consecutive Ones III (LC 1004)

### Problem Statement

Given a binary array `nums` and an integer `k`, return the maximum number of consecutive 1s in the array if you can flip at most `k` zeros.

```
Input:  nums=[1,1,1,0,0,0,1,1,1,1,0], k=2
Output: 6   (flip zeros at index 3 and 10: [1,1,1,0,0,1,1,1,1,1,1])

Input:  nums=[0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], k=3
Output: 10
```

**Constraints:** `1 <= nums.length <= 10^5`, `nums[i]` is 0 or 1, `0 <= k <= nums.length`

### Reformulation (Key Insight)

"Flip at most k zeros" means we are looking for the **longest subarray containing at most k zeros**. Once we find such a subarray, flipping all its zeros gives a consecutive run of 1s of that length. This reframes the problem as a standard variable sliding window.

**Why this reformulation is valid:** Any subarray with at most k zeros can have all its zeros flipped to ones. The resulting run of consecutive 1s has the same length as the subarray. We want the longest such subarray.

### Approach 1 — Brute Force

```cpp
int longestOnesBrute(vector<int>& nums, int k) {
    int n = nums.size(), maxLen = 0;
    for (int i = 0; i < n; i++) {
        int zeros = 0;
        for (int j = i; j < n; j++) {
            if (nums[j] == 0) zeros++;
            if (zeros <= k) maxLen = max(maxLen, j - i + 1);
            else break;
        }
    }
    return maxLen;
}
```

**Time:** O(n^2). **Space:** O(1).

### Approach 2 — Optimal: Sliding Window

```cpp
#include <bits/stdc++.h>
using namespace std;

int longestOnes(vector<int>& nums, int k) {
    int left = 0, zeros = 0, maxLen = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        if (nums[right] == 0) zeros++;          // expand window
        while (zeros > k) {                      // shrink until valid
            if (nums[left] == 0) zeros--;
            left++;
        }
        maxLen = max(maxLen, right - left + 1); // valid window
    }
    return maxLen;
}
```

**Why the `while` loop is correct:** Once `zeros > k`, the window is invalid. We shrink from the left, decrementing `zeros` whenever we remove a 0. After the `while`, the window has at most k zeros again.

**Alternative — window never shrinks (fixed-max-size trick):**

A subtler implementation: instead of shrinking the window when zeros > k, simply slide the entire window one step to the right (advance both `left` and `right`). The window size is the answer.

```cpp
int longestOnesAlt(vector<int>& nums, int k) {
    int left = 0, zeros = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        if (nums[right] == 0) zeros++;
        if (zeros > k) {                  // window is invalid
            if (nums[left] == 0) zeros--;
            left++;                        // slide: advance left by exactly 1
        }
        // right - left + 1 is always the max window size seen so far
    }
    return (int)nums.size() - left;  // final window size = n - left
}
```

**Why this works:** `right - left + 1` is non-decreasing — we never shrink the window size. When the window becomes invalid, we slide it forward by 1. The window size remains the maximum valid size seen so far. At the end, the window size `n - left` equals this maximum.

**This alternative is O(n) but uses a subtlety:** It doesn't actually "maintain" a valid window — the window may be temporarily invalid. However it correctly tracks the maximum valid window size because the window size only changes when we extend it (which we only do when the window was already valid).

**Time:** O(n). **Space:** O(1).

### Dry Run

`nums=[1,1,1,0,0,0,1,1,1,1,0]`, k=2

```
left=0, zeros=0, maxLen=0

right=0: 1. zeros=0. maxLen=1.
right=1: 1. zeros=0. maxLen=2.
right=2: 1. zeros=0. maxLen=3.
right=3: 0. zeros=1. maxLen=4.
right=4: 0. zeros=2. maxLen=5.
right=5: 0. zeros=3 > k=2. Shrink:
  nums[left=0]=1, no zeros--, left=1.
  zeros=3>2 still? After shrinking once, left=1.
  Actually the while loop: zeros=3>2, nums[0]=1, left=1. zeros still 3.
  zeros=3>2, nums[1]=1, left=2. zeros still 3.
  zeros=3>2, nums[2]=1, left=3. zeros still 3.
  zeros=3>2, nums[3]=0, zeros=2, left=4. zeros=2 <= k=2. Stop.
  Window: [4..5], size=2. maxLen=5.
right=6: 1. zeros=2. maxLen=max(5,3)=5.
right=7: 1. zeros=2. maxLen=max(5,4)=5.
right=8: 1. zeros=2. maxLen=max(5,5)=5.
right=9: 1. zeros=2. maxLen=max(5,6)=6.
right=10: 0. zeros=3>2. Shrink:
  nums[4]=0, zeros=2, left=5. Stop.
  Window: [5..10], size=6. maxLen=6.

Return: 6 ✓
```

---

## 4. Fruit Into Baskets (LC 904)

### Problem Statement

You are given an integer array `fruits` where `fruits[i]` is the type of fruit the i-th tree produces. You have two baskets, each holding only one type of fruit. Starting from any tree and picking from consecutive trees (moving right only), find the maximum number of fruits you can pick.

```
Input:  fruits=[1,2,1]       Output: 3
Input:  fruits=[0,1,2,2]     Output: 3   ([1,2,2])
Input:  fruits=[1,2,3,2,2]   Output: 4   ([2,3,2,2])
```

**Constraints:** `1 <= fruits.length <= 10^5`, `0 <= fruits[i] < fruits.length`

### Reformulation

"Two baskets, one fruit type each" = the window can contain **at most 2 distinct values**. Find the **longest subarray with at most 2 distinct elements**.

This is the same problem as "Longest Substring With At Most K Distinct Characters" with k=2. The fruit types are the "characters"; the basket constraint is k=2.

### Approach 1 — Brute Force

```cpp
int totalFruitBrute(vector<int>& fruits) {
    int n = fruits.size(), maxLen = 0;
    for (int i = 0; i < n; i++) {
        unordered_set<int> basket;
        for (int j = i; j < n; j++) {
            basket.insert(fruits[j]);
            if ((int)basket.size() <= 2) maxLen = max(maxLen, j - i + 1);
            else break;
        }
    }
    return maxLen;
}
```

**Time:** O(n^2). **Space:** O(1) (basket has at most 3 elements before break).

### Approach 2 — Optimal: Sliding Window with Frequency Map

```cpp
#include <bits/stdc++.h>
using namespace std;

int totalFruit(vector<int>& fruits) {
    unordered_map<int, int> basket;  // fruit type -> count in window
    int left = 0, maxLen = 0;

    for (int right = 0; right < (int)fruits.size(); right++) {
        basket[fruits[right]]++;             // add fruit to basket

        while ((int)basket.size() > 2) {    // too many distinct types
            basket[fruits[left]]--;
            if (basket[fruits[left]] == 0) basket.erase(fruits[left]);
            left++;
        }
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Why erase when count reaches 0:** The map tracks distinct types currently in the window. If count drops to 0, the type is no longer present; erasing it keeps `basket.size()` accurate.

**Time:** O(n) average — each element is added and removed at most once.
**Space:** O(1) — the map holds at most 3 entries before shrinking (the 3rd entry triggers shrink; shrink removes one; max ever stored simultaneously is 3).

### Generalisation to K Baskets

The only change is `> 2` → `> k`. This solves "Longest Substring with At Most K Distinct Characters":

```cpp
int longestSubstringKDistinct(string s, int k) {
    unordered_map<char, int> freq;
    int left = 0, maxLen = 0;
    for (int right = 0; right < (int)s.size(); right++) {
        freq[s[right]]++;
        while ((int)freq.size() > k) {
            freq[s[left]]--;
            if (freq[s[left]] == 0) freq.erase(s[left]);
            left++;
        }
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

### Dry Run

`fruits=[1,2,3,2,2]`

```
basket={}, left=0, maxLen=0

right=0: basket={1:1}. size=1<=2. maxLen=1.
right=1: basket={1:1,2:1}. size=2<=2. maxLen=2.
right=2: basket={1:1,2:1,3:1}. size=3>2. Shrink:
  fruits[0]=1. basket[1]--=0. Erase 1. basket={2:1,3:1}. left=1. size=2. Stop.
  maxLen=max(2, 2-1+1)=2.
right=3: basket={2:2,3:1}. size=2<=2. maxLen=max(2,3)=3.
right=4: basket={2:3,3:1}. size=2<=2. maxLen=max(3,4)=4.

Return: 4 ✓  (window [1..4] = [2,3,2,2])
```

---

## 5. Longest Repeating Character Replacement (LC 424)

### Problem Statement

Given a string `s` and an integer `k`, you can replace any character in the string with any other character, at most `k` times total. Return the length of the longest substring containing the same letter after performing the replacements.

```
Input:  s="ABAB", k=2    Output: 4   (replace both A→B or both B→A)
Input:  s="AABABBA", k=1 Output: 4   ("AABA" or "ABBA")
```

**Constraints:** `1 <= s.length <= 10^5`, `s` consists of uppercase English letters, `0 <= k <= s.length`

### Core Insight

Within any window `[left, right]` of length `len = right - left + 1`:
- The most frequent character appears `maxFreq` times.
- All other characters (`len - maxFreq` of them) need to be replaced.
- The window is valid if `len - maxFreq <= k`.

We want the longest window where `(window_length - max_frequency_in_window) <= k`.

### Why We Track maxFreq Across All Windows (Not Per Window)

A subtle optimisation: we track the global maximum frequency seen in any window so far (`maxFreq`), not the actual maximum in the current window.

**Why this is correct:** We never need to decrease `maxFreq`. The goal is to find the longest valid window. Once we've seen a window of size `maxFreq + k` (optimal so far), we only care about windows strictly larger than this. A larger window requires `maxFreq` to increase (since `len - maxFreq <= k` rearranges to `maxFreq >= len - k`). So if `maxFreq` doesn't increase, `left` advances to keep `len = maxFreq + k` constant (the window slides, it doesn't shrink).

**Result:** The window size is non-decreasing. When the window becomes invalid, we slide it forward by 1. The final window size is the answer.

```cpp
#include <bits/stdc++.h>
using namespace std;

int characterReplacement(string s, int k) {
    int freq[26] = {};
    int left = 0, maxFreq = 0;

    for (int right = 0; right < (int)s.size(); right++) {
        freq[s[right] - 'A']++;
        maxFreq = max(maxFreq, freq[s[right] - 'A']);

        // Check validity: window length - max_freq <= k
        int windowLen = right - left + 1;
        if (windowLen - maxFreq > k) {
            // Invalid: slide window forward by 1
            freq[s[left] - 'A']--;
            left++;
        }
        // Note: we don't update maxFreq when shrinking.
        // maxFreq may be stale (too high) after shrinking,
        // but that only makes the validity check MORE conservative (never more permissive).
        // The window size never decreases, so a stale maxFreq is safe.
    }
    return (int)s.size() - left;  // final window size
}
```

**Why not re-computing maxFreq when shrinking:** When `left` advances, the character `s[left-1]` is removed. Its frequency drops by 1. `maxFreq` might now be too high (stale). However:
- If `maxFreq` is stale (too high), then `windowLen - maxFreq < windowLen - actual_maxFreq`, so the validity check is stricter than necessary — we might reject a valid window. But we reject it by sliding the window (maintaining window size), not by shrinking it.
- A window that passes the stale check is definitely valid (the condition is sufficient).
- The answer is the maximum window size, and the window size never decreases (each step either expands or slides).

So the algorithm correctly tracks the **maximum window size** even if it doesn't perfectly track the valid window at every step. Re-computing `maxFreq` by scanning the freq array each time would cost O(26) per step — still O(n) total — but is unnecessary given the above argument.

**Time:** O(n). **Space:** O(26) = O(1).

### Dry Run

`s="AABABBA"`, k=1

```
freq={}, left=0, maxFreq=0

right=0 ('A'): freq[A]=1. maxFreq=1. len=1. 1-1=0<=1. Valid. Window=[0..0].
right=1 ('A'): freq[A]=2. maxFreq=2. len=2. 2-2=0<=1. Valid. Window=[0..1].
right=2 ('B'): freq[B]=1. maxFreq=2. len=3. 3-2=1<=1. Valid. Window=[0..2].
right=3 ('A'): freq[A]=3. maxFreq=3. len=4. 4-3=1<=1. Valid. Window=[0..3].
right=4 ('B'): freq[B]=2. maxFreq=3. len=5. 5-3=2>1. INVALID.
  freq[s[0]='A']--. freq[A]=2. left=1. len=4. Don't update maxFreq.
  Window size stays 4. [1..4].
right=5 ('B'): freq[B]=3. maxFreq=max(3,3)=3. len=5. 5-3=2>1. INVALID.
  freq[s[1]='A']--. freq[A]=1. left=2. len=4. Window=[2..5].
right=6 ('A'): freq[A]=2. maxFreq=max(3,2)=3. len=5. 5-3=2>1. INVALID.
  freq[s[2]='B']--. freq[B]=2. left=3. len=4. Window=[3..6].

Return: n - left = 7 - 3 = 4 ✓
```

Longest valid window was size 4 (e.g., "AABA" from index 0-3, or "ABBA" from 3-6).

---

## 6. Binary Subarrays With Sum (LC 930)

### Problem Statement

Given a binary array `nums` and an integer `goal`, return the number of non-empty subarrays with a sum equal to `goal`.

```
Input:  nums=[1,0,1,0,1], goal=2   Output: 4
  Subarrays: [0..1]=[1,0,1], [0..3]=[1,0,1,0,1]... wait:
  [1,0,1] (sum=2), [1,0,1,0] wait sum=2, [0,1,0,1] sum=2, [1,0,1] at end sum=2. 4 total.

Input:  nums=[0,0,0,0,0], goal=0   Output: 15
```

**Constraints:** `1 <= nums.length <= 3 * 10^4`, `nums[i]` is 0 or 1, `0 <= goal <= nums.length`

### Why Direct Sliding Window Fails for "Exactly"

For "at most goal": when `sum > goal`, shrink from left. Monotone — removing elements only decreases or keeps the sum. Works perfectly.

For "exactly goal": when `sum > goal`, shrink. But after shrinking, `sum` might become `< goal` — we've overshot. When `sum == goal`, we can't shrink (it breaks validity). There's no clean "shrink until valid" because there are two ways to be invalid: too high or too low.

**Solution:** AtMost identity.

### Approach 1 — Prefix Sum + Hash Map

```cpp
int numSubarraysWithSumPrefix(vector<int>& nums, int goal) {
    unordered_map<int, int> prefixCount;
    prefixCount[0] = 1;
    int prefSum = 0, count = 0;
    for (int x : nums) {
        prefSum += x;
        count += prefixCount[prefSum - goal];
        prefixCount[prefSum]++;
    }
    return count;
}
```

**Time:** O(n). **Space:** O(n).

### Approach 2 — Optimal: AtMost Sliding Window

Define `atMost(goal)` = number of subarrays with sum ≤ goal.

```
count(exactly goal) = atMost(goal) - atMost(goal - 1)
```

```cpp
int atMost(vector<int>& nums, int goal) {
    if (goal < 0) return 0;  // edge: goal=-1 should return 0
    int left = 0, sum = 0, count = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        sum += nums[right];
        while (sum > goal) {
            sum -= nums[left];
            left++;
        }
        // All subarrays [left..right], [left+1..right], ..., [right..right] are valid
        count += right - left + 1;
    }
    return count;
}

int numSubarraysWithSum(vector<int>& nums, int goal) {
    return atMost(nums, goal) - atMost(nums, goal - 1);
}
```

**Why `goal < 0` guard:** When called as `atMost(nums, goal - 1)` with `goal = 0`, we get `atMost(nums, -1)`. No subarray has sum ≤ -1 in a binary array (all sums ≥ 0), so the answer is 0. The guard returns 0 immediately.

**Why `count += right - left + 1`:** After the while loop, `sum <= goal`, meaning every subarray ending at `right` with start index ≥ `left` has sum ≤ goal. There are `right - left + 1` such subarrays.

**Time:** O(n). **Space:** O(1).

### Dry Run

`nums=[1,0,1,0,1]`, goal=2. We compute `atMost(2) - atMost(1)`.

**atMost(2):**
```
left=0, sum=0, count=0
right=0: sum=1<=2. count+=1. count=1.
right=1: sum=1<=2. count+=2. count=3.
right=2: sum=2<=2. count+=3. count=6.
right=3: sum=2<=2. count+=4. count=10.
right=4: sum=3>2. sum-=nums[0]=1. left=1. sum=2<=2. count+=4. count=14.
atMost(2)=14
```

**atMost(1):**
```
left=0, sum=0, count=0
right=0: sum=1<=1. count+=1. count=1.
right=1: sum=1<=1. count+=2. count=3.
right=2: sum=2>1. left=0: sum-=1,left=1. sum=1<=1. count+=2. count=5.
right=3: sum=1<=1. count+=3. count=8.
right=4: sum=2>1. left=1: sum-=0,left=2. sum=2>1. left=2: sum-=1,left=3. sum=1<=1. count+=2. count=10.
atMost(1)=10
```

`Answer = 14 - 10 = 4 ✓`

---

## 7. Count Number of Nice Subarrays (LC 1248)

### Problem Statement

Given an array of integers `nums` and an integer `k`, return the number of **nice** subarrays — subarrays that contain exactly `k` odd numbers.

```
Input:  nums=[1,1,2,1,1], k=3   Output: 2   ([1,1,2,1],[1,2,1,1])
Input:  nums=[2,4,6], k=1       Output: 0
Input:  nums=[2,2,2,1,2,2,1,2,2,2], k=2  Output: 16
```

**Constraints:** `1 <= nums.length <= 50000`, `1 <= k <= nums.length`, `1 <= nums[i] <= 10^5`

### Key Transformation

Replace each element with `nums[i] % 2` (1 if odd, 0 if even). Now "exactly k odd numbers in a subarray" becomes "exactly k ones in the transformed binary array" which is identical to Binary Subarrays With Sum (LC 930) with `goal = k`.

This transformation is the bridge: once you recognise the problem as LC 930 in disguise, the solution follows directly.

### Approach: AtMost Identity on Oddness

```cpp
#include <bits/stdc++.h>
using namespace std;

int atMostOdd(vector<int>& nums, int k) {
    if (k < 0) return 0;
    int left = 0, odds = 0, count = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        odds += nums[right] % 2;        // 1 if odd, 0 if even
        while (odds > k) {
            odds -= nums[left] % 2;
            left++;
        }
        count += right - left + 1;
    }
    return count;
}

int numberOfSubarrays(vector<int>& nums, int k) {
    return atMostOdd(nums, k) - atMostOdd(nums, k - 1);
}
```

**Time:** O(n). **Space:** O(1).

### Alternative: Prefix Sum with Count Array

Convert `nums[i]` to `nums[i] % 2` first, then apply the prefix sum approach from LC 930:

```cpp
int numberOfSubarraysPrefix(vector<int>& nums, int k) {
    unordered_map<int, int> prefCount;
    prefCount[0] = 1;
    int prefOdds = 0, count = 0;
    for (int x : nums) {
        prefOdds += x % 2;
        count += prefCount[prefOdds - k];
        prefCount[prefOdds]++;
    }
    return count;
}
```

**Time:** O(n). **Space:** O(n).

### Dry Run

`nums=[1,1,2,1,1]`, k=3.

Oddness array: `[1,1,0,1,1]`.

**atMostOdd(3):**
```
right=0: odds=1. count+=1=1.
right=1: odds=2. count+=2=3.
right=2: odds=2. count+=3=6.
right=3: odds=3. count+=4=10.
right=4: odds=4>3. left=0: odds-=1=3, left=1. count+=4=14.
atMostOdd(3)=14
```

**atMostOdd(2):**
```
right=0: odds=1. count+=1=1.
right=1: odds=2. count+=2=3.
right=2: odds=2. count+=3=6.
right=3: odds=3>2. left=0: odds-=1=2, left=1. count+=3=9.
right=4: odds=3>2. left=1: odds-=1=2, left=2. count+=3=12.
atMostOdd(2)=12
```

`Answer = 14 - 12 = 2 ✓`

---

## 8. The Full Sliding Window Taxonomy

All five problems (plus Longest Substring Without Repeating Characters from the previous file) fit into this taxonomy:

| Problem | Window condition | What to record | Technique |
|---|---|---|---|
| Longest Substring No Repeat | All chars unique | max window length | Expand; jump left on duplicate |
| Max Consecutive Ones III | zeros ≤ k | max window length | Expand; slide when zeros > k |
| Fruit Into Baskets | distinct types ≤ 2 | max window length | Expand; shrink when types > 2 |
| Longest Repeating Char Replace | len - maxFreq ≤ k | max window length | Slide; track global maxFreq |
| Binary Subarrays With Sum | sum == goal (exact) | count of subarrays | atMost(goal) - atMost(goal-1) |
| Count Nice Subarrays | odds == k (exact) | count of subarrays | atMost(k) - atMost(k-1) on oddness |

**Three window behaviours:**

1. **Shrink to restore validity** (Longest Substring, Fruit Into Baskets, Binary Subarrays): `while (INVALID) { remove left; left++; }` — window size can decrease.

2. **Slide to preserve max size** (Max Consecutive Ones III alt, Longest Repeating): `if (INVALID) { remove left; left++; }` — window size never decreases.

3. **AtMost subtraction for exact counting** (Binary Subarrays With Sum, Count Nice Subarrays): count at-most-K, then subtract at-most-(K-1).

---

## 9. Complexity Cheat Sheet

| Problem | Brute | Optimal | Optimal Space |
|---|---|---|---|
| Max Consecutive Ones III | O(n^2) | O(n) | O(1) |
| Fruit Into Baskets | O(n^2) | O(n) avg | O(1) (map ≤ 3 entries) |
| Longest Repeating Char Replace | O(n^2) | O(n) | O(26)=O(1) |
| Binary Subarrays With Sum (prefix) | O(n^2) | O(n) | O(n) |
| Binary Subarrays With Sum (atmost) | O(n^2) | O(n) | O(1) |
| Count Nice Subarrays | O(n^2) | O(n) | O(1) |

---

## 10. Edge Cases

| Problem | Input | Output | Reason |
|---|---|---|---|
| Max Cons. Ones III | k=0 | longest run of 1s | No flips allowed; standard Max Consecutive Ones I |
| Max Cons. Ones III | all zeros, k=n | n | Flip all zeros; entire array becomes 1s |
| Max Cons. Ones III | all ones | n | zeros=0<=k always; window spans full array |
| Fruit Into Baskets | single fruit type | n | Always 1 distinct; always valid |
| Fruit Into Baskets | [1,2,1,3,3] | 4 ([2,1,3,3] or [1,3,3]... | Window adjusts when 3rd type appears |
| Longest Repeating | k=0 | longest run of same char | No replacements; find max freq directly |
| Longest Repeating | all same char | n | All chars same; maxFreq=n; 0 replacements needed |
| Longest Repeating | k >= n | n | Can replace everything; whole string valid |
| Binary Subarrays | goal=0, all zeros | n*(n+1)/2 | Every subarray has sum 0 |
| Binary Subarrays | goal=0, has ones | count of all-zero subarrays | atMost(0)-atMost(-1)=atMost(0)-0 |
| Count Nice Subarrays | all even | 0 | No odd numbers; no nice subarrays for k>=1 |
| Count Nice Subarrays | k=1, [1] | 1 | Single odd element = one nice subarray |

---

## 11. Common Mistakes and Pitfalls

### Mistake 1: Forgetting the `goal < 0` guard in atMost

```cpp
// WRONG: when called as atMost(nums, goal-1) with goal=0, passes -1
// The while loop condition `sum > -1` is always true for binary array (sum >= 0)
// causing left to advance past right and infinite loop or wrong count.
int atMost(vector<int>& nums, int goal) {
    // Missing guard!
    int left = 0, sum = 0, count = 0;
    for (int right = ...) {
        sum += nums[right];
        while (sum > goal) { sum -= nums[left++]; }  // crashes when goal=-1
        count += right - left + 1;
    }
}

// CORRECT:
int atMost(vector<int>& nums, int goal) {
    if (goal < 0) return 0;  // critical guard
    ...
}
```

### Mistake 2: Using `if` instead of `while` in the standard shrink approach

```cpp
// WRONG: one shrink step may not be enough to restore validity
if (zeros > k) {
    if (nums[left] == 0) zeros--;
    left++;
}
// Problem: after one shrink, window may still have zeros > k

// CORRECT:
while (zeros > k) {
    if (nums[left] == 0) zeros--;
    left++;
}
```

Exception: in the "non-shrinking" variant of Max Consecutive Ones III, `if` is intentional — the window only ever slides by 1.

### Mistake 3: Erasing from map with `erase` without checking count first

```cpp
// WRONG: erases the entry even if count > 0
basket[fruits[left]]--;
basket.erase(fruits[left]);  // might erase a still-present fruit

// CORRECT: only erase when count reaches 0
basket[fruits[left]]--;
if (basket[fruits[left]] == 0) basket.erase(fruits[left]);
left++;
```

### Mistake 4: Re-computing maxFreq by scanning freq[] after shrinking (unnecessary but also wrong reasoning)

```cpp
// THINKING: "I should recompute maxFreq when window shrinks"
// WRONG reasoning: The stale maxFreq is INTENTIONAL in LC 424.
// Recomputing makes the algorithm correct but adds O(26) per step — still O(n).
// It does NOT change correctness either way, but understanding WHY stale maxFreq
// is safe is what interviewers probe.
```

### Mistake 5: Counting subarrays with `right - left` instead of `right - left + 1`

```cpp
// WRONG: off-by-one; misses the subarray [right..right]
count += right - left;

// CORRECT:
count += right - left + 1;
// Subarrays: [left..right], [left+1..right], ..., [right..right] = (right-left+1) subarrays
```

### Mistake 6: Not recognising Fruit Into Baskets as "at most 2 distinct"

This is the most common conceptual miss. The problem describes baskets and fruits in domain language. Strip the story: two baskets = at most 2 distinct values. Longest contiguous = maximum window. Problem reduces to "Longest Subarray with At Most 2 Distinct Elements."

---

## 12. Interview Reference: Patterns and Extensions

### Trigger Words for Sliding Window

| Words in problem | Pattern |
|---|---|
| "longest subarray/substring with [condition]" | Variable window: expand right, shrink when condition violated |
| "maximum [something] with at most k [constraint]" | Variable window with at-most constraint |
| "count subarrays with exactly k [something]" | atMost(k) - atMost(k-1) |
| "at most 2/k distinct characters/elements" | Sliding window + frequency map |
| "flip at most k zeros/characters" | Rephrase as "at most k of type X in window" |

### Extensions of Each Problem

**Max Consecutive Ones III (LC 1004):**
- LC 485 (Max Consecutive Ones I): k=0 — already in `array-fundamentals.md`.
- LC 487 (Max Consecutive Ones II): k=1.
- LC 2024 (Maximize the Confusion of an Exam): run the same algorithm twice, once treating 'T' as the character to replace and once 'F'.

**Fruit Into Baskets (LC 904):**
- LC 340 (Longest Substring with At Most K Distinct): generalise basket constraint from 2 to k.
- LC 159 (At Most 2 Distinct): identical to Fruit Into Baskets on a string.
- LC 992 (Subarrays with K Different Integers): exactly K distinct = atMost(K) - atMost(K-1). The same identity but for distinct element count.

**Longest Repeating Character Replacement (LC 424):**
- LC 2024 (Maximize Exam Confusion): same structure; two characters ('T' and 'F') instead of 26. Run twice.
- LC 1004 (Max Consecutive Ones III): a special case where only one character ('0') needs replacement, so maxFreq is always the count of 1s.

**Binary Subarrays With Sum (LC 930) and Count Nice Subarrays (LC 1248):**
- Both use atMost identity. The connection: Nice Subarrays reduces to Binary Subarrays by replacing each element with its oddness.
- LC 560 (Subarray Sum Equals K): general (non-binary) array. Sliding window fails for negative numbers; prefix sum + hash map is the approach. Already covered in `arrays-part3.md`.

### The Connection Graph Between Problems

```
Max Consecutive Ones I (k=0)
    ↓ allow k flips
Max Consecutive Ones III (LC 1004)
    ↓ generalise "character to flip" to any character
Longest Repeating Character Replacement (LC 424)

Fruit Into Baskets (k=2 distinct)
    ↓ generalise to k distinct
Longest Substring with At Most K Distinct (LC 340)
    ↓ count instead of length + exactly k
Subarrays with K Different Integers (LC 992) [atMost identity]

Binary Subarrays With Sum (LC 930) [binary array, sum=goal]
    ↓ replace element with oddness (num%2)
Count Number of Nice Subarrays (LC 1248) [odd count=k]
    ↓ replace element with prefix XOR trick
Subarray XOR = K [from arrays-part3.md]
```

---

## 13. Quick Revision Cheat Sheet

**Sliding Window Master Template**
```cpp
int left = 0; // window state variables
for (int right = 0; right < n; right++) {
    add(arr[right]);            // expand
    while (INVALID) {           // shrink OR:
        remove(arr[left++]);    //   slide: if(INVALID){remove(left); left++;}
    }
    ans = max(ans, right-left+1);  // or: ans += right-left+1 (count)
}
```

**Max Consecutive Ones III (LC 1004)**
- Rephrase: longest subarray with at most k zeros
- Track `zeros` count; shrink when `zeros > k`
- O(n) time, O(1) space

**Fruit Into Baskets (LC 904)**
- Rephrase: longest subarray with at most 2 distinct values
- Track frequency map; erase when count→0; shrink when `map.size() > 2`
- O(n) avg time, O(1) space (map ≤ 3 entries)

**Longest Repeating Character Replacement (LC 424)**
- Valid if `windowLen - maxFreq <= k`
- Track `maxFreq` globally (never decrease it)
- On invalid: slide by 1 (`if`, not `while`); window size never decreases
- O(n) time, O(26)=O(1) space

**The AtMost Identity (Binary Subarrays / Count Nice Subarrays)**
```
count(exactly K) = atMost(K) - atMost(K-1)
```
```cpp
int atMost(vector<int>& nums, int k) {
    if (k < 0) return 0;   // critical guard
    int left = 0, sum = 0, count = 0;
    for (int right = 0; right < n; right++) {
        sum += nums[right] [% 2 for nice subarrays];
        while (sum > k) { sum -= nums[left++] [% 2]; }
        count += right - left + 1;
    }
    return count;
}
```

**Count Nice Subarrays (LC 1248)**
- Replace each element with `nums[i] % 2` (oddness)
- Apply atMost identity with this transformed array
- Identical to Binary Subarrays With Sum with `goal = k`

**Complexity at a Glance**

| Problem | Time | Space |
|---|---|---|
| Max Consecutive Ones III | O(n) | O(1) |
| Fruit Into Baskets | O(n) avg | O(1) |
| Longest Repeating Char Replace | O(n) | O(1) |
| Binary Subarrays With Sum | O(n) | O(1) atMost / O(n) prefix |
| Count Nice Subarrays | O(n) | O(1) |

**Key distinctions that trip people up:**
- `while` (shrink fully) vs `if` (slide by 1): LC 424 uses `if`; most others use `while`.
- `count += right - left + 1` (not `right - left`).
- `goal < 0` guard in atMost function.
- Erase from map only when `count == 0` (not immediately after decrement).
