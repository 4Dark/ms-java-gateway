---
name: ms-java-gateway-deep-reference
description: >
  ms-java-gateway 完整规范深度参考。在以下场景下加载：
  - 需要了解完整的 JWT Filter 边界约束和 Security 链设计
  - 讨论网关层的熔断/降级/重试策略
  - 架构评审网关的 BFF 聚合职责
  - 需要了解 Resilience4j 与 SSE 长连接的兼容性考量时
---

# ms-java-gateway 完整规范参考

> 此文档是深度参考，日常编码请使用 Glob 触发的 `coding-java.md`

## JWT Filter 完整边界

| ✅ 允许 | ❌ 禁止 |
|---------|---------|
| 校验 JWT 签名与过期时间 | 解析业务级 Claims（`name`、`avatar`、`role`） |
| 提取 `sub` 注入 `X-User-Id` | 注入 `X-User-Name`、`X-User-Avatar` 等业务头 |
| 透传原始 JWT（`Authorization: Bearer`） | 对 Claims 字段做 fallback 适配 |
| 白名单路径放行 & OPTIONS 放行 | 查询数据库或调用外部服务进行权限校验 |

## 弹性与容错机制完整规范

```yaml
spring.cloud.gateway.httpclient:
  connect-timeout: 3000   # 连接超时 3s
  response-timeout: 30s   # SSE 流场景建议 ≥ 30s
```

- 路由调用必须包裹在熔断器（Resilience4j）中
- 下游宕机/超时时，网关必须返回标准化 JSON 错误（不暴露堆栈）
- 只对幂等请求（GET）配置自动重试，禁止对 POST/PUT 自动重试

## 统一错误处理器完整规范
- 继承 `AbstractErrorWebExceptionHandler`，构造函数注入 `ServerCodecConfigurer`
- 必须调用 `setMessageWriters()` / `setMessageReaders()`
- 消息清洗：超 200 字符截断；`ConnectException` → `"Service temporarily unavailable"`

## Trace-ID 完整规范
- 格式：32 位 hex（UUID 去连字符）
- `TraceIdFilter` Order 必须 < `JwtAuthenticationFilter` Order（当前 `-200` vs `-100`）
- 上游已注入 `X-Trace-Id` 时必须复用，不得覆盖
- 写入请求头（传递给下游）和响应头（方便前端排查）
