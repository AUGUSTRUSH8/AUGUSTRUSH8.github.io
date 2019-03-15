---
layout: post
title: '计数排序和桶排序'
tags: [read]
---

### 比较排序和非比较排序的区别

常见的快速排序、归并排序、堆排序、冒泡排序等属于比较排序。在排序的最终结果里，元素之间的次序依赖于它们之间的比较。每个数都必须和其他数进行比较，才能确定自己的位置。
在**冒泡排序**之类的排序中，问题规模为n，又因为需要比较n次，所以平均时间复杂度为O(n²)。在**归并排序、快速排序**之类的排序中，问题规模通过分治法消减为logN次，所以时间复杂度平均**O(nlogn)**。
比较排序的优势是，适用于各种规模的数据，也不在乎数据的分布，都能进行排序。可以说，比较排序适用于一切需要排序的情况。

计数排序、基数排序、桶排序则属于非比较排序。非比较排序是通过确定每个元素之前，应该有多少个元素来排序。针对数组arr，计算arr[i]之前有多少个元素，则唯一确定了arr[i]在排序后数组中的位置。
非比较排序只要确定每个元素之前的已有的元素个数即可，所有一次遍历即可解决。算法时间复杂度**O(n)**。
非比较排序时间复杂度低，但由于非比较排序需要占用空间来确定唯一位置。所以对**数据规模**和**数据分布**有一定的要求。

### 计数排序

- 适用范围

计数排序需要占用大量空间，它仅适用于数据比较集中的情况。比如 [0~100]，[10000~19999] 这样的数据。

- 过程分析

计数排序的基本思想是：`对每一个输入的元素arr[i]，确定小于 arr[i] 的元素个数`。
所以可以直接把 arr[i] 放到它输出数组中的位置上。假设有5个数小于 arr[i]，所以 arr[i] 应该放在数组的第6个位置上。

- 算法实现

需要三个数组:
待排序数组 int[] arr = new int[]{4,3,6,3,5,1};
辅助计数数组 int[] help = new int[max - min + 1]; //该数组大小为待排序数组中的最大值减最小值+1
输出数组 int[] res = new int[arr.length];

1.求出待排序数组的最大值max=6， 最小值min=1
2.实例化辅助计数数组help，help数组中每个下标对应arr中的一个元素，**help用来记录每个元素出现的次数**
3.计算 arr 中每个元素在help中的位置 position = arr[i] - min，此时 help = [1,0,2,1,1,1]; （3出现了两次，2未出现）
4.根据 help 数组求得排序后的数组，此时 res = [1,3,3,4,5,6]

```java
public class CountSort {
    public static void main(String[] args) {
        //写测试用例
    }
    public static int[] countSort(int[] arr){
        if(arr==null||arr.length==0){
            return null;
        }
        //求取待排序数组中的最大值和最小值
        int max=Integer.MIN_VALUE;
        int min=Integer.MIN_VALUE;
        for(int i=0;i<arr.length;i++){
            max=Math.max(max,arr[i]);
            min=Math.min(min,arr[i]);
        }
        //开辟辅助数组
        int[] help=new int[max];
        //计算每个数字出现的次数,映射到辅助数组当中去
        for(int i=0;i<arr.length;i++){
            //计算在辅助数组当中的索引位置
            int index=arr[i]-min;
            help[index]++;
        }
        //从辅助数组当中映射输出到输出数组当中
        int index=0;
        for(int i=0;i<help.length;i++){
            while (help[i]-->0){
                arr[index++]=i+min;//又重新把值映射回来
            }
        }
        return arr;
    }
}

```

### 桶排序

- 适用范围

桶排序可用于**最大最小值相差较大的数据情况**，比如[9012,19702,39867,68957,83556,102456]。
但桶排序要求数据的**分布必须均匀**，否则可能导致数据都集中到一个桶中。比如[104,150,123,132,20000], 这种数据会导致前4个数都集中到同一个桶中。导致桶排序失效。

- 过程分析

桶排序的基本思想是：`把数组 arr 划分为n个大小相同子区间（桶），每个子区间各自排序，最后合并`。
计数排序是桶排序的一种特殊情况，可以把计数排序当成每个桶里只有一个元素的情况。

1.找出待排序数组中的最大值max、最小值min
2.我们使用 动态数组ArrayList 作为桶，桶里放的元素也用 ArrayList 存储。桶的数量为(max-min)/arr.length+1
3.遍历数组 arr，计算每个元素 arr[i] 放的桶
4.每个桶各自排序
5.遍历桶数组，把排序好的元素放进输出数组

- 代码实现

```java
import java.util.ArrayList;
import java.util.Collections;

public class BucketSort {
    public static void main(String[] args) {
        //编写测试用例
    }
    private static void bucketSort(int[] arr){
        //找出最大最小值
        int max=Integer.MIN_VALUE;
        int min=Integer.MAX_VALUE;
        for(int i=0;i<arr.length;i++){
            max=Math.max(max,arr[i]);
            min=Math.min(min,arr[i]);
        }
        //计算桶数
        int bucketNum=(max-min)/arr.length+1;
        ArrayList<ArrayList<Integer>> bucket=new ArrayList<>();
        for(int i=0;i<bucketNum;i++){
            bucket.add(new ArrayList<Integer>());
        }
        //将每个元素放入桶中
        for(int i=0;i<arr.length;i++){
            //计算放入哪个桶中
            int num=(arr[i]-min)/arr.length;
            bucket.get(num).add(arr[i]);
        }
        //对每个桶进行排序
        for(int i=0;i<bucket.size();i++){
            Collections.sort(bucket.get(i));
        }
        System.out.println(bucket.toString());
    }

}

```

