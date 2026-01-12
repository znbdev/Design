Payment Webhook Handler
=====

这两个环节是电商交易链路中保证**资金安全**和**库存准确**最重要的两道防线。我将为您分别提供详细的伪代码流程和技术方案。

-----

## 一、 支付回调处理 (Payment Webhook Handler)

这是支付网关（微信/支付宝）通知商家“用户已付款”的接口。
**核心原则**：验签优先、幂等性、金额二次校验。

### 伪代码流程 (Pseudo-Code)

```python
# 定义在 Controller 层的回调接口
def handle_payment_webhook(request):
    # 1. 【安全第一】校验签名 (Verify Signature)
    # 防止伪造请求。使用支付平台公钥验签。
    raw_data = request.body
    signature = request.headers.get("Signature")
    if not payment_service.verify_signature(raw_data, signature):
        # 签名不对直接丢弃，可能是黑客攻击
        return http_response(400, "Invalid Signature")

    # 2. 解析回调数据
    payload = parse_xml_or_json(raw_data)
    platform_trade_no = payload["transaction_id"]  # 支付平台流水号
    merchant_order_no = payload["out_trade_no"]  # 我方订单号
    total_amount = payload["total_fee"]  # 实际支付金额
    trade_status = payload["trade_status"]  # 交易状态

    # 3. 开启数据库事务
    try:
        db.begin_transaction()

        # 4. 【查询订单】并加行锁 (Select for Update)
        # 避免并发情况下，回调和定时取消任务同时操作
        order = db.query("SELECT * FROM orders WHERE order_no = ? FOR UPDATE", merchant_order_no)

        if not order:
            return http_response(200, "SUCCESS")  # 订单不存在，但需告诉支付方收到了，避免重试

        # 5. 【幂等性检查】(Idempotency Check)
        # 如果订单已经是“已支付”状态，说明之前的回调已经处理过了
        if order.status == 'PAID':
            db.commit()
            return http_response(200, "SUCCESS")  # 直接告诉支付方成功

        # 6. 【关键】金额校验 (Amount Check)
        # 防止“1分钱买iPhone”漏洞。比较 数据库应付金额 vs 回调实付金额
        if order.pay_amount != total_amount:
            log.error("金额不一致！可能存在篡改。订单:%s, 应付:%s, 实付:%s",
                      merchant_order_no, order.pay_amount, total_amount)
            # 记录异常单据，人工介入，但不更改订单状态
            db.rollback()
            return http_response(400, "Amount Mismatch")

        # 7. 更新订单状态
        if trade_status == 'SUCCESS':
            # 更新状态为已支付
            order.status = 'PAID'
            order.pay_time = now()
            order.payment_seq = platform_trade_no  # 记录外部流水号
            db.save(order)

            # 8. 真正的扣减库存 (如果在下单阶段只是预占了lock_stock)
            # UPDATE sku SET stock = stock - N, lock_stock = lock_stock - N ...
            inventory_service.deduct_real_stock(order.items)

            # 9. 触发后续流程 (异步)
            # 发送消息队列：通知仓库发货、发送购买成功短信
            mq.send("order_paid_event", order)

        db.commit()

        # 10. 返回协议规定的成功标识 (如微信是XML的SUCCESS)
        return http_response(200, "SUCCESS")

    except Exception as e:
        db.rollback()
        log.error(e)
        # 返回错误，让支付网关稍后重试
        return http_response(500, "Internal Error")
```

-----

## 二、 超时未支付取消 (Order Timeout Cancellation)

当用户下单后，我们预占了库存。如果用户30分钟内未支付，我们需要自动取消订单并释放库存。

### 推荐方案：延迟队列 (Delay Queue)

相比于轮询数据库（效率低、对DB压力大），**延迟队列**是更优雅的工业级方案。

#### 方案架构图

1. **用户下单成功** -\> 插入数据库 -\> **发送消息到延迟队列**（设置延迟时间 30分钟）。
2. **30分钟后** -\> 消息中间件将消息投递给**消费者**。
3. **消费者** -\> 检查订单状态 -\> 执行取消逻辑。

#### 1\. 基于 Redis ZSet 的轻量级实现

如果不想引入 RabbitMQ/RocketMQ，Redis 是最简单的实现方式。利用 `ZSet` (Sorted Set)，将 `score` 设置为“过期时间戳”。

**生产者（下单时）：**

```python
def on_order_created(order_id):
    # 计算过期时间：当前时间 + 30分钟
    expire_timestamp = time.now() + 30 * 60
    # Add to Redis ZSet: key="delay_queue", score=过期时间, member=order_id
    redis.zadd("order_delay_queue", {order_id: expire_timestamp})
```

**消费者（后台常驻进程/定时任务）：**

```python
def process_timeout_orders():
    while True:
        # 1. 取出 expire_timestamp <= 当前时间 的订单
        # ZRANGEBYSCORE key -inf +inf LIMIT 0 1
        items = redis.zrangebyscore("order_delay_queue", 0, time.now(), start=0, num=1)

        if not items:
            sleep(1)  # 队列为空，休息一下
            continue

        order_id = items[0]

        # 2. 从Redis中移除（防止重复处理）
        # 如果移除成功，说明抢到了锁；如果返回0，说明被其他线程抢了
        if redis.zrem("order_delay_queue", order_id) > 0:

            # 3. 【核心逻辑】回查数据库状态
            order = db.get_order(order_id)

            # 只有处于“待支付”状态才取消
            if order and order.status == 'PENDING':
                try:
                    db.begin_transaction()

                    # 标记取消
                    order.status = 'CLOSED'
                    order.close_reason = 'TIMEOUT'
                    db.save(order)

                    # 【重要】释放预占库存
                    # UPDATE sku SET lock_stock = lock_stock - N ...
                    inventory_service.release_lock_stock(order.items)

                    db.commit()
                    log.info(f"订单 {order_id} 已因超时取消")

                except Exception:
                    db.rollback()
                    # 处理失败可能需要重试机制，或者人工告警
            else:
                log.info(f"订单 {order_id} 状态为 {order.status}，无需处理")
```

#### 2\. 基于 RabbitMQ / RocketMQ 的企业级实现

如果系统规模较大，建议使用消息中间件。

* **RabbitMQ**: 使用 `x-delayed-message` 插件，或者 TTL + 死信队列 (DLX)。

    * 下单发送消息到 `order.create` Exchange，设置 `expiration=1800000` (30分钟)。
    * 消息过期后，自动转发到 `order.close` 队列。
    * 消费者监听 `order.close` 队列。

* **RocketMQ**: 原生支持延迟消息。

    * `message.setDelayTimeLevel(16)` (对应30分钟)。

### 总结与对比

| 方案             | 优点          | 缺点                  | 适用场景            |
|:---------------|:------------|:--------------------|:----------------|
| **定时轮询 DB**    | 简单，无新技术栈    | 数据量大时性能差，有时间误差      | 极早期 MVP 项目      |
| **Redis ZSet** | 轻量，实时性好     | 需自行实现消费者轮询逻辑，需保证原子性 | 中小型项目，已有Redis设施 |
| **消息队列 (MQ)**  | 高可靠，解耦，吞吐量大 | 架构复杂，运维成本高          | 中大型电商系统 (推荐)    |

### 特别提醒：由于“并发”导致的问题

在**支付回调**和**超时取消**之间存在极小的并发冲突概率：

* **场景**：用户在第 29分59秒 支付成功。
* **并发**：
    1. 第 30分00秒，**支付回调**到达，正在尝试将状态改为 `PAID`。
    2. 第 30分00秒，**延迟任务**触发，检测到状态还是 `PENDING`（因为回调还没 Commit），尝试将状态改为 `CLOSED`。
* **解决方案**：
    * **数据库行锁**：两个逻辑在更新 `orders` 表时，都必须先获取锁。
    * **版本号/状态机校验**：
        * 回调更新 SQL: `UPDATE orders SET status='PAID' WHERE id=1 AND status='PENDING'`
        * 取消更新 SQL: `UPDATE orders SET status='CLOSED' WHERE id=1 AND status='PENDING'`
        * 数据库的原子性保证了只有一个能成功。如果回调先成功，取消操作就会影响 0 行（失败），从而避免错误取消。