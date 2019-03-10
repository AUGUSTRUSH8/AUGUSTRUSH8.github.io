---
layout: post
title: '算法-优雅的点'
tags: [code]
---

### 题目描述

小易有一个圆心在坐标原点的圆，小易知道圆的半径的平方。小易认为在圆上的点而且横纵坐标都是整数的点是优雅的，小易现在想寻找一个算法计算出优雅的点的个数，请你来帮帮他。
例如：半径的平方如果为25
优雅的点就有：(+/-3, +/-4), (+/-4, +/-3), (0, +/-5) (+/-5, 0)，一共12个点。

**输入描述**:

```xml
输入为一个整数，即为圆半径的平方,范围在32位int范围内。
```

**输出描述**:

```xml
输出为一个整数，即为优雅的点的个数
```

**示例1**

输入

```xml
25
```

输出

```xml
12
```

### 解析

本题逻辑上不难，难点在于对数据类型的认识和把握，在运用上需要特别注意

### 代码

```java
package com.leyou.test;
import java.util.Scanner;

public class Test {
    static int rsquare=0;
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        rsquare=sc.nextInt();
        int sum=findGracePoint();
        System.out.println(sum);
    }
    private static int findGracePoint(){
        int count=0;
        double r=Math.sqrt(rsquare);
        for(int i=0;i<r;i++){
            double j=Math.sqrt(rsquare-i*i);
            if(Math.abs(j-Math.round(j))<0.000000001){
                count++;
            }
        }
        return 4*count;
    }
}
```

如上，需要特别注意两点：

```java
double r=Math.sqrt(rsquare);
```

在声明Math.sqrt()的接收数据类型时，需要是`double`类型。

第二：

```java
Math.abs(j-Math.round(j))<0.000000001
```

这里在做判断的时候不要忘了Math.abs()函数的使用，如我在最开始的时候并没有加上这样一句，结果不能通过测试，细细思考一下就会发现问题所在。



