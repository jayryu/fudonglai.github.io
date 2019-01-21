---
title: 二分搜索详解
date: 2019-01-21 16:14:29
tags:
	- Search
	- Array
categories:
	- Algorithm

---



二分查找，学过一点点算法的人都会，但是很简单的查找算法变体非常广泛，并且非常不容易写正确，不信你就试试？几个月前在 LeetCode 看二分搜索的时候就有点云里雾里，道理我都懂，怎么就是写不对？？现在再回头做以前的题，好像有点开窍了，赶紧记录下来。

<!--more-->

**PS：Template II 是最核心的结论。**

首先，二分搜索不是只能简简单单的在一堆有序数列中找一个元素，再设计算法时，只要看到有序序列，第一就要想到 Binary Search，这是最高效的算法，**具体操作区别在于”二分“判断的部分**。总体来说，二分查找的框架是：

```cpp
int binarySearch(vector<int> nums，int target) {
    int lo = 0, int hi = nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2; //防止溢出而已
        if (左半边可以排除)
            调整 lo;
        else 
            调整 hi;
    }
    return lo 或 hi 或 -1;
}

```

这个简单的框架有四个地方可以自由设计：

1. `hi`的取值，可以取为`nums.size() - 1`，也可能是`nums.size()`，看具体用哪一个代码模板了（下文讲到）。
2. 如果使用不同模板，`while`条件的判断也不同，取决于上一条。
3. mid 的计算，我们知道计算机除法是向下取整，所以这个 mid 并不一定是严谨的正中间，有可能会偏差，但这个偏差一定是偏左。注意到这一点会帮助我们在模板二的调整操作。
4. 如何调整`lo`和`hi`，到底等于`mid`还是`mid+１`还是`mid-1`？这个取决于上一点，还取决于我们要搜索 target 的左侧边界还是右侧边界。你要问 target 不就是一个数吗，为什么还有左侧右侧边界？应为这个序列不一定是没有重复数字的，假设序列 `[1,2,3,3,4]`，让你查找 3 的区间，你是是不是得查找左侧边界和右侧边界？



# Template I

```cpp
int binarySearch(vector<int>& nums, int target) {
    int lo = 0, hi = nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] > target) {
            hi = mid - 1;
        } else if (nums[mid] < target) {
            lo = mid + 1;
        } else { // found target
            return mid;
        }
    }
    return -1;
}
```

最简陋的二分查找函数，只能查找单个元素，我还没想到什么适合它的变体。而且它的结果是”随机“的，就是说遇到有 target 值重复的情况，根据初始 lo 和 hi 的值的不同，返回的 target 索引可能是不同的。

话说为什么会这样？因为有一个条件判断了`nums[min] == target`的情况，这就是说只要找到了就 return，自然无法找到重复 target 的边界了。我个人不喜欢这样的搜索，有点”硬编码“的味道，我还是喜欢不用 `==`判断，让算法自然收敛到左侧或者右侧，后面的模板都按自然收敛的风格设计。

这样有啥好处呢？平均复杂度确实低一些，比如说有可能第一次循环就恰好落到 target 值了，那就直接结束了。而自然收敛的算法复杂度是稳定的 O(lgn)。不过，lgn 你懂得，顶级黑魔法了，根本不在乎多搜索这几下嘛。



# Template II

```cpp
int left_bound(vector<int>& nums, int target) {
    if (nums.empty()) return -1;
    int lo = 0, hi = nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= target) {
            hi = mid;
        } else if (nums[mid] < target) {
            lo = mid + 1;
        }
    }
    return nums[lo] == target ? lo : -1;
}
```

二分查找的高级模式，注意区别，把每轮循环对`hi`的更新修改了，把等号和大于号合并了，并在开头添加了判断。

这里的精髓有两个：

1. 到底是`>=`还是`<=`？
2. 到底是`lo = mid + 1`还是`hi = mid - 1`还是`lo`和`hi`都等于`mid`？

**解决第一个问题的关键是判断 `nums[mid]` 恰好撞上 `target` 的情况，来决定带不带等号。**我发现必须是 hi = mid 那个判断带等号，这是这个框架决定的，应为除法自然向下取整，恰好撞上 target 时必须是 hi = mid，否则如果是 lo = mid + 1 的话会跳过这个 mid，也就是跳过了 target，跳过了正确结果，必然是错误的。

**解决第二个问题的关键是思考算法结束的最后一步，已经收缩到两个元素了，不妨假设在`[1,2]`中搜索 2**，计算mid 时结果不可避免地偏左，这就必须要求`lo`必须自加才能满足 while 的结束条件，否则就会陷入无限循环（模板三解决该问题就不需要自加，而是直接更改了 while 的条件）。

在解决问题的同时我发现这个算法结果是左侧边界，为什么，是因为除法自然左偏的结果吗？不是！而是 if 条件判断中不等号中等号的位置来决定，当算法计算出`nums[mid] == target` 时，执行的更新`hi`的操作，就是说`hi`在向左收敛，最终收敛到左侧边界。

那我把等号给到小于判断上，是不是就可以收敛到右侧边界？不行！上面第一个问题就在解释，这是必须的，由除法向下取整的性质决定的，请仔细体会下。

**所以总结下，以上逻辑的因果链是：由于除法的向下取整性质，决定了必须是`lo = lo + 1`以避免无限循环，从而决定了等号必须给到大于号，否则算法会跳过正确答案而出错；因此此算法的结果是收敛到左侧边界的。**

那么我就是想让算法收敛到右侧怎么办？根据以上的因果链，关键在于除法的性质，如果让除法能向上取整就好说了。这个简单，这样写都可以：

```cpp
int mid = (lo + hi + 1) / 2;
int mid = (lo + hi) / 2 + 1;
```

不一定要严谨的向上取整，因为算法会自然调整，只要避免无限循环就会收敛。向右取整决定了避免无限循环必须更新`hi = mid - 1`；更改`hi`就决定了等号需要给到更新`lo`的条件判断里，也就是必须`<=`。代码如下：

```cpp
int right_bound(vector<int>& nums, int target) {
    if (nums.empty()) return -1;
    int lo = 0, hi = nums.size() - 1;
    while (lo < hi) {
        int mid = (lo + hi + 1) / 2;
        // int mid = lo + (hi - lo) / 2 + 1; Both OK
        if (nums[mid] > target) {
            hi = mid - 1;
       } else if (nums[mid] <= target) {
            lo = mid;
        }
    }
    return nums[hi] == target ? hi : -1;
}
```

嗯，到此，二分搜索的代码写起来已经不会有啥 bug 了，而且能够自由调整收敛方向，这个模板是我最常用的。下面再介绍一种模板吧，其实和这个差不多，一个原理，了解一下好了。



# Template III

```cpp
int lower_bound(vector<int>& nums, int target) {
        if (nums.empty()) return -1;
        int lo = 0, hi = nums.size();
        while (lo < hi - 1) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] >= target) //!!!
                hi = mid;
            else
                lo = mid;
        }
        if (hi == nums.size())  return nums[lo] == target ? lo : -1;
        if (nums[lo] == target) return lo; //!!!
        if (nums[hi] == target) return hi; //!!!
        return -1;
    }
    
    int higher_bound(vector<int>& nums, int target) {
        if (nums.empty()) return -1;
        int lo = 0, hi = nums.size();
        while (lo < hi - 1) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] > target) //!!!
                hi = mid;
            else 
                lo = mid;
        }
        if (hi == nums.size())  return nums[lo] == target ? lo : -1;
        if (nums[hi] == target) return hi; //!!!
        if (nums[lo] == target) return lo; //!!!
        return -1;
    }
```

这个模板有它的特色，就是不用担心陷入无限循环，搜索左侧边界和右侧边界除了等号的区别外，还有个判断先后的问题需要注意，不过这些细节只要认真研究边界情况，都可解决。



# Easy Problem

给一个数组，如何找到`peak element`的索引？比如 [1,2,3,4,3,2,1] 中 4 就是`peak element`，返回 4 的索引 3 。假设`nums[-1]`和`nums[nums.size()]`都是`-inf`。

这就是二分搜索的简单变体啊，只不过把 `nums[mid]` 和 target 的大小判断改成了`nums[mid]`处在”上坡“还是”下坡“的判断罢了。

```cpp
int findPeakElement(vector<int>& nums) {
        int lo = 0, hi = nums.size() - 1;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (nums[mid] < nums[mid + 1]) {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        return lo;    
    }
```



