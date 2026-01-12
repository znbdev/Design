Enterprise Application Engineering Guide
=====

这是一个为您精心整理的**通用高质量企业级应用工程设计思路总结**。

这份文档采用 Markdown 格式，结构化地涵盖了从宏观架构模式到微观工程实践的核心要素。它不仅关注“怎么做（How）”，更关注“为什么做（Why）”，旨在为架构师和技术负责人提供一个可裁剪的决策框架。

---

# 企业级应用工程设计通用指南 (Enterprise Application Engineering Guide)

## 1. 核心设计原则 (Core Design Principles)

在深入技术细节之前，必须确立指导整个生命周期的核心原则。

* **简单性 (Simplicity):** 避免过度设计。如无必要，勿增实体（Occam's Razor）。首选“模块化单体”而非盲目上微服务。
* **可演进性 (Evolvability):** 架构不是静态的。设计应允许技术栈和业务逻辑的独立演进。
* **关注点分离 (Separation of Concerns):** 严格遵循高内聚、低耦合。
* **故障隔离 (Fault Isolation):** 假设系统一定会失败，设计必须包含容错和自愈机制。
* **数据一致性 (Data Consistency):** 在 CAP 理论中明确取舍（通常企业级应用倾向于 CP 或强一致性的 AP 变体）。

---

## 2. 宏观架构模式 (Macro Architecture)

### 2.1 领域驱动设计 (DDD)

企业级应用的核心在于业务复杂度的治理。

* **战略设计:** 划分**限界上下文 (Bounded Context)**，识别核心域、支撑域和通用域。
* **战术设计:** 定义聚合根 (Aggregate Root)、实体 (Entity)、值对象 (Value Object) 和领域服务 (Domain Service)。
* **防腐层 (ACL):** 在对接外部系统或遗留系统时，建立防腐层以保护核心领域模型不受污染。

### 2.2 架构分层 (Layered Architecture)

推荐采用 **整洁架构 (Clean Architecture)** 或 **六边形架构 (Hexagonal Architecture)**：

1. **接入层 (Interface/Adapter):** 处理 HTTP、RPC、MQ 消息。
2. **应用层 (Application):** 编排业务流程，不包含业务逻辑，只做协调。
3. **领域层 (Domain):** 纯粹的业务逻辑，不依赖任何外部框架或数据库。
4. **基础设施层 (Infrastructure):** 数据库实现、缓存、第三方 API 调用。

> **关键点:** 依赖倒置原则 (DIP) —— 外层依赖内层，内层绝不依赖外层。

### 2.3 微服务 vs. 模块化单体

* **起步阶段:** 推荐 **Modular Monolith (模块化单体)**。在同一个仓库和部署单元中，通过包/命名空间严格隔离上下文。
* **扩展阶段:** 当团队规模超过“两个披萨”原则，或模块需独立扩缩容时，拆分为 **Microservices**。

---

## 3. 后端工程实践 (Backend Engineering)

### 3.1 API 设计规范

* **风格:** 优先 RESTful (资源导向)，复杂查询使用 GraphQL，内部高性能通信使用 gRPC (Protobuf)。
* **幂等性 (Idempotency):** 所有的写操作（POST/PUT/PATCH）必须通过 `Idempotency-Key` 保证幂等，防止重放攻击和网络抖动导致的重复处理。
* **版本控制:** 在 URL 或 Header 中明确版本号 (e.g., `/api/v1/...`)，杜绝破坏性变更。

### 3.2 并发与异步

* **消息队列 (MQ):** 核心链路同步，非核心链路异步。使用 Kafka/RabbitMQ/RocketMQ 进行削峰填谷和解耦。
* **事件驱动架构 (EDA):** 领域事件 (Domain Events) 触发下游业务（如：订单创建 -> 触发库存锁定 -> 触发积分发放）。

### 3.3 数据一致性

* **分布式事务:** 尽量避免 XA/2PC。
* **推荐模式:**
    * **TCC (Try-Confirm-Cancel):** 适用于强一致性要求极高的场景（如支付）。
    * **Saga 模式:** 长事务流程，通过编排或协同进行补偿（Compensating Transaction）。
    * **本地消息表 + 最终一致性:** 最稳健的通用方案。

---

## 4. 数据架构与存储 (Data Architecture)

### 4.1 存储选型 (Polyglot Persistence)

* **关系型 (RDBMS):** MySQL/PostgreSQL。存放核心业务数据，强制使用外键约束（但在高并发场景下可在应用层校验），必须支持 ACID。
* **NoSQL:**
    * Redis: 缓存、分布式锁、计数器。
    * MongoDB: 存放文档型、Schema 不固定的数据（如日志、配置、CMS 内容）。
    * Elasticsearch: 全文检索、复杂报表聚合。
* **时序数据库:** InfluxDB/Prometheus，用于监控和物联网数据。

### 4.2 缓存策略

* **缓存模式:** Cache-Aside (旁路缓存) 为主。
* **穿透/击穿/雪崩防护:** 布隆过滤器、互斥锁、随机过期时间。
* **一致性:** 数据库更新后删除缓存（Cache invalidation），而非更新缓存，以避免并发脏读。

---

## 5. 稳定性与高可用 (Stability & High Availability)

### 5.1 流量治理

* **限流 (Rate Limiting):** 网关层 (Nginx/Kong) 和 应用层 (Token Bucket/Leaky Bucket) 双重限流。
* **熔断降级 (Circuit Breaking):** 下游服务不可用时，快速失败，返回默认值或缓存数据，防止级联故障。
* **超时控制:** 每一层调用（DB, Cache, RPC）都必须设定严格的超时时间。

### 5.2 容灾 (Disaster Recovery)

* **多活架构:** 同城双活 (Active-Active) 或 异地多活 (需解决数据延迟同步问题)。
* **数据库备份:** 定期全量备份 + 实时 Binlog 增量备份。定期进行恢复演练。

---

## 6. 安全性设计 (Security)

* **零信任架构 (Zero Trust):** 不信任内网，服务间调用需 mTLS 认证。
* **认证与授权:**
    * 统一身份认证 (IAM) 基于 OAuth2 / OIDC。
    * RBAC (基于角色) + ABAC (基于属性) 的混合权限控制。
* **数据安全:**
    * 传输层：全站 HTTPS (TLS 1.3)。
    * 存储层：敏感数据（PII、密码）加密存储，密钥管理使用 KMS。
* **输入校验:** 永远不要信任前端输入，防止 SQL 注入、XSS、CSRF。

---

## 7. 可观测性 (Observability)

不再仅仅是“监控”，而是建立“由果索因”的能力。

* **Metrics (指标):** Prometheus + Grafana。关注 USE 方法 (Usage, Saturation, Errors) 和 RED 方法 (Rate, Errors,
  Duration)。
* **Logging (日志):** ELK Stack 或 Loki。结构化日志 (JSON)，包含 TraceID。
* **Tracing (链路追踪):** OpenTelemetry / Jaeger / SkyWalking。全链路透传 `Trace-Context`，可视化请求路径。

---

## 8. 基础设施与 DevOps (Infrastructure)

* **基础设施即代码 (IaC):** 使用 Terraform 或 Pulumi 管理云资源，杜绝手动运维。
* **容器化与编排:** Docker + Kubernetes (K8s)。利用 K8s 的自愈能力。
* **CI/CD:**
    * 自动化流水线 (Jenkins/GitLab CI/GitHub Actions)。
    * **质量门禁:** 单元测试覆盖率 < 80% 不允许合并，必须通过 SonarQube 静态代码扫描。
    * **部署策略:** 蓝绿部署 (Blue-Green) 或 金丝雀发布 (Canary)，实现零停机更新。

---

## 9. 总结：技术决策检查表

在进行任何设计变更时，请自问以下问题：

1. 这个设计是否引入了单点故障 (SPOF)？
2. 如果数据量增长 10 倍，这个设计还能工作吗？
3. 如果有新成员加入，他能多快理解这个设计？
4. 如果你离职了，这个系统能被维护吗？

---

