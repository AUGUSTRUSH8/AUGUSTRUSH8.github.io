---
layout: post
title: 'Mybatis源码阅读第二话--解析器模块'
tags: [read]
---

### 概述

本篇文章分析MyBatis 的解析器模块，对应`org.apache.ibatis.parsing`包当中的一些东西

![](../images/mybatis/parsing.jpg)

前一篇文章当中已经说明了它的作用，这里再来回看一下

解析器模块，主要提供了两个功能:

- 一个功能，是对 [XPath](http://www.w3school.com.cn/xpath/index.asp) 进行封装，为 MyBatis 初始化时解析 `mybatis-config.xml` 配置文件以及映射配置文件提供支持。
- 另一个功能，是为处理动态 SQL 语句中的占位符提供支持。



#### XPathParser

`org.apache.ibatis.parsing.XPathParser`类，基于 Java **XPath** 解析器，用于解析 MyBatis `mybatis-config.xml`和 `**Mapper.xml` 等 XML 配置文件。属性如下：

```java
private final Document document;
  private boolean validation;
  private EntityResolver entityResolver;
  private Properties variables;
  private XPath xpath;
```

```
document
```

 属性，XML 被解析后，生成的 

```
org.w3c.dom.Document
```

 对象。

- `validation` 属性，是否校验 XML 。一般情况下，值为 `true` 。
- `entityResolver` 属性，`org.xml.sax.EntityResolver` 对象，XML 实体解析器。默认情况下，对 XML 进行校验时，会基于 XML 文档开始位置指定的 DTD 文件或 XSD 文件。例如说，解析 `mybatis-config.xml` 配置文件时，会加载 `http://mybatis.org/dtd/mybatis-3-config.dtd` 这个 DTD 文件。但是，如果每个应用启动都从网络加载该 DTD 文件，势必在弱网络下体验非常下，甚至说应用部署在无网络的环境下，还会导致下载不下来，那么就会出现 XML 校验失败的情况。所以，在实际场景下，MyBatis 自定义了 EntityResolver 的实现，达到使用**本地** DTD 文件，从而避免下载**网络** DTD 文件的效果。
- `xpath` 属性，`javax.xml.xpath.XPath` 对象，用于查询 XML 中的节点和元素。
- `variables` 属性，变量 Properties 对象，用来替换需要动态配置的属性值。例如：


```xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
```

`variables` 的来源，即可以在常用的 Java Properties 文件中配置，也可以使用 MyBatis `<property />` 标签中配置。例如：

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

这里配置的 `username` 和 `password` 属性，就可以替换上面的 `${username}` 和 `${password}` 这两个动态属性。



##### 构造方法

XPathParser 的构造方法有16个，我们挑选其中一个作分析

```java
/**
 * 构造 XPathParser 对象
 *
 * @param xml XML 文件地址
 * @param validation 是否校验 XML
 * @param variables 变量 Properties 对象
 * @param entityResolver XML 实体解析器
 */
public XPathParser(String xml, boolean validation, Properties variables, EntityResolver entityResolver) {
    commonConstructor(validation, variables, entityResolver);
    this.document = createDocument(new InputSource(new StringReader(xml)));
  }
```

公用的构造方法`commonConstructor`逻辑如下：

```java
private void commonConstructor(boolean validation, Properties variables, EntityResolver entityResolver) {
    this.validation = validation;
    this.entityResolver = entityResolver;
    this.variables = variables;
    XPathFactory factory = XPathFactory.newInstance();
    this.xpath = factory.newXPath();
  }
```

方法`createDocument`逻辑如下，即完成xml文件到Document对象之间的转化

```java
/**
   * 创建 Document 对象
   *
   * @param inputSource XML 的 InputSource 对象
   * @return Document 对象
   */
  private Document createDocument(InputSource inputSource) {
    // important: this must only be called AFTER common constructor
    try {
      // 1> 创建 DocumentBuilderFactory 对象
      DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();
      factory.setValidating(validation);// 设置是否验证 XML

      factory.setNamespaceAware(false);
      factory.setIgnoringComments(true);
      factory.setIgnoringElementContentWhitespace(false);
      factory.setCoalescing(false);
      factory.setExpandEntityReferences(true);
      // 2> 创建 DocumentBuilder 对象
      DocumentBuilder builder = factory.newDocumentBuilder();
      builder.setEntityResolver(entityResolver);// 设置实体解析器
      builder.setErrorHandler(new ErrorHandler() {
        @Override
        public void error(SAXParseException exception) throws SAXException {
          throw exception;
        }

        @Override
        public void fatalError(SAXParseException exception) throws SAXException {
          throw exception;
        }

        @Override
        public void warning(SAXParseException exception) throws SAXException {
        }
      });
      // 3> 解析 XML 文件
      return builder.parse(inputSource);
    } catch (Exception e) {
      throw new BuilderException("Error creating document instance.  Cause: " + e, e);
    }
  }
```



##### eval方法族

XPathParser 类提供了一系列的 `eval*` 方法，用于获得 Boolean、Short、Integer、Long、Float、Double、String、Node 类型的元素或节点的“值”。当然，虽然方法很多，但是都是基于 `evaluate(String expression, Object root, QName returnType)` 方法，下面我们来看看这个方法：

```java
/**
   * 获得指定元素或节点的值
   * @param expression
   * @param root 指定节点
   * @param returnType 返回值类型
   * @return 值
   */
  private Object evaluate(String expression, Object root, QName returnType) {
    try {
      return xpath.evaluate(expression, root, returnType);
    } catch (Exception e) {
      throw new BuilderException("Error evaluating XPath.  Cause: " + e, e);
    }
  }
```

最终还是调用 `xpath` 的 `evaluate(String expression, Object root, QName returnType)` 方法，获得指定元素或节点的值。

- eval元素

eval 元素的方法，用于获得 Boolean、Short、Integer、Long、Float、Double、String 类型的**元素**的值。我们以 `evalString(Object root, String expression)` 方法为例子，看看代码：

```java
/**
   * 解析获得元素的值
   * @param root
   * @param expression
   * @return
   */
  public String evalString(Object root, String expression) {
    // <1> 获得值
    String result = (String) evaluate(expression, root, XPathConstants.STRING);
    // <2> 基于 variables 替换动态值，如果 result 为动态值
    result = PropertyParser.parse(result, variables);
    return result;
  }
```

**1.**`<1>` 处，调用 `#evaluate(String expression, Object root, QName returnType)` 方法，获得值。其中，`returnType` 方法传入的是 `XPathConstants.STRING` ，表示返回的值是 String 类型。

**2.** `<2>` 处，调用 `PropertyParser#parse(String string, Properties variables)` 方法，基于 `variables` 替换**动态值**，如果 `result` 为**动态值**。**这就是 MyBatis 如何替换掉 XML 中的动态值实现的方式。**

- eval节点

```java
public List<XNode> evalNodes(String expression) {
    return evalNodes(document, expression);
  }

  public List<XNode> evalNodes(Object root, String expression) {
    List<XNode> xnodes = new ArrayList<>();
    // <1> 获得 Node 数组
    NodeList nodes = (NodeList) evaluate(expression, root, XPathConstants.NODESET);
    // <2> 封装成 XNode 数组
    for (int i = 0; i < nodes.getLength(); i++) {
      xnodes.add(new XNode(this, nodes.item(i), variables));
    }
    return xnodes;
  }

  public XNode evalNode(String expression) {
    return evalNode(document, expression);
  }

  public XNode evalNode(Object root, String expression) {
    Node node = (Node) evaluate(expression, root, XPathConstants.NODE);
    if (node == null) {
      return null;
    }
    return new XNode(this, node, variables);
  }
```

1.`<1>` 处，返回结果有 Node **对象**和**数组**两种情况，根据方法参数 `expression` 需要获取的节点不同。

2.`<2>` 处， 最终结果会将 Node 封装成 `org.apache.ibatis.parsing.XNode` 对象，主要为了**动态值的替换**



#### GenericTokenParser

**通用**的 Token 解析器。

```java
public class GenericTokenParser {

  private final String openToken;
  private final String closeToken;
  private final TokenHandler handler;

  public GenericTokenParser(String openToken, String closeToken, TokenHandler handler) {
    this.openToken = openToken;
    this.closeToken = closeToken;
    this.handler = handler;
  }

  public String parse(String text) {
    if (text == null || text.isEmpty()) {
      return "";
    }
    // search open token
    int start = text.indexOf(openToken);
    if (start == -1) {
      return text;
    }
    char[] src = text.toCharArray();
    int offset = 0;
    final StringBuilder builder = new StringBuilder();
    StringBuilder expression = null;
    while (start > -1) {
      if (start > 0 && src[start - 1] == '\\') {
        // this open token is escaped. remove the backslash and continue.
        builder.append(src, offset, start - offset - 1).append(openToken);
        offset = start + openToken.length();
      } else {
        // found open token. let's search close token.
        if (expression == null) {
          expression = new StringBuilder();
        } else {
          expression.setLength(0);
        }
        builder.append(src, offset, start - offset);
        offset = start + openToken.length();
        int end = text.indexOf(closeToken, offset);
        while (end > -1) {
          if (end > offset && src[end - 1] == '\\') {
            // this close token is escaped. remove the backslash and continue.
            expression.append(src, offset, end - offset - 1).append(closeToken);
            offset = end + closeToken.length();
            end = text.indexOf(closeToken, offset);
          } else {
            expression.append(src, offset, end - offset);
            offset = end + closeToken.length();
            break;
          }
        }
        if (end == -1) {
          // close token was not found.
          builder.append(src, start, src.length - start);
          offset = src.length;
        } else {
          builder.append(handler.handleToken(expression.toString()));
          offset = end + closeToken.length();
        }
      }
      start = text.indexOf(openToken, offset);
    }
    if (offset < src.length) {
      builder.append(src, offset, src.length - offset);
    }
    return builder.toString();
  }
}
```

代码看起来好冗长，但是淡定，就一个 `parse(String text)` 方法，**循环**( 因为可能不只一个 )，解析以 `openToken` 开始，以 `closeToken` 结束的 Token ，并提交给 `handler` 进行处理



#### PropertyParser

动态属性解析器。

```java
public static String parse(String string, Properties variables) {
    VariableTokenHandler handler = new VariableTokenHandler(variables);
    GenericTokenParser parser = new GenericTokenParser("${", "}", handler);
    return parser.parse(string);
  }
```

关键看看上面这个类方法，由此可以得知，此类方法主要针对XML当中的`${name}`和`${password}`动态值进行赋值解析。



#### TokenHandler 

Token 处理器接口

```java
public interface TokenHandler {
  String handleToken(String content);
}
```

这个接口的继承类

![](../images/mybatis/tokenhandler.jpg)

再`parsing`这个包当中我们已经在`PropertyParser`这个类中用到了`VariableTokenHandler`,`VariableTokenHandler`是`PropertyParser`的静态内部类。



#### VariableTokenHandler

- 构造方法

```java
private static class VariableTokenHandler implements TokenHandler {
    private final Properties variables;
    private final boolean enableDefaultValue;//是否开启默认值功能。默认为 {@link #ENABLE_DEFAULT_VALUE}
    private final String defaultValueSeparator;//默认值的分隔符。默认为 {@link #KEY_DEFAULT_VALUE_SEPARATOR} ，即 ":" 。

    private VariableTokenHandler(Properties variables) {
      this.variables = variables;
      this.enableDefaultValue = Boolean.parseBoolean(getPropertyValue(KEY_ENABLE_DEFAULT_VALUE, ENABLE_DEFAULT_VALUE));
      this.defaultValueSeparator = getPropertyValue(KEY_DEFAULT_VALUE_SEPARATOR, DEFAULT_VALUE_SEPARATOR);
    }
    private String getPropertyValue(String key, String defaultValue) {
      return (variables == null) ? defaultValue : variables.getProperty(key, defaultValue);
    }
}
```

1.`variables` 属性，变量 Properties 对象。

2.`enableDefaultValue` 属性，是否开启默认值功能。默认为 `ENABLE_DEFAULT_VALUE` ，即**不开启**。想要开启，可以配置如下：

```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- Enable this feature -->
</properties>
```

`defaultValueSeparator` 属性，默认值的分隔符。默认为 `KEY_DEFAULT_VALUE_SEPARATOR` ，即 `":"` 。想要修改，可以配置如下：

```xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value="?:"/> <!-- Change default value of separator -->
</properties>
```

- handleToken方法

```java
public String handleToken(String content) {
      if (variables != null) {
        String key = content;
        // 开启默认值功能
        if (enableDefaultValue) {
          // 查找默认值
          final int separatorIndex = content.indexOf(defaultValueSeparator);
          String defaultValue = null;
          if (separatorIndex >= 0) {
            key = content.substring(0, separatorIndex);
            defaultValue = content.substring(separatorIndex + defaultValueSeparator.length());
          }
          // 有默认值，优先替换，不存在则返回默认值
          if (defaultValue != null) {
            return variables.getProperty(key, defaultValue);
          }
        }
        // 未开启默认值功能，直接替换
        if (variables.containsKey(key)) {
          return variables.getProperty(key);
        }
      }
      // 无 variables ，直接返回
      return "${" + content + "}";
    }
  }
```

