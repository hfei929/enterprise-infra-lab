# Alertmanager Configuration

## 1. Overview

在 Kubernetes 监控系统中，Prometheus 负责检测监控指标并触发告警规则，
Alertmanager 负责对告警进行统一管理与通知。

Alertmanager 提供以下能力：

- 告警分组
- 告警去重
- 告警抑制
- 告警通知

在本项目中，Alertmanager 由 kube-prometheus-stack Helm Chart 自动部署。

---

## 2. Alert Architecture

监控系统告警架构如下：

Exporter
    |
    v
Prometheus
    |
    v
Alert Rules
    |
    v
Alertmanager
    |
    v
Notification

Prometheus 通过告警规则检测异常指标，
当条件满足时将告警发送到 Alertmanager。

---

## 3. Alertmanager Components

Alertmanager 主要功能包括：

Alert Routing

根据告警标签决定告警发送方式。

Alert Grouping

将相似告警合并，避免通知风暴。

Alert Inhibition

当某些严重告警存在时抑制次级告警。

Alert Notification

支持多种通知方式：

Email
Webhook
Slack
PagerDuty

---

## 4. Alertmanager Service

查看 Alertmanager 服务：

kubectl get svc -n monitoring

示例：

alertmanager-operated

访问方式：

kubectl port-forward svc/alertmanager-operated \
9093:9093 -n monitoring

浏览器访问：

http://localhost:9093

---

## 5. Default Alert Rules

kube-prometheus-stack 默认提供大量 Kubernetes 告警规则。

常见规则包括：

NodeDown

节点不可达。

PodCrashLooping

Pod 不断重启。

KubeNodeNotReady

节点状态异常。

KubePodNotReady

Pod 未就绪。

NodeCPUHigh

节点 CPU 使用率过高。

---

## 6. Alert Rule Structure

Prometheus 告警规则示例：

groups:
- name: example-alert
  rules:
  - alert: HighCPUUsage
    expr: rate(node_cpu_seconds_total{mode!="idle"}[5m]) > 0.9
    for: 2m
    labels:
      severity: warning
    annotations:
      description: CPU usage is higher than 90%

规则说明：

expr

PromQL 查询表达式。

for

持续时间。

labels

告警标签。

annotations

告警描述信息。

---

## 7. Alertmanager Routing

Alertmanager 根据标签进行路由。

示例：

route:
  group_by: ['alertname']
  receiver: default

receivers:
- name: default
  email_configs:
  - to: admin@example.com

---

## 8. Alert Lifecycle

告警生命周期：

1 Prometheus 检测指标异常  
2 触发告警规则  
3 发送告警到 Alertmanager  
4 Alertmanager 分组与路由  
5 发送通知  

---

## 9. Deployment Verification

验证步骤：

1 查看 Prometheus Alerts 页面  
2 查看 Alertmanager UI  
3 模拟资源异常  
4 确认告警触发  

---

## 10. Summary

Alertmanager 是 Kubernetes 监控体系中的核心告警组件。

通过 Prometheus 告警规则与 Alertmanager 路由机制，
可以实现自动化运维告警能力。

下一章节将介绍 Grafana Dashboard 设计。
