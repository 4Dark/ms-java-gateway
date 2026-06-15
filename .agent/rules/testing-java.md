---
trigger: glob
globs: ["**/*Test.java", "**/*Tests.java"]
---

# Java 测试规范 (ms-java-gateway)

## 1. 环境隔离
- 测试配置 `application-test.yml` 中 `uri` 使用 `127.0.0.1`，避免 DNS 解析延迟

## 2. 鲁棒断言原则
- Filter 功能测试断言侧重**是否正确放行/拦截**（如 `status != 401`），不依赖下游业务返回码

## 3. OIDC 配置隔离
- **禁止**将 `issuer-uri` 放在 `application.yml`（触发测试环境自动发现）
- `issuer-uri` 只存在于 `application-prod.yml`，测试环境在 `application-test.yml` 显式 Mock OIDC 端点

## 4. 无状态化断言
- 网关已重构为无状态（Cookie 存储），测试必须校验 Cookie（如 `redirect_uri`）而非 `WebSession` 属性

## 5. 架构守护测试
- `ArchitectureGuardTest` 必须覆盖所有 Filter/Config Bean 装配验证和 Order 值断言
- 新增 Filter 时必须同步新增守护用例

## 6. AbstractErrorWebExceptionHandler 注意
自定义错误处理器构造函数必须调用 `setMessageWriters()`，否则测试启动报 `Property 'messageWriters' is required`
