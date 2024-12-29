---
title: 【LeetCode】1-Two-Sum
date: 2015-01-25 22:40:31
categories: LeetCode
tags: [算法,LeetCode]
---
好久没写点有技术含量的东西了，都是心态太浮躁闹的。准备开个**LeetCode系列**，希望能坚持下去。
[LeetCode](https://oj.leetcode.com/)是一个刷算法题的网站，据说很适合准备面试，它有个很好的地方是不用考虑输入输出。而且这网站现在两个主要维护人员一个是我们院前几年毕业的学姐，现在facebook工作，之前几天和老师吃饭的时候还聊到过这个事。
假期时间比较多，除了准备些语言考试，简历等杂事，我就刷一刷题，提高下面试的竞争力。
<!-- more -->

# 题目要求
Given an array of integers, find two numbers such that they add up to a specific target number.

The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.

You may assume that each input would have exactly one solution.

Input: numbers={2, 7, 11, 15}, target=9
Output: index1=1, index2=2

# 思路
这是LeetCode的第一题，难度按系统定义是中等。
题目要求给定一串数，找到两个和为给定的*target*的数字的位置，且**一定有唯一解**。
可以用排序后遍历，以及哈希表映射*value*和*position*两种方法来解决。

# 排序遍历
这方法似曾相识，似乎算法课做过类似的。
简单的说，就是先对数组排序一遍，然后*begin*和*end*元素相加，看看和与*target*的大小关系，如果太大了*end*就往前移，太小*begin*就往后移。

算法复杂度scan的话是`O(n)`的，所以就是排序的复杂度。我只用了`O(nlogn)`的排序，还没尝试用`O(n)`的排序，系统会不会说占用内存过大？( ⊙ o ⊙ )

```C++
vector<int> twoSum(vector<int> &numbers, int target) {
    vector<int> copy_numbers = numbers;
    vector<int> ret;
    ret.resize(2);
    sort(copy_numbers.begin(), copy_numbers.end());
    int left = 0;
    int right = copy_numbers.size()-1;
    int sum = 0;
    while(left < right) {
        sum = copy_numbers[left] + copy_numbers[right];
        if (sum < target) {
            left++;
            continue;
        }
        if (sum > target) {
            right--;
            continue;
        }
        // find sum is target
        int find_count = 0;
        for (int i = 0; i < numbers.size(); i++) {
            if (numbers[i] == copy_numbers[left] || numbers[i] == copy_numbers[right] && find_count < 2) {
                ret[find_count] = i+1;
                find_count++;
                if (find_count >= 2) {
                    return ret;
                }
            }
        }
        break;
    }
    return ret;
}
```
一开始我手写的快排竟然超时了，也许是因为vector内部的值换来换去效率太低了吧。
后来发现LeetCode刷题的话排序直接调用**std**的`sort()`就OK。

这里稍微有几个case被坑，一个是
Input:	[0,4,3,0], 0
Output:	4, 4
Expected:	1, 4
因为我开始的算法是先找到数字，再去遍历找位置，没有考虑**重复元素**的情况。

还有个被坑的case是输出的前后顺序问题，我一开始认为是把value小的position放在第一个输出，后来发现应该还是按照**原先position的大小顺序**进行输出，这些都是小细节问题。
Input:	[5,75,25], 100
Output:	3, 2
Expected:	2, 3

那么这个就顺利通过了，实现是**18ms**。
LeetCode还会有所有人耗时分布图，**18ms**从图上看已经超越了C++完成的90%的人了。

#哈希表
因为题目的tags中有**hashmap**一项，因此也可以用**hashmap**做个`O(n)`的解法。

思路比较简单，写成代码表示就是`map<value, position>`，然后遍历*map*找是否有*key*为`target-this.key`的项存在。

这里有个问题，因为开始我们发现有个case包含重复元素，那么怎么读入到哈希表中呢？
解决方法很简单，**在读入的时候可以先判断是否已经存在相同的key的项**，由于有唯一解，如果有重复元素的话只有两种情况：
* 它们就是要找的元素，那么`2*value==key`，可以在读入到*map*的时候直接返回。
* 它们肯定不是要找的元素，如果是的话就存在多个解，直接排除。

基于上述第二种情况甚至可以在发现重复，并且确定不是解的时候删除原来的项。

```C++
vector<int> twoSum(vector<int> &numbers, int target) {
    map<int, int> hash_map;
    vector<int> ret;
    for (int i = 0; i < numbers.size(); i++) {
        if (hash_map.find(numbers[i]) != hash_map.end()) {
            if (2*numbers[i] == target) {
                ret.push_back(hash_map[numbers[i]]);
                ret.push_back(i+1);
                return ret;
            }
            continue;
        }
        hash_map.insert(make_pair(numbers[i], i+1));
    }
    for (auto it : hash_map) {
        if (hash_map.find(target - it.first) != hash_map.end()) {
            ret.push_back(it.second);
            ret.push_back(hash_map[target-it.first]);
            if (it.second > hash_map[target-it.first]) {
                ret[0] = hash_map[target-it.first];
                ret[1] = it.second;
            }
            return ret;
        }
    }
    return ret;
}
```

最终用时是**48ms**，我猜想主要原因还是哈希表的操作太费时间了。
