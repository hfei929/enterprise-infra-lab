# ArgoCD 部署设计

## 1. 引入原因

- 实现 GitOps 自动部署
- 减少人工干预

---

## 2. 工作流程

Git → ArgoCD → Kubernetes

---

## 3. 与 Jenkins 区别

| 项目 | Jenkins | ArgoCD |
|------|--------|--------|
| 类型 | 推模式 | 拉模式 |
| 触发 | CI触发 | 自动同步 |

---

## 4. 本项目规划

当前阶段：
- 使用 Jenkins

后续阶段：
- 引入 ArgoCD 实现 GitOps
