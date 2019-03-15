---
layout: post
title: 'Java对象数组的复制难道不是浅复制'
tags: [read]
---

> 先说答案，Java对象数组的复制是浅复制！

下面开始一步一步解开迷惑：

Java编程思想451页说对象数组的复制是浅复制，看个例子

```java
import java.util.Arrays;
public class CopyTest {
    public static void main(String[] args) {
        String[] s1=new String[]{"Hi","Hi","Hi"};
        String[] s2=new String[3];
        System.arraycopy(s1,0,s2,0,s1.length);
        System.out.println(Arrays.toString(s1));
        System.out.println(Arrays.toString(s2));
        s1[2]="hier";
        System.out.println(Arrays.toString(s1));
        System.out.println(Arrays.toString(s2));

    }
}

```

看看输出：

```xml
[Hi, Hi, Hi]
[Hi, Hi, Hi]
[Hi, Hi, hier]
[Hi, Hi, Hi]
```

奇怪的是，s2字符串数组为何没有改变，难道不是浅复制？

再来看个定义：

> 浅拷贝只是复制了对象的引用，而不是对象本身的拷贝。

正常的理解应该是：

> 把s1[2]改成"Hier"，希望s2内第3个元素也变成"Hi,Hi,Hier"。因为s2里的元素和s1引用的应该都是同一个对象。如果改变这个对象的值，就等于改变s1和s2中对应对象的值。

为什么结果不对呢？

> 注意做这个测试要避开**String**和**Integer（包括所有自动装箱类）**这两种类型。

**String的问题是，它是不可变的。**你无法改变任何一个String对象的值。用"="等号给她赋值，并没有改变这个String对象的值，而是让变量指向了一个全新的对象。

```java
String s1 = "Hi"; 
String s2 = s1;
s1 = "Hier"; // "Hier"是一个全新的对象
System.out.println(s1 == s2); // false
```

**Integer的问题是，它的自动装箱机制。**你用一个int为Integer赋值，编译器会调用 "new Integer()" 创建一个全新的Integer实例，虽然他们的值相等。所以如果你想用等号 "=" 改变一个Integer的值，你又落空了。

```java
Integer i = 1000;
Integer j = 1000;
System.out.println(i == j); // false
```

所以测试的时候，要避开不可变类和自动装箱类。比如自己创建一个Node类型，持有一个int型的val域。然后用一对get()和set()函数访问域。

```java
public class Node {
   private int val;
    public Node(int n) { val = n; }
    public int getVal() { return val; }
    public void setVal(int n) { val = n; }
    public String toString() { return String.valueOf(val); }
}
```

创建两个Node[]数组，u和v。用同一个Node对象填充数组u。然后把数组u拷贝到数组v。

```java
Node[] u = new Node[10];
Node[] v = new Node[10];
Arrays.fill(u,new Node(11));
System.arraycopy(u,0,v,0,u.length);

System.out.println("u = " + Arrays.toString(u));
System.out.println("v = " + Arrays.toString(v));

// output
// u = [11,11,11,11,11,11,11,11,11,11]
// v = [11,11,11,11,11,11,11,11,11,11]
```

用setVal()函数改变其中任意一个Node对象的值，两个数组所有元素的值全都改变，证明他们指向同一个对象。

```java
u[0].setVal(22);

System.out.println("u = " + Arrays.toString(u));
System.out.println("v = " + Arrays.toString(v));

// output
// u = [22,22,22,22,22,22,22,22,22,22]
// v = [22,22,22,22,22,22,22,22,22,22]
```

