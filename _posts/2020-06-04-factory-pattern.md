---
layout: post
title: '设计模式之工厂模式'
tags: [code]
---

## 前言
工厂模式的迭代过程主要分为简单工厂模式、工厂方法、抽象工厂这三个步骤，其复杂度从左至右逐步递增，但对调用者来说则是越来越简单。

## 简单工厂模式
**定义：** 简单工厂模式是指由一个工厂对象决定创建出哪一种产品类的实例

**适用性：** 适用于工厂类创建的对象较少的场景，且客户端只需要传入工厂类的参数，对于如何创建对象的逻辑不需要关心。

以课程为例，创建一个`ICourse`接口：
```java
public interface ICourse {
    //录制课程
    public void record();
}
```

创建一个`JavaCourse`实现`ICourse`接口：
```java
public class JavaCourse implements ICourse {
    @Override
    public void record() {
        System.out.println("录制Java视频课程");
    }
}
```

客户端调用代码：
```java
public static void main(String[] args) {
        ICourse course = new JavaCourse();
        course.record();
    }
```
在上面的代码中，父类ICourse指向子类JavaCourse的引用，应用层代码需要依赖JavaCourse，倘若业务扩展，增加了PythonCourse或其他课程，那么客户端的依赖就会变得臃肿。所以，我们需要减弱这种依赖，将创建细节隐藏起来。为了代码更好扩展，使用`简单工厂`对代码进行优化，首先增加PythonCourse类。
```java
public class PythonCourse implements ICourse {
    @Override
    public void record() {
        System.out.println("录制Python视频课程");
    }
}
```

创建CourseFactory工厂类：
```java
public class CourseFactory {
    public ICourse create(String name){
        if ("java".equals(name)) {
            return new JavaCourse();
        } else if ("python".equals(name)) {
            return new PythonCourse();
        } else {
            return null;
        }
    }
}
```

修改客户端调用代码：
```java
public class CourseRecord {
    public static void main(String[] args) {
        CourseFactory courseFactory = new CourseFactory();
        courseFactory.create("java");
    }
}
```

好了，现在客户端调用是简单了，但如果我们继续添加GoCourse，那么工厂当中的create()方法就要根据新增课程修改代码逻辑，这不符合`开闭原则`,所以我们可以采用反射技术继续优化。

CourseFactory.java
```java
public class CourseFactory {
    public static ICourse create(String className){
        try {
            if (className != null && !"".equals(className)) {
                return (ICourse) Class.forName(className).newInstance();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```
修改客户端调用代码：
```java
public class CourseRecord {
    public static void main(String[] args) {
        CourseFactory courseFactory = new CourseFactory();
        ICourse course = CourseFactory.create("com.admin.hellotest.JavaCourse");
        if (course != null) {
            course.record();
        }
    }
}
```
现在CourseFactory类就不需要随着课程新增或减少而变化了，但上面的客户端调用代码就不那么优雅了，方法参数的字符串又臭又长，而且CourseFactory的返回结果还要类型强转，于是我们再改：

CourseFactory.java
```java
public class CourseFactory {
    public static ICourse create(Class<? extends ICourse> clazz){
        if (null != clazz) {
            try {
                return clazz.newInstance();
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return null;
    }
}
```

客户端代码：
```java
public class CourseRecord {
    public static void main(String[] args) {
        CourseFactory courseFactory = new CourseFactory();
        ICourse course = CourseFactory.create(JavaCourse.class);
        if (course != null) {
            course.record();
        }
    }
}
```

简单工厂模式在JDK源码当中应用非常多，比如Calendar类，看下面这段源码：
```java
/**
     * Gets a calendar with the specified time zone and locale.
     * The <code>Calendar</code> returned is based on the current time
     * in the given time zone with the given locale.
     *
     * @param zone the time zone to use
     * @param aLocale the locale for the week data
     * @return a Calendar.
     */
    public static Calendar getInstance(TimeZone zone,
                                       Locale aLocale)
    {
        return createCalendar(zone, aLocale);
    }

    private static Calendar createCalendar(TimeZone zone,
                                           Locale aLocale)
    {
        CalendarProvider provider =
            LocaleProviderAdapter.getAdapter(CalendarProvider.class, aLocale)
                                 .getCalendarProvider();
        if (provider != null) {
            try {
                return provider.getInstance(zone, aLocale);
            } catch (IllegalArgumentException iae) {
                // fall back to the default instantiation
            }
        }

        Calendar cal = null;

        if (aLocale.hasExtensions()) {
            String caltype = aLocale.getUnicodeLocaleType("ca");
            if (caltype != null) {
                switch (caltype) {
                case "buddhist":
                cal = new BuddhistCalendar(zone, aLocale);
                    break;
                case "japanese":
                    cal = new JapaneseImperialCalendar(zone, aLocale);
                    break;
                case "gregory":
                    cal = new GregorianCalendar(zone, aLocale);
                    break;
                }
            }
        }
        if (cal == null) {
            // If no known calendar type is explicitly specified,
            // perform the traditional way to create a Calendar:
            // create a BuddhistCalendar for th_TH locale,
            // a JapaneseImperialCalendar for ja_JP_JP locale, or
            // a GregorianCalendar for any other locales.
            // NOTE: The language, country and variant strings are interned.
            if (aLocale.getLanguage() == "th" && aLocale.getCountry() == "TH") {
                cal = new BuddhistCalendar(zone, aLocale);
            } else if (aLocale.getVariant() == "JP" && aLocale.getLanguage() == "ja"
                       && aLocale.getCountry() == "JP") {
                cal = new JapaneseImperialCalendar(zone, aLocale);
            } else {
                cal = new GregorianCalendar(zone, aLocale);
            }
        }
        return cal;
    }
```

还有我们非常熟悉的`LoggerFactory`，看看它的`getlogger()`方法：
```java
public static Logger getLogger(String name) {
        ILoggerFactory iLoggerFactory = getILoggerFactory();
        return iLoggerFactory.getLogger(name);
    }
//--------------------------
public static Logger getLogger(Class<?> clazz) {
        Logger logger = getLogger(clazz.getName());
        //......
        return logger;
    }
```

简单工厂模式的缺点：**工厂类的职责过重，不易于扩展复杂的产品结构**。

## 工厂方法模式
**定义：** 是指定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类，`工厂方法让类的实例化推迟到子类中进行`。

**适用性：** 
- 创建对象需要大量重复的代码。
- 客户端（应用层）不依赖于产品类实例如何被创建、实现等细节。
- 一个类通过其子类来指定创建哪个对象。

工厂方法模式中用户只需要关心所需产品对应的工厂，无需关心创建细节，而且加入新的产品符合开闭原则。

工厂方法模式主要解决产品扩展的问题，在简单工厂当中，随着产品链的丰富，如果每个产品的创建逻辑有区别的话，`工厂`类的职责会变得越来越多，有点像万能工厂，不便于维护，根据`单一职责原则`我们需要将职能继续拆分，专人做专事。还是用上面的例子举例，那就是Java课程由Java工厂创建，Python课程由Python工厂创建，Go课程由Go工厂创建，对工厂本身也做一个抽象。show me your code。

ICourseFactory 接口：
```java
public interface ICourseFactory {
    ICourse create();
}
```

子工厂 JavaCourseFactory 类：
```java
public class JavaCourseFactory implements ICourseFactory {
    @Override
    public ICourse create() {
        return new JavaCourse();
    }
}
```

子工厂 PythonCourseFactory 类：
```java
public class PythonCourseFactory implements ICourseFactory {

    @Override
    public ICourse create() {
        return new PythonCourse();
    }
}
```

客户端调用代码：
```java
public class CourseRecord {
    public static void main(String[] args) {
        ICourseFactory courseFactory = new JavaCourseFactory();
        ICourse course = courseFactory.create();
        course.record();

        courseFactory = new PythonCourseFactory();
        course= courseFactory.create();
        course.record();
    }
}
```

logback中工厂方法模式用的也非常典型，可以看看ILoggerFactory的相关实现类。

工厂方法的缺点：
- 类的个数容易过多，增加复杂度
- 增加了系统的抽象性和理解难度

## 抽象工厂模式
**定义：** 是指提供一个创建一系列相关或相互依赖对象的接口，无须指定它们具体的类。

怎么理解抽象工厂模式呢？看下图：

![](http://image.augustrush8.com/images/2020-6-4%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82.png)

从生活中来举例，比如，美的电器生产多种家用电器。那么上图中，颜色最深的正方形就代表美的洗衣机、颜色最深的圆形代表美的空调、颜色最深的菱形代表美的热水器，颜色最深的一排都属于美的品牌，都是美的电器这个产品族。再看最右侧的菱形，颜色最深的我们指定了代表美的热水器，那么第二排颜色稍微浅一点的菱形，代表海信的热水器。同理，同一产品结构下还有格力热水器，格力空调，格力洗衣机 。

最左侧的小圆圈我们就认为是具体的工厂，有美的工厂，有海信工厂，有格力工厂。每个品牌的工厂都生产洗衣机、热水器和空调。 

接下来还是以课程为例，现在课程有了新的标准，每个课程不仅要提供课程的录播视频，而且还要提供老师的课堂笔记。相当于现在的业务变更为同一个课程不单纯是一个课程信息，要同时包含录播视频、课堂笔记甚至还要提供源码才能构成一个完整的课程。在产品等级中增加两个产品 IVideo 录播视频和 INote 课堂笔记。 

IVideo 接口： 

```java
public interface IVideo {
    void record();
}
```

I
Note 接口： 

```java
public interface INote {
    void edit();
}
```

创建一个抽象工厂 CourseFactory 类： 

```java
public interface CourseFactory {
    INote createNote();
    IVideo createVideo();
}		
```

接下来，创建 Java 产品族，Java 视频 JavaVideo 类: 

```java
public class JavaVideo implements IVideo {
    @Override
    public void record() {
        System.out.println("录制 Java 视频");
    }
}

```

扩展产品等级 Java 课堂笔记 JavaNote 类： 

```java
public class JavaNote implements INote {
    @Override
    public void edit() {
        System.out.println("编写 Java 笔记");
    }
}
```

创建 Java 产品族的具体工厂 JavaCourseFactory: 

```java
public class JavaCourseFactory implements CourseFactory {
    public INote createNote() {
        return new JavaNote();
    } 
    public IVideo createVideo() {
        return new JavaVideo();
    }
}
```

Python 工厂类类似，这里不赘述，最后看看客户端调用：

```java
public class CourseRecord {
    public static void main(String[] args) {
        JavaCourseFactory factory = new JavaCourseFactory();
        factory.createNote().edit();
        factory.createVideo().record();
    }
}
```

上面的代码描述了两个产品族， Java 课程和 Python 课程，也描述了两个产品等级视频和手记。抽象工厂非常完美清晰地描述这样一层复杂的关系。但是，不知道大家有没有发现，如果我们再继续扩展产品等级，将源码 Source 也加入到课程中，那么我们的代码从抽象工厂，到具体工厂要全部调整，很显然不符合开闭原则。因此抽象工厂也是有缺点的： 

- 规定了所有可能被创建的产品集合，产品族中扩展新的产品困难，需要修改抽象工厂的接口。 
- 增加了系统的抽象性和理解难度 。

但在实际应用中，我们千万不能犯强迫症甚至有洁癖。在实际需求中产品等级结构升级是非常正常的一件事情。我们可以根据实际情况，只要不是频繁升级，可以不遵循开闭原则。代码每半年升级一次或者每年升级一次又有何不可呢 ？