---
title: 排序I - 归并 & 快排
date: 2019-01-10 19:03:18
tags:
	- Sort
	- 算法导论
	- C/C++
categories:
	- Algorithm
---



最经又刷了不少算法题，啃了啃《算法导论》，觉得很多算法光写而不拿出来总结，还是不能体会出精髓，于是将自己吸收的东西总结一下，力求能让其他人都看懂。同时练习一下 C++ 泛型编程，写成泛型算法。

<!-- more -->

# Merge Sort

归并排序，典型的分治算法；分治，典型的递归结构。

分治算法可以分三步走：分解 -> 解决 -> 合并

1. 分解原问题为结构相同的子问题。
2. 分解到某个容易求解的边界之后，进行第归求解。
3. 将子问题的解合并成原问题的解。

**递归** 并不是一种算法，而只是一种编程技巧，例如在分治算法的第二步，很容易看出递归结构，所以分治算法都是由递归实现的。关于递归，我会抽时间专门写一篇文章，从数学和计算机的角度来简单分析一下。

这里只说一点我编写递归代码的心得：**递归函数只做一件事，并且你相信它一定能做好**，千万不要跳进这个函数里面企图探究更多细节，否则就会陷入无穷的细节无法自拔，人脑能压几个栈啊。就举个最简单的例子：遍历二叉树。

```cpp
void traverse(TreeNode* root) {
    if (root == nullptr) return;
    traverse(root->left);
    traverse(root->right);
}
```

这几行代码就足以扫荡任何一棵二叉树了。我想说的是，对于递归函数`traverse(root)`，我们只要相信：给它一个根节点`root`，它就能遍历这棵树，因为写这个函数不就是为了这个目的吗？所以我们只需要把这个节点的左右节点再甩给这个函数就行了，因为我相信它能完成任务的。那么遍历一棵N叉数呢？太简单了好吧，和二叉树一模一样啊。

```cpp
void traverse(TreeNode* root) {
    if (root == nullptr) return;
    for (child : root->children)
        traverse(child);
}
```

至于遍历的什么前、中、后序，那都是显而易见的，对于N叉树，显然没有中序遍历。

写了这么多题外话，就是让你记住，**明确写出来的这个函数的职责，并且相信它一定能完成**，这样就能看懂，甚至随手写出漂亮的递归代码了。在之后的[回溯算法]()，[动态规划]()中还会有大量算法由递归实现。现在言归正传。

归并排序，我们就叫这个函数`merge_sort`吧，按照我们上面说的，要明确该函数的职责，即**对传入的一个数组排序**。OK，那么这个问题能不能分解呢？当然可以！给一个数组排序，不就等于给该数组的两半分别排序，然后合并就完事了。

```cpp
void merge_sort(一个数组) {
    if (可以很容易处理) return;
    merge_sort(左半个数组);
    merge_sort(右半个数组);
    merge(左半个数组, 右半个数组);
}
```

好了，这个算法也就这样了，完全没有任何难度。记住之前说的，相信函数的能力，传给他半个数组，那么这半个数组就已经被排好了。而且你会发现这不就是个二叉树遍历模板吗？为什么是后序遍历？因为我们分治算法的套路是 **分解 -> 解决（触底） -> 合并（回溯）** 啊，先左右分解，再处理合并，回溯就是在退栈，就相当于后序遍历了。至于`merge`函数，参考两个有序链表的合并，简直一模一样，下面直接贴代码吧。



注意三点就行：

1. 我给每一半数组的最后一位添加了一个哨兵位，理论上应该是正无穷，这里用INT_MAX代替。
2. 形参 `begin`, `end`参考 C++ 迭代器，取左闭右开区间，即数组范围在 `[begin, end)`。
3. `pivot = begin + (end - begin) / 2`实际上就是取中值，防止溢出罢了。

```cpp
void merge_sort(vector<int> &nums, int begin, int end) {
    if (begin >= end - 1) return;
    int pivot = begin + (end - begin) / 2;
    merge_sort(nums, begin, pivot);
    merge_sort(nums, pivot, end);
    merge(nums, begin, pivot, end);
}

void merge(vector<int> &nums, int begin, int pivot, int end) {
    vector<int> left(nums.begin() + begin, nums.begin() + pivot);
    vector<int> right(nums.begin() + pivot, nums.begin() + end);
    left.push_back(INT_MAX);
    right.push_back(INT_MAX);
    int p = 0, q = 0;
    for (int i = begin; i < end; i++) {
        if (left[p] < right[q]) {
            nums[i] = left[p++];
        } else {
            nums[i] = right[q++];
        }
    }
}
```



