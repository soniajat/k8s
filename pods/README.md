# 03 - Pods

## Quick Concepts

| Concept | What It Means |
|---|---|
| Pod | Smallest deployable unit. Wraps one or more containers that share network and storage. |
| Pod phase | High-level state of a pod: `Pending`, `Running`, `Succeeded`, `Failed`, `Unknown`. |
| `Pending` | Accepted by the cluster, but not running yet (scheduling or image pull). |
| `Running` | Bound to a node, with at least one container running. |
| `Succeeded` / `Failed` | All containers exited with success / at least one exited with an error. |
| `CrashLoopBackOff` | Container keeps crashing and Kubernetes keeps restarting it with increasing delay. |
| `ImagePullBackOff` | Image cannot be pulled (wrong name, tag, or registry access). |

## Commands to Remember

| Task | Command |
|---|---|
| Run a pod | `kubectl run nginx --image=nginx` |
| List pods and status | `kubectl get pods` |
| Pod details and events | `kubectl describe pod nginx` |
| View logs | `kubectl logs nginx` |
| Follow logs live | `kubectl logs -f nginx` |
| Logs of a specific container | `kubectl logs nginx -c <container>` |
| Shell inside a pod | `kubectl exec -it nginx -- /bin/sh` |
| Run a single command inside | `kubectl exec nginx -- ls /` |
| Delete a pod | `kubectl delete pod nginx` |

## Key Takeaways

- A pod is not a container. It is a wrapper around one or more containers.
- A standalone pod that is deleted is **not** recreated. Controllers like Deployments handle that.
- Pod troubleshooting order: `get` (status) -> `describe` (events) -> `logs` (app output) -> `exec` (look inside).
- The `--` in `kubectl exec` separates kubectl flags from the command to run.