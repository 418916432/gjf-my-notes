## Common
### Basic
```
FROM: Starts with a slim Python 3.12 image. Get the image from docker official.

WORKDIR: Working Directory. tells docker to work under this folder from now on. 只是进入了容器内部的目录 

RUN: Docker command, followed by shell command

COPY: Docker copy command。从宿主机复制文件到容器内部。（垮了2个系统）

USER appuser: 切换到这个用户运行

EXPOSE 8000: 容器运行时会监听8000端口

CMD: 容器启动时，执行的默认命令

ARG: 构建变量比如: ARG NEXT_PUBLIC_API_URL

ENV: Docker 指令，设置环境变量

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
### RUN pip install
```
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt
将python包安装到指定目录，并不保存缓存
```
### COPY 
```
COPY --from=builder /install /usr/local
将builder阶段中/install目录下的所有内容，复制到/usr/local

COPY . .
将宿主机上当前目录的所有内容复制到容器中的当前工作目录
```
### docker build
```
docker build \
  --build-arg NEXT_PUBLIC_API_URL=https://api.viralcopy.com \
  -t viralcopy-frontend .
  
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL

docker build的时候传入变量NEXT_PUBLIC_API_URL，这个变量再会写入名字为NEXT_PUBLIC_API_URL的环境变量

-t: 给镜像打tag，一个镜像可以有多个tag

.: DockerFile放在当前目录下


```
### docker push
```
- name: Build and push
        run: |
          docker build \
            -t ${{ env.IMAGE }}:${{ steps.vars.outputs.short_sha }} \
            -t ${{ env.IMAGE }}:latest \
            backend/
          docker push ${{ env.IMAGE }}:${{ steps.vars.outputs.short_sha }}
          docker push ${{ env.IMAGE }}:latest
          
第一次推送推送镜像和tag，第二次推送只推送tag
```
### addgroup --system
```
普通用户有这个目录/home/appuser，系统用户没有，普通用户可以登录进容器Shell（/bin/bash），系统用户不可以。前端一般用系统用户，主要是安全，攻击者获取不了shell，只有root能进入。
```
## Python sample
```
python是解释型语言，不需要编译，所以以下过程没有编译

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
## Frontend sample
```
# ── Dependencies stage ────────────────────────────────────────────────────────
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm install --legacy-peer-deps

# ── Build stage ───────────────────────────────────────────────────────────────
FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build args become env vars at build time
ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_GOOGLE_CLIENT_ID
ARG NEXT_PUBLIC_PAYPAL_CLIENT_ID
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_GOOGLE_CLIENT_ID=$NEXT_PUBLIC_GOOGLE_CLIENT_ID
ENV NEXT_PUBLIC_PAYPAL_CLIENT_ID=$NEXT_PUBLIC_PAYPAL_CLIENT_ID
ENV NEXT_TELEMETRY_DISABLED=1

RUN npm run build

# ── Production stage ──────────────────────────────────────────────────────────
FROM node:20-alpine AS production
WORKDIR /app

RUN addgroup --system --gid 1001 nodejs \
 && adduser  --system --uid 1001 nextjs

COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static   ./.next/static
COPY --from=builder --chown=nextjs:nodejs /app/public         ./public

USER nextjs
EXPOSE 3000
ENV PORT=3000
ENV NEXT_TELEMETRY_DISABLED=1
ENV NODE_ENV=production

HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 \
    CMD wget -qO- http://localhost:3000/ || exit 1

CMD ["node", "server.js"]


```