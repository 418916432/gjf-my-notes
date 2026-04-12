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
```