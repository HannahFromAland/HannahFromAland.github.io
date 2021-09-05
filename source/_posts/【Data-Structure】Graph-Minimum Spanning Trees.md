---
title: 【Data Structure】Graph- Minimum Spanning Trees
date: 2020-12-02 13:58:04
tags: 
- Graph
- Tree
category: Data Structure
---



### 相关名词

- **连通图(无向图)：** 在无向图中，若任意两个顶点$V_i$与$V_j$都有路径相通，则称该无向图为连通图。
- **强连通图(有向图)：** 有向图中，若任意两个顶点$V_i$与$V_j$都有路径相通，则称该有向图为强连通图。
- **生成树：** 含有图中全部n个顶点的一个连通子图，只有足以构成一棵树的n-1条边。即若生成树中再添加一条边，则必定成环。
- **最小生成树：** 在连通网的所有生成树中，所有边的代价和最小的生成树，称为最小生成树。

<!-- more -->

### 最小生成树算法

#### Prim算法（加点法）

每次迭代选择代价最小的边对应的点，加入到最小生成树中。算法从某一个顶点s开始，最小生成树不唯一。

**输入：**邻接矩阵，使用List存放已经被加入生成树的点，sum计算总权值

使用**数组parent**存放每个节点加入生成树时的父节点，初始化为-1，根节点为0；

**输出：**sum

> 1. 起点为任意一点，首先放入list中
>
> 2. list中不包含所有节点时
>
>    1）遍历list，找出所有能连通非List中节点的最小权重的边并加入list中
>
>    2）更新list及parent数组的对应值

**实现：**

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Scanner;

public class Prim {
    public static int[][] axis = new int[1000][2];
    public static int[][] dis = new int[1000][1000];
    public static int n;
    public static void main(String[] args){
        Scanner sc= new Scanner(System.in);
        n = sc.nextInt();
        for(int i=0;i<n;i++){
            int x = sc.nextInt();
            int y = sc.nextInt();
            axis[i] = new int[]{x,y};
        }
        for(int i =0;i<n;i++){
            for(int j=0;j<n;j++){
                if(i==j){
                    dis[i][j]=-1;
                }
                else{
                    dis[i][j]= (int) Math.sqrt(Math.pow(axis[i][1]-axis[j][1],2)
                                     +Math.pow(axis[i][0]-axis[j][0],2));
                }
            }
        }
        sc.close();
        Generate();
    }
    public static void Generate(){
        
        /* 初始化List，当List中存放所有元素时说明找到最小生成树*/
        List<Integer> list = new ArrayList<>();
        list.add(0);
        /*从0节点开始判断*/
        int sum = 0;
        int[] parent = new int[n];
        for(int i=0;i<n;i++){
            parent[i]=-1; //将parent数组初始化为-1
        }
        int weight = 0;
        int begin=0, end=0;
        while(list.size()<n){
            weight = Integer.MAX_VALUE; //每次判断需要初始化weight
            for(Integer row: list){
                /*iteration of the weight:
                 *每次在list中加入新的节点后需要更新List中所有边并选出最小代价边
                 */
                for(int i=0;i<n;i++){
                    if(!list.contains(i)){
                        if(dis[row][i]>0 && dis[row][i]<weight){
                            begin =row;
                            end = i;
                            weight = dis[row][i];
                        }
                    }
                }
            }
            /*每轮List最小边判断结束后将选出的新节点加入list
             *更新其parent数组的父节点
             *加最小权重
             */
            list.add(end);
            parent[end] = begin;
            sum+= weight;
        }
        System.out.println(Arrays.toString(parent));
        System.out.println(sum);
    }
}

```



#### Kruskal 算法（加边法）

1. 将图中所有边按照权值大小排序；
2. 按权值从小到大选择边，要求所选边连接的两个顶点必须属于不同的树（利用parent记录各节点所在树的根节点，即要求两节点的根节点不相同$\rightarrow$不在同一棵树
3. 需要构造一个三元组Edge<begin,end,weight>来表示各条边，存入List中按照weight进行排序，然后依次判断是否符合2中的条件要求（即判断begin和end的根节点是否不相同，根节点通过parent数组进行储存）

**实现：**

```java
import java.util.*;

class Edge implements Comparable<Edge>{
    private int begin;
    private int end;
    private int weight;

    public Edge(int begin, int end, int weight) {
        this.begin = begin;
        this.end = end;
        this.weight = weight;
    }

    public int getBegin() {
        return begin;
    }

    public int getEnd() {
        return end;
    }

    public int getWeight() {
        return weight;
    }

    @Override 
    public int compareTo(Edge o){
      if(o.weight >this.weight){
          return -1;
      }else{
          return 1;
      }
    }
}
public class Kruskal {
    public static int[][] axis = new int[1000][2];
    public static int[][] dis = new int[1000][1000];
    public static int n;
    public static void main(String[] args){
        Scanner sc= new Scanner(System.in);
        n = sc.nextInt();
        for(int i=0;i<n;i++){
            int x = sc.nextInt();
            int y = sc.nextInt();
            axis[i] = new int[]{x,y};
        }
        for(int i =0;i<n;i++){
            for(int j=0;j<n;j++){
                if(i==j){
                    dis[i][j]=-1;
                }
                else{
                    dis[i][j]= (int) Math.sqrt(Math.pow(axis[i][1]-axis[j][1],2)+Math.pow(axis[i][0]-axis[j][0],2));
                }
            }
        }
        sc.close();
        List<Edge> list = new ArrayList<>();
        /*
         *将图中所有边加入list中进行排序，因为无向图为对称矩阵，因此只需要加入上三角/下三角即可
         */
        for(int i=0;i<n;i++){
            for(int j=i+1;j<n;j++){ 
                if(dis[i][j]>=0){
                    list.add(new Edge(i,j,dis[i][j]));

                }
            }
        }

        Collections.sort(list);//对所有边权重进行排序

        int[] parent = new int[n];
        for(int i=0;i<n;i++){
            parent[i] = -1;
        }
        int m,k;
        int sum = 0;
        for(Edge edge:list){
            /*
            * @param m,k: 寻找各边起点和终点所在集合的根节点
            * 若m=k说明在同一集合内，将舍弃该条边；
            * 否则将该条边加入最小生成树并更新权重。
            */
            m = find(edge.getBegin(),parent);
            k = find(edge.getEnd(),parent);
            if(m!=k){
                parent[k]= m;
                sum+=edge.getWeight();
            }
        }
        System.out.println(Arrays.toString(parent));
        System.out.println(sum);
    }

    public static int find(int ch,int[] parent) {
        while (parent[ch] >= 0) {
            ch = parent[ch];
        }
        return ch;
    }
}

```



