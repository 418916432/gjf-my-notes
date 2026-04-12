## Common
### brew services 作为服务启动，关闭终端不影响
```
brew services list  # 查看所有服务状态
brew services start postgresql@16
brew services stop postgresql@16
brew services restart postgresql@16

对比手动：
pg_ctl -D /usr/local/var/postgres start
# 手动停止
pg_ctl -D /usr/local/var/postgres stop
# 查看状态
pg_ctl -D /usr/local/var/postgres status
```