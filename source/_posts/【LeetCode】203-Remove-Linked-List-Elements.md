---
title: 【LeetCode】203 Remove Linked List Elements
date: 2021-06-10 10:12:29
tags: 
- LinkedList
category: Algorithm
---

好久不见！学习博主即将开始认真营业（但愿），从做好一道小题开始重新拯救学习状态 :smiley_cat:

题目地址：[移除链表元素](https://leetcode-cn.com/problems/remove-linked-list-elements/)

## 需要很多特判的暴利解法
是最直接能够想到的方法，核心是遍历思想，对头结点和尾结点进行了特判。
* 判断的是指针的next元素是否需要删除，因此开始时需要特判头结点，结束时需要在倒数第二个元素停止
* 对头节点`while`循环删除，但之后仍然需要判断剩余链表是否为单节点（无法进入循环）

* 时间复杂度：`O(N)`
* 空间复杂度：`O(1)`


```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        if(head == null) {
            return head;
        }
        while(head.val == val){
            if(head.next != null){
                head = head.next;
            } else {
                return null;
            }
        }
        if(head.next == null) {
            if(head.val == val) {
                return null;
            }else {
                return head;
            }
        }
        ListNode p = head;
        while(p != null && p.next != null && p.next.next != null) {
            if(p.next.val == val) {
                ListNode q = p.next;
                p.next = q.next;
            } else {
                p = p.next;
            }
        }
        if(p.next.val == val){
            p.next = null;
        }
        return head;
    }
}
```




## 优化之后的迭代法
* 对头结点的特判可以通过加“哨兵”节点解决
* 发现也不需要对尾结点进行特判了...我第一个方法是在写什么orz

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        if(head == null) {
            return head;
        }
        ListNode sentinel = new ListNode(-1,head);
        ListNode p = sentinel;
        while(p.next != null) {
            if(p.next.val == val) {
                p.next = p.next.next;
            }else {
                p = p.next;
            }
        }
        return sentinel.next;
    }
}
```

***

## 递归
核心思想是持续处理子链表的头结点，如果头结点`val`值相等则删除自身
* 利用`head.next`进行递归成功解决单链表**没有上一节点元素**的问题，总之就是，很神奇...

```java
class Solution {
    public ListNode removeElements(ListNode head, int val) {
        if(head == null) {
            return head;
        }
        head.next = removeElements(head.next,val);
        if(head.val == val){
            return head.next;
        }else {
            return head;
        }
    }
}
```
