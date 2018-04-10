---
layout: post
title: "Easy Peasy"
date: 2018-04-09 00:00:00 -0600
category: Tech
tags: String
cover: /assets/easy_peasy_cover.png
math: true
---

### Introduction

I have always had hard times on problems involving wrapped around strings or repeated strings. This was meant to be an easy peasy problem, but I suffered hard on it.

Or there is something seriously wrong with LeetCode's difficulty labeling.

Quoting from [LeetCode 686](https://leetcode.com/problems/repeated-string-match/description/),

> Given two strings A and B, find the minimum number of times A has to be repeated such that B is a substring of it. If no such solution, return -1.

> For example, with A = "abcd" and B = "cdabcdab".

> Return 3, because by repeating A three times (“abcdabcdabcd”), B is a substring of it; and B is not a substring of A repeated two times ("abcdabcd").

> Note:
> The length of A and B will be between 1 and 10000.

#### Approach #1 Naive and Natural

From simple observations, we know that `B` can only be a substring of `ARepeat` when size of `B` is smaller than `ARepeat`. But when repeats of `A` is at the minimum size larger than `B`, can we guarantee `B` would appear as a substring? Consider the following example:

A: abcd (size 4)

B: dabcdabcda (size 11)

As we can see, when repeated 3 times, `A` would form "abcdabcdabcd", but `B` is still not a substring of it, because `B` has `A` split up at both ends of the string. Let the minimum number of repeats be `q`, we have to search the repeat of `q` and `q + 1` to guarantee the finding of substring `B`, should it exist.

```c++
class Solution {
public:
        int repeatedStringMatch(string A, string B) {
        string ARepeat = A;
        int rep = 1;
        while (ARepeat.size() < B.size()) {
            ARepeat += A;
            rep++;
        }
        if (ARepeat.find(B) != -1)
            return rep;
        ARepeat += A;
        rep++;
        if (ARepeat.find(B) != -1)
            return rep;
        return -1;
    }
};
```

**Complexity Analysis**

* Time complexity: $$O(m(m + n))$$.

The string `ARepeat` is of size $$m + n$$, and we run the built-in naive pattern matching (https://stackoverflow.com/questions/8869605/c-stringfind-complexity) to search for `B` (size $$m$$) on `ARepeat`, which takes the product of the length of both strings.

Thus the time complexity is: $$O(m(m + n))$$.

* Space complexity: $$O(m + n)$$.

#### Approach #2 Naive KMP

This is basically the same as above naive solution, except we implement KMP instead of C++'s built-in naive pattern matching. The KMP could be further optimized to return on first match, but that's trivial.

```c++
class Solution {
public:
    int repeatedStringMatch(string A, string B) {
        string ARepeat = A;
        int rep = 1;
        while (ARepeat.size() < B.size()) {
            ARepeat += A;
            rep++;
        }
        if (kmp(ARepeat, B).size())
            return rep;
        ARepeat += A;
        rep++;
        if (kmp(ARepeat, B).size())
            return rep;
        return -1;
    }
    
    vector<int> computeLPS(string needle) {
        int n = needle.size();
        if (!n)
            return {};
        vector<int> lps(n, 0);
        int i = 1, len = 0;
        while (i < n) {
            if (needle[i] == needle[len])
                lps[i++] = ++len;
            else if (!len)
                i++;
            else
                len = lps[len - 1];
        }
        return lps;
    }
    
    vector<int> kmp(string haystack, string needle) {
        int n = haystack.size(), m = needle.size();
        vector<int> lps = computeLPS(needle);
        int i = 0, j = 0;
        vector<int> res;
        while (i < n) {
            if (haystack[i] == needle[j]) {
                i++;
                j++;
            }
            else if (j)
                j = lps[j - 1];
            else
                i++;
            if (j >= m) {
                res.push_back(i - m);
                j = lps[m - 1];
            }
        }
        return res;
    }
};
```

**Complexity Analysis**

* Time complexity: $$O(m + n)$$.

The string `ARepeat` is of size $$m + n$$, and we run the KMP algorithm to search for `B` (size $$m$$) on `ARepeat`, which takes the sum of the length of both strings.

Thus the time complexity is: $$O(m + n)$$.

* Space complexity: $$O(m + n)$$.

#### Approach #3 Wrap-Around KMP

This is a genius variant of the KMP solution. I found it from the [discuss section](https://leetcode.com/problems/repeated-string-match/discuss/112570/C++-KMP-algo-o(m-+n)-detailed). 

Basically it creates the LPS array of `B`, and runs KMP on `A` with wrapping around, saving the building of `ARepeat` string. While still the same time complexity and slightly better space complexity, this solution is far more concise and elegant than my own.

Below is my own implementation of the wrap-around KMP solution, with some minor corrections made to the original one.

```c++
class Solution {
public:
    int repeatedStringMatch(string a, string b) {
        vector<int> lps(b.size());
        for (int i = 1, j = 0; i < b.size();) {
            if(b[j] == b[i])
                lps[i++] = ++j;
            else if (j == 0)
                i++;
            else
                j = lps[j -1];
        }
        for (auto i = 0, j = 0; i < a.size(); i++, j = lps[j-1]) {
            while (j < b.size() && a[(i + j) % a.size()] == b[j])
                ++j;
            if (j == b.size())
                return ceil((float)(i + j) / a.size());
        }
        return -1;
    }
};
```

**Complexity Analysis**

* Time complexity: $$O(m + n)$$.

The string `A` is of size $$n$$, and we run the KMP algorithm to search for `B` (size $$m$$) on `A` (wrapping around), which takes the sum of the length of both strings.

Thus the time complexity is: $$O(m + n)$$.

* Space complexity: $$O(m)$$.

Only $$O(m)$$ for LPS array.

#### Approach #4 Rabin-Karp

This is one of them mathematical genius bullshit solutions, and I currently don't have time or interest to invest in it. Only writing this section to take note for future coming back.

It's longer, takes more time to implement, and the only advantage it has over the KMP solution is that it takes $$O(1)$$ space. Still, it is the most [superior solution](https://leetcode.com/problems/repeated-string-match/solution/) by the looks of it.

### End

There are a lot of "easy" problems like this out there. They have really easy to implement naive solutions, but the optimal solutions behind those are not easy peasy at all. The OJ's are just loosing up the time limit of those and calling them easy.

Signing off.