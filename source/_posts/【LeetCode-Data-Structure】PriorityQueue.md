---
title: 【LeetCode & Data Structure】PriorityQueue
date: 2021-09-08 21:17:15
tags: 
- LinkedList
category: Data Structure
---

每日一题之重写比较器，是九月遇到的第一个`Hard`题目，用`PriorityQueue`实现可以事半功倍！:star2:

题目链接：[502.IPO](https://leetcode-cn.com/problems/ipo/)

<!-- more -->

### （伪）题解

- 看到利润和资本首先想到的是[背包问题](https://hannahfromaland.github.io/2020/12/22/%E3%80%90Dynamic-Programming%E3%80%91Knapsack-Problem/)，然而题目给出了要求投资的项目个数，进而放弃背包转为**贪心**思路
- 该题特殊之处在于对资本和利润的处理，在完成项目后可以获得纯利润，同时利润也可以增加在资本中作为启动之后项目的资本（因此可以实现一个滚雪球
- 每个项目最多被选择一次（如果使用朴素数组实现的话需要记录其是否被选过，但`PriorityQueue`可以直接将选过的元素抛出，从另一方面完美符合要求

具体思路：
- 构建二元数组实现capital的排序（为了使profits能够与之对应进行维护），需要自定义`Comparator`，实现按照项目所需资本升序排列

```java
 Arrays.sort(pack, new Comparator<int[]>() {
        @Override
        public int compare(int[] o1, int[] o2) {
            return o1[0] - o2[0]; // capital asc           
        }
        });
```

- 该题中选取资本采用的是“滚雪球”方式，需要保证每个项目均为当前没有被启动过的项目里符合资本要求且**利润最大**的，可以通过PriorityQueue+自定义Comparator实现。

  > 引用[廖雪峰老师的使用PriorityQueue](https://www.liaoxuefeng.com/wiki/1252599548343744/1265120632401152): `PriorityQueue`和`Queue`的区别在于，它的出队顺序与元素的优先级有关，对`PriorityQueue`调用`remove()`或`poll()`方法，返回的总是优先级最高的元素。要使用`PriorityQueue`，我们就必须给每个元素定义“优先级”。

  在本题中，“优先级”为所有当前资本条件可以启动的项目的利润，因此利用priorityqueue可以实现每次均返回利润最大的可启动项目，且启动过的项目会退出队列，不用继续参与比较。

### PriorityQueue的基本操作

- `add() & offer()`：都是向优先队列中插入元素，只是`Queue`接口规定二者对插入失败时的处理不同，前者在插入失败时抛出异常，后者则会返回`false`；
- `element()`和`peek()`：都是获取但不删除队首元素，也就是队列中权值最小的那个元素，二者唯一的区别是当方法失败时前者抛出异常，后者返回`null`
- `remove()`和`poll()`：获取并删除队首元素，区别是当方法失败时前者抛出异常，后者返回`null`
- 重写比较器：类似于Array中的Comparator重写

```java
PriorityQueue<Integer> pq = new PriorityQueue<>(new Comparator<Integer>(){
            @Override
            public int compare (Integer o1, Integer o2) {
                return o2 - o1; //实现倒序排列
            }
        });
```



### 题目代码实现

```java
class Solution {
    public static int n;
    public static int[][] pack;
    public int findMaximizedCapital(int k, int w, int[] profits, int[] capital) {
        n = profits.length;
        pack = new int[capital.length][2];    
        for (int i = 0; i < n; i++) {
            pack[i][0] = capital[i];
            pack[i][1] = profits[i];
        }
        // redefine the comparator
        Arrays.sort(pack, new Comparator<int[]>() {
        @Override
        public int compare(int[] o1, int[] o2) {
            return o1[0] - o2[0]; // capital asc           
        }
        });
        PriorityQueue<Integer> pq = new PriorityQueue<>(new Comparator<Integer>(){
            @Override
            public int compare (Integer o1, Integer o2) {
                return o2 - o1;
            }
        });
        int curr = 0; //维护当前在pack中遍历到的节点索引
        while (k > 0) {
            while (curr < n && pack[curr][0] <= w) {
                pq.add(pack[curr][1]);
                curr++;
            }
            if (!pq.isEmpty()) {
                w += pq.poll();
            } else {
                break;
            }
            k--;
        }
        return w;
    }  
}
```



开学前最后一更，希望开学也能保持健康的生活习惯和勤劳的更新频率(不这不是一个flag):crying_cat_face:
