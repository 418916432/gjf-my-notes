## Common
```
uvicorn app.main:app --reload: 启动FastAPI服务

uvicorn - ASGI 服务器（运行异步 Python web 应用）
    
app.main:app - 应用的位置（模块:对象）
    
--reload - 自动重载模式（开发时使用）
```
### uvicorn app.main:app --reload
```
启动 unicorn，它是一个web服务器
导入 app.main，python从上到下执行app/main.py文件。
执行app = create_app()
unicore 拿到这个app对象，开启监听端口
```
### decorator
```
def my_decorator(func):        # 装饰器名字叫 my_decorator
    def wrapper():
        print("装饰器在函数执行前做了点事")
        func()                     # 调用原来的函数
        print("装饰器在函数执行后又做了点事")
    return wrapper


@my_decorator                  # 这里用的是装饰器名字
def say_hello():               # 被装饰的函数名字叫 say_hello
    print("Hello World!")

say_hello()
```
### @asynccontextmanager
```
@asynccontextmanager
async def lifespan(app: FastAPI):
    logger.info("Starting %s (debug=%s)", settings.APP_NAME, settings.DEBUG)
    yield
    logger.info("Shutting down %s", settings.APP_NAME)
    
它是一个装饰器。app执行之前打印Starting，app执行完之后打印关闭，在这里它只定义了装饰器，并没有执行。真正的绑定代码要执行 lifespan(app)之后才能绑定成功
```
### with
```
with open("test.txt", "w", encoding="utf-8") as file:
    file.write("Hello World")
    # 这里不需要写 close()
    
自动打开文件，自动关闭文件
```
### Depends
```
async def get_current_user(
    db: Annotated[AsyncSession, Depends(get_db)],
    token: Annotated[str | None, Depends(_resolve_token)],
) -> User

FastAPI的依赖注入系统会把这些依赖注入到这些方法
```
### APIRouter
```
router = APIRouter(prefix="/users", tags=["Users"])

创建一个路由集合，相当于spring的controller

@router.get("/me", response_model=UserProfile)
async def get_profile(current_user: CurrentUser) -> UserProfile:
    return UserProfile(
        id=current_user.id,
        email=current_user.email,
        nickname=current_user.nickname,
        avatar_url=current_user.avatar_url,
        is_admin=current_user.is_admin,
        is_subscribed=current_user.is_subscribed,
        is_free_user=current_user.is_free_user,
        trial_remaining=current_user.trial_remaining,
        locale=current_user.locale,
        created_at=current_user.created_at,
    )
    
CurrentUser = Annotated[User, Depends(get_current_user)]

当fastAPI看到CurrentUser这个类型后，解析Annotated，发现depends里面的依赖，发现会自动调用依赖的这个方法来获取依赖对象。

所以current_user并不是传递进来的，反而是调用depends里面的方法获取到的

Annotated 是给源类型 User 添加额外信息，这里的额外信息就是：该怎么获取 current user
```
### Header Cookie
```
async def _resolve_token(
    authorization: Annotated[str | None, Header()] = None,
    access_token: Annotated[str | None, Cookie()] = None,
) -> str | None:
    """Extract Bearer token from Authorization header or httpOnly cookie."""
    if authorization and authorization.startswith("Bearer "):
        return authorization[7:]
    return access_token

从Header里面拿拿token，拿不到就从cookie拿
```
## sample
```
"""FastAPI application factory."""

from contextlib import asynccontextmanager

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.trustedhost import TrustedHostMiddleware
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.errors import RateLimitExceeded
from slowapi.middleware import SlowAPIMiddleware
from slowapi.util import get_remote_address

from app.core.config import settings
from app.core.logging import configure_logging, get_logger
from app.routers import admin, auth, generate, history, payment, user

configure_logging()
logger = get_logger(__name__)


@asynccontextmanager
async def lifespan(app: FastAPI):
    logger.info("Starting %s (debug=%s)", settings.APP_NAME, settings.DEBUG)
    yield
    logger.info("Shutting down %s", settings.APP_NAME)


def create_app() -> FastAPI:
    app = FastAPI(
        title=settings.APP_NAME,
        description="AI-powered social media copy generator API",
        version="1.0.0",
        docs_url="/docs" if settings.DEBUG else None,
        redoc_url="/redoc" if settings.DEBUG else None,
        lifespan=lifespan,
    )

    # ── Rate limiting ─────────────────────────────────────────────────────────
    limiter = Limiter(key_func=get_remote_address, default_limits=[f"{settings.RATE_LIMIT_PER_MINUTE}/minute"])
    app.state.limiter = limiter
    app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)
    app.add_middleware(SlowAPIMiddleware)

    # ── CORS ──────────────────────────────────────────────────────────────────
    app.add_middleware(
        CORSMiddleware,
        allow_origins=settings.ALLOWED_ORIGINS,
        allow_credentials=True,
        allow_methods=["*"],
        allow_headers=["*"],
    )

    # ── Routers ───────────────────────────────────────────────────────────────
    prefix = settings.API_PREFIX
    app.include_router(auth.router, prefix=prefix)
    app.include_router(generate.router, prefix=prefix)
    app.include_router(history.router, prefix=prefix)
    app.include_router(user.router, prefix=prefix)
    app.include_router(payment.router, prefix=prefix)
    app.include_router(admin.router, prefix=prefix)

    # ── Health check ──────────────────────────────────────────────────────────
    @app.get("/health", tags=["Health"])
    async def health_check() -> dict:
        return {"status": "ok", "service": settings.APP_NAME}

    return app


app = create_app()

```