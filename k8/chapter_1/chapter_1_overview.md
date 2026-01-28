# Chapter 1: Pods - Your First Container in K8s

> **Goal:** Run a single container in Kubernetes and understand the fundamental building block

---

## What You'll Learn

```
┌─────────────────────────────────────────────────────────────┐
│                    CHAPTER 1 OVERVIEW                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Pods are the smallest deployable units in Kubernetes      │
│                                                             │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                      POD                            │   │
│   │   ┌───────────┐   ┌───────────┐                     │   │
│   │   │ Container │   │ Container │  (optional sidecar) │   │
│   │   │  (nginx)  │   │  (logs)   │                     │   │
│   │   └───────────┘   └───────────┘                     │   │
│   │         │               │                           │   │
│   │         └───────┬───────┘                           │   │
│   │                 │                                   │   │
│   │         Shared Network & Storage                    │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Core Concepts

### 1.1 What is a Pod?

- **Smallest deployable unit** in Kubernetes (not a container!)
- Wraps one or more containers that share:
  - Network namespace (same IP, can talk via localhost)
  - Storage volumes
  - Pod-level resource limits
- Think of it as a "logical host" for your containers

### 1.2 Pod YAML Anatomy

Every Kubernetes resource follows this structure:

```yaml
apiVersion: v1          # API version for Pods
kind: Pod               # Resource type
metadata:
  name: my-pod          # Unique name in namespace
  labels:               # Key-value pairs for organization
    app: nginx
spec:
  containers:           # List of containers in this pod
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

### 1.3 kubectl Essentials

Commands you'll use constantly:

| Command | Purpose |
|---------|---------|
| `kubectl apply -f <file>` | Create/update resource |
| `kubectl get pods` | List pods |
| `kubectl describe pod <name>` | Detailed pod info |
| `kubectl logs <pod>` | View container logs |
| `kubectl exec -it <pod> -- sh` | Shell into container |
| `kubectl delete pod <name>` | Delete pod |

### 1.4 Pod Lifecycle

```
┌──────────┐    ┌─────────┐    ┌─────────────────┐
│ Pending  │───▶│ Running │───▶│ Succeeded/Failed│
└──────────┘    └─────────┘    └─────────────────┘
     │               │
     │               ▼
     │         ┌─────────┐
     └────────▶│ Unknown │  (node communication lost)
               └─────────┘
```

**States explained:**
- **Pending**: Pod accepted, waiting for scheduling/image pull
- **Running**: Pod bound to node, at least one container running
- **Succeeded**: All containers terminated successfully (exit 0)
- **Failed**: All containers terminated, at least one failed
- **Unknown**: Pod state cannot be determined

---

## Mini-Project

```
🎯 Deploy nginx in a Pod, exec into it, serve a custom HTML page

   Steps:
   1. Create nginx-pod.yaml
   2. Apply it: kubectl apply -f nginx-pod.yaml
   3. Verify: kubectl get pods
   4. Exec in: kubectl exec -it nginx-pod -- sh
   5. Create custom HTML: echo "<h1>Hello K8s!</h1>" > /usr/share/nginx/html/index.html
   6. Test: curl localhost:80 (from inside the pod)

   Verify with: kubectl exec into pod, curl localhost:80
```

---

## Folder Structure

```
chapter_1/
├── chapter_1_overview.md        # This file
├── 1.1_what_is_a_pod.md         # Deep dive into pods
├── 1.2_pod_yaml_anatomy.md      # YAML structure explained
├── 1.3_kubectl_essentials.md    # kubectl commands tutorial
├── 1.4_pod_lifecycle.md         # States and transitions
├── mini_project/
│   ├── nginx-pod.yaml           # Pod manifest
│   └── README.md                # Project instructions
└── summary.md                   # Chapter recap
```

---

## Key Takeaways

1. **Pods, not containers** - K8s manages pods; containers live inside pods
2. **Ephemeral by design** - Pods can be deleted/recreated anytime
3. **Don't use bare pods in production** - Use Deployments (Chapter 2) for self-healing
4. **Labels matter** - They're how K8s selects and groups resources

---

## What's Next?

In **Chapter 2: Deployments & ReplicaSets**, you'll learn:
- Why running bare pods is a bad idea
- How to ensure N pods are always running
- Rolling updates and rollbacks

---

## Quick Commands Reference

```bash
# Create a pod
kubectl apply -f nginx-pod.yaml

# Watch pods in real-time
kubectl get pods -w

# Get pod details
kubectl describe pod nginx-pod

# View logs
kubectl logs nginx-pod

# Stream logs (follow)
kubectl logs -f nginx-pod

# Execute command in pod
kubectl exec nginx-pod -- cat /etc/nginx/nginx.conf

# Interactive shell
kubectl exec -it nginx-pod -- /bin/sh

# Port forward for local testing
kubectl port-forward pod/nginx-pod 8080:80

# Delete pod
kubectl delete pod nginx-pod
```

---

*Chapter 1 of Kubernetes 101: From Zero to Production*
