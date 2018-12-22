---
title: 深度复制随机链表
date: 2018-12-22 23:03:31
tags:
	- LinkedList
categories:
	- Algorithm
---

以下有三种常规解决方案，和一种杂技解法。

Best Approach: Time Complexity O(n), Space Complexity O(1).

<!-- more -->

**题目及图片均源于 [LeetCode](https://leetcode.com/explore/interview/card/top-interview-questions-hard/117/linked-list/841/]) **。

题目：给一个链表，这个链表有点特殊，即每个节点还包含一个随机指针，随机指向链表中的某一个节点，请你深度复制（DeepCopy）这个链表。

```java
/**
 * Definition for singly-linked list with a random pointer.
 * class RandomListNode {
 *     int label;
 *     RandomListNode next, random;
 *     RandomListNode(int x) { this.label = x; }
 * };
 */
```

![RandomLinkedList](https://leetcode.com/problems/copy-list-with-random-pointer/Figures/138/138_Copy_List_Random_1.png)

首先想到，这随机链表不就是个图吗？首先就想到 [Clone Graph](https://leetcode.com/problems/clone-graph/)，最起码有深度优先和广度优先两种方法。

## Approach I: DFS

**Time Complexity O(n), Space Complexity O(n)**

递归实现简单易懂，确定边界条件，然后调用自己就完事了。

```java
public class Solution {
    public HashMap<RandomListNode, RandomListNode>
        visited = new HashMap<>();
    public RandomListNode copyRandomList(RandomListNode head) {
        if (head == null) return head;
        if (visited.containsKey(head)) return visited.get(head);
        RandomListNode cp = new RandomListNode(head.label);
        visited.put(head, cp);
        cp.next = copyRandomList(head.next);
        cp.random = copyRandomList(head.random);
        return cp;
    }
}
```



## Approach II: BFS

**Time Complexity O(n), Space Complexity O(n)**

这段是我用 C++ 写的，借鉴图克隆的算法，可以说都是一套模板了，个人认为没有递归舒服。

```cpp
class Solution {
public:
    unordered_map<RandomListNode*, RandomListNode*> map;
    
    RandomListNode *copyRandomList(RandomListNode *head) {
        if (!head) return NULL;
        queue<RandomListNode*> q;
        q.push(head);
        map[head] = new RandomListNode(head->label);
        while (!q.empty()){
            auto front = q.front(); q.pop();
            if (front->next && map.find(front->next) == map.end()) {
                map[front->next] = new RandomListNode(front->next->label);
                q.push(front->next);
            }
            if (front->random && map.find(front->random) == map.end()) {
                map[front->random] = new RandomListNode(front->random->label);
                q.push(front->random);
            }
            map[front]->random = map[front->random];
            map[front]->next = map[front->next];
        }
        return map[head];
    }
```



## Approach III: Double Pointer

**Time Complexity O(n), Space Complexity O(n)**

常规方法吧，类似双指针同步操作。

```java
public class Solution {
    private HashMap<RandomListNode, RandomListNode>
        visited = new HashMap<>();
    
    private RandomListNode getCloneNode(RandomListNode node) {
        if (node == null) return node;
        if (!visited.containsKey(node))
            visited.put(node, new RandomListNode(node.label));
        return visited.get(node);
    }
    
    public RandomListNode copyRandomList(RandomListNode head) {
        if (head == null) return head;
        RandomListNode oldNode = head;
        RandomListNode newNode = getCloneNode(head);
        while (oldNode != null) {
            newNode.next = getCloneNode(oldNode.next);
            newNode.random = getCloneNode(oldNode.random);
            oldNode = oldNode.next;
            newNode = newNode.next;
        }
        return visited.get(head);
    }
}
```



## Approach IV：Magic Method

**Time Complexity O(n), Space Complexity O(1)**

比较难想到，竟然不需要额外存储空间来记录已生成的节点？

本方法分为两步走：

1. 克隆原节点的 next 域和 random 域，然后将克隆的节点（cloned）放到原始节点（original）之后，此处需要链表插入操作：
  ```
   cloned.next = original.next
   original.next = cloned
  ```
  ![step1](https://leetcode.com/articles/Figures/138/138_Copy_List_Random_9_1.png)
2. 解开 cloned 和 original 的连接，分为两条链表，完成复制。

![step2](https://leetcode.com/articles/Figures/138/138_Copy_List_Random_10.png)

Java 代码实现：

```java

public class Solution {
    public RandomListNode copyRandomList(RandomListNode head) {
        if (head == null) return head;
        RandomListNode original = head;
        while (original != null) {
            RandomListNode cloned = new RandomListNode(original.label);
            cloned.next = original.next;
            original.next = cloned;
            original = cloned.next;
        }
        original = head;
        while (original != null) {
            original.next.random = 
                (original.random != null) ? original.random.next : null;
            original = original.next.next;
        }
        RandomListNode cloned_head = head.next;
        RandomListNode cloned = cloned_head;
        original = head;
        while (cloned != null) {
            original.next = cloned.next;
            original = original.next;
            cloned.next = (original != null) ? original.next : null;
            cloned = cloned.next;
        }
        return cloned_head;
    }
}
```