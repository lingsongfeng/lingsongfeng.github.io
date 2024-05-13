---
layout: post
title:  "LeetCode 快慢指针模版"
date:   2024-05-13 19:37:00 +0800
categories: algorithm
---

如何找到某一条链表的中点？

我们可以完整遍历一遍链表，计算出链表的长度，然后再遍历该长度除以 2 ，即可获取到链表的中点。但这种方式需要遍历两次链表。

在很多题解中，出现了**快慢指针**的方法，其思路是：维护两个指针，首先都从链表的头部开始。慢指针一次走一个节点，快指针一次走两个节点。当快指针走到结尾时，慢指针自然很自然就是中点了。

## 常见的快慢指针模版（个人不推荐）

以 LeetCode 148 排序链表为例，若使用归并排序的方式排序链表，则需要获取链表的中点。官方题解实现的快慢指针逻辑如下：

```c++
...
ListNode* slow = head, *fast = head;
while (fast != tail) {
    slow = slow->next;
    fast = fast->next;
    if (fast != tail) {
        fast = fast->next;
    }
}
...
```

官方题解的实现逻辑中，在`while`循环内部有额外的判断，需要额外判定快指针是否到达了尾部。**这种判定方式显得不是很简洁。**

## 更通俗易懂的快慢指针模版（推荐）

其实，我们可以省去多余的判断步骤，直接判断 `fast`指针能否一次跳跃两个节点。具体的代码如下：

```c++
ListNode* slow = head;
ListNode* fast = head;
while (fast->next && fast->next->next) {
  slow = slow->next;
  fast = fast->next->next;
}
```

这种快慢指针的实现方式更加通俗易懂，不容易写错。

需要注意的是：这种方式实现的快慢指针，**快指针不一定能走到最后一个节点，有可能是倒数第二个节点。**

但我们可以肯定的是：

- 快指针走过的距离，一定是慢指针走过距离的两倍。
- 快指针最后停下的位置，取决于链表长度的奇偶性。若长度为偶数，即有奇数个节点，快指针能走到末尾节点；若长度为奇数，即有偶数个节点，快指针只能走到倒数第二个节点。

## 相关题目

### LeetCode 141 环形链表

若链表中存在环，快慢指针一定会相遇。同时，由于**快慢指针的相对速度是1**，快慢指针不会出现快指针越过慢指针的情况。因此，此题只需要让快慢指针从链表头部出发，每次迭代后判断快慢指针是否相遇即可。

```c++
bool hasCycle(ListNode* head) {
    if (!head) return false;
    auto fast = head;
    auto slow = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (fast == slow) {
            return true;
        }
    }
    return false;
}
```

### LeetCode 148 排序链表

利用了归并排序的思想。由于链表无法随机访问，所以需要利用快慢指针获取到中点。

```c++
ListNode* merge(ListNode* a, ListNode* b) {
    ListNode dummy_impl;
    ListNode* dummy = &dummy_impl;
    ListNode* curr = dummy;
    while (a && b) {
        if (a->val < b->val) {
            curr->next = a; a = a->next;
        } else {
            curr->next = b; b = b->next;
        }
        curr = curr->next;
    }
    if (a) curr->next = a;
    if (b) curr->next = b;
    return dummy->next;
}
ListNode* merge_sort(ListNode* head) {
    if (!head || !head->next) {
        return head;
    }
    ListNode* slow;
    ListNode* fast;
    slow = fast = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    auto tmp = slow->next;
    slow->next = nullptr;
    auto hd1 = merge_sort(head);
    auto hd2 = merge_sort(tmp);
    return merge(hd1, hd2);
}
```

