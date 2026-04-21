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
### BaseModel
```
BaseModel is a class from the Pydantic library. It can
Validates the data (checks types)
Converts types if possible (e.g. string "25" → int 25)
Gives clear error messages if data is wrong
Provides nice attributes (user.username, user.email, etc.)
Can easily convert to dictionary or JSON

without BaseModel
def register_user(data: dict):
    username = data.get("username")
    email = data.get("email")
    age = data.get("age")
    
    # Manual validation - very painful
    if not username or len(username) < 3:
        raise Error("Username too short")
    if not email or "@" not in email:
        raise Error("Invalid email")
    if not isinstance(age, int) or age < 0:
        raise Error("Age must be positive number")
    
    # ... more manual checks
    
with BaseModel:
from pydantic import BaseModel
class UserRegister(BaseModel):
    username: str
    email: str
    age: int
    is_active: bool = True          # default value

```
### Field
```
Field is a helper function from Pydantic. Its job is to add extra information and rules to a field inside a BaseModel class.

input_text: str | None = Field(None, max_length=2000)
By default, its value is None. But if someone gives a string, it cannot be longer than 2000 characters."
```
### Annotated
```
Annotated means: This is the type, and here is some extra metadata about it.

platforms: Annotated[list[str], Field(min_length=1, max_length=9)]

without Annotated, the default value and field rules are mixed together, the readability is bad:
class GenerateRequest(BaseModel):
    
    platforms: list[str] = Field(
        ["instagram", "twitter"],     # ← Default value
        min_length=1,                 # ← Validation rule
        max_length=9,                 # ← Validation rule
        description="Social media platforms"
    )
    
with Annotated, the types are seperated from validation rules.
class GenerateRequest(BaseModel):
    
    platforms: Annotated[
        list[str],                                   # ← Only the type
        Field(                                       # ← Only the rules + default
            default=["instagram", "twitter"],        # Default is now inside Field
            min_length=1,
            max_length=9,
            description="Social media platforms"
        )
    ]
    
```