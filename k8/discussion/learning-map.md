# Kubernetes Learning Map: From Zero to Production (2026 Edition)

> **2026 Best Practices Applied**: This guide uses modern tooling including Gateway API, Cilium, External Secrets Operator, KEDA, and Platform Engineering patterns.

## Prerequisites Checklist

Before diving into Kubernetes, ensure you understand:

- [ ] Linux basics (commands, file system, processes)
- [ ] Containers (Docker/Podman fundamentals)
- [ ] Networking basics (IP, DNS, ports, load balancing)
- [ ] YAML syntax
- [ ] Basic understanding of distributed systems
- [ ] Git fundamentals (required for GitOps)

---

## Phase 1: First Principles - Why Kubernetes Exists

### 1.1 The Problem Statement

```
Traditional Deployment → VMs → Containers → Container Orchestration → Platform Engineering
```

**Evolution of deployment:**

| Era | Approach | Problem It Solved | New Problem Created |
|-----|----------|-------------------|---------------------|
| 1990s | Bare Metal | Direct hardware access | Resource waste, slow scaling |
| 2000s | Virtual Machines | Better resource utilization | Heavy, slow boot, OS overhead |
| 2013+ | Containers | Lightweight, fast, portable | Managing 100s of containers |
| 2014+ | Kubernetes | Orchestration at scale | Complexity, steep learning curve |
| 2024+ | Platform Engineering | Developer experience | Requires mature K8s foundation |

### 1.2 What Kubernetes Actually Does

Kubernetes is a **container orchestration platform** that:

1. **Schedules** - Decides where containers run
2. **Scales** - Adds/removes container instances
3. **Heals** - Restarts failed containers
4. **Balances** - Distributes traffic across containers
5. **Updates** - Rolls out changes without downtime

### 1.3 Core Mental Model

```
┌─────────────────────────────────────────────────────────────┐
│                    KUBERNETES CLUSTER                        │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              CONTROL PLANE (Master)                  │    │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │    │
│  │  │API Server│ │Scheduler │ │Controller│ │  etcd  │ │    │
│  │  └──────────┘ └──────────┘ └──────────┘ └────────┘ │    │
│  └─────────────────────────────────────────────────────┘    │
│                            │                                 │
│  ┌─────────────────────────┼─────────────────────────────┐  │
│  │                    WORKER NODES                        │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   │  │
│  │  │   Node 1    │  │   Node 2    │  │   Node 3    │   │  │
│  │  │ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │   │  │
│  │  │ │  Pod    │ │  │ │  Pod    │ │  │ │  Pod    │ │   │  │
│  │  │ │┌───────┐│ │  │ │┌───────┐│ │  │ │┌───────┐│ │   │  │
│  │  │ ││Container│ │  │ ││Container│ │  │ ││Container│ │   │  │
│  │  │ │└───────┘│ │  │ │└───────┘│ │  │ │└───────┘│ │   │  │
│  │  │ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │   │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘   │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 2: Core Concepts (Week 1-2)

### 2.1 The Building Blocks

Learn these in order - each builds on the previous:

#### Level 1: Pod
- Smallest deployable unit
- Contains one or more containers
- Shares network namespace (localhost communication)
- Ephemeral - can die and be replaced

```yaml
# pod.yaml - Your first K8s object
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.27-alpine  # Always use specific tags, never :latest
    ports:
    - containerPort: 80
    resources:                 # 2026: Always set resources
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
```

#### Level 2: ReplicaSet
- Ensures N copies of a Pod are running
- Self-healing: recreates failed Pods
- Rarely used directly (Deployments manage them)

#### Level 3: Deployment
- Manages ReplicaSets
- Handles rolling updates and rollbacks
- **This is what you'll use 90% of the time**

```yaml
# deployment.yaml - 2026 Production-Ready Template
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app.kubernetes.io/name: my-app
    app.kubernetes.io/version: "1.0.0"
    app.kubernetes.io/component: backend
spec:
  replicas: 3
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: my-app
  template:
    metadata:
      labels:
        app.kubernetes.io/name: my-app
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
    spec:
      securityContext:              # 2026: Pod-level security
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: my-app
        image: my-app:v1.0.0        # Use immutable tags
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
          name: http
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            # Note: CPU limits often omitted in 2026 (causes throttling)
        securityContext:            # 2026: Container-level security
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
              - ALL
        livenessProbe:
          httpGet:
            path: /healthz
            port: http
          initialDelaySeconds: 10
          periodSeconds: 15
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
        startupProbe:               # 2026: Use startup probes for slow-starting apps
          httpGet:
            path: /healthz
            port: http
          failureThreshold: 30
          periodSeconds: 10
      topologySpreadConstraints:    # 2026: Spread pods across zones
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: ScheduleAnyway
        labelSelector:
          matchLabels:
            app.kubernetes.io/name: my-app
```

#### Level 4: Service
- Stable network endpoint for Pods
- Load balances across Pod replicas
- Types: ClusterIP, NodePort, LoadBalancer

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  labels:
    app.kubernetes.io/name: my-app
spec:
  selector:
    app.kubernetes.io/name: my-app
  ports:
  - port: 80
    targetPort: http
    protocol: TCP
    name: http
  type: ClusterIP
```

### 2.2 Concept Relationship Map

```
                    User Request
                         │
                         ▼
                 ┌──────────────┐
                 │ Gateway API  │  (2026: Replaces Ingress)
                 └──────┬───────┘
                        │
                        ▼
                   ┌─────────┐
                   │ Service │  (Internal load balancer)
                   └────┬────┘
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
     ┌─────────┐   ┌─────────┐   ┌─────────┐
     │   Pod   │   │   Pod   │   │   Pod   │
     └────┬────┘   └────┬────┘   └────┬────┘
          │             │             │
          └─────────────┴─────────────┘
                        │
                        ▼
                 ┌────────────┐
                 │ Deployment │  (Manages Pods)
                 └────────────┘
```

---

## Phase 3: Hands-On Setup (Week 2)

### 3.1 Local Development Options (2026 Recommendations)

| Tool | Pros | Cons | Best For | 2026 Status |
|------|------|------|----------|-------------|
| **kind** | Fast, lightweight, K8s native | Limited addons | CI/CD, learning | **Recommended** |
| **minikube** | Full K8s, many addons | Resource heavy | Feature testing | Good |
| **k3d** | K3s in Docker, fast | K3s not full K8s | Edge simulation | Good |
| **Rancher Desktop** | GUI, containerd/dockerd | Mac/Windows only | Desktop dev | Good |
| **Podman Desktop** | Rootless, OCI-native | Newer ecosystem | Security-focused | Rising |

```bash
# 2026 Recommended: kind setup
# Install kind
brew install kind  # macOS
# or: go install sigs.k8s.io/kind@latest

# Create cluster with config
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
- role: worker
- role: worker
EOF
```

### 3.2 Essential CLI Commands

```bash
# Cluster info
kubectl cluster-info
kubectl get nodes
kubectl version --short

# Working with resources
kubectl get pods                    # List pods
kubectl get pods -o wide            # More details
kubectl get all                     # All resources
kubectl describe pod <name>         # Detailed info
kubectl get pods -A                 # All namespaces

# Creating resources
kubectl apply -f deployment.yaml    # Create/update from file
kubectl apply -k ./kustomize/       # 2026: Use Kustomize
kubectl create deployment nginx --image=nginx:1.27-alpine

# Debugging
kubectl logs <pod-name>             # View logs
kubectl logs -f <pod-name>          # Stream logs
kubectl logs -l app=my-app          # Logs by label
kubectl exec -it <pod-name> -- /bin/sh  # Shell into pod
kubectl debug <pod-name> -it --image=busybox  # 2026: Debug containers

# Scaling
kubectl scale deployment my-app --replicas=5

# Rolling updates
kubectl rollout status deployment/my-app
kubectl rollout history deployment/my-app
kubectl rollout undo deployment/my-app

# Deleting
kubectl delete pod <name>
kubectl delete -f deployment.yaml
```

### 3.3 Modern CLI Tools (2026)

```bash
# k9s - Terminal UI for K8s (essential)
brew install derailed/k9s/k9s
k9s

# kubectx/kubens - Context/namespace switching
brew install kubectx
kubectx my-cluster
kubens my-namespace

# stern - Multi-pod log tailing
brew install stern
stern my-app

# kubecolor - Colorized kubectl output
brew install kubecolor
alias kubectl=kubecolor
```

### 3.4 Practice Exercises

1. **Exercise 1**: Deploy nginx with the 2026 template, expose it, access it
2. **Exercise 2**: Scale the deployment, watch pod distribution
3. **Exercise 3**: Update the image version, observe rolling update
4. **Exercise 4**: Kill a pod, watch self-healing
5. **Exercise 5**: Use k9s to explore the cluster

---

## Phase 4: Configuration & Storage (Week 3)

### 4.1 ConfigMaps - External Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "db.example.com"
  LOG_LEVEL: "info"
  config.yaml: |
    server:
      port: 8080
    features:
      newUI: true
```

### 4.2 Secrets Management (2026 Best Practice)

**Old way (avoid):** Native K8s Secrets (base64, not encrypted at rest by default)

**2026 Best Practice:** External Secrets Operator with a secrets backend

```yaml
# External Secrets Operator - Syncs from AWS Secrets Manager, Vault, etc.
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: app-secrets
    creationPolicy: Owner
  data:
  - secretKey: database-password
    remoteRef:
      key: prod/my-app/database
      property: password
  - secretKey: api-key
    remoteRef:
      key: prod/my-app/api
      property: key
```

```yaml
# SecretStore configuration (AWS example)
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: aws-secrets-manager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-west-2
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets
            namespace: external-secrets
```

**Alternative: Sealed Secrets (GitOps-friendly)**

```bash
# Encrypt secrets for Git storage
kubeseal --format=yaml < secret.yaml > sealed-secret.yaml
```

### 4.3 Volumes & Persistent Storage

```
┌────────────────────────────────────────────────────────────────┐
│                   Storage Types (2026)                          │
├────────────────────────────────────────────────────────────────┤
│  emptyDir        │ Temporary, dies with Pod                    │
│  configMap       │ Mount ConfigMap as files                    │
│  secret          │ Mount Secret as files                       │
│  PV/PVC          │ Persistent, survives Pod death              │
│  CSI Drivers     │ AWS EBS, GCP PD, Azure Disk, Ceph, etc.    │
│  ephemeral       │ 2026: Inline volume claims                  │
└────────────────────────────────────────────────────────────────┘
```

```yaml
# PersistentVolumeClaim with modern storage class
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: gp3           # 2026: Use gp3 on AWS, premium-rwo on GKE
  volumeMode: Filesystem
```

---

## Phase 5: Networking Deep Dive (Week 4)

### 5.1 Service Types

```
┌─────────────────────────────────────────────────────────────┐
│                      SERVICE TYPES                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ClusterIP (default)     NodePort           LoadBalancer    │
│  ┌─────────────────┐    ┌─────────────────┐ ┌─────────────┐ │
│  │  Internal only  │    │ External via    │ │ Cloud LB    │ │
│  │  10.96.0.1:80   │    │ Node:30000-32767│ │ provisions  │ │
│  └─────────────────┘    └─────────────────┘ └─────────────┘ │
│         │                      │                   │         │
│         ▼                      ▼                   ▼         │
│    Pod-to-Pod            Dev/Testing          Production    │
│    communication         quick access         (use Gateway  │
│                                               API instead)  │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 Gateway API (2026 Standard - Replaces Ingress)

Gateway API is the **new standard** for HTTP routing in Kubernetes, replacing the legacy Ingress resource.

**Why Gateway API over Ingress:**
- Role-oriented design (infra team vs app team)
- Better support for advanced routing
- Native support for TCP/UDP, gRPC, TLS
- Portable across implementations

```yaml
# GatewayClass - Managed by infrastructure team
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: cilium
spec:
  controllerName: io.cilium/gateway-controller

---
# Gateway - Managed by platform/infra team
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: production-gateway
  namespace: gateway-system
spec:
  gatewayClassName: cilium
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: wildcard-tls
        kind: Secret
    allowedRoutes:
      namespaces:
        from: Selector
        selector:
          matchLabels:
            gateway-access: "true"

---
# HTTPRoute - Managed by application team
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app-route
  namespace: my-app
spec:
  parentRefs:
  - name: production-gateway
    namespace: gateway-system
  hostnames:
  - "myapp.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-service
      port: 80
      weight: 100
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: frontend-service
      port: 80
```

### 5.3 CNI: Cilium (2026 Recommendation)

Cilium is the **recommended CNI** for new clusters in 2026:
- eBPF-based (faster than iptables)
- Built-in observability (Hubble)
- Network policies with L7 awareness
- Service mesh without sidecars
- Gateway API implementation included

```yaml
# Cilium Network Policy (L7 aware)
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
metadata:
  name: api-access
spec:
  endpointSelector:
    matchLabels:
      app: my-api
  ingress:
  - fromEndpoints:
    - matchLabels:
        app: frontend
    toPorts:
    - ports:
      - port: "8080"
        protocol: TCP
      rules:
        http:
        - method: GET
          path: "/api/v1/.*"
```

### 5.4 Legacy Ingress (Still Supported)

```yaml
# Only use if Gateway API not available
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-app
            port:
              number: 80
```

---

## Phase 6: Production Patterns (Week 5-6)

### 6.1 Resource Management (2026 Best Practices)

```yaml
spec:
  containers:
  - name: app
    resources:
      requests:                    # Scheduling guarantee
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"            # OOMKill if exceeded
        # cpu: "500m"              # 2026: Often omit CPU limits (causes throttling)
```

**2026 Guidance:**
- Always set memory limits (prevents node OOM)
- Consider omitting CPU limits (prevents throttling)
- Use Vertical Pod Autoscaler (VPA) for right-sizing recommendations

```yaml
# VPA for right-sizing recommendations
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Off"   # Recommendation only, don't auto-apply
```

### 6.2 Health Checks (Complete Pattern)

```yaml
spec:
  containers:
  - name: app
    startupProbe:           # Wait for app to start
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30  # 30 * 10s = 5 min max startup
      periodSeconds: 10
    livenessProbe:          # Restart if unhealthy
      httpGet:
        path: /healthz
        port: 8080
      periodSeconds: 15
      failureThreshold: 3
    readinessProbe:         # Remove from service if not ready
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
      failureThreshold: 3
```

### 6.3 Deployment Strategies

```
┌─────────────────────────────────────────────────────────────┐
│                  DEPLOYMENT STRATEGIES                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Rolling Update (default)                                    │
│  ┌───┐ ┌───┐ ┌───┐     ┌───┐ ┌───┐ ┌───┐                   │
│  │v1 │ │v1 │ │v1 │  →  │v2 │ │v2 │ │v1 │  → all v2        │
│  └───┘ └───┘ └───┘     └───┘ └───┘ └───┘                   │
│                                                              │
│  Blue-Green (with Argo Rollouts)                            │
│  ┌───────────┐         ┌───────────┐                        │
│  │  Blue v1  │ ──────► │ Green v2  │  (instant switch)     │
│  └───────────┘         └───────────┘                        │
│                                                              │
│  Canary (with Argo Rollouts or Flagger)                     │
│  ┌───────────┐   ┌───┐                                      │
│  │  v1 (90%) │ + │v2 │  (gradual traffic shift)            │
│  └───────────┘   └───┘                                      │
└─────────────────────────────────────────────────────────────┘
```

```yaml
# Argo Rollouts Canary (2026 standard for advanced deployments)
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 5
  selector:
    matchLabels:
      app: my-app
  template:
    # ... pod template
  strategy:
    canary:
      steps:
      - setWeight: 10
      - pause: {duration: 5m}
      - setWeight: 30
      - pause: {duration: 5m}
      - setWeight: 50
      - pause: {duration: 5m}
      analysis:
        templates:
        - templateName: success-rate
        startingStep: 1
```

### 6.4 Autoscaling (2026: KEDA is Standard)

**HPA for CPU/Memory:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  behavior:                          # 2026: Fine-grained scaling behavior
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

**KEDA for Event-Driven Scaling (2026 Standard):**

```yaml
# Scale based on Kafka lag, SQS queue depth, Prometheus metrics, etc.
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: my-app-scaler
spec:
  scaleTargetRef:
    name: my-app
  minReplicaCount: 1
  maxReplicaCount: 100
  triggers:
  - type: kafka
    metadata:
      bootstrapServers: kafka.default.svc:9092
      consumerGroup: my-consumer-group
      topic: my-topic
      lagThreshold: "100"
  - type: prometheus
    metadata:
      serverAddress: http://prometheus:9090
      metricName: http_requests_total
      threshold: "1000"
      query: sum(rate(http_requests_total{app="my-app"}[2m]))
```

---

## Phase 7: Advanced Workloads (Week 7)

### 7.1 Workload Types

| Type | Use Case | Example |
|------|----------|---------|
| **Deployment** | Stateless apps | Web servers, APIs |
| **StatefulSet** | Stateful apps | Databases, Kafka |
| **DaemonSet** | One per node | Log collectors, monitoring agents |
| **Job** | Run to completion | Batch processing, migrations |
| **CronJob** | Scheduled tasks | Backups, reports |

### 7.2 StatefulSet for Databases

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      securityContext:
        fsGroup: 999
        runAsUser: 999
        runAsNonRoot: true
      containers:
      - name: postgres
        image: postgres:16-alpine
        ports:
        - containerPort: 5432
          name: postgres
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secrets
              key: password
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 50Gi
```

### 7.3 Jobs and CronJobs

```yaml
# CronJob with 2026 best practices
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
spec:
  schedule: "0 2 * * *"          # 2 AM daily
  concurrencyPolicy: Forbid       # Don't overlap
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  jobTemplate:
    spec:
      ttlSecondsAfterFinished: 86400  # Cleanup after 24h
      backoffLimit: 3
      template:
        spec:
          securityContext:
            runAsNonRoot: true
            runAsUser: 1000
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: backup-tool:v1.0
            resources:
              requests:
                memory: "256Mi"
                cpu: "100m"
              limits:
                memory: "512Mi"
```

---

## Phase 8: Security (Week 8)

### 8.1 Pod Security Standards (2026: Enforced by Default)

Pod Security Standards replace the deprecated PodSecurityPolicy:

```yaml
# Namespace-level enforcement
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

**Security Levels:**
| Level | Description |
|-------|-------------|
| `privileged` | Unrestricted (avoid) |
| `baseline` | Minimally restrictive |
| `restricted` | Hardened (recommended for production) |

### 8.2 RBAC - Role-Based Access Control

```yaml
# ServiceAccount for workload identity
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  annotations:
    # AWS IRSA
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/my-app-role
    # GCP Workload Identity
    iam.gke.io/gcp-service-account: my-app@project.iam.gserviceaccount.com

---
# Role with minimal permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: my-app
  name: my-app-role
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["my-app-secrets"]  # Specific secret only
  verbs: ["get"]

---
# Bind to ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-app-binding
  namespace: my-app
subjects:
- kind: ServiceAccount
  name: my-app
roleRef:
  kind: Role
  name: my-app-role
  apiGroup: rbac.authorization.k8s.io
```

### 8.3 Security Best Practices Checklist (2026)

```yaml
# Complete security context example
spec:
  serviceAccountName: my-app          # Dedicated service account
  automountServiceAccountToken: false # Disable if not needed
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
  volumes:
  - name: tmp                         # Writable tmp for read-only rootfs
    emptyDir: {}
```

**Checklist:**
- [ ] Pod Security Standards enforced (`restricted` level)
- [ ] Run as non-root user
- [ ] Read-only root filesystem
- [ ] Drop all capabilities
- [ ] Use dedicated ServiceAccounts
- [ ] External secrets management (ESO/Vault)
- [ ] Network policies in place
- [ ] Image scanning in CI/CD (Trivy, Grype)
- [ ] Signed images (Sigstore/Cosign)
- [ ] Runtime security (Falco, Tetragon)
- [ ] Audit logging enabled

### 8.4 Image Security (2026)

```bash
# Scan images with Trivy
trivy image my-app:v1.0.0

# Sign images with Cosign
cosign sign --key cosign.key my-registry/my-app:v1.0.0

# Verify signatures in cluster with Kyverno
```

```yaml
# Kyverno policy: Only allow signed images
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: verify-image-signature
spec:
  validationFailureAction: Enforce
  background: false
  rules:
  - name: verify-signature
    match:
      any:
      - resources:
          kinds:
          - Pod
    verifyImages:
    - imageReferences:
      - "my-registry/*"
      attestors:
      - entries:
        - keys:
            publicKeys: |
              -----BEGIN PUBLIC KEY-----
              ...
              -----END PUBLIC KEY-----
```

---

## Phase 9: Observability (Week 9)

### 9.1 The Three Pillars (2026 Stack)

```
┌─────────────────────────────────────────────────────────────┐
│                OBSERVABILITY STACK (2026)                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   LOGS               METRICS             TRACES             │
│   ┌───────────┐     ┌────────────┐     ┌────────────┐      │
│   │  Grafana  │     │ Prometheus │     │   Tempo    │      │
│   │   Loki    │     │  + Mimir   │     │  + Jaeger  │      │
│   │  + Alloy  │     │  + Thanos  │     │            │      │
│   └───────────┘     └────────────┘     └────────────┘      │
│        │                  │                  │              │
│        └──────────────────┴──────────────────┘              │
│                           │                                 │
│                    ┌─────────────┐                          │
│                    │   Grafana   │  (Unified Dashboard)     │
│                    └─────────────┘                          │
│                           │                                 │
│                    ┌─────────────┐                          │
│                    │   Hubble    │  (Cilium Observability)  │
│                    └─────────────┘                          │
└─────────────────────────────────────────────────────────────┘
```

### 9.2 Prometheus + Grafana (Kubernetes Monitoring)

```yaml
# PodMonitor (preferred over ServiceMonitor for pods)
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  podMetricsEndpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### 9.3 OpenTelemetry (2026 Standard)

```yaml
# OpenTelemetry Collector for unified telemetry
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-collector
spec:
  mode: deployment
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
    processors:
      batch:
        timeout: 10s
    exporters:
      prometheus:
        endpoint: 0.0.0.0:8889
      otlp:
        endpoint: tempo:4317
        tls:
          insecure: true
      loki:
        endpoint: http://loki:3100/loki/api/v1/push
    service:
      pipelines:
        metrics:
          receivers: [otlp]
          processors: [batch]
          exporters: [prometheus]
        traces:
          receivers: [otlp]
          processors: [batch]
          exporters: [otlp]
        logs:
          receivers: [otlp]
          processors: [batch]
          exporters: [loki]
```

### 9.4 Alerting

```yaml
# PrometheusRule for alerting
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: my-app-alerts
spec:
  groups:
  - name: my-app
    rules:
    - alert: HighErrorRate
      expr: |
        sum(rate(http_requests_total{status=~"5.."}[5m]))
        / sum(rate(http_requests_total[5m])) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate detected"
        description: "Error rate is {{ $value | humanizePercentage }}"
    - alert: PodCrashLooping
      expr: |
        increase(kube_pod_container_status_restarts_total[1h]) > 5
      for: 5m
      labels:
        severity: warning
```

---

## Phase 10: Production Deployment (Week 10-12)

### 10.1 Production Checklist (2026)

#### Cluster Setup
- [ ] Managed Kubernetes (EKS, GKE, AKS) or mature self-managed
- [ ] Multi-AZ deployment (minimum 3 zones)
- [ ] Node auto-scaling configured (Karpenter on AWS)
- [ ] Container runtime: containerd with gVisor/Kata for isolation
- [ ] Kubernetes version: Stay within N-2 of latest

#### Networking
- [ ] CNI: Cilium (preferred) or Calico
- [ ] Gateway API configured (replace Ingress)
- [ ] TLS certificates automated (cert-manager + Let's Encrypt)
- [ ] Network policies enforced (default deny)
- [ ] Service mesh if needed (Cilium, Istio, or Linkerd)

#### Storage
- [ ] Storage classes defined (fast SSD, standard)
- [ ] Backup strategy (Velero)
- [ ] CSI drivers for cloud storage

#### Security
- [ ] Pod Security Standards: `restricted`
- [ ] RBAC with least privilege
- [ ] External secrets (ESO + Vault/AWS SM)
- [ ] Image scanning + signing (Trivy + Cosign)
- [ ] Policy enforcement (Kyverno or Gatekeeper)
- [ ] Runtime security (Falco or Tetragon)
- [ ] Workload identity (IRSA/Workload Identity)

#### Observability
- [ ] Metrics: Prometheus/VictoriaMetrics + Grafana
- [ ] Logs: Loki + Alloy/Vector
- [ ] Traces: Tempo/Jaeger
- [ ] OpenTelemetry instrumentation
- [ ] Alerting configured with runbooks
- [ ] SLOs defined and monitored

#### Operations
- [ ] GitOps: ArgoCD or Flux
- [ ] Progressive delivery: Argo Rollouts
- [ ] CI/CD pipeline with security gates
- [ ] Disaster recovery tested
- [ ] Cost monitoring (OpenCost, Kubecost)
- [ ] Capacity planning

### 10.2 Managed vs Self-Managed (2026)

| Aspect | Managed (EKS/GKE/AKS) | Self-Managed |
|--------|----------------------|--------------|
| Control Plane | Provider manages | You manage |
| Upgrades | Automated/simplified | Manual |
| Support | Vendor SLA | Community |
| Cost | $70-150/month + nodes | Nodes only |
| Customization | Limited | Full control |
| **2026 Recommendation** | **Default choice** | Edge/air-gapped only |

### 10.3 GitOps with ArgoCD (2026 Standard)

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
  finalizers:
  - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/org/k8s-manifests
    targetRevision: main
    path: apps/my-app/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=foreground
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

### 10.4 Cost Optimization (2026)

```yaml
# Karpenter Provisioner (AWS) - Right-size nodes automatically
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
metadata:
  name: default
spec:
  requirements:
  - key: karpenter.sh/capacity-type
    operator: In
    values: ["spot", "on-demand"]
  - key: kubernetes.io/arch
    operator: In
    values: ["amd64", "arm64"]
  limits:
    resources:
      cpu: 1000
      memory: 1000Gi
  providerRef:
    name: default
  ttlSecondsAfterEmpty: 30
  consolidation:
    enabled: true
```

---

## 2026 Tool Recommendations Summary

| Category | 2026 Recommendation | Alternative |
|----------|---------------------|-------------|
| **Local Dev** | kind | k3d, minikube |
| **CNI** | Cilium | Calico |
| **Ingress** | Gateway API | nginx-ingress |
| **Secrets** | External Secrets Operator | Sealed Secrets |
| **Autoscaling** | KEDA + HPA | HPA only |
| **Deployments** | Argo Rollouts | Flagger |
| **GitOps** | ArgoCD | Flux |
| **Observability** | Grafana Stack (Loki/Tempo/Mimir) | Datadog, New Relic |
| **Policy** | Kyverno | OPA Gatekeeper |
| **Security** | Falco + Tetragon | Sysdig |
| **Cost** | OpenCost | Kubecost |
| **Node Scaling** | Karpenter (AWS) | Cluster Autoscaler |

---

## Learning Resources (2026)

### Official Documentation
- [Kubernetes.io](https://kubernetes.io/docs/)
- [Gateway API](https://gateway-api.sigs.k8s.io/)
- [Cilium Documentation](https://docs.cilium.io/)

### Interactive Learning
- [Killercoda](https://killercoda.com/) - Free browser-based labs
- [KodeKloud](https://kodekloud.com/) - Hands-on courses
- [Instruqt](https://play.instruqt.com/) - Vendor labs

### Certifications Path
```
KCNA (Kubernetes and Cloud Native Associate) - Entry level
        │
        ▼
CKA (Certified Kubernetes Administrator) - Operations
        │
        ├── CKAD (Application Developer) - Development
        │
        └── CKS (Security Specialist) - Security
```

### Books (2026 Editions)
- "Kubernetes Up & Running" 3rd Edition
- "Kubernetes Patterns" 2nd Edition
- "Production Kubernetes" 2nd Edition
- "Learning eBPF" - For Cilium deep-dive

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│                 K8S QUICK REFERENCE (2026)                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  OBJECT HIERARCHY:                                           │
│  Cluster → Namespace → Deployment → ReplicaSet → Pod        │
│                                                              │
│  COMMON COMMANDS:                                            │
│  kubectl get <resource>           # List resources          │
│  kubectl describe <resource>      # Details                 │
│  kubectl logs <pod>               # View logs               │
│  kubectl exec -it <pod> -- sh     # Shell access           │
│  kubectl apply -f <file>          # Create/update          │
│  kubectl delete -f <file>         # Delete                 │
│  kubectl debug <pod> --image=busybox  # Debug container    │
│                                                              │
│  RESOURCE SHORTNAMES:                                        │
│  po, deploy, svc, ns, pv, pvc, cm, sa, ing, netpol         │
│                                                              │
│  DEBUGGING:                                                  │
│  kubectl get events --sort-by='.lastTimestamp'             │
│  kubectl top pods                                           │
│  kubectl describe pod <name>                                │
│  stern <app-name>                 # Multi-pod logs         │
│                                                              │
│  2026 TOOLS:                                                 │
│  k9s                              # Terminal UI             │
│  kubectx/kubens                   # Context switching       │
│  stern                            # Log aggregation         │
│  kubecolor                        # Colorized output        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Suggested Learning Timeline

| Week | Focus | Milestone |
|------|-------|-----------|
| 1-2 | Core Concepts | Deploy first app with security context |
| 3 | Configuration | External Secrets, ConfigMaps |
| 4 | Networking | Gateway API, Services |
| 5-6 | Production Patterns | Health checks, KEDA autoscaling |
| 7 | Advanced Workloads | StatefulSets, Jobs |
| 8 | Security | Pod Security Standards, RBAC, Kyverno |
| 9 | Observability | Prometheus, Loki, Grafana |
| 10-12 | Production | GitOps with ArgoCD, real deployment |

---

*Last updated: January 2026*
*Kubernetes version coverage: 1.29 - 1.32*
