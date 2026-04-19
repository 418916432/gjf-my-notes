## Doc
### usage
```
将pod暴露为集群内部可以访问的网络服务，解决了2个问题
1. pod ip是动态变化的，比如启动新的pod
2. 需要负载均衡，多个 Pod 需要分发流量
```
### selector
```
selector:
  app: backend
  
将流量转发给pod名字为backend的pod
```
### port vs targetPort
```
ports:
  - port: 8000        # Service 的端口（别人访问这个）
    targetPort: 8000  # Pod 的端口（转发到这里）

客户端 ──► Service:8000 ──► Pod:8000
          ↑                ↑
     Service 端口      容器实际监听的端口
```
### type: ClusterIP  的含义
| Service 类型         | 访问范围            | 用途       |
| ------------------ | --------------- | -------- |
| **ClusterIP** (默认) | 只在集群内部          | 内部服务互相调用 |
| NodePort           | 集群内部 + 节点 IP:端口 | 测试、直接暴露  |
| LoadBalancer       | 公网（云厂商 LB）      | 对外提供服务   |
| ExternalName       | 外部域名            | 访问集群外服务  |
## python sample
### Service
```
apiVersion: v1              # Service 的 API 版本
kind: Service               # 资源类型
metadata:
  name: backend             # Service 名称（其他 Pod 用这个访问）
  namespace: viralcopy      # 命名空间
  annotations:
    # Use BackendConfig to increase timeout to 300s (covers DeepSeek Reasoner ~60-120s)
    cloud.google.com/backend-config: '{"default": "backend-timeout"}'
spec:
  selector:                 # 选择器：找哪些 Pod
    app: backend            # 找 labels 包含 app=backend 的 Pod
  ports:
    - protocol: TCP         # 协议
      port: 8000            # Service 监听的端口
      targetPort: 8000      # 转发到 Pod 的端口
  type: ClusterIP           # Service 类型（集群内部访问）
```
### BackendConfig
```
# GKE BackendConfig — increases backend timeout for long-running AI requests
# DeepSeek Reasoner (R1) takes 60-120s; default GKE timeout is 30s → causes 502.
# This BackendConfig is referenced by the backend Service annotation.
apiVersion: cloud.google.com/v1
kind: BackendConfig
metadata:
  name: backend-timeout
  namespace: viralcopy
spec:
  timeoutSec: 300   # 5 minutes — covers even slow Reasoner responses

告诉GKE的 Ingress / Load Balancer，把默认超时时间从30秒，延长到5分钟。这个配置是在Ingress里面读取了service的配置。否则默认30秒502
```