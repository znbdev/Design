API Interface Definition
=====

这份文档提供了电商核心交易链路的 **OpenAPI 3.0 (Swagger)** 接口定义。

重点设计了 **"创建订单" (Place Order)** 接口，这是整个电商系统中最复杂、参数校验要求最高的接口。

您可以将以下 YAML 代码直接粘贴到 [Swagger Editor](https://editor.swagger.io/) 中查看可视化文档或生成代码。

-----

# 电商核心 API 接口定义 (OpenAPI 3.0)

### 核心设计说明

1.  **RESTful 风格**: 遵循标准的 HTTP 动词语义。
2.  **幂等性 (Idempotency)**: 在创建订单接口中引入 `idempotency-key` Header，防止网络抖动导致用户重复下单。
3.  **金额计算**: 所有的金额计算都在服务端进行，前端仅提交 SKU ID 和 数量，**绝不提交价格**。

-----

```yaml
openapi: 3.0.3
info:
  title: E-Commerce Core API
  description: 电商核心交易系统 API 文档，包含商品、购物车及订单流程。
  version: 1.0.0
servers:
  - url: https://api.example-shop.com/v1
    description: 生产环境

tags:
  - name: Orders
    description: 订单交易核心接口
  - name: Products
    description: 商品查询接口

paths:
  # ----------------------------------------------------------------
  # 1. 创建订单接口 (核心)
  # ----------------------------------------------------------------
  /orders:
    post:
      tags:
        - Orders
      summary: 创建订单 (下单)
      description: >
        用户提交购买请求。系统将校验库存、计算最终价格（含运费/优惠）、预占库存并返回订单号。
        前端需在 header 中携带幂等键。
      operationId: createOrder
      security:
        - bearerAuth: []
      parameters:
        - in: header
          name: Idempotency-Key
          schema:
            type: string
            format: uuid
          required: true
          description: 幂等性Key (UUID)，防止重复提交
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/OrderCreateRequest'
      responses:
        '201':
          description: 订单创建成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderCreateResponse'
        '400':
          description: 参数错误 / 库存不足 / 商品已下架
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '401':
          description: 未授权 (需登录)

  # ----------------------------------------------------------------
  # 2. 查询订单详情
  # ----------------------------------------------------------------
  /orders/{orderNo}:
    get:
      tags:
        - Orders
      summary: 获取订单详情
      parameters:
        - in: path
          name: orderNo
          schema:
            type: string
          required: true
          description: 订单编号
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/OrderDetail'

  # ----------------------------------------------------------------
  # 3. 商品详情 (用于下单前校验)
  # ----------------------------------------------------------------
  /products/{spuId}:
    get:
      tags:
        - Products
      summary: 获取商品SPU及SKU详情
      parameters:
        - in: path
          name: spuId
          schema:
            type: integer
            format: int64
          required: true
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductDetail'

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    # --- 请求体模型 ---
    OrderCreateRequest:
      type: object
      required:
        - addressId
        - items
      properties:
        addressId:
          type: integer
          format: int64
          description: 用户收货地址ID
          example: 10086
        couponId:
          type: integer
          format: int64
          description: 用户选中的优惠券ID (可选)
          nullable: true
          example: 501
        remark:
          type: string
          description: 买家留言
          maxLength: 200
          example: "请尽快发货"
        items:
          type: array
          description: 购买商品列表
          minItems: 1
          items:
            $ref: '#/components/schemas/OrderItemInput'

    OrderItemInput:
      type: object
      required:
        - skuId
        - quantity
      properties:
        skuId:
          type: integer
          format: int64
          description: 商品SKU ID
          example: 2001
        quantity:
          type: integer
          description: 购买数量
          minimum: 1
          maximum: 99
          example: 2

    # --- 响应体模型 ---
    OrderCreateResponse:
      type: object
      properties:
        orderNo:
          type: string
          description: 生成的订单号
          example: "202310270001"
        payAmount:
          type: number
          format: double
          description: 最终需支付金额 (后端计算结果)
          example: 299.00
        expireTime:
          type: string
          format: date-time
          description: 支付截止时间
          example: "2023-10-27T10:30:00Z"
        paymentToken:
          type: string
          description: 支付网关所需的Token或参数串
          example: "wx_pay_xxxyyyzzz"

    OrderDetail:
      type: object
      properties:
        orderNo:
          type: string
        status:
          type: integer
          description: 10:待支付, 20:待发货, 30:已发货, 40:完成
        totalAmount:
          type: number
          description: 商品原价总额
        discountAmount:
          type: number
          description: 优惠金额
        freightAmount:
          type: number
          description: 运费
        payAmount:
          type: number
          description: 实付金额
        items:
          type: array
          items:
            $ref: '#/components/schemas/OrderItemSnapshot'

    OrderItemSnapshot:
      type: object
      properties:
        spuTitle:
          type: string
        skuSpecs:
          type: string
          description: 规格描述快照
          example: "颜色:黑色; 容量:256G"
        price:
          type: number
          description: 下单时的单价
        quantity:
          type: integer

    ProductDetail:
      type: object
      properties:
        id:
          type: integer
        title:
          type: string
        skus:
          type: array
          items:
            type: object
            properties:
              id:
                type: integer
              specs:
                type: object
              stock:
                type: integer
              price:
                type: number

    ErrorResponse:
      type: object
      properties:
        code:
          type: integer
          example: 4001
        message:
          type: string
          example: "商品库存不足"
```

### 关键字段解析

1.  **`Idempotency-Key` (Header)**:

    * 在下单接口中，这是至关重要的。如果用户点击下单后网络超时，用户再次点击，前端应发送**相同**的Key。
    * 后端若检测到Key已存在，直接返回上次生成的订单结果，而不是重新扣库存。

2.  **`OrderItemInput` (Request)**:

    * 注意这里**只传 `skuId` 和 `quantity`**。
    * **不要传 `price`**：千万不要信任前端传来的价格，后端必须根据 `skuId` 查数据库中的当前售价。

3.  **`paymentToken` (Response)**:

    * 创建订单成功后，通常会直接返回用于唤起微信/支付宝 SDK 的参数串，避免前端再发一次请求去获取支付参数。

-----

