---
trigger: glob
globs: ["**/*.java"]
---

# Java 编码规范 (ms-java-gateway)

## 1. 严格非阻塞 (Strictly Non-Blocking)
- **绝对禁止**: `Thread.sleep()`、传统阻塞 I/O、JDBC 驱动
- **必须使用**: `Mono`、`Flux` 以及响应式操作符（`map`, `flatMap`, `filter`）
- 极少情况需阻塞时，必须用 `subscribeOn(Schedulers.boundedElastic())` 包装

## 2. 依赖管理
- **禁止**引入 `spring-boot-starter-web`（Servlet 栈），只使用 `spring-boot-starter-webflux`

## 3. 路由与过滤器
- 路由配置优先在 `application.yaml` 中配置，特殊逻辑使用 Java Config (`RouteLocator`)
- 实现 `GlobalFilter` 处理统一鉴权 + 日志；Filter 必须实现 `Ordered` 接口
- Filter 优先级：`TraceIdFilter` Order < `JwtAuthenticationFilter` Order（当前 `-200` vs `-100`）

## 4. SSE 流式透传
- **不要**缓冲响应内容，支持 Python 服务到前端的流式透传
- `response-timeout` 不宜设置过短（建议 ≥ 30s）

## 5. 安全认证 (JWT/WebFilter/CORS)
- JWT 校验必须实现为 `WebFilter`，注入 `SecurityWebFilterChain`（`addFilterAt AUTHENTICATION` 阶段）
- 解析 `sub` 注入 `X-User-Id` 透传；禁止注入 `X-User-Name`、`X-User-Avatar` 等业务头
- 必须配置 `CorsConfigurationSource` 放行 `OPTIONS` 请求
- 原始 JWT（`Authorization: Bearer <token>`）必须透传给下游

## 6. 错误处理
- 继承 `AbstractErrorWebExceptionHandler`，必须在构造函数调用 `setMessageWriters()`
- 错误响应必须含：`traceId`、`status`、`error`、`message`、`path`、`timestamp`
- 异常消息超 200 字符截断；`ConnectException` → `"Service temporarily unavailable"`

## 7. 链路追踪
- 网关必须生成全局唯一 Trace-ID（32位 hex，UUID 去连字符），写入请求头和响应头
- 上游已注入 `X-Trace-Id` 时必须复用，不得覆盖

## 8. Nacos 白名单一致性
新增放行路径时，本地 `application.yml` 和 **Nacos 配置中心**必须同步更新。
