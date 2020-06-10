---
layout: post
title: 'Java 函数组合与集合管道模式'
tags: [code]
---

## 前言

本文主要通过介绍 Java8 当中的函数组合和集合管道模式以帮助更好的理解 Java8 特性，从而充分利用高阶函数和拉姆达表达式。对于高阶函数不太懂的同学可以回看《[Java 函数式编程演进之路](http://mp.weixin.qq.com/s?__biz=MzI4MzgwODQzOA==&mid=2247484445&idx=3&sn=2d82b88bb8604430ffc331062be94749&chksm=eb8443c4dcf3cad27c6f91a8c02779464b03c770a9e374e53eabfae96bd345ec884e69c6c124&scene=21#wechat_redirect)》这篇文章。

## 语句与表达式

在 Java 中，for 和 while 都是语句。语句执行一个操作，但不会生成任何结果。就本质而言，任何执行有用的操作的语句都会导致数据变化。这是语句表达其效果的唯一方式。而表达式则相反：它们可以得出结果而不会导致变化。

语句就像是一个个独立工作者一样，相互之间`无法转交工作结果`，而表达式的工作更像一条链：当某个人完成一项任务时，他将结果转交给链中的下一个人。

表达式帮助实现了集合管道模式，`Martin Fowler` 将该模式描述为`运算序列`，会将从一次运算收集的输出提供给下一次运算。

函数组合和集合管道模式是两种可协同工作的模式。下面将使用熟悉的 `for` 语句解决一个问题。然后介绍如何使用这函数组合和集合管道模式更高效地解决同一个问题。

## 使用 for 语句迭代和排序

首先创建一个 `Car` 类：

```java
publicclass Car {
    private String producer;
    private String model;
    privateint year;

    public Car(String producer, String model, int year) {
        this.producer = producer;
        this.model = model;
        this.year = year;
    }

    public String getProducer() {
        return producer;
    }

    public String getModel() {
        return model;
    }

    public int getYear() {
        return year;
    }
}
```

添加一个 Car 实例集合:

```java
public static List<Car> createCars() {
    return Arrays.asList(new Car("Jeep", "Wrangler", 2011),
            new Car("Jeep", "Comanche", 1990),
            new Car("Dodge", "Avenger", 2010),
            new Car("Buick", "Cascada", 2016),
            new Car("Ford", "Focus", 2012),
            new Car("Chevrolet", "Geo Metro", 1992));
}
```

接着使用`命令式编程`来迭代该列表，并获取 2000 年后制造的汽车的名称。然后按年份对这些型号进行升序排序。

```java
public static List<String> getNamesAfter2000UsingFor(List<Car> cars) {
    List<Car> carsSortedByYear = new ArrayList<>();
    for (Car car : cars) {
        if (car.getYear() > 2000) {
            carsSortedByYear.add(car);
        }
    }
    Collections.sort(carsSortedByYear, new Comparator<Car>() {
        @Override
        public int compare(Car o1, Car o2) {
            return Integer.compare(o1.getYear(), o2.getYear());
        }
    });
    List<String> names = new ArrayList<>();
    for (Car car : carsSortedByYear) {
        names.add(car.getName());
    }
    return names;
}
```

可以看出，上面的代码使用了很多的 for 语句，首先，`getNamesAfter2000UsingFor` 方法接受一个汽车列表作为参数。它提取或过滤出 2000 年后制造的汽车，将它们放在一个名为 `carsSortedByYear` 的新列表中。接下来，按照制造年份对该列表进行升序排序。最后，循环处理列表 `carsSortedByYear`，以获取型号名称，然后返回。

以上代码输出结果：

```java
Avenger
Wrangler
Focus
Cascada
```

以上代码中包含了不必要的垃圾变量，其处理示意图如下：

![](http://image.augustrush8.com/images/2020-06-10原始for例子.png){:.center}

## 使用集合管道迭代和排序

在函数编程中，通常会通过一系列更小的模块化函数或运算来对复杂运算进行操作，称为函数组合。当一个数据集合流经一个函数组合时，它就变成一个集合管道。函数组合和集合管道是函数式编程中常用的两种设计模式。

来看看使用集合管道处理上面例子的处理方式：

```java
public static List<String> getNamesAfter2000UsingPipeline(List<Car> cars){
    return cars.stream()
            .filter(car -> car.getYear()>2000)
            .sorted(Comparator.comparing(Car::getYear))
            .map(Car::getName)
            .collect(Collectors.toList());
}
```

其处理结果和之前的处理结果是一致的，但有以下几点不同：

- 函数式代码比命令式代码更简洁。
- 函数式代码不会表现出明显的易变性，而且使用了更少的垃圾变量。
- 集合管道处理方式使用的函数 / 方法都是有返回值的表达式。
- 使用了集合管道模式，而且非常富于表达。

以上集合管道代码简洁且富于表达力，部分原因是使用了`方法引用`。将一个 `lambda` 表达式传递给 `filter` 很有用，因为它可以获取给定对象的 `year` 属性并将其与 `year 2000` 进行比较。但是，将 `lambda` 表达式传递给 `map` 就没有这么有效。传递给 map 方法的表达式为 `car -> car.getModel()`，该表达式非常繁琐。该 `lambda` 表达式仅返回给定对象的某个属性，不执行任何实际计算或运算。我们最好将它替换为一个方法引用。

我们将方法引用 `Car::getModel` 传递给 map 方法，而不传递 `lambda` 表达式。类似地，我们将方法引用 `Car::getYear` 传递给 comparing 方法，而不传递拉姆达表达式 `car -> car.getYear()`。方法引用简短、简洁且富于表达力，最好尽可能地使用它们。

集合管道模式的处理示意图如下，随着数据流经各个函数，Java 8 的惰性计算和函数融合功能可帮助避免在某些情况下创建中间对象。数据在管道中传输时，函数不会使中间对象变得可见或可用。

![](http://image.augustrush8.com/images/2020-06-10集合管道例子.png){:.center}

## 小结

在命令式编程中，对于大部分数据处理，通常都会使用 for 和 while 循环。而结合使用函数组合和集合管道模式，就可以创建复杂的程序，让数据从上游流到下游，并经历一系列转换。

 