# Sliding Window and Two Pointers 

**Problems covered:**
- [LC 1358 — Number of Substrings Containing All Three Characters](https://leetcode.com/problems/number-of-substrings-containing-all-three-characters/)
- [LC 1423 — Maximum Points You Can Obtain from Cards](https://leetcode.com/problems/maximum-points-you-can-obtain-from-cards/)
- [LC 340 — Longest Substring with At Most K Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/)
- [LC 992 — Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/)
- [LC 76 — Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/)
- [LC 727 — Minimum Window Subsequence](https://leetcode.com/problems/minimum-window-subsequence/)

**Striver Playlist:** [A2Z DSA Sliding Window / Two Pointers](https://youtube.com/playlist?list=PLgUwDviBIf0q7vrFA_HEWcqRqMpCXzYAL)

---

## Table of Contents

1. [The Unifying Mental Model](#1-the-unifying-mental-model)
2. [Problem 1 — Number of Substrings Containing All Three Characters (LC 1358)](#2-problem-1--number-of-substrings-containing-all-three-characters-lc-1358)
3. [Problem 2 — Maximum Points You Can Obtain from Cards (LC 1423)](#3-problem-2--maximum-points-you-can-obtain-from-cards-lc-1423)
4. [Problem 3 — Longest Substring with At Most K Distinct Characters (LC 340)](#4-problem-3--longest-substring-with-at-most-k-distinct-characters-lc-340)
5. [Problem 4 — Subarrays with K Different Integers (LC 992)](#5-problem-4--subarrays-with-k-different-integers-lc-992)
6. [Problem 5 — Minimum Window Substring (LC 76)](#6-problem-5--minimum-window-substring-lc-76)
7. [Problem 6 — Minimum Window Subsequence (LC 727)](#7-problem-6--minimum-window-subsequence-lc-727)
8. [Substring vs Subsequence — Why the Sliding Window Breaks](#8-substring-vs-subsequence--why-the-sliding-window-breaks)
9. [The "Exactly K" via "At Most K" Reduction](#9-the-exactly-k-via-at-most-k-reduction)
10. [Common Mistakes and Pitfalls](#10-common-mistakes-and-pitfalls)
11. [Interview Reference — Pattern Map](#11-interview-reference--pattern-map)
12. [Complexity Cheat Sheet](#12-complexity-cheat-sheet)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Model

This set of six problems represents the hard tier of sliding window and two-pointer technique. They split into three families:

**Family 1 — Count all valid windows ending at each position (Problem 1):** Instead of finding one optimal window, sum contributions across all positions. The key trick: once a window from `[left, right]` is valid, every extension `[left, right+1], [left, right+2], ..., [left, n-1]` is also valid (extending only adds more characters, never removes the ones that made it valid).

**Family 2 — Transform the problem to make sliding window applicable (Problem 2):** "Take k cards from either end" doesn't look like a sliding window problem at first. The transformation — picking k cards from the ends is equivalent to *excluding* a contiguous window of size `n-k` from the middle — turns it into a minimum-subarray-sum sliding window.

**Family 3 — Variable-size window with a counting/matching constraint (Problems 3–6):** The window grows by moving `right`, and shrinks by moving `left` only when a constraint is violated (too many distinct characters) or can be tightened (window already valid, try to shrink further). The hard part is precisely defining "valid" and efficiently checking it in O(1) amortized per step.

**The "exactly K" trick (Problem 4):** Counting subarrays with **exactly** K distinct elements is hard to do directly with sliding window, because adding an element can either increase or not increase the distinct count, and there's no clean monotonic shrink condition. The trick: `exactly(K) = atMost(K) - atMost(K-1)`, where `atMost` is a clean, monotonic sliding window problem.

**Substring vs subsequence (Problems 5 vs 6):** Minimum Window Substring requires the target's characters to appear in the window with sufficient *frequency*, in any order — classic sliding window. Minimum Window Subsequence requires the target's characters to appear in the window in the *same relative order* — this breaks the standard two-pointer shrink logic and requires either DP or a specialized two-pointer "expand then contract-and-rematch" technique.

---

## 2. Problem 1 — Number of Substrings Containing All Three Characters (LC 1358)

### Problem Statement

Given a string `s` consisting only of characters `'a'`, `'b'`, and `'c'`, return the number of substrings that contain **at least one occurrence** of all three characters.

**Constraints:** `3 <= s.length <= 5*10^4`, `s` consists only of `a`, `b`, `c`.

**Examples:**
- `s = "abcabc"` → `10`
- `s = "aaacb"` → `3`
- `s = "abc"` → `1`

### Approach 1 — Brute Force: All Substrings (O(n²) or O(n³))

```cpp
int numberOfSubstrings_brute(string s) {
    int n = s.size(), count = 0;
    for (int i = 0; i < n; i++) {
        set<char> seen;
        for (int j = i; j < n; j++) {
            seen.insert(s[j]);
            if (seen.size() == 3) {
                count += (n - j); // every extension from j to n-1 is also valid
                break; // no need to keep extending this starting point
            }
        }
    }
    return count;
}
```

**Why `count += (n - j)` and break?** Once `s[i..j]` contains all three characters, extending the right end (`s[i..j+1]`, `s[i..j+2]`, ..., `s[i..n-1]`) keeps it valid — adding characters never removes existing ones. So all `n - j` extensions starting at `i` are valid in one shot.

**Time:** O(n²) worst case (still scans for each `i`). **Space:** O(1).

### Approach 2 — Sliding Window Shrink-and-Count (O(n))

**Intuition:** Maintain a window `[left, right]` using a frequency count of a, b, c. Expand `right`. Once the window contains all three characters, every further extension to the right also contains all three. So instead of extending, **shrink from the left** while still valid, and for each valid window, add `n - right` (the number of valid endings from `right` to `n-1`).

```cpp
#include <bits/stdc++.h>
using namespace std;

int numberOfSubstrings(string s) {
    int count[3] = {0, 0, 0};
    int left = 0, n = s.size();
    long long result = 0;

    for (int right = 0; right < n; right++) {
        count[s[right] - 'a']++;

        while (count[0] > 0 && count[1] > 0 && count[2] > 0) {
            result += (n - right); // all substrings starting at 'left', ending from 'right' to 'n-1'
            count[s[left] - 'a']--;
            left++;
        }
    }
    return (int)result;
}
```

**Dry Run:** `s = "abcabc"`, n=6

```
left=0, count=[0,0,0], result=0

right=0 'a': count=[1,0,0]. Not all >0.
right=1 'b': count=[1,1,0]. Not all >0.
right=2 'c': count=[1,1,1]. All >0!
  result += 6-2=4. count[a]--→[0,1,1]. left=1.
  Now count[0]=0, exit while.
right=3 'a': count=[1,1,1]. All >0!
  result += 6-3=3. count[a]--→[0,1,1]. left=2.
  count[0]=0, exit while.
right=4 'b': count=[0,2,1]. count[0]=0, not valid.
right=5 'c': count=[0,2,2]. count[0]=0, not valid.

Total result = 4+3 = 7?
```

Wait, let me recheck — expected output is 10. Let me retrace more carefully — when right=3 we add 'a' which is s[3]='a', count becomes [1,1,1] (a:1,b:1,c:1) again since we removed one 'a' at left=0 step. Let's redo fully.

```
s = "a b c a b c" (indices 0-5)
left=0, count=[a:0,b:0,c:0], result=0

right=0 (a): count=[1,0,0]. Not valid (b,c=0).
right=1 (b): count=[1,1,0]. Not valid (c=0).
right=2 (c): count=[1,1,1]. Valid!
  result += (6-2)=4 → result=4
  remove s[left=0]='a': count=[0,1,1]. left=1.
  count[a]=0 → exit while loop.
right=3 (a): count=[1,1,1]. Valid!
  result += (6-3)=3 → result=7
  remove s[left=1]='b': count=[1,0,1]. left=2.
  count[b]=0 → exit while loop.
right=4 (b): count=[1,1,1]. Valid!
  result += (6-4)=2 → result=9
  remove s[left=2]='c': count=[1,1,0]. left=3.
  count[c]=0 → exit while loop.
right=5 (c): count=[1,1,1]. Valid!
  result += (6-5)=1 → result=10
  remove s[left=3]='a': count=[0,1,1]. left=4.
  count[a]=0 → exit while loop.

Total result = 10 ✓
```

**Time:** O(n) — each character enters and leaves the window at most once.
**Space:** O(1) — fixed-size count array.

### Approach 3 — Last-Seen-Index O(n), O(1) (Most Elegant)

**Intuition:** At each position `i`, if all three characters have appeared at least once by index `i`, then the number of valid substrings *ending* at `i` equals `min(lastSeen[a], lastSeen[b], lastSeen[c]) + 1` — because any starting point from `0` to that minimum index produces a substring `s[start..i]` containing all three.

```cpp
int numberOfSubstrings_lastSeen(string s) {
    int last[3] = {-1, -1, -1};
    long long count = 0;

    for (int i = 0; i < (int)s.size(); i++) {
        last[s[i] - 'a'] = i;
        if (last[0] >= 0 && last[1] >= 0 && last[2] >= 0) {
            count += min({last[0], last[1], last[2]}) + 1;
        }
    }
    return (int)count;
}
```

**Why `+1`?** Valid starting positions for a substring ending at `i` range from `0` to `min(last[0],last[1],last[2])` inclusive — that's `min(...) + 1` choices.

**Time:** O(n). **Space:** O(1).

---

## 3. Problem 2 — Maximum Points You Can Obtain from Cards (LC 1423)

### Problem Statement

`cardPoints[i]` = points of the `i`-th card in a row. In one step, take one card from the beginning or end. You must take exactly `k` cards. Return the maximum score (sum of points taken).

**Constraints:** `1 <= cardPoints.length <= 10^5`, `1 <= cardPoints[i] <= 10^4`, `1 <= k <= cardPoints.length`.

**Examples:**
- `cardPoints = [1,2,3,4,5,6,1]`, `k=3` → `12` (take 1 from left, 2,1 — wait, take rightmost 3: 1+6+5=12)
- `cardPoints = [2,2,2]`, `k=2` → `4`

### The Key Transformation

Taking `k` cards from the two ends is equivalent to **excluding** a contiguous subarray of length `n - k` from the middle. Maximizing the sum of taken cards = `totalSum - minimizing the sum of the excluded window`.

This converts an "ends-picking" problem into a classic **minimum sliding window sum** problem.

### Approach 1 — Brute Force: Try All Splits (O(k))

For `i` cards from the left (`i` from `0` to `k`), take `k-i` from the right. Compute the sum for each split using prefix/suffix sums.

```cpp
int maxScore_bruteSplit(vector<int>& cardPoints, int k) {
    int n = cardPoints.size();
    vector<int> prefix(n + 1, 0), suffix(n + 1, 0);
    for (int i = 0; i < n; i++) prefix[i+1] = prefix[i] + cardPoints[i];
    for (int i = n-1; i >= 0; i--) suffix[i] = suffix[i+1] + cardPoints[i];

    int result = 0;
    for (int left = 0; left <= k; left++) {
        int right = k - left;
        result = max(result, prefix[left] + suffix[n - right]);
    }
    return result;
}
```

**Time:** O(n) for prefix/suffix arrays + O(k) for the split loop = O(n). **Space:** O(n).

### Approach 2 — Sliding Window on the Excluded Middle (Optimal, O(n), O(1))

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxScore(vector<int>& cardPoints, int k) {
    int n = cardPoints.size();
    int windowSize = n - k;

    if (windowSize == 0) {
        return accumulate(cardPoints.begin(), cardPoints.end(), 0);
    }

    int totalSum = accumulate(cardPoints.begin(), cardPoints.end(), 0);

    // Find minimum sum window of size 'windowSize'
    int windowSum = 0;
    for (int i = 0; i < windowSize; i++) windowSum += cardPoints[i];

    int minWindowSum = windowSum;
    for (int i = windowSize; i < n; i++) {
        windowSum += cardPoints[i] - cardPoints[i - windowSize];
        minWindowSum = min(minWindowSum, windowSum);
    }

    return totalSum - minWindowSum;
}
```

**Dry Run:** `cardPoints = [1,2,3,4,5,6,1]`, `k=3`, n=7

```
windowSize = 7-3 = 4
totalSum = 1+2+3+4+5+6+1 = 22

Initial window [0..3]: 1+2+3+4=10. minWindowSum=10.

i=4: windowSum += cardPoints[4]-cardPoints[0] = 5-1=4 → windowSum=14. min stays 10.
i=5: windowSum += cardPoints[5]-cardPoints[1] = 6-2=4 → windowSum=18. min stays 10.
i=6: windowSum += cardPoints[6]-cardPoints[2] = 1-3=-2 → windowSum=16. min stays 10.

minWindowSum = 10
Answer = 22 - 10 = 12 ✓
```

**Time:** O(n). **Space:** O(1).

### Approach 3 — Direct Sliding Window on the Picked Cards (Equivalent, Sometimes More Intuitive)

```cpp
int maxScore_direct(vector<int>& cardPoints, int k) {
    int n = cardPoints.size();
    int sum = 0;
    for (int i = 0; i < k; i++) sum += cardPoints[i]; // all k from left initially

    int maxSum = sum;
    int rightIdx = n - 1;
    for (int i = k - 1; i >= 0; i--) {
        sum -= cardPoints[i];               // give back one from left
        sum += cardPoints[rightIdx--];      // take one from right
        maxSum = max(maxSum, sum);
    }
    return maxSum;
}
```

**Time:** O(k). **Space:** O(1).

---

## 4. Problem 3 — Longest Substring with At Most K Distinct Characters (LC 340)

### Problem Statement

Given a string `s` and integer `k`, return the length of the longest substring that contains **at most** `k` distinct characters.

**Constraints:** `1 <= s.length <= 5*10^4`, `0 <= k <= 50`.

**Examples:**
- `s = "eceba"`, `k=2` → `3` ("ece")
- `s = "aa"`, `k=1` → `2`

### Approach — Sliding Window with Frequency Map

**Invariant:** The window `[left, right]` always has at most `k` distinct characters. Expand `right`; if distinct count exceeds `k`, shrink `left` until valid again.

```cpp
#include <bits/stdc++.h>
using namespace std;

int lengthOfLongestSubstringKDistinct(string s, int k) {
    if (k == 0) return 0;
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

**Dry Run:** `s = "eceba"`, `k=2`

```
left=0, freq={}, maxLen=0

right=0 'e': freq={e:1}. size=1≤2. maxLen=1
right=1 'c': freq={e:1,c:1}. size=2≤2. maxLen=2
right=2 'e': freq={e:2,c:1}. size=2≤2. maxLen=3
right=3 'b': freq={e:2,c:1,b:1}. size=3>2.
  Shrink: remove s[0]='e'. freq={e:1,c:1,b:1}. left=1. size still 3>2.
  Shrink: remove s[1]='c'. freq={e:1,b:1}. left=2. size=2≤2. exit while.
  maxLen = max(3, 3-2+1=2) = 3
right=4 'a': freq={e:1,b:1,a:1}. size=3>2.
  Shrink: remove s[2]='e'. freq={b:1,a:1}. left=3. size=2≤2.
  maxLen = max(3, 4-3+1=2) = 3

Answer: 3 ✓ ("ece")
```

**Time:** O(n) — each character enters/leaves the window once.
**Space:** O(k) — the frequency map holds at most `k+1` entries at any point.

**Why `erase` when count reaches 0?** Keeps `freq.size()` an accurate count of distinct characters in the window — essential for the `> k` check to be correct.

---

## 5. Problem 4 — Subarrays with K Different Integers (LC 992)

### Problem Statement

Given integer array `nums` and integer `k`, return the number of subarrays with **exactly** `k` different integers.

**Constraints:** `1 <= nums.length <= 2*10^4`, `1 <= nums[i], k <= nums.length`.

**Examples:**
- `nums = [1,2,1,2,3]`, `k=2` → `7`
- `nums = [1,2,1,3,4]`, `k=3` → `3`

### Why Direct Sliding Window Fails for "Exactly K"

Unlike "at most K," there's no clean monotonic shrink rule for "exactly K." Adding a new element can either keep distinct count the same or increase it; there's no way to define a simple left-pointer movement that maintains "exactly K distinct" as an invariant while also capturing all valid subarrays. The window for "exactly K" isn't a single contiguous range of valid starting points for each `right` — it's harder to characterize directly.

### The Reduction: exactly(K) = atMost(K) - atMost(K-1)

**Why this works:** `atMost(K)` counts all subarrays with ≤ K distinct integers. `atMost(K-1)` counts all subarrays with ≤ K-1 distinct integers. The difference removes everything with strictly fewer than K distinct integers, leaving exactly those with precisely K distinct integers.

```cpp
#include <bits/stdc++.h>
using namespace std;

int atMostKDistinct(vector<int>& nums, int k) {
    if (k < 0) return 0; // guard for k=0 call with K-1 when K=0 (not needed here since k>=1 in constraints)
    unordered_map<int, int> freq;
    int left = 0, count = 0;

    for (int right = 0; right < (int)nums.size(); right++) {
        freq[nums[right]]++;

        while ((int)freq.size() > k) {
            freq[nums[left]]--;
            if (freq[nums[left]] == 0) freq.erase(nums[left]);
            left++;
        }

        count += (right - left + 1); // all subarrays ending at 'right' starting from 'left' to 'right'
    }
    return count;
}

int subarraysWithKDistinct(vector<int>& nums, int k) {
    return atMostKDistinct(nums, k) - atMostKDistinct(nums, k - 1);
}
```

**Why `count += (right - left + 1)`?** For a fixed `right`, every subarray `[start, right]` for `start` from `left` to `right` has at most `k` distinct integers (since the window `[left, right]` already satisfies this, and any sub-window `[start, right]` with `start >= left` has no more distinct elements than `[left, right]`). There are `right - left + 1` such subarrays.

**Dry Run:** `nums = [1,2,1,2,3]`, `k=2`

```
atMostKDistinct(nums, 2):
left=0, freq={}, count=0

right=0 (1): freq={1:1}. size=1≤2. count+=0-0+1=1 → count=1
right=1 (2): freq={1:1,2:1}. size=2≤2. count+=1-0+1=2 → count=3
right=2 (1): freq={1:2,2:1}. size=2≤2. count+=2-0+1=3 → count=6
right=3 (2): freq={1:2,2:2}. size=2≤2. count+=3-0+1=4 → count=10
right=4 (3): freq={1:2,2:2,3:1}. size=3>2.
  Shrink: remove nums[0]=1. freq={1:1,2:2,3:1}. left=1. size=3>2.
  Shrink: remove nums[1]=2. freq={1:1,2:1,3:1}. left=2. size=3>2.
  Shrink: remove nums[2]=1. freq={2:1,3:1}. left=3. size=2≤2. exit while.
  count += 4-3+1=2 → count=12

atMostKDistinct(nums, 2) = 12

atMostKDistinct(nums, 1):
left=0, freq={}, count=0

right=0 (1): freq={1:1}. size=1≤1. count+=1 → count=1
right=1 (2): freq={1:1,2:1}. size=2>1.
  Shrink: remove nums[0]=1. freq={2:1}. left=1. size=1≤1.
  count += 1-1+1=1 → count=2
right=2 (1): freq={2:1,1:1}. size=2>1.
  Shrink: remove nums[1]=2. freq={1:1}. left=2. size=1≤1.
  count += 2-2+1=1 → count=3
right=3 (2): freq={1:1,2:1}. size=2>1.
  Shrink: remove nums[2]=1. freq={2:1}. left=3. size=1≤1.
  count += 3-3+1=1 → count=4
right=4 (3): freq={2:1,3:1}. size=2>1.
  Shrink: remove nums[3]=2. freq={3:1}. left=4. size=1≤1.
  count += 4-4+1=1 → count=5

atMostKDistinct(nums, 1) = 5

Answer = 12 - 5 = 7 ✓
```

**Time:** O(n) per call, O(n) total (two calls). **Space:** O(k) for the frequency map.

---

## 6. Problem 5 — Minimum Window Substring (LC 76)

### Problem Statement

Given strings `s` and `t`, return the minimum window substring of `s` such that every character in `t` (including duplicates) is included in the window. If no such substring exists, return `""`.

**Constraints:** `1 <= s.length, t.length <= 10^5`, consists of uppercase and lowercase English letters.

**Examples:**
- `s = "ADOBECODEBANC"`, `t = "ABC"` → `"BANC"`
- `s = "a"`, `t = "a"` → `"a"`
- `s = "a"`, `t = "aa"` → `""` (t needs two a's, s only has one)

### Approach — Sliding Window with "Formed" Counter

**Key data structures:**
- `need`: frequency map of characters required by `t`.
- `window`: frequency map of characters currently in the window.
- `formed`: count of distinct characters in `t` whose required frequency has been met in the current window.
- `required`: total distinct characters in `t` (target value for `formed`).

**The window is valid exactly when `formed == required`.**

```cpp
#include <bits/stdc++.h>
using namespace std;

string minWindow(string s, string t) {
    if (s.empty() || t.empty()) return "";

    unordered_map<char, int> need;
    for (char c : t) need[c]++;

    int required = need.size();
    unordered_map<char, int> window;
    int formed = 0;
    int left = 0;
    int bestLen = INT_MAX, bestStart = 0;

    for (int right = 0; right < (int)s.size(); right++) {
        char c = s[right];
        window[c]++;

        // This character instance just satisfied the requirement for c
        if (need.count(c) && window[c] == need[c]) {
            formed++;
        }

        // Try to shrink the window as much as possible while valid
        while (left <= right && formed == required) {
            if (right - left + 1 < bestLen) {
                bestLen = right - left + 1;
                bestStart = left;
            }

            char leftChar = s[left];
            window[leftChar]--;
            if (need.count(leftChar) && window[leftChar] < need[leftChar]) {
                formed--;
            }
            left++;
        }
    }

    return bestLen == INT_MAX ? "" : s.substr(bestStart, bestLen);
}
```

**Why check `window[c] == need[c]` exactly, not `>=`?** If we used `>=`, `formed` could be incremented multiple times for the same character as its count keeps growing past the requirement. Checking for equality ensures `formed` increments exactly once per character when its requirement is first met.

**Dry Run:** `s = "ADOBECODEBANC"`, `t = "ABC"`

```
need = {A:1, B:1, C:1}, required=3
window={}, formed=0, left=0, bestLen=INF

right=0 'A': window={A:1}. need[A]=1, window[A]==1==need[A] → formed=1
right=1 'D': window={A:1,D:1}. D not in need.
right=2 'O': window={...,O:1}. not in need.
right=3 'B': window={...,B:1}. need[B]=1, window[B]==1 → formed=2
right=4 'E': not in need.
right=5 'C': window={...,C:1}. need[C]=1, window[C]==1 → formed=3==required!
  Shrink: window valid. bestLen=6 (right-left+1=5-0+1=6), bestStart=0. "ADOBEC"
    remove s[0]='A'. window[A]=0 <1=need[A] → formed=2. left=1.
  formed≠required, exit shrink.
right=6 'O': not in need.
right=7 'D': not in need.
right=8 'E': not in need.
right=9 'B': window[B]=2. need[B]=1, window[B]≥need[B] already (was 1, becomes 2)
  window[B]==need[B]? 2==1? No → formed stays 2 (already counted at right=3... wait B count was already satisfying. Let's recheck: at right=3, window[B] became 1, matching need[B]=1, formed became 2. Now at right=9, window[B] becomes 2, which still satisfies need[B]=1 (window[B] >= need[B]), but formed doesn't change since the check is for equality at the moment of reaching exactly need[B]. formed stays 2.
right=10 'A': window[A]=1 (was 0 after removal, now 1). need[A]=1, window[A]==1 → formed=3==required!
  Shrink: bestLen check: right-left+1 = 10-1+1=10. Current best=6, not better.
    remove s[1]='D'. not in need. left=2.
  formed still 3 (D removal doesn't affect formed). Continue shrinking.
  right-left+1=10-2+1=9. Not better than 6.
    remove s[2]='O'. not in need. left=3.
  right-left+1=10-3+1=8. Not better.
    remove s[3]='B'. window[B]=1. need[B]=1, window[B]<need[B]? 1<1? No, equal not less. formed stays 3 (since condition is window[B] < need[B] which is false). left=4.
  right-left+1=10-4+1=7. Not better.
    remove s[4]='E'. not in need. left=5.
  right-left+1=10-5+1=6. Equal to best, not strictly less, so bestLen/bestStart NOT updated (we check before removal, so this check already happened before this removal). Let's recheck logic: the check happens BEFORE removing left char, at the top of the while loop. So actually each iteration: check best, then remove and move left.
  
Let me redo this more carefully with correct order:
  
  At right=10, formed=3. Enter while loop:
  Iteration 1: left=1. Check: len=10-1+1=10. Not better than 6.
    Remove s[1]='D'. left=2.
  Iteration 2: left=2. formed still 3 (D wasn't in need). Check: len=10-2+1=9. Not better.
    Remove s[2]='O'. left=3.
  Iteration 3: left=3. formed still 3. Check: len=10-3+1=8. Not better.
    Remove s[3]='B'. window[B]: was 2, now 1. need[B]=1. window[B] < need[B]? 1<1 false. formed stays 3. left=4.
  Iteration 4: left=4. formed still 3. Check: len=10-4+1=7. Not better.
    Remove s[4]='E'. not in need. left=5.
  Iteration 5: left=5. formed still 3. Check: len=10-5+1=6. Equal to best (6), not strictly less — no update.
    Remove s[5]='C'. window[C]=0 <1=need[C] → formed=2. left=6.
  formed=2≠3. Exit while.

right=11 'N': not in need.
right=12 'C': window[C]=1. need[C]=1, window[C]==1 → formed=3==required!
  Shrink: left=6. len=12-6+1=7. Not better than 6.
    Remove s[6]='O'. not in need. left=7.
  len=12-7+1=6. Equal, no update.
    Remove s[7]='D'. not in need. left=8.
  len=12-8+1=5. Better! bestLen=5, bestStart=8. "EBANC"... wait let's check s[8..12]="EBANC"? s="ADOBECODEBANC", indices 8-12 = E,B,A,N,C = "EBANC" — length 5.
    Remove s[8]='E'. not in need. left=9.
  len=12-9+1=4. Better! bestLen=4, bestStart=9. s[9..12]="BANC".
    Remove s[9]='B'. window[B]=0<1=need[B] → formed=2. left=10.
  formed≠3. Exit while.

End of string. bestLen=4, bestStart=9.
Answer: s.substr(9,4) = "BANC" ✓
```

**Time:** O(|s| + |t|) — each character in `s` is visited at most twice (once by `right`, once by `left`).
**Space:** O(|s| + |t|) for the two frequency maps (bounded by character set size in practice, often treated as O(1) for fixed alphabets).

---

## 7. Problem 6 — Minimum Window Subsequence (LC 727)

### Problem Statement

Given strings `s1` and `s2`, return the minimum contiguous substring `W` of `s1` such that `s2` is a **subsequence** of `W`. If no such window exists, return `""`. If multiple minimum-length windows exist, return the leftmost one.

**Constraints:** `1 <= s1.length <= 2*10^4`, `1 <= s2.length <= 100`, lowercase English letters.

**Examples:**
- `s1 = "abcdebdde"`, `s2 = "bde"` → `"bcde"` (not "bdde", which is also length 4 but starts later; not "deb", since subsequence requires order)

### Why This Differs from Minimum Window Substring

Minimum Window Substring (Problem 5) requires `t`'s characters to appear in the window with sufficient frequency, in **any order**. This problem requires `s2`'s characters to appear in the window in the **same relative order** as in `s2` — it's a subsequence match, not a frequency match.

This breaks the standard two-pointer "shrink while valid" logic from Problem 5, because removing characters from the left of a subsequence match can invalidate the match in a way that isn't simply "count dropped below requirement" — the order constraint means we can't use a frequency map at all.

### Approach 1 — Dynamic Programming (O(m*n) time, O(m*n) space)

**Definition:** `dp[i][j]` = the starting index in `s1` of the shortest substring `s1[?..i-1]` such that `s2[0..j-1]` is a subsequence of it, ending exactly at position `i-1`.

**Recurrence:**
- `dp[i][0] = i` (empty `s2` prefix needs an empty match, starting right at `i`)
- If `s1[i-1] == s2[j-1]`: `dp[i][j] = dp[i-1][j-1]` (this character extends the previous match)
- Else: `dp[i][j] = dp[i-1][j]` (carry forward the previous starting position, this character is just extra)

After filling the table, scan `dp[i][n]` for all `i` (where `n = |s2|`); for each valid (non -1) entry, the window length is `i - dp[i][n]`. Track the minimum.

```cpp
#include <bits/stdc++.h>
using namespace std;

string minWindow(string s1, string s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, -1));

    for (int i = 0; i <= m; i++) dp[i][0] = i; // empty s2 prefix matches starting at i

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i-1] == s2[j-1]) {
                dp[i][j] = dp[i-1][j-1];
            } else {
                dp[i][j] = dp[i-1][j];
            }
        }
    }

    int bestLen = INT_MAX, bestStart = -1;
    for (int i = 1; i <= m; i++) {
        if (dp[i][n] != -1) {
            int len = i - dp[i][n];
            if (len < bestLen) {
                bestLen = len;
                bestStart = dp[i][n];
            }
        }
    }

    return bestStart == -1 ? "" : s1.substr(bestStart, bestLen);
}
```

**Dry Run (conceptual):** `s1 = "abcdebdde"`, `s2 = "bde"`

```
s2 = "bde", n=3

dp[i][0] = i for all i (0 to 9)

Processing s1[0]='a' (i=1): doesn't match b,d,e at any j → dp[1][1..3] = dp[0][1..3] = -1,-1,-1
Processing s1[1]='b' (i=2): matches s2[0]='b' at j=1 → dp[2][1]=dp[1][0]=1. dp[2][2]=dp[1][2]=-1. dp[2][3]=-1.
Processing s1[2]='c' (i=3): no match → dp[3][1]=dp[2][1]=1. dp[3][2]=dp[2][2]=-1. dp[3][3]=-1.
Processing s1[3]='d' (i=4): matches s2[1]='d' at j=2 → dp[4][1]=dp[3][1]=1. dp[4][2]=dp[3][1]=1. dp[4][3]=dp[3][3]=-1.
Processing s1[4]='e' (i=5): matches s2[2]='e' at j=3 → dp[5][1]=dp[4][1]=1. dp[5][2]=dp[4][2]=1. dp[5][3]=dp[4][2]=1.
  dp[5][3]=1 → valid! length = 5-1=4. Window = s1[1..4] = "bcde". bestLen=4, bestStart=1.

Processing s1[5]='b' (i=6): matches s2[0]='b' → dp[6][1]=dp[5][0]=5. dp[6][2]=dp[5][2]=1. dp[6][3]=dp[5][3]=1.
  dp[6][3]=1 → length=6-1=5. Not better than 4.

Processing s1[6]='d' (i=7): matches s2[1]='d' → dp[7][1]=dp[6][1]=5. dp[7][2]=dp[6][1]=5. dp[7][3]=dp[6][3]=1.
  dp[7][3]=1 → length=7-1=6. Not better.

Processing s1[7]='d' (i=8): matches s2[1]='d' → dp[8][1]=dp[7][1]=5. dp[8][2]=dp[7][1]=5. dp[8][3]=dp[7][3]=1.
  dp[8][3]=1 → length=8-1=7. Not better.

Processing s1[8]='e' (i=9): matches s2[2]='e' → dp[9][1]=dp[8][1]=5. dp[9][2]=dp[8][2]=5. dp[9][3]=dp[8][2]=5.
  dp[9][3]=5 → length=9-5=4. Equal to current best (4), but NOT strictly less, so no update (keeps the earlier "bcde").

Final: bestLen=4, bestStart=1 → s1.substr(1,4) = "bcde" ✓
```

**Time:** O(m*n). **Space:** O(m*n) — can be optimized to O(n) by keeping only the previous row.

### Approach 2 — Two Pointers: Expand Then Contract-and-Rematch (O(m*n) worst case, often faster in practice)

**Intuition:** Use two pointers `i` (in `s1`) and `j` (in `s2`). Expand `i` while matching characters of `s2` in order. Once `j` reaches the end of `s2` (full match found ending at `i`), **backtrack**: walk `i` backward while re-matching `s2` from its end, to find the tightest possible left boundary for this particular match. This gives the minimal window for this match ending point. Then continue searching from just after the original match start.

```cpp
#include <bits/stdc++.h>
using namespace std;

string minWindow_twoPointer(string s1, string s2) {
    int m = s1.size(), n = s2.size();
    int i = 0, j = 0;
    int bestLen = INT_MAX, bestStart = -1;

    while (i < m) {
        if (s1[i] == s2[j]) {
            j++;
            if (j == n) {
                // Full match found ending at i. Backtrack to find tightest start.
                int end = i;
                j--;
                while (j >= 0) {
                    if (s1[i] == s2[j]) j--;
                    i--;
                }
                i++; // step back to the actual start position
                j++; // reset j to 0 for the next search

                if (end - i + 1 < bestLen) {
                    bestLen = end - i + 1;
                    bestStart = i;
                }
                // Continue searching from i+1 (move past this match's start)
            }
        }
        i++;
    }
    return bestStart == -1 ? "" : s1.substr(bestStart, bestLen);
}
```

**Why backtrack instead of just recording `[firstMatchStart, i]`?** The forward scan might match `s2` against a "loose" set of positions in `s1` that isn't the tightest possible window ending at `i`. Backtracking from the end re-matches `s2` from `s2[n-1]` backward, finding the rightmost (tightest) starting position that still completes the subsequence ending exactly at `i`.

**Time:** O(m*n) worst case (e.g., `s1 = "bbbbbbb...b"`, `s2 = "bb"` causes repeated rematching), but typically much faster than the DP approach in practice due to early termination.
**Space:** O(1) extra (excluding output).

### Edge Cases (Both Problems)

| Input | Expected | Reason |
|---|---|---|
| `s2` longer than `s1` | `""` | Impossible — handle as early return |
| `s2` is exactly `s1` | `s1` | Trivial full match |
| No valid subsequence exists | `""` | dp[i][n] never becomes valid for any i |
| Multiple minimum windows | Leftmost one | Use strict `<` (not `<=`) when updating best, processing left to right |

---

## 8. Substring vs Subsequence — Why the Sliding Window Breaks

This is the most important conceptual distinction in this problem set.

**Substring matching (Minimum Window Substring, LC 76):** We need the window to contain the right **frequency** of each required character, in **any order**. This is naturally suited to sliding window: as the window grows, frequencies only increase (monotonic), and as it shrinks, frequencies only decrease (monotonic). Both directions have clean, predictable effects on validity.

**Subsequence matching (Minimum Window Subsequence, LC 727):** We need the window to contain the required characters in a **specific order**. Order constraints break monotonicity in a way that frequency constraints don't: removing a character from the left of the window might still leave enough characters of each required type, but if the removed character was serving as the start of the subsequence match, the remaining characters might not be re-orderable into a valid subsequence without "looking back" — hence the need to either use DP (which explicitly tracks the match state) or a backtracking two-pointer scan.

**General principle:** Whenever the validity condition is "this set of characters appears with sufficient count" → sliding window with frequency maps applies directly. Whenever the validity condition involves "in this specific order" → you likely need DP or a more careful two-pointer technique with backtracking.

---

## 9. The "Exactly K" via "At Most K" Reduction

This pattern generalizes far beyond LC 992. Whenever you're asked to count something with an **exact** constraint, and counting with an **at most** constraint is easier (because "at most" is monotonic and sliding-window-friendly), use:

`exactly(K) = atMost(K) - atMost(K-1)`

**Other applications of this pattern:**
- Count subarrays with sum exactly K (when all elements are non-negative): `atMostSum(K) - atMostSum(K-1)` — though for general integers (with negatives), the prefix-sum hash map technique (covered in the Arrays Part III notes) is more direct.
- Count subarrays with exactly K odd numbers: `atMost(K) - atMost(K-1)`.
- Pivot index / balanced partition problems with an exact count constraint.

**Why "at most" is easier than "exactly":** "At most K" has a clean monotonic shrink condition — once the window violates "at most K," shrinking from the left can only help (removing elements can only decrease the distinct/count metric), and once it's restored, we know all sub-windows ending at the current right and starting from `left` to `right` are valid. "Exactly K" has no such clean shrink rule, because shrinking might take you from "K+1 distinct" to "K-1 distinct," skipping over the exact value you wanted.

---

## 10. Common Mistakes and Pitfalls

### Mistake 1 — Number of Substrings with All Three: forgetting `n - right`, using `right - left + 1` instead

**Wrong:**
```cpp
result += (right - left + 1); // counts within-window substrings, not extensions to the end
```

**Correct:**
```cpp
result += (n - right); // every substring s[left..k] for k from right to n-1 is valid
```

The logic here counts forward extensions (this window stays valid as `right` increases further, for THIS particular `left`), not backward starting points.

### Mistake 2 — Maximum Points from Cards: off-by-one in window size

**Wrong:**
```cpp
int windowSize = n - k - 1; // off by one
```

**Correct:**
```cpp
int windowSize = n - k; // taking k from ends leaves exactly n-k in the middle
```

### Mistake 3 — Longest Substring K Distinct: not erasing zero-count entries

**Wrong:**
```cpp
freq[s[left]]--;
left++;
// never erase — freq.size() now overcounts distinct characters (includes zero-count entries)
```

**Correct:**
```cpp
freq[s[left]]--;
if (freq[s[left]] == 0) freq.erase(s[left]);
left++;
```

### Mistake 4 — Subarrays with K Distinct: calling `atMostKDistinct` with k=0 when problem allows k=0 elsewhere

For LC 992, `k >= 1` is guaranteed, so `atMostKDistinct(nums, k-1)` with `k=1` calls `atMostKDistinct(nums, 0)`, which should correctly return 0 (no subarray can have 0 distinct integers, since arrays are non-empty). Verify the helper handles `k=0` correctly: the while loop `freq.size() > 0` triggers immediately for any non-empty window, correctly shrinking to an empty window before counting, contributing 0 each time. This works correctly without special-casing.

### Mistake 5 — Minimum Window Substring: using `>=` instead of `==` for the formed check

**Wrong:**
```cpp
if (need.count(c) && window[c] >= need[c]) formed++;
// formed increments every time, even after requirement is already met — overcounts formed
```

**Correct:**
```cpp
if (need.count(c) && window[c] == need[c]) formed++;
// increments exactly once, at the moment the requirement is first met
```

### Mistake 6 — Minimum Window Substring: decrementing `formed` with wrong condition

**Wrong:**
```cpp
window[leftChar]--;
if (need.count(leftChar) && window[leftChar] == 0) formed--; // wrong: should check < need, not == 0
```

**Correct:**
```cpp
window[leftChar]--;
if (need.count(leftChar) && window[leftChar] < need[leftChar]) formed--;
```

If `need[leftChar] = 2` and `window[leftChar]` drops from 2 to 1, the requirement is no longer met (1 < 2), so `formed` must decrement — but `window[leftChar] == 0` would miss this case.

### Mistake 7 — Minimum Window Subsequence: confusing this with Minimum Window Substring

Minimum Window Subsequence requires **order**; a standard frequency-map sliding window (correct for LC 76) gives wrong answers here, because it ignores order entirely. Always verify: does the problem require a specific relative order (subsequence) or just sufficient character counts (substring/frequency)?

### Mistake 8 — Minimum Window Subsequence DP: not initializing `dp[i][0] = i`

The base case `dp[i][0] = i` represents "matching an empty prefix of `s2` requires zero characters, and could start right at position `i`." Without this base case, the recurrence `dp[i][1]` (matching the first character of `s2`) has nothing to inherit from when `s1[i-1] == s2[0]`.

---

## 11. Interview Reference — Pattern Map

| Signal | Technique |
|---|---|
| Count substrings/subarrays containing all of a fixed small set of elements | Sliding window: shrink while valid, add `n - right` per valid left position |
| Take exactly k elements from both ends to maximize/minimize sum | Transform to "exclude middle window of size n-k", then min/max sliding window |
| Longest substring/subarray with at most K distinct elements | Sliding window with frequency map, shrink when `distinct > K` |
| Count subarrays with EXACTLY K of some property | `exactly(K) = atMost(K) - atMost(K-1)` |
| Minimum window containing all characters of target (any order, with frequency) | Sliding window + need/window maps + `formed` counter |
| Minimum window containing target as a SUBSEQUENCE (order matters) | DP `dp[i][j]` = start index, OR two-pointer expand-then-backtrack |
| Window validity check must be O(1) per step | Maintain a counter (`formed`, `distinct count`) instead of recomputing from scratch |

### Decision Tree: Substring vs Subsequence

```
Does the target's characters need to appear in a SPECIFIC ORDER in the window?
├── No (any order, just need the right counts) → Sliding window + frequency map (LC 76 style)
└── Yes (must appear in the same relative order) → DP or two-pointer backtrack (LC 727 style)
```

---

## 12. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Substrings with All Three Chars | Brute force | O(n²) | O(1) |
| Substrings with All Three Chars | Sliding window shrink | O(n) | O(1) |
| Substrings with All Three Chars | Last-seen index | O(n) | O(1) |
| Max Points from Cards | Brute force split | O(n) | O(n) |
| Max Points from Cards | Sliding window (excluded middle) | O(n) | O(1) |
| Longest Substring K Distinct | Sliding window + freq map | O(n) | O(k) |
| Subarrays with K Distinct | atMost(K) - atMost(K-1) | O(n) | O(k) |
| Minimum Window Substring | Sliding window + formed counter | O(\|s\|+\|t\|) | O(\|s\|+\|t\|) |
| Minimum Window Subsequence | DP | O(m*n) | O(m*n) |
| Minimum Window Subsequence | Two-pointer expand+backtrack | O(m*n) worst | O(1) |

---

## 13. Quick Revision Cheat Sheet

**Substrings with All Three Characters:**
```
count[3]={0,0,0}, left=0
for right: count[s[right]]++
  while all count>0: result += (n-right); count[s[left]]--; left++
```
Alternative O(1) space: track `last[a],last[b],last[c]`; if all ≥0, `count += min(last)+1`.

**Max Points from Cards:** `windowSize = n-k`. Find min-sum window of that size. `answer = totalSum - minWindowSum`.

**Longest Substring At Most K Distinct:** Sliding window + freq map. Shrink while `freq.size() > k`. Erase zero-count entries.

**Subarrays with Exactly K Distinct:** `exactly(K) = atMost(K) - atMost(K-1)`. `atMost` helper: sliding window, shrink while `distinct > k`, `count += (right-left+1)` each step.

**Minimum Window Substring (LC 76):**
```
need = freq(t), required = need.size()
window={}, formed=0, left=0
for right: window[s[right]]++
  if need.count(c) && window[c]==need[c]: formed++
  while formed==required:
    update best
    window[s[left]]--
    if need.count(s[left]) && window[s[left]] < need[s[left]]: formed--
    left++
```

**Minimum Window Subsequence (LC 727) — DP:**
```
dp[i][0]=i for all i
dp[i][j] = dp[i-1][j-1] if s1[i-1]==s2[j-1] else dp[i-1][j]
Scan dp[i][n] for all i; track min(i - dp[i][n])
```

**Substring vs Subsequence:** Order doesn't matter (just counts) → sliding window. Order matters → DP or backtrack two-pointer.

**The "at most K" trick:** Whenever "exactly K" seems hard with sliding window but "at most K" has a clean monotonic shrink rule, compute `exactly(K) = atMost(K) - atMost(K-1)`.

**Key invariant for window counting:** Once a window `[left, right]` satisfies a "contains all required elements" condition, every extension `[left, right'], right' > right` also satisfies it — only growing right never removes elements. This justifies `count += (n - right)` style additions.

**Backtracking in two-pointer subsequence match:** After finding a full subsequence match ending at `i`, walk backward re-matching `s2` from its last character to find the tightest (rightmost) valid start — this gives the minimal window for that particular ending point.
