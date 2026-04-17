## Doc
```
python最主流的数据库工具，不用写sql，直接用python对象操作数据库

不用写这个
SELECT * FROM users WHERE id = 1

这样就行了
await db.get(User, id=1)
```
### AsyncSessionLocal
```
AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    expire_on_commit=False,
    class_=AsyncSession,
)

生成数据库会话的
```
### DeclarativeBase
```
class Base(DeclarativeBase):
    """Shared declarative base for all ORM models."""
    
项目里面所有数据库表模型的父类
所有表必须继承这个Base，SQLAIchemy才能认出他们是数据库表
```
### getdb
```
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """FastAPI dependency that provides a database session per request."""
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
            
获取一个数据库连接，给依赖者去用。

一般这样用
async def xxx(db: AsyncSession = Depends(get_db)):
```