# SLI / SLO & Alerting

## 1. Background

在前面的章节中，我们已经完成：

- Metrics（Prometheus）
- Logs（Loki）
- Tracing（Tempo）
- APM（Service Map）

系统已经具备完整的可观测能力。

但仍然存在一个关键问题：

- 如何定义“系统是否健康”？
- 如何衡量“用户体验是否达标”？
- 告警应该基于什么触发？

传统监控通常基于资源指标：

- CPU 使用率
- 内存使用率

但这些指标无法直接反映用户体验。

因此，需要引入 SLI / SLO 体系。

---

## 2. Objectives

本章节目标：

1. 理解 SLI / SLO 概念
2. 定义系统可用性指标
3. 构建基于用户体验的监控模型
4. 实现 Prometheus 告警规则
5. 构建可用性驱动的告警体系

---

## 3. Core Concepts

### SLI（Service Level Indicator）

服务级指标，用于衡量系统表现。

常见指标：

- 请求成功率
- 请求延迟（Latency）
- 错误率（Error Rate）

---

### SLO（Service Level Objective）

服务级目标，是对 SLI 的约束。

例如：

- 成功率 ≥ 99.9%
- 请求延迟 < 200ms

---

### SLA（Service Level Agreement）

服务级协议，通常是对外承诺。

本实验主要关注 SLI / SLO。

---

## 4. SLI Design

常见 SLI 指标设计：

### 1）成功率（Availability）

```text
成功率 = 成功请求数 / 总请求数
2）错误率（Error Rate）
错误率 = 失败请求数 / 总请求数
3）延迟（Latency）

例如：

P95 延迟
P99 延迟
5. SLO Example

示例目标：

月可用性 ≥ 99.9%

允许不可用时间：

≈ 43 分钟 / 月
6. Prometheus Metrics Requirement

实现 SLO 前提：

必须有以下指标：

http_requests_total

标签示例：

status（HTTP 状态码）
service
method
7. Prometheus Alert Rules
7.1 高错误率告警
groups:
- name: slo-alert
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) 
          / rate(http_requests_total[5m]) > 0.05
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate > 5% in last 5 minutes"
7.2 高延迟告警
- alert: HighLatency
  expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 0.5
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "High latency detected"
7.3 服务不可用
- alert: ServiceDown
  expr: up == 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "Service is down"
8. Alert Strategy

告警策略建议：

分级
critical（严重）
warning（警告）
去噪

避免：

瞬时波动触发告警
告警风暴
结合 SLO

告警应基于：

用户体验
服务质量

而不是单纯资源指标。

9. Error Budget（进阶）

Error Budget：

允许失败的请求比例

例如：

SLO = 99.9%
Error Budget = 0.1%

用途：

控制发布频率
决定是否暂停发布
10. Integration With Observability

形成完整闭环：

Metrics → Alert → Tracing → Logs

流程：

Prometheus 触发告警
定位异常指标
跳转 Trace 分析
查看日志定位根因
11. Common Issues
1）没有业务指标

无法定义 SLI。

2）告警过多

原因：

阈值不合理
无去噪机制
3）误报

原因：

未设置 for 时间
未做平滑处理
12. Optimization
1）合理阈值

根据实际业务调整。

2）窗口时间

避免短时间波动：

5m / 10m
3）多指标结合

避免单一指标误判。

13. Validation Checklist

完成以下验证：

定义 SLI 指标
设置 SLO 目标
Prometheus 告警规则生效
Alertmanager 能接收告警
告警可触发
14. Summary

本章节完成了 SLI / SLO 与告警体系设计。

实现能力：

用户体验驱动监控
可用性指标定义
告警体系优化

标志着系统从：

“监控资源”

升级为：

“保障服务质量”

Observability 阶段完成。
