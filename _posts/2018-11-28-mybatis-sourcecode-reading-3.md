---
layout: post
title: 'Mybatis源码阅读第三话--反射模块'
tags: [read]
---

今天分析的模块是反射模块，即`org.apache.ibatis.reflection`包

![](http://image.augustrush8.com/images/mybatis/reflection.JPG)

之前简介：

> Java 中的反射虽然功能强大，但对大多数开发人员来说，写出高质量的反射代码还是 有一定难度的。MyBatis 中专门提供了反射模块，该模块对 Java 原生的反射进行了良好的封装，提了更加**简洁易用的 API**，方便上层使调用，并且对**反射操作进行了一系列优化**，例如缓存了类的元数据，提高了反射操作的性能。

### Reflector

先看`org.apache.ibatis.reflection.Reflector`这个类：

```java
public class Reflector {
  /**
   * 对应的类
   */
  private final Class<?> type;
  /**
   * 可读属性数组
   */
  private final String[] readablePropertyNames;
  /**
   * 可写属性集合
   */
  private final String[] writablePropertyNames;
  /**
   * 属性对应的 setting 方法的映射。
   *
   * key 为属性名称
   * value 为 Invoker 对象
   */
  private final Map<String, Invoker> setMethods = new HashMap<>();
  /**
   * 属性对应的 getting 方法的映射。
   *
   * key 为属性名称
   * value 为 Invoker 对象
   */
  private final Map<String, Invoker> getMethods = new HashMap<>();
  /**
   * 属性对应的 setting 方法的方法参数类型的映射。{@link #setMethods}
   *
   * key 为属性名称
   * value 为方法参数类型
   */
  private final Map<String, Class<?>> setTypes = new HashMap<>();
  /**
   * 属性对应的 getting 方法的返回值类型的映射。{@link #getMethods}
   *
   * key 为属性名称
   * value 为返回值的类型
   */
  private final Map<String, Class<?>> getTypes = new HashMap<>();
  /**
   * 默认构造方法
   */
  private Constructor<?> defaultConstructor;
  /**
   * 不区分大小写的属性集合
   */
  private Map<String, String> caseInsensitivePropertyMap = new HashMap<>();

  public Reflector(Class<?> clazz) {
    // 设置对应的类
    type = clazz;
    // <1> 初始化 defaultConstructor
    addDefaultConstructor(clazz);
    // <2> // 初始化 getMethods 和 getTypes ，通过遍历 getting 方法
    addGetMethods(clazz);
    // <3> // 初始化 setMethods 和 setTypes ，通过遍历 setting 方法。
    addSetMethods(clazz);
    // <4> // 初始化 getMethods + getTypes 和 setMethods + setTypes ，通过遍历 fields 属性。
    addFields(clazz);
    // <5> 初始化 readablePropertyNames、writeablePropertyNames、caseInsensitivePropertyMap 属性
    readablePropertyNames = getMethods.keySet().toArray(new String[getMethods.keySet().size()]);
    writablePropertyNames = setMethods.keySet().toArray(new String[setMethods.keySet().size()]);
    for (String propName : readablePropertyNames) {
      caseInsensitivePropertyMap.put(propName.toUpperCase(Locale.ENGLISH), propName);
    }
    for (String propName : writablePropertyNames) {
      caseInsensitivePropertyMap.put(propName.toUpperCase(Locale.ENGLISH), propName);
    }
  }
    //后续代码后面分析...
}
```

- `type` 属性，每个 Reflector 对应的类。
- `defaultConstructor` 属性，默认**无参**构造方法。在 `<1>` 处初始化
- `getMethods`、`getTypes` 属性，分别为属性对应的 **getting 方法**、**getting 方法的返回类型**的映射。在 `<2>` 处初始化
- `setMethods`、`setTypes` 属性，分别为属性对应的 **setting 方法**、**setting 方法的参数类型**的映射。在 `<3>` 处初始化
- `<4>` 处，初始化 `getMethods` + `getTypes` 和 `setMethods` + `setTypes` ，通过遍历 fields 属性。
- `<5>` 处，初始化 `readablePropertyNames`、`writeablePropertyNames`、`caseInsensitivePropertyMap` 属性。

#### addDefaultConstructor

查找默认无参构造方法。

```java
private void addDefaultConstructor(Class<?> clazz) {
    // 获得所有构造方法
    Constructor<?>[] consts = clazz.getDeclaredConstructors();
    // 遍历所有构造方法，查找无参的构造方法
    for (Constructor<?> constructor : consts) {
      // 判断无参的构造方法
      if (constructor.getParameterTypes().length == 0) {
        this.defaultConstructor = constructor;
      }
    }
  }
```

#### addGetMethods

初始化 `getMethods` 和 `getTypes` ，通过遍历 getting 方法。

```java
private void addGetMethods(Class<?> cls) {
    // <1> 属性与其 getting 方法的映射。
    Map<String, List<Method>> conflictingGetters = new HashMap<>();
    // <2> 获得所有方法
    Method[] methods = getClassMethods(cls);
    // <3> 遍历所有方法
    for (Method method : methods) {
      // <3.1> 参数大于 0 ，说明不是 getting 方法，忽略
      if (method.getParameterTypes().length > 0) {
        continue;
      }
      String name = method.getName();
      // <3.2> 以 get 和 is 方法名开头，说明是 getting 方法
      if ((name.startsWith("get") && name.length() > 3)
          || (name.startsWith("is") && name.length() > 2)) {
        // <3.3> 获得属性
        name = PropertyNamer.methodToProperty(name);
        // <3.4> 添加到 conflictingGetters 中
        addMethodConflict(conflictingGetters, name, method);
      }
    }
    // <4> 解决 getting 冲突方法
    resolveGetterConflicts(conflictingGetters);
  }
```

看看`addMethodConflict`这个方法：

```java
private void addMethodConflict(Map<String, List<Method>> conflictingMethods, String name, Method method) {
    List<Method> list = conflictingMethods.computeIfAbsent(name, k -> new ArrayList<>());
    list.add(method);
  }
```

#### getClassMethods

```java
private Method[] getClassMethods(Class<?> cls) {
    // 每个方法签名与该方法的映射
    Map<String, Method> uniqueMethods = new HashMap<>();
    Class<?> currentClass = cls;
    // 循环类，类的父类，类的父类的父类，直到父类为 Object
    while (currentClass != null && currentClass != Object.class) {
      // <1> 记录当前类定义的方法
      addUniqueMethods(uniqueMethods, currentClass.getDeclaredMethods());

      // we also need to look for interface methods -
      // because the class may be abstract
      // <2> 记录接口中定义的方法
      Class<?>[] interfaces = currentClass.getInterfaces();
      for (Class<?> anInterface : interfaces) {
        addUniqueMethods(uniqueMethods, anInterface.getMethods());
      }
      // 获得父类
      currentClass = currentClass.getSuperclass();
    }

    Collection<Method> methods = uniqueMethods.values();
    // 转换成 Method 数组返回
    return methods.toArray(new Method[methods.size()]);
  }
```

#### addUniqueMethods

```java
private void addUniqueMethods(Map<String, Method> uniqueMethods, Method[] methods) {
    for (Method currentMethod : methods) {
      // 忽略 bridge 方法，
      // 参见 https://www.zhihu.com/question/54895701/answer/141623158 文章
      if (!currentMethod.isBridge()) {
        // <3> 获得方法签名
        String signature = getSignature(currentMethod);
        // check to see if the method is already known
        // if it is known, then an extended class must have
        // overridden a method
        // 当 uniqueMethods 不存在时，进行添加
        if (!uniqueMethods.containsKey(signature)) {
          // 添加到 uniqueMethods 中
          uniqueMethods.put(signature, currentMethod);
        }
      }
    }
  }
```

`<3>` 处，会调用 `#getSignature(Method method)` 方法，获得方法签名。代码如下：

```java
private String getSignature(Method method) {
    StringBuilder sb = new StringBuilder();
    // 返回类型
    Class<?> returnType = method.getReturnType();
    if (returnType != null) {
      sb.append(returnType.getName()).append('#');
    }
    // 方法名
    sb.append(method.getName());
    // 方法参数
    Class<?>[] parameters = method.getParameterTypes();
    for (int i = 0; i < parameters.length; i++) {
      if (i == 0) {
        sb.append(':');
      } else {
        sb.append(',');
      }
      sb.append(parameters[i].getName());
    }
    return sb.toString();
  }
```

- 最后返回格式：`returnType#方法名:参数名1,参数名2,参数名3` 。

- 例如：`void#checkPackageAccess:java.lang.ClassLoader,boolean` 。

以上对`addGetMethods`方法中的`getClassMethods`方法沿着一条线分析了一下，对框架如何获取方法信息有了一定的了解，大体就是采用反射的方式得到类信息，然后声明一些常用的存储数据结构，如Map，将这些方法信息进行存储并返回。

下面 我们接着分析`addGetMethods`方法下的`resolveGetterConflicts`方法

#### resolveGetterConflicts

`resolveGetterConflicts`方法，解决 getting 冲突方法。最终，一个属性，只保留一个对应的方法。代码如下：

```java
private void resolveGetterConflicts(Map<String, List<Method>> conflictingGetters) {
    // 遍历每个属性，查找其最匹配的方法。因为子类可以覆写父类的方法，所以一个属性，可能对应多个 getting 方法
    for (Entry<String, List<Method>> entry : conflictingGetters.entrySet()) {
      Method winner = null;// 最匹配的方法
      String propName = entry.getKey();
      for (Method candidate : entry.getValue()) {
        // winner 为空，说明 candidate 为最匹配的方法
        if (winner == null) {
          winner = candidate;
          continue;
        }
        // <1> 基于返回类型比较
        Class<?> winnerType = winner.getReturnType();
        Class<?> candidateType = candidate.getReturnType();
        // 类型相同
        if (candidateType.equals(winnerType)) {
          //在 getClassMethods 方法中非boolean方法已经合并，此处再发现就是非法，故抛出异常
          if (!boolean.class.equals(candidateType)) {
            throw new ReflectionException(
                "Illegal overloaded getter method with ambiguous type for property "
                    + propName + " in class " + winner.getDeclaringClass()
                    + ". This breaks the JavaBeans specification and can cause unpredictable results.");
            //// 选择 boolean 类型的 is 方法
          } else if (candidate.getName().startsWith("is")) {
            winner = candidate;
          }
        } else if (candidateType.isAssignableFrom(winnerType)) {
          // OK getter type is descendant
          //因为子类可以修改放大返回值。例如，父类的一个方法的返回值为 List ，
          // 子类对该方法的返回值可以覆写为 ArrayList 。
        } else if (winnerType.isAssignableFrom(candidateType)) {
          winner = candidate;
          // <1.2> 返回类型冲突，抛出 ReflectionException 异常
        } else {
          throw new ReflectionException(
              "Illegal overloaded getter method with ambiguous type for property "
                  + propName + " in class " + winner.getDeclaringClass()
                  + ". This breaks the JavaBeans specification and can cause unpredictable results.");
        }
      }
      // <2> 添加到 getMethods 和 getTypes 中
      addGetMethod(propName, winner);
    }
  }
```

