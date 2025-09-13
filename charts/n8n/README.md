# n8n Helm Chart

这是一个用于在 Kubernetes 上部署 n8n 工作流自动化平台的 Helm Chart。

## 特性

- **主服务 + Worker 架构**：主服务处理 Web UI 和 API，Worker 处理工作流执行
- **外部数据库**：支持 PostgreSQL 数据库
- **队列支持**：使用 Redis 作为队列后端
- **持久化存储**：使用 NFS 共享存储
- **Gateway 路由**：支持 Gateway API 路由配置
- **可配置资源**：支持 CPU 和内存资源限制

## 安装

1. 确保你的 Kubernetes 集群已安装 Gateway API
2. 确保有可用的 NFS 存储类
3. 确保外部 PostgreSQL 数据库可用
4. 确保 Redis 服务已部署（用于队列）

```bash
# 安装 Chart
helm install n8n ./charts/n8n

# 或者使用自定义 values
helm install n8n ./charts/n8n -f custom-values.yaml
```

## 配置

### 必需配置

在安装前，请确保配置以下值：

1. **PostgreSQL 数据库**：
   ```yaml
   postgresql:
     host: postgresql.data.svc.cluster.local
     port: 5432
     database: n8n
     username: n8n
     password: "your-password"
   ```

2. **Gateway 配置**：
   ```yaml
   gateway:
     name: gateway
     namespace: default
     hosts:
       n8n: n8n.tuna.io
   ```

3. **存储配置**：
   ```yaml
   storage:
     class: nfs-shared
     size: 10Gi
   ```

### 可选配置

- **Worker 配置**：可以禁用 worker 或调整资源限制
- **队列配置**：可以禁用 Redis 队列或设置密码
- **认证配置**：可以修改默认的 admin/admin 认证

## 访问

安装完成后，可以通过以下方式访问：

- **Web UI**：https://n8n.tuna.io
- **默认用户名/密码**：admin/admin

## 架构说明

### 主服务 (Deployment)
- 运行 n8n 的核心服务
- 处理 Web UI 和 API 请求
- 模式：`main`

### Worker (DaemonSet)
- 运行 n8n worker 进程
- 处理工作流执行
- 模式：`worker`
- 每个节点运行一个实例

### 外部 Redis
- 作为队列后端
- 存储工作流执行任务
- 需要单独部署 Redis 服务

### 存储
- **n8n-data**：n8n 工作流和配置数据

## 故障排除

1. **检查 Pod 状态**：
   ```bash
   kubectl get pods -l app=n8n
   kubectl get pods -l app=n8n-worker
   ```

2. **检查日志**：
   ```bash
   kubectl logs -l app=n8n
   kubectl logs -l app=n8n-worker
   ```

3. **检查 Redis 连接**：
   ```bash
   kubectl get svc -n data | grep redis
   kubectl logs -l app=n8n | grep -i redis
   ```

4. **检查存储**：
   ```bash
   kubectl get pvc
   kubectl describe pvc n8n-data
   ```

## 升级

```bash
helm upgrade n8n ./charts/n8n
```

## 卸载

```bash
helm uninstall n8n
```

注意：卸载时不会删除 PVC，需要手动删除以释放存储空间。
