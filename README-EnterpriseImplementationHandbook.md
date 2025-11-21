Enterprise Implementation Handbook
=====

这是一个关于**企业级应用具体技术实现**的详细指南。

为了确保落地性，本实战文档假定技术栈为目前企业级最主流的 **Java (Spring Boot 3.x + Spring Cloud)** 配合 **Cloud Native**
组件。如果您使用的是 Go 或 Python，核心的设计模式（如中间件逻辑、分层结构）是完全通用的。

-----

# 企业级应用技术实现手册 (Enterprise Implementation Handbook)

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

### 4.1 MyBatis Plus / JPA 配置优化

* **乐观锁插件:** 在 `update` 操作时自动带上 `version` 字段，防止并发覆盖。
* **自动填充:** 统一处理 `create_time`, `update_time`, `created_by`。

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

## 7\. 基础设施即代码 (Infrastructure / Dockerfile)

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

## 8\. 总结与检查点

在实施以上代码时，请检查：

1. **异常处理:** 是否每个 `catch` 块都记录了日志？
2. **超时:** 是否所有的 HTTP 请求、DB 查询、Redis 访问都设置了明确的 `ConnectTimeout` 和 `ReadTimeout`？
3. **资源释放:** 所有的 `InputStream`, `Lock`, `ThreadLocal` 是否都在 `finally` 块中进行了清理？

