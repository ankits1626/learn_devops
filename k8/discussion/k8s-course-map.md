# Kubernetes 101: From Zero to Production (Mac Edition)

> A hands-on course with mini-projects at each chapter. Each concept is learned by building something real.

---

## Course Overview

**Target:** Absolute beginners who want to run K8s in production
**Platform:** macOS (Apple Silicon / Intel)
**Duration:** 10 chapters, self-paced
**Final Project:** Deploy a full-stack app with CI/CD to a cloud cluster

---

## What We're Building

Throughout this course, we'll build a **Job Board Application** step-by-step:

```
┌─────────────────────────────────────────────────────────────┐
│                    JOB BOARD APP                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│   │  Frontend   │───▶│   Backend   │───▶│  Database   │    │
│   │   (React)   │    │   (Go/Node) │    │ (Postgres)  │    │
│   └─────────────┘    └─────────────┘    └─────────────┘    │
│                                                              │
│   Each chapter adds a new capability:                        │
│   - Ch 1-2: Run containers                                   │
│   - Ch 3: Connect services                                   │
│   - Ch 4: Configure apps                                     │
│   - Ch 5: Store data persistently                           │
│   - Ch 6: Expose to internet                                │
│   - Ch 7: Auto-heal & scale                                 │
│   - Ch 8: Secure everything                                 │
│   - Ch 9: Monitor & debug                                   │
│   - Ch 10: Deploy to production                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Chapter Map

### Chapter 0: First Principles & Mac Setup
**Goal:** Understand why K8s exists and set up your Mac for K8s development

**Concepts:**
- The evolution: Bare Metal → VMs → Containers → Orchestration
- What problems Kubernetes solves
- K8s architecture: Control Plane, Nodes, Pods
- Installing tools on Mac: Docker Desktop / Rancher Desktop, kubectl, kind

**Mini-Project:**
```
🎯 Set up a local 3-node Kubernetes cluster using kind
   Verify with: kubectl get nodes (should show 3 nodes)
```

**Folder Structure:**
```
chapter_0/
├── 0.1_why_kubernetes.md
├── 0.2_k8s_architecture.md
├── 0.3_mac_setup.md
├── 0.4_your_first_cluster.md
├── mini_project/
│   └── kind-cluster-config.yaml
└── summary.md
```

---

### Chapter 1: Pods - Your First Container in K8s
**Goal:** Run a single container in Kubernetes

**Concepts:**
- What is a Pod (smallest deployable unit)
- Pod lifecycle (Pending → Running → Succeeded/Failed)
- kubectl basics: apply, get, describe, logs, exec, delete
- YAML anatomy for Kubernetes

**Mini-Project:**
```
🎯 Deploy nginx in a Pod, exec into it, serve a custom HTML page
   Verify: kubectl exec into pod, curl localhost:80
```

**Folder Structure:**
```
chapter_1/
├── 1.1_what_is_a_pod.md
├── 1.2_pod_yaml_anatomy.md
├── 1.3_kubectl_essentials.md
├── 1.4_pod_lifecycle.md
├── mini_project/
│   ├── nginx-pod.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 2: Deployments & ReplicaSets
**Goal:** Run multiple replicas with self-healing

**Concepts:**
- Why not use Pods directly?
- ReplicaSets: Ensuring N pods are always running
- Deployments: Managing ReplicaSets + rolling updates
- Labels and Selectors (how K8s groups things)
- Rolling updates and rollbacks

**Mini-Project:**
```
🎯 Deploy 3 replicas of a web app, kill one pod, watch it recreate
   Then: Update image version, observe rolling update
```

**Folder Structure:**
```
chapter_2/
├── 2.1_why_not_just_pods.md
├── 2.2_replicasets.md
├── 2.3_deployments.md
├── 2.4_labels_selectors.md
├── 2.5_rolling_updates.md
├── mini_project/
│   ├── deployment.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 3: Services - Connecting Your Apps
**Goal:** Enable communication between Pods

**Concepts:**
- The problem: Pod IPs are ephemeral
- Service types: ClusterIP, NodePort, LoadBalancer
- Service discovery (DNS in K8s)
- Endpoints and how Services find Pods

**Mini-Project:**
```
🎯 Deploy frontend + backend as separate Deployments
   Connect them using a ClusterIP Service
   Frontend calls backend via service DNS name
```

**Folder Structure:**
```
chapter_3/
├── 3.1_why_services.md
├── 3.2_clusterip_service.md
├── 3.3_nodeport_loadbalancer.md
├── 3.4_service_discovery_dns.md
├── mini_project/
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 4: Configuration - ConfigMaps & Secrets
**Goal:** Externalize configuration from your containers

**Concepts:**
- Why not hardcode config?
- ConfigMaps: Non-sensitive configuration
- Secrets: Sensitive data (and why K8s secrets aren't really secret)
- Mounting as env vars vs files
- 2026 Best Practice: External Secrets Operator intro

**Mini-Project:**
```
🎯 Configure the backend with:
   - Database URL from ConfigMap
   - Database password from Secret
   App reads config from environment variables
```

**Folder Structure:**
```
chapter_4/
├── 4.1_why_externalize_config.md
├── 4.2_configmaps.md
├── 4.3_secrets.md
├── 4.4_mounting_config.md
├── 4.5_external_secrets_intro.md
├── mini_project/
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── backend-with-config.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 5: Storage - Persistent Data
**Goal:** Store data that survives Pod restarts

**Concepts:**
- The problem: Container storage is ephemeral
- Volumes, PersistentVolumes (PV), PersistentVolumeClaims (PVC)
- StorageClasses (dynamic provisioning)
- StatefulSets for databases

**Mini-Project:**
```
🎯 Deploy PostgreSQL with persistent storage
   Insert data, delete the Pod, verify data survives
```

**Folder Structure:**
```
chapter_5/
├── 5.1_ephemeral_vs_persistent.md
├── 5.2_volumes_pv_pvc.md
├── 5.3_storage_classes.md
├── 5.4_statefulsets.md
├── mini_project/
│   ├── postgres-pvc.yaml
│   ├── postgres-statefulset.yaml
│   ├── postgres-service.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 6: Exposing Apps - Gateway API & Ingress
**Goal:** Make your app accessible from outside the cluster

**Concepts:**
- The problem: Services are internal by default
- Ingress (legacy but still used)
- Gateway API (2026 standard)
- TLS termination and cert-manager
- Path-based and host-based routing

**Mini-Project:**
```
🎯 Expose the Job Board app via Gateway API
   - /api/* → backend service
   - /* → frontend service
   Access via localhost with HTTPS
```

**Folder Structure:**
```
chapter_6/
├── 6.1_why_gateway_api.md
├── 6.2_gateway_httproute.md
├── 6.3_tls_cert_manager.md
├── 6.4_legacy_ingress.md
├── mini_project/
│   ├── gateway.yaml
│   ├── httproute.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 7: Reliability - Health Checks & Autoscaling
**Goal:** Make your app self-healing and auto-scaling

**Concepts:**
- Probes: Liveness, Readiness, Startup
- Resource requests and limits
- Horizontal Pod Autoscaler (HPA)
- KEDA for event-driven scaling
- Vertical Pod Autoscaler (VPA)

**Mini-Project:**
```
🎯 Add health checks to all services
   Configure HPA to scale backend 2-10 replicas based on CPU
   Load test and watch pods scale up/down
```

**Folder Structure:**
```
chapter_7/
├── 7.1_health_probes.md
├── 7.2_resource_management.md
├── 7.3_hpa_autoscaling.md
├── 7.4_keda_event_driven.md
├── mini_project/
│   ├── backend-with-probes.yaml
│   ├── hpa.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 8: Security - RBAC & Pod Security
**Goal:** Secure your cluster and workloads

**Concepts:**
- Pod Security Standards (restricted, baseline, privileged)
- Security Context (non-root, read-only fs, capabilities)
- RBAC: Roles, RoleBindings, ServiceAccounts
- Network Policies (firewall rules)
- Workload Identity (IRSA/Workload Identity)

**Mini-Project:**
```
🎯 Harden the Job Board app:
   - All pods run as non-root
   - Network policy: only frontend can talk to backend
   - Dedicated ServiceAccount with minimal permissions
```

**Folder Structure:**
```
chapter_8/
├── 8.1_pod_security_standards.md
├── 8.2_security_context.md
├── 8.3_rbac.md
├── 8.4_network_policies.md
├── 8.5_workload_identity.md
├── mini_project/
│   ├── namespace-restricted.yaml
│   ├── network-policy.yaml
│   ├── serviceaccount.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 9: Observability - Logs, Metrics, Traces
**Goal:** Monitor and debug your applications

**Concepts:**
- The three pillars: Logs, Metrics, Traces
- Prometheus + Grafana stack
- Loki for logs
- OpenTelemetry (2026 standard)
- Alerting with PrometheusRules

**Mini-Project:**
```
🎯 Set up full observability:
   - Deploy Prometheus + Grafana
   - Create a dashboard for the Job Board
   - Set up alert for high error rate
```

**Folder Structure:**
```
chapter_9/
├── 9.1_three_pillars.md
├── 9.2_prometheus_grafana.md
├── 9.3_logging_loki.md
├── 9.4_opentelemetry.md
├── 9.5_alerting.md
├── mini_project/
│   ├── prometheus-values.yaml
│   ├── grafana-dashboard.json
│   ├── alerting-rules.yaml
│   └── README.md
└── summary.md
```

---

### Chapter 10: Production - AWS EKS Deployment (Free Tier Friendly)
**Goal:** Deploy to AWS EKS with cost optimization

**Concepts:**
- AWS Free Tier: What's included, what's not
- EKS setup with eksctl
- AWS Load Balancer Controller
- ECR for container images
- IAM Roles for Service Accounts (IRSA)
- Cost optimization: t3.micro nodes, spot instances

**Mini-Project:**
```
🎯 Deploy Job Board to AWS EKS:
   - Create EKS cluster with eksctl (free tier optimized)
   - Push images to ECR
   - Deploy app with ALB Ingress
   - Access via public URL
```

**Folder Structure:**
```
chapter_10/
├── 10.1_aws_free_tier_guide.md
├── 10.2_eks_cluster_setup.md
├── 10.3_ecr_container_registry.md
├── 10.4_aws_load_balancer.md
├── 10.5_irsa_security.md
├── 10.6_cost_optimization.md
├── mini_project/
│   ├── eksctl-cluster.yaml
│   ├── aws-lb-controller/
│   ├── ecr-push.sh
│   └── README.md
└── summary.md
```

---

### Chapter 11: GitOps - ArgoCD & CI/CD
**Goal:** Implement GitOps workflow for continuous deployment

**Concepts:**
- GitOps principles: Git as source of truth
- ArgoCD installation and setup
- Application manifests and sync policies
- Kustomize for environment management
- GitHub Actions CI/CD integration
- Secrets management with AWS Secrets Manager + ESO

**Mini-Project:**
```
🎯 Full GitOps pipeline:
   - Install ArgoCD on EKS
   - Create staging and production overlays
   - GitHub Actions: Build → Push to ECR → Update manifest
   - Push to Git → ArgoCD auto-deploys
```

**Folder Structure:**
```
chapter_11/
├── 11.1_gitops_principles.md
├── 11.2_argocd_setup.md
├── 11.3_kustomize_overlays.md
├── 11.4_github_actions_cicd.md
├── 11.5_secrets_management.md
├── mini_project/
│   ├── argocd-install/
│   ├── argocd-apps/
│   ├── kustomize/
│   │   ├── base/
│   │   └── overlays/
│   │       ├── staging/
│   │       └── production/
│   ├── .github/
│   │   └── workflows/
│   │       └── deploy.yaml
│   └── README.md
└── summary.md
```

---

## AWS Free Tier & Cost Guide

### What's Free (12 months)

| Service | Free Tier | Our Usage |
|---------|-----------|-----------|
| **EC2** | 750 hrs/month t2.micro or t3.micro | Worker nodes |
| **EBS** | 30 GB storage | Node volumes |
| **ECR** | 500 MB storage | Container images |
| **S3** | 5 GB storage | Terraform state (optional) |
| **Data Transfer** | 100 GB out | Traffic |

### What's NOT Free (Budget: ~$3-5/day)

| Service | Cost | Notes |
|---------|------|-------|
| **EKS Control Plane** | $0.10/hour (~$73/month) | Unfortunately not free |
| **NAT Gateway** | $0.045/hour + data | We'll avoid this |
| **ALB** | $0.0225/hour + LCU | ~$16/month minimum |

### Cost Optimization Strategy

```
┌─────────────────────────────────────────────────────────────┐
│              COST-OPTIMIZED EKS SETUP                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   Control Plane: $0.10/hr (unavoidable)                     │
│                                                              │
│   Worker Nodes:                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  2x t3.small SPOT instances (~$0.006/hr each)       │   │
│   │  = ~$9/month for compute                            │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                              │
│   Networking:                                                │
│   ┌─────────────────────────────────────────────────────┐   │
│   │  Public subnets only (no NAT Gateway = $0)          │   │
│   │  ALB for ingress (~$16/month)                       │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                              │
│   Total: ~$100/month if running 24/7                        │
│   Pro tip: Delete cluster when not learning!                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Money-Saving Commands

```bash
# Create cluster (start learning)
eksctl create cluster -f eksctl-cluster.yaml

# Delete cluster (done for the day) - SAVES MONEY!
eksctl delete cluster --name job-board-cluster

# Cluster creation takes ~15 min, so plan your learning sessions
```

---

## Tools We'll Use (Mac)

### Container Runtime (Choose One)

| Option | Pros | Cons | Install |
|--------|------|------|---------|
| **Podman Desktop** | Rootless, no daemon, free, OCI-native | Newer ecosystem | `brew install --cask podman-desktop` |
| **Docker Desktop** | Most popular, great UI | License for enterprise | `brew install --cask docker` |
| **Rancher Desktop** | K8s included, free | Heavier | `brew install --cask rancher` |

**Recommended:** Podman (2026 standard, truly free, more secure)

```bash
# Podman setup on Mac
brew install --cask podman-desktop
podman machine init
podman machine start

# Alias for Docker compatibility
alias docker=podman
```

### Kubernetes Tools

| Tool | Purpose | Install |
|------|---------|---------|
| **kubectl** | K8s CLI | `brew install kubectl` |
| **kind** | Local K8s clusters | `brew install kind` |
| **k9s** | Terminal UI | `brew install derailed/k9s/k9s` |
| **kubectx/kubens** | Context switching | `brew install kubectx` |
| **stern** | Multi-pod logs | `brew install stern` |
| **helm** | Package manager | `brew install helm` |
| **eksctl** | EKS cluster management | `brew install eksctl` |
| **aws-cli** | AWS command line | `brew install awscli` |

### Podman + Kind Setup

```bash
# Using kind with Podman (recommended)
KIND_EXPERIMENTAL_PROVIDER=podman kind create cluster

# Or set permanently
export KIND_EXPERIMENTAL_PROVIDER=podman
```

---

## Prerequisites

Before starting, you should:

- [ ] Have a Mac with at least 8GB RAM (16GB recommended)
- [ ] Know basic terminal commands (cd, ls, cat, etc.)
- [ ] Understand what containers are (Docker basics)
- [ ] Be comfortable reading YAML
- [ ] Have Homebrew installed (`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`)
- [ ] Have an AWS account (free tier eligible)
- [ ] AWS CLI configured with credentials (`aws configure`)

---

## Course Structure

```
k8/
├── discussion/
│   ├── k8s-course-map.md      # This file
│   └── learning-map.md        # Quick reference
│
├── chapter_0/                 # First Principles & Setup
├── chapter_1/                 # Pods
├── chapter_2/                 # Deployments
├── chapter_3/                 # Services
├── chapter_4/                 # ConfigMaps & Secrets
├── chapter_5/                 # Storage
├── chapter_6/                 # Gateway API
├── chapter_7/                 # Health & Scaling
├── chapter_8/                 # Security
├── chapter_9/                 # Observability
├── chapter_10/                # AWS EKS Deployment
├── chapter_11/                # GitOps & CI/CD
│
└── job-board/                 # Final project source code
    ├── frontend/
    ├── backend/
    └── k8s/
```

---

## Learning Path

```
Week 1-2:  Chapter 0-2   │  Setup + Pods + Deployments
Week 3:    Chapter 3-4   │  Services + Config
Week 4:    Chapter 5-6   │  Storage + Gateway API
Week 5:    Chapter 7-8   │  Reliability + Security
Week 6:    Chapter 9     │  Observability
Week 7:    Chapter 10-11 │  AWS EKS + GitOps (Production!)
```

---

## Quick Reference

### Essential kubectl Commands
```bash
kubectl get pods                    # List pods
kubectl get all                     # List everything
kubectl describe pod <name>         # Debug a pod
kubectl logs <pod>                  # View logs
kubectl logs -f <pod>               # Stream logs
kubectl exec -it <pod> -- sh        # Shell into pod
kubectl apply -f <file>             # Create/update
kubectl delete -f <file>            # Delete
kubectl port-forward <pod> 8080:80  # Local access
```

### YAML Template (Every K8s Resource)
```yaml
apiVersion: <api-version>
kind: <resource-type>
metadata:
  name: <resource-name>
  namespace: <namespace>
  labels:
    app: <app-name>
spec:
  # Resource-specific configuration
```

---

## Ready to Start?

1. Read Chapter 0 concepts
2. Set up your Mac environment
3. Create your first cluster
4. Build something!

**Let's go! 🚀**

---

*Course Version: 1.0 | K8s Versions: 1.29-1.32 | Last Updated: January 2026*
