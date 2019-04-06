---
title: 【LeetCode】3-Longest-Substring-Without-Repeating-Characters
date: 2015-01-28 01:48:15
categories: LeetCode
tags: [算法,LeetCode]
---
此题第一次一遍AC。
<!--more-->

# 题目要求
Given a string, find the length of the longest substring without repeating characters. For example, the longest substring without repeating letters for "abcabcbb" is "abc", which the length is 3. For "bbbbb" the longest substring is "b", with the length of 1.

# 思路
用**两个指针**，一个表示**头**，一个表示**尾**。
再用一个**哈希表**存储这两头中间的字符，可以有两种不同的记法：
* `<char, int>` 表示当前串中有的*char*对应的*position*。
* `<char, bool>` 表示当前串中存在的*char*。
当然不一定需要使用`map`，用个`array`可以加快点效率，毕竟这里测试的`char`可以直接对应0~255。

然后大循环中**头**每次往前移动一格，并判断新增的一个`char`是否在保存的表中，如果是的话就将**尾**往前移直到找到了新加的`char`。
注意每次**尾**移动记得更新表中的数据。

# 实现
```C++
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        map<char, int> hash_map;
        int total_length = s.size();
        int max_length = 0;
        int temp_length = 0;
        int find_pos = 0;
        int right_pos = 0;
        int left_pos = 0;
        for (; right_pos < total_length; right_pos++) {
            if (hash_map.find(s[right_pos]) != hash_map.end()) {
                find_pos = hash_map.find(s[right_pos])->second;
                while(left_pos <= find_pos) {
                    hash_map.erase(hash_map.find(s[left_pos]));
                    left_pos++;
                }
            }
            hash_map.insert(make_pair(s[right_pos], right_pos));
            temp_length = right_pos - left_pos + 1;
            max_length = max(max_length, temp_length);
        }
        return max_length;
    }
};
```
这里效率较低用了**122ms**，如果用`array`的话肯定会快点。
