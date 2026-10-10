# Kubernetes Playground

A working reference for Kubernetes: commands, manifests, concepts and troubleshooting, built up while running a local cluster with kind.

## What's inside

| File / Folder | Purpose |
|---------------|---------|
| [cheatsheet.md](cheatsheet.md) | kubectl commands grouped by task |
| [troubleshooting.md](troubleshooting.md) | Debugging flow and a symptom -> cause -> fix table |
| [manifests/](manifests) | Working YAML files with comments on the non-obvious fields |
| [notes/concepts.md](notes/concepts.md) | Core concepts in short, plain language |
| [notes/labs.md](notes/labs.md) | Log of experiments: what I ran, what happened, what I learned |

## Environment

- Local cluster: [kind](https://kind.sigs.k8s.io/) (Kubernetes in Docker)
- CLI: `kubectl`

## Covered so far

- Cluster basics, architecture and kubeconfig
- kubectl essentials
- Pods
- Manifests and YAML
- Labels, selectors and annotations
- ReplicaSets
- Deployments, scaling, rolling updates and rollbacks
- Services: ClusterIP, NodePort, LoadBalancer, ExternalName

## Next

- ConfigMaps and Secrets
- Ingress
- Volumes and PersistentVolumeClaims