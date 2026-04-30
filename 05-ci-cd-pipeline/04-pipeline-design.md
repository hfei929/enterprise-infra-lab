# Pipeline 设计

## 1. 流水线阶段

- Checkout
- Build
- Push
- Deploy

---

## 2. 流程图（文字版）

git push
  ↓
Jenkins 拉代码
  ↓
构建镜像
  ↓
推送仓库
  ↓
更新 Kubernetes

---

## 3. 关键设计点

- 构建与部署分离
- 镜像版本化
- 可回滚

---

## 4. Jenkinsfile 结构说明

stage('Checkout')
stage('Build')
stage('Push')
stage('Deploy')
