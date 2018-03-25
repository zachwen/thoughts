---
layout: post
title: "Freedom Trail"
date: 2018-03-22 00:00:00 -0600
category: Tech
tags: DP
cover: /assets/freedom_trail_cover.png
math: true
---

### Introduction

This article is about the research on the algorithm problem "Freedom Trail".

Quoting from [LeetCode 514](https://leetcode.com/problems/freedom-trail/description/), 

> In the video game Fallout 4, the quest "Road to Freedom" requires players to reach a metal dial called the "Freedom Trail Ring", and use the dial to spell a specific keyword in order to open the door.

> Given a string ring, which represents the code engraved on the outer ring and another string key, which represents the keyword needs to be spelled. You need to find the minimum number of steps in order to spell all the characters in the keyword.

> Initially, the first character of the ring is aligned at 12:00 direction. You need to spell all the characters in the string key one by one by rotating the ring clockwise or anticlockwise to make each character of the string key aligned at 12:00 direction and then by pressing the center button. 
> At the stage of rotating the ring to spell the key character key[i]:
> 1. You can rotate the ring clockwise or anticlockwise one place, which counts as 1 step. The final purpose of the rotation is to align one of the string ring's characters at the 12:00 direction, where this character must equal to the character key[i].
> 2. If the character key[i] has been aligned at the 12:00 direction, you need to press the center button to spell, which also counts as 1 step. After the pressing, you could begin to spell the next character in the key (next stage), otherwise, you've finished all the spelling.

> Example:

> Input: ring = "godding", key = "gd"
> Output: 4

> Explanation:
> For the first key character 'g', since it is already in place, we just need 1 step to spell this character. 
> For the second key character 'd', we need to rotate the ring "godding" anticlockwise by two steps to make it become "ddinggo".
> Also, we need 1 more step for spelling.
> So the final output is 4.

This problem seems intimidating at first, but it's actually a textbook dynamic programming problem. Below I give my process of thoughts on it. This article is also published on [LeetCode discuss](https://discuss.leetcode.com/topic/121027/solution-by-zachwen).

#### Approach #1 Brute Force

**Intuition**

The very natural brute force approach is to simply try all possible paths to finish the desired key, and return the length of the shortest one.

**Algorithm**

Let the length of `ring` be `r`, and that of `key` be `k`. Since the string `ring` is circular, the distance between any two indices is bound by floor of $$\frac{r}{2}$$. The calculation of this "circular distance" is encapsuled in the helper function `circularDistance`.

In this naive (in terms of both space and time complexity) solution, the vector `paths` holds a list of pairs, for which the `first` of the pair is the number of steps to reach current state, and the `second` of the pair is the current index in the `ring`. We process each character of `key` one by one, and for each one of them, we find every possible move (duplicate of this character) in the ring, and expand our vector of paths by calculating the current "circular distance", which we add to each one of the existing paths of the previous step, to find our paths for this step. Finally, we iterate through all paths and find the one with minimum steps.

This solution is terribly naive and should not be taken seriously. It is only for illustrating the natural thought after first encountering the problem.

**C++**

```c++
class Solution {
public:
    int findRotateSteps(string ring, string key) {
        int r = ring.size(), k = key.size();
        vector<pair<int, int>> paths(1, make_pair(0, 0));
        for (int i = 0; i < k; ++i) {
            vector<pair<int, int>> currentPaths;
            for (int j = 0; j < r; ++j)
                if (key[i] == ring[j])
                    for (auto path : paths)
                        currentPaths.push_back(make_pair(path.first + circularDistance(r, path.second, j), j));
            paths = currentPaths;
        }
        int minSteps = INT_MAX;
        for (auto path : paths)
            if (path.first < minSteps)
                minSteps = path.first;
        return minSteps + k;
    }
    int circularDistance(int n, int i, int j) {
        int d = abs(j - i);
        return d > n / 2 ? n - d : d;
    }
};
```

**Complexity Analysis**

* Time complexity: $$O(kr^k)$$.

Every character of `key` (length $$k$$) is process one by one, and for each of them, we iterate through `ring` (length $$r$$) and find all possible moves for current step, and for each step, we iterate through the previous steps to find all the current steps. The number of steps is bound by $$O(r^k)$$.

Thus the time complexity is: $$O(kr^k)$$.

* Space complexity: $$O(r^k)$$.

Here `paths` is bounded by $$r^k$$.

---
#### Approach #2 DFS Recursion with Memoization

**Intuition**

Use depth first search to explore the solution space of possible paths: choose the path with optimal step number at every step, instead of choosing the optimal path out of all steps at last.

**Algorithm**

The definition of `r` and `k` and the helper function `circularDistance` are the same as in last approach.

We employ recursion to process each character of `key`, and choose the current move out of moves available at this step top-down. The variable `ringIdx` represents the index in `ring` of current step, and `keyIdx` is the index in `key` that is being processed at this step. `c` is a minor optimization structure that keeps track of the indices of all duplicates of a character in `ring`.

**C++**

```c++
class Solution {
private:
    unordered_map<char, vector<int>> c;
public:
    int findRotateSteps(string ring, string key) {
        for (int i = 0; i < ring.size(); ++i)
            c[ring[i]].push_back(i);
        return rotate(ring, key, 0, 0);
    }

    int rotate(string ring, string key, int ringIdx, int keyIdx) {
        if (keyIdx >= key.size())
            return 0;
        int distance = INT_MAX;
        for (int possibleIdx : c[key[keyIdx]]) {
            distance = min(distance, 1 + circularDistance(ring.size(), ringIdx, possibleIdx) + rotate(ring, key, possibleIdx, keyIdx + 1));
        }
        return distance;
    }

    int circularDistance(int n, int i, int j) {
        int d = abs(j - i);
        return d > n / 2 ? n - d : d;
    }
};
```


Note that the above solution will still result in "Time Limit Exceeded" because it has overlapping sub-problems. A memoization could be easily added to reduce the time complexity: `ringIdx` and `keyIdx` combined could encode a unique state of the exploration in the solution space, so they can be used as the key value of memoization structure, which is simply a 2-D matrix of `r` by `k`.

```c++
class Solution {
private:
    unordered_map<char, vector<int>> c;
    vector<vector<int>> mem;
public:
    int findRotateSteps(string ring, string key) {
        for (int i = 0; i < ring.size(); ++i)
            c[ring[i]].push_back(i);
        mem = vector<vector<int>>(ring.size(), vector<int>(key.size(), 0));
        return rotate(ring, key, 0, 0);
    }

    int rotate(string ring, string key, int ringIdx, int keyIdx) {
        if (keyIdx >= key.size())
            return 0;
        if (mem[ringIdx][keyIdx])
            return mem[ringIdx][keyIdx];
        int distance = INT_MAX;
        for (int possibleIdx : c[key[keyIdx]]) {
            distance = min(distance, 1 + circularDistance(ring.size(), ringIdx, possibleIdx) + rotate(ring, key, possibleIdx, keyIdx + 1));
        }
        mem[ringIdx][keyIdx] = distance;
        return distance;
    }

    int circularDistance(int n, int i, int j) {
        int d = abs(j - i);
        return d > n / 2 ? n - d : d;
    }
};
```

**Complexity Analysis**

* Time complexity: $$O(r^k)$$, without memoization; $$O(r^2k)$$, with memoization.

From the recurrence we know the following equations to be true, where $$k$$ stands for length of `key`:

$$T(k) = rT(k - 1)$$

$$rT(k - 1) = r^2T(k - 2)$$

$$...$$

$$r^{k - 1}T(1) = r^kT(0)$$

Adding them all yields: $$T(k) = r^kT(0)$$.

Hence the time complexity is: $$O(r^k)$$.

See next approach for time complexity analysis for the memoized version.

* Space complexity: $$O(1)$$, without memoization; $$O(rk)$$, with memoization.

---
#### Approach #3 Dynamic Programming Bottom-Up

**Intuition**

Process `key` one by one, bottom-up. At each step, calculate and store the current optimal paths for re-use in next step.

**Algorithm**

The idea is basically the same as last approach with memoization, but implemented using the bottom-up approach, saving much of the recursion overhead. 


The algorithm logic should be self-evident from reading the code and variable names. A catch is that the memoization structure must be initialized with `INT_MAX`, except `dp[0]`, which should be filled with zeros. Here `prevStep` and `currStep` are indices in `ring`, and `dp[i][currStep]` represents the optimal step number after processing `key[i - 1]`, with the current move stepped on `ring[currStep]`. At last, we find the optimal path in `dp[k]` with STL's quick select function.

**C++**

```c++
class Solution {
private:
    unordered_map<char, vector<int>> c;
public:
    int findRotateSteps(string ring, string key) {
        int r = ring.size(), k = key.size();
        for (int i = 0; i < r; ++i)
            c[ring[i]].push_back(i);
        vector<vector<int>> dp(k + 1, vector<int>(r, INT_MAX));
        vector<int> prevSteps(1, 0);
        fill(dp[0].begin(),dp[0].end(),0);
        for (int i = 1; i <= k; ++i) {
            for (int prevStep : prevSteps)
                for (int currStep : c[key[i - 1]])
                    dp[i][currStep] = min(dp[i][currStep], circularDistance(r, prevStep, currStep) + dp[i - 1][prevStep]);
            prevSteps = c[key[i - 1]];
        }
        nth_element(dp[k].begin(), dp[k].begin(), dp[k].end());
        return dp[k][0] + k;
    }
    
    int circularDistance(int n, int i, int j) {
        int d = abs(j - i);
        return d > n / 2 ? n - d : d;
    }
};
```

**Complexity Analysis**

* Time complexity: $$O(r^2k)$$.

The algorithm only consists of a three nested loops. The outer loop has `k` iterations, and the middle and inner one have `prevSteps` and `currSteps` iterations which are both bounded by `r`. The last quick select is $$O(r)$$. Hence the time complexicity: $$O(r^2k)$$.

* Space complexity: $$O(rk)$$.

### End

Fallout 4 was a fun game, though compared to Fallout New Vegas it sucked ass. Signing off.