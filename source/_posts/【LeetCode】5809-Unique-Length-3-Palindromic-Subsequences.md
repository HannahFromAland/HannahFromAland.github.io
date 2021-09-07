---
title: 【LeetCode】5809 Unique Length-3 Palindromic Subsequences
date: 2021-07-11 21:31:00
tags:
- Prefix
category: Algorithm
---

整理周赛中超时的题目:cry:，核心是需要用到26个字母对暴力法进行剪枝~

题目地址：[长度为 3 的不同回文子序列](https://leetcode-cn.com/problems/unique-length-3-palindromic-subsequences/)

<!-- more -->

## 首先想到的暴力解法



对字符串中所有元素进行遍历，寻找与其相同的回文串尾部（即第三个字符），之后对头尾之间的元素进行判断加入回文字串的set，最后返回set大小即可。

- 时间复杂度：为 $O(N^2)$,因此会超时 （具体傻瓜代码就不放出来影响大家阅读体验了🤣）

 



## 对前后缀进行优化

由于前后缀只可能出现26个小写字母，因此对所有出现次数超过两次（或恰好两次）的字母进行维护，再返回字符串中对头尾元素之间能出现的不同种类回文串进行判定即可。



```java
class Solution {

  public int countPalindromicSubsequence(String s) {

​    if(s == null || s.length()<3) {

​      return 0;

​    }

​    int[] alpha = new int[26];

​    char[] arrs = s.toCharArray();

​    for(char arr:arrs) {

​      alpha[arr-'a']++;

​    }

​    Set<String> set = new HashSet<>();

​    for(int i = 0 ;i < 26; i++) {

​      if(alpha[i] < 2){

​        continue;

​      } else {

​        int first = s.indexOf('a'+i);

​        int last = s.indexOf('a'+i,first+1);

​        while(last != -1) {

​          String curr = Character.toString('a'+i);

​          if(last-first ==1) {

​            last = s.indexOf('a'+i,last+1);

​          }else {

​            for(int j = first+1; j<last ;j++){

​            String fin = curr + s.substring(j,j+1)+ curr;

​            set.add(fin);

​        }

​        last = s.indexOf('a'+i,last+1);

​          }

​          

​        }

​        

​      }

​    }

​    return set.size();

  }

}

```



## 通过对回文子串的构造本质进行优化



- 方法2虽然时间复杂度比暴力解法优化了，但还是会出现超时的算例。。后来发现对于出现次数大于2的字母不需要逐个进行判断，只需要取最头和**最尾**的字符就可以包括所有可能的回文字串了（也就是成功优化掉了子结构里的while循环），最终的ac结果也十分喜人 XD

![image.png](https://i.loli.net/2021/07/11/jqlX2CaTKL3OcVY.png)

- 用到`java`中和`indexOf`相对的`lastIndexOf`方法



```java

class Solution {

  public int countPalindromicSubsequence(String s) {

​    if(s == null || s.length()<3) {

​      return 0;

​    }

​    int[] alpha = new int[26];

​    char[] arrs = s.toCharArray();

​    //构造26个字母的出现次数存储结构

​    for(char arr:arrs) {

​      alpha[arr-'a']++; 

​    }

​    Set<String> set = new HashSet<>();

​    for(int i = 0 ;i < 26; i++) {

​      if(alpha[i] < 2){

​        continue;

​      } else {

​        int first = s.indexOf('a'+i); //判断头元素位置

​        int last = s.lastIndexOf('a'+i); // 判断尾元素位置

​          String curr = Character.toString('a'+i);

​          for(int j = first+1; j<last ;j++){

​            String fin = curr + s.substring(j,j+1)+ curr;

​            set.add(fin);

​            }    

​        }

​      }

​    return set.size();

  }

}

```
