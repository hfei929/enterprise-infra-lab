````markdown
# 06-deployment-implementation.md

# 从0到企业级私有云：完整落地实施手册（最终实战修正版）

## 一、章节说明

前五章已经完成私有云平台整体设计：

```text
01-overall-architecture.md
02-network-design.md
03-storage-design.md
04-security-design.md
05-operations.md
````

本章目标是将整体方案真正落地，建设一套包含 **代码管理、持续集成、镜像仓库、GitOps 发布、容器平台、可观测体系** 的企业级私有云平台。

---

# 二、实施目标

基于三台服务器：

```text
master01
worker01
worker02
```

最终建设完成：

* Kubernetes 集群
* Gitea 私有代码仓库
* Jenkins CI 平台
* Harbor 镜像仓库
* ArgoCD GitOps 平台
* Prometheus + Grafana
* Loki 日志平台
* 企业级基础治理体系

---

# 三、完整平台链路（核心）

```text
开发人员提交代码
        ↓
      Gitea
        ↓
Webhook 触发 Jenkins
        ↓
构建镜像
        ↓
Push Harbor
        ↓
更新 GitOps Repo
        ↓
ArgoCD 自动同步
        ↓
Kubernetes 发布业务
        ↓
Prometheus / Grafana 监控
```

这是一条完整企业交付链路。

---

# 四、实施阶段划分

```text
阶段一：服务器初始化
阶段二：Kubernetes 集群建设
阶段三：代码与交付平台建设
阶段四：可观测平台建设
阶段五：安全治理建设
阶段六：业务验证上线
```

---

# 五、阶段一：服务器初始化

节点规划：

```text
master01   192.168.10.11
worker01   192.168.10.12
worker02   192.168.10.13
```

基础操作：

* Debian 最小化安装
* 固定 IP
* SSH Key 登录
* 时间同步
* hosts 配置
* 关闭 swap
* 开启 IP Forward

检查：

```bash
hostnamectl
ip a
free -h
timedatectl
```

---

# 六、阶段二：Kubernetes 集群建设

安装：

```text
containerd
kubeadm
kubelet
kubectl
```

初始化：

```bash
kubeadm init \
--apiserver-advertise-address=192.168.10.11 \
--pod-network-cidr=10.244.0.0/16
```

工作节点加入：

```bash
kubeadm join ...
```

安装网络插件：

```text
Calico
```

验收：

```bash
kubectl get nodes
kubectl get pods -A
```

目标：

```text
3 节点 Ready
```

---

# 七、阶段三：代码与交付平台建设

# 1. Gitea 私有 Git 平台

用途：

* 代码托管
* Webhook 触发 CI
* Helm/YAML 仓库存储
* 权限管理

访问地址：

```text
git.company.local
```

建议持久化 PVC。

---

# 2. Jenkins 持续集成平台

用途：

* 拉取 Gitea 代码
* 编译构建
* Docker 镜像构建
* 推送 Harbor

建议：

```text
Kubernetes Agent Pod 动态构建
```

---

# 3. Harbor 镜像仓库

用途：

* 企业镜像统一管理
* 多项目权限控制
* 镜像版本管理

地址：

```text
harbor.company.local
```

---

# 4. ArgoCD GitOps 平台

用途：

* Git 为事实源
* 自动部署
* 自动回滚
* 环境统一管理

地址：

```text
argocd.company.local
```

---

# 八、完整发布流程（真实企业模式）

```text
开发修改代码
   ↓
Push 到 Gitea
   ↓
Webhook 通知 Jenkins
   ↓
Jenkins Pipeline 执行
   ↓
构建镜像并推送 Harbor
   ↓
修改部署仓库 Tag
   ↓
ArgoCD 检测 Git 变更
   ↓
同步到 Kubernetes
   ↓
新版本上线
```

---

# 九、阶段四：可观测平台建设

部署：

```text
Prometheus
Grafana
Loki
Promtail
Alertmanager
```

命名空间建议：

```text
monitoring
logging
```

实现能力：

* 节点监控
* Pod 状态监控
* 应用监控
* 日志集中查询
* 企业微信告警

---

# 十、阶段五：安全治理建设

Namespace：

```text
dev
test
prod
cicd
monitoring
logging
```

权限：

```text
viewer
developer
ops
admin
```

其他：

* Ingress HTTPS
* Secret 管理
* RBAC
* Harbor 权限控制

---

# 十一、最终平台结构图

```text
开发平台：
Gitea

交付平台：
Jenkins
Harbor
ArgoCD

运行平台：
Kubernetes

运维平台：
Prometheus
Grafana
Loki

治理平台：
RBAC
Namespace
Quota
```

---

# 十二、最终验收清单

## 基础平台

* [ ] Kubernetes 三节点正常

## 开发平台

* [ ] Gitea 可提交代码

## 交付平台

* [ ] Jenkins 自动构建成功
* [ ] Harbor 镜像推送成功
* [ ] ArgoCD 自动发布成功

## 运维平台

* [ ] Grafana 仪表盘正常
* [ ] Loki 日志可查询
* [ ] 告警正常发送

## 治理平台

* [ ] RBAC 生效
* [ ] HTTPS 生效

---

# 十三、常见踩坑总结

## Gitea Webhook 不触发

原因：

* Jenkins 地址错误
* 网络不通
* Token 错误

---

## Harbor 推送失败

原因：

* 凭据错误
* 仓库不存在
* HTTPS 证书异常

---

## ArgoCD 不同步

原因：

* Repo 凭据错误
* YAML 错误
* 权限不足

---

## PVC Pending

原因：

* StorageClass 缺失

---

# 十四、你的实验已达到什么水平

完成本套实践后，你已经具备：

* 私有云平台建设能力
* DevOps 平台建设能力
* GitOps 发布能力
* 企业运维平台建设能力
* 平台治理能力

这已经明显超出普通运维初级水平。

---

# 十五、下一阶段演进路线

建议继续：

```text
07 三环境发布体系（dev/test/prod）
08 灰度发布（Argo Rollouts）
09 多集群治理
10 成本优化（FinOps）
11 高可用灾备体系
```

---

# 十六、总结

很多人只会部署 Kubernetes。

但真正企业级平台，需要打通：

```text
代码 → 构建 → 镜像 → 发布 → 运行 → 监控 → 治理
```

而你们这套实验，已经完成了这条完整链路。

这不是单纯实验，而是一套真实企业平台雏形。

---

```
```

