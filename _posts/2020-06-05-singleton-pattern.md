---
layout: post
title: '设计模式之工厂模式'
tags: [code]
---

## 单例模式的定义

单例模式是指确保一个类在任何情况下都绝对只有一个实例，并提供一个全局访问点。单例模式属于`创建型模式`

## 单例模式的应用场景

- 现实生活中：部门经理、公司CEO等
- J2EE标准中
  - ServletContext
  - ServletContextConfig 
- Spring框架中：ApplicationContext 
- 数据库连接池

## 饿汉式单例

饿汉式单例，顾名思义，就是指在类加载的时候就立即初始化。由于其在线程出现之前就实例化了，所以其绝对线程安全，不可能存在访问安全问题。

- 优点：没有加任何的锁、执行效率比较高，在用户体验上来说，比懒汉式更好。 
- 缺点：由于在类加载的时候就初始化，不论后面用或者不用都占据着存储空间，所以存在着内存浪费的问题。

它有两种典型的写法：

第一种：

```java
public class HungrySingleton {
    private static HungrySingleton hungrySingleton= new HungrySingleton();
    private HungrySingleton(){}

    public static HungrySingleton getInstance() {
        return hungrySingleton;
    }
}
```

第二种（利用静态代码块的机制）：

```java
public class HungrySingleton {
    private static HungrySingleton hungrySingleton;
    static {
        hungrySingleton = new HungrySingleton();
    }
    private HungrySingleton(){}

    public static HungrySingleton getInstance() {
        return hungrySingleton;
    }
}
```

饿汉式单例的一个典型应用场景就是Spring 中的 IOC 容器 `ApplicationContext`。可以看出饿汉式单例的写法很简单，也很好理解，其主要适用于单例对象比较少的情况。  

## 懒汉式单例

**特点：** 被外部类调用的时候内部类才会加载。

来看一个简单的懒汉式单例代码：

```java
public class LazySimpleSingleton {
    private LazySimpleSingleton() {

    }
    private static LazySimpleSingleton lazySimpleSingleton;

    public static LazySimpleSingleton getInstance() {
        if (lazySimpleSingleton == null) {
            return new LazySimpleSingleton();
        }
        return lazySimpleSingleton;
    }
}
```

再来一个ExecutorThread类：

```java
public class ExecutorThread implements Runnable {
    @Override
    public void run() {
        LazySimpleSingleton singleton = LazySimpleSingleton.getInstance();
        System.out.println(Thread.currentThread().getName()+":"+singleton);
    }
}
```

最后来一个客户端调用测试代码：

```java
public class LazySimpleSingletonTest {
    public static void main(String[] args) {
        Thread thread1 = new Thread(new ExecutorThread());
        Thread thread2 = new Thread(new ExecutorThread());
        thread1.start();
        thread2.start();
        System.out.println("end");
    }
}
```

看看输出结果:

```php
end
Thread-0:com.admin.hellotest.LazySimpleSingleton@1228a75b
Thread-1:com.admin.hellotest.LazySimpleSingleton@385b6ab5
```

可以看出创建了两个不同的对象，其实这个结果稍加分析就能知道是在哪个环节出了问题，想要看到更直观的效果，可以以`Thread模式`进行debug，具体操作可以百度。以上结果就表示上面的单例写法存在多线程安全隐患。那么，应该如何优化代码，使得以上的懒汉式单例在多线程环境下安全呢？我们把`getInstance()`方法加上`synchronized`关键字，此时我们再来调试，放其中一个线程调用并执行`getInstance()`方法，然后再放另一个线程调用`getInstance()`方法，可以发现其线程状态由`Running`变成了`Monitor`，如下图所示，出现了阻塞。直到第一个线程执行完，第二个线程才恢复`Running`状态继续调用`getInstance()`方法。 

![](http://image.augustrush8.com/images/2020-06-06线程状态.png)

以上使用`synchronized`关键字解决了线程安全问题，虽然我们知道`synchronized`在Java 1.7以后进行了锁升级相关的优化操作，但在线程比较多的情况下，还是会暴露出性能问题，如果CPU的分配压力上升，就会导致大批量的线程出现阻塞。那么，有没有一种更好的方式，既兼顾线程安全又提升程序性能呢？答案是肯定的。我们来看双重检查锁的单例模式： 

```java
public class LazyDoubleCheckSingleton {
    private static LazyDoubleCheckSingleton lazyDoubleCheckSingleton;
    private LazyDoubleCheckSingleton() {

    }

    public static LazyDoubleCheckSingleton getInstance() {
        if (lazyDoubleCheckSingleton == null) {
            synchronized (LazyDoubleCheckSingleton.class) {
                if (lazyDoubleCheckSingleton == null) {
                    lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton();
                }
            }
        }
        return lazyDoubleCheckSingleton;
    }
}
```

以上双检锁的方式，阻塞并不是基于整个 LazySimpleSingleton 类的阻塞，而是在 getInstance()方法内部阻塞，只要逻辑不是太复杂，对于调用者而言感知不到。 那到底有没有不用上锁的方法呢？其实我们可以使用静态内部类的方式来实现。

```java
public class LazyInnerClassSingleton {
    private LazyInnerClassSingleton() {

    }

    public static LazyInnerClassSingleton getInstance() {
        return LazyHolder.LAZY;
    }

    private static class LazyHolder{
        private static final LazyInnerClassSingleton LAZY= new LazyInnerClassSingleton();
    }
}
```

内部类在方法调用之前初始化，既兼顾了饿汉式的内存浪费问题，还兼顾了`synchronized`的性能问题。

## 反射破坏单例模式

还是以上面最后一种懒汉式单例写法为例，这里重写测试代码：

```java
public class LazyInnerClassSingletonTest {
    public static void main(String[] args) {
        try {
            Class<?> clazz = LazyInnerClassSingleton.class;
            Constructor constructor = clazz.getDeclaredConstructor(null);
            constructor.setAccessible(true);
            Object o1 = constructor.newInstance();
            Object o2 = constructor.newInstance();
            System.out.println(o1==o2);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

运行结果如下：

```php
false
```

很显然，通过反射的方式创建了两个不同的对象。现在对构造方法做点文章，使得一旦出现多次重复创建就抛出异常，改完以后是这样的，注意跟之前对比其中的构造方法。

```java
package com.admin.hellotest;

public class LazyInnerClassSingleton {
    private LazyInnerClassSingleton() {
        if (LazyHolder.LAZY != null) {
            throw new RuntimeException("不允许创建多个实例");
        }
    }

    public static LazyInnerClassSingleton getInstance() {
        return LazyHolder.LAZY;
    }

    private static class LazyHolder{
        private static final LazyInnerClassSingleton LAZY= new LazyInnerClassSingleton();
    }
}

```

然后还是上面的测试方法进行测试，看看输出结果：

```php
java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
	at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
	at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
	at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
	at com.admin.hellotest.LazyInnerClassSingletonTest.main(LazyInnerClassSingletonTest.java:11)
Caused by: java.lang.RuntimeException: 不允许创建多个实例
	at com.admin.hellotest.LazyInnerClassSingleton.<init>(LazyInnerClassSingleton.java:6)
	... 5 more
```

可以看到直接就报错了。

## 序列化破坏单例

当我们将一个单例对象创建好，有时候需要将对象序列化然后写入到磁盘，下次使用时再从磁盘中读取到对象，反序列化转化为内存对象。反序列化后的对象会重新分配内存，即重新创建。那如果序列化的目标的对象为单例对象，就违背了单例模式的初衷，相当于破坏了单例，来看一段代码： 

```java
public class SerializableSingleton implements Serializable {
    public static final SerializableSingleton instance = new SerializableSingleton();

    private SerializableSingleton() {

    }

    public static SerializableSingleton getInstance() {
        return instance;
    }
}
```

编写测试代码：

```java
public class SeriableSingletonTest {
    public static void main(String[] args) {
        SerializableSingleton s1=null;
        SerializableSingleton s2=SerializableSingleton.getInstance();
        FileOutputStream fileOutputStream;
        try {
            fileOutputStream = new FileOutputStream("SeriableSingleton.obj");
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(fileOutputStream);
            objectOutputStream.writeObject(s2);
            objectOutputStream.flush();
            objectOutputStream.close();

            FileInputStream fileInputStream = new FileInputStream("SeriableSingleton.obj");
            ObjectInputStream objectInputStream = new ObjectInputStream(fileInputStream);
            s1 = (SerializableSingleton) objectInputStream.readObject();
            objectInputStream.close();
            System.out.println(s1);
            System.out.println(s2);
            System.out.println(s1 == s2);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

输出结果：

```php
com.admin.hellotest.SerializableSingleton@72d818d1
com.admin.hellotest.SerializableSingleton@59e84876
false
```

结果还是很明显，反序列化后的对象和手动创建的对象不一样，同样违背了单例的初衷，那么如何保证反序列化的情况也实现单例呢，其实只需要增加`readResolve()`方法即可。

```java
public class SerializableSingleton implements Serializable {
    public static final SerializableSingleton instance = new SerializableSingleton();

    private SerializableSingleton() {

    }

    public static SerializableSingleton getInstance() {
        return instance;
    }
    private Object readResolve() {
        return instance;
    }
}
```

再来测试看看结果：

```php
com.admin.hellotest.SerializableSingleton@443b7951
com.admin.hellotest.SerializableSingleton@443b7951
true
```

这好像有点玄乎了啊，为什么呢？来看看`ObjectInputStream`的`readObject()`方法源码：

```java
public final Object readObject()
        throws IOException, ClassNotFoundException
    {
        if (enableOverride) {
            return readObjectOverride();
        }

        // if nested read, passHandle contains handle of enclosing object
        int outerHandle = passHandle;
        try {
            Object obj = readObject0(false);
            handles.markDependency(outerHandle, passHandle);
            ClassNotFoundException ex = handles.lookupException(passHandle);
            if (ex != null) {
                throw ex;
            }
            if (depth == 0) {
                vlist.doCallbacks();
            }
            return obj;
        } finally {
            passHandle = outerHandle;
            if (closed && depth == 0) {
                clear();
            }
        }
    }
```

可以看到`readObject()`方法当中又调用了`readObject0()`方法。

```
private Object readObject0(boolean unshared) throws IOException {
    ...
    case TC_OBJECT:
    return checkResolve(readOrdinaryObject(unshared));
    ...
}
```

其中有以上这样的一个Case语句，又调用了`readOrdinaryObject()`方法，继续跟进看看。

```
private Object readOrdinaryObject(boolean unshared)
        throws IOException
    {
        if (bin.readByte() != TC_OBJECT) {
            throw new InternalError();
        }

        ObjectStreamClass desc = readClassDesc(false);
        desc.checkDeserialize();

        Class<?> cl = desc.forClass();
        if (cl == String.class || cl == Class.class
                || cl == ObjectStreamClass.class) {
            throw new InvalidClassException("invalid class descriptor");
        }

        Object obj;
        try {
            obj = desc.isInstantiable() ? desc.newInstance() : null;
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }

        passHandle = handles.assign(unshared ? unsharedMarker : obj);
        ClassNotFoundException resolveEx = desc.getResolveException();
        if (resolveEx != null) {
            handles.markException(passHandle, resolveEx);
        }

        if (desc.isExternalizable()) {
            readExternalData((Externalizable) obj, desc);
        } else {
            readSerialData(obj, desc);
        }

        handles.finish(passHandle);

        if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            Object rep = desc.invokeReadResolve(obj);
            if (unshared && rep.getClass().isArray()) {
                rep = cloneArray(rep);
            }
            if (rep != obj) {
                // Filter the replacement object
                if (rep != null) {
                    if (rep.getClass().isArray()) {
                        filterCheck(rep.getClass(), Array.getLength(rep));
                    } else {
                        filterCheck(rep.getClass(), -1);
                    }
                }
                handles.setObject(passHandle, obj = rep);
            }
        }

        return obj;
    }
```

上面的`desc.isInstantiable()`会判断一下构造方法是否为空，构造方法不为空就返回`true`，看这句`obj = desc.isInstantiable() ? desc.newInstance() : null;`的意思就是只要有无参构造方法就会实例化。而后的代码当中会判断`desc.hasReadResolveMethod()`，`hasReadResolveMethod()`方法体是这样的：

```java
boolean hasReadResolveMethod() {
    requireInitialized();
    return (readResolveMethod != null);
}
```

逻辑非常简单，就是判断 `readResolveMethod` 是否为空，不为空就返回 `true`。 那么`readResolveMethod`变量是在哪里赋值的呢，全局查找，得到这样一段代码：

```java
readResolveMethod = getInheritableMethod(
                        cl, "readResolve", null, Object.class);
```

其逻辑其实就是通过反射找到一个无参的 `readResolve()`方法，并且保存下来。 继续回到`readOrdinaryObject()`方法，可以看到在判断完`hasReadResolveMethod()`为真以后就执行了`Object rep = desc.invokeReadResolve(obj);`，`invokeReadResolve()`方法是这样的：

```
Object invokeReadResolve(Object obj)
        throws IOException, UnsupportedOperationException
    {
        requireInitialized();
        if (readResolveMethod != null) {
            try {
                return readResolveMethod.invoke(obj, (Object[]) null);
            } catch (InvocationTargetException ex) {
                Throwable th = ex.getTargetException();
                if (th instanceof ObjectStreamException) {
                    throw (ObjectStreamException) th;
                } else {
                    throwMiscException(th);
                    throw new InternalError(th);  // never reached
                }
            } catch (IllegalAccessException ex) {
                // should not occur, as access checks have been suppressed
                throw new InternalError(ex);
            }
        } else {
            throw new UnsupportedOperationException();
        }
    }
```

可以看到在 `invokeReadResolve()`方法中用反射调用了 `readResolveMethod` 方法。通过 JDK 源码分析我们可以看出，虽然，增加 `readResolve()`方法返回实例，解决了单例被破坏的问题。但是，我们通过分析源码以及调试，我们可以看到实际上实例化了两次，也就是上面的`obj = desc.isInstantiable() ? desc.newInstance() : null;`会实例化一次，后面反射调用`readResolveMethod`也会实例化一次，只不过前面新创建的对象没有被返回而已。那如果，创建对象的动作发生频率增大，就意味着内存分配开销也就随之增大，难道真的就没办法从根本上解决问题吗？ 

## 注册式单例

注册式单例又称为登记式单例，就是将每一个实例都登记到某一个地方，使用唯一的标识获取实例。注册式单例有两种写法：一种为容器缓存，一种为枚举登记。先来看枚举式单例的写法，来看代码，创建 EnumSingleton 类： 

```java
public enum EnumSingleton {
    INSTANCE;
    private Object data;

    public Object getData() {
        return data;
    }

    public void setData(Object data) {
        this.data = data;
    }

    public static EnumSingleton getInstance() {
        return INSTANCE;
    }
}
```

这里直接说结论，枚举式单例是反序列化安全的。这里我们没有做任何的处理，却发现运行结果和我们预期的一致，为什么呢？还是来看源码，源码之下藏无可藏，我们首先用[jad工具](<http://www.javadecompilers.com/jad>)拿到以上代码的反编译代码，可以得到这样一段：

```java
    static 
    {
        INSTANCE = new EnumSingleton("INSTANCE", 0);
        $VALUES = (new EnumSingleton[] {
            INSTANCE
        });
    }
}

```

原来，枚举式单例在静态代码块中就给 INSTANCE 进行了赋值，是饿汉式单例的实现。 此外，还是回到`ObjectInputStream `的`readObject0 ()`方法：

```java
private Object readObject0(boolean unshared) throws IOException {
    ...
    case TC_ENUM:
    return checkResolve(readEnum(unshared));
    ...
}
```

看看在 `readObject0()`中的 `readEnum()` 方法:

```java
private Enum<?> readEnum(boolean unshared) throws IOException {
        if (bin.readByte() != TC_ENUM) {
            throw new InternalError();
        }

        ObjectStreamClass desc = readClassDesc(false);
        if (!desc.isEnum()) {
            throw new InvalidClassException("non-enum class: " + desc);
        }

        int enumHandle = handles.assign(unshared ? unsharedMarker : null);
        ClassNotFoundException resolveEx = desc.getResolveException();
        if (resolveEx != null) {
            handles.markException(enumHandle, resolveEx);
        }

        String name = readString(false);
        Enum<?> result = null;
        Class<?> cl = desc.forClass();
        if (cl != null) {
            try {
                @SuppressWarnings("unchecked")
                Enum<?> en = Enum.valueOf((Class)cl, name);
                result = en;
            } catch (IllegalArgumentException ex) {
                throw (IOException) new InvalidObjectException(
                    "enum constant " + name + " does not exist in " +
                    cl).initCause(ex);
            }
            if (!unshared) {
                handles.setObject(enumHandle, result);
            }
        }

        handles.finish(enumHandle);
        passHandle = enumHandle;
        return result;
    }
```

我们发现枚举类型其实通过类名和 Class 对象类找到一个唯一的枚举对象。因此，枚举对象不可能被类加载器加载多次。 

那么反射又能不能破坏枚举式单例呢？答案是也不能，来看看`Constructor`类的`newInstance()`方法：

```java
public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, null, modifiers);
            }
        }
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }
```

可以看到在`newInstance()`方法中做了强制性的判断，如果修饰符是 `Modifier.ENUM` 枚举类型，直接抛出异常。 枚举式单例也是《Effective Java》书中推荐的一种单例实现写法。在 JDK 枚举的语法特殊性，以及反射也为枚举保驾护航，让枚举式单例成为一种比较优雅的实现。 

注册式单例还有另一种写法，容器缓存的写法，创建 ContainerSingleton 类： 

```java
public class ContainerSingleton {
    private ContainerSingleton(){}
    private static Map<String,Object> ioc = new ConcurrentHashMap<String,Object>();
    public static Object getBean(String className){
        synchronized (ioc) {
            if (!ioc.containsKey(className)) {
                Object obj = null;
                try {
                    obj = Class.forName(className).newInstance();
                    ioc.put(className, obj);
                } catch (Exception e) {
                    e.printStackTrace();
                } 
                return obj;
            } else {
                return ioc.get(className);
            }
        }
    }
}
```

容器式写法适用于创建实例非常多的情况，便于管理。但是，是非线程安全的。到此，注册式单例介绍完毕。 我们还可以来看看 Spring 中的容器式单例的实现代码： 

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
		implements AutowireCapableBeanFactory {
    /** Cache of unfinished FactoryBean instances: FactoryBean name --> BeanWrapper */
    private final Map<String, BeanWrapper> factoryBeanInstanceCache = new ConcurrentHashMap<>(16);
...
}
```

