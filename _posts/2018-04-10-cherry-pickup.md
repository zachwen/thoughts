---
layout: post
title: "Cherry Pickup"
date: 2018-04-10 00:00:00 -0600
category: Tech
tags: DP
cover: /assets/cherry_pickup_cover.png
math: true
---

### Introduction

This article is about the some thoughts on the algorithm problem "Cherry Pickup".

Quoting from [LeetCode 741](https://leetcode.com/problems/cherry-pickup/description/),

> In a N x N grid representing a field of cherries, each cell is one of three possible integers.

> 0 means the cell is empty, so you can pass through;
> 1 means the cell contains a cherry, that you can pick up and pass through;
> -1 means the cell contains a thorn that blocks your way.
> Your task is to collect maximum number of cherries possible by following the rules below:

> Starting at the position (0, 0) and reaching (N-1, N-1) by moving right or down through valid path cells (cells with value 0 or 1);
> After reaching (N-1, N-1), returning to (0, 0) by moving left or up through valid path cells;
> When passing through a path cell containing a cherry, you pick it up and the cell becomes an empty cell (0);
> If there is no valid path between (0, 0) and (N-1, N-1), then no cherries can be collected.

I consider this problem the single hardest DP problem I have encountered, for its high level of abstraction, and that it is a 3-D DP problem. The process of thinking involved in this problem is so abstract that I cannot come up with even a sub-optimal solution that works correctly, other than naive backtracking: trying every route from start to finish, and back from finish to start. 

This article is not a complete explanation for the solution, and it contains only some complementary thoughts and explanations for the article posted [here](https://leetcode.com/problems/cherry-pickup/discuss/109903/Step-by-step-guidance-of-the-O(N3)-time-and-O(N2)-space-solution), an excellent solution by the famous @fun4LeetCode. While his article provides a truly genius solution, I only understood it after searching for the problem online and cross-referencing other articles.

#### DP Dimensions

While optimized to use only a 2-D matrix for memoization, this solution is actually 3-D DP.

The first dimension, which @fun4LeetCode never really pointed out, is the number of steps of the current "round-trip", respectively, starting from $$(0, 0)$$ ending in $$(i, j)$$, and starting from $$(p, q)$$ ending in $$(0, 0)$$. And so it can be at most $$2n - 2$$, since going from top left to bottom right always takes $$2n - 2$$ steps.

The second and third dimensions are respectively the row coordinate of the ending point of the forth trip, and the row coordinate of the starting point of the back trip.

The solution cleverly uses backward updating to avoid a 3-D matrix, reducing space complexcity to $$O(n^2)$$.

#### Transition Equation Set

The transition equation set, as specified in aforementioned article, is:

```
DP(i, p, numSteps) = max(DP(i, p, numSteps - 1), DP(i, p - 1, numSteps - 1), DP(i - 1, p, numSteps - 1), DP(i - 1, p - 1, numSteps - 1)) + grid[i][j] + grid[p][q];
```

But this transition equation set is really abstract. What does it mean in a **geometrical** sense? I am too lazy to plot pictures for this, so here goes the essence of the transtion equation set:

- DP(i, p, numSteps - 1): entering $$(i, j)$$ from left and leaving $$(p, q)$$ from left.

- DP(i, p - 1, numSteps - 1): entering $$(i, j)$$ from left and leaving $$(p, q)$$ from top.

- DP(i - 1, p, numSteps - 1): entering $$(i, j)$$ from top and leaving $$(p, q)$$ from left.

- DP(i - 1, p - 1, numSteps - 1): entering $$(i, j)$$ from top and leaving $$(p, q)$$ from top.

Savvy?

#### Duplicate Avoidance

I don't think @fun4LeetCode's explanation for how this solution avoids overlapping back and forth paths made any sense to me. Maybe I am just too dumb to understand it. Here is my interpretation.

The key logic for avoiding the duplicate counting of cherries is:

```dp[i][p] += grid[i][j] + (i != p ? grid[p][q] : 0);```

The DP solution builds bottom-up the optimal round-trip to take. At each position in the DP table $$(i, p)$$, the cherry at current position, if any, is de-duplicated by the above line of code. And we are basing the current optimal round-trip count on the left and top neighbors, which have already run above code for de-duplicate. Hence even if the two paths overlap, any one cherry would not be counted twice.

#### C++ Solution

My implementation of the solution is posted below, with the variables named more clearly and helpful to understanding. Some boundary checkings not mentioned in the perfect mathematical model must also included in the actual code. Also, the last bit, ```max(dp[n - 1][n - 1], 0)``` shall not be forgotten.

```c++
class Solution {
public:
    int cherryPickup(vector<vector<int>>& grid) {
        int n = grid.size();
        if (!n)
            return 0;
        vector<vector<int>> dp(n, vector<int>(n, 0));
        dp[0][0] = grid[0][0];
        int maxSteps = 2 * n - 2;
        for (int numSteps = 1; numSteps <= maxSteps; numSteps++) {
            for (int i = n - 1; i >= 0; i--) {
                for (int p = n - 1; p >= 0; p--) {
                    int j = numSteps - i, q = numSteps - p;
                    if (j < 0 || q < 0 || j >= n || q >= n || grid[i][j] < 0 || grid[p][q] < 0) {
                        dp[i][p] = -1;
                        continue;
                    }
                    if (i > 0)
                        dp[i][p] = max(dp[i][p], dp[i - 1][p]);
                    if (p > 0)
                        dp[i][p] = max(dp[i][p], dp[i][p - 1]);
                    if (i > 0 && p > 0)
                        dp[i][p] = max(dp[i][p], dp[i - 1][p - 1]);
                    if (dp[i][p] >= 0)
                        dp[i][p] += grid[i][j] + (i != p ? grid[p][q] : 0);
                }
            }
        }
        return max(dp[n - 1][n - 1], 0);
    }
};
```

**Complexity Analysis**

* Time complexity: $$O(n^3)$$.

This is already very optimal for a complex problem like this.

* Space complexity: $$O(n^2)$$.

### End

This approach is extensible: if the cherry pickup becomes round-trip space debris collecting in 3-D space, we could solve this using 4-D DP.

Signing off.