# Kubernetes StatefulSet + Longhorn 实验（MySQL）

在 Kubernetes 中，大部分应用是 **无状态应用（Stateless）**。

例如：

* Web 服务
* API 服务

但数据库属于 **有状态应用（Stateful Application）**，需要保证：

```
数据持久化
固定网络身份
稳定存储
```

Kubernetes 提供 **StatefulSet** 用于运行此类服务。

本实验将使用：

```
StatefulSet + Longhorn
```

部署一个 MySQL 服务。

---

# 一、StatefulSet 特点

StatefulSet 与 Deployment 的区别：

| 特性     | Deployment | StatefulSet |
| ------ | ---------- | ----------- |
| Pod 名称 | 随机         | 固定          |
| 存储     | 共享         | 独立          |
| 启动顺序   | 无序         | 有序          |

StatefulSet Pod 命名规则：

```
mysql-0
mysql-1
mysql-2
```

每个 Pod 都拥有：

```
独立 PVC
```

---

# 二、创建 MySQL Secret

首先创建数据库密码：

```
kubectl create secret generic mysql-pass \
--from-literal=password=123456
```

查看：

```
kubectl get secret
```

---

# 三、创建 MySQL StatefulSet

创建文件：

```
mysql-statefulset.yaml
```

内容如下：

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: m.daocloud.io/docker.io/mysql:8.0
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: longhorn
      resources:
        requests:
          storage: 5Gi
```

---

# 四、创建 Service

创建 `mysql-service.yaml`：

```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
```

应用：

```
kubectl apply -f mysql-service.yaml
kubectl apply -f mysql-statefulset.yaml
```

---

# 五、验证部署

查看 Pod：

```
kubectl get pod
```

输出示例：

```
mysql-0   Running
```

查看 PVC：

```
kubectl get pvc
```

示例：

```
mysql-data-mysql-0
```

Longhorn 会自动创建对应 PV。

---

# 六、进入 MySQL

进入容器：

```
kubectl exec -it mysql-0 -- bash
```

登录数据库：

```
mysql -uroot -p
```

密码：

```
123456
```

创建测试数据库：

```
CREATE DATABASE testdb;
SHOW DATABASES;
```

---

# 七、验证数据持久化

删除 Pod：

```
kubectl delete pod mysql-0
```

Kubernetes 会自动重建：

```
mysql-0
```

再次进入数据库：

```
SHOW DATABASES;
```

如果看到：

```
testdb
```

说明：

```
数据没有丢失
```

---

# 八、实验总结

本实验验证了：

```
StatefulSet + Longhorn
```

可以为 Kubernetes 提供：

* 数据持久化
* 独立存储
* 自动恢复

这是运行数据库、缓存、消息队列等服务的基础。

---

# 九、下一步实验

接下来将进行 **节点故障恢复实验**：

```
06-failure-recovery.md
```

验证：

```
节点宕机
Pod 迁移
数据不丢
```

实现真正的 **企业级高可用存储平台**。

