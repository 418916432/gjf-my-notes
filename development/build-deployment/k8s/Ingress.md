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
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: viralcopy-ingress
  namespace: viralcopy
  annotations:
    kubernetes.io/ingress.class: "gce"
    kubernetes.io/ingress.global-static-ip-name: "viralcopy-ip"
    networking.gke.io/managed-certificates: "viralcopy-ssl"
    kubernetes.io/ingress.allow-http: "true"
spec:
  rules:
    - host: 34-107-167-235.sslip.io
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
  name: viralcopy-ssl
  namespace: viralcopy
spec:
  domains:
    - 34-107-167-235.sslip.io
```