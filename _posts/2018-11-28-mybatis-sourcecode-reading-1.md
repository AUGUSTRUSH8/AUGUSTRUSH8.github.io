---
layout: post
title: 'Mybatis源码阅读第一话'
tags: [read]
---

![](http://image.augustrush8.com/images/mybatis/introduction.jpg){:.center}

> MyBatis SQL映射器框架使关系数据库与面向对象的应用程序的使用变得更加容易。MyBatis使用XML描述符或注释将对象与存储过程或SQL语句结合在一起。简单性是MyBatis数据映射器相对于对象关系映射工具的最大优势。

#### 第一步

从https://github.com/mybatis/mybatis-3 fork源码到自己的仓库当中，方便自己注释提交

#### 第二步

把项目导入到`idea`当中

#### 第三步

> MyBatis 想要调试，非常方便，只需要打开 `org.apache.ibatis.autoconstructor.AutoConstructorTest` 单元测试类，任意一个单元测试方法，右键，开始调试即可。

为更好的理解`AutoConstructorTest`这个类，我们先来看看`autoconstructor`这个包

![](http://image.augustrush8.com/images/mybatis/package.JPG){:.center}

##### mybatis-config.xml

```xml
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

    <!-- autoMappingBehavior should be set in each test case -->

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC">
                <property name="" value=""/>
            </transactionManager>
            <dataSource type="UNPOOLED">
                <property name="driver" value="org.hsqldb.jdbcDriver"/>
                <property name="url" value="jdbc:hsqldb:mem:automapping"/>
                <property name="username" value="sa"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="org/apache/ibatis/autoconstructor/AutoConstructorMapper.xml"/>
    </mappers>

</configuration>
```

**解释**：

- 在 `<environments />` 标签中，配置了事务管理和数据源。考虑到减少外部依赖，所以使用了 [HSQLDB](https://zh.wikipedia.org/wiki/HSQLDB) 。
- 在 `<mappers />` 标签中，配置了需要扫描的 Mapper 文件。目前，仅仅扫描 `AutoConstructorMapper.xml` 文件。

##### AutoConstructorMapper.xml

```xml
<!DOCTYPE mapper
    PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.apache.ibatis.autoconstructor.AutoConstructorMapper">
</mapper>
```

**解释**：

- 对应的接口为 `org.apache.ibatis.autoconstructor.AutoConstructorMapper` 。

##### AutoConstructorMapper

```java
public interface AutoConstructorMapper {
  @Select("SELECT * FROM subject WHERE id = #{id}")
  PrimitiveSubject getSubject(final int id);

  @Select("SELECT * FROM subject")
  List<PrimitiveSubject> getSubjects();

  @Select("SELECT * FROM subject")
  List<AnnotatedSubject> getAnnotatedSubjects();

  @Select("SELECT * FROM subject")
  List<BadSubject> getBadSubjects();

  @Select("SELECT * FROM extensive_subject")
  List<ExtensiveSubject> getExtensiveSubject();
}
```

**说明**：

- 使用注解的方法，编写 SQL 。

##### CreateDB.sql

```sql
DROP TABLE subject
IF EXISTS;

DROP TABLE extensive_subject
IF EXISTS;

CREATE TABLE subject (
  id     INT NOT NULL,
  name   VARCHAR(20),
  age    INT NOT NULL,
  height INT,
  weight INT,
  active BIT,
  dt     TIMESTAMP
);

CREATE TABLE extensive_subject (
  aByte      TINYINT,
  aShort     SMALLINT,
  aChar      CHAR,
  anInt      INT,
  aLong      BIGINT,
  aFloat     FLOAT,
  aDouble    DOUBLE,
  aBoolean   BIT,
  aString    VARCHAR(255),
  anEnum     VARCHAR(50),
  aClob      LONGVARCHAR,
  aBlob      LONGVARBINARY,
  aTimestamp TIMESTAMP
);

INSERT INTO subject VALUES
  (1, 'a', 10, 100, 45, 1, CURRENT_TIMESTAMP),
  (2, 'b', 10, NULL, 45, 1, CURRENT_TIMESTAMP),
  (2, 'c', 10, NULL, NULL, 0, CURRENT_TIMESTAMP);

INSERT INTO extensive_subject
VALUES
  (1, 1, 'a', 1, 1, 1, 1.0, 1, 'a', 'AVALUE', 'ACLOB', 'aaaaaabbbbbb', CURRENT_TIMESTAMP),
  (2, 2, 'b', 2, 2, 2, 2.0, 2, 'b', 'BVALUE', 'BCLOB', '010101010101', CURRENT_TIMESTAMP),
  (3, 3, 'c', 3, 3, 3, 3.0, 3, 'c', 'CVALUE', 'CCLOB', '777d010078da', CURRENT_TIMESTAMP);
```

**说明**：

`CreateDB.sql` 文件，用于单元测试里，初始化数据库的数据

- 创建了 `subject` 表，并初始化三条数据。
- 创建了 `extensive_subject` 表，并初始化三条数据。

##### 四个POJO

**思考：**在 AutoConstructorMapper 中，我们可以看到有四个 POJO 类。但是，从 `CreateDB.sql` 中，实际只有两个表。这个是为什么呢？

1.AnnotatedSubject

```java
public class AnnotatedSubject {
  private final int id;
  private final String name;
  private final int age;
  private final int height;
  private final int weight;

  public AnnotatedSubject(final int id, final String name, final int age, final int height, final int weight) {
    this.id = id;
    this.name = name;
    this.age = age;
    this.height = height;
    this.weight = weight;
  }

  @AutomapConstructor
  public AnnotatedSubject(final int id, final String name, final int age, final Integer height, final Integer weight) {
    this.id = id;
    this.name = name;
    this.age = age;
    this.height = height == null ? 0 : height;
    this.weight = weight == null ? 0 : weight;
  }
}
```

**说明**：

- 对应 `subject` 表。
- `@AutomapConstructor` 注解，表示 MyBatis 查询后，在创建 AnnotatedSubject 对象，使用该构造方法。（实际场景下，不常使用这个注解）

2.PrimitiveSubject

```java
public class PrimitiveSubject {
  private final int id;
  private final String name;
  private final int age;
  private final int height;
  private final int weight;
  private final boolean active;
  private final Date dt;

  public PrimitiveSubject(final int id, final String name, final int age, final int height, final int weight, final boolean active, final Date dt) {
    this.id = id;
    this.name = name;
    this.age = age;
    this.height = height;
    this.weight = weight;
    this.active = active;
    this.dt = dt;
  }
}
```

**说明**：

- 对应的也是 `subject` 表。
- 和 AnnotatedSubject 不同，**在其构造方法上，`weight` 和 `height` 方法参数的类型是 `int` ，而不是 Integer 。那么，如果 `subject` 表中的记录，这两个字段为 `NULL` 时，会创建 PrimitiveSubject 对象报错。**

3.BadSubject

```java
public class BadSubject {
  private final int id;
  private final String name;
  private final int age;
  private final Height height;
  private final Double weight;

  public BadSubject(final int id, final String name, final int age, final Height height, final Double weight) {
    this.id = id;
    this.name = name;
    this.age = age;
    this.height = height;
    this.weight = weight == null ? 0 : weight;
  }

  private class Height {

  }
}
```

**说明**：

- 对应的也是 `subject` 表。
- 和 AnnotatedSubject 不同，在其构造方法上，`height` 方法参数的类型是 Height ，而不是 Integer 。因为 MyBatis 无法识别 Height 类，所以会创建 BadSubject 对象报错。（这也就解释了它叫BadSubject的原因）

> 补充：一般情况下，POJO 对象里，不使用基本类型。

4.ExtensiveSubject

```java
public class ExtensiveSubject {
    private final byte aByte;
    private final short aShort;
    private final char aChar;
    private final int anInt;
    private final long aLong;
    private final float aFloat;
    private final double aDouble;
    private final boolean aBoolean;
    private final String aString;

    // enum types
    private final TestEnum anEnum;

    // array types

    // string to lob types:
    private final String aClob;
    private final String aBlob;

    public ExtensiveSubject(final byte aByte,
                            final short aShort,
                            final char aChar,
                            final int anInt,
                            final long aLong,
                            final float aFloat,
                            final double aDouble,
                            final boolean aBoolean,
                            final String aString,
                            final TestEnum anEnum,
                            final String aClob,
                            final String aBlob) {
        this.aByte = aByte;
        this.aShort = aShort;
        this.aChar = aChar;
        this.anInt = anInt;
        this.aLong = aLong;
        this.aFloat = aFloat;
        this.aDouble = aDouble;
        this.aBoolean = aBoolean;
        this.aString = aString;
        this.anEnum = anEnum;
        this.aClob = aClob;
        this.aBlob = aBlob;
    }

    public enum TestEnum {
        AVALUE, BVALUE, CVALUE;
    }
}
```

**说明**：

- 对应的也是 `extensive_subject` 表。
- 这是个复杂对象，基本涵盖了各种类型的数据。

##### AutoConstructorTest

现在我们来查看`org.apache.ibatis.autoconstructor.AutoConstructorTest`这个测试类，一步一步的调试

准备工作：

```java
@BeforeAll
  static void setUp() throws Exception {
    // create a SqlSessionFactory
    try (Reader reader = Resources.getResourceAsReader("org/apache/ibatis/autoconstructor/mybatis-config.xml")) {
      sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
    }

    // populate in-memory database
    BaseDataTest.runScript(sqlSessionFactory.getConfiguration().getEnvironment().getDataSource(),
        "org/apache/ibatis/autoconstructor/CreateDB.sql");
  }
```

- 创建 SqlSessionFactory 对象，基于 `mybatis-config.xml` 配置文件。
- 初始化数据到内存数据库，基于 `CreateDB.sql` SQL 文件。

#### Mybatis 整体架构

 在调试之前，我们有必要对Mybatis的总体结构有一个大致的认知。

MyBatis 的整体架构分为三层：

- 基础支持层
- 核心处理层
- 接口层

图示：

![](http://image.augustrush8.com/images/mybatis/mybatis整体结构.png){:.center}

##### 基础支持层

基础支持层，包含整个 MyBatis 的基础模块，这些模块为核心处理层的功能提供了良好的支撑。

- **反射模块**

对应`org.apache.ibatis.reflection`包

Java 中的反射虽然功能强大，但对大多数开发人员来说，写出高质量的反射代码还是 有一定难度的。MyBatis 中专门提供了反射模块，该模块对 Java 原生的反射进行了良好的封装，提了更加**简洁易用的 API**，方便上层使调用，并且对**反射操作进行了一系列优化**，例如缓存了类的元数据，提高了反射操作的性能。

- **类型模块**

对应`org.apache.ibatis.type`包

1.MyBatis 为简化配置文件提供了**别名机制**，该机制是类型转换模块的主要功能之一。

2.类型转换模块的另一个功能是**实现 JDBC 类型与 Java 类型之间**的转换，该功能在为 SQL 语句绑定实参以及映射查询结果集时都会涉及：

- 在为 SQL 语句绑定实参时，会将数据由 Java 类型转换成 JDBC 类型。
- 而在映射结果集时，会将数据由 JDBC 类型转换成 Java 类型。

- **日志模块**

对应`org.apache.ibatis.logging`包

无论在开发测试环境中，还是在线上生产环境中，日志在整个系统中的地位都是非常重要的。良好的日志功能可以帮助开发人员和测试人员快速定位 Bug 代码，也可以帮助运维人员快速定位性能瓶颈等问题。目前的 Java 世界中存在很多优秀的日志框架，例如 Log4j、 Log4j2、Slf4j 等。

MyBatis 作为一个设计优良的框架，除了提供详细的日志输出信息，还要能够集成多种日志框架，其日志模块的一个主要功能就是**集成第三方日志框架**。

- **IO模块**

对应`org.apache.ibatis.io`包

资源加载模块，主要是对类加载器进行封装，确定类加载器的使用顺序，并提供了加载类文件以及其他资源文件的功能 。

- **解析器模块**

对应`org.apache.ibatis.parsing`包

解析器模块，主要提供了两个功能:

> 一个功能，是对 [XPath](http://www.w3school.com.cn/xpath/index.asp) 进行封装，为 MyBatis 初始化时解析 `mybatis-config.xml` 配置文件以及映射配置文件提供支持。

> 另一个功能，是为处理动态 SQL 语句中的占位符提供支持。

- **数据源模块**

对应`org.apache.ibatis.datasource`包

数据源是实际开发中常用的组件之一。现在开源的数据源都提供了比较丰富的功能，例如，连接池功能、检测连接状态等，选择性能优秀的数据源组件对于提升 ORM 框架乃至整个应用的性能都是非常重要的。

MyBatis **自身提供了相应的数据源实现，当然 MyBatis 也提供了与第三方数据源集成的接口，这些功能都位于数据源模块之中**。

- **事务模块**

对应`org.apache.ibatis.transaction`包

MyBatis 对数据库中的事务进行了抽象，其自身提供了**相应的事务接口和简单实现**。

在很多场景中，MyBatis 会与 Spring 框架集成，并由 **Spring 框架管理事务**。

- **缓存模块**

对应`org.apache.ibatis.cache`包

在优化系统性能时，优化数据库性能是非常重要的一个环节，而添加缓存则是优化数据库时最有效的手段之一。正确、合理地使用缓存可以将一部分数据库请求拦截在缓存这一层。

MyBatis 中提供了**一级缓存和二级缓存**，而这两级缓存都是依赖于基础支持层中的缓 存模块实现的。这里需要读者注意的是，MyBatis 中自带的这两级缓存与 MyBatis 以及整个应用是运行在同一个 JVM 中的，共享同一块堆内存。如果这两级缓存中的数据量较大， 则可能影响系统中其他功能的运行，所以当需要缓存大量数据时，优先考虑使用 Redis、Memcache 等缓存产品。

- **binding模块**

对应`org.apache.ibatis.binding`包

在调用 SqlSession 相应方法执行数据库操作时，需要指定映射文件中定义的 SQL 节点，如果出现拼写错误，我们只能在运行时才能发现相应的异常。为了尽早发现这种错误，MyBatis 通过 Binding 模块，将用户自定义的 Mapper 接口与映射配置文件关联起来，系统可以通过调用自定义 Mapper 接口中的方法执行相应的 SQL 语句完成数据库操作，从而避免上述问题。

值得读者注意的是，开发人员无须编写自定义 Mapper 接口的实现，MyBatis 会自动为其创建动态代理对象。在有些场景中，自定义 Mapper 接口可以完全代替映射配置文件，但有的映射规则和 SQL 语句的定义还是写在映射配置文件中比较方便，例如动态 SQL 语句的定义。

- **注解模块**

对应`org.apache.ibatis.annotations`包

随着 Java 注解的慢慢流行，MyBatis 提供了**注解**的方式，使得我们方便的在 Mapper 接口上编写简单的数据库 SQL 操作代码，而无需像之前一样，必须编写 SQL 在 XML 格式的 Mapper 文件中。虽然说，实际场景下，大家还是喜欢在 XML 格式的 Mapper 文件中编写相应的 SQL 操作。

- **异常模块**

对应`org.apache.ibatis.exceptions`包

定义了 MyBatis 专有的 PersistenceException 和 TooManyResultsException 异常。

##### 核心处理层

在核心处理层中，实现了 MyBatis 的核心处理流程，其中包括 MyBatis 的**初始化**以及完成一次**数据库操作**的涉及的全部流程 。

- **配置解析**

对应`org.apache.ibatis.builder`和`org.apache.ibatis.mapping`模块，前者为配置**解析过程**，后者主要为 SQL 操作解析后的**映射**。

在 MyBatis 初始化过程中，会加载 `mybatis-config.xml` 配置文件、映射配置文件以及 Mapper 接口中的注解信息，解析后的配置信息会形成相应的对象并保存到 Configuration 对象中。例如：

- `<resultMap>`节点(即 ResultSet 的映射规则) 会被解析成 ResultMap 对象。
- `<result>` 节点(即属性映射)会被解析成 ResultMapping 对象。

之后，利用该 Configuration 对象创建 SqlSessionFactory对象。待 MyBatis 初始化之后，开发人员可以通过初始化得到 SqlSessionFactory 创建 SqlSession 对象并完成数据库操作。

- **SQL解析**

对应`org.apache.ibatis.scripting`包

拼凑 SQL 语句是一件烦琐且易出错的过程，为了将开发人员从这项枯燥无趣的工作中 解脱出来，MyBatis 实现**动态 SQL 语句**的功能，提供了多种动态 SQL语句对应的节点。例如`<where>` 节点、`<if>` 节点、`<foreach>`节点等 。通过这些节点的组合使用， 开发人 员可以写出几乎满足所有需求的动态 SQL 语句。

MyBatis 中的 `scripting` 模块，会根据用户传入的实参，解析映射文件中定义的动态 SQL 节点，并形成数据库可执行的 SQL 语句。之后会处理 SQL 语句中的占位符，绑定用户传入的实参。

- **SQL执行**

对应`org.apache.ibatis.executor`包和`org.apache.ibatis.cursor`包，前者对应**执行器**，后者对应执行**结果的游标**。

SQL 语句的执行涉及多个组件 ，其中比较重要的是 Executor、StatementHandler、ParameterHandler 和 ResultSetHandler 。

- **Executor** 主要负责维护一级缓存和二级缓存，并提供事务管理的相关操作，它会将数据库相关操作委托给 StatementHandler完成。
- **StatementHandler** 首先通过 **ParameterHandler** 完成 SQL 语句的实参绑定，然后通过 `java.sql.Statement` 对象执行 SQL 语句并得到结果集，最后通过 **ResultSetHandler** 完成结果集的映射，得到结果对象并返回。

示意图：

![](http://image.augustrush8.com/images/mybatis/sql执行.png){:.center}

- 插件层

对应`org.apache.ibatis.plugin`模块

Mybatis 自身的功能虽然强大，但是并不能完美切合所有的应用场景，因此 MyBatis 提供了插件接口，我们可以通过添加用户自定义插件的方式对 MyBatis 进行扩展。用户自定义插件也可以改变 Mybatis 的默认行为，例如，我们可以拦截 SQL 语句并对其进行重写。

由于用户自定义插件会影响 MyBatis 的核心行为，在使用自定义插件之前，开发人员需要了解 MyBatis 内部的原理，这样才能编写出安全、高效的插件。

#### 接口层

对应`org.apache.ibatis.session`模块

接口层相对简单，其核心是 SqlSession 接口，该接口中定义了 MyBatis 暴露给应用程序调用的 API，也就是上层应用与 MyBatis 交互的桥梁。接口层在接收到调用请求时，会调用核心处理层的相应模块来完成具体的数据库操作。

#### 其他模块

除了上面提到的那些层，还有一些别的模块

- JDBC模块

对应`org.apache.ibatis.jdbc`包

JDBC **单元测试**工具类。

- Lang模块

对应`org.apache.ibatis.lang`包

可暂时不管它







