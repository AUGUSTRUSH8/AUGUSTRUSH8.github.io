---
layout: post
title: '腐烂的橘子'
tags: [read]
---

这是一个有趣的算法题，其主要的思想是宽度优先搜索 :wink:

### 题目描述

在给定的网格中，每个单元格可以有以下三个值之一：

- 值 `0` 代表空单元格；
- 值 `1` 代表新鲜橘子；
- 值 `2` 代表腐烂的橘子。

每分钟，任何与腐烂的橘子（在 4 个正方向上）相邻的新鲜橘子都会腐烂。

返回直到单元格中没有新鲜橘子为止所必须经过的最小分钟数。如果不可能，返回 `-1`。

### 输入输出示例

- 例一

```txt
输入：[[2,1,1],[1,1,0],[0,1,1]]
输出：4
```

- 例二

```txt
输入：[[2,1,1],[0,1,1],[1,0,1]]
输出：-1
解释：左下角的橘子（第 2 行， 第 0 列）永远不会腐烂，因为腐烂只会发生在 4 个正向上。
```

- 例三

```txt
输入：[[0,2]]
输出：0
解释：因为 0 分钟时已经没有新鲜橘子了，所以答案就是 0 。
```

### 图形解释说明

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2019/02/16/oranges.png)

### 思路

每一轮，腐烂将会从每一个烂橘子蔓延到与其相邻的新鲜橘子上。一开始，腐烂的橘子拥有深度为 0，每一轮腐烂会从腐烂橘子传染到之相邻新鲜橘子上，并且设置这些新的腐烂橘子的深度为自己深度 +1，我们想知道完成这个过程之后的最大深度值是多少。

### 算法

我们可以用一个宽度优先遍历来建模这一过程。因为我们总是选择去使用深度值最小的（且之前未使用过的）腐烂橘子去腐化新鲜橘子，如此保证每一个橘子腐烂时的深度标号也是最小的。

我们还应该检查最终状态下，是否还有新鲜橘子。

### Java算法实现

```java
public class MinimumDepth {
    //方向移动数组
    private int[] array1={1,0,-1,0};
    private int[] array2={0,1,0,-1};
    public int orangesRottrng(int[][] grid) {
        //定义存储腐烂橘子的数据结构，这里使用ArrayDeque
        Queue<Integer> queue=new ArrayDeque();
        //定义存储深度用的数据结构，这里使用HashMap
        Map<Integer,Integer> map=new HashMap();
        //遍历二维数组，找到其中的烂橘子，并记录到queue当中去
        for(int r=0;r<grid.length;r++){
            for(int c=0;c<grid[0].length;c++){
                //找到了腐烂橘子的位置
                if(grid[r][c]==2){
                    //唯一标识该位置
                    int position=r*grid[0].length+c;
                    queue.add(position);
                    //同时使得该位置在hashmap中的初始值置为0
                    map.put(position,0);
                }
            }
        }
        //定义存储最短时间（最小深度）橘子腐烂的变量
        int ans=0;
        //宽度优先遍历
        while(!queue.isEmpty()){
            int code=queue.remove();
            //解算出该烂橘子的横纵坐标位置
            int r=code/grid[0].length;
            int c=code%grid[0].length;
            //开始寻找上下左右四个方向位置的橘子情况
            for(int i=0;i<4;i++){
                int newr=r+array1[i];
                int newc=c+array2[i];
                int newCode=newr*grid[0].length+newc;
                //判断该新位置是否符合条件
                if(newr>=0&&newr<grid.length&&newc>=0&&newc<grid[0].length&&grid[newr][newc]==1){
                    //把它变成烂橘子
                    grid[newr][newc]=2;
                    //继续向queue当中添加，好使得继续往下遍历
                    queue.add(newCode);
                    //向Map里面记录该节点在初始遍历条件下的深度值,在上一个烂橘子深度值的基础之上加1
                    map.put(newCode,map.get(code)+1);
                    //所有的遍历完以后返回结果
                    ans=map.get(newCode);
                }
            }
        }
        //最后还要检查所有的方框当中是否还存在好橘子
        for(int i=0;i<grid.length;i++){
            for(int j=0;j<grid[0].length;j++){
                if(grid[i][j]==1){
                    return -1;
                }
            }
        }
        return ans;
    }
    @Test
    public void test(){
        int[][] array={{0,2}};

        System.out.println(orangesRottrng(array));
    }
}
```

### 测试用例

如以上输入输出示例所示

### 知识点

数据结构：ArrayDeque

> 解释：`ArrayDeque`是`Deque`接口的一个实现，使用了可变数组，所以没有容量上的限制。同时，`ArrayDeque`是线程不安全的，在没有外部同步的情况下，不能再多线程环境下使用。`ArrayDeque`是`Deque`的实现类，**可以作为栈来使用**，效率高于`Stack`；**也可以作为队列来使用**，效率高于`LinkedList`。需要注意的是，`ArrayDeque`不支持`null`值。