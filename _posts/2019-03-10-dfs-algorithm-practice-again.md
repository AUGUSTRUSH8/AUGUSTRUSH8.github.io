---
layout: post
title: '算法-走迷宫'
tags: [code]
---

> 又是一次算法练习，主要知识点是深度优先搜索，递归调用，回溯。

### 题目描述

小青蛙有一天不小心落入了一个地下迷宫,小青蛙希望用自己仅剩的体力值P跳出这个地下迷宫。为了让问题简单,假设这是一个n*m的格子迷宫,迷宫每个位置为0或者1,0代表这个位置有障碍物,小青蛙达到不了这个位置;1代表小青蛙可以达到的位置。小青蛙初始在(0,0)位置,地下迷宫的出口在(0,m-1)(保证这两个位置都是1,并且保证一定有起点到终点可达的路径),小青蛙在迷宫中水平移动一个单位距离需要消耗1点体力值,向上爬一个单位距离需要消耗3个单位的体力值,向下移动不消耗体力值,当小青蛙的体力值等于0的时候还没有到达出口,小青蛙将无法逃离迷宫。现在需要你帮助小青蛙计算出能否用仅剩的体力值跳出迷宫(即达到(0,m-1)位置)。

**输入描述**:

```xml
输入包括n+1行:
 第一行为三个整数n,m(3 <= m,n <= 10),P(1 <= P <= 100)
 接下来的n行:
 每行m个0或者1,以空格分隔
```

**输出描述**:

```xml
如果能逃离迷宫,则输出一行体力消耗最小的路径,输出格式见样例所示;如果不能逃离迷宫,则输出"Can not escape!"。 测试数据保证答案唯一
```

**示例**：

输入

```xml
4 4 10 1 0 0 1 1 1 0 1 0 1 1 1 0 0 1 1
```

输出

```xml
[0,0],[1,0],[1,1],[2,1],[2,2],[2,3],[1,3],[0,3]
```

### 解决方案

```java
package com.leyou.test;

import java.util.Iterator;
import java.util.LinkedList;
import java.util.Scanner;
public class Test {
    static int m;
    static int n;
    static int p;
    static int[][] map;
    static int maxRemainEnergy=0;
    static boolean flag=false;
    static String res="";
    static LinkedList<String> list=new LinkedList<>();
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        //接收输入
        m=sc.nextInt();
        n=sc.nextInt();
        p=sc.nextInt();
        map=new int[m][n];
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                map[i][j]=sc.nextInt();
            }
        }

        dfs(0,0,p);
        if(!flag){
            System.out.println("Can not escape!");
        }else{
            System.out.println(res);
        }
    }
    //深度搜索递归函数
    public static synchronized void dfs(int x,int y,int energy){
        //直接条件判断
        if(energy<0||x<0||x>m-1||y<0||y>n-1||map[x][y]==0){
            return;
        }
        list.offer("["+x+","+y+"]");
        map[x][y]=0;
        //结果判断
        if (x==0&&y==n-1){
            //当前剩余能量还大于之前历史最大剩余能量的话
            if(energy>maxRemainEnergy){
                //更新历史最大剩余能量
                maxRemainEnergy=energy;
            }
            //更新结果集
            update();
            //重置地图
            map[x][y]=1;
            list.removeLast();
            //更新状态标志
            flag=true;
            return;
        }
        //当前位置并不是最后位置,需要继续深度搜索
        //上边
        dfs(x-1,y,energy-3);
        //下边
        dfs(x+1,y,energy);
        //左边
        dfs(x,y-1,energy-1);
        //右边
        dfs(x,y+1,energy-1);
        //当前位置下所有可能性已经搜索完了，需要回溯了
        //把矩阵原来标为不可通行的重新变为可通行
        map[x][y]=1;
        list.removeLast();

    }
    //因为要回溯，所以结果集需要及时更新
    public static void update(){
        //遍历linkedList
        Iterator iterator=list.iterator();
        StringBuilder sb=new StringBuilder();
        while (iterator.hasNext()){
            sb.append(iterator.next()+",");
        }
        //最后多加了一个逗号，把它去掉
        sb.deleteCharAt(sb.length()-1);
        res=sb.toString();
    }

}
```

