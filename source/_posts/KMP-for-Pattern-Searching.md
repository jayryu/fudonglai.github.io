---

title: KMP for Pattern Searching
date: 2018-10-26 16:18:39
tags:
	- patern search
categories:
	- Algorithm
---

**Question:** Return the index of the first occurrence of needle in haystack, or **-1** if needle is not part of haystack.

KMP (Knuth-Morris-Pratt Algorithm) is a classic and yet notoriously hard-to-understand algorithm.Fortunately, I found some perspicuous explainations.

Here are both Brute-Force method and KMP algorithm.

<!--more-->

# Brute-Force Implement

It's easy. Code is here:

```cpp
    int strStr(string haystack, string needle) {
        if (needle.empty())
            return 0;
        if (haystack.empty())
            return -1;
        int h = haystack.length(), n = needle.length();
        for (int i = 0; i <= h - n; ++i) {
            if (haystack[i] == needle.front()){
                int j = 0;
                for (; j < n && haystack[i + j] == needle[j]; ++j)
                    ;
                if (j == n)
                    return i;
            }//if
        }//for
        return -1;
    }

```

The worst case complexity of the naive algorithm is $O(m(n-m+1))$. 

 ```cpp
 text[] = "AAAAAAAAAAAAAAAAAB"
 pattern[] = "AAAAB"
 /*the worst case*/
 
 txt[] = "ABABABCABABABCABABABC"
 pat[] =  "ABABAC" 
 /*not a worst case, but a bad case for Naive*/
 ```

But the time complexity of KMP algorithm is $O(n)$ in the worst case. Of course, you need some extra space to preprocess the pattern.

Then you  can challenge yourself implement the KMP algorithm for this problem.

# KMP Algorithm

You can refer to them for the detials:

1. KMP on [Jake Boxer's Blog](http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/)
2. KMP on [Geeks-for-Geeks](http://www.geeksforgeeks.org/searching-for-patterns-set-2-kmp-algorithm/)

## Intuition

### Basic Idea 

Whenever we detect a mismatch (after some matches), we already know some of the characters in the text of the next window. We take advantage of this information to avoid matching the characters that we know will anyway match. 

To know how many characters we can skip, we pre-process pattern and get an integer array that tells us the count of characters to be skipped. 

### The Partial Match Table

Here I want to quote [Jake Boxer's perspicuous explaination](http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/).

Here’s the partial match table for the pattern “abababca”:

|char  | a | b | a | b | a | b | c | a |
| ---- | - | - | - | - | - | - | - | - |
|index | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
|value | 0 | 0 | 1 | 2 | 3 | 4 | 0 | 1 |

> **Proper prefix**: All the characters in a string, with one or more cut off the end. “S”, “Sn”, “Sna”, and “Snap” are all the proper prefixes of “Snape”.
>
> **Proper suffix**: All the characters in a string, with one or more cut off the beginning. “agrid”, “grid”, “rid”, “id”, and “d” are all proper suffixes of “Hagrid”.
>
> Here, we’re interested in the first four characters (“abab”). We have three proper prefixes (“a”, “ab”, and “aba”) and three proper suffixes (“b”, “ab”, and “bab”). This time, “ab” is in both, and is two characters long, so cell four gets value 2.
>
> let’s also try it for cell five, which concerns “ababa”. We have four proper prefixes (“a”, “ab”, “aba”, and “abab”) and four proper suffixes (“a”, “ba”, “aba”, and “baba”). Now, we have two matches: “a” and “aba” are both proper prefixes and proper suffixes. Since “aba” is longer than “a”, it wins, and cell five gets value 3.
>
> Let’s skip ahead to cell seven (the second-to-last cell), which is concerned with the pattern “abababc”. Even without enumerating all the proper prefixes and suffixes, it should be obvious that there aren’t going to be any matches; all the suffixes will end with the letter “c”, and none of the prefixes will. Since there are no matches, cell seven gets 0.
>
>



## Implement

quote [Geeks-for-Geeks](http://www.geeksforgeeks.org/searching-for-patterns-set-2-kmp-algorithm/).

```c
lps[i] = the longest proper prefix of pat[0..i] 
		which is also a suffix of pat[0..i]. 
         
Examples of lps[] construction:

For the pattern “AAAA”, 
lps[] is [0, 1, 2, 3]

For the pattern “ABCDE”, 
lps[] is [0, 0, 0, 0, 0]

For the pattern “AABAACAABAA”, 
lps[] is [0, 1, 0, 1, 2, 0, 1, 2, 3, 4, 5]

For the pattern “AAACAAAAAC”, 
lps[] is [0, 1, 2, 0, 1, 2, 3, 3, 3, 4] 

For the pattern “AAABAAA”, 
lps[] is [0, 1, 2, 0, 1, 2, 3]
```



### Preprocessing Algorithm


```python
pat[] = "AAACAAAA"

len = 0, i  = 0.
lps[0] is always 0, we move 
to i = 1

len = 0, i  = 1.
Since pat[len] and pat[i] match, do len++, 
store it in lps[i] and do i++.
len = 1, lps[1] = 1, i = 2

len = 1, i  = 2.
Since pat[len] and pat[i] match, do len++, 
store it in lps[i] and do i++.
len = 2, lps[2] = 2, i = 3

len = 2, i  = 3.
Since pat[len] and pat[i] do not match, and len > 0, 
set len = lps[len-1] = lps[1] = 1

len = 1, i  = 3.
Since pat[len] and pat[i] do not match and len > 0, 
len = lps[len-1] = lps[0] = 0

len = 0, i  = 3.
Since pat[len] and pat[i] do not match and len = 0, 
Set lps[3] = 0 and i = 4.

len = 0, i  = 4.
Since pat[len] and pat[i] match, do len++, 
store it in lps[i] and do i++.
len = 1, lps[4] = 1, i = 5

len = 1, i  = 5.
Since pat[len] and pat[i] match, do len++, 
store it in lps[i] and do i++.
len = 2, lps[5] = 2, i = 6

len = 2, i  = 6.
Since pat[len] and pat[i] match, do len++, 
store it in lps[i] and do i++.
len = 3, lps[6] = 3, i = 7

len = 3, i  = 7.
Since pat[len] and pat[i] do not match and len > 0,
set len = lps[len-1] = lps[2] = 2

len = 2, i  = 7.
Since pat[len] and pat[i] match, do len++, 
store it in lps[i] and do i++.
len = 3, lps[7] = 3, i = 8

We stop here as we have constructed the whole lps[].
```

```cpp
// Here is my preprocessing code:
vector<int> generator(string pattern){
    vector<int> lps(pattern.size());
    lps[0] = 0;
    int len = 0;
    for (int i = 1; i < pattern.size(); ) {
        if (pattern[i] == pattern[len]){
            len++;
            lps[i] = len;
            i++;
        } else {
            if (len > 0)
                len = lps[len - 1];
            else{
                len = 0; i++;
            }//else
        }//else
    }//for
    return lps;
}
```



### Matching Algorithm

```cpp
int strStr(string haystack, string needle) {
    if (needle.empty())
        return 0;
    if (haystack.empty())
        return -1;
    vector<int> lps = generator(needle);
    int j = 0;
    for (int i = 0; i < haystack.size(); ) {
        if (haystack[i] == needle[j]){
            i++, j++;
        } else {
            if (j == 0)
                i++;	
            else
                j = lps[j - 1];
        }
        if (j == needle.size()){
            return i - j;}
    }//for
    return -1;
}
```

