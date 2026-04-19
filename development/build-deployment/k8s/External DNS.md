## Doc
### flow
```
1. ExternalDNS 监控你的 Ingress
   ↓ (每隔1分钟检查一次)
2. 发现 Ingress 有注解: external-dns.alpha.kubernetes.io/hostname: "viralcopy.cc"
   ↓
3. 获取 Ingress 的 IP 地址: 34.107.167.235
   ↓
4. 调用阿里云 DNS API
   ↓
5. 自动创建/更新 A 记录: viralcopy.cc → 34.107.167.235
```
### RBAC 权限
```
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

让 ExternalDNS 有权限查看集群里的 Ingress、Service 等资源
```
### 阿里云认证
```
volumes:
  - name: alicloud-config
    secret:
      secretName: external-dns-alicloud  # 存储阿里云 AccessKey/SecretKey

ExternalDNS 用这里的secret来调用阿里云 DNS API
```
## python sample
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