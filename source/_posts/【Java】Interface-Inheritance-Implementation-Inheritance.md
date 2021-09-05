---
title: 【Java】Interface Inheritance & Implementation Inheritance
date: 2021-01-13 09:51:54
tags: Java
category: OOP
---

**Interface Inheritance** and **Implementation Inheritance** are two different type of Interface method definition.

- **Interface inheritance** (what): Simply tells what the subclasses should be able to do.
  - EX) all lists should be able to print themselves, how they do it is up to them.
- **Implementation inheritance** (how): Tells the subclasses how they should behave.
  - EX) Lists should print themselves exactly this way: by getting each element in order and then printing them.

<!-- more -->

## Interface Inheritance

### Definition

- Specifying the capabilities of a subclass using the **implements** keyword is known as **interface inheritance**.

```java
public interface List61B<Item> {
   public void addFirst(Item x);
   ...
   public void proo();
}
```

- Interface has the list of all methods signatures, which  specifies what the subclass can do, but not how.
- If the subclass is not abstract, it must override all of these methods being specified in interface, otherwise it will not compile.

**Question:** Will the code below compile? If so, what happens when it runs?

```java
public static void main(String[] args) {
   List61B<String> someList = new SLList<String>();	
   someList.addFirst("elk");
}
```

- When it runs, an `SLList` is created and its address is stored in the `someList` variable. Then the `string` “elk” is inserted into the `SLList` referred to by `addFirst`.

## Implementation Inheritance

### Definition

- Use the **default** keyword to specify a method that subclasses should inherit from an **interface.**

```java
public interface List61B<Item> {
   public void addFirst(Item x);
   public void addLast(Item y);
   public Item getFirst();
   public Item getLast();
   public Item removeLast();
   public Item get(int i);
   public void insert(Item x, int position);
   public int size();  
   // here comes the default method which have implementation
   default public void print() {
      for (int i = 0; i < size(); i += 1) {
         System.out.print(get(i) + " ");
      }
      System.out.println();
   }
}

```

- In subclasses, default method from interface can also be override, when it been called, it will use its method instead of default.

**Question:** Which print method do you think will run when the code below executes?

```java
public interface SLList<Item> implements  {
   @Override
   public void print() {
      for (Node p = sentinel.next; p != null; p = p.next) {
   	      System.out.print(p.item + " ");     	
	   }
 
	   System.out.println();
   }
}

public static void main(String[] args) {
   List61B<String> someList = new SLList<String>();
   someList.addLast("elk");
   someList.addLast("are");
   someList.addLast("watching");
   someList.print();
}
```

- The type specified at **declaration** is called **static type** (In example, the static type of `someList ` is `List61B`).
- The type specified at **implementation** (when using `new`) is called **dynamic type**  (In example, the static type of `someList ` is `SLList`).
- In overriding, if the method has been specified by subclasses, the compiler will choose to record the method through dynamic type(which is subclass in the example).
- However, in overloading, the compiler doesn't have the `dynamic type selection`, so if the method has been implemented in the interface, it will be recorded even if there exists more specified method in subclass.