---
layout: post
title: '关于shell脚本应用的几个实例'
tags: [read]
---

### 统计词频

题目描述：

> 编写一个bash脚本来计算文本文件words.txt中每个单词的频率。为简单起见，您可以假设:
>
> - words.txt只包含小写字符和空格' '字符。
> - 每个单词只能由小写字符组成
> - 单词由一个或多个空格字符分隔。

举例：

假设words.txt具有以下内容：

```java
the day is sunny the the
the sunny is is
```

您的脚本应输出以下内容，按降序频率排序：

```java
the 4
is 3
sunny 2
day 1
```

**解决方案**：

```shell
awk '{for(i=1;i<=NF;i++) a[$i]++} END {for(k in a) print k,a[k]}' words.txt | sort -k2 -nr
```

`awk`指令在之前的博文当中介绍过：[传送门](https://augustrush.me/post/linux-command-awk.html)

这里解释一下`sort`指令当中的几个参数的含义：

- `-k2`对前面传输过来的两列数据中的第二列数据进行排序
- `-nr`按数值大小的逆序进行排序

详细的`sort`指令用法参考文章：https://www.jianshu.com/p/6a6c8389a08e

### 匹配电话号码

给定一个文本文件file.txt，其中包含电话号码列表（每行一个），写一个单行bash脚本来打印所有有效的电话号码。您可能认为有效电话号码必须采用以下两种格式之一：（xxx）xxx-xxxx或xxx-xxx-xxxx。（x表示数字），您还可以假设文本文件中的每一行都不得包含前导或尾随空格。

举例：

假设file.txt具有以下内容：

```xml
987-123-4567
123 456 7890
(123) 456-7890
```

您的脚本应输出以下有效电话号码：

```xml
987-123-4567
(123) 456-7890
```

**解决方案**：

这里需要用到正则相关的知识，具体的解决方案有以下几种：

```shell
grep -P '^(\d{3}-|\(\d{3}\) )\d{3}-\d{4}$' file.txt

sed -n -E '/^([0-9]{3}-|\([0-9]{3}\) )[0-9]{3}-[0-9]{4}$/p' file.txt

awk '/^([0-9]{3}-|\([0-9]{3}\) )[0-9]{3}-[0-9]{4}$/' file.txt
```

