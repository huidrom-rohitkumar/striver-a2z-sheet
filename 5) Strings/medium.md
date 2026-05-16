> Problems Covered: Sort Characters by Frequency (LC 451) | Maximum Nesting Depth of Parentheses (LC 1614) | Roman to Integer (LC 13) | Integer to Roman (LC 12) | Implement Atoi (LC 8) | Count Substrings with All Three Characters (LC 1358 / GFG) | Longest Palindromic Substring (LC 5) | Sum of Beauty of All Substrings (LC 1781) | Reverse Every Word in a String (LC 557)

---

## Table of Contents

1. [Core Mental Models for String Problems](#1-core-mental-models-for-string-problems)
2. [Sort Characters by Frequency](#2-sort-characters-by-frequency)
3. [Maximum Nesting Depth of Parentheses](#3-maximum-nesting-depth-of-parentheses)
4. [Roman to Integer and Integer to Roman](#4-roman-to-integer-and-integer-to-roman)
5. [Implement Atoi](#5-implement-atoi)
6. [Count Substrings with All Three Characters](#6-count-substrings-with-all-three-characters)
7. [Longest Palindromic Substring](#7-longest-palindromic-substring)
8. [Sum of Beauty of All Substrings](#8-sum-of-beauty-of-all-substrings)
9. [Reverse Every Word in a String](#9-reverse-every-word-in-a-string)
10. [Unified Pattern Reference](#10-unified-pattern-reference)
11. [Complexity Cheat Sheet](#11-complexity-cheat-sheet)
12. [Common Mistakes and Edge Cases](#12-common-mistakes-and-edge-cases)
13. [Quick Revision Cheat Sheet](#13-quick-revision-cheat-sheet)

---

## 1. Core Mental Models for String Problems

Strings are arrays of characters with additional semantic structure. The key shift from array thinking to string thinking is recognizing that characters carry meaning — their frequency, type, and position enable algorithms that have no array equivalent.

**Dominant patterns across this problem set:**

| Pattern | Problems |
|---|---|
| Character frequency mapping | Sort by Frequency, Sum of Beauty |
| Running depth simulation | Nesting Depth of Parentheses |
| Rule-based greedy parsing | Roman Numerals, Atoi |
| Complement counting via last-occurrence | Count Substrings (all three chars) |
| Expand around center | Longest Palindromic Substring |
| Incremental frequency update over subarrays | Sum of Beauty |
| In-place word boundary reversal | Reverse Every Word |

**C++ string operations worth internalizing:**

```cpp
s.length()              // O(1)
s[i]                    // O(1) access
s.substr(start, len)    // O(len) — creates a new string
s += c                  // O(1) amortized
reverse(s.begin() + l, s.begin() + r);  // O(r-l)
sort(s.begin(), s.end());                // O(n log n)

// String building: use += in a loop, not concatenation
// "a" + "b" + "c" ... in a loop is O(n²); += is O(n) amortized
```

---

## 2. Sort Characters by Frequency

**Problem:** Sort `s` in decreasing order of character frequency. Ties in frequency can be broken arbitrarily.

**LeetCode 451 — Sort Characters by Frequency**

```
"tree"   → "eert" or "eetr"    ('e' appears 2×)
"cccaaa" → "cccaaa" or "aaaccc"  (tie at 3×, either order valid)
"Aabb"   → "bbAa"              ('A' ≠ 'a': distinct characters)
```

### Approach 1 — Hash Map + Sort

**Intuition:** Count frequencies, sort character-frequency pairs in descending order, build the result string by repeating each character its frequency number of times.

```cpp
string frequencySort(string s) {
    unordered_map<char, int> freq;
    for (char c : s) freq[c]++;

    vector<pair<int, char>> sorted_freq;
    for (auto& [c, f] : freq) sorted_freq.push_back({f, c});
    sort(sorted_freq.begin(), sorted_freq.end(), greater<>());

    string result = "";
    for (auto& [f, c] : sorted_freq) result += string(f, c);
    return result;
}
```

**Time:** O(n + k log k) where k = number of distinct characters (at most 26 for lowercase, 128 for ASCII). For a fixed alphabet, O(n).
**Space:** O(k).

### Approach 2 — Bucket Sort (Optimal)

**Intuition:** The maximum frequency of any character is n (if the entire string is one character). Create n+1 buckets indexed by frequency; `bucket[f]` holds all characters with frequency exactly `f`. Scan from bucket `n` down to `1` to construct the result.

```cpp
string frequencySort(string s) {
    int n = s.size();
    unordered_map<char, int> freq;
    for (char c : s) freq[c]++;

    vector<vector<char>> bucket(n + 1);
    for (auto& [c, f] : freq) bucket[f].push_back(c);

    string result = "";
    for (int f = n; f >= 1; f--)
        for (char c : bucket[f])
            result += string(f, c);
    return result;
}
```

**Time:** O(n) — counting O(n), bucketing O(k), result building O(n).
**Space:** O(n) for buckets.

### Dry Run

Input: `s = "tree"`

```
freq: {'t':1, 'r':1, 'e':2}
sorted (bucket): bucket[2]=['e'], bucket[1]=['t','r']
Result: "ee" + "t" + "r" = "eetr"  ✓
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `"a"` | `"a"` | Single character |
| `"Aabb"` | `"bbAa"` | `'A'` and `'a'` are distinct (ASCII 65 vs 97) |
| `"2a554"` | `"5542a"` | Digits and letters both valid |

---

## 3. Maximum Nesting Depth of Parentheses

**Problem:** Given a valid parenthesized string, return its maximum nesting depth.

**LeetCode 1614 — Maximum Nesting Depth of Parentheses**

```
"(1+(2*3)+((8)/4))+1"  → 3
"()()"                 → 1
"(())"                 → 2
"1"                    → 0
```

### Core Insight — Depth as a Running Counter

The depth at any position equals the number of unclosed `(` seen so far. Each `(` opens a new level; each `)` closes one. The maximum value the depth counter reaches is the answer.

```cpp
int maxDepth(string s) {
    int depth = 0, maxD = 0;
    for (char c : s) {
        if      (c == '(') maxD = max(maxD, ++depth);
        else if (c == ')') depth--;
        // all other characters: no effect
    }
    return maxD;
}
```

**Why it works:** The string is guaranteed valid, so `depth` never goes negative and returns to 0 at the end. The running maximum of `depth` is exactly the deepest nesting.

**Time:** O(n). **Space:** O(1).

### Dry Run

`s = "(1+(2*3)+((8)/4))+1"`

```
'(' → depth=1, maxD=1
'1','+' → no change
'(' → depth=2, maxD=2
'2','*','3' → no change
')' → depth=1
'+' → no change
'(' → depth=2
'(' → depth=3, maxD=3
'8','/','4' → no change
')' → depth=2
')' → depth=1
')' → depth=0
'+','1' → no change
Return 3  ✓
```

---

## 4. Roman to Integer and Integer to Roman

### Symbol Reference

```
I=1  V=5  X=10  L=50  C=100  D=500  M=1000

Six subtractive pairs:
IV=4  IX=9  XL=40  XC=90  CD=400  CM=900
```

### 4.1 Roman to Integer (LC 13)

**Key insight:** In standard Roman numerals, a symbol is subtracted if the next symbol is strictly larger; otherwise it is added.

```cpp
int romanToInt(string s) {
    unordered_map<char, int> val = {
        {'I',1},{'V',5},{'X',10},{'L',50},
        {'C',100},{'D',500},{'M',1000}
    };
    int result = 0;
    for (int i = 0; i < (int)s.size(); i++) {
        if (i + 1 < (int)s.size() && val[s[i]] < val[s[i+1]])
            result -= val[s[i]];
        else
            result += val[s[i]];
    }
    return result;
}
```

**Dry Run for "MCMXCIV" (= 1994):**

```
M(1000): next=C(100). 1000>=100 → +1000. result=1000.
C(100):  next=M(1000). 100<1000  → -100.  result=900.
M(1000): next=X(10).  1000>=10  → +1000. result=1900.
X(10):   next=C(100). 10<100    → -10.   result=1890.
C(100):  next=I(1).   100>=1    → +100.  result=1990.
I(1):    next=V(5).   1<5       → -1.    result=1989.
V(5):    no next.               → +5.    result=1994.
Return 1994  ✓
```

**Time:** O(n). **Space:** O(1) — map has exactly 7 entries.

### 4.2 Integer to Roman (LC 12)

**Key insight (greedy):** Use a lookup table containing all values including the six subtractive pairs, ordered largest to smallest. Repeatedly subtract the largest value that fits and append the corresponding symbol.

```cpp
string intToRoman(int num) {
    vector<pair<int, string>> table = {
        {1000,"M"},{900,"CM"},{500,"D"},{400,"CD"},
        {100,"C"}, {90,"XC"},{50,"L"}, {40,"XL"},
        {10,"X"},  {9,"IX"}, {5,"V"},  {4,"IV"},
        {1,"I"}
    };
    string result = "";
    for (auto& [val, sym] : table) {
        while (num >= val) { result += sym; num -= val; }
    }
    return result;
}
```

**Why greedy works:** The table covers every critical boundary including all subtractive pairs. Processing from largest to smallest and always subtracting the largest possible value constructs the unique, canonical Roman representation. The system has no ambiguity for integers in [1, 3999].

**Time:** O(1) — num ≤ 3999; the maximum output is 15 characters ("MMMCMXCIX"). **Space:** O(1).

**Dry Run for num = 1994:**

```
1000: 1994>=1000 → "M", num=994.
900:  994>=900   → "CM", num=94.
500..100: skip.
90:   94>=90     → "XC", num=4.
50..5: skip.
4:    4>=4       → "IV", num=0.
Return "MCMXCIV"  ✓
```

### Edge Cases

| Input | Output |
|---|---|
| `1` | `"I"` |
| `4` | `"IV"` |
| `9` | `"IX"` |
| `3999` | `"MMMCMXCIX"` |

---

## 5. Implement Atoi

**Problem:** Convert a string to a 32-bit signed integer following these rules: (1) skip leading whitespace, (2) read optional sign, (3) read digits until non-digit or end, (4) clamp to `[INT_MIN, INT_MAX]` on overflow, (5) return 0 if no valid conversion.

**LeetCode 8 — Implement Atoi**

```
"42"           → 42
"   -042"      → -42
"1337c0d3"     → 1337   (stops at 'c')
"words and 3"  → 0      (no leading digit or sign)
"-91283472332" → -2147483648  (INT_MIN clamped)
```

### The Five-Step Algorithm

```cpp
int myAtoi(string s) {
    int i = 0, n = s.size(), sign = 1;
    long long result = 0;

    // Step 1: Skip leading whitespace
    while (i < n && s[i] == ' ') i++;

    // Step 2: Read optional sign (exactly one)
    if (i < n && (s[i] == '+' || s[i] == '-')) {
        sign = (s[i] == '-') ? -1 : 1;
        i++;
    }

    // Step 3 & 4: Read digits, check overflow before each multiplication
    while (i < n && isdigit(s[i])) {
        int digit = s[i] - '0';
        if (result > (INT_MAX - digit) / 10) {   // would overflow
            return (sign == 1) ? INT_MAX : INT_MIN;
        }
        result = result * 10 + digit;
        i++;
    }

    // Step 5: Apply sign
    return (int)(sign * result);
}
```

**Overflow check derivation:** We want to detect `result * 10 + digit > INT_MAX` before it happens. Rearranging: `result > (INT_MAX - digit) / 10`. Integer division truncates, making this check conservative — if `result` strictly exceeds this value, the next digit definitely overflows.

**Time:** O(n). **Space:** O(1).

### Dry Run

`s = "  -042"`:

```
Step 1: skip two spaces. i=2.
Step 2: s[2]='-' → sign=-1, i=3.
Step 3: s[3]='0'→result=0. s[4]='4'→result=4. s[5]='2'→result=42.
Step 5: return -1*42 = -42  ✓
```

`s = "-91283472332"` (overflow):

```
sign=-1.
Digits: 9,1,2,8,3,4,7,2,3,3,2...
When result reaches 214748364 and next digit is 8:
  (INT_MAX - 8) / 10 = (2147483647 - 8) / 10 = 214748363
  result(214748364) > 214748363? YES → return INT_MIN.
Return -2147483648  ✓
```

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `"  "` | `0` | No digits after whitespace |
| `"+"` | `0` | Sign only, no digits follow |
| `"+-12"` | `0` | Only one sign character is accepted |
| `"2147483648"` | `2147483647` | Just above INT_MAX, clamped |

---

## 6. Count Substrings with All Three Characters

**Problem:** Given a string `s` of only 'a', 'b', 'c', count substrings containing at least one of each.

**LeetCode 1358 / GFG**

```
"abcabc" → 10
"aaacb"  → 3
"abc"    → 1
```

### Approach 1 — Brute Force O(n²)

```cpp
int countSubstrings(string s) {
    int n = s.size(), count = 0;
    for (int i = 0; i < n; i++) {
        int freq[3] = {};
        for (int j = i; j < n; j++) {
            freq[s[j]-'a']++;
            if (freq[0] > 0 && freq[1] > 0 && freq[2] > 0) count++;
        }
    }
    return count;
}
```

### Approach 2 — Last-Occurrence Formula O(n)

**Key insight:** For a substring ending at position `j` to contain all three characters, its left boundary must be at most `min(last_a, last_b, last_c)`. The number of valid left boundaries is `1 + min(last_a, last_b, last_c)` (positions 0 through min, inclusive). When any of the three has never been seen, the minimum is -1, contributing 0.

```cpp
int countSubstrings(string s) {
    int n = s.size(), count = 0;
    int last[3] = {-1, -1, -1};
    for (int j = 0; j < n; j++) {
        last[s[j] - 'a'] = j;
        count += 1 + min({last[0], last[1], last[2]});
    }
    return count;
}
```

**Time:** O(n). **Space:** O(1).

### Approach 3 — Sliding Window O(n)

For each left boundary `l`, find the minimum `r` where `s[l..r]` contains all three. Once valid, all substrings `s[l..r], s[l..r+1], ..., s[l..n-1]` are valid — add `n - r` to count. Shrink from left and repeat.

```cpp
int countSubstrings(string s) {
    int n = s.size(), count = 0;
    int freq[3] = {}, right = 0;
    for (int left = 0; left < n; left++) {
        while (right < n && (freq[0]==0 || freq[1]==0 || freq[2]==0))
            freq[s[right++]-'a']++;
        if (freq[0]>0 && freq[1]>0 && freq[2]>0)
            count += n - right + 1;
        freq[s[left]-'a']--;
    }
    return count;
}
```

**Time:** O(n) — `left` and `right` each advance at most n times. **Space:** O(1).

### Dry Run (Last-Occurrence)

`s = "abcabc"`, n=6

```
j=0: 'a', last=[0,-1,-1]. min=-1. count+=0.
j=1: 'b', last=[0,1,-1].  min=-1. count+=0.
j=2: 'c', last=[0,1,2].   min=0.  count+=1. total=1.
j=3: 'a', last=[3,1,2].   min=1.  count+=2. total=3.
j=4: 'b', last=[3,4,2].   min=2.  count+=3. total=6.
j=5: 'c', last=[3,4,5].   min=3.  count+=4. total=10.
Return 10  ✓
```

---

## 7. Longest Palindromic Substring

**Problem:** Return the longest palindromic substring of `s`. No dynamic programming.

**LeetCode 5 — Longest Palindromic Substring**

```
"babad"  → "bab" or "aba"
"cbbd"   → "bb"
"a"      → "a"
```

### What Makes a Palindrome

Every palindrome has a center. It either:
- Is a single character (odd-length: "aba", center at 'b'), or
- Is a gap between two equal characters (even-length: "abba", center between the two 'b's).

Every palindrome is uniquely determined by its center and expands symmetrically from it.

### Approach 1 — Brute Force O(n³)

Check every substring for the palindrome property. Unacceptable for n ~ 1000.

### Approach 2 — Expand Around Center O(n²) time, O(1) space

**Intuition:** For each of the `2n - 1` possible centers, expand outward as long as the characters on both sides match. Track the longest expansion found.

```cpp
// Returns the length of the longest palindrome centered at (l, r)
int expand(const string& s, int l, int r) {
    while (l >= 0 && r < (int)s.size() && s[l] == s[r]) {
        l--; r++;
    }
    // Loop exits when s[l] != s[r] or boundary hit.
    // Palindrome spans [l+1, r-1]; length = (r-1) - (l+1) + 1 = r - l - 1.
    return r - l - 1;
}

string longestPalindrome(string s) {
    int n = s.size(), start = 0, maxLen = 1;
    for (int i = 0; i < n; i++) {
        int len1 = expand(s, i, i);       // odd-length centered at i
        int len2 = expand(s, i, i + 1);   // even-length centered between i and i+1
        int best = max(len1, len2);
        if (best > maxLen) {
            maxLen = best;
            start = i - (best - 1) / 2;  // works for both odd and even
        }
    }
    return s.substr(start, maxLen);
}
```

**Recovering start index:** For an odd palindrome of length `len` centered at `i`: `start = i - (len-1)/2`. For an even palindrome of length `len` centered between `i` and `i+1`: `start = i - len/2 + 1`. The formula `i - (best-1)/2` handles both correctly due to integer division.

**Time:** O(n²) — n centers, each expansion O(n) worst case. **Space:** O(1).

### Dry Run

`s = "babad"`, n=5

```
i=0, center='b':
  odd  expand(0,0): l=-1,r=1. len=1-(-1)-1=1.
  even expand(0,1): s[0]='b', s[1]='a'. b≠a. len=1-0-1=0.
  best=1. maxLen stays 1.

i=1, center='a':
  odd  expand(1,1): (l=1,r=1)→(l=0,r=2)='b','b' match→(l=-1,r=3). len=3-(-1)-1=3.
  even expand(1,2): s[1]='a', s[2]='b'. a≠b. len=0.
  best=3>1. maxLen=3, start=1-(3-1)/2=1-1=0.

i=2, center='b':
  odd  expand(2,2): (2,2)→(1,3)='a','a' match→(0,4)='b','d' b≠d. len=4-0-1=3.
  even expand(2,3): s[2]='b', s[3]='a'. b≠a. len=0.
  best=3. NOT > maxLen=3. No update.

i=3,4: expansions yield length ≤ 3. No update.

Return s.substr(0,3) = "bab"  ✓
```

### DP Approach (for reference — O(n²) time, O(n²) space)

```
dp[i][j] = true if s[i..j] is a palindrome
Base: dp[i][i] = true, dp[i][i+1] = (s[i]==s[i+1])
Recurrence: dp[i][j] = (s[i]==s[j]) && dp[i+1][j-1]
```

Expand-around-center achieves the same time with O(1) space, so it is always preferred over DP.

### Manacher's Algorithm — O(n) Reference

Manacher's inserts sentinel characters between all characters (e.g., `"aba"` → `"#a#b#a#"`) to unify odd and even cases, then uses previously computed palindrome radii to skip redundant expansion. Result: O(n) time. In interviews, expand-around-center is almost always the expected answer.

### Edge Cases

| Input | Output | Reason |
|---|---|---|
| `"a"` | `"a"` | Single character |
| `"aa"` | `"aa"` | Even-length palindrome |
| `"ab"` | `"a"` or `"b"` | No length-2 palindrome |
| `"racecar"` | `"racecar"` | Entire string is palindrome |

---

## 8. Sum of Beauty of All Substrings

**Problem:** The beauty of a substring is `max_frequency - min_frequency` among characters that appear in it. Return the sum of beauty of all substrings of `s`.

**LeetCode 1781 — Sum of Beauty of All Substrings**

```
"aabcb" → 5
"aabcbaa" → 17
```

**Beauty examples:**
```
"aab": a→2, b→1. beauty = 2-1 = 1.
"aa":  a→2.       beauty = 0   (single distinct char: max=min).
"abc": a→1,b→1,c→1. beauty = 0.
```

### Approach — Incremental Frequency Array O(n²)

**Insight:** For each starting index `i`, extend `j` rightward while maintaining a frequency array `freq[26]`. At each step, scan all 26 entries to find the non-zero max and min. Total work per (i,j) pair is O(26) = O(1).

**Why not more efficient:** Min-frequency is non-monotone as the window grows (adding a new character can lower it). No known sub-quadratic approach exists for this specific problem as stated.

```cpp
int beautySum(string s) {
    int n = s.size(), ans = 0;
    for (int i = 0; i < n; i++) {
        int freq[26] = {};
        for (int j = i; j < n; j++) {
            freq[s[j] - 'a']++;
            int maxF = 0, minF = INT_MAX;
            for (int k = 0; k < 26; k++) {
                if (freq[k] > 0) {
                    maxF = max(maxF, freq[k]);
                    minF = min(minF, freq[k]);
                }
            }
            ans += maxF - minF;
        }
    }
    return ans;
}
```

**Critical:** Min is computed only over **non-zero** frequencies. A character absent from the substring does not participate in the beauty calculation.

**Time:** O(26 * n²) = O(n²). **Space:** O(26) = O(1).

### Dry Run

`s = "aabcb"`, n=5

```
i=0:
  j=0: freq[a]=1. max=1,min=1. beauty=0.
  j=1: freq[a]=2. max=2,min=2. beauty=0.
  j=2: freq[a]=2,freq[b]=1. max=2,min=1. beauty=1. ans=1.
  j=3: freq[a]=2,freq[b]=1,freq[c]=1. max=2,min=1. beauty=1. ans=2.
  j=4: freq[a]=2,freq[b]=2,freq[c]=1. max=2,min=1. beauty=1. ans=3.

i=1:
  j=4: freq[a]=1,freq[b]=2,freq[c]=1. beauty=1. ans=4.

i=2:
  j=4: freq[b]=2,freq[c]=1. beauty=1. ans=5.

i=3,4: all substrings have 1-2 distinct chars with equal frequency. beauty=0.

Return 5  ✓
```

---

## 9. Reverse Every Word in a String

**Problem:** Reverse the characters in each individual word, keeping word positions and spaces intact.

**LeetCode 557 — Reverse Words in a String III**

```
"Let's take LeetCode contest" → "s'teL ekat edoCteeL tsetnoC"
"Mr Ding"                     → "rM gniD"
```

**Distinguish from similar problems:**

```
LC 344 — Reverse String:           reverse the entire character array.
LC 151 — Reverse Words in String:  reverse the ORDER of words.  "hello world" → "world hello"
LC 557 — Reverse Words III:        reverse CHARACTERS within each word. "hello world" → "olleh dlrow"
```

### Approach 1 — Split and Reverse Each Word

```cpp
string reverseWords(string s) {
    string result = "", word = "";
    s += ' ';
    for (char c : s) {
        if (c == ' ') {
            reverse(word.begin(), word.end());
            if (!result.empty()) result += ' ';
            result += word;
            word = "";
        } else {
            word += c;
        }
    }
    return result;
}
```

**Time:** O(n). **Space:** O(n).

### Approach 2 — In-Place Two-Pointer (Optimal)

**Intuition:** Find word boundaries using a `start` pointer. When a space or end-of-string is encountered, reverse the segment `s[start..i-1]` in-place.

```cpp
string reverseWords(string s) {
    int n = s.size(), start = 0;
    for (int i = 0; i <= n; i++) {
        if (i == n || s[i] == ' ') {
            reverse(s.begin() + start, s.begin() + i);
            start = i + 1;
        }
    }
    return s;
}
```

**Time:** O(n) — every character is reversed at most once across all `reverse` calls (each character belongs to exactly one word).
**Space:** O(1) — in-place.

### Dry Run

`s = "Let's take"`, n=10

```
i=0..4: s[5]=' '
  reverse s[0..4] = "Let's" → "s'teL".
  start=6.

i=6..9: i=10=n
  reverse s[6..9] = "take" → "ekat".

Return "s'teL ekat"  ✓
```

---

## 10. Unified Pattern Reference

### Pattern 1 — Character Frequency Mapping

```cpp
// For lowercase-only strings: O(1) space
int freq[26] = {};
for (char c : s) freq[c - 'a']++;

// For arbitrary characters: O(k) space
unordered_map<char, int> freq;
for (char c : s) freq[c]++;
```

Used in: Sort by Frequency (counting then sorting), Sum of Beauty (tracking max/min per substring window).

### Pattern 2 — Running Depth Counter

```cpp
int depth = 0;
for (char c : s) {
    if      (c == '(') depth++;
    else if (c == ')') depth--;
}
```

Used in: Nesting Depth (take running max); Valid Parentheses (check depth ever goes negative); Minimum Remove to Make Valid.

### Pattern 3 — Rule-Based Greedy Parsing

**Roman Numerals:** Scan left to right. If current symbol's value is less than the next symbol's, subtract it; otherwise add it.

**Atoi:** Fixed state machine — skip spaces → read one sign → read digits → stop on non-digit.

### Pattern 4 — Counting Valid Substrings via Last-Occurrence

**For each right endpoint j, count how many valid left endpoints exist:**

```cpp
int last[3] = {-1, -1, -1};
for (int j = 0; j < n; j++) {
    last[s[j]-'a'] = j;
    count += 1 + min({last[0], last[1], last[2]});
}
```

**Why it works:** A substring ending at `j` is valid iff its left boundary ≤ min(last_a, last_b, last_c). There are `min(...) + 1` valid left boundaries. Contributes 0 when any character has not been seen yet (min = -1).

### Pattern 5 — Expand Around Center

```cpp
auto expand = [&](int l, int r) -> int {
    while (l >= 0 && r < n && s[l] == s[r]) { l--; r++; }
    return r - l - 1;  // palindrome length
};
for (int i = 0; i < n; i++) {
    int len = max(expand(i, i), expand(i, i+1));
    // odd: expand(i,i); even: expand(i,i+1)
}
```

Both odd-length and even-length palindromes must be tried for every center.

### Pattern 6 — Word-by-Word Processing

```cpp
int start = 0;
for (int i = 0; i <= n; i++) {
    if (i == n || s[i] == ' ') {
        // process s[start..i-1]
        start = i + 1;
    }
}
```

Identifies word segments without splitting into substrings — O(1) extra space.

---

## 11. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Sort Characters by Frequency | Hash map + sort | O(n + k log k) | O(k) |
| Sort Characters by Frequency | Bucket sort | O(n) | O(n) |
| Nesting Depth of Parentheses | Running counter | O(n) | O(1) |
| Roman to Integer | Single pass | O(n) | O(1) |
| Integer to Roman | Greedy with table | O(1) | O(1) |
| Implement Atoi | Single pass | O(n) | O(1) |
| Count Substrings (all 3 chars) | Brute force | O(n²) | O(1) |
| Count Substrings (all 3 chars) | Last-occurrence formula | O(n) | O(1) |
| Count Substrings (all 3 chars) | Sliding window | O(n) | O(1) |
| Longest Palindromic Substring | Brute force | O(n³) | O(1) |
| Longest Palindromic Substring | Dynamic programming | O(n²) | O(n²) |
| Longest Palindromic Substring | Expand around center | O(n²) | O(1) |
| Longest Palindromic Substring | Manacher's algorithm | O(n) | O(n) |
| Sum of Beauty of All Substrings | Incremental freq array | O(26 n²) = O(n²) | O(1) |
| Reverse Every Word | Split and reverse | O(n) | O(n) |
| Reverse Every Word | In-place two-pointer | O(n) | O(1) |

---

## 12. Common Mistakes and Edge Cases

### Mistake 1 — Sort by Frequency: Case Sensitivity

```cpp
// WRONG: treating 'A' and 'a' as the same
freq[tolower(c)]++;

// CORRECT: 'A' (ASCII 65) and 'a' (ASCII 97) are distinct characters
freq[c]++;   // or freq[c-'a'] only when input is guaranteed lowercase
```

### Mistake 2 — Atoi: Checking Overflow After Multiplication

```cpp
// WRONG: result*10 overflows before the check runs
result = result * 10 + digit;
if (result > INT_MAX) return sign==1 ? INT_MAX : INT_MIN;

// CORRECT: check BEFORE the multiplication
if (result > (INT_MAX - digit) / 10) return sign==1 ? INT_MAX : INT_MIN;
result = result * 10 + digit;
```

### Mistake 3 — Atoi: Accepting Multiple Sign Characters

```cpp
// WRONG: loop accepts "++42", "+-1"
while (i<n && (s[i]=='+'||s[i]=='-')) { ... i++; }

// CORRECT: read exactly one sign character
if (i<n && (s[i]=='+'||s[i]=='-')) {
    sign = (s[i]=='-') ? -1 : 1;
    i++;
}
```

### Mistake 4 — Longest Palindrome: Missing Even-Length Palindromes

```cpp
// WRONG: only odd-length centers checked
for (int i=0; i<n; i++) expand(s, i, i);

// CORRECT: both odd and even for every center
for (int i=0; i<n; i++) {
    expand(s, i, i);      // odd
    expand(s, i, i+1);    // even (handles "bb", "abba", etc.)
}
```

### Mistake 5 — Sum of Beauty: Including Zero-Frequency Characters in Min

```cpp
// WRONG: returns 0 if any of the 26 slots is zero
int minF = *min_element(freq, freq+26);

// CORRECT: only non-zero entries define the minimum
int minF = INT_MAX;
for (int k=0; k<26; k++)
    if (freq[k] > 0) minF = min(minF, freq[k]);
```

### Mistake 6 — Count Substrings: Off-by-One in Last-Occurrence Formula

```cpp
// WRONG: missing the +1
count += min({last[0], last[1], last[2]});

// CORRECT: there are min(...)+1 valid left boundaries (positions 0 through min)
count += 1 + min({last[0], last[1], last[2]});
```

### Mistake 7 — Integer to Roman: Incomplete Table (Missing Subtractive Pairs)

```cpp
// WRONG: table only has the 7 base symbols; produces wrong output for 4, 9, 40, etc.
vector<pair<int,string>> table = {{1000,"M"},{500,"D"},{100,"C"},{50,"L"},
                                   {10,"X"},{5,"V"},{1,"I"}};

// CORRECT: include all 13 entries, especially the 6 subtractive pairs
vector<pair<int,string>> table = {{1000,"M"},{900,"CM"},{500,"D"},{400,"CD"},
    {100,"C"},{90,"XC"},{50,"L"},{40,"XL"},{10,"X"},{9,"IX"},{5,"V"},{4,"IV"},{1,"I"}};
```

### Mistake 8 — Confusing LC 151 and LC 557

```
LC 151: "hello world" → "world hello"    (reverse word ORDER)
LC 557: "hello world" → "olleh dlrow"   (reverse CHARACTERS in each word)
```

### Edge Cases Summary

| Problem | Critical Edge Cases |
|---|---|
| Sort by Frequency | Single character, ties in frequency (any order valid), mixed case |
| Nesting Depth | No parentheses → 0; only one level `"()"` → 1; consecutive `"()()"` → 1 |
| Roman to Integer | Single symbol `"I"`, subtractive pairs `"IV"`, `"CM"`, max `"MMMCMXCIX"` |
| Integer to Roman | 1, 4, 9, 3999 |
| Atoi | Empty after spaces, sign only, `"+-12"` → 0, exact INT_MAX/INT_MIN |
| Count Substrings | Only one distinct char → 0; `"abc"` → 1; `"abcabc"` → 10 |
| Longest Palindrome | Single char, entire string is palindrome, even-length palindromes |
| Sum of Beauty | Single char → 0 per substring; all same char → 0 everywhere |
| Reverse Every Word | Single word, last word (no trailing space) |

---

## 13. Quick Revision Cheat Sheet

**Sort Characters by Frequency (Bucket Sort)**

```cpp
unordered_map<char,int> freq;
for (char c : s) freq[c]++;
vector<vector<char>> bucket(n+1);
for (auto& [c,f] : freq) bucket[f].push_back(c);
string res;
for (int f=n; f>=1; f--)
    for (char c : bucket[f]) res += string(f, c);
```

**Nesting Depth**

```cpp
int depth=0, maxD=0;
for (char c : s) {
    if      (c=='(') maxD=max(maxD, ++depth);
    else if (c==')') depth--;
}
return maxD;
```

**Roman to Integer**

```cpp
// subtract if current < next, otherwise add
if (i+1<n && val[s[i]] < val[s[i+1]]) result -= val[s[i]];
else                                    result += val[s[i]];
```

**Integer to Roman**

```cpp
// Greedy: 13-entry table including 6 subtractive pairs, largest first
// while(num >= val) { result += sym; num -= val; }
```

**Implement Atoi**

```cpp
// 1. skip spaces  2. one optional sign  3. digits with pre-check overflow
if (result > (INT_MAX - digit) / 10) return sign==1 ? INT_MAX : INT_MIN;
result = result*10 + digit;
```

**Count Substrings (All Three Characters)**

```cpp
// Last-occurrence formula — cleanest O(n) approach:
int last[3]={-1,-1,-1}, count=0;
for (int j=0; j<n; j++) {
    last[s[j]-'a']=j;
    count += 1 + min({last[0], last[1], last[2]});
}
```

**Longest Palindromic Substring**

```cpp
// expand(l,r) returns palindrome length; palindrome spans [l+1..r-1]
auto expand = [&](int l, int r) {
    while(l>=0 && r<n && s[l]==s[r]) { l--; r++; }
    return r-l-1;
};
int start=0, maxLen=1;
for (int i=0; i<n; i++) {
    for (int len : {expand(i,i), expand(i,i+1)}) {
        if (len > maxLen) { maxLen=len; start=i-(len-1)/2; }
    }
}
return s.substr(start, maxLen);
// O(n²) time, O(1) space. Must check BOTH (i,i) and (i,i+1) per center.
```

**Sum of Beauty**

```cpp
for (int i=0; i<n; i++) {
    int freq[26]={};
    for (int j=i; j<n; j++) {
        freq[s[j]-'a']++;
        int mx=0, mn=INT_MAX;
        for (int k=0; k<26; k++) if(freq[k]>0) { mx=max(mx,freq[k]); mn=min(mn,freq[k]); }
        ans += mx-mn;
    }
}
// Min ONLY over non-zero frequency slots.
```

**Reverse Every Word**

```cpp
int start=0, n=s.size();
for (int i=0; i<=n; i++) {
    if (i==n || s[i]==' ') {
        reverse(s.begin()+start, s.begin()+i);
        start=i+1;
    }
}
// In-place. O(n) time, O(1) space.
```

**Key Rules to Memorize**

```
Sort by Freq:    Bucket sort index = frequency. Scan buckets n→1.
Nesting Depth:   Running max on '(', decrement on ')'.
Roman→Int:       Subtract if current value < next value. Else add.
Int→Roman:       Greedy from 1000 down. Include all 6 subtractive pairs.
Atoi:            Check overflow BEFORE multiply: result > (INT_MAX-digit)/10.
Count Substrings: count += 1 + min(last_a, last_b, last_c). Returns 0 when any is -1.
Palindrome:      expand(i,i) for odd, expand(i,i+1) for even. O(n²), O(1).
Beauty Sum:      Min over non-zero freq slots only.
Reverse Words:   Find space boundaries, reverse each word segment in-place.
```

---
