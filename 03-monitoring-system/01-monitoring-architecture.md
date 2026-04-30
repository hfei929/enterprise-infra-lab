# Monitoring System Architecture

## 1. Background

在 Kubernetes 集群中，监控系统是平台稳定运行的重要组成部分。  
通过持续采集节点、容器和 Kubernetes 对象的运行指标，可以实现：

- 集群运行状态可观测
- 资源使用情况监控
- 异常告警
- 故障定位与分析

本项目在 enterprise-infra-lab 实验环境中构建一套基础监控体系，
用于验证 Kubernetes 环境下的监控架构设计与运行机制。

---

## 2. Monitoring Objectives

本监控系统的设计目标包括：

1. 实时监控 Kubernetes 集群运行状态
2. 采集节点 CPU、内存、磁盘、网络等资源指标
3. 监控 Pod、容器以及 Kubernetes 对象状态
4. 提供统一的可视化监控界面
5. 支持告警通知机制
6. 为后续系统扩展提供监控基础

---

## 3. Monitoring Stack

本项目采用当前 Kubernetes 社区广泛使用的监控技术栈：

Prometheus + Grafana + Alertmanager

组件职责如下：

Prometheus

- 负责采集监控指标
- 存储时间序列数据
- 提供 PromQL 查询接口
- 触发告警规则

Grafana

- 提供监控数据可视化
- 构建监控 Dashboard
- 支持多数据源查询

Alertmanager

- 负责接收 Prometheus 告警
- 告警分组与去重
- 告警通知（Email / Webhook 等）

---

## 4. Metrics Sources

监控系统通过 Exporter 采集不同层级的监控指标。

主要指标来源如下：

### Node Exporter

采集节点操作系统层指标：

- CPU 使用率
- 内存使用情况
- 磁盘 IO
- 网络流量

### kube-state-metrics

采集 Kubernetes 资源对象状态：

- Pod 状态
- Deployment 副本数
- StatefulSet 状态
- Node 状态

### cAdvisor

采集容器运行指标：

- Container CPU
- Container Memory
- Container Filesystem
- Container Network

---

## 5. Monitoring Architecture

整体监控架构如下：

                    +---------------------+
                    |       Grafana       |
                    |   Dashboard UI      |
                    +----------+----------+
                               |
                               v
                    +---------------------+
                    |     Prometheus      |
                    |   Metrics Storage   |
                    +----------+----------+
                               |
             +-----------------+-----------------+
             |                                   |
             v                                   v
     +---------------+                   +-------------------+
     | Node Exporter |                   | kube-state-metrics|
     +---------------+                   +-------------------+
             |                                   |
             v                                   v
     Kubernetes Nodes                    Kubernetes API Server

Alertmanager 负责接收 Prometheus 告警并进行通知分发。

---

## 6. Data Flow

监控数据流流程如下：

1. Exporter 组件采集节点或 Kubernetes 状态指标
2. Prometheus 定期通过 HTTP Pull 模式抓取指标
3. 指标数据存储在 Prometheus TSDB 中
4. Grafana 通过 PromQL 查询 Prometheus 数据
5. Alertmanager 接收 Prometheus 触发的告警

数据流简化流程：

Exporter → Prometheus → Grafana / Alertmanager

---

## 7. Deployment Strategy

监控系统采用 Helm 方式部署。

部署组件：

- Prometheus Server
- Alertmanager
- Grafana
- Node Exporter
- kube-state-metrics

Helm Chart：

kube-prometheus-stack

该 Chart 提供完整 Kubernetes 监控组件部署能力。

---

## 8. Storage Strategy

Prometheus 默认使用本地存储。

存储策略：

Retention: 15 days

数据保存在 Prometheus Pod 的 Persistent Volume 中。

后续可以扩展：

- 远程存储
- 长期指标归档

可选方案包括：

Thanos
VictoriaMetrics

---

## 9. Integration With Storage Monitoring

本项目已部署 Longhorn 作为 Kubernetes 持久化存储。

后续监控系统将增加以下指标：

- Volume Health
- Replica 状态
- IO 延迟
- Volume Capacity

用于监控分布式存储运行状态。

---

## 10. Future Evolution

未来监控体系可进一步扩展为完整的可观测性平台。

扩展方向包括：

日志系统

Loki

分布式追踪

Jaeger

长期指标存储

Thanos

形成完整的 Observability 平台：

Metrics + Logs + Tracing

---

## 11. Summary

本章节设计了 Kubernetes 集群监控系统架构。

监控体系基于 Prometheus 生态构建，主要包括：

Prometheus
Grafana
Alertmanager

该架构能够满足 Kubernetes 集群基础监控需求，并具备良好的扩展能力。

后续章节将进行 Prometheus 与 Grafana 的实际部署与验证。
