## Doc
```
```
## Python sample
```
apiVersion: autoscaling/v2    # HPA 的 API 版本
kind: HorizontalPodAutoscaler  # 资源类型：水平自动伸缩
metadata:
  name: backend-hpa            # HPA 的名字
  namespace: viralcopy         # 所属命名空间
spec:
  scaleTargetRef:              # 要伸缩的目标
    apiVersion: apps/v1
    kind: Deployment           # 目标是 Deployment
    name: backend              # 目标名称：backend
  minReplicas: 1               # 最少 1 个 Pod
  maxReplicas: 2               # 最多 2 个 Pod
  metrics:                     # 伸缩指标
    - type: Resource
      resource:
        name: cpu              # 监控 CPU
        target:
          type: Utilization
          averageUtilization: 70  # 目标：平均 CPU 使用率 70%

```