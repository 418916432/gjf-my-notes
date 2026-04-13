## Doc
```
这是一个 Kubernetes ServiceAccount，为 Pod 提供一个身份，用于与 Kubernetes API 和 GCP 服务进行认证和授权

ServiceAccount 解决了两个问题：
1. Pod 如何证明自己的身份？（而不是用人的身份）
2. Pod 需要什么权限？（访问 K8s API、GCP 资源），比如这里为了通过cloud-sql-proxy访问Cloud SQL
```
## python sample
```
apiVersion: v1                      # API 版本
kind: ServiceAccount                # 资源类型
metadata:
  name: viralcopy-sa                # ServiceAccount 名称
  namespace: viralcopy              # 命名空间
  annotations:                      # 附加配置
    iam.gke.io/gcp-service-account: viralcopy-sa@PROJECT_ID.iam.gserviceaccount.com # PROJECT_ID是需要替换的占位符
```