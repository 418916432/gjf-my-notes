## Doc
```
配置非敏感信息，以环境变量注入到容器
```
## Python sample
```
apiVersion: v1              # ConfigMap 的 API 版本
kind: ConfigMap             # 资源类型
metadata:
  name: viralcopy-config    # ConfigMap 名称
  namespace: viralcopy      # 命名空间
data:                       # 配置数据（键值对）
  APP_NAME: "ViralCopy AI"  # 应用名称
  DEBUG: "false"            # 调试模式（生产环境关闭）
  API_PREFIX: "/api/v1"     # API 路由前缀
  ...
```