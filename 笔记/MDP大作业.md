## MDP大作业文档

- 数据表设计

- product创建：

```sql
CREATE TABLE `product_wangkai116` (
  `id` int(11) unsigned NOT NULL COMMENT '''自增id''',
  `room_id` bigint(20) NOT NULL COMMENT '''房间id''',
  `room_type` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '''房型''',
  `bed_type` varchar(255) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '''床型''',
  `breakfast` varchar(45) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '‘是否提供早餐''',
  `gmt_created` datetime DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `stock` int(11) NOT NULL COMMENT '''库存''',
  PRIMARY KEY (`id`),
  UNIQUE KEY `room_id_UNIQUE` (`room_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;


```

- order创建

```sql

CREATE TABLE `order_wangkai116` (
  `id` bigint(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `order_id` bigint(11) unsigned NOT NULL COMMENT '订单id',
  `user_id` bigint(11) unsigned NOT NULL COMMENT '下单人id',
  `user_name` varchar(256) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '下单人姓名',
  `room_type` varchar(256) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '房型的名称',
  `bed_type` varchar(256) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '床型',
  `breakfast` varchar(256) COLLATE utf8mb4_unicode_ci NOT NULL COMMENT '早餐',
  `gmt_created` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_order_id` (`order_id`)
) ENGINE=InnoDB AUTO_INCREMENT=133 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='订单表';

```







## 问题回答

- **什么样的缓存更新策略能够保证缓存与数据库的一致性**

  - 首先是，当数据库更新的时候，是删除当前缓存数据，还是更新当前缓存数据。
  - 首先排除更新
    - 因为外因导致**先更新数据库的线程后更新缓存**，会产生脏数据
  - 然后考虑删除
    - 不要先删除缓存再删除数据库：当分别有写线程A和读线程B访问缓存的时候，A先把缓存删除，此时A因为某种原因阻塞，B再从缓存中找不到，然后b更新缓存，又将旧值插入到缓存中，此时A被唤醒，A将更新值插入数据库中。导致不一致。
    - 风险很低的策略：更新数据库再删除缓存，如果缓存删除的过程没出错，那么只有在对应数据缓存过时的情况下才可能产生不一致的现象。
      - 应对删除失败的情况，可以考虑在删除失败的时候，让缓存发送失败信号，线程接收到信号之后，继续进行缓存删除操作。
      - 应对缓存过时的，可以考虑采用 **延迟双删**的操作，在写线程写入数据库之后，再等待一段时间，尽量确保读线程完成操作之后，再进行一次缓存删除，从而保证一致性。
  - 总的来说，确保一致性可以考虑 先更新数据库再删除缓存+删除失败信号+延迟双删的组合

- **保持发消息方式不变，考虑到扣减信息失败，如何重新设计下单流程？**

  - 因为通常来说采用的都是异步发消息的模式，这种情况下，生产者感知不到消费者的操作是否成功。因此对于生产者而言，我们可以采用回调函数的方式，将一个异常写入send方法中，从而能够在失败的时候抛出异常，让接收方能够感知到

  ```java
  private class DemoProducerCallback implements Callback {  
      @Override
      public void onCompletion(RecordMetadata recordMetadata, Exception e) {
          if (e != null) {
              e.printStackTrace(); 
          }
      }
  }
  ProducerRecord<String, String> record = new ProducerRecord<>(...);
  producer.send(record, new DemoProducerCallback());
  ```

  

- 如果有多台server机器，如何生成订单号，保证不重复，且全局递增

  - 雪花算法
    - 不重复关键：时间戳作为id号的一部分，算法产生的是一个long型 64 比特位的值，第一位未使用。接下来是41位的毫秒单位的时间戳（可以表示69年不重复）
    - 高并发的关键：10位作为机器码，12位作为每台机器的统计一毫秒的计数量，因此提供了每**毫秒**2 ^ 22个ID序列号

- 消息发重复了如何处理？

  - 设定一个确认机制
  - 每个消息体根据自身的请求参数计算一个哈希值，消费者如果连续收到相同的哈希值，则返回一个确认信号给生产者

- 一台web机器，1s可以处理多少请求？怎么计算出来？

  - 可以参考TCP慢启动+快速恢复的策略不断对服务器调整请求数量，通过观察web服务器的状态来判断阈值，比如吞吐量。

  - 在web机器正常的情况下，吞吐量往往满足如下公式，即用户*请求处以时间这个关系，一旦请求数量增加，吞吐量不再满足曲线，那么说明已经到达瓶颈

    
  $$
    ThroughOut = \frac{U*R}{T}
    $$
    