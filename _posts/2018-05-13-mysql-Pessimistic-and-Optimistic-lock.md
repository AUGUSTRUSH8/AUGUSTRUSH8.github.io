---
layout: post
title: 'Mysql乐观锁与悲观锁'
tags: [read]
---

### 悲观锁

悲观锁（Pessimistic Lock），顾名思义，就是很悲观，**每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁**，这样别人想拿这个数据就会block直到它拿到锁。

悲观锁：假定会发生并发冲突，屏蔽一切可能违反数据完整性的操作。

Java synchronized 就属于悲观锁的一种实现，每次线程要修改数据时都先获得锁，保证同一时刻只有一个线程能操作数据，**其他线程则会被block**。

### 乐观锁

乐观锁（Optimistic Lock），顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在提交更新的时候会判断一下在此期间别人有没有去更新这个数据。**乐观锁适用于读多写少的应用场景**，这样可以提高吞吐量。

乐观锁：假设不会发生并发冲突，只在提交操作时检查是否违反数据完整性。

乐观锁一般来说有以下2种方式：

1. 使用数据版本（Version）记录机制实现，这是乐观锁最常用的一种实现方式。何谓数据版本？即为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version” 字段来实现。当读取数据时，将version字段的值一同读出，数据每更新一次，对此version值加一。当我们提交更新的时候，判断数据库表对应记录的当前版本信息与第一次取出来的version值进行比对，如果数据库表当前版本号与第一次取出来的version值相等，则予以更新，否则认为是过期数据。
2. 使用时间戳（timestamp）。乐观锁定的第二种实现方式和第一种差不多，同样是在需要乐观锁控制的table中增加一个字段，名称无所谓，字段类型使用时间戳（timestamp）, 和上面的version类似，也是在更新提交的时候检查当前数据库中数据的时间戳和自己更新前取到的时间戳进行对比，如果一致则OK，否则就是版本冲突。

Java JUC中的atomic包就是乐观锁的一种实现，AtomicInteger 通过**CAS**（Compare And Set）操作实现线程安全的自增。

### MySQL隐式和显示锁定

MySQL InnoDB采用的是两阶段锁定协议（two-phase locking protocol）。在事务执行过程中，随时都可以执行锁定，锁只有在执行 COMMIT或者ROLLBACK的时候才会释放，并且所有的锁是在同一时刻被释放。前面描述的锁定都是隐式锁定，InnoDB会根据事务隔离级别在需要的时候自动加锁。

另外，InnoDB也支持通过特定的语句进行显示锁定，这些语句不属于SQL规范：

- SELECT ... LOCK IN SHARE MODE
- SELECT ... FOR UPDATE

### 案例分析

通过一个具体案例来进行分析：考虑电商系统中的下单流程，商品的库存量是固定的，如何保证商品数量不超卖？ 其实需要保证数据一致性：某个人点击秒杀后系统中查出来的库存量和实际扣减库存时库存量的一致性就可以。

假设，MySQL数据库中商品库存表tb_product_stock 结构定义如下：

```sql
CREATE TABLE `tb_product_stock` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增ID',
  `product_id` bigint(32) NOT NULL COMMENT '商品ID',
  `number` INT(8) NOT NULL DEFAULT 0 COMMENT '库存数量',
  `create_time` DATETIME NOT NULL COMMENT '创建时间',
  `modify_time` DATETIME NOT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `index_pid` (`product_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='商品库存表';
```

对应的POJO类：

```java
class ProductStock {
    private Long productId; //商品id
    private Integer number; //库存量

    public Long getProductId() {
        return productId;
    }

    public void setProductId(Long productId) {
        this.productId = productId;
    }

    public Integer getNumber() {
        return number;
    }

    public void setNumber(Integer number) {
        this.number = number;
    }
}
```

不考虑并发的情况下，更新库存代码如下：

```java
 /**
     * 更新库存(不考虑并发)
     * @param productId
     * @return
     */
    public boolean updateStockRaw(Long productId){
        ProductStock product = query("SELECT * FROM tb_product_stock WHERE product_id=#{productId}", productId);
        if (product.getNumber() > 0) {
            int updateCnt = update("UPDATE tb_product_stock SET number=number-1 WHERE product_id=#{productId}", productId);
            if(updateCnt > 0){    //更新库存成功
                return true;
            }
        }
        return false;
    }
```

**多线程并发情况下，会存在超卖的可能**。

### 悲观锁

```java
/**
     * 更新库存(使用悲观锁)
     * @param productId
     * @return
     */
    public boolean updateStock(Long productId){
        //先锁定商品库存记录
        ProductStock product = query("SELECT * FROM tb_product_stock WHERE product_id=#{productId} FOR UPDATE", productId);
        if (product.getNumber() > 0) {
            int updateCnt = update("UPDATE tb_product_stock SET number=number-1 WHERE product_id=#{productId}", productId);
            if(updateCnt > 0){    //更新库存成功
                return true;
            }
        }
        return false;
    }
```

### 乐观锁

```sql
/**
     * 下单减库存
     * @param productId
     * @return
     */
    public boolean updateStock(Long productId){
        int updateCnt = 0;
        while (updateCnt == 0) {
            ProductStock product = query("SELECT * FROM tb_product_stock WHERE product_id=#{productId}", productId);
            if (product.getNumber() > 0) {
                updateCnt = update("UPDATE tb_product_stock SET number=number-1 WHERE product_id=#{productId} AND number=#{number}", productId, product.getNumber());
                if(updateCnt > 0){    //更新库存成功
                    return true;
                }
            } else {    //卖完啦
                return false;
            }
        }
        return false;
    }
```

使用乐观锁更新库存的时候不加锁，当提交更新时需要判断数据是否已经被修改（AND number=#{number}），只有在 number等于上一次查询到的number时 才提交更新。

**注意**：UPDATE 语句的WHERE 条件字句上需要建索引

### 乐观锁与悲观锁的区别

乐观锁的思路一般是表中增加版本字段，更新时where语句中增加版本的判断，算是一种CAS（Compare And Swep）操作，商品库存场景中number起到了版本控制（相当于version）的作用（ AND number=#{number}）。

悲观锁之所以是悲观，在于他认为本次操作会发生并发冲突，所以一开始就对商品加上锁（SELECT ... FOR UPDATE），然后就可以安心的做判断和更新，因为这时候不会有别人更新这条商品库存。

### 总结

这里我们通过 MySQL 乐观锁与悲观锁 解决并发更新库存的问题，当然还有其它解决方案，例如使用 **分布式锁**。目前常见分布式锁实现有两种：基于Redis和基于Zookeeper，基于这两种 业界也有开源的解决方案，例如 [Redisson Distributed locks ](https://link.jianshu.com?t=https://github.com/redisson/redisson/wiki/8.-distributed-locks-and-synchronizers)、 [Apache Curator Shared Lock ](https://link.jianshu.com?t=http://curator.apache.org/curator-recipes/shared-lock.html)，这里就不细说，网上Google 一下就有很多资料。