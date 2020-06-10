---
layout: post
title: '设计模式之原型模式
tags: [code]
---

## 背景

相信大多数Java开发者在开发工作当中都遇见过这样的大篇`getter()`，`setter()`代码：

```java
    public void setId(Integer id) {
        this.id = id;
    }

    public void setOwner(String owner) {
        this.owner = owner == null ? null : owner.trim();
    }

    //......
```

此时如果我们创建一个对象，然后挨个`set`赋值，这样的代码非常工整，也很好理解，但问题是，这样的代码足够优雅吗？实际上不够优雅。原型模式就能帮我们解决这样的尴尬问题。

## 定义

原型模式（Prototype Pattern）是指原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。 

## 适用场景

- 类初始化消耗资源较多
- new 产生一个对象需要非常繁琐的过程（数据准备、访问权限等等）
- 构造函数比较复杂
- 循环体中产生大量对象

## 应用

- 在 Spring 中，原型模式应用得非常广泛。例如 scope=“prototype” 

- JSON.parseObject()也是一种原型模式 

## 原型模式之简单克隆

一个标准的原型模式，是这样设计的，首先创建一个`Prototype`接口。

```java
public interface Prototype {
    Prototype clone();
}
```

创建具体需要克隆的对象 `ConcretePrototype`

```java
public class ConcretePrototypeA implements Prototype{
    private int age;
    private String name;
    private List hobbies;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public List getHobbies() {
        return hobbies;
    }

    public void setHobbies(List hobbies) {
        this.hobbies = hobbies;
    }

    @Override
    public Prototype clone() {
        ConcretePrototypeA concretePrototypeA = new ConcretePrototypeA();
        concretePrototypeA.setAge(this.age);
        concretePrototypeA.setName(this.name);
        concretePrototypeA.setHobbies(this.hobbies);
        return concretePrototypeA;
    }
}

```

创建`Client`类

 ```java
public class Client {
    private Prototype prototype;

    public Client(Prototype prototype) {
        this.prototype=prototype;
    }

    public Prototype startPrototype(Prototype concretePrototype) {
        return concretePrototype.clone();
    }
}
 ```

测试代码

```java
public class PrototypeTest {
    public static void main(String[] args) {
        //创建一个具体需要克隆的对象
        ConcretePrototypeA concretePrototypeA=new ConcretePrototypeA();
        //填充属性
        concretePrototypeA.setAge(45);
        concretePrototypeA.setName("hello");
        concretePrototypeA.setHobbies(new ArrayList<String>());
        System.out.println(concretePrototypeA);

        //创建Client对象，准备克隆
        Client client = new Client(concretePrototypeA);
        ConcretePrototypeA concretePrototypeB = (ConcretePrototypeA) client.startClone(concretePrototypeA);
        System.out.println(concretePrototypeB);

        System.out.println("克隆对象中的引用类型地址值: "+concretePrototypeB.getHobbies());
        System.out.println("原对象中的引用类型地址值: "+concretePrototypeA.getHobbies());
        System.out.println("对象地址比较: "+(concretePrototypeA.getHobbies()==concretePrototypeB.getHobbies()));
    }
}
```

输出结果

```java
com.admin.hellotest.ConcretePrototypeA@5cb0d902
com.admin.hellotest.ConcretePrototypeA@46fbb2c1
克隆对象中的引用类型地址值: []
原对象中的引用类型地址值: []
对象地址比较: true
```

从测试结果看出`hobbies`的引用地址相同，所以如果我们修改`concretePrototypeA`或者`concretePrototypeB`的`hobbies`值，那么都会改变。这就常说的浅克隆，没有复制引用对象。下面我们使用深克隆进行改造。

## 原型模式之深度克隆

我们换一个场景，大家都知道齐天大圣。首先它是一只猴子，有七十二般变化，把一根毫毛就可以吹出千万个泼猴，手里还拿着金箍棒，金箍棒可以变大变小。这就是我们耳熟能详的原型模式的经典体现。 

创建原型猴子 `Monkey` 类： 

```java
public class Monkey {
    public int height;
    public int weight;
    public Date birthday;
}
```

创建`Jingubang`类：

```java
public class Jingubang implements Serializable{
    public float h=100;
    public float d=10;
    public void big(){
        this.h *=2;
        this.d *=2;
    }

    public void small(){
        this.h /=2;
        this.d /=2;
    }
}
```

创建具体的对象齐天大圣 `QiTianDaSheng` 类： 

```java
public class QiTianDaSheng extends Monkey implements Cloneable, Serializable {
    public Jingubang jingubang;

    public QiTianDaSheng() {
        this.birthday=new Date();
        this.jingubang=new Jingubang();
    }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return this.deepClone();
    }

    public Object deepClone() {
        try {
            ByteArrayOutputStream byteArrayOutputStream =new ByteArrayOutputStream();
            ObjectOutputStream objectOutputStream=new ObjectOutputStream(byteArrayOutputStream);
            objectOutputStream.writeObject(this);

            ByteArrayInputStream byteArrayInputStream=new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
            ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);

            QiTianDaSheng copy = (QiTianDaSheng) objectInputStream.readObject();
            copy.birthday=new Date();
            return copy;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }

    public QiTianDaSheng shallowClone(QiTianDaSheng target){
        QiTianDaSheng qiTianDaSheng=new QiTianDaSheng();
        qiTianDaSheng.height=target.height;
        qiTianDaSheng.weight=target.weight;
        qiTianDaSheng.jingubang=target.jingubang;
        qiTianDaSheng.birthday=new Date();
        return qiTianDaSheng;

    }
}
```

测试代码`DeepCloneTest `:

```java
public class DeepCloneTest {
    public static void main(String[] args) {
        QiTianDaSheng qiTianDaSheng = new QiTianDaSheng();
        try {
            QiTianDaSheng clone = (QiTianDaSheng) qiTianDaSheng.clone();
            System.out.println("深度克隆："+(qiTianDaSheng.jingubang==clone.jingubang));
        } catch (Exception e) {
            e.printStackTrace();
        }

        QiTianDaSheng qiTianDaSheng1=new QiTianDaSheng();
        QiTianDaSheng s = qiTianDaSheng.shallowClone(qiTianDaSheng1);
        System.out.println("浅克隆："+(qiTianDaSheng1.jingubang==s.jingubang));
    }
}
```

运行结果：

```java
深度克隆：false
浅克隆：true
```

## 克隆破坏单例模式 

现在想象一下，如果我们克隆的对象是单例对象，那么按照上面的逻辑，深克隆就肯定会破坏单例。防止克隆破坏单例的思路有两个：

- 单例类不实现`Cloneable`接口
- 重写`clone()`方法，在clone方法当中返回单例对象即可

第二种思路代码如下：

```java
@Override
protected Object clone() throws CloneNotSupportedException {
	return INSTANCE;
}
```

