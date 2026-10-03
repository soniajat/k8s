# 01 - Fundamentals and Your First Cluster

Quick-reference notes for a DevOps engineer: why Kubernetes exists, how it is built, and how to get a local cluster running with `kind` and `kubectl`.

---

## Part 1: Introduction to Kubernetes

### Quick Concepts

| Concept | What It Means |
|---|---|
| Problem with Docker | Docker runs containers on a **single host**. It has no built-in multi-host scheduling, self-healing, scaling, or rolling updates. |
| Kubernetes (K8s) | An open-source platform that runs and manages containers across a **cluster of machines**. |
| Docker vs Kubernetes | Docker **builds and runs** containers. Kubernetes **orchestrates** them at scale. They complement each other. |
| Orchestration | Automating deploy, scale, networking, health checks, and recovery of containers across many hosts. |
| Desired state | You declare what you want (e.g. 3 replicas). Kubernetes continuously works to make reality match it. |
| Cluster | A set of machines (nodes) managed by Kubernetes as a single system. |
| Control plane | The "brain" of the cluster. It makes decisions and stores cluster state. |
| Worker node | A machine that actually runs your application containers. |
| Pod | The smallest deployable unit in Kubernetes. It wraps one or more containers. |

### Kubernetes Architecture

| Component | Where | Role |
|---|---|---|
| `kube-apiserver` | Control plane | Front door of the cluster. Every request (kubectl, controllers, kubelets) goes through it. |
| `etcd` | Control plane | Key-value store holding all cluster state. |
| `kube-scheduler` | Control plane | Picks which node a new pod should run on. |
| `kube-controller-manager` | Control plane | Runs controllers that watch state and fix drift from the desired state. |
| `kubelet` | Worker node | Agent that makes sure the pods assigned to its node are running and healthy. |
| `kube-proxy` | Worker node | Handles network rules so pods and services can communicate. |
| Container runtime | Worker node | Pulls images and runs containers (e.g. `containerd`). |

### How a Pod Gets Deployed (The Flow)

1. You run `kubectl` (or apply a manifest) and the request goes to the **API server**.
2. The API server authenticates and validates it, then saves the desired state in **etcd**.
3. The **scheduler** notices an unassigned pod and picks a suitable node.
4. The **kubelet** on that node sees the assignment.
5. The kubelet tells the **container runtime** to pull the image and start the containers.
6. The kubelet reports the pod status back to the API server.

### What Kubernetes Does for You

| Capability | Meaning |
|---|---|
| Self-healing | Restarts failed containers and replaces unhealthy pods. |
| Scaling | Scales workloads up or down manually or automatically. |
| Rolling updates and rollbacks | Updates apps gradually with the option to revert. |
| Service discovery and load balancing | Gives workloads stable networking and spreads traffic. |
| Config and secrets management | Separates configuration from container images. |
| Storage orchestration | Attaches and manages persistent storage for workloads. |

---

## Part 2: Your First Cluster

### Quick Concepts

| Concept | What It Means |
|---|---|
| `kind` | "Kubernetes IN Docker". Runs a local cluster where each node is a Docker container. Ideal for learning and testing. |
| `kubectl` | The command-line client used to talk to the Kubernetes API server. |
| Kubeconfig | A YAML file (default `~/.kube/config`) storing cluster addresses, credentials, and contexts. |
| Context | A saved combination of **cluster + user + namespace**. `kubectl` acts against the current context. |
| `kind` context naming | A kind cluster named `dev` creates the context `kind-dev`. |

### Commands to Remember

| Task | Command |
|---|---|
| Create a cluster (default name `kind`) | `kind create cluster` |
| Create a named cluster | `kind create cluster --name dev` |
| List kind clusters | `kind get clusters` |
| See nodes | `kubectl get nodes` |
| See nodes with more detail | `kubectl get nodes -o wide` |
| Check client and server version | `kubectl version` |
| Cluster endpoints info | `kubectl cluster-info` |
| List all contexts | `kubectl config get-contexts` |
| Show current context | `kubectl config current-context` |
| Switch context | `kubectl config use-context kind-dev` |
| View kubeconfig | `kubectl config view` |
| Delete a cluster | `kind delete cluster --name dev` |
| See kind nodes as containers | `docker ps` |

---

## Key Takeaways

- Kubernetes solves what Docker alone cannot: running containers reliably across **many machines**.
- It works on **desired state**. You declare it, and controllers keep reconciling reality to match.
- The **API server** is the single entry point. Every component and every `kubectl` call talks through it.
- The control plane **decides**. Worker nodes **run**.
- Pod deployment flow to remember: `kubectl` -> API server -> etcd -> scheduler -> kubelet -> container runtime.
- A `kind` cluster is just Docker containers acting as nodes, so `docker ps` shows them.
- Always check `kubectl config current-context` before running commands. Wrong-context mistakes are a classic way to hit the wrong cluster.
- Deleting a kind cluster also cleans up its context from kubeconfig.