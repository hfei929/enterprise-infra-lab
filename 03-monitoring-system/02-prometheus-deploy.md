# Prometheus Deployment Guide

## 1. Overview

本章节介绍 Kubernetes 集群监控系统的部署方案。

监控系统采用 Prometheus 生态组件，通过 Helm Chart
"kube-prometheus-stack" 完成统一部署。

该 Chart 由 Prometheus Community 维护，包含以下组件：

- Prometheus Server
- Alertmanager
- Grafana
- Node Exporter
- kube-state-metrics

该方案能够快速构建完整的 Kubernetes 监控体系。

---

## 2. Deployment Architecture

监控系统部署在 Kubernetes 集群内部。

组件部署关系如下：

Kubernetes Cluster
        |
        v
+------------------------+
| kube-prometheus-stack  |
+------------------------+
| Prometheus             |
| Alertmanager           |
| Grafana                |
| Node Exporter          |
| kube-state-metrics     |
+------------------------+

Prometheus 负责指标采集与存储，
Grafana 提供可视化监控界面，
Alertmanager 负责告警处理。

---

## 3. Namespace Design

为了便于管理监控组件，创建独立 Namespace：

monitoring

创建命令：

kubectl create namespace monitoring

---

## 4. Helm Repository Configuration

添加 Prometheus 社区 Helm 仓库：

helm repo add prometheus-community \
https://prometheus-community.github.io/helm-charts

更新 Helm 仓库：

helm repo update

验证 Chart：

helm search repo kube-prometheus-stack

---

## 5. Storage Configuration

Prometheus 需要持久化存储监控指标数据。

在本实验环境中，存储使用 Longhorn StorageClass：

longhorn

Prometheus 存储配置示例：

prometheus:
  prometheusSpec:
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: longhorn
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 10Gi

数据保留时间：

15d

---

## 6. Helm Values Configuration

创建自定义 values 文件：

monitoring-values.yaml

基础配置示例：

grafana:
  adminPassword: admin123

prometheus:
  prometheusSpec:
    retention: 15d

---

## 7. Deployment Command

使用 Helm 安装监控系统：

helm install monitoring prometheus-community/kube-prometheus-stack \
-n monitoring \
-f monitoring-values.yaml

部署完成后，系统将自动创建以下组件：

- Prometheus Server
- Alertmanager
- Grafana
- Node Exporter
- kube-state-metrics

---

## 8. Deployment Verification

检查 Pod 状态：

kubectl get pods -n monitoring

正常情况下会看到类似组件：

prometheus-server
alertmanager
grafana
node-exporter
kube-state-metrics

---

## 9. Prometheus Targets Verification

进入 Prometheus Web UI 查看监控目标：

kubectl port-forward svc/monitoring-kube-prometheus-prometheus \
9090:9090 -n monitoring

浏览器访问：

http://localhost:9090

在 Targets 页面确认以下组件状态为 UP：

node-exporter
kube-state-metrics
kubernetes-apiservers

---

## 10. Grafana Access

Grafana 服务端口：

3000

访问方式：

kubectl port-forward svc/monitoring-grafana \
3000:80 -n monitoring

浏览器访问：

http://localhost:3000

默认账号：

admin

密码：

admin123

---

## 11. Deployment Result

完成部署后，Kubernetes 集群将具备以下监控能力：

节点资源监控
Pod 与容器监控
Kubernetes 对象状态监控
基础告警能力
可视化 Dashboard

该监控体系将作为 enterprise-infra-lab 项目的基础运维能力组件。

---

## 12. Next Step

下一章节将介绍 Grafana 的配置与 Dashboard 设计：

03-grafana-deploy.md
