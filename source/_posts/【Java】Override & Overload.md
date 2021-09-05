---
title: 【Java】Override & Overload
date: 2020-12-05 13:02:30
tags: Java
category: OOP
---

方法的重写(Overriding)和重载(Overloading)是java多态性的不同表现，重写是父类与子类之间多态性的一种表现，重载可以理解成多态的具体表现形式。

<!-- more -->

### Override（重写）

重写是子类对父类的允许访问的方法的实现过程进行重新编写, 返回值和形参都不能改变。**即外壳不变，核心重写！**

重写的好处在于子类可以根据需要，定义特定于自己的行为。 即子类能够根据需要实现父类的方法。

重写方法不能抛出新的检查异常或者比被重写方法申明更加宽泛的异常。
例如： 父类的一个方法申明了一个检查异常 IOException，但是在重写这个方法的时候不能抛出 Exception 异常，因为 Exception 是 IOException 的父类，只能抛出 IOException 的子类异常。

为了满足里式替换原则，重写有有以下两个限制:

- 子类方法的访问权限必须大于等于父类方法；
- 子类方法的返回类型必须是父类方法返回类型或为其子类型。

**Notice：**重写方法时使用`@Override `注解，可以让编译器帮忙检查是否满足上面的两个限制条件。

```java
class Animal{
    public void move(){
        System.out.println("动物可以移动");
    }
}


class Dog extends Animal{
    /** dog作为animal的子类重写move()方法*/
    @Override
    public void move(){
        System.out.println("狗可以跑和走");
    }
}

public class TestDog{
    public static void main(String args[]){
        Animal a = new Animal(); // Animal 对象
        Animal b = new Dog(); // Dog 对象

        a.move();// 执行 Animal 类的方法

        b.move();//执行 Dog 类的方法
    }
}
```



### Overloading（重载）

重载(overloading) 是在一个类里面，方法名字相同，而参数不同。返回类型可以相同也可以不同。

每个重载的方法（或者构造函数）都必须有一个独一无二的参数类型列表。

最常用的地方就是构造器的重载。

**规则：**

- 被重载的方法必须改变参数列表(参数个数或类型不一样)；
- 被重载的方法可以改变返回类型；
- 被重载的方法可以改变访问修饰符；
- 被重载的方法可以声明新的或更广的检查异常；
- 方法能够在同一个类中或者在一个子类中被重载。
- 无法以返回值类型作为重载函数的区分标准。

常用的一种Overloading的形式：在类中使用private static进行函数定义，调用类内部的参数并给出返回值，再构建同名的private函数对已有private method进行调用，使得使用时可以依赖具体实例实现method功能。

```java
public class SLList {
    public class IntNode {
        public int item;
        public IntNode next;

        public IntNode(int i, IntNode n){
            item = i;
            next = n;
        }
    }
    private IntNode first;

    public SLList(int x){
        first = new IntNode(x,null);
    }
    /** Adds x to the front of the list */
    public void addFirst(int x){
        first = new IntNode(x, first);
    }

    /** returns the first item in the list */
    public int getFirst(){
        return first.item;
    }

    /** Adds x to the last of the list*/
    public void addLast(int x){
        IntNode p = first;
        while(p.next!=null){
            p = p.next;
        }
        IntNode q = new IntNode(x,null);
        p.next = q;
    }

    /** Returns the size of the list that starts at IntNode p*/
    private static int size(IntNode p){
        if(p.next ==null){
            return 1;
        }else{
            return 1+size(p.next);
        }
    }

    /** Returns the size of a given instance of the SLList */
    public int size(){
        return size(first);
    }
    public static void main(String[] args) {
        SLList L = new SLList(15);
        L.addFirst(10);
        L.addFirst(5);
        L.addLast(20);
        System.out.println(L.size());

    }
}

```



### 二者的区别与联系

|   区别   | Overloading  |                   Overriding                   |
| :------: | :----------: | :--------------------------------------------: |
| 参数列表 | **必须修改** |                  一定不能修改                  |
| 返回类型 |   可以修改   |                  一定不能修改                  |
|   异常   |   可以修改   | 可以减少或删除，一定不能抛出新的或者更广的异常 |
|   访问   |   可以修改   |     一定不能做更严格的限制（可以降低限制）     |



### Reference

- [Java全栈知识体系-重写与重载](https://www.pdai.tech/md/java/basic/java-basic-lan-basic.html#%e9%87%8d%e5%86%99%e4%b8%8e%e9%87%8d%e8%bd%bd)

- [Java重写与重载](https://www.runoob.com/java/java-override-overload.html)

