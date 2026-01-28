# Chapter 2: Deployments & ReplicaSets

> **Goal:** Never run bare pods again - ensure your apps are always running and can update without downtime

---

## Why This Matters

```text
┌─────────────────────────────────────────────────────────────────┐
│              THE PROBLEM WITH BARE PODS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Chapter 1: You ran a single pod                               │
│                                                                 │
│   $ kubectl delete pod nginx                                    │
│   pod "nginx" deleted                                           │
│                                                                 │
│   And it's GONE. Forever. Nobody recreates it.                  │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   Chapter 2: You'll use Deployments                             │
│                                                                 │
│   $ kubectl delete pod nginx-abc123                             │
│   pod "nginx-abc123" deleted                                    │
│                                                                 │
│   $ kubectl get pods                                            │
│   nginx-xyz789   1/1   Running   0   2s   ← NEW POD APPEARED!   │
│                                                                 │
│   Kubernetes: "You wanted 1 pod? Here's a new one."             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## What You'll Learn

| Topic | What It Does |
|-------|--------------|
| **ReplicaSet** | Ensures N identical pods are always running |
| **Deployment** | Manages ReplicaSets + rolling updates + rollbacks |
| **Scaling** | Run 3, 10, or 100 pods with one command |
| **Rolling Updates** | Update your app with zero downtime |
| **Rollbacks** | Something broke? Go back to previous version |

---

## The Hierarchy

```text
┌─────────────────────────────────────────────────────────────────┐
│              DEPLOYMENT → REPLICASET → PODS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  DEPLOYMENT (nginx-deployment)                          │   │
│   │  "I want 3 nginx:1.25 pods, always"                     │   │
│   │                                                         │   │
│   │  ┌─────────────────────────────────────────────────┐    │   │
│   │  │  REPLICASET (nginx-deployment-abc123)           │    │   │
│   │  │  "I maintain exactly 3 pods"                    │    │   │
│   │  │                                                 │    │   │
│   │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐          │    │   │
│   │  │  │  POD 1  │  │  POD 2  │  │  POD 3  │          │    │   │
│   │  │  │ nginx   │  │ nginx   │  │ nginx   │          │    │   │
│   │  │  └─────────┘  └─────────┘  └─────────┘          │    │   │
│   │  │                                                 │    │   │
│   │  └─────────────────────────────────────────────────┘    │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Delete a pod? ReplicaSet creates a new one.                   │
│   Update the image? Deployment creates a new ReplicaSet.        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Chapter Sections

### 2.1 ReplicaSets - The Self-Healing Layer

**What you'll learn:**
- How ReplicaSets maintain desired pod count
- Label selectors and how pods are "owned"
- Why you should NOT create ReplicaSets directly

**Hands-on:**
```bash
# Create a ReplicaSet (for learning only)
kubectl apply -f replicaset.yaml

# Delete a pod - watch it come back!
kubectl delete pod <pod-name>
kubectl get pods -w

# Scale manually
kubectl scale replicaset nginx-rs --replicas=5
```

---

### 2.2 Deployments - The Full Package

**What you'll learn:**
- Deployment YAML anatomy
- How Deployments manage ReplicaSets
- The relationship between spec.replicas and actual pods

**Hands-on:**
```bash
# Create your first deployment
kubectl create deployment nginx --image=nginx:1.25 --replicas=3

# See the hierarchy
kubectl get deployments
kubectl get replicasets
kubectl get pods

# Notice the naming pattern:
# deployment:  nginx
# replicaset:  nginx-7d9b8c9f5c
# pods:        nginx-7d9b8c9f5c-abc12, nginx-7d9b8c9f5c-xyz34
```

---

### 2.3 Scaling - More Pods, Less Problems

**What you'll learn:**
- Manual scaling with `kubectl scale`
- Declarative scaling in YAML
- Horizontal Pod Autoscaler (HPA) basics

**Hands-on:**
```bash
# Scale up
kubectl scale deployment nginx --replicas=5

# Scale down
kubectl scale deployment nginx --replicas=2

# Watch pods appear/disappear
kubectl get pods -w
```

---

### 2.4 Rolling Updates - Zero Downtime Deployments

**What you'll learn:**
- How rolling updates work
- maxSurge and maxUnavailable
- Watching a rollout in progress

**Hands-on:**
```bash
# Update the image
kubectl set image deployment/nginx nginx=nginx:1.26

# Watch the rollout
kubectl rollout status deployment/nginx

# See what happened
kubectl get replicasets
# Old RS scaled to 0, new RS scaled to 3
```

```text
┌─────────────────────────────────────────────────────────────────┐
│              ROLLING UPDATE PROCESS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Before: 3 pods running nginx:1.25                             │
│   ┌───────┐  ┌───────┐  ┌───────┐                               │
│   │v1.25  │  │v1.25  │  │v1.25  │                               │
│   └───────┘  └───────┘  └───────┘                               │
│                                                                 │
│   Step 1: Start 1 new pod with nginx:1.26                       │
│   ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐                    │
│   │v1.25  │  │v1.25  │  │v1.25  │  │v1.26 ●│ ← new              │
│   └───────┘  └───────┘  └───────┘  └───────┘                    │
│                                                                 │
│   Step 2: Terminate 1 old pod                                   │
│   ┌───────┐  ┌───────┐  ┌───────┐                               │
│   │v1.25 ✗│  │v1.25  │  │v1.26  │                               │
│   └───────┘  └───────┘  └───────┘                               │
│                                                                 │
│   ... repeat until all pods are v1.26 ...                       │
│                                                                 │
│   After: 3 pods running nginx:1.26                              │
│   ┌───────┐  ┌───────┐  ┌───────┐                               │
│   │v1.26  │  │v1.26  │  │v1.26  │                               │
│   └───────┘  └───────┘  └───────┘                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 2.5 Rollbacks - Undo When Things Break

**What you'll learn:**
- Viewing rollout history
- Rolling back to previous version
- Rolling back to specific revision

**Hands-on:**
```bash
# See history
kubectl rollout history deployment/nginx

# Oops, v1.26 is broken! Rollback!
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2

# Check what's running now
kubectl describe deployment nginx | grep Image
```

---

## Labs

### Lab 1: Self-Healing Demo (10 min)

```bash
# Create deployment with 3 replicas
kubectl create deployment web --image=nginx --replicas=3

# Note the pod names
kubectl get pods

# Delete one pod
kubectl delete pod <pod-name>

# Immediately check - new pod created!
kubectl get pods

# Delete ALL pods at once
kubectl delete pods -l app=web

# They ALL come back!
kubectl get pods -w
```

**What you learned:** Deployments ensure your desired state is maintained.

---

### Lab 2: Rolling Update & Rollback (15 min)

```bash
# Create deployment
kubectl create deployment web --image=nginx:1.24

# Update to 1.25
kubectl set image deployment/web nginx=nginx:1.25
kubectl rollout status deployment/web

# Update to 1.26
kubectl set image deployment/web nginx=nginx:1.26
kubectl rollout status deployment/web

# Check history
kubectl rollout history deployment/web

# Rollback to 1.25
kubectl rollout undo deployment/web

# Verify
kubectl describe deployment web | grep Image
```

---

### Lab 3: Break and Fix (20 min)

```bash
# Create deployment
kubectl create deployment web --image=nginx:1.25 --replicas=3

# "Deploy" a broken image
kubectl set image deployment/web nginx=nginx:doesnotexist

# Watch it fail
kubectl rollout status deployment/web
# It will hang - some pods can't start

kubectl get pods
# You'll see ImagePullBackOff

# Rollback to fix
kubectl rollout undo deployment/web

# All healthy again
kubectl get pods
```

---

## Deployment YAML Anatomy

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                    # How many pods to maintain
  selector:
    matchLabels:
      app: nginx                 # Which pods belong to this deployment
  strategy:
    type: RollingUpdate          # or Recreate
    rollingUpdate:
      maxSurge: 1                # Extra pods during update
      maxUnavailable: 0          # Don't kill old before new is ready
  template:                      # Pod template (same as Pod spec)
    metadata:
      labels:
        app: nginx               # Must match selector!
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
```

---

## Quick Reference

### Essential Commands

```bash
# Create deployment
kubectl create deployment NAME --image=IMAGE --replicas=N

# Scale
kubectl scale deployment NAME --replicas=N

# Update image
kubectl set image deployment/NAME CONTAINER=IMAGE

# Rollout status
kubectl rollout status deployment/NAME

# Rollout history
kubectl rollout history deployment/NAME

# Rollback
kubectl rollout undo deployment/NAME

# Pause/resume rollout
kubectl rollout pause deployment/NAME
kubectl rollout resume deployment/NAME

# Restart all pods (recreate)
kubectl rollout restart deployment/NAME
```

### Deployment Strategies

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| `RollingUpdate` | Gradual replacement | Default, zero-downtime |
| `Recreate` | Kill all, then create new | Database migrations, breaking changes |

---

## Best Practices

1. **Always use Deployments** - Never bare pods or ReplicaSets directly
2. **Set resource requests/limits** - Prevents resource starvation
3. **Use readiness probes** - Traffic only goes to healthy pods
4. **Use liveness probes** - Restart unhealthy containers
5. **Set `maxUnavailable: 0`** - For true zero-downtime updates
6. **Record changes** - Use `--record` flag for better history

---

## What's Next?

In **Chapter 3: Services & Networking**, you'll learn:
- How to expose your deployment to other pods
- ClusterIP, NodePort, LoadBalancer
- Service discovery with DNS

---

## Sources

- [Kubernetes Official Docs - ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)
- [KodeKloud - Deployments & ReplicaSets](https://kodekloud.com/blog/day-4-deployments-replicasets-how-kubernetes-runs-and-manages-your-app/)
- [Plural.sh - Kubernetes Deployments Guide](https://www.plural.sh/blog/kubernetes-deployments-guide/)
- [Codefresh - Kubernetes Deployment Strategies](https://codefresh.io/learn/kubernetes-deployment/)

---

*Chapter 2 of Kubernetes 101: From Zero to Production*
