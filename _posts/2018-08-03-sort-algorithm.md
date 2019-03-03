---
layout: post
title: 'Java实现八大排序算法'
tags: [read]
---

### 冒泡排序

```java
class solution{
    public void BubbleSort(int[] a){
        if(a==null){
            return;
        }
        //记录冒泡排序右边界
        int right=a.length-1;
        //循环终止条件
        while(right>0){
            //交换位置标志位
            int pos=0;
            for(start=0;start<right;start++){
                if(a[start]>a[start+1]){
                    swap(a,start,start+1);
                    pos=start;
                }
            }
            //更新右边界
            right=pos;
        }
    }
    public void swap(int[] a,int i,int j){
        int temp=a[start];
        a[start]=a[start+1];
        a[start+1]=temp;
    }
}
```

### 直接插入排序

将一个记录插入到已排序好的有序表中，从而得到一个新的记录数增1的有序表。即：先将序列的第1个记录看成是一个有序的子序列，然后从第2个记录逐个进行插入，直至整个序列有序为止。

```java
class solution{
    public void InsertSort(int[] a){
        for(right=1;right<a.length;right++){
            if(a[right]<a[right-1]){
                //记录下待插入的这个数值
                int temp=a[right];
                //后移一位
                a[right]=a[right-1];
                //向左侧循环移位比较
                int left=right-1;
                for(;left>=0&&temp<a[left];left--){
                    a[left+1]=a[left];
                }
                //找到了正确的插入位置
                a[left+1]=temp;

            }
        }
    }
}
```

### 选择排序

1. 选出最小的元素，与数组第一个位置交换
2. 选出第i小的元素，与数组第i个位置交换
3. 直到第n-1个元素，与第n个元素比较为止

```java
class solution{
    public void SelectSort(int[] a){
        for(int start=0;start<a.length,start++){
            int key=findMinVAlue(a,start);
            swap(a,key,start);
        }

    }
    public void findMinVAlue(int[] a,int start){
        int key=start;
        for(int i=start;i<a.length;i++){
            if(a[start]>a[start+1]){
                key=a[key]>a[i]?i:key;
            }
        }
        return key;
    }
    public void swap(int[] a,int i,int j){
        int temp=a[start];
        a[start]=a[start+1];
        a[start+1]=temp;
    }
}
```

### 快速排序

```java
class solution{
    public void quickSort(int[] a){
        quickSort0(0,a.length-1,a);
    }
    public void quickSort0(int low,int high,int[] a){
        if(low<high){
            int pos=partition(a,low,high);
            quickSort0(low,pos-1,a);
            quickSort0(pos+1,high,a);
        }
    }
    public int partition(int[] a,int low,int high){
        int picotKey=a[low];
        while(low<high){
            while(low<high&&a[high]>picotKey){
                high--;
            }
            swap(a,low,high);
            while(low<high&&a[low]<picotKey){
                low++;
            }
            swap(a,low,high);
        }
        return low;
    }
    public void swap(int[] a,int i,int j){
        int temp=a[start];
        a[start]=a[start+1];
        a[start+1]=temp;
    }
}
```

### 归并排序

```java
class solution{
    public void mergeSort(int[] a){
        mergeSort0(a,0,a.length-1);
    }
    public void mergeSort0(int[] a,int low,int high){
        if(low<high){
            int mid=(low+high)/2;
            mergeSort0(a,low,mid);
            mergeSort0(a,mid+1,high);
            merge(a,low,mid,high);
        }
    }
    private void merge(int[] a, int start1, int mid, int right) {
    int[] tmp = new int[a.length];
    
    int k = start1; // tmp的初始下标
    int start = start1; // 记录初始位置
    int start2 = mid + 1; // 第2个数组的起始位置
    
    for(; start1 <= mid && start2 <= right; k++) {
        tmp[k] = a[start1] < a[start2] ? a[start1++] : a[start2++];
    }
    
    // 左边剩余的合并
    while (start1 <= mid) {
        tmp[k++] = a[start1++];
    }
    // 右边剩余的合并
    while (start2 <= right) {
        tmp[k++] = a[start2++];
    }
    
    // 复制数组
    while (start <= right) {
        a[start] = tmp[start];
        start++;
    }
}
}
```

### 堆排序

堆排序不太好理解，具体参考：https://www.jianshu.com/p/be0c51a798a8 图例说明

```java
class solution{
    public void heapSort(int[] a){
        buildHeap(a,a.length);
        for(int i=a.length-1;i>0;i--){
            swap(a,0,i);
            adjustHeap(a,0,i);
            System.out.println(Arrays.toString(a));
        }
    }

    private static void adjustHeap(int[] a,int s,int length){
        int tmp=a[s];
        int child=2*s+1;//左孩子节点的位置
        while(child<length){
            // 如果有右孩子，同时右孩子值 > 左孩子值
            if(child+1<length&&a[child]<a[child+1])}{
                child++;
            }
            // 较大的子结点>父节点
            if(a[s]<a[child]){
                a[s]=a[child];// 替换父节点
                s=child;// 重新设置，待调整的下一个结点位置
                child=2*s+1;
            }else{
                break;
            }
            a[s]=tmp;
        }
    }
    /**
    * 初始堆进行调整 将a[0...length-1]建成堆
    * 调整完后，第一个元素是序列最大的元素
    */
    private void buildHeap(int[] a,int length){
        // 最后一个有孩子结点的位置是 i = (length - 1) / 2
        for(int i=(length-1)/2;i>=0;i++){
            adjustHeap(a,i,length);
        }
    }
    public void swap(int[] a, int i, int j) {
        int tmp = a[i];
        a[i] = a[j];
        a[j] = tmp;
    }
}
```

