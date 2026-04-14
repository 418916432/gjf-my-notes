## Common
### gccloud build
```
gcloud builds submit \
--project=YOUR_GCP_PROJECT_ID \
--config=cloudbuild.yaml \
--substitutions=_IMAGE_TAG=$IMAGE_TAG,_NEXT_PUBLIC_API_URL=https://34-107-167-235.sslip.io


这是一个 Google Cloud Build 命令，用于在 GCP 上构建 Docker 镜像。

YOUR_GCP_PROJECT_ID: 需要你手动替换的标记。在文档和教程中，用全大写加下划线表示"这里需要你填自己的值"。

--substitutions=: 后面跟的是键值对
```
### gcloud container clusters get-credentials
```
gcloud container clusters get-credentials viralcopy-cluster --region=us-central1
获取 GKE 集群的访问凭证，配置到本地 `kubectl`，让你可以用 `kubectl` 命令操作这个集群
```
### enable service
```
setup_gcp_project() {
  log_info "Configuring project: $PROJECT_ID / $REGION"
  gcloud config set project "$PROJECT_ID"
  gcloud config set compute/region "$REGION"
  gcloud config set compute/zone "$ZONE"

  log_info "Enabling required GCP APIs..."
  gcloud services enable \
    container.googleapis.com \
    sqladmin.googleapis.com \
    storage.googleapis.com \
    artifactregistry.googleapis.com \
    cloudresourcemanager.googleapis.com \
    iam.googleapis.com \
    servicenetworking.googleapis.com \
    secretmanager.googleapis.com \
    --quiet
  log_success "APIs enabled"
}

GCP 所有服务默认都是关闭的，你必须手动开启才能用，比如不enable就不能执行gcloud artifacts
```