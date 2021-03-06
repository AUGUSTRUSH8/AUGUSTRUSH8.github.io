---
layout: post
title: '海量数据处理'
tags: [read]
---

### 计算容量

在解决问题之前，要先计算一下海量数据需要占多大的容量。常见的单位换算如下：

- 1 byte = 8 bit
- 1 KB = 210 byte = 1024 byte ≈ 103 byte
- 1 MB = 220 byte ≈ 10 6 byte
- 1 GB = 230 byte ≈ 10 9 byte
- 1 亿 = 108

1 个整数占 4 byte，1 亿个整数占 4*108 byte ≈ 400 MB。

### 拆分

可以将海量数据拆分到多台机器上和拆分到多个文件上：

- 如果数据量很大，无法放在一台机器上，就将数据拆分到多台机器上。这种方式可以让多台机器一起合作，从而使得问题的求解更加快速。但是也会导致系统更加复杂，而且需要考虑系统故障等问题；
- 如果在程序运行时无法直接加载一个大文件到内存中，就将大文件拆分成小文件，分别对每个小文件进行求解。

有以下策略进行拆分：

- 按出现的顺序拆分：当有新数据到达时，先放进当前机器，填满之后再将数据放到新增的机器上。这种方法的优点是充分利用系统的资源，因为每台机器都会尽可能被填满。缺点是需要一个查找表来保存数据到机器的映射，查找表可能会非常复杂并且非常大。

![](http://image.augustrush8.com/images/bigdata1.png){:.center}

- 按散列值拆分：选取数据的主键 key，然后通过哈希取模 hash(key)%N 得到该数据应该拆分到的机器编号，其中 N 是机器的数量。优点是不需要使用查找表，缺点是可能会导致一台机器存储的数据过多，甚至超出它的最大容量。

![](http://image.augustrush8.com/images/bigdata2.png){:.center}

- 按数据的实际含义拆分：例如一个社交网站系统，来自同一个地区的用户更有可能成为朋友，如果让同一个地区的用户尽可能存储在同一个机器上，那么在查找一个用户的好友信息时，就可以避免到多台机器上查找，从而降低延迟。缺点同样是需要使用查找表。

![](http://image.augustrush8.com/images/bigdata3.png){:.center}

### 整合

拆分之后的结果还只是局部结果，需要将局部结果汇总为整体的结果。

### 海量数据判重

> 描述：对于海量数据，要求判断一个数据是否已经存在。这个数据很有可能是字符串，例如 URL。

#### 解决方案一之HashSet

最直观的方法是使用 HashSet 存储，那么就能以 O(1) 的时间复杂度判断一个数据是否已经存在。

考虑到数据是海量的，那么就需要使用拆分的方式将数据拆分到多台机器上，分别在每台机器上使用 HashSet 存储。我们需要使得相同的数据拆分到相同的机器上，可以使用哈希取模的拆分方式进行实现。

#### 解决方案二之BitSet

如果海量数据是整数，并且范围不大时，就可以使用 BitSet 存储。通过构建一定大小的比特数组，并且让每个整数都映射到这个比特数组上，就可以很容易地知道某个整数是否已经存在。因为比特数组比整型数组小的多，所以通常情况下单机就能处理海量数据。

![](http://image.augustrush8.com/images/bigdata4.png){:.center}

java.util.BitSet可以按位存储。
计算机中一个字节（byte）占8位（bit），我们java中数据至少按字节存储的，
比如一个int占4个字节。
如果遇到大的数据量，这样必然会需要很大存储空间和内存。
如何减少数据占用存储空间和内存可以用算法解决。
java.util.BitSet就提供了这样的算法。
比如有一堆数字，需要存储，source=[3,5,6,9]
用int就需要4*4个字节。
java.util.BitSet可以存true/false。
如果用java.util.BitSet，则会少很多，其原理是：
1，先找出数据中最大值maxvalue=9
2，声明一个BitSet bs,它的size是maxvalue+1=10
3，遍历数据source，bs[source[i]]设置成true.
最后的值是：
(0为false;1为true)
bs [0,0,0,1,0,1,1,0,0,1]
                3,   5,6,       9

这样一个本来要int型需要占4字节共32位的数字现在只用了1位！
比例32:1  

这样就省下了很大空间。

#### 解决方案三之布隆过滤器

布隆过滤器能够以极小的空间开销解决海量数据判重问题，但是会有一定的误判概率。它主要用在网页黑名单系统、垃圾邮件过滤系统、爬虫的网址判重系统。

布隆过滤器也是使用 BitSet 存储数据，但是它进行了一定的改进，从而解除了 BitSet 要求数据的范围不大的限制。在存储时，它要求数据先经过 k 个哈希函得到 k 个位置，并将 BitSet 中对应位置设置为 1。在查找时，也需要先经过 k 个哈希函数得到 k 个位置，如果所有位置上都为 1，那么表示这个数据存在。

由于哈希函数的特点，两个不同的数通过哈希函数得到的值可能相同。如果两个数通过 k 个哈希函数得到的值都相同，那么使用布隆过滤器会将这两个数判为相同。

可以知道，令 k 和 m 都大一些会使得误判率降低，但是这会带来更高的时间和空间开销。

布隆过滤器会误判，也就是将一个不存在的数判断为已经存在，这会造成一定的问题。例如在垃圾邮件过滤系统中，会将一个邮件误判为垃圾邮件，那么就收不到这个邮件。可以使用白名单的方式进行补救。

![](http://image.augustrush8.com/images/bigdata5.png){:.center}

#### 解决方案四之Trie树

Trie 树又叫又叫字典树、前缀树、单词查找树，它是一颗多叉查找树。与二叉查找树不同，键不是直接保存在节点中，而是由节点在树中的位置决定。

如果海量数据是字符串数据，那么就可以用很小的空间开销构建一颗 Trie 树，空间开销和树高有关。

![](http://image.augustrush8.com/images/bigdata6.png){:.center}

Trie树简单代码实现

```java
class Trie {
    private class Node {
        Node[] childs = new Node[26];
        boolean isLeaf;
    }

    private Node root = new Node();

    public Trie() {
    }

    public void insert(String word) {
        insert(word, root);
    }

    private void insert(String word, Node node) {
        if (node == null) return;
        if (word.length() == 0) {
            node.isLeaf = true;
            return;
        }
        int index = indexForChar(word.charAt(0));
        if (node.childs[index] == null) {
            node.childs[index] = new Node();
        }
        insert(word.substring(1), node.childs[index]);
    }

    public boolean search(String word) {
        return search(word, root);
    }

    private boolean search(String word, Node node) {
        if (node == null) return false;
        if (word.length() == 0) return node.isLeaf;
        int index = indexForChar(word.charAt(0));
        return search(word.substring(1), node.childs[index]);
    }

    public boolean startsWith(String prefix) {
        return startWith(prefix, root);
    }

    private boolean startWith(String prefix, Node node) {
        if (node == null) return false;
        if (prefix.length() == 0) return true;
        int index = indexForChar(prefix.charAt(0));
        return startWith(prefix.substring(1), node.childs[index]);
    }

    private int indexForChar(char c) {
        return c - 'a';
    }
}
```

### 海量数据排序

#### 外部排序

海量数据不能一次性读入内存，在对海量数据进行排序时，首先需要将海量数据拆分到多台机器或者多个文件，这些机器或文件称为拆分节点；然后在每个拆分节点上将数据全部读入内存并使用快速排序等方法进行排序；最后在合并节点使用多路归并方法将所有拆分节点的部分排序结果整合成最终的排序结果。外部排序也可以被称为外部归并排序。

![](http://image.augustrush8.com/images/bigdata7.png){:.center}

如果不进行额外处理，合并节点仍然无法将所有数据读入内存中。可以使用小顶堆来解决这个问题：

1. 假设有 k 个拆分节点，从这 k 个拆分节点分别读取一个最小的数据到小顶堆中。
2. 将堆顶数据移出堆并写入合并节点的最终结果文件中。
3. 确定刚才从堆中移除的数据属于哪个拆分节点，并从该拆分节点再读入一个数据。

例如以下有三个拆分节点，分别从这三个拆分节点读入 1,2,3 数据之后构建小顶堆，将堆顶数据 1 移出堆，因为数据 1 属于第一个拆分元素，因此再从第一个拆分节点读取一个数据到堆中。

![](http://image.augustrush8.com/images/bigdata8.png){:.center}

但是上面的做法需要频繁地读写磁盘，可以设置输入缓存和输出缓存来解决这个问题。为每个拆分节点都设置一个输入缓存，每次将一部分数据读入输入缓存中，只有当输入缓存数据为空时才再从磁盘读入数据。并设置一个输出缓存，只有输出缓存满时才将数据写出磁盘中。

#### BitMap

如果待排序的数据是整数，或者其它范围比较小的数据时，可以使用 BitMap 对其进行排序。BitMap 相当于一个比特数组，如果某个数据存在时就将对应的比特数组位置设置为 1，最后从头遍历比特数组就能得到一个排序的整数序列。

例如要对 {2,1,5} 进行排序，可以设置一个范围为 0~7 的比特数组，读入数据之后将比特数组第 1、2、5 位置设置为 1。最后从头遍历比特数组，将比特数组值为 1 的数据读出得到 {1,2,5} 这个已排序的数据。

![](http://image.augustrush8.com/images/bigdata9.png){:.center}

这种方法只能处理数据不重复的情况，如果数据重复，就要将比特数组转换成整数数组用于计数，这种排序方法叫做计数排序。可以把整数数组看成 32 个比特数组，32 比特可以存放的计数最大值为 232 ，在某些场景下数据的重复量不会这么大，只需要几个比特数组就能完成计数操作。

32 位整数的需要的比特数组空间为 232 bit≈ 4.2▪109 bit≈5▪108 byte≈500Mb，一台机器就可以放得下。因此对无重复的海量整数数据进行排序时，只需要设置一个比特数组就能完成该排序操作。

### Trie

如果海量数据是字符串的话，可以使用 Trie 树来完成排序操作。先读入海量字符串数据构建一个 Trie 树，最后按字典序先序遍历 Trie 树就能得到已排序的数据。为了处理数据重复问题，可以使用 Trie 树的节点存储计数信息。

