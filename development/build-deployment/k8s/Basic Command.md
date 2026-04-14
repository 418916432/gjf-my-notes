## Common
```
kubectl apply: apply yaml changes to k8s cluster

kubectl exec: 在容器中执行命令。
比如: kubectl exec -n viralcopy deploy/backend -c backend -- alembic upgrade head
-n: 指定namespace。kubectl exec -n viralcopy
deploy/backend: 指定目标：名为 backend 的 Deployment
-c backend: 指定容器名称为 backend
--: 分隔符，后面是要执行的命令

kubectl set image: 更新容器镜像
kubectl -n viralcopy set image deployment/backend \
  backend=us-central1-docker.pkg.dev/amiable-port-452501-v7/viralcopy/backend:$IMAGE_TAG
backend=...: 容器名=新镜像地址  

kubectl -l app=backend: 通过标签找
kubectl -n viralcopy get pod -l app=backend -o jsonpath='{.items[0].metadata.name}

kubectl -o output
jsonpath = 用 JSON 路径提取字段，取第0个pod的名字

kubectl exec: 进入容器内部的命令
kubectl -n viralcopy exec "$POD" -c backend -- alembic upgrade head
-c: 一个容器可能有多个容器，指定进入backend这个容器

kubectl rollout status: 查看部署状态
kubectl -n viralcopy rollout status deployment/backend --timeout=5m

```
## yaml common
### apiVersion
```
apiVersion: apps/v1
k8s api版本
```
### kind
```
kind: Deployment
管理pod的创建，比如创建几个pod
```
### metadata
```
资源的身份证，标识资源
metadata:
  name: backend           # 资源名称
  namespace: viralcopy    # 所属命名空间
  labels:                 # 标签（用于筛选）
    app: backend
```
### spec
```
资源的愿望清单，告诉Kubernetes你想要什么
spec:
  replicas: 3        # 我想要 3 个 Pod
  template: ...      # 我想要这样的 Pod
  strategy: ...      # 我想要这样更新
```
### 层级
```
yaml有点层级的概念，每个层级都有
1. 我是什么metadata
2. 我要什么spec
```
### pod
```
两个容器在同一个pod，共享网络命名空间，可以通过 `localhost` 互相访问
```
### envFrom
```
创建了操作系统提供给所有正在运行的程序的“键值对”，跟.env文件还是有点不一样的。
```
### Annotation
```
Annotation 是 Kubernetes 资源的"附加说明/配置"，用于扩展资源的功能，特别是云服务商特有的功能.
```