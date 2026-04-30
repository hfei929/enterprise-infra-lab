# Kubernetes 存储体系架构设计

在企业 Kubernetes 平台中，计算、网络、存储是三大核心能力。其中 **存储层**负责为应用提供持久化数据能力，是运行数据库、消息队列、日志系统等核心服务的基础。

本实验环境将通过部署 Longhorn 构建一套 **Kubernetes 分布式存储体系**，并逐步验证其高可用能力。

---

# 一、Kubernetes 为什么需要存储

Kubernetes 中的 Pod 默认是 **无状态的**。

例如：

```
Pod 删除 → 数据丢失
Pod 重建 → 文件不存在
```

这对于 Web 应用问题不大，但对于以下服务则无法接受：

* MySQL
* PostgreSQL
* Redis
* Elasticsearch
* Kafka

因此 Kubernetes 需要提供 **持久化存储机制**。

---

# 二、Kubernetes 存储模型

Kubernetes 通过三个核心对象管理存储：

```
StorageClass
      │
      ▼
      PVC
      │
      ▼
      PV
```

### 1 StorageClass

定义 **存储类型**，例如：

* SSD
* HDD
* 分布式存储
* 云盘

示例：

```
StorageClass: longhorn
```

---

### 2 PVC（PersistentVolumeClaim）

PVC 是 **用户申请存储的请求**。

例如：

```
申请 2GB 存储
```

示例：

```
resources:
  requests:
    storage: 2Gi
```

---

### 3 PV（PersistentVolume）

PV 是 **实际的存储资源**。

PVC 创建后：

```
PVC → 自动绑定 PV
```

---

# 三、CSI 存储插件架构

Kubernetes 通过 **CSI（Container Storage Interface）** 接入不同存储系统。

架构如下：

```
Kubernetes
     │
     ▼
 CSI Driver
     │
     ▼
 Storage System
```

常见 CSI 存储：

* Longhorn
* Ceph
* NFS
* AWS EBS
* Azure Disk

---

# 四、Longhorn 分布式存储

本实验选择 **Longhorn** 作为 Kubernetes 存储系统。

Longhorn 是一个：

```
云原生分布式块存储
```

特点：

* 完全运行在 Kubernetes 内
* 自动副本
* 自动恢复
* Web UI 管理
* 快照和备份

---

# 五、Longhorn 存储架构

Longhorn 通过 **多副本机制**保证数据安全。

架构示意：

```
                Kubernetes
                     │
                     ▼
                StorageClass
                     │
                     ▼
                     PVC
                     │
                     ▼
                     PV
                     │
                     ▼
               Longhorn Engine
                     │
         ┌───────────┼───────────┐
         │           │           │
      Replica     Replica     Replica
      node01       node02       node03
```

数据写入流程：

```
应用写入
   │
   ▼
Longhorn Engine
   │
   ▼
同步写入多个 Replica
```

即使某个节点宕机：

```
副本仍然存在
```

系统会自动：

```
重建副本
```

---

# 六、企业级高可用设计

企业 Kubernetes 存储必须满足：

### 1 高可用

```
节点宕机
磁盘损坏
网络故障
```

数据仍然存在。

---

### 2 自动恢复

例如：

```
Replica node02 Down
```

系统自动：

```
Rebuild Replica
```

---

### 3 动态扩容

PVC 支持在线扩容：

```
2Gi → 10Gi
```

---

# 七、本实验的存储架构

当前实验环境：

```
节点数量：4
控制节点：1
工作节点：3
```

存储部署在 **所有工作节点**。

```
                 Kubernetes Cluster
                         │
                         ▼
                      Longhorn
                         │
          ┌──────────────┼──────────────┐
          │              │              │
      k8s-node01     k8s-node02     k8s-node03
         disk            disk            disk
```

Longhorn 默认创建：

```
3 副本
```

---

# 八、实验验证目标

接下来的实验将验证以下能力：

1 PVC 持久化存储
2 Pod 删除数据不丢
3 StatefulSet 数据持久化
4 节点故障数据恢复

最终实现：

```
企业级 Kubernetes 存储平台
```

---

# 九、下一步实验

下一篇将部署 Longhorn 并完成存储测试：

```
02-longhorn-deploy.md
```

