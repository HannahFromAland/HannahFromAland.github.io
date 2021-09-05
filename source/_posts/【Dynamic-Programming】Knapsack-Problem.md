---
title: 【Dynamic Programming】Knapsack Problem
date: 2020-12-22 10:04:14
tags: Dynamic Programming
category: Algorithm
---

## Problem Description

一个背包的总容量为**W**,现在有**N**个物品且**物品不可分**,第**i**个物品的重量为**weight[i]**,价值为**val[i]**；
往该背包里装东西,怎样装才能使得最终包内物品的总价值最大。

<!-- more -->

### one-dimensional 01 Knapsack Problem

#### Two-dimensional Solution

每个物品只有两种状态：装入背包或不装入背包

定义一个二维数组`dp`存储最大价值，其中`dp[i][j]`表示前 i 件物品**重量不超过 j 的情况下**能达到的最大价值。设第 i 件物品重量为 w，价值为 v，根据第 i 件物品是否添加到背包中，可以分两种情况讨论:

- 第i件物品未装入背包：此时总重量不超过 j 的前 i 件物品的最大价值就是总重量不超过 j 的前 i-1 件物品的最大价值，即`dp[i][j]=dp[i-1][j]`；
- 第i件物品装入背包：此时`dp[i][j] = dp[i-1][j-weight[i]]+val[i]`

因此第i件物品是否添加至背包，取决于其重量与j的大小（若其重量本身超过j则不能添加）和以上两种情况中价值较大的情形

动态规划的状态转移方程为：

`dp[i][j] = max(dp[i-1][j],dp[i-1][j-weight[i]]+val[i])`

Java实现如下：

```java 

import java.util.Scanner;
public class pack {
    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        int w = in.nextInt();
        int n = in.nextInt();
        in.nextLine();
        int[] val = new int[n + 1];
        int[] weight = new int[n + 1];
        for (int i = 1; i <= n; i++) {
            /**
            *存入物品时即从1开始方便dp数组的计算，
            *即不需要调整dp参数或初始化二维dp数组坐标取0的情况
            **/
            weight[i] = in.nextInt();
            val[i] = in.nextInt();
        }
        in.close();
        knapsack(w, n, val, weight);
    }
public static void knapsack(int w, int n, int[] weights, int[] values) {
    int[][] dp = new int[n + 1][w + 1];
    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= w; j++) {
            if (j >= w) {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weright[i]] + val[i]);
            } else {
                dp[i][j] = dp[i - 1][j];
            }
        }
    }
    System.out.print(dp[n][w]);
	}
}
```

发现算例有一半会出现`memory limit exceeded` :cry:，二维数组空间复杂度为$O(nw)$，可以进行空间优化

#### One-dimensional Solution (Space optimization)

观察状态转移方程可以知道，前 i 件物品的状态仅与前 i-1 件物品的状态有关，因此可以将 dp 定义为一维数组，其中`dp[j]` 表示按顺序对物品进行判断时不大于重量j的当前最大价值。

状态转移方程为：`dp[j] = max(dp[j],dp[j-weight[i]]+val[i])`

**Notice:**由于一维dp数组中相当于省略了当前进行判断的物品顺序值，但应该注意的是在进行`dp[j]`判断时，比较的两种可能的`dp[j]`取值均是在i-1物品下的状态，因此如果从小到大顺序对j进行遍历会导致dp数组在`dp[j]`时存储的`dp[j-weight[i]]`实际上已经是加入第i个物品后的状态（即由于先遍历`dp[j-weight[i]]`导致`dp[i-1][j-weight[i]]`已经被`dp[i][j-weight[i]]`覆盖了），因此要对j进行从大到小的倒序遍历

Java实现：

```java
public static void knapsack(int w, int n, int[] val, int[] weight){
        int[] dp = new int[w+1];
        for(int i=1;i<=n;i++){
            for(int j=w;j>0;j--){
                if(j>=weight[i]){
                    dp[j]= Math.max(dp[j],dp[j-weight[i]]+val[i]);
                }
            }
        }
        System.out.print(dp[w]);
    }
```

### Two dimensional knapsack problem

即在背包的重量限制下加入体积限制v，物品参数加入volumn，同样求放入背包中物品的最大价值。
解决方法同一维背包，其中原始dp为三维数组，空间优化后的dp解为二维数组

优化后的Java实现：
```java
public static void pack2d(int w, int v, int num,int[] val,int[] weight, int[] vol){
        int[][] dp = new int[w+1][v+1];
        for(int i=1;i<=num;i++){
            for(int j=w;j>0;j--){
                for(int k=v;k>0;k--){
                    if(j>=weight[i]&&k>=vol[i]){
                        dp[j][k]=Math.max(dp[j][k],dp[j-weight[i]][k-vol[i]]+val[i]);
                    }
                }
            }
        }
        System.out.print(dp[w][v]);
    }
```



### Fractional Knapsack Problem 

将0-1背包中的物品由不可分变为可分，此时的背包问题转化为对各个物品依据`性价比（即value/weight）`进行排序并塞满整个背包，利用贪心求解。

