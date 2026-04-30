# Logging System Design

## 1. Background

在 Kubernetes 集群中，容器日志默认以标准输出（stdout / stderr）的形式存在。

虽然可以通过 `kubectl logs` 查看单个 Pod 日志，但在实际环境中存在明显问题：

- Pod 数量多，日志分散在不同节点
- Pod 重建后日志容易丢失
- 无法进行跨 Pod、跨服务检索
- 无法进行日志聚合分析
- 排障效率低

因此，需要构建集中式日志系统，实现统一采集、存储与查询。

本项目在 enterprise-infra-lab 实验环境中设计并实现一套 Kubernetes 日志系统，
用于验证容器日志的集中化管理与分析能力。

---

## 2. Logging Objectives

日志系统设计目标包括：

1. 实现 Kubernetes 容器日志集中采集
2. 支持跨 namespace / Pod / container 检索
3. 支持关键字搜索与过滤
4. 支持日志实时查看与历史查询
5. 支持与监控系统联动（日志 + 指标）
6. 提升故障定位效率

---

## 3. Logging Stack

本项目采用轻量级云原生日志方案：

Loki + Promtail + Grafana

组件职责如下：

Loki

- 日志存储系统
- 基于标签索引（非全文索引）
- 适合云原生场景

Promtail

- 日志采集组件
- 从节点读取容器日志文件
- 推送至 Loki

Grafana

- 日志查询与展示
- 支持与 Metrics / Tracing 联动分析

---

## 4. Logging Architecture

日志系统整体架构如下：

                    +---------------------+
                    |       Grafana       |
                    |   Logs Query UI     |
                    +----------+----------+
                               |
                               v
                    +---------------------+
                    |        Loki         |
                    |    Log Storage      |
                    +----------+----------+
                               |
                               v
                    +---------------------+
                    |      Promtail       |
                    |   Log Collector     |
                    +----------+----------+
                               |
                               v
                  Kubernetes Nodes (/var/log/containers)

---

## 5. Log Sources

日志数据主要来源于 Kubernetes 容器运行日志：

### Container Logs

路径：

/var/log/containers/*.log

来源：

- Pod stdout
- Pod stderr

---

### System Components（可选扩展）

- kubelet 日志
- container runtime 日志（containerd）

---

## 6. Data Flow

日志数据流流程如下：

1. 容器将日志输出到 stdout / stderr
2. kubelet 将日志写入节点文件系统
3. Promtail 在节点上采集日志文件
4. Promtail 将日志推送至 Loki
5. Grafana 查询 Loki 并展示日志

数据流简化：

Container → Promtail → Loki → Grafana

---

## 7. Label Design（关键设计）

Loki 不使用全文索引，依赖标签进行查询。

推荐标签设计：

- namespace
- pod
- container
- app（业务标签）

示例：
namespace="default", pod="nginx-xxx"


良好的标签设计可以显著提升查询效率。

---

## 8. Deployment Strategy

日志系统采用 Helm 部署。

部署组件：

- Loki
- Promtail

Helm Chart：

grafana/loki-stack

部署策略：

- 部署在 monitoring 命名空间
- Promtail 以 DaemonSet 方式运行
- 每个节点一个采集实例

---

## 9. Storage Strategy

Loki 存储策略特点：

- 基于对象存储（可扩展）
- 默认本地存储

建议策略：

- 短期日志（7 天）
- 避免无限增长

示例配置：
retention_period: 168h


---

## 10. Integration With Monitoring

日志系统与监控系统深度结合：

- Grafana 统一入口
- 从指标跳转日志
- 从日志定位异常原因

形成联动分析能力：

Metrics → Logs → Root Cause

---

## 11. Advantages of Loki

相比传统 ELK：

Loki 优势：

- 资源占用低
- 无需全文索引（节省存储）
- 与 Prometheus 标签模型一致
- 与 Grafana 原生集成

适合 Kubernetes 场景。

---

## 12. Future Evolution

日志系统可进一步扩展：

- 对接对象存储（S3 / MinIO）
- 日志长期归档
- 日志分级（INFO / ERROR）
- 与告警系统联动（基于日志告警）

---

## 13. Summary

本章节设计了 Kubernetes 日志系统架构。

日志系统基于 Loki 技术栈构建，主要包括：

- Promtail（采集）
- Loki（存储）
- Grafana（展示）

实现容器日志的集中化管理与查询能力。

为后续日志采集部署与验证提供基础。
