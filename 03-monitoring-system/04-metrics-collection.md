# Metrics Collection Mechanism

## 1. Overview

Prometheus 在 Kubernetes 环境中通过 Exporter 组件采集系统与应用指标。

在本项目中，主要指标来源包括：

Node Exporter
kube-state-metrics
cAdvisor

Prometheus 通过 Pull 模式定期抓取这些组件暴露的 metrics 接口。

---

## 2. Metrics Collection Model

Prometheus 使用 Pull 模型采集监控数据。

数据采集流程：

Exporter → Prometheus → TSDB → Grafana

步骤说明：

1 Exporter 暴露 metrics HTTP 接口  
2 Prometheus 定期抓取 metrics  
3 数据存储在 Prometheus TSDB  
4 Grafana 查询并展示数据  

---

## 3. Kubernetes Monitoring Components

### Node Exporter

负责采集节点操作系统指标。

主要指标：

CPU Usage  
Memory Usage  
Disk IO  
Network Traffic  

Node Exporter 在 Kubernetes 中通常以 DaemonSet 方式运行。

---

### kube-state-metrics

负责采集 Kubernetes 对象状态。

监控对象包括：

Pod  
Deployment  
StatefulSet  
Node  

这些指标来自 Kubernetes API Server。

---

### cAdvisor

负责采集容器运行指标。

主要指标：

Container CPU  
Container Memory  
Container Filesystem  
Container Network  

cAdvisor 运行在 kubelet 内部。

---

## 4. Prometheus Service Discovery

在 Kubernetes 环境中，Prometheus 不需要手动配置监控目标。

Prometheus Operator 通过以下资源自动发现监控对象：

ServiceMonitor  
PodMonitor  

---

## 5. ServiceMonitor

ServiceMonitor 用于定义 Prometheus 需要监控的 Service。

结构示例：

apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: example-monitor
spec:
  selector:
    matchLabels:
      app: example
  endpoints:
  - port: metrics
    interval: 30s

Prometheus Operator 会自动解析该资源并加入监控目标。

---

## 6. PodMonitor

PodMonitor 用于直接监控 Pod。

适用于：

无 Service 的 Pod  
Job 类型应用  

示例：

apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: example-pod-monitor
spec:
  selector:
    matchLabels:
      app: example
  podMetricsEndpoints:
  - port: metrics

---

## 7. Default Metrics in kube-prometheus-stack

kube-prometheus-stack 默认已经配置以下监控目标：

node-exporter  
kube-state-metrics  
kubelet  
kubernetes-apiservers  

这些资源由 Prometheus Operator 自动创建。

---

## 8. Metrics Query

Prometheus 使用 PromQL 查询指标。

示例：

CPU 使用率：

rate(node_cpu_seconds_total{mode!="idle"}[5m])

内存可用量：

node_memory_MemAvailable_bytes

Pod CPU 使用率：

sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)

---

## 9. Monitoring Expansion

当新的应用部署到 Kubernetes 集群时，可以通过以下方式加入监控：

创建 ServiceMonitor  
创建 PodMonitor  
部署 Exporter  

例如：

MySQL Exporter  
Redis Exporter  
Nginx Exporter  

---

## 10. Summary

Prometheus 在 Kubernetes 中通过 Exporter 和 Operator 机制实现自动监控。

核心机制包括：

Exporter
ServiceMonitor
PodMonitor

该机制使得 Kubernetes 监控体系具有良好的扩展能力。

后续章节将介绍 Alertmanager 告警配置。
