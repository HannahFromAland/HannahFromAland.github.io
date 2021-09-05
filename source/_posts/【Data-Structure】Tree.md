---
title: 【Data Structure】Tree
date: 2020-11-21 10:33:30
tags: 
- Tree
category: Data Structure
---

## 定义及名词解释

### 树

**1.节点的度：**

- 树的度是各节点度的最大值

- 节点的度是节点拥有的子树个数
	
	- 树中的叶子结点的个数计算方法：
	
	  若一棵度为4的树中度为1、2、3、4的节点的个数分别为4、3、2、2，则该树叶子节点的个数是多少？总节点个数是多少？
	
	  设树T中的结点个数为n，度为0的结点的个数为n0，度为1的结点的个数为n1，度为2的结点的个数为n2，度为3的结点的个数为n3，度为4的结点的个数为n4，则有：
	
	  **n = n0 + n1 + n2 + n3 + n4**
	
	  设树T中的总边数为e，则该树节点的**总入度 = 总出度 = 总边数**
	
	  因为除了根节点的入度为0，其余各节点的入度都为1，则有：
	
	  **e = n0 + n1 + n2 + n3 + n4 - 1**
	
	  又因为，n0的出度为0，n1的出度为1，n2的出度为2，n3的出度为3，n4的出度为4，所以：
	
	  **e = n0 * 0 + n1 * 1+ n2 * 2 + n3 * 3 + n4 * 4**
	  
	  代入即可得n0的值.
	

**2. 树的深度：** 树的层数

<!-- more -->

### 二叉树

1. 在二叉树的第 i 层上至多有 $ 2^{i-1}$个结点；[证明用归纳法]
2. 深度为 k 的二叉树至多有 $2^k -1$个结点；[利用3证明]
3. 对任何一棵二叉树T，如果其终端结点数为n0，度为2的结点数为n2，则**n0=n2+1**；[利用1中公式]

> 若一棵二叉树具有10个度为2的节点，5个度为1的节点，则度为0的节点个数为 <u>11</u>. 

4. **满二叉树**：深度为k，节点数为$ 2^k-1$;
5. **完全二叉树**：深度为k的，有n个结点的二叉树，当且仅当其每一个结点都与深度为k的满二叉树中编号从1到n的结点一一对应时，称之为完全二叉树；

- 完全二叉树性质：
  - 若`i=1`, 则结点i为二叉树的根，无双亲；若`i>1`，则其双亲结点为`i/2`；
  - 如果`2i>n`，则结点i无左孩子（在完全二叉树中若无左孩子则不可能有右孩子），即结点i为叶结点；否则其左孩子为结点`2i`;
  - 如果`2i+1>n`，则结点i无右孩子，否则其右孩子为结点`2i+1`.

- 判断是否为完全二叉树：

    设完全二叉树深度为h，除第h层外，其余任一层节点数均达到最大，且第h层节点从左至右连续；
    

> **判断依据如下：**
>
> 若存在节点左子树为空而右子树不为空，则该树一定不是完全二叉树；
>
> 若存在节点仅有左子树或左右子树均为空，则该节点所在层不能出现含有左右子树的节点（该层为最后一层）；
>
> 若当前节点的左右子树均不为空，则判断下一节点.



~~~java
public class TreeNode {
    //generate TreeNode 
	public String data = "";
	public TreeNode lchild = null;
	public TreeNode rchild = null;
		public static Boolean is_complete(TreeNode node) {
		if(node == null) {
			return true;
		}
			Queue<TreeNode> q =  new ArrayDeque<>();
			Boolean leaf = false;
			q.add(node);
			while(!q.isEmpty()) {
				TreeNode front = q.poll();
				if(front.lchild == null) {
					if(front.rchild != null) return false;
					if(front.rchild == null) leaf =true;
				}
				else {
					if(leaf) return false;
					
					if(front.rchild != null) {
						q.add(front.lchild);
						q.add(front.rchild);
					}
					if(front.rchild == null) {
						leaf = true;
						q.add(front.lchild);
					}
				}
			}
			return true;
	}
}
~~~
6. 具有n个结点的完全二叉树深度为$log_2n$+1

   证明：假设深度为k，则有$2^{k-1}-1 < n \leqslant 2^k-1$

   取对数得 $k-1\leqslant log_2n < k$, 因为k是整数，所以为$log_2n+1$


> **一棵完全二叉树中有1001个节点，其中叶子节点的个数是:**
>
> 假设完全二叉树为k层，可拆分为深度为k-1的满二叉树+第k层的叶节点
>
> 前k-1层共有节点 $2^k-1$, 本题中只能为511；
>
> 说明第k层共有节点490个，对应k-1层的双亲节点个数为245个，则k-1层叶子节点个数为11个
>
> 该树共有叶节点11+490=501个

### 二叉树的链表存储

- 二叉树的链表表示：定义`Node`

```java
public class Tree {
    private class Node{
        public int data;
        public Node lchild;
        public Node rchild;

        public Node(){
            data = 0;
            lchild = rchild = null;
        }
        public Node(int a, Node l, Node r){
            data = a;
            lchild = l;
            rchild = r;
        }
    }
}
```

- 树的二叉链表表示法：将二叉链表中的存储节点改为孩子和兄弟，其中二叉链表的左节点为孩子，右节点为兄弟

  二叉链表表示法同样适用于森林，即此时根节点同样有其他根节点作为兄弟即可

### 二叉树的遍历

#### Recursive实现

给定根节点`root`，返回遍历顺序的结点字符串

- 先序遍历：

  实现过程：在每个节点均为先遍历该节点，然后先序遍历左子树，最后先序遍历右子树

```java
public static String preOrder(Node root){
    if (root == null){
        return "";
    }else{
        String str = root.data;
        str += preOrder(root.left);
        str += preOrder(root.right);
    }
}
```

- 中序遍历：

  实现过程：每个节点先中序遍历左子树，再访问该节点，最后中序遍历右子树

```java
public static String inOrder(Node root){
    if (root == null){
        return "";
    }else{
        String str = inOrder(root.left);
        str += root.data;
        str += inOrder(root.right);
    }
}
```

- 后序遍历：

  实现过程：每个节点先对左子树进行后序遍历，然后对右子树进行后序遍历，最后访问该节点

```java
public static String postOrder(Node root){
    if (root == null){
        return "";
    }else{
        String str = postOrder(root.lchild);
        str += postOrder(root.rchild);
        str += root.data;
    }
}
```



#### Iterative实现

利用栈对遍历的节点进行维护，利用`ArrayList`按遍历顺序存储节点

- 先序遍历：首先将根节点入栈；当栈不为空时弹出栈顶元素，将其加入`ArrayList`，并在Stack中放入其右孩子和左孩子，最终返回List.

```java
public static String preOrder(Node root) {
        String str = "";
        Stack<Node> stack=new Stack<>();
        if(root==null) //如果为空树则返回
            return str;
       	stack.push(root);
        while(!stack.isEmpty()){
            Node temp=stack.pop(); 
            if(temp!=null){
                str += temp.val;//访问根节点
                stack.push(temp.rchild);
                stack.push(temp.lchild);
            }
        }
        return str;
    }
```

- 中序遍历：利用指针p对树中各个节点进行判断
  - 若p不为空则将p入栈，并使p指向其左孩子；
  - 若p为空且栈不为空则将栈顶元素出栈进行遍历，并使p指向其右孩子；
  - 若p为空且栈为空则遍历结束

```java
public static String inOrder(Node root){
    String str = "";
    Stack<Node> stack = new Stack<>();
    Node p = root;
    while(p != null || stack.isEmpty() == false){
            if(p != null){
                stack.push(p);
                p = p.lchild;
            }
            else{
                p = stack.pop();
                str += p.data;
                p = p.rchild;
            }
        }
        return str;
}
```

- 后序遍历：由于后序遍历的返回顺序为`左->右->中`，而先序遍历的返回顺序为`中->左->右`，可以将先序遍历中左右孩子的入栈顺序进行改变后得到`中->右->左`序列，再反向输出即可

```java
public static String postOrder(Node root) {
        String str = "";
        Stack<Node> stack=new Stack<>();
        if(root==null) //如果为空树则返回
            return str;
       	stack.push(root);
        while(!stack.isEmpty()){
            Node temp=stack.pop(); 
            if(temp!=null){
                str += temp.val;//访问根节点
                stack.push(temp.lchild);
                stack.push(temp.rchild);
            }
        }
        return new 
            StringBuffer(str).reverse().toString();
    }
```

- 层次遍历：直接按字符串返回可以使用队列实现

[LeetCode 102](https://leetcode.com/problems/binary-tree-level-order-traversal)

Given binary tree `[3,9,20,null,null,15,7]`,

```
    3
   / \
  9  20
    /  \
   15   7
```

return its level order traversal as:

```
[[3], [9,20], [15,7]]
```

- 将各节点和层数加入队列，可以直接使用Pair实现

  `import javafx.util.Pair;`
  
- LeetCode中没有这个包，可以手动新建Pair类实现

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

class Pair {
    private TreeNode r;
    private int c;

    public Pair(TreeNode r, int c) {
        this.r = r;
        this.c = c;
    }

    public TreeNode getKey() {
        return r;
    }

    public int getValue() {
        return c;
    }
}

class Solution {

    public List<List<Integer>> levelOrder(TreeNode root) {
        Queue<Pair> queue = new LinkedList<>();
        List<List<Integer>> res = new ArrayList<>();
        if(root == null) return new ArrayList<>();
        else{
            queue.add(new Pair(root,1));
        } 
        int currLayer = 1;
        List<Integer> layer = new ArrayList<>();
         while(!queue.isEmpty()){
            Pair curr = queue.poll();
            TreeNode node = curr.getKey();
            int la = curr.getValue();
            if(la > currLayer){
                res.add(layer);
                currLayer++;
                layer = new ArrayList<>();
                layer.add(node.val);
            }else{
                layer.add(node.val);
            }
            if(node.left != null){
                queue.add(new Pair(node.left,la+1));
            }
            if(node.right != null){
                queue.add(new Pair(node.right,la+1));
            } 
        }
        res.add(layer);
        return res;
    }
}
```

