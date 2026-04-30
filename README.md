# Kubernetes Private Cloud Lab

一个从 **0 到 1 构建企业级 Kubernetes 私有云平台** 的实验项目。

本项目通过完整的架构设计、部署实践与故障实验，逐步搭建一套接近企业生产环境的 Kubernetes 私有云平台。

项目不仅包含 **技术实现步骤**，还包含：

* 架构设计
* 风险分析
* 平台演进规划
* 企业级解决方案设计

适合：

* Kubernetes 学习者
* 云平台工程师
* 平台架构师
* SRE / DevOps 工程师

---

# 一、项目目标

通过本项目逐步实现一个完整的 **企业私有云平台架构**：

```
用户访问
   │
   ▼
Ingress / API Gateway
   │
   ▼
LoadBalancer（MetalLB）
   │
   ▼
Kubernetes Cluster
   │
   ├── Container Network (Calico)
   ├── Distributed Storage (Longhorn / Ceph)
   ├── Monitoring System
   ├── CI/CD Pipeline
   └── Private Cloud Platform
```

最终形成一个具备以下能力的平台：

* Kubernetes 集群管理
* 分布式存储
* 高可用架构
* 监控与告警
* 自动化 CI/CD
* 私有云解决方案

---

# 二、实验环境

实验环境基于本地虚拟化平台构建，后续计划迁移至企业 **vCenter 私有云环境**。

基础组件：

| 组件                | 版本            |
| ----------------- | ------------- |
| Kubernetes        | v1.34         |
| Container Runtime | containerd    |
| CNI               | Calico        |
| LoadBalancer      | MetalLB       |
| Ingress           | ingress-nginx |
| Storage           | Longhorn      |

节点规划：

| 节点         | IP            | 角色   |
| ---------- | ------------- | ---- |
| k8s-master | 192.168.2.201 | 控制节点 |
| k8s-node01 | 192.168.2.202 | 工作节点 |
| k8s-node02 | 192.168.2.203 | 工作节点 |
| k8s-node03 | 192.168.2.204 | 工作节点 |

---

# 三、项目结构

```
k8s-private-cloud-lab
│
├── 01-k8s-cluster-build
│   ├── 01-background.md
│   ├── 02-requirements.md
│   ├── 03-architecture-design.md
│   ├── 04-deployment-guide.md
│   ├── 05-risk-analysis.md
│   ├── 06-evolution-plan.md
│   └── troubleshooting.md
│
├── 02-storage-network
│   ├── 01-storage-architecture.md
│   ├── 02-longhorn-deploy.md
│   ├── 03-storageclass-design.md
│   ├── 04-pvc-test.md
│   ├── 05-statefulset-test.md
│   ├── 06-failure-recovery.md
│   └── 07-ceph-rook.md
│
├── 03-monitoring-system
│
├── 04-ci-cd-pipeline
│
├── 05-private-cloud-solution
│
└── README.md
```

---

# 四、模块说明

## 01 Kubernetes 集群建设

该模块主要介绍 **Kubernetes 平台建设过程**。

内容包括：

* Kubernetes 技术背景
* 私有云需求分析
* 集群架构设计
* kubeadm 部署
* 平台风险分析
* 平台演进规划

核心目标：

构建一个稳定、可扩展的 Kubernetes 集群基础平台。

---

## 02 存储与网络

该模块重点实现 **企业级存储能力**。

主要内容：

* Kubernetes 存储体系架构
* Longhorn 分布式存储部署
* StorageClass 设计
* PVC 持久化测试
* StatefulSet 有状态应用
* 节点故障恢复实验
* Ceph/Rook 存储扩展方案

实验目标：

验证 Kubernetes 在 **数据库等有状态服务场景**中的可用性。

---

## 03 监控系统

该模块将构建 Kubernetes **可观测性平台**。

计划部署组件：

* Prometheus
* Grafana
* Alertmanager

实现：

* 节点监控
* Pod 监控
* 存储监控
* 集群告警

---

## 04 CI/CD Pipeline

该模块将搭建完整的 **DevOps 自动化平台**。

计划组件：

* Harbor（私有镜像仓库）
* GitLab
* ArgoCD

实现：

* 代码构建
* 镜像管理
* 自动化部署
* GitOps

---

## 05 私有云解决方案

该模块将整合前面所有能力，形成一套完整的 **企业私有云解决方案**。

包括：

* 平台架构设计
* 高可用方案
* 容灾方案
* 多环境管理
* 企业级部署建议

---

# 五、学习路径

推荐阅读顺序：

```
01 Kubernetes 集群建设
        │
        ▼
02 存储与网络
        │
        ▼
03 监控系统
        │
        ▼
04 CI/CD Pipeline
        │
        ▼
05 私有云解决方案
```

最终形成完整的：

**Kubernetes 企业私有云平台。**

---

# 六、当前进度

当前已经完成：

* Kubernetes 集群搭建
* Calico 网络
* MetalLB 负载均衡
* Ingress 控制器
* Longhorn 分布式存储
* PVC 持久化测试
* 节点故障恢复实验

正在进行：

* StatefulSet 有状态应用实验

计划建设：

* Prometheus 监控平台
* Grafana 可视化
* DevOps CI/CD
* 私有云解决方案

---

# 七、项目目标

本项目的最终目标是构建一套完整的：

**企业 Kubernetes 私有云架构实验平台。**

通过一系列实验，深入理解：

* Kubernetes 平台架构
* 分布式存储
* 高可用设计
* 可观测性体系
* DevOps 自动化
* 私有云解决方案
