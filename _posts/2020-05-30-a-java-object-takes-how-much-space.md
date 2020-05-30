---
layout: post
title: '一个 Java 对象到底占用多少内存'
tags: [code]
---
## 前言
这个问题我相信对于很多有经验的Java程序猿来说也是一个比较难回答的问题，说实话，我刚开始被人问到这个问题的时候也是一脸懵逼，首先脑子里很难一下想起或者猜测这个内存占用值到底是多少，因此着手去查之，果然有对应文章阐述这个问题。下面就这个问题展开探索，学习了也便于我们日后进行内存优化相关的工作不是？

## 环境基础
- java version "1.8.0_181"

## Java当中数据类型有哪些
从大的角度可以分为两类：
- 基础数据类型
- 引用类型

### 基础数据类型又分为哪些
- byte（占用1字节）
- short（占用2字节）
- int（占用4字节）
- long（占用8字节）
- float（占用4字节）
- double（占用8字节）
- char（占用2字节）
- boolean（占用1字节）

基础数据类型所占用的内存空间大小是确定的。

### 基础数据类型在哪里存储分配
- 如果在**方法体内**定义的，这时候就是在**栈**上分配的
- 如果是**类的成员变量**，这时候就是在**堆**上分配的
- 如果是**类的静态成员变量**，在**方法区**上分配的

```java
public class Pig {
    //堆上分配
    private String name;
    //方法区分配
    private static int age;
    private int getPigWeight() {
        //栈上分配
        int wieght = 100;
        return wieght;
    }
}
```

### 引用类型的存储大小和存储位置
引用类型除对象本身之外，还有一个指向这个对象的指针，在64位虚拟机环境下，这个指针通常占用4个字节，因为默认开启了指针压缩，如果不开启指针压缩的话就占用8个字节。存储位置还是看下面代码吧。

```java
public class Pig {
    //'name'和它指向的对象都存储在堆上
    private String name = new String();
    //'color'和它所指向的对象都存储在方法区上
    private static String color = new String();
    private void show() {
        //temp则在栈上，它所指向的对象可能在堆上也可能在栈上
        String temp = new String();
        System.out.println(temp);
    }
}
```

## Java中对象到底占多大内存
Java中对象分为三种类型：
- 类对象
- 数组对象
- 接口对象

对象由三个部分组成：
- 对象头
- 实例数据
- 填充数据（padding，由于Java虚拟机规范要求对象所占用内存空间大小需要是8的倍数，所以才有这东西）

### 对象头
对象头同样由两部分组成：
- Markword
- 类指针（kclass）

#### Markword
按照官方文档对Markword的定义，Markword由64位组成，有下面四种情况：
- unused:25 hash:31 -->| unused:1   age:4    biased_lock:1 lock:2 (normal object)
- JavaThread*:54 epoch:2 unused:1   age:4    biased_lock:1 lock:2 (biased object)
- PromotedObject*:61 --------------------->| promo_bits:3 ----->| (CMS promoted object)
- size:64 ----------------------------------------------------->| (CMS free block)

其中：

- normal object（对应刚new出来的对象）
- biased object（当某个对象被作为同步锁对象时，会有一个偏向锁，其实就是存储了持有该同步锁的线程 id）
- CMS promoted object不清楚是啥
- CMS free block 不清楚是啥

我们主要关注 normal object， 这种类型的 Object 的 Markword 一共是 8 个字节，由以上定义，我们可以知道 normal object 的Markword组成是这样的：
- 开始**25**位没有使用
- **31** 位存储对象的 hash 值（这里存储的 hash 值对根据对象地址算出来的 hash 值，不是重写 hashcode 方法里面的返回值）
- 中间**1**位同样没有使用
-  **4** 位存储对象的 age（分代回收中对象的年龄，超过 15 晋升入老年代）
- 最后**3**位表示偏向锁标识和锁标识，主要就是用来区分对象的锁状态（未锁定，偏向锁，轻量级锁，重量级锁）

**小扩展**

什么时候normal object会转变为biased object呢？
```java
// 无其他线程竞争的情况下，由normal object变为biased object
synchronized(object)
```

#### 类指针
类指针存储的是**该对象所属的类在方法区的地址**， Jvm 默认对指针进行了压缩，用 4 个字节存储，如果不压缩就是 8 个字节，主要是出于提高内存分配效率的考虑。

#### 分析个实例
```java
class Animal extends Object {
     private int size;
}

Object object = new Object();
Animal animal = new Aniaml();
```
- object对象：8个字节的Markword+4个字节的类指针+4个字节的padding=16字节
- animal对象：8个字节的Markword+4个字节的类指针+4个字节的int类型数据=16字节

**分析检验**

```java
import org.openjdk.jol.info.ClassLayout;

public class Animal extends Object {
    private int size;
    public static void main(String[] args) {
        System.out.println(ClassLayout.parseClass(Object.class).toPrintable());
        System.out.println(ClassLayout.parseClass(Animal.class).toPrintable());
    }

}
```

这里用到了一个包：

```xml
<dependency>
	<groupId>org.openjdk.jol</groupId>
	<artifactId>jol-core</artifactId>
	<version>0.10</version>
</dependency>
```

**输出结果**

```java
java.lang.Object object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4        (loss due to the next object alignment)
Instance size: 16 bytes
Space losses: 0 bytes internal + 4 bytes external = 4 bytes total

com.admin.hellotest.Animal object internals:
 OFFSET  SIZE   TYPE DESCRIPTION                               VALUE
      0    12        (object header)                           N/A
     12     4    int Animal.size                               N/A
Instance size: 16 bytes
Space losses: 0 bytes internal + 0 bytes external = 0 bytes total

```

可以看出和我们的分析结果是吻合的。类对象和接口对象和我们上面分析的方法一致，数组对象稍微不同，数组类型的对象除了上面表述的字段外，还有 4 个字节存储数组的长度（所以数组的最大长度是 Integer.MAX）。所以一个数组对象占用的内存是 8 + 4 + 4 = 16 个字节，当然这里不包括数组内成员的内存。

**再看个例子**

```java
public class Dog extends Animal {
    private String name;
    private Dog brother;
    private long createTime;

    public static void main(String[] args) {
        System.out.println(ClassLayout.parseClass(Dog.class).toPrintable());

    }
}
```

输出结果：

```java
com.admin.hellotest.Dog object internals:
 OFFSET  SIZE                      TYPE DESCRIPTION                               VALUE
      0    12                           (object header)                           N/A
     12     4                       int Animal.size                               N/A
     16     8                      long Dog.createTime                            N/A
     24     4          java.lang.String Dog.name                                  N/A
     28     4   com.admin.hellotest.Dog Dog.brother                               N/A
Instance size: 32 bytes
Space losses: 0 bytes internal + 0 bytes external = 0 bytes total
```

## 对象真的全部分配在堆里吗
来看这样一段代码
```java
public class Dog extends Animal {
    private String name;
    private Dog brother;
    private long createTime;

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 100000000; i++) {
            newDog();
        }
        System.out.println("消耗时间：" + (System.currentTimeMillis() - start) + " ms");
    }

    private static void newDog() {
        new Dog();
    }
}
```
那么按照前面的分析，一个Dog对象将占用32个字节，这里创建1亿个，估算一下，大概就是3G内存占用，加上虚拟机选项配置：`-XX:+PrintGC`来看看输出结果。
```java
//没有GC输出日志
消耗时间：4 ms

Process finished with exit code 0
```
为什么会这样呢？主要是由于Dog对象并不是在堆上分配的，而是在栈上分配的，专业名词叫做`指针逃逸`，newDog()方法创建的对象并没有被外部使用，所以被优化为栈上分配，可以加上下面两个虚拟机参数禁止`指针逃逸`
。
```xml
// 虚拟机关闭指针逃逸分析
-XX:-DoEscapeAnalysis
// 虚拟机关闭标量替换
-XX:-EliminateAllocations
```

好了，以上就是今天的全部内容了，大家get了多少姿势呢？
