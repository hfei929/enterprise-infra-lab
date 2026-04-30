# Tempo Deployment

## 1. Background

在上一章节中，我们完成了分布式链路追踪（Tracing）体系的设计。

本章节将基于 Kubernetes 集群部署 Tempo，
并与 Grafana 集成，实现链路数据的存储与可视化。

需要注意：

仅部署 Tempo 并不会产生 Trace 数据，
必须配合 OpenTelemetry 才能完整工作。

---

## 2. Deployment Objectives

本章节目标包括：

1. 在 Kubernetes 集群中部署 Tempo
2. 在 Grafana 中配置 Tempo 数据源
3. 验证 Tempo 服务是否正常运行
4. 为后续 OpenTelemetry 接入做好准备

---

## 3. Environment Requirements

部署前需满足以下条件：

- Kubernetes 集群已运行
- Helm 已安装
- 已部署 Grafana（上一阶段完成）
- 已具备日志系统（Loki）基础

---

## 4. Add Helm Repository

添加 Grafana Helm 仓库（若已添加可跳过）：

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

5. Deploy Tempo

使用 Helm 部署 Tempo：

helm install tempo grafana/tempo \
  -n monitoring

说明：

tempo：Trace 存储组件
默认配置适用于实验环境
生产环境需要额外优化存储
6. Verify Deployment

检查 Pod 状态：

kubectl get pods -n monitoring

预期组件：

tempo

查看服务：

kubectl get svc -n monitoring
7. Check Tempo Logs

确认 Tempo 是否正常运行：

kubectl logs -n monitoring -l app.kubernetes.io/name=tempo

检查是否存在报错信息。

8. Configure Grafana Data Source

进入 Grafana：

Connections → Data Sources → Add data source → Tempo

配置如下：

URL: http://tempo:3100

点击 Save & Test，确认连接成功。

9. Verify Tempo Availability

在 Grafana 中：

Explore → Tempo

此时可能没有任何 Trace 数据，这是正常现象。

原因：

应用尚未接入 OpenTelemetry
没有产生 Trace 数据
10. Key Concept (重要)

Tempo 只负责：

存储 Trace
提供查询能力

不负责：

采集 Trace
生成 Trace

Trace 数据来源：

Application → OpenTelemetry → Tempo

11. OpenTelemetry Integration (核心说明)

要看到链路数据，必须接入 OpenTelemetry。

常见接入方式：

1）Java 应用（推荐）

使用 Java Agent：

-javaagent:opentelemetry-javaagent.jar
2）Go 应用

使用 OTel SDK：

// 初始化 tracer（示意）
3）Sidecar / Agent 模式

通过 OTel Collector 统一采集。

12. Common Issues
1）Grafana 无法连接 Tempo

检查：

kubectl get svc -n monitoring

确认服务地址是否正确。

2）没有 Trace 数据

原因：

未接入 OpenTelemetry（最常见）
应用未产生请求
3）Tempo Pod 异常

检查日志：

kubectl logs -n monitoring <tempo-pod>
13. Optimization（重要）
1）存储优化

默认使用本地存储，不适合生产环境。

可扩展：

MinIO
S3
2）Trace 保留时间

建议设置较短：

3~7 天
3）资源限制
resources:
  limits:
    cpu: 500m
    memory: 512Mi
14. Validation Checklist

完成以下验证：

Tempo Pod 正常运行
Grafana 成功连接 Tempo
Explore 页面可选择 Tempo
无报错日志
15. Summary

本章节完成了 Tempo 的部署与基础验证。

实现能力：

Trace 数据存储
Grafana 查询能力接入

但当前系统仍缺少：

Trace 数据来源（OpenTelemetry）

下一章节将进入：

应用链路接入与服务拓扑构建（APM）。
