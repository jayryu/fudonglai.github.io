---
title: 数组旋转算法
date: 2018-12-23 13:19:18
tags:
	- Array
	- LinkedList
	- Recursion
categories:
	- Algorithm
---

交换数组的两个区域很简单吗？

以下给出链表解法，递归解法，连续翻转，杂技解法。

<!-- more -->

题目和图片来自 [LeetCode](https://leetcode.com/explore/featured/card/top-interview-questions-easy/92/array/646/)。

## Problem: 

Given an array, rotate the array to the right by *k* steps, where *k* is non-negative.

**Example 1:**

```
Input: [1,2,3,4,5,6,7] and k = 3
Output: [5,6,7,1,2,3,4]
Explanation:
rotate 1 steps to the right: [7,1,2,3,4,5,6]
rotate 2 steps to the right: [6,7,1,2,3,4,5]
rotate 3 steps to the right: [5,6,7,1,2,3,4]
```

**Example 2:**

```
Input: [-1,-100,3,99] and k = 2
Output: [3,99,-1,-100]
Explanation: 
rotate 1 steps to the right: [99,-1,-100,3]
rotate 2 steps to the right: [3,99,-1,-100]
```

## Reflection:

这个问题实际上可以简化为如何交换数组的两部分，但是在思维模式上还是有差别，那么首先按照题目的意思，Rotate 一个数组，相当于循环位移每个元素？显然对于数组来说，看起来简单的整体移动涉及大量数据搬移，可以说非常低效。数组要是能连接成一个环就好了？那我用链表造个环不就完事了。

抛开循环移动这个思维，`Rotate`这个操作的结果不就是把数组某个节点两边数据交换了一下吗？这个操作看起来也好简单啊，swap 就完了？但是事实好像并没有这么简单。



## Approach I: Cycle Linked List

**Time Complexity: O(n), Space Complexity: O(n)**

我们创造一个环链表，记住开头元素位置，然后走相应步数之后，把这个地方作为开头，读取环链表，得到的就是结果了。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
void rotate(vector<int>& nums, int k) {
    auto head = new ListNode(0);
    ListNode *p = head;
    for (int n : nums) {
        p->next = new ListNode(n);
        p = p->next;
    }
    p->next = head->next;
    p = p->next;
    int step = nums.size() - k % nums.size();
    while (step--) p = p->next;
    for (int i = 0; i < nums.size(); i++) {
        nums[i] = p->val;
        p = p->next;
    }
}

```

## Approach II: Recursion 

**Time Complexity: O(n), Space Complexity: O(n)**

抛开整体移动，直接粗暴地交换元素的话，两个区域长度相同的话直接`swap` n/2 次就完全交换了，但是如果两个区域长度不同，长的那部分就会有剩余，该剩余区域还需要和长区域经过交换的那个区域交换，以还原长区域的原始顺序，然后这个交换过程又会产生长短不一的问题，然后请重新阅读这段话。

子问题和原问题结构完全相同，规模更小，明显的递归结构，结束边界就是两个区域长度相等。

```cpp
void recursiveSwap(vector<int>& nums, int k, int start, int length) {
    k %= length;
    if (k == 0) return;
    for (int i = 0; i < k; i++) 
        swap(nums[start + i], nums[nums.size() - k + i]);
    recursiveSwap(nums, k, start + k, length - k);
}
```

空间复杂度是最坏情况，也就是交换`nums[0]`和`nums[1...size-1]`的极端情况。

写出该递归数学形式，求解封闭解，可以把该函数改为封闭递推形式：

**Time Complexity: O(n), Space Complexity: O(1)**

```cpp
void rotate(vector<int>& nums, int k) {
    int n = nums.size();
    for (; k %= n; n -= k)
        for (int i = 0; i < k; i++)
            swap(nums[i], nums[n - k]);
}
```

关于递归式求解封闭解，需要数学技巧，并且写出来的代码基本看不懂，不会就算了。



## Approach III: Magic Method

**Time Complexity: O(n), Space Complexity: O(1)**

类似链表解法，但是不需要额外的链表节点表示，直接在数组上进行跳跃。反正描述不清楚，看代码自行体会一下。

```
nums: [1, 2, 3, 4, 5, 6]
k: 2
```

![example](https://leetcode.com/media/original_images/189_Rotate_Array.png)

```cpp
void rotate(vector<int>& nums, int k) {
        k %= nums.size();
        int step = 0;
        for (int start = 0; step < nums.size(); start++) {
            int curr = start;
            int prev_val = nums[start];
            do {
                int next = (curr + k) % nums.size();
                int temp = nums[next];
                nums[next] = prev_val;
                prev_val = temp;
                curr = next;
                step++;
            } while (curr != start);
        }
    }
```



## Approach IV: KISS

**Time Complexity: O(n), Space Complexity: O(1)**

只有看到这里的真的猛士才配拥有真正的终极算法。

交换两段数据？只要分别反转这两段，再把整个数组反转就行了，不信自己数一数。

```cpp
// Keep it simple, stupid.
void rotate(vector<int>& nums, int k) {
        k %= nums.size();
        int n = nums.size();
        reverse(nums.begin(), nums.begin() + n - k);
        reverse(nums.begin() + n - k, nums.end());
        reverse(nums.begin(), nums.end());
    }
```

