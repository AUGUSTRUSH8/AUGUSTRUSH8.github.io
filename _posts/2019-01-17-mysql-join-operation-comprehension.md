---
layout: post
title: '数据库表连接的理解'
tags: [read]
---

错综复杂的数据，需要建立模型，才能储存在数据库，模型是什么？

**模型**：

- **实体（entity）**实际的对象，带有自己的属性，可以理解成一组相关属性的容器
- **关系（relationship）**实体之间的联系，通常可以分成"一对一"、"一对多"和"多对多"等类型。

![](http://image.augustrush8.com/images/sqljoin1.png){:.center}

在关系型数据库里面，每个实体有自己的一张表（table），所有属性都是这张表的字段（field），表与表之间根据关联字段"连接"（join）在一起。所以，表的连接是关系型数据库的核心问题。

**连接类型**：

- 内连接（inner join）
- 外连接（outer join）
- 左连接（left join）
- 右连接（right join）
- 全连接（full join）

**韦恩图表示**：

内连接：

![](http://image.augustrush8.com/images/sqljoin2.png)

左连接：

![](http://image.augustrush8.com/images/sqljoin3.png)

右连接：

![](http://image.augustrush8.com/images/sqljoin4.png)

全连接：

![](http://image.augustrush8.com/images/sqljoin5.png)

**所谓"连接"，就是两张表根据关联字段，组合成一个数据集。**

问题是，两张表的关联字段的值往往是不一致的，如果关联字段不匹配，怎么处理？比如，表 A 包含张三和李四，表 B 包含李四和王五，匹配的只有李四这一条记录。

一共有四种处理方法:

- 只返回两张表匹配的记录，这叫内连接（inner join）。
- 返回匹配的记录，以及表 A 多余的记录，这叫左连接（left join）。
- 返回匹配的记录，以及表 B 多余的记录，这叫右连接（right join）。
- 返回匹配的记录，以及表 A 和表 B 各自的多余记录，这叫全连接（full join）。

四种连接的图示：

![](http://image.augustrush8.com/images/sqljoin6.png)

解释：上图中，表 A 的记录是 123，表 B 的记录是 ABC，颜色表示匹配关系。返回结果中，如果另一张表没有匹配的记录，则用 null 填充。

四种连接可以分为两大类：

- 内连接（inner join）表示只包含匹配的记录
- 外连接（outer join）表示还包含不匹配的记录

所以，左连接、右连接、全连接都属于外连接。

这四种连接的 SQL 语句如下：

```sql
SELECT * FROM A  
INNER JOIN B ON A.book_id=B.book_id;

SELECT * FROM A  
LEFT JOIN B ON A.book_id=B.book_id;

SELECT * FROM A  
RIGHT JOIN B ON A.book_id=B.book_id;

SELECT * FROM A  
FULL JOIN B ON A.book_id=B.book_id;
```

上面的 SQL 语句还可以加上`where`条件从句，对记录进行筛选，比如只返回表 A 里面不匹配表 B 的记录。

```sql
SELECT * FROM A
LEFT JOIN B
ON A.book_id=B.book_id
WHERE B.id IS null;
```

又比如，返回表 A 或表 B 所有不匹配的记录：

```sql
SELECT * FROM A
FULL JOIN B
ON A.book_id=B.book_id
WHERE A.id IS null OR B.id IS null;
```

此外，还存在一种特殊的连接，叫做"交叉连接"（cross join），指的是表 A 和表 B 不存在关联字段，这时表 A（共有 n 条记录）与表 B （共有 m 条记录）连接后，会产生一张包含 n x m 条记录的新表（见下图）

![](http://image.augustrush8.com/images/sqljoin7.png)

