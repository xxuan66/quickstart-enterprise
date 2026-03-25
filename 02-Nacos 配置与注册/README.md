# 📋 Nacos 配置与注册

> 10 分钟上手，掌握大厂服务注册与配置中心

---

## 1. 为什么用 Nacos？

**解决了什么问题：**
- 服务太多，找不到？
- 配置分散，难管理？
- 动态配置，要重启？

**大厂为什么用：**
- ✅ 阿里出品，经过双 11 考验
- ✅ 二合一：注册中心 + 配置中心
- ✅ 简单易用：比 Eureka、Config 简单
- ✅ 功能强大：支持动态配置、服务发现

**应用场景：**
- 微服务架构
- 服务注册与发现
- 动态配置管理
- 多环境配置

---

## 2. 快速开始（10 分钟上手）

### 环境要求

- JDK 8+
- Maven 3.6+

### 第一个 Nacos 示例

**步骤 1：启动 Nacos**

```bash
# 下载 Nacos
wget https://github.com/alibaba/nacos/releases/download/2.0.3/nacos-server-2.0.3.zip

# 解压
unzip nacos-server-2.0.3.zip
cd nacos/bin

# 启动（单机模式）
sh startup.sh -m standalone
```

**步骤 2：访问控制台**

浏览器打开：http://localhost:8848/nacos

默认账号：nacos / nacos

**步骤 3：添加 Maven 依赖**

```xml
<dependencies>
    <!-- Nacos 服务发现 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        <version>2021.0.1.0</version>
    </dependency>
    
    <!-- Nacos 配置管理 -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        <version>2021.0.1.0</version>
    </dependency>
</dependencies>
```

**步骤 4：配置服务提供者**

```yaml
# application.yml
spring:
  application:
    name: service-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
      config:
        server-addr: localhost:8848
        file-extension: yaml
```

**步骤 5：启动服务提供者**

```java
@SpringBootApplication
@EnableDiscoveryClient
@RestController
public class ProviderApplication {
    
    @Value("${config.key:default}")
    private String configValue;
    
    @GetMapping("/hello")
    public String hello() {
        return "Hello from provider, config: " + configValue;
    }
    
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

**步骤 6：配置服务消费者**

```yaml
# application.yml
spring:
  application:
    name: service-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
```

**步骤 7：服务调用**

```java
@SpringBootApplication
@EnableDiscoveryClient
@RestController
public class ConsumerApplication {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @GetMapping("/call")
    public String call() {
        // 获取服务实例
        List<ServiceInstance> instances = discoveryClient.getInstances("service-provider");
        if (instances.isEmpty()) {
            return "No provider found";
        }
        
        ServiceInstance instance = instances.get(0);
        String url = "http://" + instance.getHost() + ":" + instance.getPort() + "/hello";
        
        // 调用服务
        RestTemplate restTemplate = new RestTemplate();
        return restTemplate.getForObject(url, String.class);
    }
    
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

**步骤 8：测试**

访问：http://localhost:消费者端口/call

看到输出：`Hello from provider, config: default`

---

## 3. 核心概念

### Nacos 架构图

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  消费者     │────▶│    Nacos    │◀────│  提供者     │
│  Consumer   │     │  注册中心   │     │  Provider   │
│             │     │  配置中心   │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
```

### 核心组件

| 组件 | 作用 |
|------|------|
| **服务注册** | 服务提供者注册到 Nacos |
| **服务发现** | 消费者从 Nacos 获取服务列表 |
| **配置管理** | 动态配置管理 |
| **命名空间** | 隔离环境（dev/test/prod） |
| **配置分组** | 配置分组管理 |

### 工作流程

**服务注册与发现：**
1. 服务提供者启动，注册到 Nacos
2. 服务消费者启动，订阅服务
3. Nacos 推送服务列表给消费者
4. 消费者调用服务提供者

**配置管理：**
1. 在 Nacos 控制台创建配置
2. 服务启动时拉取配置
3. 配置变更，Nacos 推送通知
4. 服务动态刷新配置

---

## 4. 常用配置

### 必会配置项

**服务注册配置：**

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848           # Nacos 地址
        namespace: dev                        # 命名空间
        group: DEFAULT_GROUP                  # 分组
        cluster-name: CLUSTER-A               # 集群名
        weight: 1.0                           # 权重
        enabled: true                         # 是否启用
```

**配置中心配置：**

```yaml
spring:
  cloud:
    nacos:
      config:
        server-addr: localhost:8848           # Nacos 地址
        namespace: dev                        # 命名空间
        group: DEFAULT_GROUP                  # 分组
        file-extension: yaml                  # 文件扩展名
        auto-refresh: true                    # 自动刷新
```

### 配置优先级

**多配置源优先级：**

```
application-{profile}.yaml > application.yaml > bootstrap.yaml
```

**配置加载顺序：**
1. bootstrap.yaml（最早加载）
2. application.yaml
3. Nacos 配置中心配置
4. 命令行参数

---

## 5. 实战场景

### 场景 1：动态配置刷新

```java
@RestController
@RefreshScope  // 开启配置刷新
public class ConfigController {
    
    @Value("${config.key:default}")
    private String configValue;
    
    @GetMapping("/config")
    public String getConfig() {
        return configValue;
    }
}
```

**步骤：**
1. 在 Nacos 控制台修改配置
2. 点击"发布"
3. 访问接口，配置已更新（无需重启）

### 场景 2：多环境配置

**Nacos 配置：**
- service-provider-dev.yaml
- service-provider-test.yaml
- service-provider-prod.yaml

**启动命令：**
```bash
# 开发环境
java -jar app.jar --spring.profiles.active=dev

# 测试环境
java -jar app.jar --spring.profiles.active=test

# 生产环境
java -jar app.jar --spring.profiles.active=prod
```

### 场景 3：服务降级

```java
@FeignClient(
    name = "service-provider",
    fallback = ProviderFallback.class
)
public interface ProviderClient {
    @GetMapping("/hello")
    String hello();
}

@Component
public class ProviderFallback implements ProviderClient {
    @Override
    public String hello() {
        return "服务繁忙，请稍后再试";
    }
}
```

---

## 6. 常见问题

### 问题 1：服务注册失败

**现象：** 服务启动后，Nacos 控制台看不到服务

**解决方案：**
1. 检查 Nacos 是否启动
2. 检查 server-addr 配置
3. 检查网络是否通畅
4. 检查命名空间是否正确

### 问题 2：配置不刷新

**现象：** Nacos 配置修改后，服务配置未更新

**解决方案：**
1. 添加 @RefreshScope 注解
2. 检查 auto-refresh 配置
3. 检查配置 Data ID 是否正确

### 问题 3：服务调用失败

**现象：** 消费者调用提供者失败

**解决方案：**
1. 检查服务是否注册成功
2. 检查服务名是否正确
3. 检查网络是否通畅
4. 检查负载均衡配置

---

## 7. 面试题

### Q1：Nacos 和 Eureka 有什么区别？

**参考答案：**
- **CAP 理论：** Nacos 支持 CP 和 AP，Eureka 只支持 AP
- **功能：** Nacos 是注册中心 + 配置中心，Eureka 只是注册中心
- **连接方式：** Nacos 支持长轮询，Eureka 是定时拉取
- **厂商：** Nacos 是阿里，Eureka 是 Netflix

### Q2：Nacos 配置中心的工作原理？

**参考答案：**
1. 服务启动时从 Nacos 拉取配置
2. Nacos 使用长轮询监听配置变更
3. 配置变更时，Nacos 推送通知给服务
4. 服务收到通知后，刷新配置（@RefreshScope）

### Q3：Nacos 服务发现的工作原理？

**参考答案：**
1. 服务提供者启动，注册到 Nacos
2. 服务消费者启动，订阅服务
3. Nacos 推送服务列表给消费者
4. 消费者缓存服务列表，定期更新
5. 消费者通过负载均衡调用服务

### Q4：Nacos 如何实现配置隔离？

**参考答案：**
通过命名空间（Namespace）和分组（Group）实现隔离：
- **命名空间：** 隔离环境（dev/test/prod）
- **分组：** 隔离项目或业务线
- **Data ID：** 唯一标识一个配置

### Q5：Nacos 支持哪些负载均衡策略？

**参考答案：**
- **权重轮询：** 根据权重轮询
- **随机：** 随机选择
- **最少连接：** 选择连接数最少的
- **一致性哈希：** 相同请求到同一服务

---

**下一篇：** [Nacos 安装配置](./Nacos 安装配置.md)

---

**最后更新：** 2026-03-25  
**状态：** ✅ 已完成
