# APM & Service Map

## 1. Background

在前面的章节中，我们已经完成：

- 日志系统（Loki）
- 链路存储（Tempo）
- 可视化平台（Grafana）

但当前系统仍然存在一个关键问题：

- 没有 Trace 数据
- 无法看到调用链路
- 无法生成服务拓扑图（Service Map）

原因在于：

应用尚未接入 OpenTelemetry（OTel）。

本章节将完成应用链路接入，并构建服务级可观测能力（APM）。

---

## 2. Objectives

本章节目标：

1. 理解 APM（Application Performance Monitoring）概念
2. 完成应用接入 OpenTelemetry
3. 生成 Trace 数据
4. 在 Grafana 中查看链路
5. 构建 Service Map（服务拓扑图）

---

## 3. What is APM

APM（应用性能监控）关注：

- 请求响应时间（Latency）
- 错误率（Error Rate）
- 吞吐量（Throughput）
- 服务依赖关系

与传统监控不同：

APM 更关注“用户请求路径”，而不是系统资源。

---

## 4. Architecture

APM 架构如下：

                    +---------------------+
                    |       Grafana       |
                    |   Service Map UI    |
                    +----------+----------+
                               |
                               v
                    +---------------------+
                    |       Tempo         |
                    |   Trace Storage     |
                    +----------+----------+
                               |
                               v
                    +---------------------+
                    | OpenTelemetry Agent |
                    +----------+----------+
                               |
                               v
                        Application Pods

---

## 5. Key Requirement

必须满足以下条件才能看到 Service Map：

- 应用已接入 OpenTelemetry
- 应用之间存在调用关系
- Trace 数据成功写入 Tempo

---

## 6. OpenTelemetry Integration

### 6.1 Java 应用（推荐方式）

下载 Java Agent：

```bash id="dl-agent"
wget https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/latest/download/opentelemetry-javaagent.jar

---

6.2 启动参数配置

在应用启动时加入：

-javaagent:/path/opentelemetry-javaagent.jar \
-Dotel.service.name=user-service \
-Dotel.exporter.otlp.endpoint=http://tempo:4317

说明：

service.name：服务名称（非常关键）
endpoint：Tempo 接收地址
6.3 Kubernetes Deployment 示例
containers:
- name: app
  image: your-app
  env:
  - name: JAVA_TOOL_OPTIONS
    value: >
      -javaagent:/otel/opentelemetry-javaagent.jar
      -Dotel.service.name=demo-service
      -Dotel.exporter.otlp.endpoint=http://tempo:4317
7. Generate Traffic（关键步骤）

必须产生请求流量：

curl http://your-service

否则不会生成 Trace。

8. Verify Trace Data

进入 Grafana：

Explore → Tempo

查看：

是否出现 Trace
是否有多个 Span
9. View Trace Details

点击某个 Trace，可查看：

调用链路
每个 Span 耗时
错误信息
10. Service Map

进入 Grafana：

APM → Service Map

预期效果：

frontend → api → auth → database
11. Key Design Rules（非常重要）
1）Service Name 规范

必须统一命名：

user-service
order-service
api-gateway

否则 Service Map 会混乱。

2）必须存在调用关系

单服务不会形成拓扑图。

3）Trace 必须完整

调用链必须传递 Trace ID。

12. Common Issues
1）没有 Trace

原因：

未接入 OTel
没有流量
2）Service Map 不显示

原因：

只有一个服务
Trace 不完整
3）Trace 不连贯

原因：

Trace ID 未传递
跨服务未统一 OTel
13. Optimization
1）采样率
默认 100%，生产建议降低
2）Span 过滤

减少无意义数据。

3）资源优化

避免 OTel Agent 占用过高资源。

14. Validation Checklist

完成以下验证：

应用成功接入 OTel
能看到 Trace 数据
Trace 包含多个 Span
Service Map 成功生成
服务名称规范
15. Summary

本章节完成了 APM 能力建设。

实现能力：

应用链路追踪
Trace 可视化
服务拓扑图（Service Map）

标志着 Observability 从“系统层”进入“应用层”。

下一章节将进入：

SLI / SLO 与可用性治理。
