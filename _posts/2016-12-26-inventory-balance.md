---
title: 库存扣减
---

# 要求：
1. 不能有负数出现
2. 需要可以幂等

# 理论
1. 针对问题一，可以通过锁或者CAS操作实现（锁分为分布式锁、数据库锁）
2. 针对问题二，需要有一个业务唯一字段，对这个唯一字段只处理一次（订单ID、订单ID+业务ID）

# 方案：
1. 代码同步, 例如使用 synchronized ,lock 等同步方法
2. 使用分布式锁(zookeeper,redis等)
3. 不查询,直接更新  update table set surplus = (surplus - buyQuantity) where id = xx and (surplus - buyQuantity) > 0
4. 使用CAS, update t set surplus = 90 ,version = version+1 where id = x and version = oldVersion 
5. 使用数据库锁, select xx for update


第一种和第二种方案类似，但第一种只能单机，第二种可以分布式。但这两种方式需要考虑加锁的顺序和事务的隔离性。顺序如下：加锁->开始事务->结束事务->释放锁

第三种方案可以实现不会为负数的情况，但不能做返还库存的操作

方案四: 推荐使用.但是如果数据竞争激烈,则自动重试次数会急剧上升,需要注意.
方案五: 推荐使用.最简单的方案,但是如果事务过大,会有性能问题.操作不当,会有死锁问题

对于select for update,需要注意的有2点.
1) 统一入口:所有库存操作都需要统一使用 select for update ,这样才会阻塞, 如果另外一个方法还是普通的select, 是不会被阻塞的
2) 加锁顺序:如果有多个锁,那么加锁顺序要一致,否则会出现死锁.