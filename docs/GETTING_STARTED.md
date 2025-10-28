# BookVerse Platform - Getting Started Guide

**Complete setup and deployment instructions for the BookVerse microservices demo**

This guide provides step-by-step instructions to set up the complete BookVerse demo environment, including all microservices and the JFrog Platform configuration for secure software delivery.

---


## 🛡️ Governance & Policy Framework

BookVerse implements **enterprise-grade governance** through comprehensive unified policies that ensure quality, security, and compliance throughout the software development lifecycle.

### **🎯 Policy-Driven Development**

The BookVerse platform enforces **14 unified policies** across 4 lifecycle stages:

- **🔍 DEV Stage**: 6 policies ensuring code quality, security, and traceability
- **🧪 QA Stage**: 2 policies for comprehensive testing and security validation  
- **🚀 STAGING Stage**: 3 policies for enterprise compliance and change management
- **🏭 PROD Release**: 3 policies for final release validation and approval

### **📊 What This Means for Developers**

- **Automated Quality Gates**: Policies automatically enforce quality standards
- **Evidence Collection**: Required evidence is automatically collected during CI/CD
- **Compliance Reporting**: Real-time compliance status and audit trails
- **Guided Workflows**: Clear guidance on requirements for each stage

### **🔒 Enterprise Readiness**

BookVerse demonstrates production-ready governance capabilities:

- **Security by Design**: SAST, DAST, and penetration testing requirements
- **Supply Chain Security**: SLSA provenance and cryptographic evidence signing
- **Change Management**: ServiceNow integration for enterprise change control  
- **Audit Compliance**: Complete audit trails for regulatory compliance

## 📋 Prerequisites

### 🔑 **Required Access & Accounts**

> **Note for Local Demo**: The requirements below are for the JFrog Platform setup and GitHub integration. No external domain access is needed - the local demo uses `bookverse.demo` and `argocd.demo` domains configured in your local `/etc/hosts` file.

| Requirement | Description | Purpose |
|-------------|-------------|---------|
| **JFrog Platform** | Admin privileges on Artifactory + AppTrust | Platform provisioning and artifact management |
| **GitHub Organization** | Repository creation permissions | Source code hosting and CI/CD automation |
| **GitHub Personal Access Token** | Classic token with repo and workflow permissions | Repository management and workflow execution |

### 🛠️ **Required Tools**

#### **Core Tools** (Required)
```bash
# Verify installations
gh --version      # GitHub CLI (required: v2.0+)
curl --version    # HTTP client (usually pre-installed)
jq --version      # JSON processor (required: v1.6+)
git --version     # Git client (required: v2.25+)
```

#### **Container Tools** (Optional - for manual setup)
```bash
docker --version   # Container runtime (v20.10+)
kubectl version    # Kubernetes client (v1.21+)
helm version       # Helm package manager (v3.7+)
```

#### **Kubernetes Tools** (Required for local demo)
```bash
# Rancher Desktop (Recommended for macOS/Windows)
# Download and install from: https://rancherdesktop.io/
# - Provides Docker, kubectl, and Kubernetes cluster
# - No additional configuration needed for BookVerse demo
rancher-desktop --version  # Verify installation

# Alternative: Docker Desktop + kubectl
docker --version   # Container runtime (v20.10+)
kubectl version    # Kubernetes client (v1.21+)
helm version       # Helm package manager (v3.7+)
```

### 💻 **Tool Installation**

<details>
<summary><strong>📱 macOS Installation</strong></summary>

```bash
# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install core tools
brew install gh curl jq git

# Install Rancher Desktop (Recommended for BookVerse demo)
brew install --cask rancher

# Optional: Install tools separately if not using Rancher Desktop
# brew install docker kubectl helm

# Verify installations
for tool in gh curl jq git; do
  echo "✓ $tool: $($tool --version | head -1)"
done

# Verify Rancher Desktop
echo "✓ Rancher Desktop: Check Applications folder or run from Applications"
```
</details>

<details>
<summary><strong>🐧 Linux Installation (Ubuntu/Debian)</strong></summary>

```bash
# Update package list
sudo apt update

# Install core tools
sudo apt install -y curl jq git

# Install GitHub CLI
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install -y gh

# Install Rancher Desktop
# Download from: https://rancherdesktop.io/

# Optional: Container tools
curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
</details>

---

## 🚀 BookVerse Demo Setup

### 📥 **Step 1: Clone All BookVerse Repositories**

The BookVerse demo consists of multiple service repositories that need to be cloned individually:

```bash
# 1. Authenticate with GitHub (required for private repositories)
gh auth login
gh auth status  # Verify authentication

# 2. Create workspace directory
mkdir bookverse-demo
cd bookverse-demo

# 3. Clone all service repositories
git clone https://github.com/your-org/bookverse-inventory.git
git clone https://github.com/your-org/bookverse-recommendations.git
git clone https://github.com/your-org/bookverse-checkout.git
git clone https://github.com/your-org/bookverse-platform.git
git clone https://github.com/your-org/bookverse-web.git
git clone https://github.com/your-org/bookverse-helm.git
git clone https://github.com/your-org/bookverse-infra.git

# 4. Clone the demo orchestration repository
git clone https://github.com/your-org/bookverse-demo-init.git

# 5. Verify all repositories are cloned
ls -la
# Expected: bookverse-inventory/, bookverse-recommendations/, bookverse-checkout/, 
#          bookverse-platform/, bookverse-web/, bookverse-helm/, bookverse-infra/,
#          bookverse-demo-init/
```

### 🔧 **Step 2: Configure JFrog Platform Connection**

```bash
# 1. Navigate to the orchestration repository
cd bookverse-demo-init

# 2. Set up your JFrog Platform connection
export JFROG_URL="https://your-instance.jfrog.io"
export JFROG_ADMIN_TOKEN="your-admin-token"

# 3. Configure GitHub repository secrets (recommended)
gh secret set JFROG_ADMIN_TOKEN --body "$JFROG_ADMIN_TOKEN"
echo "✅ JFROG_ADMIN_TOKEN secret configured for repository"

gh secret set JFROG_URL --body "$JFROG_URL"
echo "✅ JFROG_URL secret configured for repository"

# 4. Verify connectivity
curl -s --header "Authorization: Bearer ${JFROG_ADMIN_TOKEN}" \
  "${JFROG_URL}/artifactory/api/system/ping"
# Expected output: OK
```

### ☸️ **Step 3: Kubernetes Setup (Optional but Recommended)**

If you plan to deploy the demo to Kubernetes (recommended for full experience), set up your cluster now.

#### **🎯 Kubernetes Options**

BookVerse demo supports multiple Kubernetes deployment options:

**Option 1: Rancher Desktop (Recommended for Local Development)**
- **Pros**: Easy setup, includes Docker + Kubernetes + kubectl
- **Cons**: Local only, resource intensive  
- **Best for**: Development, testing, demos

**Option 2: Cloud Kubernetes (AWS EKS, GKE, AKS)**
- **Pros**: Production-grade, scalable, managed
- **Cons**: Requires cloud account, cost implications
- **Best for**: Production deployments, team environments

**Option 3: Other Local Options**
- **Docker Desktop**: Alternative to Rancher Desktop
- **minikube**: Lightweight local Kubernetes
- **k3s**: Lightweight Kubernetes distribution

#### **🚀 Rancher Desktop Setup (Recommended)**

```bash
# 1. Install Rancher Desktop (if not already installed)
# macOS: brew install --cask rancher
# Windows: Download from https://rancherdesktop.io/
# Linux: Download from https://rancherdesktop.io/

# 2. Start Rancher Desktop
# - Open Rancher Desktop application
# - Enable Kubernetes in settings  
# - Wait for cluster to be ready (green status)

# 3. Verify Kubernetes cluster
kubectl cluster-info
kubectl get nodes

# Expected output:
# Kubernetes control plane is running at https://127.0.0.1:6443
# NAME                   STATUS   ROLES                  AGE
# rancher-desktop        Ready    control-plane,master   1m
```

> **Note**: If you set up Kubernetes now, make sure to set `Update K8s: true` in Step 4 to configure the registry for your cluster.

---
### 🔄 **Step 4: Switch Platform (Configure Target JFrog Platform)**

Run the Switch Platform workflow to configure your JFrog Platform instance:

```bash
# Navigate to GitHub Actions in your bookverse-demo-init repository
# Go to: https://github.com/your-org/bookverse-demo-init/actions

# 1. Select "🔄 Switch Platform" workflow
# 2. Click "Run workflow" 
# 3. Enter the following inputs:
#    - JFrog Platform Host: https://your-instance.jfrog.io
#    - Admin Token: (leave empty - secret is already configured)
#    - Confirmation: SWITCH
#    - Update K8s: true (if you set up Kubernetes in Step 3) or false (if skipping Kubernetes)

# OR run via GitHub CLI (admin token not needed since secret is configured):
gh workflow run "🔄-switch-platform.yml" \
  --field jpd_host="https://your-instance.jfrog.io" \
  --field confirm_switch="SWITCH" \
  --field update_k8s=true```

### 🚀 **Step 5: Setup Platform (Provision Complete Environment)**

Run the Setup Platform workflow to provision the entire BookVerse environment:

```bash
# Navigate to GitHub Actions in your bookverse-demo-init repository
# Go to: https://github.com/your-org/bookverse-demo-init/actions

# 1. Select "🚀 Setup Platform" workflow  
# 2. Click "Run workflow"
# 3. Monitor the workflow progress

# OR run via GitHub CLI:
gh workflow run "🚀-setup-platform.yml"

# Monitor workflow status
gh run list --workflow="🚀-setup-platform.yml"
```

The Setup Platform workflow will automatically:
- Create the `bookverse` project in JFrog Platform
- Set up JFrog artifact repositories across all package types (Docker, Python, npm, etc.)
- Configure AppTrust applications with lifecycle stages
- Create OIDC integrations for GitHub authentication
- Set up users and role-based access control
- Generate evidence keys for cryptographic signing
- Configure all GitHub repository variables

**The workflow includes comprehensive validation and will fail if any setup step encounters issues. No additional verification is needed.**

---

## 🎯 GitHub Repository Overview

The BookVerse demo consists of these **GitHub repositories** (source code):

> **Note**: This section describes GitHub repositories containing source code. The JFrog Platform will automatically create separate artifact repositories (Docker, Python, npm, etc.) for storing build artifacts.

| GitHub Repository | Purpose | Technology Stack |
|------------|---------|------------------|
| **bookverse-inventory** | Product catalog & inventory management | Python, FastAPI, SQLite |
| **bookverse-recommendations** | AI-powered recommendation engine | Python, scikit-learn, FastAPI |
| **bookverse-checkout** | Order processing & payments | Python, FastAPI, PostgreSQL |
| **bookverse-platform** | Service orchestration & aggregation | Python, FastAPI |
| **bookverse-web** | Frontend user interface | Vanilla JS, Vite, HTML5 |
| **bookverse-helm** | Kubernetes deployment charts | Helm 3, YAML |
| **bookverse-infra** | Shared libraries & DevOps tools | Python (bookverse-core), Shell scripts |
| **bookverse-demo-init** | Demo orchestration & platform setup | GitHub Actions, Shell scripts |

---

## ☸️ Kubernetes Demo Deployment

#### **BookVerse Demo Deployment**

If you set up Kubernetes in Step 3, you can now prepare the demo infrastructure:

> **⚠️ Important**: This step sets up the Kubernetes infrastructure (ArgoCD, namespaces, ingress) but **no application pods will be deployed initially**. Application pods only deploy after platform application versions are released to production through the CI/CD promotion workflow. This is preparation for when you later trigger builds and promotions.
```bash
# 1. Navigate to demo-init repository (if not already there)
cd bookverse-demo-init

# 2. Run the demo bootstrap script
./scripts/bookverse-demo.sh --setup

# This script will:
# - Detect your Kubernetes context
# - Install ArgoCD (for GitOps deployment)
# - Deploy BookVerse applications
# - Configure ingress and networking
# - Set up demo data

# 3. Verify infrastructure deployment
kubectl get pods -n argocd  # Check ArgoCD is running
kubectl get namespace bookverse-prod  # Check namespace exists
kubectl get ingress -n bookverse-prod  # Check ingress is configured

# Note: Application pods will be empty initially - they deploy after CI/CD promotes versions
kubectl get pods -n bookverse-prod  # Will show "No resources found" initially
### 🌐 **Access Demo Infrastructure**

After successful infrastructure deployment:
```bash
# The demo script will configure local DNS and provide access URLs:
echo "🔧 ArgoCD Interface: https://argocd.demo"  # Available immediately
echo "🌐 BookVerse Demo: http://bookverse.demo"   # Available after CI/CD promotes versionsecho "🔧 ArgoCD Interface: https://argocd.demo"

# How this works:
# 1. The demo script modifies /etc/hosts to point demo domains to 127.0.0.1
# 2. Traefik ingress controller routes requests to the appropriate services
# 3. No external DNS or domain registration required

### 🚀 **When Will Applications Be Available?**

The BookVerse applications will become available after:
1. **Triggering CI/CD builds** for each service (make code changes and push)
2. **Promoting through stages** (DEV → QA → STAGING → PROD)
3. **Production deployment** completes via ArgoCD

Until then, only ArgoCD will be accessible - the application pods remain undeployed.

> **📖 Want to understand what this setup does?** See the [GitOps Deployment Guide](GITOPS_DEPLOYMENT.md) for a detailed explanation of the ArgoCD configuration and GitOps workflow that gets automatically configured.``````

### 🔧 **Alternative Kubernetes Options**

<details>
<summary><strong>⚡ Quick Kubernetes Bootstrap (K8s-Only Path)</strong></summary>

**For users who want to skip JFrog Platform setup and deploy directly to Kubernetes**

This streamlined path uses the lightweight bootstrap script for immediate Kubernetes deployment without the full platform provisioning workflow. Perfect for evaluation, testing, or when you already have container images available.

> **💡 Note**: This path skips JFrog Platform setup and CI/CD integration. For the complete experience with automated builds and promotions, follow the main Getting Started flow above.

#### Prerequisites
- kubectl and helm installed
- A local cluster (Rancher Desktop recommended)  
- Container registry credentials (hostname, username, password/token; email optional)

#### Important Configuration Notes

⚠️ **Web Application Backend URLs**: The BookVerse web application requires different backend URL configurations:

- **Production/Kubernetes**: Uses internal service names (`http://inventory`, `http://recommendations`, `http://checkout`)
- **Local Development**: Requires localhost URLs with port-forwarding (`http://localhost:8001`, `http://localhost:8003`, `http://localhost:8002`)

For local testing with port-forwarding, you may need to update the web configuration manually. See the troubleshooting section in DEMO_RUNBOOK.md for details.

#### Quick Setup Steps

**1) Start from a clean slate (optional)**
```bash
cd bookverse-demo-init
./scripts/k8s/cleanup.sh --all
```

**2) Bootstrap Argo CD and deploy BookVerse (PROD)**
```bash
cd bookverse-demo-init
export REGISTRY_SERVER='your-instance.jfrog.io'       # JFrog hostname (without https://)
export REGISTRY_USERNAME='<jfrog-username>'
export REGISTRY_PASSWORD='<jfrog-password-or-token>'
export REGISTRY_EMAIL='you@example.com'   # optional; your JFrog user email is fine
./scripts/k8s/bootstrap.sh --port-forward

# Or local JFrog (example)
export REGISTRY_SERVER='localhost:8082'
export REGISTRY_USERNAME='admin'
export REGISTRY_PASSWORD='<admin-password-or-token>'
# REGISTRY_EMAIL optional
./scripts/k8s/bootstrap.sh --port-forward

# Access URLs:
# Argo CD UI: https://localhost:8081 (accept self-signed cert)
# ArgoCD admin password: S7w7PDUML4HT6sEw
# Web app: http://localhost:8080
```

**What the bootstrap script does:**
- Installs Argo CD in `argocd` namespace
- Creates `bookverse-prod` namespace
- Adds `imagePullSecrets` if all REGISTRY_* vars are provided (no defaults assumed)
- Applies `gitops/projects/bookverse-prod.yaml` and `gitops/apps/prod/platform.yaml`
- Configures ArgoCD with production settings (TLS, security headers, proper ingress)
- Waits for the Argo CD Application to be Synced and Healthy
- Optionally starts port-forwards for Argo CD and the web app

**3) Verify deployment**
```bash
kubectl -n argocd get applications.argoproj.io platform-prod
kubectl -n bookverse-prod get deploy,svc,pod
```

#### ArgoCD Production Configuration

The bootstrap includes bulletproof ArgoCD configuration that provides:
- **Secure TLS**: Self-signed certificates with proper SANs for demo use
- **Production Security**: Security headers, CSP, and HSTS via Traefik middleware  
- **gRPC-Web Support**: Full UI functionality with secure transport
- **Proper Ingress**: Correct routing to ArgoCD's internal HTTPS port (8080)
- **Automatic Setup**: No manual configuration required

#### Troubleshooting
- **ImagePullBackOff**: ensure `REGISTRY_*` variables were set before running bootstrap; re-run `bootstrap.sh` or restart deployments:
```bash
kubectl -n bookverse-prod rollout restart deploy
```
- **Argo CD app OutOfSync/Missing tags**: confirm `bookverse-helm/charts/platform/values.yaml` on `main` contains non-empty tags for all services (the demo CI/CD sets these on release).
- **ArgoCD connectivity issues**: The bootstrap now automatically applies production configuration. If issues persist, manually run:
```bash
./scripts/k8s/configure-argocd-production.sh --host argocd.demo
```
- **Access without port-forward**: enable Ingress in the Helm chart (`web.ingress.enabled=true`, set `web.ingress.host`) and expose via your local ingress controller (e.g., Traefik).

#### Uninstall
```bash
cd bookverse-demo-init
./scripts/k8s/cleanup.sh --all
```

**Notes:**
- This demo intentionally deploys only to `bookverse-prod`. DEV/QA/STAGING are not connected or monitored by Argo CD.
- Argo CD pulls manifests from the public `bookverse-helm` GitHub repo; images are pulled from JFrog registry using your provided credentials.

</details>

<details>
<summary><strong>🌥️ Cloud Kubernetes (AWS EKS)</strong></summary>

```bash
# Prerequisites: AWS CLI configured with appropriate permissions

# 1. Create EKS cluster
eksctl create cluster \
  --name bookverse-demo \
  --region us-west-2 \
  --nodegroup-name workers \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-min 1 \
  --nodes-max 3

# 2. Update kubeconfig
aws eks update-kubeconfig --region us-west-2 --name bookverse-demo

# 3. Verify cluster
kubectl cluster-info
kubectl get nodes
```
</details>

<details>
<summary><strong>🐳 Docker Desktop Alternative</strong></summary>

```bash
# 1. Install Docker Desktop
# Download from: https://www.docker.com/products/docker-desktop

# 2. Enable Kubernetes
# Docker Desktop → Settings → Kubernetes → Enable Kubernetes

# 3. Verify setup
kubectl cluster-info
kubectl get nodes
```
</details>

<details>
<summary><strong>⚡ minikube Setup</strong></summary>

```bash
# 1. Install minikube
# macOS: brew install minikube
# Linux: curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

# 2. Start minikube
minikube start --memory=8192 --cpus=4

# 3. Verify setup
kubectl cluster-info
kubectl get nodes
```
</details>

---

## 🆘 Troubleshooting

### 🔍 **Common Setup Issues**

<details>
<summary><strong>❌ JFrog connectivity issues</strong></summary>

**Problem**: Cannot connect to JFrog Platform

**Solutions**:
```bash
# 1. Verify URL format (must include https://)
echo $JFROG_URL  # Should be: https://your-instance.jfrog.io

# 2. Test connectivity
curl -s -H "Authorization: Bearer $JFROG_ADMIN_TOKEN" "$JFROG_URL/artifactory/api/system/ping"
# Expected: OK

# 3. Verify token permissions
curl -s -H "Authorization: Bearer $JFROG_ADMIN_TOKEN" "$JFROG_URL/access/api/v1/system/ping"
```
</details>

<details>
<summary><strong>❌ GitHub authentication problems</strong></summary>

**Problem**: GitHub CLI authentication fails

**Solutions**:
```bash
# 1. Re-authenticate with GitHub
gh auth logout
gh auth login --with-token < your-token-file

# 2. Verify authentication and permissions
gh auth status
gh repo list --limit 1  # Test API access

# 3. Check workflow permissions
gh api repos/your-org/bookverse-demo-init/actions/permissions
```
</details>

<details>
<summary><strong>❌ Rancher Desktop setup issues</strong></summary>

**Problem**: Rancher Desktop cluster not ready

**Solutions**:
```bash
# 1. Check Rancher Desktop status
# Open Rancher Desktop → Troubleshooting → Reset Kubernetes

# 2. Verify Docker is working
docker ps

# 3. Check kubectl context
kubectl config current-context
kubectl config get-contexts

# 4. Restart Rancher Desktop if needed
# Quit application and restart
```
</details>

<details>
<summary><strong>❌ Kubernetes deployment issues</strong></summary>

**Problem**: Pods stuck in Pending state

**Solutions**:
```bash
# 1. Check node resources
kubectl describe nodes
kubectl top nodes

# 2. Check pod details
kubectl describe pod <pod-name> -n bookverse-prod

# 3. Check for resource constraints
kubectl get events --sort-by=.metadata.creationTimestamp -n bookverse-prod
```

**Problem**: Cannot access demo URLs

**Solutions**:
```bash
# 1. Check ingress configuration
kubectl get ingress -n bookverse-prod
kubectl describe ingress -n bookverse-prod

# 2. Verify /etc/hosts modifications
cat /etc/hosts | grep bookverse

# 3. Check if demo script completed successfully
./scripts/bookverse-demo.sh
```
</details>

<details>
<summary><strong>❌ Workflow execution failures</strong></summary>

**Problem**: GitHub Actions workflows fail

**Solutions**:
```bash
# 1. Check workflow run details
gh run list --workflow="🚀-setup-platform.yml"
gh run view --log <run-id>

# 2. Verify repository secrets and variables
gh secret list -R your-org/bookverse-demo-init
gh variable list -R your-org/bookverse-demo-init

# 3. Check workflow permissions
# Go to Settings > Actions > General > Workflow permissions
# Ensure "Read and write permissions" is selected
```
</details>

---

## 🎯 Next Steps

### 🏗️ **Explore the Demo**

Ready to explore BookVerse? Here's what you can do:

- **📊 Monitor Pipelines**: Watch CI/CD pipelines in GitHub Actions
- **🔐 Explore AppTrust**: Review evidence collection and SBOM generation in JFrog Platform
- **☸️ Deploy to Kubernetes**: Use the optional Kubernetes deployment for runtime testing
- **🧪 Test Services**: Make changes to services and observe the promotion workflows

### 📚 **Learn More**

- **📖 [Demo Runbook](DEMO_RUNBOOK.md)** - Step-by-step demo execution guide
- **🏗️ [Architecture Guide](ARCHITECTURE.md)** - Understanding system design
- **🔄 [CI/CD Deployment](CICD_DEPLOYMENT_GUIDE.md)** - Pipeline configuration and automation
- **🔐 [OIDC Authentication](OIDC_AUTHENTICATION.md)** - Zero-trust authentication setup
- **🚀 [GitOps Deployment](GITOPS_DEPLOYMENT.md)** - Understanding the automated ArgoCD setup
- **🔗 [JFrog Integration](JFROG_INTEGRATION.md)** - Artifact management and security

---
## ✅ Setup Checklist

Use this checklist to ensure successful demo setup:

### 🔧 **Prerequisites**
- [ ] JFrog Platform access with admin privileges
- [ ] GitHub organization with repository creation permissions
- [ ] Required tools installed and verified (gh, curl, jq, git)
- [ ] GitHub CLI authenticated
- [ ] Rancher Desktop installed and running

### 📥 **Repository Setup**
- [ ] All GitHub service repositories cloned (inventory, recommendations, checkout, platform, web, helm, infra)
- [ ] Demo orchestration repository cloned (bookverse-demo-init)
- [ ] JFrog Platform connectivity verified

### ☸️ **Kubernetes Setup (Optional)**
- [ ] Kubernetes cluster running (Rancher Desktop or alternative)
- [ ] kubectl access verified
- [ ] Cluster ready for deployment

### 🔄 **Platform Configuration**
- [ ] Switch Platform workflow executed successfully
- [ ] Setup Platform workflow executed successfully

- [ ] Repository variables configured correctly

### ☸️ **Kubernetes Deployment**
- [ ] Kubernetes cluster access verified (Rancher Desktop or alternative)
- [ ] Demo bootstrap script executed
- [ ] Applications deployed and accessible
- [ ] All pods running and healthy

**🎉 Congratulations! Your BookVerse demo environment is ready.**

---

*Need help? Check our [Demo Runbook](DEMO_RUNBOOK.md) or reach out to the BookVerse team.*
