# 部署策略

## 1. 滚动更新（Rolling Update）

逐步替换 Pod，保证服务不中断

---

## 2. 蓝绿部署（Blue-Green）

两套环境切换

---

## 3. 灰度发布（Canary）

小流量验证

---

## 4. 回滚策略

- 使用旧镜像
- kubectl rollout undo

---

## 5. 本项目当前使用

当前：Rolling Update  
后续：引入 Canary
