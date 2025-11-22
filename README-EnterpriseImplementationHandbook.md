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

    public static <T> Result<T> success(T data) {
        // 实现成功响应逻辑
        return new Result<>("00000", "success", data, null);
    }

    public static <T> Result<T> fail(BizException e) {
        // 实现失败响应逻辑
        return new Result<>(e.getCode(), e.getMessage(), null, null);
    }
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
        try {
            return joinPoint.proceed();
        } catch (Throwable e) {
            // 处理异常
            throw new BizException("系统异常");
        }
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
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String traceId = UUID.randomUUID().toString().replace("-", "");
        MDC.put("TRACE_ID", traceId); // 放入日志上下文
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
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

## 9. 核心业务模块实现 (Core Business Modules)

### 9.1 商品管理模块 (Product Management)

#### 9.1.1 领域模型设计

```java
// domain/model/Product.java
@Entity
@Table(name = "products")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;                    // 商品名称
    
    @Column(nullable = false, length = 500)
    private String description;             // 商品描述
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal price;               // 商品价格
    
    @Column(nullable = false)
    private Integer stock;                   // 库存数量
    
    @Column(nullable = false)
    private String category;                // 商品分类
    
    @Column(length = 1000)
    private String images;                   // 商品图片URL，多个用逗号分隔
    
    @Column(nullable = false)
    private Boolean status;                  // 商品状态：true-上架，false-下架
    
    @CreatedDate
    private LocalDateTime createTime;
    
    @LastModifiedDate
    private LocalDateTime updateTime;
    
    @Version
    private Long version;                    // 乐观锁版本号
}
```

#### 9.1.2 商品管理API实现

```java
// api/controller/ProductController.java
@RestController
@RequestMapping("/api/products")
@Tag(name = "商品管理", description = "商品相关的接口")
@Validated
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @PostMapping
    @Operation(summary = "创建商品", description = "创建新的商品")
    @PreAuthorize("hasRole('ADMIN')")
    public Result<ProductResponse> createProduct(
            @Valid @RequestBody ProductRequest request) {
        Product product = productService.createProduct(request);
        return Result.success(ProductResponse.from(product));
    }
    
    @PutMapping("/{id}")
    @Operation(summary = "更新商品", description = "更新商品信息")
    @PreAuthorize("hasRole('ADMIN')")
    public Result<ProductResponse> updateProduct(
            @PathVariable Long id,
            @Valid @RequestBody ProductRequest request) {
        Product product = productService.updateProduct(id, request);
        return Result.success(ProductResponse.from(product));
    }
    
    @GetMapping("/{id}")
    @Operation(summary = "查询商品", description = "根据ID查询商品详情")
    public Result<ProductResponse> getProduct(@PathVariable Long id) {
        Product product = productService.getProduct(id);
        return Result.success(ProductResponse.from(product));
    }
    
    @GetMapping
    @Operation(summary = "商品列表", description = "分页查询商品列表")
    public Result<PageResult<ProductResponse>> getProducts(
            @RequestParam(defaultValue = "1") Integer page,
            @RequestParam(defaultValue = "10") Integer size,
            @RequestParam(required = false) String category,
            @RequestParam(required = false) String keyword) {
        
        ProductQuery query = ProductQuery.builder()
                .page(page)
                .size(size)
                .category(category)
                .keyword(keyword)
                .build();
                
        PageResult<Product> result = productService.getProducts(query);
        List<ProductResponse> responses = result.getData().stream()
                .map(ProductResponse::from)
                .collect(Collectors.toList());
                
        return Result.success(new PageResult<>(responses, result.getTotal()));
    }
    
    @PutMapping("/{id}/stock")
    @Operation(summary = "更新库存", description = "更新商品库存")
    @PreAuthorize("hasRole('ADMIN')")
    public Result<Void> updateStock(
            @PathVariable Long id,
            @Valid @RequestBody StockUpdateRequest request) {
        productService.updateStock(id, request.getQuantity());
        return Result.success();
    }
    
    @PutMapping("/{id}/status")
    @Operation(summary = "更新状态", description = "上架/下架商品")
    @PreAuthorize("hasRole('ADMIN')")
    public Result<Void> updateStatus(
            @PathVariable Long id,
            @RequestParam Boolean status) {
        productService.updateStatus(id, status);
        return Result.success();
    }
}
```

#### 9.1.3 商品服务实现

```java
// app/service/ProductService.java
@Service
@Transactional
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private ProductAssembler productAssembler;
    
    public Product createProduct(ProductRequest request) {
        // 检查商品名称是否重复
        if (productRepository.existsByName(request.getName())) {
            throw new BizException("P0001", "商品名称已存在");
        }
        
        Product product = productAssembler.toEntity(request);
        product.setStatus(true); // 默认上架
        product.setStock(0);     // 默认库存为0
        
        return productRepository.save(product);
    }
    
    public Product updateProduct(Long id, ProductRequest request) {
        Product product = getProduct(id);
        
        // 检查名称是否重复（排除自己）
        if (!product.getName().equals(request.getName()) 
                && productRepository.existsByName(request.getName())) {
            throw new BizException("P0001", "商品名称已存在");
        }
        
        productAssembler.updateEntity(product, request);
        return productRepository.save(product);
    }
    
    @Transactional(readOnly = true)
    public Product getProduct(Long id) {
        return productRepository.findById(id)
                .orElseThrow(() -> new BizException("P0002", "商品不存在"));
    }
    
    @Transactional(readOnly = true)
    public PageResult<Product> getProducts(ProductQuery query) {
        Pageable pageable = PageRequest.of(
                query.getPage() - 1, 
                query.getSize(),
                Sort.by(Sort.Direction.DESC, "createTime"));
                
        Page<Product> page = productRepository.findByConditions(
                query.getCategory(), 
                query.getKeyword(), 
                query.getStatus(),
                pageable);
                
        return new PageResult<>(page.getContent(), page.getTotalElements());
    }
    
    public void updateStock(Long id, Integer quantity) {
        Product product = getProduct(id);
        
        if (quantity < 0) {
            throw new BizException("P0003", "库存数量不能为负数");
        }
        
        product.setStock(quantity);
        productRepository.save(product);
    }
    
    public void updateStatus(Long id, Boolean status) {
        Product product = getProduct(id);
        product.setStatus(status);
        productRepository.save(product);
    }
    
    /**
     * 扣减库存（用于下单）
     */
    public boolean deductStock(Long productId, Integer quantity) {
        int updated = productRepository.deductStock(productId, quantity);
        return updated > 0;
    }
    
    /**
     * 恢复库存（用于取消订单）
     */
    public void restoreStock(Long productId, Integer quantity) {
        productRepository.restoreStock(productId, quantity);
    }
}
```

### 9.2 用户注册与认证模块 (User Registration & Authentication)

#### 9.2.1 用户领域模型

```java
// domain/model/User.java
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true, length = 50)
    private String username;                // 用户名
    
    @Column(nullable = false, unique = true, length = 100)
    private String email;                    // 邮箱
    
    @Column(nullable = false, length = 100)
    private String password;                 // 密码（加密后）
    
    @Column(length = 20)
    private String phone;                    // 手机号
    
    @Column(length = 100)
    private String nickname;                 // 昵称
    
    @Column(length = 500)
    private String avatar;                   // 头像URL
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserStatus status;               // 用户状态
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserRole role;                   // 用户角色
    
    @CreatedDate
    private LocalDateTime createTime;
    
    @LastModifiedDate
    private LocalDateTime updateTime;
    
    @Version
    private Long version;
}

// 用户状态枚举
public enum UserStatus {
    ACTIVE,      // 活跃
    INACTIVE,    // 非活跃
    BANNED       // 封禁
}

// 用户角色枚举
public enum UserRole {
    USER,        // 普通用户
    ADMIN,       // 管理员
    VIP          // VIP用户
}
```

#### 9.2.2 用户注册API

```java
// api/controller/AuthController.java
@RestController
@RequestMapping("/api/auth")
@Tag(name = "用户认证", description = "用户注册、登录相关接口")
@Validated
public class AuthController {
    
    @Autowired
    private AuthService authService;
    
    @PostMapping("/register")
    @Operation(summary = "用户注册", description = "新用户注册")
    public Result<UserResponse> register(
            @Valid @RequestBody RegisterRequest request) {
        User user = authService.register(request);
        return Result.success(UserResponse.from(user));
    }
    
    @PostMapping("/login")
    @Operation(summary = "用户登录", description = "用户登录获取token")
    public Result<LoginResponse> login(
            @Valid @RequestBody LoginRequest request) {
        LoginResponse response = authService.login(request);
        return Result.success(response);
    }
    
    @PostMapping("/refresh")
    @Operation(summary = "刷新token", description = "使用refresh token刷新access token")
    public Result<TokenResponse> refresh(
            @RequestBody RefreshTokenRequest request) {
        TokenResponse response = authService.refreshToken(request.getRefreshToken());
        return Result.success(response);
    }
    
    @PostMapping("/logout")
    @Operation(summary = "用户登出", description = "用户登出")
    @PreAuthorize("isAuthenticated()")
    public Result<Void> logout() {
        authService.logout();
        return Result.success();
    }
    
    @GetMapping("/me")
    @Operation(summary = "获取当前用户信息", description = "获取当前登录用户的信息")
    @PreAuthorize("isAuthenticated()")
    public Result<UserResponse> getCurrentUser() {
        User user = authService.getCurrentUser();
        return Result.success(UserResponse.from(user));
    }
}
```

#### 9.2.3 认证服务实现

```java
// app/service/AuthService.java
@Service
@Transactional
public class AuthService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Autowired
    private JwtTokenProvider jwtTokenProvider;
    
    @Autowired
    private AuthenticationManager authenticationManager;
    
    public User register(RegisterRequest request) {
        // 检查用户名是否已存在
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new BizException("A0001", "用户名已存在");
        }
        
        // 检查邮箱是否已存在
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new BizException("A0002", "邮箱已存在");
        }
        
        // 创建用户
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        user.setPhone(request.getPhone());
        user.setNickname(request.getNickname());
        user.setStatus(UserStatus.ACTIVE);
        user.setRole(UserRole.USER);
        
        return userRepository.save(user);
    }
    
    public LoginResponse login(LoginRequest request) {
        // 认证用户
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                        request.getUsername(), 
                        request.getPassword()
                )
        );
        
        SecurityContextHolder.getContext().setAuthentication(authentication);
        User user = (User) authentication.getPrincipal();
        
        // 生成token
        String accessToken = jwtTokenProvider.generateAccessToken(user);
        String refreshToken = jwtTokenProvider.generateRefreshToken(user);
        
        return new LoginResponse(accessToken, refreshToken, UserResponse.from(user));
    }
    
    public TokenResponse refreshToken(String refreshToken) {
        if (!jwtTokenProvider.validateRefreshToken(refreshToken)) {
            throw new BizException("A0003", "无效的refresh token");
        }
        
        String username = jwtTokenProvider.getUsernameFromRefreshToken(refreshToken);
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new BizException("A0004", "用户不存在"));
        
        String newAccessToken = jwtTokenProvider.generateAccessToken(user);
        return new TokenResponse(newAccessToken);
    }
    
    public void logout() {
        SecurityContextHolder.clearContext();
        // 可以在这里将token加入黑名单
    }
    
    @Transactional(readOnly = true)
    public User getCurrentUser() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication == null || !authentication.isAuthenticated()) {
            throw new BizException("A0005", "用户未登录");
        }
        
        String username = authentication.getName();
        return userRepository.findByUsername(username)
                .orElseThrow(() -> new BizException("A0004", "用户不存在"));
    }
}
```

### 9.3 用户下单模块 (Order Placement)

#### 9.3.1 订单领域模型

```java
// domain/model/Order.java
@Entity
@Table(name = "orders")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true, length = 32)
    private String orderNo;                 // 订单号
    
    @Column(nullable = false)
    private Long userId;                    // 用户ID
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal totalAmount;         // 订单总金额
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal discountAmount;      // 优惠金额
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal actualAmount;       // 实际支付金额
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private OrderStatus status;             // 订单状态
    
    @Column(length = 200)
    private String receiverName;             // 收货人姓名
    
    @Column(length = 500)
    private String receiverAddress;          // 收货地址
    
    @Column(length = 20)
    private String receiverPhone;            // 收货人电话
    
    @Column(length = 100)
    private String remark;                   // 订单备注
    
    @CreatedDate
    private LocalDateTime createTime;
    
    @LastModifiedDate
    private LocalDateTime updateTime;
    
    @Version
    private Long version;
    
    // 关联订单项
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderItem> items;
}

// domain/model/OrderItem.java
@Entity
@Table(name = "order_items")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false)
    private Long orderId;                   // 订单ID
    
    @Column(nullable = false)
    private Long productId;                  // 商品ID
    
    @Column(nullable = false, length = 100)
    private String productName;              // 商品名称（冗余字段）
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal price;               // 商品单价
    
    @Column(nullable = false)
    private Integer quantity;                // 购买数量
    
    @Column(nullable = false, precision = 10, scale = 2)
    private BigDecimal totalPrice;          // 小计金额
    
    @CreatedDate
    private LocalDateTime createTime;
    
    @Version
    private Long version;
}

// 订单状态枚举
public enum OrderStatus {
    PENDING_PAYMENT,    // 待支付
    PAID,              // 已支付
    SHIPPED,           // 已发货
    DELIVERED,         // 已送达
    COMPLETED,         // 已完成
    CANCELLED          // 已取消
}
```

#### 9.3.2 下单API实现

```java
// api/controller/OrderController.java
@RestController
@RequestMapping("/api/orders")
@Tag(name = "订单管理", description = "订单相关的接口")
@Validated
public class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    @PostMapping
    @Operation(summary = "创建订单", description = "用户下单创建订单")
    @PreAuthorize("hasRole('USER')")
    @Idempotent(expireTime = 30, info = "创建订单")
    public Result<OrderResponse> createOrder(
            @Valid @RequestBody CreateOrderRequest request) {
        Order order = orderService.createOrder(request);
        return Result.success(OrderResponse.from(order));
    }
    
    @GetMapping("/{id}")
    @Operation(summary = "查询订单", description = "根据ID查询订单详情")
    @PreAuthorize("hasRole('USER')")
    public Result<OrderResponse> getOrder(@PathVariable Long id) {
        Order order = orderService.getOrder(id);
        return Result.success(OrderResponse.from(order));
    }
    
    @GetMapping
    @Operation(summary = "订单列表", description = "分页查询用户订单列表")
    @PreAuthorize("hasRole('USER')")
    public Result<PageResult<OrderResponse>> getOrders(
            @RequestParam(defaultValue = "1") Integer page,
            @RequestParam(defaultValue = "10") Integer size,
            @RequestParam(required = false) OrderStatus status) {
        
        OrderQuery query = OrderQuery.builder()
                .page(page)
                .size(size)
                .status(status)
                .userId(UserContext.getCurrentUserId())
                .build();
                
        PageResult<Order> result = orderService.getOrders(query);
        List<OrderResponse> responses = result.getData().stream()
                .map(OrderResponse::from)
                .collect(Collectors.toList());
                
        return Result.success(new PageResult<>(responses, result.getTotal()));
    }
    
    @PostMapping("/{id}/cancel")
    @Operation(summary = "取消订单", description = "取消未支付的订单")
    @PreAuthorize("hasRole('USER')")
    public Result<Void> cancelOrder(@PathVariable Long id) {
        orderService.cancelOrder(id);
        return Result.success();
    }
    
    @PostMapping("/{id}/pay")
    @Operation(summary = "支付订单", description = "支付订单")
    @PreAuthorize("hasRole('USER')")
    public Result<PaymentResponse> payOrder(
            @PathVariable Long id,
            @Valid @RequestBody PaymentRequest request) {
        PaymentResponse response = orderService.payOrder(id, request);
        return Result.success(response);
    }
    
    @PostMapping("/{id}/confirm")
    @Operation(summary = "确认收货", description = "确认收货")
    @PreAuthorize("hasRole('USER')")
    public Result<Void> confirmOrder(@PathVariable Long id) {
        orderService.confirmOrder(id);
        return Result.success();
    }
}
```

#### 9.3.3 下单服务实现

```java
// app/service/OrderService.java
@Service
@Transactional
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private OrderItemRepository orderItemRepository;
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private ProductService productService;
    
    @Autowired
    private OrderAssembler orderAssembler;
    
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private DistributedLockService lockService;
    
    public Order createOrder(CreateOrderRequest request) {
        Long userId = UserContext.getCurrentUserId();
        
        // 1. 验证商品库存和价格
        List<OrderItemRequest> itemRequests = request.getItems();
        List<Product> products = validateAndGetProducts(itemRequests);
        
        // 2. 计算订单金额
        BigDecimal totalAmount = calculateTotalAmount(products, itemRequests);
        BigDecimal discountAmount = calculateDiscountAmount(totalAmount, request.getCouponCode());
        BigDecimal actualAmount = totalAmount.subtract(discountAmount);
        
        // 3. 创建订单
        Order order = new Order();
        order.setOrderNo(generateOrderNo());
        order.setUserId(userId);
        order.setTotalAmount(totalAmount);
        order.setDiscountAmount(discountAmount);
        order.setActualAmount(actualAmount);
        order.setStatus(OrderStatus.PENDING_PAYMENT);
        order.setReceiverName(request.getReceiverName());
        order.setReceiverAddress(request.getReceiverAddress());
        order.setReceiverPhone(request.getReceiverPhone());
        order.setRemark(request.getRemark());
        
        order = orderRepository.save(order);
        
        // 4. 创建订单项
        List<OrderItem> orderItems = createOrderItems(order.getId(), products, itemRequests);
        orderItemRepository.saveAll(orderItems);
        
        // 5. 扣减库存（使用分布式锁）
        deductStockWithLock(products, itemRequests);
        
        // 6. 发送订单创建事件
        OrderCreatedEvent event = new OrderCreatedEvent(order, orderItems);
        ApplicationEventPublisher publisher = getApplicationEventPublisher();
        publisher.publishEvent(event);
        
        return order;
    }
    
    @Transactional(readOnly = true)
    public Order getOrder(Long id) {
        Long userId = UserContext.getCurrentUserId();
        return orderRepository.findByIdAndUserId(id, userId)
                .orElseThrow(() -> new BizException("O0001", "订单不存在"));
    }
    
    @Transactional(readOnly = true)
    public PageResult<Order> getOrders(OrderQuery query) {
        Pageable pageable = PageRequest.of(
                query.getPage() - 1, 
                query.getSize(),
                Sort.by(Sort.Direction.DESC, "createTime"));
                
        Page<Order> page = orderRepository.findByConditions(
                query.getUserId(), 
                query.getStatus(),
                pageable);
                
        return new PageResult<>(page.getContent(), page.getTotalElements());
    }
    
    public void cancelOrder(Long id) {
        Order order = getOrder(id);
        
        // 只有待支付的订单才能取消
        if (order.getStatus() != OrderStatus.PENDING_PAYMENT) {
            throw new BizException("O0002", "订单状态不允许取消");
        }
        
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
        
        // 恢复库存
        restoreStock(order);
        
        // 发送订单取消事件
        OrderCancelledEvent event = new OrderCancelledEvent(order);
        getApplicationEventPublisher().publishEvent(event);
    }
    
    public PaymentResponse payOrder(Long id, PaymentRequest request) {
        Order order = getOrder(id);
        
        // 只有待支付的订单才能支付
        if (order.getStatus() != OrderStatus.PENDING_PAYMENT) {
            throw new BizException("O0003", "订单状态不允许支付");
        }
        
        // 调用支付服务
        PaymentResult paymentResult = paymentService.processPayment(
                order.getOrderNo(), 
                order.getActualAmount(), 
                request.getPaymentMethod()
        );
        
        if (paymentResult.isSuccess()) {
            // 更新订单状态
            order.setStatus(OrderStatus.PAID);
            orderRepository.save(order);
            
            // 发送订单支付成功事件
            OrderPaidEvent event = new OrderPaidEvent(order, paymentResult);
            getApplicationEventPublisher().publishEvent(event);
        }
        
        return PaymentResponse.from(paymentResult);
    }
    
    public void confirmOrder(Long id) {
        Order order = getOrder(id);
        
        // 只有已发货的订单才能确认收货
        if (order.getStatus() != OrderStatus.DELIVERED) {
            throw new BizException("O0004", "订单状态不允许确认收货");
        }
        
        order.setStatus(OrderStatus.COMPLETED);
        orderRepository.save(order);
        
        // 发送订单完成事件
        OrderCompletedEvent event = new OrderCompletedEvent(order);
        getApplicationEventPublisher().publishEvent(event);
    }
    
    // 私有方法
    private List<Product> validateAndGetProducts(List<OrderItemRequest> itemRequests) {
        List<Long> productIds = itemRequests.stream()
                .map(OrderItemRequest::getProductId)
                .collect(Collectors.toList());
                
        List<Product> products = productRepository.findAllById(productIds);
        
        if (products.size() != productIds.size()) {
            throw new BizException("O0005", "商品不存在");
        }
        
        // 检查库存
        for (Product product : products) {
            OrderItemRequest itemRequest = itemRequests.stream()
                    .filter(item -> item.getProductId().equals(product.getId()))
                    .findFirst()
                    .orElseThrow(() -> new BizException("O0006", "商品信息错误"));
                    
            if (product.getStock() < itemRequest.getQuantity()) {
                throw new BizException("O0007", "商品库存不足");
            }
        }
        
        return products;
    }
    
    private BigDecimal calculateTotalAmount(List<Product> products, List<OrderItemRequest> itemRequests) {
        return products.stream()
                .map(product -> {
                    OrderItemRequest itemRequest = itemRequests.stream()
                            .filter(item -> item.getProductId().equals(product.getId()))
                            .findFirst()
                            .orElseThrow(() -> new BizException("O0006", "商品信息错误"));
                    return product.getPrice().multiply(BigDecimal.valueOf(itemRequest.getQuantity()));
                })
                .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    private BigDecimal calculateDiscountAmount(BigDecimal totalAmount, String couponCode) {
        // 这里可以实现优惠券逻辑
        return BigDecimal.ZERO;
    }
    
    private String generateOrderNo() {
        return "ORD" + System.currentTimeMillis() + RandomUtils.nextInt(100, 999);
    }
    
    private List<OrderItem> createOrderItems(Long orderId, List<Product> products, List<OrderItemRequest> itemRequests) {
        return products.stream()
                .map(product -> {
                    OrderItemRequest itemRequest = itemRequests.stream()
                            .filter(item -> item.getProductId().equals(product.getId()))
                            .findFirst()
                            .orElseThrow(() -> new BizException("O0006", "商品信息错误"));
                            
                    OrderItem item = new OrderItem();
                    item.setOrderId(orderId);
                    item.setProductId(product.getId());
                    item.setProductName(product.getName());
                    item.setPrice(product.getPrice());
                    item.setQuantity(itemRequest.getQuantity());
                    item.setTotalPrice(product.getPrice().multiply(BigDecimal.valueOf(itemRequest.getQuantity())));
                    
                    return item;
                })
                .collect(Collectors.toList());
    }
    
    private void deductStockWithLock(List<Product> products, List<OrderItemRequest> itemRequests) {
        for (Product product : products) {
            OrderItemRequest itemRequest = itemRequests.stream()
                    .filter(item -> item.getProductId().equals(product.getId()))
                    .findFirst()
                    .orElseThrow(() -> new BizException("O0006", "商品信息错误"));
                    
            String lockKey = "lock:product:stock:" + product.getId();
            lockService.executeWithLock(lockKey, 5, TimeUnit.SECONDS, () -> {
                boolean success = productService.deductStock(product.getId(), itemRequest.getQuantity());
                if (!success) {
                    throw new BizException("O0007", "商品库存不足");
                }
            });
        }
    }
    
    private void restoreStock(Order order) {
        List<OrderItem> orderItems = orderItemRepository.findByOrderId(order.getId());
        for (OrderItem item : orderItems) {
            productService.restoreStock(item.getProductId(), item.getQuantity());
        }
    }
}
```

### 9.4 事件驱动架构实现

#### 9.4.1 订单事件定义

```java
// domain/event/OrderCreatedEvent.java
@Data
@AllArgsConstructor
public class OrderCreatedEvent {
    private final Order order;
    private final List<OrderItem> orderItems;
    private final LocalDateTime timestamp;
    
    public OrderCreatedEvent(Order order, List<OrderItem> orderItems) {
        this.order = order;
        this.orderItems = orderItems;
        this.timestamp = LocalDateTime.now();
    }
}

// domain/event/OrderPaidEvent.java
@Data
@AllArgsConstructor
public class OrderPaidEvent {
    private final Order order;
    private final PaymentResult paymentResult;
    private final LocalDateTime timestamp;
    
    public OrderPaidEvent(Order order, PaymentResult paymentResult) {
        this.order = order;
        this.paymentResult = paymentResult;
        this.timestamp = LocalDateTime.now();
    }
}
```

#### 9.4.2 事件处理器

```java
// app/event/OrderEventHandler.java
@Component
@Slf4j
public class OrderEventHandler {
    
    @Autowired
    private NotificationService notificationService;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private AnalyticsService analyticsService;
    
    @EventListener
    @Async
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("处理订单创建事件: {}", event.getOrder().getOrderNo());
        
        // 发送订单创建通知
        notificationService.sendOrderCreatedNotification(event.getOrder());
        
        // 更新库存统计
        inventoryService.updateInventoryStatistics(event.getOrderItems());
        
        // 更新销售统计
        analyticsService.updateSalesStatistics(event.getOrder());
    }
    
    @EventListener
    @Async
    public void handleOrderPaid(OrderPaidEvent event) {
        log.info("处理订单支付事件: {}", event.getOrder().getOrderNo());
        
        // 发送支付成功通知
        notificationService.sendPaymentSuccessNotification(event.getOrder());
        
        // 触发发货流程
        // 这里可以调用物流服务
    }
    
    @EventListener
    @Async
    public void handleOrderCancelled(OrderCancelledEvent event) {
        log.info("处理订单取消事件: {}", event.getOrder().getOrderNo());
        
        // 发送订单取消通知
        notificationService.sendOrderCancelledNotification(event.getOrder());
    }
    
    @EventListener
    @Async
    public void handleOrderCompleted(OrderCompletedEvent event) {
        log.info("处理订单完成事件: {}", event.getOrder().getOrderNo());
        
        // 发送订单完成通知
        notificationService.sendOrderCompletedNotification(event.getOrder());
        
        // 更新用户积分
        // 这里可以调用积分服务
    }
}
```

-----

## 10. 前端技术实现 (Frontend Implementation)

### 10.1 技术栈选择与版本兼容性

#### 10.1.1 推荐前端技术栈

| 技术类别 | 推荐技术 | 版本 | 说明 |
|---------|---------|------|------|
| **核心框架** | React | 18.2.0+ | 组件化开发，生态丰富，企业级首选 |
| **状态管理** | Redux Toolkit | 1.9.0+ | 现代化状态管理，简化Redux使用 |
| **路由管理** | React Router | 6.8.0+ | 声明式路由，支持懒加载 |
| **UI组件库** | Ant Design | 5.0.0+ | 企业级UI设计语言，组件丰富 |
| **样式方案** | Styled Components | 5.3.0+ | CSS-in-JS，主题定制灵活 |
| **构建工具** | Vite | 4.3.0+ | 快速构建，HMR体验优秀 |
| **TypeScript** | TypeScript | 5.0.0+ | 类型安全，提高代码质量 |
| **HTTP客户端** | Axios | 1.4.0+ | 请求拦截，响应处理，错误统一 |
| **表单处理** | React Hook Form | 7.44.0+ | 高性能表单，验证简单 |

#### 9.1.2 项目初始化配置

```bash
# 使用 Vite 创建 React + TypeScript 项目
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install

# 安装核心依赖
npm install @reduxjs/toolkit react-redux react-router-dom
npm install antd styled-components
npm install axios react-hook-form @hookform/resolvers yup
npm install @types/styled-components

# 安装开发依赖
npm install -D @types/node eslint prettier husky lint-staged
```

### 9.2 项目目录结构

```
frontend/
├── public/                 // 静态资源
│   ├── index.html
│   └── favicon.ico
├── src/
│   ├── components/         // 通用组件
│   │   ├── Layout/         // 布局组件
│   │   ├── Table/          // 表格组件
│   │   └── Form/           // 表单组件
│   ├── pages/              // 页面组件
│   │   ├── Login/          // 登录页
│   │   ├── Dashboard/      // 仪表板
│   │   └── Order/          // 订单管理
│   ├── hooks/              // 自定义Hooks
│   │   ├── useAuth.ts      // 认证Hook
│   │   └── useApi.ts       // API请求Hook
│   ├── store/              // Redux状态管理
│   │   ├── index.ts        // Store配置
│   │   ├── slices/         // Redux Slices
│   │   └── api/            // RTK Query API
│   ├── services/           // API服务
│   │   ├── api.ts          // Axios配置
│   │   ├── auth.ts         // 认证API
│   │   └── order.ts        // 订单API
│   ├── types/              // TypeScript类型定义
│   │   ├── api.ts          // API响应类型
│   │   └── common.ts       // 通用类型
│   ├── utils/              // 工具函数
│   │   ├── request.ts      // 请求工具
│   │   └── storage.ts      // 本地存储
│   ├── styles/             // 全局样式
│   │   ├── theme.ts        // Ant Design主题
│   │   └── global.css      // 全局样式
│   ├── App.tsx             // 根组件
│   ├── main.tsx            // 入口文件
│   └── vite-env.d.ts       // Vite类型声明
├── package.json
├── vite.config.ts          // Vite配置
├── tsconfig.json           // TypeScript配置
└── tailwind.config.js      // Tailwind CSS配置（可选）
```

### 9.3 响应式设计与自适应布局

#### 9.3.1 CSS Grid + Flexbox 布局方案

```typescript
// src/components/Layout/MainLayout.tsx
import styled from 'styled-components';

const LayoutContainer = styled.div`
  display: grid;
  grid-template-areas: 
    "sidebar header"
    "sidebar main";
  grid-template-columns: 240px 1fr;
  grid-template-rows: 64px 1fr;
  height: 100vh;
  
  /* 响应式断点 */
  @media (max-width: 768px) {
    grid-template-areas: 
      "header"
      "main";
    grid-template-columns: 1fr;
    grid-template-rows: 64px 1fr;
  }
`;

const Sidebar = styled.aside`
  grid-area: sidebar;
  background: #001529;
  color: white;
  
  @media (max-width: 768px) {
    display: none;
  }
`;

const Header = styled.header`
  grid-area: header;
  background: white;
  border-bottom: 1px solid #f0f0f0;
  display: flex;
  align-items: center;
  padding: 0 24px;
`;

const Main = styled.main`
  grid-area: main;
  padding: 24px;
  overflow-y: auto;
  
  @media (max-width: 768px) {
    padding: 16px;
  }
`;
```

#### 9.3.2 Ant Design 响应式栅格系统

```typescript
// src/pages/Dashboard/Dashboard.tsx
import { Row, Col, Card, Statistic } from 'antd';

const Dashboard = () => {
  return (
    <div>
      {/* 统计卡片 - 响应式布局 */}
      <Row gutter={[16, 16]}>
        <Col xs={24} sm={12} lg={6}>
          <Card>
            <Statistic
              title="总订单数"
              value={1128}
              precision={0}
            />
          </Card>
        </Col>
        <Col xs={24} sm={12} lg={6}>
          <Card>
            <Statistic
              title="今日订单"
              value={93}
              precision={0}
            />
          </Card>
        </Col>
        <Col xs={24} sm={12} lg={6}>
          <Card>
            <Statistic
              title="总用户数"
              value={4562}
              precision={0}
            />
          </Card>
        </Col>
        <Col xs={24} sm={12} lg={6}>
          <Card>
            <Statistic
              title="活跃用户"
              value={234}
              precision={0}
            />
          </Card>
        </Col>
      </Row>
      
      {/* 图表区域 */}
      <Row gutter={[16, 16]} style={{ marginTop: 16 }}>
        <Col xs={24} lg={16}>
          <Card title="订单趋势">
            {/* 图表组件 */}
          </Card>
        </Col>
        <Col xs={24} lg={8}>
          <Card title="商品分类">
            {/* 饼图组件 */}
          </Card>
        </Col>
      </Row>
    </div>
  );
};
```

### 9.4 主题系统与设计规范

#### 9.4.1 Ant Design 主题定制

```typescript
// src/styles/theme.ts
import { theme } from 'antd';

export const customTheme = {
  algorithm: theme.defaultAlgorithm,
  token: {
    // 主色调
    colorPrimary: '#1890ff',
    colorSuccess: '#52c41a',
    colorWarning: '#faad14',
    colorError: '#ff4d4f',
    
    // 字体
    fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial',
    fontSize: 14,
    
    // 圆角
    borderRadius: 6,
    
    // 间距
    padding: 16,
    margin: 16,
  },
  components: {
    Button: {
      borderRadius: 4,
      controlHeight: 36,
    },
    Card: {
      borderRadius: 8,
      boxShadow: '0 2px 8px rgba(0, 0, 0, 0.06)',
    },
    Table: {
      borderRadius: 8,
      headerBg: '#fafafa',
    },
  },
};
```

#### 9.4.2 暗色主题支持

```typescript
// src/hooks/useTheme.ts
import { useState, useEffect } from 'react';
import { theme } from 'antd';

export const useTheme = () => {
  const [isDark, setIsDark] = useState(false);

  useEffect(() => {
    // 从本地存储读取主题设置
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme === 'dark') {
      setIsDark(true);
      document.body.setAttribute('data-theme', 'dark');
    }
  }, []);

  const toggleTheme = () => {
    const newTheme = !isDark;
    setIsDark(newTheme);
    localStorage.setItem('theme', newTheme ? 'dark' : 'light');
    document.body.setAttribute('data-theme', newTheme ? 'dark' : 'light');
  };

  const themeConfig = {
    algorithm: isDark ? theme.darkAlgorithm : theme.defaultAlgorithm,
    ...customTheme,
  };

  return { isDark, toggleTheme, themeConfig };
};
```

### 9.5 状态管理与数据流

#### 9.5.1 Redux Toolkit 配置

```typescript
// src/store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import { setupListeners } from '@reduxjs/toolkit/query';
import authSlice from './slices/authSlice';
import { orderApi } from './api/orderApi';

export const store = configureStore({
  reducer: {
    auth: authSlice,
    orderApi: orderApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST', 'persist/REHYDRATE'],
      },
    }).concat(orderApi.middleware),
});

setupListeners(store.dispatch);

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

#### 9.5.2 RTK Query API 集成

```typescript
// src/store/api/orderApi.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import type { RootState } from '../index';

export interface Order {
  id: number;
  userId: number;
  status: string;
  totalAmount: number;
  createTime: string;
}

export interface CreateOrderRequest {
  userId: number;
  items: OrderItem[];
  address: string;
}

export const orderApi = createApi({
  reducerPath: 'orderApi',
  baseQuery: fetchBaseQuery({
    baseUrl: '/api/orders',
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ['Order'],
  endpoints: (builder) => ({
    getOrders: builder.query<Order[], { page: number; size: number }>({
      query: ({ page, size }) => `?page=${page}&size=${size}`,
      providesTags: ['Order'],
    }),
    getOrder: builder.query<Order, number>({
      query: (id) => `/${id}`,
      providesTags: (result, error, id) => [{ type: 'Order', id }],
    }),
    createOrder: builder.mutation<Order, CreateOrderRequest>({
      query: (order) => ({
        url: '',
        method: 'POST',
        body: order,
      }),
      invalidatesTags: ['Order'],
    }),
  }),
});

export const {
  useGetOrdersQuery,
  useGetOrderQuery,
  useCreateOrderMutation,
} = orderApi;
```

### 9.6 路由与权限控制

#### 9.6.1 路由配置

```typescript
// src/App.tsx
import React from 'react';
import { BrowserRouter as Router, Routes, Route, Navigate } from 'react-router-dom';
import { useSelector } from 'react-redux';
import { RootState } from './store';
import MainLayout from './components/Layout/MainLayout';
import Login from './pages/Login';
import Dashboard from './pages/Dashboard';
import OrderList from './pages/Order/OrderList';
import OrderDetail from './pages/Order/OrderDetail';
import ProtectedRoute from './components/Auth/ProtectedRoute';

const App: React.FC = () => {
  const { isAuthenticated } = useSelector((state: RootState) => state.auth);

  return (
    <Router>
      <Routes>
        <Route path="/login" element={<Login />} />
        <Route
          path="/*"
          element={
            <ProtectedRoute isAuthenticated={isAuthenticated}>
              <MainLayout>
                <Routes>
                  <Route path="/" element={<Navigate to="/dashboard" replace />} />
                  <Route path="/dashboard" element={<Dashboard />} />
                  <Route path="/orders" element={<OrderList />} />
                  <Route path="/orders/:id" element={<OrderDetail />} />
                </Routes>
              </MainLayout>
            </ProtectedRoute>
          }
        />
      </Routes>
    </Router>
  );
};

export default App;
```

#### 9.6.2 权限控制组件

```typescript
// src/components/Auth/ProtectedRoute.tsx
import React from 'react';
import { Navigate, useLocation } from 'react-router-dom';
import { useSelector } from 'react-redux';
import { RootState } from '../../store';

interface ProtectedRouteProps {
  children: React.ReactNode;
  isAuthenticated: boolean;
  requiredRole?: string;
}

const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  isAuthenticated,
  requiredRole,
}) => {
  const location = useLocation();
  const { user } = useSelector((state: RootState) => state.auth);

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (requiredRole && user?.role !== requiredRole) {
    return <Navigate to="/unauthorized" replace />;
  }

  return <>{children}</>;
};

export default ProtectedRoute;
```

### 9.7 性能优化策略

#### 9.7.1 代码分割与懒加载

```typescript
// src/pages/LazyPages.ts
import { lazy } from 'react';

export const Dashboard = lazy(() => import('./Dashboard/Dashboard'));
export const OrderList = lazy(() => import('./Order/OrderList'));
export const OrderDetail = lazy(() => import('./Order/OrderDetail'));
export const UserManagement = lazy(() => import('./User/UserManagement'));

// 使用 Suspense 包装
import { Suspense } from 'react';
import { Spin } from 'antd';

const LazyWrapper: React.FC<{ children: React.ReactNode }> = ({ children }) => (
  <Suspense fallback={<Spin size="large" style={{ display: 'block', margin: '20% auto' }} />}>
    {children}
  </Suspense>
);
```

#### 9.7.2 虚拟滚动优化

```typescript
// src/components/Table/VirtualTable.tsx
import React from 'react';
import { Table } from 'antd';
import { FixedSizeList as List } from 'react-window';

interface VirtualTableProps {
  data: any[];
  columns: any[];
  height: number;
}

const VirtualTable: React.FC<VirtualTableProps> = ({ data, columns, height }) => {
  const Row = ({ index, style }: any) => (
    <div style={style}>
      <Table
        dataSource={[data[index]]}
        columns={columns}
        pagination={false}
        showHeader={false}
        size="small"
      />
    </div>
  );

  return (
    <List
      height={height}
      itemCount={data.length}
      itemSize={55}
      width="100%"
    >
      {Row}
    </List>
  );
};

export default VirtualTable;
```

### 9.8 错误处理与用户体验

#### 9.8.1 全局错误边界

```typescript
// src/components/ErrorBoundary/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { Result, Button } from 'antd';

interface Props {
  children: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

class ErrorBoundary extends Component<Props, State> {
  public state: State = {
    hasError: false,
  };

  public static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  public componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Uncaught error:', error, errorInfo);
    // 发送错误报告到监控服务
  }

  public render() {
    if (this.state.hasError) {
      return (
        <Result
          status="500"
          title="500"
          subTitle="抱歉，页面出现了错误。"
          extra={
            <Button type="primary" onClick={() => window.location.reload()}>
              刷新页面
            </Button>
          }
        />
      );
    }

    return this.props.children;
  }
}

export default ErrorBoundary;
```

#### 9.8.2 网络请求错误处理

```typescript
// src/utils/request.ts
import axios, { AxiosRequestConfig, AxiosResponse, AxiosError } from 'axios';
import { message } from 'antd';

// 请求拦截器
axios.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// 响应拦截器
axios.interceptors.response.use(
  (response: AxiosResponse) => {
    const { code, message: msg, data } = response.data;
    
    if (code === '00000') {
      return data;
    } else {
      message.error(msg || '请求失败');
      return Promise.reject(new Error(msg));
    }
  },
  (error: AxiosError) => {
    if (error.response?.status === 401) {
      message.error('登录已过期，请重新登录');
      localStorage.removeItem('token');
      window.location.href = '/login';
    } else if (error.response?.status === 403) {
      message.error('没有权限访问该资源');
    } else if (error.response?.status >= 500) {
      message.error('服务器错误，请稍后重试');
    } else {
      message.error('网络错误，请检查网络连接');
    }
    return Promise.reject(error);
  }
);
```

### 9.9 构建与部署

#### 9.9.1 Vite 配置优化

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@pages': resolve(__dirname, 'src/pages'),
      '@utils': resolve(__dirname, 'src/utils'),
    },
  },
  build: {
    // 代码分割优化
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          antd: ['antd'],
          router: ['react-router-dom'],
          redux: ['@reduxjs/toolkit', 'react-redux'],
        },
      },
    },
    // 压缩配置
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
});
```

#### 9.9.2 Dockerfile 多阶段构建

```dockerfile
# 构建阶段
FROM node:18-alpine AS builder

WORKDIR /app

# 复制package文件
COPY package*.json ./
RUN npm ci --only=production

# 复制源代码
COPY . .

# 构建应用
RUN npm run build

# 生产阶段
FROM nginx:alpine

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制nginx配置
COPY nginx.conf /etc/nginx/nginx.conf

# 暴露端口
EXPOSE 80

# 启动nginx
CMD ["nginx", "-g", "daemon off;"]
```

### 9.10 前端监控与分析

#### 9.10.1 性能监控

```typescript
// src/utils/performance.ts
export const reportWebVitals = (onPerfEntry?: (metric: any) => void) => {
  if (onPerfEntry && onPerfEntry instanceof Function) {
    import('web-vitals').then(({ getCLS, getFID, getFCP, getLCP, getTTFB }) => {
      getCLS(onPerfEntry);
      getFID(onPerfEntry);
      getFCP(onPerfEntry);
      getLCP(onPerfEntry);
      getTTFB(onPerfEntry);
    });
  }
};

// 使用示例
reportWebVitals((metric) => {
  console.log(metric);
  // 发送到监控服务
});
```

#### 9.10.2 错误监控

```typescript
// src/utils/errorTracking.ts
export class ErrorTracker {
  static track(error: Error, context?: any) {
    const errorInfo = {
      message: error.message,
      stack: error.stack,
      timestamp: new Date().toISOString(),
      url: window.location.href,
      userAgent: navigator.userAgent,
      context,
    };
    
    // 发送到错误监控服务
    console.error('Tracked Error:', errorInfo);
    
    // 可集成第三方监控服务如 Sentry
    // Sentry.captureException(error, { extra: context });
  }
  
  static trackApiError(url: string, error: any) {
    this.track(new Error(`API Error: ${url}`), { url, error });
  }
}
```

-----

## 10. 总结与检查点

在实施以上代码时，请检查：

### 10.1 后端检查点
1. **异常处理:** 是否每个 `catch` 块都记录了日志？
2. **超时:** 是否所有的 HTTP 请求、DB 查询、Redis 访问都设置了明确的 `ConnectTimeout` 和 `ReadTimeout`？
3. **资源释放:** 所有的 `InputStream`, `Lock`, `ThreadLocal` 是否都在 `finally` 块中进行了清理？

### 10.2 前端检查点
1. **响应式设计:** 是否在移动端、平板、桌面端都有良好的显示效果？
2. **性能优化:** 是否实现了代码分割、懒加载、虚拟滚动等优化策略？
3. **用户体验:** 是否有适当的加载状态、错误处理、用户反馈？
4. **类型安全:** TypeScript 类型定义是否完整，避免运行时类型错误？
5. **安全性:** 是否实现了适当的权限控制、XSS防护、CSRF防护？

### 10.3 集成检查点
1. **API契约:** 前后端API接口定义是否一致，是否有完整的文档？
2. **认证授权:** JWT令牌的传递、验证、刷新机制是否完整？
3. **错误处理:** 前后端错误码、错误信息是否统一？
4. **监控告警:** 是否建立了完整的监控体系，包括性能监控、错误监控、业务监控？

