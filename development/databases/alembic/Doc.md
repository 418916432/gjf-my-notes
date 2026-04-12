## Common
```
alemic是一个数据迁移工具，专门为 SQLAlchemy（Python 最流行的 ORM）设计和开发的

alembic upgrade head: apply latest migration
```
## Manually Flow
### init
```
alembic init alembic

# 生成的文件：
# - alembic.ini (配置文件)
# - alembic/ (目录)
#   ├── env.py
#   ├── script.py.mako
#   └── versions/
```
### config env.py
```
# alembic/env.py 关键配置

from app.core.database import Base
from app.core.config import settings
import app.models  # 导入所有模型

# 告诉 Alembic 你的模型定义
target_metadata = Base.metadata

# 设置数据库连接
config.set_main_option('sqlalchemy.url', settings.DATABASE_URL)
```
### generate
```
# 1. 自动生成迁移文件（对比 models.py 和空数据库）
alembic revision --autogenerate -m "create users table"

# 3. 应用到数据库
alembic upgrade head
```
## Auto flow
