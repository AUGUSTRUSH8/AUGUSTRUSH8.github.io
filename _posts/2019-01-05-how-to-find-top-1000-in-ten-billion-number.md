---
layout: post
title: '怎么在10亿个数当中找到Top1000'
tags: [code]
---

### 需求描述

有一个文件，里面存储了10亿个商品的销售数据，找出前1000大的数据

### 解决办法

- 第一种，先排序，然后取前1000个数，或者部分排序，只排除前1000个数

**利弊**：时间复杂度都比较高，思考方式上比较直观

- 第二种，可以用分治法，这有点类似快排中partition的操作。随机选一个数t，然后对整个数组进行partition，会得到两部分，前一部分的数都大于t，后一部分的数都小于t。

![](http://image.augustrush8.com/images/quicksort1.png){:.center}

如果说前一部分总数大于1000个，那就继续在前一部分进行partition寻找。如果前一部分的数小于1000个，那就在后一部分再进行partition，寻找剩下的数。

![](http://image.augustrush8.com/images/quicksort2.png){:.center}

时间复杂度分析：partition的过程，时间是o(n)。我们在进行第一次partition的时候需要花费n，第二次partition的时候，数据量减半了，所以只要花费n/2，同理第三次的时候只要花费n/4，以此类推。而n+n/2+n/4+...显然是小于2n的，所以这个方法的渐进时间只有o(n)

![](http://image.augustrush8.com/images/quicksort3.png){:.center}

继续优化？如果计算机内存只有2个G，怎么办？

思考一下，10亿个数据，int32存储，大概需要4G内存，内存一次是放不下的。

- 第三种，将数据读取后分别写入两个文件

![](http://image.augustrush8.com/images/quicksort4.png){:.center}

由于内存放不下，只好把数据放到文件，然后再从其中一个文件读出来进行partition。

**弊端**：这样的话，磁盘会有多次读写，IO性能消耗很大，效率很低。

- 第四种，使用分布式的思想，将数据切分，然后在多台机器上分别计算前1000大的数，最后再把这些数汇总

![](http://image.augustrush8.com/images/quicksort5.png){:.center}

将数据分散在多台机器上并行计算，再汇总结果。

但是，如果只有一台机器呢？

- 第五种，使用堆排序，维护一个1000数的最小堆

![](http://image.augustrush8.com/images/quicksort6.png){:.center}

先取前n个数，构成小顶堆，然后从文件中读取数据，并且和堆顶大小相比，如果比堆顶小，就直接丢弃，如果比堆顶大，就替换堆顶，然后调整此小顶堆。

![](http://image.augustrush8.com/images/heapsort1.png){:.center}

下一个读取的数字比18大，把18替换下来。

![](http://image.augustrush8.com/images/heapsort2.png){:.center}

然后对小顶堆进行调整，保持小顶堆的性质，

![](http://image.augustrush8.com/images/heapsort3.png){:.center}

同理，44和75也进行相同的处理........所有数据处理完以后，就小顶堆内就是topN

这样，数据只需要读取一次，不会存在数据多次读写的问题，所以呢？写个代码实现吧，假设在1000个数中找出前50：

```java
public class HeapSort {
    // 父节点
    private int parent(int n) {
        return (n - 1) / 2;
    }

    // 左孩子
    private int left(int n) {
        return 2 * n + 1;
    }

    // 右孩子
    private int right(int n) {
        return 2 * n + 2;
    }

    // 构建堆
    private void buildHeap(int n, int[] data) {
        for(int i = 1; i < n; i++) {
            int t = i;
            // 调整堆
            while(t != 0 && data[parent(t)] > data[t]) {
                int temp = data[t];
                data[t] = data[parent(t)];
                data[parent(t)] = temp;
                t = parent(t);
            }
        }
    }

    // 调整data[i]
    private void adjust(int i, int n, int[] data) {
        if(data[i] <= data[0]) {
            return;
        }
        // 置换堆顶
        int temp = data[i];
        data[i] = data[0];
        data[0] = temp;
        // 调整堆顶
        int t = 0;
        while( (left(t) < n && data[t] > data[left(t)])
                || (right(t) < n && data[t] > data[right(t)]) ) {
            if(right(t) < n && data[right(t)] < data[left(t)]) {
                // 右孩子更小，置换右孩子
                temp = data[t];
                data[t] = data[right(t)];
                data[right(t)] = temp;
                t = right(t);
            } else {
                // 否则置换左孩子
                temp = data[t];
                data[t] = data[left(t)];
                data[left(t)] = temp;
                t = left(t);
            }
        }
    }

    // 寻找topN，该方法改变data，将topN排到最前面
    public void findTopN(int n, int[] data) {
        // 先构建n个数的小顶堆
        buildHeap(n, data);
        // n往后的数进行调整
        for(int i = n; i < data.length; i++) {
            adjust(i, n, data);
        }
    }

    // 打印数组
    public void print(int[] data) {
        for(int i = 0; i < data.length; i++) {
            System.out.print(data[i] + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        HeapSort heapSort=new HeapSort();
        int[] arr1 = new int[]{56, 30, 71, 18, 29, 93, 44, 75, 20, 65, 68, 34};
        System.out.println("原数组：");
        heapSort.print(arr1);
        heapSort.findTopN(5,arr1);
        System.out.println("调整后数组：");
        heapSort.print(arr1);
        // 第二组测试
        int[] arr2 = new int[1000];
        for(int i=0; i<arr2.length; i++) {
            arr2[i] = i + 1;
        }

        System.out.println("原数组：");
        heapSort.print(arr2);
        heapSort.findTopN(50,arr2);
        System.out.println("调整后数组：");
        heapSort.print(arr2);
    }

}
```

输出：

```xml
原数组：
56 30 71 18 29 93 44 75 20 65 68 34 
调整后数组：
65 68 71 75 93 18 29 30 20 44 56 34 
原数组：
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153 154 155 156 157 158 159 160 161 162 163 164 165 166 167 168 169 170 171 172 173 174 175 176 177 178 179 180 181 182 183 184 185 186 187 188 189 190 191 192 193 194 195 196 197 198 199 200 201 202 203 204 205 206 207 208 209 210 211 212 213 214 215 216 217 218 219 220 221 222 223 224 225 226 227 228 229 230 231 232 233 234 235 236 237 238 239 240 241 242 243 244 245 246 247 248 249 250 251 252 253 254 255 256 257 258 259 260 261 262 263 264 265 266 267 268 269 270 271 272 273 274 275 276 277 278 279 280 281 282 283 284 285 286 287 288 289 290 291 292 293 294 295 296 297 298 299 300 301 302 303 304 305 306 307 308 309 310 311 312 313 314 315  
......
调整后数组：
951 952 955 954 953 956 957 963 968 964 972 961 979 959 958 967 966 969 974 965 970 973 988 962 983 993 986 981 987 992 960 976 1000 982 978 977 975 985 984 990 971 997 996 991 989 999 998 980 994 995 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153 154 155 
......

```

