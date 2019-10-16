---
layout: post
title: 'Java异常封装-自定义错误码和描述'
tags: [read]
---

Java里面的异常在真正工作中使用还是十分普遍的。什么时候该抛出什么异常，这个是必须知道的。

### checked异常和unchecked异常

- **checked异常**

表示无效，不是程序中可以预测的。比如无效的用户输入，文件不存在，网络或者数据库链接错误。这些都是**外在的原因**，都不是程序内部可以控制的。

必须在代码中显式地处理。比如try-catch块处理，或者给所在的方法加上throws说明，将异常抛到调用栈的上一层。继承自java.lang.Exception（java.lang.RuntimeException除外）。

- **unchecked异常：**

**表示错误，程序的逻辑错误**。是RuntimeException的子类，比如IllegalArgumentException, NullPointerException和IllegalStateException。

**不需要在代码中显式地捕获unchecked异常做处理**。

继承自java.lang.RuntimeException（而java.lang.RuntimeException继承自java.lang.Exception）。

看张图：

![](http://image.augustrush8.com/images/javaexception.png){:.center}

### 异常封装实例

- **添加一个枚举LuoErrorCode.java如下：**

```javascript
package com.luo.errorcode;

public enum LuoErrorCode {

    NULL_OBJ("LUO001","对象为空"),
    ERROR_ADD_USER("LUO002","添加用户失败"),
    UNKNOWN_ERROR("LUO999","系统繁忙，请稍后再试....");

    private String value;
    private String desc;

    private LuoErrorCode(String value, String desc) {
        this.setValue(value);
        this.setDesc(desc);
    }

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }

    @Override
    public String toString() {
        return "[" + this.value + "]" + this.desc;
    }
}
```

这里重写了LuoErrorCode的toString方法，后面会看到为什么

- **创建一个异常类BusinessException.java，继承RuntimeException**

```java
package com.luo.exception;

public class BusinessException extends RuntimeException {

    private static final long serialVersionUID = 1L;

    public BusinessException(Object Obj) {
        super(Obj.toString());
    }

}
```

注意两点：

1. **其继承了RuntimeException，因为一般我们的业务异常都是运行时异常**
2. **这里的构造方法调用父方法super(Obj.toString());，这就是重写了LuoErrorCode的toString方法的原因了**

- **编写测试类ExceptionTest.java**

```java
package com.luo.test;

import com.luo.errorcode.LuoErrorCode;
import com.luo.exception.BusinessException;

public class ExceptionTest {

    public static void main(String args[]) {
        Object user = null;
        if(user == null){
            throw new BusinessException(LuoErrorCode.NULL_OBJ);
        }
    }
}
```

![](http://image.augustrush8.com/images/javaexception1.png){:.center}

### 总结

在我们实际项目里面，比如别人调用你接口，你可能需要先看他传过来的对象是不是空的，先判断如果传过来的对象为空会有友好的提示”[LUO001]对象为空”，不然后面的代码就会出现空指针异常了。

### 拓展

- **Error和Exception的联系**

继承结构：Error和Exception都是继承于Throwable，RuntimeException继承自Exception。

Error和RuntimeException及其子类称为未检查异常（Unchecked exception），其它异常成为受检查异常（Checked Exception）。

- **Error和Exception的区别**

Error类一般是指与虚拟机相关的问题，如系统崩溃，虚拟机错误，内存空间不足，方法调用栈溢出等。如java.lang.StackOverFlowError和Java.lang.OutOfMemoryError。对于这类错误，Java编译器不去检查他们。对于这类错误的导致的应用程序中断，仅靠程序本身无法恢复和预防，遇到这样的错误，建议让程序终止。

Exception类表示程序可以处理的异常，可以捕获且可能恢复。遇到这类异常，应该尽可能处理异常，使程序恢复运行，而不应该随意终止异常。

- **运行时异常和受检查的异常**

Exception又分为运行时异常（Runtime Exception）和受检查的异常(Checked Exception )。

RuntimeException：其特点是Java编译器不去检查它，也就是说，当程序中可能出现这类异常时，即使没有用try……catch捕获，也没有用throws抛出，还是会编译通过，如除数为零的ArithmeticException、错误的类型转换、数组越界访问和试图访问空指针等。处理RuntimeException的原则是：如果出现RuntimeException，那么一定是程序员的错误。

受检查的异常（IOException等）：这类异常如果没有try……catch也没有throws抛出，编译是通不过的。这类异常一般是外部错误，例如文件找不到、试图从文件尾后读取数据等，这并不是程序本身的错误，而是在应用环境中出现的外部错误。

- **throw 和 throws两个关键字有什么不同**

throw 是用来抛出任意异常的，你可以抛出任意 Throwable，包括自定义的异常类对象；throws总是出现在一个函数头中，用来标明该成员函数可能抛出的各种异常。如果方法抛出了异常，那么调用这个方法的时候就需要处理这个异常。

- **try-catch-finally-return执行顺序**

1、不管是否有异常产生，finally块中代码都会执行；
2、当try和catch中有return语句时，finally块仍然会执行；
3、finally是在return后面的表达式运算后执行的，所以函数返回值是在finally执行前确定的。无论finally中的代码怎么样，返回的值都不会改变，仍然是之前return语句中保存的值；

4、**finally中最好不要包含return，否则程序会提前退出，返回值不是try或catch中保存的返回值**。

举例：

```xml
情况1：try{} catch(){}finally{} return; 
按正常顺序执行。

情况2：try{ return; }catch(){} finally{} return; 
程序执行try块中return之前（包括return语句中的表达式运算）代码； 
再执行finally块，最后执行try中return; 
finally块后面的return语句不再执行。

情况3：try{ } catch(){return;} finally{} return; 
程序先执行try，如果遇到异常执行catch块， 
有异常： 
则执行catch中return之前（包括return语句中的表达式运算）代码，再执行finally语句中全部代码， 
最后执行catch块中return. finally块后面的return语句不再执行。 
无异常： 
执行完try再finally再执行最后的return语句.

情况4：try{ return; }catch(){} finally{return;} 
程序执行try块中return之前（包括return语句中的表达式运算）代码； 
再执行finally块，因为finally块中有return所以提前退出。

情况5：try{} catch(){return;}finally{return;} 
程序执行catch块中return之前（包括return语句中的表达式运算）代码； 
再执行finally块，因为finally块中有return所以提前退出。

情况6：try{ return;}catch(){return;} finally{return;} 
程序执行try块中return之前（包括return语句中的表达式运算）代码； 
有异常：执行catch块中return之前（包括return语句中的表达式运算）代码； 
则再执行finally块，因为finally块中有return所以提前退出。 
无异常：则再执行finally块，因为finally块中有return所以提前退出。

```

测试程序实例

```java
public class FinallyTest  
{
    public static void main(String[] args) {
        System.out.println(new FinallyTest().test());;
    }
    static int test()
    {
        int x = 1;
        try
        {
            x++;
            return x;
        }
        finally
        {
            ++x;
        }
    }
}
```

打印结果是2。

> 根据之前的分析，在try语句块中，在执行return语句时，要返回的结果已经准备好了，然后程序转到finally执行。在转去之前，try中先把要返回的结果存放到不同于x的局部变量中去，执行完finally之后，再从中取出返回结果，因此，即使finally中对变量x进行了改变，也不会影响返回结果。

