---
title: Fundamental DP Problem
date: 2018-10-26 10:52:13
mathjax: true
tags:
	- dynamic programming
categories:
	- Algorithm
---

There is an interesting problem in LeetCode: [Unique Paths](https://leetcode.com/problems/unique-paths/).

I found some pespicuous explaination in the comments.

<!--more-->

### Description: 

> A robot is located at the top-left corner of a *m* x *n* grid.
>
> The robot can only move either down or right at any point in time. The robot is trying to reach the bottom-right corner of the grid.
>
> How many possible unique paths are there?

### Observations:

Since the robot can only move right and down, when it arrives at a point, there are only two possibilities:

1. It arrives at that point from above (moving down to that point);
2. It arrives at that point from left (moving right to that point).

Thus we have the following equations: $Approach(i, j)=Approach(i - 1, j) + Approach(i, j-1)$

![picture](http://ph6w748rg.bkt.clouddn.com/18-10-26/15711693.jpg)

The boundary conditions of the above equation have some problems at the leftmost column and the uppermost row due to $(0, j)$ and $(i, 0)$ do not exist. To oversmart the problem automatically,  it's easier that we initialize the array to something like following:

![picture](http://ph6w748rg.bkt.clouddn.com/18-10-26/36462436.jpg)

### Implement

So, here is my codes:

```cpp
int uniquePaths(int m, int n) {
        vector<vector<int>> map(m + 1, vector<int>(n + 1, 0));
        map[1][0] = 1;
        for (int i = 1; i < m + 1; ++i) 
            for (int j = 1; j < n + 1; ++j) 
                map[i][j] = map[i][j - 1] + map[i - 1][j];
        return map[m][n];
    }
```

As can be seen, the above solution runs in $O(n^2)$ time and costs $O(m*n)$ space.

But we find that each time we update the `map[i][j]`, we only need to consider the current column(for `map[i-1, j]`) and the previous column(for `map[i, j-1]`), so only two vector can be enough.

Here is new codes:

```cpp
int uniquePaths(int m, int n){
        vector<int> c1(m + 1, 0), c2(c1);
        c1[1] = 1;
        for (int i = 1; i <= n; ++i) {
            for (int j = 1; j <= m; ++j)
                c2[j] = c1[j] + c2[j - 1];
            c1 = c2;
            c2 = vector<int>(m + 1, 0);
        }
        return c1[m];
    }
```

Further inspecting the above code, we find it needs $O(m)$ space, but if $n<m$, should I convert to $O(n)$ space? And we find that `vector c2`will convert to zero in every loop, so the operation is actually `c1[i] += c1[i-1]`. Now we can implement a more efficient approach:

```cpp
int uniquePaths(int m, int n){
       if (m > n)
           return uniquePaths(n, m);
        vector<int> c(m, 0);
        c[0] = 1;
        for (int i = 1; i <= n; ++i) 
            for (int j = 1; j <= m - 1; ++j) 
                c[j] += c[j - 1];
        return c[m - 1];
    }
```

Now it is prefect, how interesting it is!