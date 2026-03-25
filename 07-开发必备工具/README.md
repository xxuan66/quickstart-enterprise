# 🛠️ 开发必备工具

> Maven、Git、SkyWalking、ELK 快速上手

---

## 1. Maven 快速上手

### 为什么用 Maven？

**解决了什么问题：**
- 依赖管理太麻烦？
- 构建流程不统一？
- 项目结构混乱？

**核心功能：**
- ✅ 依赖管理
- ✅ 项目构建
- ✅ 统一结构

### 快速开始

**pom.xml 配置：**

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>my-app</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.7.0</version>
        </dependency>
        
        <!-- MySQL -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.28</version>
        </dependency>
    </dependencies>
    
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**常用命令：**

```bash
# 清理
mvn clean

# 编译
mvn compile

# 测试
mvn test

# 打包
mvn package

# 安装
mvn install

# 跳过测试打包
mvn package -DskipTests
```

---

## 2. Git 工作流

### 为什么用 Git？

**解决了什么问题：**
- 代码版本管理？
- 团队协作？
- 代码回滚？

### 快速开始

**基本配置：**

```bash
# 配置用户信息
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 初始化仓库
git init

# 克隆仓库
git clone https://github.com/user/repo.git
```

**常用命令：**

```bash
# 查看状态
git status

# 添加文件
git add .

# 提交
git commit -m "提交信息"

# 推送
git push origin main

# 拉取
git pull origin main

# 创建分支
git checkout -b feature-branch

# 切换分支
git checkout feature-branch

# 合并分支
git merge feature-branch

# 查看日志
git log
```

### Git 工作流

**开发流程：**
```
main 分支（生产）
  │
  ├── develop 分支（开发）
  │     │
  │     ├── feature-1 分支
  │     ├── feature-2 分支
  │     └── feature-3 分支
```

**操作步骤：**
```bash
# 1. 从 develop 创建功能分支
git checkout develop
git checkout -b feature-xxx

# 2. 开发并提交
git add .
git commit -m "feat: 完成 xxx 功能"

# 3. 推送到远程
git push origin feature-xxx

# 4. 创建 Pull Request

# 5. 合并到 develop

# 6. 删除功能分支
git branch -d feature-xxx
```

---

## 3. SkyWalking 链路追踪

### 为什么用 SkyWalking？

**解决了什么问题：**
- 微服务调用链太长？
- 性能瓶颈难定位？
- 故障排查困难？

**核心功能：**
- ✅ 链路追踪
- ✅ 性能监控
- ✅ 服务依赖分析

### 快速开始

**步骤 1：启动 SkyWalking**

```bash
# 使用 Docker 启动
docker run -d \
  --name skywalking \
  -e SW_CORE_RECORD_DATA_TTL=2 \
  -p 12800:12800 \
  -p 8080:8080 \
  apache/skywalking-oap-server:8.9.1
```

**步骤 2：添加 Agent**

```bash
# 下载 Agent
wget https://downloads.apache.org/skywalking/8.9.1/apache-skywalking-apm-8.9.1.tar.gz

# 配置 agent.config
vim config/agent.config
```

**步骤 3：启动应用**

```bash
java -javaagent:/path/to/skywalking-agent.jar \
     -Dskywalking.agent.service_name=my-app \
     -Dskywalking.collector.backend_service=localhost:12800 \
     -jar app.jar
```

**步骤 4：查看链路**

访问：http://localhost:8080

---

## 4. ELK 日志系统

### 为什么用 ELK？

**解决了什么问题：**
- 日志分散难收集？
- 日志量大难查询？
- 故障排查效率低？

**核心组件：**
- **Elasticsearch：** 搜索引擎
- **Logstash：** 日志收集
- **Kibana：** 可视化展示

### 快速开始

**步骤 1：启动 Elasticsearch**

```bash
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  elasticsearch:7.15.0
```

**步骤 2：启动 Kibana**

```bash
docker run -d \
  --name kibana \
  -p 5601:5601 \
  -e ELASTICSEARCH_HOSTS=http://elasticsearch:9200 \
  kibana:7.15.0
```

**步骤 3：配置 Logstash**

```ruby
# logstash.conf
input {
  file {
    path => "/var/log/app/*.log"
    start_position => "beginning"
  }
}

filter {
  grok {
    match => { "message" => "%{COMBINEDAPACHELOG}" }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
  }
}
```

**步骤 4：查看日志**

访问：http://localhost:5601

---

## 5. 面试题

### Q1：Maven 的生命周期？

**参考答案：**
- clean：清理
- default：编译、测试、打包
- site：生成站点

### Q2：Git merge 和 rebase 的区别？

**参考答案：**
- **merge：** 保留完整历史，有合并节点
- **rebase：** 历史线性，更干净

### Q3：SkyWalking 的工作原理？

**参考答案：**
- Java Agent 字节码增强
- 自动采集调用链
- 上报到 OAP 服务器
- 存储到数据库
- UI 展示

### Q4：ELK 如何收集日志？

**参考答案：**
- Logstash 收集日志
- 过滤和解析
- 发送到 Elasticsearch
- Kibana 展示

---

**最后更新：** 2026-03-25  
**状态：** ✅ 已完成
