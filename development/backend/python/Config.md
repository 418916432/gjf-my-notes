## Doc
### BaseSetting
```
专门用来读 .env/环境变量的配置工具类

不用BaseSettings的写法:

import os
from dotenv import load_dotenv

load_dotenv()

APP_NAME = os.getenv("APP_NAME", "default")
DEBUG = os.getenv("DEBUG", "False").lower() == "true"
ALLOWED_ORIGINS = os.getenv("ALLOWED_ORIGINS", "").split(",")

使用BaseSettings的写法:

from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    APP_NAME: str = "default"
    DEBUG: bool = False
    ALLOWED_ORIGINS: list[str] = []
```
### SettingsConfigDict
```
model_config = SettingsConfigDict(env_file=".env", case_sensitive=True, extra="ignore")

告诉BaseSettings怎么去读配置文件，真正创建这个对象的时候，BaseSettings会使用这个对象model_config去读配置
```