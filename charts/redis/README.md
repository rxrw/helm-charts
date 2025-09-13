# Redis Helm Chart

这是一个用于在 Kubernetes 上部署 Redis 内存数据库的 Helm Chart。

## 特性

- **持久化存储**：支持数据持久化
- **安全配置**：支持密码认证
- **资源限制**：可配置的 CPU 和内存限制
- **健康检查**：内置存活性和就绪性探针
- **配置管理**：通过 ConfigMap 管理 Redis 配置

## 安装

```bash
# 安装 Redis Chart
helm install redis ./charts/redis

# 或者使用自定义 values
helm install redis ./charts/redis -f custom-values.yaml
```

## 配置

### 基本配置

```yaml
# 命名空间
namespace: data

# 镜像配置
image:
  repository: redis
  tag: "7-alpine"
  pullPolicy: IfNotPresent

# 端口配置
port: 6379

# 存储配置
storage:
  class: nfs-shared
  size: 1Gi

# Redis 配置
redis:
  password: ""  # 设置密码（可选）
  persistence:
    enabled: true
  appendonly: "yes"
```

### 安全配置

```yaml
# 设置 Redis 密码
redis:
  password: "your-secure-password"

# 安全上下文
securityContext:
  runAsUser: 999
  runAsGroup: 999
  fsGroup: 999
```

### 资源限制

```yaml
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

## 使用

### 连接 Redis

从其他 Pod 连接 Redis：

```bash
# 使用 redis-cli 连接
kubectl exec -it <redis-pod> -- redis-cli

# 如果设置了密码
kubectl exec -it <redis-pod> -- redis-cli -a <password>
```

### 从应用程序连接

```yaml
# 连接字符串
redis://redis.data.svc.cluster.local:6379

# 带密码的连接字符串
redis://:password@redis.data.svc.cluster.local:6379
```

## 监控

### 检查 Pod 状态

```bash
kubectl get pods -l app=redis -n data
```

### 查看日志

```bash
kubectl logs -l app=redis -n data
```

### 检查存储

```bash
kubectl get pvc -n data
kubectl describe pvc redis-data -n data
```

## 升级

```bash
helm upgrade redis ./charts/redis
```

## 卸载

```bash
helm uninstall redis
```

注意：卸载时不会删除 PVC，需要手动删除以释放存储空间。

## 故障排除

1. **Pod 无法启动**：
   - 检查存储类是否可用
   - 检查资源限制是否合理
   - 查看 Pod 事件和日志

2. **连接失败**：
   - 检查 Service 是否正常
   - 检查网络策略
   - 验证密码配置

3. **数据丢失**：
   - 检查 PVC 状态
   - 验证持久化配置
   - 检查存储类配置
