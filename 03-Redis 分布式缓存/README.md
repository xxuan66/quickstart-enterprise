# 💾 Redis 分布式缓存

> 10 分钟上手，掌握大厂缓存技术

---

## 1. 为什么用 Redis？

**解决了什么问题：**
- 数据库查询慢？
- 高并发扛不住？
- 需要分布式锁？

**大厂为什么用：**
- ✅ 性能极高：10 万 + QPS
- ✅ 功能丰富：缓存、锁、消息队列
- ✅ 持久化：RDB + AOF
- ✅ 高可用：哨兵、集群模式

**应用场景：**
- 数据缓存
- 分布式锁
- 会话管理
- 排行榜
- 消息队列

---

## 2. 快速开始（10 分钟上手）

### 环境要求

- Docker（推荐）或本地安装

### 第一个 Redis 示例

**步骤 1：启动 Redis**

```bash
# 使用 Docker 启动（推荐）
docker run -d -p 6379:6379 --name redis redis:latest

# 或本地安装启动
redis-server
```

**步骤 2：连接 Redis**

```bash
# 命令行连接
redis-cli

# 测试
127.0.0.1:6379> ping
PONG
```

**步骤 3：添加 Maven 依赖**

```xml
<dependencies>
    <!-- Spring Data Redis -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    
    <!-- Redis 连接池 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-pool2</artifactId>
    </dependency>
</dependencies>
```

**步骤 4：配置 Redis**

```yaml
# application.yml
spring:
  redis:
    host: localhost
    port: 6379
    password:           # 密码（如果有）
    database: 0
    lettuce:
      pool:
        max-active: 8   # 最大连接数
        max-idle: 8     # 最大空闲连接
        min-idle: 0     # 最小空闲连接
```

**步骤 5：使用 RedisTemplate**

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

@GetMapping("/set")
public String set() {
    redisTemplate.opsForValue().set("key", "value");
    return "OK";
}

@GetMapping("/get")
public Object get() {
    return redisTemplate.opsForValue().get("key");
}
```

**步骤 6：使用 StringRedisTemplate**

```java
@Autowired
private StringRedisTemplate stringRedisTemplate;

@GetMapping("/setStr")
public String setStr() {
    stringRedisTemplate.opsForValue().set("name", "Redis");
    return "OK";
}

@GetMapping("/getStr")
public String getStr() {
    return stringRedisTemplate.opsForValue().get("name");
}
```

**步骤 7：测试**

访问接口，看到返回结果。

---

## 3. 核心概念

### Redis 数据结构

| 类型 | 说明 | 使用场景 |
|------|------|---------|
| **String** | 字符串 | 缓存、计数器 |
| **List** | 列表 | 消息队列、最新列表 |
| **Hash** | 哈希 | 对象存储 |
| **Set** | 集合 | 去重、好友关系 |
| **ZSet** | 有序集合 | 排行榜 |

### 架构图

```
┌─────────────┐     ┌─────────────┐
│   应用      │────▶│    Redis    │
│ Application │     │   Cache     │
└─────────────┘     └─────────────┘
```

### 持久化方式

**RDB（快照）：**
- 定期保存数据快照
- 恢复快，文件小
- 可能丢失最后一次快照后的数据

**AOF（追加日志）：**
- 记录每次写操作
- 数据更安全
- 文件大，恢复慢

---

## 4. 常用配置

### 必会命令

**String 操作：**
```bash
SET key value              # 设置值
GET key                    # 获取值
SETNX key value            # 不存在则设置（分布式锁）
SETEX key seconds value    # 设置过期时间
INCR key                   # 自增 1
DECR key                   # 自减 1
```

**Hash 操作：**
```bash
HSET key field value       # 设置字段
HGET key field             # 获取字段
HGETALL key                # 获取所有字段
HDEL key field             # 删除字段
```

**List 操作：**
```bash
LPUSH key value            # 左边插入
RPUSH key value            # 右边插入
LPOP key                   # 左边弹出
RPOP key                   # 右边弹出
LRANGE key start stop      # 获取范围
```

**Set 操作：**
```bash
SADD key member            # 添加成员
SMEMBERS key               # 获取所有成员
SISMEMBER key member       # 判断是否成员
SREM key member            # 删除成员
```

**ZSet 操作：**
```bash
ZADD key score member      # 添加成员
ZRANGE key start stop      # 获取排名
ZREVRANGE key start stop   # 倒序获取
ZREM key member            # 删除成员
```

### 过期时间配置

```java
// 设置过期时间
redisTemplate.expire("key", 300, TimeUnit.SECONDS);

// 设置时带过期时间
redisTemplate.opsForValue().set("key", "value", 5, TimeUnit.MINUTES);
```

---

## 5. 实战场景

### 场景 1：缓存热点数据

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

public User getUserById(Long id) {
    String key = "user:" + id;
    
    // 1. 先查缓存
    User user = (User) redisTemplate.opsForValue().get(key);
    if (user != null) {
        return user;
    }
    
    // 2. 缓存没有，查数据库
    user = userMapper.selectById(id);
    
    // 3. 写入缓存，5 分钟过期
    redisTemplate.opsForValue().set(key, user, 5, TimeUnit.MINUTES);
    
    return user;
}
```

### 场景 2：分布式锁

```java
public void doWithLock(String key) {
    String lockKey = "lock:" + key;
    
    // 尝试获取锁
    Boolean success = redisTemplate.opsForValue()
        .setIfAbsent(lockKey, "locked", 10, TimeUnit.SECONDS);
    
    if (Boolean.TRUE.equals(success)) {
        try {
            // 执行业务逻辑
            doBusiness();
        } finally {
            // 释放锁
            redisTemplate.delete(lockKey);
        }
    } else {
        // 获取锁失败
        throw new RuntimeException("获取锁失败");
    }
}
```

### 场景 3：计数器

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

// 文章阅读量
public void incrementViewCount(Long articleId) {
    String key = "article:view:" + articleId;
    redisTemplate.opsForValue().increment(key);
}

// 获取阅读量
public Long getViewCount(Long articleId) {
    String key = "article:view:" + articleId;
    return (Long) redisTemplate.opsForValue().get(key);
}
```

### 场景 4：排行榜

```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

// 添加分数
public void addScore(String userId, double score) {
    redisTemplate.opsForZSet().add("leaderboard", userId, score);
}

// 获取前 10 名
public Set<Object> getTop10() {
    return redisTemplate.opsForZSet().reverseRange("leaderboard", 0, 9);
}

// 获取用户排名
public Long getRank(String userId) {
    return redisTemplate.opsForZSet().reverseRank("leaderboard", userId);
}
```

---

## 6. 常见问题

### 问题 1：缓存穿透

**现象：** 查询不存在的数据，缓存不命中，请求直达数据库

**解决方案：**
1. 缓存空对象
2. 布隆过滤器
3. 接口层校验

### 问题 2：缓存击穿

**现象：** 热点 key 过期，大量请求直达数据库

**解决方案：**
1. 设置热点数据永不过期
2. 加分布式锁
3. 二级缓存

### 问题 3：缓存雪崩

**现象：** 大量 key 同时过期，请求直达数据库

**解决方案：**
1. 过期时间加随机值
2. 热点数据永不过期
3. 限流降级

### 问题 4：分布式锁失效

**现象：** 锁自动过期，业务未执行完

**解决方案：**
1. 使用看门狗机制（Redisson）
2. 设置合理过期时间
3. 使用 RedLock

---

## 7. 面试题

### Q1：Redis 为什么这么快？

**参考答案：**
- 纯内存操作
- 单线程，无锁竞争
- IO 多路复用
- 高效的数据结构

### Q2：Redis 的持久化方式有哪些？

**参考答案：**
- **RDB：** 定期快照，恢复快，可能丢数据
- **AOF：** 记录写操作，数据安全，文件大
- **混合持久化：** RDB + AOF，兼顾两者优点

### Q3：Redis 有哪些数据类型？

**参考答案：**
- String：字符串
- List：列表
- Hash：哈希
- Set：集合
- ZSet：有序集合

### Q4：Redis 如何实现分布式锁？

**参考答案：**
使用 SETNX 命令：
```bash
SETNX lock_key unique_value
EXPIRE lock_key 10
```

### Q5：缓存穿透、击穿、雪崩有什么区别？

**参考答案：**
- **穿透：** 查询不存在的数据
- **击穿：** 热点 key 过期
- **雪崩：** 大量 key 同时过期

---

**下一篇：** [Redis 快速入门](./Redis 快速入门.md)

---

**最后更新：** 2026-03-25  
**状态：** ✅ 已完成
