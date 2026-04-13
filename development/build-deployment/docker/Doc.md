## Common
### Basic
```
FROM: Starts with a slim Python 3.12 image. Get the image from docker official.

WORKDIR: Working Directory. tells docker to work under this folder from now on. 只是进入了容器内部的目录 

RUN: Docker command, followed by shell command

COPY: Docker copy command。从宿主机复制文件到容器内部。（垮了2个系统）

```
### FROM python:3.12-slim AS builder
```
AS:
FROM python:3.12-slim AS builder
FROM python:3.12-slim AS production

给这个构建阶段起一个名字
```
### RUN apt-get update && apt-get install
```
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential libpq-dev curl \
    && rm -rf /var/lib/apt/lists/*
    
- RUN → Docker 指令，告诉 Docker：“请执行下面的命令”
- apt-get update → 更新软件包列表（必须先做）
- && → “并且”，前一个命令成功才执行下一个
- apt-get install -y ... → 使用 apt-get 安装软件
    - -y：自动确认安装（不用手动输入 yes）
    - --no-install-recommends：只安装必要的，不装推荐的额外包（让镜像更小）
- build-essential libpq-dev curl → 要安装的三个包
- rm -rf /var/lib/apt/lists/* → 安装完后删除缓存，减小镜像体积
```
## Python sample
```
FROM python:3.12-slim AS builder

WORKDIR /build

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential libpq-dev curl \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ── Production stage ──────────────────────────────────────────────────────────
FROM python:3.12-slim AS production

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq-dev curl \
    && rm -rf /var/lib/apt/lists/* \
    && useradd -m -u 1000 appuser

COPY --from=builder /install /usr/local
COPY . .

RUN chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```