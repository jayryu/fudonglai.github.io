---
title: Non-Repeating Longest Substring
date: 2018-10-20 18:24:49
mathjax: true
tags:	
	- Substring
categories: 
	- Algorithm
---

An ingenuity to use map to solve Substring.

<!--more-->

### Example:

> Input: "pwwkew"
> Output: 3
> Explanation: The answer is "wke", with the length of 3. 

###Solution


```cpp
 * Solution (DP, O(n)):
 * 
 * Assume L[i] = s[m...i], denotes the longest substring without repeating
 * characters that ends up at s[i], and we keep a hashmap for every
 * characters between m ... i, while storing <character, index> in the
 * hashmap.
 * We know that each character will appear only once.
 
 * Then to find s[i+1]:
 * 1) if s[i+1] does not appear in hashmap
 *    we can just add s[i+1] to hash map. and L[i+1] = s[m...i+1]
 * 2) if s[i+1] exists in hashmap, and the hashmap value (the index) is k
 *    let m = max(m, k), then L[i+1] = s[m...i+1], we also need to update
 *    entry in hashmap to mark the latest occurency of s[i+1].
 * 
 * Since we scan the string for only once, and the 'm' will also move from
 * beginning to end for at most once. Overall complexity is O(n).
 *
 * If characters are all in ASCII, we could use array to mimic hashmap.
 */

class Solution {
public:
   int lengthOfLongestSubstring(string s) {
        int len = 0, m = 0;
        vector<int> dict(256, -1);
        for (int i = 0; i < s.size(); ++i) {
            m = max(m, dict[s[i]] + 1);
            dict[s[i]] = i;
            len = max(len, i - m + 1);
        }
        return len;
    }
};
```

### Complement

Now I've meet some point of it:

- ASSIC uses one byte to encode, $2^{8}=256$, so we can regard a 256-size vector as a hashtable. The ASSIC of character will be key, its index will be value.
- Remeber, every value in `dict` which not equals $-1$ represents the element's index clostest to $i$'th element. 

- If all the character is unique, $m$ always equals 0, longest length will increase one by one.
- If  encount a character repeates, then we compare $m$ and the index. If $m$ is greater than index, it means the element occured in a substring before, but no big deal for current substring count. If $m$ is less than index, it means the current substring can't be longer any more, so $m$ will update to the index.
- In other word, every time we update $m$ , we actually find a substring, which is `string[m: i]`.
- As for varaible `len`, it compares every loop. Only when current substring `i - m + 1`longer than before, it will update to a large length.

I think the partern of solution can have a double pointer expension.