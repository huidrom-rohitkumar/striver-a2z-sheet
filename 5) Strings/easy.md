
## Problems Covered

- [LC 1021 — Remove Outermost Parentheses](https://leetcode.com/problems/remove-outermost-parentheses/)
- [LC 151 — Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/)
- [LC 125 — Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)
- [LC 1903 — Largest Odd Number in a String](https://leetcode.com/problems/largest-odd-number-in-string/)
- [LC 14 — Longest Common Prefix](https://leetcode.com/problems/longest-common-prefix/)
- [LC 205 — Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/)
- [LC 796 — Rotate String](https://leetcode.com/problems/rotate-string/)
- [LC 242 — Valid Anagram](https://leetcode.com/problems/valid-anagram/)

---

## Table of Contents

1. [The Unifying Mental Model](#1-the-unifying-mental-model)
2. [C++ String Essentials](#2-c-string-essentials)
3. [ASCII and Character Arithmetic](#3-ascii-and-character-arithmetic)
4. [Problem: Remove Outermost Parentheses (LC 1021)](#4-problem-remove-outermost-parentheses-lc-1021)
5. [Problem: Reverse Words in a String (LC 151)](#5-problem-reverse-words-in-a-string-lc-151)
6. [Problem: Valid Palindrome (LC 125)](#6-problem-valid-palindrome-lc-125)
7. [Problem: Largest Odd Number in a String (LC 1903)](#7-problem-largest-odd-number-in-a-string-lc-1903)
8. [Problem: Longest Common Prefix (LC 14)](#8-problem-longest-common-prefix-lc-14)
9. [Problem: Isomorphic Strings (LC 205)](#9-problem-isomorphic-strings-lc-205)
10. [Problem: Check Whether One String is a Rotation (LC 796)](#10-problem-check-whether-one-string-is-a-rotation-lc-796)
11. [Problem: Valid Anagram (LC 242)](#11-problem-valid-anagram-lc-242)
12. [Related Concepts and Extensions](#12-related-concepts-and-extensions)
13. [Interview Pattern Map](#13-interview-pattern-map)
14. [Common Mistakes and Pitfalls](#14-common-mistakes-and-pitfalls)
15. [Complexity Cheat Sheet](#15-complexity-cheat-sheet)
16. [Quick Revision Cheat Sheet](#16-quick-revision-cheat-sheet)

---

## 1. The Unifying Mental Model

String problems combine arrays, hashing, two pointers, parsing, and greedy logic. Most beginner-to-medium string problems reduce to one of five primitives. Recognizing which primitive applies is 80% of solving the problem.

| Primitive | Description | Problems in this set |
|---|---|---|
| Linear scan with counter | Track a numeric property (depth, parity, frequency) as you walk the string | Remove Outermost Parentheses, Largest Odd Number |
| Two-pointer / reversal | Start at both ends and converge, or reverse segments in place | Valid Palindrome, Reverse Words |
| Character frequency table | Count occurrences, compare two tables for equality | Valid Anagram |
| Character-to-character bijection | Maintain a two-way mapping between characters of two strings | Isomorphic Strings |
| String doubling / substring search | Concatenate string with itself; search for target as substring | Check Rotation |

---

## 2. C++ String Essentials

### Construction and Access

```cpp
string s = "hello";
string s2(5, 'a');          // "aaaaa"
string s3 = s.substr(1, 3); // "ell" (start_index, length)

s[i];        // unchecked — undefined behaviour on out-of-bounds
s.at(i);     // checked — throws std::out_of_range
s.size();    // or s.length() — identical, O(1)
s.empty();
```

### Mutation

```cpp
s.push_back('x');
s += "world";              // concatenate — use += not +
s.erase(2, 3);             // erase 3 chars starting at index 2
s.insert(2, "XY");         // insert before index 2
reverse(s.begin(), s.end());
sort(s.begin(), s.end());
```

### Search and Conversion

```cpp
s.find("ll");              // returns index, or string::npos if not found
s.find_first_of("aeiou");  // position of first vowel
stoi("42");                // string to int
stoll("12345678901");      // string to long long
to_string(42);             // int to string

// Split by whitespace (no built-in; use stringstream)
stringstream ss("hello world foo");
string token;
while (ss >> token) { /* token = "hello", then "world", then "foo" */ }
```

### Character Classification (`#include <cctype>`)

```cpp
isdigit(c); isalpha(c); isalnum(c); islower(c); isupper(c);
tolower(c); toupper(c);
```

### Cost Model — Memorize These

| Operation | Time | Note |
|---|---|---|
| `s[i]` | O(1) | Direct indexing |
| `s.size()` | O(1) | Cached internally |
| `s += c` (char) | Amortised O(1) | Like `vector::push_back` |
| `s += t` (string) | O(\|t\|) | Copies t's characters |
| `s.substr(i, len)` | O(len) | Always allocates a new string |
| `s.find(t)` | O(n·m) naive | O(n) with KMP (not built-in) |
| `reverse(...)` | O(n) | In-place |
| `sort(...)` | O(n log n) | |

**Critical:** `result += c` inside a loop is O(n) amortised total. `result = result + c` inside a loop creates a new string at each step — O(n²) total. **Always use `+=`.**

---

## 3. ASCII and Character Arithmetic

```cpp
// Digit to integer
char ch = '7';
int x = ch - '0';    // x = 7   (ASCII: '7'=55, '0'=48, 55-48=7)

// Integer to digit character
int x = 5;
char ch = x + '0';   // ch = '5'

// Lowercase letter to 0-indexed position
int idx = ch - 'a';  // 'a'→0, 'b'→1, ..., 'z'→25

// Case conversion
char upper = 'A' + (ch - 'a');   // lowercase → uppercase
char lower = 'a' + (ch - 'A');   // uppercase → lowercase
// Or simply: tolower(ch), toupper(ch)
```

**Frequency array pattern:**

```cpp
int freq[26] = {};
for (char c : s) freq[c - 'a']++;   // count occurrences
```

**When to use which storage for character mapping:**

| Structure | Use when |
|---|---|
| `int freq[26]` | Lowercase English only; fastest |
| `int freq[256]` | Any ASCII; use `(unsigned char)c` as index |
| `unordered_map<char,int>` | Unicode, arbitrary charset, or when key is not a char |

**Important:** When using `char` as an array index, always cast to `unsigned char`. On systems where `char` is signed (most x86), characters 128–255 produce negative indices — undefined behaviour.

```cpp
unsigned char idx = (unsigned char)c;
freq[idx]++;    // safe for all 256 ASCII values
```

---

## 4. Problem: Remove Outermost Parentheses (LC 1021)

### Problem Statement

A valid parentheses string is **primitive** if it is non-empty and cannot be split into two non-empty valid parentheses strings. Given a valid parentheses string `s`, find its primitive decomposition `s = P1 + P2 + ... + Pk` and return `s` with the outermost parentheses of every primitive removed.

```
"(()())(())"    →  "()()()"    primitives: "(()())" and "(())"
"(()())(())(()(()))" → "()()()()(())"
"()()"          →  ""           each "()" reduces to ""
```

### What "Primitive" Means

A primitive string is one where the balance of `(` minus `)` never returns to 0 before the very last character. Equivalently, the first `(` and the last `)` are matched to each other, not to any inner character.

### Approach 1: Stack Decomposition

Identify primitive boundaries by when the stack becomes empty (balance returns to 0). Strip the first and last character of each primitive.

```cpp
string removeOuterParentheses_stack(string s) {
    string result;
    stack<char> st;
    int start = 0;

    for (int i = 0; i < (int)s.size(); i++) {
        if (s[i] == '(') st.push('(');
        else              st.pop();

        if (st.empty()) {
            result += s.substr(start + 1, i - start - 1);
            start = i + 1;
        }
    }
    return result;
}
// Time: O(n), Space: O(n)
```

### Approach 2: Depth Counter (Optimal)

Track a `depth` counter. The outermost `(` of a primitive is the one that takes depth from 0 to 1. The outermost `)` is the one that takes depth from 1 to 0. Every other character is "inner" and belongs in the result.

Rules:
- On `(`: increment depth **first**, add to result only if `depth > 1`.
- On `)`: add to result only if `depth > 1`, then decrement depth.

The ordering is critical: for `(`, depth=1 after increment means "this is the outermost opening" — skip it. For `)`, depth=1 before decrement means "this is the outermost closing" — skip it.

```cpp
string removeOuterParentheses(string s) {
    string result;
    result.reserve(s.size());
    int depth = 0;

    for (char c : s) {
        if (c == '(') {
            depth++;
            if (depth > 1) result += c;   // inner opening
        } else {
            if (depth > 1) result += c;   // inner closing
            depth--;
        }
    }
    return result;
}
// Time: O(n), Space: O(1) extra
```

### Dry Run

```
s = "(()())(())"

c='(' depth:0→1, depth>1? No. skip.
c='(' depth:1→2, depth>1? Yes. result="("
c=')' depth>1? Yes. result="()". depth:2→1.
c='(' depth:1→2, Yes. result="()("
c=')' depth>1? Yes. result="()()". depth:2→1.
c=')' depth>1? No. depth:1→0. skip.    [end of first primitive "(()())"]
c='(' depth:0→1, No. skip.
c='(' depth:1→2, Yes. result="()()("
c=')' Yes. result="()()()" depth:2→1.
c=')' No. depth:1→0. skip.

Answer: "()()()"  ✓
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `"()"` | `""` | Single primitive; both chars are outermost |
| `"()()"` | `""` | Two primitives, each reduces to empty |
| `"((()))"` | `"(())"` | One primitive; strip one level |

---

## 5. Problem: Reverse Words in a String (LC 151)

### Problem Statement

Reverse the order of words in string `s`. Remove leading/trailing spaces; collapse multiple internal spaces to single spaces. A word is a maximal sequence of non-space characters.

```
"the sky is blue"   →  "blue is sky the"
"  hello   world  " →  "world hello"
"a good   example"  →  "example good a"
```

### Approach 1: stringstream Split

```cpp
string reverseWords_stream(string s) {
    stringstream ss(s);
    vector<string> words;
    string word;
    while (ss >> word) words.push_back(word);

    string result;
    for (int i = (int)words.size() - 1; i >= 0; i--) {
        result += words[i];
        if (i > 0) result += ' ';
    }
    return result;
}
// Time: O(n), Space: O(n)
```

### Approach 2: Manual Word Extraction

Skip spaces, extract each word, store in vector, join in reverse.

```cpp
string reverseWords_manual(string s) {
    vector<string> words;
    int n = s.size(), i = 0;

    while (i < n) {
        while (i < n && s[i] == ' ') i++;   // skip spaces
        string word;
        while (i < n && s[i] != ' ') word += s[i++];   // collect word
        if (!word.empty()) words.push_back(word);
    }

    string ans;
    for (int k = (int)words.size() - 1; k >= 0; k--) {
        ans += words[k];
        if (k > 0) ans += ' ';
    }
    return ans;
}
// Time: O(n), Space: O(n)
```

### Approach 3: In-Place Reverse (O(1) Extra Space)

Reverse the entire string, then reverse each individual word. Compact spaces in the same pass.

```cpp
string reverseWords(string s) {
    reverse(s.begin(), s.end());   // reverse entire string
    int n = s.size();
    int i = 0, j = 0;

    while (j < n) {
        while (j < n && s[j] == ' ') j++;   // skip spaces
        if (j == n) break;
        if (i > 0) s[i++] = ' ';            // separator between words

        int wordStart = i;
        while (j < n && s[j] != ' ') s[i++] = s[j++];   // copy word
        reverse(s.begin() + wordStart, s.begin() + i);   // reverse the word
    }

    s.resize(i);
    return s;
}
// Time: O(n), Space: O(1) extra
```

### Dry Run (Approach 3)

```
s = "  hello   world  "

After reverse entire: "  dlrow   olleh  "

j skips spaces to j=2 ('d')
i=0, i>0? No. wordStart=0. Copy "dlrow": s="dlrow...", i=5, j=7.
Reverse s[0..4]: "world".

j=7 spaces, skip to j=10 ('o').
i=5, i>0? Yes, s[5]=' ', i=6. wordStart=6. Copy "olleh": s="world olleh", i=11, j=15.
Reverse s[6..10]: "hello".

j=15, 16 are spaces, skip to j=17 = n → break.
resize to 11: "world hello"

Answer: "world hello"  ✓
```

---

## 6. Problem: Valid Palindrome (LC 125)

### Problem Statement

A phrase is a palindrome if, after converting to lowercase and removing all non-alphanumeric characters, it reads the same forward and backward.

```
"A man, a plan, a canal: Panama"  →  true
"race a car"                       →  false
" "                                →  true   (empty after filtering)
```

### Approach: Two Pointers

```cpp
bool isPalindrome(string s) {
    int lo = 0, hi = (int)s.size() - 1;

    while (lo < hi) {
        while (lo < hi && !isalnum(s[lo])) lo++;   // skip non-alphanumeric
        while (lo < hi && !isalnum(s[hi])) hi--;

        if (tolower(s[lo]) != tolower(s[hi])) return false;
        lo++;
        hi--;
    }
    return true;
}
// Time: O(n), Space: O(1)
```

### Dry Run

```
s = "race a car"
      0123456789

lo=0 'r', hi=9 'r': match, lo=1, hi=8
lo=1 'a', hi=8 'a': match, lo=2, hi=7
lo=2 'c', hi=7 'c': match, lo=3, hi=6
lo=3 'e', hi=6 ' ': hi not alnum → hi=5
lo=3 'e', hi=5 'a': tolower 'e' != 'a' → return false

Answer: false  ✓
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `"a"` | `true` | Single character |
| `" "` | `true` | Empty after filtering |
| `"0P"` | `false` | '0' != 'p' after tolower |

---

## 7. Problem: Largest Odd Number in a String (LC 1903)

### Problem Statement

Given a numeric string `num` (no leading zeros), return the largest-valued odd integer that is a non-empty substring of `num`. If none exists, return `""`.

```
"52"    →  "5"       (rightmost odd digit at index 0)
"4206"  →  ""        (no odd digit)
"35427" →  "35427"   (last digit 7 is odd, entire string returned)
```

### Key Insight

A number is odd if and only if its **last digit** is odd. Among all substrings, we want the one with the largest value. Longer substrings representing larger numbers with the same leading digits are preferred — so we want the longest prefix that ends in an odd digit. Scanning from the right finds the last (rightmost) odd digit, which gives the longest valid prefix.

### Optimal Approach: Scan from Right

```cpp
string largestOddNumber(string num) {
    for (int i = (int)num.size() - 1; i >= 0; i--) {
        if ((num[i] - '0') % 2 == 1) {
            return num.substr(0, i + 1);
        }
    }
    return "";
}
// Time: O(n) worst case, Space: O(1) extra (output is O(n))
```

### Dry Run

```
num = "35420"

i=4: '0' → 0%2=0, even, skip
i=3: '2' → 0, skip
i=2: '4' → 0, skip
i=1: '5' → 1, odd → return substr(0,2) = "35"

Answer: "35"  ✓
```

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `"1"` | `"1"` | Single odd digit |
| `"2"` | `""` | Single even digit |
| `"4206"` | `""` | No odd digit anywhere |
| `"13579"` | `"13579"` | Last digit odd; return whole string |

---

## 8. Problem: Longest Common Prefix (LC 14)

### Problem Statement

Find the longest common prefix string among all strings in `strs`. Return `""` if none exists.

```
["flower","flow","flight"]    →  "fl"
["dog","racecar","car"]       →  ""
```

### Approach 1: Vertical Scan (Optimal)

Compare characters column by column. At column `j`, check if all strings agree. Return the prefix at the first mismatch or when any string is exhausted.

```cpp
string longestCommonPrefix(vector<string>& strs) {
    if (strs.empty()) return "";

    for (int j = 0; j < (int)strs[0].size(); j++) {
        char c = strs[0][j];
        for (int i = 1; i < (int)strs.size(); i++) {
            if (j >= (int)strs[i].size() || strs[i][j] != c) {
                return strs[0].substr(0, j);
            }
        }
    }
    return strs[0];
}
// Time: O(S) where S = total characters. Space: O(1)
```

### Dry Run

```
strs = ["flower", "flow", "flight"]

j=0: c='f'. strs[1][0]='f' ✓, strs[2][0]='f' ✓
j=1: c='l'. strs[1][1]='l' ✓, strs[2][1]='l' ✓
j=2: c='o'. strs[2][2]='i' ≠ 'o' → return substr(0,2) = "fl"

Answer: "fl"  ✓
```

### Approach 2: Sort + Compare Extremes

After sorting, the LCP of the array equals the LCP of the first and last strings. Any string between them lexicographically must share at least this prefix.

```cpp
string longestCommonPrefix_sort(vector<string>& strs) {
    sort(strs.begin(), strs.end());
    string& first = strs.front();
    string& last  = strs.back();

    int j = 0;
    while (j < (int)first.size() && j < (int)last.size() && first[j] == last[j])
        j++;
    return first.substr(0, j);
}
// Time: O(S log n), Space: O(1)
```

**Why this works:** For any string `s` in the sorted array, `first ≤ s ≤ last` lexicographically. If `first[j] == last[j] == c`, then `s[j] == c` for all `s` between them (by the definition of lexicographic ordering). So the LCP of first and last is the LCP of the entire array.

### Edge Cases

| Input | Output | Note |
|---|---|---|
| `["a"]` | `"a"` | Single string is its own LCP |
| `["", "b"]` | `""` | Empty string has no prefix |
| `["abc","abc"]` | `"abc"` | All identical |
| `["ab","a"]` | `"a"` | Shorter string exhausted at j=1 |

---

## 9. Problem: Isomorphic Strings (LC 205)

### Problem Statement

Two strings `s` and `t` of equal length are **isomorphic** if characters in `s` can be replaced to get `t` such that:
- All occurrences of a character are replaced by the same character.
- No two different characters in `s` map to the same character in `t`.
- A character may map to itself.

```
"egg", "add"   →  true   (e→a, g→d)
"foo", "bar"   →  false  (o maps to both a and r)
"paper","title" → true   (p→t, a→i, e→e, r→r)
"badc","baba"  →  false  (a→b and c→b violates bijectivity)
```

### Why Two Maps Are Required

A single `s→t` map catches: one character in `s` mapping to two different characters in `t`. But it misses: two different characters in `s` mapping to the same character in `t`, which violates bijectivity.

```
s="ab", t="aa"
s→t only: a→a (ok), b→a (no conflict in s→t). Incorrectly returns true.
t→s check: a→a (ok), then a→b conflicts with a→a. Correctly returns false.
```

### Approach 1: Two Hash Maps

```cpp
bool isIsomorphic(string s, string t) {
    unordered_map<char, char> sToT, tToS;

    for (int i = 0; i < (int)s.size(); i++) {
        char cs = s[i], ct = t[i];

        if (sToT.count(cs) && sToT[cs] != ct) return false;   // s→t conflict
        if (tToS.count(ct) && tToS[ct] != cs) return false;   // t→s conflict

        sToT[cs] = ct;
        tToS[ct] = cs;
    }
    return true;
}
// Time: O(n), Space: O(1) — maps hold at most 256 entries
```

### Approach 2: Two char Arrays (Optimal for ASCII)

Replace hash maps with 256-element arrays for smaller constants.

```cpp
bool isIsomorphic(string s, string t) {
    char sToT[256] = {};
    char tToS[256] = {};

    for (int i = 0; i < (int)s.size(); i++) {
        unsigned char cs = s[i], ct = t[i];

        if (sToT[cs] && sToT[cs] != ct) return false;
        if (tToS[ct] && tToS[ct] != cs) return false;

        sToT[cs] = ct;
        tToS[ct] = cs;
    }
    return true;
}
// Time: O(n), Space: O(1)
```

**Note on `unsigned char`:** On systems where `char` is signed, values 128–255 produce negative array indices — undefined behaviour. Always cast to `unsigned char` when using char as an array index.

### Dry Run

```
s="foo", t="bar"

i=0: cs='f', ct='b'. sToT[f]=0→b. tToS[b]=0→f.
i=1: cs='o', ct='a'. sToT[o]=0→a. tToS[a]=0→o.
i=2: cs='o', ct='r'. sToT[o]=a ≠ r → return false.

Answer: false  ✓
```

### Edge Cases

| s | t | Output | Note |
|---|---|---|---|
| `"a"` | `"a"` | `true` | Identity mapping |
| `"ab"` | `"aa"` | `false` | Two s-chars map to same t-char |
| `"aa"` | `"ab"` | `false` | One s-char maps to two t-chars |

---

## 10. Problem: Check Whether One String is a Rotation (LC 796)

### Problem Statement

Return `true` if `s` can become `goal` after some number of left-shifts (move leftmost char to the right end).

```
s="abcde", goal="cdeab" → true   (shift left twice)
s="abcde", goal="abced" → false
s="aa",    goal="aa"    → true
```

### The Doubling Trick — Correctness Proof

**Claim:** `goal` is a rotation of `s` ⟺ `|s| == |goal|` AND `goal` is a substring of `s + s`.

**Proof (→):** A rotation by `k` positions gives `goal = s[k..n-1] + s[0..k-1]`. In `s + s = s[0..n-1] + s[0..n-1]`, the substring starting at position `k` of length `n` is exactly `s[k..n-1] + s[0..k-1] = goal`.

**Proof (←):** Any length-n substring of `s + s` starting at position `k` (0 ≤ k < n) equals the rotation of `s` by k positions. If `goal` is such a substring, it is a rotation of `s`.

### Approach 1: Brute Force — Try All Rotations O(n²)

```cpp
bool rotateString_brute(string s, string goal) {
    if (s.size() != goal.size()) return false;
    int n = s.size();
    for (int k = 0; k < n; k++) {
        bool match = true;
        for (int i = 0; i < n; i++)
            if (s[(i + k) % n] != goal[i]) { match = false; break; }
        if (match) return true;
    }
    return false;
}
```

### Approach 2: Doubling + `find` — O(n²) worst case, O(n) typical

```cpp
bool rotateString(string s, string goal) {
    if (s.size() != goal.size()) return false;
    return (s + s).find(goal) != string::npos;
}
// Time: O(n^2) worst case (naive find), O(n) typical
// Space: O(n) for s+s
```

At the problem's constraint of n ≤ 100, this is always accepted in interviews.

### Approach 3: Doubling + KMP — True O(n)

```cpp
bool rotateString_kmp(string s, string goal) {
    if (s.size() != goal.size()) return false;
    string text = s + s;
    string pat = goal;
    int n = text.size(), m = pat.size();

    // Build KMP failure function for pattern
    vector<int> fail(m, 0);
    for (int i = 1; i < m; i++) {
        int j = fail[i - 1];
        while (j > 0 && pat[i] != pat[j]) j = fail[j - 1];
        if (pat[i] == pat[j]) j++;
        fail[i] = j;
    }

    // KMP search in text
    int j = 0;
    for (int i = 0; i < n; i++) {
        while (j > 0 && text[i] != pat[j]) j = fail[j - 1];
        if (text[i] == pat[j]) j++;
        if (j == m) return true;
    }
    return false;
}
// Time: O(n), Space: O(n)
```

### Dry Run

```
s="abcde", goal="cdeab"

doubled = "abcdeabcde"
find("cdeab") → found at index 2: "abcde[cdeab]"

Return true  ✓
```

### Edge Cases

| s | goal | Output | Note |
|---|---|---|---|
| `""` | `""` | `true` | Both empty |
| `"a"` | `"a"` | `true` | Trivial rotation |
| `"ab"` | `"ba"` | `true` | One shift |
| `"abc"` | `"abcd"` | `false` | Different lengths: return false immediately |
| `"aa"` | `"aa"` | `true` | Identical strings |

---

## 11. Problem: Valid Anagram (LC 242)

### Problem Statement

Return `true` if `t` is an anagram of `s` — same characters with same frequencies, possibly in different order.

```
"anagram", "nagaram"  →  true
"rat", "car"          →  false
```

**Definition:** Two strings are anagrams iff they are permutations of each other — same multiset of characters.

### Approach 1: Sort Both — O(n log n)

```cpp
bool isAnagram_sort(string s, string t) {
    if (s.size() != t.size()) return false;
    sort(s.begin(), s.end());
    sort(t.begin(), t.end());
    return s == t;
}
```

### Approach 2: Frequency Array (Optimal) — O(n)

Increment count for each character in `s`, decrement for each in `t`. If any count is non-zero at the end, the strings differ.

```cpp
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;

    int freq[26] = {};
    for (char c : s) freq[c - 'a']++;
    for (char c : t) freq[c - 'a']--;

    for (int x : freq)
        if (x != 0) return false;

    return true;
}
// Time: O(n), Space: O(1) — array of fixed size 26
```

**Single-pass variant:**

```cpp
bool isAnagram_v2(string s, string t) {
    if (s.size() != t.size()) return false;
    int freq[26] = {};
    for (int i = 0; i < (int)s.size(); i++) {
        freq[s[i] - 'a']++;
        freq[t[i] - 'a']--;
    }
    for (int x : freq) if (x != 0) return false;
    return true;
}
```

### Dry Run

```
s="anagram", t="nagaram"

After s: freq[a]=3, freq[n]=1, freq[g]=1, freq[r]=1, freq[m]=1
After t (decrement): n,a,g,a,r,a,m → all counts return to 0

All freq = 0 → return true  ✓
```

### Follow-Up: Unicode Characters

For arbitrary character sets, use `unordered_map`:

```cpp
bool isAnagram_unicode(string s, string t) {
    if (s.size() != t.size()) return false;
    unordered_map<char, int> freq;
    for (char c : s) freq[c]++;
    for (char c : t) {
        freq[c]--;
        if (freq[c] < 0) return false;
    }
    return true;
}
```

### Edge Cases

| s | t | Output | Note |
|---|---|---|---|
| `"a"` | `"a"` | `true` | Trivial |
| `"ab"` | `"a"` | `false` | Different lengths: early exit |
| `"aab"` | `"bba"` | `false` | a:2,b:1 vs b:2,a:1 |
| `""` | `""` | `true` | Both empty |

---

## 12. Related Concepts and Extensions

### From Anagram: Group Anagrams (LC 49)

Use the sorted form of a word as its canonical key. All words that are anagrams share the same sorted key.

```cpp
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    unordered_map<string, vector<string>> groups;
    for (string& s : strs) {
        string key = s;
        sort(key.begin(), key.end());
        groups[key].push_back(s);
    }
    vector<vector<string>> result;
    for (auto& [key, group] : groups) result.push_back(group);
    return result;
}
```

### From Isomorphic: Word Pattern (LC 290)

Match a pattern string (e.g., `"abba"`) against a sequence of words. Identical bijection logic: word ↔ pattern character, two maps required.

### From Rotation: Repeated String Match (LC 686)

Find the minimum number of times `a` must be repeated so that `b` is a substring. Repeat `a` until length ≥ `|b|`, then check if `b` is a substring (once or twice).

### From LCP: Trie for LCP Queries

For many LCP queries across a large set of strings, build a Trie. LCP of any two strings is found at their divergence point.

### From Remove Outermost Parentheses: Valid Parentheses (LC 20)

Same depth-counter technique. Multiple bracket types use a stack; single-type uses an integer counter.

### Key Harder Problems

| Problem | Foundation from this set |
|---|---|
| LC 438 — Find All Anagrams in a String | Frequency array + sliding window |
| LC 49 — Group Anagrams | Sorted string as map key |
| LC 290 — Word Pattern | Two bijection maps (isomorphic) |
| LC 686 — Repeated String Match | Rotation / doubling trick |
| LC 680 — Valid Palindrome II | Two-pointer with one skip allowed |
| LC 28 — Find Index of First Occurrence | KMP (foundation of rotation check) |

---

## 13. Interview Pattern Map

| Problem signal | Technique | Key structure |
|---|---|---|
| Remove/keep chars by nesting depth | Depth counter | Integer |
| Reverse word order | Split + reverse, or in-place reverse-all + reverse-words | Pointers / stringstream |
| Palindrome check | Two pointers from ends, skip non-alnum | Two pointers |
| Odd/even digit property of substrings | Scan from the right | Single pointer |
| Longest common prefix of array | Vertical scan or sort + compare extremes | None |
| Character-to-character structure mapping | Two bijection maps | Two arrays or hash maps |
| Rotation check | Doubling + substring search | Concatenated string |
| Anagram / frequency equality | Frequency table | int[26] or hash map |
| Sliding window anagram count | Frequency table + two pointers | int[26] + window |
| Pattern matching (word ↔ char) | Two bijection maps | Same as isomorphic |

---

## 14. Common Mistakes and Pitfalls

### Mistake 1: Single map for Isomorphic Strings

```cpp
// WRONG: only s→t map; misses "two s-chars mapping to same t-char"
if (sToT.count(cs) && sToT[cs] != ct) return false;
sToT[cs] = ct;
// For s="ab", t="aa": a→a (ok), b→a (no conflict in s→t). Returns true. Wrong.

// CORRECT: check both s→t and t→s maps.
```

### Mistake 2: Not checking size equality before rotation/anagram

```cpp
// WRONG for rotation: (s+s).find(goal) can match even if lengths differ
bool rotate_wrong(string s, string g) { return (s+s).find(g) != string::npos; }

// CORRECT: size check is the very first line
if (s.size() != goal.size()) return false;
```

### Mistake 3: `result = result + c` in a loop (O(n²))

```cpp
// WRONG: creates new string at each step — O(n^2) total
string result = "";
for (char c : s) result = result + c;

// CORRECT: use += for amortized O(1) per append
string result;
result.reserve(s.size());
for (char c : s) result += c;
```

### Mistake 4: Wrong depth ordering in Remove Outermost Parentheses

```cpp
// WRONG: for ')', decrement then check — keeps the outermost ')'
depth--;
if (depth > 0) result += ')';

// CORRECT: check then decrement
if (depth > 1) result += ')';   // depth=1 means "this is outermost", skip
depth--;
```

For `(`: increment first, then check `depth > 1`.
For `)`: check `depth > 1` first, then decrement.

### Mistake 5: Out-of-bounds in LCP vertical scan

```cpp
// WRONG: accesses strs[i][j] without checking string length
if (strs[i][j] != c) return ans;   // UB if j >= strs[i].size()

// CORRECT: check length first
if (j >= (int)strs[i].size() || strs[i][j] != c) return strs[0].substr(0, j);
```

### Mistake 6: Not casting to `unsigned char` for array-indexed maps

```cpp
// WRONG on signed-char platforms: negative index for chars 128-255
freq[c - 'a']++;        // ok for lowercase only
sToT[s[i]] = t[i];      // WRONG if s[i] > 127: sToT[(negative_index)] = ...

// CORRECT
unsigned char cs = s[i];
sToT[cs] = ct;
```

### Mistake 7: Forgetting to handle leading/trailing/extra spaces in Reverse Words

The manual approach must skip spaces before, after, and between words. The `stringstream >> word` approach handles this automatically.

---

## 15. Complexity Cheat Sheet

| Problem | Approach | Time | Space |
|---|---|---|---|
| Remove Outermost Parens | Stack decomposition | O(n) | O(n) |
| Remove Outermost Parens | Depth counter | O(n) | O(1) extra |
| Reverse Words | stringstream split | O(n) | O(n) |
| Reverse Words | In-place reverse | O(n) | O(1) extra |
| Valid Palindrome | Two pointers | O(n) | O(1) |
| Largest Odd Number | Scan from right | O(n) | O(1) extra |
| Longest Common Prefix | Vertical scan | O(S) | O(1) |
| Longest Common Prefix | Sort + compare | O(S log n) | O(1) |
| Isomorphic Strings | Two hash maps | O(n) | O(1) |
| Isomorphic Strings | Two char arrays | O(n) | O(1) |
| Check Rotation | Doubling + find | O(n²) worst | O(n) |
| Check Rotation | Doubling + KMP | O(n) | O(n) |
| Valid Anagram | Sort | O(n log n) | O(1) |
| Valid Anagram | Frequency array | O(n) | O(1) |

---

## 16. Quick Revision Cheat Sheet

Self-contained for same-day interview revision.

---

### Remove Outermost Parentheses

```cpp
int depth = 0;
for (char c : s) {
    if (c == '(') { depth++; if (depth > 1) result += c; }
    else          { if (depth > 1) result += c; depth--; }
}
// Outermost chars: those that transition depth 0↔1.
// On '(': increment first, keep if depth > 1.
// On ')': keep if depth > 1, then decrement.
```

---

### Reverse Words

```cpp
// Approach 1: stringstream (handles all space cases automatically)
stringstream ss(s); string w;
while (ss >> w) words.push_back(w);
// then join words in reverse order

// Approach 2: In-place (O(1) space)
reverse(s.begin(), s.end());       // reverse whole string
// then reverse each word and compact spaces
```

---

### Valid Palindrome

```cpp
int lo=0, hi=(int)s.size()-1;
while (lo < hi) {
    while (lo<hi && !isalnum(s[lo])) lo++;
    while (lo<hi && !isalnum(s[hi])) hi--;
    if (tolower(s[lo]) != tolower(s[hi])) return false;
    lo++; hi--;
}
return true;
```

---

### Largest Odd Number

```cpp
for (int i=(int)num.size()-1; i>=0; i--)
    if ((num[i]-'0') % 2 == 1) return num.substr(0, i+1);
return "";
// A number is odd iff its last digit is odd.
// Scan from right: first odd digit gives the longest valid prefix.
```

---

### Longest Common Prefix

```cpp
for (int j=0; j<(int)strs[0].size(); j++) {
    char c = strs[0][j];
    for (int i=1; i<(int)strs.size(); i++)
        if (j >= (int)strs[i].size() || strs[i][j] != c)
            return strs[0].substr(0, j);
}
return strs[0];
// Vertical scan. Check j >= strs[i].size() BEFORE accessing strs[i][j].
```

---

### Isomorphic Strings

```cpp
char sToT[256]={}, tToS[256]={};
for (int i=0; i<(int)s.size(); i++) {
    unsigned char cs=s[i], ct=t[i];
    if (sToT[cs] && sToT[cs]!=ct) return false;
    if (tToS[ct] && tToS[ct]!=cs) return false;
    sToT[cs]=ct; tToS[ct]=cs;
}
return true;
// Two maps = bijection. sToT catches one-to-many. tToS catches many-to-one.
// Cast to unsigned char to avoid negative indices on signed-char platforms.
```

---

### Check Rotation

```cpp
if (s.size() != goal.size()) return false;
return (s + s).find(goal) != string::npos;
// Every rotation of s appears as a substring of s+s.
// Size check is mandatory — find can match even with different lengths.
```

---

### Valid Anagram

```cpp
if (s.size() != t.size()) return false;
int freq[26] = {};
for (char c : s) freq[c-'a']++;
for (char c : t) freq[c-'a']--;
for (int x : freq) if (x != 0) return false;
return true;
// O(n) time, O(1) space. For unicode: use unordered_map<char,int>.
```

---

### ASCII Quick Reference

```
'0'=48  '9'=57   digit→int: c-'0'    int→digit: x+'0'
'A'=65  'Z'=90   lowercase index: c-'a'  (range 0-25)
'a'=97  'z'=122

isalnum(c): letter or digit
isalpha(c): letter
isdigit(c): '0'-'9'
tolower(c), toupper(c)
```

---

### Critical Implementation Rules

```
1. Use += not + in loops for string concatenation.
2. Always size-check before anagram or rotation checks.
3. Isomorphic strings require TWO maps (s→t and t→s).
4. Depth counter for parens: increment/decrement ORDER matters.
5. LCP vertical scan: check j >= strs[i].size() before strs[i][j].
6. Use (unsigned char)c as array index for non-lowercase characters.
```
