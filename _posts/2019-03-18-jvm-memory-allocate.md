---
layout: post
title: 'String str = new String("Hello");'
tags: [read]
---

String有两种赋值方式，**第一种是通过“字面量”赋值**。比如下面这行，

```java
String str = "Hello";
```

**第二种是通过new关键字创建新对象**。比如下面这样，

```javascript
String str = new String("Hello");
```

这两种方式到底有什么不同。程序执行的时候，内存里到底有几个实例？**“实例”**存在了内存的哪里？**”字面量“**又存在了哪里？**”变量“**又存在了哪里？概念很容易搞混。下面我们一个一个来讲。讲之前，先回顾一下内存。

![](http://image.augustrush8.com/images/jvmshili1.png){:.center}

上面这张是虚拟机的结构图，其他先不管，我们主要看中间五彩这一条叫 “**运行时数据区（Run-time Data Areas）”**。就是虚拟机管理的内存。就是大白话的“内存”。其中后面两个，一个程序计数器（PC Registers），一个本地方法栈（Native Method Stack）和今天讲的没关系，先忽略。一般讲起来虚拟机内存最主要的就是三块：

1. **堆（Heap）**：最大一块空间。存放对象实例和数组。全局共享。
2. **栈（Stack）**：全称 “虚拟机栈（JVM Stacks）”。存放基本型，以及对象引用。线程私有。
3. **方法区（Method Area）**：“类”被加载后的信息，常量，静态变量存放在这儿。全局共享。在HotSpot里也叫“永生代”。但两者不能等同。

![](http://image.augustrush8.com/images/jvmshili2.png){:.center}

上图中，首先Heap堆分成“新生代”，“老年代”，先不用管它，这是GC垃圾回收时候的事。重要的是Stack栈区里的**“局部变量表（Local Variables）”**和**“操作数栈（Operand Stack）”**。因为栈是线程私有的，每个方法被执行的时候都会创建一个**“栈帧（Stack Frame）”**。而每个栈帧里对应的都维护着一个局部变量表和操作数栈。我们老说基本型和对象引用存在栈里，其实就是存在局部变量表里。而操作数栈是线程实际的操作台。看下面这张图，做个加法100+98，局部变量表就是存数据的地方，一直不变，到加法做完再把和加进去。操作数栈就很忙了，先把两个数字压进去，再求和，算出来以后再弹出去。

![](http://image.augustrush8.com/images/jvmshili3.png){:.center}

中间这个非堆（Non-Heap）可以粗略地理解为非堆里包含了永生代，而永生代里又包括了方法区。上面说了，每个类加载完之后，类的信息都存在方法区里。和String最相关的是里面的**“运行时常量池（Run-time Constant Pool）”**。它是每个类私有的。后面会说到，每个class文件里的“常量池”在类被加载器加载之后，就映射存放在这个地方。另外一个是**“字符串常量池（String Pool）”**。和运行时常量池不是一个概念。字符串常量池是全局共享的。位置就在第二张图里Interned String的位置，可以理解为在永生代里，方法区外面。后面会讲到，String.intern()方法，字符串驻留之后，引用就放在这个String Pool。

可以开始说String了。

比如下面这个**Test.java**文件。在主线程方法main里声明了一个字面量是"Hello"的字符串str。

```java
package com.ciao.shen.java.string;

class Test{
    public void f(String s){...};

    public static void main(String[] args){
        String str = "Hello";
        ...
    }
}
```

编译成**Test.class**文件之后，如下图，除了版本、字段、方法、接口等描述信息外，还有一个也叫**“常量池（Constant Pool Table）”**的东西（淡绿色区块）。但这个常量池和内存里的常量池不是一个东西。class文件里的常量池主要存两个东西：**“字面量（****Literal）”**和**“符号引用量(Symbolic References)”**。其中字面量就包括类中定义的一些常量，因为String是不可变的，由final关键字修饰过了，所以代码里的“Hello”字符串，就是作为字面量（常量）写在class的常量池里。

![](http://image.augustrush8.com/images/jvmshili4.jpg){:.center}

运行程序用到Test类的时候，Test.class文件的信息就会被解析到内存的方法区里。class文件里常量池里大部分数据会被加载到“运行时常量池”。但String不是。例子中的"Hello"的一个引用会被存到同样在Non Heap区的字符串常量池（String Pool）里。而“Hello”本体还是和所有对象一样，创建在Heap堆区。R大的文章里，测试的结果是在新生代的Eden区。但因为一直有一个引用驻留在字符串常量池，所以不会被GC清理掉。这个Hello对象会生存到整个线程结束。如下图所示，字符串常量池的具体位置是在过去说的永生代里，方法区的外面。

![](http://image.augustrush8.com/images/jvmshili5.jpg){:.center}

> **！注意：**这只是在Test类被类加载器加载时候的情形。主线程中的str变量这时候都还没有被创建，但Hello的实例已经在Heap里了，对它的引用也已经在字符串常量池里了。

等主线程开始创建str变量的时候，虚拟机就会到字符串常量池里找，看有没有能equals("Hello")的String。如果找到了，就在栈区当前栈帧的局部变量表里创建str变量，然后把字符串常量池里对Hello对象的引用复制给str变量。找不到的话，才会在heap堆重新创建一个对象，然后把引用驻留到字符串常量区。然后再把引用复制栈帧的局部变量表。

![](http://image.augustrush8.com/images/jvmshili6.jpg){:.center}

如果我们当时定义了很多个值为"Hello"的String，比如像下面代码，有三个变量str1,str2,str3，也不会在堆上增加String实例。局部变量表里三个变量统一指向同一个堆内存地址。

```java
package com.ciao.shen.java.string;

class Test{
    public void f(String s){...};

    public static void main(String[] args){
        String str1 = "Hello";
        String str2 = "Hello";
        String str3 = "Hello";
        ...
    }
}
```

![](http://image.augustrush8.com/images/jvmshili7.jpg){:.center}

上图中str1,str2,str3之间可以用==来连接。

但如果是用new关键字来创建字符串，情况就不一样了，

```java
package com.ciao.shen.java.string;

class Test{
    public void f(String s){...};

    public static void main(String[] args){
        String str1 = "Hello";
        String str2 = "Hello";
        String str3 = new String("Hello");
        ...
    }
}
```

这时候，str1和str2还是和之前一样。但str3因为new关键字会在Heap堆申请一块全新的内存，来创建新对象。虽然字面还是"Hello"，但是完全不同的对象，有不同的内存地址。

![](http://image.augustrush8.com/images/jvmshili8.jpg){:.center}

当然String#intern()方法让我们能手动检查字符串常量池，把有新字面值的字符串地址驻留到常量池里。

最后补充一下，JDK 7开始Hotspot把Interned String从PermGen挪到Heap堆，JDK 8又彻底取消了PermGen。但不管怎样，基本原理还是不变的。