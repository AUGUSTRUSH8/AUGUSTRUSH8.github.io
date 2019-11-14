---
layout: post
title: '优化到极致的代码'
tags: [code]
---

读下去，你会有所收获的，相信我！

### 需求描述

题目：我现在需要实现一个栈，这个栈除了可以进行普通的push、pop操作以外，还可以进行getMin的操作，getMin方法被调用后，会返回当前栈的最小值，你会怎么做呢？可以假设栈里面存的都是int整数。

### 解决方案

- 第一反应

可以用一个变量来保存最小值，在push的时候更新这个最小值。

![](http://image.augustrush8.com/images/stack1.png){:.center}

问题：最小值被pop出去了怎么办？

![](http://image.augustrush8.com/images/stack2.png){:.center}

那就只能再遍历一遍整个栈，再更新最小值了。

![](http://image.augustrush8.com/images/stack3.png){:.center}

时间复杂度、空间复杂度：pop操作时间O(n)，其他操作时间都是O(1)，空间O(1)。

想要时间更短，怎么办？

- 更优的解法

那就空间换时间了，创建一个辅助栈，辅助栈中保存最小值。

![](http://image.augustrush8.com/images/stack4.png){:.center}

辅助栈每次push当前的最小值，pop时，两栈同时pop。

时间、空间复杂度：时间O(1)，空间O(n)。

代码实现（注意这里是用ArrayList模拟栈操作）：

```java
import java.util.ArrayList;
import java.util.List;

public class MinStack {
    private static List<Integer> data=new ArrayList<>();
    private static List<Integer> mins=new ArrayList<>();
    private void push(int num){
        data.add(num);
        if(mins.size()==0){
            mins.add(num);
        }else {
            // 辅助栈mins每次push当时最小值
            int min=getMin();
            if(min>=num){
                mins.add(num);
            }else {
                mins.add(min);
            }
        }

    }
    private int pop(){
        // 栈空，异常，返回-1
        if(data.size()==0){
            return -1;
        }
        int num=data.get(data.size()-1);
        //pop时两栈同步pop
        data.remove(data.size()-1);
        mins.remove(mins.size()-1);
        return num;
    }
    private int getMin(){
        int min=0;
        // 栈空，异常，返回-1
        if(mins.size()==0){
            return -1;
        }else{
            min=mins.get(mins.size()-1);
        }
        return min;
    }
}

```

**思考**：代码有什么问题？

当栈内为空的时候，返回-1，但是如果用户push过-1，那么返回-1的时候，是用户push进来的值，还是栈为空，就不得而知了。

那定义成什么好呢？**Integer.MaxValue**? 那还是会存在同样的问题。

**再定义一个类**，里面包括一个int的data和一个boolean的isSuccess，正常情况下isSuccess是true，栈为空的话，isSuccess是false。这个方案呢？

如果这样的话，用户需要再用一个类去接收返回值，然后再从里面取出真正的数，把问题复杂化了。

使用**Integer**来定义返回值呢？如果是空，就代表栈为空就行了。它和int的区别就是它多了一个null，正好用来返回异常情况。

还是不对，没有站在使用者的角度考虑这个问题，使用你这个栈的人，在pop的时候，他并不知道可能返回null，如果他不做判断，后面的代码就可能抛出空指针了。

那么，**最好的处理方式到底是啥**？

使用Java推荐的方式，**显式的抛出异常**，如果使用者不捕获的话，在编译期就会报错，把错误暴露在编译期。并且**不需要和任何人商量所谓的特殊返回值了**。



- 还有更优的解法，空间可以再小一点点

想象一下，如果依次入栈2 1 2 3 4 5 6，辅助栈当中会存什么？

![](http://image.augustrush8.com/images/stack5.png){:.center}

可以看到，辅助栈后面全是1，这一块可以再优化一下

我们可以在push的时候判断一下，如果比最小值还大，就不加入辅助栈。pop的时候，如果不是最小值，辅助栈就不出栈。这样一来，辅助栈就不会有大量重复元素了。

![](http://image.augustrush8.com/images/stack6.png){:.center}

push的时候进行判断，如果数值比当前最小值大，就不动mins栈了，这样mins栈中不会保存大量冗余的最小值。pop的时候同样进行判断，只有pop出的数就是当前最小值的时候，才让mins出栈。

**思考**：如果再来一个和最小值相等的数要入栈，要不要添加到辅助栈。

![](http://image.augustrush8.com/images/stack7.png){:.center}

如果push一个和最小值相等的元素，还是要入mins栈。不然当这个最小值pop出去的时候。data中还会有一个最小值元素，而mins中却已经没有最小值元素了。

**思考**：其实还可以继续优化。如果入栈顺序是2 1 1 1 1 1 1 1，辅助栈当中还是一堆1。

怎么解决？如果辅助栈中存的不是最小值，而是最小值的索引就好了。

![](http://image.augustrush8.com/images/stack8.png){:.center}

mins栈中改存最小值在data数组中的索引。这样一来，当push了与最小值相同元素的时候，就不需要动mins栈了。而pop的时候，pop出的元素的索引如果不是mins栈顶元素，mins也不出栈。同时，获取最小值的时候，需要拿到mins栈顶元素作为索引，再去data数组中找到相应的数作为最小值。

```java
import java.util.ArrayList;
import java.util.List;

public class MinStack {
    private static List<Integer> data=new ArrayList<>();
    private static List<Integer> mins=new ArrayList<>();
    private void push(int num){
        data.add(num);
        if(mins.size()==0){
            mins.add(0);
        }else {
            // 辅助栈mins每次push当时最小值
            int min=getMin();
            if(min>=num){
                mins.add(data.size()-1);
            }
        }

    }
    private int pop(){
        // 栈空，异常，返回-1
        if(data.size()==0){
            throw new RuntimeException("stack is empty!");
        }
        int num=data.get(data.size()-1);
        //pop时两栈同步pop
        data.remove(data.size()-1);
        if(data.size()-1 == mins.get(mins.size()-1)){
            mins.remove(mins.size()-1);
        }
        return num;
    }
    private int getMin(){
        int min=0;
        int minIdex=0;
        // 栈空，异常，返回-1
        if(mins.size()==0){
            throw new RuntimeException("stack is empty!");
        }else{
            minIdex=mins.get(mins.size()-1);
        }
        return data.get(minIdex);
    }
}
```

以上，就是优化后的代码。

### 总结

数据结构和算法的设计是一个程序员的内功，工作时虽然用不到这么细，但是在学习其他知识的底层原理的时候，到处都是数据结构和算法。