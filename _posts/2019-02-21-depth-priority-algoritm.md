---
layout: post
title: '正方形数组的数目'
tags: [read]
---

### 深度优先搜索

给定一个非负整数数组 `A`，如果该数组每对相邻元素之和是一个完全平方数，则称这一数组为*正方形*数组。

返回 A 的正方形排列的数目。两个排列 `A1` 和 `A2` 不同的充要条件是存在某个索引 `i`，使得 A1[i] != A2[i]。

**示例 1：**

```xml
输入：[1,17,8]
输出：2
解释：
[1,8,17] 和 [17,8,1] 都是有效的排列。
```

**示例 2：**

```xml
输入：[2,2,2]
输出：1
```

#### 思路（核心：回溯）

构造一张图，包含所有的边 i 到 j ，如果满足 A[i] + A[j] 是一个完全平方数。我们的目标就是求这张图的所有**哈密尔顿路径**，**即经过图中所有点仅一次的路径**。

#### 算法

我们使用 `count` 记录对于每一种值还有多少个节点等待被访问，与一个变量 `todo` 记录还剩多少个节点等待被访问。

#### Java代码实现

```java
public class numSquarefulPerms {
    //定义存储数组中某个元素个数的存储结构
    private Map<Integer,Integer> count;
    //定义存储能够与某个特定元素组成完全平方数的数据集合
    private Map<Integer,List<Integer>> graph;
    public int numSquarefulPerms(int[] A) {
        //为全局变量分配空间
        count=new HashMap<>();
        graph=new HashMap<>();
        //求取并记录数组的大小
        int N=A.length;
        //遍历整个数组，统计每个元素的个数
        for(int x:A){
            count.put(x,count.getOrDefault(x,0)+1);
        }
        //遍历整个数组，寻找能够与每个元素组成完全平方数的数据集
        for(int x: count.keySet()){
            graph.put(x,new ArrayList());
        }
        for(int x:count.keySet()){
            for(int y:count.keySet()){
                int r=(int) (Math.sqrt(x+y)+0.5);
                if(r*r==x+y){
                    graph.get(x).add(y);
                }
            }
        }
        /**
         * 开始以count keySet当中的每一个元素为根往下
         * 深度搜索能够满足条件的路径，并累加路径数
         */
        //先定义路径数
        int ans=0;
        //开始搜索
        for(int x:count.keySet()){
            ans+=dfs(x,N-1);
        }
        return ans;
    }
    /**
     * 深度优先搜索函数
     * @param  x    [description] x表示排在当前搜索的根节点
     * @param  todo [description] todo表示搜索路径的剩余节点数,它很重要，有了它算法才有收敛的可能
     * @return      [description]
     */
    private int dfs(int x,int todo){
        //先把count当中对应x的值减一
        count.put(x,count.get(x)-1);
        //定义返回结果,注意这个ans和下面那个ans定义的区别和用意
        int ans=1;
        //开始深度搜索了
        if(todo!=0){
            ans=0;
            for(int y:graph.get(x)){
                if(count.get(y)!=0){
                    ans+=dfs(y,todo-1);
                }
            }
        }

        count.put(x,count.get(x)+1);
        return ans;
    }
    @Test
    public void test(){
        int[] array={2,2,2};
        System.out.println(numSquarefulPerms(array));
    }
}

```

#### 关键代码

```java
/**
     * 深度优先搜索函数
     * @param  x    [description] x表示排在当前搜索的根节点
     * @param  todo [description] todo表示搜索路径的剩余节点数,它很重要，有了它算法才有收敛的可能
     * @return      [description]
     */
    private int dfs(int x,int todo){
        //先把count当中对应x的值减一
        count.put(x,count.get(x)-1);
        //定义返回结果,注意这个ans和下面那个ans定义的区别和用意
        int ans=1;
        //开始深度搜索了
        if(todo!=0){
            ans=0;
            for(int y:graph.get(x)){
                if(count.get(y)!=0){
                    ans+=dfs(y,todo-1);
                }
            }
        }

        count.put(x,count.get(x)+1);
        return ans;
    }
```

仔细理解上面这一段代码！

#### 再解释

注意这其中回溯的思想的运用，类似于我需要遍历某颗多叉树，这颗多叉树的高度并不高，我可以挨个从它的子节点往下遍历，搜索每一个可能性，某一条路径走通了，就表示这条路径是可行的，一旦某个路径走不通了（没有可行解了），那么该条路径的所有其他可能子节点都可以被剪枝，不用再搜索该节点以后的其他子树了。当遍历完所有以后，统计可行路径的个数即可（当然是一边搜索一边计数咯）