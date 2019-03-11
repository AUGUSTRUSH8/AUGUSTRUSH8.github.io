---
layout: post
title: '最优化商家利益算法'
tags: [code]
---

### 题目描述

某餐馆有n张桌子，每张桌子有一个参数：a 可容纳的最大人数； 有m批客人，每批客人有两个参数:b人数，c预计消费金额。 在不允许拼桌的情况下，请实现一个算法选择其中一部分客人，使得总预计消费金额最大

**输入描述**:

```xml
输入包括m+2行。 第一行两个整数n(1 <= n <= 50000),m(1 <= m <= 50000) 第二行为n个参数a,即每个桌子可容纳的最大人数,以空格分隔,范围均在32位int范围内。 接下来m行，每行两个参数b,c。分别表示第i批客人的人数和预计消费金额,以空格分隔,范围均在32位int范围内。
```

**输出描述**:

```xml
输出一个整数,表示最大的总预计消费金额
```

示例1

输入

```xml
3 5 2 4 2 1 3 3 5 3 7 5 9 1 10
```

输出

```xml
20
```

### 题目分析

本题刚开始看可能有点懵，因为里面涉及到的东西第一遍读下来感觉信息量稍微有点多，但全局来看就是一个最优化的问题--如何在商家既有的硬件条件之下，让商家的消费流量更多。

从这道题来看，用贪心就可以解决问题了，关键的问题是注意时间复杂度，稍不留意可能就因为时间超标导致无法全部AC，后面会有说明。

### 代码

```java
package com.leyou.test;
import java.util.*;

public class Test {
    static int table=0;//桌子数量
    static int batch=0;//有多少批客户
    static int[] tableSize;//每个桌子可容纳的客户数目
    static int[][] each;//每一批客户有多少人，愿意最大消费多少钱
    static boolean[] occupied;//记录每张桌子的占有状态
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        table=sc.nextInt();//多少桌
        batch=sc.nextInt();//多少批客人
        tableSize=new int[table];
        for (int i=0;i<table;i++){
            tableSize[i]=sc.nextInt();
        }
        each=new int[batch][2];
        for(int j=0;j<batch;j++){
            for(int k=0;k<2;k++){
                each[j][k]=sc.nextInt();
            }
        }
        occupied=new boolean[table];
        Arrays.sort(tableSize);
        Arrays.sort(each, new Comparator<int[]>() {
            @Override
            public int compare(int[] o1, int[] o2) {
                return o2[1]-o1[1];
            }
        });
        long result=getMaximumMoney();
        System.out.println(result);
    }
    public static Long getMaximumMoney(){
        long res=0L;
        //循环每一批客户，因为客户是按照消费水平进行逆序排列的，所以这就是典型的贪心算法了
        for(int i=0;i<batch;i++){
            if(each[i][0]>tableSize[table-1]){
                continue;
            }
            int index=binarySearch(tableSize,each[i][0]);
            for(int j=index;j<table;j++){
                if(occupied[j]==false){
                    occupied[j]=true;
                    res+=each[i][1];
                    break;
                }
            }
        }
        return res;
    }
    //这里采用二分搜索的方式判断新来的一批客户的数目最低的桌子容量需求。时间复杂度需要严格控制，不然很难AC
    public static int binarySearch(int[] arr,int tar){
        int low=0;
        int high=table-1;
        int mid=0;
        //这里需要注意，刚开始我掉了一个=号就导致结果有误
        while (low<=high){
            mid=(low+high)>>1;
            if(arr[mid]>=tar){
                high=mid-1;
            }else{
                low=mid+1;
            }
        }
        return low;
    }


}
```

现在挨个介绍以上代码需要注意的地方：

- 第一，当我们需要对一维数组进行排序的时候，可以不用自己去写循环，直接借用Arrays工具类就可以

```java
Arrays.sort(tableSize);
```

>Arrays.sort(int[] a)
>
>这种形式是对一个数组的所有元素进行排序，并且是按**从小到大**的顺序。

**扩展**，其他方式的用法：

**扩展一**：

>Arrays.sort(int[] a, int fromIndex, int toIndex)
>
>这种形式是对数组部分排序，也就是对数组a的下标从fromIndex到toIndex-1的元素排序，注意：下标为toIndex的元素不参与排序！

**扩展二**：

>public static <T> void sort(T[] a,int fromIndex, *int toIndex,  Comparator<? super T> c)*
>
>上面有一个拘束，就是排列顺序只能是从小到大，如果我们要从大到小，就要使用这种方式

**用法示例**：

```java
 1 package test;
 2 
 3 import java.util.Arrays;
 4 import java.util.Comparator;
 5 
 6 public class Main {
 7     public static void main(String[] args) {
 8         //注意，要想改变默认的排列顺序，不能使用基本类型（int,double, char）
 9         //而要使用它们对应的类
10         Integer[] a = {9, 8, 7, 2, 3, 4, 1, 0, 6, 5};
11         //定义一个自定义类MyComparator的对象
12         Comparator cmp = new MyComparator();
13         Arrays.sort(a, cmp);
14         for(int i = 0; i < a.length; i ++) {
15             System.out.print(a[i] + " ");
16         }
17     }
18 }
19 //Comparator是一个接口，所以这里我们自己定义的类MyComparator要implents该接口
20 //而不是extends Comparator
21 class MyComparator implements Comparator<Integer>{
22     @Override
23     public int compare(Integer o1, Integer o2) {
24         //如果o1小于o2，我们就返回正值，如果o1大于o2我们就返回负值，
25         //这样颠倒一下，就可以实现反向排序了
26         if(o1 < o2) { 
27             return 1;
28         }else if(o1 > o2) {
29             return -1;
30         }else {
31             return 0;
32         }
33     }
34     
35 }
```

- 第二，针对二位数组的排序方式，二维数组同样也是可以借助工具类的，只需要指定排序的位置就行

```java
Arrays.sort(each, new Comparator<int[]>() {
            @Override
            public int compare(int[] o1, int[] o2) {
                return o2[1]-o1[1];
            }
        });
```

以上，道理其实和一维数组类似的，只是泛型的指定变成了一维数组。

- 第三，如以上代码注释里面所说，在写二分搜索算法的时候，一定要注意逻辑的严密性，最初我因为写错了`while`循环判断条件而导致无法AC

```java
while (low<=high)//最初我漏掉了一个等号
```

- 第四，注意mid的写法问题，同样是在二分搜索算法当中

```java
mid=(low+high)>>1;
```

写成上述代码的形式一方面是不大保险，`low+high`容易超过数据类型的大小限制。而采用下面的形式一方面是要安全一些，另一方面则是运行时间要小一些（虽然很微妙）

将上面这段代码我写成：

```java
mid=low+(high-low)>>1;
```

就会出现问题，这种写法是**错误**的，

正确的写法是写成

```java
mid=low+((high-low)>>1);
```

此时运行就是正确的，而且算法的运行时间也降到了七百多毫秒。