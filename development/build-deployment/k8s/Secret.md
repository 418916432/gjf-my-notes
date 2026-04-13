## Doc
```
存储敏感信息，通过环境变量注入到容器
```
## python sample
```
apiVersion: v1              # Secret 的 API 版本
kind: Secret                # 资源类型
metadata:
  name: viralcopy-secrets   # Secret 名称
  namespace: viralcopy      # 命名空间
type: Opaque                # 类型：普通密钥（不是 TLS 或 Docker 凭证）
data:                       # 编码后的数据
  SECRET_KEY: <base64...>   # Django/FastAPI 的密钥
  DATABASE_URL: <base64...> # 数据库连接串
  ...
```