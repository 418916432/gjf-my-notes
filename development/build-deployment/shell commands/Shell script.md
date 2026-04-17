## Doc
### set -euo pipefail
```
让shell脚本编程严格模式，报错就停止

-e: 命令一报错，整个脚本立刻停止
-u: 使用未定义的变量，直接报错
-o: option
-o pipefail: 管道里任何一步错，整个都算失败
```
### log fuction
```
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log_info()    { echo -e "${BLUE}[INFO]${NC}  $*"; }
log_success() { echo -e "${GREEN}[OK]${NC}    $*"; }
log_warn()    { echo -e "${YELLOW}[WARN]${NC}  $*"; }
log_error()   { echo -e "${RED}[ERROR]${NC} $*"; exit 1; }

$* 传递进来的所有参数
```
### Argument parsing
```
while [[ $# -gt 0 ]]; do
  case $1 in
    --project)   PROJECT_ID="$2";      shift 2 ;;
    --region)    REGION="$2";          shift 2 ;;
    --tag)       IMAGE_TAG="$2";       shift 2 ;;
    --skip-infra) SKIP_INFRA=true;     shift   ;;
    --skip-build) SKIP_BUILD=true;     shift   ;;
    --help)
      echo "Usage: $0 --project PROJECT_ID [--region REGION] [--tag TAG] [--skip-infra] [--skip-build]"
      exit 0 ;;
    *) log_error "Unknown argument: $1" ;;
  esac
done

$#: 传入的参数个数

case $1 in
    --project)   PROJECT_ID="$2";      shift 2 ;;
$1: 当前第一个参数
参数是 `--project`，就把下一个参数 $2 存到 `PROJECT_ID`
shift 2: 扔掉前 2 个参数

--skip-infra) SKIP_INFRA=true;     shift   ;;
当参数是skip-infra，设置true，丢掉当前变量

```
### main
```
main shell 脚本的入口

main() {
  check_prerequisites

  if [[ "$SKIP_INFRA" == "false" ]]; then
    setup_gcp_project
    setup_artifact_registry
    setup_cloud_sql
    setup_gcs
    setup_iam
    setup_gke
    setup_static_ip
  fi

  if [[ "$SKIP_BUILD" == "false" ]]; then
    build_and_push
  fi

  apply_k8s
  wait_for_rollout
  run_migrations
  print_summary
}

```
### $
```
main "$@"
调用main函数，并把参数原封不动的传给它

./deploy.sh arg1 arg2 arg3
$# = 3
$@ = "arg1" "arg2" "arg3", 所有参数，分开
$* = "arg1 arg2 arg3", 所有参数，合并成一个

"$@": 
用 "$@"（正确）→ 保持原样："hello world" "foo bar"

$@:
用 $@（错误）→ 被拆成："hello" "world" "foo" "bar"
```
### 函数与命令
```
命令从上到下自动执行，函数需要调用
```
### 执行shell脚本
```
# 第一步：给脚本添加执行权限（只需做一次） chmod +x aaa.sh

# 第二步：执行脚本 ./aaa.sh
```
## Sample
```
#!/usr/bin/env bash
# =============================================================================
#  ViralCopy AI — Full Google Cloud Deployment Script
#  Usage: ./scripts/deploy.sh [--project PROJECT_ID] [--region REGION] [--env ENV]
#
#  This script:
#   1. Validates required tools (gcloud, kubectl, docker)
#   2. Sets up GCP project (APIs, IAM, Artifact Registry, Cloud SQL, GCS)
#   3. Creates a GKE Autopilot cluster
#   4. Builds and pushes Docker images
#   5. Applies all Kubernetes manifests
#   6. Runs database migrations
#   7. Prints the live URLs
#
#  Prerequisites:
#   - gcloud CLI authenticated (gcloud auth login)
#   - Docker running
#   - kubectl installed
#   - A domain pointed at the static IP created by this script
# =============================================================================

set -euo pipefail

# ── Color helpers ──────────────────────────────────────────────────────────────
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log_info()    { echo -e "${BLUE}[INFO]${NC}  $*"; }
log_success() { echo -e "${GREEN}[OK]${NC}    $*"; }
log_warn()    { echo -e "${YELLOW}[WARN]${NC}  $*"; }
log_error()   { echo -e "${RED}[ERROR]${NC} $*"; exit 1; }

# ── Defaults ───────────────────────────────────────────────────────────────────
PROJECT_ID=""
REGION="us-central1"
ZONE="us-central1-a"
CLUSTER_NAME="viralcopy-cluster"
NAMESPACE="viralcopy"
DB_INSTANCE="viralcopy-db"
DB_NAME="viralcopy"
DB_USER="viralcopy_app"
GCS_BUCKET="viralcopy-uploads-${PROJECT_ID}"
REGISTRY_REPO="viralcopy"
IMAGE_TAG=$(git rev-parse --short HEAD 2>/dev/null || echo "latest")
SKIP_INFRA=false
SKIP_BUILD=false

# ── Argument parsing ───────────────────────────────────────────────────────────
while [[ $# -gt 0 ]]; do
  case $1 in
    --project)   PROJECT_ID="$2";      shift 2 ;;
    --region)    REGION="$2";          shift 2 ;;
    --tag)       IMAGE_TAG="$2";       shift 2 ;;
    --skip-infra) SKIP_INFRA=true;     shift   ;;
    --skip-build) SKIP_BUILD=true;     shift   ;;
    --help)
      echo "Usage: $0 --project PROJECT_ID [--region REGION] [--tag TAG] [--skip-infra] [--skip-build]"
      exit 0 ;;
    *) log_error "Unknown argument: $1" ;;
  esac
done

[[ -z "$PROJECT_ID" ]] && log_error "PROJECT_ID is required. Use --project YOUR_PROJECT_ID"

REGISTRY="${REGION}-docker.pkg.dev"
BACKEND_IMAGE="${REGISTRY}/${PROJECT_ID}/${REGISTRY_REPO}/backend"
FRONTEND_IMAGE="${REGISTRY}/${PROJECT_ID}/${REGISTRY_REPO}/frontend"
DB_CONNECTION_NAME="${PROJECT_ID}:${REGION}:${DB_INSTANCE}"
GCS_BUCKET="${GCS_BUCKET:-viralcopy-uploads-${PROJECT_ID}}"

# ── Prerequisite checks ────────────────────────────────────────────────────────
check_prerequisites() {
  log_info "Checking prerequisites..."
  for cmd in gcloud kubectl docker; do
    command -v "$cmd" &>/dev/null || log_error "'$cmd' not found. Please install it."
  done
  gcloud auth list --filter=status:ACTIVE --format="value(account)" | grep -q "." \
    || log_error "Not authenticated. Run: gcloud auth login"
  log_success "All prerequisites satisfied"
}

# ── GCP project setup ──────────────────────────────────────────────────────────
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

# ── Artifact Registry ──────────────────────────────────────────────────────────
setup_artifact_registry() {
  log_info "Creating Artifact Registry repository: $REGISTRY_REPO"
  gcloud artifacts repositories create "$REGISTRY_REPO" \
    --repository-format=docker \
    --location="$REGION" \
    --description="ViralCopy AI Docker images" \
    --quiet 2>/dev/null || log_warn "Registry already exists, skipping"

  gcloud auth configure-docker "${REGISTRY}" --quiet
  log_success "Artifact Registry ready"
}

# ── Cloud SQL (PostgreSQL) ─────────────────────────────────────────────────────
setup_cloud_sql() {
  log_info "Creating Cloud SQL instance: $DB_INSTANCE (this may take 5-10 minutes)..."

  if gcloud sql instances describe "$DB_INSTANCE" --quiet &>/dev/null; then
    log_warn "Cloud SQL instance '$DB_INSTANCE' already exists, skipping creation"
  else
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
    log_success "Cloud SQL instance created"
  fi

  log_info "Creating database and user..."
  gcloud sql databases create "$DB_NAME" \
    --instance="$DB_INSTANCE" --quiet 2>/dev/null || true

  DB_PASSWORD=$(openssl rand -base64 32 | tr -d /=+ | head -c 32)
  gcloud sql users create "$DB_USER" \
    --instance="$DB_INSTANCE" \
    --password="$DB_PASSWORD" --quiet 2>/dev/null || {
      log_warn "User may already exist — skipping user creation"
    }

  # Store password in Secret Manager
  echo -n "$DB_PASSWORD" | gcloud secrets create viralcopy-db-password \
    --data-file=- --quiet 2>/dev/null || \
  echo -n "$DB_PASSWORD" | gcloud secrets versions add viralcopy-db-password \
    --data-file=- --quiet

  log_success "Cloud SQL configured. DB password stored in Secret Manager as 'viralcopy-db-password'"
}

# ── Cloud Storage ──────────────────────────────────────────────────────────────
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

# ── Service Account + Workload Identity ───────────────────────────────────────
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

# ── GKE Cluster ────────────────────────────────────────────────────────────────
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

# ── Static IP ─────────────────────────────────────────────────────────────────
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

# ── Docker build & push ────────────────────────────────────────────────────────
build_and_push() {
  log_info "Building and pushing Docker images (tag: $IMAGE_TAG)..."

  # Backend
  docker build -t "${BACKEND_IMAGE}:${IMAGE_TAG}" -t "${BACKEND_IMAGE}:latest" backend/
  docker push "${BACKEND_IMAGE}:${IMAGE_TAG}"
  docker push "${BACKEND_IMAGE}:latest"
  log_success "Backend image pushed: ${BACKEND_IMAGE}:${IMAGE_TAG}"

  # Frontend — read env vars from env file if present
  NEXT_PUBLIC_API_URL="${NEXT_PUBLIC_API_URL:-https://34-107-167-235.sslip.io}"
  NEXT_PUBLIC_GOOGLE_CLIENT_ID="${NEXT_PUBLIC_GOOGLE_CLIENT_ID:-}"
  NEXT_PUBLIC_PAYPAL_CLIENT_ID="${NEXT_PUBLIC_PAYPAL_CLIENT_ID:-}"

  docker build \
    --build-arg "NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}" \
    --build-arg "NEXT_PUBLIC_GOOGLE_CLIENT_ID=${NEXT_PUBLIC_GOOGLE_CLIENT_ID}" \
    --build-arg "NEXT_PUBLIC_PAYPAL_CLIENT_ID=${NEXT_PUBLIC_PAYPAL_CLIENT_ID}" \
    -t "${FRONTEND_IMAGE}:${IMAGE_TAG}" \
    -t "${FRONTEND_IMAGE}:latest" \
    frontend/
  docker push "${FRONTEND_IMAGE}:${IMAGE_TAG}"
  docker push "${FRONTEND_IMAGE}:latest"
  log_success "Frontend image pushed: ${FRONTEND_IMAGE}:${IMAGE_TAG}"
}

# ── Apply K8s manifests ────────────────────────────────────────────────────────
apply_k8s() {
  log_info "Applying Kubernetes manifests..."

  # Substitute placeholders
  find k8s/ -name "*.yaml" | while read -r f; do
    sed -i "s|PROJECT_ID|${PROJECT_ID}|g;s|REGION|${REGION}|g;s|IMAGE_TAG|${IMAGE_TAG}|g" "$f" \
      2>/dev/null || \
    sed -i '' "s|PROJECT_ID|${PROJECT_ID}|g;s|REGION|${REGION}|g;s|IMAGE_TAG|${IMAGE_TAG}|g" "$f"
  done

  # Check secrets file
  if [[ ! -f k8s/secrets.yaml ]]; then
    log_error "k8s/secrets.yaml not found! Copy k8s/secrets.template.yaml to k8s/secrets.yaml and fill in values."
  fi

  kubectl apply -f k8s/namespace.yaml
  kubectl apply -f k8s/service-account.yaml
  kubectl apply -f k8s/configmap.yaml
  kubectl apply -f k8s/secrets.yaml
  kubectl apply -f k8s/backend/deployment.yaml
  kubectl apply -f k8s/backend/service.yaml
  kubectl apply -f k8s/backend/hpa.yaml
  kubectl apply -f k8s/frontend/deployment.yaml
  kubectl apply -f k8s/frontend/service.yaml
  kubectl apply -f k8s/ingress.yaml

  log_success "All K8s manifests applied"
}

# ── Wait for rollout ───────────────────────────────────────────────────────────
wait_for_rollout() {
  log_info "Waiting for deployments to roll out..."
  kubectl -n "$NAMESPACE" rollout status deployment/backend  --timeout=5m
  kubectl -n "$NAMESPACE" rollout status deployment/frontend --timeout=5m
  log_success "Rollouts complete"
}

# ── Run DB migrations ──────────────────────────────────────────────────────────
run_migrations() {
  log_info "Running database migrations..."
  BACKEND_POD=$(kubectl -n "$NAMESPACE" get pod \
    -l app=backend \
    -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)

  if [[ -n "$BACKEND_POD" ]]; then
    kubectl -n "$NAMESPACE" exec "$BACKEND_POD" -- alembic upgrade head
    log_success "Migrations complete"
  else
    log_warn "No backend pod found — skipping migrations (run manually once pods are ready)"
  fi
}

# ── Print summary ─────────────────────────────────────────────────────────────
print_summary() {
  STATIC_IP=$(gcloud compute addresses describe viralcopy-ip \
    --global --format="value(address)" 2>/dev/null || echo "unknown")

  echo ""
  echo -e "${GREEN}══════════════════════════════════════════════════${NC}"
  echo -e "${GREEN}  ViralCopy AI deployed successfully! 🚀${NC}"
  echo -e "${GREEN}══════════════════════════════════════════════════${NC}"
  echo ""
  echo "  Static IP:    $STATIC_IP"
  echo "  Frontend:     https://34-107-167-235.sslip.io"
  echo "  Backend API:  https://34-107-167-235.sslip.io"
  echo "  API Docs:     https://34-107-167-235.sslip.io/docs  (debug mode only)"
  echo ""
  echo -e "${YELLOW}Next steps:${NC}"
  echo "  1. Update DNS A records to $STATIC_IP"
  echo "  2. Wait for GKE ManagedCertificate provisioning (~15 min)"
  echo "  3. Configure GitHub Actions secrets (see .github/workflows/README.md)"
  echo ""
  echo "  Useful commands:"
  echo "    kubectl -n $NAMESPACE get pods"
  echo "    kubectl -n $NAMESPACE logs -l app=backend -f"
  echo "    kubectl -n $NAMESPACE logs -l app=frontend -f"
  echo "    gcloud sql connect $DB_INSTANCE --user=$DB_USER --database=$DB_NAME"
  echo ""
}

# ── Main ───────────────────────────────────────────────────────────────────────
main() {
  check_prerequisites

  if [[ "$SKIP_INFRA" == "false" ]]; then
    setup_gcp_project
    setup_artifact_registry
    setup_cloud_sql
    setup_gcs
    setup_iam
    setup_gke
    setup_static_ip
  fi

  if [[ "$SKIP_BUILD" == "false" ]]; then
    build_and_push
  fi

  apply_k8s
  wait_for_rollout
  run_migrations
  print_summary
}

main "$@"

```