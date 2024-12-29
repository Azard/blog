---
title: 【LeetCode】2-Add-Two-Numbers
date: 2015-01-26 4:09:00
categories: LeetCode
tags: [算法,LeetCode]
---
很快完成了LeetCode的第二题。
<!-- more -->

# 题目要求
You are given two linked lists representing two non-negative numbers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8

# 思路
题目比较简单，有两种方法。
* 其一，可以先读第一个链表，再读第二个链表，得到10进制数后再相加，最后再重新生成链表。这里有个问题就是存储位数的限制，我用了`uint64_t`可以通过全部测试，方法比较暴力。
* 其二，可以一位一位的读，边读边生成返回的链表，这样就没有位数的限制。

# 暴力的方法
这里我的实现也不是很优化，比较多的冗余，还有很多小细节可以做。
但毕竟不是为了跑分，只是实现一个**正确的算法**。
耗时**61ms**。

```C++
ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
    uint64_t v1 = 0;
    uint64_t p1 = 1;
    uint64_t v2 = 0;
    uint64_t p2 = 1;
    while(l1) {
        v1 += p1 * l1->val;
        p1 *= 10;
        l1 = l1->next;
    }
    while(l2) {
        v2 += p2 * l2->val;
        p2 *= 10;
        l2 = l2->next;
    }
    uint64_t sum = v1 + v2;
    ListNode* root = new ListNode(0);
    ListNode* now = root;
    while(true) {
        now->val = sum % 10;
        sum /= 10;
        if (sum == 0) {
            return root;
        }
        ListNode* temp = new ListNode(0);
        now->next = temp;
        now = temp;
    }
}
```
很简洁明了没什么多说的，在跑的第一次我用的`int`然后有个case溢出了。
随后我全部替换成了`uint64_t`之后AC。

# 比较机智的方法
一位一位的去读，需要注意的是**进位**的情况。
耗时**57ms**。

```C++
ListNode *addTwoNumbers(ListNode *l1, ListNode *l2) {
    ListNode* root = new ListNode(0);
    ListNode* now = root;
    bool carry_flag = false;
    bool l1_end = false;
    bool l2_end = false;
    int temp = 0;

    while(true) {
        temp = (carry_flag ? 1 : 0) + (l1_end ? 0 : l1->val) + (l2_end ? 0 : l2->val);
        carry_flag = (temp >= 10);
        now->val = temp % 10;
        if (!l1_end) {
            l1 = l1->next;
            l1_end = (l1 == NULL);
        }
        if (!l2_end) {
            l2 = l2->next;
            l2_end = (l2 == NULL);
        }
        if (l1_end && l2_end) {
            if (carry_flag) {
                ListNode* last_node = new ListNode(1);
                now->next = last_node;
            }
            return root;
        }
        ListNode* new_node = new ListNode(0);
        now->next = new_node;
        now = new_node;
    }
    return root;
}
```
