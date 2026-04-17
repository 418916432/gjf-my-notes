## Common
```
数据验证: 用 Python 类型注解来定义数据规则，自动进行数据验证、序列化和反序列化
```
### 基础用法
```
from pydantic import BaseModel

# 定义数据模型（就像定义表格结构）
class User(BaseModel):
    name: str          # 必须是字符串
    age: int           # 必须是整数
    email: str         # 必须是字符串
    
# 使用模型 - 自动验证
user = User(name="Alice", age=25, email="alice@example.com")
print(user.name)  # Alice

# 错误数据会报错
user = User(name="Bob", age="not a number", email="bob@example.com")
# ValidationError: age 应该是 int，但得到 str
```
### 类型转化
```
比如以下字符串要被转化成List<str> 
ALLOWED_ORIGINS=https://a.com,https://b.com,https://c.com

# 在 Pydantic 自动赋值之前执行
@field_validator("ALLOWED_ORIGINS", mode="before")
    @classmethod
    def parse_cors(cls, v):
        if isinstance(v, str):
            return [origin.strip() for origin in v.split(",")]
        return v
```