# Observability Architecture

## 1. Background

在 Kubernetes 集群中，仅依赖监控系统（Metrics）已经无法满足企业级运维需求。

虽然 Prometheus + Grafana 可以提供：

- 资源使用情况
- 系统运行状态
- 基础告警能力

但在实际生产环境中仍然存在以下问题：

- 无法查看应用日志细节
- 无法分析请求调用链路
- 无法快速定位性能瓶颈
- 无法从用户视角衡量系统可用性

因此，需要在监控系统基础上，引入完整的可观测性体系（Observability）。

本项目在 enterprise-infra-lab 实验环境中构建一套完整的可观测性平台，
用于验证 Kubernetes 环境下日志、链路追踪及可用性治理能力。

---

## 2. Observability Objectives

本可观测性系统的设计目标包括：

1. 实现 Kubernetes 日志集中采集与检索
2. 支持跨服务请求链路追踪
3. 提供服务级调用拓扑关系
4. 实现日志、指标、链路的统一关联分析
5. 提供从系统到应用的全链路可观测能力
6. 支持基于用户体验的可用性评估（SLO）

---

## 3. Observability Stack

本项目采用当前云原生主流可观测性技术栈：

Metrics + Logs + Tracing

具体组件如下：

Prometheus

- 负责采集监控指标
- 提供时间序列数据存储
- 支持 PromQL 查询

Grafana

- 提供统一可视化界面
- 支持 Metrics / Logs / Traces 多数据源
- 展示 Dashboard 与 Service Map

Loki

- 日志聚合系统
- 存储容器日志
- 支持标签索引查询

Promtail

- 日志采集组件
- 从节点采集容器日志
- 推送至 Loki

Tempo

- 分布式链路追踪系统
- 存储 Trace 数据
- 与 Grafana 深度集成

OpenTelemetry

- 统一观测数据采集标准
- 采集 Trace 数据
- 支持多语言接入

---

## 4. Observability Components

可观测性体系由三大核心组成：

### Metrics（指标）

- 系统资源监控
- Kubernetes 状态监控
- 应用性能指标

数据来源：

- Node Exporter
- kube-state-metrics
- cAdvisor

---

### Logs（日志）

- 应用运行日志
- 错误日志分析
- 容器标准输出采集

特点：

- 支持按标签检索
- 支持多维度过滤（namespace / pod / container）

---

### Tracing（链路追踪）

- 分布式请求链路分析
- 请求耗时分解
- 服务依赖关系分析

核心概念：

- Trace：一次完整请求
- Span：链路中的一个节点

---

## 5. Observability Architecture

整体可观测性架构如下：

                    +---------------------+
                    |       Grafana       |
                    | Unified UI (All)    |
                    +----------+----------+
                               |
        +----------------------+----------------------+
        |                      |                      |
        v                      v                      v
+---------------+     +---------------+     +----------------+
|  Prometheus   |     |     Loki      |     |     Tempo      |
|  Metrics      |     |    Logs       |     |    Traces      |
+-------+-------+     +-------+-------+     +--------+-------+
        |                     |                      |
        v                     v                      v
 Exporters              Promtail              OpenTelemetry
        |                     |                      |
        +---------- Kubernetes Cluster -------------+

---

## 6. Data Flow

可观测性数据流如下：

### Metrics 数据流

Exporter → Prometheus → Grafana

---

### Logs 数据流

Container stdout → Promtail → Loki → Grafana

---

### Tracing 数据流

Application → OpenTelemetry → Tempo → Grafana

---

## 7. Deployment Strategy

可观测性系统基于 Helm 部署。

主要组件：

- Loki Stack（Loki + Promtail）
- Tempo
- OpenTelemetry（应用侧接入）

部署策略：

- 统一部署在 monitoring 命名空间
- 与现有 Prometheus 体系集成
- 通过 Grafana 实现统一入口

---

## 8. Storage Strategy

不同数据类型采用不同存储策略：

### Metrics

- 存储在 Prometheus TSDB
- 默认保留周期：15 天

---

### Logs

- 存储在 Loki
- 建议设置 retention（如 7 天）

---

### Traces

- 存储在 Tempo
- 可配置短期存储（如 3~7 天）

---

## 9. Integration With Existing Monitoring

本可观测性系统基于已有监控体系扩展：

- 复用 Prometheus
- 复用 Grafana
- 扩展 Loki 日志能力
- 扩展 Tempo 链路能力

最终形成统一平台：

Metrics + Logs + Tracing

---

## 10. Future Evolution

可观测性体系可进一步演进：

- 日志长期存储（对象存储）
- Trace 采样优化
- APM 深度分析
- Service Mesh 集成（如 Istio）
- 自动化故障诊断

逐步向 SRE 体系演进。

---

## 11. Summary

本章节设计了 Kubernetes 可观测性系统架构。

该体系在原有监控基础上扩展：

- 日志系统（Loki）
- 链路追踪（Tempo）
- 统一可视化（Grafana）

形成完整 Observability 平台：

Metrics + Logs + Tracing

为后续日志采集、链路追踪与 SLO 实践提供基础。

后续章节将进行日志系统与链路追踪系统的部署与验证。
