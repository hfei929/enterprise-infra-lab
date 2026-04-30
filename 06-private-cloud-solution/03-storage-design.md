````markdown
# 03-storage-design.md

# 从0到企业级私有云：存储设计方案（数据底座篇）

## 一、为什么私有云必须重视存储设计

在企业级私有云平台中，计算资源负责运行业务，网络负责连接业务，而 **存储负责承载业务数据**。

如果存储设计不合理，常见结果通常是：

- Jenkins 重启后任务数据丢失
- Harbor 镜像仓库容量爆满
- 数据库 Pod 重建后数据消失
- PVC 长期 Pending
- IO 性能不足导致业务卡顿
- 备份缺失导致无法恢复
- 节点故障后数据不可用

因此，私有云建设时，存储体系必须作为核心基础能力进行规划。

---

# 二、企业级私有云存储总体分层

建议采用分层设计：

```text
┌────────────────────────────┐
│ 应用数据层                 │
│ MySQL / Redis / ES / MQ    │
└────────────────────────────┘

┌────────────────────────────┐
│ 平台组件层                 │
│ Jenkins / Harbor / GitLab  │
└────────────────────────────┘

┌────────────────────────────┐
│ Kubernetes 存储抽象层      │
│ PV / PVC / StorageClass    │
└────────────────────────────┘

┌────────────────────────────┐
│ 后端存储层                 │
│ LocalPV / NFS / Ceph       │
└────────────────────────────┘

┌────────────────────────────┐
│ 物理磁盘层                 │
│ SSD / SATA / RAID / SAN    │
└────────────────────────────┘
````

---

# 三、Kubernetes 存储核心概念

## PV（PersistentVolume）

集群中的持久卷资源。

## PVC（PersistentVolumeClaim）

业务申请存储空间。

## StorageClass

定义动态供给策略。

## 关系模型

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Backend Storage
```

---

# 四、常见存储方案对比

| 方案       | 特点      | 是否推荐      |
| -------- | ------- | --------- |
| hostPath | 本地目录挂载  | 实验临时使用    |
| LocalPV  | 本地磁盘高性能 | 单机业务可用    |
| NFS      | 简单共享存储  | 实验/中小企业推荐 |
| Ceph     | 分布式高可用  | 企业生产推荐    |
| SAN/NAS  | 传统企业存储  | 视预算而定     |

---

# 五、实验环境最佳方案（推荐你当前使用）

## 三节点环境建议：

```text
master01
worker01
worker02
```

推荐使用：

```text
NFS + StorageClass
```

原因：

* 部署简单
* 多节点共享
* 支持 Jenkins / Harbor / Grafana
* 易维护
* 学习成本低

---

# 六、NFS 存储架构示意

```text
Jenkins PVC ─┐
Harbor PVC  ├──> NFS Server
Grafana PVC ┤
Logs PVC    ┘
```

---

# 七、NFS 推荐规划

假设 NFS 服务器：

```text
192.168.10.50
/data/nfs
```

目录建议：

```text
/jenkins
/harbor
/grafana
/backup
/logs
```

---

# 八、生产环境推荐方案：Ceph

如果进入企业级生产环境，建议采用：

```text
Rook-Ceph
```

可提供：

* RBD（块存储）
* CephFS（文件共享）
* S3 Object（对象存储）

优势：

* 高可用
* 横向扩容
* 多副本容灾
* Kubernetes 原生集成

---

# 九、平台组件存储需求建议

| 组件         | 是否需要持久化 | 建议容量  |
| ---------- | ------- | ----- |
| Jenkins    | 是       | 20G+  |
| Harbor     | 是       | 200G+ |
| Grafana    | 是       | 10G   |
| Prometheus | 是       | 50G+  |
| Loki       | 是       | 100G+ |
| MySQL      | 是       | 按业务规划 |

---

# 十、StorageClass 设计建议

建议至少区分两类：

## 标准型

```text
standard
```

用于：

* Jenkins
* Grafana
* 普通业务

---

## 高性能型

```text
fast-ssd
```

用于：

* MySQL
* Redis
* 高频 IO 服务

---

# 十一、访问模式设计（重要）

| 模式  | 含义    |
| --- | ----- |
| RWO | 单节点读写 |
| ROX | 多节点只读 |
| RWX | 多节点读写 |

推荐：

* Jenkins：RWX
* Harbor：RWX
* MySQL：RWO
* Redis：RWO

---

# 十二、数据备份设计（必须有）

企业级平台不能只有存储，没有备份。

建议：

## 平台组件备份

```text
Jenkins Home
Harbor Data
Grafana SQLite
```

每日备份。

---

## 业务数据备份

```text
MySQL dump
Redis RDB/AOF
对象存储快照
```

---

## 备份目标位置

* 独立 NAS
* 备份服务器
* 异地对象存储

---

# 十三、容量规划建议

## 初期实验环境

```text
总容量：500G~1T
```

## 小型企业

```text
2T~10T
```

## 中大型企业

```text
10T+
```

需按增长率规划。

---

# 十四、常见故障场景

## 1. PVC Pending

原因：

* 无默认 StorageClass
* Provisioner 异常
* 容量不足

---

## 2. Pod 卡在 ContainerCreating

原因：

* 挂载失败
* NFS 权限异常

---

## 3. Harbor 推镜像失败

原因：

* 磁盘满
* inode 耗尽

---

## 4. Prometheus 崩溃

原因：

* TSDB 数据过大
* IO 性能差

---

# 十五、实验环境最佳实践

推荐：

```text
Debian + NFS Server
nfs-subdir-external-provisioner
default StorageClass
每日 rsync 备份
```

---

# 十六、企业生产最佳实践

推荐：

```text
SSD RAID10
独立存储网络
Rook-Ceph
冷热数据分层
快照备份
异地容灾
```

---

# 十七、总结

私有云的真正价值，不只是把应用跑起来，而是 **让数据稳定、安全、可恢复地长期运行**。

存储设计决定：

* 数据是否可靠
* 平台是否稳定
* 运维是否轻松
* 故障后是否能恢复

很多平台不是死于 Kubernetes，而是死于存储。

---

# 下一章预告

```text
04-security-design.md
```

企业级私有云安全体系设计（RBAC / Secret / 网络隔离 / 审计日志）

```
```

