# Starting Your Kind Cluster

> Quick reference for starting your kind cluster after a restart

---

## The Problem

After your Mac restarts or Podman stops, you may see:

```bash
kubectl get nodes
# Error: connection refused
```

This happens because the kind container is stopped, not deleted.

---

## Quick Fix (90% of cases)

```bash
# 1. Start the kind container
podman start playground-control-plane

# 2. Verify it's running
podman ps

# 3. Test kubectl
kubectl get nodes
```

That's it! Your cluster should be back.

---

## Troubleshooting Flow

```text
┌─────────────────────────────────────────────────────────────────┐
│              CLUSTER NOT WORKING?                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Step 1: Is Podman machine running?                            │
│   ─────────────────────────────────                             │
│   $ podman machine list                                         │
│                                                                 │
│   If stopped:                                                   │
│   $ podman machine start                                        │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   Step 2: Is the kind container running?                        │
│   ──────────────────────────────────────                        │
│   $ podman ps -a                                                │
│                                                                 │
│   If stopped (Exited status):                                   │
│   $ podman start playground-control-plane                       │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   Step 3: Does the cluster exist?                               │
│   ───────────────────────────────                               │
│   $ kind get clusters                                           │
│                                                                 │
│   If no clusters listed:                                        │
│   $ kind create cluster --name playground --config kind-config.yaml│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Common Commands

| What | Command |
|------|---------|
| Check Podman machine | `podman machine list` |
| Start Podman machine | `podman machine start` |
| List all containers | `podman ps -a` |
| Start kind container | `podman start playground-control-plane` |
| Stop kind container | `podman stop playground-control-plane` |
| List kind clusters | `kind get clusters` |
| Delete cluster | `kind delete cluster --name playground` |
| Create cluster | `kind create cluster --name playground --config kind-config.yaml` |

---

## Understanding the Layers

```text
┌─────────────────────────────────────────────────────────────────┐
│              THE STACK                                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  kubectl                                                │   │
│   │  (talks to API server on 127.0.0.1:XXXXX)               │   │
│   └─────────────────────────────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Kind Container: playground-control-plane               │   │
│   │  (contains Kubernetes control plane + node)             │   │
│   └─────────────────────────────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Podman Machine (Linux VM)                              │   │
│   │  (runs containers on Mac)                               │   │
│   └─────────────────────────────────────────────────────────┘   │
│                           │                                     │
│                           ▼                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Your Mac (macOS)                                       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   If ANY layer is stopped, kubectl won't work!                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Why "Connection Refused"?

```text
kubectl → tries 127.0.0.1:55145 → nothing listening → connection refused

The port number (55145) is dynamic - it changes each time the
container starts. kubectl gets it from ~/.kube/config which
kind updates automatically.
```

---

## Nuclear Option: Start Fresh

If nothing works, delete and recreate:

```bash
# Delete the cluster
kind delete cluster --name playground

# Recreate it
kind create cluster --name playground --config kind-config.yaml

# Verify
kubectl get nodes
```

Note: This deletes all your pods, deployments, services, etc.

---

## One-Liner Health Check

```bash
# Check everything at once
podman machine list && podman ps && kubectl get nodes
```

If all three commands succeed, your cluster is healthy!
