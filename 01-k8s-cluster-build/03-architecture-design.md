# Kubernetes 平台架构设计（第一阶段）

## 一、当前架构拓扑

本阶段 Kubernetes 平台采用 kubeadm 部署方式，整体架构如下：

- 1 台 Control Plane 节点
- 3 台 Worker 节点
- etcd 采用 stacked 模式（内嵌于 Master）
- 容器运行时：containerd
- 网络插件：Calico v3.28.2

当前阶段定位为平台验证与能力构建阶段。

---

## 二、架构设计说明

### 1. 部署方式选择：kubeadm

选择 kubeadm 作为部署工具的原因：

- 官方推荐
- 结构清晰
- 有助于理解 Kubernetes 底层组件
- 适合能力训练阶段

---

### 2. 单 Master 设计说明

当前采用单控制节点架构，原因包括：

- 实验环境资源有限
- 处于能力验证阶段
- 降低复杂度

风险：

- Control Plane 单点故障
- etcd 与 Master 共存，可靠性受限

---

### 3. 网络设计

采用 Calico 作为 CNI 插件，原因：

- 支持 NetworkPolicy
- 性能较好
- 生产环境广泛使用

---

## 三、当前阶段定位

本集群属于第一阶段能力验证环境，目标为：

- 熟悉控制平面组件
- 掌握 Pod 网络模型
- 理解 Service 与 DNS 机制
- 为后续高可用改造做准备

---

## 四、存在的不足

- 无 Control Plane 高可用
- 未部署 Ingress 控制器
- 未规划持久化存储
- 未建立监控体系
- 未建立 CI/CD 流水线

---

## 五、后续演进方向

第二阶段目标：

- 引入多 Master 架构
- 独立 etcd 或外置 etcd
- 部署 Ingress 控制器
- 接入持久化存储（NFS / CSI）
- 构建监控系统（Prometheus + Grafana）

