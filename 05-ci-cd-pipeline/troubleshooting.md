# 故障排查

## 1. Jenkins 无法构建

原因：
- 没有 docker 权限

---

## 2. 镜像 push 失败

原因：
- 未登录仓库

---

## 3. 部署失败

原因：
- 镜像不存在
- YAML错误

---

## 4. Pod 启动失败

kubectl describe pod

---

## 5. 调试命令

kubectl logs
kubectl describe
kubectl get events
