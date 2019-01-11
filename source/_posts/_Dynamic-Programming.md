---
title: Dynamic Programming
date: 2018-12-30 12:03:10
tags: 
	- Dynamic Programming
categories:
	- Algorithms
---

```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        return helper(coins, amount, 0);
    }
    
    int helper(vector<int>& coins, int amount, int step) {
        if (amount == 0) return step;
        if (amount < 0) return -1;
        int ans = INT_MAX;
        for (int coin : coins) {
            int temp = helper(coins, amount - coin, step + 1);
            if (temp != -1)
                ans = min(ans, temp);
        }
        return ans;
    }
};

```



```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        if (amount < 1) return 0;
        return help(coins, amount, vector<int>(amount + 1, 0));
    }
    
    int help(vector<int>& coins, int amount, vector<int> memo) {
        if (amount < 0) return -1;
        if (amount == 0) return 0;
        if (memo[amount] != 0) return memo[amount];
        int ans = INT_MAX;
        for (int coin : coins) {
            int temp = help(coins, amount - coin, memo);
            if (temp != -1) 
                ans = min(temp, ans) + 1;
        }
        memo[amount] = (ans == INT_MAX) ? -1 : ans;
        return memo[amount];
    }
};
```

