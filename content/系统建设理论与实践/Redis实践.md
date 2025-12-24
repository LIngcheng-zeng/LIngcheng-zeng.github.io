# 实践

## Redis key过期监听

### Redis key 过期机制

1. Redis key 在过期后不会被立即清除
2. Redis 定期检测过期key 时 ，或者过期Key被访问时，删除key,此时出发过期键的监听事件。
3. Redis 消息发布到channel后，若消费者没订阅，则消息丢失
4. Redis 的订阅模式只有广播模式，多个消费者会重复消费
5. Redis官方不推荐使用