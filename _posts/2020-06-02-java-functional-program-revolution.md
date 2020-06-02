---
layout: post
title: 'Java函数式编程演进之路'
tags: [code]
---

## 前言

作为 Java 开发人员，向来听说比较多或者说比较擅长的是`命令式`编程和`面向对象`编程，而自 Java8 以来，推出了一组新的函数特性和语法，而函数式编程通常有这样几个优点：

- 简洁，更具表达力
- 更不容易出错
- 更容易并行化

但很多同学从之前的命令式编程思维过渡到函数式编程思维感觉还是有些生硬，因此这里特地讲讲 Java 函数式编程的演进之路。

## 命令式编程

命令式编程用一句话简单概括就是 `tell what & tell how`，即告诉程序做什么和怎么做，来看一段在字符串列表当中寻找 `Nemo` 字符串的代码示例。

```
import java.util.*;
 
publicclass FindNemo {
  public static void main(String[] args) {
    List<String> names =
      Arrays.asList("Dory", "Gill", "Bruce", "Nemo", "Darla", "Marlin", "Jacques");
 
    findNemo(names);
  }
   
  public static void findNemo(List<String> names) {
    boolean found = false;
    for(String name : names) {
      if(name.equals("Nemo")) {
        found = true;
        break;
      }
    }
     
    if(found)
      System.out.println("Found Nemo");
    else
      System.out.println("Sorry, Nemo not found");
  }
}
```

上面的 `findNemo()` 方法首先会初始化一个可变 flag 变量，也称之为 `garbage` 变量。我们开发人员经常会把这类变量命名为 `tmp` 或 `temp`，以表明这些变量不应该存在的态度，在上面的例子中，该变量命名为 found。

紧接着，程序会在给定的 `names` 列表当中循环判断，每次处理一个列表元素，如果与待寻找的 `Nemo` 字符串匹配，就把 `flag` 变量置为 `true`，最后通知控制流跳出循环。

以上就是一个非常典型的命令式编程思维书写的代码，也是许多 Java 程序员最熟悉的方式 —— 你需要定义代码的每一步：告诉程序迭代每个元素，然后比较值，然后设置 `flag` 变量，最后跳出循环。当然，命令式编程也有很多优点，最大的优点莫过于为我们提供了完全的`控制权` , 利弊同源，另一方面，我们需要执行所有工作，关注每一个细节逻辑。

## 声明式编程

同样，声明式编程用一句话简单概括就是 `tell what not how`，我们只需要告诉程序要做什么，但实现细节留给底层的函数库。现在我们对之前的代码进行改写:

```
public static void findNemo(List<String> names) {
  if(names.contains("Nemo"))
    System.out.println("Found Nemo");
  else
    System.out.println("Sorry, Nemo not found");
}
```

可以看到，在这个版本当中没有任何的 `garbage` 变量，我们只是使用内置的 `contains()` 方法来完成工作，告诉程序**要做什么** —— 检查集合是否包含我们要寻找的值，但是细节实现留给了底层的方法。

在命令式示例中，我们控制着迭代，而且程序完全按照要求来操作。在声明式版本中，我们不关心工作如何完成，只要它完成即可。`contains()` 的实现可能不同，但只要结果符合预期，我们就会很开心，花更少的精力获得了同样的结果。

训练自己使用声明式编程思维思考，将大大简化向函数式编程思维的过渡，因为函数式编程是以声明式编程为基础的而建立的。

## 函数式编程

以上提到函数式编程是基于声明式的，但简单的使用声明式编程并不等于函数式编程，因为函数式编程`合并了声明式编程和高阶函数`。

### Java 中的高阶函数

通常我们在 Java 中书写函数代码是这样的流程：将对象传递给方法 -> 在方法内创建处理对象 -> 从方法中返回对象。实际上，我们也可以对`函数`执行同样的操作，也就是说，可以将函数传递给方法，在方法内创建函数，并从方法返回函数。

这里有几个概念需要明确一下，`方法`是类的一部分（静态方法或实例方法），但`函数`对于`方法`而言是本地函数，不能特意与类或实例关联。可以接收、创建或返回函数的函数或方法被称为`高阶函数`。有点拗，细品一下。

### 函数式编程示例

来看一段代码：

```
import java.util.*;
 
publicclass UseMap {
  public static void main(String[] args) {
    Map<String, Integer> pageVisits = new HashMap<>();
     
    String page = "https://augustrush8.com";
     
    incrementPageVisit(pageVisits, page);
    incrementPageVisit(pageVisits, page);
     
    System.out.println(pageVisits.get(page));
  }
   
  public static void incrementPageVisit(Map<String, Integer> pageVisits, String page) {
    if(!pageVisits.containsKey(page)) {
       pageVisits.put(page, 0);
    }
     
    pageVisits.put(page, pageVisits.get(page) + 1);
  }
}
```

上面程序当中，`main()` 函数创建了一个 `HashMap`，其中包含一个网站的页面访问次数。每次访问给定页面，`incrementPageVisit()` 方法都会增加计数。

`incrementPageVisit()` 方法是使用命令式编程书写的，其功能职责是：递增给定页面的计数，将该计数存储在 Map 中。该方法不知道给定页面是否有计数，所以它首先会检查是否存在计数。如果不存在，那么它会插入一个 “0” 作为该页面的计数；如果存在的话就获得该计数，递增它，并将新值存储在 Map 中。

而声明式思考要求我们将此方法的设计思路从 `如何做` 转变为 `做什么`。当调用方法 `incrementPageVisit()` 时，我们希望将给定页面的计数初始化为 1 或将运行值递增 1。这就是`做什么`。

所以，我们理所当然就会想 JDK 库当中 Map 接口是否提供了完成以上功能需求的方法，也就是说，寻找一个知道如何完成给定任务的内置方法。

实际上，`merge()` 方法能够实现我们的目标，同时它也是一个`高阶函数`，其函数原型是这样的。

```
* @param key key with which the resulting value is to be associated
     * @param value the non-null value to be merged with the existing value
     *        associated with the key or, if no existing value or a null value
     *        is associated with the key, to be associated with the key
     * @param remappingFunction the function to recompute a value if present
     * @return the new value associated with the specified key, or nullif no
     *         value is associated with the key
efault V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
```

也就是我们之前所说的可以接收函数的函数，那么我们就可以进行函数式编程改造了，看下面代码：

```
public static void incrementPageVisit(Map<String, Integer> pageVisits, String page) {
    pageVisits.merge(page, 1, (oldValue, value) -> oldValue + value);
}
```

以上代码当中，`page` 作为第一个参数传递给 `merge()`，表示该`键`的值应该更新。第二个参数是给该键分配的初始值，如果 Map 中不存在该键，就把第二个参数（本例中是 “1”）赋值给该键。第三个参数（一个 lambda 表达式）接收 map 中该键对应的值作为其参数，并且将该值作为变量传递给 `merge` 方法中的第二个参数，这个 lambda 表达式返回的是它的参数的和，实际上就是递增计数。

至此，`incrementPageVisit()` 方法中的多行代码就被函数式编程简化成了一行代码。由此可见声明式考虑问题有助于我们的跳跃性思维。

## 最后

在 Java 程序中采用函数式方法和语法有许多好处：代码简洁，更富于表达，不易出错，更容易并行化，而且通常比面向对象的代码更容易理解。我们面临的挑战在于将思维方式从命令式编程转变为声明式思考。

尽管函数式编程不那么容易或直观，但你可以通过学习关注你想要`程序实现的目的`而不是关注你希望它`执行的方式`，从而实现思维上的巨大飞跃。通过允许底层函数库管理执行代码，你将逐步直观地了解高阶函数，它们是函数式编程的构建基石。