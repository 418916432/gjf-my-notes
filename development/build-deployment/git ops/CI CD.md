## Doc
### basic
```
env: git的语法

${{ }}: git action表达式，请 GitHub 在运行工作流时，把这里面的内容计算（替换）成实际的值

secrets: git存储敏感信息的地方，GitHub 仓库 → Settings → Secrets and variables → 

Actions → New repository secret

vars: 是 GitHub 提供的非敏感变量存储，GitHub 仓库 → Settings → Secrets and variables → Actions → Variables → New repository variable

run: 是github的指令，后面跟的是shell命令

services: 在运行当前job时，额外启动一个或者多个服务容器，并让你的代码去访问他们，比如测试数据库，redis，消息队列等等

steps: 定义某个job要执行的几个step

uses: 使用别人写好的workflow（git action官方的）

needs.: 是 GitHub Actions 中用来引用“上一个 Job 的输出” 的固定语法

continue-on-error: true 就算执行失败了整个workflow继续往下跑，不中断
```
### name
```
name: Backend CI/CD workflow的名字
```
### on
```
on:
  push:
    branches: [main]
    paths:
      - "backend/**"
      - ".github/workflows/backend.yml"
        
当push到main分支，并且在以下路径有改动的时候，触发这个workflow
```
### workflow_dispatch
```
workflow_dispatch:
    inputs:
      environment:
        description: "Target environment"
        required: true
        default: "production"
        type: choice
        options: [production, staging]
        
允许手动触发这个workflow

手动触发时，需要你选择一个参数（输入）。
- 参数名叫 environment
- 描述：目标环境
- 必须填写（required: true）
- 默认值是 production
- 类型是 **选择框**（choice）
- 可选的值只有两个：production 和 staging
```
### env
```
env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
  REGION: ${{ vars.GCP_REGION || 'us-central1' }}
  CLUSTER_NAME: viralcopy-cluster
  REGISTRY: ${{ vars.GCP_REGION || 'us-central1' }}-docker.pkg.dev
  IMAGE: ${{ vars.GCP_REGION || 'us-central1' }}-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/viralcopy/backend

设置github action中会用到的环境变量，供后面使用  

```
### run
```
run: |
执行多行shell命令

|: YAML 语法，表示下面可以写多行命令
```
### jobs
```
jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    
这个workflow要执行哪些任务
    
runs-on: ubuntu-latest 
在最新的 Ubuntu 系统上运行这个任务，github自动分配一台虚拟机
```
### working-directory
```
defaults:
      run:
        working-directory: backend

设置默认工作目录为，后续所有run命令都在这个文件下执行
```
### service
```
services:
  postgres:                       --服务名字
    image: postgres:16-alpine
    env:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: viralcopy_test
    ports: ["5432:5432"]
    options: >-
      --health-cmd pg_isready
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
      
GitHub Actions 会自动为你做以下事情：

1. 额外启动一个 PostgreSQL 容器
2. 使用 postgres:16-alpine 这个镜像
3. 设置数据库用户名、密码、数据库名
4. 把容器的 5432 端口映射到 Job 机器的 5432 端口
5. 等待数据库真正健康可用（通过 health check）
6. 你的测试代码可以通过 localhost:5432 去连接这个数据库
```
### uses: actions/checkout@v4
```
下载代码到git的虚拟机，每个 job 都是一台全新、独立、干净的虚拟机！它们互相之间没有任何代码、文件共享，所以必须每个 job 都 checkout，不是重复，是必须

代码默认在这个路径：/home/runner/work/你的仓库名/你的仓库名/
第一层 /home/runner/work/viralcopy：这是 GitHub Actions 为每个仓库准备的工作空间（workspace）
第二层 /home/runner/work/viralcopy/viralcopy：这是真正把你的仓库代码 checkout（克隆）下来的目录
然后自动进入这个目录 /home/runner/work/viralcopy/viralcopy
```

### actions/setup-node@v4
```
- uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: npm
          cache-dependency-path: frontend/package-lock.json

安装nodejs
安装npm
告诉cache-dependency-path package-lock.json的位置
```
### with
```
- uses: actions/setup-python@v5
  with:
    python-version: "3.12"
    cache: pip
    cache-dependency-path: backend/requirements-dev.txt
    
with: 输入参数

cache: pip 自动缓存pip下载的依赖包，换存在github action的云端缓存服务，默认保存时间7天，缓存按照 repo+branch+key来存储
计算一个 缓存 Key（通常包含以下信息）：
- Python 版本（3.12）
- 操作系统（ubuntu-latest）
- requirements-dev.txt 文件的内容哈希值（hash）
  
cache-dependency-path: 只根据 backend/requirements-dev.txt 这个文件的内容来决定是否使用缓存
```
### needs
```
build-and-push:
	name: Build & Push Docker Image
	needs: test

必须等 test job 成功
```
### outputs
```
outputs:
  image_tag: ${{ steps.meta.outputs.tags }}
  short_sha: ${{ steps.vars.outputs.short_sha }}
  
把当前job中产生的结果，传递给其他job的机制

image_tag: ${{ steps.meta.outputs.tags }}
将meta step输出的tags的值，取出来并命名为image_tag，其他job里面这样使用: ${{ needs.build-and-push.outputs.short_sha }}
```
### >> $GITHUB_OUTPUT
```
echo "short_sha=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

>>: 表示追加内容到这个文件

$GITHUB_OUTPUT: 特殊的环境变量，指向一个临时文件，这个文件就是job的output文件，执行step创建，step执行完销毁，github在step结束之前会自动把临时文件内容读出来，保存在workflow里面，比如steps.meta.outputs.tags
```
## python sample
```
name: Backend CI/CD

on:
  push:
    branches: [main]
    paths:
      - "backend/**"
      - ".github/workflows/backend.yml"
  workflow_dispatch:
    inputs:
      environment:
        description: "Target environment"
        required: true
        default: "production"
        type: choice
        options: [production, staging]

env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
  REGION: ${{ vars.GCP_REGION || 'us-central1' }}
  CLUSTER_NAME: viralcopy-cluster
  REGISTRY: ${{ vars.GCP_REGION || 'us-central1' }}-docker.pkg.dev
  IMAGE: ${{ vars.GCP_REGION || 'us-central1' }}-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/viralcopy/backend

jobs:
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: backend
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: viralcopy_test
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip

      - name: Install dependencies
        run: pip install -r requirements-dev.txt

      - name: Lint (ruff)
        run: ruff check app/

      - name: Type check (mypy)
        run: mypy app/ --ignore-missing-imports

      - name: Run tests
        env:
          DATABASE_URL: postgresql+asyncpg://test:test@localhost:5432/viralcopy_test
          SECRET_KEY: test-secret-key
          GOOGLE_CLIENT_ID: test
          OPENAI_API_KEY: sk-test
        run: pytest tests/ -v --cov=app --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: backend/coverage.xml
        continue-on-error: true

  build-and-push:
    name: Build & Push Docker Image
    needs: test
    runs-on: ubuntu-latest
    outputs:
      image_tag: ${{ steps.meta.outputs.tags }}
      short_sha: ${{ steps.vars.outputs.short_sha }}
    steps:
      - uses: actions/checkout@v4

      - id: vars
        run: echo "short_sha=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v2

      - name: Configure Docker for Artifact Registry
        run: gcloud auth configure-docker ${{ env.REGISTRY }} --quiet

      - name: Build and push
        run: |
          docker build \
            -t ${{ env.IMAGE }}:${{ steps.vars.outputs.short_sha }} \
            -t ${{ env.IMAGE }}:latest \
            backend/
          docker push ${{ env.IMAGE }}:${{ steps.vars.outputs.short_sha }}
          docker push ${{ env.IMAGE }}:latest

  deploy:
    name: Deploy to GKE
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - name: Get GKE credentials
        uses: google-github-actions/get-gke-credentials@v2
        with:
          cluster_name: ${{ env.CLUSTER_NAME }}
          location: ${{ env.REGION }}

      - name: Run database migrations
        run: |
          POD=$(kubectl -n viralcopy get pod -l app=backend -o jsonpath='{.items[0].metadata.name}')
          kubectl -n viralcopy exec "$POD" -c backend -- alembic upgrade head
        continue-on-error: true

      - name: Update backend image
        run: |
          kubectl -n viralcopy set image deployment/backend \
            backend=${{ env.IMAGE }}:${{ needs.build-and-push.outputs.short_sha }}

      - name: Wait for rollout
        run: kubectl -n viralcopy rollout status deployment/backend --timeout=5m

      - name: Verify deployment
        run: |
          kubectl -n viralcopy get pods -l app=backend
```
## frontend sample
```
# ── Frontend CI/CD Pipeline ───────────────────────────────────────────────────
name: Frontend CI/CD

on:
  push:
    branches: [main]
    paths:
      - "frontend/**"
      - ".github/workflows/frontend.yml"
  workflow_dispatch:

env:
  PROJECT_ID: ${{ secrets.GCP_PROJECT_ID }}
  REGION: ${{ vars.GCP_REGION || 'us-central1' }}
  CLUSTER_NAME: viralcopy-cluster
  REGISTRY: ${{ vars.GCP_REGION || 'us-central1' }}-docker.pkg.dev
  IMAGE: ${{ vars.GCP_REGION || 'us-central1' }}-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/viralcopy/frontend

jobs:
  lint-and-typecheck:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: frontend
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run type-check

  build-and-push:
    name: Build & Push Docker Image
    needs: lint-and-typecheck
    runs-on: ubuntu-latest
    outputs:
      short_sha: ${{ steps.vars.outputs.short_sha }}
    steps:
      - uses: actions/checkout@v4

      - id: vars
        run: echo "short_sha=$(git rev-parse --short HEAD)" >> $GITHUB_OUTPUT

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - name: Set up Cloud SDK
        uses: google-github-actions/setup-gcloud@v2

      - name: Configure Docker for Artifact Registry
        run: gcloud auth configure-docker ${{ env.REGISTRY }} --quiet

      - name: Build and push
        run: |
          docker build \
            --build-arg NEXT_PUBLIC_API_URL=${{ vars.NEXT_PUBLIC_API_URL }} \
            --build-arg NEXT_PUBLIC_GOOGLE_CLIENT_ID=${{ secrets.NEXT_PUBLIC_GOOGLE_CLIENT_ID }} \
            --build-arg NEXT_PUBLIC_PAYPAL_CLIENT_ID=${{ vars.NEXT_PUBLIC_PAYPAL_CLIENT_ID }} \
            -t ${{ env.IMAGE }}:${{ steps.vars.outputs.short_sha }} \
            -t ${{ env.IMAGE }}:latest \
            frontend/
          docker push ${{ env.IMAGE }}:${{ steps.vars.outputs.short_sha }}
          docker push ${{ env.IMAGE }}:latest

  deploy:
    name: Deploy to GKE
    needs: build-and-push
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Authenticate to Google Cloud
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}

      - name: Get GKE credentials
        uses: google-github-actions/get-gke-credentials@v2
        with:
          cluster_name: ${{ env.CLUSTER_NAME }}
          location: ${{ env.REGION }}

      - name: Update frontend image
        run: |
          kubectl -n viralcopy set image deployment/frontend \
            frontend=${{ env.IMAGE }}:${{ needs.build-and-push.outputs.short_sha }}

      - name: Wait for rollout
        run: kubectl -n viralcopy rollout status deployment/frontend --timeout=5m

      - name: Verify deployment
        run: kubectl -n viralcopy get pods -l app=frontend

```