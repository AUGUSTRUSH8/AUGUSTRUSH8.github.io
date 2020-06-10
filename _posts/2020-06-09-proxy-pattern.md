---
layout: post
title: '设计模式之代理模式'
tags: [code]
---

## 代理模式应用场景

- 租房中介
- 婚介
- 售票黄牛
- 经纪人
- **事务代理**
- **非侵入式日志监听**

## 定义

为其他对象提供一种代理，以控制对这个对象的访问。 代理对象在客服端和目标对象之间起到中介作用，代理模式属于结构型设计模式。 

## 代理模式使用目的

- 保护目标对象
- 增强目标对象

## 代理模式分类

- 静态代理
- 动态代理

## 静态代理

**业务背景：** 在分布式业务场景中，我们通常会对数据库进行分库分表 ，分库分表之后使用 Java 操作时，就可能需要配置多个数据源，我们通过设置数据源路由来动态切换数据源。 

创建 `Order` 订单实体 ：

```java
public class Order {
    private Object orderInfo;
    private Long createTime;
    private String id;

    public Object getOrderInfo() {
        return orderInfo;
    }

    public void setOrderInfo(Object orderInfo) {
        this.orderInfo = orderInfo;
    }

    public Long getCreateTime() {
        return createTime;
    }

    public void setCreateTime(Long createTime) {
        this.createTime = createTime;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }
}
```

`OrderDao` 持久层操作类 

```java
public class OrderDao {
    public int insert(Order order) {
        System.out.println("OrderDao 创建 Order 成功");
        return 1;
    }
}
```

`IOrderService` 接口 

```java
public interface IOrderService {
    int createOrder(Order order);
}
```

`OrderService` 实现类 

```java
public class OrderService implements IOrderService{
    private OrderDao orderDao;

    /**
     * 本来是注入的，现在为了使用方便，直接构造方法中初始化
     */
    public OrderService(){
        orderDao = new OrderDao();
    }

    @Override
    public int createOrder(Order order) {
        System.out.println("OrderService 调用 orderDao 创建订单");
        return orderDao.insert(order);
    }
}
```

接着使用`静态代理`完成功能：根据订单创建时间自动按年进行分库存储。根据开闭原则，原来的逻辑不予修改，直接通过代理对象完成。

创建数据源路由对象：`DynamicDataSourceEntry` 类  

```java
public class DynamicDataSourceEntry {
    //默认数据源
    public static final String DEFAULT_SOURCE = null;

    private static final ThreadLocal<String> local = new ThreadLocal<>();

    private DynamicDataSourceEntry() {

    }

    //清空数据源
    public static void clear() {
        local.remove();
    }

    //获取当前正在使用的数据源的名字
    public static String get() {
        return local.get();
    }

    //还原当前切面的数据源
    public static void restore() {
        local.set(DEFAULT_SOURCE);
    }

    //设置已知名字的数据源
    public static void set(String source) {
        local.set(source);
    }

    //根据年份动态设置数据源
    public static void set(int year) {
        local.set("DB_"+year);
    }
}
```

创建切换数据源的代理 `OrderServiceSaticProxy` 类 

```java
public class OrderServiceSaticProxy implements IOrderService{
    private SimpleDateFormat yearFormat = new SimpleDateFormat("yyyy");
    private IOrderService orderService;

    public OrderServiceSaticProxy(IOrderService orderService) {
        this.orderService=orderService;
    }
    @Override
    public int createOrder(Order order) {
        before();
        Long time = order.getCreateTime();
        Integer dbRouter=Integer.valueOf(yearFormat.format(new Date(time)));
        System.out.println("静态代理类自动分配到【DB_" + dbRouter + "】 数据源处理数据。 ");
        DynamicDataSourceEntry.set(dbRouter);
        orderService.createOrder(order);
        after();
        return 0;
    }
    private void before(){
        System.out.println("Proxy before method.");
    }
    private void after(){
        System.out.println("Proxy after method.");
    }
}
```

测试代码

```java
public class TestStaticProxy {
    public static void main(String[] args) {
        try {
            Order order = new Order();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
            Date date = sdf.parse("2020/06/09");
            order.setCreateTime(date.getTime());
            OrderServiceSaticProxy orderService = new OrderServiceSaticProxy(new OrderService());
            orderService.createOrder(order);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

运行结果

```java
Proxy before method.
静态代理类自动分配到【DB_2020】 数据源处理数据。 
OrderService 调用 orderDao 创建订单
OrderDao 创建 Order 成功
Proxy after method.
```

以上就是静态代理的预期结果，比较好理解。

## 动态代理

动态代理和静态对比基本思路是一致的，只不过动态代理功能更加强大，随着业务的扩展适应性更强。 还是看上面的数据源动态路由业务，创建动态代理的类 `OrderServiceDynamicProxy`

```java
public class OrderServiceDynamicProxy implements InvocationHandler {
    private SimpleDateFormat yearFormat = new SimpleDateFormat("yyyy");
    private Object target;

    public Object getInstance(Object target) {
        this.target=target;
        Class<?> clazz = target.getClass();
        return Proxy.newProxyInstance(clazz.getClassLoader(),clazz.getInterfaces(),this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        before(args[0]);
        Object object = method.invoke(target, args);
        after();
        return object;
    }
    private void before(Object target){
        try {
            System.out.println("Proxy before method.");
            Long time = (Long) target.getClass().getMethod("getCreateTime").invoke(target);
            Integer dbRouter = Integer.valueOf(yearFormat.format(new Date(time)));
            System.out.println("静态代理类自动分配到【DB_" + dbRouter + "】 数据源处理数据。 ");
            DynamicDataSourceEntry.set(dbRouter);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
    private void after(){
        System.out.println("Proxy after method.");
    }
}
```

测试代码

```java
public class DynamicProxyTest {
    public static void main(String[] args) {
        try {
            Order order = new Order();
            SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy/MM/dd");
            Date date = simpleDateFormat.parse("2020/06/09");
            order.setCreateTime(date.getTime());
            IOrderService orderService = (IOrderService) new OrderServiceDynamicProxy().getInstance(new OrderService());
            orderService.createOrder(order);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

输出结果

```java
Proxy before method.
静态代理类自动分配到【DB_2020】 数据源处理数据。 
OrderService 调用 orderDao 创建订单
OrderDao 创建 Order 成功
Proxy after method.
```

可以看到效果是一样的，但是，动态代理实现之后，我们不仅能实现 Order 的数据源动态路由，还可以实现其他任何类的数据源路由。

## 静态代理和动态的本质区别 

1、静态代理只能通过手动完成代理操作，如果被代理类增加新的方法，代理类需要同步新增，违背开闭原则。
2、动态代理采用在运行时动态生成代码的方式，取消了对被代理类的扩展限制，遵循开闭原则。
3、若动态代理要对目标类的增强逻辑扩展，结合策略模式，只需要新增策略类便可完成，无需修改代理类的代码。  

## 代理模式的优缺点 

使用代理模式具有以下几个优点：

1、代理模式能将代理对象与真实被调用的目标对象分离。

2、一定程度上降低了系统的耦合度，扩展性好。

3、可以起到保护目标对象的作用。

4、可以对目标对象的功能增强。

当然，代理模式也是有缺点的： 

1、代理模式会造成系统设计中类的数量增加。

2、在客户端和目标对象增加一个代理对象，会造成请求处理速度变慢。

3、增加了系统的复杂度。 

## Spring 中的代理选择原则 

1、当 Bean 有实现接口时，Spring 就会用 JDK 的动态代理

2、当 Bean 没有实现接口时，Spring 选择 CGLib。

3、Spring 可以通过配置强制使用 CGLib，只需在 Spring 的配置文件中加入如下代码： 

```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```





 