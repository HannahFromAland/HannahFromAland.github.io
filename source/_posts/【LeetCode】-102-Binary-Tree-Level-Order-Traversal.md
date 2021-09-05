---
title: 【LeetCode】 102.Binary Tree Level Order Traversal
date: 2021-03-22 18:14:41
tags: 
- BFS
- Tree
category: Algorithm
---

整理剑指 Offer 32 - I & II （主站102题），实现二叉树层次遍历。（为面试攒人品）

<!-- more -->

### [无层记录要求的32- I](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-lcof/)

- 只需要按以int[]数组按序返回层次遍历结果即可，利用队列实现

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */

 import java.util.Queue;
 import java.util.ArrayList;
 import java.util.LinkedList;

class Solution {
    public int[] levelOrder(TreeNode root) {
        ArrayList<Integer> res = new ArrayList<>();
        if(root == null) {
            return new int[0]; // 特例处理
        } else{
            Queue<TreeNode> queue = new LinkedList<>();
            queue.add(root);
            while (!queue.isEmpty()) {
                TreeNode curr = queue.poll();
                res.add(curr.val);
                if(curr.left != null){
                    queue.add(curr.left);
                }
                if(curr.right != null){
                    queue.add(curr.right);
                }
            }
            int[] d = new int[res.size()];
            for(int i=0;i<res.size();i++) {
                d[i] = res.get(i);
            }
            return d; // ArrayList遍历转数组
        }
    }
}
```

- 复杂度分析：
  - 时间复杂度 $O(N)$ ：*N* 为二叉树的节点数量
  - 空间复杂度 $O(N)$ ： 最差情况下，即当树为平衡二叉树时，最多有$N/2$个树节点**同时**在 `queue` 中，使用 $O(N)$ 大小的额外空间



### [有层数记录的32 - II ](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

- 要求返回类型为`List<List<Integer>>` ，且分层存入内部List中
- 初始想到的方法是利用flag存当前层数，其余遍历方式与I题相同，但需要在队列中同时存入树的节点和其所在层数，可以通过javafx包的Pair实现，但leetcode没有（:cold_sweat:,需要自己构造class

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
 import java.util.LinkedList;
 import java.util.Queue;
 import java.util.ArrayList;
class Pair{
    private TreeNode node;
    private int layer;

    public Pair(TreeNode node, int layer){
        this.node = node;
        this.layer = layer;
    }
    
    public TreeNode getKey() {
        return node;
    }

    public int getValue(){
        return layer;
    }

}
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> all = new ArrayList<>();
        if(root == null) {
            return new ArrayList<>();
        }
        Queue<Pair> queue = new LinkedList<>();
        queue.add(new Pair(root,1));
        int flag = 1;
        List<Integer> l = new ArrayList<>();
        while (!queue.isEmpty()) {
            Pair curr = queue.poll();
            int curr_l = curr.getValue();
            TreeNode curr_t = curr.getKey();
            if(curr_l == flag) {
                l.add(curr_t.val);
            } else{
                all.add(l);
                l = new ArrayList<>();
                l.add(curr_t.val);
                flag++;
            }
            if(curr_t.left != null) {
                queue.add(new Pair(curr_t.left, curr_l+1));
            }
            if(curr_t.right != null) {
                queue.add(new Pair(curr_t.right, curr_l+1));
            }
        }
        if(!l.isEmpty()){
            all.add(l);
        }
        return all;
    }
}
```

- 更简单的方式是直接通过对队列中的现有元素（同层）进行循环（用到`queue.size()`)保证逐层输出

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
 import java.util.ArrayList;
 import java.util.LinkedList;
 import java.util.Queue;
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if(root == null) {
            return new ArrayList<>();
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()) {
            ArrayList layer = new ArrayList<>();
            for(int i=queue.size(); i>0; i--) { //keypoint！
                TreeNode curr = queue.poll();
                layer.add(curr.val);
                if (curr.left != null) {
                    queue.add(curr.left);
                }
                if (curr.right != null ) {
                    queue.add(curr.right);
                }
            }
            res.add(layer);
        }
        return res;
    }
}
```

- 复杂度分析：
  - 时间复杂度 $O(N)$ ：*N* 为二叉树的节点数量
  - 空间复杂度 $O(N)$ ： 最差情况下，即当树为平衡二叉树时，最多有$N/2$个树节点**同时**在 `queue` 中，使用 $O(N)$ 大小的额外空间