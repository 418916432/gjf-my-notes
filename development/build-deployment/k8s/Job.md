## Doc
```
用于运行一次性任务, 一次性任务，完成后就停止
```
## sample
```
apiVersion: batch/v1          # Job 的 API 版本
kind: Job                     # 资源类型
metadata:
  name: db-migrate            # Job 名称
  namespace: viralcopy        # 命名空间
  labels:
    app: db-migrate
spec:
  ttlSecondsAfterFinished: 600  # 完成后 600 秒（10分钟）后自动删除
  backoffLimit: 3               # 失败后最多重试 3 次
  template:                     # Pod 模板（定义要运行的 Pod）
    spec:
      restartPolicy: OnFailure  # 失败时才重启
      serviceAccountName: viralcopy-sa  # 使用这个 ServiceAccount
      containers:
        - name: migrate          # 迁移容器
          image: BACKEND_IMAGE   # 使用 backend 镜像（是需要手动替换的占位符）
          command: ["alembic", "upgrade", "head"]  # 执行迁移命令
          envFrom:               # 环境变量（与 backend 相同）
            - configMapRef:
                name: viralcopy-config
            - secretRef:
                name: viralcopy-secrets

        - name: cloud-sql-proxy  # 数据库代理容器
          image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.11
          args:
            - "--structured-logs"
            - "--port=5432"
            - "SQL_CONNECTION_NAME"
          securityContext:
            runAsNonRoot: true
          resources:
            requests:
              memory: "32Mi"
              cpu: "50m"

```