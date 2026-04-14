# 环境配置

## 2.1 开发环境要求

### JDK 8+

Flowable 要求 JDK 8 或更高版本，推荐使用 JDK 8 或 JDK 11。

```bash
# 检查 JDK 版本
java -version

# 推荐输出
java version "1.8.0_xxx"
```

### Maven/Gradle

推荐使用 Maven 3.6+ 或 Gradle 5.6+。

```bash
# 检查 Maven 版本
mvn -version

# 检查 Gradle 版本
gradle -v
```

### 数据库支持

Flowable 支持多种数据库：

| 数据库 | 最低版本 | 推荐版本 |
|--------|----------|----------|
| MySQL | 5.6+ | 5.7+ / 8.0+ |
| Oracle | 11g+ | 12c+ |
| PostgreSQL | 9.4+ | 10+ |
| SQL Server | 2012+ | 2016+ |
| H2 | 1.4+ | 最新版 |

## 2.2 Maven 依赖配置

### 核心依赖

```xml
<properties>
    <flowable.version>6.8.0</flowable.version>
    <spring.boot.version>2.7.18</spring.boot.version>
</properties>

<dependencies>
    <!-- Flowable 核心引擎 -->
    <dependency>
        <groupId>org.flowable</groupId>
        <artifactId>flowable-engine</artifactId>
        <version>${flowable.version}</version>
    </dependency>

    <!-- Flowable Spring Boot 集成 -->
    <dependency>
        <groupId>org.flowable</groupId>
        <artifactId>flowable-spring-boot-starter</artifactId>
        <version>${flowable.version}</version>
    </dependency>

    <!-- Flowable REST API -->
    <dependency>
        <groupId>org.flowable</groupId>
        <artifactId>flowable-spring-boot-starter-rest</artifactId>
        <version>${flowable.version}</version>
    </dependency>

    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>

    <!-- HikariCP 连接池 -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>4.0.3</version>
    </dependency>

    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>${spring.boot.version}</version>
    </dependency>

    <!-- Spring Boot Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <version>${spring.boot.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 配置文件 (application.yml)

```yaml
server:
  port: 8080

spring:
  application:
    name: flowable-workflow
  datasource:
    url: jdbc:mysql://localhost:3306/flowable_db?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai&nullCatalogMeansCurrent=true
    username: root
    password: your_password
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      minimum-idle: 5
      maximum-pool-size: 20
      idle-timeout: 30000
      max-lifetime: 1800000
      connection-timeout: 30000

flowable:
  # 数据库表更新策略
  database-schema-update: true
  # 启用异步执行器
  async-executor-activate: true
  # 启用历史记录
  history-level: audit
  # 是否检查流程定义
  check-process-definitions: false
  # 流程定义位置
  process-definition-location-prefix: classpath:/processes/
```

## 2.3 数据库配置

### 创建数据库

```sql
-- 创建数据库
CREATE DATABASE flowable_db DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 授权
GRANT ALL PRIVILEGES ON flowable_db.* TO 'flowable'@'localhost' IDENTIFIED BY 'flowable_password';
FLUSH PRIVILEGES;
```

### 数据库初始化

Flowable 会在首次启动时自动创建所需的数据库表。主要表结构包括：

#### 流程定义相关表

| 表名 | 说明 |
|------|------|
| ACT_RE_DEPLOYMENT | 部署信息表 |
| ACT_RE_PROCDEF | 流程定义表 |
| ACT_RE_MODEL | 模型表 |

#### 运行时表

| 表名 | 说明 |
|------|------|
| ACT_RU_EXECUTION | 执行对象表 |
| ACT_RU_TASK | 任务表 |
| ACT_RU_VARIABLE | 变量表 |
| ACT_RU_IDENTITYLINK | 身份关联表 |
| ACT_RU_EVENT_SUBSCR | 事件订阅表 |
| ACT_RU_JOB | 作业表 |

#### 历史表

| 表名 | 说明 |
|------|------|
| ACT_HI_PROCINST | 流程实例历史表 |
| ACT_HI_ACTINST | 活动实例历史表 |
| ACT_HI_TASKINST | 任务历史表 |
| ACT_HI_VARINST | 变量历史表 |
| ACT_HI_DETAIL | 详情历史表 |
| ACT_HI_COMMENT | 评论历史表 |
| ACT_HI_ATTACHMENT | 附件历史表 |

#### 通用表

| 表名 | 说明 |
|------|------|
| ACT_GE_PROPERTY | 属性表 |
| ACT_GE_BYTEARRAY | 字节数组表 |

#### 身份表

| 表名 | 说明 |
|------|------|
| ACT_ID_USER | 用户表 |
| ACT_ID_GROUP | 用户组表 |
| ACT_ID_MEMBERSHIP | 成员关系表 |

### 连接池配置推荐

```yaml
spring:
  datasource:
    hikari:
      # 最小空闲连接数
      minimum-idle: 10
      # 最大连接数
      maximum-pool-size: 50
      # 连接空闲超时时间（毫秒）
      idle-timeout: 600000
      # 连接最大生命周期（毫秒）
      max-lifetime: 1800000
      # 连接超时时间（毫秒）
      connection-timeout: 30000
      # 连接测试查询
      connection-test-query: SELECT 1
      # 是否自动提交
      auto-commit: true
```

## 2.4 启动类配置

```java
import org.flowable.engine.ProcessEngine;
import org.flowable.engine.RepositoryService;
import org.flowable.engine.RuntimeService;
import org.flowable.engine.TaskService;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;

@SpringBootApplication
public class FlowableApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = SpringApplication.run(FlowableApplication.class, args);
        
        // 获取 Flowable 服务
        ProcessEngine processEngine = context.getBean(ProcessEngine.class);
        RepositoryService repositoryService = context.getBean(RepositoryService.class);
        RuntimeService runtimeService = context.getBean(RuntimeService.class);
        TaskService taskService = context.getBean(TaskService.class);
        
        System.out.println("Flowable 引擎启动成功！");
        System.out.println("流程定义数量: " + repositoryService.createProcessDefinitionQuery().count());
    }
}
```

## 2.5 验证配置

### 启动应用

```bash
mvn spring-boot:run
```

### 检查数据库表

启动成功后，检查数据库中是否创建了 Flowable 所需的表：

```sql
USE flowable_db;
SHOW TABLES;
```

应该看到 20+ 张表以 `ACT_` 开头。

## 常见问题

### 问题 1：数据库连接失败

**解决方案**：
- 检查数据库服务是否启动
- 验证数据库连接参数是否正确
- 检查防火墙设置

### 问题 2：表结构创建失败

**解决方案**：
- 确保数据库用户有足够的权限
- 检查数据库字符集设置
- 尝试手动执行建表 SQL

### 问题 3：版本兼容性问题

**解决方案**：
- 使用推荐的 Flowable 版本
- 确保 Spring Boot 版本与 Flowable 版本兼容
- 检查依赖冲突
