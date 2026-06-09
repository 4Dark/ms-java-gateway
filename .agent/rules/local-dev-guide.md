---
trigger: glob
globs: ["**/*.yaml", "**/*.yml"]
---

# 本地开发指南 (ms-java-gateway)

## 服务配置
- **端口**: `8281`
- **VS Code 启动**: `.vscode/launch.json`（含必备环境变量如 Nacos 连接）

## JVM 内存红线
Spring Boot 3 + OAuth2 Client 架构下，`MaxMetaspaceSize` **严禁低于 128m**。
推荐配置：`-Xmx256m -Xms256m -XX:MaxMetaspaceSize=128m -XX:+UseG1GC`

## AI 重启规范
重启网关时，必须读取 `.vscode/launch.json` 获取正确的 JVM 参数与环境变量。

## 请求流转日志
网关入口（`【网关请求】`）和出口（`【网关响应完成】`）必须使用 `INFO` 级别，用于生产追踪。
