# 📨 Kafka 消息队列

> 10 分钟上手，掌握大厂消息队列技术

---

## 1. 为什么用 Kafka？

**解决了什么问题：**
- 系统耦合严重？
- 流量峰值扛不住？
- 需要异步处理？

**大厂为什么用：**
- ✅ 高吞吐：百万级消息/秒
- ✅ 可扩展：分布式架构
- ✅ 持久化：消息持久存储
- ✅ 生态完善：流处理、日志收集

**应用场景：**
- 异步解耦
- 流量削峰
- 日志收集
- 数据管道
- 流处理

---

## 2. 快速开始（10 分钟上手）

### 环境要求

- Docker（推荐）
- JDK 8+

### 第一个 Kafka 示例

**步骤 1：启动 Kafka（使用 Docker）**

```bash
# 启动 ZooKeeper
docker run -d --name zookeeper \
  -e ZOOKEEPER_CLIENT_PORT=2181 \
  zookeeper:3.4.9

# 启动 Kafka
docker run -d --name kafka \
  -e KAFKA_BROKER_ID=0 \
  -e KAFKA_ZOOKEEPER_CONNECT=192.168.1.100:2181 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.100:9092 \
  -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
  -p 9092:9092 \
  wurstmeister/kafka:2.12-2.2.1
```

**步骤 2：添加 Maven 依赖**

```xml
<dependencies>
    <!-- Spring Kafka -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
    </dependency>
</dependencies>
```

**步骤 3：配置 Kafka**

```yaml
# application.yml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: test-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      auto-offset-reset: earliest
```

**步骤 4：发送消息**

```java
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

@GetMapping("/send")
public String send() {
    kafkaTemplate.send("test-topic", "Hello Kafka");
    return "OK";
}
```

**步骤 5：接收消息**

```java
@KafkaListener(topics = "test-topic", groupId = "test-group")
public void listen(String message) {
    System.out.println("收到消息：" + message);
}
```

**步骤 6：测试**

访问发送接口，在控制台看到收到的消息。

---

## 3. 核心概念

### Kafka 架构图

```
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Producer │────▶│  Kafka   │────▶│ Consumer │
│ 生产者   │     │  Broker  │     │ 消费者   │
└──────────┘     └──────────┘     └──────────┘
                      │
                 ┌────┴────┐
                 │ZooKeeper│
                 └─────────┘
```

### 核心组件

| 组件 | 作用 |
|------|------|
| **Producer** | 消息生产者 |
| **Consumer** | 消息消费者 |
| **Broker** | Kafka 服务器 |
| **Topic** | 消息主题 |
| **Partition** | 分区 |
| **Consumer Group** | 消费者组 |
| **ZooKeeper** | 协调服务 |

### 工作流程

1. 生产者发送消息到 Topic
2. Kafka 将消息写入 Partition
3. 消费者从 Partition 拉取消息
4. 消费者提交 Offset（消费进度）

---

## 4. 常用配置

### 生产者配置

```yaml
spring:
  kafka:
    producer:
      bootstrap-servers: localhost:9092
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      acks: all              # 确认机制：0/1/all
      retries: 3             # 重试次数
      batch-size: 16384      # 批量大小
      buffer-memory: 33554432 # 缓冲区大小
```

### 消费者配置

```yaml
spring:
  kafka:
    consumer:
      bootstrap-servers: localhost:9092
      group-id: test-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      auto-offset-reset: earliest  # earliest/latest/none
      enable-auto-commit: true     # 自动提交 offset
      auto-commit-interval: 1000   # 提交间隔（毫秒）
```

### 确认机制

| acks | 说明 | 可靠性 | 性能 |
|------|------|--------|------|
| **0** | 不等待确认 | 低 | 高 |
| **1** | Leader 确认 | 中 | 中 |
| **all** | 所有副本确认 | 高 | 低 |

---

## 5. 实战场景

### 场景 1：异步解耦

```java
// 订单服务
@Service
public class OrderService {
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    public void createOrder(Order order) {
        // 1. 创建订单
        orderMapper.insert(order);
        
        // 2. 发送消息，通知其他系统
        kafkaTemplate.send("order-topic", order);
        
        // 3. 立即返回，不等待其他系统处理
    }
}

// 库存服务
@Component
public class InventoryListener {
    
    @KafkaListener(topics = "order-topic", groupId = "inventory-group")
    public void listen(Order order) {
        // 扣减库存
        inventoryMapper.decrease(order.getProductId());
    }
}

// 物流服务
@Component
public class ShippingListener {
    
    @KafkaListener(topics = "order-topic", groupId = "shipping-group")
    public void listen(Order order) {
        // 创建物流单
        shippingMapper.create(order);
    }
}
```

### 场景 2：流量削峰

```java
// 秒杀活动
@Service
public class SeckillService {
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    public void seckill(SeckillRequest request) {
        // 1. 快速验证
        if (!validate(request)) {
            throw new RuntimeException("验证失败");
        }
        
        // 2. 发送到消息队列（削峰）
        kafkaTemplate.send("seckill-topic", request);
        
        // 3. 立即返回，排队处理
        Result.success("排队中，请稍后查看结果");
    }
}

// 后台处理
@Component
public class SeckillListener {
    
    @KafkaListener(topics = "seckill-topic", groupId = "seckill-group")
    public void listen(SeckillRequest request) {
        // 后台慢慢处理
        processSeckill(request);
    }
}
```

### 场景 3：日志收集

```java
// 日志服务
@Service
public class LogService {
    
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;
    
    public void log(String level, String message) {
        // 发送到日志主题
        kafkaTemplate.send("log-topic", level, message);
    }
}

// 日志处理
@Component
public class LogListener {
    
    @KafkaListener(topics = "log-topic", groupId = "log-group")
    public void listen(ConsumerRecord<String, String> record) {
        String level = record.key();
        String message = record.value();
        
        // 存储到 Elasticsearch
        elasticsearchService.save(level, message);
    }
}
```

---

## 6. 常见问题

### 问题 1：消息重复消费

**现象：** 同一条消息被消费多次

**解决方案：**
1. 消费者幂等处理
2. 数据库唯一键
3. Redis 去重

### 问题 2：消息丢失

**现象：** 消息发送成功，但消费者没收到

**解决方案：**
1. 生产者设置 acks=all
2. 消费者手动提交 offset
3. 开启 Kafka 副本

### 问题 3：消息积压

**现象：** 消费者处理不过来，消息大量积压

**解决方案：**
1. 增加消费者数量
2. 优化消费逻辑
3. 增加 Partition 数量

### 问题 4：顺序消费

**现象：** 需要保证消息顺序

**解决方案：**
1. 发送到同一个 Partition
2. 单线程消费
3. 使用消息队列的顺序消息功能

---

## 7. 面试题

### Q1：Kafka 为什么吞吐量高？

**参考答案：**
- 顺序读写磁盘
- 零拷贝技术
- 批量发送
- 数据压缩

### Q2：Kafka 如何保证消息不丢失？

**参考答案：**
- **生产者：** acks=all，重试机制
- **Broker：** 多副本，ISR 机制
- **消费者：** 手动提交 offset

### Q3：Kafka 如何保证消息顺序？

**参考答案：**
- 发送到同一个 Partition
- 单线程消费
- 分区内有序

### Q4：Kafka 的 Partition 和 Consumer 有什么关系？

**参考答案：**
- 一个 Partition 只能被一个 Consumer 消费
- 一个 Consumer 可以消费多个 Partition
- Consumer 数量 <= Partition 数量

### Q5：Kafka 如何保证消息不重复？

**参考答案：**
- 消费者幂等处理
- 数据库唯一键
- Redis 去重

---

**下一篇：** [Kafka 快速入门](./Kafka 快速入门.md)

---

**最后更新：** 2026-03-25  
**状态：** ✅ 已完成
