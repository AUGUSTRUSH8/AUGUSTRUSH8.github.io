---
layout: post
title: '《漫画算法：小灰的算法之旅》读书总结'
tags: [read]
---

《漫画算法：小灰的算法之旅》这本书很好，作者将很多苦涩难懂的算法知识以对话漫画的形式一步一步剖析并给出对应示例代码，比起某一些教条式的阐述分析解更能让人接受吸收，具体的可以去看原书。下面我将针对其中的某些点进行记录总结，由于这方面我已经有了一些自己的知识沉淀，所以并不是从头到尾按顺序阅读的，所以主要按照我自己的跳跃式阅读节奏看的。

### 如何实现抢红包算法

这个问题是我个人在面试阿里三面的时候面试官给的一道题，所以比较感兴趣这其中的一些思考和解法，先看看这道题。

> **需求阐述**：“双十一” 快要到了， 我们需要上线一个发放红包的功能。 这个功能类似于微信群发红包的功能。 例如一个人在群里发了100块钱的红包， 群里有10个人一起来抢红包， **每人抢到的金额随机分配**。
>
> **要求**： 
>
> 1. 所有人抢到的金额之和要等于红包金额，不能多也不能少。
> 2. 每个人至少抢到1分钱。
> 3. 要保证红包拆分的金额尽可能分布均衡，不要出现两极分化太严重的情况。 

**base解法**：每次拆分的金额 = 随机区间[1分, 剩余金额-1分] 

其实以上并不能称之为base解法，因为base解法是要能够完成对应需求要求的解法，但实际上述解法并没有，但这个解法非常直观，由它去衍生别的解法比较好。

那么以上解法有什么问题？它随机的结果不均衡（结合每次随机的区间均值去分析就好了）

**better解法**：二倍均值法

假设剩余红包金额为m元，剩余人数为n，那么有如下公式。
每次抢到的金额 = 随机区间 [0.01， m /n × 2 - 0.01]元 

以上算法就解决了不公平的问题，同样使用以上的区间均值法去分析就知道为什么公平了

code:

```java
/**
* 拆分红包
* @param totalAmount 总金额（以分为单位）
* @param totalPeopleNum 总人数
*/
public static List<Integer> divideRedPackage(Integer
	totalAmount, Integer totalPeopleNum){
	List<Integer> amountList = new ArrayList<Integer>();
	Integer restAmount = totalAmount;
	Integer restPeopleNum = totalPeopleNum;
	Random random = new Random();
	for(int i=0; i<totalPeopleNum-1; i++){
 //随机范围：[1，剩余人均金额的2倍-1] 分
		int amount = random.nextInt(restAmount /
			restPeopleNum * 2 - 1) + 1;
		restAmount -= amount;
		restPeopleNum --;
		amountList.add(amount);
	}
	amountList.add(restAmount);
	return amountList;
}

public static void main(String[] args){
	List<Integer> amountList = divideRedPackage(1000, 10);
	for(Integer amount : amountList){
		System.out.println(" 抢到金额：" + new BigDecimal(amount).
			divide(new BigDecimal(100)));
	}
}
```

这个算法有什么问题？它每次抢到的金额都要小于剩余人均金额的2倍 ，并不是完全自由的随机抢红包。

**better better解法**：线段树解法

这个解法就能既做到**公平**，又**不超过总金额**，还能**提高随机抢红包的自由度**呢

其思想就主要是将整个待分金额想象成为一根线段，要把它分给n个人，只需要在这个线段的中间随机砍n-1次即可，随机的范围是[1,m-1] （m为红包金额），需要注意的就是随机点重复的问题。

demo code:

```java
List<Integer> generatePacketsByLineCutting(int people, int money) {
    List<Integer> packets = new ArrayList<>();
    Random random = new Random();
    Set<Integer> points = new TreeSet<>();
    while (points.size() < people - 1) {
        points.add(random.nextInt(money - 1));
    }
    points.add(money);
    int pre = 0;
    for (int p : points) {
        packets.add(p - pre);
        pre = p;
    }
    return packets;
}
```

### 二叉树的深度遍历和广度遍历

深度遍历很明显就是一种类似于栈操作的过程，其主要有非递归和递归两种解法，递归的解法非常直观易懂，如下简单的前序遍历递归表示：

```java
/**
 * 二叉树前序遍历
 * @param node 二叉树节点
 */
   public static void preOrderTraveral(TreeNode node){
       if(node == null){
           return;
       }
       System.out.println(node.data);
       preOrderTraveral(node.leftChild);
       preOrderTraveral(node.rightChild);
   }
```

那么怎么用非递归解答这样的问题呢，如下，这个需要多理解练习，看一遍不够。

```java
public static void preOrderTraveralWithStack(Node root){
        if (root==null){
            return;
        }
        Stack<Node> nodeStack = new Stack<>();
        while (root!=null || !nodeStack.isEmpty()){
            while (root!=null){
                System.out.println(root.getVal());
                nodeStack.push(root);
                root = root.getLeft();
            }
            if (!nodeStack.isEmpty()){
                Node curr = nodeStack.pop();
                root = curr.getRight();
            }
        }
    }
```

二叉树的层次遍历就没那么烧脑了，非常纯粹的队列操作，demo代码如下：

```java
public static void levelOrderTraversal(Node root){
        Queue<Node> queue =new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()){
            Node curr = queue.poll();
            System.out.println(curr.getVal());
            if (curr.getLeft()!=null){
                queue.offer(curr.getLeft());
            }
            if (curr.getRight()!=null){
                queue.offer(curr.getRight());
            }
        }
    }
```

### 看看二叉堆的存储数据结构和衍生品

二叉堆，本质上是一颗完全二叉树，分为两个类型：

- 最大堆（任何一个父节点的值， 都大于或等于它左、 右孩子节点的值 ）
- 最小堆（任何一个父节点的值， 都小于或等于它左、 右孩子节点的值 ）

二叉堆的自我调整：

- 插入节点
- 删除节点
- 构建二叉堆

主要包括两个动作：**上浮+下沉**

二叉堆虽然是一个完全二叉树， 但它的存储方式并不是链式存储， 而是顺序存储。 换句话说， **二叉堆的所有节点都存储在数组中** 

衍生品？  **优先队列**

其主要代码示意如下：

```java
public class PriorityQueue {
    private int[] array;
    private int size;
    public PriorityQueue(){
        array=new int[32];
    }
    public void enqueue(int key){
        if (size>=array.length){
            resize();
        }
        array[size++]=key;
        upAdjust();
    }
    //上浮操作
    public void upAdjust(){
        int childIndex = size-1;
        int parentIndex= (childIndex-1)/2;
        while (childIndex>0){
            int currVal=array[childIndex];
            int parentVal= array[parentIndex];
            if (currVal<parentVal){
                break;
            }
            int temp = array[childIndex];
            array[childIndex]= array[parentIndex];
            array[parentIndex]=temp;
            childIndex=parentIndex;
            parentIndex=childIndex/2;

        }

    }
    //下沉操作
    public void downAdjust(){
        int parentIndex = 0;
        int childIndex =1;
        int temp = array[parentIndex];
        while (childIndex<size){
            if (childIndex+1<size && array[childIndex]<array[childIndex+1]){
                childIndex++;
            }
            if (temp>array[childIndex]){
                break;
            }
            array[parentIndex]=array[childIndex];
            parentIndex= childIndex;
            childIndex=2*childIndex+1;
        }
        array[parentIndex]=temp;
    }
    public void resize(){
        int newSize= size*2;
        array= Arrays.copyOf(array,newSize);
    }
}
```

### 几个典型的排序算法

事实上，冒泡排序是有很多的优化步骤的，下面是一段不错的优化后冒泡排序算法：

```java
public static void bubbleSort(int[] arr){
    	//记录上次发生交换的位置
        int lastExchangeIndex= 0;
    	//记录基本有序位置，也就是后面在进行比较的时候可以停止比较的位置
        int sortBoarder=arr.length-1;
        for (int i = 0; i < arr.length - 1; i++) {、
            //每次大循环的时候，先把已经有序标志置为true，这样，只有没有发生交换，将直接跳出循环
            boolean sorted=true;
            for (int j = 0; j < sortBoarder; j++) {
                int temp=0;	
                if (arr[j]>arr[j+1]){
                    temp=arr[j];
                    arr[j]=arr[j+1];
                    arr[j+1]=temp;
                    sorted=false;
                    lastExchangeIndex=j;
                }
            }
            sortBoarder = lastExchangeIndex;
            if (sorted){
                break;
            }
        }
    }
```

