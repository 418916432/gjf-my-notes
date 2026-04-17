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
### Artifact Registry
```
gcloud artifacts repositories create "$REGISTRY_REPO" \
    --repository-format=docker \
    --location="$REGION" \
    --description="ViralCopy AI Docker images" \
    --quiet 2>/dev/null || log_warn "Registry already exists, skipping"

  gcloud auth configure-docker "${REGISTRY}" --quiet
  
set up artifact registry,，用来存储镜像的
```
### Cloud SQL
```
gcloud sql instances create "$DB_INSTANCE" \
      --database-version=POSTGRES_16 \
      --region="$REGION" \
      --tier=db-g1-small \
      --storage-size=20 \
      --storage-auto-increase \
      --availability-type=REGIONAL \
      --backup \
      --backup-start-time=04:00 \
      --maintenance-window-day=SUN \
      --maintenance-window-hour=6 \
      --quiet
      
create postgre sql db instance


gcloud sql databases create "$DB_NAME" \
--instance="$DB_INSTANCE" --quiet 2>/dev/null || true
create db

gcloud sql users create "$DB_USER" \
    --instance="$DB_INSTANCE" \
    --password="$DB_PASSWORD" --quiet 2>/dev/null || {
      log_warn "User may already exist — skipping user creation"
    }
create db user
```
### Cloud Storage
```
setup_gcs() {
  log_info "Creating GCS bucket: gs://$GCS_BUCKET"
  gcloud storage buckets create "gs://$GCS_BUCKET" \
    --location="$REGION" \
    --uniform-bucket-level-access \
    --quiet 2>/dev/null || log_warn "Bucket already exists, skipping"

  # Allow public read for uploaded images
  gcloud storage buckets add-iam-policy-binding "gs://$GCS_BUCKET" \
    --member=allUsers \
    --role=roles/storage.objectViewer \
    --quiet 2>/dev/null || true
  log_success "GCS bucket ready"
}

用于存储文件
```
### service account
```
setup_iam() {
  log_info "Creating GCP Service Account for workload identity..."
  SA_NAME="viralcopy-sa"
  SA_EMAIL="${SA_NAME}@${PROJECT_ID}.iam.gserviceaccount.com"

  gcloud iam service-accounts create "$SA_NAME" \
    --description="ViralCopy AI workload identity SA" \
    --display-name="ViralCopy SA" \
    --quiet 2>/dev/null || log_warn "Service account already exists"

  # Grant Cloud SQL Client
  gcloud projects add-iam-policy-binding "$PROJECT_ID" \
    --member="serviceAccount:${SA_EMAIL}" \
    --role="roles/cloudsql.client" --quiet

  # Grant GCS Object Admin
  gcloud projects add-iam-policy-binding "$PROJECT_ID" \
    --member="serviceAccount:${SA_EMAIL}" \
    --role="roles/storage.objectAdmin" --quiet

  # Grant Secret Manager reader
  gcloud projects add-iam-policy-binding "$PROJECT_ID" \
    --member="serviceAccount:${SA_EMAIL}" \
    --role="roles/secretmanager.secretAccessor" --quiet

  # Workload Identity binding
  gcloud iam service-accounts add-iam-policy-binding "${SA_EMAIL}" \
    --role="roles/iam.workloadIdentityUser" \
    --member="serviceAccount:${PROJECT_ID}.svc.id.goog[${NAMESPACE}/viralcopy-sa]" \
    --quiet 2>/dev/null || true

  # Patch the K8s service-account.yaml with actual values
  sed -i "s/PROJECT_ID/${PROJECT_ID}/g" k8s/service-account.yaml 2>/dev/null || \
  sed -i '' "s/PROJECT_ID/${PROJECT_ID}/g" k8s/service-account.yaml

  log_success "IAM configured"
}
```
### GKE Cluster
```
setup_gke() {
  log_info "Creating GKE Autopilot cluster: $CLUSTER_NAME (may take 5-10 minutes)..."

  if gcloud container clusters describe "$CLUSTER_NAME" \
      --region="$REGION" --quiet &>/dev/null; then
    log_warn "Cluster '$CLUSTER_NAME' already exists, getting credentials"
  else
    gcloud container clusters create-auto "$CLUSTER_NAME" \
      --region="$REGION" \
      --workload-pool="${PROJECT_ID}.svc.id.goog" \
      --quiet
    log_success "GKE Autopilot cluster created"
  fi

  gcloud container clusters get-credentials "$CLUSTER_NAME" \
    --region="$REGION" --quiet
  log_success "kubectl configured for $CLUSTER_NAME"
}
```
### Static IP
```
setup_static_ip() {
  log_info "Reserving global static IP: viralcopy-ip"
  gcloud compute addresses create viralcopy-ip \
    --global --quiet 2>/dev/null || log_warn "Static IP already exists"

  STATIC_IP=$(gcloud compute addresses describe viralcopy-ip \
    --global --format="value(address)")
  log_success "Static IP: $STATIC_IP"
  log_warn "Point your DNS A records to: $STATIC_IP"
  log_warn "  https://34-107-167-235.sslip.io"
  log_warn "  Domain currently: https://34-107-167-235.sslip.io"
  
}

static IP needs to be created in GCP first, then it can be used to bind to a domain
```
### gcloud builds
```
gcloud builds submit \
  --project=YOUR_GCP_PROJECT_ID \
  --config=cloudbuild.yaml \
  --substitutions=_IMAGE_TAG=$IMAGE_TAG,_NEXT_PUBLIC_API_URL=https://34-107-167-235.sslip.io
在google上构建镜像
  
steps:
  # ── Backend ──────────────────────────────────────────────────────────────────
  - name: "gcr.io/cloud-builders/docker"
    id: build-backend
    args:
      - build
      - --platform=linux/amd64
      - -t
      - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/backend:${_IMAGE_TAG}"
      - -t
      - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/backend:latest"
      - -f
      - backend/Dockerfile
      - backend

  - name: "gcr.io/cloud-builders/docker"
    id: push-backend
    waitFor: ["build-backend"]
    args:
      - push
      - --all-tags
      - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/backend"

  # ── Frontend ─────────────────────────────────────────────────────────────────
  - name: "gcr.io/cloud-builders/docker"
    id: build-frontend
    args:
      - build
      - --platform=linux/amd64
      - --build-arg
      - "NEXT_PUBLIC_API_URL=${_NEXT_PUBLIC_API_URL}"
      - --build-arg
      - "NEXT_PUBLIC_GOOGLE_CLIENT_ID=${_NEXT_PUBLIC_GOOGLE_CLIENT_ID}"
      - -t
      - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/frontend:${_IMAGE_TAG}"
      - -t
      - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/frontend:latest"
      - -f
      - frontend/Dockerfile
      - frontend

  - name: "gcr.io/cloud-builders/docker"
    id: push-frontend
    waitFor: ["build-frontend"]
    args:
      - push
      - --all-tags
      - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/frontend"

substitutions:
  _REGION: us-central1
  _IMAGE_TAG: latest
  # Set this to your backend's public URL before building.
  # Frontend API URL: https://34-107-167-235.sslip.io (all traffic via HTTPS ingress)
  # When you have a domain, change this to https://api.yourdomain.com
  _NEXT_PUBLIC_API_URL: https://34-107-167-235.sslip.io
  _NEXT_PUBLIC_GOOGLE_CLIENT_ID: ""

options:
  machineType: E2_HIGHCPU_8
  logging: CLOUD_LOGGING_ONLY

images:
  - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/backend:${_IMAGE_TAG}"
  - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/backend:latest"
  - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/frontend:${_IMAGE_TAG}"
  - "${_REGION}-docker.pkg.dev/${PROJECT_ID}/viralcopy/frontend:latest"

cloudbuild.yaml
它和 GitHub Actions 的 workflow 功能类似，但这是 Google Cloud 官方的 CI/CD 工具, 在 Google Cloud 上自动构建并推送两个 Docker 镜像.  
```