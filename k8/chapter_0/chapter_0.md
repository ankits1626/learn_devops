# Chapter 0: First Principles & Local Setup

## Goal

Understand **why** Kubernetes exists, learn its core architecture, then set up your Mac with a working local cluster.

By the end of this chapter, you'll have a 3-node Kubernetes cluster running locally in your playground.

---

## What We'll Cover

| Section | Topic | Type |
|---------|-------|------|
| 0.1 | Why Kubernetes? | Theory |
| 0.2 | Kubernetes Architecture | Theory |
| 0.3 | Containers 101 | Theory |
| 0.4 | YAML 101 | Theory |
| 0.5 | Networking Basics | Theory |
| 0.6 | Mac Setup (Podman, kubectl, kind) | Setup |
| 0.7 | Your First Cluster | Hands-on |

---

## Section Summaries

### 0.1 Why Kubernetes?

The evolution of deployment: Bare Metal → VMs → Containers → Kubernetes.

**Problems K8s solves:**
- Auto-healing (crashed containers restart automatically)
- Auto-scaling (handle traffic spikes)
- Rolling updates (zero-downtime deployments)
- Service discovery (containers find each other)

**Key concept:** Declarative vs Imperative - you describe WHAT you want, K8s figures out HOW.

---

### 0.2 Kubernetes Architecture

**Control Plane (Brain):**
- API Server - front door, all requests go through it
- Scheduler - decides which node runs which pod
- Controller Manager - watches and fixes state
- etcd - database storing all cluster state

**Worker Nodes (Muscle):**
- kubelet - agent on each node, manages pods
- kube-proxy - handles networking
- Container Runtime - runs containers (containerd)

**Pod:** Smallest deployable unit. Wraps one or more containers.

---

### 0.3 Containers 101

#### What is a Container?

A container packages your app + dependencies into a single unit that runs anywhere.

```text
┌─────────────────────────────────┐
│           CONTAINER             │
├─────────────────────────────────┤
│  Your App Code                  │
│  + Runtime (Node, Python, etc.) │
│  + Libraries & Dependencies     │
│  + System tools                 │
│                                 │
│  Everything needed to run       │
└─────────────────────────────────┘
```

#### Containers vs VMs

| Aspect | VM | Container |
|--------|-----|-----------|
| Size | Gigabytes | Megabytes |
| Startup | Minutes | Seconds |
| Isolation | Full OS | Shared kernel |
| Resource usage | Heavy | Lightweight |

#### Image vs Container

- **Image** = blueprint/recipe (read-only, stored on disk)
- **Container** = running instance of an image (has state, uses CPU/memory)

One image can spawn many containers:
```text
nginx:latest (image)
    ├── nginx-container-1 (running)
    ├── nginx-container-2 (running)
    └── nginx-container-3 (running)
```

#### Docker vs Podman

| Feature | Docker | Podman |
|---------|--------|--------|
| License | Paid for business | Free |
| Architecture | Daemon (background service) | Daemonless |
| Security | Runs as root | Rootless by default |
| Commands | `docker ...` | `podman ...` (identical!) |

**We use Podman** - it's free, more secure, and commands are the same as Docker.

#### Essential Container Commands

```bash
# Run a container
podman run nginx                    # Run nginx (foreground)
podman run -d nginx                 # Run in background (detached)
podman run -d -p 8080:80 nginx      # Map port 8080→80

# List containers
podman ps                           # Running containers
podman ps -a                        # All containers (including stopped)

# Container lifecycle
podman stop <container_id>          # Stop a container
podman start <container_id>         # Start a stopped container
podman rm <container_id>            # Remove a container
podman rm -f <container_id>         # Force remove (stop + remove)

# Inspect and debug
podman logs <container_id>          # View container output
podman logs -f <container_id>       # Follow logs (like tail -f)
podman exec -it <container_id> sh   # Shell into container

# Images
podman images                       # List local images
podman pull nginx                   # Download an image
podman rmi nginx                    # Remove an image
```

#### Building Images with Dockerfile

A `Dockerfile` is a recipe for building an image:

```dockerfile
# Start from a base image
FROM node:20-alpine

# Set working directory inside container
WORKDIR /app

# Copy package files first (for caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Expose port (documentation)
EXPOSE 3000

# Command to run when container starts
CMD ["npm", "start"]
```

Build and run:
```bash
podman build -t my-app .            # Build image, tag as "my-app"
podman run -d -p 3000:3000 my-app   # Run it
```

#### Image Naming Convention

```text
registry.io/namespace/image:tag
─────────── ───────── ───── ───
     │          │       │    │
     │          │       │    └── Version (default: latest)
     │          │       └─────── Image name
     │          └─────────────── User/organization
     └────────────────────────── Registry (default: docker.io)

Examples:
  nginx                          → docker.io/library/nginx:latest
  postgres:15                    → docker.io/library/postgres:15
  ghcr.io/myorg/myapp:v1.0.0    → GitHub Container Registry
```

#### Quick Practice

```bash
# 1. Run nginx
podman run -d -p 8080:80 --name my-nginx nginx

# 2. Check it's running
podman ps

# 3. Visit http://localhost:8080 in browser

# 4. View logs
podman logs my-nginx

# 5. Shell into container
podman exec -it my-nginx sh
ls /usr/share/nginx/html
exit

# 6. Stop and remove
podman rm -f my-nginx
```

---

### 0.4 YAML 101

Kubernetes uses YAML for all configuration. Master these basics:

#### What is YAML?

YAML = "YAML Ain't Markup Language" - a human-readable data format.

```yaml
# This is a comment
name: my-application    # key: value
version: 1.0
```

#### Key-Value Pairs

```yaml
name: nginx
image: nginx:latest
port: 80
enabled: true
```

#### Nested Objects (Indentation = 2 spaces)

```yaml
metadata:
  name: my-pod
  labels:
    app: web
    tier: frontend
```

**Important:** Use spaces, NOT tabs. Always 2-space indentation.

#### Lists (Arrays)

```yaml
# List of strings
fruits:
  - apple
  - banana
  - orange

# List of objects
containers:
  - name: nginx
    image: nginx
  - name: redis
    image: redis
```

#### Multi-line Strings

```yaml
# Literal block (preserves newlines)
description: |
  This is line 1
  This is line 2

# Folded block (joins lines with spaces)
summary: >
  This is a very long
  description that will
  become one line.
```

#### Common K8s YAML Structure

Every Kubernetes resource follows this pattern:

```yaml
apiVersion: v1              # API version
kind: Pod                   # Resource type
metadata:                   # Metadata about the resource
  name: my-pod
  labels:
    app: web
spec:                       # Specification (what you want)
  containers:
    - name: nginx
      image: nginx:latest
```

#### YAML Gotchas

```yaml
# Strings that look like other types - quote them!
version: "1.0"              # Without quotes: 1.0 becomes float
enabled: "true"             # Without quotes: true becomes boolean
port: "80"                  # Without quotes: 80 becomes integer

# Special characters need quotes
message: "Hello: World"     # Colon in string
path: "/api/v1"             # Slashes are fine without quotes
```

#### Validate Your YAML

```bash
# Check syntax before applying
kubectl apply -f my-file.yaml --dry-run=client

# Or use yamllint
brew install yamllint
yamllint my-file.yaml
```

---

### 0.5 Networking Basics

Understanding these concepts is essential for Kubernetes:

#### Ports

A port is a number (0-65535) that identifies a specific process/service on a machine.

```text
┌─────────────────────────────────────┐
│           YOUR COMPUTER             │
├─────────────────────────────────────┤
│                                     │
│   Port 80  → Web server (HTTP)      │
│   Port 443 → Web server (HTTPS)     │
│   Port 22  → SSH                    │
│   Port 3000 → Your Node.js app      │
│   Port 5432 → PostgreSQL            │
│                                     │
└─────────────────────────────────────┘
```

Common ports to know:

| Port | Service |
|------|---------|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 27017 | MongoDB |

#### localhost vs 0.0.0.0

```text
localhost (127.0.0.1)
  → Only accessible from the same machine
  → "Talk to myself"

0.0.0.0
  → Listen on ALL network interfaces
  → Accessible from other machines
  → Use this in containers!
```

**Container gotcha:** Apps inside containers must bind to `0.0.0.0`, not `localhost`, to be accessible from outside.

#### Port Mapping (Container → Host)

```bash
podman run -p 8080:80 nginx
#            ────  ──
#             │    └── Container port (nginx listens on 80)
#             └─────── Host port (you access via localhost:8080)
```

```text
┌──────────────────┐        ┌──────────────────┐
│    YOUR MAC      │        │    CONTAINER     │
│                  │        │                  │
│  localhost:8080 ─┼───────▶│ nginx on :80     │
│                  │        │                  │
└──────────────────┘        └──────────────────┘
```

#### DNS (Domain Name System)

DNS translates names to IP addresses:

```text
google.com  →  142.250.80.46
```

In Kubernetes, services get DNS names automatically:
```text
my-service              →  10.96.45.23
my-service.default      →  10.96.45.23  (with namespace)
my-service.default.svc  →  10.96.45.23  (full form)
```

#### IP Addresses

```text
IPv4: 192.168.1.100     (4 numbers, 0-255 each)
IPv6: 2001:db8::1       (longer, less common for now)

Special addresses:
  127.0.0.1    → localhost (this machine)
  10.x.x.x     → Private network (common in K8s)
  192.168.x.x  → Private network (home routers)
  172.16-31.x.x → Private network (Docker default)
```

#### Container Networking Model

```text
┌──────────────────────────────────────────────────────────┐
│                     HOST MACHINE                         │
│                                                          │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│   │ Container A │    │ Container B │    │ Container C │  │
│   │ 172.17.0.2  │    │ 172.17.0.3  │    │ 172.17.0.4  │  │
│   └──────┬──────┘    └──────┬──────┘    └──────┬──────┘  │
│          │                  │                  │         │
│          └──────────────────┼──────────────────┘         │
│                             │                            │
│                    ┌────────┴────────┐                   │
│                    │  Docker Bridge  │                   │
│                    │   172.17.0.1    │                   │
│                    └────────┬────────┘                   │
│                             │                            │
└─────────────────────────────┼────────────────────────────┘
                              │
                         Host Network
```

Containers can talk to each other via the bridge network.

---

### 0.6 Mac Setup

**Tools to install:**

| Tool | Purpose | Command |
|------|---------|---------|
| Podman | Container runtime | `brew install podman` |
| kubectl | K8s CLI | `brew install kubectl` |
| kind | Local clusters | `brew install kind` |
| k9s | Terminal UI | `brew install derailed/k9s/k9s` |
| kubectx | Context switching | `brew install kubectx` |

**Podman setup:**

```bash
podman machine init
podman machine set --rootful
podman machine start
```

**Shell config (~/.zshrc):**

```bash
# Kubernetes
export KIND_EXPERIMENTAL_PROVIDER=podman
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kga='kubectl get all'

# kubectl autocomplete
source <(kubectl completion zsh)
```

---

### 0.7 Your First Cluster

Create a 3-node cluster:

```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

```bash
kind create cluster --name playground --config kind-config.yaml
kubectl get nodes
```

Expected output:

```text
NAME                       STATUS   ROLES           AGE   VERSION
playground-control-plane   Ready    control-plane   2m    v1.31.0
playground-worker          Ready    <none>          90s   v1.31.0
playground-worker2         Ready    <none>          90s   v1.31.0
```

---

## kubectl Quick Reference

Essential commands you'll use constantly:

### Get Resources

```bash
kubectl get pods                    # List pods in current namespace
kubectl get pods -A                 # List pods in ALL namespaces
kubectl get pods -o wide            # More details (node, IP)
kubectl get nodes                   # List cluster nodes
kubectl get services                # List services
kubectl get all                     # List common resources
```

### Describe & Inspect

```bash
kubectl describe pod <name>         # Detailed info about a pod
kubectl describe node <name>        # Detailed info about a node
kubectl logs <pod-name>             # View pod logs
kubectl logs -f <pod-name>          # Follow logs (stream)
kubectl logs <pod> -c <container>   # Logs from specific container
```

### Create & Delete

```bash
kubectl run nginx --image=nginx     # Quick pod creation
kubectl apply -f file.yaml          # Create/update from file
kubectl delete pod <name>           # Delete a pod
kubectl delete -f file.yaml         # Delete resources in file
```

### Debug & Execute

```bash
kubectl exec -it <pod> -- sh        # Shell into pod
kubectl exec <pod> -- ls /app       # Run command in pod
kubectl port-forward <pod> 8080:80  # Forward local port to pod
```

### Context & Namespace

```bash
kubectl config get-contexts         # List available clusters
kubectl config use-context <name>   # Switch cluster
kubectl get ns                      # List namespaces
kubectl config set-context --current --namespace=<ns>  # Switch namespace
```

### Quick Tips

```bash
# Shorthand resource names
kubectl get po          # pods
kubectl get svc         # services
kubectl get deploy      # deployments
kubectl get ns          # namespaces
kubectl get no          # nodes

# Output formats
kubectl get pods -o yaml            # YAML output
kubectl get pods -o json            # JSON output
kubectl get pods -o name            # Just names

# Dry run (test without creating)
kubectl apply -f file.yaml --dry-run=client
```

---

## Playground Project

Your playground grows with each chapter:

```text
~/k8s-playground/
├── kind-config.yaml        # Chapter 0
├── pods/                   # Chapter 1
├── deployments/            # Chapter 2
├── services/               # Chapter 3
└── ...
```

---

## Prerequisites

- Mac with 8GB+ RAM (16GB recommended)
- macOS 12+ (Monterey or newer)
- Homebrew installed
- Basic terminal knowledge
- Familiarity with containers (even just `docker run hello-world`)

---

## Chapter Checklist

By the end of Chapter 0, you should be able to:

- [ ] Explain why Kubernetes exists (the 4 problems it solves)
- [ ] Name the Control Plane components and their roles
- [ ] Explain what a Pod is
- [ ] Run `kubectl get nodes` and see 3 Ready nodes
- [ ] Run a simple pod with `kubectl run nginx --image=nginx`

---

**Next:** Chapter 1 - Pods (The Building Blocks)
