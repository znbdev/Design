Enterprise Implementation Handbook
=====

这是一个关于**企业级应用具体技术实现**的详细指南。

为了确保落地性，本实战文档假定技术栈为目前企业级最主流的 **Java 17+ (Spring Boot 3.x + Spring Cloud)** 配合 **Cloud Native**
组件。

-----

# 企业级应用技术实现手册 (Enterprise Implementation Handbook)

## 0. 依赖管理与版本兼容性 (Dependency Management & Version Compatibility)

### 0.1 Maven 父项目配置 (Parent POM)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company</groupId>
    <artifactId>enterprise-app</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>api</module>
        <module>app</module>
        <module>domain</module>
        <module>infra</module>
        <module>common</module>
    </modules>

    <properties>
        <!-- Java 17+ required for Spring Boot 3.x -->
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        
        <!-- Spring Boot & Spring Cloud -->
        <spring-boot.version>3.2.0</spring-boot.version>
        <spring-cloud.version>2023.0.0</spring-cloud.version>
        <spring-cloud-alibaba.version>2022.0.0.0</spring-cloud-alibaba.version>
        
        <!-- Database 数据库相关 -->
        <mysql.version>8.0.33</mysql.version>  <!-- MySQL 主数据库 -->
        <h2.version>2.2.224</h2.version>        <!-- H2 内存数据库，用于单元测试 -->
        <hikari.version>5.0.1</hikari.version>    <!-- HikariCP 高性能数据库连接池 -->
        
        <!-- Redis & Cache 缓存与分布式锁 -->
        <redisson.version>3.24.3</redisson.version>  <!-- Redisson 分布式锁和Redis客户端 -->
        
        <!-- Message Queue 消息队列 -->
        <rocketmq.version>5.1.4</rocketmq.version>  <!-- RocketMQ 消息队列（可替换为RabbitMQ） -->
        
        <!-- Security 安全认证 -->
        <jjwt.version>0.12.3</jjwt.version>        <!-- JWT JSON Web Token 令牌 -->
        <spring-security.version>6.2.0</spring-security.version>  <!-- Spring Security 安全框架 -->
        
        <!-- Documentation API文档 -->
        <springdoc.version>2.2.0</springdoc.version>  <!-- SpringDoc OpenAPI 3.0 文档生成 -->
        
        <!-- Utilities 工具类库 -->
        <lombok.version>1.18.30</lombok.version>    <!-- Lombok 代码简化注解处理器 -->
        <commons-lang3.version>3.13.0</commons-lang3.version>  <!-- Apache Commons Lang3 工具类 -->
        <jackson.version>2.15.3</jackson.version>    <!-- Jackson JSON 序列化/反序列化 -->
        
        <!-- Validation 数据校验 -->
        <jakarta.validation.version>3.0.2</jakarta.validation.version>  <!-- Jakarta Bean Validation API -->
        <hibernate-validator.version>8.0.0.Final</hibernate-validator.version>  <!-- Hibernate Validator 校验实现 -->
        
        <!-- Testing 测试框架 -->
        <testcontainers.version>1.19.3</testcontainers.version>  <!-- Testcontainers 集成测试容器 -->
        <wiremock.version>3.2.0</wiremock.version>    <!-- WireMock HTTP 服务模拟 -->
        
        <!-- Monitoring 监控指标 -->
        <micrometer.version>1.12.0</micrometer.version>  <!-- Micrometer 应用监控指标收集 -->
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- Spring Boot BOM -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            
            <!-- Spring Cloud BOM -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            
            <!-- Database Dependencies 数据库相关依赖 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
                <!-- MySQL 数据库驱动，用于生产环境数据持久化 -->
            </dependency>
            
            <dependency>
                <groupId>com.h2database</groupId>
                <artifactId>h2</artifactId>
                <version>${h2.version}</version>
                <!-- H2 内存数据库，用于单元测试和开发环境快速验证 -->
            </dependency>
            
            <dependency>
                <groupId>com.zaxxer</groupId>
                <artifactId>HikariCP</artifactId>
                <version>${hikari.version}</version>
                <!-- HikariCP 高性能数据库连接池，Spring Boot 默认选择 -->
            </dependency>
            
            <!-- Redis & Distributed Lock Redis缓存与分布式锁 -->
            <dependency>
                <groupId>org.redisson</groupId>
                <artifactId>redisson-spring-boot-starter</artifactId>
                <version>${redisson.version}</version>
                <!-- Redisson 分布式锁和 Redis 客户端，支持分布式场景 -->
            </dependency>
            
            <!-- Message Queue 消息队列 -->
            <dependency>
                <groupId>org.apache.rocketmq</groupId>
                <artifactId>rocketmq-spring-boot-starter</artifactId>
                <version>${rocketmq.version}</version>
                <!-- RocketMQ 消息队列，用于异步处理和系统解耦 -->
            </dependency>
            
            <!-- Security & JWT 安全认证与JWT令牌 -->
            <dependency>
                <groupId>io.jsonwebtoken</groupId>
                <artifactId>jjwt-api</artifactId>
                <version>${jjwt.version}</version>
                <!-- JWT API 接口定义 -->
            </dependency>
            
            <dependency>
                <groupId>io.jsonwebtoken</groupId>
                <artifactId>jjwt-impl</artifactId>
                <version>${jjwt.version}</version>
                <!-- JWT 实现类，包含签名和验证逻辑 -->
            </dependency>
            
            <dependency>
                <groupId>io.jsonwebtoken</groupId>
                <artifactId>jjwt-jackson</artifactId>
                <version>${jjwt.version}</version>
                <!-- JWT Jackson 集成，用于 JSON 序列化 -->
            </dependency>
            
            <!-- API Documentation API文档生成 -->
            <dependency>
                <groupId>org.springdoc</groupId>
                <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
                <version>${springdoc.version}</version>
                <!-- SpringDoc OpenAPI 3.0 文档生成，替代 Swagger2 -->
            </dependency>
            
            <!-- Utilities 工具类库 -->
            <dependency>
                <groupId>org.projectlombok</groupId>
                <artifactId>lombok</artifactId>
                <version>${lombok.version}</version>
                <!-- Lombok 注解处理器，简化 Java 代码编写 -->
            </dependency>
            
            <dependency>
                <groupId>org.apache.commons</groupId>
                <artifactId>commons-lang3</artifactId>
                <version>${commons-lang3.version}</version>
                <!-- Apache Commons Lang3 常用工具类集合 -->
            </dependency>
            
            <dependency>
                <groupId>com.fasterxml.jackson.core</groupId>
                <artifactId>jackson-databind</artifactId>
                <version>${jackson.version}</version>
                <!-- Jackson 数据绑定，JSON 序列化/反序列化核心库 -->
            </dependency>
            
            <!-- Validation 数据校验框架 -->
            <dependency>
                <groupId>jakarta.validation</groupId>
                <artifactId>jakarta.validation-api</artifactId>
                <version>${jakarta.validation.version}</version>
                <!-- Jakarta Bean Validation API 标准接口 -->
            </dependency>
            
            <dependency>
                <groupId>org.hibernate.validator</groupId>
                <artifactId>hibernate-validator</artifactId>
                <version>${hibernate-validator.version}</version>
                <!-- Hibernate Validator 校验实现，支持注解式校验 -->
            </dependency>
            
            <!-- Testing 测试框架相关依赖 -->
            <dependency>
                <groupId>org.testcontainers</groupId>
                <artifactId>testcontainers-bom</artifactId>
                <version>${testcontainers.version}</version>
                <type>pom</type>
                <scope>import</scope>
                <!-- Testcontainers 集成测试框架，支持 Docker 容器化测试 -->
            </dependency>
            
            <dependency>
                <groupId>com.github.tomakehurst</groupId>
                <artifactId>wiremock-jre8</artifactId>
                <version>${wiremock.version}</version>
                <!-- WireMock HTTP 服务模拟，用于 API 测试和 Mock -->
            </dependency>
            
            <!-- Monitoring 应用监控与指标收集 -->
            <dependency>
                <groupId>io.micrometer</groupId>
                <artifactId>micrometer-registry-prometheus</artifactId>
                <version>${micrometer.version}</version>
                <!-- Micrometer Prometheus 指标注册，用于应用监控 -->
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### 0.2 核心模块依赖配置

#### 0.2.1 API 模块 (api/pom.xml)

```xml
<dependencies>
    <!-- Spring Boot Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- API Documentation -->
    <dependency>
        <groupId>org.springdoc</groupId>
        <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    </dependency>
    
    <!-- Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    
    <!-- Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
    </dependency>
    
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
    </dependency>
    
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
    </dependency>
    
    <!-- App Module -->
    <dependency>
        <groupId>com.company</groupId>
        <artifactId>app</artifactId>
        <version>${project.version}</version>
    </dependency>
    
    <!-- Common Module -->
    <dependency>
        <groupId>com.company</groupId>
        <artifactId>common</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies>
```

#### 0.2.2 基础设施模块 (infra/pom.xml)

```xml
<dependencies>
    <!-- Database -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
    </dependency>
    
    <!-- Redis -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.redisson</groupId>
        <artifactId>redisson-spring-boot-starter</artifactId>
    </dependency>
    
    <!-- Message Queue -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    
    <!-- Monitoring -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
    
    <!-- Domain Module -->
    <dependency>
        <groupId>com.company</groupId>
        <artifactId>domain</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies>
```

#### 0.2.3 测试依赖配置 (通用)

```xml
<dependencies>
    <!-- Spring Boot Test -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- Testcontainers for Integration Testing -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <scope>test</scope>
    </dependency>
    
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>mysql</artifactId>
        <scope>test</scope>
    </dependency>
    
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>redis</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- WireMock for HTTP Service Mocking -->
    <dependency>
        <groupId>com.github.tomakehurst</groupId>
        <artifactId>wiremock-jre8</artifactId>
        <scope>test</scope>
    </dependency>
    
    <!-- H2 for Unit Testing -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 0.3 版本兼容性说明

| 组件类别 | 推荐版本 | 兼容性说明 |
|---------|---------|-----------|
| **Java** | 17+ | Spring Boot 3.x 要求 Java 17+，推荐使用 LTS 长期支持版本 |
| **Spring Boot** | 3.2.0 | 稳定版本，支持所有主流组件，企业级应用首选 |
| **Spring Cloud** | 2023.0.0 | 与 Spring Boot 3.2.0 完全兼容，微服务架构支持 |
| **MySQL** | 8.0.33 | 推荐使用 8.0+ 版本，性能和安全性显著提升 |
| **H2 Database** | 2.2.224 | 内存数据库，用于单元测试和快速原型开发 |
| **HikariCP** | 5.0.1 | Spring Boot 默认连接池，高性能、低延迟 |
| **Redis** | 7.0+ | Redisson 3.24.3 兼容 Redis 7.0+，缓存和分布式锁 |
| **RabbitMQ** | 3.12+ | Spring AMQP 官方支持，稳定可靠的消息队列 |
| **JWT** | 0.12.3 | 最新版本，安全性更好，支持更多加密算法 |
| **Jackson** | 2.15.3 | Spring Boot 默认 JSON 处理库，性能优异 |
| **Lombok** | 1.18.30 | 代码简化工具，减少样板代码，提高开发效率 |
| **Swagger/OpenAPI** | 2.2.0 | SpringDoc 替代 Swagger2，支持 Spring Boot 3.x 和 OpenAPI 3.0 |

### 0.4 环境配置建议

#### 0.4.1 开发环境 (application-dev.yml)

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/enterprise_dev?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: root
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      idle-timeout: 300000
      connection-timeout: 30000
      max-lifetime: 1800000
      leak-detection-threshold: 60000
      
  redis:
    host: localhost
    port: 6379
    password: 
    database: 0
    timeout: 3000ms
    lettuce:
      pool:
        max-active: 20
        max-idle: 10
        min-idle: 5
        max-wait: 3000ms

# JPA 配置
jpa:
  hibernate:
    ddl-auto: update
  show-sql: true
  properties:
    hibernate:
      format_sql: true
      dialect: org.hibernate.dialect.MySQL8Dialect
  open-in-view: false

# 日志配置
logging:
  level:
    com.company.project: DEBUG
    org.springframework.web: DEBUG
    org.springframework.security: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{TRACE_ID}] %-5level %logger - %msg%n"
```

#### 0.4.2 生产环境 (application-prod.yml)

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 50
      minimum-idle: 10
      idle-timeout: 300000
      connection-timeout: 30000
      max-lifetime: 1800000
      leak-detection-threshold: 60000
        
  redis:
    lettuce:
      pool:
        max-active: 50
        max-idle: 20
        min-idle: 10
        max-wait: 3000ms

# 日志配置 - 生产环境
logging:
  level:
    com.company.project: INFO
    org.springframework.web: WARN
    org.springframework.security: WARN
    org.springframework.transaction: WARN
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{TRACE_ID}] %-5level %logger - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{TRACE_ID}] %-5level %logger - %msg%n"
  file:
    name: /var/log/enterprise-app/application.log
    max-size: 100MB
    max-history: 30

# 监控配置
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  metrics:
    export:
      prometheus:
        enabled: true
```

-----

## 1\. 工程脚手架与目录结构 (Project Scaffolding)

采用 **整洁架构 (Clean Architecture)** 与 **DDD** 结合的目录结构。这种结构确保了业务逻辑（Domain）与框架（Infrastructure）的完全解耦。

```text
com.company.project
├── api                 // [接入层] Controller, DTO, RPC Implementation
│   ├── request         // 入参 DTO
│   ├── response        // 出参 DTO (ViewModel)
│   └── OrderController.java
├── app                 // [应用层] Orchestration, CQS, Event Listeners
│   ├── service         // 应用服务 (非业务逻辑，仅编排)
│   └── assembler       // DTO <-> Entity 转换器
├── domain              // [核心领域层] 纯 Java 对象，无 Spring 依赖
│   ├── model           // Aggregates, Entities, Value Objects
│   ├── service         // 领域服务 (处理跨实体逻辑)
│   └── gateway         // 接口定义 (Repository Interface, External API Interface)
├── infra               // [基础设施层] 实现 Domain 定义的接口
│   ├── persistence     // DB 实现 (MyBatis/JPA), DO (Data Object)
│   ├── external        // 第三方 API 客户端实现
│   └── config          // Spring 配置类
└── common              // [公共组件] Utils, Constants, Base Exception
```

-----

## 2\. 统一交互规约实现 (Interface Protocol)

企业级应用必须有统一的响应格式和异常处理机制，严禁直接返回 `500` 堆栈信息给前端。

### 2.1 统一响应对象 (Generic Response)

```java
// 所有的 API 返回都必须包装在此对象中
public class Result<T> {
    private String code;      // 业务状态码 (e.g., "00000", "A0401")
    private String message;   // 提示信息
    private T data;           // 业务数据
    private String traceId;   // 链路ID，方便排查问题

    public static <T> Result<T> success(T data) { ...}

    public static <T> Result<T> fail(BizException e) { ...}
}
```

### 2.2 全局异常处理器 (Global Exception Handler)

利用 `@ControllerAdvice` 捕获所有异常，映射为友好的错误码。

```java

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(BizException.class)
    public Result<Void> handleBizException(BizException e) {
        // 记录 WARN 日志
        log.warn("Business exception: {}", e.getMessage());
        return Result.fail(e.getCode(), e.getMessage());
    }

    @ExceptionHandler(Exception.class)
    public Result<Void> handleSystemException(Exception e) {
        // 记录 ERROR 日志，包含堆栈
        log.error("System exception", e);
        // 生产环境隐藏具体错误，返回通用提示
        return Result.fail("B0001", "系统繁忙，请稍后再试");
    }
}
```

-----

## 3\. 高并发与稳定性组件实现 (High Concurrency Implementation)

### 3.1 接口幂等性 (Idempotency)

防止网络重试导致的重复下单或重复扣款。

* **技术选型:** Redis + Lua 脚本 + AOP 注解。
* **实现逻辑:**
    1. 客户端先请求 `token` 接口获取唯一 Ticket。
    2. 提交业务请求时带上 `Ticket`。
    3. 服务端执行 `delete if exists` 逻辑。

**AOP 实现核心代码:**

```java

@Aspect
@Component
public class IdempotentAspect {
    @Autowired
    private RedisTemplate redisTemplate;

    @Around("@annotation(idempotent)")
    public Object around(ProceedingJoinPoint joinPoint, Idempotent idempotent) {
        String token = getTokenFromHeader();
        // LUA 脚本保证原子性：查找并删除
        Boolean success = redisTemplate.execute(deleteScript, Collections.singletonList(token));

        if (!success) {
            throw new BizException("请勿重复提交");
        }
        return joinPoint.proceed();
    }
}
```

### 3.2 分布式锁 (Distributed Lock)

用于解决“超卖”或资源竞争问题。

* **技术选型:** Redisson (推荐，自动续期，实现简单)。
* **最佳实践:** 务必在 `finally` 块中释放锁。

<!-- end list -->

```java

@Autowired
private RedissonClient redisson;

public void deductInventory(Long skuId, Integer count) {
    String lockKey = "lock:inventory:" + skuId;
    RLock lock = redisson.getLock(lockKey);

    try {
        // 尝试加锁，最多等待 5s，上锁后 10s 自动解锁
        if (lock.tryLock(5, 10, TimeUnit.SECONDS)) {
            // 业务逻辑：查库存 -> 判断 -> 扣减
        } else {
            throw new BizException("系统繁忙，请稍后重试");
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        // 只能释放自己的锁
        if (lock.isHeldByCurrentThread()) {
            lock.unlock();
        }
    }
}
```

-----

## 4\. 数据持久化与一致性 (Persistence & Consistency)

### 4.1 Spring Data JPA 配置优化

* **乐观锁:** 使用 `@Version` 注解实现乐观锁，防止并发覆盖。
* **审计功能:** 使用 `@CreatedDate`, `@LastModifiedDate`, `@CreatedBy` 等注解自动处理审计字段。
* **实体配置:** 合理使用 `@Entity`, `@Table`, `@Column` 等注解进行数据库映射。

### 4.2 最终一致性实现 (Local Message Table)

在不能使用强事务（2PC）的跨服务场景下，使用**本地消息表**模式。

**流程实现:**

1. **事务 A:** \* 执行业务 SQL (订单表)。
    * Insert 消息表 (`status = PENDING`)。
    * Commit 事务。
2. **后台线程/定时任务:**
    * 扫描消息表 `PENDING` 记录。
    * 发送到 MQ。
    * 发送成功后 Update 消息表 `status = SENT`。
3. **消费端:**
    * 消费 MQ 消息。
    * 执行业务。
    * **关键:** 消费端需实现幂等。

-----

## 5\. 可观测性实施 (Observability)

### 5.1 全链路 TraceID 注入 (MDC)

确保所有日志（包括异步线程池中的日志）都带有同一个 TraceID。

```java
// 拦截器：在请求进入时生成 TraceID
public class TraceIdInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(...) {
        String traceId = UUID.randomUUID().toString().replace("-", "");
        MDC.put("TRACE_ID", traceId); // 放入日志上下文
        return true;
    }

    @Override
    public void afterCompletion(...) {
        MDC.clear(); // 必须清理，防止内存泄漏
    }
}

// Logback 配置 pattern
// <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{TRACE_ID}] %-5level %logger - %msg%n</pattern>
```

### 5.2 线程池装饰器

默认线程池会丢失 MDC 上下文，需要自定义装饰器。

```java
public class MdcTaskDecorator implements TaskDecorator {
    @Override
    public Runnable decorate(Runnable runnable) {
        Map<String, String> contextMap = MDC.getCopyOfContextMap();
        return () -> {
            try {
                if (contextMap != null) MDC.setContextMap(contextMap);
                runnable.run();
            } finally {
                MDC.clear();
            }
        };
    }
}
```

-----

## 6\. 安全与上下文 (Security & Context)

### 6.1 用户上下文传递 (UserContext)

不要在每个 Service 方法参数中传递 User 对象。使用 `ThreadLocal`。

```java
public class UserContext {
    private static final ThreadLocal<CurrentUser> holder = new ThreadLocal<>();

    public static void set(CurrentUser user) {
        holder.set(user);
    }

    public static CurrentUser get() {
        return holder.get();
    }

    public static void clear() {
        holder.remove();
    }
}
```

### 6.2 过滤器链

1. **AuthFilter:** 解析 Header 中的 JWT -\> 校验签名 -\> 解析 UserInfo -\> 存入 `UserContext`。
2. **业务逻辑:** 直接调用 `UserContext.get().getUserId()`。
3. **Response:** 请求结束，清理 `UserContext`。

-----

## 7\. API文档配置 (API Documentation)

### 7.1 SpringDoc OpenAPI 配置

使用 SpringDoc 替代已停止维护的 Swagger2，支持 Spring Boot 3.x。

```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
                .info(new Info()
                        .title("企业级应用 API")
                        .version("1.0.0")
                        .description("基于 Spring Boot 3.x + Spring Cloud 的企业级应用接口文档")
                        .contact(new Contact()
                                .name("开发团队")
                                .email("dev@company.com")))
                .components(new Components()
                        .addSecuritySchemes("bearer-jwt", new SecurityScheme()
                                .type(SecurityScheme.Type.HTTP)
                                .scheme("bearer")
                                .bearerFormat("JWT")
                                .in(SecurityScheme.In.HEADER)
                                .name("Authorization")))
                .addSecurityItem(new SecurityRequirement().addList("bearer-jwt"));
    }

    @Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
                .group("public")
                .pathsToMatch("/api/public/**")
                .build();
    }

    @Bean
    public GroupedOpenApi adminApi() {
        return GroupedOpenApi.builder()
                .group("admin")
                .pathsToMatch("/api/admin/**")
                .build();
    }
}
```

### 7.2 Controller 注解示例

```java
@RestController
@RequestMapping("/api/orders")
@Tag(name = "订单管理", description = "订单相关的接口")
@Validated
public class OrderController {

    @PostMapping
    @Operation(summary = "创建订单", description = "创建新的订单")
    @ApiResponse(responseCode = "200", description = "创建成功",
            content = @Content(schema = @Schema(implementation = OrderResponse.class)))
    @ApiResponse(responseCode = "400", description = "参数错误")
    @ApiResponse(responseCode = "500", description = "系统错误")
    public Result<OrderResponse> createOrder(
            @Valid @RequestBody @Parameter(description = "订单创建请求") OrderRequest request) {
        // 业务逻辑
        return Result.success(orderResponse);
    }

    @GetMapping("/{id}")
    @Operation(summary = "查询订单", description = "根据ID查询订单详情")
    public Result<OrderResponse> getOrder(
            @PathVariable @Parameter(description = "订单ID") Long id) {
        // 业务逻辑
        return Result.success(orderResponse);
    }
}
```

### 7.3 DTO 注解示例

```java
@Data
@Schema(description = "订单创建请求")
public class OrderRequest {

    @Schema(description = "用户ID", example = "12345", required = true)
    @NotNull(message = "用户ID不能为空")
    private Long userId;

    @Schema(description = "商品列表", required = true)
    @NotEmpty(message = "商品列表不能为空")
    @Valid
    private List<OrderItemRequest> items;

    @Schema(description = "收货地址", example = "北京市朝阳区xxx街道")
    @NotBlank(message = "收货地址不能为空")
    @Length(max = 200, message = "收货地址长度不能超过200字符")
    private String address;
}

@Data
@Schema(description = "订单响应")
public class OrderResponse {

    @Schema(description = "订单ID", example = "10086")
    private Long orderId;

    @Schema(description = "订单状态", example = "PAID")
    private String status;

    @Schema(description = "订单总金额", example = "299.00")
    @DecimalMin(value = "0.01", message = "订单金额必须大于0")
    private BigDecimal totalAmount;

    @Schema(description = "创建时间", example = "2023-12-01T10:30:00")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createTime;
}
```

### 7.4 全局异常响应文档化

```java
@RestControllerAdvice
@Tag(name = "全局异常处理", description = "统一异常处理")
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @Operation(summary = "参数校验异常", hidden = true)
    public Result<Void> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
                .map(error -> error.getField() + ": " + error.getDefaultMessage())
                .collect(Collectors.joining(", "));
        return Result.fail("A0401", message);
    }

    @ExceptionHandler(BizException.class)
    @Operation(summary = "业务异常", hidden = true)
    public Result<Void> handleBizException(BizException e) {
        log.warn("Business exception: {}", e.getMessage());
        return Result.fail(e.getCode(), e.getMessage());
    }
}
```

### 7.5 访问地址

* **Swagger UI**: `http://localhost:8080/swagger-ui.html`
* **OpenAPI JSON**: `http://localhost:8080/v3/api-docs`
* **分组API**: `http://localhost:8080/v3/api-docs/admin`

-----

## 8\. 基础设施即代码 (Infrastructure / Dockerfile)

使用多阶段构建 (Multi-stage Build) 减小镜像体积，不仅安全且高效。

```dockerfile
# Stage 1: Build
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
# 跳过测试以加速构建
RUN mvn package -DskipTests

# Stage 2: Runtime (使用精简版 JRE)
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/app.jar app.jar

# 暴露端口
EXPOSE 8080

# 优化 JVM 参数：感知容器内存限制
ENTRYPOINT ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75.0", "-jar", "app.jar"]
```

-----

## 9\. 总结与检查点

在实施以上代码时，请检查：

1. **异常处理:** 是否每个 `catch` 块都记录了日志？
2. **超时:** 是否所有的 HTTP 请求、DB 查询、Redis 访问都设置了明确的 `ConnectTimeout` 和 `ReadTimeout`？
3. **资源释放:** 所有的 `InputStream`, `Lock`, `ThreadLocal` 是否都在 `finally` 块中进行了清理？

