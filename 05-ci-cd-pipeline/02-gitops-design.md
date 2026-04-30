# GitOps 设计

## 1. 什么是 GitOps

Git 作为“唯一事实源（Single Source of Truth）”

---

## 2. 核心思想

- 声明式配置
- 自动同步
- 可回滚

---

## 3. 优势

- 审计能力强
- 变更可追溯
- 支持自动回滚

---

## 4. 在本项目中的应用

- Kubernetes YAML 存储在 Git
- 部署通过 CI/CD 或 ArgoCD 触发
