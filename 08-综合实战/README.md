# 🎯 综合实战

> 完整项目示例，整合所有技术

---

## 项目介绍

**项目名称：** 电商平台

**技术栈：**
- **后端框架：** Spring Boot
- **RPC 框架：** Dubbo
- **注册中心：** Nacos
- **缓存：** Redis
- **消息队列：** Kafka
- **容器化：** Docker
- **负载均衡：** Nginx

**架构图：**

```
┌──────────┐
│  Client  │
│  客户端  │
└────┬─────┘
     │
┌────┴─────┐
│  Nginx   │
│  负载均衡 │
└────┬─────┘
     │
┌────┴─────────────────┐
│  Spring Boot 应用    │
│  ├─ Dubbo 服务       │
│  ├─ Nacos 注册       │
│  ├─ Redis 缓存       │
│  └─ Kafka 消息       │
└──────────────────────┘
     │
┌────┴─────┐
│  MySQL   │
│  数据库  │
└──────────┘
```

---

## 项目结构

```
ecommerce-platform/
├── ecommerce-api/           # API 接口层
├── ecommerce-service/       # 服务层
├── ecommerce-dao/          # 数据访问层
├── ecommerce-model/        # 数据模型
└── pom.xml                 # Maven 配置
```

---

## 环境搭建

### 1. 启动基础设施

```bash
# Nacos
docker run -d --name nacos \
  -e MODE=standalone \
  -p 8848:8848 \
  nacos/nacos-server:2.0.3

# Redis
docker run -d --name redis \
  -p 6379:6379 \
  redis:latest

# Kafka
docker run -d --name kafka \
  -e KAFKA_BROKER_ID=0 \
  -e KAFKA_ZOOKEEPER_CONNECT=192.168.1.100:2181 \
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.1.100:9092 \
  -p 9092:9092 \
  wurstmeister/kafka:2.12-2.2.1

# MySQL
docker run -d --name mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -p 3306:3306 \
  mysql:5.7
```

### 2. 配置 Nacos

**创建配置：**

- Data ID: ecommerce-service.yaml
- Group: DEFAULT_GROUP
- 内容：

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ecommerce
    username: root
    password: 123456
  
  redis:
    host: localhost
    port: 6379
  
  kafka:
    bootstrap-servers: localhost:9092

dubbo:
  application:
    name: ecommerce-service
  registry:
    address: nacos://localhost:8848
  protocol:
    name: dubbo
    port: 20880
```

---

## 核心代码

### 1. 服务提供者

```java
// 服务接口
public interface OrderService {
    Order createOrder(OrderRequest request);
    Order getOrderById(Long id);
}

// 服务实现
@Service
public class OrderServiceImpl implements OrderService {
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    @Override
    public Order createOrder(OrderRequest request) {
        // 1. 创建订单
        Order order = new Order();
        BeanUtils.copyProperties(request, order);
        orderMapper.insert(order);
        
        // 2. 缓存订单
        String key = "order:" + order.getId();
        redisTemplate.opsForValue().set(key, order, 30, TimeUnit.MINUTES);
        
        // 3. 发送消息
        kafkaTemplate.send("order-topic", order);
        
        return order;
    }
    
    @Override
    public Order getOrderById(Long id) {
        // 1. 先查缓存
        String key = "order:" + id;
        Order order = (Order) redisTemplate.opsForValue().get(key);
        if (order != null) {
            return order;
        }
        
        // 2. 查数据库
        order = orderMapper.selectById(id);
        
        // 3. 写入缓存
        redisTemplate.opsForValue().set(key, order, 30, TimeUnit.MINUTES);
        
        return order;
    }
}

// 服务暴露
@Configuration
public class DubboConfig {
    
    @Bean
    public ServiceConfig<OrderService> orderServiceConfig(OrderService orderService) {
        ServiceConfig<OrderService> service = new ServiceConfig<>();
        service.setInterface(OrderService.class);
        service.setRef(orderService);
        return service;
    }
}
```

### 2. 服务消费者

```java
@Component
public class OrderController {
    
    @Reference
    private OrderService orderService;
    
    @PostMapping("/order")
    public Result<Order> createOrder(@RequestBody OrderRequest request) {
        Order order = orderService.createOrder(request);
        return Result.success(order);
    }
    
    @GetMapping("/order/{id}")
    public Result<Order> getOrder(@PathVariable Long id) {
        Order order = orderService.getOrderById(id);
        return Result.success(order);
    }
}
```

### 3. Docker 部署

```dockerfile
# Dockerfile
FROM openjdk:8-jdk-alpine

WORKDIR /app

COPY target/ecommerce-service.jar app.jar

EXPOSE 8080 20880

ENTRYPOINT ["java", "-jar", "app.jar"]
```

```yaml
# docker-compose.yml
version: '3'
services:
  ecommerce-service:
    build: .
    ports:
      - "8080:8080"
      - "20880:20880"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
    networks:
      - my-network

networks:
  my-network:
    driver: bridge
```

### 4. Nginx 配置

```nginx
upstream ecommerce {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}

server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://ecommerce;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

## 部署流程

### 1. 本地开发

```bash
# 克隆项目
git clone https://github.com/xxuan66/ecommerce-platform.git

# 安装依赖
mvn clean install

# 启动服务
mvn spring-boot:run
```

### 2. 构建镜像

```bash
# 打包
mvn clean package

# 构建 Docker 镜像
docker build -t ecommerce-service:1.0 .
```

### 3. 部署到服务器

```bash
# 推送镜像
docker push ecommerce-service:1.0

# 启动容器
docker run -d \
  -p 8080:8080 \
  -p 20880:20880 \
  -e SPRING_PROFILES_ACTIVE=prod \
  ecommerce-service:1.0
```

### 4. 配置 Nginx

```bash
# 编辑配置
vim /etc/nginx/conf.d/ecommerce.conf

# 重新加载
nginx -s reload
```

---

## 监控与排查

### 1. SkyWalking 链路追踪

访问：http://localhost:8080/skywalking

查看：
- 服务拓扑图
- 调用链路
- 性能指标

### 2. ELK 日志系统

访问：http://localhost:5601

查看：
- 应用日志
- 错误日志
- 访问日志

### 3. Nacos 控制台

访问：http://localhost:8848/nacos

查看：
- 服务列表
- 配置信息
- 集群状态

---

## 常见问题

### 问题 1：服务注册失败

**解决方案：**
1. 检查 Nacos 是否启动
2. 检查配置是否正确
3. 检查网络是否通畅

### 问题 2：缓存穿透

**解决方案：**
1. 缓存空对象
2. 布隆过滤器
3. 接口层校验

### 问题 3：消息丢失

**解决方案：**
1. 生产者 acks=all
2. 消费者手动提交
3. 开启副本

---

## 总结

**本项目整合了：**
- ✅ Dubbo RPC 调用
- ✅ Nacos 服务注册与配置
- ✅ Redis 分布式缓存
- ✅ Kafka 消息队列
- ✅ Docker 容器化部署
- ✅ Nginx 负载均衡

**学完后你可以：**
- ✅ 理解大厂技术架构
- ✅ 掌握核心中间件使用
- ✅ 具备微服务开发能力
- ✅ 能够独立部署项目

---

**最后更新：** 2026-03-25  
**状态：** ✅ 已完成
