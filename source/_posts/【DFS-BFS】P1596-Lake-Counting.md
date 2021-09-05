---
title: 【DFS&BFS】P1596 Lake Counting
date: 2020-10-31 14:11:01
tags: 
- DFS
- BFS
- Search
category: Algorithm
---
## Problem
Due to recent rains, water has pooled in various places in Farmer John's field, which is represented by a rectangle of N x M (1 <= N <= 100; 1 <= M <= 100) squares. Each square contains either water ('W') or dry land ('.'). Farmer John would like to figure out how many ponds have formed in his field. A pond is a connected set of squares with water in them, where a square is considered adjacent to all eight of its neighbors. Given a diagram of Farmer John's field, determine how many ponds he has.
由于近期的降雨，雨水汇集在农民约翰的田地不同的地方。我们用一个NxM(1<=N<=100;1<=M<=100)网格图表示。每个网格中有水('W') 或是旱地('.')。一个网格与其周围的八个网格相连，而一组相连的网格视为一个水坑。约翰想弄清楚他的田地已经形成了多少水坑。给出约翰田地的示意图，确定当中有多少水坑。

<!-- more -->

## Input
Line 1: Two space-separated integers: N and M * Lines 2..N+1: M characters per line representing one row of Farmer John's field. Each character is either 'W' or '.'. The characters do not have spaces between them.
第1行：两个空格隔开的整数：N 和 M 第2行到第N+1行：每行M个字符，每个字符是'W'或'.'，它们表示网格图中的一排。字符之间没有空格。

## Output
Line 1: The number of ponds in Farmer John's field.
一行：水坑的数量

## BFS (Using Stack)
```java
import java.util.Scanner;
public class Main {
	public static void main(String[] args) {
		Scanner input = new Scanner(System.in);
		int n = input.nextInt();
		int m = input.nextInt();
		input.nextLine();                                                 //waterpool1
		String[][] field = new String[n+1][m+1];
		for(int i=0;i<n;i++) {
			String a = input.nextLine();
			for(int j=0;j<m;j++) {
				String s = String.valueOf(a.charAt(j));           //waterpool2
				field[i][j]= s;
				}
			
		} 
		input.close();
	/*	for(int i=0;i<n;i++) {
			for(int j=0;j<m;j++) {
				System.out.print(field[i][j]);
			}
			System.out.println();
		}
	*/
		int pool =0;
		int[][] mark = new int[n+1][m+1];
		
		Stack route = new Stack();
		for(int i =0;i<n;i++) {
			for(int j= 0;j<m;j++) {
				if(mark[i][j]==0 && field[i][j].equals("W")==true) {  //waterpool3
					int[] start ={i,j};
					mark[i][j]=1;
					route.push(start);
					pool++;       //waterpool4
					while(route.is_empty()==false) {
						start = route.pop();
						//System.out.println(start[0]+" "+start[1]);
						move(route,field,start,m,n,mark);
					}
			}
				else {
					mark[i][j]=1;
				} 
			}
		}
		System.out.println(pool);
	}
	public static void move(Stack stack,String[][] field, int[] a, int m, int n ,int[][] mark) {

		int x = a[0];
		int y = a[1];
		int[] moveX = {1,1,1,0,0,-1,-1,-1};   //Hint1
		int[] moveY = {0,1,-1,1,-1,0,1,-1};
		String j = "W";

		for(int i=0;i<8;i++) {
			if(x+moveX[i]>=0 && x+moveX[i]<n 
					&& y+moveY[i]>=0 && y+moveY[i]<m
					&& mark[x+moveX[i]][y+moveY[i]]==0
					&& j.equals(field[x+moveX[i]][y+moveY[i]])) { //waterpool4
							int[] next = {x+moveX[i],y+moveY[i]};
							stack.push(next);
							mark[next[0]][next[1]]=1;
						}
			}
	}
	public static class Stack{
		//declare the volume
		public final int INITIAL_STACK_SIZE = 1000;
		public int[][] base;// store the x,y for myth need to use 2-dimensional list
		public int top;
	
	public Stack(){
        base = new int[10000][2];					//waterpool5
        top = 0;
    	}
	public void push(int[] a) {
		base[top++] = a;
	}
	public int[] get_top() {
		return base[top-1];
	}
	public int[] pop(){
		top--;
		return base[top];
		}
	public boolean is_empty() {
		if(top==0) {
			return true;
		}
		else {
			return false;
			}
		}
	}
}
```
- Waterpool1: Scanner 输入的问题

> **next()** 方法在读取内容时，会过滤掉有效字符前面的无效字符，对输入有效字符之前遇到的空格键、Tab键或Enter键等结束符，next()方法会自动将其过滤掉；只有在读取到有效字符之后，next()方法才将其后的空格键、Tab键或Enter键等视为结束符；所以next()方法不能得到带空格的字符串.   
**nextLine()** 方法返回的是Enter键之前没有被读取的所有字符，是可以得到带空格的字符串的.
**next()方法读取到空白符结束，nextLine()读取到回车结束(“\r”）.**   
Quote: [java的nextLine()的一个报错引发的思考](https://blog.csdn.net/ssjdoudou/article/details/83987504)

 Solution: 在nextInt读入两数字后再加nextLine()读取换行符，不做处理。之后就可以顺利读入地形图了.


- Waterpool2: 将field按单个字符以字符串的形式输入需要先讲nextLine读入的整行解析为char，再转为字符串

```java
1. String s = String.valueOf('c'); //效率最高的方法

2. String s = String.valueOf(new char[]{'c'}); //将一个char数组转换成String

3. String s = Character.toString('c');
// Character.toString(char)方法实际上直接返回String.valueOf(char)

4. String s = new Character('c').toString();

5. String s = "" + 'c';
// 虽然这个方法很简单，但这是效率最低的方法
// Java中的String Object的值实际上是不可变的，是一个final的变量。
// 所以我们每次对String做出任何改变，都是初始化了一个全新的String Object并将原来的变量指向了这个新String。
// 而Java对使用+运算符处理String相加进行了方法重载。
// 字符串直接相加连接实际上调用了如下方法：
// new StringBuilder().append("").append('c').toString();

6. String s = new String(new char[]{'c'});
```
 Quote: [Java中char和String的转换](https://blog.csdn.net/Yaokai_AssultMaster/article/details/52082763)

- Waterpool3&Waterpool4: 判定两字符串相等使用`a.equals(b)`,且调用时需保证a为常量b为变量，不然会报*java.lang.NullPointerException*.

> 在Java中由于字符串是对象类型，所以不能简单的用“==” 判断两个字符串是否相等，而使用 equals()方法比较两个对象的内容.  
 注意： equals()方法比较的是对象的内容（区分字母的大小写格式） ，但是如果使用“==”操作符比较两个对象时， 比较的是两个对象的内存地址， 所以它们不相等 （即使内容相同， 不同对象的内存地址也是不相同的）   
 Quote: [java 判断两个字符串相等](https://blog.csdn.net/yeyingss/article/details/53690098)

> 变量.equals(常量)且变量为null时，变成了null.equals("常量")，而null不属于String类型，所以无法调用String.class里面的equals方法，而常量.equals(变量)时，常量最少也是  "" ，属于String类型，所以可以调用它下面的方法.    
Quote: [String.equals报java.lang.NullPointerException](https://blog.csdn.net/Mint6/article/details/77969326)

- Waterpool5: Stack初始化时内存大小  
Stack中存储相连的“W”位置（位置信息为int[]）,因此初始化Stack为二维数组，base[i][j]中j取值为0,1两种可能，而i的最大极端情况为100\*100均为水坑，因此应为10000.

- Hint1: 向八个方向搜索时不用写8个if，将8个方位的xy坐标变化值对应存入数组使用for循环即可模拟~

## DFS (Using Recursion)

```java
import java.util.Scanner;
public class fieldDfs {
	public static String[][] field = new String[1000][1000];
	public static int[][] vis = new int[1000][1000];//在class内定义公共的数组和字符串进行调用，不然需要每次在调用dfs时存入，当算例较大时会Stack Overflow
	public static void main(String[] args) {
		Scanner input = new Scanner(System.in);
		int n = input.nextInt();
		int m = input.nextInt();
		input.nextLine();
		
		for(int i=0;i<n;i++) {     
			String a = input.nextLine();
			for(int j=0;j<m;j++) {
				String s = String.valueOf(a.charAt(j)); //char to string
				field[i][j]= s;
				}
		}
		input.close();
		int ans = 0;
		
		for(int i=0;i<n;i++) {
			for(int j=0;j<m;j++) {
				if("W".equals(field[i][j]) && vis[i][j]==0) { 
					ans++;
					dfs(n,m,i,j);
					
				}
			}
		}
		System.out.println(ans);
	}
	public static int[] moveX = {1,1,1,0,0,-1,-1,-1};
	public static int[] moveY = {0,1,-1,1,-1,0,1,-1};
	public static void dfs(int n,int m,int x, int y) {
		//System.out.println(x+" "+y);
		vis[x][y]=1;
		
		for(int i=0;i<=7;i++) {
			int new_x = x+moveX[i];
			int new_y = y+moveY[i];
			if(new_x>=0 && new_x<n 
					&& new_y>=0 
					&& new_y<m
					&&"W".equals(field[new_x][new_y]) 
					&& vis[new_x][new_y]==0){
				//System.out.println(new_x+" "+new_y);
				dfs(n,m,new_x,new_y);
			}
		}
	}
}
```

