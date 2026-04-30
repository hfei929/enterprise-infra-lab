# Grafana Deployment and Configuration

## 1. Overview

Grafana 是 Kubernetes 监控系统中的可视化组件。

在本项目中，Grafana 通过 Helm Chart
"kube-prometheus-stack" 自动部署，并默认配置
Prometheus 作为数据源。

Grafana 主要提供以下能力：

- 监控指标可视化
- Dashboard 管理
- 数据查询与分析
- 告警展示

---

## 2. Deployment Architecture

Grafana 作为监控系统的展示层组件运行在 Kubernetes 集群中。

系统结构：

Prometheus
    |
    v
Grafana
    |
    v
Dashboard

Grafana 通过 PromQL 从 Prometheus 查询监控数据。

---

## 3. Service Configuration

Grafana Service 默认类型为：

ClusterIP

服务名称：

monitoring-grafana

查看 Service：

kubectl get svc -n monitoring

示例输出：

NAME                 TYPE        CLUSTER-IP       PORT(S)
monitoring-grafana   ClusterIP   10.96.x.x        80/TCP

---

## 4. Access Method

在实验环境中，通过 port-forward 访问 Grafana。

执行命令：

kubectl port-forward svc/monitoring-grafana \
3000:80 -n monitoring

浏览器访问：

http://localhost:3000

---

## 5. Login Credentials

默认账号：

admin

默认密码：

admin123

首次登录后建议修改密码。

---

## 6. Data Source Configuration

Grafana 默认会自动配置 Prometheus 数据源。

查看方法：

Settings → Data Sources

Prometheus 地址通常为：

http://monitoring-kube-prometheus-prometheus:9090

状态应显示为：

Data source is working

---

## 7. Built-in Dashboards

kube-prometheus-stack 会自动导入 Kubernetes 监控 Dashboard。

常见 Dashboard 包括：

Kubernetes / Compute Resources / Node

显示节点资源使用情况：

- CPU
- Memory
- Network
- Disk

Kubernetes / Compute Resources / Pod

显示 Pod 与容器资源使用情况。

Kubernetes / Networking

展示网络相关指标。

Kubernetes / API Server

展示 Kubernetes API Server 指标。

---

## 8. Dashboard Structure

Grafana Dashboard 通常包含以下监控维度：

Cluster Level

- Node 状态
- API Server
- Scheduler

Node Level

- CPU 使用率
- 内存使用
- 磁盘 IO

Pod Level

- Pod CPU
- Pod Memory
- Pod Restart

Container Level

- Container CPU
- Container Memory

---

## 9. Dashboard Customization

Grafana 支持自定义 Dashboard。

常见操作包括：

- 添加 Panel
- 编写 PromQL 查询
- 设置图表类型
- 设置刷新周期

例如：

CPU 使用率查询：

rate(node_cpu_seconds_total{mode!="idle"}[5m])

---

## 10. Monitoring Storage System

在本项目中，Kubernetes 使用 Longhorn 作为持久化存储。

后续可以在 Grafana 中增加存储监控：

Volume 状态
Replica 状态
IO 延迟
Volume 容量使用率

用于监控分布式存储运行状态。

---

## 11. Deployment Verification

Grafana 部署验证步骤：

1. 访问 Grafana Web UI
2. 确认 Prometheus Data Source 正常
3. 打开 Kubernetes Dashboard
4. 确认监控图表显示数据

---

## 12. Next Step

下一章节将介绍监控指标采集机制：

04-metrics-collection.md
