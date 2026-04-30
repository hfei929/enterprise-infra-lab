````markdown id="n2g8vd"
# 04-security-design.md

# 从0到企业级私有云：安全设计方案（治理防线篇）

## 一、为什么私有云必须优先考虑安全

很多团队建设私有云时，重点放在：

- Kubernetes 集群部署
- CI/CD 自动化
- 监控告警平台
- 应用快速上线

但企业真实环境里，**安全体系往往比功能建设更重要**。

如果缺乏安全设计，常见问题包括：

- 开发人员误删生产资源
- 任意人员可访问集群 API
- Harbor 镜像被篡改
- Pod 横向访问其他业务
- Secret 明文泄露
- 节点被入侵后横向扩散
- 无操作审计记录

因此，私有云建设必须同时具备 **交付能力 + 安全治理能力**。

---

# 二、企业级安全总体架构

建议采用纵深防御模型：

```text
┌────────────────────────────┐
│ 访问控制层                 │
│ SSO / MFA / VPN / Bastion  │
└────────────────────────────┘

┌────────────────────────────┐
│ 平台权限层                 │
│ RBAC / Namespace 隔离      │
└────────────────────────────┘

┌────────────────────────────┐
│ 工作负载安全层             │
│ Pod Security / Policy      │
└────────────────────────────┘

┌────────────────────────────┐
│ 网络安全层                 │
│ NetworkPolicy / TLS        │
└────────────────────────────┘

┌────────────────────────────┐
│ 数据安全层                 │
│ Secret / Backup / Encrypt  │
└────────────────────────────┘

┌────────────────────────────┐
│ 审计追踪层                 │
│ Audit Log / SIEM / Logs    │
└────────────────────────────┘
````

---

# 三、身份认证设计（谁能登录）

## 企业推荐接入：

* LDAP
* Active Directory
* OAuth2
* OIDC
* SSO 单点登录

---

## 实验环境建议

```text id="tn0j2u"
本地账号 + 强密码 + SSH Key
```

避免：

```text id="u2kp6d"
root/root123
admin/admin
```

---

# 四、Kubernetes RBAC 权限设计（极重要）

RBAC 是集群安全核心。

## 推荐角色划分

| 角色        | 权限范围               |
| --------- | ------------------ |
| viewer    | 只读查看               |
| developer | dev/test namespace |
| qa        | test 环境操作          |
| ops       | 发布与运维              |
| admin     | 集群管理               |

---

## 示例权限边界

```text id="2wwj5g"
开发人员：
只能访问 dev namespace

测试人员：
只能访问 test namespace

运维人员：
可访问 prod namespace
```

---

# 五、Namespace 隔离设计

建议按环境隔离：

```text id="r1b2g9"
dev
test
prod
monitoring
logging
cicd
```

好处：

* 权限清晰
* 配额清晰
* 故障隔离
* 审计方便

---

# 六、Secret 凭据安全设计

企业平台常见敏感信息：

* 数据库密码
* Harbor 凭据
* API Token
* TLS 证书
* Jenkins 凭据

---

## 禁止做法

```text id="a4tt2v"
password: 123456
写入 yaml 文件提交 Git
```

---

## 推荐做法

* Kubernetes Secret
* External Secrets
* Vault
* 密钥定期轮换

---

# 七、镜像安全设计

镜像是供应链入口，必须治理。

## 建议策略：

* Harbor 私有仓库统一管理
* 禁止使用 latest 标签
* 固定版本号
* 漏洞扫描
* 镜像签名（进阶）

---

## 推荐规范

```text id="7v4mre"
app:v1.0.0
app:v1.0.1
```

避免：

```text id="yo55o7"
app:latest
```

---

# 八、节点安全加固

Kubernetes 节点即服务器，需按 Linux 安全基线处理。

## Debian 建议：

* 禁止 root 远程登录
* 使用 sudo
* SSH Key 登录
* 修改默认端口（可选）
* 启用防火墙
* 自动安全更新
* 时间同步

---

## 必要检查项

```text id="wbh7hn"
ufw status
ss -tunlp
last
journalctl
```

---

# 九、网络安全设计

## Pod 间默认互通并不安全

建议启用：

```text id="8h2c6r"
NetworkPolicy
```

实现：

* A 服务仅访问 B 服务
* 禁止跨命名空间访问
* 限制数据库仅被业务访问

---

## Ingress 安全

建议：

* HTTPS/TLS
* HSTS
* 来源 IP 限制
* WAF（生产）

---

# 十、Pod 安全设计

建议限制：

* 禁止 privileged
* 禁止 hostNetwork
* 禁止 hostPath 滥用
* 禁止 root 用户运行
* 只读文件系统（进阶）

---

## 示例目标

```text id="mxz6we"
runAsNonRoot: true
```

---

# 十一、审计日志设计（企业很看重）

需要知道：

* 谁删除了 Deployment
* 谁修改了 Secret
* 谁登录了 Jenkins
* 谁发布了生产版本

---

## 建议开启：

```text id="n8d1u4"
Kubernetes Audit Log
Harbor Audit
Jenkins Build History
Linux auth.log
```

---

# 十二、CI/CD 安全设计

流水线也是高风险区域。

建议：

* Jenkins 最小权限运行
* 凭据统一 Credential 管理
* Agent 临时 Pod
* Pipeline 审批流程
* Prod 发布需人工确认

---

# 十三、备份与恢复安全

安全不仅是防攻击，也包括防误删。

建议：

* etcd 定期备份
* YAML 配置 Git 保存
* PVC 数据备份
* Harbor 镜像备份

---

# 十四、实验环境最佳实践（你的场景）

推荐：

```text id="fyd4qx"
RBAC 基础启用
dev/test/prod namespace
Harbor 私库
SSH Key 登录
Grafana 强密码
Ingress HTTPS
```

---

# 十五、企业生产最佳实践

推荐：

```text id="1kszq6"
LDAP/AD 登录
MFA
堡垒机
Vault
镜像扫描
NetworkPolicy 全面启用
审计平台接入
发布审批流
```

---

# 十六、常见安全事故场景

## 1. 开发误删生产 Deployment

原因：

权限过大。

---

## 2. Secret 上传 GitHub

原因：

凭据管理缺失。

---

## 3. latest 镜像导致回滚失败

原因：

版本不可追踪。

---

## 4. Pod 被入侵横向扫描

原因：

无 NetworkPolicy。

---

# 十七、总结

企业私有云的成熟度，不在于装了多少组件，而在于：

* 权限是否可控
* 发布是否可追踪
* 数据是否安全
* 网络是否隔离
* 故障是否可审计

安全体系不是附加项，而是平台基线。

---

# 下一章预告

```text id="8muh8k"
05-operations.md
```

企业级私有云运维体系设计（监控 / 日志 / 告警 / 备份 / 升级 / SOP）

```
```

