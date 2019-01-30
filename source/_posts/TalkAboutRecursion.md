---
title: 浅谈递归
date: 2019-01-30 22:35:40
tags:
	- Tree
	- Recursion
categories:
	- Algorithm
---

递归，是计算机独有的技能，由于我们的大脑无法堆栈记录状态，所以很难递归的解决问题。但是，递归的思想是可以练就的，有了思想，就命令计算机去完成操作就行了。不要看数学上的什么递归式求封闭解，一堆推倒让人不想看，我们有计算机，不需要求递推封闭解就能优雅解决问题。

<!--more-->

以下会举例说明我对递归的一点理解，**如果你不想看下去了，请记住这几个问题怎么回答：**

1. 如何给一堆数字排序？ 答：先排左边再排右边，最后合并就行了，至于怎么排左边和右边，请重新阅读这句话。
2. 孙悟空身上有多少根毛？ 答：一根毛加剩下的毛。
3. 你今年几岁？ 答：去年的岁数加一岁......

递归代码最重要的两个特征：结束条件和自我调用。自我调用是在解决子问题，而结束条件定义了最简子问题的答案。其实仔细想想，**递归运用最成功的是什么？我认为是数学归纳法。**我们高中都学过数学归纳法，使用场景大概是：我们推不出来某个求和公式，但是我们试了几个比较小的数，似乎发现了一点规律，然后编了一个公式，看起来应该是正确答案。但是数学是很严谨的，你哪怕穷举了一万个数都是正确的，但是第一万零一个数正确吗？这就要数学归纳法发挥神威了，可以假设我们编的这个公式在第 k 个数时成立，如果证明在第 k + 1 时也成立，那么我们编的这个公式就是正确的。

那么数学归纳法和递归有什么联系？我们刚才说了，递归代码必须要有结束条件，如果没有的话就会进入无穷无尽的自我调用，直到内存耗尽。而数学证明的难度在于，你可以尝试有穷种情况，但是难以将你的结论延伸到无穷大。这里就可以看出联系了 —— 无穷。

递归代码的精髓在于调用自己去解决规模更小的子问题，直到到达结束条件；而数学归纳法之所以有用，就在于不断把我们的猜测递归地加一，扩大结论的规模，没有结束条件，从而把结论延伸到无穷无尽，也就完成了猜测正确性的证明。

**从编程的视角来看**（后续准备写一篇从归纳法出发，指导编程的文章），为什么要递归？

首先为了训练逆向思考的能力。递推的思维，总是看着眼前的问题思考对策，解决问题是将来时；递归的思维，逼迫我们倒着思考，看到问题的尽头，把解决问题的过程看做过去时。

第二，练习分析问题的结构，当问题可以被分解时，你能敏锐发现这个特点，进而高效解决问题。多想想归并排序是怎么做的。

第三，跳出细节，从整体上看问题。再说说归并排序，其实可以不用递归来划分左右区域的，但是代价就是代码极其难以理解，大概看一下代码：

```java
void sort(Comparable[] a){    
    int N = a.length;
    // 这么复杂，是对排序的不尊重。我拒绝研究这样的代码。
    for (int sz = 1; sz < N; sz = sz + sz)
        for (int lo = 0; lo < N - sz; lo += sz + sz)
            merge(a, lo, lo + sz - 1, Math.min(lo + sz + sz - 1, N - 1));
}

/* 我还是选择递归，简单，漂亮 */
void sort(Comparable[] a, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    sort(a, lo, mid);
    sort(a, mid + 1, hi);
    merge(a, lo, mid, hi);
}

```

看起来简洁漂亮是一方面，关键是可解释性很强：把左半边排序，把右半边排序，最后合并两边。而非递归版本看起来不知所云，充斥着各种难以理解的边界计算细节，而且归并排序的空间复杂度只有 logn ，对数级别的复杂度更常数级也差不多了，所以我更倾向于递归版本。

显然有时候递归处理是高效的，比如归并排序，有时候是低效的，比如数孙悟空身上的毛，因为堆栈会消耗额外空间，而简单的递推不会消耗空间。比如这个例子，给一个链表头，计算它的长度：

```java
/* 典型的递推遍历框架 */
public int size(Node head) {
    int size = 0;
    for (Node p = head; p != null; p = p.next) size++;
    return size;
}
/* 我偏要递归，万物皆递归 */
public int size(Node head) {
    if (head == null) return 0;
    return size(head.next) + 1;
}
```



关于**写递归的技巧**，之前讲快排和归并的文章写了：明白一个函数的作用并相信它能完成这个任务，千万不要试图跳进细节。以下用 LeetCode 的一道题来说明：给一课二叉树，和一个目标值，节点上的值有正有负，返回树中和等于目标值的路径条数，让你编写 pathSum 函数：

```
/* 来源于 LeetCode PathSum III： https://leetcode.com/problems/path-sum-iii/ */
root = [10,5,-3,3,2,null,11,3,-2,null,1], sum = 8

      10
     /  \
    5   -3
   / \    \
  3   2   11
 / \   \
3  -2   1

Return 3. The paths that sum to 8 are:

1.  5 -> 3
2.  5 -> 2 -> 1
3. -3 -> 11
```

```cpp
/* 看不懂没关系，底下有更详细的分析版本，这里突出体现递归的简洁优美 */
int pathSum(TreeNode root, int sum) {
    if (root == null) return 0;
    return count(root, sum) + pathSum(root.left, sum) + pathSum(root.right, sum);
}
int count(TreeNode node, int sum) {
    if (node == null) return 0;
    return (node.val == sum) + count(node.left, sum - node.val) + count(node.right, sum - node.val);
}
```

题目看起来很复杂吧，不过代码却极其简洁，这就是递归的魅力。我来简单总结这个问题的**解决过程**：

首先明确，递归求解树的问题必然是要遍历整棵树的，所以二叉树的遍历框架（分别对左右孩子递归调用函数本身）必然要出现在主函数 pathSum 中。那么对于每个节点，他们应该干什么呢？他们应该看看，自己和脚底下的小弟们包含多少条符合条件的路径。好了，这道题就结束了。

按照前面说的技巧，根据刚才的分析来定义清楚每个递归函数应该做的事：

PathSum 函数：给他一个节点和一个目标值，他返回以这个节点为根的树中，和为目标值的路径总数。

count 函数：给他一个节点和一个目标值，他返回以这个节点为根的树中，能凑出几个以该节点为路径开头，和为目标值的路径总数。

```cpp
/* 有了以上铺垫，详细注释一下代码 */
int pathSum(TreeNode root, int sum) {
    if (root == null) return 0;
    int pathImLeading = count(root, sum); // 自己为开头的路径数
    int leftPathSum = pathSum(root.left, sum); // 左边路径总数（相信他能算出来）
    int rightPathSum = pathSum(root.right, sum); // 右边路径总数（相信他能算出来）
    return leftPathSum + rightPathSum + pathImLeading;
}
int count(TreeNode node, int sum) {
    if (node == null) return 0;
    // 我自己能不能独当一面，作为一条单独的路径呢？
    int isMe = (node.val == sum) ? 1 : 0;
    // 左边的小老弟，你那边能凑几个 sum - node.val 呀？
    int leftBrother = count(node.left, sum - node.val); 
    // 右边的小老弟，你那边能凑几个 sum - node.val 呀？
    int rightBrother = count(node.right, sum - node.val);
    return  isMe + leftBrother + rightBrother; // 我这能凑这么多个
}
```

还是那句话，明白每个函数能做的事，并相信他们能够完成。

总结下，PathSum 函数提供的二叉树遍历框架，在遍历中对每个节点调用 count 函数，看出先序遍历了吗（这道题什么序都是一样的）；count 函数也是一个二叉树遍历，用于寻找以该节点开头的目标值路径。好好体会吧！