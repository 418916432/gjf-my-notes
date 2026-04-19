## Doc
### 元数据 (metadata)
```
metadata:
  name: viralcopy-ingress           # Ingress 名称
  namespace: viralcopy              # 命名空间
  annotations:                      # GCP 特定配置
    kubernetes.io/ingress.class: "gce"     # 使用 GCE Load Balancer
    kubernetes.io/ingress.global-static-ip-name: "viralcopy-ip"  # 固定 IP
    networking.gke.io/managed-certificates: "viralcopy-ssl"      # SSL 证书
    kubernetes.io/ingress.allow-http: "true"   # 允许 HTTP（自动跳转 HTTPS）
    
kubernetes.io/ingress.class: "gce" 
使用 GCP 的 HTTP(S) Load Balancer

global-static-ip-name 
使用固定公网 IP（不会变）

managed-certificates
自动管理 SSL 证书

allow-http: "true"
允许 HTTP 访问（自动重定向到 HTTPS）
```
### annotations
```
Annotation 是 Kubernetes 资源的"附加说明/配置"，用于扩展资源的功能，特别是云服务商特有的功能.

比如切换到GCE Load Balancer kubernetes.io/ingress.class: "gce"。默认是nginx-ingress

```
### 路由规则 (rules)
```
spec:
  rules:
    - host: 34-107-167-235.sslip.io    # 域名（sslip.io 解析到 IP）
      http:
        paths:
          # 后端 API 路由
          - path: /api/*
            backend:
              service:
                name: backend
                port:
                  number: 8000
          
          # 健康检查路由
          - path: /health
            backend:
              service:
                name: backend
                port:
                  number: 8000
          
          # OpenAPI 文档
          - path: /openapi.json
            backend:
              service:
                name: backend
                port:
                  number: 8000
          
          # 前端路由（所有其他路径）
          - path: /*
            backend:
              service:
                name: frontend
                port:
                  number: 3000
```
### ManagedCertificate 配置
```
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: viralcopy-ssl
  namespace: viralcopy
spec:
  domains:
    - 34-107-167-235.sslip.io    # 为这个域名申请 SSL 证书
```
## python sample
### Ingress
```
# ── GKE Ingress with HTTPS via custom domain viralcopy.cc ─────────────────────
# Domain: viralcopy.cc  →  DNS A record → 34.107.167.235 (GKE global static IP)
# Frontend: https://viralcopy.cc/
# Backend:  https://viralcopy.cc/api/* and /health
#
# IP: 34.107.167.235 is a GKE-managed global static IP (auto-reserved, stable).
# ManagedCertificate provisions TLS automatically once DNS propagates (15-30 min).
#
# DNS setup (in your domain registrar):
#   A record:  viralcopy.cc  →  34.107.167.235
#   A record:  www.viralcopy.cc → 34.107.167.235  (optional)
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: viralcopy-ingress
  namespace: viralcopy
  annotations:
    kubernetes.io/ingress.class: "gce"
    networking.gke.io/managed-certificates: "viralcopy-ssl-cc"
    kubernetes.io/ingress.allow-http: "false"
    # ExternalDNS: auto-update Alibaba Cloud DNS A record when IP changes
    external-dns.alpha.kubernetes.io/hostname: "viralcopy.cc"
spec:
  rules:
    - host: viralcopy.cc
      http:
        paths:
          # Backend routes — API, health, OpenAPI spec
          - path: /api/*
            pathType: ImplementationSpecific
            backend:
              service:
                name: backend
                port:
                  number: 8000
          - path: /health
            pathType: ImplementationSpecific
            backend:
              service:
                name: backend
                port:
                  number: 8000
          - path: /openapi.json
            pathType: ImplementationSpecific
            backend:
              service:
                name: backend
                port:
                  number: 8000
          # Frontend — everything else
          - path: /*
            pathType: ImplementationSpecific
            backend:
              service:
                name: frontend
                port:
                  number: 3000
  defaultBackend:
    service:
      name: frontend
      port:
        number: 3000
---
apiVersion: networking.gke.io/v1
kind: ManagedCertificate
metadata:
  name: viralcopy-ssl-cc
  namespace: viralcopy
spec:
  domains:
    - viralcopy.cc

```
### external-dns
```
# ExternalDNS — watches Ingress resources and syncs A records to Alibaba Cloud DNS
# Credentials are stored in Secret "external-dns-alicloud" (not committed to git)
# When the GKE Ingress IP changes, ExternalDNS auto-updates viralcopy.cc DNS

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-dns
  namespace: viralcopy

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: external-dns
rules:
  - apiGroups: [""]
    resources: ["services", "endpoints", "pods", "nodes"]
    verbs: ["get", "watch", "list"]
  - apiGroups: ["extensions", "networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["get", "watch", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: external-dns
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
  - kind: ServiceAccount
    name: external-dns
    namespace: viralcopy

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: viralcopy
  labels:
    app: external-dns
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      containers:
        - name: external-dns
          image: registry.k8s.io/external-dns/external-dns:v0.14.2
          args:
            - --source=ingress
            - --domain-filter=viralcopy.cc        # only manage records under this domain
            - --provider=alibabacloud
            - --policy=upsert-only                # never delete DNS records automatically
            - --alibaba-cloud-zone-type=public
            - --registry=txt                      # use TXT ownership records
            - --txt-owner-id=viralcopy-gke        # unique ID to track owned records
            - --txt-prefix=externaldns-            # prefix TXT records to avoid conflicts (e.g. externaldns-viralcopy.cc)
            - --log-level=info
            - --interval=1m                       # sync every minute
          volumeMounts:
            - name: alicloud-config
              mountPath: /etc/kubernetes
              readOnly: true
          resources:
            requests:
              cpu: 10m
              memory: 32Mi
            limits:
              cpu: 50m
              memory: 64Mi
      volumes:
        - name: alicloud-config
          secret:
            secretName: external-dns-alicloud

```