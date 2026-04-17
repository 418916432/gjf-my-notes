## Common
## venv
```
python3.12 -m venv .venv: 创建虚拟环境，隔离环境 
# 项目 A
cd project_a
python3.12 -m venv .venv
source .venv/bin/activate
pip install django==3.2   # 只安装在 project_a/.venv/

# 项目 B
cd project_b
python3.12 -m venv .venv
source .venv/bin/activate
pip install django==4.2   # 只安装在 project_b/.venv/
# 互不干扰！

source .venv/bin/activate: 激活虚拟环境
(.venv) $ deactivate: 推出虚拟环境


```
### install
```
pip install -r requirements-dev.txt 'pydantic[email]'
```
### pytest
```
python的测试框架

# 详细模式（最常用）
pytest tests/ -v
```
### .env
```
python环境变量配置文件
```
### mypy
```
检查代码中的类型是否正确

比如：
def add(a: int, b: int) -> int: # 加了类型注解 return a + b

add("hello", 123) # ← mypy 会在这里报错，因为类型不匹配
```
### """
```
"""FastAPI application factory."""
This is a comment
```
### 执行顺序
```
python默认，从上到下，从左到右一行一行执行
```
### 创建对象
```
svc = UserService(db)
```
### object method
```
def 普通方法(self):
    pass
self->具体对象本身

def configure_logging() -> None:
    """Set up root logger with JSON-friendly formatting for GCP Cloud Logging."""
    
def
configure_logging()
None

对象方法需要一个对象去调用   
```
### class method
```
class User:
    @classmethod
    def hello(cls):
        print("hi")

# 调用：类直接调用
User.hello() 
```
### lru_cache
```
@lru_cache
def get_settings() -> Settings:
    return Settings()
    
给方法加缓存
```
### yield
```
def f():
    print("第一步")
    yield 123
    print("第三步")
    
暂停把值扔出去，等外面用完了，在回来继续跑
```
### next()
```
让暂停的生成器继续跑，知道下一个yield
```
### Annotated
```
给一个类型添加附加信息，但不改变原本类型

from typing import Annotated

# 给 int 附加一个 "年龄" 说明
Age = Annotated[int, "年龄必须大于0"]


CurrentUser = Annotated[User, Depends(get_current_user)]
把原本的User 和附加的依赖规则打包在一起，生成一个新的类型CurrentUser
```
### httpx
```
async with httpx.AsyncClient() as client:

它是一个异步http客户端，这里异步对象的意思是，发起了请求，不阻塞，不然这个客户端啥事都干不了。
```