# Loki & Promtail Deployment

## 1. Background

在上一章节中，我们完成了日志系统的架构设计。

本章节将基于 Kubernetes 集群，部署 Loki + Promtail 日志系统，
并验证日志采集与查询流程。

---

## 2. Deployment Objectives

本章节目标包括：

1. 在 Kubernetes 集群中部署 Loki
2. 部署 Promtail 采集日志
3. 验证日志采集是否正常
4. 在 Grafana 中查询日志数据
5. 完成日志系统的基础验证

---

## 3. Environment Requirements

在开始部署前，需要满足以下条件：

- Kubernetes 集群已运行
- Helm 已安装
- 已部署 Prometheus + Grafana（上一阶段完成）
- kubectl 可正常访问集群

---

## 4. Add Helm Repository

添加 Grafana 官方 Helm 仓库：

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

---

## 5. Deploy Loki Stack

使用 Helm 一键部署 Loki + Promtail：

```bash
helm install loki grafana/loki-stack \
  -n monitoring --create-namespace

说明：

- loki：日志存储组件
- promtail：日志采集组件
- 默认包含基础配置，适合实验环境

---

## 6. Verify Deployment

查看 Pod 状态：

```bash
kubectl get pods -n monitoring

预期组件：

- loki
- promtail（DaemonSet，每个节点一个）

检查 Promtail：

```bash
kubectl get daemonset -n monitoring

---

7. Check Promtail Logs

确认 Promtail 是否正常采集日志：

```bash
kubectl logs -n monitoring -l app=promtail

关键观察点：

- 是否成功读取日志文件
- 是否成功推送至 Loki
- 是否存在报错

---


## 8. Access Grafana

获取 Grafana 访问方式：

```bash
kubectl get svc -n monitoring

访问 Grafana Web UI。

## 9. Configure Loki Data Source

在 Grafana 中添加 Loki 数据源：

路径：

```bash
Connections → Data Sources → Add data source → Loki

配置：

URL: http://loki:3100

保存并测试连接。

## 10. Query Logs

进入 Grafana：

Explore → Loki

执行查询：

```bash
{namespace="default"}

或：

```bash
{pod=~".*"}

验证是否能看到日志数据。

## 11. Log Label Verification

检查日志标签是否正常：

- namespace
- pod
- container

这些标签用于后续日志过滤与分析。

## 12. Common Issues
###1）Grafana 无日志

检查：

```bash
kubectl logs -n monitoring -l app=promtail

确认 Promtail 是否正常运行。

### 2）Promtail 未采集日志

检查：

节点是否存在日志目录
Promtail 是否挂载 /var/log/containers
### 3）Loki 无数据

检查 Loki Pod：

```bash
kubectl logs -n monitoring -l app=loki
## 13. Optimization (重要)
### 1）日志保留策略

默认无限增长，必须限制：

```bash
limits_config:
  retention_period: 168h
### 2）日志过滤

避免采集无用日志（如系统日志）。

### 3）资源限制

为 Loki 和 Promtail 设置资源限制：

```bash
resources:
  limits:
    cpu: 500m
    memory: 512Mi

---

## 14. Validation Checklist

完成以下验证：

- Loki Pod 正常运行
- Promtail 在所有节点运行
- Grafana 能连接 Loki
- 能查询到 Pod 日志
- 标签字段正常

---

## 15. Summary


本章节完成了 Loki + Promtail 日志系统的部署。

实现能力：

 - 容器日志自动采集
 - 日志集中存储
 - Grafana 可视化查询

标志着 Kubernetes 日志系统正式落地。
