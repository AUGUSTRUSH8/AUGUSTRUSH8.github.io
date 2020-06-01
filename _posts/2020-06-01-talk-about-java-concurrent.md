---
layout: post
title: '重新审视Java并发'
tags: [code]
---

## 前言
以前学习Java并发的时候看各种资料，搜索各种解释，但无奈看完别人写的文章后总感觉还差点意思，老觉着有什么地方没有吃透，今天就来重新审视一下以前学的各种Java并发基础知识，以求进步一点点。

## 疑惑
- Java内存模型到底是个啥。
- 各个文章里面老是说主内存、工作内存，它们到底是什么。
- 可见性是什么，为什么不可见。
- 原子性是什么，咋样的才是原子的。
- 有序性又是什么，和我们代码的“显式顺序”有多大关系。

## Java内存模型
Java内存模型**是一个规范**，其主要定义了**JVM在计算机内存当中的工作方式**，主要规定了两点：
- 规定了一个线程如何以及何时可以看到其他线程修改过后的共享变量的值，即线程之间共享变量的可见性。
- 如何在需要的时候对共享变量进行同步。

实际上，我们只需要知道Java内存模型是帮我们屏蔽底层硬件细节的就好了，程序猿只要按照它的给定规则来写代码，就可以实现**write once，run everywhere**了。以上既然说到了Java内存模型是一种**规范**，那怎么实现这个规范呢？来看看JVM对Java内存模型的实现吧。

![](http://image.augustrush8.com/images/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.png)

怎么样，很熟悉吧，在JVM的实现当中把内存分成了几个块。

- 堆（Java进程当中所有线程共享）
- 方法区（同上）
- 虚拟机栈（每个线程独有）
- 本地方法栈（同上）
- PC计数器（同上）

下面我们就来看看到底什么是主内存，什么是工作内存。

## 主内存&工作内存

- 主内存就是堆+方法区
- 工作内存就是虚拟机栈+本地方法栈+PC计数器

## 一段前奏

先来看这样一段简单的代码：

```java
    private int count;
    public void increment() {
        this.count++;
    }
    public int getCount() {
        return this.count;
    }

    public static void main(String[] args) {
        Counter counter = new Counter();
        int times = 100000000;
        long currTime = System.currentTimeMillis();
        for (int i = 0; i < times; i++) {
            counter.increment();
        }
        System.out.println("current count is："+counter.getCount());
        System.out.println("time taked: "+(System.currentTimeMillis()-currTime)+"ms");
    }
}
```

结果输出：

```php
current count is：100000000
time taked: 9ms
```

看看字节码信息：

```php
//指令
javac Counter.java
javap -verbose Counter.class
```

可以找到increment()方法的字节码信息是这样的：

```java
public void increment();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: dup
         2: getfield      #2                  // Field count:I
         5: iconst_1
         6: iadd
         7: putfield      #2                  // Field count:I
        10: return
```

这段代码的运行时栈帧图示是下图这样的：

![](http://image.augustrush8.com/images/2020-6-1-%E6%93%8D%E4%BD%9C%E6%95%B0%E6%A0%88.png)

那么，有序性、可见性、原子性在上面这段代码当中是如何体现的呢？

- 循环从 0 到一亿，是严格按顺序执行的，这叫**有序性**
- 循环过程中，前一次对 count 的修改对后面可见，这叫**可见性**
- 因为是严格按顺序执行的，所以 count++ 操作中间不会交叉执行，所以其实在单线程环境中，可以认为满足**原子性**

对于以上提到的三个条件，只要有一个被打破了，那么结果就有可能不正确。

## 可见性

可见性问题到底是由什么引起的呢，先说结论：**因为CPU告诉缓存的存在，在多核环境当中会出现可见性问题**。为什么呢？我们还是以上面那段程序为例，注意到count自增的时候有两个指令的存在，分别是`getfield`和`putfield`，这两条指令是跟主存打交道的。

- getField 指令会从主存中读取 count 的值，但是并不是每次都从主存中读，因为 CPU 高速 cache 的存在，我们 count 值有可能会从 cache 中读，导致读的并不是最新的
- putField 指令会将 count 新的值写入主内存，但是也不是立即生效，别的 CPU 的高速 cache 中的 count 不会立即更新，CPU 会使用缓存一致性协议来做同步，这个对我们是透明的。

这个过程的图示如下图：

![](http://image.augustrush8.com/images/2020-6-1%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%AF%E8%A7%81%E6%80%A7.png)

那么有什么办法可以解决可见性的问题呢？Java的volatile关键字可以！volatile修饰的变量有三个主要特性：

- 写volatile变量的时候，会刷新各个CPU当中该变量的缓存值，载入最新的值。
- 读volatile变量的时候，会使得当前CPU当中该变量的缓存值失效，从主存当中载入最新的值。
- volatile可以禁止指令重排序。

好了，既然volatile可以解决之前提到的可见性问题，那我们就给变量加上volatile修饰符看看。修改后的代码是这样的：

```java
public class Counter {
    private volatile int count;
    public void increment() {
        this.count++;
    }
    public int getCount() {
        return this.count;
    }

    public static void main(String[] args) {
        Counter counter = new Counter();
        int times = 100000000;
        int half = times/2;
        Thread thread1 = new Thread(() ->{
            for (int i = 0; i < half; i++) {
                counter.increment();
            }
        });

        Thread thread2 = new Thread(() ->{
            for (int i = 0; i < half; i++) {
                counter.increment();
            }
        });
        long startTime = System.currentTimeMillis();
        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("count:" + counter.getCount());
        System.out.println("take time:" + (System.currentTimeMillis() - startTime) + "ms");

    }
}

```

看看运行结果：

```php
count:50686418
take time:1560ms
```

结果不对啊，时间也变长了好多，这是为什么呢？主要是由于volatile只保证了可见性，但无法保证原子性，实际上以上代码，如果你装了对应的Java语法bug检查插件，会提示你这样一句话`Non-atomic operation on volatile field 'count' `，其图解如下图所示。

![](http://image.augustrush8.com/images/2020-6-1%E9%9D%9E%E5%8E%9F%E5%AD%90%E6%80%A7%E5%B9%B6%E5%8F%91.png)

>在 T1-T6 时间内，初始 count=0，经过二次 ++ 操作，最后 count 的值还是 1，在我们上面的例子中，5 千万次的循环会出现大量类似的错误覆盖写入。根据我们上面分析的 volatile 的语义，在 T5 时刻，Thread1 对 count 的修改对 Thread2 是可见的，这里的可见指的是，如果此时调用 getfield 指令，拿到的值会是 Thread1 修改的最新的 1，但是遗憾的是，Thread2 对此一无所知，只是按着自己的步骤将错误的 1 写入了 count 中。

有了上面的分析我们知道，要想在并发条件下使得输出结果正确，关键就是要让`increment()`方法满足**原子性**，换句话说，我们想要的自增效果是这样的：

> 在 putfield 之前，检查下当前栈中存储的 count 是不是最新的，如果不是最新的重新读取 count，然后重试，如果是最新的，直接写入更新值

这里需要注意的是，以上提到的方法流程本身要满足原子性，同时，之前反编译得到的`increment()`方法的的每一条字节码指令也要是原子的（实际也确实是这样的，这一点由JMM保证），这样我们上面提到的方法流程才有意义。实际上，JDK 中 `Unsafe` 包里面的 `CAS` 方法就是这个思路，具体的底层本地函数实现可以参看参考文章第二篇。这里直接给出优化后的代码。

第一步，Unsafe工具类：

```java
public class UnsafeUtil {
    public static Unsafe getUnsafeObject() {
        Class clazz = AtomicInteger.class;
        try {
            Field uFiled = clazz.getDeclaredField("unsafe");
            uFiled.setAccessible(true);
            return (Unsafe) uFiled.get(null);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    public static long getVariableOffset(Object target, String variableName) {
        Object unsafeObject = getUnsafeObject();
        if (unsafeObject != null) {
            try {
                Method method = unsafeObject.getClass().getDeclaredMethod("objectFieldOffset", Field.class);
                method.setAccessible(true);
                Field targetFiled = target.getClass().getDeclaredField(variableName);
                return (long) method.invoke(unsafeObject, targetFiled);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return -1;
    }

}
```

第二步，改写Counter类：

```java
public class Counter {
    private volatile int count;
    private Unsafe mUnsafe;
    private long countOffset;
    public Counter() {
        mUnsafe = UnsafeUtil.getUnsafeObject();
        countOffset = UnsafeUtil.getVariableOffset(this, "count");
    }
    public void increment() {
        int cur = getCount();
        while (!mUnsafe.compareAndSwapInt(this, countOffset, cur, cur+1)) {
            cur = getCount();
        }
    }
    public int getCount() {
        return this.count;
    }


    public static void main(String[] args) {
        Counter counter = new Counter();
        int times = 100000000;
        int half = times/2;
        Thread thread1 = new Thread(() ->{
            for (int i = 0; i < half; i++) {
                counter.increment();
            }
        });

        Thread thread2 = new Thread(() ->{
            for (int i = 0; i < half; i++) {
                counter.increment();
            }
        });
        long startTime = System.currentTimeMillis();
        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("count:" + counter.getCount());
        System.out.println("take time:" + (System.currentTimeMillis() - startTime) + "ms");

    }
}

```

看看新的输出结果：

```php
count:100000000
take time:3857ms
```

可以看到，结果正确了，但花费的时间同样进一步增加了。

## 原子性

前面提到过，单个字节码指令是保证原子性的，但是多条字节码指令的组合就不一定了，总结起来，保证程序原子性的几个主要方法有下面三种：

- **CAS + 自旋**
- **synchronized 关键字**
- **concurrent 包提供的 Lock，具体实现类比如 ReentrantLock**

CAS+自旋的方式，前面已经介绍过了，这里不再赘述，直接来看看`synchronized`关键字。`synchronized`关键字的用法主要有下面三种：

```java
// 普通类方法同步
synchronized publid void method() {}
// 类静态方法同步
synchronized public static void method() {}
// 代码块同步
synchronized(object) {
}
```

几种不同用法的区别是

- 普通类 synchronized 同步的就是对象本身
- 静态方法同步的是类 Class 本身
- 代码块同步的是我们在括号内部填入的对象

那么，我们用`synchronized`关键字来改写一下我们的`increment()`方法。

```java
public class Counter {
    private volatile int count;
    public void increment() {
        synchronized (Counter.class) {
            this.count++;
        }
    }
    public int getCount() {
        return this.count;
    }


    public static void main(String[] args) {
        Counter counter = new Counter();
        int times = 100000000;
        int half = times/2;
        Thread thread1 = new Thread(() ->{
            for (int i = 0; i < half; i++) {
                counter.increment();
            }
        });

        Thread thread2 = new Thread(() ->{
            for (int i = 0; i < half; i++) {
                counter.increment();
            }
        });
        long startTime = System.currentTimeMillis();
        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("count:" + counter.getCount());
        System.out.println("take time:" + (System.currentTimeMillis() - startTime) + "ms");

    }
}
```

运行结果如下：

```php
count:100000000
take time:3330ms
```

这个结果是正确的，而且时间花费比`CAS`方法还少？这是为什么呢，按照我们的固有执念，CAS不应该比`synchronized`快吗？实际上，`synchronized`自`Java 1.7`之后引入了`偏向锁`、`轻量级锁`之后，性能有了很大提升，因此我们在使用`synchronized`的时候不需要有太大的心理负担，如果不是对性能有非常高的需求，使用`synchronized`是没有问题的。

还是来看看`synchronized`改写后的`increment()`方法的字节码指令吧。

```
public void increment();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=3, args_size=1
         0: ldc           #2                  // class com/admin/hellotest/Counter
         2: dup
         3: astore_1
         4: monitorenter  //注意这里
         5: aload_0
         6: dup
         7: getfield      #3                  // Field count:I
        10: iconst_1
        11: iadd
        12: putfield      #3                  // Field count:I
        15: aload_1
        16: monitorexit   //注意这里
        17: goto          25
        20: astore_2
        21: aload_1
        22: monitorexit   //注意这里
        23: aload_2
        24: athrow
        25: return
```

注意上面标注的三个地方，可以看到其中有1个`monitorenter`指令，2个`monitorexit`指令，实际上是因为编译器帮我们加上了`try-finally`语句块，以确保monitor一定会被释放，即使出现了异常，也会有第二个`monitorexit`帮我们释放锁。实际编译后的代码可以理解为下面的形式：

```java
// 我们的实现
synchronized (Counter.class) {
    this.count++;
}

// 编译后，相当于下面的伪代码
monitorenter  Counter.class;
try {
    this.count++;
    monitorexit  Counter.class;
} finally {
    monitorexit  Counter.class;
}
```

以上针对保证原子性的三个操作前两个已经介绍完毕，下面再来看看第三个方法——`ReentrantLock`，`ReentrantLock`的用法主要如下所示：

```java
// 参数表示 是否是公平锁，公平锁严格按照等待顺序获取锁，但是吞吐率低性能差
// 非公平锁性能高，但是有可能会出现锁等待饥饿
ReentrantLock reentrantLock = new ReentrantLock(false);
// 一个标准的使用方式
reentrantLock.lock();
try {
    // do something
} finally {
    reentrantLock.unlock();
}
```

需要注意的是**加锁的 lock 方法的调用，一定要在 try-catch-finally 的前面，不能在内部，因为如果在内部调用 lock，如果代码在 lock 之前就出现异常了，就会出现我们没有加锁就执行了 finally 里面的释放锁，肯定会有问题的。**

然后我们使用`ReentrantLock`来改写一下我们的`increment()`代码：

```java
public class Counter {
    private int count;
    public void increment() {
        this.count++;
    }
    public int getCount() {
        return this.count;
    }


    public static void main(String[] args) {
        Counter counter = new Counter();
        int times = 100000000;
        int half = times/2;
        Lock lock = new ReentrantLock(false);
        Thread thread1 = new Thread(() ->{
            for (int i = 0; i < half; i++) {
                lock.lock();
                try {
                    counter.increment();
                }finally {
                    lock.unlock();
                }
            }
        });

        Thread thread2 = new Thread(() ->{
            for (int i = 0; i < half; i++) {
                lock.lock();
                try {
                    counter.increment();
                }finally {
                    lock.unlock();
                }
            }
        });
        long startTime = System.currentTimeMillis();
        thread1.start();
        thread2.start();

        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("count:" + counter.getCount());
        System.out.println("take time:" + (System.currentTimeMillis() - startTime) + "ms");

    }
}
```

输出结果：

```php
count:100000000
take time:3679ms
```

可以看出结果也是对的，时间消耗也比CAS少，这就说明CAS的方式不一定时间一定少，所以我们日后遇到类似的问题的时候最好自己写代码考证一下，而不是简单的凭直觉去判断。关于`ReentrantLock`的源码分析部分，参考文章2也给出了一些源码分析，我觉得很精彩，感兴趣的同学可以去看看。

## 有序性

直接来看一个赋值的简单例子：

```java
int a = 1;   //1
int b = 2;   //2
int c = a;   //3
```

我们通常预想的执行顺序应该是1、2、3，可实际执行顺序则可能是2、1、3或1、3、2，这就是指令重排序。指令重排序主要有两种：

- CPU 级的指令重排序
- 编译器级的指令重排序

重排序的目的是优化程序的运行速度，但是前提是不能够破坏as-if-serial语义，就拿上面的代码来说，第三条对`c`的赋值依赖于第一条语句中对`a`的赋值，因此第三条语句的执行永远排在第一条语句之后。在单线程有序性还比较容易保证，但是在多线程情况就会变得复杂起来。所以 JMM 中抽象出了一个 `happen-before` 原则，这个原则是 `JMM` 给我们开发者的承诺，让我们写代码时对多线程情况下的有序性有一个正确的预期。这个原则有下面 5 条。

- 同一个线程中，程序中前面的代码 happen-before 后面的代码

- 对一个 monitor 的解锁 happen-before 对这个 monitor 的加锁

- 对一个 volatile 变量的写 happen-before 对这个 volatile 变量的读

- 线程 start 方法调用 happen-before 线程内的所有 action

-  在 A 线程调用了 B 线程的 join，则 B 线程内的操作 happen-before 于 A 线程后续的操作

**当然 happen-before 具有传递性，如果 A happen-before B, B happen-before C，则 A 也 happen-before C。** 需要注意的是，happen-before 并不完全等同于时间意义上的先执行，比如上面的例子中，根据第一条 happen-before 原则，int a = 1; 这条语句 happen-before int b = 2; 这条语句，但是由于二者之间没有依赖关系，可以指令重排，所以可以是 int b = 2；先执行，这是合法的，并不违背 happen-before 原则。

理解这几条 happen-before 原则后，很多我们平时经常写的并发代码就有了理论依据，比如第二条，加锁 happen-before 解锁，所以保证了锁的同步范围内的代码，具有原子性和有序性，同时加锁和解锁都会插入内存屏障，可见性也得到保障，所以加锁后的代码是线程安全的。再比如第三条，volatile 的写 happen-before 于 volatile 的读，有了这一条，多线程之间 volatile 修饰的共享变量的可见性得到保证。

另外几条原则比较好理解，大家可以自行结合实际代码加深理解，这里就不赘述了。

## 参考文章

```php
[1] https://juejin.im/post/5d3eafe95188255d3d296e09
[2] https://juejin.im/post/5d16a633e51d455a2f2202a3
```

