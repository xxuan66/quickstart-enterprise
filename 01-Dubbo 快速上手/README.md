# 📡 Dubbo 快速上手

> 10 分钟上手，掌握大厂 RPC 框架

---

## 1. 为什么用 Dubbo？

**解决了什么问题：**
- 服务间调用太复杂？
- HTTP 性能不够？
- 需要服务治理？

**大厂为什么用：**
- ✅ 高性能：基于 Netty，性能卓越
- ✅ 服务治理：负载均衡、容错、降级
- ✅ 生态完善：Spring Cloud Alibaba 集成
- ✅ 阿里出品：经过双 11 考验

**应用场景：**
- 微服务架构
- 服务间调用
- 高性能 RPC 场景

---

## 2. 快速开始（10 分钟上手）

### 环境要求

- JDK 8+
- Maven 3.6+
- ZooKeeper（服务注册中心）

### 第一个 Dubbo 接口

**步骤 1：创建项目**

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=dubbo-demo -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

**步骤 2：添加依赖**

```xml
<dependencies>
    <!-- Dubbo 依赖 -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
        <version>3.0.7</version>
    </dependency>
    
    <!-- ZooKeeper 客户端 -->
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
        <version>5.1.0</version>
    </dependency>
</dependencies>
```

**步骤 3：定义接口**

```java
// 定义服务接口
public interface DemoService {
    String sayHello(String name);
}
```

**步骤 4：实现接口（服务提供者）**

```java
// 服务实现
public class DemoServiceImpl implements DemoService {
    @Override
    public String sayHello(String name) {
        return "Hello, " + name;
    }
}
```

**步骤 5：配置服务提供者**

```java
// 服务提供者启动类
public class ProviderBootstrap {
    public static void main(String[] args) throws Exception {
        // 配置应用
        ApplicationConfig application = new ApplicationConfig("dubbo-demo-provider");
        
        // 配置注册中心
        RegistryConfig registry = new RegistryConfig("zookeeper://127.0.0.1:2181");
        
        // 配置协议
        ProtocolConfig protocol = new ProtocolConfig("dubbo", 20880);
        
        // 配置服务
        ServiceConfig<DemoService> service = new ServiceConfig<>();
        service.setApplication(application);
        service.setRegistry(registry);
        service.setProtocol(protocol);
        service.setInterface(DemoService.class);
        service.setRef(new DemoServiceImpl());
        
        // 暴露服务
        service.export();
        
        System.out.println("服务提供者启动成功...");
        
        // 保持运行
        System.in.read();
    }
}
```

**步骤 6：配置服务消费者**

```java
// 服务消费者启动类
public class ConsumerBootstrap {
    public static void main(String[] args) throws Exception {
        // 配置应用
        ApplicationConfig application = new ApplicationConfig("dubbo-demo-consumer");
        
        // 配置注册中心
        RegistryConfig registry = new RegistryConfig("zookeeper://127.0.0.1:2181");
        
        // 配置服务引用
        ReferenceConfig<DemoService> reference = new ReferenceConfig<>();
        reference.setApplication(application);
        reference.setRegistry(registry);
        reference.setInterface(DemoService.class);
        
        // 获取服务代理
        DemoService demoService = reference.get();
        
        // 调用服务
        String result = demoService.sayHello("Dubbo");
        System.out.println(result);
    }
}
```

**步骤 7：运行**

1. 启动 ZooKeeper
2. 运行 `ProviderBootstrap`
3. 运行 `ConsumerBootstrap`
4. 看到输出：`Hello, Dubbo`

---

## 3. 核心概念

### Dubbo 架构图

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  消费者     │────▶│  注册中心   │◀────│  提供者     │
│  Consumer   │     │ ZooKeeper   │     │  Provider   │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 核心组件

| 组件 | 作用 |
|------|------|
| **Provider** | 服务提供者 |
| **Consumer** | 服务消费者 |
| **Registry** | 服务注册中心（ZooKeeper/Nacos） |
| **Monitor** | 监控中心 |
| **Container** | 服务运行容器 |

### 工作流程

1. 服务提供者启动，注册服务到注册中心
2. 服务消费者启动，从注册中心订阅服务
3. 注册中心返回服务提供者地址列表
4. 消费者通过负载均衡选择提供者调用
5. 监控中心记录调用次数和时间

---

## 4. 常用配置

### 必会配置项

**服务提供者配置：**

```java
// 基本配置
service.setInterface(DemoService.class);  // 接口
service.setRef(new DemoServiceImpl());    // 实现类
service.setVersion("1.0.0");              // 版本号
service.setGroup("dubbo-group");          // 分组

// 超时配置
service.setTimeout(3000);                 // 超时时间（毫秒）
service.setRetries(0);                    // 重试次数

// 负载均衡
service.setLoadbalance("roundrobin");     // 负载均衡策略
```

**服务消费者配置：**

```java
// 基本配置
reference.setInterface(DemoService.class);
reference.setVersion("1.0.0");
reference.setGroup("dubbo-group");

// 超时配置
reference.setTimeout(3000);
reference.setRetries(0);

// 负载均衡
reference.setLoadbalance("roundrobin");
```

### 负载均衡策略

| 策略 | 说明 | 适用场景 |
|------|------|---------|
| **random** | 随机 | 默认，提供者性能有差异 |
| **roundrobin** | 轮询 | 提供者性能相近 |
| **leastactive** | 最少活跃调用 | 慢请求优先 |
| **consistenthash** | 一致性哈希 | 需要相同请求到同一提供者 |

---

## 5. 实战场景

### 场景 1：服务降级

```java
// 配置服务降级
service.setMock("true");
service.setMockClass(DemoServiceMock.class);

// 降级实现
public class DemoServiceMock implements DemoService {
    @Override
    public String sayHello(String name) {
        return "服务繁忙，请稍后再试";
    }
}
```

### 场景 2：超时重试

```java
// 配置超时和重试
reference.setTimeout(5000);    // 5 秒超时
reference.setRetries(2);       // 重试 2 次
```

### 场景 3：多版本管理

```java
// 提供者：发布新版本
service.setVersion("2.0.0");

// 消费者：指定版本
reference.setVersion("2.0.0");
```

---

## 6. 常见问题

### 问题 1：服务注册失败

**现象：** 提供者启动后，注册中心看不到服务

**解决方案：**
1. 检查 ZooKeeper 是否启动
2. 检查注册中心地址配置
3. 检查网络是否通畅

### 问题 2：服务调用超时

**现象：** 消费者调用超时

**解决方案：**
1. 增加超时时间：`reference.setTimeout(5000)`
2. 检查提供者性能
3. 检查网络延迟

### 问题 3：负载均衡不生效

**现象：** 请求都打到同一台服务器

**解决方案：**
1. 检查负载均衡策略配置
2. 确认有多台提供者
3. 检查服务版本是否一致

---

## 7. 面试题

### Q1：Dubbo 的工作原理是什么？

**参考答案：**
1. 服务提供者启动，注册服务到注册中心
2. 服务消费者启动，从注册中心订阅服务
3. 注册中心返回服务提供者地址列表
4. 消费者通过负载均衡选择提供者调用
5. 监控中心记录调用情况

### Q2：Dubbo 支持哪些协议？

**参考答案：**
- **dubbo**：默认协议，高性能
- **http**：基于 HTTP 协议
- **hessian**：基于 Hessian 协议
- **rmi**：基于 RMI 协议
- **thrift**：基于 Thrift 协议

### Q3：Dubbo 的负载均衡策略有哪些？

**参考答案：**
- **random**：随机
- **roundrobin**：轮询
- **leastactive**：最少活跃调用
- **consistenthash**：一致性哈希

### Q4：Dubbo 服务降级怎么做？

**参考答案：**
通过配置 mock 实现服务降级：
```java
service.setMock("true");
service.setMockClass(DemoServiceMock.class);
```

### Q5：Dubbo 如何处理超时？

**参考答案：**
1. 配置超时时间：`setTimeout(3000)`
2. 配置重试次数：`setRetries(0)`
3. 使用异步调用
4. 设置熔断降级

---

**下一篇：** [环境搭建](./环境搭建.md)

---

**最后更新：** 2026-03-25  
**状态：** ✅ 已完成
