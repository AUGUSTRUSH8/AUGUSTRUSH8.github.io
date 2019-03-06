---
layout: post
title: 'HashSet类解读'
tags: [read]
---

### 元素不能重复

Set中的元素，不能重复

```java
package collection;
  
import java.util.HashSet;
  
public class TestCollection {
    public static void main(String[] args) {
         
        HashSet<String> names = new HashSet<String>();
         
        names.add("gareen");
         
        System.out.println(names);
         
        //第二次插入同样的数据，是插不进去的，容器中只会保留一个
        names.add("gareen");
        System.out.println(names);
    }
}
```

### 没有顺序

Set中的元素，没有顺序。 严格的说，是没有按照元素的插入顺序排列。HashSet的具体顺序，既不是按照插入顺序，也不是按照hashcode的顺序。

**HashSet源代码**中的部分注释：

```java
/**
 * It makes no guarantees as to the iteration order of the set; 
 * in particular, it does not guarantee that the order will remain constant over time. 
*/
```

翻译：**不保证Set的迭代顺序; 确切的说，在不同条件下，元素的顺序都有可能不一样**

换句话说，同样是插入0-9到HashSet中， 在JVM的不同版本中，看到的顺序都是不一样的。 所以在开发的时候，不能依赖于某种**臆测的顺序**，这个顺序本身是**不稳定的**

```java
package collection;
 
import java.util.HashSet;
 
public class TestCollection {
    public static void main(String[] args) {
        HashSet<Integer> numbers = new HashSet<Integer>();
 
        numbers.add(9);
        numbers.add(5);
        numbers.add(1);
 
        // Set中的元素排列，不是按照插入顺序
        System.out.println(numbers);
 
    }
}
```

### 遍历

Set不提供get()来获取指定位置的元素，所以遍历需要用到**迭代器，或者增强型for循环**

```java
package collection;
  
import java.util.HashSet;
import java.util.Iterator;
  
public class TestCollection {
    public static void main(String[] args) {
        HashSet<Integer> numbers = new HashSet<Integer>();
         
        for (int i = 0; i < 20; i++) {
            numbers.add(i);
        }
         
        //Set不提供get方法来获取指定位置的元素
        //numbers.get(0)
         
        //遍历Set可以采用迭代器iterator
        for (Iterator<Integer> iterator = numbers.iterator(); iterator.hasNext();) {
            Integer i = (Integer) iterator.next();
            System.out.println(i);
        }
         
        //或者采用增强型for循环
        for (Integer i : numbers) {
            System.out.println(i);
        }
         
    }
}
```

### HashSet和HashMap的关系

通过观察HashSet的源代码，可以发现HashSet自身并没有独立的实现，而是在里面封装了一个Map。HashSet是作为Map的key而存在的，而value是一个命名为PRESENT的static的Object对象，因为是一个类属性，所以只会有一个。

```java
package collection;
 
import java.util.AbstractSet;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Set;
 
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    //HashSet里封装了一个HashMap
    private  HashMap<E,Object> map;
 
    private static final Object PRESENT = new Object();
 
    //HashSet的构造方法初始化这个HashMap
    public HashSet() {
        map = new HashMap<E,Object>();
    }
 
    //向HashSet中增加元素，其实就是把该元素作为key，增加到Map中
    //value是PRESENT，静态，final的对象，所有的HashSet都使用这么同一个对象
    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
 
    //HashSet的size就是map的size
    public int size() {
        return map.size();
    }
 
    //清空Set就是清空Map
    public void clear() {
        map.clear();
    }
     
    //迭代Set,就是把Map的键拿出来迭代
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }
 
}
```

### 使用HashSet

题目：

```java
创建一个长度是100的字符串数组
使用长度是2的随机字符填充该字符串数组
统计这个字符串数组里重复的字符串有多少种
```

```java
 
import java.util.HashSet;
 
public class TestCollection {
    public static void main(String[] args) {
 
        String[] ss = new String[100];
        // 初始化
        for (int i = 0; i < ss.length; i++) {
            ss[i] = randomString(2);
        }
        // 打印
        for (int i = 0; i < ss.length; i++) {
            System.out.print(ss[i] + " ");
            if (19 == i % 20)
                System.out.println();
        }
 
        HashSet<String> result = new HashSet<>();
 
        for (String s1 : ss) {
            int repeat = 0;
            for (String s2 : ss) {
                if (s1.equalsIgnoreCase(s2)) {
                    repeat++;
                    if (2 == repeat) {
                        // 当repeat==2的时候，就找到了一个非己的重复字符串
                        result.add(s2);
                        break;
                    }
                }
            }
        }
 
        System.out.printf("总共有 %d种重复的字符串%n", result.size());
        if (result.size() != 0) {
            System.out.println("分别是：");
            for (String s : result) {
                System.out.print(s + " ");
            }
        }
    }
 
    private static String randomString(int length) {
        String pool = "";
        for (short i = '0'; i <= '9'; i++) {
            pool += (char) i;
        }
        for (short i = 'a'; i <= 'z'; i++) {
            pool += (char) i;
        }
        for (short i = 'A'; i <= 'Z'; i++) {
            pool += (char) i;
        }
        char cs[] = new char[length];
        for (int i = 0; i < cs.length; i++) {
            int index = (int) (Math.random() * pool.length());
            cs[i] = pool.charAt(index);
        }
        String result = new String(cs);
        return result;
    }
}
```

