---
title: Java基础
date: 2020-07-07 17:59:18
categories: 
 - Java
 - 基础
tags: 
 - Java基础
---
# Java零散知识（之后可能会整理）
[http://hollischuang.gitee.io/tobetopjavaer/#/basics/object-oriented/object-oriented-vs-procedure-oriented](http://hollischuang.gitee.io/tobetopjavaer/#/basics/object-oriented/object-oriented-vs-procedure-oriented)

<!-- more -->

## VO DTO DO PO

- **从前端传后端是VO，从后端传前端是DTO，表对应实体类是PO**

- VO (View Object) :视图对象，用于展示层，它的作用是把某个指定页面(或组件)的所有数据封装起来。
- DTO (Data Transfer Object) :数据传输对象,这个概念来源于J2EE的设计模式，原来的目的是为了EJB的分布式应用提供粗粒度的数据实体，以减少分布式调用的次数,从而提高分布式调用的性能和降低网络负载，但在这里，我泛指用于展示层与服务层之间的数据传输对象。
- DO (Domain Object) :领域对象,就是从现实世界中抽象出来的有形或无形的业务实体。
- PO (Persistent Object) :持久化对象，它跟持久层(通常是关系型数据库)的数据结构形成一一对应的映射关系，如果持久层是关系型数据库,那么,数据表中的每个字段(或若干个)就对应PO的一个(或若干个)属性。

## Java类的加载顺序

### 1、有继承关系的加载顺序

关于关键字static，大家都知道它是静态的，相当于一个全局变量，也就是这个属性或者方法是可以通过类来访问，当class文件被加载进内存，开始初始化的时候，被static修饰的变量或者方法即被分配了内存，而其他变量是在对象被创建后，才被分配了内存的。

所以在类中，加载顺序为：

1. 首先加载父类的静态字段或者静态语句块

2. 子类的静态字段或静态语句块

3. 父类普通变量以及语句块

4. 父类构造方法被加载

5. 子类变量或者语句块被加载

6. 子类构造方法被加载

父类代码：

```java
public class FuLei {
static int num  = 5;//1.首先被加载
static{
    System.out.println("静态语句块已经被加载"+num); //2.被加载
}
int count = 0; //5.被加载
{
    System.out.println("普通语句块"+count++);//6.被加载
}
public FuLei(){
    System.out.println("父类的构造方法在这时候加载count="+count);//7.被加载
}
}
```

子类代码：

```java
public class ZiLei extends FuLei {
static{
    System.out.println("静态语句块和静态变量被初始化的顺序与代码先后顺序有关"); //3.被加载
}
static int num = 45;//4.被加载
int numre = 0; //8.被加载
{
    numre++;
    System.out.println("numre"+numre);//9.被加载

}
public ZiLei(){
    System.out.println("子类构造方法");//10.被加载
}
public static void main(String[] args){
    ZiLei ht = new ZiLei();
}
}
```

console打印：

```
静态语句块已经被加载5
静态语句块和静态变量被初始化的顺序与代码先后顺序有关
普通语句块0
父类的构造方法在这时候加载count=1
numre1
子类构造方法
```

**注意**

当class文件被加载进内存，开始初始化的时候，被static修饰的变量或者方法即被分配了内存，而其他变量是在对象被创建后，才被分配了内存的。

将子类代码中的创建对象注释掉

```java
// ZiLei ht = new ZiLei();
```

console打印：

```
静态语句块已经被加载5
静态语句块和静态变量被初始化的顺序与代码先后顺序有关
```

### 2、没有继承关系的加载顺序

代码示例

```java
public class Test {
public static void main(String[] args) {
    new Test();                         //4.第四步，new一个类，但在new之前要处理匿名代码块
}

static int num = 4;                    //2.第二步，静态变量和静态代码块的加载顺序由编写先后决定

{
    num += 3;
    System.out.println("b");           //5.第五步，按照顺序加载匿名代码块，代码块中有打印
}

int a = 5;                             //6.第六步，按照顺序加载变量

{ // 成员变量第三个
    System.out.println("c");           //7.第七步，按照顺序打印c
}

Test() { // 类的构造函数，第四个加载
    System.out.println("d");           //8.第八步，最后加载构造函数，完成对象的建立
}

static {                              // 3.第三步，静态块，然后执行静态代码块，因为有输出，故打印a
    System.out.println("a");
}

static void run()                    // 静态方法，调用的时候才加载// 注意看，e没有加载
{
    System.out.println("e");
}
}
```

console打印：

```
a
b
c
d
```

### 3、注意

- 静态代码块（只加载一次）
- 构造方法（创建一个实例就加载一次）
- 静态方法，调用的时候才会加载，不调用的时候不会加载
- 静态语句块和静态变量被初始化的顺序与代码先后顺序有关

## 静态方法和非静态方法的区别是什么

静态方法和非静态方法的区别总结如下：

1. 静态方法属于类所有，类实例化前即可使用； 
2. 非静态方法可以访问类中的任何成员，静态方法只能访问类中的静态成员； 
3. 因为静态方法在类实例化前就可以使用，而类中的非静态变量必须在实例化之后才能分配内存；    
4. static内部只能出现static变量和其他static方法!而且static方法中还不能使用this等关键字，因为它是属于整个类；
5. 静态方法效率上要比实例化高，静态方法的缺点是不自动进行销毁，而实例化的则可以做销毁； 
6. 静态方法和静态变量创建后始终使用同一块内存，而使用实例的方式会创建多个内存。
7. 主要区别：静态方法在创建对象前就可以使用了，非静态方法必须通过new出来的对象调用。
8. 静态方法与实例方法在性能和占用内存上没有明显的区别，是否声明为静态方法需要从类型的非静态字段、事件、面向对象扩展和多态这三方面来考虑。