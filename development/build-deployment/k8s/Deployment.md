## Doc
### kind
```
kind: Deployment
管理pod的创建，比如创建几个pod

```
### metadata
```
metadata:
  name: backend           # 资源名称
  namespace: viralcopy    # 所属命名空间
  labels:                 # 标签（用于筛选）
    app: backend
```
### template
```
Pod 模板（描述 Pod 长什么样), POD里面也有metadata和spec

serviceAccountName: viralcopy-sa
Kubernetes 集群里有两种“用户”, User Account比如管理员通过 `kubectl` 操作集群, ServiceAccount给pod里的程序用的，Python代码要跟 Kubernetes API 对话，或者要获取 GCP 云资源的权限，就需要这个身份
  
```
### template.spec.containers
```
containers: 定义了这个pod的几个容器。
- name: backend                                    # 容器名称
  image: REGION-docker.pkg.dev/PROJECT_ID/viralcopy/backend:IMAGE_TAG  # 镜像
  imagePullPolicy: Always                          # 总是拉取最新镜像
  ports:
    - containerPort: 8000                          # 容器监听 8000 端口
  envFrom:                                         # 环境变量
    - configMapRef:
        name: viralcopy-config                     # 从 ConfigMap 读取
    - secretRef:
        name: viralcopy-secrets                    # 从 Secret 读取（敏感信息）
  livenessProbe: ...                               # 存活探针
  readinessProbe: ...                              # 就绪探针
  resources: ...                                   # 资源限制


- name: cloud-sql-proxy
          image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.11
          args:
            - "--structured-logs"
            - "--port=5432"
            - "PROJECT_ID:REGION:viralcopy-db"
          securityContext:
            runAsNonRoot: true
          resources:
            requests:
              memory: "32Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "200m"
              
- name: cloud-sql-proxy                              # 容器名称
  image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.11  # 镜像
  args:                                              # 启动参数
    - "--structured-logs"                            # 结构化日志
    - "--port=5432"                                  # 监听 5432 端口
    - "PROJECT_ID:REGION:viralcopy-db"               # 数据库实例
  securityContext:
    runAsNonRoot: true                               # 不以 root 运行
  resources:                                         # 资源限制
    requests:
      memory: "32Mi"
      cpu: "50m"
    limits:
      memory: "128Mi"
      cpu: "200m"
      
Proxy 会利用 GKE 节点的服务账号或 Workload Identity，自动与 Google Cloud 进行身份认证并获取短期有效的凭证。
```
## python sample
```
# ── Backend Deployment ────────────────────────────────────────────────────────
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: viralcopy
  labels:
    app: backend
spec:
  replicas: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: backend
    spec:
      serviceAccountName: viralcopy-sa
      containers:
        # ── FastAPI application ──────────────────────────────────────────────
        - name: backend
          image: REGION-docker.pkg.dev/PROJECT_ID/viralcopy/backend:IMAGE_TAG
          imagePullPolicy: Always
          ports:
            - containerPort: 8000
          envFrom:
            - configMapRef:
                name: viralcopy-config
            - secretRef:
                name: viralcopy-secrets
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 15
            periodSeconds: 20
            failureThreshold: 3
          readinessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 5
            periodSeconds: 10
          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "1000m"
              memory: "1Gi"

        # ── Cloud SQL Auth Proxy (sidecar) ───────────────────────────────────
        - name: cloud-sql-proxy
          image: gcr.io/cloud-sql-connectors/cloud-sql-proxy:2.11
          args:
            - "--structured-logs"
            - "--port=5432"
            - "PROJECT_ID:REGION:viralcopy-db"
          securityContext:
            runAsNonRoot: true
          resources:
            requests:
              memory: "32Mi"
              cpu: "50m"
            limits:
              memory: "128Mi"
              cpu: "200m"

```